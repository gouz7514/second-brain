[Immer](https://github.com/immerjs/use-immer)는 편리하고, 변경 구문을 사용할 수 있게 해주며 복사문 생성을 도와주는 인기 있는 라이브러리
Immer를 사용하면 객체를 변경하는 것처럼 보일 수 있다.

```javascript
updatePerson(draft => {  
	draft.artwork.city = 'Lagos';  
});
```