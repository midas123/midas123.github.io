---
 layout: single
 title: 스프링시큐리티 - 필터 설정
 tag: [spring, spring-security, security-filter]
 kinds: 포스트
 toc: true
 toc_sticky: true
---







스프링 시큐리티 내에서 요청에 대한 인증과 접근을 관리하는 시프링 시큐리티 필터에 대해 간단하게 알아보았습니다.

## The Security Filter Chain

Spring Security’s web infrastructure는 서블릿 필터 기반입니다. 필터는 내부적으로 서블릿을 사용하거나 서블릿 기반에 프레임워크(ex. Spring MVC)를 사용하지 않습니다. 그러므로 특정 웹 기술과 깊게 연관되지 않습니다.

그리고 시큐리티 필터는 HttpServletRequest와 HttpServletResponse를 다룹니다. request가 어디에서 오는지는 상관하지 않습니다. (웹 브라우저, 웹  서비스 클라이언트, HttpInvoker, AJAX 등)

스프링 시큐리티는 내부적으로 필터 체인을 유지합니다. 그리고 각 필터마다 역할이 주어져 있고 configuration에 의해서 추가 또는 제거 됩니다. 필터 간에 의존이 있기 때문에 순서가 매우 중요합니다. xml 네임 스페이스 설정을 사용한다면 필터는 자동으로 설정되므로 따로 스프링 빈을 정의할 필요 없습니다. 하지만 네임스페이스는 시큐리티 필터를 완전히 관리할 수 없고 커스텀 설정을 지원하지 않습니다.

<br>

## DelegatingFilterProxy

서블릿 필터를 사용하려면 web.xml 필터를 설정해야 합니다. 그렇지 않으면 서블릿 컨테이너는 필터를 무시합니다. 모든 필터 클래스는 스프링 빈으로 다른 스프링 빈처럼 스프링 의존 주입과 라이프 사이클 혜택을 받습니다.

DelegatingFilterProxy는 web.xml과 ApplicationContext 사이에 연결을 제공합니다. ApplicationContext는 컨테이너 내의 스프링 빈의 초기화, 설정 등을 책임지는 스프링 컨테이너를 나타냅니다.

```xml
<filter>
<filter-name>myFilter</filter-name>
<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
<filter-name>myFilter</filter-name>
<url-pattern>/*</url-pattern>
</filter-mapping>
```

DelegatingFilterProxy는 실제로 필터의 로직을 실행하지 않습니다. 스프링 어플리케이션 컨텍스트에 있는 빈을 통해서 필터의 메서드를 위임합니다.

<br>

## FilterChainProxy

스프링 시큐리티 웹 기반은 오직 FilterChainProxy 인스턴스 위임에 의해서만 사용되어야 합니다. 스프링 시큐리티 필터 자체를 사용해서는 안됩니다. 예를 들어 웹 어플리케이션 컨텍스트 파일에 각각의 스프링 시큐리티 필터 빈을 순서까지 고려해서 선언하고 각 필터를 일치시키는 설정을 web.xml에 추가한다면 필터가 많아지면 많아 질수록 매우 혼란 스러울 것 입니다.

FilterChainProxy는 하나의 entry를 web.xml에 추가해서 어플리케이션 전체의 웹 시큐리티 빈을 관리하도록 합니다.

```xml
<bean id="filterChainProxy" class="org.springframework.security.web.FilterChainProxy">
<constructor-arg>
    <list>
    <sec:filter-chain pattern="/restful/**" filters="
        securityContextPersistenceFilterWithASCFalse,
        basicAuthenticationFilter,
        exceptionTranslationFilter,
        filterSecurityInterceptor" />
    <sec:filter-chain pattern="/**" filters="
        securityContextPersistenceFilterWithASCTrue,
        formLoginFilter,
        exceptionTranslationFilter,
        filterSecurityInterceptor" />
    </list>
</constructor-arg>
</bean>
```

네임 스페이스 엘리먼트 중 filter-chain은 어플리케이션에 필요한 시큐리티 필터 체인을 만드는데 사용됩니다. 특정 URL 패턴을 필터가 담긴 리스트에 맵핑 설정 합니다. 

그러면 런타임에서 FilterChainProxy가 현재 웹 요청에 일치하는 URI 패턴의 위치를 파악하고 해당되는 필터 리스트가 그 요청에 적용됩니다. 필터는 정의된 순서로 호출됩니다.

<br>

## 필터 우회하기

filters ="none" 속성을 사용하면 해당 URI 패턴에 대한 요청을 스프링 시큐리티 필터를 거치지 않게 됩니다.인증이나 권한 부여 절차를 거치지 않고 자유롭게 접근 가능해 집니다. 

요청을 처리하는 동안 SecurityContext의 컨텐츠를 사용하고 싶다면 꼭 시큐리티 필터를 거치게 설정해야 합니다. 만약 그렇게 하지 않는다면 SecurityContextHolder는 컨텐츠가 없는 null 상태가 됩니다.

<br>

출처:https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#security-filter-chain