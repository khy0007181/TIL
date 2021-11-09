# 프로젝트 설정

## 프로젝트 생성
- Gradle Project
- Java 11
- Spring Boot 2.5.6
- Dependencies
    - Spring Web
    - Spring Data JPA
    - H2 Database
    - Lombok
<br>

### IntelliJ Gradle 대신에 자바로 바로 실행하기
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/ProjectSetting_1.png"></p>

- 최근 IntelliJ 버전은 Gradle로 실행을 하는 것이 기본 설정이다.
    - 실행속도가 느리기 때문에 다음과 같이 변경하면 자바로 바로 실행하므로 좀 더 빨라진다.
    - Preferences Build, Execution, Deployment Build Tools Gradle
    - Build and run using: Gradle IntelliJ IDEA
    - Run tests using: Gradle IntelliJ IDEA
<br>

### Lombok 적용
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/ProjectSetting_2.png"></p>

- 최근 Lombok이 plugin 설치에서 bundle로 변경되었다.
    - 설치할 필요 없음
- Preferences Annotation Processors 검색 Enable annotation processing 체크 (재시작)
- 임의의 테스트 클래스를 만들고 @Getter, @Setter 확인
<br>

## Querydsl 설정과 검증

### build.gradle 파일 Querydsl 설정 추가
```gradle
plugins {
    id 'org.springframework.boot' version '2.5.6'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    // 추가
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
    extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    // 추가
    implementation 'com.querydsl:querydsl-jpa'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
    useJUnitPlatform()
}

// 추가
def querydslDir = "$buildDir/generated/querydsl"

querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
sourceSets {
    main.java.srcDir querydslDir
}
configurations {
    querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
```
<br>

### Querydsl 환경설정 검증
- 검증용 엔티티 생성
```java
@Entity
@Getter @Setter
public class Hello {

    @Id @GeneratedValue
    private Long id;
}
```
- 검증용 Q 타입 생성
    - Gradle - Tasks - build - clean
    - Gradle - Tasks - other - compileQuerydsl
    - 완료하면 build.gradle에서 설정한 build/generated에 querydsl폴더가 생긴다.
    - Q타입은 컴파일 시점에 자동 생성되므로 버전관리(GIT)에 포함하지 않는 것이 좋다. 
        - 앞서 설정에서 생성 위치를 gradle build 폴더 아래 생성되도록 했기 때문에 이 부분도 자연스럽게 해결된다. (대부분 gradle build 폴더를 git에 포함하지 않는다.)  
- 테스트 케이스로 실행 검증
```java
@SpringBootTest
@Transactional
class QuerydslApplicationTests {
    @Autowired
    EntityManager em;

    @Test
    void contextLoads() {
        Hello hello = new Hello();
        em.persist(hello);
        JPAQueryFactory query = new JPAQueryFactory(em);
        QHello qHello = QHello.hello; //Querydsl Q타입 동작 확인
        Hello result = query
                .selectFrom(qHello)
                .fetchOne();
        Assertions.assertThat(result).isEqualTo(hello);
        //lombok 동작 확인 (hello.getId())
    Assertions.assertThat(result.getId()).isEqualTo(hello.getId());
	}
}
```
- 만약 테스트 실행 시 error: cannot find symbol가 나오면 Build Tools - Gradle 설정에서 Build and run using과 Run tests using 옵션을 Gradle에서 IntelliJ IDEA로 변경하면 된다.
<br>

## 라이브러리 살펴보기
- gradle 의존관계 보기
    - `./gradlew dependencies --configuration compileClasspath`
- Querydsl 라이브러리
    - querydsl-apt - Querydsl 관련 코드 생성 기능 제공
    - querydsl-jpa - querydsl 라이브러리
- 스프링 부트 라이브러리
    - spring-boot-starter-web
        - spring-boot-starter-tomcat - 톰캣(웹서버)
        - spring-webmvc - 스프링 웹 MVC
    - spring-boot-starter-data-jpa
        - spring-boot-starter-aop
        - spring-boot-starter-jdbc
            - HikariCP 커넥션 풀 (부트 2.0 기본)
        - hibernate + JPA - 하이버네이트 + JPA
        - spring-data-jpa - 스프링 데이터 JPA
    - spring-boot-starter(공통) - 스프링 부트 + 스프링 코어 + 로깅
        - spring-boot
            - spring-core
        - spring-boot-starter-logging
            - logback, slf4j
- 테스트 라이브러리
    - spring-boot-starter-test
        - junit - 테스트 프레임워크, 스프링 부트 2.2부터 junit5(jupiter) 사용
            - 과거 버전은 vintage
        - mockito - 목 라이브러리
        - assertj - 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
            - [AssertJ](https://joel-costigliola.github.io/assertj/index.html)
        - spring-test - 스프링 통합 테스트 지원
    - 핵심 라이브러리
        - 스프링 MVC
        - JPA, 하이버네이트
        - 스프링 데이터 JPA
        - Querydsl
    - 기타 라이브러리
        - H2 데이터베이스 클라이언트
        - 커넥션 풀 - 부트 기본은 HikariCP
        - 로깅 SLF4J & LogBack
        - 테스트
<br>

## H2 데이터베이스 설치
- 개발이나 테스트 용도로 가볍고 편리한 DB, 웹 화면 제공
- [H2 Database]()
- H2 Database 버전은 스프링 부트 버전에 맞춘다.
- 권한 주기 - `chmod 755 h2.sh`
- 데이터베이스 파일 생성 방법
    - `jdbc:h2:~/querydsl` (최초 한번)
    - `~/querydsl.mv.db` 파일 생성 확인
    - 이후 부터는 `jdbc:h2:tcp://localhost/~/querydsl`로 접속