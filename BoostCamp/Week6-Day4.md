## 오늘 한 일
- Coordinator 순환 참조 문제 해결
- 테이블뷰 셀 스와이프 액션 -> 이슈 닫기 구현
- 무한스크롤 페이지네이션 구현

## Coordinator 순환 참조 문제 해결
<img width="1076" alt="스크린샷 2024-10-03 오전 3 41 28" src="https://github.com/user-attachments/assets/5ad44ebe-7622-4171-9727-c9aa3c450b41">
- 뷰컨트롤러는 코디네이터를 처리할 수 있는 델리게이트를 갖는데, `weak`를 모두 붙였음에도 위와 같이 순환참조 문제가 발생했다.
- 코디네이터마다 `finishDelegate`를 프로퍼티로 갖고 있는데, `weak` 설정을 안 해줘서 문제가 발생했던 것이다.
- weak를 잘 쓰자.

<br>

## 테이블뷰 셀 스와이프 액션 -> 이슈 닫기 구현
- UITableViewDelegate 프로토콜 내부에 다음과 같은 메소드가 있다.
    ``` swift
    func tableView(
        _ tableView: UITableView,
        trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath
    ) -> UISwipeActionsConfiguration? { }
    ```
    - trailing으로 하면 오른쪽 -> 왼쪽
    - leading으로 하면 왼쪽 -> 오른쪽

- 내부 리턴 값으로 UISwipeActionsConfiguration을 갖는다.
    - Foundation 프레임워크 안에 있는 클래스로, NSObject만을 상속받는다.
    - `performsFirstActionWithFullSwipe` 프로퍼티를 통해 스와이프를 많이 했을 때, 바로 액션을 하게 할 수 있다.
        - 참고로 디폴트는 true이다.
    - 생성할 때 `convenience init`에 의해 `UIContextualAction`들을 넣어줘야 하는데, style에 `destructive`와 `normal`이 있다. (Alert와 비슷한듯)

<br>

## 무한스크롤 페이지네이션 구현
- 본 프로젝트에서는 깃허브 API로부터 `Issue`와 `Assignee`를 받아온다.
    - 페이지와 페이지 당 개수를 설정할 수 있는데, 디폴트는 한 페이지와 30개이다.
    - 나는 개수는 상관없고, 스크롤이 다 되어갈 때 즈음에 다음 페이지를 추가 요청할 예정이다.

- Request 요청을 할 URL에 쿼리를 넣어준다
    ``` swift
    var queryParameters: [String: Any]? {
        switch self {
        case .fetchIssues(let page), .fetchAssignees(let page):
            return ["page": page]
        case .createIssue, .closeIssue:
            return nil
        }
    }
    ```
    - page는 Int 값이나, 나중에 makeURL() 메소드에서 String으로 감싸서 쿼리를 생성한다.
    - 그러면 다음과 같은 URL이 완성된다.
    - `https://api.github.com/repos/../../issues?page=1` 혹은 `https://api.github.com/repos/../../assignees?page=1`

- API 요청 작업을 완료했다면, 이제 TableView에서 처리할 단계이다.
    ``` swift
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {}
    ```
    - 우리가 많이 본 `UITableViewDatasource`를 채택하면 반드시 구현해야 하는 메소드이다.
    - 각 행에 셀을 만드는 메소드로, 100개의 셀이 보여져야 한다면 100개가 다 보이는게 아니라 한 화면에 들어갈 수 있는 셀만큼만 생성하고 나머지는 스크롤 됐을 때 Row에 맞게끔 IndexPath로 셀을 생성한다.
    (말이 좀 이상한데?)
    - 이 메소드에서 다음과 같이 코드를 작성한다.
        ``` swift
        if indexPath.row == viewModel.issues.count - 3 {
            viewModel.fetchMoreIssues()
        }
        ```
    - viewModel의 `issues`를 관측하고 있는 상황이고, issues.count는 기본적으로 30개이다.
    - 그럼 내가 27번째 셀을 만들 때 `viewModel.fetchMoreIssues()`이 동작한다.
    ``` swift
    final class IssueListViewModel {
        @Published var issues: [IssueEntity] = []
        private var cancellables: Set<AnyCancellable> = []
        private var issueUseCase: IssueUseCase
        private var page: Int = 1
        
        ...

        func fetchMoreIssues() {
            defer { page += 1 }
            issueUseCase.fetchIssues(page: page)
                .receive(on: DispatchQueue.global())
                .catch { _ in Just([]) }
                .sink { [weak self] issues in
                    self?.issues.append(contentsOf: issues)
                }
                .store(in: &cancellables)
        }
    }
    ```
    - 위는 fetchMoreIssues() 동작이다. 프로퍼티 page에 맞는 이슈들을 요청하고, 작업이 다 끝나면 page를 `defer`에 의해 1 증가시킨다.
    - 그리고 현재 `issues` 프로퍼티에 새로 받은 issues를 추가해주면 화면에서 다음 페이지를 잘 보여줄 수 있게 된다.


