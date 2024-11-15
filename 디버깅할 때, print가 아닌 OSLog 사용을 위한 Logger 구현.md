# 문제 상황

우리가 평소 디버깅을 위해 흔히 사용하는 print는 간단한 출력에는 유용하지만, 다음과 같은 단점이 있다

- **성능**: print는 출력 결과를 처리하는 동안 메인 스레드에서 실행되어 앱 성능에 영향을 줄 수 있다.
- **레벨 구분 없음**: print는 메시지를 구분할 방법이 없어, 로그의 중요도(예: 디버그, 정보, 에러 등)를 명확히 알기 어렵다.
- **릴리스 빌드에서의 노출**: print는 릴리스 빌드에서도 동작하며, 개발 중 남긴 로그가 사용자에게 노출될 위험이 있다.
- **검색 및 필터링 불편**: Xcode의 콘솔에서 print 로그는 별도의 태그 없이 출력되므로 특정 로그를 찾기 어렵다.

본 편에서는 print가 아닌, OSLog를 사용하기 위해 어떻게 Logger를 구현했는지 알아보겠다!

---

# 문제 해결

### 개요

OSLog는 print를 사용해야 할 상황에서 print를 사용하지 않고 디버깅이 가능한 로깅 프레임워크로, 성능 최적화와 레벨별 구분, 필터링 기능을 제공해준다.

또한 릴리스 빌드에서 자동으로 로그를 제한할 수 있어 보안 및 성능 측면에서도 뛰어나다.

### MHLogger 구현

OSLog를 간편하게 사용하기 위해 MHLogger라는 간단한 로거를 구현했다.

MH는 Memorial House의 약자이다 ^ㅇ^

이 로거는 debug, info, error, network와 같은 다양한 로그 수준을 제공하며,

제네릭을 활용해 모든 타입의 데이터를 처리할 수 있도록 설계했다.

```swift
import OSLog

public enum MHLogger 
    public static func debug<T>(_ object: T?) {
        #if DEBUG
        let message = object != nil ? "[Debug] \(object!)" : "[Debug] nil"
        os_log(.debug, "%@", message)
        #endif
    }
   
    public static func info<T>(_ object: T?) {
        #if DEBUG
        let message = object != nil ? "[Info] \(object!)" : "[Info] nil"
        os_log(.info, "%@", message)
        #endif
    }
    
    public static func error<T>(_ object: T?) {
        #if DEBUG
        let message = object != nil ? "[Error] \(object!)" : "[Error] nil"
        os_log(.error, "%@", message)
        #endif
    }
    
    public static func network<T>(_ object: T?) {
        #if DEBUG
        let message = object != nil ? "[Network] \(object!)" : "[Network] nil"
        os_log(.debug, "%@", message)
        #endif
    }
}
```

위처럼 구현하여 #if DEBUG를 사용해 디버그 빌드에서만 로그를 출력하도록 제한한다.

릴리스 빌드에서는 아무 동작도 하지 않으므로, 성능과 보안이 모두 보장된다.

### MHLogger 사용법

MHLogger는 여러 로그 레벨을 제공하며, 이를 통해 로그의 중요도를 구분할 수 있다.

사용법은 간단한데, MHLogger는 제네릭으로 파라미터를 받기 때문에 String뿐만 아니라 Int, Bool, 사용자 정의 객체 등 모든 타입을 로깅할 수 있어 아래와 같이 사용될 수 있다!

<img width="340" alt="스크린샷 2024-11-16 오전 4 22 48" src="https://github.com/user-attachments/assets/bfde9c9f-579f-43b9-b9e6-ca11445fea16">


print를 지양합시다 〰️ 

---

# 배운 점

디버깅에서 단순한 print 사용은 제한적이고 비효율적이다.

OSLog를 활용한 MHLogger는 성능, 보안, 가독성 측면에서 print보다 훨배 좋음

또, 프로젝트의 디버깅 및 로그 관리를 체계적으로 수행할 수 있게 하여 개발 과정뿐만 아니라 유지 보수 및 확장성에서도 유용할 지도 ?

→ macOS 기본 앱인 콘솔과 함께 로깅이 가능하기 때문 !
