# Audit Output Format

## 응답 템플릿

```
▶ Operating under: 1·Think Before Coding · 2·Simplicity First · 3·Surgical Changes · 4·Goal-Driven Execution

# Principle Audit — <date> — <scope>

## Summary
- 차원 1 (긴 함수/파일): N findings
- 차원 2 (스코프 완화): N findings
- 차원 3 (검증 증거 부재): N findings
- 차원 4 (vague 코멘트/커밋): N findings
- 총 punch list: blocker N, major N, minor N, nit N

## Findings

### 차원 1: 긴 함수/파일 (원칙 2)
- [major] `<file>` (<line count>줄) — split 검토
  • 명령: `wc -l <file>` → <result>
- [minor] `<file>` (<line count>줄)

### 차원 2: 스코프 완화 발자국 (원칙 3)
- [major] commit `<sha>` "<subject>" — <N>개 top-level 디렉토리 동시 변경
  • 명령: `git show --stat <sha>`
  • 권고: 분리해서 별 PR 회고

### 차원 3: 검증 증거 부재 (원칙 4)
- [blocker] PR `<sha>` "<subject>" — src 변경 <N>줄, test 변경 0줄
  • 명령: `git show --numstat <sha>`
  • 권고: follow-up PR로 테스트 추가

### 차원 4: vague 코멘트/커밋 (원칙 4)
- [minor] `<file>:<line>` — `// should work probably`
- [nit] commit `<sha>` "<subject>" — 모호한 커밋 메시지

## 이전 audit과의 비교 (있을 경우)
- 새로 발견: <N>건
- 해소됨: <N>건 (<목록>)
- 지속: <N>건 (<목록>)

## 권고 다음 액션
1. blocker N건은 다음 audit 전까지 해결
2. major 중 우선순위 N건은 이번 sprint
3. minor·nit는 follow-up issue로 일괄 처리

✓ 원칙 체크
- 1: 가정/불확실성 — false positive 가능성 있는 휴리스틱은 "후보" 마킹함 (예시 N건)
- 2: 추가 추상화/스코프 초과 — 0건. 4차원만 스캔.
- 3: 변경 라인이 요청에 직접 추적됨 — 자체 수정 0건. 보고만.
- 4: 검증 — 각 차원 명령 실행 결과 첨부. 발견 0인 차원도 명령 + "0 hits"로 evidence 남김.
```

## 규칙

- **차원별 헤더 항상 출력**, finding 0건이어도 "0 findings, 명령 X 실행함"으로 표기
- **모든 finding은 위치 + 명령 + 권고 3종 세트**
- **이전 audit 비교**는 같은 scope으로 이전 결과가 있을 때만(파일·메모리·이슈에 저장된 경우)
- **자동 수정 안 함** — 보고만. 수정은 별 워크플로우.

## 결과 영구화 (선택)

audit 결과를 따라가려면:
- GitHub issue로: `gh issue create --title "Audit YYYY-MM-DD" --body "<출력>"`
- 파일로: `docs/audits/YYYY-MM-DD.md`로 commit
- 둘 다 사용자가 명시적으로 요청할 때만(원칙 3 — 무관한 변경 X)

## Severity 매핑

| Severity | 의미 | 차원 예 |
|---|---|---|
| **blocker** | 다음 audit 전까지 해결 권고 | 차원 3: 동작 변경 + 테스트 0 |
| **major** | 이번 sprint 안에 처리 | 차원 1: 500+ 줄 파일, 차원 2: multi-concern PR |
| **minor** | follow-up issue로 묶음 | 차원 1: 300~500줄, 차원 4: vague comment 누적 |
| **nit** | 무시 가능 | 차원 4: 단발 vague comment |

## 빈 audit 처리

코드베이스가 아주 작거나 깨끗해서 0 finding인 경우:
```
## Summary
- 차원 1: 0 findings (스캔: `find ... -exec wc -l ...`, 결과 비어있음)
- 차원 2: 0 findings
- 차원 3: 0 findings (`git log --since="1 month ago" -- src/` → 0건)
- 차원 4: 0 findings

원칙 4 검증: 4차원 모두 명령 실행 + 0 hit 확인. signal fatigue 가능성 있으니 다음 audit에서 임계값(500줄, 100줄) 재조정 고려.
```
