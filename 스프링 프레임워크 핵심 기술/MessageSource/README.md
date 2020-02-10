# #Spring/강의/IoC컨테이너7:MessageSource

## MessageSource

국제화(i18n) 기능을 제공하는 인터페이스 (메시지를 다국화하는 방법)

ApplicationContext extends MessageSource
- getMessage(String code, Object[] args, default, Locale, Ioc)
- …

스프링 부트를 사용한다면 별다른 설정 필요없이 messages.properties 사용할 수 있음
- messages.properties
- messages_ko_kr.properties
- …

릴로딩 기능이 있는 메시지 소스 사용하기

```java
// Application 코드
@Bean
public MessageSource messageSource() {
	var messagetSource = new ReloadableResourceBundleMessageSource();
	messagetSource.setBasename(“classpath:/messages”);
	messagetSource.setDefaultEncoding(“UTF-8”);
	messagetSource.setCacheSeconds(3);
	return messagetSource;
}

// AppRunner
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        while(true) {
            System.out.println(messageSource.getMessage(“greeting”, new String[]{“jeonghun”}, Locale.KOREA));
            System.out.println(messageSource.getMessage("greeting", new String[]{"jeonghun"}, Locale.getDefault()));
            Thread.sleep(1000);
        }
    }
}

// 결과
// 1초마다 resources 읽어와서 출력
// 빌드만하면 변경사항 바로 적용 
```



[image:1608879F-4AAF-4D5C-AD31-D08C698EB7F1-59534-0001D05BE34BAEE3/스크린샷 2020-01-31 오후 10.02.06.png]