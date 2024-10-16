## 오늘 모듈화된 환경에서 테스트를 진행해봤다.
### 핵심 키워드
- setUp -> 테스트 메소드 -> tearDown 순서
- 모듈화에서 테스트 진행하기 (Mock 어디에 둘 지 고민)
- 프로토콜로 맵핑해줘야 하는 이유 (테스트할 때 Mock 쓰려고, createLabelForm, updateLabelForm)
- 비동기일 때 테스트

<br>

## Presentation의 ViewModel 테스트 할 때 MockUseCase는 어디에 둬야 할까 ?
> 우리는 작일 모듈화를 진행하여 Clean Architecture 별로 모듈을 분리하였다.

### Presentation의 ViewModel을 테스트 하며 느낀점
- 테스트는 Presentation 모듈에 두고 진행해야 함
- 테스트에 필요한 Mock UseCase 객체는 Domain에 두어야 한다.
    - 왜 ? -> UseCase 프로토콜이 Domain에 있기 때문.
    - 프로토콜이 왜 필요한데? -> 프로토콜을 통해서 구현체는 냅두고 Mock을 만들기 때문

<br>

## 비동기 테스트 진행할 때 주의할 점
- Combine에 대한 비동기 테스트를 진행했는데, sink에서 recieve되기 전에 종료돼 버렸다.
	- sleep 쓰면 안되긴 했음
- `XCTestExpectation`과 해당 인스턴스의 `fulfill()` 메소드를 통해 wait(for:, timeout)에 시간 텀을 설정해줄 수 있었음
