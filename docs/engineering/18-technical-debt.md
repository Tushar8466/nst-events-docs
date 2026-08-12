# Technical Debt Registry

## OPEN TECHNICAL DEBT — NOT FIXED

### `cancel_registration` Waitlist Promotion Issue
- **Current Behavior**: When a user cancels their registration, the system does not automatically promote the next waitlisted user to `REGISTERED` status due to concurrency and state-management complexities.
- **Impact**: Waitlisted users are not automatically moved up when spots open up, requiring manual intervention or wait for batch jobs.
- **Why it is Deferred**: It was intentionally NOT fixed during the frontend contract alignment phase (Phase 18/19) because it requires a safe, transactionally sound background worker process to handle the cascading state changes without race conditions.
- **Intended Future Remediation**: We plan to implement a robust background job via the `nst_worker` queue to handle waitlist promotions asynchronously, ensuring that database locks and race conditions are mitigated.
