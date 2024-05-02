[cache] 환경에서 문제인것 같다.

스플레쉬 화면 작동 원리
	1. onCreate(메인 화면이 생성될 때) => SplashScreen.show()
	2. WebView onLoadEnd 가 호출될 때 => SplashScreen.hide()

1. Web 환경에서 bundle js (React) 파일을 불러올 수 없다
	1. Splash 화면이 안넘어가짐
		1. 로그라도 보고싶어서 => App.tsx() => useEffect(() => hide()) <= onCreate() 상황이랑 똑같음
		2. Splash 가 화면에서 안넘어가졌다
2. Local  bundle.js (RN) 파일을 불러올 수 없다

캐시 bundle1.js (main.bundle.js)
code push => main.bundle.js

main.bundle.js => 



