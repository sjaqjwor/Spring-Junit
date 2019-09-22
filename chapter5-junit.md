# chapter 5. FIRST

### - 좋은 테스트를 위한 FIRST 속성

1. **\[F\]ast:** 빠른
2. **\[I\]solated:** 고립된
3. **\[R\]epeatable:** 반복 가능한
4. **\[S\]elf-validating:** 스스로 검증 가능한
5. **\[T\]imely:** 적시의

## 1.\[F\]irst: 빠르다

> 로컬에 있는 로직만 테스트하도록 외부 시스템 접근을 최소화하자.
>
> > "전형적인 자바 시스템에는 단위 테스트 수천 개가 필요하다.  
> > 평균 200ms가 걸린다면, 2500개의 테스트를 수행하는데 8분이 걸린다"

### 예시코드 \(외부시스템 접근 분리\)

#### 1. 접근 분리 전

```java
public class StatCompiler {
    QuestionController controller = new QuestionController();

    public void responseByQuestion(List answers) {
        Map<Integer, Map<Boolean, AtomicInteger>> responses = new HashMap<>();

        answers.stream().forEach(answer -> incrementHistogram(responses, answer));

        return convertHistogramIdsToText(responses);

    }

    private void convertHistogramIdsToText(Map responses) {
        Map textResponses = new HashMap<>();

        // 질문과 질문에 대한 결과를 textResponses에 저장 
        responses.keySet().stream().forEach(id -> textResponses.put(
                controller.find(id).getText(), // 이 부분에서 질문을 조회하기 위해 db 접근함.
                responses.get(id)
        ));

        return textResponses;
    }


    private void incrementHistogram(Map responses, BooleanAnswer answers) {
        // answer(질문 답)에 대한 결과를 +1씩 증가시키며 responses에 저장.

    }  
}
```

responseByQuestion\(\)를 테스트하기 위해선 연관되어 있는 convertHistogramIdsToText\(\) 메소드 때문에 무조건 외부시스템에 접근 할 수 밖에 없다.

#### 2. 접근 분리 후

```java
public class StatCompiler {
    // questions 변수를 직접 받도록하여 테스트 시, 외부시스템 접근하지 않도록 함.
    public void responseByQuestion(List answers, Map questions) {
        Map<Integer, Map<Boolean, AtomicInteger>> responses = new HashMap<>();

        answers.stream().forEach(answer -> incrementHistogram(responses, answer));

        return convertHistogramIdsToText(responses, questions);

    }

    // 외부시스템 접근하는 코드를 questionText 메소드로 분리.
    public Map<Integer, String> questionText(List<BooleanAnswer> answers) {
        Map<Integer, String> questions = new HashMap<>();

        answers.stream().forEach(answer -> {
            if(!questions.containsKey(answer.getQuestionId())) {
                questions.put(answer.getQuestionId(),
                                controller.find(answer.getQuestionId()).getText());
            }
        });

        return questions;
    }

    private void convertHistogramIdsToText(Map responses, Map questions) {
        Map textResponses = new HashMap<>();

        // 질문과 질문에 대한 결과를 textResponses에 저장 
        responses.keySet().stream().forEach(id -> textResponses.put(
                questions.get(id),
                responses.get(id)
        ));

        return textResponses;
    }


    private void incrementHistogram(Map responses, BooleanAnswer answers) {
        // answer(질문 답)에 대한 결과를 +1씩 증가시키며 responses에 저장.

    }  
}
```

외부 시스템 접근하는 코드를 questionText\(\) 메소드로 분리하여 responseByQuestion\(\) 메소드를 빠르게 테스트해볼 수 있다.

```text
@Test
public void responsesByQuestionAnswersCountsByQuestionText() {
    StatCompiler stats = new StatCompiler();
    List<BooleanAnswer> answers = new ArrayList<>();
    answers.add(new BooleanAnswer(1, true));
    answers.add(new BooleanAnswer(1, true));
    answers.add(new BooleanAnswer(1, true));
    answers.add(new BooleanAnswer(1, false));
    answers.add(new BooleanAnswer(2, true));
    answers.add(new BooleanAnswer(2, true));
    Map<Integer, String> questions = new HashMap<>();
    // 아래와 같이 questions 직접 생성하여 responseByQuestion 메소드에 인수로 사용
    questions.put(1, "Tuition reimbursement?");
    questions.put(2, "Relocation package?");

    Map<String, Map<Boolean, AtomicInteger>> responses = stats.responseByQuestion(answers, questions);

    assertThat(responses.get("Tuition reimbursement?").get(Boolean.TRUE).get(), equalTo(3));
    assertThat(responses.get("Tuition reimbursement?").get(Boolean.FALSE).get(), equalTo(1));
    assertThat(responses.get("Relocation package?").get(Boolean.TRUE).get(), equalTo(2));
    assertThat(responses.get("Relocation package?").get(Boolean.FALSE).get(), equalTo(0));
}
```

## 2.f\[I\]rst: 고립시킨다

1. 데이터베이스와의 의존성을 최소화하자.
   1. 테스트를 하기 전에, 데이터베이스의 데이터가 올바른지 체크해야하는 불편함.
   2. 다른 개발자들이 같은 테스트를 동시에 실행하여 데이터를 바꿀 수도 있음.
   3. 테스트는 실행 시, 언제나 같은 결과를 가져야한다. \(데이터 때문에 다른 결과가 발생할 수 있다.\)
2. 다른 단위테스트와의 의존성을 만들지말자.
   1. 다른 단위테스트와 의존하게 될 경우, 어디서 오류가 발생했는지 찾기 위해 비교적 많은 시간이 소요된다.

> 객체 지향 클래스 원칙인 SRP\(단일 책임 원칙\)은 테스트 메소드를 생성할 때도 적용된다.  
> 테스트 메소드도 SRP 지침처럼 "작고 단일한 목적"을 가져야하며 항상 고립되어 있어야 한다.

## 3.fi\[R\]st: 좋은 테스트는 반복 가능해야 한다

> 반복 가능한 테스트는 실행할 때마다 결과가 같아야 한다.  
> 즉, 반복 가능한 테스트를 만들기 위해서 테스트 메소드는 외부 환경과 격리되어야 한다.

예를 들어, 데이터베이스의 데이터 생성날짜를 테스트한다고 생각해보자.  
현재 시간과 같은 외부 환경은 계속 바뀌므로 테스트하기 힘들지만, 아래 코드처럼 테스트 코드를 작성할 수 있다.

```java
public class QuestionController {
    private Clock clock = Clock.systemUTC();

    public int addBooleanQuestion(String text) {
        return persist(new BooleanQuestion(text));
    }

    void setClock(Clock clock) {
        this.clock = clock;
    }

    private int persist(Persistable object) {
        object.setCreateTimestamp(clock.instant());
        execute(em -> em.persist(object));
        return object.getId();
    }
}
```

```text
@Test
public void questionAnswersDateAdded() { 
    Instant now = new Date().toInstant(); // 1.
    controller.setClock(Clock.fixed(now, ZoneId.of("America/Denver"))); // 2.
    int id = controller.addBooleanQuestion("text"); // 3.

    Question question = controller.find(id);

    assertThat(question.getCreateTimestamp(), equaulTo(now)); // 4.
}
```

1. now 변수에 현재 시간을 할당
2. controller의 멤버변수인 clock에 now 변수값을 인수로 사용
3. controller로 db에 질문 등록 시, now 값을 timestamp로 사용
4. 단언 부분에서 타임스탬프 값과 now값이 동일한지 비교

위 코드처럼, 외부 상태에 영향을 받지 않도록 테스트 코드를 작성해야 언제나 같은 결과를 볼 수 있다.  
다른 결과가 나오는 테스트 코드는 진짜 버그인지, 테스트 코드의 문제인지 알수 없으므로 굉장히 중요하다.

## 4.fir\[S\]t: 스스로 검증 가능하다

1. 어떤 메소드를 검증할 때, junit을 사용하지 않고 main\(\)메소드로 검증한다고 생각해보자. 해당 메소드의 결과 값이 정상적인 값인지, 개발자가 출력문을 통해 직접 확인해야한다. \(junit은 한번 코드를 작성해놓으면 초록,빨강과 같은 신호로 체크할 수 있다.\)
2. 코드를 변경하거나 배포할 때, 테스트를 자동화할 수 있다. 젠킨스와 같은 배포도구를 사용하면, 소스코드를 배포하기전에 빌드서버에서 소스코드를 빌드하고 모든 테스트를 실행, 검증한 뒤에 배포서버에 코드를 배포하게 할 수 있다.

> infinitest같은 플러그인도 있는데, 소스코드 변경을 감지하여 관련된 테스트코드를 자동으로 실행해준다고 한다.

## 5.firs\[t\]: 적시에 사용한다

단위 테스트는 새로 구현하는 코드에 대해서는 아주 좋은 습관이며,  
단위 테스트를 많이 만들 수록 오히려 테스트 대상 코드들이 줄어든다고 한다.

하지만, 레거시 코드에 대한 테스트 코드를 작성하는 일은 자제하자. \(시간낭비라고 함.\)  
해당 코드에 큰 결함이 없으며 변경할 예정이 없다면 테스트 코드를 굳이 작성할 필요 없다.

> 코드 작성 -&gt; 테스트 코드 작성 단계가 익숙해지면  
> 테스트 코드 작성 -&gt; 코드 작성 순서의 단계를 고려해보자.

