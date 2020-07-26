# 1.Querydsl 설정

## 1.1. gradle 설정

```code
plugins {
    ...
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10" //추가
    id 'java'
}

dependencies {
    ...
    implementation 'com.querydsl:querydsl-jpa' //querydsl 추가
    implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.8' //query logging
}

//querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
 jpa = true
 querydslSourcesDir = querydslDir
}
sourceSets {
 main.java.srcDir querydslDir
}
configurations {
 querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
 options.annotationProcessorPath = configurations.querydsl
}
//querydsl 추가 끝
```

## 1.2. generate querydsl class

- gradle > other > compileQuerydsl 실행
  - ex) Member.java -> QMember.java 사용

## 1.3. JPQL / querydsl 구문 비교

```code
@Test
    public void startJPQL() {
        //find member1
        Member findMember = entityManager.createQuery("select m from Member m where m.username = :username", Member.class)
                .setParameter("username", "member1")
                .getSingleResult();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }

    @Test
    public void startQuerydsl() {
        JPAQueryFactory queryFactory = new JPAQueryFactory(entityManager);
        QMember m = new QMember("m");

        Member findMember = queryFactory
                .select(m)
                .from(m)
                .where(m.username.eq("member1"))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
```

- querydsl 사용시 preparementStatement에 자동으로 파라미터 바인딩됨
- 컴파일시 오류 체크 가능
- entityManager가 멀티스레딩에 문제없도록 설계되어있으므로, entityManager를 사용하는 querydsl도 멀티스레딩에 문제가 없다.

## 1.4. Q클래스

- 1.3.의 쿼리를 아래와 같이 static import로 개선할 수 있음

```code
import static study.querydsl.entity.QMember.member;

Member findMember = queryFactory
                .select(member)
                .from(member)
                .where(member.username.eq("member1"))
                .fetchOne();
```
