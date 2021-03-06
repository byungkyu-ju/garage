
**Q1.SOLID 원칙을 설명하시오**

- SRP 단일책임의원칙
  - 컴포넌트,클래스, 메서드는 변경할 이유가 단 하나뿐이어야 한다는 원칙.
  ex) 규모있는 시스템은 복잡하다. 복잡성을 다루기 위해서는 정리가 필수이며, 변경을 할때는 영향이 미치는 컴포넌트만 이해해도 충분하다.

- OCP 개방폐쇄의 원칙
  - 컴포넌트,클래스,모듈 함수는 확장에는 열려있고, 변경에는 닫혀있어야 한다.
  ex) interface

- LSP 리스코프 치환의 원칙
  - 한 타입이 다른 타입의 서브ㅏ입이 되기 위해서는 슈퍼타입에 순응해야한다는 것이다.
  ex) Employee는 Person에 구조적으로 순응하며 따라서 Person을 대체할 수 있다. > 객체지향의 사실과 오해

- ISP 인터페이스 분리의 원칙
  - 클래스는 자신이 사용하지 않는 인터페이스는 구현하지 않아야 한다. 인터페이스 역할의 단일책임을 가져야함.

- DIP 의존성역전의 원칙
  - 상위모듈이 하위모듈에 의존하는 관계를 역전시켜, 상위모듈을 독립시킬 수 있다.
  ex) 상위모듈에서는 하위모듈에서 제공하는 인터페이스를 통해 기능을 사용하도록 함으로써 결합도를 낮추고, 동시에 SRP를 지킴
  
**Q2.gc에 대해 설명하시오**

- java에서는 코드로 메모리를 명시적으로 지우지 않기 때문에 gc가 필요없는 객체를 찾아 지우는 작업을 한다.

- GC 프로세스
  - heap = young generation(MinorGC) / old generation(MajorGC)
  - young generation(eden / survival 0 / survival 1) -> old generation
  - 메모리의 heap 영역은 young generation과 old generation으로 나뉘는데, 최초의 Object는 young generation의 Eden영역에 존재하며 Eden영역이 가득차면 MinorGC가 발생한다.
MinorGC가 발생할때 유효한 참조가 없는 Object는 메모리에서 해제되며, 유효한 참조가 남아있는 Object는 다음단계인 Survival 0으로 이동한다. 그리고 기존에 Survival 0에 있던 유효한 Object들은 Survival 1로 이동하며, 역시 유효하지 않은 Object들은 메모리에서 해제된다.
마지막으로 Survival1에 있던 유요한 오브젝트들은 Survival1이 가득 차게 되면 young generation에서 old generation으로 이동하게 된다.

**Q3.자바 클래스로더를 설명하시오**

- User defined Class Loader -> System Class Loader -> Extension Class Loader -> Bootstrap Class Loader
  - System Class Loader : $CLASSPATH에 정의도니 경로를 탐색하여 그곳에 있는 클래스들을 로딩함.

**Q4.제너릭에 대해 설명하시오**

- 클래스에서 사용할 타입을 외부에서 설정하는것. 컴파일 과정에서 타입체크를 할 수 있다.
- 컴파일시 타입을 체크하기 때문에 사전에 에러를 잡을 수 있으며, 타입만 다르기 때문에 코드의 재사용성을 증가시킬 수 있다.

**5.SpringFramework에 대해 설명하시오**

- 자바 엔터프라이즈 개발을 쉽게 도와주는 프레임워크
- 특징
  - IoC : 객체의 생성을 컨테이너가 관리함
  - DI : 객체의 의존성을 외부에서 주입시킴
  - AOP : 공통된 기능을 재사용하기 위한 모듈화를 가능하게 함


**6.Spring Bean Scope에 대해 설명하시오**

- 스프링은 스스로 빈의 생명주기를 관리해주고, 어플리케이션에서 빈이 필요한 시점에 스프링이 빈을 주입해준다. 이때 기본적으로 모든 빈을 싱글톤으로 생성하여 관리한다.
- 종류
  - singleton
    - 어플리케이션 구동시 최초 인스턴스를 생성하고, Spring IoC Container내 객체가 하나만 존재하게 한다.
  - prototype
    - bean 생성시 여러개의 객체가 존재하게 한다.
  - request
    - HTTP Request에서 하나의 객체만 존재하게 한다.
  - session
    - session내 하나의 객체만 존재하게 한다.
  - global session
    - global session내 하나의 객체만 존재하게 한다.
  - application
    - ServletContext내 하나의 객체만 존재하게 한다.

**Q7.Mybatis와 ORM 각각의 장단점**

- Mybatis는 SQL로 DB를 다루는 Persistance Framework이다. JDBC에서 처리하는 코드와 파라미터 및 매핑을 대신하여 쉽게 사용할 수 있도록 도와준다.
  - 장점
    - 다양한 SQL문법을 통해 데이터베이스 설계 기반의 데이터를 조회할 수 있다
  - 단점
    - 별도의 SQL언어를 사용해야한다.

- ORM은 객체와 데이터베이스를 연결해주는 기능이다.
  - 장점
    - SQL을 사용하지 않고 객체를 통해서 데이터를 가져올 수 있으며, 객체지향적인 코드로 좀 더 직관적으로 어플리케이션을 구현할 수 있도록 한다.
    - 대부분 DB에 종속적이지 않다.
  - 단점
    - 러닝커브가 높다.

**8.Thread와 Process의 차이**

- Process는 OS로 부터 자원을 할당받는 작업의 단위이고, Thread는 한 Process내에서 동작되는 여러 실행의 흐름.
- Process내의 주소공간이나 자원들을 같은 Process내에 Thread리 공유하면서 실행

**Q9.REST가 무엇인지 설명하시오**

- REpresentational State Transter
- HTTP만으로 쉽게 통신할 수 있는 방법을 제공(GET,POST,PUT,PUSH)
- 6가지 만족조건
  - Client-Server를 분리하여 별도로 진화할 수 있어야 한다.
  - Stateless (클라이언트에서 서버로 전달하는 요청에 전달하려는 모든것이 담겨 있어야 한다.별도 저장된 정보를 사용하면 안된다.)
  - Cache의 사용가능여부를 명시해야한다.
  - 계층화된 아키텍쳐를 사용함으로써, 각 계층을 마음대로 넘나들 수 없도록 해야한다.
  - Uniform Interface (지정된 규약)
    - Identification of resources (URL에 리소스를 명시해야한다.)
    - Manipulation of resources through representations  
    (representations- GET / POST / PUT /PUSH 를 통해서 리소스가 조작되어야한다.)
    - self-descrive messages (메시지 스스로 설명이 가능해야한다.)
    - HATEOAS(Hypermedia As The Engine Of Application State)  
      링크를 통해 어플리케이션의 상태변화가 가능해야한다.


**10.SOA와 MSA의 정의를 설명하시오**

- SOA(Service Oriented Architecture)
  - 어플리케이션을 비즈니스 업무단위로 나누고, 기능단위로 묶어서 이를 각각의 서비스라는 이름으로 조합하는 아키텍쳐
- MSA(Microservice Architecture)
  - 어플리케이션을 여러개의 독립된 서비스로 나누고, 이를 조합하여 기능을 사용하는 아키텍쳐
  - 장점
    - 서비스를 잘게 나누어 전체 시스템의 결합도를 낮춘다.
    - 서비스를 각각 독립적으로 배포할 수 있다.
    - 재사용성과 확장성이 높다.
    - 서로 다른 플랫폼과 언어를 사용할 수 있어 목적에 맞는 시스템을 각각 구현할 수 있다.
  - 단점
    - 서비스들이 잘게 나뉘기 때문에 시스템의 복잡성이 증가한다.
    - 큰 규모의 프로젝트에서 서비스호출을 추적하거나 모니터링하기 어렵다.
    - 로그관리가 어려우며, 별도의 로그관리 시스템이 필요하다.(elk)