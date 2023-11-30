https://react.dev/learn/referencing-values-with-refs
컴포넌트로 하여금 일부 정보를 기억하면서 렌더링을 유발하지 않는 방법

```javascript
import { useRef } from 'react';

const ref = useRef(0);
```

`useRef`는 다음과 같은 객체를 반환한다.

```javascript
{
	current: 0,
}
```

`ref.current` 프로퍼티를 통해 값에 접근할 수 있다. 이 값은 React가 추적하지 않는 일종의 비밀 주머니이다. 이것이 바로 React의 단방향 데이터 흐름에서 "escape hatch"가 된다.

DOM을 조작하는 것은 바람직하지 않지만 값을 재설정하는 경우는 사용해도 된다..?