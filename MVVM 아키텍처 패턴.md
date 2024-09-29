# MVVM Architecture Pattern
> UI를 갖는 응용 프로그램을 위한 아키텍처 패턴 (Model - View - ViewModel)

> [네이버 부스트캠프 멘토님께서 추천한 아티클을 읽고, 스스로 이해한 내용을 정리한 글입니다.](https://gyuwon.github.io/blog/2017/03/05/mvvm-architectural-pattern.html#comment-1)

## 개요
- 뷰의 추상화를 만드는 것이 핵심
    - 뷰의 추상화로인해 재사용 가능하고, 테스트가 쉬워진다.
- 프로그램 구조가 단순해지고 기존 Controller의 막중한 책임감, 역할을 분리하여 독립적으로 구현할 수 있다.
- 기본 iOS UIKit 구조인 `MVC 패턴`의 단점을 개선하고자 나온 패턴이라 생각한다.

## MVC와 다른 점
> Model - View - Controller

- 모델과 뷰는 MVC와 동일하다. 가장 큰 차이는 `Controller`와 `ViewModel`의 역할 차이이다.
- ViewModel은 '뷰의 모형'으로 `추상화된 뷰`라는 의미이다.
    - 시각적인 View를 위해 독립된 공간에서 추상화 작업을 한다.
    - 추상화를 위해서 많이 언급했는데, 예로 .. (여기부터 어려워서 MVVM 프젝 더 경험해보고 마무리 하겠습니다)


## 배운 점
- ViewModel은 View(iOS의 `ViewController`)에서 뷰 그리는 역할만 할 수 있게 연산, API 작업 .. 등을 하는 일이라 생각했는데,
'추상화된 뷰'라는 키워드로 바라보지 않은 편협한 사고를 일깨웠다.