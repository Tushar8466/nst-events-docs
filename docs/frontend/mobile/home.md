# Mobile Home Screen

## 1. Identity
- **Screen ID**: MOB-03
- **Screen Name**: Home Feed
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(tabs)/home`

## 3. Role / Access
- **Visibility**: Authenticated User
- **Access**: Any
- **UX Guard**: Redirect to Auth if unauthenticated.

## 4. Entry Points
- App launch / Tab bar.

## 5. Exit / Navigation
- **Click Feed Item**: Deep link to event detail.

## 6. Layout Hierarchy
```text
Screen (bg: #F9FAFB)
├── Header (Greeting)
└── ScrollView
    ├── Section (Live Now)
    │   └── Horizontal Scroll (EventCardMobile)
    ├── Section (Upcoming for You)
    │   └── Horizontal Scroll (EventCardMobile)
    └── Section (My Clubs Updates)
        └── Horizontal Scroll (EventCardMobile)
```

## 7. Component Map
- **ScrollView**: Native scroll
- **EventCardMobile**: Standard mobile event card

## 8. Content / Data Map
| Live Event | `title, state, is_locked, start_time` | string, enum, boolean, ISO | Filtered client-side from `/v1/events` where `state == PUBLISHED` and time is now |
| Upcoming Event | `title, start_time` | string, ISO | Sorted client-side from `/v1/events` |
| Notifications Count | `length` | number | Derived from `/v1/notifications` where `read_at == null` |

## 9. API Map
- **V1 Architecture Note**: There is **NO** dedicated `/v1/home/feed` endpoint. The Mobile Home screen composes its data by parallel-fetching existing V1 endpoints.

- **Method**: `GET`
- **Path**: `/v1/events`
- **Purpose**: Fetch the raw event list to compute "Live Now" and "Upcoming" locally.

- **Method**: `GET`
- **Path**: `/v1/notifications`
- **Purpose**: Fetch user notifications to show an unread badge on the Home screen if applicable.

## 10. UI States
- **Loading**: 
  - **Combined Loading**: Show full-screen skeleton layout while the parallel `Promise.all` request resolves.
- **Empty**: Empty state image with "Check out the Events tab to discover activities!"
- **Error**: 
  - **Partial Failure**: If notifications fail but events load, render the events and show a subtle error toast. If events fail, show a main Error boundary with a retry button.

## 11. Interaction Specification
- **Trigger**: Tap Event Card.
- **Action**: Navigate to `MOB-05` (Event Detail).

## 12. Form Specification
- **Not Applicable**.

## 13. Responsive / Adaptation
- **Mobile**: Native vertical scroll for sections, horizontal scroll for cards.

## 14. Accessibility
- **Role**: `summary`.
- **VoiceOver**: Read section headers logically.

## 15. Motion
- **Animation**: Native push on navigation.

## 16. Security
- Read-only personalized feed based on user token.

## 17. Cache / Server State
- **Query Key**: `['events', 'list'], ['notifications', 'list']` (Parallel queries)
- **Stale Behavior**: Native React Query stale times applied per resource.
- **Invalidation**: On pull-to-refresh, invalidate both queries.

## 18. Acceptance Criteria
- AC-MOB-03-01: Home screen loads without calling a backend aggregation route.
- AC-MOB-03-02: Pull-to-refresh invalidates both event and notification queries simultaneously.

## 19. Specification Gaps / Open Decisions
- **None**. The V1 architecture is formally settled on client-side composition.
