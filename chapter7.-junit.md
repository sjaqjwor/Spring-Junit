# Chapter 7. 경계조건 : Correct 기법

-   Conformance(준수) : 값이 기대한 양식을 준수하고 있는가?
-   Ordering(순서) : 값의 집합이 적절하게 정렬되거나 정렬되지 않았나?
-   Range(범위) : 이성적인 최솟값과 최댓값 안에 있는가?
-   Reference(참조) : 코드 자체에서 통제할 수 없는 어떤 외부 참조를 초함하고 있는가?
-   Existence(존재) : 값이 존재하는가(널이 아니거나 , 0이 아니거나 ,집합에 존재하는가?)
-   Cardinality(기수) : 정확히 충분한 값들이 있는가?
-   Range(범위)

### 준수
테스트를 통해 검증 할 데이터가 형식에 맞는 데이터 인지 확인이 필요합니다.<br>
ex).
- 기대하는 값이 이메일이라면 name@test.co.kr

### 순서
데이터 순서 혹은 커다란 컬렉션에 있는 데이터 한 조각의 위치는 코드가 쉽게 잘못 될 수<br>
있는 부분이므로 주의해야합니다.

```
// CORECT : Ordering(순서)

public class ProfilePoolTest {
    private ProfilePool pool;
    private Profile langrsoft;
    private Profile smeltInc;
    private BooleanQuestion doTheyReimburseTuition;

    @Before
    public void create() {
        pool = new ProfilePool();
        langrsoft = new Profile("Langrsoft");
        smeltInc = new Profile("Smelt Inc.");
        doTheyReimburseTuition = new BooleanQuestion(1, "Reimburses tuition?"); // 수업료를 상환합니까?
    }

    @Test
    public void scoresProfileInPool() {
        langrsoft.add(new Answer(doTheyReimburseTuition, Bool.TRUE));
        pool.add(langrsoft);

        pool.score(soleNeed(doTheyReimburseTuition, Bool.TRUE, Weight.Important));

        assertThat(langrsoft.score(), equalTo(Weight.Important.getValue()));
    }

    private Criteria soleNeed(Question question, int value, Weight weight) {
        Criteria criteria = new Criteria();
        criteria.add(new Criterion(new Answer(question, value), weight));
        return criteria;
    }


    @Ignore
    @Test
    public void answersResultsInScoredOrder() {
        smeltInc.add(new Answer(doTheyReimburseTuition, Bool.FALSE));
        pool.add(smeltInc);
        langrsoft.add(new Answer(doTheyReimburseTuition, Bool.TRUE));
        pool.add(langrsoft);

        pool.score(soleNeed(doTheyReimburseTuition, Bool.TRUE, Weight.Important));
        List<Profile> ranked = pool.ranked();

        assertThat(ranked.toArray(), equalTo(new Profile[]{ langrsoft,smeltInc }));
    }
}

```


```
public class ProfilePool {
    private List<Profile> profiles = new ArrayList<Profile>();

    public void add(Profile profile) {
        profiles.add(profile);
    }

    public void score(Criteria criteria) {
        for (Profile profile: profiles)
            profile.matches(criteria);
    }

    public List<Profile> ranked() {
        Collections.sort(profiles,
                (p1, p2) -> ((Integer)p1.score()).compareTo(p2.score()));
        return profiles;
    }
}
```

ranked() 메서드에 compareTo 메서드에서 p1 , p2의 위치에 따라 테스트 코드의 결과가
달라지는 것을 확인 할 수 있다.



### 범위
원은 360도 입니다.
원에 각도에는 음수가 안닌 0도 ~ 360도 이라는 범위 제약이 있습니다.

아래에의 코드를 통해 원에 대한 테스트 코드를(범위 제약) 봐보겠습니다.

#### BearingTest

```
import main.ch05.iloveyouboss.domain.BearingOutOfRangeException;
import main.ch07.scratch.Bearing;
import org.junit.*;
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;

public class BearingTest {
    //@Test(expected=BearingOutOfRangeException.class)
    @Test(expected=NullPointerException.class)
    public void throwsOnNegativeNumber() {
        new Bearing(-1);
    }

    @Test(expected=BearingOutOfRangeException.class)
    public void throwsWhenBearingTooLarge() {
        new Bearing(Bearing.MAX + 1);
    }

    @Test
    public void answersValidBearing() {
        assertThat(new Bearing(Bearing.MAX).value(), equalTo(Bearing.MAX));
    }

    @Test
    public void answersAngleBetweenItAndAnotherBearing() {
        assertThat(new Bearing(15).angleBetween(new Bearing(12)), equalTo(3));
    }

    @Test
    public void angleBetweenIsNegativeWhenThisBearingSmaller() {
        assertThat(new Bearing(12).angleBetween(new Bearing(15)), equalTo(-3));
    }
}

```

### @After 메서드를 통해 테스트가 완료되었을 때마다 제약 사항 불변식 구현 

#### RectangleTest
```

import main.ch07.scratch.Point;
import main.ch07.scratch.Rectangle;
import org.junit.After;
import org.junit.Ignore;
import org.junit.Test;

import static org.hamcrest.CoreMatchers.equalTo;
import static org.junit.Assert.assertThat;
import static test.ch07.scratch.ConstrainsSidesTo.constrainsSidesTo;


public class RectangleTest {
    private Rectangle rectangle;

    @Test
    public void answersArea() {
        rectangle = new Rectangle(new Point(5, 5), new Point (15, 10));
        assertThat(rectangle.area(), equalTo(50));
    }

    @Ignore
    @ExpectToFail
    @Test
    public void allowsDynamicallyChangingSize() {
        rectangle = new Rectangle(new Point(5, 5));
        rectangle.setOppositeCorner(new Point(130, 130));
        assertThat(rectangle.area(), equalTo(15625));
    }
    
    @After
    public void ensureInvariant() {
        assertThat(rectangle, constrainsSidesTo(100));
    }
        
}

```

### 불변 메서드를 내장하여 범위 테스트 

#### SparseArray
```
public class SparseArray<T> {
    public static final int INITIAL_SIZE = 1000;
    private int[] keys = new int[INITIAL_SIZE];
    private Object[] values = new Object[INITIAL_SIZE];
    private int size = 0;

    public void put(int key, T value) {
        if (value == null) return;

        int index = binarySearch(key, keys, size);
        if (index != -1 && keys[index] == key)
            values[index] = value;
        else
            insertAfter(key, value, index);
    }

 .......

    public void checkInvariants() throws util.InvariantException {
        long nonNullValues = Arrays.stream(values).filter(Objects::nonNull).count();
        if (nonNullValues != size)
            throw new util.InvariantException("size " + size +
                    " does not match value count of " + nonNullValues);
    }

 .......
   
}
```

#### SparseArrayTest
```
public class  SparseArrayTest{
    SparseArray<Object> array = null;

     @Before
     public void create(){
         array = new SparseArray<>();
     }
    
    
    @ExpectToFail
    @Ignore
    @Test
    public void handlesInsertionInDescendingOrder() {
        array.put(7, "seven");
        array.checkInvariants();
        array.put(6, "six");
        array.checkInvariants();
        assertThat(array.get(6), equalTo("six"));
        assertThat(array.get(7), equalTo("seven"));
    }

}
```

### Reference(참조)
메서드를 테스트할 때 고려해야할 점<br>
  - 범위를 넘어서는 것을 참조하고 있지 않은지
  - 외부 의존성은 무엇인지
  - 특정 상태에 있는 객체를 의존하고 있는지 여부
  - 반드시 존재해야 하는 그 외 다른 조건들
 
어떤 상태에 대해 가정할 때는 그 가정이 맞지 않으면 코드가 합리적으로 잘 동작하는지 검사해야 한다.

예를 들자면..!<br>
   - 고객의 계정 히스토리를 표시하는 웹은 고객이 먼저 로그인해야 정상 작동한다.
   - 스택의 pop() 메서드를 호출할 때는 스택이 비어 있으면 안된다.
   - 차량의 변속기를 주행에서 주차로 변경할 때는 먼저 차를 멈추어야 한다.
 

#### TransmissionTest
```

import main.ch07.transmission.Car;
import main.ch07.transmission.Gear;
import main.ch07.transmission.Transmission;
import org.junit.Before;
import org.junit.Test;
import static org.hamcrest.CoreMatchers.*;
import static org.hamcrest.MatcherAssert.assertThat;

public class TransmissionTest {
    private Transmission transmission;
    private Car car;

    @Before
    public void create(){
        car = new Car();
        transmission = new Transmission(car);
    }

    @Test
    public void 시동을키고_기어가_DRIVE인지() {
        transmission.shift(Gear.DRIVE);
        car.accelerateTo(35);
        assertThat(transmission.getGear(),equalTo(Gear.DRIVE));
    }

    @Test
    public void 차가정지하지않고_기어를_PARK으로_변경하는경우(){
        transmission.shift(Gear.DRIVE);
        car.accelerateTo(80);

        transmission.shift(Gear.PARK);
        assertThat(transmission.getGear(),equalTo(Gear.DRIVE));
    }

    @Test
    public void 차를_정지하고자_기어를_PARK으로_변경하는경우(){
        transmission.shift(Gear.DRIVE);
        car.accelerateTo(80);

        car.brakeToStop();
        transmission.shift(Gear.PARK);
        assertThat(transmission.getGear(),equalTo(Gear.PARK));
    }

    @Test
    public void 차를_정지하고자_기어를_PARK으로_변경하는경우_기어가_DRIVE면(){
        transmission.shift(Gear.DRIVE);
        car.accelerateTo(80);

        car.brakeToStop();
        transmission.shift(Gear.PARK);
        assertThat(transmission.getGear(),equalTo(Gear.DRIVE));
    }

}

```

### Existence(존재)
   - 어떤 인자를 허용하거나 필드를 유지하는 메서드에 대해 그 값이 null , 0 혹은
        비어 있는 경우라면 어떤 일이 일어날지 생각해 보자. 

### Cardinality(기수)
   - 0-1-N 법칙

- 0인 경우
- 어떤 것이 1인경우
- 어떤 것의 컬렉션을 다루는 경우

ex) 팬케이크 가게에서 상위 10(N)개의 음식 목록을 유지해야 한다면?

* 목록에 항목이 하나도 없을 때
* 목록에 항목이 하나도 있을 때
* 목록에 항목이 하나도 없을 때 한 항목 추가하기
* 목록에 항목이 하나만 있을 때 한 항목 추가하기
* 목록에 항목이 아직 N가 미만일때 한 항목 추가하기
* 목록에 항목이 아직 N가 있을 때 한 항목 추가하기

0 ,1 과 다수로 설계를 단순화 한다면 나중에 상위의 개수에 대한 비즈니스 로직이<br>
변경되더라도 약간의 코드 추가로 처리가 된다.

[약간의 코드 추가]
```
public static final int MAX = N;
```

        
### Time(시간)
   -  상대적 시간(시간 순서)
        - 메소드의 호출 순서 ex) . open() -> read() -> close() 
   -  절대적 시간(측정된 시간)
        - 기준에 따라 달라지는 시간의 계산 
   -  동시성 문제들


