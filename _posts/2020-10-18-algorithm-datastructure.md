---
title: "Data Structure"
date: 2021-01-09 19:44:28 -0400
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

## 4. Infix Notation(중위 표기법)
- 중위 표기법(Infix Notation)은 일반적으로 수식을 표기할 때 사용하는 방법
- 사람이 보기 편리하나, 컴퓨터가 이 수식을 처리하기 까다로움
- 연산자를 피연산자의 사이에 두는 방식
- ex) 1+2, 3*4, 2-1

## 5. Postfix Notation(후위 표기법)
- 연산자를 피연산자 다음에 두는 것
- 컴퓨터가 수식을 특별한 변환없이 처리할 수 있음.
- ex) 1 2 +, 3 4 *, 2 1 -
- 전위 표기법(Prefix Notation)은 연산자를 앞에 두는 방식이며,  폴란드 표기법(Polish Notation) 이라고 함
- 후위 표기법은 역폴란드 표기법(Reverse Polish Notation)이라고도 함.
>**중위 표기법 -> 후위 표기법**
> - 3+2*3 -> 323*+
> - 3 + 2 * 3 -> 3 2 3 * +
> - (3+2) * 3 -> 32+3*

>*후위 표기법으로 표현된 식을 계산하는 방법*
>1. 피연산자는 stack에 넣음
>2. 연산자를 만나면 피연산자 2개를 stack에서 꺼내 계산하고, 계산된 결과를 다시 stack에 넣음

## 6. Shunting-yard Algorithm(차량기지 알고리즘)
- 중위 표기법으로 표현된 식을 후위 표기법으로 바꾸는 알고리즘.
- 연산자를 저장하는 stack을 기반으로 이루어져 있음. 

1. Example
- stack의 top에 있는 연산자보다 우선순위가 작거나 같으면, stack에 있는 연산자를 result에 추가함.

3 + 2 X 3 - 4

연산자/피연산자 | 연산자 stack | result
 ---|:---:|---: 
 3 |  | 3
 + | + | 3
 2|+|32
 X|+X|32
 3|+X|323
 -|+|323X
  |||323X+
  ||-|323X+
  4|-|323X+4
  |||323X+4-
 
 - 괄호가 있을 경우 '('는 연산자 stack에 넣고 ')'가 나오면 여는 괄호가 나올 때까지 연산자 stack에서 계속해서 연산자를 꺼냄. **(result에 가로는 제외)**
 

> 출처: [codeplus](https://code.plus/)
