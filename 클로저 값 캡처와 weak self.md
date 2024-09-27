## 클로저에서 weak self
> [링크 바로가기](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/32#discussion_r1773639911)

``` swift
NetworkHandler.requestApi  { [weak self] issues in
    DispatchQueue.main.async { // 여기서 weak self 안 해도 되는가 ?
        guard let self = self else { return }
        self.issueList = issues // issueList 업데이트
        self.setUpTableView()
    }
} 
```

- Q. NetworkHandler 클로저에서는 `[weak self]`를 사용했는데,
DispatchQueue 클로저에서는 사용하지 않았다.
- A. 각 클로저마다 캡처가 동작하므로, `[weak self]`를 붙여줘야 함
안 붙여도 되는 경우가 있긴 한데, 붙여야 할 경우에 `[weak self]`를 안 쓰면 문제가 생기니 웬만하면 붙이는 것을 추천.