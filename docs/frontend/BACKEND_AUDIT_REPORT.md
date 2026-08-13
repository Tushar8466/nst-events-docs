# Backend Audit Report — Full Cross-Audit

> **Status**: COMPLETE | **Date**: 2026-08-13
> **Scope**: Every file in `docs/frontend/**`, `docs/api/**`, `docs/backend/**`, `docs/database/**`, `docs/architecture/**`, `docs/dashboard/**`, `docs/mobile/**`, `docs/product/**`, `docs/security/**`, `docs/engineering/**`, plus root-level `PROJECT_CONTEXT.md`, `README.md`, `replacement.txt`.

---

## Section A: Summary Table

| Issue ID | Category | Severity | Screens Affected | One-line Description |
|---|---|---|---|---|
| BACKEND-GAP-001 | MISSING ENDPOINT | BLOCKING | WEB-02 (Dashboard) | `GET /v1/dashboard/summary` does not exist in any backend doc |
| BACKEND-GAP-002 | ENUM MISMATCH | BLOCKING | WEB-03 (Events), DATA_CONTRACT | `registration_state` enum values `OPEN`/`CLOSED` don't exist; backend uses `event_state` (`PUBLISHED`/`ARCHIVED`) |
| BACKEND-GAP-003 | MISSING FIELD | BLOCKING | WEB-03 (Events) | Events list expects `location` field but schema has `location_name`; expects `capacity` but schema has `max_capacity` |
| BACKEND-GAP-004 | ENUM MISMATCH | BLOCKING | DATA_CONTRACT (User) | `global_role` uses `SUPER_ADMIN`, `EVENT_ADMIN` which don't exist; canonical values are `STUDENT`, `FACULTY_ADMIN`, `PLATFORM_ADMIN` |
| BACKEND-GAP-005 | ENUM MISMATCH | BLOCKING | DATA_CONTRACT (Club) | `club_status` omits `DISSOLVED` value; backend has `ACTIVE`, `INACTIVE`, `DISSOLVED` |
| BACKEND-GAP-006 | ENUM MISMATCH | BLOCKING | DATA_CONTRACT (Registration) | `NOT_REGISTERED` is listed as a `registration_status` value but doesn't exist in `04-enums.md` |
| BACKEND-GAP-007 | ENUM MISMATCH | DEGRADED | DATA_CONTRACT (AttendanceRecord) | `status` uses `VALID`/`INVALID` but canonical enum is `PRESENT`/`ABSENT`/`EXCUSED` |
| BACKEND-GAP-008 | MISSING ENDPOINT | BLOCKING | MOB-03 (Home) | `get_home_feed()` RPC is referenced as the sole data source for the Home screen but does not exist in any backend doc |
| BACKEND-GAP-009 | UNDOCUMENTED RESPONSE SHAPE | BLOCKING | WEB-04 / MOB-05 (Event Detail) | `GET /v1/events/:id` response shape is described only as "Full event detail with clubs, sessions, registration stats" — no field-level definition |
| BACKEND-GAP-010 | MISSING QUERY CAPABILITY | DEGRADED | WEB-03 (Events) | Events list uses `?status=OPEN` filter but backend uses `?filter_state=PUBLISHED`; naming and values diverge |
| BACKEND-GAP-011 | MISSING QUERY CAPABILITY | DEGRADED | WEB-03 (Events) | Events list uses `?page=1` pagination but backend uses `?cursor=` cursor pagination |
| BACKEND-GAP-012 | MISSING ENDPOINT | BLOCKING | WEB-12 (Approvals) | No dedicated approvals list endpoint; screen has no API map — frontend must filter `GET /v1/events?filter_state=PENDING_APPROVAL` which requires role-based visibility not documented |
| BACKEND-GAP-013 | MISSING ENDPOINT | BLOCKING | WEB-14 (User Management) | No `GET /v1/admin/users` list endpoint in routing matrix; screen has no API map |
| BACKEND-GAP-014 | MISSING ENDPOINT | BLOCKING | WEB-13 (Queue Monitoring) | Screen has no API map; must use `GET /v1/admin/queue/monitoring` + `GET /v1/admin/queue/dead-letters` but screen spec doesn't reference them |
| BACKEND-GAP-015 | MISSING ENDPOINT | BLOCKING | WEB-06 (Teams) | No `GET /v1/events/:id/teams` endpoint to list teams; screen has no API map |
| BACKEND-GAP-016 | UNDOCUMENTED RESPONSE SHAPE | BLOCKING | WEB-07 / MOB-10 (Notifications) | Screen has no API map; must use `GET /v1/notifications` but consumed fields are not specified |
| BACKEND-GAP-017 | UNDOCUMENTED RESPONSE SHAPE | BLOCKING | WEB-08 / MOB-11 (Profile) | Screen has no API map; must use `GET /users/me` but consumed fields are not specified in the screen spec |
| BACKEND-GAP-018 | UNDOCUMENTED RESPONSE SHAPE | BLOCKING | WEB-11 (Admin) | Admin screen has no API map at all — it's a hub for audit logs, user management, leaderboard recalculation |
| BACKEND-GAP-019 | MISSING ENDPOINT | BLOCKING | MOB-13 (Waitlist Flow) | Waitlist screen requires queue position ("You are #4") but no endpoint returns waitlist position |
| BACKEND-GAP-020 | REAL-TIME / SSE GAP | DEGRADED | Dashboard (Attendance Monitoring) | Dashboard attendance monitoring doc (13-attendance-management-flow.md) says "Real-time feed updated via SSE" for attendance records, but `21-realtime-listen-notify-contract.md` only defines `registration_count` and `waitlist_update` events — no `attendance_count` or per-record attendance SSE event |
| BACKEND-GAP-021 | AMBIGUOUS/CONFLICTING SPEC | DEGRADED | SSE Contract vs API Contract | `04-api-contract-freeze.md` §SSE lists 7 event types (`attendance_count`, `registration_count`, `waitlist_update`, `session_opened`, `session_closed`, `lock_status`, `heartbeat`) but `21-realtime-listen-notify-contract.md` §5 only defines 2 database-emitted types (`registration_count`, `waitlist_update`). 5 event types have no NOTIFY source |
| BACKEND-GAP-022 | AUTHORIZATION MISMATCH | DEGRADED | Dashboard (Operations Mode / Manual Attendance) | `10-operations-mode.md` says Core Members can "manually verify attendance" but `POST /events/:id/attendance/manual` in routing matrix requires `PLATFORM_ADMIN` only |
| BACKEND-GAP-023 | AUTHORIZATION MISMATCH | DEGRADED | Product (Dispute Review) | `attendance-dispute-system.md` says "Club Admin, Faculty Mentor, Faculty Admin" can review disputes; `GET /attendance/disputes` routing matrix says Auth=None (no specific role listed), but `04-api-contract-freeze.md` says `CLUB_ADMIN, Faculty, Platform Admin` |
| BACKEND-GAP-024 | AUTHORIZATION MISMATCH | DEGRADED | Product (Event Create) | `permission-matrix.md` allows `FACULTY_MENTOR`, `FACULTY_ADMIN` to Create Draft Events, but routing matrix `POST /v1/events` only authorizes `CLUB_ADMIN`, `CORE_MEMBER` (via "None" + service-level check implied) |
| BACKEND-GAP-025 | AUTHORIZATION MISMATCH | DEGRADED | Product (Submit for Approval) | `permission-matrix.md` allows Faculty Mentor/Faculty Admin/Platform Admin to Submit for Approval, but routing matrix + RPC only allow `CLUB_ADMIN`, `CORE_MEMBER` |
| BACKEND-GAP-026 | AUTHORIZATION MISMATCH | DEGRADED | Product (Event Registration) | `permission-matrix.md` blocks Faculty Mentor/Faculty Admin/Platform Admin from registering for events, but routing matrix shows `POST /v1/events/:id/register` Auth=None (any authenticated) |
| BACKEND-GAP-027 | MISSING ENDPOINT | BLOCKING | Product (Competition Results) | `competition-results-management.md` describes a "Competition Results Entry" dashboard screen but no web screen spec exists for it; also no `GET /v1/events/:id/results` in routing matrix (though it's in contract freeze) |
| BACKEND-GAP-028 | MISSING ENDPOINT | BLOCKING | Product (Leadership Handover) | Leadership handover product doc describes a Dashboard workflow but no web screen spec exists; endpoints `POST /clubs/:id/leadership/transfer`, `POST /leadership/:id/approve|reject` exist in contract freeze but NOT in routing matrix |
| BACKEND-GAP-029 | MISSING ENDPOINT | DEGRADED | Product (Attendance CSV Export) | `permission-matrix.md` lists "Export Attendance CSV" action for Club Admin+ but no export endpoint exists in any backend doc |
| BACKEND-GAP-030 | MISSING ENDPOINT | DEGRADED | Product (Analytics) | `permission-matrix.md` lists "View Event Analytics", "View Club Analytics", "View Campus Analytics" but no analytics endpoints exist |
| BACKEND-GAP-031 | MISSING FIELD | DEGRADED | MOB-13 (Leaderboard) | `student_leaderboard_mv` has `attendance_points`, `contribution_points`, `competition_points` "stubbed as 0" (V2) but DATA_CONTRACT lists them as `non-null` fields the frontend will render |
| BACKEND-GAP-032 | AMBIGUOUS/CONFLICTING SPEC | DEGRADED | Notifications table | `03-table-catalog.md` §notifications has `is_read BOOLEAN` but `03-prisma-schema-plan.md` §notifications has `read_at TIMESTAMPTZ NULL` — different column name and type |
| BACKEND-GAP-033 | AMBIGUOUS/CONFLICTING SPEC | DEGRADED | Audit logs | `03-table-catalog.md` §audit_logs uses `id UUID`, `target_id UUID` but `03-prisma-schema-plan.md` uses `id BIGSERIAL`, `entity_id UUID`, `entity_type TEXT`; completely different schema |
| BACKEND-GAP-034 | AMBIGUOUS/CONFLICTING SPEC | DEGRADED | Leaderboard scores | `03-table-catalog.md` §leaderboard_scores has `event_id`, `source_type`, `description` but `03-prisma-schema-plan.md` has `source_id`, `reason` (no `event_id`, no `source_type`, no `description`) |
| BACKEND-GAP-035 | AMBIGUOUS/CONFLICTING SPEC | DEGRADED | Announcements | `03-table-catalog.md` uses `author_id` but `03-prisma-schema-plan.md` uses `created_by` for same column |
| BACKEND-GAP-036 | MISSING ENDPOINT | BLOCKING | Dashboard (Announcements) | `08-announcement-center.md` and notification type `CLUB_ANNOUNCEMENT` reference `create_announcement` RPC but no announcement endpoints exist in routing matrix or contract freeze |
| BACKEND-GAP-037 | MISSING ENDPOINT | BLOCKING | Notification types | `SYSTEM_ALERT` references `create_system_alert` RPC that doesn't exist anywhere; `EVENT_REMINDER` references an "Event Reminder Scheduler" that is not documented; `ATTENDANCE_ALERT` references "Attendance Session Scheduler" that is not documented |
| BACKEND-GAP-038 | UNDOCUMENTED RESPONSE SHAPE | DEGRADED | Notifications | `GET /v1/notifications` response shape says "Paginated personal notifications" but doesn't specify exact fields; DATA_CONTRACT has `read` (boolean) but Prisma has `read_at` (timestamp) |
| BACKEND-GAP-039 | MISSING ENDPOINT | BLOCKING | DATA_CONTRACT (Notification) | DATA_CONTRACT §Notification has `read` as boolean, but the actual schema uses `read_at` (timestamp) — frontend needs to derive read state from `read_at != null` |
| BACKEND-GAP-040 | AUTHORIZATION MISMATCH | DEGRADED | Product (QR Generation) | `permission-matrix.md` allows Platform Admin to "Generate Event QR" but routing matrix only allows `CLUB_ADMIN, CORE_MEMBER` |
| BACKEND-GAP-041 | AUTHORIZATION MISMATCH | DEGRADED | Product (Audit Logs) | `permission-matrix.md` restricts audit log viewing to Platform Admin only, but `04-api-contract-freeze.md` `GET /admin/audit-logs` allows both `PLATFORM_ADMIN` and `FACULTY_ADMIN` |
| BACKEND-GAP-042 | AUTHORIZATION MISMATCH | DEGRADED | Product (Core Member + Add Member) | `permission-matrix.md` allows Core Members to "Add Member to Club" but routing matrix `POST /clubs/:id/members` requires `CLUB_ADMIN, FACULTY_MENTOR` |
| BACKEND-GAP-043 | MISSING ENDPOINT | DEGRADED | Product (Club Edit) | `permission-matrix.md` shows "Edit Club Profile" for Club Admin and Platform Admin, but no `PATCH /clubs/:id` endpoint exists |
| BACKEND-GAP-044 | MISSING ENDPOINT | BLOCKING | Mobile (Club Search) | `GET /clubs/search` exists in routing matrix but is not under `/v1/` prefix — inconsistent with all other v1 endpoints |
| BACKEND-GAP-045 | AMBIGUOUS/CONFLICTING SPEC | DEGRADED | Event state naming | `11-event-data-model.md` calls the enum `event_status` but `04-enums.md` canonical name is `event_state` (formerly `event_status`) |
| BACKEND-GAP-046 | MISSING FIELD | DEGRADED | WEB-04 (Event Detail) | Event detail consumes `location` but schema column is `location_name`; no documented transformation |
| BACKEND-GAP-047 | UNDOCUMENTED RESPONSE SHAPE | DEGRADED | Clubs | `GET /clubs/:id` response in contract freeze shows `members: [...]` and `event_count` but `event_count` is not a column — it must be computed server-side, and exact member object shape is undocumented |
| BACKEND-GAP-048 | MISSING ENDPOINT | DEGRADED | Admin (Role Change) | `PATCH /admin/users/:id/role` exists in contract freeze but NOT in routing matrix |
| BACKEND-GAP-049 | MISSING ENDPOINT | DEGRADED | Admin (Force Transfer) | `POST /admin/clubs/:id/force-transfer` exists in contract freeze but NOT in routing matrix |
| BACKEND-GAP-050 | MISSING ENDPOINT | DEGRADED | Admin (Points Adjust) | `POST /admin/points/adjust` exists in contract freeze but NOT in routing matrix |
| BACKEND-GAP-051 | MISSING ENDPOINT | DEGRADED | Participation Role | `PATCH /events/:id/registrations/:regId/role` exists in contract freeze but NOT in routing matrix |
| BACKEND-GAP-052 | MISSING ENDPOINT | DEGRADED | Results | `POST /events/:id/results` exists in contract freeze but NOT in routing matrix |
| BACKEND-GAP-053 | MISSING ENDPOINT | DEGRADED | Search | `GET /search` exists in contract freeze but NOT in routing matrix |
| BACKEND-GAP-054 | MISSING ENDPOINT | DEGRADED | Admin Audit Logs | `GET /admin/audit-logs` exists in contract freeze but NOT in routing matrix |
| BACKEND-GAP-055 | MISSING ENDPOINT | DEGRADED | Leadership | `POST /clubs/:id/leadership/transfer`, `POST /leadership/:id/approve`, `POST /leadership/:id/reject` exist in contract freeze but NOT in routing matrix |
| BACKEND-GAP-056 | MISSING ENDPOINT | DEGRADED | Registration Status | `GET /v1/events/:id/my-registration` exists in routing matrix but NOT documented in contract freeze with a response shape |
| BACKEND-GAP-057 | AMBIGUOUS/CONFLICTING SPEC | DEGRADED | Push token endpoint | `13-notification-data-model.md` says push tokens are sent via "PATCH /users/me or a dedicated endpoint" but contract freeze has `POST /users/me/push-token` as a dedicated endpoint |
| BACKEND-GAP-058 | AMBIGUOUS/CONFLICTING SPEC | DEGRADED | Notification table `is_read` vs `read_at` | `03-table-catalog.md` has `is_read BOOLEAN` but all other docs use `read_at TIMESTAMPTZ NULL`; frontend DATA_CONTRACT has `read` (boolean) |
| BACKEND-GAP-059 | UNDOCUMENTED RESPONSE SHAPE | DEGRADED | Attendance | `GET /v1/events/:id/attendance` and `GET /v1/users/me/attendance` return "Paginated attendance records with user info" / "Own attendance history" but no field-level schema |
| BACKEND-GAP-060 | UNDOCUMENTED RESPONSE SHAPE | DEGRADED | Registrations | `GET /v1/events/:id/registrations` and `GET /v1/users/me/registrations` lack field-level response schemas |

---

## Section B: Full Detail Per Issue

---

### BACKEND-GAP-001: Dashboard Summary Endpoint Missing

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-001 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | `GET /v1/dashboard/summary` |
| **Frontend Requires** | Aggregated widget data: `upcoming_events` list, `pending_approvals` list/count, `my_clubs` summary with `member_count`. Source: [dashboard.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/web/dashboard.md) §8-9 |
| **Backend Docs Say** | Not found in any backend doc. Not in routing matrix, not in contract freeze, not in RPC catalog. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Add `GET /v1/dashboard/summary`. Auth: Bearer JWT, Any authenticated. Response: `{ upcoming_events: [{ id, title, start_time, club_name }], pending_approvals_count: number, pending_approvals: [{ id, title, club_name }], my_clubs: [{ id, name, member_count, role }] }`. Role-filtered: `pending_approvals` only populated for Faculty+. |

---

### BACKEND-GAP-002: `registration_state` Enum Does Not Exist

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-002 |
| **Category** | ENUM MISMATCH |
| **Backend Capability** | `event_state` enum (`DRAFT`, `PENDING_APPROVAL`, `PUBLISHED`, `ARCHIVED`) |
| **Frontend Requires** | `registration_state` enum with values `OPEN`, `CLOSED`. Source: [events.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/web/events.md) §8 line 54, [DATA_CONTRACT.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/DATA_CONTRACT.md) §Event line 18 |
| **Backend Docs Say** | No `registration_state` enum exists in [04-enums.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/04-enums.md). The closest concepts are `event_state` (lifecycle) and `registration_status` (per-user: REGISTERED/WAITLISTED/CANCELLED). `OPEN`/`CLOSED` are not canonical values anywhere. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Either: (a) Add a computed/virtual `registration_state` field to `GET /v1/events` response, derived as: `OPEN` if `state == PUBLISHED && (max_capacity IS NULL OR registration_count < max_capacity) && !is_locked`, `CLOSED` otherwise. Or (b) update frontend DATA_CONTRACT to use `state` (event_state enum) and derive open/closed client-side from `state`, `registration_count`, `max_capacity`, `is_locked`. |

---

### BACKEND-GAP-003: Field Name Mismatches on Events

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-003 |
| **Category** | MISSING FIELD |
| **Backend Capability** | `events` table columns: `location_name`, `max_capacity` |
| **Frontend Requires** | `location` (not `location_name`), `capacity` (not `max_capacity`). Source: [events.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/web/events.md) §8 lines 53/55, [DATA_CONTRACT.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/DATA_CONTRACT.md) §Event lines 17/19 |
| **Backend Docs Say** | [03-prisma-schema-plan.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/03-prisma-schema-plan.md) §5 uses `location_name TEXT` and `max_capacity INTEGER NULL`. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Either rename columns to `location`/`capacity` in the schema, or alias them in the API serialization layer (`location_name` → `location`, `max_capacity` → `capacity`), and document the mapping in the contract freeze. |

---

### BACKEND-GAP-004: DATA_CONTRACT `global_role` Uses Wrong Values

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-004 |
| **Category** | ENUM MISMATCH |
| **Backend Capability** | `global_role` enum: `STUDENT`, `FACULTY_ADMIN`, `PLATFORM_ADMIN` |
| **Frontend Requires** | `global_role` values `SUPER_ADMIN`, `EVENT_ADMIN`, etc. Source: [DATA_CONTRACT.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/DATA_CONTRACT.md) §User line 7 |
| **Backend Docs Say** | [04-enums.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/04-enums.md) §global_role: `STUDENT`, `FACULTY_ADMIN`, `PLATFORM_ADMIN`. No `SUPER_ADMIN` or `EVENT_ADMIN`. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Fix DATA_CONTRACT to use canonical enum values: `STUDENT`, `FACULTY_ADMIN`, `PLATFORM_ADMIN`. |

---

### BACKEND-GAP-005: DATA_CONTRACT Club Status Missing `DISSOLVED`

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-005 |
| **Category** | ENUM MISMATCH |
| **Backend Capability** | `club_status`: `ACTIVE`, `INACTIVE`, `DISSOLVED` |
| **Frontend Requires** | Club `status` values `ACTIVE` or `INACTIVE` only. Source: [DATA_CONTRACT.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/DATA_CONTRACT.md) §Club line 38 |
| **Backend Docs Say** | [04-enums.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/04-enums.md) §club_status has 3 values including `DISSOLVED`. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Update DATA_CONTRACT to include `DISSOLVED`. Frontend must handle `DISSOLVED` clubs (e.g., display a "This club has been permanently closed" state). |

---

### BACKEND-GAP-006: `NOT_REGISTERED` Is Not a Backend Enum Value

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-006 |
| **Category** | ENUM MISMATCH |
| **Backend Capability** | `registration_status`: `REGISTERED`, `WAITLISTED`, `CANCELLED` |
| **Frontend Requires** | `NOT_REGISTERED` as a possible status value. Source: [DATA_CONTRACT.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/DATA_CONTRACT.md) §Registration line 23 |
| **Backend Docs Say** | [04-enums.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/04-enums.md) §registration_status: only `REGISTERED`, `WAITLISTED`, `CANCELLED`. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | `NOT_REGISTERED` is a frontend-derived state (no registration row exists). Document in DATA_CONTRACT that `NOT_REGISTERED` is derived client-side when `GET /v1/events/:id/my-registration` returns 404, not from the enum. |

---

### BACKEND-GAP-007: Attendance Status `VALID`/`INVALID` Don't Exist

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-007 |
| **Category** | ENUM MISMATCH |
| **Backend Capability** | `attendance_status`: `PRESENT`, `ABSENT`, `EXCUSED` |
| **Frontend Requires** | `status` values `VALID`, `INVALID`, etc. Source: [DATA_CONTRACT.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/DATA_CONTRACT.md) §AttendanceRecord line 62 |
| **Backend Docs Say** | [04-enums.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/04-enums.md) §attendance_status: `PRESENT`, `ABSENT`, `EXCUSED`. |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Fix DATA_CONTRACT to use canonical values: `PRESENT`, `ABSENT`, `EXCUSED`. |

---

### BACKEND-GAP-008: `get_home_feed()` RPC Does Not Exist

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-008 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | `get_home_feed()` — planned target API |
| **Frontend Requires** | Aggregated home feed: upcoming events, active attendance windows, notification badge count, club updates, pending actions, with delta sync via `last_fetched_at`. Source: [05-home-screen-architecture.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/mobile/05-home-screen-architecture.md) §Data Fetching, [10-home-priority-system.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/mobile/10-home-priority-system.md), [API_DEPENDENCY_MATRIX.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/API_DEPENDENCY_MATRIX.md) (marked `DEFERRED - DO NOT CALL`) |
| **Backend Docs Say** | Not in routing matrix, RPC catalog, contract freeze, or edge function catalog. SPECIFICATION_GAPS.md flags it as `BACKEND CONTRACT GAP`. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Either: (a) Implement `GET /v1/home/feed` endpoint returning `{ upcoming_events, active_attendance_windows, unread_notification_count, club_updates, pending_actions }` with `?last_fetched_at=` delta parameter. Or (b) Document that mobile Home screen will compose from multiple existing endpoints (`GET /v1/events`, `GET /v1/notifications`, etc.) and remove the single-endpoint architecture from the mobile docs. |

---

### BACKEND-GAP-009: `GET /v1/events/:id` Response Shape Undocumented

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-009 |
| **Category** | UNDOCUMENTED RESPONSE SHAPE |
| **Backend Capability** | `GET /v1/events/:id` |
| **Frontend Requires** | Full event detail including `title`, `description`, `start_time`, `end_time`, `location`, `clubs`, `sessions`, `registration_count`, `max_capacity`, `state`, `event_type`, `visibility`, `registration_type`, `is_locked`, `metadata`, `attendance_type`. Source: [event-detail.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/web/event-detail.md) §8-9 |
| **Backend Docs Say** | [04-api-contract-freeze.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md) §GET /events/:id says "Full event detail with clubs, sessions, registration stats" — no field-level schema. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Add explicit response schema to contract freeze: `{ id, title, description, start_time, end_time, location_name, event_type, state, visibility, registration_type, attendance_type, is_locked, max_capacity, registration_count, metadata, created_by, clubs: [{ club_id, club_name, is_primary }], sessions: [{ id, title, start_time, end_time, open_at, close_at, geofence_radius }] }`. |

---

### BACKEND-GAP-010 & BACKEND-GAP-011: Events List Query Parameter Mismatches

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-010 / BACKEND-GAP-011 |
| **Category** | MISSING QUERY CAPABILITY |
| **Backend Capability** | `GET /v1/events` supports `?filter_state=`, `?cursor=`, `?limit=`, `?sort=` |
| **Frontend Requires** | `?status=OPEN` (wrong parameter name and value) and `?page=1` (offset pagination, not cursor). Source: [events.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/web/events.md) §9 line 59 |
| **Backend Docs Say** | [04-api-contract-freeze.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md) §GET /events: uses `filter_state`, `filter_event_type`, `filter_club_id`, `cursor`, `limit`. |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Fix frontend screen spec to use backend's actual query parameters: `?filter_state=PUBLISHED&cursor=&limit=20`. |

---

### BACKEND-GAP-012: Approvals Screen Has No API

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-012 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | No dedicated approvals endpoint |
| **Frontend Requires** | A list of events in `PENDING_APPROVAL` state visible to Faculty Mentor+. Source: [approvals.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/web/approvals.md) §9 (API Map = N/A), [dashboard.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/web/dashboard.md) §8 (Pending Approvals widget) |
| **Backend Docs Say** | `GET /v1/events?filter_state=PENDING_APPROVAL` is the implicit approach but this requires the events endpoint to correctly RLS-filter based on faculty club association. Not explicitly documented. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Document that `GET /v1/events?filter_state=PENDING_APPROVAL` is the canonical approach for the Approvals screen, and confirm that events in `PENDING_APPROVAL` state are visible to the relevant Faculty Mentor/Admin via RLS. Alternatively, add `GET /v1/approvals/pending` as a convenience endpoint. |

---

### BACKEND-GAP-013: No User List Endpoint for User Management

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-013 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | No `GET /v1/admin/users` or equivalent |
| **Frontend Requires** | A paginated user list with search, role filtering, and actions (revoke sessions, change role). Source: [user-management.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/web/user-management.md) §9 (API Map = N/A), [permission-matrix.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/product/roles/permission-matrix.md) §System Settings |
| **Backend Docs Say** | Only individual user endpoints exist: `GET /users/me`, `GET /users/:id/profile`. No admin user listing. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Add `GET /v1/admin/users`. Auth: `PLATFORM_ADMIN`. Query: `cursor`, `limit`, `q` (search), `filter_global_role`. Response: `{ data: [{ id, email, full_name, global_role, created_at }], pagination }`. |

---

### BACKEND-GAP-015: No Team Listing Endpoint

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-015 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | No `GET /v1/events/:id/teams` endpoint |
| **Frontend Requires** | List of teams with members for team-based events (Join Team flow, Team Listing Page). Source: [12-registration-flow.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/mobile/12-registration-flow.md) §Join Team Flow, Team Creation, [teams.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/web/teams.md) |
| **Backend Docs Say** | Only mutation endpoints exist: `POST /v1/events/:id/teams` (create), `POST /v1/teams/:id/join`, `DELETE /v1/teams/:id/leave`. No read endpoint. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Add `GET /v1/events/:id/teams`. Auth: Any authenticated. Response: `{ data: [{ id, name, leader_id, leader_name, member_count, members: [{ user_id, full_name, registration_status }] }], pagination }`. |

---

### BACKEND-GAP-019: Waitlist Position Not Available

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-019 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | No endpoint returns waitlist queue position |
| **Frontend Requires** | "You are #4 on the waitlist" display. Source: [13-waitlist-flow.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/mobile/13-waitlist-flow.md) §Display Elements line 16 |
| **Backend Docs Say** | `register_event` RPC returns `registration_id` and `status` only. No position. `GET /v1/events/:id/my-registration` response shape is not documented. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Add `waitlist_position` to the response of `POST /v1/events/:id/register` (when status is `WAITLISTED`) and to `GET /v1/events/:id/my-registration`. Computed as: `SELECT COUNT(*) FROM event_registrations WHERE event_id = $1 AND registration_status = 'WAITLISTED' AND deleted_at IS NULL AND registered_at <= current_user_registered_at`. |

---

### BACKEND-GAP-020: SSE Attendance Live Feed Not Supported

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-020 |
| **Category** | REAL-TIME / SSE GAP |
| **Backend Capability** | `GET /v1/events/:id/live` SSE stream |
| **Frontend Requires** | "Real-time dashboard feed updated via SSE... after each successful `mark_attendance` RPC write". Source: [13-attendance-management-flow.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/dashboard/13-attendance-management-flow.md) §Attendance Monitoring |
| **Backend Docs Say** | [21-realtime-listen-notify-contract.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/21-realtime-listen-notify-contract.md) §5 only emits `registration_count` and `waitlist_update`. No `attendance_count` or per-record attendance event is defined. |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Add `attendance_marked` event type to the LISTEN/NOTIFY contract: `pg_notify('event_<id>_live', '{"type":"attendance_marked","payload":{"session_id":"uuid","user_id":"uuid","count":number}}')`. Emit from `mark_attendance` RPC. Also add to SSE Events Emitted list. |

---

### BACKEND-GAP-021: SSE Event Types Inconsistency

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-021 |
| **Category** | AMBIGUOUS/CONFLICTING SPEC |
| **Backend Capability** | SSE event types |
| **Frontend Requires** | 7 event types listed in contract freeze: `attendance_count`, `registration_count`, `waitlist_update`, `session_opened`, `session_closed`, `lock_status`, `heartbeat` |
| **Backend Docs Say** | [04-api-contract-freeze.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md) §SSE lists all 7 types. [21-realtime-listen-notify-contract.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/21-realtime-listen-notify-contract.md) §5 only defines 2 native PG types (`registration_count`, `waitlist_update`). `heartbeat` is Express-generated. `attendance_count`, `session_opened`, `session_closed`, `lock_status` have NO documented NOTIFY source or Express generation logic. |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | For each undocumented SSE type, either: (a) add the pg_notify call to the relevant RPC (`lock_event`→`lock_status`, session create/update→`session_opened`/`session_closed`, `mark_attendance`→`attendance_count`) and document in the LISTEN/NOTIFY contract, or (b) document that these are Express-emitted (not PG-emitted) and add Express emission logic docs. |

---

### BACKEND-GAP-022: Operations Mode Manual Attendance Auth Mismatch

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-022 |
| **Category** | AUTHORIZATION MISMATCH |
| **Backend Capability** | `POST /v1/events/:id/attendance/manual` |
| **Frontend Requires** | Operations Mode says Core Members can "manually verify attendance for students with broken phones". Source: [10-operations-mode.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/dashboard/10-operations-mode.md) §Attendance Monitoring |
| **Backend Docs Say** | [02-api-routing-matrix.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md) line 40: Authorization = `PLATFORM_ADMIN` only. [05-rpc-catalog.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/05-rpc-catalog.md) §manual_mark_attendance: "Caller: Platform Admin". |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Either: (a) Update Operations Mode doc to clarify that only Platform Admin can do manual marks (Core Members can only view attendance), or (b) Expand `POST /events/:id/attendance/manual` authorization to include `CLUB_ADMIN, CORE_MEMBER` with appropriate audit logging, updating routing matrix and RPC catalog. Product decision required. |

---

### BACKEND-GAP-024: Faculty Can't Create Events Despite Permission Matrix

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-024 |
| **Category** | AUTHORIZATION MISMATCH |
| **Backend Capability** | `POST /v1/events` |
| **Frontend Requires** | Faculty Mentor, Faculty Admin, Platform Admin can create draft events. Source: [permission-matrix.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/product/roles/permission-matrix.md) line 17, [12-event-management-flow.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/dashboard/12-event-management-flow.md) §Faculty Event Creation |
| **Backend Docs Say** | [04-api-contract-freeze.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md) §POST /events: Roles = `CLUB_ADMIN, CORE_MEMBER`. Faculty roles not listed. |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Add `FACULTY_MENTOR`, `FACULTY_ADMIN`, `PLATFORM_ADMIN` to the Authorization column for `POST /v1/events` in both routing matrix and contract freeze. |

---

### BACKEND-GAP-027: Competition Results Screen Missing

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-027 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | `POST /v1/events/:id/results`, `GET /v1/events/:id/results` |
| **Frontend Requires** | "Competition Results Entry" and "Competition Results Review" dashboard screens. Source: [competition-results-management.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/product/leaderboard/competition-results-management.md) §Submission Flow |
| **Backend Docs Say** | `POST /events/:id/results` and `GET /events/:id/results` exist in [04-api-contract-freeze.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md) but NOT in the [02-api-routing-matrix.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md). No frontend web screen spec exists for results entry. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | (a) Add `POST /v1/events/:id/results` and `GET /v1/events/:id/results` to routing matrix. (b) Create a frontend web screen spec for "Competition Results" (e.g., WEB-15). |

---

### BACKEND-GAP-028: Leadership Handover Endpoints Not in Routing Matrix

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-028 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | Leadership transfer endpoints |
| **Frontend Requires** | Dashboard leadership handover workflow. Source: [leadership-handover.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/product/clubs/leadership-handover.md) |
| **Backend Docs Say** | Endpoints exist in [04-api-contract-freeze.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md) §Leadership Transfer but are absent from [02-api-routing-matrix.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md). No web screen spec. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | (a) Add all three leadership endpoints to routing matrix. (b) Create a frontend web screen spec for "Leadership Transfer" or integrate into club detail screen. |

---

### BACKEND-GAP-029: Attendance CSV Export Not Documented

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-029 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | No CSV export endpoint |
| **Frontend Requires** | "Export Attendance CSV" for Club Admin, Faculty, Platform Admin. Source: [permission-matrix.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/product/roles/permission-matrix.md) line 29 |
| **Backend Docs Say** | Not found in any backend doc. |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Add `GET /v1/events/:id/attendance/export?format=csv`. Auth: `CLUB_ADMIN, FACULTY_MENTOR, FACULTY_ADMIN, PLATFORM_ADMIN`. Response: `text/csv` with Content-Disposition header. |

---

### BACKEND-GAP-030: Analytics Endpoints Not Documented

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-030 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | No analytics endpoints |
| **Frontend Requires** | Event Analytics, Club Analytics, Campus Analytics screens. Source: [permission-matrix.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/product/roles/permission-matrix.md) lines 38-40 |
| **Backend Docs Say** | Not found in any backend doc. |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Defer to V2, or add: `GET /v1/analytics/events/:id` (event-level stats), `GET /v1/analytics/clubs/:id` (club-level), `GET /v1/analytics/campus` (platform-wide). |

---

### BACKEND-GAP-031: Leaderboard Breakdown Columns Stubbed

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-031 |
| **Category** | MISSING FIELD |
| **Backend Capability** | `student_leaderboard_mv` columns `attendance_points`, `contribution_points`, `competition_points` |
| **Frontend Requires** | Non-null breakdown values displayed. Source: [DATA_CONTRACT.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/DATA_CONTRACT.md) §Leaderboard (Student) lines 81-83 |
| **Backend Docs Say** | [03-table-catalog.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/03-table-catalog.md) §student_leaderboard_mv: "Intentionally deferred to V2, currently stubbed as 0". |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Frontend should still render these columns but display "0" gracefully (not as a bug). Document in DATA_CONTRACT that these fields are always 0 in V1. Add a V2 milestone to populate them. |

---

### BACKEND-GAP-032: Notifications `is_read` vs `read_at` Conflict

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-032 |
| **Category** | AMBIGUOUS/CONFLICTING SPEC |
| **Backend Capability** | `notifications` table |
| **Frontend Requires** | Consistent read-state field |
| **Backend Docs Say** | [03-table-catalog.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/03-table-catalog.md) §notifications line 196: `is_read BOOLEAN`. [03-prisma-schema-plan.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/03-prisma-schema-plan.md) §13 line 199: `read_at TIMESTAMPTZ NULL`. [DATA_CONTRACT.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/DATA_CONTRACT.md) §Notification line 70: `read` (boolean). |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Prisma schema (`read_at TIMESTAMPTZ NULL`) is authoritative. Update table catalog to match. Frontend derives boolean from `read_at != null`. Update DATA_CONTRACT to `read_at` (string ISO, nullable). |

---

### BACKEND-GAP-033: Audit Logs Schema Conflict

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-033 |
| **Category** | AMBIGUOUS/CONFLICTING SPEC |
| **Backend Capability** | `audit_logs` table |
| **Frontend Requires** | Consistent audit log schema |
| **Backend Docs Say** | [03-table-catalog.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/03-table-catalog.md) §audit_logs: PK=`id UUID`, columns=`action, actor_id, target_id, metadata, ip_address`. [03-prisma-schema-plan.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/03-prisma-schema-plan.md) §19: PK=`id BIGSERIAL`, columns=`actor_id, action, entity_type, entity_id, previous_state, new_state, ip_address`. Completely different schemas. |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Prisma schema is authoritative. Update table catalog to match: PK=`id BIGSERIAL`, columns=`actor_id, action, entity_type, entity_id, previous_state JSONB, new_state JSONB, ip_address, created_at`. |

---

### BACKEND-GAP-034: Leaderboard Scores Schema Conflict

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-034 |
| **Category** | AMBIGUOUS/CONFLICTING SPEC |
| **Backend Capability** | `leaderboard_scores` table |
| **Frontend Requires** | Consistent leaderboard score data |
| **Backend Docs Say** | [03-table-catalog.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/03-table-catalog.md) §leaderboard_scores: columns include `event_id, source_type, description`. [03-prisma-schema-plan.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/03-prisma-schema-plan.md) §18: columns include `source_id, reason` (NO `event_id`, NO `source_type`, NO `description`). |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Prisma schema is authoritative. Update table catalog to match: `user_id, club_id, points, reason, source_id, created_at`. |

---

### BACKEND-GAP-036: Announcement Endpoints Missing

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-036 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | No announcement CRUD endpoints |
| **Frontend Requires** | Announcement Center for creating/viewing announcements. Source: [08-announcement-center.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/dashboard/08-announcement-center.md), [07-notification-center.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/dashboard/07-notification-center.md) §CLUB_ANNOUNCEMENT type references `create_announcement` RPC |
| **Backend Docs Say** | `announcements` table exists in schema but no CRUD endpoints in routing matrix or contract freeze. `create_announcement` RPC referenced in notification center doc but not in RPC catalog. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | Add: `POST /v1/clubs/:id/announcements` (CLUB_ADMIN), `GET /v1/announcements` (Any auth, filtered by club memberships), `GET /v1/announcements/:id` (Any auth). Add `create_announcement` to RPC catalog. |

---

### BACKEND-GAP-037: Notification Scheduler RPCs Missing

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-037 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | `create_system_alert` RPC, Event Reminder Scheduler, Attendance Session Scheduler |
| **Frontend Requires** | `SYSTEM_ALERT`, `EVENT_REMINDER`, `ATTENDANCE_ALERT` notification types to be generated. Source: [07-notification-center.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/dashboard/07-notification-center.md) §8-10 |
| **Backend Docs Say** | Not found in RPC catalog, routing matrix, or edge function catalog. The notification types are defined in the notification center doc but their producers (`create_system_alert RPC`, `Event Reminder Scheduler`, `Attendance Session Scheduler`) are completely undocumented. |
| **Severity** | BLOCKING |
| **Recommended Backend Change** | (a) Add `POST /v1/admin/system-alerts` (PLATFORM_ADMIN) with `create_system_alert` RPC to RPC catalog. (b) Document the Event Reminder Scheduler as a pg_cron or nst-worker cron job. (c) Document the Attendance Session Scheduler similarly. |

---

### BACKEND-GAP-043: No Club Edit Endpoint

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-043 |
| **Category** | MISSING ENDPOINT |
| **Backend Capability** | No `PATCH /v1/clubs/:id` endpoint |
| **Frontend Requires** | "Edit Club Profile" for Club Admin and Platform Admin. Source: [permission-matrix.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/product/roles/permission-matrix.md) line 35 |
| **Backend Docs Say** | Only `PATCH /clubs/:id/status` exists (status changes only, Platform Admin). No general club edit endpoint. |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Add `PATCH /v1/clubs/:id`. Auth: `CLUB_ADMIN, PLATFORM_ADMIN`. Request: `{ name?, description?, banner_url? }`. Response 200: updated club object. |

---

### BACKEND-GAP-044: Club Endpoints Not Under `/v1/` Prefix

| Field | Value |
|---|---|
| **Issue ID** | BACKEND-GAP-044 |
| **Category** | AMBIGUOUS/CONFLICTING SPEC |
| **Backend Capability** | Club and User endpoints |
| **Frontend Requires** | Consistent API prefix for all endpoints |
| **Backend Docs Say** | [02-api-routing-matrix.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md): Clubs mount at `/clubs` (not `/v1/clubs`), Users mount at `/users` (not `/v1/users`). All other endpoints use `/v1/` prefix. |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Clarify whether this is intentional (clubs/users were implemented before the v1 prefix convention) or a bug. If intentional, document the prefix exception. If not, migrate to `/v1/clubs` and `/v1/users`. |

---

### BACKEND-GAP-048 through BACKEND-GAP-055: Contract Freeze vs Routing Matrix Gaps

| Field | Value |
|---|---|
| **Issue IDs** | BACKEND-GAP-048 through BACKEND-GAP-055 |
| **Category** | AMBIGUOUS/CONFLICTING SPEC |
| **Backend Capability** | Multiple endpoints |
| **Backend Docs Say** | The following endpoints exist in [04-api-contract-freeze.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md) but are ABSENT from [02-api-routing-matrix.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md): `PATCH /admin/users/:id/role`, `POST /admin/clubs/:id/force-transfer`, `POST /admin/points/adjust`, `PATCH /events/:id/registrations/:regId/role`, `POST /events/:id/results`, `GET /events/:id/results`, `GET /search`, `GET /admin/audit-logs`, `POST /clubs/:id/leadership/transfer`, `POST /leadership/:id/approve`, `POST /leadership/:id/reject` |
| **Severity** | DEGRADED |
| **Recommended Backend Change** | Add all 11 endpoints to the routing matrix with correct Auth, Authorization, Service, and Response columns. The routing matrix warns it is authoritative — these gaps mean the endpoints may exist in code but are not "officially" tracked. |

---

## Section C: Reverse-Case Findings (Backend Exists, No Frontend Screen Uses It)

| # | Backend Capability | Location | Frontend Coverage | Assessment |
|---|---|---|---|---|
| RC-001 | `POST /v1/attendance/sync-offline` | [Routing matrix L37](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md#L37) | No web or mobile screen spec references offline sync | Expected: Operations Mode (dashboard) should trigger this, but the screen spec doesn't exist yet |
| RC-002 | `GET /clubs/search` | [Routing matrix L48](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md#L48) | No frontend screen references this search endpoint separately from `GET /clubs` | May be dead surface; clarify if mobile club discovery uses this |
| RC-003 | `PATCH /v1/notifications/preferences` / `GET /v1/notifications/preferences` | [Routing matrix L75-76](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md#L75) | No web or mobile screen spec for notification preferences | Missing screen: need a "Notification Settings" screen in profile |
| RC-004 | `POST /users/me/push-token` | [Routing matrix L89](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md#L89) | No mobile screen spec references push token registration | Expected: handled silently on mobile login, not a screen — but should be documented in mobile auth screen spec |
| RC-005 | `POST /v1/events/:id/lock` / `POST /v1/events/:id/unlock` | [Routing matrix L64-65](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md#L64) | Only referenced in Operations Mode ("Lockdown Event" button) but no web screen spec integrates it | Should be documented in event detail or operations mode screen spec |
| RC-006 | `assign_participation_role` RPC | [RPC Catalog](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/05-rpc-catalog.md#L59) | No frontend screen spec references role assignment | Missing screen: need a "Participant Role Assignment" UI on event detail for admins |
| RC-007 | `adjust_points_disciplinary` RPC | [RPC Catalog](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/05-rpc-catalog.md#L111) | No frontend screen spec references disciplinary point adjustment | Should be in Admin screen spec |
| RC-008 | `POST /v1/admin/users/:userId/revoke-sessions` | [Routing matrix L34](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md#L34) | No web screen references session revocation | Should be in User Management screen spec |
| RC-009 | `submit_competition_result` RPC | [RPC Catalog](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/05-rpc-catalog.md#L107) | No frontend screen spec for competition results entry | Product doc references it but no screen spec exists |
| RC-010 | `initiate/approve/reject_leadership_transfer` RPCs | [RPC Catalog](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/05-rpc-catalog.md#L138) | No frontend screen spec for leadership transfer | Product doc references it but no screen spec exists |
| RC-011 | `force_transfer_leadership` RPC | [RPC Catalog](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/05-rpc-catalog.md#L156) | No frontend screen references platform admin force transfer | Should be in Admin or Club Management screen |
| RC-012 | `announcements` table | [Prisma Schema](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/03-prisma-schema-plan.md#L230) | No CRUD endpoints, no screen spec | Schema exists but no API or UI |
| RC-013 | `event_results` table | [Prisma Schema](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/03-prisma-schema-plan.md#L182) | Endpoints exist in contract freeze only, no routing matrix entry, no screen spec | End-to-end not wired |
| RC-014 | Flagged Attendances Panel | [13-attendance-management-flow.md](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/dashboard/13-attendance-management-flow.md#L12) | Dashboard doc describes it in detail but no web screen spec and no API to get flagged records or mark as reviewed / revoke | Missing: needs `GET /v1/events/:id/attendance?filter_flagged=true` (exists in contract) + `PATCH /v1/attendance/:id/review` (missing) + `DELETE /v1/attendance/:id` (missing) |

---

## Section D: Cross-Document Contradictions (AMBIGUOUS/CONFLICTING SPEC)

### D-1: SSE Event Types — Contract Freeze vs LISTEN/NOTIFY Contract
- **Contract Freeze** ([04-api-contract-freeze.md L713](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md#L713)): Lists 7 SSE event types: `attendance_count`, `registration_count`, `waitlist_update`, `session_opened`, `session_closed`, `lock_status`, `heartbeat`.
- **LISTEN/NOTIFY** ([21-realtime-listen-notify-contract.md §5](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/21-realtime-listen-notify-contract.md#L22)): Only 2 PG-native types: `registration_count`, `waitlist_update`. 
- **DATA_CONTRACT** ([DATA_CONTRACT.md §SSE](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/DATA_CONTRACT.md#L109)): Only 3 types: `registration_count`, `waitlist_update`, `heartbeat`.
- **Impact**: 5 event types exist in contract freeze that have no implementation path documented.

### D-2: Notifications `is_read` vs `read_at`
- **Table catalog** ([03-table-catalog.md L196](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/03-table-catalog.md#L196)): `is_read BOOLEAN`
- **Prisma schema** ([03-prisma-schema-plan.md L199](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/03-prisma-schema-plan.md#L199)): `read_at TIMESTAMPTZ NULL`
- **DATA_CONTRACT** ([DATA_CONTRACT.md L70](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/frontend/shared/DATA_CONTRACT.md#L70)): `read` (boolean)
- **Impact**: Three different representations of the same concept across three docs.

### D-3: Audit Logs Schema Divergence
- **Table catalog** ([03-table-catalog.md L227-235](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/03-table-catalog.md#L227)): PK=UUID, cols=`action, actor_id, target_id, metadata, ip_address`
- **Prisma schema** ([03-prisma-schema-plan.md L273-283](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/03-prisma-schema-plan.md#L273)): PK=BIGSERIAL, cols=`actor_id, action, entity_type, entity_id, previous_state, new_state, ip_address`
- **Schema architecture** ([schema-architecture.md L82-83](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/schema-architecture.md#L82)): PK=UUID, cols=`actor_id, action, entity_type, entity_id, previous_state, new_state, ip_address`
- **Impact**: PK type disagrees (UUID vs BIGSERIAL). Column names disagree (`target_id`/`metadata` vs `entity_type`/`entity_id`/`previous_state`/`new_state`).

### D-4: Leaderboard Scores Schema Divergence
- **Table catalog** ([03-table-catalog.md L216-226](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/03-table-catalog.md#L216)): cols include `event_id`, `source_type`, `description`
- **Prisma schema** ([03-prisma-schema-plan.md L263-271](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/03-prisma-schema-plan.md#L263)): cols = `user_id, club_id, points, reason, source_id, created_at` (no `event_id`, `source_type`, `description`)
- **Impact**: Completely different column sets for the same table.

### D-5: Announcements `author_id` vs `created_by`
- **Table catalog** ([03-table-catalog.md L214](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/03-table-catalog.md#L214)): uses `author_id`
- **Prisma schema** ([03-prisma-schema-plan.md L237](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/03-prisma-schema-plan.md#L237)): uses `created_by`

### D-6: Push Token Registration Endpoint
- **Notification data model** ([13-notification-data-model.md L152](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/13-notification-data-model.md#L152)): "sends the token to Express via `PATCH /users/me` or a dedicated endpoint"
- **Contract freeze** ([04-api-contract-freeze.md L145](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md#L145)): Dedicated `POST /users/me/push-token` endpoint
- **Impact**: Minor — contract freeze is authoritative, but the older data model doc has stale info.

### D-7: Event State Enum Naming
- **Event data model** ([11-event-data-model.md L11](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/11-event-data-model.md#L11)): refers to `event_status` enum
- **Enums doc** ([04-enums.md L8](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/database/04-enums.md#L8)): canonical name is `event_state` (with note "formerly `event_status`")
- **Impact**: Stale naming in event data model doc.

### D-8: `GET /events/:id/results` Duplicated at Two Positions
- **Contract freeze** ([04-api-contract-freeze.md L10-16](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md#L10) and [L686-691](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md#L686)): The `GET /events/:id/results` endpoint is defined twice, once at the very top (appears to be a paste error alongside `PATCH /events/:id/sessions/:sessionId`) and once in the correct Results section.
- **Impact**: Confusing duplicate; the top occurrence looks like a rendering artifact.

### D-9: `GET /attendance/disputes` Authorization
- **Routing matrix** ([02-api-routing-matrix.md L42](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md#L42)): Authorization = `None`
- **Contract freeze** ([04-api-contract-freeze.md L460-466](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/backend/04-api-contract-freeze.md#L460)): Roles = `CLUB_ADMIN, Faculty, Platform Admin`
- **Product doc** ([attendance-dispute-system.md L15](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/product/attendance/attendance-dispute-system.md#L15)): "Club Admin, Faculty Mentor, Faculty Admin (Dashboard)"
- **Impact**: Routing matrix says no authorization required, contract freeze restricts to admin roles. The routing matrix also has a second `GET /v1/attendance/disputes` at line 42 with Auth=None for students to view their own disputes, creating ambiguity.

### D-10: Manual Attendance — Who Can Do It?
- **Operations Mode** ([10-operations-mode.md L16](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/dashboard/10-operations-mode.md#L16)): "Allows Core Members to manually verify attendance"
- **Routing matrix** ([02-api-routing-matrix.md L40](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/02-api-routing-matrix.md#L40)): `PLATFORM_ADMIN` only
- **Product attendance doc** ([attendance-system.md L12](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/product/attendance/attendance-system.md#L12)): "Only Platform Admins have override capabilities"
- **RPC catalog** ([05-rpc-catalog.md L116](file:///Users/sohamdhande/Docs_Local/nst-events-docs/docs/api/05-rpc-catalog.md#L116)): "Caller: Platform Admin"
- **Impact**: Operations Mode doc contradicts 3 other authoritative sources.
