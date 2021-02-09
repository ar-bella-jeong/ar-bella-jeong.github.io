---
title: "DDL, DML, DCL"
date: 2021-02-09 20:41:28 -0400
categories: database
tags:
  - database
---

명령어 종류 | 명령어 | 설명
 ---|:---:|---: 
 데이터 조작어(DML: Data Manipulation Language) | SELECT, INSERT, UPDATE, DELETE | db에 들어있는 data를 조회하거나 변형을 가하는 명령어
데이터 정의어(DDL: Data Definition Language) | CREATE, ALTER, DROP, RENAME, TRUNCATE | Table과 같은 data 구조를 정의하는데 사용됨
데이터 제어어(DCL: Data Control Language) | GRANT, REVOKE | db에 접근하고 객체들을 사용하도록 권한을 주고 회수하는 명령어
트랜잭션 제어어(TCL: Transaction Control Language)|COMMIT, ROLLBACK, SAVEPOINT|DML에 의해 조작된 결과를 transaction 별로 제어하는 명령어

> 출처: https://brownbears.tistory.com/180
