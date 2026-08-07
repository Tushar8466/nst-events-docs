# Pessimistic Registration Flow

## Philosophy
NST-Events operates on a strict **Pessimistic Registration Flow**. Because event capacity is highly competitive, the UI must never show "Registered" or optimistic success states before backend confirmation.

## Flow Sequence
1. **Tap Register**: User initiates action.
2. **Confirmation Modal**: Explicit user consent.
3. **Confirm**: User confirms.
4. **Loading State**: UI blocks further interaction and displays a spinner.
5. **`register_event` RPC**: Client awaits backend PostgreSQL transaction (which executes lock-free atomic capacity updates).
6. **Response**: Backend replies.

## Outcomes
* **REGISTERED**: Success UI.
* **WAITLISTED**: Routes to the Waitlist Flow.
* **FAILED**: Error toast (e.g., network failure, banned user).

## Team Registration (If Applicable)
* **Partial Teams**: Event capacity counts individuals, not whole teams. Teams may become partially registered.
* **Join Flow**: When a user joins a team, if `registration_count < max_capacity`, they are `REGISTERED`. If full, they individually become `WAITLISTED`. There is no whole-team waitlisting.
* **Leader Transfer**: If a team leader leaves, leadership transfers automatically to the oldest ACTIVE, `REGISTERED` member (ordered by `registered_at` ASC). `WAITLISTED` members are never eligible. If no `REGISTERED` members remain, the team and all its remaining `WAITLISTED` registrations are immediately soft-deleted.
* **Team Lifecycle Invariant**: A team MUST exist IFF at least one active registration references that team. A team is only soft-deleted when active registrations == 0.
