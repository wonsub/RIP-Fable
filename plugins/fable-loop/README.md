# fable-loop (Codex edition)

A general-purpose autonomous agent loop for the [Codex CLI](https://github.com/openai/codex),
ported from the Claude Code edition in this repo. It turns a non-trivial task into a
disciplined cycle — **plan → build → verify → distill** — with strict role
separation and external memory. Model-agnostic.

### New in v0.2 — operating disciplines (D1–D6)

On top of the four patterns, v0.2 adds six disciplines, each an externalized,
ID'd decision (see `memory/decisions.md`):

- **D1 — Self-authored goals.** The architect role can author/refine an
  underspecified goal and give every lane its own dedicated `/goal`. Codex builds
  lanes in sequence, but each stays independently scoped to a disjoint file set.
- **D2 — Distrust by default.** Re-derive untrusted values (e.g. rebuild a date
  from the engine clock), clamp to legal ranges (a score to 1–5), and gate every
  trust boundary with four-value tests — normal / mapped / None / tampered. See
  `skills/fable-loop-orchestration/references/contracts.md`.
- **D3 — Externalize decisions.** Every non-obvious decision gets an ID plus a
  reason/cost/escape-hatch comment at the decision site, logged in `memory/decisions.md`.
- **D4 — Boundary first.** Declare what you will **not** touch before building, to
  open scope by convergence rather than divergence.
- **D5 — Prefer reversibility.** Layer additively over rewriting; keep one decision
  per commit with its reason preserved.
- **D6 — Broaden "done".** Done = behavior + tests + docs + verification, not just
  "it runs."

## Install

From a local clone (this repo is the marketplace root):

```bash
codex plugin marketplace add /absolute/path/to/RIP-Fable
```

Then enable the plugin. Either use the Codex desktop app's plugin UI, or add this to
`~/.codex/config.toml`:

```toml
[plugins."fable-loop@rip-fable"]
enabled = true
```

Start a new Codex thread so the skill is picked up.

## Use

Trigger the loop in plain language — the `fable-loop-orchestration` skill engages
when a task benefits from a spec, verification, and iteration:

- "Run this as a fable loop"
- "Architect then build the X feature"
- "Use the self-correcting agent loop on this"

## How the Codex port differs from the Claude Code edition

Codex has no session hooks and does not spawn sub-agents, so two patterns are
re-expressed as protocol the single agent follows (see `skills/fable-loop-orchestration/SKILL.md`):

| Claude Code edition          | Codex edition                                             |
|------------------------------|-----------------------------------------------------------|
| `SessionStart` hook reads handoff | Skill instructs the agent to read `memory/handoff.md` first |
| `SubagentStop` hook distills lessons | Skill's distill step appends to `memory/lessons.md`   |
| `architect` / `builder` / `verifier` sub-agents | Roles the one agent adopts in sequence (`references/roles.md`); run the verify step in a fresh thread for independence |

## Layout

```
plugins/fable-loop/
├── .codex-plugin/plugin.json
├── skills/fable-loop-orchestration/
│   ├── SKILL.md
│   └── references/{roles,loop-protocol,memory-protocol,contracts}.md
└── memory/{handoff,gates,lanes,lessons,decisions}.md   # ship empty, populated per run
```

## License

[MIT](../../LICENSE) © Wonsub
