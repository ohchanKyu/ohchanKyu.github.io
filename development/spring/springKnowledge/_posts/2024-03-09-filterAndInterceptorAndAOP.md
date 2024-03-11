---

layout: post
title:  "[Spring] Filter / Interceptor / AOP"
date:   2024-03-09 01:38:04 +0900

---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## Filter / Interceptor / AOP
Spring Project를 작성하다보면 공통적인 로직이 많이 나오는 경우가 많다.  
이러한 로직을 처리하기 위해 Filter, Interceptor, AOP 등을 사용한다.  

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
 
### Filter Method

### Interceptor


- ![Full-image](/assets/img/filterAndInterceptor/InterceptorContext.png){:.lead width="300" height="100" loading="lazy"}

### AOP

## Filter Code

## Interceptor Code

## AOP Code

출처 : https://mangkyu.tistory.com/173