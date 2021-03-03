# 함수형 인터페이스와 람다 표현식

## 함수형 인터페이스와 람다 표현식 소개

### 함수형 인터페이스 (Functional Interface)
```java
@FunctionalInterface
public interface RunSomething {

    void doIt();
}
```
- 추상 메소드를 딱 하나만 가지고 있는 인터페이스
    * 2개 있으면 안된다.
- SAM (Single Abstract Method) 인터페이스
- `@FuncationInterface` 애노테이션을 가지고 있는 인터페이스
    * Java가 제공해주는 애노테이션
    * 애노테이션을 붙이고 함수형 인터페이스를 위반하면 컴파일 에러 발생
- 정의한 함수형 인터페이스를 사용하는 방법 : 람다 표현식
<br>

### 람다 표현식(Lambda Expressions)
- 함수형 인터페이스의 인스턴스를 만드는 방법으로 쓰일 수 있다.
- 코드를 줄일 수 있다.
- 메소드 매개변수, 리턴 타입, 변수로 만들어 사용할 수도 있다.
- 함수형 인터페이스 사용
    * Java 8 이전
        - 인터페이스의 구현체를 만들어서 사용
    ```java
    public class Foo {

        public static void main(String[] args) {
            // 익명 내부 클래스(anonymous inner class)
            RunSomething runSomething = new RunSomething() {
                @Override
                public void doIt() {
                    System.out.println("Hello");
                }
            };
        }
    }
    ```
    * Java 8에서는 인터페이스가 1개인 경우 람다 표현식 사용
        - IDE에서는 단축키로 람다 표현식으로 바꿔준다.
    ```java
    public class Foo {

        public static void main(String[] args) {
            RunSomething runSomething = () -> System.out.println("Hello");
        }
    }
    ```
<br>

### Java에서의 함수형 프로그래밍
- 함수를 First class object로 사용할 수 있다.
    * 자바에서 함수는 특수한 형태의 object일 뿐이다.
    * 자바는 객체지향 언어이므로 함수를 변수에 할당하고, 메서드의 파라미터로 전달하고, 리턴 타입으로 리턴할 수 있다.
- 순수 함수(Pure function)
    * 수학적인 개념으로 가장 중요한 것은 입력받은 값이 동일한 경우 결과가 항상 같아야 한다는 것이다.
        - 이것을 보장하지 못한다면, 또는 그럴 여지가 있을 경우 함수형 프로그래밍이라고 보기 어렵다.
    ```java
    @FunctionalInterface
    public interface RunSomething {

        int doIt(int number);
    }
    ```
    ```java
    public class Foo {

        public static void main(String[] args) {
            RunSomething runSomething = (number) -> {
                return number + 10;
            };

            // 항상 11이어야 한다.
            runSomething.doIt(1);
            runSomething.doIt(1);
            runSomething.doIt(1);
        }
    }
    ``` 
    * 사이드 이팩트가 없다.
        - 함수 밖에 있는 값을 변경하는 경우 pure한 함수라고 할 수 없다.
    ```java
    public class Foo {

        public static void main(String[] args) {
            RunSomething runSomething = new RunSomething() {
                int baseNumber = 10;
                
                @Override
                public int doIt(int number) {
                    baseNumber++;
                    return number + baseNumber;                
                }
            }
        }
    }
    ```
    * 상태가 없다.
        - 함수 밖에 있는 값을 사용할 경우 pure한 함수라고 할 수 없다.
        - final일 경우 사용 가능
    ```java
    public class Foo {

        public static void main(String[] args) {
            RunSomething runSomething = new RunSomething() {
                int baseNumber = 10;
                
                @Override
                public int doIt(int number) {
                    return number + baseNumber;                
                }
            }
        }
    }
    ```
- 고차 함수(Higher-Order Function)
    * 함수가 함수를 매개변수로 받을 수 있고 함수를 리턴할 수도 있다.
- 불변성
<br>

## Java에서 제공하는 함수형 인터페이스

### Java가 기본으로 제공하는 함수형 인터페이스
- 자바에서 미리 정의해둔 자주 사용할만한 함수 인터페이스
    * 일반적인 함수 인터페이스는 [java.util.function 패키지](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)에 들어있다.
- `Function<T, R>`
    * T 타입을 받아서 R타입을 리턴하는 함수 인터페이스
    * `apply()` 메서드만 구현하면 된다.
    ```java
    public class Plus10 implements Function<Integer, Integer> {
        @Override
        public Integer apply(Integer integer) {
            return integer + 10;
        }
    }
    ```
    ```java
    public class Foo {

        public static void main(String[] args) {
            Plus10 plus10 = new Plus10();
            System.out.println(plus10.apply(1));
        }
    }
    ```
    * 람다 표현식을 사용해 바로 구현해도 된다.
    ```java
    public class Foo {

        public static void main(String[] args) {
            Function<Integer, Integer> plus10 = (number) -> number + 10;
            System.out.println(plus10.apply(1));
        }
    }
    ```
    * 함수를 조합할 수 있는 메서드를 default 메서드로 제공한다.
        - `andThen()`
        - `compose()`
    ```java
    public class Foo {

        public static void main(String[] args) {
            Function<Integer, Integer> plus10 = (number) -> number + 10;
            Function<Integer, Integer> multiply2 = (number) -> number * 2;

            Function<Integer, Integer> multiply2AndPlus10 = plus10.compose(multiply2); // multiply2 한 값이 plus10의 입력값으로

            System.out.println(multiply2AndPlus10.apply(2)); // 14
            System.out.println(plus10.andThen(multiply2).apply(2)); // 24
        }
    }
    ```
- `BiFunction<T, U, R>`
    * 두 개의 값(T, U)를 받아서 R 타입을 리턴하는 함수 인터페이스
- `Consumer<T>`
    * T 타입을 받아서 아무값도 리턴하지 않는 함수 인터페이스
    * 함수 조합용 메서드
        - `andThen()`
```java
public class Foo {

    public static void main(String[] args) {
        Consumer<Integer> printT = (i) -> System.out.println(i);

        printT.accept(10);
    }
}
```
- `Supplier<T>`
    * T 타입의 값을 제공하는 함수 인터페이스
```java
public class Foo {

    public static void main(String[] args) {
        Supplier<Integer> get10 = () -> 10;
        System.out.println(get10.get()); // 10
    }
}
```
- `Predicate<T>`
    * T 타입을 받아서 boolean을 리턴하는 함수 인터페이스
    * 함수 조합용 메서드
        - `And()`
        - `Or()`
        - `Negate()`
```java
public class Foo {

    public static void main(String[] args) {
        Predicate<String> startsWithhayoung = (s) -> s.startsWith("hayoung");
        Predicate<Integer> isEven = (i) -> i % 2 == 0;
    }
}
```
- `UnaryOperator<T>`
    * `Function<T, R>`의 특수한 형태로, 입력값 하나를 받아서 동일한 타입을 리턴하는 함수 인터페이스
    * `Function<T, R>`을 상속받기 때문에 제공하는 메서드를 사용할 수 있다.
- `BinaryOperator<T>`
    * `BiFunction<T, U, R>`의 특수한 형태로, 동일한 타입의 입렵값 두개를 받아 리턴하는 함수 인터페이스
<br>
