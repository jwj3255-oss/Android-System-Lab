---
layout: post
title: "[Swift 스터디 5편] Swift의 정수: 프로토콜, 제네릭, 프로토콜 지향 프로그래밍(POP)"
date: 2026-04-29 13:27:00 +0900
categories: [Swift]
tags: [Swift, iOS, Study, Protocol, Generics, POP]
---

안녕하세요! Swift 스터디 다섯 번째 시간입니다. 저번 4편 포스팅에서는 상속, 타입캐스팅, 그리고 타입에 새로운 기능을 부여하는 익스텐션에 대해 알아보았습니다.

오늘은 Swift 언어를 가장 잘 설명할 수 있는 핵심적인 패러다임이자, 이 시리즈의 하이라이트인 **프로토콜 지향 프로그래밍(Protocol-Oriented Programming, POP)**에 대해 다루어 보려고 합니다. 이를 이해하기 위한 필수 개념인 **프로토콜(Protocol)**과 **제네릭(Generics)**부터 차근차근 살펴보겠습니다. 🚀

---

## 1. 프로토콜 (Protocols)

프로토콜은 특정 역할이나 기능을 하기 위해 필요한 **메서드, 프로퍼티 등의 요구사항을 정의하는 '청사진(Blueprint)'**입니다. 프로토콜은 직접 기능을 구현하지는 않으며, 클래스, 구조체, 열거형이 이 프로토콜을 **채택(Adopt)**하여 그 요구사항을 실제 코드로 구현하게 됩니다.

다른 객체지향 언어의 인터페이스(Interface)와 유사한 개념입니다.

```swift
// 무언가 소리를 낼 수 있다는 프로토콜 정의
protocol SoundMaking {
    // 읽기 전용 프로퍼티 요구
    var soundLevel: Int { get }
    
    // 메서드 요구 (구현부 없음)
    static func makeSound() // 프로토콜의 메서드는 관례상 static 키워드를 사용해서 정의해 타입 메서드임을 나타냅니다.
}

// 구조체가 프로토콜 채택 및 구현
struct Dog: SoundMaking {
    var soundLevel: Int = 10
    
    func makeSound() {
        print("멍멍!")
    }
}

// 클래스가 프로토콜 채택 및 구현
class Car: SoundMaking {
    var soundLevel: Int = 80
    
    class func makeSound() { // class 키워드를 사용하면 Car를 상속하는 자식 클래스에서 이 메서드를 재정의할 수 있습니다. static 키워드는 재정의할 수 없습니다.
        print("부릉부릉!")
    }
}
```

프로토콜을 사용하면 전혀 연관이 없는 타입들(위 예시의 `Dog`와 `Car`)이라도 동일한 `SoundMaking` 타입으로 묶어서 다룰 수 있습니다. 배열을 `[SoundMaking]` 타입으로 만들어서 반복문을 돌리는 것도 가능하죠!

가끔은 메서드가 인스턴스 내부의 값을 변경할 필요가 있습니다. 그럴 때는 `mutating` 키워드를 사용합니다.
- 프로토콜에서 메서드 앞에 `mutating` 키워드를 붙이면, 프로토콜을 채택한 **구조체**와 **열거형**은 이 메서드 내부에서 프로퍼티의 값을 변경할 수 있습니다.
- **클래스**는 `mutating` 키워드가 없어도 기본적으로 프로퍼티를 변경할 수 있습니다.

```swift
protocol Counter {
    mutating func increment() // 메서드가 내부 값을 변경할 것임을 나타냄
}

struct IntCounter: Counter {
    var count = 0
    
    mutating func increment() {
        count += 1
    }
}

class IntCounter: Counter {
    var count = 0
    
    func increment() {
        count += 1
    }
}
```

이번에는 프로토콜의 이니셜라이저에 대해 알아봅시다. 프로토콜에서 이니셜라이저를 정의할 수 있습니다. 
클래스 타입에서 프로토콜의 이니셜라이저 요구에 부합하는 이니셜라이저를 구현할 때는 `required` 키워드를 사용해야 합니다.

```swift
protocol Initializable {
    init(value: Int)
}

struct MyStruct: Initializable {
    var value: Int
    
    init(value: Int) {
        self.value = value
    }
}

class MyClass: Initializable {
    required init(value: Int) {
        self.value = value
    }
}
```
---

## 2. 제네릭 (Generics)

제네릭은 **타입(Type)에 의존하지 않고 범용적이고 재사용 가능한 함수나 타입을 작성**할 수 있게 해주는 기능입니다. Swift 표준 라이브러리의 대다수(Array, Dictionary 등)가 제네릭으로 구현되어 있습니다.

예를 들어 두 변수의 값을 바꾸는 함수를 만들 때, `Int`용, `String`용, `Double`용 함수를 각각 만들면 매우 비효율적입니다. 제네릭을 사용하면 이를 하나로 통합할 수 있습니다.

### 제네릭 함수
꺾쇠 기호(`<T>`)를 사용하여 임의의 타입 `T`를 정의합니다. (꼭 T가 아니어도 되지만 관용적으로 대문자 하나 또는 의미 있는 CamelCase 이름을 사용합니다.)

```swift
// 어떤 타입이든 두 변수의 값을 바꿔주는 제네릭 함수
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}

var someInt = 3
var anotherInt = 107
swapTwoValues(&someInt, &anotherInt) // 정수형 스왑 가능

var someString = "hello"
var anotherString = "world"
swapTwoValues(&someString, &anotherString) // 문자열 스왑 가능!
```

### 제네릭 타입
클래스, 구조체, 열거형 자체도 제네릭으로 만들 수 있습니다. 스택(Stack) 자료구조를 제네릭으로 만들어 보겠습니다. 함수의 매개변수는 `T`였지만, 클래스, 구조체, 열거형 등 타입에 대해선 `Element`를 사용합니다.

```swift
struct Stack<Element> {
    var items: [Element] = []
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    mutating func pop() -> Element {
        return items.removeLast()
    }
}

var stackOfStrings = Stack<String>()
stackOfStrings.push("uno")
stackOfStrings.push("dos")

var stackOfAny = Stack<Any>()
stackOfAny.push(1)
stackOfAny.push("hello")
stackOfAny.push(3.14) // [1, "hello", 3.14]
```

제네릭에 특정 프로토콜을 준수하는 타입만 들어올 수 있도록 **타입 제약(Type Constraints)**을 걸 수도 있습니다. (예: `func someFunction<T: Equatable>(...)`)

---

## 3. 프로토콜 지향 프로그래밍 (POP)

객체지향 프로그래밍(OOP)에서는 '상속'을 통해 공통 기능을 재사용했습니다. 하지만 상속은 클래스에서만 가능하고, 단일 상속만 허용되며, 부모 클래스와 강하게 결합되는 단점이 있습니다.

Swift는 **프로토콜 초기 구현(Protocol Default Implementation)**을 통해 이를 완벽하게 극복하고, 구조체(값 타입)에서도 상속과 같은 재사용성을 누릴 수 있게 만들었습니다. 이것이 바로 **프로토콜 지향 프로그래밍(Protocol-Oriented Programming)**입니다!

### 익스텐션(Extension)과 프로토콜의 만남
지난 포스팅에서 배운 **익스텐션**을 프로토콜에 적용하면, 청사진에 불과했던 프로토콜에 **실제 구현(Default Implementation)**을 제공할 수 있습니다.

```swift
// 1. 프로토콜 정의 (청사진)
protocol Flyable {
    func fly()
}

// 2. 프로토콜의 기본 구현 제공 (POP의 핵심!)
extension Flyable {
    func fly() {
        print("푸드덕 푸드덕 날아갑니다!")
    }
}

// 3. 구조체에서 채택 (클래스가 아니어도 상속처럼 기능 재사용 가능!)
struct Bird: Flyable {
    // fly()를 직접 구현하지 않아도, 익스텐션의 기본 구현이 사용됩니다!
}

struct Airplane: Flyable {
    // 원한다면 자신의 상황에 맞게 재정의(Overriding) 할 수도 있습니다.
    func fly() {
        print("제트 엔진으로 날아갑니다!")
    }
}

let sparrow = Bird()
sparrow.fly() // "푸드덕 푸드덕 날아갑니다!"

let boeing747 = Airplane()
boeing747.fly() // "제트 엔진으로 날아갑니다!"
```

스위프트 표준 라이브러리에 정의되어 있는 타입은 실제 구현 코드를 보고 수정할 수 없기 때문에 익스텐션, 프로토콜, 프로토콜의 초기구현을 사용해 기본 타입에 기능을 추가해볼 수 있습니다.

```swift
protocol SelfPrintable {
    func printSelf()
}

extension SelfPrintable {
    func printSelf() {
        print(self)
    }
}

extension Int: SelfPrintable { }
extension String: SelfPrintable { }
extension Double: SelfPrintable { }

1004.printSelf() // 1004
"Hello".printSelf() // "Hello"
3.14.printSelf() // 3.14
```

### POP의 장점
1. **값 타입(Struct, Enum)의 활용**: 참조 타입인 클래스의 상속 대신, 안전한 값 타입들을 사용하면서도 코드를 재사용할 수 있습니다.
2. **수평적 확장 (다중 채택 가능)**: 한 타입이 여러 개의 프로토콜을 동시에 채택할 수 있으므로(다중 상속과 유사), 더 작고 명확한 단위로 기능을 분리해 조립할 수 있습니다.
3. **유연함**: 기존 타입에 소스코드를 건드리지 않고도 익스텐션으로 새로운 프로토콜을 채택시켜 기능을 확장할 수 있습니다.

---

### 마무리

4편과 5편에 걸쳐 Swift의 강력한 타입 시스템에 대해 알아보았습니다. 

Swift를 처음 배울 때는 클래스를 이용한 객체지향 방식으로 접근하기 쉽지만, 언어의 진정한 성능과 안정성을 이끌어내기 위해서는 **"구조체(값 타입) + 프로토콜 + 익스텐션"**을 조합하는 프로토콜 지향 프로그래밍 방식으로 사고의 전환을 해보는 것이 좋습니다. 애플 역시 Swift 표준 라이브러리 전체를 이 POP 패러다임을 기반으로 설계했습니다.

지금까지 길었던 Swift 기초/심화 스터디를 함께해 주셔서 감사합니다! 배운 문법들을 바탕으로 실제로 앱을 만들어보며 부딪혀보는 것이 가장 좋은 학습법입니다. 앞으로의 개발 여정을 응원합니다! 🚀
