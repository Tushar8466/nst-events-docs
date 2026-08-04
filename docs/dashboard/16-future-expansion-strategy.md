# Future Expansion Strategy

## Domain Injection Pattern
The Single Dashboard Shell is designed to accept new modules gracefully without breaking the Sidebar Architecture.

## Future Modules
* **Certificates**: Automated generation and tracking.
* **Recruitment**: Managing club intakes and interviews.
* **Media Galleries**: Event photo management.
* **Multi-Campus**: Fully realizing multi-campus logic (note: docs refer to candidate field names as `active_campus_id` in UI/dashboard contexts and `tenant_id` in database architecture; exact schema naming is TBD and must be formalized via an ADR before implementation).
