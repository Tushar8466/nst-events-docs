# [ENG-ADR-001] Local-first Quality Gates

**Document ID**: ENG-ADR-001
**Version**: 1.0.0
**Status**: Accepted
**Authority Level**: Normative
**Document Type**: Architecture Decision Record
**Owner**: Engineering
**Last Reviewed**: 2026-08-08
**Next Review**: 2027-02-08
**Review Cadence**: 6 Months
**Date**: 2026-08-08
**Decision Makers**: Principal Software Architect
**Supersedes**: None
**Related ADRs**: [ENG-ADR-004](./ENG-ADR-004-ci-verification.md)

## Status
Accepted

## Context
Developers experience significant velocity degradation when waiting for remote Continuous Integration (CI) pipelines to validate basic code formatting, typing, and isolated logic errors. However, pushing unchecked code to the remote repository pollutes the branch history and clogs shared CI runners.

## Decision
We will adopt a "Local-first" Quality Gate model (Tier A). Developers MUST execute lightning-fast validation (linting, formatting, type-checking, fast unit tests) directly on their local machines prior to committing or pushing code. 

To preserve developer velocity, Tier A gates MUST NOT include heavy operations such as Docker container builds, full end-to-end integration tests, or extensive database spin-ups. These heavy operations are strictly deferred to Tier B (Remote).

## Alternatives Considered
- **CI-Only Validation**: Relying purely on GitHub Actions for all validation. Rejected due to the feedback loop exceeding 10 minutes, severely impacting developer flow.
- **Full Local Emulation**: Forcing developers to run the entire integration suite locally via Docker before pushing. Rejected as it degrades local machine performance and takes too long.

## Consequences
- **Positive**: Immediate developer feedback. Reduced CI billable minutes. Cleaner remote commit history.
- **Negative**: Requires strict maintenance of local tooling configurations (e.g., Husky, Lefthook) across the engineering team to prevent drift between local environments and CI environments.

## Related Documents
[ENG-004](../03-quality-gates.md)
