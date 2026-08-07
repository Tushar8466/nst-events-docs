# Release Process

**Document ID**: ENG-007
**Version**: 1.0.0
**Status**: Frozen
**Authority Level**: Normative
**Document Type**: Policy
**Owner**: Engineering
**Supersedes**: None
**Last Reviewed**: 2026-08-08
**Next Review**: 2027-02-08
**Review Cadence**: 6 Months

## Purpose
This document defines the canonical lifecycle of a release, detailing how code transitions from development into a versioned, deployed state in production.

## Scope
- Release pipeline stages.
- Semantic versioning policy.
- Changelog generation and maintenance policy.

## Out of Scope
- Emergency hotfix recovery specifics (see [ENG-013: Disaster Recovery](./12-disaster-recovery.md)).
- Git workflow mechanics (see [ENG-003: Git Workflow](./02-git-workflow.md)).

## References
- [ENG-003: Git Workflow](./02-git-workflow.md)
- [ENG-013: Disaster Recovery](./12-disaster-recovery.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of Release Process policy. |

---

## 1. The Release Lifecycle

The release pipeline follows a strict, sequential path:

1. **Development**: Feature work is performed on isolated branches.
2. **Feature Complete**: Development is concluded; local tests pass.
3. **Quality Gates**: Remote CI (Tier B) evaluates the code.
4. **Merge**: Code is merged into the integration branch (`main`).
5. **Release Candidate (RC)**: An artifact is minted, versioned, and deployed to staging.
6. **Production**: Following staging certification, the RC artifact is deployed to production.
7. **Hotfix**: Emergency, out-of-band patches applied directly against the production commit, then backported to `main`.
8. **Rollback**: Instantaneous reversion to a previous healthy state (not a fix-forward approach).
9. **Patch Release**: Standard progression of minor bug fixes.

## 2. Semantic Versioning Policy

NST Events adheres strictly to Semantic Versioning (`MAJOR.MINOR.PATCH`).

* **MAJOR**: Architectural overhauls, backwards-incompatible API changes, or major data schema restructuring.
* **MINOR**: New features, new API endpoints, or substantial architectural additions that are backwards-compatible.
* **PATCH**: Bug fixes, security patches, performance improvements, and documentation updates.

Versions are minted prior to the Release Candidate stage to ensure the artifact retains a static identity across environments.

## 3. Changelog Policy

The repository maintains a single `CHANGELOG.md` file following the "Keep a Changelog" format.

* **Source of Truth**: The changelog is a legal historical record of what was actually shipped. It is derived solely from implementation evidence (code, API routes, migrations), not planned features.
* **Categorization**: Entries must be grouped under `Added`, `Changed`, `Fixed`, `Removed`, `Security`, `Performance`, or `Infrastructure`.
* **Automation**: The changelog should leverage the Conventional Commits history for drafting, but must be manually curated before final release to ensure conciseness, professionalism, and the removal of implementation details (e.g., removing specific file names).
