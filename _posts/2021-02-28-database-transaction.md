---
title: "Transaction"
date: 2021-02-28 17:37:28 -0400
categories: database
tags:
  - database
  - transaction
---

## Transaction이란?
Computer Science 분야에서 Transaction은 **쪼개질 수 없는 업무처리의 단위**를 의미한다.

부분 작업들 여러개가 모여진 Transaction을 처리하기 위해 Database은 다음의 두가지 명령어를 활용하는데, 바로 Commit과 Rollback이다.

- Commit: 모든 부분작업이 정상적으로 완료하면 이 변경사항을 한꺼번에 DB에 반영.
- Rollback: 부분 작업이 실패하면 Transaction 실행 전으로 되돌린다.

이때, 모든 연산을 취소하지 않고 정해진 부분까지만 되돌리고 싶을 때 사용하는 것이 savepoint이다.

- SAVEPOINT
	- 취소하려는 지점을 SAVEPOINT로 명시한 뒤 ROLLBACK TO <savepoint 이름>;을 실행하면 지정한 해당 SAVEPOINT 지점까지 처리한 작업이 ROLLBACK 된다.

## Database Isolation
다수의 Transaction 경쟁 시 발생할 수 있는 여러 case의 문제가 있다. 아래는 그 중 하나이다.
- Dirty read
	- (read-uncommited 일 시)
	- Transaction A가 1->2로 modify하고, 아직 commit하지 않은 상태에서 Transaction B가 값을 읽는 경우 2를 조회
	- 그 후 A가 rollback되면 B가 읽은 값은 잘못된 값임

위와 같은 상황을 방지하기 위해 Database에는 Isolation이라는 속성이 존재한다. Transaction에서 일관성이 없는 데이터를 허용하도록 하는 수준이다.

- read_uncommited (level0)
	- commit 되지 않은(transaction 처리중인) 데이터에 대한 읽기를 허용
- read_commited (level1)
	- transaction이 commit된 확정 data만 읽기 허용.
- Repeatable_read (level2)
	- transaction이 완료될 때 까지 select문이 사용하는 모든 data에 Shared lock이 걸리므로, 다른 User는 그 영역에 해당하는 data에 대한 수정이 불가능
	- 선행 transaction이 읽은 data는 transaction이 종료될 때까지 부행 transaction이 갱신하거나 삭제가 불가능 하기때문에 일관성 있는 result를 return한다.
- Serializable (level3)
	- data의 일관성 및 동시성을 위해 MVCC(Multi Version Concurrency Control)을 사용하지 않는다.
	- transaction이 완료될 때까지 select문장이 사용하는 모든 data에 shared lock이 걸린다.
