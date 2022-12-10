# 2장: 동작 파라미터화 코드 전달하기

이 장의 내용
  - 변화하는 요구사항에 대응
  - 동작 파라미터화
  - 익명 클래스
  - 람다 표현식 미리보기
  - 실전 예제 : Comparator, Runnable, GUI

변화하는 요구사항은 소프트웨어 엔지니어링에서 피할 수 없는 문제다. **동작 파라미터화 (behavior parameterization)**을 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다.

**동작 파라미터화란 아직 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다.**

예를 들어 컬렉션을 처리할 때 다음과 같은 메서드를 구현한다고 가정하자:
  - 리스트이 모든 요소에 대해서 '어떤 동작'을 수행할 수 있음
  - 리스트 관력 작업을 끝낸 다음에 '어떤 다른 동작'을 수행할 수 있음
  - 에러가 발생하면 '정해진 어떤 다른 동작'을 수행할 수 있음

동작 파라미터화로 이처럼 다양한 기능을 수행할 수 있다, 우선 이 방법을 코드로 구현하는 방법을 예제를 이용해서 살펴보자.

## 2.1 변화하는 요구사항에 대응하기

기존의 농장 재고목록 애플리케이션에 리스트에서 녹색 사과만 필터링하는 기능을 추가한다고 가정하자.

### 녹색 사과 필터링

```java
  enum Color {
    RED,
    GREEN
  }

  public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();         //사과 누적 리스트
    for (Apple apple : inventory) {
      if (apple.getColor() == Color.GREEN) {        //녹색 사과만 선택
        result.add(apple);
      }
    }
    return result;
  }
```

위에 코드는 녹색사과를 필터링 해서 result 리스트에 추가해준다. 하지만 농부가 변심하여 갑자기 빨간 사과도 필터링하고 싶어졌다고 한다면 어떻게 될까? 가장 간단한 방법은 위 코드를 복사해서 빨간 사과를 필터링 하도록 변경하는 방법이 있을것이다.

하지만 이렇게 반복되는 코드가 존재한다면 가장 좋은 방법은 그 코드를 **추상화** 하는 것이다.

### 색을 파라미터화

그렇다면 filterGreenApples의 코드를 반복 사용하지 않고 filterRedApples를 구현할 수 있는 방법이 있을까? 색을 파라미터화 하면 된다!

```java
  public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getColor() == color) {
        result.add(apple);
      }
    }
    return result;
  }
```

이렇게 파라미터로 색을 전달받도록 구현하면, 메서드를 호출할 때 색상을 지정해 주는게 가능해진다!

```java
  List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
  List<Apple> redApples = filterApplesByColor(inventory, RED);
```

하지만 여기서 농부가 나타나 이제는 색상 말고 무게로도 필터링 하고 싶다고 요청한다면? 여기서 똑똑한 프로그래머라면 색상과 마찬가지로 무게의 요구 기준도 얼마든지 바뀔수 있다는 사실을 눈치챘을 것이다! 

그래서 위에 색상처럼 무게도 파라미터로 추가하여 구현해보자.

```java
  public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getWeight() > weight) {
        result.add(apple);
      }
    }
    return result;
  }
```

이렇게 색상과 같은 방법을 활용하여 문제를 해결하였다. 하지만 이는 소프트웨어 공학의 **DRY (don't repeat yourself)** 원칙을 어기는 것이다. 이 문제를 해결하고자 색과 무게를 filter라는 메서드로 합치는 방법도 있다. 그러면 어떤 기준으로 필터링할지 구분해주는 flag를 추가해야한다. 하지만 이 방법은 실무에서는 절대 사용해서는 안된다.

## 2.2 동작 파라미터화

이 전에서 파라미터를 추가하는 방법이 아닌 변화하는 요구사항에 좀 더 유연하게 대응할 수 있는 방법이 절실하다는 것을 깨달았을 것이다. 그리하여 선택 조건을 결정하는 인터페이스를 정의 해보도록 하자.

```java
  public interface ApplePredicate {
    boolean test(Apple a);
  }
```

위 인터페이스를 활용하여 아래처럼 다양한 선택 조건을 가진 여러 버전의 ApplePredicate을 정의할 수 있다.

```java
  public class AppleWeightPredicate implements ApplePredicate {     //무거운 사과만 선택
    @Override
    public boolean test(Apple apple) {
      return apple.getWeight() > 150;
    }
  }

  public class AppleColorPredicate implements ApplePredicate {      //녹색 사과만 선택
    @Override
    public boolean test(Apple apple) {
      return apple.getColor() == Color.GREEN;
    }
  }
```

위 조건에 따라 filter 메서드가 다르게 동작할 것이라고 예상할 수 있다. 이를 **전략 디자인 패턴 (strategy design patter)** 이라고 부른다. 전략 디자인 패턴은 각 알고리즘을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다. 위 예제에서는 ApplePredicate가 알고리즘 패밀리이고 AppleHeavyWeightPredicate와 AppleColorPredicate가 전략이다.

### 추상적 조건으로 필터링

```java
  public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if(p.test(apple)){          //프레디케이트 객체로 사과 검사 조건을 캡슐화했다.
            result.add(apple);
        }
    }
    return result;
  }
```

이제는 필요에 따라서 ApplePredicate을 적절하게 구현하는 클래스만 만들면 된다. 만약 농부가 무게가 150 이상인 빨간색 사과를 필터링 하고 싶다고 한다면 아래와 같이 ApplePredicate를 구현해주면 된다.

```java
  static class AppleRedAndHeavyPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {          //무게가 150 이상인 빨간색 사과 필터링.
      return apple.getColor() == Color.RED && apple.getWeight() > 150;
    }
  }

  이제 이렇게 구현한 객체를 필터 조건으로 넘겨주면 된다!

  List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```

드디어 성공적으로 filterApples 메서드의 동작을 파라미터화한 것이다!

### 한개의 파라미터, 다양한 동작

지금까지 살펴본 것처럼 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이 바로 **동작 파라미터화의 강점이다**.

## 2.3 복잡한 과정 간소화

하지만 위와 같은 방법도 새로운 동작을 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의한 다음에 인스턴스화해야 한다. 이를 간단히 해결할수 있도록 클래스의 선언과 인스턴스화를 동시에 수행할 수 있는 **익명 클래스 (anonymous class)**라는 기법이 존재한다.

```java
  List<Apple> redApples = filterApples(inventory, new ApplePredicate() {  //filterApples 메서드의 동작을 직접 파라미터화
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
  });
```

하지만 익명 클래스도 단점이 여전히 존재한다. 여전히 많은 공간을 차지하며, 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다. 한눈에 이해할 수 있어야 좋은 코드이지만 익명 코드는 다른 개발자들이 읽었을때 가독성이 현저히 떨어진다. 그리고 여전히 객체를 만들고 명시적으로 새로운 동작을 정의하는 메서드를 구현해야 한다는 점은 변하지 않았다!

### 람다 표현식 사용

자바 8의 람다 표현식을 사용하면 위 코드가 아래처럼 간략하게 변한다.

```java
  List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

### 리스트 형식으로 추상화

```java
  public interface Predicate<T> {
    boolean test(T t);
  }

  public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for(T e : list) {
        if(p.test(e)){
            result.add(e);
        }
    }
    return result;
  }
```

## 실전 예제
### Comparator로 정렬하기

아까 들었던 예처럼 컬렉션 정렬을 반복되는 프로그래밍 작업이다. 아까와 같이 농부가 색깔로 정렬하고 싶다가, 갑자기 나중에 무게로 정렬하고 싶다면?

이러한 상황에 유연하게 대응할 수 있도록 *java.util.Comparator*객체를 이용해서 sort의 동작을 파라미터화 할 수 있다.

```java
  public interface Comparator<T> {
    int compare(T o1, T o2);
  }
```

이걸 사용해서 무게가 적은 순서로 목록에서 사과를 정령해보자.

```java
  inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
  });
```

이걸 람다를 사용하면 더 간단하게 코드를 구현할 수 있다.

```java
  inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```


### Runnale로 코드 블록 실행하기

Runnable을 이용해서 다양한 동작을 스레드로 실행하는 방법.

```java
  Thred t = new Thread(new Runnable() {
    public void run() {
        System.out.println("Hello World!");
    }
  });

  람다를 사용하면 더욱 간략해진다.

  Thread t = new Thread(() -> System.out.println("Hello World!"));
```

### Callable을 결과로 반환하기

이 방식은 Runnable의 업그레디으 버전이라고 생각할 수 있다. 우선은 이렇게만 알아두자.

```java
  Future<String> threadName = executorService.submit(() -> Thread.currentThread().getName());
```

## 마치며
  - 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.
  - 동작 파라미터화를 이용하면 변화하는 요구사항에 더 유연하게 대응할 수 있다.
  - 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있다.