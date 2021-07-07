# Item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다 



### java 7이전 일련의 원소를 반환할 때

- Collection, Set, List ( Collection interface )
- Iterable or Array

만약 Collection 메소드를 사용할 수 없다면, Iterable 인터페이스를 사용하였습니다.

<br>

### **java 8 이후 스트림 개념이 도입된 후**

- 원소 시퀸스를 반환할 때는 스트림을 사용합니다.

하지만 for-each 로 원소들을 반복하기 원하는 경우 스트림은 적절치 않습니다.

<br>

**반환값을 선택할 때는 스트림과 반복을 고려해야 합니다**

- Stream은 for-each를 지원하지 않습니다. Stream은 Iterable 인터페이스가 정의된 것과 동일하게 작동하지만, Stream이 Iterable을 확장하지 않기 때문입니다.

<br>

### Stream<E>를 Iterable<E>로 중개해주는 어댑터

```
public static <E> Iterable<E> iterableOf(Stream<E> stream) {  
  return stream::iterator;
}
// java에서 따로 지원해주는 API는 없다.

```

위와 같은 어댑터를 이용하면 Stream 에서도 for-each 문으로 반복할 수 있게 됩니다.

<br>

### 파일을 읽을 때 Stream과 Iterable

이전 45장에서 파일을 읽을 때

- Stream 버전: `Files.lines`
- Iterable 버전: `scanner`

java에서는 `File.lines` 가 파일을 읽는 경우 중간의 예외를 자동으로 처리해주기 때문에 훨씬 유리합니다.

히지만 Iterable 버전에서는 바로 `File.lines` 을 사용할 수가 없습니다.

그래서 Iterable만 반환하는 API에서는 파일을 읽기 위해 Stream으로 처리할 수 있도록 만들어주어야 합니다.

```
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false)
}
// java에서 따로 지원해주는 API는 없다.

```

위와 같은 어댑터를 이용하면 Iterable 값들도 `File.lines` 을 사용할 수 있습니다.

<br>

### Iterable의 하위 타입이면서 stream 메소드도 제공하는 Collection 인터페이스

Collection 인터페이스는 Iterable의 하위 타입이면서 스트림을 동시에 지원합니다.

그러므로 **원소 시퀸스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는 것이 일반적으로 좋습니다.**

**주의사항**

무조건 Collection 인터페이스만 사용하여 덩치큰 시퀸스를 메모리에 올리는 실수를 범하지 말아야 합니다.

ex) 책에서는 멱함수를 예로 들며 멱함수와 같은 케이스에서 Stream과 Iterable을 모두 호환시키기 위해 Collection 인터페이스를 이용하는 것은 잘못되었다고 지적합니다.

이런 문제가 생길 때는 Collection 인터페이스이거나 하위타입을 이용하는 것이 아니라 Collection 구현체를 직접 만들어 사용하라고 합니다.

ex) `AbstractCollection` 을 사용해서 Collection 구현체를 만드는 경우 Iterable용 메서드 2개만 추가하면 기존의 Collection 인터페이스처럼 Stream과 Iterable을 모두 지원할 수 있다고 합니다. 동시에 메모리의 효율성도 살릴 수 있다고 합니다.