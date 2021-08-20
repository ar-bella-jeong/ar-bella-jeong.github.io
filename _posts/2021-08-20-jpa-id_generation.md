---
title: "[JPA] @Id Generation.AUTO, IDENTITY 차이"
date: 2021-08-20 14:34:28 -0400
categories: jpa
tags:
  - jpa
comments: true
---
## @GeneratedValue(strategy=GenerationType.AUTO) 
하기는 insert query 실행 시 generate된 DML이다.
```sql
Hibernate: select next_val as id_val from hibernate_sequence for update
Hibernate: update hibernate_sequence set next_val= ? where next_val=?
Hibernate: insert into ex_entity (name, id) values (?, ?)
```
insert query 전에 hibernate_sequence table에 select 및 update query가 실행된 것을 볼 수 있다.
Id 생성을 위해 hibernate_sequence table의 sequence 값을 가져와  그 값을 id로 사용한다. 이 설정이 @Id @GeneratedValue default로 동작 한다.
![](https://blog.kakaocdn.net/dn/v4sKR/btqF1TGoDiF/J76uneFABDJtZ8tKvphk91/img.png)
![](https://blog.kakaocdn.net/dn/T3LjK/btqF1Atwm7t/jsyC4s3rOkb2V0QTtn4fck/img.png)

## @GeneratedValue(strategy=GenerationType.IDENTITY) 
insert query 가 PK 값 없이 수행되고, database의 auto_increment 동작이 수행 된다.
ddl-auto: create를 사용중이라면 pk option이 auto_increment로 생성된다.

## 주의 사항
### maximum-pool-size
spring.jpa.datasource.hikari.maximum-pool-size= 1
repeatCnt=3
-   **@GeneratedValue(strategy = GenerationType.AUTO) 경우 테스트**

connection을 가져올 수 없어 timeout이 발생. 
save() 수행 시 hibernate_sequence table에 select, update를 먼저 실행하는데 이때  별개의 sub transaction을 생성하면서 추가적인 Connection을 사용하게 된다. 
(아마 다른 transaction에 묶일 시 빨리 commit되지 않아 다른 session에서 seq table lock이 걸릴 것을 생각한 듯..)
이전 사용한 connection이 아직 반납되지 않아 deadlock이 발생.

-   **@GeneratedValue(strategy = GenerationType.IDENTITY) 경우 테스트**

수행 성공. 심지어 repeatCnt=50 으로 테스트해도 문제없다.


> 출처
> - https://lion-king.tistory.com/entry/JPA-JPA-Id-GenerationTypeAUTO-IDENTITY

