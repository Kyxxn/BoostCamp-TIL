# 코디네이터 패턴을 적용해보며
> Input, Output 관점으로 살펴보겠습니다.

## 초반에 잘못 이해한 코디네이터

<img width="700" alt="스크린샷 2024-10-01 오전 1 50 35" src="https://github.com/user-attachments/assets/d3ee8e48-0b5e-4481-b930-d02b8e77a271">

- 처음에 `IssueCreator`와 `Assignee`는 `Issue List`위에서 present 되는 방식이라,
`Issue List Coordinator` 하나로만 작동하고, 내부 메소드로 상황에 맞는 `ViewController`를 띄워주면 된다고 생각했다.
- 그러나 이건 OCP를 위배하는 코드라 판단, 아래와 같이 뷰컨트롤러와 코디네이터를 1:1 맵핑해주었다.

## 새로 적용해본 코디네이터

1. 처음에 앱이 실행되면 `SceneDelegate`에서 최상위 코디네이터인 `AppCoordinator`를 생성하고 해당 인스턴스의 `.start()` 메소드를 호출한다.
    - 화면에 보여지는 것은 `SceneDelegate`에서 생성한 `navigationController`이고, 내부 navigationControllers의 프로퍼티를 변경함으로써 화면을 보여주는 뷰컨트롤러를 바꿔준다.
2. `AppCoordinator`는 본 프로젝트의 최초 메인화면인 `IssueList`를 보여주어야 한다.
    - `IssueListCoordinator`를 생성하고, 파라미터로 `SceneDelegate의 navigationController`를 넘겨준다.
        - 위 생성한 코디네이터의 `finishDelegate`를 AppCoordinator로 한다.
        - `.start()` 메소드를 호출한다.
            - `IssueListViewController`를 생성한다.
            - 위 뷰컨트롤러의 `coordinator 프로퍼티`를 IssueListCoordinator로 지정한다.
            - 위 뷰컨트롤러를 SceneDelegate의 navigationController의 viewControllers에 넣어준다.
    - `AppCoordinator`는 `var childCoordinators: [Coordinator]` 프로퍼티에 `IssueListCoordinator`를 생성해줬으니 해당 코디네이터를 추가한다.
3. `IssueList 뷰컨트롤러`에서 화면 전환할 수 있는 것이 우측 상단에 "+" 이슈를 생성하는 버튼이 있다.
    - `IssueList의 뷰`에서 "+" 버튼이 눌리면, Delegate에 의해 `IssueList 뷰컨`의 `coordinator.didTapCreatorButton()`을 호출한다.
    - `IssueListCoordinator`는 위 뷰컨트롤러에서 넘어온 정보를 처리할 수 있게 `IssueList 뷰컨의 델리게이트`를 채택하고 메소드를 구현한다.
        - `didTapCreatorButton` 메소드가 실행되면 `IssueCreator Coordinator`를 생성한다.
        - 자신의 `childCoordinators` 프로퍼티에 `IssueCreator Coordinator`를 추가하고, `start()`를 메소드를 호출해준다.
            - `IssueCreatorViewController`를 생성한다.
            - 위 뷰컨트롤러의 coordinator 프로퍼티를 IssueCreator Coordinator로 지정한다.
            - `IssueCreator Coordinator`의 navigationController(SceneDelegate에서 넘어온 것) 프로퍼티에 present로 `IssueCreatorViewController`를 띄워준다.

<br>
``` mermaid
sequenceDiagram
    participant SceneDelegate
    participant AppCoordinator
    participant IssueListCoordinator
    participant IssueCreatorCoordinator
    participant NavigationController
    participant IssueListViewController
    participant IssueCreatorViewController
    participant User

    SceneDelegate->>AppCoordinator: init(navigationController)
    AppCoordinator->>AppCoordinator: start()
    AppCoordinator->>IssueListCoordinator: init(navigationController)
    AppCoordinator->>IssueListCoordinator: start()
    IssueListCoordinator->>IssueListViewController: init()
    IssueListCoordinator->>IssueListViewController: set coordinator
    IssueListCoordinator->>NavigationController: set viewControllers = [IssueListViewController]
    User->>IssueListViewController: taps "+"
    IssueListViewController->>IssueListCoordinator: didTapIssueCreatorButton()
    IssueListCoordinator->>IssueCreatorCoordinator: init(navigationController)
    IssueListCoordinator->>IssueCreatorCoordinator: start()
    IssueCreatorCoordinator->>IssueCreatorViewController: init()
    IssueCreatorCoordinator->>IssueCreatorViewController: set coordinator
    IssueCreatorCoordinator->>NavigationController: present IssueCreatorViewController modally
```