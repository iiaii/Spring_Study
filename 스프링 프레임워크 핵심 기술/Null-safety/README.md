# #Spring/강의/null-safety


## 스프링 프레임워크 5에 추가된 Null 관련 애노테이션 
* @NonNull 
* @Nullable 
* @NonNullApi (패키지 레벨 설정) 
* @NonNullFields (패키지 레벨 설정) 

## 목적 

* (툴의 지원을 받아) 컴파일 시점에 최대한 NullPointerException을 방지하는 것 

```java
@Service
public class EventService {

    @NonNull
    public String createEvent(@NonNull String name) {
        return “hello”+name;
    }
}


@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        eventService.createEvent(null);
    }
}

```