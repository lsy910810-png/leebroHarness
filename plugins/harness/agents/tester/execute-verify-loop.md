# Execute-Verify Loop (원칙 4 상세)

tester agent의 핵심 워크플로우. "테스트를 짰다"가 아니라 "테스트를 짜고 실제로 실행해 통과를 확인했다"가 완료 기준.

## Loop 단계

```
        ┌──────────────────────────────┐
        ▼                              │
  1. Define success                    │
        │                              │
        ▼                              │
  2. Detect framework  → 모호하면 ask  │
        │                              │
        ▼                              │
  3. Write minimal test                │
        │                              │
        ▼                              │
  4. Execute (실제 실행 — 추측 금지)   │
        │                              │
        ▼                              │
  5. Verify result                     │
        │                              │
   pass?─┴─fail?                       │
    │      │                           │
    │      ▼                           │
    │  6. Diagnose (root cause)        │
    │      │                           │
    │      ▼                           │
    │  7. Fix (test? code? infra?) ───┘
    │
    ▼
  8. Re-verify against Simplicity
        │
        ▼
  9. Report with evidence
```

## 1. Define success
한 줄로. 관찰 가능한 형태:
- ❌ "auth가 잘 동작한다"
- ✅ "POST /login은 잘못된 비밀번호 시 401 반환, body에 `{ error: 'invalid_credentials' }` 포함"

## 3. Write minimal test
- happy path 1개 + edge 1~2개. 끝.
- 같은 동작을 다양한 시나리오로 N번 검증 ❌ (원칙 2)
- 미리 만들어둔 helper·fixture 재사용 ✅

## 4. Execute
- 실제 명령 실행. Bash tool로.
- 출력을 응답에 첨부 (잘라서라도)
- 명령이 hang되면 timeout 명시하고 그 사유 보고

## 5. Verify
- 통과: pass count + 새 테스트가 실제로 돌았는지(grep으로 확인 가능)
- 실패: error 메시지·stack·실패 line 캡처

## 6. Diagnose
실패 원인을 분류:

| 분류 | 증상 | 다음 단계 |
|---|---|---|
| **테스트 잘못** | assertion이 실제 동작과 다름 | 테스트 수정 (구현 의도 재확인) |
| **코드 버그** | 구현이 명세와 다름 | 코드 수정 (원칙 3: 최소 변경) |
| **인프라** | DB 연결, 모듈 못 찾음, env 변수 누락 | 설정 보강 또는 사용자 확인 |
| **flaky** | 시간·순서 의존 | 격리·mock·retry 도입 (별 PR 권장) |

## 7. Fix
- 한 번에 한 가설만. 동시 변경 금지.
- 4로 돌아가 재실행.
- Loop 횟수 추적 — 4회 이상 실패면 멈추고 보고 (원칙 1: confused면 stop)

## 8. Re-verify against Simplicity (원칙 2 → 4)
goal 통과 후:
- 추가한 테스트 코드에 불필요한 추상화 없는지
- helper 함수 만든 게 단일 호출이면 인라인으로 되돌리기
- mock 너무 많이 쓰지 않았는지 (실제 코드 경로 못 검증할 수 있음)

## 9. Report with evidence
응답 종료 체크 4번에 다음 형식으로:

```
- 4: 검증 — `npx vitest run src/auth/login.spec.ts`
  ```
  ✓ src/auth/login.spec.ts (3)
    ✓ returns 401 for invalid password
    ✓ returns 200 for valid credentials
    ✓ locks account after 5 failures

  Test Files  1 passed (1)
       Tests  3 passed (3)
  ```
  Loop: 1회 (한 번에 통과). Simplicity 재점검: helper 미추가, mock 1개(DB).
```

## 금지

- "should pass" 식 보고 (실행 안 했다는 뜻)
- 출력 첨부 없이 "tests pass"만 말하기
- 실패 진단 없이 "이상하게 안 됨" 보고
- 4회 넘게 같은 가설로 재시도 — 멈추고 사용자에게 (원칙 1)
