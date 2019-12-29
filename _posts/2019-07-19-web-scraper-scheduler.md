---
 layout: single
 title: 웹 어플리케이션 내에 스케줄러 구현
 tag: [springboot, Timer]
 kinds: 포스트
---

스케줄러 API를 이용해서 [WebScraper](/2019/06/02/web-scraper.html) 프로젝트에 새로운 기능을 추가했습니다. 이전에는 사용자가 버튼을 눌러 수동으로 특정 사이트를 스크랩 하는 방식이었는데 스케줄러 기능을 추가하여 일정한 시간마다 사이트를 스크랩 하도록 구현했습니다.

'스케줄러'로 검색해보니 java.util.Timer 클래스와 스프링에서 제공하는 TaskExecutor, TaskScheduler를 찾을 수 있었습니다. 그리고 프로젝트에 어느 부분에 이 코드를 구현하고 어떻게 동작하는지  알아보았습니다. 

스케줄러는 구동 중인 어플리케이션 서버의 백그라운드 환경에서 동작해야 합니다. 스프링에 TaskExecutor, TaskScheduler 인터페이스는 비동기적 실행과 스케줄링을 제공하는 abstraction 입니다. 

또한 스프링은 java.util.Timer 또는 [Quartz Scheduler](https://www.quartz-scheduler.org/)를 이용한 스케줄링을 지원합니다. 이 외부 클래스 사용하려면 스케줄링 클래스에서 FactoryBean 인터페이스를 구현해야 합니다.

스케줄러를 구현한 클래스에 @Component을 붙여 스프링 컨테이너에 빈을 등록하면 클래스에 작성한 스케줄링이 서버의 백그라운드에서 동작하게 됩니다. 스케줄링 클래스 추가과정은 아래와 같습니다.

## 개발환경

- springboot 2.1.5
- jpa - hibernate
- gradle
- maria-db(구글 클라우드)

<br>

첫번째, @EnableScheduling추가합니다. 만약 비동기적 실행을 허용하려면 @EnableAsync을 추가합니다.

```java
@SpringBootApplication
@EnableScheduling
public class SpringWebScraperApplication {
	
	public static void main(String[] args) {
		SpringApplication.run(SpringWebScraperApplication.class, args);
	}

}
```

<br>

두번째, 스케줄링을 실행할 클래스를 작성하고 @Component로 스프링 컨테이너에 등록합니다. 값을 리턴하지 않는 메서드에 @Scheduled을 추가하고 매일 0시에 실행되도록 cron을 설정해줍니다. zone을 빈칸으로 두면 서버 타임존이 그대로 적용됩니다. 

```java
@Component
public class WebScrapScheduler {
	@Autowired
	WebScrapService webScrapService;
	
	@Scheduled(cron="0 0 0 * * *", zone ="")
	public void ScrapingWeb() {
		webScrapService.scrapArticles();
	}
	
}
```

정말 간단하게 스케줄링을 설정해서 매일 scrapArticles()가 실행되도록 구현했습니다.



<br>



※스레드풀: 미리 초기화된 하나 이상의 스레드가 주어진 작업을 수행하기전 idle 상태로 대기 하는 곳



https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#scheduling

https://blog.outsider.ne.kr/1066

https://softwareengineering.stackexchange.com/questions/173575/what-is-a-thread-pool

  