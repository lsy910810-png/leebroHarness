# Troubleshooting

실제 install·운영 중 만난 에러들 + 해결책. 새 에러 발견 시 여기 누적.

---

## 1. `Plugin "harness" not found in any marketplace`

### 증상
```
/plugin install harness
→ Plugin "harness" not found in any marketplace
```

`/plugin marketplace add` 자체는 성공했는데 install이 실패.

### 원인
`marketplace.json` schema 누락. 초기 작성 시 minimum 예시(plugins 배열만)로 만들었는데, Claude Code 실제 스키마는 다음을 **필수**로 요구:
- 최상위 `name` 필드 (marketplace 식별자, kebab-case)
- 최상위 `owner` 객체 (최소 `name` 필드)
- plugin entry에서 `path` 대신 `source` 키

### Fix

```json
{
  "name": "leebroharness",
  "owner": {
    "name": "leebroHarness"
  },
  "plugins": [
    {
      "name": "harness",
      "source": "./plugins/harness",
      "description": "...",
      "version": "0.3.0"
    }
  ]
}
```

### 검증
```
/plugin install harness@leebroharness
```
`@<marketplace-name>` 접미사 — marketplace.json의 최상위 `name`과 일치해야 함.

### 참고
v0.2.0 → v0.2.1 사이에 발견·수정됨 ([commit 77374b6](https://github.com/lsy910810-png/leebroHarness/commit/77374b6)).

---

## 2. `Failed to load hooks from ... hooks.json`

### 증상
Plugin install은 성공, agents/commands 작동, 하지만 plugin 상세에서:
```
error:
Failed to load hooks from .../hooks.json: [
  {
    "expected": "record",
    "code": "invalid_type",
    "path": ["hooks"],
    "message": "Invalid input: expected record, received undefined"
  }
]
→ Check hooks.json file syntax and structure
```

### 원인
`hooks.json` 최상위에 `hooks` 키 wrapper 누락. 우리가 처음엔 events를 직접 최상위에 둠:
```json
// 잘못
{ "PostToolUse": [...], "Stop": [...] }
```

Claude Code는 `data.hooks.<EventName>` 경로로 파싱. `data.hooks`가 없어 실패.

### Fix
```json
// 올바름
{
  "hooks": {
    "PostToolUse": [...],
    "Stop": [...]
  }
}
```

scaffold의 `settings.json`은 처음부터 `hooks` 안에 넣었지만, plugin의 별도 `hooks.json`은 같은 구조여야 함을 놓침.

### 검증
```bash
node -e "const j=JSON.parse(require('fs').readFileSync('plugins/harness/hooks/hooks.json','utf8'));console.log('top-level keys:', Object.keys(j))"
# top-level keys: [ 'hooks' ]   ← 정상
```

### 참고
v0.2.1에서 fix ([commit 1fd1e92](https://github.com/lsy910810-png/leebroHarness/commit/1fd1e92)).

---

## 3. `.claude/settings.json` vs `settings.local.json` 혼동

### 증상
- 기존 프로젝트에 `.claude/settings.local.json`만 있음
- "기존 settings 있으니 머지해야 하나?" 헷갈림
- scaffold cp하면 충돌할 것 같아서 망설임

### 원인 (둘은 다른 파일)
| 파일 | 출처 | git tracked? | 역할 |
|---|---|---|---|
| `.claude/settings.json` | **사람이 작성** (scaffold 또는 직접) | ✅ tracked | 프로젝트 baseline permissions·hooks |
| `.claude/settings.local.json` | **Claude Code 자동 생성** (per-user) | ❌ gitignore | 사용자가 prompt에서 "always allow" 선택 시 누적 cache |

### Fix (의사결정 차트)
```
.claude/settings.json 있음?
├── 있음 → 머지 필요 (apply-to-existing-project.md Case 3 또는 4)
└── 없음 → 충돌 없이 cp 가능 (settings.local.json 있어도 무관)
```

### 핵심 룰
- `settings.local.json`은 **건드리지 않음** — Claude Code가 알아서 관리
- `.gitignore`에 `.claude/settings.local.json` 추가 권고 (scaffold들 0.3.1+엔 이미 포함)

### 참고
bamtok 적용 시 발견. apply-to-existing-project.md 보강 후보.

---

## 4. Multi-PC 경로 잔재 (`lsy91` → `august` 등)

### 증상
다른 PC에서 작업한 프로젝트를 옮겨오면 `.claude/settings.local.json`에 이전 PC 경로 하드코딩:
```json
{
  "permissions": {
    "allow": [
      "Bash(/c/Users/lsy91/Desktop/...)",
      ...
    ]
  }
}
```

→ 새 PC에서 같은 명령 실행해도 prompt 다시 뜸 (path 다름).

### 원인
`settings.local.json`은 user-specific cache라 portable하지 않음. PC 이전 시 그대로 따라옴.

### Fix (3가지 옵션)

#### Option A: 일괄 치환
```bash
# Windows Git Bash
sed -i 's|/c/Users/lsy91|/c/Users/august|g' .claude/settings.local.json
sed -i 's|C:\\\\Users\\\\lsy91|C:\\\\Users\\\\august|g' .claude/settings.local.json
```
백업 먼저:
```bash
cp .claude/settings.local.json .claude/settings.local.json.bak
```

#### Option B: 그냥 삭제 (Claude Code가 새로 생성)
```bash
rm .claude/settings.local.json
```
다음 세션에서 prompt 다시 나오겠지만 그때 "always allow" 누르면 새 PC 경로로 누적.

#### Option C: 일부만 정리
필요한 항목만 수동으로 경로 갱신, 나머지는 그대로.

### 추천
**Option B (삭제)** — settings.local.json은 cache라 잃어도 됨. 깨끗한 출발이 더 안전.

### 참고
bamtok에 lsy91 잔재 있음. 사용자가 정리 시점 결정 (이번 plan scope 외).

---

## 5. 큰 monorepo에서 typecheck PostToolUse hook이 너무 느림

### 증상
`scaffolds/next-ts/.claude/settings.json`의 PostToolUse hook이 매 TS edit마다 `tsc --noEmit` 실행:
- 단일 Next.js 앱(파일 수 < 1000): 2~5초 — 견딜 만
- 큰 monorepo(파일 수 5000+): 5~30초 — 매 Edit 차단

### Fix (3가지 옵션)

#### Option A: hook 제거 (가장 간단)
`.claude/settings.json`의 `hooks.PostToolUse` 배열에서 typecheck 항목 빼기.

#### Option B: 변경된 워크스페이스만 (Turbo 경우)
```json
{
  "command": "turbo run typecheck --filter=...changed --no-cache 2>&1 | tail -50"
}
```

#### Option C: incremental + 백그라운드
- `tsc --incremental --noEmit` (캐시)
- 또는 hook 제거하고 IDE LSP에 위임

### 추천
**Option A** (단순) 또는 **Option B** (turbo 모노레포만). turbo-monorepo scaffold는 hooks 미적용으로 시작 — 사용자가 안정화 후 직접 추가.

### 참고
bamtok 적용 시 hooks 보류 결정 ([CLAUDE.md hooks 섹션](https://github.com/lsy910810-png/leebroHarness/blob/main/scaffolds/turbo-monorepo/CLAUDE.md)).

---

## 6. CRLF/LF 변환 경고 매 commit마다

### 증상
```
warning: in the working copy of 'X.md', LF will be replaced by CRLF the next time Git touches it
```
Windows에서 git이 텍스트 파일 commit할 때마다.

### 원인
`core.autocrlf` 기본값이 OS별로 다름. `.gitattributes` 없이는 git이 line-ending 추측.

### Fix
repo 루트에 `.gitattributes`:
```
* text=auto eol=lf
*.bat text eol=crlf
*.cmd text eol=crlf
```

→ 모든 텍스트 파일은 LF로 정규화, Windows batch만 CRLF.

### 참고
v0.3.1에서 추가됨.

---

## 새 에러 발견 시

1. 위 패턴(증상·원인·Fix·검증·참고)으로 새 섹션 추가
2. v0.3.x 누적되면 별 문서로 분리 검토
3. claude-code-guide나 외부 docs 참조 시 url + 발견 일자 명시
