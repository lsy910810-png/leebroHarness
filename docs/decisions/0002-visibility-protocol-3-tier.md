# ADR 0002: Adopt 3-Tier Visibility Protocol for Operating Principles

- **Status**: Accepted
- **Date**: 2026-05-04
- **Supersedes**: —

## Context

사용자가 4 operating principles(Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution)를 정의하고 "에이전트가 동작할 때 항상 명시되어야 한다"고 명시적으로 요구.

처음 설계는 4원칙을 agent system prompt에 **임베드만** 했음. 그러나 사용자가 push back: 임베드는 LLM 내부 상태일 뿐, **사용자 화면(응답 출력)에는 안 보임**. 매 응답에서 원칙이 어떻게 적용됐는지 사용자가 검증할 수 없음.

핵심 요구: **agent가 동작할 때 4원칙 적용을 출력에서 visible하게**. 임베드만으론 부족, 출력 측에서도 명시적이어야 함.

## Decision

응답 출력에 **3단 visibility protocol** 강제:

1. **시작 앵커** (every response): 첫 줄에 무조건
   ```
   ▶ Operating under: 1·Think Before Coding · 2·Simplicity First · 3·Surgical Changes · 4·Goal-Driven Execution
   ```

2. **인라인 결정 태그**: 비자명한 판단 즉시 근거 원칙 표기
   ```
   [원칙 2] 단일 호출이라 helper 추출 안 함
   [원칙 1] 가정: 이 함수가 sync로 호출됨 — 확인 필요
   [원칙 3] 인접한 deprecated 코드 발견. 삭제 안 하고 보고만
   ```

3. **종료 체크 블럭**: 마지막에 4원칙 각각 한 줄로 정산
   ```
   ✓ 원칙 체크
   - 1: 가정/불확실성 — (구체 예 또는 "해당 없음")
   - 2: 추가 추상화/스코프 초과 — 0건 또는 N건 (사유)
   - 3: 변경 라인이 요청에 추적됨 — yes/no(사유)
   - 4: 검증 — (실행 명령 + 결과 또는 "검증 불가, 사유")
   ```

이 protocol은 **모든 agent와 scaffolded 프로젝트의 메인 세션**에 적용. 짧은 응답이라도 앵커 + 종료 체크는 필수. 협상 불가.

## Consequences

**긍정**:
- 매 응답에서 원칙 적용을 사용자가 즉시 검증 가능
- 임베드만 했을 때보다 LLM의 원칙 준수율 향상 (출력에 적어야 하므로 사고 강제)
- 종료 체크의 "4: 검증" 항목이 "should work" 식 보고를 차단 (원칙 4 자동 강제)
- 인라인 태그가 결정 근거를 추적 가능하게 함 (post-mortem 시 유용)

**부정**:
- 응답이 verbose해짐 (앵커 1줄 + 종료 체크 4~6줄 = 매 응답 5~7줄 오버헤드)
- 짧은 답변(예: "yes" 한 단어로 끝낼 수 있는 질문)에도 protocol 적용 — 비례적 무거움
- "원칙 체크" 항목 작성 자체에 토큰 추가 소비

**후속 작업**:
- 사용 후 verbose함이 비례 안 되면 Light variant로 전환(앵커 + 종료 체크만, 인라인 태그 생략) — 사용자 합의로
- 짧은 응답 전용 short-form 도입 검토는 보류 (Simplicity First — 먼저 사용해보고)

## Alternatives considered

### A. 임베드만 (status quo, 처음 설계)
- **기각 사유**: 사용자 화면에 원칙 적용 안 보임. 검증 불가.

### B. 앵커만 (Light)
- 첫 줄 한 줄만, 나머지 자유.
- **기각 사유**: 의례적인 헤더로 전락 위험. 본문에 실제 적용 안 보임.

### C. 종료 체크만
- 응답 끝에 4원칙 자체 점검 블럭.
- **기각 사유**: 작업 중 결정 근거가 본문에 안 보임. 끝에서야 회고 — late.

### D. Heavy (3단 + 매 도구 호출 전 애노테이션)
- 매 Bash/Edit/Read 전에 "applying [원칙 N]" 한 줄.
- **기각 사유**: 토큰 부담 큼, 주의 분산. 가치 대비 무거움.

### E. 3단 (선택됨)
- 앵커 + 인라인 태그 + 종료 체크.
- **선택 사유**: 시작·중간·끝 모두 cover. 검증 가능 + 결정 근거 보임 + 회고 강제. 사용자가 명시적 합의.

## Related

- 코드 위치 (3단 protocol 인라인 5곳):
  - `plugins/harness/agents/code-reviewer.md` (full)
  - `plugins/harness/agents/tester.md` (full)
  - `plugins/harness/agents/principle-auditor.md` (full)
  - `scaffolds/general/CLAUDE.md` (short form)
  - `scaffolds/next-ts/CLAUDE.md` (short form)
- 메모리: `decisions.md` 항목 #3, `feedback_principles.md`
- 문서: `PRINCIPLES.md` "Visibility Protocol" 섹션
- 출처: 사용자가 이 세션에서 직접 정의. ETH Zurich agent file 연구 (간접 영향: 사용자 명시적 출력이 LLM 준수율 높임).
