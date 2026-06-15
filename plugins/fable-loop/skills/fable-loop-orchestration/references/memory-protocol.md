# Memory Protocol

Short-map external memory. The principle: **memory files rot, so the handoff stays
a short map and detail lives in linked files.**

## File layout

```
memory/
├── handoff.md      # THE short map. Always current. Read first, written last.
├── gates.md        # Frozen acceptance gates, written by architect, never edited mid-run.
├── lanes.md        # Lane specs + disjoint file/asset sets.
└── lessons.md      # Distilled one-line lessons, append-only.
```

## handoff.md — the short map

This is the only file always kept in working context. Keep it under ~30 lines.

```markdown
# Handoff — <task name>

## Goal
<one sentence: what "done" means>

## Status
| Lane | Files/Assets | Gate | State |
|------|--------------|------|-------|
| A    | src/api/*    | G1   | PASS  |
| B    | src/ui/*     | G2   | LOOP (budget 2/3) |
| C    | docs/*       | G3   | TODO  |

## Links
- Gates: ./gates.md
- Lane specs: ./lanes.md
- Lessons so far: ./lessons.md

## Next action
<the single next step>
```

## Rules

1. **Read handoff.md first** on every resume. It is the source of truth for state.
   (In Codex, you do this manually at the start of a loop — there is no
   SessionStart hook.)
2. **Write handoff.md last** at every stage boundary, after pruning.
3. **Prune aggressively** — anything the next stage does not need leaves the short
   map and moves to a detail file (or is dropped).
4. **Never inline detail** — full gate text, full specs, full transcripts go in
   their files and are loaded only when that stage needs them.
5. **Intermediate state stays out of context** — large outputs, raw logs, and
   build artifacts live on disk, referenced by path, not pasted into the prompt.
6. **lessons.md is append-only** and is the cross-session memory. Future runs read
   it to avoid repeating resolved mistakes.

## Why not just use the big context window

A large context window is not memory. Stuffing everything in pollutes reasoning,
makes compaction lossy, and lets stale detail override fresh decisions. A short
map + linked detail keeps each stage reasoning over exactly what it needs.
