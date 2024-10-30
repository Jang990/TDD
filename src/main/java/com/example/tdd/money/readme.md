## 해야할 일 (매 커밋마다 업데이트)
* $5 + 10CHF = $10(환율이 2:1인 경우)
* **$5 * $5 = $10**

#### 세부사항
이 장은 설계 선택에 대한 내용이기 때문에 세부사항이 좀 길다.

TDD는 번뜩이는 통찰을 보장하지 못한다. <br>
하지만 확신을 주는 테스트와 정리된 코드를 통해 더 나은 방향에 대한 영감이 번뜩일 때 쉽게 코드에 적용할 수 있다.

다중 통화를 사용하고 있다는 사실이 시스템의 나머지 코드에 드러나지 않았으면 좋겠다. <br>
이 부분이 설계상 가장 어려운 부분이다.

편하게 여러 환율을 표현하면서 산술 연산처럼 다룰 해법이 있으면 좋겠다. <br>
객체가 우리를 구해줄 것이다. <br>
사칭 사기꾼(임포스터) 객체 : 외부 프로토콜은 같으면서도 내부 구현은 다른 새로운 객체를 말한다. <br>
`Money`와 비슷하게 동작하지만 사실 두 `Money`의 합을 나타내는 객체를 만드는 것이다. <br>

2가지 아이디어가 생각났다.

1. 한 가지는 `Money`의 합을 마치 지갑처럼 취급하는 것.
2. 다른 한 가지는 '($2 + 3CHF) * 5'와 같은 수식이다.

2번째 아이디어를 적용하면 연산의 결과로 `Expression`들이 생기고, 환율을 이용해서 `Expression`을 단일 통화로 축약할 수 있을 것 같다. <br>

2번째 아이디어를 적용한 덧셈 연산 테스트를 생각해보자.<br>
아마 테스트의 마지막줄은 다음과 같이 끝날 것 같다.
```java
assertEquals(Money.dollar(10), reduced);
```

`Expression`에 환율을 적용함으로써 얻어지는 `reduced`(축약된)를 생각했다. <br>
실세계에서 환율이 적용되는 곳은 어딘가? 은행.
```java
Money reduced = bank.reduce(sum, "USD");
assertEquals(Money.dollar(10), reduced);
```

덧셈 연산을 한 `Expression sum`에 환율을 적용한 상황을 생각했다.

<br>

우린 지금 **설계상 중요한 결정**을 내렸다. <br>
`Expression`인 `sum`을 이용해서 `sum.reduce("USD", bank)`를 할 수 있었는데, <br> 
왜 `Bank`가 `reduce()`를 수행할 책임을 맡아야 하나? <br>
쉽게 답하면 "그게 제일 먼저 떠올랐다."이다. 이건 유익하진 않다. 떠오른 이유를 풀어보자.

1. `Expression`은 우리가 하려는 일의 핵심에 해당한다. <br>
나는 핵심이 되는 객체가 다른 부분에 대해서 될 수 있는 한 모르도록 노력한다. <br>
그렇게 하면 핵심 객체가 가능한 오랫동안 유연할 수 있다. (+ 테스트 용이성, 재활용성, 이해 쉬운 코드)
2. `Expression`과 관련된 오퍼레이션이 많을 것 같다. <br>
덧셈, 뺄셈 등등 모든 오퍼레이션을 `Expression`에만 추가한다면 `Expression`은 무한히 커질 수 있다.

충분한 이유가 되지는 않겠지만, 당장 결정하기엔 부족함이 없다. <br>
또한 `Bank`가 별 필요 없게 된다면 축약을 구현할 책임을 `Expression`으로 기꺼이 옮길 생각도 있다.

<br>

다시 돌아오자. 우리는 2번째 아이디어를 적용한 덧셈 연산 테스트를 생각하고 있었다. <br>
은행을 통해 환율을 적용했기 때문에 은행 객체가 필요하다.
```java
Bank bank = new Bank();
Money reduced = bank.reduce(sum, "USD");
assertEquals(Money.dollar(10), reduced);
```

그리고 `Expression sum`이 필요하다. <br> 
우린 `Expression` 아이디어를 적용했기 때문에 두 `Money`의 합은 `Expression`이어야 한다.
```java
Expression sum = five.plus(five);
Bank bank = new Bank();
Money reduced = bank.reduce(sum, "USD");
assertEquals(Money.dollar(10), reduced);
```

5 달러를 만드는건 간단하다.
```java
Money five = Money.dollar(5);
Expression sum = five.plus(five);
Bank bank = new Bank();
Money reduced = bank.reduce(sum, "USD");
assertEquals(Money.dollar(10), reduced);
```

이제 테스트로 틀을 잡았으니 컴파일이 되게 만들면 된다.

빠르게 초록 막대를 보게 만들고 이 장은 마무리한다. 다음 장에서 진짜 구현을 할 것이다.

<br>

### 목표 : 다중 통화를 지원하는 Money객체 만들기
다음 보고서는 단일 통화(**달러**)로 표시하는 보고서이다.

| 제목   | 주   | 가격  | 합계    |
|------|-----|-----|-------|
| IBM  | 1000 | 25  | 25000 |
| GE   | 400 | 100 | 40000 |
|  |     | 합계  | 65000 |

<br>

하지만 다중 통화를 지원하는 보고서를 만들려면 통화 단위를 추가해야 한다.

| 제목   | 주   | 가격         | 합계           |
|------|-----|------------|--------------|
| IBM  | 1000 | 25**USD**  | 25000**USD** |
| GE   | 400 | 150**CHF** | 60000**CHF** |
|  |     | 합계         | 65000**USD** |

* USD - 미국 달러
* CHF - 스위스 프랑
* ...