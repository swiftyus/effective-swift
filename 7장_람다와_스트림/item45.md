# Item 45. 스트림은 주의해서 사용해라



스트림 파이프라인은

`소스 스트림` - `중간 연산` - `종단 연산` 으로 진행됩니다.

### 중간 연산

- map
  - mapToInt → IntStream
  - mapToLong → LongStream
  - ...
- flatMap
- filter

...

중간의 연산은 스트림을 어떠한 방식으로 변환합니다.



### 스트림의 장점

1. 코드 길이가 짧습니다.
2. 코드가 선언적이고 깔끔해집니다.

### 스트림의 단점

1. 잘못 사용하면 읽기 어렵습니다.
2. 유지보수가 어려워질 수 있습니다.

스트림을 사용하는 경우

**Java**

```
public class Anagrams {
  public static void main(String[] args) throws IOException {
    Path dictionary = Paths.get(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    try (Stream<String> words = Files.lines(dictionary) {
      words.collect(
        groupingBy(word -> word.chars().sorted()
                    .collect(StringBuilder::new,
                      (sb, c) -> sb.append((char) c),
                      StringBuilder::append).toString()
                  )
      )
      .values().stream()
      .filter(group -> group.size() >= minGroupSize)
      .map(group -> group.size() + ": " + group)
      .forEach(System.out.println);
    }
  }
}

```

**Swift**

```
class Anagrams {
    typealias Group = [String: [String]]
    
    private let words: [String]
    private let minGroupSize: Int = 5
    
    init(from source: [String] = []) {
        self.words = source
    }
    
    func invoke() {
        Group.init(grouping: words, by: { $0.lowercased().sorted(by: { $0 > $1 }).description })
            .map { $0.value }
            .filter { $0.count <= minGroupSize }
            .map { "\\($0.count): \\($0)" }
            .forEach { print($0) }
    }
}

let anagram: Anagrams = .init(from: ["abc", "bac", "ddd", "dde"])
anagram.invoke()

// result
// 1: ["ddd"]
// 2: ["abc", "bac"]
// 1: ["dde"]

```



### 람다 사용시 주의점

- **람다 매개변수의 이름은 주의해서 정해라**

람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지됩니다.

- **도우미 메소드를 잘 활용하라**

도우미 메소드를 잘 활용하면 반복적인 코드들을 확실하게 줄이면서 가독성도 높힐 수 있다.

- **스트림과 반복문을 적절히 조합하라**

람다에서는 final이거나 final 객체만 읽을 수 있기 때문에, 지역변수를 수정할 수 없습니다.

람다에서는 `return`, `break`, `continue` 와 같은 키워드를 사용할 수 없습니다.

**수정된 코드**

```
class Anagrams {
    typealias Group = [String: [String]]
    
    private let words: [String]
    private let minGroupSize: Int = 5
    
    init(from source: [String] = []) {
        self.words = source
    }
    
    func invoke() {
        Groups.init(grouping: words, by: { (word) in
            word.lowercased().sorted(by: { $0 > $1 }).description
        })
        .map { (groups) -> [String] in groups.value  }
        .filter { (group) in group.count <= minGroupSize }
        .map { (group) in "\\(group.count): \\(group)" }
        .forEach { print($0) }
    }
}

```

- 필요할 때는 인자에 이름을 붙이고 그렇지 않을때는 `$0` `$1` 등으로 간단하게 표기합니다.

### 스트림을 사용하기 좋은 환경

1. 원소들의 시퀸스를 일관되게 변환합니다.
2. 원소들의 시퀸스를 필터링합니다.
3. 원소들의 시퀸스를 하나의 연산을 사용해 결합합니다.
4. 원소들의 시퀸스를 컬렉션에 모읍니다.
5. 원소들의 시퀸스에서 특정 조건을 만족하는 원소를 찾습니다.