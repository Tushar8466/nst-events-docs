# Engineering Standards

**Document ID**: ENG-002
**Version**: 1.0.0
**Status**: Frozen
**Authority Level**: Normative
**Document Type**: Standard
**Owner**: Engineering
**Supersedes**: None
**Last Reviewed**: 2026-08-08
**Next Review**: 2027-02-08
**Review Cadence**: 6 Months

## Purpose
This document defines the normative baseline for code formatting, styling, naming conventions, review expectations, and branch strategy to ensure a uniform codebase.

## Scope
- Code formatting and linting rules.
- Naming conventions.
- Branching and commit conventions.
- Code review expectations.

## Out of Scope
- Vendor-specific tool configurations (see [ENG-009: Tooling Contract](./08-tooling-contract.md)).
- Git lifecycle operations (see [ENG-003: Git Workflow](./02-git-workflow.md)).

## References
- [ENG-001: Glossary](./00-glossary.md)
- [ENG-ADR-002: Conventional Commits](./adrs/ENG-ADR-002-conventional-commits.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of Engineering Standards. |

---

## 1. Branch Strategy

The repository follows a trunk-based development strategy with short-lived feature branches.
* **Main Branch**: The `main` branch is the sole integration branch and is permanently protected. Direct pushes are prohibited.
* **Feature Branches**: Must be branched directly from `main` and should merge back into `main` within 2-3 days.
* **Branch Naming**: Must follow the convention: `type/issue-id-short-description` (e.g., `feat/auth-middleware`, `fix/cache-invalidation`).

## 2. Commit Conventions

All commits must adhere to the Conventional Commits standard.
* Format: `<type>(<scope>): <subject>`
* Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
* A descriptive body is strongly recommended for complex changes.

## 3. Formatting and Code Style

* **Consistency**: Formatting is strictly automated via formatters. Manual debates over whitespace are prohibited.
* **Linting**: Code must pass all strict linting rules without warnings. The `any` type in TypeScript is prohibited unless explicitly bypassed with a documented rationale.
* **Imports**: Absolute imports should be favored from the root of a package/app. Imports between monorepo packages must be done via explicit package exports.

## 4. Naming Conventions

* **Files and Directories**: `kebab-case` strictly (e.g., `user-profile.tsx`, `auth-service.ts`).
* **Variables and Functions**: `camelCase` (e.g., `getUserProfile`).
* **Classes and Types/Interfaces**: `PascalCase` (e.g., `UserProfileService`, `UserContext`).
* **Constants**: `UPPER_SNAKE_CASE` (e.g., `MAX_RETRY_ATTEMPTS`).
* **Booleans**: Should be prefixed with `is`, `has`, or `should` (e.g., `isValidated`).

## 5. Review Expectations

* **Self-Review**: Developers must complete a self-review of their own diff before requesting external review.
* **Turnaround**: Pull Requests should receive a first-pass review within 24 hours.
* **Approval Requirement**: A minimum of one engineering approval is required. AI-generated changes require explicit human sign-off.
* **Scope Control**: PRs must remain focused on a single logical change. Scope creep must be pushed to a new branch.

## 6. Documentation Requirements

* All new features must include updates to canonical documentation prior to code being merged.
* See [ENG-010: Documentation Architecture](./09-documentation-architecture.md) for specifics on documentation expectations.
