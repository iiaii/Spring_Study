# #Spring/강의/IoC컨테이너6:Environment-프로파일

## 프로파일 (어떤 환경)
- 빈들의 묶음
- Environment 의 역할은 활성화할 프로파일 확인 및 설정

## 프로파일 유즈 케이스
- 테스트 환경에서는 A라는 빈을 사용하고, 배포환경에서는 B라는 빈을 사용하고 싶다
- 이 빈은 모니터링 용도니까 테스트할 때는 필요가 없고 배포할 때만 등록되면 좋겠다

## 프로파일 정의하기
```java
// 러너 코드
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationContext ctx;

    @Autowired
    BookRepository bookRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Environment environment = ctx.getEnvironment();
        System.out.println(Arrays.toString(environment.getActiveProfiles()));
        System.out.println(Arrays.toString(environment.getDefaultProfiles()));
    }
}
```
- 클래스에 정의하기
	- @Configuration @Profile(“test”)
```java
@Configuration
@Profile(“test”)
public class TestConfiguration {

	@Bean
	public BookeRepository bookRepository() {
		return new TestBookRepository();
	}
}
```
	- @Component @Profile(“test”)
```java
@Repository
@Profile(“test”)
public class TestBookRepository implements BookRepository {
}
```
- 메소드에 정의
	- @Bean @Profile(“test”)
```java
@Configuration
public class TestConfiguration {

	@Bean
	@Profile(“test”)
	public BookeRepository bookRepository() {
		return new TestBookRepository();
	}
}
```

## 프로파일 설정하기
- -Dspring.profile.active=“test,A…”
- @ActiveProfiles (테스트용)

## 프로파일 표현식
- ! : not (!prod : prod 아니고는 다 됨)
- & : and
- | : or
	