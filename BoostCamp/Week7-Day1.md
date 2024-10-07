## 오늘 한 일
- CocoaPod으로 SwiftLint 적용 (ToastView도 했는데 간단하니 패스)
    - Xcode 16.0에 따른 코코아팟 설치 문제 해결
    - SandBox 에러 해결
    - Lint 적용 문제 해결
- TabBarCoordinator 적용
- 스유 공부(예정)

<br>

## 상세 과정 기록
- [Xcode16.0에서 CocoaPods install 안되는 문제 해결 기록](https://github.com/Kyxxn/TIL/blob/main/Xcode16.0%EC%97%90%EC%84%9C%20pod%20install%20%EB%AC%B8%EC%A0%9C%20%ED%95%B4%EA%B2%B0%EA%B3%BC%EC%A0%95.md)

- [SwiftLint 적용 & 겪은 에러 해결과정 기록](https://github.com/Kyxxn/TIL/blob/main/SwiftLint%20%EC%A0%81%EC%9A%A9%20%26%20%EA%B2%AA%EC%9D%80%20%EC%97%90%EB%9F%AC%20%ED%95%B4%EA%B2%B0%EA%B3%BC%EC%A0%95%20%EA%B8%B0%EB%A1%9D.md)

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

<br>

## 기존 UIKit 프로젝트에 Swift UI 도입
### 어디까지 스유로 적용할까 ?
생각해야할 것들
- 기존의 화면은 살려야 함
- UIKit 위에 스유가 올라가야 함
- SceneDelegate를 삭제 하지 않아야 함

<br>

아래와 같이 결정
```
SceneDelegate `UIKit`
ㄴ UITabbarController `UIKit`
    ㄴ Issue `UIKit`
    ㄴ Label: `SwiftUI`
    ㄴ Milestone: `SwiftUI`
```

<br>

### 스유는 지금부터 공부 시작.
