# RLS Architecture

## RLS Role in the New Architecture
PostgreSQL Row Level Security (RLS) is the **secondary, defense-in-depth** authorization layer. The **primary** authorization layer is Express RBAC middleware, which validates the user's JWT and role before any database query executes.

RLS is written as a **guardrail**, not a business logic engine. It answers one question: "should this database role be allowed to read or write this row if the application sends this query?" Express has already answered the higher-level question "should this user be allowed to perform this operation?" before the query reaches the database.

RLS ensures that even if Express middleware is misconfigured, has a bug, or is bypassed, the database will independently enforce access control at the row level.

## How RLS Knows the Current User — Critical Implementation Detail

Prisma uses a **connection pool** (PgBouncer or the built-in Prisma pool). Connection-level `SET` statements would leak across requests sharing the same connection. Therefore, the session variable **must** be set inside a transaction boundary using `SET LOCAL`, which is scoped to the current transaction only.

### Implementation Pattern (TypeScript)

Every protected database operation uses a utility wrapper:

```typescript
// packages/database/src/client.ts
import { PrismaClient } from '@prisma/client';

export async function withUserContext<T>(
  userId: string,
  fn: (tx: any) => Promise<T>
): Promise<T> {
  return prisma.$transaction(async (tx) => {
    await tx.$executeRaw`SELECT set_config('app.user_id', ${userId}, true)`;
    return fn(tx);
  });
}
```

Usage in a route handler:
```typescript
// routes/events.ts
import { withUserContext } from '@nst/database';

router.get('/events', authenticate, async (req, res) => {
  const events = await withUserContext(req.user.id, (tx) =>
    tx.event.findMany({ where: { state: 'PUBLISHED' } })
  );
  res.json(events);
});
```

### RLS Denial Behavior (Prisma)
When RLS policies deny access to a row, the database driver (Prisma) surfaces this in different ways depending on the operation type:
- **`UPDATE` / `DELETE`**: RLS denial is completely silent at the database level. Prisma will attempt to update/delete rows matching the query, find 0 rows due to RLS, and throw a `P2025` error ("Record to update not found"). This intentionally mimics a 404 (Not Found).
- **`INSERT` / Raw Queries**: RLS denial throws a hard database exception. Prisma surfaces this as a raw SQL error with code `42501` (Insufficient Privilege). It does **not** throw Prisma's generic `P2004` (constraint failed). Application code must catch `err.code === '42501'` to return a 403 Forbidden.

### Why `set_config(..., true)` not `SET LOCAL`
`SELECT set_config('app.user_id', '<uuid>', true)` with the third argument `true` is equivalent to `SET LOCAL` — it is transaction-scoped. This is the preferred form when using Prisma's `$executeRaw` because it avoids raw SQL string interpolation for the setting name.

### What Happens Without the Transaction
If a query runs outside the `withUserContext` wrapper, `current_setting('app.user_id')` returns an empty string. RLS policies must handle this gracefully using `nullif(current_setting('app.user_id', true), '')::uuid` to avoid runtime errors.

### RLS Policy Reference
```sql
-- Safe pattern used in all RLS policies
CREATE OR REPLACE FUNCTION current_user_id() RETURNS uuid AS $$
  SELECT nullif(current_setting('app.user_id', true), '')::uuid;
$$ LANGUAGE sql STABLE;
```
Policies use `current_user_id()` for clean, consistent identity resolution.

### Pre-Authentication Writes (OAuth Upsert)
When the application needs to write to a protected table before a user's session is established (e.g., during OAuth login where `current_user_id()` cannot be set because the user does not exist yet), do **not** weaken the table's RLS policies to allow writes when `current_user_id() IS NULL`. Doing so breaks the zero-trust model and allows any query accidentally executing outside `withUserContext` to modify the table.

Instead, use a dedicated **`SECURITY DEFINER` Postgres function**. This explicitly elevates privileges for one highly specific operation while keeping the underlying table completely locked down:

```sql
CREATE OR REPLACE FUNCTION upsert_oauth_user(p_google_sub TEXT, p_email TEXT, p_full_name TEXT)
RETURNS SETOF users
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = public
AS $$
BEGIN
  RETURN QUERY
  INSERT INTO users (id, google_sub, email, full_name, global_role)
  VALUES (gen_random_uuid(), p_google_sub, p_email, p_full_name, 'STUDENT'::"GlobalRole")
  ON CONFLICT (google_sub) WHERE deleted_at IS NULL
  DO UPDATE SET
    email = EXCLUDED.email,
    full_name = EXCLUDED.full_name,
    updated_at = now()
  RETURNING *;
END;
$$;
```
This function hardcodes defaults (like `global_role`) and prevents updates to restricted fields like `deleted_at`, ensuring the privilege elevation cannot be abused.

## Trust Model
We operate on a **Zero Trust** internal model. An authenticated JWT proves *identity*. *Authorization* is derived from relational data (`club_memberships`) evaluated at query time — not stored in JWT claims.

## RLS Scope: What RLS Should and Should Not Do

**RLS should:**
- Enforce row visibility rules (e.g., only show `DRAFT` events to their organizers)
- Prevent writes to rows the user does not own
- Provide a safety net if Express RBAC is bypassed

**RLS should NOT:**
- Encode complex multi-step business logic (that belongs in Express route handlers)
- Replace Express RBAC checks (they are complementary, not alternatives)
- Perform heavy subqueries on every row scan (keep policies index-friendly)

## Policy Design Principles
1. **Per-transaction scoping**: Always use `withUserContext` wrapper. Never set `app.user_id` at connection level.
2. **Use the helper function**: Reference `current_user_id()` in all policies, not raw `current_setting(...)` calls.
3. **Minimize subqueries**: `EXISTS` subqueries must use indexed columns. Avoid nested subqueries.
4. **Explicit grants**: Default state is `DENY ALL`. Every access must be explicitly granted.
5. **Mirror RBAC**: Every RLS policy must have a corresponding Express RBAC check. Discrepancies are security defects.

## Security Invoker Views
Views used to filter records execute with the permissions of the calling user (`security_invoker = true`), meaning RLS cascades securely through views.

## RLS Lifecycle
Policies are defined in standard SQL migration files (`.sql`) and managed entirely via source control. They are applied by the database administrator on each deployment.
