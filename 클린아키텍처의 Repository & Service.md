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

## 24/10/02 추가
## 클린 아키텍처 구조 및 역할
> 클린 아키텍처 적용이 본 프로젝트가 처음입니다. [지난 멘토님 코멘트 링크](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/120#pullrequestreview-2331094737)

> 지난 번에 Repository와 Service를 하나의 레이어로 묶어서 정의해두었다는 멘토님 피드백을 받고 아래와 같이 나누어보았습니다 ! 

<img width="716" alt="스크린샷 2024-10-01 오후 1 49 49" src="https://github.com/user-attachments/assets/32685744-e719-45a4-9284-b4c0f6e54ffc">

[사진 출처: JK 블로그](https://medium.com/@jungkim/%EB%B2%84%ED%84%B0%ED%94%8C%EB%9D%BC%EC%9D%B4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EB%A5%BC-%EC%86%8C%EA%B0%9C%ED%95%A9%EB%8B%88%EB%8B%A4-9d4abd71c3c1)

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

