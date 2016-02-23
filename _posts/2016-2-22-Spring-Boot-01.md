---
layout: post
title: Spring Boot Hello - Step 01
---

### 개발환경 (Environment)
* STS 3.7.2
* JDK 1.8.0_65
* Mac OS

#### 소스 코드 (Source)
<https://github.com/wall72/SpringBootHello/releases/tag/0.0.1-SNAPSHOT>

#### 1. Create Project
* New > Spring Starter Project
* Packaging: War
* Dependencies: Web

![Screenshot #0](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-01.png?raw=true)
![Screenshot #0](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-02.png?raw=true)

#### 2. build.gradle dependency 추가
* jstl, tomcat-embed-jasper 추가

```
compile('javax.servlet:jstl')
providedRuntime('org.apache.tomcat.embed:tomcat-embed-jasper')
```

* Gradle > Refresh All

#### 3. application.properties 수정
; Server, Spring.mvc 설정

```
#Server
server.port=8080
server.display-name=spring-boot-jsp
server.session.timeout=3600

#Spring MVC
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

#### Spring Boot 소스 폴더구조
![Screenshot #1](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-06.png?raw=true)

#### 4. Controller 생성
; hello.web.HelloController

```java
package hello.web;

import java.util.Date;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HelloController {
    @RequestMapping("/hello")
    public String home(Model model) {
        model.addAttribute("title", "Spring Boot Demo");
        model.addAttribute("time", new Date().toString());
        return "hello";
    }
}
```

#### 5. JSP 생성
; webapp/WEB-INF/jsp/hello.jsp

![Screenshot #2](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-04.png?raw=true)

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html>
<html lang="ko">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Welcome Hello Page</title>
</head>
<body>
    <h2>Title : ${title}</h2>
    <h2>Time : ${time}</h2>
</body>
</html>
```

#### 6. Start
; Run As > Spring Boot App

#### 7. Test
; http://localhost:8080/hello

![Screenshot #3](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-05.png?raw=true)

#### 8. Reference Sites
* [origoni's Blog from Millky](http://millky.com/@origoni/post/1100 "STS로 Spring Boot 웹 프로젝트 시작하기")
* [Ticket Monster의 개발 이야기](http://tmondev.blog.me/220596351807 "웹 프로젝트의 간편한 시작, Spring Boot 와 데모 프로젝트")
