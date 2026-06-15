# RIP-Fable — `fable-loop`

A general-purpose autonomous agent loop for [Claude Code](https://claude.com/claude-code), modeled on Mythos-class loop design. It turns a non-trivial task into a disciplined cycle — **plan → build → verify → distill** — with strict role separation and external memory. Model-agnostic: runs on whatever Claude model is active (Opus 4.8, Sonnet 4.6, etc.).

> *한국어 안내는 아래 [한국어](#한국어) 섹션을 참고하세요.*

> **Two editions.** This repo ships the loop for **both** Claude Code (root
> `fable-loop/`, below) and the **Codex CLI** (`plugins/fable-loop/`). For Codex,
> run `codex plugin marketplace add <repo>` and enable `fable-loop@rip-fable` —
> see [plugins/fable-loop/README.md](plugins/fable-loop/README.md).

---

## English

### What it does

Four patterns, always applied together:

1. **Architect / Builder separation** — the strongest reasoning writes a spec and frozen gates; builders implement against that spec and never redesign.
2. **fail → investigate → verify → distill loop** — failures drive structural fixes, and each resolution is distilled into one line for reuse.
3. **Verifier sub-agent gating** — a separate agent reads the real artifact against frozen gates, replacing unreliable self-critique.
4. **Short-map external memory** — a small always-current handoff map plus linked detail files, instead of a giant context log that rots.

**New in v0.2 — operating disciplines (D1–D6):** on top of the four patterns, six externalized, ID'd disciplines (logged in `memory/decisions.md`):

- **D1 — Self-authored goals.** The architect authors/refines an underspecified goal and gives every lane its own dedicated `/goal`; spawn as many parallel builders as there are disjoint lanes — no more.
- **D2 — Distrust by default.** Re-derive untrusted values (rebuild a date from the engine clock), clamp to legal ranges (a score to 1–5), and gate every trust boundary with four-value tests — normal / mapped / None / tampered.
- **D3 — Externalize decisions.** Each non-obvious decision gets an ID plus a reason/cost/escape-hatch comment at the decision site.
- **D4 — Boundary first.** Declare what you will *not* touch before building — open scope by convergence, not divergence.
- **D5 — Prefer reversibility.** Layer additively over rewriting; one decision per commit with its reason preserved.
- **D6 — Broaden "done".** Done = behavior + tests + docs + verification, not just "it runs."

### Install

Requires [Claude Code](https://docs.claude.com/en/docs/claude-code). In an interactive `claude` session, run:

```
/plugin marketplace add wonsub/RIP-Fable
/plugin install fable-loop@rip-fable
```

The first command registers this GitHub repo as a plugin marketplace; the second installs the `fable-loop` plugin from it. Restart the session when prompted so the agents, skill, and hooks load.

<details>
<summary>Alternative: install from a local clone</summary>

```
git clone https://github.com/wonsub/RIP-Fable.git
```

Then, in `claude`:

```
/plugin marketplace add /absolute/path/to/RIP-Fable
/plugin install fable-loop@rip-fable
```
</details>

### Verify it loaded

```
/plugin            # fable-loop should appear as installed
/agents            # fable-architect, fable-builder, fable-verifier should be listed
```

### Apply it

Trigger the loop in plain language — the `fable-loop-orchestration` skill engages automatically when a task benefits from a spec, verification, and iteration. Examples:

- "Run this as a fable loop"
- "Architect then build the X feature"
- "Use the self-correcting agent loop on this"

For trivial one-shot tasks the loop stays out of the way.

### Components

| Type   | Name                       | Purpose                                                |
|--------|----------------------------|--------------------------------------------------------|
| Skill  | `fable-loop-orchestration` | Drives the whole loop; loads protocol references.      |
| Agent  | `fable-architect`          | Writes spec + frozen gates. Never implements.          |
| Agent  | `fable-builder`            | Implements one lane against its spec. No redesign.     |
| Agent  | `fable-verifier`           | Independent PASS/FAIL against frozen gates.            |
| Hooks  | SessionStart, SubagentStop | Loads the short map; distills lessons after subagents. |

### Memory layout

The plugin keeps external memory under its own `memory/` directory:

```
memory/
├── handoff.md   # short map — always current, read first / written last
├── gates.md     # frozen acceptance gates
├── lanes.md     # lane specs with disjoint file sets
├── lessons.md   # distilled one-line lessons (append-only, cross-session)
└── decisions.md # externalized decision log (D3): IDs + reason/cost/escape-hatch
```

These ship empty and are populated per run.

### Requirements

No environment variables, no external services. Drop it in and go.

### Uninstall

```
/plugin uninstall fable-loop@rip-fable
/plugin marketplace remove rip-fable
```

### Codex CLI edition

A Codex-native port lives in [`plugins/fable-loop/`](plugins/fable-loop/). It carries the same four patterns; because Codex has no session hooks and no spawnable sub-agents, the hook behavior and the architect/builder/verifier roles are folded into the skill protocol.

**Install** — in a terminal, register this repo as a Codex marketplace (named `rip-fable`):

```bash
codex plugin marketplace add wonsub/RIP-Fable
```

`owner/repo`, an HTTPS/SSH Git URL, or a local clone path all work as the source.

**Enable** — either turn it on in the Codex desktop app's plugin UI, or add this block to `~/.codex/config.toml`:

```toml
[plugins."fable-loop@rip-fable"]
enabled = true
```

**Apply** — start a new Codex thread (so the skill loads), then trigger it the same way: "Run this as a fable loop". See [plugins/fable-loop/README.md](plugins/fable-loop/README.md) for details.

**Uninstall** — remove the `[plugins."fable-loop@rip-fable"]` block from `~/.codex/config.toml`, then:

```bash
codex plugin marketplace remove rip-fable
```

---

## 한국어

### 개요

[Claude Code](https://claude.com/claude-code)용 범용 자율 에이전트 루프 플러그인입니다. 비자명한 작업을 **계획 → 구현 → 검증 → 정제(plan → build → verify → distill)** 사이클로 규율 있게 처리합니다. 역할을 엄격히 분리하고 외부 메모리를 사용하며, 현재 활성화된 어떤 Claude 모델에서도 동작합니다(Opus 4.8, Sonnet 4.6 등).

### 네 가지 패턴

1. **아키텍트/빌더 분리** — 가장 강한 추론이 명세와 동결된 합격 기준(gate)을 작성하고, 빌더는 그 명세대로 구현만 하며 재설계하지 않습니다.
2. **fail → investigate → verify → distill 루프** — 실패가 구조적 수정을 유도하고, 해결된 내용은 한 줄로 정제해 재사용합니다.
3. **검증 서브에이전트 게이팅** — 별도 에이전트가 실제 산출물을 동결된 gate에 대해 검증해, 신뢰할 수 없는 자기검증을 대체합니다.
4. **짧은 지도(short-map) 외부 메모리** — 썩어가는 거대한 컨텍스트 로그 대신, 항상 최신인 작은 핸드오프 지도 + 링크된 상세 파일을 사용합니다.

**v0.2 신규 — 운영 규율(D1–D6):** 네 가지 패턴 위에, ID가 붙어 외부화된 여섯 가지 규율을 추가합니다(`memory/decisions.md`에 기록):

- **D1 — 스스로 goal 작성.** 아키텍트가 불충분한 goal을 직접 작성·정교화하고, 각 레인에 전용 `/goal`을 부여합니다. 겹치지 않는 레인 수만큼만 병렬 빌더를 spawn합니다(그 이상은 금지).
- **D2 — 기본값은 불신(계약적 사고).** 신뢰할 수 없는 값은 재유도하고(모델이 준 날짜를 버리고 엔진 시계로 재조립), 합법 범위로 clamp하며(점수는 1–5), 모든 신뢰 경계를 4종 경계값 테스트(정상 / 매핑 / None / 변조)로 두릅니다.
- **D3 — 결정의 외화.** 비자명한 결정마다 ID를 붙이고, 결정 지점 주석에 이유·비용·탈출구를 남깁니다.
- **D4 — 경계 먼저 선언.** 풀기 전에 "건드리지 않을 것"을 적어 변경 반경을 좁힙니다 — 발산이 아니라 수렴으로 스코프를 엽니다.
- **D5 — 가역성 선호.** 갈아엎지 않고 위에 얹으며, 결정 하나당 커밋 하나로 변경 이유를 보존합니다.
- **D6 — 넓은 '완료' 정의.** 완료 = 동작 + 테스트 + 문서 현행화 + 검증이며, 단지 "코드가 도는 것"이 아닙니다.

### 설치

[Claude Code](https://docs.claude.com/en/docs/claude-code)가 필요합니다. 대화형 `claude` 세션에서 실행하세요:

```
/plugin marketplace add wonsub/RIP-Fable
/plugin install fable-loop@rip-fable
```

첫 명령은 이 GitHub 저장소를 플러그인 마켓플레이스로 등록하고, 두 번째 명령은 거기서 `fable-loop` 플러그인을 설치합니다. 안내가 뜨면 세션을 재시작해 에이전트·스킬·훅을 로드하세요.

<details>
<summary>대안: 로컬 클론으로 설치</summary>

```
git clone https://github.com/wonsub/RIP-Fable.git
```

그런 다음 `claude`에서:

```
/plugin marketplace add /절대경로/RIP-Fable
/plugin install fable-loop@rip-fable
```
</details>

### 로드 확인

```
/plugin            # fable-loop이 설치됨으로 표시
/agents            # fable-architect, fable-builder, fable-verifier 목록 확인
```

### 적용 방법

자연어로 루프를 트리거하면 됩니다 — 명세·검증·반복이 필요한 작업이면 `fable-loop-orchestration` 스킬이 자동으로 작동합니다. 예:

- "이걸 fable 루프로 진행해줘"
- "X 기능을 architect부터 하고 build해줘"
- "자기교정 에이전트 루프로 처리해줘"

사소한 단발성 작업에서는 루프가 끼어들지 않습니다.

### 구성 요소

| 종류  | 이름                       | 역할                                            |
|-------|----------------------------|-------------------------------------------------|
| Skill | `fable-loop-orchestration` | 전체 루프를 구동하고 프로토콜 참조를 로드.      |
| Agent | `fable-architect`          | 명세 + 동결된 gate 작성. 구현은 하지 않음.      |
| Agent | `fable-builder`            | 한 레인을 명세대로 구현. 재설계 금지.           |
| Agent | `fable-verifier`           | 동결된 gate에 대해 독립적 PASS/FAIL 판정.       |
| Hooks | SessionStart, SubagentStop | 짧은 지도 로드, 서브에이전트 후 lesson 정제.    |

### 메모리 구조

플러그인은 자체 `memory/` 디렉터리에 외부 메모리를 둡니다:

```
memory/
├── handoff.md   # 짧은 지도 — 항상 최신, 가장 먼저 읽고 가장 마지막에 씀
├── gates.md     # 동결된 합격 기준
├── lanes.md     # 겹치지 않는 파일셋을 가진 레인 명세
├── lessons.md   # 한 줄로 정제된 교훈 (추가 전용, 세션 간 공유)
└── decisions.md # 외화된 결정 로그 (D3): ID + 이유/비용/탈출구
```

### 요구 사항

환경 변수·외부 서비스 불필요. 설치만 하면 바로 사용 가능합니다.

### 제거

```
/plugin uninstall fable-loop@rip-fable
/plugin marketplace remove rip-fable
```

### Codex CLI 에디션

Codex 전용 포트는 [`plugins/fable-loop/`](plugins/fable-loop/)에 있습니다. 네 가지 패턴은 동일하며, Codex에는 세션 훅·spawn 가능한 서브에이전트가 없어 훅 동작과 architect/builder/verifier 역할은 스킬 프로토콜에 흡수돼 있습니다.

**설치** — 터미널에서 이 저장소를 Codex 마켓플레이스(`rip-fable`)로 등록합니다:

```bash
codex plugin marketplace add wonsub/RIP-Fable
```

소스로는 `owner/repo`, HTTPS/SSH Git URL, 로컬 클론 경로 모두 사용 가능합니다.

**활성화** — Codex 데스크톱 앱의 플러그인 UI에서 켜거나, `~/.codex/config.toml`에 다음 블록을 추가합니다:

```toml
[plugins."fable-loop@rip-fable"]
enabled = true
```

**적용** — 새 Codex 스레드를 시작하면(스킬 로드) 동일하게 자연어로 트리거하세요: "이걸 fable 루프로 진행해줘". 자세한 내용은 [plugins/fable-loop/README.md](plugins/fable-loop/README.md) 참고.

**제거** — `~/.codex/config.toml`에서 `[plugins."fable-loop@rip-fable"]` 블록을 삭제한 뒤:

```bash
codex plugin marketplace remove rip-fable
```

---

## License

[MIT](./LICENSE) © Wonsub
