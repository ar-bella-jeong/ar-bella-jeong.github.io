---
title: "C Advisor"
date: 2021-05-12 09:39:28 -0400
categories: container
tags:
  - container
  - google
  - kubernetes
comments: true
---

cAdvisor (Container Advisor) provides container users an understanding of the resource usage and performance characteristics of their running containers. It is a running daemon that collects, aggregates, processes, and exports information about running containers.

For [Kubernetes](https://github.com/kubernetes/kubernetes) users, cAdvisor can be run as a daemonset.

## K8s Resource metric pipeline
리소스 메트릭 파이프라인은  [Horizontal Pod Autoscaler](https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale)  컨트롤러와 같은 클러스터 구성요소나  `kubectl top`  유틸리티에 관련되어 있는 메트릭들로 제한된 집합을 제공한다. 이 메트릭은 경량의 단기 인메모리 저장소인  [metrics-server](https://github.com/kubernetes-sigs/metrics-server)에 의해서 수집되며  `metrics.k8s.io`  API를 통해 노출된다.

metrics-server는 클러스터 상의 모든 노드를 발견하고 각 노드의  [Kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)에 CPU와 메모리 사용량을 질의한다. Kubelet은 각각의 파드를 해당하는 컨테이너로 변환하고 컨테이너 런타임 인터페이스를 통해서 컨테이너 런타임에서 개별 컨테이너의 사용량 통계를 가져온다. Kubelet은 이 정보를 레거시 도커와의 통합을 위해 kubelet에 통합된 cAdvisor를 통해 가져온다.

> 출처: https://github.com/google/cadvisor

> 출처: https://kubernetes.io/ko/docs/tasks/debug-application-cluster/resource-usage-monitoring/
