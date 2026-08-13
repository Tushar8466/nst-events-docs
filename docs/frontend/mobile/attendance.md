# Mobile Attendance Screen

## 1. Identity
- **Screen ID**: MOB-09
- **Screen Name**: Attendance Scanner
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/events/[id]/attendance`

## 3. Role / Access
- **Visibility**: CLUB_ADMIN, CORE_MEMBER, FACULTY_MENTOR, PLATFORM_ADMIN
- **Access**: Only authorized admins for the specific event.
- **UX Guard**: Redirect back if user lacks permission.

## 4. Entry Points
- "Scanner" button on Event Detail page.

## 5. Exit / Navigation
- **Back**: Returns to Event Detail.

## 6. Layout Hierarchy
```text
Screen (bg: #000000 / Camera Viewfinder)
├── Header (Overlay, text: "Scan QR")
├── div (Camera View)
│   └── Frame (Target Area)
└── BottomSheet (Manual Entry & Recent Scans)
    ├── TabSwitcher (Recent | Manual)
    ├── List (Recent Scans)
    │   └── Card (Record: Name, Time, Badge)
    └── Form (Manual Entry)
        ├── Input (Search User ID / Name)
        ├── Select (Status: PRESENT, ABSENT, EXCUSED)
        └── Button (Mark Attendance)
```

## 7. Component Map
- **CameraView**: Native camera module.
- **BottomSheet**: Slide-up panel.
- **Card**: Item wrapper.
- **Badge**: Status visualizer.

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| User Name | `userId` | string | Inferred from ID via client resolution |
| Time | `markedAt` | string ISO | "Just now", "2m ago" |
| Status | `status` | enum | Badge (`PRESENT`, `ABSENT`, `EXCUSED`) |

## 9. API Map
- **Method**: `POST`
- **Path**: `/v1/events/:id/attendance/manual`
- **Purpose**: Submit scanned QR code payload or manual mark.
- **Status**: Exists (`02-api-routing-matrix.md` row 40).

- **Method**: `GET`
- **Path**: `/v1/events/:id/attendance`
- **Purpose**: Fetch recent records to populate the bottom sheet.
- **Status**: Exists (`02-api-routing-matrix.md` row 38).

## 10. UI States
- **Loading**: Camera permission prompt.
- **Empty**: Bottom sheet shows "No scans yet."
- **Error**: Toast on invalid QR (e.g., "Invalid TOTP" or "Geofence violation").

## 11. Interaction Specification
- **Trigger**: Camera reads valid QR.
- **Action**: Vibrate device, fire `POST /v1/events/:id/attendance/manual` with QR payload. Play success sound.
- **Trigger**: Submit manual form.
- **Action**: Fire same `POST` mutation with specific user ID and status.

## 12. Form Specification
- **Manual Entry**:
  - `userId`: Target participant.
  - `status`: Select (`PRESENT`, `ABSENT`, `EXCUSED`).

## 13. Responsive / Adaptation
- **Mobile**: Full screen camera with bottom sheet overlay.

## 14. Accessibility
- **Role**: `dialog` for bottom sheet.
- **VoiceOver**: Announce successful scan loudly.

## 15. Motion
- **Animation**: Bottom sheet spring animation. Green flash on successful scan.

## 16. Security
- API enforces RBAC, device collision rules, TOTP validity, and geofencing.

## 17. Cache / Server State
- **Mutation Key**: `['attendance-mark', eventId]`
- **Invalidation**: On success, invalidates `['attendance', eventId]` to refresh recent list.

## 18. Acceptance Criteria
- AC-MOB-09-01: Correctly fires POST mutation on valid QR scan.
- AC-MOB-09-02: Displays status exactly as `PRESENT, ABSENT, EXCUSED`.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: The UI assumes a split-screen camera and bottom-sheet layout standard for modern scanning apps. Without a concrete wireframe, the exact placement of the manual entry tab is an inferred structural assumption. Note: CSV Export is omitted from this spec entirely, as it is a web/desktop administration feature.
