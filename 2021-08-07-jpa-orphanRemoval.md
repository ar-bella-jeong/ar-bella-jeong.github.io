---
title: "[JPA] orphanRemoval 용도"
date: 2021-08-07 13:59:28 -0400
categories: jpa
tags:
  - jpa
comments: true
---

```java
@OneToMany(cascade = {CascadeType.ALL}, fetch = FetchType.EAGER, orphanRemoval = true)
```
보통 1:N 관계 table 설정할 때 저렇게 option을 추가해준다.
JPA에서 자식 entity의 수정은 insert update delete순으로 이어진다.
- 변경된 자식을 먼저 insert
- 기존의 자식을 NULL로 update
- orphanRemoval=true로 하면 기존 NULL로 처리된 자식을 DELETE한다.

FK(JoinColumn)값이 NULL로 변한 자식을 고아객체라 하는데 orphanRemoval option이 바로 이 고아객체를 삭제해주는 역할을 한다.

> 출처
> - https://dev-elop.tistory.com/entry/JPA-orphanRemoval-%EC%9A%A9%EB%8F%84
