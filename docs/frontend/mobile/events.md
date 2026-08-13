# Mobile Events Directory

## 1. Identity
- **Screen ID**: MOB-04
- **Screen Name**: Events Directory
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md`, `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(tabs)/events`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any
- **UX Guard**: Redirect to Auth if unauthenticated.

## 4. Entry Points
- Bottom Tab Bar.

## 5. Exit / Navigation
- **Click Event**: Navigates to `MOB-05` (Event Detail).

## 6. Layout Hierarchy
```text
Screen (bg: #F9FAFB)
├── Header (text: "Events")
├── div (Filters Carousel: All | Upcoming | By Club)
└── FlatList
    └── Card (EventCardMobile)
        ├── h3 (title)
        ├── p (start_time to end_time)
        ├── p (location_name)
        └── Badge (Derived State: Open/Closed)
```

## 7. Component Map
- **FlatList**: Native scroll
- **EventCardMobile**: Item wrapper
- **FilterCarousel**: Horizontal native scroll for tags

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Title | `title` | string | Raw Text |
| Location | `location_name` | string | Raw Text |
| Time | `start_time`, `end_time` | string ISO | Formatted date/time |
| Capacity/Availability | `max_capacity`, `registration_count`, `state` | mixed | Client-side derived: if `state == PUBLISHED` and (`max_capacity` is null OR `registration_count < max_capacity`), display "Open", else "Closed". |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/events`
- **Query Params**: `cursor`, `limit`, `filter_state=PUBLISHED`
- **Purpose**: Fetch browsable events.
- **Status**: Exists (`02-api-routing-matrix.md` row 56).

## 10. UI States
- **Loading**: Skeleton items.
- **Empty**: "No events found matching filters."
- **Error**: Error boundary.

## 11. Interaction Specification
- **Trigger**: Tap Event Card.
- **Action**: Navigate to Event Detail.

## 12. Form Specification
- **Filters**: Trigger refetch with new query params (e.g., `filter_club_id`).

## 13. Responsive / Adaptation
- **Mobile**: Vertical scroll.

## 14. Accessibility
- **Role**: `list`.

## 15. Motion
- **Animation**: Native push.

## 16. Security
- Read-only data.

## 17. Cache / Server State
- **Query Key**: `['events', filters]`
- **Stale Behavior**: 5 minutes.
- **Invalidation**: Pull to refresh.

## 18. Acceptance Criteria
- AC-MOB-04-01: Displays `location_name` and uses `cursor`/`limit` for pagination.
- AC-MOB-04-02: Derives the "Open"/"Closed" visual badge purely on the client side from `state`, `max_capacity`, and `registration_count` (does NOT rely on a backend `registration_state` enum).

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: A horizontal filter carousel is included at the top. The specific filtering tabs (All, Upcoming, By Club) represent standard UX, but depend on whether the backend schema supports those explicit query params (e.g., `filter_club_id` is supported, but "Upcoming" requires a date-based filter parameter).
