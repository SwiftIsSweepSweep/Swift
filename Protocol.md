# Protocol

## 사용하는 이유

---

- structure, enumeration, class의 ***청사진*** 제공

## Extension과 같이 사용할 때

- Protocol의 기본 구현을 제공할 수 있다.
- Protocol의 기능을 확장할 수 있다.
    1. ex) A 클래스가 B 프로토콜을 따른다.
    2. B클래스를 확장한다.
    3. A클래스는 확장한 기능을 사용할 수 있다.
    
    ```swift
    // 더 스위프트 스러운 표현
    extension CustomStringConvertible where Self == KimFamily {
    	var description: String { return "kim" }
    }
    
    // 더 제네릭한 표현
    extension Sequence where Self.Iterator.Element == KimFaily {
    	// ....
    }
    ```
    
- 안되는 것‼️ : 상속
    
    다른 프로토콜을 상속받도록 하는 것. (프로토콜 상속은 프로토콜 선언하는 곳에서만 할 수 있다.
    

## Protocol도 static, class 프로퍼티, 메서드를 가질 수 있다.

---

```swift
protocol KimFamily {
	static var lastName : String = "김"
	class func lastNameInitial(): String 
}
```

## Mutating Function

---

- mutating function은 protocol에서도 mutating으로 표시해야 한다.
- mutating으로 선언된 함수도 class에서 구현할 때는 mutating을 붙이지 않아도 된다.

```swift
/**
 mutating으로 선언된 함수도 class에서 구현할 때는 mutating을 붙이지 않아도 된다.
 */

protocol Mutating {
    mutating func plus()
}

class MutatingClass: Mutating {
    var number: Int = 0
    
    func plus() {
        number += 1
    }
}
```

## 프로토콜에서 initalizer를 가질 수 있다.

---

### 클래스에서 프로토콜의 initializer를 구현할 때, required 키워드를 붙여야 한다.

- 이유: 부모 클래스가 따르는 프로토콜을 자식 클래스에서도 따라야 한다. 자식 클래스에서도 프로토콜의 initalizer를 꼭 갖도록 만들기 위해서.

```swift
protocol DefaultInitalizable {
    init()
}

class DefaultInitializer: DefaultInitalizable {
    required init() {}
}
```

### final로 선언된 클래스라면, required를 붙이지 않아도 된다.

```swift
final class FinalClas: DefaultInitalizable {
    init() {}
}
```

### 자식 클래스에서 intialzier를 갖는 프로토콜을 따를때, 이미 부모 클래스로부터 프로토콜의 initailizer와 동일한 것을 상속받는 다면 `required override` 를 붙여야 한다.

```swift
class SubClass: SuperClass, DefaultInitalizable {
    required override init() {}
}
```

### 프로토콜에서 failable initializer도 가질 수 있다.

- failable initialzer 구현 방식

| in 프로토콜 | failable initializer | nonfailable intializer |
| --- | --- | --- |
| 구현체 | faliable initialzer | nonfailable initializer |
|  | nonfailable initizlier | implicitly unwrapped failable initializer |

```swift
protocol FailableIntializable {
    init?()
}

class FailableIntialzier: FailableIntializable {
    required init() {}
}
```

```swift
protocol DefaultInitalizable {
    init()
}

class NonfailableInitializer: DefaultInitalizable {
    required init!() {}
}
```

<aside>
💡 implicitly unwrapped failable initialzier는 언제 사용하는 것일까?

---

1) init()?를 nil을 반환하지 않는 intializer로 오버라이딩할 때 
2) init()? → init!()로 델리게이트 할 때 

</aside>

## 프로토콜을 타입으로 사용할 수 있다.

---

- 프로토콜을 타입으로 사용할 때, 구체 타입에만 정의되어있는 것들은 사용 못한다. 오직 프로토콜에 명시된 것만 사용가능하다.
- 구체타입에만 정의되어있는 것을 사용하고 싶으면 downcasting을 해야 한다.

## Delegation 패턴

---

1. 객체의 책임을 다른 객체에게 대신해달라고 맡기는 것
2. Delegation 패턴 구현방법: 다른 객체에게 맞길 책임을 protocol로 구현한다
3. 사용예시
    1. 특정 **액션에 응답**하는 로직이 상황에 따라 달라지게 할 때
        - button listener를 구현하는 것→ button이 눌렸을 때의 로직을 개발자가 정의하게 한다.
        - = 한 객체의 상태 변화가 일어날 때, 일어나야 하는 이벤트를 delegate 패턴으로 구현한다.
    2. 외부로부터 데이터를 얻어올 때, 데이터의 출처를 몰라도 될때 
4. 사용 이유
    1. 보일러 플레이트 코드를 줄여준다.
    2. 동일한 인터페이스로 처리 로직을 다르게 할 수 있다.
    

```swift
protocol Dice
```

## 조건에 따라서 Protocol을 따르도록 만들 수 있다.

---

- where를 사용하면, 제네릭 위치에 어떤 구체 타입이 들어오느냐에 따라서, 프로토콜을 따를지 말지가 달라질 수 있다.

```swift
protocol TextRepresentable {
	var description: String { get }
} 

extension Array: TextRepresentable where Element: TextRepresentable {
	var description: String {
		let itemAsText = self.map { $0.description }
		return "["+ itemsAsText.joined(seperator: ",") + "]"
	}
}

extension Dice: TextRepresentable {
	var description: String {
		return "A \(sides)-sided dice"
	}
}

let d6 = Dice()
let d12 = Dice()
let myDice = [d6, d12]
myDice.description // 🔵

let lastNames = ["kim", "park"]
lastNames.description // ❌ 컴파일 에러 발생
```

- 예시 설명: 
Array의 요소(Element)가 TextRepresentable 프로토콜을 따를 때, Array도 TextRepresenteable을 따른다.
- 만일 프로토콜 구현체가 여러 조건을 만족한다면? Swift 컴파일러는 가장 구체적인 조건을 가진 extension을 사용한다.
    
    ```swift
    extension Array: TextRepresentable where Element: TextRepresentable {
    	var description: String {
    		let itemAsText = self.map { $0.description }
    		return "["+ itemsAsText.joined(seperator: ",") + "]"
    	}
    }
    
    extension Array: TextRepresentable where Element: Int {
    	var description: String {
    		let itemAsTest = self.map { $0.description {
    		return "number:" + itemAsText.joined(seperator: ",")
    	}
    }
    
    let numbers = [1, 2, 3, 4, 5]
    print(numbers.description) // -> 2번째 extension의 description을 사용한다.
    ```
    

## 이미 프로토콜의 구성요소를 구현하고 있는 것을 extension으로 프로토콜을 따르도록 할 때

---

- extension의 바디를 비워두어도 된다.

```swift
protocol TextRepresentable {
	var description: String { get }
} 

struct Hamster {
	var name: String
	var description: String {
		return "A hamster named \(name)"
	}	
}

extension Hamster: TextRepresentable {}
```

## Swift에서 제공하는 Equatable, Hashable, Comparable 프로토콜은 특정 조건을 만족하면 프로토콜 구현을 자동으로 해준다.

---

1. Equatable 구현을 자동으로 해주는 조건
    1. Strcuture의 경우: 모든 stored property가 Equatable을 따를 때
    2. Enumeration의 경우: asssicatedtype을 가지지 않을 때 **or** 모든 associatedtype이 Equatable을 따를때
2. Hashable 구현을 자동으로 해주는 조건
    1. Strcuture의 경우: 모든 stored property가 Hashable을 따를 때
    2. Enumeration의 경우: asssicatedtype을 가지지 않을 때 **or** 모든 associatedtype이 Hashable을 따를때
3. Comparable 구현을 자동으로 해주는 조건
    1. Enumerataion의 경우: rawValue를 갖지 않을때 **or** assciatedtype을 가지면 모든 assocatedtype이 Comparable을 따를 때

## 프로토콜 **합성**

---

- 프로토콜을 합쳐서 사용하는 것
- 형태: `SomeProtocol` & `AnotherProtocol`
- 프로토콜 합성에 Class를 같이 사용할 수 있다. 단, 1개의 클래스만 가능
- 예시) Named 프로토콜, Aged 프로토콜 합성

```swift
protocol Named {
    var name: String { get set }
}

protocol Aged {
    var age: Int { get set }
}

struct Person: Named & Aged {
    var name: String
    var age: Int
}

func wishHappyBirthday(to celebrator: Named & Aged) {
    print("Happy Birthday, \(celebrator.name), you're \(celebrator.age)")
}
```

- wishHappyBirthday는 Named와 Aged프로토콜을 ***모두 구현한*** 인스턴스를 파라미터로 받을 수 있다.
- 구체타입이 무엇인지는 상관이 없다!

<aside>
💡 `struct Person: SomeProtocol, AnotherProtocol` 과 차이점이 무엇인가?

---

- 만일 SomeProtocol과 AnotherProtocol을 구현한 타입을 파라미터로 받고 싶다고 가정하자.

```swift
func foo(person: Person)

func foo(abstract: SomeProtocol & AnotherProtocol)
```

- 첫번째 foo()는 Person 인스턴스만 인자로 받을 수 있다.
- 두번째 foo()는 Person 인스턴스 뿐만 아니라 SomeProtocol과 AnotherProtocol을 구현한 타입을 파라미터로 받을 수 있다.
    
    → 두번째 foo()를 사용해야 한다.
    
</aside>

## 특정 프로토콜을 따르는지 체크

---

- `is` : true 반환 → 특정 프로토콜을 따르는 것
- `as?` :downcasting 시도 성공 → 특정 프로토콜을 따르는 것
- `as!` : downcasting 시도 성공, 실패시 runtime error → 특정 프로토콜을 따르는 것

## Optional Requirement

---

- 프로토콜의 구성요소 중 구현해도 되고, 안해도 되는 것을 Optional Requirement라고 한다.
- 사용 방법:
    
    ```swift
    @objc protocol CounterDataSource {
    	@objc optional func increment(forCount count: Int) -> Int
    	@objc optional var fixedIncrement: Int { get }
    }
    ```
    
    - 선언할 때: `optional` 키워드 사용 & `@objc` 키워드 사용
        
        `@objc` 프로토콜은  클래스 or `@objc` 클래스만 따를 수 있다.
        
    - 사용할 때:
        - 메서드의 경우 사용할 때 `?` 표시
        - 프로퍼티의 경우 optional 타입이 된다.
        - (구현되어있지 않았을 수도 있기 때문이다.)
- 언제 사용? Objective-C하고 같이 동작해야 하는 코드를 만들 때

## 사용 예시

---

- 애플에서 권장: 다중 상속을 통해 → 기능을 조립하는 방식으로 사용
- 추상화 수단  (객체지향에서 인터페이스(자바의 그 인터페이스 아님) 구현할 때 사용)
- DIP를 구현할 때 사용
- Delegate 패턴