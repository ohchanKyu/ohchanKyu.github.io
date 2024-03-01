---

layout: post
title:  "Controller"
date:   2024-02-20 01:38:04 +0900

---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## Controller란?

## Annotation
### - @RequestMapping


## Controller vs Rest Controller
### - 기본 Controller
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





