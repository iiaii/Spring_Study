# #Spring/강의/Resource추상화

ApplicationContext는 빈 팩토리 기능만 하는 것이 아니라 ResourceLoader, EventPublisher, MessageSource 등의 기능을 한다 (단순한 빈 팩토리가 아님)


org.springframework.core.io.Resource

## 특징
- java.net.URL을 추상화 한 것 (org.springframework.core.io.Resource로 감싼것)
- 스프링 내부에서 많이 사용하는 인터페이스

## 추상화 한 이유
- 클래스패스 기준으로 리소스 읽어오는 기능 부재
- ServiceContext 기준으로 상대 경로로 읽어오는 기능 부재
- 새로운 핸들러를 등록하여 특별한 URL 접미사를 만들어 사용할 수 있지만 구현이 복잡하고 편의성 메서드가 부족


## 인터페이스부재
[Resource (Spring Framework 5.2.3.RELEASE API)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/Resource.html)

- 상속 받은 인터페이스
- 주요 메서드 (많이 쓰임)
	- getInputStream()
	- exist()
	- isOpen()
	- getDescription() : 전체 경로를 포함한 파일 이름 또는 실제 URL
	
## 구현체
- UrlResource : java.net.URL 참고. 기본으로 지원하는 프로토콜 http, https, ftp, file..
- ClassPathResource : 지원하는 접두어 classpath:
- FileSystemResource
- ServletContextReousrce (가장 많이 쓰임) : 웹 어필리케이션 루트에서 상대 경로로 리소스 찾는다.
- …

## 리소스 읽어오기
- Resource의 타입은 location의 문자열과 ApplicationContext의 타입에 따라 결정
	- ClassPathXmlApplicationContext -> ClassPathResource
	- FileSystemXmlApplicationContext -> FileSystemResource
	- WebApplicationContext -> ServletContexResource
- **ApplicationContext의 타입에 상관없이 리소스 타입을 강제하려면 java.net.URL 접두어(classpath:) 중 하나를 사용할 수 있다.**
	- classpath:me/whiteship/config.xml -> ClassPathResource (추천)
	- file:///some/resource/path/config.xml -> FileSystemResource

```java
Resource resource = resourceLoader.getResource(“classpath:test.txt”);
resource.getClass() // ClassPathResource
// classpath 접두어를 붙이면 타입이 달라짐


Resource resource = resourceLoader.getResource(“test.txt”);
resource.getClass() // ServletContextResource
```


