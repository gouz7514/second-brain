
https://github.com/vercel/next.js/discussions/39821
### `_app_`
https://nextjs.org/docs/pages/building-your-application/routing/custom-app
- Global React and Next.js initialization
- 서버로 요청이 들어왔을 때 가장 먼저 실행되는 컴포넌트
- 모든 초기화 과정의 루트
- 페이지에 적용할 공통 레이아웃

> [!info] `_app`은 다음과 같은 경우 사용한다
> - 공통 레이아웃
> - 페이지에 추가 데이터 주입
> - global css

### `_document`
[nextjs docs - Custom Document](https://nextjs.org/docs/pages/building-your-application/routing/custom-document)

- `_app` 다음에 실행
- Global HTML initialization

> [!info] `_document`는 다음과 같은 특징을 가진다.
> - 서버 사이드에서만 동작한다
> - 페이지가 제대로 렌더링되려면 `<Html>`, `<Head />`, `<Main />` , `<NextScript />`가 필요하다