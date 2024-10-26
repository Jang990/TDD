## 해야할 일 (매 커밋마다 업데이트)
* $5 + 10CHF = $10(환율이 2:1인 경우)
* ~~$5 * 2 = $10~~
* **amount를 private으로 만들기**
* ~~Dollar 사이드 이펙트?~~ 
* Money 반올림?
* ~~equals()~~
* hashcode()
* Equal null
* Equal object

#### 세부사항
`Dollar.times()`연산은 |호출을 받은 객체의 값 * 호출 파라미터|를 값으로 갖는 Dollar 반환

```java
    @Test
    void testMultiplication() {
        Dollar five = new Dollar(5);
        Dollar product = five.times(2);
        assertEquals(10, product.amount);
        product = five.times(3);
        assertEquals(15, product.amount);
    }
```
하지만 현재 테스트는 정확히 그것을 말하고 있지 않다.

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