---
title: "MySQL Storage Engine MyISAM vs InnoDB"
date: 2021-04-30 17:29:28 -0400
categories: database
tags:
  - database
  - mysql
comments: true
---

MySQL의 Storage Engine으로 여러가지가 존재하는데, 그 중 가장 많이 사용하는 것이 MyISAM과 InnoDB이다.

## MyISAM
ISAM(Indexed Sequential Access Method)의 단점을 보완하기 위해 나온 upgrade version으로 비-트랜잭션-세이프(non-transaction-safe) table을 관리한다. InnoDB보다 먼저 출시 됬다.

InnoDB에 비하여 별다른 기능이 없으나 속도가 빠르다. 득히 <U>Select 작업 속도가 빨라 읽기 작업에 적합하다.</U>

Full-text Indexing이 가능하여 검색하고자 하는 내용에 대한 복합겁색이 가능하다.

**그러나** data 무결성이 보장되지 않아, 개발자나 DBA가 처리해야 한다.

> 데이터 무결성(Data Integrity)?
> 
> 완전한 수명주기를 거치며 data의 정확성과 일관성을 유지하고 보증하는 것을 가리키며 database나 RDBMS systemd의 중요한 기능이다.

또한 Transaction에 대한 지원이 없으며, **Table-level Lock**을 사용하기 때문에 쓰기 작업(INSERT, UPDATE) 속도가 느리다.

## InnoDB
Transaction-safe storage이다. Commit, Rollback, 장애복구, row-level locking, 외래키 등 다양한 기능을 지원한다.

data integrity에 대한 보장이 되고 동시성 제어가 가능하다.

**Row-level Lock(행 단위 Lock)을 사용**하기 때문에 (INSERT, UPDATE, DELETE)에 대한 속도가 빠르며 복구능력이 좋은 것이 장점이다.

하지만, system resource를 많이 사용하고 Full-text indexing이 불가능한 것이 단점이다.

이 두 종류의 DB를 함께 사용할 수 있긴 하나, backup방법에 차이가 있어 번거로우며 Lock에 대한 Level이 달라 사용에 문제가 생길 수 있다.
