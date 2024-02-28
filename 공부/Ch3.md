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






  
