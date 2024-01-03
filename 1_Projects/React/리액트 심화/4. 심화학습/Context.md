> [!참고 자료]
> - [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)

리액트에서는 정보 전달을 위해 부모 컴포넌트에서 자식 컴포넌트로 prop을 사용한다.
그러나 많은 컴포넌트를 거치거나, 많은 컴포넌트에서 동일한 정보를 필요로 하는 경우 prop을 전달하는 것이 복잡해진다.
이를 해결하기 위해 **Context**를 사용한다.

> [!참고]
> [Consumer](https://react.dev/reference/react/createContext#consumer)를 사용하는 접근 방식도 있지만, 이 방식은 provider 없이 사용하는 경우에만 사용한다.
#### Step 1. Context 생성하기
`createContext` 사용해서 context를 생성하고 export 한다.

```javascript
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

#### Step 2. Context 사용하기
`useContext` Hook과 생성한 Context를 사용한다.

```jsx
import { useContext } from 'react';  
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {  
	const level = useContext(LevelContext);  
	// ...  
}
```

이후 `Heading` 컴포넌트가 아닌 `Section`이 level을 받는다.
```html
// AS_IS
<Section>
	<Heading level={4}>Sub-sub-heading</Heading>  
	<Heading level={4}>Sub-sub-heading</Heading>  
	<Heading level={4}>Sub-sub-heading</Heading>  
</Section>

// TO_BE
<Section level={4}>  
	<Heading>Sub-sub-heading</Heading>  
	<Heading>Sub-sub-heading</Heading>  
	<Heading>Sub-sub-heading</Heading>  
</Section>
```

Context를 사용하고 있지만, 제공하지 않았기 때문에 모든 Heading의 크기가 동일하다.
#### Step 3. Context 제공하기
```jsx
// AS_IS
export default function Section({ children }) {  
	return (  
		<section className="section">  
			{children}  
		</section>  
	);  
}

// TO_BE
import { LevelContext } from './LevelContext.js';  

export default function Section({ level, children }) {  
	return (  
		<section className="section">  
			<LevelContext.Provider value={level}>  
				{children}  
			</LevelContext.Provider>  
		</section>  
);  
}
```


### Context를 사용하기 전에 고려할 것
다음은 Context를 사용하기 전 고려해볼 몇 가지 대안들
1. Prop 전달하기로 시작하기
2. 컴포넌트를 추출하고 JSX를 children으로 전달하기

> [!warning] Context 사용에 있어 주의할 점
> - 자주 변경되는 경우에는 적합하지 않다