---
layout: post
title: Spring Boot Hello - Step 01
---

#### 목표
* Spring Boot 를 이용하여 Web 프로젝트를 생성한다.
* Spring Boot 프로젝트 구조를 이해한다.
* JSP 사용을 위한 설정을 추가한다.
* 샘플 화면을 보여주기 위한 Contoller, JSP 를 추가한다.
* Spring Boot Application을 실행하고 결과를 확인한다.

#### 개발환경 (Environment)
* STS 3.7.2
* JDK 1.8.0_65
* Mac OS

#### 소스 코드 (Source)
* 본 Tutorial에서 완성된 코드가 포함된 Release
* [0.0.1-SNAPSHOT](https://github.com/wall72/SpringBootHello/releases/tag/0.0.1-SNAPSHOT "Spring Boot Hello - Step 01")
* 앞으로 진행되는 Tutorials도 같은 Repository에 Push 될 것이다.

#### 1. Create Project
* New > Spring Starter Project
* Packaging: War (웹 프로젝트를 위한 패키징)
* Dependencies: Web (Spring Web, MVC 를 사용하기 위함)

![Screenshot #0](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-01.png?raw=true)
![Screenshot #0](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-02.png?raw=true)

#### 2. build.gradle dependency 추가
* JSTL사용을 위해 jstl 을 추가
* Application 구동시 실행되는 Tomcat에 JSP 실행을 위한 런타임 tomcat-embed-jasper 추가

```
compile('javax.servlet:jstl')
providedRuntime('org.apache.tomcat.embed:tomcat-embed-jasper')
```

* JSP, JSTL 코드 작성시 IDE에서 에러가 발생하지 않도록 Dependencies를 Update한다. (Maven의 Update Project와 동일)
* {Project} > Gradle > Refresh All

* 참고: build.gradle 내용을 살펴보면 기본 내장 WAS 엔진으로 tomcat이 이미 추가되어 있는 것을 알 수 있다.

```
providedRuntime('org.springframework.boot:spring-boot-starter-tomcat')
```

#### 3. application.properties 수정
* 기본생성된 application.properties는 Spring Boot Application의 여러 설정 정보를 프로퍼티로 기술하기 위한 파일이다.
* 서버 기동시 사용할 정보를 추가한다.
* Spring MVC 관련 정보를 추가한다.

```
#Server
server.port=8080
server.display-name=spring-boot-jsp
server.session.timeout=3600

#Spring MVC
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

* [작성참조](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html "Common application properties")

#### Spring Boot 소스 폴더구조
* Spring Boot Project 로 생성되는 Project Directory Structure는 Maven의 그 것과 동일하다.
* 소스 폴더 내의 Structure는 아래 구조를 따라 Application을 Package root에 두고 web,service,domain 의 폴더 구조를 권고한다.

![Screenshot #1](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-06.png?raw=true)

* [작성참조](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-locating-the-main-class "Locating the main application class")

#### 4. Controller 생성
* web 패키지에 Controller 클래스를 추가한다.
* hello.web.HelloController

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

* JSP에 값을 전달하여 표시하기 위해 샘플코드를 추가했다.

* [작성참조](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html "Web MVC framework")

#### 5. JSP 생성
* application.properties에 기술한 jsp 경로에 jsp 파일을 추가한다.
* webapp/WEB-INF/jsp/hello.jsp

![Screenshot #2](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-04.png?raw=true)

* Maven의 Project Structure에 따라 {Project}/src/webapp 폴더에 Web resources 가 추가된다.

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

* JSTL을 이용하여 전달된 값을 표시하는 간단한 샘플코드를 추가했다.

#### 6. Start
* Spring Boot Project의 기동은 Dependency로 추가된 Tomcat 등의 WAS를 Application의 기동으로 실행하는 방법과 IDE툴의 기능을 이용하는 방법이 있다.
* 기본적으로 Spring Boot Project 를 이용하여 실행한다.
* {Project} > Run As > Spring Boot App

#### 7. Test
* 웹브라우저로 결과를 확인한다.
* http://localhost:8080/hello

![Screenshot #3](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-05.png?raw=true)

#### 8. Reference Sites
* [origoni's Blog from Millky](http://millky.com/@origoni/post/1100 "STS로 Spring Boot 웹 프로젝트 시작하기")
* [Ticket Monster의 개발 이야기](http://tmondev.blog.me/220596351807 "웹 프로젝트의 간편한 시작, Spring Boot 와 데모 프로젝트")
