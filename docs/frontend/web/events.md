# Events Screen

## 1. Identity
- **Screen ID**: WEB-03
- **Screen Name**: Events List
- **Platform**: Web (Dashboard)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/events`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any
- **UX Guard**: Redirect to `/login` if unauthenticated.

## 4. Entry Points
- Global Sidebar Navigation -> "Events"

## 5. Exit / Navigation
- **Row/Card Click**: Navigates to `/(app)/event-detail`

## 6. Layout Hierarchy
```text
SidebarNavigation (bg: #111827)
├── ContextSwitcher
├── div (bg: #F9FAFB, padding: space-6)
│   ├── BreadcrumbTrail
│   ├── h1 (text: "Events", color: #111827, typography: text-2xl, font-weight: 600)
│   └── div (layout: grid)
│       └── Card (bg: #FFFFFF, border: rounded-lg, margin-top: space-6)
│           ├── h2 (Title, typography: text-lg, font-weight: 600)
│           ├── span (Date & Time, typography: text-sm)
│           ├── span (Location, typography: text-sm)
│           ├── Badge (Derived Open/Closed Status)
│           └── span (Capacity, typography: text-sm)
```

## 7. Component Map
- **SidebarNavigation**: Context tree (from 01-component-inventory)
- **ContextSwitcher**: Role dropdown (from 01-component-inventory)
- **BreadcrumbTrail**: Routing trace (from 01-component-inventory)
- **Card**: Wrapper (from 07-component-library)
- **Badge**: Status indicator (from 07-component-library)

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Card Title | `title` | string | Raw Text |
| Date & Time | `startTime`, `endTime` | string ISO | Raw Text |
| Location | `locationName` | string | Raw Text (omit if null) |
| Status Badge | `state`, `maxCapacity`, `registrationCount`, `isLocked` | boolean derived | UPPERCASE (OPEN, CLOSED) derived client-side |
| Capacity | `registrationCount`, `maxCapacity` | number | `{registration_count} / {max_capacity}` (if max_capacity null, show "Unlimited") |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/events?filter_state=PUBLISHED&cursor=&limit=20`
- **Purpose**: Fetch list of all events visible to the active role context.
- **Consumed Fields**: `id`, `title`, `startTime`, `endTime`, `locationName`, `state`, `maxCapacity`, `registrationCount`, `isLocked`
- **Cache Key**: `['events', 'list']`

## 10. UI States
- **Loading**: Render `Card` components containing skeleton rows (bg: `#F3F4F6` pulsing).
- **Empty**: Render standard empty state inside `Card`. Text: "No events found".
- **Error**: Standard Error Boundary. Text: "Failed to load events." Action: "Retry" button.
- **Unauthorized/Forbidden**: Redirects instantly per UX Guard.

## 11. Interaction Specification
- **Trigger**: Click on any event card.
- **Action**: Navigate to `/(app)/event-detail`.

## 12. Form Specification
- **Not Applicable** (This is a read-only list screen).

## 13. Responsive / Adaptation
- **Desktop (>=1024px)**: Grid layout for cards. Sidebar pinned.
- **Tablet (768px - 1023px)**: Sidebar collapses. 
- **Mobile (<768px)**: Single column stack for cards.

## 14. Accessibility
- **Role**: `main` for page content.
- **Keyboard Interaction**: `Tab` through cards. `Enter` to select card.
- **Focus**: Distinct focus ring on interactive cards.

## 15. Motion
- **Animation**: Default platform transitions.

## 16. Security
- Only fields returned by `GET /v1/events` are rendered.
- `qr_secret` is never exposed or requested on this screen.

## 17. Cache / Server State
- **Query Key**: `['events', 'list']`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: Invalidated automatically when mutations succeed.

## 18. Acceptance Criteria
1. Screen mounts and fetches data from `GET /v1/events`.
2. Loading skeleton is visible during the network request.
3. Event Cards accurately render all mapped fields.
4. Clicking a card successfully navigates to `/(app)/event-detail`.
5. Empty state is correctly shown if the array is empty.

## 19. Specification Gaps / Open Decisions
- **None**: All fields, routing, and design tokens are fully resolved from existing documentation or explicitly finalized as product decisions.
