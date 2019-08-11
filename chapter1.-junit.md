# Chapter 1. 첫 번째 JUnit 테스트 만들기

## 1.1 단위 테스트를 작성하는 이유

책에서는 팻과 데일의 이야기를 통해 테스트 코드를 작성해야 하는 설명한다.

결국 하고자 하는 말은 개발할 때 작은 단위로 테스트코드를 작성해 놓는다면 이후에 문제가 생겼을 때 그러한 문제가 어디서 일어났는지 빠르게 찾을 수 있으며 코드의 이해도도 높아진다는 말이다.



## 1.2 JUnit의 기본 : 첫 번째 테스트 통과

다음의 코드들을 테스트 해보자.

{% code-tabs %}
{% code-tabs-item title="Scoreable.java" %}
```java
@FunctionalInterface
public interface Scoreable {
   int getScore();
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="ScoreCollection.java" %}
```java
public class ScoreCollection {
   private List<Scoreable> scores = new ArrayList<>();
   
   public void add(Scoreable scoreable) {
      scores.add(scoreable);
   }
   
   public int arithmeticMean() {
      int total = scores.stream().mapToInt(Scoreable::getScore).sum();
      return total / scores.size();
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

ScoreCollection.java 파일에서 마우스 우클릭 &gt; Go To &gt; Test \(Ctrl + Shift + T\)를 하면 창이 뜨고 arithmeticMean\(\):int 메서드만 체크를 해주고 OK를 하면 다음과 같은 클래스가 생성될 것이다.

{% code-tabs %}
{% code-tabs-item title="ScoreCollection.java" %}
```java
public class ScoreCollectionTest {
    
    @Test
    public void arithmeticMean() {
    
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

그런 뒤에 실행해보면 통과하는 것을 볼 수 있다.



## 1.3 테스트 준비, 실행, 단언

앞에서 빈 껍데기를 테스트하여 JUnit을 사용할 준비가 되었으면 내용을 채워보자.

{% code-tabs %}
{% code-tabs-item title="ScoreCollectionTest.java" %}
```java
public class ScoreCollectionTest {

    @Test
    public void answersArithmeticMeanOfTwoNumbers() {
        ScoreCollection collection = new ScoreCollection();
        collection.add(() -> 5);
        collection.add(() -> 7);

        int actualResult = collection.arithmeticMean();

        assertThat(actualResult, equalTo(6));
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

간단히 코드를 설명하면 List안에 5 와 7을 추가\(line 5~8\)한다. 그리고 actualResult라는 변수는 해당 List를 순회하면서 원소를 다 더한 뒤 List의 크기로 나눠 준 값\(line 9\)이 들어간다. assertThat\(\) 메서드는 실제 결과와 matcher객체를 인자로 받고 기대되는 값인 6과 비교\(line 11\)한다. 

실행해보면 통과했다는 녹색 막대를 볼 수 있다. 연습을 위해 틀린 기댓값도 적용하여 실행시켜보자.

## 1.4 테스트가 정말로 뭔가를 테스트 하는가?

테스트 코드를 작성하는데 그치지말고 항상 해당 테스트가 실패하는지 확인해야 한다. 테스트 코드를 잘 작성하는 개발자들은 항상 테스트에서 먼저 실패한다. 실제로 TDD 실무자들은 어떻게 주기를 형성하여 작성하는지는 12장에서 설명한다.

## 1.5 마치며

책의 예제는 초반이기에 단순한 코드를 작성했지만 실제 프로젝트에 테스트 코드를 작성해본 개발자라면 알겠지만 이렇게 단순하지 않다. 테스트 코드를 작성하는데 익숙하지 않은 개발자라면 테스트 코드를 작성하는게 굉장히 번거롭고 짜증날 것이다. 

**하지만 효율적인 유지보수를 위해서는 테스트 코드작성은 필수이다. -test-**

