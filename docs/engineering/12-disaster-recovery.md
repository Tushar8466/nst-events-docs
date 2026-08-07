# Disaster Recovery

**Document ID**: ENG-013
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
This document codifies the emergency procedures required to restore the NST Events platform to a healthy state following catastrophic failures, preventing ad-hoc decision-making during incidents.

## Scope
- Deployment and CI pipeline failures.
- Data corruption and database recovery.
- Secret rotation and security incidents.

## Out of Scope
- Infrastructure-as-code definitions.
- Uptime SLA business agreements.

## References
- [ENG-007: Release Process](./06-release-process.md)

## Version History
| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2026-08-08 | Initial creation of the Disaster Recovery policy. |

---

## 1. Failed Deployments

If a deployment to production fails or introduces severe regressions (e.g., increased 5xx errors):
* **Action**: Initiate an immediate Rollback.
* **Rule**: Do not attempt to "fix-forward" by pushing new commits to production while it is broken. Revert the production environment to the last known healthy artifact.
* **Follow-up**: Investigate the failure locally or in staging, push a fix through the standard [Release Process (ENG-007)](./06-release-process.md).

## 2. CI Pipeline Failures

If the remote CI pipeline fails persistently on the `main` branch:
* **Action**: Halt all merges. The repository is considered locked.
* **Rule**: The offending commit must be reverted immediately to restore CI health, unless a trivial, guaranteed fix can be deployed in under 10 minutes.

## 3. Failed Database Migrations

If a schema migration fails partially during deployment:
* **Action**: Do not manually alter tables to match the expected state.
* **Rule**: Execute the down-migration (if supported) or restore the database from the immediate pre-deployment snapshot. Identify the locking or constraint issue locally.

## 4. Corrupted Databases

In the event of severe data corruption or catastrophic loss:
* **Action**: Initiate the Point-in-Time Recovery (PITR) protocol from the continuous backup storage.
* **Rule**: Recovery must target a timestamp immediately preceding the corruption event. Post-recovery, notify all stakeholders of potential data loss during the delta window.

## 5. Emergency Hotfixes

If a critical vulnerability or complete outage occurs that bypasses standard CI timelines:
* **Action**: A Principal Architect may authorize a Hotfix.
* **Rule**: The fix is applied to a hotfix branch branched directly from the current production tag, subjected to expedited Tier A testing, and deployed. It must then be backported to `main`.

## 6. Secret Rotation

In the event of a credential leak (detected by the Security Pipeline or external report):
* **Action**: Immediately revoke the compromised secret at the provider level.
* **Rule**: Generate a new secret, inject it into the secure deployment configuration, and trigger a rolling restart of all affected services (API, Workers).
* **Follow-up**: Conduct an audit of logs to determine if the leaked secret was exploited.

## 7. Production Incidents

For any P0/P1 incident:
* **Action**: Establish an incident bridge. Appoint a single Incident Commander.
* **Rule**: All actions taken during the incident must be documented in a post-mortem within 48 hours, detailing the root cause and required preventative engineering work.
