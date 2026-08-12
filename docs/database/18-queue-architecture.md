# Queue Architecture

## native queue Integration
We leverage the `native queue` extension to keep our queue collocated with our relational data. This allows transactional enqueueing (e.g., generating an event and queueing its notification in the same transaction).

## Notification Processing
Background workers pull from the queue to dispatch push notifications to external services.

## Strategies
* **Retry Strategy**: Exponential backoff up to 3 attempts.
* **Dead Letter Strategy**: Failed messages are moved to a dead letter queue (DLQ) for manual inspection by Platform Admins.
* **Autovacuum Tuning**: Queue tables experience high churn. Aggressive autovacuum settings are applied to the `native queue` schema to prevent bloat.


## Current Implementation (Phase 19)
- **`nst_worker` Role**: The queue operates under a dedicated PostgreSQL role `nst_worker` with isolated schema access.
- **Queue Claiming (`SKIP LOCKED`)**: Workers claim jobs using `SELECT ... FOR UPDATE SKIP LOCKED` to prevent concurrency issues and lock contention.
- **Lease Recovery**: Stalled jobs (where `locked_at` exceeds the timeout) are automatically recovered and retried.
- **Delivery Semantics**: The system guarantees **at-least-once** delivery. It does NOT guarantee exactly-once delivery; idempotency must be handled by the consumer.
- **Payload Validation**: Worker payloads are strictly validated against Zod schemas upon extraction from the database.

