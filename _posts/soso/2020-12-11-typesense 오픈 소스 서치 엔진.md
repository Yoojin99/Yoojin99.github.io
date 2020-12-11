---
title:  "typesense - 오픈 소스 검색 엔진"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - soso
  
tags:
  - typesense
last_modified_at: 2020-12-11
---

[홈페이지](https://typesense.org)

- 매우 빠르고, 검색어에 연관된 결과를 찾아줌 
ㅤ→ Typo Tolerance(오타 허용) : 어느정도 알아서 오타를 인식하고 처리
- 검색 순위를 편하게 조정 가능
- 검색시 특정 필드별로 정렬 지원 
- Facet & Filter 탐색 지원 : 검색 결과를 특정 필드단위로 묶어서 보여주고 필터링
- 특정 결과에 접근하기 위한 API Key 범위 제한 가능
- Raft 기반 클러스터링 
- Linux/Mac 바이너리 및 도커 이미지 제공
- C로 작성된 오픈소스

FAQ에서
- ElasticSearch 와는 뭐가 다른가요 ? 
ㅤ→ ES는 설치 및 관리가 복잡하지만, TypeSense는 "Time-to-Market"을 위해 만들어진거라 빠르게 설치가 가능하고, 물론 스케일링도 할 수 있음
- Algolia 와는 뭐가 다른가요 ?
ㅤ→ 알고리아는 꽤 좋은 검색엔진 SaaS지만 비쌈. TypeSense는 자체 호스팅도 가능하고, SaaS 버전도 저렴(저장된 레코드 나 검색당 과금이 아니고, 사용 시간 및 밴드위스 당으로 과금)
ㅤ→ 기능상으로 TypeSense는 ElasticSearch 보다는 Algolia 랑 비슷
- 속도가 빠른데 메모리 풋프린트는 ?
ㅤ→ 기본적으로 TypeSense서버는 30메가 정도의 메모리를 차지하고, 데이터 인덱싱을 시작하면 늘어나는데 굉장히 간결한 데이터구조를 유지함
ㅤ→ 해커뉴스 글 제목 1백만개가 JSON으로 88MB인데, 이거를 Typesense가 인덱스해서 메모리에 올리면 165MB 정도를 사용

참고로 typesense를 이용해서 만든 음식 레시피 검색 엔진이 있다. [링크](https://recipe-search.typesense.org)

- 오픈소스 검색엔진인 Typesense 를 이용해서 만든 데모 
ㅤ→ 한글자 입력할 때마다 실시간으로 결과를 보여줌
- 데이터분석용으로 공개된 2백만개 레시피 데이터셋을 S3에 올리고, 3 노드 Typesense 백엔드로 실행 
- Algoria의 InstantSearch.js 를 TypeSense에 연결해서 사용해 깔끔한 검색화면을 보여줌

소스코드 : https://github.com/typesense/showcase-recipe-search
2백만개 레시피 데이터셋 : https://github.com/Glorf/recipenlg
