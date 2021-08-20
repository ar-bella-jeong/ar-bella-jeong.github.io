---
title: "[Kubernetes] Controller"
date: 2021-08-20 16:19:28 -0400
categories: kubernetes
tags:
  - kubernetes
comments: true
---
Robotics와 자동화에서 control loop는 system 상태를 조절하는 종료되지 않는 loop이다. ex) 실내 온도 조절기

k8s에서 controller는 cluster의 상태를 관찰 한 다음, 필요한 경우에 생성 또는 변경을 요청하는 control loop이다.

## Controller pattern
controller는 적어도 하나 이상의 k8s resource 유형을 추적한다. 

### API server를 통한 제어
#### ex) job controller
k8s 내장 controller의 예시이다. 내장 controller는 cluster API server와 상호 작용하며 상태를 관리한다.

(일단 schedule되면, pod object는 kubelet의 의도한 상태 중 일부가 된다.)

job controller가 새로운 작업을 확인하면, cluster 어딘가에서 node 집합의 kubelet이 작업을 수행하기에 적합한 수의 pod를 실행하게 한다. -> job controller는 pod나 container를 스스로 실행하지 않는다. 대신, api server에 pod를 생성하거나 삭제하도록 지시한다. control plain의 다른 component는 신규 작업 정보에 대응하여 완료시킨다.

또한 controller는 job을 위한 작업이 종료 되면, job object가 `Finished`로 표시되도록 update한다.

### 직접 제어
job과는 대조적으로, 일부 controller는 cluster 외부의 것을 변경해야 할 필요가 있다. ex) [autoscaler](https://github.com/kubernetes/autoscaler/)

controller는 API server에서 의도한 상태를 찾은 다음, 외부 system과 직접 통신해서 현재 상태를 보다 가깝게 만든다. 그 후 API server에 다시 보고한다.

K8s extension을 통해 IP 주소 관리 도구, storage service, cloud provider의 API 같은 서비스 등과 간접저그로 연동하여 이를 구현한다.

## Controller를 실행하는 방법
k8s에는 [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) 내부에서 실행되는 중요한 핵심 동작을 제공하는 내장된 controller 집합이 있다.  deployment controller나 job controller가 그 예시이다.

원하는 경우 controller를 직접 작성해서  control plane 외부에서 실행해서 확장할 수도 있다. 

> 출처
> - https://kubernetes.io/ko/docs/concepts/architecture/controller/
