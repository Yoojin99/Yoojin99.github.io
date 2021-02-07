---
title:  "Stack"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - cs
  
tags:
  - cs-ds
  
last_modified_at: 2020-11-03
---

## Stack

Stack은 선형 자료구조의 일종으로 LIFO(Last in first out)이다. 

### python

python에서는 나는 list로 stack을 구현했다.

참고로 slice, pop, del을 모두 사용해서 pop을 구현할 수 있긴 하다.

* remove() : 지우고자 하는 인덱스가 아니라 값을 입력한다. 지우고자 하는 값이 2개 이상이 있으면 가장 앞에 있는 값을 지운다.
* pop(), del : 리스트의 인덱스를 받아서 지운다. pop()은 지워진 인덱스의 값을 반환하지만 del은 반환하지 않는다.
  이 때문에 del이 pop()보다 수행속도가 더 빠르다. remove()와 동일하게 pop()과 del은 특정 인덱스를 삭제하고 리스트를 재조정한다.

  또한 del은 pop()과 다르게 리스트의 범위를 지정해서 삭제할 수 있다. 
* slice : 슬라이싱은 원본 리스트의 값을 그대로 유지하면서 원하고자 하는 범위만큼 출력할 수 있다.

del<remove<pop<slice 순으로 실행할 때 시간이 소요된다. del이 가장 빠르다.

### C++

stack의 기본 함수는 다음과 같다.

* 추가 및 삭제
  * push(element) : top에 원소를 추가
  * pop() : top에 있는 원소를 삭제
* 조회
  * top() : top에 있는 원소를 반환
* 기타
  * empty() : 스택이 비어있으면 true, 아니면 false를 반환
  * size() : 스택 사이즈를 반환
  
```c++
#include <iostream>
#include <stack>

using namespace std;

int main() {
  stack <int> s; // 스택 생성
  
  s.push(1);
  s.push(2);
  s.push(3);
  
  cout<<"top element: "<<s.top()<<'\n'; //3
  
  s.pop(); //3 삭제
  s.pop(); //2 삭제
  
  cout<<"size : "<<s.size()<<'\n';
  
  cout<<"empty: "<<(s.empty() ? "Y" : "N")<<"\n";
  
  return 0;
}
```


## Queue

Queue는 선형 자료구조의 일종으로 FIFO(First int first out)이다. Java Collection에서 queue는 인터페이스이다. 

### C++

queue의 기본 함수는 다음과 같다.

* 추가 및 삭제
  * push(element) : 큐에 원소를 추가
  * pop() : 큐에 있는 원소를 삭제(앞의 원소)
* 조회
  * front() : 큐 제일 앞에 있는 원소를 반환
  * back() : 큐 제일 뒤에 있는 원소를 반환
* 기타
  * empty() : 큐가 비어있으면 true, 아니면 false를 반환
  * size() : 큐 사이즈를 반환
  
```c++
#include <iostream>
#include <queue>

using namespace std;

int main() {

  queue<int> q; //큐 생성
  
  q.push(1);
  q.push(2);
  q.push(3);
  
  q.pop();
  q.pop();
  
  cout<<"front : " <<q.front()<<'\n';
  
  cout<<"back: " << q.back() << '\n';
  
  cout<<"size: "<<q.size()<< '\n';
  
  cout << "empty : " << (q.empty()? "Y" : "N") << '\n';
  
  return 0;
}
```
