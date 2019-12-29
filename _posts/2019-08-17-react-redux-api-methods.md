---
layout: single
title: Redux - 주요 API 함수
tag: [redux, react]
kinds: 포스트
toc: true
toc_sticky: true
---

redux를 사용하면 react 앱에서 사용되는 모든 로컬 state를 한 곳에 저장하고 관리할 수 있다. 리액트 컴포넌트의 로컬 state를 저장하는 곳을 redux store라고 하며, 필요에 따라서 state를 업데이트하거나 그 값을 불러올 수도 있다. 이런 과정에서 사용되는 여러 api 함수가 있는데 각각의 역할과 사용 방법 등을 간단하게 정리해보려고 한다. 출처 링크에 가면 더 자세한 내용을 확인할 수 있다.

<br>

**요약**

- connect - 컴포넌트와 store를 연결하고 mapStateToProps와 mapDispatchToProps를 함수 인자로 전달
- mapStateToProps - 컴포넌트의 props와 store에 state를 맵핑해서 값을 가져올때 사용
- mapDispatchToProps - action을 dispatching하면서 store에 state를 업데이트 할때 사용




<br>

# Store 접근하기

React Redux는 아주 깊숙히 위치한 컴포넌트로 Store에 접근하도록 Context를 사용한다. 그리고 이 context 객체는 React.createContext()로 만든다. 
Provider 컴포넌트는 Redux store와 내부에 state를 context에 넣는다. 그리고 connect 함수로 state에 저장된 값을 읽고 업데이트 한다.

<br>

## 커스텀 Context 제공하기

디폴트 context를 사용하는 대신, 아래처럼 커스텀 context 인스턴스를 공급할 수 있다. 

```react
<Provider context={MyContext} store={store}>
  <App />
</Provider>
```

커스텀 context를 사용하면 디폴트 context는 사용되지 않으므로 connect로 연결된 컴포넌트에서도 같은 커스텀 context를 제공해야 한다.

```react
// You can pass the context as an option to connect
export default connect(
  mapState,
  mapDispatch,
  null,
  { context: MyContext }
)(MyComponent);

// or, call connect as normal to start
const ConnectedComponent = connect(
  mapState,
  mapDispatch
)(MyComponent);

// Later, pass the custom context as a prop to the connected component
<ConnectedComponent context={MyContext} />
```

<br>

## 다수의 Store 사용하기

Redux는 한 개의 store를 사용하도록 디자인 되었지만 필요할 경우, 여러 개의 store를 사용할 수 있다. 각 store는 분리되어 있는 context 인스턴스에서 고립되어 있다.

```react
// a naive example
const ContextA = React.createContext();
const ContextB = React.createContext();

// assuming reducerA and reducerB are proper reducer functions
const storeA = createStore(reducerA);
const storeB = createStore(reducerB);

// supply the context instances to Provider
function App() {
  return (
    <Provider store={storeA} context={ContextA} />
      <Provider store={storeB} context={ContextB}>
        <RootModule />
      </Provider>
    </Provider>
  );
}

// fetch the corresponding store with connected components
// you need to use the correct context
connect(mapStateA, null, null, { context: ContextA })(MyComponentA)

// You may also pass the alternate context instance directly to the connected component instead
<ConnectedMyComponentA context={ContextA} />

// it is possible to chain connect()
// in this case MyComponent will receive merged props from both stores
compose(
  connect(mapStateA, null, null, { context: ContextA }),
  connect(mapStateB, null, null, { context: ContextB })
)(MyComponent);
```

<br>

출처:https://react-redux.js.org/using-react-redux/accessing-store

<br><br>

# connect()

connect() 함수는 리액트 컴포넌트와 Redux store를 연결한다. 이 함수는 연결된 컴포넌트에 store에 저장된 데이터를 제공하거나 action을 store로 dispatch하는데 사용된다.

```react
function connect(mapStateToProps?, mapDispatchToProps?, mergeProps?, options?)
```

mapStateToProps는 Redux store에 state를 다룰때 사용되고 mapDispatchToProps는 action을 dispatch할때 사용한다.

※connect() 사용 예제:https://react-redux.js.org/api/connect#example-usage

<br><br>

# mapStateToProps: 데이터 추출

mapStateToProps는 connect 함수의 첫번째 인자로 들어간다. (첫번째 인자를 null 또는 undefined으로 두면 store를 구독하지 않음) 그리고 이 함수는 connect로 연결된 컴포넌트에서 store의 데이터를 가져오는데 사용된다. 

<br>

## 특징

- store내에 state가 변경될때 마다 호출됨.
- store 전체의 state를 받아서 컴포넌트가 필요로 하는 데이터를 리턴함.

```react
function mapStateToProps(state, ownProps?)
```

**state** - Redux store의 전체 state로 store.getState()로 리턴받는 것과 같다.

**ownProps** - store에서 데이터를 가져올때 사용되는 props이다. 예를 들어, store에서 id를 가진 데이터를 가져올때, 특정 id 값을 ownProps로 전달한다.

<br>

## mapStateToProps 사용 방법

```react
// TodoList.js

function mapStateToProps(state) {
  const { todos } = state
  return { todoList: todos.allIds }
}

export default connect(mapStateToProps)(TodoList)
```

**return** - 이 함수는 plain object를 리턴해야한다. 그리고 리턴한 객체는 컴포넌트에서 사용된다. (렌더링 퍼포먼스를 관리해야 할때 함수를 리턴하는 경우도 있음)

- 객체 안에 각 필드는 컴포넌트에서 prop으로 사용됨.
- 필드 값은 컴포넌트의 re-render를 결정하는데 사용됨.

<br>

출처: https://react-redux.js.org/using-react-redux/connect-mapstate

------

<br><br>

# mapDispatchToProps: 액션 보내기

mapDispatchToProps는 action을 store로 dispatching하는데 사용된다. dispatch는 Redux store의 함수로, store.dispatch는 state 업데이트를 트리거 할 수 있는 유일한 방법이다.

<br>

## mapDispatchToProps 사용방법

React-Redux에서 컴포넌트는 store에 직접 접근할 수 없고 connect를 통해서만 가능하다. 그리고 컴포넌트에서 액션을 dispatch하는 방법은 2가지이다.

<br>

1. 디폴트로 connect로 연결된 컴포넌트는 props.dispatch를 받는다. 그리고 스스로 action을 dispatch한다.
2. connect 함수의 인자로 mapDispatchToProps를 넘겨서 dispatch를 실행하는 함수를 만들고 그 함수를 컴포넌트의 props로 전달 할 수 있다.

<br>

위에서 말한 방법 중 1번, connect() 함수의 두번째 인자를 넣지 않으면 컴포넌트는 디폴트로 dispatch를 받는다.

```react
connect()(MyComponent)
// which is equivalent with
connect(
  null,
  null
)(MyComponent)

// or
connect(mapStateToProps /** no second argument */)(MyComponent)
```

위 방법으로 컴포넌트를 연결하고 받은 props.dispatch를 이용해서 아래처럼 action을 dispatch 할 수 있다.

```react
function Counter({ count, dispatch }) {
  return (
    <div>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <span>{count}</span>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>reset</button>
    </div>
  )
}
```

<br>

2번, mapDispatchToProps을 파라미터로 제공, 이 방법에서 mapDispatchToProps는 컴포넌트가 dispatch 해야하는 action을 직접 열거할 수 있게 해준다. 

아래는 버튼을 클릭했을때 dispatching 하는 예제이다.

```react
// button needs to be aware of "dispatch"
<button onClick={() => dispatch({ type: "SOMETHING" })} />

// button unaware of "dispatch",
<button onClick={doSomething} /
```

모든 action creator를 액션을 dispatch하는 함수로 감싸면 연결된 컴포넌트에서는 dispatch를 받을 수 없게 된다.

또한, 아래처럼 action을 dispatching하는 함수를 자식 컴포넌트로 보낼 수도 있다. 

```react
// pass down toggleTodo to child component
// making Todo able to dispatch the toggleTodo action
const TodoList = ({ todos, toggleTodo }) => (
  <div>
    {todos.map(todo => (
      <Todo todo={todo} onClick={toggleTodo} />
    ))}
  </div>
)
```

props로 함수를 전달 받는 자식 컴포넌트는 Redux에 대해 모르는 상태이므로 Redux store와의 연결 코드는 캡슐화된다.

<br>

## mapDispatchToProps 정의하기

mapDispatchToProps를 함수 또는 객체로 정의할 수 있다. Redux는 dispatching 동작을 커스텀하지 않는 한, 객체로 정의하는 것을 추천한다.

<br>

## 객체로 정의

- 각 필드는 action creator이다.
- 컴포넌트에서 더 이상 dispatch를 받을 수 없음.

```react
const mapDispatchToProps = {
  increment,
  decrement,
  reset
}
```

이렇게 정의하면 내부적으로 아래 코드가 실행된다.

```react
// React Redux does this for you automatically:
dispatch => bindActionCreators(mapDispatchToProps, dispatch)
```

그리고 아래처럼 변수이름을 사용하거나 connect 안에 정의할 수도 있다.

```react
import {increment, decrement, reset} from "./counterActions";

const actionCreators = {
  increment,
  decrement,
  reset
}

export default connect(mapState, actionCreators)(Counter);

// or
export default connect(
  mapState,
  { increment, decrement, reset }
)(Counter);
```

출처:https://react-redux.js.org/using-react-redux/connect-mapdispatch

<br>

------



