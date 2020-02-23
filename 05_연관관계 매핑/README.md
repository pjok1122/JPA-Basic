# 연관관계 매핑

객체의 참조와 테이블의 참조의 차이를 이해하고 이를 극복하는 방법에 대해서 알아보자.

## 잘못된 설계

객체를 테이블에 맞춰서 설계하면 다음과 같다. Member는 Team에 대한 외래키를 가지고 있는 모습이다.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    @Column(name = "USERNAME")
    private String name;
    @Column(name = "TEAM_ID")
    private Long teamId;
    …
}
@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;
    private String name;
…
}
```

위와 같은 방식으로 코드를 설계하게 되면 '객체지향'이 아닌 코드를 작성하게 된다. 객체와 객체가 연결되어있지 않다는 것이 보인다. 따라서 이러한 문제점을 극복하는 방법이 있다.

## 단방향 매핑(@ManyToOne)

기존의 Entity 모델에서 FK부분을 @ManyToOne 이라는 애노테이션을 사용하여 객체를 주입한다. 코드로 살펴보자.

```java

@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    @Column(name = "USERNAME")
    private String name;

    //========= 차이점 ===========
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    //===========================
    …
}
@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;
    private String name;
…
}
```

@ManyToOne을 이용하여 다대일 관계를 나타내었고, 멤버변수로 `Team` 객체를 둔 것을 확인할 수 있다. 그럼에도 테이블에 문제가 없는 이유는 `@JoinColumn`을 이용해서 테이블에서 누구와 조인을 해야하는지를 명시했기 때문이다.

객체를 이렇게 설계하면 객체지향스럽게 사용이 가능하다. 간단한 삽입 조회 코드를 살펴보자.

```java

EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
transaction.begin();

try{
    //team 영속성 관리
    Team team = new Team();
    team.setName("Team1");
    em.persist(team);

    //member 영속성 관리
    Member member = new Member();
    member.setUsername("member1");
    member.setTeam(team);
    em.persist(member);

    em.flush()      //1차 캐시 DB에 반영
    em.clear()      //1차 캐시 비우기.

    //DB 조회
    Member findMember = em.find(Member.class, member.getId());
    //객체 참조
    Team findTeam = findMember.getTeam();

    transaction.commit();
} catch(Exception e){
    transaction.rollback();
} finally{
    em.close();
}

emf.close();
```

간만에 전문 코드를 작성했는데, 핵심은 try{} 부분만 보면 된다. 특히 관심있게 볼 부분은 `member.setTeam(team)`을 사용해서 team을 저장하는 부분과 `findMember.getTeam()`을 이용해서 Team 객체를 불러왔다는 점이다.

**em.persist(team), em.persist(member) 둘 다 영속성으로 관리하지 않고 Member만 관리하는 경우 DB에 반영되지 않는 문제가 있음.**

## 양방향 매핑(@ManyToOne, @OneToMany)

이미 위에서 살펴본 것처럼 설계해도 문제없이 잘 동작한다. 즉, 설계는 이미 끝났다. 그런데도 사실 테이블과 객체 사이에는 차이점이 남아있다.

테이블의 관점에서 봤을 때는 `Member`를 기준으로 `Team`과 Join할 수 있고 `Team`을 기준으로 `Member`와 Join하는 것도 가능하다. 하지만 위의 설계는 `Member`에서 `Team`을 조인(조회)하는 것은 가능하지만, `Team`에서 `Member`를 조인(조회)하는 것은 불가능하다. 이러한 차이점을 극복하기 위해서는 `가짜 매핑(mappedBy)`을 이용해야 한다.

이런 양방향 매핑은 Entity 설계 당시부터 고민할 필요는 없다. **우선 단방향으로 설계하고 역방향 조회가 필요한 상황이 자주 나타난다면, 그때가서 양방향으로 수정해도 늦지 않는다.**

Entity 코드를 살펴보자.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    @Column(name = "USERNAME")
    private String name;

    //=======연관관계 주인========
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    //===========================
    …
}
@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;
    private String name;

    //====== 가짜 매핑 ===========
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();
    //===========================
…
}
```

@ManyToOne을 가지고 있는 엔티티(외래키를 가지고 있는 테이블)를 `연관관계 주인`이라고 부르며, `mappedBy`를 사용하는 엔티티를 `가짜 매핑`이라고 부른다.

### 많이 하는 실수

`연관관계의 주인(Member)`에서 `team`을 변경하면 테이블에 반영되지만, `주인이 아닌 객체(Team)`에서 `members`를 변경하면 테이블에 반영되지 않는다.

```java
Team team = new Team();
team.setName("team1");
em.persist(team);

Member member = new Member();
member.setName("member1");

//역방향(주인이 아닌 방향)만 설정
team.getMembers().add(member);

em.persist(member);
```

이런 경우 Member 테이블에는 변화를 주지 못하므로 `member.getTeam()` 값은 NULL이다.

따라서 반드시 연관관계 주인에도 값을 설정하도록 한다.

```java
member.setTeam(team);
team.getMembers().add(member);
```

매번 이렇게 작성하는 것보다는 `연관관계 편의 메소드`를 만들어 한 번에 처리하는 것이 안전하다.

```java
public class Member {
    ...

    public void changeTeam(Team team){
        setTeam(team);
        team.getMembers().add(this);
    }
```

**역방향에 값을 설정안해줘도 동작하지만, 1차 캐시에서 조회되는 경우 잘못된 데이터를 가져올 수 있으므로 가급적이면 편의 메소드를 사용하자.**

## 양방향 연관관계 주의사항

양방향 매핑 시에 무한루프를 조심하자.

1. 양방향 관계에 있으므로 `toString()`을 단순 구현하면 무한루프에 빠지게 된다.

2. JSON 생성 라이브러리를 사용할 때에도 서로가 서로를 참조하고 있으므로 무한루프에 빠질 수 있다.

3. Controller에서 Entity를 통째로 반환하지말고 DTO를 만들어서 반환하는 것이 좋다. 테이블의 구조가 바뀌면 API 스펙이 달라지는 문제가 발생할 수 있기 때문!
