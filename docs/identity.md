---
layout: default
title: 기본 키를 매핑
parent: JPA
nav_order: 1
---

# 기본 키를 매핑하는 두 가지 방법
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

이 부분은 [영속성 컨텍스트](https://nswon.github.io//docs/association/)를 보고 오면 더 이해가 가기 쉽다. 
영속성 컨텍스트는 엔티티를 식별자 값으로 구분하므로 엔티티를 영속 상태로 만들려면 식별자 값이
반드시 있어야 한다. 식별자를 할당하는 다양한 전략이 있다. 그 중 많이 쓰는 2가지 전략을 알아보겠다.

## 기본키 직접 할당 전략

기본 키를 직접 할당하려면 @Id로 매핑시키면 된다.

```java
@Id
@Column(name="id")
private String id;
```

직접 할당하는 방법은 em.persist() 로 엔티티를 저장하기 전에 에플리케이션에서 기본 키를 직접 할당하는 방법이다.

```java
Member member = new Member();
member.setAge(18); //직접 할당
em.persist(board);
```

## IDENTITY 전략

IDENTITY는 기본 키 생성을 데이터베이스에 위임하는 전략이다. 한마디로 기본 키 생성을 DB에서 대신 해준다는 소리다. 주로 MySQL, PostgreSQL 등 에서 사용한다. 그 중에서 많이 쓰이는 MySQL의 AUTO\_INCREMENT 기능은 데이터베이스가 기본 키를 자동으로 생성 해준다.

```java
CREATE TABLE MEMBER (
	ID Integer AUTO_INCREMENT PRIMARY KEY,
    NAME varchar(255)
);
```

데이터베이스에서 INSERT를 할 때 ID를 비워두면 자동으로 값이 채워진다.

엔티티에선 아래 코드처럼 @Id와 @GeneratedValue 어노테이션을 같이 사용해야 한다.

```java
@Entity
public class Member {

	@Id @GeneratedValue(strateay = GenerationType.IDENTITY)
    private Long id;
}
```