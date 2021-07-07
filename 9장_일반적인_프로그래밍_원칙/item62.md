# Item 62 다른 타입이 적절하다면 문자열 사용을 피하라



## 문자열을 쓰지 않아야 할 사례들

<br>

### 1. 문자열은 다른 값 타입을 대신하기에 적합하지 않습니다.

입력받는 데이터가 문자열일 때만 문자열을 사용하도록 합니다.

**데이터가 수치형일 때**

```
// 잘못된 예
struct Person {
  let name: String
	let age: String
} 

let gwonii: Person = .init(name: "gwonii", age: "twenty-nine")

// 올바른 예
struct Person {
	let name: String
  let age: Int
}

let gwonii: Person = .init(name: "gwonii", age: 29)

```

**데이터가 "예/아니오" 로 구분될 때**

```
// 잘못된 예
struct Car {
  let type: CarType
  let name: String

	// 2륜, 4륜 등의 구동방식
  let drivingSystem: String
}

let dreamCar: Car = .init(type: .convertible, name: "911", drivingSystem: "2wd")

// 올바른 예
struct Car {
	let type: CarType
	let name: String
	let is4WD: Bool
}

let dreamCar: Car = .init(type: .convertible, name: "911", is4WD: false)

```

**데이터가 열거형으로 구분될 때**

```
struct Car {
	let type: CarType
	let name: String
	let drivingSystem: DrivingSystem
}

enum DrivingSystem: Customstringconvertible {
	case FF
  case FR
	case MR
	case RR
	case 4WD

	var description: String {
		switch self {
			case .FF:
				return "frontEngineFrontWheel"
			case .FR:
				return "frontEngineRearWheel"
			.
			.
			.
		}
  }
}

let dreamCar: Car = .init(type: .convertible, name: "911", drivingSystem: .FR)

```

<br>

### 2. 문자열은 혼합 타입을 대신하기에 적합하지 않다.

혼합된 데이터를 하나의 문자열로 표현하는 것은 대체로 좋지 않은 생각입니다.

```
String compoundKey = className + "#" + i.next()

```

일하면서 직접 작성하게 된 코드...

```
// LocalNotification ID
let identifier: String = "\\(commuteType.description)_\\(weekday)_\\(count)"

```

**단점**

- 직접 출력된 문자열만 봐서는 내용을 정확히 인지할 수 없습니다.
- 요소를 개별적으로 접근하려고 할 때는 불필요한 파싱 작업이 생깁니다.
- 오류의 가능성이 큽니다.

**개선방안**

```
struct LocalNotificationIdentifer {
	static func isEqualCommuteType(lhs: Self, rhs: Self) -> Bool {
		return lhs.commuteType.rawValue == rhs.commuteType.rawValue
	}

	static func isEqualWeekday(lhs: Self, rhs: Self) -> Bool {
		return lhs.weekday.rawValue == rhs.weekday.rawValue
	}

	let commuteType: CommuteType
	let weekday: Weekday
	let number: Int
}

```

이런식으로 객체를 만들어서 활용할 수 있습니다.

<br>

### 3. 문자열은 권한을 표현하기에 적합하지 않습니다.

```
public class ThreadLocal {
	private ThreadLocal() { } // 외부에서 객체 생성 불가

	public static void set(String key, Object: value);

	public static Object get(String key);
}

```

`String` 키값을 이용하여 쓰레드를 구분하고 있습니다.

위와 같은 방식은 두 클라이언트가 서로 소통하지 않고 키값을 사용하게 된다면, 두 클라이언트는 제대로 기능을 하지 못할 것입니다.

```
public class ThreadLocal {
	private ThreadLocal() { } 

	public static class Key { 
		Key() {}
	}

	public static Key getKey() {
		return new Key();	
	}

	public static void set(Key key, Object: value);
	public static Object get(Key key);
}

// ThreadLocal 클래스 자체를 Key값으로 사용하기
public final class ThreadLocal<T> {
	public ThreadLocal();
	public void set(T value);
	public T get();
}

```

위의 코드에서는 Thread 별로 고유한 키값을 맵핑시킬 수 있다. 이로써 Thread의 권한을 적절하게 표현할 수 있게 되었습니다.