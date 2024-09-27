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
