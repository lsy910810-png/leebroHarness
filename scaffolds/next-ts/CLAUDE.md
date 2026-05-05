# Project: <name>

Next.js + TypeScript. Harness Engineering scaffold(next-ts). 4원칙은 항상 적용됩니다.

## Where to start

- (선택) [docs/map.md](docs/map.md) — 시스템 지도. 처음이면 먼저.
- (선택) [docs/architecture/](docs/architecture/) — forward-only layers + Providers 적용 시.
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

3. **종료 체크**: 마지막에 `✓ 원칙 체크` 블럭(1·2·3·4 각 한 줄, 검증 명령 + 결과 첨부).

짧은 응답이라도 앵커와 종료 체크는 항상 포함. 협상 불가.

## Project Meta

- **스택**: Next.js (App Router) + TypeScript
- **주요 디렉토리**: `app/`, `components/`, `lib/`
- **개발**: `npm run dev`
- **빌드**: `npm run build`
- **린트**: `npm run lint`
- **타입체크**: `npx tsc --noEmit`
- **테스트**: (jest/vitest 셋업 후 채우기)

## 컨벤션

- App Router. `app/` 안에 라우트 그룹과 `layout.tsx` 컨벤션.
- 비밀값은 `.env.local`(읽기 deny). 클라이언트 노출용은 `NEXT_PUBLIC_` 접두.
- 컴포넌트는 server component default. 인터랙션 필요 시에만 `"use client"`.

## Computational sensors (자동)

settings.json에 다음 hook이 박혀 있습니다:
- **PostToolUse(Edit/Write)**: TS 파일 변경 시 `tsc --noEmit` 자동 실행 → 타입 에러 즉시 알림 (exit 2로 차단)
- **Stop**: 세션 종료 전 `npm run lint` + `npm test` 실행 → 실패 시 차단

너무 느리면 settings.json `hooks` 섹션 조정.
