# Install Guide

## Plugin 설치

### 1. Marketplace 등록

Claude Code 세션에서 — GitHub URL (다른 PC에서도 동일하게):
```
/plugin marketplace add https://github.com/lsy910810-png/leebroHarness
```

또는 이 PC의 로컬 경로:
```
/plugin marketplace add C:/Users/august/Desktop/Harness Engineering
```

내부적으로 `.claude-plugin/marketplace.json`을 읽어 `harness` 플러그인을 노출합니다.

### 2. 플러그인 설치 + enable

```
/plugin install harness
```

### 3. 확인

- `/plugin` — 설치된 플러그인 목록에 `harness` 표시되는지.
- `/agents` — `code-reviewer`, `tester`, `principle-auditor`가 보이는지.
- `/review`, `/test`, `/audit` — 슬래시 명령으로 자동완성되는지.
- `/audit` 한 번 호출 → principle-auditor가 4차원(긴 함수/파일·스코프 완화·검증 부재·vague 코멘트) 스캔하고 punch list 출력하는지.

## Scaffold 적용

### 신규 프로젝트 (빈 폴더)

```bash
# Next.js + TypeScript
cp -r "C:/Users/august/Desktop/Harness Engineering/scaffolds/next-ts/.claude" .
cp "C:/Users/august/Desktop/Harness Engineering/scaffolds/next-ts/CLAUDE.md" .

# 범용
cp -r "C:/Users/august/Desktop/Harness Engineering/scaffolds/general/.claude" .
cp "C:/Users/august/Desktop/Harness Engineering/scaffolds/general/CLAUDE.md" .
```

복사 후 `CLAUDE.md`의 `<name>`, Project Meta 섹션 채우기.

### 이미 작업 중인 프로젝트

기존 `CLAUDE.md`나 `.claude/settings.json` 있으면 단순 cp는 위험(덮어쓰기). 4가지 case별 머지 가이드는 **[apply-to-existing-project.md](apply-to-existing-project.md)** 참조.

## Hook 동작 확인

- 임의의 `.ts`/`.tsx`/`.js`/`.jsx`/`.json`/`.md` 파일을 Edit/Write → prettier가 설치돼 있다면 자동 포맷.
- 세션 종료 → beep 소리(Windows).

prettier가 프로젝트에 설치돼 있지 않으면 npx가 임시 설치해 실행합니다(`--yes` 플래그). 자동 포맷이 거슬리면 `plugins/harness/hooks/hooks.json`에서 PostToolUse 항목을 제거하세요.

## 트러블슈팅

- **`/plugin install`이 실패**: `.claude-plugin/marketplace.json` 경로와 `plugins/harness/.claude-plugin/plugin.json`의 `name` 일치 확인.
- **agent가 자동 호출 안 됨**: agent description의 "Use proactively" 문구 + tools 명시 확인.
- **hook이 안 돌음**: `/hooks` 명령으로 활성 hook 확인. plugin이 enable 상태인지 확인.
