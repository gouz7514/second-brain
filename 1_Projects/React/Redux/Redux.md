### React Context의 단점
- 여러 개의 context가 필요한 경우 코드 복잡도가 증가할 수 있다.
- 테마 변경과 같은 빈도가 낮은 업데이트의 경우에는 좋지만, 데이터가 자주 변경되는 경우에는 좋지 않다


### Redux 작동 방식
[Redux 공식 문서](https://ko.redux.js.org/introduction/getting-started/)
![[Pasted image 20240103143231.png]]

> [!Reducer Function?]
> - 입력을 받아서 그 입력을 변환하고 줄이는 함수
> - 입력을 변환해서 새로운 결과를 반환한다.

다음과 같은 점을 주의해야 한다.
- redux 상태를 절대로 직접 수정해서는 안된다.
- 상태를 변경하는 유일한 방법은 "무엇인가 일어났음을 의미하는" `action` 객체를 생성하고, 어떤 일이 일어났는지 알려주도록 action을 `dispatch`하는 것이다.
- action이 dispatch되면, store는 루트 `reducer` function을 동작시키고, 기존 상태와 액션을 기반으로 새로운 상태를 계산하도록 한다.
- 마지막으로, store는 `subscriber`에게 상태가 변경되었음을 알려 UI가 변경될 수 있도록 한다.


### 리액트 컴포넌트에서 리덕스 사용하기
[참고 링크](https://react-redux.js.org/introduction/getting-started)
#### react-redux 설치
#### 상태 생성하기
```javascript
// store/index.js
import { createStore } from 'redux'

const counterReducer = (state = { counter: 0 }, action) => {
	if (action.type === 'increment') {
		return {
			counter: state.counter + 1
		}
	}
  
	if (action.type === 'decrement') {
		return {
			counter: state.counter - 1
		}
	}
  
	return state
}

const store = createStore(counterReducer)

export default store
```

#### 리액트 앱에 상태 전달하기
```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';

import { Provider } from 'react-redux';
import store from './store';

import './index.css';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
	<Provider store={store}>
		<App />
	</Provider>
);
```

#### 컴포넌트에서 action을 dispatch하기
```jsx
import { useSelector, useDispatch } from 'react-redux';

const Counter = () => {
	const counter = useSelector(state => state.counter);
	const dispatch = useDispatch();

	const incrementHandler = () => {
		dispatch({ type: 'increment' });
	};

	const decrementHandler = () => {
		dispatch({ type: 'decrement' });
	};

	return (
		<main>
			<div>{ counter }</div>
			<div>
				<button onClick={incrementHandler}>Increment</button>
				<button onClick={decrementHandler}>Decrement</button>
			</div>
		</main>
	)
}
```

> [!warning] 절대로 state를 변경해서는 안된다.
> 상태를 변경할 떄는 새로운 state 객체를 반환해야 한다.