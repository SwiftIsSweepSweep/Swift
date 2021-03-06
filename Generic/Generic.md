# Generic

<aside>
π‘ μ©μ΄

---

- μ λ€λ¦­ νμ: μ λ€λ¦­μ μ¬μ©νλ class, structure, enumeration
- μ λ€λ¦­ ν¨μ: μ λ€λ¦­μ μ¬μ©νλ ν¨μ
- νμ νλΌλ―Έν°: `Class<T>`μμ `T`
</aside>

## μ λ€λ¦­ ν¨μ μ€λ²λ‘λ©

---

- μ λ€λ¦­ ν¨μμμ νΉμ  νμμΌ λλ§ λ€λ₯Έ λ‘μ§μ΄ μ€νλλλ‘ νκ³  μΆμ λ μ λ€λ¦­ ν¨μλ₯Ό μ€λ²λ‘λ©νλ©΄ λλ€.
    
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
    
- μ΄λ κ² ν  κ²½μ°, swapValue(1, 2) λ©μλλ₯Ό μ€ννμ λ **νμμ΄ μ§μ λ ν¨μκ° μ λ€λ¦­ ν¨μλ³΄λ€ μ°μ μμκ° λμμ** μ€λ²λ‘λ©ν ν¨μκ° μ€νλλ€.

## **Ty**pe Constraints (νμ μ μ½)

---

- νμ μ μ½μ νΉμ  μ‘°κ±΄μ λ§μ‘±νλ κ²½μ°μλ§ μ λ€λ¦­ ν¨μλ μ λ€λ¦­ νμμ μ¬μ©ν  μ μλλ‘ μ ννλ€.
- νΉμ  μ‘°κ±΄ μμ) νΉμ  ν΄λμ€λ₯Ό μμλ°λ νμ, νΉμ  νλ‘ν μ½μ λ°λ₯΄λ νμ

## νμ μ μ½ Syntax

---

```swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
}
```

- νμ νλΌλ―Έν° `T`μ μ½λ‘  `:` λ€μμ ν΄λμ€, νλ‘ν μ½μ μ΄λ€.
- Tλ₯Ό λμ ν΄μ μ¬μ©ν  μ μλ νμ: `SomeClass`λ₯Ό μμλ°μ ν΄λμ€
Uλ₯Ό λμ ν΄μ μ¬μ©ν  μ μλ νμ: `SomeProtocol`μ λ°λ₯΄λ νμ

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

- Protocolμμ μ¬μ©νλ μ λ€λ¦­ νμ
- Protocolμ comfortν  λ associated typeμ΄ λ¬΄μμΈμ§ κ²°μ λλ€.
- `typealias` λ₯Ό μ¬μ©νμ¬ μ½λ μμ associated typeμΌλ‘ λ¬΄μμ μ¬μ©ν μ§ ννν  μ μλ€.

### Swift Compilerλ asssociated typeμ μΆλ‘ ν  μ μλ€.

- Protocol κ΅¬νμ²΄κ° λ§λ€μ΄μ§λ©΄, Swift Compilerλ κ΅¬νμ²΄μ λ΄μ©μ λ³΄κ³ μ associatedTypeμ μΆλ‘ ν  μ μλ€.
- λ°λΌμ `typealias` λ₯Ό μ¬μ©ν΄μ associated typeμ λͺμμ μΌλ‘ μ§μ νμ§ μμλ λμνλ€.

### AssociatedTypeμλ νμ μ μ½(Type Contraint)μ μΆκ°ν  μ μλ€.

```swift
protocol Container {
	associatedtype Item: Equatable
	mutating func append(_ item: Item) 
	var count: Int { get }
	subscript(i: Int) -> Item { get }
}
```

- μ λ€λ¦­μ νμ μ μ½μ μΆκ°νλ λ°©μ(= μ½λ‘  `:` μ μ΄μ©ν λ°©μ)κ³Ό λμΌνλ€.
- `Container`λ₯Ό comport νλ €λ©΄ `Equatable` νλ‘ν μ½μ λ°λ₯΄λ(comform) νμμ `Item` μΌλ‘ μ§μ ν΄μΌ νλ€.

### Associated Typeμ νμ μ μ½μΌλ‘ Associated Typeμ΄ μν νλ‘ν μ½μ μ¬μ©ν  μ μλ€.

```swift
protocol SuffixableContainer: Container {
	associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
	
	func suffix(_ size: Int) -> Suffix
}
```

- μ μμμμ associated type `Suffix` λ 2κ°μ§ νμ μ μ½μ κ°λλ€.
    1. `SuffixableContainer`λ₯Ό λ°λ₯΄λ(comform) νμμ΄μ΄μΌ νλ€.
    2. `Suffix`μ `Item` νλ‘νΌν°λ `Container` μ `Item` νλ‘νΌν°μ νμμ΄ κ°μμΌ νλ€.
- 1λ² νμ μ μ½μ ν΅ν΄μ μ μ μλ κ²
    - ***associated type(`Suffix`)μ νμμ μ½μΌλ‘, associated typeμ΄ μμλ protocol(`SuffixableContainer`)μ μ¬μ©ν  μ μλ€.***
    

## μ λ€λ¦­ where μ  (**Generic Where Clauses)**

---

- νμ νλΌλ―Έν°(`Class<T>` μμ `T`μ ν΄λΉ)/associated typeμ νμ μ μ½μ μ£Όλ λλ€λ₯Έ λ°©λ²
- `where` μ μ¬μ©νμ λλ§ μ¬μ©ν  μ μλ νμ μ μ½:
    - βνμ νλΌλ―Έν°(`Class<T>`μμ `T` μ ν΄λΉ ), associated typeμ΄ νΉμ  νμκ³Ό ***μΌμΉν΄μΌ νλ€.***β

### Syntax

```swift
func isEqual<C1: Container, C2: Container> (someContainer: C1, anotherContainer: C2) 
-> Bool 
**where C1.Item == C2.Item, C1.Item: Equatable** {
	return someContainer == anotherContainer
}
```

- `where` ν€μλλ₯Ό μ¬μ©
- μμ μ€λͺ
    - (Β `C1:Β Container`) `C1`Β νμμ΄Β `Container`Β protocolμ λ°λΌμΌ νλ€.
    - (Β `C2:Β Container`) `C2`Β νμμ΄Β `Container`Β protocolμ λ°λΌμΌ νλ€.
    - (`C1.ItemΒ ==Β C2.Item`) `C1`μ `Item` νμμ΄ `C2`μ `Item` νμκ³Ό μΌμΉν΄μΌ νλ€.
    - (`C1.Item:Β Equatable`) `C1`μ `Item` νμμ΄ `Equatable`Β protocol μ λ°λΌμΌ νλ€.

### extension κ³Ό where κ°μ΄ μ¬μ©νκΈ°

```swift
extension Array where Element: Equatable {
    mutating func pop() -> Element { return self.removeLast() }
}
```

- μλ―Έ:  `Array` μ  `Element` κ° `Equatable` μ λ°λ₯΄λ κ²½μ°μλ§ extensionμΌλ‘ μΆκ°λ `pop()` λ©μλλ₯Ό μ¬μ©ν  μ μλ€.
- μΈμ  μ¬μ©β
    
    νΉμ  μ‘°κ±΄μ λ§μ‘±νλ κ²½μ°μλ§ extensionμΌλ‘ μΆκ°λ λ©μλ, νλ‘νΌν°λ₯Ό μ¬μ©ν  μ μλλ‘ νκ³  μΆμ λ 
    
- μμ

```swift
struct Name {
	let value: String
}

let nums = [1, 2, 3] 
let names = [ Name(value: "nylah"), Name(value: "river"), Name(value: "terry")]

nums.pop() // μ»΄νμΌ π΅, Intλ Equatableμ λ°λ₯Έλ€. λ°λΌμ pop()μ μ¬μ©ν  μ μλ€.
names.pop() // μ»΄νμΌ β, Nameμ Equatableμ λ°λ₯΄μ§ μλλ€. λ°λΌμ pop()μ μ¬μ©ν  μ μλ€.

```

### νλ‘ν μ½ extenstion μμλ whereλ₯Ό μ¬μ©ν  μ μλ€.

```swift
protocol Container {
	associatedtype: Item

 // do something
}

// Itemμ νμμ΄ Equatable νλ‘ν μ½μ λ°λ₯Ό λμλ§ startsWith λ©μλλ₯Ό μ¬μ©ν  μ μλ€.
extension Container where Item: Equatable {
    func startsWith(_ item: Item) -> Bool {
        return count >= 1 && self[0] == item
    }
}

// Itemμ νμμ΄ Doubleκ³Ό μΌμΉ ν  λλ§ average() λ₯Ό μ¬μ©ν  μ μλ€.
extension Container where Item == Double {
	func average() -> Double {
        var sum = 0.0
        for index in 0..<count {
            sum += self[index]
        }
        return sum / Double(count)
    }
}

print([1260.0, 1200.0, 98.6, 37.0].average()) // μ»΄νμΌ π΅
```

## βassociated typeμ μ μΈν  λβ whereλ₯Ό μ¬μ©νμ¬  associated typeμ νμ μ μ½μ μΆκ°ν  μ μλ€.

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

- associated type `Iterator` λ₯Ό μ μΈν  λ, whereλ₯Ό μ¬μ©νμ¬ νμ μ μ½μ (βassociated typeμΈ `Iterator` μ νμ μ μ½μΌλ‘ Iteratorμ `Element`κ° `Item` νμκ³Ό μΌμΉν΄μΌ νλ€.β) μ μΈν κ²μ νμΈν  μ μλ€.

### whereλ₯Ό μ΄μ©νμ¬ λΆλͺ¨ νλ‘ν μ½μ associated typeμ λν νμ μ μ½μ μΆκ°ν  μ μλ€.

```swift
protocol ComparableContainer: Container where Item: Comparable { }
```

- νλ‘ν μ½μ΄ λ λ€λ₯Έ νλ‘ν μ½μ λ°λ₯Ό(comform)  λ, conform νλ νλ‘ν μ½μ λΆλͺ¨ νλ‘ν μ½μ΄λΌκ³  νλ€.
- μμμμ ComparableContainer νλ‘ν μ½μ΄ Container νλ‘ν μ½μ λ°λ₯΄κ³  μλ€.
    
    μ΄λ ComparableContainerμ λΆλͺ¨ νλ‘ν μ½μ Containerκ° λλ€.
    
- λΆλͺ¨ νλ‘ν μ½μ associated typeμ λν νμ μ μ½μ μΆκ°ν  μ μλ€.
- μμμμλ `ComparableContainer` κ° μμ μ λΆλͺ¨ νλ‘ν μ½μΈ `Container` μ `Item` μ νμ μ μ½μ μΆκ°νκ³  μλ€.
- μ μ½ μ‘°κ±΄μ ν΄μν΄λ³΄λ©΄, β`Item` μ΄ `Comaparable` νλ‘ν μ½μ λ°λΌμΌ νλ€. `Container` μ `Item` νμμ΄ `Comparable`μΈ νλ‘ν μ½μ΄ `ComparableContainer` μ΄λ€.β μλ―Έμ΄λ€.

## Generic Subscript

---

- Subscriptμμ μ λ€λ¦­μ μ¬μ©ν  μ μλ€.
- νμ μ μ½, where λλ€ μ¬μ©ν  μ μλ€.
- μμ

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

- subscriptμμ μ λ€λ¦­ νμμΌλ‘ `Indices` λ₯Ό μ¬μ©νκ³  μλ€.
- μ λ€λ¦­ νμ `Indices` μ νμ μ μ½
    - νμ μ μ½ 1οΈβ£Β (`subscript<Indices: Sequence>`)
        
        : `Indices`λ Sequenceλ₯Ό λ°λΌμΌ νλ€.
        
    - νμ μ μ½ 2οΈβ£Β (`where Indicies.Iterator.Element == Int` )
        
        : `Indicies.Iterator.Element` λ Int νμμ΄μ΄μΌ νλ€.