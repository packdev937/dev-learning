# JPA란?

### JPA는 Java Persistance API이다

RAM은 휘발성 데이터를 저장할 수 있다. (컴퓨터가 꺼지면 다 사라져 버린다)

데이터가 날라가지 않도록 데이터를 하드디스크에 영구적으로 저장한다.

Persistance라는 것은 어떤 데이터가 영구히 저장되도록 한다.

DBMS는 하드디스크의 일부분을 이용하여 데이터를 저장하는 것이다.

✭ Java Persistance API는 자바의 데이터를 영구히 저장할 수 있는 환경을 제공하는 API이다.

**API**

- 애플리케이션 프로그래밍 인터페이스 (API)
- 프로그램을 만들기 위해 제공되는 인터페이스의 개념이란?

  프로토콜은 약속을 의미하고 인터페이스 또한 약속을 의미하지만 그 방식이 다르다.

  인터페이스는 상하관계가 존재하는 약속이다.

  **example)**

  A 데이터 사용을 원하면 밤 12시 부터 아침 6시까지만 이용해!

  프로토콜은 서로 간의 권리가 동등한 약속이다.

  **example)**

  수 많은 프로토콜로 만들어진 것이 www


### JPA는 ORM 기술이다

ORM은 Object Relational Mapping 이라고 하는데, 객체를 데이터베이스에 연결하는 방법론 같은 것이다.

ORM → 노예

**example)**

건물을 짓는 설계도가 있으면 설계도는 2D로 구성되어 있다.

이를 토대로 건물을 지으면 3D 건물이 생긴다. 이를 모델링 이라고 한다.

✭ 즉, 모델링이라는 것은 추상적인 개념을 현실 세계에 뽑아내는 걸 얘기한다.

데이터 베이스에 Team 이라는 테이블이 있다고 가정하자.

| ID | INT |
| --- | --- |
| Name | verchar |
| Year | verchar |

이를 자바에서 Input하기도 하고 Output하기도 한다.

Input 하는 행위를 DML (Deleter, Update, Insert) 라고 하고, Output 하는 행위를 Select라고 한다.

이를 집어 넣고 가져올 때 자바가 가지고 있는 데이터 타입과 데이터베이스가 가지고 있는 데이터 타입이 다르다.

따라서 클래스를 통해 데이터 베이스에 있는 테이블을 모델링 해야 한다.

```java
Class Team {
	int id;
	String name;
	String year;
}
```

우리는 이를 Database 에 있는 데이터를 Java에 모델링한다고 한다.

보통은 순서는 Database → Java 이다. 하지만 ✭ 이를 역으로 만들 수 있다.

일단은 Class를 먼저 만들고 데이터 베이스를 자동으로 생성할 수 있다. 이 때 JPA 인터페이스가 필요하다. 이를 ORM 이라고 한다.

### JPA는 반복적인 CRUD 작업을 생략하게 해준다.

Select, Select All, Delete, Update, Insert 등은 자주 일어나는 반복적인 동작들이다.

1. 데이터 베이스가 있고 자바가 있으면 자바가 데이터베이스에게 커넥션을 요청한다 (연결 해도 돼?)
2. 데이터베이스가 자바를 검증하고 세션을 열어준다. 그러면 자바는 커넥션을 갖는다.
3. 커넥션 뒤에 자바는 쿼리를 전송할 수 있다.
4. 데이터베이스는 쿼리를 통해 작업을 수행하고 어떤 데이터를 만들어내고 이 데이터를 다시 자바에 응답하게 된다.
5. 자바는 이 응답 데이터를 받아서 자바의 Object에 맞게 변환해야 한다.

✭ 하지만 JPA는 이 모든 것을 단순하게 처리하게 도와준다.

### JPA는 영속성 컨텍스트를 가지고 있다.

**영속성**이란 어떤 데이터를 **영구적**으로 **저장**하게 해주는 것이다.

**컨텍스트란 무엇인가?**

컨텍스트는 대상에 대한 모든 정보이다.

**example)**

*“난 너의 모든 컨텍스트를 가지고 있어”* → *“난 너의 모든 것들을 알고 있어”*

컨텍스트란 모든 것에 달라붙을 수 있다. 어떤 대상에 달라 붙는 순간 모든 정보를 가지고 있는 것이 컨텍스트.

이 컨텍스트는 넘겨줄 수 있다.

**example)**

데이터베이스에 어떤 동물 데이터를 저장하려고 한다 가정하자.

다이렉트하게 DB에 접근하는 것이 아닌 항상 중간에 위치한 영속성 컨텍스트에 접근한다.

✭ 영속성 컨텍스트는 자바와 데이터 베이스의 중간 다리 역할을 한다. 자바의 요청이 들어오면 해당 요청을 받아드려 데이터베이스에 전송하기도 하고, 만약 자바에서 어떤 값을 수정해서 영속성 컨텍스트와 데이터 베이스의 값이 다르다면 수정된 것으로 파악해 자동으로 update()를 진행한다.

(영속성 컨텍스트의 모든 일들은 자동으로 처리된다)

영속성 컨텍스트와 데이터베이스의 데이터를 맞추는 것을 동기화라고 한다.

### JPA는 DB와 OOP의 불일치성을 해결하기 위한 방법론을 제공한다

Database에 Team이라는 테이블이 있고 Player이라는 테이블이 있다고 가정하자.

Team의 Column 은 ID, Team 이름, 창립 연도로 구성되어 있고, Player는 ID, 선수 이름, teamID로 구성되어 있다. 이 때 teamID는 참조 키로 FK라고 불린다.

우리가 Player를 자바로 가져온다고 가정해보자.

테이블을 참고하면 다음과 같은 코드가 만들어진다

```java
class Player {
	int id;
	String name;
	int teamID;
}
```

하지만 우리가 숫자로 된 teamID를 보고 team을 유추할 수 있을까?

(✭DB는 객체 저장이 불가능하기 때문에 int로 된 teamID로 team table을 참조한다) 자바는 이를 위해 Team 객체로 필드를 설정하면 된다.

```java
class Player {
	int id;
	String name;
	~~int teamID;~~
	Team team;
}
```

하지만 team 객체를 어떻게 불러올까? ORM을 사용하면 참조 키를 사용하여 자동으로 teamID에 맞는 팀 객체를 불러와준다. 따라서 이를 통해 DB와 OOP의 불일치성을 해결해준다.

### JPA는 OOP의 관점에서 모델링을 할 수 있게 해준다 (상속, 콤포지션, 연관관계)

```java
class Car extends TimeStamp{
	int id;
	int price;
	String name;
	Engine engine;
}

class Engine extends TimeStamp{
	int id;
	int speed;
}

class TimeStamp {
	LocalTime updateTime;
	LocalTime createTime;
}
```

### 방언 처리가 용이하여 Migration 하기 좋고, 유지보수에도 좋다

스프링 → JPA → DB

이 때, JPA는 mySQL 뿐만 아니라 다양한 SQL 언어를 지원한다. (Oracle, MariaDB, Postgre)

JPA는 추상화 객체로 구성되어 있기 때문에 언어를 연결하기만 하면 된다.

### JPA는 쉽지만 어렵다

쿼리를 한번만 날려도 될거 같은데 여러 번 날리는 경우도 있고, 데이터 양이 많아지면 처리할 쿼리도 많아지기 때문에 이를 효율적으로 관리하는 것이 중요하다. 