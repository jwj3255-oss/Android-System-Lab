---
layout: post
title: "[Swift 스터디 1편] Swift의 세계로: 기초 문법부터 옵셔널까지"
date: 2026-04-24 13:51:00 +0900
categories: [Swift]
tags: [Swift, iOS, Study]
---

안녕하세요! 이번 포스팅을 시작으로 애플(Apple)의 모바일 및 데스크톱 앱 개발을 위한 핵심 언어인 **Swift(스위프트)**에 대해 스터디해 보려고 합니다. 

최근 Apple에서 Wi-Fi Aware 프로토콜을 공식 지원하게 됨으로써 저희 부서에서도 Swift를 이용한 iOS Wi-Fi Aware 테스트 애플리케이션을 작성할 기회가 생겨 공부하게 되었습니다. 

Swift는 강력하면서도 직관적인 언어로, '안전성(Safe)'과 '신속성(Fast)'을 매우 중요하게 생각하는 언어입니다. 오늘은 그 첫걸음으로 Swift를 다루기 위해 반드시 알아야 하는 기초 문법부터, Swift의 꽃이라고 불리는 '옵셔널(Optional)'까지 한 번에 정리해 보겠습니다! 🚀

---

## 1. 스위프트 기초 (Basics)

프로그래밍을 시작할 때 가장 먼저 배우는 것은 값을 저장하는 방법입니다. Swift에서는 **변수**와 **상수**를 명확하게 구분하여 사용합니다.

- `let` (상수): 한 번 값을 할당하면 이후에 변경할 수 없습니다. (안전한 코딩을 위해 스위프트가 권장하는 방식입니다!)
- `var` (변수): 값을 언제든지 변경할 수 있습니다.

```swift
let maximumNumberOfLoginAttempts = 10 // 상수: 값 변경 불가
var currentLoginAttempt = 0           // 변수: 값 변경 가능

// currentLoginAttempt = 1 (가능 - var)
// maximumNumberOfLoginAttempts = 11 (오류 발생! - let)
```

Swift는 똑똑하게도 우리가 값을 넣으면 자동으로 타입을 유추하는 **타입 추론(Type Inference)** 기능을 지원하기 때문에 굳이 타입을 명시하지 않아도 됩니다. 

print() 함수에 대해서도 알아봅시다. 스위프트 표준 라이브러리에는 print() 함수 외에도 dump() 함수가 있습니다. dump() 함수는 print() 함수보다 더 자세한 정보를 출력합니다. 

```swift
let name = "원준"
let age = 25

print(name, age) // 원준 25
dump(name, age) // name: "원준", age: 25
```

Swift 만의 독특한 주석인 마크업 문법을 활용한 문서화 주석에 대해서도 알아봅시다. 

```swift
/// 함수에 대한 설명을 적습니다.
/// - Parameters:
///   - name: 함수의 매개변수
///   - age: 함수의 매개변수
/// - Returns: 함수의 반환값
func sayHello(name: String, age: Int) -> String {
    return "Hello, \(name)!"
}
```
마크업 문법을 따른 위 주석은 퀵헬프를 통해 확인할 수 있습니다.

---

## 2. 데이터 타입 기본 (Basic Data Types)

Swift에는 다양한 기본 데이터 타입이 존재합니다. 

- **Int / UInt**: 정수형 타입 (음수 포함 / 0과 양수만)
- **Float / Double**: 실수형 타입 (32비트 / 64비트) — *Swift는 주로 Double 사용을 권장합니다.*
- **Bool**: 참/거짓 (true / false)
- **Character**: 단일 문자 (`"A"`, `"가"`)
- **String**: 문자열 (`"Hello, Swift!"`)

```swift
let age: Int = 25
let height: Double = 175.5
let isDeveloper: Bool = true
let greeting: String = "안녕하세요!"
```

---

## 3. 데이터 타입 고급 (Advanced Data Types)

기본 데이터 타입 외에도 여러 데이터를 하나로 묶어서 관리하는 **컬렉션(Collection)** 타입들이 있습니다.

- **Array (배열)**: 순서(Index)가 있는 데이터의 집합
- **Dictionary (딕셔너리)**: 순서 없이 키(Key)와 값(Value)의 쌍으로 이루어진 집합
- **Set (세트)**: 순서가 없고, 중복을 허용하지 않는 집합

```swift
// Array
var names: [String] = ["철수", "영희", "민수"]
names.append("지연") // 배열에 데이터 추가

// Dictionary
var capitals: [String: String] = ["한국": "서울", "미국": "워싱턴 D.C."]
print(capitals["한국"]!) // "서울" 

// Set (중복 제거에 유용)
var numbers: Set<Int> = [1, 2, 3, 3, 4] // [1, 2, 3, 4] 로 저장됨
```

여기에 추가로 어떤 타입이든 받을 수 있는 `Any`와 클래스 인스턴스만 받는 `AnyObject` 같은 범용 타입도 존재하지만, 사용하지 않는 것을 추천합니다.

---

## 4. 연산자 (Operators)

Swift의 연산자는 다른 프로그래밍 언어와 거의 유사합니다.

- **산술 연산자**: `+`, `-`, `*`, `/`, `%` (나머지)
- **비교 연산자**: `==`, `!=`, `>`, `<`, `>=`, `<=`, `===`,
- **논리 연산자**: `&&`(AND), `||`(OR), `!`(NOT)
- **삼항 연산자**: `조건 ? 참일 때 값 : 거짓일 때 값`

```swift
let score = 85
let hasPassed = score >= 80 // true

// 삼항 연산자
let message = hasPassed ? "합격입니다" : "불합격입니다" 

// 참조가 같은 인스턴스를 가르키는지 비교
let referenceA: SomeClass = SomeClass()
let referenceB: SomeClass = SomeClass()
let referenceC: SomeClass = referenceA

print(referenceA === referenceB) // false
print(referenceA === referenceC) // true
```

다른 점이 있다면, 오버플로에 대비한 연산을 해주는 오버플로 연산자와 nil 병합 연산자가 있습니다.

- **오버플로 연산자**: `&+`, `&-`, `&*`, `&/`, `&%`
- **nil 병합 연산자**: `??`

```swift
let a: UInt8 = 255
let b: UInt8 = 1

print(a &+ b) // 0

let optionalName: String? = nil
let name: String = optionalName ?? "손님"

print(name) // "손님"
```

---

## 5. 흐름 제어 (Control Flow)

### 조건문 (if, switch)
Swift의 `switch`문은 다른 언어에 비해 훨씬 강력합니다! 모든 경우의 수를 덮어야(Exhaustive) 하며, `break`를 쓰지 않아도 조건에 맞으면 자동으로 빠져나옵니다.
또한, case 구문에 범위 연산자를 사용할 수도 있습니다.

```swift
let fruit = "Apple"

if fruit == "Apple" {
    print("사과입니다.")
} else {
    print("사과가 아닙니다.")
}

// Swift의 강력한 switch문
switch fruit {
case "Apple":
    print("맛있는 사과")
case "Banana", "Mango": // 여러 개 동시 매칭 가능
    print("노란 과일")
default: // 반드시 default가 있어야 함 (모든 케이스를 다루지 않았다면)
    print("알 수 없는 과일")
}

let integerValue: Int = 5

swtich integerValue {
case 0:
    print("Value == zero")
case 1...10:
    print("Value is between 1 and 10")
case Int.min..<0, 101..<Int.max:
    print("Value < 0 or Value > 100")
default:
    print("10 < Value <= 100")
}

// 결과
// Value is between 1 and 10
```

### 반복문 (for-in, while)
배열이나 딕셔너리 같은 컬렉션을 순회할 때는 `for-in`을 주로 사용합니다.

```swift
let maxCount = 3
for i in 1...maxCount {  // 1부터 3까지 (1, 2, 3)
    print("\(i)번째 실행") 
}

var count = 0
while count < 3 {
    print("while 문 실행")
    count += 1
}
```
다른 프로그래밍 언어의 do-while에 해당하는 것은 repeat-while 구문입니다.

```swift
var count = 0
repeat {
    print("repeat-while 문 실행")
    count += 1
} while count < 3
```

---

## 6. 함수 (Functions)

함수는 특정 작업을 수행하는 코드의 블록입니다. Swift에서는 `func` 키워드를 사용합니다.
특히 Swift의 함수는 **전달인자 레이블(Argument Label)**을 가질 수 있어, 코드를 읽을 때 마치 영어 문장을 읽는 것처럼 자연스럽게 만들어줍니다.
전달인자 레이블을 사용하고 싶지 않으면 와일드카드 식별자를 사용하면 됩니다.

```swift
// name은 매개변수 이름이자 전달인자 레이블
func sayHello(name: String) -> String {
    return "Hello, \(name)!"
}

// from은 밖에서 쓸 때(전달인자 레이블), 
// origin은 안에서 쓸 때(매개변수 이름)
func go(from origin: String, to destination: String) {
    print("\(origin)에서 출발하여 \(destination)으로 갑니다.")
}

sayHello(name: "Swift")
go(from: "서울", to: "부산")

// 전달인자 레이블이 없는 함수 정의와 사용
func sayHello(_ name: String) -> String {
    return "Hello, \(name)!"
}

sayHello("Swift")
```

### 빌림과 소비 매개변수
스위프트에서는 전달인자가 어떻게 사용될 것인지 borrowing이나 consuming을 붙여 더욱 상세한 제약을 부여할 수 있습니다.
borrowing 제어자는 함수가 매개변수의 값을 유지하지 않는다는 것을 나타냅니다. 이 경우 호출자에게 전달인자의 소유권이 있습니다.

```swift
var storedValue: Int = 0

func isLessThan(lhs: borrowing Int, rhs: borrowing Int) -> Bool {
    return lhs < rhs
}

func store(_ value: borrowing Int) {
    storedValue = value // 오류 발생!! 빌린 값은 암시적으로 복사할 수 없습니다.
}
```

반면, consuming 매개변수 제어자는 함수가 전달인자의 소유권을 갖고 함수가 종료되기전에 해당 값을 저장하거나 파괴하는 책임을 갖게 됩니다.
consuming은 함수를 호출하는 쪽에서 함수 호출 후에 더 이상 전달 값을 사용할 필요가 없을 때 오버헤드를 줄일 수 있습니다.

```swift
func store(_ value: consuming Int) {
    storedValue = value // 오류 없음! 소유권을 가져왔기 때문에 저장 가능
}
```

---

## 7. 옵셔널 (Optionals) - ⭐️ Swift의 꽃!

다른 언어에서 `NullPointerException`으로 고통받으셨던 분들, 주목해 주세요! Swift에는 **"값이 있을 수도, 없을 수도 있음(nil)"**을 명확히 나타내는 **옵셔널**이라는 개념이 있습니다.

타입 뒤에 `?`를 붙여서 옵셔널을 만듭니다.

```swift
var optionalString: String? = "안녕하세요"
optionalString = nil // 에러 아님! (옵셔널이므로 nil 할당 가능)

var normalString: String = "반갑습니다"
// normalString = nil // 🚨 컴파일 에러 발생!
```

**옵셔널은 포장지(Box)와 같습니다.** 안에 값이 있는지 없는지 모르기 때문에, 값을 안전하게 꺼내 쓰는 **옵셔널 바인딩(Optional Binding)** 과정을 거쳐야 합니다.

### 옵셔널 값을 안전하게 꺼내는 방법
1. **if let / guard let (옵셔널 바인딩)**: 값이 있으면 꺼내서 쓰고, 없으면 무시합니다. (가장 안전하고 추천하는 방법)
2. **?? (Nil 병합 연산자)**: 값이 없으면 기본값을 제공합니다.
3. **! (강제 추출)**: 느낌표를 써서 포장지를 강제로 찢고 꺼냅니다. (값이 없으면 앱이 크래시 나므로 절대 비추천!)

```swift
var myName: String? = "원준"

// 1. if let 사용 (안전!)
if let name = myName {
    print("제 이름은 \(name)입니다.")
} else {
    print("이름이 없습니다.")
}

// 2. nil 병합 연산자 (간편!)
print("제 이름은 \(myName ?? "무명")입니다.")

// 3. 강제 추출 (위험!)
print(myName!) 
```
