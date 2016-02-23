---
layout: post
title: Spring Boot Hello - Step 02
---

#### 목표
* Spring Boot 프로젝트에 JPA 사용을 위한 설정을 추가한다.
* H2 데이터베이스를 설정한다.
* 데이터저장을 위해 Entity 클래스, JpaRepository 인터페이스 를 추가한다.
* Spring MVC 를 통해 데이터입출력을 처리하기 위해 Controller, Service 클래스를 추가한다.
* 샘플 화면을 보여주기 위한 JSP 를 추가한다.
* Spring Boot Application을 실행하고 결과를 확인한다.

#### 개발환경 (Environment)
* STS 3.7.2
* JDK 1.8.0_65
* Mac OS

#### 소스 코드 (Source)
* 본 Tutorial에서 완성된 코드가 포함된 Release
* [0.0.2-SNAPSHOT](https://github.com/wall72/SpringBootHello/releases/tag/0.0.2-SNAPSHOT "Spring Boot Hello - Step 02")

#### 1. build.gradle dependency 추가
* JPA 사용을 위해 jpa 를 추가
* JPA의 저장소로 RDBMS H2 사용을 위해 h2 추가

```
compile("org.springframework.boot:spring-boot-starter-data-jpa")
runtime("com.h2database:h2")
```

* JPA 관련 코드 작성시 IDE에서 에러가 발생하지 않도록 Dependencies를 Update한다. (Maven의 Update Project와 동일)
* {Project} > Gradle > Refresh All

#### 2. H2 Console 설정 추가
* H2 데이터베이스를 접속하여 저장소를 확인, 관리 하기 위해 application.properties 에 설정 추가

```
# H2 Web Console (H2ConsoleProperties)
spring.h2.console.enabled=true
spring.h2.console.path=/console
```

* Spring Boot 의 application.properties 기본설정을 살펴보면 H2 의 디폴트 기동 정보를 알 수 있다.
* [작성참조](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html "Common application properties")

* 접속테스트 (http://localhost:8080/console)
* JDBC URL > jdbc:h2:mem:testdb

![Screenshot #1](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-07.png?raw=true)

#### 3. Entity 클래스 추가
* 자료 저장을 위한 저장단위 클래스 선언을 위해 domain 패키지에 Entity 클래스를 추가한다.
* hello.domain.Post

```java
package hello.domain;

import java.util.Date;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Post {
    @Id
    @GeneratedValue
    int id;

    String subject;

    String content;

    Date regDate;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getSubject() {
        return subject;
    }

    public void setSubject(String subject) {
        this.subject = subject;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public Date getRegDate() {
        return regDate;
    }

    public void setRegDate(Date regDate) {
        this.regDate = regDate;
    }
}
```

* JPA 에서 자료저장 단위 Entity 식별은 @Entity 어노테이션으로 정의한다.
* Entity 의 ID (like Primary Key) 는 @Id 를 해당 멤버변수 선언 위에 추가한다.
* 추가로 자동 부여된 값을 부여하고 싶을 경우 (like sequence) @GeneratedValue 를 지정한다.

* 어플리케이션 기동 후 H2 Console 확인하면 테이블 자동 생성되어있음을 확인할 수 있다.

![Screenshot #2](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-08.png?raw=true)

#### 4. Repository 인터페이스 추가
* JPA 를 통한 자료 저장을 위해 domain 패키지에 JpaRepository 를 상속한 인터페이스를 추가한다.
* hello.domain.PostRepository

```java
package hello.domain;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PostRepository extends JpaRepository<Post, Integer> {

}
```

* That's it! 이것만으로 JPA 를 통한 저장소 접근을 위한 모든 준비가 끝났다.

* [사용참조](http://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html "org.springframework.data.jpa.repository")

#### 5. Service 클래스 추가
* 비즈니스 로직을 처리하기 위해 service 패키지에 Service 클래스를 추가한다.
* hello.service.PostService

```java
package hello.service;

package hello.service;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import hello.domain.Post;
import hello.domain.PostRepository;

@Service
public class PostService {

    @Autowired
    private PostRepository postRepository;

    public Post savePost(Post post) {
        return postRepository.save(post);
    }

    public void deletePost(int id) {
        postRepository.delete(id);
    }

    public List<Post> listPost() {
        return postRepository.findAll();
    }

    public Post findOne(int id) {
        return postRepository.findOne(id);
    }
}
```

#### 6. Controller 클래스 추가
* 클라이언트에서 요청한 리퀘스트를 처리하기 위해 web 패키지에 Controller 클래스를 추가한다.
* hello.web.PostController

```java
package hello.web;

import java.util.Date;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import hello.domain.Post;
import hello.service.PostService;

@Controller
@RequestMapping("/post")
public class PostController {

    @Autowired
    private PostService postService;

    @RequestMapping(value = "/write", method = RequestMethod.GET)
    public String form(Post post) {
        return "form";
    }

    @RequestMapping(value = "/write", method = RequestMethod.POST)
    public String write(Post post) {
        post.setRegDate(new Date());
        return "redirect:/post/" + postService.savePost(post).getId();
    }

    @RequestMapping("/{id}/delete")
    public String delete(@PathVariable int id) {
        postService.deletePost(id);
        return "redirect:/post/list";
    }

    @RequestMapping("/list")
    public String list(Model model) {
        List<Post> postList = postService.listPost();
        model.addAttribute("postList", postList);

        return "blog";
    }

    @RequestMapping("/{id}")
    public String view(Model model, @PathVariable int id) {
        Post post = postService.findOne(id);
        model.addAttribute("post", post);
        return "post";
    }
}
```

* 블로그 목록 조회 > /post/list
* 블로그 글 조회 > /post/{id}
* 글 작성 화면 호출 > /post/write (GET)
* 글 작성 저장 > /post/write (POST)
* 글 삭제 > /post/{id}/delete


#### 7. JSP 추가

* blog.jsp

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html>
<html lang="ko">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Blog</title>
</head>
<body>
    <div>
        <h2>List of Post</h2>
    </div>
    <c:if test="${postList.size() == 0}">
        <div>
            <h2>no data</h2>
        </div>
    </c:if>

    <c:forEach var="post" items="${postList}">
        <div>
            <h2>
                <c:out value="${post.id}" />
                <a href="/post/${post.id}">
                    <c:out value="${post.subject}"/>
                </a>
                <c:out value="${post.regDate}" />
                <a href="/post/${post.id}/delete">
                    delete
                </a>
            </h2>
        </div>
    </c:forEach>
</body>
</html>
```

* post.jsp

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html>
<html lang="ko">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Post</title>
</head>
<meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
<title>${post.subject}</title>
</head>
<body>
    <div>
        <h2>${post.subject}</h2>
        Posted on ${post.regDate}
    </div>
    <br>
    <div>
        id      : ${post.id}<br>
        subject : ${post.subject}<br>
        content : ${post.content}<br>
        regDate :${post.regDate}<br>
    </div>
</body>
</html>
```

* form.jsp

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html>
<html lang="ko">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Write</title>
</head>
<body>
    <div>
        <h2>Write</h2>
    </div>
    <div>
        <form action="/post/write" method="post">
            Subject : <input type="text" name="subject"><br>
            Content : <input type="text" name="content"><br>
            <button type="submit">Submit</button>
        </form>
    </div>
</body>
</html>
```

8. 테스트
* 블로그 목록 > http://localhost:8080/post/list
* 블로그 글 > http://localhost:8080/post/{id}
* 글 작성 > http://localhost:8080/post/write
* 글 삭제 > http://localhost:8080/post/{id}/delete