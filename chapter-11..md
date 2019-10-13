# Chapter 11. 테스트 리팩토링

## 11. 1 이해 검색

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
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

## 11.4 테스트 냄새 : 부적절한 정보

때로는 테스트에는 부적절하지만, 당장 컴파일되어야 하기 때문에 데이터를 넣기도 한다. 예를 들어 메서드가 테스트에는 어떤 영향도 없는 부가적인 인수를 취하기도 한다. 현재 코드에서는 그 의미가 불분명한 '매직 리터럴'들을 포함한다. 여기서 매직 리터럴은 프로그래밍에서 상수로 선언하지 않은 숫자 리터럴을 의미하며 코드에는 되도록 사용하면 안된다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
Search search = new Search(stream, "practical joke", "1");
```
{% endcode-tabs-item %}
{% endcode-tabs %}

그리고 다음 코드를 보자.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
assertThat(search.getMatches(), containsMatches(new Match[] { 
         new Match("1", "practical joke", 
                   "or a vast practical joke, though t") }));
```
{% endcode-tabs-item %}
{% endcode-tabs %}

여기서 문자열 "1" 이 무엇을 의미하는지 확신할 수 없다. 따라서 의미를 파악하려면 Match와 Search 클래스를 뒤져보아야한다. 그리고 이제 "1" 이 검색 제목을 의미하고 실제로는 신경 쓰지 않는 필드 값이라는 것을 알았다. 그렇다면 이를 의미 있는 이름을 가진 상수를 도입해 즉시 파악할 수 있도록 하는 것이 좋다. 

Search클래스의 생성에 대한 두 번째 호출은 제목 인수로 URL을 포함한다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
URLConnection connection = 
            new URL("http://bit.ly/15sYPA7").openConnection();
InputStream inputStream = connection.getInputStream();
search = new Search(inputStream, "smelt", "http://bit.ly/15sYPA7");
```
{% endcode-tabs-item %}
{% endcode-tabs %}

언뜻 보면, 위에 나온 URL생성자에 포함된 URL과 Search 생성자의 포함된 세번째 인수 URL이 연관있어 보이지만 실제로는 그렇지 않다. 그렇기 때문에 혼란을 줄 수 있으므로 Search의 URL은 상수로 대체한다. 이를 적용하면 아래와 같다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
public class SearchTest {
   private static final String A_TITLE = "1";
   @Test
   public void testSearch() throws IOException {
      String pageContent = "There are certain queer times and occasions "
            + "in this strange mixed affair we call life when a man "
            + "takes this whole universe for a vast practical joke, "
            + "though the wit thereof he but dimly discerns, and more "
            + "than suspects that the joke is at nobody's expense but "
            + "his own.";
      byte[] bytes = pageContent.getBytes();
      ByteArrayInputStream stream = new ByteArrayInputStream(bytes);
      // search
      Search search = new Search(stream, "practical joke", A_TITLE);
      Search.LOGGER.setLevel(Level.OFF);
      search.setSurroundingCharacterCount(10);
      search.execute();
      assertFalse(search.errored());
      assertThat(search.getMatches(), containsMatches(new Match[] 
         { new Match(A_TITLE, "practical joke", 
                              "or a vast practical joke, though t") }));
      stream.close();

      // negative
      URLConnection connection = 
            new URL("http://bit.ly/15sYPA7").openConnection();
      InputStream inputStream = connection.getInputStream();
      search = new Search(inputStream, "smelt", A_TITLE);
      search.execute();
      assertTrue(search.getMatches().isEmpty());
      stream.close();
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 11.5 테스트 냄새 : 부푼 생성

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
String pageContent = "There are certain queer times and occasions "
            + "in this strange mixed affair we call life when a man "
            + "takes this whole universe for a vast practical joke, "
            + "though the wit thereof he but dimly discerns, and more "
            + "than suspects that the joke is at nobody's expense but "
            + "his own.";
byte[] bytes = pageContent.getBytes();
ByteArrayInputStream stream = new ByteArrayInputStream(bytes);
```
{% endcode-tabs-item %}
{% endcode-tabs %}

테스트 코드에서 Search 객체의 생성자에 InputStream 객체를 넘긴다. 그 생성은 세 문장으로 이루어져있는데 이를 줄일이기 위해 도우미 메서드를 만들자.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
public class SearchTest {
   private static final String A_TITLE = "1";

   @Test
   public void testSearch() throws IOException {
      InputStream stream =
            streamOn("There are certain queer times and occasions "
             + "in this strange mixed affair we call life when a man "
             + "takes this whole universe for a vast practical joke, "
             + "though the wit thereof he but dimly discerns, and more "
             + "than suspects that the joke is at nobody's expense but "
             + "his own.");
      // search
      Search search = new Search(stream, "practical joke", A_TITLE);
      // ...
   }

   private InputStream streamOn(String pageContent) {
      return new ByteArrayInputStream(pageContent.getBytes());
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 11.6 테스트 냄새: 다수의 선언

때로는 단일 테스트에 다수의 단언이 필요하기는 하지만, 일반적으로 여러 개의 단언이 있다는 것은 테스트 케이스를 두 개 포함하고 있다는 증거가 된다. 첫번 째 사례는 입력에 대해 검색 결과를 찾는 것이고, 두 번째는 매칭되는 것이 없는 경우이다. 테스트를 두 개로 분할한다면 각각을 좀 더 간결하게 테스트 맥락에 맞도록 나타낼 수 있다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
public class SearchTest {
   private static final String A_TITLE = "1";

   @Test
   public void returnsMatchesShowingContextWhenSearchStringInContent() 
         throws IOException {
      InputStream stream = 
            streamOn("There are certain queer times and occasions "
            + "in this strange mixed affair we call life when a man "
            + "takes this whole universe for a vast practical joke, "
            + "though the wit thereof he but dimly discerns, and more "
            + "than suspects that the joke is at nobody's expense but "
            + "his own.");
      // search
      Search search = new Search(stream, "practical joke", A_TITLE);
      Search.LOGGER.setLevel(Level.OFF);
      search.setSurroundingCharacterCount(10);
      search.execute();
      assertFalse(search.errored());
      assertThat(search.getMatches(), containsMatches(new Match[]
         { new Match(A_TITLE, "practical joke", 
                              "or a vast practical joke, though t") }));
      stream.close();
   }

   @Test
   public void noMatchesReturnedWhenSearchStringNotInContent() 
         throws MalformedURLException, IOException {
      URLConnection connection = 
            new URL("http://bit.ly/15sYPA7").openConnection();
      InputStream inputStream = connection.getInputStream();
      Search search = new Search(inputStream, "smelt", A_TITLE);
      search.execute();
      assertTrue(search.getMatches().isEmpty());
      inputStream.close();
   }
   // ...

   private InputStream streamOn(String pageContent) {
      return new ByteArrayInputStream(pageContent.getBytes());
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

별 생각없이 테스트를 둘로 쪼갰다면 두 번째 테스트의 stream.close\(\)가 컴파일 되지 않음을 알 수 있을 것이다. inputStream으로 바꿔주자. 이렇게 리팩토링을 해보면 작은 결함들을 발견할 수 있다.

## 11.7 테스트 냄새 : 테스트와 무관한 세부 사항들

로그를 끄는 문장, 그리고 스트림을 사용한 후에 닫는 문장 이런 것들은 필요하지만 테스트에서는 군더더기가 될 수 있다. 그렇기에 @Before와 @After 메서드를 이용하자. 또한 @After 메서드의 stream.close\(\) 호출의 이점을 누리기 위해서 두 번째 테스트에서 inputStream 지역 변수가 대신 stream 필드를 참조하도록 한다.

또 assertFalse\(search.errored\(\)\) 의 단언을 생각해보자. 이 잘 생각해보면 search.errored\(\)호출 결과가 true가 되는 테스트케이스가 어디일지 생각해보자. 해당 단언을 제거하고 추후 테스트를 추가하기 위해 메모해두고 넘어가자.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
public class SearchTest {
   private static final String A_TITLE = "1";
   private InputStream stream;
   
   @Before
   public void turnOffLogging() {
      Search.LOGGER.setLevel(Level.OFF);
   }
   
   @After
   public void closeResources() throws IOException {
      stream.close();
   }

   @Test
   public void returnsMatchesShowingContextWhenSearchStringInContent() {
      stream = streamOn("There are certain queer times and occasions "
            + "in this strange mixed affair we call life when a man "
            + "takes this whole universe for a vast practical joke, "
            + "though the wit thereof he but dimly discerns, and more "
            + "than suspects that the joke is at nobody's expense but "
            + "his own.");
      Search search = new Search(stream, "practical joke", A_TITLE);
      search.setSurroundingCharacterCount(10);
      search.execute();
      assertThat(search.getMatches(), containsMatches(new Match[]
         { new Match(A_TITLE, "practical joke", 
                              "or a vast practical joke, though t") }));
   }

   @Test
   public void noMatchesReturnedWhenSearchStringNotInContent() 
         throws MalformedURLException, IOException {
      URLConnection connection = 
            new URL("http://bit.ly/15sYPA7").openConnection();
      stream = connection.getInputStream();
      Search search = new Search(stream, "smelt", A_TITLE);
      search.execute();
      assertTrue(search.getMatches().isEmpty());
   }
   // ...

   private InputStream streamOn(String pageContent) {
      return new ByteArrayInputStream(pageContent.getBytes());
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 11.8 테스트 냄새 : 잘못된 조직

테스트에서 어느 부분들이 준비, 실행, 단언 부분인지 아는 것은 테스트를 빠르게 인지할 수 있게 한다. 따라서 줄바꿈을 이용해 그 의도를 나타낼 수 있다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
public class SearchTest {
   private static final String A_TITLE = "1";
   private InputStream stream;
   
   @Before
   public void turnOffLogging() {
      Search.LOGGER.setLevel(Level.OFF);
   }
   
   @After
   public void closeResources() throws IOException {
      stream.close();
   }

   @Test
   public void returnsMatchesShowingContextWhenSearchStringInContent() {
      stream = streamOn("There are certain queer times and occasions "
            + "in this strange mixed affair we call life when a man "
            + "takes this whole universe for a vast practical joke, "
            + "though the wit thereof he but dimly discerns, and more "
            + "than suspects that the joke is at nobody's expense but "
            + "his own.");
      Search search = new Search(stream, "practical joke", A_TITLE);
      search.setSurroundingCharacterCount(10);

      search.execute();

      assertThat(search.getMatches(), containsMatches(new Match[]
         { new Match(A_TITLE, "practical joke", 
                              "or a vast practical joke, though t") }));
   }

   @Test
   public void noMatchesReturnedWhenSearchStringNotInContent() 
         throws MalformedURLException, IOException {
      URLConnection connection = 
            new URL("http://bit.ly/15sYPA7").openConnection();
      stream = connection.getInputStream();
      Search search = new Search(stream, "smelt", A_TITLE);

      search.execute();

      assertTrue(search.getMatches().isEmpty());
   }

   private InputStream streamOn(String pageContent) {
      return new ByteArrayInputStream(pageContent.getBytes());
   }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 11.9 테스트 냄새 : 암시적 의미

각 테스트는 "왜 그러한 결과를 기대하는가?" 에 대해 대답할 수 있어야 한다. 사람들이 이 테스트 코드의 단언을 보고 왜 이러한 결과가 나오지? 라는 질문에 해답을 얻기 위해 테스트 코드를 이해하려고 많은 시간이 걸리면 안된다. 첫번 째 테스트 코드를 보면 매우 긴 문자열에 대해 partical joke가 있는지 찾기 위한 코드이지만 너무 길어서 이를 읽는 독자는 짜증이 날 수 있다. 좀 더 나은 테스트 데이터로 바꾸자.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
@Test
public void returnsMatchesShowingContextWhenSearchStringInContent() {
      stream = streamOn("rest of text here"
            + "1234567890search term1234567890"
            + "more rest of text");
      Search search = new Search(stream, "search term", A_TITLE);
      search.setSurroundingCharacterCount(10);

      search.execute();

      assertThat(search.getMatches(), containsMatches(new Match[]
         { new Match(A_TITLE, 
                    "search term", 
                    "1234567890search term1234567890") }));
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

두 번째 테스트도 살펴보자. 이 테스트는 실제 URL의 입력 스트림을 사용하기 때문에 느리다. 그래서 이 또한 다른 단위 테스트로 변경할 것이다. 그리고 stream 필드에 조금은 임의의 텍스트를 넣어 초기화한다. 테스트의 의도를 이해하기 쉽도록 "text that doesn't match"로 검색한다.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
@Test
public void noMatchesReturnedWhenSearchStringNotInContent() {
      stream = streamOn("any text");
      Search search = new Search(stream, "text that doesn't match", A_TITLE);

      search.execute();

      assertTrue(search.getMatches().isEmpty());
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 11.10 새로운 테스트 추가

처음 지저분한 테스트를 깔끔하게 테스트 두 개로 분할하면서 이제 추가하기로 했던 테스트들을 만들자. 먼저 검색할 때 errored\(\) 질의에 true가 반환되는 테스트를 작성하자.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
@Test
public void returnsErroredWhenUnableToReadStream() {
      stream = createStreamThrowingErrorWhenRead();
      Search search = new Search(stream, "", "");

      search.execute();
      
      assertTrue(search.errored());
}

private InputStream createStreamThrowingErrorWhenRead() {
      return new InputStream() {
         @Override
         public int read() throws IOException {
            throw new IOException();
         }
      };
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

반대 테스트도 추가하자.

{% code-tabs %}
{% code-tabs-item title="SearchTest.java" %}
```java
@Test
public void erroredReturnsFalseWhenReadSucceeds() {
      stream = streamOn("");
      Search search = new Search(stream, "", "");
      
      search.execute();
      
      assertFalse(search.errored());
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

 

