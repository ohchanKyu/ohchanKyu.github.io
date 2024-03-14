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
- init() : 필터가 생성될 때 수행되는 메소드  
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

따라서 Interceptor에 의한 전체적인 흐름을 보면 다음과 같다.  

1. 사용자가 URL로 Http 요청을 보낸다.  
2. Tomcat과 같은 WAS (Web Server)로 요청이 전달된다.  
3. Filter에서 공통 로직을 처리한다.  
4. Spring Context에 있는 디스패치 서블릿으로 요청이 전달된다. 
5. **Spring Context에 존재하는 Interceptor를 거치게 된다. (2개 이상일 경우 등록 순으로 전달됨)**  
6. Spring Context에 있는 우리가 직접 만든 Controller로 요청이 전달되고 로직을 수행한다.  

Filter와 다르게 5번이 추가된 것으로 디스패치 서블릿의 요청을 Interceptor에서 가로챈다.  

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
- preHandle : 컨트롤러 호출 전에 호출되며 반환 타입은 Boolean 이다.  
- postHandle : 컨트롤러 호출 후 ModelAndView를 반환한 뒤에 호출된다.    
- afterCompletion : 뷰가 렌더링 된 후에 호출된다.  

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
Filter와 Interceptor의 차이점은 다음과 같다.  
- 예외 처리
- ServletRequest, ServletResponse 교체 
- View 렌더링의 제어

### 예외 처리
Filter와 Interceptor은 다른 Context에 위치하기 때문에 예외를 처리하는 부분이 다르다.  
Filter는 Web Context 영역에서 관리되므로 예외가 발생한다면 Spring Context의 도움을 받지 않는다.  
만약 JWT를 Filter를 통해 권한 토큰 검사를 할 경우 유요하지 않은 인증이라면  
상태코드 401과 함께 예외처리를 해야한다. 하지만 Filter에서 Exception을 던진다면,  
Web Context에 있는 Servlet으로 전달하기 때문에 Web Server(WAS)는 500 상태 코드로 응답한다.  

~~~java
@Override
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

	HttpServletRequest request = (HttpServletRequest) servletRequest;
	String token = jwtUtils.resolveToken(request);
	
	if (!jwtUtils.validateToken(token)){
		throw new CustomJwtException(token);
	}
}
~~~
위의 예시처럼 해당 코드가 있다고 하자.  
만약 요청 토큰이 유효하지 않다면 CustomJwtException을 예외로 던져준다.  
하지만 이 코드는 **CustomJwtException을 예외로 던지지 않고, 500을 응답코드**로 던져준다.  

~~~java
@Override
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
	log.info("ExceptionTestFilter do filter!");
	HttpServletResponse response = (HttpServletResponse) servletResponse;
	response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
	response.getWriter().write("InValid Authorized!");
}
~~~
- ![Full-image](/assets/img/filterAndInterceptor/exceptionTest1.png){:.lead width="300" height="100" loading="lazy"}

만약 Filter에서 예외처리를 하기 위해서는 위의 코드처럼  
servletResponse 응답 객체를 통해 상태코드를 설정하고 메시지를 설정해주어야 한다.  
이는 전역적으로 Exception을 처리하기 힘들고 응답 객체의 메소드를 통해  
Status, Header, Message 모두 설정해주어야 한다.  

하지만 Interceptor는 **Spring Context안에 있으므로 @ExceptionHandler**를 이용가능하다.  
이는 Spring에서 제공하는 어노테이션으로 전역적으로 예외를 처리할 수 있도록 해준다.  
또한 Spring Context안에 존재하므로 ResponseEntity 객체를 이용하여  
응답 상태를 쉽게 설정할 수 있다는 장점도 존재한다.  
그리고 예외가 Servlet의 ErrorController로 전달되지 않기 때문에,  
Spring Context 안에서 예외 처리를 하여 Client에게 전달이 가능하다.  

즉 **전역 예외처리를 위해서는 Interceptor가 유리**한 것이다.  

### ServletRequest, ServletResponse
Filter에서는 **ServletRequest와 ServletResponse를 교체**할 수 있다.  
이는 Request와 Response의 내부 상태를 변경하는 것이 아닌  
아예 다른 Request, Response 객체로 변경하는 것이다.  
이에 대한 예시로는 HttpServletRequest의 body를 로깅할때가 있다.  
**HttpServletRequest는 body의 내용을 한 번만 읽을 수 있다.**  

REST API를 구현 시, Filter에서 Json형태의 body를 데이터로 받아 모두 로깅할때가 있다.  
이때 Filter에서 body를 읽기 위해 getInputStream()을 사용하는데,  
InputStream은 한 번 읽으면 다시 읽을 수가 없다.  
따라서 Interceptor나 Filter에서 body를 읽게 된다면,  
Controller에서 body로 읽은 Json 데이터를 바인딩할 때 IO Exception이 발생하게 된다.  

~~~java
@Override
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
	 filterChain.doFilter(new CustomServletRequest(servletRequest),servletResponse);
}
~~~
따라서 위처럼 여러 번 InputStream을 열 수 있도록 Custom ServletRequest를 사용해야 한다.  

### View Rendering
Interceptor의 postHandle()에서 ModelAndView() 객체를 파라미터로 받을 수 있다.  
ModelAndView를 반환받고 PostHandler()이 호출되기 때문에 가능하다.  
따라서 View를 렌더링 하기 전에 추가 작업을 해줄 수 있다.  

예를 들어 Admin과 User가 있을 때, 관리자에게만 보여주어야 하는 정보가 있을 것이다.  
이러한 정보들을 ModelAndView 객체를 통하여 데이터를 변경할 수 있다.  

즉 Filter와 다르게 **요청 처리와 View 렌더링 사이에 무언가를 수행**하려는 경우  
HandlerInterceptor를 사용하면 제어를 쉽게 할 수 있다.  

## 관련 포스트
- [Web-Context-Filter]
- [Spring-Filter-Bean]


출처  
- https://mangkyu.tistory.com/173  
- https://parkmuhyeun.github.io/woowacourse/2023-05-05-Filter-Interceptor/  


[Web-Context-Filter]: ../../springDevelopment/_posts/2024-02-26-Filter.md
[Spring-Filter-Bean]: ./2024-03-13-FilterBean.md