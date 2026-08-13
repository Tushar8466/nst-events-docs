# Notification Center

The Notification Center is strictly for operational system alerts and announcements.

## Canonical Notification Types

Below is the exhaustive, single-source-of-truth definition for every notification emitted by the system.

### 1. WAITLIST_PROMOTED
* **Notification Type**: `WAITLIST_PROMOTED`
* **Priority**: HIGH
* **Producer**: Node.js Notification Producer Service (invoked by `DELETE /v1/events/:id/register` or `DELETE /v1/teams/:id/leave` API handlers after RPC returns promoted IDs)
* **Consumer**: Promoted Student
* **Title Template**: "You're off the waitlist!"
* **Body Template**: "You have been promoted to a registered spot for {event_title}."
* **Routing Target**: `/events/{event_id}`
* **Deep Link**: `{ target: "/events/{event_id}", params: { event_id } }`
* **Metadata Schema**: v1
* **Entity IDs**: `event_id`
* **Action Payload**: None
* **Push Eligible**: Yes
* **Preference Gate**: Bypasses category preferences. Respects `push_enabled`.
* **Badge Behavior**: Yes
* **Action Buttons**: `[View Event]`
* **Expiry Rule**: Event End
* **Audit Event**: `EVENT_REGISTRATION_PROMOTED`
* **Idempotency Scope**: `SHA256("WAITLIST_PROMOTED" + user_id + event_id + "promoted")`

### 2. APPROVAL_REQUEST
* **Notification Type**: `APPROVAL_REQUEST`
* **Priority**: HIGH
* **Producer**: `create_event` / `update_event` RPC (transitions to PENDING_APPROVAL)
* **Consumer**: Faculty Mentor / Platform Admin
* **Title Template**: "Action Required: Event Approval"
* **Body Template**: "{club_name} has requested approval for {event_title}."
* **Routing Target**: `/approvals/{event_id}`
* **Deep Link**: `{ target: "/approvals/{event_id}", fallback: "/approvals", params: { event_id } }`
* **Metadata Schema**: v1
* **Entity IDs**: `event_id`, `club_id`
* **Action Payload**: `{ status: "PENDING_APPROVAL" }`
* **Push Eligible**: Yes
* **Preference Gate**: Bypasses category preferences. Respects `push_enabled`.
* **Badge Behavior**: Yes
* **Action Buttons**: `[Review]`
* **Expiry Rule**: Event Start
* **Audit Event**: `EVENT_STATE_TRANSITION`
* **Idempotency Scope**: `SHA256("APPROVAL_REQUEST" + user_id + event_id + "pending_approval")`

### 3. EVENT_APPROVED
* **Notification Type**: `EVENT_APPROVED`
* **Priority**: NORMAL
* **Producer**: `approve_event` RPC
* **Consumer**: All Students with club_announcements enabled for the club (followers/members)
* **Title Template**: "New Event: {event_title}"
* **Body Template**: "{club_name} just published a new event!"
* **Routing Target**: `/events/{event_id}`
* **Deep Link**: `{ target: "/events/{event_id}", fallback: "/events", params: { event_id } }`
* **Metadata Schema**: v1
* **Entity IDs**: `event_id`, `club_id`
* **Action Payload**: `{ status: "PUBLISHED" }`
* **Push Eligible**: Yes
* **Preference Gate**: `club_announcements`
* **Badge Behavior**: Yes
* **Action Buttons**: `[View Event]`
* **Expiry Rule**: Event Start
* **Audit Event**: `EVENT_STATE_TRANSITION`
* **Idempotency Scope**: `SHA256("EVENT_APPROVED" + user_id + event_id + "published")`

### 4. EVENT_REJECTED
* **Notification Type**: `EVENT_REJECTED`
* **Priority**: HIGH
* **Producer**: `reject_event` RPC
* **Consumer**: Event Creator / Club Admins
* **Title Template**: "Event Rejected: {event_title}"
* **Body Template**: "Your event was rejected. Feedback: {review_notes}"
* **Routing Target**: `/events/{event_id}/edit`
* **Deep Link**: `{ target: "/events/{event_id}/edit", fallback: "/dashboard", params: { event_id } }`
* **Metadata Schema**: v1
* **Entity IDs**: `event_id`
* **Action Payload**: `{ status: "REJECTED" }`
* **Push Eligible**: Yes
* **Preference Gate**: Bypasses category preferences. Respects `push_enabled`.
* **Badge Behavior**: Yes
* **Action Buttons**: `[Edit Event]`
* **Expiry Rule**: 7 days
* **Audit Event**: `EVENT_STATE_TRANSITION`
* **Idempotency Scope**: `SHA256("EVENT_REJECTED" + user_id + event_id + "rejected")`

### 5. ATTENDANCE_DISPUTE_RESOLVED
* **Notification Type**: `ATTENDANCE_DISPUTE_RESOLVED`
* **Priority**: NORMAL
* **Producer**: `resolve_attendance_dispute` RPC
* **Consumer**: Student who filed the dispute
* **Title Template**: "Attendance Dispute {status}"
* **Body Template**: "Your dispute for {event_title} has been {status}."
* **Routing Target**: `/attendance/disputes/{dispute_id}`
* **Deep Link**: `{ target: "/attendance/disputes/{dispute_id}", fallback: "/attendance/disputes", params: { dispute_id } }`
* **Metadata Schema**: v1
* **Entity IDs**: `event_id`, `dispute_id`
* **Action Payload**: `{ status: "{status}" }`
* **Push Eligible**: Yes
* **Preference Gate**: `attendance_alerts`
* **Badge Behavior**: Yes
* **Action Buttons**: `[View Details]`
* **Expiry Rule**: 30 days
* **Audit Event**: `DISPUTE_RESOLVED`
* **Idempotency Scope**: `SHA256("ATTENDANCE_DISPUTE_RESOLVED" + user_id + dispute_id + status)`

### 6. ROLE_CHANGED
* **Notification Type**: `ROLE_CHANGED`
* **Priority**: NORMAL
* **Producer**: `promote_member` RPC (or assign club admin)
* **Consumer**: Target User whose role changed
* **Title Template**: "Your role has been updated"
* **Body Template**: "You are now a {role} in {club_name}."
* **Routing Target**: `/clubs/{club_id}`
* **Deep Link**: `{ target: "/clubs/{club_id}", fallback: "/clubs", params: { club_id } }`
* **Metadata Schema**: v1
* **Entity IDs**: `club_id`
* **Action Payload**: `{ role: "{role}" }`
* **Push Eligible**: Yes
* **Preference Gate**: Bypasses category preferences. Respects `push_enabled`.
* **Badge Behavior**: Yes
* **Action Buttons**: `[View Club]`
* **Expiry Rule**: 30 days
* **Audit Event**: `CLUB_ROLE_UPDATED`
* **Idempotency Scope**: `SHA256("ROLE_CHANGED" + user_id + club_id + role)`

### 7. CLUB_ANNOUNCEMENT
* **Notification Type**: `CLUB_ANNOUNCEMENT`
* **Priority**: NORMAL
* **Producer**: `create_announcement` RPC (Announcement Center publish)
* **Consumer**: Targeted Audience (All Students, Club Members, or Event Attendees)
* **Title Template**: "New Announcement from {club_name}"
* **Body Template**: "{announcement_title}"
* **Routing Target**: `/announcements/{announcement_id}`
* **Deep Link**: `{ target: "/announcements/{announcement_id}", fallback: "/announcements", params: { announcement_id } }`
* **Metadata Schema**: v1
* **Entity IDs**: `club_id`, `announcement_id`
* **Action Payload**: None
* **Push Eligible**: Yes
* **Preference Gate**: `club_announcements`
* **Badge Behavior**: Yes
* **Action Buttons**: `[Read]`
* **Expiry Rule**: 7 days
* **Audit Event**: `ANNOUNCEMENT_CREATED`
* **Idempotency Scope**: `SHA256("CLUB_ANNOUNCEMENT" + user_id + announcement_id)`

### 8. SYSTEM_ALERT
* **Notification Type**: `SYSTEM_ALERT`
* **Priority**: HIGH
* **Producer**: `create_system_alert` RPC (Platform Admin only)
* **Consumer**: All Active Users
* **Title Template**: "System Alert"
* **Body Template**: "{alert_body}"
* **Routing Target**: `/dashboard`
* **Deep Link**: `{ target: "/dashboard", fallback: "/", params: {} }`
* **Metadata Schema**: v1
* **Entity IDs**: None
* **Action Payload**: None
* **Push Eligible**: Yes
* **Preference Gate**: Bypasses category preferences. Respects `push_enabled`.
* **Badge Behavior**: Yes
* **Action Buttons**: None
* **Expiry Rule**: 7 days
* **Audit Event**: None
* **Idempotency Scope**: `SHA256("SYSTEM_ALERT" + user_id + alert_id)`

### 9. EVENT_REMINDER
* **Notification Type**: `EVENT_REMINDER`
* **Priority**: HIGH
* **Producer**: Event Reminder Scheduler (Node.js cron / background worker)
* **Consumer**: Registered Attendees (Status: REGISTERED)
* **Title Template**: "Reminder: {event_title} starts soon!"
* **Body Template**: "Your registered event {event_title} starts at {start_time}."
* **Routing Target**: `/events/{event_id}`
* **Deep Link**: `{ target: "/events/{event_id}", fallback: "/events", params: { event_id } }`
* **Metadata Schema**: v1
* **Entity IDs**: `event_id`
* **Action Payload**: None
* **Push Eligible**: Yes
* **Preference Gate**: `event_reminders`
* **Badge Behavior**: Yes
* **Action Buttons**: `[View Event]`
* **Expiry Rule**: Event Start
* **Audit Event**: None
* **Idempotency Scope**: `SHA256("EVENT_REMINDER" + user_id + event_id + "24h")`

### 10. ATTENDANCE_ALERT
* **Notification Type**: `ATTENDANCE_ALERT`
* **Priority**: HIGH
* **Producer**: Attendance Session Scheduler (Node.js cron / background worker)
* **Consumer**: Registered Attendees (Status: REGISTERED)
* **Title Template**: "Check-in Open: {event_title}"
* **Body Template**: "Check-in for {session_title} is now open. Open your scanner!"
* **Routing Target**: `/attendance/scan/{session_id}`
* **Deep Link**: `{ target: "/attendance/scan/{session_id}", fallback: "/events", params: { session_id } }`
* **Metadata Schema**: v1
* **Entity IDs**: `event_id`, `attendance_id` (session_id)
* **Action Payload**: None
* **Push Eligible**: Yes
* **Preference Gate**: `attendance_alerts`
* **Badge Behavior**: Yes
* **Action Buttons**: `[Check In]`
* **Expiry Rule**: Session Close Time
* **Audit Event**: None
* **Idempotency Scope**: `SHA256("ATTENDANCE_ALERT" + user_id + session_id + "open")`

## Notification Lifecycle
Generated by backend producers (RPCs or Schedulers), written to the `notifications` table, enqueued in `notification_jobs`, delivered via Expo Push Notification or SSE for real-time in-app delivery, marked as read by the user, and eventually archived.

## Deep Links & Action Routing
Clicking a notification immediately navigates the dashboard shell based on the explicit `Routing Target`. Refer to the **Deep Link Contract** in `13-notification-data-model.md` for the canonical metadata payload schema.

## Priority Rules
Urgent actions (Approvals) float to the top and trigger unread badges. Priority `HIGH` guarantees synchronous queue dispatch when possible.
