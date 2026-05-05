# ADR 0001: Adopt Progressive Disclosure for Agent Knowledge

- **Status**: Accepted
- **Date**: 2026-05-04
- **Supersedes**: —

## Context

초기 v0.1.0의 agent 파일(`code-reviewer.md`, `tester.md`)은 4원칙 + visibility protocol + role + 워크플로우 + 도메인 지식(체크리스트·진단 패턴 등)을 **모두 system prompt에 인라인 박음**. 결과는:

- agent.md 파일이 250+ 줄로 비대
- 작은 작업(예: 한 줄 typo 검토)에도 전체 prompt 로드 — 토큰 낭비
- 보안 리뷰 룰처럼 한 agent에 국한되지 않는 지식이 여러 곳에 중복 가능
- agent 추가/수정 시 모든 지식이 한 파일에 묶여 있어 변경 영향 범위 큼

HumanLayer "Skill Issue: Harness Engineering for Coding Agents" 글에서 progressive disclosure 패턴 — agent.md를 슬림한 TOC로 두고 상세 지식을 lazy-load — 이 LLM 성능에 도움된다는 점이 명시됨. ETH Zurich 연구도 큰 agent 파일이 오히려 성능 저하시킨다고 보고.

## Decision

agent 지식을 다음 3계층으로 분리.

1. **Always-loaded** (agent.md 본문, system prompt): 4원칙 전문 + Visibility Protocol + 슬림 role + Knowledge Base TOC
2. **Per-agent lazy-load** (`agents/<name>/<topic>.md`): 해당 agent에만 쓰이는 mechanics — output 형식, severity 룰, framework 탐지 등
3. **Cross-agent skill** (`skills/<name>/SKILL.md` + sibling): 여러 agent와 메인 세션이 description 매칭으로 자동 활성화하는 도메인 지식

agent는 작업 진행 중 필요할 때 마크다운 링크 또는 `Read` tool로 lazy-load. Skills는 Claude Code의 description 매칭이 자동 활성화.

## Consequences

**긍정**:
- agent.md 줄수 절반 가까이 감소 (예: code-reviewer 250줄 → 109줄)
- 작은 작업에 큰 prompt 로드 방지 — 토큰·관련성 효율
- 도메인 지식 중복 제거 (예: security-review가 code-reviewer + tester 둘 다 활용)
- 새 agent 추가 시 슬림 agent.md + 폴더만 만들면 됨 — 변경 범위 작음

**부정**:
- 파일 수 증가 (3 agent → 3 agent + 11 sibling 파일 + skill 폴더)
- 동기화 책임 증가 (agent.md TOC와 실제 sibling 파일 일치 유지)
- markdown 상대 링크 lazy-load 동작은 Claude Code 런타임 의존 — 검증 사용자 액션 필요
- "어디 있는지" 메모리 부담 (해결: `docs/extend.md`에 "어디에 뭐 추가" 표 + 메모리 layout cheat sheet)

**후속 작업**:
- 첫 사용 후 lazy-load가 실제로 expect대로 동작하는지 확인 (agent가 sibling 파일을 진짜 read하는지)
- 동작 불완전하면 markdown 링크 → 명시적 "Read X when Y" 지시로 보강

## Alternatives considered

### A. 모든 지식을 agent.md에 인라인 (status quo, v0.1.0)
- **기각 사유**: 위 Context의 4가지 문제. 작은 작업에도 무거운 prompt.

### B. Skills만 사용 (per-agent 폴더 없음)
- **기각 사유**: agent 고유 mechanics(output 형식, severity 룰)는 cross-agent 공유 의미 없음. SKILL.md description 매칭 의도(여러 곳에서 자동 활성)와 안 맞음. 이런 지식까지 skill로 만들면 skill 카탈로그 노이즈.

### C. Per-agent 폴더만 사용 (Skills 없음)
- **기각 사유**: 보안 룰처럼 여러 agent + 메인 세션이 공유할 도메인 지식이 같은 텍스트로 N개 폴더에 중복. DRY 깨짐.

### D. 외부 markdown 링크 + 자동 import (CLAUDE.md `@import` 같은 기능)
- **기각 사유**: Claude Code의 import 기능 동작이 plugin 컨텍스트에서 보장 안 됨. 브라이트라인을 lazy-load(요청 시 read)로 두는 게 deterministic.

## Related

- 코드 위치:
  - `plugins/harness/agents/code-reviewer/{checklists,severity-tagging,output-format}.md`
  - `plugins/harness/agents/tester/{framework-detection,execute-verify-loop,diagnosis-patterns}.md`
  - `plugins/harness/agents/principle-auditor/{audit-checks,audit-output}.md`
  - `plugins/harness/skills/security-review/{SKILL,injection,secrets}.md`
- 메모리: `decisions.md` 항목 #2
- 문서: `docs/extend.md` "Progressive Disclosure" 섹션
- 출처:
  - [Skill Issue: Harness Engineering for Coding Agents | HumanLayer](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents)
  - Claude Code Skills documentation (description 매칭 메커니즘)
