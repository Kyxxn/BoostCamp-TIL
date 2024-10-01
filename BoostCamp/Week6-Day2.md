# 멘토 코드리뷰 반영
## 클린 아키텍처 구조 및 역할
> 클린 아키텍처 적용이 본 프로젝트가 처음입니다.

> 지난 번에 Repository와 Service를 하나의 레이어로 묶어서 정의해두었다는 멘토님 피드백을 받고 아래와 같이 나누어보았습니다 ! [멘토님 코멘트 링크](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/120#pullrequestreview-2331094737)

### Network - Service 
- 네트워크 API를 요청하는 부분
- 이슈를 받아오거나 담당자를 받아오고, 결과를 DTO에 디코딩하여 반환해준다.

### Storage - Persistence
- 로컬 DB로부터 데이터를 받아오는 역할
- 본 프로젝트에서는 `issues.json` 더미 데이터를 받아오는 것으로 구현했다.

### Data - Repository
- Repository는 내부 프로퍼티로 두 가지를 갖는다.
    - `let service: IssueService`: 네트워크 API 요청 후 데이터 받아오기
    - `let storage: IssueStorage`: 로컬 DB 및 더미 데이터 받아오기
- service가 DTO를 반환하면 `Domain`에 맞게끔 `Entity`로 변환하는 작업을 거친다.

<br>

## DIContainer 적용
> DIContainer 적용이 본 프로젝트가 처음입니다.

### 도입하려고 하는 이유
- 새로운 화면으로 전환하고자 할 때,
Service 계층부터 ViewModel까지 다 의존성을 주입하고 컨트롤러를 생성해야 했습니다.
    ``` swift
    let issueService = IssueService()
    let issueRepository = IssuesRepository(service: issueService)
    let fetchIssuesUseCase = FetchIssuesUseCase(issueRepository: issueRepository)
    let issueListViewModel = IssueListViewModel(fetchIssuesUseCase: fetchIssuesUseCase)
    let issueListViewController = IssueListViewController(viewModel: issueListViewModel)
    window?.rootViewController = UINavigationController(rootViewController: issueListViewController)
    ```
- 컨트롤러에서 새로운 화면으로 전환하고자 할 때마다 이렇게 긴 코드를 작성한다는 관점,
그리고 컨트롤러가 의존성을 주입해준다는 관점에서 DIContainer의 필요성을 느꼈습니다.

### 내가 DIContainer를 사용하는 방법
- `SceneDelegate`에서 아래와 같이 DIContainer에 미리 의존성을 주입해둡니다.
    ``` swift
    private extension SceneDelegate {
        func registerDependency() {
            DIContainer.shared.register(IssueStorage.self, object: IssueStorageImpl())
            DIContainer.shared.register(IssueService.self, object: IssueServiceImpl())
            DIContainer.shared.register(IssueRepository.self, object: IssueRepositoryImpl())
            DIContainer.shared.register(IssueUseCase.self, object: IssueUseCaseImpl())
            DIContainer.shared.register(IssueListViewModel.self, object: IssueListViewModel())
            DIContainer.shared.register(AssigneeRegistrationViewModel.self, object: AssigneeRegistrationViewModel())
            DIContainer.shared.register(IssueCreatorViewModel.self, object: IssueCreatorViewModel())
        }
    }
    ```
- **클린 아키텍처의 각 계층마다 의존성 주입을 init에서 디폴트 값으로 지정하고 있습니다.** (이 부분이 모호합니다)
    ``` swift
    struct IssueRepositoryImpl: IssueRepository {
        private let service: IssueService
        private let storage: IssueStorage

        init(
            service: IssueService = DIContainer.shared.resolve(IssueService.self),
            storage: IssueStorage = DIContainer.shared.resolve(IssueStorage.self)
        ) {
            self.service = service
            self.storage = storage
        }
        ...
    }
    ```

### DIContainer 활용법 질문
- 현재 제 코드는 init( ~~ ) 생성자의 프로퍼티에서 디폴트 값으로 모두 채워두었습니다.
    - 따라서 제 코드는 재사용성 & 의존성 주입을 담당하긴 하지만 수작업으로 `OOO UseCase`는 `OOO Repository`를 쓸 거야. `ㅁㅁㅁ UseCase`는 `ㅁㅁㅁ Repository`를 쓸거야 라고 단정지어두 느낌이라 확장성이나 유지보수가 좋은 코드는 아니라 생각합니다.
- 저는 DIContainer의 장점이 `의존성 주입을 담당`한다는 것 외에도,
동일한 한 화면을 여러 화면에서 띄운다면 재사용성의 장점도 있다고 생각합니다.
    - e.g. 쇼핑 앱의 장바구니..?
    - 제 코드의 DIContainer는 `의존성 주입`과 `재사용성`은 지원하지만, 유지보수가 좋은 코드가 아니라 생각되는데
    위 상황에서 멘토님의 의견이 궁금합니다 !

<br>

## cell에서 URL을 Image로 변환하기
### 상황
1. 담당자들을 보여주는 TableView을 배치하려 합니다.
2. Service 계층에서 [담당자] 배열을 받아오고, ... ViewModel에서 [담당자] Entity를 @Published로 갖고 있습니다.
3. Controller는 ViewModel의 [담당자]를 관측하고 있어서 값이 바뀌면 `TableViewDatasource` 메소드에서 cell을 configure 합니다.
    ``` swift
    extension AssigneeRegistrationViewController: UITableViewDataSource {
        
        func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {

            guard let cell = tableView.dequeueReusableCell(withIdentifier: AssigneeTableViewCell.identifier,
                                                        for: indexPath) as? AssigneeTableViewCell
            else { return UITableViewCell()}

            let user = viewModel.assignees[indexPath.row]
            cell.configure(user: user)
            return cell
        }
    }
    ```
    - user는 다음과 같은 Entity 입니다.
    ``` swift
    struct UserEntity: Codable {
        let id: Int
        let login: String
        let avatarURL: String?
    }
    ```
4. **여기서 궁금한 사항이 생겼습니다.**
cell이 user를 받게 되면 `avatarURL`을 갖고 UIImage로 변환을 해야 합니다.

### 내가 떠올린 해결방안
- cell은 보여지는 `View`인데, URL을 통해 이미지를 받아오는 작업을 한다는 것은 아키텍처 구조상 맞지 않다고 생각했습니다.
- 제가 떠올린 방법으로 `ImageLoader`라는 싱글톤을 만들고 cell은 비동기로 url을 해당 싱글톤에 넘겨주면 그 결과를 토대로 Image를 보여주려 했습니다.
- **그러나**, ImageLoader가 비동기로 이미지를 갖고 온다고 하지만, `import UIKit`을 해서 UIImage를 반환해야 하는게 맞나 ? 라고 생각했습니다.
    - 위 해결책으로 ImageLoader가 `UIImage`를 반환하지 않고, `Data`를 반환하고 cell에서 해당 Data를 UIImage로 바꾸는 작업을 하면 `ImageLoader`가 UIKit을 의존하지 않아도 될 거라 생각했습니다.
    - 그런데 또.. Data로 받는데 1번, UIImage로 바꾸는데 1번, 총 2번의 이미지 처리를 해야하니 굉장히 비효율적일 것 같습니다!


위 상황에 대해 여러가지 방안을 떠올려봤는데,
멘토님의 견해가 궁금합니다..