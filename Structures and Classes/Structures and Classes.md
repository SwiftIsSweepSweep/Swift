# Structures and Classes

- 다른 언어들(Objective C 등)과 달리, 스위프트는 구조체와 클래스를 만드는데 별도의 인터페이스 및 구현 파일을 만들 필요 없다. 하나의 파일에 구조체와 클래스를 정의하면, 자동으로 외부 인터페이스가 사용된다.

<aside>
💡 클래스의 인스턴스는 전통적으로 Object로 알려져 있다. 그러나, 스위프트의 구조체와 클래스는 다른 언어보다 기능에 더 가깝고 이 챕터의 대부분도 클래스 또는 구조체 타입의 인스턴스를 적용할 수 있는 기능을 설명한다. 따라서, 일반적인 용어로 인스턴스가 사용된다.

</aside>

# **Comparing Structures and Classes**

### 공통적인 기능

1. 값을 저장하는 프로퍼티 선언 가능
2. 기능을 제공하는 메소드 선언 가능
3. 값에 접근 가능한 subscript 선언 가능 
4. 초기 상태를 설정하기 위한 생성자 선언 가능
5. 기본적인 구현을 넘어선 기능 확장 가능 
6. 특정 종류의 표준 기능을 제공하는 프로토콜 채택 가능 

### 클래스만 가능한 기능

1. 한 클래스가 다른 클래스의 특성을 사용할 수 있는 상속 
2. 타입 캐스팅을 통해 런타임에 클래스 인스턴스의 타입을 확인 가능 
3. 클래스의 인스턴스 할당을 해제하는 Deinitializers
4. 클래스 인스턴스의 둘 이상의 참조를 허용 

→ 이런 추가적인 기능은 complexity 비용을 증가 시킴

→ 일반적으로 추론하기 쉬운 구조체를 사용하고, 필요할 경우 클래스를 사용 

→ 대부분의 사용자 지정 데이터 형식은 구조체와 열거형 

[Choosing Between Structures and Classes](https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes)

## **Definition Syntax**

- 구조체와 클래스는 비슷하게 선언한다.
- 구조체는 `struct` 키워드를 클래스는 `class` 키워드를 사용하면 된다.
- Swift 스타일을 따라 구조체와 클래스는 UpperCamelCase를 사용하는 것이 좋다.
    - 메서드나 프로퍼티는 lowerCamelCase를 사용하는 것이 좋다.

```swift
struct Resolution {
    var width = 0
    var height = 0
}
class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}
```

## **Structure and Class Instances**

- 위의 정의는 어떻게 보일지만 정의한 것이지 실제로 특정 값을 의미하지 않는다. 따라서, 인스턴스를 만들어야 한다.
- 구조체와 클래스는 비슷하게 인스턴스를 만든다.

```swift
let someResolution = Resolution()
let someVideoMode = VideoMode()
```

→ default value로 가장 간단하게 인스턴스를 만드는 문법 

## **Accessing Properties**

- 점(.)을 사용해 인스턴스의 프로퍼티에 접근할 수 있다.
- 새 값을 할당할 수도 있다.

```swift
someVideoMode.resolution.width = 1280
print("The width of someVideoMode is now \(someVideoMode.resolution.width)")
// Prints "The width of someVideoMode is now 1280"
```

## **Memberwise Initializers for Structure Types**

- 구조체는 memberwise initializer를 자동으로 생성한다
- 새로운 인스턴스의 초기화 값은 프로퍼티의 이름으로 초기화 할 수 있다

```swift
let vga = Resolution(width: 640, height: 480)
```

- 구조체와 다르게, 클래스는 default memberwise initializer가 없다.

# **Structures and Enumerations Are Value Types**

- Value Type: 변수 또는 상수에 할당되거나, 함수에 전달될 때 값이 복사됨
- integers, floating-point numbers, Booleans, strings, arrays and dictionaries는 값 타입
    - 위의 것들은 구조체로 구현되어 있음
- 모든 구조체와 열거형은 값 타입
    
    → 즉, 이 인스턴스들의 프로퍼티는 항상 복사되며 전달된다.
    

<aside>
💡 arrays, dictionaries, and strings과 같은 표준 라이브러리에 의해 정의된 컬렉션 타입은 최적화를 통해 복사 비용을 줄였다.
즉시 복사를 하지 않고, 원본과 복사본 사이에 저장되는 메모리를 공유한다. 만약, 복사본의 값이 수정되려고 하면 수정 직전에 원본을 복사한다. 따라서, 코드에서는 즉시 복사된 것처럼 보인다.

</aside>

```swift
let hd = Resolution(width: 1920, height: 1080)
var cinema = hd

cinema.width = 2048
print("cinema is now \(cinema.width) pixels wide")
// Prints "cinema is now 2048 pixels wide"
print("hd is still \(hd.width) pixels wide")
// Prints "hd is still 1920 pixels wide"
```

- hd가 cinema에 할당될 때, 새로운 복사본이 만들어지고 새로운 복사본이 할당된다.
- cinema의 width를 수정해도 hd에 영향을 주지 않는다.

![Untitled](Structures%20da4f2/Untitled.png)

[열거형에 적용한 예제]

```swift
enum CompassPoint {
    case north, south, east, west
    mutating func turnNorth() {
        self = .north
    }
}
var currentDirection = CompassPoint.west
let rememberedDirection = currentDirection
currentDirection.turnNorth()

print("The current direction is \(currentDirection)")
print("The remembered direction is \(rememberedDirection)")
// Prints "The current direction is north"
// Prints "The remembered direction is west"
```

- rememberedDirection에 영향 X

# **Classes Are Reference Types**

- Reference Type: 변수 또는 상수에 할당되거나, 함수에 전달될 때 값이 복사되지 않고 기존 인스턴스에 대한 참조가 사용됨

```swift
let tenEighty = VideoMode()
tenEighty.resolution = hd
tenEighty.interlaced = true
tenEighty.name = "1080i"
tenEighty.frameRate = 25.0

let alsoTenEighty = tenEighty
alsoTenEighty.frameRate = 30.0

print("The frameRate property of tenEighty is now \(tenEighty.frameRate)")
// Prints "The frameRate property of tenEighty is now 30.0"
```

- tenEighty과 alsoTenEighty은 같은 VideoMode 인스턴스를 참조한다.
- tenEighty와 alsoTenEighty은 서로 영향을 주고 받는다.
- tenEighty와 alsoTenEighty은 상수로 선언되어 있지만, 그 프로퍼티는 변경할 수 있다.
    - 왜냐하면, tenEighty와 alsoTenEighty의 상수 값은 실제로 변경되지 않았기 때문이다.
    - 이 둘은 인스턴스를 “저장"하지 않는다. 대신, 인스턴스를 참조한다.
    - 변경된 것은 VideoMode의 속성이지, VideoMode의 레퍼런스 상수값이 아니다.
- 레퍼런스 타입은 추론하기 어렵다.
    - 프로그램에서 이 둘이 떨어져있으면, 비디오 모드가 변경되는 모든 방법을 찾기 어려울 수 있다.
    - 이 둘을 사용하는 경우, 항상 서로의 코드를 고려해야 한다.
    - 이와 반대로, 값 유형은 추론하기 쉽다.

![Untitled](Structures%20da4f2/Untitled%201.png)

## **Identity Operators**

- 클래스는 참조 타입이기 때문에 여러 상수와 변수가 하나의 클래스 인스턴스를 참조할 수 있다.
(구조체나 열거형은 항상 복사하기 때문에 같지 않다.)
- 다음의 연산자를 통해 정확히 두 개의 상수 또는 변수가 동일한 인스턴스를 참조하는지 여부를 확인할 수 있다.
    - Identical to (`===`)
    - Not identical to (`!==`)

```swift
if tenEighty === alsoTenEighty {
    print("tenEighty and alsoTenEighty refer to the same VideoMode instance.")
}
// Prints "tenEighty and alsoTenEighty refer to the same VideoMode instance."
```

- “identical(===)”과 “equal(==)”은 같지 않다.
    - identical: 클래스 유형의 두 상수 또는 변수가 정확히 동일한 클래스 인스턴스를 참조한다는 의미
    - equal: 타입 설계자가 정의한 동등한 의미에 따라, 두 인스턴스들이 같은 값을 가지고 있음을 의미

## **Pointers**

- C, C++, Objective-C는 메모리 주소를 참조하기 위해 포인터를 사용한다.
- Swift의 참조 타입의 인스턴스를 가지는 변수 또는 상수는 C의 포인터와 비슷하지만, 메모리 주소에 대한 직접적인 포인터가 아니기 때문에 참조를 만들 때 “*”를 써주지 않아도 된다.
    - Swift의 참조는 다른 변수 또는 상수를 정의처럼 정의된다.
    - 직접 포인터와 상호 작용할 때를 위해, 표준 라이브러리는 포인터 및 버퍼 타입을 제공한다.

# ****Choosing Between Structures and Classes****

1. 기본적으로 구조체 사용
2. Objective-C와 상호운용성이 필요할 때 클래스 사용
3. 모델링한 데이터의 동일성을 통제할 필요가 있다면 클래스 사용
4. 구조체와 프로토콜을 사용해서 공통된 기능을 공유 

## ****Choose Structures by Default****

- Swift의 구조체는 다른 프로그래밍 언어의 클래스에는 없는 많은 기능을 포함한다.
    - 저장 프로퍼티, 계산 프로퍼티, 메소드 포함 및 프로토콜 채택 가능
- 숫자형, 문자열, 배열, 딕셔너리 같은 여러 타입 등 Swift 표준 라이브러리와 Foundation 프레임 워크는 구조체를 통해 구현
- 구조체를 사용하면 앱의 전체 상태에 대해 고려할 필요가 없다.
    - 구조체에 변경이 생겨도 의도적으로 앱의 흐름에 끼워넣지 않는 이상 변경 사항은 신경쓰지 않아도 된다.

## ****Use Classes When You Need Objective-C Interoperability****

- 데이터 처리를 위해 Obejctive-C API 사용하거나, 데이터 모델을 Objective-C 프레임워크에 선언된 클레스 계층내에 적용해야 한다면 클래스나 클래스 상속을 사용

## ****Use Classes When You Need to Control Identity****

- 클래스는 identify의 의미를 가진다.
- 앱 전체에 걸쳐 공유하는 클래스의 인스턴스를 변경하면, 변경사항은 해당 인스턴스를 가지는 모든 코드에 영향을 준다.
- 인스턴스의 동일성을 처리해야 한다면, 클래스 사용
- File Handling, Network Connection, Hardware intermeiaries (CBCentralManager - core bluetooth) 등에서 사용
- 예를 들어, 로컬 DB 연결을 나타내는 타입이 있다면 앱에서 DB 상태를 완전히 제어해야 한다. 이 경우, 클래스를 사용하는 것이 적절하다.
    - 하지만, 앱의 공유 DB 개체에 대한 액세스를 제한해야 한다.

<aside>
💡 Identity는 조심히 다뤄야 한다. 클래스 인스턴스를 앱 전체에 공유하면, 로직상의 에러가 발생하기 쉽다. 공유가 많이 된 인스턴스를 변경하면 어떤 결과가 발생할지 예상하기 어려우므로 코드를 더 정확히 작성해야 한다.

</aside>

## ****Use Structures When You Don't Control Identity****

- Identity를 관리할 필요 없으면 구조체를 사용한다.
- 예를 들어, 원격 DB를 사용하는 앱에서는 인스턴스의 동일성을 외부 엔티티가 완전히 제어하고 식별자로 통신할 수 있다. 앱 모델의 일관성이 서버에 저장되어 있으면, 모델을 식별자와 함께 구조체에 저장하면 된다.

## ****Use Structures and Protocols to Model Inheritance and Share Behavior****

- 구조체와 클래스는 상속을 지원한다.
- 구조체와 프로토콜은 프로토콜만 채택할 수 있고 클래스를 상속하지 못 한다.
    - 그러나, 구조체와 프로토콜을 함께 사용하면 클래스의 상속과 같은 계층 구조를 만들어 사용할 수 있다.
- 개발 초기부터 상속 관계를 설계한다면 프로토콜 상속을 추천한다.
    - 프로토콜은 클래스, 구조체, 열거형에서 사용할 수 있지만, 클래스는 오로지 클래스에서만 상속할 수 있기 때문이다.
    - 데이터를 모델링 방법을 선택하는 상황이라면, 프로토콜 상속을 통해 데이터 타입의 계층 구조를 설계하고, 구조체가 해당 프로토콜을 채택하는 방식을 사용한다.




[Structures and Classes - The Swift Programming Language (Swift 5.6)](https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html)
[](https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes)
