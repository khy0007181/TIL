# Optional

## Optional 소개
- 자바 프로그래밍에서 NullPointerException을 종종 보게 되는 이유
    * null을 리턴하거나 null 체크를 깜빡할 경우
- 메소드에서 작업 중 특별한 상황에서 값을 제대로 리턴할 수 없는 경우 선택할 수 있는 방법
    * 예외를 던진다. 
        - 스택 트레이스를 찍어두기 때문에 비싸다.
    * null을 리턴한다.
        - 비용 문제가 없지만 그 코드를 사용하는 클라이언트 코드가 주의해야 한다.
    * (Java 8부터) Optional을 리턴한다.
        - 클라이언트의 코드에게 명시적으로 빈 값일 수도 있다는 걸 알려주고, 빈 값인 경우에 대한 처리를 강제한다.
<br>

### Optional
- 오직 값 한 개가 들어있을 수도 없을 수도 있는 컨네이너.
    * `Optional.ofNullable(T value)`
        - null일 수 있다.
    * `Optional.of(T value)`
        - null이 아닌 것으로 가정한다.
        - null이 오면 NullPointException이 발생한다.
    * `Optional.empty()`
        - null을 담고 있는, 비어있는 Optional 객체를 얻어온다.
        - 비어있는 객체는 Optional 내부적으로 미리 생성해놓은 singleton인스턴스이다.
<br>

### 주의사항
- 리턴값으로만 쓰기를 권장한다.
    * 문법적 오류없이 쓸 수 있지만 권장되지 않는다.
    * 메소드 매개변수 타입, 맵의 키 타입, 인스턴스 필드 타입으로 사용 X
- Optional을 리턴하는 메소드에서 null을 리턴하지 않도록 주의해야 한다.
- 프리미티브 타입용 Optional을 따로 있다. `OptionalInt`, `OptionalLong`,...
- Collection, Map, Stream Array, Optional은 Opiontal로 감싸지 말아야 한다.
    * 컨테이너 성격의 인스턴스들은 비어있다는 것을 표현할 수 있다.
<br>

### 참고
- [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)
- [Tired of Null Pointer Exceptions? Consider](https://www.oracle.com/technical-resources/articles/java/java8-optional.html)
<br>

## Optional API

### Optional 만들기
- `Optional.of()`
- `Optional.ofNullable()`
- `Optional.empty()`
<br>

### Optional에 값이 있는지 없는지 확인하기
- `isPresent()`
- `isEmpty()` 
    * Java 11부터 제공
```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> optional = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        System.out.println(optional.isPresent()); // true
        System.out.println(optional.isEmpty()); // false
    }
}
```
<br>


### Optional에 있는 값 가져오기
- `get()`
- 만약 Optional이 비어있다면?
    * RuntimeException이 발생한다.
    * 가급적이면 `get()`을 사용하지 않고 밑에 있는 메서드를 사용하는 것이 낫다.
```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> optional = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        OnlineClass onlineClass = optional.get();
        System.out.println(onlineClass.getTitle()); // spring boot
    }
}
```
<br>

### Optional에 값이 있는 경우에 그 값 사용하기
- `ifPresent(Consumer)`
```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> optional = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        optional.ifPresent(oc -> System.out.println(oc.getTitle())); // spring boot
    }
}
```
<br>

### Optional에 값이 있으면 가져오고 없으면 다른 값을 반환
- `orElse(T)`
- 다음 코드에서 optional에 값이 있으나 없으나 `createNewClass()`의 출력은 실행된다.
```java
public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> optional = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("jpa"))
                .findFirst();

        OnlineClass onlineClass = optional.orElse(createNewClass());
        System.out.println(onlineClass.getTitle()); 

        // creating new online class
        // New class
    }

    private static OnlineClass createNewClass() {
        System.out.println("creating new online class");
        return new OnlineClass(10, "New class", false);
    }
```
<br>

### Optional에 값이 있으면 가져오고 없는 경우에 Supplier 실행
- `orElseGet(Supplier)`
    * `orElse()`와 달리 optional에 값이 있는 경우 Supplier를 실행하지 않는다.
```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> optional = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        OnlineClass onlineClass = optional.orElseGet(App::createNewClass);
        System.out.println(onlineClass.getTitle()); // spring boot
    }

    private static OnlineClass createNewClass() {
        System.out.println("creating new online class");
        return new OnlineClass(10, "New class", false);
    }
}
```
<br>

### Optional에 값이 있으면 가져오고 없는 경우 에러를 던지기
- `orElseThrow()`
    * 기본적으로는 NoSuchElementException을 던지지만 원하는 에러가 있다면 Supplier로 제공해주면 된다.
        - Lambda Expressions 또는 Method Reference 사용
```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> optional = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        // Lambda
        OnlineClass onlineClass = optional.orElseThrow(() -> {
           return new IllegalArgumentException();
        });

        // Method Reference
        OnlineClass onlineClass = optional.orElseThrow(IllegalStateException::new);

        System.out.println(onlineClass.getTitle());
    }
}
```
<br>

### Optional에 들어있는 값 걸러내기
- `Optional filter(Predicate)`
    * 있다는 가정하에 동작하면 없는 경우 아무 일도 일어나지 않는다.
    * 결과는 Optional이다. 
        - filter에 해당되면 그 Optional 그대로 나오고 해당되지 않으면 비어있는 Optional이 나온다.
```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> optional = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        Optional<OnlineClass> onlineClass = optional.filter(oc -> !oc.isClosed());
        System.out.println(onlineClass.isEmpty()); // true
    }
}
```
<br>

### Optional에 들어있는 값 변환하기
- `Optional map(Function)`
- `Optional flatMap(Function)`
    * Optional 안에 들어있는 인스턴스가 Optional인 경우에 사용하면 편리하다.
    * Stream의 `flatMap()`과는 다르다.
    * map으로 변환한 타입이 Optional이면 `flatMap()` 사용
- map을 사용하면 결과는 Optional이며 Function이 리턴하는 타입에 따라 Optional이 담고 있는 타입이 달라진다.
```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> optional = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        // map으로 변환한 타입이 Optional이면?
        Optional<Progress> progress = optional.flatMap(OnlineClass::getProgress);
        
        // 이렇게 2번 꺼낼 필요가 없다.
        Optional<Optional<Progress>> progress1 = optional.map(OnlineClass::getProgress);
        Optional<Progress> progress2 = progress1.orElse(Optional.empty());
    }
}
```