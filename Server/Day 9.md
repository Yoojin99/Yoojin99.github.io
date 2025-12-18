
# multipart/form-data

웹에서 파일 (이미지, 비디오, 문서 등)을 서버로 전송할 때 사용하는 HTTP 요청의 인코딩 방식

일반적인 test form data (`application/x-www-form-urlencoded`) 와는 데이터를 쪼개서 보내는 방식 자체가 다름

## 왜 사용하는가

일반적인 폼 전송 방식은 데이터를 `key=value&key=value` 형태의 긴 문자열로 보냄. 

Binary 데이터 (파일) 은 문자열 형태에 적합하지 않고, 크기 큼.

`multiplart/form-data` 은 여러 종류의 데이터 (텍스트+파일) 을 part 로 나눠 한 번에 보낼 수 있게 함

## 구조

핵심은 `boundary` 라는 고유한 식별자. 서버는 식별자를 보고 '여기서부터 파일 데이터고 여기서부터는 텍스트구나' 를 구분

```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=AaB03x

--AaB03x
Content-Disposition: form-data; name="username"

yj
--AaB03x
Content-Disposition: form-data; name="profile_pic"; filename="me.png"
Content-Type: image/png

(바이너리 이미지 데이터...)
--AaB03x--
```

## Backend 에서 처리 (Express + Multer)

Node.js(Express) 서버는 기본적으로 `multiplart/form-data` 를 해석하지 못함. `multer` 같은 middleware 를 사용해야 함

```sh
npm install multer
npm install -D @types/multer
```

```ts
import express from 'express';
import multer from 'multer';

const app = express();
const upload = multer({ dest: 'uploads' }); // 파일이 저장될 경로

// 'profile'은 client 가 보낸 form data 의 key 이름
app.post('/upload', upload.single('profile'), (req, res) => {
	console.log(req.file); // upload 된 파일 정보
	console.log(req.body); // 함께 보낸 텍스트 데이터
	res.send('업로드 성공!');
});
```

## Frontend 에서 보내기 (FormData)

```js
const formData = new FormData();
const fileField = document.querySelector('input[type="file"]');

formData.append('username', 'yj');
formData.append('profile', fileField.files[0]);

fetch('/upload', {
  method: 'POST',
  body: formData, // 브라우저가 자동으로 Content-Type과 Boundary를 설정합니다.
});
```

## 주의사항

1. Header 설정 금지 : `fetch` 나 `axios` 를 쓸 때 `Content-Type: multipart/form-data` 를 수동으로 적으면 안됨. 브라우저가 `boundary` 값을 포함해 자동으로 생성해야 해서 수동으로 적으면 오히려 서버가 데이터를 해석하지 못함
2. 보안 : 파일 업로드 기능을 서버 용량을 채우거나 악성 스크립트를 올리는 등 공격의 대상이 되기 쉽기 때문에 반드시 파일 확장자 검사, 용량 제한 설정을 해야 함

# multer for handling file uploads

Multer middleware 를 사용해서 들어오는 파일을 가공해 `req.file` / `req.files` 에 담아주고, 지정한 위치에 파일을 저장해주는 역할을 함

## Multer 설치

```sh
npm install multer
npm install -D @types/multer
```

## 설정 : 어디에 저장할 것인가

* 저장 방식 2가지
	1. 폴더 지정
	2. 파일명까지 조정하는 `diskStorage` 방식

### 폴더 지정 (Dest)

```ts
import multer from 'multer';

const upload = multer({ dest: 'uploads/' }); // 파일이 'uploads' 폴더에 저장됨 (파일명은 임의로 변경됨)
```

### 파일명까지 지정 (Storage)

파일의 원래 이름을 유지/중복 방지를 위해 prefix 를 붙이고 싶을 때 사용

```ts
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/'); // 저장할 폴더
  },
  filename: (req, file, cb) => {
    // 파일명: 현재시간-원본이름
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, uniqueSuffix + '-' + file.originalname);
  }
});

const upload = multer({ storage: storage });
```

## Router 에서 사용

Upload 방식에 따라 middleware 함수가 달라짐

### 단일 파일 업로드 (`.single`)

```ts
// 'profileImage'는 프론트엔드에서 보낸 폼 데이터의 key 이름입니다.
app.post('/profile', upload.single('profileImage'), (req, res) => {
  // req.file: 업로드된 파일 정보
  // req.body: 함께 보낸 텍스트 필드들
  console.log(req.file);
  res.send('파일 업로드 완료!');
});
```

### 여러 파일 업로드 (`.array` / `.fields`)

```ts
// 같은 이름의 파일 여러 개
app.post('/photos', upload.array('photos', 5), (req, res) => {
  // req.files에 배열로 담깁니다.
});

// 서로 다른 필드 이름의 파일들
app.post('/register', upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]), (req, res) => {
  // req.files.avatar, req.files.gallery로 접근 가능
});
```

### 파일 필터링 및 용량 제한

서버 성능, 보안 위해 필요

```ts
const upload = multer({
  storage: storage,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB 제한
  },
  fileFilter: (req, file, cb) => {
    // 이미지 파일만 허용
    if (file.mimetype.startsWith('image/')) {
      cb(null, true);
    } else {
      cb(new Error('이미지만 업로드 가능합니다!'), false);
    }
  }
});
```

### Prisma 와 연결

서버 local folder 에 파일을 저장하고 그 파일의 경로 (URL) 을 DB 에 저장

```ts
app.post('/upload', upload.single('image'), async (req, res) => {
  if (!req.file) return res.status(400).send('파일이 없습니다.');

  // DB에는 파일 자체가 아니라 '경로'를 저장합니다.
  const post = await prisma.post.create({
    data: {
      title: req.body.title,
      imageUrl: `/uploads/${req.file.filename}`, // 저장된 경로
    }
  });

  res.json(post);
});
```

# Serving static files with express.static()

파일을 서버에 저장했다면 파일을 브라우저에서 볼 수 있도록 Serve 해야 함. Express 에서 `express.static()` middleware 를 사용해서 할 수 있음

## 기본 사용법

서버의 특정 폴더를 외부에서 접근 가능한 'static folder' 로 지정.

```ts
import express from 'express';
import path from 'path';

const app = express();

// 'uploads' 폴더 안에 있는 파일들을 루트 경로(/)에서 접근 가능하게 만듭니다.
app.use(express.static('uploads'));

app.listen(3000);
// uploads/me.png 라는 파일이 있을 때, browser 에서 http://localhost:3000/me.png 로 접속 가능
```

## 가상 경로 설정

실제 폴더 구조, URL 경로를 다르게 설정하고 싶을 때 사용. 보안상 폴더 이름을 숨기거나, URL 을 더 명확하게 만들 때 유용

```ts
// '/public'이라는 가상 경로를 통해 'uploads' 폴더에 접근합니다.
app.use('/public', express.static('uploads'));

// http://localhost:3000/public/me.png
```

## 절대 경로 사용 (가장 안전)

서버를 실행하는 위치 (Working directory) 가 달라지면 상대 경로가 꼬일 수 있음. 이를 방지하기 위해 노드의 `__dirname`, `path` 모듈을 사용하는 것이 표준

```ts
import path from 'path';

// 현재 파일 위치를 기준으로 절대 경로를 생성합니다.
app.use('/static', express.static(path.join(__dirname, 'uploads')));
```

## 전체 흐름

1. Multer : 파일을 `uploads/123.jpg` 로 저장
2. Prisma : DB 에는 `/static/123.jpg` 라는 URL 을 저장
3. Static : 클라이언트가 URL 로 요청하면 Express 가 `uploads/123.jpg` 파일을 보내줌

## 주의사항

* 보안 : `express.static()` 은 지정된 폴더 안의 모든 파일을 누구나 볼 수 있게 공개하기 때문에 민감한 정보가 담긴 파일은 이 폴더에 넣으면 안됨
* 다중 설정 : `app.use()` 를 여러 번 써서 여러 개의 폴더를 정적으로 지정할 수 있음
```ts
app.use('/images', express.static('public/images'));
app.use('/css', express.static('public/css'));
```

