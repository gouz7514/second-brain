- 공식 블로그 작성일 : 2024.06.20
- [원문 링크](https://devblogs.microsoft.com/typescript/announcing-typescript-5-5/)

---
## Inferred Type Predicates

타입스크립트는 변수의 타입이 코드 내에서 어떻게 변하는지 추적하는 과정을 거친다.
```tsx
interface Bird {
    commonName: string;
    scientificName: string;
    sing(): void;
}

// Maps country names -> national bird.
// Not all nations have official birds (looking at you, Canada!)
declare const nationalBirds: Map<string, Bird>;

function makeNationalBirdCall(country: string) {
  const bird = nationalBirds.get(country);  // bird has a declared type of Bird | undefined
  if (bird) {
    bird.sing();  // bird has type Bird inside the if statement
  } else {
    // bird has type undefined here.
  }
}
```

위와 같이 `undefined` 케이스를 핸들링하면, 코드가 더 복잡하고 지저분해진다.

과거에는, 배열에 이런 식으로 타입을 정의하는 것이 훨씬 까다로웠다.  
아래 코드는 이전 버전에서 에러를 발생시킨다.
```tsx
function makeBirdCalls(countries: string[]) {
  // birds: (Bird | undefined)[]
  const birds = countries
    .map(country => nationalBirds.get(country))
    .filter(bird => bird !== undefined);

  for (const bird of birds) {
    bird.sing();  // error: 'bird' is possibly 'undefined'.
  }
}
```

위 코드는 논리적으로는 완벽하지만, 타입스크립트에서는 에러가 발생한다.  
5.5버전에서는 정상적으로 작동한다
```tsx
function makeBirdCalls(countries: string[]) {
  // birds: Bird[]
  const birds = countries
    .map(country => nationalBirds.get(country))
    .filter(bird => bird !== undefined);

  for (const bird of birds) {
    bird.sing();  // ok!
  }
}
```
`birds` 변수의 타입이 더 정확해졌다.

이는 타입스크립트가 `filter` 함수에 대해 [type predicate](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)를 추론하기 때문이다.  
아래처럼 함수를 떼어내면 더 명확히 알 수 있다.
```tsx
// function isBirdReal(bird: Bird | undefined): bird is Bird
function isBirdReal(bird: Bird | undefined) {
  return bird !== undefined;
}
```

여기서 `bird is Bird` 가 type predicate이다.  
이는 만약 위 함수가 `true`를 리턴한다면, `bird` 변수는 `Bird` 타입이라는 것을 의미한다.  
(즉, `value is type`)

`Array.prototype.filter` 의 타입 명세는 type predicate를 진행해, 수행 결과가 더 명확한 타입을 갖고 타입 체크 과정을 통과하게 된다.

아래와 같은 조건을 만족한다면 typescript는 함수가 type predicate를 리턴한다고 추론할 것이다.

1. 함수가 명확한 리턴 타입 혹은 type predicate 표현식을 가지지 않는 경우
2. 함수가 하나의 `return` 문을 가지고, implicit return문이 없는 경우
3. 함수가 파라미터를 변형하지 않는 경우
4. 함수가 파라미터를 정제하는 `boolean` 표현식을 리턴하는 경우

type predicate의 추가 예시를 들자면 다음과 같다.
```tsx
// const isNumber: (x: unknown) => x is number
const isNumber = (x: unknown) => typeof x === 'number';

// const isNonNullish: <T>(x: T) => x is NonNullable<T>
const isNonNullish = <T,>(x: T) => x != null;
```

이전에는, 타입스크립트는 위 함수들이 단지 `boolean` 타입을 리턴한다고만 추론했을 것이다.  
이제는 `x는 number 타입` 또는 `x는 NonNulable<T>` 와 같이 type predicate 를 추론하게 된다.

type predicate는 “if and only if” 구문을 갖는다. 만약 함수가 `x is T` 를 리턴한다면 이는 다음을 의미한다.
1. 만약 함수가 `true` 를 리턴하면 `x` 는 `T` type 이다.
2. 만약 함수가 `false` 를 리턴하면 `x` 는 `T` type을 갖지 않는다.

type predicate가 추론되기를 기대하지만 그렇지 않다면, 두번째 규칙을 위반하는 것일수도 있다.  
이는 종종 진실성(truthiness) 체크로 인한 결과이다.
```tsx
function getClassroomAverage(students: string[], allScores: Map<string, number>) {
  const studentScores = students
    .map(student => allScores.get(student))
    .filter(score => !!score);

  return studentScores.reduce((a, b) => a + b) / studentScores.length;
  //     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  // error: Object is possibly 'undefined'.
}
```

Typescript는 `score => !!score` 에 대해 type predicate를 추론하지 못한다.  
만약 이 구문이 `true` 를 리턴하면, `score` 는 `number` 타입인 것이다.  
하지만 만약 `false` 를 리턴하면, `score` 는 `undefined` 이거나 `number` (정확히는 `0`)인 것이다.  
이는 함수가 목표로 하는 평균 점수를 구하는 데 문제를 야기하게 된다.  
(0점 학생이 걸러지면 학생 수는 줄어들고 그만큼 평균 점수는 올라가기 때문에!)

따라서, 위 함수에서 `undefined` 를 걸러내는 것이 좋다.
```tsx
function getClassroomAverage(students: string[], allScores: Map<string, number>) {
  const studentScores = students
    .map(student => allScores.get(student))
    .filter(score => score !== undefined);

  return studentScores.reduce((a, b) => a + b) / studentScores.length;  // ok!
}
```

위와 같은 진실성 검사는 모호하지 않은 객체 타입에 대해 type predicate를 추론하게 된다.  
함수는 반드시 `boolean` 을 리턴해야 type predicate의 추론 후보가 될 수 있다. (`x ⇒ !!x` 는 될지 몰라도 `x => x` 는 안 된다.)

명시적 type predicate는 이전과 같이 동작한다.  
Typescript는 동일한 type predicate에 대한 추론 여부는 체크하지 않는다.  
명시적 type predicate(”is”)는 type assertion(”as”) 보다 안전하지 않다.

Typescript가 더 상세하게 type을 추론한다면 돌아가는 코드도 안 돌아가게 될 수 있다.
```tsx
// Previously, nums: (number | null)[]
// Now, nums: number[]
const nums = [1, 2, 3, null, 5].filter(x => x !== null);

nums.push(null);  // ok in TS 5.4, error in TS 5.5
```

위에 대한 해결책은 type에 explicit type annotation을 사용하는 것이다.
```typescript
const nums: (number | null)[] = [1, 2, 3, null, 5].filter(x => x !== null);
nums.push(null);  // ok in all versions
```

---
## Control Flow Narrowing for Constant Indexed Access
> 한국어로 번역하면 "상수 인덱스 접근에 대한 제어 흐름 좁히기.."가 될 것 같다.  
> 즉, 상수를 index로 사용해 접근할 때 타입을 점차 좁혀나가는 것을 의미한다.

`obj`와 `key`가 상수일 때 `obj[key]` 형태의 표현식을 좁힐 수 있다.
```typescript
function f1(obj: Record<string, unknown>, key: string) {
    if (typeof obj[key] === "string") {
        // Now okay, previously was error
        obj[key].toUpperCase();
    }
}
```
위 코드에서 `obj` 와 `key` 는 변경되지 않기 때문에 typescript는 `typeof` 를 통해 체크를 한 뒤 `obj[key]`의 타입을 `string`으로 좁힐 수 있다.

---
## JSDoc `@import` 태그
자바스크립트 파일 내에서 오로지 타입 체크를 위해 무언가를 import 하는 것은 정말 귀찮은 작업이다.  
특정 타입이 필요할지라도 런타임에 없다면 사용할 수도 없다.  
(타입스크립는 정적 타입 언어로 컴파일 시에 타입 검사가 이루어지지만, 자바스크립트는 실행 중에 모든 것이 평가되고 적용되는 동적 언어)
```javascript
// ./some-module.d.ts
export interface SomeType {
    // ...
}

// ./index.js
import { SomeType } from "./some-module"; // ❌ runtime error!

/**
 * @param {SomeType} myValue
 */
function doSomething(myValue) {
    // ...
}
```

위 코드에서 `SomeType`은 런타임에 존재하지 않으므로, import되지 않을 것이다.  
대신 namespace import를 사용할 수 있다.

```javascript
import * as someModule from "./some-module";

/**
 * @param {someModule.SomeType} myValue
 */
function doSomething(myValue) {
    // ...
}
```

그렇지만 여전히 `some-module`이 import되어야 한다.  
대신 JSDoc 주석에 `import(...)`를 사용할 수 있다.
```javascript
/**
 * @param {import("./some-module").SomeType} myValue
 */
function doSomething(myValue) {
    // ...
}
```

만약 이를 여러 곳에서 재사용하고 싶으면, `typedef`를 사용할 수 있다.
```javascript
/**
 * @typedef {import("./some-module").SomeType} SomeType
 */

/**
 * @param {SomeType} myValue
 */
function doSomething(myValue) {
    // ...
}
```
이렇게 코드를 작성하면 `SomeType`을 전역적으로 사용할 수 있지만, 여러 import 문이 있다면 이야기는 달라질 것이다 (더 귀찮아진다..)

이를 해결하기 위해! Typescript는 ECMAScript imports와 동일한 문법을 가진 새로운 `@import` 주석을 도입했다.

```javascript
/** @import { SomeType } from "some-module" */

/**
 * @param {SomeType} myValue
 */
function doSomething(myValue) {
    // ...
}
```

namespace import를 사용하면 최종적으로는 아래와 같이 된다.
```javascript
/** @import * as someModule from "some-module" */

/**
 * @param {someModule.SomeType} myValue
 */
function doSomething(myValue) {
    // ...
}
```

---
## Regular Expression Syntax Checking
지금까지 Typescript는 코드 내에 있는 많은 정규식들을 생략해왔다.  
정규식은 기술적으로 다른 문법을 가지고 있으며, 굳이 Typescript가 이런 정규식들을 자바스크립트의 초기 버전으로 컴파일할 필요가 없었기 때문이다.  
그러나, 이는 정규식 내의 많은 문제점이 발생되지 않을 수 있게 되어 결국은 에러로 이어질 수 있다.

이제는 Typescript가 정규식에 대해 기본적인 문법 체크를 하게 된다.
```typescript
let myRegex = /@robot(\s+(please|immediately)))? do some task/;
//                                            ~
// error!
// Unexpected ')'. Did you mean to escape it with backslash?
```

간단한 예시지만 이를 통해 많은 실수를 잡아낼 수 있다.  
Typescript의 체크 과정은 단순한 구문 검사보다 조금 더 나아가, 예를 들어 존재하지 않는 역참조(backreferences)와 관련된 문제도 잡아낼 수 있다.
```typescript
let myRegex = /@typedef \{import\((.+)\)\.([a-zA-Z_]+)\} \3/u;
//                                                        ~
// error!
// This backreference refers to a group that does not exist.
// There are only 2 capturing groups in this regular expression.
```

```typescript
let myRegex = /@typedef \{import\((?<importPath>.+)\)\.(?<importedEntity>[a-zA-Z_]+)\} \k<namedImport>/;
//                                                                                        ~~~~~~~~~~~
// error!
// There is no capturing group named 'namedImport' in this regular expression.
```

또한, 대상 ECMAScript 버전보다 최신인 정규 표현식이 사용될 때도 인식할 수 있다.
```typescript
let myRegex = /@typedef \{import\((?<importPath>.+)\)\.(?<importedEntity>[a-zA-Z_]+)\} \k<importedEntity>/;
//                                  ~~~~~~~~~~~~         ~~~~~~~~~~~~~~~~
// error!
// Named capturing groups are only available when targeting 'ES2018' or later.
```

정규 표현식 플래그(i, g 와 같은)에 대해서도 마찬가지다.

위와 같은 체크 과정은 정규 표현식 리터럴에 대해서만 한정된다. `new RegExp`로 선언하는 경우 해당 문자열에 대해 체크하지 않는다.

---
## Support for New ECMAScript `Set` Methods
Typescript 5.5는 ECMAScript에 새롭게 등장한 `Set` 타입을 포함한다.

`union`, `intersection`, `difference`, `symmetricDifference`와 같은 메소드들은 `Set`을 인자로 받아 새로운 `Set`을 결과로 만든다.  
`isSubsetOf`, `isSupersetOf`, `isDisjointFrom`와 같은 메소드든 `Set`을 `boolean`값으로 만든다. 이 메소드들은 원본 `Set`을 변화시키지 않는다.

(자세한 내용은 [원문 코드](https://devblogs.microsoft.com/typescript/announcing-typescript-5-5/#support-for-new-ecmascript-set-methods) 참고)

---
## Isolated Declarations
Declaration files (`.d.ts` 파일들)는Typescript에게 특정 라이브러리와 모듈의 형태를 설명해준다.  
이 파일들은 라이브러리의 타입 정의를 포함함과 동시에 함수 바디와 같은 상세한 실행 내용 등은 포함하지 않는다.  
라이브러리를 분석할 필요 없이 Typescript가 라이브러리를 체크할 수 있게 해준다.  
수기로 declaration files들을 작성할 수 있지만, 더 안전하고 간단한 방법은 `--declaration`  을 사용해 Typescript가 자동으로 이 파일들을 생성하게 하는 것이다.

Typescript 컴파일러와 API는 declaration files 들을 생성하는 역할을 맡아왔으나, 다른 도구를 사용하고 싶거나 전통적인 빌드 프로세스가 확장되지 않는 일부 사용 사례가 있을 수 있다.

[상세한 내용은 원문에서..](https://devblogs.microsoft.com/typescript/announcing-typescript-5-5/#isolated-declarations)

---
## The `${configDir}` Template variable for Configuration Files
베이스가 되는 `tsconfig.json` 파일을 생성하고 이를 여러 코드베이스에서 재사용하는 경우는 매우 흔하다. `extends` 키워드를 사용해 이를 가능하게 한다

```json
{
    "extends": "../../tsconfig.base.json",
    "compilerOptions": {
        "outDir": "./dist"
    }
}
```

위 방법이 가지는 이슈중 하나는, `tsconfig.json`파일 내의 모든 경로들이 파일 자체에 대해 상대적이라는 것이다. 만약 여러 프로젝트에서  공유되는 공통의 `tsconfig.base.json` 파일이 있다면, 상대 경로는 원하는 것처럼 동작하지 않을 수 있다.
```json
{
    "compilerOptions": {
        "typeRoots": [
            "./node_modules/@types"
            "./custom-types"
        ],
        "outDir": "dist"
    }
}
```
만약 작성자의 의도가 위 파일을 extend 하는 `tsconfig.json` 파일들은
1. 원하는 `tsconfig.json` 파일에 대해 상대적으로 `dist`  디렉토리를 output으로 가지며,
2. 원하는 `tsconfig.json`파일에 대해 상대적으로 `custom-types` 디렉토리를
를 만족해야 한다면, 이는 원하는대로 되지 않을 것이다.  
`typeRoots`경로는 extend해서 사용하는 프로젝트가 아닌,  공유되는 `tsconfig.base.json`파일의 경로에 대해 상대적이기 때문이다.  
공통의 파일을 extend하는 모든 프로젝트는 같은 내용의 `outDir`,`typeRoots`를 선언해줘야 한다.  
이는 여러 프로젝트간 동기화를 어렵게하며, 이는 `typeRoots`외에 `path`와 같은 다른 옵션에서도 발생하는 문제이다.

이를 해결하기 위해 Typescript 5.5에서는 새로운 템플릿 변수 `${configDir}`이 등장하였다.  
`${configDir}`를 특정 경로 필드에 사용하면, 해당 변수는 컴파일 시에 설정 파일이 포함된 디렉토리르 대체된다. 즉, 다음과 같이 작성할 수 있다.
```json
{
    "compilerOptions": {
        "typeRoots": [
            "${configDir}/node_modules/@types"
            "${configDir}/custom-types"
        ],
        "outDir": "${configDir}/dist"
    }
}
```

위 파일을 extend해서 사용하면, 경로들이 원하는 `tsconfig.json`에 대해 상대적이 될 것이다.  이를 통해 configuration file을 공유하고 여러 프로젝트에서 더 용이하게 관리할 수 있게 된다.

---
## Consulting `package.json` Dependencies for Declaration Files Generation
Typescript에서는 아래와 같은 에러가 종종 발생한다.
```
The inferred type of "X" cannot be named without a reference to "Y". This is likely not portable. A type annotation is necessary.
```

이는 종종 Typescript의 declaration file 생성이 명시적으로 import되지 않은 파일의 내용에서 자신을 발견하는 경우에 발생한다.  
이런 파일에 대해 import를 하는 것은 경로가 상대 경로인 경우 위험할 수 있다. 그러나 `package.json`의 dependencies(또는 `peerDependencies`, `optionalDependencies`)에 명시된 dependencies를 사용하는 경우는 이러한 import 문을 사용하는 것이 안전할 수 있다.

Typescript 5.5에서는 이러한 경우에 더 관대해져서, 오류가 발생하지 않게 된다.

- [관련 이슈](https://github.com/microsoft/TypeScript/issues/42873), [관련 PR](https://github.com/microsoft/TypeScript/pull/58176)

---
## Editor and Watch-Mode Reliability Improvements
Typescript는 `--watch`  모드와 코드 에디터 integration이 더 신뢰성 있도록 몇가지 새로운 기능을 추가하고 기존 로직을 수정했다. 이는 TSServer 혹은 editor의 재시작 현상을 덜 발생시킬 것이다.

### Correctly Refresh Editor Errors in Configuration Files
Typescript는 `tsconfig.json` 파일에 대해 에러를 발생시킬 수 있다.  
그러나 이 에러는 프로젝트의 로딩 과정에서 발생하며, editor는 일반적으로 `tsconfig.json` 파일에 대한 에러를 직접 발생시키지 않는다. 이는 즉 `tsconfig.json` 파일에서 발생한 에러가 모두 수정이 되어도, Typescript가 비어있는 error 상태를 갱신하지 않기 때문에, 유저가 editor를 새로고침하지 않는 이상 error를 계속 보게 된다.

Typescript 5.5는 이 현상을 해결하기 위해 event를 발생시킨다. [PR 참고](https://github.com/microsoft/TypeScript/pull/58120)

### Better Handling for Deletes Followed by Immediate Writes
어떤 도구들은 파일을 덮어쓰는 대신, 해당 파일을 지우고 새로 만드는 방식을 채택한다. `npm ci` 명령어가 바로 이런 방식으로 동작한다.

이 방식은 이러한 도구들에는 효율적일 수 있지만, 감시(watch)중인 항목을 삭제하면 해당 항목과 그 모든 전이적 의존성을 제거할 수 있는 Typescript의 편집기 시나리오에서는 문제가 될 수 있다.

Typescript 5.5는 삭제된 프로젝트의 일부를 새로운 생성 이벤트가 감지될 때까지 유지하는 더 정교한 접근 방식을 가지게 된다. 이러한 방식은 `npm ci`와 같은 명령어가 Typescript와 더 잘 동작하게 한다. 더 자세한 정보는 [여기로](https://github.com/microsoft/TypeScript/pull/57492)

### Symlinks are Tracked in Failed Resolutions
Typescript가 모듈을 해석하지 못할 때, 모듈이 나중에 추가될 경우를 대비해 해석에 실패한 경로를 살펴볼 필요가 있다. 이전에는 심볼릭 링크된 디렉토리에 대해서는 이 작업이 수행되지 않았고, 이로 인해 하나의 프로젝트에서 빌드가 일어났을 때 다른 프로젝트에서 이를 인식하지 못하는 모노레포와 같은 시나리오에서 신뢰성 문제가 발생할 수 있었다. Typescript 5.5에서는 이러한 문제가 해결되어, 에디터를 자주 다시 시작할 필요가 없을 것이다.

[PR에서 더 자세한 정보 알아보기!](https://github.com/microsoft/TypeScript/pull/58139)

### Project References Contribute to Auto-Imports
auto import는 더 이상 프로젝트 reference 설정에서 종속 프로젝트에 적어도 하나의 명시적인 import를 요구하지 않는다. 대신에, `tsconfig.json`의 `references` 필드에 나열된 항목에 대해 auto import 완성이 제대로 작동해야 한다.

[PR에서 더 자세한 정보 알아보기](https://github.com/microsoft/TypeScript/pull/55955)

---
## Performance and Size Optimizations
### Monomorphized Objects in Language Service and Public API
Typescript 5.0에서, `Node` 및 `Symbol` 객체가 일관된 properties들과 초기화 순서를 가지도록 했다. 이를 통해 다양한 작업에서 다형성이 줄어들어, 런타임에서 properties를 더 빠르게 가져올 수 있다.

이 변경을 통해 컴파일러의 속도가 크게 향상되는 것을 목격했다. 그러나, 이러한 변경의 대부분은 데이터 구조의 내부 할당자(allocator)에 대해 수행되었다. Language Service 및 Typescript의 공개 API는 특정 객체에 다른 할당자를 사용한다. 이를 통해 language service에서만 사용되는 데이터는 컴파일러에서 사용되지 않으므로 Typescript 컴파일러가 더 가벼워질 수 있었다.

Typescript 5.5에서는 Language service와 공개 API에 대해 동일한 단형화 작업이 수행되었다. 이는 에디터와 타입스크립트 API를 사용하는 빌드 도구가 상당히 빨라진다는 것을 의미한다. 실제로 벤치마크에서 공개 API의 할당자를 사용할 때 빌드 시간이 5~8% 빨라지고, language service 작업이 10~20% 빨라지는 것을 확인했다. 이로 인해 메모리의 사용이 증가할 수 있지만, 충분히 가치가 있다고 생각하며 메모리 오버헤드를 줄일 방법을 찾을 계획이다.

[더 자세한 정보는 여기로](https://github.com/microsoft/TypeScript/pull/58045)

(내용 추가)
- 할당자(allocator) ([위키백과 - 할당자](https://ko.wikipedia.org/wiki/%ED%95%A0%EB%8B%B9%EC%9E%90))
	- 프로그래밍에서 메모리 할당을 관리하는 컴포넌트 또는 알고리즘
- language service
	- 개발자가 코드를 작성하는 동안 보다 향상된 개발 경험을 제공하기 위한 기능 모음
	- [TypeScript-Compiler-Notes : GLOSSARY.md](https://github.com/microsoft/TypeScript-Compiler-Notes/blob/main/GLOSSARY.md) 참고

### Monomorphized Control Flow Nodes
Typescript 5.5에서는 제어 흐름 그래프의 노드가 단형화되어 항상 일관된 형태를 유지하도록 변경된다. 이를 통해, 체크 시간이 1% 정도 단축된다.

[더 자세한 정보는 여기](https://github.com/microsoft/TypeScript/pull/57977)

(내용 추가)  
프로그래밍에서 다형성(polymorphism)이란, 프로그램 언어의 각 요소들이 다양한 자료형(type)에 속하는 것이 허가되는 성질을 가리킨다. 반댓말은 단형성으로 프로그램 언어의 각 요소가 한 가지 형태만 가지는 성질을 가리킨다.

즉, 이 변경 사항의 핵심은 기존에 다형적 구조를 가진 컴파일러의 제어 흐름 노드가 단형화되었다는 것이다.
### Optimizations on our Control Flow Graph
많은 경우, 제어 흐름 분석은 새로운 정보를 제공하지 않는 노드를 탐색하게 된다. 특정 노드의 선행 조건에 조기 종료 또는 효과가 없는 경우, 이러한 노드들은 항상 건너뛸 수 있다는 것을 관찰했다. 이에 따라, Typescript는 제어 흐름 그래프를 구성할 때 제어 흐름 분석에 유용한 정보를 제공하는 이전 노드와 연결해 이를 활용한다. 이로 인해 더 평탄한 제어 흐름 그래프가 생성되고 탐색이 더 효율적일 수 있게 된다. 이러한 최적화는 적절한 성능 향상을 제공하며, 특정 코드에서는 빌드 시간이 최대 2% 감소하는 결과를 가져왔다.

[더 자세한 정보는 여기](https://github.com/microsoft/TypeScript/pull/58013)

### Skipped Checking in `transpileModule` and `transpileDeclaration`
Typescript의 `transpileModule` API는 단일 Typescript 파일의 내용을 Javascript로 컴파일하는데 사용할 수 있다. 이와 유사하게, `transfileDeclaration` API(아래 참조)는 단일 Typescript 파일에 대한 선언 파일(Declaration files)을 생성하는데 사용할 수 있다. 이러한 API의 문제 중 하나는, Typescript가 결과를 출력하기 전에 내부적으로 전체 타입 검사 과정을 수행한다는 것이었다. 이는 출력 단계에서 사용할 특정 정보를 수집하기 위해 필요했다.

Typescript 5.5에서, 전체 타입 검사를 진행하지 않고 필요한 경우에만 정보를 수집하는 방법을 찾았으며, `transpileModule`과 `transpileDeclaration`은 기본적으로 이 기능을 활성화한다. 이로 인해 [ts-loader](https://www.npmjs.com/package/ts-loader)의 `transpileOnly` 및 [ts-jest](https://www.npmjs.com/package/ts-jest)와 같이 이런 API와 통합되는 도구는 눈에 띄는 속도 향상을 경험할 수 있게 됐다. 우리의 테스트 환경에서, 일반적으로 [`transpileModule`을 사용할 때 빌드 시간이 약 2배 빨라지는 것을 확인했다.](https://github.com/microsoft/TypeScript/pull/58364#issuecomment-2138580690)

### Typescript Package Size Reduction
[Typescript 5.0에서 진행한 Module로의 migration](https://devblogs.microsoft.com/typescript/typescripts-migration-to-modules/)을 더 활용하여, [`tsserver.js`와 `typingInstaller.js`가 각각 독립적인 번들을 생성하는 대신 공통 API 라이브러리에서 가져오도록 하여](https://github.com/microsoft/TypeScript/pull/55326) 전체 패키지 크기를 크게 줄였다.

이로 인해 Typescript의 디스크 상 크기는 30.2 MB에서 20.4 MB로 줄어들었고, 압축된 크기는 5.5 MB에서 3.7 MB로 감소했다!
### Node Reuse in Declaration Emit
`isolatedDeclarations` 기능을 지원하기 위해 작업하는 과정에서, Typescript가 선언 파일을 생성할 때 input 소스 코드를 직접 복사하는 빈도를 크게 개선했다.

예를 들어 다음과 같은 코드를 작성한다고 할 때
```typescript
export const strBool: string | boolean = "hello";
export const boolStr: boolean | string = "world";
```

union 타입은 같지만 그 순서가 다르다. 선언 파일을 생성할 때, Typescript는 두 가지의 가능한 결과를 내놓을 수 있다.

첫번째는 각각 타입에 대해 일관성 있는 표현식을 사용하는 것이다
```typescript
export const strBool: string | boolean;
export const boolStr: string | boolean;
```

두번째는 쓰인 그대로 type 선언을 재사용하는 것이다
```typescript
export const strBool: string | boolean;
export const boolStr: boolean | string;
```

두번째 방식이 다음과 같은 이유로 더 선호된다
- 많은 비슷한 표현들이 있지만, 여전히 선언 파일에서 유지하는 것이 더 낫다는 의도를 담고 있다
- 타입의 새로운 표현을 생성하는 것은 다소 비용이 들 수 있으므로, 이를 피하는 것이 좋다
- 사용자가 작성한 타입이 일반적으로 생성된 타입 표현보다 더 짧다.

Typescript 5.5에서는, Typescript가 입력 파일에 작성된 타입을 정확히 그대로 출력할 수 있도록 개선했다. 대부분의 경우, 이러한 개선은 성능 향상 측면에서 눈에 보이지 않을 수 있다. 이전에는 Typescript가 syntax 노드를 새롭게 생성하고 이를 문자열로 직렬화하는 과정이 필요했다. 이제는, Typescript가 원래의 syntax node에서 직접 작업할 수 있게 되어 훨씬 더 저렴하고 빠르다.

### Caching Contextual Types from Discriminated Unions
Typescript가 object literal과 같은 표현식의 문맥적 타입을 요청할 때, 종종 유니온 타입을 직면하게 된다. 이런 경우, Typescript는 알려진 값을 가진 속성을 기준으로 union의 구성원을 필터링하려고 한다. 이 작업은 특히 많은 속성으로 구성된 객체를 다룰 때 꽤 비용이 많이 들 수 있다.

Typescript 5.5에서는 [이러한 계산의 대부분을 한 번만 캐시하여 Typescript가 object literal의 모든 속성에 대해 재계산할 필요가 없도록 했다.](https://github.com/microsoft/TypeScript/pull/58372) 이 최적화 덕분에 Typescript 컴파일러 자체를 컴파일하는데 250ms가 단축되었다.

---
## Easier API Consumption from ECMAScript Modules
이전에는 Node.js에서 ECMAScript 모듈을 작성할 때, named import는 Typescript 패키지에서 사용할 수 없었다.
```typescript
import { createSourceFile } from "typescript"; // ❌ error

import * as ts from "typescript";
ts.createSourceFile // ❌ undefined???

ts.default.createSourceFile // ✅ works - but ugh!
```

이는 [cjs-module-lexer](https://github.com/guybedford/es-module-lexer)가 Typescript가 생성한 CommonJS 코드를 해석하지 못했기 때문이다. 이 문제가 해결되어 이제 사용자는 Node.js의 ECMAScript 모듈에서 Typescript npm 패키지로부터 named import를 사용할 수 있다.

---
## The `transpileDeclaration` API
Typescript의 API는 `transpileModule`이라는 함수를 제공한다. 이 함수는 하나의 Typescript 파일을 쉽게 컴파일할 수 있도록 설계되었다. 그러나 전체 프로그램에 대한 접근 권한이 없기 때문에, 코드가 `isolateModules` 옵션에서 오류를 일으키는 경우 올바른 결과값을 만들어내지 못할 수 있다.

Typescript 5.5에서는 `transpileDeclaration`이라는 새로운 비슷한 API가 추가되었다. 이 API는 `transpileModule`과 비슷하지만, 특정 원본 텍스트를 기반으로 단일 declaration file을 생성하도록 설계되었다. `transpileModule`과 같이 전체 프로그램에 대한 접근 권한은 없으며, 비슷한 주의 사항이 적용된다. 즉, 입력 코드가 `isolatedDeclarations`  옵션 하에서 오류가 없을 때만 정확한 선언 파일을 생성할 수 있다.

원하는 경우, 이 함수는 `isolatedDeclarations`  모드에서 모든 파일에 걸쳐 declaration 생성을 병렬화하는데 사용할 수 있다.

더 자세한 정보는 [PR에서 확인](https://github.com/microsoft/TypeScript/pull/58261)

---
## Notable Behavioral Changes
이 섹션은 업그레이드로 인해 변경된, 이해하고 알고 있어야 할 변경사항에 대해 다룬다. deprecation, removal 혹은 새로운 제한 사항이 될 수도 있으며, 기능적으로 개선된 버그 수정이 포함될 수 있지만, 이는 새로운 오류를 야기해 기존 빌드에 영향을 미칠 수도 있다.
### Disabling Features Deprecated in Typescript 5.0
Typescript 5.0에서는 다음 옵션과 동작이 deprecate 되었다.
- `charset`
- `target: ES3`
- `importsNotUsedAsValues`
- `noImplicitUseStrict`
- `noStrictGenericChecks`
- `keyofStringsOnly`
- `suppressExcessPropertyErrors`
- `suppressImplicitAnyIndexErrors`
- `out`
- `preserveValueImports`
- `prepend` in project references
- implicitly OS-specific `newLine`

deprecate된 위 옵션들을 사용하려면, `5.0` 과 함께 `ignoreDeprecations` 라는 새로운 옵션을 선언해야 한다.

Typescript 5.5에서는, 이 옵션들은 더 이상 동작하지 않는다. tsconfig 내에 정의할 수는 있지만, Typescript 6.0 부터는 에러로 취급될 것이다. 향후 예정된 deprecation 전략에 대해서는 [Flag Deprecatino Plan](https://github.com/microsoft/TypeScript/issues/51000)을 참고.

[이러한 사용 중단 계획에 대한 자세한 정보는 Github에서 확인할 수 있으며](https://github.com/microsoft/TypeScript/issues/51909), 코드를 적절히 조정하는 방법에 대한 제안도 포함되어 있다.

### `lib.d.ts` Changes
DOM을 위해 생성된 type은 코드의 타입 체크 과정에 영향을 끼칠 수 있다. 더 자세한 정보는 [Typescript 5.5를 위한 DOM 업데이트](https://github.com/microsoft/TypeScript/pull/58211)를 참고

### Strict Parsing for Decorators
Typescript 에서 decorator가 처음으로 도입된 이후, 문법이 더욱 엄격해졌다. Typescript는 이제 허용하는 형식(form)에 대해 더 엄격하다. 기존의 데코레이터는 에러를 방지하기 위해 괄호로 묶어야 할 수도 있다
```typescript
class DecoratorProvider {
    decorate(...args: any[]) { }
}

class D extends DecoratorProvider {
    m() {
        class C {
            @super.decorate // ❌ error
            method1() { }

            @(super.decorate) // ✅ okay
            method2() { }
        }
    }
}
```

[더 자세한 정보는 여기에서](https://github.com/microsoft/TypeScript/pull/57749)

### `undefined` is No Longer a Definable Type Name
Typescript는 내장 타입과 충돌할 수 있는 타입 선언을 허용하지 않아왔다.
```typescript
// Illegal
type null = any;
// Illegal
type number = any;
// Illegal
type object = any;
// Illegal
type any = any;
```

그러나 버그로 인해, 이는 `undefined` 에 대해서는 적용되지 않았다. 5.5부터는 정상적으로 에러로 처리되게 된다.
```typescript
// Now also illegal
type undefined = any;
```

`undefined`라는 이름의 타입 선언에 대한 단순 참조는 처음부터 작동하지 않았다. 정의할 수는 있었지만, 이를 자격 없는 타입 이름(unqualified type name)으로 사용할 수는 없었다.
```typescript
export type undefined = string;
export const m: undefined = "";
//           ^
// Errors in 5.4 and earlier - the local definition of 'undefined' was not even consulted.
```

[더 자세한 정보는 여기에서](https://github.com/microsoft/TypeScript/pull/57575)

### Simplified Reference Directive Declaration Emit
declaration file을 생성할 때, Typescript는 필요하다고 판단이 되면 reference directive를 생성한다. 예를 들어, 모든 Node.js 모듈은 전역으로 생성되기 때문에 모듈 해석만으로는 로드할 수 없다. 아래와 같은 파일의 경우는
```typescript
import path from "path";
export const myPath = path.parse(__filename);
```

원래 코드에 reference directive가 등장하지 않았음에도 다음과 같은 declaration file을 생성하게 된다
```typescript
/// <reference types="node" />
import path from "path";
export declare const myPath: path.ParsedPath;
```

Typescript는 또한, 필요하지 않다고 판단이 되면 reference directive를 제거한다. 예를 들어, `jest`로 reference directive를 선언한다고 해도, declaration file을 생성할 때는 reference directive가 필요하지 않을 수 있다. Typescript는 이를 그냥 제거해버린다.

```typescript
/// <reference types="jest" />
import path from "path";
export const myPath = path.parse(__filename);
```

Typescript는 위 코드가 주어져도 아래와 같은 결과를 만든다.
```typescript
/// <reference types="node" />
import path from "path";
export declare const myPath: path.ParsedPath;
```

`isolatedDeclarations` 과정에서, 타입 검사를 하지 않거나 하나 이상의 파일 컨텍스트(file context)를 사용하지 않고 선언 파일을 생성하려고 하는 자들에게 이 로직이 실현 불가능하다는 것을 깨달았다. 또한, 이 동작은 사용자 입장에서도 이해하기 어렵다. 타입 검사 중에 무슨 일이 일어나는지 정확히 알지 않는 한, reference directive가 생성된 파일에 나타나는지 그 여부가 일관되지 않고 예측하기 어렵다. `isolatedDeclarations`가 활성화된 상태에서 declaration file이 다르게 작동하는 것을 방지하기 위해, 우리는 생성 방식을 변경해야 한다는 것을 깨달았다.

[실험](https://github.com/microsoft/TypeScript/pull/57569)을 통해, Typescript가 reference directive를 생성한 거의 모든 경우가 Node.js나 react를 포함시키기 위한 것이라는 것을 발견했다. 이는 최종 사용자가 이미 tsconfig.json의 "types"나 다른 라이브러리의 import를 통해 해당 타입을 참조하고 있을 것이라는 기대가 있는 경우로, 더 이상 이런 reference directive를 생성하지 않아도 문제가 발생할 가능성은 매우 낮다. 이 방식은 이미 `lib.d.ts`에서 사용되고 있다는 점도 주목해야 한다. Typescript는 모듈이 `WeakMap`을 내보낼 때, `lib="es2015"`에 대한 참조를 생성하지 않고, 대신 최종 사용자가 환경의 일부로 이를 포함했을 것이라고 가정한다.


라이브러리 작성자가 작성한 reference directive에 대한 [추가 실험](https://github.com/microsoft/TypeScript/pull/57656)을 통해, 거의 모든 reference directive가 제거되어 결과에 나타나지 않는다는 것을 발견했다. 보존된 대부분의 reference directive는 제대로 작동하지 않았으며, 보존될 의도가 없는 것으로 보였다.

이러한 결과를 바탕으로, Typescript 5.5에서는 declaration file을 생성할 때 reference directive를 크게 단순화하기로 결정했다. 보다 일관된 전략을 통해 라이브러리 작성자와 사용자들이 declaration file을 더 잘 제어할 수 있도록 돕고자 한다.

Reference directive는 더 이상 자동으로 생성되지 않는다. `preserve="true"` 속성으로 주석이 추가되지 않는한, 사용자가 작성한 reference directive는 더 이상 보존되지 않는다.

구체적으로 예시를 들자면, 다음과 같은 파일은
```typescript
/// <reference types="some-lib" preserve="true" />
/// <reference types="jest" />
import path from "path";
export const myPath = path.parse(__filename);
```

아래와 같이 생성될 것이다.
```typescript
/// <reference types="some-lib" preserve="true" />
import path from "path";
export declare const myPath: path.ParsedPath;
```

이전 버전의 Tyepscript에서는 알려지지 않은 속성을 무시하기 때문에 `preserve="true"` 속성을 더해도 이전 버전과 호환될 것이다.

이 변화로 인해 성능도 개선되었다. 우리의 벤치마크에 따르면, 선언 파일 생성을 활성화한 프로젝트에서 생성(emit) 단계가 1~4% 개선되었다.

---
