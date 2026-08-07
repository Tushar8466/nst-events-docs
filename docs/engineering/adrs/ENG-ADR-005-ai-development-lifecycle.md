# [ENG-ADR-005] AI Development Lifecycle

**Document ID**: ENG-ADR-005
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
AI-assisted coding agents dramatically increase implementation velocity. However, without strict architectural guardrails, autonomous agents tend to hallucinate implementations, invent unapproved architectural patterns, duplicate existing business logic, and cause severe repository drift.

## Decision
We mandate a rigid 11-step lifecycle for all AI-driven implementations, governed by an immutable AI Constitution.

Crucially, AI agents are prohibited from modifying code without first conducting comprehensive repository and documentation audits. Furthermore, AI agents must present an Implementation Plan and secure explicit human authorization before executing code changes. All AI outputs must conclude with an independent audit and Production Certification.

## Alternatives Considered
- **Unrestricted Agent Access**: Allowing AI to read issue tickets and commit directly to `main` or feature branches. Rejected due to an unacceptable risk of architectural drift, undocumented endpoints, and security vulnerabilities.
- **AI as a pure autocomplete (Copilot only)**: Rejecting autonomous agent workflows entirely. Rejected because it abandons massive productivity gains for complex refactors and boilerplate generation.

## Consequences
- **Positive**: Ensures that AI outputs perfectly align with canonical documentation. Prevents the accumulation of "black box" code that human maintainers do not understand.
- **Negative**: Introduces friction into the AI workflow, requiring the agent to pause for human consensus and perform potentially redundant audits.

## Related Documents
[ENG-008](../07-ai-development-workflow.md), [ENG-011](../10-repository-invariants.md)
