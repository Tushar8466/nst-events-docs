# Mobile Clubs Directory Screen

## 1. Identity
- **Screen ID**: MOB-12
- **Screen Name**: Clubs Directory
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(tabs)/clubs`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any
- **UX Guard**: Redirect to Auth if unauthenticated.

## 4. Entry Points
- Bottom Tab Bar.

## 5. Exit / Navigation
- **Click Club Card**: Navigates to `MOB-11` (Club Detail).

## 6. Layout Hierarchy
```text
Screen (bg: #F9FAFB)
├── Header (text: "Clubs", Search Input)
└── FlatList (Grid or Stack layout)
    └── Card (Club Card)
        ├── img (banner_url placeholder)
        ├── h2 (name)
        └── div (Footer)
            ├── Badge (status)
            ├── span (event_count)
            └── span (members.length)
```

## 7. Component Map
- **FlatList**: Native list.
- **Card**: Club container.
- **Badge**: Status visualizer.

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Banner | `bannerUrl` | string | Image src (fallback to default gradient) |
| Name | `name` | string | Raw Text |
| Status | `status` | enum | Badge (`ACTIVE`, `INACTIVE`, `DISSOLVED`) |
| Event Count | `event_count` | number | `{count} Events` |
| Member Count | `members` | array | `members.length` Members |

## 9. API Map
- **Method**: `GET`
- **Path**: `/clubs` (or `/clubs/search`)
- **Purpose**: Fetch directory of clubs.
- **Status**: Exists (`02-api-routing-matrix.md` rows 48-49).

## 10. UI States
- **Loading**: Skeleton cards.
- **Empty**: "No clubs found."
- **Error**: Error boundary with retry.

## 11. Interaction Specification
- **Trigger**: Tap Club Card.
- **Action**: Navigate to `MOB-11`.

## 12. Form Specification
- **Search Bar**: Input triggering API refetch against `/clubs/search`.

## 13. Responsive / Adaptation
- **Mobile**: Native vertical stack or 2-column grid.

## 14. Accessibility
- **Role**: `list`.

## 15. Motion
- **Animation**: Standard push.

## 16. Security
- Read-only public/authenticated data.

## 17. Cache / Server State
- **Query Key**: `['clubs']`
- **Stale Behavior**: 10 minutes.
- **Invalidation**: Pull to refresh.

## 18. Acceptance Criteria
- AC-MOB-10-01: Correctly utilizes `event_count` and `members` fields.
- AC-MOB-10-02: Reflects accurate `status` enums.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: "Create Club" button is omitted from the mobile spec intentionally, assuming platform-level administration (club creation) is restricted to the Web platform for usability, even for PLATFORM_ADMIN users.
