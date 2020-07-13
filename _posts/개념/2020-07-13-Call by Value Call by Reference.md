---
title:  "Call by value와 Call by reference의 차이"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - 개념
  
tags:
  - 공부
  - Call by value
  - Call by reference
last_modified_at: 
---

Call by value와 Call by reference의 차이
=============================

함수 호출 방법은 크게 두 가지가 있다.

1. Call by value(값에 의한 호출)
2. Call by refernce(참조에 의한 호출)

많은 교재에서는 그림으로 예시를 들고 있다. 예를 들어, 컵에 물을 채워서 이 물을 직접 가져다가 다른 컵에서 사용하냐,
아니면 똑같은 컵과 물을 한 컵 더 준비를 하여 사용을 하느냐의 식이다.

**Call by value(값에 의한 호출)**은 인자로 받은 값을 복사하여 처리를 한다.
**Call by reference(참조에 듸한 호출)**은 인자로 받은 값의 주소를 참조하여 직접 값에 영향을 준다.

<U>즉 값을 복사를 해서 처리하냐, 아니면 직접 참조를 하느냐의 차이이다.</U>

Call by value를 하게 되면 프로그래밍 구조상 복사를 하기 때문에 메모리량이 늘어난다.

만약 이때 많은 계산이 들어가게 된다면 과부하가 일어나게 된다. 하지만 복사처리가 되기 때문에 원래의 값은 영향을 받지 않아 안전하다.

## Call by value(값에 의한 호출)

* 장점 : 복사하여 처리하기 때문에 안전하다. 원래의 값이 보존된다.
* 단점 : 직접 참조를 하기 때문에 메모리가 사용량이 늘어난다.

대표적인 예로는 void swap(int a, int b)가 있다.

```C++
#include <stdio.h>
 
void swap(int num1, int num2)
{
    int temp = num1;
    num1 = num2;
    num2 = temp;
}

void main()
{
    int a = 20, b = 60;
    swap(a, b);
    printf("a: %d, b: %d", a, b);
}
```

이러면 결과로 a는 20, b는 60이 나온다.

원하는 결과가 나오지 않은 모습이다.

![1](https://user-images.githubusercontent.com/41438361/87274841-b03e1600-c517-11ea-804d-861255b66975.png)

위의 그림을 보면 이해가 더 쉽다.

swap()을 호출하게 되면 a와 b의 값만 받아와서 내부적으로 처리를 하고 아무 것도 넘기지 않는다.

변수를 주소로 가져오거나 포인터를 통해서 가져온 것이 아니기 때문에, 새로운 변수를 만들어서 값을 대입하여 처리한 것이다.

이 경우 교체는 되지 않고 swap() 내부에서만 처리가 되게 된다.



## Call by reference(참조에 의한 호출)

* 장점 : 복사하지 않고 직접 참조를 하므로 빠르다.
* 단점 : 직접 참조를 하기 때문에 원래 값이 영향을 받는다.

```C++
#include <stdio.h>
 
void swap(int &num1, int &num2)
{
    int temp = num1;
    num1 = num2;
    num2 = temp;
}

void main()
{
    int a = 20, b = 60;
    swap(a, b);
    printf("a: %d, b: %d", a, b);
}
```

![2](https://user-images.githubusercontent.com/41438361/87274967-0b700880-c518-11ea-939d-e41a35ddf1f4.png)

위에서는 main의 a, b의 주소를 가져와서 처리했다.

직접 주소를 가져와서 처리를 했기때문에 a, b의 값이 교체가 된 것을 볼 수 있다.

이런 방법에서는 주소나 포인터를 사용하여 직접 변수에 접근하기 때문에 리스크가 있다.


