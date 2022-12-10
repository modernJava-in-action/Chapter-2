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

