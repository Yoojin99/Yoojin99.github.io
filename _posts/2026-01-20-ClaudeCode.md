---
title: "Claude"
excerpt: "본문의 주요 내용을 여기에 입력하세요"

categories:
  - ios
tags:
  - []

permalink: /swift/concurrency/

toc: true
toc_sticky: true

date: 2025-12-23
last_modified_at: 2025-12-23
---

https://code.claude.com/docs/en/how-claude-code-works#work-effectively-with-claude-code

# Agentic loop

Cluade 에 작업을 줄 때 세 개의 phase 를 거침

* Context 수집
* take action
* verify results

Claude 는 tool 을 사용해서 파일을 찾고 코드를 이해하고, 변화를 줄 수 있음

https://mintcdn.com/claude-code/ELkJZG54dIaeldDC/images/agentic-loop.svg?w=1100&fit=max&auto=format&n=ELkJZG54dIaeldDC&q=85&s=73b2a7040c4c93821c4d5bbee9f4a2d4

위 loop 는 질문에 맞게 적용됨
  * e.g. codebase 질문을 하면 오직 context gathering phase 만 이루어짐
  * e.g. bug fix cycle 은 모든 phase 를 반복적으로 수행함