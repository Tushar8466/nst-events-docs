# [ENG-ADR-002] Conventional Commits

**Document ID**: ENG-ADR-002
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
**Related ADRs**: None

## Status
Accepted

## Context
A chaotic and unstandardized commit history makes it impossible to automatically generate semantic version bumps or readable changelogs. It also obscures the intent of historical changes when debugging regressions.

## Decision
We will strictly enforce the Conventional Commits v1.0.0 specification across the entire repository. Every commit message must adhere to the format `<type>(<scope>): <subject>`.

Valid types are restricted to: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, and `revert`. 

## Alternatives Considered
- **Free-form commit messages**: Relies on human discipline. Rejected because it precludes automation and degrades over time.
- **Squash-and-Merge only without validation**: Ensures a clean `main` history but loses granular context during complex PR reviews.

## Consequences
- **Positive**: Enables fully automated `CHANGELOG.md` generation. Removes subjectivity from commit formatting. Enhances history grepability.
- **Negative**: Introduces a minor learning curve for new developers. Requires local Git hooks (Tier A) to enforce the formatting before the commit is created.

## Related Documents
[ENG-002](../01-engineering-standards.md), [ENG-007](../06-release-process.md)
