---  
layout: post  
title: "URL의 구성요소, Host name, Port name"  
subtitle: ""  
categories: cs
tags: cs-cn url, host, port
comments: true  
header-img: 
---  
  
> `URL의 구성요소를 알아보자.`  

---

## Components of URL

URL(Uniform Resource Locator)는 URI(Universal Resource Identifier)의 한 특정한 타입이다. URL은 주로 인터넷에 존재하는 resource의
위치를 나타낸다. URL은 웹 클라이언트가 서버에 resource를 요청할 때 주로 사용된다.

이 글에서는 URL, URI에 대해 요약할 것이다. 더 자세한 URI와 URL 개념은 Internet Society와 IETF(Internet Engineering Task Force) Request
for Comments document RFC 2396에 정의되어 있으니 이를 참고하면 된다. [링크](https://www.ietf.org/rfc/rfc2396.txt)

**간단하게 URI는 resource를 구별하는 문자열이다. URL는 resource의 이름이나 다른 특성이 아닌 그것의 위치나 접근하는데 사용하는 방법에 의해 정의되는
URI다.**

Resource 식별자의 새로운 형태인 IRI(Internationalized Resource Identifier)는 English 말고도 적합한 언어의 문자와 형태를 사용하는 것을 허용한다. IRI는
애플리케이션이 IRI를 지원하는 Request와 response와 관련이 있을 때 URI나 URL을 대체해서 사용할 수 있다. IRI에 대한 더 많은 정보는 [Internationalized Resource Identifiers (IRIs)](https://www.ibm.com/docs/en/cics-ts/5.2?topic=cics-internationalized-resource-identifiers-iris)
에서 확인할 수 있다.

 HTTP나 HTTPs를 위한 URL은 보통 3~4 개의 요소로 이루어져 있다.
 
 1. **Scheme**. Sheme은 인터넷의 resource에 접근하기 위해 사용되는 프로토콜을 정의한다. HTTP(SSL 없는)가 될 수도 있다. HTTPS(SSL 있음)가 될 수도 있다.
 2. **Host**. Host 이름은 resource를 가지고 있는 주체를 정의한다. 예를 들어, `www.example.com`이 될 수 있다. 서버는 host의 이름 내에서 서비스를 제공하지만, host와 서버는 일대일 관계를 가지지는 않는다. Host는 Host name을 의미한다고 생각하면 된다. Host name뒤에는 **port number**가 붙을 수 있다. 유명한 port number는 보통 URL에서 빠지게 된다. 대부분의 서버는 HTTP와 HTTS의 유명한 port 번호를 사용하기 때문에 대부분의 http URL들은 port 번호를 자기 안에 포함시키고 있지 않다.
 3. **Path**. Path은 웹 클라이언트가 접근하고 싶어하는 host 내의 특정 resource를 정의한다. 예를 들어 `/software/htp/cics/index.html`이 될 수 있다.
 4. **Query string**. 만약 query string이 사용되었다면 path 뒤에 오고, resource가 특정 목적에 의해 사용될 수 있다는 정보를 제공하게 된다. 예를 들어 검색을 위한 인자나 처리되어야 하는 데이터가 될 수 있다. query string은 주로 이름-값 의 쌍으로 이루어져 있다. 예를 들어 `term=blubird`와 같이 올 수 있다. 이름-값 쌍은 ampersand(`&`)에 의해 분리되고, `term=bluebird&source=browser-search`와 같이 올 수 있다.

URL의 scheme, host는 대소문자 구분을 하지 않는데, path와 query string은 대소문자를 구분한다. 일반적으로 URL은 모두 소문자로 쓴다.

URL의 요소들은 아래와 같이 형태로 제한되어 URL을 구성한다.

```
scheme://host:port/path?query
```

- scheme은 뒤에 콜론(`:`)이 붙고, 두 개의 forward slash가 붙는다.
- 만약 port 번호가 명시되어있다면 host 이름 뒤에 콜론으로 구분해서 명시한다.
- path 이름은 하나의 forward slash로 시작한다.
- 만약 query string이 있다면 물음표로 시작한다.

아래 그림은 HTTP URL의 문법을 나타낸 그림이다.

<img width="590" alt="image" src="https://user-images.githubusercontent.com/41438361/158112138-b4af03f4-3fe7-4cf3-ae3e-a8855f0d7ec0.png">

그리고 HTTP URL의 예시로 아래와 같은 URL이 있을 수 있다.

```
http://www.example.com/software/index.html
```

그리고 만약 port 번호가 있다면 URL은 아래와 같이 될 것이다.

```
http://www.example.com:1030/softward/index.html
```

URL은 뒤에 단위 식별자가 붙을 수 있다. URL과 단위 식별자를 구분하는 구분자는 `#` 문자다. 단위 식별자는 웹 브라우저나 막 받은 아이템의 function을 가리키는데 사용한다.
예를 들어 만약 URL이 HTML 페이지를 정의한다면 단위 식별자는 섹션의 ID를 사용해서 페이지의 섹션을 명시하는데 사용될 수 있다. 이 경우 웹 브라우저는 일반적으로
페이지의 해당 섹션을 보여주게 된다. 

FTP(File Transfer Protocol)이나 Gopher와 같은 다른 프로토콜도 URL을 사용한다. 이런 프로토콜에서 사용되는 URL은 HTTP에서 상요되는 것돠 다른 문법을 가질 수 있다.

## Host name

인터넷의 host, web site는 `www.example.com`과 같은 host 이름에 의해 정의된다. Host name은 domain name이라고 불릴 때도 있다. Host name은 IP 주소에 매핑되지만,
host name과 IP 주소는 일대일 관계를 가지지는 않는다.

Host name은 웹 클라이언트가 host에게 HTTP request를 생성할 때 사용된다. Request를 만드는 사용자가 host name을 명시하지 않고 서버 IP 주소를 명시할 수도 있지만,
일반적인 방법은 아니다. Host name은 숫자로 된 IP 주소보다 사용하기 더 편하다. 회사, 기관, 그리고 개개인은 사용자에게 잘 기억될 수 있을만한 host name을 골라
웹 사이트에 사용한다.

더 중요한 것은 현대의 HTTP 구현에서 HTTP request에서 host name을 사용하는 것은 아래의 결과를 초래한다.

- 많은 서버에 의해 제공될 수 있는 하나의 host 이름내의 서비스들은 다른 IP 주소를 가지게 된다.
- 많은 IP 주소를 가지는 하나의 서버는 많은 host에서 서비스를 제공할 수 있다. 이런 걸 virtual hosting이라고 한다.

Host 이름은 DNS 서버나 domain name server로 알려진 서버에서 IP 주소로 매핑된다. 더 광범위한 네트워크에서 많은 DNS 서버는 협력해서 host 이름과
IP 주소를 매핑해준다.

## Port number

서버에서 하나 이상의 user process는 같은 시간에 TCP를 사용할 수 있다. 각 프로세스와 관련된 데이터를 구분하기 위해 port 번호가 사용된다. Port 번호는 
16 비트이고, 최댓값으로 65535를 가질 수 있지만 실제로 사용되는 건 되게 작은 범위에 해당된다.

클라이언트가 처음 server process에 요청하면 연결을 활성화하기 위해 잘 알려진 port 번호를 사용할 수 있다. 유명한 port 번호는 IANA(Internet Assigned Numbers Authority)에 의해
인터넷에서 널리 사용되는 특정 서비스에 할당됐다. 유명한 port 번호는 0 ~ 1023의 범위를 가지고 있고, 그 예는 아래와 같다.

<img width="539" alt="image" src="https://user-images.githubusercontent.com/41438361/158114475-0afd1a5b-2e33-48f7-95ac-bc17c6dfaed4.png">

유명한 port들은 클라이언트와 서버 process들 사이의 연결 위해서만 사용된다. 연결이 된 후에 서버는 별도로 단기적으로 사용되는 port 번호를 할당한다. 
이런 ephemeral port 번호들은 unique한 port 번호이고, 프로세스들이 연결을 시작할 때 동적으로 할당된다. 그리고 통신이 끝나면 해제된다.




* 출처
* https://www.ibm.com/docs/en/cics-ts/5.2?topic=concepts-components-url
* https://www.ibm.com/docs/en/cics-ts/5.2?topic=concepts-host-names#dfhtl28
* https://www.ibm.com/docs/en/cics-ts/5.2?topic=concepts-port-numbers#dfhtl2d
