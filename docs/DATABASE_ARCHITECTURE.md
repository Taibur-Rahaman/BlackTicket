# Database Architecture — BlackTicket

**Status:** Logical schema specification v1 draft  
**Engine:** MySQL/MariaDB with InnoDB, subject to hosting verification.

## 1. Design principles

- Use internal UUIDs or equivalent non-sequential internal identifiers for core entities.
- Public codes are for reference/display, not secrets or authorization.
- Store reusable credential material only as cryptographic hashes where feasible.
- Use transactions and row-level locking for inventory and central ticket consumption.
- Make payment callbacks and sync ingestion idempotent.
- Preserve audit and validation history.
- Use foreign keys, unique constraints, and indexes to enforce integrity.
- Avoid storing payment-card data; let the gateway handle sensitive payment details.

## 2. Logical entities

### Identity and authorization
- `users`: identity, login fields, account status, timestamps.
- `roles` / permission definitions: role and permission catalog.
- `user_role_assignments`: role assignment and scope.
- `organizers`: organizer profile and verification state.
- `staff_assignments`: staff-to-event/gate scope and active status.
- `devices`: device identity, approval/revocation state, public-key or attestation metadata as selected.

### Events and inventory
- `events`: organizer, public event code, title, venue, timezone, schedule, state.
- `gates`: event-linked gate definitions.
- `ticket_types`: event, label, price/currency, sale window, inventory total.
- `inventory_counters` or equivalent authoritative fields: reserved and sold counts.
- `inventory_reservations`: reservation identifier, quantity, expiry, state.

### Commerce and tickets
- `orders`: customer, public order code, totals/currency, status.
- `order_items`: ticket type, quantity, unit price snapshot, line totals.
- `payments`: provider, provider transaction/event IDs, amount/currency, normalized status, verification metadata.
- `tickets`: internal ticket UUID, public Ticket ID/code, event, ticket type, order item, lifecycle state.
- `ticket_credentials`: ticket reference, credential hash, status/version, creation/revocation metadata.

### Validation and synchronization
- `validation_events`: each scan attempt/event, ticket/event, staff, gate, device, mode, result, reason, client event ID, event sequence/provenance, server receipt time, reconciliation status.
- `sync_batches`: device, batch ID, received time, status, processing metadata.
- `device_snapshots` or snapshot metadata: version, event scope, issued time, expiry, integrity/signature metadata.
- `audit_logs`: actor, action, resource type/ID, request/correlation ID, safe metadata, time.

Names are logical proposals, not final migration names. The schema should be normalized and reviewed before migrations are written.

## 3. Integrity constraints

1. Inventory invariant: `reserved + sold <= total`.
2. Available inventory is computed; it is not a client-controlled stored value.
3. Reservation and sale changes are transactional and concurrency-safe.
4. Each ticket has a distinct internal ID and public ticket code.
5. Public ticket IDs/codes are never treated as bearer secrets.
6. Credential hash has an appropriate uniqueness constraint where required.
7. A ticket can have multiple validation event records, but at most one authoritative successful consumption for the central event/ticket scope.
8. Sync events are idempotent by stable `client_event_id` scoped to the device or globally unique.
9. Provider transaction/event identifiers are unique within the provider scope when the provider contract permits.
10. Audit records are append-only in application behavior.
11. Foreign keys prevent orphaned orders, tickets, gates, and validation records.
12. Store amounts in integer minor units or a rigorously specified decimal format; never use binary floating point for money.

## 4. Transactions

### Inventory reservation
- Begin transaction.
- Lock the ticket-type/inventory row.
- Re-read authoritative total, reserved, and sold values.
- Reject if requested quantity cannot be reserved.
- Increase reserved count and create reservation.
- Commit.
- On payment success, convert reservation to sold/issued according to the approved state machine.
- On expiry/failure, release reservation according to approved timeout rules.

### Online ticket consumption
- Begin transaction.
- Lock or atomically update the ticket/consumption row.
- Verify eligible state and that no authoritative consumption exists.
- Create validation event and mark authoritative consumption in the same transaction.
- Commit only if all checks succeed.
- Concurrent attempts must not both be successful.

### Payment callback
- Verify provider callback/signature and/or query provider status.
- Persist callback/event identifier and process idempotently.
- Lock the relevant payment/order/reservation rows.
- Apply only permitted state transitions.
- Issue tickets only after trusted payment verification and according to the approved lifecycle.
- Repeated callbacks must not duplicate issuance.

### Sync ingestion
- Authenticate device and verify its status.
- Validate batch and per-event integrity.
- Insert by idempotency key; repeated event IDs return the prior processing result.
- Preserve raw/normalized evidence needed for audit.
- Reconcile conflicts without deleting either event.

## 5. Indexing guidance

Review indexes for:
- event status and publication window
- organizer-to-event lookup
- ticket type and event inventory
- order customer/status/created time
- provider transaction ID and callback ID
- ticket credential hash
- ticket/event status and authoritative consumption
- validation event by event, gate, device, ticket, and time
- sync batch/device/status
- audit resource/actor/time

Use EXPLAIN and representative load tests before adding speculative indexes.

## 6. Retention and privacy

Define retention periods for personal data, payment metadata, raw sync evidence, and audit records before launch. Keep PII out of QR payloads and avoid logging raw credentials, passwords, tokens, or payment secrets. Deletion/anonymization rules must preserve legally required transaction and audit integrity.

## 7. Still unresolved

Final table names/columns/types, exact public ID format, state machines, credential algorithm, device key lifecycle, event timezone model, refund/transfer data, snapshot TTL, retention periods, and OD-007 reconciliation comparator must be finalized before migrations are implemented.