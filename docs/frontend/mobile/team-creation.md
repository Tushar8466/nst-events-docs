# Mobile Team Creation Screen

## 1. Identity
- **Screen ID**: MOB-07
- **Screen Name**: Team Creation
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/events/[id]/team-creation`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Must be registered for the event.
- **UX Guard**: Redirect if not registered.

## 4. Entry Points
- From Event Detail (`MOB-05`).

## 5. Exit / Navigation
- **Success**: Navigates to Team Management (`MOB-08`).
- **Cancel**: Returns to Event Detail.

## 6. Layout Hierarchy
```text
Screen (bg: #FFFFFF)
├── Header (text: "Create a Team")
└── ScrollView
    ├── div (Instructions)
    │   └── p (text: "Enter a name for your team. You can invite members later.")
    └── form
        ├── input (label: "Team Name")
        └── Button (text: "Create Team", variant: primary)
```

## 7. Component Map
- **ScrollView**: Native layout
- **Input**: Form text field
- **Button**: Submit action

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Team Name | `team_name` | Local State | String (Input) |

## 9. API Map
- **Method**: `POST`
- **Path**: `/v1/events/:id/teams`
- **Purpose**: Create a new team for the event.
- **Status**: Exists (`02-api-routing-matrix.md` row 79). Verified to be implemented in `registrationsRouter`.

## 10. UI States
- **Loading**: Spinner on submit button.
- **Empty**: N/A.
- **Error**: Form validation errors or backend rejection toast.

## 11. Interaction Specification
- **Trigger**: Tap "Create Team".
- **Action**: Fire `POST /v1/events/:id/teams` mutation. On success, invalidate event queries and navigate to the team view.

## 12. Form Specification
- **Fields**:
  - `team_name`: string, required, min 3 chars, max 50 chars.

## 13. Responsive / Adaptation
- **Mobile**: Full screen native form.

## 14. Accessibility
- **Role**: `form`.
- **Keyboard**: Ensure focus moves to input automatically.

## 15. Motion
- **Animation**: Keyboard avoid view transitions.

## 16. Security
- User must hold a valid JWT and be verified as an event participant.

## 17. Cache / Server State
- **Mutation Key**: `['create-team', eventId]`
- **Invalidation**: Invalidates `['event', eventId]` to update registration/team status.

## 18. Acceptance Criteria
- AC-MOB-07-01: Successfully submits the form to `POST /v1/events/:id/teams`.
- AC-MOB-07-02: Navigates to Team Management on success.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: Section 3 assumes that a user must already be registered for the event before they can create a team. While logically sound for most event platforms, this rule is not explicitly documented in the API routing matrix or data contract. If the backend allows team creation *during* registration or independent of it, the UX flow must be updated.
