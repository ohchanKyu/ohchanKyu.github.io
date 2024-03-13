---

layout: post
title:  "[Spring] Filter / Interceptor 로그인 처리는 어디에서?"
date:   2024-03-09 01:38:04 +0900

---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## Filter / Interceptor
Spring Project를 작성하다보면 **공통적인 로직**이 많이 나오는 경우가 많다.  
이러한 로직을 처리하기 위해 Filter, Interceptor 등을 사용한다.  

### Filter
필터(Filter)는 디스패치 서블릿에게 요청이 전달되기 전/후에 URL 패턴에 맞는  
모든 요청에 대해 추가적인 로직을 처리할 수 있는 기능을 제공한다.  
디스패치 서블릿은 Spring Context에서 가장 앞쪽에 존재하는 front Controller이다.  
즉 우리가 흔히 생각하는 @Controller를 통해 만든 Controller는  
모두 디스패치 서블릿을 거쳐서 해당 로직을 수행하게 된다. 

- ![Full-image](/assets/img/filterAndInterceptor/filterContext.png){:.lead width="300" height="100" loading="lazy"}

위의 사진을 보면 더 쉽게 이해가 가능하다.  
Filter를 보면 Spring Context 밖에 존재하고 Web Context 안에 존재한다.  
즉 **Filter는 Spring Container가 아닌 Tomcat과 같은 WAS(Web Container)에 의해 관리**가 되는 것이다.  

따라서 Filter에 의한 전체적인 흐름을 보면 다음과 같다.  

1. 사용자가 URL로 Http 요청을 보낸다.  
2. Tomcat과 같은 WAS (Web Server)로 요청이 전달된다.  
3. Filter에서 공통 로직을 처리한다.  
4. Spring Context에 있는 디스패치 서블릿으로 요청이 전달된다.  
5. Spring Context에 있는 우리가 직접 만든 Controller로 요청이 전달되고 로직을 수행한다.  


### Filter의 사용
그렇다면 Filter는 언제 써야될까?  
필터는 주로 요청에 대한 XSS방어, 인코딩 변환처리, 요청에 대한 인증, 권한 체크 등에 사용된다.  
예를 들어 JWT 인증을 구현하였다면, Filter를 통해 헤더에 해당 유효한 토큰이 있는지 검사할 수 있다.  
이에 대표적인 예가 **Spring Security**인 것이다.  
위의 구조를 보면 Filter는 **Spring Context 밖에 존재하므로 Bean 객체를 사용하지 못한다.**   
하지만 **Filter Interface를 구현한 객체는 Bean으로 등록이 가능하다.**  

~~~java
// file: "Filter.java"
public interface Filter {
    public default void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    public default void destroy() {}
}
~~~

~~~java
// file: "LogFilter.java"
@Slf4j
public class LogFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("Log Filter Init!");
    }
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) servletRequest;

        String requestURL = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        log.info("REQUEST URL : [{}] [{}]",uuid,requestURL);
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {
        log.info("Log Filter Destroy!");
    }
}
~~~

~~~java
// file: "WebMvcConfiguration.java"
@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {

    @Bean
    public FilterRegistrationBean<Filter> logFilter(){
        FilterRegistrationBean<Filter> logFilter = new FilterRegistrationBean<Filter>();
        logFilter.setFilter(new LogFilter());
        logFilter.setOrder(1);
        return logFilter;
    }
}
~~~

### Interceptor

- ![Full-image](/assets/img/filterAndInterceptor/InterceptorContext.png){:.lead width="300" height="100" loading="lazy"}


### 차이점
공통로직을 처리하는 것은 동일하지만 전역 Spring ExceptionHandler로 예외를 처리하기 위해서는,  
Interceptor 구현이 편리하다.

출처 : https://mangkyu.tistory.com/173