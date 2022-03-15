# extensions

**Extensions** 은 이미 존재하는 class, structure, enumeration, protocol에 새로운 기능을 추가하는 것이다. 

Swift Extension으로 가능한 것들은 다음과 같다.

- computed instance/type property 추가
- instance/type 메서드 추가
- 이니셜라이져 추가
- 섭스크립트 추가
- 새로운 nested types 추가
- 특정 프로토콜을 준수하도록 기능 추가

**상속과 익스텐션**

익스텐션은 타입에 새로운 기능을 추가하고, 이미 존재하는 기능을 재정의할 수는 없다. 상속은 클래스에서만 가능하지만 익스텐션은 structure, class, protocol, generic 등에서 가능하다. 

# Syntax

```swift
extension 확장할_타입_이름 {

}
```

`extension` 키워드를 사용한다. 새로운 protocol을 따르도록(conform) 추가하고 싶을 떄에는

```swift
extension 확장할_타입_이름: 프로토콜_1, 프로토콜_2 {
	// 해당 protocol들에 필요한 implementations
}
```

- 항상 file scope에서만 선언할 수 있고,
- fileprivate(또는 private) 접근제어자를 이용하여 해당 파일 내에서만 접근 가능한 프로퍼티/메서드 등을 추가할 수 있다. [*global variable/constant로 선언된 경우 private와 fileprivate는 동일하게 동작](https://stackoverflow.com/questions/39739813/private-vs-fileprivate-on-declaring-global-variables-consts-in-swift3)
- 원래 타입에 fileprivate/private로 선언된 프로퍼티에 접근할 수는 없다. 하지만 private로 선언된 경우에 한해서만, 원래 타입이 정의된 파일과 동일한 파일에 위치할 경우에 extension에서 해당 Property에 접근할 수 있다.

**private과 fileprivate**

fileprivate는 같은 파일 내에 어떤 코드에서도 접근 가능하다. 반면, private는 같은 파일이어도 다른 타입 코드는 접근 불가하고 자신을 확장하는 extension이 같은 파일에 존재할 경우 접근 가능함.

## Computed Properties

extension을 통해 computed properties를 추가할 수 있다. 다시 말하면 stored prope

- instance computed property와 type computed property(static)에 추가할 수 있고, stored property에는 추가할 수 없다.

```swift
extension Double {
	var km: Double { return self * 1_000.0 }
}
```

→ read-only 프로퍼티이기 때문에 getter만 생성해주면서 get 키워드를 생략했다.

## Method

instance method와 type method를 모두 추가할 수 있다.

```swift
extension Int {
    func repetitions(task: () -> Void) {
        for _ in 0..<self {
            task()
        }
    }
}

// usage
3.repetitions {
	print("hello")
}
```

## Initializers

**클래스에서,** 새로운 convenience initializer를 클래스에 추가할 수 있다. 하지만 designated initializer나 deinitializer를 추가할 수는 없다. 

**value-type(structure, enumeration…)에서,** 원래 타입에서 defaut 또는 memberwise initializer를 이용하고 있다면 extension initializer에서 기본/멤버와이즈 이니셜라이져를 호출할 수 있다. 즉 custom initializer가 존재하는 경우에는 extension에서 정의한 initializer에서 기본/멤버와이즈 이니셜라이져를 호출할 수 없다.


> 💡 기본 이니셜라이져는 저장 프로퍼티의 기본값이 모두 지정되어 있고, 동시에 사용자 정의 이니셜라이져가 정의되어있지 않은 상태에서 제공된다. (클래스와 구조체 모두)

> 구조체는 사용자 정의 이니셜라이저를 구현하지 않으면 프로퍼티의 이름으로 매개변수를 갖는 memberwise initializer를 기본으로 제공한다. (클래스는 안제공함) initializer 호출시 default value를 가지는 프로퍼티는 생략할 수 있다.

> 이니셜라이저에서 다른 이니셜라이저를 호출하는 것은 *initializer delegation* 이라고 부른다. value type에서 self.init은 custom initializer 내에서만 사용할 수 있다. 그리고 custom initializer를 생성하면 default/memberwise initializer는 더이상 이용할 수 없다. 

> 클래스에는 designated/convenience initializer라는 두 타입의 이니셜라이져가 존재한다. designated initializer는 primary initializer이다. 해당 클래스가 가지는 모든 프로퍼티를 초기화하고, superclass initializer를 호출한다. 

> convenience initializer는 secondary initialzier이다. 항상 다른 initializer를 내부에서 호출하게 되고 호출에서 결국은 하나의 designated initializer를 호출해야 한다.



```swift
struct Comic {
    var title: String
    let author: String
}

Comic(title: "나혼랩", author: "나혼랩작가") // 사용자 정의 이니셜라이져가 없는 경우 프로퍼티들을 모두 매개변수로 갖는 memberwise initializer가 제공됨.

struct ComicWithInitializer {
    var title: String
    let author: String
    init(comicTitle: String, comicAuthor: String) {
        title = comicTitle
        author = comicAuthor
    }
}
// Error: labels 인수가 올바르지 않습니다 -> custom initializer가 존재하는 경우 memberwise initialzier는 이용할 수 없음
// ComicWithInitializer(title: "나혼랩", author: "나혼랩 작가")
```

```swift
struct Comic {
    var title: String
    let author: String
}

Comic(title: "나혼랩", author: "나혼랩작가") // 사용자 정의 이니셜라이져가 없는 경우 프로퍼티들을 모두 매개변수로 갖는 memberwise initializer가 제공됨.

struct ComicWithInitializer {
    var title: String
    let author: String
    init(comicTitle: String, comicAuthor: String) {
        title = comicTitle
        author = comicAuthor
    }
}
// Error: labels 인수가 올바르지 않습니다 -> custom initializer가 존재하는 경우 memberwise initialzier는 이용할 수 없음
// ComicWithInitializer(title: "나혼랩", author: "나혼랩 작가")

extension Comic {
    init(title: String) {
        self.init(title: title, author: "작자 미상")
    }
}

Comic(title: "가주가 되겠습니다")

extension ComicWithInitializer {
    init(title: String) {
        self.init(comicTitle: title, comicAuthor: "작자 미상")
    }
}
```

다른 모듈에서 정의된 structure에 extension을 통해 Initializer 를 추가하는 경우, 기존의 initializer를 호출한 후에 `self` 를 이용할 수 있다.

## Subscripts

`[]` 를 통해 프로퍼티에 접근하는 것

예를 들어, array[1] 또는 dictionary[”key”]처럼 collection, list, sequence에서 멤버 element에 접근할 때 이용한다. 

```swift
extension Int {
    subscript(digitIndex: Int) -> Int {
        var decimalBase = 1
        for _ in 0..<digitIndex {
            decimalBase *= 10
        }
        return (self / decimalBase) % 10
    }
}
746381295[0]
// returns 5
746381295[1]
// returns 9
746381295[2]
// returns 2
746381295[8]
// returns 7
```

## Nested Types

```swift
extension Int {
    enum Kind {
        case negative, zero, positive
    }
    var kind: Kind {
        switch self {
        case 0:
            return .zero
        case let x where x > 0:
            return .positive
        default:
            return .negative
        }
    }
}

// intInstance.kind 로 접근 가능하다.
```

# Adding Protocol Conformance with an Extension

extension을 통해 protocol conformance를 추가할 수도 있다. 이를 위해 extension 안에서 프로토콜을 따르기 위해 필요한 프로퍼티, 메서드, 서브스크립트를 추가해야한다. 추가 프로퍼티/메서드/서브스크립트 없이 이미 프로토콜을 따르고 있는 경우에는 중괄호를 열자마자 닫아주면 된다.

```swift
protocol TextRepresentable {
    var textualDescription: String { get }
}

extension Dice: TextRepresentable {
    var textualDescription: String {
        return "A \(sides)-sided dice"
	}
}
```

- 단, Extension을 통해서는 computed property만 추가할 수 있기 때문에 stored property 대신 computed property를 추가해줬다.

## Conditinally Conforming to a protocol

타입이 *특정 조건*을 만족할 때에만 프로토콜을 conform하도록 할 수도 있다. 제네릭을 이용한 `where` 절을 이용한다.


> 👉 **generic where**: associated type이 특정 프로토콜을 따르거나, 특정 타입 파라미터가 associated type과 일치하는지의 여부를 확인한다. `where + 조건` 문법을 가진다.

```swift
import Foundation

protocol Container {
    associatedtype Item
    var items: [Item] { get }
    mutating func append(_ item: Item)
}

struct IntStack: Container {
    var items: [Int] = []
    mutating func append(_ item: Int) {
        items.append(item)
    }
}

struct StringStack: Container {
    var items: [String] = []
    mutating func append(_ item: String) {
        items.append(item)
    }
}

extension Container where Item == Int {
    var count: Int {
        items.count
    }
}

var intStack = IntStack()
intStack.append(1)
print(intStack.count)

var stringStack = StringStack()
stringStack.append("a")
//print(stringStack.count)
```

# Protocol Extensions

- extension을 통해 이를 만족하는 클래스/structure에 메서드나, initializer, subscript, computed property를 제공할 수 있다.
    
    ```swift
    protocol RandomNumberGenerator {
        var number: Double { get }
    }
    extension RandomNumberGenerator {
        func randomBool() -> Bool {
            return number > 0.5
        }
    }
    class ABC: RandomNumberGenerator { 
        var number = 0.5
    }
    
    let generator = ABC()
    print(generator.randomBool())
    ```
    

ABC 클래스가 RandomNumberGenerator를 따르기 때문에, extension을 통해 제공된 `randomBool` 함수를 이용할 수 있다. randomBool 함수를 ABC에서 구현할 수도 있다. 이러한 경우에는 ABC에 randomBool 함수가 존재하는 경우 ABC의 randomBool 함수를 이용하고, 존재하지 않는 경우에는 RandomNumberGenerator의 함수를 이용한다.

```swift
protocol MyProtocol {
    func myNumber() -> Int
}
extension MyProtocol {
    func myNumber() -> Int {
        return 1
    }
}

class MyClass: MyProtocol {}
let myInstance = MyClass()
print(myInstance.myNumber())
```

이미 프로토콜에 정의된 함수나 property를 정의할 수도 있다.
