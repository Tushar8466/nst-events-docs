# Queue Monitoring Screen

## 1. Identity
- **Screen ID**: WEB-13
- **Screen Name**: Queue Monitoring
- **Platform**: Web
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `DATA_CONTRACT.md` (Schema undocumented), `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(app)/admin/queues`

## 3. Role / Access
- **Visibility**: PLATFORM_ADMIN
- **Access**: PLATFORM_ADMIN
- **UX Guard**: Redirect to `/dashboard` if lacking permissions.

## 4. Entry Points
- Admin Hub navigation.

## 5. Exit / Navigation
- **Back**: Admin Hub.

## 6. Layout Hierarchy
```text
SidebarNavigation
├── div
│   ├── BreadcrumbTrail
│   ├── h1 (text: "Queue Monitoring")
│   └── div (Dead Letter Queue List)
│       └── table
│           ├── tr (header: Message ID, Queue Name, Error, Timestamp, Action)
│           └── tr (DLQ row)
```

## 7. Component Map
- **SidebarNavigation**: Context tree
- **Table**: Data display
- **Button**: Actions (Retry, Discard)

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Data Mapping | Unknown | Unknown | Endpoints confirmed, but response shape undocumented |

## 9. API Map
- **Method**: `GET`
- **Path**: `/v1/admin/queue/monitoring`
- **Purpose**: Fetch queue health statistics.
- **Status**: Exists (per `02-api-routing-matrix.md`).

- **Method**: `GET`
- **Path**: `/v1/admin/queue/dead-letters`
- **Purpose**: Fetch Dead Letter Queue (DLQ) messages.
- **Status**: Exists (per `02-api-routing-matrix.md`).

- **Method**: `POST`
- **Path**: `/v1/admin/queue/dead-letters/:id/replay`
- **Purpose**: Retry a specific DLQ message.
- **Status**: Exists (per `02-api-routing-matrix.md`).

## 10. UI States
- **Loading**: Skeletons.
- **Empty**: "Queues are healthy. No dead letters."
- **Error**: Error boundary.

## 11. Interaction Specification
- **Trigger**: Click Retry on a DLQ item.
- **Action**: Fire `POST /v1/admin/queue/dead-letters/:id/replay`.

## 12. Form Specification
- **Not Applicable**.

## 13. Responsive / Adaptation
- **Desktop**: Full table.
- **Mobile**: Hidden/Unsupported (desktop only feature).

## 14. Accessibility
- **Role**: `main`.

## 15. Motion
- **Animation**: Default.

## 16. Security
- Protected admin route.

## 17. Cache / Server State
- **Query Key**: `['admin-queues']`
- **Stale Behavior**: 0 minutes (always fetch fresh for queue monitoring).
- **Invalidation**: On successful replay mutation.

## 18. Acceptance Criteria
- AC-WEB-13-01: Connects to `/v1/admin/queue/monitoring` and `/v1/admin/queue/dead-letters`.
- AC-WEB-13-02: Successfully fires `/v1/admin/queue/dead-letters/:id/replay` on action.

## 19. Specification Gaps / Open Decisions
- **UNDOCUMENTED RESPONSE SHAPE**: Endpoints confirmed to exist (see `02-api-routing-matrix.md` rows 1-3), but exact response field names for the monitoring/dead-letter payloads are not documented anywhere. Recommend a quick codebase check specifically for the response DTO/serializer on these 3 routes before finalizing Section 8's field mapping.
- **OPEN UX ASSUMPTION**: The table columns (Message ID, Queue Name, Error, Action) are inferred based on standard DLQ monitoring UI patterns. No explicit source document defined these UI requirements.
