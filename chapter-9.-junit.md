# Chapter 9. 더 큰 설계 문제

## 9.1 Profile 클래스와 SRP

{% code-tabs %}
{% code-tabs-item title="Profile.java" %}
```java

/***
 * Excerpted from "Pragmatic Unit Testing in Java with JUnit",
 * published by The Pragmatic Bookshelf.
 * Copyrights apply to this code. It may not be used to create training material, 
 * courses, books, articles, and the like. Contact us if you are in doubt.
 * We make no guarantees that this code is fit for any purpose. 
 * Visit http://www.pragmaticprogrammer.com/titles/utj2 for more book information.
***/
package iloveyouboss;

import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class Profile {
   private Map<String,Answer> answers = new HashMap<>();

   private int score;
   private String name;

   public Profile(String name) {
      this.name = name;
   }
   
   public String getName() {
      return name;
   }

   public void add(Answer answer) {
      answers.put(answer.getQuestionText(), answer);
   }

   public boolean matches(Criteria criteria) {
      calculateScore(criteria);
      if (doesNotMeetAnyMustMatchCriterion(criteria))
         return false;
      return anyMatches(criteria);
   }

   private boolean doesNotMeetAnyMustMatchCriterion(Criteria criteria) {
      for (Criterion criterion: criteria) {
         boolean match = criterion.matches(answerMatching(criterion));
         if (!match && criterion.getWeight() == Weight.MustMatch) 
            return true;
      }
      return false;
   }

   private void calculateScore(Criteria criteria) {
      score = 0;
      for (Criterion criterion: criteria) 
         if (criterion.matches(answerMatching(criterion))) 
            score += criterion.getWeight().getValue();
   }

   private boolean anyMatches(Criteria criteria) {
      boolean anyMatches = false;
      for (Criterion criterion: criteria) 
         anyMatches |= criterion.matches(answerMatching(criterion));
      return anyMatches;
   }

   private Answer answerMatching(Criterion criterion) {
      return answers.get(criterion.getAnswer().getQuestionText());
   }

   public int score() {
      return score;
   }

   public List<Answer> classicFind(Predicate<Answer> pred) {
      List<Answer> results = new ArrayList<Answer>();
      for (Answer answer: answers.values())
         if (pred.test(answer))
            results.add(answer);
      return results;
   }
   
   @Override
   public String toString() {
     return name;
   }

   public List<Answer> find(Predicate<Answer> pred) {
      return answers.values().stream()
            .filter(pred)
            .collect(Collectors.toList());
   }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

이상적이지 않은 설계  

* 회사 혹은 인물 정보를 추적하고 관리
* 조건의 집합이 프로파일과 매칭되는지 여부 혹은 그 정도를 알려주는 점수 계산
* 위의 내용으로 보아 단일책임원칙\(SRP\) 위반

{% hint style="info" %}
SOLID 클래스의 설계 원칙

* 단일 책임 원칙\(SRP\) : 클래스는 변경할 때 한 가지 이유만 있어야함. 클래스는 작고 단일 목적을 추구함.
* 개방 폐쇄 원칙\(OCP\) : 클래스는 확장에 열려 있고 변경에는 닫혀 있어야함. 기존 클래스의 변경을 최소화해야함.
* 리스코프 치환 원칙\(LSP\) : 하위 타입은 반드시 상위 타입을 대체할 수 있어야함. 클라이언트 입장에서 오버라이딩한 메서드가 기능성을 깨면 안됨.
* 인터페이스 분리 원칙\(ISP\) : 클라이언트는 필요하지 않는 메서드에 의존하면 안됨. 커다란 인터페이스를 다수의 작은 인터페이스로 분할
* 의존성 역전 원칙\(DIP\) : 고수준 모듈은 저수준 모듈을 의존해서는 안됨. 둘 다 추상 클래스에 의존해야함. 추상클래스는 구체 클래스에 의존해서는 암됨. 구체 클래스는 추상 클래스에 의존해야함.
{% endhint %}

## 9.2 새로운 클래스 추출

```java
public boolean matches(Criteria criteria) {
   score = new MatchSet(answers, criteria).getScore();
   if (doesNotMeetAnyMustMatchCriterion(criteria))
      return false;
   return anyMatches(criteria);
}
```

매칭하는 기능 분리

{% code-tabs %}
{% code-tabs-item title="MatchSet.java" %}
```java
/***
 * Excerpted from "Pragmatic Unit Testing in Java with JUnit",
 * published by The Pragmatic Bookshelf.
 * Copyrights apply to this code. It may not be used to create training material, 
 * courses, books, articles, and the like. Contact us if you are in doubt.
 * We make no guarantees that this code is fit for any purpose. 
 * Visit http://www.pragmaticprogrammer.com/titles/utj2 for more book information.
***/
package iloveyouboss;

import java.util.*;

public class MatchSet {
   private Map<String, Answer> answers;
   private int score = 0;

   public MatchSet(Map<String, Answer> answers, Criteria criteria) {
      this.answers = answers;
      calculateScore(criteria);
   }

   private void calculateScore(Criteria criteria) {
      for (Criterion criterion: criteria) 
         if (criterion.matches(answerMatching(criterion))) 
            score += criterion.getWeight().getValue();
   }

   private Answer answerMatching(Criterion criterion) {
      return answers.get(criterion.getAnswer().getQuestionText());
   }

   public int getScore() {
      return score;
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Profile 클래스를 실세계 개념에 잘 맞는다는 이유로 단일 클래스로 한정한다면 피해가 커짐 \( 클래스는 점점 커질 것이며 복잡해짐, 클래스를 수정할 때 마다 관련 없는 항목들이 깨지기 쉬움 \)

## 9.3 명령-질의 분리

```java
public boolean matches(Criteria criteria) {
   MatchSet matchSet = new MatchSet(answers, criteria);
   score = matchSet.getScore();
   return matchSet.matches();
}
```

* 계산된 점수를 Profile객체의 필드로 저장하는 어색한 부작용 존재
* Profile객체는 단일 점수를 가지 않으며 조건과 매칭될 때만 점수 산출
* Profile 클래스에 score 삭제

{% hint style="info" %}
어떤 값을 반환하고 부작용을 발생시키는 \(시스템에 있는 어떤 클래스 혹은 엔터티의 상태 변경\) 메서드는 명령-질의 분리 원칙 위반.

메서드는 명령을 실행하거나 질의에 대답할 수 있으며, 두 작업을 모두 하면 암됨.
{% endhint %}

## 9.4 단위 테스트의 유지 보수 비용

### 9.4.1 자신을 보호하는 방법

코드 중복의 두가지 문제점

1. 테스트를 따르기가 어려움
2. 작은 코드 조작들을 단일 메서드로 추출하면 그 코드 조작들을 변경해야 할 때 미치는 영향을 최소화 할 수 있음.다수의 장소에 흩어진 테스트를 수정하기 보다 단일 장소를 수정하는 것이 유리

private 메서드를 테스트하려는 충동은 클래스가 필요 이상으로 커졌다는 힌트 - 내부 동작을 새 클래스로 옮기고 public으로 만드는 것이 좋음

단위 테스트가 어려워 보임

### 9.4.2 깨진 테스트 고치기

{% code-tabs %}
{% code-tabs-item title="MatchSetTest.java" %}
```java
/***
 * Excerpted from "Pragmatic Unit Testing in Java with JUnit",
 * published by The Pragmatic Bookshelf.
 * Copyrights apply to this code. It may not be used to create training material, 
 * courses, books, articles, and the like. Contact us if you are in doubt.
 * We make no guarantees that this code is fit for any purpose. 
 * Visit http://www.pragmaticprogrammer.com/titles/utj2 for more book information.
***/
package iloveyouboss;

import static org.junit.Assert.*;
import java.util.*;
import org.junit.*;
import static org.hamcrest.CoreMatchers.*;

public class MatchSetTest {
   private Criteria criteria;
   private Question questionReimbursesTuition;
   // ...
   private Answer answerReimbursesTuition;
   private Answer answerDoesNotReimburseTuition;

   private Question questionIsThereRelocation;
   private Answer answerThereIsRelocation;
   private Answer answerThereIsNoRelocation;

   private Question questionOnsiteDaycare;
   private Answer answerNoOnsiteDaycare;
   private Answer answerHasOnsiteDaycare;

   private Map<String,Answer> answers;

   @Before
   public void createAnswers() {
      answers = new HashMap<>();
   }

   @Before
   public void createCriteria() {
      criteria = new Criteria();
   }

   @Before
   public void createQuestionsAndAnswers() {
   // ...
      questionIsThereRelocation =
            new BooleanQuestion(1, "Relocation package?");
      answerThereIsRelocation =
            new Answer(questionIsThereRelocation, Bool.TRUE);
      answerThereIsNoRelocation =
            new Answer(questionIsThereRelocation, Bool.FALSE);

      questionReimbursesTuition =
            new BooleanQuestion(1, "Reimburses tuition?");
      answerReimbursesTuition =
            new Answer(questionReimbursesTuition, Bool.TRUE);
      answerDoesNotReimburseTuition =
            new Answer(questionReimbursesTuition, Bool.FALSE);

      questionOnsiteDaycare =
            new BooleanQuestion(1, "Onsite daycare?");
      answerHasOnsiteDaycare =
            new Answer(questionOnsiteDaycare, Bool.TRUE);
      answerNoOnsiteDaycare =
            new Answer(questionOnsiteDaycare, Bool.FALSE);
   }

   private void add(Answer answer) {
      answers.put(answer.getQuestionText(), answer);
   }

   private MatchSet createMatchSet() {
      return new MatchSet(answers, criteria);
   }

   @Test
   public void matchAnswersFalseWhenMustMatchCriteriaNotMet() {
      add(answerDoesNotReimburseTuition);
      criteria.add(
            new Criterion(answerReimbursesTuition, Weight.MustMatch));

      assertFalse(createMatchSet().matches());
   }

   @Test
   public void matchAnswersTrueForAnyDontCareCriteria() {
      add(answerDoesNotReimburseTuition);
      criteria.add(
            new Criterion(answerReimbursesTuition, Weight.DontCare));

      assertTrue(createMatchSet().matches());
   }
   // ...

   @Test
   public void matchAnswersTrueWhenAnyOfMultipleCriteriaMatch() {
      add(answerThereIsRelocation);
      add(answerDoesNotReimburseTuition);
      criteria.add(new Criterion(answerThereIsRelocation, Weight.Important));
      criteria.add(new Criterion(answerReimbursesTuition, Weight.Important));

      assertTrue(createMatchSet().matches());
   }

   @Test
   public void matchAnswersFalseWhenNoneOfMultipleCriteriaMatch() {
      add(answerThereIsNoRelocation);
      add(answerDoesNotReimburseTuition);
      criteria.add(new Criterion(answerThereIsRelocation, Weight.Important));
      criteria.add(new Criterion(answerReimbursesTuition, Weight.Important));

      assertFalse(createMatchSet().matches());
   }

   @Test
   public void scoreIsZeroWhenThereAreNoMatches() {
      add(answerThereIsNoRelocation);
      criteria.add(new Criterion(answerThereIsRelocation, Weight.Important));

      assertThat(createMatchSet().getScore(), equalTo(0));
   }

   @Test
   public void scoreIsCriterionValueForSingleMatch() {
      add(answerThereIsRelocation);
      criteria.add(new Criterion(answerThereIsRelocation, Weight.Important));

      assertThat(createMatchSet().getScore(), equalTo(Weight.Important.getValue()));
   }

   @Test
   public void scoreAccumulatesCriterionValuesForMatches() {
      add(answerThereIsRelocation);
      add(answerReimbursesTuition);
      add(answerNoOnsiteDaycare);
      criteria.add(new Criterion(answerThereIsRelocation, Weight.Important));
      criteria.add(new Criterion(answerReimbursesTuition, Weight.WouldPrefer));
      criteria.add(new Criterion(answerHasOnsiteDaycare, Weight.VeryImportant));

      int expectedScore = Weight.Important.getValue() + Weight.WouldPrefer.getValue();
      assertThat(createMatchSet().getScore(), equalTo(expectedScore));
   }

   // TODO: missing functionality--what if there is no matching profile answer for a criterion?

}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

MatchSet 코드를 테스트하는 코드는 더 이상 Profile 객체를 생성하지 않아도 됨.

테스트가 더 직관적이고, 작성하기 쉬워졌음.

## 9.5 다른 설계에 관한 생각들

MatchSet\(\) 생성자는 점수를 계산하는 작업을 함. \( 계산된 점수를 클라이언트에서 사용하지 않는다면 그것을 계산하는 노력은 낭비임 \)

요청을 받았을 때 점수를 계산하도록 코드 변경

{% code-tabs %}
{% code-tabs-item title="MatchSet.java" %}
```java
public int getScore() {
   int score = 0;
   for (Criterion criterion: criteria) 
      if (criterion.matches(answerMatching(criterion))) 
         score += criterion.getWeight().getValue();
   return score;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Score 필드 삭제, calculateScore\(\) 메서드는 getScore\(\) 매서드 내부로 인라인됨.

> answer 컬렉션 다루기
>
> Profile 클래스에서 질문 내용을 키로 사용하는 Map&lt;String, Answer&gt; 객체 생성
>
> 동시에 answers 맴 참조를 새로 생성되는 matchSet 객체로 넘김 
>
> 두 클래스가 어떻게 답변을 탐색하고 점수를 구하는지에 대한 정보를 너무 많이 가지고 있다는 의미
>
> 따라서 AnswerCollection 클래스로 분리

{% hint style="info" %}
방문자 패턴 - 방문자 패턴은 특수한 경우에만 쓰기 때문에 일반적인 상황에서는 권장 X

데이터 구조와 연산을 분리하여 데이터 구조의 원소들을 변경하지 않고 새로운 연산을 추가 할 구 있다. 새로운 연산을 추가하려면 새로운 방문자를 추가하기만 하면 됨

참고 링크

[https://lktprogrammer.tistory.com/58](https://lktprogrammer.tistory.com/58)
{% endhint %}



