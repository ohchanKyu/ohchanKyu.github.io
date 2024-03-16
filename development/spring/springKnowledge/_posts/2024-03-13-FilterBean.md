---

layout: post
title:  "[Spring] Filter에서 Bean 객체 사용이 가능한 이유"
date:   2024-03-09 01:38:04 +0900

---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## Filter의 개요  
Filter는 Web Context에서 관리되므로 Spring의 Bean으로 등록못한다는 설명이 많다.  
아래 그림은 Web Context와 Spring Context에 대한 이해를 높이는 그림이다.  

- ![Full-image](/assets/img/filterAndInterceptor/InterceptorContext.png){:.lead width="300" height="100" loading="lazy"}

해당 그림을 보면 Spring 컨테이너 보다 큰 서블릿 컨테이너의 Filter가 Spring에 의해  
관리되는 것은 이상할 수 있다. 하지만 **현재 Filter는 Spring Bean으로 등록 및 의존성 주입**도 가능하다.  
예전에 Filter는 실제로 Spring 컨테이너에서 관리되지 않았고, Bean의 의존성 주입도 불가능하였다.  
하지만 **DelegatingFilterProxy**의 등장으로 Bean 등록 및 의존성 주입도 가능해졌다.  

## DelegatingFilterProxy
필터(Filter)는 서블릿이 제공하는 기술이므로 서블릿 컨테이너에 의해 생성되며 서블릿 컨테이너에 등록이 된다.  
그렇기 때문에 스프링의 빈으로 등록도 불가능했으며, 빈을 주입받을 수 없었다.  
하지만 <span style="background-color:#fff5b1">필터에서도 DI와 같은 스프링 기술을 필요로 하는 상황이 발생하면서,</span>
스프링 개발자는 필터도 스프링 빈을 주입받을 수 있도록 대안을 마련하였는데, 그것이 바로 **DelegatingFilterProxy**이다.

Spring 1.2부터 DelegatingFilterProxy가 나오면서 서블릿 필터(Servlet Filter) 역시 스프링에서 관리가 가능하다.  
DelegatingFilterProxy는 서블릿 컨테이너에서 관리되는 프록시용 필터로써 우리가 만든 필터를 가지고 있다.  
우리가 만든 필터는 스프링 컨테이너의 빈으로 등록되는데, 요청이 오면 DelegatingFilterProxy가 요청을 받아서  
우리가 만든 필터(스프링 빈)에게 요청을 위임한다.  

이러한 동작 원리를 쉽게 정리하면 다음과 같다.  

1. Filter 구현체가 스프링 빈으로 등록    
2. ServletContext가 Filter 구현체를 갖는 DelegatingFilterProxy를 생성  
3. ServletContext가 DelegatingFilterProxy를 서블릿 컨테이너에 필터로 등록  
4. 요청이 오면 DelegatingFilterProxy가 필터 구현체에게 요청을 위임하여 필터 처리를 진행  

## Spring boot
SpringBoot라면 **DelegatingFilterProxy조차 필요가 없다.**
SpringBoot가 <span style="background-color:#fff5b1">내장 웹서버(Tomcat)를 지원하면서 서블릿 컨테이너까지 SpringBoot가 제어가능하기 때문이다. </span>
SpringBoot에서는 ServletContext에 필터(Filter) 빈을 DelegatingFilterProxy로 감싸서 등록하지 않아도 된다.     
SpringBoot가 서블릿 필터의 구현체 빈을 찾으면 DelegatingFilterProxy 없이 바로 필터 체인(Filter Chain)에  
필터를 등록해주기 때문이다.  

따라서 SpringBoot를 통해 Filter Class를 만들었다면 DelegatingFilterProxy로 감싸지 않고  
**Bean으로 등록만 해주면 된다.**    

## Bean 등록 및 DI Test 
~~~java
// file: 'XssFilter.java'
@Component
@Slf4j
public class XssFilter implements Filter {

   private final TestService testService;

   public XssFilter(TestService testService){
       this.testService = testService;
   }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("XSS filter init!");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        log.info("XSS filter do filter!");
        testService.TestServiceMethod();
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {
        log.info("XSS filter destroy!");
    }
}
~~~

~~~java
// file: 'TestService.java'
@Slf4j
@Service
public class TestService {

    public void TestServiceMethod(){
        log.info("Filter Bean Test!");
    }
}
~~~
위의 코드를 보면 XssFilter라는 Test용 Filter Class를 만들고, @Component를 통해 Bean 객체로 만들어준다.  
또한 DI(의존성 주입)를 확인하기 위해 @Service Annotation을 붙인 Bean을 필드 변수로 선언한 후,  
생성자 의존성 주입을 사용한다.  

- ![Full-image](/assets/img/filterAndInterceptor/filterBeanTest.png){:.lead width="300" height="100" loading="lazy"}
위의 사진을 통해 Test가 성공적인 것을 볼 수 있다.  
생성자 주입을 통해 TestService Bean 객체가 주입된것을 볼 수 있고  
doFilter() 메소드에서 해당 TestService의 메소드를 호출하였을 때 로그가 정상적으로 출력된다.  

따라서 SpringBoot를 사용함으로써  
**Filter도 Bean 객체로 등록가능하며, DI(의존성 주입)을 통해 다른 Bean 객체도 사용가능**함을 알 수 있다.

## 출처  
- https://mangkyu.tistory.com/221