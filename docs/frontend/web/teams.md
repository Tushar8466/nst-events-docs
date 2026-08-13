# Teams Screen

## 1. Identity
- **Screen ID**: WEB-06
- **Screen Name**: Teams
- **Platform**: Web
- **Implementation Status**: PLANNED
- **Specification Status**: READY
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/events/[id]/teams`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any
- **UX Guard**: Redirect to `/login` if unauthenticated.

## 4. Entry Points
- From `WEB-04` (Event Detail) if the event allows team registration.

## 5. Exit / Navigation
- **Back**: Returns to `WEB-04` (Event Detail).

## 6. Layout Hierarchy
```text
SidebarNavigation (bg: #111827)
├── ContextSwitcher
├── div (bg: #F9FAFB, padding: space-6)
│   ├── BreadcrumbTrail
│   ├── div (Header)
│   │   ├── h1 (text: "Teams")
│   │   └── Button (text: "Create Team")
│   └── div (layout: grid)
│       └── Card (Team Item)
│           ├── h2 (Team Name)
│           ├── ul (Member List)
│           │   └── li (User full_name)
│           └── div (Actions)
│               └── Button (Join / Leave)
```

## 7. Component Map
- **SidebarNavigation**: Context tree (from 01-component-inventory)
- **BreadcrumbTrail**: Routing trace (from 01-component-inventory)
- **Card**: Wrapper (from 07-component-library)

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Team Name | `team_name` | string | Raw Text |
| Member List | `members` | array | Map over array, extract `full_name` property from User object |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/events/:id/teams`
- **Purpose**: Fetch all teams for the event.
- **Status**: **IMPLEMENTED**. (Phase 21J)

- **Method**: `POST`
- **Path**: `/v1/events/:id/teams/:teamId/join` (or similar)
- **Purpose**: Join a specific team.
- **Status**: Exists (`/:id/join`), but unusable without the list of teams.

## 10. UI States
- **Loading**: Skeleton cards for teams.
- **Empty**: "No teams formed yet. Be the first to create one!"
- **Error**: Standard Error Boundary.
- **Blocked**: Render a "Coming Soon" or "Blocked by Backend" overlay since `GET /v1/events/:id/teams` is missing.

## 11. Interaction Specification
- **Trigger**: Click "Join"
- **Action**: Fire `POST /v1/teams/:teamId/join`. Show toast on success.

## 12. Form Specification
- **Not Applicable** for the list view (creation would be a separate modal).

## 13. Responsive / Adaptation
- **Desktop (>=1024px)**: Grid layout (3 columns).
- **Tablet (768px - 1023px)**: Grid layout (2 columns).
- **Mobile (<768px)**: 1 column stack.

## 14. Accessibility
- **Role**: `main` for page content.
- **Aria-Labels**: Ensure buttons read "Join [Team Name]".

## 15. Motion
- **Animation**: Default transitions for list items entering.

## 16. Security
- API checks user is registered for the event before allowing join.

## 17. Cache / Server State
- **Query Key**: `['event-teams', eventId]`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: On successful join/leave mutation.

## 18. Acceptance Criteria
- AC-WEB-06-01: Cannot be fully implemented or verified until `GET /v1/events/:id/teams` is built (BE-CONFIRMED-004).
- AC-WEB-06-02: User can join a team successfully via `POST` once list is available.

## 19. Specification Gaps / Open Decisions

