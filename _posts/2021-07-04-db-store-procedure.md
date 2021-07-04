# [DB] Stored Procedure?
DB 내부에 저장된 일련의 SQL 명령문들을 하나의 함수처럼 실행하기 위한 쿼리의 집합  
  
## SP의 장점
### 개발 측면
- 반복적인 작업을 피할 수 있다
	- 같은 query 여러 줄을 쓰지 않아도 된다.
- 개발 언어에 비 의존적이다.
	- application의 개발 언어가 바뀌었을 때, 필요한 stored procedure를 가져올 수 있는지만 확인하면 된다.
- 확장 및 유지 보수가 간편해 진다.
	- DB 관련 작업을 수정할 때 stored procedure만 건드리면 되기 때문이다.
### 성능 측면
- SP는 최적화 되고 cache 된다.
![](https://t1.daumcdn.net/cfile/tistory/24400142554F2D9F03)
- network traffic을 감소 시킨다.
	- SP 사용 시 SQL문이 server에 저장되므로 query문 자체를 전달하지 않아도 된다. client들은 매개변수만 전달한다.
### 보안 측면
- SP에서 참조하고 있는 table로 외부에서 접근하는 것을 제한할 수 있다.
## SP의 단점
- 처리 성능이 낮다.
	- 문자나 숫자 연산에 Stored procedure를 사용하면 C나 Java보다 느린 성능을 보여준다.
- 디버깅이 어렵다
- DB 확장이 매우 힘들다.
	- service 사용자가 많아져 확장이 필요할 시 DB 수를 늘리는 것 보다 WAS의 수를 늘리는 것이 더 효율적이기 때문에 대부분의 개발에서는 DB에는 최소의 부담만 주고 대부분의 logic은 WAS에서 처리할 수 있게 한다.

## MSSQL에서 SP 사용하기
![](https://t1.daumcdn.net/cfile/tistory/215F9E40554F32360A)
SP는 CREATE PROCEDURE 쿼리문을 통해 만들 수 있다.
@가 붙어 있는 name은 매개 변수이다.
  SET NOCOUNT OFF 일 경우 반환하는 값은 SP 내부에서 SELECT, UPDATE 등을 사용한 횟수이다. 따라서 원하는 값을 반환 받기 위해서는 SET NOCOUNT ON으로 설정해주어야 한다.

> 출처: [https://itability.tistory.com/51](https://itability.tistory.com/51) [aBiLiTy BLoG]

> 출처: https://runcoding.tistory.com/31
