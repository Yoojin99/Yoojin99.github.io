
# Middleware concept: (req, res, next) => {}

Express 서버의 핵심 엔진.

## Argument

* `req` (Request) : Client 가 보낸 모든 정보가 담김 (URL, Header, Body, Cookie 등). 새로운 정보를 추가해서 다음 단계로 넘길 수 있음
* `res` (Response) : Client 에게 응답을 보낼 때 사용. `res.send()` / `res.json()` 을 호출하면 middleware 체인이 종료
* `next` (Next function) : 자신이 할 일 끝났으니 다음 middleware 로 가라는 뜻. 호출 안 할 경우 요청은 다음 단계로 못 넘어가고 pending 상태에 빠짐

```ts
const authChecker = (req, res, next) => {
  const password = req.query.pw;

  if (password === '1234') {
    next(); // 통과! 다음 단계(라우터)로 진행
  } else {
    res.status(401).send('비밀번호가 틀렸습니다.'); // 여기서 끝! next()를 부르지 않음
  }
};
```

# Built-in middleware: express.json()

Express 에서 가잔 많이 사용하는 built-in middleware. `req.body` 를 읽기 위해 반드시 필요함

Client 가 Server 에 JSON 데이터를 보낼 때 실제 데이터는 HTTP 의 Body 에 실려 옴. 서버는 데이터가 JSON / 단순 텍스트인지 처음에 알지 못하고, chunk 단위로 들어오기 때문에 즉시 읽을 수 없음

* 설정 전 : `req.body` 를 출력하면 `undefined` 가 나옴
* 설정 후 : `req.body` 에 보낸 JSON 데이터가 객체 형태로 담겨 있음

## 동작 방법

들어오는 요청의 헤더 먼저 봄

1. 헤더 확인 : 요청 헤더의 `Content-Type` 이 `application/json`인지 확인
2. 데이터 조립 : chunk 로 들어오는 body 데이터를 모두 모음
3. Parsing : 모인 데이터를 `JSON.parse()` 를 사용해서 JS 객체로 바꿈
4. 할당 : 변환된 객체를 `req.body` 에 넣고 `next()` 를 ㅠㅗ출

## 사용

```ts
import express from 'express';
const app = express();

// 💡 모든 요청에 대해 JSON 해석기를 가동합니다.
app.use(express.json());

app.post('/login', (req, res) => {
  // express.json() 덕분에 바로 사용할 수 있습니다.
  const { email, password } = req.body; 
  console.log(`로그인 시도: ${email}`);
  res.send('요청 수신 완료');
});
```

# CORS middleware

CORS (Cross-Origin Resource Sharing) Middleware : 웹 보안의 가장 큰 장벽 중 하나인 CORS 정책을 해결하기 위해 사용

Web broswer 의 기본 보안 원칙인 SOP(Same-Origin Policy, 동일 출처 정책) 때문에, 다른 도메인에서 우리 서버로 데이터를 요청시 브라우저가 이를 차단. 이를 허용해주는 출입증 역할을 함

## CORS 가 발생하는 상황

* FE : `http://localhost:3000`(React 등)
* BE: `http://localhost:5000` (Express)

위 둘은 domain/port 가 다르기 때문에 서로 다른 origin 으로 간주됨. 브라우저는 보안상 이유로 FE 가 BE 에 요청하는 걸 막아버리는데, 서버에서 '이 FE 는 내가 허용한 애' 라는 것을 알려줘야 함

## 사용

`cors` 패키지

### 설치

```sh
npm install cors
# TypeScript라면 타입 정의도 설치
npm install -D @types/cors
```

### 모든 요청 허용

보안 크게 중요하지 않거나, 개발 초기 단계에서 사용

```ts
import express from 'express';
import cors from 'cors';

const app = express();

// 💡 모든 도메인에서의 요청을 허용합니다.
app.use(cors()); 

app.get('/data', (req, res) => {
  res.json({ message: "CORS 해결 완료!" });
});
```

### 특정 도메인만 허용

```ts
const corsOptions = {
  origin: 'http://localhost:3000', // 허용할 도메인
  methods: ['GET', 'POST', 'PUT', 'DELETE'], // 허용할 HTTP 메서드
  credentials: true, // 쿠키나 인증 헤더를 허용할지 여부
};

app.use(cors(corsOptions));
```

### 원리

브라우저는 실제 요청을 보내기 전, Preflight(예비 요청) 을 먼저 보냄. (HTTP method 는 `OPTIONS` 를 사용)

1. 브라우저 : 서버야, 나 `localhost:3000` 인데 `POST 요청 보내도 되니? (`OPTIONS` 요청)
2. CORS middleware : ㅇㅇ, 응답헤더 `Access-Control-Allow-Origin` 을 설정해줌
3. 브라우저 : 실제 데이터를 담은 요청을 보냄

# Error handling middleware

서버 실행 중 발생하는 온갖 에러들을 한 곳에 모아 예쁘게 정리해서 Client 에 보내는 역할

매개변수가 4개.

## 구조 : `(err, req, res, next) => {}`

Express 는 argument 가 4개인 것을 보고 에러를 처리하는 애라는 걸 앎

```ts
app.use((err: any, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack); // 에러 내용을 서버 콘솔에 출력
  res.status(500).json({
    message: "서버에서 문제가 발생했습니다!",
    error: err.message
  });
});
```

* `err`: 이전 단계(router/middleware) 에서 넘어온 에러 개게

## 에러 보내기

일반 router / middleware 에서 `next()` 함수에 parameter 를 넣어서 호출하면 express 는 즉시 중간에 있는 모든 router 를 건너뛰고 error handling middleware 로 점프

```ts
app.get('/users/:id', (req, res, next) => {
  try {
    const user = findUser(req.params.id);
    if (!user) {
      // 1. 에러를 만들어서 던집니다.
      const error = new Error("유저를 찾을 수 없어요.");
      (error as any).status = 404;
      throw error; 
    }
    res.json(user);
  } catch (err) {
    // 2. catch 문에서 next(err)를 호출하면 에러 미들웨어로 이동!
    next(err); 
  }
});
```

## 써야 하는 이유

1. 중복 코드 제거 : 모든 router 마다 `res.status(500).json` 코드를 작성할 필요 없음. 모든 에러 처리를 서버 파일 하단에 한 번만 적어두면 됨
2. 보안성 : 에러 발생시 서버 내부 코드 구조 / DB 정보 노출되면 위험. Error middleware 사용시 사용자에게는 문제가 발생했다는 메세지만 보여주고, 상세 에러는 서버 로그에만 남길 수 있음
3. 중앙 집중 관리

```ts
// 1. 일반 라우터들...
app.get('/', (req, res) => { /* ... */ });

// 2. 404 처리 미들웨어 (위에서 매칭 안 된 경우)
app.use((req, res, next) => {
  res.status(404).send("존재하지 않는 페이지입니다.");
});

// 3. 최종 에러 핸들러 (맨 마지막!)
app.use((err, req, res, next) => {
  const status = err.status || 500;
  res.status(status).json({
    status: "error",
    message: err.message || "Internal Server Error"
  });
});
```

# Input validation with zod (npm install zod)

Zod : 데이터를 처리하기 전에 '이 데이터가 내가 원하는 형식이 맞나?' 를 검사함, 서버의 안정성, 보안에 필수. TS 와 궁합이 잘 맞는 스키마 선언, 유효성 검사 라이브러리

## 사용하는 이유

1. 타입 추론 : Schema 를 정의하면 TS 타입을 따로 만들 필요 없이 자동으로 추출할 수 있음
2. 간결함 : `if/else` 수십 줄을 단 몇 줄로 끝냄
3. Runtime 체크 : TS 의 타입 시스템은 빌드 후 사라지지만, Zod 는 실제 데이터가 들어오는 순간 (runtime) 에 잡아냄

## 사용법

```sh
npm install zod
```

```ts
import { z } from 'zod';

// 1. 검증하고 싶은 데이터의 '설계도(Schema)'를 정의합니다.
const UserSchema = z.object({
  id: z.number(),
  name: z.string().min(2, "이름은 최소 2글자 이상이어야 합니다."),
  email: z.string().email("올바른 이메일 형식이 아닙니다."),
  age: z.number().int().positive().optional(), // 선택 사항
});

// 2. TypeScript 타입을 스키마로부터 자동으로 추출합니다. (중복 정의 불필요!)
type User = z.infer<typeof UserSchema>;

// 3. 데이터 검증 실행
const result = UserSchema.safeParse({
  id: 1,
  name: "G",
  email: "invalid-email"
});

if (!result.success) {
  // 에러 발생 시 상세한 에러 내용 확인 가능
  console.log(result.error.format()); 
}
```

### Express Middleware 로 Zod 활용하기

실무에서는 middleware 형태로 만들어서 router 도착 전 데이터를 미리 걸러냄

```ts
import { Request, Response, NextFunction } from 'express';
import { z, ZodError } from 'zod';

// 유효성 검사를 위한 고차 미들웨어
const validate = (schema: z.AnyZodObject) => 
  (req: Request, res: Response, next: NextFunction) => {
    try {
      // req.body를 스키마에 맞춰 검사하고 정제함
      schema.parse(req.body);
      next();
    } catch (error) {
      if (error instanceof ZodError) {
        // Zod 에러가 나면 400 Bad Request와 함께 상세 에러 응답
        return res.status(400).json({
          status: 'fail',
          errors: error.errors.map(e => ({ path: e.path, message: e.message }))
        });
      }
      next(error);
    }
  };

// 적용 예시
const RegisterSchema = z.object({
  username: z.string().min(4),
  password: z.string().min(8),
});

app.post('/register', validate(RegisterSchema), (req, res) => {
  res.send("유효성 검사 통과! 회원가입을 진행합니다.");
});
```

### Zod 의 강력한 기능들

* `.strict()` : 스키마에 정의되지 않은 다른 필드가 들어오면 에러를 냄
* `.default()` : 값이 없을 때 기본값을 채워줌
* `.refine()` : 커스텀 검증 로직을 추가할 때 사용