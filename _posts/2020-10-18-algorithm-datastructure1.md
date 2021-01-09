---
title: "Data Structure"
date: 2020-10-31 21:52:28 -0400
categories: algorithm
tags:
  - algorithm
  - data structure
---

## 1. Stack
시간 복잡도 O(N)
### 사용할 때
맨 위에 있는 data가 의미가 있을 때
무언가를 뒤집어야 할 때
- C++ : std::stack
- Java: Stack
- Python3 : list

### Problem) Editor
https://www.acmicpc.net/problem/1406

string이 중간에 비거나 추가될 때, 뒤의 string을 앞이나 뒤로 shift 시키는 연산은 O(M)이라, string length가 N일 시 약 O(M^2)이라 봐야한다. 

string length가 60만 일시 M^2은 3600억이다. 1억의 1s로 가정하고 있으므로 string shift는 너무 느리다.

string 중간의 삽입으로 stack을 사용하여 cursor를 기준으로 left, right을 나눠서 풀 수있다.

**어느 특정 위치에 data를 삽입, 삭제를 효율적으로 할 수 있는** data structure인 linked list로도 해결 가능

## 2. Queue
graph 인 BFS algorithm에서 주로 queue를 사용
- C++ : STL의 queue
- Java: java.util.Queue
- Python3 : Collections.deque or 직접 구현

## 3. 덱(Deque)
양 끝에서만 자료를 넣고 양 끝에서 뺄 수 있는 자료 구조. (Double-ended queue)
원형 queue 를 extend 하여 구현할 수 있음.
BFS에서 더 다룰 예정
