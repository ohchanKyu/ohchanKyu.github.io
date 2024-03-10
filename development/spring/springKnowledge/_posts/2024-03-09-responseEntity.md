---

layout: post
title:  "[Spring] REST API에서의 ReponseEntity 사용 이유"
date:   2024-03-07 01:38:04 +0900

---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


## Restful API의 등장
**Restful API**란 무엇일까?  
<span  style="background-color:#fff5b1">두 컴퓨터 시스템이 인터넷을 통해 정보를 안전하게 교환하기 위해 사용하는 인터페이스</span>
AWS에 나와있는 정의를 살펴보면 다음과 같이 정의된다. 즉 서로 다른 시스템에서 정보를 교환하고,  
데이터 통신을 지향하기 위한 규약이라고 말할 수 있다. 쉽게 말하면 ServiceA라는 시스템에서 어떠한 데이터를  
다른 ServiceB라는 시스템에게 전달하고자 할 때 어떠한 형태로 데이터를 보내줄지에 대한 규약이다.  

배경을 좀 더 살펴보면 PC이외에도 모바일, 태블릿 등 여러가지의 기기들이 나오면서 해당 스마트 기기마다  
서로 다른 서버를 만드는 것은 비효율적인 일이 되었다. 따라서 하나의 서버에서 데이터와 로직을 생성하고,  
Client Side에서만 해당 스마트 기기에 맞게 UI를 구현해주고 하나의 서버에서 필요한 로직과 데이터를  
받아오는 것이다. 이처럼 Server와 Client side를 분류하여 server에서는 로직과 데이터 제공에 집중하여  
규약에 맞게 제공하는 것이 Restful API이다.  
{:.note}

## Http Response
Http Reponse는 크게 **Status, Headers, Body** 3가지로 구성된다.  

### Status
status는 상태 코드로 200,201...400,401처럼 응답의 상태코드를 나타낸다. 

Response Status (참고 자료)
- 1xx : 전송 프로토콜 수준의 정보 교환
- 2xx : 클라이언트 요청이 성공적으로 수행됨
- 3xx : 클라이언트는 요청을 완료하기 위해 추가적인 행동을 취해야 함
- 4xx : 클라이언트의 잘못된 요청
- 5xx : 서버쪽 오류로 인한 상태 코드  

### Headers
Headers는 응답으로 부가적인 정보를 전송할 수 있도록 해줍니다.  
Header에는 주로 인증과 캐싱에 대한 정보를 담으며, 전송할 데이터(JSON,XML)과 함께 요구사항 정보를  
추가해야 한다면 Headers에 정보를 담아서 전송할 수 있도록 한다.  

### Body
Body부분이 Client측으로 전송할 데이터이다. JSON, XML, HTML등의 데이터를 포함한다.  

## @Controller + @ResponseBody / @RestController
Controller에서도 Client측으로 데이터 전송이 가능하다. 단 일반적인 @Controller 어노테이션은  
view(화면)을 return하기 때문에 **@ResponseBody**를 추가하여야 view를 return하는 것이 아닌  
직접 데이터를 return한다. @ResponseBody를 이용할 경우 Spring은 HTTP 응답에 return하는 데이터 값을  
자동으로 변경해준다. 따라서 Controller Class에서 각 메소드에 @ResponseBody를 붙여주면  
데이터를 전송하는 API를 만들 수 있다.  

**Spring4.0**에서는 **@RestController** 어노테이션을 선언해주면서 컨트롤러 클래스의 각 메서드마다  
@ResponseBody를 추가할 필요가 없어졌고, 모든 메서드는 @ResponseBody 애노테이션이 기본으로 작동이 된다.  

즉 **@RestController와 @Controller + 모든 메소드의 @ResponseBody**는 같은 것이다.  

~~~java
@Controller
public class MemberController {

    @ResponseBody
    @RequestMapping("/member")
    public String responseBodyTestMethod(){
        return "ResponseBody Test!";
    }
}

@RestController
public class MemberRestController {

      @GetMapping("/rest/member")
      public String restControllerTestMethod(){
          return "RestController Test!";
      }
}
~~~
- ![Full-image](/assets/img/responseEntity/responseBody.png){:.lead width="300" height="100" loading="lazy"}
- ![Full-image](/assets/img/responseEntity/restController.png){:.lead width="300" height="100" loading="lazy"}


## ResponseEntity

## ResponseEntity 사용 이유