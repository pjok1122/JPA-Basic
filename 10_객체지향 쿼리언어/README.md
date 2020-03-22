# 객체지향 쿼리언어 JPQL

## JPA에서 사용할 수 있는 쿼리 방법들

JPA는 다양한 쿼리 방법들을 지원한다. 이 다양한 방법들을 한 번 살펴보고 특징을 파악해보자.

### JPQL

- JPQL은 `객체 지향 쿼리 언어`로 **테이블이 아닌 엔티티를 대상**으로 검색 쿼리를 작성한다는 특징이 있다. JPQL로 작성된 쿼리는 SQL 쿼리로 변경되어 JDBC 통신을 하게 된다.

- JPQL은 SQL문법과 유사하며 ANSI 표준에 정의되어있는 키워드는 전부 구현이 되어있다. 예를 들자면, `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `JOIN`을 지원한다.

- JPQL로 작성된 쿼리는 특정 데이터베이스에 의존하지 않는다. 따라서 `MySQL`을 사용하다가 `Oracle`로 변경해도 문제없이 동작한다.

- JPQL은 동적쿼리를 작성하기가 어렵다는 단점이 있다. 동적쿼리를 작성하려면 `query`를 끊어서 `+`연산을 이용해야 하는데, 이는 대단히 비효율적이다. 동적쿼리를 작성해야 한다면 일반적으로 `QueryDSL`이라는 오픈소스 라이브러리를 사용한다.

**예시 코드**

```java
List<Member> result = em.createQuery("select m from Member m where m.name like '%hello%'", Member.class)
                        .getResultList();
```

- `em.createQuery()`에 반환 타입을 명시하면 `TypedQuery<Object>`를 반환하고, 명시하지 않으면 `Query`를 반환한다. 둘 다 메소드 체이닝을 이용해서 사용하는 것이 편리하다.

### JPA Criteria

동적쿼리를 작성할 수 있게 도와주는 표준 기술이지만, 실무에서는 적합하지 않다. 그 이유는 사용방법이 복잡하여 유지보수가 힘들기 때문이다. 이거 쓰지말고 `QueryDSL` 쓰는 것이 좋다. 간단한 사용 방법만 구경하고 넘어가자.

```java
//Criteria 사용 준비
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);
//루트 클래스 (조회를 시작할 클래스)
Root<Member> m = query.from(Member.class);
//쿼리 생성 CriteriaQuery<Member> cq =
query.select(m).where(cb.equal(m.get("username"), “kim”));
List<Member> resultList = em.createQuery(cq).getResultList();
```

- `select`, `from`, `where`을 따로따로 처리할 수 있지만 코드의 가독성이 상당히 떨어진다.

### QueryDSL

- `JPA Criteria`와 마찬가지로 Java 코드로 SQL문을 작성할 수 있어, 컴파일 시점에 문법적인 오류를 찾을 수 있다.

- `Criteria`와 동일하게 JPQL 빌더 역할을 하지만, 사용법이 단순하다.

- 실무에서 권장되는 동적 쿼리 작성 방법이다.

**예시 코드**

```java
JPAFactoryQuery query = new JPAFactoryQuery(em);
QMember m = QMember.member;

List<Member> list = query.selectFrom(m)
                    .where(m.age.gt(18))
                    .orderBy(m.name.desc())
                    .fecth();
```

### Native SQL

JPA가 제공하는 SQL을 직접 작성하는 기능이다. JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능을 사용해야 할 때 사용한다. 하지만 이 마저도 `JdbcTemplate`을 사용하는게 더 나아보인다.

**예제 코드**

```java
String sql = "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = 'kim'"
List<Member> resultList = em.createNativeQuery(sql, Member.class)
                            .getResultList();
```

### JDBC 직접 사용. Spring JdbcTemplate, Mybatis 등

- JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, Mybatis 등을 함께 사용할 수 있다.
- 앞서 살펴본 쿼리 작성법들은 쿼리 발생 시, 영속성 컨텍스트를 `flush`하고 데이터를 조회한다.
- 반면, JDBC는 JPA 기술이 아니기 때문에, 수동으로 영속성 컨텍스트를 `flush`하고 쿼리를 실행해야 올바른 결과를 얻을 수 있다.

### 결론

실무에서는 `JPQL`과 `QueryDSL`을 주로 사용하므로 이 둘을 중점적으로 공부하자. 특히, `QueryDSL`은 `JPQL`에 의존하고 있기 때문에 `JPQL`을 명확히 공부해두는 것이 좋다.

<br><hr>

## JPQL 기본 문법과 기능

### JPQL 특징

- JPQL은 객체 지향 쿼리 언어로, 테이블이 아닌 엔티티를 대상으로 쿼리한다.
- JPQL은 SQL을 추상화해서 특정데이터베이스에 의존하지 않는다.
- JPQL은 결국 SQL로 변환된다.

### JPQL 문법

**JPQL 작성방법**

```java
Strgin str = "select m from Member (as) m where m.age > 18";
List<Member> list = em.createQuery(str, Member.class)
                    .getResultList();
```

- 엔티티와 속성은 대소문자를 구분한다. ex) Member, age
- JPQL 키워드는 대소문자를 구분하지 않는다. ex) SELECT, select
- Member는 테이블 이름이 아닌 엔티티의 이름을 사용한다. ex) @Entity(name="Member")
- 엔티티에 대한 별칭 사용은 필수이며, as는 생략이 가능하다.

<br>

**반환타입 구분**

```java
//반환 타입이 명확할 때
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class)

//반환 타입이 명확하지 않을 때
Query query = em.createQuery("select m.username, m.age from Member m");
```

- 반환타입이 명확할 때는 `TypedQuery<Object>`를 사용하고, 명확하지 않을 때는 `Query`를 사용한다.

<br>

**결과 조회 API**

```java
//반환 결과가 여러 개 일 때,
List<Member> list = em.createQuery("select m from Member m", Member.class)
                    .getResultList();

//결과가 정확히 하나일 때,
Member member = em.createQuery("select m from Member m", Member.class)
                    .getSingleResult();
```

- 결과가 하나이상일 때는 `getResultList()`를 사용한다. 만약 반환 결과가 없다면 빈 리스트를 반환해주기 때문에 null에 대한 대응을 할 필요가 없다.

- `getSingleResult()`는 반환 결과가 정확히 하나일 때만 사용한다. 만약 결과가 없다면 `NoResultException` 예외가 발생하고, 결과가 두 개 이상이라면 `NonUniqueResultException`이 발생한다. 반환 결과가 없을 때 예외를 발생시키는 것이 단점이다.

- `getSingleResult()`는 예외가 많이 발생하기 때문에 `Spring data JPA`에서는 `getSingleResult()`를 좀 변형(?)시켜서 결과가 없을 때 null을 반환하도록 만들었다.

<br>

**파라미터 바인딩**

```java
//이름을 기준으로 파라미터를 바인딩한다.
em.createQuery("select m from Member m where m.username=:username", Member.class)
.setParameter("username", usernameParam);

//위치를 기준으로 파라미터를 바인딩한다.
em.createQuery("select m from Member m where m.username=?1", Member.class)
.setParameter(1, usernameParam);
```

- 위치를 기준으로 바인딩하는 방식은 파라미터의 개수가 추가되거나 위치가 바뀔 시 버그가 발생할 소지가 높다.
- 따라서 위치를 기준으로 파라미터를 바인딩하는 방식을 지양하고 이름을 사용하는 방식을 지향하자.

<br>

### 프로젝션

프로젝션이란 select 절에 조회할 대상을 지정하는 것을 말한다. 프로젝션의 대상으로는 `엔티티`, `임베디드 타입`, `스칼라 타입`이 올 수 있다.

**프로젝션 종류**

```java
//엔티티 프로젝션
em.createQuery("select m from Member m", Member.class)                  (1)
em.createQuery("select m.team from Member m", Team.class)               (2)
em.createQuery("select t from Member m join m.team t", Team.class)      (3)

//임베디드 타입 프로젝션
em.createQuery("select m.address from Member m", Address.class)

//스칼라 타입 프로젝션
em.createQuery("select distinct m.username, m.age from Member m")
```

- 엔티티 프로젝션 (2)는 JPA에서 내부적으로 `Inner join` 쿼리가 발생한다. 하지만 JPQL만 봤을 때, join 쿼리가 발생한다는 것을 예측하기가 어렵다. 따라서 가급적 (3)처럼 실제 sql과 비슷하게 코드를 작성하는 것이 추후에 튜닝이나 성능개선을 생각했을 때 유리하다.

- 임베디드 타입은 엔티티가 아니기 때문에 `from` 절에 사용할 수 없다는 한계점이 있긴 하다.

- distinct를 사용해서 중복을 제거할 수도 있다.

- **프로젝션의 결과는 영속성 컨텍스트에 관리되기 때문에 엔티티를 update 하는 경우 update 쿼리가 발생한다.**

<br>

**프로젝션 여러값 조회**

`TypedQuery`인 경우에는 결과 조회에 `Generic`을 이용하면 손쉽게 원하는 결과를 얻을 수 있다. 이번에는 `Query`를 사용하는 경우에 결과를 어떻게 받아야 하는지에 대해 알아보자.

```java

//Object[] 타입으로 조회
List<Object[]> resultList = em.createQuery("select m.username, m.age from Member m")
                                .getResultList();

for(Object[] result : resultList){
    System.out.print("username : " + result[0])
    System.out.println(", age : " + result[1])
}


//new 명령어로 조회
List<MemberDto> resultList = em.createQuery("select new package.path.MemberDto(m.username, m.age) from Member m", MemberDto.class);
                                .getResultList();

for(MemberDto member : resultList){
    System.out.print("username : " + member.getUsername())
    System.out.println(", age : " + member.getAge())
}
```

- Object[] 타입으로 조회하는 것보다는 객체를 새로 정의하여 new 명령어를 사용하는 편이 낫다. new 명령어 뒤에는 객체의 생성자를 호출하는 것과 동일하기 때문에 패키지 경로를 작성해야 한다.

- new 명령어를 사용할 때의 단점은 패키지 명을 포함한 전체 클래스를 작성해야 하기 때문에 쿼리의 길이가 길어질 수 있다. 또 다른 단점으로는 생성자를 호출하는 것과 동일하기 때문에 조회하고자 하는 데이터의 순서와 타입이 일치해야 한다.

- QueryDSL을 이용하면 보다 쉽게 작성이 가능하다고 한다.

<br>

### 페이징 API

JPA는 아주 편리한 페이징 API를 제공한다.

**예시 코드**

```java
String jpql = "select m from Member m order by m.age desc";
List<Member> members = em.createQuery(jpql, Member.class)
                        .setFirstResult(0)
                        .setMaxResults(10)
                        .getResultList();
```

- `setFirstResult()`에는 가져올 데이터의 순번을 넘겨준다. 인덱스는 0부터 시작한다.
- `setMaxResults()`에는 가져올 데이터의 개수를 넘겨준다.
- 페이징은 데이터베이스의 종류에 따라 방법이 아주 다양하다. 하지만 JPA의 페이징 API를 이용하면 모든 데이터베이스에 대해서 동일한 방식으로 사용이 가능하다.

**페이징 SQL 쿼리**

```sql
# MySQL
SELECT
    M.ID AS ID,
    M.AGE AS AGE,
    M.TEAM_ID AS TEAM_ID,
    M.NAME AS NAME
FROM MEMBER M
ORDER BY M.NAME DESC LIMIT ?, ?

# Oracle
SELECT * FROM
    (
        SELECT ROW_.*, ROWNUM ROWNUM_
        FROM
            (
                SELECT
                    M.ID AS ID,
                    M.AGE AS AGE,
                    M.TEAM_ID AS TEAM_ID,
                    M.NAME AS NAME
                FROM MEMBER M
                ORDER BY M.NAME
            ) ROW_
        WHERE ROWNUM <= ?
    )
WHERE ROWNUM_ > ?
```

- Oracle은 페이징 하나 구현하기 위해 3번의 Select 중첩이 일어나는 모습이다.. JPA의 페이징은 혁신이다..!

<br><hr>

### 조인

**JPA 조인의 종류**

```java
//내부 조인[INNER JOIN]
String query = "select m from Member m [inner] join m.team t";

//외부 조인[OUTER JOIN]
String query = "select m from Member m left [outer] join m.team t";

//세타 조인
String query = "select m from Member m, Team t where m.username = t.name";
```

- 세타 조인은 `cross join` 쿼리가 발생한다. 즉, 카테시안 곱 연산이 이루어지며, 연관관계가 없는 두 엔티티를 엮을 때 사용할 수 있다.

<br>

**ON절을 활용한 필터링**

```java
//JPQL
String query = "select m from Member m left join m.team t on t.name='teamA'";
```

```sql
#실행되는 SQL
SELECT m.*
FROM MEMBER m
LEFT JOIN TEAM t
ON m.TEAM_ID = t.ID AND t.NAME = 'teamA';
```

- 회원과 팀을 조인하면서, 팀 이름이 'teamA'인 팀만 조인할 수 있다.

**연관관계가 없는 엔티티 외부 조인**

```java
//JPQL
String query = "select m from Member m left join Team t on m.username = t.name";
```

```sql
#실행되는 SQL
SELECT m.*
FROM MEMBER m
LEFT JOIN TEAM t
ON m.username = t.name;
```

- 회원의 이름과 팀의 이름은 아무런 연관관계가 없다. `(JPA 2.1, Hibernate 5.1)` 부터는 아무런 연관관계가 없는 두 엔티티를 조인하고자 할 때, `on` 절을 활용할 수 있다.

- `연관관계`가 있는 경우에는 `join` 대상에 연관관계가 있는 엔티티를 넘겨주지만, `연관관계`가 없는 경우에는 `엔티티` 이름을 넘겨준다는 특징이 있다.

<br>

### 서브쿼리

JPA는 `NOT`, `EXISTS`, `ALL`, `ANY`, `SOME`, `IN` 이라는 키워드를 제공한다.

- `EXISTS` : 서브쿼리에 결과가 존재하면 참
- `ALL` : 모두 만족하면 참
- `ANY`,`SOME` : 조건을 하나라도 만족하면 참
- `IN` : 서브쿼리의 결과 중 하나라도 같은 값이 있으면 참

```java
//나이가 평균보다 많은 회원
String query = "select m from Member m
                where m.age > (select avg(m2.age) from Member m2)";

//한 건이라도 주문한 고객
String query = "select m from Member m
                where (select count(o) from Order o where m = o.member) > 0";

//teamA 소속인 회원 조회
String query = "select m from Member m
                where exists (select t from m.team t where t.name = 'teamA')";

//전체 상품 각각의 재고보다 주문량이 많은 주문들 조회
String query = "select o from Order o
                where o.orderAmount > (select p.stockAmount from Product p)";

//어떤 팀이든 팀에 소속된 회원
String query = "select m from Member m
                where m.team = any (select t from Team t)";
```

- JPA는 `WHERE`, `HAVING` 절에만 서브 쿼리 사용이 가능하다. JPA의 구현체인 `Hibernate`에서는 `SELECT`에서도 서브 쿼리를 사용할 수 있다.

- 현재 JPQL은 `FROM` 절의 서브 쿼리는 지원하지 않는다. 가능하면 `JOIN`을 이용해서 문제를 해결하거나, 어플리케이션의 비즈니스 로직에서 문제를 처리하는 것이 좋다.

<br>

### JPQL 타입 표현

- 문자 : `'Hello'`, `'She''s'`
- 숫자 : `10L`, `10D`, `10F`
- Boolean : `TRUE(true)`, `FALSE(false)`
- Enum : `jpabook.MemberType.Admin` (패키지명 포함)
- Entity : `Type(m) = Member` (상속 관계에서만 사용)

<br>

### 조건식 - Case

**Case문 예시**

```sql
#기본 CASE
SELECT
    CASE WHEN m.age <= 10 then '학생요금'
         WHEN m.age >= 60 then '경로요금'
         ELSE '일반요금'
    END
FROM MEMBER m
```

```sql
#단순 CASE
SELECT
    CASE t.name
        WHEN '팀A' THEN '인센티브110%'
        WHEN '팀B' THEN '인센티브120%'
        ELSE '인센티브105%'
    END
FROM TEAM t
```

- CASE 문은 위의 상황처럼 분기를 이용할 때 사용한다.

<br>

**coalesce와 nullif**

```java
//사용자 이름이 없으면, '이름 없는 회원'을 반환한다.
String query = "select coalesce(m.username, '이름 없는 회원') as username from Member m";
List<String> result = em.createQuery(query, String.class)
                        .getResultList();

//사용자 이름이 '관리자'이면 null을 반환하고 나머지는 본인의 이름을 반환한다.
String query = "select nullif(m.username, '관리자') from Member m";
List<String> result = em.createQuery(query, String.class)
                        .getResultList();
```

- `coalesce`는 하나씩 조회해서 null이 아니면 반환한다.
- `nullif`는 두 값이 같으면 null을 반환한다. 다르면 첫 번째 값을 반환.

<br>

### JPQL 함수

#### JPQL 기본 함수

- `CONCAT`
- `SUBSTRING`
- `TRIM`
- `LOWER`, `UPPER`
- `LENGTH`
- `LOCATE`
- `ABS`, `SQRT`, `MOD`
- `SIZE`

#### 사용자 정의 함수

하이버네이트는 사용 전 방언에 추가해야 한다. 사용하는 DB의 방언을 상속받고, 사용자 정의 함수를 등록한다. 마지막으로 생성한 객체를 `persistence.xml`에 등록하면 된다.

```java
//H2Dialect를 상속받은 객체에 사용하고자 하는 함수를 추가한다.
public class MyH2Dialect extends H2Dialect{
    public MyH2Dialect(){
        registerFunction("group_concat", new StandaradSQLFunction("group_concat", StandardBasicTypes.STRING))
    }
}

//새로 생성된 객체 MyH2Dialect를 persistence.xml에 추가한다.
<property name="hibernate.dialect" value="dialect.MyH2Dialect">


//표준 방법으로 사용할 때는 function()을 이용하고, hibernate에서는 등록된 함수를 바로 사용할 수 있도록 도와준다.
String query = "select function('group_concat', m.username) from Member m";
String query = "select group_concat(m.username) from Member m";
List<String> result = em.createQuery(query, String.class)
                        .getResultList();
```
