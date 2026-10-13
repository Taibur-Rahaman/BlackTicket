# Open Decisions Register

Use this file as the decision log. A status of **Approved** means the project owner explicitly approved the direction; **Proposed** is not approval.

## Decisions reported as approved

| ID | Topic | Current decision | Notes |
|---|---|---|---|
| OD-001 | Application stack | Laravel modular monolith + PWA direction | Verify actual hosting capabilities before final architecture lock. |
| OD-002 | Database | MySQL/MariaDB depending on hosting support | Confirm version, transaction behavior, and operational access. |
| OD-008 | QR identity | Opaque credential; no PII; Ticket ID is not secret | Exact token construction and crypto details remain open. |
| OD-009 | Offline trust | Approved device + fresh snapshot | Snapshot freshness TTL remains open. |
| OD-010 | v1 offline sync | Local consume + central sync; peer/mesh deferred | No promise of coordination between disconnected devices. |
| OD-011 | Conflict retention | Retain both conflicting offline events | Reconciliation rule must be explicitly documented; never silently delete the non-authoritative event. |

**Payment:** use an adapter boundary and choose the gateway later. This is the current planning direction; do not assume a specific provider is approved.

## Blocking decisions

| ID | Decision needed | Why it matters | Suggested owner/status |
|---|---|---|---|
| OD-003 | Payment gateway and server-side verification contract | Callback signature, status query, refund support, reconciliation | Product owner — Open |
| OD-004 | Customer ticket delivery method | Email, SMS, WhatsApp, download, account page, or combination | Product owner — Open |
| OD-005 | Event, order, ticket, and payment state machines | Prevents ambiguous transitions and invalid issuance/refund behavior | Product owner + engineering — Open |
| OD-006 | Payment adapter details and failure/refund behavior | Adapter boundary is planned; exact provider and contract remain open | Product owner — Open |
| OD-007 | Offline conflict authenticity/ordering comparator | Required before authoritative reconciliation behavior can be specified | Security/engineering — Blocking |
| OD-012 | Offline snapshot freshness TTL | Determines how long disconnected validation is allowed | Product owner + security — Blocking |
| OD-013 | Device enrollment, approval, key provisioning, revocation | Determines trust and recovery of scanner devices | Product owner + security — Blocking |
| OD-014 | QR credential format, entropy, signing, rotation, lookup | Security-critical implementation detail | Security/engineering — Blocking |
| OD-015 | Event ID/public code format | Display, URL, enumeration resistance, integrations | Product owner — Open |
| OD-016 | Recent-scan protection window and override UX | Scanner behavior and staff workflow | Product owner — Open |
| OD-017 | Organizer event publication/approval process | Trust, moderation, and platform operations | Product owner — Open |
| OD-018 | Ticket transfer, resale, cancellation, refund policy | Affects ticket lifecycle and validation | Product owner — Open |
| OD-019 | Complimentary tickets, discounts, and zero-price policy | Pricing and abuse prevention | Product owner — Open |
| OD-020 | Staff assignment across events/gates | Determines RBAC scope and shift workflow | Product owner — Open |
| OD-021 | Maximum scale assumptions | Hosting, indexing, caching, queue, and load-test targets | Product owner — Open |
| OD-022 | Assigned seating and ticket-type behavior | Inventory model and customer checkout | Product owner — Open |
| OD-023 | Event timezone and schedule semantics | Event start/end and scan windows | Product owner — Open |
| OD-024 | Backup, restore, RPO/RTO targets | Operational recovery | Product owner — Open |
| OD-025 | PWA-only vs native scanner app | Device storage, installation, background sync, hardware support | Product owner — Open |

## Conflict policy

Both offline validation events are retained permanently. During reconciliation, the system determines the authoritative consumption according to the documented validation ordering/authenticity rules. The later conflicting event is not silently deleted; it becomes a conflict/rejected/reviewable event with a complete audit trail.

Device wall-clock timestamp alone must not decide the winner because clocks can drift or be manipulated. The comparator may consider authenticated device identity, signed event sequence/counters, snapshot/version provenance, server receipt order, and other verified evidence, but the final rule must be reviewed and approved before implementation.

## Decision template

For each resolved item, record:
- Decision and rationale
- Status and approval date
- Alternatives considered
- Security/privacy/data implications
- Documents requiring updates
- Tests/acceptance criteria