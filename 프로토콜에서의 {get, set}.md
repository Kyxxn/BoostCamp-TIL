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
