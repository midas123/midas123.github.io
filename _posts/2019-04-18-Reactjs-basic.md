---
layout: single
title: React 기본 개념 정리
tag: [javascript, react]
kinds: 포스트
toc: true
toc_sticky: true
---

요즘은 리액트 같은 라이브러리 혹은 프레임워크를 이용해서 사용자의 브라우저에서 화면 렌더링을 담당하고 그때그때 데이터가 필요한 경우 서버에 요청하는 SPA(Single Page Application) 방식이 많이 사용되는 것으로 알고 있습니다. 이 방식은 서버 측에서는 자원의 사용을 최소화하고 사용자 측에서는 서버의 응답 시간이 줄어들기 때문에 사용 경험이 향상되는 장점이 있는 것으로 알고 있습니다. 

이런 자바스크립트 기반 기술이 웹 개발자 커뮤니티에서도 자주 화두가 되면서 자연스럽게 관심을 갖게 되었고 react 공식문서를 보고 정리해보았습니다.

# Reactjs란?

React는 프레임워크가 아닌 라이브러리이다. 유저 인터페이스를 위해 만들어졌다. 
복잡한 UI 관련 코드들를 '**컴포넌트**' 라고 부르는 좀 더 작고 고립 된 조각으로 구성한다. React의 Virtual DOM에 의해서 DOM 엘레먼츠가 바뀔때마다 전체 뷰가 다시 로드하는 대신 수정된 부분만 업데이트 하기 때문에 처리속도 더 빨라졌고
각 컴포넌트를 재사용하면서 UI 관련 중복 코드를 줄일 수 있다.

<br>

## 컴포넌트 클래스

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

# React 엘레먼트(element)

위에 React.createElement() 메서드는 각각 리엑트 엘레먼트를 반환한다. 엘레먼트는 컴포넌트를 구성하는 요소이다.
자바스크립트 객체처럼 값을 저장해서 전달 할 수 있다. 이런 엘레먼트가 모여 만들어진 컴포넌트는 곧 하나의 DOM 객체가 된다. 

```react
const element = <h1>Hello, world</h1>;
```

<br>

## 엘리먼트로 DOM 렌더링하기

HTML 파일에 'root'로 div태그를 만든다. 이 'root'에는 React DOM이 관리하는 모든게 들어간다.(ex. 컴포넌트, 엘리먼트)

```react
<div id="root"></div>
```

리액트만으로 만들어진 어플리케이션은 하나의 root DOM node를 갖는다. 만약 다른 앱과 리액트를 통합한다면 원하는 만큼의 독립된 root DOM node를 가질 수 있다.

```react
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

<br>

## 렌더링 된 엘리먼트 수정하기

엘리먼츠는 불변한다. 한번 만들어진 엘리먼트의 자식 또는 속성을 바꿀 수 없다. UI를 수정 하려면 새로운 엘리먼트를 만들어야 한다. (아래 코드에서는 1초 마다 엘리먼트가 생성됨)

```react
function tick() {
  const element = (
 //코드
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```

엘리먼트를 새로 생성해서 업데이트 할때에는 React DOM에 의해서 이전과 비교해서 바뀌 부분만이 업데이트 된다.

------

<br>

# 컴포넌트

전체 UI를 부분적으로 나누어 독립적이고, 재사용 가능한 형태로 만들 것을 컴포넌트라고 한다. 컴포넌트는 자바스크립트 function과 같다. 임의의 input(props)을 받아서 사용자 화면에 보여질 React 엘리먼츠를 리턴한다. [react-component-api-reference](https://reactjs.org/docs/react-component.html)

## 컴포넌트 정의

```react
//javascript function
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

```react
//ES6 class
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

※ES6 클래스가 function과 다른 특징은 state와 라이프사이클이다.

<br>

## 컴포넌트 렌더링

```react
const element = <Welcome name="Sara" />;
```

위처럼 유저가 정의한 컴포넌트(Welcome)를 나타내는 엘리먼트는 JSX 속성(name)을 객체 형태로 전달 받는다. 이 객체를 'props' 이라고 한다.

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

※ 컴포넌트 이름은 항상 대문자로 시작해야 한다. 

<br>

## 컴포넌트 조립

컴포넌트는 다른 컴포넌트에서 참조되어 출력될 수 있다. 이러한 이유로 버튼, 폼, 대화상자, 화면 등 컴포넌트로 표현되는 것들을 재사용 할 수 있다.

```react
<Welcome name="Sara" />
<Welcome name="Cahal" />
<Welcome name="Edite" />
```

<br>

## 컴포넌트 추출

컴포넌트 내에서 몇 번씩 사용되는 부분을 새로운 컴포넌트로 만든다. 버튼, 패널, 아바타 또는 앱, 댓글 등은 좋은 재사용 컴포넌트 후보이다.

<br>

# props

properties의 줄임말로 컴포넌트에서 사용되는 파라미터이다. 

## Props Read-only

리액트의 모든 컴포넌트는 반드시 props를 수정,변조해서는 안된다.(pure vs impure) 

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

UI에 동적인 변화가 필요한 경우엔 state를 사용한다.

<br>

# state

만약 컴포넌트가 UI 화면에서 보여주는 값이 계속 수정/변조되야 한다면? 예를 들어, 컴포넌트로 구현한 시계를 초 단위로 업데이트 할때 state를 사용한다. 

state는 ES6 class 형태의 컴포넌트에서만 사용할 수 있다. 컴포넌트에서 state를 사용하려면 우선 최초로 state를 할당하는 constructor가 필요하다. [why-do-we-write-super-props](https://overreacted.io/why-do-we-write-super-props/)

```react
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {}; //assigns the initial state 
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

## 컴포넌트에 라이프 사이클 메서드 추가하기

많은 컴포넌트가 있는 어플리케이션 내에서 컴포넌트가 파기되었을때 사용하던 리소스를 다시 사용 가능하게 만드는 것은 매우 중요하다.

컴포넌트가 DOM으로 최초 렌더링 되었을때를 리액트에서는 '마운팅' 이라고 한다.
그리고 컴포넌트가 DOM이 제거 되었을때를 '언마운팅' 이라고 한다.

컴포넌트가 '마운팅', '언마운팅' 되었을때 어떤 코드를 실행하려면 아래 메서드를 정의한다. 이 메서드가 라이프 사이클 메서드 이다. [react-lifecycle-methods-diagram](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

```react
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {};
  }
  componentDidMount() {//마운팅
  }

  componentWillUnmount() {//언마운팅
  }
  render() {
  //코드
    );
  }
}
```

<br>

타이머 예제)

```react
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {

  }

  componentWillUnmount() {

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
```

<br>

## State 올바르게 사용하기

**1.state를 직접 수정하지 않는다. -> setState()를 사용한다.**

```react
// Wrong
this.state.comment = 'Hello';
// Correct
this.setState({comment: 'Hello'});
```

this.state는 constructor 내에서만 할당되어야한다.

<br>

**2.state는 비동기적으로 업데이트 될 수 있다.**
리액트는 퍼포먼스를 위해서 다수의  setState()를 batch해서 호출할 수 있다. setState() 내에 this.props와 this.state 비동기적으로 업데이트 될 수 있으므로 setState()의 인자로 객체가 아닌 function을 사용해야 한다.
그러면 첫번재 인자로 state를 받은 후 업데이트가 적용되었을때 props를 두 번째로 받게 된다.

```react
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

<br>

**3.state의 수정내용은 병합된다.**
state에 몇 개의 변수가 포함되어 있을때 setState()로 개별적으로 그 변수를 수정할 수 있다. 

<br>

## 데이터는 아래로 흐른다

다른 컴포넌트의 state에 접근할 수 없다. 오직 state를 정의한 컴포넌트에서만 접근할 수 있다. 데이터는 위에서 아래로 또는 단방향으로 흐른다. 
컴포넌트가 소유한 state는 props로 자식 컴포넌트에게 전달할 수 있다. 그리고 자식 컴포넌트는 props가 어떤 컴포넌트에서 왔는지 알지 못한다.
어떤 state라도 항상 특정 컴포넌트에게만 소유되고 state의 데이터는 아래에 있는 자식 컴포넌트에게만 영향을 준다.

<br>

출처: https://reactjs.org/docs/hello-world.html

