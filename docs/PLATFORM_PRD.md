# BlackTicket — Platform Product Requirements Document

**Version:** 1.0 draft  
**Status:** For review; not a final implementation contract  
**Product market:** Bangladesh  
**Delivery principle:** Documentation first, then implementation after blocking decisions are approved.

## 1. Product overview

BlackTicket is a digital event ticketing platform that helps attendees discover events and purchase tickets, organizers publish and manage their events, administrators supervise the platform, and gate staff validate entry tickets online or in a controlled offline mode.

The platform must prioritize reliable ticket issuance, duplicate-entry prevention, safe payment handling, scoped permissions, and an auditable record of validation activity.

## 2. Problem statement

Manual and fragmented ticketing workflows can make it difficult to control ticket inventory, verify purchases, detect copied QR codes, coordinate multiple gates, and continue entry operations during network interruptions. BlackTicket aims to provide one centrally managed ticketing workflow with a constrained offline mode and later reconciliation.

## 3. Goals

- Allow customers to discover published events and purchase available tickets.
- Let organizers create and manage only their authorized events.
- Let administrators manage users, events, inventory, configuration, and operational issues according to role.
- Issue a distinct ticket for each attendee/entry unit, even when several tickets belong to one order.
- Validate tickets at authorized event gates and prevent repeated online consumption.
- Support approved-device offline validation with controlled snapshots and central synchronization.
- Preserve all validation evidence and audit sensitive actions.
- Keep payment-provider integration replaceable.
- Build a maintainable v1 on a hosting-compatible stack.

## 4. Non-goals for v1

- Peer-to-peer/mesh synchronization between disconnected devices.
- Blockchain-based ticket verification.
- A guarantee that two isolated offline devices can always prevent the same copied credential from being accepted twice.
- A specific payment gateway before provider selection.
- Unapproved ticket resale, transfer, or refund behavior.
- Guaranteed delivery through SMS, email, WhatsApp, or any specific channel before the delivery decision is made.
- Assigned seating unless separately specified and approved.

## 5. User roles

- **Super Admin:** full platform-level authority.
- **Admin:** operational administration within assigned platform permissions.
- **Organizer:** manages their own authorized events, event settings, and event inventory.
- **Staff:** validates tickets only within assigned event, gate, and approved device scope.
- **Customer/Attendee:** discovers events, places orders, completes payment, and accesses issued tickets.

The role names do not replace resource-level authorization. Every sensitive action must be checked server-side.

## 6. Core user journeys

### 6.1 Event discovery and purchase

1. Customer browses published events.
2. Customer selects an event and ticket type/quantity.
3. System checks availability and reserves inventory using a concurrency-safe operation.
4. Customer completes the provider-hosted or otherwise approved payment flow.
5. Backend verifies payment status with trusted server-side evidence.
6. Tickets are issued only after the configured payment success condition is met.
7. Customer receives access through a delivery method that remains to be decided.

A browser redirect alone must never be treated as proof of payment.

### 6.2 Organizer event management

1. Organizer signs in.
2. Backend verifies organizer permissions and event scope.
3. Organizer drafts an event and defines ticket types, inventory, prices, and sale windows within allowed rules.
4. Event follows a defined approval/publication lifecycle.
5. Organizer monitors sales and event operations for their own events only.

Publication approval workflow remains an open decision.

### 6.3 Gate validation

1. Staff signs in on an approved device.
2. Device and staff are checked for active status and assigned event/gate scope.
3. Scanner reads the QR credential.
4. Validation engine verifies credential, event, ticket state, and consumption status.
5. Online mode atomically consumes a valid ticket and records the outcome.
6. Offline mode uses a valid local snapshot and local consumed-state store, then queues a signed/auditable event for sync.
7. UI clearly distinguishes valid, invalid, already-used, wrong-event, recent-scan, and review-required results.

### 6.4 Offline recovery and synchronization

1. An approved device enters offline mode only when its cached snapshot is within the configured freshness policy.
2. It validates against the locally available event/ticket data and records local consumption.
3. It queues events with stable client-generated event IDs and integrity metadata.
4. On reconnection, it uploads events idempotently.
5. Server verifies device/event authenticity, permissions, snapshot context, and event integrity.
6. Conflicts are preserved and reconciled using an explicitly documented rule. Neither event is silently deleted.

## 7. Functional requirements

### FR-01 Identity and access
- Support authentication for customer, organizer, staff, and administrative accounts.
- Enforce role and resource scopes on the backend.
- Support revocation/suspension of users and devices.
- Audit privileged changes and sensitive actions.

### FR-02 Event management
- Create, edit, publish, unpublish, and archive events according to an approved state machine.
- Store event venue, schedule, timezone, organizer, status, and relevant public details.
- Enforce organizer ownership/scope on all event operations.
- Define event ID structure separately; IDs must never grant access.

### FR-03 Ticket inventory
- Support ticket types with total inventory, sale windows, and price.
- Prevent overselling during concurrent reservations/purchases.
- Enforce invariant: reserved quantity + sold quantity must not exceed total inventory.
- Calculate available inventory from authoritative values rather than trusting a client-supplied number.
- Do not permit an organizer to set a sale price to zero unless an approved complimentary/discount feature explicitly allows it.

### FR-04 Orders and payments
- One order may contain multiple tickets.
- Order ID and Ticket ID must be distinct.
- Payment integration must use a provider adapter.
- Verify callbacks/signatures and payment status server-side.
- Handle duplicate callbacks idempotently.
- Do not issue tickets from an unverified client success page.
- Refund, partial payment, timeout, and cancellation semantics remain open.

### FR-05 Ticket issuance
- Issue a unique ticket record for each entry unit after payment success criteria are satisfied.
- Generate an opaque credential independent of the public Ticket ID.
- Store only a suitable cryptographic hash of the bearer credential in the database where the design permits hash-only storage.
- QR payload must not contain PII, payment secrets, or authorization claims trusted without server/device verification.
- Ticket access URLs must not expose predictable credentials.

### FR-06 Validation
- Validate tickets against event and ticket state.
- Enforce staff ∩ device ∩ event ∩ gate scope.
- Central online consumption must be atomic.
- Record valid and rejected attempts with reason, actor, device, gate, mode, and timestamp.
- Support multi-gate operations.
- Apply configurable recent-scan protection; proposed default is 60 seconds, but this is not approved yet.
- Staff override, if enabled, must require permission and create an audit event.

### FR-07 Offline mode
- Offline validation is available only to approved devices with a sufficiently fresh snapshot.
- v1 supports local consume followed by central synchronization.
- v1 does not promise peer/mesh coordination.
- Clearly show offline state and any reduced assurance.
- Retain both sides of conflicts and preserve audit evidence.

### FR-08 Reporting and audit
- Provide role-scoped event, sales, order, ticket, and validation reports.
- Record administrative changes, payment state transitions, issuance, validation, overrides, device approvals/revocations, and synchronization conflicts.
- Audit history must not be silently edited or deleted by ordinary application flows.

### FR-09 Configuration
- Domain names, API URLs, payment settings, TTLs, and secrets must be environment/configuration values.
- No production secret may be committed to source control.
- Environment URLs must not be hard-coded throughout the codebase.

## 8. Non-functional requirements

- **Security:** server-side authorization, credential secrecy, safe session handling, rate limiting, input validation, and audited privileged operations.
- **Integrity:** atomic online consumption, concurrency-safe inventory, idempotent payments and sync.
- **Availability:** useful operator feedback during connectivity loss; safe degraded mode rather than pretending offline state is globally current.
- **Performance:** target 30–50+ scans per device per minute on suitable hardware; verify by load testing and define test hardware.
- **Accessibility/usability:** readable scan results, clear color plus text/icon indicators, and touch-friendly scanner UI.
- **Maintainability:** modular monolith boundaries and explicit service contracts.
- **Privacy:** collect only necessary personal data; keep PII out of QR payloads and public URLs.
- **Observability:** structured operational logs, metrics, and actionable error messages without leaking secrets.

## 9. Acceptance criteria for v1

- Customers cannot receive tickets based only on a client-side payment success screen.
- Inventory cannot exceed configured total under concurrent requests.
- An order with multiple tickets creates separately identifiable tickets.
- A valid online ticket can be consumed only once by the authoritative central system.
- Staff cannot validate outside assigned event/gate/device scope.
- Pending or revoked devices cannot validate.
- Offline validation is blocked for a stale or invalid snapshot.
- Duplicate sync submission does not create duplicate validation events.
- Conflicting offline events remain available for review and audit.
- Public ticket identifiers cannot be used as authentication credentials.
- Security-sensitive actions have an auditable record.
- Automated tests cover authorization, concurrency, payment callback idempotency, credential handling, and offline reconciliation.

## 10. Decisions still required

See [OPEN_DECISIONS.md](OPEN_DECISIONS.md). In particular: payment gateway, ticket delivery, exact lifecycle/state machines, offline freshness TTL, device approval, QR credential algorithm, event ID format, recent-scan behavior, and the offline conflict authenticity/ordering comparator.

## 11. Release gate

This PRD is a draft baseline, not permission to begin production coding. Implementation begins only after the architecture and blocking domain/security decisions are reviewed and approved.