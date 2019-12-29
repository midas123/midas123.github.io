---
 layout: single
 title: Web-Scraper
 tag: [springboot, jpa, hibernate, gradle, maria-db, jsoup]
 kinds: 프로젝트
 player: solo
 detail: 스크랩한 웹 사이트의 게시글 검색
 toc: true
 toc_sticky: true
---

웹 페이지에서 게시글 제목과 url을 parsing하고 토큰화하여 DB에 저장하고 사용자가 그 내역을 키워드로 검색할 수 있도록 구현했습니다. 이 어플리케이션은 하나 이상의 웹 페이지에서 가져온 다수의 게시글을 한번에 검색 할 수 있는게 특징입니다.

jsoup 라이브러리를 사용하였으며 사이트 마다 URL 형태가 다르기 때문에 웹 페이지를 parsing하는 메서드의 코드는 조금씩 달라야 했습니다. 그래서 WebScraper 추상 클래스를 상속 받고 메서드를 오버라이드해서 웹 페이지의 URL 구조의 맞게 parsing하도록 구현했습니다.

테이블은 items와 itemindex 2개로 구성했습니다. items에서는 게시글 제목과 URL을 저장하고 itemindex 테이블은 게시글 제목을 문자열로 토큰화하여 별도로 저장합니다. 사용자가 검색한 키워드를 토큰 테이블(itemindex)에서 매칭하고 참조키를 이용해서 items 테이블에서 게시글과 URL을 찾아 사용자에게 출력합니다. 

[프로젝트-github-링크](https://github.com/midas123/yk-toy-project)

<br>

## 개발환경

- springboot
- jpa - hibernate
- gradle
- maria-db(구글 클라우드)

<br>

## ERD

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\0602\web-scraper_20190604_13_59.jpg){: .center-image }

<br>

## 클래스 구조



엔티티 - Items, ItemIndexes

서비스 - ItemService, WebScrapService

추상 클래스 - WebScraper

DAO - ItemIndexRepository, ItemRepository

DTO - ItemRequestDto

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\0602\web-scraper-classes.jpg){: .center-image }

<br>

## FLOW CHART

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\0602\web-scraper-flow.png){: .center-image }

<br>

##  사용자 화면

'java regular expression' 키워드로 검색한 결과 입니다.

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\0602\web-scraper-page.jpg){: .center-image }

<br>

## 사용 기술

### jsoup-JAVA-library

jsoup 자바 라이브러리를 이용하여 웹 페이지를 parsing하고 글 제목과 URL을 담고 있는 html 엘리먼트를 추출했습니다. 그리고 엘리먼트에서 특정 데이터를 꺼내기 위해서 자바 정규 표현식을 사용하였습니다.

```java
	@Override
	public Elements getCategoryNameAndLink(List<String> links) {
		Elements categoryElements = new Elements();
		for(String s: links) {
			try {
				Document document = Jsoup.connect(s).get();
				categoryElements = document.select("li a[href~=\\/(.)[/.*(?i)spring.*|.*(?i)java.*]]");
			} catch (IOException e) {
				e.printStackTrace();
				System.err.println(e.getMessage());
			}
		}
		return categoryElements;
	}
```

<br>

### Tokenization

parsing한 html 엘리먼트의 url을 토큰화 했습니다. 예를 들어 url이 'https:// ...../java/java-regular-expression/' 일때 tokenizer() 메서드로 'java, regular, expression'의 문자열을 token으로 추출합니다.

```java
	@Override
	public String tokenizer(String link) throws MalformedURLException {
		String path = new URL(link).getPath();
		String token ="";
		if(path.length()>0) {
			String title = path.replaceFirst("/", "").replaceAll("/", "-");
			String[] tokens = title.trim().split("-");
			
			Set<String> set = new LinkedHashSet <>(); 
			for(String str: tokens) {
				set.add(str);
			}
			
			tokens = set.toArray(new String[set.size()]);
			token = String.join(",", tokens);
		}
		return token;
	}
```

<br>

### 추상 클래스

웹 사이트마다 URL 구조 약간씩 다르기 때문에 문자열로 parsing 하는 과정도 조금씩 달라야 했습니다. 그래서 구현해야 하는 추상 메서드를 포함한 WebScraper 추상클래스를 정의하고 자식 클래스에서 메서드를 작성했습니다.

```java
public abstract class WebScraper {
	abstract List<String> setLinks(String url);
	
	abstract Elements getCategoryNameAndLink(List<String> links);
	
	abstract List<ItemRequestDto> getArticleTitleAndLink(Elements categoryLinks);
	
	abstract String tokenizer(String link) throws MalformedURLException;
}
```

