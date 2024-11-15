# 문제 상황

홈 화면에서 컬렉션 뷰를 통해 책들을 Flow하게 보여준다.

<img src="https://github.com/user-attachments/assets/3f678b83-ff1b-41f7-a2e1-fefeb14ad638">

그리고 책을 생성하기 위해서는 우측 하단의 Floating 버튼이 있는데,

이 버튼이 마지막 행의 오른쪽 책을 가리기 때문에 유저에서 ux적으로 불편함을 줄 수 있을 것이라 생각했다.

이 문제를 해결해 나간 과정을 서술해보겠다.

---

# 문제 해결

### 아이디어

플로팅 버튼은 우측 하단에 계속 띄워진 채 존재한다.

마지막 행의 오른쪽 셀이 문제기 때문에 팀원과 논의한 끝에 ‘컬렉션뷰의 맨 아래에 도달하면 버튼을 숨기면 좋겠다’고 결론이 났다.

그래서 버튼을 깜빡거리게 isHidden과 같은 요소를 사용할까? 했지만,

유저에게 재미를 주기 위해 컬렉션 뷰 아래 바닥에 닿으면 아래로 슝 사라지고, 위로 스크롤하면 슝 올라오게 하려고 했다.

(슝 이란 말로밖에 형용할 수가 없네요..ㅋㅋ)

---

## 해결 과정

### 컬렉션 뷰 맨 아래에 도달을 감지

컬렉션 뷰 맨 아래에 도달했다는 것을 감지하기 위해 2가지 방법을 떠올렸다.

- `func scrollViewDidScroll(_ scrollView:)` 메소드 사용
- 컬렉션 뷰 델리게이트의 `func collectionView(_ collectionView:, willDisplay cell:, forItemAt indexPath:)` 메소드를 사용하여 마지막 아이템을 감지하기

→ 2번째 방법보다는 1번 방법이 나을 것이라 생각했다.

CollectionViewDelegate는 ScrollViewDelegate를 채택한다.

`@MainActor **public** **protocol** UICollectionViewDelegate : UIScrollViewDelegate {}` 

`UIScrollViewDelegate` 에는 scrollViewDidScroll이라는 메소드가 있는데, 스크롤이 될 때 감지를 하는 메소드이다.

이 메소드에서 **현재 스크롤 위치가 전체 콘텐츠 높이에서 화면 높이를 뺀 위치보다 아래에 있는지**를 검증한다.

나는 해당 델리게이트 메소드를 다음과 같이 작성했다.

```swift
public func scrollViewDidScroll(_ scrollView: UIScrollView) {
    let offsetY = scrollView.contentOffset.y
    let contentHeight = scrollView.contentSize.height
    let height = scrollView.frame.size.height

    offsetY > contentHeight - height
    ? hideFloatingButton()
    : showFloatingButton()
}
```

수식으로 표현하면 다음과 같다.

현재 스크롤 위치 > 전체 콘텐츠 높이 - 화면 높이

현재 스크롤 위치가 더 크면 맨 아래에 도달했다는 뜻이므로 버튼을 숨기고,

작으면 아직 맨 아래에 도달하지 않았다는 뜻이므로 버튼을 띄운다.

### 플로팅 버튼의 Constraint를 조절하여 애니메이션 구현하기

이제 현재 화면이 컬렉션 뷰 맨 아래 도달은 감지할 수 있게 됐다.

그러면 어떻게 플로팅 버튼을 숨기고 늘리는 애니메이션을 만들 수 있을까 ?

내가 구현한 방식은 런타임 도중에 플로팅 버튼의 Bottom Anchor constraint 값을 바꾸는 것이다.

맨 마지막 하단에 닿았을 때 Constraint를 증가시켜서 레이아웃을 더 아래로 위치하게 하고, 다시 위로 스크롤하면 Constraint를 감소시켜 화면에 보이게 한다.

그리고 이 과정을 UIView.animate(withDuration:) 메소드를 통해 딜레이를 주며 위치를 이동시키면 원하는 결과를 얻을 수 있다.

## 구현 과정

### 1. 프로퍼티 추가

위 구현을 하기 위해 두 가지 프로퍼티를 추가한다.

- floatingButtonBottomConstraint: NSLayoutConstraint = Bottom Anchor의 Constraint를 관리
- isFloatingButtonHidden = 플로팅 버튼이 숨겨져있는 상태인지

### 2. Bottom Anchor 보관하기

```swift
public final class HomeViewController: UIViewController {
    private var floatingButtonBottomConstraint: NSLayoutConstraint?
    private let makingBookFloatingButton = UIButton(type: .custom)
    ...
    
    func configureConstraints() {
			  floatingButtonBottomConstraint = makingBookFloatingButton.bottomAnchor.constraint(
			      equalTo: view.safeAreaLayoutGuide.bottomAnchor,
			      constant: -24
			  )
        floatingButtonBottomConstraint?.isActive = true
    }
}
```

플로팅 버튼의 Bottom Anchor를 view.safeAreaLayoutGuide.bottomAnchor로 주고, 초기 constant를 -24로 주었다.

그리고 그 리턴 값을 floatingButtonBottomConstraint에 저장해주고 isActive = true로 함으로써 오토레이아웃을 활성화해준다.

이제 floatingButtonBottomConstraint은 view.safeAreaLayoutGuide.bottomAnchor로부터 플로팅 버튼의 Bottom Anchor constant를 조절할 수 있게 됐다.

### 3. ScrollViewDidScroll 메소드 관리

ScrollViewDidScroll 메소드에서 **현재 스크롤 위치가 전체 콘텐츠 높이에서 화면 높이를 뺀 위치보다 아래에 있는지**를 검증한다.

맨 아래에 도달한 경우 (= offsetY > contentHeight - height) hideFloatingButton 메소드를 호출한다.

플로팅 버튼의 상태를 나타내는 isFloatingButtonHidden 프로퍼티를 true로 만들고,

위에서 설정한 Bottom Anchor 값을 갖는 floatingButtonBottomConstraint의 constant를 120 증가시킨다.

그리고 view.layoutIfNeeded()를 호출하여 뷰를 다시 그리는데, 이때 UIView의 animate를 주면 서서히 내려가는 것처럼 보인다.

```swift
extension HomeViewController: UICollectionViewDelegate {
    public func scrollViewDidScroll(_ scrollView: UIScrollView) {
        let offsetY = scrollView.contentOffset.y
        let contentHeight = scrollView.contentSize.height
        let height = scrollView.frame.size.height

		    offsetY > contentHeight - height
		    ? hideFloatingButton()
		    : showFloatingButton()
    }
    
    private func hideFloatingButton() {
        guard let floatingButtonBottomConstraint,
              !isFloatingButtonHidden else { return }
        isFloatingButtonHidden = true
        floatingButtonBottomConstraint.constant = 120
        UIView.animate(withDuration: 0.3) { [weak self] in
            self?.view.layoutIfNeeded()
        }
    }
    
    private func showFloatingButton() {
        guard let floatingButtonBottomConstraint,
              isFloatingButtonHidden else { return }
        isFloatingButtonHidden = false
        floatingButtonBottomConstraint.constant = -24
        UIView.animate(withDuration: 0.3) { [weak self] in
            self?.view.layoutIfNeeded()
        }
    }
}
```

### 구현 끝!

![플로팅 버튼 애니메이션](https://github.com/user-attachments/assets/ea846953-b085-4a56-a38a-bdd15c405298)


---

# 배운 점

- Constraint를 통해 런타임 시점에 오토레이아웃을 조절할 수 있다.
- scrollViewDidScroll 메소드의 동작 방식을 이해했다.

---

# 참조 링크

없음
