# Opque Types - Swift Language Guide

함수나 메소드에서 opaque return type을 사용한다면 리턴 값의 타입 정보를 숨길 수 있습니다.
함수의 리턴타입에 구체적인 타입을 명시하는 대신 해당 타입이 따르는 프로토콜을 명시합니다.
리턴타입을 숨기는 것은 모듈과 모듈을 호출하는 코드 사이에서 해당 리턴 값을 private 하게 둘 수 있습니다.
리턴타입을 프로토콜 타입으로 설정하는 것과는 달리 opque type은 타입의 identity를 유지합니다.(module의 클라이언트는 접근할 수 없지만 컴파일러는 타입 정보에 접근할 수 있습니다.)


## The Problem That Opaque Types Solve

예를 들어, 아스키 코드로 도형을 만드는 모듈을 코딩한다고 가정합니다.
아스키 도형 에는 draw()함수가 있어서 도형의 모양을 string으로 리턴해줍니다. 
이는 Shape 프로토콜에 명시되어 있습니다.

```swift
protocol Shape {
    func draw() -> String
}

struct Triangle: Shape {
    var size: Int
    func draw() -> String {
        var result: [String] = []
        for length in 1...size {
            result.append(String(repeating: "*", count: length))
        }
        return result.joined(separator: "\n")
    }
}
let smallTriangle = Triangle(size: 3)
print(smallTriangle.draw())
// *
// **
// ***
```

아래와 같이 제네릭을 이용하여 도형을 위 아래로 뒤집을 수 있는 코드를 구현할 수 있습니다.
하지만 이런 접근 방식에는 중대한 제약이 있다. flipped result가 만들어질 때 제네릭이 노출될 수 밖에 없다.
However, there’s an important limitation to this approach: The flipped result exposes the exact generic types that were used to create it.
(인자로 타입을 넣어줘서 그런건가? opaque는 안그런가?)

```swift
struct FlippedShape<T: Shape>: Shape {
    var shape: T
    func draw() -> String {
        let lines = shape.draw().split(separator: "\n")
        return lines.reversed().joined(separator: "\n")
    }
}
let flippedTriangle = FlippedShape(shape: smallTriangle)
print(flippedTriangle.draw())
// ***
// **
// *
```


(이 방법은 아래처럼 여러개를 중첩해서 쓸 때 제네릭이 엄청 많아진다는 단점이 있는듯하다)
아래의 `JoinedShape<T: Shape, U: Shape>` 를 통해 2개의 shape를 합칠 수 있습니다.
예를 들어, flipped triangle과 다른 triangle을 합친다면 `JoinedShape<FlippedShape<Triangle>, Triangle>` 처럼 표현할 수 있습니다.

This approach to defining a `JoinedShape<T: Shape, U: Shape>` structure that joins two shapes together vertically, like the code below shows, results in types like `JoinedShape<FlippedShape<Triangle>, Triangle>` from joining a flipped triangle with another triangle.


```swift
struct JoinedShape<T: Shape, U: Shape>: Shape {
    var top: T
    var bottom: U
    func draw() -> String {
        return top.draw() + "\n" + bottom.draw()
    }
}
let joinedTriangles = JoinedShape(top: smallTriangle, bottom: flippedTriangle)
print(joinedTriangles.draw())
// *
// **
// ***
// ***
// **
// *
```

도형 생성에 대한 자세한 정보를 노출하면 전체 반환 유형을 명시해야 하므로 ASCII 아트 모듈의 public 인터페이스에 포함되지 않는 유형이 누출될 수 있습니다.
모듈 내부의 코드는 다양한 방식으로 동일한 모양을 만들 수 있으며, 모양을 사용하는 모듈 외부의 다른 코드는 변환 목록에 대한 구현 세부 정보를 고려할 필요가 없습니다.
JoinedShape나 FlippedShape와 같은 Wrapper type들은 module 사용자에게 중요하지 않으면 보이지 않아야 합니다.
모듈의 public interface은 join과 flip에 같은 동작들로 구성되어야 하고 이 동작들은 또 다른 Shape타입을 리턴해야 한다.

## Returning an Opaque Type

opaque type은 제네릭의 reverse 버전이라고 생각할 수 있다.
제네릭 타입들은 함수를 호출하는 코드가 해당 함수의 매개 변수에 대한 타입을 선택하고 함수 구현에서 추상화된 방식으로 값을 반환하도록 합니다.
예를들어, 아래 함수의 리턴값은 들어오는 인자에 달려있습니다.

```swift
func max<T>(_ x: T, _ y: T) -> T where T: Comparable { ... }
```

max(_:_:)함수를 호출하는 코드에서 x와 y값을 고르면 T에 해당하는 구체적인 타입이 결정된다.
호출하는쪽에서는 Comparable을 따르는 어떤 타입도 사용할 수 있다.
함수 내부에서는 general하게 코드가 작성되어서 넘겨주는 쪽에서 어떤 타입을 넘겨주어도 대응할 수 있다.
max(_:_:)의 구현은 모든 Comparable 타입이 갖고 있는 타입들의 기능만을 사용한다.

이런 역할들은 opaque return type에서 reverse되어진다.
An opaque type lets the function implementation pick the type for the value it returns in a way that’s abstracted away from the code that calls the function.
예를 들어, 아래 함수는 사다리꼴(trapezoid)를 리턴하는데 리턴값이 어떤 타입인지 노출되지 않습니다.

```swift
struct Square: Shape {
    var size: Int
    func draw() -> String {
        let line = String(repeating: "*", count: size)
        let result = Array<String>(repeating: line, count: size)
        return result.joined(separator: "\n")
    }
}

func makeTrapezoid() -> some Shape {
    let top = Triangle(size: 2)
    let middle = Square(size: 2)
    let bottom = FlippedShape(shape: top)
    let trapezoid = JoinedShape(
        top: top,
        bottom: JoinedShape(top: middle, bottom: bottom)
    )
    return trapezoid
}
let trapezoid = makeTrapezoid()
print(trapezoid.draw())
// *
// **
// **
// **
// **
// *
```

makeTrapezoid() 함수의 리턴형은 some Shape이다.
이로인해, return 타입은 특정 구체 타입이 아닌 Shape protocol을 따르는 타입이다. 
따라서 public interface에 필요한 중요 정보만을 노출시키고 특정 타입에 대한 정보를 노출시키지 않는다.
이 구현은 2개의 삼각형과 1개의 사각형을 이용한다. 하지만 함수는 리턴타입을 바꾸지 않고 다양한 방법으로 구현이 바뀔 수 있다.

이 예는 opaque return type이 제네릭의 reverse하다는 것을 보여준다.
makeTrapezoid()는 SHape 프로토콜을 따르는 어떤 리턴타입도 리턴할 수 있다.
이것은 마치 제네릭함수를 호출하는 부분과 비슷하다.
위 함수를 호출하는 코드는 general하게 쓰여져야 한다.(마치 generic 함수의 구현부처럼) 그래서 다른 어떤 Shape 값과도 상호작용할 수 있다.

또한 아래 함수처럼 opaque return type과 generic을 같이 쓸 수도 있다.

제네릭은 함수의 시작을 general하게 만든다면 opaque return type은 함수의 끝을 general하게 만드는 것 같다.
    
```swift
func flip<T: Shape>(_ shape: T) -> some Shape {
    return FlippedShape(shape: shape)
}
func join<T: Shape, U: Shape>(_ top: T, _ bottom: U) -> some Shape {
    JoinedShape(top: top, bottom: bottom)
}

let opaqueJoinedTriangles = join(smallTriangle, flip(smallTriangle))
print(opaqueJoinedTriangles.draw())
// *
// **
// ***
// ***
// **
// *
```

이 예제의 opaqueJoinedTriangles는 위의 joinedTriangles와 같은 역할을 한다.
그러나 위의 예제와는 달리 flip(_:)과 join(_:_:)은 해당 타입을 한번 감싸서 opaque return type으로 리턴한다. 따라서 중간에 등장하는 타입을 보이지 않게 해준다.
두 함수 모두 제네릭 함수인데 타입 파라미터는 해당 정보를 필요로 하는 FlippedShape와 JoinedShape에게 전달된다.

만약 opaque return type을 갖고 있는 함수가 리턴하는 부분이 여러곳에 있다면, 항상 같은 타입을 리턴해야한다.
제네릭 함수의 경우 리턴타입에 제네릭을 사용할 수 있지만 모두 같은 타입이어야 한다.
예를들어 아래와 같은 함수는 에러를 던진다.

```swift
func invalidFlip<T: Shape>(_ shape: T) -> some Shape {
    if shape is Square {
        return shape // Error: return types don't match
    }
    return FlippedShape(shape: shape) // Error: return types don't match
}
```

에러를 안나게 하려면 아래처럼 FlippedShape를 수정하면 된다. 
    
```swift
struct FlippedShape<T: Shape>: Shape {
    var shape: T
    func draw() -> String {
        if shape is Square {
            return shape.draw()
        }
        let lines = shape.draw().split(separator: "\n")
        return lines.reversed().joined(separator: "\n")
    }
}
```

항상 같은 타입을 리턴해야 한다고 해서 opaque return type이 제네릭을 사용하지 못하는 것이 아니다.
Here’s an example of a function that incorporates its type parameter into the underlying type of the value it returns:

```swift
func `repeat`<T: Shape>(shape: T, count: Int) -> some Collection {
    return Array<T>(repeating: shape, count: count)
}
```

[T]가 항상 다른 타입일 수 있지만 some Collection을 통해 하나의 타입으로 리턴할 수 있다.
    
In this case, the underlying type of the return value varies depending on T: Whatever shape is passed it, repeat(shape:count:) creates and returns an array of that shape. Nevertheless, the return value always has the same underlying type of [T], so it follows the requirement that functions with opaque return types must return values of only a single type.

## Differences Between Opaque Types and Protocol Types

opaque type으로 리턴하는 것은 protocol type을 사용하는 것과 매우 비슷합니다.
이 2방식에 차이점은 타입의 identity를 보존하는 지이다.
opaque type은 특정 타입을 알려준다. 비록 이 opaque return type을 갖고있는 함수를 호출하는 쪽에서는 볼 수 없다.
반면 protocol type은 해당 protocol을 따르는 어떠한 타입도 가리킬 수 있다.
일반적으로 말해서 protocol type은 더 유연하게 여러 타입들을 저장할 수 있고 opaque type은 특정 타입을 갖게 해준다.

예를 들어, 아래 함수에서 opaque type 대신 protocol type을 리턴한다.

```swift
func protoFlip<T: Shape>(_ shape: T) -> Shape {
    return FlippedShape(shape: shape)
}
```

여기 protoFlip과 위의 flip은 같은 바디를 갖고 있고 똑같은 타입을 리턴한다.
flip과는 달리 protoType은 Shape 프로토콜을 따르는 그 어떤것도 리턴할 수 있다.
protoFlip은 flip보다 더 느슨하게 API 상호작용을 하게 된다.
여러 타입을 리턴할 수 있는 유연성을 갖게 된다.

```swift
func protoFlip<T: Shape>(_ shape: T) -> Shape {
    if shape is Square {
        return shape
    }

    return FlippedShape(shape: shape)
}
```
이 버전은 Square나 FlippedShape를 리턴할 수 있는데 이는 어떤 도형이 인자로 들어오는지에 달렸다.
이 함수를 통해 만들어진 2개의 flipped shape는 완전히 다른 타입이 될 수도 있다.
protoFlip에서처럼 더 적은 정보를 리턴하는 것은 타입 정보에 따라 동작하는 여러 기능들을 사용할 수 없음을 의미한다.
예를 들어 아래 == operator는 사용할 수 없다.

```swift
let protoFlippedTriangle = protoFlip(smallTriangle)
let sameThing = protoFlip(smallTriangle)
protoFlippedTriangle == sameThing  // Error
```

마지막 줄의 에러는 몇가지 이유 때문이다.
Shape 프로토콜에는 == operator가 명시되지 않았다.
== operator를 추가하려고 한다면 또 다른 문제를 만나게 되는데 이는 lhs와 rhs의 타입을 알아야 한다.
이런 종류의 operator는 Self 타입을 인자로 받는 경우가 종종 있다. 이 Self는 프로토콜을 따르는 구체 타입과 매칭된다.

프로토콜 타입을 리턴타입으로 사용하는 것은 해당 프로토콜을 따르는 어떤 타입도 리턴타입이 될 수 있는 유연함을 제공한다.
그러나 그 유연함의 댓가로 리턴 타입의 특정 기능들을 사용할 수 없게 된다.
이 에를 어떻게 == operator가 사용되지 못하는지를 보여준다. 이는 프로토콜 타입에 의해 보존되지 않는 특정 타입정보에 달려잇다.(프로토콜에 명시되지 않으면 못쓴다.)


이 프로토콜 타입 방식의 또 다른 문제는 shape의 변환이 중첩되지 않는다.
삼각형의 flipping 결과는 Shape 타입이다. 그리고 protoFlip은 Shape protocol을 따르는 타입을 인자로 받을 수 있다.
그러나, 프로토콜 타입의 값은 protocol을 따르지 않는다. 즉 protoFlip의 리턴값은 Shape protocol을 따르지 않는다.
이 말은 protoFlip(protoFlip(smallTriangle))과 같은 중첩 변환이 불가능하다. 

이와는 달리 opaque type은 해당 타입의 identity를 보존한다.
스위프트는 associated types을 추론할 수 있는데 이로써 opaque return value를 리턴 값으로 지정할 수 있다.(프로토콜 타입은 불가능한데 반해)


```swift
protocol Container {
    associatedtype Item
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
extension Array: Container { }
```
Container를 함수의 리턴타입으로 사용할 수 없다. 왜냐하면 프로토콜은 associatedtype을 갖고 있기 때문이다. 
    
```swift
// Error: Protocol with associated types can't be used as a return type.
func makeProtocolContainer<T>(item: T) -> Container {
    return [item]
}

// Error: Not enough information to infer C.
func makeProtocolContainer<T, C: Container>(item: T) -> C {
    return [item]
}
```

opaque type을 리턴 타입으로 사용함으로써 API의 요구하는대로 작성할 수 잇다. 

```swift
func makeOpaqueContainer<T>(item: T) -> some Container {
    return [item]
}
let opaqueContainer = makeOpaqueContainer(item: 12)
let twelve = opaqueContainer[0]
print(type(of: twelve))
// Prints "Int"
```
    
twelve의 타입은 Int가 된다. 왜냐하면 opaque type에서는 타입 추론이 일어나기 때문이다.
makeOpaqueContainer(item:)에서 표현되는 opaque container는 [T]이다. 여기서 T는 Int가 되게 되는데 그리하여 Int배열의 리턴형과 associated type인 Item의 자료형은 Int로 추론된다.
Container의 subscript는 Item을 리턴하고 이는 Int를 의미한다.
