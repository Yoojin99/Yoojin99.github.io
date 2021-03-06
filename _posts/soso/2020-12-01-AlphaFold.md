---
title:  "AlphaFold - 단백질 접힘 문제의 솔루션"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - soso
  
tags:
  - 
last_modified_at: 2020-12-01
---

## 단백질 접힘 문제

단백질은 아미노산이 어떻게 결합되어 있는지에 따라서 그 종류가 결정이 된다. 아미노산이 사슬 형태로 연결되어서 입체적으로 접힌 복잡한 구조로 되어있다.
이 구조에 따라 여러가지 다른 기능을 하는 것이다. 

하지만 아미노산의 배열만을 가지고 단백질의 구조를 예측하는 것은 매우 어렵다. 이를 예측하는 인공지능 모델이 있다고 한다.

## AlphaFold

AlphaFold는 단백질 데이터 뱅크의 약 170,000개 단백질 구조 데이터와 그 밖에 구조가 알려지지 않은 단백질 데이터를 학습했다.

약 128개의 TPUv3 코어를 이용해서 몇 주간 학습했다고 한다. 아미노산의 배열이 주어졌을 때 단백질의 구조를 예측하는 작업을 하는 것 같다.

## CASP(Critical Assessment of protein Structure Prediction)

CASP는 1994년에 최초로 개최된 2년마다 열리는 세계 단백질 구조 예측 대회이다.

아직 구조가 알려지지 않은 단백질의 구조를 예측하는 대회이다. AlphaFold의 초기버전이 2018년 대회에 참가했고,
2020년에 열린 CASP14에서 정확도 평균 92.4 로 1위를 했다. 이는 원자 1개 너비 정도의 오차라고 한다.

## 기대효과 및 전망

AlphaFold 는 코로나 바이러스 단백질을 예측했다.

앞으로 이를 이용해서 여러 생물학적 문제 해결과 신약 개발에 도움이 될 것으로 전망된다.
