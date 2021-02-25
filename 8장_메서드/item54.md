# Item 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라



### 핵심 정리

> null이 아닌, 빈 배열이나 컬렉션을 반환하라. null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 그렇다고 성능이 좋은 것도 아니다.
>
> -출처: 이펙티브 자바



이펙티브 자바에서는 반환할 값이 없을 경우 null을 반환하는 게 아니라 빈 배열이나 컬렉션을 반환하라고 권하고 있습니다.

그렇다면 Swift 에서는 어떻게 바라보는게 좋을까요?

### Empty Array vs Optional

Swift에서는 특별히 Best Practice나 공공연하게 권장되는 바는 없는 것 같습니다. 제 생각에는 목적에 맞게 데이터를 더 잘 설명할 수 있는 걸 선택하는 것이 좋을 것 같습니다.

* return Empty Array: 
  Swift는 Type Safety (타입 안전성)을 중요시하는 언어입니다. 이런 면에서 생각해봤을 때 반환할 값이 없을 때라도 메서드의 리턴 타입을 nil이 아닌 빈 배열을 반환하는 것은 이미 타입의 래퍼(wrapper) 역할을 한다고 할 수 있습니다.
  
  EX) 쇼핑몰에서 모든 유저가 장바구니를 가지고 있지만 장바구니에 아무것도 담겨있지 않은 상태를 길이가 0인 빈 배열로 표현할 수 있습니다.

* return Optional: 
  nil을 사용했을 때 말이 되는 경우는 호출자에게 값이 존재하지 않는 경우(nil)나 존재하지 않을 수도 있음을 확실히 전달하려는 경우. 

추가적으로 스터디원분들의 의견이 있으면 이야기 나눠보고 싶습니다 :)

### 참고

1. [Optional array vs. empty array in Swift](https://stackoverflow.com/questions/26811787/optional-array-vs-empty-array-in-swift)