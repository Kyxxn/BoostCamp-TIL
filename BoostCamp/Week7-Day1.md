## 오늘 한 일
- CocoaPod으로 SwiftLint 적용 (ToastView도 했는데 간단하니 패스)
    - Xcode 16.0에 따른 코코아팟 설치 문제 해결
    - SandBox 에러 해결
    - Lint 적용 문제 해결
- TabBarCoordinator 적용
- 스유 공부(예정)

<br>

## 상세 과정 기록
- [Xcode16.0에서 CocoaPods install 안되는 문제 해결 기록]()
- SwiftLint 적용
    - SandBox 에러를 겪었는데, `Build Settings - User Script Sandboxing`을 No로 설정해서 보안 이슈 만들어버림 (나중에 처리)
    - excluded에 `Pods`를 넣어도 계속 외부 라이브러리의 lint까지 에러를 잡았는데, 이건 캐시 문제인지..? `swiftlint clear-cache` 명령을 하고 Xcode 강종하고나니 해결됐다. (억까인듯?)
- TabBarCoordinator 적용
    - 늘 하던대로 `SceneDelegate -> AppCoordinator -> TabBarCoordinator`를 `start()`해준다.
    - `TabBarCoordinator`에서는 내가 설정한 TabBarItem에 따라 각각 Navigation을 만들어서 해당 Item에 맞는 Coordinator를 설정한다. (지금은 IssueList만 박아둠)
    - 4개의 Item에 대한 코디네이터를 모두 start()하여 TabBarCoordinator - childCoordinators 프로퍼티에서 각 코디네이터 관리해준다.
    ``` swift
    // TabBarCoordinator의 주요 코드
    final class TabBarCoordinator: Coordinator {
        ...

        func start() {
        let navigationControllers = TabBarItem.allCases.map(generateNavigationController)
        
        configureTabBarController(with: navigationControllers)
        self.navigationController.viewControllers = [tabBarController]
    }
    
    private func configureTabBarController(with viewControllers: [UIViewController]) {
        tabBarController.setViewControllers(viewControllers, animated: false)
    }
    
    private func generateNavigationController(item: TabBarItem) -> UINavigationController {
        let navigationController = UINavigationController()
        navigationController.tabBarItem = item.component
        generateCoordinator(item: item, navigationController: navigationController)
        return navigationController
    }
    
    private func generateCoordinator(item: TabBarItem, navigationController: UINavigationController) {
        var coordinator: Coordinator
        switch item {
        case .issue:
            coordinator = IssueListCoordinator(navigationController: navigationController)
        case .label:
            coordinator = IssueCreatorCoordinator(navigationController: navigationController)
        case .milestone:
            coordinator = IssueListCoordinator(navigationController: navigationController)
        case .profile:
            coordinator = IssueListCoordinator(navigationController: navigationController)
        }
        addChildCoordinator(with: coordinator)
    }
    
    private func addChildCoordinator(with coordinator: Coordinator) {
        self.childCoordinators.append(coordinator)
        coordinator.start()
    }
    ```

### 스유는 지금부터 공부 시작.