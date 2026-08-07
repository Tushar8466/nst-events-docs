# Quality Gates

**Document ID**: ENG-004
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
This document establishes the two-tier Quality Gate architecture that prevents non-compliant code from entering the integration pipeline.

## Scope
- Definition of Tier A (Local) Quality Gates.
- Definition of Tier B (Remote) Quality Gates.

## Out of Scope
- CI architecture responsibilities (see [ENG-005: CI/CD Architecture](./04-ci-cd-architecture.md)).
- Security-specific scanning (see [ENG-006: Security Pipeline](./05-security-pipeline.md)).

## References
- [ENG-001: Glossary](./00-glossary.md)
- [ENG-ADR-001: Local-first Quality Gates](./adrs/ENG-ADR-001-local-first-quality-gates.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of Quality Gates standard. |

---

## Architecture Principles

NST Events employs a bifurcated approach to Quality Gates to maintain high developer velocity while guaranteeing code integrity.

### Tier A (Local) Gates

Tier A Gates execute on the developer's local machine via pre-commit or pre-push hooks.

* **Velocity Rule**: Tier A must complete execution within approximately 2–3 minutes.
* **Prohibited Constraints**: Do NOT put container builds, full integration test suites, or heavy database migrations inside local quality gates.
* **Scope**: 
  - Code formatting validation.
  - Strict linting.
  - Type checking (across all workspaces).
  - Fast, isolated unit tests.
  - Secret scanning (local pass).

If Tier A fails, the commit or push is aborted.

### Tier B (Remote) Gates

Tier B Gates execute on the remote CI server triggered by pull requests or pushes to integration branches.

* **Velocity Rule**: Tier B favors comprehensiveness over speed. It may take considerably longer than Tier A.
* **Scope**:
  - Re-execution of Tier A to ensure local gates were not bypassed.
  - Full application build tests.
  - Container image builds.
  - Exhaustive integration and end-to-end tests requiring database spin-ups.
  - Deep dependency auditing and SAST.

Tier B serves as the ultimate arbiter of code quality. A failure in Tier B blocks the merge, regardless of local state.
