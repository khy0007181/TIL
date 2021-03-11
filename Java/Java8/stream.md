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