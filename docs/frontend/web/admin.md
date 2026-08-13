# Admin Hub Screen

## 1. Identity
- **Screen ID**: WEB-11
- **Screen Name**: Admin Hub
- **Platform**: Web
- **Implementation Status**: PLANNED
- **Specification Status**: READY WITH GAPS
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/admin`

## 3. Role / Access
- **Visibility**: PLATFORM_ADMIN
- **Access**: PLATFORM_ADMIN
- **UX Guard**: Redirect to `/dashboard` if user lacks PLATFORM_ADMIN role.

## 4. Entry Points
- Sidebar Navigation.

## 5. Exit / Navigation
- **Audit Logs**: (Active)
- **Point Adjustments**: (Deferred)
- **User Management**: Navigates to `/(app)/admin/users` (Active, see WEB-14).

## 6. Layout Hierarchy
```text
SidebarNavigation (bg: #111827)
├── div (bg: #F9FAFB, padding: space-6)
│   ├── BreadcrumbTrail
│   ├── h1 (text: "Platform Administration")
│   └── div (layout: grid, 2 columns)
│       ├── Card (Quick Actions)
│       │   ├── h2 (text: "Point Adjustments")
│       │   ├── p (text: "Manually adjust student or club points.")
│       │   └── Button (text: "Adjust Points" - DISABLED/BLOCKED)
│       └── Card (Audit Logs)
│           ├── h2 (text: "Recent Audit Logs")
│           └── table
│               ├── tr (header: Action, Actor ID, Target, Time)
│               └── tr (log row)
```

## 7. Component Map
- **SidebarNavigation**: Context tree (from 01-component-inventory)
- **Card**: Widget wrapper (from 07-component-library)
- **Table**: Data display
- **Button**: Action trigger

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Action | `action` | string | Raw text (e.g. "EVENT_APPROVED") |
| Actor ID | `actorId` | string | Raw text |
| Target Entity | `entityType`, `entityId` | string | `{entityType} - {entityId}` |
| Time | `createdAt` | string ISO | Relative time formatting |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/admin/audit-logs`
- **Purpose**: Fetch recent system actions for display.
- **Status**: **IMPLEMENTED**. (Phase 21J)

- **Method**: `N/A`
- **Path**: `N/A (Deferred)`
- **Purpose**: Point Adjustments are DEFERRED — NOT V1.
- **Status**: **DEFERRED — NOT V1**. Leaderboard scores are derived/immutable in V1. No manual user or club adjustment endpoint exists.

## 10. UI States
- **Loading**: Skeleton blocks.
- **Empty**: N/A (Admin hub).
- **Error**: Error boundary.
- **Deferred State**: Show "Coming Soon" states for Point Adjustments, as it is deferred to V2.

## 11. Interaction Specification
- **Trigger**: Click "Adjust Points" (when unblocked).
- **Action**: Opens Point Adjustment modal.

## 12. Form Specification
- **Point Adjustment Modal** (Future):
  - Fields: Target (User/Club ID), Amount (number), Reason (string).
  - Depends on BE-CONFIRMED-008.

## 13. Responsive / Adaptation
- **Desktop (>=1024px)**: 2-column grid for widgets.
- **Tablet (768px - 1023px)**: 1-column stack.
- **Mobile (<768px)**: 1-column stack.

## 14. Accessibility
- **Role**: `main` for page content.

## 15. Motion
- **Animation**: Default transitions.

## 16. Security
- Entire route and underlying APIs must enforce strictly `PLATFORM_ADMIN`.

## 17. Cache / Server State
- **Query Key**: `['admin-audit-logs']`
- **Stale Behavior**: N/A until endpoint built.
- **Invalidation**: N/A.

## 18. Acceptance Criteria
- AC-WEB-11-01: Audit log table correctly populates with `GET /v1/admin/audit-logs`.

## 19. Specification Gaps / Open Decisions
- **DEFERRED DEPENDENCIES**: Point adjustment functions are explicitly deferred by BE-CONFIRMED-008 (design decision).
