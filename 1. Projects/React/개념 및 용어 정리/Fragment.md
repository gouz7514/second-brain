#컴포넌트

https://ko.react.dev/reference/react/Fragment

---

- 하나의 엘리먼트가 필요한 상황에서 엘리먼트를 `<Fragment>`로 감싸서 그룹화할 수 있다.
- `Fragment` 안에서 그룹화된 엘리먼트는 DOM 결과물에 영향을 주지 않는다. 즉, 엘리먼트가 그룹화되지 않은 것과 같다.

### 주의 사항
- Fragment에 key를 사용하려면 명시적으로 Fragment를 import해서 렌더링해야 한다.
- state가 초기화되는 경우가 코드에 따라 다르다. 정확한 내용은 [여기](https://gist.github.com/clemmy/b3ef00f9507909429d8aa0d3ee4f986b)를 참고
