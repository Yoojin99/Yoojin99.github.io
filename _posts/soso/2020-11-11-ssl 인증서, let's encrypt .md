---
title:  "let's encrypt, ssl 인증서, redirect"
excerpt: ""

categories:
  - soso
tags:
  - ssl 인증서
  - let's encrypt
last_modified_at: 2020-11-11
---

[잘 정리된 블로그 글](https://12bme.tistory.com/80)

## SSL 인증서란? 

정보의 양이 굉장히 방대해지면서 기밀을 유지해야 하는 정보들도 주고 받게 되었고, 이를 보호하는 것이 필요해지게 되었다.

이런 보안 정보들을 주고 받기 위해 있는 것이 SSL 프로토콜이다. SSL(Secure Socket Layer) 프로토콜은 처음에 Netscape사에서 웹 서버와 브라우저 사이의 보안을 위해 만들었다.
SSL은 Certificate Authority(CA)라 불리는 third party로부터 서버와 클라이언트의 인증을 하는데 사용된다.

HTTP는 Hypertext Transfer Protocol의 약자이다. 즉 Hypertext인 HTML을 전송하기 위한 통신규약을 의미한다. 
HTTPS는 마지막의 S는 Over Secure Socket Layer의 약자로 보안이 강화된 HTTP라고 생각하면 된다. HTTP는 암호화되지 않은 방법으로 데이터를 전송하기 때문에 서버와 클라이언트가 주고 받는 메세지를 몰래 빼돌리거나 해킹하는 것이 매우 쉽다.

정리하면 HTTPS는 SSL 프로토콜 위에서 돌아가는 프로토콜을 의미한다.

SSL(Secure Socket Layer)와 TLS(Transport Layer Security Protocol)은 같은 의미이다. 네스케이프에 의해 SSL이 발명되었고, 이것이 
폭넓게 사용되다가 표준화 기구인 IETF의 관리로 변경되면서 TLS라는 이름으로 바뀌었다. TLS 1.0은 SSL 3.0을 계승한다.

SSL 과 SSL 디지털 인증서를 이용하면 다음과 같은 장점이 있다.

* 통신 내용이 공격자에게 노출되는 것을 막을 수 있다.
* 클라이언트가 접속하려는 서버가 신뢰할 수 있는 서버인지 판단할 수 있다.
* 통신 내용의 악의적인 변경을 방지할 수 있다.

**인증서의 역할은 클라이언트가 접속한 서버가 클라이언트가 의도한 서버가 맞는지 보장한다.** 이 역할을 하는 민간기업들이 
있는데 이런 기업들을 CA(Certificate authority)/Root Certificate 라고 한다. CA는 아무 기업이나 할 수 있는 것이 아니고 신뢰성이공인된 기업들만 참여할 수 있다.

SSL 인증서에는 다음의 정보가 포함되어 있다.

1. 서비스의 정보(인증서를 발급한 CA, 서비스의 도메인 등)
2. 서버 측 공개키(공개키의 내용, 공개키의 암호화 방법)

이런 SSL 인증서를 발급해주는 CA들이 여러개 있다. 그 중에서도 무료로 SSL 인증서를 발급해주는 사이트인
`Let's encrypt`라는 사이트가 있다. 

[참고 글](https://techmonger.github.io/46/free-ssl-google-cloud/)

## SSL 인증서 발급받기

우선 내가 실행시켰던 환경은 Google Cloud Platform의 Compute Engine > VM 인스턴스의 SSH shell로 실행시켰다.

참고로 GCP에서 VM 인스턴스의 ssh shell을 실행시키는 방법은 아래의 화면에서 초록색으로 표시된 부분을 클릭하면 실행된다.

![image](https://user-images.githubusercontent.com/41438361/98773492-65340100-242c-11eb-962e-0c269d99aad5.png)

우선 아파치 configuration 파일들을 백업한다. 이제부터 아래에 나오는 명령어들은 앞에 웬만해서 sudo를 붙여서 실행해주자.

```
mkdir /tmp/apache_config_backup/
cp -r /etc/apache2/* /tmp/apache_config_backup/
```

그리고 SSL 인증서를 발급받기 위한 certbot client를 다운받는다.

```
sudo apt-get install python-certbot-apache
```

마지막으로 apache 서버에서 동작하는 인증서를 받는다.

1. certificate 설치 프로그램을 작동시킨다.
  ```
  sudo certbot --authenticator webroot -installer apache
  ```
  
  위와 같이 명령어를 치면 apache 서버가 실행될 것이다.
  
2. 이메일 주소를 입력한다.
3. a를 눌러 수락
4. y를 누른다.
5. ssl 인증서를 받고자 하는 도메인 주소를 입력한다. (example.com)
6. web server root를 입력한다. (/var/www/html) - 그대로 입력해도 잘 작동했다.
7. http -> https redirect를 할 건지 보고 1/2를 선택한다.
8. 잘 되었으면 아래와 같이 성공적으로 인증서 발급이 되었다는 메세지가 뜰 것이다.
  ![image](https://user-images.githubusercontent.com/41438361/98774194-dc1dc980-242d-11eb-9058-e9289781437d.png)

## redirection 설정하기

내가 하고자 했던 것은 이전의 더 이상 지원하지 않는 구 홈페이지에 접속한 사용자들도 새롭게 만들어진 홈페이지로 
redirect 시켜서 신 홈페이지로 이동할 수 있도록 하는 것이었다.

즉, http://A나 https://A로 접속하는 사용자들이 https://B로 redirect 될 수 있도록 하는 것이었다.

### http://A to https://B

우선, http://A로 접속한 사용자들이 https://B로 redirect 될 수 있게 하는 방법은 다음과 같다.

먼저 `/var/www` 경로로 이동을 해주자.

```
cd /var/www
```

그리고 설정파일에서 설정을 해준다. 나는 apache 버전이 2.4.38로 높아서 가능했는데, apache 버전이 낮으면 내가 한 방법대로 되지 않을 수도 있다.

참고로 apache 버전을 확인하려면 아래의 커맨드를 입력해주면 된다.

```
dpkg -l | grep apache
```

![image](https://user-images.githubusercontent.com/41438361/98775294-2d2ebd00-2430-11eb-8d30-ae450659d0c3.png)

```
sudo vi /etc/apache2/sites-available/000-default.conf
```

를 치면 아래와 같이 파일의 내용을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/98775770-1f2d6c00-2431-11eb-9c0d-a90bf8f01d5e.png)

초록색으로 표시된 부분에 redirect 되었으면 좋겠는 주소(B)를 써주자. 참고로 맨 위의 `VirtualHost *:80`은 80포트(http)를 listen하고 있다는 뜻이다.
즉 http로 들어온 것을 listen해서 특정 작업을 하겠다는 뜻이다.

### https://A to https://B

위에서 인증서를 발급했을 때, ssl 관련 파일이 하나 더 생겼을 것이다. 파일 이름을 다 쓰기 귀찮을 때는 tab을 눌러서 자동완성을 시켜주자.

아래의 커맨드를 입력해준다.

```
sudo vi /etc/apache2/sites-available/000-default-le-ssl.conf
```

마찬가지로 파일의 내용을 아래와 같이 확인할 수 있다. 여기서는 맨 위에 `VirtualHost *:443`이라고 되어 있는 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/98776276-f35eb600-2431-11eb-80ca-28dc3125292d.png)

맨 아래의 초록색 부분(주소를 입력하는 부분)에 마찬가지로 새로 redirect 하고 싶은 주소(B)를 적어준다.

이러면 세팅 끝이다.

세팅을 완료하고

```
sudo systemctl restart apache2
```

를 입력해서 apache 서버를 재시작해주자. 이제 http://A 나 https://A로 접속하면 https://B로 접속이 가능해진다! (http://B 에서 https://B는 이미 redirect가 되어 있게 설정되어 있었다.)









