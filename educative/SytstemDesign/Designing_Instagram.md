# Designing Instagram

## 1.What is Instagram

인스타그램은 사용자간에 사진이나 동영상을 올리고 공유할 수 있는 SNS이다.  
인스타그램의 사용자들은 올리는 내용을 공개/비공개로 지정할 수 있는데, 공개로 지정하면   사용자들 모두가 볼 수 있으며  
비공개로 지정하면 지정된 사용자들만 볼 수 있다.
또한 facebook,twitter,flicker, tumblr와 같은 다른 SNS에도 공유할 수 있다.  
'뉴스피드'는 사용자가 팔로우하는 사람들의 최근 사진들을 보여주며,  
이 글에서는 인스타그램의 간단한 버전을 분석할 계획이다.

## 2. Requirements and Goals of the System

이 글은 아래의 관점을 기준으로 인스타그램을 설계를 분석한다.

### Functional Requirements

- 사용자는 사진을 업로드/다운로드/조회를 할 수 있다.
- 사용자는 사진/동영상의 이름을 기준으로 검색한다.
- 사용자는 다른 사용자를 팔로우할 수 있다.
- 시스템은 사용자가 팔로우한 팔로워를 기준으로 상단에 노출된 사진을
   뉴스피드에 보여준다.

### Non-function Requirements

- 시스템은 고가용성을 지녀야한다.
- 뉴스피드는 0.2초안에 생성되어야한다.
- 사용자가 사진을 보지 않는 동안에는 일관성이 유지되지 않아도 무방하다.
- 시스템은 신뢰성이 높아야 하며, 업로드된 사진이나 동영상은 유실되면 안된다.

#### Not in scope

- 사진에 태그추가, 태그로 사진검색, 사진의 댓글, 사진에 태깅한 유저, 팔로우한 사용자는 고려하지 않는다.

## 3. Some Design Considerations

시스템은 많은 데이터를 읽기 때문에, 빠르게 사진들을 보여줄 수 있는 시스템을 구성하는것에 집중해야한다.

- 사용자들은 많은 사진을 올릴 수 있기 때문에, 
시스템을 설계하는 동안 저장공간을 효율적으로 관리하는 것을 고려하여 설계해야한다.
- 사진을 보는동안 대기시간이 오래 걸려서는 안된다.
- 데이터는 100% 신뢰할 수 있어야 하며, 시스템은 사용자가 올린 사진이 절대 지워지지 않는다는것을 보장해야한다.

## 4. Capacity Estimation and Constraints

- 500M의 사용자가 있으며, 1M의 사용자가 매일 이용한다고 가정한다.
- 2M의 새로운 사진들이 매일 올라오며, 매초 23개의 새로운 사진들이 올라온다.
- 평균 사진의 크기는 200KB이다.
- 하루 동안 사진의 저장에 필요한 용량은 400GB이다 (2M * 200KB)
- 10년 동안 필요한 용량은 약 1425TB이다. (400GB*365*10)

## 5. High Level System Design

- 시스템설계에 사진을 올리는 시나리오와 사진을 조회/검색하는 시나리오가 필요하다.
- 사진을 저장하고, 사진저장에 필요한 메타데이터를 저장할 Object Storage가 필요하다.

![[High Level System Design]](<./images/Designing_Instagram//5-1.png>)

## 6. Database Schema

DB에는 사용자데이터, 업로드한 사진, 팔로우 내역들을 저장해야한다.  
Photo 테이블은 사진과 관련된 데이터를 저장하며, 관련된 사진을 빠르게 보여주기 위해 PhotoID와 CreationDate에 Index를 생성해야한다.


![[Database Schema]](<./images/Designing_Instagram/6-1.png>)

정석적으로는 위 스키마를 저장하기 위해서 조인을 하기 위해 MySQL과 같은 RDBMS를 써야 한다.  
하지만 RDB는 규모를 늘리거나 줄일때 어려움을 겪는다. 자세한 내용은 SQL vs. NoSQL을 참고하면 된다.  
그리고 사진은 Hadoop이나 S3에 분산하여 저장할 수 있다.  
또한 스키마를 분산된 key-value의 장점을 활용할 수 있는 NoSQL에 저장할 수도 있다.  
사진과 관련된 메타데이터는 테이블에 저장되는데 PhotoID를 key로 사용하고, value에는 PhotoLocation, UserLocation, CreationTimestamp를 포함한다.

저장할 때 누구와 관련된 사진인지, 그리고 팔로우한 사용자들 간의 관계를 저장해야하는데 
이 테이블들간의 관계는 Cassandra와 같은 wide-column datastore을 통해 저장할 수 있다.

UserPhoto테이블의 key는 UserID, value에는 PhotoID들을 저장하며, 각각 다른 열에 저장한다.

Cassandra같은 일반적인 key-value형태의 저장은 신뢰성을 제공하기 위해 항상 replicas를 만든다.
또한 데이터저장에 있어서 즉시 데이터를 삭제하지 않으며, 시스템에서 영구적으로 지우지 않는 한 삭제하지 않는다.

## 7. Data Size Estimation

데이터가 테이블에 어떻게 저장되고, 10년 동안 얼마만큼의 용량이 필요한지 예측해본다.  

**User** :int와 dateTime은 4바이트로 가정하면, 한 row는 68bytes를 차지한다.  

```code1
UserID (4 bytes) + Name (20 bytes) + Email (32 bytes) + DateOfBirth (4 bytes) + CreationDate (4 bytes) + LastLogin (4 bytes) = 68 bytes
```

500M의 사용자를 가정으로 한다면 32GB가 필요하다.  

```code2
500 million * 68 = 32GB
```

Photo사진은 한개의 row당 284bytes가 필요하다.  

```code4
PhotoID (4 bytes) + UserID (4 bytes) + PhotoPath (256 bytes) + PhotoLatitude (4 bytes) + PhotLongitude(4 bytes) + UserLatitude (4 bytes) + UserLongitude (4 bytes) + CreationDate   (4 bytes) = 284 bytes
```

매일 2M의 사진이 업로드된다면, 매일 0.5GB의 용량이 필요하다.  

```code5
2M * 284 bytes = 0.5GB per day  
```

10년동안 1.88TB 가 필요하다.

**UserFollow** : UserFollow는 각 row당 8bytes가 필요하다. 500M의 사용자에 평균 500명의 사용자를 follow한다고 가정하면 1.82TB가 필요하다.  

```code6
500 million users * 500 followers * 8 bytes = 1.82TB
```

10년간 전체 필요한 용량은 3.7TB 이다.  

```code7
32GB + 1.88TB + 1.82TB = 3.7TB
```

## 8. Component Design

사진 업로드는 느리지만, 캐시를 사용한다면 빠르게 조회할 수 있다.  
사진 업로드는 느리기 때문에 업로드하는 사용자들은 많은 connection을 사용한다.  
업로드하는 요청이 많다면, 조회기능이 제대로 되지 않을 수 있다는 의미이다.  
시스템을 디자인하기 이전에 웹서버의 커넥션이 제한적이라는것을 명심해야한다.
만약 웹서버가 500개의 Connection만을 가질 수 있다면, 이를 초과한 Connection은 연결할 수 없다.  
이런 병목현상을 해결하기 위해 업로드를 위한 서버와 조회를 위한 서버를 분리하여 시스템을 안전하게 해야 한다.  
사용자의 요청을 분리한다면 각 서버를 개별적으로 확장하고 최적화 할 수 있다.

![[Component Design]](<./images/Designing_Instagram/8-1.png>)

## 9. Reliability and Redundancy

서비스에서 반드시 파일이 유실되어서는 안된다. 그러므로 서버가 죽더라도 다른 서버에서 검색할 수 있도록 복사된 다른 서버를 운영해야한다.  
이런 규칙은 다른 시스템의 구성에도 적용된다.  
시스템의 고가용성을 필요로 할 때, 복제된 여러 개의 서비스를 운영해야 한다. 그렇게 해야 일부 서비스가 죽더라도 시스템을 계속 운영할 수 있기 때문이다.  
이중화는 시스템의 [단일장애점(single point of failure)][1]을 제거한다.

만약 하나의 인스턴스로 서비스를 운영해야한다면, 서비스를 복사해두어 서비스를 운영하지 않더라도 기존 시스템에 장애가 생겼을때 복구할 수 있다.

이중화를 한다면 단일장애점(single point of failure)을 제거하고, 문제가 생겼을 경우 백업 또는 대체하는 기능을 사용할 수 있다.
예를 들어 프로덕션 환경에서 동일한 서비스의 인스턴스 2개가 운영되고 있을때, 하나의 인스턴스가 장애가 발생하더라도 시스템은 다른 인스턴스로 정상적인 서비스운영이 가능하다. 장애조치는 자동으로 될 수도 있고, 수동으로 해야될 수도 있다.

![[Reliability and Redundancy]](<./images/Designing_Instagram/9-1.png>)

## 10. Data Sharding (메타데이터 샤딩)

### a.Partitioning based on UserID

UserID 를 기준으로 동일한 샤드에 모든 사진을 넣는다고 가정한다.
하나의 샤드가 1TB라면, 3.7TB의 데이터를 저장하기 위해서는 4개의 샤드가 필요하다.
더 나은 성능과 확장성을 위해 10개의 샤드를 유지한다고 가정할때, UserID % 10 의 샤드번호를 찾고 데이터를 저장할 수 있다.

**PhotoID는 어떻게 만드는가?**

 각 DB 샤드는 PhotoID에 자동으로 증가하는 시퀀스를 지정할 수 있으며, PhotoID에 ShardID를 추가하여 시스템에서 고유하도록 만든다.

**스키마를 분리할때 발생할 다른 이슈?**

- Hot User를 어떻게 처리하는가? 여러사람들은 hot User를 follow하고, 많은 사람들은 그들이 업로드한 사진들을 본다.
- 몇몇 사용자들은 다른 사용자에 비해 많은 사진을 가지게 되기 때문에, 시스템이 고르게 분배되지 않는다.
- 몇몇 사용자의 사진을 하나의 샤드에 저장하지 못하면 어떻게 되는가? 사진들을 많은 샤드에 분산시킨다면 속도가 오래걸리는가?
- 사용자의 모든 사진을 하나의 샤드에 저장하게 되면 샤드가 다운되거나 응답시간이 길어질 때 해당 데이터를 사용하지 못하게 될 수도 있다.

### b.Partitioning based on PhotoID

고유한 PhotoID를 생성하고, PhotoID % 10으로 shard 번호를 찾으면 위 문제를 해결할 수있다. PhotoID는 시스템에서 고유하기 때문에 PhotoID에 ShardID를 추가할 필요가 없다.

**PhotoID는 어떻게 만드는가?**

PhotoID에 지정할 자동으로 생성되는 시퀀스값을 샤드에서 찾을 수 없다. 왜냐하면 먼저 저장할 샤드를 찾기 위해 PhotoID를 알아야 하기 때문이다. 한가지 방법은 별도의 DB 인스턴스를 사용해서 자동으로 ID를 생성하는것이다.  
만약 64비트가 적절하다면 , 64비트만 포함하는 필드를 정의할 수 있다. 그래서 시스템에 사진을 추가하고 싶을때, 이 테이블에 새로운 row를 추가하고, 생성한 ID를 새로운 사진의 PhotoID로 사용할 수 있다.  

**DB에서 Key를 생성하는것이 단일장애점(single point of failure)이 되지는 않는가?**

이 문제를 해결하기 위해 2개의 DB를 정의하고, 다른 ID를 생성하면 된다.(홀수/짝수)  
MySQL의 경우 아래의 스크립트로 시퀀스를 정의한다.  

```code8
KeyGeneratingServer1:
auto-increment-increment = 2
auto-increment-offset = 1

KeyGeneratingServer2:
auto-increment-increment = 2
auto-increment-offset = 2
```

라운드로빈과 다운타임의 조절을 위해 2개의 DB 사이에 로드밸런서를 둘 수 있다. 2개의 서버는 생성되는 키값이 동일하지 않을 수 있지만, 시스템에는 문제가 되지 않는다. 사용자를 위한 ID 테이블, 사진의 댓글 테이블, 그리고 다른 object를 정의하여 이 구조를 확장할 수 있다.
또는 Designing a URL Shortening service like TinyURL 에서 설명하는 키생성 스키마와 유사한 기능을 구현할 수 있다.

**How can we plan for the future growth of our system?**

미래의 데이터 증가를 감당하기 위해 다수의 논리적 파티션을 둘 수 있다. 초기에는 한개의 물리적 DB에 여러개의 논리적 파티셔닝을 할 수 있다. 각 DB 서버는 여러개의 DB 인스턴스를 올릴 수 있으므로, 각각의 논리 파티션에 별도의 DB를 나눌 수 있다.
따라서 DB에 많은 데이터가 있다면, 논리파티션을 다른 서버로 이관할 수 있다.
논리적 파티션을 DB에 맵핑할 별도의 config file을 관리할 수 있다. 이는 파티션을 쉽게 이동할 수 있도록 한다. 언제든지 파티션을 이동시키고 싶을때는 변경된 사항을 알리기 위해 이 config파일만 update하면 된다.

## 11. Ranking and News Feed Generation

사용자에게 News Feed를 만들어 제공하려면 사용자들이 업로드한 많은 사진이 필요하다.  
간단하게 사용자의 뉴스피드에 100개의 최신 사진을 보여줘야 한다고 가정하자.  
어플리케이션 서버는 먼저 팔로우한 유저들의 목록을 가져온 다음 100개의 사진에 대한 메타데이터를 가지고 온다.
마지막으로 서버는 이 사진들을 100장의 사진들을 정렬하기 위해 랭킹알고리즘에 넣는다. (최신의,유사한 사진을 기준)  그리고 이를 사용자에게 보여준다.  
위의 문제점은 여러테이블에 쿼리작업을 하고, 결과를 정렬/병합/순위를 매기기 위해 많은 시간이 소요될 수 있다는 것이다.  
그렇다면 효율성을 향상시키기 위해 뉴스피드를 사전에 생성하여 저장하고 별도의 테이블에 저장해야한다.

### Pre-generating the News Feed

지속적으로 사용자의 뉴스피드를 생성하고, UserNewsFeed 테이블에 저장하는 서버를 둘 수 있다. 그렇게 해서 사용자가 뉴스피드에서 최신의 사진을 보려할때,
이 테이블을 간단하게 쿼리로 조회하고 결과를 사용자에게 보여줄 수 있다.
서버가 사용자의 News Feed를 생성하려고 할 때마다, 먼저 서버는 UserNewsFeed 테이블에 쿼리를 실행하여 마지막에 생성된 News Feed의 시간을 찾는다.
그리고 News Feed 데이터는 그 이후에 생성된다. 

**What are the different approaches for sending News Feed contents to the users?**

**1. Pull** : 클라이언트는  뉴스피드를 서버로부터 정기적으로 또는 필요할때 수동으로 조회할 수 있다. 이때 발생할 문제점은  
a) 클라이언트가 요청하기 전까지 데이터를 가지고 오지 않을 수 있다.  
b) 새로운 데이터가 없으면 빈 응답이 나다난다.  

**2. Push** : 서버는 가능한만큼 사용자에게 새로운 데이터를 보여줄 수 있다. 이를 효율적으로 관리하기 위해 사용자는 서버로부터 업데이트를 받기 위해 를 Long Poll request를 유지해야한다.
이때 발생 가능한 문제점은, 많은 사람들이 팔로우 하는 사용자이거나 수백만명이 따르는 유명한 사람이다. 이런경우 서버는 업데이트를 빈번히 요청해야한다.

**3. Hybrid** :팔로워가 많은 유저를 pull-based model로 이동시키고 그들이 팔로우하는 사용자에게 푸시할 수 있다. 또다른 방법은 서버가 특정 빈도 이상으로 많은 사용자에게 추가로 정기적으로 데이터를 가져게 하는 방법이다.

뉴스피드 생성의 자세한 내용은 Designing Facebook’s Newsfeed 를 참고하면 된다.

## 12.News Feed Creation with Sharded Data

사용자에게 뉴스피드를 제공하는데 가장 중요한 요구사항은 모든 사용자들의 최신 사진을 가져오는 것이다.  
이를 위해서는 사진생성시간에 대한 정렬 매커니즘이 필요하다.  
효율성을 위해 사진생성시간을 PhotoID와 합칠 수 있다. PhotoID에는 기본 인덱스가 있기 때문에 빠르게 조회할 수도 있다.
PhotoID가 시간/PhotoID로 구성되어있다고 가정할때, 첫번째는 epoch time을 나타내고, 2번째는 자동시퀀스 의 값을 나타낸다. 새로운 PhotoID를 만들기 위해 현재의 epoch time을 가져와서 DB에서 생성되는 자동 시퀀스ID를 추가하여 생성한다.이 PhotoID에서 shard number를 찾아낼 수 있고, 사진을 해당 샤드에 저장할 수 있다.

**What could be the size of out PhotoID?**

Epoch time이 오늘부터 시작된다고 가정할때, 50년동안 저장시 필요한 비트수는 얼마나 되는가?  

```code8
86400 sec/day * 365 (days a year) * 50 (years) => 1.6 billion seconds
```

이 숫자를 저장하려면 31비트가 필요하다. 평균적으로 초당 약 23개의 사진이 저장되며,  
자동 시퀀스 저장시 9비트가 저장되는것을 허용할 수 있다.
그래서 매초 (2^9 => 512)  512개의 사진을 저장할 수 있으며, 자동으로 증가하는 시퀀스를 매초 재설정 할 수 있다.

Data Sharding에 대한 자세한 내용은 Designing Facebook’s Newsfeed 을 참고하면 된다.

## 13.Cache and Load balancing

전세계의 분산된 사용자에게 제공할 사진 전송 시스템이 필요하다.  
해당 서비스는 지역적으로 분산된 사진을 캐시서버를 사용하여 사용자에게 더 밀접하게 전달해야한다.  
최신의 DB row를 캐싱하기위해 메타데이터 서버용 캐시를 도입할 수 있는데,  
MemCache를 사용하여 데이터와 어플레케이션에 캐시가 적용되어 있으면 DB를 빠르게 조회할 수 있다.  
LRU(Least Recently Used)알고리즘은 해당 시스템에 적합한 캐시 추출 정책이다. 이 정책으로 가장 최근에 조회한 행을 삭제한다.

<b>How can we build more intelligent cache?</b>

만약 80-20의 법칙을 사용한다면 매일의 20% 트래픽이 80%의 트래픽을 생성한다는 것을 의미하므로  
사진들이 유명한 사람들에게 읽힌다는것을 확신할 수 있다.  
따라서 매일 읽는 사진과 메타데이터 중 20%를 캐싱하면 된다.

------

[1]: https://ko.wikipedia.org/wiki/%EB%8B%A8%EC%9D%BC_%EC%9E%A5%EC%95%A0%EC%A0%90  

> <https://www.educative.io> Grokking the System Design Interview - Designing Instagram
