## 해야할 일 (매 커밋마다 업데이트)
* $5 + 10CHF = $10(환율이 2:1인 경우)
* ~~$5 * 2 = $10~~
* ~~amount를 private으로 만들기~~
* ~~Dollar 사이드 이펙트?~~ 
* Money 반올림?
* ~~equals()~~
* hashcode()
* Equal null
* Equal object
* ~~5CHF * 2 = 10CHF~~
* Dollar/Franc 중복
* ~~공용 equals~~
* ~~**공용 times**~~
* ~~Franc과 Dollar 비교하기~~
* ~~통화?~~

#### 세부사항
`Money`의 구현체가 `Franc`인지 `Dollar`인지 정말로 중요한 정보인가? <br>
`Franc` 코드에서 `times()`가 `Money`를 직접 반환하게 만들어보자.

```java
    // Franc 코드
    public Money times(int multiplier) {
        // return Money.franc(amount * multiplier);
        return new Money(amount * multiplier, currency);
    }
```
컴파일러는 `Money`를 콘크리트 클래스로 바꿔야 한다고 말한다.

빨간 막대인 상황에서는 테스트를 추가로 작성하지 않는다. <br>
보수적인 방법을 따르자면 변경된 코드를 되돌려서 다시 초록 막대 상태로 돌아간다.<br>
(때때로 빨간 막대 상태에서도 테스트를 새로 하나 작성하기도 한다.)

보수적으로 진행해보자. 다시 `Franc.times()`가 `Franc`를 생성해서 반환하게 한다.

우리는 `Franc(10, 'CHF')`가 `Money(10, 'CHF)`와 서로 같기를 바란다. <br>
이걸 그대로 테스트로 사용하는 것이다.
```java
    @Test
    void testDifferentClassEquality() {
        assertEquals(new Money(10, "CHF"),
                new Franc(10, "CHF")
        );
    }
```
(책에서는 여기부터 보폭을 넓혀 너무 상세한 설명은 생략되지만, 맥락으로 충분히 이해할 수 있을 것이다.) <br>
예상대로 해당 테스트는 실패한다. 

`equals()`에서 클래스가 아니라 `currency`를 비교해야 하는 것이다.

이제 `times()`를 `Money` 클래스로 끌어 올릴 수 있다.

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