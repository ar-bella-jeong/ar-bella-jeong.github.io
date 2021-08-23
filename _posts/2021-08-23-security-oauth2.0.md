---
title: "[Security] OAuth 2.0"
date: 2021-08-23 12:38:28 -0400
categories: security
tags:
  - security
comments: true
---
OAuth란, third-party application이 자원의 소유자인 서비스 이용자를 대신하여 서비스를 요청할 수 있도록 자원 접근 권한을 위임하는 방법으로, 인증을 위한 open standard protocol이다.

예로들어 특정 site에서 google 연동 기능을 위해 인증이 진행되어야 한다면, user에게 id/password를 얻어 인증을 하면 될 것이다. 헌데 뭘 믿고 이 사이트에게 user의 id/password를 제공해준단 말인가?

따라서 AccessToken을 발급받아 그를 기반으로 원하는 기능을 구현하는 방식이 있다. AccessToken이란 login을 하지 않고 인증을 할 수 있도록 해주는 'token'정도로 생각하면 된다. user가 해당 site에 google service에 대한 접근을 허용해 줄 시 AccessToken에는 해당 정보가 남아있어 이를 이용할 수 있게 된다. 이 token을 발급받기 위한 일련의 과정을 interface로 정희해둔 것이 바로 OAuth이다.

## OAuth 2.0 개념
**O**pen **Auth**orization

 OAuth 1.0에서 web app이 아닌 app에서는 사용하기 곤란하다는단점을 보완하여 OAuth 2.0가 나타났고, 보안 강화를 위해 Access Token의 Life-time을 지정하여 사용한다.

## OAuth 2.0 동작 방식
![](https://blog.kakaocdn.net/dn/CKvXR/btqygUfJK3N/Qv0JA43seLVlfJfrkLmh8K/img.png)

## OAuth 특징과 기능
- HTTPS를 사용하고 암호화가 필요X
- Signature 단순화 정렬과 URL encoding이 필요X
- OAuth 1.0에는 access token 만료X

> 출처
> - https://blog.naver.com/pjok1122/221583426424
> - https://onepinetwopine.tistory.com/389
