# Mobile Profile Screen

## 1. Identity
- **Screen ID**: MOB-11
- **Screen Name**: Profile
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(tabs)/profile`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any
- **UX Guard**: Redirect to Auth if unauthenticated.

## 4. Entry Points
- Bottom Tab Bar ("Profile").

## 5. Exit / Navigation
- **Logout**: Logs out and redirects to Login screen (`MOB-02`).

## 6. Layout Hierarchy
```text
Screen (bg: #F9FAFB)
├── Header (text: "Profile")
└── ScrollView
    ├── div (User Info Card)
    │   ├── img (Avatar placeholder)
    │   ├── h2 (User full_name)
    │   ├── p (User email)
    │   └── Badge (global_role)
    └── div (Settings List)
        ├── Card (Row: Push Notifications Toggle)
        └── Button (text: "Logout", variant: destructive)
```

## 7. Component Map
- **Card**: Settings row wrapper
- **Badge**: Role visualizer
- **Button**: Action trigger

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Name | `full_name` | string | Raw Text |
| Email | `email` | string | Raw Text |
| Role | `global_role` | enum | UPPERCASE (`STUDENT`, `FACULTY_ADMIN`, `PLATFORM_ADMIN`) |

## 9. API Map
- **Method**: `GET`
- **Path**: `/users/me`
- **Purpose**: Fetch current user profile details.
- **Status**: Exists (`02-api-routing-matrix.md` row 86).

- **Method**: `POST`
- **Path**: `/auth/logout`
- **Purpose**: Log out the user.
- **Status**: Exists (`02-api-routing-matrix.md` row 47).

## 10. UI States
- **Loading**: Skeletons.
- **Empty**: N/A.
- **Error**: Error boundary.

## 11. Interaction Specification
- **Trigger**: Tap "Logout".
- **Action**: Fire `POST /auth/logout`, clear stored JWTs, clear query cache, and redirect to `MOB-02`.

## 12. Form Specification
- **Not Applicable** (Read-only view).

## 13. Responsive / Adaptation
- **Mobile**: Native vertical scroll layout.

## 14. Accessibility
- **Role**: `main`.

## 15. Motion
- **Animation**: Default tab switch.

## 16. Security
- Read-only private data for current user token.

## 17. Cache / Server State
- **Query Key**: `['user', 'me']`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: Cleared on logout.

## 18. Acceptance Criteria
- AC-MOB-11-01: Displays `full_name`, `email`, and `global_role` correctly.
- AC-MOB-11-02: Logout clears tokens and redirects.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: Profile editing (`PATCH /users/me`) is not included on this screen, assuming profile edits happen during onboarding or via web. If mobile editing is required, form fields must be added.
