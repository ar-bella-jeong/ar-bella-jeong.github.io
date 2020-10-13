---
title: "Servlet"
date: 2020-10-13 18:39:28 -0400
categories: java
tags:
  - java
  - servlet
---

## Java Servlet
### 1. Servlet
- Web Programming에서 client의 request를 처리하고 그 결과를 다시 client에게 전송하는 servlet class의 구현 규칙을 지킨 Java Programming 기술이다.
- Java를 사용하여 Web을 만들기 위한 기술

#### [Servlet 특징]
- client의 요청에 대해 동적으로 작동하는 
Web application component.
- MVC Pattern에서 controller로 이용
- HTML을 사용하여 요청에 응답
- HTML 변경 시 servlet을 재 컴파일
#### [Servlet 동작 방식]
![Image for post](https://miro.medium.com/max/1253/1*depxgx6grHz56KJSyTkfxw.png)

### 2. Servlet Container
request를 받아주고 response를 할 수 있게 web server와 socket을 만들어 통신함. 대표적인 예로 tomcat.
#### [Servlet container의 역할]
