## 오늘 시도해본 것들
- 오늘 학교 일정이 많이 잡히고, 집오니 집중이 깨져서 조금 여유부렸습니다..
- 뒤늦게 일어나서 Coordinator 패턴을 공부하고 제 프로젝트에 어떻게 적용할 지 생각해봤습니다 !

## Coordinator 도입하기
> 지난 주에 코디네이터 개념을 공부했었습니다. 이걸 바탕으로 제 프로젝트에 적용해보고 개념을 실습해보려 합니다! [개념정리 링크](https://github.com/Kyxxn/TIL/blob/main/Coordinator%20Design%20Pattern.md)

1. 내 앱은 Issue List 위에서 다 동작한다.
    - 처음 시작되면 `Issue List`가 보이고,
    - 우측 상단 + 를 누르면 Present 방식으로 `Issue Creator`를 띄우고,
    - 담당자를 클릭하면 navi.pushVC 방식으로 `Assignee Registation`을 띄운다.
 `AppCoordinator`를 제외하면, 그 바로 아래 `IssueCoordinator`가 작업을 다 해줘도 될 것 같았다.
2. 그래서 제가 이해한 코디네이터로 제 프로젝트 내의 뷰컨트롤러 흐름을 다음과 같이 그려보았습니다.

### 여기서 문제
- 코디네이터는 구현하기 나름이라고 하지만, 그래도 처음하는 입장에서 확장성이니 의존성을 고려할 필요는 있다고 생각합니다.
- 다른 분들 중에 코디네이터를 하신 분들 내용을 많이 참고했습니다.
    - 팀원이신 혜민님은 아래 사진처럼 각각의 코디네이터를 지정해줬고,
    해당 코디네이터가 이동할 화면에 대한 메소드를 처리할 때, 그 코디네이터 인스턴스를 만들고 start로 요청하셨다. [코드 링크](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/blob/S076/IssueTrackerApp/IssueTrackerApp/Source/Presenter/App/Coordinator/IssueListCoordinator.swift)
    - S068 다경님은 아래와 같이 (나와 같음) 구상을 하셨다.
    그리고 하나의 코디네이터에서 메소드 별로 띄워줄 화면의 코디네이터가 아닌, 뷰컨트롤러를 생성해서 present하고 계셨다. [코드 링크](IssueManager/IssueManager/Coordinators/IssueCoordinator.swift)

지금와서 보니 제 방식은 OCP 원칙을 위배하고 있다고 생각합니다.
코디네이터에서 띄워줄 화면의 VC를 만들지 않고, 코디네이터를 만들어서 해당 코디네이터의 start() 메소드를 실행하는 것이 옳아 보이네요..
정리하다보니 답이 떠오른 것 같습니다. 내일 혜민님한테 여쭤보며 제가 이해한 내용이 맞는지 확인해볼 것 같습니다!