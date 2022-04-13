# Properties
- Computed Property는 enum에서도 가능하다.
- Stored Property는 class와 structure에서만 가능하다


- 인스턴스에 프로퍼티 이외에도 타입 자차에 대한 프로퍼티를 만들 수 있는데(static) 이를 Type Property라고 한다.

# Stored Properties
- 클래스나 스트럭처의 인스턴스에 값을 저장하는 방식
- var, let 사용가능


## Stored Properties of Constant Structure Instances

- 스트럭처의 인스턴스를 let으로 저장하면 스트럭처 내부의 값을 바꿀수 없다.(var로 선언하였다 하더라도)
- 아래 예시
```swift
struct FixedLengthRange {
    var firstValue: Int
    let length: Int
}
let rangeOfFourItems = FixedLengthRange(firstValue: 0, length: 4)
// this range represents integer values 0, 1, 2, and 3
rangeOfFourItems.firstValue = 6
// this will report an error, even though firstValue is a variable property
```
- 그 이유는 스트럭처에 value type이기 때문임. 인스턴스가 상수화되어서 내부의 프로퍼티 마저 상수화되어버림

- 하지만 클래스는 reference type이기 때문에 다르다. 인스턴스가 상수로 선언되어도 내부의 var로 선언된 stored property들은 변경가능하다.


## Lazy Stored Properties
- lazy Stored property는 값이 사용되기 전까지 초기값이 연산되지 않는다.

> NOTE
lazy property는 변수만 가능하다.(상수 불가능)
상수는 인스턴스의 초기화가 끝나기 전에 값을 갖고 있어야 하는데 lazy property는 초기화가 끝나도 값이 없기때문에 변수를 사용해야 한다.

- lazy Stored property는 2가지 용도로 많이 쓰이는데 
  1. property의 초기값이 외부의 값으로 부터 정해지는 경우
  2. property의 값을 구할 때 연산이 많이 걸리는 경우

아래 코드의 경우 DataImporter에서 파일을 불러오는 코드를 작성한다고 가정했을때 꽤많은 시간이 소요된다. DataManager가 단순히 data array에 데이터만 추가한다고 하면 importer가 필요없을 수 있다. 따라서 lazy keyword를 통해 다음과 같이 importer가 진짜 필요한 시점에 생성되도록 할 수 있다.
```swift
class DataImporter {
    /*
    DataImporter is a class to import data from an external file.
    The class is assumed to take a nontrivial amount of time to initialize.
    */
    var filename = "data.txt"
    // the DataImporter class would provide data importing functionality here
}

class DataManager {
    lazy var importer = DataImporter()
    var data: [String] = []
    // the DataManager class would provide data management functionality here
}

let manager = DataManager()
manager.data.append("Some data")
manager.data.append("Some more data")
// the DataImporter instance for the importer property hasn't yet been created
```
아래 코드를 작성하기 전까지 importer는 초기화 되지 않는다.
```swift
print(manager.importer.filename)
// the DataImporter instance for the importer property has now been created
// Prints "data.txt"
```
>NOTE
만약 lazy stored property가 여러개의 쓰레드에 의해 동시에 접근된다면, 딱한번만 초기화된다는 보장은 없다

## Stored Properties and Instance Variables
If you have experience with Objective-C, you may know that it provides two ways to store values and references as part of a class instance. In addition to properties, you can use instance variables as a backing store for the values stored in a property.

Swift unifies these concepts into a single property declaration. A Swift property doesn’t have a corresponding instance variable, and the backing store for a property isn’t accessed directly. This approach avoids confusion about how the value is accessed in different contexts and simplifies the property’s declaration into a single, definitive statement. All information about the property—including its name, type, and memory management characteristics—is defined in a single location as part of the type’s definition.

# Computed Properties
- Enumeration에는 Stored property가 없지만 Computed property는 가질 수 있다. 다만 값을 저장할 수는 없다.

```swift
struct Point {
    var x = 0.0, y = 0.0
}
struct Size {
    var width = 0.0, height = 0.0
}
struct Rect {
    var origin = Point()
    var size = Size()
    var center: Point {
        get {
            let centerX = origin.x + (size.width / 2)
            let centerY = origin.y + (size.height / 2)
            return Point(x: centerX, y: centerY)
        }
        set(newCenter) {
            origin.x = newCenter.x - (size.width / 2)
            origin.y = newCenter.y - (size.height / 2)
        }
    }
}
var square = Rect(origin: Point(x: 0.0, y: 0.0),
                  size: Size(width: 10.0, height: 10.0))
let initialSquareCenter = square.center
// initialSquareCenter is at (5.0, 5.0)
square.center = Point(x: 15.0, y: 15.0)
print("square.origin is now at (\(square.origin.x), \(square.origin.y))")
// Prints "square.origin is now at (10.0, 10.0)"
```

## Shorthand Setter Declaration

- setter에 굳이 newCenter를 명시 안해도 newValue로 접근해서 set 시기에 받을 인자를 사용할 수 있다.
- 아래 코드는 위와 같은 동작을 한다.
```swift
struct AlternativeRect {
    var origin = Point()
    var size = Size()
    var center: Point {
        get {
            let centerX = origin.x + (size.width / 2)
            let centerY = origin.y + (size.height / 2)
            return Point(x: centerX, y: centerY)
        }
        set {
            origin.x = newValue.x - (size.width / 2)
            origin.y = newValue.y - (size.height / 2)
        }
    }
}
```

## Shorthand Getter Declaration
- getter가 한줄로만 이루어져 있다면(single expression) 굳이 return을 안적어도 return 된다. 
  - 위의 룰은 function에서도 그대로 작동한다(Implicit Return)
- 아래 코드는 위와 같은 동작을 한다.

```swift
struct CompactRect {
    var origin = Point()
    var size = Size()
    var center: Point {
        get {
            Point(x: origin.x + (size.width / 2),
                  y: origin.y + (size.height / 2))
        }
        set {
            origin.x = newValue.x - (size.width / 2)
            origin.y = newValue.y - (size.height / 2)
        }
    }
}
```


## Read-Only Computed Properties
- computed property에 getter만 있고 setter가 없다면 읽기 전용 computed property라고 한다.
  - 표현만으로 알 수 있지만 읽기만 되고 set이 되지 않는다.

>NOTE
Computed property는(read-only computed property포함) 항상 var로 선언되어야 한다. 왜냐하면 고정된 값이 아니기 때문이다. 상수는 인스턴스 초기화후에 할당된 값이 변하면 안되기 때문이다.

>(You must declare computed properties—including read-only computed properties—as variable properties with the var keyword, because their value isn’t fixed. The let keyword is only used for constant properties, to indicate that their values can’t be changed once they’re set as part of instance initialization.)

- read-only computed property는 get을 생략하고 쓸 수 있다.

```swift
struct Cuboid {
    var width = 0.0, height = 0.0, depth = 0.0
    var volume: Double {
        return width * height * depth
    }
}
let fourByFiveByTwo = Cuboid(width: 4.0, height: 5.0, depth: 2.0)
print("the volume of fourByFiveByTwo is \(fourByFiveByTwo.volume)")
// Prints "the volume of fourByFiveByTwo is 40.0"
```

## Property Observers
- 프로퍼티의 변화를 감시하는 문법
- 프로퍼티에 property observer를 달면 프로퍼티의 새로운 값이 할당될 때마다 property observer가 실행된다.
  - 현재 값과 같은 값을 할당해도 property observer가 실행된다.

프로퍼티 옵저버를 추가할 수 있는 곳을 다음과 같다:

- Stored properties that you define
- Stored properties that you inherit
- Computed properties that you inherit

- 상속된 프로퍼티에 프로퍼티 옵저버를 추가하기 위해서는 property를 override해서 추가해야 한다.
- 위의 3곳중에 언급이 되지 않은 직접 정의한 computed property의 경우 setter가 property observer의 역할을 대신할 수 있다.

프로퍼티 옵저버는 아래 표기된것처럼 2가지가 있다:

- willSet: 값이 저장되기 직전에 실행된다.
- didSet: 값이 저장되자마자 실행된다.

- willSet 옵저버를 구현하면 새로 할당된 값이 상수 인자로 넘어오게 된다. 이 인자의 이름을 새로 지정할 수도 있고 newValue라는 파라미터 이름으로 사용할 수 있다.

- didSet도 마찬가지로 원래 지정되었던 값이 상수 인자로 넘어오게 된다. 인자의 이름을 지정하지 않으면 oldValude로 사용할 수 있다.

>NOTE
The willSet and didSet observers of superclass properties are called when a property is set in a subclass initializer, after the superclass initializer has been called. They aren’t called while a class is setting its own properties, before the superclass initializer has been called.

For more information about initializer delegation, see Initializer Delegation for Value Types and Initializer Delegation for Class Types.


willSet, didSet 예시
```swift
class StepCounter {
    var totalSteps: Int = 0 {
        willSet(newTotalSteps) {
            print("About to set totalSteps to \(newTotalSteps)")
        }
        didSet {
            if totalSteps > oldValue  {
                print("Added \(totalSteps - oldValue) steps")
            }
        }
    }
}
let stepCounter = StepCounter()
stepCounter.totalSteps = 200
// About to set totalSteps to 200
// Added 200 steps
stepCounter.totalSteps = 360
// About to set totalSteps to 360
// Added 160 steps
stepCounter.totalSteps = 896
// About to set totalSteps to 896
// Added 536 steps
```

>NOTE
프로퍼티 옵저버가 달린 프로퍼티를 함수의 in-out parameter로 넣는다면 willSet과 didSet이 항상 호출된다. in-out parameter는 copy-in, copy-out 메모리 모델때문인데 함수가 종료될때 인자로 넘어온 프로퍼티를 항상 덮어쓰기 때문이다.

# Property Wrappers
프로퍼티마다 동일한 동작을 해야할 때 프로퍼티 래퍼를 사용하면 편하다

프로퍼티 래퍼를 만드려면 구조체, enum, class에 wrappedValue 프로퍼티를 만들어주면 된다.

```swift
@propertyWrapper
struct TwelveOrLess {
    private var number = 0
    var wrappedValue: Int {
        get { return number }
        set { number = min(newValue, 12) }
    }
}
```

>NOTE
위에 코드에서 number에 경우 wrappedValue를 통해서만 접근할 수 있게 private으로 선언되어 있는것을 주의!


```swift
struct SmallRectangle {
    @TwelveOrLess var height: Int
    @TwelveOrLess var width: Int
}

var rectangle = SmallRectangle()
print(rectangle.height)
// Prints "0"

rectangle.height = 10
print(rectangle.height)
// Prints "10"

rectangle.height = 24
print(rectangle.height)
// Prints "12"
```

height 와 width 프로퍼티는 위의 TwelveOrLess에서 정의된 대로 초기값 0을 갖는다. height에 10을 넣으면 10이 저장되고 height에 24를 할당하면 `min(newValue, 12)`에 의해 12가 할당된다.

프로퍼티 래퍼는 `@`을 통해 문법적 간편함을 제공하는데 만약 이 기호를 사용하고 싶지 않다면 다음과 같이 코드를 작성할 수도 있다.
```swift
struct SmallRectangle {
    private var _height = TwelveOrLess()
    private var _width = TwelveOrLess()
    var height: Int {
        get { return _height.wrappedValue }
        set { _height.wrappedValue = newValue }
    }
    var width: Int {
        get { return _width.wrappedValue }
        set { _width.wrappedValue = newValue }
    }
}
```

## Setting Initial Values for Wrapped Properties
위에서는 프로퍼티래퍼를 적용한 프로퍼티의 초기값은 0으로 고정되었었다. 여기에 여러가지 초기값을 주기위해 프로퍼티 래퍼에 생성자를 추가할 수 있다.

```swift
@propertyWrapper
struct SmallNumber {
    private var maximum: Int
    private var number: Int

    var wrappedValue: Int {
        get { return number }
        set { number = min(newValue, maximum) }
    }

    init() {
        maximum = 12
        number = 0
    }
    init(wrappedValue: Int) {
        maximum = 12
        number = min(wrappedValue, maximum)
    }
    init(wrappedValue: Int, maximum: Int) {
        self.maximum = maximum
        number = min(wrappedValue, maximum)
    }
}
```


첫번재 생성자 사용(초기값을 직접 정해주지 않음)
```swift
struct ZeroRectangle {
    @SmallNumber var height: Int
    @SmallNumber var width: Int
}

var zeroRectangle = ZeroRectangle()
print(zeroRectangle.height, zeroRectangle.width)
// Prints "0 0"
```

두번째 생성자 사용
프로퍼티 래퍼를 적용한 프로퍼티에 초깃값을 정해주면 자동으로 init(wrappedValue: Int)가 실행되고 지정된 초기값이 인자로 들어간다.

```swift
struct UnitRectangle {
    @SmallNumber var height: Int = 1
    @SmallNumber var width: Int = 1
}

var unitRectangle = UnitRectangle()
print(unitRectangle.height, unitRectangle.width)
// Prints "1 1"
```

세번째 생성자 사용
만약 wrappedValue이외의 인자를 넣어주고 싶으면 다음과 같은 문법을 사용하면 된다.


```swift
struct NarrowRectangle {
    @SmallNumber(wrappedValue: 2, maximum: 5) var height: Int
    @SmallNumber(wrappedValue: 3, maximum: 4) var width: Int
}

var narrowRectangle = NarrowRectangle()
print(narrowRectangle.height, narrowRectangle.width)
// Prints "2 3"

narrowRectangle.height = 100
narrowRectangle.width = 100
print(narrowRectangle.height, narrowRectangle.width)
// Prints "5 4"
```

height와 width는 다른 인스턴스이다. 각 인스턴스마다 다른 초기 설정을 줄 수 있다. 이 3번째 방식이 가장 널리쓰는 방식이다.


다른 인자가 있더라도 wrappedValue는 초기값을 지정해주는 방식으로 인자를 넘겨줄 수 있다. <- 이거 희한하네
아래 height와 width를 보면 된다.


```swift
struct MixedRectangle {
    @SmallNumber var height: Int = 1
    @SmallNumber(maximum: 9) var width: Int = 2
}

var mixedRectangle = MixedRectangle()
print(mixedRectangle.height)
// Prints "1"

mixedRectangle.height = 20
print(mixedRectangle.height)
// Prints "12"
```

## Projecting a Value From a Property Wrapper

Property wrapper에 projected value라는 추가 기능이 있다. $표시를 이용해서 추가적인 값을 저장할 수 있다.
이때 이름은 projectedValue를 사용해야 한다.


```swift
@propertyWrapper
struct SmallNumber {
    private var number: Int
    private(set) var projectedValue: Bool

    var wrappedValue: Int {
        get { return number }
        set {
            if newValue > 12 {
                number = 12
                projectedValue = true
            } else {
                number = newValue
                projectedValue = false
            }
        }
    }

    init() {
        self.number = 0
        self.projectedValue = false
    }
}
struct SomeStructure {
    @SmallNumber var someNumber: Int
}
var someStructure = SomeStructure()

someStructure.someNumber = 4
print(someStructure.$someNumber)
// Prints "false"

someStructure.someNumber = 55
print(someStructure.$someNumber)
// Prints "true"
```

projectedValue은 어떤 타입도 저장할 수 있다. 여기에서는 bool타입만을 리턴하는데 더 많은 데이터를 리턴하고 싶다면 costume type을 만들면 된다.

밑에 처럼 projectedNumber에 @height, @width로 접근할 수 있다. 이때 self는 생략가능하다.

```swift
enum Size {
    case small, large
}

struct SizedRectangle {
    @SmallNumber var height: Int
    @SmallNumber var width: Int

    mutating func resize(to size: Size) -> Bool {
        switch size {
        case .small:
            height = 10
            width = 20
        case .large:
            height = 100
            width = 100
        }
        return $height || $width
    }
}
```


# Global and Local Variables

stored property와 computed property모두 global, local scope에서 사용할 수 있다. 이때는 타입의 property가 아니기 때문에 명칭을 stoed variable, compputed variable이라고 부른다.

>NOTE
global 변수나 상수는 lazy키워드가 없어도 항상 lazy하게 작동한ㄷk.
local 변수와 상수는 lazy하게 작동할 수 없다.(lazy키워드 붙이면 에러가 난다.)

아래 예시처럼 local scope에서는 property wrapper를 사용할 수 있다.(global scope에서는 불가능)

```swift
func someFunction() {
    @SmallNumber var myNumber: Int = 0

    myNumber = 10
    // now myNumber is 10

    myNumber = 24
    // now myNumber is 12
}
```

## Type Properties
인스턴스에 종속되지 않은 type property를 만들 수 있다.
이때 stored type property는 lazy하게 작동되면 동시에 여러 쓰레드가 접근한다 하더라도 딱 한번만 초기화 되는게 보장된다.
또한 stored type property는 initializer가 없기때문에 항상 초기값을 갖고 있어야 한다.

## Type Property Syntax

type property를 만드는 경우 static 키워드를 사용한다.
다만 클래스의 computed 프로퍼티의 경우 subclass가 override를 허용하려면 class키워드를 사용해야 한다.
아래는 예시
class에 computed type property를 만드는 경우 class 키워드를 사용해야한다. (그 외에는 전부 static 키워드를 사용한다.)

```swift
struct SomeStructure {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
        return 1
    }
}
enum SomeEnumeration {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
        return 6
    }
}
class SomeClass {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
        return 27
    }
    class var overrideableComputedTypeProperty: Int {
        return 107
    }
}
```
>NOTE
위의 예제는 모두 read-only인데 read-write도 같은 문법으로 작성가능함.

## Querying and Setting Type Properties
타입 프로퍼티에도 값을 저장할 수 있다. 이때 instance가 아닌 type에 저장되는것을 주의해야 한ㄷㅏ.

```swift
print(SomeStructure.storedTypeProperty)
// Prints "Some value."
SomeStructure.storedTypeProperty = "Another value."
print(SomeStructure.storedTypeProperty)
// Prints "Another value."
print(SomeEnumeration.computedTypeProperty)
// Prints "6"
print(SomeClass.computedTypeProperty)
// Prints "27"
```
