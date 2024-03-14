---

layout: post
title:  "[Spring] Filter / Interceptor 세분화된 차이점"
date:   2024-03-09 01:38:04 +0900

---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## Filter / Interceptor
Spring Project를 작성하다보면 **공통적인 로직**이 많이 나오는 경우가 많다.  
중복되는 코드를 반복하는 것보다 하나의 로직으로 묶어서 처리하는 방법이 유지보수에도 유리하다.  
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

~~~java
public interface Filter {
    public default void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    public default void destroy() {}
}
~~~
Filter Interface는 다음과 같다.  
- init() : 필터 가 생성될 때 수행되는 메소드  
- doFilter() : Request, Response가 필터를 거칠 때 수행되는 메소드  
- destroy() : 필터가 소멸될 때 수행되는 메소드  

### Filter의 사용

그렇다면 Filter는 언제 써야될까?  
- XSS방어  
- 인코딩 변환처리  
- 요청에 대한 인증 및 권한 체크  

Filter는 주로 위와 같은 구현 요소가 있을 때 사용한다.  
예를 들어 JWT 인증을 구현하였다면, Filter를 통해 헤더에 해당 유효한 토큰이 있는지 검사할 수 있다.  
이에 대표적인 예가 **Spring Security**인 것이다.  

Filter는 **Spring Context 밖에 존재하므로 Bean 객체를 사용하지 못한다.**라는 설명이 많다.  
하지만 Filter 역시 **Bean으로 등록가능하며 다른 Bean 객체로 주입이 가능**하다.  
이에 대한 자세한 내용은 아래 포스팅을 참고하자.  
[Spring-Filter-Bean] - Filter에서 Bean 객체 사용이 가능한 이유


### Interceptor

- ![Full-image](/assets/img/filterAndInterceptor/InterceptorContext.png){:.lead width="300" height="100" loading="lazy"}

인터셉터(Interceptor)는 Spring이 제공하는 기술로써,  
디스패처 서블릿이 **컨트롤러를 호출하기 전/후에 요청과 응답을 참조 혹은 가공**할 수 있다.  
즉, 웹 컨테이너(WAS)에서 동작하는 필터와 달리 **인터셉터는 Spring Context에서 동작**한다.  

인터셉터는 여러개를 등록할 수 있으며, 등록된 순으로 순차적으로 실행시킨다.  
인터셉터 Class에서 preHandle()을 구현하였을 때 true를 return 시키면 Controller 로직을 수행하고,  
false를 return 한다면 Controller의 로직을 실행하지 않고 요청이 중단되는 것이다.  

~~~java
public interface HandlerInterceptor {
	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		return true;
	}

	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}
}
~~~
HandlerInterceptor Interface는 다음과 같다.  
- preHandler: 컨트롤러 호출 전에 호출되며 반환 타입은 Boolean 이다.  
- postHandler: 컨트롤러 호출 후 ModelAndView를 반환한 뒤에 호출된다.    
- afterCompletion: 뷰가 렌더링 된 후에 호출된다.  

아래 그림을 통해 이해를 높일 수 있다.  
- ![Full-image](/assets/img/filterAndInterceptor/interceptorDetail.png){:.lead width="300" height="100" loading="lazy"}

Interceptor는 위처럼 컨트롤러의 호출전, 호출 후, 요청 완료 이후 3가지나 세분화되어 호출된다.  
이러한 세 메서드의 호출은 위의 그림과 같은 흐름대로 호출되는 것이다.  

### Interceptor의 사용
인터셉터는 언제 사용하는 것이 좋을까?  
- 세부적인 보안 및 인증/인가 공통 작업  
- API 호출에 대한 로깅 또는 검사  
- Controller로 넘겨주는 데이터의 가공  

Intercpetor는 주로 위와 같은 구현 요소가 있을 때 사용한다.  
하지만 의문점이 들 수 있다. 지금까지 본 차이점에 따르면  
Interceptor는 Spring Context 작동하다 보니 Filter보다 좀 더 정교한 로직을 수행하거나,  
혹은 3개의 호출시점에 로직을 추가로 작성할 수 있어 더 유연하다는 점 밖에 보이지 않는다.  

그리고 사용 예시를 보면 Filter에서 구현한 것은 Interceptor에서도 구현가능해 보인다.  
따라서 **Filter와 Interceptor에서만 가능한 구현**을 정확하게 이해할 필요가 있다.  


## 세분화된 차이점
공통로직을 처리한다는 점은 Filter와 Interceptor와 같다.  
하지만 Filter와 Interceptor의 차이점이 존재한다.  
Filter는 단순히 디스패치 서블릿 호출 전 doFilter 하나만 호출되어 사용되는데,  


## 관련 포스트
- [Web-Context-Filter]
- [Spring-Filter-Bean]


출처 : https://mangkyu.tistory.com/173
    https://mangkyu.tistory.com/221
https://parkmuhyeun.github.io/woowacourse/2023-05-05-Filter-Interceptor/
[Web-Context-Filter]: ../../springDevelopment/_posts/2024-02-26-Filter.md
[Spring-Filter-Bean]: ./2024-03-13-FilterBean.md