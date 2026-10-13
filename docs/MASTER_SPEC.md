# BlackTicket — Master Specification

**Status:** Draft baseline for review  
**Product:** Bangladesh-focused event ticketing platform  
**Repository:** BlackTicket  
**Development mode:** Documentation-first; production code is not authorized yet.

## Purpose

This document defines how project documents are organized and which rules must remain consistent. It is not a substitute for detailed module specifications.

## Source-of-truth order

1. Approved entries in `OPEN_DECISIONS.md`
2. `PLATFORM_PRD.md` for product behavior and scope
3. `SYSTEM_ARCHITECTURE.md` for component boundaries
4. Domain specifications for database, authorization, tickets, validation, offline sync, security, and tests
5. AI prompts and implementation notes

If documents conflict, do not silently choose one. Record the conflict in `OPEN_DECISIONS.md`, identify affected documents, and request a decision.

## Approved baseline reported by project owner

- Laravel + PWA on current PHP hosting is the selected direction, subject to verifying actual hosting capabilities.
- MySQL/MariaDB is the database direction, depending on host support.
- Payment gateway remains provider-neutral behind an adapter until selected.
- QR credential is opaque; no PII in QR; Ticket ID is not a secret.
- Offline validation requires an approved device and a sufficiently fresh snapshot. The freshness TTL is unresolved.
- v1 uses local consumption followed by central sync. Peer/mesh sync is deferred.
- Both conflicting offline validation events are retained. Reconciliation follows documented authenticity/ordering rules; the non-authoritative event remains reviewable and audited.

## Non-negotiable invariants

1. Frontend state is never authoritative for payment, inventory, authorization, or ticket consumption.
2. Server-side authorization must validate the actor and resource scope on every sensitive operation.
3. A ticket credential must not expose personal information or be derived from a public ticket identifier.
4. Ticket consumption must be atomic in the central system.
5. Payment callbacks must be authenticated/verified and idempotent.
6. Inventory must not oversell under concurrent requests.
7. Audit history for security-sensitive actions is append-only.
8. Synchronization must be idempotent and preserve conflicting evidence.
9. Device time alone must not decide an offline conflict.
10. Subdomains are not a security boundary.
11. Secrets and environment-specific URLs must be configuration-driven.
12. Do not introduce blockchain as a default solution; use standard cryptography, signed events where appropriate, and audited reconciliation.

## Document completion gates

Before implementation begins, review and approve:
- PRD and v1 scope
- system and deployment architecture, including hosting compatibility
- database schema and migrations plan
- RBAC and resource scoping
- ticket lifecycle and credential format
- validation authenticity/ordering comparator for offline conflicts
- offline snapshot freshness and device revocation behavior
- payment gateway adapter contract and issuance rules
- event/order/ticket state machines
- testing, threat model, backup, recovery, and release criteria

## Change control

Every material change must state:
- reason and expected benefit
- affected requirements and documents
- security/data migration implications
- backward compatibility
- test and acceptance criteria
- whether an open decision is resolved or created

Do not label a proposal as approved unless the project owner explicitly approves it.