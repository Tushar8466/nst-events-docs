# Security Contract

- Frontend authorization is UX-only.
- Backend is the security authority.
- Web: In-memory Web access token, HttpOnly refresh cookie. No sensitive local persistence.
- Mobile: SecureStore.
- Cache isolation: React Query cache must be cleared on logout.
- No user-controlled authorization.
