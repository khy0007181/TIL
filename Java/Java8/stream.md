# Stream

### Stream
- sequence of elements supporting sequential and parallel aggregate operations
- 데이터를 담고 있는 저장소(컬렉션)가 아니다.
- 연속된 데이터를 처리하는 operation들의 모임이다.
- Funtional in nature, 스트림이 처리하는 데이터 소스를 변경하지 않는다.
- 스트림으로 처리하는 데이터는 오직 한번만 처리한다.
- 무제한일 수도 있다. (Short Circuit 메소드를 사용해서 제한할 수 있다.)
- 중개 오퍼레이션은 근본적으로 lazy하다.
    * terminal operation이 마지막에 있어야 intermediate operation이 실행된다.
- 손쉽게 병렬 처리할 수 있다.
    * stream은 `parallelStream()`을 받아서 처리하면 JVM이 병렬적으로 처리해준다.
```java
public class App {

    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("hayoung");
        names.add("kimhayoung");
        names.add("paperCar");
        names.add("foo");

        // 결과가 또 다른 Stream이 된다.
        Stream<String> stringStream = names.stream().map(String::toUpperCase);

        // 실제 데이터를 변경하지 않는다.
        names.forEach(System.out::println); // 그대로 소문자로 출력

        // 만약 collect()를 사용하지 않았다면?
        // intermediate operation가 있고 terminal operation가 없으므로 map()안의 println이 실행되지 않는다.
        List<String> collect = names.stream().map((s) -> {
            System.out.println(s);
            return s.toUpperCase();
        }).collect(Collectors.toList());
        collect.forEach(System.out::println); // 대문자로 출력

//        루프를 도는 이러한 코드는 병렬적으로 처리하기 힘들다.
//        for(String name : names) {
//            if(name.startsWith("k")) {
//                System.out.println(name.toUpperCase());
//            }
//        }

        // parallelStream을 사용해서 병렬 처리
        List<String> collect1 = names.parallelStream().map(String::toUpperCase)
                .collect(Collectors.toList());
        collect1.forEach(System.out::println);

    }

}
```
<br>

### Stream 파이프라인
- 0 또는 다수의 중개 오퍼레이션 (intermediate operation)과 한개의 종료 오퍼레이션 (terminal operation)으로 구성한다.
- 스트림의 데이터 소스는 오직 terminal operation을 실행할 때에만 처리한다.
    * intermediate operation들은 terminal operation이 오기전까지는 실행하지 않는다.
<br>

### intermediate operation
- **Stream을 리턴한다.**
- Stateless / Stateful 오퍼레이션으로 더 상세하게 구분할 수도 있다. 
    * 대부분은 Stateless지만 `distinct()`나 `sorted()` 처럼 이전 이전 소스 데이터를 참조해야 하는 오퍼레이션은 Stateful 오퍼레이션이다.
- `filter()`, `map()`, `limit()`, `skip()`, `sorted()`, ...
<br>

### terminal operation
- **Stream을 리턴하지 않는다.**
- `collect()`, `allMatch()`, `count()`, `forEach()`, `min()`, `max()`, ...
<br>

### 참고
- [Interface Stream<T>](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
- [Package java.util.stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)
<br>

## 스트림 API 예제

### 걸러내기
- `Filter(Predicate)`
- 예) 이름이 3글자 이상인 데이터만 새로운 스트림으로 
<br>

### 변경하기
- `Map(Function)` 또는 `FlatMap(Function)`
- 예) 각각의 Post 인스턴스에서 String title만 새로운 스트림으로
- 예) List<Stream<String>>을 String의 스트림으로
<br>

### 생성하기
- `generate(Supplier)` 또는 `Iterate(T seed, UnaryOperator)`
- 예) 10부터 1씩 증가하는 무제한 숫자 스트림
- 예) 랜덤 int 무제한 스트림
<br>

### 제한하기
- `limit(long)` 또는 `skip(long)`
- 예) 최대 5개의 요소가 담긴 스트림을 리턴한다.
- 예) 앞에서 3개를 뺀 나머지 스트림을 리턴한다.
<br>

### 스트림에 있는 데이터가 특정 조건을 만족하는지 확인
- `anyMatch()`, `allMatch()`, `nonMatch()`
- 예) k로 시작하는 문자열이 있는지 확인한다. (true 또는 false를 리턴한다.)
- 예) 스트림에 있는 모든 값이 10보다 작은지 확인한다.
<br>

### 개수 세기
- `count()`
- 예) 10보다 큰 수의 개수를 센다.
<br>

### 스트림을 데이터 하나로 뭉치기
- `reduce(identity, BiFunction)`, `collect()`, `sum()`, `max()`
- 예) 모든 숫자 합 구하기
- 예) 모든 데이터를 하나의 List 또는 Set에 옮겨 담기
<br>

### 예제 코드
```java
public class App {

    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(2, "spring data jpa", true));
        springClasses.add(new OnlineClass(3, "spring mvc", false));
        springClasses.add(new OnlineClass(4, "spring core", false));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        List<OnlineClass> javaClasses = new ArrayList<>();
        javaClasses.add(new OnlineClass(6, "The Java, Test", true));
        javaClasses.add(new OnlineClass(7, "The Java, Code manipulation", true));
        javaClasses.add(new OnlineClass(8, "The Java, 8 to 11", false));

        List<List<OnlineClass>> hayoungEvents = new ArrayList<>();
        hayoungEvents.add(springClasses);
        hayoungEvents.add(javaClasses);

        // spring 으로 시작하는 수업
        springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .forEach(oc -> System.out.println(oc.getId()));

        // close 되지 않은 수업
        springClasses.stream()
                // .filter(oc -> !oc.isClosed())로 해도 되지만 더 단축도 가능
                .filter(Predicate.not(OnlineClass::isClosed))
                .forEach(oc -> System.out.println(oc.getId()));

        // 수업 이름만 모아서 스트림 만들기
        springClasses.stream()
                .map(OnlineClass::getTitle)
                .forEach(System.out::println);

        // 두 수업 목록에 들어있는 모든 수업 아이디 출력
        hayoungEvents.stream()
                .flatMap(Collection::stream)
                .forEach(oc -> System.out.println(oc.getId()));

        // 10부터 1씩 증가하는 무제한 스트림 중에서 앞에 10개 빼고 최대 10개 까지만
        Stream.iterate(10, i -> i + 1)
                .skip(10)
                .limit(10)
                .forEach(System.out::println);

        // 자바 수업 중에 Test가 들어있는 수업이 있는지 확인
        boolean test = javaClasses.stream().anyMatch(oc -> oc.getTitle().contains("Test"));
        System.out.println(test);

        // 스프링 수업 중에 제목에 spring이 들어간 것만 모아서 List로 만들기
        List<String> spring = springClasses.stream()
                .map(OnlineClass::getTitle)
                .filter(t -> t.contains("spring"))
                .collect(Collectors.toList());
        spring.forEach(System.out::println);
        
    }

}
```
<br>
