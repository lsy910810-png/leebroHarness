# 기존 프로젝트에 하네스 적용하기

이미 작업 중인 프로젝트에 하네스를 적용할 때의 가이드. **새 프로젝트 셋업이라면 [install.md](install.md) 참조 — 여기는 머지 시나리오용.**

## 핵심 원칙: 두 layer 분리

하네스는 항상 두 부분:

| Layer | 무엇이 | 어디에 | 빈도 |
|---|---|---|---|
| **Plugin** (글로벌) | agents · commands · hooks | `~/.claude/plugins/` | PC당 **한 번만** |
| **Scaffold** (프로젝트별) | settings.json (permissions·hooks) + CLAUDE.md (4원칙·visibility protocol·메타) | 각 프로젝트 루트의 `.claude/` + `CLAUDE.md` | 새 프로젝트마다 |

**기존 프로젝트의 risk**: 이미 `CLAUDE.md`나 `.claude/settings.json`이 있을 수 있어 **단순 cp는 위험**(덮어씌움).

---

## Step 1: Plugin 글로벌 install (한 번만, 어디서든)

이미 했으면 skip. 안 했으면 Claude Code CLI에서:

```
/plugin marketplace add https://github.com/lsy910810-png/leebroHarness
/plugin install harness@leebroharness
```

확인:
```
/plugin       # harness 0.2.0 표시
/agents       # code-reviewer, tester, principle-auditor 3개
```

이 단계는 어느 폴더에서 해도 되고, 한 번 install하면 모든 프로젝트(현재·미래)에 동일 적용.

---

## Step 2: 기존 프로젝트 상태 진단

프로젝트 디렉토리에서:

```bash
cd /path/to/existing-project
ls -la CLAUDE.md .claude/ 2>&1
```

다음 4가지 case 중 어디에 해당하는지 판단.

---

## Case 1: `.claude/`도 없고 `CLAUDE.md`도 없음

**가장 단순**. 그냥 cp.

```bash
cd /path/to/existing-project

# Next.js + TypeScript 프로젝트
cp -r "C:/Users/august/Desktop/Harness Engineering/scaffolds/next-ts/.claude" .
cp "C:/Users/august/Desktop/Harness Engineering/scaffolds/next-ts/CLAUDE.md" .

# 또는 범용
cp -r "C:/Users/august/Desktop/Harness Engineering/scaffolds/general/.claude" .
cp "C:/Users/august/Desktop/Harness Engineering/scaffolds/general/CLAUDE.md" .
```

이후 `CLAUDE.md`의 `<name>` + Project Meta 섹션 채우기.

→ **Step 4 (검증)** 로 이동.

---

## Case 2: 기존 `CLAUDE.md`만 있음

기존 CLAUDE.md엔 프로젝트 도메인·컨벤션 등 가치 있는 내용이 이미 있을 것. **덮어쓰지 말고 머지**.

### 2-1. 백업
```bash
cp CLAUDE.md CLAUDE.md.bak
```

### 2-2. 두 섹션만 추가
기존 CLAUDE.md를 열어 다음 두 섹션을 적절한 위치(보통 상단 인트로 다음)에 **삽입**:

`scaffolds/next-ts/CLAUDE.md` 또는 `scaffolds/general/CLAUDE.md`에서 다음 부분 그대로 복사:

- `## Operating Principles (always apply)` 섹션 전체 (4원칙)
- `## Visibility Protocol (every response)` 섹션 전체 (3단)

이후 기존 콘텐츠는 **그대로 유지**. 기존의 컨벤션·도메인 설명·테스트 명령 등은 손대지 않음.

### 2-3. .claude/settings.json도 같이 복사 (있으면 좋음)
```bash
cp -r "C:/Users/august/Desktop/Harness Engineering/scaffolds/next-ts/.claude" .
```

(기존 `.claude/`가 없으니 충돌 없음)

→ **Step 4 (검증)** 로 이동.

---

## Case 3: 기존 `.claude/settings.json`만 있음

`CLAUDE.md`는 cp로 그냥 복사 가능. settings.json은 **머지** 필요.

### 3-1. CLAUDE.md 복사
```bash
cp "C:/Users/august/Desktop/Harness Engineering/scaffolds/next-ts/CLAUDE.md" .
```

(기존 CLAUDE.md 없으니 충돌 없음)

### 3-2. settings.json 머지
기존 `.claude/settings.json`의 `permissions.allow`/`deny` 배열에 scaffold 거 **합치기** (중복 제거):

```json
{
  "permissions": {
    "allow": [
      "<기존 항목들 그대로>",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(npm run dev)",
      "Read(./**)"
    ],
    "deny": [
      "<기존 deny 그대로>",
      "Read(./.env*)"
    ]
  },
  "hooks": {
    "<기존 hooks 있으면 PostToolUse·Stop 배열에 scaffold 항목 추가>"
  }
}
```

scaffold 전체 복사하려면 백업 후 덮어쓰기:
```bash
cp .claude/settings.json .claude/settings.json.bak
cp "C:/Users/august/Desktop/Harness Engineering/scaffolds/next-ts/.claude/settings.json" .claude/settings.json
# 이후 .bak 보면서 사라진 기존 항목 다시 추가
```

→ **Step 4 (검증)** 로 이동.

---

## Case 4: 둘 다 있음 (`CLAUDE.md` + `.claude/settings.json`)

Case 2 + Case 3을 둘 다 적용. 또는 가장 가벼운 옵션:

### Case 4-Alt: Plugin만으로 시작 (머지 부담 회피)

기존 CLAUDE.md엔 **딱 한 줄**만 추가:
```markdown
## Operating Principles
See [Harness Engineering PRINCIPLES.md](https://github.com/lsy910810-png/leebroHarness/blob/main/PRINCIPLES.md) — apply to every response.
```

`.claude/settings.json`은 손대지 않음.

**Trade-off**:
- ✅ 기존 자산 100% 보존, 작업 1분
- ✅ Plugin agents(`/review`, `/test`, `/audit`)는 정상 작동 — 4원칙 자동 적용
- ⚠️ 메인 세션 출력의 visibility protocol(앵커·종료 체크)은 **약화** — agent 호출 안에선 강제되지만 일반 코딩 세션에선 강제 약함

원칙을 더 강하게 박고 싶으면 Case 2 머지 권고.

---

## Step 4: 검증

머지 후 새 Claude Code 세션 열어서 다음 확인:

1. **응답 첫 줄 앵커**: `▶ Operating under: 1·Think Before Coding · 2·Simplicity First · 3·Surgical Changes · 4·Goal-Driven Execution`
   - 보이면 → CLAUDE.md 머지 성공
   - 안 보이면 → CLAUDE.md에 Operating Principles + Visibility Protocol 섹션 추가됐는지 재확인

2. **응답 마지막 `✓ 원칙 체크` 블럭**: 4원칙 각 한 줄
   - 보이면 → protocol 강제 작동
   - 안 보이면 → 새 세션 재시작, 또는 CLAUDE.md 위치 확인 (프로젝트 루트인지)

3. **Permissions 작동**: 자주 쓰는 명령(예: `npm run dev`)이 prompt 없이 실행되는지
   - 안 되면 → settings.json `permissions.allow` 머지 빠짐

4. **Plugin 작동**: `/review` 호출 시 code-reviewer agent 응답
   - 안 되면 → `/plugin install harness@leebroharness` 안 됐을 수 있음, 다시 install

5. **(next-ts만) hooks 작동**: TS 파일 Edit 후 typecheck 실행되는지
   - 너무 느리거나 안 되면 → settings.json `hooks` 섹션 손대거나 제거

---

## Common pitfalls (자주 하는 실수)

### 1. cp -r로 기존 `.claude/`나 `CLAUDE.md` 덮어씌움
→ 항상 백업 먼저. `cp .claude/settings.json .claude/settings.json.bak`

### 2. next-ts scaffold를 다른 스택 프로젝트(Python·Go 등)에 적용
→ permissions이 npm/tsc/next 위주라 안 맞음. **`general/` scaffold** 쓰거나 [extend.md](extend.md)의 "새 scaffold 추가" 가이드 따라 본인 스택용 새 scaffold 만들기.

### 3. typecheck hook이 너무 느림
→ next-ts scaffold의 `tsc --noEmit`이 큰 프로젝트(5000+ 파일)에서 매 Edit마다 5~30초. 거슬리면 settings.json `hooks.PostToolUse` 항목 제거하고 Stop hook만 유지.

### 4. CLAUDE.md 머지 후 기존 컨벤션이 과도하게 무시됨
→ 4원칙 섹션을 너무 위에 두면 LLM이 그것만 보고 끝낼 수 있음. **인트로 → 4원칙 → 기존 컨벤션·도메인** 순서 유지 권고.

### 5. Plugin이 GitHub URL로 install 안 됨
→ `/plugin marketplace add` 시 `https://github.com/lsy910810-png/leebroHarness` (확장자 `.git` 있어도 OK). 안 되면 로컬 경로로:
```
/plugin marketplace add C:/Users/august/Desktop/Harness Engineering
```

---

## 빠른 참조 (TL;DR)

```bash
# 1. Plugin (어디서든, 한 번만)
# Claude Code CLI:
/plugin marketplace add https://github.com/lsy910810-png/leebroHarness
/plugin install harness@leebroharness

# 2. 프로젝트 상태 확인
cd /path/to/existing-project
ls -la CLAUDE.md .claude/ 2>&1

# 3. Case에 따라:
#   Case 1 (둘 다 없음): cp -r ... .claude && cp ... CLAUDE.md
#   Case 2 (CLAUDE.md만): 백업 후 두 섹션 머지 + .claude/ cp
#   Case 3 (.claude/만): CLAUDE.md cp + settings.json 머지
#   Case 4 (둘 다 있음): Case 2 + Case 3, 또는 Plugin만 + CLAUDE.md 한 줄

# 4. 검증
# 새 Claude Code 세션 → 응답에 ▶ 앵커 + ✓ 종료 체크 보이는지
```

질문 있으면 본 가이드 + [install.md](install.md) + [extend.md](extend.md) 참조.
