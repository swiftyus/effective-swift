# Item 65. 리플렉션보다는 인터페이스를 사용하라

<br>

### 리플렉션이란?

객체를 통해 클래스 정보를 분석하는 프로그램 기법입니다.

구체적인 클래스 타입을 알지 못하더라도, 컴파일된 코드를 분석하여 역으로 클래스 정보를 알아내어 클래스를 사용할 수 있게 합니다.

리플렉션은 정적/동적으로 클래스를 생성하고 교체하는 방식으로 프레임워크의 유연성을 높히기 위해 사용된다고 합니다.

```
Class testClass = TestClass.class

// 클래스 이름을 통해서 클래스를 동적 로딩하는 메소드입니다.
Class testObejct = Class.forName(TestClass)

```

`runtime` 에 동적으로 원하는 라이브러리를 로딩할 수 있지만, 컴파일타임에는 오류를 감지할 수 없어 런타임에 에러가 발생할 수도 있습니다.

<br>

### 리플렉션의 단점

- 컴파일타임 타입 검사가 주는 이점을 사용할 수 없습니다.

동적으로 클래스를 생성하고 사용하려고 했을 때, 개발자의 실수로 없는 클래스를 생성하거나 `deprecated` 클래스를 생성하여 에러가 발생할 수 있습니다.

- 리플렉션을 이용하면 코드가 지저분하고 장황해집니다.

리플렉션을 통해 클래스 객체를 생성하거나 클래스 메소드를 사용하려고 할 때 수많은 예외처리를 해야합니다.

- 성능이 떨어집니다.

리플렉션을 통한 메소드 호출은 일반 메소드 호출보다 훨씬 느립니다.

<br>

### 리플렉션을 사용해야 하는 경우

리플렉션은 코드 분석 도구나 의존관계 주입 프레임워크에 사용됩니다.

- 리플렉션을 사용할 때는 아주 제한된 형태로만 사용되어야 단점을 피하고 이점을 얻을 수 있습니다.

<br>

### 리플렉션과 인터페이스의 사용

위에서도 알 수 있듯이 리플렉션을 사용하는 것에는 여러 패널티들이 있습니다. 그렇기 때문에 꼭 사용해야 되는 상황이라면 최소한으로 사용하고 인터페이스를 함께 사용하도록 해야 합니다.

<br>

**리플렉션으로 생성하고 인터페이스를 참조하는 경우**

```
public static void main(String[] args) {

	// Class 객체 
	Class<? extends Set<String> targetClass = null;

	try { 
		targetClass = (Class<? extends Set<String>>)
			Class.forName(args[0]);
	} catch (ClassNotFoundException e) {
		fatalError("클래스를 찾을 수 없습니다.")
	}

	// 생성자
	Constructor<? extends Set<String>> constructor = null;
	
	try {
		constructor = targetClass.getDeclaredConstructor();
	} catch (NoSuchMethodException e) {
		fatalError("메소드를 찾을 수 없습니다.");
	}

	// 인스턴스
	Set<String> instance = null;
	
	try { 
		instance = constructor.newInstance();
	} catch (IllegalAccessException e) {
		fatalError("생성자에 접근할 수 없습니다.")
	} 
	.
	.
	. // 여러 error 처리

	instance.addAll(Arrays.asList(args).sublist(1, args.length));
	system.out.println(instance);
}

```

위의 코드는 인스턴스를 생성할 때만 리플렉션을 사용하고

이후 메소드들은 리플렉션을 통해 호출하는 것이 아니라 인터페이스의 메소드를 이용하여 호출하고 있습니다.

이렇게 리플렉션의 사용은 제한적으로 하고 인터페이스를 활용하는 것이 좋습니다.

- **(Mirror Document)** **https://developer.apple.com/documentation/swift/mirror**
- NSClassFromString

