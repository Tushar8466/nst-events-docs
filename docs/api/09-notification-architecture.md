# Notification Architecture

## Queue Flow
1. **Trigger**: The Node.js Notification Producer Service (invoked by API handlers after transactions commit) inserts a message into the `notifications` and `notification_jobs` tables atomically via a Prisma Interactive Transaction.
2. **Poll**: The `nst-worker` Kubernetes Deployment polls the `notification_jobs` table every 5 seconds (batch of 100 messages) using `FOR UPDATE SKIP LOCKED`. See `docs/api/13-worker-deployment.md` for the full deployment model.
3. **Delivery**: The worker calls the Expo Push API.
4. **Finalization**: Success/Fail states are written back to the `notifications` and `notification_jobs` tables via Prisma.

## Idempotency Contract
To prevent duplicate notifications from being generated for the same logical event (e.g., a user tapping "Approve" twice quickly, or an RPC retrying), all queue payloads must include an `idempotency_key`.

**Canonical Key Generation:**
```text
SHA256(notification_type + user_id + primary_entity_id + logical_event)
```

**Rules:**
- **Duplicate Detection**: If a message with an identical idempotency key is processed within the same context, it is dropped as a duplicate.
- **Intentional Reuse**: If an action is logically identical (e.g., "Waitlist Promoted for Event A"), the key remains the same to prevent a second alert.
- **New Keys**: A new key MUST be generated if the logical event represents a distinctly new occurrence (e.g., a recurring weekly meeting reminder should append the timestamp or week number to the `logical_event` string).

**Example:**
For a waitlist promotion: `SHA256("WAITLIST_PROMOTED" + "user-123" + "event-456" + "promotion-event")`

## Retry Architecture
The notification system uses an **application-controlled exponential retry strategy** over the native PostgreSQL queue.

- The worker claims messages using `SELECT ... FOR UPDATE SKIP LOCKED`.
- The worker is explicitly responsible for updating the `available_at`, `status`, and `attempt_count` fields upon failure.

**Transient Failures** (e.g., network timeout, HTTP 5xx, Expo rate limiting):
- **Attempt 1**: Retry after 30 seconds (`available_at = now() + interval '30 seconds'`)
- **Attempt 2**: Retry after 2 minutes (`available_at = now() + interval '2 minutes'`)
- **Attempt 3**: Retry after 10 minutes (`available_at = now() + interval '10 minutes'`)
- **Attempt 4**: Mark as Dead Letter (`status = 'DEAD_LETTER'`)

**Permanent Failures** (e.g., `DeviceNotRegistered`, invalid token, malformed payload, HTTP 4xx validation failures):
- MUST NOT retry.
- Invalid push tokens are immediately deleted from the `push_tokens` table.

## NotificationJobStatus Lifecycle & Transition Rules

The queue utilizes exactly 8 states defined in the `NotificationJobStatus` enum: `PENDING`, `PROCESSING`, `WAITING_FOR_RECEIPTS`, `COMPLETED`, `RETRY_PENDING`, `FAILED`, `DEAD_LETTER`, and `ARCHIVED`.

### State Definitions
1. **PENDING**: The initial state. The job is waiting to be claimed by a worker. (`available_at <= now()`)
2. **PROCESSING**: The worker has claimed the job and locked the row (`locked_at = now()`).
3. **WAITING_FOR_RECEIPTS**: The worker successfully sent the notification and received Expo ticket IDs. The job is waiting for the receipt check. (`available_at` is set to `now() + EXPO_RECEIPT_DELAY_MINUTES`).
4. **COMPLETED**: The worker successfully verified Expo receipts or delivered without requiring a push (e.g., no tokens). Terminal state.
5. **RETRY_PENDING**: A transient failure occurred (e.g., HTTP 5xx, or RateExceeded). The job is waiting for its next retry attempt.
6. **FAILED**: A permanent failure occurred (e.g., `DeviceNotRegistered`, `MessageTooBig`, `InvalidCredentials`). Terminal state.
7. **DEAD_LETTER**: A transient failure occurred on the final attempt (Attempt 4). Requires manual review. Terminal state.
8. **ARCHIVED**: The target entity (user or event) was deleted before processing. Terminal state.

### Exact Transition Rules
- `PENDING` → `PROCESSING`
- `RETRY_PENDING` → `PROCESSING`
- `PROCESSING` → `WAITING_FOR_RECEIPTS`
- `WAITING_FOR_RECEIPTS` → `COMPLETED`
- `PROCESSING` → `RETRY_PENDING`
- `PROCESSING` → `FAILED`
- `PROCESSING` → `ARCHIVED`
- `PROCESSING` → `DEAD_LETTER`

**Admin Interventions:**
- `DEAD_LETTER` → `PENDING` (Triggered manually via `POST /admin/queue/dead-letters/:id/replay`)

There are NO other valid transitions. Messages cannot go from `COMPLETED` back to `PENDING`, nor from `FAILED` to `RETRY_PENDING`.

## Queue Administration

The operational administration layer enables Platform Admins to monitor the queue and recover from `DEAD_LETTER` states.

- **Inspection**: `GET /admin/queue/dead-letters` provides a paginated view of all `DEAD_LETTER` jobs.
- **Replay**: `POST /admin/queue/dead-letters/:id/replay` resets the job state.
  - Transactionally updates: `status` back to `PENDING`, `attempt_count` to `0`, `last_error` to `null`, `ticket_ids` to `null`, and `available_at` to `now()`.
  - Generates a canonical audit event (`Action: QUEUE_JOB_REPLAY`).
- **Monitoring**: `GET /admin/queue/monitoring` provides aggregated statistics across the `notification_jobs` table (e.g. queue depth, retry count).

## Dual-Axis Dispatch Architecture

The worker dispatches jobs using the ordered pair `(job_type, status)`. Neither field alone is sufficient to determine execution logic.
- `job_type` defines the fundamental type of work (e.g., `SEND_PUSH`). The payload remains immutable. The worker must never mutate `payload` or `job_type`.
- `status` defines the current execution stage within that job type's lifecycle.

Execution state is represented only by `status`, `ticket_ids`, `attempt_count`, and `available_at`.

### Canonical Dispatch Algorithm

```javascript
switch (job.payload.job_type) {
    case 'SEND_PUSH':
        switch(job.status) {
            case 'PENDING':
            case 'RETRY_PENDING':
                // Execute: send push notification
                executePush(job);
                break;
            case 'WAITING_FOR_RECEIPTS':
                // Execute: fetch Expo receipts (NOT send another push)
                executeReceiptPolling(job);
                break;
            default:
                throw new Error("Invalid state");
        }
        break;
}
```

### Complete Receipt Lifecycle
No secondary queue rows are created. No payload mutation occurs. No duplicate push delivery occurs.
1. `SEND_PUSH` (Payload created by producer)
2. `PENDING` (Worker claims job)
3. `PROCESSING` (Worker locks job)
4. Send notifications to Expo Push API
5. Store `ticket_ids` in the dedicated database field
6. `WAITING_FOR_RECEIPTS` (Worker waits for `EXPO_RECEIPT_DELAY_MINUTES`)
7. Fetch Expo receipts using stored `ticket_ids`
8. `COMPLETED` (Terminal state)

## Failure Matrix

| Failure Source | Retry? | Status Transition | Secondary Action |
|---|---|---|---|
| Network timeout | Yes | `RETRY_PENDING` (or `DEAD_LETTER` if max) | None |
| HTTP 429 / 5xx | Yes | `RETRY_PENDING` (or `DEAD_LETTER` if max) | None |
| DeviceNotRegistered | No | `FAILED` | Delete row from `push_tokens` |
| MessageTooBig | No | `FAILED` | None |
| InvalidCredentials | No | `FAILED` | None |
| Malformed payload | No | `FAILED` | None |
| Deleted user/event | No | `ARCHIVED` | None |
| Prisma / Database error | Yes | `RETRY_PENDING` (or `DEAD_LETTER` if max) | None |

## Canonical Worker Configuration

All worker deployments must adhere to these canonical configurations, passed via environment variables where applicable:
- `WORKER_POLL_INTERVAL_MS=5000`
- `WORKER_BATCH_SIZE=100`
- `WORKER_MAX_RETRIES=4`
- `WORKER_SHUTDOWN_TIMEOUT_MS=25000`
- `EXPO_RECEIPT_DELAY_MINUTES=15`
- `EXPO_MAX_BATCH_SIZE=100`

## Rate Limiting & Delivery Tracking
To prevent spam, clubs are limited to 2 broadcast notifications per week. Delivery tracking is handled by correlating Expo receipts back to the notification records.
