---
  layout: single
  title: OFF-THE-BALL 구현기능 설명
---

<br>
## 프로젝트 요약 및 페이지 구성
<br>
첫번째 팀 프로젝트 입니다. 이 프로젝트의 주제는 팀/개인 단위로 참여 가능한 축구경기 스케줄링 및 커뮤니티 사이트 입니다. 
**struts** 2.0 프레임워크를 사용하였고 **javascript**로 유효성검사 기능을 구현했습니다. 

<br>
{% include offtheball-slides.html %}
<br>

제가 팀원으로써 담당한 파트는 아래와 같습니다.

**담당 파트** - 회원가입 / 로그인 / 회원정보 수정 / 관리자 페이지 일부 구현
<br>

## INDEX

- ### 회원 가입/탈퇴  구현

1. [회원가입](#회원-가입)
2. [회원정보 중복 확인](#회원정보-중복-확인)
3. [자바스크립트로 유효성 검사](#자바스크립트로-유효성-검사)
4. [프로필 이미지 업로드](#프로필-이미지-업로드)
5. [회원정보 수정 및 탈퇴](#회원정보-수정-탈퇴)

- ### 로그인  구현

1. [로그인](#로그인)
2. [아이디 찾기](#아이디-찾기)
3. [비밀번호 찾기 - 이메일 전송](#비밀번호-찾기-이메일-전송)   

- ### 관리자 페이지 구현

1. [회원 목록 게시판&검색](#회원-목록-게시판-검색)
2. [회원 정보 상세페이지&회원 등급 변경](#회원-정보-상세페이지-등급변경) 

<br><br><br><br>  

# 회원 가입 페이지 구현
<br>
![]({{ site.url }}{{ site.baseurl }}\assets\images\post\OFFTHEBALL\OFFTHEBALL-membership.jpg){: .center-image }



------



## 회원 가입

form tag로 제출한 회원 정보를 struts의 인터셉터가 memVO(DTO) 객체에 담아줍니다.
getter/setter 메서드로 일일이 DTO 객체에 담는 코드를 줄일 수 있었습니다.

```java
	@Override
	public void prepare() throws Exception {
		memberParam = new memVO();
	}

	@Override
	public memVO getModel() {
		return memberParam;
	}

	public String execute() throws Exception {
		memberParam.setAdmin_yn(genUser);
		memberParam.setM_joindate(m_joindate.getTime());
		sqlMapper.insert("memSQL.insertMem", memberParam);
```
<br><br> 

## 회원정보 중복 확인

회원가입시 사용자가 입력한 ID, 닉네임, 이메일이 중복되는지 확인합니다. DB에서 SELECT 문으로 검색하여 일치하는 값이 없을 경우  getIdcheckresult()는 1을 리턴합니다. idcheckresult의 값에 따라서 웹브라우저에 출력되는 화면이 달라집니다. 닉네임과 이메일 중복 체크도 같은 방식으로 동작합니다.

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\OFFTHEBALL\OFFTHEBALL-checkid.jpg){: .center-image }

```java
public String idcheck() throws Exception {
		memberResult = (memVO) sqlMapper.queryForObject("memSQL.idcheck", memberParam);
		if (memberResult == null)
			idcheckresult = 1;
		return SUCCESS;
	}
```
<br><br> 


## 자바스크립트로 유효성 검사



정규표현식을 사용해서 회원 가입시 입력한 데이터에 대해서 좀 더 정확하게 유효성 검사가 동작하도록 구현했습니다.  

```javascript
function checkIt(){
 	var userinput = eval("document.userinput");
 	//아이디 유효성 검사 정규식
	var idExp = /^[a-zA-Z0-9]{4,12}$/; 
	//이름 유효성 검사 정규식
	var nameExp = /[~!@\#$%<>^&*\()\-=+_\’]/gi;
	//닉네임 유효성 검사 정규식
	var nickExp = /[~!@\#$%<>^&*\()\-=+_\’]/gi;
	//이메일 유효성 검사 정규식
	var emailExp = /^[0-9a-zA-Z]([-_\.]?[0-9a-zA-Z])*@[0-9a-zA-Z]([-_\.]?[0-9a-zA-Z])*\.[a-zA-Z]{2,3}$/i;
	//휴대폰 번호 유효성 검사 정규식
	var phoneExp = /^01([0|1|6|7|8|9]?)-?([0-9]{3,4})-?([0-9]{4})$/;
	//출생연도
	var yearExp =/^[0-9]{4}$/;
	var id = document.userinput.m_id.value;
	var pw = document.userinput.m_passwd.value;
	var name = document.userinput.m_name.value;
	var nick = document.userinput.m_nickname.value;
	 if(!idExp.test(id)){
		alert("아이디는 특수문자/한글/공백은 사용할 수 없습니다.(4자 이상)");
	 	return false;
	 }
	 if(pw.length<8||pw.length>20){
		 alert("비밀번호는 8자리 ~20자리 이내로 입력해주세요.");
			  return false;
	 }
```
<br><br> 


## 프로필 이미지 업로드

회원정보를 저장하는 excute() 메서드에서 동작합니다. 저장소에서 파일 이름이 겹치지 않도록 회원ID로 파일명을 새로 만들고 원본 파일명을 따로 저장했습니다.

```java
import java.io.File;
import org.apache.commons.io.FileUtils;

if (getUpload() != null) {
			// 파일명 + ID
			String file_name = "file_" + memberParam.getM_id();
			// 파일 확장자를 가져온다
			String file_ext = getUploadFileName().substring(getUploadFileName().lastIndexOf('.') + 1,
					getUploadFileName().length());
			// 파일의 경로와 이름을 file객체에 담는다
			File destFile = new File(fileUploadPath + file_name + "." + file_ext);
			// 임시파일을 복사 후 설정한 이름과 경로(서버 컴퓨터)에 저장한다.
			FileUtils.copyFile(getUpload(), destFile);
			memberParam.setM_id(memberParam.getM_id());
			memberParam.setProf_image_org(getUploadFileName());
			memberParam.setProf_image_save(file_name + "." + file_ext);

		}else {
			String file_name = "noImage.jpg";
			memberParam.setM_id(memberParam.getM_id());
			memberParam.setProf_image_org(file_name);
			memberParam.setProf_image_save(file_name);
		}
		// DB에 파일의 원본이름과 새로 설정한 이름을 업데이트한다.
		sqlMapper.update("memSQL.updateProfile", memberParam);
		return SUCCESS;
```
<br><br> 

## 회원정보 수정 탈퇴

회원이 로그인 후 자신의 가입 정보를 확인하고 수정하고 탈퇴할 수 있는 기능을 구현했습니다. 회원 정보와 프로필 이미지를 구분해서 처리하도록 하였습니다.

<br> 

*-회원 정보 수정*

```java
	public String modifyMemPro() throws Exception {
		//회원 정보 수정내역을 DB에 저장
		sqlMapper.update("memSQL.updateMem", memberParam);

		if (getUpload() != null) { //회원정보 수정 form태그 제출에 프로필 이미지가 포함되었을때
			//이전 프로필 이미지 삭제
			if(!memberParam.getProf_image_save().equals("noImage.jpg")) {
				deletefile = new File(fileUploadPath + memberParam.getProf_image_save());

				if (deletefile.isFile())
					FileUtils.forceDelete(deletefile);
			} 
			//새로운 프로필 이미지 저장
			String file_name = "file_" + memberParam.getM_id();
			String file_ext = getUploadFileName().substring(getUploadFileName().lastIndexOf('.') + 1,
					getUploadFileName().length());
			File destFile = new File(fileUploadPath + file_name + "." + file_ext);
			FileUtils.copyFile(getUpload(), destFile);
			
			memberParam.setProf_image_org(getUploadFileName());
			memberParam.setProf_image_save(file_name + "." + file_ext);
			memberParam.setM_id(memberParam.getM_id());
			//회원 프로필 이미지 수정 내역을 DB에 저장
			sqlMapper.update("memSQL.updateProfile", memberParam);
		}

		return SUCCESS;
	}
```
<br> 
*-회원 탈퇴*

```java
public String deleteMem() throws Exception {
		if (memberResult.getProf_image_save() != null) {
			//파일 삭제
			File deleteFile = new File(fileUploadPath + memberResult.getProf_image_save());
			deleteFile.delete();
		}
		//회원정보 삭제
		sqlMapper.delete("memSQL.deleteMem", memberParam);
    	deletemembercheck = 1; //처리결과를 구분해서 화면 출력
		session.remove("session_id"); //세션에 저장된 ID 삭제
		session.remove("session_adminYN"); //회원등급 삭제
}
```

<br><br><br><br> 

------



# 로그인 페이지 구현

<br>
## 로그인

로그인시 입력한 데이터로 회원 정보를  DB에서 SELECT 문으로 검색하고 그 결과를 객체에 담아서 if문으로 처리하였습니다. if문 조건에 만족하면 세션에 회원 ID와 멤버등급을 저장합니다.

```java
	public String execute() throws Exception {
		memberResult = (memVO)sqlMapper.queryForObject("memSQL.loginPro",memberParam);
		if(memberResult != null) {
			int adminYN = memberResult.getAdmin_yn();
				session.put("session_id", memberResult.getM_id());
				session.put("session_adminYN", adminYN);
				return SUCCESS;
			
		} else {
			return ERROR;
		}
		
	}
```
<br><br> 


## 아이디 찾기

사용자가 아이디 찾기 요청시 입력한 데이터로 DB에서 회원ID를 찾아서 결과를 출력해줍니다. 아이디를 그대로 보여주지 않고 " * "으로 가려서 출력해주도록 구현했습니다. 그러나 당시에 작성한 코드(위)는 id에서 정해진 부분만을 일부 가리는 방식이였고 문제점을 보완한 코드(아래)를 작성하였습니다.

*-수정 전*

```java
	public String findId() throws Exception{
		memberResult = (memVO)sqlMapper.queryForObject("memSQL.findId",memberParam);
		
		if(memberResult != null) {
		String findIdResult = memberResult.getM_id();
		int idLength = findIdResult.length();
		if(5 < idLength) {
			m_id = findIdResult.substring(0,1) + "*" + findIdResult.substring(2,3) + "*" + findIdResult.substring(4,idLength);
			memberResult.setM_id(m_id);
			}
		}
		return SUCCESS;
	}
```

*-수정 후*

```java
		if(memberResult != null) {
		String findIdResult = memberResult.getM_id();
		
		String[] idToArray = findIdResult.split("");
		List<String> con_Id = new LinkedList<>();
		for(int i =0; i<idToArray.length; i++) {
			if(i>1 && i%2 == 0) {
				con_Id.add(i, "*");
			} else {
				con_Id.add(i, idToArray[i]);
			}
		}
		String[] idArray = con_Id.toArray(new String[con_Id.size()]);
		String blindId = String.join("", idArray);
		memberResult.setM_id(blindId);
```
*-출력 화면*

![](C:\Users\ykk\midas123\assets\images\post\OFFTHEBALL\oft-oft-findidresult.jpg){: .center-image }

<br><br> 


## 비밀번호 찾기 이메일 전송

사용자가 비밀번호 찾기 요청시 입력한 데이터로 DB에서 회원 비밀번호를 찾은 후 회원에게 이메일로 자동 전송해주는 기능을 구현하였습니다. JavaMail API를 이용했습니다. 

```java
	public String findPw() throws Exception{
		memberResult = (memVO)sqlMapper.queryForObject("memSQL.findPw",memberParam);
		if(memberResult !=null) {
		String line = System.getProperty("line.separator");
		String subject = memberResult.getM_name()+"님, 비밀번호를 알려드립니다. -OFFTHEBALL";
		String content =
			      "안녕하세요. OFF THE BALL 관리자 입니다." + line
			    + memberResult.getM_name() + "님, 고객님 비밀번호는 " + memberResult.getM_passwd() +" 입니다." + line + line
			    +"저희 OFF THE BALL은 비밀번호 찾기 결과를 가입하신 이메일로 보내드리고 있습니다." + line
			    + "비밀번호 찾기를 요청하신 적이 없거나 다른 문의사항이 있으시면 아래 메일로 문의해주시기 바랍니다." + line + line
			    + "about.offtheball@gmail.co.kr" + line + line + line
			    + "※주의: 저희는 고객님의 개인정보를 묻지 않습니다. 관리자 사칭에 주의하시기 바랍니다." + line;
		Emailsend mail = new Emailsend();
		mail.GmailSet(memberResult.getM_email(), subject, content);
		return SUCCESS;
		} else {
			return ERROR;
		}
	}
```
<br>

*-이메일 수신 결과*

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\OFFTHEBALL\OFFTHEBALL-findpw-result.jpg){: .center-image }



# 관리자 페이지

## 회원 목록 게시판 검색

회원 목록을 출력하는 게시판 입니다. 회원 검색 기능도 추가하였습니다. 회원의 아이디, 이름, 이메일로 회원을 검색할 수 있습니다. 

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\OFFTHEBALL\ofb-memberlist.jpg){: .center-image }

*-회원 이름으로 검색한 결과*

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\OFFTHEBALL\ofb-memberlistsearch.jpg){: .center-image }



```java
@SuppressWarnings("unchecked")
	public String memberList() throws Exception {
		if(session.get("session_id") == null) { //로그인 안된 사용자 처리
			return LOGIN;
		}
		if((Integer)session.get("session_adminYN") != 1) { //관리자 등급이 아닌 사용자 처리
			return ERROR;
		}
		String paging = "adminMemList";
		
		if (getSearch() == null || getSearch().equals("")) { //회원목록 페이지 처리
			memlist = sqlMapper.queryForList("memSQL.memList");
			totalCount = memlist.size();
			//페이징
			page = new pagingAction(currentPage, totalCount, blockCount, blockPage, paging);
		} else { //회원 정보 검색 결과 페이지 처리
			HashMap<String, Object> searchMap = new HashMap<>();
			String topics[] = { "m_id", "m_name", "m_email" }; //회원 아이디,이름,이메일 중 선택 후 회원 정보 검색
			searchMap.put("param1", topics[getTopic()]);
			searchMap.put("param2", "%" + getSearch() + "%"); //검색어
			memlist = sqlMapper.queryForList("memSQL.memSearch", searchMap);
			totalCount = memlist.size();
			//페이징
			page = new pagingAction(currentPage, totalCount, blockCount, blockPage, getTopic(), getSearch(),paging);																							
		}
		//웹브라우저 화면에서 출력될 페이징 html코드
		pagingHtml = page.getPagingHtml().toString();
		int lastCount = totalCount;
		if (page.getEndCount() < totalCount)
			lastCount = page.getEndCount() + 1;
		memlist = memlist.subList(page.getStartCount(), lastCount);

		return SUCCESS;
	}
```



## 회원 정보 상세페이지 등급변경

회원 목록에서 회원을 클릭하면 상세페이지로 이동합니다. 회원 등급 변경 기능을 구현하였습니다.

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\OFFTHEBALL\ofb-memberdetail.jpg){: .center-image }



```java
	public String memberView() throws Exception {
		if(session.get("session_id") == null) {
			return LOGIN;
		}
		if((Integer)session.get("session_adminYN")!=1) {
			return ERROR;
		}
		memberResult = (memVO) sqlMapper.queryForObject("memSQL.memListView", memberParam);
		if (memberResult.getProf_image_save() != null) {
			prof_image_save = memberResult.getProf_image_save();
			prof_image_org = memberResult.getProf_image_org();
			
			profpath = request.getContextPath() + "/admin/member/profUpload/" + prof_image_save;
		}
		return SUCCESS;
	}

	public String adminRightModi() throws Exception {
		sqlMapper.update("memSQL.updateAdminYN", memberParam);
		return SUCCESS;
	}
```

