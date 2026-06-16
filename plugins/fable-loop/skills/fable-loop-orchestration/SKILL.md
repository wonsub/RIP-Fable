---
name: fable-loop-orchestration
description: Use this skill to run a non-trivial, multi-step task as an autonomous agent loop in Codex — phrases like "run this as a loop", "use the fable loop", "architect then build", "self-correcting agent", "long-horizon task", or any task that needs plan → build → verify → distill iteration with self-correction. Applies to coding, content pipelines, research synthesis, and any goal where the work should be verified before being accepted.
---

# Fable Loop Orchestration

Run a task as a disciplined autonomous loop built on four patterns observed in
Mythos-class loop design. You never blend roles: plan, build, verify, and remember
are separate steps with separate outputs.

> **Codex note.** Codex runs this as a single agent that switches roles in
> sequence — it does not spawn separate sub-agents and has no session hooks. The
> role discipline and the verification gate are therefore enforced by *you*
> following this protocol: change roles deliberately, freeze the gates in a file
> before building, and verify by reading the real artifact. For the strongest
> independent verification, run the verify step in a fresh Codex thread
> (`codex` new conversation, or `codex resume` a clean one) so the context that
> built the artifact is not the context that grades it.

## When to engage

Engage for any task that is too large or too risky for a single pass: anything
that benefits from a spec before execution, anything where a wrong result is
expensive, or anything spanning multiple files / scenes / stages. For trivial
one-shot answers, do NOT run the loop — just answer.

## The four patterns (always apply all four)

### 1. Architect / Builder separation

The strongest reasoning does the **design**; execution follows an exhaustive spec.
Never let the same step both decide and implement — weak planning hurts more than
weak execution.

- **Architect role**: produce a spec only. No implementation. Slice the work into
  one shippable unit, then split that unit into 1–4 **lanes with disjoint file
  (or asset) sets** so lanes never collide. Write acceptance gates BEFORE any build.
- **Builder role**: each lane gets the full spec + its gates, and produces the
  artifact. The builder does not redesign; if a spec is wrong, it stops and
  returns to the architect role, it does not improvise.

Gathering/building can interleave. **Synthesis and design are done in one focused
pass, never split across half-finished lanes.**

### 2. fail → investigate → verify → distill loop

This is the core iteration. On every lane:

1. **Run** the builder output against its gates.
2. **fail** — capture the exact failure, not a summary.
3. **investigate** — find the structural cause, not the surface symptom. Prefer a
   structural change over a patch.
4. **verify** — re-run the gate. If still failing, return to investigate (cap at a
   set retry budget, then escalate to the user).
5. **distill** — compress what was learned into one or two lines and write it to
   external memory (see pattern 4). The distilled lesson, not the full transcript,
   is what carries forward.

Use `references/loop-protocol.md` for the exact step contract and retry budget.

### 3. Independent verification gating

Do NOT rely on self-critique from the same reasoning that produced the work.
Self-critique is unreliable because the same context that produced the error also
rates it. Instead, run a dedicated **verify** step against the original goal and
the acceptance gates, ideally in a fresh Codex thread.

- Gates are **frozen and external** — written to `memory/gates.md` before the
  build, never edited mid-run to make a failing artifact pass.
- The verify step reads the actual diff / actual output, not the builder's claim
  that it works. Read the real artifact; do not accept "it passes" as evidence.
- The verify step returns a binary PASS / FAIL plus specific, located reasons on
  FAIL.

The role contract for architect, builder, and verifier is in `references/roles.md`.

### 4. Short-map external memory

Memory files rot. Never carry a giant running log in context.

- Keep a **short map** as the handoff: current goal, lane status table, and links
  to detail files. This lives in `memory/handoff.md`.
- Detail (full gate definitions, lane specs, distilled lessons) lives in separate
  linked files, loaded only when needed.
- Intermediate state that does not need reasoning belongs **outside the context
  window** — in files — not stuffed into the prompt.
- At each stage boundary, update the short map and prune anything the next stage
  does not need.

See `references/memory-protocol.md` for the exact file layout.

## Operating disciplines (v0.2)

The four patterns above are *how the loop runs*. These disciplines are *how each
role behaves* inside it. Each is an externalized decision logged in
`memory/decisions.md` (see pattern D3).

### D1 — Self-authored goals; sequential roles with dedicated goals

- When the user's goal is **underspecified**, the architect role authors or
  sharpens it itself — stating the assumptions it makes, and confirming with the
  user first when the task is high-stakes.
- Every lane gets its **own dedicated goal** (its `/goal`): one observable sentence
  for that lane, not merely a file set. The builder role works from its lane goal,
  not the whole project goal.
- Codex has no spawnable sub-agents, so lanes are built in sequence rather than in
  parallel — but each lane is still scoped to its own dedicated goal and disjoint
  file set, so the work could fan out to parallel agents on a host that supports
  them. Carve **as many lanes as needed, no more.**

### D4 — Declare the boundary before solving

Before any build, the architect role writes down what it will **not** touch — an
explicit out-of-scope / "do-not-touch" list — to narrow the change radius up
front. Open scope by **convergence, not divergence**: name the files, modules, and
behaviors that stay frozen, so the build can only grow inward toward the goal.

### D5 — Prefer reversibility

Build **additively**: layer new behavior on top of what exists instead of rewriting
it, so a change can be undone by removing it rather than reconstructing the old
code. Preserve each decision's **reason in its own commit** — one decision, one
commit message carrying reason · cost · escape-hatch — and record the escape hatch
in `memory/decisions.md`. If you cannot name how to undo a change, it is too big;
slice it.

### D6 — "Done" means more than "it runs"

A unit is done only when all four hold: it **works** (behavior), it is **tested**
(including the four-value boundary tests from D2 wherever a trust boundary is
crossed), its **docs are current**, and it has passed **independent verification**.
Code that merely runs is not done — the verifier checks all four, not just the gate.

## Execution order

1. **Read `memory/handoff.md` first.** (This replaces the SessionStart hook from
   the Claude Code edition.) If it holds an in-progress task, resume from its
   "Next action". If it is the empty template or absent, initialize it.
2. **Architect**: write the spec, lanes, and frozen gates to `memory/gates.md`
   and `memory/lanes.md`; update `memory/handoff.md`. Stop and let the user
   confirm scope if the task is high-stakes.
3. **Build** each lane against its **dedicated lane goal**, spec, and gate. Codex
   builds lanes sequentially (it has no spawnable sub-agents), but each lane is
   still scoped to its own `/goal` and disjoint file set — so on a host that
   supports sub-agents the lanes fan out to one parallel builder each, no more (D1).
4. **Verify** each lane against frozen gates by reading the real artifact —
   preferably in a fresh thread.
5. **Loop** failing lanes through fail→investigate→verify→distill until they pass
   or hit the retry budget.
6. **Distill** lessons into `memory/lessons.md`; update the short map. (This
   replaces the SubagentStop hook: after each resolved failure, append one line
   "Symptom X caused by Y, fixed by Z" and refresh `handoff.md`.)
7. **Report** finished work for human review — present the artifact, not a
   play-by-play.

## Guardrails

- One architect pass per work block — judgment only, never writes the artifact.
- Never widen scope mid-loop. New scope = new architect pass.
- Never edit a gate to force a pass. A gate change requires a new architect decision.
- Escalate to the user when the retry budget is exhausted, when lanes can't be made
  disjoint, or when a gate itself appears wrong.
- This loop is model-agnostic. It runs on whatever model Codex is configured with —
  it does not require any specific model to function.
