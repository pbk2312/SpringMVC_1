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
