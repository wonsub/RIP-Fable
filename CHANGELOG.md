# Changelog

All notable changes to the **fable-loop** plugin are documented here.
이 문서는 **fable-loop** 플러그인의 주요 변경 사항을 기록합니다.

The format follows [Keep a Changelog](https://keepachangelog.com/) and the project
adheres to [Semantic Versioning](https://semver.org/).

---

## [0.2.0] — 2026-06-16

Adds six "operating disciplines" (**D1–D6**) on top of the original four patterns,
applied identically to **both** editions (Claude Code + Codex CLI). Fully additive:
the four-pattern core is preserved, not rewritten.

### English

#### Added — operating disciplines (D1–D6)

- **D1 — Self-authored goals.** The architect can author or sharpen an
  underspecified goal and gives every lane its own dedicated `/goal` (one observable
  sentence). Spawn as many parallel builders as there are disjoint lanes — no more.
- **D2 — Distrust by default (contractual thinking).** Trust nothing by default —
  not external input, not model output, not your future self. Re-derive untrusted
  values (e.g. rebuild a date from the engine clock), clamp to legal ranges (a score
  to 1–5), and gate every trust boundary with a four-value test
  (**normal / mapped / None / tampered**). New doc: `references/contracts.md`.
- **D3 — Externalize decisions.** Every non-obvious decision gets an ID plus a
  `reason · cost · escape-hatch` comment at the decision site. New template:
  `memory/decisions.md`.
- **D4 — Declare the boundary first.** Write an explicit "DO NOT TOUCH" list before
  building, to open scope by **convergence, not divergence**.
- **D5 — Prefer reversibility.** Layer additively over rewriting; preserve each
  decision's reason in its own commit.
- **D6 — Broaden "done".** Done = behavior + tests + docs + verification, not just
  "the code runs."

#### Changed

- Bumped to `0.2.0` across both `plugin.json` files, the marketplace manifest, and
  the Claude `SKILL.md` metadata.
- `SKILL.md` (both editions) gains an **"Operating disciplines (v0.2)"** section.
- `loop-protocol.md` gains a **"Definition of done (D6)"** section.
- Role docs (`agents/*.md` and Codex `references/roles.md`) gain D1/D4/D5/D6 behavior
  and a `DO NOT TOUCH` / `DONE CHECK` field.
- `memory-protocol.md` documents the new `decisions.md` file.
- READMEs (root EN+KR, both editions) document D1–D6.

#### How to apply

**Claude Code** — in an interactive `claude` session:

```
/plugin marketplace update rip-fable
/plugin update fable-loop@rip-fable
```

(If `update` is unavailable in your build: `/plugin uninstall fable-loop@rip-fable`
then `/plugin install fable-loop@rip-fable`.) Restart the session when prompted so
the 0.2.0 agents, skill, and hooks load.

**Codex CLI** — in a terminal:

```bash
codex plugin marketplace upgrade rip-fable
```

Ensure `~/.codex/config.toml` still has `[plugins."fable-loop@rip-fable"] enabled = true`,
then start a new Codex thread so the 0.2.0 skill loads.

### 한국어

#### 추가 — 운영 규율 (D1–D6)

- **D1 — 스스로 goal 작성.** 아키텍트가 불충분한 goal을 직접 작성·정교화하고, 각
  레인에 전용 `/goal`(관측 가능한 한 문장)을 부여합니다. 겹치지 않는 레인 수만큼만
  병렬 빌더를 spawn합니다(그 이상은 금지).
- **D2 — 기본값은 불신(계약적 사고).** 외부 입력도, 모델 출력도, 미래의 자기 자신도
  일단 의심합니다. 신뢰할 수 없는 값은 재유도하고(모델이 준 날짜를 버리고 엔진 시계로
  재조립), 합법 범위로 clamp하며(점수는 1–5), 모든 신뢰 경계를 4종 경계값 테스트
  (**정상 / 매핑 / None / 변조**)로 두릅니다. 신규 문서: `references/contracts.md`.
- **D3 — 결정의 외화.** 비자명한 결정마다 ID를 붙이고, 결정 지점 주석에
  `이유 · 비용 · 탈출구`를 남깁니다. 신규 템플릿: `memory/decisions.md`.
- **D4 — 경계 먼저 선언.** 풀기 전에 "건드리지 않을 것"을 적어, 발산이 아니라
  **수렴**으로 스코프를 엽니다.
- **D5 — 가역성 선호.** 갈아엎지 않고 위에 얹으며, 각 결정의 이유를 별도 커밋으로
  보존합니다.
- **D6 — 넓은 '완료' 정의.** 완료 = 동작 + 테스트 + 문서 현행화 + 검증이며, 단지
  "코드가 도는 것"이 아닙니다.

#### 변경

- 두 `plugin.json`, 마켓플레이스 매니페스트, Claude `SKILL.md` 메타데이터를 모두
  `0.2.0`으로 올렸습니다.
- `SKILL.md`(양 에디션)에 **"Operating disciplines (v0.2)"** 섹션 추가.
- `loop-protocol.md`에 **"Definition of done (D6)"** 섹션 추가.
- 역할 문서(`agents/*.md`, Codex `references/roles.md`)에 D1/D4/D5/D6 동작과
  `DO NOT TOUCH` / `DONE CHECK` 필드 추가.
- `memory-protocol.md`에 신규 `decisions.md` 파일 문서화.
- README(루트 영/한, 양 에디션)에 D1–D6 문서화.

#### 적용 방법

**Claude Code** — 대화형 `claude` 세션에서:

```
/plugin marketplace update rip-fable
/plugin update fable-loop@rip-fable
```

(`update`가 없으면 `/plugin uninstall fable-loop@rip-fable` 후
`/plugin install fable-loop@rip-fable`.) 안내가 뜨면 세션을 재시작해 0.2.0 에이전트·
스킬·훅을 로드하세요.

**Codex CLI** — 터미널에서:

```bash
codex plugin marketplace upgrade rip-fable
```

`~/.codex/config.toml`에 `[plugins."fable-loop@rip-fable"] enabled = true`가 그대로
있는지 확인한 뒤, 새 Codex 스레드를 시작하면 0.2.0 스킬이 로드됩니다.

---

## [0.1.0]

Initial release. The four patterns: architect/builder separation,
fail → investigate → verify → distill iteration, verifier sub-agent gating, and
short-map external memory. Ships for both Claude Code and the Codex CLI.

최초 릴리스. 네 가지 패턴: 아키텍트/빌더 분리, fail → investigate → verify → distill
반복, 검증 서브에이전트 게이팅, 짧은 지도 외부 메모리. Claude Code와 Codex CLI 양쪽
에디션으로 제공.

[0.2.0]: https://github.com/wonsub/RIP-Fable/releases/tag/v0.2.0
[0.1.0]: https://github.com/wonsub/RIP-Fable/releases/tag/v0.1.0
