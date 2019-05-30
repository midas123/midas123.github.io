---
layout: single
title: React 기본 개념 정리
tag: [javascript, react]
---

요즘은 리액트 같은 라이브러리 혹은 프레임워크를 이용해서 화면 렌더링을 사용자의 브라우저에서 담당하고 그때그때 필요한 데이터만 서버에 요청하는 SPA(Single Page Application) 방식이 많이 사용되고 있다.
이 방식은 서버 측에서는 자원의 사용을 최소화하고 사용자 측에서는 서버의 응답을 기다리는 시간이 줄기 때문에 사용 경험이 향상되는 장점이 있는 것으로 알고 있다. 

이런 자바스크립트 기반 기술이 웹 개발자 커뮤니티에서도 자주 화두가 되면서 자연스럽게 관심을 갖게 되었고 그 중 react를 먼저 공부하려고 한다. 

# Reactjs란?

React는 프레임워크가 아닌 라이브러리이다. 유저 인터페이스를 위해 만들어졌다. 
복잡한 UI 관련 코드들를 '**컴포넌트**' 라고 부르는 좀 더 작고 고립 된 조각으로 구성한다. React의 Virtual DOM에 의해서 DOM 엘레먼츠가 바뀔때마다 전체 뷰가 다시 로드하는 대신 수정된 부분만 업데이트 하기 때문에 처리속도 더 빨라졌고
각 컴포넌트를 재사용하면서 UI 관련 중복 코드를 줄일 수 있다.

<br>

# React 컴포넌트 클래스

Reactjs는 UI 화면을 여러 개의 컴포넌트로 쪼개서 구성한다. 각 컴포넌트는 캡슐화되어 독립적으로 동작한다. 그러므로 각각의 심플한 컴포넌트를 이용해서 복잡한 UI 화면을 만들 수 있다. 먼저 컴포넌트 구조를 정리하고 컴포넌트 내에서 사용되는 [props](#props), [state](#state), [라이프 사이클 메서드](#컴포넌트-라이프-사이클-메서드)를 간단하게 정리하려고 한다.  ES6에서 기본적인 컴포넌트 클래스의 모습은 아래와 같다.

```react
class Square extends React.Component {
  render() {
    return (
     <div className="shopping-list">
      <h1>Shopping List for {props.name}</h1>
      <ul>
        <li>Instagram</li>
        <li>WhatsApp</li>
        <li>Oculus</li>
      </ul>
	</div>
    );
  }
}
ReactDOM.render(
  <Square name="jane"/>,
  document.getElementById('root')
);
```

Square 컴포넌트는 React.Component 클래스를 상속 받고, render() 메서드를 통해서 화면에서 보여줄 view를 return 한다. 이 메서드의 return 괄호 안에 있는 코드는 [JSX](https://reactjs.org/docs/introducing-jsx.html) 문법에 의해서 작성된 html 코드이다. JSX 안에서는 html 코드와 함께 어떠한 javascript도 사용 가능하다.

참고로 JSX 문법을 대신해서 아래와 같은 코드를 직접 작성해도 된다. 참고로 JSX는 개발자의 편의를 위해 만들어진 문법으로 빌드 타임시 자동으로 아래 코드로 변환 된다.

```react
return React.createElement('div', {className: 'shopping-list'},
  React.createElement('h1', /* ... h1 children ... */),
  React.createElement('ul', /* ... ul children ... */)
);
```



최종적으로 ReactDOM.render() 메서드가 Square 컴포넌트를 Virtual DOM으로 반환하고 JSX 문법으로 작성한 내용이 'root' 컨테이너 안에 위치한 상태로 웹브라우저 화면에 출력된다.



<br>

**React 엘레먼트(element)**

위에 React.createElement() 메서드는 각각 리엑트 엘레먼트를 반환한다. 엘레먼트는 컴포넌트를 구성하는 요소이다.
자바스크립트 객체처럼 값을 저장해서 전달 할 수 있다. 이런 엘레먼트가 모여 만들어진 컴포넌트는 곧 하나의 DOM 객체가 된다.



------

<br>



## props

properties의 줄임말로 컴포넌트에서 사용되는 파라미터이다.  사용 예시는 아래와 같다.

<br>

**컴포넌트 렌더링**

```react
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

<br>

**컴포넌트 조립**

```react
function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}
```

<br>

**컴포넌트 추출**

```react
function formatDate(date) {
  return date.toLocaleDateString();
}
function Comment(props) {
  return (
    <div className="Comment">
    <UserInfo user={props.author}/>
      <div className="Comment-text">{props.text}</div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
function Avatar(props){
  return(
      <img className="avatar"
        src={props.user.avatarUrl}
        alt={props.user.name}/>
  )
}
function UserInfo(props){
  return(
     <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  )
}
const comment = {
  date: new Date(),
  text: 'I hope you enjoy learning React!',
  author: {
    name: 'Hello Kitty',
    avatarUrl: 'https://placekitten.com/g/64/64',
  },
};
ReactDOM.render(
  <Comment
    date={comment.date}
    text={comment.text}
    author={comment.author}
  />,
  document.getElementById('root')
);
```

<br>

**Props Read-only**

컴포넌트는 props를 수정,변조해서는 안된다.(pure vs impure)

```react
//pure
function sum(a, b) {
  return a + b;
}

//impure
function withdraw(account, amount) {
  account.total -= amount;
}
```





## state

만약 컴포넌트가 실행되는 동안 그 값이 계속 수정/변조되야 하는 객체를 사용해야 한다면? 예를 들어, 컴포넌트로 구현한 시계를 초 단위로 업데이트 할때 state를 사용한다. 

state는 class 형태의 컴포넌트에서만 사용할 수 있고 아래 코드의 실행과정은 아래와 같다.
우선 state를 할당하는 constructor가 필요하다. 그리고 componentDidMount() 메서드에 1초 간격으로 tick() 메서드를 실행하는 코드를 작성한다. tick() 메서드는 새로운 Date 객체를 생성하고 그 값을 state 객체에 저장한다. 결국 1초 간격으로 현재 시각이 업데이트 되고 이를 뷰에서 확인 할 수 있는 것이다.

<br>

## 컴포넌트 라이프 사이클 메서드

위에서 설명한 componentDidMount() 메서드 처럼 컴포넌트의 라이프 사이클 마다 동작하는 메서드가 있다. 이 메서드를 오버라이드해서 작성한 코드는 메서드의 라이프 사이클에 맞춰서 실행된다. 

**componentDidMount()** - 컴포넌트의 output이 DOM에 렌더링 된 직 후에 실행된다.

**componentWillUnmount()** - 컴포넌트가 DOM에서 해제된 직 후 실행된다. 

더 많은 라이프 사이클 메서드에 대한 정보는 여기서 확인 -> [react-lifecycle-methods-diagram](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)



<br>

```react
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

<br>

참고:

[react-lifecycle-methods-diagram](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

[react-component](https://reactjs.org/docs/react-component.html)

[components-and-props](https://reactjs.org/docs/components-and-props.html)

[adding-lifecycle-methods-to-a-class](https://reactjs.org/docs/state-and-lifecycle.html#adding-lifecycle-methods-to-a-class)