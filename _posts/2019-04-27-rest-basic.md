---
  layout: single
  title: REST API 기본 개념
---

REST(**Re**presentational **S**tate **T**ransfer)는 HTTP 프로토콜을 사용하는 웹 서비스 어플리케이션의 아키텍쳐 스타일이다. RESTful 웹 서비스는 REST를 베이스로 하는 웹 서비스를 말한다.

이 아키텍처의 특징은 리소스 중심으로 구성되며 서버가 클라이언트에게 오직 리소스에 대한 접근만을 제공하는 것이다. 

<br>

## Resource

리소스는 REST에서 중요한 개념이다.  문서, 이미지, 일시적인 서비스, 어떤 리소스 그룹, 유저ID 혹은 사람 같이 실재하고 이름 지을 수 있는 모든 것이 리소스가 될 수 있다. 그리고 이런 리소스를 구분하고 확인하기 위해서 URI를 사용한다. 

<br>

##  HTTP methods

REST API에서 클라이언트 측 요청에 주로 사용되는 http 메서드로는 GET/POST/DELETE/PUT 등이 있다.

<br>

## REST 6가지 원칙

Restful한 웹 서비스를 만들기 위해서 지켜야하는 원칙이다.

<br>

**Uniform interface** : 리소스 중심으로 인터페이스를 정해야 한다. 웹 서비스 내에서 리소스는 각각 하나의 URI를 가져야 한다. 또한 웹 서비스의 resource representation들은 네이밍 컨벤션, 링크 포맷, 데이타 포맷 등 특정한 지침을 따라야 한다. 모든 리소스는 HTTP GET으로 접근할 수 있어야 한다.

※[representation](https://stackoverflow.com/questions/33706191/what-is-the-difference-between-resource-and-resource-representation-in-rest)은 서버가 응답으로 보내는 리소스가 JSON/XML 등의 형태로 표현되는 것이다.

<br>

**Client-server**: 클라이언트와 서버가 상호 간의 의존 없이 분리되어 있어야 한다. 서로 분리되어 있으므로 언제든지 개별적으로 개발되거나 교체될 수 있어야 한다.

<br>

**Stateless**: 서버는 클라이언트의 최근 HTTP request에 대한 어떤한 것도 저장해서는 안된다. 모든 요청은 이전에 발생한 요청과 관계 없이 새롭다.(No session, no history)
어플리케이션의 state에 대한 책임은 서버가 아닌 클라이언트에게 있다.

<br>

**Cacheable**: 모든 자원은 캐싱을 적용하거나 하지 않을 수 있는데 REST API에서는 가능하다면 캐싱을 적용해야 한다. 서버는 캐싱의 적용 여부를  HTTP 헤더에서 지시할 수 있다. 또한 캐싱은 서버 또는 클라이언트에서 실행될 수 있다. 캐싱을 사용하므로써 클라이언트 측에서는 불필요한 요청 제거로 인한 성능 향상을, 서버 측은 확장성을 높일 수 있다.

※캐싱: 최초 요청시 리소스의 복사본을 저장하고 재요청 시에 리소스 대신 복사본을 사용하는 것.

<br>

**Layered system**: 계층 구조를 이루는 시스템을 사용할 수 있다. 예를 들어 어플리케이션 구동 서버, 리소스 저장 서버, 인증 요청 처리하는 서버를 나눈다. 클라이언트 측에서는 어느 서버에 연결되었는지 알 수 없어야 한다.

<br>

**Code on demand**(선택적):서버는 클라이언트에게 대부분 XML/JSON 형태의 정적인 자원을 응답으로 보낸다. 하지만 필요할 경우 어플리케이션을 지원하는, 실행 가능한 코드를 보낼 수 있다.

<br>

**참고**:

[rest-architectural-constraints](https://restfulapi.net/rest-architectural-constraints/#uniform-interface)

<br>

