---
title: "[MSA] 3. 서비스 모델링하기"
date: 2022-06-12 12:31:28 -0400
categories: msa
tags:
  - msa
comments: true
---
장점을 극대화하고 잠재적인 단점을 회피할 수 있는 마이크로서비스의 경계에 대해 생각해본다.
## 3.1 무엇이 좋은 서비스를 만드는가?
**느슨한 결합**(loose coupling)과 **강한 응집력**(high cohesion)이다. 이 두 개념이 잘못되면 나머지도 의미가 없다.
## 3.2 경계가 있는 context
Eric Evans의 '도메인 주도 설계'는 실제 세상의 domain(지식이나 영향력, 활동의 영역)을 modeling하는 system 구축 방법에 중점을 두고 있다. 여기서 context는 외부와 통실할 필요가 없는 것뿐만 아니라 다른 context 외부와 공유되는 model이 함께 존재한다.
## 3.3 Business 능력
 Shared module이 필요 시에 **data**에 대한 생각이 빈약한 CRUD 기반의 서비스를 초래하는 경우가 많다. 따라서 '이 context는 무엇을 하는가?'와 '그 일을 하기 위해 어떤 data가 필요한가'를 먼저 물어보라. context의 능력(capability) 관점으로 봐야한다.

## 3.4 거북이 밑에 거북이
 다른 팀에 의해 관리된다면 각 module은 최상위 계층의 마이크로서비스가 될 자격이 있다. 반면 한 팀에 의해 관리된다면 내포 모델이 더 합리적일 것이다.
 
  큰 덩어리로 묶을(chunk up)시 각 context 내 각각의 서비스에 대해 stub(다른 프로그래밍 기능을 대리하는 코드, 임시로 대치)을 만들 필요가 없어 testing이 단순해 진다. 예를 들어 end-to-end test를 수행할 때 다른 service들이 stub을 준비하곤 한다.
  
## 3.6 기술적 경계
기술 접합부에 따라 서비스 경계를 모델링하는 결정이 항상 잘못된 것은 아니다.

## 3.7 마치며
- 경계가 있는 context는 접합부를 찾는 데 중요한 도구

> 출처
>-   Build Microservices - 샘 뉴먼 저
