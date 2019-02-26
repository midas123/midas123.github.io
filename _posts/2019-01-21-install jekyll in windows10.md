---
  layout: single
  title: 윈도우10에서 jekyll 설치하기
  tag: [jekyll]
---



## 윈도우에서 jekyll 설치

1.루비인스톨러 설치 (2.4 버전이상)

[루비 인스톨러 다운로드]: https://rubyinstaller.org/downloads/

   ※gem은 프로그래밍 언어 루비에 패키지 매니져

2.윈도우 실행창에서 'cmd' 입력해서 명령 프롬프트 실행 후 설치 여부 확인

```
ruby -v
gem -v
```

3.jekyll 설치 및 확인

```
gem install jekyll bundler
jekyll -v
```



## jekyll로 static 사이트 생성 및 실행

1.기본 static 사이트 생성(실행위치 C:\Users\username)

```
jekyll new "사이트명"
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/post/0121/jek-start3.jpg)

2.사이트 구동 (실행위치 C:\Users\username\사이트명)

```
cd "사이트명"

bundle exec jekyll serve
```

3.웹브라우저에서 로컬호스트 URL로 사이트에 접속

![]({{ site.url }}{{ site.baseurl }}/assets/images/post/0121/jek-start2.jpg)



※사이트 구동 중 컨버전 에러 발생시 

![]({{ site.url }}{{ site.baseurl }}/assets/images/post/0121/jek-start1.jpg)

아래 명령어 입력

```
chcp 65001
```

여기까지 진행되면 기본적인 레이아웃만 있는 사이트를 로컬 환경에서 실행할 수 있습니다. 이제 마음에 드는 테마를 설치하고 자기 입맛대로 사이트를 커스텀 하면 됩니다. 테마를 설치하기 전에 front matter에 대해 알아야 합니다.

------



## Front matter 

jekyll을 시작하면서 가장 먼저 알아야 할 것 입니다. html, css 등을 직접 작성하지 않고 이것만 잘 설정해줘도 그럴듯한 사이트가 완성 됩니다.front matter는 yaml 또는 json 형태의 데이터 입니다. "키 값": "밸류" 형태로 사이트에 설정이 담겨 있습니다. html 또는 md 파일 상단에 위치 합니다.  아래 처럼 예시 처럼 "---"으로 감싸줍니다.

```
---
layout: post
title:  "Welcome to Jekyll!"
date:   2019-01-21 12:57:18 +0900
categories: jekyll update

---

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.
```

여기서 layout, title, date, categories 등 다양한 설정을 합니다.
예를 들어, 마크다운 작성한 포스트의 레이아웃은  여기서 설정한 post로 보여집니다.
html 또는 md(마크다운 파일)은 텍스트나 코드 형태의 컨텐츠를 담고 있고 설정된 layout이 씌워진 형태가 최종적으로 웹브라우저에 출력됩니다.

------



## 사이트 구조(폴더 구조)

![]({{ site.url }}{{ site.baseurl }}/assets/images/post/0121/jek-start4.jpg)



**_posts**: 마크다운으로 작성한 포스트를 여기에 저장합니다.

**_site**: 사이트 구동시 생성되는 폴더 입니다. 사이트의 최종 버전 이므로 건드릴 필요없습니다. 

**_config.yml:** 사이트의 모든 설정

**gemfile:** 사이트의 디펜던시 설정(theme 설정)

------



## 참고

[jekyll 공식 resources](https://jekyllrb.com/resources/)

[minimal-mistakes 테마 가이드 문서](https://mmistakes.github.io/minimal-mistakes/docs/configuration/#)

