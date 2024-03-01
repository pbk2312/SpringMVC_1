## 프론트 컨트롤러 패턴

프론트 컨트롤러 도입 전

![image (2)](https://github.com/pbk2312/SpringMVC_1/assets/156402683/8f79f330-b7c9-47a4-b4d5-b9959ced4b51)


프론트 컨트롤러 도입 후

![image (3)](https://github.com/pbk2312/SpringMVC_1/assets/156402683/7b30011b-e522-4024-aa45-983d9fdb4ec1)


* 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음!!
* 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
* 입구를 하나로!!
* 공통 처리 가능
* 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨

### 스프링 웹 MVC의 DispatcherServlet이 FrontController 패턴으로 구현되어 있음


V1 구조

![image](https://github.com/pbk2312/SpringMVC_1/assets/156402683/daf2c1ea-eb86-4921-b149-83c627638c2f)


Controller V1

```ruby
public interface ControllerV1 {

    // 각 컨트롤러들은 이 인터페이스를 구현하면 된다
    void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```
서블릿과 비슷한 모양의 컨트롤러 인터페이스를 도입한다.각 컨틀롤러들은 이 인터페이스를 구현하면 된다.프론트 컨트롤러는 이 인터페이스를 호출해서 구현과 관계없이 로직의 일관성을 가져살 수 있다.

**유료강의인점을 고려하여 컨트롤러 코드들 생략**
* MemberFormControllerV1 - 회원 등록 컨트롤러
* MemberSaveControllerV1 - 회원 저장 컨트롤러
* MemberListControllerV1 - 회원 목록 컨트롤러


## FrontControllerServletV1 - 프론트 컨트롤러
```ruby
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {


    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveController());
        controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
    }

    Map<String, ControllerV1> controllerMap = new HashMap<>();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        System.out.println("FrontControllerServletV1.service");

        String requestURI = request.getRequestURI();

        ControllerV1 controller = controllerMap.get(requestURI);

        if (controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        controller.process(request,response); // new-form,save,members 에 있는 메서드 실행

    }
}
```

**urlPatterns**
* `urlPatterns = "/front-controller/v1/*"` : `/front-controller/v1` 를 포함한 하위 모든 요청
은 이 서블릿에서 받아들인다.
* 예) `/front-controller/v1` , `/front-controller/v1/a` , `/front-controller/v1/a/b`
  
**service()**
* 먼저 `requestURI`를 조회해서 실제 호출할 컨트롤러를 `controllerMap`에서 찾는다.만약 없다면 `404(SC_NOT_FOUND)` 상태 코드를 반환한다.컨트롤라를 찾고 `controller.process(request,response)`를 호출해서 해당 컨틀롤러를 실행한다.

## View 분리 - V2

모든 컨트롤러(new-form,save,members)에서 뷰로 이동하는 부분에 중복이 있고,깔끔하지 않다.
```ruby
 String viewPath = "/WEB-INF/views/new-form.jsp";
 RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
 dispatcher.forward(request, response);
```
이 부분을 깔끔하게 분리하기 위해 별도로 뷰를 처리하는 객체를 만들자

## V2 구조

![image (1)](https://github.com/pbk2312/SpringMVC_1/assets/156402683/ededf695-e6b0-44e7-9c2b-540acc94b0e3)


MyView

```ruby
  public class MyView {
     private String viewPath;
     public MyView(String viewPath) {
         this.viewPath = viewPath;
     }
     public void render(HttpServletRequest request, HttpServletResponse response)
 throws ServletException, IOException {
         RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
         dispatcher.forward(request, response);
     }
}
```

ControllerV2
```ruby
 public interface ControllerV2 {
     MyView process(HttpServletRequest request, HttpServletResponse response)
 throws ServletException, IOException;
 }
```


**유료 강의인점을 고려하여 코드 생략**

* MemberFormControllerV2 - 회원 등록 폼 (MyView 반환)
```ruby
  return new MyView("/WEB-INF/views/new-form.jsp");
```


* MemberSaveControllerV2 - 회원 저장(MyView 반환)
```ruby
  return new MyView("/WEB-INF/views/save-result.jsp");
```

* MemberListControllerV2 - 회원 목록(Myview)반환
```ruby
  return new MyView("/WEB-INF/views/members.jsp");
```

이제 각 컨트롤러는 복잡한 dispatcher.forward() 직접 생성 X


프론트 컨트롤러 V2 
```ruby
//WEB-INF/views/new-form.jsp
@WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {

    Map<String, ControllerV2> controllerMap = new HashMap<>();

    public FrontControllerServletV2() {
        //WEB-INF/views/new-form.jsp
        controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
        controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
    }



    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {


        //WEB-INF/views/new-form.jsp
        String requestURI = request.getRequestURI();

        ControllerV2 controller = controllerMap.get(requestURI);

        if (controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyView view = controller.process(request, response);

        view.render(request,response);

    }
}

```

ControllerV2 반환 타입이 `MyView` 객체의 `render()`를 호출하는 부분을 모두 일관되게 처리할 수 있다.각각의 컨트롤러는 `MyView` 객체를 생성해서 반환하면 된다.


## Model 추가 - V3

### 서블릿 종속성 제거
### 뷰 이름 중복 제거  
* /WEB-INF/views/new-form.jsp -> new-form
* /WEB-INF/views/save-result.jsp -> save-result
* /WEB-INF/views/members.jsp -> members


![스크린샷 2024-03-01 오후 3 36 22](https://github.com/pbk2312/SpringMVC_1/assets/156402683/ea7f8987-1e9a-4899-b273-9ba33ed26af5)



지금까지 컨트롤러에서 서블릿에 종속적인 HttpServletRequest를 사용했다.그리고 Model도 request.setAttribute()를 통해 데이터를 저장하고 뷰에 전달했다.
서블릿 종속성을 제거하기 위해 Model을 직접 만들고,추가로 View 이름까지 전달하는 객체를 만들어 본다.


ModelView

```ruby
@Getter @Setter
public class ModelView {

    private String viewName;
    private Map<String , Object> model = new HashMap<>();

    public ModelView(String viewName) {
        this.viewName = viewName;
    }

}
```
뷰의 이름과 뷰를 랜더링할 때 필요한 model 객체를 가지고 있다.model은 단순히 map으로 되어 있으므로 컨트롤러에서 뷰에 필요한 데이터를 key,value로 넣어주면 된다.

ControllerV3

```ruby
public interface ControllerV3 {

    ModelView process(Map<String , String> paramMap);

}
```
**유료강의인점을 고려하여 컨트롤러 코드들 생략**

* MemberFormControllerV3 - 회원 등록 폼
```ruby
  return new ModelView("new-form");
```
* MemberSaveControllerV3 - 회원 저장
```ruby
 ModelView mv = new ModelView("save-result");
        mv.getModel().put("member", member);
        return mv;
```
* MemberListControllerV3 - 회원 목록
```ruby
        List<Member> members = memberRepository.findAll();
        ModelView mv = new ModelView("members");

        mv.getModel().put("members", members);

        return mv;
```


FrontControllerServletV3
```ruby
//WEB-INF/views/new-form.jsp
@WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {

    Map<String, ControllerV3> controllerMap = new HashMap<>();

    public FrontControllerServletV3() {
        //WEB-INF/views/new-form.jsp
        controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
        controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
        controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());
    }


    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {


        //WEB-INF/views/new-form.jsp
        String requestURI = request.getRequestURI();

        ControllerV3 controller = controllerMap.get(requestURI);

        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }


        // paramMap

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);


        String viewName = mv.getViewName();// 논리 이름 Ex) new-form

        // EX) /WEB-INF/views/new-form.jsp
        MyView view = viewResolver(viewName);

        view.render(mv.getModel(),request, response);


    }

    private static MyView viewResolver(String viewName) {
        MyView view = new MyView("/WEB-INF/views/" + viewName + ".jsp");
        return view;
    }

    private static Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}


```


MyView 메서드 추가
  
  ```ruby
  public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws IOException,ServletException{
  
        modelToRequestAttribute(model, request); // 모델에 있는 값을 다 꺼내고 setAttribute 로 다 넣음
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);

    }

    private static void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        model.forEach((key, value) -> request.setAttribute(key,value));
    }
```


## 뷰 리졸버
`My view view = viewResolver(viewName)`


컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경,그리고 실제 물리 경로가 있는 MyView 객체를 반환
* 논리 뷰 이름:members
* 물리 뷰 경로:/WEB-INF/views/member.jsp

`view.render(mv.getModel(),request,response)`


* 뷰 객체를 통해서 HTML 화면 렌더링
* 뷰 객체의 render()는 모델 정보도 함께 받는다
* JSP는 request.getAttribute()로 데이터를 조회하기 때문애,모델의 데이터를 꺼내서 request.setAttribute()로 담는다
* JSP는 포워드해서 JSP를 렌더링 한다.


## 단순하고 실용적인 컨트롤러 - V4

![스크린샷 2024-03-01 오후 4 18 50](https://github.com/pbk2312/SpringMVC_1/assets/156402683/bb696b99-7af6-4bc5-a1fe-326eac1fe68d)

* 기본적인 구조는 V3와 같다.대신에 컨트롤러가 `ModelView`를 반환하지 않고,`ViewName`만 반환


ControllerV4

```ruby

public interface ControllerV4 {

    /**
     *
     * @param paramMap
     * @param model
     * @return
     */


    String process(Map<String , String> paramMap ,Map<String , Object> model);


}
```
이번 버전은 인터페이스에 ModelView가 없다.model 객체는 파라미터로 전달되기 때문에 그냥 사용하면 되고,결과로 뷰의 이름만 반환해주면 된다.


**유료강의인점을 고려하여 컨트롤러 코드들 생략**

* MemberFormControllerV4
```ruby
return "new-form";
```

* MemberSaveControllerV4
```ruby
  List<Member> members = memberRepository.findAll();

        model.put("members",members);
        return "members";
```

* MemberListControllerV4
```ruby
    String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.put("member", member);

        return "save-result";

```


FrontControllerV4
```ruby
//WEB-INF/views/new-form.jsp
@WebServlet(name = "frontControllerServletV4", urlPatterns = "/front-controller/v4/*")
public class FrontControllerServletV4 extends HttpServlet {

    Map<String, ControllerV4> controllerMap = new HashMap<>();

    public FrontControllerServletV4() {
        //WEB-INF/views/new-form.jsp
        controllerMap.put("/front-controller/v4/members/new-form", new MemberFormControllerV4());
        controllerMap.put("/front-controller/v4/members/save", new MemberSaveControllerV4());
        controllerMap.put("/front-controller/v4/members", new MemberListControllerV4());
    }


    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {


        //WEB-INF/views/new-form.jsp
        String requestURI = request.getRequestURI();

        ControllerV4 controller = controllerMap.get(requestURI);

        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }


        // paramMap

        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>(); // 추가
        String viewName = controller.process(paramMap, model);

        // EX) /WEB-INF/views/new-form.jsp
        MyView view = viewResolver(viewName);

        view.render(model, request, response);


    }

    private static MyView viewResolver(String viewName) {
        MyView view = new MyView("/WEB-INF/views/" + viewName + ".jsp");
        return view;
    }

    private static Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```


**모델 객체 전달**
```ruby
  Map<String, Object> model = new HashMap<>(); 
```

모델 객체를 프론트 컨트롤러에서 생성해서 넘겨준다.컨트롤러에서 모델 객체에 값을 담으면 여기에 그대로 담겨있게 된다.

뷰의 논리 이름을 직접 반환
```ruby
String viewName = controller.process(paramMap, model);

        // EX) /WEB-INF/views/new-form.jsp
        MyView view = viewResolver(viewName);

```

컨트롤러가 직접 뷰의 논리이름을 반환하므로 이 값을 사용해서 실제 물리 뷰를 찾을 수 있다


## 유연한 컨트롤러1 - V5 


만약 어떤 개발자는 `ControllerV3`방식으로 개발하고 싶고,어떤 개발자는 `ControllerV4` 방식으로 개발하고 싶다면 어떻게 해야할까?
```ruby
public interface ControllerV3 {
    ModelView process(Map<String , String> paramMap);
}

```
```ruby
public interface ControllerV4 {
    String process(Map<String , String> paramMap ,Map<String , Object> model);
}
```

## 어댑터 패턴
두개의 컨트롤러는 완전히 다른 인터페이스다.따라서 호환이 불가능하다.마치 V3는 110V,V4는 220V 전기 콘센트 같은 것이다.이럴 때 사용하는 것이 어댑터이다.

![스크린샷 2024-03-01 오후 4 39 34](https://github.com/pbk2312/SpringMVC_1/assets/156402683/2f1aa236-446c-4bf9-a52b-f05308729dec)

* 핸들러 어댑터:중간에 어댑터 역할을 하는 어댑터가 추가되었는데 이름이 핸들러 어댑터이다.여기서 어댑터 역할을 해주는 덕분에 다양한 컨트롤러를 호출할 수 있다.
* 핸들러:컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경했다.그 이유는 이제 어댑터가 있기 때문에 꼭 컨트롤러의 개념 뿐만 아니라 어떤 것이든 해당 종류의 어댑터만 있으면 다 처리할 수 있기 때문이다.


어댑터는 이렇게 구현해야 한다는 어댑터용 인터페이스이다

MyHandlerAdapter

```ruby
public interface MyHandlerAdapter {

    boolean supports(Object handler); // 처리가능하면 true , 불가능 이면 false

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws ServletException, IOException;

}

```

* boolean supports(Object handler)
  * handler는 컨트롤러를 말한다.
  * 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드다
*  ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws ServletException, IOException;
  * 어댑터는 실제 컨트롤러를 호출하고,그 결과로 ModelView를 반환해야한다
  * 실제 컨트롤러가 ModelView를 반환하지 못하면 , 어댑터가 ModelView를 직접 생성해서라도 반환해야한다
  * 이전에는 프론트 컨트롤러가 실제 컨트롤러를 호출했지만 이제는 어댑터를 통해서 실제 컨트롤러가 호출된다.


실제 어댑터를 구현
먼저 ControllerV3을 지원하는 어댑터 구현


```ruby
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {


    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV3); // ControllerV3 인스턴스야?
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        // MemberFormControllerV3
        ControllerV3 controller = (ControllerV3) handler;


        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        return mv;


    }


    private static Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}

```
