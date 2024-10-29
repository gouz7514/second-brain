[RN 공식문서 - Core Components and Native Components](https://reactnative.dev/docs/intro-react-native-components)

RN은 React와 앱 플랫폼의 네이티브 특징을 활용해 안드로이드와 ios 애플리케이션을 만들기 위한 오픈 소스 프레임워크이다.

RN에서는 자바스크립트를 활용해 Native API를 활용하고, UI를 그릴 수 있다.

![[Pasted image 20241030003356.png]]

RN은 우리와 운영체제 사이에 있는 일종의 번역가이다. RN은 안드로이드와 iOS와 같은 Native와 연결해주는 bridge.

![[Pasted image 20241030003849.png]]
- Native에서 Event가 발생하면 event에 관한 데이터를 수집하고 넘긴다.
- Bridge 역할을 하는 RN은 해당 정보를 JS에게 넘기고
- JS는 정보를 바탕으로 UI를 변경하는 native method를 호출한다.
- 다시 RN을 거치고 Native로 전달돼서 UI를 바꾼다!

## View
안드로이드와 iOS 환경에서는 **view**가 UI의 기본 구성 요소!
텍스트, 이미지, 인풋, 작은 버튼까지도 모두 view의 일종이다.

## Native Components
In Android development, you write views in Kotlin or Java; in iOS development, you use Swift or Objective-C. With React Native, you can invoke these views with JavaScript using React components. At runtime, React Native creates the corresponding Android and iOS views for those components. Because React Native components are backed by the same views as Android and iOS, React Native apps look, feel, and perform like any other apps. We call these platform-backed components **Native Components.**

리액트 컴포넌트를 활용한 자바스크립트로 안드로이드, iOS view를 호출할 수 있다. RN 컴포넌트는 안드로이드 및 iOS와 동일한 view에 의해 지원되기 때문에 외형도 성능도 유사하게 유지된다. 이런 플랫폼 지원 컴포넌트를 **Native Component** 라고 부른다.

RN은 필수 Native Component를 몇가지 지원한다. 이들을 [Core Components](https://reactnative.dev/docs/intro-react-native-components#core-components)라고 부른다.