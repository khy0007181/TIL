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
    public class foo {

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
    public class foo {

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
    public class foo {

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
    public class foo {

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
    public class foo {

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
