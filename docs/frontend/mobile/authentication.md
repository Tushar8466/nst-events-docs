# Mobile Authentication Screen

## 1. Identity
- **Screen ID**: MOB-02
- **Screen Name**: Authentication (Login)
- **Platform**: Mobile (iOS/Android)
- **Implementation Status**: PLANNED
- **Specification Status**: SPECIFICATION COMPLETE
- **Canonical Source Documents**: `02-api-routing-matrix.md`

## 2. Route
- **Route**: `/(auth)/login`

## 3. Role / Access
- **Visibility**: Public
- **Access**: Unauthenticated users.
- **UX Guard**: Redirect to Home tab if already authenticated.

## 4. Entry Points
- App launch when no valid token exists.

## 5. Exit / Navigation
- **Success**: Navigates to `/(tabs)/home` (or Onboarding `MOB-01` if first login).

## 6. Layout Hierarchy
```text
Screen (bg: #111827)
├── div (Hero Image/Illustration)
└── div (Bottom Sheet Container, bg: #FFFFFF)
    ├── img (NST Logo)
    ├── h1 (text: "NST Events")
    ├── p (text: "Discover and manage university events seamlessly.")
    └── Button (text: "Continue with Google", icon: GoogleLogo)
```

## 7. Component Map
- **Button**: OAuth trigger

## 8. Content / Data Map
| UI Element | Source Field | Source Type | Format / Transformation |
|---|---|---|---|
| Header | Static | Text | Branding |

## 9. API Map
- **Method**: `GET`
- **Path**: `/auth/google`
- **Purpose**: Initiate OAuth2 flow via native webview or deep link.
- **Status**: Exists (`02-api-routing-matrix.md` row 44).

- **Method**: `GET`
- **Path**: `/auth/google/callback`
- **Purpose**: Callback handler for deep linking back to the app.
- **Status**: Exists (`02-api-routing-matrix.md` row 45).

- **Method**: `POST`
- **Path**: `/users/me/push-token`
- **Purpose**: Register device for push notifications post-login.
- **Status**: Exists (`02-api-routing-matrix.md` row 89).

## 10. UI States
- **Loading**: Spinner inside button during OAuth redirect.
- **Empty**: N/A.
- **Error**: Toast on OAuth failure (e.g., "Login cancelled" or network error).

## 11. Interaction Specification
- **Trigger**: Tap "Continue with Google".
- **Action**: Opens a secure web session (`SFSafariViewController` or `CustomTabs`) to `/auth/google`. On callback, stores JWT securely and fires `/users/me/push-token` (if permissions granted).

## 12. Form Specification
- **Not Applicable**.

## 13. Responsive / Adaptation
- **Mobile**: Full screen layout, bottom-heavy for thumb reachability.

## 14. Accessibility
- **Role**: `main`.
- **VoiceOver**: Announce "Continue with Google button".

## 15. Motion
- **Animation**: Fade in illustration, slide up bottom container on load.

## 16. Security
- Utilizes PKCE or standard mobile OAuth flow. Tokens stored in secure enclave / keychain.

## 17. Cache / Server State
- **Query Key**: N/A.
- **Invalidation**: Clears any lingering caches and fetches `['user', 'me']` on success.

## 18. Acceptance Criteria
- AC-MOB-02-01: Successfully opens OAuth webview and completes login.
- AC-MOB-02-02: Securely handles the deep-link callback and stores tokens.

## 19. Specification Gaps / Open Decisions
- **OPEN UX ASSUMPTION**: The UI assumes a bottom-heavy modal style layout (common for mobile auth screens), but no specific wireframes dictate this exact layout.
- **OPEN UX ASSUMPTION**: Assumes `POST /users/me/push-token` is fired immediately after successful auth, which is standard, but could be deferred to Onboarding depending on final product decisions.
