## 모듈화 과정
> Dynamic Framework를 사용했다.

1. New - project 생성 후 그 안에 파일을 타겟 설정해서 채워넣고 빌드를 한다.
2. 위 프로젝트를 종료하고, 위 디렉토리 전체를 IssueTracker 안에 넣는다.
3. xcworkspace를 켜서 맨 바깥에 .xcodeproj에 대한 Reference만 추가

<br>

## 하다가 알게된 TMI
1. Foundation에 Combine은 내가 아는 Combine과 다름
    - sink와 같은 것들은 import Foundation만 해서는 안됨
2. 파일을 꼭 하나 추가하고 할 것. (Empty 파일을 만들면 타겟 추가가 안됨)
    - Storage 계층 만들다가 알게된 건데, 위 모듈화 과정에서 1번 동작을 꼭 해줘야 `import Storage`가 가능함
3. Network는 이미 존재하는 프레임워크이다.
    - 네트워크 프레임워크 이름을 `Network`로 했음.. (여기서 3시간 길바닥에 버림)
    - `NetworkService`로 수정
4. Bundle 주입 문제
    - 오강훈 멘토님께서 Bundle에서 토큰 주입이나 URL 주입 등등을 분리해야할 필요가 있다고 말씀하심
    - 무슨 뜻인지 몰랐으나, `Storage`계층에서 `Resource - issues.json`에 있는 JSON을 Bundle로 막 가져다 썼는데, 모듈화를 하니까 이게 안됨
    - 의존성 주입이 필요했음

<br>

### Reference
- [Framework 모듈화 작업](https://daddy73e.tistory.com/4)
