# 스프링 MVC - 구조 이해

**직접 만든 MVC 프레임워크 구조**

![스크린샷 2024-03-02 오후 2 03 48](https://github.com/pbk2312/SpringMVC_1/assets/156402683/cb12995a-44b5-40ec-bc73-6c1f2459c44f)


**SpringMVC 구조**

![스크린샷 2024-03-02 오후 2 04 10](https://github.com/pbk2312/SpringMVC_1/assets/156402683/a997d1a7-2c03-47af-9ac5-bcd9ebdec1cb)


**직접 만든 프레임워크-> 스프링 MVC 비교**
* FrontController -> DispatcherServlet
* handlerMappingMap -> HandlerMapping
* MyHandlerAdapter -> HandlerAdapter
* ModelView -> ModelAndView
* viewResolver -> ViewResolver
* MyView -> View

### DispatcherServlet 구조 살펴보기

스프링 MVC도 프론트 컨트롤러 패턴으로 구현

### DispatcherSerlvet 서블릿 등록
* `DispatcherSerlvet`도 부모 클래스가 HttpServlet 상속 받아 사용,서블릿으로 동작
* 스프링 부트는 `DispatcherSerlvet`을 서블릿으로 자동으로 등록하면서 모든 경로(`urlPatterns="/")`에 대하여 매핑한다.

### 요청흐름
* 서블릿이 호출되면 `HttpServlet`이 제공하는 `service()`가 호출된다.
* 스프링 MVC는 `DispatcherServlet`의 부모인 `FrameworkServlet`에서 `service()`를 오버라이드 해두었다.
* `FrameworkServlet.service()`를 시작으로 여러 메서드가 호출되면서 `DispatcherServlet.doDispatch()`가 호출된다.


DispatcherServlet.doDispatch()
```ruby
protected void doDispatch(HttpServletRequest request, HttpServletResponse
 response) throws Exception {

     HttpServletRequest processedRequest = request;
     HandlerExecutionChain mappedHandler = null;
     ModelAndView mv = null;

// 1. 핸들러 조회
mappedHandler = getHandler(processedRequest);
if (mappedHandler == null) {
 noHandlerFound(processedRequest, response);
return; }


//2.핸들러 어댑터 조회-핸들러를 처리할 수 있는 어댑터
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

// 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환 mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
     processDispatchResult(processedRequest, response, mappedHandler, mv,
 dispatchException);
}

 private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView
 mv, Exception exception) throws Exception {
// 뷰 렌더링 호출
render(mv, request, response);
 }

 protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
     View view;
String viewName = mv.getViewName(); //6. 뷰 리졸버를 통해서 뷰 찾기,
     7.View 반환
     view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
     // 8. 뷰 렌더링
     view.render(mv.getModelInternal(), request, response);
 }

```

**SpringMVC 구조**


![스크린샷 2024-03-02 오후 2 04 10](https://github.com/pbk2312/SpringMVC_1/assets/156402683/a997d1a7-2c03-47af-9ac5-bcd9ebdec1cb)


**동작 순서**
1. 핸들러 조회:핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. 핸들러 어댑터 조회:핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행:핸들러 어댑터를 실행한다.
4. 핸들러 실행:핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 **변환**해서 반환한다.
6. viewResolver 호출:뷰 리볼버를 찾고 실행한다.
7. View 반환:뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고,렌더링 역할을 담당하는 뷰 객체를 반환한다.
8. 뷰 렌더링:뷰를 통해서 뷰를 렌더링 한다.


## 핸들러 매핑과 핸들러 어댑터

스프링 부트가 자동 등록하는 핸들러 매핑과 핸들러 어댑터

**HandlerMapping**


`
0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
`


`
1 = BeanNameUrlHandlerMapping:스프링 빈의 이름으로 핸들러를 찾는다
`

**HandlerAdapter**



`
0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
`


`
1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
`


`
2 = SimpleControllerHandlerAdapter :  Controller 인터페이스(애노테이션X, 과거에 사용) 처리
`


## 뷰 리졸버

스프링 부트는 `InternalResourceViewResolver` 라는 뷰 리졸버를 자동으로 등록하는데, 
이때 `application.properties` 에 등록한 `spring.mvc.view.prefix` , `spring.mvc.view.suffix` 설정 정보를 사용해서 등록한다.


**스프링 부트가 자동 등록하는 뷰 리졸버**

`
1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능 에 사용)
`



`
2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다. 
`

## 스프링 MVC


**@ReqeustMapping**


* `RequestMappingHandlerMapping`
* `RequestMappingHandlerAdapter`

**실무에서는 99.9% 이 방식의 컨트롤러를 사용**한다



**유료 강의인점을 고려해 코드 생략**

**SpringMemberFormControllerV1 - 회원 등록 폼**

* `@Controller`
  * 스프링이 자동으로 스프링 빈으로 등록한다(내부에 @Component 존재)
  * 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다.
* `@Reqeustmapping`:요청 정보를 매핑한다.해당 URL이 호출되면 이 메서드가 호출된다.애노테이션 기반으로 동작하기 때문에,메서드의 이름은 임의로 지으면 된다.
* `ModelAndView`:모델과 뷰 정보를 담아서 반환화면 된다.


**SpringMemberSaveControllerV1 - 회원 저장**

```ruby
   @RequestMapping("/springmvc/v1/members/save")
    private ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);


        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);
        return mv;

    }
```

* mv.addObject("member",member)
  * 스프링이 제공하는 `ModelAndView`를 통해 Model데이터를 추가할때는 `addObject()`를 사용하면 된다.이 데이터는 이후 뷰를 렌더링 할 때 사용된다.


**SpringMemberListControllerV1 - 회원 목록**



**V2는 생략**


## 스프링 MVC 실용적인 방식

**SpringMemberControllerV3**
```ruby
/**
* v3
* Model 도입
* ViewName 직접 반환
* @RequestParam 사용
* @RequestMapping -> @GetMapping, @PostMapping
* */
 @Controller
 @RequestMapping("/springmvc/v3/members")
 public class SpringMemberControllerV3 {

     private MemberRepository memberRepository = MemberRepository.getInstance();

     @GetMapping("/new-form")
     public String newForm() {
         return "new-form";
     }

     @PostMapping("/save")
     public String save(
             @RequestParam("username") String username,
             @RequestParam("age") int age,
             Model model) {
         Member member = new Member(username, age);
         memberRepository.save(member);
         model.addAttribute("member", member);
         return "save-result";
     }

     @GetMapping
     public String members(Model model) {
         List<Member> members = memberRepository.findAll();
         model.addAttribute("members", members);
         return "members";
}
 }
```

*  **Model 파라미터**
   *`save()`,`member()`를 보면 Model 파라미터로 받는 것을 확인할 수 있다.

*  **ViewName 직접 반환**

*  **RequestParam 사용**
   * 스프링은 HTTP 요청 파라미터를 `@RequestParam`으로 받을 수 있다.
   * `@ReqeustParam("username") 은 reqeust.getParameter("username")와 거의 같은 코드라 생각하면 된다.
   * 물론 GET 쿼리 파라미터,POST Form 방식을 모두 지원한다.
 
* **@ReqeustMapping -> @GetMapping,@PostMapping**
  * `*@ReqeustMapping`은 URL만 매칭하는 것이 아니라,HTTP Method도 함께 구분할 수 있다.
  * 예를 들어 URL이 `/new-form` 이고 HTTP Method가 GET인 경우를 모두 만족하는 매핑을 하려면 다음과 같이 처리하면 된다.
    ```ruby
    @ReqeustMapping(value = "/new-form" , method = RequestMethod.GET)

    
    ``` 
