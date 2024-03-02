# 스프링 MVC - 기본 기능


## 로킹 라이브러리

스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리`(spring-boot-starter-logging)`가 함께 포함한다.

* SLF4J
* Logback

**로그 선언**
* `private Logger log = LoggerFactory.getLogger(getClass());`
* `private static final Logger log = LoggerFactory.getLogger(Xxx.class)`
* `@Slf4j` : 롬복 사용 가능

**로그 호출**
* `log.info("hello")`
* `System.out.println("hello")`

**LogTestController**

```ruby
//@Slf4j
 @RestController
 public class LogTestController {

   private final Logger log = LoggerFactory.getLogger(getClass());

     @RequestMapping("/log-test")
     public String logTest() {
         String name = "Spring";
         log.trace("trace log={}", name);
         log.debug("debug log={}", name);
         log.info(" info log={}", name);
         log.warn(" warn log={}", name);
         log.error("error log={}", name);

//로그를 사용하지 않아도 a+b 계산 로직이 먼저 실행됨, 이런 방식으로 사용하면 X log.debug("String concat log=" + name);
return "ok";
} }
```

**매핑 정보**

* `@RestController`
   * `@Controller`는 반환 값이 `String`이면 뷰 이름으로 인식된다.그래서 **뷰를 찾고 뷰가 렌더링** 된다.
   * `@RestController`는 반환 값으로 뷰를 찾는 것이 아니라,**HTTP 메시지 바디에 바로 입력**한다.따라서 실행 결과로 ok 메시지를 받을 수 있다.`@Responsebody`와 관련있다.


**로그 레벨**


* LEVEL:TRACE > DEBUG > INFO > WARN > ERROR
* 개발 서버는 **debug** 출력
* 운영 서버는 **info** 출력

**로그 레벨 설정**

application.properties
```
#전체 로그 레벨 설정(기본 info) logging.level.root=info

#hello.springmvc 패키지와 그 하위 로그 레벨 설정 logging.level.hello.springmvc=debug
```

**올바른 로그 사용법**

* log.debug("data={}",data)
  * 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.
 

