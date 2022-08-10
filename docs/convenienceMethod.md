---
layout: default
title: 연관관계 편의 메서드
parent: JPA
nav_order: 1
---

# 연관관계 편의 메서드
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 양방향 연관관계

엔티티들은 대부분 다른 엔티티와 연관관계가 있다. 회원은 팀에 소속될 수 있기 때문에 회원 엔티티와 팀 엔티티에 관계를 맺어야 한다. 
이 부분은 [JPA가 생겨난 이유](https://skatpdnjs.tistory.com/31?category=1013012)와 [패러다임의 불일치](https://skatpdnjs.tistory.com/42?category=1013012) 포스팅에서 자세히 나와있다. 참고하자.

간단하게 다시 말하자면 객체는 참조를 사용해서 관계를 맺고 테이블은 외래 키를 사용해서 관계를 맺는다. 
객체와 테이블이 관계를 맺을 때 완전히 다르게 맺는 것이다. 
그래서 우리는 객체의 참조와 테이블의 외래 키를 매핑하는 것이 이 장의 목표이다. 
회원과 팀을 양방향 관계로 매핑해보자. 

```java
//매핑한 회원 엔티티
@Entity
public class Member {
	
    @Id
    @Column(name = "member_id")
    private String id;
    
    private String username;
    
    @ManyToOne
    @JoinColumn(name = "team_id")
    private Team team;
    
    //연관관계 설정
    public void setTeam(Team team) {
    	this.team = team;
    }
    //...
}
```

```java
//매핑한 팀 엔티티
@Entity
public class Team {
	
    @Id
    @Column(name = "team_id")
    private String id;
    
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
    
    //...
}
```

@ManyToOne과 @OneToMany로 2개의 단방향 연관관계를 맺어주고, mappedBy를 통해 연관관계의 주인을 지정해줌으로써
양방향이라는 것을 표현했다.

## 양방향 연관관계의 주의점

mappedBy의 의미를 정확히 알아야 한다. mappedBy는 연관관계의 주인을 정해주는 것이고, 그 말은 연관관계의 주인인 쪽에 관계를 설정해야지 DB에 반영이 된다. 
예를 들어보자. 아래는 회원과 팀을 생성해서 팀에 회원을 저장하는 코드이다. 

```java
public void 연관관계_설정() {
	
    //회원1 저장
    Member member1 = new Member("member1", "회원1");
    em.persist(member1);
    
    //회원2 저장
    Member member2 = new Member("member2", "회원2");
    em.persist(member2);
    
    Team team1 = new Team("team1", "팀1");
    team.getMembers().add(member1);
    team.getMembers().add(member2);
    
    em.persist(team1);
}
```

위 코드를 실행하고 회원을 조회하면 team\_id가 null이 뜰것이다. 왜냐하면 연관관계의 주인은 Member 엔티티인데, Member한테 관계를 설정하지 않고 자기 자신한테만 관계를 설정하고 있다. 그러면 DB에 반영이 되지 않는다. 

아래는 양쪽 모두 관계를 설정하는 코드이다. 

```java
public void 연관관계_설정() {
	
    //회원1 저장
    Member member1 = new Member("member1", "회원1");
    em.persist(member1);
    
    //회원2 저장
    Member member2 = new Member("member2", "회원2");
    em.persist(member2);
    
    Team team1 = new Team("team1", "팀1");
    
    //연관관계 설정
    member1.setTeam(team1);
    team.getMembers().add(member1);
    member2.setTeam(team1);
    team.getMembers().add(member2);
    
    em.persist(team1);
}
```

이렇게 해주어야지 조회할 때 제대로 된 team\_id 값이 나올 것이다. 

## 연관관계 메서드

연관관계를 설정하는 메서드를 연관관계 메서드라고 한다. 양방향 관계에서는 아래의 두 코드를 하나의 것처럼 사용하는 것이 안전하다. 
따로따로 사용하다가 실수로 한 줄을 넣지 않게 되면 둘 중 하나만 호출이 되어 양방향이 깨질 수 있기 때문이다. 

```java
member.setTeam(team);
team.getmembers().add(member);
```

그래서 위 두 코드를 하나로 묶은 코드를 연관관계 편의 메서드라고 한다. 위 코드와 아래코드는 같은 의미이다. 

```java
public void setTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}
```

## 실전 예시

회원과 게시판 관계를 맺어보자.

게시판에서 매핑

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "member_id")
private Member writer;
```

회원에서 매핑

```java
@OneToMany(mappedBy = "writer")
private List<Board> boardList = new ArrayList<>();
```

그리고 연관관계 편의 메서드를 작성해보자. 

게시판
```java
public void confirmWriter(Member writer) {
    this.writer = writer;
    writer.addBoard(this)
}
```

회원
```java
public void addBoard(Board board) {
    boardList.add(board)
}
```


아까도 말했지만 게시판에서 this.writer = writer을 통해서 회원의 값을 게시판에 저장하고,
저장했으면 게시판도 회원에 저장해야 한다. 
아래는 게시판을 저장하는 코드다. 

```java
@Transactional
@Override
public Long create(BoardCreateRequestDto requestDto) {
    Board board = requestDto.toEntity();
    board.confirmWriter(memberRepository.findByEmail(SecurityUtil.getLoginUserEmail()).orElseThrow(() -> new MemberException(MemberExceptionType.NOT_FOUND_MEMBER)));
    boardRepository.save(board);

    return board.getId();
    }
```
아래는 회원을 찾는 로직이다. 
```java
memberRepository.findByEmail(SecurityUtil.getLoginUserEmail())
        .orElseThrow(() -> new MemberException(MemberExceptionType.NOT_FOUND_MEMBER))
```

이렇게 되면 찾은 회원을 게시판에 넣고, 회원쪽에서도 this를 통해 자기자신도 회원의 게시판 리스트에 저장하는 것이다.