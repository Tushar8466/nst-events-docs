# AI Build Rules

- NEVER invent an endpoint.
- NEVER invent a response field.
- NEVER trust frontend role checks as authorization.
- NEVER persist access tokens in localStorage unless explicitly specified.
- NEVER invent enum values.
- NEVER bypass the API contract.
- NEVER use historical documentation as current implementation authority.
- NEVER create a backend workaround for a missing specification.

WHEN INFORMATION IS MISSING: STOP and report a specification gap.
WHEN TWO CURRENT DOCUMENTS CONFLICT: STOP and report the contradiction.
