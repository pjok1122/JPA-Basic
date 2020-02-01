# 엔티티 매핑

## @Entity

`@Entity`가 붙은 클래스는 JPA가 관리하는 엔티티가 된다. 이 클래스는 데이터베이스의 테이블과 매핑된다.

## @Table

`@Table`은 엔티티와 매핑할 테이블을 지정할 수 있다.

- name : 매핑할 테이블 이름 (생략 가능)
- catalog : 데이터베이스 카탈로그 매핑
- schema : 데이터베이스 schema 매핑
- uniqueConstraints : DDL 생성 시에 유니크 제약조건 생성

```java
@Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE",
columnNames = {"NAME", "AGE"} )})
```

## hibernate.bhm2ddl.auto

`hibernate.bhm2ddl.auto` 속성을 지정하면 `@Entity`로 지정된 엔티티들을 테이블로 생성 및 삭제할 수 있다.

- create : 기존 테이블 삭제 후 생성
- create-drop : create와 같으나 종료 시점에 테이블 삭제
- update : 새로 생긴 제약조건이나 컬럼들을 반영한다. 컬럼이 삭제되는 경우에는 반영되지 않음.
- validate : 엔티티와 테이블이 정상 매핑이 되었는지만 확인한다.
- None : 사용하지 않음.

### 운영 서버에서는 반드시 validate나 None을 사용한다.

운영 서버에서 update 사용 시, 테이블 전체에 Lock이 걸려, 서버가 몇 분간 멈출 수 있기 때문에 권장되지 않음.

다른 개발자와 함께 사용하는 테스트 서버에서도 가급적이면 create, update는 사용하지 말자. update 쿼리문을 직접 작성해서 테스트 서버에 반영해본 후, 운영 서버에 반영해보는 것이 Best !

<br><hr>

## 필드와 컬럼 매핑

### @Column

테이블의 컬럼에 매핑할 때 사용한다. 여러가지 속성으로 길이, Null, Unique 등을 조절할 수 있다.

- `name` : 필드와 매핑할 테이블의 컬럼 이름
- `insertable`, updatable : 삽입과 변경이 가능한 지에 대한 여부
- `nullable` : `false`로 지정하면 `NOT NULL`이 DDL에 추가된다.
- `unique` : 필드에 유니크 제약조건을 걸지만, 제약조건 이름이 이상하므로 가급적 사용하지 않음. `@Table`을 사용하자.
- `columnDefinition` : 데이터베이스의 컬럼 정보를 직접 작성할 수 있다. ex) `varchar(255) default 'EMPTY'`
- `length` : 문자 길이 제약조건, `String` 타입에서만 사용한다.
- `precision`, `scale` : `precision`은 소수점을 포함한 전체 자릿수, `scale`은 소수의 자릿수를 나타낸다. 큰 숫자와 정밀한 소수를 다루어야할 때 사용한다.

```java


### @Enumerated

enum 타입을 매핑할 때 사용한다. 

- `value` : `EnumType.ORDINAL`, `EnumType.STRTNG` 두 가지 방법이 있는데, `ORDINAL`을 사용하면 데이터베이스에 enum의 순서를 저장하게 되고, `STRING`을 사용하게 되면 enum 이름을 사용하게 된다. 반드시 `STRING`을 사용하도록 하자.

```java
@Enumerated(EnumType.STRING)
private RoleType roleType;
```

### @Temporal

날짜 타입(`java.util.Date`, `java.util.Calendar`)을 매핑할 때 사용한다. 최신 하이버네이트 버전은 `LocalDate`, `LocalDateTime`을 지원한다.

- `TemporalType.Date` : 데이터베이스의 date 타입과 매핑 `(2020-2-1)`
- `TemporalType.TIME` : 시간, 데이터베이스 time 타입과 매핑 `(11:11:11)`
- `TemporalType.TIMESTAMP` : 날짜와 시간, 데이터베이스의 timestamp 타입과 매핑 `(2020-2-1 11:11:11)`

```java
@Temporal(TemporalType.TIMESTAMP)
private Date createdDate;
@Temporal(TemporalType.TIMESTAMP)
private Date lastModifiedDate;
```

```java
private LocalDateTime createDate;
private LocalDateTime lastModifiedDate;
```

### @LOB

데이터베이스의 BLOB, CLOB에 매핑된다. 문자열 타입이면 CLOB, 그 외에는 BLOB에 매핑된다.

```java
@LOB
private String description;
```

### @Transient

이 애노테이션이 붙은 필드는 테이블에 매핑되지 않는다. 임시로 어떤 값을 보관하고 싶을 때 사용.

```java
@Transient
private Integer temp;
```

