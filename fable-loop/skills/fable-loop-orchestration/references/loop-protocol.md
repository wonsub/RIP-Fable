# Loop Protocol

Exact step contract for the fail → investigate → verify → distill loop.

## Definition of done (D6)

A lane is **done** only when all four hold — not just the first:

1. **Behavior** — the artifact works against its frozen gate.
2. **Tests** — it is tested, including the four-value boundary tests
   (normal / mapped / None / tampered) wherever it crosses a trust boundary (D2).
3. **Docs** — any docs the change affects are updated to match.
4. **Verification** — it has passed independent verification (verifier sub-agent).

Code that merely runs is not done. The verifier checks all four.

## Retry budget

- Default: **3** investigate→verify cycles per lane.
- After the budget is exhausted, STOP the lane and escalate to the user with: the
  failing gate, the last three investigation notes, and the current artifact state.
- Never silently keep retrying. Looping past the budget usually means the spec or
  the gate is wrong, which is an architect decision, not a builder one.

## Step contract

### Run
- Execute the builder artifact against its frozen gates exactly as written.
- Capture raw output: actual error text, actual failing assertion, actual diff.

### fail
- Record the failure verbatim in the lane's detail file. Do not paraphrase the
  error into a summary — the exact text is the signal.

### investigate
- Ask: what is the structural cause? Trace from symptom to root.
- Prefer a structural fix (change the design of the unit) over a local patch.
- Write a one-line hypothesis before changing anything.

### verify
- Re-run the same frozen gate. No editing the gate.
- PASS → go to distill. FAIL → decrement budget, return to investigate.

### distill
- Compress the resolved problem into 1–2 lines: "Symptom X was caused by Y;
  fixed by Z." Append to `memory/lessons.md`.
- The distilled line, not the transcript, is what future lanes and future
  sessions read.

## Parallelism rule

- Lanes with provably disjoint file/asset sets run in parallel.
- Synthesis (merging lane outputs, writing the final decision/report) is always
  single-threaded and done after all lanes pass.

## Failure escalation template

```
LANE: <name>
GATE FAILED: <gate id + expected vs actual>
INVESTIGATION NOTES:
  1. <hypothesis → result>
  2. <hypothesis → result>
  3. <hypothesis → result>
BUDGET: exhausted
ASK: <specific decision needed from user — usually "is this gate correct?" or
      "should scope change?">
```
