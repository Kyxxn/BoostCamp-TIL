URLSession Data / DataTask
## 오늘 한 일
- 오늘 모듈화를 성공적으로 마치고, 로그인을 구현할 때 의존성을 계속 생각하며 코드를 작성했다.
    - 그 결과물로 `GitHub OAuth` 로그인에 대한 Flow를 고민했고, 아래와 같이 진행하기로 결정했다.
- KeyChain에 대한 사용법을 익혔으나, 공부는 아직 제대로 안됨

<br>

### 모듈화를 적용한 내 아키텍처에서 새로운 Feature 개발
> 새로운 Feature라기엔 의존성이 너무 왔다갔다하는 작업이라 거의 최종보스 급이었다.

<img width="882" alt="스크린샷 2024-10-15 오후 8 25 03" src="https://github.com/user-attachments/assets/dcf3a854-0da9-477a-aaf1-6bc0895b060a">

- 위 사진처럼 단계를 따른다. (DIP는 생략)
- OpenURL을 갔다오면 SceneDelegate의 아래 메소드에 깃허브가 넘겨준 Code라는 데이터가 넘어온다.
    ` func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>)`
    - 이 데이터를 NotificationCenter로 뷰모델에 보내도록 처리
- ViewModel에 보내지 않고 바로 Network에 보낼 수도 있으나, 클린 아키텍처의 구조와 Combine으로 뷰가 바인딩할 수 있게끔 하고자 위처럼 구현했다.

<br>

### KeyChain
공부하자.


<br>

### 레퍼런스
- [KeyChain](https://velog.io/@tmdckd232/iOS-Keychain%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EC%9E%90)
