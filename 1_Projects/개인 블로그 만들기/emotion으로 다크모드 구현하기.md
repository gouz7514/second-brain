### 알아야 할 개념 정리
- [ThemeProvider](https://emotion.sh/docs/theming)
- [emotionCache](https://emotion.sh/docs/@emotion/cache)
	- 요 놈은 스타일용 캐시를 만들어서 다른 곳에서 사용할 때 쓰인다. 여기서는 패스!

### rough
- 다크모드 훅 구현하기
- 로컬 스토리지에 값 저장하고 불러오기
	- 클라이언트 단에서 동작해야해서 flickering 발생 -> 어떻게 해결하지?
	- ~~getInitialProps~~


https://voyage-dev.tistory.com/134

위 블로그 설명에 따르면
#### theme.ts 색상 지정
#### `_app.tsx` 에  ThemeProvider로 제공
`ThemeProvider`를 사용하면 `props.theme`을 이용해 테마 값에 접근할 수 있다.
#### useDarkmode 훅
- 다크모드를 사용하려면 localStorage와 window 객체가 필요
- 브라우저 환경에서 접근해야 하므로 useEffect를 활용한다

#### ThemeContext를 사용해 다크모드 값을 넘겨준다
#### 새로고침 시 flickering 이슈
`_document.ts`에  dangerouslySetInnerHTML을 사용해 data-theme 설정