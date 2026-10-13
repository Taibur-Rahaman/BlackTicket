# Security Architecture and Threat Model

## 1. Security goals

Protect ticket validity, payment integrity, customer privacy, inventory integrity, staff/device authorization, and the audit trail. The backend—not the frontend—must make authoritative decisions.

## 2. Core controls

- Opaque, high-entropy QR credential independent of public Ticket ID.
- No PII or payment secrets in QR payloads.
- Credential hashes in storage where feasible; never log raw credentials.
- Server-side role and resource-scope authorization on every sensitive endpoint.
- Atomic online ticket consumption and concurrency-safe inventory.
- Verified, idempotent payment callbacks.
- Approved-device checks and fresh-snapshot rules for offline validation.
- Idempotent sync ingestion with provenance/integrity verification.
- Append-only audit records for security-sensitive actions.
- HTTPS, secure cookies, CSRF protections where applicable, safe session lifecycle, rate limiting, and input validation.
- Secrets in environment/secret management, never committed.
- Minimize PII, redact logs, and define retention.
- Backups and tested restore procedure.

## 3. Threat register

| Threat | Required mitigation / test |
|---|---|
| QR copying/replay | Opaque credential, one-time central consumption, conflict visibility offline |
| Concurrent duplicate scans | Transaction/lock or atomic compare-and-set; race tests |
| Ticket ID enumeration | IDs are not credentials; rate limits and safe responses |
| Staff abuse/account sharing | Scoped assignments, session/device controls, audit and anomaly monitoring |
| Lost/stolen device | Device approval/revocation and encrypted/protected local data where supported |
| Modified client/frontend | Server-side authority; signed/integrity-checked offline events |
| Offline data theft | Minimize cached data; storage protections; no unnecessary PII |
| Clock manipulation | Separate device time/server receipt time; no timestamp-only conflict rule |
| Stale offline snapshot | TTL enforcement and visible blocked/review state |
| Device revocation while offline | Explicit bounded-risk policy; deny on next central contact |
| Local database tampering | Integrity checks and event authentication; test tampering |
| Malicious/replayed sync | Device authentication, event integrity, idempotency keys |
| Payment callback replay | Signature/status verification and idempotent processing |
| Payment success/order failure | Reconciliation job and consistent transaction/state handling |
| Inventory oversell | Locked transactional reservation and concurrent tests |
| Cross-organizer access | Server-side ownership/scope checks and negative tests |
| Manual override abuse | Explicit permission, reason, audit, monitoring |
| Credential/session theft | Secure session settings, expiration/revocation, rate limits |
| API abuse/DoS | Rate limiting, input limits, monitoring, host-capacity review |
| Audit tampering | Append-only application behavior and restricted DB permissions/backups |
| PII/secret leakage | Structured redaction, no secrets in URLs/QR/logs, privacy tests |

## 4. Offline security limitations

Two disconnected devices may independently accept the same copied credential. v1 central synchronization can detect and reconcile the conflict later but cannot retroactively prevent both local accepts. This is a fundamental limitation of disconnected operation without coordination. Product and operations must explicitly accept this risk and define the reconciliation comparator before launch.

## 5. Key and device lifecycle

The design must specify generation, storage, rotation, revocation, recovery, and compromise response for device keys and any signing keys. Avoid embedding long-lived server secrets in the PWA. If browser-held keys are proposed, assess extraction risk and platform limitations. Do not implement a custom cryptographic protocol without review.

## 6. Audit requirements

Audit events should capture actor, action, resource, result, timestamp, correlation ID, and safe contextual metadata. Important actions include role/permission changes, device approvals/revocations, event publication changes, price/inventory changes, payment transitions, ticket issuance/revocation, validation overrides, and sync conflicts. Never include passwords, raw credentials, session tokens, or provider secrets.

## 7. Security release gate

Before production:
- review threat model and credential protocol;
- test authorization boundaries and IDOR;
- test online race conditions and offline conflict cases;
- test payment callback forgery/replay;
- test local tampering, stale snapshots, device revocation, and sync replay;
- review dependencies and secrets;
- verify backup/restore and incident response;
- document known limitations and obtain product-owner acceptance.