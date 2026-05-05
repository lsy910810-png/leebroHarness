# Review Output Format

code-reviewer agent의 응답 템플릿. Visibility Protocol 3단(앵커/태그/체크)은 이미 항상 적용되며, 본 템플릿은 그 안의 본문 형식입니다.

## 전체 형식

```
▶ Operating under: 1·Think Before Coding · 2·Simplicity First · 3·Surgical Changes · 4·Goal-Driven Execution

## Scope
- 검토 대상: <파일/diff 범위, 변경 line 수>
- 컨텍스트: <PR 설명·issue·커밋 메시지에서 파악한 의도>

## Findings

### Blocker
- [blocker · 원칙 N] `<file>:<line>` — <문제 한 줄>
  • <상세 설명·재현 시나리오>
  • 조치: <구체 수정안>

### Major
- [major · 원칙 N] `<file>:<line>` — ...

### Minor
- [minor · 원칙 N] `<file>:<line>` — ...

### Nit
- [nit] `<file>:<line>` — ...

## Out of scope (mentioned, not actioned)
- <발견했지만 본 PR과 무관> — 후속 권고

## 요약
- blocker N, major N, minor N, nit N
- 머지 가능 여부: ✅ / ⚠️ <조건> / ❌ <blocker 사유>

✓ 원칙 체크
- 1: 가정/불확실성 — <검토 중 명확하지 않아 보고한 항목, 또는 "해당 없음">
- 2: 추가 추상화/스코프 초과 — 0건 (검토만 수행, 코드 수정 없음)
- 3: 변경 라인이 요청에 직접 추적됨 — yes/no(스코프 위반 발견 N건 보고)
- 4: 검증 — 검토 차원 N개 점검 완료, 결과는 위 Findings에 evidence와 함께
```

## 규칙

- **빈 섹션은 생략하거나 "없음"으로 명시** — 빈 헤더 두지 않음
- **모든 finding은 file:line 명시** — vague 한 위치 금지
- **모든 finding은 조치 포함** — "이거 안 좋다"만 안 됨
- **자체 코드 수정 안 함** — 검토자 역할(원칙 3 + role)
- **모르는 것은 모른다고** — "이 함수가 어디서 호출되는지 grep으로 못 찾음 → 의도 확인 필요" (원칙 1)

## 빠른 PR(파일 1~2개) vs 큰 PR

- 빠른 PR: 위 템플릿 그대로
- 큰 PR (10+ 파일): Findings를 파일별로 그룹핑 가능
  ```
  ## Findings by file

  ### src/auth/login.ts
  - [blocker · 원칙 4] line 42 — ...

  ### src/auth/middleware.ts
  - [major · 원칙 2] line 15 — ...
  ```

## "머지 가능 여부" 판단

- ✅: blocker 0개 + major 0~1개 (major도 명확히 후속 PR 가능한 경우만)
- ⚠️: blocker 0개 but major 2+개 → 조건부
- ❌: blocker 1개 이상

이 판단은 항상 명시 — "looks good" 류 vague 평가 금지.
