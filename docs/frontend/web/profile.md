# Profile Screen

## 1. Identity
- **Screen ID**: WEB-08
- **Screen Name**: My Profile
- **Platform**: Web
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/profile`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any
- **UX Guard**: Redirect to `/login` if unauthenticated.

## 4. Entry Points
- Click Avatar / Name in Sidebar Navigation.

## 5. Exit / Navigation
- **Logout**: Logs out and redirects to `/login`.

## 6. Layout Hierarchy
```text
SidebarNavigation (bg: #111827)
├── div (bg: #F9FAFB, padding: space-6)
│   ├── BreadcrumbTrail
│   ├── h1 (text: "My Profile")
│   └── div (layout: grid, 2 columns)
│       ├── Card (Personal Details)
│       │   ├── img (Avatar Placeholder)
│       │   ├── h2 (User full_name)
│       │   ├── p (User email)
│       │   └── Badge (global_role)
│       └── Card (Account Actions)
│           ├── Button (text: "Logout", variant: destructive)
│           └── (Future items: notification preferences, etc.)
```

## 7. Component Map
- **SidebarNavigation**: Context tree (from 01-component-inventory)
- **Card**: Layout wrapper (from 07-component-library)
- **Badge**: Role indicator
- **Button**: Actions

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Name | `full_name` | string | Raw Text (fallback to "Set your name") |
| Email | `email` | string | Raw Text |
| Role | `global_role` | enum | UPPERCASE (`STUDENT`, `FACULTY_ADMIN`, `PLATFORM_ADMIN`) |

## 9. API Map
- **Method**: `GET`
- **Path**: `/users/me`
- **Purpose**: Fetch the currently authenticated user's profile.
- **Consumed Fields**: `id`, `email`, `full_name`, `global_role`.
- **Status**: Exists (`02-api-routing-matrix.md` row 86).

- **Method**: `POST`
- **Path**: `/auth/logout`
- **Purpose**: Invalidate server session.
- **Status**: Exists (`02-api-routing-matrix.md` row 47).

## 10. UI States
- **Loading**: Skeleton placeholders for text inside the card.
- **Empty**: N/A (cannot have empty profile if authenticated).
- **Error**: Standard Error Boundary with a retry button.

## 11. Interaction Specification
- **Trigger**: Click "Logout".
- **Action**: Fire `POST /auth/logout`, clear local JWT/cookies, clear QueryClient cache, and redirect to `/login`.

## 12. Form Specification
- **Not Applicable** (This screen is read-only. See Section 19 regarding `PATCH /users/me`).

## 13. Responsive / Adaptation
- **Desktop (>=1024px)**: 2-column grid.
- **Tablet/Mobile**: Stacked cards.

## 14. Accessibility
- **Role**: `main`.
- **Aria-Labels**: Descriptive labels for avatar placeholder and logout button.

## 15. Motion
- **Animation**: Default page transition.

## 16. Security
- Data fetched strictly via `/users/me` endpoint relying on JWT, ensuring users can only see their own private info (like email).

## 17. Cache / Server State
- **Query Key**: `['user', 'me']`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: On logout, must clear entire query cache.

## 18. Acceptance Criteria
- AC-WEB-08-01: Displays `full_name`, `email`, and `global_role` from `GET /users/me`.
- AC-WEB-08-02: Successfully logs out via `POST /auth/logout` and redirects.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: Placed the Logout button inside the Profile screen. Some designs place logout solely in the sidebar dropdown. I assumed a secondary access point here for thoroughness, but this should be reconciled with final design comps.
- **OPEN UX ASSUMPTION**: The `02-api-routing-matrix.md` lists `PATCH /users/me` as an existing, working endpoint (row 87). However, I have specified this profile screen as strictly read-only, inferring that profile editing is out of scope for the current UI iteration. If profile editing is in scope, a form UI must be added to this screen.
