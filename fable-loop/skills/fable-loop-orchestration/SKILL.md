---
name: fable-loop-orchestration
description: >
  This skill should be used when the user wants to run a non-trivial, multi-step
  task as an autonomous agent loop — phrases like "run this as a loop", "use the
  fable loop", "architect then build", "self-correcting agent", "long-horizon task",
  "delegate to subagents", or any task that needs plan → build → verify → distill
  iteration with self-correction. Applies to coding, content pipelines, research
  synthesis, and any goal where the work should be verified before being accepted.
metadata:
  version: "0.2.0"
---

# Fable Loop Orchestration

Run a task as a disciplined autonomous loop built on four patterns observed in
Mythos-class loop design. The orchestrator (you) never blends roles: plan, build,
verify, and remember are separate steps with separate outputs.

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

- **Architect step**: produce a spec only. No implementation. Slice the work into
  one shippable unit, then split that unit into 1–4 **lanes with disjoint file
  (or asset) sets** so lanes never collide. Write acceptance gates BEFORE any build.
- **Builder step(s)**: each lane gets the full spec + its gates, and produces the
  artifact. Builders do not redesign; if a spec is wrong, they stop and report
  back to the architect, they do not improvise.

Gathering/building can parallelize. **Synthesis and design never parallelize.**

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

### 3. Verifier sub-agent gating

Do NOT rely on self-critique. Self-critique is unreliable because the same context
that produced the error also rates it. Instead, hand the artifact to the
**verifier sub-agent** (`agents/verifier.md`), which checks the artifact against
the original goal and the acceptance gates with fresh context.

- Gates are **frozen and external** — written down before the build, never edited
  mid-run to make a failing artifact pass.
- The verifier reads the actual diff / actual output, not the builder's claim that
  it works. Agents game visible tests; the verifier reads the real artifact.
- Verifier returns a binary PASS / FAIL plus specific, located reasons on FAIL.

### 4. Short-map external memory

Memory files rot. Never carry a giant running log in context.

- Keep a **short map** as the handoff: current goal, lane status table, and links
  to detail files. This lives in `memory/handoff.md`.
- Detail (full gate definitions, lane specs, distilled lessons) lives in separate
  linked files, loaded only when needed.
- Intermediate state that does not need reasoning belongs **outside the context
  window** — in files or variables — not stuffed into the prompt.
- At each stage boundary, update the short map and prune anything the next stage
  does not need.

See `references/memory-protocol.md` for the exact file layout.

## Operating disciplines (v0.2)

The four patterns above are *how the loop runs*. These disciplines are *how each
role behaves* inside it. Each is an externalized decision logged in
`memory/decisions.md` (see pattern D3).

### D1 — Self-authored goals; parallel agents with dedicated goals

- When the user's goal is **underspecified**, the architect authors or sharpens it
  itself — stating the assumptions it makes, and confirming with the user first
  when the task is high-stakes.
- Every lane gets its **own dedicated goal** (its `/goal`): one observable sentence
  for that lane, not merely a file set. A builder is handed its lane goal, not the
  whole project goal.
- Spawn **as many parallel builders as there are disjoint lanes — as many as
  needed, no more.** Parallelism is only for provably non-colliding lanes; design
  and synthesis stay single-threaded.

### D4 — Declare the boundary before solving

Before any build, the architect writes down what it will **not** touch — an
explicit out-of-scope / "do-not-touch" list — to narrow the change radius up
front. Open scope by **convergence, not divergence**: name the files, modules, and
behaviors that stay frozen, so the build can only grow inward toward the goal.

## Execution order

1. **Read** `memory/handoff.md` if it exists (resume) — otherwise initialize it.
2. **Architect**: write the spec, lanes, and frozen gates. Stop and let the user
   confirm scope if the task is high-stakes.
3. **Build** each lane (parallel where file sets are disjoint).
4. **Verify** each lane via the verifier sub-agent against frozen gates.
5. **Loop** failing lanes through fail→investigate→verify→distill until they pass
   or hit the retry budget.
6. **Distill** lessons into memory; update the short map.
7. **Report** finished work for human review — present the artifact, not a play-by-play.

## Guardrails

- One architect session per work block — judgment only, never writes the artifact.
- Never widen scope mid-loop. New scope = new architect pass.
- Never edit a gate to force a pass. A gate change requires a new architect decision.
- Escalate to the user when the retry budget is exhausted, when lanes can't be made
  disjoint, or when a gate itself appears wrong.
- This loop is model-agnostic. It runs on whatever model is active (Opus 4.8, Sonnet
  4.6, etc.) — it does not require any specific model to function.
