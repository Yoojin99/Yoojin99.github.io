
# Express: routing, middleware

Express : Node.js 를 위한 가장 인기 있고 표준적인 web framework.

`node:http` 는 '원시 재료', Express 는 똑같은 서버를 훨씬 더 적은 코드로 만들 수 있게 해줌

|**특징**|**Native (node:http)**|**Express**|
|---|---|---|
|**코드 양**|매우 많고 복잡함|간결하고 읽기 쉬움|
|**라우팅**|직접 `if/else`로 구현|`app.get()`, `app.post()` 제공|
|**데이터 처리**|스트림으로 직접 구현|미들웨어로 자동 처리|
|**확장성**|라이브러리 연동이 번거로움|방대한 플러그인(Middleware) 생태계|

## Express 가 해결해주는 문제들

### 간결한 Routing

* `node.http` : `if (method === 'GET' && url === '/user')`

대신 직관적인 메서드 제공

```ts
app.get('/user', (req, res) => {
	res.send('GET 요청 응답');
})
```

### 편리한 body parsing

`req.on('data')` / `req.on('end')` 를 쓰며 stream 을 직접 모을 필요가 없고, middleware 를 쓰면 `req.body` 로 바로 읽을 수 있음

### Middleware 사용

요청이 서버에 도착해서 response 가 나갈 때까지 필요한 작업들(log 기록, 인증 체크, 에러 처리)을 조립식으로 끼워 넣을 수 있음

## Routing

Client 의 request URL 과 HTTP method (GET, POST, ...) 등에 따라 어떤 함수를 실행할지 결정하는 매커니즘

* Express 구조 : `app.METHOD(PATH, HANDLER)`
	* METHOD : HTTP request method
	* PATH : 서버 경로 (e.g. `/users`, `/login`)
	* HANDLER : 해당 경로로 요청이 들어왔을 때 실행될 함수

```ts
// '/products' 경로로 GET 요청이 오면 이 라우트가 실행됨
app.get('/products', (req, res) => {
  res.send('상품 목록입니다.');
});

// '/products' 경로로 POST 요청이 오면 이 라우트가 실행됨
app.post('/products', (req, res) => {
  res.send('상품을 등록했습니다.');
});
```

## Middleware

중간 공정 과정. **Request, response 사이에서 특정 작업을 수행하는 함수**. Request 가 목적지 (routing handler) 에 도착하기 전에 거쳐가는 곳.

* 하는 일 : 코드 실행, request/response 객체 수정, 다음 middleware 로 넘기거나 응답 종료
* `next()` : 현재 middleware 가 작업을 마쳤으니 다음 곳으로 가라라는 함수

### 종류

* Application middleware : 모든 요청에서 실행 (e.g. log 기록, JSON 파싱)
* Router middleware : 특정 경로에서만 실행 (e.g. 결제 페이지 접근 전 로그인 체크)
* Error handling middleware : 서버 에러 발생 시 처리

```ts
// 모든 요청의 시간을 기록하는 middleware
app.use((req, res, next) => {
	console.log(`요청 시간: ${new Date()}`);
	next(); // 이걸 안 하면 다음 단계로 못 넘어감!!
});
```

## Routng + Middleware

일반적으로 login 체크 -> profile 조회

```ts
// 1. middleware : 로그인 확인 함수
const checkLogin = (req, res, next) => {
	const isLoggedIn = true; // 실제로는 토큰 등을 확인
	
	if (isLoggedIn) {
		next(); // 로그인 됐으면 다음(route)으로 이동
	} else {
		res.status(401).send('로그인이 필요합니다.');
	}
};

// 2. routing : mypage 조회, middleware 를 거쳐야만 도달 가능
app.get('/mypage', checkLogin, (req, res) => {
	res.send('반갑습니다! 당신의 프로필입니다.');
});
```

## 굳이 Middleware 가 있는 이유

내 생각 : 굳이 middleware 를 둔 이유가 있나? 로직 자체를 routing 하는 곳에 포함시켜도 될 거 같고, 그냥 보기에는 단순히 로직의 재사용성을 위함인것 같은데 그냥 일반 함수를 분리하면 똑같은 것이 아닐까? middleware 은 왜 만들어진 것인가?

1. 제어권의 위임과 흐름 제어 (`next()`) : 일반 함수는 호출하면 값을 return 하고 끝나지만, 미들웨어는 다음 단계로 진행할지, 응답을 종료할지에 대한 결정권을 가짐
2. 수평적 확장 (파이프라인 구조) : router 는 앞서 실행된 middleware 들이 req 에 미리 준비해둔 데이터를 가져다 쓰기만 하면 됨
3. Global Application : 수백 개의 router 가 있는 대형 프로젝트에서 모든 요청에 대해 로그를 남겨야 한다면?
	1. 일반 함수 : 수백 개의 router 에서 함수를 호출해야 함
	2. Middleware : `app.use(myMiddleware)` 한 줄로 모든 router 에 자동 적용

# app.get(), app.post(), app.put(), app.delete()

Express 에서 HTTP method - 특정 경로 (URL) 을 연결하는 routing 함수

## `app.get(path, callback)`

데이터를 `req.query`/ `req.param` 으로 받음

```ts
app.get('/users', (req, res) => {
  // DB에서 유저 목록을 가져와서 응답
  res.json([{ id: 1, name: 'Kim' }, { id: 2, name: 'Lee' }]);
});
```

## `app.post(path, callback)`

데이터를 주로 `req.body` 에 담아서 받음

```ts
app.post('/users', (req, res) => {
  const newUser = req.body; // { name: 'Park' }
  // DB에 저장 로직...
  res.status(201).json({ message: '생성 완료', user: newUser });
});
```

## `app.put(path, callback)`

수정할 데이터의 전체 내용을 보내야 함

```ts
app.put('/users/:id', (req, res) => {
  const { id } = req.params;
  const updatedData = req.body;
  // id에 해당하는 데이터를 updatedData로 통째로 갈아끼움
  res.json({ message: `${id}번 유저 정보가 수정되었습니다.` });
});
```

## `app.delete(path, callback)`

```ts
app.delete('/users/:id', (req, res) => {
  const { id } = req.params;
  // DB에서 해당 id 삭제 로직...
  res.json({ message: `${id}번 유저가 삭제되었습니다.` });
});
```

# Route parameters /users/:id

`users/:id` 에서 `:id` 같은 형태를 Route Parameter 라고 함. URL 의 특정 부분을 변수로 처리하겠다는 약속. `req.params` 객체에 담겨짐

```ts
app.get('/users/:id', (req, res) => {
	const userId = req.params.id;
	
	res.send(`${userId}번 사용자의 정보를 조회 중입니다.`);
});
```

## vs Query Parameter

* Route parameter `/users/123`
	* 식별할 때 사용
	* 데이터 자체를 가리키는 고유한 주소처럼 사용됨
	* 특정 유저, 특정 게시글, 특정 상품 페이지
* Query parameter `/users?sort=age`
	* 정렬, 필터링, 검색할 때 사용
	* 데이터의 목록을 보여줄 때 옵션을 주는 용도
	* "나이순으로 정렬"

## 여러 개 사용

```ts
// 예: "1번 블로그"의 "5번 게시글"
// URL: /blogs/1/posts/5
app.get('/blogs/:blogId/posts/:postId', (req, res) => {
  const { blogId, postId } = req.params;
  res.send(`${blogId}번 블로그의 ${postId}번 글입니다.`);
});
```

## 선언 순서

Route parameter 는 '아무 값이나 다 먹어치우는 성질' 있음. 구체적인 경로보다 뒤에 작성해야 함

```ts
// 1. 구체적인 경로를 위에!
app.get('/users/me', (req, res) => {
  res.send("내 프로필입니다.");
});

// 2. 파라미터 경로를 아래에!
app.get('/users/:id', (req, res) => {
  res.send(`${req.params.id}번 유저입니다.`);
});
```

# req.params, req.query, req.body

Express 에서 Client 가 보낸 데이터를 받아오는 3가지 핵심 경로

## req.params (Path Variables)

URL 경로의 일부가 데이터인 경우

Route 경로에 설정한 colon `:` 자이에 들어오느 값. 특정 리소스를 식별할 때 사용

* URL 형태 : `/users/123`
* 서버 설정 : `app.get('/users/:id', ...)`
* 결과 `{ id: '123' }`

```ts
app.get('/products/:category/:id', (req, res) => {
  const { category, id } = req.params;
  res.send(`${category} 카테고리의 ${id}번 상품을 조회합니다.`);
});
```

## req.query (Query Parameters)

URL 끝에 `?` 뒤에 붙여서 보내는 경우

데이터를 필터링, 정렬, 검색할 때 사용. key=value 쌍으로 이어져 있음

* URL 형태 : `/search?keyword=apple&page=2`
* 서버 설정 : 별도의 경로 설정 없이 `app.get('/search', ...)`
* 결과 : `{ keyword: 'apple', page: 2 }`

```ts
app.get('/posts', (req, res) => {
  const { sort, limit } = req.query;
  // 예: /posts?sort=desc&limit=10
  res.send(`${limit}개의 게시물을 ${sort} 순으로 정렬합니다.`);
});
```

## req.body (Request Body)

요청 본문에 숨겨서 보내는 대용량/민감 데이터

POST, PUT, PATCH 메서드에서 사용, JSON 객체/데이터 주고받을 때 사용. URL 에 노출 안 됨

* 데이터 형태 : `{ "name" : "Alice", "age" : 20 }`
* 주의사항 : 반드시 `app.use(express.json())` 같은 middleware 를 설정해야 읽을 수 있음
* 결과 : `{ "name" : "Alice", "age" : 20 }`

```ts
app.post('/register', (req, res) => {
  const { name, email } = req.body;
  res.json({ message: `${name}님, 회원가입을 환영합니다!` });
});
```

### 왜 middleware 를 설정해야만 읽을 수 있는가

`req.params`, `req.query` 는 URL 이라는 한정된 문자열 안에 들어가지만 `req.body` 는 데이터의 양, 종류가 차원이 다르기 때문

1. 데이터가 stream 으로 들어오기 때문 : HTTP 요청에서 body 데이터는 한 덩어리가 한 번에 들어오는 게 아니라 네트워크를 통해 chunk 들로 나눠서 들어옴
	* `node:http` : `req.on('data')`, `req.on('end')` 이벤트를 직접 구현해서 chunk 를 이어 붙어야 함
	* middleware : `express.json()` 같은 middleware 가 chunk 를 모아주는 과정을 대신 해줌
2. 데이터의 형식이 다양 : URL 은 무조건 문자열이지만 body 에는 다양한 데이터가 들어올 수 있음. 해석기가 필요.
3. 효율성과 보안

# res.json(), res.status(), res.send()

Client 에게 응답을 보낼 때 사용하는 핵심 메서드

## res.send()

범용 응답 도구. 가장 기본이 되는 메서드. 문자열, HtML, 객체, 배열 등 알아서 판단해서 보내줌

* 특징 : 데이터 타입에 맞춰 `Content-Type` 헤더를 자동으로 설정 (e.g. 문자열 `text/html`, 객체 `application/json`)
* 간단한 텍스트 / HTML 조각을 응답할 때 사용

```ts
res.send('<h1>안녕하세요!</h1>'); // 브라우저는 HTML로 인식
res.send({ message: 'Hello' });  // 브라우저는 JSON으로 인식
```

## res.json()

JSON 전용 응답 도구. 

* 특징
	1. 전달된 인자를 `JSON.stringify()` 를 통해 문자열로 바꿈
	2. `Content-Type` 을 무조건 `application/json` 으로 고정
	3. 인자가 `null` / `undefined` 여도 JSON 규격에 맞게 변환하려 노력
* API 서버를 만들 때 가장 많이 사용

```ts
// res.send()로 객체를 보내는 것과 비슷해 보이지만, 
// API 서버라면 "난 무조건 JSON만 보내!"라는 의도를 명확히 하기 위해 이걸 씁니다.
res.json({ id: 1, name: 'Gemini' });
```

## res.status()

상태 코드 설정도구, 전송은 안 함. 응답의 status code 를 지정. 뒤에 `send()` `json()`을 붙여야만 response 가 정송됨

* 특징 : Method Chaining 을 지원

```ts
// 201(Created) 코드를 설정하고 JSON 데이터를 보냄
res.status(201).json({ message: '회원가입 성공' });

// 404(Not Found) 코드를 설정하고 에러 메시지를 보냄
res.status(404).send('페이지를 찾을 수 없습니다.');
```

⚠️ 하나의 router 안에서 응답 메서드를 두 번 호출하면 서버가 터짐. 응답은 딱 한 번만.

Read: expressjs.com/en/starter/hello-world through "Basic Routing"

# 실습

Express 설치

```sh
npm init -y
npm install express
npm install -D typescript @types/node @types/express ts-node

npx tsc --init
```