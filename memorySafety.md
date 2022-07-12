# Memory Safety

스위프트는 기본적으로 코드에서 발생할 수 있는 unsafe한 동작들을 방지 한다.
예를 들어, 변수가 초기화 되어야만 사용할 수 있고, 메모리가 deallocated 되기전에 접근이 안되고, 배열에서 indece가 out-of-bounds 에러가 나는지 체크해준다.
스위프트는 여러개의 접근이 한 메모리 지역을 접근하더라도 코드가 메모리에 배타적으로 접근하게함으로써 메모리 충돌이 일어나지 않도록 한다.
스위프트는 자동적으로 메모리 관리를 하기 때문에 대부분의 경우 메모리에 대해 신경 쓸 필요가 없다.
코드 내에 충돌이 있다면, 컴파일 타임이나 런타임에 에러를 받을 수 있다.


## Understanding Conflicting Access to Memory

변수에 값을 지정하거나 함수에 인자를 넘겨줄 때 메모리 접근이 일어난다.
예를들어, 다음 코드에서는 읽기, 쓰기 메모리 접근이 일어난다.
```swift
// A write access to the memory where one is stored.
var one = 1

// A read access from the memory where one is stored.
print("We're number \(one)!")
```

다른 코드에서 같은 메모리 location에 동시에 접근하면 메모리 충돌이 일어날 수 있다.
한 메모리 location에 여러 접근을 한다면 예상할 수 없는, 일관적이지 않은 동작이 일어날 수 있다.
스위프트에서는, 하나의 값을 바꾸기 위해 여러줄의 코드가 실행될 수 있는데 이 시간동안 같은 값을 접근하려는 시도가 있을 수 있다.


![](https://docs.swift.org/swift-book/_images/memory_shopping_2x.png)
가계부로 빗댄 메모리 충돌 설명

- 가계부 수정중에 Total에 접근한다면 시기에 따라서 다른 값을 얻을 수 있게 되는 문제가 있다.

>NOTE
만약 concurrent나 multithreade code를 작성해 본적이 있다면 이런 메모리 충돌에 익숙할 것이다.
그러나 메모리 충돌이 concurrent나 multithreade 코드가 아니더라도 single thread에서도 발생할 수 있다.
만약 싱글 스레드에서 메모리 충돌을 겪는다면, 스위프트에서는 컴파일 타임이나 런타임에서 에러 메세지를 받아볼 수있다.(에러메세지 없이 에러가 나지는 않을 거라는 뜻인듯)
multithreaded 코드에서 "Thread Sanitizer를 사용하면 thread를 넘나드는 메모리 충돌을 감지할 수 있다."

## 메모리 접근의 특징
메모리 충돌을 이해하기 위해서는 메모리 접근의 3가지 특징을 알 필요가 있다.
1. 메모리가 읽기 접근인지 쓰기 접근인지
2. 접근 시간
3. 접근되는 메모리 location

특히, 다음 상황을 모두 만족시키는 2가지 접근이 일어난다면 메모리 충돌이 발생할 수 있다.

- 쓰기 접근이거나 nonatomic 접근 중 하나를 만족

- 접근들이 한가지 지역을 접근

- 접근 시간이 서로 겹침

읽기와 쓰기 접근의 차이는 명확한데 쓰기 접근은 해당 메모리 지역을 바꾼다.(읽기 접근은 바꾸지 않음)
메모리 지역(location in memory)는 무엇이 접근될 지를 뜻한다.(예를 들어, 변수, 상수, property)

메모리 접근 시간은 순간적이거나(instantaneous) long-term일 수 있다.

아토믹 오퍼레이션이란 C 아토믹 오퍼레이션을 뜻한다. 
C 아토믹 오퍼레이션의 목록을 보고 싶다면 stdatomic(3) 메뉴얼 페이지를 참고



접근이 순간적(instantaneous)하다는 것은 해당접근 끝날때까지 다른 코드가 실행되지 않음을 뜻한다.
따라서 2개의 순간적(instantaneous)한 코드는 동시에 실행될 수 없다.
대부분의 메모리 접근은 순간적(instantaneous)하다.
예를들어 아래 코드에서의 읽기 접근과 쓰기 접근은 전부 instantaneous합니다.


```swift
func oneMore(than number: Int) -> Int {
    return number + 1
}

var myNumber = 1
myNumber = oneMore(than: myNumber)
print(myNumber)
// Prints "2"
```

롱텀 접근은 다른 롱텀 접근나 instantaneous접근 코드와 시간이 중첩될 수 있다.

중첩되는 접근이 주로 일어나는 코드는 "함수나 메서드에서 in-out 파라미터를 사용"하거나 "스트럭트에서 mutating 메소드"를 사용하는 것이다.

이제 하나씩 살펴보자

## Conflicting Access to In-Out Parameters

함수는 모든 in-out parameter에 대해 롱텀 write access 를 갖습니다.

in-out 파라미터에 대한 쓰기 접근은 모든 non-in-out파라미터가 evaluated 된 뒤부터 함수의 호출이 끝날때까지 유지됩니다.
만약에 여러개의 in-out 파라미터가 있다면 파라미터에 쓰여있는 순서대로 쓰기 접근이 시작됩니다.

롱텀 쓰기 접근으로 인한 쓰기 서 in-out파라미터로 넘어온 값의 오리지널 값을 접근할 수 없다.

```swift
var stepSize = 1

func increment(_ number: inout Int) {
    number += stepSize
}

increment(&stepSize)
// Error: conflicting accesses to stepSize
```
- 위 코드에서 함수로 넘어온 inout parameter에 대해 쓰기 접근이 시작되는데 함수내부에서 stepsize에 대한 읽기 접근이 일어나므로 각 접근이 중첩되므로 충돌이 일어납니다. 
![](https://docs.swift.org/swift-book/_images/memory_increment_2x.png)

이 충돌을 해결하는 방법은 stepSize를 복사해버리면 됩니다.
```swift
// Make an explicit copy.
var copyOfStepSize = stepSize
increment(&copyOfStepSize)

// Update the original.
stepSize = copyOfStepSize
// stepSize is now 2
```
이렇게 되면 쓰기접근이 시작되기 전에 읽기 접근이 끝나기 때문에 충돌이 일어나지 않는다.

다른 문제점으로는 여러개의 in-output 파라미터로 하나의 값을 넘기면 충돌이 생길 수 있다.
For example:


```swift
func balance(_ x: inout Int, _ y: inout Int) {
    let sum = x + y
    x = sum / 2
    y = sum - x
}
var playerOneScore = 42
var playerTwoScore = 30
balance(&playerOneScore, &playerTwoScore)  // OK
balance(&playerOneScore, &playerOneScore)
// Error: conflicting accesses to playerOneScore
```
다른 인자 메모리 위치에 접근하는 첫번째 경우 시간이 중첩되지만 접근하는 메모리 위치가 다르기때문에 충돌이 일어나지 않습니다.
이와 달리, 두번째 예시는 같은 곳을 접근하기때문에 충돌이 일어납니다.


## Conflicting Access to self in Methods

구조체의 뮤테이팅 메소드는 메소드가 호출된 시간 동안 self에 대해 쓰기 접근을 하게 됩니다.


```swift
struct Player {
    var name: String
    var health: Int
    var energy: Int

    static let maxHealth = 10
    mutating func restoreHealth() {
        health = Player.maxHealth
    }
}
```
mutatin methos 인 restoreHealth()는 메소드가 호출된 순간부터 리턴될때까지 self에 대해 쓰기 접근을 합니다.
이 경우, 접근에 대해 중첩이 되는 코드가 없다.

하지만, 아래처럼 in-out 파라미터로 다른 Player의 인스턴스를 파라미터로 받는다면 접근의 중첩이 생길 수 있습니다.

```swift
extension Player {
    mutating func shareHealth(with teammate: inout Player) {
        balance(&teammate.health, &health)
    }
}

var oscar = Player(name: "Oscar", health: 10, energy: 10)
var maria = Player(name: "Maria", health: 5, energy: 10)
oscar.shareHealth(with: &maria)  // OK
```
위의 코드는 Oscar.shareHealth가 maria의 인스턴스를 파라미터로 받기 때문에 충돌이 일어나지 않습니다.
위 함수는 oscar에 대해 쓰기 접근이 일어나고(mutating 메소드이기때문에) in-out 파라미터로 받은 maria에 대해서도 동일한 시간동안 쓰기 접근이 일어난다.
하지만 메모리가 다른 지역을 접근하기 때문에 충돌이 일어나지 않습니다.
![](https://docs.swift.org/swift-book/_images/memory_share_health_maria_2x.png)

그러나 oscar.shareHealth가 oscar를 in-out parameter로 받는다면 충돌이 일어날 수 있다.
```swift
oscar.shareHealth(with: &oscar)
// Error: conflicting accesses to oscar
```

위 코드에서는 같은 곳을 같은 시간에 접근하기 때문에 충돌이 일어날 수 있다.

![](https://docs.swift.org/swift-book/_images/memory_share_health_oscar_2x.png)

## Conflicting Access to Properties

구조체의 프로퍼티나 튜플의 element는 값 타입이기때문, 부분을 수정해도 전체가 다 수정된다.
이 말은 프로퍼티들 중 하나에 대한 읽기 쓰기 접근은 전체 타입에 대한 읽기, 쓰기 접근이라고 한다.

예를 들어, 다음과 같은 코드는 충돌을 일으킨다.

```swift
var playerInformation = (health: 10, energy: 20)
balance(&playerInformation.health, &playerInformation.energy)
// Error: conflicting access to properties of playerInformation
```

위의 케이스에서 balance()에 in-out 파라미터로 playInformation의 health와 energy가 넘어오는데 각각 전체 타입에 쓰기 접근이 일어난다.
health에 의해서 접근된 메모리와 energy에 의해 접근된 메모리가 같은곳이고, 같은 시간이기 때문에 충돌을 일으킨다.

위와 비슷한 예가 전역변수로 선언된 구조체의 프로퍼티 접근에서도 일어납니다.

```swift
var holly = Player(name: "Holly", health: 10, energy: 10)
balance(&holly.health, &holly.energy)  // Error
```

사실 구조체 프로터피에 대한 대부분의 접근이 안전하게 중첩된다.(충돌이 일어나지 않는다.)

예를들어, 아래의 예와 같이 타입을 글로벌 변수로 선언하지 않고 로컬 변수로 선언한다면 구조체에 저장 프로퍼티에 중첩된 접근을 하더라도 컴파일러가 안전하다고 여길 수 있게 된다.

```swift
func someFunction() {
    var oscar = Player(name: "Oscar", health: 10, energy: 10)
    balance(&oscar.health, &oscar.energy)  // OK
}
```

컴파일러가 위와 같은 상황은 메모리 세이프티하다고 보장해주게 된다. 왜냐하면 두개의 저장프로퍼티가 어떤식으로든 서로 연관이 없기 때문이다.

구조체에 프로퍼티에 대한 중첩 접근을 제한한다고 해서 항상 메모리 세이프티한 것은 아니다.
메모리 세이프티 이상으로 해당 메모리에 대한 접근이 배타적이어야 한다. 이 말은 어떠한 접근이 메모리 세이프티 하지만 메모리에 배타적으로 접근하지 않을 수 있다는 말이다.
스위프트는 위와 같은 배타적이지 않은 메모리 접근을 메모리 세이프티하다고 보장해줄 수 있다.
스위프트가 구조체 프로퍼티에 대한 중첩 접근을 메모리 세이프한것으로 판정해 주기 위해서는 다음과 같은 조건이 필요하다.

- 오직 인스턴스의 저장 프로퍼티에만 접근해야 한다. 계산 프로퍼티나 클래스 프로퍼티는 안된다.

- 구조체는 로컬 변수로 선언되어야 한다. 전역 변수로 선언되면 안된다.

- 구조체는 클로저에 의해 캡처되거나 캡처되더라도 nonescaping 클로저에 의해서 캡처되어야 한다.

만약 위와 같은 조건이 성립되지 않으면 접근을 허용하지 못한다.
