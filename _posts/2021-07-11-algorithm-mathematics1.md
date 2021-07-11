---
title: "[Algorithm] Mathematics 1"
date: 2021-07-11 13:41:28 -0400
categories: algorithm
tags:
  - algorithm
comments: true
---

## Modular Arithmetic (나머지 연산)
- ( A+ B ) % M = (( A % M ) + ( B % M )) % M
- ( AX B ) % M = (( A % M ) X ( B % M )) % M
- ( A - B ) % M = ((A mod M) - (B mod M) + M) mod M -> 음수가 발생할 수 있으므로

> language 별 minus 경우 결과의 부호
> -		ex) (6%3 - 5%3) % 3일 경우
> 		C11, C++14: -2
> 		Java: -2
> 		Python3: 1

## Greatest Common Divisor (최대공약수)
- 줄여서 GCD라고 씀
- 최대공약수가 1인 두 수를 서로소(Coprime)라고 함.
- Euclidean algorithm(유클리드 호제법)을 이용하면 빠르게 계산 가능
>a%b = r이면, GCD(a,b) = GCD(b,r) 이 같다. r이 0이면 그 때, b가 최대 공약수이다.

## Least Common Multiple (최소공배수)
- 줄여서 LCM이라고 함.
- GCD를 응용해서 구할 수 있음.
> g = GCD(a, b) 일 경우, LCM(a,b) = g * (a/g) * (b/g) 이다.

## Prime Number (소수)
- 약수가 1과 자기 자신 밖에 없는 수
- 두 가지 종류의 Algorithm이 있음.
	- Prime number인지 판별
	- 숫자 N이하의 Prime numbers얻기

### Prime number인지 판별
- N이 Prime number가 되려면, 2보다 크거나 같고,  루트 N 보다 작거나 같은 자연수로 나누어 떨어지면 안된다.

```C++
bool prime(int n) {
	if(n < 2) {
		return false;
	}
	// 실수는 근사 값 이므로, 정수형으로 검색토록
	for(int i=2; i*i<=n; i++) {
		if(n % i == 0) {
			return false;
		}
	}
	return true;
}
```

### 숫자 N이하의 Prime numbers 얻기
위의 방법을 이 문제에 적용할 시 O(N*루트N)이라 상대적으로 느림.

#### Sieve of Eratosthenes (에라토스테네스의 체)
1. 2부터 N까지 모든 수를 준비, index = 1부터 시작
2. index 이후 아직 지워지지 않은 수 중에서 가장 작은 수 검색(소수 발견)하여 index로 사용
3. index * index > N이면 종료. 아니면 진행
4. 그 수의 배수를 모두 지움. 
5. 2번으로 돌아가 반복.

```C
for(int i=2;i<=n;i++){
	if(check[i] == false) {
		prime[pn++] = i; // prime number 저장
		// i제곱부터 그 이후 배수 검색
		// i(i+(0~)) <= n
		// overflow가 발생할 수 있어 i*2로 사용 권장
		for(int j=i*i; j<=n;j+=i) {
			check[j] = true;
		}
	}
}
```

## Goldbach's conjecture (골드바흐의 추측)
- 2보다 큰 모든 짝수는 두 소수의 합으로 표현 가능하다 -> 2+3이라면 5보다 큰 모든 홀수는 세 소수의 합으로 표현 가능으로 바뀜.
- 아직 증명은 안됬으나, 10^18 이하에서는 참인 것이 증명되어 있다.

> 출처: https://code.plus/
> 출처: https://velog.io/@yerin4847/W1-%EC%9C%A0%ED%81%B4%EB%A6%AC%EB%93%9C-%ED%98%B8%EC%A0%9C%EB%B2%95

## Factorial(팩토리얼)
 - 6! = 720
 - 8! = 40320
 - 10! = 3628800

> **소인수분해(prime factorization)**
> 1보다 큰 자연수를 **소인수**(소수인 인수)들만의 곱으로 나타내는 것이다.

### 팩토리얼 0의 개수
https://www.acmicpc.net/problem/1676
factorial 한 값에서 뒷자리에 연속되는 0의 갯수를 알아내는 것은 소인수분해를 하면 알 수 있다.
10을 만들어내는 소수의 조합은 오직 2X5 만 있다.

어쨌든 수학 문제 중에 제일 중요한건 소수이다. 소수 문제가 있으면 에라토스테네스의 체를 사용하면 된다.
위의 factorial은 그냥 참고용으로 보면 된다.

> 출처
> - https://code.plus/lecture/485
