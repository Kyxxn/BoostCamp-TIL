## 오늘 한 일
> [오늘 PR 바로가기](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/413#issue-2575782827)
- ViewModel의 네트워크 API 요청 로직 수정
-  UseCase 책임 분리 및 Layer 구조 변경
- Milestone 탭바 구현

<br>

### 오늘 알게된 점 정리
- [UIKit과 SwiftUI 같이 쓸 때, NavigationBar Item 설정하기](https://github.com/Kyxxn/TIL/blob/main/UIKit%EA%B3%BC%20SwiftUI%20%EA%B0%99%EC%9D%B4%20%EC%93%B8%20%EB%95%8C%20Navi%20Bar%20%EC%84%A4%EC%A0%95.md#uikit%EA%B3%BC-swiftui-%EA%B0%99%EC%9D%B4-%EC%93%B8-%EB%95%8C-navigationbar-item-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)

<br>

## 궁금한 점
### 사용되지 않는 cancellable 제거
> 호출할 때마다 cancellable이 store에 쌓이는데 사용되지 않아서 제거하려 했다.

> 사실 제거하는게 맞는지 모르겠음

   - `fetchMoreIssues`가 호출될 때마다 사용되지 않는 `cancellable`이 쌓이는 문제를 해결하였다.
   - `private var cancellable: AnyCancellable?`에 할당하여 finished 시점에서 Set 집합에 있는 해당 `cancellable`을 삭제하였다.

### Before
<img width="1578" alt="스크린샷 2024-10-09 오후 3 19 39" src="https://github.com/user-attachments/assets/76ceff01-19ea-426a-a86a-d133f785a86b">

### After
<img width="840" alt="스크린샷 2024-10-09 오후 3 29 09" src="https://github.com/user-attachments/assets/ded69155-6173-4a33-9bff-d43e56cceee5">
