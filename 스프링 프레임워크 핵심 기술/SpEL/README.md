# #Spring/강의/SpEL


## 스프링 EL
- 객체 그래프를 조회하고 조작하는 기능을 제공
- Unified EL 과 비슷하지만, 메소드 호출을 지원하며 문자열 템플릿도 제공
- OGNL, MVEL, JBOss EL 등 자바에서 사용할 수 있는 여러 EL이 있지만, SpEL은 모든 스프링 프로젝트 전반에 걸쳐 사용할 EL로 만들었다
- 스프링 3.0 부터 지원


## SpEL 구성
- ExpressionParser​​ parser = new SpelExpressionParser()
- StandardEvaluationContext context = new Standard​EvaluationContext​​(bean)
- Expression expression = parser.parseExpression(“SpEL 표현식”)
- String value = expression.getvalue(context, String.class)


## 문법
- # {“표현식”}
- ${“프로퍼티”}
- 표현식은 프로퍼티를 가질 수 있지만, 반대는 안됨
	- # { ${my.data} +1 }
- [레퍼런스](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html%23expressions-language-ref)


## 활용
- @Value
- @ConditionalOnExpression
- 스프링 시큐리티 [15. Expression-Based Access Control](https://docs.spring.io/spring-security/site/docs/3.0.x/reference/el-access.html)
	- 메소드 시큐리티, @PreAuthorize, @PostAuthorize, @PreFilter, @PostFilter
	- XML 인터셉터 URL 설정
	- …
- 스프링 데이터 [SpEL support in Spring Data JPA @Query definitions](https://spring.io/blog/2014/07/15/spel-support-in-spring-data-jpa-query-definitions)
	- @Query
- Thymeleaf [Thymeleaf에서 SpEL로 Enum 접근하기 :: Outsider’s Dev Story](https://blog.outsider.ne.kr/997)
- …

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Value("#{1+1}")   // #{} : 표현
    int value;

    @Value("#{'hello'+'world'}")
    String greeting;

    @Value("#{1 eq 1}")
    boolean trueOrFalse;

    @Value("#{sample.data}") // bean으로 등록한것 읽어 올수도 있음
    int sampleData;

    @Value("${my.value}") // ${} : 프로퍼티
    int myValue;

    // application.properties 에 my.value = 100 저장하면 읽어옴
    // 표현식 안에 프로퍼티를 넣는 것은 가능 (반대는 안됨)
    @Value("#{${my.value} eq 100}") // ${} : 프로퍼티
    boolean isMyValue;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("==============");
        System.out.println(value);
        System.out.println(greeting);
        System.out.println(trueOrFalse);

        System.out.println(sampleData);

        System.out.println(myValue);
        System.out.println(isMyValue);


        // SpEL 구성
        ExpressionParser parser = new SpelExpressionParser();
        Expression expression = parser.parseExpression("2 + 100");
        Integer value = expression.getValue(Integer.class);
        System.out.println(value);
    }
}

// 출력
==============
2
helloworld
true
200
100
true
102
```