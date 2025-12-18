
# Node.JS 란

Node.js 는 **브라우저 밖에서도 JavaScript 를 실행할 수 있게 해주는 Runtime 환경**

Java 는 모든 os 에서 virtual machine 환경 안에서 runtime 이 구동됨.

Node.JS 는 web browser 에 종속적인 JavaScript 가 외부에서 실행될 수 있는 runtime 환경을 제공해서 여러 os 환경에서 실행할 수 있는 환경을 제공함

## 왜 Node.js 가 유명한가? (주요 특징)

1. 비동기 I/O, Non-blocking

* 일반적인 서버 : 한 번의 하나의 일만 처리하고 뒷 사람을 기다리게 함
* Node.js : 모든 주문을 다 받고, 완성되는 대로 서빙

=> 데이터 요청이 많은 실시간 서비스에 유리

2. Single thread

비동기 방식 때문에 적은 자원으로도 수많은 요청을 빠르게 처리 가능

3. Full Stack

Frontend 에서 사용하는 JS 를 backend 에도 쓸 수 있다는 장점

# Install & Setup

```
brew install node
# node -v 로 설치 확인
npm install -g typescript tsx
```

* `brew install node`
	* 의존성 확인 및 설치 : `icu4c` (유니코드 지원), `ca-certificates` 같은 필수 라이브러리 확인, 없으면 설치
	* Node.js 바이너리 다운로드 및 배치 : Node.js 실행 파일들을 다운로드해서 특정 위치에 저장
	* Symbolic Link 생성 : 설치된 위치는 복잡해서 터미널 어디서든 `node` 라 쳐도 실행될 수 있는 지름길 (symbolic link) 생성
	* NPM (Node Package Manager) 설치 : 전 세계 개발자들이 만든 오픈소스 라이브러리를 내 프로젝트에 가져올 때 사용하는 도구
* `npm install -g typescript tsx`
	* Global 환경에 typescript, tsx 설치
	* TypeScript : 설치하면 터미널에서 `tsc` 라는 명령어를 사용할 수 있음. 내가 작성한 `.ts` 파일을 browser / Node.js 가 이해할 수 있는 `.js` 파일로 컴파일
	* tsx (TypeScript Execute) : 예전에는 `.ts` 파일을 실행하려면 `tsc` 로 변환해서 `node` 로 실행해야 했는데, 변환 과정 없이 즉시 터미널에서 `.ts` 파일을 실행시켜줌
	* (-g 없이 일단 설치)


# TypeScript

TypeScript 의 type system (JS 코드에 '이름표', '규칙'을 붙여주는 보안 장치) 는 더 나은 code completion, 이른 error detection 등의 장점이 있음.

## Before Start

### Rethinking the Class

C#, Java 는 OOP 언어라 부름. Class 가 code 조직의 기본 unit 이고, 모든 데이터와 runtime 에서의 동작의 기본 컨테이너. 모든 기능과 데이터를 클래스에 저장하는 것은 좋은 도메인 모델이 될 수 있지만, 모든 도메인에서 이게 필요하지 않음.

#### Free Functions and Data

JavaScript 에서 function 은 어디에나 존재할 수 있고, `class` / `struct` 안에 위치할 필요 없이 데이터는 자유롭게 전달될 수 있음. 이런 OOP 계층에 묶여있지 않는 "Free" functions 은 JS 에서 프로그램을 작성하는데 선호되는 모델

### OOP in TypeScript

Class 를 사용할 수 있음. TypeScript 는 JavaScript class 를 지원하며, interface, inheritance, static method 를 지원함

### Rethinking Types

#### Nominal Reified Type Systems (명목적 구체화된 type system)

* Nominal : 이름. 데이터의 생김새 (구조) 가 같아도 이름이 다르면 서로 다른 타입으로 간주. 타입의 이름이 일치해야만 같은 타입으로 인정
* Refied : 실체화됨. Type 정보가 컴파일 단계에서 사라지지 않고 runtime 까지 그대로 살아있는것. Runtime 에서 type 을 질의했을 때 답 가능.

TypeScript 는 두 개념과 정반대. 

* Structural : 이름이 달라도 생김새가 같으면 같은 타입으로 봄
* Type Eraseure : JS 로 컴파일 되는 순간 모든 타입 정보가 지워짐

```ts
interface Dog { name: string; }
interface Person { name: string; }

let myDog: Dog = { name: "바둑이" };
let myPerson: Person = myDog; // ✅ TypeScript 에서는 통과
```

#### Types as Sets

C# 이나 Java 에서는 runtime type 과 그들의 compile-time 정의와 일대일 대응이 되도록 했음. 

TypeScript 에서는 type 을 공통점이 있는 값들의 집합으로 생각함. 따라서 TypeScript 에서는 모든 type 이 집합이기 때문에, `string` 에 속하면서도 `number` 집합에 속할 수 있다고 생각하는 것이 자연스러움. 단순히 두 집합 (`string | number`) 의 교집합에 속하는 값들이 여기에 해당.

#### Erased Structural Types

TypeScript 에서 객체는 단순히 하나의 정확한 type 에 해당하지 않음. 만약 객체가 특정 interface 를 만족하도록 구성했다면 그 객체를 interface 가 요구되는 곳에 사용될 수 있음

```ts
interface Pointlike {
	x: number;
	y: number;
}

interface Named {
	name: string;
}

// 같은 속성을 가지고 있어서 자연스럽게 interface 를 따르는 것처럼 사용 가능
const obj = {
	x: 0,
	y: 0,
	name: "Origin",
};

function logPoint(point: Pointlike) {
	console.log("x = " + point.x + ", y = " + point.y);
}

function logName(x: Named) {
	console.log("Hello, " + x.name);
}

logPoint(obj);
logName(obj);
```

* https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-oop.html

## The Basics

https://www.typescriptlang.org/docs/handbook/2/basic-types.html

```ts
// Accessing the property 'toLowerCase'
// on 'message' and then calling it
message.toLowerCase();

// Calling 'message'
message();
```

위 코드에서 `message` 의 값을 우리가 모른다고 가정했을 때, 위 두 코드의 결과가 어떤건지 알기 어려움.

* `message` 는 호출가능한가?
* `toLowerCase` 라는 property 가 있는가? 있다면 그것 또한 호출 가능한가?
* 모든 값들이 호출 가능하면 무엇을 return 하는가?

```ts
const message = "Hello World!";
```

위와 같이 정의되어 있다면 `message.toLowerCase()` 에서 소문자로 변형된 문자열을 받을 것이고 두 번째 코드에서 exception 이 발생할 것.

`string` / `number` 같은 primitives 같은 값들에 대해서는 `typeof` operator 를 사용해서 runtime 에서 타입을 알 수 있음. 하지만 함수의 경우 type 을 정의할 수 있는 runtime 매커니즘이 없음

```ts
function fn(x) {
	return x.flip();
}
```

위 코드는 객체가 `flip` 이라는 호출 가능한 property 가 있어야지만 제대로 동작할텐데, **JS 는 runtime 에서 이를 확인할 수 있는 방법이 없음. JS 에서는 단순히 실행해서 확인하는 수밖에 없음. 이런 동작은 코드가 실행되기 전에 코드가 무엇을 할 지 예상하기 어렵게 하고, 코드를 작성할 때 코드가 뭘 하는지 개발자가 알기 어렵게 함.**

이런 관점에서 type 은 어떤 값들이 `fn` 에 전달될 수 있는지 설명하는 개념임. **JS 는 dynamic typing 만을 제공해서, 실제로 확인하기 위해서 코드를 실행하는 방식을 채택하고 있음.**

이와 반대로 **static type 시스템**은 코드가 실행되기 전에 예상할 수 있게 해줌.

### Static type-checking

위 예제에서 `string` 을 함수로서 호출하려고 할 때 `TypeError` 를 받았음. 대부분의 사람들은 코드를 실행하는 도중에 에러를 만나고 싶어하지 않음.

이상적으로, 코드를 실행하기 전에 버그가 있는지 찾는 tool 을 가질 수 있음. TypeScript 같은 static type-checker 가 이런 역할을 함. **Static type system 은 코드를 실행하기 전에 값들이 어떤 형태와 동작을 할 수 있는지 설명함.** TypeScript 같은 type-cecker 는 그 정보를 활용해서 어떤 일들이 발생할 수 있는지를 알려줌

![[Pasted image 20260403235908.png]]

위 예제에서 코드를 실행하기 전에 TypeScript 는 에러 메세지를 보여줌

### Non-exception Failures

Runtime errors : JS runtime 이 뭔가 이상한 일이 발생했을 때 알려줌. 이런 에러가 발생하는 이유는 ECMAScript 명세에서 무언가 예상치 못한 일이 발생했을 때 언어가 어떤 방식으로 대응해야 하는지에 대한 명시적인 지침이 있기 때문.

명세에서는 callable 이 아닌 것을 호출하려 할 때 에러를 던져야 한다고 함. 특정 property 에 접근하는데 그 property 가 객체에 존재하지 않을 때 당연히 에러를 던져야 한다고 생각할 수 있지만 JS 는 단순히 `undefined` 값을 리턴함

```js
const user = {
	name: "Daniel",
	age: 26,
};

user.location; // returns undefined
```

궁극적으로 'valid' 한 JS 가 에러를 즉각적으로 던지지 않더라도 **static type system 은 어떤 코드에 대한 호출이 에러로 표시되어야 하는지를 알려야 함.** TypeScript 에서 아래 예시에서 `location` 이 정의되지 않았다고 에러를 생성함

```ts
const user = {
	name: "Daniel",
	age: 26,
};

user.location;
// Property 'location' does not exist on type '{ name: string; age: number; }'.
```

TS 는 많은 버그를 감지함. 오타, 호출되지 않는 function, 간단한 logic error 등등

### Types for Tooling

TS 는 코드에서 실수를 했을 때 버그를 탐지할 수 있음.

Type-checker 는 올바른 property 에 접근하고 있는지와 같은 것들을 확인할 수 있는 정보를 가지고 있음. 그 정보를 가지고, 어떤 property 를 사용할 것인지를 제안할 수도 있음.

Core type-checker 는 에러 메세지와 code completion 을 제공할 수 있음

### `tsc`, TypeScript compiler

`tsc` : TypeScript compiler. tsc 를 사용하려면 npm 을 통해 가져와야 함

`npm install -g typescript`

Type script 를 설치하고, console 에 log 를 찍는 코드를 `hello.ts` 에 생성한 후 `tsc hello.ts` 로 실행하면 , `tsc` 를 실행했지만 아무일도 안 일어남. 코드 상에 type error 가 없기 때문에 console 에 아무런 output 도 없음.

하지만 `hello.js` 파일이 새로 생김. 이는 `tsc` 가 JavaScript 파일로 compile 한 결과물.

```ts
// This is an industrial-grade general-purpose greeter function:

function greet(person, date) {
	console.log(`Hello ${person}, today is ${date}!`);
}

greet("Brendan");
```

Type-checking error 를 발생하게 한 후 `tsc hellos.ts` 를 다시 실행하면 에러를 볼 수 있음

### Emitting with Errors

위의 type-checking error 가 발생한 예제에서 `hello.js` 는 바뀌긴 했는데, 정확히 `ts` 파일이랑 동일하게 생김. `tsc` 가 코드에 대한 에러를 보여주긴 했지만, 이는 TS 의 핵심 개념 중 하나임.

코드를 type-checking 하는 것은 실행할 수 있는 프로그램의 종류를 제한하는 것이기 때문에, type-checker 가 허용가능한 것들을 찾는 것에 대한 tradeoff 가 있을 수 있음. (안전함을 얻는 대신 자유로움을 포기함) 대부분의 경우 큰 문제가 되지는 않지만, JS 코드를 TS 코드로 변환하는 도중 type-checking 에러를 받았는데, type-checker 를 위해 코드를 수정해야 하는 경우가 있다고 해보자. 이런 경우 원래 JS 코드는 잘 동작하는데 TS 로 변환해야 하는가? 

그래서 TS 는 이런 상황에서 병목이 되지 않게 `noEmitOnError` compiler option 을 사용할 수 있음.

### Erased Types

```js
"use strict";

// 1. Type annotation 이 없어졌음
function greet(person, date) {
	// backtick 을 상요하던게 일반 string 을 사용하도록 바뀜
	console.log("Hello ".concat(person, ", today is ").concat(date.toDateString(), "!"));
}

greet("Maddison", new Date());
```

위 코드는 바뀐 javascript.

Type annotation 은 JS 의 일부가 아니기 때문에 바로 TypeScript 을 브라우저나 runtime 에서 실행할 수 없고, compiler 가 필요한 이유. Compiler 는 TS 종속적인 코드를 변환해서 실행할 수 있는 형태로 바꿔줘야 함. 대부분의 TS 종속적인 코드는 지워지기 때문에 type annotation 또한 지워짐

**Type annotation 은 프로그램의 runtime 동작을 바꿀 수 없음**

### Downleveling

TS 에서 backtick 을 사용한 문자열이 일반 큰 따옴표를 사용한 것을 JS 에서 확인할 수 있었음.

Template string 은 ECMAScript 의 2015 버전의 기능임. TS 는 ECMAScript 의 최신 버전의 코드를 낮은 버전의 것으로 작성할 수 있게 됨. 이는 최신 버전의 ECMAScript 를 더 낮은 버전의 것으로 이동하는 downleveling 을 필요로 하게 됨.

TS 는 ES5 (ECMAScript 5) 를 target 하는데, 이는 더 오래된 버전임. `target` 옵션을 사용해서 더 최신의 것을 선택할 수 있음. `--target es2015` 를 실행하면 TS 가 ECMAScript 2015 를 타게팅해서 그 버전의 코드를 실행할 수 있게 해줌. `tsc --target es2015 hello.ts` 를 실행하면 그대로 JS 에서 backtick 이 사용된 것을 확인할 수 있음

### Strictness

TypeScript 는 기본적으로 관대함. Type 은 optional (안 적어도 ㄱㅊ), lenient inference (어떤 타입인지 헷갈리는 경우 에러 내는 대신 any 로 취급) 를 가지며 `null`. `undefined` 값에 대한 체크를 하지 않음.

반면 TS 가 최대한 검증할 수 있는 건 검증하기 위해 strictness 설정을 제공함. Stictness 설정은 static type-checking 을 switch 대신 dial 형태로 변경해서, 그 수치를 높일수록 TS 가 검증하는게 더 많아짐. 더 오래 걸릴 수는 있지만 더 많은 확인을 하고 정확한 tooling 을 제공함.

TS 는 여러 키고 끌 수 있는 type-checking strictness flag 가 있음. CLI 에서 `strict` flag 를 설정하거나 `tsconfig.json` 에서 `"strict" : true` 로 설정할 수 있음. `noImplicitAny` 와 `strictNullChecks` 가 여기 포함됨

#### `noImplicitAny`

TS 는 관대한 `any` 타입을 추론하도록 하지 않음. `any` 로 fall back 하는 것은 최악으로, 단지 JS 와 똑같음. 

`noImplicitAny` 플래그를 설정하면 `any` 로 추론된 타입에 대해 에러를 뱉음

#### `strictNullChecks`

기본적으로 `null` 이나 `undefined` 같은 값은 다른 타입에 할당 가능. 코드를 작성하기는 편하지만 실수할 위험이 높아짐. 이 플래그를 사용하면 `null` 나 `undefined` 를 더 명시적으로 다룰 수 있게 해줌

## Everyday Types

### Primitives: `string`, `number`, `boolean`

### Arrays

`number[]` / `Array<number>` 와 같이 선언 가능

### `any`

Typechecking 에러를 원하지 않는 경우 사용 가능

```ts
let obj: any = { x: 0 };
// None of the following lines of code will throw compiler errors.
// Using `any` disables all further type checking, and it is assumed
// you know the environment better than TypeScript.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

### Type Annotations on Variables

`const`, `var`, `let` 을 사용할 때 type annoation 을 통해 타입을 명시적으로 표현할 수 있음

```ts
let myName: string = "Alice";
```

TS 는 가능한 경우에 자동으로 타입을 추론하기 때문에 type annotation 이 꼭 필요하지는 않음

### Functions

```ts
// Parameter type annotation. 붙은 경우 type-checking 함
function greet(name: string) {}

// Return type annotation
// TS 가 자동으로 함수의 리턴 타입을 추론하기 때문에 필요하지는 않음
function getFavoriteNumber(): number {
	return 26;
}

// Promise 를 return하는 경우
async function getFavoriteNumber(): Promise<number> {
	return 26;
}

// Anonymous function
const names = ["Alice", "Bob", "Eve"];

// Contextual typing for function - parameter s inferred to have type string
names.forEach(function (s) {
	console.log(s.toUpperCase());
});

// parameter s 가 type annotation 이 없어도 `forEach` 함수의 type 을 사용해서 항상 `s` 가 갖는 타입을 알 수 있음 : contextual typing (함수가 등장한 context 가 type 을 알려줌)

// Contextual typing also applies to arrow functions
names.forEach((s) => {
	console.log(s.toUpperCase());
});
```

### Object Types

JS 의 property 가 있는 값

```ts
// The parameter's type annotation is an object type
function printCoord(pt: { x: number; y: number }) {
	console.log("The coordinate's x value is " + pt.x);
	console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 3, y: 7 }); // , 나 ; 를 사용해서 property 구분 가능
```

각 property 의 type 도 optional. Type 을 명시하지 않는 경우 `any` 라고 가정함

```ts
// Optional Property
function printName(obj: { first: string; last?: string }) {
	// ...
}

// Both OK
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });

// optional property 에 대한 undefined 검사
function printName(obj: { first: string; last?: string }) {
// Error - might crash if 'obj.last' wasn't provided!
console.log(obj.last.toUpperCase());
// 'obj.last' is possibly 'undefined'.

if (obj.last !== undefined) {
// OK
	console.log(obj.last.toUpperCase());
}

// A safe alternative using modern JavaScript syntax:
console.log(obj.last?.toUpperCase());
}
```

### Union Types

TS 의 type system 은 operator 를 사용해서 이미 존재하는 것들에서 새로운 타입을 생성할 수 있게 해줌

```ts
// Union type
function printId(id: number | string) {
	console.log("Your ID is: " + id);
}

// OK
printId(101);

// OK
printId("202");

// Error
printId({ myID: 22342 });

Argument of type '{ myID: number; }' is not assignable to parameter of type 'string | number'.

// separator 를 첫 번째 요소 전에 쓸 수도 있음
function printTextOrNumberOrBool(
	textOrNumberOrBool:
	| string
	| number
	| boolean
) {
	console.log(textOrNumberOrBool);
}
```

Union type : 두 개 이상의 type 으로 조합된 타입. 두 타입 중 하나 일 수 있음

```ts
function printId(id: number | string) {
	if (typeof id === "string") {
		// In this branch, id is of type 'string'
		console.log(id.toUpperCase());
	} else {
		// Here, id is of type 'number'
		console.log(id);
	}
}

function welcomePeople(x: string[] | string) {
if (Array.isArray(x)) {
		// Here: 'x' is 'string[]'
		console.log("Hello, " + x.join(" and "));
	} else {
		// Here: 'x' is 'string'
		console.log("Welcome lone traveler " + x);
	}
}
```

### Type Aliases

Object type, union type 을 직접 type annotation 에 작성하는게 편하지만 일반적으로 하나의 타입을 이름으로 여러 번 사용하는 경우가 많음

Type alias : type 의 이름.

```ts
type Point = {
	x: number;
	y: number;
};

function printCoord(pt: Point) {
	console.log("The coordinate's x value is " + pt.x);
	console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });

type ID = number | string;
```

Alias 는 오직 alias 일 뿐이고, 이를 사용해서 같은 타입의 여러 다른 버전을 생성할 수 없음

```ts
type UserInputSanitizedString = string;

function sanitizeInput(str: string): UserInputSanitizedString {
	return sanitize(str);
}

// Create a sanitized input
let userInput = sanitizeInput(getInput());

// Can still be re-assigned with a string though
// 근본적으로 두 타입이 같기 때문에 가능
userInput = "new input"; 
```

### Interfaces

Interface declaration 은 object type 에 이름을 붙이기 위한 방법중 하나

```ts
interface Point {
	x: number;
	y: number;
}

function printCoord(pt: Point) {
	console.log("The coordinate's x value is " + pt.x);
	console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

#### Type Aliases, Interface 의 차이

큰 차이는 type 은 새로운 property 를 추가하기 위해 re-open 될 수 없지만 interface 는 항상 확장할 수 있다는 점

* Extend

```ts
// interface
interface Animal {
  name: string;
}  

interface Bear extends Animal {
  honey: boolean;
} 
 
const bear = getBear();
bear.name;
bear.honey;

// type
type Animal = {
  name: string;
}  

type Bear = Animal & { 
  honey: boolean;
}  

const bear = getBear();
bear.name;
bear.honey;
```

* 새로운 필드 추가 (interface 만 가능)

```ts
interface Window {
  title: string;
}  

interface Window {
  ts: TypeScriptAPI;
}  

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```

### Type Assertions

TS 이 알지 못하는 값의 타입에 대한 정보를 개발자가 아는 경우 type assertion 을 통해 더 구체적인 타입을 명시할 수 있음

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;

const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

Type annotation 같이 type assertion 은 compiler 에 의해 지워지고 runtime 동작에 영향을 줄 수 없음

### Literal Types

Type 위치에 특정 문자열 / 숫자를 작성할 수 있음

JS에서 `var`, `let` 은 둘다 변수 안에 할당된 것을 수정할 수 있었고, `const` 는 그렇지 않음.

```ts
let changingString = "Hello World";
changingString = "Olá Mundo"
// Because `changingString` can represent any possible string, that
// is how TypeScript describes it in the type system

const constantString = "Hello World";
// Because `constantString` can only represent 1 possible string, it
// has a literal type representation
constantString;

let x: "hello" = "hello";
// OK
x = "hello";

// ...
x = "howdy";
// ❌ Type '"howdy"' is not assignable to type '"hello"'.

function printText(s: string, alignment: "left" | "right" | "center") {
	// ...
}

// OK
printText("Hello, world", "left");

printText("G'day, mate", "centre");
// ❌ Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.

function compare(a: string, b: string): -1 | 0 | 1 {
	return a === b ? 0 : a > b ? 1 : -1;
}

interface Options {
	width: number;
}

function configure(x: Options | "auto") {
	// ...
}

// OK
configure({ width: 100 });
configure("auto");

configure("automatic");
// ❌ Argument of type '"automatic"' is not assignable to parameter of type 'Options | "auto"'.
```

#### Literal Inference

객체로 변수를 초기화할 때 TS 는 객체의 property 가 나중에 변할 수 있다고 가정함

```ts
const obj = { counter: 0 };

//    
if (someCondition) {
	obj.counter = 1; // 에러로 취급 안함
}
```

TS 는 `obj.counter` 가 `number` 타입을 갖도록 하지, `0`을 강제하지는 않음.

```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;

const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
// ❌ Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
```

`req.method` 는 `string` 으로 추론되지, `"GET"` 으로 추론되지 않기 때문에 TS 는 이것을 에러로 간주함

```ts
// 1. type assertion 사용

// literal type "GET" 사용
const req = { url: "https://example.com", method: "GET" as "GET" };
handleRequest(req.url, req.method as "GET")

// 2. as const 를 사용해서 전체 object 를 type literal 로 변환
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

`as const` suffix 는 type system 을 위한 `const` 처럼 동작하는데, 모든 property 가 `string` 이나 `number` 같이 일반적인 버전이 아닌 literal type 으로 할당되는 것을 보장함

### `null` and `undefined`

JS 는 빈 / 초기화되지 않은 값을 표현하기 위해 `null`, `undefined` 를 사용함

```ts
function liveDangerously(x?: number | null) {
	// No error
	console.log(x!.toFixed());
}
```

## Narrowing

### `typeof` type guards

JS 는 `typeof` operator 를 지원해서 runtime 에서 값들의 타입에 대한 정보를 알 수 있게 해줌

```ts
function printAll(strs: string | string[] | null) {
	if (typeof strs === "object") {
		for (const s of strs) {
			// ❌ 'strs' is possibly 'null'.
			console.log(s);
		}
	} else if (typeof strs === "string") {
		console.log(strs);
	} else {
		// do nothing
	}
}
```

`typeof null` 은 "object"

### Truthiness narrowing

`if` 문은 항상 조건이 `boolean` 타입을 가질 것이라 예상하지 않음

```ts
function getUsersOnlineMessage(numUsersOnline: number) {
	if (numUsersOnline) {
		return `There are ${numUsersOnline} online now!`;
	}
	
	return "Nobody's here. :(";
}
```

`0`, `NaN`, `""`, `0n`, `null`, `undefined` 는 모두 `false` 로 취급되고 나머지는 무조건 `true`

```ts
function printAll(strs: string | string[] | null) {
	// `strs` 가 null 인 경우도 여기서 걸러짐
	if (strs && typeof strs === "object") {
		for (const s of strs) {
			console.log(s);
		}
	} else if (typeof strs === "string") {
		console.log(strs);
	}
}
```

### Equality narrowing

`switch` 문, `===`, `!==`, `==`. `!=` 를 사용할 수 있음

```ts
function example(x: string | number, y: string | boolean) {
	if (x === y) {
		// x, y 가 같은 경우는 그들의 타입이 같아야지 성립하기 때문에 TS 는 여기서 x, y 가 둘 다 무조건 string인 것을 알게 됨
		x.toUpperCase();
		y.toLowerCase();
	} else {
		console.log(x);
		console.log(y);
	}
}
```

* `===` : Strict Equality, 데이터 타입까지 일치하는지 확인
* `==` : Abstract Equality, 비교하려는 두 값이 타입이 다르면 JS 가 알아서 타입을 바꾼 후 값이 같은지 확인

### `in` operator narrowing

JS 는 객체 / prototype chain 이 특정 이름의 property 가 있는지 검증해주는 `in` operator 가 있음. TS 는 이를 사용해서 poetential type 을 narrow down.

`"value" in x` 는 `"value"` 는 string literal이고 `x` 는 union type 임.

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    // 🔵 "True" 브랜치: 여기가 실행된다는 건 animal에 'swim'이 있다는 뜻!
    // 타입스크립트는 이제 animal을 'Fish' 타입으로 확신하고 처리합니다.
    animal.swim(); 
  } else {
    // 🔴 "False" 브랜치: 'swim'이 없다는 뜻!
    // 남은 가능성은 'Bird'뿐이므로 animal을 'Bird' 타입으로 간주합니다.
    animal.fly();
  }
}

type Human = { name: string; age?: number }; // age가 있을 수도, 없을 수도 있음
type Robot = { serial: number };

function check(subject: Human | Robot) {
  if ("age" in subject) {
    // age가 선택적(optional) 속성이더라도, 
    // 일단 존재한다면 이 녀석은 Robot이 아니라 Human일 확률이 높다고 좁혀줍니다.
    console.log(subject.name); 
  }
}
```

### `instanceof` narrowing

JS 는 특정 값이 다른 값의 instance 인지 확인해주는 operator 가 있음. Prototype chain 을 훑는데, `typeof` 가 알아내지 못하는 구체적인 객체 타입 찾을 때 사용. `typeof` 는 주로 원시 값 등에 사용

```ts
// interface 는 체크 불가.
function logValue(x: Date | string) {
  if (x instanceof Date) {
    // 이제 x는 Date 객체로 좁혀짐!
    console.log(x.toUTCString());
  } else {
    // x는 자동으로 string이 됨
    console.log(x.toUpperCase());
  }
}
```

### Assignments

```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";
// let x: string | number

x = 1;
console.log(x);
// let x: number

x = "goodbye!";
console.log(x);
// let x: string

x = true;
// ❌ 얘는 안됨
```

### Using type predicates

Type 변화를 직접 제어하는 경우도 있음.

사용자 정의된 type guard 를 정의하기 위해서 function 의 return type 을 type predicate 로 정의하면 됨

```ts
function isFish(pet: Fish | Bird): pet is Fish {
	return (pet as Fish).swim !=== undefined;
}
```

`pet is Fish` 가 type predicate. Predicate 는 `parameterName is Type` 의 형태를 가지고, `parameterName` 은 현재 function signature 의 인자 이름이어야 함.

`isFish` 가 호출될 때마다 TS 는 그 변수를 narrow 해서 특정 타입을 준수하는지 확인할 것임

### Discriminated unions

대부분의 경우 JS 는 더 복잡한 구조체를 다룸. TS 에서 객체들의 종류를 쉽게 구분하기 위해 사용되는 패턴

* Union : `Circle | Square`
* Discriminant : `kind` 같은 이름표 붙임
* Literal Type

```ts
interface Shape {
	kind: "circle" | "square"; // union of string literal types
	radius?: number;
	sideLength?: number;
}

function handleShape(shape: Shape) {
	// oops!
	if (shape.kind === "rect") {
		// ❌ This comparison appears to be unintentional because the types '"circle" | "square"' and '"rect"' have no overlap.
		// ...
	}
}

function getArea(shape: Shape) {
	return Math.PI * shape.radius ** 2; // non-null assertion ! 를 써서 `shape.radius` 가 정의되었다고 할 수도 있지만 이상적인 방법은 아님
	// ❌ 'shape.radius' is possibly 'undefined'.
}

interface Circle {
	kind: "circle"; // common property
	radius: number;
}

interface Square {
	kind: "square"; // common property
	sideLength: number;
}

type Shape = Circle | Square;

function getArea(shape: Shape) {
	return Math.PI * shape.radius ** 2;
	// ❌ Property 'radius' does not exist on type 'Shape'. Property 'radius' does not exist on type 'Square'.
}

// union 의 모든 type 이 공통된 literal type 이 있는 경우 TS 는 discriminated union 으로 간주해서 union 의 member 를 narrow out 할 수 있음
function getArea(shape: Shape) {
	if (shape.kind === "circle") {
		return Math.PI * shape.radius ** 2;
		// ✅ (parameter) shape: Circle
	}
}

function getArea(shape: Shape) {
	switch (shape.kind) {
		case "circle":
			return Math.PI * shape.radius ** 2;
		case "square":
			return shape.sideLength ** 2;
	}
}
```

### `never` type

TS 는 `never` type 을 사용해서 존재하지 않는 상태를 표시하기 위해 사용함

```ts
type Shape = Circle | Square;

function getArea(shape: Shape) {
	switch (shape.kind) {
		case "circle":
			return Math.PI * shape.radius ** 2;
		case "square":
			return shape.sideLength ** 2;
		default:
			const _exhaustiveCheck: never = shape;
		return _exhaustiveCheck;
	}
}

interface Triangle {
	kind: "triangle";
	sideLength: number;
}

type Shape = Circle | Square | Triangle;

//위 코드 const _exhaustiveCheck 부분에서 `Triangle` is not assignable to type 'never' 에러가 뜸
```