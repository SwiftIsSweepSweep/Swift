# [2022] Explore structured concurrency in Swift

### 등장 배경

- Swift 5.5
- structured concurrency 라는 개념을 바탕으로 만들어짐
    
    struced concurrency 라는 개념은 사실 structed programming을 기반으로 합니다.
    
    그렇다면 structured programming이란 무엇일까요?
    
    프로그램의 코드가 순서대로 실행되는 것이 아니라 한곳에서 다른 곳으로 뛰어넘어서 실행될 수 있었기 때문에 이해하기 어려웠습니다. 
    
    [In the early days of computing, programs were hard to read](https://developer.apple.com/videos/play/wwdc2021-10134/?time=34) [because they were written as a sequence of instructions,](https://developer.apple.com/videos/play/wwdc2021-10134/?time=37) [where control-flow was allowed to jump all over the place.](https://developer.apple.com/videos/play/wwdc2021-10134/?time=40)
    
    요즘 프로그래밍 언어들은 코드를 깔끔하게 만들기 위해서 structed programming을 지향하기 때문에, 요즘은 이러한 형태의 코드를 잘 볼 수 없습니다.
    
- completion handler의 문제점:
    - callback 중첩 → 코드 복잡함
    - 개발자가 completion handler를 호출하는 것을 까먹으면 예상치 못한 버그가 발생할 수 있다.

### 병렬 프로그램을 만들 때

- ⇒ Task를 이용하면 됩니다!
- Task는 async function과 같이 Swift에서 비동기 프로그래밍을 지원하기 위해 등장했습니다.

## 병렬 프로그램을 만들 때

⇒ Task를 이용하면 됩니다!

1. Task는 async function와 같이 Swift에서 비동기 프로그래밍을 지원하기 위해 등장했습니다.
2. Task는 비동기 프로그램을 실행할 수 있는 새로운 실행 context를 제공합니다. 
(여기서 실행 context란 동시에 동작할 수 있는 실행 흐름들을 의미합니다!) 
⇒ Task는 비동기 프로그램을 실행하는 새로운 스레드를 제공합니다.
3. 모든 Task는 다른 Task와 동시에 실행될 수 있습니다. 
4. Task를 생성하는 순간 안전하고, 효율적으로 실행될 수 있도록 자동으로 스케줄링 됩니다.
5. 컴파일러에서 버그가 있는 Task인지 점검해줍니다.
6. 마지막으로 async function을 호출한다고 해서 Task가 생성되는 것은 아닙니다. 
우리가 async function을 호출했을 때, 자동으로 새로운 스레드에서 실행될 것이라고 생각하기 쉽지만 그렇지 않습니다. 병렬적으로 동작하도록 만들기 위해서는 명시적으로 Task를 만들어야 합니다.
7. Task는 cancellation, priority, 관련 속성을 가지며,  task-local variables 을 갖습니다.

![스크린샷 2022-06-15 오후 11.24.19.jpg](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.24.19.jpg)

## Task 종류

### `async let binding` 기법으로 생성하는 방법

- 일반적인 let 사용문법은 다음과 같습니다.

![Untitled](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/Untitled.png)

- = 오른쪽 부분을 실행하고, 그 결과를 result에 담습니다. 이어서 다음 명령어를 실행합니다.
- 만약 = 오른쪽 부분이 `URLSession.shared.data()` 처럼 다운로드 받는 로직이어서 시간이 오래 걸리는 로직이라면, 우리는 프로그램이 효율적으로 동작하도록 만들기 위해, 다운로드 로직이 실행될 동안 다른 작업이 처리되도록 구현하고 싶을 수 있습니다.
- ⇒ 이럴 때 사용하는 것이 `async let binding` 입니다.

### `async let binding` 동작 순서

![스크린샷 2022-06-15 오후 11.31.09.jpg](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.31.09.jpg)

- Child Task를 하나 생성하며, 이 Task가 = 오른쪽 (여기서는 데이터를 다운받는 URLSession.shared.data)를 실행합니다.
- 현재 Task(현재 실행 스레드라고 보면된다.)는 Child Task의 결과가 result 변수에 담길것이라고 표시를 해둡니다.
- 그리고 현재 Task는 다음 명령어를 실행하다가, result 변수를 사용하는 시점이 오면, child task의 작업이 끝날 때까지 대기합니다.

## 두 코드의 차이점이 무엇일까요?

### 첫번째

![스크린샷 2022-06-21 오후 8.11.18.jpg](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.11.18.jpg)

### 두번째

![스크린샷 2022-06-21 오후 8.14.14.jpg](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.14.14.jpg)

- 정답
    1. `try await`
        1. 첫번째 `URLSession.shared.data`가 완료된 후에 두번째 `URLSession.shared.data` 가 실행됩니다.
    2. `async let`
        1. await을 
        2. 내부에서 Child Task 생성되고, 이 Child Task에서 다운로드 작업이 일어나기 때문에, try await을 쓰지 않아도 됩니다.
        3. 대신, Child Task의 결과를 담고 있는 `data`, `metadata` 를 사용하는 곳에서 try await을 호출해주어야 합니다.

### 왜 Structured Concurrency라고 하는가?

- Task들이 tree를 이룬다.
- 부모 Task - 자식 Task 존재
    - 자식 Task는 부모 Task의 속성들(우선순위, 실행취소)를 그대로 물려 받습니다.
- async function을 호출하면, 자식 Task가 생성되는 것이 아니라 동일한 Task에서 실행됩니다.
하지만 Task내부에서 Task를 생성하면, 이때 부모 Task와 자식 Task가 생겨나게 됩니다.
    
    ![스크린샷 2022-06-21 오후 8.22.31.jpg](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.22.31.jpg)
    
- ⇒ Task는 함수와 tree를 구성하지 않습니다. Task끼리만 tree를 구성합니다.
다만, 함수 내부에서 생성된 Task는 함수가 실행종료되면 같이 종료될 수 있습니다.
- tree가 구성되는 방식
    - 부모 task와 자식 task간에 링크로 연결이 됩니다.
- 이렇게 링크로 연결됨으로써, 부모 task는 모든 자식 task가 완료된 후에만 종료될 수 있습니다.
    - → 비동기 프로그램이 비정상적으로 동작하는 것을 막아줍니다.
    
    ![스크린샷 2022-06-21 오후 8.25.55.jpg](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.25.55.jpg)
    
    - `try await metadata` 여기서 에러가 발생
    `fetchOneThumbnail`은 에러를 throw하면서 종료되어야 한다.
    - 하지만 바로 종료하지 않고 `URLSession.shared.data(for:imageReq)` 를 실행하고 있는 task를 cancelled로 표시하고, 이 task가 종료되면 그때서야 `fetchOneThumbnail`함수를 끝냅니다.
- task를 cancelled로 표시하는 것이 task를 중단시키지 않습니다.
cancelled로 표시되어도 task는 그대로 실행이 되고, task 실행 결과만 사용되지 않고 버려지게 됩니다.
- task가 cancelled되었을 때, 모든 자식 task는 자동으로 cancell 될 것입니다.

<aside>
💡 이것만 알고가기‼️
- structed concurrency인 이유?
Task간에 부모 자식 관계를 이루며, 자식 Task가 모두 종료된 후에야 부모 Task가 종료된다.

- 이렇게 하는 이유?
부모 Task가 종료되었는데 자식 Task가 계속해서 실행된다면(Task leaking)이 비정상적인 프로그래밍 동작을 일으킬 수 있기 때문입니다!

</aside>

## Task 작성시 주의할 점!

- 애플에서는 Task가 취소되었는지 확인하고, Task의 실행을 중단시키는 코드를 꼭 작성할 것을 권고하고 있습니다.
    - Task가 취소되었는지 체크하는 메서드를 제공합니다.
        1. `try Task.checkCancellation`
            - 현재 Task가 cancell됬으면 throw error
        2. `Task.isCancelled`
            1. 현재 Task가 cancell됬으면 return true
- 오래 걸리는 작업을 병렬로 처리하도록 구현할 때도 취소를 염두해두고 API를 구현해야 합니다.
(사용자는 취소될 수 있는 Task를 다시 호출할 수도 있고, ‘작업 중단하기’ 버튼과 같은 것을 클릭했을 때 실행중이던 Task가 바로 종료되는 것으로 오해할 수 있기 때문에!)

## Task의 개수가 지정되지 않았을 때, `TaskGroup` 사용

![스크린샷 2022-06-21 오후 8.40.54.jpg](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.40.54.jpg)

- `try await fetchOneThumnail 덕분에`
- 한번의 loop당 2개의 task(`data` , `metadata`)만 생성된다.
- ⇒ 하지만 thumb nail을 다운받는 모든 Task를 한번에 실행하고 싶다면? `TaskGroup`

### TaskGroup

![스크린샷 2022-06-21 오후 8.47.47.jpg](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.47.47.jpg)

- child task를 그룹짓는 것
- TaskGroup이 종료되면 내부의 모든 Task 종료됨
    - GroupTask와 내부 Task간에 부모 -자식 관계가 형성되기 때문
- TaskGroup에 추가된 Task들은 순서대로 종료❌, 랜덤으로 종료된다.

- 이 코드의 문제점: race condition
- 이 코드는 컴파일 error 발생
    
    b) `thumbnails` 를 여러 task에서 접근해서 수정하기 때문
    
- task를 생성할 때 넘겨주는 closure는 @Sendable Closure이다.
Sendable Closure는 mutable 한 변수를 capture할 수 없습니다.
- 왜냐하면 새로운 Task에서 mutable 한 변수를 수정할 수 있으면, data race를 발생시킬 수 있기 때문입니다.
- ⇒ “Task에서 사용하는 데이터들은 전부 thread-safe해야 합니다.” ex) value 타입, actor

- 해결 방법
    
    ![스크린샷 2022-06-21 오후 8.55.57.jpg](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.55.57.jpg)
    
    parent task에 결과값을 그대로 return 합니다.
    
    ![스크린샷 2022-06-21 오후 8.56.40.jpg](%5B2022%5D%20Explore%20structured%20concurrency%20in%20Swift%20d9276acf678c44b9847b55a98cb9b2d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.56.40.jpg)
    
    parent task에서는 `for await` 반복문을 이용하여 결과값을 가져옵니다.