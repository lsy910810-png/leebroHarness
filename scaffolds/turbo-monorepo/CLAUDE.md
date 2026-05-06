# Project: <name>

Turbo monorepo (pnpm workspace). [Harness Engineering](https://github.com/lsy910810-png/leebroHarness) scaffold(turbo-monorepo). 4원칙은 항상 적용됩니다. 전문 원칙은 [PRINCIPLES.md](https://github.com/lsy910810-png/leebroHarness/blob/main/PRINCIPLES.md).

## Where to start

- (선택) [docs/map.md](docs/map.md) — 시스템 지도. 처음이면 먼저.
- 자주 쓰는 명령은 아래 "Project Meta" 참조.

## Operating Principles (always apply)

1. **Think Before Coding** — 가정 명시·해석 분기·간단한 길 제안·헷갈리면 멈추고 묻기.
   - 모를 때 추측 X, 묻기. 다중 해석 있으면 silent pick X. 더 쉬운 방법 있으면 push back.

2. **Simplicity First** — 요구된 만큼만. speculative 추상화·flexibility·불가능 시나리오 핸들링 금지.
   - 단일 호출 helper 안 만듦. 200줄을 50줄로 줄일 수 있으면 줄임. *Test: 시니어가 overcomplicated라 할 만한가?*

3. **Surgical Changes** — 요청에 직접 추적 안 되는 줄 안 건드림. 인접 코드 "improve" 금지. 본인이 만든 orphan만 청소.
   - 무관한 dead code 발견 시 보고만, 삭제 X. 기존 스타일 유지. *Test: 모든 변경 라인이 요청에 추적되나?*
   - **모노레포 주의**: 한 commit이 여러 app 건드릴 수 있음 — 작업 의도가 cross-app이면 OK, 아니면 분리 권고.

4. **Goal-Driven Execution** — 시작 전 "끝났다" 정의. execute-verify 루프. 증거(로그·테스트·출력) 제시.
   - "should work" 금지. 검증 못 했으면 명시적으로 보고. *Test: 제3자가 evidence로 완료 재현 가능한가?*

## Visibility Protocol (every response)

매 응답마다 다음 3단을 지킵니다:

1. **시작 앵커**: `▶ Operating under: 1·Think Before Coding · 2·Simplicity First · 3·Surgical Changes · 4·Goal-Driven Execution`

2. **결정 인라인 태그**: 비자명한 판단 시 `[원칙 N] <근거>` 즉시 표기.

3. **종료 체크**: 마지막에 `✓ 원칙 체크` 블럭(1·2·3·4 각 한 줄, 검증 명령 + 결과 첨부).

짧은 응답이라도 앵커와 종료 체크는 항상 포함. 협상 불가.

## Project Meta

### 스택
- **모노레포**: Turbo + pnpm workspace
- **Runtime**: Node (LTS 권장)
- **Apps**: `apps/<app-name>/` (각자 README + 자체 명령)
- **Packages**: `packages/<pkg-name>/` (공유 코드)

(채워주세요 — 각 app·package 한 줄씩)

### 자주 쓰는 명령

```bash
# 개발 (모든 app 동시)
pnpm dev

# 특정 app만
pnpm dev --filter <app-name>

# 빌드 (turbo 캐시)
pnpm build

# 테스트
pnpm test                          # 전체
pnpm test --filter <app-name>      # 특정 app

# 린트·포맷
pnpm lint
pnpm format

# 타입체크
pnpm typecheck                     # 또는 pnpm -r exec tsc --noEmit

# 변경된 워크스페이스만 (CI 효율)
pnpm test --filter=...[origin/main]
```

### 컨벤션
- **pnpm workspace** — `pnpm-workspace.yaml`로 packages·apps 구성
- **Turbo task graph** — `turbo.json`이 build/dev/test/lint dependency 정의
- **공유 config** — `packages/typescript-config`, `packages/eslint-config` 등 패키지로 분리
- **Branch**: `main` 기준, feature 브랜치는 `feat/...` 또는 `fix/...`

### 보안
- **`.env*`은 Read deny** (settings.json에서 차단)
- 비밀값은 `.env.local`. 클라이언트 노출용은 `NEXT_PUBLIC_` (Next.js) 등 프레임워크 컨벤션

## Hooks (현재 미적용 — 점진적 도입 예정)

monorepo는 typecheck/lint/test가 무거울 수 있어 hooks 신중히. 활성화 시 권고:
- PostToolUse: `turbo run typecheck --filter=...changed` (변경 워크스페이스만)
- Stop: `pnpm lint && pnpm test --filter=...changed` (전체보다 가벼움)

너무 느리거나 막히면 hook 자체 제거.
