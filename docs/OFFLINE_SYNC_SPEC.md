# Offline Validation and Synchronization Specification

## 1. v1 boundary

v1 supports **local consumption on an approved device, followed by central synchronization**. Peer-to-peer/mesh synchronization is deferred. The system must not claim that isolated devices coordinate with one another.

## 2. Device trust

- Only approved, active devices can enter offline validation mode.
- Each device must have a defined identity and a secure method for authenticating queued events.
- Device enrollment, key provisioning, recovery, and revocation workflow remain open.
- Pending or revoked devices cannot validate.
- A lost device must be revocable centrally; behavior while that device remains disconnected must be described as a limitation.

## 3. Snapshot requirements

A snapshot should be scoped to approved events/gates and include the minimum data needed for offline eligibility checks, plus version, issue time, expiry/freshness metadata, and integrity protection. It must not include unnecessary customer PII or reusable raw secrets.

The following are open: exact TTL, refresh schedule, snapshot signature/encryption format, offline ticket eligibility fields, and handling of event/ticket changes after snapshot issuance.

If a snapshot is stale, malformed, tampered with, or outside its allowed scope, offline validation must be blocked or routed to an explicit review/fallback flow. Do not silently accept.

## 4. Local consumption and queue

For each offline scan:
- validate local authorization and snapshot context;
- check local consumed state;
- write local consumed state and the queued validation event atomically where storage supports it;
- assign a stable `client_event_id`;
- capture device identity, event/gate scope, snapshot version, local sequence/counter where approved, event time, and integrity metadata;
- persist until the server acknowledges durable processing;
- never discard queued events merely because a sync attempt failed.

The local store must be protected against casual access and tampering. PWA storage limitations and browser/device security must be assessed before finalizing the implementation.

## 5. Sync protocol requirements

1. Authenticate the device and verify current device status.
2. Submit a batch with stable batch/event identifiers.
3. Verify request integrity and each event's provenance.
4. Deduplicate by `client_event_id`; duplicate submissions return the previous outcome.
5. Persist all received event evidence.
6. Apply the approved reconciliation rule.
7. Return per-event results: accepted, duplicate/already processed, conflict/review, rejected with reason, or retryable error.
8. Device deletes/marks an event delivered only after durable server acknowledgement.
9. Retried batches must be safe and idempotent.

## 6. Conflict behavior

Both conflicting offline validation events are retained permanently. The authoritative consumption is selected only by the documented authenticity/ordering comparator. The non-authoritative event remains recorded and reviewable. Do not silently delete, overwrite, or hide it.

OD-007 is a blocker for the final reconciliation implementation. Device wall-clock time alone is not an acceptable comparator.

## 7. Revocation and stale information

An offline device cannot instantly learn that a user, ticket, event, or device was revoked after the snapshot was issued. The product must define a maximum stale window and communicate the risk. A revoked device must be denied on its next central contact; previously queued evidence should be retained and evaluated under the approved policy.

## 8. Observability

Track pending queue count, oldest unsynced event age, last successful sync, rejected/conflicted events, device status, snapshot age, and repeated sync errors. Logs must never contain raw QR credentials or secrets.

## 9. Acceptance criteria

- Offline mode is unavailable to unapproved devices.
- Stale/invalid snapshots cannot silently authorize scans.
- Local consumption survives normal app restart and temporary network loss.
- Sync retries do not duplicate events.
- All conflict evidence is retained.
- Server acknowledgement is required before queued events are considered fully synchronized.
- User interface clearly shows offline status and pending sync count.