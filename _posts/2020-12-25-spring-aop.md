---
title: "Spring AOP"
date: 2020-12-25 11:48:28 -0400
categories: spring
tags:
  - java
  - spring
---

**AOP**(Aspect Oriented Programming)
> **Aspect Oriented(관점  지향)?**
> - 핵심적인 관점: 핵심 비지니스 logic
> - 부가적인 관점 : 핵심 logic을 실행하기 위한 database 연결, logging, file I/O 등
## AOP 주요 개념
1. Concern(관심사)의 분리와 Weaving
- Weaving: 분리한 concern을 다시 module에 삽입하는 것
- Spring AOP뿐만 아니라 다양한 AOP framework에서도 같은 뜻으로 사용
2. Spring AOP에서 쓰이는 용어
- Aspect: 부가기능의 module화
- Target: Aspect를 적용하는 곳(class, method...)
- Advice: 부가기능을 실제 구현한 곳
> **Spring의 Advice type**
> Advice Type | Description
> ---|:---:|
 > Around Advice |  Jointpoint 앞과 뒤에서 실행되는 Advice
 > Before Advice |  Jointpoint 앞에서 실행되는 Advice
 > After Returning Advice | Jointpoint method 호출이 정상적으로 종료된 뒤에 실행되는 Advice
 > After Throwing Advice | 예외가 던져질 때 실행되는 Advice
 > Introduction | Class에 interface과 구현을 추가하는 특수한 Advice
 
- JointPoint: Advice가 적용 될 위치 ex) method 진입, constructor call 시점, field값 get할 경우 등
- PointCut: JointPoint의 상세 spec 정의. 하나 또는 복수의 Jointpoint를 하나로 묶은 것을 Pointcut이라고 한다. ex) 'A란 method의 진입 시점에 호출 할 것'
- Advisor: Advice와 Pointcut을 하나로 묶어 다루는 것을 Advisor라고 함. Spring AOP에만 존재함.
## Spring AOP 특징
- Proxy Pattern 기반의 AOP 구현체

## 실전 AOP
1. AspectJ style의 AOP 이용
AspectJ는 제록스 팔로알토 연구소에서 개발했던 AOP의 구현으로, 현재 eclipse.org 개발 project에서 관리하고 있다.
- Pointcut 정의 구문
method 실행을 Pointcut으로 할 경우의 expression 속성 값은 다음과 같이 기술한다.
**execution(접근 수준 result값의 package name, class name, method name(parameter type, parameter type..) throws exception type)**
Pointcut에는 wildcard를 사용할 수 있으며 논리 연산자도 사용 가능하다.
- Pointcut 정의에 사용할 수 있는 wildcard

wildcard | description
 ---|:---:
*|'.'을 포함하지 않는 임의의 문자열
..|'.'을 포함하는 임의의 문자열
+|하위 class또는 하위 interface

```java
package sample1;

import org.aspectj.lang.ProceedingJointPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.util.StopWatch;

@Aspect
public class LoggingSample {
	@Around("execution(* sayHello())")
	public Object logAround(ProceedingJointPoint pjp) throws Throwable {
	...
```

- AspectJ의 annotation

annotation| description
 ---|:---:
@Aspect|Advice인 class에 mark를 부여하는 annotation
@Around|Around Advice인 method에 부여하는 annotation
@Before|Before Advice인 method에 부여하는 annotation
@After|After Advice인 method에 부여하는 annotation
@AfterReturing|After Returning Advice인 method에 부여하는 annotation
@AfterThrowing|After Throwing Advice인 method에 부여하는 annotation
