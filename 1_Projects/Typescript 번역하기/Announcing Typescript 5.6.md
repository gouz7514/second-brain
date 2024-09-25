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