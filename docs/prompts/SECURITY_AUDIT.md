# Security Audit Prompt

Review the BlackTicket repository for security weaknesses. Read the PRD, master specification, architecture, RBAC, ticket, validation, offline/sync, database, and open-decision documents before assessing implementation.

## Review areas

- Authentication, sessions, authorization, and resource-level access control
- Organizer/customer isolation and IDOR
- QR credential entropy, storage, exposure, replay, and revocation
- Atomic ticket consumption and concurrent scans
- Inventory reservation/oversell
- Payment callback verification, replay, idempotency, and reconciliation
- Device approval, revocation, offline snapshot freshness, local storage
- Signed/integrity-checked offline events and sync idempotency
- Conflicting offline scans and audit preservation
- Input validation, rate limiting, CSRF/CORS, secure headers, dependency risks
- Secret handling, PII exposure, logs, backups, and retention
- Admin privilege escalation and manual validation overrides

## Rules

- Do not claim a vulnerability exists without showing evidence.
- Distinguish confirmed findings, plausible risks, and unverified assumptions.
- Rank findings by severity and explain impact and exploit prerequisites.
- Provide file/line references where possible.
- Recommend a fix and a regression test for each finding.
- Do not make destructive changes during an audit.
- Do not introduce custom cryptography without a reviewed design.
- Flag unresolved decisions that prevent a reliable conclusion, especially the offline conflict comparator and snapshot TTL.

## Report

1. Scope and files reviewed
2. Executive summary
3. Findings by severity with evidence
4. Exploit scenario and impact
5. Recommended remediation and test
6. Unresolved assumptions/open decisions
7. Explicitly state what was not tested