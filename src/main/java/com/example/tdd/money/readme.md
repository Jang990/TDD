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
* **공용 times**
* ~~Franc과 Dollar 비교하기~~
* ~~통화?~~

#### 세부사항
지금 `Dollar`와 `Franc`의 `times()` 메소드를 동일하게 만들기 위한 확실한 방법이 없다. <br>
팩토리 메소드를 인라인 시키면 어떨까? <br>
바로 전 장에서 팩토리 메소드를 호출하도록 바꿨는데 실망스러운 일이다. <br>
하지만 때로는 전진하기 위해 물러서야 할 때도 있는 법이다.

```java
    // Dollar 코드
    public Money times(int multiplier) {
        // return Money.dollar(amount * multiplier);
        return new Dollar(amount * multiplier, currency);
    }

    // Franc 코드
    public Money times(int multiplier) {
        // return Money.franc(amount * multiplier);
        return new Franc(amount * multiplier, currency);
    }
```

`Money`의 구현체가 `Franc`인지 `Dollar`인지 정말로 중요한 정보인가? <br>
우리가 만든 `Money` 시스템의 정보를 기반으로 조심스럽게 생각해봐야 할 문제다.

하지만 우린 깔끔한 코드와 그 코드가 잘 작동할 거라는 믿음을 줄 수 있는 테스트 코드들이 있다. <br>
몇 분 동안 고민하는 대신 그냥 수정하고 테스트를 돌려서 컴퓨터에게 직접 물어보자.

컴퓨터라면 10~15초에 대답할 문제를 최고의 소프트웨어 엔지니어들은 5~10분 동안 고민하곤 한다.

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