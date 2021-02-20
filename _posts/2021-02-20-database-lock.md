---
title: "Database Lock"
date: 2021-02-20 09:18:28 -0400
categories: database
tags:
  - database
---

Database는 자원(data)에 대해서 동시에 접근하는 경우가 생길 수 밖에 없는데 이럴 경우 data가 오염될 수 있다. 그렇게 되지 않도록 data의 일관성과 무결성을 유지해야할 필요성이 있다. DBMS(Database Management System)가 사용하는 방법이 **Lock*이다. OS lock과 유사하게 DB에도 lock이 존재하는 것이다.

## Lock
**Transaction 처리의 순차성을 보장하기 위한 방법**이다. DBMS마다 Lock을 구현하는 방식과 세부적인 방법이 다르다.
 DBMS가 자동으로 적용하거나 수동 적용도 가능하다.

### Lock의 종류
**공유(Shared) Lock과 배타(Exclusive) Lock이 있다. Shared Lock은 Read Lock이라고 불리며, Exclusive Lock은 Write Lock이라고도 불린다.**
#### Shared Lock
data를 읽을 때 사용되어지는 Lock이다. Shared lock 끼리는 동시 접근이 가능하다. 즉, 하나의 data를 읽는 것은 여러 user가 동시에 할 수 있다는 것이다. 하지만 Shared lock이 설정된 data에 exclusive lock을 사용할 순 없다.
#### Exclusive Lock
data를 변경하고자 할 때 사용되며, transaction이 완료될 때 까지 유지된다. exclusive lock는 해제 될 때까지 다른 transaction(읽기 포함)은 해당 resource에 접근할 수 없다. 다른 transaction이 수행되고 있는 data에 대해서는 함께 lock 설정이 불가능하다.
### Lock의 설정 범위(Level)
- Database
	- 전체 database를 기준으로 lock 하는 것이다. 즉 1개의 session만이 DB data에 접근이 가능하다. 해당 기능은 일반적으로 사용하지 않는다. **사용하는 때가 있다면 DB의 software version을 올린다 던지 주요한 DB update에 사용한다.**
- File
	- database file을 기준으로 lock을 설정한다. file이란 table, row등과 같은 실제 data가 쓰여지는 물리적인 저장소이다. 해당 범위의 lock은 잘 사용되지 않는다.
- **Table**
	- Table의 모든 행을 update하는 등의 전체 table에 영향을 주는 변경을 수행할 때 유용하다. 즉, **DDL(create, alter, drop 등) 구문과 함께 사용되며 DDL Lock이라고도 한다.**
- Page, Block
	- file일부인 page와 block을 기준으로 lock을 설정한다. 잘 사용되지 않는다.
- Column
	- lock 설정 및 해제에 resource가 많이 들기 때문에 일반적으로 사용되지는 않는다. 지원하는 DBMS도 많지 않다.
- **Row**
	- 1개의 row를 기준으로 lock 설정을 한다.  DML에 대한 lock으로 가장 일반적으로 사용하는 lock이다.
### 블로킹(Blocking)
**블로킹은 Lock간(배타-배타, 배타-공유)의 경합이 발생하여 특정 Transaction이 작업을 진행하지 못하고 멈춰선 상태**를 말한다. Blocking을 해소하기 위해서는 이전 transaction이 완료(commit OR rollback)되어야 한다. 이런 경합은 성능에 좋지 않은 영향을 미친다.
![](https://blog.kakaocdn.net/dn/Hkc78/btqL3wfemow/apZ9CtndatkDz9ATfpLYoK/img.png)
> lock는 transaction 내부 db level에서 발생하고, transaction이 완료되면 풀린다.

- DB를 사용하는 programming을 진행할시 주의사항
	- 한 transaction의 길이를 너무 길게하는 것은 경합의 확률을 올린다.
	- 처음부터 설계할 때 같은 data를 갱신하는 transaction이 동시에 수행되지 않도록 한다.
	- query를 오랜시간 잡아두지 않도록 적절한 tuning을 진행한다.

### 교착상태(DeadLock)
**교착상태는 두 트랜잭션이 각각 Lock을 설정하고 다음 서로의 Lock에 접근하여 값을 얻어오려고 할 때 이미 각각의 트랜잭션에 의해 Lock이 설정되어 있기 때문에 양쪽 트랜잭션 모두 영원히 처리가 되지않게 되는 상태**를 말한다.
![](https://blog.kakaocdn.net/dn/IAs6r/btqL39jMHtW/mzTqIspCi0K0n01KTKz0h0/img.png)
**그래서 교착상태가 발생하면 DBMS가 둘 중 한 트랜잭션에 에러를 발생시킴으로써 문제를 해결합니다. 교착상태가 발생할 가능성을 줄이기 위해서는 접근 순서를 동일하게 하는것이 중요**하다. 즉, 위와 같은 경우 game_master를 업데이트 한 후 game_detail을 업데이트 한다와 같은 규칙을 정해 테이블 접근의 교차가 일어나지 않도록 하는것이 중요하다.

>출처 : https://sabarada.tistory.com/121
