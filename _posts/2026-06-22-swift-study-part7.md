---
layout: post
title: "[Swift 스터디 7편] Swift 고급 2: 에러 핸들링과 타입 시스템의 진화"
date: 2026-06-22 00:30:00 +0900
categories: [Swift]
tags: [Swift, iOS, Study, ErrorHandling, OpaqueType, ResultBuilder]
---

안녕하세요! 현생이 바빠 오랜만에 포스팅을 하게 되었습니다. 
이번에는 Swift 스터디 7번째 시간입니다. 

오늘은 Swift 고급 파트의 두 번째 시간으로, 예외 상황에 대처하는 **오류 처리(Error Handling)** 방식과 최근 Swift 버전을 거듭하며 더욱 강력해진 타입 시스템인 **불명확 타입(some)과 상자형 프로토콜 타입(any)**, 그리고 SwiftUI와 같은 선언형 프레임워크의 근간이 되는 **결과 구축자(Result Builders)**에 대해 알아보겠습니다. 🚀

---

## 1. 오류 처리 (Error Handling)

프로그램 실행 중 예기치 못한 에러가 발생했을 때 프로그램이 죽지 않고 유연하게 대처할 수 있도록 Swift는 에러 처리 기능을 제공합니다.

### 1) 에러 정의 및 던지기 (`throws` & `throw`)
에러를 정의하려면 `Error` 프로토콜을 채택하는 열거형을 만듭니다. 그리고 에러가 발생할 수 있는 함수는 화살표(`->`) 앞에 `throws` 키워드를 명시해야 합니다.

```swift
enum VendingMachineError: Error {
    case invalidSelection
    case outOfStock
    case insufficientFunds(coinsNeeded: Int)
}

func vend(itemNamed name: String) throws {
    let itemPrice = 100
    let deposited = 50
    
    // 조건이 맞지 않으면 에러를 '던짐'
    if deposited < itemPrice {
        throw VendingMachineError.insufficientFunds(coinsNeeded: itemPrice - deposited)
    }
    print("\(name) 제공 완료!")
}
```

### 2) 에러 처리하기 (`do-catch` & `try`)
`throws`가 붙은 함수를 호출할 때는 반드시 앞에 `try` 키워드를 붙여야 하며, 에러를 잡아내기 위해 `do-catch` 블록으로 감싸줍니다.

```swift
do {
    try vend(itemNamed: "콜라")
} catch VendingMachineError.insufficientFunds(let coins) {
    print("돈이 부족합니다. \(coins)원이 더 필요합니다.")
} catch {
    print("알 수 없는 에러 발생: \(error)")
}
```

- **`try?`**: 에러가 발생하면 무시하고 `nil`을 반환합니다. (에러의 구체적 원인이 중요하지 않을 때 사용)
```swift
let x : Optional = try? someThrowingFunction(shouldThrowError: true)
print(x)    // nil
```
- **`try!`**: 에러가 절대 발생하지 않을 것이라 확신할 때 사용합니다. (에러 발생 시 크래시)
```swift
let x : Optional = try! someThrowingFunction(shouldThrowError: true) // 런타임 오류 발생!!
```

### 3) 다시 던지기 (`rethrows`)
함수나 메서드는 `rethrows` 키워드를 사용하여 자신의 매개변수로 전달받은 함수가 오류를 던진다는 것을 나타낼 수 있습니다.
단, 다시 던지기 내부의 catch 절에서는 다시 던지기 함수의 매개변수로 전달받은 오류던지기 함수만 호출하고 결과로 던져진 오류만 제어할 수 있습니다.
```swift
func someThrowingFunction() throws {
    enum SomeError: Error {
        case justSomeError
    }
    throw SomeError.justSomeError
}

func someFunction(callback: () throws -> Void) rethrows {
    enum AnotherError: Error {
        case justAnotherError
    }

    do {
        // 매개변수로 전달한 오류 던지기 함수이므로 catch 절에서 제어 가능
        try callback()
    } catch {
        throw AnotherError.justAnotherError
    }

    do {
        // 매개변수로 전달한 오류 던지기 함수가 아니므로 catch 절에서 제어 불가능!
        try someThrowingFunction()
    } catch {
        // 오류 발생!
        throw AnotherError.justAnotherError
    }
    // catch 절 외부에서 단독으로 오류 던지기도 불가능! 오류 발생!
    throw AnotherError.justAnotherError
}
```

### 4) 후처리 (`defer`)
반드시 에러와 관련된 처리는 아니지만, `defer` 구문을 사용하여 코드 블록을 나가기 전에 꼭 실행해야 하는 코드를 작성해줄 수 있습니다.
여러 개의 `defer`구문이 하나의 범위 내부에 속해 있다면 맨 마지막에 작성된 구문부터 역순으로 실행됩니다.
```swift
func someFunction() {
    print("1")

    defer {
        print("2")
    }

    do {
        defer {
            print("3")
        }
        print("4")
    }

    defer {
        print("5")
    }

    print("6")
}

someFunction()
// 1 4 3 6 5 2 순으로 출력됨
```

---

## 2. 불명확 타입과 상자형 프로토콜 타입 (some vs any)

Swift 5.1과 5.6을 거치며 프로토콜을 다루는 두 가지 중요한 키워드인 `some`과 `any`가 도입되었습니다. 두 가지는 다형성을 제공하지만 그 방식과 성능이 다릅니다.

### 불명확 타입 (Opaque Types: `some`)
`some` 키워드는 컴파일러는 정확한 타입을 알고 있지만, 외부(호출하는 쪽)에는 그 타입이 무엇인지 숨기고 **"특정 프로토콜을 준수하는 무언가"**라고만 알려주는 방식입니다.

```swift
// 'Shape' 프로토콜을 준수하는 "어떤(some) 단일 타입"을 반환
func makeSquare() -> some Shape {
    return Square() 
}
```
`some`은 항상 '동일한 하나의 구체적인 타입'만 반환해야 합니다. 컴파일 타임에 타입이 확정되므로 **성능 최적화(Static Dispatch)**에 매우 유리합니다. SwiftUI의 `var body: some View`가 바로 이 개념입니다.

### 상자형 프로토콜 타입 (Boxed Protocol Types: `any`)
반면 `any` 키워드는 **"이 프로토콜을 준수하는 아무 타입이나 박스(Box)에 담아서 허용하겠다"**는 의미입니다. 이를 실존 타입(Existential Type)이라고 합니다.

```swift
// Shape를 준수하는 다양한 타입이 배열에 담길 수 있습니다.
var shapes: [any Shape] = [Square(), Circle(), Triangle()]

// 반환값도 여러 타입이 될 수 있습니다.
func makeRandomShape() -> any Shape {
    if Bool.random() { return Square() } 
    else { return Circle() }
}
```

```swift
// 만약 다음과 같이 some Shape를 사용한다면, 
// Shape를 준수하는 타입 중 단 하나의 타입만 들어갈 수 있음. (타입이 섞이지 않음)
var shapes: [some Shape]
```

`any`는 런타임에 박스를 열어서 실제 타입을 확인해야 하므로(Dynamic Dispatch) 메모리 및 성능 오버헤드가 발생합니다. 따라서 가능하면 성능이 좋은 `some`을 사용하고, 여러 타입이 섞여야만 하는 경우에만 `any`를 사용하는 것이 좋습니다.

---

## 3. 결과 구축자 (Result Builders)

**결과 구축자(Result Builders)**는 일련의 선언적 코드 블록을 조합하여 최종적으로 하나의 결과를 만들어내는 문법적 마법입니다. `@resultBuilder` 속성을 사용해 정의할 수 있으며, 복잡한 데이터 계층 구조를 직관적으로 작성하게 해줍니다.

SwiftUI에서 `VStack`이나 `HStack` 안에 여러 개의 뷰를 나열하기만 해도 화면이 그려지는 이유가 바로 이 결과 구축자 덕분입니다.

```swift
@resultBuilder
struct HTMLBuilder {
    // 여러 개의 문자열을 받아 하나의 문자열로 결합하는 빌더
    static func buildBlock(_ components: String...) -> String {
        return components.joined(separator: "\n")
    }
}

// 빌더를 파라미터로 받는 함수
func makeHTML(@HTMLBuilder content: () -> String) -> String {
    return "<html>\n\(content())\n</html>"
}

// 쉼표나 return 없이 선언적으로 작성 가능!
let webPage = makeHTML {
    "<body>"
    "    <h1>Hello, Swift!</h1>"
    "    <p>This is built with Result Builders.</p>"
    "</body>"
}

print(webPage)
```
위처럼 배열의 요소들을 쉼표(`,`) 없이 줄바꿈만으로 나열할 수 있게 해주어, DSL(Domain Specific Language, 도메인 특화 언어)을 설계하는 데 핵심적인 역할을 합니다.

---

### 마무리

오늘은 예외 상황을 처리하는 **Error Handling**과, 모던 Swift 타입 시스템의 양대 산맥인 **`some`과 `any`의 차이**, 그리고 선언적 UI의 기반이 되는 **Result Builders**에 대해 알아보았습니다.

특히 `some`과 `any`는 최신 Swift 코드(SwiftUI, 최신 아키텍처 등)를 읽기 위해 반드시 이해해야 하는 중요한 개념이니 잘 정리해 두시길 바랍니다!

마지막 8편에서는 모던 Swift의 정수이자 가장 강력한 기능들인 **동시성(Concurrency)**과 **매크로(Macros)**에 대해 다루도록 하겠습니다. 감사합니다! 🚀
