<<<<<<< HEAD
##Junit 단언
-   테스트에 넣을 수 있는 정적 메소드
-   조건이 참인지 검증하는 방법

```
@Test
public void 나는_바보인가(){
    철진.addStatus("바보");
   
=======
# Chapter 3. 단언

* 테스트에 넣을 수 있는 정적 메소드
* 조건이 참인지 검증하는 방법

```text
@Test
public void 나는_바보인가(){
    철진.addStatus("바보");

>>>>>>> upstream/master
    ## 해당부분이 단언이다.
    assertTrue(철진.is바보());
}

@Test
public void 키가_180보다_큰가(){
    int 키 = 180;
    승기.addStatus(190);
<<<<<<< HEAD
    
=======

>>>>>>> upstream/master
    ## 해당 부분이 단언이다.
    assertTrue(승기.get키() > 키)
}
```

<<<<<<< HEAD
#### assertThat
```
=======
### assertThat

```text
>>>>>>> upstream/master
assertThat({actual},{match})

## 예제
assertThat(승기.getKey(),equalTo(187))
승기의 키는 187과 같아야한다.
결과 true
```
<<<<<<< HEAD
##### {actual}
-   검증하고자 하는 값
-   실제 값

##### {match}
-   검증하고자 하는 값과 비교하는 값
-   equalTo에는 자바 인스턴스 및 기본형 값을 다 넣을 수 있다.
-   다양한 단언 {match}가 존재한다.
    -   is(boolean)
    -   startWith 등등
    

#### 다양한 햄크래스트 매처
-   CoreMathcers 클래스가 매처 모음을 제공
-   다양한 햄크레스트 매처를 도입할 수록 테스트고드의 표현력이 깊어진다.
##### 햄크래스트 매처의 역할들
-   객체타입을 검사
-   객체의 참조가 같은 인스턴스인지 검사
-   다수의 매처를 결합하여 둘다 혹은 둘 중에 어떤 것이든 성공하는지 검사
-   컬랙션이 요소를 포함하거나 조건에 부합하는지 검사
-   컬랙션이 아이템 몇개를 모두 포함하는지 검사
-   컬랙션에 있는 모든 요소가 매처를 준수하는지
-   더 다양하니 API를보고 사용해라




=======

#### {actual}

* 검증하고자 하는 값
* 실제 값

#### {match}

* 검증하고자 하는 값과 비교하는 값
* equalTo에는 자바 인스턴스 및 기본형 값을 다 넣을 수 있다.
* 다양한 단언 {match}가 존재한다.
  * is\(boolean\)
  * startWith 등등

### 다양한 햄크래스트 매처

* CoreMathcers 클래스가 매처 모음을 제공
* 다양한 햄크레스트 매처를 도입할 수록 테스트고드의 표현력이 깊어진다.

  **햄크래스트 매처의 역할들**

* 객체타입을 검사
* 객체의 참조가 같은 인스턴스인지 검사
* 다수의 매처를 결합하여 둘다 혹은 둘 중에 어떤 것이든 성공하는지 검사
* 컬랙션이 요소를 포함하거나 조건에 부합하는지 검사
* 컬랙션이 아이템 몇개를 모두 포함하는지 검사
* 컬랙션에 있는 모든 요소가 매처를 준수하는지
* 더 다양하니 API를보고 사용해라

  **컴퓨터는 부동 소수점 수를 표현할 수 없다.**

* 즉 test코드로 비교하고 싶다면 두수가 벌어질 수 잇ㄴ느 공차 또는 허용 오차를 지정해야한다.
* 햄크래스트는 isCloseTo매처를 사용

  \`\`\`

  assertThat\(2.32\*3,closeTo\(6.96,0.0005\)\)

## 0.0005가 오차

```text
#### 단언 설명
-   단언에 설명을 추가 할 수 있다.
-   해당 메시지는 결과가 false일 때 출력된다.즉 설명이다.
```

assertThat\({message},{actual},{match}\)

```text
-   메시지를 출력할 수 있어 좋으나 차라리 테스트 코드를 한번에 이해 할 수 있게 작성하는게 더 좋은 방법이다.

#### 예외 처리 방법
1.  어노테이션 사용
```java
 @Test(expected = NullPointerException.class)
    public void test(){
        Exception exception = new Exception();
        exception.getName().substring(3);
    }
    ## NullPoint가 발생해야 test 통과
```

1. try/catch와 fail\(구시대적 유물\)

   ```java
   @Test
    public void test1(){
        try{
            Exception exception = new Exception();
            exception.getName().substring(3);
        }catch (NullPointerException e){
            System.out.println("null point exception");
        }
    }
   ```

2. ExpectedException\(신 문물\)

   ```java
   ## 기본적인 ExpectedException 반환
   @Rule
   public ExpectedException thrown = ExpectedException.none();

   @Test
   public void test2(){
       thrown.expect(NullPointerException.class);
       thrown.expectMessage("null point exception");

       Exception exception = new Exception();
       exception.getName();
   }
   ```

3. rule
   * 하나의 테스트 클래스 내에서 동작방식을 재정의하거나 추가하기 위해서 사용한다.
4. 람다를 위한 fishbowl이 있다.
   * maven에 의존성을 추가하고 사용해야한다.

     [http://stefanbirkner.github.io/fishbowl/download.html](http://stefanbirkner.github.io/fishbowl/download.html)
>>>>>>> upstream/master

