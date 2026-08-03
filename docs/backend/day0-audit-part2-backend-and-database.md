# NST-Events Day-0 Repository Extraction Audit
# Part 2 of 4 — Backend Foundation, Database Foundation, Frontend & Dashboard

> Continues from Part 1. Authority: docs/backend/03-prisma-schema-plan.md,
> docs/database/*, docs/security/01-rls-architecture.md,
> docs/backend/02-repository-structure.md

---

## Section 5 — Backend Foundation Requirements

These are the exact files that must exist and compile before any domain feature
is written. Derived from Phase 0 + Phase 1 of the implementation roadmap.

### 5.1 Express App Bootstrap

**`apps/api/src/index.ts`** — HTTP server entry point
```typescript
import { createApp } from './app';
const app = createApp();
const PORT = process.env.PORT ?? 3000;
app.listen(PORT, () => console.log(`nst-api listening on :${PORT}`));
```

**`apps/api/src/app.ts`** — App factory (middleware chain)
```typescript
import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import cookieParser from 'cookie-parser';
import { errorHandler } from './middleware/error-handler';
import { authRouter } from './modules/auth/auth.router';
// ...all module routers imported and mounted

export function createApp() {
  const app = express();
  app.use(helmet());
  app.use(cors({ origin: process.env.ALLOWED_ORIGINS, credentials: true }));
  app.use(express.json());
  app.use(cookieParser(process.env.COOKIE_SECRET));
  app.get('/health', (_, res) => res.json({ status: 'ok' }));
  app.use('/v1/auth', authRouter);
  // ...mount all routers under /v1/
  app.use(errorHandler); // must be last
  return app;
}
```

### 5.2 Environment Variables

**`apps/api/src/config/env.ts`** — Typed loader (fail-fast at startup)
```typescript
import { z } from 'zod';

const schema = z.object({
  DATABASE_URL:          z.string().url(),
  JWT_SECRET:            z.string().min(32),
  GOOGLE_CLIENT_ID:      z.string(),
  GOOGLE_CLIENT_SECRET:  z.string(),
  COOKIE_SECRET:         z.string().min(32),
  PORT:                  z.coerce.number().default(3000),
  NODE_ENV:              z.enum(['development', 'test', 'production']).default('development'),
  ALLOWED_ORIGINS:       z.string().default('http://localhost:3001'),
});

export const env = schema.parse(process.env);
```

**Root `.env.example`** — Complete reference (all vars):
```bash
# Database
DATABASE_URL=postgresql://nst:nst@localhost:5432/nstevents

# JWT (≥256-bit random string)
JWT_SECRET=change-me-in-production-min-32-chars

# Google OAuth (from Google Cloud Console)
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_CALLBACK_URL=http://localhost:3000/v1/auth/google/callback

# Cookie signing
COOKIE_SECRET=change-me-in-production-min-32-chars

# Server
PORT=3000
NODE_ENV=development
ALLOWED_ORIGINS=http://localhost:8081,http://localhost:3001

# Worker (separate process)
EXPO_ACCESS_TOKEN=your-expo-access-token
POLL_INTERVAL_MS=5000
BATCH_SIZE=100
```

### 5.3 Middleware Layer — Exact Files

| File | Purpose | Required By |
|---|---|---|
| `middleware/authenticate.ts` | Verify `Authorization: Bearer <jwt>`. Attach `req.user = { id: uuid }`. Return 401 if missing/invalid/expired | Phase 2, all protected endpoints |
| `middleware/authorize.ts` | `requireRole(roles[])` — resolves live `club_memberships` from DB, rejects 403 if not satisfied. `requireClubRole(clubId, roles[])` — checks membership for specific club | Phase 3 RBAC, all mutating endpoints |
| `middleware/validate.ts` | Zod middleware factory: `validate(schema)` → calls `schema.parse(req.body)`, returns 400 with field errors on failure | All POST/PATCH endpoints |
| `middleware/error-handler.ts` | Global Express error handler. Converts `AppError` subclasses to RFC 7807 JSON. Catches unexpected errors → 500 | Phase 0, foundation |
| `middleware/rate-limit.ts` | `express-rate-limit` config objects exported per-route. Auth: 10/min/IP. Attendance: 5/min/user. QR: 10/min/user. Registrations: 10/min/user. Teams: 20/hr/user. Admin: 100/min/user | Phase 10, `docs/api/11-rate-limiting-strategy.md` |

### 5.4 Core Library Utilities — Exact Files

**`apps/api/src/lib/jwt.ts`**
```typescript
import jwt from 'jsonwebtoken';
import { env } from '../config/env';

export function signJwt(userId: string): string {
  return jwt.sign({ sub: userId }, env.JWT_SECRET, { expiresIn: '15m', algorithm: 'HS256' });
}

export function verifyJwt(token: string): { sub: string } {
  return jwt.verify(token, env.JWT_SECRET) as { sub: string };
}
```

**`apps/api/src/lib/db.ts`** — Critical RLS wrapper (see `docs/security/01-rls-architecture.md`)
```typescript
import { prisma } from './prisma';

export async function withUserContext<T>(
  userId: string,
  fn: (tx: typeof prisma) => Promise<T>
): Promise<T> {
  return prisma.$transaction(async (tx) => {
    // SET LOCAL scopes the variable to this transaction only — safe with connection pooling
    await tx.$executeRaw`SELECT set_config('app.user_id', ${userId}, true)`;
    return fn(tx);
  });
}
```

**`apps/api/src/lib/errors.ts`**
```typescript
export class AppError extends Error {
  constructor(public status: number, public code: string, message: string) {
    super(message);
  }
}
export class NotFoundError extends AppError {
  constructor(msg = 'Not found') { super(404, 'NOT_FOUND', msg); }
}
export class ForbiddenError extends AppError {
  constructor(msg = 'Forbidden') { super(403, 'FORBIDDEN', msg); }
}
export class ConflictError extends AppError {
  constructor(msg = 'Conflict') { super(409, 'CONFLICT', msg); }
}
export class UnprocessableError extends AppError {
  constructor(msg: string) { super(422, 'UNPROCESSABLE', msg); }
}
```

**`apps/api/src/types/express.d.ts`** — Request augmentation
```typescript
declare global {
  namespace Express {
    interface Request {
      user?: { id: string };
    }
  }
}
export {};
```

### 5.5 Health Endpoint

`GET /health` — must exist before any feature development. Returns `{ status: 'ok', timestamp: ISO }`. K8s readiness probe hits this endpoint. No auth required.

### 5.6 Logging

The documentation does not mandate a specific logging library. The following is implied by production requirements:
- **Development**: `console.log` / `console.error` is acceptable for Phase 0
- **Production (Phase 12)**: Structured JSON logs. Recommended: `pino` + `pino-http` middleware. Emit `level`, `msg`, `req.id`, `req.method`, `req.url`, `res.statusCode`, `responseTime`
- **Requirement source**: Phase 12 Deliverables (monitoring), K8s liveness/readiness probes

### 5.7 Module Pattern

Every domain module follows this exact three-file pattern (from `docs/backend/02-repository-structure.md`):

| File | Responsibility | Constraint |
|---|---|---|
| `*.router.ts` | Express Router. Applies `authenticate`, `authorize`, `validate` middleware. Delegates to service. Touches `req`/`res`. | Must not contain business logic |
| `*.service.ts` | Business logic. Calls Prisma or invokes RPCs via `$queryRaw`. Returns plain objects. | Must never touch `req`/`res` |
| `*.schema.ts` | Zod schemas for request validation + response typing. Exports inferred TypeScript types. | No side effects |

---

## Section 6 — Database Foundation Requirements

### 6.1 Prisma Schema Location and Role

**Location**: `packages/database/prisma/schema.prisma`

**What Prisma manages** (generated DDL in migration 0001):
- All tables (columns, data types, NOT NULL, DEFAULT values)
- All enums (14 enums — see Section 6.3)
- All relations (FK constraints, cascade rules)
- Basic indexes (`@@index`, `@@unique`)

**What Prisma CANNOT manage** (requires raw SQL migrations):

| Layer | Migration File | Contents |
|---|---|---|
| PostgreSQL extensions | `0002_extensions` | `CREATE EXTENSION "uuid-ossp"`, `pgcrypto`, `postgis`, `pg_cron`; pgmq install |
| Helper function + RLS | `0003_rls_policies` | `current_user_id()` function; all `CREATE POLICY` statements; `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` |
| Triggers | `0004_triggers` | `updated_at` triggers (all tables), audit triggers (events, club_memberships, attendance_records, event_registrations), soft-delete cascade triggers (event→registrations/sessions/teams, club→memberships/event_clubs) |
| Views | `0005_views` | `active_events`, `active_clubs`, `active_memberships` — all with `security_invoker = true`, backed by `INSTEAD OF DELETE` triggers |
| Stored procedures (RPCs) | `0006_rpcs` | All 17+ RPCs (see Section 6.4) |
| Materialized views | `0007_materialized_views` | `club_leaderboard_mv`, `student_leaderboard_mv` with `CREATE UNIQUE INDEX` for `CONCURRENTLY` refresh |
| Full-text search | `0008_search` | Generated `tsvector` columns + GIN indexes on `events`, `clubs`, `users` |
| pg_cron jobs | `0009_pgcron` | MV refresh every 5 min; expired token cleanup daily |
| pgmq queues | `0010_pgmq_queues` | `SELECT pgmq.create('notifications')` |

### 6.2 Migration Workflow (Exact Commands)

```bash
# 1. Define tables/enums/relations in schema.prisma
# 2. Generate base DDL (do not apply yet)
npx prisma migrate dev --create-only --name init

# 3. Append raw SQL to generated migration.sql (or create separate files)
# 4. Apply all migrations
npx prisma migrate dev

# 5. Production apply (no interactive prompt)
npx prisma migrate deploy

# 6. Regenerate TypeScript client after schema changes
npx prisma generate
```

### 6.3 Complete Enum List (Prisma-managed, from `docs/backend/03-prisma-schema-plan.md`)

| Enum Name | Values |
|---|---|
| `GlobalRole` | `STUDENT`, `FACULTY_ADMIN`, `PLATFORM_ADMIN` |
| `ClubRole` | `MEMBER`, `CORE_MEMBER`, `CLUB_ADMIN`, `FACULTY_MENTOR` |
| `ClubStatus` | `ACTIVE`, `INACTIVE`, `DISSOLVED` |
| `EventType` | `WORKSHOP`, `SEMINAR`, `COMPETITION`, `MEETUP`, `HACKATHON`, `OTHER` |
| `EventState` | `DRAFT`, `PENDING_APPROVAL`, `PUBLISHED`, `ARCHIVED` |
| `EventVisibility` | `PUBLIC`, `PRIVATE` |
| `RegistrationType` | `INDIVIDUAL`, `TEAM` |
| `RegistrationStatus` | `REGISTERED`, `WAITLISTED`, `CANCELLED` |
| `AttendanceStatus` | `PRESENT`, `ABSENT`, `EXCUSED` |
| `AttendanceMethod` | `QR`, `MANUAL`, `SYSTEM` |
| `AttendanceType` | `SINGLE`, `MULTI_SESSION` |
| `ParticipationRole` | `ATTENDEE`, `VOLUNTEER`, `ORGANIZER`, `SPEAKER`, `MENTOR` |
| `CompetitionResult` | `WINNER`, `RUNNER_UP`, `SECOND_RUNNER_UP`, `TOP_10`, `PARTICIPANT` |
| `DisputeStatus` | `PENDING`, `APPROVED`, `REJECTED` |
| `HandoverStatus` | `PENDING`, `APPROVED`, `REJECTED` |

### 6.4 Complete Table List (19 tables)

| Table | Soft Delete | Audit Trigger | RLS | Notes |
|---|---|---|---|---|
| `users` | ✅ | ✅ | Own row + public profile | Domain CHECK constraint (SQL) |
| `refresh_tokens` | ❌ (hard delete) | ❌ | None — Express only | pg_cron cleanup |
| `clubs` | ✅ | DELETE only | All auth: SELECT; Platform Admin: UPDATE | `search_vector` GIN |
| `club_memberships` | ✅ | Role changes | All auth: SELECT; Club Admin: INSERT/UPDATE/DELETE | Partial unique index |
| `events` | ✅ | State transitions | PUBLISHED+PUBLIC: all; DRAFT: organizers | `location_geofence GEOGRAPHY` (SQL type) |
| `event_clubs` | ❌ | ❌ | All auth: SELECT; Club Admin: INSERT/DELETE | Composite PK |
| `attendance_sessions` | ✅ | ❌ | Same as parent event | `open_at`/`close_at` CHECK (SQL) |
| `attendance_records` | ❌ | ✅ ALL writes | Self+organizers: SELECT; RPC only: INSERT | `UNIQUE(session_id, user_id)` |
| `event_registrations` | ✅ | ❌ | Self+organizers: SELECT; Self via RPC: INSERT | Composite FK to `teams(id, event_id)` |
| `teams` | ✅ | ❌ | All auth: SELECT; Leader: UPDATE/DELETE | `UNIQUE(event_id, name)` |
| `event_results` | ❌ | ❌ | All auth: SELECT; Club Admin+Faculty: INSERT | `UNIQUE(event_id, user_id)` |
| `notifications` | ❌ (user-delete) | ❌ | Self only | Worker: INSERT only |
| `notification_preferences` | ❌ | ❌ | Self only | One row per user |
| `push_tokens` | ❌ (hard delete) | ❌ | None — Express/worker only | `UNIQUE(device_id)` UPSERT pattern |
| `announcements` | ❌ | ❌ | All auth: SELECT; Club Admin+Platform Admin: INSERT | `club_id` nullable |
| `attendance_disputes` | ❌ | ❌ | Self+organizers: SELECT; Self: INSERT; Club Admin: UPDATE | `UNIQUE(session_id, user_id)` |
| `leadership_handover_requests` | ❌ | ❌ | Participants+Faculty+Platform Admin | Partial unique on `(club_id) WHERE status='PENDING'` |
| `leaderboard_scores` | ❌ (immutable) | ❌ | All auth: SELECT; Express only: INSERT | BRIN index on `created_at` |
| `audit_logs` | ❌ (immutable) | N/A (IS the audit) | Platform Admin+Faculty Admin: SELECT; SECURITY DEFINER triggers: INSERT | `BIGSERIAL` PK (not UUID) |

### 6.5 Complete Stored Procedure (RPC) List

All RPCs are defined in SQL migration `0006_rpcs/migration.sql`.

| RPC | Caller | Key Logic |
|---|---|---|
| `register_event(event_id, user_id)` | Express | Lock-free atomic increment on `events.registration_count` vs `max_capacity`; insert or waitlist |
| `create_team(event_id, user_id, team_name)` | Express | Atomic team + leader registration |
| `join_team(team_id, user_id)` | Express | Check team size from `events.metadata.team_size_max` |
| `leave_team(team_id, user_id)` | Express | Transfer leadership if leader leaves |
| `process_waitlist(event_id)` | Express | Promote first WAITLISTED → REGISTERED; notify |
| `submit_event_for_approval(event_id, user_id)` | Express | DRAFT → PENDING_APPROVAL; enqueue notification to Faculty |
| `approve_event(event_id, actor_id)` | Express | PENDING_APPROVAL → PUBLISHED; enqueue push to registered users |
| `reject_event(event_id, actor_id, reason)` | Express | PENDING_APPROVAL → DRAFT; notify Club Admin |
| `lock_event(event_id)` | Express | Set `is_locked = true` |
| `unlock_event(event_id)` | Express | Set `is_locked = false` |
| `mark_attendance(session_id, user_id, totp_token, lat, lng, device_id, ...)` | Express | Validate TOTP + `ST_DWithin` geofence + device collision; insert `attendance_records`; award points |
| `sync_offline_attendance(records[])` | Express | Batch insert for organizer offline mode; geofence validated; TOTP NOT re-validated |
| `resolve_attendance_dispute(dispute_id, resolution, actor_id)` | Express | `SECURITY DEFINER`; sets `attendance_records.status = 'EXCUSED'` if APPROVED |
| `initiate_leadership_transfer(club_id, initiator_id, successor_id)` | Express | Insert `leadership_handover_requests` |
| `approve_leadership_transfer(request_id, actor_id)` | Express | Swap `CLUB_ADMIN` membership; `FACULTY_ADMIN` fallback allowed |
| `reject_leadership_transfer(request_id, actor_id, notes)` | Express | Set status REJECTED |
| `force_transfer_leadership(club_id, new_admin_id, actor_id)` | Express | `SECURITY DEFINER`; Platform Admin bypass |
| `create_club(name, description, initial_admin_id)` | Express | Atomic club + first `CLUB_ADMIN` membership |

### 6.6 PostGIS Strategy

| Aspect | Detail |
|---|---|
| **Column** | `events.location_geofence GEOGRAPHY(Point, 4326)` — typed via `ALTER COLUMN` in SQL migration. Prisma models it as `Unsupported("geography(Point,4326)")` |
| **Index** | GiST index on `location_geofence` in `0008_search` migration |
| **Validation** | `ST_DWithin(events.location_geofence, ST_SetSRID(ST_MakePoint(lng, lat), 4326), geofence_radius)` inside `mark_attendance` RPC |
| **Prisma access** | Read via `prisma.$queryRaw`; write via `prisma.$executeRaw` with parameterized ST functions |
| **Docker image** | `postgis/postgis:16-3.4` — includes PostGIS extension. pgmq must be compiled separately or use a community image |

### 6.7 pgmq Strategy

| Aspect | Detail |
|---|---|
| **Queue creation** | `SELECT pgmq.create('notifications')` in migration `0010_pgmq_queues` |
| **Enqueue** | RPCs enqueue within the SAME transaction as the triggering action (e.g., `approve_event` → enqueue → commit) |
| **Dequeue** | `nst-worker` polls: `SELECT pgmq.read('notifications', 30, 100)` — 30s visibility timeout, batch 100 |
| **Ack (delete)** | `SELECT pgmq.delete('notifications', msg_id)` after confirmed Expo delivery |
| **DLQ** | After 3 failures: `SELECT pgmq.archive('notifications', msg_id)` → moves to dead letter archive |
| **Autovacuum** | Aggressive autovacuum settings on pgmq schema tables to prevent bloat |

### 6.8 Materialized View Strategy

| Aspect | Detail |
|---|---|
| **Views** | `club_leaderboard_mv`, `student_leaderboard_mv` |
| **Source tables** | Aggregated from `attendance_records`, `event_results`, `event_registrations`, `leaderboard_scores` |
| **Refresh** | `pg_cron` job: `REFRESH MATERIALIZED VIEW CONCURRENTLY club_leaderboard_mv` every 5 minutes |
| **Manual refresh** | `POST /admin/leaderboard/recalculate` (Platform Admin) — triggers immediate CONCURRENT refresh |
| **Unique index** | Required for `CONCURRENTLY` — create in same migration as MVs |
| **Prisma access** | Read via `prisma.$queryRaw` — Prisma cannot model MVs natively |
| **Never** | Row-level triggers for refresh (causes lock contention) |

### 6.9 Seed Strategy

**`packages/database/prisma/seed.ts`** — Development-only seed data. Must create:
- 1 Platform Admin user
- 1 Faculty Admin user
- 3 Faculty Mentor users
- 10 Student users
- 3 Clubs (each with memberships: 1 Club Admin, 2 Core Members, 5 Members)
- 2 Published events (one INDIVIDUAL, one TEAM)
- Sample attendance sessions
- Sample notifications

**Run**: `npx prisma db seed` (configured in `package.json` `prisma.seed` field)

---

## Section 7 — Frontend Foundation Requirements

### 7.1 Mobile App — Where It Lives

**Decision: Move into `apps/mobile/`**

The documentation unambiguously places the mobile application at `apps/mobile/` in the monorepo:
- `docs/backend/02-repository-structure.md`: "apps/mobile/ — Expo React Native app"
- ADR-003: Monorepo Architecture — all four apps coexist in the same repo
- ADR-004: Use Expo React Native

If an existing Expo project exists at the repository root (pre-monorepo), it must be **moved** into `apps/mobile/`. Do not recreate it — preserve existing component work. Update `package.json` name to `@nst/mobile`.

### 7.2 Mobile Navigation Structure

From `docs/mobile/` (ADR-026: Mobile application should be student-first):

```
app/
├── _layout.tsx              # Root: auth gate, token refresh logic
├── (auth)/
│   └── login.tsx            # Google OAuth initiation
└── (tabs)/
    ├── _layout.tsx          # Bottom tab navigator
    ├── index.tsx            # Home feed (get_home_feed delta-sync)
    ├── events.tsx           # Event discovery + search
    ├── clubs.tsx            # Club browsing
    ├── notifications.tsx    # Notification inbox (unread badge)
    └── profile.tsx          # User profile + attendance history
```

Deeper screens (event detail, QR scanner, team management, dispute submission, leaderboard) are pushed on the navigation stack.

### 7.3 Mobile State Management

| Layer | Tool | Scope |
|---|---|---|
| **Server state** | `@tanstack/react-query` | All API data (events, clubs, notifications). Handles caching, refetching, pagination |
| **Auth state** | `zustand` | `{ userId, accessToken, isLoggedIn }` — in-memory only (not persisted) |
| **Optimistic UI** | React Query mutations | Registration, read-notification |

### 7.4 Mobile API Layer

**`apps/mobile/lib/api.ts`** — Typed fetch client
- Base URL: `https://api.nstsdc.org/v1` (from env)
- Attaches `Authorization: Bearer <accessToken>` on every request
- Auto-refreshes token via `POST /auth/refresh` on 401 (interceptor pattern)
- Types imported from `@nst/shared`

---

## Section 8 — Dashboard Requirements

### 8.1 Is a Separate Dashboard Required?

**Yes — explicitly required.** Authority:
- ADR-005: Use Next.js for administration dashboards (Accepted)
- ADR-025: Administration should happen on web dashboards (Accepted)
- ADR-902: Reject Mobile-Based Admin Operations (Rejected ADR)
- MASTER_CONTEXT.md Architecture table: "Web Dashboard | Next.js | Admin, approval, operations"

### 8.2 Framework

**Next.js 14+ with App Router.** From ADR-005: "Next.js provides better out-of-the-box optimizations" over a React SPA (Vite), which was explicitly considered and rejected.

### 8.3 Routing Structure

```
app/
├── layout.tsx               # Root layout (font, theme provider)
├── (auth)/
│   └── login/
│       └── page.tsx         # Google OAuth for dashboard users
└── (shell)/
    ├── layout.tsx           # App shell: sidebar + header + command palette
    ├── page.tsx             # Dashboard home / analytics overview
    ├── events/
    │   ├── page.tsx         # Event list + approval queue
    │   ├── [id]/
    │   │   └── page.tsx     # Event detail + state machine controls
    │   └── new/
    │       └── page.tsx     # Event creation form
    ├── clubs/
    │   ├── page.tsx         # Club list
    │   └── [id]/
    │       └── page.tsx     # Club detail + membership management
    ├── attendance/
    │   ├── page.tsx         # Attendance reports
    │   └── disputes/
    │       └── page.tsx     # Dispute review queue
    ├── leaderboard/
    │   └── page.tsx         # Leaderboard + manual recalculate
    ├── notifications/
    │   └── page.tsx         # Announcement composer
    ├── audit-logs/
    │   └── page.tsx         # Audit log viewer (Platform Admin + Faculty Admin)
    ├── users/
    │   └── page.tsx         # User management (Platform Admin only)
    └── operations/
        └── page.tsx         # Operations mode (ADR-981)
```

### 8.4 Dashboard Dependencies

| Package | Purpose |
|---|---|
| `@nst/shared` | Role types, DTOs, Zod schemas |
| `next` | Framework |
| `@tanstack/react-query` | Server state management |
| `zustand` | Auth state (`{ userId, accessToken }`) |
| `recharts` | Analytics charts (attendance trends, leaderboard history) |
| `@radix-ui/react-*` | Accessible headless UI primitives (dialogs, dropdowns, tables) |
| `cmdk` | Command palette (`Ctrl+K`) — referenced in `docs/dashboard/` as required UX |

### 8.5 Dashboard API Layer

Same pattern as mobile — typed fetch client in `apps/dashboard/lib/api.ts`. Calls `https://api.nstsdc.org/v1` with Bearer JWT. Token stored in memory (not localStorage). Refresh token in HttpOnly cookie (auto-sent by browser).
