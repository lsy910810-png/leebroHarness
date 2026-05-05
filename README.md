# Harness Engineering

신규 프로젝트의 **기본기**로 쓸 Claude Code 하네스 모음. 4가지 핵심 원칙(Think Before Coding / Simplicity First / Surgical Changes / Goal-Driven Execution)이 모든 agent와 모든 새 프로젝트에 박혀 들어갑니다.

원칙 본문은 [PRINCIPLES.md](./PRINCIPLES.md). 아키텍처 결정 배경은 [docs/architecture-guide.md](./docs/architecture-guide.md).

## 구성 (v0.2.0)

- **Plugin** (`plugins/harness/`) — 한 번 install하면 어디서든 enable.
  - **agents**: `code-reviewer`, `tester`, `principle-auditor`
  - **commands**: `/review`, `/test`, `/audit`
  - **skills**: `security-review` (injection·secrets — description 매칭으로 자동 활성화)
  - **hooks**: PostToolUse(prettier 자동 포맷), Stop(beep)
- **Scaffolds** (`scaffolds/`) — 프로젝트마다 복사해서 시작.
  - `general/`: 언어 무관 안전 baseline
  - `next-ts/`: Next.js + TypeScript. 추가로 typecheck(PostToolUse)·lint+test(Stop) hook 포함

## 빠른 시작

### 1. Marketplace 등록 (한 번만)

GitHub URL (어느 PC에서나):
```
/plugin marketplace add https://github.com/lsy910810-png/leebroHarness
```

또는 로컬 경로:
```
/plugin marketplace add C:/Users/august/Desktop/Harness Engineering
```

### 2. 플러그인 설치

```
/plugin install harness@leebroharness
```

`@leebroharness`는 marketplace 이름 (marketplace.json의 top-level `name` 필드).

설치 확인은 `/plugin`. agent 확인은 `/agents`.

### 3. 새 프로젝트에 scaffold 적용

```bash
# Next.js 프로젝트
cp -r "C:/Users/august/Desktop/Harness Engineering/scaffolds/next-ts/.claude" /path/to/new-project/
cp "C:/Users/august/Desktop/Harness Engineering/scaffolds/next-ts/CLAUDE.md" /path/to/new-project/

# 범용
cp -r "C:/Users/august/Desktop/Harness Engineering/scaffolds/general/.claude" /path/to/new-project/
cp "C:/Users/august/Desktop/Harness Engineering/scaffolds/general/CLAUDE.md" /path/to/new-project/
```

### 4. 자주 쓰는 명령

- `/review` — 현재 브랜치/diff를 code-reviewer agent로 검토
- `/test [target]` — tester agent로 테스트 작성·실행·검증
- `/audit [scope]` — principle-auditor가 4원칙 위반 스캔 (긴 함수/파일·스코프 완화·검증 부재·vague 코멘트)

### 5. 원칙 갱신 시

`PRINCIPLES.md` 수정 후 [docs/extend.md](./docs/extend.md)의 동기화 체크리스트에 따라 **6곳**(PRINCIPLES.md + 3 agent full text + 2 scaffold condensed) 반영.

## 더 읽을거리

- [docs/install.md](./docs/install.md) — 설치 단계 + 검증
- [docs/apply-to-existing-project.md](./docs/apply-to-existing-project.md) — **이미 작업 중인 프로젝트에 머지하는 가이드** (4 case 분기)
- [docs/extend.md](./docs/extend.md) — agent/command/skill 추가, progressive disclosure 패턴, 원칙 동기화
- [docs/architecture-guide.md](./docs/architecture-guide.md) — Forward-only layers, Providers, repository-first context (opt-in)
