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

여기서 `bird is Bird` 가 type predicate이다. 이는 만약 위 함수가 `true`를 리턴한다면, `bird` 변수는 `Bird` 타입이라는 것을 의미한다. (즉, `value is type`)

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
대신 namespace import를 사용할 수 있다

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
