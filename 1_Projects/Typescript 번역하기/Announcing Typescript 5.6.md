> [!tip] 공식 블로그 작성일 : 2024.09.09
> [원문 링크](https://devblogs.microsoft.com/typescript/announcing-typescript-5-6/)

Typescript 5.6 베타 버전 이후로, [Typescript의 언어 서비스가 `tsconfig.json` 파일을 검색하는 방식과 관련된 변경 사항](https://devblogs.microsoft.com/typescript/announcing-typescript-5-6-beta/#search-ancestor-configuration-files-for-project-ownership)을 revert했다.  
이전에는 `tsconfig.json`이라는 이름을 가진 모든 프로젝트 파일을 찾기 위해 계속해서 탐색했다. 이로 인해 많은 참조된 프로젝트가 열릴 수 있기 때문에, [이 동작을 되돌렸으며](https://github.com/microsoft/TypeScript/pull/59634) [Typescript 5.7에서 이를 다시 도입할 방법](https://github.com/microsoft/TypeScript/pull/59688)을찾고 있다.

추가로, 베타에 비해 몇몇 새로운 타입의 이름이 변경되었다. 이전에 Typescript는 `Iterator.prototype`을 기반으로 하는 모든 값을 설명하기 위해 `BuiltinIterator`라는 단일 타입을 제공했다. 이 타입의 이름이 `IteratorObject`로 변경되었으며, 다른 타입 파라미터를 가지게 되었다. 또한 `ArrayIterator`, `MapIterator` 등의 여러 하위 타입이 추가되었다.

`--stopOnBuildErrors`라는 이름의 새로운 플래그가 `--build` 모드에 추가되었다. 프로젝트 빌드시에 에러가 발생하면, 다른 프로젝트도 빌드를 중단할 것입니다. 이 플래그는 Typescript 5.6 이전 버전의 동작고 유사한 기능을 제공한다. [Typescript 5.6에서는 에러가 발생하더라도 항상 빌드를 진행하기 때문](https://devblogs.microsoft.com/typescript/announcing-typescript-5-6/#allow---build-with-intermediate-errors)이다.

새로운 에디터 기능이 추가되어 [커밋 문자에 대한 직접 지원](https://devblogs.microsoft.com/typescript/announcing-typescript-5-6/#granular-commit-characters)과 [자동 임포트에서 제외 패턴을 설정](https://devblogs.microsoft.com/typescript/announcing-typescript-5-6/#exclude-patterns-for-auto-imports)할 수 있다.

## Disallowed Nullish and Truthy Checks

정규식을 작성한 뒤 `.test(...)`를 호출하지 않은 적이 있을 것이다
```javascript
if (/0x[0-9a-f]/) {
    // 아래 로직은 항상 동작한다
    // ...
}
```

혹은 `>=` 대신 `=>`를 작성할 수도 있다
```javascript
if (x => 0) {
    // 아래 로직은 항상 동작한다
    // ...
}
```

혹은 복잡한 표현식에서 괄호를 잘못 사용했을 수 있다.
```javascript
if (
    isValid(primaryValue, "strict") || isValid(secondaryValue, "strict") ||
    isValid(primaryValue, "loose" || isValid(secondaryValue, "loose"))
) {
    //                           ^^^^ 👀 괄호가 닫히지 않음
}
```

위 모든 예시는 작성자가 의도한 대로 동작하지 않지만, 유효한 자바스크립트 코드이다. Typescript 또한 이전에는 이런 예시들이 큰 문제가 없었다.

그러나 약간의 실험을 통해, 위의 예시같은 많은 버그들을 잡아낼 수 있다는 것을 발견했다. Typescript 5.6에서는, 컴파일러가 특정 조건이 항상 참(truthy) 또는 nullish로 평가될 것을 문법적으로 확인할 수 있을 때 에러를 발생시킨다. 즉 위 예시들에서는 다음과 같은 에러를 볼 수 있게 된다.

```javascript
if (/0x[0-9a-f]/) {
//  ~~~~~~~~~~~~
// error: This kind of expression is always truthy.
}

if (x => 0) {
//  ~~~~~~
// error: This kind of expression is always truthy.
}

function isValid(value: string | number, options: any, strictness: "strict" | "loose") {
    if (strictness === "loose") {
        value = +value
    }
    return value < options.max ?? 100;
    //     ~~~~~~~~~~~~~~~~~~~
    // error: Right operand of ?? is unreachable because the left operand is never nullish.
}

if (
    isValid(primaryValue, "strict") || isValid(secondaryValue, "strict") ||
    isValid(primaryValue, "loose" || isValid(secondaryValue, "loose"))
) {
    //                    ~~~~~~~
    // error: This kind of expression is always truthy.
}

```

ESLint의 `no-constant-binary-expression` rule을 사용해 비슷한 결과를 얻어낼 수 있으며, [ESLint 블로그 포스트에서 몇몇 성과](https://eslint.org/blog/2022/07/interesting-bugs-caught-by-no-constant-binary-expression/)에 대해 확인할 수 있습니다. 그러나 Typescript가 수행하는 새로운 검사 방식은 ESLint rule과 완전히 일치하지 않으며, 이런 검사 기능이 Typescript 자체에 내장되어 있는 것이 유용하다고 생각합니다.

몇몇 표현식은 항상 truthy 혹은 nullish 하더라도 허락될 수 있습니다. 특히 `true`, `false`, `0` 그리고 `1`은 항상 truthy 혹은 falsy 하더라도 다음 코드와 같이 허락됩니다.

```javascript
while (true) {
    doStuff();

    if (something()) {
        break;
    }

    doOtherStuff();
}
```

여전히 유용한 코드이며,
```javascript
if (true || inDebuggingOrDevelopmentEnvironment()) {
    // ...
}
```
위 코드 또한 terating/debugging에 유용한 코드입니다.

검사 실행 방식 또는 잡아낼 수 있는 버그에 대해서 궁금하다면 [해당 기능에 대한 PR](https://github.com/microsoft/TypeScript/pull/59217)을 살펴보자.

## Iterator Helper Methods
JavaScript는 `iterables`와 `iterator` 라는 개념을 가지고 있다
- `iterables` : `[Symbol.iterator]()`를 호출함으로써 iterate하면서 iterator를 가져올 수 있는 것
- `iterator` : iterate 과정에서 다음 값을 가져오기 위해 사용할 수 있는 `next()` 메소드를 갖는 것

대체로, `for / of` 문이나 `[...spread]`를 사용할 때는 이런 개념들을 생각하지 않게 된다. 그러나 Typescript는 이것들을 `Iterable` 과 `Iterator` (혹은 이 두가지 모두처럼 동작하는 `IterableIterator`) 타입을 사용해 모델링하며, 이 타입들은 `for / of`와 같은 구문이 작동하기 위해 필요한 최소한의 개념들이다.

`Iterable`은 Javascript의에서 다양한 곳에 사용할 수 있어 편리하지만, 많은 사람들이 배열에서 map, filter, 그리고 특정 이유로 reduce 같은 메소드가 없어서 불편함을 느낀다. 그렇기 때문에 최근 [ECMAScript에서 대부분의 `IterableIterator`에 배열의 여러 메소드를 추가하자는 제안](https://github.com/tc39/proposal-iterator-helpers)이 나왔다. 

예를 들어, 이제부터 모든 생성자는 `map`과 `take` 메소드를 갖는 객체를 만들어낸다.
```javascript
function* positiveIntegers() {
    let i = 1;
    while (true) {
        yield i;
        i++;
    }
}

const evenNumbers = positiveIntegers().map(x => x * 2);

// Output:
//    2
//    4
//    6
//    8
//   10
for (const value of evenNumbers.take(5)) {
    console.log(value);
}
```

`keys()`, `values()`, `entries()`, `Map`, `Set` 에 대해서도 마찬가지다.
```javascript
function invertKeysAndValues<K, V>(map: Map<K, V>): Map<V, K> {
    return new Map(
        map.entries().map(([k, v]) => [v, k])
    );
}
```

새로운 `Iterator` 객체를 확장할 수도 있다.
```javascript
/**
 * Provides an endless stream of `0`s.
 */
class Zeroes extends Iterator<number> {
    next() {
        return { value: 0, done: false } as const;
    }
}

const zeroes = new Zeroes();

// Transform into an endless stream of `1`s.
const ones = zeroes.map(x => x + 1);
```

또한 이미 존재하는 `Iterable`과 `Iterator`를 `Iterator.from`을 사용해 새로운 타입으로 변환할 수 있다.
```javascript
Iterator.from(...).filter(someFunction);
```

이 새로운 메소드들은 최신 버전의 Javascript runtime 혹은, 새로운 `Iteratore` 객체를 위한 polyfill을 사용할 때 동작할 것이다.

이제 네이밍에 관련해서 이야기할 차례이다.

위에서 언급했듯이 Typescript는 `Iterable`과 `Iterator` 타입을 갖는다. 그러나 앞서 언급했듯이 이런 기능들은 특정 작업이 제대로 작동하도록 보장하는 일종의 "프로토콜" 역할을 한다. 이는 Typescript에서 `Iterable`   또는 `Iterator`로 선언된 모든 값들이 위에서 언급한 메소드를 갖는 것은 아니다.

그러나 `Iterator`라는 새로운 런타임 값이 있다. JavaScript에서 `Iterator`와 `Iterator.prototype`을 실제 값으로 참조할 수 있다. 그러나 Typescript는 타입 검사용으로 `Iterator`라는 것을 이미 정의하고 있기 때문에 둘 사이에 이름 충돌이 발생한다. 이러한 문제로 인해 Typescript는 이런 내장된 iterator(반복자)를 설명하기 위해 별도의 타입을 도입해야 한다.

Typescript 5.6에서는  `IteratorObject`라는 새로운 타입을 도입했으며, 이 타입은 다음과 같이 정의된다

```typescript
interface IteratorObject<T, TReturn = unknown, TNext = unknown> extends Iterator<T, TReturn, TNext> {
    [Symbol.iterator](): IteratorObject<T, TReturn, TNext>;
}
```

`IteratorObject`의 하위 타입(`ArrayIterator`, `SetIterator`, `MapIterator` 등)을 만들어 내는 빌트인 컬렉션과 메소드, `lib.d.ts` 내의 코어 Javascript와 DOM 타입, 그리고 `@types/node`가 이 새로운 타입을 위해 업데이트되었다.

이와 유사하게, `AsyncIteratorObject` 타입이 추가되어 비슷한 역할을 한다. Javascript에는 아직 `AsyncIterable`에 메서드를 제공하는 런타임 값인 `AsyncIterator`가 존재하지 않지만, [현재 제안 단계](https://github.com/tc39/proposal-async-iterator-helpers)에 있으며 이 새로운 타입은 이를 위해 준비된 것이다.

- [해당 타입 PR 링크](https://github.com/microsoft/TypeScript/pull/58222)
- [proposal-iterator-helpers](https://github.com/tc39/proposal-iterator-helpers)