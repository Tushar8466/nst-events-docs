# Mobile Notifications Screen

## 1. Identity
- **Screen ID**: MOB-10
- **Screen Name**: Notifications
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/notifications`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any
- **UX Guard**: Redirect to Auth if unauthenticated.

## 4. Entry Points
- Bell icon in Header.

## 5. Exit / Navigation
- **Click Notification**: Deep link to target event or screen.

## 6. Layout Hierarchy
```text
Screen (bg: #F9FAFB)
├── Header (text: "Notifications")
└── FlatList
    └── Card (Notification Item)
        ├── div (Unread Indicator Dot)
        ├── h3 (title)
        ├── p (body)
        └── span (created_at)
```

## 7. Component Map
- **FlatList**: Native scroll list
- **Card**: Notification wrapper

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Title | `title` | string | Raw Text |
| Body | `body` | string | Raw Text |
| Time | `created_at` | string ISO | Relative time formatting |
| Unread State | `read_at` | string ISO (nullable) | If `read_at` is null, render unread indicator dot |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/notifications`
- **Purpose**: Fetch user notifications.
- **Status**: Exists (`02-api-routing-matrix.md` row 72).

- **Method**: `PATCH`
- **Path**: `/v1/notifications/:id/read`
- **Purpose**: Mark notification as read.
- **Status**: Exists (`02-api-routing-matrix.md` row 74).

## 10. UI States
- **Loading**: Skeleton list items.
- **Empty**: "No notifications yet."
- **Error**: Error boundary.

## 11. Interaction Specification
- **Trigger**: Tap Notification.
- **Action**: Fire `PATCH /v1/notifications/:id/read` mutation if unread, then deep link to target screen based on notification `type`.

## 12. Form Specification
- **Not Applicable**.

## 13. Responsive / Adaptation
- **Mobile**: Native vertical list.

## 14. Accessibility
- **Role**: `list`.

## 15. Motion
- **Animation**: Native push transition.

## 16. Security
- Private user notifications only.

## 17. Cache / Server State
- **Query Key**: `['notifications']`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: Optimistically update local state on read mutation.

## 18. Acceptance Criteria
- AC-MOB-10-01: Displays notifications list and derives unread status from `read_at == null`.
- AC-MOB-10-02: Marks items read via PATCH.

## 19. Specification Gaps / Open Decisions
- **BLOCKED DEPENDENCY (ANNOUNCEMENTS)**: Global Announcements are blocked by BE-CONFIRMED-011. This screen strictly handles user-specific notifications (`GET /v1/notifications`).
- **OPEN UX ASSUMPTION**: Deep-linking map based on notification `type` is standard UX, but specific route mappings per type are open implementation details.
