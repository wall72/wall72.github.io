---
layout: post
title: Spring Boot Hello - Step 03
---

#### 목표
* Spring Boot 프로젝트에 Spring Security 사용을 위한 설정을 추가한다.
* User 데이터저장을 위해 Entity 클래스, JpaRepository 인터페이스 를 추가한다.
* 초기 데이터 적재를 위해 import.sql 파일을 추가한다.
* Spring Security 설정 클래스를 추가한다.
* 로그인을 위한 JSP 를 추가한다.
* 로그인 처리를 위한 UserDetailService 를 상속 받은 클래스를 추가한다.
* Spring Security Remember-me 를 설정 한다.

#### 개발환경 (Environment)
* STS 3.7.2
* JDK 1.8.0_65
* Mac OS

#### 소스 코드 (Source)
* 본 Tutorial에서 완성된 코드가 포함된 Release
* [0.0.3-SNAPSHOT](https://github.com/wall72/SpringBootHello/releases/tag/0.0.3-SNAPSHOT "Spring Boot Hello - Step 03")

#### 1. build.gradle dependency 추가
* Spring Security 사용을 위해 추가

```
compile("org.springframework.boot:spring-boot-starter-security")
```

* Spring Security 관련 코드 작성시 IDE에서 에러가 발생하지 않도록 Dependencies를 Update한다. (Maven의 Update Project와 동일)
* {Project} > Gradle > Refresh All

#### 2. Entity 클래스 추가
* 사용자 정보 저장 및 읽기를 위한 Entity 클래스를 추가한다.
* hello.domain.User

```java
package hello.domain;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class User {
    @Id
    @GeneratedValue
    int id;

    String username;

    String password;

    String role;

    // {Getters, Setters}
}
```

* {Getters, Setters} 부분은 STS의 기능으로 채운다.

#### 3. Repository 인터페이스 추가
* 사용자 정보 저장 및 읽기를 위한 Repository 인터페이스를 추가한다.
* hello.domain.UserRepository

```java
package hello.domain;

import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Integer> {
    public User findByUsername(String username);
}
```

* User 저장소에 id와는 별개로 username을 구성하였으므로 이를 이용해 사용자를 찾아오는 findByUsername 함수를 추가했다.

* [사용참조](http://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html "org.springframework.data.jpa.repository")

#### 4. DBMS 초기화 파일 추가
* Spring boot 의 JPA는 기본적으로 Hibernate를 사용하므로 이를 이용해 초기 데이터 적재를 처리할 import.sql 파일을 resources 폴더에 추가한다.
* User 테이블의 사용자 정보로 로그인 처리를 하기 위함이다.
* {Project}/src/main/resources/import.sql

```sql
insert into User(id, username, password, role) values (1, 'user', 'password', 'USER');
insert into User(id, username, password, role) values (2, 'admin', 'password', 'ADMIN');
insert into post(id, subject, content, reg_date) values (1, 'Blog Title #1', 'Content - blur blur', CURRENT_TIMESTAMP);
insert into post(id, subject, content, reg_date) values (2, 'Blog Title #2', 'Content - klur klur', CURRENT_TIMESTAMP);
```

* 초기 데이터로 Post 도 몇개 넣었다.

#### 5. Controller 클래스에 /login 추가
* 로그인을 위해 클라이언트에서 요청한 '/login' 리퀘스트에 대해 로그인 페이지를 응답하기 위해 함수를 추가한다.
* '/post' 하위에 넣을 수는 없으니 HelloController 를 이용한다.
* hello.web.HelloController

```java
@Controller
public class HelloController {

    // ...

    @RequestMapping(value = "/login", method = RequestMethod.GET)
    public String login(Post post) {
        return "login";
    }
}
```

#### 6. login.jspP 추가
* 로그인 처리를 위한 login.jsp 를 추가한다.
* username, password, 외에 csrf 를 추가로 정의 한다.
* csrf는 CSRF 공격을 방어하기 위한 것으로 Spring boot에 적용된 Spring Security 에선 기본 설정 항목이다. (옵션을 끌 수도 있지만 보안상 권고하지 않는다.)

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html>
<html lang="ko">
<head>
    <title>Log-in</title>
</head>
<body>
    <form action="/login" method="post">
        User Name : <input type="text" name="username"><br>
        Password : <input type="password" name="password"><br>
        <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
        <input type="submit" value="Log In">
    </form>
</body>
</html>
```

#### 7. Spring Security 설정 클래스 추가
* Spring Security 설정을 위한 클래스를 추가한다.
* hello.WebSecurityConfig

```java
package hello;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/hello", "/post/list").permitAll() // 모두 허용
                .anyRequest().authenticated()                    // 나머지는 인증
                .antMatchers("/post/write", "/post/*/delete").hasRole("ADMIN") // ADMIN만 접근 가능
                .and()
            .formLogin()
                .loginPage("/login").permitAll()
                .and()
            .logout()
                .permitAll()
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .inMemoryAuthentication()
                .withUser("user").password("password").roles("USER").and()
                .withUser("admin").password("password").roles("ADMIN");
    }
}
```

* configure 함수를 override 해서 설정항목을 정의해 주고
* 로그인 처리를 위한 configureGlobal 함수를 기술 한다.
* 일단은 JPA연계 없이 간단히 메모리에 User 정보를 정의해서 테스트한다.

* [사용참조](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#jc "Spring Security Java Configuration")

* 어플리케이션 실행 후 정책대로 되는지 확인한다.
* 로그아웃은 http://localhost:8080/logout


#### 8. Service 클래스 추가
* 로그인을 처리하기 위해 service 패키지에 UserDetailService 클래스를 상속받은 서비스를 추가한다.
* hello.service.SimpleUserDetailService

```java
package hello.service;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import hello.domain.UserRepository;

@Service
public class SimpleUserDetailService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        hello.domain.User user = userRepository.findByUsername(username);

        List<GrantedAuthority> authorities = buildUserAuthority(user.getRole());

        return new User(user.getUsername(), user.getPassword(), true, true, true, true, authorities);
    }

    private List<GrantedAuthority> buildUserAuthority(String userRole) {
        List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();

        authorities.add(new SimpleGrantedAuthority(userRole));

        return authorities;
    }
}

```

* JPA를 통해 User 저장소에서 유저를 찾아오는 역할을 한다.
* loadUserByUsername 을 Override 해서 JPA에서 찾아온 User, Authority 를 생성하여 리턴한다.
* Authority는 일단은 사용자 당 하나만 부여하도록 간소화 했다.

* 앞서 만들었던 Spring Security 설정 파일에 적용한다.

```java
// ...
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService);
    }
// ...
```

* 어플리케이션 실행 후 정책대로 되는지 확인한다.
* 로그아웃은 http://localhost:8080/logout


#### 9. Remember-me 의 적용
* Spring Security의 로그인 유지 기능인 Remember-me를 사용한다.
* 서버에서 세션이 타임 아웃되더라도 Remember-me 에 설정된 시간 (디폴트: 2주 - 14days) 동안은 Remember-me 가 자동 로그인 처리를 해준다. (쿠키를 통한 상태 유지)

* login.jsp

```html
   <form action="/login" method="post">
        User Name : <input type="text" name="username"><br>
        Password : <input type="password" name="password"><br>
        Remember me : <input type="checkbox" name="remember-me"><br>
        <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
        <input type="submit" value="Sign In">
    </form>
```

* WebSecurityConfig

```java
// ...
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/hello", "/post/list").permitAll()
                .antMatchers("/post/write", "/post/*/delete").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login").permitAll()
                .and()
            .logout()
                .deleteCookies("remember-me")
                .permitAll()
                .and()
            .rememberMe();
    }
// ...
```

* [사용참조](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#ns-remember-me "Spring Security Remember-Me Authentication")

#### 10. Next
* 추가적으로 PasswordEncoder 적용에 대한 내용도 있으니 아래 Reference를 참고 한다.

#### 11. Reference Sites
* [Kielczewski.eu](http://kielczewski.eu/2014/12/spring-boot-security-application "Spring Boot Security Application")
* [Namooz Blog](http://www.namooz.com/2015/12/07/spring-boot-thymeleaf-10-spring-boot-security-final "Spring Boot + Thymeleaf 10 – Spring Security (final!)")
* [woniper](http://blog.woniper.net/232 "Spring Boot Data JPA 설정")
