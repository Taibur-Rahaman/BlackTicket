# System Architecture — BlackTicket

**Status:** Proposed v1 architecture; verify hosting capabilities before lock-in.

## 1. Proposed deployment shape

Use a Laravel modular monolith backed by MySQL/MariaDB and a Progressive Web App (PWA) scanner, provided the actual Hostinger plan supports the required PHP version, Composer workflow, database version, HTTPS, persistent storage, scheduled tasks, and any needed queue/background processing.

The current temporary hosting notes were:
- Temporary domain: `goldenticketbd.coffeezone.shop`
- Temporary path: `/home/u661931912/domains/coffeezone.shop/public_html/goldenticketbd`
- Future domain idea: `goldenticketbd.com` (not confirmed purchased)

These are deployment notes only. Use configuration values such as `APP_URL`; do not hard-code domains in application code.

## 2. Logical modules

- Identity and sessions
- Users, roles, and authorization
- Organizer and event management
- Ticket types and inventory
- Orders and payment adapter
- Ticket issuance and credentials
- Online validation engine
- Device registration and authorization
- Offline snapshot and local consumption
- Sync ingestion and reconciliation
- Reporting and audit
- Notifications/delivery adapter (provider undecided)

Modules may be separated by code boundaries and interfaces without being separate deployed services in v1.

## 3. Main trust boundaries

1. Public browser/customer interface
2. Authenticated organizer/admin interface
3. Staff scanner/PWA and local device storage
4. Laravel backend/API
5. MySQL/MariaDB
6. External payment provider
7. Offline event queue and sync ingestion boundary

The backend is authoritative for payment, inventory, authorization, and central ticket consumption. The scanner UI is not authoritative. A subdomain split alone does not create a security boundary.

## 4. Online validation sequence

1. Authenticate staff/session.
2. Verify device is approved and not revoked.
3. Verify staff, device, event, and gate intersection.
4. Parse opaque credential and look up the corresponding credential hash safely.
5. Check event/ticket eligibility and central consumption state.
6. Atomically consume the ticket using a transaction/locking strategy.
7. Persist validation event and audit metadata.
8. Return a clear result.

Concurrent scans must not both succeed online.

## 5. Offline validation sequence

1. Verify local device approval material and snapshot freshness.
2. Verify ticket/event membership from the snapshot and local credential material.
3. Check local consumed state.
4. Record local consumption and a durable queued event with a unique client event ID and integrity metadata.
5. Display that the result was accepted offline, not centrally confirmed.
6. Sync after reconnection.
7. Server authenticates and validates the event; duplicate submissions are idempotent.
8. Reconcile conflicts using the approved comparator, preserving every event.

The exact signature format, snapshot TTL, and authenticity/ordering comparator remain unresolved.

## 6. Database and transaction rules

- Use InnoDB transactions for critical writes.
- Inventory reservation must serialize conflicting updates; enforce `reserved + sold <= total`.
- Compute available inventory from total/reserved/sold.
- Ticket consumption must use an atomic compare-and-set or transaction/locking design.
- Payment callbacks must have unique provider event/transaction identifiers and idempotent handling.
- Sync ingestion must be idempotent by `client_event_id`.
- Audit records are append-only in application behavior.
- Store credential hashes rather than reusable raw bearer credentials.

## 7. Payment boundary

Define a provider-neutral adapter contract for creating/checking payment intents, verifying callbacks, querying transaction status, and normalizing provider states. The actual gateway, signature rules, refund semantics, and timeout behavior are open. Never issue tickets based solely on a browser redirect.

## 8. Hosting verification checklist

Before architecture approval, confirm:
- Supported PHP/Laravel versions and required extensions
- Composer installation and deployment method
- MySQL/MariaDB version and transaction/locking behavior
- HTTPS and secure cookie support
- Persistent storage writable by the application but not publicly exposed
- Cron/scheduler support
- Queue worker support or a documented synchronous fallback
- Request timeout, upload limits, memory limits, and rate-limit strategy
- Database backup and restore procedure
- PWA service worker and HTTPS support
- Safe environment variable/secret management

If shared hosting cannot safely support a required capability, revise the architecture explicitly rather than pretending the capability exists.

## 9. Future extraction

Payment, notification, and sync processing can be extracted later if scale or operational needs justify it. Do not create separate services solely because different subdomains are possible. Any future extraction requires versioned contracts, authentication between services, observability, and a migration plan.