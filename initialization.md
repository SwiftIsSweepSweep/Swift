## Setting Initial Values for Stored Properties
- 클래스와 스트럭처는 인스턴스가 생성되는 시점에 모든 stored property가 초기화되어야 한다.
- 옵셔널 프로퍼티또한 nil로 초기화 되는 것이다.
- 기본값을 할당하는 2가지 방법(정의와 동시에 할당, 생성자에서 할당) 둘다 프로퍼티 옵저버를 호출하지 않는다.
- 아래 코드에서 willSet과 didSet은 `a.height = 40`에 한번만 호출됨
``` swift
struct Person {
    var height = 170 { //여기서 170이 할당되지만 프로퍼티 옵저버 호출 X
        didSet {
            print(oldValue)
        }
        willSet {
            print(newValue)
        }
    }
    
    init() {
        self.height = 30 //여기서 30이 할당되지만 프로퍼티 옵저버 호출 X
    }
    
    
}

var a = Person()
a.height = 40
```
- 결과
```
40
30
Program ended with exit code: 0
```
## Default Initializers
- 구조체나 클래스에서 모든 프로퍼티가 기본값을 갖고 있거나 옵셔널이고 생성자를 하나도 구현하지 않았다면 Default Initializer가 실행된다.
- Default Initializer = 모든 프로퍼티의 값을 기본값(default value)로 할당해주고 인스턴스를 생성하는 생성자

- 예시, init()을 구현하지 않았지만 기본값을 할당해주는 Default 생성자가 호출됨
```swift
class ShoppingListItem {
    var name: String?
    var quantity = 1
    var purchased = false
}
var item = ShoppingListItem()
```

## Memberwise Initializers for Structure Types (클래스에는 없음)
- 구조체에서 기본적으로 제공하는 생성자
- 이미 기본값이 할당된 변수는 생략할 수 있다.
- 예시
```swift
struct Size {
    var width = 0.0, height = 0.0
}

//width, height 둘다 할당
let twoByTwo = Size(width: 2.0, height: 2.0)

//width 생략
let zeroByTwo = Size(height: 2.0)

//height 생략
let zeroByZero = Size()
```

## Initializer Delegation for Value Types
- Initializer Delegation = 생성자 안에서 다른 생성자를 호출하는 코드
- Initializer Delegation은 Value type과 class type 둘다 있음
- Value type은 상속이 없기때문에 규칙이 더 단순함
- Initializer Delegation이 일어나는 코드에서도 Convenience 키워드가 필요없음

- Custom 생성자를 value type에 정의하면, 더 이상 기본 생성자를 사용할 수 없음
  - 구조체라면 멤버와이즈 생성자 또한 접근 못함
  - Custom 생성자와 기본 생성자를 둘다 사용하고 싶다면 Custom 생성자를 extension에 정의하면 됨

- 예시
```swift
struct Size {
    var width = 0.0, height = 0.0
}
struct Point {
    var x = 0.0, y = 0.0
}    
struct Rect {
    var origin = Point()
    var size = Size()
    init() {}
    init(origin: Point, size: Size) {
        self.origin = origin
        self.size = size
    }
    //이부분에서 delegation이 일어남. 하지만 convenience 키워드가 필요없음
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

## Class Inheritance and Initialization
- 클래스에만 convenience init과 designated init이 있음
- 모든 클래스에는 최소 한개의 designated 생성자가 있어야함
- 같은 클래스의 다른 생성자를 호출하려면 convenience init으로 만들어야 한다.

### Initializer Delegation for Class Types

>Rule 1
A designated initializer must call a designated initializer from its immediate superclass.
Rule 2
A convenience initializer must call another initializer from the same class.
Rule 3
A convenience initializer must ultimately call a designated initializer.

A simple way to remember this is:
- Designated initializers must always delegate up.
- Convenience initializers must always delegate across.
![](https://i.imgur.com/XngfhQ2.png)

>1. 같은 클래스의 다른 생성자를 호출하려면 convenience 키워드가 필요함
>2. 상위 클래스의 생성자(super.init())은 designated 생성자에서만 가능하고 여기서 불려지는 생성자 또한 designated 생성자만 가능함


## Two-Phase Initialization

- 클래스의 초기화는 2단계로 이루어짐

1 phase는 위로 올라가는 과정
2 phase는 아래로 내려가는 과정

- 해당 객체에 메소드에는 2phase에서만 접근할 수 있다.
- 상위 클래스의 프로퍼티나 메소드는 2phase에서만 접근할 수 있다.
- 해당 객체의 프로퍼티는 1phase, 2phase 둘다 접근할 수 있다.
- 하위 클래스의 모든 프로퍼티가 초기화 되지 않으면 상위클래스의 designated initializer는 호출되지 못한다.
- 실습 코드
```swift
import UIKit
class Parent {
    var name: String
    init(name: String) {
        self.name = name
    }
    
    func say() {
        print("haha")
    }
}

class Child: Parent {
    var childName: String
    init(childName: String) {
        self.childName = childName
        print(self.childName)
        //----------여기까지 1phase
        //여기서는 이 객체의 메소드에 접근 불가
        //상위 클래스에 그 어떤것도 접근 불가
        //오직 이 객체의 프로퍼티만 접근가능
        super.init(name: "papa")
        //-----------------------------
        //----------여기부터 2phase
        super.say()
        super.name = "hihi"
        print(self.name)
        self.sayHi()
    }
    
    convenience init(age: Int) {
        self.init(childName: "child")
        self.childName = "hihi"
    }
    
    func sayHi() {
        print("HI")
    }
}
```



>Safety check 1
상위 클래스의 생성자를 호출하기전에 해당 클래스의 모든 프로퍼티가 초기화되어야함
클래스가 생성될 때 모든 저장 프로퍼티의 값을 알아야 메모리를 할당할 수 있음. 따라서 해당 클래스의 designated 생성자가 모든 저장 프로퍼티의 값이 초기화되었는지 판단함

>Safety check 2
하위 클래스에서 상위클래스로부터 상속된 프로퍼티에 접근하려면 designated initializer는 델리게이트 업으로 상위 클래스의 생성자를 먼저 호출해야 함.
그렇지 않으면(하위 클래스에서 상위클래스의 프로퍼티 값을 할당하면) 어차피 상위클래스의 designated initializer에 의해 값이 덮어씌워짐 <- `근데 이게 Xcode에서 문법적으로 불가능함`

>Safety check 3
> 컨비니언스 이니셜라이저는 프로퍼티에 값을 할당하기 전에 무조건 다른 이니셜라이저를 호출해서 모든 프로퍼티의 값이 할당되어야 한다. 그렇지 않으면 할당된 값이 다른 이니셜라이저에 의해 덮어씌워짐 <- `근데 이게 Xcode에서 문법적으로 불가능함`


>Safety check 4
생성자는 1 phase initialization이 완료되기 전에 다음과 같은 행동을 할 수 없다.
인스턴스의 메소드 호출, 인스턴스의 프로퍼티 접근, self를 값으로써 접근
1 phase initialization이 완료된 시점은 클래스의 인스턴스가 완전히 생성된 시기가 아니지만 인스턴스 프로퍼티에 접근, 메소드 호출이 가능하다.


## Initializer Inheritance and Overriding

- 하위 클래스가 상위 클래스와 같은 생성자를 만들 때 override 키워드를 써야한다.
  - 이는 하위클래스의 간편 생성자가 상위 클래스의 지정 생성자와 같은 경우에도 마찬가지다.
  - 하위클래스의 지정생성자가 상위클래스의 간편생성자가 같은 경우에는 override키워드를 쓸 필요가 없다. 
  - 하위클래스의 간편생성자가 하위클래스의 지정생성자를 바로 호출할 수가 없기때문에 override하는게 아니라고 함

경우1) 하위의 지정 생성자가 상위의 지정생성자를 override하는경우 -> override 키워드 필요함
```swift
class Animal {
    let name: String
    
    init(name: String) {
        self.name = name
    }
}

class Dog: Animal {

    override init(name: String) { //override 안쓰면 에러남
        super.init(name: name)
    }
}
```

경우2) 하위의 간편생성자가 상위의 지정생성자와 같은경우 -> override 키워드 필요함
```swift
class Animal {
    let name: String

    init(name: String) {
        self.name = name
    }

    init(lastName: String) {
        self.name = "kim" + lastName
    }
}

class Dog: Animal {
    let age: Int

    init(name: String, age: Int) {
        self.age = age
        super.init(name: name)
    }

    override convenience init(lastName: String) {
        self.init(name: "kim" + lastName, age: 30)
    }
}
```

경우3) 하위의 지정생성자가 상위의 간편생성자와 같은 경우 -> override 키워드 필요없음
```swift
class Animal {
    let name: String

    init(name: String) {
        self.name = name
    }

    convenience init(lastName: String) {
        self.init(name: "noName" + lastName)
    }
}

class Dog: Animal {
    let age: Int

    init(lastName: String) { //override 쓰면 에러남
        self.age = 30
        super.init(name: "pikachu")
    }
}
```

경우3) 하위의 간편생성자가 상위의 간편생성자와 같은경우 -> override 키워드 필요없음
```swift
class Animal {
    let name: String

    init(name: String) {
        self.name = name
    }

    convenience init(lastName: String) {
        self.init(name: "noName" + lastName)
    }
}

class Dog: Animal {
    let age: Int

    init(name: String, age: Int) {
        self.age = age
        super.init(name: name)
    }

    convenience init(lastName: String) {//override 키워드 필요없음
        self.init(name: "noName" + lastName, age: 30)
    }
}
```

| 하위 클래스의 생성자      | 상위 클래스의 생성자 | override 키워드 여부 |
| ----------- | ----------- | --------------|
| 지정      | 지정       |  override 필요함 |
| 간편   | 지정        |    override 필요함  |
| 지정   | 간편        |    override 필요없음  |
| 간편   | 간편        |    override 필요없음  |


- 하위 클래스가 phase 2에서 아무런 할 일이 없고(상위클래스의 프로퍼티를 수정하지 않고 현재 클래스의 메소드에 접근하지 않을때), 그리고 상위 클래스의 지정 생성자가 argument가 없을때 super.init()을 생략할 수 있다.
- 아래는 예시

```swift
class Vehicle {
    var numberOfWheels = 0
    var description: String {
        return "\(numberOfWheels) wheel(s)"
    }
}

class Hoverboard: Vehicle {
    var color: String
    init(color: String) {
        self.color = color
        // super.init() implicitly called here
        //만약 상위 클래스의 프로퍼티에 접근하고 싶으면 super.init()을 명시해야함
    }
    override var description: String {
        return "\(super.description) in a beautiful \(color)"
    }
}

```

## Automatic Initializer Inheritance
- 하위 클래스는 상위클래스의 지정 생성자를 기본적으로 상속하지 않는다.(특정 조건에서만 상속함)


- 하위 클래스에서 새로운 프로퍼티를 만들었을 때 다음 2가지 룰이 적용됨
  - 첫번째 룰
    - 하위 클래스에서 지정생성자를 만들지 않으면 상위 클래스의 모든 지정생성자가 상속됨
  - 두번째 룰
    - 하위 클래스에서 슈퍼클래스의 모든 지정 생성자를 만들면(첫번재 룰에 의해 상속된 경우 포함) 상위 클래스의 모든 간편 생성자가 상속됨
    - 이는 서브클래스에서 추가적인 간편생성자를 만든 경우도 포함된다.
    
NOTE

A subclass can implement a superclass designated initializer as a subclass convenience initializer as part of satisfying rule 2.

