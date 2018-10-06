---
layout: post
title: "[DB] Connection pool"
date: 2018-10-06
tags: [DB]
comments: true
---

`Connection Pool`이란 데이터베이스와 애플리케이션을 효율적으로 연결하는 라이브러리로 웹 애플리케이션에서 필수적이다. <br>
웹 애플리케이션 서버로 상용 제품을 사용하면 보통 제조사에서 제공하는 커넥션 풀 구현체를 사용한다. 그 외에는 오픈소스 라이브러리로 `Apache Commons DBCP`와 `Tomcat-JDBC`, `BoneCP`, `HikariCP` 등이 있다. 주로 많이 쓰이는 것은 `HikariCP`라고 한다. <br>
<br>
커넥션 풀 라이브러리를 잘 사용하면 데이터베이스와 애플리케이션의 일부분에서 발생하는 문제가 전체로 전파되지 않게 할 수 있고, 일시적인 문제가 긴 시간 이어지지 않게 할 수 있다. 반대로 값을 적절히 설정하지 않아 커넥션 풀이 애플리케이션에서 병목 지점이 되는 경우도 있다. <br>
웹 애플리케이션의 요청은 대부분 DBMS로 연결되기 때문에 커넥션 풀 라이브러리 설정은 전체 애플리케이션의 성능과 안정성에 영향을 미치는 핵심이다. <br>

이렇게 중요한 커넥션 풀 라이브러리를 적절하게 사용하려면 라이브러리의 내부 구조와 원리, 속성값의 의미를 이해해야 한다. <br>
<br>

## 버전 선택과 속성 설정 방법

#### JDK 버전과 Commons DBCP 버전

JDK의 버전에 따라서 Connection이나 Statement 같은 JDBC(Java Database Connectivity)의 인터페이스가 조금씩 다르므로 사용하는 JDK의 버전에 맞게 Commons DBCP 버전을 선택해야 안정된 동작을 기대할 수 있다.

Commons DBCP 주요 버전과 그에 대응하는 JDK 버전과 JDBC 버전은 다음과 같다.

| Commons DBCP 버전 | JDK 버전 | JDBC 버전 |
|---|---|---|
| Commons DBCP 2 | JDK 7 | JDBC 4.1 |
| Commons DBCP 1.4 | JDK 6 | JDBC 4 |
| Commons DBCP 1.3 | JDK 1.4~1.5 | JDBC 3 |

`Connection.getMetaData()` 메서드를 호출한 다음 커넥션을 닫지 않았을 때 메모리 누수가 발생하는 문제 등 주요 버그가 `Commons DBCP 1.4.1`에서 패치되어 `Commons DBCP 2`에 반영되었다.

#### Commons DBCP 속성 설정

Commons DBCP의 속성은 `BasicDataSource` 클래스의 Setter 메서드로 설정할 수 있다. Spring 프레임워크를 사용한다면 다음과 같이 bean 설정으로 속성을 등록한다.

Commons DBCP 1.x BasicDataSource 클래스 설정 사례
```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"  
    destroy-method="close"
    p:driverClassName="${db.driverClassName }"
    p:url="${db.url}"
    p:username="${db.username}"
    p:password="${db.password}"
    p:maxActive="${db.maxActive}"
    p:maxIdle="${db.maxIdle}"
    p:maxWait="${db.maxWait}"
/>
```

Commons DBCP 2.x의 BasicDataSource 클래스 설정 사례

```xml
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource"  
    destroy-method="close"
    p:driverClassName="${db.driverClassName }"
    p:url="${db.url}"
    p:username="${db.username}"
    p:password="${db.password}"
    p:maxTotal="${db.maxTotal}"
    p:maxIdle="${db.maxIdle}"
    p:maxWaitMillis="${db.maxWaitMills}""
/>
```

참고 : https://d2.naver.com/helloworld/5102792
