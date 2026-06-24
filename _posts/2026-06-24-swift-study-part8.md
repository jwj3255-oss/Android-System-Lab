---
layout: post
title: "[Swift 스터디 8편] Swift 고급 3: 모던 Swift의 정수, 동시성과 매크로"
date: 2026-06-24 23:00:00 +0900
categories: [Swift]
tags: [Swift, iOS, Study, Concurrency, AsyncAwait, Macro]
---

안녕하세요! Swift 스터디 대망의 마지막 8번째 시간입니다. 

오늘은 Swift의 최신 기술 트렌드를 이끄는 가장 중요하고 파워풀한 두 가지 주제, 바로 비동기 프로그래밍을 획기적으로 개선한 **동시성(Concurrency)** 모델과 보일러플레이트 코드를 마법처럼 줄여주는 **매크로(Macros)**에 대해 알아보겠습니다. 🚀

---

## 1. 동시성 (Concurrency)

과거 iOS 개발자들은 백그라운드 작업을 처리하기 위해 GCD(Grand Central Dispatch, `DispatchQueue`)와 탈출 클로저(`@escaping`)를 주로 사용했습니다. 하지만 이 방식은 콜백 지옥(Callback Hell)을 유발하고, 코드의 가독성을 떨어뜨리며, 스레드 간 데이터 동기화 문제(Data Race)를 잡기 힘들었습니다.

이를 해결하기 위해 Swift 5.5부터 강력한 네이티브 동시성 모델인 **`async/await`**가 도입되었습니다.

### 1) async / await
`async` 키워드는 이 함수가 비동기로 동작함을 나타내며, `await` 키워드는 비동기 함수의 결과가 도착할 때까지 현재 스레드를 양보(Yield)하고 기다리겠다는 의미입니다.
비동기 코드를 마치 동기 코드처럼 한 줄 한 줄 직관적으로 읽을 수 있게 해줍니다.

```swift
// 네트워크에서 이미지를 다운로드하는 비동기 함수
func fetchImage(id: String) async throws -> UIImage {
    let url = URL(string: "https://example.com/image/\(id)")!
    // await를 만나면 일시 정지(Suspend)되고, 결과를 받으면 재개(Resume)됨
    let (data, _) = try await URLSession.shared.data(from: url)
    return UIImage(data: data)!
}

// 사용 시
func updateUI() async {
    do {
        // 콜백 없이 직관적으로 값을 받아옴!
        let image = try await fetchImage(id: "123")
        print("이미지 다운로드 성공")
    } catch {
        print("에러 발생: \(error)")
    }
}
```

### 2) Task
동기적인 맥락(일반 함수 등)에서 비동기(`async`) 함수를 호출하려면, 새로운 비동기 컨텍스트를 만들어주어야 합니다. 이때 사용하는 것이 `Task`입니다.

```swift
func buttonTapped() {
    // 동기 함수 내부에서 비동기 작업 시작
    Task {
        await updateUI()
        await Task.yield()
    }
}
```
Task.yield() 메서드는 다른 작업에게 작업을 명시적으로 양보할 때 사용합니다. 
지금 수행하는 작업이 무겁고 오래 걸리는 작업이라면 다른 작업에게 중간중간 양보해서 다른 작업이 너무 늦게 처리되지 않도록 명시적으로 기회를 제공합니다.

### 3) Actor (액터)
여러 스레드에서 동시에 하나의 데이터에 접근하고 수정하면 값이 꼬이는 **데이터 레이스(Data Race)**가 발생합니다. Swift는 이 문제를 근본적으로 차단하기 위해 **`actor`**라는 새로운 참조 타입을 도입했습니다.

`actor`는 내부적으로 자신의 상태(프로퍼티)에 한 번에 하나의 작업만 접근할 수 있도록 격리(Isolation)합니다. 따라서 외부에서 액터의 상태에 접근할 때는 반드시 `await`를 붙여 순서를 기다려야 합니다.

```swift
actor BankAccount {
    var balance: Int = 0
    
    // 동시에 여러 곳에서 접근해도 안전하게 처리됨!
    func deposit(amount: Int) {
        balance += amount
    }
}

let account = BankAccount()
Task {
    // 액터의 상태 변경은 비동기적으로(순서를 기다려) 이루어져야 함
    await account.deposit(amount: 1000)
    print(await account.balance)
}
```

### 4) Sendable 프로토콜

비동기 및 동시성 환경에서 태스크(Task)나 액터(Actor) 간에 데이터를 안전하게 전달할 수 있는 타입을 나타내기 위해 **`Sendable`** 프로토콜을 사용합니다. 이는 컴파일러가 동시성 안전성을 체크하기 위한 **마커 프로토콜(Marker Protocol)**입니다.

- **값 타입**: 구조체(Struct)나 열거형(Enum)은 내부 멤버들이 모두 `Sendable`을 준수하면 자동으로 `Sendable`이 됩니다.
- **참조 타입**: 클래스(Class)는 기본적으로 `Sendable`이 아닙니다. 클래스를 `Sendable`로 만드려면 `final`이어야 하고 내부 프로퍼티가 모두 불변(Immutable)이어야 하며, 그렇지 않은 경우 `@unchecked Sendable`을 선언하여 개발자가 수동으로 동기화를 보장해야 합니다.

```swift
// 모든 프로퍼티가 값 타입이므로 자동으로 Sendable 준수
struct UserInfo: Sendable {
    let name: String
    let age: Int
}

// 클래스는 기본적으로 Sendable이 아님 (컴파일러 경고 발생)
class MutableUser {
    var name: String = ""
}
```

### 5) Copyable과 Noncopyable (`~Copyable`)

Swift의 모든 타입은 기본적으로 값을 복사할 수 있는 **`Copyable`** 프로토콜을 암시적으로 준수합니다. 하지만 Swift 5.9부터는 복사를 금지하고 고유한 소유권을 유지할 수 있도록 하는 **Noncopyable** 타입을 지원합니다.

`~Copyable` 문법을 사용하여 "복사 기능이 없다"고 선언할 수 있습니다. 

- **메모리 및 자원 관리**: 파일 핸들이나 시스템 자원처럼 복사되면 안 되고 유일한 소유권이 유지되어야 하는 객체를 정의할 때 매우 유용합니다.
- **동시성 환경에서의 이점**: 값이 임의로 복사되어 여러 스레드로 흩어지는 것을 방지하므로, 데이터 흐름과 소유권을 컴파일 타임에 완벽히 추적하여 더 안전한 코드를 작성할 수 있습니다.

```swift
// 복사가 불가능한 구조체 정의
struct FileDescriptor: ~Copyable {
    let fd: Int
    
    // 소유권이 사라질 때 호출될 소멸자 정의 가능 (구조체임에도 deinit 지원!)
    deinit {
        // 파일 닫기 등의 정리 작업
    }
}

var file = FileDescriptor(fd: 1)
// let copiedFile = file // 🚨 에러! 복사할 수 없음
```

---

## 2. 매크로 (Macros)

Swift 5.9에서 새롭게 도입된 **매크로(Macros)**는 컴파일 타임에 코드를 분석하고, 반복적이고 지루한 코드(Boilerplate)를 자동으로 생성해 주는 마법 같은 기능입니다. 개발자는 코드를 짧게 작성하고, 나머지는 매크로가 컴파일 전에 펼쳐서 채워 넣습니다.

매크로는 적용 방식에 따라 크게 **프리스탠딩 매크로**와 **연결된 매크로**로 나뉘며, 세부적인 역할(Role)에 따라 더 세분화됩니다.

### 1) 프리스탠딩 매크로 (Freestanding Macros)
문맥에 얽매이지 않고 독립적으로 새로운 코드를 만들어내는 매크로입니다. `#` 기호를 붙여 사용합니다.
- **`@freestanding(expression)`**: 값을 반환하는 식을 생성합니다. (예: `#URL("https://apple.com")` - 잘못된 주소 입력 시 컴파일 타임에 에러를 던지도록 안전하게 구현할 수 있음)
- **`@freestanding(declaration)`**: 하나 이상의 선언(함수, 변수, 타입 등)을 추가로 생성합니다. (예: `#warning("경고 메시지")`)

```swift
// 코드를 빌드할 때 매크로가 이를 실제 URL 객체 생성 코드로 안전하게 변환해 줌
let url = #URL("https://apple.com")
```

### 2) 연결된 매크로 (Attached Macros)
기존에 존재하는 타입, 변수, 함수 등의 선언부 바로 위에 부착되어 동작하는 매크로입니다. `@` 기호를 붙여 사용합니다.
- **`@attached(member)`**: 해당 타입 내부에 새로운 멤버(프로퍼티, 메서드, 이니셜라이저 등)를 주입합니다.
- **`@attached(accessor)`**: 프로퍼티에 `get`, `set`, `willSet`, `didSet` 같은 접근자를 자동으로 생성해 줍니다. (예: `@Observable`)
- **`@attached(peer)`**: 선언된 요소와 동일한 스코프 내의 바로 옆(동일 레벨)에 새로운 선언을 생성합니다. (예: 비동기 메서드를 기반으로 한 완료 콜백 형태의 메서드를 자동 생성)
- **`@attached(memberAttribute)`**: 해당 타입의 개별 멤버들에 일괄적으로 특정 속성(예: `@objc`)을 덧붙입니다.
- **`@attached(extension)`**: 해당 타입의 `extension`을 새로 만들어내어 특정 프로토콜 채택 및 구현을 자동으로 추가해 줍니다.

```swift
@Observable // Attached Macro 적용 (내부적으론 member, accessor 등의 역할 수행)
class User {
    var name: String = "원준"
    var age: Int = 25
}
```
위의 단 4줄짜리 코드는 컴파일 시간에 매크로에 의해 수십 줄의 옵저버 패턴 코드로 자동 확장(Expand)됩니다. 이를 통해 개발자는 비즈니스 로직에만 집중할 수 있게 되었습니다!

### 3) 사용자 정의 매크로 만들기 (Custom Macros)

Swift 매크로는 단순한 문자열 텍스트 치환 방식이 아닙니다. 컴파일러가 코드를 읽어 구문 분석을 마친 **추상 구문 트리(AST, Abstract Syntax Tree)** 상태의 코드를 제공하면, 우리가 만든 매크로 로직이 이를 분석하여 안전한 코드를 새로 덧붙이는 구조입니다.

이를 직접 개발하려면 Apple이 제공하는 **`swift-syntax`** 라이브러리를 활용해야 합니다. 매크로 개발 프로세스는 크게 두 가지 컴포넌트로 분리됩니다.

#### 1. 매크로 선언 (Declaration) - 클라이언트가 사용하는 인터페이스 정의
클라이언트 앱이나 프레임워크 타겟 내에서 매크로의 형식을 지정하고, 실제 구현 모듈과 타입을 매핑합니다.
```swift
// 인자로 넘어온 표현식의 실제 값과 해당 식의 코드 텍스트를 튜플로 반환하는 매크로
@freestanding(expression)
public macro stringify<T>(_ value: T) -> (T, String) = #externalMacro(module: "MyMacroPlugins", type: "StringifyMacro")
```

#### 2. 매크로 구현 (Implementation) - AST를 파싱하여 코드 생성
`swift-syntax` 패키지를 이용해 외부 컴파일러 플러그인 모듈에서 AST 노드를 조작하는 실제 구현체를 작성합니다.
```swift
import SwiftCompilerPlugin
import SwiftSyntax
import SwiftSyntaxBuilder
import SwiftSyntaxMacros

public struct StringifyMacro: ExpressionMacro {
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) -> ExprSyntax {
        // 호출 시 매크로 인자로 들어온 첫 번째 표현식 추출
        guard let argument = node.argumentList.first?.expression else {
            fatalError("인자가 부족합니다.")
        }
        
        // "(값, "코드텍스트")" 형태의 새로운 AST 코드로 변환하여 반환
        return "(\(argument), \(literal: argument.description))"
    }
}
```
개발자는 이 사용자 정의 매크로를 활용해 프로젝트에 종속된 불필요한 보일러플레이트를 완전히 제거하고, 코드 품질을 높이는 유용한 사내 라이브러리 도구를 만들 수 있습니다.

---

### 대단원의 마무리

지금까지 총 8편에 걸쳐 Swift의 기초 문법부터 객체지향, 함수형, 프로토콜 지향 패러다임, 그리고 동시성과 매크로 같은 최신 고급 기술까지 숨 가쁘게 달려왔습니다.

Swift는 지금도 계속해서 진화하고 있는 강력하고 매력적인 언어입니다. 이 스터디 시리즈가 여러분이 Swift 코드를 더 쉽게 읽고, 더 우아하게 작성하는 데 탄탄한 밑거름이 되었기를 바랍니다. 

이제 직접 Xcode를 켜고 여러분만의 아이디어를 앱으로 구현해 보세요! 수고하셨습니다! 🎉
