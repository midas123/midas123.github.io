---
  layout: single
  title: Springboot-Hibernate 개인 프로젝트

---

이전에 진행한 팀 프로젝트에서 spring과 mybatis를 기반으로 XML 설정 파일로 ORM을 세팅해서   사용해본 경험이 있습니다. 

당시 mybatis를 사용하면서 ORM과 hibernate에 대해서 알게 되었습니다. hibernate가 해외에서는 널리 사용되고 있고 국내에서도 많이 늘고 있는 것으로 알고 있습니다. 그래서 작은 개인 프로젝트를 진행하며 공부해보았습니다. 

이전에 mybatis로 진행한 프로젝트에서는 xml 설정 파일에 데이터베이스에 접근하는 쿼리를 작성하여 데이터베이스 위주로 기능하는 프로젝트를 만들었다면, 이번 개인 프로젝트에서는 hibernate를 이용하여 전자와 상반되는 구조로 만들게 되었습니다. 

자바 클래스에서 entity 객체를 설정하고 DB에 접근하는 코드를 작성해야 했기 때문에 처음 습득하는 단계에서 어려움이 많았습니다.

먼저 관련 이론을 상세하게 공부하기 보다는 springboot과 hibernate 기반의 예제 코드를 찾아보고 그 안에 있는 각 어노테이션과 클래스 기능을 파악하고 프로젝트에 적용해보며 공부하였습니다.

<br><br>

## INDEX

- **프로젝트 개요**

  [프로젝트 구조](#프로젝트-구조)

  [처리흐름](#처리-흐름)

  [개체 관계도](#개체-관계도)

- **구현 기능**

  [스프링 시큐리티 설정](#스프링-시큐리티-설정)

  [회원가입](#회원가입)

  [회원가입-유효성 검사(Validation)](#회원가입-유효성-검사) 

  [비밀번호 변경 - 이메일 token 발송](#비밀번호-변경-이메일-token-발송)

<br><br><br>

## 프로젝트 구조

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\springboot\project-structure.jpg){: .center-image }





<br><br>

## 처리 흐름

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\springboot\springboot-flowchart.jpeg){: .center-image }

<br><br>

## 개체 관계도

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\springboot\springboot_20190429_11_30.jpg){: .center-image }

<br><br>



## 스프링 시큐리티 설정



스프링 시큐리티 필터 설정으로 ROLE이 없는 사용자는 css, js, 이미지 파일 등의 리소스와 특정 페이지만 접근이 가능하도록 설정했습니다. 

사용자 로그인 기능은 스프링 시큐리티가 처리합니다. UserDetailsService 인터페이스를 구현한 CustomUserDetailsService 클래스의 loadUserByUsername() 메서드에서 사용자의 계정과 이메일 계정 인증 여부를 확인 후 반환한 CustomUserDetails 객체에 의해서 스프링 시큐리티 로그인 과정이 진행됩니다.

<br>

*-WebSecurityConfig*

```java
@Configuration
@EnableWebSecurity
@ComponentScan(basePackageClasses = CustomUserDetailsService.class)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	 
	 @Autowired
	 private UserDetailsService userDetailsService;
	 
	 @Autowired
	 public void configAuthentication(AuthenticationManagerBuilder auth) throws Exception {
	  auth.userDetailsService(userDetailsService).passwordEncoder(passwordencoder());
	 } 
	
	@Override
    protected void configure(HttpSecurity http) throws Exception {
		http.csrf().disable();
		http.authorizeRequests()
        .antMatchers("/css/**", "/js/**", "/img/**", "/login**").permitAll()
        .antMatchers("/emailConfirmation/**", "/ResetPassword/**").permitAll()
        .antMatchers("/registration").hasRole("ANONYMOUS")
        .antMatchers("/board").hasRole("USER").anyRequest().authenticated()
        .and()
		.formLogin()
        .loginPage("/")
        .loginProcessingUrl("/loginPro")
        .defaultSuccessUrl("/board", true)
        .failureUrl("/login?error")
        .usernameParameter("username")
        .passwordParameter("password")
        .permitAll()
        .and()
        .logout()
        .and()
        .exceptionHandling().accessDeniedPage("/403");
	}
	
	@Bean
    public PasswordEncoder passwordencoder() {
        return new BCryptPasswordEncoder();
    }
}
```

<br>

*-CustomUserDetailsService*

```java
@Service("customUserDetailsService")
public class CustomUserDetailsService implements UserDetailsService{
	
	private final UserRepository userRepository;
	private final UserRolesRepository userRolesRepository;
	
	@Autowired
	public CustomUserDetailsService(UserRepository userRepository,UserRolesRepository userRolesRepository) {
	    this.userRepository = userRepository;
	    this.userRolesRepository=userRolesRepository;
	}
	
	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		Users user=userRepository.findByUsername(username);
		if(user != null && user.getIsenabled() == false || user == null) {
			throw new UsernameNotFoundException("아이디 또는 비밀번호를 잘못 입력하셨습니다.");
		} else{
			List<String> userRoles=userRolesRepository.findRoleByUsername(username);
			return new CustomUserDetails(user,userRoles);
		}
	}
}
```

<br><br>

## 회원가입

회원가입 요청시 데이터가 JSON 형식으로 넘어오도록 컨트롤러를 구성했습니다.  userResistrationPro() 메서드에서 사용자가 입력한 이메일, 닉네임 중복 여부를 확인합니다. 그리고 사용자 역할(ROLE)을 부여하고 UserRole 테이블에 별도로 저장합니다.  비밀번호는 암호화되며 다른 사용자 정보와 함께 Users 테이블에 저장됩니다.

@Transactional에 의해서 모든 과정은 일괄 실행됩니다. 마지막에 sendVerifycationEmail() 메서드가 사용자의 이메일 계정을 인증하는 token 이메일을 발송합니다. token은 EmailToken 테이블에 저장 됩니다.

<br>

*-컨트롤러*

```java
@RestController
@AllArgsConstructor
public class WebRestController {
	
	@Autowired
	private UserServiceImpl userServiceImpl;
	
    @PostMapping("/registration")
    public int joinMember(@Valid @RequestBody UserRequestDto UserDto) {
    	return userServiceImpl.userResistrationPro(UserDto);
    }
}
```

<br>

-'UserServiceImpl' 서비스 클래스

```java
@Transactional
public int userResistrationPro(UserRequestDto dto){
		verifyDuplicattionEmail(dto.getUsername());//이메일 중복 확인
		verifyDuplicattionNickName(dto.getNickname());//넥네임 중복 확인
		dto.setPassword(passwordEncoder.encode(dto.getPassword())); //비밀번호 암호화
		int userId = userRepository.save(dto.toEntity()).getUserid();//사용자 정보 DB저장
		UserRoleResistration(userId);//사용자 ROLE DB저장
		sendVerifycationEmail(dto, userId);//사용자 이메일계정 인증메일 발송
		return userId;
}
	
```

<br>

### 회원가입 - 사용자 이메일 계정 인증

confirmEmailToken() 메서드가 사용자의 인증 요청의 파라미터로 넘어온 token이 DB에 저장된 token과 일치하는지 확인한 후 사용자 계정을 활성화 합니다. @Controller 어노테이션을 사용한 컨트롤러이므로 모든 처리 후 활성화 완료 페이지(confirmDone)를 view로 반환합니다.

*-컨트롤러*

```java
  @GetMapping("/emailConfirmation/account")
	  public String confirmUserAccount(@RequestParam("token")String emailToken) {
		  userServiceImpl.confirmEmailToken(emailToken);
		  return "confirmDone";
	  }
```

<br>

*-UserServiceImpl의 UserServiceImpl메서드*

```java
public void confirmEmailToken(String token) {
		EmailToken emailtoken = emailTokenRepository.findByConfirmationToken(token);
        if(emailtoken != null) {
            Users user = userRepository.findByUsernameIgnoreCase(emailtoken.getUser().getUsername());
            user.setIsenabled(true);
            userRepository.save(user);//사용자 계정 활성화
            emailTokenRepository.delete(emailtoken);
        } else {
        	throw new ValidCustomException("인증 주소가 적절하지 않습니다.", "confirm-email-errormessage");
        }
	}
```

<br><br>

### 회원가입 유효성 검사

이전 프로젝트에서 javascript와 jquery를 이용해서 유효성 검사를 구현한 경험이 있습니다. 하지만 웹브라우저에서 javascript가 작동 안되는 경우가 있기 때문에 이번에는 서버 측에서 유효성 검사를 구현하게 되었습니다. 물론 서버와 웹브라우저에서 동시에 유효성검사를 구현하는 방법이 최선 일 것입니다.

회원가입시 사용자가 입력한 정보의 유효성 검사 코드는 UserRequestDto 클래스에 작성했습니다. 클래스의 필드 값은 Users 엔티티로 전달되는 파라미터 입니다. 각 필드마다 유효성 검사를 실행하는 constraint 어노테이션이 붙어있습니다.

몇몇 필드는 기본 제공 되는 어노테이션만으로 유효성 검사를 하기에 부족한 부분이 있기 때문에 별도의 constraint 어노테이션 클래스가 필요해서 따로 추가하게 되었습니다. 

@EmailValid는 username(이메일) 필드의 유효성 검사를 좀 더 정확하게 하며, @PasswordMatch 어노테이션은  사용자가 처음 입력한 비밀번호와 다시 한번 입력한 비밀번호를 비교해서 비밀번호를 정확히 입력했는지 확인하는 기능을 합니다.

<br>

*-UserRequestDto*

```java
@Getter
@Setter
@NoArgsConstructor
@PasswordMatch.List({ 
	@PasswordMatch(field = "password", fieldMatch = "confirmPassword", message = "비밀번호가 서로 다릅니다."), 
})
public class UserRequestDto {
	@NotBlank(message="이메일을 작성해주세요.")
	@EmailValid
	private String username;
	
	@NotBlank(message="닉네임을 작성해주세요.")
	private String nickname;
	
	@NotBlank(message="비밀번호를 작성해주세요.")
	private String password;
	
	@NotBlank(message="비밀번호를 다시 한번 작성해주세요.")
	private String confirmPassword;
	
	@NotBlank(message = "전화번호를 작성해주세요.")
	@Pattern(regexp = "[0-9]{10,11}", message = "10~11자리의 숫자만 입력가능합니다")
	private String phoneNumber;
    
}
```

<br><br>

바로 위에 유효성 검사가 동작하기 전, 사용자가 웹브라우저 화면에서 회원가입 버튼을 누르면 아래 기능이 동작합니다. jquery로 가져온 데이터들을 JSON 데이터 타입으로 만들고 ajax를 이용해서 POST 형식으로 서버측으로 보냅니다.

<br>

*-main.js*

```javascript
   join : function () {
        var data1 = {
            username: $('#username').val(),
            nickname: $('#nickname').val(),
            phoneNumber: $('#phoneNumber').val(),
            password: $('#password').val(),
            confirmPassword: $('#password2').val()
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

<br>

만약 회원가입시 입력한 데이터가 유효성 검사 조건에 맞지 않을 경우에는 위 ajax의 .fail() 메서드가 서버측으로 부터  response 객체를 받습니다. 이 객체는 필드 네임과 에러 코드, 메시지 등을 담고 있는 error 객체가 있습니다.

아래 코드에서 그 error 객체의 내용을 확인해서 에러 메세지를 html 형태로 사용자 화면에 추가합니다.

<br>

```javascript
var markingErrorField = function (response) {
    const errorFields = response.responseJSON.errors;
    if(!errorFields){
        alert(response.response.message);
        return;
    }

    var $field, error;
    for(var i=0, length = errorFields.length; i<length;i++){
        error = errorFields[i];
        $field = $('#'+error['field']);
        if($field && $field.length > 0){
        	$('#'+error['field']+'-error').remove();
            $field.after('<span class="error-message" id="'+error['field']+'-error">'+error.defaultMessage+'</span>');
        }
        if(error['code'] == "PasswordMatch"){
        	$('#password2-error').remove();
        	$('#password2').after('<span class="error-message" id="password2-error">'+error.defaultMessage+'</span>');
        }
        if(error['code'] == "confirm-email-errormessage"){
        		alert(error.defaultMessage);
        }
    }
    return false;
};
```

<br><br>

## 비밀번호 변경 이메일 token 발송

회원 가입시 입력한 이메일을 비밀번호 변경 요청 페이지에서 입력하면 이메일 검증 후 token을 발송합니다. 이 후 사용자가 token이 포함된 링크를 클릭하면 token의 유효성을 서버에서 확인하고 비밀번호를 변경 페이지로 이동합니다.

아래 코드에서 각 메서드의 기능을 순서대로 설명하겠습니다. PasswordReset() 메서드에서 이메일과 token의 유효성 검사 결과를 view로 출력합니다.

유효성 검사가 실행되는 곳은 PasswordResetPro() 메서드 입니다. 이메일 주소를 검사하고 만약 error 메세지가 있을 경우 view로 반환합니다. 아무 문제가 없을 경우에는 정상적으로 token 생성과 이메일 발송을 처리합니다. 

PasswordChangeForm()에서 token의 유효성을 검사하며, 이상이 없을 경우 비밀번호 변경 페이지를 반환합니다. token은 30분의 유효기간을 가지며 그 이후에는 해당 token으로 비밀번호를 변경할 수 없습니다.

PasswordTokenResetPro()에서는 DB에 저장되어 있는 유효한 token으로 사용자 정보에 접근하여 비밀번호 변경을 처리합니다.

<br>

*-PasswordResetController*

```java
 	@GetMapping("/ResetPassword")
    public String PasswordReset(@ModelAttribute("pfdto") PasswordResetRequestDto pfdto, String 			mailSended, Model model) {
	  	if(mailSended != null) {
	  		model.addAttribute("mailSended", true);
	  	}
        return "passwordResetForm";
    }
    
    @PostMapping("/ResetPassword/verificationMail")
    public String PasswordResetPro(@ModelAttribute("pfdto") @Valid PasswordResetRequestDto pfdto,
                                            BindingResult result, Errors errors) {
        if (result.hasErrors()){
            return "passwordResetForm";
        }
        Users user = userRepository.findByUsername(pfdto.getEmail());
        if (user == null){
            result.rejectValue("email", null, "이메일 주소가 존재하지 않습니다.");
            return "passwordResetForm";
        }
        //token 생성 후 DB 저장
        PasswordResetToken token = new PasswordResetToken();
        token.setToken(UUID.randomUUID().toString());
        token.setUser(user);
        token.setExpiryDate(30);
        tokenRepository.save(token);
		
        //메일 발송
        SimpleMailMessage mailMessage = new SimpleMailMessage();
        mailMessage.setTo(user.getUsername());
        mailMessage.setSubject("비밀번호 변경 이메일 입니다.");
        mailMessage.setFrom("admin@gmail.com");
        mailMessage.setText("링크를 클릭해서 비밀번호를 변경합니다..: "
        +"http://localhost:8080/ResetPassword/valid?token="+token.getToken());
        emailSendService.sendEmail(mailMessage);
        return "redirect:/ResetPassword/?mailSended";
    }
    
    @GetMapping("/ResetPassword/valid")
    public String PasswordTokenResetForm(@ModelAttribute("prdto") PasswordResetDto prdto, 
    		@RequestParam(required = false) String token, Model model) {
        PasswordResetToken resetToken = tokenRepository.findByToken(token);
        if (resetToken == null){
            model.addAttribute("error", "유효한 token이 아닙니다.");
        } else if (resetToken.isExpired()){
            model.addAttribute("error", "token의 유효기간이 종료되었습니다.");
            tokenRepository.delete(resetToken);
        } else {
            model.addAttribute("token", resetToken.getToken());
        }
        return "passwordResetToken";
    }
    
    @PostMapping("/ResetPassword/pro")
    @Transactional
    public String PasswordTokenResetPro(Model model, @ModelAttribute("prdto") @Valid 				PasswordResetDto prdto, BindingResult result, Errors errors) {
    	PasswordResetToken token = tokenRepository.findByToken(prdto.getToken());
    	if(token == null) {
    		model.addAttribute("error", "유효한 token이 아니므로 비밀번호를 변경 할 수 없습니다.");
    		return "403";
    	}
        if (result.hasErrors()){
        	model.addAttribute("token", prdto.getToken());
        	return "passwordResetToken";
        }
        Users user = token.getUser();
        String updatedPassword = passwordEncoder.encode(prdto.getPassword());
        userRepository.updatePassword(updatedPassword, user.getUserid());
        tokenRepository.delete(token);
        return "redirect:/ResetPassword/done";
    }
    
    @GetMapping("/ResetPassword/done")
    public String passwordCheangeDone() {
        return "passwordResetDone";
    }
```

<br><br>

## 결론

어플리케이션 동작 중 만들어진 데이터가 데이터베이스에 저장되도록 변환해주는 영속화 프레임 워크 중 하나인 mybatis를 처음 접하였습니다. 그래서 sql 위주의 xml 설정에 먼저 익숙해진 상태였습니다. 

hibernate는 mybatis와 상반되게 기본적인 sql 쿼리를 일일이 작성할 필요없이 간단한 자바 코드 한 줄이면 해결되고 DB에 따라서 sql 문법이나 관련 설정을 바꿀 필요가 없는 편리함이 있었습니다. 

하지만 초기 설정시 작성해야 하는 자바 코드가 생소해서 적응하는게 쉽지 않았습니다. 그럼에도 불구하고 작은 프로젝트를 차근차근 진행하면서 각 어노테이션과 클래스를 공부하고 기능을 하나둘씩 추가하면서 오류를 수정하는 경험을 통해서 공부하면서 알게 된 hibernate 장점들에 대해 어느정도 공감할 수 있었습니다.  









