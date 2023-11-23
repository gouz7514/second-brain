#용어 

https://legacy.reactjs.org/docs/faq-internals.html

https://callmedevmomo.medium.com/virtual-dom-react-%ED%95%B5%EC%8B%AC%EC%A0%95%EB%A6%AC-bfbfcecc4fbb

---

- Virtual DOM은 UI의 이상적인 또는 "가상"의 표현을 메모리에 저장하고 ReactDOM과 같은 라이브러리에 의해 "실제" DOM과 동기화하는 프로그래밍 개념이다. 이 과정을 [[Reconciliation]]이라고 한다.
- 이러한 접근방식이 React의 선언적 API를 가능하게 한다. React에게 원하는 상태의 UI를 알려주면 DOM이 그 상태와 일치하도록 한다
- 즉, 실제 DOM을 조작하는 것이 아닌 가상돔을 조작하고 이를 실제 DOM에 적용
