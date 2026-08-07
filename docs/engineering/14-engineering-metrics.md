# Engineering Metrics

**Document ID**: ENG-015
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
This document establishes the definitions and collection methods for engineering health metrics. It ensures that code quality, security, and repository drift are measurable and visible.

## Scope
- Definitions of key engineering metrics.
- Measurement origination and ownership.

## Out of Scope
- Hardcoded numerical thresholds (these are dynamic operational policies).
- Business metrics or analytics.

## References
- [ENG-004: Quality Gates](./03-quality-gates.md)
- [ENG-006: Security Pipeline](./05-security-pipeline.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of Engineering Metrics. |

---

## 1. Metric Definitions

Metrics must be automatically gathered. Manual metric tracking is prohibited.

### Type Safety
* **What is measured**: Percentage of codebase covered by strict static typing versus explicit or implicit `any` usage.
* **Who owns the metric**: Lead Frontend & Backend Engineers.
* **How it is measured**: Executed via the TypeScript compiler API during the CI integration phase.
* **Where measurement originates**: Tier B Quality Gates.

### Linter Compliance
* **What is measured**: The volume of bypassed lint rules (e.g., `eslint-disable` comments).
* **Who owns the metric**: The development team.
* **How it is measured**: Automated AST scanning via the primary linting tool.
* **Where measurement originates**: Tier B Quality Gates.

### Test Coverage
* **What is measured**: The percentage of source lines and logic branches covered by automated tests.
* **Who owns the metric**: Quality Assurance / Engineering.
* **How it is measured**: Coverage instrumentation tools during test execution.
* **Where measurement originates**: CI Pipeline.

### Security Vulnerabilities
* **What is measured**: Count of active CVEs within the dependency tree, categorized by severity.
* **Who owns the metric**: Principal Software Architect.
* **How it is measured**: Dependency auditing tools.
* **Where measurement originates**: Security Pipeline (ENG-006).

### Repository Drift
* **What is measured**: The volume of undocumented endpoints, database columns, or unreferenced components relative to the canonical documentation.
* **Who owns the metric**: Principal Software Architect.
* **How it is measured**: Scripted cross-referencing between AST outputs and markdown documentation schemas.
* **Where measurement originates**: Scheduled weekly cron jobs in CI.

### Technical Debt (Dead Code)
* **What is measured**: The volume of unused exports, unreachable routes, or stale feature flags.
* **Who owns the metric**: Engineering.
* **How it is measured**: Dead code elimination and tree-shaking analysis tools.
* **Where measurement originates**: Build Pipeline.
