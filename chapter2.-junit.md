## Junit 진짜로써 보기 ##

테스트 배치에 대한 구조 <br>
**준비-실행-단언(AAA, Arrange-Act-Assert)**

아래의 Profile.java 는 테스트 대상의 코드입니다.<br>
회사, 구직자의 질문 답변을  가지고 각각이 원하는 인재 , 원하는 회사인지 선별해주는 코드입니다.

*기타 부가적인 코드는 아래의 깃헙주소에서 확인 할 수 있습니다.<br>
[github](https://github.com/gilbutITbook/006814/tree/master/iloveyouboss_06)


```java
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
        score = 0;

        boolean kill = false;
        boolean anyMatches = false;
        for (Criterion criterion: criteria) {
            Answer answer = answers.get(
                    criterion.getAnswer().getQuestionText());
            boolean match =
                    criterion.getWeight() == Weight.DontCare ||
                            answer.match(criterion.getAnswer());

            if (!match && criterion.getWeight() == Weight.MustMatch) {
                kill = true;
            }
            if (match) {
                score += criterion.getWeight().getValue();
            }
            anyMatches |= match;
        }
        if (kill)
            return false;
        return anyMatches;
    }

    public int score() {
        return score;
    }
}
```

위에 코드에 테스트 케이스를 테스트 코드로 작성해보도록 하겠습니다.<br>

```java
import org.junit.Test;

import static org.junit.Assert.*;

public class ProfileTest {

    @Test
    public void matches가_false인경우() {
        Profile profile = new Profile("Bull Hockey, Inc.");
        Question question = new BooleanQuestion(1, "Got bonuses?");
        Answer profileAnswer = new Answer(question, Bool.FALSE);
        profile.add(profileAnswer);
        Criteria criteria = new Criteria();
        Answer criteriaAnswer = new Answer(question, Bool.TRUE);
        Criterion criterion = new Criterion(criteriaAnswer, Weight.MustMatch);
        criteria.add(criterion);

        boolean matches = profile.matches(criteria);

        assertFalse(matches);
    }

    @Test
    public void matches가_참인경우() {
        Profile profile = new Profile("Bull Hockey, Inc.");
        Question question = new BooleanQuestion(1, "Got milk?");
        Answer profileAnswer = new Answer(question, Bool.FALSE);
        profile.add(profileAnswer);
        Criteria criteria = new Criteria();
        Answer criteriaAnswer = new Answer(question, Bool.TRUE);
        Criterion criterion = new Criterion(criteriaAnswer, Weight.DontCare);
        criteria.add(criterion);

        boolean matches = profile.matches(criteria);
        assertTrue(matches);
    }


}
```








### @Before 메서드로 테스트 초기화  ###

Junit 테스트를 실행할 때마다 @Before 애너테이션으로 선언한 메서드를 먼저 실행하므로 테스트 하기전에 초기화해야하는 로직 OR 공통 로직에  선언합니다.

아래와 같이 **@Before** 애너테이션으로 공통부분을 초기화 하므로서 코드가 훨씬 간결해진 것을 확인 할 수 있습니다.<br> 

테스트마다 새로운 인스턴스를 생성하여 테스트를 진행하기 때문에 아래의 순서로 실행이 됩니다.<br> 

1.@Before 메서드를 호출하여 초기화<br> <br> 
2.matchAnswersFalseWhenMustMatchCriteriaNotMet() 메서드 테스트<br> <br> 
3.@Before 메서드를 호출하여 초기화<br> <br> 
4.matchAnswersTrueForAnyDontCareCriteria() 메서드 테스트<br> 

{% code-tabs %}
{% code-tabs-item title="Scoreable.java" %}
```java
public class ProfileTest2 {
    private Profile profile;
    private BooleanQuestion question;
    private Criteria criteria;

    @Before
    public void create() {
        profile = new Profile("Bull Hockey, Inc.");
        question = new BooleanQuestion(1, "Got bonuses?");
        criteria = new Criteria();
    }

    @Test
    public void matchAnswersFalseWhenMustMatchCriteriaNotMet() {
        profile.add(new Answer(question, Bool.FALSE));
        criteria.add(
                new Criterion(new Answer(question, Bool.TRUE), Weight.MustMatch));

        boolean matches = profile.matches(criteria);

        assertFalse(matches);
    }

    @Test
    public void matchAnswersTrueForAnyDontCareCriteria() {
        profile.add(new Answer(question, Bool.FALSE));
        criteria.add(
                new Criterion(new Answer(question, Bool.TRUE), Weight.DontCare));

        boolean matches = profile.matches(criteria);

        assertTrue(matches);
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}
