
# HTTP Method

Hypertext Transfer Protocol : Client 가 Server 에 '내가 지금 무엇을 하려고 하는지' 알리는 명령어

|**메서드**|**역할**|**주요 특징**|
|---|---|---|
|**GET**|데이터 **조회**|서버에서 정보를 가져올 때 사용 (게시글 읽기 등)|
|**POST**|데이터 **생성**|서버에 새로운 데이터를 저장할 때 사용 (회원가입, 글쓰기)|
|**PUT**|데이터 **수정(전체)**|리소스를 새로운 데이터로 완전히 갈아치울 때 사용|
|**PATCH**|데이터 **수정(일부)**|리소스의 특정 부분만 변경할 때 사용|
|**DELETE**|데이터 **삭제**|특정 리소스를 삭제할 때 사용|

* GET
	* 데이터를 보낼 때 URL 뒤에 query string `?id=123` 을 붙여서 보냄
	* 브라우저에 결과가 캐싱돼서 속도가 빠름
* POST
	* 데이터를 Message Body 에 숨겨서 보냄 (GET 보다 보안상 유리, 대용량 전송 가능)
	* 서버의 상태를 변경시키기 때문에 결과가 캐싱되지 않음
* PUT
	* 일부 필드를 누락해서 보내면 그 필드는 삭제되거나 기본값으로 덮어 씌워질 수 있음
* PATCH
	* 리소스의 일부분만 수정하기 때문에 훨씬 효율적, 안전
* DELETE

* 명등성 (Idempotent) : 연산을 여러 번 적용해도 결과가 달라지지 않는 성질.
	* O : GET, PUT, DELETE
	* X : POST

# Status Code

Server 가 Client 의 요청에 대해 '그 요청 어떻게 처리됐어' 하고 알려주는 3자리 숫자 응답

|**클래스**|**의미**|**설명**|
|---|---|---|
|**1xx**|Informational|요청이 수신되어 처리 중 (실무에서 자주 보긴 어렵습니다)|
|**2xx**|Successful|**성공!** 요청이 정상적으로 처리됨|
|**3xx**|Redirection|**이동!** 요청을 완료하려면 다른 주소로 가야 함|
|**4xx**|Client Error|**너가 잘못했어!** 클라이언트 측의 요청 오류|
|**5xx**|Server Error|**내가 잘못했어!** 서버 측의 오류|

* ✅ 2xx (성공)
	* **200 OK**: 요청이 아주 성공적으로 처리됨. (가장 많이 보는 코드)
	* **201 Created**: 요청이 성공해서 새로운 리소스가 생성됨. (POST 요청 후 결과)
	* **204 No Content**: 요청은 성공했지만, 응답으로 보내줄 데이터는 없음.
* 🔄 3xx (리다이렉션)
	* **301 Moved Permanently**: 요청한 리소스의 URL이 영구적으로 변경됨.
	* **302 Found / 307 Temporary Redirect**: 요청한 리소스가 일시적으로 다른 URL에 있음.
	* **304 Not Modified**: 캐시를 사용하라는 뜻. 클라이언트가 가진 파일이 최신이니 다시 다운로드할 필요 없음.
* ❌ 4xx (클라이언트 오류)
	* **400 Bad Request**: 요청 자체가 문법적으로 틀림.
	* **401 Unauthorized**: 인증되지 않음. (로그인 필요)
	* **403 Forbidden**: 권한이 없음. (로그인은 했지만 접근 불가)
	* **404 Not Found**: 요청한 리소스를 찾을 수 없음. (주소 오타 등)
* 💥 5xx (서버 오류)
	* **500 Internal Server Error**: 서버 내부 문제로 에러 발생. (가장 흔한 서버 에러)
	* **502 Bad Gateway**: 게이트웨이 역할을 하는 서버가 뒤쪽 서버로부터 잘못된 응답을 받음.
	* **503 Service Unavailable**: 서버가 일시적으로 과부하되었거나 점검 중임.

# Header, Request Body, Query Parameters

HTTP 메세지는 header / body 로 나뉘고, 데이터를 전달하는 방식에 따라 query parameter 를 사용하기도 함

택배 비유

* Query parameter : 택배 박스 겉면에 적힌 메모
* Header : 택배 운송장
* Body : 택배 박스 안의 실제 물건

## Query Parameter

URL 끝에 `?` 를 붙이고 `key=value` 형식으로 데이터 전달. **데이터를 필터링, 정렬, 검색**할 때 사용

* 형태 : `https://example.com/products?category=shoes&color=blue`
* 특징 : URL 에 데이터가 그대로 노출됨
	* 보안이 중요하지 않은 단순 정보 전달에 사용됨
	* 길에 제한 있음
	* GET 에서 주로 사용됨

## HTTP Header 

보내는 사람, 받는 사람, 물건의 종류, 주의사항 등 **데이터에 대한 메타데이터**를 담고 있음

* 주요 정보
	* Content-Type : body 에 담긴 데이터의 형식 (e.g. `application/json`, `text/html`)
	* Authorization : 사용자 인증을 위한 토큰 / 키
	* User-Agent : 어떤 브라우저 / 기기에서 요청했는지 정보
	* Cookie : 브라우저 저장된 세션 정보 등

## Request Body

서버로 보내려면 진짜 데이터가 들어있는 곳

* 형태 : 주로 JSON
* 특징
	* URL 에 노출되지 않아 Query Parameter 보다 안전하지만 HTTPS 암호화가 필요
	* 대용량 데이터를 보낼 수 있음
	* POST, PUT, PATCH 메서드에서 주로 사용

