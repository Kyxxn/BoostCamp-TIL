## 오늘 한 일
- SwiftUI로 `Form`, `Section`, `@ViewBuilder`, ..을 사용해 뷰를 그려봤다.
- MVVM과 Coordinator를 같이 할 때, ViewModel에서 coordinator를 관리해야 할 필요성을 느꼈다.

<br>

### MZ한 SwiftUI
- Combine을 배운지 얼마 안된 나지만, 스유는 엠지한 거 같다.
- 아래 코드처럼 적으면 TextField에 글자를 적으면 `viewModel.label.name` 값이 바뀌고, 그에 따라 `viewModel.label`이 바뀌면 동작하는 `LabelTagView`도 알아서 바뀐다.
    ``` swift
    struct LabelFormView: View {
        weak var delegate: LabelFormViewDelegate?
        @ObservedObject var viewModel: LabelFormViewModel    

        var body: some View {
            TextField("필수 입력", text: $viewModel.label.name)
            ..
        }
    }
    ```
- 이게 선언형..? ㄴ(ㅇㅁㅇ)ㄱ

<br>


### MVVM + Coordinator에서 VM이 코디네이터 관리
> 현재 구조는 MVVM + C이지만, View가 코디네이터 델리게이트를 들고 있습니다.

사진

0. 기획 요구사항에 따라 위 사진의 Present된 화면은 추가 & 수정 모두가 가능해야 한다.
1. 우측 상단의 저장 버튼이 눌리면 뷰(SwiftUI 기준)가 delegate를 통해 `delegate.didTapSaveButton`을 호출하여 POST 작업을 마친다.
2. POST가 완료되면 `navigationController.presentedViewController` 즉, LabelList 화면은 Fetch가 되어야 한다.
    - 그러기 위해서는 LabelList의 @Published 프로퍼티인 labels가 값이 바뀌어야 한다.
    - 지금 구조에서는 이걸 하려면 엄청 복잡한 거 같음 (머리로 암산하다가 박살남)
3. 이걸 뷰모델에서 코디네이터로 관리하면 `LabelList의 ViewModel`과 `LabelForm의 ViewModel`이 코디네이터 `LabelListCoordinator`에 바로바로 delegate를 한 번씩만 호출하면 돼서 금방 해결될 것이라 생각된다.

<br>

이 작업을 주말에 혼자 보드에 그려가며 해볼 예정이다.