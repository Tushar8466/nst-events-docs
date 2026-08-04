# Attendance Management Flow

## QR Flow & Presentation Mode
Triggered via Operations Mode. Renders the cryptographic TOTP seed optimized for high-visibility projector display.

## Attendance Monitoring
Real-time dashboard feed updated via SSE (Server-Sent Events) pushed by the Express backend after each successful `mark_attendance` RPC write.

## Manual Verification & Audit Logging
Manual attendance overrides are performed exclusively via a Platform-Admin-gated RPC. When invoked, the action is securely recorded in the `audit_logs` table (as `method = 'MANUAL'`), preventing untraceable grade inflation.

## Flagged Attendances Panel
**Visibility**: shown when one or more `attendance_records` in the current session have `audit_metadata.flagged = true`

**Displays**:
* Warning banner: "X attendance(s) flagged for review"
* List of flagged records showing:
  * Student name
  * Scan timestamp
  * Flag reason
  * Device ID (truncated)
  * Colliding student name (if `device_collision`)

**Actions available per flagged record**:
* Mark as Reviewed (clears flag, adds review note to audit log)
* Revoke Attendance (removes attendance record, adds to audit log)
* Open Dispute (creates a manual dispute record for the student)

**Access**: Club Admin, Faculty Mentor, Faculty Admin, Platform Admin
