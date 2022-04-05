# Generic

<aside>
💡 용어

---

- 제네릭 타입: 제네릭을 사용하는 class, structure, enumeration
- 제네릭 함수: 제네릭을 사용하는 함수
- 타입 파라미터: `Class<T>`에서 `T`
</aside>

## 제네릭 함수 오버로딩

---

- 제네릭 함수에서 특정 타입일 때만 다른 로직이 실행되도록 하고 싶을 때 제네릭 함수를 오버로딩하면 된다.
    
    ```swift
    func swapValues<T>(_ a: inout T, _ b: inout T) {
        print("generic func")
        let tempA = a
        a = b
        b = tempA
    }
     
    func swapValues(_ a: inout Int, _ b: inout Int) {
        print("specialized func")
        let tempA = a
        a = b
        b = tempA
    }
    ```
    
- 이렇게 할 경우, swapValue(1, 2) 메서드를 실행했을 때 **타입이 지정된 함수가 제네릭 함수보다 우선순위가 높아서** 오버로딩한 함수가 실행된다.

## **Ty**pe Constraints (타입 제약)

---

- 타입 제약은 특정 조건을 만족하는 경우에만 제네릭 함수나 제네릭 타입을 사용할 수 있도록 제한한다.
- 특정 조건 예시) 특정 클래스를 상속받는 타입, 특정 프로토콜을 따르는 타입

## 타입 제약 Syntax

---

```swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
}
```

- 타입 파라미터 `T`와 콜론 `:` 다음에 클래스, 프로토콜을 쓴다.
- T를 대신해서 사용할 수 있는 타입: `SomeClass`를 상속받은 클래스
U를 대신해서 사용할 수 있는 타입: `SomeProtocol`을 따르는 타입

## Associated Type

---

```swift
protocol Container {
	**associatedtype Item**
	mutating func append(_ item: Item)
	var count: Int { get } 
	subscript(i: Int) -> Item { get }
}
```

```swift
struct IntContainer: Container {
    **typealias Item = Int**
    
    var array: [Int] = []
    
    mutating  func append(_ item: Int) {
        array.append(item)
    }
    
    var count: Int {
        return array.count
    }
    
    subscript(i: Int) -> Int {
        return array[i]
    }
}
```

- Protocol에서 사용하는 제네릭 타입
- Protocol을 comfort할 때 associated type이 무엇인지 결정된다.
- `typealias` 를 사용하여 코드 상에 associated type으로 무엇을 사용할지 표현할 수 있다.

### Swift Compiler는 asssociated type을 추론할 수 있다.

- Protocol 구현체가 만들어지면, Swift Compiler는 구현체의 내용을 보고서 associatedType을 추론할 수 있다.
- 따라서 `typealias` 를 사용해서 associated type을 명시적으로 지정하지 않아도 동작한다.

### AssociatedType에도 타입 제약(Type Contraint)을 추가할 수 있다.

```swift
protocol Container {
	associatedtype Item: Equatable
	mutating func append(_ item: Item) 
	var count: Int { get }
	subscript(i: Int) -> Item { get }
}
```

- 제네릭에 타입 제약을 추가하는 방식(= 콜론 `:` 을 이용한 방식)과 동일하다.
- `Container`를 comport 하려면 `Equatable` 프로토콜을 따르는(comform) 타입을 `Item` 으로 지정해야 한다.

### Associated Type의 타입 제약으로 Associated Type이 속한 프로토콜을 사용할 수 있다.

```swift
protocol SuffixableContainer: Container {
	associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
	
	func suffix(_ size: Int) -> Suffix
}
```

- 위 예시에서 associated type `Suffix` 는 2가지 타입 제약을 갖는다.
    1. `SuffixableContainer`를 따르는(comform) 타입이어야 한다.
    2. `Suffix`의 `Item` 프로퍼티는 `Container` 의 `Item` 프로퍼티와 타입이 같아야 한다.
- 1번 타입 제약을 통해서 알 수 있는 것
    - ***associated type(`Suffix`)의 타입제약으로, associated type이 소속된 protocol(`SuffixableContainer`)을 사용할 수 있다.***
    

## 제네릭 where 절 (**Generic Where Clauses)**

---

- 타입 파라미터(`Class<T>` 에서 `T`에 해당)/associated type에 타입 제약을 주는 또다른 방법
- `where` 을 사용했을 때만 사용할 수 있는 타입 제약:
    - “타입 파라미터(`Class<T>`에서 `T` 에 해당 ), associated type이 특정 타입과 ***일치해야 한다.***”

### Syntax

```swift
func isEqual<C1: Container, C2: Container> (someContainer: C1, anotherContainer: C2) 
-> Bool 
**where C1.Item == C2.Item, C1.Item: Equatable** {
	return someContainer == anotherContainer
}
```

- `where` 키워드를 사용
- 예시 설명
    - ( `C1: Container`) `C1` 타입이 `Container` protocol을 따라야 한다.
    - ( `C2: Container`) `C2` 타입이 `Container` protocol을 따라야 한다.
    - (`C1.Item == C2.Item`) `C1`의 `Item` 타입이 `C2`의 `Item` 타입과 일치해야 한다.
    - (`C1.Item: Equatable`) `C1`의 `Item` 타입이 `Equatable` protocol 을 따라야 한다.

### extension 과 where 같이 사용하기

```swift
extension Array where Element: Equatable {
    mutating func pop() -> Element { return self.removeLast() }
}
```

- 의미:  `Array` 의  `Element` 가 `Equatable` 을 따르는 경우에만 extension으로 추가된 `pop()` 메서드를 사용할 수 있다.
- 언제 사용❓
    
    특정 조건을 만족하는 경우에만 extension으로 추가된 메서드, 프로퍼티를 사용할 수 있도록 하고 싶을 때 
    
- 예시

```swift
struct Name {
	let value: String
}

let nums = [1, 2, 3] 
let names = [ Name(value: "nylah"), Name(value: "river"), Name(value: "terry")]

nums.pop() // 컴파일 🔵, Int는 Equatable을 따른다. 따라서 pop()을 사용할 수 있다.
names.pop() // 컴파일 ❌, Name은 Equatable을 따르지 않는다. 따라서 pop()을 사용할 수 없다.

```

### 프로토콜 extenstion 에서도 where를 사용할 수 있다.

```swift
protocol Container {
	associatedtype: Item

 // do something
}

// Item의 타입이 Equatable 프로토콜을 따를 때에만 startsWith 메서드를 사용할 수 있다.
extension Container where Item: Equatable {
    func startsWith(_ item: Item) -> Bool {
        return count >= 1 && self[0] == item
    }
}

// Item의 타입이 Double과 일치 할 때만 average() 를 사용할 수 있다.
extension Container where Item == Double {
	func average() -> Double {
        var sum = 0.0
        for index in 0..<count {
            sum += self[index]
        }
        return sum / Double(count)
    }
}

print([1260.0, 1200.0, 98.6, 37.0].average()) // 컴파일 🔵
```

## ‘associated type을 선언할 때’ where를 사용하여  associated type에 타입 제약을 추가할 수 있다.

---

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }

    **associatedtype Iterator: IteratorProtocol where Iterator.Element == Item**
    func makeIterator() -> Iterator
}
```

![Untitled](Generic%20c60a1/Untitled.png)

- associated type `Iterator` 를 선언할 때, where를 사용하여 타입 제약을 (“associated type인 `Iterator` 의 타입 제약으로 Iterator의 `Element`가 `Item` 타입과 일치해야 한다.”) 선언한 것을 확인할 수 있다.

### where를 이용하여 부모 프로토콜의 associated type에 대한 타입 제약을 추가할 수 있다.

```swift
protocol ComparableContainer: Container where Item: Comparable { }
```

- 프로토콜이 또 다른 프로토콜을 따를(comform)  때, conform 하는 프로토콜을 부모 프로토콜이라고 한다.
- 예시에서 ComparableContainer 프로토콜이 Container 프로토콜을 따르고 있다.
    
    이때 ComparableContainer의 부모 프로토콜은 Container가 된다.
    
- 부모 프로토콜의 associated type에 대한 타입 제약을 추가할 수 있다.
- 예시에서는 `ComparableContainer` 가 자신의 부모 프로토콜인 `Container` 의 `Item` 에 타입 제약을 추가하고 있다.
- 제약 조건을 해석해보면, “`Item` 이 `Comaparable` 프로토콜을 따라야 한다. `Container` 의 `Item` 타입이 `Comparable`인 프로토콜이 `ComparableContainer` 이다.” 의미이다.

## Generic Subscript

---

- Subscript에서 제네릭을 사용할 수 있다.
- 타입 제약, where 둘다 사용할 수 있다.
- 예시

```swift
extension Container {
	subscript<Indices: Sequence>(indicies: Indicies) -> [Item] 
		where Indicies.Iterator.Element == Int {
		var result: [Item] = []
            for index in indices {
                result.append(self[index])
            }
            return result
	}
}
```

- subscript에서 제네릭 타입으로 `Indices` 를 사용하고 있다.
- 제네릭 타입 `Indices` 의 타입 제약
    - 타입 제약 1️⃣ (`subscript<Indices: Sequence>`)
        
        : `Indices`는 Sequence를 따라야 한다.
        
    - 타입 제약 2️⃣ (`where Indicies.Iterator.Element == Int` )
        
        : `Indicies.Iterator.Element` 는 Int 타입이어야 한다.