---
title: "[AWS] Amazon API Gateway"
date: 2022-01-21 20:12:28 -0400
categories: aws
tags:
  - aws
  - msa
  - network
comments: true
---
Amazon API Gateway는 AWS가 제공하는 Managed Service이다. API의 흐름을 제어하고 authentication, logging 등의 service를 제공한다. API Gateway는 Microservice 및 Serverless architecture의 기둥 중 하나가 되었다.
## API Gateway의 architecture
![
                API 게이트웨이 아키텍처 다이어그램
            ](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/images/Product-Page-Diagram_Amazon-API-Gateway-How-Works.png)
최대 수십만개 동시 API 호출 허용 및 traffic 관리, 권한 부여 및 access control, monitoring, API version 관리가 포함된다. 다양한 기능에 access할 수 있게 해주는 "정문" 역할을 한다.
## API Gateway의 기능
- [인증](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/apigateway-control-access-to-api.html) 메커니즘(예: AWS Identity and Access Management 정책
 -   API를 게시하기 위한  [개발자 포털](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/apigateway-developer-portal.html).
 -   변경 사항을 안전하게 롤아웃하기 위한  [Canary 릴리스 배포](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/canary-release.html).
 -   성능 지연 시간 파악 및 학습을 위해  [AWS X-Ray](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/apigateway-xray.html)와 통합.

### HTTP API vs REST API

https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/http-api-vs-rest.html

2015년 API Gateway의 첫번째 버전으로써 일반적인 용도인 REST API가 나왔고 2019년에 HTTP API를 발표했다.
#### Price
HTTP API가 REST API보다 저렴
#### Performance
HTTP API가 최대 60% 빠름.
#### Feature
##### Throttling(조절)
HTTP, REST API 모두 account-level의 throttling을 지원한다. 기본적으로 API Gateway는 steady-state(안정 상태) request rate를 10,000 requests per second(rps)로 제한한다. REST API는 api key별(user)로 usage plan을 지원한다.

AWS WAF(web application firewall)은 REST API와 통합되어 공격으로 부터 추가 보호도 가능하다.

HTTP API를 사용하는 주된 이유는 비용이며, REST API의 기능이 필요한 경우는 REST API를 사용하는 것이 좋다.

## API Gateway 구성과정
### Application Deploy
application deploy 및 ALB 구성까지 마친다.
### Amazon API Gateway API 등록
- API Gateway에서 제공하는 API 유형은 아래와 같이 4가지가 있다.
![](https://blog.kakaocdn.net/dn/wIxDR/btqK6Awr4TY/DlExkLdrEQiV0zk3kezYyk/img.png)
- REST API 선택 후 API 관련 간단한 기본 설정을 입력하고 아래 화면에서 api url을 설정하면 된다.
![](https://blog.kakaocdn.net/dn/bFlTdF/btqK4IBKUBe/YBjElxrKJq1Qnp2GD1Zig0/img.png)
- resource가 생성되면 마지막 url을 선택하고 method를 생성한다.
![](https://blog.kakaocdn.net/dn/kVGzc/btqK6NWDrlE/qIvbVabTZC6eVRlKIUi1j1/img.png)
- 하기와 같이 복수의 method 생성도 가능하다.
![](https://blog.kakaocdn.net/dn/n7TZA/btqK8256VQq/mlCP975vDaKkwAIySkkLl0/img.png)

## API Gateway URL 관리
Amazon API Gateway에서 Deploy 된 API는 다음과 같은 URL 형태를 갖는다.

`https://{restapi-id}.execute-api.{region}.amazonaws.com/{stage_name}`
e.g.) https://asdqwefgrt.execute-api.ap-northeast-2.amazonaws.com/testapi

- restapi-id : api deploy 시 무작위로 생성되며, 지역 내 고유한 도메인을 부여하기 위함이다.
- region : 지역 코드가 들어간다.
- stage_name : api deploy를 위한 stage 명이 들어간다.

 이는 사실 사용자에게 굉장히 불친절한 도메인으로 그대로 사용할 경우 Client에게 혼선을 줄 수 있어 AWS는 사용자 지정 도메인을 사용하여 도메인을 매핑할 수 있다.
![](https://blog.kakaocdn.net/dn/dq5mvp/btqLpT9n4rV/0f0WKQLlZkUCHWwdQbTC4k/img.png)
`https://{api.nrson.com}/{basePath}`

- api.nrson.com : 도메인 이름은 임의로 지정이 가능하지만, AWS Route53 또는 그밖의 DNS 서비스를 제공하는 인터넷 호스팅 업체를 통해 도메인이 등록되어야 한다.
- basePath : stage name을 api의 default path로 사용한다.

## API Gateway Flow
api gateway는 Back End에 존재하는 대상 그룹을 연결해 주는 단일 포인트로써 존재한다.
### Public subnet과의 연동
[API Gateway - ALB - Target Group - EKS, ECS, EC2 등]
![](https://blog.kakaocdn.net/dn/F6uHi/btqK83qqwY9/MJ6ZUDMbSdTBJTv23sf5dk/img.png)
대체로 이 구성은 POC 및 test시에 이용된다. 반드시 ALB에 대한 보안구성을 꼼꼼히 해야한다.
### Private subnet과의 연동
[API Gateway - VPCLink - NLB - Target Group - EKS, ECS, EC2 등]
![](https://blog.kakaocdn.net/dn/dpP2LQ/btqK7z4EZhO/RPE4LK0YHNlRttsKxtNXtk/img.png)
보안측면에서 강점이 있지만 VPCLink를 사용하기 위해 NETWORK LOAD BALANCER를 사용해야한다. ALB는 APPLICATION 로드밸런서로 경로 기반 라우팅이 가능하지만, NLB는 포트 기반 라우팅을 지원하여 라우팅 처리해야 한다. 다만 ALB 대비 대용량 API 처리에 용이하고, 고정 IP를 사용할 수 있다는 장점이 있다.

앞서 살펴본 ALB 형태와 연결하는 방식이외에 NLB Type의 로드밸런서와 연결하기 위해서는 다음과 같이 VPCLink를 사용해야 한다.
![](https://blog.kakaocdn.net/dn/bzsifw/btqO0fnGnQu/o9a6vXkHxtTVExBcZFV8o1/img.png)

> 출처
> - https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/welcome.html
> - https://waspro.tistory.com/657
> - https://www.learnaws.org/2020/09/12/rest-api-vs-http-api/
