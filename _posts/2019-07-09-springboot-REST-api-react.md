---
 layout: single
 title: 개발 단계에서 springboot-react 연동
 tag: [springboot, react, fetch]
 kinds: 포스트
---

이전 프로젝트의 프론트엔드를 reactjs로 만들 계획이며, 먼저 개발 단계에서 springboot REST api를 백엔드로 하고 reactjs로 프론트엔드로 세팅하는 방법을 알아보았습니다. 이 방법은 proxy를 사용하기 때문에 개발 단계에서만 유효합니다.

<br>



**1.먼저 스프링부트 프로젝트(백엔드)와 [리액트 프로젝트(프론트엔드) 생성](https://reactjs.org/docs/create-a-new-react-app.html)합니다.**

<br>

**2.백엔드에서 http://localhost:8080/api/user로 요청시 user 정보를 리턴해주도록 합니다.** 

```java
@RestController
@RequestMapping("/api")
public class UserRestController {
	@GetMapping("/user")
	public Users getPost(Users user) {
		user.setUsername("john");
		user.setEmail("1234@email.com");
		return user;
	}
}
```

```java
public class Users {
	private String username;
	private String email;
	
	public String getUsername() {
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
}
```

<br>

**3.리액트 프로젝트에서 [proxy를 수동으로 설정](https://facebook.github.io/create-react-app/docs/proxying-api-requests-in-development#configuring-the-proxy-manually)합니다. 아래 4번에서 [Fetch API](https://developer.mozilla.org/ko/docs/Web/API/Fetch_API)를 이용해서 '/api'로 리소스를 요청하면 proxy에 의해서 웹 서버 주소로 'http://localhost:8080/'를 사용합니다.**

-setupProxy.js

```react
const proxy = require('http-proxy-middleware');

module.exports = function(app) {
  app.use(proxy('/api', { target: 'http://localhost:8080/' }));
};
```

<br>

**4.백엔드로 요청한 user 정보를 출력하는 화면을 간단하게 구성합니다. [리액트 라이프 사이클 메서드](https://reactjs.org/docs/react-component.html#the-component-lifecycle) 중 하나인componentDidMount() 메서드에서 리소스를 원격으로 fetch 합니다.**

```react
import React from 'react';
import './App.css';
import Userinfo from './Userinfo.js';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {username:"", email:""};
  }
  componentDidMount() {
  fetch('/api/user')
        .then(response => 
          response.json())
        .then(jsonData => {
          console.log(jsonData) //{username: "john", email: "1234@email.com"}
          this.setState({"username":jsonData.username, "email":jsonData.email})
        })
 
        }
  render(){
    return (
    <div className="App">
        <Userinfo username={this.state.username} email={this.state.email}/>
    </div>
    );
  }  
}

export default App;
```

```react
import React from 'react';
import './Userinfo.css';

class Userinfo extends React.Component{
    render(){
        return(
            <div className="Userinfo">
                username: {this.props.username}<br></br>
                email: {this.props.email}
            </div>
        )
    }
}

export default Userinfo;
```



<br>

참고:

https://www.robinwieruch.de/react-fetching-data/#react-where-fetch-data

https://flaviocopes.com/react-state-vs-props/

