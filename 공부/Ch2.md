# 서블릿

## 스프링 부트 서블릿 환경 구성
* @ServletComponentScan
  * 스프링 부트는 서블릿을 직접 등록해서 사용할 수 있도록 @ServletComponentScan을 지원한다.

```ruby
@ServletComponentScan // 하위 패키지를 다 뒤져서 서블릿을 다 찾고 서블릿 등록
@SpringBootApplication
public class ServletApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServletApplication.class, args);
    }
}

```
* @WebServlet:서블릿 애노테이션
  * name: 서블릿 이름
  * urlPatterns:URL 매핑
```ruby
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
```
* HTTP 요청을 통해 URL이 호출되면 서블릿 컨테이너는 다음 메서드를 호출한다
```ruby
protected void service(HttpServletRequest request, HttpServletResponse response)
```

## 서블릿 컨테이너 동작 방식

![119252128-76cd4100-bbe5-11eb-8ed5-3589ed62a43d](https://github.com/pbk2312/HTTP/assets/156402683/04bb7762-f789-4d5e-9c9f-a79520f69e27)
사진 출처:https://tecoble.techcourse.co.kr/post/2021-05-23-servlet-servletcontainer/

* http://localhost:8080/hello 가 요청되면 HTTP 요청메시지를 기반으로 request,response 객체가 생성되어 서블릿 컨테이너 안으로 던져진 후 helloServlet이 응답메시지를 반환한다.
* Response 객체 정보로 HTTP 응답을 생성한다.


## HttpServletRequest 역할
* 서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용활 수 있도록 개발자 대신에 HTTP 요청 메시지를 파싱한다.그리고 그 결과를 HttpServletRequest 객체에 담아서 제공한다.
*  HttpServletRequest를 사용하면 HTTP 요청 메시지를 편리하게 조회가 가능하다

### HttpServletReqeust 제공하는 기본 기능들 
```ruby
// StartLine
request.getMethod()); //GET
request.getProtocol()); // HTTP / 1.1
request.getScheme()); //http
request.getRequestURL()); // /request-header
request.getRequestURI()); //username=hi
request.isSecure()); //https 사용


//Header 모든 정보
   request.getHeaderNames().asIterator()
                .forEachRemaining(headerName -> System.out.println(headerName + ": "
                        + request.getHeader(headerName)));


// Header 편리한 조회
 request.getServerName()); //[Host 편의 조회]
 request.getLocales().asIterator()
                .forEachRemaining(locale -> System.out.println("locale = " + locale)); // [Accept-Language 편의 조회]

if (request.getCookies() != null) {
            for (Cookie cookie : request.getCookies()) {
                System.out.println(cookie.getName() + ": " + cookie.getValue());
            }
        } //[cookie 편의 조회]

 request.getContentType()// [Content 편의 조회]
 request.getContentLength())
 request.getCharacterEncoding()

```

**GET 방식은 getContentType()이 대부분 NULL**


## HTTP 요청 데이터 - 개요
**주로 3가지 많이 사용**
* GET - 쿼리 파라미터
  * 메시지 바디 없이,URL의 쿼리파라미터에 데이터를 포함해서 전달
* POST - HTML Form
  * 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=20
* HTTP message body에 데이터를 직접 담아서 요청
  * HTTP API에서 주로 사용,JSON,XML,TEXT
* 데이터 형식은 주로 JSON 사용
  * POST,PUT,PATCH
 
```ruby
request.getParameter("username"); // 단일 파라미터 조회
request.getParameterValues("username"); // 이름이 같은 복수 파리미터 조회
reqeust.getParameterNames(); // 파라미터 이름들 모두 조회
reqeust.getParameterMap(); // 파라미터를 Map으로 조회
```

* POST의 HTML Form을 전송하면 웹 브라우저는 다음 형식으로 HTTP 메시지를 만든다
  * 요청 URL: http://localhost:8080/request-param
  * content-type:application/x-www-form-urlencoded
  * message body:username=hello&age=20 


* **request.getParameter()는 GET URL 쿼리 파라미터 형식도 지원하고,POST HTML Form 형식도 둘다 지원**

> **GET URL 쿼리파라미터 형식**으로 클라이언트에서 서버로 데이터를 전달할때는 HTTP 메시지 바디를 사용하지 않기 때문에 content-type이 없다
> **POST HTML Form 형식**으로 데이터를 전달하면 HTTP 메시지 바디에 해당 데이터를 포함해서 보내기 때문에 바디에 포함한 데이터 형식인지 content-type을 꼭 지정해야한다.이렇게 폼으로 데이터를 전송하는 형식을  **application/x-www-form-urlencoded** 라 한다.

### HTTP 메시지 바디의 데이터를 InputStream을 사용해서 직접 읽기 가능
 
```ruby
 ServletInputStream inputStream = request.getInputStream();
 String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8); // 스트림 -> 스트링으로 변환
```

### JSON 형식으로 파싱
```ruby
 ServletInputStream inputStream = request.getInputStream();
 String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8); // 스트림 -> 스트링으로 변환
 HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
 helloData.getUsername();
 helloData.getAge();
```
* JSON 결과를 파싱해서 사용할 수 있는 자바 객체로 변환하려면 Jackson 라이브러리(ObjectMapper)를 사용
  

## HTTPServletResponse - 기본 사용법

```ruby
// [status=line]
   response.setStatus(HttpServletResponse.SC_OK); // http 200 응답코드


 // [response-headers]
   response.setHeader("Content-Type", "text/plain;charset=utf-8")
   response.setHeader("Cache-Control", "no-cache,no-store,must-revalidate"); // 캐시를 무효화 하겠다
   response.setHeader("Pragma", "no-cache"); // 과거 버전까지 캐시를 없앰
   response.setHeader("my-header", "hello");

// [Header 편의 메서드]
   PrintWriter writer = response.getWriter();
   writer.println("안녕하세요");

// Content 편의
   response.setHeader("Content-Type", "text/plain;charset=utf-8");
   response.setContentType("text/plain");
   response.setCharacterEncoding("utf-8"); //response.setContentLength(2); //(생략시 자동 생성)


// 쿠키
  Cookie cookie = new Cookie("myCookie", "good");
  cookie.setMaxAge(600); //600초
  response.addCookie(cookie);


// 리다이렉션
  response.sendRedirect("/basic/hello-form.html");
```
