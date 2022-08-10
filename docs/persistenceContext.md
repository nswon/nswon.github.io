---
layout: default
title: 영속성 컨텍스트 개념과 원리
parent: JPA
nav_order: 1
---

# 영속성 컨텍스트 개념과 원리
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 영속성 컨텍스트 란 ?

눈에 보이지 않는 논리적인 개념

EntityManager을 사용하여 DB에 바로 접근하는 것이 아닌, `영속성 컨텍스트에 접근`하여 저장, 삭제 등 가능

## 엔티티의 생명주기

`비영속`: 객체가 영속성 컨텍스트와 관련이 없는 상태

```java
public void data() {
    Member member = new Member();
}
```

`영속`: 객체가 영속성 컨텍스트에 의해 관리되고 있는 상태. 아래와 같이 EntityManager을 통해 저장시킨 상태

```java
@PersistenceContext //Entity를 영구저장하는 환경을 만듬
private EntityManager em; //EntityManager 선언

public void data() {
    Member member = new Member();
    em.persist(member); //저장
}
```

`준영속`: 영속 컨텍스트에 저장되어 있다가 분리된 상태

```java
@PersistenceContext 
private EntityManager em;

public void data() {
    Member member = new Member();
    em.detach(member); //분리
}
```

`삭제`: 영속성 컨텍스트에서 객체를 완전히 분리시킨 상태

```java
@PersistenceContext
private EntityManager em;

public void data() {
    Member member = new Member();
    em.remove(member); //삭제
}
```

## 영속성 컨텍스트를 왜 사용할까 ?

그렇다면 왜 DB에 바로 저장하지 않고 중간 단계인 영속성 컨텍스트를 통해 저장을 할까 ?

이점이 존재하기 때문이다. 대표적인 이점으로는 `1차캐시`, `쓰기지원` 등이 있다.

## 1차캐시

```java
public void data() {
    Member member = new Member();
    member.setAge(18);
    em.persist(member);
    Member findAgeMember = em.find(Member.class, 18);
}
```

위 코드를 살펴보면 DB접근이 없다는 것을 확인 할 수 있다. 만약 DB에 저장하고 찾는 과정이었다면, persist()를 통해 insert 하고, find()를 통해 select를 할 것이다. 하지만 영속성 컨텍스트에 저장이 되어있어서 `DB까지 접근할 필요가 없어진다`. 쉽게 말해서 성능이 엄청나게 향상이 된다.

## 트랜잭션을 지원하는 쓰기 지연

![트랜잭션 쓰기 지원 사진](/images/persistenceContext-1.png)

persist를 한다고 해서 DB에 INSERT 하는 것이 아닌, commit을 해야 발생한다. 1차캐시와 별도로 쓰기 지연 SQL 저장소라는 곳에서 INSERT SQL을 저장한다. 버퍼링 기능을 사용해 한꺼번에 쿼리를 동작시킬 수 있다.

코드로 작성한 모습이다.

```java
// 트랜잭션 획득
EntityTransaction tx = em.getTransaction();

em.persist(member1);
em.persist(member2);
//INSERT SQL은 데이터베이스에 가지 않고 SQL저장소에 쌓인다.

transaction.commit(); // 트랜잭션 커밋

// 트랜잭션을 커밋하는 순간 데이터베이스에 한꺼번에 INSERT SQL을 보낸다.
```

## 플러시

그렇다면 SQL들을 데이터베이스에 보내는 방법은 commit() 밖에 없을까 ? flush() 를 사용하면 강제로 쓰기지연SQL저장소에 있는 것들을 DB로 보낸다. 동작하는 법은 3가지가 있다.

entityManager.flush() 호출

```java
em.persist(member1);
em.persist(member2);

em.flush(member);
```

트랜잭션 커밋

```java
em.persist(member1);
em.persist(member2);

transaction.commit(member);
```

JPQL 실행

```java
em.persist(member1);
em.persist(member2);

List<Member> members = em.createQuery("select m from Member m", Member.class)
                .getResultList();
```