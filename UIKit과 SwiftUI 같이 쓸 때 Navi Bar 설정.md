## UIKit과 SwiftUI 같이 쓸 때, NavigationBar Item 설정하기
> 본 프로젝트는 UIHostingController를 이용해서 UIKit 위에 SwiftUI를 띄웁니다.

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