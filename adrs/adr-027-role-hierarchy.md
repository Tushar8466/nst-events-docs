# ADR-027: Role hierarchy

## Status
Superseded by two-tier model in 04-enums.md (global_role + club_role)

## Context
We need a formalized permission ladder to match the real-world institution.

## Decision
Implement hierarchy: Platform Admin > Faculty Admin > Faculty Mentor > Club Admin > Core Member > Member > Student.

## Consequences
Positive: Reflects actual campus operations accurately.
Negative: Highly complex authorization logic.

## Alternatives Considered
* **Flat roles**: Rejected as it doesn't allow for delegated authority.

## Future Impact
Serves as the foundation for all platform access control.
