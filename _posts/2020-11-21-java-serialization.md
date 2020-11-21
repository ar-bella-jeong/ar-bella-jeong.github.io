---
title: "Java Serialization"
date: 2020-11-21 15:45:28 -0400
categories: java
tags:
  - java
---

## What is Java serialization?
- JVM의 memory(heap or stack)에 있는 object를 byte 형태로 변환 및 byte to object하는 역직렬화를 얘기한다.

## How to use Java serialization?
### Java Serialization condition
- primitive, java.io.Serializable interface를 상속 받은 object는 직렬화 할 수 있는 basic condition을 가진다.
```java
...
public Company implements Serializable {
	private String name;
	private String address;
	private int employeeCount;
	...
}
```
### Serialization method
use java.io.ObjectOutputStream
```java
Company company = new Company("ARAR", "Seoul", 20);
try(ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
	try(ObjectOutputStream oos = new ObjectOutputStream(baos)){
		oos.writeObject(company);
		byte serialData = baos.toByteArray();
	}
}
```
### Deserialization condition
 Java serialization target object has to have same **serialVersionUID**. 
```java
private static final long serailVersionUID = 1L;
```
> **What is serailVersionUID?**
serialVersionUID is unique ID using for serialization, If not declare, JVM automatically create it by default.
 Therefore, there aren't any problem, however, Java strongly recommends declaring it.
> <span style="color:green">
>* Since calculating default serialVersionUID by JVM sensitively reflects the detail of class, it may vary depending on compiler, which may cause unexpected InvalidClassException during deserialization.
> </span>
