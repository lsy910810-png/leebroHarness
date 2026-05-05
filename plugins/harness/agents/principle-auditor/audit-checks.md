# Audit Checks

4가지 차원의 구체 스캔 명령과 판정 기준.

## 차원 1: 긴 함수/파일 (원칙 2 — Simplicity First)

### 긴 파일 스캔
```bash
# 500줄 이상 소스 파일
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.py" -o -name "*.go" -o -name "*.rs" \) \
  -not -path "./node_modules/*" -not -path "./.next/*" -not -path "./dist/*" -not -path "./.git/*" \
  -exec wc -l {} \; | awk '$1 > 500 {print}' | sort -rn
```

### 긴 함수 스캔 (휴리스틱)
TypeScript/JavaScript:
```bash
# 함수 정의 라인부터 다음 빈 줄까지가 100줄 이상 — 휴리스틱이라 false positive 가능
ast-grep --pattern 'function $NAME($$$) { $$$ }' --lang ts | awk '/^function/{c=1;n=0} c{n++} /^$/{if(c && n>100) print prev; c=0} {prev=FILENAME":"NR}'
```

휴리스틱이 부정확하면 grep으로 "function ... {" 패턴 찾고 사람이 점검 권고.

### 판정
- **major**: 500줄 이상 파일 또는 100줄 이상 함수 (신규 또는 기존)
- **minor**: 300~500줄 파일 또는 50~100줄 함수 (검토 권고)

### Caveat
- 자동 생성 파일(스키마·번역)은 제외
- 테스트 파일은 임계값 2배 (테스트는 길어도 OK)

---

## 차원 2: 스코프 완화 발자국 (원칙 3 — Surgical Changes)

### Multi-concern PR 휴리스틱
```bash
# 최근 commit 또는 PR diff에서 이질적 디렉토리 변경
git log --since="1 week ago" --pretty=format:"%H" | while read sha; do
  dirs=$(git show --name-only --pretty=format: $sha | awk -F/ '{print $1}' | sort -u | wc -l)
  files=$(git show --name-only --pretty=format: $sha | wc -l)
  if [ "$dirs" -gt 4 ] && [ "$files" -gt 5 ]; then
    echo "$sha: $dirs top-level dirs, $files files — possible scope creep"
  fi
done
```

### "drive-by" 변경 패턴
```bash
# 한 PR에 명백히 무관한 키워드 두 개 이상 동시 등장
git log --since="1 week ago" -p | \
  grep -E "^\+.*(rename|refactor|cleanup|fix typo)" -B 1 | \
  grep "^commit"
```

### 판정
- **major**: 같은 commit에 4+ 다른 top-level 디렉토리 변경 + 5+ 파일
- **minor**: commit message에 "and"·"plus"·"also" 같은 이중 의도 표현

### Caveat
- `package.json` + `package-lock.json` 같은 자동 동반 파일은 카운트 제외
- 의도된 큰 리팩토링 PR은 사람이 manually whitelist

---

## 차원 3: 검증 증거 부재 (원칙 4 — Goal-Driven Execution)

### 동작 변경 vs 테스트 추가 비율
```bash
# 최근 N개 PR에서 src/ 변경 라인 수 vs test/spec 변경 라인 수
git log --since="1 month ago" --pretty=format:"%H %s" --numstat | \
  awk '
    /^[a-f0-9]{40}/ { if(sha) print sha,subj,src,tst; sha=$1; subj=$2;src=0;tst=0; next }
    /\.(test|spec)\.[jt]sx?$/ { tst+=$1 }
    /\.[jt]sx?$/ && !/\.(test|spec)\./ { src+=$1 }
    END { print sha,subj,src,tst }
  ' | awk '$3 > 50 && $4 == 0 { print }'
```

→ src/ 50줄 이상 변경했는데 test/spec 0줄인 PR 출력.

### 판정
- **blocker** (다음 audit 전까지 해결 권고): 동작 변경 50+ 라인 + 테스트 변경 0 라인
- **major**: 동작 변경 200+ 라인 + 테스트 변경 < 10 라인

### Caveat
- 문서·설정 전용 PR은 자동 제외
- 의도적 "테스트 별 PR" 패턴이 있으면 사람이 화이트리스트

---

## 차원 4: vague 코멘트/커밋 메시지 (원칙 4 — 검증 회피의 흔적)

### 코드 주석 grep
```bash
grep -rEn --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" --include="*.go" \
  -- "(// |# |/\* )(should work|probably|might|maybe|hope|이론상|아마|되겠지)" \
  --exclude-dir=node_modules --exclude-dir=.git
```

### 커밋 메시지 grep
```bash
git log --since="1 month ago" --pretty=format:"%h %s" | \
  grep -iE "should work|probably|maybe|hopefully|fix stuff|misc|wip|untested"
```

### 판정
- **minor**: 각 hit마다 한 항목. 누적 5+ 면 패턴이라고 보고
- **nit**: 단발성 1~2건은 noise일 수 있음

### Caveat
- 영어/한국어 키워드 다 포함
- "should not" 같은 가지치기 안 되는 패턴은 사람이 검토
- **Self-reference 주의**: 본 auditor의 docs(audit-checks.md, audit-output.md, severity-tagging.md), code review docs, ADR(0002 같이 패턴을 인용하는 결정문서)은 grep 패턴을 메타 텍스트로 포함. 위 명령은 `*.ts/.tsx/.js/.py/.go`만 include라 자연 보호되지만, markdown까지 확장 시:
  ```bash
  grep -rEn --include="*.md" -- "should work|probably|...." . \
    --exclude-dir=docs/decisions \
    --exclude=audit-checks.md --exclude=audit-output.md --exclude=severity-tagging.md
  ```

---

## Auditor 적용 범위 (중요)

**source code 프로젝트용**이라 다음 케이스엔 의미 제한:

- **Metadata-only repo** (markdown/json만, 소스 0): 차원 1(긴 함수)·차원 4(소스 vague 코멘트) 적용 대상 없음. 차원 2·3은 git 있으면 의미 있음.
- **Git 미초기화 repo**: 차원 2·3 N/A (commit history 없음). 차원 1·4만 가능.
- **단일 commit만 있는 repo**: 차원 2·3 의미 약함.

이런 repo는 별도 sanity check가 더 적합:
- 하네스 자체: `docs/extend.md`의 "검증 명령" 섹션 (invariant grep + JSON 유효성 + 줄수)
- 다른 metadata repo: 프로젝트별 invariant 정의 후 grep

---

## 실행 순서 권장

1. 차원 1·2는 빠르고 deterministic — 항상 돌리기
2. 차원 3은 git log 기반 — 1개월치만(메모리 절약)
3. 차원 4는 grep — 결과 많으면 head -50 정도로 자르기

발견 0건일 때도 **각 차원에 어떤 명령을 돌렸는지** evidence 첨부 (원칙 4 — 검증 못 했으면 못 했다고).
