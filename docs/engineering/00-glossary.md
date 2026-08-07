# Glossary

**Document ID**: ENG-001
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
This document provides the single, canonical dictionary for all engineering terminology used within the NST Events Engineering Handbook. Other documents must reference these definitions rather than redefining terms.

## Scope
- Core engineering definitions.
- Workflow and architectural terminology.

## Out of Scope
- Product, marketing, or business terminology.
- Temporary or feature-specific definitions.

## References
- [ENG-000: README](./README.md)
- [ENG-017: Documentation Quality Standard](./16-documentation-quality-standard.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of the Glossary. |

---

## Canonical Definitions

### Architecture Decision Record (ADR)
A document that captures an important architectural decision made along with its context and consequences.

### Architecture
The fundamental structural organization of the repository, including its components, their relationships to each other and to the environment, and the principles guiding its design and evolution.

### Authority
The precedence level of a document. If a conflict occurs, the document with higher authority invalidates the other. 

### Certification
The final formal confirmation that an implemented feature or system matches the required architecture, passes all quality gates, and is ready for production.

### Frozen Document
A document whose status is locked. It represents immutable historical records (like ADRs) or finalized API contracts that cannot be changed without initiating a version bump or replacement workflow.

### Gate (or Quality Gate)
An automated enforcement mechanism (local or remote) that prevents code from progressing to the next stage if specific criteria (lint, typecheck, tests) are not met.

### Informative
Information provided for context, guidance, or examples. Informative statements are non-binding and do not mandate behavior.

### Invariant
A constitutional truth or rule within the repository that must never be broken under any circumstances (e.g., "Packages never import apps").

### Normative
Requirements that must be followed strictly to comply with the standard. Normative statements are binding rules.

### Pipeline
An automated sequence of operations that code progresses through (e.g., CI/CD Pipeline, Security Pipeline) consisting of multiple sequential Stages.

### Repository Drift
The state where the actual implementation code diverges from the canonical documentation or architecture. Repository drift is strictly prohibited.

### Review
A manual evaluation process, either of code (Code Review) or architecture/documentation (Documentation Review), required before advancement.

### Stage
A distinct segment within a Pipeline (e.g., Build Stage, Test Stage).

### Validation
The process of executing quality gates or checks to ensure code complies with engineering standards and invariants.
