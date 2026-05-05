---
description: tester agent에 위임해 변경된 코드의 테스트 작성·실행·검증
---

`tester` subagent를 호출해 다음에 대한 테스트를 작성·실행·검증하세요.

대상: $ARGUMENTS (없으면 현재 브랜치에서 변경된 파일들)

agent는 execute-verify 루프(원칙 4)를 따르고, 결과는 실행 명령 + 출력을 응답 종료 체크에 첨부합니다.
