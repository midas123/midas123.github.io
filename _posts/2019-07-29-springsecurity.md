---
 layout: single
 title: 스프링시큐리티 - authentication
 tag: [spring, spring-security, authentication]
 kinds: 포스트
 toc: true
 toc_sticky: true
---

프로젝트에서 간단하게 스프링-시큐리티를 적용해본 경험이 있습니다. 별도의 서비스 클래스를 만들고 UserDetailsService 인터페이스를 구현하여 loadUserByUsername()메서드에서 사용자가 입력한 정보와 DB에 있는 사용자 정보를 비교해서 유효성을 확인하도록 구현했었는데 인증 과정에 대해서 좀 더 구체적으로 공부하고자 스프링 문서를 보고 일부 내용을 정리했습니다.

<br>

## SecurityContextHolder

SecurityContextHolder는 가장 기본적인 객체 입니다. 어플리케이션을 이용중인 principal을 포함하는 security context가 SecurityContextHolder에 저장 됩니다. SecurityContextHolder는 ThreadLocal를 디폴트로 사용하여 정보를 저장합니다. 그러므로 실행 중인 스레드 내에서는 항상 사용 가능합니다. 요청이 처리된 후 이 스레드는 자동으로 정리되므로 이 방법은 안전합니다.

{: .notice--info} 

※만약 어플리케이션에서 스레드로컬을 사용할 수 없는 경우 SecurityContextHolder의 strategy 속성을 변경해야 합니다.



SecurityContextHolder에 저장된 principal은 Authentication 오브젝트를 이용해서 가져올 수 있습니다. 어플리케이션의 어느 부분에서드 아래 코드 블럭으로 principal 정보에 접근합니다.

```java
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

if (principal instanceof UserDetails) {
String username = ((UserDetails)principal).getUsername();
} else {
String username = principal.toString();
}
```

<br>

## UserDetailsService

위 코드처럼 Authentication 객체에서 principal을 얻을 수 있습니다. 그리고 principal 객체는 UserDetails 객체로 캐스팅 될 수 있습니다. UserDetails 스프링 시큐리티에 코어 인터페이스 입니다. principal를 나타내며(대신하며), 확장성과 어플리케이션에 특화된 성질을 가지고 있습니다. UserDetails는 어플리케이션의 유저 데이터베이스와 이것을 필요로 하는 SecurityContextHolder 사이에 있는 아답터라고 볼 수도 있습니다. 데이터 베이스에서 사용자 정보를 나타낼때 UserDetails를 어플리케이션의 객체로 casting할 수도 있습니다. 그러면 get메서드로 같은 business-specific 메서드를 호출할 수 있습니다.

UserDetailsService 인터페이스는 loadUserByUsername()메서드에서 String username을 인자로 받아서 UserDetails 객체를 리턴합니다. UserDetails 객체가 필요한 곳에 제공하려면 아래 코드를 사용합니다.

```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```

<br>

인증(authentication)과정에서 UserDetails는 SecurityContextHolder에 저장되는 Authentication 객체를 만드는데 사용됩니다.

그리고 UserDetailsService는 in-memory map 또는 JDBC 등을 이용한 다양한 실행을 제공합니다. 하지만 대부분의 개발자는 어플리케이션의 엔티티 객체를 표현하는 DAO 객채 위에서 UserDetailsService를 구현합니다. 그리고 이 인터페이스가 리턴하는 객체는 SecurityContextHolder에서 항상 획득할 수 있다는 것을 기억해야 합니다.

{: .notice--info} 

※UserDetailsService는 사용자 데이터를 위한 DAO로서 프레임워크 내에 다른 컴포넌트로 이 데이터를 제공하는 기능만 합니다. AuthenticationManager처럼 유저를 인증하지는 않습니다. (커스텀 인증 과정은 AuthenticationProvider를 직접 구현해서 만듭니다.)

<br>

## GrantedAuthority

Authentication이 principal과 함께 제공하는 메서드로 getAuthorities()가 있습니다. 이 메서드는 principal에 부여된 권한을 가지고 있는 GrantedAuthority 배열 객체를 제공합니다. GrantedAuthority는 principal에 부여된 역할(role)으로 스프링 시큐리티에서 이 역할을 분석합니다. 이 역할은 이후에 웹, 메서드, 도메인 객체 등에 대한 권한으로 설정됩니다. 그리고 GrantedAuthority 객체는 UserDetailsService에 의해 로드됩니다.

<br>

## 요약

{: .notice--danger}

- SecurityContextHolder는 SecurityContext에 접근을 제공한다.
- SecurityContext는 Authentication과 요청에 대한 보안 정보를 저장한다.
- Authentication은 principal을 나타낸다.
- GrantedAuthority는 principal에 부여된 어플리케이션 범위를 갖는 권한을 반영한다.
- UserDetails는 Authentication 객체를 만드는데 필요한 정보를 제공한다. 그 정보는 어플리케이션 DAO 또는 다른 보안 데이터에서 가져온다.
- UserDetailsService는 String타입의 username을 받아서 UserDetails 객체를 리턴한다.



<br><br>

## Authentication

Spring Security는 다양한 인증 환경에 사용될 수 있습니다. Spring Security를 독립적으로 사용하는 것을 권장하지만 Authentication을 관리하는 다른 컨테이너와 integration 또한 지원합니다.

**일반적인 Authentication 절차**

{: .notice--info} 

1. 사용자가 username과 password를 입력

2. 시스템에서 username과 password 정확한지 확인

3. 사용자 권한을 포함한 사용자 정보를 획득

4. 사용자에 대한 security context를 성립

5. 사용자의 security context 정보를 토대로 접근을 관리하는 메커니즘으로 특정 기능에 대한 접근을 보호함

   

**위 인증 절차 1~3번에 상세 과정(스프링시큐리티 내부)**

{: .notice--info} 

1. 사용자의 username과 password를 UsernamePasswordAuthenticationToken 인스턴스에 저장
2. 토큰을 validation하기 위해 AuthenticationManager 인스턴스에 전달
3. 인증이 성공하면 AuthenticationManager는 Authentication 인스턴스를 리턴
4. SecurityContextHolder.getContext().setAuthentication(…)를 호출해서 security context를 성립

이 과정은 보통 내부적으로 발생합니다. SecurityContextHolder가 제대로 된 Authentication 객체를 보관해야 사용자는 인증 됩니다.

<br>

## 직접 SecurityContextHolder 컨텐츠 세팅하기

스프링 시큐리티는 Authentication 객체를 SecurityContextHolder에 어떤 방식으로 넣는 방식은 신경쓰지 않습니다. 다만 AbstractSecurityInterceptor가 사용자 권한을 확인하기 전, SecurityContextHolder는 principal을 represent하는 Authentication을 가지고 있어야 합니다.

스프링 시큐리티 기반이 아닌 인증 시스템과 상호 운용성(interoperability)을 위해서 직접 필터 또는 MVC 컨트롤러를 작성할 수 있습니다. 예를 들어, 컨트롤 권한이 거의 없는 회사의 레거시 인증 시스템을 그대로 가져오는 경우도 스프링 시큐리티를 쉽게 적용할 수 있습니다.

써드 파티 사용자 정보를 읽는 필터를 작성하고 Authentication 객체를 만듭니다. 그리고 SecurityContextHolder에 집어넣습니다. 이 경우, 자동적으로 처리되는  built-in authentication infrastructure에 대해서도 신경써야 합니다. 예를 들어, request 사이에 conext를 캐시하기 위해서  HTTP session을 미리 만들어야 할지도 모릅니다. (사용자에게 response 보내기 직전 그리고 보낸 후에는 세션을 만들 수 없음)



<br>

## 일반적인 웹 어플리케이션 내에서 요청 인증 과정

{: .notice--info}

1. 홈페이지에서 링크를 클릭
2. 요청이 서버로 전달되고 서버는 보호되는 리소스에 대한 요청인지 확인

3. 인증이 안된 상태라면 서버는 인증 안내를 response로 사용자에게 보냄. response는 HTTP code이거나 웹 페이지 리다이렉팅
4. 인증 메커니즘에 따라서 사용자는 리다이렉팅 되어서 form을 작성하게 됨. 또는 쿠키, 인증서, 기본적인 인증 박스로 사용자의 identity를 확인
5. 웹 브라우저는 다시 서버로 요청을 보냄. 작성한 form을 HTTP POST에 담거나 HTTP 헤더에 인증 정보를 담음.
6. 다음으로 서버는 credentials이 유효한지 확인하게 됨. 만약 유효하면 다음 단계가 진행되고 아니라면 2번부터 다시 진행하게 됨.

7. 인증 절차를 마친 최초 요청은 재인증을 받게 됨. 이때 보호 중인 리소스에 대한 사용자의 권한이 충분한지 확인하고 권한이 낮거나 없다면 사용자는 HTTP error code 403을 받게 됨.

위 과정에서 주요한 역할을 하는 스프링 시큐리티 클래스들이 있습니다. 주요한 클래스로는 ExceptionTranslationFilter, AuthenticationEntryPoint, AuthenticationManager을 호출하는 인증 메커니즘

<br>

## ExceptionTranslationFilter

스프링 시큐리티에서 발생하는 exception을 감지하는 역할을 하는 필터 입니다. (이러한 익셉션은 일반적으로AbstractSecurityInterceptor에 의해 발생합니다.) 

ExceptionTranslationFilter는 유저가 인증은 되었지만 접근 권한이 없을 경우 HTTP 403코드를 리턴하고 인증이 먼저 필요할 경우 AuthenticationEntryPoint를 실행합니다.

<br>

## AuthenticationEntryPoint

위 과정에서 3번째 단계를 담당합니다. 모든 인증 시스템은 AuthenticationEntryPoint를 갖게 됩니다.

<br>

## 인증 메커니즘

사용자가 웹 브라우저에서 HTTP form이나 헤더를 통해서 credentials을 제출하고 나면 서버에서는 이 인증 정보를 수집해야 합니다. 위 과정에서 6번째 입니다. 사용자에게서 인증 정보를 수집하면 Authentication 객체가 만들어집니다. 그리고 AuthenticationManager로 전달됩니다. 

인증 메커니즘이 완전한 Authentication 객체를 받게되면 request가 유효한 것으로 간주합니다. 그리고 Authentication을 SecurityContextHolder에 넣습니다. 그리고 최초 요청을 재인증하여 권한을 확인합니다. AuthenticationManager가 요청을 거절하면 사용자는 다시 인증을 시도 해야 합니다.

<br>

## Requests 사이에 SecurityContext를 저장

어플리케이션 타입에 따라서 SecurityContext를 저장하는 전략이 필요할 수 있습니다. 일반적인 웹 앱에서는 사용자가 한번 로그인 한 다음에는 세션id로 인증을 받게 됩니다. 서버는 세션 시간 동안 principal 정보를 캐싱합니다.

스프링 시큐리티에서는 요청 간에 SecurityContext 저장하는 책임이 SecurityContextPersistenceFilter에 있습니다. HTTP requests 사이에서 context를 HttpSession 속성으로 저장합니다.

각 요청 시에 context를 SecurityContextHolder에 복구하고, 요청이 완료되면 SecurityContextHolder를 clear 합니다. 보안을 위해서는 HttpSession와 직접 상호작용 해서는 안됩니다. 항상 SecurityContextHolder를 사용해야 합니다.

RESTful 웹 서비스에서는 HTTP session을 사용하지 않고 매 요청마다 재인증을 하게 됩니다. 그러나 SecurityContextPersistenceFilter는 인증 체인 안에 포함되어 있어 여전히 중요합니다. 매 요청 후 SecurityContextHolder가 clear 되도록 해야합니다.

<br>

출처: https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#technical-overview



