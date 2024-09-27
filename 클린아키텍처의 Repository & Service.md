## Repository 계층과 Service 계층
> 클린 아키텍처에서 Data 계층과 Service 계층을 분리하였다.

<img width="719" alt="스크린샷 2024-09-27 오후 1 42 52" src="https://github.com/user-attachments/assets/1a3435e3-7521-4ade-b971-a84bbc083822">

- 어제까지의 나는 `Data - Repository`, `Data - Service`와 같이 Data 계층에 같이 존재한다고 생각했다.
    - [멘토님의 피드백](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/120#pullrequestreview-2331094737)
    - 멘토님의 Repository와 Service를 분리하라는 말씀은 이해했지만, 이게 그러면 Data 계층과 Service 계층으로 나누라는 뜻인지 아직 이해가 되지 않는다.
    - TODO: 멘토님께 Data 계층과 Service 계층을 말씀하신 게 맞는지 여쭤보기
- `Data - Repository`에서는 Service 계층으로부터 받아온 데이터를 관리하는 곳이다.
    - **이때 가져온 데이터가 서버에서 온 거일 수도, DB일 수도 있다.**
- `Service - DataSource`는 데이터와 직접적으로 맞닿게 하여 데이터를 가져오는 역할을 한다.

### 정리
- Repository와 Service를 하나의 레이어로 묶어서 정의하지 말고, Clean Architecture에서는 둘은 분리되는 것이 맞는 것이 맞다.
- Service: 데이터를 가져오는 역할
- Repository: 데이터와 직접 닿기 보다 Service 계층으로부터 오는 내용을 관리해주는 역할
