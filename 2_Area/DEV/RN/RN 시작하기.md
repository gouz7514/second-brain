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