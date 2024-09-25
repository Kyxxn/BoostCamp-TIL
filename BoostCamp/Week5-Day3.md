## 오늘 내가 시도해본 것들
- Clean Architecture 아키텍처에 Combine을 적용해서 반응형으로 개발을 하려하고 있다.
    1. 뷰가 변경되면 ViewModel이 UseCase를 호출하고, UseCase는 Repository를 호출한다.
    2. Repository가 API 요청을 하여 issues를 받아내면, 그 결과가 성공했을 경우 AnyPublisher<[IssueEntity], Never>의 첫 번째 IssueEntity로 들어가고, 실패한 경우 Never로 처리 된다.
    3. UseCase는 동일하게 Repository의 리턴값 AnyPublisher<[IssueEntity], Never>을 반환하여 ViewModel은 이 결과를 통해 사용하게 된다.
    4. ViewModel이 사용할 때는 .assign을 통해 어떤 클래스의 프로퍼티를 등록할 지를 정한다. 또한, ViewModel이 사라졌을 때 메모리 누수 방지를 위해 store에도 저장한다.
    5. ViewController는 ViewModel의 $issues를 구독하고 있다가, sink 즉 변화가 생기면 해당 변화된 정보를 얻어서 로직을 처리한다. 컨트롤러 또한 store를 사용하여 메모리 누수를 막는다.

- 왜 Combine을 사용했는가 ?
    - Combine, Concurrency, RxSwift, Delegate, NotificationCenter, Closure.. 등등
    지금껏 클로저를 통해서 바인딩해왔는데, 기능이 추가될 수록 가독성이 안 좋아지는 것 같았다.
    - 그래서 Pub-Sub 패턴을 적용하고 싶었는데, 반응형 프로그래밍을 해보고자 채택했다.
    - 무엇보다도 애플 제공 프레임워크라서 이기도 하다.

- URLSession으로 JSON 받아서 디코딩하기
    - 나의 아키텍처에서 Repository만 있고 Service가 없다.
    (영규님 코드를 오늘 많이 봤는데 영규님 기준 DataSource입니다.)
    - 내가 원하는 건 DTO를 통해서 JSON을 받고, Domain Model (= Entity)에 맞게 UseCase에서 사용하는 것인데,
    지금 구조는 처음부터 DTO가 아닌 Entity로 받고 있다.
    - Repository에서 DTO로 받고, DTO 구조체에 `func toEntity() -> OOOEntity`로 구현하려는데
    DTO로 디코딩을하면 계속 `keyNotFound`가 떴다. JSON 키 매칭이 안되는 것, [어제 공부한 내용이다..](https://github.com/Kyxxn/TIL/blob/main/CodingKey.md)

## 배운 점
- Combine
    - Pub-Sub 패턴을 그나마 잘 익히고 있어서 관련된 Operator 메소드만 어떤 역할을 하는지 잘 알면
    적용하는데에는 큰 문제가 없었다. (근데 Operator가 너무 많음)
    - Contoller와 ViewModel만 Combine으로 연결하나? 라고 생각했는데 `UseCase, Repository` 모두 다 하는 걸 알게 됐다.

- 디코딩
    - 디코딩 또한 거의 처음이다.. 지난 챌린지나 멤버십의 인코딩&디코딩 하는 미션들도 100% 구현을 못 해서 많이 겪어보지 않았고, 구현하더라도 정말 간단한 내용이라 CodingKey까지 쓸 것도 없었는데 이번에 삽질하면서 많이 익혔다.
    - `decoder.dateDecodingStrategy`: iso8601 로 설정하면 날짜 문자열을 Data로 받아올 수가 있다..!
    - `decoder.keyDecodingStrategy`: convertFromSnakeCase을 설정하면 JSON의 스네이크 케이스를 해석할 수 있다고 하나, Codable의 CodingKey를 사용하면 될 것 같다.


## 학습 내용
- 컴바인은 로우하게 시리즈로 작성해야 할 것 같아 주말에 적겠습니다.


## 참조링크
- [Zedd Combine](https://zeddios.tistory.com/1003)
- [Combine 정리 (1)](https://origogi.github.io/ios/swift-combine-1/)
- [Combine 정리 (2)](https://origogi.github.io/ios/swift-combine-2/) // 일단 여기까지 봄
- [Combine 정리 (3)](https://origogi.github.io/ios/swift-combine-3/)
- [Combine 정리 (4)](https://origogi.github.io/ios/swift-combine-4/)
- [Combine 정리 (5)](https://origogi.github.io/ios/swift-combine-5/)

