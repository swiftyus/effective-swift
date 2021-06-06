# item58. 전통적인 for문보다는 for-each문을 사용하라

`Array`나 `Set`과 같은 컬렉션을 순회 할 때, for문을 사용할지, forEach문을 사용할지 고민됩니다. 두 방법 모두 순차적으로 element에 접근이 가능합니다. 메커니즘이 매우 유사해 선호도나 스타일에 따라 달리 사용하기도 하지만 몇가지 뚜렷한 면에서 차이가 있습니다. 책에서는 전통적인 for문과 향상된 for문(swift의 for in문)을 비교하고있는데, swift 3버전 이후로는 전통적인 for문을 이용할 수 없게 되어 이번 아이템에선 대표적으로 사용되는 swift의 for문과 forEach문의 차이점에 대해 알아봅니다.



### for문

for문의 반복은 코드 제어 흐름 내에서 직접 수행되어 이러한 반복을 훨씬 더 정확하게 제어 할 수 있다는 것입니다. 예를 들어 `continue` 키워드를 이용해서 다음 요소로 넘어갈지 혹은 `break` 키워드를 이용해서 반복을 멈출지 결정할 수 있습니다.

```swift
func categorizeDogs(among dogs: [Dog]) -> [Dog] {
  var results = [Dog]()

  for dog in dogs {
    guard dog.kind == .jindo else {
      // 즉시 다음 요소로 스킵
      continue
    }

    results.append(dog)

    guard results.count < 5 else { break }
  }

  return results
}
```

for문에서는 `where`절을 적용할 수 있어 위와 같은 코드는 아래처럼 간결하게 사용할 수도 있습니다.
```swift
for dog in dogs where dog.kind == .jindo {
  results.append(dog)

  guard results.count < 5 else { break }
}
```
이러한 흐름 제어 관련 기능(`continue`, `break`, `return`, `where`)은 for문을 선택하는 데 좋은 이유가 될 수 있습니다. 그리고 배열을 순회하면서 그 요소의 값 일부 혹은 전체를 변형하거나 선택된 요소를 제거해야 할 때와 여러 배열을 병렬로 순회하는 경우에도 for문을 이용하는 것이 좋습니다. 하지만 이러한 수준의 제어가 필요하지 않은 경우라면 forEach를 통해 조금 더 간단한 코드를 만들 수 있습니다.

<br>

### forEach

```swift
@inlinable
public func forEach(_ body: (Element) throws -> Void) rethrows {
  for element in self {
    try body(element)
  }
}
```
위의 코드는 forEach의 구현부입니다. closure 방식으로 사용되고 내부에선 for문을 사용해 모든 요소를 반복하고 각 요소를 파라메터로 사용해 요소의 갯수만큼 클로저를 호출하는 메서드임을 알 수 있습니다.

```swift
  dogs.forEach { dog in
    results.append(dog)
  }
```
이처럼 사용이 가능하지만 for문에서 사용이 가능했던 `continue`, `break`, `return`, `where`등의 키워드는 forEach 내부에서는 사용이 불가능합니다. 컬렉션 타입의 모든 요소를 예외없이 반복하는 경우에 forEach를 사용하면 안전하게 사용할 수 있습니다.

<br>

### References
- [SwiftBySundell](https://www.swiftbysundell.com/tips/picking-between-for-and-for-each/)
- [DonnyWals](https://www.donnywals.com/choosing-between-a-for-loop-and-a-foreach/)
