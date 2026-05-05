# Extend Guide

추가 자료:
- [architecture-guide.md](architecture-guide.md) — forward-only layers, Providers, repository-first context (opt-in for projects)
- [install.md](install.md) — plugin 설치·scaffold 적용 단계

---

## 새 agent 추가

1. `plugins/harness/agents/<name>.md` 생성.
2. YAML frontmatter:
   ```yaml
   ---
   name: my-agent
   description: <한 줄 설명>. Use proactively when ...
   tools: Read, Grep, Glob, Bash
   model: sonnet
   ---
   ```
3. **반드시** 본문 첫머리에 다음 두 섹션을 그대로 붙여넣기:
   - `# Operating Principles (always apply)` — `PRINCIPLES.md`의 1~4번 전문
   - `# Visibility Protocol (must follow in EVERY response)` — 3단 protocol 전문
   (기존 `code-reviewer.md`나 `tester.md`에서 복사하면 빠릅니다.)
4. 그 아래 `# Role: ...`로 agent 고유 워크플로우.
5. **(선택) 지식이 크면 Progressive Disclosure** — 아래 섹션 참조.
6. plugin 재로드: `/plugin disable harness && /plugin enable harness` (또는 새 세션 시작).

## Progressive Disclosure — agent를 슬림하게 유지

agent.md에 모든 지식을 박으면 system prompt가 무거워지고, 일부 검토만 필요한 경우에도 전체가 로드됨. 다음 두 가지로 분리합니다.

### 패턴 A — Per-agent 지식 폴더

agent 고유 mechanics(output 형식, severity 룰, framework 탐지 등)는 sibling 폴더로:

```
plugins/harness/agents/
├── my-agent.md           ← 슬림 TOC + 원칙 + role
└── my-agent/             ← 지식 폴더
    ├── 01-checklists.md
    ├── 02-output-format.md
    └── examples/
        └── good-pr.md
```

agent.md의 `# Knowledge Base` 섹션에 링크:
```markdown
## Per-agent 자료
- **`my-agent/01-checklists.md`** — <언제 read하는지>
- **`my-agent/02-output-format.md`** — 보고서 작성 직전 read
```

agent는 검토 진행 중 필요할 때 Read tool로 해당 파일을 lazy-load. 마크다운 링크 형태(`[file](./my-agent/01.md)`)도 가능.

### 패턴 B — Skills (cross-agent 공유)

여러 agent와 메인 세션이 공유할 도메인 지식은 Skill로:

```
plugins/harness/skills/
└── my-skill/
    ├── SKILL.md          ← description으로 자동 매칭
    ├── detail-1.md
    └── detail-2.md
```

`SKILL.md` frontmatter:
```yaml
---
name: my-skill
description: <언제 이 skill을 써야 하는지 명확히>. Use when reviewing <X>, verifying <Y>.
---
```

agent.md에서 사용:
```markdown
## Skills
- **`my-skill`** skill — <어떤 상황에서 자동 활성화>. 키워드 X, Y 발견 시 끌어와 사용.
```

description 매칭으로 Claude가 자동으로 SKILL.md를 컨텍스트에 로드. SKILL.md는 짧은 index고, sibling 파일은 SKILL.md가 시키는 대로 lazy-read.

### 어디에 둘지 판단

| 판단 | A (per-agent) | B (skill) |
|---|---|---|
| 한 agent만 씀 | ✅ | ❌ |
| 여러 agent + 메인 세션 공유 | ❌ | ✅ |
| agent의 출력 형식·내부 룰 | ✅ | ❌ |
| 도메인 지식(보안 룰, framework 패턴) | △ | ✅ |
| 자주 바뀜 | A 또는 B 둘 다 OK | OK |

### 현재 구현 예시

- `agents/code-reviewer/` — checklists, severity-tagging, output-format (per-agent)
- `agents/tester/` — framework-detection, execute-verify-loop, diagnosis-patterns (per-agent)
- `agents/principle-auditor/` — audit-checks, audit-output (per-agent)
- `skills/security-review/` — injection, secrets (cross-agent: code-reviewer + tester 둘 다 사용)

## 새 slash command 추가

1. `plugins/harness/commands/<name>.md` 생성.
2. frontmatter:
   ```yaml
   ---
   description: <명령 설명>
   ---
   ```
3. 본문에 agent 위임 또는 직접 지시 작성. `$ARGUMENTS`로 사용자 입력 받음.
4. command는 짧으므로 4원칙 임베드 안 함 — agent로 위임하면 agent가 처리.

## 원칙 동기화 체크리스트

`PRINCIPLES.md` 수정 시 다음 **6곳**을 모두 반영해야 합니다. 두 flavor 주의:

**Full text (단일 출처와 동일하게)**:
- [ ] `PRINCIPLES.md` (단일 출처, 가장 먼저 수정)
- [ ] `plugins/harness/agents/code-reviewer.md` — `# Operating Principles` 섹션
- [ ] `plugins/harness/agents/tester.md` — `# Operating Principles` 섹션
- [ ] `plugins/harness/agents/principle-auditor.md` — `# Operating Principles` 섹션

**Condensed (1줄 요약 + 1~2 sub-bullet — D 슬림화 적용 후)**:
- [ ] `scaffolds/general/CLAUDE.md` — `## Operating Principles` 섹션
- [ ] `scaffolds/next-ts/CLAUDE.md` — `## Operating Principles` 섹션

Visibility Protocol을 바꾼 경우 6곳 다 같이 갱신. scaffold CLAUDE.md의 protocol은 short form이므로 줄여 반영.

검증:
```bash
# Full text 4곳 (PRINCIPLES.md + 3 agents). extend.md는 grep 패턴 인용으로 self-match라 제외.
grep -rl 'Don.t .improve. adjacent code' "C:/Users/august/Desktop/Harness Engineering" --include='*.md' --exclude=extend.md | wc -l
# 4

# Condensed: 4원칙 이름 모두 등장 (full text 4 + condensed 2 = 6, README/docs도 hit하면 더)
grep -rl 'Surgical Changes' "C:/Users/august/Desktop/Harness Engineering" --include='*.md' | wc -l
# 6 이상 (PRINCIPLES + 3 agents + 2 scaffolds + 가능한 README/docs)
```

## 새 scaffold 추가

예: Python scaffold

1. `scaffolds/python/.claude/settings.json` — Python 도구 allowlist (`uv`, `pytest`, `ruff`).
2. `scaffolds/python/CLAUDE.md` — `general/CLAUDE.md`를 복사해 Project Meta만 Python 스택용으로 교체.
3. `README.md`의 "scaffold 적용" 섹션에 사용 예시 추가.

## sync 자동화 (보류)

5곳 일치를 자동 보장하는 스크립트는 일부러 만들지 않았습니다(원칙 2 — Simplicity First). 텍스트가 짧고 자주 안 바뀌므로 수동 동기화로 충분. 갱신 빈도가 늘어나면 그때 도입.

## 검증 명령

하네스 자체의 health check. 메모리 `harness_layout.md`가 invariant만 유지하기 위해 명령 블럭을 여기로 이전.

```bash
cd "C:/Users/august/Desktop/Harness Engineering"

# 파일 개수 (현재 v0.2.0 기준 30 + decisions/README 등 신규)
find . -type f -not -path "./.claude/*" | wc -l

# Full text 임베드 4곳 (PRINCIPLES.md + 3 agents). extend.md self-match 방지.
grep -rl 'Don.t .improve. adjacent code' . --include='*.md' --exclude=extend.md

# 4원칙 이름이 등장하는 모든 파일 (full 4 + condensed 2 + docs/README mention들)
grep -rl 'Surgical Changes' . --include='*.md' | wc -l

# JSON 5개 유효성
for f in .claude-plugin/marketplace.json plugins/harness/.claude-plugin/plugin.json \
         plugins/harness/hooks/hooks.json scaffolds/general/.claude/settings.json \
         scaffolds/next-ts/.claude/settings.json; do
  cat "$f" | node -e "JSON.parse(require('fs').readFileSync(0,'utf8'))" && echo "$f: ok"
done

# scaffold CLAUDE.md 줄수 (target ≤ 80)
wc -l scaffolds/{general,next-ts}/CLAUDE.md

# agent.md 줄수 (target ≤ 200, slim TOC)
wc -l plugins/harness/agents/*.md
```

원칙 임베드/JSON/줄수 중 하나라도 expected와 다르면 그게 drift 신호 — 추적해서 동기화.
