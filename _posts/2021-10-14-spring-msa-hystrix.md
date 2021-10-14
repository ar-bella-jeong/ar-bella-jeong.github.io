---
title: "[Spring][MSA] Spring Cloud Hystrix"
date: 2021-10-14 13:56:28 -0400
categories: spring
tags:
  - spring
  - msa
comments: true
---
## 1. Histrix의 이해
모든 전기를 사용하는 곳에는 누전차단기가 있다. 

 누전차단기는 전기 사용 중 누전, 과전류, 합선으로  **전기사고가 발생하기 전에 전기를 미리 차단**하는 역할을 한다.

누전차단기와 같이  **전류를 차단하는 장치를 통틀어 우리는 Circuit breaker(전류차단기)**라고 부른다.

어떤 마이크로서비스가 메모리가 새거나(누전) 트래픽이 갑자기 몰려(과부하) 매우 느려지거나 정지되었다고 생각해 보자. 

 그 마이크로서비스를 호출하는 마이크로서비스 역시 계속 느려지고 장애가 전체에 번져 서비스 중단으로 이어질 수 있다.

Hystrix는  **마이크로서비스의 전류차단기(Circuit Breaker) 역할을 하는 오픈소스**이다.

### HOW?
Consumer마이크로서비스와 Producer마이크로서비스 사이에 Circuit Breaker를 통해 통신하도록 하면 된다.

실제 개발할때는 Consumer마이크로서비스에 Hystrix client를 추가한다.

Hystrix는 Producer가 일정횟수 이상 비정상적인 응답을 주면 Circuit Breaker를 Close상태에서 Open상태로 Trip(이동)한다. 빠른 실패(Fast failing)를 위한 Fallback method를 호출하는데, Fallback method는 에러메시지나 캐싱된 결과를 리턴한다.
![](https://blog.kakaocdn.net/dn/bhbhC4/btqW8Fysesv/6qAGkmxsesvNw1PQGih1Vk/img.png)
Circuit breaker는 OPEN된 상태에서 일정시간이 지나면 Producer마이크로서비스를 1번 호출하여 정상적 결과가 오면, Circuit Breaker의 상태를 'CLOSED'로 바꾸고, Producer마이크로서비스를 호출하기 시작한다.
비정상적 결과가 오면, OPEN된 상태에서 일정시간이 경과했는지 계산하는 타이머를 0으로 초기화한다.
![](https://blog.kakaocdn.net/dn/PNFeq/btqXj87BTTq/JBGACpu7gM1obnr6Kw4bt1/img.jpg)
> 참고) Hystrix는 고슴도치라는 뜻이라 logo도 고슴도치 이다. 고슴도치가 가시를 이용해 몸을 보호하듯이, Hystrix를 이용해 마이크로서비스를 장애로부터 보호한다는 의미인 듯 하다.

## 2. Hystrix 적용
```java
@HystrixCommand(fallbackMethod="getCoffeeFallback") 
public List<String> getCoffees(String param) { 
 // producer에 호출
} 

public List<String> getCoffeeFallback(String param, Throwable t) { 
	System.err.println("###### ERROR =>"+t.toString()); 
	return Collections.emptyList(); 
}
```
@HystrixCommand 주석을 붙이면, 해당 메소드는 Hystrix client를 통해 통신하게 된다. Circuit breaker가 오픈되었을때 대신 수행할 FallbackMethod까지 지정한다.

자세한 hystrix tunning은 property를 통해 가능하다. e.g.) thread pool size, timeout..

## **3. Hystrix dashboard & Turbine**
dashboard를 제공하는 라이브러리는 Hystrix dashboard이다.

Hystrix dashboard앱을 생성해 실행하고 모니터링할 대상 앱의 주소를 입력하여 실시간으로 Circuit breaker의 상태를 볼 수 있다.
![](https://blog.kakaocdn.net/dn/bqhGrl/btqWX5kssru/xILM4s7lMp9L1ceiUsidQk/img.png)
![](https://blog.kakaocdn.net/dn/LgI05/btqWVR1yZ2v/OqM1XiPG72BJobrCuKKF0K/img.png)

한 페이지에서 여러 어플리케이션의 Circuit breaker 상태를 모니터링하려면 Turbine이라는 라이브러리를 사용하면 된다.
### Turbine
ctuator 'hystrix.stream'을 이용하여 한번에 하나의 어플리케이션은 모니터링할 수 있지만,

복잡한 실제 운영환경에서는 여러 어플리케이션을 한꺼번에 모니터링할 수 있는 방법이 필요하다.

**Turbine은 여러 어플리케이션의 hystrix.stream정보를 수집하는 서버**이다. 이 또한 was를 따로 생성해야 한다.
![](https://blog.kakaocdn.net/dn/QyRKQ/btqWWsNWsTu/LFwlJ0uWUfsSYsawiAStAK/img.png)
hystrix dashboard를 웹브라우저에 열고, turbine서버의 주소를 입력한다.
![](https://blog.kakaocdn.net/dn/dSAymH/btqXgpu23nq/8LpQUbY9hdbK5tDbRMJhQK/img.png)
아래와 같이 Circuit breaker상태를 한꺼번에 볼 수 있다.
![](https://blog.kakaocdn.net/dn/c9cwML/btqXgoiAvhB/4h3Uasf5N6ej1fBiccIqFk/img.png)
> 출처
> - https://happycloud-lee.tistory.com/215
