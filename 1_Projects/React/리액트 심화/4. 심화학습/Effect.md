리액트의 역할은
- UI를 렌더링하고
- 사용자 입력에 반응하는 것

이외의 로직들은 모두 Side Effect(or Effect)라고 칭할 수 있다.
- Http request
- 스토리지에 데이터 저장
- 어떤 액션에 대해 응답으로 실행되는 액션이 있다면 모두 Side Effect

이런 로직들을 핸들링하기 위해 useEffect 훅을 사용한다.
(참고 링크 : [Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects))

> - Effect는 한 마디로 리액트 컴포넌트에서 일어나는 특정 사건에 반응해 발생하는 코드 조각이라고 할 수 있다.
> - 또한, Effect는 주로 리액트 코드를 벗어난 특정 외부 세스템과 동기화하기 위해 사용한다.
> - Effect는 렌더링 뒤에 실행된다.

### [useEffect](https://react.dev/reference/react/useEffect)
useEffect는 다음과 같은 파라미터를 받는다
- `setup`
	- Effect 로직 함수. setup은 **클린업 함수**를 리턴할 수도 있다.
	- 변경된 의존성에 대해 리렌더링이 발생할 때, 이전 값을 가지고 **클린업 함수**가 실행되고, 컴포넌트가 DOM에서 제거될 때 **클린업 함수**가 실행된다.
	- **클린업 함수**는 모든 **새로운** 사이드이펙트함수가 실행되기 전에 실행된다.
- `dependencies` (optional)
	- setup 함수 내부에서 참조되는 모든 반응형 값들이 포함된 배열
	- React는 각각의 의존성들을 Object.is 비교법을 통해 이전 값과 비교한다.
	- Effect로 사용되는 모든 것을 종속성으로 추가해야 한다. 즉, 구성 요소가 다시 렌더링 되어 변경될 수 있는 경우만 종속성으로 추가되어야 한다.

> [!warning] 주의사항
> - 만약 dependencies가 객체이거나 컴포넌트 내부에 선언된 함수인 경우 Effect가 필요 이상으로 재실행될 수 있다. 이 경우 불필요한 의존성을 제거하거나, "이전 상태를 기반으로 상태를 업데이트"할 수 있다.
> - Effect로 수행하는 작업이 시각적인 작업을 수행하고 지연이 눈에 띄게 발생한다면 `useLayoutEffect`를 사용해라

useEffect는 다음과 같은 경우에 사용할 수 있다
- 외부 시스템과 연결
- 커스텀 Hook을 Effect로 감싸기
- 리액트로 작성되지 않은 위젯 제어
- 데이터 페칭 등

참고 자료
- [React Hooks - Understanding Component Re-renders](https://medium.com/@guptagaruda/react-hooks-understanding-component-re-renders-9708ddee9928)
- [Writing Resilient Components](https://overreacted.io/writing-resilient-components)