# Validation Engine Specification

**Status:** Requirements draft. The offline conflict comparator is a blocking open decision (OD-007).

## 1. Responsibilities

The validation engine determines whether a ticket is eligible for entry, records the scan outcome, and prevents duplicate central consumption. It must not rely on UI state or public ticket identifiers as proof.

## 2. Online flow

1. Authenticate the staff session/device request.
2. Verify staff, device, event, and gate scope.
3. Validate credential format and safely resolve the credential hash.
4. Verify ticket is issued/active and belongs to the requested event.
5. Check authoritative consumption state.
6. Atomically create the authoritative successful consumption and validation event.
7. Write audit metadata and return a structured result.

If two requests race for the same ticket, at most one can consume it successfully.

## 3. Offline flow

1. Confirm device is approved and local authorization material is valid.
2. Confirm snapshot is within the approved freshness TTL and matches the event scope.
3. Verify credential against the local snapshot/cryptographic design.
4. Check local consumed-state storage.
5. Atomically record local consumption and create a durable event with a stable `client_event_id`.
6. Show an explicit offline acceptance state.
7. Queue the event for central sync.

Offline acceptance is not a guarantee of global uniqueness: two disconnected devices may independently accept the same copied credential. The system must communicate and reconcile this risk rather than claiming impossible guarantees.

## 4. Result categories and scanner UX

- **Green / Valid:** centrally accepted or clearly labelled offline-accepted.
- **Red / Invalid:** malformed/unknown credential, wrong event, cancelled/revoked ticket, or already consumed according to authoritative/local state.
- **Yellow / Review:** stale snapshot, ambiguous conflict, recent-scan protection, or a condition requiring staff review.
- **Network/Device error:** explicit error; never silently treat failure as valid.

Use text and icons as well as color. The proposed recent-scan default is 60 seconds, with a suggested configurable range of 30–120 seconds; this is not yet approved. If enabled, show the prior scan time/gate/staff where authorized and provide only a permission-controlled, audited override.

## 5. Validation event data

A validation record should capture:
- stable event/record ID and client event ID
- ticket and event references
- staff, gate, and device references
- outcome and reason
- online/offline mode
- device event sequence/counter and snapshot/version provenance, if selected
- device-reported event time and server receipt time as separate fields
- signature/integrity verification outcome
- sync state and reconciliation outcome
- correlation/audit metadata

Do not treat the device clock as authoritative by itself.

## 6. Conflict reconciliation

Both conflicting offline validation events are retained permanently. During reconciliation, the system determines the authoritative consumption according to documented validation ordering/authenticity rules. The later conflicting event is not silently deleted; it becomes a conflict/rejected/reviewable event with a complete audit trail.

OD-007 must define the comparator before authoritative conflict resolution is implemented. Candidate evidence may include device identity and key status, signed sequence/counter, snapshot version and provenance, event integrity, and server receipt information. No single field is approved as the winner rule yet. Wall-clock timestamp alone is insufficient.

## 7. Recent-scan and override behavior

The recent-scan feature is intended to catch repeated scans caused by camera/UI behavior or staff confusion. Its time window and exact workflow are open. Any manual override must:
- require an explicit permission;
- capture reason and staff identity;
- record prior result and current context;
- be rate-limited/monitored where appropriate;
- create an append-only audit record.

## 8. Performance and reliability

Initial target: 30–50+ scans per device per minute on suitable hardware, subject to load testing. Tests must cover low-end devices, intermittent connectivity, rapid repeat scans, concurrent central requests, clock skew, storage failures, and app reload/restart while events are queued.

## 9. Required acceptance tests

- Online concurrent scans yield no more than one authoritative success.
- Wrong-event and unauthorized-gate attempts are rejected.
- Pending/revoked devices cannot validate.
- A stale snapshot blocks offline validation.
- Repeated sync requests are idempotent.
- Duplicate offline events are both retained and reconciled visibly.
- Manual override is denied without permission and audited when allowed.
- Scanner UI distinguishes offline acceptance from central confirmation.