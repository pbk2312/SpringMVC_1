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




**로그 사용시 장점**

* 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
* 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
* 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다. 특히 파 일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
* 성능도 일반 System.out보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한다.


**PathVariable(경로 변수)사용**
```ruby
/**
* PathVariable 사용
* 변수명이 같으면 생략 가능
* @PathVariable("userId") String userId -> @PathVariable String userId */
 @GetMapping("/mapping/{userId}")
 public String mappingPath(@PathVariable("userId") String data) {
     log.info("mappingPath userId={}", data);
     return "ok";
 }
```
* http://localhost:8080/mapping/userA

최근 HTTP API는 다음과 같이 리소스 경로에 식별자를 넣는 스타일을 선호한다.

* /mapping/userA
* /users/1
* `@RequestMapping` 은 URL 경로를 템플릿화 할 수 있는데, `@PathVariable` 을 사용하면 매칭 되는 부분을 편리하게 조회할 수 있다.
* `@PathVariable` 의 이름과 파라미터 이름이 같으면 생략할 수 있다.

**PathVariable(경로 변수)사용 - 다중**
```ruby
/**
* PathVariable 사용 다중 */

 @GetMapping("/mapping/users/{userId}/orders/{orderId}")
 public String mappingPath(@PathVariable String userId, @PathVariable Long
 orderId) {
     log.info("mappingPath userId={}, orderId={}", userId, orderId);
return "ok";
}
```

* http://localhost:8080/mapping/users/userA/orders/100


### 요청 매핑 - API 예시

**회원 관리 API**

* 회원 목록 조회: GET `/users`
* 회원 등록: POST `/users`
* 회원 조회: GET `/users/{userId}`
* 회원 수정: PATCH `/users/{userID}`
* 회원 삭제: DELETE `/users/{userId}`

**MappingClassController**

```ruby

@RestController
@RequestMapping("/mapping/users")
public class MappingClassController {


    @GetMapping
    public String users() {
        return "get users";
    }


    @PostMapping
    public String adduser() {
        return "post user";
    }

    @GetMapping("/{userId}")
    public String findUser(@PathVariable String userId) {
        return "get userId=" + userId;
    }

    @PatchMapping("/{userId}")
    public String updateUser(@PathVariable String userId) {
        return "update userId=" + userId;
    }

    @DeleteMapping("/{userId}")
    public String deleteUser(@PathVariable String userId) {
        return "delete userId=" + userId;
    }


}

```



**HTTP 요청 - 기본,헤더 조회**



**ReqeustHeaderController**

```ruby
@Slf4j
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod,
                          Locale locale,
                          @RequestHeader MultiValueMap<String, String> headerMap,
                          @RequestHeader("host") String host,
                          @CookieValue(value = "myCookie", required = false) String cookie
    ) {

        log.info("request={}", request);
        log.info("response={}", response);
        log.info("httpMethod={}", httpMethod);
        log.info("locale={}", locale);
        log.info("headerMap={}", headerMap);
        log.info("header host={}", host);
        log.info("myCookie={}", cookie);

        return "ok";


    }

}

```
* `HttpServletRequest`
* `HttpServletResponse`
* `HttpMethod` : HTTP 메서드를 조회한다. `org.springframework.http.HttpMethod`
* `Locale` : Locale 정보를 조회한다.
* `@RequestHeader MultiValueMap<String, String> headerMap`
  * 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
* `@RequestHeader("host") String host`
  * 특정 HTTP 헤더를 조회한다.
  * 속성
    * 필수 값 여부: `required`
    * 기본 값 속성: `defaultValue`
* `@CookieValue(value = "myCookie", required = false) String cookie`
  * 특정 쿠키를 조회한다.
  * 속성
    * 필수 값 여부: `required`
    * 기본 값: `defaultValue`
   

**MultiValueMap**

* MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
* HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
  * **keyA=value1&keyA=value2**
 
```ruby
 MultiValueMap<String, String> map = new LinkedMultiValueMap();
 map.add("keyA", "value1");
 map.add("keyA", "value2");
 //[value1,value2]
List<String> values = map.get("keyA");
``` 

**@Slf4j**
다음 코드를 자동으로 생성해서 로그를 선언해준다. 개발자는 편리하게 `log` 라고 사용하면 된다.

```ruby
 private static final org.slf4j.Logger log =
 org.slf4j.LoggerFactory.getLogger(RequestHeaderController.class);
```

### HTTP 요청 파라미터 - @RequestParam



**requestParamV1**
```ruby
/**
* 반환 타입이 없으면서 이렇게 응답에 값을 직접 집어넣으면, view 조회X */
* 
     @RequestMapping("/request-param-v1")
     public void requestParamV1(HttpServletRequest request, HttpServletResponse
 response) throws IOException {
         String username = request.getParameter("username");
         int age = Integer.parseInt(request.getParameter("age"));
         log.info("username={}, age={}", username, age);
         response.getWriter().write("ok");
     }
```

**request.getParameter()**
여기서는 단순히 HttpServletRequest가 제공하는 방식으로 요청 파라미터를 조회했다.


**requestParamV2**
```ruby
 @ResponseBody
 @RequestMapping("/request-param-v2")
 public String requestParamV2(
         @RequestParam("username") String memberName,
         @RequestParam("age") int memberAge) {
     log.info("username={}, age={}", memberName, memberAge);
     return "ok";
 }
```

* `@RequestParam` : 파라미터 이름으로 바인딩
* `@ResponseBody` : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력

* **@RequestParam**의 `name(value)` 속성이 파라미터 이름으로 사용
  * @RequestParam("**username**") String **memberName**
  * -> request.getParameter("**username**")
 


**requestParamV3**(이거 선호)

```ruby
**
* @RequestParam 사용
* HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능 */

 @ResponseBody
 @RequestMapping("/request-param-v3")
 public String requestParamV3(
         @RequestParam String username,
         @RequestParam int age) {
     log.info("username={}, age={}", username, age);
     return "ok";
}
```
HTTP 파라미터 이름이 변수 이름과 같으면 `@RequestParam(name="xx")` 생략 가능


**requestParamV4**
```ruby

/**
* @RequestParam 사용
* String, int 등의 단순 타입이면 @RequestParam 도 생략 가능 */
 @ResponseBody
 @RequestMapping("/request-param-v4")
 public String requestParamV4(String username, int age) {
     log.info("username={}, age={}", username, age);
     return "ok";
 }

```
`String` , `int` , `Integer` 등의 단순 타입이면 `@RequestParam` 도 생략 가능

너무 생략하면 과하다!!


**파라미터 필수 여부 - requestParamRequired**
```ruby
 /**
  * @RequestParam.required
* /request-param-required -> username이 없으므로 예외 *
* 주의!
* /request-param-required?username= -> 빈문자로 통과 *
* 주의!
* /request-param-required
* int age -> null을 int에 입력하는 것은 불가능, 따라서 Integer 변경해야 함(또는 다음에 나오는
defaultValue 사용) */

 @ResponseBody
 @RequestMapping("/request-param-required")
 public String requestParamRequired(
         @RequestParam(required = true) String username,
         @RequestParam(required = false) Integer age) {
     log.info("username={}, age={}", username, age);
return "ok";

}
```

* @RequestParam.required`
  * 파라미터 필수 여부
  * 기본값이 파라미터 필수( `true` )이다.

**기본 값 적용 - requestParamDefault**

```ruby
/**
  * @RequestParam
* - defaultValue 사용 *
* 참고: defaultValue는 빈 문자의 경우에도 적용 * /request-param-default?username=
*/

 @ResponseBody
 @RequestMapping("/request-param-default")
 public String requestParamDefault(
         @RequestParam(required = true, defaultValue = "guest") String username,
         @RequestParam(required = false, defaultValue = "-1") int age) {
     log.info("username={}, age={}", username, age);
     return "ok";
 }
```

**파라미터를 Map으로 조회하기 - requestParamMap** 

```ruby
 /**
  * @RequestParam Map, MultiValueMap
  * Map(key=value)
  * MultiValueMap(key=[value1, value2, ...]) ex) (key=userIds, value=[id1, id2])
  */
 @ResponseBody
 @RequestMapping("/request-param-map")
 public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
     log.info("username={}, age={}", paramMap.get("username"),
 paramMap.get("age"));
     return "ok";
 }
```

파라미터의 값이 1개가 확실하다면 `Map` 을 사용해도 되지만, 그렇지 않다면 `MultiValueMap` 을 사용하자.



## HTTP 요청 파라미터 - @ModelAttribute

**@ModelAttribute는** 

```ruby
@RequestParam String username;
 @RequestParam int age;
 HelloData data = new HelloData();
 data.setUsername(username);
 data.setAge(age);

```

**생략해줌**



* 롬복 `@Data`
  * `@Getter` , `@Setter` , `@ToString` , `@EqualsAndHashCode` , `@RequiredArgsConstructor` 를
자동으로 적용해준다.  


**@ModelAttribute 적용 - modelAttributeV1**

```ruby
/**
* @ModelAttribute 사용
* 참고: model.addAttribute(helloData) 코드도 함께 자동 적용됨, 뒤에 model을 설명할 때 자세히
설명
  */

 @ResponseBody
 @RequestMapping("/model-attribute-v1")
 public String modelAttributeV1(@ModelAttribute HelloData helloData) {
     log.info("username={}, age={}", helloData.getUsername(),
 helloData.getAge());
     return "ok";
 }
```
