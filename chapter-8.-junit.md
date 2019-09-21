# 깔끔한 코드로 리팩토링하기

```java
class Profile{
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
}
```
-   리팩토링에서 클래스 , 메서드 , 변수의 **네이밍** 이 중요하다

### 메서드 추출
-   메서드의 복잡도를 줄여 코드가 무엇을 담당하는지 쉽게 이해하도록 구현해라

```java

class Profile{
 public boolean matches(Criteria criteria) {
      score = 0;
      
      boolean kill = false;
      boolean anyMatches = false;
      for (Criterion criterion: criteria) {   
         Answer answer = answers.get(
               criterion.getAnswer().getQuestionText()); 
         
         boolean match = matches(criterion,answer);
              
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
   private boolean matches(Criteria criterion, Answer answer){
      return  criterion.getWeight() == Weight.DontCare || answer.match(criterion.getAnswer());
   }
}
```
-  저수준 메소드로 나누었기 때문에 구체적인 구현사항을 보지 않아도 네이밍만으로 유추가능
-   하지만 문제점
    -   메소드로 뺀 메소드에 문제가 있다.
    -   Criteria의 weigth메소드가 변경이 되면 Profile이 변경이된다.
    -   즉 변경에 대해 유연하지 못하다.
    

```java


class Criterion implements Scoreable{
    public boolean matches(Answer answer){
        return getWeight() == Weight.DontCare || answer.match(getAnswer);     
    }
}

class Profile{
 public boolean matches(Criteria criteria) {
      score = 0;
      
      boolean kill = false;
      boolean anyMatches = false;
      for (Criterion criterion: criteria) {   
          # 요기도 문제
          # 디메테르의 법칙을을 위반
         Answer answer = answers.get(
               criterion.getAnswer().getQuestionText()); 
         
         boolean match = criterion.matches(criterion,answer);
              
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
   private Answer answerMatching(Criterion criterion){
     return answers.get(criterion.getAnswer().getQuestionText());
   }
}
```
-   디메테르 법칙
    -   다른 객체로 전파되는 연쇄적인 메서드 호출을 피해야한다
    - 디메테르 원칙 규칙
    객체 O의 메소드 m은 다음의 객체들의 타입의 메소드만 호출해야 한다는 법칙이다.
    1. O 객체 자신의 메소드들. (O itself)
    2. m의 파라미터로 넘어온 객체들의 메소드들.(M's parameters)
    3. m 안에서 생성 되거나 초기화된 객체의 메소드들.(Any objects created/instantiated within M)
    4. O객체의 직접 소유하는 객체의 메소드들.(O's direct component objects)  
    5.  O객체의 m에서 접근이 가능한 전역변수의 메소드들.(A global variable, accessible by O, in the scope of M)
