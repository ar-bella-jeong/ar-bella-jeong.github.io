---
title: "[Security] JWT"
date: 2022-06-08 10:50:28 -0400
categories: security
tags:
  - security
  - jwt
  - authentication
  - authorization
comments: true
published: true
---
기존에는 Session을 server에 저장해서 stateful 하게 관리하는 방식으로 인증 system이 이루어 졌었으나, memory 문제 및 확장성 그리고 여러 domain에서의 cookie관리가 어려워 Stateless한 구조를 지향하게 되었다. 시스템 규모가 커짐에 따라 서버 기반 인증 방식이 한계를 보였기 때문이다.

하여 현대 Web Service에서 token을 사용하여 사용자들의 인증 작업을 처리하는 것이 가장 좋은 방법이다. 이 때 주로 사용하는 것이 JWT(JSON Web Token)이다.
## JSON Web token?
JWT(JSON Web Token)은 당사자 간의 정보를 JSON 개체로 안전하게 전송하기 위한 간결한 방법을 정의하는 open standard([RFC 7519](https://tools.ietf.org/html/rfc7519)) 이다. 이 정보는 디지털 서명 되어 있으므로 신신뢰할 수 있다. JWT는 secret (**HMAC** algorithm)을 사용하거나, **RSA** 또는 **ECDSA**를 사용하는 public/private key pair를 사용해서 서명할 수 있다.

JWT는 token자체를 정보로 사용하는 Self-Contained 방식으로 정보를 안전하게 전달한다. Logic 예시는 하기와 같다.
![](https://blog.kakaocdn.net/dn/rdboS/btqArUrgcMr/HWY80zNL9reAv6FeE6AYE1/img.png)
## When should you use JSON Web Tokens?
- Authorization
	- SSO에서 보통 JWT를 인증 매체로 사용
- Information Exchange
	- 정보를 안전하게 전송 가능하다. 
	- e.g) public/private key pair를 사용하여 JWT에 서명할 수 있으므로 sender확인이 가능하다. 또한 header와 payload를 사용하여 서명이 계산되므로 content가 변조되지 않았는지 확인할 수 있다.

## JWT structure
JWT는 Header, Payload, Signature의 3 부분으로 이루어지며, Json 형태인 각 부분은 Base64로 인코딩 되어 표현된다.  또한 각각의 부분을 이어 주기 위해  **.**  구분자를 사용하여 구분한다.  

따라서 JWT는 일반적으로 다음과 같습니다.
```
xxxxx.yyyyy.zzzzz
```
![](https://blog.kakaocdn.net/dn/cL07DS/btqArTTqJqU/aKbkKW5KiuPpr7kefx4yAk/img.png)
### Header
토큰의 헤더는 **typ**(token type)과 **alg**(서명 알고리즘: HMAC SHA256 or RSA) 두 가지 정보로 구성된다.  
```json
{ "alg": "HS256", "typ": JWT }
```
### Payload
토큰에서 사용할 정보의 조각들인  **클레임(Claim)**이 담겨 있다.  Claim은 entity(일반적으로 user) 및 추가 data에 대한 설명이다. 클레임은 총 3가지로 나누어지며, Json(Key/Value) 형태로 다수의 정보를 넣을 수 있다.  : _registered_, _public_, and _private_ claims.
#### Registered Claim
필수는 아니지만 token 정보를 표현하기 위해 권장되는 미리 정의된 클레임 집합이다. 또한 JWT를 간결하게 하기 위해 key는 모두 length 3의 string이다. 여기서 subject로는 unique한 값을 사용하는데, 사용자 이메일을 주로 사용한다.  
 -   iss: 토큰 발급자(issuer)
-   sub: 토큰 제목(subject)
-   aud: 토큰 대상자(audience)
-   exp: 토큰 만료 시간(expiration), NumericDate 형식으로 되어 있어야 함 ex) 1480849147370
-   nbf: 토큰 활성 날짜(not before), 이 날이 지나기 전의 토큰은 활성화되지 않음
-   iat: 토큰 발급 시간(issued at), 토큰 발급 이후의 경과 시간을 알 수 있음
-   jti: JWT 토큰 식별자(JWT ID), 중복 방지를 위해 사용하며, 일회용 토큰(Access Token) 등에 사용
 
#### Public claims
JWT를 사용하는 사람들이 마음대로 정의할 수 있다. 충돌 방지를 위해 [IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml) 에서 정의하거나 URI 포맷을 이용하며, 예시는 아래와 같다.  
```json
{
	"https://ar-bella-jeong.github.io": true
}
```
#### Private Claim
서버와 클라이언트 사이에 임의로 지정한 정보를 저장한다. 아래의 예시와 같다.  
```json
{ "token_type": access }  
```
payload의 예는 다음과 같다.
```json
{ 
	"sub": "1234567890", 
	"name": "John Doe", 
	"admin": true 
}
```
### Signature
서명 부분을 생성하려면 인코딩된 헤더, 인코딩된 페이로드를 secret key를 이용해 헤더(Header)에서 정의한 알고리즘으로 해싱을 하고, 이 값을 다시 BASE64로 인코딩하여 생성한다.  
  
예를 들어 HMAC SHA256 알고리즘을 사용하려는 경우 서명은 다음과 같은 방식으로 생성된다.
```
HMACSHA256( 
	base64UrlEncode(header) + "." + 	
	base64UrlEncode(payload), 
	secret)
```
signature는 메시지가 도중에 변경되지 않았는지 확인하는 데 사용되며 private key로 서명된 토큰의 경우 JWT의 보낸 사람이 누구인지 확인할 수도 있다.
### How do JSON Web Tokens work?
생성 된 token은 SAML과 같은 XML 기반 표준과 비교할 때 더 간결하다.
![인코딩된 JWT](https://cdn.auth0.com/content/jwt/encoded-jwt3.png)
JWT를 실제 적용할 때 [jwt.io debugger](https://jwt.io/#debugger-io) 를 사용하여 JWT 를 decoding, 확인 및 생성할 수 있다.
![JWT.io 디버거](https://cdn.auth0.com/blog/legacy-app-auth/legacy-app-auth-5.png)
생성된 토큰은 HTTP 통신을 할 때 Authorization이라는 key의 value로 사용된다. 일반적으로 value에는 Bearer이 앞에 붙여진다.  
```json
{ 
	"Authorization": "Bearer {생성된 토큰 값}", 
}  
```
HTTP 헤더를 통해 JWT 토큰을 보내는 경우 토큰이 너무 커지지 않도록 해야한다. 일부 서버는 8KB 이상의 헤더를 허용하지 않는다. JWT에 너무 많은 정보를 포함하려는 경우 [Auth0 Fine-Grained Authorization](https://fga.dev/)같은 대체 solution이 필요할 수 있다.

다음 다이어그램은 JWT를 얻고 API 또는 리소스에 액세스하는 데 사용하는 방법을 보여준다.

### JWT 단점 및 고려사항
-   Self-contained: 토큰 자체에 정보를 담고 있으므로 양날의 검이 될 수 있다.
-   토큰 길이: 토큰의 페이로드(Payload)에 3종류의 클레임을 저장하기 때문에, 정보가 많아질수록 토큰의 길이가 늘어나 네트워크에 부하를 줄 수 있다.
-   Payload 인코딩: 페이로드(Payload) 자체는 암호화 된 것이 아니라, BASE64로 인코딩 된 것이다. 중간에 Payload를 탈취하여 디코딩하면 데이터를 볼 수 있으므로, JWE로 암호화하거나 Payload에 중요 데이터를 넣지 않아야 한다.
-   Stateless: JWT는 상태를 저장하지 않기 때문에 한번 만들어지면 제어가 불가능하다. 즉, 토큰을 임의로 삭제하는 것이 불가능하므로 토큰 만료 시간을 꼭 넣어주어야 한다.
-   Tore Token: 토큰은 클라이언트 측에서 관리해야 하기 때문에, 토큰을 저장해야 한다.

> 출처
> - https://jwt.io/
> - https://mangkyu.tistory.com/56
