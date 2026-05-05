# Framework Detection

테스트를 작성·실행하기 전에 프로젝트의 테스트 framework·명령어를 자동 탐지합니다.

## 탐지 순서

1. **`package.json`의 `scripts.test`** 확인 — 가장 신뢰할 수 있는 source
2. **devDependencies**에서 framework 추론
3. 1·2가 모호하면 사용자에게 묻기 (원칙 1: 추측 금지)

## Node / TypeScript

`package.json` 기준:

```json
{
  "scripts": {
    "test": "jest",                  // → jest
    "test": "vitest",                // → vitest
    "test": "node --test",           // → node:test (built-in)
    "test": "playwright test",       // → playwright (e2e)
    "test:unit": "jest --selectProjects unit",
    "test:e2e": "playwright test"
  },
  "devDependencies": {
    "jest": "...", "vitest": "...", "@playwright/test": "..."
  }
}
```

실행 명령:
- jest: `npx jest <pattern>` 또는 `npm test -- <pattern>`
- vitest: `npx vitest run <pattern>` (watch는 `vitest`)
- node:test: `node --test <file>` 또는 `node --test test/`
- playwright: `npx playwright test <file>`

설정 파일 확인: `jest.config.*`, `vitest.config.*`, `playwright.config.*`

## Python

```bash
# 우선순위
pyproject.toml: [tool.pytest.ini_options]   → pytest
pytest.ini, tox.ini                          → pytest
setup.cfg [tool:pytest]                      → pytest
manage.py + tests/ 디렉토리                  → Django: python manage.py test
unittest 패턴 (test_*.py 안에 unittest.TestCase) → unittest 또는 pytest
```

실행 명령:
- pytest: `pytest <path>` 또는 `python -m pytest <path>`
- unittest: `python -m unittest <module>`
- Django: `python manage.py test <app>`

가상환경 활성화 필요할 수 있음 (`.venv/bin/activate`, `poetry run`, `uv run`).

## Go

`go.mod` 있고 `*_test.go` 파일 있으면 native:
- `go test ./...` (전체)
- `go test ./pkg/...` (패키지)
- `go test -run TestName ./pkg`

## Rust

`Cargo.toml`:
- `cargo test` (전체)
- `cargo test <pattern>`
- `cargo test --package <pkg>`

## 모호한 경우 (원칙 1)

다음 중 하나라도 해당하면 **추측 말고 사용자에게 묻기**:
- `package.json`에 `test` script 없음
- 여러 framework 동거 (jest + vitest + playwright)
- monorepo: 어느 workspace의 테스트 돌릴지 불명
- CI에서만 돌고 로컬 명령 명확치 않음

## 탐지 결과 보고

응답에 명시:
```
## Framework
- 탐지된 framework: vitest 1.4.0 (devDependencies, vitest.config.ts 존재)
- 실행 명령: `npx vitest run <pattern>`
- 가정 (원칙 1): pattern 없으면 모든 테스트 실행 → 변경 파일만 돌리려면 `--changed`
```
