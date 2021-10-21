---
title: "[Spring][MSA] Spring Cloud Zuul"
date: 2021-10-14 19:08:28 -0400
categories: spring
tags:
  - spring
  - msa
  - network
comments: true
---

service을 운영하고 개발하는 team이라면 LEGACY라는 괴물은 오래되면 될수록, 변화시키기도 어렵고 개발 비용도 많이 든다. 이 때 API Gateway를 도입하면 legacy를 고립시키고, domain 단위로 feature를 조금씩 떼어낼 수 있다.



## API Gateway?
**API Gateway는 API의 요청자인 Client(웹어플리케이션 또는 모바일앱)와 API의 제공자인 backend service를 연결하는 중계자**이다.

MSA에서 언급되는 component 중 하나이며, 모든 client request에 대한 endpoint를 통합하는 서버이다. 마치 proxy server 처럼 동작한다. 그리고 인증 및 권한, 모니터링, logging, circuit break 등 추가적인 기능이 있다.

**API Gateway가 필요한 이유**는  **안전한 API유통과 Client 요청별로 유연하게 대처하기 위해서**이다. Client요청에 유연하게 대응한다는 의미는 Client 유형(웹브라우저, 모바일앱 등)에 따라 맞는 API를 연결한다거나, Client를 사용하는 사용자의 권한이나 속성에 따라 적절한 결과를 리턴한다는 것이다.

점진적으로 legacy system을 신규 시스템으로 교체하거나(Strangler pattern), traffic 일부만 새로운 서비스로 routing(Canari deployment)도 가능하다.

또한 MSA의 경우  client  입장에서 다수의 endpoint가 생기게 되며, endpoint 변경이 일어났을 때, 관리하기가 힘들다. 그래서 MSA 환경에서는 service에 대한 domain을 하나로 통합할 수 있는 API gateway가 필요한 것이다.

참고로, 오픈소스 API Gateway에는 kong, Netflix Zuul, Spring cloud gateway, ServiceComb EdgeService등이 있다.

## Zuul?
Zuul server는 API Gateway 이다.
![](https://techblog.woowahan.com/wp-content/uploads/img/2017-06-06/zuul-netflix-cloud-architecture.png)
### Netflix에서는 왜 Zuul을 사용하나?
client 요청은 많은 traffic과 예상하지 못한 형태의 요청으로 경고없이 운영에 이슈를 발생시킨다. 이러한 상황에 신속하고 동적으로 해결하기 위해 groovy 언어로 작성 된 다양한 형태의 Filter를 실행한다. 
Filter에 기능 정의 및 issue 상황 발생 시 적절한 filter를 추가함으로 써 이슈사항을 대비할 수 있다.

Netfilx Zuul은 인증/인가, L/B & Routing, Logging, Circuit break를 어떻게 제공할까?
zuul은 java로 개발된 서버이고 customizing할 수 있는 application이다. class로 개발된 filter가 이용되며, overriding하여 필요한 수행을 추가할 수 있다.

보통 들어오는 요청에 대해 'pre' filter에서 인증/인가 처리를 하고, routing filter에서 L/B, Routing, Circuit break를 처리하며, Post filter에서 요청과 응답에 대한 Logging을 처리한다. L/B는 ribbon이라는 라이브러리가 사용되고, Routing은 zuul core라이브러리가 사용되며, Circuit break는 Hystrix라이브러리가 사용된다.
![](https://blog.kakaocdn.net/dn/bBmH6F/btqXj80P0lg/IPeTTfz2ymCzdXl8DWqUV0/img.png)

### Netflix filter의 기능
-   Authentication and Security
    -   클라이언트 요청시, 각 리소스에 대한 인증 요구 사항을 식별하고 이를 만족시키지 않는 요청은 거부
-   Insights and Monitoring
    -   의미있는 데이터 및 통계 제공
-   Dynamic Routing
    -   필요에 따라 요청을 다른 클러스터로 동적으로 라우팅
-   Stress Testing
    -   성능 측정을 위해 점차적으로 클러스터 트래픽을 증가
-   Load Shedding
    -   각 유형의 요청에 대해 용량을 할당하고, 초과하는 요청은 제한
-   Static Response handling
    -   클러스터에서 오는 응답을 대신하여 API GATEWAY에서 응답 처리
![](https://techblog.woowahan.com/wp-content/uploads/img/2017-06-06/Zuul-Core-Architecture.png)
-   Filter File Manager에서는 일정 주기(정해진 시간) 마다 정해진 directory에서 groovy로 정의된 filter 파일을 가져온다.

### MSA Architecture sample
![](https://blog.kakaocdn.net/dn/JZv2M/btqW8FZvoJy/cWeDqhQAVhKcpfKKCXYsfK/img.png)

요청이 들어오면 zuul은 자동으로 eureka에서 마이크로서비스를 찾아 그 주소로 라우팅한다.

config server는 git에서 config파일을 읽고, config 변경시 Message broker(예:rabbitMQ, kafka등)에 변경내용을 반영하고 그럼 각 마이크로서비스는 변경을 통지받고 config server를 통해 최신 config를 갱신하게 된다.

각 마이크로서비스는 ribbon을 이용하여 직접 다른 마이크로서비스를 load balancing하여 연결할 수 있다.

### Zuul Components
-   zuul-core : 위에서 설명한 Zuul의 Request Lifecycle를 담당하고, Fliter를 컴파일하고 실행한는 기능을 담당하고 있는 Zuul의 core library
-   zuul-netflix : 기본적은 Zuul에 NetflixOSS library를 추가한다.
-   zuul-simple-webapp : zuul-core만 사용한 아주 기본적은 web application
-   zuul-netflix-webapp : zuul-core와 zuul-netflix를 함께 사용한 web application

zuul-netflix-webapp을 도입하기에는 학습곡선이 커서 spring cloud netflix를 많이 찾는다.

## Spring cloud netflix
spring boot에 NetflixOSS를 통합적으로 제공한다. annotation과 yml설정 만으로도 아주 쉽게 NetflixOSS를 사용할 수 있다.

### Spring Cloud Zuul
Spring cloud Netflix zuul은 2018년 12월부터 더 이상의 기능 개선 없이 유지만 하는 Maintenance Mode로 변경되었다.
![](https://blog.kakaocdn.net/dn/d0KboD/btqWWs8iyQW/20T2t4KzIWY6nAqlUIh1Vk/img.png)
이미 Spring boot 2.4.X부터는 zuul, hystrix가 더 이상 제공되지 않지만 아직은 Zuul을 많이 사용하고 있다.

> 출처
> - https://happycloud-lee.tistory.com/213
> - https://techblog.woowahan.com/2523/
