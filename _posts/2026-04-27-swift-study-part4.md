---
layout: post
title: "[Swift 스터디 4편] 객체지향 심화: 서브스크립트부터 익스텐션까지"
date: 2026-04-27 19:00:00 +0900
categories: [Swift]
tags: [Swift, iOS, Study, OOP]
---

안녕하세요! Swift 스터디 네 번째 시간입니다. 

지난 시간들에서는 구조체, 클래스 등 객체지향의 기초와 함수형 프로그래밍에 대해 다루었습니다. 이번 포스팅부터는 Swift에서 기존 타입을 더욱 유연하고 강력하게 다룰 수 있게 해주는 심화 문법들을 2편에 걸쳐 알아보려 합니다.

오늘 다룰 4편에서는 인덱스를 통한 접근을 돕는 **서브스크립트**, 객체지향의 꽃인 **상속**과 **타입캐스팅**, 그리고 기존 타입에 새로운 기능을 덧붙일 수 있는 **익스텐션(확장)**에 대해 알아보겠습니다. 🚀

---

## 1. 서브스크립트 (Subscripts)

서브스크립트는 클래스, 구조체, 열거형의 컬렉션, 리스트, 시퀀스 등의 멤버 요소에 접근할 수 있는 **단축 문법**입니다. 우리가 흔히 배열의 특정 요소에 접근할 때 `array[0]`이나 딕셔너리에 접근할 때 `dictionary["key"]`와 같이 대괄호(`[]`)를 사용하는 것이 바로 서브스크립트입니다.

Swift에서는 개발자가 직접 정의한 타입에도 이 서브스크립트를 구현할 수 있습니다.

```swift
struct TimesTable {
    let multiplier: Int
    
    // 서브스크립트 정의
    subscript(index: Int) -> Int {
        return multiplier * index
    }
}

let threeTimesTable = TimesTable(multiplier: 3)
// 메서드 호출 없이 대괄호로 바로 접근 가능!
print("6에 3을 곱하면 \(threeTimesTable[6])입니다.") // 18
```

읽기 전용뿐만 아니라 `get`, `set`을 이용해 읽고 쓰기가 모두 가능한 서브스크립트도 구현할 수 있으며, 여러 개의 파라미터를 받는 다중 서브스크립트도 가능합니다.

---

## 2. 상속 (Inheritance)

상속은 객체지향 프로그래밍의 핵심 요소로, 한 클래스가 다른 클래스의 메서드, 프로퍼티, 기타 특성을 물려받는 것을 말합니다. Swift에서는 **클래스(Class)만이 상속을 지원**하며 구조체나 열거형은 지원하지 않습니다.

- **기본 클래스 (Base Class)**: 아무 클래스도 상속받지 않은 클래스
- **서브클래스 (Subclass)**: 다른 클래스를 상속받은 클래스

```swift
// 기본 클래스
class Vehicle {
    var currentSpeed = 0.0
    var description: String {
        return "현재 속도: \(currentSpeed) km/h"
    }
    func makeNoise() {
        // 기본 차량은 소음이 없음
    }
}

// Vehicle을 상속받는 서브클래스
class Bicycle: Vehicle {
    var hasBasket = false
}

let myBike = Bicycle()
myBike.currentSpeed = 15.0
print(myBike.description) // "현재 속도: 15.0 km/h"
```

### 오버라이딩 (Overriding)
서브클래스는 부모 클래스에서 물려받은 특성을 자신의 입맛에 맞게 재정의할 수 있습니다. 이때 `override` 키워드를 사용합니다. 재정의를 방지하고 싶다면 부모 클래스에서 `final` 키워드를 붙여주면 됩니다 (`final class` 또는 `final var`).

```swift
class Train: Vehicle {
    override func makeNoise() {
        print("칙칙폭폭!")
    }
    
    // 프로퍼티 재정의 (부모의 값을 활용하기 위해 super 사용)
    override var description: String {
        return super.description + " (기차입니다)"
    }
}
```

이니셜라이저 중 Swift에서의 특이한 점은 convenience와 required가 존재한다는 점입니다. 
convenience는 다른 이니셜라이저를 호출해서 객체를 생성하는 방식이고, required는 자식클래스에서 반드시 구현해야하는 이니셜라이저입니다. 

```swift
class Vehicle {
    var currentSpeed = 0.0

    // 기본 이니셜라이저
    init(speed: Double) {
        currentSpeed = speed
    }

    // 편의 이니셜라이저 (Convenience Initializer)
    // 다른 이니셜라이저를 호출해서 초기화함
    convenience init() {
        self.init(speed: 0.0)
    }

    required init(speed: Double) {
        self.currentSpeed = speed
    }
}

class Bicycle: Vehicle {
    var hasBasket: Bool
    
    // 서브클래스에서 부모의 기본 이니셜라이저를 호출하려면
    // super.init()을 반드시 호출해야 함!
    init(speed: Double, hasBasket: Bool) {
        self.hasBasket = hasBasket
        super.init(speed: speed) // ⭐️ 부모 이니셜라이저 호출 필수!
    }

    // ⭐️ Required Initializer (필수 이니셜라이저)
    // 부모의 모든 이니셜라이저를 여기서 구현해야 함
    required init(speed: Double) {
        self.hasBasket = false
        super.init(speed: speed)
    }

    convenience init(basket: Bool) {
        self.init(speed: 10.0, hasBasket: basket)
    }
}
```

---

## 3. 타입캐스팅 (Type Casting)

인스턴스의 타입을 확인하거나, 부모/자식 클래스 타입으로 변환하는 것을 **타입캐스팅**이라고 합니다. Swift에서는 `is`와 `as` 연산자를 사용합니다.

### 타입 확인 (`is`)
해당 인스턴스가 특정 서브클래스 타입인지 확인할 때 사용합니다. 결과를 `Bool`로 반환합니다.

```swift
let myTrain = Train()
if myTrain is Vehicle {
    print("myTrain은 Vehicle 타입이기도 합니다.")
}
```

### 다운캐스팅 (`as?`, `as!`)
부모 클래스 타입의 상자(참조)에 담겨있는 인스턴스를 실제 서브클래스 타입으로 다루고 싶을 때 다운캐스팅을 사용합니다. 실패할 가능성이 있으므로 두 가지 형태를 제공합니다.

- **`as?` (조건부 다운캐스팅)**: 실패하면 `nil`을 반환합니다. (안전함, 주로 `if let`과 함께 사용)
- **`as!` (강제 다운캐스팅)**: 캐스팅에 실패하면 런타임 에러(크래시)가 발생합니다. (확실할 때만 사용)

```swift
let someVehicle: Vehicle = Train() // 타입은 Vehicle이지만 실제론 Train 인스턴스

// 1. 안전한 캐스팅 (as?)
if let train = someVehicle as? Train {
    train.makeNoise() // 칙칙폭폭!
} else {
    print("기차가 아닙니다.")
}

// 2. 강제 캐스팅 (as!) - 확신이 있을 때만 사용!
let realTrain = someVehicle as! Train
```

---

## 4. 익스텐션 (Extensions)

익스텐션(확장)은 말 그대로 **기존의 클래스, 구조체, 열거형, 프로토콜 타입에 새로운 기능을 추가**하는 문법입니다. 소스 코드를 알 수 없는 기존 라이브러리나 기본 타입(`Int`, `String` 등)에도 마음대로 기능을 추가할 수 있어 매우 강력합니다!

익스텐션으로 추가할 수 있는 기능은 다음과 같습니다:
- 연산 프로퍼티 추가 (저장 프로퍼티는 추가 불가)
- 새로운 메서드 추가
- 새로운 이니셜라이저 추가
- 서브스크립트 추가
- 중첩 타입 추가
- 새로운 프로토콜 채택

익스텐션은 타입에 새로운 기능을 추가할 수는 있지만, 기존에 존재하는 기능을 재정의(Overriding)할 수는 없습니다.

```swift
// Double 기본 타입에 확장(Extension)을 통해 기능 추가하기
extension Double {
    var km: Double { return self * 1_000.0 }
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    
    // 메서드 추가
    func printMeters() {
        print("\(self) 미터입니다.")
    }
}

let distance = 25.4.km
print("25.4km는 \(distance)미터입니다.") // 25400.0미터입니다.
distance.printMeters()
```

익스텐션을 잘 활용하면, 관련된 코드들을 논리적인 블록으로 나누어 가독성을 높일 수 있으며 프로토콜과 결합하여 엄청난 시너지(Protocol-Oriented Programming)를 내게 됩니다.

---

### 마무리

이번 포스팅에서는 기존의 코드를 재사용하고 확장하는 객체지향의 중요한 문법들(서브스크립트, 상속, 타입캐스팅, 익스텐션)을 살펴보았습니다. 

특히 **익스텐션(Extension)**은 다른 언어에서는 찾아보기 힘든 Swift만의 큰 장점 중 하나로, 코드를 굉장히 깔끔하게 관리할 수 있게 도와줍니다.

다음 포스팅(5편)에서는 이번에 배운 익스텐션과 찰떡궁합인 **프로토콜**, 그리고 타입에 의존하지 않는 유연한 코드를 작성하게 해주는 **제네릭**, 최종적으로 Swift의 꽃인 **프로토콜 지향 프로그래밍(POP)**에 대해 다루어 보겠습니다. 기대해 주세요! 🚀
