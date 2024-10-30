## 해야할 일 (매 커밋마다 업데이트)
* **$5 + 10CHF = $10(환율이 2:1인 경우)**
* ~~$5 * $5 = $10~~
* $5 + $5에서 Money 반환하기
* ~~Bank.reduce(Money)~~
* ~~Money에 대한 통화 변환을 수행하는 Reduce~~
* ~~Reduce(Bank, String)~~

#### 세부사항
드디어 이 모든 작업의 시초인 $5 + 10CHF에 대한 테스트를 추가할 준비가 됐다. <br>
이게 우리가 원하는 코드다.
```java
    @Test
    void testMixedAddition() {
        Expression fiveBucks = Money.dollar(5);
        Expression tenFrancs = Money.franc(10);
        Bank bank = new Bank();
        bank.addRate("CHF", "USD", 2);
        Money result = bank.reduce(fiveBucks.plus(tenFrancs), "USD");
        assertEquals(Money.dollar(10), result);
    }
```

컴파일 에러가 무지 많다. <br>
`Money`에서 `Expression`으로 일반화할 때 우리가 어설프게 남겨둔 것들이 꽤 된다. <br>
독자를 어지럽히고 싶지 않지만 이젠느 독자를 어지럽힐 때가 됐다.

처음 코드 수정이 다음으로 계속해서 퍼져나갈 수 있게 할 것이다.

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