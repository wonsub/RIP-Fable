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
└── lessons.md   # distilled one-line lessons (append-only, cross-session)
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
