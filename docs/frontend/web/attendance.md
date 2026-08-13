# Attendance Screen

## 1. Identity
- **Screen ID**: WEB-09
- **Screen Name**: Attendance Management
- **Platform**: Web
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/events/[id]/attendance`

## 3. Role / Access
- **Visibility**: CLUB_ADMIN, CORE_MEMBER, FACULTY_MENTOR, PLATFORM_ADMIN
- **Access**: Only authorized admins for the specific event.
- **UX Guard**: Redirect to Event Detail if user lacks permission.

## 4. Entry Points
- "Manage Attendance" button on Event Detail page.

## 5. Exit / Navigation
- **Back**: Returns to Event Detail.

## 6. Layout Hierarchy
```text
SidebarNavigation
├── div (Padding Wrapper)
│   ├── BreadcrumbTrail
│   ├── div (Header & Actions)
│   │   ├── h1 (text: "Attendance")
│   │   └── div
│   │       ├── Button (text: "Generate QR")
│   │       └── Button (text: "Export CSV")
│   └── div (List/Table)
│       └── table
│           ├── tr (header: Participant, Marked At, Method, Status)
│           └── tr (record row)
│               ├── td (User Name inferred from user_id)
│               ├── td (markedAt)
│               ├── td (method)
│               └── td (status badge)
```

## 7. Component Map
- **SidebarNavigation**: Context tree
- **Table**: Data display
- **Badge**: Status visualizer
- **Button**: Action triggers

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Name | `userId` (expanded) | string | Needs client-side join or API expansion to show `full_name` |
| Marked At | `markedAt` | string ISO | Formatted Time |
| Method | `method` | enum | Raw Text |
| Status | `status` | enum | Badge (`PRESENT`, `ABSENT`, `EXCUSED`) |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/events/:id/attendance`
- **Purpose**: Fetch all attendance records for the event.
- **Status**: Exists (`02-api-routing-matrix.md` row 38).

- **Method**: `POST`
- **Path**: `/v1/attendance/generate-qr`
- **Purpose**: Generate a new QR code payload.
- **Status**: Exists (`02-api-routing-matrix.md` row 35).

- **Method**: `GET`
- **Path**: `/v1/events/:id/attendance/export`
- **Purpose**: Export attendance as CSV.
- **Status**: **IMPLEMENTED**. (Phase 21J)

- **Method**: `POST`
- **Path**: `/v1/events/:id/attendance/manual`
- **Purpose**: Manually mark a student.
- **Status**: Exists (`02-api-routing-matrix.md` row 40).

## 10. UI States
- **Loading**: Skeletons.
- **Empty**: "No attendance records yet."
- **Error**: Error boundary.

## 11. Interaction Specification
- **Trigger**: Click "Generate QR".
- **Action**: Opens QR Modal fetching from `POST /v1/attendance/generate-qr`.
- **Trigger**: Click "Export CSV".
- **Action**: Triggers CSV download using `GET /v1/events/:id/attendance/export`.

## 12. Form Specification
- **Manual Attendance Modal** (Inferred UX requirement for manual mark):
  - Field: User ID / Search.
  - Field: Status (`PRESENT`, `ABSENT`, `EXCUSED`).
  - Field: Method (defaults to `MANUAL`).
  - Action: Fires `POST /v1/events/:id/attendance/manual`.

## 13. Responsive / Adaptation
- **Desktop**: Full table layout.
- **Mobile**: Stacked list cards.

## 14. Accessibility
- **Role**: `main`.

## 15. Motion
- **Animation**: Default.

## 16. Security
- Relies on strict RBAC validation at the endpoint level.

## 17. Cache / Server State
- **Query Key**: `['attendance', eventId]`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: Pull to refresh or manual mutation.

## 18. Acceptance Criteria
- AC-WEB-09-01: Displays attendance records correctly.
- AC-WEB-09-02: QR generation works via POST.
- AC-WEB-09-03: Export CSV successfully triggers file download.

## 19. Specification Gaps / Open Decisions

- **OPEN UX ASSUMPTION**: The manual attendance UI (a modal to pick a user and mark them) is inferred from the existence of the `manual` POST endpoint. If a specific design exists for this, it must override this structural assumption.
