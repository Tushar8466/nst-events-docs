# NST-Events Day-0 Repository Extraction Audit
# Part 4 of 4 — Final Output: Canonical Structure, Setup Checklist & Bootstrap Commands

> This is the actionable execution document. A backend team with this file and
> Parts 1–3 can create the repository and begin Phase 1 immediately without
> making architectural assumptions.

---

## Section 12 — Final Output

---

### 12.A — Canonical Repository Structure

The single source of truth for what the repository must look like on Day 0.
See Part 1 §Section 1 for the complete annotated tree. This is the condensed
reference version.

```
nst-events/
├── apps/
│   ├── api/          @nst/api      Express REST API (nst-api, 2-3 replicas)
│   ├── worker/       @nst/worker   native queue notification worker (nst-worker, 1 replica)
│   ├── mobile/       @nst/mobile   Expo React Native student app
│   └── dashboard/    @nst/dashboard Next.js admin dashboard
├── packages/
│   ├── database/     @nst/database  Prisma schema + migrations + seed
│   ├── shared/       @nst/shared    Types + constants + Zod validators (all apps)
│   └── config/       @nst/config    ESLint + TSConfig + Prettier (all apps)
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
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── turbo.json
├── package.json          # pnpm workspace root
├── pnpm-workspace.yaml
├── .env.example
├── .nvmrc                # "20"
├── .eslintrc.js
├── .prettierrc.js
└── .gitignore
```

---

### 12.B — Exact Day-0 Setup Checklist

Complete this checklist **in order** before writing any business logic.

#### Pre-Repository (Human decisions needed)

- [ ] **Provision Google OAuth credentials** — Create OAuth 2.0 client in Google Cloud Console. Add redirect URIs: `http://localhost:3000/v1/auth/google/callback` (dev), `https://api.nstsdc.org/v1/auth/google/callback` (prod)
- [ ] **Decide on native queue Docker image** — Options: Tembo community image, custom build from `postgis/postgis:16-3.4`, or cloud-hosted PG with native queue. Must be running locally before Phase 1
- [ ] **Create GitHub repo** `nst-events` under NST GitHub organization
- [ ] **Add GitHub Secrets** (for CI): `GHCR_TOKEN`, `NST_CLUSTER_KUBECONFIG`
- [ ] **Create NST K8s namespace** `nst-events`

#### Phase 0 Checklist (Repository Bootstrap)

**Monorepo scaffold**
- [ ] Initialize repo with `git init && git commit --allow-empty -m "chore: init"`
- [ ] Create `pnpm-workspace.yaml`
- [ ] Create root `package.json` with workspace config
- [ ] Install Turborepo: `pnpm add -Dw turbo`
- [ ] Create `turbo.json` with `build`, `dev`, `typecheck`, `lint`, `test` pipelines

**Packages scaffold** (create `packages/config` first, others depend on it)
- [ ] `packages/config/` — ESLint base, TSConfig bases, Prettier config
- [ ] `packages/shared/` — `package.json`, `tsconfig.json`, empty `src/` dirs
- [ ] `packages/database/` — `package.json`, `tsconfig.json`, `prisma/schema.prisma` (empty initially)

**Apps scaffold**
- [ ] `apps/api/` — `package.json`, `tsconfig.json`, `src/index.ts`, `src/app.ts`
- [ ] `apps/worker/` — `package.json`, `tsconfig.json`, `src/index.ts`
- [ ] `apps/mobile/` — Move or scaffold Expo project; update `package.json` name to `@nst/mobile`
- [ ] `apps/dashboard/` — Scaffold Next.js 14 App Router; update `package.json` name to `@nst/dashboard`

**Foundation files**
- [ ] `.env.example` with all required env vars documented (copy from Part 2 §5.2)
- [ ] `.env` with local development values (gitignored)
- [ ] `.nvmrc` containing `20`
- [ ] `.gitignore` (node_modules, dist, .env, *.js.map, .turbo)
- [ ] Root `.eslintrc.js` extending `@nst/config`
- [ ] Root `.prettierrc.js` extending `@nst/config`

**Docker**
- [ ] `docker/docker-compose.yml` — PostgreSQL 16 + PostGIS + native queue
- [ ] `docker/Dockerfile.api` — stub (fill fully in Phase 12)
- [ ] `docker/Dockerfile.worker` — stub

**K8s manifests** (stubs — fill values in Phase 12)
- [ ] `k8s/api-deployment.yaml`
- [ ] `k8s/worker-deployment.yaml`
- [ ] `k8s/postgres-statefulset.yaml`
- [ ] `k8s/secrets.yaml`
- [ ] `k8s/ingress.yaml`

**CI/CD**
- [ ] `.github/workflows/ci.yml` — pnpm install + turbo typecheck + turbo lint + turbo test
- [ ] `.github/workflows/deploy.yml` — docker build + push + kubectl rollout

**Husky + lint-staged**
- [ ] Install: `pnpm add -Dw husky lint-staged`
- [ ] Init: `npx husky install`
- [ ] Add pre-commit hook: `npx lint-staged`
- [ ] Configure `.lintstagedrc.json`

**API foundation files** (must compile before Phase 1)
- [ ] `apps/api/src/config/env.ts` — Zod-parsed env loader
- [ ] `apps/api/src/lib/prisma.ts` — PrismaClient singleton
- [ ] `apps/api/src/lib/db.ts` — `withUserContext` wrapper
- [ ] `apps/api/src/lib/jwt.ts` — `signJwt` / `verifyJwt`
- [ ] `apps/api/src/lib/errors.ts` — `AppError` class hierarchy
- [ ] `apps/api/src/types/express.d.ts` — `req.user` augmentation
- [ ] `apps/api/src/middleware/authenticate.ts` — JWT middleware stub
- [ ] `apps/api/src/middleware/error-handler.ts` — RFC 7807 error handler
- [ ] `apps/api/src/app.ts` — Express factory with `/health` endpoint
- [ ] `apps/api/src/index.ts` — `app.listen()`

**Phase 0 Verification**
- [ ] `pnpm install` — succeeds, zero errors
- [ ] `docker compose up -d` — PostgreSQL healthy on `localhost:5432`
- [ ] `pnpm turbo typecheck` — zero TypeScript errors
- [ ] `pnpm turbo lint` — zero lint errors
- [ ] `curl localhost:3000/health` — returns `{ "status": "ok" }`
- [ ] GitHub Actions CI — passes on first push

#### Phase 1 Checklist (Database Foundation)

- [ ] Complete `packages/database/prisma/schema.prisma` with all 19 tables, 15 enums, all relations
- [ ] Run `npx prisma migrate dev --create-only --name init` → generates `0001_init/migration.sql`
- [ ] Create `0002_extensions/migration.sql` — all `CREATE EXTENSION` statements
- [ ] Create `0003_rls_policies/migration.sql` — `current_user_id()` + all RLS policies
- [ ] Create `0004_triggers/migration.sql` — `updated_at`, audit, soft-delete cascade triggers
- [ ] Create `0005_views/migration.sql` — soft-delete views with `security_invoker = true`
- [ ] Create `0006_rpcs/migration.sql` — all 17+ stored procedures
- [ ] Create `0007_materialized_views/migration.sql` — `club_leaderboard_mv`, `student_leaderboard_mv`
- [ ] Create `0008_search/migration.sql` — `tsvector` columns + GIN indexes + GiST on `location_geofence`
- [ ] Create `0009_pgcron/migration.sql` — MV refresh every 5 min; token cleanup daily
- [ ] Create `0010_native queue_queues/migration.sql` — `CREATE TABLE notification_jobs`
- [ ] `npx prisma migrate dev` — applies all migrations cleanly
- [ ] `npx prisma generate` — TypeScript client generated
- [ ] Create `packages/database/prisma/seed.ts`
- [ ] `npx prisma db seed` — loads test data

**Phase 1 Verification**
- [ ] All 19 tables exist with correct column types
- [ ] `SELECT current_user_id()` inside transaction returns UUID
- [ ] `SELECT ST_DWithin(...)` — PostGIS query executes
- [ ] `SELECT native queue.create('test')` — native queue is functional
- [ ] RLS enabled: `SELECT relrowsecurity FROM pg_class WHERE relname = 'events'` → `true`
- [ ] Seed data: Platform Admin exists, 3 clubs exist, 2 events exist

---

### 12.C — Exact Files and Folders to Create Before Writing Business Logic

This is the minimum set. Create in this order:

```
ORDER   PATH                                          NOTES
──────  ───────────────────────────────────────────  ─────────────────────────────
1.      pnpm-workspace.yaml                          Enables workspace linking
2.      package.json (root)                          Workspace root + turbo scripts
3.      turbo.json                                   Pipeline definitions
4.      .nvmrc                                       Pin Node 20
5.      .env.example                                 Document all vars
6.      .env                                         Local secrets (gitignored)
7.      .gitignore                                   Prevent secrets commit
8.      packages/config/package.json
9.      packages/config/tsconfig.base.json           "strict": true
10.     packages/config/tsconfig.node.json           For api + worker
11.     packages/config/tsconfig.react.json          For dashboard
12.     packages/config/tsconfig.expo.json           For mobile
13.     packages/config/eslint-base.js
14.     packages/config/prettier.config.js
15.     packages/shared/package.json
16.     packages/shared/tsconfig.json
17.     packages/shared/src/types/roles.ts           Role enums (needed by RBAC)
18.     packages/shared/src/types/errors.ts          RFC 7807 shape
19.     packages/shared/src/constants/roles.ts
20.     packages/shared/src/constants/limits.ts
21.     packages/database/package.json
22.     packages/database/tsconfig.json
23.     packages/database/prisma/schema.prisma       All tables + enums
24.     packages/database/src/client.ts              PrismaClient singleton
25.     packages/database/src/context.ts             withUserContext()
26.     apps/api/package.json
27.     apps/api/tsconfig.json
28.     apps/api/src/config/env.ts                   Fail-fast env loader
29.     apps/api/src/lib/errors.ts                   AppError hierarchy
30.     apps/api/src/lib/jwt.ts                      signJwt / verifyJwt
31.     apps/api/src/lib/prisma.ts                   PrismaClient re-export
32.     apps/api/src/lib/db.ts                       withUserContext import
33.     apps/api/src/types/express.d.ts              req.user augmentation
34.     apps/api/src/middleware/error-handler.ts     RFC 7807 handler
35.     apps/api/src/middleware/authenticate.ts      JWT → req.user stub
36.     apps/api/src/middleware/validate.ts          Zod middleware factory
37.     apps/api/src/middleware/authorize.ts         RBAC stub
38.     apps/api/src/middleware/rate-limit.ts        Rate limit configs
39.     apps/api/src/app.ts                          Express factory + /health
40.     apps/api/src/index.ts                        app.listen()
41.     apps/worker/package.json
42.     apps/worker/tsconfig.json
43.     apps/worker/src/index.ts                     Polling loop stub
44.     apps/worker/src/health.ts                    /health endpoint
45.     docker/docker-compose.yml                    PG + native queue locally
46.     docker/Dockerfile.api                        Multi-stage build stub
47.     docker/Dockerfile.worker                     Multi-stage build stub
48.     k8s/*.yaml (5 files)                         Manifest stubs
49.     .github/workflows/ci.yml
50.     .github/workflows/deploy.yml
```

---

### 12.D — Exact Commands to Bootstrap the Repository

Run these commands in sequence. Every command is documented with its purpose.

```bash
# ─── Prerequisites ────────────────────────────────────────────────────────────
node --version           # Must be v20.x.x
docker --version         # Must be 24+
pnpm --version           # Must be 9.x; install: npm install -g pnpm@9

# ─── Step 1: Initialize repository ──────────────────────────────────────────
mkdir nst-events && cd nst-events
git init
git commit --allow-empty -m "chore: init empty repository"

# ─── Step 2: Root workspace setup ────────────────────────────────────────────
# Create pnpm-workspace.yaml:
cat > pnpm-workspace.yaml << 'EOF'
packages:
  - "apps/*"
  - "packages/*"
EOF

# Create root package.json:
cat > package.json << 'EOF'
{
  "name": "nst-events",
  "private": true,
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "typecheck": "turbo typecheck",
    "lint": "turbo lint",
    "test": "turbo test",
    "prepare": "husky install"
  },
  "devDependencies": {}
}
EOF

# ─── Step 3: Install Turborepo ───────────────────────────────────────────────
pnpm add -Dw turbo

# ─── Step 4: Create all directory structure ───────────────────────────────────
mkdir -p apps/{api,worker,mobile,dashboard}
mkdir -p packages/{database,shared,config}
mkdir -p docker k8s .github/workflows

# ─── Step 5: Install Husky ───────────────────────────────────────────────────
pnpm add -Dw husky lint-staged
npx husky install

# ─── Step 6: Install pnpm workspace packages ──────────────────────────────────
# In packages/database/:
pnpm --filter @nst/database add prisma @prisma/client

# In packages/shared/:
pnpm --filter @nst/shared add zod

# In apps/api/:
pnpm --filter @nst/api add express helmet cors cookie-parser \
  jsonwebtoken otplib express-rate-limit zod \
  passport passport-google-oauth20

pnpm --filter @nst/api add -D \
  typescript ts-node-dev @types/express @types/jsonwebtoken \
  @types/cookie-parser @types/passport @types/passport-google-oauth20

pnpm --filter @nst/api add @nst/database @nst/shared

# In apps/worker/:
pnpm --filter @nst/worker add expo-server-sdk node-fetch
pnpm --filter @nst/worker add -D typescript ts-node-dev @types/node
pnpm --filter @nst/worker add @nst/database @nst/shared

# ─── Step 7: Initialize Prisma ────────────────────────────────────────────────
cd packages/database
npx prisma init --datasource-provider postgresql
# → Creates prisma/schema.prisma and .env (move DATABASE_URL to root .env)
cd ../..

# ─── Step 8: Start local database ────────────────────────────────────────────
docker compose -f docker/docker-compose.yml up -d

# Wait for healthy:
docker compose -f docker/docker-compose.yml ps
# postgres → healthy

# ─── Step 9: Run first migration ──────────────────────────────────────────────
# (After schema.prisma is populated with all 19 tables)
cd packages/database
npx prisma migrate dev --name init
npx prisma generate
npx prisma db seed
cd ../..

# ─── Step 10: Verify full stack ──────────────────────────────────────────────
pnpm install                         # All workspace deps installed
pnpm turbo typecheck                 # Zero TS errors across all packages
pnpm turbo lint                      # Zero ESLint errors
pnpm --filter @nst/api dev &         # Start API with hot reload
sleep 3
curl http://localhost:3000/health    # → {"status":"ok"}

# ─── Step 11: Verify database features ───────────────────────────────────────
psql postgresql://nst:nst@localhost:5432/nstevents << 'SQL'
SELECT current_user_id();            -- Should return NULL (no context yet)
SELECT postgis_version();            -- Should return PostGIS version
SELECT native queue.create('test_queue');   -- Should succeed
DROP TABLE native queue.q_test_queue;       -- Cleanup
SQL

# ─── Phase 0 DONE ─────────────────────────────────────────────────────────────
echo "Phase 0 complete. Ready to begin Phase 1: Database Foundation."
```

---

## Quick Reference Summary

| Artifact | Location |
|---|---|
| **Part 1**: Structure + Inventory + Tooling | `docs/backend/day0-audit-part1-structure-and-inventory.md` |
| **Part 2**: Backend + Database + Frontend Foundation | `docs/backend/day0-audit-part2-backend-and-database.md` |
| **Part 3**: Infrastructure + Dev Sequence + Gap Analysis | `docs/backend/day0-audit-part3-infra-and-sequence.md` |
| **Part 4**: Canonical Structure + Checklist + Commands | `docs/backend/day0-audit-part4-final-output.md` (this file) |

| Blocking Gap | Resolution Owner |
|---|---|
| native queue Docker image | Backend Tech Lead — decide by Phase 0 Day 1 |
| Google OAuth credentials | Platform Admin / Google Cloud access holder |
| Database hosting provider | Needed before Phase 12; does not block Phases 0–11 |
| CSRF on mobile | Architecture answer in Part 3 §11: mobile exempt, dashboard uses SameSite=Strict |

| Phase | Duration | Critical Path |
|---|---|---|
| 0 — Repository Setup | 2–3 days | ✅ Yes |
| 1 — Database Foundation | 3–5 days | ✅ Yes |
| 2 — Authentication | 3–4 days | ✅ Yes |
| 3 — Core Domain | 3–4 days | ✅ Yes |
| 4 — Event System | 4–5 days | ✅ Yes |
| 5 — Registration | 3–4 days | ✅ Yes |
| 6 — Attendance | 5–7 days | ✅ Yes |
| 7 — Notifications | 3–4 days | Parallel after Ph.4 |
| 8 — SSE | 2–3 days | Parallel after Ph.6 |
| 9 — Workers | 3–4 days | Parallel after Ph.7 |
| 10 — Security | 3–4 days | After Ph.2–9 |
| 11 — Testing | 4–5 days | After Ph.10 |
| 12 — Deployment | 2–3 days | Final |
| **Total** | **~8 weeks** | |
