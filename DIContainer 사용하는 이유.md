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