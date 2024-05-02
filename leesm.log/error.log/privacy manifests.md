앱(=인아웃)을 빌드하고 배포하다. 아래와 같은 메일을 **App Store Connect** 측에서 받게되었다.

![[Pasted image 20240502102438.png]]

메일의 내용은 **앱 개인정보 보호 매니페스트(Privacy Manifest)가 도입되면서,  2024년 5월부터 1일까지 Apple 필수 API를 사용했을 때 사용 이유를 설명하지 않으면 앱이 통과되지 않는다 라는 내용**이였다.  
  
해당 메일을 받고 **Archive**에서 **generate Privacy Report**를 통해 Privacy 정보를 확인한 결과

![[Pasted image 20240502102715.png]]

**AdbrixRmKit** 의 **PrivacyInfo.xcprivacy**에서 **NSPrivacyCollectedDataTypes** Key를 찾을 수 없다는 결과를 확인할 수 있었다.

이 이슈에 대해 인아웃에서 **Privacy Manifest**를 따로 생성하여 처리해야 하는건지, **Adbrix**에서 추후 관련 업데이트가 있는 건지 메일을 요청하였고

![[Pasted image 20240502102945.png]]

위와 같은 답변을 받을 수 있었다.

참고:
[apple developer](https://developer.apple.com/support/third-party-SDK-requirements/ild)