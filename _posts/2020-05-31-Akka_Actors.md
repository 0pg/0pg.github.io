---
title: "Akka - Actors"
categories:
  - akka
tags:
  - akka actor
last_modified_at: 2020-05-31T00:00:00+09:00
toc: true
toc_sticky: true
---
# Akka 란?
* LightBend 사에서 ActorModel 을 구현한 오픈소스 프로젝트
	* Scala 로 구현되었으며 Java API 를 제공
* Actor Model 이란?
	* actor 를 기본 단위로하여 message 를 이용한 비동기 통신을하는 모델
	* multi-core 환경에서 최대의 효율을 내기 위한 방안으로써 주목받음
		* 기본적으로 공유자원이 없어 멀티스레드 환경에서 이로인한 문제를 방지할 수 있음
		* 비동기 통신으로 스레드 블락을 최대한 피할 수 있음
* 높은 동시성을 제공한다
* 확장성이 좋다
	* 장소 투명성을 제공 (location transparency)
	* 다른 Actor 의 위치에 대해 알지 못해도 됨
* fault-torlerance 제공
	* Actor 는 다른 Actor 가 어떤 일을 하는지 궁금하지 않음

## akka-typed
* 구체적인 Type 을 가지는 Actor Api 를 제공한다
* 흔히 얘기하는 Akka 라 하면 akka-typed 를 의미
	* 기존의 Untyped Actor Api 는 Akka Classic 으로 명명하여 구별됨
* 파라미터로써 메세지에 대한 타입을 추가
* Actor 가 Behavior 를 통해서 정의되도록 변경
* 메세지에 송신 Actor 의 ActorRef 를 담는 방식으로 통신
	* Akka Classic 에는 sender() 메서드로 송신 Actor 주소를 알 수 있었는데, type 이 추가되며 어떤 Actor 라도 메세지를 전송 할 수 있기때문에 수신 Actor 입장에서 type 을 특정할 수 없어 deprecated 되었다고 함
* Akka Classic 과 호환이 가능하며 혼용할 수 있음

## Akka Basic Component
### ActorSystem
* 생성한 Actor 들을 관리하는 컨테이너
	* 모든 Actor 는 ActorSystem 내부에서 동작
	* Actor 들은 ActorSystem의 thread scheduling, configuration 등을 공유한다
* JVM 당 하나의 ActorSystem 을 생성하여 사용하는 것이 일반적

### ActorRef
* Actor 객체를 감싸는 proxy
* Actor 객체는 ActorRef 에 의해 캡슐화되어 외부에서 접근할 수 없다
	* 메세지 교환 시 목적지는 Actor 객체가 아닌 ActorRef 가 된다
* Akka 에서의 장소 투명성의 기반이 되는 역할

### Actor
* Akka 어플리케이션의 기본 단위
* 계층 구조를 가진다
	* 계층 구조 아래에서 Actor(ActorRef) 는 고유한 path 를 가지는데, 이 path 가 Actor(ActorRef) 의 식별자가 된다
* 메세지를 교환하여 통신한다
	* 다른 Actor 에 유한한 수의 메세지 전송
	* 다음 메세지에 대한 내부 동작 및 상태를 변경
	* 다른 Actor 를 생성 (child actor)
* Actor 는 명시적으로 종료해주어야 한다
	* 더 이상 참조가 없어도 release 되지 않음
* 메세지 전송에 있어 기본적으로 At-most-once 룰을 따른다
**Actor's happens-before**
* 동일한 Actor 쌍 사이에서 메세지의 송신은 수신보다 먼저 일어남이 보장된다
* 동일한 Actor 내에서 다음 메세지는 현재 메세지가 처리완료된 이후에 처리됨이 보장된다
**메세지와 메세지 사이의 Actor 의 상태에 대한 스레드 사이의 가시성이 보장됨**
* 단, Future 에 의한 접근은 별도의 동기화나 가시성 보장이 필요하다
* Actor 는 한번에 하나의 메세지만, 단일 스레드에 의해 실행된다
	* 한 스레드가 메세지를 처리하는동안 다른 스레드에 의한 접근은 제한됨


**Actor Internal Component**
* State
	* Actor 내부의 상태
* Behavior
	* 메세지를 받은 Actor 가 수행해야 할 동작이 정의되어 있음
	* 동적으로 바꿀 수 있음
	* 사실상 Actor 는 behavior 들의 집합
* Mailbox
	* 수신한 메시지들이 쌓여있는 버퍼
		* Akka 는 다양한 Mailbox 구현체들을 제공
		* Custom Mailbox 를 구현하여 사용할 수 있음
	* 같은 Actor 쌍 사이의 메세지들은 송/수신 순서가 보장된다
* Child Actor
	* 모든 Actor 는 parent - child 관계를 가짐
	* Child Actor 는 Parent Actor path 의 하위 path 를 가짐
* Supervisor Strategy
	* parent 는 child 를 supervise 할 수 있다
	* Actor 의 예외를 감지하고 수행해야 할 동작이 정의되어 있음
		* 무시, 중단, 재시작, 전파 등


# Actor Structure
## Guardian Actor
* ActorSystem 에 의해 생성되는 최상위 레벨 Actor
* Root Guardian(/) 이라고도 하며 부모 Actor 가 없는 유일한 Actor
	* Guardian Actor 가 종료되면 ActorSystem 도 종료된다
* User Guardian 과 System Guardian 의 supervisor
￼￼￼￼

### System Guardian
* 프레임워크 레벨 기능이나 유틸리티 제공을 위해 생성되는 system actor 들을 관리 (/system)
	* logging listener, ActorSystem 시작 시 설정에 따라 자동 실행되는 actor 들, ...

### User Guardian
* 실제 application logic 이 구현되어있는 Actor 들의 root
	* 일반적으로 Root Actor 라고 하면 User Guardian(/user) 을 의미
* Root Actor 의 child 를 Top Level Actor 라고 한다
	* ActorSystem 의 context 로 생성된 Actor

## Actor Path
* Actor 는 식별자로써 유일한 ActorPath 를 가지며, 메세지를 수신하기위한 주소로 사용되기도 한다
* Logical Path 와 Physical Path
	* logical path 는 root guardian 으로부터 현재 actor 까지의 path로, root 에 도달할 때 까지의 parental path
	* physical path 는 현재 actor 의 actorSystem 의 주소를 포함한 path로, 현재 actor 의 실제 물리적인 위치
* Akka 는 Logical Path 만으로 통신할 수 있는 api 를 제공하며, logical path는 장소 투명성을 제공할 수 있게하는 중요한 개념 중 하나이다

## Actor Lifecycle
### Actor lifecycle signal
* Akka Classic 의 postStart, postRestart signal 은 제거
	* Actor 생성 시 생성자에서 초기화하는 것으로 대체
* PreRestart
	* restart 가 시작되기 전에 preRestart signal 을 받는다
	* 일반적으로 리소스 정리를 위한 훅
	* restart 시 child actor 들은 모두 정리된다
* PostStop
	* Actor 가 멈추면 postStop signal 을 받는다
	* 일반적으로 리소스 정리를 위한 훅
	* restart 상황에는 postStop 시그널을 받지 않음

### Create Actor
* ActorSystem.create()
	* top level actor 생성
* ActorContext.spawn()
* ActorContext
	* child actor 생성 및 supervision
	* 다른 actor 종료 감지
	* 로깅
	* 메세지 어댑터 생성
	* request-response interaction 제공
	* 자신의 ActorRef 를 받아올 수 있음
	* thread-safe 하지 않음

### Stopping Actor
* Behaviors.stopped() 를 반환
* ActorContext.stop()
* parent actor 의 종료

### Restarting Actor
* parent actor 가 restart 되면 child actor 들은 stop 시킨다
	* 일반적으로 생성자에서 child spawn 을 수행한다
	* parent actor 초기화 시 child actor 들을 다시 생성하는데, 기존 child actor 는 정리가 안된채로 남아있을 수 있음
	* 생성자나 withStopChildern() 옵션으로 restart 시 child spwan 코드 실행을 피할 수 있다

### Watching Actor
* 다른 Actor 의 종료 상태를 전달받음
* 재시작이나 한시적 장애상황이 아닌 종료
* watch(targetActor)
	* targetActor 가 종료되면 Terminated signal 을 받음
	* Terminated signal 객체는 targetActor 의 정보를 포함하고 있음
* watchWith(targetActor, customTerminated)
	* watch 와 거의 동일하지만, targetActor 종료 시 CustomTerminated signal 객체를 받음
* unwatch(targetActor)
	* targetActor 에 대한 종료 시그널을 받지 않음
	* targetActor 가 이미 종료되었을 경우 Terminated signal 을 받지만 signal handler 에 의해 실행되지는 않는다
	* 같은 이유로 mailbox 에 이미 enqueue 되어 있는 targetActor 의 Terminated 메세지는 무시 (handler 에 의한 실행 X)
**Signal**
* signal 은 System 에 의해 전송되는 메세지
* 수신을 보장한다는 점에서 일반 메세지와 구별됨

## Interaction Pattern
### tell
* ActorRef.tell(message)
* Fire-and-Forget
	* 메세지만 전송하고 이후는 생각하지 않는다
	* 수신 Actor 가 메세지를 받지 못해도 알 방법이 없음

### ask
* 다른 Actor 에 메세지를 보내고 메세지에 대한 응답을 기다린다
* ActorContext.ask()
	* ActorContext 에 메세지 처리 후 수행할 callback 을 등록한다
	* 동일한 ActorContext 를 공유하는 Actor 사이 ask 통신할 때 사용
* AskPattern.ask()
	* 같은 ActorContext 를 공유하고 있지 않은 경우
	* 응답을 기다리지 않고 Future 를 반환
		* ActorSystem 에 의해 temporary actor가 생성되며 전송자의 proxy 처럼 동작한다
		* proxy actor 가 수신 actor 의 response 메세지를 받아서 처리

## Coordinated Shutdown
* ActorSystem(JVM) 종료 과정을 여러 phase 로 구분하고, phase 별로 task 를 실행할 수 있도록 되어있다
* 새로운 phase 또는 각 phase 에 임의의 task 를 정의해 추가할 수 있다
	* ActorSystem.terminate()
		* akka.coordinated-shutdown.exit-jvm=on 옵션을 통해 JVM까지 종료시킬 수 있다
	* JVM process 가 kill signal 을 받아 종료될 때
		* akka.coordinated-shutdown.run-by-jvm-shutdown-hook=off 옵션을 통해 coordinated shutdown 을 생략할 수 있다
	* 임의로 CoordinatedShutdown 을 호출하여 실행시킬 수 있음
* akka.coordinated-shutdown.phases 에 각 phase 가 정의되어 있다
* 단, phase 사이 순환관계가 생기면 안된다
* depends-on 에 phase 의 작업들이 끝나기 전엔 해당 phase 는 실행되지 않음
	* 단, 설정된 timeout 이 발생하면 의존 phase 들과 관계 없이 해당 phase 실행

## Fault Tolerance
### Akka fault tolerance
* 장애 격리
	* supervisor 가 actor 를 종료할지 말지 결정할 수 있다
* 다중화
	* 기존 actor 를 다른 actor 로 교체할 수 있다
* 구조
	* 계층구조를 통한 관리로 다른 actor 에 영향을 최소화하며 대상 actor 만 교체할 수 있다
* 일시 정지
	* actor 가 동작을 수행할 수 없는 경우 supervisor 가 동작을 결정할 때 까지 해당 actor 의 mailbox 는 정지 상태가 된다
* 관심사 분리
	* actor 의 메세지 처리와 오류 처리 흐름이 분리되어 독립적으로 개발할 수 있다
### Supervision
* Actor 내부의 특정 예외에 대한 동작 정의
	* 기본 동작은 stop 으로 정의되어 있다
* child 생성 시 supervise() 로 감싸서 설정할 수 있음

### 계층구조에 따른 예외 전파
* child actor 에서 발생한 예외에 대한 처리가 supervisor 에 정의되어있지 않으면 parent actor 로 전달한다
* parent actor 는 child 의 ChildFailed signal 을 처리하거나 supervisor 를 통해 child actor 의 장애를 처리할 수 있다
* ChildFailed signal
	* parent actor 가 child actor 에 대한 watch() 를 등록했을 때
	* child actor 의 종료 시그널이며 Terminated 를 확장한 클래스
	* parent actor 에서 child actor 와 다른 actor 들의 종료 시그널을 구별하기 위한 목적
* 만약 parent actor 가 child actor 의 Terminated signal 을 처리하지 않는다면 child 는 DeathPactException 을 발생시키며 종료
* child actor 의 original exception 은 Terminated signal 객체로부터 얻을 수 있음
	* original exception 에 대한 처리를 하려면 Terminated signal 처리를 등록해야 함

## Actor Discovery
* ActorRef 는 actor 생성 또는 Receptionist 를 통해 얻을 수 있다
### Receptionist
* Receptionist 는 ServiceKey 객체를 이용해 ActorRef 를 등록하거나 조회하는 기능을 제공하는 Actor 이다
	* Actor 이기때문에 등록, 조회 등의 동작도 메세지 전송을 통해 이루어짐
	* 조회시 전달한 key 에 등록된 ActorRef Set 을 포함하는 Receptionist.Listing 객체를 전달
	* Cluster 환경에서 Actor 를 찾을 때 주로 사용됨
* register(key, actorRef)
	* Receptionist 에 key 와 actorRef 를 등록
	* 동일한 key 로 여러 Actor Reference 를 등록할 수 있다
* subscribe(key, subscriber)
	* Receptionist 목록에 변화가 있을 때마다 subscriber 에게 변화된 Receptionist.Listing 을 수신한다
* find(key, replyTo)
	* 현재 상태의 Receptionist.Listing 을 조회한다
* deregister(key, actorRef)
	* Receptionist 에 등록된 actorRef 를 제거한다
	* 제거 후에도 일정 기간 subscriber 로부터 메세지를 수신할 수 있음
		* Receptionist 는 deregister 에 대한 정보를 subscriber 들에게 메세지로 전달 (비동기)

## Routers
* routee actor 들로 메세지를 포워딩하는 actor
### Pool Router
* Routee 를 직접 생성하고 관리
	* Routee 의 parent actor
* Routee 의 behavior 와 함께 생성된다
	* Router 가 parent actor 지만 Router 의 parent actor 에서 routee 의 behavior 를 정의
* 지정된 수 만큼의 routee 를 생성하여 메세지를 라우팅
	* 종료된 routee 는 router 에서 제거됨
	* 모든 routee 가 종료되면 router 도 스스로 종료한다
* routee 를 생성할 때 routee의 dispatcher 를 설정할 수 있다

### Group Router
* ServiceKey 와 함께 생성되어 Receptionist 를 통해 메세지를 라우팅할 가용한 actor 를 찾는다
	* reachable 한 actor 가 하나도 없으면 그냥 unreachable 한 actor 메세지를 전송함
	* ActorRef list 가 비어있으면 그것을 Dropped stream 에 publish 한다
* Cluster 에 참여중인 노드들의 actor 에 라우팅 할 수 있다

### Routing Strategies
* Round Robin
* Random
* Consistent Hashing

## Stash
* mailbox 에서 꺼낸 메세지를 임시로 저장할 수 있는 버퍼
	* Actor 의 현재 behavior 로 처리할 수 없는 메세지의 경우
	* stash 된 메세지들은 메모리에 계속 들고있게 되므로 주의해야 함
* unstahAll() 은 stash 된 순서대로 메세지를 꺼낸다

## Dispatchers
* actor 가 실행될 수 있게 thread 를 할당한다
* 모든 MessageDispatcher 는 Executor 를 구현하고 있음
* **Default Dispatcher**
	* Actor System 의 기본 dispatcher
	* fork-join-executor
* **Internal Dispatcher**
	* Akka module 들에 의해 내부적으로 생성되는 actor 들을 보호하기 위해 default dispatcher 와 분리시킨 dispatcher
	* 별도의 설정이 없으면 내부 모듈 actor 들은 internal dispatcher 를 사용
	* internal dispatcher 의 strategy 도 설정할 수 있다

> **blocking operation 은 해당 dispatcher 를 공유하는 다른 Actor 의 실행에도 영향을 끼치므로 주의** 
>
>blocking operation 이 필요한 actor 들은 dispatcher 를 분리하여 처리하는 것을 고려해볼 수 있다  

### Types of dispatchers
* Dispatcher
	* 여러 Actor 가 공유 가능한 dispatcher
* **PinnedDispatcher**
	* 각 Actor 마다 가지는 공유 불가능한 고유한 dispatcher
	* Actor 마다 독립적인 dispatcher 할당이 가능

## Mailbox
* Actor 가 아직 처리하지 않은 메세지를 들고있는 버퍼
* default mailbox 는 unbounded (**SingleConsumerOnlyUnboundedMailbox**)
	* bounded mailbox 도 제공함
	* bounded mailbox 가 가득 차면 이후 메세지는 deadletter 로 처리

### Selecting a Mailbox Type for an Actor
* MailboxSelector 를 통해 특정 mailbox 를 선택할 수 있다
* custom mailbox 를 구현해 사용할 수 있다
* Akka 는 다양한 mailbox 구현체를 제공

### Mailbox Implementations
* **Non-Blocking mailboxes**
	* SingleConsumerOnlyUnboundedMailbox
		* BalancingDispatcher 와 사용 불가능
	* UnboundedMailbox
	* NonBlockingBoundedMailbox
	* UnboundedControlAwareMailbox
		* ControlMessage 를 우선적으로 전달
	* UnboundedPriorityMailbox
		* 우선순위가 같은 메세지들에 대해 별도로 정의된 동작 없음
	* UnboundedStablePriorityMailbox
		* 우선순위가 같은 메세지들에 대해 FIFO 순서 처리
* **Blocking mailboxes**
	* BoundedMailbox
	* BoundedPriorityMailbox
	* BoundedStablePriorityMailbox
	* BoundedControlAwareMailbox
mailbox 가 가득 찼을 때 sender 를 block 시킴 mailbox-push-timeout-time=n

## Testing
* Akka는 테스트를 위한 test toolkit 을 제공한다
* Junit 과 함께 사용할 수 있다

### Synchronous behavior testing
* 단일 Actor 단위의 유닛테스트를 위한 툴
	* child actor 생성
	* message 전송 및 처리 결과
* **BehaivorTestKit**
	* test 환경에서 ActorContext 를 대체
	* child 생성, 감시, 내부 변화 (상태, 이벤트, ...) 에 대한 접근을 제공
* **TestInBox**
	* test 환경에서 ActorRef 를 대체
	* 수신한 메세지에 대한 접근을 제공
	* 자체 인스턴스를 반환하는 팩토리 메서드가 있음

### Asynchronous testing
* ActorSystem 단위의 통합 테스트를 위한 툴
* test framwork integration 제공
	* 다른 test framework 과 호환가능 (JUnit, ..)
* **ActorTestKit**
	* test 환경에서 ActorSystem 대체
	* behaivor mocking 지원
	* ActorTestKit 을 root 로 하는 actor test structure 를 만들 수 있다
	* mock behavior 를 제공하여 테스트를 위한 불필요한 의존성을 대체할 수 있게 한다
* **TestProbe**
	* async test 환경에서 ActorRef 를 대체
	* ActorTestKit 으로부터 생성
	* 수신한 메세지에 대한 접근을 제공

> **Actor instance 에 대한 wrapper 들을 test kit 으로 교체함으로써 내부의 actor 에 직접 접근하여 상태를 확인할 수 있게 하는 구조이다**
