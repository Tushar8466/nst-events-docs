# AI Development Workflow

**Document ID**: ENG-008
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
This document codifies the mandatory workflow for all AI-assisted contributions to the NST Events repository. It ensures that AI agents operate deterministically, safely, and in alignment with canonical architecture.

## Scope
- AI Constitution and immutable rules.
- Pre-implementation auditing workflow.
- Execution and certification lifecycle.
- Failure conditions for AI workflows.

## Out of Scope
- Specific AI models or vendors.
- Prompt engineering best practices.

## References
- [ENG-010: Documentation Architecture](./09-documentation-architecture.md)
- [ENG-011: Repository Invariants](./10-repository-invariants.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of the AI Development Workflow. |

---

## 1. The AI Constitution

The AI Constitution defines immutable rules for AI-assisted development. Under no circumstances may an AI agent or human operator authorize the violation of these rules.

* **Never Invent Architecture**: AI agents must implement features based exclusively on existing canonical documentation. If a feature lacks documentation, the AI must halt and request architectural design.
* **Documentation Precedes Implementation**: All changes to business logic, API contracts, or database schemas must be codified in documentation *before* code is modified.
* **Repository Audit Before Implementation**: Every task must begin with a comprehensive audit of the existing codebase and canonical documentation.
* **No Undocumented Features**: Code that is not explicitly described in the canonical documentation is prohibited.
* **No Duplicated Business Logic**: AI agents must seek out and reuse existing abstractions, utilities, and components before writing new logic.
* **No Repository Drift**: The source code must perfectly mirror the documentation. Divergence is considered a critical failure.
* **No Production Placeholders**: `TODO` comments, mocked endpoints, or placeholder data are strictly prohibited in production code unless they represent explicitly deferred features documented via an ADR.
* **Independent Audit**: Every implementation concludes with an independent audit and production certification before merge.

## 2. Development Lifecycle

The mandatory lifecycle for AI-assisted contributions follows a strict 11-step process:

1. **Repository Audit**: Scan existing code to understand current state.
2. **Documentation Audit**: Verify the target feature's architecture in the canonical documentation.
3. **Dependency Audit**: Ensure necessary internal/external packages are present.
4. **Implementation Plan**: Draft an `implementation_plan.md` artifact detailing proposed changes.
5. **Approval Rules**: The user/architect must explicitly approve the implementation plan before any code is modified.
6. **Implementation**: Execute the approved plan.
7. **Independent Audit**: Verify the implemented code against the initial plan.
8. **Repository Drift Audit**: Ensure the final codebase does not contradict any canonical documentation.
9. **Production Certification Workflow**: Generate a final artifact verifying that the code meets production standards.
10. **Quality Gates**: Ensure Tier A (Local) Quality Gates pass successfully.
11. **Merge**: Submit the Pull Request for remote CI validation and human review.

## 3. Failure Conditions

An AI workflow must be aborted and reverted if any of the following failure conditions are met:
* The AI modifies a file not specified in the approved Implementation Plan.
* The AI introduces a new tool or dependency without explicitly updating [ENG-009](./08-tooling-contract.md).
* The Tier A Quality Gates fail and cannot be resolved within two retry attempts.
* The AI hallucinates an API endpoint, database column, or architectural pattern not present in the repository.
