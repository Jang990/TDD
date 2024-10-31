## 해야할 일 (매 커밋마다 업데이트)
* ~~$5 + 10CHF = $10(환율이 2:1인 경우)~~
* ~~$5 * $5 = $10~~
* ~~**$5 + $5에서 Money 반환하기**~~
* ~~Bank.reduce(Money)~~
* ~~Money에 대한 통화 변환을 수행하는 Reduce~~
* ~~Reduce(Bank, String)~~
* ~~**Sum.plus**~~
* ~~**Expression.times**~~

#### 세부사항
TDD로 구현할 땐 테스트 코드의 줄 수와 모델 코드의 줄 수가 비슷한 상태로 끝난다. <br>
그래서 TDD가 경제적이기 위해선 동일한 기능을 구현하되 절반의 코드로 해거나, 만들어 내는 코드가 2배가 되야 할 것이다. <br>
물론 디버깅, 통합작업, 팀원에게 코드 설명 시간 등등의 요소를 포함해야 한다는 것도 기억하자.

`Expession.plus`를 끝마치려면 `Sum.plus()`를 구현해야 한다. <br>
그리고 나서 `Expression.times()`를 구현하면 전체 예제가 끝난다.

다음은 `Sum.plus()`에 대한 테스트다.
```java
    @Test
    void testSumPlusMoney() {
        Expression fiveBucks = Money.dollar(5);
        Expression tenFrancs = Money.franc(10);
        Bank bank = new Bank();
        bank.addRate("CHF", "USD", 2);
        Expression sum = new Sum(fiveBucks, tenFrancs).plus(fiveBucks);
        Money result = bank.reduce(sum, "USD");
        assertEquals(Money.dollar(15), result);
    }
```
`Expression sum = new Sum(fiveBucks, tenFrancs).plus(fiveBucks);`와 <br>
`Expression sum = fiveBucks.plus(tenFrancs).plus(fiveBucks);`는 같은 코드이다. <br>
하지만 다시 말하지만 이것은 `Sum.plus()`에 대한 테스트이다. <br>
위 방식이 우리 의도를 더 직접적으로 드러낸다.

테스트가 통과하도록 `Sum.plus`를 구현하자.
```java
    @Override
    public Expression plus(Expression addend) {
        return new Sum(this, addend);
    }
```
테스트가 코드보다 더 길다. (29장 픽쳐스로 줄일 수 있다.)<br> 
또 코드는 `Money`의 코드와 똑같다. (멀리서 추상 클래스의 외침이 들려온다.)<br>
리팩터링은 나중으로 남기고 쭉쭉 나아가 보자.

일단 `Sum.times()`가 작동하게 만들 수만 있다면 `Expression.times()`를 선언하는 일은 어렵지 않다.
```java
    @Test
    void testSumTimes() {
        Expression fiveBucks = Money.dollar(5);
        Expression tenFrancs = Money.franc(10);
        Bank bank = new Bank();
        bank.addRate("CHF", "USD", 2);
        Expression sum = new Sum(fiveBucks, tenFrancs).times(2);
        Money result = bank.reduce(sum, "USD");
        assertEquals(Money.dollar(20), result);
    }
```
(이번에도 테스트 코드가 길다.) <br>
`Sum.times`가 없기 때문에 컴파일 에러를 일으킨다. <br> 
`Sum.times`를 구현하고 `Expression`에 `times`를 선언해주자.
```java
    // Sum.times
    public Expression times(int multiplier) {
        return new Sum(augend.times(multiplier), 
                addend.times(multiplier));
    }
```

우리의 목표였던 `$5 + 10CHF = $10`를 만드는데 성공했다. <br>
추가적으로 우린 13장에서 우리는 같은 통화(달러)를 갖는 경우엔 그냥 `Money`(달러)로 반환해도 좋겠다고 생각했다. <br>
(`$5 + $5에서 Money 반환하기` 마지막 할 일 지워보기)

이렇게 깔끔하게 정리했으면 좋겠다는 생각이 들면 다음과 같이 실험 테스트 코드를 만들어본다.
```java
    @Test
    void testPlusSameCurrencyReturnsMoney() {
        Expression sum = Money.dollar(1).plus(Money.dollar(1));
        assertTrue(sum instanceof Money);
    }
```
이 테스트는 외부에 드러나는 객체의 행위에 더하기에 집중하지 않고, <br>
내부적으로 `Money` 인스턴스를 반환해주는지 확인하는 구현 중심 테스트라 지저분하다.

지금까지 우리가 만들어온 시스템에서 이 테스트를 통과시킬 분명하고도 깔끔한 방법이 없다. <br>
이 실험은 실패했다. 추가 기능은 만지지 말고 이 실험 테스트 코드를 삭제하고 떠나자.

<br>

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