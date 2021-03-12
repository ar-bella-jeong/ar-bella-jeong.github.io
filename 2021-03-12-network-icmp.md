---
title: "ICMP (Internet Control Message Protocol)"
date: 2021-03-12 15:11:28 -0400
categories: network
tags:
  - network
comments: true
---

TCP/IP에서 IP packet을 처리할 때 발생되는 문제를 알려주는 protocol 이다.

IP는 오로지 packet을 destination에 도달시키기 위한 content로만 구성되어 있다. 

 하여, 만일 전달해야 할 host가 비정상적인 경우에 출발지 host에 이러한 사실을 알려야하지만, IP에는 그러한 error에 대한 처리 방법이 명시되어 있지 않다.

이러한 IP으 부족한 점을 메꾸기 위하여 사용되는 것이 ICMP protocol이다.

connection error 상황이 발생할 경우 IP header에 기록되어 있는 출발지 host로 **error 상황(error message, code)을 보내주는 역할을 수행**한다.

error message로는 크게 **Destination Unreachable, Time Exceeded(시간이 오래걸려 목적지에 도달하지 못함), Redirect, source quench** 등이 있다.

> **Source Quench**
> - ICMP v4 error message중 하나 (현재, 표준에서 제외됨. 비현행)
	> 	- network 상의 통신량이 폭주하여 목적지 또는 router 등의 memory나 buffer 용량을 초과하여 IP 데이터그램이 유실 되는 상태가 되면,  error message를 송신 측에 통보하는 일종의 flow control 및 혼잡 제어의 역할을 한다.
>> **Datagram?**
>> - Internet을 통해 전달되는 정보의 기본 단위
>> ![](https://postfiles.pstatic.net/20110808_181/twers_13127843467850CwVU_PNG/ip_datagram.png?type=w2)
>> version: IP protocol의 version (IPv4, IPv6)

error message안에 error code도 존재한다. 예를 들면 Destination Unreachable에 관련한 error cod는 Network Unreachable(Code: 0), Host Unreachable(Code: 1), Protocol Unreachable(Code: 2), Port Unreachable(Code: 3) 이 있다.

통신 유뮤 확인 시 자주쓰는 ping도 ICMP protocol을 이용한 방법이다.

> 출처: https://m.blog.naver.com/PostView.nhn?blogId=rbdi3222&logNo=220602423771
