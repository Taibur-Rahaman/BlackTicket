# BlackTicket — AI Master Prompt

You are assisting with the BlackTicket event ticketing platform. Follow repository documentation as the source of truth.

## First action in every new work session

1. Read `README.md`, `docs/MASTER_SPEC.md`, `docs/PLATFORM_PRD.md`, and `docs/OPEN_DECISIONS.md`.
2. Read the relevant architecture/domain specifications before suggesting changes.
3. Identify contradictions, missing requirements, and blocking open decisions.
4. Do not claim to have read files that are not present in the repository.

## Current phase

The project is documentation-first. **Do not write production application code** until the project owner approves the core specifications and blocking decisions. If asked to implement before then, explain which decisions block safe implementation and propose documentation/tests first.

## Architecture baseline

- Proposed Laravel modular monolith + PWA scanner.
- MySQL/MariaDB depending on actual hosting support.
- Payment provider behind an adapter; gateway not selected.
- v1 offline model is local consumption followed by central sync.
- Peer-to-peer/mesh sync is deferred.
- QR credential is opaque and contains no PII; public Ticket ID is not a secret.
- Store credential hashes where feasible.
- Central online consumption must be atomic.
- Inventory reservations must be concurrency-safe.
- Sync is idempotent.
- Audit events are append-only in application behavior.
- Preserve both conflicting offline validation events. Do not use last-write-wins or silently delete evidence.
- Device wall-clock time alone must not decide conflicts.
- Frontend state is never authoritative for payment, inventory, authorization, or ticket consumption.
- No blockchain unless the owner specifically approves a justified use case.
- URLs and secrets are configuration-driven.

## Decision discipline

Never invent a final decision for payment gateway, ticket delivery, refund/transfer policy, event/ticket state machines, snapshot TTL, device approval workflow, QR algorithm, recent-scan window, or offline conflict comparator. Record gaps in `docs/OPEN_DECISIONS.md`.

## When asked to change documentation

- Inspect existing documents first.
- Keep terminology and requirements consistent across docs.
- Clearly distinguish approved decisions from proposals.
- Update cross-references and acceptance criteria.
- Report files changed, unresolved questions, and potential contradictions.
- Do not silently remove requirements.

## When implementation is eventually authorized

- Work in small, reviewable tasks.
- Write tests before or alongside behavior.
- Enforce backend authorization and transactions.
- Do not store/log raw QR credentials, passwords, tokens, or payment secrets.
- Never claim tests passed unless they were actually run and results are available.
- Summarize changed files, commands run, test results, risks, and next step.

## Output format

1. Understanding and scope
2. Relevant docs and decisions
3. Proposed/implemented changes
4. Tests and evidence
5. Open questions and next step