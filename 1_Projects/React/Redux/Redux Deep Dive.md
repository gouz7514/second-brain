- Reducer는  순수해야 하며, 부수 효과가 없고 동기적이어야 한다.
- HTTP 요청과  같은 비동기 코드는 Reducer 함수에 포함될 수 없다.

그렇다면 Side Effect나 비동기 작업을 어디에서 수행해야 할까?
- 동기적, 또는 side-effect free한 로직의 경우 : **reducer**
- 비동기, side effect 로직의 경우 : **action creators**, **components**

[리덕스 공식문서](https://redux.js.org/style-guide/#put-as-much-logic-as-possible-in-reducers)에서는 다음과 같이 설명하고 있다.

> [!참고] 
> **Reducer에 가능한 많은 로직을 추가해라**
> 액션을 준비하고 dispatch하는 코드가 아닌, reducer에 가능한 많은 로직을 포함시키는 것을 권장한다. 이를 통해 실제 앱의 로직에 대한 테스트를 수월하게 하고, 디버깅에 있어 효율성을 가져올 수 있으며, 데이터 변조나 버그와 같은 흔한 실수를 방지할 수 있다.
> 
> **Reducer는 상태의 모양(Shape)을 가지고 있어야 한다**
> redux의 root state는 하나의 reducer 함수에 의해 계산된다. 유지보수를 위해 각각의 "slice reducer"가 각자의 초기 상태와 이를 조작하는 로직을 갖게 된다.
> 
> 이에 추가로, slice reducer는 계산된 상태의 일부로 반환되는 다른 값에 대한 통제를 실행해아 한다.

