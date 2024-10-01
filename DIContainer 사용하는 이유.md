## DIContainer 적용 이유
- 의존성 주입을 담당하는 컨테이너로, 싱글톤 패턴으로 주로 사용된다.
- `SceneDelegate`와 같은 곳에서 아래와 같이 특정 의존성을 미리 주입해둔다. [아래 코드 링크](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/blob/S005/IssueTracker/IssueTracker/App/Sources/SceneDelegate.swift)
    ``` swift
    extension SceneDelegate {
        func registerDependency(){
            let contianer = DIContainer.shared
            
            // MARK: Domain - Dependency , 의존성 주입 순서대로 주입, DataSource -> Repository -> UseCase
            contianer.register(type: RemoteIssueDataSource.self, component: RemoteIssueDataSourceImpl())
            contianer.register(type: IssueRepository.self, component: IssueRepositoryImpl())
            contianer.register(type: FetchAllIssuesUseCase.self, component: FetchAllIssuesUseCaseImpl())
            contianer.register(type: CreateIssueUseCase.self, component: CreateIssueUseCaseImpl())
            contianer.register(type: FetchSingleIssueUseCase.self, component: FetchSingleIssueUseCaseImpl())
        
            // MARK: Feature - Dependency
            contianer.register(type: IssueDetailFactory.self, component: IssueDetailFactoryImpl())
            contianer.register(type: CreateIssueFactory.self, component: CreateIssueFactoryImpl())
            contianer.register(type: IssuelistFactory.self, component: IssuelistFactoryImpl())
        }
    }
    ```
- **같은 화면을 여러 화면에서 띄워줘야 하는 경우**에 재사용성으로 인한 효율적인 동작을 하게 된다.

## 24/10/02 추가

## DIContainer 적용
> DIContainer 적용이 본 프로젝트가 처음입니다.

### 도입하려고 하는 이유
- **새로운 화면으로 전환**하고자 할 때,
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

<br>

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

<br>

### DIContainer 활용법 질문
- 현재 제 코드는 init( ~~ ) 생성자의 프로퍼티에서 디폴트 값으로 모두 채워두었습니다.
    - 따라서 제 코드는 재사용성 & 의존성 주입을 담당하긴 하지만,
    수작업으로 `OOO UseCase`는 `OOO Repository`를 쓸 거야. `ㅁㅁㅁ UseCase`는 `ㅁㅁㅁ Repository`를 쓸거야 라고 단정지어둔 느낌이라 확장성이나 유지보수가 좋은 코드는 아니라 생각합니다.

- 저는 DIContainer의 장점이 `의존성 주입을 담당`한다는 것 외에도,
동일한 한 화면을 여러 화면에서 띄운다면 재사용성의 장점도 있다고 생각합니다.
    - e.g. 쇼핑 앱의 장바구니..?

<br>

제 코드의 DIContainer는 `의존성 주입`과 `재사용성`은 지원하지만, 유지보수가 좋은 코드가 아니라 생각되는데

위 상황에서 멘토님의 의견이 궁금합니다 !

<br>