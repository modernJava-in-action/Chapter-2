# Chapter 2 - 동작 파라미터화 코드 전달하기
**동작 파라미터화**(behavior parameterization)를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있습니다.  
동작 파라미터화란 `아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록`을 의미합니다. 이 코드 블록은 나중에 프로그램에서 호출합니다.  
즉, 코드 블록의 실행은 나중으로 미뤄집니다.  
  
예를 들어 나중에 실행될 메서드의 인수로 코드 블록을 전달할 수 있습니다. 결과적으로 코드 블록에 따라 메서드의 동작이 파라미터화됩니다.  
컬렉션을 처리할 때 다음과 같은 메서드를 구현한다고 가정해봅니다.  
- 리스트의 모든 요소에 대해서 '어떤 동작'을 수행할 수 있음(ex:forEach)    
- 리스트 관련 작업을 끝낸 다음에 '어떤 다른 동작'을 수행할 수 있음    
- 에러가 발생하면 '정해진 어떤 다른 동작'을 수행할 수 있음  
동작 파라미터화로 이처럼 다양한 기능을 수행할 수 있습니다.  
  
동작 파라미터화를 추가하려면 쓸데없는 코드가 늘어나는데, Java8은 람다 표현식으로 이 문제를 해결합니다.  

## 2.1 변화하는 요구사항에 대응하기
### 2.1.1 녹색 사과 필터링
```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (GREEN.equals(apple.getColor())) { //녹색 사과를 선택하는 데 필요한 조건 
        result.add(apple);
      }
    }
    return result;
  }
```
녹색이라는 조건이 좀 더 다양한 색으로 바뀐다면 적절히 대응할 수 없다.  
**거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.**

### 2.1.2 두 번째 시도 : 색을 파라미터화
색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하면 좀 더 유연하게 대응하는 코드를 만들 수 있다.  
```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getColor().equals(color)) {
        result.add(apple);
      }
    }
    return result;
  }
```
다음처럼 구현한 메서드를 호출할 수 있습니다.  
```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```
그런데 농부가 무게의 기준도 요구해서 무게 정보 파라미터를 추가했습니다.  
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
코드를 보면 목록을 검색하고, 각 사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복됩니다.  
DRY(don't repeat yourselt) 원칙을 어기는 것입니다.  
### 2.1.3 세 번째 시도 : 가능한 모든 속성으로 필터링 
모든 속성을 메서드 파라미터로 추가한 모습입니다.(하지 말아야 할 것)  
```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, Color color,
                                                  int weight, boolean flag) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if ((flag && apple.getColor().equals(color)) ||
        (!flag && apple.getWeight() > weight)) {
      result.add(apple);
    }
  }
  return result;
}
```
요구사항이 바뀌었을 때 유연하게 대응할 수 없습니다. true와 false가 어떤 것을 의미하는지 알 수 없습니다.  
동작 파라미터화를 이용해서 유연성을 얻는 방법을 설명합니다.  
## 2.2 동작 파라미터화
우리의 선택 조건을 다음처럼 결정할 수 있습니다. 사과의 어떤 속성에 기초해서 `불리언값`을 반환하는 방법이 있다.(예를 들어 사과가 녹색인가? 150그램 이상인가?)  
참 또는 거짓을 반환하는 함수를 **프레디케이트**라 합니다. 선택 조건을 결정하는 인터페이스를 인터페이스를 정의합니다.  
```java
public interface ApplePredicate {
  boolean test(Apple apple);
}
```
다음 예제처럼 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있습니다.  
```java
public class AppleHeavyWeightPredicate implements ApplePredicate{
  @Override
  public boolean test(Apple apple) {
    return apple.getWeight() > 150;
  }
}
public class AppleGreenColorPredicate implements ApplePredicate{

  @Override
  public boolean test(Apple apple) {
    return GREEN.equals(apple.getColor());
  }
}
```
ApplePredicate는 사과 선택 전략을 캡슐화하였습니다. 위 조건에 따라 filter 메서드가 다르게 동작할 것이라고 예상할 수 있습니다.  
이를 `전략 디자인 패턴(strategy design patter)`이라고 부릅니다.  
  
전략 디자인 패턴은 각 알고리즘을 캡슐화하는 `알고리즘 패밀리`를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법입니다.  
이 예제에서는 ApplePredicate가 알고리즘 패밀리, AppleHeavyWeightPredicate와 AppleGreenColorPredicate가 전략입니다.  
    
그런데 ApplePredicate는 어떻게 다양한 동작을 수행할가요? filterApples에서 ApplePredicate 객체를 받아 사과의 조건을 검사하도록 메서드를 고쳐야 합니다.  
이렇게 **동작 파라미터화**, 즉 메서드가 다양한 동작 (또는 전략)을 받아서 내부적으로 다양한 동작을 수행할 수 있습니다.  

### 2.1.1 네 번째 시도 : 추상적 조건으로 필터링 
다음은 ApplePredicate를 이용한 필터 메서드입니다.  
```java
//==ApplePredicate 를 이용한 필터 메서드==//
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if (p.test(apple)) {
      result.add(apple);
    }
  }
  return result;
}
```
이제 필요한 대로 다양한 ApplePredicate 구현체를 만들어서 filterApples 메서드로 전달할 수 있습니다.  
예를 들어 농부가 `150그램이 넘는 빨간 사과를 검색해달라고 부탁`하면 우리는 ApplePredicate를 적절하게 구현하는 클래스만 만들면 됩니다.  
```java
public class AppleRedAndHeavyPredicate implements ApplePredicate{
  @Override
  public boolean test(Apple apple) {
    return RED.equals(apple.getColor()) && apple.getWeight() > 150;
  }
}

//사용
List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```
ApplePredicate 객체에 의해(위에서는 AppleRedAndHeavyPredicate) filterApples의 동작이 결정됩니다.  
즉, 우리는 `filterApples`의 동작을 파라미터화한 것입니다.  
  
위 예제에서 가장 중요한 구현은 test메서드입니다. filterApples의 새로운 동작을 정의하는 것이 test 메서드입니다.  
안타깝게도 filterApples메서드는 객체만 인수로 받으므로 test 메서드를 ApplePredicate 객체로 감싸서 전달해야 합니다.  
  
람다를 이용하면 여러 개의 ApplePredicate 클래스를 정의하지 않고도 `apple.getWeight() > 150` 같은 표현식을 filterApples 메서드로 전달할 수 있습니다.  

`컬렉션 탐색 로직`과 `각 항목에 적용할 동작`을 분리할 수 있다는 것이 동작 파라미터화의 강점입니다.  
## 2.3 복잡한 과정 간소화
현재 filterApples 메서드로 새로운 동작을 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의한 다음에 인스턴스화해야 한다.  
이는 상당히 번거로운 작업이며 시간 낭비입니다.  
  
참고)인스턴스화는 새로운 객체를 위해 `힙 영역의 메모리`를 할당하고, 변수에 할당된 메모리주소(reference)를 반환합니다.  
 - 객체의 생성자가 실행됩니다  
 - 런타임에 새로운 객체를 위해 힙 영역의 메모리를 할당합니다  
 - 새로운 객체를 위해 힙 영역에 할당된 메모리 주소는 스택 영역의 변수에 저장됩니다.  
  
익명 클래스와 람다를 이용해 코드의 양을 줄일 수 있습니다.  
### 2.3.1 익명 클래스
익명 클래스를 이용하면 `클래스 선언`과 `인스턴스화`를 동시에 할 수 있습니다.  

### 2.3.2 다섯 번째 시도 : 익명 클래스 사용 
```java
List<Apple> redApplesWithAnonymousClass = filterApples(inventory, new ApplePredicate() {
    @Override
    public boolean test(Apple apple) {
      return RED.equals(apple.getColor());
    }
  });
```
코드의 장황함, 반복되는 지저분한 코드들은 람다 표현식을 이용해 처리할 수 있습니다.  
### 2.3.3 여섯 번째 시도 : 람다 표현식 사용 
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```
### 2.3.4 일곱 번째 시도 : 리스트 형식으로 추상화
```java
public interface Predicate<T> {
    boolean test(T t);
  }

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
  List<T> result = new ArrayList<>();
  for (T e : list) {
    if (p.test(e)) {
      result.add(e);
    }
  }
  return result;
}
List<Integer> evenNumbers = filter(numbers , (Integer i) -> i % 2 == 0); 
```
추상화를 통해 다양한 형식을 사용 가능합니다.  

## 2.4 실전 예제


