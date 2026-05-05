# Secrets Handling

API 키·토큰·비밀번호·암호화 키가 코드/로그/응답으로 새지 않게 점검합니다.

## 위험 패턴

### 하드코딩
```js
// BAD
const API_KEY = "sk_live_a1b2c3..."
const dbPassword = "admin1234"
const stripeSecret = "sk_test_xxxxx"
```

### .env 파일 commit
```
// BAD: .env가 .gitignore에 없거나, .env.example에 실제 값 들어감
git add .env
```

### 로그·에러 메시지 누출
```js
// BAD
console.log("auth header:", req.headers.authorization)
logger.error(`Failed: ${err.stack}`)  // err.stack에 token 포함 가능
res.json({ debug: { config: process.env } })  // 응답에 env dump
```

### URL/query string에 비밀값
```js
// BAD: ?token=xxx 는 server log·browser history·referer header에 남음
fetch(`/api?api_key=${KEY}`)
```

### Git history에 남은 키
- 한 번 commit된 key는 force-push로 지워도 fork·clone에 남음 → **rotate 필수**

## 안전 패턴

```js
// 환경변수 사용
const API_KEY = process.env.STRIPE_SECRET
if (!API_KEY) throw new Error('STRIPE_SECRET not set')

// 헤더에 비밀값
fetch('/api', { headers: { Authorization: `Bearer ${KEY}` } })

// 로그에서 마스킹
const masked = key.slice(0, 4) + '...' + key.slice(-4)
logger.info({ apiKey: masked })
```

## grep 키워드 (코드 스캔용)

### 하드코딩 의심
- `(?i)(api[_-]?key|secret|password|token|access[_-]?key)\s*[:=]\s*["'][\w-]{16,}["']`
- AWS: `AKIA[0-9A-Z]{16}`, `aws_secret_access_key`
- GitHub: `ghp_[a-zA-Z0-9]{36}`, `gho_`, `ghs_`
- Stripe: `sk_(live|test)_[a-zA-Z0-9]{24,}`
- Slack: `xox[baprs]-[a-zA-Z0-9-]+`
- JWT: `eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+`
- private key: `-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY-----`

### 환경변수 잘못 노출
- `process.env`가 응답·로그·클라이언트 번들로 가는지
- Next.js: `NEXT_PUBLIC_` prefix는 클라이언트 노출이라는 걸 인지하고 있는지

## 점검 체크리스트

- [ ] `.env*` 파일이 `.gitignore`에 있는가? `git check-ignore .env`
- [ ] `git log --all -p | grep -E '(api[_-]?key|secret)'` 으로 history 스캔
- [ ] CI 로그에 secret이 echo되지 않는가? (build script `set -x` 주의)
- [ ] error stack trace를 client에 그대로 보내지 않는가?
- [ ] 비밀값을 query string으로 받지 않는가?
- [ ] 서드파티 SDK가 자체적으로 로깅하는 값을 검토했는가? (예: axios interceptor가 헤더 출력)

## 발견 시 보고 형식

```
[blocker · 원칙 4] <파일>:<라인> — Hardcoded secret: <키 종류, 마스킹된 값>
  • 노출 범위: <git history? prod log? client bundle?>
  • 즉시 조치: 키 rotate → 환경변수로 이전
  • 추가 조치: pre-commit hook(secretlint 등) 설치 권고
```
