# Item. 41 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라. 



## 목차 

* 마커 인터페이스란? 
* 마커 어노테이션 VS 마커 인터페이스 
* 정리





## 마커 인터페이스란?

책에서 정의하기로는 

**아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스** 라고 정의합니다.



### 마커 인터페이스의 목적 

여기서 말하는 **마커 인터페이스** 는 디자인 패턴의 한 종류인 **마커 인터페이스 패턴** 으로 여겨질 수 있습니다. 

이런 **마커 패턴**을 사용하는 이유는 크게 두 가지로 볼 수 있습니다.

1. 객체에 대한 정보 구체화
2. 객체의 런타임에 대한 정보 제공 

<Br>



#### 1. 객체에 대한 정보 구체화

`Java` 에서는 

```java
public interface Describable { }

public class Student implements Describable {
  private String name;
  private int age;
  
  public Student (String name, int age) {
    this.name = name;
    this.age = age;
  }
  
  public void printDescription() {
    System.out.println(this.name + "'s" + "age is " + this.age);
  }
}
```

위와 같이 빈 `interface`를 만들어서 객체의 특성을 구체화 할 수 있습니다. 

<Br>

개발 초기에는 **`Serializable`**, **`Cloneable`** 와 같은 표현을 하기 위해 사용되었다고 합니다. 

**Swfit** 에서는 **`Codable`**, **`Decodable`**, **`Encodable`** 등이 **마커 인터페이스 패턴** 형태로 사용되곤 합니다. 

<Br>

#### 2. 객체의 런타임에 대한 정보 제공

위에 1번 **객체에 대한 정보 구체화** 가 코드의 의미적인 측면에서의 용도로 사용되었다면 

2번은 런타임에 객체 타입을 구체화하기 위해 사용됩니다. 

<Br>

```swift
protocol Encodable {}
protocol Decodable {}
typealias Codable = Encodable & Decodable

struct Student: Codable {
  private name: String
  private age: Int
}

open class JSONEncoder {
  public struct OutputFormatting : OptionSet { ... }
  
  public init() {}
  open var outputFormatting: JSONEncoder.OutputFormatting
  open func encode<T>(_ value: T) throws -> Data where T : Encodable
}


func getStudentData(_from student: Student) -> Data? { 
  let encoder: JSONEncoder = JSONEncoder()
  return encoder.encode(student)
}
// 위와 같이 Student를 직접 매개변수의 타입으로 사용하여 메소드를 만들 수도 있지만, 

func getData(_from object: Codable) -> Data? {
  let encoder: JSONEncoder = JSONEncoder()
  return encoder.encode(object)
}
// 위 처럼 Codable 이라는 마커를 이용해서 확장성을 더 넓힐 수도 있습니다. 

let gwonii: Student = .init(name: "gwonii", age: 28)
let jason: Adult = .init(name: "jason", age: 28, maritalStatus: .single)
// getData 메소드를 이용한다면 두 객체의 JSON 데이터를 얻을 수 있습니다.
```

> `Swift` 에서는 `protocol `을 이용해서 **마커 인터페이스 패턴** 을 구현합니다. 



## 마커 어노테이션 VS 마커 인터페이스 

* **마커 어노테이션** 의 어노테이션으로써 다양한 기능들을 사용할 수 있지만, **마커 인터페이스** 는 어노테이션이 제공해주는 기능들을 사용할 수 없습니다. 
* **마커 인터페이스** 는 인스턴스들을 구분하는 타입으로 사용될 수 있지만, **마커 어노테이션** 은 그렇지 않습니다.

<Br>

**마커 어노테이션** 과 **마커 인터페이스** 는 각자의 쓰임이 있습니다. 

새로 추가하는 메소드가 없이 타입 정의가 목적이라면 **마커 인터페이스** 를 사용하는 것이 좋습니다. 

반면 어노테이션을 적극 활용하여 프레임워크의 일부로 그 마커를 편입시키고자 한다면 **마커 어노테이션** 을 사용하는 것이 좋습니다. 



## 정리

`Java`의 경우 **마커 인터페이스 패턴** 을 위해서 **어노테이션**을 만들고 자주 사용한다. 하지만 `Swift` 에서는 따로 어노테이션이 

존재 하지 않기에 `protocol` 을 이용해서 직접 구현해야 합니다. 

이전에 **마커 인터페이스 패턴** 에 대해 잘 알지 못했지만 타입을 마킹하는 용도로 무의식적으로 자주 사용해왔었습니다. 

이제부터는 목표를 분명히 하여 **마커 인터페이스 패턴** 을 사용하면 코드의 퀄리티가 좋아질 것 같은 기분이 듭니다. 


