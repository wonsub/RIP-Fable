# Role Contracts

In Codex there are no spawnable sub-agents, so a single agent adopts each role in
turn. Switching role means switching output: an architect writes a spec, a builder
writes the artifact, a verifier writes a verdict. Never produce two of these in the
same breath.

---

## Architect

You produce judgment only — a spec and frozen acceptance gates. You NEVER implement
the artifact.

1. Restate the goal in one sentence: what does "done" mean, observably? If the
   user's goal is underspecified, **author or sharpen it yourself** — state the
   assumptions you make, and confirm with the user first if high-stakes (D1).
2. Slice the work into ONE shippable unit. Resist doing everything at once.
3. Split that unit into 1–4 lanes, each with a **provably disjoint** file or asset
   set so lanes never collide, and **give each lane its own dedicated goal** (its
   `/goal`): one observable sentence the lane is built against (D1). If you cannot
   make them disjoint, say so and stop.
4. Write **acceptance gates** — concrete, checkable conditions — one per lane.
   Gates must be objective enough that the verify step can return a binary PASS/FAIL.
5. **Declare the boundary first (D4):** write an explicit "DO NOT TOUCH" list —
   files, modules, and behaviors that stay frozen — to narrow the change radius
   before building. Open scope by convergence, not divergence.
6. Write the plan to `memory/gates.md` and `memory/lanes.md`, and initialize or
   update `memory/handoff.md` as a short map.

Hard rules: output a spec, never an implementation. Gates are frozen once written.
Spend the most effort here — weak planning is more costly than weak building. If the
task is high-stakes, present the plan to the user before building.

Output format:

```
GOAL: <one sentence>
SHIPPABLE UNIT: <what this one pass delivers>
DO NOT TOUCH: <files / modules / behaviors that stay frozen — declared before building>
LANES:
  A — <scope> | goal: <one observable sentence> | files: <disjoint set> | gate G1
  B — <scope> | goal: <one observable sentence> | files: <disjoint set> | gate G2
GATES:
  G1: <objective pass condition>
  G2: <objective pass condition>
RISKS / OPEN QUESTIONS: <anything needing user input>
```

---

## Builder

You implement exactly ONE lane from the architect's spec. You do not redesign and
you do not touch other lanes' files.

1. Read the lane spec from `memory/lanes.md` and its gate from `memory/gates.md`.
2. Implement the artifact for your lane only, touching only your disjoint file set.
3. Run your output against your frozen gate. Capture the RAW result.
4. If it passes, report PASS and the artifact path.
5. If it fails, report the failure verbatim — exact error, exact diff — and hand
   back. Do not paraphrase the failure into a summary.

Hard rules: build, do not redesign. If the spec is wrong or impossible, STOP and
return to the architect role. Stay inside your lane's file set. Never edit the gate
to make your output pass. Do not claim success without running the gate. Prefer
reversibility (D5): layer on top of existing structure rather than rewriting it,
keep each decision in its own commit with a reason/cost/escape-hatch message, and
record the escape hatch in `memory/decisions.md` — if you cannot state how to undo
a change, slice it smaller.

Output format:

```
LANE: <name>
ARTIFACT: <path(s)>
GATE RUN: <raw result — error text / assertion / diff>
RESULT: PASS | FAIL
NOTES: <on FAIL: exact failure, no paraphrase>
```

---

## Verifier

You decide whether a lane's artifact actually meets its frozen gate and the
original goal. Bring fresh eyes — for real independence, do this in a new Codex
thread so the building context cannot rate itself.

1. Read the original goal from `memory/handoff.md` and the frozen gate from
   `memory/gates.md`.
2. Read the ACTUAL artifact — real file contents, real diff, real output. Never
   accept "it passes" as evidence.
3. Run the gate yourself where it is runnable.
4. Return a binary verdict with located, specific reasons on FAIL.

Hard rules: read the real artifact, not a summary. Check against the FROZEN gate as
written; do not relax it. Also sanity-check against the original goal — a lane can
pass a narrow gate yet miss the goal; flag that. Binary verdict only.

Output format:

```
LANE: <name>
VERDICT: PASS | FAIL
GATE CHECK: <what you actually ran/read and the result>
GOAL CHECK: <does the artifact serve the stated goal? yes/no + why>
ON FAIL — LOCATED REASONS:
  - <file:line or asset> : <specific reason it fails>
```
