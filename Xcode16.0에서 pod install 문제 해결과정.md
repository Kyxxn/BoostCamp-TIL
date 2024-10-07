## Xcode16.0에서 pod install 안되는 이슈
> Xcode 16.0 업데이트 이후, CocoaPods로 install하면 에러가 발생하는 이슈가 있었다.

문제의 파일: `.xcodeproj/project.pbxproj`
[관련 코코아팟 이슈](https://github.com/CocoaPods/CocoaPods/issues/12583)

### 문제 해결 과정
문제의 파일은 해당 경로이다.

1. `nano 프젝명.xcodeproj/project.pbxproj`을 통해 CLI에서 편집기로 열어준다.
2. `objectVersion = 77;` -> `objectVersion = 56;`으로 변경
3. 아래 두 파일 제거
    - minimizedProjectReferenceProxies = 1;
    - preferredProjectObjectVersion = 77;

`.pbxproj`에서 단어 검색이 어려우면 `nano`에서 `ctrl + W`로 단어 찾으면 된다.

다시 `pod install`하면 정상 작동함!