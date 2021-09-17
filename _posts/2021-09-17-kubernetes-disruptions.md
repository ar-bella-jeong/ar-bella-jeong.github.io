---
title: "[Kubernetes] Disruptions(중단)"
date: 2021-09-17 19:00:28 -0400
categories: kubernetes
tags:
  - kubernetes
comments: true
---
Pod에는 다양한 유형의 중단이 발생할 수 있다.
 예상 가능한 상황에서 pod가 없어지는 것을 Voluntary disruptions 라고 하고, kernel panic이나 VM crash같은 예기치 못한 상황에서 pod가 없어지는 것을 Involuntary disruptions라고 한다.
## Pod disruption budgets
application owner는 각  application에 ㄷ해 PodDisruptionBudget(PDB)을 생성할 수 있다. PDB는 voluntary disruptions으로 인해 동시에 다운될 replica application에 pod수를 제한한다. 즉 application replica count가 특정 비율 아래로 떨어지지 않도록 하는데 사용한다.
e.g.) `spec.replicas: 5` `PDB: 4`이면 Eviction API는 한번에 하나 pod를 자발적으로 중단할 수 있다.

이는 성능을 유지하기 위해서 일정 수 의 Pod수를 유지해야 하는 경우에 유용하다.

PDB를 설정하면 관리자가 node upgrade를 위해서 node를 down시키거나 autoscaler에 의해서 node가 down될 경우, Pod 수를 일정하게 유지하지 못하면, node down이나 auto scaler에 의한 scale down을 막고, pod를 일정 수준으로 유지할 수 있을 때 다시 그 동작을 하도록 한다.

pod또는 deployment를 직접 삭제하는 대신 [Eviction API](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/#eviction-api)를 호출하여 PodDisruptionBudgets를 준수하는 tool을 사용해야 한다. ex) drain

PDB는 involuntary disruptions을 막을 수 없지만 budget에는 포함된다.

PDB는 yaml file을 이용해서 resource로 정의될 수 있다.  하기 두 값은 Pod이 숫자 및 %로 정의 가능하다.
- minAvaliable: 최소 유지
- maxUnavailable: 최대 허용 Unhealthy한 pod 수

PDB 안에 label selector를 정의해서 사용할 pod를 선택할 수 있다.
```yaml
apiVersion: policy/v1beta1  
kind: PodDisruptionBudget
metadata:
	name: arar-pdb
spec:
	minAvailable: 2  
	selector:
		matchLabels:
			app: arar-app
```

## How to perform Disruptive Actions on your Cluster
Cluster의 모든 node에 system software upgrade와 같은 중단 작업을 수행해야 하는 경우 다음과 같은 몇가지 option이 있다.
- upgrade 중 downtime을 accept 한다.
- 다른 replica cluster로 fail over한다.
	- down time은 없지만 human effort가 듬
- 중단 허용(disruption tolerant) application을 작성하고 PDB를 사용
	- no down time


> 출처
> - https://kubernetes.io/docs/concepts/workloads/pods/disruptions/
> - https://bcho.tistory.com/1305
