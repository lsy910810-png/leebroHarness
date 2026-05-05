---
description: principle-auditor agent로 4원칙 위반을 코드베이스에서 스캔
---

`principle-auditor` subagent를 호출해 현재 코드베이스에 대한 원칙 audit을 실행하세요.

스코프: $ARGUMENTS (없으면 git이 추적하는 모든 소스 파일)

agent는 4가지 차원(긴 함수/파일·스코프 완화·검증 부재·vague 코멘트)을 스캔하고 punch list로 보고합니다. 자동 수정은 안 합니다.
