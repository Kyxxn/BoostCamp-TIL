## cell에서 URL을 Image로 변환하기
### 상황
1. 담당자들을 보여주는 TableView을 배치하려 합니다.
2. Service 계층에서 [담당자] 배열을 받아오고, ... ViewModel에서 [담당자] Entity를 @Published로 갖고 있습니다.
3. Controller는 ViewModel의 [담당자]를 관측하고 있어서 값이 바뀌면 `TableViewDatasource` 메소드에서 cell을 configure 합니다.
    ``` swift
    extension AssigneeRegistrationViewController: UITableViewDataSource {
        
        func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {

            guard let cell = tableView.dequeueReusableCell(withIdentifier: AssigneeTableViewCell.identifier,
                                                        for: indexPath) as? AssigneeTableViewCell
            else { return UITableViewCell()}

            let user = viewModel.assignees[indexPath.row]
            cell.configure(user: user) // 해당 코드 부분
            return cell
        }
    }
    ```
    - user는 다음과 같은 Entity 입니다.
    ``` swift
    struct UserEntity: Codable {
        let id: Int
        let login: String
        let avatarURL: String?
    }
    ```
4. **여기서 궁금한 사항이 생겼습니다.**
cell이 user를 받게 되면 `avatarURL`을 갖고 UIImage로 변환을 해야 합니다.

### 내가 떠올린 해결방안
- cell은 보여지는 `View`인데, URL을 통해 이미지를 받아오는 작업을 한다는 것은 아키텍처 구조상 맞지 않다고 생각했습니다.
- **제가 떠올린 첫 번째 방법**으로 `ImageLoader`라는 싱글톤을 만들고 cell은 비동기로 url을 해당 싱글톤에 넘겨주면 그 결과를 토대로 Image를 보여주려 했습니다.
    - **그러나**, ImageLoader가 비동기로 이미지를 갖고 온다고 하지만, `import UIKit`을 해서 UIImage를 반환해야 하는게 맞나 ? 라고 생각했습니다.

- **두 번째 해결책**으로 ImageLoader가 `UIImage`를 반환하지 않고, `Data`를 반환하고 cell에서 해당 Data를 UIImage로 바꾸는 작업을 하면 `ImageLoader`가 UIKit을 의존하지 않아도 될 거라 생각했습니다.
    - 그런데 또.. Data로 받는데 1번, UIImage로 바꾸는데 1번, 총 2번의 이미지 처리를 해야하니 굉장히 비효율적일 것 같습니다!

