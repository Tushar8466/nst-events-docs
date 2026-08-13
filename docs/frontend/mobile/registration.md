# Mobile Registration Confirmation Screen

## 1. Identity
- **Screen ID**: MOB-06
- **Screen Name**: Registration Confirmation
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/events/[id]/register`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any user not yet registered.
- **UX Guard**: Redirect back if already registered.

## 4. Entry Points
- "Register" button on Event Detail (`MOB-05`).

## 5. Exit / Navigation
- **Success / Cancel**: Returns to Event Detail (`MOB-05`).

## 6. Layout Hierarchy
```text
Screen (bg: #FFFFFF)
├── Header (text: "Confirm Registration")
└── div (Padding Wrapper)
    ├── Card (Summary Box)
    │   ├── h2 (Event title)
    │   └── p (text: "Tap confirm to reserve your spot for this event.")
    └── div (Action Buttons)
        ├── Button (text: "Confirm Registration", variant: primary)
        └── Button (text: "Cancel", variant: ghost)
```

## 7. Component Map
- **Card**: Summary box wrapper
- **Button**: Action triggers

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Event Title | `title` | string | Raw Text (from event query cache) |

## 9. API Map
- **Method**: `POST`
- **Path**: `/v1/events/:id/register`
- **Purpose**: Register the user for the event.
- **Status**: Exists (`02-api-routing-matrix.md` row 77).

## 10. UI States
- **Loading**: Button loading spinner.
- **Empty**: N/A.
- **Error**: Rejection toast (e.g. "Event full").

## 11. Interaction Specification
- **Trigger**: Tap "Confirm Registration".
- **Action**: Fire `POST /v1/events/:id/register`. On success, invalidate event queries and navigate back.

## 12. Form Specification
- **Not Applicable** (1-tap confirmation).

## 13. Responsive / Adaptation
- **Mobile**: Full screen modal layout.

## 14. Accessibility
- **Role**: `dialog`.

## 15. Motion
- **Animation**: Slide up modal transition.

## 16. Security
- Token-authenticated API mutation.

## 17. Cache / Server State
- **Mutation Key**: `['register-event', eventId]`
- **Invalidation**: Invalidates `['event', eventId]` and `['event-my-registration', eventId]`.

## 18. Acceptance Criteria
- AC-MOB-06-01: Fires POST mutation to `/v1/events/:id/register`.
- AC-MOB-06-02: Successfully returns to Event Detail upon completion.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: The registration action uses a dedicated full-page screen route `/(app)/events/[id]/register`. If an inline bottom sheet modal is preferred by the UI team, this route can be converted to an action sheet.
