---
 layout: single
 title: 스프링부트 RESTful 웹 서비스
 tag: [java, rest-api, springboot, maven]
 kinds: 프로젝트
 player: solo
 detail: 게시글 조회/쓰기/수정/삭제/추천/댓글
 toc: true
 toc_sticky: true
---

앞서 스프링부트 + 스프링시큐리티를 사용해서 회원가입/로그인을 구현하였고 이어서 REST api 구조의 기본적인 게시판을 구현했습니다. 이번 프로젝트의 주제는 기본적인 게시판으로 REST API의 원칙을 준수하는 RESTful 웹 서비스의 백엔드 서버를 만드는데 집중했습니다.

게시판에서 사용자가 요청하게될 자원은 게시글(post)이 입니다. 그래서 post를 기준으로 일관적인 식별자 주소를 만들었습니다. post에 접근하는 주소는 기본적으로 '/post/{post_id}' 형태이고 게시글 조회/쓰기/수정/삭제/추천 등의 기능의 구분은 http 메서드(GET,POST,PUT,DELETE)를 이용하였습니다. 그리고 요청한 자원만을 리턴해주는 백엔드 서버로서 뷰 페이지를 리턴하지 않고 json 타입의 텍스트 데이터만 리턴합니다. 

또한, spring HATEOAS를 사용해서 최초 요청의 응답을 받은 사용자가 자연스럽게 다음 자원으로 이동하도록 했습니다. 

[프로젝트-github-링크](https://github.com/midas123/spring-board-jpa)

<br>

# 개발환경

------



- IDE - 이클립스
- 운영체제 - 윈도우10
- 테스트 도구 - postman
- java8 / springboot 2.1.3 / jpa(hibernate) / maven

<br>

# ERD

------



**테이블 구성**

테이블은 POSTS(게시글) / POSTCOMMENT(댓글) / POSTLIKES(추천) 총 3개로 구성하였습니다. 

<br>

**테이블 관계**

게시글과 댓글 테이블  -> 1:N 관계

게시글, 댓글 테이블과 추천 테이블 -> 각각 1:N 관계

게시글과 댓글 테이블의  post_id와 com_id 칼럼은 PK 입니다. 추천 테이블에서 FK 칼럼으로 참조합니다. 또한 댓글 테이블은 post_id를 FK 칼럼으로 참조합니다.

<br>

**POSTCOMMENTS의 일부 칼럼 설명**

- COM_GROUP_SEQ: 게시글 내에서 부모 댓글의 순번 입니다.

- COM_RE_SEQ: 부모 댓글 그룹에 속하는 자식 댓글의 순번 입니다.

- COM_DEPTH:  UI 화면에서 부모 댓글과 자식 댓글을 구분할때 사용합니다.

  <br>

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\0511\spring-jpa_20190511_37_29.png){: .center-image }





<br>

# 클래스 구조

------

**Entity**: Post, PostComments, PostLikes

**DTO**: PostRequestDto, PostCommentRequestDto 등..

**DAO**: PostRepository, PostCommentRepository 등..

**Service**: PostService, PostCommentService

그외 컨트롤러와 커스텀 익셉션 클래스, ResourceAssembler 클래스가 있습니다.



![]({{ site.url }}{{ site.baseurl }}\assets\images\post\0511\class-a.png){: .center-image }

<br>

# 프로젝트 기능 요약

------



- 게시판 쓰기/수정/삭제

- 댓글 수정/삭제/댓글 답변

- 게시글/댓글 추천(중복 추천 방지)

  

<br>

# 프로젝트 사용 기술 

------

**목차**

1. [RESTful 컨트롤러](#컨트롤러)
2. [Spring-HATEOAS](#스프링-라이브러리)
3. [Entity / DTO / DAO / Service layer](#프로젝트-계층-구조)
4. [커스텀 익셉션 & Optional](#커스텀-익셉션-옵셔널)

------

<br>

<br>

## 컨트롤러

RESTful한 웹 서비스 구조를 갖추기 위해서 게시글 자원의 주소를 'post'로 동일하게 구성하고 HTTP 메서드(GET/POST/PUT/DELETE) 맵핑을 나누었습니다. 클라이언트 요청시 메서드 종류에 따라서 다른 처리가 진행됩니다. 

```java
@RestController
public class PostRestController {
	@Autowired
	private PostService postService;
	
	@Autowired
	PostResourceAssembler assembler;
	
	@GetMapping("/post/all")
	public Resources<Resource<Posts>> postList(){
		List<Resource<Posts>> posts = postService.getAllPost().stream()
				.map(assembler::toResource).collect(Collectors.toList());
		
		return new Resources<>(posts,
				linkTo(methodOn(PostRestController.class).postList()).withSelfRel());
	}
	
	@GetMapping("/post/{post_id}")
	public Resource<Posts> getOnePost(@PathVariable long post_id) {
		Posts post = postService.getOnePost(post_id);
		Resource<Posts> resource = new Resource<Posts>(post);
		Link link = new Link("http://localhost:8080/post/all");
		resource.add(link);
		return resource;
	}
	
	@PostMapping("/post")
	public String WritePost(@RequestBody PostRequestDto dto) {
		postService.writePost(dto);
		return "PostDone";
	}
	
	@PutMapping("/post/{post_id}")
	public String updatePost(@PathVariable long post_id, @RequestBody PostRequestDto dto) {
		postService.updatePost(post_id, dto);
		return "PostUpdated";
	}
	
	@DeleteMapping("/post/{post_id}")
	public String deletePost(@PathVariable long post_id) {
		postService.deletePost(post_id);
		return "PostDeleted";
	}
	
	@PutMapping("/post/like")
	public String likedPost(@Valid @RequestBody PostLikeRequestDto dto) {
		System.out.println("controller:"+dto.toString());
		postService.likePost(dto);
		return "PostLiked";
	}
}
```

<br>

## 스프링 라이브러리

게시글을 조회한 사용자가 바로 다음 자원을 이용하도록 응답 객체에 다른 자원의 url을 포함하였습니다. 응답 URL을 생성을 지원하는 **Spring HATEOAS**를 사용하였습니다. 

아래처럼 ResourceAssembler를 구현하고 오버라이드한 toResource()에서 url를 작성하는 클래스를 만든 후

```java
import org.springframework.hateoas.Resource;
import org.springframework.hateoas.ResourceAssembler;
import org.springframework.stereotype.Component;
import static org.springframework.hateoas.mvc.ControllerLinkBuilder.*;

import com.yk.web.post.entity.Posts;

@Component
public class PostResourceAssembler implements ResourceAssembler<Posts, Resource<Posts>>{
	@Override
	public Resource<Posts> toResource(Posts entity) {
		
		 return new Resource<>(entity,
			      linkTo(methodOn(PostRestController.class).getOnePost(entity.getPost_id())).withSelfRel(),
			      linkTo(methodOn(PostRestController.class).postList()).withRel("all"));
	}
}
```

PostRestController에서 이 클래스를 의존 주입하고 postList()가 엔티티 객체(Posts)를 랩핑하는  Resource 객체를 리턴합니다.

```java
@Autowired
PostResourceAssembler assembler;
	
@GetMapping("/post/all")
public Resources<Resource<Posts>> postList(){
		List<Resource<Posts>> posts = postService.getAllPost().stream()
				.map(assembler::toResource).collect(Collectors.toList());
		
return new Resources<>(posts,
				linkTo(methodOn(PostRestController.class).postList()).withSelfRel());
	}
```

위 toResource() 메서드가 리턴한 결과는 아래와 같은 형태로 응답 객체에 담깁니다.

```json
 "_links": {
        "self": {
            "href": "http://localhost:8080/post/all"
        }
    }
```

http://localhost:8080/post/282로 요청한 결과는 아래와 같습니다.

```json
{
    "createdDate": "2019-05-12T10:15:42.525",
    "modifiedDate": "2019-05-12T22:00:47.403",
    "post_id": 282,
    "nickname": "yk",
    "p_title": "제목2",
    "p_content": "내용2",
    "p_counts": 1,
    "postLikes": [
        {
            "like_id": 287,
            "likes": 1,
            "kinds": "post",
            "nickname": null
        }
    ],
    "comments": [],
    "_links": {
        "self": {
            "href": "http://localhost:8080/post/all"
        }
    }
}
```

<br>

# 프로젝트 구조 

------



## Entity 클래스

컬럼 이름과 제약조건과 엔티티 관계를 설정했습니다. lombok 어노테이션을 활용하여 getter/setter 및 빌더 관련 자바코드를 줄였습니다. 

```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Entity
public class Posts extends BaseTimeEntity{
	@Id
	@GeneratedValue(strategy = GenerationType.AUTO)
	private long post_id;
	
	@Column
	private String nickname;
	
	@Column(length=100)
	private String p_title;
	
	@Column
	private String p_content;
	
	@Column
	private long p_counts;

	@JsonManagedReference 
	@OneToMany(mappedBy = "post", fetch = FetchType.LAZY)
	private List<PostLikes> postLikes = new ArrayList<>();

	@JsonManagedReference 
	@OneToMany(mappedBy = "post", fetch = FetchType.LAZY)
	@OrderBy("com_group_seq asc, com_re_seq asc")
	private List<PostComments> comments = new ArrayList<>();
	
	@Builder
	public Posts(long post_id, String nickname, String p_title, String p_content, long p_counts,
			List<PostLikes> postLikes, List<PostComments> comments) {
		this.post_id = post_id;
		this.nickname = nickname;
		this.p_title = p_title;
		this.p_content = p_content;
		this.p_counts = p_counts;
		this.postLikes = postLikes;
		this.comments = comments;
	}
```

<br>



## DTO(Data Transfer Object)

클라이언트의 요청 데이터를 맵핑하는 RequestDto 객체에서 파라미터의 validation을 실행하고 엔티티 객체로 변환 합니다.

```java
@Getter
@Setter
@NoArgsConstructor
public class PostRequestDto {
	private long post_id;
	
	@NotBlank
	private String nickname;
	
	@NotBlank
	private String p_title;
	
	@NotBlank
	private String p_content;

	private long p_counts;
	
	private long likes;
	
	public Posts toEntity() {
		return Posts.builder()
				.nickname(nickname)
				.p_content(p_content)
				.p_title(p_title)
				.p_counts(p_counts)
				.build();
	}
	
}

```

<br>



## DAO(Data Access Object)

JpaRepository를 상속하는 PostRepository 인터페이스를 DB에 접근하는 DAO로 사용하였습니다. 간단한 엔티티 객체 저장은 JpaRepository의 save() 메서드를 활용하고 JOIN 문법이나 WHERE 조건 등이 필요한 경우는 @Query 어노테이션으로 쿼리를 만들어서 사용했습니다.

```java
@Repository
public interface PostRepository extends JpaRepository<Posts, Long>{
    
@Modifying
@Query("UPDATE Posts SET p_counts = p_counts + 1 WHERE post_id = ?1")
void updatePostCounts(long post_id);
	
@Modifying
@Query("UPDATE Posts t SET t.p_title= :p_title, t.p_content= :p_content WHERE post_id = :post_id")
public void updatePostTitleAndContent(@Param("post_id") long post_id, @Param("p_title") String p_title, @Param("p_content") String p_content);
	
@Query(value="SELECT * FROM Posts p LEFT JOIN (SELECT post_id, SUM(likes) AS likes FROM PostLikes GROUP BY post_id) b ON b.post_id = p.post_id ORDER BY p.post_id DESC", nativeQuery=true)
public List<Posts> getAllPost();
	
@Query(value="SELECT * FROM Posts p LEFT JOIN (SELECT post_id, SUM(likes) AS likes FROM PostLikes GROUP BY post_id) l ON p.post_id = l.post_id WHERE p.post_id = :post_id", nativeQuery=true)
public Posts getOnePost(@Param("post_id") long post_id);
	
}
```

<br>



## Service-layer

서비스 레이어는 DTO 객체를 엔티티 객체로 변환하고 DAO 메서드를 실행해서 DB에 저장하는 코드와 서비스 로직을 실행하는 코드로 구성되어 있습니다.

```java
@Service
public class PostService {
	@Autowired
	private PostRepository postRepository;
	
	@Autowired
	private PostLikeRepository postLikeRepository;
	
	//게시글 쓰기
	public void writePost(PostRequestDto dto) {
		postRepository.save(dto.toEntity());
	}
	
	//모든 게시글 목록 조회
	public List<Posts> getAllPost(){
		List<Posts> pl = postRepository.getAllPost();
		for(int i=0; i<pl.size(); i++) {
			Posts post = pl.get(i);
			List<PostLikes> postlikelist = post.getPostLikes();
			if(postlikelist.size()>0) {
				postlikelist = Posts.addAllLikes(postlikelist, "post");
				post.setPostLikes(postlikelist);
			}
			if(post.getComments() != null) {
				List<PostComments> com = post.getComments();
				for(int j=0; j<com.size(); j++) {
					if(com.get(j) != null) {
						List<PostLikes> ps = Posts.addAllLikes(post.getComments().get(j).getPostLikes(), "comment");
						com.get(j).setPostLikes(ps);
					}
				}
			}
		}
		return pl;
	}
	
	//게시글 조회
	@Transactional
	public Posts getOnePost(long post_id) {
		updatePostHits(post_id);
		Posts post = postRepository.getPost(post_id);
		
		if(post.getPostLikes() != null) {
			List<PostLikes> ps = Posts.addAllLikes(post.getPostLikes(), "post");
			post.setPostLikes(ps);
		}
		if(post.getComments() != null) {
			List<PostComments> com = post.getComments();
			for(int i=0; i<com.size(); i++) {
				if(com.get(i) != null) {
					List<PostLikes> ps = Posts.addAllLikes(post.getComments().get(i).getPostLikes(), "comment");
					com.get(i).setPostLikes(ps);
				}
			}
		}
		return post;
	}
	
	//게시글 조회수+
	private void updatePostHits(long post_id) {
		postRepository.updatePostCounts(post_id);
	}
	
	//게시글 수정
	@Transactional
	public void updatePost(long post_id, PostRequestDto dto) {
		postRepository.updatePostTitleAndContent(post_id ,dto.getP_title() ,dto.getP_content());
	}
	
	//게시글 삭제
	public void deletePost(long post_id) {
		postRepository.deleteById(post_id);
	}
	
	//게시글 추천
	@Transactional
	public void likePost(PostLikeRequestDto dto) {
		long postid = dto.getPost_id();
		isLikedBefore(postid, dto.getNickname());
		dto.setPost(new Posts(postid));
		dto.setKinds("post");
		
		if(dto.getIsLikeUP() == true) {
			dto.setLikes(1);
			postLikeRepository.save(dto.toEntity());
		}	
		if(dto.getIsLikeUP() == false) {
			dto.setLikes(-1);
			postLikeRepository.save(dto.toEntity());
		}
	}
	
	//중복 추천 방지
	private void isLikedBefore(long post_id, String nickname) {
		Optional<PostLikes> PostLikesOp = postLikeRepository.isLikedCheck(post_id, nickname);
		if(PostLikesOp.isPresent()) {
			throw new PostLikeException("이미 추천한 글 입니다.");
		}
	}
}
```

<br>

## 커스텀 익셉션 옵셔널

서비스 레이어에서 사용자의 닉네임으로 중복 추천 여부를 확인하고 만약 중복 추천일 경우, BAD_REQUEST 상태코드와 함께 메세지를 리턴하는 PostLikeException 클래스가 실행되도록 하였습니다.

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class PostLikeException extends RuntimeException{
	public PostLikeException(String message) {
		super(message);
	}
}
```

*-PostService 서비스 클래스*

DAO 역할을 하는 PostLikeRepository에 isLikedCheck() 메서드는 post_id와 nickname으로 추천 테이블을 검색하고 그 결과를 Optional 객체가 저장합니다. 만약 결과가 존재할 경우 isPresent()가 true를 리턴하고   PostLikeException 예외를 발생시킵니다.

```java
	//중복 추천 방지
	private void isLikedBefore(long post_id, String nickname) {
		Optional<PostLikes> PostLikesOp = postLikeRepository.isLikedCheck(post_id, nickname);
		if(PostLikesOp.isPresent()) {
			throw new PostLikeException("이미 추천한 글 입니다.");
		}
	}
```

<br>