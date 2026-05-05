# Diagnosis Patterns

테스트 실패 시 흔한 패턴과 진단·해결 가이드.

## 1. "Cannot find module" / Import error

### 증상
```
Cannot find module 'src/utils' from 'src/foo.test.ts'
```

### 원인 후보
- path alias(`@/`, `~/`) 설정 누락 (`tsconfig.json` paths vs jest/vitest config 불일치)
- 새로 추가한 파일이 build 캐시 밖에 있음
- 상대 경로 vs 절대 경로 혼재

### 진단
```bash
# alias 매핑 확인
grep -r "moduleNameMapper\|alias" jest.config.* vitest.config.* tsconfig.json
```

### 해결
- jest: `moduleNameMapper` 또는 `pathsToModuleNameMapper`
- vitest: `resolve.alias`
- 둘 다 안 되면 상대 경로로 일단 통과 → 별 PR로 alias 정합

## 2. "ReferenceError: X is not defined"

### 원인 후보
- globals 누락 (`describe`, `test`, `expect`)
- jsdom vs node environment 불일치
- TypeScript strict null check

### 해결
- jest: `setupFiles` 또는 `globals: true`
- vitest: `vitest.config.ts`의 `test.globals: true`
- DOM API 필요: `test.environment: 'jsdom'`

## 3. Async test가 끝나기 전에 종료

### 증상
- assertion이 실행 안 되는 것 같음
- "0 assertions" 표시

### 원인 후보
- `async` 함수에서 `await` 누락
- promise를 return 안 함
- callback 기반 인데 `done` 안 호출

### 해결
```js
// BAD
test('foo', () => {
  fetch('/api').then(r => expect(r.ok).toBe(true))
})

// GOOD
test('foo', async () => {
  const r = await fetch('/api')
  expect(r.ok).toBe(true)
})
```

## 4. "Timeout exceeded" / Hung test

### 원인 후보
- 외부 의존성(DB, network) 실제 호출
- promise가 resolve 안 됨
- infinite loop / await on never-resolving

### 진단
- 테스트 격리 (단일 테스트 실행: `vitest run -t "test name"`)
- 외부 호출 grep
- timeout 늘려서 진짜 hang인지 느린지 구분

### 해결
- 외부 의존성 mock
- timeout 명시 (`test('x', { timeout: 10000 }, ...)`)
- 진짜 무한 루프면 그건 코드 버그 → 6번 분류로

## 5. Snapshot mismatch

### 원인 후보
- 의도한 변경 (snapshot 갱신 필요)
- 의도치 않은 회귀 (코드 버그)

### 진단
diff를 사람이 읽음. 의도한 변경이면:
```bash
npx vitest run -u   # snapshot 갱신
```
의도치 않으면 코드 수정.

**금지**: 검토 없이 `-u`로 무조건 갱신 (원칙 4 — 검증 회피)

## 6. Flaky tests (가끔 실패)

### 원인 후보
- 시간 의존 (`Date.now`, timeout race)
- 순서 의존 (이전 테스트의 부작용)
- 외부 상태 (실제 DB·파일시스템)

### 진단
- 같은 테스트를 N번 반복 실행 (`vitest run --repeat 10`)
- `--isolate` 또는 단일 실행

### 해결
- fake timers (`vi.useFakeTimers()`)
- `beforeEach`에서 명시적 reset
- 외부 의존성 mock

flaky는 별 PR로 격리하는 게 원칙(원칙 3 — 본 작업 스코프 밖).

## 7. "Unhandled promise rejection"

### 원인 후보
- `.catch()` 없는 async 호출
- test에서 throw하는 부분이 await되지 않음

### 해결
```js
await expect(badCall()).rejects.toThrow('specific message')
// 또는
await expect(badCall()).rejects.toMatchObject({ code: 'INVALID' })
```

## 8. Type error in test only

### 원인 후보
- mock의 타입이 실제와 안 맞음
- test용 helper의 타입 누락

### 해결
- `vi.mocked()` 또는 `jest.mocked()`로 typed mock
- 의도적이면 `// @ts-expect-error` + 코멘트

## 진단 보고 형식

```
## Diagnosis
- 분류: <테스트 잘못 / 코드 버그 / 인프라 / flaky>
- 가설: <한 줄>
- 증거: <어떤 출력·grep 결과로 그 가설인지>
- 시도할 fix: <한 가지>
- 결과: pass / fail (다음 가설은...)
```

## 멈추는 시점 (원칙 1)

다음 중 하나면 fix 시도 중단하고 사용자에게:
- 같은 가설로 4회 이상 시도
- 진단이 분류 안 됨 (어디서 실패하는지 불명)
- 인프라 문제로 보이는데 권한·접근 없음
