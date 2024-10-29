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
* **Dollar/Franc 중복**
* ~~공용 equals~~
* 공용 times
* ~~Franc과 Dollar 비교하기~~
* 통화?

#### 세부사항
하위 클래스에 대한 직접적인 참조가 적어진다면? 하위 클래스를 제거하기 수월하다. <br>
테스트에서 하위 클래스의 참조가 사라지게 만들어보자.

Money에서 `Dollar`를 반환하는 팩토리 메소드를 만들어보자. <br>
이제 테스트 코드에서 `Dollar` 참조를 모두 제거하고 테스트 코드를 돌려보자. <br>
컴파일 에러가 발생한다. 생각해보니 `times()`는 Money에 선언돼있지 않다.<br>

`Money`를 추상클래스로 만들고 `times`를 추상 메소드로 만들었다.<br>
이제 테스트 코드가 동작하고, `new Dollar`나 `Dollar dollar`등의 참조를 테스트 코드에서 찾을 수 없다.

`Franc`도 이와 같이 테스트 코드에서 참조를 전부 제거해준다.<br>
이제 테스트 코드를 잘 살펴보면 두 테스트는 똑같은 `Money.times()`를 테스트하고 있다. <br>
```java
    @Test
    void testMultiplication() {
        Money five = Money.dollar(5);
        assertEquals(Money.dollar(10), five.times(2));
        assertEquals(Money.dollar(15), five.times(3));
    }

    @Test
    void testFrancMultiplication() {
        Money five = Money.franc(5);
        assertEquals(Money.franc(10), five.times(2));
        assertEquals(Money.franc(15), five.times(3));
    }
```

`testFrancMultiplication`를 제거하면 전체 코드에 대한 확신이 조금이라도 줄어드나? <br>
그럴 가능성이 조금이나마 있기 때문에 일단 남겨두자.

Dollar/Franc 중복을 완전히 없앨 순 없었지만, <br>
어쨋든 우린 하위 클래스의 존재를 테스트에서 분리했기 때문에 다른 코드에 영향을 주지 않고 상속 구조를 마음대로 바꿀 수 있어졌다. <br>
다음 장들에서 천천히 지워나가 보자.

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