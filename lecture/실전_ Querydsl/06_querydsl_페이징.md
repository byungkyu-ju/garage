# 6.querydsl 페이징

## 6.1. 사용법

- fetchResults로 조회해서 자동으로 조회한 갯수를 호라용

```code
QueryResults<MemberTeamDto> results = queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetchResults();
        List<MemberTeamDto> content = results.getResults();
        long total = results.getTotal();

        return new PageImpl<>(content, pageable, total);
--------------------------------

MemberSearchCondition condition = new MemberSearchCondition();
PageRequest pageRequest = PageRequest.of(0, 3);

Page<MemberTeamDto> result = memberRepository.searchPageSimple(condition, pageRequest);

assertThat(result.getSize()).isEqualTo(3);
assertThat(result.getContent()).extracting("username").containsExactly("member1","member2", "member3");

```

- 별도로 전체 개수 쿼리를 조회하여 분리.
- 성능최적화시 활용

```code
List<MemberTeamDto> content = queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        long total = queryFactory
                .select(member)
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                .fetchCount();

        return new PageImpl<>(content, pageable, total);
```

## 6.2. CountQuery 최적화

- count 쿼리가 생략 가능한 경우 생략해서 처리
  - 페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때
  - 마지막 페이지 일 때(offset + 컨텐츠 사이즈를 더해서 전체 사이즈 확인)

```code
JPAQuery<Member> countQuery = queryFactory
                .select(member)
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                ;

//return PageableExecutionUtils.getPage(content, pageable, () -> countQuery.fetchCount());
return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchCount);
// content와 pageable을 내부에서 비교해서 생략 가능한 경우 ountQuery.fetchCount()를 조회하지 않음
```
