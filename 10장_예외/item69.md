# Item 69. 예외는 진짜 예외 상황에만 사용하라

이번 장에서는 말 그대로 예외처리는 진짜 예외 처리가 필요한 부분에서만 예외처리를 하라는 것입니다.

<br>

### **잘못된 예외처리**

**java**

```
try {
	int i = 0;
	while(true) {
	  range[i++].climb();
  }
} catch (ArrayIndexOutOfBoundsException e) {
}

```

**swift**

```
let array: [String] = ["one","two","three","four","five"]

do {
    var index = 0
    while(true) {
				let value = try array[index]      
// yellow error: No calls to throwing functions occur within 'try' expression
        print("\\(value)")
        index += 1
    }
} catch (let error) {
    print("fatalError: \\(error.localizedDescription)")
}

// output
// Fatal error: Index out of range: file Swift/ContiguousArrayBuffer.swift, line 444
// 2021-03-20 19:43:37.193369+0900 Test_Iterator[18969:1365911] Fatal error: Index out of range: file Swift/ContiguousArrayBuffer.swift, line 444

```

- Swift에서 `Index out of range` Error 타입으로 제공하는 것이 없어서 `let error` 대신 catch하였습니다.
- Swift에서는 배열에 접근할 때 try 문을 통해 throw할 방법이 없습니다.
- 직접 catch 한 것 외에도 시스템에서 FatalError를 내보냅니다.

<br>

**배열에 직접 접근할 때 try문을 사용할 수 없는 이유**

Array를 inex를 통해 직접 접근하는 경우

```
let value = array[3]

```

NSArray의 object 메소드를 사용하게 됩니다.

```
func object(at index: Int) -> Any 

```

- 애초에 `object(:at)` 라는 메소드에서 throw 처리가 안되어 있기 때문에 `try` 를 사용할 수가 없습니다.

책에서는 위와 같은 예시를 통해 잘못된 예외처리 상황을 설명하고 있습니다.

<br>

### **잘못된 이유**

1. 예외는 예외 상황에 쓸 용도로 설계되었기 때문에 **최적화가 되어있지 않습니다**. 그렇기 때문에 위와 같은 try-catch 블럭에 반복문을 설계한 것은 전체적인 성능을 떨어뜨립니다.
2. 기본적으로 배열을 순회하는 표준 관용구에서는 JVM에서 최적화해 처리해줍니다.
3. 위의 코드의 경우 배열에 문제가 있는 경우에 `ArrayIndexOutOfBoundsException` 예외로 처리되기 때문에 디버깅하기가 더 어려워집니다.

**Java**

```
for (Iterator<Foo> i = collection.iterator(); i.hasNext();) {
  Foo foo = i.next()
	...
}

```

일반적으로 java에서는 Iterator 타입에서 제공하는 hasNext()를 통해 쉽게 반복문을 사용할 수 있습니다.

**Swift**

```
// IteratorProtocol.swift 

public protocol IteratorProtocol {
  mutating func next() -> Self.Element?
}

// Example

struct Countdown: Sequence {
    let start: Int

    func makeIterator() -> CountdownIterator {
        return CountdownIterator(self)
    }
}

struct CountdownIterator: IteratorProtocol {
    let countdown: Countdown
    var times = 0

    init(_ countdown: Countdown) {
        self.countdown = countdown
    }

    mutating func next() -> Int? {
        let nextNumber = countdown.start - times
        guard nextNumber > 0
            else { return nil }

        times += 1
        return nextNumber
    }
}

let countdown = Countdown(start: 3)
for count in countdown {
    print("\\(count)...")
}
// Prints "3..."
// Prints "2..."
// Prints "1..."

```

Swift에서는 IteratorProtocol을 통해 직접 구현체를 만들 수 있습니다.

- Swift에서는 hasNext() 메소드는 제공하고 있지 않습니다.
- Swift에서는 nil을 이용해 java의 hasNext() 메소드를 대신합니다.

<br>

### 상태 검사 메소드, 옵셔널, 특정 값

책에서는 상태 검사 메소드 이외에 옵셔널을 사용하거나 특정 값을 사용하는 선택지도 있다고 합니다.

1. 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인으로 상태가 변할 수 있다면 **옵셔널**이나 **특정 값**을 사용합니다.( 상태 검사 메소드를 호출하는 사이에 상태가 변할 수도 있기 때문입니다. )
2. 성능이 중요한 상황에서 상태 검사 메소드가 상태 의존적 메소드 작업 일부를 중복 수행한다면 **옵셔널**이나 **특정 값**을 사용합니다.
3. 다른 모든 경우엔 상태 검사 메소드 방식이 조금 더 낫습니다.

예외는 예외 상황에서 쓸 의도로 설계되었습니다. 정상적인 제어 흐름에서 사용해서는 안되며, 이를 강요하는 API를 만들어서는 안됩니다.