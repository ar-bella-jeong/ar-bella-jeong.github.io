---
title: "[Kubernetes][AWS] Cluster AutoScaler"
date: 2021-09-23 11:00:28 -0400
categories: kubernetes
tags:
  - kubernetes
  - aws
comments: true
---
The Kubernetes [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)는 다음 조건 중 하나가 true일 때, k8s cluster의 크기를 자동으로 조절하는 tool입니다.
- resource부족으로 실행에 실패한 pod가 있을 시
- 장기간 사용률이 낮은 node가 있으며 해당 pod를 다른 기존 node에 배치할 수 있다.

Cluster Autoscaler는 [Deployment](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler/cloudprovider/aws/examples)의 형태로 cluster에 설치 된다.

## Basics
### What types of pods can prevent CA from removing a node?
- PodDisruptionBudget가 있는 pod

## Release
Test는 GCP에서만 이루어지긴 했으나, Kubernetes 1.12 부터 Autoscaler의 version 체계가 k8s minor release와 정확히 일치하도록 변경 되었다.

## Deployment considerations
Cluster Autoscaler가 계속 실행되게 하려면 특별한 주의가 필요하다. `priorityClassName: system-cluster-critical` 를 pod spec에 설정하면 evict를 방지할 수 있다.
v1.0.3 부턴 `cluster-autoscaler.kubernetes.io/safe-to-evict` annotation지원도 추가되었다.

### Scaling considerations
#### Multiple Availability Zone
AWS에선 node group을 여러개 구성하고 각 group의 범위를 단일 Availability 로 지정한 후 `--balance-similar-node-groups` 기능을 활성화 하기를 권장한다. 혹 node group을 하나만 생성할 시엔 둘 이상의 Availability Zone에 걸쳐 있도록 해아한다.
#### Use EBS volumes as persistent storage
EBS Volume을 이용 시 stateful application을 구축할 수 있다. 그러나 single AZ에서만 구축하도록 제한된다. 더 나은 solution으로 각 AZ에 대해 별도 Amazon EBS volume을 사용하여 둘 이상의 AZ에 걸쳐 sharding되는 stateful application을 구축하는 것을 고려할 수 있다. 이로써 application의 가용성을 높일 수 있다.
> 참고
> - https://stackoverflow.com/questions/58675990/sharded-load-balancing-for-stateful-services-in-kubernetes

### CA and HPA
기본적으로 pod의 CPU usage는 kubelet에서10s마다 scrap하고 metric server에서 1m마다 kubelet에서 가져온다. HPA는 metric server에서 30s마다 확인한다. HPA는 replicas 수를 변경하고 3분 동안은 backoff를 하지만 일반적으로 1분에 가깝다.

> 출처
>- https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html
>- https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler
>- https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-the-parameters-to-ca
