# Repository Evolution Policy

**Document ID**: ENG-018
**Version**: 1.0.0
**Status**: Active
**Authority Level**: Normative
**Document Type**: Policy
**Owner**: Engineering
**Supersedes**: None
**Last Reviewed**: 2026-08-08
**Next Review**: 2027-02-08
**Review Cadence**: 6 Months

---

## Purpose

The Engineering Handbook serves as the long-term constitution of the NST Events repository. To maintain its integrity, authority, and utility, the repository governance requires controlled evolution. This document prevents documentation drift and defines the governance model for updating the engineering constraints, policies, and architecture rather than defining implementation details.

---

## Scope

This policy governs the following documentation assets:
- Engineering Handbook (ENG-XXX documents)
- Architecture Decision Records (ENG-ADR-XXX)
- Repository Standards
- Canonical Architecture references
- Repository policies

---

## Out of Scope

The following areas are excluded from this governance document and should be managed via standard pull request workflows:
- Application implementation
- API behavior
- Database schema
- Infrastructure implementation

---

## Repository Evolution Principles

- **Prefer updating existing documents over creating new ones**: Keep related context centralized.
- **Never duplicate canonical information**: A concept must only be defined in one place.
- **One concept → one owner document**: Responsibility for a rule or definition must reside entirely in a single document.
- **Stable document identifiers are permanent**: An `ENG-XXX` or `ENG-ADR-XXX` ID must never be reused or reassigned.
- **Documentation evolves slower than implementation**: Documentation represents constitutional law, while implementation details change frequently.
- **Historical context must be preserved**: Archiving and deprecating is preferred over silently overwriting major decisions.

---

## New Document Policy

A new handbook document **SHOULD** be created when introducing a:
- New engineering domain (e.g., a fundamentally new infrastructure layer or platform capability).
- New governance concern that cannot logically fit into existing policies.
- New permanent architectural responsibility (e.g., establishing a dedicated AI agent workflow).

A new handbook document **MUST NOT** be created to document:
- A specific feature implementation.
- A temporary workaround or a transient state.
- Documentation that overlaps significantly with an existing domain.

---

## Existing Document Modification Policy

- **When to update**: To clarify ambiguity, fix errors, or reflect approved evolution of a concept without changing its fundamental purpose.
- **When to extend**: To add new rules or sections that fall under the existing document's domain.
- **When to split**: When a document grows too large or encompasses multiple distinct domains that warrant separate ownership. The original document must link to the newly split documents.
- **When to merge**: When two documents cover the same domain or have high duplication. One document absorbs the other, and the absorbed document is marked as Deprecated, pointing to the survivor.
- **When to archive**: When a policy or domain is completely removed from the repository context.

---

## ADR Policy

An Architecture Decision Record (ADR) is **REQUIRED** for:
- Architectural decisions (e.g., selecting a new core pattern).
- Long-term tradeoffs with significant impact.
- Repository governance changes (e.g., altering the CI pipeline structure).
- Technology adoption (e.g., adopting a new primary framework).
- Breaking engineering policy defined in the handbook.

An ADR is **NOT REQUIRED** for:
- Refactoring internal implementations that adhere to existing architecture.
- Routine dependency updates (unless switching major vendors/technologies).
- Feature implementations that follow established canonical architecture.

---

## Document Lifecycle

Documents transition through the following states:

1. **Draft**: Initial creation, under active development.
2. **Review**: Submitted for architectural review and approval.
3. **Active**: Approved, authoritative, and enforced.
4. **Frozen**: The document represents a foundational policy that is highly resistant to change (e.g., the core handbook constitution).
5. **Deprecated**: The document is no longer the recommended standard and will be phased out. It must point to a replacement.
6. **Archived**: The document is historical and no longer enforced.

---

## Ownership

- **Document Owner**: Typically "Engineering" or a specific engineering domain team. They are responsible for the accuracy of the document.
- **Reviewer**: Peers within the engineering team who validate the content.
- **Approval Authority**: The Principal Software Architect or equivalent technical leadership.
- **Repository Architect Responsibilities**: Ensure all changes align with the Repository Evolution Principles, maintain the document map, and approve PRs modifying Normative documents.

---

## Versioning Policy

Documentation follows Semantic Versioning for policy changes:
- **Major (X.y.z)**: Significant governance changes, new invariant rules, or major structural restructuring.
- **Minor (x.Y.z)**: New sections added, rules extended, or minor policy adjustments that are backwards-compatible.
- **Patch (x.y.Z)**: Typo fixes, broken link repairs, formatting improvements, or metadata updates.

---

## Deprecation Policy

When a document becomes deprecated:
- Its Status metadata must be updated to **Deprecated**.
- A clear banner must be placed at the top of the document indicating it is deprecated and referencing the replacement `ENG-XXX` document.
- The document identifier remains reserved.
- Any new documentation must reference the replacement document.

---

## Cross-reference Policy

- **ENG-XXX identifiers**: All cross-references to handbook policies must use the stable ID (e.g., `ENG-001`), not just the title.
- **No filename-based references**: Links should use relative paths but the textual representation must emphasize the concept or ID, not the file name (e.g., `[Glossary (ENG-001)](./00-glossary.md)`).
- **No duplicate concepts**: Do not redefine concepts from other documents. Link to the canonical owner document instead.

---

## Review Cadence

- **Review Frequency**: Documents must define a `Review Cadence` (e.g., 6 Months, 1 Year). The `Next Review` date must be updated upon completion of a review.
- **Emergency Review Process**: Can be triggered by the Repository Architect if an incident or critical architectural failure reveals a flaw in the current documentation.

---

## Repository Constitution Protection

The following foundational documents form the immutable core of the repository's governance. Any modification to these documents requires explicit architectural approval (and typically an ADR):
- ENG-000 (README / Document Map)
- ENG-001 (Glossary)
- ENG-008 (AI Development Workflow)
- ENG-009 (Tooling Contract)
- ENG-010 (Documentation Architecture)
- ENG-011 (Repository Invariants)
- ENG-018 (Repository Evolution Policy)

---

## References

- [ENG-000: Engineering Handbook Index](./README.md)
- [ENG-010: Documentation Architecture](./09-documentation-architecture.md)

---

## Version History

| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial release establishing the repository evolution policy. |
