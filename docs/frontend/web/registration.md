# Registration Screen

## 1. Identity
- **Screen ID**: WEB-05
- **Screen Name**: Registration Confirmation
- **Platform**: Web
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/events/[id]/register`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any user not yet registered for the event.
- **UX Guard**: Redirect back to Event Detail if already registered.

## 4. Entry Points
- Click "Register" from Event Detail (`WEB-04`).

## 5. Exit / Navigation
- **Back / Cancel**: Returns to Event Detail (`WEB-04`).
- **Success**: Redirects to Event Detail with a success toast.

## 6. Layout Hierarchy
```text
SidebarNavigation (bg: #111827)
├── div (bg: #F9FAFB, padding: space-6)
│   ├── BreadcrumbTrail
│   ├── h1 (text: "Confirm Registration")
│   └── Card (Confirmation Box)
│       ├── p (text: "You are registering for: {Event Title}")
│       └── div (flex)
│           ├── Button (text: "Cancel", variant: outline)
│           └── Button (text: "Confirm Registration", variant: primary)
```

## 7. Component Map
- **SidebarNavigation**: Context tree (from 01-component-inventory)
- **BreadcrumbTrail**: Navigation trace
- **Card**: Container (from 07-component-library)
- **Button**: Action triggers

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Event Title | `title` | string | Raw Text (fetched from parent Event cache) |

## 9. API Map
- **Method**: `POST`
- **Path**: `/v1/events/:id/register`
- **Purpose**: Register the current user for the event.
- **Consumed Fields**: None (uses authenticated JWT context).
- **Status**: Exists (`02-api-routing-matrix.md` row 77).

## 10. UI States
- **Loading**: Inline spinner on "Confirm Registration" button. Disabled state for "Cancel" button.
- **Empty**: N/A.
- **Error**: Standard toast or inline alert (e.g. "Event is full", "Already registered").

## 11. Interaction Specification
- **Trigger**: Click "Confirm Registration".
- **Action**: Fire `POST /v1/events/:id/register` mutation. On success, invalidate registration queries and navigate back.
- **Trigger**: Click "Cancel".
- **Action**: Navigate back without mutation.

## 12. Form Specification
- **Not Applicable** (This is a 1-click confirmation view, no input fields).

## 13. Responsive / Adaptation
- **Desktop (>=1024px)**: Center card in viewport.
- **Tablet/Mobile**: Full width card, stacked buttons if space is constrained.

## 14. Accessibility
- **Role**: `main`.
- **Keyboard Interaction**: Focusable buttons, clear focus states on Confirm vs Cancel.

## 15. Motion
- **Animation**: Default transitions.

## 16. Security
- Mutation protected by auth token. Backend handles duplicate registration rejection.

## 17. Cache / Server State
- **Mutation Key**: `['event-register', eventId]`
- **Invalidation**: Successfully mutating must invalidate `['event-my-registration', eventId]` and `['event', eventId]` to refresh UI state elsewhere.

## 18. Acceptance Criteria
- AC-WEB-05-01: Displays the correct event `title` pulled from cache.
- AC-WEB-05-02: Successfully fires `POST /v1/events/:id/register`.
- AC-WEB-05-03: Redirects to Event Detail on success or cancellation.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: This spec outlines a dedicated full-page route `/(app)/events/[id]/register` for registration confirmation. While standard, some apps handle 1-click registration via an inline modal on the Event Detail screen itself. This spec assumes the dedicated route path is maintained as per the original screen list.
