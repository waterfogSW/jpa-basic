# 자바 ORM표준 JPA프로그래밍

## JPA 소개

### SQL 중심적인 개발의 문제점

- 지금은 객체를 관계형 DB에 저장하여 관리하는 시대
- SQL의존적인 개발을 할 경우 추후 수정 변경이 어렵다.
- 패러다임의 불일치(객체지향 vs RDBMS)
    - 객체 저장의 현실적인 대안은 RDBMS이지만, 객체를 SQL로 변환하는 비용이 많이든다.
    - 상속관계
        - 테이블의 상속관계 구현의 어려움 -> 자바 컬렉션 조회
    - 연관관계
        - 객체는 참조
        - 테이블은 외래키
        - 객체모델링, 자바 컬렉션에서 관리
    - 객체 그래프 탐색
        - 객체는 자유롭게 객체 그래프를 탐색할 수 있어야 한다
- 객체답게 모델링 할수록 매핑 작업만 늘어난다

### JPA?

**ORM**

- Object Relational Mapping
- ORM 프레임워크가 객체와 관계형 데이터 베이스 매핑
- 패러다임 불일치 문제를 해결

**데이터 베이스 방언**

- JPA는 특정 데이터 베이스에 종속되지 않는다.
- 각 데이터베이스가 제공하는 SQL문법과 함수는 조금씩 다르다.
- 방언(dialect) : SQL표준을 지키지 않는 특정 데이터 베이스만의 고유 기능
    - ex) org.hibernate.dialect.OracleDialect, org.hibernate.dialectH2Database

### JPA의 성능 최적화 기능

**1. 1차 캐시와 동일성 보장**

- 같은 트랜잭션 안에서는 같은 엔티티를 반환(조회성능 향상)
- Repeatable Read보장

**2. 트랜잭션을 지원하는 쓰기 지연**

- 트랜잭션 커밋할대 까지 INSERT SQL을 모음
- JDBC BATCH SQL기능을 사용하여 한번에 전송

**3. 지연로딩, 즉시로딩**

- 지연로딩 : 객체 실제 사용될때 로딩
- 즉시로딩 : JOIN SQL로 한번에 연관된 객체까지 미리 조회

## Hello JPA 애플리케이션 개발

### 프로젝트 환경 설정(gradle)

**build.gradle**

```
    implementation 'org.hibernate:hibernate-entitymanager:5.3.10.Final'
    implementation 'com.h2database:h2:2.1.210'

    implementation group: 'javax.xml.bind', name: 'jaxb-api', version: '2.3.1'
    implementation group: 'com.sun.xml.bind', name: 'jaxb-core', version: '2.3.0.1'
    implementation group: 'com.sun.xml.bind', name: 'jaxb-impl', version: '2.3.1'

    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.7.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.7.0'
```

**h2 데이터베이스**

- 경로 : ~/test
- 데이터베이스 최초 생성시에는 경로에 tcp를 제거하고 접근한다.

*persistence.xml*

```text
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <class>hellojpa.Member</class>
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```

### 데이터 생성

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin(); // 트랜잭션 시작

        try {
            Member member = new Member();
            member.setId(2L);
            member.setName("HelloB");

            em.persist(member);

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}
```

### 데이터 조회

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();

        try {
            Member findMember = em.find(Member.class, 1L);
            System.out.println("findMember.id = " + findMember.getId());
            System.out.println("findMember.id = " + findMember.getId());

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}

```

### 데이터 수정

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();

        try {
            Member findMember = em.find(Member.class, 1L);
            findMember.setName("HelloJPA");
            //persist를 하지 않더라도 변경을 감지하여 커밋시점에 업데이트를 진행한다.

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}
```

### 데이터 삭제

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();

        try {
            Member findMember = em.find(Member.class, 1L);
            em.remove(findMember);

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}
```

### 주의사항

- 엔티티 매니저 팩토리는 하나만 생성하여 애플리케이션 전체에서 공유하여야 한다
- 엔티티 매니저는 스레드간에 공유해선 안된다
- JPA의 모든 데이터 변경은 트랜잭션 안에서 진행

### JPQL

- JPQL은 엔티티 객체를 대상으로 쿼리
- SQL은 데이터베이스 테이블 대상으로 쿼리
- 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리

## 영속성 관리

### 엔티티 매니저 팩토리, 엔티티 매니저

- EntityManagerFactory를 통해 고객의 요청이 올때마다 EntityManager를 생성한다.
    - EntityManagerFactory는 생성하는데 비용이 크기 대문에 애플리케이션 전체에서 한 번만 생성해 공유하도록 설계되어 있다.
    - Thread-Safe O
- EntityManager가 생성될 때 마다 하나의 영속성 컨텍스트가 생성되며, EntityManager를 통해 영속성 컨텍스트를 관리한다.
    - 생성하는데 비용이 거의 들지 않는다
    - Thread-Safe X

### 영속성 컨텍스트

- 엔티티를 영구 저장하는 환경
    - `EntityManager.persist(entity);`
- 논리적인 개념으로 눈에 보이지 않는다.
- 엔티티 매니저를 통해 영속성 컨텍스트에 접근한다.

### 영속성 컨텍스트의 이점

- 1차 캐시
    - 트랜잭션 마다의 별도의 캐시를 가짐
- 동일성 보장
    - 1차캐시에서 조회하여 동일한 객체임을 보장할 수 있음.
- 트랜잭션을 지원하는 쓰기 지연
    - 쓰기지연 SQL 저장소, flush
- 변경 감지
    - 스냅샷과의 비교를 통해 변경 감지
- 지연 로딩

### 엔티티의 생명 주기

- 비영속(new)
    - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
- 영속(managed)
    - 영속성 컨텍스트에 의해 관리되는 상태
- 준영속(detached)
    - 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed)
    - 삭제된 상태

```text
//비영속
Member member = new Member();
member.setId(100L);
member.setName("HelloJPA");

//영속
System.out.println("=== Before ===");
em.persist(member);

// 준영속
em.detach(member);

// 객체를 삭제한 상태
em.remove(member);
```

### 1차 캐시

```text
//엔티티를 생성한 상태(비영속)
Member member = new Member();
member.setId(100L);
member.setName("HelloJPA");

//1차 캐시에 저장됨
System.out.println("=== Before ===");
em.persist(member);

//1차 캐시에서 조회
Member findMember = em.find(Member.class, "member1");

//1차 캐시에 없음 -> DB에서 조회 - > DB에서 조회한 member2 를 1차캐시에 저장 -> 반환
Member findMember2 = em.find(Member.class, "member2");
```

- 1차캐시는 데이터 베이스의 한 트랜젝션에서만 효과가 있다.

### 영속 엔티티의 동일성 보장

```text
// 비영속
Member member = new Member();
member.setId(102L);
member.setName("HelloJPA");

// 영속
em.persist(member);

Member findMember1 = em.find(Member.class, 102L);
Member findMember2 = em.find(Member.class, 102L);

System.out.println("result = " + (findMember1 == findMember2));
```

결과 : result = true

- 1차 캐시로 반복 가능한 읽기 등급의 트랜잭션 격리 수준을 데이터 베이스가 아닌 애플리케이션 차원에서 제공한다.

### 트랜잭션을 지원하는 쓰기 지연

em.persist(memberA)

- 1차캐시에 memberA저장
- 쓰기 지연 SQL저장소에 INSERT SQL을 생성하여 쌓아둔다

transaction.commit();

- 쓰기 지연 SQL저장소의 SQL을 DB로 flush

```text
// 비영속
Member member1 = new Member(150L, "A");
Member member2 = new Member(160L, "B");

// 영속
em.persist(member1);
em.persist(member2);
System.out.println("============");

tx.commit();
```

결과 : 선을 그은후 쿼리가 발생하는것을 확인할 수 있음

```text
============
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```

### 변경 감지(Dirty Checking)

```text
Member member1 = em.find(Member.class, 150L);
member1.setName("ZZZZ");

System.out.println("============");

tx.commit();
```

결과 : `em.persist(member1)` 코드를 삽입하지 않더라도 commit시점에 update쿼리가 발생하는것을 볼 수 있음.

```text
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
============
Hibernate: 
    /* update
        hellojpa.Member */ update
            Member 
        set
            name=? 
        where
            id=?
```

1. flush()
2. 엔티티와 스냅샷 비교
3. 스냅샷과 차이가 있으면, UPDATE SQL을 생성하여 쓰기지연 SQL저장소에 쌓는다
4. flush : 업데이트 쿼리를 데이터 베이스에 반영
5. commit

값을 변경하면 커밋시점에 update쿼리가 발생하므로 persist하지 않는것이 좋다

### flush

영속성 컨텍스트의 변경 내용을 데이터 베이스에 반영

- em.flush() : 직접 호출
- 트랜잭션 커밋 : 자동 호출
- JPQL쿼리 실행 : 자동 호출
    - JPQL은 쿼리 생성 이전 flush를 수행후 쿼리를 생성한다.

```text
// 비영속
Member member = new Member(200L, "member200");
em.persist(member);
em.flush();

System.out.println("============");

tx.commit();
```

결과 : 커밋 이전에 호출되는것을 확인할 수 있음.

```text
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
============
```

**플러시 모드 옵션(웬만하면 손대지 말기)**

- FlushModeType.AUTO
    - 커밋이나 쿼리를 실행할 때 플러시(기본값)
- FlushModeType.COMMIT
    - 커밋할 때만 플러시

**플러시는!**

- 영속성 컨텍스트를비우지 않는다
- 변경내용을 데이터 베이스에 동기화
- 트랜잭션이라는 작업단위가 중요 -> 커밋 직전에만 동기화 되면 된다.

### 준영속 상태(Detached)

- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 상태
- 영속성 컨텍스트가 더이상 관리하지 않으므로 변경감지등의 기능이 수행되지 않음

준영속 상태로 만드는 방법

- em.detach(entity) : 특정 엔티티 영속성 컨텍스트에서 분리
- em.clear() : 영속성 컨텍스트 모두 초기화
- em.close() : 영속성 컨텍스트 종료

## 엔티티 매핑

- 객체, 테이블 매핑
    - @Entity, @Table
- 필드, 컬럼 매핑
    - @Column
- 기본키 매핑
    - @Id
- 연관관계 매핑
    - @ManyToOne, @JoinColumn

### 객체와 테이블 매핑

**@Entity**

- @Entity가 붙은 클래스는 JPA가 관리한다.
- JPA를 사용해서 테이블과 매핑할 클래스는 @Entity를 사용하여야 한다
- 주의점
    - 기본 생성자 필수(public, protected 생성자)
    - final클래스, enum, interface, inner클래스 사용X
    - 저장할 필드에 final 사용X
- `@Entity(name = "Member")`
    - JPA에서 사용할 엔티티의 이름을 지정할 수 있다.

**Table**

- 엔티티와 매핑할 테이블지정

### 데이터베이스 스키마 자동 생성

- DDL을 애플리케이션 실행시점에 자동 생성
- 테이블 중심 -> 객체 중심
- 데이터 베이스 방언을 활용하여 각 데이터 베이스에 맞는 적절한 DDL생성
- 생성된 DDL은 개발 장비에서만 사용

> DDL이란?  
> CREATE, ALTER, DROP, TRUNCATE, GRANT, REVOKE, COMMENT 과 같은 데이터 베이스의
> 스키마 객체를 생성하거나 삭제 하는 등의 작업을 하는 언어

persistence.xml

```text
...
<property name="hibernate.hbm2ddl.auto" value="create" />
...
```

**value 옵션**

- create
    - 기존테이블 삭제후 다시 생성(drop -> create)
- create-drop
    - create옵션과 같으나 종료시점에 drop(drop -> create -> drop)
    - 테스트 케이스를 실행시킬때 주로 사용
- update
    - 변경된 부분만 적용(alter)
- validate
    - 엔티티와 테이블이 정상 매핑되었는지 확인
- none
    - hibernate.hbm2ddl.auto 기능을 사용하지 않는다.(주석처리와 같음)

**주의**

- 운영장비에는 절대 create, create-drop, update를 사용해선 안된다.(데이터를 잃어버릴수도 있음)
- 개발 초기 단계는 create, update
- 테스트 서버는 update, validate
- 스테이징과 운영서버는 validate, none
- 가급적 여러명이 쓰는 개발서버, 스테이징 운영서버에서는 사용하지 않는다.

### DDL 생성 기능

- 제약 조건 추가
    - @Column(nullable = false, length = 10)
    - @Column(unique = true), : 해당 컬럼에 있어 존재하는 값이 유일해야 한다.

### 필드와 컬럼 매핑

#### 매핑어노테이션

```java

@Entity
public class Member {
    @Id
    private Long id;
    @Column(name = "name", insertable = true, updatable = true)
    private String username;
    private Integer age;
    @Enumerated(EnumType.STRING)
    private RoleType roleType;
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;
    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;
    @Lob
    private String description;

    public Member() {
    }

}
```

**@Column**

- name
    - 컬럼 이름
- insertable, updatable
    - 등록, 변경 가능 여부
- nullable
    - null값 허용 여부
- unique
    - uniuqe 제약조건
- columnDefinition
    - 데이터베이스 컬럼 정보를 직접 줄 수 있다
    - `columnDefinition = "varchar(100) default 'Empty'"`
- length
    - 문자 길이 제약 조건
- precision
    - BigDecimal타입에서 사용, 소수점을 포함한 전체 자릿수
- scale
    - 소수점 자릿수

**@Emumerated**

- EnumType.ORDINAL(기본 설정값) : enum타입의 순서가 데이터베이스에 저장된다(**사용하지 말것**)
    - enum타입을 추가할 경우 운영상에 문제가 발생할 수 있다.
- EnumType.STRING : emum타입 이름이 문자로 데이터베이스에 저장됨
    - 순서에 의한 문제가 없기 때문에 STRING으로 사용하는것을 권장

**@Temporal**

- 날짜 타입을 매핑할 때 사용
- LocalDAte, LocalDateTime을 사용할 때는 생략 가능

**@Lob**

- 매핑하는 필드 타입이 문자면 CLOB
- 나머지는 BLOB매핑
    - CLOB : String, char[], java.sql.CLOB
    - BLOB : byte[], java.sql.BLOB

**@Transient**

- 메모리에서만 임시로 사용

### 기본키 매핑

- 직접 할당 : @Id
- 자동 생성 : @GeneratedValue
    - IDENTITY : 데이터베이스에 위임, MYSQL - autoincrement
    - SEQUENCE : 데이터베이스 시퀀스 오브젝트 사용, oracle
        - @SequenceGenerator
    - TABLE : 키 생성용 테이블 사용, 모든 DB에서 사용가능
        - @TableGenerator
    - AUTO : 방언에 따라 자동 지정, 기본값

#### IDENTITY 전략

```java

@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private String id;
```

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();

        try {
            Member member1 = new Member();
            member1.setUsername("san-a");
            Member member2 = new Member();
            member2.setUsername("san-b");

            em.persist(member1);
            em.persist(member2);

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}
```

**결과**

```text
    create table Member (
       id varchar(255) generated by default as identity,
        name varchar(255) not null,
        primary key (id)
    )
```

- INSERT SQL실행 전까지 ID를 알 수 없다.
- 따라서 em.persist를 호출한 시점에 즉시 INSERT SQL을 실행하고 DB에서 식별자를 조회한다.

#### SEQUENCE 전략

```java

@Entity
@SequenceGenerator(
        name = "MEMBER_SEQ_GENERATOR",
        sequenceName = "MEMBER_SEQ", //매핑할 데이터베이스 시퀀스 이름
        initialValue = 1, allocationSize = 1
)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
            generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
```

**결과**

```text
Hibernate: create sequence hibernate_sequence start with 1 increment by 1
Hibernate: 
    
    create table Member (
       id bigint not null,
        name varchar(255) not null,
        primary key (id)
    )
```

- 데이터 베이스에 SEQUENCE 오브젝트를 생성하여 사용
    - SEQUENCE 오브젝트는 1부터 증가
- `call next value for MEMBER_SEQ`쿼리를 통해 다음 시퀀스 값을 DB로 부터 가져온 후 INSERT 쿼리 호출
- IDENTITY 방식과 달리 버퍼링이 가능
- 성능최적화 allocationSize
    - call next value로 인한 추가적인 네트워크 소요 -> allocationSize
        - allocationSize : 미리 allocationSize의 개수만큼 DB에 올려놓고, 쓰는방식
        - 동시성 이슈없이 문제 해결 가능

### TABLE 전략

- 키 생성 전용 테이블을 하나 만들어 데이터 베이스 시퀀스를 흉내내는 전략
- 장점 : 모든 데이터베이스에 적용 가능
- 단점 : 성능저하
- 실무에서는 관례상 DB에서 사용하는 것들이 있기 때문에 잘 사용하지 않는다.

```java

@Entity
@TableGenerator(
        name = "MEMBER_SEQ_GENERATOR",
        table = "MY_SEQUENCES",
        pkColumnValue = "MEMBER_SEQ", allocationSize = 1
)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
            generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
```

#### 권장하는 식별자 전략

- 기본키의 제약 조건
    - null X, 변하면 안된다
- 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. -> 대리키를 사용하자
- 권장 : Long형 + 대체키 + 키 생성전략 사용, autoincrement, sequence object사용

## 연관관계 매핑

- 용어 이해
  - 방향 : 단방향, 양방향
  - 다중성 : 다대일, 일대일, 다대다
  - 연관관계의 주인 : 객체의 양방향 연관관계는 관리하는 객체가 필요

**예제**
- 회원과 팀이 존재
- 회원은 하나의 팀에만 소속
- 회원과 팀은 다대일 관계

### 단방향 연관관계

**객체를 테이블에 맞추어 모델링 한 경우(외래키 식별자를 직접 다룸)**

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    @Column(name = "TEAM_ID")
    private Long teamId;
}
```

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();

        try {
            Team team = new Team();
            team.setName("TeamA");
            em.persist(team);

            Member member = new Member();
            member.setUsername("member1");
            member.setTeamId(team.getId());
            em.persist(member);

            Member findMember = em.find(Member.class, member.getId());

            Long findTeamId = findMember.getTeamId();
            Team findTeam = em.find(Team.class, findTeamId);

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}
```

- 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력관계를 만들 수 없다.
- 테이블은 외래키 조인을 사용하여 연관테이블을 찾는다
- 반면, 객체는 참조를 사용해 연관된 객체를 찾는다. -> 패러다임의 차이

**객체 지향 모델링**

```java

@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();

        try {
            Team team = new Team();
            team.setName("TeamA");
            em.persist(team);

            Member member = new Member();
            member.setUsername("member1");
            member.setTeam(team);
            em.persist(member);

            Member findMember = em.find(Member.class, member.getId());

            Team findTeam = findMember.getTeam();
            System.out.println("findTeam = " + findTeam.getName());

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}
```

### 양방향 연관관계와 연관 관계의 주인

```java

@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

```java

@Entity
public class Team {
    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>(); // null point error를 방지하기 위한 관례
}
```

- 객체 연관 관계 = 2개
    - 회원 -> 팀 연관관계 1개(단방향)
    - 팀 -> 회원 연관관계 1개(단방향)
- 테이블 연관 관계
    - 회원 <-> 팀 연관관계 1개(양방향)
- **객체의 양방향 관계**는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 2개이다.
- 반면, **테이블의 양방향 관계**는 외래키 하나로 두 테이블의 연관 관계를 관리(양쪽으로 조인 가능)

**연관 관계의 주인**

- 연관관계의 주인만이 외래 키를 관리(등록, 수정)
    - 주인이 아닌쪽은 읽기만 가능
- 주인은 mappedBy 속성 사용X
    - 주인이 아니면, mappedBy 속성으로 주인을 지정
- **외래키가 있는곳을 주인으로 정하자**

### 양방향 매핑시 많이 하는 실수

양방향 매핑시 연관관계의 주인에 값을 입력해야 한다.

(주인에 값을 입력하지 않은 경우)

```java
public class JpaMain {
    public static void main(String[] args) {
        //...
        Team team = new Team();
        team.setName("TeamA");
        em.persist(team);

        Member member = new Member();
        member.setName("member1");

        //역방향(주인이 아닌 방향)만 연관관계 설정
        team.getMembers().add(member);
        em.persist(member);
    }
}
```

(주인에도 값을 입력한 경우)

```java
public class JpaMain {
    public static void main(String[] args) {
        //...
        Team team = new Team();
        team.setName("TeamA");
        em.persist(team);

        Member member = new Member();
        member.setName("member1");

        team.getMembers().add(member);
        //연관관계의 주인에도 값 설정
        member.setTeam(team);
        em.persist(member);
    }
}
```

#### 양방향 연관관계 주의

- 순수한 객체 관계를 고려하면 항상 **양쪽다** 값을 입력해야 한다.
    - Member(주인), Team(역방향) 둘다 값을 넣어 주어야 한다.
- 연관 관계 편의 메서드를 생성하자(둘중 하나에만 만드는것이 좋다)

**Member에 만드는 경우**

```java

@Entity
public class Member {
    //...
    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
    //...
}
```

**Team에 만드는 경우**

```java

@Entity
public class Team {
    //...
    public void addMember(Member member) {
        member.setTeam(this);
        members.add(member);
    }
    //...
}
```

- 양방향 매핑시에 무한 루프를 조심하자
  - toString(), lombok, JSON생성 라이브러리 -> stack overflow, 장애 발생
  - Controller에서 엔티티 반환하지 말기, DTO로 변환하여 반환할것

#### 양방향 매핑 정리

- 단방향 매핑만으로도 이미 연관관계 매핑은 완료된다.
    - 가급적이면 설계시에는 단방향 매핑으로, 필요시 양방향 매핑 진행
- 단방향 매핑을 잘하고 양방향은 필요할 때 추가해도 된다(테이블에 영향을 주지 않음)

#### 연관관계의 주인을 정하는 기준

- 비즈니스 로직을 기준으로 주인을 선택하면 안됨.
- 외래키의 위치를 기준으로 정해야 한다.

## 다양한 연관관계 매핑

### 연관관계 매핑시 고려사항 3가지

- 다중성(1:1, 1:N, N:M)
    - 1:N : @OneToMany
    - N:1 : @ManyToOne
    - 1:1 : @OneToOne
    - N:M : @ManyToMany(실무에서 사용해서는 안됨)
- 단방향, 양방향
    - 테이블
        - 외래키 하나로 양쪽 조인 가능
        - 방향이라는 개념이 존재하지 않음
    - 객체
        - 참조용 필드가 있는 쪽으로만 참조 가능
        - 한쪽만 참조하면 단방향
        - 양쪽이 서로 참조하면 양방향
- 연관관계의 주인
  - 테이블은 외래키 하나로 두 테이블이 연관 관계를 맺음
  - 반면 객체는 참조가 2군데있음. 테이블의 외래키를 관리할 곳을 지정해야 한다.
  - 연관관계의 주인 : 외래 키를 관리하는 참조
  - 주인의 반대편 : 외래키에 영향을 주지 않음

### 다대일 (N : 1)

#### 다대일 단방향

```java

@Entity
public class Member {
    //...
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    //...
}
```

- 외래키가 있는곳(Member)에 참조를 걸면 된다.
- 가장 많이 사용하는 연관 관계

#### 다대일 양방향

- Team에서도 member를 찾을 수 있도록함.
- 테이블에 영향을 주지 않음

```java

@Entity
public class Team {
    //...
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
    //...
}
```

- 외래키가 있는쪽이 연관관계의 주인
- 양쪽이 서로 참조하도록 개발

### 일대다 (1:N)

- TEAM 을 중심으로 외래키를 관리, 연관관계의 주인이 TEAM(실무에서 권장하지 않음)

#### 일대다 단방향

```java

@Entity
public class Team {
    //...
    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();

    //...
}
```

```java

@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

//    주석처리
//    @ManyToOne
//    @JoinColumn(name = "TEAM_ID")
//    private Team team;
}
```

- 일(1)이 연관관계의 주인
- 테이블 일대다 관계는 항상 다(N) 쪽에 외래키가 있음
- 객체와 테이블의 차이때문에 반대편 테이블의 외래키를 관리하는 특이한 구조
- @JoinColumn을 꼭 사용해야 한다. 그렇지 않으면 조인 테이블 방식을 사용함
    - TEAM_MEMBER 이라는 중간 테이블을 생성하여 관리됨

**단점**

- 엔티티가 관리하는 외래키가 다른 테이블에 있음 -> 추가적인 update쿼리 발생
- 추가적인 업데이터 쿼리가 발생하여 추후 유지 보수, 추적 관리에 어려움을 겪을 수 있으므로 실무에서는 권장하지 않음
- 객체적으로 손해를 보더라도 다대일 양방향 매핑을 사용하는것을 권장

#### 일대다 양방향

```java

@Entity
public class Member {
    //...

    @ManyToOne
    @JoinColumn(insertable = false, updatable = false)
    private Team team;
    //...
}
```

- 공식적으로 존재하지 않음
- Member의 team은 읽기 전용 필드로 사용. 양방향 처럼 사용
- 다대일 양방향을 사용하자

