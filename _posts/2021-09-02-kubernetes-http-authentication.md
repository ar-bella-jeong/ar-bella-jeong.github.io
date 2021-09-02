---
title: "[Kubernetes][Security] HTTP Authentication"
date: 2021-09-02 17:07:28 -0400
categories: kubernetes
tags:
  - kubernetes
  - security
comments: true
---
HTTP Authentication이란 HTTP 프로토콜에서 제공하는 인증 방법 중 하나이다. HTTP Header를 통해 인증 정보를 서버에 전달한다. 

## HTTP Authentication
### Authorization Header
HTTP Authentication이 적용된 서버에 인증 없이 접속을 하면 `401` error code를 응답 받는다.
사용자 인증 정보를 넣기 위해서는 HTTP Header에 `Authorization` 필드를 기입해야 한다. (과거에 Authentication과 Authorization을 혼용해서 사용한듯..)

```
Authorization: <type> <credentials>
```
-   `type`: 인증 방식을 선언. 쿠버네티스에서는 Basic과 Bearer 타입을 사용.
-   `credentials`: 사용자 인증정보를 기입한다. ID 및 비밀번호, 혹은 토큰값을 넣는다.

### Basic type
사용자의 ID 및 비밀번호를 `:` delimiter를 이용하여 합친 다음 base64 인코딩하여 서버로 전송한다. 암호화 기술은 아니므로 보안을 위해 HTTPS로 접속하는 것을 권장한다.
```
Authorization: Basic BASE64($USER:$PASSWORD)
```

### Bearer type
server에서 지정한 어떤 string도 입력 가능하다.
```
Authorization: Bearer hello-world-token
```
굉장히 허술한 느낌을 받는다. 이를 보완하고자 쿠버네티스에서 Bearer 토큰을 전송할 때 주로 `jwt` (JSON Web Token) 토큰을 사용한다.

### JSON Web Token (jwt)
`jwt`는 `X.509 Certificate`와 마찬가지로 private key를 이용하여 토큰을 서명하고 public key를 이용하여 서명된 메세지를 검증한다. `X.509 Certificate`의 lightweight JSON 버전이라고 생각하면 편리하다.
![](https://coffeewhale.com/assets/images/k8s-auth/02-01.png)
#### JWT 형식
`jwt`는 쿠버네티스에서 뿐만 아니라 다양한 웹 사이트에서 인증, 권한 허가, 세션관리 등의 목적으로 사용한다.
`jwt`의 형식은 크게 3가지 파트로 나뉜다.
-   Header: 토큰 형식와 암호화 알고리즘을 선언.
-   Payload: 전송하려는 데이터를 JSON 형식으로 기입.
-   Signature: Header와 Payload의 변조 가능성을 검증.

각 파트는 base64 URL 인코딩이 되어서 `.`으로 합쳐지게 된다. `jwt`의 최종 결과물은 다음과 같이 생성된다.

```java
base64UrlEncoded(header).base64UrlEncoded(payload).HASHED_SIGNATURE

header = {
  "alg": "HS256",
  "typ": "JWT"
}

payload = {
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}


JWT(header, payload)
// 예시) eyJhbGciOiJIUzII6IkpXVCJ9.eyJzdWIiOiIxM3ODkwIiwibmkDIyfQ.SflKxwRJSMeKK4fwpssw5c
```
#### Signature 파트
여기서 `HASHED_SIGNATURE` 부분이 `jwt` 데이터의 무결성을 보장한다. 

아래는 `HASHED_SIGNATURE` 생성 pseudo 코드이다.
```java
# HASHED_SIGNATURE
RSASHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  $PRIVATE_KEY
)
```
주의해야 할 점은 `jwt` 기술이 인증 기술일 뿐 보안 기술은 아니라는 점 이다. `jwt` 토큰을 탈취 당하는 순간, 해커는 해당 토큰을 이용하여 서버에 정상적으로 접근할 수 있다.

### Service Account Token
쿠버네티스에서는 Bearer 토큰을 사용할때 단순한 문자열이 아닌 위조 방지 장치가 내장된 `jwt` 토큰을 사용한다고 하였는데, 바로 쿠버네티스의 `ServiceAccount` 리소스의 사용자 토큰을 생성할 때 `jwt`를 사용한다. 아래의 예시는 `ServiceAccount`를 통해 생성된 `jwt` 예시이다.
![](https://coffeewhale.com/assets/images/k8s-auth/02-02.png)

아래에 `jwt`의 서명이 Invalid하다는 결과가 나오는 것은 아직 Public key를 통하여 사용자의 `jwt` 토큰을 검증하지 않았기 때문이다.
![](https://coffeewhale.com/assets/images/k8s-auth/02-03.png)

## Using HTTP Authentication
### API 서버 Basic Auth
base64나 hash값이 아닌 plain text로 생성한다.
```
password,user,uid,"group1,group2,group3"
```

-   `password`: basic auth 인증에 사용할 비밀번호
-   `user`: basic auth 인증에 사용할 사용자명
-   `uid`: 쿠버네티스에서 인식하는 식별자
-   `group#`: 쿠버네티스 내부 그룹 지정

`basic-auth`  파일 생성 (username: user1 / password: pass1)
```bash
sudo bash -c 'echo pass1,user1,user1,system:masters > /etc/kubernetes/pki/basic-auth'
```
해당 파일을 api 서버 설정에  `--basic-auth-file`  옵션으로 추가합니다.

```bash
# API 서버 설정 파일
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
# -----[kube-apiserver.yaml]------
    - kube-apiserver
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    # .....
    - --basic-auth-file=/etc/kubernetes/pki/basic-auth
# --------------------------------
```
#### curl을 이용한 Basic auth 테스트
```bash
kubectl cluster-info
# Kubernetes master is running at https://XXXX:XXX

# API 서버 주소 및 포트 설정
API_SERVER_ADDR=XXXX  # 예시) localhost
API_SERVER_PORT=XXX   # 예시) 6443

# curl - basic auth 사용자 인증
curl -k --user user1,pass1 https://$API_SERVER_ADDR:$API_SERVER_PORT/api
curl -k -H "Authorization: Basic $(echo -n user1:pass1 | base64)" https://$API_SERVER_ADDR:$API_SERVER_PORT/api
```
#### kubectl을 이용한 Basic auth 테스트
```bash
# kubectl 신규 사용자 설정 - basic-auth
kubectl config --kubeconfig=$HOME/kubeconfig set-credentials basic-user --username=user1 --password=pass1
kubectl config --kubeconfig=$HOME/kubeconfig set-context kubernetes-admin@kubernetes --user=basic-user
kubectl config --kubeconfig $HOME/kubeconfig view

# kubectl - basic auth 사용자 인증
kubectl --kubeconfig $HOME/kubeconfig get pod -n kube-system

# 혹은 단순히 --username, --password 파라미터를 이용할 수도 있습니다.
kubectl get pod -n kube-system --username user1 --password pass1
```
### API 서버 Bearer token 인증
#### 단순 Bearer token 인증
basic auth file과 형식이 유사하다.
```
token,user,uid,"group1,group2,group3"
```

#### Service Account token 인증
```bash
kubectl get serviceaccount default -oyaml
# apiVersion: v1
# kind: ServiceAccount
# metadata:
#   name: default
#   namespace: default
# secrets:
# - name: default-token-xxxx

JWT_TOKEN=$(kubectl get secret default-token-xxx -ojson | jq -r .data.token | base64 -d)
echo $JWT_TOKEN
# eyJhbGXXX.XXXXX.XXX
```
혹은 Pod 실행시 내부에 mount되는 `Secret` token을 확인할 수도 있다.
```bash
kubectl run cat-token --image k8s.gcr.io/busybox --restart OnFailure -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
JWT_TOKEN=$(kubectl logs cat-token)
echo $JWT_TOKEN
# eyJhbGXXX.XXXXX.XXX
```
##### kubectl을 이용한 Bearer token 테스트
```
kubectl api-versions --token $JWT_TOKEN
```

> 출처
> - https://coffeewhale.com/kubernetes/authentication/http-auth/2020/05/03/auth02/#json-web-token-jwt



