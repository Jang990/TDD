## 해야할 일 (매 커밋마다 업데이트)
* $5 + 10CHF = $10(환율이 2:1인 경우)
* $5 * $5 = $10
* **$5 + $5에서 Money 반환하기**
* ~~**Bank.reduce(Money)**~~
* **Money에 대한 통화 변환을 수행하는 Reduce**
* **Reduce(Bank, String)**

#### 세부사항
이제 `Bank.reduce()`는 `Sum`을 전달받는다. <br>
이제 `Bank.reduce()`의 가짜 구현을 걷어내보자. <br>
가짜 구현으로 인해 테스트가 깨지도록 인자를 선택했다.
```java
    @Test
    void testReduceSum() {
        Expression sum = new Sum(Money.dollar(3), Money.dollar(4));
        Bank bank = new Bank();
        Money result = bank.reduce(sum, "USD");
        assertEquals(Money.dollar(7), result);
    }    
```

우리가 Sum을 계산하면 결과는 Money가 돼야하고, Money의 양은 두 Money의 합이여야 하고, 
통화는 우리가 Bank.reduce()에 요청한 통화여야 한다.
```java
public class Bank {
    public Money reduce(Expression source, String to) {
        Sum sum = (Sum) source;
        int amount = sum.augend.amount + sum.addend.amount;
        return new Money(amount, to);
    }
}
```

테스트가 통과한다. 하지만 이 코드는 두 가지 이유로 지저분하다.

1. 캐스팅(형변환) 문제 : 이 코드는 모든 Expression에 대해 작동해야 한다. 
2. 공용 필드 문제 : 공용 필드와 그 필드에 대한 두 단계에 걸친 참조

간단히 고칠 수 있는 문제들이다. <br>
공용 필드 문제는 로직을 `Sum` 클래스로 옮기면 끝이다.  
```java
    // Bank 메소드
    public Money reduce(Expression source, String to) {
        Sum sum = (Sum) source;
        return sum.reduce(to);
    }

    // Sum 메소드
    public Money reduce(String to) {
        int amount = augend.amount + addend.amount;
        return new Money(amount, to);
    }
```
이걸 해결하고 보니 문득 Money를 `Bank.reduce()`에 넘기면 어떻게 처리해야 할지 고민이 든다. <br>
예를 들어 Dollar를 Franc으로 바꾸고 싶을 때... 지금 구현은 이 문제를 해결하지 못한다.

할 일 목록에 `Bank.reduce(Money)`를 추가한다. <br> 
지금은 초록 막대 상태이고, 현재 위 코드에서 뭘 더해야 할지 명확하지 않으니 <br>
이것에 대한 테스트 코드를 작성해보자.
```java
    @Test
    void testReduceMoney() {
        Bank bank = new Bank();
        Money result = bank.reduce(Money.dollar(1), "USD");
        assertEquals(Money.dollar(1), result);
    }
```

클래스를 명시적으로 검사하는 코드들이 추가돼야 하면 항상 다형성을 사용하도록 바꾸는 것이 좋다. <br>
Expression에 reduce를 추가해서 다형성을 지원하도록 만들어보자. <br>
일단 Money는 자신을 반환하는 정도로만 구현하자.


이렇게 코드를 변경하면서 `Expression.reduce`에 환율을 계산하는 `Bank`가 매개변수로 필요하다는 것을 확신하게 됐다. <br>
이것도 할 일 목록에 추가한다.

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