---
layout: post
title: "[Swift 스터디 3편] 함수형 프로그래밍: 클로저부터 모나드까지"
date: 2026-04-27 17:00:00 +0900
categories: [Swift]
tags: [Swift, iOS, Study, Functional]
---

안녕하세요! Swift 스터디 세 번째 시간입니다. 지난 포스팅에서는 구조체와 클래스 등 객체지향 프로그래밍(OOP)의 특징을 살펴보았습니다.

Swift는 객체지향 프로그래밍 패러다임과 더불어 **함수형 프로그래밍(Functional Programming)** 패러다임도 함께 수용하고 있는 멀티 패러다임 언어입니다. 함수형 프로그래밍은 상태 변화를 최소화하고, 함수의 실행이 프로그램의 상태에 영향을 미치지 않는 '순수 함수'를 지향하여 코드를 더 안전하게 만들어줍니다.

오늘은 Swift에서 함수형 프로그래밍을 구현하는 데 핵심이 되는 요소들인 클로저, 옵셔널 체이닝, 고차 함수(맵, 필터, 리듀스), 그리고 모나드까지 깊이 있게 알아보겠습니다! 🚀

---

## 1. 클로저 (Closures)

클로저는 **코드에서 전달 및 사용할 수 있는 독립적인 블록(일급 객체)**입니다. 우리가 흔히 알고 있는 함수(`func`)도 사실 이름이 있는 클로저의 한 종류입니다. 클로저는 변수나 상수에 저장할 수 있고, 함수의 매개변수로 전달할 수도 있습니다.

```swift
// 일반적인 함수
func add(a: Int, b: Int) -> Int {
    return a + b
}

// 클로저 표현식
let addClosure: (Int, Int) -> Int = { (a: Int, b: Int) -> Int in
    return a + b
}

print(addClosure(3, 4)) // 7
```

클로저의 가장 강력한 장점 중 하나는 **문법의 간소화**입니다. Swift 컴파일러의 타입 추론 기능을 이용하면 코드를 굉장히 짧게 줄일 수 있습니다.

```swift
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]

// 1. 기본 클로저 전달
var reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 > s2
})

// 2. 타입 유추 (매개변수와 반환 타입 생략 가능)
reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 })

// 3. 단일 표현식 클로저의 암시적 반환 (return 생략)
reversedNames = names.sorted(by: { s1, s2 in s1 > s2 })

// 4. 짧은 인자 이름 ($0, $1 사용)
reversedNames = names.sorted(by: { $0 > $1 })

// 5. 후행 클로저 (마지막 인자가 클로저일 경우 괄호 밖으로 빼기)
reversedNames = names.sorted { $0 > $1 }

// 6. 연산자 메서드 (가장 극단적인 축약)
reversedNames = names.sorted(by: >)
```

클로저는 정의된 위치의 주변 문맥을 통해 상수나 변수를 캡처할 수 있습니다.

```swift
func makeAdder(for number: Int) -> () -> Int {
    var runningTotal = 0
    // 외부의 number를 캡처하여 클로저 내에서 사용
    return { () -> Int in
        runningTotal += number
        return runningTotal
    }
}

let addTwo = makeAdder(for: 2)
print(addTwo()) // 2
print(addTwo()) // 4
print(addTwo()) // 6
```

함수의 전달인자로 전달한 클로저가 함수 종료 후에 호출될 때 클로저가 함수를 탈출한다고 표현합니다.
클로저 매개변수 이름의 콜론(:) 뒤에 @escaping 키워드를 사용하여 클로저가 탈출하는 것을 허용한다고 명시해줄 수 있습니다.
```swift
typealias VoidVoidClosure = () -> Void

func functionWithNoescapeClosure(closure: VoidVoidClosure) {
    closure() 
}

func functionWithEscapingClosure(completionHandler: @escaping 
    VoidVoidClosure) -> VoidVoidClosure {
    return completionHandler
}

class SomeClass {
    var x = 10

    func runNoescape() { 
        functionWithNoescapeClosure { 
            x = 200 // 비탈출 클로저에서 self 키워드는 선택 사항임
        }
    }

    func runEscaping() -> VoidVoidClosure {
        return functionWithEscapingClosure { 
            self.x = 300 // 탈출 클로저에서 self 키워드는 필수임.
        }
    }
}

let instance = SomeClass()
instance.runNoescape()
print(instance.x) // 200

let escapingClosure = instance.runEscaping()
escapingClosure() // 탈출 클로저가 실행됨. 
print(instance.x) // 300

```

---

## 2. 옵셔널 체이닝과 빠른 종료 (Optional Chaining & Early Exit)

함수형 프로그래밍에서는 예기치 않은 상태(nil)를 안전하게 다루는 것이 중요합니다.

### 옵셔널 체이닝 (Optional Chaining)

옵셔널 체이닝은 옵셔널을 반복적으로 풀지 않고, 하위 프로퍼티나 메서드에 안전하게 접근하는 방식입니다. 체인 중간에 하나라도 `nil`이 있다면 전체 결과가 `nil`이 됩니다. `?.` 문법을 사용합니다.

```swift
class Room { var number = 101 }
class Address { var room: Room? }
class Person { var address: Address? }

let minsu = Person()

// 옵셔널 체이닝을 사용하지 않았을 때 (강제 추출 - 위험!)
// let roomNumber = minsu.address!.room!.number 

// 옵셔널 체이닝 사용
if let roomNumber = minsu.address?.room?.number {
    print("민수의 방 번호는 \(roomNumber)입니다.")
} else {
    print("방 번호를 알 수 없습니다.") // 현재 address가 nil이므로 여기가 실행됨
}
```

### 빠른 종료 (Early Exit - guard)

`guard` 구문은 조건이 거짓(`false`)일 때 현재 실행 중인 코드 블록(주로 함수)을 빠르게 빠져나오게 해줍니다. 깊은 중첩(Depth)을 방지하고 가독성을 크게 높여줍니다.

```swift
func printAge(age: Int?) {
    // if let을 사용한 경우 (Depth가 깊어짐)
    if let safeAge = age {
        print("나이는 \(safeAge)살 입니다.")
    } else {
        print("나이가 없습니다.")
        return
    }

    // guard let을 사용한 경우 (빠른 종료)
    guard let safeAge2 = age else {
        print("나이가 없습니다.")
        return
    }
    // guard를 통과하면 safeAge2를 함수 전체에서 자유롭게 사용 가능!
    print("나이는 \(safeAge2)살 입니다.")
}
```

---

## 3. 맵, 필터, 리듀스 (Map, Filter, Reduce)

Swift는 배열, 딕셔너리 같은 컬렉션을 처리하기 위해 강력한 **고차 함수(Higher-order Function)**를 제공합니다. `for-in` 루프를 대체하여 코드를 훨씬 선언적(Declarative)으로 만들어줍니다.

- **Map (맵)**: 컨테이너가 담고 있던 각각의 데이터를 꺼내서 클로저를 통해 변형한 후, 새로운 컨테이너를 반환합니다.
- **Filter (필터)**: 컨테이너 내부의 값을 걸러서 새로운 컨테이너로 추출합니다. 클로저의 반환값이 참(`true`)인 항목만 남깁니다.
- **Reduce (리듀스)**: 컨테이너 내부의 요소들을 하나로 합칩니다.

```swift
let numbers = [1, 2, 3, 4, 5]

// 1. Map (각 숫자에 2를 곱하기)
let doubledNumbers = numbers.map { $0 * 2 }
print(doubledNumbers) // [2, 4, 6, 8, 10]

// 2. Filter (짝수만 걸러내기)
let evenNumbers = numbers.filter { $0 % 2 == 0 }
print(evenNumbers) // [2, 4]

// 3. Reduce (모든 숫자의 합 구하기 - 초기값 0)
let sum = numbers.reduce(0) { $0 + $1 }
// 또는 더 축약하여: let sum = numbers.reduce(0, +)
print(sum) // 15

// 응용: 짝수만 골라서 2배로 만든 뒤 모두 합하기 (체이닝)
let result = numbers.filter { $0 % 2 == 0 }
                    .map { $0 * 2 }
                    .reduce(0, +)
print(result) // (2*2) + (4*2) = 12
```

이처럼 고차 함수를 연결(Chaining)하여 사용하면 어떤 작업(What)을 하는지에 집중할 수 있어 코드가 명확해집니다.

---

## 4. 모나드 (Monad)

함수형 프로그래밍을 공부하다 보면 **모나드(Monad)**라는 무시무시한 단어를 만나게 됩니다. 수학적 개념에서 출발했지만, 프로그래밍에서는 **"값이 있을 수도 있고 없을 수도 있는 상태(Context)를 감싸는 포장지"**라고 이해하면 쉽습니다. 

놀랍게도 Swift의 **옵셔널(Optional)**이 바로 모나드의 대표적인 예시입니다! `Optional`은 `nil`일지도 모르는 값을 안전하게 다룰 수 있도록 `Optional`이라는 문맥 안에 감싸놓은 것입니다.

모나드의 핵심은 값을 포장한 상태에서도 연산을 이어갈 수 있게 해주는 `flatMap` (또는 `compactMap`)입니다.

### flatMap과 compactMap

- **`map`**: 옵셔널 내부의 값을 변형하지만, 결과가 다시 옵셔널로 감싸집니다. 중첩된 옵셔널(`Optional<Optional<Int>>`)이 발생할 수 있습니다.
- **`flatMap`** / **`compactMap`**: 컨텍스트(포장지) 내부의 컨텍스트를 **평평하게(Flatten)** 펴줍니다. (Swift 4.1 이후로 1차원 배열에서 `nil`을 제거하고 옵셔널 바인딩을 할 때는 `compactMap`을 사용합니다.)

```swift
let optionalString: String? = "123"

// map을 사용한 경우 (결과가 Optional(Optional(123)) 즉, Int?? 형태가 됨)
let mapResult: Int?? = optionalString.map { Int($0) }

// flatMap을 사용한 경우 (결과가 Optional(123) 즉, Int? 형태로 평평해짐)
let flatMapResult: Int? = optionalString.flatMap { Int($0) }

// 배열에서의 compactMap (nil을 걸러내고 유효한 값만 추출)
let stringArray = ["1", "2", "three", "4"]
let intArray = stringArray.compactMap { Int($0) }
print(intArray) // [1, 2, 4] (변환에 실패한 "three"의 nil은 무시됨)
```

모나드를 이해하면 안전하게 데이터 파이프라인을 구축하고, 옵셔널 체인처럼 예기치 않은 상황을 유연하게 넘기는 함수형 코드를 작성할 수 있습니다.

---

### 마무리

오늘은 Swift의 함수형 프로그래밍 파트인 클로저, 옵셔널 체이닝 및 빠른 종료, 고차 함수(맵, 필터, 리듀스), 그리고 모나드까지 살펴보았습니다. 

처음에는 `$0`이나 클로저 문법이 낯설 수 있지만, 손에 익으면 객체지향 방식만 사용할 때보다 훨씬 간결하고 부작용(Side Effect)이 적은 강력한 코드를 작성할 수 있습니다. Swift가 왜 함수형 패러다임을 강조하는지 꼭 직접 코드를 작성하며 느껴보시길 바랍니다! 

다음 포스팅에서도 유익한 Swift 주제로 찾아뵙겠습니다. 감사합니다! 🚀
