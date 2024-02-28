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

**유료강의인점을 고려하여 컨틀롤러 코드들 생략**
* MemberFormControllerV1 - 회원 등록 컨트롤러
* MemberSaveControllerV1 - 회원 저장 컨트롤러
* MemberListControllerV1 - 회원 목록 컨틀롤러


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
