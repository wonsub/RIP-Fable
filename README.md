# RIP-Fable — `fable-loop`

A general-purpose autonomous agent loop for [Claude Code](https://claude.com/claude-code), modeled on Mythos-class loop design. It turns a non-trivial task into a disciplined cycle — **plan → build → verify → distill** — with strict role separation and external memory. Model-agnostic: runs on whatever Claude model is active (Opus 4.8, Sonnet 4.6, etc.).

> *한국어 안내는 아래 [한국어](#한국어) 섹션을 참고하세요.*

---

## English

### What it does

Four patterns, always applied together:

1. **Architect / Builder separation** — the strongest reasoning writes a spec and frozen gates; builders implement against that spec and never redesign.
2. **fail → investigate → verify → distill loop** — failures drive structural fixes, and each resolution is distilled into one line for reuse.
3. **Verifier sub-agent gating** — a separate agent reads the real artifact against frozen gates, replacing unreliable self-critique.
4. **Short-map external memory** — a small always-current handoff map plus linked detail files, instead of a giant context log that rots.

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
└── lessons.md   # distilled one-line lessons (append-only, cross-session)
```

These ship empty and are populated per run.

### Requirements

No environment variables, no external services. Drop it in and go.

### Uninstall

```
/plugin uninstall fable-loop@rip-fable
/plugin marketplace remove rip-fable
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
└── lessons.md   # 한 줄로 정제된 교훈 (추가 전용, 세션 간 공유)
```

### 요구 사항

환경 변수·외부 서비스 불필요. 설치만 하면 바로 사용 가능합니다.

### 제거

```
/plugin uninstall fable-loop@rip-fable
/plugin marketplace remove rip-fable
```

---

## License

[MIT](./LICENSE) © Wonsub
