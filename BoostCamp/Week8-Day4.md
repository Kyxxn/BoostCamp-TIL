## 메모리 & 디스크 캐싱 직접 구현해보기
### 핵심 키워드
- 캐싱 동작 원리
- 메모리와 디스크 값 동기화 고민
- 메모리 이미지 캐싱 용량 결정하기
- TTL 도입하여 디스크 캐싱 관리하기 (파일 아이노드의 마지막 수정으로 비교)

<br>

## 모듈 결정
- Clean Architecture로 모듈화가 되어 있는 내 프로젝트에서 `Presentation` 계층에 비동기로 이미지를 불러오는 `ImageLoader` 싱글톤이 있다.
- 위 싱글톤은 Presentation에서만 쓰는데, 캐싱이라는 기능 자체가 이미지 뿐만 아니라 카카오톡의 채팅방처럼 텍스트를 저장할 수도 있어서 Core 계층에 두기로 정했다.
- `MemoryCache`는 싱글톤 클래스로, `DiskCache`는 enum과 static 프로퍼티/메소드를 활용한 네임스페이스로 정했다.

<br>

## 메모리 캐싱 동작 정의
### Load FLOW
- 메모리에서 사진을 찾는다.
	``` swift
	public final class MemoryCache: ImageCacheable {
		public static let shared = MemoryCache()
		private let memoryCache = NSCache<NSString, AnyObject>()
		
		public func loadImage(from urlString: String) -> UIImage? {
			let urlNSString = urlString as NSString
			// 메모리에 있다면,
			if let memoryCacheImage = memoryCache.object(forKey: urlNSString) as? UIImage {
				return memoryCacheImage
			}
			
			// 메모리에 없다면,
			if let imageData = DiskCache.loadData(from: urlString),
			let diskCacheimage = UIImage(data: imageData) {
				memoryCache.setObject(diskCacheimage, forKey: urlNSString)
				return diskCacheimage
			}
			
			return nil
		}
	}
	```
	- 있다면? 꺼내서 리턴
	- 없다면? 디스크까지 접근

- 디스크에서 사진을 찾는다.
	``` swift
	public enum DiskCache {
		private static let fileManager = FileManager.default
    	private static let expirationInterval: TimeInterval = Constants.dueDate

		static func loadData(from urlString: String) -> Data? {
			guard let filePath = createfilePath(from: urlString),
				fileManager.fileExists(atPath: filePath.path()) else { return nil }
			
			guard let attributes = try? fileManager.attributesOfItem(atPath: filePath.path()),
				let lastModifiedDate = attributes[.modificationDate] as? Date
			else { return nil }
			
			if Date().timeIntervalSince(lastModifiedDate) > expirationInterval {
				try? fileManager.removeItem(at: filePath)
				return nil
			}
			try? fileManager.setAttributes([.modificationDate: Date.now], ofItemAtPath: filePath.path())
			return fileManager.contents(atPath: filePath.path())
		}
	```
	- 디스크에서 찾았다면?
		- TTL이 만료되었는 지 확인하고 만료되면 nil 반환
        - TTL이 만료되지 않으면, TTL 갱신 후 디스크에 있는 이미지 리턴
        - 마지막으로 메모리 업데이트 후 이미지 리턴
	- 디스크에서 못 찾았다면? nil을 반환하고 메모리는 네트워크 통신 후 메모리와 디스크를 업데이트

### Save FLOW
- 메모리: 바로 저장하고, 디스크도 갱신한다. (동기화를 위해)
	- 이때 메모리에 저장한 값과 디스크에 저장되어 있는 값이 같으면 저장 안함 (코스트 아끼기 위해 조건식 추가)
	- 메모리 != 디스크 인 경우, 디스크에 저장된 값 업데이트
- 디스크: FileManager의 홈 디렉토리에 있는 cachesDirectory를 관리한다.

<br>

## 메모리 이미지 캐싱 용량 결정하기
> [레퍼런스](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/284#issuecomment-2392562524)

<img width="869" alt="스크린샷 2024-10-18 오전 2 45 28" src="https://github.com/user-attachments/assets/118833dd-76ae-4606-8753-fa0d48a48db2">

- NSCache의 용량을 결정할 때 `totalCostLimit`을 결정을 뭘로 할지 고민했다.
	- 디폴트 값은 0이고, 0은 한도가 없다는 뜻이다. (무제한으로 받는듯)
- 그래도 메모리가 계속 차면 앱이 죽어버릴 수 있으니까 나만의 용량 설정이 필요했는데, 멘토님의 피드백을 보았다.
- 앱이 실행되는 기기의 메모리 용량을 가져오는 방법이 있으니 5~10% 정도를 캐시에 쓰면 괜찮을 것 같다는 말을 듣고 아래와 같이 작성했다.
- TMI) Xcode에서 제공해주는 시뮬레이터의 램은 16GB이다.

``` swift
public final class MemoryCache: ImageCacheable {
    public static let shared = MemoryCache()
    private let memoryCache = NSCache<NSString, AnyObject>()
    
    private var calculatedMemoryLimit: Int {
        let physicalMemoryBytes = ProcessInfo.processInfo.physicalMemory / 1024 / 1024
        let memoryLimitMegaBytes = Int(physicalMemoryBytes / 100 * 5)
        return memoryLimitMegaBytes
    }

    private init() {
        memoryCache.totalCostLimit = calculatedMemoryLimit
    }
	...
}
```

<br>

## TTL 도입하여 디스크 캐싱 관리하기
> [레퍼런스](https://cultured-farmhouse-488.notion.site/117a44a4d9ab8064922fe363045b951b)

- 디스크에 캐시가 저장이 되면 관리해주지 않을 시 앱의 용량이 증가하게 된다.
	- 내 앱의 경우 담당자 프로필 사진 270명을 저장했더니 앱 용량이 20MB 증가했다.
- cachesDirectory에 프로필 사진 URL에 따라서 키값으로 저장해주는데, 이게 곧 `FileManager의 createFile`로 파일 저장된다.
- 파일로 저장되는 게 핵심임. 파일의 아이노드 중 마지막 수정 날짜를 통해서 TTL과 비교한다.
- 캐시의 유효시간인 TTL을 하루로 설정하고, 파일의 최근 수정날짜와 비교하면서 디스크에 저장된 파일 최신화를 관리한다.
- 또, 주기적으로 관리해주면 좋은데 이를 SceneDelegate에서 앱이 백그라운드로 갈 때 `clearExpriedCacheData`를 호출하여 디스크에 담긴 캐싱 데이터들 중에 TTL 만료된 걸 정리해준다.

``` swift
public enum DiskCache {
    private static let fileManager = FileManager.default
    private static let expirationInterval: TimeInterval = Constants.dueDate

    static func loadData(from urlString: String) -> Data? {
        guard let filePath = createfilePath(from: urlString),
              fileManager.fileExists(atPath: filePath.path()) else { return nil }
        
        guard let attributes = try? fileManager.attributesOfItem(atPath: filePath.path()),
              let lastModifiedDate = attributes[.modificationDate] as? Date
        else { return nil }
        
        if Date().timeIntervalSince(lastModifiedDate) > expirationInterval {
            try? fileManager.removeItem(at: filePath)
            return nil
        }
        try? fileManager.setAttributes([.modificationDate: Date.now], ofItemAtPath: filePath.path())
        return fileManager.contents(atPath: filePath.path())
    }

	private enum Constants {
        static let dueDate: Double = 60 * 60 * 24
    }

	public static func clearExpriedCacheData() {
        guard let cacheDirectoryPath,
              let fileURLs = try? fileManager.contentsOfDirectory(
                at: cacheDirectoryPath,
                includingPropertiesForKeys: [.contentModificationDateKey])
        else { return }
        
        for fileURL in fileURLs {
            do {
                let attribute = try fileManager.attributesOfItem(atPath: fileURL.path())
                if let lastModifiedDate = attribute[.modificationDate] as? Date {
                    if Date().timeIntervalSince(lastModifiedDate) > Constants.dueDate {
                        try fileManager.removeItem(at: fileURL)
                    }
                }
            } catch {
                print("캐시 디렉토리의 각 파일 별로 속성 관리에서 문제: \(error)")
            }
        }
    }
}

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    func sceneDidEnterBackground(_ scene: UIScene) {
        DiskCache.clearExpriedCacheData()
    }
    
}
```

<br>

### TMI
<img width="502" alt="스크린샷 2024-10-18 오전 1 29 57" src="https://github.com/user-attachments/assets/3a3bdb55-93dd-4c8f-b59a-31f1fb8b2892">

오늘도 열심히 해따

## 레퍼런스
- [애플 공식문서 totalCostLimit](https://developer.apple.com/documentation/foundation/nscache/1407672-totalcostlimit)
- [메모리 이미지 캐싱 용량 결정하기](https://github.com/boostcampwm-2024/swift-p3-issue-tracker/pull/284#issuecomment-2392562524)
- [TTL 도입하여 디스크 캐싱 관리](https://cultured-farmhouse-488.notion.site/117a44a4d9ab8064922fe363045b951b)
