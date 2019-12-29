---
layout: single
title: Redux 기본 개념 정리
tag: [redux, react]
kinds: 포스트
toc: true
toc_sticky: true
---

Redux는 자바스크립트 어플리케이션을 위한 state 관리 도구 입니다. 그리고 리액트와 사용하는 경우가 많습니다. 리액트는 기본적으로 컴포넌트에서 로컬 state를 관리하기 때문에 컴포넌트 숫자가 늘어날 수록 state 관리가 어렵습니다. Redux는 Store라는 공간에서 어플리케이션의 state를 저장하고 각 컴포넌트에서 저장된 state에 접근하고 업데이트 할 수 있게 해줍니다.

[why-use-redux-reasons-with-clear-examples](https://blog.logrocket.com/why-use-redux-reasons-with-clear-examples-d21bffd5835/)

# Actions

액션은 store에 저장할 정보를 운반하는 역할을 한다. 액션은 store.dispatch()를 이용해서 store로 보낸다. 아래 코드는 todo 앱에 사용되는 액션 중 하나이다.

```javascript
const ADD_TODO = 'ADD_TODO'
```

액션은 자바스크립트 객체로 액션 타입을 나타내는 속성('ADD_TODO')을 꼭 가지고 있어야 한다. 타입은 상수 문자열로 정의되어야 하고 앱의 규모가 크다면 별도의 모듈 안에서 작성해도 된다. 그리고 모듈은 아래처럼 import 한다. [reducing-boilerplate](https://redux.js.org/recipes/reducing-boilerplate)

```javascript
import { ADD_TODO, REMOVE_TODO } from '../actionTypes'
```

타입을 제외한 에러, 페이로드, 메타 등의 속성을 포함하는 액션의 구조는 사용자에게 달려있다. [flux-standard-action](https://github.com/redux-utilities/flux-standard-action)
각각의 액션은 어플리케이션에서 사용하는 객체 전체 보다는 배열의 인덱스나 유니크 ID 등 최소한의 데이터만 전달하는게 좋다.

```javascript
{
  type: TOGGLE_TODO,
  index: 5
}
```

<br>

# Action Creators

말그대로 액션을 만드는 함수이다. 액션 크레이터는 단순히 액션을 리턴한다. 액션 사용과 테스트가 쉬워진다.

```javascript
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

<br>

# Reducers

리듀서는 어플리케이션의 state가 액션 타입에 따라서 어떻게 변해야 하는지 명세한다. 

## Designing the State  Shape

모든 어플리케이션 state는 하나의 객체로 저장된다. 어떤 데이터와 UI state를 state 트리에 저장하되 둘을 구분해야 한다. [organizing-state](https://redux.js.org/faq/organizing-state)

복잡한 구조의 앱에서 각각의 엔티티가 서로를 참조하는 것을 원한다면 state를 최대한 정규화해야 한다. 모든 엔티티는 ID를 가지고 저장되어야 하고 다른 엔티티나 리스트에서 ID로 참조되어야 한다.

<br>

## Handling Action 

리듀서는 이전의 state와 액션을 받아서 다음 state를 리턴한다. 리듀서는 순수해야 합니다. 인자를 조작하거나 API를 요청하지 말아야 합니다.

```javascript
(previousState, action) => newState
```

리듀서에서는 state를 조작하지 않는다. Object.assign()을 이용해서 state 객체를 새로 복사한다.
반드시 Object.assign()의 첫번째 파라미터를 비워둬야 한다. state를 첫번째에 두면 변해버린다.
swich 조건문을 사용해서 액션 타입에 알맞은 값을 리턴한다. 그리고 알 수 없는 액션에 대비해서 default case에서 이전 state를 리턴해야 한다.  

※Object.assign()는 ES6의 일부이므로 오래된 브라우저에선 지원되지 않는다. 대신 [polyfill](https://polyfill.io/v3/) 또는 [`_.assign()`](https://lodash.com/docs#assign)을 사용한다.

```javascript
import {
  ADD_TODO,
  TOGGLE_TODO,
  SET_VISIBILITY_FILTER,
  VisibilityFilters
} from './actions'

...

function todoApp(state = initialState, action) {
  //state = initialState는 아래 코드를 대신함  
  //if (typeof state === 'undefined') {
    //return initialState
  //}  
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos, //전개구문(Spread_syntax) 
          {
            text: action.text,
            completed: false
          }
        ]
      })
    default:
      return state
  }
}
```

전개구문 예시: [Spread_syntax](https://codeburst.io/javascript-es6-the-spread-syntax-f5c35525f754)

<br>

## Splitting Reducers

리듀서를 아래처럼 나누면 각 리듀서 별로 state를 관리하게 된다. 나누어진 리듀서는 combineReducers()를 이용해서 다시 합칠 수 있다.

```javascript
import { combineReducers } from 'redux'
import {
  ADD_TODO,
  TOGGLE_TODO,
  SET_VISIBILITY_FILTER,
  VisibilityFilters
} from './actions'
const { SHOW_ALL } = VisibilityFilters

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

<br>

# Store

앞서 상태에 대해 설명해주는 액션과 그 액션에 따라서 state를 업데이트하는 리듀서에 대해 공부했다. 스토어는 이것들을 하나로 묶어준다. 스토어의 역할은 아래와 같다.

- 어플리케이션 state를 보관한다.
- getState()를 이용한 state에 대한 접근을 허용한다.
- dispatch()로 state의 업데이트를 허용한다.
- subscribe()로 리스너를 등록한다.
- 등록되지 않은 리스너를 subscribe()를 리턴하는 함수로 다룬다.

<br>

어플리케이션에서는 오직 하나의 스토어만 가질 수 있다. 서비스 로직 별로 데이터를 나누려면 리듀서를 나누는 방법을 선택해야 한다.

스토어는 [createStore()](https://redux.js.org/api/createstore)에 리듀서를 파라미터로 사용하면 만들 수 있다. 그리고 두 번째 인자로 initial state를 정의할 수 있다. 

```javascript
import { createStore } from 'redux'
import todoApp from './reducers'
const store = createStore(todoApp)
```

<br>

# Data Flow

redux는 엄격한 단방향의 데이터 플로우 구조에서 동작한다. 이 말은 곧 어플리케이션의 모든 데이터는 같은 라이프사이클을 갖는 것과 같고 어플리케이션의 로직을 만드는게 더 쉬워지고 예상 가능해진다. 또한 정규화를 장려하므로 같은 데이터를 가진 다수의 중복 코드를 지양한다.

redux 앱에서 데이터 라이프사이클은 아래와 같다.

1. store.dispatch(action) 호출
2. store가 reducer에게 인자를 전달(액션, 현재 state 트리)
3. 루트 reducer가 다수의 reducer의 결과물을 하나의 state tree와 결합
4. store가 루트 reducer가 리턴한 state tree를 저장

<br>

# Usage with React



## Installing React Redux 

```
npm install --save react-redux
```

<br>

## Presentational and Container Components

redux를 위한 리액트 바인딩을 presentational 컴포넌트와 container 컴포넌트로 구분한다. 이 방법은 앱의 구조를 파악과 컴포넌트 재사용이 쉽다.

| Presentational Components | Container Components             |                                                |
| ------------------------: | :------------------------------- | ---------------------------------------------- |
|                   Purpose | How things look (markup, styles) | How things work (data fetching, state updates) |
|            Aware of Redux | No                               | Yes                                            |
|              To read data | Read data from props             | Subscribe to Redux state                       |
|            To change data | Invoke callbacks from props      | Dispatch Redux actions                         |
|               Are written | By hand                          | Usually generated by React Redux               |

container 컴포넌트는 Redux store와 연결된다. 기술적으로 store.subscribe()를 사용해서 container 컴포넌트를 작성할 수 있지만, React-Redux의 성능 최적화를 직접 구현하기 어렵기 때문에 이 방법을 권하지 않는다.
이러한 이유로 이 컴포넌트를 직접 작성하기 보다는 connect() 함수를 이용해서 생성한다.

<br>

## Presentational Components

일반적인 리액트 컴포넌트이다. 로컬 state 또는 라이프 사이클 메서드가 필요한 경우가 아니라면 아래 코드처럼 간단하게 함수 컴포넌트를 사용한다. 

```react
import React from 'react'
import PropTypes from 'prop-types'

const Todo = ({ onClick, completed, text }) => (
  <li
    onClick={onClick}
    style={{
      textDecoration: completed ? 'line-through' : 'none'
    }}
  >
    {text}
  </li>
)

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  completed: PropTypes.bool.isRequired,
  text: PropTypes.string.isRequired
}

export default Todo
```

<br>

## Container Components

presentational 컴포넌트를 Redux에 연결하려면 이 컴포넌트가 필요하다. 이전에 언급한 것처럼 기술적으로만 보면 container 컴포넌트는 Redux state 트리의 일부분을 읽는 store.subscribe()를 사용하는 리액트 컴포넌트이다. 하지만 여러가지 최적화를 제공하고 불필요한 re-render를 예방 할 수 있는 React-Redux 라이브러리의 connect() 함수를 사용하는 것을 추천한다. 

connect()를 사용하기 위해서는 mapStateToProps 함수를 정의해야 한다. 이 함수는 현재의 Redux store state가 presentational 컴포넌트의 prop으로 전달되기 위해서 어떻게 변해야 하는지를 기술한다.

```react
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
    case 'SHOW_ALL':
    default:
      return todos
  }
}

const mapStateToProps = state => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```

또한 container 컴포넌트는 action을 dispatch 할 수 있다. 위와 비슷한 방식으로, dispatch()를 받고 presentational 컴포넌트에 주입하고자 하는 콜백 props를 리턴하는 mapDispatchToProps()을 호출한다.

```react
const mapDispatchToProps = dispatch => {
  return {
    onTodoClick: id => {
      dispatch(toggleTodo(id))
    }
  }
}
```

mapStateToProps와mapDispatchToProps를 통시에 전달

```react
import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

<br>

## Other Components

presentational과 container 구분이 어려운 컴포넌트를 말한다. 예를 들면 form 엘리먼트와 함수가 결합되어 있는 경우가 있다. 아래 AddTodo 컴포넌트는 프레젠테이션과 로직이 섞여있다.

```react
import React from 'react'
import { connect } from 'react-redux'
import { addTodo } from '../actions'

let AddTodo = ({ dispatch }) => {
  let input

  return (
    <div>
      <form
        onSubmit={e => {
          e.preventDefault()
          if (!input.value.trim()) {
            return
          }
          dispatch(addTodo(input.value))
          input.value = ''
        }}
      >
        <input
          ref={node => {
            input = node
          }}
        />
        <button type="submit">Add Todo</button>
      </form>
    </div>
  )
}
AddTodo = connect()(AddTodo)

export default AddTodo
```

<br>

## Passing the Store

container 컴포넌트는 Redux store에 접근해야 한다. 모든 container 컴포넌트로 store를 전달하는 방법 외에 Provider라는 React-Redux 컴포넌트로 한번에 store 전달하는 방법이 있다. 아래와 같이 root 컴포넌트를 렌더링할때 사용해주면 된다. 

```react
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

const store = createStore(todoApp)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

<br><br>

출처: https://redux.js.org/basics/actions

