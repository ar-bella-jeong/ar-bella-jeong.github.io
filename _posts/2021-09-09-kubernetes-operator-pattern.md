---
title: "[Kubernetes] Operator pattern"
date: 2021-09-09 19:28:28 -0400
categories: kubernetes
tags:
  - kubernetes
comments: true
---
[CRD](https://kubernetes.io/ko/docs/concepts/extend-kubernetes/api-extension/custom-resources/)를 사용하여 application 및 component를 관리하는 K8s의 Software extension이다. Operators는 kubernetes의 원칙, 특히 [Control loop](https://ar-bella-jeong.github.io/kubernetes/kubernetes-controller/) 를 따른다.

 CRD와 Operator는 kubernetes의 전반적인 개념과 동작 원리를 가장 잘 보여준다.

## Operators in Kubernetes
 Kubernetes는 자동화를 위해 설계되었다.

Kubernetes' [controllers](https://kubernetes.io/docs/concepts/architecture/controller/) concept을 사용하면  kubernetes 자체의 code를 수정하지 않고도 cluster의 동작을 확장할 수 있다. Operators는 [Custom Resource](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)에 대한 controller 역할을 하는 Kubernetes API의 clients이다.

## Deploying Operators
Operator를 배포하는 가장 일반적인 방법은 CRD 정의 및 관련 controller를 cluster에 추가하는 것이다. controller는 일반적으로 여타 application과 마찬가지로 control plane 바깥에서 배포하여 실행한다. 

## Writing your own Operator
kubernetes 생태계에 원하는 Operator가 없으면 Kubernetes API의 client 역학을 하는 own Operator를 직접 작성이 가능하다.

다음은 Cloud native operator를 작성하는 데 사용할 수 있는 몇가지 lib및 tool이다.
-   [Charmed Operator Framework: juju](https://juju.is/)
-   [kubebuilder](https://book.kubebuilder.io/)
-   [KubeOps](https://buehler.github.io/dotnet-operator-sdk/)  (.NET operator SDK)
-   [KUDO](https://kudo.dev/)  (Kubernetes Universal Declarative Operator)
-   [Metacontroller](https://metacontroller.github.io/metacontroller/intro.html)  along with WebHooks that you implement yourself
-   [Operator Framework](https://operatorframework.io/)
-   [shell-operator](https://github.com/flant/shell-operator)

> 출처
> - https://kubernetes.io/ko/docs/concepts/extend-kubernetes/operator/
> - https://getoutsidedoor.com/2019/07/27/kubernetes-crd%EC%99%80-operator/
