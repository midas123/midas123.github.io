---
 layout: single
 title: 스프링부트 JPA 이미지 파일 업로드
---

사용자가 회원가입 화면에서 프로필 이미지를 업로드 하는 기능을 추가하였습니다. 

 



## ERD

이미지 테이블 칼럼은 아래와 같이 구성했습니다.

- 파일ID

- 파일 확장자

- 파일 원본 이름

- 파일 저장 이름

- USERID(참조키)

  

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\0514\spring-jpa_20190514_17_14.png){: .center-image }





application.yml 설정 추가

```
spring:
  servlet:
    multipart:
      enabled: true
      file-size-threshold: 2KB
      max-file-size: 200MB
      max-request-size: 220MB
```



파이

