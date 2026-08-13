# Mobile Implementation Contract

## Expo Router hierarchy
Uses `(auth)` and `(app)` boundaries. See [Mobile Navigation](../../mobile/01-mobile-navigation.md).

## Session lifecycle
Managed via SecureStore and React Query. See [Registration Flow](../../mobile/12-registration-flow.md).

## Cache clearing on logout
Logout MUST clear React Query private data.

## AMBIGUOUS — PRODUCT DECISION REQUIRED
Mobile tab customization is undefined.
