# item 50. 적시에 방어적 복사본을 만들라

해당 아이템에서는 되도록 가변 타입을 사용하지 말고, 혹시나 가변 타입을 사용하는 경우 어떻게 해야 불변식을 보장할 수 있는지를 설명하고 있습니다.

외부에서 악의적인 의도나 혹은 단순한 프로그래머의 실수로 우리가 만든 클래스를 오작동하게 만들 수 있기 때문에 방어적으로 프로그래밍해야 한다고 설명합니다.

아래의 클래스는 기간을 표현하는 클래스입니다.

```swift
final class Period {
    private var start: Date
    private var end: Date
    
    init(start: Date, end: Date) throws {
        if start > end {
            throw NSError(domain: "\(start)가 \(end)보다 늦다.", code: 999)
        }
        
        self.start = start
        self.end = end
    }
    
    public func getStart() -> Date {
        return self.start
    }
    
    public func getEnd() -> Date {
        return self.end
    }
    
    //...
}

var start = Date()
var end = Date()

let period = try? Period(start: start, end: end)

// period class의 end에 영향이 갈까요?
end.addTimeInterval(1000)
```

Java에서는 Date가 가변 타입이므로 외부의 `start` 혹은 `end` 변수를 수정하게 될 경우 `period` 내부의 `start`, `end` 값 또한 변경됩니다.

하지만 Swift에서는 이러한 부작용이 적용되지 않습니다. Swift의 `값 타입(Value Type)`은 기본적으로 깊은 복사를 지원하고 Date가 `값 타입`인 struct(구조체)로 구성되어 있기 때문입니다.

<br>

하지만 class같은 `참조 타입(Reference Type)`인 경우 얕은 복사에 대한 부작용을 걱정해야 합니다.

```swift
class Person {
    var isHungry: Bool = true
}

let lin = Person()
let anotherPerson = lin

print(anotherPerson.isHungry) //true
```

<br>

이 책에서는 방어적 복사본을 만들어 반환하거나, 유효성을 검사하기전에 방어적 복사본을 만들어 해당 복사본으로 유효성 검사를 진행하라고 합니다.

그럼 class와 같은 참조 타입은 어떤식으로 방어적 복사본을 만들 수 있을까요?

```swift
class Person: NSCopying {
    func copy(with zone: NSZone? = nil) -> Any {
        let copy = Person()
        copy.isHungry = self.isHungry
        
        return copy
    }
}

let lin = Person()
let anotherPerson = lin.copy()
lin.isHungry = false

print(lin.isHungry) // false
print(anotherPerson.isHungry) // true
```
class의 경우 `NSCopying` protocol을 채택하고 `copy` 메소드를 구현해 내부에서 새 객체를 만들어 반환하는 방법이 있습니다.

이렇게 불변식을 구성하는 것도 좋은 방법이지만, 성능 저하가 일어날 수 있고 메서드나 생성자의 매개변수로 넘기는 행위가 그 객체의 통제권을 명백히 이전함을 뜻하기도 하기 때문에 항상 방어적으로 복사해 저장해야 하는 것은 아닙니다.

하지만 이럴 경우 방어적 복사를 수행하는 대신에 해당 객체의 내부 요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명확히 명시하도록 해야합니다.

