# #Spring/IoC_DI_Bean
[Spring 예제로 배워보는 IoC/DI & Spring Bean Life Cycle | Sehun Kim](https://sehun-kim.github.io/sehun/springbean-lifecycle/)

1. **Java Bean** vs **Spring Bean**

Java Bean은 데이터를 표현하는 것을 목적으로 하는 자바 클래스

```
<Java Bean 규약>
1. 기본생성자가 존재해야한다.
2. 모든 멤버변수의 접근제어자는 private이다.
3. 멤버변수마다 getter/setter가 존재해야한다. 
(속성이 boolean일 경우 is를 붙힘)
4. 외부에서 멤버변수에 접근하기 위해서는 메소드로만 접근할 수 있다.
5. Serializable(직렬화)가 가능해야한다.
```

— 직렬화

시스템 내부에서 사용하는 객체 혹은 데이터를 외부의 시스템에서도 사용할 수 있도록 변환시키는 것을 말함. 

자바에서 JVM의 Heap 영역에 상주한 객체를 byte 형태로 변환(직렬화)
byte 형태를 다시 자바 객체로 변환(역직렬화)
`CSV, Json 포맷으로 자바 객체를 변경시키는 것도 직렬화하는 것`
Serializable interface를 implements 한 클래스는 직렬화 가능

```java
import java.io.Serializable;

// 직렬화가 가능하도록 Serializable 인터페이스를 구현
public class Person implements Serializable {
    // 모든 멤버변수의 접근자는 private
    private String name;
    private int age;
    private String address;

    // 기본생성자가 있어야한다.
    public Person() {
    }

    // 기본생성자가 있다면 매개변수가 있는 생성자가 있어도 무방함
    public Person(String name, int age, String address) {
        this.name = name;
        this.age = age;
        this.address = address;
    }

    // 각 멤버변수에 접근할 수 있는 getter/setter가 있어야한다.
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

Spring은 뷰 영역 (JSP)에 데이터를 출력하고 싶을 때 Java Bean 규약에 맞춰 만들어진 객체를 사용, 객체들을 외부 저장소에 저장하고 전송할 때 사용

Spring Bean은 Spring Framework의 Container 에 의해 등록, 생성, 조회, 관계설정이 되는 객체. Java Object 와 동일하지만 IoC 방식으로 관리되는 오브젝트를 의미함.
(Spring Bean은 별다른 생성 규칙이 없음)

2. IoC / DI

Spring 삼각형

[image:44333A7B-515C-48ED-9D16-184A595A3CC8-35508-0000D59048A74F6B/스크린샷 2020-01-06 오후 8.54.17.png]	
		`스프링이란 IoC와 AOP를 지원하는 경량의 컨테이너 프레임워크이다`

IoC (제어의 역전, 대신 해준다)
DI (의존성 주입, 대신 넣어준다)
-> Spring 에서 대신해주는 것은 미리 찜해 놓은 객체를 생성하고 관계를 설정시켜주고 소멸시킨다

보통의 프로그램 실행흐름은 무언가 필요한 쪽에서 객체를 만들고, 만들어진 객체의 메서드를 직접 호출해서 사용. 여기서 객체는 프로그램의 흐름에 능동적으로 참여

- 의존 역전의 원칙 (DIP) 을 적용하여 작성한 코드

```java
interface SoccerBall {
  void touchBall();
}

class AdidasSoccerBall implements SoccerBall {
  public void touchBall() {
    System.out.println(“아디다스 축구공이 굴러간다!”);
  }
}

class NikeSoccerBall implements SoccerBall {
  public void touchBall() {
    System.out.println("나이키 축구공이 굴러간다!");
  }
}

class SoccerPlayer {
  private SoccerBall ball;

  public void setSoccerBall(SoccerBall ball) {
    this.ball = ball;
  }

  public void playSoccer() {
    System.out.println("축구선수가 공을 찼다!");
    this.ball.touchBall();
  }
}

public class Driver {
  public static void main(String[] args) {
    SoccerPlayer sp = new SoccerPlayer();

    // NikeSoccerBall
    SoccerBall nikeBall = new NikeSoccerBall();
    sp.setSoccerBall(nikeBall);
    sp.playSoccer();

    // AdidasSoccerBall
    SoccerBall adidasBall = new AdidasSoccerBall();
    sp.setSoccerBall(adidasBall);
    sp.playSoccer();
  }
}
```

- Spring 코드

```java
interface SoccerBall {
  String touchBall();
}

@Component(“adidasBall”) // adidasBall이란 이름을 가진 Bean으로 등록
public class AdidasSoccerBall implements SoccerBall {
  public String touchBall() {
      return "아디다스 축구공이 굴러간다!";
  }
}

@Component("nikeBall") // nikeBall이란 이름을 가진 Bean으로 등록
public class NikeSoccerBall implements SoccerBall {
  public String touchBall() {
      return “나이키 축구공이 굴러간다!”;
  }
}

@Component // 의존성을 주입받는 객체도 Bean으로 등록되어야 한다.
public class SoccerPlayer {
    @Autowired
    @Qualifier(“nikeBall”)
    private SoccerBall ball;

    public String playSoccer() {
        return "축구선수가 공을 찼다! \n" + this.ball.touchBall();
    }
}

@RestController
public class SoccerController {
    @Autowired // SoccerPlayer라는 타입을 가진 Bean을 찾아서 주입시킴
    private SoccerPlayer soccerPlayer;

    @RequestMapping("/soccer")
    public String soccerDriver() {
        return soccerPlayer.playSoccer();
    }
}
```

기존 코드에서는 main()에서 축구선수가 축구공을 set받아 사용하였지만 이번에는 어떤 곳에서도 SoccerBall 객체를 생성하지 않는다.

해당 코드에선 보이지 않겠지만 @Component라는 어노테이션이 붙은 클래스들은 Spring의 Container가 알아서 Spring Bean 객체로 등록하고 생성한다. 이렇게 생성된 객체는 @Autowired라는 어노테이션이 붙은 변수의 타입(타입이 같은 Bean 여러개 있다면 이름을 본다. @Qualifier
)을 보고 해당 변수에 객체를 주입하게 된다.

[image:803EF4A6-3D40-4EB1-97D9-66A014BA28C3-35508-0000D9BE8D799068/스크린샷 2020-01-06 오후 10.11.43.png]

스프링의 Container가 대신 객체를 생성해주고 알아서 객체를 주입해준다. 이렇게 생성된 객체는 자신이 어디에 쓰일지 알지 못한다. 이것이 제어 역전의 원칙 이며 스프링은 DI(의존성 주입) 라는 개념으로 구현하고 있다.

### IoC, DI

[IoC(제어의 역전), DI(의존성 주입) 이란??](https://pks424.tistory.com/entry/IoC-DI%EB%9E%80)

스프링은 거대한 컨테이너임과 동시에 IoC/DI를 기반으로 하고 있는 거룩한 존재이며 서비스 추상화를 통해 분리되는 3단 변신로봇이다 - 토비의 스프링

자바의 아버지 - 제임스 고슬링
스프링 혁명가 - 로드 존슨

- 컨테이너

작성한 코드의 처리과정을 위임받은 독립적인 존재
-> 적절한 설정만 되어 있으면 작성한 코드를 스스로 참조해 객체 생성과 소멸을 제어함

스프링과 더불어 대표적인 컨테이너 **톰캣**과 같은 WAS(Web Application Server)

- IoC / DI 

둘다 스프링 만의 고유한 방식은 아님

IoC (제어의 역전) : 대신 해줌

컨테이너는 작성한 코드 컨트롤(객체의 생성과 소멸)을 대신 해줌. (언제 호출될지 모르는 코드 작성 가능해짐)

DI (의존성 주입) : 미리 찜해 놓음

주는 놈은 생각도 않았는데 먼저 말부터 꺼내놓는 방식.
-> 객체의 메서드가 어떻게 작성되었는지 알 필요 없이 클래스의 기능을 추상적으로 묶어둔 인터페이스를 가져다 쓰는 방식

ex) 인터페이스 선언후 구체적으로 구현하고, 그것을 사용하는 것

정리하자면, 
언제 사용될지 모르는 코드를 작성 (IoC / 알아서 해줄거니까)
코드가 뭔지도 모르면서 가져다 쓰는 방식 (DI)

- 서비스 추상화

추상화란 하위시스템의 공통점을 뽑아 분리하는것 -> 하위시스템을 몰라도 (막 바뀌어도) 일관된 방법으로 접근 가능




3. Container

앞서 Spring Bean이 *스프링 컨테이너(스프링 IoC 컨테이너)에* 의해 관리되는 객체란 것을 배웠다. 그럼 이런 역할을 해주는 Container는 무엇인가?

[image:E479AAB5-E59C-43DC-970C-B95C35A42018-35508-0000D9D7300E8041/스크린샷 2020-01-06 오후 10.13.30.png]
여러가지로 불림. Spring Container, DI Container, IoC Container, Bean Container ..

스프링 컨테이너는 
::프로그래머가 작성한 코드의 처리과정을 위임받아 독립적으로 처리하는 존재::

사용하는 이유

[image:E3E1F69D-F156-41CC-B800-AF1DBC8369EC-35508-0000D9F6634A17AB/스크린샷 2020-01-06 오후 10.15.44.png]
 
객체를 사용하기 위해 new 연산자, 생성자, getter/setter 를 사용해야했음.
한 어플리케이션에는 이러한 객체가 무수히 많이 존재하고, 서로 참조하고 있을 것임.
(GTA 에서 차, 사람 등등)
정도가 심할수록 의존성이 높은것. 낮은 결합도와 높은 캡슐화는 OOP에서 중요함
(유지보수 경험이 적으면 DI 필요성 느끼기 힘들지만, 테스트 코드 구현시 느낄수 있음)

객체간 의존성을 낮추기 위해 Spring 컨테이너가 사용됨
```
— 코드가 깔끔해지고 사용하기 쉬움
— 재사용하기 좋다
— 테스트하기 쉽다
```

— 스프링 IoC 컨테이너 종류

> **Bean Factory**
> Bean 객체를 생성하고 관리하는 인터페이스. 팩토리 패턴을 구현한것.
> 클라이언트 요청이 있을때 객체를 생성함 (Lazy init)

> **ApplicationContext** : 가장 많이 쓰임
> BeanFactory를 상속받은 인터페이스. 부가적인 기능이 많아 자주 사용.
> 구동되는 시점에 등록된 Bean 객체들을 스캔하여 객체화함 (Eager init)

— Configuration MetaData

Container에 Bean의 메타정보를 등록하기 위한 설정방법 두 가지

1. xml 설정파일을 통한 등록
2. Java Config(.java파일과 어노테이션)을 이용한 등록 - 많이 쓰임 (spring boot 표준)
```java
@Configuration
public class WebConfig {
  @Bean(name = “beanA”)
  public BeanA beanA {
    return new BeanA();
  }

  @Bean(name = “beanB”)
  public BeanB beanB(BeanA beanA) {
    BeanB beanB = new BeanB();
    beanB.setBeanA(beanA);
    return beanB;
  }
}
```

ex) @Bean, @Component, @Controller, @Service, @Repository

- @Bean은 개발자가 컨트롤 할 수 없는 외부 라이브러리 Bean으로 등록하고 싶은 경우
(메소드로 return 되는 객체를 Bean으로 등록)
- @Component는 개발자가 직접 컨트롤할 수 있는 클래스(직접 만든)를 Bean으로 등록하고 싶은 경우
(선언된 Class를 Bean으로 등록)
- @Controller, @Service, @Repository 등 은 @Component를 비즈니스 레이어에 따라 명칭을 달리 지정해준 것
- @Recource : 이름으로 참조할 Bean을 검색하여 주입한다. (JAVA 표준)
- @Autowired : 타입으로 참조할 Bean을 찾아 주입한다. (SPRING 표준)

4. Spring Bean LifeCycle

[image:8161F6B9-3B8C-4D52-A9FB-5FD8CAD68272-35508-0000DB2D74DC5A2C/스크린샷 2020-01-06 오후 10.38.00.png]
[image:333FABB6-38F4-47D3-9A25-200B0E592D2C-35508-0000DB313DEBC307/스크린샷 2020-01-06 오후 10.38.17.png]


— 의존 관계에 따른 생명주기 변화

- 의존관계가 없는 경우

[image:BB049462-470C-4C70-9DD3-4B2771142B60-35508-0000DB5269F48655/nonedistart.png]

[image:9ABC13B7-0122-4E6E-A94A-46542962F94E-35508-0000DB53A995A3BF/nonediquit.png]

- 의존관계가 있는 경우

[image:407D7267-7532-44E1-837E-6913AFBD1D89-35508-0000DB5687D029EB/distart.png]

[image:D06EE7C2-22C4-4EE4-A460-409574F8623B-35508-0000DB57D230F41C/diquit.png]

