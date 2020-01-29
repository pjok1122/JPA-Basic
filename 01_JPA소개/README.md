# JPA Basic

## JPA 소개

JPA는 `Java Persistence API`의 줄임말로 자바 진영의 ORM 기술 표준이다. 즉, JPA를 잘 이해하기 위해서는 `ORM 기술`을 이해해야 한다. ORM(`Object Relational Mapping`)은 객체와 관계형 데이터베이스의 데이터를 매핑시켜주며, 둘 사이에 발생하는 패러다임의 불일치를 해결해주는 기술이다.

자바에서 JPA를 구현한 구현체는 `하이버네이트`, `EclipseLink`, `DataNucleus` 세 가지가 있지만 가급적이면 자료가 많은 `하이버네이트`를 사용하는 것이 좋다.

<br>

![JPA](images/JPA.png)

사진에서 보는 것처럼 우리는 JPA에게 Entity Object를 넘겨주기만 하면, JPA는 내부적으로 쿼리문을 생성하고 JDBC API를 호출하여 DB에 접근한다. 이 사실도 물론 중요하지만, 더욱이 중요한 사실은 JPA가 `객체`와 `RDB` 사이의 패러다임의 불일치를 해결해준다는 사실이다.

## JPA 장점

JPA의 장점은 `생산성`, `유지보수`, `패러다임의 불일치 해결` 등 다양한 장점이 있다. 각 장점들을 기존의 JDBC나 MyBatis 프레임워크와 비교했을 때 어떤 차이가 있는지 간단히 살펴보자.

### 생산성

JPA에서는 SQL문을 개발자가 작성할 필요가 없다. JPA의 CRUD에 관한 코드를 간단히 살펴보자.

```java
jpa.persist(member) //저장

Member member = jpa.find(memberId) //조회

member.setName("변경할 이름") //변경

jpa.remove(member) //삭제
```

한 줄짜리 코드로 SQL문 없이 CRUD가 가능하다. 더욱 재밌는 사실은 `member.setName()`이다. 단순히 객체의 멤버변수를 변경했을 뿐인데, 데이터베이스에 저장된 값이 변경된다는 것이다. 이는 내부적으로 JPA가 AOP를 사용하고 있기 때문에 가능한 `마법`이라고 생각하고 넘어가자. 이러한 `마법` 덕분에 우리는 데이터베이스에 저장된 데이터들을 마치 `Collection<Mebmber>`에서 데이터를 꺼내고 변경하는 것과 같은 효과를 누릴 수 있다.


### 유지보수

![유지보수](images/JPA_유지보수.png)

이미 만들어져있는 Member 객체에 `tel`이라는 변수를 추가해야 한다고 하자. 만약 JDBC API를 사용한다면 보이는 것처럼 `CRUD`를 수정해야한다. 이는 굉장한 노가다에 가깝고, 사람이 직접 타이핑하는 SQL에는 실수가 따르기 마련이다.

하지만, JPA를 사용한다면, SQL문을 작성할 필요가 없기 때문에 Member라는 클래스에 단순히 `private String tel`만 추가해주면 모든 유지보수가 끝이 난다.

### 패러다임의 불일치 해결

<br>

#### JPA와 상속

![상속](images/JPA_상속.png)

객체지향프로그래밍에는 `상속`이라는 개념이 존재한다. 하지만 `RDBMS`에는 어떨까? 상속이라는 개념이 존재하지 않는다. 그럼, 객체가 상속을 받고있다면 RDB에서는 어떻게 처리해야할까?

JPA를 사용하지 않는다면 `album`이라는 객체를 RDB에 저장하기 위해 `ITEM` 테이블과 `ALBUM` 테이블에 대한 두 번의 `INSERT`문을 작성해야 한다. `albumId`로 앨범 객체를 조회할 때에는 `ALBUM`과 `ITEM`을 JOIN해서 Album 객체를 생성해내야 한다. 이는 굉장히 수고스러운 작업이고, 효율적이지도 못하다.

JPA를 사용한다면 개발자는 `jap.persist(album);`만 호출하면 데이터를 저장할 수 있고, `Album album = jpa.find(Album.class, albumId);` 한 줄의 코드로 객체를 넘겨받을 수 있다. 위에서 우리가 직접 구현해야 했던 작업들을 JPA가 전부 대신해준다는 의미이다.

<br>

#### JPA와 연관관계, 객체 그래프 탐색

```java
class Member{
    private String id;
    private Team team;
}
```

`Member` 객체가 `Team` 이라는 객체를 멤버변수로 가지고 있는 모습이다. 누군가가 이 Member에 대한 DAO를 만들었고, 우리가 이 API를 호출하고자 한다면 다음과 같이 호출할 것이다.

```java
class MemberService {
...
    public void process() {
    Member member = memberDAO.find(memberId); //비신뢰적!!
    Team team = member.getTeam(); //getTeam()하면 데이터가 들어있을까?
    }
}
```

우리는 여기서 memberDAO가 member 객체만 반환하는지, team 객체까지 반환하는지를 확인할 수가 없다. 즉 코드를 확인하기 전에는 신뢰할 수가 없다. 하지만 JPA를 사용하면 신뢰하고 사용할 수 있다.

또 JPA는 연관관계에 있는 객체를 RDB에 같이 저장해주는 기능이 있다. 따라서 다음과 같이 member라는 객체를 DB에 저장하면, team 또한 DB에 함께 저장된다.

```java
member.setTeam(team);
jpa.persist(member);
```

<br>

## JPA의 성능 최적화 기능

### 1차 캐시와 동일성 보장

JPA는 동일한 트랜잭션 내에서 조회한 엔티티는 반드시 같음을 보장해준다. 동일한 엔티티 조회의 경우, JPA는 이전의 조회 결과를 캐싱해서 돌려주기 때문에 동일한 엔티티를 갖는다고 볼 수 있다.

```java
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId); //SQL
Member member2 = jpa.find(Member.class, memberId); //캐시
member1 == member2; // true
```

### 트랜잭션을 지원하는 쓰기 지연 

#### INSERT

트랜잭션을 커밋할 때까지 INSERT SQL문을 모아두었다가 `JDBC BATCH SQL` 기능을 이용하여 한번에 SQL을 전송할 수 있다.

```java
transaction.begin(); // [트랜잭션] 시작
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
//커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

#### UPDATE, DELETE

UPDATE와 DELETE로 인한 로우 락 시간을 최소화 한다. 

```java
transaction.begin(); // [트랜잭션] 시작
changeMember(memberA);
deleteMember(memberB);
비즈니스_로직_수행(); //비즈니스 로직 수행 동안 DB 로우 락이 걸리지 않는다.
//커밋하는 순간 데이터베이스에 UPDATE, DELETE SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

### 지연 로딩과 즉시 로딩

![지연로딩_즉시로딩](images/지연로딩_즉시로딩.png)

지연로딩은 `team.getName()`을 호출했을 때 SELECT 쿼리를 전달해 엔티티를 가져오는 방법이고, 즉시 로딩은 member를 가져올 때, member와 관련있는 `team` 엔티티까지 함께 조회하는 방법을 의미한다. JPA는 옵션 하나로 지연 로딩과 즉시 로딩을 조절할 수 있다.

Member와 Team이 거의 항상 같이 조회된다면 즉시 로딩을 사용하는 편이 좋다.

