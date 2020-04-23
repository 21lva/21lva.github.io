---
layout: post
title:  "Controller"
name: sb_controller.md
category: spring-boot
---

@Contorller
==========

- MVC에서 C의 역할을 하는 Controller
- View를 **Model** 혹은 **ModelAndVie** 를 통해서 전달하는 역할(viewResovler를 통해서)

- View 파일이 아닌 단순 데이터(json 등)을 전달하기 위해서는 해당 메서드에 **@ResponseBody** 를 사용한다

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping(value = "/retrieveUserInfo", method = RequestMethod.POST)
    public @ResponseBody UserVO retrieveUserInfo(@RequestBody UserVO userVO){
        return userVO;
    }

    @RequestMapping(value = "/userInfoView", method = RequestMethod.POST)
    public String loginUserView(@RequestBody UserVO userVO, Model model){
        model.addAttribute("userInfo", userVO);
        return "/user/userInfoView";
    }

}
```

- - -

@RestController
====================

- @Controller +  @ResponseBody 라고 생각하면 된다
- message converter를 통해서 http response에 쓰이고 전달된다
  - message converter를 이용하고자 하면 com.fasterxml.jackson.core라는 라이브러리를 사용해야 한다.
  - spring-boot에서는 자동으로 적용해준다

- 아래와 같이 content type를 지정해서 전달받거나(consumes) 전달할(produces) 수 있다

```java
@GetMapping(value = "/xml2",
  consumes=MediaType.APPLICATION_JSON_UTF8_VALUE,
 produces = MediaType.APPLICATION_XML_VALUE)
public String xml2() {
    return "<a><b>content</b></a>";
}
```

- - -

@PathVariable
-------------

- 경로의 값을 얻을 수 있다

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping(value="/{id}/{name}", method=RequestMethod.GET)
    public void getMethod(@PathVariable int id, @PathVariable String name) {
        System.out.println("id=" + id + ", name=" + name);
    }
}
```

@RequestParam
---------------

- URL의 패래미터를 얻을 수 있다
  - **URL/hello?id=12&name=inhyuk** 의 id, name이 parameter이다

```java

@RestController
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping(method=RequestMethod.GET)
    public void getMethod(
            @RequestParam String id,
            @RequestParam Map<String, String> queryParameters,
            @RequestParam MultiValueMap<String, String> multiMap) {

        System.out.println("id=" + id);
        System.out.println(queryParameters);
        System.out.println(multiMap);
    }
}
```

@RequestHeader
----------------

- header의 값을 얻는다

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping(method=RequestMethod.GET)
    public void getMethod(@RequestHeader(value="Test-Header",required=false,defaultValue="HELLO") String value) {
        System.out.println("Test-Header=" + value);
    }
}
```

- curl을 이용하여 테스트 해보자

```
$ curl -H "Test-Header: hoge" http://localhost:8080/hello
```

@RequestBody
--------------

- request의 body값을 가져옴

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping(method=RequestMethod.POST)
    public void getMethod(@RequestBody String body) {
        System.out.println("body=" + body);
    }
}
```

@ReponseStatus
--------------

- 응답에 status 값을 지정해주고 이유를 전달
- 아무것도 지정하지 않으면 200(OK)가 전달된다

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping(method=RequestMethod.GET)
    @ResponseStatus(value=HttpStatus.BAD_REQUEST,reason="NONONO")
    public void getMethod() {
    }
}
```

- - -

참고자료
========

- [자료1](https://mangkyu.tistory.com/49)
- [자료2](https://wondongho.tistory.com/76)
- [Pathvariable, RequestParam 등](http://www.devkuma.com/books/pages/471)
