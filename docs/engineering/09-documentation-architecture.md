# Documentation Architecture

**Document ID**: ENG-010
**Version**: 1.0.0
**Status**: Frozen
**Authority Level**: Normative
**Document Type**: Architecture
**Owner**: Engineering
**Supersedes**: None
**Last Reviewed**: 2026-08-08
**Next Review**: 2027-02-08
**Review Cadence**: 6 Months

## Purpose
This document defines the structural architecture of the documentation itself. It establishes rules for canonical sources, cross-referencing, precedence, and modification workflows to prevent fragmentation and repository drift.

## Scope
- Canonical source rules.
- Duplicate definition prevention.
- Frozen documentation policy.

## Out of Scope
- Documentation formatting conventions (see [ENG-017: Documentation Quality Standard](./16-documentation-quality-standard.md)).

## References
- [ENG-017: Documentation Quality Standard](./16-documentation-quality-standard.md)
- [ENG-001: Glossary](./00-glossary.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of Documentation Architecture. |

---

## 1. Canonical Source Rules

NST Events operates on a strict "Single Source of Truth" policy.
* **Implementation Intent**: Documentation is the authoritative source of implementation intent. If the code contradicts the documentation, the code is considered a bug.
* **Engineering Principles**: The Engineering Handbook (`docs/engineering/`) is the sole authority on *how* engineering is performed.
* **Product Architecture**: The `docs/` directory is the sole authority on *what* is built (Database Schemas, API Contracts, UI components).

## 2. Document Precedence

If two canonical sources appear to conflict, the following precedence applies (Highest to Lowest):
1. **Security & Cryptographic Standards**
2. **Database Schema Contracts**
3. **API Contracts**
4. **Engineering Handbook (Normative Rules)**
5. **Architecture Decision Records (ADRs)**
6. **Implementation Source Code**
7. **Informative Guides / Tutorials**

## 3. Duplicate Definition Prevention

Duplication of knowledge breeds inconsistency.
* A concept, rule, or architectural pattern must be defined in exactly **one** place.
* Every subsequent mention of that concept must link back to its canonical source using stable Document IDs (e.g., `[Reference](./00-glossary.md)`).
* The [Glossary (ENG-001)](./00-glossary.md) is the absolute authority for terminology.

## 4. Frozen Documentation Policy

Documents marked with the Status `Frozen` represent finalized, approved contracts that have been merged into the canonical repository.
* **Immutability**: Frozen documents cannot be edited to alter their fundamental architectural intent.
* **Modification Workflow**: To change a Frozen document, an engineer must:
  1. Author an Architecture Decision Record (ADR) detailing the proposed change.
  2. Achieve consensus and merge the ADR.
  3. Bump the Version number of the Frozen document and apply the changes in a synchronized Pull Request alongside the code implementation.

## 5. Documentation Update Workflow

Documentation must be updated atomically with the code it describes.
* A Pull Request modifying business logic is invalid if the corresponding canonical documentation is not updated in the same PR.
* AI agents are strictly prohibited from generating implementations without first auditing and updating documentation.
