### 리덕스 모듈 만들기

리덕스 모듈이란 액션 타입,액션 생성 함수,리듀서 가 모두 들어있는 자바스크립트 파일을 의미한다.리덕스를 사용하기 위해 필요한 액션 타입,액션 생성 함수,리듀서 는 각각 다른 파일에 저장할 수도 있다.

한파일에 몰아서 작성해보자 -Ducks패턴이라 칭함

Ducks패턴은 한 파일에 액션 타입,액션생성함수,리듀서 함수도 선언하는 패턴이다. 주의할 점은 reducer 함수는 export default로 내보내주고 액션 생성 함수는 export로 내보내준다는 점이다.

이렇게 작성된 파일을 리덕스 모듈이라고 하며 나중에 액션 생성 함수를 불러올때 하나씩불러오거나 한꺼번에 불러올 수도 있다. Ducks패턴은 특히 리덕스를 처음 배우는 과정에서 사용하면 정말 쉽고 편하다. (리덕스 관련 코드들을 불리하는 방식은 정해지지 않았기 때문에 자유롭게 변경해도 상관은 없다.)

### 리덕스 모듈 만들기 -실습

다음과 같이 리덕스 모듈을 만들어보자!

두개의 모듈을 만드는데 첫번째 모듈은 counter이다

```react
//modules/counter.js



//1.우선 액션타입부터 선언하자
//Ducks 패턴을 사용할때는 액션 타입 선언시 문자열 앞부분에 counter/접두사를 붙임 다른 이름과 중복되지 않기 위해 모듈 이름을 앞에 붙인 것이다.
const SET_DIFF = "counter/SET_DIFF"; //나중에 counter에서 + , -를 할 수 있게 할 건데 몇씩 더할지 정하는 것
const INCREASE = "counter/INCREASE";
const DECREASE = "counter/DECREASE";

//2.그 다음으로 액션 생성 함수 작성
//export  키워드 => 내보내줘서 나중에 불러와서 사용함
export const setDiff = (diff) => ({ type: SET_DIFF, diff }); //파라미터로 받은 diff값을 액션 객체 내부에 넣어줌
export const increase = () => ({ type: INCREASE }); //따로 받아오는 파라미터는 없다.
export const decrease = () => ({ type: DECREASE }); //따로 받아오는 파라미터는 없다.

//3.리듀서에서 관리한 초기 상태 선언해줌- 모듈의 초기 상태!
const initialState = {
  //두가지 값을 관리함
  number: 0,
  diff: 1, //나중에 increase,decrease할때 1씩 +,- / 이 값을 바꿀 수 있게 할 예정 만약 5로 바꾸면 5단위로 +,-
};

//4.리듀서 만듬
//초기 상태는 initialState로! 액션받아와서 액션 타입에 따라 상태를 다르게 업데이트해주도록 작성.
export default function counter(state = initialState, action) {
  switch (action.type) {
    case SET_DIFF:
      return {
        ...state,
        diff: action.diff,
      };
    case INCREASE:
      return {
        ...state,
        number: state.number + state.diff,
      };
    case DECREASE:
      return {
        ...state,
        number: state.number - state.diff,
      };
    default:
      return state;
  }
}

```

두번째 모듈은 todos이다

```react
//modules/todos.js

const ADD_TODO = "todos/ADD_TODO"; //할일 항목 추가하는 액션
const TOGGLE_TODO = "todos/TOGGLE_TODO"; //할일 항목 체크를 하는 액션

let nextId = 1; //id값을 가져와서 사용하기 위해 변수로 선언하고 1로 초기화
//이제 액션 생성 함수를 선언해보자
export const addTodo = (text) => ({
  type: ADD_TODO,
  todo: {
    id: nextId++, //새로운 todo항목을 만들 때 id값을 필요로 함, 한번 호출하면 아이디값을 +1해줌
    text, //text를 받아와 넣어줌
  },
});
export const toggleTodo = (id) => ({
  //특정 아이디를 선택해서  toggle해줄것이기 때문에 파라미터로 id값을 받음
  type: TOGGLE_TODO,
  id, //id를 받아와 넣어줌
});

//초기 상태도 선언해주는데 제일처음엔 데이터가 없기때문에 일단 빈배열로 선언해줌!
const initialState = [
  /*이런식으로 추후 배열에 어떤식으로 데이터가 들어갈지 참고용으로 주석 달아놓으면 좋음
{
    id:1,
    text:'예시',
    done:false
}
*/
];

//todos라는 리듀서를 만들고 export default로 내보내줌
export default function todos(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return state.concat(action.todo); //state가 바로 배열타입이므로 바로 concat 메서드 사용가능
    case TOGGLE_TODO: //불변성을 지키면서 특정 id값 가진 항목을 찾아서 done값을 반전시켜줘야 함 => map 사용
      return state.map((
        todo //todo 항목을 가지고 만일에 todo.id와 action을 통해서 가져오는 id와 일치하면 해당 todo항목을 복사하고  done값을 반전시켜줌
      ) => (todo.id === action.id ? { ...todo, don: !todo.done } : todo));
    default:
      return state;
  }
}

```

modules/index.js 에 루트 리듀서를 만들어서 counter.js,todos,js 이 두개의 모듈에 있는 각각의 counter,todos 리듀서를 합쳐준다.

```react
//modules/index.js

import { combineReducers } from "redux"; //루트 리듀서를 만들 때에는 combineReducers라는 함수를 redux에서 받아와서 사용
import counter from "./counter"; //counter리듀서 불러오기
import todos from "./todos"; //tpdos리듀서 불러오기

//루트 리듀서 만들기
const rootReducer = combineReducers({
  counter,
  todos,
});

//두개의 리덕스 모듈을 만들었고 거기 있는 리듀서들을 합쳐서 루트 리듀서를 만듬
//그리고 루트 리듀서를 내보내줌
export default rootReducer;



```

​

리액트 프로젝트에 리덕스를 적용하는 방법 우선 다음 패키지를 설치하기위해 터미널에 해당 코드를 입력한다.

```bash
$ yarn add react-redux
```

그리고 index.js.에서 다음과 같이 코드를 작성해준다.

```react
//src/index.js

import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
// import "./exercise";
import { Provider } from "react-redux"; //리액트 리덕스에서 provider를 불러와서 리액트 프로젝트에서 리덕스를 적용할 수 있음
import { createStore } from "redux"; // createStore store를 만들어주는 리덕스의 함수
//createStore사용할 때는 reducer를 파라미터로 넣음
//reducer를 파라미터로 넣으려면 rootReducer를 불러와야함
import rootReducer from "./modules"; // 우리가 modules에서 index.js라는 이름으로 rootReducer를 내보내주었다.
//만약 src의 index.js에서 modules 디렉토리를 불러오게되면 바로 modules의 index.js를 바로 불러오게됨

//스토어만듬
const store = createStore(rootReducer);
console.log(store.getState());

//리액트 프로젝트에서 리덕스를 적용해보자!
//App컴포넌트를 Provider로 감싸줌
ReactDOM.render(
  // Provider의 props를 통해서 store 값을 설정해줌
  //이렇게 하면 리액트 컴포넌트 어디서든지 스토어를 사용할 수 있게 됨!
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

```

### 카운터 구현

리액트와 리덕스를 연동해서 카운터를 구현해보자!

우선 프레젠테이셔널 컴포넌트를 만든다.

```react
//components/Counter.js

//프레젠테이셔널 컴포넌트 -리덕스 스토어에 직접적으로 접근하지않고 필요한 값,함수를 props로 받아와서 사용하는 컴포넌트를 말한다.
//Counter.js => 프레젠테이셔널 컴포넌트

import React from "react";

function Counter({ number, diff, onIncrease, onDecrease, onSetDiff }) {
  //props를 통해서 상태와 함수들을 가져올것이다
  //우리가 여기에 인풋하나를 만들건데 input의 value를 onSetDiff안에 넣어 호출하기위해

  const onChange = (e) => {
    onSetDiff(parseInt(e.target.value, 10)); //e.target.value가 문자열이므로 숫자로 형변환해줘야함
  };
  return (
    <div>
      <h1>{number}</h1>
      <div>
        <input type="number" value={diff} onChange={onChange} />
        <button onClick={onIncrease}>+</button>
        <button onClick={onDecrease}>-</button>
      </div>
    </div>
  );
}
//프레젠테이셔널 컴포넌트의 개발이 끝났다.
//프레젠테이셔널 컴포넌트는 주로 UI를 선언하는것에 집중하고 필요한 함수나 값은 props로 받아와서 사용한다.

export default Counter;

```

그 다음 컨테이너 컴포넌트를 만든다.

```react
//containers/CounterContainer.js

//컨테이너 컴포넌트란 리덕스에 있는 상태를 조회하거나 액션을 디스패치 할수있는 컴포넌트를 의미한다.
import React from "react";
import Counter from "../components/Counter"; // Counter컴포넌트를 렌더링해줌
import { useSelector, useDispatch } from "react-redux"; //상태조회할때는 useSelector를 사용한다.
import { increase, decrease, setDiff } from "../modules/counter";

function CounterContainer() {
  const { number, diff } = useSelector((state) => ({
    //state가 가리키는 것은 우리가 만든 store에서 getState()해서 반환되는 객체:상태 이다.
    number: state.counter.number,
    diff: state.counter.diff,
  }));
  const dispatch = useDispatch(); //useDispatch(): 언제든지 dispatch를 통해 특정 액션을 발생시킬 수 있음

  const onIncrease = () => dispatch(increase()); //onIncrease가 호출되면 counter.js의 액션 생성함수 increase가 호출되면서 액션이 만들어지고 디스패치가 된다.
  const onDecrease = () => dispatch(decrease()); //onDecrease가 호출되면counter.js의 액션 생성함수 decrease가 호출되면 액션이 만들어지고 디스패치가 된다.
  const onSetDiff = (diff) => dispatch(setDiff(diff)); //onSetDiff가 호출되면 counter.js의 액션 생성함수 setDiff에 파라미터로 diff를 넣어주면서 diff값이 들어가있는 액션이 만들어지고 디스패치가 된다.

  // number, diff, onIncrease,onDecrease,onSetDiff 이것들을 그다음 Counter 컴포넌트에 하나하나 전달해줌
  return (
    <Counter
      number={number}
      diff={diff}
      onIncrease={onIncrease}
      onDecrease={onDecrease}
      onSetDiff={onSetDiff}
    />
  );
}
//컨테이너 컴포넌트 개발이 모두 끝났다.
//리액트 컴포넌트에서 리덕스를 연동할때는 useSelector, useDispatch라는 훅을 사용한다.
//useSelector는 상태를 조회하는 훅이다. 그래서 useSelector 안에 함수를 넣어주는데 해당함수의 파라미터에서는 state를 가져오는데 이 state가 바로 리덕스의 현재 상태이다.
//리덕스의 현재상태에서 어떤것을 불러올것이냐 했을 때 number랑 diff를 가져올거라해서 객체를 반환해주면 그것을 디스트럭쳐링 할당해 number와 diff를 추출해서 따로 사용가능하게 해주었다.
//그리고 액션을 디스패치할때는 useDispatch라는 훅을 사용하는데 그래서 useDispatch() 하게되면 dispatch 함수를 사용할 수 있게 됨
export default CounterContainer;

```

App 컴포넌트에서 CounterContainer 컴포넌트를 렌더링 한다.

```react
//App.js

import React from "react";
import CounterContainer from "./containers/CounterContainer";

function App() {
  return <CounterContainer />;
}

export default App;

```

리액트 컴포넌트에서 리덕스를 사용할때 프리젠테이셔널 컴포넌트와 컨테이너 컴포넌트를 분리해서 작업을 했는데 프리젠테이셔널 컴포넌트에서는 단순히 UI를 선언하는것에만 더 집중하고 상태관리는 컨테이너 컴포넌트에서 하도록 맡겨서 컨테이너 컴포넌트에서 리덕스 스토어 상태를 불러오고 어떤 액션 생성 함수가 호출되면 액션이 생성되는데 이 액션을 디스패치되는 작업을 처리해주었다.

프리젠테이셔널 컴포넌트에서는 버튼이 클릭되면 받아온 props를 호출하고 받아온 값을 특정 부분에서 보여주는 형태로 구현해주었다. 이렇게 둘을 분리해서 사용하면 프리젠테이셔널 컴포넌트의 재사용률도 높여줄수도 있고 관심사를 분리할 수 있기 때문에 굉장히 유용한 패턴이라고 할 수 있다.

이 패턴은 리덕스의 창시자가 공개한 것으로 리덕스를 사용하는 사용자에게 당연시 여겨져 왔다.
(리덕스 메뉴얼에서도 이에대해 언급되기도 함) 컨테이너와 프레젠테이셔널 컴포넌트를 나눈것은 복잡한 상태 로직을 분리시키기 위함이었는데 Hooks를 사용해도 비슷한 작업을 할 수 있다.

### **리덕스 개발자 도구 적용하기**

리덕스 개발자 도구를 사용하면 현재 스토어의 상태를 개발자 도구에서 조회할 수 있고 지금까지 어떤 액션들이 디스패치 되었는지 그리고 액션에 따라 상태가 어떻게 변하여왔는지 확인 할 수 있고 액션의 상태를 뒤로 되돌릴 수도 있으며 액션을 개발자도구에서 바로 디스패치 할 수도 있다.

1.브라우저에서 redux devtoos 확장 프로그램을 추가해준다.

2.터미널에서 아래 코드를 입력한다.

```bash
$ yarn add redux-devtools-extension
```

3.프로젝트 파일에서 composeWithDevTools함수를 불러와준다.

```react
import { composeWithDevTools } from "redux-devtools-extension"; //
```

4. createStore 안에 두번째 파라미터로 composeWithDevTools를 호출해준다.

```
const store = createStore(rootReducer, composeWithDevTools());
```

이렇게 하면 개발자도구에서 리덕스 상태를 조회하고 디스패치도 할 수 있고 디스패치된 모든 액션 내역을 볼 수 있다.

![](./img/image-20210105231942289.png)

<center>페이지에서 리덕스 개발자도구를 열면 나오는 화면</center>
