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
Headers는 응답으로 부가적인 정보를 전송할 수 있도록 해준다.    
Header에는 주로 인증과 캐싱에 대한 정보를 담으며, 전송할 데이터(JSON,XML)과 함께 요구사항 정보를  
추가해야 한다면 Headers에 정보를 담아서 전송할 수 있도록 한다.  

### Body
Body부분이 Client측으로 전송할 데이터이다. JSON, XML, HTML등의 데이터를 포함한다.  

## Controller / RestController
Controller에서도 Client측으로 데이터 전송이 가능하다. 단 일반적인 @Controller 어노테이션은  
view(화면)을 return하기 때문에 **@ResponseBody**를 추가하여야 view를 return하는 것이 아닌  
직접 데이터를 return한다. @ResponseBody를 이용할 경우 Spring은 HTTP 응답에 return하는 데이터 값을  
자동으로 변경해준다. 따라서 Controller Class에서 각 메소드에 @ResponseBody를 붙여주면  
데이터를 전송하는 API를 만들 수 있다.  

**Spring4.0**부터는 **@RestController** 어노테이션을 선언해주면서 컨트롤러 클래스의 각 메서드마다  
@ResponseBody를 추가할 필요가 없어졌고, 모든 메서드는 @ResponseBody 애노테이션이 기본으로 작동이 된다.  

즉 **@RestController와 @Controller + 모든 메소드의 @ResponseBody**는 같은 것이다.  

### @Controller + @ResponseBody
~~~java
@Controller
public class MemberController {

    @ResponseBody
    @RequestMapping("/member")
    public String responseBodyTestMethod(){
        return "ResponseBody Test!";
    }
}
~~~
- ![Full-image](/assets/img/responseEntity/responseBody.png){:.lead width="300" height="100" loading="lazy"}

### @RestController
~~~java
@RestController
public class MemberRestController {

      @GetMapping("/rest/member")
      public String restControllerTestMethod(){
          return "RestController Test!";
      }
}
~~~
- ![Full-image](/assets/img/responseEntity/restController.png){:.lead width="300" height="100" loading="lazy"}

해당 코드는 @Controller + @ResponseBody와 @RestController를 통해 Java Object를 return해준다.  
PostMan으로 테스트 하였을 때 각 데이터인 Java String 객체가 return된다.  
위에서 설명한 것처럼 @ResponseBody를 이용할 경우 Spring은 Http 응답에  
응답 상태코드는 200, Header에 대한 정보는 기본적으로 5가지의 정보를 담아서 return해준다.  

즉 @ResponseBody + @Controller와  @RestController로 테스트한 결과를 정리하면 다음과 같다.  

- Status - 응답을 성공적으로 반환한다면 200
- Header - 5개의 정보
- Body - Java Object

### @ResponseStatus
~~~java
@ResponseBody
@ResponseStatus(HttpStatus.ACCEPTED)
@RequestMapping("/status/member")
public String responseBodyStatusTestMethod(){
    return "ResponseBody Status Test!";
}
~~~

- ![Full-image](/assets/img/responseEntity/ResponseStatus.png){:.lead width="300" height="100" loading="lazy"}
물론 @ResponseBody를 이용해서 응답 상태코드는 변환이 가능하다.  
위의 코드처럼 @ResponseStatus()를 이용하여 HttpStatus 객체를 통해 원하는 응답 상태코드를 설정할 수 있다.  
여기서는 ACCEPTED로 설정하여 아래에 응답이 200이 아닌 202로 설정된 것을 볼 수 있다.  

이처럼 상태코드 변환은 가능하지만 단점으로는 HTTP 규약 중  
하나인 **Header에 대해서는 유동적으로 설정하기 어렵다**는 것이다. 따라서 ResponseEntity를 사용한다.  

## ResponseEntity
**ResponseEntity는 HTTP 응답을 빠르게 만들어주기 위한 객체**이다.  
@ResponseBody는 어노테이션을 통해 해당 메소드가 view가 아닌 데이터를 return하지만,  
ResponseEntity는 객체로 사용된다. 즉 응답으로 보낼 Header, Status, Body를 모두 담은 요소를  
ResponseEntity 객체로 만들어서 반환하는 것이다. 이를 통해 Status 응답 코드와 Header를 보다  
유동적으로 객체의 구성요소로 포함시켜 데이터를 return할 수 있다.  

- ResponseEntity.ok()  
~~~java
@GetMapping("/rest/member")
public ResponseEntity<String> responseEntityMethodTest(){
    return ResponseEntity.ok().body("ResponseEntity Test!");
}

@GetMapping("/rest/member")
public ResponseEntity<String> responseEntityMethodTest(){
    return ResponseEntity.ok("ResponseEntity Test!");
}
~~~
- ![Full-image](/assets/img/responseEntity/responseEntity.png){:.lead width="300" height="100" loading="lazy"}
- ![Full-image](/assets/img/responseEntity/responseEntityBasicHeader.png){:.lead width="300" height="100" loading="lazy"}

위의 코드는 ResponseEntity 객체를 생성하여 Http 응답을 반환하는 코드이다.  
코드를 Test해보면 응답 상태코드는 200, Header 정보는 5가지로 되어있는 것을 볼 수 있다.  
또한 위의 2개의 코드는 모두 같은 결과를 return하며, header 혹은 응답 상태코드를 변경할 일이 없다면,  
두번째와 같이 코드를 작성하여도 된다.  

- Status 변경  
~~~java
@GetMapping("/rest/status/member")
public ResponseEntity<String> responseEntityStatusTest(){
    return ResponseEntity.status(HttpStatus.ACCEPTED).body("Status Test!");
}
~~~
- ![Full-image](/assets/img/responseEntity/responseEntityStatus.png){:.lead width="300" height="100" loading="lazy"}

만약 응답 상태코드를 변경하고 싶다면 builder 패턴을 이용하므로 ResponseEntity 객체의  
status를 호출하여 HttpStatus 객체를 이용하여 응답코드를 변경할 수 있다.  

- Header 추가  
~~~java
@GetMapping("/rest/header/member")
public ResponseEntity<String> responseEntityHeaderTest(){
    HttpHeaders headers = new HttpHeaders();
    headers.set("Type", "Header Test");
    return ResponseEntity.ok().headers(headers).body("Header Test!");
}
~~~
- ![Full-image](/assets/img/responseEntity/responseEntityHeader.png){:.lead width="300" height="100" loading="lazy"}
- ![Full-image](/assets/img/responseEntity/responseEntityModifyHeader.png){:.lead width="300" height="100" loading="lazy"}

만약 Header를 변경하고 싶다면 HttpHeaders 객체를 생성한 후 set() 메소드를 통해  
key와 value 값을 설정해줄 수 있다. 그 후 위와 마찬가지로 builder 패턴을 이용하므로,  
ResponseEntity 객체의 headers를 호출하여 HttpHeaders 객체를 전달해준다.  
위의 사진을 보면 Header에 Type : HeaderTest 이렇게 key와 value값으로 추가된 것을 볼 수 있다.  


ResponseEntity 객체를 테스트 하였을때를 정리하면 다음과 같다.  

- Status - 응답을 성공적으로 반환한다면 200
- Header - 5개의 정보
- Body - Java Object

@ResponseBody를 이용하였을 때와 동일하지만 ResponseEntity는 좀 더 유동적이라는 장점이 있다.  
@ResponseBody를 이용할 경우 상태코드를 변경하기 위해서는 @ResponseStatus를 이용해야하고,  
Header를 변경하기 위해서는 추가적인 로직이 필요하다. 즉 유연하지 못하다.  
하지만 ResponseEntity 객체를 이용하면 Builder 패턴을 이용하여 쉽게 응답 상태코드와 Header 정보를  
변경할 수 있다.  

### Builder 패턴의 사용 이유
~~~java
public class ResponseEntity<T> extends HttpEntity<T> {
    private final Object status;

    public ResponseEntity(HttpStatusCode status) {
        this((Object)null, (MultiValueMap)null, (HttpStatusCode)status);
    }

    public ResponseEntity(@Nullable T body, HttpStatusCode status) {
        this(body, (MultiValueMap)null, (HttpStatusCode)status);
    }

    public ResponseEntity(MultiValueMap<String, String> headers, HttpStatusCode status) {
        this((Object)null, headers, (HttpStatusCode)status);
    }

    public ResponseEntity(@Nullable T body, @Nullable MultiValueMap<String, String> headers, HttpStatusCode status) {
        this(body, headers, (Object)status);
    }

    public ResponseEntity(@Nullable T body, @Nullable MultiValueMap<String, String> headers, int rawStatus) {
        this(body, headers, (Object)rawStatus);
    }
~~~

ResponseEntity의 생성자를 보면 다음과 같다. 여러개의 생성자를 가지고 있어서,  
Status, Header, Body를 추가로 설정할 필요 없을때 null로 전달하여도 상관 없다.  
하지만 생성자를 이용하였을때의 return 구문을 보면 다음과 같다.  

~~~java
@GetMapping("/rest/constructor/member")
public ResponseEntity<String> responseEntityConstructorTest(){
    return new ResponseEntity<String>("Constructor Test", HttpStatus.valueOf(200));
}
~~~

위의 코드처럼 응답 상태코드 전달 시 응답코드를 하드코딩으로 해야되기 때문에,  
Builder Pattern을 권장하는 것이다.  

### 와일드카드
이는 어려운 게념이 아닌 제네릭 타입을 사용하겠다는 것이다.  
즉 제네릭 타입으로 타입을 명시하지 않고, 런타임까지 유연하게 ResponseEntity 객체를 이용하겠다는 것이다.  

~~~java
@GetMapping("/rest/generic/member")
public ResponseEntity<?> responseEntityGenericTest(){
    return ResponseEntity.ok("Generic Test");
}
~~~

하지만 이와 같은 방법은 권장되지 않는다.  
와일드카드는 객체 지향 개념에 적합하지 않고 가독성까지 떨어진다는 단점을 보유하고 있다.  
따라서 객체 타입이 명확하지 않은 경우 제네릭 타입보다는 Object 타입을 사용하는 것이 좋다.  

## ResponseEntity 사용 이유

ResponseEntity의 사용 이유를 살펴보면 다음과 같다.  
