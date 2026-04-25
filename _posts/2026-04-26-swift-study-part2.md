---
layout: post
title: "[Swift 스터디 2편] 객체지향 프로그래밍: 구조체와 클래스부터 접근제어까지"
date: 2026-04-26 02:00:00 +0900
categories: [Swift]
tags: [Swift, iOS, Study, OOP]
---

안녕하세요! Swift 스터디 두 번째 시간입니다. 지난 포스팅에서는 변수, 데이터 타입, 흐름 제어, 그리고 Swift의 핵심인 옵셔널에 대해 알아보았는데요.

오늘은 Swift를 활용해 본격적인 프로그램을 만드는데 필수적인 **객체지향 프로그래밍(OOP, Object-Oriented Programming)**의 핵심 요소들을 살펴보려고 합니다. 프로그램을 더 체계적이고 재사용 가능하게 만들어주는 구조체, 클래스, 프로퍼티, 메서드, 그리고 접근제어에 대해 정리해 보겠습니다! 🚀

---

## 1. 구조체와 클래스 (Structures and Classes)

Swift에서 데이터를 묶고 관련 기능을 포함하는 사용자 정의 타입을 만들 때 **구조체(Struct)**와 **클래스(Class)**를 사용합니다. 두 가지는 비슷해 보이지만 아주 중요한 차이가 있습니다.

### 구조체(Struct) vs 클래스(Class)

가장 큰 차이점은 데이터가 전달되는 방식입니다!

- **구조체 (Value Type, 값 타입)**: 데이터를 전달할 때 원본의 '복사본'을 전달합니다. 하나의 값이 변경되어도 다른 쪽에는 영향을 주지 않아 안전하며, Swift 표준 라이브러리의 대부분(String, Int, Array 등)이 구조체로 이루어져 있습니다. Swift는 구조체 사용을 권장합니다.
- **클래스 (Reference Type, 참조 타입)**: 데이터를 전달할 때 원본의 '메모리 주소'를 전달합니다. 여러 곳에서 하나의 인스턴스를 공유하므로 한쪽에서 값을 변경하면 다른 쪽도 변경됩니다. 상속(Inheritance)이 필요하거나 Objective-C와 상호 운용해야 할 때 주로 사용합니다.

```swift
// 구조체 선언
struct Resolution {
    var width = 0
    var height = 0
}

// 클래스 선언
class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}

// 인스턴스 생성
var someResolution = Resolution()
let someVideoMode = VideoMode()

// 값 타입 동작 방식
var cinema = someResolution
cinema.width = 2048
print("someResolution의 width: \(someResolution.width)") // 여전히 0! (서로 독립적)

// 참조 타입 동작 방식
let tenEighty = someVideoMode
tenEighty.frameRate = 25.0
let alsoTenEighty = tenEighty
alsoTenEighty.frameRate = 30.0
print("tenEighty의 frameRate: \(tenEighty.frameRate)") // 30.0! (서로 같은 주소를 가리킴)
```

스위프트는 꼭 필요한 경우에만 '진짜 복사'를 하므로, 구조체를 사용하는 것에 대해 성능 저하를 걱정하지 않아도 됩니다. 
컴파일러가 판단해서 꼭 복사를 할 필요가 없을 경우, 알아서 효율적으로 최적화해 줄 것입니다.

---

## 2. 프로퍼티와 메서드 (Properties and Methods)

구조체나 클래스, 열거형 내부에 있는 변수나 상수를 **프로퍼티**라고 하고, 함수를 **메서드**라고 합니다.

### 프로퍼티 (Properties)

- **저장 프로퍼티 (Stored Property)**: 값을 메모리에 저장하는 가장 단순한 형태의 프로퍼티입니다.
- **지연 저장 프로퍼티 (Lazy Stored Property)**: 처음 호출될 때까지 값이 초기화되지 않는 프로퍼티입니다. (`lazy` 키워드 사용)
- **연산 프로퍼티 (Computed Property)**: 실제 값을 저장하지 않고, 특정 연산을 통해 값을 반환하거나(`get`), 다른 프로퍼티의 값을 세팅하는(`set`) 프로퍼티입니다. 
연산 프로퍼티는 메서드로 구현할 때 보다 간결하고 직관적으로 표현할 수 있습니다.
- **타입 프로퍼티 (Type Property)**: 타입 자체에서 가지는 프로퍼티입니다.(`static` 또는 `class` 키워드 사용) 해당 타입의 모든 인스턴스가 공통으로 사용하는 값입니다.

```swift
struct Rect {
    // 저장 프로퍼티
    var width: Double = 0.0
    var height: Double = 0.0
    
    // 연산 프로퍼티
    var area: Double {
        get { return width * height }
        set(newArea) {
            // 다른 프로퍼티를 조작하여 면적에 맞는 값을 설정할 수 있음
            width = sqrt(newArea.width)
            height = sqrt(newArea.height)
        }
    }

    // 타입 프로퍼티
    static var maxHeight = 100
}

class DataImporter {
    var filename = "data.txt"
    // 복잡한 초기화가 필요한 클래스라면 초기화를 미루기 위해 지연 저장 프로퍼티 사용
    lazy var data: [String] = {
        // 파일에서 데이터를 읽어오는 복잡한 작업
        return [String]()
    }()
}

var importer = DataImporter() // importer는 선언과 동시에 초기화 됨
print(importer.data) // importer의 data는 지연 저장 프로퍼티이므로 접근 시 초기화 됨
```

연산 프로퍼티에서 newValue로 매개변수 이름을 대신할 수 있습니다.

```swift
struct Rect {
    // 저장 프로퍼티
    var width: Double = 0.0
    var height: Double = 0.0
    
    // 연산 프로퍼티
    var area: Double {
        get { return width * height }
        set {
            // 다른 프로퍼티를 조작하여 면적에 맞는 값을 설정할 수 있음
            width = sqrt(newValue.width)
            height = sqrt(newValue.height)
        }
    }
}
```

```swift
class AClass {
    // 저장 타입 프로퍼티
    static var typeProperty: Int = 0
    
    // 저장 인스턴스 프로퍼티
    var instanceProperty: Int = 0 {
        didSet {
            Self.typeProperty = instanceProperty + 100   
        }
    }

    // 연산 타입 프로퍼티
    class var typeComputedProperty: Int {
        get {
            return typeProperty
        }
        set {
            typeProperty = newValue
        }
    }
}

AClass.typeProperty = 123

let a: AClass = AClass()
a.instanceProperty = 100

print(AClass.typeProperty) // 200
print(AClass.typeComputedProperty) // 200
```

### 프로퍼티 감시자
프로퍼티 감시자를 사용하면 프로퍼티의 값이 변경됨에 따라 적절한 작업을 취할 수 있습니다. 
프로퍼티 감시자에는 프로퍼티의 값이 변경되기 직전에 호출하는 (`willSet`) 메서드와 프로터피의 값이 변경된 직후에 호출하는 (`didSet`) 메서드가 있습니다.
(`willSet`) 메서드에는 변경될 값인 (`newValue`)가, (`didSet`) 메서드에는 변경되기 전 값인 (`oldValue`)가 매개변수로 전달됩니다.

```swift
class Account {
    var credit: Int = 0 {
        willSet {
            print("잔액이 \(credit)원에서 \(newValue)원으로 변경될 예정입니다.")
        }

        didSet {
            print("잔액이 \(oldValue)원에서 \(credit)원으로 변경되었습니다.")
        }
    }
}
```

앞서 설명한 연산 프로퍼티와 프로퍼티 감시자는 전역변수와 지역변수 모두에 사용할 수 있습니다.


### 메서드 (Methods)

- **인스턴스 메서드 (Instance Method)**: 인스턴스를 생성해야만 호출할 수 있는 메서드입니다.
- **타입 메서드 (Type Method)**: 인스턴스를 생성하지 않고 타입 자체에서 호출할 수 있는 메서드입니다. (`static` 또는 `class` 키워드 사용)

```swift
class Counter {
    var count = 0
    
    // 생성된 모든 Counter 객체를 추적하기 위한 배열
    static var allInstances: [Counter] = []
    
    init() {
        Counter.allInstances.append(self)
    }

    // 인스턴스 메서드
    func increment() {
        count += 1
    }
    
    // 타입 메서드 (클래스 레벨에서 동작)
    static func resetCounter() {
        // 관리 중인 모든 인스턴스를 순회하며 리셋
        for instance in allInstances {
            instance.count = 0
        }
    }
}

let c1 = Counter()
let c2 = Counter()

c1.increment() // c1.count = 1
c2.increment() // c2.count = 1

Counter.resetAllInstances() // c1, c2의 count가 각각 0으로 초기화됨
```

*참고: 구조체(값 타입) 내에서 프로퍼티의 값을 변경하는 인스턴스 메서드를 작성할 때는 `mutating` 키워드를 붙여야 합니다.*

---

## 3. 인스턴스 생성 및 소멸 (Initialization & Deinitialization)

### 이니셜라이저 (Initializer)
클래스나 구조체, 열거형 인스턴스를 생성할 때, 모든 저장 프로퍼티는 초기값을 가지고 있어야 합니다. 초기화 과정은 `init` 키워드를 사용해 구현합니다.

```swift
struct Fahrenheit {
    var temperature: Double
    
    // 초기화 구문
    init() {
        temperature = 32.0 // 프로퍼티에 기본값 할당
    }
    
    // 매개변수가 있는 초기화 구문
    init(fromKelvin kelvin: Double) {
        temperature = kelvin * 1.8 - 459.67
    }
}

var f1 = Fahrenheit() 
var f2 = Fahrenheit(fromKelvin: 273.15)
```

매개변수가 없는 `init()`은 기본 이니셜라이저이며, 저장 프로퍼티에 명시적으로 초기값이 할당되어 있으면 자동으로 제공되기도 합니다. 특히 구조체는 모든 저장 프로퍼티를 매개변수로 받는 **멤버와이즈 이니셜라이저(Memberwise Initializer)**를 자동으로 제공해 줍니다.

### 디이니셜라이저 (Deinitializer)
디이니셜라이저는 클래스 인스턴스가 메모리에서 해제되기 직전에 호출됩니다. `deinit` 키워드를 사용하며, **클래스에서만** 사용할 수 있습니다. 파일 닫기나 메모리 관련 추가 해제 작업이 필요할 때 유용합니다.

```swift
class Player {
    var coinsInPurse: Int
    init(coins: Int) {
        coinsInPurse = coins
    }
    
    deinit {
        print("Player 인스턴스가 메모리에서 해제되었습니다.")
    }
}

var playerOne: Player? = Player(coins: 100)
playerOne = nil // 이 시점에 deinit 블록이 실행됨
```

---

## 4. 접근제어 (Access Control)

프로그램의 크기가 커지면 보안이나 캡슐화를 위해 특정 코드 세부 구현을 감춰야 할 때가 있습니다. Swift는 요소의 접근 수준을 지정할 수 있는 다양한 키워드를 제공합니다.

접근 수준은 가장 개방적인 것부터 가장 제한적인 순으로 다음과 같습니다:

1. **`open`**: 모듈 외부에서도 접근 가능하며, 클래스를 다른 모듈에서 상속하거나 오버라이드할 수 있습니다. (클래스와 클래스 멤버에만 사용)
2. **`public`**: 모듈 외부에서 접근 가능하지만, 다른 모듈에서 상속/오버라이드는 불가능합니다. (프레임워크 작성 시 주로 사용)
3. **`internal`** (기본값): 같은 모듈(.app, .framework 등) 내의 모든 소스 파일에서 자유롭게 접근할 수 있습니다.
4. **`fileprivate`**: 선언된 소스 파일 내부에서만 접근할 수 있습니다.
5. **`private`**: 선언된 블록과 동일한 파일 내의 extension 안에서만 허용되는 가장 제한적인 접근 수준입니다.

```swift
public class SomePublicClass {                 // 외부 모듈에서도 접근 가능한 클래스
    public var somePublicProperty = 0          // 외부 모듈에서도 접근 가능한 프로퍼티
    var someInternalProperty = 0               // 기본 접근 수준(internal), 같은 모듈 내에서 접근 가능
    fileprivate func someFilePrivateMethod() {}// 이 소스 파일 내에서만 접근 가능한 메서드
    private func somePrivateMethod() {}        // 이 클래스 정의 내부에서만 접근 가능한 메서드
}
```

접근제어를 적극적으로 활용하여 외부에서 알 필요가 없는 내부 프로퍼티나 로직은 `private` 등으로 숨겨(캡슐화), 더 안전하고 결합도가 낮은 코드를 작성하는 것이 객체지향 프로그래밍의 핵심 원칙 중 하나입니다.

---

### 마무리

오늘은 Swift에서 객체지향 프로그래밍을 할 때 필수적인 요소들인 구조체/클래스의 차이점, 프로퍼티와 메서드, 초기화 및 해제, 그리고 접근제어에 대해 알아보았습니다. 코드를 설계할 때 **"값 타입(구조체)을 쓸까, 참조 타입(클래스)을 쓸까?"**, **"이 맴버는 `private`로 감춰야 할까?"** 고민하면서 작성해 보면 Swift의 매력을 더 깊이 느낄 수 있을 것입니다.

다음 시간에도 Swift의 유용한 기능들을 계속해서 이어나가 보겠습니다! 🚀
