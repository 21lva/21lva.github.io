---
layout: post
title:  "환경설정, @Value"
name: sb_property.md
---

환경 프로퍼티 파일 설정
==================

- 다양한 환경을 설정하는 파일을 yaml(한 파일 안에 다양한 설정을 저장할 수 있음)에 저장한다
- 환경(local, test, 배포 등)에 맞게 여러 설정값을 저장할 수 있다
  - **-** 를 기준으로 인식

```yaml
server: 80

---

spring:
  profiles: local
server:
  port: 8080

---

spring:
  profiles: dev
server:
  port: 8081
```

- dev 환경을 사용하고자 하면

```
java -jar ... -D spring.profiles.active=dev
```

- - -

YAML 파일 mapping
==================

- properties 혹은 yaml 파일에 있는 값을 사용하기 위해 **@Value** 혹은 **@ConfigurationProperties** 를 사용

@Value
------

- 프로퍼티의 key를 사용하여 특정한 값을 호출

```yaml
property:
  test:
    name: hi hello
propertyTest: test
propertyTestList: a,b,c
```

- 위의 값을 mapping 해보자

```java
@Runwith(SpringRunner.class)
@SpringBootTest
public class AutoConfigurationApplicationTest{
  //key mapping
  @Value("${property.test.name}")
  private Stirng propertyTestName;

  //key mapping
  @Value("${propertyTest}")
  private Stirng propertyTest;

  //SpEL을 사용하여 mapping
  @Value("#{'${propertyTestList}'.split(',')}")
  private Stirng propertyTestList;

  //YAML에 key가 존재하지 않으면 default 값으로 매핑
  @Value("${noKey:default value}")
  private Stirng defaultValue;

  @Test
  public void testValue(){
    //생략
  }
}
```
