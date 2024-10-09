## 오늘 한 일
- ViewModel의 네트워크 API 요청 로직 수정
    - 뷰모델 생성 시 바로 API 호출하는 로직 제거
    - receive(on:) 메소드 제거 및 sink 메소드 수정
    - 사용되지 않는 cancellable 제거

-  UseCase 책임 분리 및 Layer 구조 변경
    - UseCase 책임 분리 (원래 UseCase 하나였는데, CRUD에 따라 나눔)
    - Data Layer에 있는 Repository interface를 Domain Layer로 이동
    - Service & Storage Layer에 있는 interface를 Data Layer로 이동

- Milestone 탭바 구현
    - MilestoneListView 코디네이터 연동
    - Milestone Fetch API 호출 구현
    - MilestoneCell 커스텀하여 List에 보여주기

## 오늘 알게된 점
### SwiftUI에서 NavigationBar Item 설정하기

- 아래 코드에서 당연히 스유의 `NavigationView`로 안 감싸져 있으니 뜨면 우측 상단에 + 버튼과 네비게이션 제목이 안 보이는게 맞다.
    - 근데 내 화면에서는 잘 보임 ㄴ(ㅇㅁㅇ)ㄱ
    ``` swift
    struct ToolbarView: View {
        var body: some View {
            List {
                Text("리스트 셀 1")
                Text("리스트 셀 2")
            }
            .navigationTitle("네비게이션바 제목")
            .toolbar {
                ToolbarItem(placement: .topBarTrailing) {
                    Button {
                        print("tapped")
                    } label: {
                        Image(systemName: "plus")
                    }
                }
            }
        }
    }
    ```
- 이유를 알아보니 내 모든 구조는 UIKit 코디네이터 패턴을 사용하고 있으므로 스유를 포함한 모든 화면이 네비게이션으로 구성되어 있다.
    ``` swift
    final class ToolbarCoordinator: Coordinator {
        var navigationController: UINavigationController
        
        ...
        
        func start() {
            var toolbarView = ToolbarView()
            let toolbarHostingController = ToolbarHostingController(rootView: toolbarView)
            navigationController.viewControllers = [toolbarHostingController]
        }
    }
    ```
    - HostingController를 통해서 스유 화면을 담아서 UIKit의 `NavigationController`에 띄우고 있다.
    - 따라서 스유의 `NavigationView`를 안 써도 UIKit의 네비를 통해 네비게이션 바가 보이는 것!
- 추가로 아래와 같이 내가 커스텀 해준 다음 아래와 같이 작성하면 스유 화면에서 Navigation 제목, 아이템을 설정하지 않아도 UIKit 네비 구성으로 들어간다!
    ``` swift
    final class ToolbarHostingController: UIHostingController<ToolbarView> {
        override func viewDidLoad() {
            super.viewDidLoad()
            navigationController?.navigationBar.prefersLargeTitles = true
            navigationItem.title = "네비게이션바 제목"
            navigationItem.rightBarButtonItem = UIBarButtonItem(
                barButtonSystemItem: .add,
                target: self,
                action: #selector(tapped)
            )
        }
    }
    ```

<br>

## 궁금한점
### 사용되지 않는 cancellable 제거
> 제거하는게 맞는지 모르겠음

   - `fetchMoreIssues`가 호출될 때마다 사용되지 않는 `cancellable`이 쌓이는 문제를 해결하였다.
   - `private var cancellable: AnyCancellable?`에 할당하여 finished 시점에서 Set 집합에 있는 해당 `cancellable`을 삭제하였다.

### Before
<img width="1578" alt="스크린샷 2024-10-09 오후 3 19 39" src="https://github.com/user-attachments/assets/76ceff01-19ea-426a-a86a-d133f785a86b">

### After
<img width="840" alt="스크린샷 2024-10-09 오후 3 29 09" src="https://github.com/user-attachments/assets/ded69155-6173-4a33-9bff-d43e56cceee5">