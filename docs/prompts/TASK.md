# Task Prompt Template

Use this prompt for one focused BlackTicket task.

## Prompt

You are working in the BlackTicket repository. First inspect `README.md`, `docs/MASTER_SPEC.md`, `docs/PLATFORM_PRD.md`, `docs/OPEN_DECISIONS.md`, and the domain specs relevant to this task.

**Task:** [describe one bounded task]

**Allowed scope:** [list files/modules allowed to change]

**Out of scope:** [list what must not change]

Before editing:
1. Restate the requirement and acceptance criteria.
2. Identify dependencies, contradictions, and open decisions.
3. If a blocking decision is unresolved, do not guess; update the decision register or ask for approval.
4. State a short implementation/test plan.

While working:
- Preserve established invariants.
- Prefer the smallest complete change.
- Add or update tests for behavior and failure cases.
- Do not hard-code domains, secrets, or environment-specific values.
- Do not introduce unrelated refactors.

After working:
- List changed files and explain why.
- Report commands/tests actually run and their real outcomes.
- List known limitations and remaining decisions.
- Do not claim success without evidence.

**Current phase reminder:** no production code until the project owner approves the required core specifications and blocking decisions.