
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