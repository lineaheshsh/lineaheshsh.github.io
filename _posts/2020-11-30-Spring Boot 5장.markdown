---
layout: post
title: Spring Boot 5장
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2020-11-30 22:50:00'
category: Spring
published: true
image:  스프링부트.png
tags:   SpringBoot Spring Gradle OAuth2.0
---

### 스프링 부트 5장(OAuth2.0)  

###### 이번 화에서는 스프링 시큐리티와 OAuth2.0으로 로그인 기능 구현하는 방법을 진행해보려고 한다.

###### 스프링 시큐리티는 막강한 인증과 인가(혹은 권한부여) 기능을 가진 프레임워크이다. 사실상 스프링 기반의 애플리케이션에서는 보안을 위한 표준이라고 보면 된다.  
###### 인터셉터, 필터 기반의 보안 기능을 구현하는 것보다 스프링 시큐리티를 통해 구현하는 것을 적극적으로 권장하고 있다.

#### 1. 스프링 시큐리티와 스프링 시큐리티 Oauth2.0 클라이언트  
---
###### 많은 서비스에서 로그인 기능을 id/password 방식보다는 구글, 페이스북, 네이버 로그인과 같은 소셜 로그인 기능을 사용한다.  
###### 왜 많은 서비스에서 소셜 로그인을 사용할까? 직접 구현할 경우 배보다 배꼽이 커지는 경우가 많기 때문이다. 직접 구현하면 다음을 전부 구현해야 한다.  

+ 로그인 시 보안
+ 비밀번호 찾기
+ 회원가입 시 이메일 혹은 전화번호 인증
+ 비밀번호 변경
+ 회원정보 변경

###### OAuth 로그인 구현 시 앞선 목록의 것들을 모두 구글, 페이스북, 네이버 등에 맡기면 되니 서비스 개발에 집중할 수 있다.  
###### 자 그럼 먼저 구글 서비스에 신규 서비스를 생성해보자. 여기서 발급된 인증 정보(clientId와 clientSecret)를 통해서 로그인 기능과 소셜 서비스 기능을 사용할 수 있으니 무조건 발급 받고 시작해야 한다.  

###### 구글 클라우드 플랫폼 주소(https://console.cloud.google.com)로 이동한다. OAuth 클라이언트 ID로 발급을 받는다. (블로그에 발급 방법까지 다 쓰기에 너무 길어져 발급방법은 구글링을 통해 찾아보면 될 것 이다. )  

###### 발급 받은 클라이언트 ID와 클라이언트 보안 비밀코드를 프로젝트에서 설정을 해보자.  

+ application-oauth 등록
###### 4장에서 만들었던 application.properties가 있는 src/main/resources/ 디렉토리에 application-oauth.properties파일을 생성한다.  
![이미지]({{ site.url }}/images/5장application-oauth.png)
###### 그리고 해당 파일에 클라이언트 ID와 클라이언트 보안 비밀코드를 다음과 같이 등록한다.  
```properties
## Google Login Info
spring.security.oauth2.client.registration.google.client-id=클라이언트ID
spring.security.oauth2.client.registration.google.client-secret=클라이언트 보안 비밀
spring.security.oauth2.client.registration.google.scope=profile,email
```
  - 많은 예제에서는 이 scope를 별도로 등록하지 않고 있습니다.
  - 기본값이 openid,profile,email이기 때문입니다.
  - 강제로 profile,email을 등록한 이유는 openid라는 scope가 있으면 Open Id Provider로 인식하기 때문입니다.
  - 이렇게 되면 OpenId Provider인 서비스(구글)와 그렇지 않은 서비스(네이버,카카오 등)로 나눠서 각각 OAuth2Service를 만들어야 합니다.
  - 하나의 OAuth2Service로 사용하기 위해 일부러 openid scope를 빼고 등록합니다.
  
###### 스프링 부트에서는 properties의 이름을 application-xxx.properties로 만들면 xxx라는 이름의 profile이 생성되어 이를 통해 관리 할 수 있다.  
###### 즉, profile=xxx라는 식으로 호출하면 `해당 properties의 설정들을 가져올` 수 있다. 호출방식은 여러 방식이 있지만 이 책에서는 스프링 부트의 기본 설정 파일인 application.properties에서 application-oauth.properties를 포함하도록 구성한다.  
```properties
spring.profiles.include=oauth
```
###### 이제 이 설정값을 사용할 수 있게 되었다.  

+ .gitignore 등록  
###### 보안을 위해 깃허브에 application-oauth.properties 파일이 올라가는 것을 방지하자. 1장에서 만들었던 .gitignore에 다음과 같이 한 줄의 코드를 추가한다.
```
application-oauth.properties
```
###### 추가한 뒤 커밋했을 때 커밋 목록에 application-oauth.properties가 나오지 않으면 성공이다.
---
###### 구글의 로그인 인증정보를 발급 받았으니 프로젝트 구현을 진행하자. 먼저 사용자 정보를 담당할 도메인인 User 클래스를 생성하자. 패키지는 domain 아래에 user 패키지를 생성한다.  

```java
package com.zzangho.project.springboot.domain.user;

import com.zzangho.project.springboot.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column
    private String picture;

    // JPA로 데이터베이스로 저장할 때 Enum값을 어떤 형태로 저장할지를 결정합니다.
    // 기본적으로는 int로 된 숫자가 저장된다.
    // 숫자로 저장되면 데이터베이스로 확인할 때 그 값이 무슨 코드를 의미하는지 알 수가 없습니다.
    // 그래서 문자열(EnumType.STRING)로 저장될 수 있도록 선언합니다.
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(String name, String email, String picture, Role role) {
        this.name = name;
        this.email = email;
        this.picture = picture;
        this.role = role;
    }

    public User update(String name, String picture) {
        this.name = name;
        this.picture = picture;

        return this;
    }

    public String getRoleKey() {
        return this.role.getKey();
    }
}
```
###### 각 사용자의 권한을 관리할 Enum 클래스 Role을 생성한다.  
```java
package com.zzangho.project.springboot.domain.user;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum Role {

    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;
}
```

###### 스프링 시큐리티에서는 권한 코드에 항상 ROLE_이 앞에 있어야만 한다. 그래서 코드별 키 값을 ROLE_GUEST, ROLE_USER 등으로 저장한다.  
###### 마지막으로 User의 CRUD를 책임질 UserRepository도 생성한다.  

```java
package com.zzangho.project.springboot.domain.user;


import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    
    // 소셜 로그인으로 반환되는 값 중 email을 통해 이미 생성된 사용자인지 처음 가입하는 사용자인지 판단하기 위한 메소드입니다.
    Optional<User> findByEmail(String email);
}
```

###### User 엔티티 관련 코드를 모두 작성했으니 본격적으로 시큐리티 설정을 진행하자  
```
compile('org.springframework.boot:spring-boot-starter-oauth2-client')   //스프링 시큐리티(소셜 로그인 등 클라이언트 입장에서 소셜 기능 구현 시 필요)
```

+ spring-boot-starter-oauth2-client
  - 소셜 로그인 등 클라이언트 입장에서 소셜 기능 구현 시 필요한 의존성이다.
  - spring-security-oauth2-client와 spring-security-oauth2-jose를 기본으로 관리해준다.
  
###### build.gradle 설정이 끝났으면 OAuth 라이브러리를 이용한 소셜 로그인 설정 코드를 작성한다.  
###### config.auth 패키지를 생성한다. 앞으로 `시큐리티 관련 클래스는 모두 이곳에` 담는다고 보면 된다.  

###### SecurityConfig 클래스를 생성하고 다음과 같이 코드를 작성한다. 아직 `CustomOAuth2UserService 클래스를 만들지 않아` 컴파일 에러가 발생하지만 코드 설명을 본 뒤 바로 다음 코드를 작성하면 된다.  

```java
package com.zzangho.project.springboot.config.auth;

import com.zzangho.project.springboot.domain.user.Role;
import lombok.RequiredArgsConstructor;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                    .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**", "/vendor/**").permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/user/login")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(customOAuth2UserService);
    }
}

```
+ @EnableWebSecurity
  - Spring Security 설정들을 활성화시켜 줍니다.
+ csrf().disable().headers().frameOptions().disable()
  - h2-console 화면을 사용하기 위해 해당 옵션들을 disable 해야 한다.
+ authorizeRequests
  - URL별 권한 관리를 설정하는 옵션의 시작점입니다.
  - authorizeRequests가 선언되어야만 antMatchers 옵션을 사용할 수 있습니다.
+ antMatchers
  - 권한 관리 대상을 지정하는 옵션입니다.
  - URL, HTTP 메소드별로 관리가 가능하다.
  - "/"등 지정된 URL들은 permitAll() 옵션을 통해 전체 열람 권한을 주었습니다.
  - "/api/v1/**" 주소를 가진 API는 USER 권한을 가진 사람만 가능하도록 했습니다.
+ anyRequest
  - 설정된 값들 이외 나머지 URL들을 나타냅니다.
  - 여기서는 authenticated()을 추가하여 나머지 URL들은 모두 인증된 사용자들에게만 허용하게 합니다.
  - 인증된 사용자 즉, 로그인한 사용자들을 이야기합니다.
+ logout().logoutSuccessUrl("/")
  - 로그아웃 기능에 대한 여러 설정의 진입점입니다.
  - 로그아웃 성공 시 /주소로 이동합니다.
+ oauth2Login
  - OAuth2 로그인 기능에 대한 여러 설정의 진입점입니다.
+ userInfoEndPoint
  - OAuth2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당한다.
+ userService
  - 소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록합니다.
  - 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능을 명시할 수 있습니다.
  
###### 설정 코드 작성이 끝났다면 `CustomOAuth2UserService` 클래스를 생성한다. 이 클래스에서는 구글 로그인 이후 가져온 사용자의 정보들을 기반으로 가입 및 정보수정, 세션 저장 등의 기능을 지원 합니다. 

```java
package com.zzangho.project.springboot.config.auth;

import com.zzangho.project.springboot.config.auth.dto.OAuthAttributes;
import com.zzangho.project.springboot.config.auth.dto.SessionUser;
import com.zzangho.project.springboot.domain.user.User;
import com.zzangho.project.springboot.domain.user.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.client.userinfo.DefaultOAuth2UserService;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserRequest;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserService;
import org.springframework.security.oauth2.core.OAuth2AuthenticationException;
import org.springframework.security.oauth2.core.user.DefaultOAuth2User;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Service;

import javax.servlet.http.HttpSession;
import java.util.Collections;

@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService<OAuth2UserRequest, OAuth2User> delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration()
                                           .getRegistrationId();
        String userNameAttributeName = userRequest.getClientRegistration()
                                                  .getProviderDetails()
                                                  .getUserInfoEndpoint()
                                                  .getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                                                                 attributes.getAttributes(),
                                                                 attributes.getNameAttributeKey()
        );
    }

    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                                  .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                                  .orElse(attributes.toEntity());

        return userRepository.save(user);
    }
}
```

###### 구글 사용자 정보가 업데이트 되었을 때를 대비하여 update 기능도 같이 구현되었다. 사용자의 이름이나 프로필 사진이 변경되면 User 엔티티에도 반영이 된다.  
###### CustomOAuth2UserService 클래스까지 생성되었다면 OAuthAttributes 클래스를 생성한다.  
```java
package com.zzangho.project.springboot.config.auth.dto;

import com.zzangho.project.springboot.domain.user.Role;
import com.zzangho.project.springboot.domain.user.User;
import lombok.Builder;
import lombok.Getter;

import java.util.Map;

@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture) {
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {

        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                              .name((String) attributes.get("name"))
                              .email((String) attributes.get("email"))
                              .picture((String) attributes.get("picture"))
                              .attributes(attributes)
                              .nameAttributeKey(userNameAttributeName)
                              .build();
    }

    public User toEntity() {
        return User.builder()
                    .name(name)
                    .email(email)
                    .picture(picture)
                    .role(Role.GUEST)
                    .build();
    }
}

```

###### config.auth.dto 패키지에 SessionUser 클래스를 추가한다.  

```java
package com.zzangho.project.springboot.config.auth.dto;

import com.zzangho.project.springboot.domain.user.User;
import lombok.Getter;

import java.io.Serializable;

@Getter
public class SessionUser implements Serializable {
    private String name;
    private String email;
    private String picture;

    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}

```

###### SessionUser에는 인증된 사용자 정보만 필요하다. 그 외에 필요한 정보들은 없으니 name, email, picture만 필드로 선언한다.  

###### 스프링 시큐리티가 잘 적용되었는지 확인하기 위해 화면에 로그인 버튼을 추가해 보자.  
###### index.mustache에 로그인 버튼과 로그인 성공 시 사용자 이름을 보여주는 코드이다.  

```html
    <h1>스프링 부트로 시작하는 웹 서비스</h1>

    <div class="col-md-12">
        <div class="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
                \{\{\#name\}\}
                    {{name}}
                    <a href="/logout" class="btn btn-info active" role="button">Logout</a>
                {{/name}}
                {{^name}}
                    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
                    <a href="/oauth2/authorization/naver" class="btn btn-secondary active" role="button">Naver Login</a>
                {{/name}}
            </div>
        </div>
        <br>
        <!-- 목록 출력 영역 -->
        ...
```

###### index.mustache에서 userName을 사용할 수 있게 IndexController에서 userName을 model에 저장하는 코드를 추가한다.  
```java
import javax.servlet.http.HttpSession;

@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession  httpSession;

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());
        SessionUser user = (SessionUser) httpSession.getAttribute("user");
        
        if ( user != null ) {
            System.out.println(user.getName());
            model.addAttribute("name", user.getName());
        }
        return "index";
    }
```

###### 그럼 한번 프로젝트를 실행해서 테스트를 해보자. http://localhost:8080으로 접속을 해보면 Google Login 버튼이 보일 것이다.  
###### 클릭해 보면 평소 다른 서비스에서 볼 수 있던 것처럼 구글 로그인 동의 화면으로 이동하며 본인의 계정을 선택하면 로그인 과정이 진행되어 화면에 이름이 노출이 된다.  

---

###### 다음으로는 어노테이션 기반으로 개선하기를 해보겠다.  

###### 일반적인 프로그래밍에서 개선이 필요한 나쁜 코드에는 어떤 것이 있을까?  
###### 가장 대표적으로 같은 코드가 반복되는 부분이다.  
###### 자 그럼 앞서 만든 코드에서 개선할만한 것은 무엇이 있을까? 바로 IndexController에서 세션값을 가져오는 부분이라고 생각한다.

```java
SessionUser user = (SessionUser) httpSession.getAttribute("user");
```

###### 그래서 이 부분을 메소드 인자로 세션값을 바로 받을 수 있도록 변경해 보자.  
###### config.auth 패키지에 다음과 같이 @LoginUser 어노테이션을 생성한다.  

```java
package com.zzangho.project.springboot.config.auth;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

// 이 어노테이션이 생성될 수 있는 위치를 지정한다.
// PARAMETER로 지정했으니 메소드의 파라미터로 선언된 객체에서만 사용할 수 있다.
// 이 외에도 클래스 선언문에 쓸 수 있는 TYPE 등이 있다.
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {   //이 파일을 어노테이션 클래스로 지정한다. LoginUser라는 이름을 가진 어노테이션이 생성됬다고 보면 된다.
}

```

###### 그리고 같은 위치에 LoginUserArgumentResolber를 생성한다. LoginUserArgumentResolver라는 HandlerMethodArgumentResolver 인터페이스를 구현한 클래스이다.  
###### HandlerMethodArgumentResolver는 한가지 기능을 지원한다. 바로 조건에 맞는 경우 메소드가 있다면 HandlerMethodArgumentResolver의 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있다.  

```java
package com.zzangho.project.springboot.config.auth;

import com.zzangho.project.springboot.config.auth.dto.SessionUser;
import lombok.RequiredArgsConstructor;
import org.springframework.core.MethodParameter;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;

import javax.servlet.http.HttpSession;

@RequiredArgsConstructor
@Component
// HandlerMethodArgumentResolver는 조건에 맞는 경우 메소드가 있다면
// HandlerMethodArgumentResolver의 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있다.
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {

    private final HttpSession httpSession;

    // 컨트롤러 메소드의 특정 파라미터를 지원하는지 판단한다.
    // 여기서는 파라미터에 @LoginUser 어노테이션이 붙어 있고, 파라미터 클래스 타입이 SessionUser.class인 경우 true를 반환한다.
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());

        return isLoginUserAnnotation && isUserClass;
    }

    // 파라미터에 전달할 객체를 생성한다. 여기서는 세션에서 객체를 가져온다.
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) throws Exception {

        return httpSession.getAttribute("user");
    }
}

```

###### @LoginUser를 사용하기 위한 환경은 구성이 되었다.  
###### 자 이제 이렇게 생성된 LoginUserArgumentResolver가 스프링에서 인식될 수 있도록 WebMvcConfigurer에 추가하겠다. config 패키지에 WebConfig 클래스를 생성하여 다음과 같이 설정을 추가 한다.  

```java
package com.zzangho.project.springboot.config;

import com.zzangho.project.springboot.config.auth.CertificationInterceptor;
import com.zzangho.project.springboot.config.auth.LoginUserArgumentResolver;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.List;

@RequiredArgsConstructor
@Configuration
public class WebConfig implements WebMvcConfigurer {
    private final LoginUserArgumentResolver loginUserArgumentResolver;

    // HandlerMethodArgumentResolver는 항상 WebMvcConfigurer의 addArgumentResolvers를 통해 추가해야 한다.
    // 다른 Handler-MethodArgumentResolver가 필요하다면 같은 방식으로 추가해 주면 된다.
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(loginUserArgumentResolver);
    }
}

```

###### 모든 설정이 끝났으니 처음 언급한 대로 IndexController의 코드에서 반복되는 부분들을 모두 @LoginUser로 개선하자.  

```java
import javax.servlet.http.HttpSession;

@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession  httpSession;

    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user) {
        model.addAttribute("posts", postsService.findAllDesc());
        
        if ( user != null ) {
            System.out.println(user.getName());
            model.addAttribute("name", user.getName());
        }
        return "index";
    }
```

---

###### 다음으로는 세션 저장소로 데이터베이스를 사용해보자.  

+ spring-session-jdbc 등록  

###### 먼저 build.gradle에 다음과 같이 의존성을 등록한다.  

```properties
compile('org.springframework.session:spring-session-jdbc')  // 세션 저장소 - 데이터베이스
```

###### 그리고 application.properties에 세션 저장소를 jdbc로 선택하도록 코드를 추가한다.  
```properties
spring.session.store-type=jdbc
```

---

###### 마지막으로 네이버 로그인을 추가해 보자.  
###### 네이버도 구글과 같이 https://developers.naver.com/apps/#/register?api=nvlogin 으로 접속하여 client ID와 client Secret을 발급 받는다.  

###### 해당 키값들을 application-oauth.properties에 등록한다. 네이버에서는 스프릴ㅇ 시큐리티를 공식 지원하지 않기 때문에 전부 수동으로 입력해야 한다.  

```properties
spring.security.oauth2.client.registration.naver.client-id=jEI3rFr_p1HraumkBayY
spring.security.oauth2.client.registration.naver.client-secret=fDekPwBXcU
spring.security.oauth2.client.registration.naver.redirect-uri={baseUrl}/{action}/oauth2/code/{registrationId}
spring.security.oauth2.client.registration.naver.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.naver.scope=name,email,profile_image
spring.security.oauth2.client.registration.naver.client-name=Naver
spring.security.oauth2.client.provider.naver.authorization-uri=https://nid.naver.com/oauth2.0/authorize
spring.security.oauth2.client.provider.naver.token-uri=https://nid.naver.com/oauth2.0/token
spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
# 기준이 되는 user_name의 이름을 네이버에서는 response로 해야 한다. 이유는 네이버의 회원 조회 시 반환되는 JSON 형태 때문이다.
spring.security.oauth2.client.provider.naver.user-name-attribute=response
```

+ user_name_attribute=response
  - 기준이 되는 user_name의 이름을 네이버에서는 response로 해야 한다.
  - 이유는 네이버의 회원 조회 시 반환되는 JSON 형태 때문이다.  

###### 스프링 시큐리티에선 하위 필드를 명시할 수 없다. 최상위 필드들만 user_name으로 지정 가능하다. 하지만 네이버의 응답값 최상위 필드는 resultCode, message, response 이다.  ###### 이러한 이유로 스프링 시큐리티에서 인식 가능한 필드는 저 3개 중에 골라야 한다. 본문에서 담고 있는 response를 user_name으로 지정하고 이후 자바 코드로 response의 id를 user_name으로 지정하겠다.  

###### OAuthAttributes에 다음과 같이 네이버인지 판단하는 코드와 네이버 생성자를 추가하자.  

```java
package com.zzangho.project.springboot.config.auth.dto;

import com.zzangho.project.springboot.domain.user.Role;
import com.zzangho.project.springboot.domain.user.User;
import lombok.Builder;
import lombok.Getter;

import java.util.Map;

@Getter
public class OAuthAttributes {
    ...

    // OAuth2User에서 반환하는 사용자 정보는 Map이기 때문에 값 하나하나를 변환해야 한다.
    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        if ("naver".equals(registrationId)) {
            return ofNaver("id", attributes);
        }

        return ofGoogle(userNameAttributeName, attributes);
    }

    ...

    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes) {
        Map<String, Object> response = (Map<String, Object>) attributes.get("response");
        return OAuthAttributes.builder()
                .name((String) response.get("name"))
                .email((String) response.get("email"))
                .picture((String) response.get("profile_image"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    ...
}

```

###### 마지막으로 index.mustache에 네이버 로그인 버튼을 추가한다.  

```html
...
        {{^name}}
            <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
            <a href="/oauth2/authorization/naver" class="btn btn-secondary active" role="button">Naver Login</a>
        {{/name}}
    </div>
</div>
```

###### 다시 어플리케이션을 실행하고 접속을 해보면 네이버 로그인 버튼이 보일 것이다. 해당 버튼을 클릭하면 네이버 로그인 성공!  
