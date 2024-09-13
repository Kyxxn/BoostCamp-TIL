## 학습 목표
- 추상화를 하는 단위, 어디까지 할 것인가 ?
- 프로토콜에서 { get }, { get set } 의미를 잘 알고 쓰기
- Squash Merge를 해야할 때를 알기

---

# 학습 내용
## 추상화를 하는 단위, 어디까지 할 것인가 ?
현재 내 코드는 `Shapable` 프로토콜이 Shape가 갖는 모든 속성을 다 갖고 있다.

``` swift
protocol Shapable: AnyObject {
    var identifier: String { get }
    var serialNumber: Int { get }
    var origin: Point { get set }
    var size: Size { get set }
    var color: RGBColor { get set }
    var alpha: AlphaType { get set }
    var type: ShapeCategory { get }
    
    func updateColor(color: RGBColor)
    func updateAlpha(alpha: AlphaType)
    func updateOrigin(x: Double, y: Double)
    func updateSize(width: Double, height: Double)
    func postShapeUpdatedNotification()
}
```

이런 구조로 작성되면 뭐가 문제일까 ?

가장 큰 문제는 `Shapable` 프로토콜을 채택하는 구현체에서 사용하지 않는 프로퍼티까지 구현해야 할 수 있다는 점이다.

본 미션에서 `사진` 버튼을 클릭하면 만들어지는 `Photo` 모델의 경우, 위 프로토콜에서 `color`를 사용하지 않는다.

이 상황이 딱 내가 말한 문제에 봉착하는 것이라 생각한다.

**`객체지향의 ISP를 위배하는 코드라 생각한다.`**

지금의 경우야 코드가 겹치는 프로퍼티가 많다. 그렇기 때문에 메모리적으로도 큰 문제가 없는데, 이런 상황이 잦아지면 `성능 낭비 & 가독성 문제`가 커질 수 있다고 생각한다.

### 그러나,

위 사진과 같이 하나의 `Shapable`만 사용한다면, 타입캐스팅을 하지 않고도 구현할 수 있다는 장점이 있었다.

뭐가 좋은 구조일까? `compoision`이라는 성질을 이용해서 잘게 프로토콜을 쪼개고 사용하는게 좋을까? 아니면 `Shapable` 하나에 다 때려박는게 좋을까 ?

결론은 잘게 쪼개는 것이라 생각한다.

- 객체지향의 ISP를 지킬 수 있고,
- 불필요한 프로퍼티/메소드를 갖지 않아도 되어 코드의 복잡성을 줄일 수 있고 유지보수를 늘릴 수 있다.
- 가독성 및 메모리 낭비도 줄일 수 있다고 생각한다.
    - 상속 관계까지 가게 된다면, 자식이 사용하지 않는 프로퍼티더라도 부모의 프로퍼티를 참조하고 있으므로 Dynamic Dispatch가 동작한다고 생각한다 (뇌피셜)

### 개선해보기
``` swift
protocol Shapable {
    var identifier: String { get }
    var serialNumber: Int { get }
    var type: ShapeCategory { get }

    func postShapeUpdatedNotification()
}

protocol Locatable {
    var origin: Point { get set }
    func updateOrigin(x: Double, y: Double)
}

protocol Sizable {
    var size: Size { get set }
    func updateSize(width: Double, height: Double)
}

protocol Colorable {
    var color: RGBColor { get set }
    func updateColor(color: RGBColor)
}

protocol Transparent {
    var alpha: AlphaType { get set }
    func updateAlpha(alpha: AlphaType)
}

protocol Shapable: Shapable, Locatable, Sizable, Colorable, Transparent {}
```
이렇게 하므로써 ISP를 준수하여 코드 복잡성을 줄일 수 있다고 생각한다.

<br>

## 프로토콜에서 { get }, { get set } 의미를 잘 알고 쓰기

기존의 내 코드에서 { get set } 을 잘 생각하지 않고 사용하고 있었다.

**아래 코드에서 문제점이 보이는가 ?**

``` swift
protocol Shapable: AnyObject {
    var identifier: String { get }
    var serialNumber: Int { get }
    var origin: Point { get set }
    var size: Size { get set }
    var color: RGBColor { get set }
    var alpha: AlphaType { get set }
    var type: ShapeCategory { get }
    
    func updateColor(color: RGBColor)
    func updateAlpha(alpha: AlphaType)
    func postShapeUpdatedNotification()
}

extension Shapable {
    func updateColor(color: RGBColor) {
        self.color = color
    }
    
    func updateAlpha(alpha: AlphaType) {
        self.alpha = alpha
    }
    
    func updateOrigin(x: Double, y: Double) {
        self.origin = Point(x: x, y: y)
    }
    
    func updateSize(width: Double, height: Double) {
        self.size = Size(width: width, height: height)
    }
}
```

{ get set }을 하면 구현체에서 internal과 같이 사용된다.
즉, 만들어진 인스턴스가 프로퍼티에 접근할 수 있다는 뜻이다.

``` swift
var shape: Shapable = Rectangle()
shape.size = Size(width: 0, height: 0)
```
이게 가능하다는 뜻,

그럼 `func updateSize(width: Double, height: Double)`는 왜 필요할까 ?

내 말이다..

`extension Shapable`에 확장으로 걸어줄 게 아니라, 이걸 사용하는 구현체에서 메소드를 지정해주고 프로토콜은 { get }으로 해줘야 프로퍼티의 접근 제어자를 지킬 수 있다.

그럼 아래와 같은 코드처럼 프로퍼티 접근 제어자를 잘 지정해줄 수 있다.

``` swift
var shape: Shapable = Rectangle()
shape.size = Size(width: 0, height: 0) // 에러
if let shape = shape as? Rectangle { 
    shape.updateSize(width: 0, height: 0)
}
```

타입 캐스팅을 통해 Rectangle에 있는 updateSize를 사용하면 된다!

<br>

## Squash Merge 사용하는 시점

부스트캠프 멘토께서 캠퍼에게 아래와 같은 말씀을 하셨다.

"오늘은 squash 머지를 할 것인데요, squash머지를 하는 이유와 생길수 있는 문제에 대해서도 한번 보시면 좋을 것 같아요"

Squash Merge: 파생된 브랜치에서 한 커밋 히스토리들을 하나로 합쳐서 머지하는 것

즉, Merge된 브랜치에서는 파생된 브랜치에서 한 세부 작업들에 대한 커밋단위의 리뷰가 불가능하다.

합쳐진 커밋 중에서 특정 시점으로 되돌아가는 것도 어려울 것 같다.

그러나, 불필요하게 길게 만들어져있는 커밋 히스토리를 볼 필요가 없다고 판단되는 경우 Squash 머지가 좋다고 생각한다.

아래와 같이 브랜치 간의 그림을 그려 생각해봤다.

![스크린샷 2024-09-14 오전 2 11 09](https://github.com/user-attachments/assets/7845e697-78ba-40e9-9daa-6f165460c2c3)


`develop` 브랜치 입장에서 `feature-Model`과 `feature-Componenet`, `feature-View`에서 발생한 커밋 히스토리를 알 필요가 있을까 ?

즉, `feature`가 `develop`에 Merge할 때는 기존과 동일하게 `3-way Merge`를 사용하지만, `feature`로부터 파생된 브랜치들이 `feature`에 머지될 때는 Squash Merge가 좋을 거 같다고 생각했다.

---

## 배운 점
- 추상화에 대한 깊은 고민을 할 수 있게 됨
- 프로토콜에서 { get set } 의 역할이 언제 빛을 발하는지 알게 됨
    - 프로토콜 타입으로 선언됐을 때 { get }만 있는 경우 프로퍼티 set이 불가능
- Squash Merge 동작을 복습했고, 언제 사용해야 할지와 발생할 수 있는 문제를 생각해보았다.

---

## 참조 링크
- 부스트캠프 동료들 견해

