---
title: "MMM, MHA"
date: 2021-05-06 14:36:28 -0400
categories: database
tags:
  - database
comments: true
---

## MMM (Multi - Master Replication Manager)
- Perl 기반의 Auto Failover Open Source
- DB Server에서 Agent 실행 이후 MMM monitor와 통신을 하는 방식 ( Health check, Failover 수행)

### MMM의 구조

#### Master(Active)와 Master(Standby) 양방향 복제
- Master(Standby)는 data가 변경되지 않도록 MMM monitor로 부터 읽기모드로 제어된다.

#### Slave 추가
- Master(Active)와 Master(Standby)의 양방향 복제 + Slave
- 단방향 복제의 Slave가 하나씩 추가되는 구조. 이 역시 Master 제외하고는 모두 읽기 모드로 제어

### MMM Failover
- Master(Active)에서 Master의 역할을 뺏는 작업을 진행한다.
	- 읽기모드로 변경
	- Session kill
	- VIP 회수
- Master(Standby) 혹은 Slave로 복제를 재구성 한다.
	- 복제 재구성 전 복제 delay가 있는지 확인 후 이 때 Master(Standby) 기준으로 복제를 진행.
- Master(Standby)에 대한 읽기 모드 해제 후 VIP 할당.

> Failover의 원인?
> query가 무거운 것이 들어와서 OS의 메모리 점유율이 높은 경우
> hypervisor 장애로 인한 경우
>> Hypervisor?
>> 물리적 하드웨어에 설치된 소프트웨		어 계층으로, 물리적 머신을 다수의 가상 머신으로 분할할 수 있도록 해줌.

#### MMM Failover 과정에서 복제 깨지는 경우

Master(Active)에서 insert query 진행 후 Slave 복제를 먼저 완료 후 Master(Active)에 ACK를 보냈다. 
근데 갑자기 Master(Active)에 장애가 발생하여 Master(Standby)가 Active로 승격되면 일단 해당 query 작업은 진행이 되나 이를 Slave에도 복제를 시도하여 PK오류가 발생하는 경우가 있다.
> Multi Slave환경에서 미약하지만 복제 Crash 가능성이 존재함.

> ACK?
> TCP에서 segment를 잘 받았으면 ACK, 못받았으면 NAK

## MHA(Master High Availability)
- Perl 기반의 Auto Failover Opensource
- Agentless 방식

### MHA 구조
- 하나의 master와 slave구조로 이루어져 있다.
- **단뱡향 복제**로 이루어짐

### MHA Failover
#### MHA Failover 후속처리
- Master DB가 장애가 나는 경우 기존 Slave와의 복제를 끊고 나머지 DB들로 복제를 재구성 함.
- Master DB가 정상동작을 한 이후라도 작업이 끊겼으므로, 복제를 재구성 해주는 작업 필요.
- 
#### MHA Failover 대상
- 기준: **가장 최신의 data**를 가지고 있는 DB를 Master로 승격함.

위에 언급하였던 MMM의 복제 crush 현상 방지 위해 별도의 절차를 거침
- 복제를 구성하는 대상 : binary log, relay log file

해당 file 들을 가져와서 bin와 relay간에 차이나는 data를 추출 한다.

> 출처: https://jwdeveloper.tistory.com/215
