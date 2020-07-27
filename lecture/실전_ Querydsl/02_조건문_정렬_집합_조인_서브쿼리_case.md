# 2.Querydsl 조건문_정렬_집합_조인_서브쿼리_case

## 2.1. 조건문

 ```code

member.username.eq("member1") // username = 'member1'
member.username.ne("member1") //username != 'member1'
member.username.eq("member1").not() // username != 'member1'
member.username.isNotNull() //이름이 is not null
member.age.in(10, 20) // age in (10,20)
member.age.notIn(10, 20) // age not in (10, 20)
member.age.between(10,30) //between 10, 30
member.age.goe(30) // age >= 30
member.age.gt(30) // age > 30
member.age.loe(30) // age <= 30
member.age.lt(30) // age < 30
member.username.like("member%") //like 검색
member.username.contains("member") // like ‘%member%’ 검색
member.username.startsWith("member") //like ‘member%’ 검색
 ```

## 2.2. 결과 조회

```code
fetch() : 리스트 조회, 데이터 없으면 빈 리스트 반환
fetchOne() : 단 건 조회
  결과가 없으면 : null
  결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException
fetchFirst() : limit(1).fetchOne()
fetchResults() : 페이징 정보 포함, total count 쿼리 추가 실행
fetchCount() : count 쿼리로 변경해서 count 수 조회
```

## 2.3. 정렬

```code
desc() , asc() : 일반 정렬
nullsLast() , nullsFirst() : null 데이터 순서 부여
```

## 2.4. 페이징

- 페이징은 성능문제가 발생할 수 있으므로, 경우에 따라 분리

```code
offset
limit
```

## 2.5. 집합

- 여러 종류가 있을 때 Tuple로 조회할 수 있으나, 주로 DTO를 통해 조회한다.

```code
count()
sum()
avg()
max()
min()
groupby()
having(item.price.gt(1000))
```

## 2.6. 기본 조인

- 첫 번째 파라미터에 조인 대상을 지정하고, 두 번째 파라미터에 별칭(alias)으로 사용할 Q 타입을 지정.

```code
join() , innerJoin() : 내부 조인(inner join)
leftJoin() : left 외부 조인(left outer join)
rightJoin() : rigth 외부 조인(rigth outer join)
```

## 2.7. 세타조인

- JPA 연관관계가 없는 필드로 조인

```code
 em.persist(new Member("teamA"));
 em.persist(new Member("teamB"));

 List<Member> result = queryFactory
    .select(member)
    .from(member, team)
    .where(member.username.eq(team.name))
    .fetch();

 assertThat(result).extracting("username")
                    .containsExactly("teamA", "teamB");
}
```

- 구버전에서는 외부 조인이 불가능했으나, JPA2.1부터 ON절 조인을 지원함
- 하지만, 내부조인이면 where 절로 해결하고, 외부조인이 필요한 경우만 on절을 사용

```code
 List<Tuple> result = queryFactory
    .select(member, team)
    .from(member)
    .leftJoin(member.team, team).on(team.name.eq("teamA"))
    .fetch();
```

- 연관관계가 없는 엔티티 외부조인
- hibernate 5.1부터 추가됨

```code
em.persist(new Member("teamA"));
em.persist(new Member("teamB"));

List<Tuple> result = queryFactory
    .select(member, team)
    .from(member)
    .leftJoin(team).on(member.username.eq(team.name))
    .fetch();

주의! 문법을 잘 봐야 한다. leftJoin() 부분에 일반 조인과 다르게 엔티티 하나만 들어간다.
일반조인: leftJoin(member.team, team)
on조인: from(member).leftJoin(team).on(xxx)
```

## 2.8. 페치조인

- SQL 조인을 활용해서 연관된 엔티티를 SQL한번에 조회. 최적화에 주로 사용
- @PersistenceUnit EntityManagerFactory.isLoaded
  - 이미 로딩된 엔티티인지 확인할 수 있음
- join(), leftJoin() 등 조인 기능 뒤에 fetchJoin() 이라고 추가하면 된다.

```code
// 미적용 - 지연로딩으로 쿼리 각각 실행

@PersistenceUnit
EntityManagerFactory emf;

em.flush();
em.clear();

Member findMember = queryFactory
    .selectFrom(member)
    .where(member.username.eq("member1"))
    .fetchOne();

boolean loaded =
emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());

assertThat(loaded).as("페치 조인 미적용").isFalse();


// 적용
em.flush();
em.clear();

Member findMember = queryFactory
    .selectFrom(member)
    .join(member.team, team).fetchJoin()
    .where(member.username.eq("member1"))
    .fetchOne();

boolean loaded =
emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
assertThat(loaded).as("페치 조인 적용").isTrue();

```

## 2.9. 서브쿼리

- JPAExpressions 사용

```code

QMember memberSub = new QMember("memberSub"); //member명칭 중복을 피하기 위해 생성

List<Member> result = queryFactory
    .selectFrom(member)
    .where(member.age.eq(
        JPAExpressions
        .select(memberSub.age.max())
        .from(memberSub)
        ))
        .fetch();

assertThat(result).extracting("age")
.containsExactly(40);

```

- JPA JPQL 서브쿼리의 한계점으로 from 절의 서브쿼리(인라인 뷰)는 지원하지 않는다. 당연히 Querydsl
도 지원하지 않는다. 하이버네이트 구현체를 사용하면 select 절의 서브쿼리는 지원한다. Querydsl도 하
이버네이트 구현체를 사용하면 select 절의 서브쿼리를 지원한다.

- from 절의 서브쿼리 해결방안
  - 서브쿼리를 join으로 변경한다. (가능한 상황도 있고, 불가능한 상황도 있다.)
  - 애플리케이션에서 쿼리를 2번 분리해서 실행한다.
  - nativeSQL을 사용한다.

## 2.10. Case문

- 복잡한 case문은 CaseBuilder()를 사용

```code
//단순
List<String> result = queryFactory
    .select(member.age
        .when(10).then("열살")
        .when(20).then("스무살")
    .otherwise("기타"))
    .from(member)
    .fetch();

//복잡
List<String> result = queryFactory
    .select(new CaseBuilder()
            .when(member.age.between(0, 20)).then("0~20살")
            .when(member.age.between(21, 30)).then("21~30살")
        .otherwise("기타"))
    .from(member)
    .fetch();

```

## 2.11. 상수, 문자더하기

- 상수 사용시 Expressions.constant(xx)

```code
List<Tuple> result = queryFactory
    .select(member.username, Expressions.constant("A"))
    .from(member)
    .fetch();

```

- 참고: 문자가 아닌 다른 타입들은 stringValue() 로 문자로 변환할 수 있다.  
이 방법은 ENUM을 처리할 때도 자주 사용한다.

```code
String result = queryFactory
    .select(member.username.concat("_").concat(member.age.stringValue()))
    .from(member)
    .where(member.username.eq("member1"))
    .fetchOne();
```
