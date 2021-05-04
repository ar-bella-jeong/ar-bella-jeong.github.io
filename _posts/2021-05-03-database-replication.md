---
title: "[MySQL] Replication"
date: 2021-05-04 14:41:28 -0400
categories: database
tags:
  - database
  - replication
comments: true
---

![](https://1.bp.blogspot.com/-udaBYMIAjio/VWA3gzePy2I/AAAAAAAACP0/o9eXxkuXmv4/s320/mysql%2Breplication%2B-%2Boverview%25282%2529.png)

replication은 다음과 같은 순서로 진행된다.
1.  마스터 데이터베이스가 binary log를 만들어 이벤트를 기록한다.
2.  각 슬레이브는 어떤 이벤트까지 저장되어 있는지를 기억하고 있다.
3.  슬레이브의 IO thread를 통해서 마스터에 이벤트를 요청하고 받는다.
4.  마스터는 이벤트를 요청받으면 binlog dump thread를 통해서 클라이언트에게 이벤트를 전송한다.
5.  IO thread는 전송받은 덤프 로그를 이용하여 relay log를 만든다.
6.  SQL thread는 relay log를 읽어서 이벤트를 다시 실행하여 슬레이브에 데이터를 복사한다.

## binlog dump thread
slave가 event를 요청하면 binary log에 lock을 걸고, event를 읽어 slave로 event를 전송한다. 이때, binary log를 너무 긴 시간 락하지 않기 위해서 슬레이브에 전송하기 전에 binary log를 읽고 바로 락을 해제한다.

마스터는 슬레이브에 대한 정보를 전혀 가지고 있지 않다. 슬레이브가 있는지 없는지, 몇 개의 슬레이브가 붙어있는지, 각 슬레이브가 어디까지 데이터를 복사했는지, 보내야 할 이벤트가 있는지 전혀 모른다. 덕분에 마스터는 큰 부하 없이 데이터를 복사할 수 있다.

binlog dump thread는 슬레이브가 마스터에 컨넥트할 때 생성되지만, 여러 개의 슬레이브가 붙어도 단 하나의 스레드만 생성된다.

## I/O thread
각 슬레이브는 자신이 어디까지 데이터를 복사했는지 기억하고 있다가 master에게 다음 event를 전송해달라고 요청한다. 그 뒤 Relay log에 저장하는데, 따라서 이 thread가 정지된 상황에서 master의 bin log가 지워지면, slave는 master의 data를 복제할 수 없다.

## SQL Thread
Relay log를 읽어 실행 시키고, Relay log를 지운다. SQL을 실행시키는 스레드와 마스터로부터 값을 복사해오는 스레드가 분리되어 있다는 것은 매우 중요한 특징이다.

 보통 MySQL의 데이터를 백업하여 아카이브를 만드는 것은 mysqldump를 이용한다.  하지만 mysqldump는 MySQL이 쿼리를 실행하고 있을 때 실행되면 데이터가 깨지는 문제가 발생한다. 하지만 replication을 이용하면, 슬레이브의 SQL thread만 정지시키면 되기 때문에 안전하게 백업을 만들 수 있다.

> 출처: https://blog.seulgi.kim/2015/05/how-mysql-replication.html
