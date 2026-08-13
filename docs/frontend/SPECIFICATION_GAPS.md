# Specification Gaps

## TAB-CUSTOMIZATION-001
- AREA: Mobile Tab Customization
- MISSING INFORMATION: Which tabs can be customized and how
- STATUS: PRODUCT DECISION REQUIRED (This does not block the Web implementation. For Mobile, this feature can be deferred without affecting other active screens.)

## GET_HOME_FEED-001
- AREA: Mobile Home Screen
- MISSING INFORMATION: The exact endpoint behavior
- STATUS: RESOLVED — V1 USES EXISTING APIS

## WAITLIST-001
- AREA: Waitlist
- STATUS: RESOLVED — EXISTING REGISTRATIONS FILTER SUFFICIENT

## ANALYTICS-001
- AREA: Analytics
- STATUS: DEFERRED — NOT V1

## ANNOUNCEMENTS-001
- AREA: Announcements
- STATUS: DEFERRED — NOT V1

## POINTS-ADJUSTMENT-001
- AREA: Admin Points Adjustment
- STATUS: DEFERRED — PRODUCT/DATA-MODEL FUTURE WORK

## DATA-FIELDS-CLUB-001
- STATUS: RESOLVED. Populated in DATA_CONTRACT.md via Prisma Schema inspection.

## DATA-FIELDS-ATTENDANCE-001
- STATUS: RESOLVED. Populated in DATA_CONTRACT.md via Prisma Schema inspection.

## DATA-FIELDS-LEADERBOARD-001
- STATUS: RESOLVED. Populated in DATA_CONTRACT.md via actual API validation.

## SSE-PAYLOAD-001
- STATUS: RESOLVED. Investigated Postgres migrations and Express routers to fully extract deterministic event types (`registration_count`, `waitlist_update`, `heartbeat`). Fully documented in DATA_CONTRACT.md.
