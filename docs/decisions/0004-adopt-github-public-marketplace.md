# ADR 0004: Promote Local Marketplace to Public GitHub Repo (MIT-licensed)

- **Status**: Accepted
- **Date**: 2026-05-05
- **Supersedes**: —

## Context

v0.1.0~v0.2.0 동안 하네스는 **로컬 file:// marketplace**로 운용. `/plugin marketplace add C:/Users/august/Desktop/Harness Engineering`로 이 PC에서만 install 가능. 결과적으로:

- **백업 없음** — 디렉토리 손상 시 복구 불가
- **멀티 PC 동기화 불가** — 다른 워크스테이션에서 사용하려면 수동 복사
- **공유 불가** — 팀원이나 다른 사람에게 share 불가
- **변경 history 없음** — git init 안 된 상태라 v0.1.0 → v0.2.0 변경 추적 안 됨 (decisions.md가 비공식 changelog 역할)

ADR-0003(#7)에서 "GitHub repo로 marketplace 옮기기"를 **향후 확장**으로 명시했고, v0.2.0 sanity check 시점(install 직전)에 실행 결정.

## Decision

**Public GitHub repo로 승격**:
- URL: `https://github.com/lsy910810-png/leebroHarness`
- License: **MIT** (LICENSE 파일 repo 루트에 추가)
- Branch: `main` (default)
- Commit author: `lsy910810-png` + `lsy910810-png@users.noreply.github.com` (repo-local config, GitHub privacy email)

설치 옵션 두 가지로 확장:
```
# GitHub URL (어느 PC에서나, 다른 사람도)
/plugin marketplace add https://github.com/lsy910810-png/leebroHarness

# 로컬 경로 (이 PC에서)
/plugin marketplace add C:/Users/august/Desktop/Harness Engineering
```

## Consequences

**긍정**:
- 어느 PC에서나 설치 가능 — multi-workstation workflow 활성
- 백업 자동화 — git push + GitHub remote가 source of truth
- 변경 history 추적 — `git log`로 v0.1.0~v0.2.0 변경 가시화 (단, 첫 commit이 v0.2.0 시점이라 v0.1.0 history는 잃음)
- ADR-0001~0003의 4차원 audit 중 차원 2·3(git 의존)이 활성화됨 — principle-auditor가 commit history 기반 검증 가능
- 다른 사람·팀원에게 share·fork 가능 — MIT 라이센스로 사용 자유

**부정**:
- **모든 변경 public** — 메모리(`~/.claude/projects/.../memory/`)는 별도 디렉토리라 안전하지만, 커밋 메시지·코드에 secret 들어가지 않게 주의 필요
- v0.1.0의 history는 **잃음** — 첫 commit이 v0.2.0 baseline이라 진화 과정 추적 불가 (decisions.md/ADR로 부분 보존)
- gh CLI 부재로 GitHub repo 자동 생성 불가 — repo는 사용자가 web에서 수동 생성

**후속 작업**:
- git tag `v0.2.0` 부여 (이 ADR과 함께 commit) → GitHub Releases 페이지 활성
- 향후 v0.3.0+ 변경 시 ADR-0005+ 추가 + tag

## Alternatives considered

### A. Local file:// 유지 (status quo)
- **기각 사유**: 백업·공유·멀티 PC 모두 못 함. ADR-0003 "향후 확장"의 핵심.

### B. Private GitHub repo
- **기각 사유**: 다른 사람이 `/plugin marketplace add`로 설치하려면 SSH/PAT 셋업 필요 — 마찰 큼. 공유 의도와 안 맞음. 메모리는 별도 디렉토리라 secret risk 낮음.

### C. GitLab or Bitbucket
- **기각 사유**: GitHub이 dominant이고 Claude Code의 marketplace add가 GitHub URL을 자연스레 인식. GitLab/Bitbucket도 작동은 하지만 friction.

### D. GitHub Gist (single-file hosting)
- **기각 사유**: marketplace는 디렉토리 구조(`.claude-plugin/marketplace.json` + `plugins/harness/`) 필요. Gist는 flat file이라 부적합.

### E. NPM / pip / 다른 패키지 매니저
- **기각 사유**: Claude Code plugin 시스템과 호환 안 됨. 추가 wrapper 필요해 복잡도 큼.

### F. Public GitHub repo + MIT (선택됨)
- **선택 사유**: 백업 + 공유 + history + MIT permissive. 가장 단순하고 ecosystem 자연스러움.

## Related

- 코드 위치:
  - `LICENSE` (repo 루트, MIT 본문)
  - `.gitignore` (`.claude/settings.local.json` 등 per-user 파일 제외)
  - `README.md`, `docs/install.md` — marketplace 등록 명령에 GitHub URL 옵션 추가
  - `scaffolds/general/CLAUDE.md`, `scaffolds/next-ts/CLAUDE.md` — "Harness Engineering" + "PRINCIPLES.md" 링크 GitHub URL로 복원
  - `marketplace.json`, `plugin.json` — version 0.2.0 (변경 없음)
- 메모리: `decisions.md` 항목 #8
- 관련 ADR: ADR-0003 (#7 OpenAI patterns)에서 "GitHub repo로 marketplace 옮기기"를 향후 확장으로 명시한 그 결정의 실행
- 첫 커밋: `2d8f352` (Initial commit: Harness Engineering v0.2.0)
- URL 복원 커밋: `b2dc018` (Restore GitHub URLs in scaffold CLAUDE.md + README install options)
- 출처: 직전 세션의 사용자 의사 결정 (Public 선택, MIT 채택, lsy910810-png/leebroHarness URL 제공)
