# Mobile Dispute Flows

## 1. Identity
- **Screen ID**: MOB-14
- **Screen Name**: Dispute Flows
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `02-api-routing-matrix.md` (Schema undocumented)

## 2. Route
- **Route**: `/(app)/disputes`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any
- **UX Guard**: Redirect to Auth if unauthenticated.

## 4. Entry Points
- Profile Screen or Attendance Screen.

## 5. Exit / Navigation
- **Back**: Returns to previous screen.

## 6. Layout Hierarchy
```text
Screen (bg: #F9FAFB)
├── Header (text: "Attendance Disputes")
└── ScrollView
    ├── Section (My Disputes List)
    │   └── FlatList
    │       └── Card (Dispute Item)
    │           ├── h3 (Event Name / Date)
    │           ├── Badge (Status: Pending, Resolved)
    │           └── p (Dispute Reason)
    └── Button (text: "Submit New Dispute", floating CTA)
```

## 7. Component Map
- **ScrollView**: Native scroll
- **FlatList**: Native list for existing disputes
- **Card**: Item wrapper
- **Badge**: Status indicator
- **Button**: Floating action button (FAB)

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Data Mapping | Unknown | Unknown | Endpoints confirmed, but response shape undocumented |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/attendance/disputes`
- **Purpose**: Fetch user's attendance disputes.
- **Status**: Exists (per `02-api-routing-matrix.md`).

- **Method**: `POST`
- **Path**: `/v1/attendance/disputes`
- **Purpose**: Submit a new attendance dispute.
- **Status**: Exists (per `02-api-routing-matrix.md`).

- **Method**: `PATCH`
- **Path**: `/v1/attendance/disputes/:id`
- **Purpose**: Resolve an attendance dispute (Admin only).
- **Status**: Exists, but likely not usable in this generic user-facing mobile screen.

## 10. UI States
- **Loading**: Skeletons.
- **Empty**: "No disputes submitted."
- **Error**: Error boundary.

## 11. Interaction Specification
- **Trigger**: Tap "Submit New Dispute".
- **Action**: Opens a modal/sheet. Form submission fires `POST /v1/attendance/disputes`.

## 12. Form Specification
- **New Dispute Modal** (Inferred UX requirement):
  - Field: Event ID (hidden or selected).
  - Field: Reason (text area).
  - Depends on actual POST payload schema.

## 13. Responsive / Adaptation
- **Mobile**: Native screens.

## 14. Accessibility
- **Role**: `main`.

## 15. Motion
- **Animation**: Native push/modal presentation.

## 16. Security
- API relies on authenticated token to submit disputes for the current user.

## 17. Cache / Server State
- **Query Key**: `['attendance-disputes']`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: On successful dispute creation.

## 18. Acceptance Criteria
- AC-MOB-14-01: Successfully connects to `/v1/attendance/disputes` endpoints.

## 19. Specification Gaps / Open Decisions
- **UNDOCUMENTED RESPONSE SHAPE**: Endpoints confirmed to exist (see `02-api-routing-matrix.md` rows 41-43), but exact response field names and the POST payload shape for disputes are not documented anywhere in `DATA_CONTRACT.md`. Recommend a quick codebase check specifically for the dispute DTOs before finalizing Section 8's field mapping.
- **OPEN UX ASSUMPTION**: The UI assumes a list of disputes and a floating action button to submit a new one. Without explicit design specs, this is a best-effort structural guess based on standard mobile list-and-create patterns.
