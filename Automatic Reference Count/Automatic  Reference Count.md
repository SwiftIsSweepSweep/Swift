# Automatic Reference Count (ARC)

## 참고 자료

---

- Swift 공식 문서 - ARC
    
    [Automatic Reference Counting - The Swift Programming Language (Swift 5.6)](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)
    
- ccodingem - ARC
    
    [[weak self] in Swift Made Easy - Codingem](https://www.codingem.com/weak-self-in-swift/)
    

## 서론

---

- Swift는 **메모리 사용을 관리**하기 위해 ARC를 사용한다.
- ARC는 자동으로 더 이상 사용하지 않는 클래스 인스턴스의 메모리를 해제한다.
    - → 일반적으로 개발자가 메모리 관리를 할 필요가 없게 만들어 준다.
- **ARC는 레퍼런스 타입에 대해서만 동작한다.**

## ARC 동작 방식

---

- Swift를 사용해서 클래스의 인스턴스를 생성하면, 인스턴스에게 메모리가 할당된다.
- 이 메모리에는 인스턴스의 타입, 프로퍼티 값이 저장된다.
- ARC는 인스턴스가 더이상 사용되지 않을 때 메모리를 해제한다.
- ARC가 인스턴스가 더이상 사용되지 않을 때를 판별하는 방법:
    - 해당 인스턴스를 참조하는 상수, 프로퍼티, 변수의 개수(= 레퍼런스 카운트)를 센다.
    - 레퍼런스 카운트가 0일 때 해당 인스턴스가 더이상 사용되지 않는다고 판단 → 메모리 수거
    
    (= 적어도 하나의 상수, 프로퍼티, 변수가 인스턴스를 참조하면 해당 인스턴스는 사용중인 것으로 판단한다.)
    
- 일반적으로 인스턴스를 변수, 상수, 프로퍼티에 할당할 때 ‘강한 참조'(strong reference)가 생성된다고 한다.
    - 이렇듯, 인스턴스의 메모리가 해제되지 않도록 만드는 참조를 강한 참조 (strong reference)라고 부른다.
    - 이유: 인스턴스를 강하게 hold하여, 인스턴스의 메모리가 해제되지 않게 만들기 때문이다.

## 레퍼런스 카운트

---

- 변수/상수/프로퍼티가 인스턴스를 참조하면 레퍼런스 카운트가 1 증가한다.

## Strong Reference Cycle

---

- = 2개의 인스턴스가 서로를 참조하여, 서로의 레퍼런스 카운트가 0이 되지 못하게 만들고, ARC가 인스턴스의 메모리를 회수하지 않는 현상
- 예시) 프로퍼티로 서로를 참조하는 경우
    
    ![Untitled](Automatic%20%208101e/Untitled.png)
    

## Strong Reference Cycle 해결 방법

---

```swift
class Person {
	let name: String, 
	init(name: String) {
		self.name = name
	}

	var apartment: Apartment?
	deinit { print("\(name) is being deinitialized") }
}

class Apartment {
	let unit: String
  init(unit: String) { self.unit = unit }
  var tenant: Person?
  deinit { print("Apartment \(unit) is being deinitialized") }
}

var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john

john = nil
unit4A = nil
```

![Untitled](Automatic%20%208101e/Untitled%201.png)

1. weak 키워드
    - 참조하는 인스턴스가 자신보다 짧은 라이프 사이클을 가진 경우
    - 인스턴스를 참조하는 변수 앞에 weak를 붙이면, 이 변수는 인스턴스를 weak reference한다.
    - it’s appropriate for an apartment to be able to have no tenant at some point in its lifetime, and so a weak reference is an appropriate way to break the reference cycle in this case.
2. unowned 키워드
    
    상대방 인스턴스가 자신보다 길거나 같은 라이프 사이클을 가진 경우 
    

### weak **References**

- weak reference는 참조하는 인스턴스를 강하게 hold하지 않는다.
- 따라서 weak reference로 참조하는 인스턴스는 자신에 대한 참조가 존재하더라도, ARC에 의해서 수거될 수 있다.
- **weak reference로 선언된 값은 deallocated되었을 때 nil로 바뀌어야 하므로, 
1) var
2) optional type**
- weak reference 로 선언된 값이 ARC에 의해서 nil값이 되면, property observer는 call 받지 않는다.

### 위의 예시를 weak reference로 해결했을 때 참조 그래프

- 코드
    
    ```swift
    class Person {
        let name: String
        init(name: String) { self.name = name }
        var apartment: Apartment?
        deinit { print("\(name) is being deinitialized") }
    }
    
    class Apartment {
        let unit: String
        init(unit: String) { self.unit = unit }
        **weak var tenant: Person?**
        deinit { print("Apartment \(unit) is being deinitialized") }
    }
    
    var john: Person?
    var unit4A: Apartment?
    
    john = Person(name: "John Appleseed")
    unit4A = Apartment(unit: "4A")
    
    john!.apartment = unit4A
    unit4A!.tenant = john
    
    john = nil
    // Prints "John Appleseed is being deinitialized"
    
    ```
    
- Apartment 인스턴스는 person 인스턴스를 약한 참조한다.
    
    ![Untitled](Automatic%20%208101e/Untitled%202.png)
    
- = 변수 `john` 에 nil을 할당함으로써, `john`이 가지고 있었던 strong reference를 없애면, 
Person 인스턴스에 대한 참조가 없어진다.
- 결과: Person 인스턴스에 대한 strong reference 가 존재하지 않기 때문에, 
Person 인스턴스는 deallocated 된다.
**그리고 tenant 프로퍼티 값은 nil이 된다.**
    
    ![Untitled](Automatic%20%208101e/Untitled%203.png)
    

### unowned reference

- unowned refererence도 참조하는 인스턴스를 강하게 hold하지 않는다.
    
    = unowned referencce로 참조하는 인스턴스는, 자신에 대한 참조가 존재하더라도  ARC에 의해서 수거될 수 있다.
    
- weak reference와의 차이점
    1. unowned reference는 상대 인스턴스가 자신보다 생명주기가 같거나 긴경우에 사용한다.
    2. 항상 값을 가지는 것으로 인식된다.
    → unowned로 참조하는 인스턴스의 타입은 optional 아니어도 된다.
    **→ ARC는 unowned로 참조하는 인스턴스의 값을 절대 nil로 설정하지 않는다.**
- unowned reference로 참조하는 프로퍼티, 변수 선언방법
    
    ```swift
    unowned var name: String
    ```
    
- **사용법, 사용할 때 주의사항: 절대 deallocated되지 않는 인스턴스를 참조할 때  사용**
    - b) 만일 unowned reference로 참조하는 인스턴스가 deallocated 된 후에, 동일한 인스턴스의 value에 접근하면 run time exception 발생

### unowned reference로 strong reference cycle 해결

```swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
    deinit { print("\(name) is being deinitialized") }
}

class CreditCard {
    let number: UInt64
    unowned let customer: Customer
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { print("Card #\(number) is being deinitialized") }
}

var john: Customer?

john = Customer(name: "John Appleseed")
john!.card = CreditCard(number: 1234_5678_9012_3456, customer: john!)

john = nil
// Prints "John Appleseed is being deinitialized"
// Prints "Card #1234567890123456 is being deinitialized"
```

- unowned를 사용한 이유: Customer는 CreditCard를 가지지 않을 수 있지만, CrdietCard는 항상 자신을 사용하는 Customer가 존재함 (Customer의 생명주기가 더 길다.)
    - → Customer 클래스는 CreditCard를 optional 타입으로 갖는다.
    - → CrdeitCard는 Customer를 unowned reference로 참조한다.
1. 초기 상태
    
    ![Untitled](Automatic%20%208101e/Untitled%204.png)
    
2. 결과: Customer 인스턴스, CreditCard인스턴스 둘다 메모리에서 사라진다.
- 정리: `unowned` 키워드를 사용하여 참조한 인스턴스가 deallocated가 되면, 해당 인스턴스에 대한 참조도 사라진다.

<aside>
📔 safe unowned reference ↔ unsafe unonwed reference

---

위에서 사용한 것은 safe unowned reference 이다. unsafe unowned refenrence도 존재한다.

- unsafe unowned reference 사용하는 경우: 퍼포먼스 최적화 등을 이유로 runtime safety check을 하지 않고 싶을 때
- 사용법: `unowned(unsafe)` 키워드 사용
- 주의 사항: unsafe unowned reference로 참조하는 인스턴스가 deallocated 되었을 때, 해당 인스턴스를 참조하면
</aside>

### **Unowned Optional References**

- = optional & unowned reference
- weak reference와 같은 맥락에서 사용될 수 있다.
    - weak reference: ARC가 nil값으로 자동으로 설정해준다.
    ↔ unowned optional reference를 사용할 때는. 프로그래머가 직접 unowned optional reference로 참조하는 인스턴스가 유효한 객체이거나, 혹은 nil이 되도록 만들어주어야 한다.
- unowned reference: nil값을 가질 일이 없다.
↔ unowned optional reference는 nil 값을 가질 수 있다.
- ⇒ 핵심 unowned optional reference로 참조하는 객체가 항상 유효하도록 만들어주저야 하고, 
더이상 유효하지 않으면 nil값으로 바꾸어 주어야 한다.
- 예시
    
    ![Untitled](Automatic%20%208101e/Untitled%205.png)
    
    - Like non-optional unowned references, you’re responsible for ensuring that `nextCourse` always refers to a course that hasn’t been deallocated. 
    **In this case, for example, when you delete a course from `department.courses`
     you also need to remove any references to it that other courses might have.**
    - = `nextCourse` 가 현재 unowned optional reference로 참조하는 값
    - `department.courese` 에서 특정 course 하나를 더이상 참조하지 않는다면,
    이 course는 reference count가 0이 된다. 따라서 메모리에서 deallocated된다.
    - 메모리에서 deallocated된 `nextCourse` 로 갖는 Course 인스턴스가 존재한다면, 이 인스턴스의 `nextCourse` 를 nil로 설정해주어야 한다.

<aside>
📔 value type은 Unowned 를 붙일 수 없지만, Optional(enum 타입 = value type) 은 예외이다.

</aside>

### Unowned reference ➕ Unwrapped Optional Properties (❗️)

---

- 사용하는 경우
    1. **두 인스턴스가 모두 값을 가진다.** 
    2. **초기화된 이후로  nil이 될 일이 없다.**
    - → refenence cycle이 발생하지 않으면서, 직접 접근해서 사용할 수 있다.
- 예시
    
    ```swift
    class Country {
        let name: String
        var capitalCity: City!
        init(name: String, capitalName: String) {
            self.name = name
            self.capitalCity = City(name: capitalName, country: self)
        }
    }
    
    class City {
        let name: String
        unowned let country: Country
        init(name: String, country: Country) {
            self.name = name
            self.country = country
        }
    }
    
    var country = Country() 
    country = nil
    ```
    
    - Country의 initializer에서 City가 무조건 생성되게 만들 때 사용
    - Country
    - Coutnry의 capitalCity를 non optional value 처럼 접근해서 사용할 수 있고, strong reference cycle도 피할 수 있다.
- strong reference cycle을 피할 수 있는 이유?
    - 현재 상황: Country → City, City → Country를 참조하고 있는 상황
    - Country를 deallocated하면, city도 같이 사라진다.

## Strong Reference Cycle for Closures

---

- 발생하는 경우:
    - closure 내부에서 closure 바깥의 클래스 인스턴스를 참조하는 경우
    - b) closure는 자신이 사용하는 모든 레퍼런스 타입을 강한 참조 한다.
- 예시) 클래스의 property로 closure를 사용, 이 클로저가 self의 프로퍼티 및 메서드 사용

```swift
class HTMLElement {
	let name: String
	let text: String

	lazy var asHtml: () -> String {
		if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
    } else {
            return "<\(self.name) />"
    }
	}

	init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
  }

  deinit {
    print("\(name) is being deinitialized")
  }
}
```

- 💡 asHtml을 lazy로 선언한 이유
    
    프로퍼티의 디폴트 값에서는 self를 사용하지 못한다.
    
    lazy 로 선언할 경우, initializtion이 끝난 후에 접근할 수 있기 때문에, self에 접근할 수 있다.
    
- HTMLElement의 인스턴스와 asHtml에 할당되는 closure 간 reference cycle 발생
    
    ![Untitled](Automatic%20%208101e/Untitled%206.png)
    

### Closure를 사용할 때 발생하는 Reference Cycle 해결 방법 = capture list 만들기

- capture list란?
    1. closure 에서 레퍼런스 capture할 때 사용하는 규칙을 명시하는 list
    = closure에서 참조하는 레퍼런트 타입을 어떻게 참조할 것인지 나타내는 것 
    = weak reference? unowned reference
    2. 레퍼런스를 weak 혹은 unowned로 capture해둘 수 있다.
- 사용 방법
    
    ```swift
    class Big {
    
    	lazy var someClosure = { [unowned self, weak delegate = self.delegate] (index: String, stringToProcess: String) -> String in 
    		// closure body
    	}
    }
    ```
    

## Weak & Unowned Reference

---

1. closure에서 unowned reference로 레퍼런스를 캡처하면 캡처한 reference와 클로저는 항상 서로를 참조한다. 그리고 메모리에서 사라질 때, closure와 closure에서 unowned reference로 참조하는 인스턴스는 같이 사라진다.
    - 왜냐하면  unowned reference는 자신보다 생명주기가 긴 인스턴스를 참조할 때 사용하기 때문이다.