# Chapter 11. 테스트 리팩토링

## 11. 1 이해 검색

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
import java.io.*;
import java.net.*;
import java.util.*;
import org.junit.*;
import java.util.logging.*;
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;

public class SearchTest {
   @Test
   public void testSearch() {
      try {
        String pageContent = "There are certain queer times and occasions "
              + "in this strange mixed affair we call life when a man "
              + "takes this whole universe for a vast practical joke, "
              + "though the wit thereof he but dimly discerns, and more "
              + "than suspects that the joke is at nobody's expense but "
              + "his own.";
         byte[] bytes = pageContent.getBytes();
         ByteArrayInputStream stream = new ByteArrayInputStream(bytes);
         // search
         Search search = new Search(stream, "practical joke", "1");
         Search.LOGGER.setLevel(Level.OFF);
         search.setSurroundingCharacterCount(10);
         search.execute();
         assertFalse(search.errored());
         List<Match> matches = search.getMatches();
         assertThat(matches, is(notNullValue()));
         assertTrue(matches.size() >= 1);
         Match match = matches.get(0);
         assertThat(match.searchString, equalTo("practical joke"));
         assertThat(match.surroundingContext, 
               equalTo("or a vast practical joke, though t"));
         stream.close();

         // negative
         URLConnection connection = 
               new URL("http://bit.ly/15sYPA7").openConnection();
         InputStream inputStream = connection.getInputStream();
         search = new Search(inputStream, "smelt", "http://bit.ly/15sYPA7");
         search.execute();
         assertThat(search.getMatches().size(), equalTo(0));
         stream.close();
      } catch (Exception e) {
         e.printStackTrace();
         fail("exception thrown in test" + e.getMessage());
      }
   }
}


```
{% endcode-tabs-item %}
{% endcode-tabs %}

위의 코드는 어떤 테스트를 하려는 코드인지 알 수 없다. 이 코드를 더 깔끔하고 표현력 좋은 테스트 코드로 바꾸어 나가는 과정을 가져보자.

## 11.2 테스트 냄새 : 불필요한 테스트 코드

현재 testSerach\(\) 메서드로 구성된 테스트 코드는 어떤 예외도 던지지 않을 것으로 보인다. 테스트 코드가 예외를 던진다면 try/catch 블록이 System.out 으로 스택 트레스를 분출할 것이기 때문이다. 다른 말로 테스트 메서드는 예외를 기대하지 않는다. 예외를 기대하지 않는다면 날아가게 throws 하게 두면 된다. JUnit이 테스트에서 던지는 예외들을 잡아서 오류로 표시하고 출력창에 스택 트레이스로 보여주기 때문이다. 명시적 try/catch 블록은 부가가치가 없다. 따라서 try/catch 블록은 제거하고 IOException을 throws 하도록 변경한다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
import java.io.*;
import java.net.*;
import java.util.*;
import org.junit.*;
import java.util.logging.*;
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;

// eliminate unnecessary test code (assert not null, try/catch)
public class SearchTest {
   @Test
   public void testSearch() throws IOException {
      String pageContent = "There are certain queer times and occasions "
            + "in this strange mixed affair we call life when a man "
            + "takes this whole universe for a vast practical joke, "
            + "though the wit thereof he but dimly discerns, and more "
            + "than suspects that the joke is at nobody's expense but "
            + "his own.";
      byte[] bytes = pageContent.getBytes();
      // ...
      stream.close();
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

try/catch 블록이 제거되면서 조금은 깔끔해 보인다. 이제 또 다른 곳을 보자.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
List<Match> matches = search.getMatches();
assertThat(matches, is(notNullValue()));
assertTrue(matches.size() >= 1);
```
{% endcode-tabs-item %}
{% endcode-tabs %}

serach.getMatches\(\) 호출결과는 matches 에 할당된다. 그리고 그 다음 문장은 matches 변수가 null이 아님을 단언한다. 마지막 단언은 matches 변수의 크기가 1보다 크거나 같다는 것을 검증한다.

어떤 변수를 참조하기 전에 그 변수가 null이 아님을 검사하는 것은 안전하고 좋은 일이다. 하지만 그것은 프로덕션 코드\(실제 서비스되는 코드\)에서는 좋아도 이 테스트에서는 불필요하다. 추가했을 때의 이점이 없기 때문이다. 아래에 matches.size\(\) 에서 matches가 null이라면 예외를 던질 것이고 JUnit은 이것을 잡아 테스트를 오류로 처리할 것이기 때문이다. 따라서 제거한다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
import java.io.*;
import java.net.*;
import java.util.*;
import org.junit.*;
import java.util.logging.*;
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;

// eliminate unnecessary test code (assert not null, try/catch)
public class SearchTest {
   @Test
   public void testSearch() throws IOException {
      String pageContent = "There are certain queer times and occasions "
            + "in this strange mixed affair we call life when a man "
            + "takes this whole universe for a vast practical joke, "
            + "though the wit thereof he but dimly discerns, and more "
            + "than suspects that the joke is at nobody's expense but "
            + "his own.";
      byte[] bytes = pageContent.getBytes();
      // ...
      ByteArrayInputStream stream = new ByteArrayInputStream(bytes);
      // search
      Search search = new Search(stream, "practical joke", "1");
      Search.LOGGER.setLevel(Level.OFF);
      search.setSurroundingCharacterCount(10);
      search.execute();
      assertFalse(search.errored());
      List<Match> matches = search.getMatches();
      assertTrue(matches.size() >= 1);
      Match match = matches.get(0);
      assertThat(match.searchString, equalTo("practical joke"));
      assertThat(match.surroundingContext, equalTo(
            "or a vast practical joke, though t"));
      stream.close();

      // negative
      URLConnection connection = 
            new URL("http://bit.ly/15sYPA7").openConnection();
      InputStream inputStream = connection.getInputStream();
      search = new Search(inputStream, "smelt", "http://bit.ly/15sYPA7");
      search.execute();
      assertThat(search.getMatches().size(), equalTo(0));
      stream.close();
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 11.3 테스트 냄새 : 추상화 누락

잘 구성된 테스트는 시스템과 상호 작용을 아래와 코드 같이 '데이터 준비하기', '시스템과 동작하기', '결과 단언하기' 세 가지 관점에서 보여준다. \(4.1절 AAA\)

{% code-tabs %}
{% code-tabs-item title="ScoreCollectionTest.java" %}
```java
import static org.junit.Assert.*;
import static org.hamcrest.CoreMatchers.*; 
import org.junit.*;

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

현재 뒤죽박죽된 테스트는 search.getMatches\(\) 호출에서 반환된 매칭 목록에 대해 단언문 5줄을 포함한다. 이 5줄을 읽어야만 무슨 작업인지 이해할 수 있다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
List<Match> matches = search.getMatches();
assertTrue(matches.size() >= 1);
Match match = matches.get(0);
assertThat(match.searchString, equalTo("practical joke"));
assertThat(match.surroundingContext, 
        equalTo("or a vast practical joke, though t"));
```
{% endcode-tabs-item %}
{% endcode-tabs %}

하나의 개념을 구체화하고 있는 단언이며 하나의 개념을 구체화하고 있다는 말은 하나의 assert문으로도 충분히 테스트할 수 있는 케이스라는 말이된다. 이 단언문 5줄을 포괄하는 사용자 정의 단언문을 만들어보자. 사용자 정의 단언문에 대한 자세한 내용은 7.3.1절에 나와있다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
import java.io.*;
import java.net.*;
import java.util.*;
import org.junit.*;
import java.util.logging.*;
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;
import static util.ContainsMatches.*;

public class SearchTest {
   @Test
   public void testSearch() throws IOException {
      String pageContent = "There are certain queer times and occasions "
            // ...
      search.execute();
      assertFalse(search.errored());
      assertThat(search.getMatches(), containsMatches(new Match[] { 
         new Match("1", "practical joke", 
                   "or a vast practical joke, though t") }));
      stream.close();
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="ContainsMatches.java" %}
```java
import java.util.*;
import org.hamcrest.*;

public class ContainsMatches extends TypeSafeMatcher<List<Match>> {
   private Match[] expected;

   public ContainsMatches(Match[] expected) {
      this.expected = expected;
   }

   @Override
   public void describeTo(Description description) {
      description.appendText("<" + expected.toString() + ">");
   }

   private boolean equals(Match expected, Match actual) {
      return expected.searchString.equals(actual.searchString)
         && expected.surroundingContext.equals(actual.surroundingContext);
   }

   @Override
   protected boolean matchesSafely(List<Match> actual) {
      if (actual.size() != expected.length)
         return false;
      for (int i = 0; i < expected.length; i++)
         if (!equals(expected[i], actual.get(i)))
            return false;
      return true;
   }

   @Factory
   public static <T> Matcher<List<Match>> containsMatches(Match[] expected) {
      return new ContainsMatches(expected);
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

여기서 잠깐 기억이 안날 것을 위해 사용자 정의 매처는 기본적으로 TypeSafeMatcher클래스를 상속하고 매칭하고자 하는 타입을 지정한다. 또한 MatchesSafely\(\), describeTo\(\) 메서드를 Override 해야하며 MatchesSafely\(\) 메서드는 제약사항을 적어주면 되고, 이를 어길 경우 false를 반환한다. describeTo\(\) 메서드는 단언이 실패할 때 제공할 의미 있는 메시지를 기재한다.

이렇게 단일 개념을 구현하는 여러 줄을 발견했다면 깔끔한 1줄로 추출할 수 있는지 고민해 보아야한다.

이번엔 테스트의 두 번째 덩어리에 대체 추상화를 도입해보자. 마지막 단언을 보면 결과 크기가 0인지 비교한다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
public class SearchTest {
   @Test
   public void testSearch() throws IOException {
      // ...
      URLConnection connection = 
            new URL("http://bit.ly/15sYPA7").openConnection();
      InputStream inputStream = connection.getInputStream();
      search = new Search(inputStream, "smelt", "http://bit.ly/15sYPA7");
      search.execute();
      assertThat(search.getMatches().size(), equalTo(0));
      stream.close();
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

여기서 확인하고자 하는 것은 size가 '비어있다' 를 확인하는 개념이다. 그렇기 때문에 다음과 같이 바꾸면 조금 더 이해하기 편한 코드가 된다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
public class SearchTest {
   @Test
   public void testSearch() throws IOException {
      // ...
      URLConnection connection = 
            new URL("http://bit.ly/15sYPA7").openConnection();
      InputStream inputStream = connection.getInputStream();
      search = new Search(inputStream, "smelt", "http://bit.ly/15sYPA7");
      search.execute();
      assertThat(search.getMatches().isEmpty());
      stream.close();
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}



