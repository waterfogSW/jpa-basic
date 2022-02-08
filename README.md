# 자바 ORM표준 JPA프로그래밍

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

