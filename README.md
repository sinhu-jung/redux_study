## React-Redux

+ Redux는 기본적으로 flux 패턴을 따름

  ```bash
  Action -> Dispatch -> Store -> View
  ```

  + redux의 데이터 흐름은 동일하게 단방향으로 `view(컴포넌트)`에서 `Dispatch(store에서 주는 state를 바꾸는 함수)`라는 함수를 동해 `action(디스 패치 함수 이름)`이 발동되고 `reducer`에 정의된 로직에 따라 `store의 state`가 변화하고 그 `state`를 쓰는 `view(컴포넌트)`가 변하는 흐름을 따릅니다.



#### 패키지 설치

+ `redux-devtools-extension`은 크롬 확장 프로그램에서 `redux dev tools`를 통해 설치 할 수 있고, redux의 데이터 흐름을 알아보기 쉽게 하기 위해 사용합니다.

+ `redux-logger`는 redux를 통해 바뀔 이전 state, dispatch 실행으로 인해 바뀐 state가 콘솔에 찍혀 디버깅 쉽게 해주는 라이브러리입니다.

```ba
npm i redux react-redux redux-devtools-extension redux-logger
```



#### reducer 정의

+ reducer는 `store`에 들어갈 state와 state를 바꿀 함수를 정의하는 곳입니다.

+ 기본적으로 순수함수로 코딩하고, 불변성을 지켜야 합니다.

  #### 불변성을 지켜야 하는 이유

  + **불변성을 지켜야하는 이유**는 redux는 이전 state와 바뀐 state를 구분하는 방법이 참조값이 바뀌었는지 확인하고, 참조값이 바뀌면, state가 바뀌었다고 redux가 인식하여, 해당 state를 사용하는 컴포넌트에게 리렌더링을 요청하기 때문입니다.
    + 그렇기 때문에, `state.test = action.test`와 같이 직접적으로 state를 변경하면 참조값이 변하지 않아 redux는 값이 바뀌었다고 인식하지 않고 리렌더링 되지 않습니다.
    + `state.test = {...test, action.test}`
    + 또는 `immer`라는 라이브러리를 사용하여 쉽게 불변성을 유지합니다.

1.  rootReducer 정의

   ```react
   // reducers/index.js
   
   /** root reducer */
   import { combineReducers } from "redux";
   import counter from "./counter";
   
   // 여러 reducer를 사용하는 경우 reducer를 하나로 묶어주는 메소드입니다.
   // store에 저장되는 리듀서는 오직 1개입니다.
   const rootReducer = combineReducers({
     counter
   });
   
   export default rootReducer;
   ```

2. 세부 reducer를 정의합니다.

   ```react
   // reducers/Counter.js
   
   // reducer가 많아지면 action상수가 중복될 수 있으니
   // 액션이름 앞에 파일 이름을 넣습니다.
   export const INCRESE = "COUNT/INCRESE";
   
   export const increseCount = count => ({ type: INCRESE, count });
   
   const initalState = {
     count: 0
   };
   
   const counter = (state = initalState, action) => {
     switch (action.type) {
       case INCRESE:
         return {
           ...state,
           count: action.count + 1
         };
   
       // default를 쓰지 않으면 맨처음 state에 count값이 undefined가 나옵니다 꼭! default문을 넣으세요
       default:
         return state;
     }
   };
   
   export default counter;
   ```

3. app에 store에 넣고 만든 reducer 반영

   ```react
   import React from 'react';
   import ReactDOM from 'react-dom';
   import { createStore, applyMiddleware, compose } from "redux";
   import { Provider } from "react-redux";
   import logger from "redux-logger";
   import { composeWithDevTools } from "redux-devtools-extension";
   
   import App from './App';
   import rootReducer from './redux/reducers'
   
   // 배포 레벨에서는 리덕스 발동시 찍히는 logger를 사용하지 않습니다.
   const enhancer =
     process.env.NODE_ENV === "production"
       ? compose(applyMiddleware())
       : composeWithDevTools(applyMiddleware(logger));
   
   // 위에서 만든 reducer를 스토어 만들때 넣어줍니다
   const store = createStore(rootReducer, enhancer);
   
   ReactDOM.render(
     <React.StrictMode>
       <Provider store={store}>
         <App />
       </Provider>
     </React.StrictMode>,
     document.getElementById('root')
   );
   
   ```

4. 컴포넌트에서 redux 사용

   ```react
   import { useSelector, useDispatch } from "react-redux";
   import { increseCount } from "./redux/reducers/counter.js";
   
   const Counter = () => {
   
     // dispatch를 사용하기 위한 준비
     const dispatch = useDispatch();
   
     // store에 접근하여 state 가져오기
     const { count } = useSelector(state => state.counter);
   
     const increse = () => {
       // store에 있는 state 바꾸는 함수 실행
       dispatch(increseCount(count));
     };
   
     return (
       <div>
         {count}
         <button onClick={increse}>증가</button>
       </div>
     );
   };
   
   export default Counter;
   ```

   
