# Authorization and RBAC — BlackTicket

## 1. Roles

- **Super Admin:** full platform administration, including configuration and privileged account/device operations.
- **Admin:** explicitly granted administrative permissions; do not assume every Admin has Super Admin powers.
- **Organizer:** event-scoped management for events owned/assigned to the organizer.
- **Staff:** validation-only capabilities within assigned event, gate, and approved device scope.
- **Customer:** own profile, orders, payments, and issued tickets, subject to privacy rules.

## 2. Authorization principles

- Enforce authorization server-side on every request and sensitive operation.
- Use deny-by-default permission checks.
- A role name alone is insufficient; validate resource ownership and scope.
- Never trust role, event ID, gate ID, organizer ID, or device status sent by the client without resolving and checking them server-side.
- Prevent insecure direct object references by checking ownership/scope after loading each resource.
- Changes to roles, permissions, event assignments, and device status must be audited.
- Use least privilege and separate administrative duties where practical.

## 3. Scope intersections

### Organizer operations
Organizer access requires:
- active user
- required permission
- organizer membership/ownership
- event belongs to that organizer or is explicitly assigned
- event state allows the requested operation

### Staff validation
A validation attempt must satisfy all of:
- active authenticated staff account
- active staff assignment to the event
- assignment includes the requested gate
- device is approved and not revoked
- device is permitted for that event/operational context
- requested action is allowed for the staff permission set

The effective rule is: **staff ∩ device ∩ event ∩ gate**. Any failed condition denies validation.

### Device states
- `pending`: cannot validate.
- `approved`: may validate only within granted scope and freshness requirements.
- `revoked`: cannot perform new validation or sync operations except a narrowly scoped recovery/revocation-reporting endpoint if explicitly designed.
- Additional states such as lost, suspended, or expired may be added after workflow approval.

## 4. Permission matrix (high-level)

| Capability | Super Admin | Admin | Organizer | Staff | Customer |
|---|---|---|---|---|---|
| Platform configuration | Yes | Scoped/explicit | No | No | No |
| Manage administrative accounts | Yes | Explicitly granted | No | No | No |
| Manage organizers | Yes | Explicitly granted | Own profile only | No | No |
| Create/manage event | Yes | Scoped | Own/assigned event | No | No |
| Configure ticket inventory | Yes | Scoped | Own/assigned event | No | No |
| View platform-wide reports | Yes | Explicitly granted | No | No | No |
| View own event reports | Yes | Scoped | Own/assigned event | No | No |
| Validate tickets | Explicitly for operations | Explicitly for operations | Not by default | Assigned scope only | No |
| Override validation | Explicit permission only | Explicit permission only | Not by default | Not by default; approval required | No |
| View own orders/tickets | Support access audited | Support access audited | No | No | Yes |
| Approve/revoke devices | Yes | Explicitly granted | Proposed workflow unresolved | No | No |

This matrix is a baseline; exact permission keys and administrative scopes require approval.

## 5. Required authorization tests

- Organizer A cannot read/update Organizer B's event, ticket types, inventory, orders, or reports.
- Staff cannot validate for an unassigned event or gate.
- Staff on a pending or revoked device cannot validate.
- Changing event/gate IDs in requests does not bypass scope checks.
- Customer A cannot read Customer B's order or ticket.
- A limited Admin cannot use ungranted Super Admin capabilities.
- Unauthorized override attempts are denied and logged.
- Revoked sessions/tokens/devices cannot continue privileged operations.
- API errors do not leak whether a secret credential exists when that distinction would aid abuse.

## 6. Open workflow questions

Device enrollment/approval, organizer verification, admin permission granularity, staff multi-event assignments, emergency override, and lost-device recovery are unresolved. Record decisions in `OPEN_DECISIONS.md` before implementation.