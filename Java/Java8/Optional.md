# Optional

## Optional 소개
- 자바 프로그래밍에서 NullPointerException을 종종 보게 되는 이유
    * null을 리턴하거나 null 체크를 깜빡할 경우
- 메소드에서 작업 중 특별한 상황에서 값을 제대로 리턴할 수 없는 경우 선택할 수 있는 방법
    * 예외를 던진다. 
        - 스택트레이스를 찍어두기 때문에 비싸다.
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
- [](https://www.oracle.com/technical-resources/articles/java/java8-optional.html)
<br>
