# Access Control

생성일: 2022년 7월 25일 오후 10:42

[Access Control - The Swift Programming Language (Swift 5.7)](https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html)

# Access Control

- 특정 코드의 접근을 다른 소스파일이나 모듈에서 제한하는 것
- 세부적인 구현은 감추고 필요한 만큼 공개하여 다른 곳에서 사용 가능
- 개별 타입(individual types)(클래스, 구조체, 열거)만 아니라, 해당 타입에 속하는 프로퍼티, 메소드, 이니셜라이저 및 첨자(subscripts)에 대해 특정 접근 레벨을 지정 가능
- Swift는 다양한 수준의 접근 제어 외에도 일반적인 시나리오에 대한 기본 접근 레벨을 제공
    - 명시적 접근 제어 레벨을 지정할 필요성을 줄여줌

<aside>
💡 접근 제어를 적용할 수 있는 코드의 다양한 측면(프로퍼티, 타입, 함수 등)을 간단하게 `엔티티` 라고 한다.

</aside>

# **Modules and Source Files**

- Swift의 접근 제어는 모듈과 소스파일에 기반을 두고 있다.
- 모듈: 코드를 배포하는 단일 단위로 하나의 프레임워크(UIKit 등)나 앱 / import를 사용하여 가져올 수 있다.
- 소스파일: 모듈 내의 단일 Swift 소스 코드 파일

# **Access Levels**

다섯 가지 접근 레벨 존재

- Open / Public : 다른 모듈에서 사용 가능
    - Open은 클래스 및 클래스 구성원에게만 적용 가능
    - Open은 다른 모듈에서 오버라이드와 서브클래싱이 가능하지만, Public은 불가능 (Public은 정의된 모듈 내에서만 서브 클래싱, 오버라이드 가능)
    - 클래스를 Open으로 표시하면 다른 모듈의 코드가 미치는 영향을 고려해서 코드를 설계했다는 것을 나타냄
- Internal : 기본 접근레벨로 해당 모듈 전체에서 사용 가능
- File-private: 특정 엔티티를 선언한 파일 안에서만 사용 가능
- Private: 특정 엔티티가 선언된 괄호 안에서만 사용 가능

<aside>
💡 접근 레벨은 위 → 아래 순으로 높다(제약이 없는) → 낮다(제약이 많은) 순

</aside>

## **접근레벨 가이드 원칙 (Guiding Principle of Access Levels)**

일반 가이드 원칙을 따른다.

→ 더 낮은(더 제한적) 레벨을 갖고 있는 다른 엔티티에 특정 엔티티를 선언해 사용할 수 없다.

- public 변수는 다른 internal, file-private 혹은 private 타입에서 정의될 수 없다. 왜냐하면 그 타입은 public 변수가 사용되는 모든 곳에서 사용될 수 없을 것이기 때문에.
- 함수는 그 함수의 파라미터 타입이나 리턴 값 타입보다 더 높은 접근 레벨을 가질 수 없다. 왜냐하면 함수에는 접근 가능하지만 파라미터에 접근이 불가능 하거나 혹은 반환 값 타입보다 접근 레벨이 낮아 함수를 사용하는 관련 코드에서 이용할 수 없을 수 있기 때문에.

## **기본 접근레벨 (Default Access Levels)**

아무런 접근 레벨을 명시하지 않은 경우 internal을 갖게 된다.

## **단일 타겟 앱을 위한 접근레벨 (Access Levels for Single-Target Apps)**

단일 타겟 앱에서는 특별히 접근레벨을 명시할 필요가 없지만 필요에 따라 `file-private`, `private`등을 사용해 앱내에서 구현 세부사항을 숨길 수 있다.

## **프레임워크를 위한 접근레벨 (Access Levels for Frameworks)**

프레임워크를 개발한다면 `public`혹은 `open`으로 지정해서 다른 모듈에서 볼 수 있고 접근 가능하도록 만들어야 한다.

<aside>
💡 프레임워크의 구체적 구현을 감추고 싶은 부분이 있다면 그 부분은 `internal`로 선언하면 된다. 그리고 노출 시키고 싶은 API만 `public` 혹은 `open`으로 지정한다.

</aside>

## **유닛테스트 타겟을 위한 접근레벨 (Access Levels for Unit Test Targets)**

기본적으로 `open`이나 `public`으로 지정된 엔티티만 다른 모듈에서 접근 가능하다. 하지만 유닛테스트를 하는 경우 모듈 import 앞에 `@testable`이라는 에트리뷰트를 붙여주면 해당 모듈을 테스트가 가능한 모듈로 컴파일해 사용****

# **접근제어 문법 (Access Control Syntax)**

```swift
public class SomePublicClass {}
internal class SomeInternalClass {}
fileprivate class SomeFilePrivateClass {}
private class SomePrivateClass {}

public var somePublicVariable = 0
internal let someInternalConstant = 0
fileprivate func someFilePrivateFunction() {}
private func somePrivateFunction() {}
```

- `internal` 접근레벨은 생략할 수 있다.

# **커스텀 타입 (Custom Types)**

- 커스텀 클래스에 특정 접근레벨 지정 가능

→ 특정 타입의 접근 레벨 지정을 하는 것은 그 타입 멤버(프로퍼티, 메소드 초기화, 서브스크립트)의 접근 레벨에 영향을 미친다.

- 예시: file-private 클래스를 정의했다면, file-private 클래스가 정의된 소스 파일 안에서 프로퍼티, 또는 함수 매개변수, 반환값을 사용할 수 있다.

<aside>
💡 public 타입은 기본적으로 internal 멤버를 갖는다.
→ public API를 만들시 노출되지 말아야 할 API가 실수로 노출되는 것을 막기 위해서이다. 
그래서 노출시키고 싶은 멤버는 명시적으로 `public` 접근제어자를 붙여줘야 한다.

</aside>

```swift
public class SomePublicClass {                  // explicitly public class
    public var somePublicProperty = 0            // explicitly public class member
    var someInternalProperty = 0                 // implicitly internal class member
    fileprivate func someFilePrivateMethod() {}  // explicitly file-private class member
    private func somePrivateMethod() {}          // explicitly private class member
}

class SomeInternalClass {                       // implicitly internal class
    var someInternalProperty = 0                 // implicitly internal class member
    fileprivate func someFilePrivateMethod() {}  // explicitly file-private class member
    private func somePrivateMethod() {}          // explicitly private class member
}

fileprivate class SomeFilePrivateClass {        // explicitly file-private class
    func someFilePrivateMethod() {}              // implicitly file-private class member
    private func somePrivateMethod() {}          // explicitly private class member
}

private class SomePrivateClass {                // explicitly private class
    func somePrivateMethod() {}                  // implicitly private class member
}
```

## **Tuple Types**

튜플에서 사용되는 모든 타입의 접근레벨 중 가장 제한적인 접근레벨을 갖는다.

예를 들어, 만약 하나는 `internal` 다른 하나는 `private` 접근 권한을 갖는 2개의 타입으로 구성된 튜플은 더 낮은 레벨인 `private` 접근 레벨을 갖습니다.

<aside>
💡 튜플은 자체적으로 접근레벨을 선언하지 않고 사용하는 클래스, 구조체, 열거형 그리고 함수 등에 따라 자동으로 최소 접근레벨을 부여 받는다. 즉, 튜플은 명시적으로 접근권한을 명시하지 않는다.

</aside>

## **Function Types**

- 함수 타입의 접근 레벨은 파라미터 타입과 리턴 타입의 접근레벨 중 가장 제한적인 접근레벨로 계산돼 사용
- 그것에 맞는 접근 레벨을 함수 앞에 명시해 줘야 한다.

```swift
func someFunction() -> (SomeInternalClass, SomePrivateClass) {
    // function implementation goes here
}
```

- 명시적인 접근 레벨을 명시하지 않은 함수로 컴파일시 에러가 발생
- 반환값 중에 접근레벨이 `private`인 `SomePrivateClass`가 존재하기 때문에 `someFunction()`
은 `internal` 접근레벨로 선언될 수 없다.

```swift
private func someFunction() -> (SomeInternalClass, SomePrivateClass) {
    // function implementation goes here
}
```

- 함수 앞에 `private` 접근레벨을 지정해 줘야 한다.

## **Enumeration Types**

열거형에서 각 case는 enum의 접근레벨을 따르고 개별적으로 다른 접근레벨을 지정할 수 없다.

```swift
public enum CompassPoint {
    case north
    case south
    case east
    case west
}
```

- 각 case는 enum의 접근레벨 `public`을 따라 모두 `public`접근레벨을 갖는다.

### **Raw Values and Associated Values**

열거형 내의 고유값과 연관값을 사용하는 경우, 반드시 열거형보다 같거나 높은 접근 레벨을 가져야 한다.

즉, `internal` 접근 레벨을 갖고 있는 열거형 타입에서 `private` 접근 레벨을 갖는 고유값을 사용할 수 없다.****

## **Nested Types**

- `private`로 선언된 타입의 중첩 타입은 자동으로 `private`접근레벨을 갖는다.****
- `file-private`으로 선언된 경우 중첩타입은 `file-private`을 갖는다.
- `public` 혹은 `internal`로 선언된 타입에서 중첩타입은 자동으로 `internal`접근레벨을 갖는다.
    - `public`으로 선언된 타입에서 `public`으로 선언된 중첩타입을 사용하고 싶으면 명시적으로 `public`접근자를 중첩타입에 적어줘야 한다.

# **Subclassing**

- 서브클래스는 슈퍼클래스보다 더 높은(덜 제한적인) 접근레벨을 가질 수 없다.
- 예시: 슈퍼클래스가 `internal` 를 갖는데 그것을 서브클래싱해서 `pubic` 서브클래스를 만들 수 없다.
- 하지만 메소드는 서브클래스에서 더 높은 접근 레벨을 갖는 메소드로 오버라이드 할 수 있다.

```swift
public class A {
    fileprivate func someMethod() {}
}

internal class B: A {
    override internal func someMethod() {}
}
```

- class B의 someMethod()는 슈퍼클래스의 `fileprivate`보다 더 높은 접근레벨인 `internal`을 갖도록 오버라이드 됐다.

이게 가능한 이유는 class A와 class B가 같은 소스파일에 선언돼 있어서 class B의 someMethod 구현에서 `super.someMethod()`를 호출하는 것이 유효하기 때문이다.

```swift
public class A {
    fileprivate func someMethod() {}
}

internal class B: A {
    override internal func someMethod() {
        super.someMethod()
    }
}
```

# **Constants, Variables, Properties, and Subscripts**

상수, 변수, 프로퍼티 등은 그 타입보다 더 높은 접근레벨을 가질 수 없다. 

즉, `private` 타입에서 `public` 프로퍼티를 선언할 수 없다. 

아래와 같이 `private` 클래스 변수의 접근레벨은 `private`이 돼야 한다.

```swift
private class SomePrivateClass {}

private var privateInstance = SomePrivateClass() // 컴파일 에러 미발생
public var privateInstance = SomePrivateClass() // 컴파일 에러 발생
```

## **Getters and Setters**

상수, 변수, 프로퍼티 그리고 서브스크립트의 게터와 세터는 자동으로 해당 상수, 변수, 프로퍼티 그리고 서브스크립트가 갖는 접근레벨을 동일하게 갖는다. 

필요에 따라 세터의 접근레벨을 게터보다 낮게 정할 수 있다.

- `fileprivate(set)`, `private(set)`, `internal(set)` 키워드 사용
- 이 규칙은 저장 프로퍼티 뿐 아니라, 연산 프로퍼티에도 동일하게 적용

```swift
struct TrackedString {
    private(set) var numberOfEdits = 0
    var value: String = "" {
        didSet {
            numberOfEdits += 1
        }
    }
}

var stringToEdit = TrackedString()
stringToEdit.value = "This string will be tracked."
stringToEdit.value += " This edit will increment numberOfEdits."
stringToEdit.value += " So will this one."
print("The number of edits is \(stringToEdit.numberOfEdits)")
// Prints "The number of edits is 3"
```

- 게터만을 이용해 변경된 숫자 확인 가능
- 세터에 접근레벨이 지정되지 않았다면 TrackedString이 internal이기 때문에 외부에서도 접근 가능

```swift
public struct TrackedString {
    public private(set) var numberOfEdits = 0
    public var value: String = "" {
        didSet {
            numberOfEdits += 1
        }
    }
    public init() {}
}
```

- 다음과 같은 방식으로 게터 레벨도 설정 가능

# **Initializers**

초기자의 접근레벨은 타입의 레벨과 같거나 낮다(더 제한적). 

- 한 가지 예외: 지정 초기자(required Initializer) - 반드시 속한 타입과 접근레벨이 같아야 한다.

## Default Initializers

- 타입의 접근레벨이 public이 아니면 타입과 같은 접근레벨을 갖는다.
- 타입의 접근레벨이 public이면 기본 초기자는 internal 접근레벨을 갖는다.

## Default Memberwise Initializers for Structure Types

- 모든 저장 프로퍼티가 private일 경우, Default Memberwise Initializers의 접근레벨은 private
- 모든 저장 프로퍼티가 file private일 경우, Default Memberwise Initializers의 접근레벨은 file private
- 만약 public 구조체가 다른 모듈에서 사용될 때는 반드시 Memberwise Initializers를 타입을 정의해 public으로 지정해야 한다.

# **Protocols**

- 프로토콜에 명시적 접근레벨을 할당하려면, 프로토콜이 정의될 때 할당해라.
    
    이 방식으로 특정 접근레벨 내에서만 채택할 수 있는 프로토콜을 만들 수 있다.
    
- 프로토콜 정의 내의 각 요구사항 접근제한 수준은 프로토콜과 동일한 수준으로 자동 설정
- 프로토콜이 지원하는 제한레벨과 다른 제한레벨 수준으로 설정할 수 없다.
- 이렇게 하면 프로토콜을 채택한 어떤 타입에서도 프로토콜의 요구사항을 볼 수 있다.

<aside>
💡 public 프로토콜을 정의하는 경우, 프로토콜 요구사항을 구현할 때, 해당 요구사항에 대한 public 접근레벨을 요구한다. 이 동작은 public 타입으로 정의될 때 그 타입의 멤버가 internal이던 다른 타입과는 다르다.

</aside>

## **Protocol Inheritance**

- 기존 프로토콜로부터 상속한 새로운 프로토콜을 정의한다면, 새로운 프로토콜은 상속받은 프로토콜과 거의 같은 접근 레벨을 갖는다.
- 예시: internal 프로토콜로부터 상속하는 public 프로토콜은 불가능하다.

## **Protocol Conformance**

- 타입은 타입 자체보다 더 낮은 접근 레벨(더 제한적)으로 프로토콜을 준수할 수 있다.
    - 예시: 다른 모듈에서 사용되는 public 타입을 정의할 수 있지만 internal 프로토콜 준수는 internal 프로토콜의 정의 모듈 내에서만 사용된다.
- 특정 프로토콜 타입 준수에서 컨텍스트는 타입의 접근 수준과 프로토콜의 접근 수준의 최소이다.
    - 예시: 타입이 public이라면 프로토콜은 internal로 준수되며, 타입의 프로토콜에 준수는 internal이 된다.
- 프로토콜을 준수하는 타입을 확장하거나 작성할 때, 각각의 프로토콜 요구사항의 타입 구현이 적어도 프로토콜을 준수하는 타입으로서 같은 접근 수준을 가져야 함을 보장해야 한다.
    - 예시: 만약 public 타입은 internal 프로토콜을 준수한다면, 각각의 프로토콜 요구사항의 타입 구현은 적어도 “internal”이 되어야 한다.

# **Extensions**

- extension으로 추가된 모든 타입 멤버는 원래 형식에서 선언된 타입 멤버들과 동일한 기본 접근 레벨을 갖는다.
    - 예시: public 타입을 확장한다면, 새로운 타입 멤버는 internal의 기본 접근 수준을 가지게 된다.
    - 예시: file-private 타입을 확장한다면, 새로운 타입 멤버는 file-private 기본 접근 수준을 가지게 된다.
- extension에 명시적으로 접근 레벨을 표시하여 extension 내에 정의된 멤버들에게 새로운 기본 접근 레벨을 세팅할 수 있다.
- 새로운 기본 접근 수준은 여전히 개별 타입 멤버에 대한 extension 내에서 오버라이드 될 수 있다.
- extension을 사용하여 프로토콜을 준수하는 경우, 접근 레벨을 명시적으로 지정할 수 없다.
    - 대신, 프로토콜 자체의 접근 레벨은 extension 내에서 각 프로토콜 요구 사항 구현에 대한 기본 접근 단계를 제공하는데 사용된다.

## **Private Members in Extensions**

동일한 파일 내에 있는 extension은 그 코드의 원본 타입 정의의 일부로 쓰여질 수 있다.

- 원본 정의에 private 멤버로 선언한 것을 extension에서 멤버로 접근할 수 있다.
- 하나의 extension에서 private 멤버로 선언한 것을 같은 파일의 다른 extension에서 접근할 수 있습니다.
- 하나의 extension에서 private 멤버로 선언한 것을 원본 선언에서 멤버로 접근할 수 있습니다.

```swift
protocol SomeProtocol {
    func doSomething()
}

struct SomeStruct {
    private var privateVariable = 12
}

extension SomeStruct: SomeProtocol {
    func doSomething() {
        print(privateVariable)
    }
}
```

- 첫 번째 경우 충족

# **Generics**

제네릭 타입 또는 제네릭 함수를 위한 접근 레벨은 제네릭 타입 또는 함수 자신의 접근 수준과 타입 인자에 모든 타입 제약의 접근 수준의 최소이다.

# **Type Aliases**

- 정의한 타입 별칭은 액세스 제어를 위해 별개의 유형으로 취급된다.
- 타입 별칭은 원 타입의 접근 레벨보다 덜하거나 같은 수준을 가질 수 있다.
- 예시: private의 타입 별칭으로 private, file-private, internal, public, or open type를 가질 수 있다.
- 예시: public의 타입 별칭으로 internal, file-private, or private를 가질 수 없다.