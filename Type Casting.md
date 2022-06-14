# Type Casting

타입 캐스팅은 “is” 또는 “as” 연산자로 구현

[타입 캐스팅 사용 이유]

- 인스턴스 타입을 확인하기 위해
- 인스턴스를 다른 슈퍼클래스 또는 서브클래스로 다루기 위해
- 타입이 프로토콜을 준수하는지 여부를 확인하기 위해

# **Defining a Class Hierarchy for Type Casting**

클래스 및 서브클래스의 계층 구조에 타입 캐스팅을 사용할 수 있다.

- 인스턴스의 타입을 체크하고, 같은 계층 구조 내에서 다른 클래스로 인스턴스를 캐스트 할 수 있다.

<예시>

```swift
class MediaItem {
	var name: String
	init(name: String) {
		self.name = name
	}
}
```

- MediaItem: Base Class

```swift
class Movie: MediaItem {
	var director: String
	init(name: String, director: String) {
		self.director = director
		super.init(name: name)
	}
}

class Song: MediaItem {
	var artist: String
	init(name: String, artist: String) {
		self.artist = artist
		super.init(name: name)
	}
}
```

- MediaItem의 서브 클래스들

```swift
let library = [
	Movie(name: "Casablanca", director: "Michael Curtiz"),
	Song(name: "Blue Suede Shoes", artist: "Elvis Presley"),
	Movie(name: "Citizen Kane", director: "Orson Welles"),
	Song(name: "The One And Only", artist: "Chesney Hawkes"),
	Song(name: "Never Gonna Give You Up", artist: "Rick Astley")
]
// the type of "library" is inferred to be [MediaItem]
```

- 2개의 Movie 인스턴스와 3개의 Song 인스턴스 포함
- library 배열의 타입은 초기화되는 콘텐츠에 의해 추론된다.
- Swift 타입 검사기는 Movie와 Song이 공통된 슈퍼클래스로 MediaItem을 가지고 있다고 추론하므로, library 배열의 타입을 [MediaItem]으로 추론한다.
- 배열에서 어떤 값을 꺼냈을 때, 그 값의 타입은 MediaItem이다. Movie 또는 Song 타입 지정을 위해서는 타입을 체크하거나 다운 캐스팅 해야 한다.

# **Checking Type**

타입 검사 연산자(is)를 사용하여 인스턴스가 특정 하위 클래스 타입인지를 확인한다.

- true/false로 반환

```swift
var movieCount = 0
var songCount = 0

for item in library {
	if item is Movie {
		movieCount += 1
	} else if item is Song {
		songCount += 1
	}
}

print("Media library contains \(movieCount) movies and \(songCount) songs")
// Prints "Media library contains 2 movies and 3 songs"
```

- Movie/Song 타입 개수 확인 가능

# **Downcasting**

특정 클래스 타입의 상수나 변수는 실제로 서브클래스의 인스턴스를 참조할 수도 있다.

→ 타입 캐스트 연산자(as? or as!)를 사용해 다운캐스팅 할 수 있다.

다운캐스팅은 실패할 수 있기 때문에, 2가지 형태로 제공된다.

- 조건 형식(`as?`): 옵셔널 타입을 리턴 / 특정 타입이 확실치 않을 때 사용
    
    → 실패할 경우 nil 반환
    
- 강제 형식(`as!`) : 결과를 강제로 unwrapping해서 리턴 / 특정 타입이 확실할 때 사용
    
    → 실패할 경우 런타임 에러 발생 
    

```swift
for item in library {
	if let movie = item as? Movie {
		print("Movie: \(movie.name), dir. \(movie.director)")
	} else if let song = item as? Song {
		print("Song: \(song.name), by \(song.artist)")
	}
}

// Movie: Casablanca, dir. Michael Curtiz
// Song: Blue Suede Shoes, by Elvis Presley
// Movie: Citizen Kane, dir. Orson Welles
// Song: The One And Only, by Chesney Hawkes
// Song: Never Gonna Give You Up, by Rick Astley
```

- 배열의 항목이 Movie or Song이기 때문에 as? 조건 형식을 사용하는 것이 적절
    - 다운캐스팅을 시도할 때, 옵셔널 값을 리턴
    - 옵셔널 바인딩을 통해 실제 값이 있는 체크 (if let ~ 구문)

<aside>
💡 캐스팅은 실제로 인스턴스를 수정하거나 값을 변경하지 않는다.
기본 인스턴스는 그대로 유지되며, 단순히 해당 인스턴스가 캐스팅된 타입의 인스턴스로 처리 및 접근된다.

</aside>

# **Type Casting for Any and AnyObject**

- Any: 함수 타입을 포함한 모든 타입의 인스턴스를 나타낸다.
- AnyObject: 모든 클래스 타입의 인스턴스를 나타낸다.

→ Any와 AnyObject가 제공하는 동작 및 기능이 명시적으로 필요한 경우에만 사용해라

→ 항상 코드 내에서 타입을 구체적으로 지정하는 것이 좋다.

```swift
var things: [Any] = []

things.append(0)
things.append(0.0)
things.append(42)
things.append(3.14159)
things.append("hello")
things.append((3.0, 5.0))
things.append(Movie(name: "Ghostbusters", director: "Ivan Reitman"))
things.append({ (name: String) -> String in "Hello, \(name)" })

for thing in things {
    switch thing {
    case 0 as Int:
        print("zero as an Int")
    case 0 as Double:
        print("zero as a Double")
    case let someInt as Int:
        print("an integer value of \(someInt)")
    case let someDouble as Double where someDouble > 0:
        print("a positive double value of \(someDouble)")
    case is Double:
        print("some other double value that I don't want to print")
    case let someString as String:
        print("a string value of \"\(someString)\"")
    case let (x, y) as (Double, Double):
        print("an (x, y) point at \(x), \(y)")
    case let movie as Movie:
        print("a movie called \(movie.name), dir. \(movie.director)")
    case let stringConverter as (String) -> String:
        print(stringConverter("Michael"))
    default:
        print("something else")
    }
}

// zero as an Int
// zero as a Double
// an integer value of 42
// a positive double value of 3.14159
// a string value of "hello"
// an (x, y) point at 3.0, 5.0
// a movie called Ghostbusters, dir. Ivan Reitman
// Hello, Michael
```

- Any or AnyObject 타입의 상수 또는 변수의 유형을 검색하기 위해서는, switch case문에 is or as 패턴을 사용할 수 있다.

 

<aside>
💡 Any 타입은 옵셔널 타입을 포함한 모든 타입을 나타낸다.
스위프트는 Any 타입이 기대되는 곳에 옵셔널 값을 사용하면 경고를 표시한다.
Any 타입으로서 옵셔널 값을 사용하고 싶다면, 아래와 같이 옵셔널 값을 Any로 명시적 캐스트 한다.

</aside>

```swift
let optionalNumber: Int? = 3
things.append(optionalNumber)        // Warning
things.append(optionalNumber as Any) // No warning
```
