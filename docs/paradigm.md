---
layout: default
title: 패러다임의 불일치
parent: JPA
nav_order: 1
---

# 패러다임의 불일치
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 패러다임의 불일치란 ?

객체와 관계형 데이터베이스가 지향하는 목적이 서로 다름으로 인해 둘의 기능과 표현 방법이 다른 것을 말한다.
기능과 표현 방법이 다르다는게 무슨 말일까 ?

## 객체모델과 관계형 데이터베이스 모델은 지향하는 패러다임이 서로 다르다.

애플리케이션은 발전하면서 그 내부의 복잡성도 점점 커진다. 이 복잡성을 제어할 수 있는 장치가 바로 객체지향 프로그래밍이다.
도메인 모델을 정의할 때도 객체지향적으로 개발하면 객체지향 언어가 가진 장점들을 활용할 수 있다.
문제는 객체지향적으로 정의한 도메인 모델을 저장할 때 발생한다. 단순히 객체를 저장하는 일이라면 문제가 발생하지 않겠지만,
실무에선 단순히 객체만을 저장하는 일은 거의 없을 것이다. 

예를 들어,  사용자가 회원가입을 하면 회원이라는 객체 안에 저장된 후 이 객체를 관계형 데이터베이스에 보관해야 한다.
이 때, 회원 객체를 저장할 때 회원 객체가 팀 객체를 참조하고 있다고 가정해보자.
단순히 회원 객체를 저장하게 되면 참조하는 팀 객체를 잃어버리는 현상이 발생한다.
패러다임의 불일치를 해결하지 못하는 것은 아니다. 개발자가 중간에서 해결할 수 있지만, 문제는 객체와 관계형 데이터베이스 사이의 패러다임 불일치 문제가 많이 발생한다는 것이다. 그로 인해 문제를 해결하기 위해 너무 많은 시간과 코드를 소비한다.
그럼 패러다임의 불일치로 인해 발생하는 문제를 살펴보자.

## 상속

패러다임의 불일치로 인해 발생하는 대표적인 2가지로 상속과 연관관계가 있는데, 먼저 상속할 때의 문제점을 알아보자.
객체는 상속이라는 개념이 있지만 테이블은 상속이라는 개념이 없다.

다음과 같은 객체가 있다.

```java
abstract class Item {
	Long id;
	String name;
	int price;
}



class Album extends Item {
	String artist;
}



class Movie extends Item {
	String director;
	String actor;
}
```

Album 객체를 저장하려면 이 객체를 분리해서 두 SQL을 만들어야 한다.

```sql
INSERT INTO ITEM …
INSERT INTO ALBUM …
```

Movie 객체도 마찬가지다.

```sql
INSERT INTO ITEM …
INSERT INTO MOVIE …
```

조회하는 것도 쉬운일이 아니다. Album을 조회해야 한다면 Item과 Album 테이블을 조인해서 조회한 후 그 결과를 Album 객체로 생성해야 한다. 이런 과정이 패러다임의 불일치를 해결하려고 소모하는 시간과 코드이다. 

## 2-1-1. JPA와 상속

JPA는 개발자가 패러다임의 불일치를 해결하려고 소모하는 시간과 비용 문제를 없애준다. 대신 해결해준다는 소리다.
개발자는 그냥 자바 컬렉션에 객체를 저장하듯이 JPA에게 객체를 저장하면 된다.
아까 예로 들었던 Album 객체를 저장하는 것을 JPA를 이용해보자.

```java
jpa.persist(album);
```

개발자는 이렇게 작성만 해주면 된다. 그럼 JPA가 자동으로 다음과 같은 SQL을 생성해서 저장한다.

```sql
INSERT INTO ITEM ...
INSERT INTO ALBUM ...
```

아까 쉬운일이 아니라는 조회도 쉽게 할 수 있다. find() 메서드를 써 간단하게 객체를 조회하면 된다.

```java
String albumId = "id100";
Album album = jpa.find(Album.class, albumId);
```

그럼 JPA가 자동으로 두 테이블을 조인한 후 필요한 데이터를 조회해서 그 결과를 반환해준다.

```sql
SELECT I.*, A.*
FROM ITEM I
JOIN ALBUM A ON I.ITEM_ID = A.ITEM_ID
```

## 연관관계

아마 상속보다 더 대표적인게 연관관계가 아닐까 싶다. 
객체는 참조를 사용해서 다른 객체와 연관관계를 가지고 참조에 접근해서 연관된 객체를 조회한다. 

다른 객체와 연관관계를 가진고, 참조에 접근.. 연관된 객체를 조회한다.. 코드를 보기 전까진 이해가 어려울 수 있으니 코드로 설명하겠다.
Member 객체는 Team 객체를 참조한다. 

```java
class Member {
	Team team;
    ...
    Team getTeam() {
    	return team;
    }
}

class Team {
	...
}
```

Member 객체는 Member.team 필드에 Team 객체의 참조를 보관함으로써 Team 객체와 관계를 맺는다.
따라서 참조 필드에 접근하면 Member와 연관된 Team을 조회할 수 있게 된다.

```java
member.getTeam();
```

## 객체지향 모델링

아까 객체는 연관된 객체의 참조를 보관해야 연관된 객체를 찾을 수 있다고 했다.
그럼 객체의 참조를 보관해보자.

```java
class Member {
	String id;
    Team team; //연관된 객체의 PK가 아닌 참조로 연관관계를 맺음
    String username;
    
    Team getTeam() {
    	return team;
    }
}

class Team {
	Long id;
    String name;
}
```

그럼 Member 객체와 연관된 Team 을 조회할 수 있다.

```java
Team team = member.getTeam();
```

하지만 이렇게 객체지향 모델링을 사용하면 객체를 테이블에 저장하거나 조회하기가 쉽지 않다.
Member 객체는 team 필드로 연관관계를 맺지만, MEMBER 테이블은 TEAM\_ID 외래 키로 연관관계를 맺기 때문이다.

즉 객체 모델은 외래 키가 필요없고 참조만 있으면 된다. 반면에 테이블은 참조가 필요 없고 외래 키만 있으면 된다.
결국 개발자가 중간에서 참조필드를 외래 키로 변환시키는 역할을 해야 한다.

## 저장

객체를 데이터베이스에 저장하려면 필드를 외래 키 값으로 변환해야 한다고 했다.
다음처럼 외래 키 값을 찾아서 INSERT SQL을 만들 수 있다.

```java
member.getId(); //MEMBER_ID PK에 저장
member.getTeam().getId(); //TEAM_ID FK에 저장
member.getUsername();
```

## JPA와 연관관계

위 작업을 JPA를 이용하면 패러다임의 불일치를 해결할 수 있다.

```java
member.setTeam(team);
jpa.persist(member);
```