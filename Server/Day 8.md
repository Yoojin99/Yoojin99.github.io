
# What JWT is

JWT (JSON Web Token) : 당사자 간의 정보를 JSON 객체로 안전하게 전송하기 위한 개방형 표준 (RFC 7519)

서버가 사용자에게 발급해주는 디지털 통행증 같은 느낌

## 왜 쓰는지

* Session : 서버가 '누가 로그인했는지' 를 메모리 / DB 에 적어두고 기억해야 함
* JWT : 서버는 기억하지 않고, 사용자에게 정보를 담은 증명서를 줌. 사용자가 증명서를 들고 오면 서버는 서명만 확인하고 통과시킴

## JWT 구조

세 토막 문자열. `aaa.bbb.ccc` 로 점으로 구분된 세 부분

1. Header

어떤 종류의 토큰인지 (typ: JWT), 어떤 암호화 알고리즘 (alg: HS256) 등을 썼는지 담고 있음

2. Payload

실제 정보가 들어있는 곳. 유저 ID, 이름, 권한, 토큰 만료 시간 등을 담음. 누구나 볼 수 있기 때문에 비밀번호같은 민감 정보는 절대로 넣으면 안됨

3. Signature

토큰이 도중에 조작되지 않았는지 확인하는 가장 중요한 부분.

`Header + Payload + 서버의 Secret Key` 를 합쳐서 만듦. 서버만 알고 있는 비밀키가 있어야만 올바른 서명을 만들 수 있음

## JWT 인증 과정

1. 로그인 : 클라이언트가 아이디/비번을 보냄
2. 발급 : 서버가 확인 후, 유저 정보를 담은 JWT를 서명해서 응답
3. 저장 : 클라이언트는 토큰을 브라우저에 저장
4. 요청 : 이후 클라이언트는 API 를 호출할 때마다 HTTP Header 에 토큰 실어 보냄 (`Authorization: Bearer <token>`)
5. 검증 : 서버는 토큰의 서명만 체크, 유효하면 요청을 허용

## JWT의 장단점

* 장점
	* Stateless : 서버가 유저 상태를 저장할 필요가 없어 서버 확장이 유리
	* 모바일 최적화 : 쿠키를 쓰기 힘든 모바일 앱에서 잘 작동
* 단점
	* 한 번 발급하면 통제 불가 : 유효기간 끝나기 전까지 강제로 로그아웃 시키는 것이 어려움. 이를 보완하기 위해 `Access/Refresh Token` 전략을 씀
	* 데이터 크기 : 담는 정보가 많아질수록 토큰 길이가 길어져 네트워크 비용이 생김

# Password hashing with bcrypt

DB에 사용자의 비밀번호를 저장할 때 중요한 원칙은 비밀번호 원문 Plain Text 를 그대로 저장하지 않는다는 것. DB가 해킹당하더라도 비밀번호가 유출되지 않도록 bcrypt 를 사용해 안전하게 Hashing 해야 함

## bcrypt 란

비밀번호 저장만을 위해 설계된 강력한 해시 함수

* 단방향성 : 해시된 결과값에서 원문 비밀번호를 역추적하는 것이 수학적으로 불가능
* Salt : 똑같은 비밀번호라도 해시할 때마다 매번 다른 결과값이 나오게 만듦. (rainbow table 공격 방지)
* 느린 연산 : 해싱 속도를 의도적으로 조절해서 해커가 무작위로 대입하는 brute force 공격을 어렵게 만듦

## 설치 및 사용

```sh
npm install bcrypt
npm install -D @types/bcrypt
```

1. 비밀번호 암호화 (Signup 할 때)

사용자가 회원가입할 때 비밀번호를 hashing 해서 DB 에 저장

```ts
import bcrypt from 'bcrypt';

const password = 'mySecretPassword123';
const saltRounds = 10; // 해싱 강도 (보통 10~12 권장)

const hashedPassword = await bcrypt.hash(password, saltRounds);
// 결과 예: $2b$10$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6L6s57Wy6UunmMei
```

2. 비밀번호 검증 (Login 시)

로그인할 때는 입력받은 비밀번호를 다시 hashing 해서 비교하지 않고 `compare` 함수 사용.

```ts
const isMatch = await bcrypt.compare(password, hashedPassword);

if (isMatch) {
  console.log("로그인 성공!");
} else {
  console.log("비밀번호가 틀렸습니다.");
}
```

3. 실 사용 코드

```ts
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcrypt';

const prisma = new PrismaClient();

async function createUser(email: string, password: string) {
  // 1. 비밀번호 해싱
  const hashedPassword = await bcrypt.hash(password, 10);

  // 2. DB에 저장
  return await prisma.user.create({
    data: {
      email,
      password: hashedPassword, // 암호화된 비밀번호 저장
    },
  });
}
```

# Auth flow: register → hash password → save / login → compare → return JWT

1. Register : 사용자가 비밀번호를 입력하면 서버는 암호화해서 DB 에 저장

```ts
// POST /auth/register
async function register(req, res) {
  const { email, password } = req.body;

  // 1. 비밀번호 해싱 (bcrypt)
  const saltRounds = 10;
  const hashedPassword = await bcrypt.hash(password, saltRounds);

  // 2. DB에 저장 (Prisma)
  const newUser = await prisma.user.create({
    data: {
      email,
      password: hashedPassword, // 원문이 아닌 해시값 저장
    },
  });

  res.status(201).json({ success: true, message: "User registered" });
}
```

2. 로그인 : 입력받은 비밀번호와 db 의 hash 값 비교한 뒤, 일치하면 JWT 발급

```ts
// POST /auth/login
async function login(req, res) {
  const { email, password } = req.body;

  // 1. 이메일로 유저 찾기
  const user = await prisma.user.findUnique({ where: { email } });
  if (!user) return res.status(404).json({ error: "User not found" });

  // 2. 비밀번호 비교 (bcrypt.compare)
  // 입력된 password를 해싱해서 DB의 user.password와 비교함
  const isMatch = await bcrypt.compare(password, user.password);
  
  if (!isMatch) return res.status(401).json({ error: "Invalid password" });

  // 3. JWT 발급
  const token = jwt.sign(
    { userId: user.id, email: user.email }, // Payload (데이터)
    process.env.JWT_SECRET,                 // Secret Key (서명용)
    { expiresIn: '1h' }                     // 유효 기간
  );

  // 4. 토큰 반환
  res.status(200).json({ success: true, token });
}
```