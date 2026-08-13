# State Contract

## Authentication state
- Web: In-memory token. Refresh via HttpOnly cookie.
- Mobile: Zustand + SecureStore.

## Server state
React Query for all API responses. Must be cleared on logout.
