---  
layout: post  
title: "[iOS] - Build Time Analyzing, 빌드 시간 분석하기"  
subtitle: ""  
categories: app
tags: app-ios Xcode preview 
comments: true  
header-img: 

---  
  
> `빌드 시간을 분석하여 빌드가 오래 걸리는 부분을 찾아내 개선하자.`  

---

# 빌드 시간을 단축해야 하는 이유? (Keep the Build Fast)

빌드 시간을 단축해야 하는 이유는 무엇일까? 단순히 빌드에 소요되는 시간을 단축해서 작업 속도를 빠르게 하기 위함이기도 하지만, 더 자세하고 구체적인 이유들이 있다.

먼저 Continous Integration(지속적 통합)이라는 개념이 있다. 지속적 통합은 팀 멤버들이 그들의 작업을 자주 통합하는 소프트웨어 개발 방법으로, 주로 각 사람마다 하루에 최소 한 번 통합해서 하루에 여러번의 통합이 일어나는 것이다.
각 통합은 통합할 때의 에러를 최대한 빨리 탐지하기 위해 테스트를 포함한 자동화된 빌드에 의해 검증된다. 많은 팀들은 이런 접근 방식이 통합할 때의 문제를 줄여주고 팀이 결합력 있는 소프트웨어를 더욱 빠르게 개발하게 해준다는 것을 발견했다.

이 지속적인 통합에서 중요한 요소 중 하나는 빠른 피드백을 제공하는 것이다. CI(Continous Integration) 활동을 가장 저해하는 것은 빌드가 오래 걸리는 것이다. 여기에서 Martinfowler는 보통 빌드에 한 시간이 걸리면 납득할 수 없는 시간이라고 생각한다고 얘기한다.
하지만 이 빌드 속도를 가속화하는 것이 어려운 작업인 경우가 있다.

하지만 대부분의 프로젝트에서 10분 빌드에 대한 XP 가이드라인이 적용되는 것은 합리적이다. 대부분의 현대 프로젝트들이 이를 만족하고 있다. 빌드 시간을 일 분 줄이는 것은 각 개발자들이 매번 commit할 때마다 일 분씩 아끼게 해주는 것이기 때문에 빌드 시간을 줄이는 것은 의미가 있는 작업이다.
CI가 빈번한 commit을 요구하기 때문에, 빌드를 하는데 걸리는 시간도 늘어날 수밖에 없다.

아마 가장 중요한 단계는 deployment pipeline을 설정하는 단계일 것이다. Deployment pipeline(build pipeline/staged build)는 여러 번의 빌드가 순서대로 수행된다는 생각이 뒷받침하고 있다. Mainline에 commit을 하면 첫 번째 빌드를 실행시키는데, 이를 commit build라 한다.
Commit build는 누군가가 mainline에 필요한 빌드이다. Commit build는 빨리 완료되어야 하는 빌드이고, 결과적으로 버그를 찾을 확률이 낮아지게 될 것이다. 이는 버그를 찾는 수준을 조절해서 다른 사람이
계속 작업할 수 있을 만한 적당한 속도를 찾는 것이 중요하다.

Commit build가 성공적으로 완료되면 다른 사람들은 신뢰할 수 있는 기반에서 작업할 수 있다. 이것의 간단한 예시는 바로 두 단계 deployment pipeline이다. 첫 번째 단계는 컴파일을 하고 데이터 베이스가 완전히 제거된 상태에서 더 지역화 된 유닛 
이런 테스트는 굉장히 10분 가이드라인을 지키면서 굉장히 빠르게 실행될 수 있다. 하지만 더 넓은 스케일의 상호작용, 특히 실제 데이터베이스를 포함한 상황에서 발생할 수 있는 버그는 찾지 못할 것이다. 
두 번째 단계는 


한 개발자가 커밋 후 빌드를 하고 이것이 완료되면 다른 개발자들은 신뢰할 수 있는 기반에서 작업을 할 수 있다. 그래서 하루에 여러 번 통합이 일어나기 때문에 여러 번
빌드가 진행될 수 있고, 빌드에 소요되는 시간이 오래 걸리면 그에 따라 피드백과 작업이 느려질 수 밖에 없는 것이다.

더 자세한 내용은 [martinFowler의 continous Integration 글](https://www.martinfowler.com/articles/continuousIntegration.html)을 읽어보면 좋을 것 같다.

# 




* 참고
* https://www.martinfowler.com/articles/continuousIntegration.html
* http://nangpuni.net/?p=957
* 
