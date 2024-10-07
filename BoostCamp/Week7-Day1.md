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

- SwiftLint 적용
    - .xcfilelist 에러
        - 처음 에러는 `.xcfilelist`에서 겪었는데, `Secret.xcconfig`로만 설정해둬서 발생한 문제였다. 
          ![스크린샷 2024-10-07 오후 6 22 48](https://github.com/user-attachments/assets/1204fcbc-55b1-4949-a700-07051a940d2c)
        - 아래와 같이 `Secret.xcconfig` 하나, `Pods-IssueTracker.debug.xcconfig`하나 각각 넣어줘서 해결했다. (맞는지 모르겠음...)
          <img width="1430" alt="스크린샷 2024-10-08 오전 12 12 03" src="https://github.com/user-attachments/assets/2a1b02c1-ddec-41f6-b6a8-3b9292ada5df">
    - SandBox 에러
        - <img width="671" alt="스크린샷 2024-10-08 오전 12 15 38" src="https://github.com/user-attachments/assets/6cbb8dad-c97c-4d25-aef0-c7a4f982c4b2">
        - `Build Settings - User Script Sandboxing`을 No로 설정해서 보안 이슈 만들어버림 (나중에 처리)
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
