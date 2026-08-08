# [ENG-ADR-003] Pre-Push Validation

**Document ID**: ENG-ADR-003
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
**Related ADRs**: [ENG-ADR-001](./ENG-ADR-001-local-first-quality-gates.md)

## Status
Accepted

## Context
While pre-commit hooks catch styling and basic linting issues, deeper invariant violations (such as committing plain-text secrets, violating architectural import boundaries, or failing the test suite) can still be accidentally pushed to the remote repository if validation is bypassed or skipped during the commit phase.

## Decision
We mandate a hard boundary at the Git Push lifecycle event. A Husky Pre-Push execution hook must be configured to run the canonical `pnpm validate:local` command, representing the complete Tier A Quality Gate suite.

If this suite fails, the push is aborted locally, and the code never touches the network.

## Alternatives Considered
- **Pre-commit only**: Running the full Tier A suite on every single commit. Rejected because it slows down developers who commit frequently as a save mechanism.
- **Server-side pre-receive hooks**: Rejected as GitHub/hosted Git providers often do not support custom pre-receive hooks, and the feedback loop is slower.

## Consequences
- **Positive**: Protects the remote repository from obvious breakages and secret leaks. Allows developers to commit rapidly (bypassing pre-commit if needed) while ensuring the network boundary is secure.
- **Negative**: Pushing code takes slightly longer (up to 3 minutes).

## Related Documents
[ENG-003](../02-git-workflow.md), [ENG-004](../03-quality-gates.md)
