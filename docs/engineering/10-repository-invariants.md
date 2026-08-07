# Repository Invariants

**Document ID**: ENG-011
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
This document is the constitution of the repository. It defines architectural truths and invariant rules that must almost never change. 

## Scope
- Fundamental architectural principles.
- Codebase integrity rules.
- Documentation authority rules.

## Out of Scope
- Implementation specifics (e.g., database authorization strategies, specific framework ownership).
- Tooling (see [ENG-009: Tooling Contract](./08-tooling-contract.md)).

## References
- [ENG-008: AI Development Workflow](./07-ai-development-workflow.md)
- [ENG-010: Documentation Architecture](./09-documentation-architecture.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of Repository Invariants. |

---

## 1. Architecture

**Invariant: Isolation of Concerns**
* **Category**: Architecture
* **Criticality**: Critical
* **Authority**: Normative
* **Rationale**: Prevents monolithic entanglement. Packages (e.g., UI libraries, shared utilities) must never import from Apps (e.g., the API server, mobile app). Dependencies flow strictly downward.

**Invariant: Separation of Presentation and Logic**
* **Category**: Architecture
* **Criticality**: High
* **Authority**: Normative
* **Rationale**: The User Interface owns presentation solely. Business invariants and state enforcement belong to the backend canonical architecture. 

**Invariant: Explicit Optimism**
* **Category**: Architecture
* **Criticality**: Medium
* **Authority**: Normative
* **Rationale**: Optimistic UI/business logic is prohibited unless explicitly designed and documented via an ADR. Strict synchronous consistency is the default.

## 2. Documentation

**Invariant: Documentation is Authoritative**
* **Category**: Documentation
* **Criticality**: Critical
* **Authority**: Normative
* **Rationale**: Documentation takes precedence over implementation intent. If code behaves differently than the canonical documentation, the code is considered broken.

**Invariant: Scope of Engineering Documentation**
* **Category**: Documentation
* **Criticality**: High
* **Authority**: Normative
* **Rationale**: Engineering documentation (this handbook) governs *how* work is done. It must never redefine or contradict application architecture or product requirements.

## 3. Repository

**Invariant: No Repository Drift**
* **Category**: Repository
* **Criticality**: Critical
* **Authority**: Normative
* **Rationale**: Source code must perfectly reflect the documentation and ADRs. Drift destroys trust and breaks the AI-assisted workflow.

**Invariant: No Dead Code**
* **Category**: Repository
* **Criticality**: High
* **Authority**: Normative
* **Rationale**: Unused endpoints, components, or variables increase maintenance burden and security surface area. They must be removed immediately upon deprecation.

## 4. Quality

**Invariant: Independent Auditing**
* **Category**: Quality
* **Criticality**: High
* **Authority**: Normative
* **Rationale**: Every implementation phase, especially AI-driven workflows, must begin with a repository audit and conclude with an independent audit against the initial plan.

## 5. Security

**Invariant: Zero-Trust Boundaries**
* **Category**: Security
* **Criticality**: Critical
* **Authority**: Normative
* **Rationale**: Internal services must authenticate and authorize requests just as external clients do.

## 6. Git

**Invariant: Protected Mainline**
* **Category**: Git
* **Criticality**: Critical
* **Authority**: Normative
* **Rationale**: The `main` integration branch must never accept direct pushes. All code must traverse the CI Pipeline Quality Gates.

## 7. Production

**Invariant: No Placeholder Logic**
* **Category**: Production
* **Criticality**: Critical
* **Authority**: Normative
* **Rationale**: Production code must not contain placeholder implementations, mocked security, or bypassed logic (e.g., `// TODO: add auth`).

**Invariant: Architecture Changes Require ADRs**
* **Category**: Production
* **Criticality**: High
* **Authority**: Normative
* **Rationale**: Fundamental shifts in data flow, storage, or external contracts must be captured in an ADR to preserve the historical decision-making context.
