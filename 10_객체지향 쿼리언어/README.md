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

## JPQL
