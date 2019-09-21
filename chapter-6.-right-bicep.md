# Chapter 6. Right - BICEP : 무엇을 테스트할 것인가?

**Right - BICEP : 무엇을 테스트하는 것이 중요한지 도와줄 수 있는 지침**

* Right : 결과가 올바른가?
* B : 경계 조건\(boundary condition\)은 맞는가?
* I : 역 관계\(inverse relationship\)를 검사할 수 있는가?
* C : 다른 수단을 활용하여 교차 검사\(cross-check\)할 수 있는가?
* E : 오류 조건\(error condition\)을 강제로 일어나게 할 수 있는가?
* P : 성능 조건\(performance characteristics\)은 기준에 부합하는가?

## 6.1 \[Right\] - BICEP : 결과가 올바른가?

테스트 코드는 먼저 기대한 결과가 산출되는지 검증할 수 있어야 한다.

{% code-tabs %}
{% code-tabs-item title="ScoreCollection.java" %}
```java
@Test
public void answersArithmeticMeanOfTwoNumbers() {
    ScoreCollection collection = new ScoreCollection();
    collection.add(() -> 5);
    collection.add(() -> 7);

    int actualResult = collection.arithmeticMean();

    assertThat(actualResult, equalTo(6));
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

해당 예시코드에서 더 많은 수들이나 혹은 큰 수들을 넣어 테스트를 해볼 수 있지만 Right에서 말하고자 하는 것은 내가 이 코드의 시나리오에 대해 정확히 이해하고 있는가이다. 그러니 예상되는 결과 값을 개발자 자신이 이해하고 있어야한다는 것이고 예상하지 못한다면 잠시 개발을 보류하는 것도 좋다.

하지만 모든 요구사항이 명확하지 않고 중간에 변경되는 경우도 많기 때문에 개발자가 자신의 판단으로 최선의 판단을 해야한다.

## 6.2 Right - \[B\]ICEP : 경계 조건은 맞는가?

우리가 개발을 하면서 마주하는 수많은 결함들은 모서리 사례\(coner case\)라고 표현하고 있다. coner case와 함께 자주 나오는 용어인 edge case가 있다. 이 둘이 무엇인지 잠깐 짚고 넘어가자.

엣지케이스\(edge case\)와 코너 케이스\(coner case\)는 프로그래밍, 디버깅, 단위 테스트 등에서 사용되는 비슷하지만 다른 용어다.

### Edge Case

엣지 케이스란 알고리즘이 처리하는 데이터의 값이 알고리즘의 특성에 따른 일정한 범위를 넘을 경우에 발생하는 문제를 가리킨다.

예를 들면 fixnum이라는 변수의 값이 -128 ~ 127의 범위를 넘는 순간 문제가 발생하는 경우가 있을 수 있다. 어떤 분모가 0이 되는 상황처럼 데이터의 특정값에 대해 문제가 발생하는 경우도 마찬가지다.

엣지 케이스는 알고리즘의 특성에 따라 개발자가 면밀히 검토하여 예상할 수 있는 문제다. 이런 문제는 디버그가 쉽기도 하고 테스트를 통해 미리 방지하기도 쉽다.

### Coner Case

코너 케이스는 여러 가지 변수와 환경의 복합적인 상호작용으로 발생하는 문제다.

예를 들어 fixnum이라는 변수의 값으로 128이 입력되었을 때, A 기계에서 테스트했을 때는 정상작동하지만 B 기계에서는 오류가 발생한다면 코너 케이스라고 할 수 있다. 같은 장치에서라도 시간이나 다른 환경에 따라 오류가 발생하기도 하고 정상작동 하기도 한다면 이것도 코너 케이스다.

특히 멀티코어 프로그래밍에서 만나기 쉬운 오류일 것이다.

코너 케이스는 오류가 발생하는 상황을 재현하기가 쉽지 않아 디버그와 테스트가 어렵다.

코너케이스와 엣지케이스가 무엇인지 정리를 했으니 우리가 생각해야하는 경계 조건들을 살펴보자

* 모호하고 일관성 없는 입력 값. 예를 들어 특수 문자가 포함된 파일이름
* 잘못된 양식의 데이터. 예를 들어 최상위 도메인이 빠진 이메일 주소\(fred@foobar\)
* 수치적 오버플로를 일으키는 계산
* 비거나 빠진 값. 예를 들어 0, 0.0, " " 혹은 null
* 이상적인 기댓값을 훨씬 벗어나는 값. 예를 들어 150세의 나이
* 교실의 당번표처럼 중복을 허용해서는 안 되는 목록에 중복 값이 있는 경우
* 정렬이 안 된 정렬 리스트 혹은 그 반대. 정렬 알고리즘에 이미 정렬된 입력 값을 넣는 경우나 정렬 알고리즘에 역순 데이터를 넣는 경우
* 시간 순이 맞지 않는 경우. 예를 들어 HTTP 서버가 OPINIONS 메서드의 결과를 POST 메서드보다 먼저 반환해야 하지만 그 후에 반환하는 경우

{% code-tabs %}
{% code-tabs-item title="ScoreCollection.java" %}
```java
import java.util.*;

public class ScoreCollection {
    private List<Scoreable> scores = new ArrayList<>();

    public void add(Scoreable scoreable) {
        scores.add(scoreable);
    }

    public int arithmeticMean() {
        int total = scores.stream().mapToInt(scorealbe::getScore).sum();
        return total / scores.size();
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

위 코드는 1장에서 나왔던 코드이다. 그다지 문제는 없어보이지만 경계 조건을 보면서 살펴보자

#### Problem 1. 입력된 Scoreable 인스턴스가 null일 경우

{% code-tabs %}
{% code-tabs-item title="ScoreCollectionTest.java" %}
```java
@Test(expected=IllegalArgumentException.class)
public void throwsExceptionWhenAddingNull() {
    collection.add(null);
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

arithmeticMean\(\) 메서드에서는 당연히 NullPointerException이 발생하게 된다. 따라서 테스트 코드에서 먼저 걸러내야 하고 클라이언트가 유효하지 않은 값을 넣자마자 오류가 발생하도록 하는 것이 좋다.

#### Problem 1. 해결 방법

{% code-tabs %}
{% code-tabs-item title="ScoreCollection.java" %}
```java
public void add(Scoreable scoreable) {
    if(scoreable == null) throw new IllegalArgumentException{
        scores.add(scoreable);
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### Problem 2. 0으로 나누어 나는 오류

{% code-tabs %}
{% code-tabs-item title="ScoreCollectionTest.java" %}
```text

```
{% endcode-tabs-item %}

{% code-tabs-item title=undefined %}
```java
@Test
public void ansersZeroWhenNoElementsAdded() {
    assertThat(collection.arithmeticMean(), equalTo(0));
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### Problem 2. 해결 방법

{% code-tabs %}
{% code-tabs-item title="ScoreCollection.java" %}
```java
public int arithmeticMean() {
    if(scores.size() == 0) return 0;
    // ...
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### Problem 3.  큰 정수 입력을 다루어 숫자들의 합이 Integer.MAX\_VALUE를 초과하는 경우

{% code-tabs %}
{% code-tabs-item title="ScoreCollectionTest.java" %}
```java
@Test
public void dealsWithIntegerOverflow() {
    collection.add(() -> Integer.MAX_VALUE);
    collection.add(() -> 1);

    assertThat(collection.arithmeticMean, equalTo(1073741824))
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Integer.MAX\_VALUE 는 2147483647로 위에 1073741824보다 크다. 1을 더했는데 오히려 값이 줄어든 것을 확인할 수 있다.

#### Problem 3. 해결 방법

{% code-tabs %}
{% code-tabs-item title="ScoreCollection.java" %}
```java
long total = score.stream().mapToLong(Scoreable::getScore).sum();
return (int)(total / scores.size());
```
{% endcode-tabs-item %}
{% endcode-tabs %}

long타입에서 int타입으로 다운 캐스팅이 이루어졌다. 무언가 추가적인 검사를 해야 할 것처럼 보이지만 그렇지 않다. add\(\) 메서드에서는 개별 입력 값을 int 타입으로 한정하고 개수만큼 나누게 되면 int 최댓값보다 작은 값만 반환할 수 밖에 없다.

클래스를 설계할 때 이러한 잠재적인 정수 오버플로 등을 고려할지 여부는 전적으로 개발자 몫이다. 클래스가 외부에서 호출하는 API이고 클라이언트를 완전히 믿을 수 없다면 위와 같은 보호가 필요하지만 클라이언트가 같은 팀 소속이라면 보호절을 제거하고 클라이언트 측에게 이러한 사실을 알려도 된다. 불필요한 검사절을 줄여주기 때문이다.

보호절을 제거하는 선택을 했다면 주석을 통해 경고를 남겨도 괜찮지만 더 좋은 방법은 코드 제한 사항을 문서화하는 테스트를 추가하는 이다.

## 6.3 경계 조건에서는 CORRECT를 기억하라

CORRECT 약어는 잠재적인 경계 조건을 기억하는 데 도움이 된다. 각 항목에 대해 유사한 조건을 테스트하려는 메서드에 해당되며, 해당 조건들을 위반했을 때 어떤 일이 일어날지 고려해보면 된다.

* \[C\]onformance \(준수\) : 값이 기대한 양식을 준수하고 있는가?
* \[O\]rdering \(순서\) : 값의 집합이 적절하게 정렬되거나 정렬되지 않았나?
* \[R\]ange \(범위\) : 이성적인 최솟값과 최댓값 안에 있는가?
* \[R\]eference \(참조\) : 코드 자체에서 통제할 수 없는 어떤 외부 참조를 포함 하고 있는가?
* \[E\]xistence \(존재\) : 값이 존재하는가? \(null이 아니거나\(non-null\), 0이 아니거나, 집합에 존재하는가 등\)
* \[C\]ardinality \(기수\) : 정확히 충분한 값들이 있는가?
* \[T\]ime \(절대적 혹은 상대적 시간\) : 모든 것이 순서대로 일어나는가? 정확한 시간에? 정시에?

이 경계 조건에 대한 내용은 다음 7장에서 조금 더 자세하게 다루고 일단은 넘어가자.

## 6.4 Right - B\[I\]CEP : 역 관계를 검사할 수 있는가?

때때로 논리적인 역 관계를 적용하여 검사할 수 있다. 무슨말이냐 하면 우리가 수학에서 곱셈으로 나눗셈을 검증하고 뺄셈으로 덧셈을 검증하는 것과 같은 말이다.

우리는 뉴턴의 알고리즘을 활용해서 제곱근을 구한다. 어떤 숫자의 제곱근을 유도하고 그 결과를 제곱하면\(즉, 자기 자신을 곱하면\) 우리가 시작했던 값과 같은 숫자를 얻어야 함을 기억하고 다음의 코드를 보자.

{% code-tabs %}
{% code-tabs-item title="NewtonTest.java" %}
```java
import org.junit.*;
import static org.junit.Assert.*;
import static org.hamcrest.number.IsCloseTo.*;
import static java.lang.MAth.abs;

public class NewtonTest {
    static class Newton {
        private static final double TOLERANCE = 1E-16;

        public static double squareRoot(double n) {
            double approx = n;
            while(abs(approx - n / approx) > TOLERANCE * approx) {
                approx = (n / approx + approx) / 2.0;
            }
            return approx;
        }
    }

    @Test
    public void squareRoot() {
        double result = Newton.squareRoot(250.0);
        assertThat(result * result, closeTo(250.0, Newton.TOLERANCE));
        //clolseTo -> 부동소수점을 비교하기 위한 메서드
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 6.5 Right - BI\[C\]EP : 다른 수단을 활용하여 교차 검사할 수 있는가?

우리는 Math.sqrt\(\) 라는 좋은 메서드를 두고 위에서 제곱근을 구하는 함수를 만들었다. 이러한 로직이 동일한 결과 값을 내는지 확인하기 위해 Math.sqrt\(\) 메서드를 통해 확인할 수 있다.

{% code-tabs %}
{% code-tabs-item title="NewtonTest.java" %}
```java
assertThat(Newton.squareRoot(1969.0), closeTo(Math.sqrt(1969.0), Newton.TOLERANCE));
```
{% endcode-tabs-item %}
{% endcode-tabs %}

또 다른 예로는 도서 대출 시스템을 생각해 볼 수 있다. 도서관에 대출된 도서와 대출되지 않은 도서의 도서 개수를 합하면 도서의 총 수량과 같아야하는 점을 이용해 서로 교차 검사할 수 있다.

결국 클래스의 서로 다른 조각 이터를 사용해서 모든 데이터가 합산되는지 확인해 보는 방법이다.

## 6.6 Right - BIC\[E\]P : 오류 조건을 강제로 일어나게 할 수 있는가?

위처럼 유효하지 않은 인자들을 다루는 것은 쉽다고 볼 수 있지만 또 다른 문제들이 있다. 예를 들어 디스크가 꽉 차거나, 네트워크 선이 떨어지거나 하는 문제들말이다. 이러한 특정 네트워크 오류를 시뮬레이션 하려면 특별한 기법이 필요한데 그에 대한 한 가지 방법을 10.1 장에서 다뤄볼 예정이다.

하지만 먼저 우리가 고려해야할 환경적인 제약 사항들이 어떤 것들이 있는지 보자.

* 메모리가 가득 찰 때
* 디스크 공간이 가득 찰 때
* 벽시계 시간에 관한 문제들\(서버와 클라이언트 간 시간이 달라서 발생하는 문제\)
* 네트워크 가용성 및 오류들
* 시스템 로드
* 제한된 색상 팔레트
* 매우 높거나 낮은 비디오 해상도

## 6.7 Right - BICE\[P\] : 성능 조건은 기준에 부합하는가?

정말 많은 프로그래머들이 성능 문제가 어디에 있는지 최적의 해법이 무엇인지 추측을 한다. 하지만 문제는 이러한 추측이 때때로 잘못되었다는 것이다. 성능 문제를 추측으로 대응하기보다는 단위 테스트를 설계해서 진짜 문제가 어디에 있으며 예상한 변경 사항으로 어떤 차이가 생겼는지 파악해야 한다.

{% code-tabs %}
{% code-tabs-item title="ProfileTest.java" %}
```java
@Test
public void findAnswers() {
      int dataSize = 5000;
      for (int i = 0; i < dataSize; i++)
         profile.add(new Answer(
               new BooleanQuestion(i, String.valueOf(i)), Bool.FALSE));
      profile.add(
         new Answer(
           new PercentileQuestion(
                 dataSize, String.valueOf(dataSize), new String[] {}), 0));

      int numberOfTimes = 1000;
      long elapsedMs = run(numberOfTimes,
         () -> profile.find(
               a -> a.getQuestion().getClass() == PercentileQuestion.class));

      assertTrue(elapsedMs < 1000);
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="ProfileTest.java" %}
```java
private long run(int times, Runnable func) {
   long start = System.nanoTime();
   for (int i = 0; i < times; i++)
      func.run();
   long stop = System.nanoTime();
   return (stop - start) / 1000000;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

앞의 코드는 그 코드가 특정 시간 안에 실행되는지를 테스트하기 위해 뒤에 run\(\) 메서드를 이용했다.

성능 테스트를 진행할 때는 몇 가지 주의 사항이 있다.

* 충분한 횟수만큼 실행해야 한다.
* 반복하는 코드 부분을 자바\(JVM\)가 최적화하지 못하는지 확인해야 한다.
* 최적화되지 않은 테스트는 일반적인 테스트 코드보다 매우 느리기 때문에 분리해서 해봐야한다. 성능이 떨어지는 테스트 코드를 추가하여 전체 테스트 시간이 늘어나는 것은 좋지 못한 방법이다.
* 동일한 머신이라도 실행 시간은 시스템 로드처럼 잡다한 요소에 따라 달라질 수 있다.

물론 단위 성능 측적은 변경하고 시간을 측정하고 하는 반복적인 테스트들을 통해 비교하고 개선해 나가는 방법이며 성능이 핵심 고려 사항이라면 당연히 성능 측정 도구를 사용하는 것도 좋은 방법이다.

