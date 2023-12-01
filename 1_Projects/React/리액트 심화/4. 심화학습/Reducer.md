> [!참고 자료]
> - [Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer)

state가 많아지면 로직도 복잡해지고 관리가 힘들어진다.
이를 통합해(consolidate)서 관리하기 위해 **reducer**를 사용할 수 있다.

특히, 다른 상태를 기반으로 상태를 업데이트하는 경우는 바람직하지 않다!
따라서, 함수형 표현을 써야하는데 이 경우 다른 상태를 기반으로 하기 때문에 불가능하다.

reducer를 사용하기 위해 `useReducer` 훅을 사용한다.
### [useReducer](https://react.dev/reference/react/useReducer)

#### Step 1. state를 설정하는 작업에서 action을 dispatch 함수로 전달하는 것으로 **바꾸기**
ex) task manager 작성
```javascript
// AS_IS
function handleAddTask(text) {
	setTasks([
	...tasks,
	{
		id: nextId++,
		text: text,
		done: false,
	},
	]);
}
  
function handleChangeTask(task) {
	setTasks(
		tasks.map((t) => {
			if (t.id === task.id) {
				return task;
			} else {
				return t;
			}
		})
	);
}
  
function handleDeleteTask(taskId) {
	setTasks(tasks.filter((t) => t.id !== taskId));
}
```

state를 설정해 React에게 "무엇을 할 지" 지시하는 대신, 이벤트 핸들러에서 "action"을 전달해 "사용자가 방금 한 일"을 지정한다.

```javascript
// TO_BE
function handleAddTask(text) {  
	dispatch({  
		type: 'added',  
		id: nextId++,  
		text: text,  
	});  
}  

function handleChangeTask(task) {  
	dispatch({  
		type: 'changed',  
		task: task,  
	});  
}  

function handleDeleteTask(taskId) {  
	dispatch({  
		type: 'deleted',  
		id: taskId,  
	});  
}
```

`dispatch`에 전달되는 객체를 "action"이라고 한다.

#### Step 2. reducer 함수 **작성하기**
reducer 함수는 state에 대한 로직을 넣는 곳.
이 함수는 현재 state 값과 action 객체를 인자로 받고 다음 state값을 반환한다.

1. 첫번째 인자에 현재 state 선언
2. 두번째 인자에 `action`  객체 선언
3. 다음 state 반환하기 (React가 state에 설정하게 될 값)

```javascript
function tasksReducer(tasks, action) {  
	switch (action.type) {  
		case 'added': {  
			return [
				...tasks, {  
					id: action.id,  
					text: action.text,  
					done: false  
				}
			];  
		}  
		case 'changed': {  
			return tasks.map(t => {  
				if (t.id === action.task.id) {  
					return action.task;  
				} else {  
					return t;  
				}  
			});  
		}  
		case 'deleted': {  
			return tasks.filter(t => t.id !== action.id);  
		}  
		default: {  
			throw Error('Unknown action: ' + action.type);  
		}  
	}  
}
```

이렇게 작성된 reducer 함수를 컴포넌트 바깥에 정의할 수 있다.

> [!faq] 왜 reducer라고 부를까?
> 컴포넌트 내 코드의 양을 줄이기도 하지만, 배열에서 사용하는 `reduce()` 연산에서 이름을 따 왔다.
> 
> `reduce`는 지금까지의 결과와 현재 아이템을 인자로 받고 다음 결과를 반환한다.
> 비슷하게 React의 reducer는 지금까지의  state와 action을 인자로 받고 다음 state를 반환한다. 이 과정에서 여러 

#### Step 3. 컴포넌트에서 reducer **사용하기**

```javascript
import { useReducer } from 'react'

const [tasks, dispatch] = useReducer(tasksReducer, initialTasks)
```


---
이렇게 `useReducer`를 사용하고 `useEffect`를 사용하는 경우, 이펙트의 불필요한 실행을 방지할 수 있다.

```javascript
const [emailState, dispatchEmail] = useReducer(emailReducer, {
	value: '',
	isValid: null
})
  
const [passwordState, dispathPassword] = useReducer(passwordReducer, {
	value: '',
	isValid: null
})
const [formIsValid, setFormIsValid] = useState(false);
  
const { isValid: emailIsValid } = emailState
const { isValid: passwordIsValid } = passwordState
  
useEffect(() => {
	const identifier = setTimeout(() => {
		console.log('Checking form validity!')
		setFormIsValid(
			emailIsValid && passwordIsValid
		)
	}, 500)
  
	return () => {
		console.log("CLEANUP")
		clearTimeout(identifier)
	}
}, [emailIsValid, passwordIsValid])
```


### `useState`와 `useReducer` 비교하기
- code size
	- `useReducer`는 reducer 함수, action을 모두 작성해야 하지만, 여러 이벤트 핸들러에서 비슷한 방식을 사용하는 경우 `useReducer`를 사용하면 코드의 양을 줄일 수 있다.
- 디버깅
	- `useReducer`를 사용하면 각 reducer에 console log를 추가해 확인할 수 있지만 디버깅이 조금 더 복잡할 수도 있다.