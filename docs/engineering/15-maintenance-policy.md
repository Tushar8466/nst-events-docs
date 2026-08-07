# Maintenance Policy

**Document ID**: ENG-016
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
This document codifies the procedures for maintaining the lifecycle of the Engineering Handbook itself and other canonical documents. It ensures that documentation does not silently rot over time.

## Scope
- Document review cadences.
- Ownership transfer.
- Deprecation and archival workflows.

## Out of Scope
- Code maintenance (see [ENG-014: Dependency Management](./13-dependency-management.md)).

## References
- [ENG-010: Documentation Architecture](./09-documentation-architecture.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of the Maintenance Policy. |

---

## 1. Review Cadence

Every canonical document must carry a explicitly declared `Review Cadence` in its metadata header (typically 6 Months or 1 Year).
* **Trigger**: CI automation will open an issue when a document passes its `Next Review` date.
* **Action**: The designated `Owner` must audit the document against current repository practices.
* **Resolution**: The owner either bumps the `Last Reviewed` / `Next Review` dates via a PR, or initiates an update via the ADR workflow if the document requires changes.

## 2. Ownership Transfer

Ownership of a document must be tied to a role or team (e.g., "Principal Software Architect", "Engineering"), not a specific individual's name.
* If a team structure changes, a bulk update PR must be executed to reassign ownership of affected documents to the newly formed team or role.

## 3. Document Deprecation

When an engineering practice or architecture is slated for removal but is still actively used in legacy parts of the codebase:
* **Action**: The document's `Status` must be changed to `Deprecated`.
* **Warning**: A high-visibility warning block must be added to the top of the document pointing to the superseding practice or document (via `Supersedes`).

## 4. Archival Policy

When a document's contents are entirely removed from the active codebase and no longer apply to any systems:
* **Action**: Do not delete the file. Change the `Status` to `Archived`.
* **Rationale**: Archived documents provide critical historical context for future engineers attempting to understand why previous systems were removed.

## 5. Versioning Strategy

Documentation versioning follows standard major/minor semantic rules, independent of the software release version.
* **Major**: A fundamental rewrite of the document's rules or scope.
* **Minor**: Clarifications, formatting updates, or small additions that do not change the normative requirements.
