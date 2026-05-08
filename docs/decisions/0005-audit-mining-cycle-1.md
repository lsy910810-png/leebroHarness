# ADR 0005: Audit Log Mining Cycle 1 — KB Lookup 실패 + ID/Severity 표준화

- **Status**: Accepted
- **Date**: 2026-05-08
- **Supersedes**: 0001 (부분적 — principle-auditor 한정)

## Context

v0.3.1 릴리스 후 실제 프로젝트 2곳(Symphony, mini_api)에서 `principle-auditor`가 5회 invoke됨. 각 호출의 input/output이 jsonl session log에 보존되어 있었음. 본 cycle은 그 데이터를 mining해 evidence-based로 하네스를 갱신하는 첫 시도.

mining 추출본: `~/scratch/audit-mining-extracts.md` (142KB, 5 audit prompt + final punch list).

## 발견 (cross-validated, 5/5 audits)

### Finding 1 — KB Lookup 100% 실패 (universal)

5개 audit 모두 동일 패턴:

1. agent.md의 "Knowledge Base" 섹션을 read.
2. Sibling 파일 경로 `principle-auditor/audit-checks.md`, `principle-auditor/audit-output.md`를 cwd-relative로 해석.
3. `<project-cwd>/principle-auditor/audit-checks.md`를 Read 시도 → 못 찾음.
4. "Knowledge base files are not present" / "Knowledge Base 파일이 없습니다" 출력 후 폴백.
5. 자체 추론으로 4 dimension·severity·output format 재구성.

**근본 원인**: Claude Code 1.0.x는 subagent context에 plugin root path를 노출하지 않음. `${CLAUDE_PLUGIN_DIR}` 같은 substitution 토큰 없음. 실제 KB 파일은 `~/.claude/plugins/cache/leebroharness/harness/0.3.1/agents/principle-auditor/`에 존재하지만 agent가 그 경로를 알 수 없음.

**Cross-validation**: 5/5 audits = 100%. Symphony 4회 + mini_api 1회 모두 동일.

**관찰된 결과**: KB 없이도 모든 audit이 thorough함 — 16~19 findings, 4 차원 모두 커버, file:line 정확. 즉 KB 부재가 audit 품질에 측정 가능한 영향 없었음. agent의 자체 추론으로 충분했음.

### Finding 2 — ID/Severity Vocabulary 자유 변동

5개 audit에서 ID 포맷 + severity vocabulary가 매번 다름:

| Audit | ID 포맷 | Severity vocab |
|---|---|---|
| Symphony-1 | `H1`, `M3`, `L2` | HIGH/MEDIUM/LOW |
| Symphony-2 | `D1-1`, `D2-1`, `D3-3` | High/Medium/Low/Info |
| Symphony-3 | `D1-1`, `D2-1` | HIGH/MEDIUM/LOW/INFO |
| Symphony-4 | `P1-A`, `P2-B`, `P3-D` | Medium/Low |
| mini_api-1 | `D1-01`, `D2-02` | HIGH/MEDIUM/LOW |

KB의 `audit-output.md`가 `blocker/major/minor/nit` 4-tier vocabulary를 정의하지만 — 위 Finding 1으로 인해 한 번도 적용 안 됨. ID 포맷은 KB에서도 정의 안 했음(공백).

**영향**: 같은 프로젝트 두 audit 결과를 사람이 비교하기 어려움 (`H1` vs `D1-1` 매칭 불가). 자동화 도구가 audit 결과 파싱하기도 어려움.

## Decision

### 1. Progressive Disclosure 폐기 (principle-auditor 한정)

ADR 0001은 모든 agent에 progressive disclosure를 채택했지만, 본 cycle 결과 **principle-auditor 한정으로 인라인 임베드 채택**.

- agent.md에 4 차원 임계값 + ID·severity 표준 + 응답 템플릿 인라인.
- KB 파일 2개(`audit-checks.md`, `audit-output.md`) + 폴더 삭제.
- agent.md 줄 수: 92 → 173 (~80줄 증가). 여전히 manageable.

ADR 0001은 **부분적으로 supersede**. code-reviewer / tester는 보존 — 이들의 audit 로그가 아직 모이지 않아 동일 문제인지 cross-validate 불가능. 향후 cycle 2에서 동일 mining 후 결정.

### 2. ID·Severity 표준화 명시

agent.md에 강제:
- ID: `D<dim>-<index>` (예: `D1-1`, `D2-3`). 자유 vocabulary 금지.
- Severity: `blocker/major/minor/nit` 4단. `HIGH/MEDIUM/LOW` 등 사용 금지.

### 3. 임계값 명시 (이전 KB에서 옮김)

- D1 (긴 함수/파일): 파일 500+ 줄, 함수 100+ 줄 (테스트 2배)
- D2 (스코프 완화): commit 4+ 디렉토리 + 5+ 파일 동시 변경
- D3 (검증 부재): src 50+ 줄에 test 0줄 (또는 200+에 < 10줄)
- D4 (vague 코멘트): 누적 5+

## Consequences

**긍정**:
- KB lookup 실패 100%가 fix됨 (agent.md 직접 읽음)
- ID·severity 비교 가능해짐 → 자동화/이전 audit 비교 가능
- 임계값 명시 → false positive 줄어들 가능성

**부정**:
- agent.md ~80줄 증가 → ADR 0001의 "슬림 agent.md" 원칙 부분적 후퇴
- code-reviewer / tester는 동일 문제일 가능성 높지만 본 cycle에선 보존 — 일관성 일시 깨짐 (code-reviewer는 KB lookup, principle-auditor는 inline)

**후속**:
- code-reviewer / tester 자체에 대한 audit 로그 mining (cycle 2) → 동일 문제 확인 시 동일 fix
- ADR 0001 status를 "Partially Superseded by 0005" 또는 "Active for code-reviewer/tester only"로 갱신 (별 작업)
- principle-auditor 재 audit 검증 (Phase 4) — Symphony / mini_api에서 새 출력 형식 확인

## Verification

Phase 4에서 Symphony와 mini_api에 다시 `/audit` 돌려:
- 출력에 `D1-1` 같은 표준 ID가 사용되는지
- severity가 `blocker/major/minor/nit` vocabulary로 통일되는지
- "Knowledge base files are not present" 폴백 메시지가 더 이상 안 나오는지 (agent.md 직접 read 후 inline content로 진행)

## Related

- mining 산출물: `~/scratch/audit-mining-extracts.md`
- 변경 파일:
  - `plugins/harness/agents/principle-auditor.md` (수정)
  - `plugins/harness/agents/principle-auditor/audit-checks.md` (삭제)
  - `plugins/harness/agents/principle-auditor/audit-output.md` (삭제)
  - `plugins/harness/agents/principle-auditor/` (폴더 삭제)
  - `.claude-plugin/marketplace.json` (0.3.1 → 0.3.2)
  - `plugins/harness/.claude-plugin/plugin.json` (0.3.1 → 0.3.2)
- Source data: 5 jsonl files in `~/.claude/projects/C--Users-august-Desktop-{Symphony,mini-api}/`
- 메모리: `decisions.md` 항목 추가 예정 ("Cycle 1 mining: KB lookup universal failure → inline embed")
- 출처:
  - claude-code-guide agent 응답 (plugin path env 부재 확인)
  - Claude Code Skills doc (skill만 ${CLAUDE_SKILL_DIR} 지원)
