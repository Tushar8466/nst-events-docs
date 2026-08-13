# Mobile Event Detail Screen

## 1. Identity
- **Screen ID**: MOB-05
- **Screen Name**: Event Detail
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/events/[id]`

## 3. Role / Access
- **Visibility**: Public / Authenticated.
- **Access**: Any
- **UX Guard**: Action buttons gated by auth/role.

## 4. Entry Points
- `MOB-03` (Home), `MOB-04` (Events), or Deep Links.

## 5. Exit / Navigation
- **Register**: Navigates to `MOB-06` (Registration).
- **Scanner**: (Admins) Navigates to `MOB-09` (Attendance Scanner).
- **Team**: Navigates to `MOB-07` or `MOB-08` (Team Creation/Management).

## 6. Layout Hierarchy
```text
Screen (bg: #FFFFFF)
├── Header (Native back button, Share icon)
└── ScrollView
    ├── img (Event Banner placeholder)
    ├── div (Padding Wrapper)
    │   ├── h1 (title)
    │   ├── div (Clubs List: maps eventClubs array)
    │   ├── div (Meta row: start_time, end_time, location_name)
    │   ├── div (Live Stats: registration_count - from SSE)
    │   └── p (description)
    └── StickyFooter
        ├── h3 (Derived status: "Open for Registration" / "Closed")
        ├── Button (text: "Register Now")
        └── Button (text: "Scan Attendance" - ADMIN ONLY)
```

## 7. Component Map
- **ScrollView**: Native scroll.
- **StickyFooter**: Fixed bottom action bar.
- **Button**: Action triggers.

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Title | `title` | string | Raw Text |
| Organizing Clubs | `eventClubs` | array | Map `club_name` |
| Location | `locationName` | string | Raw Text |
| Schedule | `startTime`, `endTime` | string ISO | Formatted Time Range |
| Registration Count | `registrationCount` | number | Real-time live update via SSE |
| Registration Status | `status` | enum | From `GET /my-registration` |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/events/:id`
- **Purpose**: Fetch event details.
- **Status**: Exists (`02-api-routing-matrix.md` row 57).

- **Method**: `GET`
- **Path**: `/v1/events/:id/my-registration`
- **Purpose**: Fetch user's registration state.
- **Status**: Exists (`02-api-routing-matrix.md` row 82).

- **Method**: `GET`
- **Path**: `/v1/events/:id/live`
- **Purpose**: Establish SSE connection for live metric updates.
- **Status**: Exists (`02-api-routing-matrix.md` row 83).

## 10. UI States
- **Loading**: Skeleton layout.
- **Empty**: N/A.
- **Error**: 404 Not Found screen.

## 11. Interaction Specification
- **Trigger**: Load component.
- **Action**: Establishes SSE connection for `registrationCount` updates if supported by mobile networking library.
- **Trigger**: Tap "Register".
- **Action**: Navigate to `MOB-06`.

## 12. Form Specification
- **Not Applicable**.

## 13. Responsive / Adaptation
- **Mobile**: Vertical scroll with sticky footer for primary CTA.

## 14. Accessibility
- **Role**: `article`.
- **VoiceOver**: Read title and dates first.

## 15. Motion
- **Animation**: Native push transition.

## 16. Security
- Read-only viewing. Actions rely on strictly validated tokens.

## 17. Cache / Server State
- **Query Key**: `['event', eventId]`, `['event-my-registration', eventId]`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: Real-time via SSE.

## 18. Acceptance Criteria
- AC-MOB-05-01: Displays `locationName`, `maxCapacity`, and `eventClubs`.
- AC-MOB-05-02: Connects to SSE endpoint and updates `registrationCount`.
- AC-MOB-05-03: Excludes `attendance_count` per BE-CONFIRMED-012.

## 19. Specification Gaps / Open Decisions
- **DEFERRED DEPENDENCY (SSE TYPES)**: Just as with the web platform, `attendance_count` live updates are **DEFERRED** (BE-CONFIRMED-012) and must be excluded from this screen.
- **OPEN UX ASSUMPTION**: The sticky footer layout for the registration CTA is an inferred mobile UX best practice, not strictly specified by wireframes.
