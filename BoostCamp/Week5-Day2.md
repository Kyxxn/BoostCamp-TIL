## 오늘 내가 시도해본 것들
- MVVM을 위해 Combine에 대한 개념을 공부하려 시도했다 (진짜 시도만 함)
    - Combine에 대한 개념을 적용해보고자 어디부터 어디까지 적용해야할 지 고민해보았다.
    - ViewController - ViewModel만 하는 건지, UseCase Repository까지 다 하는건지.. 궁금하다.
- Clean Architecture를 적용하였고, 이는 잘 반영되고 있다고 생각한다.
    - 단, 여기서 `ViewModel - UseCase - Repository`를 class로 구현해야 할 이유를 아직 잘 모르겠다.
    - 다른 고수분들(캠퍼)의 코드를 보니 final class로 했던데 이유가 궁금해졌다..
- Moya를 사용하지 않고 Data - Network 작업을 어떻게 할 지 고민해보았다.
    - URLSession 작업까지는 하지 않았지만, HTTPMethods를 열거형으로 관리했다.
    - Moya와 유사하게 TargetType과 같은 baseURL, path, header.. 를 관리하는 프로토콜을 생성했다.

## 배운 점
- Clean Architecture와 관련한 관심사 분리를 위한 목적을 다시금 되뇌이었다.
    - 그 과정에서 DTO와 Entity의 역할도 잘 나누어 모델링할 수 있었다.
    - UseCase에서 DIP를 통해 RepositoryProtocol로부터 결합도를 낮출 수 있었다. (어차피 RepositoryProtocol을 채택하면 똑같은 메소드를 UseCase가 호출할테니까)
- CodingKeys의 역할을 깨달았다. 아래에 기술하겠다.
    

## 학습 내용
- [CodingKey 내용 정리](https://github.com/Kyxxn/TIL/blob/main/CodingKey.md)
    - CodingKey가 뭔지
    - CodingKey가 왜 동작하는지


## 참조링크
- [CodingKey에 대해](https://junbok97.tistory.com/140)
