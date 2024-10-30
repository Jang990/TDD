## 해야할 일 (매 커밋마다 업데이트)
* $5 + 10CHF = $10(환율이 2:1인 경우)
* $5 * $5 = $10
* $5 + $5에서 Money 반환하기
* ~~Bank.reduce(Money)~~
* **Money에 대한 통화 변환을 수행하는 Reduce**
* Reduce(Bank, String)

#### 세부사항
2프랑이 있는데 이걸 달러로 바꾸고 싶다.
이렇게 써놓고 보니 이미 테스트 케이스가 만들어진 것 같다.
```java
    @Test
    void testReduceMoneyDifferentCurrency() {
        Bank bank = new Bank();
        bank.addRate("CHF", "USD", 2);
        Money result = bank.reduce(Money.franc(2), "USD");
        assertEquals(Money.dollar(1), result);
    }
```
`bank.addRate()`가 테스트 코드에 나온 이유는 12장에서 다음과 같은 설계를 선택했기 때문이다. <br>
`12장 내용 중. 실세계에서 환율이 적용되는 곳은 어디인가? 은행`

사실 이 테스트 코드는 한 줌의 지저분한 코드면 초록 막대를 볼 수 있다. <br>
이 코드로 인해 갑자기 `Money`가 환율에 대해 알게 돼 버렸다. 우웩.
```java
    //Money 클래스의 메소드
    public Money reduce(String to) {
        int rate = (currency.equals("CHF") && to. equals("USD")) 
            ? 2
            : 1;
        return new Money(amount /rate, to);
    }
```


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