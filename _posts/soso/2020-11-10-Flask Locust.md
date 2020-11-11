---
title:  "Flask, locust"
excerpt: ""

categories:
  - soso
tags:
  - flask
  - server
last_modified_at: 
---

파이썬으로 서버를 구축할 수 있는 아주 좋은 웹 어플리케이션 프레임워크가 있다. 바로 Flask이다.

![image](https://user-images.githubusercontent.com/41438361/98616693-acdd5e80-2340-11eb-8ed4-3b5ae34b4012.png)

[Flask 공식 페이지](https://flask.palletsprojects.com/en/1.1.x/)

코드도 정말 간단하다. 

QuickStart의 첫번째 예제를 보면 다음과 같이 나와있다.

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

위에서 hello_world라는 함수는 client가 server에 요청을 할 때(GET, POST 등등) url 뒤의 주소가 끝나는(뒤에 덧붙여진)
형식에 따라 다른 동작을 할 수 있도록 설정된 것이다.

annotation으로 각 함수가 어떤 주소에 매핑되는지 설정한다. 위 코드의 '/' 부분에는 정규표현식이 올 수도 있다.

```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```

처럼 get, post일 때 처리하는 방법을 다르게 설정해 줄 수도 있다. 정말 간단하게 들어오는 요청에 따라 다르게 동작을 처리할 
수 있다는 점이 좋은 것 같다.

이렇게 flask를 이용해서 서버를 구축한다고 했는데, 좀 더 자세히 적자면 사용자가 늘어날 수록 flask 인스턴스를 여러개 생성하게 된다.
정확히 어디에서(OS단?) 어떤 원리로 이 인스턴스가 알아서 늘어나고 줄어드는지는 확인을 해봐야 할 것 같다. 
사용자가 많아질 수록 인스턴스를 많이 생성해서 더 많은 요청을 처리할 수 있다고 했는데, 이렇게 인스턴스를 늘려서
많아진 request들을 처리하는 방식이 수평 scaling이라고 한다. CPU 등의 성능을 좋게 만들어 서버가 구동되는 환경을 좋게 만들어
더 많은 요청을 처리할 수 있게 하는 것은 수직 scaling이라고 한다.

이렇게 서버를 만들면 서버가 얼마나 견디는지, 즉 얼마나 많은 사용자의 request를 처리할 수 있는지 test 해봐야한다.

이것을 stress test, 부하 테스트라고 한다. 예를 들어 1초에 얼마나 많은 요청까지 견딜 수 있는지 테스트하는 것이다.
이를 또 도와주는 python 오픈 소스 툴이 있다. 바로 Locus이다.

[Locust 공식 페이지](https://locust.io/)

이 locust는 사용자의 접속이 굉장히 많은 것처럼(request를 많이 생성할 수 있다) 서버에 요청을 보낼 수 있고, 
서버가 견딜 수 있는지 테스트할 수 있게 도와준다.

