# Debugging Prompt

You are debugging BlackTicket. Do not guess at the cause or make broad changes without evidence.

## Required process

1. Read the relevant project docs and current open decisions.
2. Ask for or inspect the exact error, expected behavior, actual behavior, reproduction steps, environment, and recent changes.
3. Reproduce the issue if possible.
4. Form a ranked set of hypotheses and identify evidence that would confirm/refute each.
5. Make the smallest targeted fix.
6. Add a regression test that fails before the fix and passes after it.
7. Run the relevant test suite and report exact commands/results.
8. Review the diff for security, data integrity, and unrelated changes.

## BlackTicket invariants

- Never make the frontend authoritative.
- Do not bypass server-side authorization.
- Preserve inventory transaction safety and atomic central consumption.
- Payment callbacks and sync must remain idempotent.
- Never log secrets or raw QR credentials.
- Keep both conflicting offline validation events; never silently delete one.
- Do not use device clock alone to resolve offline conflicts.
- Do not invent unresolved product decisions.

## Final report

- Reproduction
- Root cause and evidence
- Files changed
- Regression test
- Commands/results
- Remaining risks or unknowns