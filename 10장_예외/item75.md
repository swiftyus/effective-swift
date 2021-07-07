# Item 75. 예외의 상세 메시지에 실패 관련 정보를 담으라



개발자가 실패를 분석하기 위해서 예외 메시지는 필수적으로 사용됩니다. 더군다나 그 실패가 재현하기 어렵다면 더더욱 그렇습니다.

따라서 예외의 `toString` 메소드에 실패에 대한 원인을 최대한 많이 담아야 합니다.

<br>

**실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 합니다.**

```swift
class IndexOutOfRangeError: LocalizedError {

    struct IndexOutOfRangeErrorContents {
        private let lowerBound: Int
        private let upperBound: Int
        private let index: Int
        
        init(lowerBound: Int, upperBound: Int, index: Int) {
            self.lowerBound = lowerBound
            self.upperBound = upperBound
            self.index = index
        }
    }
    
    private let contents: IndexOutOfRangeErrorContents
    
		// 해당 error에 대한 설명
    var errorDescription: String? {
        return "Index out of range"
    }
    
		// 해당 error에 대한 사용자를 위한 설명
    var errorDescriptionToUser: String? {
        return "잘못된 접근입니다."
    }
    
		// 해당 error의 원인
    var failureReason: String? {
        return "\\(dump(contents))"
    }
    
    init(contents: IndexOutOfRangeErrorContents) {
        self.contents = contents
    }
}

```

책의 `IndexOutOfBoundsException`에 대한 코드를 Swift로 변환하였습니다.

- `LocalizedError` 는 error를 채택하고 있는 프로토콜입니다.
- 에러 메시지에서 사용될 내용들은 내부 `struct`를 이용하여 표현합니다.
- 통상적으로 필요할 수 있는 description, descritionToUser, failureReson을 구현하였습니다.

<br>

**LocalizedError**

```swift
public protocol LocalizedError : Error {

    /// A localized message describing what error occurred.
    var errorDescription: String? { get }

    /// A localized message describing the reason for the failure.
    var failureReason: String? { get }

    /// A localized message describing how one might recover from the failure.
    var recoverySuggestion: String? { get }

    /// A localized message providing "help" text if the user requests help.
    var helpAnchor: String? { get }
}

```