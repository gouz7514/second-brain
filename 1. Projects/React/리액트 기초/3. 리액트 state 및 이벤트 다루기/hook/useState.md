https://react.dev/reference/react/useState
https://ko.react.dev/learn/state-a-components-memory#meet-your-first-hook

### 사용법
```javascript
import { useState } from 'react'

const [index, setIndex] = useState(0);
```

### 해부하기
1. **컴포넌트가 처음 렌더링된다.**
2. **state를 업데이트힌디.** 사용자가 버튼을 클릭하면 `setIndex(index + 1)`을 호출한다. 이는 React에 `index`는 `1`임을 기억하게 하고 또 다른 렌더링을 유발한다.
3. **컴포넌트가 두 번째로 렌더링된다.** React는 여전히 `useState(0)`를 보지만, `index`를 `1`로 설정한 것을 기억하고 있기 때문에, 이번에는 `[1, setIndex]`를 반환한다.

여러 개의 state 변수를 사용하는 경우 [[state 구조 선택]]을 확인!

> [!faq] React는 어떤 state를 반환할지 어떻게 알 수 있을까?
> 훅은 **동일한 컴포넌트의 모든 렌더링에서 안정적인 호출 순서에 의존한다.**
> 최상위 수준에서 훅을 호출해야하는 규칙만 잘 따르면, 훅은 항상 같은 순서로 호출된다
> 
> 내부적으로 React는 모든 컴포넌트에 대해 한 쌍의 state 배열을 가진다.
> 또한 렌더링 전에 0으로 설정된 현재 인덱스 쌍을 유지한다
> `useState`를 호출할 때마다, React는 다음 state 쌍을 제공하고 인덱스를 증가시킨다
> 
> 실제 코드는 현재 훅이 다음 훅을 기억하고 이를 가리키는 방식으로 이루어진다

### 자주하는 실수
#### 이전 상태를 기반으로 업데이트할 때는 함수형 표현을 써야 한다
```javascript
const [count, setCount] = useState(0)

function onClickCounter(amount) {
	// setCount((count) => count + adjustment)
	setCount(count + adjustment) // 이 방식은 몇 번을 호출하든 결국 1만 변경됨
}
```

리액트는 상태 업데이트를 예약한다
따라서 이론적으로 함수형이 아닌 접근법을 사용하면 오래되었거나 잘못된 상태 스냅샷에 의존하게 될 수도 있다
함수형 표현은 내부 함수에서 제공하는 스냅샷이 항상 최신 상태 스냅샷이 되도록 보장해 준다.

#### 상태 업데이트는 바로 반영되지 않는다
컴포넌트가 리렌더링될때까지 기다려야 한다
정확히는 **useState 훅 자체는 동기 함수**이다. 그러나, 리렌더링이 일어나는 과정이 비동기이다.
업데이트 내용을 큐에 담아서 한번에 업데이트하기 때문에.

#### primitive vs non-primitive
non primitive 즉, 참조 타입은 참조에 의한 전달을 하므로 같은 값이어도 다른 객체이다.
따라서, 같은 값을 전달해도 리렌더링이 발생함
**특히, useEffect에 인자로 넘길 때 object를 넘기지 않도록 주의해야 한다!**

---
참고 링크

![](https://www.youtube.com/watch?v=GGo3MVBFr1A)