---
layout: default
title: JPA가 탄생한 배경
parent: JPA
nav_order: 1
---

# JPA가 탄생한 배경
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 옛날에 에플리케이션을 개발하는 법

애플리케이션을 개발하면 관계형 데이터베이스를 데이터 저장소로 사용하면서 오랫동안 SQL을 다룬다.

몇 년 전에는 개발자들이 JDBC API를 직접 사용해서 SQL문을 작성해가며 개발을 해왔다. 그러다 보니 애플리케이션의 비즈니스 로직보다 SQL문과 JDBC API를 작성하는 데 더 많은 시간을 보내게 된 것이다. 

시간이 지나고 아이바티스(마이바티스), Jdbc Template 등을 SQL 매퍼라 불리는데, SQL 매퍼를 통해 JDBC API 사용 코드를 줄일 수 있었다. 하지만 등록, 조회, 수정, 삭제(CRUD)용 SQL은 여전히 반복해서 작성해야 했다. 

## 반복, 반복, 반복!

자바로 개발하는 애플리케이션은 관계형 데이터베이스를 데이터 저장소로 사용한다.
데이터베이스에 데이터를 저장하려면 SQL을 사용해야 한다. 밑에 그림을 보자. 

![자바 관계형 데이터베이스](/images/jpabackground-1.png)

자바에서 개발한 애플리케이션을 JDBC API를 사용하여 SQL문을 작성해 데이터베이스에 전달하는 방식이다. 

이 때 SQL을 직접 다뤄야 하는데, 여기서 문제점이 발생한다.
예시를 들어보자. 회원을 등록, 조회, 수정, 삭제하는 기능을 개발해보자.
아래는 자바에서 사용할 회원 객체이다.

```java
public class Member {
	
    private String memberId;
    private String name;
    ...
}
```

다음으로 회원 객체를 데이터베이스에 관리할 목적으로 만든 객체(DAO)를 만들자.

```java
public class MemberDAO {
	
    public Member find(String memberId) {...}
}
```

이제 MemberDAO의 find() 메소드를 완성해서 회원을 조회하는 기능을 개발해보자.

보통 아래와 같은 순서로 개발을 진행할 것이다.


- 회원 조회용 SQL을 작성한다.
- JDBC API를 사용해서 SQL을 실행한다.
- 조회 결과를 Member 객체로 매핑한다.


그럼 회원 조회 기능과 저장 기능을 만들어보자.

```java
public class MemberDAO {
	
    public Member find(String memberId) {...}
    public void save(Member member) {...}
}
```

회원 등록용 SQL을 작성한다.

```java
String sql = "INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES(?, ?)";
```

회원 객체의 값을 꺼내서 등록 SQL에 전달한다.

```java
pstmt.setString(1, member.getMemberId());
pstmt.setString(2, member.getName());
```

JDBC API를 사용해서 SQL을 실행한다.

```java
pstmt.excuteUpdate(sql);
```

이렇게 조회하고 저장하는 기능을 만들었다.  수정하고 삭제하는 기능은 위 과정을 반복하면 된다. 문제는 한번 CRUD를 하면 너무 많은 SQL과 JDBC API를 코드로 작성해야한다는 점이다. 만약 테이블마다 CRUD는 있을 수 밖에 없을 건데, 실무에선 테이블이 적어도 100개 있다면 위 과정을 100번 반복해야 한다는 점이다. 

## SQL에 의존적인 개발

앞에서 만든 회원 객체를 관리하는 MemberDAO를 완성하고 CRUD 기능을 완성하였다.
그런데 갑자기 회원의 연락처도 함께 저장해달라는 요구사항이 추가 되었다.
그럼 회원의 연락처를 추가해보자.

```java
public class Member {
	
    private String memberId;
    private String name;
    private String tel; //추가
    ...
}
```

연락처를 저장할 수 있도록 INSERT SQL을 수정했다.

```java
String sql = "INSERT INTO MEMBER(MEMBER_ID, NAME, TEL) VALUES(?, ?, ?)";
```

그 다음 회원 객체의 연락처 값을 꺼내서 등록 SQL에 전달했다.

```java
pstmt.setString(3, member.getTel());
```

이제 조회를 해보니 모든 연락처가 null로 출력되었다. 조회 SQL에 연락처를 추가하지 않았기 때문이다.
그럼 조회 SQL을 수정해야 한다. 조회 SQL 변경은 생략하도록 하겠다.
코드에 집중하지 말고 흐름을 보자.

CRUD기능을 추가하기 위해 직접 SQL문을 작성하고 JDBC API를 통해 데이터베이스에 넣고 뺀다.
다른 요구사항이 들어오면 CRUD코드와 SQL문을 변경해야 한다.
이 말은 회원 객체에 필드 하나를 추가할 뿐인데도 DAO의 CRUD 코드와 SQL 대부분을 변경해야 한다.
엔티티와 강한 의존관계를 가지고 있기 때문이다.

## 정리

에플리케이션에서 SQL을 직접 다룰 때 발생하는 문제점 요약

-   진정한 의미의 계층 분할이 어렵다.
-   엔티티를 신뢰할 수 없다.
-   SQL에 의존적인 개발을 피하기 어렵다.

## 옛날 객체의 사용법

옛날에는 객체를 단순히 테이블에 맞추어 데이터를 전달하는 용도로만 사용되었다. 즉, 객체 모델링보다 테이블 설계에 초점을 맞춘 것이다. 
객체지향적인 장점을 포기한 것이다. 그럼 객체지향적으로 개발하면 되지 않을 까 ? 라는 생각이 있었지만,
객체 모델링을 하게 되면 세밀하게 되어있는 객체를 데이터베이스에 저장하거나 조회하기는 어려웠고, 관계형 데이터베이스와의 차이점도 메워야 했다. 결국은 더 많은 SQL코드를 작성해야만 했다. 

그래서 나온것이 ORM 프레임워크 표준 기술인 JPA가 등장했다.

## JPA의 장점

위에서 봤던 직접 SQL문과 JDBC API를 사용한 문제점들이 발생했다. JPA는 어떻게 해결할까 ?
JPA가 제공하는 CRUD API를 간단히 알아보자.

저장기능

```java
jpa.persist(member);
```

조회기능

```java
String memberId = "helloId";
Member member = jpa.find(Member.class, memberId);
```

수정 기능

```java
Member member = jpa.find(Member.class, memberId);
member.setName("이름변경")
```

JPA는 SQL을 개발자 대신 작성해서 실행해주는 것 이상의 기능들을 제공한다.
즉, 개발자들이 반복적인 SQL을 짜는 귀찮음을 JPA에 맡기고 더 좋은 객체 모델링과 더 많은 테스트를 작성하는데 시간을 보낼 수 있다.