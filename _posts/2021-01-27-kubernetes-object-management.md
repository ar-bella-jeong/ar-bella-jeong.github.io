---
title: "Kubernetes Object Management"
date: 2021-01-27 12:19:28 -0400
categories: kubernetes
tags:
  - kubernetes
---

Kubernetes를 사용하면서 자주 사용하는게 kubectl 명령어이다. 보통 create나 apply를 사용하여 container를 사용하는데, 차이점이 있다.

- Kubernetes Object Management

Management technique | Operates on | Recommended environment
 ---|:---:|---: 
 Imperative commands|Live objects|Development
 Imperative object configuration|Individual files|Production
 Declarative object configuration|Directories of files|Production

## Imperative Commands
#### 장점
- cluster에 특정 object를 한번에 실행하거나 시작할 수 있는 가장 쉬운 방법이다. (실제 이미지를 바로 실행 함)
#### 단점
- 기존 설정에 대한 history 제공을 하지 않음.(그래서 권장 환경이 Development)
- 변경사항이나 audit 내역, template등을 제공하지 않음.

> audit?
> Kubernetes API 서버에서 cluster backend에 대한 감사(audit) log이다. Admin의 Activity log나 data access log들을 수집한다.

ex)

```bash
kubectl run nginx --image nginx
```

## Imperative object configuration
- 최소한 1개 이상의 YAML이나 JSON format의 file을 이용해서 Object를 생성.

ex)

```bash
kubectl create -f nginx.yaml
```

### Imperative Object configuration vs Imperative Commands
#### 장점
설정파일 내용을 git 같은 곳에 저장이 가능하기 때문에 설정에 대한 변경 history가 확인 가능.
#### 단점
YAML 파일 작성이 필요하며 template 사용을 위해 Object schema에 대해서 이해가 필요.

### Imperative Object configuration vs Declarative object configuration
#### 장점
더 간단함.
#### 단점
- file로 적용할때에만 동작한다. directory는 불가능.
- 실행중인 object를 update하기 위해서는 설정파일에 반영을 해야함.

## Declarative object configuration
모든 설정 파일들은 directory 안에 들어 있고 object를 생성하거나 patch 한다.

ex)

```bash
kubectl apply -f configs/
```

###  Declarative object configuration vs Imperative Object configuration
#### 장점
실행중인 object를 직접 적용한 변경사항을 설정파일에 merge하지 않더라도 유지된다.

#### 단점
디버깅 하기 어렵고 예상한 결과가 아닐 경우 이해하기 어려움.

ex) sample_deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: nginx-deployment
spec:
	selector:
		matchLabels:
			app: nginx
	minReadySeconds: 5
	template:
		metadata:
			 labels:
				 app: nginx
		spec:
			containers:
			- name: nginx
			  image: nginx:1.7.9
			  ports:
			  - containerPort: 80
```

```bash
# Create
kubectl apply -f sample_deployment.yaml
# Live Configuration
kubectl get -f sample_deployment.yaml -o yaml
```

```yaml
kind: Deployment
metadata:
	annotaions:
		# This is the json representation
		kubectl.kubernetes.io/last-applied-configuration: |
			{"apiVersion":"apps/v1"
		...
```

**kubectl.kubernetes.io/last-applied-configuration** 어노테이션에 보면 **sample_deployment.yaml**에 적용한 내용들이 나와있다.

```bash
kubectl scale deployment/nginx-deployment --replicas=2
# Live configuration
kubectl get -f sample_deployment.yaml -o yaml
```

상기와 같은 경우는 apply를 하지 않았기 때문에 **kubectl.kubernetes.io/last-applied-configuration** 어노테이션에 포함되지 않는다. create나 replace 도 마찬가지이다.

> 출처: https://blusky10.tistory.com/373
