# Injection (SQL / NoSQL / Command / Template)

## SQL injection

### 위험 패턴
```js
// BAD: 문자열 결합
db.query(`SELECT * FROM users WHERE id = ${userId}`)
db.query("SELECT * FROM users WHERE name = '" + name + "'")

// BAD: ORM 우회
prisma.$queryRawUnsafe(`SELECT ... ${input}`)
```

### 안전 패턴
```js
// 매개변수 바인딩
db.query('SELECT * FROM users WHERE id = $1', [userId])
db.prepare('SELECT * FROM users WHERE name = ?').get(name)

// ORM 안전 query
prisma.user.findMany({ where: { id: userId } })
prisma.$queryRaw`SELECT * FROM users WHERE id = ${userId}`  // tagged template은 안전
```

### grep 키워드
- `query(`, `execute(`, `raw(`, `$queryRawUnsafe`
- `${`, `+ ` 옆에 SQL 키워드(`SELECT`, `WHERE`, `INSERT`, `UPDATE`, `DELETE`)
- `f"... {var} ..."` (Python f-string 안 SQL)

## NoSQL injection (MongoDB 등)

### 위험 패턴
```js
// BAD: 사용자 입력을 객체째로 query에 전달
db.users.find({ username: req.body.username, password: req.body.password })
// 공격자가 password = { $ne: null } 보내면 우회됨
```

### 안전 패턴
```js
// 타입 강제 + 명시적 비교
const username = String(req.body.username)
const password = String(req.body.password)
db.users.find({ username, password })
```

## Command injection

### 위험 패턴
```js
// BAD
exec(`convert ${userInput} output.png`)
exec('grep ' + pattern + ' file.txt')
```

### 안전 패턴
```js
// argv 배열 사용 (shell 해석 안 함)
execFile('convert', [userInput, 'output.png'])
spawn('grep', [pattern, 'file.txt'])
```

### grep 키워드
- `exec(`, `execSync(`, `child_process.exec`
- `os.system`, `subprocess.call(... shell=True ...)`
- `shell_exec`, backtick 명령 (PHP/bash)

## Template / Path injection

- 사용자 입력을 템플릿에 그대로 주입 → server-side template injection
- `path.join(userInput)` → `../../../etc/passwd` 가능. `path.resolve` 후 base 디렉토리 prefix 검사 필요

## 검증 방법

발견 시 reviewer는 다음을 확인하라고 권고:
1. 해당 함수에 **악성 입력 주입 테스트**가 있는가? (e.g., `'; DROP TABLE--`, `{ $ne: null }`)
2. 정적 분석 도구가 해당 패턴을 잡는가? (eslint-plugin-security, semgrep, bandit)
3. 프레임워크의 안전 API를 일관되게 쓰는가? (raw query는 예외적 사용으로 한정)
