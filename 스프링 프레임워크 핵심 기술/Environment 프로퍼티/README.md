# #Spring/강의/IoC컨테이너6:Environment-프로퍼티

## 프로퍼티
- 다양한 방법으로 정의할 수 있는 설정값
- Environment의 역할은 프로퍼티 소스 설정 및 프로퍼티 값 가져오기

## 프로퍼티 우선순위
- StandardServletEnvironment의 우선순위
	- ServerletConfig 매개변수
	- ServerletContent 매개변수
	- JNDI (java:comp/env/)
	- JVM 시스템 프로퍼티 (-D{key}=“{value}”)
	- JVM 시스템 환경변수 (운영체제 환경 변수)

@PropertySource
- Environment를 통해 프로퍼티 추가하는 방법

## 스프링 부트 외부 설정 참고 
- 기본 프로퍼티 소스 지원 (application.properties)
- 프로파일까지 고려한 계층형 프로퍼티 우선순위 제공

[image:967B0888-C0C8-4EB5-895F-428B377D0EA0-59534-0001CD51462C9C4B/스크린샷 2020-01-31 오후 9.06.21.png]
[image:12EF04DF-1F97-4189-BCA9-F3429243A4E5-59534-0001CD49A9B76419/스크린샷 2020-01-31 오후 9.05.46.png]

```java
// AppRunner 코드
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationContext ctx;

    @Autowired
    BookRepository bookRepository;

    @Value(“${app.name}”)
    String appName;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Environment environment = ctx.getEnvironment();

        System.out.println(environment.getProperty(“app.name”));
        System.out.println(appName);
    }
}

// Demospring51Application.java
@SpringBootApplication
@PropertySource(“classpath:/app.properties”) // 여기 추가
public class Demospring51Application {

	public static void main(String[] args) {
		SpringApplication.run(Demospring51Application.class, args);
	}

}
```