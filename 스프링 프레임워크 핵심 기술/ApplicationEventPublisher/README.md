# #Spring/강의/IoC컨테이너8:ApplicationEventPublisher
ApplicationContext가 상속받는 또다른 인터페이스
이벤트 프로그래밍에 필요한 인터페이스 제공. 옵저버 패턴의 구현체

## ApplicationEventPublisher extends ApplicationEventPublisher

- publishEvent(ApplicationEvent event)

## 이벤트 만들기

- ApplicationEvent 상속 받거나
- 스프링 4.2부터는 상속받지 않고 이벤트로 사용 가능 (권장)

## 이벤트 발생시키는 방법

- applicationEventPublisher.publishEvent(new MyEvent(this,200));

## 이벤트 처리하는 방법

- ApplicationListener<이벤트> 구현한 클래스 만들어서 빈으로 등록하기
- 스프링 4.2 부터는 @EventListener를 사용해서 빈의 메서드에 사용 가능
- 기본적으로는 synchronized (메인 스레드로 실행)
- 순서를 정하고 싶으면 @Order(Ordered.HIGHEST_PRECEDENCE) (더하면 다음)
- 비동기적으로 실행하고 싶으면 @Async 후 @SpringBootApplication
에 @EnableAsync 추가

## 스프링이 제공하는 기본 이벤트

- ContextRefreshedEvent : ApplicationContext를 초기화 했거나 리프래시 했을때 발생
- ContextStartedEvent : ApplicationContext를 start()하여 라이프사이클 빈들이 시작 신호를 받은 시점에 발생
- ContextStoppedEvent : ApplicationContext를 stop()하여 라이프사이클 빈들이 정지신호를 받은 시점에 발생
- ContextClosedEvent : ApplicationContext를 close()하여 싱글톤 빈이 소멸되는 시점에 발생
- RequestHandledEvent : HTTP 요청을 처리했을때 발생

```java
// AppRunner
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationEventPublisher applicationEventPublisher;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        applicationEventPublisher.publishEvent(new MyEvent(this,200));
    }
}

// MyEvent
public class MyEvent {

    private int data;
    private Object source;

    public MyEvent(Object source, int data) {
        this.source = source;
        this.data = data;
    }

    public int getData() {
        return data;
    }
}

// MyEventHandler
@Component
public class MyEventHandler {

    @EventListener
    @Async
    public void handle(MyEvent myEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println(“이벤트 받았다. 데이터는 “+myEvent.getData());
    }

    @EventListener
    @Async
    public void handle(ContextRefreshedEvent contextRefreshedEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println(“ContextRefreshedEvent”);

    }

    @EventListener
    @Async
    public void handle(ContextClosedEvent contextClosedEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println(“ContextClosedEvent”);

    }
}


```

