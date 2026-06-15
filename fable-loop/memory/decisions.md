# Decision Log (D3)

Externalize decisions as traceable objects. Every non-obvious decision gets an ID
(D1, D2, …) here, and the artifact itself carries a comment with the SAME ID plus
**reason · cost · escape-hatch**. Decisions live in writing — in this file and in
the code — never only in chat or in your head.

## Format

| ID | Decision | Reason | Cost / trade-off | Escape hatch (how to undo) |
|----|----------|--------|------------------|----------------------------|
| D1 | <what was decided> | <why this, vs the alternative> | <what it costs> | <how to reverse it> |

Mirror the row at the decision site as a code comment:

    # D1: <decision> — reason: <why>; cost: <trade-off>; escape: <how to undo>

## Rules

1. Every non-obvious decision gets an ID BEFORE it is implemented.
2. The artifact carries the same ID in a comment, so the decision is discoverable
   from the code, not just from this file.
3. Record the **escape hatch**. If you cannot state how to undo it, the decision
   is probably too big — slice it (this is the reversibility preference, D5).
4. Append-only within a run; supersede (don't silently edit) with a new row that
   references the one it replaces.

(empty — populated per run)
