# 피어세션에서 한 내용 정리
> Clean Architecture + MVVM + Coordinator + DIContainer에 대한 내용 정리

> 계층 별 네이밍은 다음과 같습니다.

<img width="1401" alt="스크린샷 2024-09-26 오후 4 00 16" src="https://github.com/user-attachments/assets/c9d7c7ce-cf6a-44ae-8e68-189084b445dc">

## 계층 별로 struct와 class
- 우리가 평소에 고민하던 내용과 같다. (참조, 상속 ..)
- 용범님: `DataSource`(= 나의 Service) 계층에서 BaseDataSource를 갖고 있기 때문에 상속을 위해 class로 구현했고, 나머지는 참조 방식이 필요없어서 struct로 구현

<br>

- Network 계층 추상화
- Repository와 Service 계층
- DIContainer 적용 이유
- Coordinator Design Pattern
- 프레임워크 추가로 모듈화 구현하기
    - 앞전에 창우님이 설명해주신 것과 유사하나, 그 때와 달리 `New Project`를 사용
- 클로저에서 weak self

<br>

## 레퍼런스
- [Static Library 정리](https://medium.com/delightroom/ios-library-framework-swift-package-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-%EC%B0%A8%EC%9D%B4%EC%A0%90-1f42c7848771)
