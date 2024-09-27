# 피어세션에서 한 내용 정리
> Clean Architecture + MVVM + Coordinator + DIContainer에 대한 내용 정리

> 계층 별 네이밍은 다음과 같습니다.

<img width="1401" alt="스크린샷 2024-09-26 오후 4 00 16" src="https://github.com/user-attachments/assets/c9d7c7ce-cf6a-44ae-8e68-189084b445dc">

## 계층 별로 struct와 class
- 우리가 평소에 고민하던 내용과 같다. (참조, 상속 ..)
- 용범님: `DataSource`(= 나의 Service) 계층에서 BaseDataSource를 갖고 있기 때문에 상속을 위해 class로 구현했고, 나머지는 참조 방식이 필요없어서 struct로 구현

<br>

- [Network 계층 추상화](https://github.com/Kyxxn/TIL/blob/main/Network%20%EA%B3%84%EC%B8%B5%20%EC%B6%94%EC%83%81%ED%99%94.md)
- [클린 아키텍처에서 Repository와 Service](https://github.com/Kyxxn/TIL/blob/main/%ED%81%B4%EB%A6%B0%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EC%9D%98%20Repository%20%26%20Service.md)
    - TODO: 멘토님께 Data 계층과 Service 계층을 말씀하신 게 맞는지 여쭤보기
- [DIContainer 적용 이유](https://github.com/Kyxxn/TIL/blob/main/DIContainer%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%20%EC%9D%B4%EC%9C%A0.md)
- [Coordinator Design Pattern](https://github.com/Kyxxn/TIL/blob/main/Coordinator%20Design%20Pattern.md)
- [프레임워크 추가로 모듈화 구현하기](https://github.com/Kyxxn/TIL/blob/main/Framework%20%EC%B6%94%EA%B0%80%EB%A1%9C%20%EB%AA%A8%EB%93%88%ED%99%94%20%EA%B5%AC%ED%98%84.md)
    - 앞전에 창우님이 설명해주신 것과 유사하나, 그 때와 달리 `New Project`를 사용
- [클로저에서 weak self](https://github.com/Kyxxn/TIL/blob/main/%ED%81%B4%EB%A1%9C%EC%A0%80%20%EA%B0%92%20%EC%BA%A1%EC%B2%98%EC%99%80%20weak%20self.md)

<br>

## 레퍼런스
- [Static Library 정리](https://medium.com/delightroom/ios-library-framework-swift-package-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-%EC%B0%A8%EC%9D%B4%EC%A0%90-1f42c7848771)
