# Approvals Screen

## 1. Identity
- **Screen ID**: WEB-12
- **Screen Name**: Event Approvals
- **Platform**: Web
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/admin/approvals`

## 3. Role / Access
- **Visibility**: FACULTY_MENTOR, PLATFORM_ADMIN
- **Access**: Restricted.
- **UX Guard**: Redirect if lacking permission.

## 4. Entry Points
- Sidebar Navigation.

## 5. Exit / Navigation
- **View Details**: Navigate to Event Detail view (read-only for review).

## 6. Layout Hierarchy
```text
SidebarNavigation
├── div
│   ├── BreadcrumbTrail
│   ├── h1 (text: "Event Approvals")
│   └── div (List)
│       └── Card (Approval Request Item)
│           ├── h3 (Event Title)
│           ├── p (Date & Time)
│           ├── div (Clubs List)
│           └── div (Actions)
│               ├── Button (text: "Reject", variant: destructive)
│               └── Button (text: "Approve", variant: primary)
```

## 7. Component Map
- **SidebarNavigation**: Context tree
- **Card**: Item wrapper
- **Button**: Action triggers

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Event Title | `title` | string | Raw Text |
| Date & Time | `start_time`, `end_time` | string ISO | Formatted date range |
| Clubs | `eventClubs` | array | Map `club_name` |
| Status | `state` | enum | Always `PENDING_APPROVAL` in this view |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/events`
- **Query Params**: `?filter_state=PENDING_APPROVAL` (and cursor pagination)
- **Purpose**: Fetch events awaiting approval.
- **Status**: Exists (`02-api-routing-matrix.md` row 56).

- **Method**: `POST`
- **Path**: `/v1/events/:id/approve`
- **Purpose**: Approve an event.
- **Status**: Exists (`02-api-routing-matrix.md` row 62).

- **Method**: `POST`
- **Path**: `/v1/events/:id/reject`
- **Purpose**: Reject an event.
- **Status**: Exists (`02-api-routing-matrix.md` row 63).

## 10. UI States
- **Loading**: Skeletons.
- **Empty**: "No events pending approval."
- **Error**: Error boundary.

## 11. Interaction Specification
- **Trigger**: Click "Approve".
- **Action**: Fire `POST /v1/events/:id/approve`.
- **Trigger**: Click "Reject".
- **Action**: Open modal for rejection reason (optional depending on API), then fire `POST /v1/events/:id/reject`.

## 12. Form Specification
- **Rejection Modal** (Inferred UX):
  - Field: Reason (text area). Needed if rejection triggers an informative notification.

## 13. Responsive / Adaptation
- **Desktop**: Full width list items.
- **Mobile**: Stacked cards.

## 14. Accessibility
- **Role**: `main`.

## 15. Motion
- **Animation**: Card exit animation on approve/reject success.

## 16. Security
- Only `FACULTY_MENTOR` and `PLATFORM_ADMIN` can fetch with `filter_state=PENDING_APPROVAL` or execute approval mutations.

## 17. Cache / Server State
- **Query Key**: `['events', 'pending']`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: On approve/reject mutation, remove item from list optimistically.

## 18. Acceptance Criteria
- AC-WEB-10-01: Accurately fetches events using `cursor`, `limit`, and `filter_state=PENDING_APPROVAL`.
- AC-WEB-10-02: Successfully fires approval/rejection mutations.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: The rejection action implies a modal to provide a rejection reason (which is standard for approvals). If the backend `POST /v1/events/:id/reject` endpoint does not accept a reason payload, this modal should be simplified to a standard confirmation dialog.
