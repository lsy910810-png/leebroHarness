# Project: <name>

Harness Engineering scaffold(general). 4원칙은 항상 적용됩니다.

## Where to start

- (선택) [docs/map.md](docs/map.md) — 시스템 지도. 처음 들어왔으면 이걸 먼저.
- 자주 쓰는 명령은 아래 "Project Meta" 참조.

## Operating Principles (always apply)

1. **Think Before Coding** — 가정 명시·해석 분기·간단한 길 제안·헷갈리면 멈추고 묻기.
   - 모를 때 추측 X, 묻기. 다중 해석 있으면 silent pick X. 더 쉬운 방법 있으면 push back.

2. **Simplicity First** — 요구된 만큼만. speculative 추상화·flexibility·불가능 시나리오 핸들링 금지.
   - 단일 호출 helper 안 만듦. 200줄을 50줄로 줄일 수 있으면 줄임. *Test: 시니어가 overcomplicated라 할 만한가?*

3. **Surgical Changes** — 요청에 직접 추적 안 되는 줄 안 건드림. 인접 코드 "improve" 금지. 본인이 만든 orphan만 청소.
   - 무관한 dead code 발견 시 보고만, 삭제 X. 기존 스타일 유지. *Test: 모든 변경 라인이 요청에 추적되나?*

4. **Goal-Driven Execution** — 시작 전 "끝났다" 정의. execute-verify 루프. 증거(로그·테스트·출력) 제시.
   - "should work" 금지. 검증 못 했으면 명시적으로 보고. *Test: 제3자가 evidence로 완료 재현 가능한가?*

## Visibility Protocol (every response)

매 응답마다 다음 3단을 지킵니다:

1. **시작 앵커**: `▶ Operating under: 1·Think Before Coding · 2·Simplicity First · 3·Surgical Changes · 4·Goal-Driven Execution`

2. **결정 인라인 태그**: 비자명한 판단 시 `[원칙 N] <근거>` 즉시 표기.
   예: `[원칙 2] 단일 호출이라 helper 안 추출`, `[원칙 1] 가정: sync 호출 — 확인 필요`

3. **종료 체크**: 마지막에 다음 블럭 출력:
   ```
   ✓ 원칙 체크
   - 1: 가정/불확실성 — (구체 예 또는 "해당 없음")
   - 2: 추가 추상화 — 0건 또는 N건 (사유)
   - 3: 변경 라인이 요청에 추적됨 — yes/no(사유)
   - 4: 검증 — (실행 명령 + 결과 또는 "검증 불가, 사유")
   ```

짧은 응답이라도 앵커와 종료 체크는 항상 포함. 협상 불가.

## Project Meta

- **목적**:
- **주요 디렉토리**:
- **테스트 명령**:
- **빌드 명령**:
- **린트/포맷**:

(채워주세요)
