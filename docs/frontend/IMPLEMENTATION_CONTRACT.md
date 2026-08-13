# Frontend Implementation Contract

## A. Purpose
To serve as the definitive entry point for AI code generation.

## B. Scope
Web (Next.js) and Mobile (Expo) frontends.

## C. Authority hierarchy
1. `docs/frontend/IMPLEMENTATION_CONTRACT.md`
2. Screen specifications
3. Shared component specifications
4. `docs/api/02-api-routing-matrix.md`

## D. Web Architecture
See [Web Implementation Contract](./web/IMPLEMENTATION_CONTRACT.md).

## E. Mobile Architecture
See [Mobile Implementation Contract](./mobile/IMPLEMENTATION_CONTRACT.md).

## H. API Integration Rules
Only use endpoints listed in `docs/api/02-api-routing-matrix.md`. Do NOT invent APIs.

## M. Security Rules
Frontend authorization is UX-only. The backend is the security authority.
