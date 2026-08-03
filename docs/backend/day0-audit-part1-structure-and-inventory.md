# NST-Events Day-0 Repository Extraction Audit
# Part 1 of 4 — Structure, Inventory & Tooling

> **Authority**: MASTER_CONTEXT.md, docs/backend/01-implementation-roadmap.md,
> docs/backend/02-repository-structure.md, ADR-001, ADR-003, ADR-004, ADR-005
>
> **Status**: IMPLEMENTATION-READY | **Date**: 2026-06-11

---

## Section 1 — Day-0 Canonical Repository Structure

The exact tree that must exist **before any feature development begins**.
Every path is derived directly from `docs/backend/02-repository-structure.md`
and the K8s/CI requirements in `docs/backend/01-implementation-roadmap.md`.

```
nst-events/
│
├── apps/
│   ├── api/                              # nst-api — Express REST server
│   │   ├── src/
│   │   │   ├── index.ts                  # HTTP server bootstrap + listen()
│   │   │   ├── app.ts                    # Express app factory (middleware chain)
│   │   │   ├── config/
│   │   │   │   └── env.ts                # Typed env-var loader (Zod-parsed)
│   │   │   ├── middleware/
│   │   │   │   ├── authenticate.ts       # JWT verify → req.user
│   │   │   │   ├── authorize.ts          # requireRole() / requireClubRole()
│   │   │   │   ├── rate-limit.ts         # express-rate-limit configuration
│   │   │   │   ├── error-handler.ts      # RFC 7807 global error handler
│   │   │   │   └── validate.ts           # Zod middleware factory
│   │   │   ├── modules/
│   │   │   │   ├── auth/
│   │   │   │   │   ├── auth.router.ts
│   │   │   │   │   ├── auth.service.ts
│   │   │   │   │   └── auth.schema.ts
│   │   │   │   ├── users/
│   │   │   │   │   ├── users.router.ts
│   │   │   │   │   ├── users.service.ts
│   │   │   │   │   └── users.schema.ts
│   │   │   │   ├── clubs/
│   │   │   │   │   ├── clubs.router.ts
│   │   │   │   │   ├── clubs.service.ts
│   │   │   │   │   └── clubs.schema.ts
│   │   │   │   ├── events/
│   │   │   │   │   ├── events.router.ts
│   │   │   │   │   ├── events.service.ts
│   │   │   │   │   └── events.schema.ts
│   │   │   │   ├── registrations/
│   │   │   │   │   ├── registrations.router.ts
│   │   │   │   │   ├── registrations.service.ts
│   │   │   │   │   └── registrations.schema.ts
│   │   │   │   ├── attendance/
│   │   │   │   │   ├── attendance.router.ts
│   │   │   │   │   ├── attendance.service.ts
│   │   │   │   │   ├── attendance.schema.ts
│   │   │   │   │   └── totp.ts           # TOTP generation + 15s window validation
│   │   │   │   ├── notifications/
│   │   │   │   │   ├── notifications.router.ts
│   │   │   │   │   ├── notifications.service.ts
│   │   │   │   │   └── notifications.schema.ts
│   │   │   │   ├── leaderboard/
│   │   │   │   │   ├── leaderboard.router.ts
│   │   │   │   │   ├── leaderboard.service.ts
│   │   │   │   │   └── leaderboard.schema.ts
│   │   │   │   ├── admin/
│   │   │   │   │   ├── admin.router.ts
│   │   │   │   │   ├── admin.service.ts
│   │   │   │   │   └── admin.schema.ts
│   │   │   │   └── sse/
│   │   │   │       ├── sse.router.ts
│   │   │   │       └── sse.service.ts
│   │   │   ├── lib/
│   │   │   │   ├── prisma.ts             # PrismaClient singleton
│   │   │   │   ├── db.ts                 # withUserContext(userId, fn) wrapper
│   │   │   │   ├── jwt.ts                # signJwt() / verifyJwt()
│   │   │   │   └── errors.ts             # AppError, NotFoundError, ForbiddenError
│   │   │   └── types/
│   │   │       └── express.d.ts          # Augment Express.Request with req.user
│   │   ├── tests/
│   │   │   ├── integration/              # Supertest integration tests
│   │   │   ├── unit/                     # Pure logic unit tests
│   │   │   └── helpers/
│   │   │       └── test-db.ts            # migrate + seed + teardown helpers
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── worker/                           # nst-worker — pgmq notification worker
│   │   ├── src/
│   │   │   ├── index.ts                  # Polling loop bootstrap
│   │   │   ├── consumers/
│   │   │   │   └── notification.consumer.ts   # pgmq read → Expo Push API
│   │   │   ├── lib/
│   │   │   │   ├── expo-push.ts          # Expo Push API client
│   │   │   │   └── queue.ts              # pgmq read/delete/archive helpers
│   │   │   └── health.ts                 # HTTP /health for K8s liveness probe
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── mobile/                           # Expo React Native student app
│   │   ├── app/                          # Expo Router file-based navigation
│   │   │   ├── (auth)/
│   │   │   │   └── login.tsx
│   │   │   ├── (tabs)/
│   │   │   │   ├── index.tsx             # Home feed
│   │   │   │   ├── events.tsx
│   │   │   │   ├── clubs.tsx
│   │   │   │   ├── notifications.tsx
│   │   │   │   └── profile.tsx
│   │   │   └── _layout.tsx
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   │   └── api.ts                    # Typed fetch client against nst-api
│   │   ├── package.json
│   │   ├── app.json
│   │   └── tsconfig.json
│   │
│   └── dashboard/                        # Next.js admin dashboard
│       ├── app/                          # Next.js App Router
│       │   ├── (auth)/
│       │   │   └── login/page.tsx
│       │   ├── (shell)/
│       │   │   ├── layout.tsx            # Sidebar shell
│       │   │   ├── events/
│       │   │   ├── clubs/
│       │   │   ├── attendance/
│       │   │   ├── audit-logs/
│       │   │   └── operations/           # Operations mode
│       │   └── layout.tsx
│       ├── components/
│       ├── lib/
│       │   └── api.ts                    # Typed fetch client against nst-api
│       ├── package.json
│       ├── next.config.ts
│       └── tsconfig.json
│
├── packages/
│   ├── database/                         # Prisma schema, migrations, seed
│   │   ├── prisma/
│   │   │   ├── schema.prisma             # Single source of truth for all tables
│   │   │   ├── migrations/
│   │   │   │   ├── 0001_init/
│   │   │   │   │   └── migration.sql     # Prisma-generated base DDL
│   │   │   │   ├── 0002_extensions/
│   │   │   │   │   └── migration.sql     # CREATE EXTENSION uuid-ossp, pgcrypto, postgis, pg_cron
│   │   │   │   ├── 0003_rls_policies/
│   │   │   │   │   └── migration.sql     # current_user_id() fn + all RLS policies
│   │   │   │   ├── 0004_triggers/
│   │   │   │   │   └── migration.sql     # updated_at triggers, audit triggers, soft-delete cascades
│   │   │   │   ├── 0005_views/
│   │   │   │   │   └── migration.sql     # active_events, active_clubs, active_memberships (security_invoker)
│   │   │   │   ├── 0006_rpcs/
│   │   │   │   │   └── migration.sql     # All stored procedures
│   │   │   │   ├── 0007_materialized_views/
│   │   │   │   │   └── migration.sql     # club_leaderboard_mv, student_leaderboard_mv
│   │   │   │   ├── 0008_search/
│   │   │   │   │   └── migration.sql     # Generated tsvector columns + GIN indexes
│   │   │   │   ├── 0009_pgcron/
│   │   │   │   │   └── migration.sql     # MV refresh every 5min, token cleanup
│   │   │   │   └── 0010_pgmq_queues/
│   │   │   │       └── migration.sql     # SELECT pgmq.create('notifications')
│   │   │   └── seed.ts                   # Development seed data
│   │   ├── src/
│   │   │   ├── client.ts                 # PrismaClient singleton export
│   │   │   └── context.ts                # withUserContext(userId, fn) definition
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── shared/                           # Types, constants, validators (all apps)
│   │   ├── src/
│   │   │   ├── types/
│   │   │   │   ├── api.ts                # Request/response DTOs
│   │   │   │   ├── events.ts             # Event-related types
│   │   │   │   ├── roles.ts              # Role enums + type guards
│   │   │   │   └── errors.ts             # RFC 7807 error shape
│   │   │   ├── constants/
│   │   │   │   ├── roles.ts              # Role hierarchy constants
│   │   │   │   ├── limits.ts             # Rate limits, capacity limits
│   │   │   │   └── points.ts             # Leaderboard point values
│   │   │   └── validators/
│   │   │       └── schemas.ts            # Shared Zod schemas
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── config/                           # Shared ESLint, TSConfig, Prettier configs
│       ├── eslint-base.js
│       ├── tsconfig.base.json
│       ├── tsconfig.node.json            # For api + worker
│       ├── tsconfig.react.json           # For dashboard
│       ├── tsconfig.expo.json            # For mobile
│       ├── package.json
│       └── prettier.config.js
│
├── docker/
│   ├── docker-compose.yml                # Local dev: PostgreSQL 16 + pgmq + pg_cron
│   ├── Dockerfile.api                    # Production image for nst-api
│   └── Dockerfile.worker                 # Production image for nst-worker
│
├── k8s/
│   ├── api-deployment.yaml               # nst-api, 2-3 replicas
│   ├── worker-deployment.yaml            # nst-worker, 1 replica
│   ├── postgres-statefulset.yaml         # 2-node CNPG (fits in 8GB RAM)
│   ├── secrets.yaml                      # K8s Secret manifest (gitignored values)
│   └── ingress.yaml                      # Cloudflare Tunnel → *.nstsdc.org
│
├── .github/
│   └── workflows/
│       ├── ci.yml                        # Lint + typecheck + test (all PRs)
│       └── deploy.yml                    # Build + push + rollout to NST Cluster
│
├── turbo.json                            # Turborepo pipeline definitions
├── package.json                          # Root pnpm workspace config
├── pnpm-workspace.yaml                   # Workspace glob patterns
├── .env.example                          # All required env vars documented
├── .env                                  # Local secrets (gitignored)
├── .eslintrc.js                          # Root ESLint config (extends packages/config)
├── .prettierrc.js                        # Root Prettier config
├── .gitignore
└── docs/                                 # Architecture documentation (this repo)
```

### Dependency Graph (Hard Rules)

```
packages/config     ──→  (no dependencies)
packages/shared     ──→  zod only
packages/database   ──→  prisma, @prisma/client only
apps/api            ──→  @nst/database, @nst/shared
apps/worker         ──→  @nst/database, @nst/shared
apps/mobile         ──→  @nst/shared only
apps/dashboard      ──→  @nst/shared only
```

> **Hard Rule**: No circular dependencies. No app package imports another app
> package. `packages/database` is backend-only. `packages/shared` is consumed
> by all apps.

---

## Section 2 — Application Inventory

### 2.1 `apps/api` — nst-api

| Field | Value |
|---|---|
| **Purpose** | Single HTTP entry point for all clients. Handles auth, RBAC, route handling, Prisma queries, SSE streaming, RPC invocation |
| **Runtime** | Node.js 20 LTS |
| **Framework** | Express 4.x + TypeScript 5.x strict |
| **Build command** | `pnpm --filter @nst/api build` → `tsc -p tsconfig.json` |
| **Dev command** | `pnpm --filter @nst/api dev` → `ts-node-dev --respawn src/index.ts` |
| **Deployment target** | K3s Deployment `nst-api`, **2–3 replicas**, NST Cluster |
| **Exposed port** | `3000` (internal); routed via Cloudflare Tunnel to `api.nstsdc.org` |
| **Dependencies** | `@nst/database`, `@nst/shared`, `express`, `jsonwebtoken`, `otplib` (TOTP), `express-rate-limit`, `helmet`, `cors`, `zod`, `passport`, `passport-google-oauth20` |
| **Key env vars** | `DATABASE_URL`, `JWT_SECRET`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `COOKIE_SECRET`, `PORT`, `NODE_ENV`, `ALLOWED_ORIGINS` |

---

### 2.2 `apps/worker` — nst-worker

| Field | Value |
|---|---|
| **Purpose** | Dedicated pgmq consumer. Polls `notifications` queue every 5 s (batch 100). Dispatches push notifications via Expo Push API. Does NOT serve HTTP traffic except `/health` |
| **Runtime** | Node.js 20 LTS |
| **Framework** | Plain TypeScript — no HTTP framework |
| **Build command** | `pnpm --filter @nst/worker build` → `tsc -p tsconfig.json` |
| **Dev command** | `pnpm --filter @nst/worker dev` → `ts-node-dev src/index.ts` |
| **Deployment target** | K3s Deployment `nst-worker`, **1 replica**, NST Cluster |
| **Health check** | `GET /health` on port `3001` — K8s liveness probe only |
| **Dependencies** | `@nst/database`, `@nst/shared`, `expo-server-sdk`, `node-fetch` |
| **Key env vars** | `DATABASE_URL`, `EXPO_ACCESS_TOKEN`, `POLL_INTERVAL_MS` (default 5000), `BATCH_SIZE` (default 100) |
| **Replica constraint** | Intentionally 1. pgmq visibility timeout prevents duplicate processing |

---

### 2.3 `apps/mobile` — Expo React Native

| Field | Value |
|---|---|
| **Purpose** | Student-first mobile app. Events, attendance, notifications, leaderboard, QR scanning |
| **Runtime** | Expo SDK 51+ (React Native 0.74+) |
| **Framework** | Expo Router (file-based navigation) |
| **Build command** | `expo build` / EAS Build for App Store / Play Store |
| **Dev command** | `pnpm --filter @nst/mobile start` → `expo start` |
| **Deployment target** | iOS App Store, Google Play (via EAS) |
| **Dependencies** | `@nst/shared`, `expo-router`, `expo-camera` (QR), `expo-notifications`, `expo-location`, `@tanstack/react-query`, `zustand`, `nativewind` |
| **API surface** | Calls `https://api.nstsdc.org/v1` exclusively. Never imports backend packages |

---

### 2.4 `apps/dashboard` — Next.js Admin Dashboard

| Field | Value |
|---|---|
| **Purpose** | Web admin surface for Club Admins, Faculty, Platform Admins. Event management, approval workflows, attendance review, audit logs, operations mode |
| **Runtime** | Node.js 20 LTS (Next.js server) |
| **Framework** | Next.js 14+ (App Router) |
| **Build command** | `pnpm --filter @nst/dashboard build` → `next build` |
| **Dev command** | `pnpm --filter @nst/dashboard dev` → `next dev` |
| **Deployment target** | K3s Deployment or static export; routed to `dashboard.nstsdc.org` |
| **Dependencies** | `@nst/shared`, `next`, `react`, `@tanstack/react-query`, `zustand`, `recharts` (analytics), `@radix-ui/react-*` (headless UI) |
| **API surface** | Calls `https://api.nstsdc.org/v1` exclusively. Never imports backend packages |

---

## Section 3 — Package Inventory

### 3.1 `packages/database` (`@nst/database`)

| Field | Value |
|---|---|
| **Purpose** | Single source of truth for all database access. Prisma schema, all migrations, seed script, `withUserContext` utility |
| **Contents** | `prisma/schema.prisma`, 10 SQL migration files, `seed.ts`, `src/client.ts`, `src/context.ts` |
| **Consumers** | `apps/api`, `apps/worker` |
| **Forbidden consumers** | `apps/mobile`, `apps/dashboard` — database package must never be imported by frontend |
| **Runtime deps** | `@prisma/client` |
| **Dev deps** | `prisma` (CLI) |
| **Key exports** | `prisma` (singleton), `withUserContext(userId, fn)` |

---

### 3.2 `packages/shared` (`@nst/shared`)

| Field | Value |
|---|---|
| **Purpose** | Shared TypeScript types, constants, Zod schemas across all four apps |
| **Contents** | `types/api.ts`, `types/events.ts`, `types/roles.ts`, `types/errors.ts`, `constants/roles.ts`, `constants/limits.ts`, `constants/points.ts`, `validators/schemas.ts` |
| **Consumers** | `apps/api`, `apps/worker`, `apps/mobile`, `apps/dashboard` |
| **Runtime deps** | `zod` only |
| **Forbidden deps** | `prisma`, `@prisma/client`, any app package |
| **Key exports** | Role enums, RFC 7807 error type, Zod schemas for request validation, rate limit constants, leaderboard point values |

---

### 3.3 `packages/config` (`@nst/config`)

| Field | Value |
|---|---|
| **Purpose** | Eliminates duplicated tooling config. Single source for ESLint rules, TSConfig bases, Prettier config |
| **Contents** | `eslint-base.js`, `tsconfig.base.json`, `tsconfig.node.json`, `tsconfig.react.json`, `tsconfig.expo.json`, `prettier.config.js` |
| **Consumers** | All four apps, both backend packages |
| **Runtime deps** | None |
| **Dev deps** | ESLint plugins (`@typescript-eslint`, `eslint-plugin-import`), Prettier plugins |

---

## Section 4 — Root Tooling Requirements

### 4.1 pnpm Workspaces

| Field | Value |
|---|---|
| **Why** | Package manager for the monorepo. Hoists shared deps, manages workspace links (`@nst/*`), symlinks packages locally for zero-publish development |
| **Config file** | `pnpm-workspace.yaml`, root `package.json` `"workspaces"` field |
| **Required by** | ADR-003 (Monorepo Architecture), `docs/backend/02-repository-structure.md` |
| **Version** | pnpm 9.x |
| **Install** | `npm install -g pnpm@9` |

### 4.2 Turborepo

| Field | Value |
|---|---|
| **Why** | Orchestrates build/lint/test pipelines across all workspace packages. Caches outputs so only changed packages rebuild. Required for the 6-developer team velocity |
| **Config file** | `turbo.json` at repo root |
| **Required by** | `docs/backend/01-implementation-roadmap.md` Phase 0 Deliverables, `docs/backend/02-repository-structure.md` |
| **Pipelines needed** | `build`, `dev`, `lint`, `typecheck`, `test` |
| **Risk** | Turborepo workspace config may conflict with Prisma generated client paths — documented in roadmap §Phase 0 Risks |

### 4.3 TypeScript (strict mode)

| Field | Value |
|---|---|
| **Why** | Type safety across entire stack. Prisma generates types from `schema.prisma`; `packages/shared` exports them. Strict mode eliminates `any` holes |
| **Config** | `packages/config/tsconfig.base.json` with `"strict": true`, `"noUncheckedIndexedAccess": true` |
| **Required by** | ADR-001, Phase 0 Definition of Done: "TS compiles zero errors" |
| **Version** | TypeScript 5.4+ |

### 4.4 Prisma ORM

| Field | Value |
|---|---|
| **Why** | Type-safe database access. Generates TypeScript types from `schema.prisma`. Manages table/enum/relation migrations |
| **Config file** | `packages/database/prisma/schema.prisma` |
| **Required by** | ADR-001, MASTER_CONTEXT.md Architecture table, all database docs |
| **Version** | Prisma 5.x |
| **CLI usage** | `npx prisma migrate dev`, `npx prisma migrate deploy`, `npx prisma generate`, `npx prisma db seed` |
| **Limitation** | Cannot manage: RLS policies, triggers, views, materialized views, stored procedures, pg_cron, pgmq — all require raw SQL migrations |

### 4.5 ESLint

| Field | Value |
|---|---|
| **Why** | Code quality enforcement. Prevents import rule violations (no `apps/api` importing `@nst/mobile`), enforces TypeScript best practices |
| **Config** | Root `.eslintrc.js` extending `packages/config/eslint-base.js` |
| **Required by** | Phase 0 Deliverables, CI pipeline (`ci.yml`) |
| **Key rules** | `@typescript-eslint/no-explicit-any`, `import/no-restricted-paths` (enforce dep graph), `no-unused-vars` |

### 4.6 Prettier

| Field | Value |
|---|---|
| **Why** | Uniform formatting across TypeScript, JSON, Markdown. Prevents style debates in PRs |
| **Config** | `packages/config/prettier.config.js`, root `.prettierrc.js` |
| **Required by** | Phase 0 Deliverables |

### 4.7 Husky + lint-staged

| Field | Value |
|---|---|
| **Why** | Pre-commit hooks run `lint-staged` to ensure no unformatted or type-error code is committed. Prevents broken CI |
| **Config** | `.husky/pre-commit`, `.lintstagedrc.json` |
| **Required by** | Phase 0 Deliverables (implied by CI pipeline requirement) |
| **Commands** | `husky install` in root `prepare` npm script |

### 4.8 Docker Compose

| Field | Value |
|---|---|
| **Why** | Local development environment. Starts PostgreSQL 16 with PostGIS + pg_cron + pgmq in a single command |
| **Config file** | `docker/docker-compose.yml` |
| **Required by** | Phase 0 Prerequisites, Phase 0 Definition of Done: "`docker compose up` starts PG+pgmq" |
| **Services** | `postgres` (PostgreSQL 16 + PostGIS + pgmq), `pgadmin` (optional, dev only) |
| **Image** | `ghcr.io/pramsey/pg_featureserv` or `postgis/postgis:16-3.4` with pgmq extension added |

### 4.9 GitHub Actions

| Field | Value |
|---|---|
| **Why** | Automated CI on every PR. Automated deploy on merge to `main` |
| **Config files** | `.github/workflows/ci.yml`, `.github/workflows/deploy.yml` |
| **Required by** | Phase 0 Deliverables, Phase 12 Deliverables |
| **CI steps** | `pnpm install` → `turbo lint` → `turbo typecheck` → `turbo test` |
| **Deploy steps** | `docker build` → `docker push ghcr.io/nst/*` → `kubectl rollout restart` |

### 4.10 Vitest (Testing)

| Field | Value |
|---|---|
| **Why** | Fast unit and integration testing for TypeScript. Compatible with Turborepo pipeline. Supertest for HTTP integration tests |
| **Required by** | Phase 11 Deliverables: "Unit: JWT, TOTP, role resolution, Zod schemas; Integration: OAuth flow, registration capacity locking…" |
| **Frameworks** | `vitest` (unit), `supertest` (HTTP integration), `@testcontainers/postgresql` (isolated DB) |
| **Note** | The documentation specifies "Vitest/Jest" — Vitest is recommended for speed and ESM compatibility |

### 4.11 Node.js 20 LTS

| Field | Value |
|---|---|
| **Why** | LTS release with native `crypto` module (SHA-256 for refresh tokens), stable `fetch` API, `EventEmitter` for SSE |
| **Required by** | Phase 0 Prerequisites |
| **Pin method** | `.nvmrc` file at repo root containing `20` |
