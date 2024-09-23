> [!tip] 공식 블로그 작성일 : 2024.09.09
> [원문 링크](https://devblogs.microsoft.com/typescript/announcing-typescript-5-6/)

Typescript 5.6 베타 버전 이후로, [Typescript의 언어 서비스가 `tsconfig.json` 파일을 검색하는 방식과 관련된 변경 사항](https://devblogs.microsoft.com/typescript/announcing-typescript-5-6-beta/#search-ancestor-configuration-files-for-project-ownership)을 revert했다.  
이전에는 `tsconfig.json`이라는 이름을 가진 모든 프로젝트 파일을 찾기 위해 계속해서 탐색했다. 이로 인해 많은 참조된 프로젝트가 열릴 수 있기 때문에, [이 동작을 되돌렸으며](https://github.com/microsoft/TypeScript/pull/59634) [Typescript 5.7에서 이를 다시 도입할 방법](https://github.com/microsoft/TypeScript/pull/59688)을찾고 있다.

추가로, 베타에 비해 몇몇 새로운 타입의 이름이 변경되었다. 이전에 Typescript는 `Iterator.prototype`을 기반으로 하는 모든 값을 설명하기 위해 `BuiltinIterator`라는 단일 타입을 제공했다. 이 타입의 이름이 `IteratorObject`로 변경되었으며, 다른 타입 파라미터를 가지게 되었다. 또한 `ArrayIterator`, `MapIterator` 등의 여러 하위 타입이 추가되었다.

`--stopOnBuildErrors`라는 이름의 새로운 플래그가 `--build` 모드에 추가되었다. 프로젝트 빌드시에 에러가 발생하면, 다른 프로젝트도 빌드를 중단할 것입니다. 이 플래그는 Typescript 5.6 이전 버전의 동작고 유사한 기능을 제공한다. [Typescript 5.6에서는 에러가 발생하더라도 항상 빌드를 진행하기 때문](https://devblogs.microsoft.com/typescript/announcing-typescript-5-6/#allow---build-with-intermediate-errors)이다.

새로운 에디터 기능이 추가되어 [커밋 문자에 대한 직접 지원](https://devblogs.microsoft.com/typescript/announcing-typescript-5-6/#granular-commit-characters)과 [자동 임포트에서 제외 패턴을 설정](https://devblogs.microsoft.com/typescript/announcing-typescript-5-6/#exclude-patterns-for-auto-imports)할 수 있다.

