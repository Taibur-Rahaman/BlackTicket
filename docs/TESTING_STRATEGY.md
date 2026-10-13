# Testing Strategy — BlackTicket

## 1. Testing principles

Tests must verify security and data invariants, not only successful UI paths. Every critical failure mode should have a reproducible test. Do not begin production implementation until the blocking behavior has been specified.

## 2. Test layers

- **Unit:** state transitions, permission predicates, credential parsing, payment-state mapping, conflict comparator.
- **Integration:** database transactions, inventory reservation, ticket issuance, callback idempotency, sync ingestion, audit writes.
- **API/security:** authentication, RBAC, resource scope, rate limiting, validation errors, IDOR, secret leakage.
- **Concurrency:** simultaneous inventory reservations and duplicate ticket scans.
- **Offline/PWA:** snapshot expiry, local queue persistence, restart, intermittent network, duplicate retries, local tampering.
- **End-to-end:** event setup → purchase/payment verification → ticket issuance → online/offline validation → synchronization/reporting.
- **Load:** representative scan throughput and checkout/inventory contention.
- **Recovery:** backup restoration and reprocessing of pending sync/payment events.

## 3. Required critical scenarios

### Orders and payments
- Valid callback and server-side status verification.
- Forged callback rejected.
- Duplicate callback does not duplicate tickets.
- Out-of-order callbacks do not violate state machine.
- Payment success with application interruption can be reconciled.
- Failed/expired payment releases inventory according to approved rules.
- Partial refund behavior is tested after policy is decided.

### Inventory
- Two customers compete for the last available tickets.
- Concurrent reservations never exceed total inventory.
- Reservation expiry/retry cannot release or reserve the same quantity twice.
- Price changes do not alter an existing order's stored price snapshot unexpectedly.

### Authorization
- Organizer cannot access another organizer's resources.
- Customer cannot access another customer's order/ticket.
- Staff cannot validate an unassigned event/gate.
- Pending/revoked devices are blocked.
- Admin cannot exceed granted permission.
- Unauthorized override is denied and audited.

### Ticket and validation
- Unknown, malformed, revoked, cancelled, wrong-event, and already-used credentials are rejected.
- Public Ticket ID alone cannot validate.
- Concurrent online scans result in at most one authoritative success.
- No raw credential or PII appears in logs/QR payload.
- Recent-scan behavior follows the final approved rule.

### Offline and sync
- Unapproved device cannot validate offline.
- Stale or tampered snapshot is rejected.
- Local consumption survives expected app restart.
- Failed sync can be retried safely.
- Duplicate client event IDs are idempotent.
- Both conflicting offline events remain in records.
- Reconciliation uses the approved authenticity/ordering comparator, not wall-clock time alone.
- Revoked device behavior is tested after reconnect.

## 4. Performance

Initial product target is 30–50+ scans per device per minute on suitable hardware. Establish a reproducible device/network/test dataset and measure median/p95 latency, error rate, queue growth, and recovery after connectivity returns. Do not claim the target is met until a recorded test demonstrates it.

## 5. Release gates

A release candidate requires:
- all critical automated tests passing;
- no unresolved critical/high security findings;
- migration and rollback/recovery plan;
- validated backup restore;
- configuration and secret review;
- documented offline limitations;
- acceptance of open product risks;
- observability and incident procedures;
- product-owner approval.

## 6. Test data and privacy

Use synthetic or anonymized data. Never place production credentials, customer PII, or real payment secrets in fixtures, logs, or screenshots.