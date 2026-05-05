# Code Review Checklists

차원별 점검 항목. 검토 시 해당 차원에 발견되는 게 있으면 보고. 없으면 종료 체크에 "해당 없음"으로 명시.

## 1. 명명·가독성

- 식별자가 **무엇을 하는지가 아니라 무엇인지**를 드러내는가? (`getUserData` ❌ → `fetchActiveUserProfile` 또는 그냥 `user` ✓)
- bool은 `is/has/can/should` 접두 또는 명백한 명사형
- 약어 남발 안 함 (`usr`, `cfg`, `mgr` ❌)
- 함수 길이 **30줄 이상**이면 분해 가능한지 검토
- 중첩 depth **3단계 이상**이면 early return으로 평탄화
- 매직 넘버는 const로

## 2. 에러 처리 (원칙 2: Simplicity First)

- **실제 발생 가능 경로만** try/catch
- "혹시 모르니" 식 catch-all 안 둠
- catch에서 swallow(빈 catch, console.log만) 금지 — 로그 + 재throw 또는 명시적 처리
- 사용자에게 가는 에러 메시지가 내부 상세(stack trace, SQL) 노출 안 함
- async 함수에서 unhandled rejection 검토 (`.catch()` 없는 promise)

## 3. 보안 (보안 의심 시 `/harness:security-review` skill 사용)

- 입력 신뢰 경계 명확
- 출력 인코딩 (DB·HTML·shell 컨텍스트별)
- 인증/인가 체크 위치 (middleware? 각 endpoint?)
- secrets 코드 안에 없음

상세는 security-review skill로 위임.

## 4. 테스트 커버리지 (원칙 4: Goal-Driven Execution)

- 변경된 동작을 검증하는 테스트가 같이 들어왔는가?
- happy path + edge case 1~2개 있는가?
- 테스트가 **실제 실행되어 통과**한 증거가 PR에 있는가?
- 변경된 코드 라인 중 테스트로 커버 안 된 분기가 명백히 보이는가?

테스트가 없으면 `[blocker · 원칙 4]`로 태그.

## 5. 스코프 규율 (원칙 3: Surgical Changes)

- diff의 모든 변경 라인이 **사용자 요청에 직접 추적**되는가?
- 인접한 코드의 "improvement"가 끼어 있지는 않은가? (포맷팅·rename·refactor)
- 무관한 dead code 삭제가 있는가? → 보고만 하고 revert 권고
- import 정리·주석 수정 등이 본 작업과 분리됐는가?

스코프 위반 발견 시 `[major · 원칙 3]`로 태그.

## 6. 동시성·상태 (해당 시)

- 공유 상태(전역 변수, 모듈 캐시) 변경이 있는가?
- async 작업 간 race condition 가능성?
- DB 트랜잭션 경계가 적절한가? (부분 commit 시 일관성 깨짐?)

## 7. 성능 (해당 시)

- N+1 query 패턴
- 큰 collection을 메모리에 다 올리는 패턴 (`fetchAll().filter()` 대신 query에서 필터링)
- 불필요한 re-render (React)

## Out-of-scope 처리

검토 중 발견했지만 본 PR과 무관한 이슈:
- **파일에 별도 섹션**(`## Out of scope (mentioned, not actioned)`)으로 보고
- 삭제·수정 시도 안 함 (원칙 3)
- 후속 작업 권고만
