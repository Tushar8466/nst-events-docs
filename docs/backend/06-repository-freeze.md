# NST-Events Repository Freeze

> **Status**: FROZEN | **Version**: 1.0 | **Date**: 2026-06-11
> **Authority**: MASTER_CONTEXT.md, docs/backend/*, docs/database/*, docs/security/*, docs/api/*, All Accepted ADRs

---

## SECTION 1 — Repository Tree

```
nst-events/
├── apps/
│   ├── api/
│   ├── worker/
│   ├── mobile/
│   └── dashboard/
├── packages/
│   ├── database/
│   ├── shared/
│   └── config/
├── docker/
├── k8s/
├── .github/
│   └── workflows/
├── package.json
├── pnpm-workspace.yaml
├── turbo.json
├── .env.example
├── .gitignore
├── .nvmrc
├── .eslintrc.js
└── .prettierrc.js
```

---

## SECTION 2 — Package Ownership

### `apps/api`
*   **Purpose**: Single HTTP entry point for all clients. Handles auth, RBAC, route handling, SSE streaming, RPC invocation via `$queryRaw`, and error handling.
*   **Owner**: Backend Team
*   **Dependencies**: `@nst/database`, `@nst/shared`, all `node_modules` (express, zod, passport, etc.).
*   **Public API**: HTTP endpoints at `https://api.nstsdc.org/v1`.
*   **Internal Files**: `src/index.ts`, `src/app.ts`, `src/config/`, `src/middleware/`, `src/modules/`, `src/lib/`, `src/types/`, `tests/`.
*   **Import Rules**: MUST NOT import `@nst/mobile`, `@nst/dashboard`, or `@nst/worker`.

### `apps/worker`
*   **Purpose**: Dedicated native queue consumer. Polls `notifications` queue every 5s. Delivers Expo push notifications. Does NOT handle HTTP traffic (except `/health` for K8s).
*   **Owner**: Backend Team
*   **Dependencies**: `@nst/database`, `@nst/shared`, `expo-server-sdk`, `node-fetch`.
*   **Public API**: None (Internal process polling PostgreSQL).
*   **Internal Files**: `src/index.ts`, `src/health.ts`, `src/consumers/`, `src/lib/queue.ts`, `src/lib/expo-push.ts`.
*   **Import Rules**: MUST NOT import `@nst/api`, `@nst/mobile`, or `@nst/dashboard`.

### `apps/mobile`
*   **Purpose**: Student-first Expo React Native app.
*   **Owner**: Frontend (Mobile) Team
*   **Dependencies**: `@nst/shared`, Expo SDK, React Native, `@tanstack/react-query`, `zustand`, `nativewind`.
*   **Public API**: None. Consumes API HTTP endpoints.
*   **Internal Files**: `app/` (Expo Router), `components/`, `hooks/`, `store/`, `lib/api.ts`, `theme/`, `assets/`.
*   **Import Rules**: MUST NOT import `@nst/database`, `@nst/api`, or `@nst/worker`.

### `apps/dashboard`
*   **Purpose**: Next.js 14 App Router admin dashboard for Club Admins, Faculty, and Platform Admins.
*   **Owner**: Frontend (Web) Team
*   **Dependencies**: `@nst/shared`, `next`, `react`, `@tanstack/react-query`, `zustand`, `@radix-ui/react-*`, `recharts`, `cmdk`.
*   **Public API**: None. Consumes API HTTP endpoints.
*   **Internal Files**: `app/` (Next Router), `components/`, `hooks/`, `store/`, `lib/api.ts`.
*   **Import Rules**: MUST NOT import `@nst/database`, `@nst/api`, or `@nst/worker`.

### `packages/database`
*   **Purpose**: Single source of truth for the database schema, all migrations, the `withUserContext` RLS wrapper, and the Prisma client singleton.
*   **Owner**: Backend Team (Senior Review Required)
*   **Dependencies**: `prisma`, `@prisma/client`.
*   **Public API**: `prisma` singleton, `withUserContext()`.
*   **Internal Files**: `prisma/schema.prisma`, `prisma/migrations/`, `prisma/seed.ts`, `src/client.ts`, `src/context.ts`.
*   **Import Rules**: MUST NOT import from any app or other package.

### `packages/shared`
*   **Purpose**: Shared TypeScript types, constants, Zod schemas, and error definitions.
*   **Owner**: Backend Team Leads
*   **Dependencies**: `zod`.
*   **Public API**: API DTOs, Event types, Role enums, validation schemas.
*   **Internal Files**: `src/types/`, `src/constants/`, `src/validators/`.
*   **Import Rules**: MUST NOT import `prisma`, `@prisma/client`, or any app package.

### `packages/config`
*   **Purpose**: Eliminates duplicated tooling configuration. ESLint rules, TypeScript compiler options, Prettier settings.
*   **Owner**: Tech Lead
*   **Dependencies**: ESLint plugins, Prettier plugins.
*   **Public API**: `eslint-base.js`, `tsconfig.base.json`, `tsconfig.node.json`, `tsconfig.react.json`, `tsconfig.expo.json`, `prettier.config.js`.
*   **Internal Files**: Config files.
*   **Import Rules**: MUST NOT import any app or library packages.

---

## SECTION 3 — Root Configuration Files

### `package.json`
*   **Purpose**: Root workspace config, dependencies, and turborepo scripts.
*   **Contents**: `private: true`, `scripts: { dev, build, lint, typecheck, test, prepare }`, Husky hook setup.

### `pnpm-workspace.yaml`
*   **Purpose**: Defines the pnpm workspace monorepo boundaries.
*   **Contents**: `packages: ['apps/*', 'packages/*']`

### `turbo.json`
*   **Purpose**: Orchestrates build/lint/test pipelines across all workspace packages with caching.
*   **Contents**: Pipeline definitions for `build`, `dev`, `lint`, `typecheck`, `test` with inputs/outputs/dependencies.

### `.env.example`
*   **Purpose**: Document all required environment variables for the system.
*   **Contents**: `DATABASE_URL`, `JWT_SECRET`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `COOKIE_SECRET`, `PORT`, `NODE_ENV`, `ALLOWED_ORIGINS`, `EXPO_ACCESS_TOKEN`, `POLL_INTERVAL_MS`, `BATCH_SIZE`.

### `.gitignore`
*   **Purpose**: Prevent sensitive data and build artifacts from entering version control.
*   **Contents**: `node_modules`, `dist`, `.env`, `*.js.map`, `.turbo`.

### `.nvmrc`
*   **Purpose**: Pin Node.js version.
*   **Contents**: `20`

### `.eslintrc.js`
*   **Purpose**: Root ESLint configuration.
*   **Contents**: Extends `packages/config/eslint-base.js`.

### `.prettierrc.js`
*   **Purpose**: Root Prettier configuration.
*   **Contents**: Extends `packages/config/prettier.config.js`.

---

## SECTION 4 — API Application Structure

```
apps/api/
├── src/
│   ├── config/
│   │   └── env.ts
│   ├── middleware/
│   │   ├── authenticate.ts
│   │   ├── authorize.ts
│   │   ├── error-handler.ts
│   │   ├── rate-limit.ts
│   │   └── validate.ts
│   ├── modules/
│   │   ├── auth/
│   │   ├── users/
│   │   ├── clubs/
│   │   ├── events/
│   │   ├── registrations/
│   │   ├── attendance/
│   │   ├── notifications/
│   │   ├── leaderboard/
│   │   ├── admin/
│   │   └── sse/
│   ├── lib/
│   │   ├── prisma.ts
│   │   ├── db.ts
│   │   ├── jwt.ts
│   │   └── errors.ts
│   ├── types/
│   │   └── express.d.ts
│   ├── index.ts
│   └── app.ts
└── tests/
    ├── integration/
    ├── unit/
    └── helpers/
```

### Purpose, Ownership, & Allowed Dependencies

*   **`config/`**:
    *   *Purpose*: Zod-validated environment variable loader. Fails fast on missing variables.
    *   *Ownership*: Backend.
    *   *Dependencies*: `zod`.
*   **`middleware/`**:
    *   *Purpose*: Reusable Express middleware (Auth, RBAC, error handling, rate limiting).
    *   *Ownership*: Senior Backend.
    *   *Dependencies*: `express`, `express-rate-limit`, `zod`, `jsonwebtoken`.
*   **`modules/`**:
    *   *Purpose*: Feature domains. Each contains `*.router.ts`, `*.service.ts`, `*.schema.ts`. Routers handle HTTP, services handle business logic, schemas handle validation.
    *   *Ownership*: Backend / Senior Backend.
    *   *Dependencies*: `@nst/database`, `@nst/shared`, `express`.
*   **`lib/`**:
    *   *Purpose*: Core utilities (`withUserContext` RLS wrapper, JWT sign/verify, Custom Error classes).
    *   *Ownership*: Senior Backend.
    *   *Dependencies*: `@nst/database`, `jsonwebtoken`.
*   **`types/`**:
    *   *Purpose*: TypeScript declarations (`req.user` augmentation).
    *   *Ownership*: Backend.
    *   *Dependencies*: `express`.
*   **`tests/`**:
    *   *Purpose*: Supertest integration tests and unit tests.
    *   *Ownership*: Backend.
    *   *Dependencies*: `vitest`, `supertest`.


---

## SECTION 5 — Worker Structure

```
apps/worker/
├── src/
│   ├── consumers/
│   │   └── notification.consumer.ts
│   ├── lib/
│   │   ├── expo-push.ts
│   │   └── queue.ts
│   ├── index.ts
│   └── health.ts
├── package.json
└── tsconfig.json
```

### Components
*   **`consumers/`**: Logic to poll `native queue`, process messages, handle retries, and write back delivery status.
*   **`lib/queue.ts`**: Helper functions to execute `native queue.read`, `native queue.delete`, and `native queue.archive` (DLQ) via `prisma.$queryRaw`.
*   **`lib/expo-push.ts`**: Expo Push API integration, including handling `DeviceNotRegistered` errors to hard-delete `push_tokens` rows.
*   **`health.ts`**: Minimal HTTP server (e.g., Express) on port 3001 serving `GET /health` for K8s liveness probes.
*   **`index.ts`**: Worker bootstrap loop initializing `setInterval` with `POLL_INTERVAL_MS`.

---

## SECTION 6 — Mobile Structure

```
apps/mobile/
├── app/
│   ├── _layout.tsx
│   ├── (auth)/
│   │   ├── _layout.tsx
│   │   └── login.tsx
│   └── (tabs)/
│       ├── _layout.tsx
│       ├── index.tsx
│       ├── events.tsx
│       ├── clubs.tsx
│       ├── notifications.tsx
│       └── profile.tsx
├── components/
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   ├── Badge.tsx
│   │   └── LoadingSpinner.tsx
│   ├── events/
│   │   ├── EventCard.tsx
│   │   └── EventStateChip.tsx
│   ├── attendance/
│   │   └── QRScanner.tsx
│   └── notifications/
│       └── NotificationItem.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useEvents.ts
│   ├── useClubs.ts
│   └── useNotifications.ts
├── store/
│   └── auth.store.ts
├── lib/
│   ├── api.ts
│   └── token.ts
├── theme/
│   └── index.ts
├── assets/
│   ├── fonts/
│   └── images/
├── app.json
├── package.json
└── tsconfig.json
```

### Components
*   **`app/`**: Expo Router file-based navigation layout.
*   **`components/`**: Reusable NativeWind UI components, domain-specific components (Events, QR Scanner).
*   **`hooks/`**: React Query wrappers for data fetching and caching.
*   **`store/`**: Zustand auth state (`userId`, `accessToken`).
*   **`lib/api.ts`**: Typed fetch client targeting `api.nstsdc.org` with Bearer token injection and auto-refresh interceptors.
*   **`theme/`**: Design tokens and styling constants.
*   **`assets/`**: Local fonts and static images.

---

## SECTION 7 — Dashboard Structure

```
apps/dashboard/
├── app/
│   ├── layout.tsx
│   ├── (auth)/
│   │   └── login/
│   │       └── page.tsx
│   └── (shell)/
│       ├── layout.tsx
│       ├── page.tsx
│       ├── events/
│       │   ├── page.tsx
│       │   ├── new/
│       │   │   └── page.tsx
│       │   └── [id]/
│       │       └── page.tsx
│       ├── clubs/
│       │   ├── page.tsx
│       │   └── [id]/
│       │       └── page.tsx
│       ├── attendance/
│       │   ├── page.tsx
│       │   └── disputes/
│       │       └── page.tsx
│       ├── leaderboard/
│       │   └── page.tsx
│       ├── notifications/
│       │   └── page.tsx
│       ├── audit-logs/
│       │   └── page.tsx
│       ├── users/
│       │   └── page.tsx
│       └── operations/
│           └── page.tsx
├── components/
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── DataTable.tsx
│   │   ├── Dialog.tsx
│   │   ├── Badge.tsx
│   │   └── StatCard.tsx
│   ├── shell/
│   │   ├── Sidebar.tsx
│   │   ├── Header.tsx
│   │   └── CommandPalette.tsx
│   ├── events/
│   │   ├── EventForm.tsx
│   │   ├── ApprovalActions.tsx
│   │   └── StateMachineView.tsx
│   └── attendance/
│       └── DisputeReviewCard.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useEvents.ts
│   └── useClubs.ts
├── store/
│   └── auth.store.ts
├── lib/
│   └── api.ts
├── next.config.ts
├── package.json
└── tsconfig.json
```

### Components
*   **`app/`**: Next.js App Router. `(shell)` contains the authenticated layout with a sidebar.
*   **`components/ui/`**: Radix UI headless primitives mapped to design system.
*   **`components/shell/`**: Navigation sidebar, command palette (`cmdk`).
*   **`components/events/` & `attendance/`**: Complex forms and data tables.
*   **`hooks/` & `store/` & `lib/api.ts`**: Similar to mobile, specialized for web auth (HttpOnly cookie handling).

---

## SECTION 8 — Database Package

```
packages/database/
├── prisma/
│   ├── schema.prisma
│   ├── seed.ts
│   └── migrations/
│       ├── 0001_init/
│       ├── 0002_extensions/
│       ├── 0003_rls_policies/
│       ├── 0004_triggers/
│       ├── 0005_views/
│       ├── 0006_rpcs/
│       ├── 0007_materialized_views/
│       ├── 0008_search/
│       ├── 0009_pgcron/
│       └── 0010_native queue_queues/
├── src/
│   ├── client.ts
│   └── context.ts
├── package.json
└── tsconfig.json
```

### Components
*   **`schema.prisma`**: Single source of truth for tables, enums, and basic relations.
*   **`migrations/`**: Raw SQL logic bridging Prisma limitations. Required for PostGIS, native queue, triggers, RLS, RPCs, MVs, FTS, and pg_cron.
*   **`seed.ts`**: Development data (Admin, clubs, events, users).
*   **`src/context.ts`**: The `withUserContext(userId, fn)` transaction wrapper required to inject `app.user_id` for RLS.
*   **`src/client.ts`**: PrismaClient singleton.

---

## SECTION 9 — Shared Package

```
packages/shared/
├── src/
│   ├── types/
│   │   ├── api.ts
│   │   ├── events.ts
│   │   ├── roles.ts
│   │   └── errors.ts
│   ├── constants/
│   │   ├── roles.ts
│   │   ├── limits.ts
│   │   └── points.ts
│   └── validators/
│       └── schemas.ts
├── package.json
└── tsconfig.json
```

### Components
*   **`types/api.ts`**: API request and response DTO definitions.
*   **`types/roles.ts`**: Role enums and type guards.
*   **`types/errors.ts`**: RFC 7807 error shape definition.
*   **`constants/limits.ts`**: Rate limiting and capacity limits constants.
*   **`constants/points.ts`**: Leaderboard reward point mappings.
*   **`validators/schemas.ts`**: Shared Zod schemas for input validation.

---

## SECTION 10 — Infrastructure

```
nst-events/
├── docker/
│   ├── docker-compose.yml
│   ├── Dockerfile.api
│   └── Dockerfile.worker
├── k8s/
│   ├── api-deployment.yaml
│   ├── worker-deployment.yaml
│   ├── postgres-statefulset.yaml
│   ├── secrets.yaml
│   └── ingress.yaml
└── .github/
    └── workflows/
        ├── ci.yml
        └── deploy.yml
```

### Components
*   **`docker-compose.yml`**: Local dev environment running PostgreSQL 16, PostGIS, native queue, and pg_cron.
*   **`Dockerfile.api` / `Dockerfile.worker`**: Production multi-stage Docker builds.
*   **`postgres-statefulset.yaml`**: 2-node CNPG PostgreSQL deployment suitable for the 8GB RAM NST cluster nodes.
*   **`k8s/*.yaml`**: Day-0 manifests for NST Cluster deployment.
*   **`ci.yml`**: GitHub Actions testing pipeline (install -> lint -> typecheck -> test).
*   **`deploy.yml`**: CD pipeline (build docker -> push ghcr.io -> kubectl rollout restart -> prisma migrate deploy).

---

## SECTION 11 — Day-0 Bootstrap

**Exact Order of Operations:**

1.  **Folder Creation:**
    *   `mkdir -p apps/{api,worker,mobile,dashboard}`
    *   `mkdir -p packages/{database,shared,config}`
    *   `mkdir -p docker k8s .github/workflows`
2.  **Root Tooling Setup:**
    *   Create `package.json`, `pnpm-workspace.yaml`, `turbo.json`.
    *   Create `.env.example`, `.nvmrc` (v20), `.gitignore`.
3.  **Config Package Creation:**
    *   Initialize `packages/config/` with `eslint-base.js`, `prettier.config.js`, `tsconfig.*.json`.
    *   Link from root `.eslintrc.js` and `.prettierrc.js`.
4.  **Shared & Database Package Setup:**
    *   Initialize `packages/shared/` with `zod`.
    *   Initialize `packages/database/` with `prisma` init.
5.  **Application Scaffolding:**
    *   Scaffold `apps/api/` (Express + TS config).
    *   Scaffold `apps/worker/` (TS config).
    *   Scaffold `apps/mobile/` (Expo create, move to dir).
    *   Scaffold `apps/dashboard/` (Next.js create, move to dir).
6.  **Dependency Installation:**
    *   Link `@nst/shared` and `@nst/database` into `apps/api` and `apps/worker`.
    *   Link `@nst/shared` into `apps/mobile` and `apps/dashboard`.
    *   Run `pnpm install` at root.
7.  **Infrastructure Initialization:**
    *   Create `docker-compose.yml`.
    *   Add GH Actions workflows.

---

## SECTION 12 — Final Freeze

### Repository Freeze Checklist

All items **MUST** exist before Phase 1 (Database Foundation) begins.

*   [x] **[REQUIRED]** `pnpm-workspace.yaml` and `turbo.json` correctly linked.
*   [x] **[REQUIRED]** `packages/config` containing base ESLint/Prettier/TSConfigs.
*   [x] **[REQUIRED]** `packages/shared` exporting basic Zod schemas and constants.
*   [x] **[REQUIRED]** `packages/database` containing `schema.prisma`.
*   [x] **[REQUIRED]** `apps/api` containing foundational middleware (`authenticate`, `error-handler`, `validate`), `app.ts`, and `index.ts`.
*   [x] **[REQUIRED]** `apps/worker` containing polling loop structure and `/health` endpoint.
*   [x] **[REQUIRED]** `apps/mobile` Expo project scaffolded with React Query and Zustand.
*   [x] **[REQUIRED]** `apps/dashboard` Next.js project scaffolded with Radix UI and React Query.
*   [x] **[REQUIRED]** `docker-compose.yml` running PostgreSQL with PostGIS + native queue.
*   [x] **[REQUIRED]** `.env.example` documenting all configuration keys.
*   [x] **[REQUIRED]** `.github/workflows/ci.yml` testing pipeline.
*   [ ] **[FUTURE]** Media storage integrations (e.g., S3 setup for image uploads) are explicitly deferred to post-V1.
*   [ ] **[OPTIONAL]** Local pgAdmin or DB GUI inside docker-compose.