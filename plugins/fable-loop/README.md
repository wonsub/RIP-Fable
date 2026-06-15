# fable-loop (Codex edition)

A general-purpose autonomous agent loop for the [Codex CLI](https://github.com/openai/codex),
ported from the Claude Code edition in this repo. It turns a non-trivial task into a
disciplined cycle — **plan → build → verify → distill** — with strict role
separation and external memory. Model-agnostic.

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
│   └── references/{roles,loop-protocol,memory-protocol}.md
└── memory/{handoff,gates,lanes,lessons}.md   # ship empty, populated per run
```

## License

[MIT](../../LICENSE) © Wonsub
