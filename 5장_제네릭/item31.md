# Item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

자바에서 매개변수화 타입은 불공변입니다. 이 때문에 제네릭을 사용한 API을 오직 매개변수화된 해당 타입 하나로만 사용할 수 있어서 API 유연성이 다소 떨어집니다. 이 때 한정적 와일드카드를 사용하면 하위 타입 또는 상위 타입도 입력할 수 있어 더 유연한 API를 만들 수 있습니다. 자바에서의 한정적 와일드카드에 대해 요약하고, Objective-C와 스위프트에서 이와 관련된 내용에 대해 정리해보겠습니다.

<br>

### 자바의 한정적 와일드카드 사용하기

```java
public class Stack<E> {
    // ...
    public void pushAll(Iterable<E> src) {
        for (E e : src)
            push(e);
    }
}
```

위와 같이 pushAll 메서드를 작성할 경우, 매개변수화 타입은 불공변이기 때문에 스택을 `Stack<Number>`로 선언한 후에 `Number`의 하위 타입인 `Integer` 타입의 요소를 push하려 한다면 에러가 발생합니다.

```java
public void pushAll(Iterable<? extends E> src) {
    // ...
}
```

이렇게 한정적 와일드카드 타입을 사용하여 Iterable 인터페이스를 구현한 E의 하위 타입도 입력할 수 있도록 만들 수 있습니다.

만약 pushAll과 반대인 popAll을 구현한다면, 스택 요소의 타입보다 상위 타입의 매개변수로 받아야 하므로 다음과 같이 super를 사용해야 합니다.

```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

<br>

### Objective-C에서의 `__covariant`와 `__contravariant`

Objcective-C에는 `__covariant`와 `__contravariant`라는 키워드가 있습니다. 제네릭 파라미터 앞에 `__covariant`를 붙이는 것은 서브타입들을 받아들일 수 있음을 의미하며, `__contravariant`를 붙이는 것은 슈퍼타입들을 받아들일 수 있음을 의미합니다.

```objectivec
@interface Queue<__covariant ObjectType> : NSObject

- (void)enqueue:(ObjectType)value;
- (ObjectType)dequeue;

@end
```

<br>

### 스위프트 제네릭에서 하위 타입 받기

스위프트 제네릭 또한 자바처럼 불공변입니다.

```swift
class Garage<Car> { ... }

func put(in garage: Garage<Car>) { ... }

put(in: Garage<Car>()) // 가능
put(in: Garage<PoliceCar>()) // 에러
```

마지막 줄에서 Garage<Car> 타입을 받는 매개변수에 Garage<PoliceCar>를 입력하고 있어서 다음과 같은 에러가 발생합니다.

```
Cannot convert value of type 'Garage<PoliceCar>' to expected argument type 'Garage<Car>'
```

하지만 스위프트에는 Objective-C처럼 서브타입 또는 슈퍼타입을 받아들일 수 있음을 나타내는 키워드는 없습니다. 스위프트 제네릭에서 서브 타입만을 입력할 수 있게 만드려면 다음과 같이 제네릭 type constraint를 이용할 수 있습니다.

```swift
func put<C: Car>(in garage: Garage<C>) { ... }

put(in: Garage<Car>())
put(in: Garage<PoliceCar>()) // 가능
```

Type constraint를 이용해 Car의 서브클래스들만 입력할 수 있도록 제한하긴 했지만, 이는 자바의 한정적 와일드카드 타입과는 다른 기능이어서 완전히 동일한 역할을 수행해 주지는 않습니다. 또한 제네릭 type constraint를 이용해 슈퍼 타입만 받는 방법은 찾을 수 없었습니다.

<br>

### 스위프트에서 공변인 경우

스위프트에서, 커스텀 타입의 제네릭은 불공변이지만 배열은 공변입니다.

```swift
class Car { ... }
class PoliceCar: Car { ... }

func drive(_ cars: Array<Car>) { ... }

drive(Array<Car>())
drive(Array<PoliceCar>()) // 가능
```

그 이유를 추측하기 위해 NSArray의 Objective-C 인터페이스를 보면 다음과 같습니다.

```objectivec
@interface NSArray<__covariant ObjectType> : NSObject
```

제네릭 파라미터가 `__covariant`로 선언되어 있어 서브 타입들도 배열에 입력할 수 있습니다. 스위프트에서의 배열은 NSArray와 bridging되어 있어서 호환성을 위해 공변이 아닐까 추측됩니다.

<br>

### 결론

자바에서 매개변수화 타입이 불공변인 것 때문에 제네릭을 이용한 API를 설계할 때 문제점이 있으니, 한정적 와일드카드 타입을 사용하여 API 유연성을 높이는 편이 좋습니다.

Objective-C에서는 비슷한 역할을 하는 `__covariant`와 `__contravariant` 키워드가 있지만, 스위프트에는 이러한 기능이 없고, 제네릭 type constraint를 이용해 하위 타입만 받을 수 있도록 제약할 수 있긴 하지만, 자바의 한정적 와일드카드 타입과는 그 역할이 다릅니다.

추가적으로, 스위프트에서 배열이 공변인 점에 대해 기술하고 그 이유를 추측해 보았습니다.

<br>

### References

- [NSArray](https://developer.apple.com/documentation/foundation/nsarray?language=objc)
- [Covariance and Contravariance](https://www.mikeash.com/pyblog/friday-qa-2015-11-20-covariance-and-contravariance.html)

