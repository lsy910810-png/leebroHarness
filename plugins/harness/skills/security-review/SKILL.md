---
name: security-review
description: Security checklist for reviewing code. Covers injection (SQL/NoSQL/command) and secrets handling. Use when reviewing code for security risks (queries, env access, hardcoded credentials) or verifying that security-sensitive changes are tested.
---

# Security Review

코드의 보안 위험을 점검할 때 사용하는 체크리스트입니다. 이 SKILL.md는 짧은 index고, 상황별 상세는 sibling 파일을 read하세요.

## Quick triage

PR/diff를 보고 다음 키워드가 있으면 해당 sibling 파일을 read:

| 트리거 | Read |
|---|---|
| `query`, `prepare`, `execute`, `WHERE ${...}`, raw SQL/NoSQL 조립 | [injection.md](injection.md) |
| `process.env`, `os.environ`, `.env`, hardcoded string with `_KEY`/`_TOKEN`/`_SECRET` | [secrets.md](secrets.md) |

> Auth(bcrypt/jwt/session/csrf) 관련 검토는 본 skill scope 외 — 필요 시 별 skill 또는 사람이 검토. 향후 auth.md 추가 시 description 업데이트 + ADR 작성.

## Universal checks (모든 보안 검토에 적용)

- **입력 신뢰 경계 확인** — 외부에서 들어온 모든 값(req.body, query string, header, cookie, file upload)을 신뢰 안 함.
- **출력 인코딩** — DB·HTML·shell·로그로 나가는 데이터가 컨텍스트별 escape됐는지.
- **권한 검사 위치** — 인증(누구냐) ≠ 인가(뭘 할 수 있냐). 둘 다 해당 endpoint에서 확인되는지.

## 검토 결과 보고

발견 시 다음 형식으로:
```
[blocker · 원칙 4] <파일>:<라인> — <취약점 종류>: <짧은 설명>
  • PoC/시나리오: <어떻게 악용 가능한가>
  • 조치: <구체적 수정안>
  • 검증: <어떤 테스트로 확인 가능>
```

심각도 기준:
- **blocker**: 인증 우회·SQL injection·secret 노출 등 즉시 악용 가능
- **major**: 위험하지만 추가 조건 필요 (예: 특정 DB 권한)
- **minor**: 방어 깊이 향상(예: 이미 sanitize됐지만 prepared statement로 바꾸면 좋음)
