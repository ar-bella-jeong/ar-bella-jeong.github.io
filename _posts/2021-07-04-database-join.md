---
title: "[DB] Join"
date: 2021-07-04 18:03:28 -0400
categories: database
tags:
  - database
comments: true
---

![](https://t1.daumcdn.net/cfile/tistory/99219C345BE91A7E32)
## INNER join
ON 대신 WHERE를 쓸 수 있으며 일반적으로 사용하는 JOIN이다.
![](https://t1.daumcdn.net/cfile/tistory/243BF43A58340E0A06)

## LEFT OUTER, RIGHT OUTER join
join 조건 시 일치 하지 않는 값이 있더라도 표시하게 한다.
- LEFT OUTER = LEFT
- RIGHT OUTER = RIGHT
![](https://t1.daumcdn.net/cfile/tistory/26310B3458340C9F1C)

## CROSS join
집합 곱의 개념이다.

A= {a, b, c, d} , B = {1, 2, 3} 일 때
A CROSS JOIN B 는
(a,1), (a, 2), (a,3), (b,1), (b,2), (b,3), (c, 1), (c,2), (c,3), (d, 1), (d, 2), (d,3)
와 같이 결과가 나타난다. 결과의 계수는 n(A) * n(B) = 4 * 3 = 12 이다.

```sql
SELECT  s._id, s.title, gg.name  
FROM girl_group AS gg 
CROSS JOIN song AS s;

SELECT  s._id, s.title, gg.name  
FROM girl_group AS gg, song AS s;  
```
두 쿼리의 결과는 같다.

cross join은 너무 많은 레코드를 생성할 위험이 있기 때문에, 많이 사용하지는 않는다.  


> 출처: https://futurists.tistory.com/17
