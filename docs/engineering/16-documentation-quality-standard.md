# Documentation Quality Standard

**Document ID**: ENG-017
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
This document establishes the rigorous qualitative and formatting standards required for all documentation in the NST Events platform. It ensures consistency, readability, and long-term maintainability of canonical texts.

## Scope
- RFC 2119 terminology enforcement.
- Markdown and structure conventions.
- Diagram, code block, and cross-reference standards.

## Out of Scope
- Documentation structural architecture (see [ENG-010: Documentation Architecture](./09-documentation-architecture.md)).

## References
- [ENG-010: Documentation Architecture](./09-documentation-architecture.md)
- [ENG-001: Glossary](./00-glossary.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of the Documentation Quality Standard. |

---

## 1. RFC 2119 Terminology

To avoid ambiguity, normative documents must adhere to RFC 2119 definitions for requirement levels:
* **MUST** / **REQUIRED** / **SHALL**: Absolute requirements.
* **MUST NOT** / **SHALL NOT**: Absolute prohibitions.
* **SHOULD** / **RECOMMENDED**: Valid reasons may exist in particular circumstances to ignore a particular item, but the full implications must be understood.
* **SHOULD NOT** / **NOT RECOMMENDED**: Valid reasons may exist when particular behavior is acceptable, but it is generally discouraged.
* **MAY** / **OPTIONAL**: Truly optional choices.

## 2. Markdown Conventions

* **Format**: GitHub Flavored Markdown (GFM) strictly.
* **Heading Hierarchy**: 
  * Documents must have exactly one H1 (`#`) title at the top.
  * Subsequent headings must flow sequentially (H2 `##`, then H3 `###`). Skipping heading levels is prohibited.
* **Naming**: File names must be `kebab-case.md`. Numbered prefixes (e.g., `01-`) are permitted for strict ordering within a directory.

## 3. Cross-References

* **Stable Identifiers**: Whenever referencing an engineering document, use the format `[Title (ENG-XXX)](./00-glossary.md)` or `[ENG-XXX](./00-glossary.md)`.
* **Prohibitions**: Do not use ambiguous phrases like "see the other document" or "refer to the security docs". Provide exact, relative links.

## 4. Diagram Standards

* **Tooling**: Diagrams MUST be written in Mermaid.js code blocks directly within the markdown. Binary images (`.png`, `.jpg`) are strictly prohibited for architectural diagrams to ensure they remain editable and version-controlled.
* **Simplicity**: Do not overcrowd diagrams. If a diagram requires more than 15 distinct nodes, break it into two smaller, focused diagrams.

## 5. Code Block Rules

* **Syntax Highlighting**: All fenced code blocks MUST include a language identifier (e.g., ````typescript ````).
* **Context**: Avoid massive, multi-page code blocks. Isolate the specific lines required to demonstrate the concept.

## 6. Document Extension vs. Creation

* **Extension**: If a new concept fits logically under the `Scope` of an existing document, it MUST be appended as a new section.
* **Creation**: A new document SHALL only be created if the concept fundamentally falls outside the `Out of Scope` definition of all existing relevant documents. Unnecessary fragmentation is considered a documentation defect.
