# Changelog

All notable changes to Harness Engineering. Format inspired by [Keep a Changelog](https://keepachangelog.com).

Living document `~/.claude/projects/.../memory/decisions.md`는 reasoning 보존용 (왜 이렇게 결정했는지). 본 CHANGELOG는 **변경 사실** 표준 위치.

## [Unreleased]

(다음 release에 들어갈 변경 누적)

---

## [0.3.1] — 2026-05-06

### Added
- `docs/troubleshooting.md` — install·운영 중 만난 6가지 실제 에러와 fix 모음 (plugin not found, hooks load fail, settings.local 혼동, multi-PC path artifact, monorepo typecheck 속도, CRLF/LF 경고)
- `CHANGELOG.md` — 표준 위치 changelog (이 파일)
- `.gitattributes` — line-ending 정규화 (LF in repo, native checkout). Windows commit 시 CRLF/LF 경고 제거
- `.gitignore` 추가 — 3 scaffold 각각 (`.claude/settings.local.json`·OS·stack-specific 항목)

### Changed
- `README.md` — CHANGELOG 및 troubleshooting 링크 추가

---

## [0.3.0] — 2026-05-06

### Added
- **`scaffolds/turbo-monorepo/`** — 3번째 scaffold. bamtok 적용 경험 환류 (Turbo + pnpm workspace, multi-app). pnpm/turbo/prisma/docker permissions
- **`.github/workflows/ci.yml`** — GitHub Actions로 invariant 자동 검증:
  - JSON 6개 유효성
  - 4원칙 full-text 정확히 4 파일 임베드
  - marketplace·plugin version 일치
  - 3 scaffold 모두 CLAUDE.md + settings.json 존재
  - scaffold CLAUDE.md ≤ 80줄 hint (warn-only)
  - ADR README가 참조하는 ADR 파일 실재

### Changed
- `README.md` — turbo-monorepo scaffold + CI 링크 추가
- `docs/install.md` — turbo-monorepo cp 명령 추가

### Tag
[`v0.3.0`](https://github.com/lsy910810-png/leebroHarness/releases/tag/v0.3.0)

---

## [0.2.1] — 2026-05-05

### Fixed
- **`hooks.json` schema**: 최상위에 `hooks` 키 wrapper 추가. 이전엔 events(`PostToolUse`, `Stop`)를 직접 최상위에 둬서 Claude Code의 plugin loader가 "expected record, received undefined" 에러 발생 → 모든 hooks 작동 불가
- 변경 형식:
  ```
  Before: { "PostToolUse": [...], "Stop": [...] }
  After:  { "hooks": { "PostToolUse": [...], "Stop": [...] } }
  ```

### Tag
[`v0.2.1`](https://github.com/lsy910810-png/leebroHarness/releases/tag/v0.2.1)

---

## [0.2.0] — 2026-05-05

### Added
- **Progressive Disclosure** ([ADR-0001](docs/decisions/0001-progressive-disclosure.md)):
  - Per-agent 폴더: `agents/code-reviewer/`, `agents/tester/`, `agents/principle-auditor/`
  - Skills: `skills/security-review/` (injection·secrets)
- **principle-auditor agent** + `/audit` command — 4원칙 위반 정기 스캔 (긴 함수/파일·스코프 완화·검증 부재·vague 코멘트)
- **Computational sensors** (next-ts scaffold settings.json hooks):
  - PostToolUse `tsc --noEmit` (typecheck)
  - Stop `pnpm lint && pnpm test`
  - 둘 다 가드 추가: tsconfig.json·package.json scripts 부재 시 skip
- **`docs/architecture-guide.md`** ([ADR-0003](docs/decisions/0003-adopt-openai-harness-patterns.md)) — Forward-only layers, Providers, repository-first context (opt-in)
- **GitHub repo 승격** ([ADR-0004](docs/decisions/0004-adopt-github-public-marketplace.md)) — `lsy910810-png/leebroHarness`, MIT license. local file:// 외 GitHub URL로도 marketplace add 가능
- **`LICENSE`** (MIT)
- **`docs/decisions/`** — ADR 시스템 (3 baseline + 1 GitHub adoption)
- **`docs/apply-to-existing-project.md`** — 기존 프로젝트에 머지 가이드 (4 case 분기)
- Memory 구조 정리: `feedback_principles.md` + `decisions.md` + `harness_layout.md`

### Changed
- **scaffold CLAUDE.md slim화** (~150줄 → ~60줄): condensed 4원칙 + visibility protocol short form. 전문은 PRINCIPLES.md 링크
- **`marketplace.json` schema fix** ([ADR-0004](docs/decisions/0004-adopt-github-public-marketplace.md)): top-level `name` + `owner` 필드 추가, plugin entry는 `path` → `source`
- **Stop hook cross-platform**: Windows powershell beep + Mac/Linux BEL char 분기
- **scaffold CLAUDE.md broken github 링크 정리** (`https://github.com/`로 가는 placeholder 제거 → 실제 URL 복원)
- **Visibility Protocol 3-tier** ([ADR-0002](docs/decisions/0002-visibility-protocol-3-tier.md)): 시작 앵커 + 결정 인라인 태그 + 종료 체크 — 매 응답에서 4원칙 적용 가시화

### Tag
[`v0.2.0`](https://github.com/lsy910810-png/leebroHarness/releases/tag/v0.2.0)

---

## [0.1.0] — 2026-05-04

### Added
- **4 Operating Principles** (PRINCIPLES.md): Think Before Coding · Simplicity First · Surgical Changes · Goal-Driven Execution
- **Plugin** (`plugins/harness/`):
  - agents: `code-reviewer`, `tester`
  - commands: `/review`, `/test`
  - hooks: PostToolUse(prettier), Stop(beep)
- **Scaffolds**:
  - `general/` — 언어 무관 baseline
  - `next-ts/` — Next.js + TypeScript
- Local marketplace (`.claude-plugin/marketplace.json`)
- Docs: `install.md`, `extend.md`
- Feedback memory (`feedback_principles.md`)

### Note
Git history 시작 = v0.2.0 commit (`2d8f352`). v0.1.0 시점 baseline은 [decisions.md](https://github.com/lsy910810-png/leebroHarness/blob/main/docs/decisions/README.md)와 ADR로 부분 보존.

---

## Versioning

- **Patch** (0.3.0 → 0.3.1): bug fix, docs only
- **Minor** (0.2.1 → 0.3.0): new scaffold, agent, command, skill (additive)
- **Major** (0.x → 1.0): breaking changes (e.g., 4원칙 텍스트 변경, schema 변경)

새 release 시 본 CHANGELOG의 `[Unreleased]` 섹션에 누적 → tag 시점에 버전 헤더로 promote.
