# BlackTicket

Bangladesh-focused event ticketing platform. This repository is currently in the **documentation-first planning phase**. Production application code must not be started until the core specifications and blocking decisions are reviewed and approved.

## Documentation

- [Platform PRD](docs/PLATFORM_PRD.md) — product goals, actors, scope, requirements, and acceptance criteria
- [Master Specification](docs/MASTER_SPEC.md) — source-of-truth rules and document map
- [System Architecture](docs/SYSTEM_ARCHITECTURE.md) — proposed v1 architecture
- [Open Decisions](docs/OPEN_DECISIONS.md) — unresolved decisions and blockers
- [Database Architecture](docs/DATABASE_ARCHITECTURE.md) — logical schema and integrity constraints
- [Authorization and RBAC](docs/AUTH_RBAC.md) — roles, scopes, and authorization rules
- [Ticket System Specification](docs/TICKET_SYSTEM_SPEC.md) — ticket identity, credential, lifecycle, and issuance
- [Validation Engine Specification](docs/VALIDATION_ENGINE_SPEC.md) — online/offline scan and conflict-handling requirements
- [Offline and Sync Specification](docs/OFFLINE_SYNC_SPEC.md) — v1 offline mode and central synchronization
- [Security Architecture](docs/SECURITY_ARCHITECTURE.md) — threats and security requirements
- [Testing Strategy](docs/TESTING_STRATEGY.md) — required test coverage and release gates
- [AI Master Prompt](docs/AI_MASTER_PROMPT.md) — instructions for AI coding tools
- [Task Prompt](docs/prompts/TASK.md) — implementation-task template
- [Debug Prompt](docs/prompts/DEBUG.md) — evidence-led debugging template
- [Security Audit Prompt](docs/prompts/SECURITY_AUDIT.md) — security review template

## Current planning baseline

- Proposed stack: Laravel modular monolith, MySQL/MariaDB, and a PWA scanner, subject to host compatibility verification.
- v1 offline model: local ticket consumption on an approved device, followed by central synchronization. Peer-to-peer/mesh synchronization is deferred.
- QR credentials must be opaque and contain no personal information. Public ticket IDs are identifiers, not secrets.
- Conflicting offline validation events must both be retained and audited; do not use last-write-wins or silently delete an event.
- Payment gateway selection remains open; use an adapter boundary until the provider is chosen.
- No production code until core specifications and blocking decisions are approved.

The product/domain name and future domain are not deployment approvals. All URLs must be configuration-driven, never hard-coded throughout the application.