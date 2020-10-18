# Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

클래스가 내부적으로 하나 이상의 자원에 의존하고 그 자원이 클래스 동작에 영향을 줄 때의 경우 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라. 

### 정적 유틸리티와 싱글턴을 잘못 사용한 예  

사용하는 자원에 따라 동작이 달라지는 클래스에서 정적 유틸리티 클래스나 싱글턴 방식은 적합하지 않다.

1. 정적 유틸리티를 잘못 사용한 예
```swift
class SpellChecker {
  private static let dictionary = Lexicon()
  private init() { ... }
  
  static func isValid(word: String) -> Bool { ... } 
  static func suggestion(typo: String) -> [String] { ... }
}
```
2. 싱글턴을 잘못 사용한 예
```swift
class SpellChecker {
  private let dictionary = Lexicon()
  private init( ... ) {}
  
  static let sharedInstance = SpellChecker()
  func isValid(word: String) -> Bool { ... } 
  func suggestion(typo: String) -> [String] { ... }
}
```
3. 상수 dictionary을 변수로 변경하고 다른 사전으로 교체하는 메서드를 추가한 예
```swift
class SpellChecker {
  private var dictionary = Lexicon()
  private init( ... ) {}
  
  static let sharedInstance = SpellChecker()
  func isValid(word: String) -> Bool { ... } 
  func suggestion(typo: String) -> [String] { ... }
  func changeDictionary(to newDictionary: Lexicon) { ... }
}
```

  * 코드 1과 코드 2는 사전을 단 하나만 사용한다고 가정하고 구현하였다. 유연하지 않고 테스트하기 어렵다.
  * 코드 3은 멀티스레드 환경에서 사용할 수 없다. 

  **따라서 클래스가 여러 인스턴스를 지원해야하며 클라이언트가 원하는 자원을 사용해야하는 경우에는 정적 유틸리티 클래스나 싱글턴 방식을 사용하기 보다 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주자.(의존 객체 주입의 한 형태로 생성자 주입에 해당)**

### 생성자 주입을 사용한 예

1. 생성자 주입을 사용한 예 
```swift
class SpellChecker {
  private let dictionary = Lexicon()
  private init(newDictionary: Lexicon) {
    self.dictionary = newDictionary
  }
  
  func isValid(word: String) -> Bool { ... } 
  func suggestion(typo: String) -> [String] { ... }
}
```
* 코드 4는 테스트 용이성을 높여준다.
* 생성자에 팩터리 메서드 패턴(Factory Method pattern)을 이용하여 자원 팩터리(호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체)를 넘겨주는 방식으로 변형해 활용할 수 있다.

### 의존성 주입(Dependency Injection)

  * 의존성 주입(Dependency Injection)이란?

    - 의존성: 함수에 필요한 클래스 또는 참조 변수나 객체에 의존하는 것.
    - 주입: 내부에서 필요한 객체를 생성하여 참조/사용하지 않고 외부에서 객체를 생성해 넣어주는 것.
    - 의존성 주입: 코드에서 두 모듈 간의 연결. 객체지향 언어에서는 두 클래스 간의 관계라고도 한다. 

  * 의존성 주입의 장점
    1. 객체 간의 결합도(Coupling)을 낮춰 의존성을 줄여 유지보수가 용이해진다.
       - 객체 간 의존성(종속성)이 감소해 변경에 민감하지 않다.  
    2. 재사용성이 증가한다. 
    3. 리팩토링이 수월하다.
    4. Protocol을 사용하는 경우, Protocol에 구현체를 쉽게 교체하면서 상황에 따라 적절한 행동을 정의할 수 있다.
    5. 테스트가 용이하다. 
       - 주입할 의존 객체를 Mock 객체로 구현한 후 주입할 수 있다.
    6. 보일러 플레이트 코드(Boilerplate code , 꼭 필요하면서 간단한 기능에 비해 많은 코드를 필요로 하는 코드를 의미 한다. 예를 들면 setter, getter을 의미한다.) 감소시킬 수 있다.
    7. 같은 자원을 사용하려는 여러 클라이언트가 의존 객체들을 안전하게 공유할 수 있다. 
    8. 생성자, 정적 팩터리, 빌더 모두에 똑같이 응용할 수 있다.

  * DI의 단점은?
    1. 의존성 주입을 위한 선행 작업 필요하다.
    2. 코드를 추척하고 읽기 어려울 수 있다.

### 의존성 주입 방법

1. Constructor Injection 

   ```swift
   protocol Drinkable {
       var volume: Int { get }
   }
   
   struct Pepsi: Drinkable {
       var volume: Int {
           return 500
       }
   }
   
   class VendingMachine {
       let beverage: Drinkable
       
       init(_ beverage: Drinkable) {
           self.beverage = beverage
       }
   }
   
   let vendingMachine = VendingMachine(Pepsi())
   ```

2. Property Injection 

   ```swift
   protocol Drinkable {
       var volume: Int { get }
   }
   
   struct Pepsi: Drinkable {
       var volume: Int {
           return 500
       }
   }
   
   class VendingMachine {
       var beverage: Drinkable?
   }
   
   let vendingMachine = VendingMachine()
   vendingMachine.beverage = Pepsi()
   ```

3. Method Injection 

   ```swift
   protocol Drinkable {
       var volume: Int { get }
   }
   
   struct Pepsi: Drinkable {
       var volume: Int {
           return 500
       }
   }
   
   class VendingMachine {
       var beverage: Drinkable?
       
       func setupBeverage(_ beverage: Drinkable) {
           self.beverage = beverage
       }
   }
   
   let vendingMachine = VendingMachine()
   vendingMachine.setupBeverage(Pepsi())
   ```

### iOS 예시

```swift
// Property Injection 
import UIKit

class ViewController: UIViewController {

    lazy var requestManager: RequestManager? = RequestManager()

}

class newViewController {

  let viewController = ViewController()
  viewController.requestManager = RequestManager()

 }

// Using Protocol
protocol Serializer {

    func serialize(data: AnyObject) -> NSData?

}

class RequestSerializer: Serializer {

    func serialize(data: AnyObject) -> NSData? {
        ...
    }

}

class DataManager {

    var serializer: Serializer? = RequestSerializer()

}
```