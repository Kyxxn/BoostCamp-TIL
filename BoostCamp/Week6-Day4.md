## 오늘 한 일
- Coordinator 순환 참조 문제 해결
- 테이블뷰 셀 스와이프 액션 -> 이슈 닫기 구현
- 무한스크롤 페이지네이션 구현

- **Coordinator 순환 참조 문제 해결**
    - [링크 바로가기](https://github.com/Kyxxn/TIL/blob/main/Coordinator%20Design%20Pattern.md#10%EC%9B%94-4%EC%9D%BC-%EC%B6%94%EA%B0%80-appcoordinator%EA%B0%80-%EC%82%AC%EB%9D%BC%EC%A0%B8%EC%84%9C-main-%ED%99%94%EB%A9%B4-issue-list%EA%B0%80-%EC%82%AC%EB%9D%BC%EC%A7%80%EB%8A%94-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0)
    - 뷰컨트롤러는 코디네이터를 처리할 수 있는 델리게이트를 갖는데, `weak`를 모두 붙였음에도 위와 같이 순환참조 문제가 발생했다.
    - 코디네이터마다 `finishDelegate`를 프로퍼티로 갖고 있는데, `weak` 설정을 안 해줘서 문제가 발생했던 것이다.
    - weak를 잘 쓰자.

- **테이블뷰 셀 스와이프 액션 -> 이슈 닫기 구현**
    - [링크 바로가기](https://github.com/Kyxxn/TIL/blob/main/%ED%85%8C%EC%9D%B4%EB%B8%94%EB%B7%B0%20%EC%85%80%20%EC%8A%A4%EC%99%80%EC%9D%B4%ED%94%84%20%EC%95%A1%EC%85%98.md)
    - UITableViewDelegate 메소드 공부
    - UISwipeActionsConfiguration 공부

- **무한스크롤 페이지네이션 구현**
    - [링크 바로가기](https://github.com/Kyxxn/TIL/blob/main/%EB%AC%B4%ED%95%9C%EC%8A%A4%ED%81%AC%EB%A1%A4%20%ED%8E%98%EC%9D%B4%EC%A7%80%EB%84%A4%EC%9D%B4%EC%85%98%20%EA%B5%AC%ED%98%84.md)
    - 1. 쿼리 파라미터 추가하기
    - 2. TableView에서 무한 스크롤 처리하기
    - 3. ViewModel에서 다음 페이지 데이터 요청하기
