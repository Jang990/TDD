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
* ~~**Dollar/Franc 중복**~~
* ~~공용 equals~~
* ~~공용 times~~
* ~~Franc과 Dollar 비교하기~~
* ~~통화?~~

#### 세부사항
이제 생성자 밖에 없는 `Dollar`와 `Franc`를 클래스를 제거할 수 있다. <br>
Dollar는 바로 지울 수 있는데 Franc는 앞 장에서 만든 `testDifferentClassEquality` 때문에 지울 수 없다.

다시 생각해보자. <br>
이 테스트를 지워도 될 정도로 다른 곳에서 동치성 테스트를 충분히 하고 있나?

충분하다. 아니 사실 좀 과하다. 3,4번째 assertion은 중복되니 지우는게 좋다.
```java
    @Test
    void testEquality() {
        assertTrue(Money.dollar(5).equals(Money.dollar(5)));
        assertFalse(Money.dollar(5).equals(Money.dollar(6)));
        assertTrue(Money.franc(5).equals(Money.franc(5)));
        assertFalse(Money.franc(5).equals(Money.franc(6)));
        assertFalse(Money.franc(5).equals(Money.dollar(5)));
    }
```

내 실수가 있었고 이 커밋에서 수정한다.
1. 8장에서 팩토리 메소드의 반환형을 구체 클래스가 아닌 `Money`로 바꾸지 않았다.
2. 10장에서 팩토리 메소드 리턴문을 `new Money`로 바꾸지 않았다. 


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