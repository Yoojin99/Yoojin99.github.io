
# REST conventions: plural nouns, proper status codes, consistent response

REST(Representational State Transfer) 는 웹의 장점(HTTP) 를 최대한 활용하기 위한 API 설계 가이드라인.

서버-client 가 서로 데이터를 주고받을 때 '이런 규칙으로 대화하면 이해하기 쉽다'는 전 세계적인 약속

## REST 핵심 3요소

[noun]을 [verb] 한다

1. Resource : 우리가 다루는 데이터. URL 로 표현
2. Verb : 데이터를 어떻게 할 것인가? HTTP Method 로 표현
3. Representation : 데이터를 어떤 형식으로 보낼 것인가? 주로 JSON

## RESTful 한 설계를 위한 원칙

1. Client-Server 구조 : 서버는 데이터만 주고, client 는 화면만 그림. 서로의 역할이 명확히 나뉘어야 함
2. Stateless : 서버는 클라의 상태를 기억하지 않음. 각 요청은 그 자체로 필요한 모든 정보를 담고 있어야 함
3. Cacheable : HTTP 라는 기존 웹 인프라는 그대로 쓰기 때문에 자주 바뀌지 않는 데이터는 브라우저/중간 서버에 캐싱함
4. Uniform Interface : URL 만 보고도 무엇을 하려는지 알 수 있어야 함

## 협업의 효율을 결정짓는 규칙

### Plural Nouns : 리소스는 항상 복수형으로

URL 경로는 행위가 아닌 **데이터(명사)** 를 표현해야 함. 리소스는 collection 을 의미하므로 복수형 사용이 표준

|**행위**|**안 좋은 예 (Single/Verb)**|**좋은 예 (Plural)**|
|---|---|---|
|**전체 조회**|`GET /getUser`|`GET /users`|
|**단일 조회**|`GET /user/1`|`GET /users/1`|
|**생성**|`POST /createUser`|`POST /users`|
|**삭제**|`DELETE /user/delete/1`|`DELETE /users/1`|

### Proper Status Codes : 적절한 상태 코드 반환

HTTP status code 는 server 가 client 에 보내는 **결과 요약**. 상황에 맞는 코드를 써서 client 가 대응할 수 있게 하기

|**코드**|**이름**|**상황**|
|---|---|---|
|**200**|**OK**|요청 성공 (조회, 수정 등)|
|**201**|**Created**|새로운 리소스 생성 성공 (**POST** 결과)|
|**204**|**No Content**|성공했으나 돌려줄 데이터가 없음 (**DELETE** 결과)|
|**400**|**Bad Request**|클라이언트 입력값 오류 (Zod 검증 실패 등)|
|**401**|**Unauthorized**|인증되지 않은 사용자 (로그인 필요)|
|**403**|**Forbidden**|권한이 없는 리소스 접근 (관리자 전용 등)|
|**404**|**Not Found**|요청한 리소스가 존재하지 않음|
|**500**|**Internal Server Error**|서버 내부 버그 (DB 연결 끊김 등)|

### Consistent Response : 일관된 응답 구조

성공, 실패 시의 구조를 통일

```ts
app.get('/users/:id', async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: Number(req.params.id) }
  });

  if (!user) {
    // 1. 적절한 상태 코드 (404)
    // 2. 일관된 응답 구조
    return res.status(404).json({
      success: false,
      data: null,
      error: "User not found"
    });
  }

  // 성공 시 200 OK와 데이터 반환
  res.status(200).json({
    success: true,
    data: user,
    error: null
  });
});
```

# Environment variables with dotenv (npm install dotenv)

Environment variable : db 비밀번호, API 키와 같이 코드에 직접 노출되면 안 되는 민감한 정보, 실행 환경마다 달라지는 설정을 관리하기 위해 사용

`dotenv` 라이브러리 : 프로젝트 루트에 있는 `.env` 파일의 내용을 Node.js 의 `process.env` 객체로 로드해주는 도구

## 1. 설치 및 초기 설정

```sh
npm install dotenv
```

root directory 에 `.env` 파일 생성 후 보안이 필요한 변수들 정의. `=` 앞뒤로 공백이 없어야 함

```
PORT=3000
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
JWT_SECRET="super-secret-key"
```

## 2. 코드에서 사용하기

진입점 (e.g. `index.ts` / `app.ts`) 최상단에서 `dotenv` 설정

```ts
import dotenv from 'dotenv';
// .env 파일의 내용을 process.env에 로드합니다.
dotenv.config();

// 이제 어디서든 process.env로 접근 가능합니다.
const port = process.env.PORT || 4000;
const dbUrl = process.env.DATABASE_URL;

console.log(`서버가 ${port}번 포트에서 실행 중입니다.`);
```

## 3. `.gitignore` 설정

절대로 `.env` 파일은 git 저장소에 올리면 안됨. `.gitignore` 파일에 `.env` 추가.

대신 팀원들이 어떤 변수가 필요한지 알 수 있도록 `.env.example` 파일을 만들어 변수명만 공유하는 것이 관례

`.env.example`

```
PORT=3000
DATABASE_URL=""
JWT_SECRET=""
```

## 4. Prisma 와 `.env`

Prisma 는 기본적으로 `dotenv` 기능을 내장하고 있음

`npx prisma` 명령어 실행시 자동으로 `.env` 파일 읽음. 

## 5. TypeScript 에서 환경 변수 타입 안전하게 쓰기

`process.env.PORT` 를 호출하면 TS 는 이 값이 있는지 없는지 몰라서 `string | undefined` 타입으로 추론. 안전하게 사용하기 위해서 Zod 활용 가능

```ts
import { z } from 'zod';

const envSchema = z.object({
  PORT: z.string().default("3000"),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string(),
});

// 환경 변수 검증
const env = envSchema.parse(process.env);

export const ENV = env; // 이제 ENV.PORT는 안전하게 사용 가능!
```