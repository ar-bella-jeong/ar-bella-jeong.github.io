---
title: "[Software] Strangler fig Pattern"
date: 2022-01-21 20:45:28 -0400
categories: software
tags:
  - msa
comments: true
---
Monolithic architecture로 된 system이 계속 자라나면서 architecture가 복잡해지고 business logic이 파편화 되면 code를 관리하기 어려워진다. 그러다보면 배포가 까다롭고 무서워진다.

이를 한번에 고치는것은 risk가 있으므로 점진적으로 code를 분리해나가는 작업이 필요하다.

FE의 Strangler Pattern은 page를 새 application으로 점진적으로 migration하는 것으로 구성되며 request를 legacy 및 새 application으로 routing하려면 facade가 필요하다. 이 접근 방식을 따르면 legacy 기능을 migration 하는 동안 새 application에서 새로운 기능을 개발할 수도 있다.
![스트래처 Fig 패턴 다이어그램](https://docs.microsoft.com/ko-kr/azure/architecture/patterns/_images/strangler.png)
## 어원
Martin Fowler가 아내인 Cindy Folwer와 함께 호주에 갔을 때 본  Strangler figs라는 이름의 식물에서 기원한다.
[Martin Fowler의 StranglerFigApplication](https://martinfowler.com/bliki/StranglerFigApplication.html)

reverse proxy로서의 Nginx는 request를 다른 application으로 routing하는데 유용하다.

api gateway로 service통합하는 방법도 있다.

그 외 내부 service 끼리의 호출이 발생할 경우 event성 비동기 호출을 지향하도록 지침한다.


> 출처
> - https://medium.com/@felipegaiacharly/strangler-pattern-for-frontend-865e9a5f700f
> - https://docs.microsoft.com/ko-kr/azure/architecture/patterns/strangler-fig
> - https://johngrib.github.io/wiki/pattern/strangler/
