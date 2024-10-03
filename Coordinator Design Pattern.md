## Coordinator Design Pattern
### Coordinator 개념
- 화면 전환만을 담당하는 것
- 뷰 컨트롤러에서 `present`, `dismiss`, `navi~.pushVC`, `navi~.popVC`를 사용하여 화면 전환을 하곤 한다.
- 그러나 이 또한, `뷰컨트롤러가 화면전환을 담당`하고 있기 떄문에 SRP를 지키지 않음

### Coordinator 적용 이유
- 화면 전환만 담당하게끔 하여 뷰컨트롤러의 화면 전환 의존성을 제거함
이로써 뷰컨트롤러는 View에 대한 처리만 할 수 있게 된다.
- 현재 화면에 다른 뷰컨이 많이 쌓여 있을 때, 어떤 특정 화면(초기 화면)으로 가야 한다면 `pop` 혹은 `dismiss`를 계속 해줘야 한다.
- 코디네이터를 사용하면 쌓인 컨트롤러 제거와 같은 역할만 담당하므로 뷰컨트롤러로부터 의존성 분리 가능하다.
용범님의 ex) 계약서 1장 2장 .. 100장 쌓였는데, 계약하기 싫으면 바로 덮어버리는 것과 같은 맥락

<br>

---

# [10월 3일 추가] 코디네이터 패턴을 적용해보며
## 초반에 잘못 이해한 코디네이터
> 아래 이미지는 잘못된(?) 이미지임, 개선 전 이미지

<img width="700" alt="스크린샷 2024-10-01 오전 1 50 35" src="https://github.com/user-attachments/assets/d3ee8e48-0b5e-4481-b930-d02b8e77a271">

- 처음에 `IssueCreator`와 `Assignee`는 `Issue List`위에서 present 되는 방식이라,
`Issue List Coordinator` 하나로만 작동하고, 내부 메소드로 상황에 맞는 `ViewController`를 띄워주면 된다고 생각했다.
- 그러나 이건 OCP를 위배하는 코드라 판단, 아래와 같이 뷰컨트롤러와 코디네이터를 1:1 맵핑해주었다.

## 새로 적용해본 코디네이터

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

<br>

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

# [10월 4일 추가] AppCoordinator가 사라져서 Main 화면 Issue List가 사라지는 문제 해결

<img width="308" alt="스크린샷 2024-10-03 오전 3 07 52" src="https://github.com/user-attachments/assets/f8a4d055-ee2d-4a0c-a9a6-0463fdf7a179">
<img width="310" alt="스크린샷 2024-10-03 오전 3 08 29" src="https://github.com/user-attachments/assets/e2421ba2-2712-4744-af30-b45d0afffb21">

- 왼쪽 사진은 SceneDelegate에서 `AppCoordinator`를 메소드 내의 지역 객체로 만든 경우이다.
	- 이때 `AppCoordinator.start()`를 해서 생성된 IssueListCoordinator를 AppCoordinator로 지정하는데, SceneDelegate 메소드가 끝나면 더이상 IssueListCoordinator를 접근할 수 있는게 없다..? 라고 이해했다 (순환참조 문제 이렇게 이해하는게 맞는지 모르겠다)
	- ARC에 대해 다시 공부해야할 필요성을 느꼈다.
- 결론은 `SceneDelegate`에서 프로퍼티로 `AppCoordinator`를 계속 들고 있어서 유지할 수 있게 하여 오른쪽과 같이 해결했다.

<br>

## Coordinator끼리 순환참조 문제 해결

<img width="1076" alt="스크린샷 2024-10-03 오전 3 41 28" src="https://github.com/user-attachments/assets/5ad44ebe-7622-4171-9727-c9aa3c450b41">

- 코디네이터 내부에 `finishDelegate`가 있는데 그걸 `weak var`가 아닌 그대로 선언해버려서 서로 묶여버리는 순환참조가 발생했다.
- `weak` 키워드를 붙임으로써 해결할 수 있었다.

### 해결했더니 아래와 같은 경고가 뜨기 시작함 (왜 와이..?)
- 이 내용은 해결하지 못 했다. 왜인지 모르겠으니,, 다음 번에 확인해보기

<img width="423" alt="스크린샷 2024-10-03 오전 3 43 56" src="https://github.com/user-attachments/assets/a6ba6564-52a5-44ec-8271-24b30964f71b">
<img width="321" alt="스크린샷 2024-10-03 오전 3 44 04" src="https://github.com/user-attachments/assets/f126239a-24ed-4bba-bf22-61f89fdc7f71">