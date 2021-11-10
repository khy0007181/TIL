# 기본 문법

## JPQL vs Querydsl
- JPQL
    - 문자이므로 오류 발생 시 실행 시점에 발생
    - 파라미터 바인딩을 직접해야 한다.
- Querydsl
    - 코드이므로 오류 발생 시 컴파일 시점에 발생
    - 파라미터 바인딩이 자동 처리된다.
    - JPAQueryFactory를 필드 레벨로 사용할 수 있다.
        - 트랜잭션 마다 별도의 영속성 컨텍스트를 제공하기 때문에, 동시성 문제는 걱정하지 않아도 된다.
            - 멀티스레드 환경에서도 사용 가능
            - 문제없게 설계되어 있다. (Transaction 별로 바인딩되어 있다.)
```java
@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Autowired
    EntityManager em;

    JPAQueryFactory queryFactory;

    @BeforeEach
    public void before() {
        queryFactory = new JPAQueryFactory(em);
        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
        em.persist(teamA);
        em.persist(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 20, teamA);
        Member member3 = new Member("member3", 30, teamB);
        Member member4 = new Member("member4", 40, teamB);

        em.persist(member1);
        em.persist(member2);
        em.persist(member3);
        em.persist(member4);
    }

    @Test
    public void startJPQL() {
        //member1 찾기
        String qlString =
                "select m from Member m " +
                        "where m.username = :username";
        Member findMember = em.createQuery(qlString, Member.class)
                .setParameter("username", "member1")
                .getSingleResult();
        assertThat(findMember.getUsername()).isEqualTo("member1");
    }

    @Test
    public void startQuerydsl() {
        //member1 찾기
        JPAQueryFactory queryFactory = new JPAQueryFactory(em);
        QMember m = new QMember("m");
        Member findMember = queryFactory
                .select(m)
                .from(m)
                .where(m.username.eq("member1")) //파라미터 바인딩 처리
                .fetchOne();
        assertThat(findMember.getUsername()).isEqualTo("member1");
    }   
}
```
<br>

## 기본 Q-Type 활용

### Q클래스 인스턴스를 사용하는 2가지 방법
```java
QMember qMember = new QMember("m"); // 별칭 직접 지정
QMember qMember = QMember.member; // 기본 인스턴스 사용
```
<br>

### 기본 인스턴스를 static import와 함께 사용
- 가장 깔끔하고 권장된다.
- 같은 테이블을 join해야 하는 경우에만 별칭 사용하면 된다.
```java
import static com.example.querydsl.entity.QMember.*;

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    ...

    @Test
    public void startQuerydsl() {
        Member findMember = queryFactory
                .select(member)
                .from(member)
                .where(member.username.eq("member1"))
                .fetchOne();
        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}
```
<br>

### Querydsl 사용 시 실행되는 JPQL보는 방법
- Querydsl은 JPQL의 빌더 역할을 하는 것이다. 
- 결과적으로 Querydsl을 사용하는 코드는 JPQL이 된다.
- application.yml에 파일에 코드를 추가하면 확인할 수 있다.
    - `spring.jpa.properties.hibernate.use_sql_comments: true`
<br>
