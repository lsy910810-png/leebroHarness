# Architecture Guide (opt-in)

OpenAI harness engineering의 핵심 architectural 패턴을 Claude Code 환경에 맞춰 정리. **강제 아님** — 프로젝트가 채택하면 agent가 코드베이스를 더 잘 추론합니다(repository-first context design).

> 출처 종합: OpenAI "Harness engineering" (2026), Martin Fowler "Harness engineering for coding agents", HumanLayer skill issue post.

---

## 핵심 아이디어

**Agent = Model + Harness**. 모델은 멈춰있어도 harness만 잘 짜면 결과가 좋아집니다. harness의 architectural 측면이 다음 셋:

1. **Forward-only layers** — 의존성을 한 방향으로만 흐르게
2. **Providers pattern** — 횡단 관심사를 단일 인터페이스로 격리
3. **Repository-first context** — repo 자체가 agent에게 legible하도록 구성

---

## 1. Forward-only Layers

OpenAI 사례: `Types → Config → Repo → Service → Runtime → UI`

각 레이어는 자기보다 "앞"의 레이어에만 의존. 거꾸로는 X. 이렇게 하면:
- agent가 변경 영향 범위를 추론하기 쉬움
- 순환 의존이 구조적으로 불가능
- 새 기능을 어느 레이어에 넣을지 명확

### 예: Next.js 프로젝트에 적용

```
types/         ← 도메인 모델, zod 스키마, 순수 타입
config/        ← 환경 변수, 상수, feature flag (env로부터 읽고 type을 import)
repo/          ← DB·외부 API 접근. config + types만 의존
service/       ← 비즈니스 로직. repo + types 의존
runtime/       ← 서버 액션, route handlers, server components
              ← service + repo + types 의존 (config는 직접 X — service 거쳐서)
app/, components/  ← UI. server side는 runtime, client side는 service의 client-safe 부분만
```

### 강제 방법 (computational sensor)

ESLint `import/no-restricted-paths` 또는 `eslint-plugin-boundaries`:
```js
// eslint.config.js
{
  rules: {
    "boundaries/element-types": ["error", {
      default: "disallow",
      rules: [
        { from: "config", allow: ["types"] },
        { from: "repo", allow: ["types", "config"] },
        { from: "service", allow: ["types", "repo"] },
        { from: "runtime", allow: ["types", "config", "repo", "service"] },
        { from: "app", allow: ["types", "service", "runtime"] }
      ]
    }]
  }
}
```

→ 위반 시 lint fail. agent가 즉시 알아챔(shift-left).

---

## 2. Providers Pattern

횡단 관심사(auth, telemetry, feature flags, logger, DB connection)는 **하나의 명시적 인터페이스(Provider)**를 통해서만 코드에 들어옵니다. 모든 곳에서 직접 import 안 함.

### 좋지 않은 예
```ts
// 여러 파일에서 각자 다른 방식으로 logger 가져옴
import { console } from 'console'
import logger from '@/lib/logger'
import { log } from 'pino'
```

### Providers 적용
```ts
// types/providers.ts — 단일 인터페이스 정의
export interface Logger { info(msg: string, ctx?: object): void; error(...): void }
export interface AuthProvider { currentUser(): Promise<User | null> }
export interface FeatureFlags { isEnabled(name: string): boolean }

// runtime/providers.ts — 구체 구현 한 곳에 모음
export const logger: Logger = pino({ ... })
export const auth: AuthProvider = clerkAuth()
export const flags: FeatureFlags = launchDarkly()

// 다른 코드는 항상 runtime/providers에서 import
import { logger, auth } from '@/runtime/providers'
```

### 효과
- agent가 "logger 어디서 오지?" 헷갈릴 일 없음
- 테스트에서 mock 한 곳만 바꾸면 됨
- 새 implementation 갈아끼우기 = providers.ts 한 줄 변경

---

## 3. Repository-First Context Design

repo 자체가 agent에게 legible — 처음 들어온 agent가 README + `docs/` 만 보고 도메인을 파악할 수 있어야 합니다.

### 권장 docs 구조

```
docs/
├── map.md                   ← 1-2 페이지짜리 시스템 지도. 모든 새 세션이 먼저 읽음
├── architecture/
│   ├── layers.md            ← forward-only layers 정의·예외
│   └── providers.md         ← provider 인터페이스 카탈로그
├── domain/                  ← 비즈니스 도메인 글로서리
│   ├── entities.md          ← User, Order, ... 도메인 객체와 그들의 관계
│   └── workflows.md         ← 핵심 시나리오 (signup, checkout, refund...)
├── ops/
│   ├── deployment.md
│   ├── observability.md     ← 로그·메트릭·트레이싱 위치
│   └── runbooks/            ← 장애 대응 절차
└── decisions/               ← ADR (Architecture Decision Records)
    └── 0001-...md
```

### 강제 방법 (cross-link 검증)

```bash
# docs 안 마크다운 링크가 끊어진 곳 찾기
grep -rEo '\]\([^)]+\.md\)' docs/ | while IFS=: read -r src link; do
  rel=$(echo "$link" | sed -E 's/.*\(([^)]+)\).*/\1/')
  target=$(dirname "$src")/"$rel"
  [ -f "$target" ] || echo "broken link in $src: $rel"
done
```

CI에 넣어서 끊어진 링크 = build fail.

### CLAUDE.md와의 관계

`CLAUDE.md`는 짧은 진입점:
```markdown
# Project: <name>

## Where to start
- 시스템 지도: [docs/map.md](docs/map.md)
- Architecture: [docs/architecture/layers.md](docs/architecture/layers.md)
- 도메인 모델: [docs/domain/entities.md](docs/domain/entities.md)

## Operating Principles
[4원칙 + visibility protocol]

## 자주 쓰는 명령
- 개발: `npm run dev`
- 테스트: `npm test`
- typecheck: `npx tsc --noEmit`
```

---

## 도입 순서 권장

이 가이드의 모든 걸 한 번에 도입하면 무겁습니다. 단계적으로:

1. **Day 1**: `docs/map.md` 1페이지 — 시스템 개요 + 주요 디렉토리 + 핵심 시나리오 1~2개
2. **Week 1**: forward-only layers 컨벤션 정함 + ESLint boundaries 룰 추가
3. **Week 2**: provider 인터페이스 1~2개부터(logger, auth) 표준화
4. **Month 1**: `docs/architecture/`, `docs/domain/` 채우기
5. **Continuous**: ADR 누적

---

## 안티패턴 (피할 것)

- **레이어 무시한 import** — 빠르다고 service에서 UI로 직접 import → ESLint로 잡음
- **Provider 우회** — `console.log` 직접 호출 → ESLint `no-console` + 룰
- **docs와 코드의 drift** — docs 갱신 안 하면 cross-link 깨짐 → CI 검증
- **AGENTS.md/CLAUDE.md 자동 생성** — 사람이 손으로 짧게 (60줄 이하 권장). HumanLayer 연구: 자동 생성 파일은 성능을 오히려 떨어뜨림.
- **모든 횡단 관심사를 다 Provider화** — 한 번 쓰는 거에 인터페이스 만들지 않기 (원칙 2)

---

## 우리 harness와의 관계

이 가이드는 **scaffold + 사용자 프로젝트** 단에 적용. 하네스 자체(`plugins/harness/`)는 plugin 구조라 불필요.

- A. Computational sensor (typecheck/lint/test hook)는 [scaffolds/next-ts/.claude/settings.json](../scaffolds/next-ts/.claude/settings.json)에 박혀 있음 — forward-only를 강제하려면 ESLint boundaries 룰을 프로젝트가 추가
- B. principle-auditor agent는 차원 1·2가 본 가이드의 "긴 파일·스코프 완화"와 동일
- D. 슬림 CLAUDE.md는 본 가이드의 "60줄 이하" 권고와 통함
