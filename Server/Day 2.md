
# Node.js 

Node.js 는 JS 코드를 browser 밖에서 실행할 수 있게 해주는 런타임 환경. 서버 측에서 JS 코드를 실행할 수 있는 오픈소스, 크로스플랫폼 런타임 환경

## v8 엔진

V8 은 JS 코드를 실행하는 엔진. Google 이 Chrome browser 에서 JS 를 실행하려고 만들었음. 

NodeJS 는 V8을 사용해서 컴퓨터에서 JS 를 실행함.

* Google 이 개발한 오픈소스
* C++ 로 구성, 개발
* Nodejs 런타임, Chrome browser 에서 사용됨

JavaScript 코드를 받아서 compile, 실행하는 엔진.

### Interpret vs Compile

|구분|Interpret|Compile|
|-|-|-|
|번역|한 줄씩 읽고 실행|전체를 번역 후 한 번에 실행|
|장점|코드 전체가 compile 되기까지 기다릴 필요 없이 한 줄 변환하기 때문에 실행 속도 빠름|효율적인 실행 코드가 생성됨|
|단점|실행시 interpreter 필요|기계어에 종속적인 실행코드 생성되므로 실행 기계가 달라지면 새로 compile 해야 함|

1. JIT Compilation

과거의 JS 는 interpreter 방식이라 느렸음. V8 은 JIT (Just-In-Time) compilation 사용해서 JS 코드 실행. JIT 는 코드를 실행하는 시점에 JS 를 machine code 로 직접 compile 해서 CPU 가 바로 이해할 수 있는 코드로 바꾸기 때문에 속도가 빠름

2. Optimization

V8 은 자주 쓰이는 함수라고 판단되는 경우 해당 부분을 더 효율적인 machine code 로 작성해서 저장해둠 (Hot Code). 예상과 다르게 동작하면 최적화를 해제 (Deoptimization) 하기도 함

3. Garbage Collection

더 이상 사용하지 않는 메모리를 알아서 찾고 정리함

### 왜 v8 이 중요한가?

v8 의 압도적인 성능 때문에 무거운 서버 로직을 처리할 수 있는 언어로 JS 가 사용될 수 있게 되었음

v8 은 간단히 '내 JS 코드를 엄청난 속도의 기계어로 바꿔주는 고성능 엔진'

# Node.js 의 핵심 개념

1. Event Loop : Node.js 가 single thread 에도 불구하고 여러 동시 접속을 처리할 수 있는 비결. 오래 걸리는 작업은 Worker Thread 에 던져두고, Main Thread 는 계속 다음 명령을 받음. 작업이 끝나면 callback 을 통해 결과를 전달
2. Asynchronous programming : Node.js 코드의 90%는 비동기. 과거는 callback 지옥의 위험이 있었지만 현재 `Promise`, `async/await` 를 사용함
3. Module System (CommonJS vs ESModules) : 코드를 분리하고 가져오는 방식
	* CommonJS : 전통적인 방식 `require()`, `module.exports`
	* ES Modules (ESM) : 최신 표준, 브라우저와 호환 `import`, `export`

## Module System

Node.js 의 Module : 코드를 기능별로 쪼개서 관리하고, 필요할 때마다 가져다 쓰는 방식

### CommonJS (CJS)

Synchronous 하게 로드됨. 모듈을 다 불러올 때까지 다음 코드가 실행되지 않음

```js
// math.js (내보내기)
const add = (a, b) => a + b;
module.exports = add;

// app.js (가져오기)
const add = require('./math');
console.log(add(1, 2));
```

### ES Modules (ESM)

ECMAScript(JS 공식 표준) 모듈 방식. 최신 브라우저, Node.js 환경에서 권장됨

Asynchronous 하게 로드되며, static 분석이 가능해 성능 최적화에 유리함

`package.json` 에 `"type" : "module"` 을 추가하거나 파일 확장자를 `.mjs` 로 씀

```js
// math.mjs (내보내기)
export const add = (a, b) => a + b;

// app.mjs (가져오기)
import { add } from './math.mjs';
console.log(add(1, 2));
```

* 모듈 파일 생성 `math.ts`

```ts
export const add = (a: number, b: number) => a + b;
```

* 파일 import `test.ts`

```ts
import { add } from './math.js'; // 확장자 생략 불가

console.log(add(1, 2));
```

## package.json

프로젝트의 명세서, 설계도.

프로젝트가 어떤 이름을 가졌는지, 어떤 package 를 사용하는지, 어떻게 실행해야 하는지에 대한 정보가 담긴 JSON 형식의 파일

1. Dependency Management (Like Cocoapods)
2. Script 실행 : 복잡하고 긴 터미널 명령어를 짧은 별명으로 만들어 저장할 수 있음. e.g. `npx tsx test.ts` 대신 `npm run dev` 라고만 쳐도 실행 가능하게
3. 프로젝트 정보 저장 : 이름, 버전, 작성자, 라이선스 등

```json
{
  "name": "my-study-project",        // 프로젝트 이름
  "version": "1.0.0",                 // 프로젝트 버전
  "type": "module",                   // ESM 방식을 쓰겠다는 설정 (중요!)
  "scripts": {
    "start": "node index.js",         // npm start로 실행
    "dev": "tsx index.ts"             // npm run dev로 실행
  },
  "dependencies": {                   // 실행할 때 꼭 필요한 라이브러리
    "express": "^4.18.2"
  },
  "devDependencies": {                // 개발할 때만 필요한 도구 (TS 등)
    "typescript": "^5.0.0",
    "tsx": "^4.0.0"
  }
}
```

* `npm init`
* `npm init -y` : 여러 질문에 기본적으로 yes 라 대답

위 명령어를 입력하면 `package.json` 생성됨

# npm

Node.js 의 package 를 관리할 수 있는 도구

* `npm install` : `package.json` 파일에 정의된 dependencies 들을 `node_modules` 폴더에 설치하는 명령어
* `npm run` : `package.json` 의 scripts 객체에서 임의의 명령을 실행함 

✅ https://www.npmjs.com/package/chalk 의존성 추가해보기

`npm install chalk`

1. `package.json` 의 `"dependencies"` 에 추가됨
2. node_modules 폴더에 chalk 가 추가됨

```json
{
  "name": "day2",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs",
  "dependencies": {
    "chalk": "^5.6.2" // chalk 가 추가됨
  }
}
```

✅ Scripts 에 명령어 등록해보기

```json
"scripts": {
  "dev": "tsx day2.ts" // npx 가 안 붙어도 된다!
},
```

`npm run dev` -> `npx tsx day2.ts` 가 실행됨

* 긴 명령어를 줄여줌
* 환경 구축이 쉬워짐 : `tsc` 나 `tsx` 를 전역으로 설치 안했을 때 command not found 에러가 떴는데 `npm run` 을 통해 실행하면 npm 이 자동으로 현재 폴더의 `node_modules/.bin` 을 뒤져서 실행해줌
* 협업의 표준 : 새로운 개발자는 `scripts` 부분을 보고 명령어를 바로 알 수 있음

# tsconfig.json

`tsc` (TS 컴파일러) 를 위한 설정.

TS -> JS 변환할 때 얼마나 엄격하게 검사할 지, 어떤 버전의 자바스크립트로 내보낼 지 등을 정함

1. Compile 옵션 지정 : TS -> JS 변환할 때 세부 규칙. 최신 JS 로 바꿀지, 옛날 브라우저도 지원하게 바꿀지 등
2. 프로젝트 범위 지정 : 검사할 폴더와 무시할 폴더 정함
3. 엄격도 조절 : Type check 를 까다롭게 할지 등을 정함

```json
{
  "compilerOptions": {
    "target": "ES6",           // 변환될 자바스크립트 버전
    "module": "ESNext",        // 모듈 시스템 방식 (CommonJS vs ESM)
    "strict": true,            // 모든 엄격한 타입 체크 활성화 (권장!)
    "outDir": "./dist",        // 컴파일된 .js 파일들이 저장될 폴더
    "rootDir": "./src",        // 실제 .ts 소스 코드가 있는 폴더
    "esModuleInterop": true,   // CommonJS 모듈을 ESM처럼 편하게 불러오게 함
    "skipLibCheck": true       // 라이브러리 파일(.d.ts)의 타입 검사는 건너뜀 (속도 향상)
  },
  "include": ["src/**/*"],     // src 폴더 아래의 모든 파일을 포함
  "exclude": ["node_modules"]  // 제외할 폴더
}
```

`npx tsc --init` 을 통해 생성

* `npx` : Node package execute. npm 을 좀 더 편하게 사용하기 위해 사용됨. `npm 5.2.0` 버전부터 자동으로 설치됨. 패키지를 설치하지 않고도 원하는 패키지를 실행할 수 있음. 일회용.
* `npm` : Node package manager. 패키지 관리가 목적.

`tsconfig.json` 이 없다면 `tsc` 명령어를 칠 때마다 터미널에 수십 개의 옵션을 직접 적어야 함.

`tsc src/index.ts --target es6 --module commonjs --strict true ...` 대신 `npx tsc` 만 쳐도 설정 파일 내용대로 알아서 빌드가 진행됨

# Node built-in module

`npm install` 없이도 바로 불러와서 사용할 수 있는 도구.

## fs (File System)

컴퓨터의 파일을 읽고, 쓰고, 삭제하는 파일 시스템을 제어하는 모듈. 동기, 비동기 방식 모두 지원하지만 성능상 비동기 방식을 사용하는게 좋음

```ts
import fs from 'fs/promises';

// 파일 쓰기
await fs.writeFile('hello.txt', 'Hello Node.js!');

// 파일 읽기
const data = await fs.readFile('hello.txt', 'utf-8');
console.log(data);
```

## path

파일/폴더 경로를 다루는 도구. 운영체제마다 경로 구분자 (윈도우는 `\`, mac, linux 는 `/`) 가 다른데, 모듈을 쓰면 OS 에 상관없이 안전하게 경로 합칠 수 있음

```ts
import path from 'path';

// 경로 합치기 (OS에 맞게 자동으로 / 또는 \ 삽입)
const fullPath = path.join('src', 'services', 'user.ts'); 
console.log(fullPath); // src/services/user.ts (맥 기준)

// 파일 이름만 추출
console.log(path.basename(fullPath)); // user.ts

// 확장자만 추출
console.log(path.extname(fullPath)); // .ts
```

## http

웹 서버를 직접 만들거나 HTTP 요청을 보낼 때 사용. 현대는 `Express` 같은 프레임워크를 쓰는데, 내부적으로는 이 모듈을 기반으로 동작함

```ts
import http from 'http';

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World\n');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});
```

## crypto

암호화, 해시 생성, 비밀번호 검증 등 보안 관련 기능 담당

```ts
import crypto from 'crypto';

// 비밀번호를 해시값으로 변환 (SHA-256 알고리즘)
const password = 'my-secret-password';
const hash = crypto.createHash('sha256').update(password).digest('hex');

console.log(hash); // 아주 긴 암호화된 문자열 출력
```