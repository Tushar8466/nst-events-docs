# Tooling Contract

**Document ID**: ENG-009
**Version**: 1.0.0
**Status**: Frozen
**Authority Level**: Informative
**Document Type**: Policy
**Owner**: Engineering
**Supersedes**: None
**Last Reviewed**: 2026-08-08
**Next Review**: 2027-02-08
**Review Cadence**: 6 Months

## Purpose
This is the **only** document in the Engineering Handbook permitted to name specific tools, vendors, or frameworks. It maps the theoretical principles described across the handbook to their concrete, current implementations. 

## Scope
- Authorized languages, frameworks, and runtimes.
- Deployment and CI/CD tooling.
- Quality and formatting tools.

## Out of Scope
- Architectural principles (these belong in the rest of the Handbook).

## References
- [ENG-000: README](./README.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of the Tooling Contract. |

---

## 1. Development Environment

The officially supported development environments are **macOS** and **Linux**. Windows is not natively supported for local development unless running via WSL2.

## 2. Core Stack

| Domain | Authorized Tool | Version Constraints |
|--------|-----------------|---------------------|
| Runtime | **Node.js** | LTS (v20+) |
| Package Manager | **pnpm** | v9+ |
| Monorepo Orchestration | **Turbo** (Turborepo) | Latest |
| Language | **TypeScript** | v5+ (Strict Mode Mandatory) |
| Backend Framework | **Express.js** | v4.x |
| ORM | **Prisma** | Latest |
| Mobile Framework | **Expo** (React Native) | SDK 51+ |
| Mobile Routing | **Expo Router** | Latest |
| Web Framework | **Next.js** | v14+ (App Router) |
| Database | **PostgreSQL** | v16+ (with PostGIS) |

## 3. Quality & Formatting

| Domain | Authorized Tool | Notes |
|--------|-----------------|-------|
| Code Formatting | **Prettier** | Strict automation; no manual debate. |
| Linting | **ESLint** | Typescript-eslint recommended rules. |
| Fast Rust-based Linting | **Biome** | Evaluated for future migration, currently supplementary. |
| Unit Testing | **Vitest** | Preferred over Jest for speed and ESM support. |

## 4. Git & Workflow

| Domain | Authorized Tool | Notes |
|--------|-----------------|-------|
| Git Hooks | **Husky** | Auto-installed via `prepare` script. Enforces Tier A local Quality Gates. |
| CI/CD Pipeline | **GitHub Actions** | Enforces Tier B remote Quality Gates. |

## 5. Infrastructure & Deployment

| Domain | Authorized Tool | Notes |
|--------|-----------------|-------|
| Local Data | **Docker** & **Docker Compose** | Used strictly for PostgreSQL spin-ups locally. |
| Production Orchestration | **Kubernetes** (**K3s**) | Powers the NST Cluster deployment. |
| Production Logging | **Pino** | Structured JSON logging for observability. |
