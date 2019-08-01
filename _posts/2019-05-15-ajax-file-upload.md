---
 layout: single
 title: ajax로 파일 업로드하기
 kinds : 프로젝트
 player: solo
 tag: [java, springboot, jpa, hibernate, ajax, maven]
 toc: true
 toc_sticky: true
---

기존 [프로젝트](https://midas123.github.io/2019/04/08/Springboot-Hibernate-project.html)에서 회원가입 시 이미지 파일을 업로드 할 수 있도록 수정하였습니다. 파일 정보를 저장하는 엔티티를 추가하고 파일 이름을 변경하여 경로에 저장하는 코드를 추가하였습니다. 그리고 ajax 코드에서 컨텐츠 타입을 multipart/form-data으로 변경하였습니다.



## 개발환경

- springboot 2.1.3
- jpa
- maven
- java8

<br>

## 변경 사항

- UserImages 엔티티, DTO, DAO, 서비스 클래스 추가
- 자바 클래스에서 WebMvcConfigurer를 구현 후 업로드 파일이 저장될 경로와 url 맵핑을 설정
- application.yml에 업로드 파일 사이즈 설정 추가
- html 페이지에 form 태그 안에 file 타입의 input 태그 추가
- ajax 코드 수정
- 컨트롤러에 UUID 클래스를 이용한 저장용 파일 이름 생성 코드 추가

<br>



## UserImages 엔티티, DTO, DAO, 서비스 클래스

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\0514\spring-jpa_20190514_17_14.png){: .center-image }

UserImages 엔티티의 userid는 참조키이며 Users 엔티티의 userid 칼럼과 1:1 맵핑 관계 입니다.

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class UserImages {
	@Id
	@GeneratedValue(strategy= GenerationType.AUTO)
	private long image_id;
	
	@Column(length=20, nullable=false)
	private String file_type;
	
	@Column(length=100, nullable=false)
	private String file_origin_name;
	
	@Column(length=100, nullable=false)
	private String file_save_name;
	
	@Lob
	private byte[] file_data; //파일을 DB에 저장할때 사용하는 칼럼
	
	@OneToOne
	@JoinColumn(name="userid")
	private Users users_images;
	
    //@Builder 코드 생략..
}
```

<br>

DTO 클래스

```java
@Getter
@Setter
@NoArgsConstructor
public class UserImagesRequestDto {
	private long image_id;
	
	private String file_type;
	
	private String file_origin_name;
	
	private String file_save_name;
	
	private Users users_images;
	
	private MultipartFile file_data;
	
	//@Builder 코드 생략
	
}
```

<br>

DAO

```java
@Repository
public interface UserImagesRepository extends JpaRepository<UserImages, Long>{
}
```

<br>

서비스 클래스 - 파일 정보를 DB에 저장하고 실제 파일을 지정한 경로에 저장

```java
@Service
public class UserImageService {
	private UserImagesRepository userImagesRepository;
	
	public UserImageService(UserImagesRepository userImagesRepository) {
		this.userImagesRepository = userImagesRepository;
	}
	
	public void saveFile(UserImagesRequestDto dto) throws IOException {
		dto = saveFileToPath(dto);
		userImagesRepository.save(dto.toEntity());
	}
	
	private UserImagesRequestDto saveFileToPath(UserImagesRequestDto dto) throws IOException{
		String ex_uploadPath = "C:\\Users\\yk1\\Desktop\\upload2\\images\\";
		byte[] bytes = dto.getFile_data().getBytes();
		Path path = Paths.get(ex_uploadPath+dto.getFile_save_name());
		Files.write(path, bytes);
		dto.setFile_data(null);
		return dto;
	}
}
```

<br>

## 자바 클래스에서 WebMvcConfigurer를 구현 후 업로드 경로 맵핑

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
    	String in_path = "classpath:/static/"; //프로젝트 내부 css, js
    	String ex_path = "file:///C:/Users/yk1/Desktop/upload2/images/";//업로드 파일 저장경로(프로젝트 외부)
       registry
               .addResourceHandler("/**", "/images/**").addResourceLocations(in_path, ex_path);
    }
} 
```

<br>

## form 태그 안에 file 타입의 input 태그 추가

```html
<form id="form-data" enctype="multipart/form-data" method="post">
//...
   <div class="member-form-item">
        <label for="file">프로필 이미지</label>
        <input type="file" name="file_data"/>
   </div>
//...
</form>
```

<br>

## ajax 코드 수정

기존 ajax 코드는 아래와 같습니다. form 태그 안에 input 태그 값을 jquery 셀렉터로 가져와서 객체에 담은 후 JSON.stringify()로 JSON 문자열로 변환하고 ajax 요청을 보냅니다.

```javascript
 join : function () {
        var data1 = {
            username: $('#username').val(),
            nickname: $('#nickname').val(),
            phonenumber: $('#phonenumber').val(),
            password: $('#password').val(),
            confirmpassword: $('#password2').val()
        };
        $.ajax({
            type: 'POST',
            url: '/registration',
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data1)
        }).done(function() {
            alert('회원 가입 되었습니다. 회원 이메일을 인증해주세요.');
            location.href="/";
        }).fail(function (response) {
        	markingErrorField(response);
        });
    }
```

주요한 변경사항으로는 먼저 enctype을 multipart/form-data으로 변경해야 합니다. 참고로 enctype 기본 값은 application/x-www-form-urlencoded으로 위에선 코드를 생략 됩니다. 그리고 data1 객체를 JSON 문자열로 바꿔주는 JSON.stringify() 메서드를 제거해야 합니다.

변경 된 ajax 코드는 아래와 같습니다. FormData 객체를 이용해서 form 태그에서 제출한 데이터를 ajax 요청으로 보냅니다.

```javascript
joinFileUpload : function(){
    	   var form = $('#form-data')[0];
    	   var data = new FormData(form);

    	    $.ajax({
    	        type: "POST",
    	        enctype: 'multipart/form-data',
    	        url: "/registration",
    	        data: data,
    	        processData: false, 
    	        contentType: false
    	    }).done(function() {
                alert('회원 가입 되었습니다. 회원 이메일을 인증해주세요.');
                location.href="/";
    	    }).fail(function (response) {
            	markingErrorField(response);
            });
    }
```

<br>

## UUID 클래스를 이용한 저장용 파일 이름 생성

UUID 객체는 고유한 식별자로 사용되는 숫자를 생성합니다. 고유한 번호로 파일명을 만들어서 업로드 파일 이름이 중복되는 것을 방지했습니다.

```java
@PostMapping("/registration")
public long userRegistration(@Valid UserRequestDto userDto) throws IOException {

    MultipartFile file = dto.getFile_data();
    String fileOriginName = file.getOriginalFilename();
	UUID uuid = UUID.randomUUID();
    dto.setFile_save_name(uuid.toString()+"_"+fileOriginName);
    //이하 생략    
    }
```

