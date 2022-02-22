# Closures

## 개념

- self-contained blocks of functionality that can be passed around and used in your code.
- similar to blocks in C and Objective-C and to lambdas in other programming languages.
- 정의된 context로부터의 모든 상수 및 변수에 대한 reference를 캡처하고 저장할 수 있다. → closing over those constants and variables.
    - Swift handles all of the memory management of capturing for you.

## 종류

1. 전역함수는 이름은 있지만 값을 캡처하지 않는 클로저
2. 중첩함수는 이름이 있고 enclosing 함수에서 값을 캡처할 수 있는 클로저
3. 클로저 표현식은 주변 context의 값을 캡처할 수 있는 경량 구문으로 작성된 이름 없는 클로저

## 장점

일반적으로 간결하고 혼란이 없는 최적화와 함께 깔끔하고 명확한 스타일을 제공한다.

최적화는 다음을 포함한다.

- Inferring parameter and return value types from context
- Implicit returns from single-expression closures
- Shorthand argument names
- Trailing closure syntax

# Closure Expressions

클로저 표현식은 간단하게 inline closure를 작성하는 방법이다.

클로저 표현식은 의도를 잃지 않고 짧은 형태로 클로저를 쓰기 위한 몇 가지 구문 최적화를 제공한다.

아래의 예제에서 더 알 수 있다. 

## The Sorted Method

Swift의 표준 라이브러리는 `sorted(by:)`를 제공한다. 

- `sorted(by:)` 메소드
    - 사용자가 제공한 sorting 클로저를 기반으로 타입의 값을 정렬한다.
    - 정렬이 끝나면 메서드는 이전 정렬과 동일한 유형 및 크기의 새 배열 반환
    - 같은 타입의 두 인자를 사용하는 클로저 제공
        - 첫 번째 값이 두 번째 값 앞에 나타나는지(ture) 뒤에 나타나는지(false) 여부를 나타내는 Bool 값 반환
        - Ex: (String, String) → Bool

### 예제는 sorted 메소드를 사용하여 문자열 배열을 알파벳 역순으로 정렬

```swift
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
```

### [First Way] 올바른 유형의 일반 함수를 작성하고 sorted 메소드에 인수로 전달

```swift
func backward(_ s1: String, _ s2: String) -> Bool {
	return s1 > s2
}
var reversedNames = names.sorted(by: backward)
```

→ 다소 긴 방법으로 클로저 표현식 구문을 사용한 sorting 클로저 인라인으로 쓰는 것이 좋다. 

## **Closure Expression Syntax**

클로저 표현식 구문은 다음과 같은 일반적 형식을 갖는다.

```swift
{ (parameters) -> return type in
		statements 
}
```

- 매개변수는 in-out일 수 있지만 기본값을 가질 수 없다.
- 가변 파라미터 이름을 지정한 경우 가변 파라미터를 사용할 수 있다.
- 매개변수 타입 및 리턴 타입으로 튜플을 사용할 수 있다.

### [Second Way] 클로저 표현식 버전의 sorted 메소드

```swift
reversedNames = names.sorted(by: (s1: String, s2: String) -> Bool in
	return s1 > s2
})
```

- [First Way]의 backward 함수와 파라미터 및 리턴값 선언 동일
    - `(s1: String, s2: Stirng) → Bool`
- 그러나, 클로저 표현식의 경우 매개 변수와 리턴 타입이 중괄호 바깥이 아닌 내부에 표현됨

→ 정렬 방법에 대한 전반적인 호출 동일하게 유지 

## Inferring Type From Context

Sorting 클로저는 메소드에 인수로 전달되기 때문에, 스위프트는 매개변수 타입 및 리턴 타입을 추론 가능하다. 

즉, 매개변수 타입 및 리턴 타입을 작성할 필요가 없다. 

모든 유형을 유추할 수 있으므로 반환 화살표(→)와 매개변수 주위의 괄호도 생략 가능하다.

### [Third Way] 타입 추론 사용

```swift
reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 })
```

- 함수 또는 메소드에 인라인 클로저를 전달할 때, 항상 매개변수와 리턴 유형을 추론할 수 있다.
- 원하는 경우 함수를 명시적으로 만들 수 있으며 모호함을 피할 수 있다.

## Implicit Returns from Single-Expression Closures

단일 표현식 클로저는 `return` keyword를 생략 가능하다. 

암시적으로 단일 표현식의 결과를 반환할 수 있다.

### [Fourth Way] 리턴 키워드 생략

```swift
reversedNames = names.sorted(by: { s1, s2 in s1 > s2 })
```

- sorted 메소드는 클로저가 Bool 값을 반환함을 명확히 한다. 모호함이 없으며 return 키워드를 생략할 수 있다.

## Shorthand Argument Names

스위프트는 자동적으로 인라인 클로저에 shorthand argument names를 제공한다. 

argument name을 $0, $1, $2 등으로 사용할 수 있다.

shorthand argument names를 사용하면, 클로저의 매개변수 리스트를 생략할 수 있다.

- shorthand argument names 타입은 함수 유형에서 추론
- 가장 큰 번호는 클로저가 가지고 있는 매개변수의 수를 결정
- `in` keyword 생략 가능

### [Fifth Way] Shorthand Argument Names 사용

```swift
reversedNames = names.sorted(by: { $0 > $1 })
```

- 가장 큰 번호가 $1이기 때문에 2개의 매개변수를 가지고 있는 것으로 이해

## Operator Methods

연산자만 전달하여 추론할 수 있다.

스위프트의 String 타입은 ‘>’ 연산자로 문자열별 구현을 정의하고 Bool 타입의 값을 반환한다. 

### [Sixth Way] 연산자 메소드 사용

```swift
reversedNames = names.sorted(by: >)
```

# Trailing Closures

- 함수의 마지막 인수로 긴 클로저 표현식을 함수에 전달해야 하는 경우, 후행 클로저를 사용할 수 있다.
- 후행 클로저는 함수 호출의 괄호 뒤에 쓴다.
- 첫 번째 클로저에 대한 인수 레이블을 작성하지 않는다.

```swift
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // function body goes here
}

// Here's how you call this function without using a trailing closure:

someFunctionThatTakesAClosure(closure: {
    // closure's body goes here
})

// Here's how you call this function with a trailing closure instead:

someFunctionThatTakesAClosure() {
    // trailing closure's body goes here
}
```

### [Seventh Way] 후행 클로저 사용

```swift
reversedNames = names.sorted { $0 > $1 }
```

- 후행 클로저를 사용할 때 유일한 인수로 제공되는 경우에 () 생략 가능

후행 클로저는 한 줄에 인라인으로 쓸 수 없을 정도로 충분히 길 때가 제일 유용 ex) Array의 map(_:)

## Multiple Trailing Closure

- 첫 번째 후행 클로저에 대한 인수 레이블 생략
- 두 번째 후행 클로저부터는 레이블 지정

```swift
func loadPicture(from server: Server, completion: (Picture) -> Void, onFailure: () -> Void) {
	if let picture = download("photo.jpg", from: server) {
		completion(picture)
	} else {
		onFailure()
	}
}
```

```swift
loadPicture(from: someServer) { picture in
	someView.currentPicture = pictrue
} onFailure: {
	print("Couldn't download the next picture.")
}
```

# Capturing Values

클로저는 정의된 주변 context로부터 상수와 변수를 캡처할 수 있다. 

그러면, 클로저는 상수와 변수를 정의한 범위가 더 이상 존재하지 않더라도 클로저 내부에서 참조하고 수정할 수 있다. 

캡처할 수 있는 가장 간단한 클로저는 다른 함수의 본문에 쓰여진 중첩 함수이다.

중첩 함수는 외부 함수의 인수를 캡처할 수 있으며 외부 함수 내에 정의된 상수 및 변수도 캡처할 수 있다. 

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
	var runningTotal = 0
	func incrementer() -> Int {
		runningTotal += amount
		return runningTotal
	}
	return incrementer
}
```

- 주변 context에서 runningTotal 및 amount 레퍼런스 캡처
    - 레퍼런스로 캡처하면 makeIncrementer() 호출이 종료되더라도 runningTotal 및 amount가 사라지지 않는다.
    - 다음에 runningTotal 그대로 사용 가능
        
        <aside>
        💡 스위프트 최적화에서 value가 클로저에 의해 변경되지 않고, 클로저가 생성된 후에도 변경되지 않는다면 스위프트는 value의 카피값을 캡처하고 저장할 수 있다. 
        
        또한, 스위프트는 변수가 더 이상 필요하지 않을 때 폐기하는 것과 관련된 모든 메모리 관리를 처리한다.
        
        </aside>
        

```swift
let incrementByTen = makeIncrementer(forIncrement: 10)

incrementByTen()
// returns a value of 10
incrementByTen()
// returns a value of 20
incrementByTen()
// returns a value of 30
```

# Closures Are Reference Types

위의 예제에서 `incrementByTen`는 상수이지만 runningTotal 변수는 계속 증가시킬 수 있다.

함수와 클로저가 레퍼런스 타입이기 때문이다. 

함수나 클로저를 상수 또는 변수로 지정하는 것은 실제로는 그 함수나 클로저에 대한 참조를 설정하는 것이다.

# Escaping Closures

클로저가 함수에 인자로 전해질 때 함수를 탈출한다고 하지만, 클로저는 함수가 리턴된 후에 호출된다.

클로저를 함수의 매개변수 중 하나로 사용할 때, 매개변수의 유형 앞에 `@escape` 를 작성하여 클로저를 탈출 할 수 있다. 

클로저가 탈출 할 수 있는 한 가지 방법은 함수 외부에 정의된 변수에 저장하는 것이다.

예를 들어, 비동기 연산을 하는 많은 함수들은 완료 핸들러로서 클로저 인수를 받는다. 함수는 연산을 시작한 후에 돌아오지만, 연산이 완료될 때까지는 클로저가 호출되지 않는다.

```swift
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
	completionHandlers.append(completionHandler)
}
```

- 매개변수를 @escaping으로 표시하지 않으면 컴파일 오류가 발생한다.

self를 참조하는 escaping 클로저는 그 self가 클래스의 인스턴스를 가리킨다면 특별한 고려사항이 필요하다.

escaping 클로저 안에서 self를 캡처하는 것은 강력한 참조 사이클을 만들기 쉽다.

→ 일반적으로 클로저는 암시적으로 변수를 캡처하지만, 이 경우에는 명시적이어야 한다. 

만약, self를 캡처한다면 사용할 때 self를 명시적으로 표현하거나, self를 클로저의 캡처 목록에 포함해야 한다. 

```swift
func someFunctioinWithNonescapingClosure(closure: () -> Void) {
	closure()
}

class SomeClass {
	var x = 10
	func doSomething() {
		someFunctionWithEscapingClosure { self.x = 100 }
    someFunctionWithNonescapingClosure { x = 200 }
	}
}

let instance = SomeClass()
instance.doSomething()
print(instance.x) // Prints "200"

completionHandlers.first?()
print(instance.x) // Prints "100"
```

# Autoclosures

자동 클로저는 함수에 인수로 전달되는 표현식을 래핑하기 위해 자동으로 생성되는 클로저이다.

어떤 인수도 필요하지 않고, 호출되면 그 안에 싸인 표현식의 값을 반환한다. 

명시적 클로저 대신 정규 표현식을 작성하여 함수의 매개변수 주위에 중괄호를 생략할 수 있다.
