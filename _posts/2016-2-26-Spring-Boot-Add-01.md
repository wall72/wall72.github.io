---
layout: post
title: Spring Boot Hello - Appendix 01
---

#### 목표
* Spring Boot 프로젝트에 트랜잭션 설정을 추가한다.
* 로그 기록을 위하여 Spring Boot 에서 제공하는 slf4j를 사용 한다.
* Spring Security 적용시 H2 Console 을 사용가능하게 한다.

#### 개발환경 (Environment)
* STS 3.7.2
* JDK 1.8.0_65
* Mac OS

#### 소스 코드 (Source)
* 본 Tutorial의 1, 2 완성된 코드가 포함된 Release
* [0.0.3-SNAPSHOT](https://github.com/wall72/SpringBootHello/releases/tag/0.0.3-SNAPSHOT "Spring Boot Hello - Step 03")

#### 1. 트랜잭션이 처리 설정 추가
* Spring Service 함수에 @Transactional 을 추가한다.
; savePost(), deletePost()
; deletePost() 마지막에 다음 라인을 넣어서 테스트 한다.

```java
throw new RuntimeException(); // throw an Exception
```

* [사용참조](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/transaction.html#tx-decl-explained "Understanding the Spring Framework’s declarative transaction implementation")


#### 2. 로그  기록 설정
* spring-boot-starter-logging 을 추가
* spring-boot-starter-web 을 사용하면 기본적으로 logback 설정됨
* 사용할 클래스에 Logger를 추가하여 사용

```java
Logger logger = LoggerFactory.getLogger(getClass());
// ...
logger.info("Logging what? -->>");
```

* [사용참조](http://www.slideshare.net/whiteship/ss-47273947 "스프링 부트와 로깅")


#### 3. Spring Security 적용시 H2 console 접근 허용
* Security 설정에 "/console/**" 접근 허용 추가
* Security 설정에 csrf 끄기 처리
* Production에 적용해선 안된다.

```java
// ...
http
     .authorizeRequests()
         .antMatchers("/hello", "/post/list", "/console/**").permitAll()
// ...
http.csrf().disable();
http.headers().frameOptions().disable();
// ...
```

* [사용참조](http://gokgo.tistory.com/112 "스프링 부트와 스프링 시큐리티에서 H2 데이터베이스 콘솔사용하기")
