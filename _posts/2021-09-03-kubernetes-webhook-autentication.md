---
title: "[Kubernetes][Security] Webhook 인증"
date: 2021-09-03 11:43:28 -0400
categories: kubernetes
tags:
  - kubernetes
  - security
comments: true
---
## Webhook 인증
Webhook이란 특정 이벤트가 발생하였을 때, 사전에 정의된 웹 URL로 이벤트 정보와 함께 요청을 보내어 후속 작업을 처리할 수 있게 고안된 체계이다. 쿠버네티스 API 서버에도 인증 처리를 위한 Webhook을 구현할 수 있는 메커니즘이 존재하는데, 이때 특정 이벤트는 Authenticate 이벤트가 되고 함께 전송되는 이벤트 객체(정보)는 사용자가 전송한 토큰이 된다.
### Webhook 인증 서버
아래의 이미지는 쿠버네티스 Webhook 인증 순서도이다.
![](https://coffeewhale.com/assets/images/k8s-auth/04-01.png)
### 요청:  `TokenReview`
```json
{
  "kind": "TokenReview",
  "apiVersion": "authentication.k8s.io/v1beta1",
  "metadata": {
    "creationTimestamp": null
  },
  "spec": {
    "token": "$BEARER_TOKEN"
  }
}
```
 사용자가 인증을 위해 전송한 `Bearer token`이 Webhook 인증 서버로 전달되어 해당 값이 유효한지 확인한다.
### 응답:  `TokenReview`  +  `status`
```java
{
  "kind": "TokenReview",
  "apiVersion": "authentication.k8s.io/v1beta1",
  "metadata": {
    "creationTimestamp": null
  },
  "spec": {
    "token": "$BEARER_TOKEN"
  },
  "status": {
    "authenticated": true,
    "user": {
      "username": "user1",
      "uid": "user1",
      "groups": [ "system:masters" ]
    }
  }
}
```
 쿠버네티스에서 사용할 식별자 및 그룹 정보가 들어 있다.

이제 이를 어떻게 적용할 수 있을지 살펴보자.

### API 서버 설정
사용자로부터 API 서버로 인증 요청이 들어왔을때, 해당 이벤트를 Webhook 인증서버로 보내야하는데, 이를 위해서 API 서버에 `--authentication-token-webhook-config-file` 옵션을 추가해 줘야 한다.  해당 옵션은 API 서버로 인증 요청이 들어왔을 때, 어디로 이벤트를 전달할지 알려주는 설정 파일이 저장된 위치를 가리킨다.
```bash
# API 서버 설정 파일
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
# -----[kube-apiserver.yaml]------
    - kube-apiserver
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    # .....
    - --authentication-token-webhook-config-file=/etc/kubernetes/pki/webhook.yaml
# --------------------------------
```
`/etc/kubernetes/pki/webhook.yaml` 설정 파일의 모양은 다음과 같다.
```yaml
# /etc/kubernetes/pki/webhook.yaml
apiVersion: v1
kind: Config
clusters:
- name: flask-auth
  cluster:
    server: <WEBHOOK_SERVER_ADDR>   # https://<ADDR>:<PORT> 형식
    insecure-skip-tls-verify: true  # tls 설정 disable
users:
- name: kube-apiserver
contexts:
- context:
    cluster: flask-auth
    user: kube-apiserver
  name: auth
current-context: auth
```
## Webhook 인증 실험
이제 실제로 Webhook 서버를 구축하여 정상적으로 쿠버네티스 인증이 처리되는 확인해보자.
```python
from flask import Flask, request, jsonify
import pprint

app = Flask(__name__)


@app.route('/', methods=['POST'])
def auth():
	# API 서버로부터 TokenReview 수신
    tokenReview = request.json

    # 인증 결과 (하드코딩)
    status = {}
    status['authenticated'] = True
    status['user'] = {
    	'username': 'alice',
    	'uid': 'alice',
        'groups': ['system:masters']
    }

    # TokenReview에 인증결과 객체 삽입
    tokenReview['status'] = status
    pprint.pprint(tokenReview)

    # API 서버로 json 응답
    return jsonify(tokenReview)


if __name__ == '__main__':
    app.run(host= '0.0.0.0', port=5000, debug=True)
```

# Summary
Webhook을 이용한 인증의 장점은 쿠버네티스에서 제공하지 않는 인증 방식도 Webhook 서버를 통하여 연동할 수 있다는 점이다.  Webhook 서버가 마치 Glue 컴포넌트가 되어 두 시스템간 컨버터 역할을 담당하는 것이다. 이로서 엄청한 유연함을 가진다.

각 클라우드 서비스의 완전 관리형 쿠버네티스 서비스인 `EKS`, `AKS`, `GKE`들도 전부 이 Webhook을 이용하여 각 플랫폼에 맞게 인증을 수행한다. (AWS IAM, Azure AD 등)

> 출처
> - https://coffeewhale.com/kubernetes/authentication/webhook/2020/05/05/auth04/
