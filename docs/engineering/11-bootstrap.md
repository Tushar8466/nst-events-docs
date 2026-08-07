# Bootstrap Guide

**Document ID**: ENG-012
**Version**: 1.0.0
**Status**: Frozen
**Authority Level**: Normative
**Document Type**: Guide
**Owner**: Engineering
**Supersedes**: None
**Last Reviewed**: 2026-08-08
**Next Review**: 2027-02-08
**Review Cadence**: 6 Months

## Purpose
This document provides the canonical lifecycle for onboarding a new developer to the repository. Following these steps guarantees a reproducible, production-ready local development environment within approximately 15 minutes.

## Scope
- End-to-end local environment setup.
- Quality Gate validation.
- Initial database seeding.

## Out of Scope
- Vendor-specific installation commands for prerequisites (refer to official documentation for your OS).

## References
- [ENG-004: Quality Gates](./03-quality-gates.md)
- [ENG-009: Tooling Contract](./08-tooling-contract.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of the Bootstrap Guide. |

---

## 1. Prerequisites Validation

Ensure your environment complies with the [Tooling Contract (ENG-009)](./08-tooling-contract.md). You must have the following installed:
* Git
* Compatible Node.js LTS
* Approved Package Manager
* Local Container Engine (for databases)

## 2. Clone Repository

Securely clone the repository and navigate into the root directory.

## 3. Install Dependencies

Execute the package manager's install command to fetch exact dependencies as specified in the lockfile. Do not update or alter the lockfile during initial bootstrap.

## 4. Configure Environment

1. Duplicate the `.env.example` file to `.env`.
2. Provision local cryptographic secrets (e.g., JWT secrets) using secure random generators. Do not use production secrets locally.

## 5. Initialize Database

1. Spin up the local database container.
2. Execute the ORM migration commands to apply the latest schema.
3. Run the database seed script to populate canonical test data (e.g., dummy users, clubs, and events).

## 6. Validate Installation

Execute the Tier A Local Quality Gates ([ENG-004](./03-quality-gates.md)):
1. Run the strict linter.
2. Run the type-checker.
3. Execute the unit test suite.

If any of these fail on a fresh clone of the `main` branch, halt and report a repository drift issue.

## 7. Run Development Environment

Initiate the Monorepo orchestrator to start the backend API, the background worker, and the web/mobile frontends concurrently.

## 8. Ready for Development

You are now fully onboarded. Please review the [Git Workflow (ENG-003)](./02-git-workflow.md) before making your first commit.
