# SwiftLint 적용 & 겪은 에러 해결과정 기록
## 첫 번째 에러: '.xcfilelist'
- 처음 에러는 `.xcfilelist`에서 겪었는데, `Secret.xcconfig`로만 설정해둬서 발생한 문제였다. 
    ![스크린샷 2024-10-07 오후 6 22 48](https://github.com/user-attachments/assets/1204fcbc-55b1-4949-a700-07051a940d2c)
- 아래와 같이 `Secret.xcconfig` 하나, `Pods-IssueTracker.debug.xcconfig`하나 각각 넣어줘서 해결했다. (맞는지 모르겠음...)
    <img width="1430" alt="스크린샷 2024-10-08 오전 12 12 03" src="https://github.com/user-attachments/assets/2a1b02c1-ddec-41f6-b6a8-3b9292ada5df">

<br>

## 두 번째 에러: SandBox
- <img width="671" alt="스크린샷 2024-10-08 오전 12 15 38" src="https://github.com/user-attachments/assets/6cbb8dad-c97c-4d25-aef0-c7a4f982c4b2">
- `Build Settings - User Script Sandboxing`을 No로 설정해서 보안 이슈 만들어버림 (나중에 처리)

<br>

## 세 번째 에러: Lint 적용이 이상함
- excluded에 `Pods`를 넣어도 계속 외부 라이브러리의 lint까지 에러를 잡았는데, 이건 캐시 문제인지..? `swiftlint clear-cache` 명령을 하고 Xcode 강종하고나니 해결됐다. (억까인듯?)
