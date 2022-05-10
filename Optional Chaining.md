# Optional Chaining

- Optional Chaining은 현재 nil일 가능성이 있는 옵셔널 타입의 프로퍼티, 메서드, 서브 스크립트를 쿼리하고 호출하는 프로세스
- 값이 있으면 호출에 성공; 값이 nil이면 nil 반환
- 여러 개의 쿼리를 연결할 수 있으며, 하나라도 nil이면 전체 체인이 실패한다.

# **Optional Chaining as an Alternative to Forced Unwrapping**

- 옵셔널 체이닝은 값 뒤에 물음표(?)를 붙여서 표현 가능
- 옵셔널을 사용할 수 있는 값: 프로퍼티, 메소드, 서브 스크립트
- 강제 언래핑(! 사용)과 문법 유사
    - 차이점: 값이 없으면 강제 언래핑은 런타임 에러가 발생하지만, 옵셔널 체이닝은 nil 반환
- 옵셔널 체이닝에 의해 nil 값이 호출될 수 있기 대문에 옵셔널 체이닝의 값은 항상 옵셔널
    - 옵셔널 값을 반환하지 않는 값(프로퍼티, 메소드, 서브 스크립트)를 호출하더라도 옵셔널 체이닝에 의해 옵셔널 값으로 반환
- 옵셔널 체이닝의 호출 결과는 예상되는 반환 값과 동일한 타입이지만 옵셔널로 래핑함
    - ex) 예상되는 결과로 Int 반환; 옵셔널 체이닝을 사용하면 Int? 반환

# **Defining Model Classes for Optional Chaining**

- 옵셔널 체이닝을 한 레벨이 아닌 여러 레벨로 사용할 수 있다. (multilevel optional chaining)
- 복잡한 구조의 모델에서 하위 프로퍼티에 접근해서 값이 있는지 확인할 수 있다.
    - 체인 중 하나라도 nil을 반환하면 실패한다.

```swift
class Person {
    var residence: Residence?
}

class Residence {
    var rooms: [Room] = []
    var numberOfRooms: Int {
        return rooms.count
    }
    subscript(i: Int) -> Room {
        get {
            return rooms[i]
        }
        set {
            rooms[i] = newValue
        }
    }
    func printNumberOfRooms() {
        print("The number of rooms is \(numberOfRooms)")
    }
    var address: Address?
}

class Room {
    let name: String
    init(name: String) { self.name = name }
}

class Address {
    var buildingName: String?
    var buildingNumber: String?
    var street: String?
    func buildingIdentifier() -> String? {
        if let buildingNumber = buildingNumber, let street = street {
            return "\(buildingNumber) \(street)"
        } else if buildingName != nil {
            return buildingName
        } else {
            return nil
        }
    }
}
```

- buildingNumber, street 둘 다 있으면 반환 실패하면 buildingName 반환, 이것도 실패하면 nil 반환

# **Accessing Properties Through Optional Chaining**

- 강제 언래핑의 대안으로 옵셔널 체이닝을 사용하면, 옵셔널 값의 속성에 접근하고 해당 속성이 접근이 성공적인지 확인할 수 있다.
- 옵셔널 체이닝을 이용해 프로퍼티에 접근할 수 있다.

```swift
let john = Person()
if let roomCount = john.residence?.numberOfRooms {
    print("John's residence has \(roomCount) room(s).")
} else {
    print("Unable to retrieve the number of rooms.")
}
// Prints "Unable to retrieve the number of rooms."
```

- residence?가 nil이기 때문에 옵셔널 체이닝 결과 nil 호출

```swift
func createAddress() -> Address {
    print("Function was called.")

    let someAddress = Address()
    someAddress.buildingNumber = "29"
    someAddress.street = "Acacia Road"

    return someAddress
}
john.residence?.address = createAddress()
```

- 실행 결과 "Function was called.”가 출력되지 않음 → 메소드가 아예 실행되지 않았다.
    - john.residence?가 nil이기 때문에 할당 불가능

# **Calling Methods Through Optional Chaining**

- 옵셔널 체이닝을 사용해 메소드를 호출할 수 있다.
- 메소드에서 리턴 값을 정의하지 않아도 작업 수행 가능

```swift
func printNumberOfRooms() {
    print("The number of rooms is \(numberOfRooms)")
}
```

- 리턴 값이 명시되지 않았지만 자동으로 Void? 타입을 갖는다.
    - 옵셔널 체이닝을 사용하여 옵셔널 값을 반환

```swift
if john.residence?.printNumberOfRooms() != nil {
    print("It was possible to print the number of rooms.")
} else {
    print("It was not possible to print the number of rooms.")
}
// Prints "It was not possible to print the number of rooms."
```

- 다음과 같이 nil로 비교해서 제대로 메소드가 실행이 되는지 확인 가능

```swift
if (john.residence?.address = someAddress) != nil {
    print("It was possible to set the address.")
} else {
    print("It was not possible to set the address.")
}
// Prints "It was not possible to set the address."
```

- 프로퍼티를 세팅하는 것도 다음과 같은 방식으로 확인 가능

# **Accessing Subscripts Through Optional Chaining**

- 옵셔널 체이닝을 이용해 옵셔널 값을 서브스크립트로 접근 가능
- 서브스크립트의 대괄호 앞에 ?가 붙는다. (항상 옵셔널 표현식 바로 뒤에 ?가 붙는다.)

```swift
if let firstRoomName = john.residence?[0].name {
    print("The first room name is \(firstRoomName).")
} else {
    print("Unable to retrieve the first room name.")
}
// Prints "Unable to retrieve the first room name."
```

- 할당도 가능

```swift
john.residence?[0] = Room(name: "Bathroom")
// residence가 nil이라 실패 
```

## **Accessing Subscripts of Optional Type**

서브스크립트가 옵셔널 값을 반환하는 경우(딕셔너리 타입) 서브 스크립트를 닫는 대괄호 뒤에 ?를 추가하여 옵셔널 값을 연결한다.

```swift
var testScores = ["Dave": [86, 82, 84], "Bev": [79, 94, 81]]
testScores["Dave"]?[0] = 91
testScores["Bev"]?[0] += 1
testScores["Brian"]?[0] = 72
// the "Dave" array is now [91, 82, 84] and the "Bev" array is now [80, 94, 81]
```

- 딕셔너리가 key 값을 잘못 넣어서 반환에 실패해도 오류가 나지 않고 nil이 반환된다.

# **Linking Multiple Levels of Chaining**

- 옵셔널 체이닝이 여러 단계에 걸쳐 연결될 수 있다.
- 여러 단계가 겹쳐도 더 옵셔널해지지는 않는다. (상위 값과 하위 값이 다 옵셔널이라고 옵셔널+가 되지 않는다.)

다른 말로 하자면

- 검색하려는 타입이 옵셔널이 아닌 경우에도 옵셔널 체이닝으로 옵셔널 타입이 된다.
- 검색하려는 타입이 이미 옵셔널인 경우 옵셔널 체이닝 때문에 더 심한 옵셔널이 되지 않는다.

따라서

- 옵셔널 체이닝을 통해 Int 값을 검색하려고 하면 체인 수준에 관계 없이 항상 Int?가 반환된다.
- 마찬가지로 옵셔널 체이닝을 통해 Int? 값을 검색하려고 하면 여러 단계에 상관없이 Int?가 반환된다.

```swift
let johnsHouse = Residence()
johnsHouse.rooms.append(Room(name: "Living Room"))
johnsHouse.rooms.append(Room(name: "Kitchen"))
john.residence = johnsHouse

if let johnsStreet = john.residence?.address?.street {
    print("John's street name is \(johnsStreet).")
} else {
    print("Unable to retrieve the address.")
}
// Prints "Unable to retrieve the address."
```

- 2단계의 옵셔널 체이닝으로 연결되어 있다.
- residence?는 유효하지만 address?의 값이 nil이므로 옵셔널 체이닝은 실패한다.
- 옵셔널 체이닝이 성공할 경우, street는 String? 타입의 값을 반환한다.

# **Chaining on Methods with Optional Return Values**

- 옵셔널 체이닝에서 반환 값이 있는 메소드를 호출할 수 있다.

```swift
if let buildingIdentifier = john.residence?.address?.buildingIdentifier() {
    print("John's building identifier is \(buildingIdentifier).")
}
// Prints "John's building identifier is The Larches."
```

- buildingIdentifier()의 반환 값은 String?이다

```swift
if let beginsWithThe =
    john.residence?.address?.buildingIdentifier()?.hasPrefix("The") {
    if beginsWithThe {
        print("John's building identifier begins with \"The\".")
    } else {
        print("John's building identifier doesn't begin with \"The\".")
    }
}
// Prints "John's building identifier begins with "The"."
```

- 다음과 같이 메소드의 반환 값에 옵셔널 체이닝 할 수 있다.
