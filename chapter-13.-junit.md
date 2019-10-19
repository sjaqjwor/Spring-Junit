# chapter 12 까다로운 테스트

### 멀티스레드 테스트하는 방법

**스레드 통제와 애플리케이션 코드 사잉의 중첩을 최소화**
  - 즉 일반적인 비지니스 로직과 멀티스레드를 수행하는 로직을 분해하여 설계를 하여라
  
**다른 사람의 코드를 믿어라**


```java
// 분해전
public class ProfileMatcher{

    public void findMatchingProfiles(Criteria criteria, MatchListener listener){
         ExecutorService executor = 
                    Executors.newFixedThreadPool(DEFAULT_POOL_SIZE);
        
              List<MatchSet> matchSets = profiles.values().stream()
                    .map(profile -> profile.getMatchSet(criteria)) 
                    .collect(Collectors.toList());
              for (MatchSet set: matchSets) {
                 Runnable runnable = () -> {
                    if (set.matches())
                       listener.foundMatch(profiles.get(set.getProfileId()), set);
                 };
                 executor.execute(runnable);
              }
              executor.shutdown();
    }
}

// 분헤후
public class ProfileMatcher {
   public void findMatchingProfiles(
         Criteria criteria, MatchListener listener) {
      ExecutorService executor = 
            Executors.newFixedThreadPool(DEFAULT_POOL_SIZE);
      for (MatchSet set: collectMatchSets(criteria)) {
         Runnable runnable = () -> {
            if (set.matches())
               listener.foundMatch(profiles.get(set.getProfileId()), set);
         };
         executor.execute(runnable);
      }
      executor.shutdown();
   }

   List<MatchSet> collectMatchSets(Criteria criteria) {
      List<MatchSet> matchSets = profiles.values().stream()
            .map(profile -> profile.getMatchSet(criteria))
            .collect(Collectors.toList());
      return matchSets;
   }
}

// 더분해후
public class ProfileMatcher {
   public void add(Profile profile) {
      profiles.put(profile.getId(), profile);
   }

   public void findMatchingProfiles(
         Criteria criteria, MatchListener listener) {
      ExecutorService executor = 
            Executors.newFixedThreadPool(DEFAULT_POOL_SIZE);

      for (MatchSet set: collectMatchSets(criteria)) {
         Runnable runnable = () -> process(listener, set);
         executor.execute(runnable);
      }
      executor.shutdown();
   }

   void process(MatchListener listener, MatchSet set) {
      if (set.matches())
         listener.foundMatch(profiles.get(set.getProfileId()), set);
   } 

   List<MatchSet> collectMatchSets(Criteria criteria) {
      List<MatchSet> matchSets = profiles.values().stream()
            .map(profile -> profile.getMatchSet(criteria))
            .collect(Collectors.toList());
      return matchSets;
   }
}
```
##### 테스트 코드 보면서 설명

### 데이터 베이스 테스트

```java
 public Map<Integer,String> questionText(List<BooleanAnswer> answers) {
      Map<Integer,String> questions = new HashMap<>();
      answers.stream().forEach(answer -> {
         if (!questions.containsKey(answer.getQuestionId()))
            questions.put(answer.getQuestionId(), 
               controller.find(answer.getQuestionId()).getText()); });
      return questions;
   }
```
- 인메모리 db가 아닌 실제 사용하는 데이터베이솨 테스트를 진행하고자 하면 적절한 데이터가 있어야 한다
  - 문제점
     - 테스트 코드를 진행하면서 데이터가 변하게 되고 데이터에 변동으로 테스트가 망가진다.
     - 그렇기에 테스트안에서 생성하고 데이터를 관리해라

