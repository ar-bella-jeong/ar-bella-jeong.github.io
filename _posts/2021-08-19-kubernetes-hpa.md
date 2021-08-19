---
title: "Horizontal Pod Autoscaler"
date: 2021-08-19 16:33:28 -0400
categories: kubernetes
tags:
  - kubernetes
comments: true
---

Horizontal Pod Autoscaler는 CPU 사용량 (또는 [사용자 정의 메트릭](https://git.k8s.io/community/contributors/design-proposals/instrumentation/custom-metrics-api.md), 아니면 다른 애플리케이션 지원 메트릭)을 관찰하여 레플리케이션 컨트롤러(ReplicationController), 디플로이먼트(Deployment), 레플리카셋(ReplicaSet) 또는 스테이트풀셋(StatefulSet)의 파드 개수를 자동으로 스케일한다. Horizontal Pod Autoscaler는 크기를 조정할 수 없는 오브젝트(예: 데몬셋(DaemonSet))에는 적용되지 않는다.

컨트롤러는 사용자가 지정한 목표값과 일치하도록 레플리케이션 컨트롤러 또는 디플로이먼트에서 레플리카 개수를 주기적으로 조정한다.

## Horizontal Pod Autoscaler는 어떻게 작동하는가?
![Horizontal Pod Autoscaler 다이어그램](https://d33wubrfki0l68.cloudfront.net/4fe1ef7265a93f5f564bd3fbb0269ebd10b73b4e/1775d/images/docs/horizontal-pod-autoscaler.svg)
Horizontal Pod Autoscaler는 컨트롤러 관리자의 `--horizontal-pod-autoscaler-sync-period` 플래그(default 15초)에 의해 제어되는 주기를 가진 컨트롤 루프로 구현된다.
- 파드 단위 리소스 메트릭(예 : CPU)의 경우 컨트롤러는 각 파드에 대한 리소스 메트릭 API에서 메트릭을 가져온다. 파드의 컨테이너 중 일부에 적절한 resource request가 설정되지 않은 경우 해당 메트릭에 대해 아무런 조치도 취하지 않는다.
- 파드 단위 사용자 정의 메트릭의 경우, 컨트롤러는 사용률 값이 아닌 원시 값을 사용한다는 점
- 오브젝트 메트릭 및 외부 메트릭의 경우 `autoscaling/v2beta2` API 버전에서는, 비교가 이루어지기 전에 해당 값을 파드의 개수로 선택적으로 나눌 수 있다.

HorizontalPodAutoscaler는 보통 일련의 API 집합(`metrics.k8s.io`, `custom.metrics.k8s.io`, `external.metrics.k8s.io`)에서 메트릭을 가져온다. `metrics.k8s.io` API는 대개 별도로 시작해야 하는 metric-server에 의해 제공된다.

### 알고리즘 세부 정보
```
Desired replica count = ceil[current replica count * ( current metric value / desired metric value )]
```
`targetAverageValue` 또는 `targetAverageUtilization`가 지정되면, `currentMetricValue`는 HorizontalPodAutoscaler의 스케일 목표 안에 있는 모든 파드에서 주어진 메트릭의 평균을 취하여 계산된다. 허용치를 확인하고 최종 값을 결정하기 전에, pod가 not ready한 상태거나 readiness probe fail로 service가 떨어져 나온 경우 metric 계산에서 제외한다.

 누락된 metric이 있는 경우 scale in시 100% 사용, out시 0%를 소비하고 있다고 가정하고 보수적으로 계산한다. 규모를 약화 시키는 것이다.

 여러 metric이 지정된 경우 각 metric에 의해 계산이 수행되고 가장 큰 값이 선택된다.


> 출처: https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale/
