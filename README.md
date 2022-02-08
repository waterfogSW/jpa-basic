# 자바 ORM표준 JPA프로그래밍

데이터 베이스 방언
- JPA는 특정 데이터 베이스에 종속되지 않는다.
- 각 데이터베이스가 제공하는 SQL문법과 함수는 조금씩 다르다.
- 방언(dialect) : SQL표준을 지키지 않는 특정 데이터 베이스만의 고유 기능
  - ex) org.hibernate.dialect.OracleDialect, org.,hibernate.dialectH2Database
