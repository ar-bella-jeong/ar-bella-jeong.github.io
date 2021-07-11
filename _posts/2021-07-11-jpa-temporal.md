---
title: "[JPA] @Temporal Annotation"
date: 2021-07-11 18:14:28 -0400
categories: jpa
tags:
  - jpa
comments: true
---
JPA를 사용할 때, 보통 날짜에 java.util.Date 객체를 사용한다.

하지만, DB에는 여러 형태의 날짜가 존재한다.
date(년월일), time(시분초), timestamp(년월일 시분초) 등의 type들이 존재한다.

```java
import java.util.Date;
import lombok.Data;
import javax.persistence.*;

@Entity
@Data
public class Article {
	@Id
	@GeneratedValue
	int id;

	String subject;

	@Column(length = 100000000)
	String content;

	@Temporal(TemporalType.DATE)
	Date regDate;

	@Temporal(TemporalType.TIME)
	Date regTime;
	
	@Temporal(TemporalType.TIMESTAMP)
	Date regTimestamp;
}
```
상기 예제를 보면 모두 Date 객체 이지만, @Temporal이라는 annotation을 사용하여, DB type에 맞도록 mapping할 수 있다.

- TemporalType.Date : 년-월-일 의 date 타입
- TemporalType.Time : 시:분:초 의 time 타입
- TemporalType.TIMESTAMP : date + time 의 timestamp(datetime) 타입

annotation을 사용하지 않을 경우 기본값은 timestamp이다.

JPA database dialect(방언)에 의해, database type에 따라 자동으로 작성된다.
> 출처
> - https://devofhwb.tistory.com/93
