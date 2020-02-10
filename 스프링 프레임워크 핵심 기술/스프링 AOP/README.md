# #Spring/강의/스프링AOP


## AOP (Aspect-Oriented Programming)

AOP는 OOP를 보완하는 수단으로 흩어진 Aspect를 모듈화 할 수 있는 프로그래밍기법

[image:583E1749-BECA-440B-A79F-E43CCFCB4ABC-83586-000221E623485DD7/스크린샷 2020-02-05 오후 6.38.18.png]
(팔레트에 물감 모아놓고 필요한 곳을 지정해 놓은 느낌?)

## AOP 주요 개념
- Aspect (모듈화 한것 (묶은것)) 와 Target (적용이 되는 대상)
- Advice (해야할 일들)
- Join point (합류점 (메서드 실행 시점), 메서드 실행할때 Advice 끼워넣어라) 와 Pointcut (어디에 적용해야 되는지)


## AOP 구현체
- https://en.wikipedia.org/wiki/Aspect-oriented_programming 
- 자바
	- AspectJ
	- 스프링 AOP (국한적으로 기능 제공)


## AOP 적용 방법
Aspect 를 언제 어떻게 적용할 것인가

- 컴파일 타임 : (자바 -> 클래스) 할때 바이트 코드를 조작해서 생성
- 로드 타임 : 순수하게 컴파일하고 클래스파일을 로딩하는 시점에 조작 (로드타임위빙)
- 런타임 (현실적이고 자주 쓰임) : 스프링 AOP가 사용하는 방법, 빈을 만들때 (프록시 빈) 



---
# 프록시 기반 AOP

## 스프링 AOP 특징
- 프록시 기반의 AOP 구현체
- 스프링 빈에만 AOP를 정용할 수 있다.
- 모든 AOP기능을 제공하는 것이 목적이 아니라, 스프링 IoC와 연동하여 엔터프라이즈 
애플리케이션에서 가장 흔한 문제에 대한 해결책을 제공하는 것이 목적. 

## 프록시 패턴
- 기존 코드 변경 없이 접근 제어 또는 기능 추가를 하기 위해
[image:19F5E4D4-E861-4BE6-845E-BF7CCF89EA38-83586-00023F3A1AD377FA/스크린샷 2020-02-06 오후 6.24.37.png]
- 기존 코드를 건드리지 않고 성능을 측정 (프록시 패턴)

```java
// 인터페이스
// Subject
public interface EventService {
    void createEvent();
    void publishEvent();
    void deleteEvent();
}

```

```java
// 인터페이스 구현체
// realSubject
@Service
public class SimpleEventService implements EventService {

    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(“Create an event”);
    }

    @Override
    public void publishEvent() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(“published an event”);
    }

    @Override
    public void deleteEvent() {
        System.out.println(“Delete an event”);
    }
}

```

```java
// 프록시 패턴 (구형 - 귀찮음)
// ProxySimpleEventService
@Primary // 빈 최상위로 등록
@Service
public class ProxySimpleEventService implements EventService {

    @Autowired
    SimpleEventService simpleEventService;
//    EventService simpleEventService; 상관없음

    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.createEvent();
        System.out.println(System.currentTimeMillis()-begin);
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.publishEvent();
        System.out.println(System.currentTimeMillis()-begin);
    }

    @Override
    public void deleteEvent() {
        simpleEventService.deleteEvent();
    }
}
```

## 문제점
* 매번 프록시 클래스를 작성해야 하는가? 
* 여러 클래스 여러 메소드에 적용하려면? 
* 객체들 관계도 복잡하고… 


## 그래서 등장한 스프링 AOP
* 스프링 IoC 컨테이너가 제공하는 기반 시설과 Dynamic 프록시를 사용하여 여러 
복잡한 문제 해결. 

* 동적 프록시: 동적으로 프록시 객체 생성하는 방법 
	- 자바가 제공하는 방법은 인터페이스 기반 프록시 생성. 
	- CGlib은 클래스 기반 프록시도 지원. 

* 스프링 IoC: 기존 빈을 대체하는 동적 프록시 빈을 만들어 등록 시켜준다. 
	- 클라이언트 코드 변경 없음.
	- AbstractAutoProxyCreator implements BeanPostProcessor 




---
# 스프링 AOP

애노테이션 기반의 스프링 @AOP

## 의존성 추가
```xml
// pom.xml
<dependency>
	<groupId>org.springframework.boot</groupId> 	
	<artifactId>spring-boot-starter-aop</artifactId> 
</dependency>
```

## 애스팩트 정의
- @Aspect
- 빈으로 등록해야 하니까 (컴포넌트 스캔을 사용한다면) @Component도 추가 

## 포인트컷 정의
- @Pointcut(표현식)
- 주요 표현식
	- execution
	- @annotation
	- bean
- 포인트컷 조합
	- &&, ||, !

## 어드바이스 정의
- @Before
- @AfterReturning
- @AfterThrowing
- @Around

https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-pointcuts 

```java
// PerfAspect
@Component
@Aspect
public class PerfAspect {

    // Advice
//    @Around(“execution(* me.whiteship..*.EventService.*(..))”) // pointcut 지정 - 어디에 적용하겠다
//    @Around(“@annotation(PerfLogging)”)
    @Around(“bean(simpleEventService)”)
    public Object logPerf(ProceedingJoinPoint pip) throws Throwable {
        long begin = System.currentTimeMillis();
        Object retVal = pip.proceed();
        System.out.println(System.currentTimeMillis()-begin);
        return retVal;
    }

    @Before(“bean(simpleEventService)”)
    public void hello() {
        System.out.println(“hello”);
    }
}

// 어노테이션
/*
이 어노테이션을 사용하면 성능을 측정해 줍니다
 */

// 애너테이션 정보를 얼마나 유지할 것인가 [소스 - 컴파일하면 사라짐 / 런타임 - 굳이 필요없음]
// 기본값이 class
@Retention(RetentionPolicy.CLASS)
@Documented //자바독
@Target(ElementType.METHOD) // 메서드 대상
public @interface PerfLogging {
}


```

[image:106D06E1-5FF2-4D0B-BCF4-BC6BF2950C01-83586-0002419A7AAA40C2/스크린샷 2020-02-06 오후 7.08.09.png]
-> 어노테이션으로 하면 해당 메서드에 추가해야함

