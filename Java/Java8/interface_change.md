# 인터페이스의 변화

## 인터페이스 기본 메소드와 스태틱 메소드

### 기본 메소드 (Default Methods)
- 인터페이스에 메소드 선언이 아니라 구현체를 제공하는 방법
- 해당 인터페이스를 구현한 클래스를 깨트리지 않고 새 기능을 추가할 수 있다.
- 기본 메소드는 구현체가 모르게 추가된 기능으로 그만큼 리스크가 있다.
    * 기본 메소드의 기능이 모든 인스턴스에 정상적으로 동작할 것이라는 보장이 없다.
    * 컴파일 에러는 아니지만 구현체에 따라 런타임 에러가 발생할 수 있다.
    * `@implSpec` 자바독 태그 사용해서 반드시 문서화 해야 한다.
        - 제공하는 기본 메소드의 구현체가 어떤 역할을 하는 것이다라고 명시하는 것이 좋다.
- Object가 제공하는 기능(equals, hasCode)는 기본 메소드로 제공할 수 없다.
    * 구현체가 재정의해야 한다.
- 본인이 수정할 수 있는 인터페이스에만 기본 메소드를 제공할 수 있다.
- 인터페이스를 상속받는 인터페이스에서 다시 추상 메소드로 변경할 수 있다.
- 인터페이스 구현체가 재정의 할 수도 있다.
```java
public interface Foo {

    void printName();

    /**
     * @implSpec
     * 이 구현체는 getName()으로 가져온 문자열을 대문자로 바꿔 출력한다.
     */
    default void printNameUpperCase() {
        System.out.println(getName().toUpperCase());
    }

    String getName();

}
```
```java
public class DefaultFoo implements Foo{

    String name;

    public DefaultFoo(String name) {
        this.name = name;
    }

    @Override
    public void printName() {
        System.out.println(this.name);
    }

    @Override
    public String getName() {
        return this.name;
    }
    
//    재정의 할 수도 있다.
//    @Override
//    public void printNameUpperCase() {
//        System.out.println(this.name.toUpperCase());
//    }
}
```
```java
public class App {

    public static void main(String[] args) {
        Foo foo = new DefaultFoo("hayoung");
        foo.printName(); // hayoung
        foo.printNameUpperCase(); // HAYOUNG

    }

}
```
- 만약 구현하는 인터페이스에 기본 메소드가 중복된다면?
    * diamond problem
    * 컴파일 에러가 발생한다.
```java
public interface Foo {

    default void printNameUpperCase() {
        System.out.println(getName().toUpperCase());
    }

}
```
```java
public interface Bar {

    default void printNameUpperCase() {
        System.out.println("Bar");
    }
}
```
```java
public class DefaultFoo implements Foo, Bar{

    // 기본 메소드 printNameUpperCase() 중 어떤 것을 사용해야 할 지 모르기 때문에 컴파일 에러 발생
    // 직접 Overriding을 해야 한다.
}
```
<br>

### 스태틱 메소드
- 해당 타입 관련 헬퍼 또는 유틸리티 메소드를 제공할 때 인터페이스에 스태틱 메소드를 제공할 수 있다.
```java
public interface Foo {

    ...

    static void printAnything() {
        System.out.println("Foo");
    }

}
```
```java
public class App {

    public static void main(String[] args) {
        ...

        foo.printAnything(); // Foo
    }

}
```
<br>

### 참고
- [Evolving Interfaces](https://docs.oracle.com/javase/tutorial/java/IandI/nogrow.html)
- [Default Methods](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)
<br>

## Java 8 API의 기본 메소드와 스태틱 메소드
- Java 8에서 추가한 기본 메소드로 인한 API 변화
<br>

### [Iterable](https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html)의 기본 메소드
- `forEach()`
```java
public class App {

    public static void main(String[] args) {
        List<String> name = new ArrayList<>();
        name.add("hayoung");
        name.add("kimhayoung");
        name.add("paperCar");
        name.add("foo");

        name.forEach(System.out::println);
    }

}
```
- [`spliterator()`](https://docs.oracle.com/javase/8/docs/api/java/util/Spliterator.html)
    * iterator와 비슷하지만 쪼갤 수 있는 기능을 가지고 있는 iterator라고 생각하면 된다.
```java
public class App {

    public static void main(String[] args) {
        List<String> name = new ArrayList<>();
        name.add("hayoung");
        name.add("kimhayoung");
        name.add("paperCar");
        name.add("foo");

        Spliterator<String> spliterator = name.spliterator();
        Spliterator<String> spliterator1 = spliterator.trySplit(); // 반으로 나눠진다.
        while(spliterator.tryAdvance(System.out::println));
        System.out.println("==========");
        while (spliterator1.tryAdvance(System.out::println));
    }
}
```
<br>

### [Collection](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html)의 기본 메소드
- `stream() / parallelStream()`
    * `stream()`은 element들을 stream으로 만들어서 functional한 처리를 할 수 있다.
- `removeIf(Predicate)`
```java
public class App {

    public static void main(String[] args) {
        List<String> name = new ArrayList<>();
        name.add("hayoung");
        name.add("kimhayoung");
        name.add("paperCar");
        name.add("foo");

        // k로 시작하는 것을 삭제
        name.removeIf(s -> s.startsWith("k"));

        name.forEach(System.out::println);
        // hayoung
        // paperCar
        // foo
    
    }

}
```
- `spliterator()`
<br>

### [Comparator](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html)의 기본 메소드 및 스태틱 메소드
- `reversed()`
    * 반대로 정렬
- `thenComparing()`
    * 추가적으로 compare 하고 싶은 경우 사용
- `static reverseOrder()` / `naturalOrder()`
- `static nullsFirst()` / `nullsLast()`
    * null 값인 경우 우선순위 결정
- `static comparing()`
<br>
