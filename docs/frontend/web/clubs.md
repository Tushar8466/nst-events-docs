# Clubs Directory Screen

## 1. Identity
- **Screen ID**: WEB-10
- **Screen Name**: Clubs Directory
- **Platform**: Web
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/clubs`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any
- **UX Guard**: Redirect to Auth if unauthenticated.

## 4. Entry Points
- Sidebar Navigation -> "Clubs".

## 5. Exit / Navigation
- **Click Club Card**: Navigates to `/(app)/clubs/[id]` (Club Detail).
- **Click "Create Club"**: (Platform Admins only) Navigates to Club Creation modal/screen.

## 6. Layout Hierarchy
```text
SidebarNavigation
├── div
│   ├── BreadcrumbTrail
│   ├── div (Header)
│   │   ├── h1 (text: "Clubs Directory")
│   │   ├── input (Search bar)
│   │   └── Button (text: "Create Club" - ADMIN ONLY)
│   └── div (Grid Layout)
│       └── Card (Club Card)
│           ├── img (banner_url placeholder)
│           ├── h2 (name)
│           ├── p (description)
│           └── div (Footer)
│               ├── span (status badge)
│               ├── span (event_count)
│               └── span (members count via array length)
```

## 7. Component Map
- **SidebarNavigation**: Context tree
- **Card**: Club container
- **Badge**: Status visualizer
- **Input**: Search field

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Banner | `bannerUrl` | string | Image src (fallback to gray block) |
| Name | `name` | string | Raw Text |
| Description | `description` | string | Truncated Text |
| Status | `status` | enum | Badge (`ACTIVE`, `INACTIVE`, `DISSOLVED`) |
| Event Count | `event_count` | number | `{count} Events` |
| Member Count | `members` | array | `members.length` Members |

## 9. API Map
- **Method**: `GET`
- **Path**: `/clubs` (or `/clubs/search` if using search)
- **Purpose**: Fetch all clubs or search them.
- **Status**: Exists (`02-api-routing-matrix.md` rows 48-49).

## 10. UI States
- **Loading**: Grid of skeleton cards.
- **Empty**: "No clubs found matching your criteria."
- **Error**: Error boundary.

## 11. Interaction Specification
- **Trigger**: Click Club Card.
- **Action**: Navigate to Club Detail page.

## 12. Form Specification
- **Search Bar**: Debounced text input triggering refetch against `/clubs/search` (or local filter if fetching all).

## 13. Responsive / Adaptation
- **Desktop (>=1024px)**: 3-column grid.
- **Tablet (768px - 1023px)**: 2-column grid.
- **Mobile (<768px)**: 1-column grid.

## 14. Accessibility
- **Role**: `main`.
- **Aria-Labels**: Card links should read "View {Club Name} details".

## 15. Motion
- **Animation**: Staggered fade-in for cards.

## 16. Security
- "Create Club" button must be strictly conditionally rendered only if `global_role == PLATFORM_ADMIN`.

## 17. Cache / Server State
- **Query Key**: `['clubs', list]`
- **Stale Behavior**: 10 minutes.
- **Invalidation**: Manual refresh.

## 18. Acceptance Criteria
- AC-WEB-03-01: Correctly maps `event_count` and `members` array from the `GET /clubs` endpoint.
- AC-WEB-03-02: Status badge correctly uses `ACTIVE`, `INACTIVE`, or `DISSOLVED`.
- AC-WEB-03-03: "Create Club" is only visible to PLATFORM_ADMIN.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: The specific query parameters for `/clubs/search` (e.g., `?q=...` or `?name=...`) are not explicitly defined in the routing matrix. The frontend will need to match whatever the backend schema currently enforces for that route.
