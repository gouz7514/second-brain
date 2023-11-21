https://ko.react.dev/learn/choosing-the-state-structure

State를 잘 구조화하면 수정과 디버깅이 즐거운 컴포넌트를 만들 수 있다!

> [!state를 구조화할 때 고려해야 할 팁들]
> - 단일 vs 다중 state 변수를 사용하는 경우
> - State를 구성할 때 피해야 할 사항
> - 상태 구조의 일반적인 문제를 해결하는 방법

### State 구조화 원칙
#### 1. 연관된 state 그룹화하기
두 개 이상의 state 변수를 항상 동시에 업데이트한다면, 단일 state 변수로 병합하는 것을 고려해라

```javascript
// AS_IS
const [x, setX] = useState(0);  
const [y, setY] = useState(0);

// TO_BE
const [position, setPosition] = useState({ x: 0, y: 0 });
```

> [!warning] State 변수가 객체인 경우 다른 필드를 명시적으로 복사하지 않고 [하나의 필드만 업데이트할 수 없다](https://ko.react.dev/learn/updating-objects-in-state)

#### 2. state의 모순 피하기
여러 state 조각이 서로 모순되고 "불일치"할 수 있는 방식으로 state를 구성하는 것은 실수가 발생할 여지를 만든다.

#### 3. 불필요한 state 피하기
렌더링 중에 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있다면, 컴포넌트의 state에 해당 정보를 넣지 않아야 한다.

> [!tip] Props를 state에 미러링해서는 안된다.
> State는 첫번째 렌더링 중에만 초기화 되기 때문에 전달받은 props를 그대로 state에 저장하면(미러링) 나중에 다른 값의 props가 전달될 때 state 변수가 업데이트되지 않는다.
> 
> Props를 상태로 "미러링"하는 것은 특정 prop에 대한 모든 업데이트를 무시하기를 원할 때에만 의미가 있다. 이 경우, 관례에 따라 prop의 이름을 `initial` 또는 `default`로 시작하여 새로운 값이 무시됨을 명확히 한다.

#### 4. State의 중복 피하기
여러 상태 변수 간 또는 중첩된 객체 내에서 동일한 데이터가 중복될 경우 동기화를 유지하기 어렵다.

#### 5. 깊게 중첩된 state 피하기
깊게 계층화된 state는 업데이트하기 쉽지 않다.
**만일 state가 쉽게 업데이트하기에 너무 중첩되어 있다면, "평탄"하게 만드는 것을 고려해라**

이상적으로 메모리 사용량을 개선하기 위해서는 삭제한 항목을 "테이블" 객체에서 제거해야 한다.
또한 업데이트 로직을 간결하게 하기 위해 [[Immer로 간결한 갱신 로직 작성하기]]를 사용한다.