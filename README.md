# e2edemo

#### 문제 이해(20분) : 전체 문제와 답안 Template 확인하면서, PPT에 키워드 기입 가능
#### 답안 작성(2시간) 후 등록 : 정확히 알고 있는 답안부터 작성, 본인 블로그 참조하고, 인터넷 검색은 NO
#### 추가 보안(30~40분) 후 2차 등록 : 인터넷 검색은 이때 필요시 사용

### flow chart: https://hello-woody.tistory.com/14
### msa 개요: https://velog.io/@tedigom/MSA-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-2-MSA-Outer-Architecure
### microservices: https://microservices.io/

### 클라우드 환경, 아키텍트 패턴, MSA 
#### 애플리케이션아키텍처패턴: 애플리케이션에대해어떤아키텍처를선택해야하는가?				
* 모놀리식아키텍처: 애플리케이션을배포가능한단일단위로설계				
* 마이크로서비스아키텍처: 느슨하게결합된서비스모음으로애플리케이션을설계				
#### 분할(Decomposition) : 애플리케이션을서비스로분할하는방법은무엇인가?				
* 비즈니스역량별분할(Decompose by business capability) : 비즈니스역량별서비스정의				
* 서브도메인별분할(Decompose by subdomain) : 도메인주도설계에의한결과물인서브도메인에따른서비스정의				
* 독자적서비스(Self-contained Servcie) : 다른서비스로부터의응답을받을때까지기다리지않고동기요청을처리하도록서비스를설계				
* 팀별서비스(Service per team)				
#### 마이크로서비스로의 리팩토링				
* 교살자무화과나무애플리케이션(Strangler Application)				
* 부패방지계층(Anti-Corruption Layer)				

# 업무용 응용프로그램 개발 방안
## A고객사 시스템의 채널 확대 및 사용자 증가에 따라 발생하는 인증 및 세션 관리 문제 해결을 위한 인증 방식 개선 방안  
#### Azure Token https://learn.microsoft.com/ko-kr/entra/identity-platform/access-tokens
### 인증 문제1 - 교차절단관련(Cross-cutting concerns) : 교차절단문제를처리하는방법은무엇인가?
* 마이크로서비스섀시(MicroserviceChassis) : 교차문제를처리하고서비스개발을단순화하는프레임워크
* 외부화된구성정보(Externalized configuration) : 데이터베이스위치및자격증명과같은모든구성을외부화
* 서비스템플릿(Service Template) : 표준교차문제를구현하고신속한신규서비스개발을위해개발자가복사하기위한템플릿을이용
### 인증 문제2 - 보안: 요청을처리하는서비스에요청자의ID를전달하는방법은무엇인가?
* 접근토큰(Access Token) : 서비스간에교환되는사용자정보를안전하게저장하는토큰
  + Ref. https://tansfil.tistory.com/58?category=475681
    1. 계정정보를 요청 헤더에 넣는 방식
    2. Session / Cookie 방식
    3. 토큰 기반 인증 방식 (ft. JWT) -> stateless, session 저장소 x

## A고객사 비즈니스 유연성과 성능 관점에서 상품을 관리하기 위한 데이터 모델을 새롭게 설계하고 설계 사유 제시  
### 데이터 모델 문제 - 데이터관리: 데이터일관성을유지하고쿼리를구현하는방법은무엇인가?
* 데이터베이스아키텍처
  + 서비스별데이터베이스(Database per Service) : 각서비스별로자체고유데이터베이스구성
  + 공유데이터베이스(Shared Database) : 서비스가데이터베이스를공유
* 쿼리
  + API 구성(API Composition) : 데이터를소유한서비스를호출하고메모리내조인을수행하여쿼리를구현
  + 명령과쿼리책임분리(CQRS) : 효율적으로쿼리할수있는하나이상의구체화된뷰를유지관리하여쿼리를구현
* 데이터일관성유지
  + 집합(Aggregate) : 도메인주도설계의집합을바탕으로일관성유지
  + 사가(Saga) : 서비스전반에걸쳐데이터일관성을유지하기위해로컬트랜잭션의이력인사가를사용
    - 보상 가능 트랜잭션(compensatable transaction): 보상 트랜잭션으로 롤백 가능한 트랜잭션
    - 피벗 트랜잭션(pivot transaction): 사가의 진행/중단 지점. 피봇 트랜잭션이 커밋되면 사가는 완료될 때까지 실행. 피봇 트랜잭션은 보상 가능 트랜잭션, 재시도 가능한 트랜잭션 그 어느 쪽도 아니지만, 최종 보상 가능 트랜잭션 또는 최초 재시도 가능 트랜잭션이 될 수 있음
    - 재시도 가능 트랜잭션(retriable transaction): 피봇 트랜잭션 직후의 트랜잭션. 반드시 성공
  + 도메인이벤트(Domain event) : 데이터가변경될때마다이벤트발행
  + 이벤트소싱(Event sourcing) : 이벤트의이력으로집합(Aggregate)들의영속성을관리

## 예약 처리 프로세스와 데이터 모델에서 발생하고 있는 동시성 이슈의 해결 방안 제시  
### 동시성 문제 - 커뮤니케이션패턴
* 커뮤니케이션스타일: 서비스가서로및외부클라이언트와통신하는데사용하는통신메커니즘은무엇인가?
  + 원격프로시저호출(Remote Procedure Invocation) : 서비스간통신에원격프로시저호출기반프로토콜사용
  + 메시징(Messaging) : 서비스간통신에비동기식메시징사용
  + 도메인별프로토콜(Domain-Specific) : 도메인별프로토콜사용
  + 멱등성소비자(Idempotent Cunsumer) : 메시지소비자가동일한메시지로여러번호출해도문제가없도록처리

* 서비스 탐색: RPI 기반서비스의클라이언트는서비스인스턴스의네트워크위치를어떻게탐색할수있는가?
  + 서비스레지스트리(Service-registry) : 서비스인스턴스위치데이터베이스
  + 클라이언트측탐색(Client-side discovery) : 클라이언트가서비스레지스트리를쿼리하여서비스인스턴스의위치를탐색
  + 서버측탐색(Server-side discovery) : 라우터는서비스레지스트리를쿼리하여서비스인스턴스의위치를검색합니다.
  + 자체등록(Self registration) : 서비스인스턴스가서비스레지스트리에자신을등록
  + 제3자등록(3rd party registration) : 제3자가서비스레지스트리에서비스인스턴스를등록

* 트랜잭션메시징: 데이터베이스트랜잭션의일부로메시지를발행하는방법은무엇인가?
  + 트랜잭션보낸편지함(Transactional outbox)
  + 트랜잭션로그테일링(Transationlog tailing)
  + 퍼블리셔를폴링(Polling publisher)

* 외부API : 외부클라이언트는서비스와어떻게통신할것인가?
  + API 게이트웨이(API gateway) : 각클라이언트에서비스에대한통합인터페이스를제공하는서비스
  + 프론트엔드용백엔드(Backendsfor frontends) : 각클라이언트유형에대한별도의API 게이트웨이
* 신뢰성(Reliability) : 네트워크또는서비스오류가다른서비스로연쇄되는것을방지하는방법은무엇인가?
  + 서킷브레이커(Circuit Breaker) : 원격호출의실패율이임계값을초과하면즉시실패하는프록시를통해원격서비스를호출

