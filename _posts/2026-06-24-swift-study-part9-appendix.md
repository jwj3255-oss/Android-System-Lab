---
layout: post
title: "[Swift 스터디 Appendix] 타 언어 개발자가 보면 생소한 Swift 특유의 문법 및 기능 정리"
date: 2026-06-24 23:25:00 +0900
categories: [Swift]
tags: [Swift, iOS, Study, Appendix]
---

안녕하세요! Swift 스터디의 부록(Appendix) 편입니다. 

C, C++, Java, Kotlin 등 다른 프로그래밍 언어의 기본 지식은 가지고 있지만, Swift를 처음 접했을 때 **"이건 다른 언어에선 못 보던 건데?"** 혹은 **"이름은 비슷한데 동작 방식이 특이하네?"** 싶은 Swift만의 고유하고 생소한 기능들을 모아서 정리했습니다.

이름만 봐도 대략 유추할 수 있는 평범한 예약어나 문법은 제외하고, Swift 특유의 색깔을 가진 핵심 요소들만 5가지 카테고리로 나누어 정리해 보겠습니다! 🚀

---

## 1. 주요 프로토콜 (Protocols)

Swift는 **프로토콜 지향 언어**이기 때문에, 표준 라이브러리의 다양한 마법 같은 기능들이 프로토콜을 통해 제공됩니다.

### `Identifiable`
- **개념**: 인스턴스를 고유하게 식별할 수 있음을 나타내는 프로토콜입니다.
- **특징**: 이 프로토콜을 채택하려면 반드시 고유한 ID 값을 가지는 `id` 프로퍼티를 정의해야 합니다. 다른 언어에서는 인터페이스 없이 그냥 멤버 필드(`id`)를 두는 편이지만, Swift(특히 SwiftUI)에서는 컬렉션 뷰에서 개별 요소를 구분하기 위해 필수적으로 활용됩니다.
- **예시**:
  ```swift
  struct User: Identifiable {
      let id = UUID() // 고유 식별자 생성
      let name: String
  }
  ```

### `CaseIterable`
- **개념**: 열거형(Enum)의 모든 케이스를 배열처럼 순회할 수 있는 컬렉션을 자동 생성해 주는 프로토콜입니다.
- **특징**: 다른 언어의 `Enum.values()`와 유사하지만, Swift는 컴파일러가 자동으로 `allCases`라는 타입 프로퍼티를 만들어 이를 배열처럼 다룰 수 있게 해줍니다.
- **예시**:
  ```swift
  enum Direction: CaseIterable {
      case east, west, south, north
  }
  for direction in Direction.allCases {
      print(direction)
  }
  ```

### `ExpressibleBy...Literal` 계열
- **개념**: 사용자 정의 타입을 일반 리터럴(정수, 문자열 등)로 직접 대입하여 초기화할 수 있도록 지원하는 프로토콜 군입니다.
- **특징**: 다른 언어에서는 객체를 생성할 때 항상 `new MyClass("value")` 형식을 갖추어야 하지만, Swift에서는 아래와 같이 선언하여 대입만으로 객체를 빌드할 수 있습니다.
- **예시**:
  ```swift
  struct Email: ExpressibleByStringLiteral {
      let address: String
      // 문자열 리터럴로 생성될 때의 이니셜라이저 구현
      init(stringLiteral value: String) {
          self.address = value
      }
  }
  let myEmail: Email = "user@example.com" // 마치 문자열을 대입하듯 초기화!
  ```

### `Codable`
- **개념**: 외부 데이터 포맷(예: JSON, XML 등)과 Swift 모델 간의 인코딩/디코딩(직렬화/역직렬화)을 처리할 수 있음을 나타내는 유니온 프로토콜(`Encodable & Decodable`)입니다.
- **특징**: Java나 C# 같은 언어에서는 JSON 직렬화를 하려면 Jackson/Gson 등의 외부 라이브러리를 임포트하고 복잡한 어노테이션이나 리플렉션을 거쳐야 하지만, Swift에서는 타입 선언 시 `Codable` 프로토콜을 붙여두기만 하면 컴파일러가 모든 파싱 및 직렬화 코드를 뒤에서 자동으로 생성(Synthesize)해 줍니다.
- **예시**:
  ```swift
  struct Product: Codable {
      let name: String
      let price: Int
  }
  
  let jsonData = """
  {
      "name": "아이패드",
      "price": 800000
  }
  """.data(using: .utf8)!
  
  // JSON 디코딩이 단 두 줄로 완료!
  let product = try JSONDecoder().decode(Product.self, from: jsonData)
  print(product.name) // 아이패드
  ```

### `Comparable`
- **개념**: 타입의 두 인스턴스를 서로 비교하여 순서(크고 작음)를 판별할 수 있음을 나타내는 프로토콜입니다.
- **특징**: `Equatable` 프로토콜을 상속받습니다. 다른 언어에서는 대소 비교를 위해 `compareTo()` 같은 메서드를 따로 쓰거나 모든 비교 연산자(>, <, >=, <=)를 일일이 오버로딩해야 하지만, Swift에서는 단 하나의 연산자인 **`<` (보다 작음) 연산자**만 오버라이딩하면 나머지 연산자들(>, <=, >=)은 컴파일러가 수학적 논리에 따라 자동으로 완성해 줍니다.
- **예시**:
  ```swift
  struct Student: Comparable {
      let name: String
      let score: Int
      
      // < 연산자 하나만 구현해도 >, <=, >= 가 전부 자동 합성됩니다.
      static func < (lhs: Student, rhs: Student) -> Bool {
          return lhs.score < rhs.score
      }
  }
  
  let minsu = Student(name: "민수", score: 80)
  let yong = Student(name: "영희", score: 95)
  print(minsu < yong)  // true
  print(minsu >= yong) // false (자동 구현)
  ```

---

## 2. 주요 함수 (Functions)

### `stride(from:to:by:)` / `stride(from:through:by:)`
- **개념**: 특정 범위 안에서 일정한 간격으로 값을 건너뛰며 순회할 수 있는 시퀀스를 만들어 주는 함수입니다.
- **특징**: Swift의 `for-in` 문에는 전통적인 C스타일의 `for(int i=0; i<10; i+=2)` 문법이 없기 때문에, 일정한 증가/감소폭을 다룰 때 이 함수가 그 대안으로 적극 사용됩니다.
  - `to`: 종점 직전까지 범위 제공 (미만)
  - `through`: 종점을 포함하여 범위 제공 (이하)
- **예시**:
  ```swift
  // 1부터 9까지 2씩 증가: 1, 3, 5, 7, 9
  for i in stride(from: 1, through: 10, by: 2) {
      print(i)
  }
  ```

### `dump(_:)`
- **개념**: 인스턴스의 자세한 내부 상태 및 구조를 트리 형태로 표준 출력하는 디버깅용 내장 함수입니다.
- **특징**: `print()`가 객체의 간단한 텍스트 묘사(Description)만 출력한다면, `dump()`는 리플렉션을 통해 내부 프로퍼티 구조와 참조 여부까지 샅샅이 분해하여 구조적으로 보여줍니다.
- **예시**:
  ```swift
  let user = User(name: "원준")
  dump(user)
  /* 출력 결과:
   - User
     - id: UUID
       - uuid: (16 bytes)
     - name: "원준"
  */
  ```

### `zip(_:_:)`
- **개념**: 두 개의 시퀀스(배열 등)를 하나로 묶어 튜플의 시퀀스로 결합해 주는 함수입니다.
- **특징**: 두 컬렉션의 인덱스를 일치시켜 돌릴 때 C스타일로 카운터 변수를 쓰지 않고, 함수형 패러다임에 어울리게 묶어서 순회할 수 있도록 돕습니다.
- **예시**:
  ```swift
  let names = ["철수", "영희"]
  let scores = [90, 85]
  for (name, score) in zip(names, scores) {
      print("\(name): \(score)점")
  }
  ```

---

## 3. 예약어 (Keywords)

### `inout`
- **개념**: 함수 내부에서 외부로부터 전달된 매개변수(인자)의 값을 직접 수정하여 호출한 원본에 반영되도록 하는 키워드입니다.
- **특징**: C언어의 포인터 전달이나 C++의 참조자(`&`)처럼 값 타입(구조체)의 원본을 변경하고 싶을 때 사용합니다. 함수 호출 시 인자 앞에 `&` 기호를 붙여 전달해야 합니다.
- **예시**:
  ```swift
  func swapValues(_ a: inout Int, _ b: inout Int) {
      let temp = a
      a = b
      b = temp
  }
  var x = 1, y = 2
  swapValues(&x, &y) // x = 2, y = 1
  ```

### `defer`
- **개념**: 코드 블록을 빠져나갈 때(return, throw 등 어떤 경로든 상관없이) 가장 마지막에 반드시 실행하도록 보장하는 구문을 정의하는 키워드입니다.
- **특징**: Java의 `finally`와 비슷하지만, 예외 처리 구문 외에 일반 함수나 로컬 블록 등 어디서나 사용할 수 있습니다. 파일 닫기나 자원 해제 코드를 위쪽에 선언해 두어 누수를 예방할 수 있습니다.
- **예시**:
  ```swift
  func processFile() {
      let file = openFile()
      defer { closeFile(file) } // 함수가 끝나기 직전에 무조건 실행 보장
      
      // 파일 작업 수행... (중간에 에러가 나거나 리턴되어도 안전)
  }
  ```

### `mutating`
- **개념**: 구조체(Struct)나 열거형(Enum) 같은 값 타입 내의 메서드 중에서, 내부 프로퍼티 값을 수정할 것임을 명시하는 키워드입니다.
- **특징**: 클래스(참조 타입)에서는 필요 없지만, 값 타입은 기본적으로 인스턴스 메서드 내에서 자기 내부를 변경할 수 없도록 격리되어 있기 때문에 변경이 필요한 경우 이 키워드를 선언해 주어야 컴파일이 가능해집니다.
- **예시**:
  ```swift
  struct Point {
      var x = 0.0, y = 0.0
      mutating func moveBy(x deltaX: Double, y deltaY: Double) {
          x += deltaX // mutating이 없으면 컴파일 에러 발생!
          y += deltaY
      }
  }
  ```

### `fallthrough`
- **개념**: Swift의 `switch` 구문에서 매칭된 case의 블록이 끝난 뒤, 하위 case의 코드로 실행을 강제 전환하는 키워드입니다.
- **특징**: C/Java는 `break`를 명시하지 않으면 자동으로 다음 case로 흘러가지만, Swift는 `break`가 암시적으로 내장되어 있어 조건이 맞으면 빠져나갑니다. 만약 C언어처럼 흘려보내고 싶다면 `fallthrough`를 선언해야 합니다.
- **예시**:
  ```swift
  let num = 3
  switch num {
  case 3:
      print("3입니다.")
      fallthrough // 다음 case 4의 블록도 실행됨
  case 4:
      print("그리고 4입니다.")
  default:
      break
  }
  ```

### `associatedtype`
- **개념**: 프로토콜을 정의할 때, 구체적인 타입을 지정하지 않고 프로토콜을 채택하는 타입에 의해 결정되도록 남겨두는 **연관 타입** 지정 키워드입니다.
- **특징**: 프로토콜에서는 일반 클래스처럼 `<T>`와 같은 제네릭 문법을 직접 사용하기 어렵기 때문에 `associatedtype`을 제공하여 템플릿 역할을 수행합니다.
- **예시**:
  ```swift
  protocol Container {
      associatedtype Item
      mutating func append(_ item: Item)
  }
  ```

---

## 4. 디버깅 식별자, 컴파일러 제어구문, 사용 가능 조건 확인

### 디버깅 식별자 (`#file`, `#function`, `#line` 등)
- **개념**: 전처리기가 아닌 Swift 컴파일러가 빌드 과정에서 해당 코드가 위치한 파일 경로, 함수명, 줄 번호 등으로 직접 치환해 주는 지시어입니다.
- **특징**: 디버깅 로그 모듈을 자체 제작할 때 매우 요긴하게 활용됩니다.
- **예시**:
  ```swift
  func logMessage(_ message: String, file: String = #file, function: String = #function, line: Int = #line) {
      print("[\(file):\(line)] \(function) - \(message)")
  }
  ```

### 사용 가능 조건 확인 (`#available` & `@available`)
- **개념**: 실행 환경의 OS 버전 정보를 실시간으로 판단하여 특정 버전 이상의 API만 실행하도록 보장하는 컴파일 제어식입니다.
- **특징**: 다른 언어는 런타임에 시스템 버전을 알아내는 API를 호출해 직접 `if`로 걸러야 하지만, Swift에서는 이를 문법 레벨에서 강제합니다. 새로운 OS 전용 API를 사용하려면 반드시 컴파일 타임에 이 분기 처리를 해두지 않으면 빌드가 되지 않습니다.
- **예시**:
  ```swift
  // 함수 수준에 적용
  @available(iOS 15.0, *)
  func useNewFeature() {
      // iOS 15 이상에서만 사용 가능한 API들
  }

  // 코드 내부에서 동적 분기
  if #available(iOS 15.0, *) {
      useNewFeature()
  } else {
      // 구버전 대체 코드
  }
  ```

---

## 5. 속성 (Attributes)

속성은 컴파일러에게 선언부(타입, 함수 등)에 대한 부가 정보를 전달하는 어노테이션 같은 개념입니다.

### `@discardableResult`
- **개념**: 함수의 반환값(Return Value)을 사용하는 쪽에서 무시하고 넘어가더라도 컴파일러가 경고를 내지 않도록 지시하는 속성입니다.
- **특징**: Swift 컴파일러는 기본적으로 반환값을 받아서 처리하지 않는 호출부가 있다면 경고(Unused Result)를 발생시킵니다. 작업 성공 여부 등을 반환하지만 굳이 매번 검사하지 않아도 되는 함수에 유용하게 부착합니다.
- **예시**:
  ```swift
  @discardableResult
  func logToDatabase(message: String) -> Bool {
      // 로깅을 하고 성공 여부를 반환
      return true
  }
  
  logToDatabase(message: "시스템 에러 발생") // 반환값을 안 써도 경고가 뜨지 않음!
  ```

### `@dynamicMemberLookup`
- **개념**: 타입 내부에 정의되지 않은 멤버(프로퍼티) 이름으로 접근을 요청하더라도, 런타임에 딕셔너리처럼 해당 키 값을 동적으로 조율하도록 허용해 주는 속성입니다.
- **특징**: 파이썬이나 자바스크립트 같은 동적 타입 언어와의 상호 운용성(Interoperability)을 높이기 위해 추가된 매우 이색적인 기능입니다. 서브스크립트 구현을 필요로 합니다.
- **예시**:
  ```swift
  @dynamicMemberLookup
  struct DynamicPerson {
      var info: [String: String]
      
      subscript(dynamicMember member: String) -> String {
          return info[member] ?? "정보 없음"
      }
  }

  let person = DynamicPerson(info: ["age": "25", "city": "Seoul"])
  print(person.age) // 실제 구조체엔 age 프로퍼티가 없지만 "25" 출력!
  ```

---

### 마무리

다른 언어에서 Swift로 넘어올 때 헷갈리기 쉬운 대표적인 핵심 요소들을 정리해 보았습니다.

기본적으로 C계열의 패밀리 문법을 따르고 있는 것처럼 보이지만, **값 타입의 메모리 안정성을 보장하기 위한 제약(`mutating`, `inout`)**, **강력한 템플릿 생태계를 구성하는 컴파일러 지시문(`associatedtype`, `@available`)**, 그리고 **예외 보장 지연 실행(`defer`)** 등 Swift를 한층 더 모던하고 안전한 프로그래밍 언어로 돋보이게 만드는 고유한 특성들이 숨겨져 있습니다.

이 Appendix를 토대로 프로젝트의 코드를 다시 살펴보신다면, 그동안 이해가 잘 가지 않았던 독특한 Swift 코드의 의도를 완벽하게 파악하실 수 있을 것입니다! 🚀
