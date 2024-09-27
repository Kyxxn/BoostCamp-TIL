# 피어세션에서 한 내용 정리
> Clean Architecture + MVVM + Coordinator + DIContainer에 대한 내용 정리

> 계층 별 네이밍은 다음과 같습니다.

<img width="1401" alt="스크린샷 2024-09-26 오후 4 00 16" src="https://github.com/user-attachments/assets/c9d7c7ce-cf6a-44ae-8e68-189084b445dc">

## 계층 별로 struct와 class
- 우리가 평소에 고민하던 내용과 같다. (참조, 상속 ..)
- 용범님: `DataSource`(= 나의 Service) 계층에서 BaseDataSource를 갖고 있기 때문에 상속을 위해 class로 구현했고, 나머지는 참조 방식이 필요없어서 struct로 구현

<br>

## Network 계층 추상화
- `EndPoint`들도 `EndPointType`과 같이 추상화하여 네트워크 작업할 때 DIP를 적용하여 문제해결 가능
- [네트워크 계층에서 EndPointType 추상타입 정의](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/blob/S074/IssueManager/NetworkKit/NetworkKit/EndPointType.swift)
- [데이터 계층에서 EndPoint 구체타입 정의](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/blob/S074/IssueManager/Data/Data/EndPoint/IssueEndPoint.swift)
    ``` swift
    enum IssueEndPoint: EndpointType {
        case fetchIssueList(state: String)
        case createIssue(dto: IssueCreationRequestModel)
    }
    ```
    이렇게 열거형으로 관리하면 `FetchIssueListEndPoint`, `CreateIssueEndPoint`와 같이 해줄 필요없음 (= 중복된 프로퍼티 코드 제거 가능)

<br>

## Repository와 Service 계층
> 클린 아키텍처에서 Data 계층과 Service 계층을 분리하였다.

<img width="719" alt="스크린샷 2024-09-27 오후 1 42 52" src="https://github.com/user-attachments/assets/371b49c2-5772-4aec-9c23-310ec6ebf3db">


- 어제까지의 나는 `Data - Repository`, `Data - Service`와 같이 Data 계층에 같이 존재한다고 생각했다.
    - [멘토님의 피드백](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/120#pullrequestreview-2331094737)
- `Data - Repository`에서는 Service 계층으로부터 받아온 데이터를 관리하는 곳이다.
    - **이때 가져온 데이터가 서버에서 온 거일 수도, DB일 수도 있다.**
- `Service - DataSource`는 데이터와 직접적으로 맞닿게 하여 데이터를 가져오는 역할을 한다.

### 정리
- Repository와 Service를 하나의 레이어로 묶어서 정의하지 말고, Clean Architecture에서는 둘은 분리되는 것이 맞는 것이 맞다.
- Service: 데이터를 가져오는 역할
- Repository: 데이터와 직접 닿기 보다 Service 계층으로부터 오는 내용을 관리해주는 역할

<br>

## DIContainer 적용 이유
- 의존성 주입을 담당하는 컨테이너로, 싱글톤 패턴으로 주로 사용된다.
- `SceneDelegate`와 같은 곳에서 아래와 같이 특정 의존성을 미리 주입해둔다. [DIContainer 링크](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/blob/S005/IssueTracker/IssueTracker/Modules/UtilityModule/DI/DIContainer/DIContainer.swift)
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

<br>

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

## 프레임워크 추가로 모듈화 구현하기
> 앞전에 창우님이 설명해주신 것과 유사하나, 그 때와 달리 `New Project`를 사용

<img width="500" alt="스크린샷 2024-09-27 오후 1 21 29" src="https://github.com/user-attachments/assets/4cb811b2-5dce-4290-a66d-bb2632d68b7c">

- `New Project` - `Framework & Library`에서 `Framework`를 선택한다.
    - 참고로 Static Library는 리소스 파일들(image, assets, nibs, strings file)을 포함할수가 없음 [레퍼런스 링크](https://medium.com/delightroom/ios-library-framework-swift-package-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-%EC%B0%A8%EC%9D%B4%EC%A0%90-1f42c7848771)
- Framework를 생성했다면, 내부에 Temp.swift 파일을 하나 만든다.
- 기존 프로젝트에서 `Add Files to '현재프로젝트'`를 하면 해당 프레임워크를 모듈로 추가해준다.
- 최상위 프로젝트(= 모듈)에서 Targets에 의존성을 줄 수 있다.

<br>

## 클로저에서 weak self
> [링크 바로가기](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/32#discussion_r1773639911)

``` swift
NetworkHandler.requestApi  { [weak self] issues in
    DispatchQueue.main.async { // 여기서 weak self 안 해도 되는가 ?
        guard let self = self else { return }
        self.issueList = issues // issueList 업데이트
        self.setUpTableView()
    }
} 
```

- Q. NetworkHandler 클로저에서는 `[weak self]`를 사용했는데,
DispatchQueue 클로저에서는 사용하지 않았다.
- A. 각 클로저마다 캡처가 동작하므로, `[weak self]`를 붙여줘야 함
안 붙여도 되는 경우가 있긴 한데, 붙여야 할 경우에 `[weak self]`를 안 쓰면 문제가 생기니 웬만하면 붙이는 것을 추천.

<br>

## 레퍼런스
- [Static Library 정리](https://medium.com/delightroom/ios-library-framework-swift-package-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-%EC%B0%A8%EC%9D%B4%EC%A0%90-1f42c7848771)
