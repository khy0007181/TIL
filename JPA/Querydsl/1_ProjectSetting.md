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
- 최근 IntelliJ 버전은 Gradle로 실행을 하는 것이 기본 설정이다.
    - 실행속도가 느리기 때문에 다음과 같이 변경하면 자바로 바로 실행하므로 좀 더 빨라진다.
    - Preferences Build, Execution, Deployment Build Tools Gradle
    - Build and run using: Gradle IntelliJ IDEA
    - Run tests using: Gradle IntelliJ IDEA
<br>

### Lombok 적용
- 최근 Lombok이 plugin 설치에서 bundle로 변경되었다.
    - 설치할 필요 없음
- Preferences Annotation Processors 검색 Enable annotation processing 체크 (재시작)
- 임의의 테스트 클래스를 만들고 @Getter, @Setter 확인
<br>
