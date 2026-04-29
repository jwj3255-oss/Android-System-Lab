---
layout: post
title: "[Swift 스터디 6편] Swift 고급 1: 문법 심화와 메모리 관리(ARC)"
date: 2026-04-29 17:51:00 +0900
categories: [Swift]
tags: [Swift, iOS, Study, ARC, Memory, Pattern]
---

안녕하세요! Swift 스터디 6번째 시간입니다. 지난 포스팅들에서는 프로토콜 지향 프로그래밍(POP) 등 Swift의 코어 패러다임을 살펴보았습니다.

오늘부터는 Swift를 더욱 깊이 이해하고 전문가 수준으로 다루기 위한 **고급 파트**를 3개의 포스트로 나누어 다루어보려 합니다. 첫 번째 시간에는 유용한 문법적 특징인 **타입 중첩, 패턴과 where 절**, 그리고 iOS 개발자라면 반드시 이해해야 할 **ARC와 메모리 안전성(Memory Safety)**에 대해 알아보겠습니다. 🚀

---

## 1. 타입 중첩 (Nested Types)

특정 클래스나 구조체, 열거형 내부에서만 사용할 목적으로 새로운 타입을 정의하는 것을 타입 중첩이라고 합니다. 타입 안에 다른 타입을 선언함으로써, 타입 간의 연관성을 명확히 하고 전역 스코프를 깔끔하게 유지할 수 있습니다.

```swift
struct BlackjackCard {
    // 중첩 열거형
    enum Suit: Character {
        case spades = "♠", hearts = "♡", diamonds = "♢", clubs = "♣"
    }
    
    // 중첩 구조체
    struct Rank {
        enum RankType { case ace, face, numeric } // 중첩 안에 또 중첩 가능!
        var type: RankType
        var value: Int
    }
    
    // BlackjackCard의 프로퍼티로 중첩 타입 사용
    let suit: Suit
    let rank: Rank
}

// 외부에서 중첩 타입에 접근할 때는 점(.) 구문을 사용합니다.
let theSpadesSymbol = BlackjackCard.Suit.spades.rawValue
```

---

## 2. 패턴 (Patterns)과 where 절

Swift에서 **패턴**은 단일 값 또는 복합적인 값을 나타내는 구조입니다. `switch`, `if`, `guard`, `for-in` 구문 등에서 데이터를 추출하거나 형태를 확인할 때 유용합니다.
여기에 **`where` 절**을 결합하면 패턴에 부가적인 조건을 걸어 훨씬 강력하게 필터링을 할 수 있습니다.

```swift
let point = (1, -1)

// 튜플 패턴 매칭과 where 절을 이용한 조건 필터링
switch point {
case let (x, y) where x == y:
    print("x와 y가 같은 선상에 있습니다.")
case let (x, y) where x == -y:
    print("x와 y의 부호가 반대인 선상에 있습니다.")
case (let x, let y):
    print("아무 곳에나 있습니다. 좌표: (\(x), \(y))")
}

// for-in 루프에서의 where 절
let numbers = [1, 2, 3, 4, 5, 6, 7]
for number in numbers where number % 2 == 0 {
    print("\(number)는 짝수입니다.")
}
```

---

## 3. ARC (Automatic Reference Counting)

Swift는 메모리 사용량을 관리하기 위해 **ARC(자동 참조 카운트)**를 사용합니다. ARC는 인스턴스가 더 이상 필요하지 않을 때(참조 횟수가 0이 될 때) 자동으로 메모리에서 해제해 주는 방식입니다.
ARC와 다른 프로그래밍 언어에서 사용되는 가비지 컬렉션의 가장 큰 차이점은 참조를 계산하는 시점입니다. ARC는 인스턴스가 언제 메모리에서 해제되어야 할지를 컴파일 타임에 결정합니다. 가비지 컬렉션은 그렇지 않죠.

하지만 클래스(참조 타입) 간에 서로를 참조하여 참조 카운트가 0이 되지 않는 **강한 순환 참조(Strong Reference Cycle)**가 발생하면 메모리 누수(Memory Leak)가 생깁니다. 이를 해결하기 위해 `weak`와 `unowned` 키워드를 사용합니다.

- **강한 참조 (`strong`)**: 기본값. 참조할 때 카운트를 +1 증가시킵니다.
- **약한 참조 (`weak`)**: 참조할 때 카운트를 증가시키지 않습니다. 참조하던 인스턴스가 메모리에서 해제되면 자동으로 `nil`이 되므로 반드시 **옵셔널 변수(`var`)**로 선언해야 합니다.
- **미소유 참조 (`unowned`)**: `weak`와 같이 카운트를 증가시키지 않지만, 인스턴스가 항상 존재할 것이라 확신할 때 사용합니다. 옵셔널이 아니며, 인스턴스가 해제된 후 접근하려 하면 앱이 크래시납니다.

```swift
class Person {
    let name: String
    // 강한 순환 참조를 피하기 위해 weak 사용
    weak var apartment: Apartment? 
    
    init(name: String) { self.name = name }
    deinit { print("\(name) 메모리 해제") }
}

class Apartment {
    let unit: String
    // 서로 강하게 참조하면 둘 다 메모리에서 영원히 해제되지 않음!
    var tenant: Person?
    
    init(unit: String) { self.unit = unit }
    deinit { print("아파트 \(unit) 메모리 해제") }
}

var wonjun: Person? = Person(name: "원준") // Person 인스턴스의 참조 횟수 : 1
var unit4A: Apartment? = Apartment(unit: "4A") // Apartment 인스턴스의 참조 횟수 : 1

wonjun?.apartment = unit4A // Person -> Apartment weak 참조. Apartment 참조 횟수는 여전히 1
unit4A?.tenant = wonjun // Apartment -> Person strong 참조. Person 참조 횟수: 2

unit4A = nil // Apartment 참조 횟수 : 0, Person의 참조 횟수 : 1. Apartment 메모리 해제
wonjun = nil // Person 참조 해제. 참조 횟수 : 0, Person 메모리 해제
```
만약 apartment가 강한 참조였다면 wonjun에 nil이 할당되어도 unit4A.tenant가 wonjun을 바라보고 있기에, wonjun은 여전히 참조 횟수 1이므로 메모리에서 해제되지 않습니다. 
또한 unit4A에 nil이 할당되어도 wonjun.apartment가 unit4A를 바라보고 있기에 unit4A는 여전히 참조 횟수 1이므로 메모리에서 해제되지 않는 상황이 발생합니다. 이것이 바로 강한 순환 참조입니다.

### 미소유 참조
```swift
class Person {
    let name: String

    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
    deinit { print("\(name) 메모리 해제") }
}

class CreditCard {
    let number: String
    unowned let owner: Person
    init(number: String, owner: Person) {
        self.number = number
        self.owner = owner
    }
    deinit { print("카드 \(number) 메모리 해제") }
}

var wj: Person? = Person(name: "wj") // Person 참조 횟수: 1
wj?.card = CreditCard(number: "1234-5678-9012-3456", owner: wj!) // CreditCard 참조 횟수: 1, Person 참조 횟수: 1

wj = nil // wj 메모리 해제. 카드 1234-5678-9012-3456 메모리 해제.
// Person 참조 횟수: 0, CreditCard 참조 횟수: 0
```
### 클로저의 강한참조 순환 및 획득목록
인스턴스끼리만이 아니라, 클로저 내부에서도 강한 참조 순환이 발생할 수 있습니다. 예를 들어 클로저 내부에서 인스턴스의 프로퍼티에 접근하거나 메서드를 호출하면 클로저가 해당 인스턴스를 강하게 참조하게 됩니다. 이 때 클로저가 해당 인스턴스의 소유권을 넘겨받는 것이기 때문에 메모리 누수가 발생합니다. 이를 해결하기 위해 **획득 목록(Capture List)**을 사용하여 클로저가 인스턴스를 약하게 참조하도록 할 수 있습니다.

```swift
class Event {
    let name: String
    var ticketPrice: Int
    
    var cancellable:(() -> Void)?
    init(name: String, ticketPrice: Int) {
        self.name = name
        self.ticketPrice = ticketPrice
    }
    deinit { print("이벤트 \(name) 메모리 해제") }
}

var movie: Event? = Event(name: "Movie", ticketPrice: 10000)

// 클로저가 movie를 강하게 참조하여 메모리 누수 발생
movie?.cancellable = {
    print("\(movie!.name) 티켓 취소 완료")
    movie = nil
}

// 강한 참조 해결
movie?.cancellable = {
    [weak movie] in // 획득목록은 클로저를 감싸는 중괄호 {} 바로 앞에 [] 안에 작성하고, in과 함께 씁니다.
    guard let event = movie else { return }
    print("\(event.name) 티켓 취소 완료")
    movie = nil
}
```

---

## 4. 메모리 안전성 (Memory Safety)

Swift는 기본적으로 메모리에 안전하게 접근하도록 설계되어 있습니다. 코드가 초기화되지 않은 메모리에 접근하거나 메모리가 해제된 후 접근하는 것을 방지합니다.
특히 Swift 4부터 **메모리에 대한 배타적 접근(Exclusive Access to Memory)**이라는 규칙이 강화되었습니다.

만약 두 개의 변수가 같은 메모리 위치를 가리키고 있는데, 한쪽에서 수정(write) 작업이 진행 중일 때 다른 쪽에서 읽기(read)나 수정(write)을 시도하면 컴파일 타임 혹은 런타임에 **충돌(Conflict)** 에러를 발생시킵니다. 

주로 함수에서 `inout` 파라미터로 자신의 프로퍼티를 넘길 때 이러한 충돌이 발생하기 쉽습니다.

```swift
var stepSize = 1

func increment(_ number: inout Int) {
    number += stepSize // 에러! number(수정 중)와 stepSize(읽기)가 같은 메모리를 참조할 수 있음
}

// 이를 해결하려면 복사본을 만들어 전달해야 합니다.
var copyOfStepSize = stepSize
increment(&copyOfStepSize)
stepSize = copyOfStepSize
```

---

### 마무리

오늘은 Swift 고급 과정의 첫 번째로 코드를 더 깔끔하게 만들어주는 문법(타입 중첩, 패턴, where)과 앱의 성능 및 안정성을 결정짓는 메모리 관리(ARC, 메모리 안전성)에 대해 알아보았습니다. 

특히 ARC의 순환 참조 문제와 `weak`, `unowned`의 적절한 사용은 iOS 면접에서도 단골로 등장하는 핵심 개념이니 꼭 숙지해 두시길 바랍니다!

다음 시간(7편)에는 에러를 우아하게 처리하는 방법과 최신 Swift의 강력한 타입 시스템인 **불명확 타입과 상자형 프로토콜**, 그리고 SwiftUI의 기반이 되는 **결과 구축자**에 대해 다루겠습니다. 기대해 주세요! 🚀
