## 해야할 일 (매 커밋마다 업데이트)
* $5 + 10CHF = $10(환율이 2:1인 경우)
* ~~**$5 * $5 = $10**~~
* $5 + $5에서 Money 반환하기
* ~~Bank.reduce(Money)~~
* ~~**Money에 대한 통화 변환을 수행하는 Reduce**~~
* ~~**Reduce(Bank, String)**~~

#### 세부사항
환율에 대한 일은 모두 `Bank`가 처리해야 한다. <br>
`Expression.reduce`의 인자로 `Bank`를 자연스럽게 처리될 것 같다.

`Expression.reduce`에 `Bank` 인자를 추가해주고 구현체들에도 추가해주자. <br>
그리고 올바른 환율을 `Bank`에게 물어보자
```java
public class Bank {
    public Money reduce(Expression source, String to) {
        return source.reduce(this, to);
    }

    public int rate(String from, String to) {
        return (from.equals("CHF") && to.equals("USD"))
                ? 2
                : 1;
    }
}
```
테스트 코드에서도 2(`bank.addRate("CHF", "USD", 2);`)가 나오고 <br> 
지금 이 코드에서도 2가 중복되서 나온다.

`Bank`에서 환율표를 가지고 있다가 필요할 때 찾아볼 수 있게 해야 한다. <br>
두 개의 통화(프랑 -> 달러)와 환율을 매핑시키는 해시 테이블을 사용할 수 있겠다.

해시 테이블 키를 위한 `Pair` 객체를 만들어보자. <br>
키로 쓸 것이기 때문에 `equals()`와 `hashcode()`를 구현해야 한다. <br>
빠른 구현을 위해 `hashcode()`가 0만 반환하게 만든다. <br> 
이러면 리스트 검색처럼 선형 검색을 하게 되지만 지금은 문제 없다. 나중에 많은 통화를 다룰 때 개선하면 된다.

테스트를 돌려보면 빨간 막대를 확인할 수 있다.
```java
    // 빨간 테스트
    @Test
    void testReduceMoney() {
        Bank bank = new Bank();
        Money result = bank.reduce(Money.dollar(1), "USD");
        assertEquals(Money.dollar(1), result);
    }
```
동일 통화간(달러 -> 달러) 환율은 1이다. <br> 
하지만 해시테이블에 정보가 없기 때문에 NPE가 발생한다.

예상치 못한 뜻밖의 에러이기 때문에, 우리가 발견한 내용을 <br>
나중에 읽을 다른 사람들에게도 알려주기 위해 테스트를 만들어 두자.
```java
    @Test
    void testIdentityRate() {
        assertEquals(1, new Bank().rate("USD", "USD"));
    }
```

이제 에러가 두 곳에서 발생하지만, 한 곳만 바꿔 주면 두 개가 모두 없어질 것이다. <br>
`if(from.equals(to)) return 1;` - rate에 이 코드만 추가해주면 된다.


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