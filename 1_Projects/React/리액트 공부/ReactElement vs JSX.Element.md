https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts

---
먼저 ReactNode에 대해 알아보자

## `React.ReactNode`
```typescript
type ReactNode = | ReactElement | string | number | Iterable<ReactNode> | ReactPortal | boolean | null | undefined
```
- `ReactNode`는 `ReactElement`를 비롯해 대부분의 자바스크립트 데이터 타입을 아우르는 범용적인 타입이다. ([DefinitelyTyped - React ReactNode](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts#L478-L489))
```typescript
type PropsWithChildren<P = unknown> = P & { children?: ReactNode | undefined };
```
- `PropsWithChildren`의 `children` 의 기본 타입이다. ([DefinitelyTyped - React PropsWithChildren](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts#L1586))
- 거의 모든 타입을 할당할 수 있다.

## `ReactElement` vs `JSX.Element`
`ReactElement`와 `JSX.Element`의 공통점은 모두 `React.createElement`의 리턴 값이라는 점이다.
### `React.createElement`
`createElement`를 사용하면 React 엘리먼트를 생성할 수 있다.

#### `createElement(type, props, ...children)`

**매개변수**
- `type` 
- `props`
- `(optional) ...children`

**반환값**
아래 프로퍼티를 가지는 React 엘리먼트 객체를 반환한다.
- `type` : 전달받은 `type`
- `props` : `ref`, `key`를 제외한 전달받은 `props`
- `ref` : 전달받은 `ref`, 없을 경우 `null`
- `key` : 전달받은 `key`, 없을 경우 `null`

### `ReactElement`
https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts#L312C5-L334
```typescript
/**
     * Represents a JSX element.
     *
     * Where {@link ReactNode} represents everything that can be rendered, `ReactElement`
     * only represents JSX.
     *
     * @template P The type of the props object
     * @template T The type of the component or tag
     *
     * @example
     *
     * ```tsx
     * const element: ReactElement = <div />;
     * ```
     */
interface ReactElement<
        P = any,
        T extends string | JSXElementConstructor<any> = string | JSXElementConstructor<any>,
    > {
        type: T;
        props: P;
        key: string | null;
    }
```
- `ReactElement`는 React의 엘리먼트를 위한 React 타입이다.
- type이 받는 T 제너릭은 해당 HTML 태그의 타입을 받고, props는 그 외의 컴포넌트가 갖고 있는 속성을 받는다.
- `ReactElement`는 JSX element만을 말한다

### `JSX.Element`
https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts#L4247
```typescript
declare global {
	namespace JSX {
		interface Element extends React.ReactElement<any, any> {}
	}
}
```
- `JSX.Element`는 JSX 를 위한 TS type
- ReactElement의 타입과 props를 모두 any로 확장한 인터페이스

---
### 참고한 글들
- https://www.howdy-mj.me/react/react-node-and-jsx-element
- https://velog.io/@hanei100/ReactElement-vs-ReactNode-vs-JSX.Element
- https://stackoverflow.com/questions/58123398/when-to-use-jsx-element-vs-reactnode-vs-reactelement
- https://merrily-code.tistory.com/209
- https://github.com/microsoft/TypeScript/issues/21699