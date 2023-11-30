
![[Pasted image 20231123170700.png]]
위와 같은 경우 모달이 페이지 위에 떠 있는 상황인데, 구조적으로 좋지 않다.
다른 HTML 코드 안에 위치하고 스타일링, 접근성 관점에서 좋지 않다
ex) 스크린 리더가 해석할 때 오버레이라고 인식할 수 없다.
이를 해결하기 위해 포탈을 사용한다

포탈을 사용하는 방법은 다음과 같다
#### 1. 컴포넌트를 이동시킬 장소를 만든다
#### 2. react-dom 삽입
#### 3. createPortal
https://react.dev/reference/react-dom/createPortal
```jsx
<div>  
	<SomeComponent />  
	{createPortal(children, domNode, key?)}  
</div>
```
- `children` : 렌더링되어야 하는 리액트 노드
- `domNode` : 렌더링될 실제 DOM 영역

> [!warning] Portal을 사용할 때 주의점
> 애플리케이션의 접근성이 준수되는지 확인하는 것이 중요하다.
> 모달을 만들 때는 [WAI-ARIA 모달 제작 관행](https://www.w3.org/WAI/ARIA/apg/#dialog_modal)을 따르는 것이 중요하다.

> [!info] WAI-ARIA
> Web Accessibility Initiative - Accessible Rich Internet Application
