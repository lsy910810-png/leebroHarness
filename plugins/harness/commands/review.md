---
description: 현재 브랜치/diff를 code-reviewer agent에 위임해 검토받기
---

`code-reviewer` subagent를 호출해 현재 브랜치의 변경사항을 검토하세요.

검토 대상: $ARGUMENTS (없으면 현재 브랜치의 staged + unstaged 변경)

agent는 4원칙(Think Before Coding / Simplicity First / Surgical Changes / Goal-Driven Execution)을 기준으로 검토하고, 응답은 앵커 → 본문 → 원칙 체크 형식을 따릅니다.
