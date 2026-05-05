# Severity Tagging Rules

발견 항목은 항상 `[severity · 원칙 N] <설명>` 형식으로 태깅합니다.

## Severity 4단계

### `blocker` — 머지하면 안 됨
- 보안 취약점 (SQL injection, secret 노출, auth 우회)
- 데이터 손실/손상 가능
- 프로덕션 다운 가능 (무한 루프, OOM, deadlock)
- 변경된 동작에 대한 **테스트 부재**(원칙 4)
- 명백한 회귀

### `major` — 머지 전 고치는 게 강력 권장
- 명확한 버그지만 일반 사용자 흐름 외에서만 발생
- 잘못된 에러 처리(swallow, 잘못된 메시지)
- 스코프 위반(원칙 3) — 무관한 변경이 섞여 있음
- 성능 저하 가능성 있는 패턴(N+1 등)이지만 즉시 영향 없음

### `minor` — 의견; 머지 후 후속 PR도 가능
- 명명/구조 개선
- 가독성 향상
- 더 안전한 패턴(이미 안전하지만 prepared statement 같은 정공법 권고)

### `nit` — 취향; 무시 가능
- 포맷팅, 사소한 일관성

## 원칙 번호 매핑

발견의 근거 원칙을 번호로 함께 적어 어떤 원칙이 위반/적용됐는지 즉시 보입니다.

- **원칙 1 (Think Before Coding)**: 추측에 의존, 가정 미명시, 불확실한 동작에 의문 안 던짐
- **원칙 2 (Simplicity First)**: 불필요한 추상화·옵션·error handling
- **원칙 3 (Surgical Changes)**: 스코프 위반, 인접 코드 "개선"
- **원칙 4 (Goal-Driven Execution)**: 테스트 부재, 검증 증거 부재, "should work" 식 보고

복수 원칙에 걸리면 가장 강한 위반 한 개로 태그하고 본문에 다른 원칙 언급.

## 예시

```
[blocker · 원칙 4] src/auth/login.ts:42 — 새 비밀번호 검증 로직에 테스트 없음
  • 기존 테스트가 path 변경만 다룸
  • 조치: login.spec.ts에 happy + 잘못된 비번 + 잠긴 계정 케이스 추가

[major · 원칙 3] src/auth/login.ts:15-30 — 로그인 작업에 무관한 logger 리팩토링 포함
  • 변경 라인이 두 작업으로 섞임
  • 조치: 분리해서 별도 PR로

[minor · 원칙 2] src/auth/utils.ts:5 — `validateInput(x, opts={strict:false})` — opts 단일 호출 지점
  • 호출자가 한 곳뿐이라 옵션 객체 불필요
  • 조치: `validateInput(x)`로 단순화 (선택)

[nit] src/auth/login.ts:88 — trailing whitespace
```

## 결과 요약

PR 마지막에 짧은 요약:
```
요약: blocker 1, major 1, minor 1, nit 1
머지 가능 여부: ❌ blocker 해결 후
```
