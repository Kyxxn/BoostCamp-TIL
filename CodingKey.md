## CodingKey가 뭔데 ?
- Codable을 채택한 모델은 JSON 파싱하려면 키값이 일치해야 한다.
- Swift와 같이 카멜케이스(e.g. parkHyoJun)를 사용하는 경우,
JSON의 스네이크 케이스(e.g. park_hyo_jun)와 다를 때 중첩 열거형을 통해 대신할 수 있다.

### 왜 쓰는진 알겠는데 어떻게 동작..?
- Codable을 채택한 모델에서 내부에 `enum CodingKeys: CodingKey`를 구현하곤 하는데
인코딩/디코딩 때는 해당 CodingKeys를 사용하지 않는다. 아래 상황을 봐보자.
- LabelEntity의 title 같은 경우 내가 사용하고 싶은 프로퍼티 명, name의 경우 JSON 키 값이다.
    ``` swift
    // MARK: DTO
    struct Label: Codable {
        let id: Int
        let nodeId: String
        let url: String
        let name: String
        let color: String
        let isDefault: Bool
        let description: String?
        
        enum CodingKeys: String, CodingKey {
            case id
            case nodeId = "node_id"
            case url
            case name
            case color
            case isDefault = "default"
            case description
        }
    }

    // MARK: Entity
    struct IssueEntity: Codable {
        let title: String
        let milestone: MilestoneEntity?
        let labels: [LabelEntity]
    }

    struct MilestoneEntity: Codable {
        let title: String
    }

    struct LabelEntity: Codable {
        let title: String
        
        enum CodingKeys: String, CodingKey {
            case title = "name"
        }
    }
    ```
- 그럼 디코딩할 때는 labelEntity.CodingKeys.title.rawValue를 사용할까? 그렇지 않았다.
    ``` swift
    private func decodeDummyData() -> [IssueEntity] {
        guard let url = Bundle.main.url(forResource: "issues", withExtension: "json") else { return [] }
        
        do {
            let data = try Data(contentsOf: url)
            let decoder = JSONDecoder()
            let issues = try decoder.decode([IssueEntity].self, from: data)
            return issues
        } catch {
            print(error)
            return []
        }
    }
    ```
- 흠.. 잘 되는 건 알겠으나 왜 되는지는 모르겠어서 GPT에게 물어보았다..
	- Swift의 Codable은 자동으로 구조체의 프로퍼티 이름과 JSON의 키 이름을 매칭합니다.
	- JSON의 키와 구조체의 프로퍼티 이름이 다르면 DecodingError가 발생합니다.
	- CodingKeys를 사용하지 않아도 Swift는 자동으로 키를 매칭하며, 매칭에 실패하면 keyNotFound 에러를 발생시킵니다.

    실제로 프로퍼티 매칭에 실패하면 아래와 같이 keyNotFound 에러가 뜬다.
    ```
    keyNotFound(CodingKeys(stringValue: "naㅇe", intValue: nil), Swift.DecodingError.Context(codingPath: [_CodingKey(stringValue: "Index 0", intValue: 0), CodingKeys(stringValue: "labels", intValue: nil)
    ```