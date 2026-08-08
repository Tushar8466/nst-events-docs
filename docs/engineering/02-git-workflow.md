# Git Workflow

**Document ID**: ENG-003
**Version**: 1.0.0
**Status**: Frozen
**Authority Level**: Normative
**Document Type**: Workflow
**Owner**: Engineering
**Supersedes**: None
**Last Reviewed**: 2026-08-08
**Next Review**: 2027-02-08
**Review Cadence**: 6 Months

## Purpose
This document outlines the deterministic Git lifecycle for all code contributions, mapping the journey from local development through to the protected main branch.

## Scope
- Developer local workflow.
- Commit and Pre-push boundaries.
- Remote CI integration steps.

## Out of Scope
- Specific branching formats (see [ENG-002: Engineering Standards](./01-engineering-standards.md)).
- Post-merge release procedures (see [ENG-007: Release Process](./06-release-process.md)).

## References
- [ENG-001: Glossary](./00-glossary.md)
- [ENG-004: Quality Gates](./03-quality-gates.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of Git Workflow. |

---

## The Deterministic Lifecycle

Every code contribution must pass strictly through the following stages:

### 1. Developer Implementation
The developer branches off the latest `main`, implements the feature, writes corresponding tests, and updates relevant canonical documentation.

### 2. Local Validation (Tier A)
Before committing, the developer triggers local Validation gates ([ENG-004](./03-quality-gates.md)). This stage must execute rapidly (under 3 minutes) and includes basic linting, formatting checks, and type validation.

### 3. Commit
The code is committed using the enforced Conventional Commits standard.

### 4. Pre-Push Execution
Upon initiating a push to the remote, an automated Husky pre-push hook executes. This stage runs the canonical `pnpm validate:local` command which enforces immutable Repository Invariants (lint and typecheck), ensuring no placeholder logic or obvious violations enter the remote repository. If this Gate fails, the hook exits non-zero and the push is rejected locally.

### 5. Remote Sync
The feature branch is synced to the remote repository.

### 6. Continuous Integration (Tier B)
The remote repository automatically triggers the comprehensive CI Pipeline. This tier performs exhaustive testing, builds, security scans, and database migrations. CI is the ultimate source of truth.

### 7. Review and Certification
A peer or Principal Architect reviews the Pull Request. For AI-assisted workflows, explicit Production Certification is conducted here.

### 8. Protected Branch Merge
Once CI is green and approvals are met, the Pull Request is merged into the protected `main` branch. 

### 9. Release
The merged code is now subject to the standard Release Process ([ENG-007](./06-release-process.md)).
