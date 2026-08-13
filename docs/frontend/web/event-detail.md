# Event Detail Screen

## 1. Identity
- **Screen ID**: WEB-04
- **Screen Name**: Event Detail
- **Platform**: Web
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/events/[id]`

## 3. Role / Access
- **Visibility**: Public or Authenticated (depending on event visibility rules).
- **Access**: Any
- **UX Guard**: None for viewing, but interacting (registering) requires Auth.

## 4. Entry Points
- Click from Events Directory (`WEB-03`).
- Deep link from Notifications or external sharing.

## 5. Exit / Navigation
- **Register**: Navigates to Registration Confirmation (`WEB-05`).
- **Manage Attendance**: (Admins only) Navigates to Attendance Management (`WEB-09`).
- **Teams**: Navigates to Teams directory (`WEB-06`).

## 6. Layout Hierarchy
```text
SidebarNavigation
├── div
│   ├── BreadcrumbTrail
│   └── div (2-Column Layout)
│       ├── Main Column
│       │   ├── h1 (Event title)
│       │   ├── div (Clubs List: maps eventClubs array)
│       │   ├── div (Meta row: start_time, end_time, location_name)
│       │   ├── div (Live Stats: registration_count - from SSE)
│       │   └── p (description)
│       └── Sidebar Column
│           └── Card (Action Box)
│               ├── h3 (Derived status: "Open for Registration" / "Closed")
│               ├── span (Spots left: max_capacity - registration_count)
│               ├── Button (text: "Register Now")
│               └── Button (text: "Manage Attendance" - ADMIN ONLY)
```

## 7. Component Map
- **SidebarNavigation**: Context tree
- **Card**: Action box wrapper
- **Button**: Action triggers

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Title | `title` | string | Raw Text |
| Organizing Clubs | `eventClubs` | array | Map `club_name` for each entry |
| Location | `locationName` | string | Raw Text |
| Schedule | `startTime`, `endTime` | string ISO | Formatted Time Range |
| Registration Count | `registrationCount` | number | Real-time live update via SSE |
| Availability | `maxCapacity`, `registrationCount`, `state` | mixed | Client-side derived rule |
| Registration Status | `status` | enum | From `GET /my-registration` (`REGISTERED`, `WAITLISTED`, `CANCELLED`) |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/events/:id`
- **Purpose**: Fetch core event details.
- **Status**: Exists (`02-api-routing-matrix.md` row 57).

- **Method**: `GET`
- **Path**: `/v1/events/:id/my-registration`
- **Purpose**: Fetch user's registration status.
- **Status**: Exists (`02-api-routing-matrix.md` row 82).

- **Method**: `GET`
- **Path**: `/v1/events/:id/live`
- **Purpose**: Establish SSE connection for live updates.
- **Status**: Exists (`02-api-routing-matrix.md` row 83).

## 10. UI States
- **Loading**: Full page skeleton layout.
- **Empty**: N/A (Event must exist).
- **Error**: 404 Not Found boundary.

## 11. Interaction Specification
- **Trigger**: Load component.
- **Action**: Establishes SSE connection. Merges `registrationCount` SSE payloads into local query cache.
- **Trigger**: Click "Register".
- **Action**: Navigate to `/register` sub-route.

## 12. Form Specification
- **Not Applicable** (Viewing details).

## 13. Responsive / Adaptation
- **Desktop (>=1024px)**: Main details on left, Action Box on right sidebar.
- **Tablet/Mobile**: Single column stack, Action Box at the top or sticky bottom.

## 14. Accessibility
- **Role**: `article`.

## 15. Motion
- **Animation**: Smooth number ticker transition for `registrationCount` updates.

## 16. Security
- Read-only data view. Administrative action buttons must be gated strictly by user's `global_role` or club membership.

## 17. Cache / Server State
- **Query Key**: `['event', eventId]`, `['event-my-registration', eventId]`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: Receives live updates via SSE, so cache is kept fresh automatically.

## 18. Acceptance Criteria
- AC-WEB-04-01: Displays `locationName`, `maxCapacity`, and iterating `eventClubs`.
- AC-WEB-04-02: Receives and renders live `registrationCount` via SSE payload.
- AC-WEB-04-03: Excludes any live `attendance_count` metric due to BE-CONFIRMED-012.

## 19. Specification Gaps / Open Decisions
- **DEFERRED DEPENDENCY (SSE TYPES)**: While the `/live` SSE endpoint exists, it only broadcasts `registrationCount` and `heartbeat`. The `attendance_count` and other live metrics originally planned for this screen are **DEFERRED** (BE-CONFIRMED-012) and cannot be implemented in V1.
- **OPEN UX ASSUMPTION**: The layout (Main Column + Action Box Sidebar) is a structural UX inference. It is a highly standard pattern for event platforms, but lacks explicit wireframe validation.
