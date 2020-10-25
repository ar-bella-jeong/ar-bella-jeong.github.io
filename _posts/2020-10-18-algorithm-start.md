---
title: "Start Algorithm"
date: 2020-10-18 21:12:28 -0400
categories: algorithm
tags:
  - algorithm
---
## Algorithm
- 스스로 풀어내는 것이 나쁜 생각이 아니나, 조금 고민해보고 모르겠으면, 답을 보고 이해하자!
- **이 문제를 푸는 알고리즘이 무엇인지** 각각의 알고리즘의 특징을 알고, 왜 그 알고리즘으로 다른 문제를 풀 수 있었는지를 위주로 기억해서 문제에 적용한다.

## Time Complexity
### Big O Notation
- 알고리즘의 효율성을 나타내는 표기법
- 최악의 경우를 기준으로 산정
>#### Example
>- N명의 사람이 M개의 게임을 하는데 걸리는 시간이 H(i)
>- 모든 사람이 게임을 완료하는데 걸리는 시간 = **O(max(H(i))**
>> 미리 계산해보고 timeout이 되지 않을 것 같을 때 구현하는 것이 Good!

#### 1s의 input size
- O(1)
- O(logN)
- O(N) : 1억
- O(NlogN) : 5백만
- O(N^2) : 1만
- O(N^3) : 500
- O(2^N) : 20
- O(N!) : 10
>  **Note:** 이를 통해 문제에서 요구하는 Algorithm의 Time Complexity을 유추할 수 있음.
> >실제로는 이보다 성능이 더 좋게 나옴!

#### 계산
- Big O Notation에서 상수는 버린다
> O(3N^2) = O(N^2)
> O(5) = O(1)
- 변수가 같으면  큰 것만 빼고 다 버린다.
> O(N^2 + NlogN) = O(N^2)
- 변수가 다르면 놔둔다
> O(N^2 + M)

## Memory
### 사용한 메모리
- 보통 가장 많은 공간을 사용하는 것은 배열
> int a[10000][10000]; -> 10000*10000*4B = 400,000,000B = 381.469MB
>> O(N) = 1억/1s

## 입출력
### C++ 입출력
- cin/cout은 scanf/printf 보다 느리기 떄문에, 입/출력이 많은 문제의 경우에는 scanf/printf를 사용하는 것이 좋다.
> cin/cout의 경우 아래 세 줄을 추가하면 scanf/printf만큼 빨라진다.
>> ```cpp
>> ios_base::sync_with_stdio(false); // C++, C standard stream i/o 연산의 비동기화 -> C++ stream이 각각의 i/o 연산에 대해 buffer를 사용하여 i/o 연산 속도를 크게 향상
>> cin.tie(nullptr); // cin을 cout으로 부터 untie(이전 output flush를 막음)
>> cout.tie(nulltpr); // 사용 안해도 충분(cin untie시 기본적으로 cout은 buffer가 가득차거나 수동적으로 flush를 시켜주기 전까지 출력 X)
>> ```
>> 이 경우에는 cin/cout와 scanf/printf를 섞어 쓰면 안된다. (C, C++ 입출력 순서 보장 X)

## Example
### 1. A+B-4
https://www.acmicpc.net/problem/10950
>**If not provide input size**
>>- C: scanf -> return 읽어들인 input 개수
>>- C++: cin -> return cin object, 단 조건식안에 들어가면 예외적으로 operator에 의해 bool로 바뀜. (scan 여부)
>>- Java: sc.hasNextInt() -> Scanner class의 member function으로 확인 가능

출처 :  https://www.acmicpc.net/
