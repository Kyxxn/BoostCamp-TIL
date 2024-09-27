## 프레임워크 추가로 모듈화 구현하기

<img width="500" alt="스크린샷 2024-09-27 오후 1 21 29" src="https://github.com/user-attachments/assets/4cb811b2-5dce-4290-a66d-bb2632d68b7c">

- `New Project` - `Framework & Library`에서 `Framework`를 선택한다.
    - 참고로 Static Library는 리소스 파일들(image, assets, nibs, strings file)을 포함할수가 없음 [레퍼런스 링크](https://medium.com/delightroom/ios-library-framework-swift-package-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-%EC%B0%A8%EC%9D%B4%EC%A0%90-1f42c7848771)
- Framework를 생성했다면, 내부에 Temp.swift 파일을 하나 만든다.
- 기존 프로젝트에서 `Add Files to '현재프로젝트'`를 하면 해당 프레임워크를 모듈로 추가해준다.
- 최상위 프로젝트(= 모듈)에서 Targets에 의존성을 줄 수 있다.
