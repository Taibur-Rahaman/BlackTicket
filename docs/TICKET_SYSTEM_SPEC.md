# Ticket System Specification

**Status:** Draft domain specification. Credential algorithm and lifecycle transitions are not locked.

## 1. Three separate concepts

- **Internal Ticket ID:** stable internal primary key, preferably UUID/ULID or equivalent.
- **Public Ticket Code:** human-readable reference for support and display. It is not a secret and must not authorize access.
- **Ticket Credential:** high-entropy opaque bearer value used by the QR flow. It must not be derived from the public Ticket Code and must contain no PII.

Order ID is separate from Ticket ID. One order may contain multiple independently identifiable tickets.

## 2. QR payload requirements

The QR payload must:
- carry only the minimum opaque credential or a safely designed credential reference;
- contain no name, email, phone, payment details, or other PII;
- not contain a public Ticket ID as the sole proof of validity;
- avoid embedding trusted permissions or mutable ticket state;
- be unguessable with sufficient entropy and protected against brute-force lookup;
- be revocable/invalidatable through the chosen credential lifecycle;
- be verified by the trusted backend online, and by a defined cryptographic/local snapshot design offline.

The exact encoding, entropy, signature/MAC strategy, key rotation, credential expiry, and QR URL format are open decisions. Do not implement a guessed format.

## 3. Credential storage

- Generate credentials using a cryptographically secure random source or an approved cryptographic construction.
- Store a cryptographic hash of the credential in the database where hash-only lookup is compatible with the design.
- Never log raw credentials.
- Do not expose credentials in analytics, crash reports, referrer-bearing URLs, or debug output.
- Define safe credential rotation and revocation.
- Protect offline verification material and snapshots on approved devices.

## 4. Issuance flow

1. Customer places an order for a selected event/ticket type.
2. Backend validates event/ticket sale rules and reserves inventory transactionally.
3. Payment adapter creates or tracks the payment attempt.
4. Backend verifies successful payment using trusted provider evidence.
5. Backend applies an approved, idempotent order/payment transition.
6. System creates the required ticket records and credentials exactly once.
7. System records issuance and delivery status.
8. Customer receives ticket access using the delivery method selected by product policy.

Issuance must not be triggered solely by a frontend redirect. Duplicate provider callbacks must not create duplicate tickets.

## 5. Lifecycle (conceptual only)

Potential states include:
- pending/reserved
- issued/active
- consumed/validated
- cancelled
- refunded
- expired
- revoked

These are candidate states, not a final state machine. Exact allowed transitions and their relationship to orders, payments, and validation must be approved. In particular, the behavior of cancellation/refund while a device is offline is unresolved.

## 6. Ticket validity invariants

- A ticket belongs to one event and one ticket type.
- A ticket is linked to its order item/order according to the schema.
- Only eligible issued tickets can validate.
- A central ticket can have at most one authoritative successful consumption.
- All scan attempts and conflict outcomes are retained according to the validation policy.
- Public IDs cannot be used as authentication credentials.
- Revocation and eligibility checks must be applied in online validation; offline behavior is constrained by snapshot freshness.

## 7. Ticket access URL

If ticket pages use URLs:
- use authenticated ownership or an independent high-entropy access token;
- never use Ticket ID alone as authorization;
- avoid leaking credential values through analytics/referrer headers;
- apply rate limits and prevent enumeration;
- define token expiry/revocation if a bearer link is selected.

## 8. Cancellation, refunds, transfer, and resale

No policy is assumed here. Before implementation, decide:
- whether tickets can be transferred;
- whether resale is allowed;
- whether cancellation/refund revokes ticket credentials;
- how partial order refunds affect individual tickets;
- how already-validated tickets are handled;
- how offline devices learn about revocation;
- whether complimentary/zero-price tickets are allowed and under what permissions.

## 9. Acceptance criteria

- Multiple tickets in one order have distinct Ticket IDs and credentials.
- Guessing a Ticket ID does not reveal or validate a ticket.
- Raw credentials are not stored in logs or plaintext database fields.
- Repeated payment callbacks do not duplicate issuance.
- Invalid, cancelled, revoked, wrong-event, or already-consumed tickets cannot pass central validation.
- Every lifecycle transition is validated server-side and audited.
- Credential design is documented and security-reviewed before code is written.