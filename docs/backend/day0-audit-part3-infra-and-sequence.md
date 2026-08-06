# NST-Events Day-0 Repository Extraction Audit
# Part 3 of 4 — Infrastructure, Development Sequence & Gap Analysis

> Continues from Part 2. Authority: docs/backend/01-implementation-roadmap.md,
> docs/backend/05-development-order.md, docs/api/13-worker-deployment.md,
> adrs/README.md, MASTER_CONTEXT.md

---

## Section 9 — Infrastructure Requirements

### 9.1 Docker Compose (Local Development)

**File**: `docker/docker-compose.yml`

Required services before any code can run:

```yaml
version: '3.9'

services:
  postgres:
    image: ghcr.io/tembo-io/tembo-local:latest   # PostgreSQL 16 + PostGIS + pgmq + pg_cron
    # Alternative: build a custom image from postgis/postgis:16-3.4 + install pgmq
    environment:
      POSTGRES_USER: nst
      POSTGRES_PASSWORD: nst
      POSTGRES_DB: nstevents
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U nst -d nstevents"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

> **Risk (from roadmap §Phase 0)**: pgmq Docker image availability varies. If a
> pre-built image with pgmq is not available, build a custom image:
> `FROM postgis/postgis:16-3.4` + compile pgmq from source. This must be
> resolved before Phase 1 can begin.

**Phase 0 DoD**: `docker compose up -d` → PostgreSQL accepting connections on
`localhost:5432`. `pg_isready` returns healthy.

---

### 9.2 Kubernetes Manifests

All manifests live in `k8s/`. These are Day-0 stubs; final values require
secrets and image tags before Phase 12.

#### `k8s/api-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nst-api
spec:
  replicas: 2          # 2-3 replicas per roadmap §Phase 12
  selector:
    matchLabels:
      app: nst-api
  template:
    metadata:
      labels:
        app: nst-api
    spec:
      containers:
        - name: api
          image: ghcr.io/nst/nst-api:latest
          ports:
            - containerPort: 3000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: nst-secrets
                  key: DATABASE_URL
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: nst-secrets
                  key: JWT_SECRET
            - name: GOOGLE_CLIENT_ID
              valueFrom:
                secretKeyRef:
                  name: nst-secrets
                  key: GOOGLE_CLIENT_ID
            - name: GOOGLE_CLIENT_SECRET
              valueFrom:
                secretKeyRef:
                  name: nst-secrets
                  key: GOOGLE_CLIENT_SECRET
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
```

#### `k8s/worker-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nst-worker
spec:
  replicas: 1          # Intentionally 1 — pgmq visibility timeout prevents duplicates
  selector:
    matchLabels:
      app: nst-worker
  template:
    metadata:
      labels:
        app: nst-worker
    spec:
      containers:
        - name: worker
          image: ghcr.io/nst/nst-worker:latest
          command: ["node", "dist/worker/index.js"]
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: nst-secrets
                  key: DATABASE_URL
            - name: EXPO_ACCESS_TOKEN
              valueFrom:
                secretKeyRef:
                  name: nst-secrets
                  key: EXPO_ACCESS_TOKEN
          livenessProbe:
            httpGet:
              path: /health
              port: 3001
```

#### `k8s/secrets.yaml` (values must be base64-encoded; never commit real values)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nst-secrets
type: Opaque
data:
  DATABASE_URL: <base64>
  JWT_SECRET: <base64>
  GOOGLE_CLIENT_ID: <base64>
  GOOGLE_CLIENT_SECRET: <base64>
  COOKIE_SECRET: <base64>
  EXPO_ACCESS_TOKEN: <base64>
```

#### `k8s/ingress.yaml`
```yaml
# Cloudflare Tunnel routes:
# api.nstsdc.org     → nst-api:3000
# dashboard.nstsdc.org → nst-dashboard (Next.js)
```

#### `k8s/postgres-statefulset.yaml`
- **CRITICAL CONSTRAINT**: NST Cluster worker nodes are limited to **8 GB RAM**
- A 3-node CNPG cluster + MinIO will cause OOM cascading failures
- **V1 Decision**: 2-node CNPG cluster + external S3 for WAL archiving
- Authority: `docs/api/13-worker-deployment.md` §Operational Constraints

---

### 9.3 Production Docker Images

**`docker/Dockerfile.api`**
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY pnpm-lock.yaml ./
COPY package.json ./
RUN corepack enable && pnpm install --frozen-lockfile
COPY . .
RUN pnpm --filter @nst/database generate
RUN pnpm --filter @nst/api build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/apps/api/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

**`docker/Dockerfile.worker`**
```dockerfile
FROM node:20-alpine AS builder
# Same multi-stage pattern, different entry point
CMD ["node", "dist/worker/index.js"]
```

---

### 9.4 Secrets Strategy

| Secret | Used By | Source |
|---|---|---|
| `DATABASE_URL` | `nst-api`, `nst-worker` | NST Cluster K8s Secret → env var |
| `JWT_SECRET` | `nst-api` | ≥256-bit random string; `openssl rand -hex 32` |
| `GOOGLE_CLIENT_ID` | `nst-api` | Google Cloud Console → OAuth 2.0 credential |
| `GOOGLE_CLIENT_SECRET` | `nst-api` | Google Cloud Console |
| `COOKIE_SECRET` | `nst-api` | ≥256-bit random string |
| `EXPO_ACCESS_TOKEN` | `nst-worker` | Expo dashboard → Access Tokens |

**Local dev**: `.env` file at repo root (gitignored). Template in `.env.example`.
**Production**: K8s Secret `nst-secrets`. Values managed outside git (manual insert or external secrets operator).

---

### 9.5 CI/CD Requirements

**`.github/workflows/ci.yml`** — Runs on every PR and push to `main`:
```yaml
on: [push, pull_request]
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo typecheck
      - run: pnpm turbo lint
      - run: pnpm turbo test
        env:
          DATABASE_URL: postgresql://nst:nst@localhost:5432/nstevents_test
    services:
      postgres:
        image: postgis/postgis:16-3.4
        # + pgmq setup for integration tests
```

**`.github/workflows/deploy.yml`** — Runs on merge to `main`:
```yaml
on:
  push:
    branches: [main]
jobs:
  deploy:
    steps:
      - docker build + push nst-api → ghcr.io/nst/nst-api:<sha>
      - docker build + push nst-worker → ghcr.io/nst/nst-worker:<sha>
      - kubectl rollout restart deployment/nst-api
      - kubectl rollout restart deployment/nst-worker
      - npx prisma migrate deploy  # run before pod restart
```

---

## Section 10 — Development Sequence

This converts the 12-phase roadmap into an executable, team-owned plan with
explicit definitions of done for each phase.

---

### Phase 0 — Repository Bootstrap
**Duration**: 2–3 days | **Owner**: Senior Backend

| Item | Detail |
|---|---|
| **Deliverables** | `nst-events/` monorepo initialized; pnpm workspaces; Turborepo; all 4 apps scaffolded; 3 packages; `docker-compose.yml`; `.env.example`; ESLint + Prettier + Husky; GitHub Actions CI; `tsconfig` (strict) in all packages |
| **Dependencies** | None — this IS the foundation |
| **Definition of Done** | `pnpm install` succeeds; `docker compose up -d` → PG healthy; `pnpm turbo dev` starts API with hot reload; `pnpm turbo typecheck` → zero errors; CI pipeline passes on first commit |

**Parallel work during Phase 0**:
- Mobile team: Expo project setup in `apps/mobile/`, auth screen UI (no API yet)
- Dashboard team: Next.js in `apps/dashboard/`, shell layout, sidebar navigation

---

### Phase 1 — Database Foundation
**Duration**: 3–5 days | **Owner**: Senior Backend (schema + SQL), Mid Backend (seed)

| Item | Detail |
|---|---|
| **Deliverables** | `schema.prisma` (all 19 tables, 15 enums, all relations); 10 raw SQL migrations (extensions, RLS, triggers, views, RPCs, MVs, search, pg_cron, pgmq); seed script with test data |
| **Dependencies** | Phase 0 complete; Docker Compose PG running |
| **Senior-only items** | `withUserContext` impl; all RLS policies; soft-delete cascade triggers; audit triggers; `mark_attendance` RPC; `register_event` RPC |
| **Junior-safe items** | Seed script; migration testing; `updated_at` trigger scaffolding |
| **Definition of Done** | `npx prisma migrate dev` → zero errors; `npx prisma db seed` → loads test data; `ST_DWithin` query returns expected results; `current_user_id()` returns UUID inside transaction; all 19 tables created with correct constraints; RLS enabled on all tables |

---

### Phase 2 — Authentication
**Duration**: 3–4 days | **Owner**: Senior Backend

| Item | Detail |
|---|---|
| **Deliverables** | `GET /auth/google`, `GET /auth/google/callback`, `POST /auth/refresh`, `POST /auth/logout`; JWT sign/verify (15-min HS256); opaque refresh token (SHA-256, 30-day, HttpOnly cookie); `authenticate` middleware; `withUserContext` integrated into all routes; domain restriction (`@adypu.edu.in`, `@newtonschool.co`) |
| **Dependencies** | Phase 1; Google OAuth credentials |
| **Senior-only items** | OAuth callback (token verify + domain restrict + upsert); refresh token rotation with family revocation; `withUserContext` correctness |
| **Integration Checkpoint IC-1** | Mobile + Dashboard can complete Google OAuth and receive JWT |
| **Definition of Done** | New user upserted on `google_sub`; returning user matched without duplicate; JWT expires 15 min; refresh rotation works; revoked tokens rejected; non-institutional domains → 403; `app.user_id` set correctly in all queries |

---

### Phase 3 — Core Domain Models (Users, Clubs, RBAC)
**Duration**: 3–4 days | **Owner**: Senior + Mid Backend

| Item | Detail |
|---|---|
| **Deliverables** | RBAC middleware (`requireRole`, `requireClubRole`); `GET/PATCH /users/me`, `GET /users/:id/profile`, `POST /users/me/push-token`; `GET /clubs`, `GET /clubs/:id`, `POST /clubs`, `POST/PATCH/DELETE /clubs/:id/members`, `PATCH /clubs/:id/status`; leadership transfer endpoints; `GET /search?type=clubs` |
| **Dependencies** | Phase 2 |
| **Senior-only items** | RBAC middleware (primary auth layer); RLS for `club_memberships` |
| **Integration Checkpoint IC-2** | Frontend can list clubs, view members, see role-based UI |
| **Definition of Done** | Club CRUD with RBAC; membership role changes audited; users update only own profile; RLS independently rejects unauthorized access; FTS returns ranked results |

---

### Phase 4 — Event System
**Duration**: 4–5 days | **Owner**: Senior + Mid Backend

| Item | Detail |
|---|---|
| **Deliverables** | Full event CRUD; `POST /events/:id/submit-for-approval`, `approve`, `reject`, `lock`, `unlock`; sessions CRUD; event state machine (DRAFT→PENDING_APPROVAL→PUBLISHED→ARCHIVED); multi-club via `event_clubs`; JSONB metadata validation per `event_type`; `GET /leaderboard/students`, `GET /leaderboard/clubs` (from MVs) |
| **Dependencies** | Phase 3 |
| **Integration Checkpoint IC-3** | Mobile shows published events feed; Dashboard shows event creation + approval end-to-end |
| **Definition of Done** | State transitions follow documented FSM; only Faculty can approve; multi-club events work; locked events reject registrations/scans; soft-delete cascades |

**After Phase 4, start parallel streams**:
- **Stream B** (Phase 7 — Notifications): Mid Backend
- **Stream F** (Mobile screens): Frontend Mobile team
- **Stream G** (Dashboard event management): Frontend Dashboard team

---

### Phase 5 — Registration System
**Duration**: 3–4 days | **Owner**: Senior Backend (RPCs), Mid Backend (routes)

| Item | Detail |
|---|---|
| **Deliverables** | `register_event` RPC with lock-free atomic increment; `POST /events/:id/register`, `DELETE /events/:id/register`, `POST /events/:id/teams`, `POST/DELETE /teams/:id/join`, `DELETE /teams/:id/leave`, `PATCH /teams/:id`; `GET /events/:id/registrations`, `GET /users/me/registrations`; waitlist auto-promote |
| **Dependencies** | Phase 4 |
| **Senior-only items** | `register_event` RPC (capacity locking, race conditions); waitlist promotion atomicity |
| **Integration Checkpoint IC-4** | Student can register; organizer sees list; capacity limits enforced |
| **Definition of Done** | 500+ concurrent registrations don't oversell; waitlist promoted FIFO; team constraints from `events.metadata`; team leader transfer on leave |

---

### Phase 6 — Attendance System
**Duration**: 5–7 days | **Owner**: Senior Backend

| Item | Detail |
|---|---|
| **Deliverables** | `POST /attendance/generate-qr` (TOTP, 15s window with ADR-005 format); `POST /attendance/mark` → `mark_attendance` RPC (TOTP + ST_DWithin + device collision, returning `attendance_records` per ADR-005 revision); `POST /attendance/sync-offline` (organizer batch); `GET /events/:id/attendance`, `GET /users/me/attendance`; dispute system (`POST/GET/PATCH /attendance/disputes`) |
| **Dependencies** | Phase 5 |
| **Senior-only items** | `mark_attendance` RPC (highest complexity); offline sync trust model; `resolve_attendance_dispute` (SECURITY DEFINER) |
| **Integration Checkpoint IC-5** | Full QR scan → mark → audit flow works end-to-end |
| **Definition of Done** | TOTP rotates 15s with `±1 window` drift tolerance and `ATTENDANCE_QR_SECRET` (Clarified by ADR-005); geofence via ST_DWithin; device collision flagged (not rejected); offline sync restricted to CLUB_ADMIN/CORE_MEMBER; disputes within 24h; all writes trigger audit logs |

---

### Phase 7 — Notifications (Parallel with Phase 5/6)
**Duration**: 3–4 days | **Owner**: Mid Backend

| Item | Detail |
|---|---|
| **Deliverables** | `GET /notifications`, `PATCH /notifications/:id/read`, `PATCH /notifications/read-all`; `GET/PATCH /notifications/preferences`; pgmq enqueueing from event approval/rejection RPCs |
| **Dependencies** | Phase 4 (event state transitions trigger notifications) |

---

### Phase 8 — SSE Real-Time (After Phase 6)
**Duration**: 2–3 days | **Owner**: Senior Backend

| Item | Detail |
|---|---|
| **Deliverables** | `GET /events/:id/live` SSE stream; events: `attendance_count`, `registration_count`, `waitlist_update`, `session_opened`, `session_closed`, `lock_status`, `heartbeat`; JWT auth via query param; 30s heartbeat; `Last-Event-ID` reconnection; max 3 connections/user; **PG LISTEN/NOTIFY for cross-replica fan-out** |
| **Dependencies** | Phase 6 |
| **P1 Note** | PG LISTEN/NOTIFY for cross-replica fan-out is a **P1 deliverable**, not optional — required for correctness with 2–3 replicas. From roadmap §Phase 8 Risks |

---

### Phase 9 — Background Workers (After Phase 7)
**Duration**: 3–4 days | **Owner**: Mid Backend

| Item | Detail |
|---|---|
| **Deliverables** | `nst-worker` deployment; pgmq polling loop (5s, batch 100); Expo Push API; retry backoff (1m/5m/15m); DLQ after 3 failures; delivery write-back; `DeviceNotRegistered` hard-delete; `pg_cron` jobs: MV refresh (5 min), expired token cleanup, archived event transition |
| **Dependencies** | Phase 7 |
| **Integration Checkpoint IC-6** | Push notifications arrive after event approval; inbox shows unread |

---

### Phase 10 — Security Hardening
**Duration**: 3–4 days | **Owner**: Senior Backend

| Item | Detail |
|---|---|
| **Deliverables** | Complete RLS suite vs Table Policy Matrix; all rate limits via `express-rate-limit`; CORS (Helmet.js); Zod validation on all inputs; SQL injection audit; CSRF for refresh cookie; audit log completeness check (no NULL actors) |
| **Dependencies** | Phases 2–9 complete |
| **Open Decision** | CSRF behavior on mobile clients — HttpOnly cookie + SameSite=Strict should work for web; mobile uses Bearer JWT (no cookie), so CSRF does not apply to mobile. Dashboard uses HttpOnly cookie — standard CSRF protection applies |

---

### Phase 11 — Testing
**Duration**: 4–5 days | **Owner**: All Backend

| Item | Detail |
|---|---|
| **Deliverables** | **Unit**: JWT, TOTP, role resolution, Zod schemas (Vitest). **Integration**: OAuth flow (mocked Google), registration capacity locking, attendance with geofence, event state machine, soft-delete cascade, RLS enforcement, notification pipeline (Supertest + testcontainers). **Load**: 500 concurrent registrations, 200 scans/min, 1000-user broadcast |
| **Dependencies** | Phase 10 |
| **Definition of Done** | 80%+ coverage on business logic; all RPCs have integration tests; concurrency tests verify no overselling; RLS tests verify independent enforcement; CI green on all branches |

---

### Phase 12 — Deployment
**Duration**: 2–3 days | **Owner**: Senior Backend

| Item | Detail |
|---|---|
| **Deliverables** | Docker images: `nst-api`, `nst-worker`; K8s manifests finalized; Secrets populated; Cloudflare Tunnel Ingress; liveness/readiness probes; migration strategy: `prisma migrate deploy` + raw SQL in CI; rollback procedure documented |
| **Dependencies** | Phase 11 complete; Open Decision: DB hosting provider |
| **Definition of Done** | `api.nstsdc.org` responds 200; worker processes notifications; migrations apply cleanly; health checks pass; TLS via Cloudflare; rollback tested |

---

## Section 11 — Gap Analysis

These are repository-level decisions that remain partially unresolved and
could force engineers to make assumptions on Day 1. Each is classified by
blocking severity.

---

### 🔴 BLOCKING

| Gap | Detail | Resolution Needed |
|---|---|---|
| **pgmq Docker image** | The documentation requires pgmq as a PostgreSQL extension but no specific Docker image is mandated. `postgis/postgis:16-3.4` does not include pgmq. Engineers need a ready-to-run local dev image before Phase 1 can begin | Team must decide: (A) Build custom Docker image with pgmq compiled from source, (B) Use a community image (e.g., Tembo's), or (C) Use Neon/Supabase-hosted PG for local dev with pgmq pre-installed |
| **Database Hosting Provider** | Flagged as "Under Review" in MASTER_CONTEXT.md. K8s manifests differ significantly between CNPG StatefulSet vs. external managed PostgreSQL | Needed before Phase 12. Does NOT block Phases 0–11 which use Docker Compose |
| **Google OAuth Credentials** | `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` must be provisioned before Phase 2. Requires a Google Cloud project and OAuth 2.0 credential with correct redirect URIs | Needed before Phase 2 starts (≈Day 5–8) |
| **CSRF on mobile** | `docs/api/12-api-freeze-v1.md` lists this as "Blocking for T-117 only." Mobile uses Bearer JWT (no cookie), so CSRF does not apply to mobile auth. However, the dashboard uses HttpOnly cookie — CSRF protection must be decided for dashboard-initiated requests | Architecture answer: SameSite=Strict on refresh cookie + `X-Requested-With` header check. Mobile exempt. Unblock T-117 with this decision |

---

### 🟠 HIGH PRIORITY (Non-Blocking for Phase 0–1, blocks later phases)

| Gap | Detail | Resolution Needed Before |
|---|---|---|
| **Testing framework choice** | Roadmap says "Vitest/Jest." Choice affects tooling configuration (jest.config vs vitest.config, ESM compatibility). Vitest is recommended but not formally decided | Phase 11 (ideally decided at Phase 0 to configure CI correctly) |
| **Logging framework** | Documentation does not name a logger. `console.log` is acceptable for Phase 0–1 but production requires structured JSON logs. Without a decision, each developer may introduce different logging patterns | Phase 3–4 (before API surface grows) |
| **pnpm version pinning** | ADR-003 mentions "Turborepo/Yarn Workspaces" as an alternative — pnpm is implied by modern Turborepo setups but never formally named. If some developers use npm, workspace linking will break | Phase 0 — must be decided on Day 1 |
| **Expo SDK version** | ADR-004 says "Expo React Native" but does not specify SDK version. Expo SDK 50 vs 51 vs 52 have different Expo Router and NativeWind compatibility implications | Phase 0 for mobile team |
| **Next.js version and rendering mode** | ADR-005 says "Next.js" but does not specify version or whether to use App Router (Next.js 13+) vs Pages Router. This audit assumes App Router (Next.js 14+) based on modern practice | Phase 0 for dashboard team |

---

### 🟡 NICE TO HAVE (No implementation blocker)

| Gap | Detail |
|---|---|
| **API versioning strategy** | Base URL is `https://api.nstsdc.org/v1`. What happens when breaking changes are needed in V2? No deprecation policy is documented |
| **Error monitoring** | No Sentry / Datadog integration is specified. Production errors will be visible only in K8s pod logs until monitoring is added |
| **Database migration rollback** | Roadmap mentions "rollback procedure" but does not define it. Prisma does not support automatic down-migrations. Manual rollback scripts are needed for each migration |
| **pnpm vs Yarn Berry** | ADR-003 mentions "Turborepo/Yarn Workspaces" but modern Turborepo docs recommend pnpm. This audit assumes pnpm throughout. If Yarn Berry is preferred, `pnpm-workspace.yaml` changes to `package.json#workspaces` |
| **Rate limiting across replicas** | Roadmap §Phase 10 Risks: "Rate limiting across replicas needs shared state or per-replica limits." In-memory `express-rate-limit` does not share state across pods. V1 accepts per-replica limits. If true global limits are needed, a Redis store is required — which is NOT in the current stack |
| **Expo Push token cleanup** | The `push_tokens` stale cleanup runs via `pg_cron` at 90 days. The cron expression is not specified in any document |

---
