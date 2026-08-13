# Pessimistic Registration Flow & UI Specifications

**Implementation Status:** PLANNED / IN PROGRESS


## Philosophy
NST-Events operates on a strict **Pessimistic Registration Flow**. Because event capacity is highly competitive, the UI must never show "Registered" or optimistic success states before backend confirmation.

## Registration UI States
### Buttons & Actions
* **Register Button**: Visible if `status == OPEN` and user is not registered. Blue primary styling.
* **Cancel Registration Button**: Visible if user is `REGISTERED` or `WAITLISTED`. Danger styling (Red).
* **Disabled State**: If `status == CLOSED` or `capacity == full` (and waitlist full). Button greyed out.
* **Loading State**: When a network request is active, buttons display an inline spinner, text changes to "Processing...", and the button is `disabled`.
* **Offline State**: If `navigator.onLine` is false, Register buttons are disabled.

### Skeleton Loaders & Empty States
* **Skeleton Loaders**: While fetching data (e.g. `GET /v1/users/me/registrations` or `GET /v1/events/:id/my-registration` for participant state, as opposed to the administrative `GET /v1/events/:id/registrations` list), display 3 pulsing grey skeleton rows representing list items.
* **Empty Registrations**: If a user has no active registrations, show an empty state graphic (e.g., calendar icon) and the text: "No upcoming events. Pull to refresh."
* **Empty Teams**: If an event has no teams formed, show an empty state graphic with the text: "No teams formed yet. Be the first to create one!"
* **Empty Waitlist**: "No users currently on the waitlist."
* **Background Sync**: Do not show full-screen loading spinners on background refresh. Show a small subtle spinner in the header.

## Team Management UI
### Create Team Flow
* **Screen**: Team Creation Modal.
* **Navigation**: Tap "Create Team" -> Opens Modal.
* **Loading State**: Spinner on the "Create" button while `POST /v1/events/:id/teams` is executing. Modal cannot be closed while loading.
* **Success State**: Modal dismisses, UI reflects user as Team Leader. Toast: "Team Created".
* **Failure State**: Modal stays open, displays inline red text error.

### Join Team Flow
* **Screen**: Team Listing Page -> Join Confirmation Dialog.
* **Navigation**: Tap "Join" next to a team -> Opens Dialog "Join [Team Name]?".
* **Loading State**: "Confirm" button shows spinner, calls `POST /v1/teams/:id/join`.
* **Success State**: Dialog dismisses, user added to team roster.
* **Failure State**: Dialog dismisses, Toast shows failure reason (e.g. "Capacity reached").

### Leave Team Flow
* **Screen**: Team Detail Page.
* **Navigation**: Tap "Leave Team" -> Opens Danger Dialog "Are you sure?".
* **Loading State**: Spinner on "Leave" button, calls `DELETE /v1/teams/:id/leave`.
* **Success State**: Dialog dismisses, user removed from roster.
* **Failure State**: Toast "Failed to leave team".

### Leader Transfer UX
* **Visual Indicator**: The Team Leader has a yellow "Crown" icon next to their name in the roster.
* **Automatic Transfer**: When the leader leaves, the crown icon moves automatically to the next oldest `REGISTERED` member without any user intervention, driven by the API response payload.

### Team Dissolution UX
* **Trigger**: The last registered member leaves.
* **UX**: The team disappears from the Team Listing Page immediately upon successful API response.

## Toasts Matrix
All toasts appear at the bottom of the screen, have a duration of 4000ms, and can be swiped to dismiss.
| Event | Type | Title | Body |
|---|---|---|---|
| Success Reg | Success | Registered! | You are confirmed for [Event Name]. |
| Waitlisted | Info | Waitlisted | The event is full. You are on the waitlist. |
| Promotion | Success | Promoted! | You have been moved off the waitlist for [Event Name]! |
| Join Team | Success | Team Joined | You joined [Team Name]. |
| Leave Team | Info | Team Left | You left [Team Name]. |
| API Failure | Error | Action Failed | Capacity reached or action invalid. |
| Offline | Error | No Connection | You are currently offline. |
| Session Exp | Error | Session Expired | Please log in again. |
| Server Error| Error | Server Error | An unexpected error occurred. |
| Reconnect | Info | Reconnected | Realtime sync restored. |

## Outcomes & State Synchronization
* **REGISTERED**: Success UI.
* **WAITLISTED**: Routes to the Waitlist Flow screen automatically if registration returns waitlist status.
* **FAILED**: Error toast (e.g., network failure, banned user). Does not change local state.
