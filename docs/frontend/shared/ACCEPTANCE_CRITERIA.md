# Acceptance Criteria

AC-EVENT-DETAIL-01: Event Detail renders `title`, `start_time`, `location` from `GET /v1/events/:id`. It MUST NOT use `event.name`.
AC-REG-01: Participant status is strictly derived from `GET /v1/events/:id/my-registration`.
AC-REG-02: The frontend NEVER uses `GET /v1/events/:id/registrations` to determine the current user's registration status.
AC-TEAM-01: Team creation sends `team_name` to `POST /v1/events/:id/teams`.
AC-AUTH-01: Web access token is never persisted to localStorage.
AC-CLUB-01: Club screen renders only fields defined in DATA_CONTRACT.md (\`id, name, description, banner_url, status, created_at\`).
AC-ATTENDANCE-01: Frontend never consumes or stores \`qr_secret\`. It is strictly stripped from frontend logic.
AC-LEADERBOARD-01: Leaderboard ordering follows backend response ordering (descending by \`total_points\`) and is not recomputed client-side.
AC-SSE-01: Frontend handles only documented SSE event types (registration_count, waitlist_update, heartbeat).
