# Data Contract

## User
- `id` (string, non-null): Unique identifier.
- `email` (string, non-null): User's email.
- `full_name` (string, nullable): User's full name.
- `global_role` (enum, non-null): STUDENT, FACULTY_ADMIN, PLATFORM_ADMIN
- Source Endpoint: `GET /users/me`
- Security: Protected, Private

## Event
- `id` (string, non-null): Event ID.
- `title` (string, non-null): The event title (DO NOT USE `name`).
- `description` (string, nullable): HTML or Markdown description.
- `start_time` (string ISO, non-null): Start time.
- `end_time` (string ISO, non-null): End time.
- `location_name` (string, nullable): Location description.
- `state` (enum, non-null): DRAFT, PENDING_APPROVAL, PUBLISHED, ARCHIVED
- `max_capacity` (number, nullable): Max participants.
- `registration_count` (number, non-null): Current registration count.
- `eventClubs` (array, non-null): Array of associated clubs. Each entry contains `club_id`, `club_name`, and `is_primary`.
- Source Endpoint: `GET /v1/events/:id`
> **Note**: open/closed for registration purposes is DERIVED client-side from state == PUBLISHED && (max_capacity IS NULL OR registration_count < max_capacity) && !is_locked — there is no single backend enum for this.

## Registration
- `status` (enum, non-null): REGISTERED, WAITLISTED, CANCELLED.
- `team_id` (string, nullable): Associated team if applicable.
- Source Endpoint: `GET /v1/events/:id/my-registration`
> **Note**: "not registered" is derived client-side from a 404 response on GET /v1/events/:id/my-registration, not a real enum value.

## Team
- `id` (string, non-null): Team ID.
- `team_name` (string, non-null): Team name.
- `members` (array, non-null): Array of User objects.
- Source Endpoint: `POST /v1/events/:id/teams`

## Club
- `id` (string, non-null): Club ID.
- `name` (string, non-null): Club name.
- `description` (string, nullable): Club description.
- `banner_url` (string, nullable): Banner image URL (currently deferred).
- `status` (enum, non-null): ACTIVE, INACTIVE, DISSOLVED.
- `event_count` (number, non-null): Number of Events.
- `members` (array, non-null): Array of members (`{ user_id, role, full_name, avatar_url }`).
- Source Endpoint: `GET /clubs/:id`

## AttendanceSession
- `id` (string, non-null): Session ID.
- `event_id` (string, non-null): Event ID.
- `title` (string, non-null): Session Title.
- `start_time` (string ISO, non-null): Session start.
- `end_time` (string ISO, non-null): Session end.
- `open_at` (string ISO, non-null): Scan window open time.
- `close_at` (string ISO, non-null): Scan window close time.
- `geofence_radius` (number, non-null): Geofence radius.
- `created_by` (string, non-null): Creator ID.
- `created_at` (string ISO, non-null): Creation timestamp.
- Source Endpoint: `GET /v1/events/:id/attendance`

## AttendanceRecord
- `id` (string, non-null): Record ID.
- `session_id` (string, non-null): Associated Session ID.
- `user_id` (string, non-null): Participant User ID.
- `marked_by` (string, nullable): ID of admin/organizer who marked it.
- `marked_at` (string ISO, non-null): Timestamp of scan.
- `method` (enum, non-null): QR, MANUAL, etc.
- `status` (enum, non-null): PRESENT, ABSENT, EXCUSED.
- Source Endpoint: `GET /v1/events/:id/attendance` (Nested or Array)

## Notification
- `id` (string, non-null): Notification ID.
- `type` (string, non-null): Category.
- `title` (string, non-null): Alert title.
- `body` (string, non-null): Main text.
- `read_at` (string ISO, nullable): Read timestamp.
- `created_at` (string ISO, non-null): Timestamp.
- Source Endpoint: `GET /v1/notifications`
> **Note**: read state is derived client-side as `read_at != null`.

## Announcement
- [SPECIFICATION GAP: Field schema undocumented]
> **Note**: no CRUD endpoints exist yet — see BACKEND_CHANGES_REQUIRED.md

## Leaderboard (Student)
- `user_id` (string, non-null): User ID.
- `display_name` (string, nullable): Display Name.
- `total_points` (number, non-null): Total Points.
- `attendance_points` (number, non-null): STUBBED (Currently returns 0).
- `contribution_points` (number, non-null): STUBBED (Currently returns 0).
- `competition_points` (number, non-null): STUBBED (Currently returns 0).
- `last_refreshed_at` (string ISO, non-null): Timestamp of last view refresh.
- Source Endpoint: `GET /v1/leaderboard/students`

## Leaderboard (Club)
- `club_id` (string, non-null): Club ID.
- `club_name` (string, non-null): Club Name.
- `total_points` (number, non-null): Total Points.
- `event_count` (number, non-null): Number of Events.
- `member_count` (number, non-null): Number of Members.
- `last_refreshed_at` (string ISO, non-null): Timestamp of last view refresh.
- Source Endpoint: `GET /v1/leaderboard/clubs`

## Admin User
- `id` (string, non-null): User ID.
- `email` (string, non-null): User email.
- `display_name` (string, nullable): User display name.
- `global_role` (enum, non-null): PLATFORM_ADMIN, FACULTY_ADMIN, STUDENT.
- Source Endpoint: `GET /v1/admin/users`

## Dashboard Summary
- `total_events` (number, non-null): Total events.
- `active_participants` (number, non-null): Total active participants.
- `upcoming_sessions` (number, non-null): Sessions occurring soon.
- `pending_approvals` (number, non-null): Events pending approval.
- Source Endpoint: `GET /v1/dashboard/summary`

## AuditLog
- `id` (number, non-null): ID (BigInt).
- `entity_type` (string, non-null): Entity type.
- `entity_id` (string, non-null): Entity ID.
- `previous_state` (object, nullable): Previous state.
- `new_state` (object, nullable): New state.
- `actor_id` (string, non-null): Actor ID.
- `action` (string, non-null): Action performed.
- `ip_address` (string, nullable): IP address.
- `created_at` (string ISO, non-null): Creation timestamp.
- Source Endpoint: `GET /v1/admin/audit-logs`

## LeaderboardScore
- `sourceId` (string, non-null): Source ID.
- `reason` (string, non-null): Reason for points.

## SSE / Live Events
- Source Endpoint: `GET /v1/events/:id/live`
- Connection: EventSource / Server-Sent Events
- Authentication: Bearer token via query parameter (`?token=...`)
- Keep-Alive: Emits `heartbeat` event every 30s.

> **Note**: Only 2 event types are actually implemented: `registration_count` (via pg_notify) and `heartbeat` (via Express setInterval). `attendance_count`, `session_opened`, `session_closed`, and `lock_status` are documented elsewhere in this repo but DO NOT EXIST in any trigger or emission code — do not build UI that depends on them until BACKEND_CHANGES_REQUIRED.md item for SSE is resolved.

### Payload Schema
All events are emitted as stringified JSON in the `data` field with the shape:
`{ "type": string, "payload": object }`

#### Event Types:
1. **`registration_count`**
   - `payload.count` (number, non-null): The new total registration count for the event.
   - Emitted when: A user registers, joins a team, leaves, or cancels.

2. **`waitlist_update`**
   - `payload.user_id` (string UUID, non-null): The ID of the user promoted from waitlist.
   - `payload.status` (string, non-null): Always `"REGISTERED"`.
   - Emitted when: A waitlisted user is automatically promoted due to a cancellation.

3. **`heartbeat`**
   - `payload.timestamp` (string ISO, non-null): The server time.
   - Emitted when: Every 30 seconds by the Express server.
