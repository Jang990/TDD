## 해야할 일 (매 커밋마다 업데이트)
* ~~**$5 + 10CHF = $10(환율이 2:1인 경우)**~~
* ~~$5 * $5 = $10~~
* $5 + $5에서 Money 반환하기
* ~~Bank.reduce(Money)~~
* ~~Money에 대한 통화 변환을 수행하는 Reduce~~
* ~~Reduce(Bank, String)~~
* **Sum.plus**
* **Expression.times**

#### 세부사항
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
`fiveBucks`, `tenFrancs`의 자료형을 `Money`로 바꿔보자. <br>
컴파일 에러는 없어졌지만 10USD 대신 15USD(5+10)가 결과로 나와 테스트가 실패한다.

`Sum.reduce()`가 인자를 제대로 축약하지 못하고 있다.
```java
    // Sum.reduce()
    @Override
    public Money reduce(Bank bank, String to) {
        // int amount = augend.amount + addend.amount;
        int amount = augend.reduce(bank, to).amount 
                + addend.reduce(bank, to).amount;
        return new Money(amount, to);
    }
```
위와 같이 변경하면 테스트가 통과한다.

`Expression`이어야 하는 `Money`들을 없애야 한다. <br>
파급 효과를 피하기 위해 가장자리에서 작업해 올라갈 것이다. <br>

이제 `Sum`의 `augend`와 `addend`도 `Expression`으로 취급할 수 있다. <br>
따라서 `Sum`의 생성자도 `Expression`으로 받을 수 있다.

(`Sum`이 자꾸 컴포지트(Composite) 패턴을 상기시키는데 일반화시킬 만큼 강하진 않다. <br>
`Sum`이 둘 이상의 인자를 갖는 순간이 온다면 적용해도 충분하다.)

`Money.plus()`, `Money.times()`도 인자가 `Expression`으로 취급될 수 있다. <br>
이제 이 코드는 `Expression`이 `plus`와 `times` 오퍼레이션을 포함해야 함을 제안하고 있다.


이제 테스트 케이스에 나오는 `plus`안에 `Money tenFrancs`를 `Expression`으로 바꿀 수 있다.

이제 `fiveBucks`를 `Expression`으로 바꿔보자. <br>
바꾸고 나서 할 일은 컴파일러가 알려줄 것이다. <br>

컴파일러는 `Expression`에 `plus`가 없다고 말해준다. 추가해주자. <br>
그리고 `Sum.plus()`는 스텁 구현으로 만들고 할 일 목록에 적어두자. <br>
마지막으로 Expression times를 구현하면 전체 예제가 끝날 것이다.

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