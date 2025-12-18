
# SQL basics: SELECT, INSERT, UPDATE, DELETE, WHERE, JOIN

SQL(Structured Query Language) : DB 에 query 를 하기 위한 가장 표준적인 언어.

## SELECT

데이터 조회

* 구조 : `SELECT [COLUMN] FROM [TABLE]`

```sql
SELECT name, email FROM users;
-- 모든 컬럼을 다 가져올 때는 * 를 씀
SELECT * FROM users;
```

## WHERE

조건 필터링. 특정 조건을 만족하는 데이터만 골라내고 싶을 때 사용. `SELECT`, `UPDATE`, `DELETE`

```sql
SELECT * FROM users
WHERE age >= 20;
```

## INSERT 

데이터 추가. 새로운 Row 를 테이블에 삽입

* 구조 : `INSERT INTO [TABLE] ([COLUMNS]) VALUES ([VALUES]);`

```sql
INSERT INTO users (name, age, email)
VALUES ('Kim', 25, 'kim@test.com');
```

## UPDATE

데이터 수정. 이미 저장된 데이터를 바꿀 때 사용. `WHERE` 를 빼먹으면 모든 행의 데이터가 다 바뀌어버리는 대참사

* 구조 : `UPDATE [TABLE] SET [COLUMN=VALUE] WHERE [CONDITION]`

```sql
UPDATE users
SET name = 'Lee'
WHERE id = 1;
```

## DELETE

행 삭제. `WHERE` 없으면 table 전체 데이터가 날라감

* 구조 : `DELETE FROM [TABLE] WHERE [CONDITION];`

```sql
DELETE FROM users
WHERE id = 5;
```

## JOIN

테이블 합치기. 테이블에 흩어진 정보를 한 번에 묶어서 볼 때 사용. 

```sql
SELECT users.name, orders.product_name
FROM users
JOIN orders ON users.id = orders.user_id;
```

# ORM

ORM 은 SQL 을 위한 Core Data 같은 느낌. 

ORM (Object-Relational Mapping) : Code 상 object 와 database 의 table (relational) 을 자동으로 연결해주는 번역기 역할. 모델링 된 객체와 관계를 바탕으로 SQL 자동으로 생성해주는 도구

## ORM 을 쓰는 이유

Core Data 를 쓸 때 SQLite 를 쓰지 않고 Swift 객체로 데이터를 다루는 것처럼 ORM 도 SQL 문법 대신 프로그래밍 언어 문법으로 db 를 조작할 수 있게 함

|**특징**|**생 SQL (Raw SQL)**|**ORM (Prisma)**|**Core Data 비유**|
|---|---|---|---|
|**조회**|`SELECT * FROM users;`|`prisma.user.findMany()`|`NSFetchRequest`|
|**생성**|`INSERT INTO users...`|`prisma.user.create()`|`NSEntityDescription`|
|**타입**|결과가 문자열/숫자 뭉치|**TypeScript 타입 지원**|`NSManagedObject`|
|**장점**|자유도가 높고 빠름|생산성이 높고 오타 방지|직관적인 데이터 관리|

## Prisma

JS, TS 커뮤니티에서 주목받는 ORM 프레임워크

## Prisma 의 3가지 핵심 요소

1. Prisma Schema (`schema.prisma`) 

데이터 Model 을 정의

```
model User {
  id    Int     @id @default(autoincrement())
  name  String
  posts Post[] // 1:N 관계 정의
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  author   User   @relation(fields: [authorId], references: [id])
  authorId Int
}
```

2. Prisma Client : 코드를 작성할 때 사용하는 자동 생성된 라이브러리. 스키마를 정의하면 내 데이터 구조에 맞는 메서드들이 생김
3. Prisma Studio : DB 안에 데이터가 잘 들어있는지 GUI 로 확인할 수 있는 browser 기반의 tool.

## 장단점

* 장점
	* 생산성 : SQL Query 를 길게 짤 필요가 없어 개발 속도가 빠름
	* Type 안정성 : 존재하지 않는 column 을 참조하면 코드 작성 중 에러를 볼 수 있음
	* DB 교체 용이 : SQL 을 Prisma 가 알아서 처리해주기 때문에 나중에 DB 엔진을 고쳐도 코드를 거의 고칠 필요가 없음
* 단점
	* 추상화 비용 : 아주 복잡하고 미세한 튜닝이 필요한 처리는 직접 SQL 을 짜는 것보다 느릴 수 있음
	* 학습 곡선 : ORM 자체의 문법과 동작 방식을 이해해야 함

# Prisma schema language (define models like Swift structs)

PSL (Prisma schema language) 는 Prisma 의 핵심. 

`schema.prisma` 파일에 작성, Model, Field, Attribute 로 구성됨

## 기본 모델 구조

Prisma Model 은 DB 의 table 이 되고, 각 필드는 table 의 column 이 됨

|**특징**|**Swift Struct**|**Prisma Model**|
|---|---|---|
|**정의 키워드**|`struct User { ... }`|`model User { ... }`|
|**필드 선언**|`let id: Int`|`id Int`|
|**Optional**|`var bio: String?`|`bio String?`|
|**기본값**|`var age: Int = 0`|`age Int @default(0)`|

```
model User {
  id    Int     @id @default(autoincrement()) // Primary Key
  email String  @unique                       // Unique 제약조건
  name  String?                               // Optional (String?)
  createdAt DateTime @default(now())          // 생성 시각 자동 기록
}
```

## Field Types

Prisma 는 다양한 데이터 타입을 지원하고 사용하는 DB(PostgreSQL ,MSQL 등) 에 맞춰 적절한 타입으로 매핑

* `String` : 텍스트
* `Int` / `BigInt` : 정수
* `Float` / `Decimal` : 실수
* `Boolean` : 참/거짓
* `DateTime` : 날짜, 시간
* `Json`

## Attribute

필드 이름 뒤에 `@`로 시작하는 키워드를 붙여 세부 설정.

* `@id` : 이 필드를 Primary Key 로 지정
* `@default(...)` : 기본값을 설정
* `@unique` : 중복된 값을 허용하지 않음
* `@updatedAt` : 데이터가 수정될 때마다 자동으로 현재 시간을 기록
* `@map("db_column_name")`

## Relationships

1:N

한 명의 User 가 여러 개의 Post 를 가질 때

```
model User {
  id    Int    @id @default(autoincrement())
  posts Post[] // 1번 유저는 여러 포스트를 가짐 (Swift의 [Post] 배열 느낌)
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  author   User   @relation(fields: [authorId], references: [id])
  authorId Int    // 실제 DB에 저장되는 Foreign Key (외래키)
}
```

## Enum 

```
enum Role {
  USER
  ADMIN
}

model User {
  id   Int  @id
  role Role @default(USER)
}
```

# Prisma Client — type-safe database queries

정의한 Prisma Schema 를 바탕으로 자동으로 생성되는 Query Builder. 

## 어떻게 자동 생성이 되는가

`npx prisma generate` 를 입력하면 Prisma 는 `node_modules` 안에 정의된 schema 에 맞는 TS 타입을 생성함

코드를 짤 때 존재하지 않는 table 을 부르거나 오타를 내면 실행하기도 전에 빨간 밑줄이 뜸

## Query CRUD

Prisma Client 는 비동기 방식으로 작동함

### Create

```ts
const newUser = await prisma.user.create({
	data: {
		name: 'Kim',
		email: 'kim@example.com'
	}
})
```

### Read

`findUnique`, `findFirst`, `findMany` 등 다양한 메서드를 제공

```ts
// ID로 하나 찾기
const user = await prisma.user.findUnique({
  where: { id: 1 },
});

// 조건에 맞는 여러 개 찾기
const activeUsers = await prisma.user.findMany({
  where: { isActive: true },
  orderBy: { createdAt: 'desc' }, // 정렬
});
```

### Update

```ts
const updatedUser = await prisma.user.update({
  where: { email: 'kim@example.com' },
  data: { name: 'Kim Junior' },
});
```

### Delete

```ts
await prisma.user.delete({
  where: { id: 1 },
});
```

### Relation 용 쿼리 Include & Select

JOIN 을 복잡하게 고민할 필요 없이 `include` 로 충분

```ts
// 유저를 가져오면서 그 유저가 쓴 포스트들도 같이 가져오기 (Eager Loading)
const userWithPosts = await prisma.user.findUnique({
  where: { id: 1 },
  include: { posts: true }, 
});

// 특정 필드만 골라서 가져오기 (성능 최적화)
const slimUser = await prisma.user.findUnique({
  where: { id: 1 },
  select: { name: true, email: true }, // id나 다른 필드는 안 가져옴
});
```

### Type 안정성

**일반적인 SQL**

```js
const user = await db.query("SELECT names FROM users"); // name 을 names 로 오타 냈는데 코드 실행 전에는 에러인지 모름
```

**Prisma Client**

```ts
const user = await prisma.user.findUnique({
	where: { id: 1 },
	select: { names: true } // 에러가 뜸. names 속성이 없다...
});
```

# Migrations — versioned database changes

Migration : Sourcecode 의 git 같은 존재. DB Schema (구조) 가 변경될 때마다 그 기록을 버전별로 저장하고 관리하는 시스템.

DB 설계도가 어떻게 변해왔는지 기록하는 일기장.

Prisma 는 `prisma migrate` 명령어를 통해 이를 수행

## Migration 이 왜 필요한가

혼자 개발할 때는 DB 를 껐다 키거나 초기화해도 되지만, 팀 단위 프로젝트에서는 불가

* 변경 기록 추적 : 투가, 언제, 왜 테이블 구조를 바꿨는지 알 수 있음
* 팀원 간 동기화 : 내가 user table 에 `phone` column 추가하면 팀원들도 내 migration 파일을 받아 실행하면 똑같은 DB 구조를 갖게 됨
* 안전한 배포 : local 에서 테스트한 DB 변경 사항을 Production 에 안전하고 동일하게 적용할 수 있음

## Prisma Migrate

1. Schema 수정

`schema.prisma` 파일에서 모델 수정

```
model User {
  id    Int    @id @default(autoincrement())
  name  String
  age   Int?   // 새로운 필드 추가
}
```

2. Migration 생성 및 실행

```sh
npx prisma migrate dev --name add_age_to_user
```

- SQL 생성됨 : Schema 차이점을 분석해 `CREATE TABLE`, `ALTER TABLE` 같은 실제 SQL 파일을 만듦
- DB 반영 : 생성된 SQL 을 현재 연결된 DB 에 실제로 실행

3. 결과물 확인

`prisma/migrations` 폴더 안에 timestamp 와 함께 만든 SQL 파일이 저장됨. 파일을 git에 commit 해서 공유함

## 주요 명령어

|**명령어**|**용도**|**사용 시점**|
|---|---|---|
|**`migrate dev`**|마이그레이션 생성 + DB 반영|**개발 중** 스키마를 바꿀 때|
|**`migrate deploy`**|기존 마이그레이션 파일만 실행|**배포 서버**에 변경사항 적용 시|
|**`migrate reset`**|DB를 완전히 밀고 새로 시작|데이터가 꼬여서 초기화가 필요할 때|
|**`db push`**|마이그레이션 기록 없이 DB 반영|프로토타이핑 등 기록이 필요 없을 때|

## Best Practices

1. 데이터 유실 주의 : 필수 (Required) 필드를 새로 추가할 때 기존 데이터가 있다면 migration 이 실패하거나 데이터가 날아갈 수 있음. `default` 값을 주거나 `Optional` 로 먼저 만든 뒤 데이터를 채워야 함
2. Migration 파일은 수정 금지 : 생성된 SQL 파일을 직접 수정하는 것은 위험. `schema.prisma` 를 고치고 새로 migration 을 생성
3. Git 관리 : `prisma/migrations` 폴더는 반드시 git 에 포함시켜야 함

## PostgreSQL

전세계에서 가장 진보되고 널리 쓰이는 오픈 소스 관계형 데이터베이스 (RDBMS)

데이터를 table 혀태로 저장하고 SQL 을 통해 관리하는 시스템.

Prisma 는 단순히 ORM 이고, 실제 db 는 PostgreSQL.

```sh
brew install postgresql@17 && brew services start postgresql@17
```

# 실습

```sh
brew install postgresql@17
brew services start postgresql@17

npm install prisma -D
npm install @prisma/client

npm install @prisma/adapter-pg pg
npm install -D @types/pg
  
npx prisma init
```

`lib/prisma.ts`

```ts
import { PrismaClient } from '../generated/prisma';
import { PrismaPg } from '@prisma/adapter-pg';
import 'dotenv/config';

const adapter = new PrismaPg({ connectionString: process.env.DATABASE_URL });

// Prisma Client 인스턴스 생성
const prisma = new PrismaClient({ adapter });

export default prisma;
```

Why a database server?
  PostgreSQL is a server database — it runs as a background process that manages data, handles multiple connections, enforces permissions, etc. It's more powerful than a simple file, but requires a running process.
   Think of it like how your iOS app connects to a backend server — PostgreSQL is that server, but for data.


PostgreSQL 은 db 단에서의 transaction 과 lock 을 통해 data 가 corrupt 되는 것을 방지함. (db-level conflict 를 방지)

  SQLite is different — it's just a file with no server. 

Why PrismaPg?
  Prisma v7 changed how it connects to databases. Previously Prisma handled the connection internally. Now in v7, you provide the connection via an "adapter" — a small bridge between Prisma and the actual database
  driver. PrismaPg is that bridge for PostgreSQL, using the pg package (the standard Node.js PostgreSQL driver) under the hood.

It's essentially:
  Your code → Prisma → PrismaPg adapter → pg driver → PostgreSQL server

  This is a Prisma v7 specific thing — older tutorials won't mention it because it didn't exist before.