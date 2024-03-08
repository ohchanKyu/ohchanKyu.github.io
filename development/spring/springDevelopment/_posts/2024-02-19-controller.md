---

layout: post
title:  "[Spring] Controller"
date:   2024-02-20 01:38:04 +0900

---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## Controller란?
Spring framework는 기본적으로 MVC 패턴을 사용한다.  
그렇다면 MVC 패턴은 무엇일까?  
MVC 패턴은 Model, View, Controller라는 3개의 역할을 나누어 작업을 진행한다.  
MVC 패턴의 관계를 살펴보면 다음과 같다.  
1. 사용자가 어떤 요청을 한다.(ex.웹 접속, 버튼 클릭)
2. 요청을 Controller에서 받는다.
3. Controller는 Service 계층에서 원하는 로직을 처리한 후 결과를 Model에 저장한다.  
4. Model에 저장한 결과를 바탕으로 시각적인 요소를 출력하는 View를 제어하여 사용자에게 제공한다.  

View : html,css,js가 합쳐진 jsp라고 생각하면 된다.  
Model : Controller에서 제어한 Java의 변수를 저장한다고 생각하면 된다.  
Controller : 요청을 받고 여러 로직을 처리 후 필요한 정보들을 Model에 저장하는 역할이다.  
위와 같이 생각하면 편리하다.
{:.note}

따라서 Controller는 사용자의 요청을 받고 Service 계층에서 비즈니스 로직을 처리하고  
필요한 정보들을 Model에 저장하여 View를 만들고 사용자에게 다시 보여준다고 생각하면 된다.  

사용자가 일반적인 Web 페이지에 접속한다고 생각하면 다음과 같다.  
사용자가 https://demo/home 이런 URL로 접속했다고 하자.  
그러면 Controller는 이러한 요청을 감지하고 이에 대한 로직을 처리 후 해당 웹 화면을 보여준다. 
{:.note}

그렇다면 Spring에서 어떤 특정한 Class가 Controller라는 것을 어떻게 알 수 있을까?  
바로 Annotation을 통해 알 수 있다.  
Spring framework는 대부분 Annotation을 통해 해당 계층이 어떤 역할을 하는지 알려준다.  
- @Controller  
@Controller라는 Annotation을 붙여서 해당 Class가 Controller라고 알려주는 것이다.  


## Annotation
### - @RequestMapping
@RequestMapping은 URL 요청에 대해 어떤 Controller를 사용할 것인지에 대한 Annotation이다.  
위에서 처럼 사용자는 웹에 접속 시 https://demo/home라는 URL로 접속하면  
해당 URL를 감지하고 적절한 Controller를 연결해야 한다.

~~~java
@Controller
public class MainController {

    @RequestMapping("/home")
    public String homePage(){
        return "index.html";
    }

}
~~~

@Controller와 @RequestMapping에 대한 예시이다.  
- @Controller  
해당 어노테이션을 통해 Controller Class라고 알려준다.  
- @RequestMapping  
/home이라는 URL로 접속시 사용자에게 index.html이라는 html 문서를 보여준다.

~~~java
@Controller
@RequestMapping("/test")
public class TestController {

    @RequestMapping("/home")
    public String homePage(){
        return "index.html";
    }

    @RequestMapping("/blog")
    public String blogPage(){
        return "blog.html";
    }
}
~~~
프로젝트가 커질수록 Controller의 수가 많아질 수 있다.  
하나의 Controller만 사용하는 것이 아닌 유지보수가 쉽게 용도에 맞게 여러 Controller를 사용하는 것이  
바람직할 수도 있다. 따라서 Controller가 여러개 일때 위의 코드처럼 @RequestMapping을 Class위에 하여    
URL에 따라 구분하는 것도 좋은방법이다.  
위의 코드에 따르면 http://localhost:8080/test/home으로 URL 작성해야 해당 메소드로 연결된다.  
또한 http://localhost:8080/test/blog URL로 접속한다면 blogPage()라는 메소드로 연결된다.  


## Controller vs Rest Controller
### - @Controller
### - Controller + @ResponseBody
### - @RestController

## Http Request / Rest API (Get,Post,Put,Patch,Delete)


## Controller Params
### - @RequestParam
~~~java
@GetMapping("/getEmail")
public ResponseEntity<String> requestParamTest(@RequestParam("email") String email){
    return ResponseEntity.ok(email);
}
~~~

### - @RequestPart
~~~java

~~~

### - @RequestBody
Member DTO
~~~java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Member {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private String email;
    private String password;
    @Column(length = 600)
    private String description;
    @Convert(converter = StringListConverter.class)
    private List<String> demoDate;
}
~~~

~~~java
@PostMapping("/addMember")
public ResponseEntity<Member> requestBodyTest(@RequestBody Member member){
    return ResponseEntity.ok(member);
}
~~~

### - @PathVariable
~~~java
@GetMapping("/{email}")
public ResponseEntity<String> pathVariableTest(@PathVariable String email){
    return ResponseEntity.ok(email);
}
~~~





