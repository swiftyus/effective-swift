# Item 36. 비트 필드 대신 EnumSet을 사용하라

## 목차

- 비트 필드의 사용
- EnumSet의 사용 
- Swift에서의 개념 적용
  - OptionSet의 사용
  - EnumSet 처럼 확장하기
- 정리

---

## 비트 필드의 사용 

### 사용 목적

보통 **비트 필드**를 이용하는 목적은 8비트, 32비트, 64비트만을 사용하여 효율적으로 데이터를 사용하기 위함입니다. 

또한 **비트 연산**을 통하여 효율성을 극대화 시킬 수 있습니다. 



```java
public class Text {
  public static final int STYLE_BOLD    = 1 << 0
  public static final int STYLE_ITALIC    = 1 << 1
  public static final int STYLE_UNDERLINE   = 1 << 2
  public static final int STYLE_STRIKETHROUGH   = 1 << 3
 
    // or 등의 비트 연산자를 이용하여 스타일을 적용시킨다. 
  public void applyStyles(int styles) { ... }
}
```

책에서는 위와같이 비트 필드를 이용하여 상수를 선언하고 사용하고 있습니다. 

<Br>

하지만 위와 같은 방식으로 사용하는 경우 몇 가지의 문제점을 가지고 있습니다. 

### 문제점

1. 비트를 사용하여도 **정수 상수 열거의 단점**을 그대로 갖고 있습니다. 
2. 상수들의 **raw value**들을 해석하기 어렵습니다. 
3. 열거된 상수들을 **순회**하기 힘듭니다. 
4. 사전에 **최소, 최대값을 예측**하여 적절한 타입을 결정해야 합니다. ( 이후 최대, 최소 값이 변하는 경우 API 자체를 변경해야 하는 문제 )


---

## EnumSet의 사용

책에서는 위와 같은 비트 필드 대신 `EnumSet`의 사용을 권장하고 있습니다. 

<Br>

### EnumSet이란? 

* 열거 타입 상수 값으로 구성된 집합 클래스입니다. 

* `Set` 인터페이스를 완벽히 구현하고 있습니다. 

* 타입 안정적입니다. 

* 다른 Set 구현체와 함께 사용이 가능합니다.

  ​

<Br>

### EnumSet의 장점

`EnumSet`은 java.util 패키지에 정의되어 있으며 내부는 비트 벡터로 구현되어 있습니다. 그렇기 때문에 비트 필드를 이용하는 것과 유사한 이점을 얻을 수 있습니다. 

<Br>

`EnumSet`에는 `removeAll`, `retainAll` 과 같은 메소드가 정의되어 있어서 대량작업을 하는 경우에도 이점이 있습니다. 

<Br>

```java
public class Text {
  public enum Style { 
    Bold,
    ITALIC,
    UNDERLINE,
    STRIKETHROUGH
  }
  
  public void applyStyles(Set<Style> styles) { ... }
}
```

위와 같이 `enum`을 정의해서 기존의 비트 필드의 상수 정수들을 표현할 수 있습니다. 

<Br>

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC))
```

그리고 직접 메소드를 사용할 때는 위와 같이 사용됩니다. 



> (참고) applyStyles 메소드를 정의할 때 `EnumSet`을 매개변수로 사용하지 않고 `Set`을 전달하는 이유는 `EnumSet`이 사용된다고 가정한다면, 상위의 인터페이스를 사용하는 것이 **확장성**이 높기 때문입니다. 

----

## Swift에서의 개념 적용 

먼저 **Swift** 에서는 **java** 와 같이 비트 벡터를 기반으로 작성된 `EnumSet`과 같은 타입은 존재하지 않습니다. 하지만 **Swift** 에서는 `OptionSet` 이라는 타입이 존재합니다. 

<Br>

### 1. OptionSet

#### 정의

You use the `OptionSet` protocol to represent bitset types, where individual bits represent members of a set.

Set의 형태로 비트 타입을 이용할 때 사용된다고 합니다. 

<Br>

#### 사용방법

`OptionSet` 의 기본적인 사용방법은 

```swift
struct ShippingOptions: OptionSet {
    let rawValue: Int

    static let nextDay    = ShippingOptions(rawValue: 1 << 0)
    static let secondDay  = ShippingOptions(rawValue: 1 << 1)
    static let priority   = ShippingOptions(rawValue: 1 << 2)
    static let standard   = ShippingOptions(rawValue: 1 << 3)

    static let express: ShippingOptions = [.nextDay, .secondDay]
    static let all: ShippingOptions = [.express, .priority, .standard]
}
```

<Br>

`OptionSet` 프로토콜을 채택하는 경우 

```swift
let singleOption: ShippingOptions = .priority
let multipleOptions: ShippingOptions = [.nextDay, .secondDay, .priority]
let noOptions: ShippingOptions = []
```

와 같이 사용될 수 있습니다. 

<Br>

```swift
let purchasePrice = 87.55

var freeOptions: ShippingOptions = []
if purchasePrice > 50 {
    freeOptions.insert(.priority)
}

if freeOptions.contains(.priority) {
    print("You've earned free priority shipping!")
} else {
    print("Add more to your cart for free priority shipping!")
}
```

<Br>

`OptionSet` 은 여러 메소드가 정의되어 있습니다. 

<img src="resources/item36-optionsSetMethod.png">

#### OptionSet 정리

아직 다양한 방법으로 사용해보지는 못했지만, java의 `EnumSet` 과 같이 다양하게 활용될 수 있다고 생각됩니다. 

-----

### 2. OptionSet 확장해서 사용하기

`OptionSet`을 사용할 때는 보통 `struct` 를 사용하는 것으로 보여진다. 그런데 이렇게 `Struct` 에 `OptionSet` 을 채택해서 사용하는 경우 필드들을 직접 순회할 수 없습니다. 

`OptionSet` 을 이용한 `struct` 의 필드들을 순회하기 위해서는 

```swift
struct OptionSetIterator<Element: OptionSet>: IteratorProtocol where Element.RawValue == Int {
    private let value: Element

    public init(element: Element) {
        self.value = element
    }

    private lazy var remainingBits = value.rawValue
    private var bitMask = 1

    public mutating func next() -> Element? {
        while remainingBits != 0 {
            defer { bitMask = bitMask &* 2 }
            if remainingBits & bitMask != 0 {
                remainingBits = remainingBits & ~bitMask
                return Element(rawValue: bitMask)
            }
        }
        return nil
    }
}
```

직접 `Sequence` 와 `ItertorProtocol` 을 이용해서 **Iterator**를 만들어야 합니다. 

<Br>

그러고나서 

```swift
extension OptionSet where Self.RawValue == Int {
   func makeIterator() -> OptionSetIterator<Self> {
      return OptionSetIterator(element: self)
   }
}
```

`makeIterator` 를 오버라이딩 합니다. 

<Br>

그러고 나면

```swift
struct ShippingOptions: OptionSet, Sequence {
  // Sequence 프로토콜을 채택한다. 
    let rawValue: Int

    static let nextDay    = ShippingOptions(rawValue: 1 << 0)
    static let secondDay  = ShippingOptions(rawValue: 1 << 1)
    static let priority   = ShippingOptions(rawValue: 1 << 2)
    static let standard   = ShippingOptions(rawValue: 1 << 3)

    static let express: ShippingOptions = [.nextDay, .secondDay]
    static let all: ShippingOptions = [.express, .priority, .standard]
}

let allShippingOptions: ShippingOptions = .all

func printShippingOptions(_ shippingOptions: ShippingOptions) {
  for option in shippingOptions {
    print(option)
  }
}
```

 <Br>

이렇게 아주 거창한(?) 코드들을 만들고 나서야 `OptionsSet` 을 순회할 수 있게 됩니다. 

하지만 이러한 방법 보다 조금 더 간편한 방법이 있어 소개해보려고 합니다. 

----

### 3. Enum 확장해서 사용하기 



```swift
protocol Option: RawRepresentable, Hashable, CaseIterable {}
```

먼저 `OptionsSet` 을 대신할 `Option` 을 선언합니다. 

<Br>

```swift
enum ShippingOptions: String, Option {
  case nextDay, secondDay, priority, standard
}
```

이전에 정의한 `ShippingOptions` 를 `Enum` 으로 구현합니다. 

<Br>

```swift
extension Set where Element: Option {
    var rawValue: Int {
        var rawValue = 0
        for (index, element) in Element.allCases.enumerated() {
            if self.contains(element) {
                rawValue |= (1 << index)
            }
        }

        return rawValue
    }
}
```

그리고 `Set` 에 `Option` 을 이용해서 비트값을 이용해서 순회하도록 구성하였습니다. 



## 정리 

 사실 개발을 하면서 비트 값과 비트 연산을 통해 코드를 구현할 일이 거의 없었는데, 이번에 `OptionSet`과 `Set` 을 확장해서 사용하는 코드들을 보고 내가 너무 모르고 있다는 것을 다시 한 번 깨닫게 되었습니다. 

위의 개념들을 실제 코드에 사용하려면 다양한 사례들을 보면서 더 많은 학습이 필요할 것으로 보입니다! 화이팅! 

> **`참고자료`**
>
> https://nshipster.com/optionset/
>
> https://stackoverflow.com/questions/36819163/optionsettype-and-enums
>
> https://swiftrocks.com/how-optionset-works-inside-the-swift-compiler
>
> http://minsone.github.io/mac/ios/swift-sequence
>
> https://www.hackingwithswift.com/example-code/language/how-to-make-a-custom-sequence
