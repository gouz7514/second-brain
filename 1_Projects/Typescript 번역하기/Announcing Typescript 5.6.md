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

## Strict Builtin Iterator Checks (and `strictBuiltinIteratorReturn`)

`Iterator<T, TReturn>`의 `next()` 메소드를 호출하면, `value`와 `done` 속성을 가진 객체를 리턴한다. 이 객체는 `IteratorResult` 타입으로 모델링되어있다.
```typescript
type IteratorResult<T, TReturn = any> = IteratorYieldResult<T> | IteratorReturnResult<TReturn>;

interface IteratorYieldResult<TYield> {
    done?: false;
    value: TYield;
}

interface IteratorReturnResult<TReturn> {
    done: true;
    value: TReturn;
}
```

위 코드에서 네이밍의 경우, generator 함수의 동작 방식에서 영감을 받았다. generator 함수는 값을 yield(생성 또는 결과값을 출력)할 수 있고 최종 값을 반환할 수 있다. 그러나 yield된 값과 최종 반환 값의 타입은 서로 관련이 없을 수 있다.

```typescript
function abc123() {
    yield "a";
    yield "b";
    yield "c";
    return 123;
}

const iter = abc123();

iter.next(); // { value: "a", done: false }
iter.next(); // { value: "b", done: false }
iter.next(); // { value: "c", done: false }
iter.next(); // { value: 123, done: true }
```

새로운 `IteratorObject` 타입을 도입하면서, 안전하게 `IteratorObject`를 구현하는데 어려움이 있음을 발견했다. 동시에 `TReturn`이 any 타입인 경우, `IteratorResult` 에서 안전성 문제가 오랫동안 존재해왔다. 예를 들어 `IteratorResult<string, any>`가 있다고 가정하고 value를 사용하게 되면 `string | any` 타입이 되는데, 이는 결국 `any` 타입으로 취급된다.

```typescript
function* uppercase(iter: Iterator<string, any>) {
    while (true) {
        const { value, done } = iter.next();
        yield value.toUppercase(); // oops! forgot to check for `done` first and misspelled `toUpperCase`

        if (done) {
            return;
        }
    }
}
```

모든 반복자에 대한 문제를 수정하는 것은 수많은 변화를 적용해야 하기 때문에 어렵지만, 새로 생성되는 대부분의 `IteratorObject`에서는 이를 개선할 수 있다.

Typescript 5.6에서는 `BuiltinIteratorReturn`이라는 새로운 본질적인 타입과 `--strictBuiltinIteratorReturn`이라는 새로운 `--strict`-mode 플래그를 도입했다. `IteratorObject`가 `lib.d.ts`와 같은 곳에서 사용될 때는 항상 `TReturn`에 대해 `BuiltinIteratorReturn` 타입으로 작성된다 (하지만 더 구체적인 MapIterator, ArrayIterator, SetIterator가 더 자주 사용된다)

```typescript
interface MapIterator<T> extends IteratorObject<T, BuiltinIteratorReturn, unknown> {
    [Symbol.iterator](): MapIterator<T>;
}

// ...

interface Map<K, V> {
    // ...

    /**
     * Returns an iterable of key, value pairs for every entry in the map.
     */
    entries(): MapIterator<[K, V]>;

    /**
     * Returns an iterable of keys in the map
     */
    keys(): MapIterator<K>;

    /**
     * Returns an iterable of values in the map
     */
    values(): MapIterator<V>;
}
```

기본적으로 `BuiltinIteratorReturn`은 `any` 타입이지만, `--stirctBuiltinIteratorReturn` 모드라면 `undefined` 타입이다. 이 모드에서는 `BuiltinIteratorReturn` 을 사용하면 위 예시는 올바르게 에러를 발생시킨다.

```typescript
function* uppercase(iter: Iterator<string, BuiltinIteratorReturn>) {
    while (true) {
        const { value, done } = iter.next();
        yield value.toUppercase();
        //    ~~~~~ ~~~~~~~~~~~
        // error! ┃      ┃
        //        ┃      ┗━ Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
        //        ┃
        //        ┗━ 'value' is possibly 'undefined'.

        if (done) {
            return;
        }
    }
}
```

`BuiltinIteratorReturn`이 통상적으로 `lib.d.ts` 내에서 `IteratorObject`와 같이 등장하는 것을 보게 될 것이다. 가능한 경우 `TReturn`에 대해 더 명확하게 자성하는 것을 권장하낟.

더 자세한 정보는 [여기](https://github.com/microsoft/TypeScript/pull/58243)서 확인할 수 있다.

## Support for Arbitrary Module Identifiers
Javascript는 모듈이 문자열 리터럴로 유효하지 않은 식별자 이름을 바인딩하여 export하는 것을 허용한다.
```javascript
const banana = "🍌";

export { banana as "🍌" };
```

이와 유사하게, Javascript는 임의의 이름을 가진 import를 가져와 유효한 식별자에 바인딩하는 것을 허용한다.
```javascript
import { "🍌" as banana } from "./foo"

/**
 * om nom nom
 */
function eat(food: string) {
    console.log("Eating", food);
};

eat(banana);
```

다른 언어들은 유효한 식별자를 정의하는 규칙이 다를 수 있기 때문에 다른 언어들과의 상호 운용성에서 유용하다. 또한 [esbuild의 inject 기능](https://esbuild.github.io/api/#inject)처럼 코드를 생성하는 도구들에게도 유용할 수 있다.

Typescript 5.6부터는 이런 임의의 모듈 식별자를 사용할 수 있다. 변경 사항은 [여기](https://github.com/microsoft/TypeScript/pull/58640)에서 확인 가능하다.

## The --noUncheckedSideEffectImports Option
Javascript에서는 아무런 값을 가져오지 않음에도 해당 모듈을 `import`하는 것이 가능하다.
```javascript
import "some-module";
```

이러한 import는 side effect를 실행해야만 유용한 동작을 제공할 수 있기 때문에 side effect import라고 불린다. (전역 변수 등록, 프로토타입에 polyfill 추가)

Typescript에서는 이 문법에 대해 꽤 이상한 점이 있다.  
만약 `import`가 유효한 소스 파일로 해석될 수 있다면, Typescript는 해당 파일을 로드하고 체크했다. 반면, 소스 파일을 찾을 수 없으면 Typescript는 아무 경고 없이 해당 import를 무시했다.

이는 놀라운 동작이지만 부분적으로는 Javascript 생태계에서 패턴을 모델링한 결과이다. 예를 들어, 번들러에서 CSS나 다른 에셋을 로드하기 위해 특별한 로더들과 함께 사용되기도 했다. 다음과 같이 번들러가 특정 `.css` 파일을 포함하도록 설정될 수 있다.

```typescript
import "./button-component.css";

export function Button() {
    // ...
}
```

그럼에도, 이는 side effect import에서 발생할 수 있는 오타를 감출 수 있다. 그래서 Typescript 5.6에서는 이러한 경우를 잡아내기 위해 `--noUncheckedSideEffectImports`라는 새로운 컴파일러 옵션을 도입했다. 이 옵션이 활성화되면, side effect import에 대해 소스 파일을 찾을 수 없을 때 오류를 발생시킨다.

```typescript
import "oops-this-module-does-not-exist";
//     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// error: Cannot find module 'oops-this-module-does-not-exist' or its corresponding type declarations.
```

이 옵션을 활성화하면, 위 코드처럼 동작하던 코드가 에러를 발생시킬 수 있다. 이를 해결하기 위해, 단순히 에셋에 대해  side effect import를 작성하려는 사용자들은 와일드카드 지정자가 포함된 _ambient module declaration_ 을  작성하는 것이 더 나은 방법일 수 있다. 전역 파일에 다음과 같이 작성할 수 있다.

```typescript
// ./src/globals.d.ts

// Recognize all CSS files as module imports.
declare module "*.css" {}
```

이미 프로젝트 내에 이런 파일이 존재할 수 있다. `vite init` 명령어를 실행하면 이와 비슷한 `vite-env.d.ts` 파일이 생성된다.

해당 옵션은 기본적으로 off이지만 한번 시도해볼 것을 권장한다.

더 자세한 정보는 [여기](https://github.com/microsoft/TypeScript/pull/58941)서 확인할 수 있다.

## The --noCheck Option
Typescript 5.6에서는 모든 input 파일에 대해 타입 체크를 생략하는 `--noCheck`라는 새로운 옵션을 도입했다. 이는 출력 파일을 생성하기 위해 필요한 의미 분석을 수행할 때 불필요한 타입 체크를 피하는데 유용하다.

이 옵션의 사용 시나리오 중 하나는, Javascript 파일 생성과 타입 체크를 분리하여 두 작업을 각각의 단계로 실행하는 것이다. 예를 들어, 반복 작업 중에는 `tsc --noCheck`를 실행하고, 철저한 타입 체크를 위해서는 `tsc --noEmit`을 실행할 수 있다. 또한 두 작업을 병렬로 실행할 수 있으며, 심지어 `--watch`모드에서도 가능하다. 다만, 두 작업을 동시에 실행할 경우에는 `tsBuildInfoFile`경로를 따로 지정하는 것이 좋다.

`--noCheck`는 선언 파일을 생성하는 경우에도 유용하다. `–isolatedDeclarations`를 준수하는 프로젝트에서 `--noCheck`가 지정된 경우, TypeScript는 타입 체크 과정 없이도 빠르게 선언 파일을 생성할 수 있다. 이렇게 생성된 선언 파일은 빠른 구문적 변환에만 의존하게 된다.

`–noCheck`가 지정되었지만 프로젝트에서 `–isolatedDeclarations`를 사용하지 않는 경우, TypeScript는 여전히 `.d.ts` 파일을 생성하기 위해 필요한 만큼의 타입 체크를 수행할 수 있다. 이 점에서 `–noCheck`는 약간 오해의 소지가 있는 이름일 수 있지만, 이 과정은 전체 타입 체크보다는 느슨하게 진행되며, 주로 타입 주석이 없는 선언의 타입만 계산한다. 이는 전체 타입 체크보다 훨씬 빠르게 처리될 수 있다.

`--noCheck`는 Typescript API를 통해서도 표준 옵션으로 사용할 수 있다. 내부적으로는 `transpileModule`과 `transpileDeclaration`을 사용해 속도를 높였다. 이제 어떤 빌드 도구라도 이 플래그를 활용해 다양한 맞춤형 전략을 통해 빌드 속도를 개선할 수 있다.

더 자세한 정보는 [TypeScript 5.5에서 noCheck를 내부적으로 강화하기 위해 수행된 작업](https://github.com/microsoft/TypeScript/pull/58364)과 이를 [명령어에서 공개적으로 사용할 수 있도록 한 관련 작업](https://github.com/microsoft/TypeScript/pull/58839)을 참조하면 된다.

## Allow --build with Intermediate Errors
Typescript의 프로젝트 참조(project references) 개념은 코드베이스를 여러 프로젝트로 구성하고 그들 사이에 의존성을 만들 수 있게 해준다. `--build` 모드에서 Typescript 컴파일러를 실행하는 것은, 여러 프로젝트에 걸친 빌드를 실제로 수행하고, 어떤 프로젝트와 파일을 컴파일해야 하는지 파악하는 내장 방식이다.

이전에는 `--build` 모드는 `--noEmitOnError`로 간주되어 에러를 맞닥뜨리는 순간 build가 중단되었다. 이는 즉 "상위" 종속성이 빌드 과정에서 에러가 발생된다면 해당 종속성을 사용하는 "하위" 프로젝트는 절대로 체크되거나 빌드될 수 없음을 의미했다. 이론적으로는 타당한 방식인데, 만약 프로젝트에 에러가 있다면 해당 프로젝트는 종속성에 대해 일관된 상태가 아닐 수 있기 때문이다.

실제로, 이러한 엄격함이 업그레이드 작업을 더욱 어렵게 만들었다. 만약 프로젝트 B가 프로젝트 A에 의존한다면, 프로젝트 B에 더 익숙한 사람은 의존성이 업그레이드되기 전까지는 프로젝트 B를 업그레이드할 수 없게 된다. 프로젝트 A가 먼저 업그레이드되어야 하므로 프로젝트 B의 작업이 막히는 것이다.

Typescript 5.6부터는 빌드 과정에서 종속성에 중간 오류가 있어도 프로젝트 빌드가 계속 진행된다. 중간 오류가 발생하면 해당 오류는 꾸준히 보고되고, 가능한 최선의 방식으로 출력 파일이 생성된다. 그러나 지정된 프로젝트의 빌드는 끝까지 완료된다.

오류가 있는 프로젝트에서 빌드를 중단하고 싶다면, `--stopOnBuildErrors`라는 새로운 플래그를 사용할 수 있다. 이 플래그는 CI 환경에서 실행할 때나, 다른 프로젝트들이 많이 의존하는 프로젝트를 반복적으로 작업할 때 유용하다. 

이를 달성하기 위해, Typescript는 이제 `--build`로 호출된 모든 프로젝트에 대해 `tsbuildinfo`파일을 항상 (`--incremental` 또는 `--composite`이 선언되지 않더라도) 생성한다. 이는 `--build`가 어떻게 호출되었는지와 앞으로 수행해야 할 작업의 상태를 추적하기 위함이다.

더 자세한 정보는 [여기](https://github.com/microsoft/TypeScript/pull/58838)서 확인할 수 있다.

## Region-Prioritized Diagnostics in Editors
Typescript는 파일에 대해 진단을 할 때(에러, 제안, deprecation 과 같은), 보통은 전체 파일에 대해 검사를 진행할 것이다. 대부분의 경우 이는 괜찮지만, 엄청나게 거대한 파일의 경우 지연이 발생할 수 있다. 단순 오타 수정은 단순한 작업이지만, 거대한 파일에서는 몇 초가 소요될 수 있기 때문에 이는 매우 두려운 상황일 수 있다.

이를 해결하기 위해 Typescript 5.6에서는 _region-priorizited diagnositics_ 또는 _region-prioritized checking_ 이라 불리는 기능을 도입했다. 파일들에 대해 단순히 검사를 요청하는 대신, 에디터는 특정 파일의 관련 영역을 제공할 수 있으며, 이 영역은 보통 사용자가 현재 보고 있는 파일의 부분을 의미한다. Typescript 언어 서버는 파일의 특정 영역과 전체 파일에 대해 각각 두 가지 진단 결과를 제공할 수 있다. 이를 통해 큰 파일을 편집할 때, 빨간 밑줄이 사라질 때까지 기다리는 시간이 줄어들어 편집 속도가 훨씬 더 빠르게 느껴지게 된다.

Typescript의 checker.ts 파일에서 테스트한 결과, 전체 파일에 대한 진단은 3330ms가 걸렸다. 반면, region-based 진단의 경우 143ms에 불과했다. 남은 전체 파일에 대한 응답이 3200ms가 걸렸지만, 빠른 편집 작업에서는 이러한 차이가 큰 영향을 미칠 수 있다.

이 기능은 진단 결과를 더 일관되게 보고하기 위한 작업도 포함하고 있다. type-checker가 캐싱을 활용해 중복된 작업을 피하는 방식 때문에, 동일한 타입 간의 후속 검사가 종종 다른 오류 메시지를 생성할 수 있었다. 더 자세하게 말하면, 지연된 비순차적 검사는 에디터의 두 위치에서 서로 다른 진단 결과를 보고할 수 있었다. 이 문제는 이번 기능 도입 이전에도 존재했지만, 이 문제를 더 악화시키고 싶지 않았다. 이번 작업을 통해 이러한 오류 불일치를 많이 해결했다.

현재 이 기능은 Typescript 5.6 이후 버전과 VSC 환경에서 활용 가능하다.

더 자세한 정보는 [여기](https://github.com/microsoft/TypeScript/pull/57842)서 확인할 수 있다.

## Granular Commit Characters
Typescript는 이제 각 자동 완성 항목에 대해 자체적인 _커밋 문자_ 를 제공한다. 커밋 문자는 사용자가 입력할 때 현재 제안된 자동 완성 항목을 자동으로 확정하는 특정 문자이다.

이는 에디터에서 특정 문자를 입력할 때 에디터가 현재 제안된 자동 완성 항목을 더 자주 commit하게 된다는 의미이다. 다음 예시를 보자.

```typescript
declare let food: {
    eat(): any;
}

let f = (foo/**/
```

만약 커서가 `/**/`에 있다면, 작성하고자 하는 코드가 `let f = (food.eat())`가 될지 `let f = (foo, bar) => foo + bar`가 될지 불확실하다. 우리는 우리가 다음으로 작성하는 문자에 따라 에디터가 다르게 자동 완성을 해주리라 기대하게 될 것이다. 예를 들어, 만약 우리가 `.`을 입력하면, 에디터가 `food` 변수를 사용할 가능성이 높게 될 것이다. 그러나 만약 `,`를 입력하면, arrow function 안의 파라미터를 작성하게 될 것이다.

불행히도 이전에는 Typescript가 에디터에게 현재 텍스트가 새로운 파라미터 이름을 정의할 수 있다고 신호를 보냈기 때문에 어떤 커밋 문자도 안전하지 않았다. 그래서 `.`을 입력해도, 에디터가 `food`를 사용해 자동 완성해야 할 명확한 상황임에도 불구하고 아무 동작도 하지 않았다.

이제 Typescript는 각 자동 완성 항목에 대해 안전하게 커밋할 수 있는 문자를 명시적으로 나열한다. 이 기능으로 인해 당장 즉각적인 변화를 느끼지는 않겠지만, 커밋 문자를 지원하는 에디터에서는 시간이 지남에 따라 자동 완성 동작이 개선될 것이다. 개선된 점을 바로 보려면 [VSC Insiders](https://code.visualstudio.com/insiders/)에서  [Typescript nightly extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-next)를 사용해 볼 수 있다. 위 코드에서 `.`을 입력하면 `food`를 자동 완성해 줄 것이다.

더 자세한 정보는 다음에서 확인할 수 있다.
- [커밋 문자 추가에 대한 PR](https://github.com/microsoft/TypeScript/pull/59339)
- [context에 의존하는 커밋 문자에 대한 수정 작업](https://github.com/microsoft/TypeScript/pull/59523)

## Exclude Patterns for Auto-Imports
Typescript는 이제 특정 지정자(specifier)에서 auto-import 제안을 필터링하기 위해 정규 표현식 패턴 목록을 지정할 수 있습니다. 예를 들어, `lodash`와 같은 패키지에서 모든 "깊은" import를 제외하려면 VSC에서 다음과 같은 설정을 구성할 수 있습니다.

```json
{
    "typescript.preferences.autoImportSpecifierExcludeRegexes": [
        "^lodash/.*$"
    ]
}
```

반대로, 패키지의 진입점에서 import하는 것을 금지하고 싶을 수도 있습니다.
```json
{
    "typescript.preferences.autoImportSpecifierExcludeRegexes": [
        "^lodash$"
    ]
}
```

다음과 같이 설정하면 `node:`에 대해서도 import를 방지할 수 있다.
```json
{
    "typescript.preferences.autoImportSpecifierExcludeRegexes": [
        "^node:"
    ]
}
```

`i`나 `u`와 같은 특정 정규식 flag를 지정하고 싶으면, 정규식을 슬래시로 감싸야 한다. 이때, 내부에 있는 슬래시는 escape 처리를 해야 한다.
```json
{
    "typescript.preferences.autoImportSpecifierExcludeRegexes": [
        "^./lib/internal",        // no escaping needed
        "/^.\\/lib\\/internal/",  // escaping needed - note the leading and trailing slashes
        "/^.\\/lib\\/internal/i"  // escaping needed - we needed slashes to provide the 'i' regex flag
    ]
}
```

JavaScript에 대해서도 VSC에서 `javascript.preferences.autoImportSpecifierExcludeRegexes`
 설정을 통해 적용할 수 있다.

`typescript.preferences.autoImportFileExcludePatterns`와 겹치는 것처럼 보이지만, 차이가 있다. 기존의 `autoImportFileExcludePatterns`는 파일 경로를 제외하기 위해 글롭(glob) 패턴 목록을 사용한다. 특정 파일 또는 디렉토리에 대해 auto-import를 피하고 싶은 대다수의 상황에 있어서는 간단할 수 있지만, 항상 그렇지는 않을 것이다. 예를 들어, `@types/node` 패키지를 사용하는 경우, 동일한 파일이 `fs`와 `node:fs`를 모두 선언하기 때문에 `autoImportExcludePatterns`를 사용해 둘 중 하나를 필터링할 수 없다.

새로운 `autoImportSpecifierExcludeRegexes` 옵션은 모듈 지정자에 특화되어 있어, `fs` 또는 `node:fs` 둘 중 하나를 제외하는 표현식을 작성할 수 있다.더 나아가, 패턴을 사용해 auto-import가 다른 지정자 스타일을 선호하도록 설정할 수 있다. (`./foo/bar.js`를 `#foo/bar.js`보다 선호하게 할 수 있다.)

더 자세한 정보는 [여기](https://github.com/microsoft/TypeScript/pull/59543)서 확인할 수 있다.

## Notable Behavioral Changes
이 섹션에서는 알아두면 좋을 변경 사항들에 대해 다루고 있으며 deprecation, 기능 제거 또는 새로운 제약일 수도 있다. 버그 수정을 위한 기술적 개선도 포함되며, 이는 기존 빌드에 새로운 에러를 야기할 수도 있다.

### lib.d.ts
DOM을 위해 생성된 type은 코드베이스의 type을 검사하는데 영향을 끼칠 수 있다. 더 자세한 정보는, [DOM과 lib.d.ts 관련 이슈](https://github.com/microsoft/TypeScript/issues/58764)들을 살펴보자

### .tsbuildinfo is Always Written
`--build` 모드에서 중간 의존성에 오류가 있어도 프로젝트 빌드를 계속 진행할 수 있도록 하기 위해, 그리고 명령줄에서 `--noCheck`를 지원하기 위해 Typescript는 이제 --build 호출 시 모든 프로젝트에 대해 항상 `.tsbuildinfo` 파일을 생성한다. 이는 `--incremental` 옵션의 활성화 여부과 관계없이 발생한다. 더 자세한 정보는 [여기](https://github.com/microsoft/TypeScript/pull/58626)서 확인할 수 있다.

### Respecting File Extensions and package.json from within node_modules
Node.js가 12버전에서 ECMAScript module을 지원하기 이전에는, Typescript가 `node_modules`에서 찾은 `.d.ts` 파일이 CommonJS로 작성된 Javascript 파일인지, ECMAScript 모듈인지 알 수 있는 좋은 방법이 없었다. 대부분의 npm 패키지가 CommonJS만 사용하던 시기에는 큰 문제가 없었으며 의심스러운 경우, Typescript는 모든 것이 CommonJS처럼 동작한다고 가정할 수 있었다. 그러나 만약 이 가정이 틀리다면 이는 안전하지 않은 import를 허용할 수 있다.

```typescript
// node_modules/dep/index.d.ts
export declare function doSomething(): void;

// index.ts
// Okay if "dep" is a CommonJS module, but fails if
// it's an ECMAScript module - even in bundlers!
import dep from "dep";
dep.doSomething();
```

실제로 많이 발생하는 경우는 아니다. 그러나 Node.js가 ECMAScript 모듈을 지원하기 시작한 이후로, ESM (ECMAScript Module)의 공유수도 점점 늘어갔다. 다행히, Node.js는 Typescrip가 주어진 파일이 ECMAScript module인지 혹은 CommonJS 모듈인지 결정하도록 하는 메커니즘을 도입했다. `.mjs`와 `.cjs` 파일 확장자 그리고 `package.json`의 `"type"`필드가 바로 그것이며, Typescript 4.7에서`.mts`, `.cts` 파일들을 통해 이들을 지원했다. 그러나 Typescript는 오직 `--module node16`과 `--module nodenext`에서만 이들을 읽을 수 있었기에, `--module esnext`, `--moduleResolution bundler`를 사용하는 사람에게는 불안전한 import가 여전히 문제로 남게 되었다.

이를 해결하기 위해 Typescript 5.6은 module forat 정보를 수집하고 위와 같은 모든 `module` 모드에서 발생하는 모호함을 해결하기 위해 사용한다. 형식별 파일 확장자(`.mts` 또는 `.cts`)는 어디에서든 인식되며, `module` 설정에 관계없이 `node_modules` 의 의존성 내에서는 `package.json`의 `"type"`필드가 참조된다. 이전에는, CommonJS 결과를 `.mjs` 로 바꾸거나 혹은 그 반대의 경우만 기술적으로 가능했다.

```typescript
// main.mts
export default "oops";

// $ tsc --module commonjs main.mts
// main.mjs
Object.defineProperty(exports, "__esModule", { value: true });
exports.default = "oops";
```

이제는 `.mts` 파일이 CommonJS 결과물을 만들지 않고, `.cts` 파일이 ESM 결과물을 만들어내지 않는다.

이 동작들은 Typescript 5.5 버전의 초기 릴리즈 버전에서 이미 제공되었으나, 5.6부터 이 동작은 `node_modules` 내의 파일에만 확장된다.

더 자세한 정보는 [여기](https://github.com/microsoft/TypeScript/pull/58825)서 확인할 수 있다.

### Correct Override Checks on Computed Properties
이전에는 `override`로 선언된 속성이 base class member (override 대상이 된 클래스의 멤버)의 존재 여부를 제대로 검사하지 않았다. 이와 유사하게, 만약 `noImplicitOverride`를 사용하면, `override` 를 선언하는 것을 잊더라도 아무 에러가 발생하지 않을 것이다.

Typescript 5.6에서는 이 두 케이스 모두 속성을 제대로 검사한다.
```typescript
const foo = Symbol("foo");
const bar = Symbol("bar");

class Base {
    [bar]() {}
}

class Derived extends Base {
    override [foo]() {}
//           ~~~~~
// error: This member cannot have an 'override' modifier because it is not declared in the base class 'Base'.

    [bar]() {}
//  ~~~~~
// error under noImplicitOverride: This member must have an 'override' modifier because it overrides a member in the base class 'Base'.
}
```
