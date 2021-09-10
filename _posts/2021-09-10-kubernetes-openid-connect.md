---
title: "[Kubernetes][Security] OpenID Connect"
date: 2021-09-10 10:28:28 -0400
categories: kubernetes
tags:
  - kubernetes
  - security
comments: true
---
## What is OpenID Connect?
OpenID Connect 1.0은 OAuth 2.0 프로토콜 위에 있는 간단한 ID 계층이다.  JSON format을 이용한 RESTful API 형식을 사용하여 인증한다.

### OAuth 2.0
OAuth는 권한허가를 처리하기 위해 만들어진 표준 protocol 이다. Google, Facebook, Twitter 등에서 자신의 service를 외부 system에서 사용할 수 있도록 하기 위한 기술이다. 사용자 인증보다는 제한된 사람에게 제한된 권한을 어떻게 잘 부여할 것인가에 대해서 중점적으로 다룬다.
 이에 반해 OpenID는 인증 system으로써 사용자 정보를 관리하고 인증하는 것에 초점이 맞춰져 있다.

### Identify Provider (OpenID Provider)
줄여서 IdP는 실제 사용자 정보를 제공해 주는 신원 제공자 이다. OpenID Connect는 인증 표준 규격을 정의하는 Protocol에 가깝고 실제 인증 및 사용자 정보 제공은 IdP를 통해 수행된다. 이 역할은 OAuth가 수행한다.
![](https://coffeewhale.com/assets/images/k8s-auth/03-02.png)
OAuth 진영에서 OAuth 기술은 Authentication 기술이 아니라고 명시한다. OAuth에서 제공해주는 access token은 특정 액션을 위해 일시적으로 권한을 허가해 준 토큰일 뿐이지 사용자에 대한 정보를 전혀 담고 있지 않다고 얘기한다. (access token을 가지고 있는 누구나 해당 권한을 사용할 수 있다.) access token을 발급하기 위해 사용자 인증을 거치긴 하였지만 access token 자체가 사용자 신원 정보를 대표해서는 안된다고 설명한다. 그렇다면  OpenID Connect에선 OAuth를 어떻게 사용하는 걸까?

### OpenID scope & ID token
OpenID Connect가 OAuth에게 사용자 신원 정보 제공 권한을 요청하는 방식이다.
![](https://coffeewhale.com/assets/images/k8s-auth/03-03.png)
이때 “사용자 정보 제공” 권한을 `openid` scope, 사용자 신원 정보가 담긴 토큰을 `id token`이라고 부른다. 위 그림에서 OpenID Connect는 실제 Component는 아닌 Protocol에 가깝고 모든 구현은 OAuth에 있다.
![](https://coffeewhale.com/assets/images/k8s-auth/03-04.png)
`id token`의 형태는 `jwt` 형식을 따른다. 하여 token이 변조 되었는지는 k8s에서 쉽게 확인할 수 있다.

## Kubernetes와의 인증 메커니즘
![](https://coffeewhale.com/assets/images/k8s-auth/03-05.png)


> 출처
> - https://openid.net/connect/
> - https://coffeewhale.com/kubernetes/authentication/oidc/2020/05/04/auth03/
