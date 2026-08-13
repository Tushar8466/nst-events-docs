# Mobile Onboarding Screen

## 1. Identity
- **Screen ID**: MOB-01
- **Screen Name**: User Onboarding
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `02-api-routing-matrix.md`, `DATA_CONTRACT.md`

## 2. Route
- **Route**: `/(app)/onboarding`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: First-time logins only.
- **UX Guard**: Automatically skipped if user profile is already fully populated.

## 4. Entry Points
- Redirected from `MOB-02` (Login) upon first successful authentication.

## 5. Exit / Navigation
- **Success**: Navigates to `/(tabs)/home` (`MOB-03`).

## 6. Layout Hierarchy
```text
Screen (bg: #FFFFFF)
├── Header (text: "Complete Your Profile")
└── ScrollView
    ├── img (Avatar Picker)
    ├── form
    │   ├── input (label: "Full Name")
    │   └── p (text: "Email: read-only from SSO")
    └── Button (text: "Get Started", variant: primary)
```

## 7. Component Map
- **Input**: Form field.
- **Button**: Submit action.

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Full Name | `full_name` | Local State | String (Input) |
| Email | `email` | string | Raw Text (from `GET /users/me`) |

## 9. API Map
- **Method**: `GET`
- **Path**: `/users/me`
- **Purpose**: Fetch existing SSO data (like email).
- **Status**: Exists (`02-api-routing-matrix.md` row 86).

- **Method**: `PATCH`
- **Path**: `/users/me`
- **Purpose**: Save the finalized `full_name` (and potentially other future preferences).
- **Status**: Exists (`02-api-routing-matrix.md` row 87).

## 10. UI States
- **Loading**: Form submit spinner.
- **Empty**: N/A.
- **Error**: Form validation errors (e.g., "Name cannot be empty").

## 11. Interaction Specification
- **Trigger**: Tap "Get Started".
- **Action**: Fire `PATCH /users/me` with form data, then navigate to Home.

## 12. Form Specification
- **Fields**:
  - `full_name`: string, required, min 2 chars.

## 13. Responsive / Adaptation
- **Mobile**: Keyboard-avoiding vertical layout.

## 14. Accessibility
- **Role**: `form`.
- **Keyboard**: Focus on input natively.

## 15. Motion
- **Animation**: Slide in from right (standard navigation).

## 16. Security
- Only mutates the authenticated user's own profile.

## 17. Cache / Server State
- **Mutation Key**: `['user-update']`
- **Invalidation**: On success, invalidates `['user', 'me']`.

## 18. Acceptance Criteria
- AC-MOB-01-01: Correctly prepopulates email from `/users/me`.
- AC-MOB-01-02: Successfully fires `PATCH /users/me` to update `full_name`.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: The specific fields required during onboarding are not detailed in product docs. I have inferred `full_name` as a mandatory step based on `DATA_CONTRACT.md` (which marks `full_name` as nullable initially but likely required for UX). If additional fields (like club interests) are added to the backend schema, this form must expand.
