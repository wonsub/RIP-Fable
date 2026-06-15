# fable-loop

A general-purpose autonomous agent loop, modeled on Mythos-class loop design. It
turns a non-trivial task into a disciplined cycle: **plan → build → verify →
distill**, with strict role separation and external memory. Model-agnostic — runs
on whatever Claude model is active.

> **Install:** in an interactive `claude` session, run
> `/plugin marketplace add wonsub/RIP-Fable` then
> `/plugin install fable-loop@rip-fable`.
> Full instructions (English / 한국어) are in the [repository README](../README.md).

## What it does

Four patterns, always applied together:

1. **Architect / Builder separation** — the strongest reasoning writes a spec and
   frozen gates; builders implement against that spec and never redesign.
2. **fail → investigate → verify → distill loop** — failures drive structural
   fixes, and each resolution is distilled into one line for reuse.
3. **Verifier sub-agent gating** — a separate agent reads the real artifact against
   frozen gates, replacing unreliable self-critique.
4. **Short-map external memory** — a small always-current handoff map plus linked
   detail files, instead of a giant context log that rots.

### New in v0.2 — operating disciplines (D1–D6)

On top of the four patterns, v0.2 adds six disciplines, each an externalized,
ID'd decision (see `memory/decisions.md`):

- **D1 — Self-authored goals.** The architect can author/refine an underspecified
  goal and give every lane its own dedicated `/goal`; spawn as many parallel
  builders as there are disjoint lanes — no more.
- **D2 — Distrust by default.** Re-derive untrusted values (e.g. rebuild a date
  from the engine clock), clamp to legal ranges (a score to 1–5), and gate every
  trust boundary with four-value tests — normal / mapped / None / tampered. See
  [`references/contracts.md`](skills/fable-loop-orchestration/references/contracts.md).
- **D3 — Externalize decisions.** Every non-obvious decision gets an ID plus a
  reason/cost/escape-hatch comment at the decision site, logged in `memory/decisions.md`.
- **D4 — Boundary first.** Declare what you will **not** touch before building, to
  open scope by convergence rather than divergence.
- **D5 — Prefer reversibility.** Layer additively over rewriting; keep one decision
  per commit with its reason preserved.
- **D6 — Broaden "done".** Done = behavior + tests + docs + verification, not just
  "it runs."

## Components

| Type   | Name                        | Purpose                                            |
|--------|-----------------------------|----------------------------------------------------|
| Skill  | `fable-loop-orchestration`  | Drives the whole loop; loads protocol references.  |
| Agent  | `fable-architect`           | Writes spec + frozen gates. Never implements.      |
| Agent  | `fable-builder`             | Implements one lane against its spec. No redesign. |
| Agent  | `fable-verifier`            | Independent PASS/FAIL against frozen gates.         |
| Hooks  | SessionStart, SubagentStop  | Loads the short map; distills lessons after subagents. |

## Memory layout

```
memory/
├── handoff.md   # short map — always current, read first / written last
├── gates.md     # frozen acceptance gates
├── lanes.md     # lane specs with disjoint file sets
├── lessons.md   # distilled one-line lessons (append-only, cross-session)
└── decisions.md # externalized decision log (D3): IDs + reason/cost/escape-hatch
```

## Usage

Trigger the loop by asking, e.g.:

- "Run this as a fable loop"
- "Architect then build the X feature"
- "Use the self-correcting agent loop on this"

For trivial one-shot tasks the loop stays out of the way — it only engages for
work that benefits from a spec, verification, and iteration.

## Setup

No environment variables required. No external services. Drop it in and go.

## Notes

This plugin encodes loop-design patterns publicly described for Mythos-class
models, but does not depend on any specific model being available. It works the
same on Opus 4.8, Sonnet 4.6, or any current Claude model.
