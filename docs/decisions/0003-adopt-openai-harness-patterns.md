# ADR 0003: Selectively Adopt OpenAI Harness Engineering Patterns

- **Status**: Accepted
- **Date**: 2026-05-05
- **Supersedes**: —

## Context

OpenAI가 2026년 초 "Harness engineering: leveraging Codex in an agent-first world" 글로 5개월간 백만 줄 코드를 codex agents로 작성한 경험을 정리. Martin Fowler가 "Harness engineering for coding agent users"로 패턴을 일반화. HumanLayer가 "Skill Issue: Harness Engineering for Coding Agents"로 Claude Code 적용 사례 제시.

이 글들이 v0.1.0 하네스를 만든 후 발견됨. v0.1.0은 4원칙 + visibility protocol + 기본 agents·hooks·scaffolds까지였고, OpenAI/Fowler/HumanLayer가 제시하는 추가 패턴들이 많았음. 모두 도입할지, 일부만 할지, 안 할지 판단 필요.

핵심 이유: 우리 v0.1.0은 inferential control(LLM 기반: code-reviewer, tester)에 치우쳤고 computational control(deterministic: typecheck, lint, structural test)이 약했음. shift-left 정신이 부족.

## Decision

**4가지 패턴을 도입**, 3가지는 명시적으로 도입 안 함:

### 도입한 것

1. **Guides vs Sensors 프레이밍** (Fowler) — 기존 자산을 재해석
   - Guides (feedforward): PRINCIPLES.md, agent prompts, scaffold CLAUDE.md
   - Sensors (feedback): hooks, code-reviewer, tester, principle-auditor
   - 둘 다 필요하다는 framing이 architecture-guide.md에 명시

2. **Recurring cleanup agent** (OpenAI's "garbage collection" 패턴)
   - `principle-auditor` agent 추가
   - 4차원(긴 함수/파일·스코프 완화·검증 부재·vague 코멘트) 스캔
   - `/audit` slash command로 호출
   - 자동 PR 발행 안 함 (보고만, 수정은 사람)

3. **Computational sensors via hooks** (Fowler shift-left, HumanLayer hooks 예시)
   - next-ts scaffold의 settings.json에 PostToolUse(`tsc --noEmit`) + Stop(`npm run lint && npm test`)
   - 범용 scaffold엔 안 박음 (언어 종속이라 보일러플레이트 안 됨)

4. **Repository-first context design** (OpenAI)
   - `docs/architecture-guide.md` 신설 — forward-only layers + Providers + 권장 docs 구조 (map.md, architecture/, domain/, ops/, decisions/)
   - **opt-in**: scaffold 자체엔 강제 안 함, 채택 가이드만 제시

### 명시적으로 도입 안 한 것

- **Forward-only layers strict 강제** (ESLint boundaries 룰을 모든 scaffold에 박는 것) — 프로젝트별 적합도 다름. 가이드까지만.
- **LogQL/PromQL 텔레메트리 통합** — 인프라 종속, 프로젝트마다 다름. 보일러플레이트 X.
- **Auto-PR cleanup agent** — `principle-auditor`는 보고만, 수정은 사람 + 별 PR. OpenAI가 한 자동 수정 PR은 risk 큼 (원칙 3 위반 가능성).

## Consequences

**긍정**:
- v0.2.0에 computational + inferential 양쪽 sensor 갖춤 (이전엔 inferential 위주)
- shift-left 가능 — typecheck PostToolUse가 즉각 피드백
- 4원칙 위반을 정기 스캔할 메커니즘 확보 (recurring cleanup)
- architecture 가이드 문서로 프로젝트가 채택 시 agent legibility 상승

**부정**:
- next-ts의 typecheck PostToolUse가 큰 프로젝트(5000+ 파일)에서 5~30초 걸릴 수 있음 — 사용자가 체감 후 incremental 옵션·디렉토리 좁히기·Stop만 사용 등으로 튜닝 필요
- principle-auditor의 휴리스틱은 false positive 있음 (특히 차원 2: 의도된 큰 PR을 잘못 잡을 수 있음). 사용 후 임계값 조정
- 패턴 도입으로 파일 수 증가 (architecture-guide.md, principle-auditor agent + 폴더, /audit command)

**후속 작업**:
- 첫 `/audit` 결과 보고 휴리스틱 임계값(파일 500줄, 함수 100줄, dir 4+ 등) 조정
- typecheck hook 속도 체감 후 incremental 옵션 도입 검토
- 새 reversal 발생 시 ADR로 history 보존

## Alternatives considered

### A. 모두 도입 (forward-only strict, telemetry, auto-PR)
- **기각 사유**: 모든 프로젝트에 강제하면 무거움. 본 하네스는 baseline — 프로젝트가 추가 채택하는 게 맞음.

### B. 아무것도 안 함 (v0.1.0 그대로)
- **기각 사유**: computational sensor 부재, recurring cleanup 없음, shift-left 정신 약함. 명백한 개선 기회 놓침.

### C. Codex CLI 패턴 그대로 복사
- **기각 사유**: AGENTS.md vs CLAUDE.md, Codex skills vs Claude Code Skills, Codex hooks vs Claude Code hooks 등 primitive가 다름. 번역 필요.

### D. 선택적 도입 (선택됨)
- 위 4개만, 무거운 3개는 가이드까지만.
- **선택 사유**: Simplicity First (원칙 2). 가치 대비 비용이 명확한 것만. 무거운 건 프로젝트 채택 시 추가.

## Related

- 코드 위치:
  - `plugins/harness/agents/principle-auditor.md` + `agents/principle-auditor/{audit-checks,audit-output}.md`
  - `plugins/harness/commands/audit.md`
  - `scaffolds/next-ts/.claude/settings.json` (hooks 섹션)
  - `docs/architecture-guide.md`
- 메모리: `decisions.md` 항목 #7 (출처 + Codex→Claude 번역 표)
- 출처:
  - [Harness engineering: leveraging Codex in an agent-first world | OpenAI (2026)](https://openai.com/index/harness-engineering/) — 원문 직접 fetch 막힘, [InfoQ 요약](https://www.infoq.com/news/2026/02/openai-harness-engineering-codex/) 참조
  - [Harness engineering for coding agent users | Martin Fowler](https://martinfowler.com/articles/harness-engineering.html) — guides/sensors 프레이밍, 3 regulation categories
  - [Skill Issue: Harness Engineering for Coding Agents | HumanLayer](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents) — Claude Code 적용 사례, hooks 예시, AGENTS.md/CLAUDE.md 권고
