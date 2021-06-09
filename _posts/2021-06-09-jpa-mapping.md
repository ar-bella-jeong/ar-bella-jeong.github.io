---
title: "[JPA] 연관 관계 mapping - @OneToMany @ManyToOne @OneToOne"
date: 2021-06-09 18:42:28 -0400
categories: jpa
tags:
  - jpa
  - database
  - spring
comments: true
---

## 1. 연관 관계 mapping시 고려사항
### 1-1. 다중성
-   다대일 [N:1] :  `@ManyToOne`
-   일대다 [1:N] :  `@OneToMany`
-   일대일 [1:1] :  `@OneToOne`
-   다대다 [N:M] :  `@ManyToMany`
### 1-2. 단방향 or 양방향
### 1-3. 연관 관계의 주인
-   테이블은 외래 키 하나로 두 테이블이 연관관계를 맺음
-   객체 양방향 관계는 A->B, B-> A처럼 참조가 2군데

## 2. 다대일 [N:1]
여기서 연관 관계의 주인은 N이다.
### 2-1. 다대일 단방향
User = N : Region : 1
User에서 Region을 참조 한다.
```java
@Entity
@Getter
@Setter
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	...
	@ManyToOne
	@JoinColumn(name="region_id") // FK
	private Region region;
}
```
```java
@Entity
@Getter
@Setter
public class Region {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private  Long id;
}
```
해당 관계는 가장 많이 사용하며, 다대일의 반대는 일대다 이다.
### 2-2. 다대일 양방향
User=N : Region=1
연관 관계 주인이 FK를 관리하는데, 반대쪽인 Region은 어차피 읽기만 관리 가능하므로, List를 추가하기만 하면 된다. 이때, mappedBy로 연관관계의 주인을 읽을 것이라는 것 명시가 중요하다.
```java
@Entity
@Getter
@Setter
public class Region {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private  Long id;
	
	@OneToMany(mappedBy = "region") //참조를 당하는 쪽에서 읽기만 가능! 
	private List<Member> members = new ArrayList<>();
}
```

## 3. 일대다 [1:N]
### 3-1. 일대다 단방향
권장하지 않는 방법이다.
Region을 중심으로 해서 Region에서 FK를 관리한다. Region에선 User를 알고 싶지만, User에서 Region을 알고 싶지 않는 경우이다.
```java
@Entity
@Getter
@Setter
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
}
```
```java
@Entity
@Getter
@Setter
public class Region {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private  Long id;
	
	@OneToMany
	@JoinColumn(name = "member_id")
	// 기본은 상대 table의 PK를 join하고 대상 table column을 지정하려면 referencedColumnName을 사용한다.
	private List<Member> members = new ArrayList<>();
}
```
상기와 같이 구성하면 Region을 건듬에도 불구하고 User table에 영향이 간다. 이런 구조는 항상 **다(N) 쪽에 외래 키가 있다.**

- @JoinColumn을 꼭 사용해야 한다. 그렇지 않으면 Join table 방식을 사용한다. (중간에 region_user 라는 region_id, user_id를 가지고 있는 중간 table이 생성 됨.) 하여 운영이 어렵다.

- Entity가 관리하는 FK가 다른 table에 존재하다 보니, 연관 관계 관리를 위해 추가로 UPDATE SQL이 실행된다.

- 이 방법 보단 **다대일 양방향 매핑**을 사용하자! 쓰지 않는 entity에도 관계를 설정해야 하겠지만..

### 3-2. 일대다 양방향
사실 공식적으로 존재하는 mapping이 아니라서 하기와 같이  우회해서 구현해야 한다.
```java
@Entity
@Getter
@Setter
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@ManyToOne
	@JoinColumn(name="region_id", insertable=false, updatable=false)
	private Region region;
}
```
상기와 같은 경우는 모두 @JoinColumn이 붙어서 둘다 연관관계의 주인이 된다. 값은 다 쓰지만 writing을 전부 막아 read 전용으로 만든다. 관리는 Region이 하고 User는 읽기만 하는 것이다. 해당 경우도 **다대일 양방향을 사용**하는 것이 좋다.

## 4. 일대일 [1:1]
FK에 database unique 조건이 추가되어야 일대일 관계가 된다. **다대일[N:1] 단방향 관계 매핑과 annotation만 다르고 거의 유사하다.**

> @JoinColumn은 default 값이 있지만 지저분하니 name을 정해주자.
```java
@Entity
@Getter
@Setter
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@OneToOne
	@JoinColumn(name="device_id")
	private Device device;
}
```
```java
@Entity
public class Device {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@OneToOne(mappedBy="user")
	private User user;
}
```
주 table(많이 접근하는 table)에 FK를 설정해서 대상 객체를 참조하는 것처럼 구성하면 된다. 이는 **객체지향 개발자들이 선호하고, JPA 매핑이 편리하다.** 

- 역으로 대상 table에 FK가 존재하는건 전통적인 database 개발자들이 선호하는 방식이다. 일대일에서 일대다로 변경 시 Device table이 User의 FK을 가지고 있으므로 table 구조를 유지할 수 있다.  단점으론 User로 접근 하는 경우가 많아 양방향 mapping을 구현해야만 한다. 또한 JPA의 기본 proxy의 한계로 **Lazy loading으로 설정해도 항상 즉시 로딩이 된다.**  그 이유는 대상 table에 외래 key가 있어 User로 접근 시 Device를 얻어오려면 결국 Device table을 조회해야 하기 때문에 Proxy 객체로 던져주질 않는다.

## 4. 다대다 [N:M]
실무에선 사용하지 않는 것을 추천한다. 
이를 지원하려면 일대다 다대일 관계로 풀기 위해 보조 table을 만들어 연결해줘야 한다. 
![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/16_many_to_many.png?raw=true)
@ManyToMany annotation을 사용하고 @JoinTable로 연결 table을 지정해줄 수 있다.
### 4-1. 다대다 단방향
```java
@Entity
public class User{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@ManyToMany
	@JoinTable(name="user_device")
	private List<Device> devices = new ArrayList<>();
}
```
하위 Join table은 자동으로 생성된 것이다.
```sql
Hibernate:   
 create table user_device (  
 user_id bigint not null,  
 device_id bigint not null  
 )
 ...
Hibernate:   
 alter table user_device 
 add constraint k38fhajsdf984hhdfh
 foreign key (user_id )   
 references User  
Hibernate:   
 alter table user_device 
 add constraint 39uajdfkjfh934ifhgaksdfj
 foreign key (device_id)   
 references Device
```

### 4-2. 다대다 양방향

 ```java
@Entity
public class Device {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@ManyToMany(mappedBy="devices")
	private List<User> users = new ArrayList<>();
}
```
  
편리해 보이지만 실무에서 사용하면 안된다. join table이 단순히 연결만 하는 것이 아니라, 추가 data가 많이 들어갈 수 있다. 하지만 mapping 정보만 넣는것이 가능하고 추가정보를 넣는 것 자체가 불가능한다. 그리고 중간 table이라 예상하지 못하는 query들이 execute 될 수 있다.

  ### 4-3. 다대다 한계 극복
  - **연결 table을 entity로 승격** 시킨다. 그리고 @ManyToOne @OneToMany로 바꿔주는 것이다. 또한 연결 table에 **독립적으로 generated 되는 id를 사용해야 한다.** 두 table에 종속되지 않고 더 유연하게 개발 할 수 있다. 비지니스적인 제약 조건이 커지면 PK를 운영중에 update하는 상황이 발생한다.
```java
@Entity
public class User{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@OneToMany(mappedBy="user")
	private List<Device> devices = new ArrayList<>();
}
```
```java
@Entity
public class Device {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@OneToMany(mappedBy="device")
	private List<User> users = new ArrayList<>();
}
```
```java
@Entity
public class Order {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@ManyToOne
	@JoinColumn(name="user_id")
	private User user;
	
	@ManyToOne
	@JoinColumn(name="device_id")
	private Device device;
}
```

> 출처: https://ict-nroo.tistory.com/127
> 
> 출처: https://jyami.tistory.com/21
> 
> 출처: https://ict-nroo.tistory.com/126
