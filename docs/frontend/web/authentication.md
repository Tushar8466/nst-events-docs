# Authentication Screen

## 1. Identity
- **Screen ID**: WEB-01
- **Screen Name**: Authentication (Login)
- **Platform**: Web
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(auth)/login`

## 3. Role / Access
- **Visibility**: Public (Unauthenticated Users)
- **Access**: Anyone
- **UX Guard**: Redirect to `/dashboard` if already authenticated.

## 4. Entry Points
- Root route `/` redirects here if unauthenticated.
- Manual navigation.

## 5. Exit / Navigation
- **Success**: Redirects to `/dashboard` (or intended redirect URL).

## 6. Layout Hierarchy
```text
Screen (bg: #111827 / Dark)
├── div (Center Card, bg: #FFFFFF)
│   ├── img (NST Logo)
│   ├── h1 (text: "Welcome to NST Events")
│   ├── p (text: "Sign in with your university account to continue.")
│   └── Button (text: "Continue with Google", icon: GoogleLogo)
```

## 7. Component Map
- **Card**: Wrapper
- **Button**: OAuth trigger

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Header | Static | Text | "Welcome to NST Events" |

## 9. API Map
- **Method**: `GET`
- **Path**: `/auth/google`
- **Purpose**: Initiate OAuth2 flow with Google.
- **Status**: Exists (`02-api-routing-matrix.md` row 44).

- **Method**: `GET`
- **Path**: `/auth/google/callback`
- **Purpose**: Handle OAuth callback (usually handled implicitly by backend redirection).
- **Status**: Exists (`02-api-routing-matrix.md` row 45).

- **Method**: `POST`
- **Path**: `/auth/refresh`
- **Purpose**: Refresh JWT tokens (background).
- **Status**: Exists (`02-api-routing-matrix.md` row 46).

## 10. UI States
- **Loading**: Spinner inside Google button when clicked.
- **Empty**: N/A.
- **Error**: URL param error (e.g., `?error=access_denied`) displays as a red alert box above the button.

## 11. Interaction Specification
- **Trigger**: Click "Continue with Google".
- **Action**: Standard `window.location.href = '/auth/google'` redirect.

## 12. Form Specification
- **Not Applicable** (OAuth flow, no local credentials).

## 13. Responsive / Adaptation
- **Desktop**: Centered floating card.
- **Mobile**: Full screen card.

## 14. Accessibility
- **Role**: `main`.
- **Aria-Labels**: "Sign in with Google button".

## 15. Motion
- **Animation**: Subtle fade-in on load.

## 16. Security
- Implements strict OAuth2 state parameters. Relies on HTTPOnly cookies for token storage.

## 17. Cache / Server State
- **Query Key**: N/A.
- **Invalidation**: On successful login callback, `['user', 'me']` is fetched.

## 18. Acceptance Criteria
- AC-WEB-01-01: Successfully redirects user to Google OAuth flow.
- AC-WEB-01-02: Handles error query parameters gracefully.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: The design assumes a standalone card layout for the login screen. No specific visual assets (like the NST logo) were explicitly documented, so standard placeholder UI elements are assumed.
