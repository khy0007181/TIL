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

## 검색 조건 쿼리

### 기본 검색 쿼리
- 검색 조건은 `.and()`, `or()`를 메서드 체인으로 연결할 수 있다.
- 참고) select와 from을 selectFrom으로 합칠 수 있다.
```java
@Test
public void search() {
    Member findMember = queryFactory
            .selectFrom(member)
            .where(member.username.eq("member1")
                    .and(member.age.eq(10)))
            .fetchOne();
    assertThat(findMember.getUsername()).isEqualT("member1");
}
```
<br>

### JPQL이 제공하는 모든 검색 조건 제공
```java
member.username.eq("member1") // username = 'member1'
member.username.ne("member1") // username != 'member1'
member.username.eq("member1").not() // username != 'member1'

member.username.isNotNull() // username is not null

member.age.in(10, 20) // age in (10, 20)
member.age.notIn(10, 20) // age not in (10, 20)
member.age.between(10,30) // between 10, 30

member.age.goe(30) // age >= 30
member.age.gt(30) // age > 30
member.age.loe(30) // age <= 30
member.age.lt(30) // age < 30

member.username.like("member%") // like 검색
member.username.contains("member") // like ‘%member%’ 검색
member.username.startsWith("member") // like ‘member%’ 검색

...

```
<br>

### AND 조건을 파라미터로 처리하는 방법
- and를 사용하지 않고 괄호 안에서 `,`로 구분
- and만 사용할 경우 유용하다.
```java
@Test
public void search() {
    Member findMember = queryFactory
            .selectFrom(member)
            .where(
                    member.username.eq("member1"),
                    member.age.eq(10)
            )
            .fetchOne();
    assertThat(findMember.getUsername()).isEqualT("member1");
}
```
<br>

## 결과 조회
- `fetch()` : 리스트 조회, 데이터 없으면 빈 리스트 반환
- `fetchOne()` : 단 건 조회
    - 결과가 없으면 : null
    - 결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException
- `fetchFirst()` : limit(1).fetchOne()
- `fetchResults()` : 페이징 정보 포함, total count 쿼리 추가 실행
    - `getResult()`, `getLimit()` ... 등으로 값을 받아서 사용
    - 쿼리가 2번 나간다.
    - 참고) paging 쿼리가 복잡할 경우, 컨텐츠를 가져오는 쿼리와 실제 totalCount를 가지고 오는 쿼리가 다를 경우가 있다. 
        - 성능 때문에 total을 가져오는 쿼리를 더 간단하게 만든다. 따라서 복잡할 경우 사용하면 안되고 쿼리를 따로 날려야 한다.
- `fetchCount()` : count 쿼리로 변경해서 count 수 조회
```java
@Test
public void resultFetch() {
    //List
    List<Member> fetch = queryFactory
            .selectFrom(member)
            .fetch();
    //단건
    Member findMember1 = queryFactory
            .selectFrom(member)
            .fetchOne();
    //처음 한 건 조회
    Member findMember2 = queryFactory
            .selectFrom(member)
            .fetchFirst();
    //페이징에서 사용
    QueryResults<Member> results = queryFactory
            .selectFrom(member)
            .fetchResults();
    //count 쿼리로 변경
    long count = queryFactory
            .selectFrom(member)
            .fetchCount();
}
```
<br>

## 정렬
- `desc()`, `asc()` - 일반 정렬
- `nullsLast()`, `nullsFirst()` - null 데이터 순서 부여
    - null이면 마지막에 출력, 처음 출력
```java
/**
 * 회원 정렬 순서
 * 1. 회원 나이 내림차순(desc)
 * 2. 회원 이름 올림차순(asc)
 * 단 2에서 회원 이름이 없으면 마지막에 출력(nullslast)
 */
@Test
public void sort() {
    em.persist(new Member(null, 100));
    em.persist(new Member("member5", 100));
    em.persist(new Member("member6", 100));
    
    List<Member> result = queryFactory
            .selectFrom(member)
            .where(member.age.eq(100))
            .orderBy(member.age.desc(), member.username.asc().nullsLast())
            .fetch();
    
    Member member5 = result.get(0);
    Member member6 = result.get(1);
    Member memberNull = result.get(2);
    assertThat(member5.getUsername()).isEqualTo("member5");
    assertThat(member6.getUsername()).isEqualTo("member6");
    assertThat(memberNull.getUsername()).isNull();
}
```
<br>
