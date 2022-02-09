# 자바 ORM표준 JPA프로그래밍

ORM
- Object Relational Mapping
- 객체와 관계형 데이터 베이스 매핑

데이터 베이스 방언
- JPA는 특정 데이터 베이스에 종속되지 않는다.
- 각 데이터베이스가 제공하는 SQL문법과 함수는 조금씩 다르다.
- 방언(dialect) : SQL표준을 지키지 않는 특정 데이터 베이스만의 고유 기능
  - ex) org.hibernate.dialect.OracleDialect, org.,hibernate.dialectH2Database
  
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
- 비영속
  - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
- 영속
  - 영속성 컨텍스트에 의해 관리되는 상태
- 준영속
  - 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제
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

**영속 엔티티의 동일성 보장**

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

- 영속성 컨텍스트를비우지 않는다
- 변경내용을 데이터 베이스에 동기화
- 트랜잭션이라는 작업단위가 중요

### 준영속 상태
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 상태
- 영속성 컨텍스트가 더이상 관리하지 않으므로 변경감지등의 기능이 수행되지 않음

준영속 상태로 만드는 방법
- em.detach(entity) : 특정 엔티티 영속성 컨텍스트에서 분리
- em.clear() : 영속성 컨텍스트 모두 초기화
- em.close() : 영속성 컨텍스트 종료
