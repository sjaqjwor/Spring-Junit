# Chapter 10. 목 객체 사용

## stub과 mock을 이용한 외부 환경과의 최소화

1. 외부 환경과 의존성을 가진 예시코드 문제점 파악 
2. 문제점 코드를 테스트하기 위한 stub의 이해와 사용
3. stub에 인자 검증 기능 추가
4. Mockito의 mock 사용하기
5. Mockito의 DI\(의존성 주입\) 사용하여 테스트 하기
6. mock을 안전하게 사용하기 위한 확인사항 몇가지

### 1. 외부 환경과 의존성을 가진 예시코드 문제점 파악

* Http 인터페이스

  ```java
  public interface Http {
    String get(String url) throws IOException;
  }
  ```

* Http 인터페이스를 구현한 클래스

  ```java
  public class HttpImpl implements Http {
    public String get(String url) throws IOException {
        CloseableHttpClient client = HttpClients.createDefault();
        HttpGet request = new HttpGet(url);
        CloseableHttpResponse response = client.execute(request);
        try {
            HttpEntity entity = response.getEntity();
            return EntityUtils.toString(entity);
        } finally {
            response.close();
        }
    }
  }
  ```

* HttpImpl 클래스를 사용하는 문제 예시코드

  ```java
  public class AddressRetriever {
    public Address retrieve(double latitude, double longitude) throws IOException, ParseException {
        // 1. http 호출 준비
        String parms = String.format("lat=%.6f&lon=%.6f", latitude, longitude);

        // http.get() 메소드를 사용하여 http request 요청 수행
        String response = new HttpImpl().get(
          "http://open.mapquestapi.com/nominatim/v1/revers?format=json&" + parms
        );

        // 2. address 객체 생성 로직
        JSONObject obj = (JSONObject)new JSONParser().parse(response);

        JSONObject address = (JSONObject)obj.get("address");
        String country = (String)address.get("country_code");
        if (!country.equals("us"))
            throw new UnsupportedOperationException(
                    "cannot support non-US addresses at this time");

        String houseNumber = (String)address.get("house_number");
        String road = (String)address.get("road");
        String city = (String)address.get("city");
        String state = (String)address.get("state");
        String zip = (String)address.get("postcode");
        return new Address(houseNumber, road, city, state, zip);
    }
  }
  ```

  위 코드의 문제점은 다음과 같다.

* http request 요청으로 외부 환경을 호출하여 테스트 속도 감소
* http request를 항상 신뢰할 수 있을지 장담하지 못함 \(우리의 통제 밖에 있음\)

  > 1. 이번 목차에서는 http request가 항상 신뢰할 수 있는 리턴값을 준다고 생각하고 테스트 진행  
  > 2. 우리의 테스트 코드 목적은 http api 테스트가 아닌 로직 테스트 이기 때문에, 위 메소드의 로직만 테스트 한다
  >    1. http 호출 준비 \(인자 검증\)
  >    2. address 객체 생성 \(로직 검증\)

### 2. 문제점 코드를 테스트하기 위한 stub의 이해와 사용

> stub 이란?  
> 테스트 용도로 사용하기 위해 하드 코딩한 값을 반환하는 구현체

http request의 리턴 값은 항상 신뢰할 수 있다고 판단하고,  
해당 리턴 json 값을 stub으로 만들어서 로직만 테스트 해보자

* retrieve\(\) 테스트 메소드 작성

  ```java
  public class AddressRetrieverTest {
    @Test
    public void answersAppropriateAddressForValidCoordinates() throws IOException, ParseException {
        // Http 인터페이스 인스턴스를 stub(하드코딩 값)으로 생성
        Http http = (String url) -> "{\"address\":{" +
                        "\"house_number\":\"324\", " +
                        "\"road\":\"North Tejon Street\", " +
                        "\"city\":\"Colorado Springs\", " +
                        "\"state\":\"Colorado\", " +
                        "\"postcode\":\"80903\"," +
                        "\"country_code\":\"us\"}" +
                        "}";
        AddressRetriever retriever = new AddressRetriever(http); // 인스턴스를 인수로 넘김

        Address address = retriever.retrieve(38.0, -104.0);

        assertThat(address.houseNumber, equalTo("324"));
        assertThat(address.road, equalTo("North Tejon Street"));
        assertThat(address.city, equalTo("Colorado Springs"));
        assertThat(address.state, equalTo("Colorado"));
        assertThat(address.zip, equalTo("80903"));
    }
  }
  ```

* 테스트 코드에 맞추어 AddressRetriever 클래스 내부 로직 변경

  ```java
  public class AddressRetriever {
    private Http http;

    public AddressRetriever(Http http) {
        this.http = http;    
    }

    public Address retrieve(double latitude, double longitude) throws IOException, ParseException {
        String parms = String.format("lat=%.6f&lon=%.6f", latitude, longitude);
        String response = http.get(
          "http://open.mapquestapi.com/nominatim/v1/revers?format=json&" + parms
        );

        JSONObject obj = (JSONObject)new JSONParser().parse(response);

        JSONObject address = (JSONObject)obj.get("address");
        String country = (String)address.get("country_code");
        if (!country.equals("us"))
            throw new UnsupportedOperationException(
                    "cannot support non-US addresses at this time");

        String houseNumber = (String)address.get("house_number");
        String road = (String)address.get("road");
        String city = (String)address.get("city");
        String state = (String)address.get("state");
        String zip = (String)address.get("postcode");
        return new Address(houseNumber, road, city, state, zip);
    }
  }
  ```

위 코드에서 stub을 사용하여 테스트 코드를 작성한 결과 1. http request를 하지 않아도 되기 때문에 Address 객체 생성 로직만 검증 가능. 2. 기존 retrieve\(\) 메소드는 http request를 하기 위해 HttpImpl\(\).get\(\) 과 같은 코드 사용\(강한 의존도\)  
테스트 코드를 짜면서 http 구현체\(stub\)을 생성자 인수로 보내기 위해 내부 로직 변경\(느슨한 의존도\)

### 3. stub에 인자 검증 기능 추가

위 테스트 코드를 약간만 수정하여, 멍청한 stub에 약간 지능을 추가할 수 있다. \(인자 검증\)

```java
public class AddressRetrieverTest {
    @Test
    public void answersAppropriateAddressForValidCoordinates() throws IOException, ParseException {
        // http 인스턴스 구현체(stub)에 인자를 검증하는 코드 추가 
        Http http = (String url) -> {
            if(!url.contains("lat=38.000000f&lon=-104.000000")) {
                fail("url " + url + " does not contain correct params");
            }

           return "{\"address\":{" +
                    "\"house_number\":\"324\", " +
                    "\"road\":\"North Tejon Street\", " +
                    "\"city\":\"Colorado Springs\", " +
                    "\"state\":\"Colorado\", " +
                    "\"postcode\":\"80903\"," +
                    "\"country_code\":\"us\"}" +
                    "}";
        };
        AddressRetriever retriever = new AddressRetriever(http);

        Address address = retriever.retrieve(38.0, -104.0);

        assertThat(address.houseNumber, equalTo("324"));
        assertThat(address.road, equalTo("North Tejon Street"));
        assertThat(address.city, equalTo("Colorado Springs"));
        assertThat(address.state, equalTo("Colorado"));
        assertThat(address.zip, equalTo("80903"));
    }
}
```

### 4. Mockito의 mock 사용하기

> mock 이란?  
> 1. 의도적으로 흉내 낸 동작을 제공하고 \(모의 객체라고도 하며 가짜 객체를 의미\) 2. 수신한 인자가 모두 정상인지 여부를 검증하는 테스트 구조물

```java
public class AddressRetrieverTest {
    @Test
    public void answersAppropriateAddressForValidCoordinates() throws IOException, ParseException {
        Http http = mock(Http.class);                               // 1. mock 생성
        when(http.get(contains("lat=38.000000&lon=-104.000000"))).  // 2. 테스트 기대사항 정의
                thenReturn("{\"address\":{" +                       // 3. 기대 사항 충족 시, 리턴 값
                        "\"house_number\":\"324\", " +
                        "\"road\":\"North Tejon Street\", " +
                        "\"city\":\"Colorado Springs\", " +
                        "\"state\":\"Colorado\", " +
                        "\"postcode\":\"80903\"," +
                        "\"country_code\":\"us\"}" +
                        "}");
        AddressRetriever retriever = new AddressRetriever(http);

        Address address = retriever.retrieve(38.0, -104.0);

        assertThat(address.houseNumber, equalTo("324"));
        assertThat(address.road, equalTo("North Tejon Street"));
        assertThat(address.city, equalTo("Colorado Springs"));
        assertThat(address.state, equalTo("Colorado"));
        assertThat(address.zip, equalTo("80903"));
    }
}
```

1. 모의 객체를 생성. 또는 목 인스턴스 합성이라고도 말함.
2. when\(\) 정적 메서드로 테스트 기대 사항을 정의. \(인자 검증\)
3. 인자 검증 완료 시, 지정된 하드코딩 json값 리턴

### 5. Mockito의 DI\(의존성 주입\) 사용하여 테스트 하기

Mockito에서 제공하는 DI를 사용하여 내부 로직을 변경하지 않고도 목 객체를 대상 클래스로 넘길 수 있다.

```java
public class AddressRetrieverTest {
    // 1. 합성할 mock 인스턴스 정의
    @Mock
    private Http http;

    // 2. mock 인스턴스를 주입할 인스턴스 변수 정의
    @InjectMocks
    private AddressRetriever retriever;

    // 3. mock 객체 주입
    @Before
    public void createRetriever() {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void answersAppropriateAddressForValidCoordinates() throws IOException, ParseException {
        when(http.get(contains("lat=38.000000&lon=-104.000000"))).
                thenReturn("{\"address\":{" +
                        "\"house_number\":\"324\", " +
                        "\"road\":\"North Tejon Street\", " +
                        "\"city\":\"Colorado Springs\", " +
                        "\"state\":\"Colorado\", " +
                        "\"postcode\":\"80903\"," +
                        "\"country_code\":\"us\"}" +
                        "}");

        Address address = retriever.retrieve(38.0, -104.0);

        assertThat(address.houseNumber, equalTo("324"));
        assertThat(address.road, equalTo("North Tejon Street"));
        assertThat(address.city, equalTo("Colorado Springs"));
        assertThat(address.state, equalTo("Colorado"));
        assertThat(address.zip, equalTo("80903"));
    }
}
```

* MockitoAnnotations.initMocks\(this\)
  * @Mock 애너테이션이 붙은 필드를 생성, 합성하고 -&gt; mock\(Http.class\)
  * @InjectMocks 애너테이션이 붙은 객체에 주입한다. \(주입 시, 아래 3단계를 거친다.\)
    1. 해당 클래스에서 적절한 생성자를 탐색
    2. 적절한 생성자가 없다면, 적절한 세터 탐색
    3. 적절한 세터가 없다면, 적절한 필드 탐색 

```java
public class AddressRetriever {
    private Http http = new HttpImpl();

    public Address retrieve(double latitude, double longitude) throws IOException, ParseException {
        String parms = String.format("lat=%.6f&lon=%.6f", latitude, longitude);
        String response = http.get(
          "http://open.mapquestapi.com/nominatim/v1/revers?format=json&" + parms
        );

        JSONObject obj = (JSONObject)new JSONParser().parse(response);

        JSONObject address = (JSONObject)obj.get("address");
        String country = (String)address.get("country_code");
        if (!country.equals("us"))
            throw new UnsupportedOperationException(
                    "cannot support non-US addresses at this time");

        String houseNumber = (String)address.get("house_number");
        String road = (String)address.get("road");
        String city = (String)address.get("city");
        String state = (String)address.get("state");
        String zip = (String)address.get("postcode");
        return new Address(houseNumber, road, city, state, zip);
    }
}
```

* Mockito의 di를 이용하면, 위 코드와 같이 생성자 코드를 제거할 수 있다.  

  즉, 처음 만들었던 코드에서 변경을 최소화 할 수 있다. \(아래와 같은 작업이 필요 없음\) 

* 처음 만든 코드에서 테스트를 위해 생성자 코드 추가
* 생성자로 테스트 코드를 위해 만든 http 객체 주입

### 6. mock을 안전하게 사용하기 위한 확인사항 몇가지

* mock은 실제 동작을 대신한다고 가정하고 테스트 코드를 구현하는 것이다. 즉, mock을 안전하게 사용하고 있는지 자신에게 몇 가지 질문을 해야 한다.
* mock이 실제 운영코드의 동작을 올바르게 묘사하는가?
* 운영코드가 생각지도 못한 다른 형식의 반환값을 반환하는가?
* 운영코드는 예외를 던지는가?
* 운영코드는 null을 반환하는가? 

> 마지막으로, stub과 mock을 사용한 각 기능의 단위테스트 만으로는 오류가 나지 않았어도,  
> 전체 통합 테스트 시에는 오류가 생길 수 있으므로 통합테스트를 작성이 꼭 필요하다.  
> [통합테스트란?](https://terms.naver.com/entry.nhn?docId=3533038&cid=58528&categoryId=58528&expCategoryId=58528)

