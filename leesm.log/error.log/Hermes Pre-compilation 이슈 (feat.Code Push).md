#codepush
#react-native
# 사전 컴파일 (Pre-compilation)
> 사전 컴파일(Pre-compilation)이란 `Hermes` 을 엔진을 사용하고, 릴리즈 빌드중 이라면, 번들링 후 **사전 컴파일 과정을 추가로 진행하는 것(=바이트 코드 형태로 컴파일)**을 의미한다.

![[pre-compilation.gif]]

`Hermes` 이전 환경에서는 **빌드 타임 이후 생성된 번들을 런타임에 파싱하고 컴파일하여 실행하는 과정을** 거치게 된다. 대부분의 디바이스에서는 문제가 되지 않지만 저사양 기기에서는 실행 속도를 늦추게 되는 원인이 된다.

`Hermes` 엔진은 이러한 문제를 해결하기 위해 **생성된 번들을 빌드 타임에 `사전 컴파일(Pre-compilation)`하여 즉시 실행 가능한 바이트 코드(Bytecode) 형태로 변환**한다. 

빌드 타임에 컴파일하는 코드를 확인하고 싶다면 아래 링크를 통해 확인할 수 있다
- [Android Gradle Plugin](https://github.com/facebook/react-native/blob/v0.72.0/packages/react-native-gradle-plugin/src/main/kotlin/com/facebook/react/tasks/BundleHermesCTask.kt#L92-L113)
- [iOS Shell Script](https://github.com/facebook/react-native/blob/v0.72.0/packages/react-native/scripts/react-native-xcode.sh#L170-L186)
# CodePush 이슈
> CodePush란, React Native 개발자가 모바일 앱 업데이트를 사용자의 디바이스에 직접 배포할 수 있도록 하는 App Center 클라우드 서비스이다.

최근 인아웃에서 `CodePush`를 통해 배포하였고, 해당 번들 파일이 `Hermes` 컴파일 되지 않는 이슈가 발생하였다.
아래는 `AppCenter` 가 React Native 빌드 설정을 통해  `Hermes` 활성화 유무를 확인하고, `JS 번들` 파일을  컴파일하는 코드이다.

``` javascript
/* appcenter-cli Hermes 활성화 여부*/
function getAndroidHermesEnabled(gradleFile: string): boolean {
  return parseBuildGradleFile(gradleFile).then((buildGradle: any) => {
    return Array.from(buildGradle['project.ext.react'] || [])
              .some((line: string) => /^enableHermes\s{0,}:\s{0,}true/.test(line))
  })
}

function getiOSHermesEnabled(podFile: string): boolean {
  // ..
  const podFileContents = fs.readFileSync(podPath).toString()
  return /([^#\n]*:?hermes_enabled(\s+|\n+)?(=>|:)(\s+|\n+)?true)/.test(podFileContents)
}
```

해당 코드를 확인해보면, **ios는 `Podfile`에서 `hermes_enabled` 옵션을, Android 에서는 `GradleFile` 에서 `enableHermes` 옵션**을 통해 `Hermes` 활성화 유무를 확인하고 있다.

하지만 해당 코드는 **React Native 0.71  이상 부터는 유효하지 않는 코드**가 되었다. 0.71 버전 이상부터 **Hermes 활성화 방법이 변경**되었기 때문이다.

- **android/gradle.properties** 변경점
	- ![[Pasted image 20240508155103.png]]
- **ios/Podfile** 변경점
	- ![[Pasted image 20240508155234.png]]

`AppCenter`에서는 이와 같은 문제를 인식하고 있지만, 해당 문제를 처리하지 않기로 결정하였다
![[Pasted image 20240508155436.png]]
https://github.com/microsoft/appcenter-cli/issues/2212#issuecomment-1746388065

앞의 내용을 통해, 0.71 버전 이상부터는 bytecode로 컴파일 되지 않았음을 짐작할 수 있지만, 조금 더 확실하게 눈으로 확인할 수 있는 방법이 있다.

JS 번들 파일을 IDE 등으로 열어 보았을 때, 정상적으로 Hermes 컴파일이 됐다면, bytecode라서 아래와 같이 경고가 발생하지만,  

![[Pasted image 20240508160253.png]]

JS 번들은 일반 텍스트로 읽을 수 있다.

``` javascript
var __BUNDLE_START_TIME__=this.nativePerformanceNow?nativePerformanceNow():Date.now(),__DEV__=false,process=this.process||{},__METRO_GLOBAL_PREFIX__='';process.env=process.env||{};process.env.NODE_ENV=process.env.NODE_ENV||"production";  
!(function(r){"use strict";r.__r=i,r[`${__METRO_GLOBAL_PREFIX__}__d`]=function(r,n,o){if(null!=e[n])return;var i={dependencyMap:o,factory:r,hasError:!1,importedAll:t,importedDefault:t,isInitialized:!1,publicModule:{exports:{}}};e[n]=i},r.__c=o,r.__registerSegment=function(r,t,n){s[r]=t,n&&n.forEach((function(t){e[t]||v.has(t)||v.set(t,r)}))};var e=o(),t={},n={}.hasOwnProperty;function o(){return e=Object.create(null)}function i(r){var t=r
...
```

## 해결 방법
`AppCenter`에서  `Hermes` 를 사용한다고 명시적으로 적어주는 것이다. `--use-hermes` 옵션을 사용하게 되면, `appcenter-cli`는 **Hermes 활성화 여부를 체크하지 않고 항상 Hermes 컴파일을 수행**하게 된다