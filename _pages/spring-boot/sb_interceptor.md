---
layout: post
title:  "Interceptor"
name: sb_interceptor.md
category: spring-boot
---

- [spring MVC에서의 Interceptor](todo)

의존성
-----

- interceptor 는 **spring-webmvc** 에 정의되어 있다.
  - **spring-boot-starter-web** 을 사용하면 자동으로 포함된다

구현
----

- Controller로 보내기 전에 호출됨
- 기본 인터페이스인 **HandlerInterceptorAdaptor** 를 구현하여 사용한다
- 3가지 메서드 사용.
  - preHandle(): Controller가 작동하기 전. 자주 쓰임. 반환 값이 true이면 controller로 넘어간다
  - postHandle(): Controller가 작동한 후
  - afterCompletion(): Controller와 View가 작동한 후


```java
@Component
public class HttpInterceptor extends HandlerInterceptor {

	private static final Logger logger = Logger.getLogger(HttpInterceptor.class);

	@Override
	public boolean preHandle(HttpServletRequest request,
							 HttpServletResponse response,
							 Object handler) {
		return true;
	}

	@Override
	public void postHandle( HttpServletRequest request,
							HttpServletResponse response,
							Object handler,
							ModelAndView modelAndView) {
	}

	@Override
	public void afterCompletion(HttpServletRequest request,
								HttpServletResponse response,
								Object handler,
								Exception ex) {
	}
}
```

등록
----

- interceptor를 사용하기 위해서는 config class를 생성해야 한다
- 인터페이스 **WebMvcConfigurer** 를 구현한다
- intercept할 대상 지정
  - **addPathPatterns** : 배열형태로 인터셉트 할 패턴을 넣어줄 수 있다.
  - **excludePathPatterns** : 1개씩 인터셉트할 패턴을 넣어 줄 수 있다.

```java
@Configuration
public class Configurations implements WebMvcConfigurer {

	@Override
	public void addInterceptors(InterceptorRegistry registry) {
        List<String> URL_PATTERNS = Arrays.asList("/배열로가능1", "/배열로가능2");
		registry.addInterceptor(new MyInterceptor())  
			.addPathPatterns(URL_PATTERNS)   
			.excludePathPatterns("/원하는패턴1")
			.excludePathPatterns("/원하는패턴2");
	}
}
```

- - -

참고자료
======

- <https://lts0606.tistory.com/246>
- <https://lts0606.tistory.com/246>
