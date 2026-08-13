# Codebase Verification Report

**Date:** 2026-08-13
**Target Codebase:** `nst-events` (backend)
**Objective:** Verify the claims made in `BACKEND_AUDIT_REPORT.md` against the actual, deployed Prisma schema, Express routes, and RPC functions in the backend repository.

## Executive Summary

The audit of the `nst-events` backend codebase reveals that a significant majority of the "gaps" identified in the frontend documentation are **false gaps (DOCS WERE WRONG)**, where the frontend documentation has drifted from the actual, implemented backend reality. 

However, several **true gaps (CONFIRMED GAP)** were found where the backend genuinely lacks critical endpoints required by the UI designs (e.g., Dashboard summary, Admin lists).

Below is the definitive ground-truth classification of all 60 claims (grouped by category for readability).

---

## 1. Missing Endpoints (CONFIRMED GAPS)
These are endpoints required by the frontend documentation that **DO NOT EXIST** in the current `nst-events` backend routing or services.

*   **Dashboard & Home:**
    *   `GET /v1/dashboard/summary` - **STATUS: CONFIRMED GAP**. No such endpoint exists in `apps/api/src/`.
    *   `GET /v1/home/feed` (or `get_home_feed`) - **STATUS: CONFIRMED GAP**. Not implemented.
*   **Waitlist & Teams:**
    *   `GET /v1/events/:id/waitlist` - **STATUS: CONFIRMED GAP**. Waitlist position calculation is not implemented.
    *   `GET /v1/events/:id/teams` - **STATUS: CONFIRMED GAP**. The `teams.router.ts` only supports `/:id/join` and `/:id/leave`. It cannot list teams for an event.
*   **Administration & Operations:**
    *   `GET /v1/admin/users` - **STATUS: CONFIRMED GAP**. `admin/users.router.ts` only has `/:userId/revoke-sessions`.
    *   `POST /v1/admin/users/:id/role` - **STATUS: CONFIRMED GAP**. No role adjustment endpoint exists.
    *   `GET /v1/admin/audit-logs` - **STATUS: CONFIRMED GAP**. No router exposes the `audit_logs` table.
    *   `POST /v1/admin/points/adjust` - **STATUS: CONFIRMED GAP**. Manual point adjustment not implemented.
*   **Analytics & Export:**
    *   `GET /v1/events/:id/attendance/export` - **STATUS: CONFIRMED GAP**. No CSV/export functionality in `attendance.router.ts`.
    *   `GET /v1/analytics/*` - **STATUS: CONFIRMED GAP**. No analytics router exists.
*   **Announcements:**
    *   `GET /v1/announcements` / `POST /v1/announcements` - **STATUS: CONFIRMED GAP**. No announcements router exists, despite the `announcements` DB table existing.

---

## 2. Schema and Enum Mismatches (DOCS WERE WRONG)
The frontend documentation defines schemas/enums that contradict the actual `schema.prisma`. The backend code is correct, and the docs must be updated to match the code.

*   **Event Schema Fields:**
    *   `location` vs `location_name` - **STATUS: DOCS WERE WRONG**. Code uses `locationName` (`location_name`).
    *   `capacity` vs `max_capacity` - **STATUS: DOCS WERE WRONG**. Code uses `maxCapacity` (`max_capacity`).
*   **Event States:**
    *   `registration_state` (`OPEN`/`CLOSED`) - **STATUS: DOCS WERE WRONG**. This field does not exist. The backend governs lifecycle purely via `EventState` (`DRAFT`, `PENDING_APPROVAL`, `PUBLISHED`, `ARCHIVED`) and `registration_count` vs `max_capacity`.
*   **Global Roles:**
    *   `SUPER_ADMIN` / `EVENT_ADMIN` - **STATUS: DOCS WERE WRONG**. The `schema.prisma` defines `GlobalRole` as `STUDENT`, `FACULTY_ADMIN`, `PLATFORM_ADMIN`.
*   **Club Roles & Status:**
    *   `ClubStatus` - **STATUS: DOCS WERE WRONG**. Defined in DB as `ACTIVE`, `INACTIVE`, `DISSOLVED`.
*   **Registration & Attendance:**
    *   `RegistrationStatus` - **STATUS: DOCS WERE WRONG**. Defined as `REGISTERED`, `WAITLISTED`, `CANCELLED`. `NOT_REGISTERED` is not a valid enum value (it is the absence of a record).
    *   `AttendanceStatus` - **STATUS: DOCS WERE WRONG**. Defined as `PRESENT`, `ABSENT`, `EXCUSED`. (Docs claimed `VALID`/`INVALID`).
*   **Notifications:**
    *   `is_read` - **STATUS: DOCS WERE WRONG**. Code uses `readAt` (`read_at`) nullable timestamp.
*   **Audit Logs & Leaderboard:**
    *   `AuditLog` fields - **STATUS: DOCS WERE WRONG**. Code uses `id` (BigInt), `entityType`, `entityId`, `previousState`, `newState`.
    *   `LeaderboardScore` fields - **STATUS: DOCS WERE WRONG**. Code uses `sourceId` and `reason` (not `event_id` or `source_type`).
*   **Announcements:**
    *   `author_id` - **STATUS: DOCS WERE WRONG**. Code uses `createdBy` (`created_by`).

---

## 3. Real-Time (SSE) Payload Mismatches (PARTIAL / DOCS WRONG)
*   **SSE Events Emitted:** The frontend documentation claims SSE supports `attendance_count`, `session_opened`, `session_closed`, and `lock_status`.
*   **STATUS: DOCS WERE WRONG / CONFIRMED GAP**. The actual PostgreSQL `emit_event_live_update` function and the Express `sse.router.ts` **only** emit:
    1.  `registration_count` (via `pg_notify`)
    2.  `heartbeat` (via Express `setInterval`)
    *   No triggers exist in the migrations to emit the other events. The frontend cannot rely on them.

---

## 4. Query Parameter & Pagination Mismatches (DOCS WERE WRONG)
*   **`GET /v1/events` Query Params:** Docs claim `status` and `page`. 
*   **STATUS: DOCS WERE WRONG**. The `ListEventsQuerySchema` in `events.schema.ts` strictly uses `cursor` and `limit` for pagination, and specific filters like `filter_state`, `filter_event_type`, `filter_club_id`, `filter_visibility`. 

---

## 5. Authorization Constraints (DOCS WERE WRONG)
*   **`POST /v1/events` (Create Event):** Docs claim `FACULTY_MENTOR` can create events. 
*   **STATUS: DOCS WERE WRONG**. In `events.service.ts`, `createEvent` explicitly checks if the user is a `GLOBAL_ADMIN_ROLE` (`PLATFORM_ADMIN`, `FACULTY_ADMIN`) or has the club role `CLUB_ADMIN` / `CORE_MEMBER`. A `FACULTY_MENTOR` will receive a 403 Forbidden.

## 6. Response Payload Mismatches (DOCS WERE WRONG)
*   **`GET /v1/clubs/:id`:** Docs claim it returns `owner_id` and `created_at` at the root.
*   **STATUS: DOCS WERE WRONG**. `clubs.service.ts` returns `{ id, name, description, banner_url, status, event_count, members }`. The `members` array contains `{ user_id, role, full_name, avatar_url }`.
*   **`GET /v1/events/:id`:** Docs expect a flat `club_id`.
*   **STATUS: DOCS WERE WRONG**. `events.service.ts` returns the `eventClubs` array (due to the M:N relationship to support collaborative events).

---

## Conclusion
The `nst-events` backend codebase is the authoritative source of truth. The frontend documentation must be immediately remediated to remove non-existent schema fields, correct the query parameters, and drop the assumption that unimplemented SSE events exist. 

Simultaneously, backend development (Phase 20S) must be scheduled to implement the **CONFIRMED GAPS** (Dashboard, Admin, Export) before the frontend UI can be finalized.

---

## Follow-Up Verification: Team Creation & Leaderboard

*   **Team Creation Route:** `POST /v1/events/:id/teams`
    *   **STATUS: DOCS WERE WRONG (Gap is False)**. The original report missed this because it only checked `teams.router.ts`. The route is actively defined in the registrations router.
    *   **Evidence**: Found in `apps/api/src/modules/registrations/registrations.router.ts` at line 37. It defines `POST /events/:id/teams`. It applies `authenticate` middleware, validates the request body expects `{ team_name: string }` via `CreateTeamSchema`, and calls `teamsService.createTeam`. The service (`apps/api/src/modules/teams/teams.service.ts` line 6) successfully executes the raw SQL `create_team` function and returns `{ team_id: string, name: string, leader_id: string }`.

*   **Leaderboard Endpoints:** `GET /v1/leaderboard/students` & `GET /v1/leaderboard/clubs`
    *   **STATUS: PARTIALLY EXISTS**. The endpoints and underlying views are real, but the specific point breakdown fields are explicitly hardcoded to 0.
    *   **Evidence**: 
        *   Endpoints are defined in `apps/api/src/modules/leaderboard/leaderboard.router.ts` (lines 10, 19).
        *   The materialized view `student_leaderboard_mv` is defined in `packages/database/prisma/migrations/20260806120000_phase5_attendance_leaderboard/migration.sql` (line 53). It explicitly hardcodes `0::INTEGER AS attendance_points`, `contribution_points`, and `competition_points`.
        *   This is an intentional placeholder for an unfinished implementation. The migration file comments at line 230 confirm: *"Skeleton implementation for Milestone 1. Business logic for TOTP, geofence, device collision, and point calculation is deferred to Milestone 2."*
