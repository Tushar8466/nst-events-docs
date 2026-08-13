# Mobile Team Management Screen

## 1. Identity
- **Screen ID**: MOB-08
- **Screen Name**: Team Management
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: READY
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/events/[id]/team-management`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Must be a member of the team.
- **UX Guard**: Redirect back to Event Detail if user is not in a team.

## 4. Entry Points
- From Event Detail (`MOB-05`).

## 5. Exit / Navigation
- **Back**: Returns to Event Detail.

## 6. Layout Hierarchy
```text
Screen (bg: #F9FAFB)
├── Header (text: "My Team")
└── ScrollView
    ├── div (Team Info)
    │   └── h2 (Team Name)
    ├── div (Members List)
    │   └── FlatList
    │       └── Card (Member Item)
    │           ├── img (Avatar placeholder)
    │           └── span (User full_name)
    └── div (Actions)
        └── Button (text: "Leave Team", variant: destructive)
```

## 7. Component Map
- **ScrollView**: Native scroll layout
- **FlatList**: Native list for members
- **Card**: Item wrapper
- **Button**: Action trigger

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Team Name | `team_name` | string | Raw Text |
| Member List | `members` | array | Map over array, extract `full_name` property from User object |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/events/:id/teams` (or equivalent "My Team" endpoint)
- **Purpose**: Fetch the team details and member list.
- **Status**: **IMPLEMENTED**. (Phase 21J)

- **Method**: `DELETE`
- **Path**: `/v1/teams/:teamId/leave`
- **Purpose**: Leave the team.
- **Status**: Exists (`02-api-routing-matrix.md` row 85), but unusable without the team fetching endpoint.

## 10. UI States
- **Loading**: Skeletons.
- **Empty**: N/A (must be in a team to see this).
- **Error**: Error boundary.
- **Blocked State**: Show global alert banner that Team Management is currently unbuildable.

## 11. Interaction Specification
- **Trigger**: Tap "Leave Team".
- **Action**: Opens confirmation alert. If confirmed, fire `DELETE /v1/teams/:teamId/leave` mutation.

## 12. Form Specification
- **Not Applicable**.

## 13. Responsive / Adaptation
- **Mobile**: Native vertical layout.

## 14. Accessibility
- **Role**: `main`.
- **Aria-Labels**: Ensure destructive action is announced properly.

## 15. Motion
- **Animation**: Native push/pop.

## 16. Security
- API enforces that a user can only view their own team and leave it.

## 17. Cache / Server State
- **Query Key**: `['event-team', eventId]`
- **Stale Behavior**: N/A until endpoint exists.
- **Invalidation**: N/A.

## 18. Acceptance Criteria
- AC-MOB-08-01: Cannot be fully verified until a `GET` endpoint for teams is built (BE-CONFIRMED-004).

## 19. Specification Gaps / Open Decisions

- **OPEN UX ASSUMPTION**: I have included a "Leave Team" button based on the existence of the `leave` endpoint. The layout structurally infers a standard list of members, which aligns with the `Team` object in `DATA_CONTRACT.md`.
