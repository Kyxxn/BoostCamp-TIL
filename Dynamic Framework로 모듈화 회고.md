## Dynamic Framework로 모듈화한 과정
> Dynamic Framework를 사용했다.

1. New - project 생성 후 그 안에 파일을 타겟 설정해서 채워넣고 빌드를 한다.
2. 위 프로젝트를 종료하고, 위 디렉토리 전체를 IssueTracker 안에 넣는다.
3. xcworkspace를 켜서 맨 바깥에 .xcodeproj에 대한 Reference만 추가

<br>

## 하다가 알게된 TMI
0. Foundation에 Combine은 내가 아는 Combine과 다름
    - sink와 같은 것들은 import Foundation만 해서는 안됨

1. `Command SwiftVerifyEmittedModulelnterface failed with a nonzero exit code` 에러가 뜨면 `Build Libraries for Distribution` 옵션을 `Yes -> No`로 변경하면 된다.
   
   <img width="285" alt="스크린샷 2024-10-14 오후 9 34 03" src="https://github.com/user-attachments/assets/a9ea1828-5ea1-498d-a7cf-4c938c151a62">

    - `Build Libraries for Distribution`가 활성화되면 컴파일러는 프레임워크를 빌드할 때마다 Swift Module Interface 포맷의 파일인 `.swiftinterface파일을 생성`한다. [관련 레퍼런스](https://minsone.github.io/ios/mac/ios-wwdc-2019-binary-frameworks-in-swift-little-summary-and-translate)
    - 그러나, 모든 계층에 해당 옵션을 다 활성화해버리면 여러 개의 `.swiftinterface` 파일이 생성되는 것으로 추측..
    - 그래서 의존 관계를 아무것도 갖지 않는 `Domain` 계층에만 `YES`로 활성화하고, 나머지는 모두 `No`로 해줘야 에러가 발생하지 않는다.

3. 파일을 꼭 하나 추가하고 할 것. (Empty 파일을 만들면 타겟 추가가 안됨)
    - Storage 계층 만들다가 알게된 건데, 위 모듈화 과정에서 1번 동작을 꼭 해줘야 `import Storage`가 가능함

4. Network는 이미 존재하는 프레임워크이다.
   <img width="481" alt="스크린샷 2024-10-15 오전 1 02 33" src="https://github.com/user-attachments/assets/1c74cb9a-3a3c-4af7-bcf0-2822246d9f77">
    - 네트워크 프레임워크 이름을 `Network`로 했음.. (여기서 3시간 길바닥에 버림)
    - `NetworkService`로 수정

5. Bundle 주입 문제
    - 오강훈 멘토님께서 Bundle에서 토큰 주입이나 URL 주입 등등을 분리해야할 필요가 있다고 말씀하심
    - 무슨 뜻인지 몰랐으나, `Storage`계층에서 `Resource - issues.json`에 있는 JSON을 Bundle로 막 가져다 썼는데, 모듈화를 하니까 이게 안됨
    - 의존성 주입이 필요했음

### 4번 과정 설명
<img width="264" alt="스크린샷 2024-10-15 오전 3 09 33" src="https://github.com/user-attachments/assets/280dec7d-987f-4929-b1ca-b98e2b655c61">
<img width="268" alt="스크린샷 2024-10-15 오전 3 10 42" src="https://github.com/user-attachments/assets/706806f8-2ce0-47aa-a731-ad7ab9474c2e">

- 상황: `StorageService` 내부에서 `Bundle.main.url(forResource: "issues", withExtension: "json")`를 사용한다. `issues.json`을 어디에 두는게 맞을까 ?
- 좌측 사진처럼 프레임워크와 같은 계층이 두면 안됨
- 우측 사진처럼 해야 Bundle 접근이 가능.
- 근데 이렇게 하느니 `IssueTracker의 SceneDelegate`와 같은 곳에서 의존성을 주입해주는 게 맞다고 봄.
- [위 문제를 해결하여 개선한 승완님 코드](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/181/files#diff-b344ac1dd83fb6b13655429981de671153d3f387fea4e30a1a939ae1888d52b0)

<br>

### 최종
<img width="238" alt="스크린샷 2024-10-15 오전 3 14 46" src="https://github.com/user-attachments/assets/cd473cd8-19c4-41a7-8860-61b9b0a3c113">

<img width="506" alt="스크린샷 2024-10-15 오전 3 24 52" src="https://github.com/user-attachments/assets/dd816f54-5c93-401e-8102-e15922fd325b">

끝나니까 새벽 3시 반이지만, 언젠가 한 번은 해봤어야 할 일이라 느껴져서 힘든건 상관없다.

`Network` 프레임워크 이름 잘못 지은 거.. 해결해서 다행이다.

### Reference
- [Build Libraries for Distribution 설명](https://minsone.github.io/ios/mac/ios-wwdc-2019-binary-frameworks-in-swift-little-summary-and-translate)
- [Framework 모듈화 작업](https://daddy73e.tistory.com/4)
