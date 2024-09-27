## Repository 계층과 Service 계층
> 클린 아키텍처에서 Data 계층과 Service 계층을 분리하였다.

사진

- 어제까지의 나는 `Data - Repository`, `Data - Service`와 같이 Data 계층에 같이 존재한다고 생각했다.
    - [멘토님의 피드백](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/120#pullrequestreview-2331094737)
- `Data - Repository`에서는 Service 계층으로부터 받아온 데이터를 관리하는 곳이다.
    - **이때 가져온 데이터가 서버에서 온 거일 수도, DB일 수도 있다.**
- `Service - DataSource`는 데이터와 직접적으로 맞닿게 하여 데이터를 가져오는 역할을 한다.

### 정리
- Repository와 Service를 하나의 레이어로 묶어서 정의하지 말고, Clean Architecture에서는 둘은 분리되는 것이 맞는 것이 맞다.
- Service: 데이터를 가져오는 역할
- Repository: 데이터와 직접 닿기 보다 Service 계층으로부터 오는 내용을 관리해주는 역할
