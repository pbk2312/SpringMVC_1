# 템블릿 엔진으로... 
---- 

## 서블릿과 자바코드만으로 HTML 작성
```ruby
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("MemberSaveServlet.service");
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter w = response.getWriter();

        w.write("<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" + "</head>\n" +
                "<body>\n" +
                "성공\n" +
                "<ul>\n" +
                "    <li>id=" + member.getId() + "</li>\n" +
                "    <li>username=" + member.getUsername() + "</li>\n" +
                " <li>age=" + member.getAge() + "</li>\n" + "</ul>\n" +
                "<a href=\"/index.html\">메인</a>\n" + "</body>\n" +
                "</html>");


    }

```
**매우 보기 싫음**
> 자바 코드로 HTML을 만들어 내는 것보다 차라리 HTML 문서에 동적으로 변경해야 하는 부분만 자바 코드를 넣는 다면 더 편리
* 따라서 **템블릿 엔진(JSP,Thymeleaf,Freemarker,Velocity)등장**


### JSP 작성
```ruby
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```
첫줄은 JSP문서라는 뜻이다.JSP문서는 이렇게 시작해야 한다.
* JSP는 서버 내부에서 서블릿으로 변환된다
* JSP는 자바 코드를 그대로 다 사용 가능


import문
```ruby
`<%@ page import="hello.servlet.domain.member.MemberRepository" %>`
```

이 부분에는 자바 코드를 입력 가능
```ruby
`<% ~~ %>`
```
이 부분은 자바 코드 출력 하능
```ruby
<%= ~~ %>`

```

> 서블릿으로 개발할 때는 뷰(View)화면을 위한 HTML을 만드는 작업이 자바 코드에 섞여 지저분하고 복잡했다.하지만 JSP를 사용한 덕분에 어느정도 깔끔해졌다.하지만 JSP도 수백,수천줄이 넘어가면 정말 지옥같을 것이다.

# MVC 패턴의 등장
* 너무 많은 역할
* 변경의 라이프 사이클
  * 예) UI를 일부 수정하는 일과 비즈니스 로직을 수정하는 일은 각각 다르게 발생할 확률이 높고 대부분 서로에게 영향을 주지 않는다.이렇게 변경의 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수 하기 좋지 않다.(따라서 변경 주기가 다르면 분리해야한다) 
* 기능 특화
* Model View Controller
  * MVC 패턴은 컨트롤러(Controller),뷰(View),모델(Model)로 역할을 나눈것
   * 컨트롤러:HTTP 요청을 받아 파라미터를 검증하고,비즈니스 로직을 실행한다.그리고 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다
   * 모델:뷰에 출력할 데이터를 담아둔다.뷰에 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고,화면을 렌더링 하는 일에 집중가능
   * 뷰:모델에 담겨있는 데이터를 사용해서 화면에 그리는 일에 집중한다

 
> 컨트롤러에 비즈니스 로직을 둘 수도 있지만,이렇게 되면 컨트롤러 너무 많은 역할 부여 -> 일반적으로 비즈니스 로직은 서비스(Service)라는 계층을 별도로 만들어 처리
> 그리고 컨트롤러는 비즈니스 로직이 있는 서비스를 호출하는 역할 담당



MVC 패턴1


![image](https://github.com/pbk2312/SpringMVC_1/assets/156402683/fa78277d-23de-4cba-9a45-fe70375e1cae)

MVC 패턴2


![image (1)](https://github.com/pbk2312/SpringMVC_1/assets/156402683/8b9304b0-8bf4-4129-b4b9-2997a0c45219)


## MVC 패턴 적용

```ruby
 @Override
 protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);// 여기로 이동할거야
        dispatcher.forward(request,response);
    }
```

* dispatcher.forward(): 다른 서블릿이나 JSP로 이동할 수 있는기능.서버 내부에서 다시 호출이 발생한다.(리다이렉트랑 다름)
* /WEB-INF:이 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없다.우리가 기대하는 것은 항상 컨트롤러를 통해서 JSP를 호출하는 것이다.

## **redirect VS forward**
리다이렉트는 실제 클라이언트(웹 브라우저)에 응답이 나갔다가,클라이언트가 redirect 경로로 다시 요청한다.
따라서 클라이언트가 인지할 수 있고,URL 경로도 실제로 변경된다.반면에 포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못한다.

new-form.jsp


```ruby
<!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
<form action="save" method="post">
    username: <input type="text" name="username"/>
    age: <input type="text" name="age"/>
    <button type="submit">전송</button>
</form>
```
여기서 form의 action을 보면 절대 경로(/로 시작)가 아니라 상대경로(/로 시작X)인 것을 확인할 수 있다.이렇게 상대경로를 사용하면 폼 전송시 현재 URL이 속한 계층 경로 + save 가 호출된다

* 현재 계층 경로:/servlet-mvc/members
* 결과:/servlet-mvc/members/save


회원 저장 - 컨트롤러

```ruby
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        System.out.println("MemberSaveServlet.service");
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);


        memberRepository.save(member);

        // Model 에 데이터를 보관
        request.setAttribute("member",member);
        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response);

    }
```
request가 제공하는 setAttribute()를 사용하면 reqeust 객체에 데이터를 보관해서 뷰에 전달 가능
뷰는 request.getAttribute()로 데이터를 꺼냄

회원 저장 - 뷰


```ruby
<li>
    id=${member.id}
</li>
<li>
    username=${member.username}
</li>
<li>
    age=${member.age}
</li>
```

getAttribute()로 데이터를 꺼낼 수 있지만 JSP에서 제공하는 ${}문법으로 편리하게 조회 가능

*회원 목록-컨트롤러는 생략


회원 목록 조회-뷰
```ruby
    <tbody>
    <c:forEach var="item" items="${members}">
        <tr>
            <td>${item.id}</td>
            <td>${item.username}</td>
            <td>${item.age}</td>
        </tr>
    </c:forEach>

    </tbody>
```

모델에 담아둔 members를 JSP가 제공하는 taglib기능을 사용해서 반복하여 출력

<c:forEach> 기능을 사용하려면 다음과 같이 선언해야한다
```ruby
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
```


## MVC 패턴의 한계

컨트롤러를 보면 딱봐도 중복이 많고,필요하지 않는 코드들도 많이 보인다

### MVC 컨트롤러의 단점


#### **포워드 중복**
View로 이동하는 코드가 항상 중복 호출된다.

```ruby
 RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
 dispatcher.forward(request, response);
```

#### **ViewPath 중복**

```ruby
String viewPath = "/WEB-INF/views/new-form.jsp";
```
* prefix: /WEB-INF/views/
* suffix: .jsp

#### **사용하지 않는 코드** 
```ruby
 HttpServletRequest request, HttpServletResponse response
```
#### **공통 처리가 어렵다.**
기능이 복잡해질 수 록 컨트롤러에서 공통으로 처리해야 하는 부분이 점점 더 많이 증가할 것이다. 단순히 공통 기능을 메서드로 뽑으면 될 것 같지만, 결과적으로 해당 메서드를 항상 호출해야 하고, 실수로 호출하지 않으면 문제가 될 것이 다. 그리고 호출하는 것 자체도 중복이다.

#### **정리하면 공통 처리가 어렵다는 문제가 있다.**

* 이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다. 소위 **수문장 역할**을 하는 기능이 필요하다. **프론트 컨트롤러(Front Controller) 패턴**을 도입하면 이런 문제를 깔끔하게 해결할 수 있다.(입구를 하나로!)
스프링 MVC의 핵심도 바로 이 프론트 컨트롤러에 있다.









  
