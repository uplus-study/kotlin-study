# 클래스, 객체, 인터페이스

### 코틀린 인터페이스
- 추상 메서드, 구현 있는 메서드(자바8의 디폴트 메서드) 정의 가능
- 상태(필드) X
- 자바의 `extends`, `implements` -> `:`
- 자바의 `@Override` (선택) -> `override` 변경자 (필수)
- 자바의 디폴트 메서드 -> 키워드 추가 X, 메서드 본문 추가하면 됨!
- 두 메서드를 아우르는 구현을 하위 클래스에 직접 구현하게 강제함, 하나만 해도 OK
    ```Kotlin
    class Button: Clickable, Focusable {
        override fun click() = println("I was clicked")
        override fun showOff() {
            super<Clickable>.showOff()
            super<Focusable>.showOff()
        }
    }
    ```

### open, final, abstract (상속 제어 변경자)
- 자바에서는 final 키워드를 통해 명시적으로 상속 금지함 => 기본적으로 상속 가능
  - 취약한 기반 클래스
  : 하위 클래스가 기반 클래스에 대해 가졌던 가정이 기반 클래스를 변경함으로써 깨져서 발생
- 코틀린에서는 클래스와 메서드 기본적으로 final
  - 상속 허용하려면 `open` 변경자 추가 필요
  - final이 없는 override 메서드나 프로퍼티는 기본적으로 열려 있음
  - 스마트 캐스트 가능 => val + 커스텀 접근자 없는 경우 => 프로퍼티가 final
- 클래스 abstract 선언 가능
  - 구현이 없는 추상 멤버만 있기 때문에 하위 클래스는 항상 오버라이드 해야 함
  - 항상 열려있기 때문에 open 명시 필요 X

|변경자|변경자가 붙은 멤버는|설명|
|---|---|---|
|final|오버라이드 O|클래스 멤버의 기본 변경자|
|open|오버라이드 X|반드시 open 명시해야 오버라이드 O|
|abstract|반드시 오버라이드|추상 클래스의 멤버에만 사용, 구현 있으면 X|
|override|상위 클래스나 인스턴스 멤버 오버라이드 중|오버라이드하는 멤버는 열려있음, 하위 클래스의 오버라이드 금지하려면 final 명시 필요|

### 가시성 변경자 
- 코드 기반에 있는 선언에 대해 클래스 외부 접근 제어
- public, protected, private(자바와 같음)
- 코틀린 기본 가시성: public
- 자바 기본 가시성인 패키지 전용은 없음
  - 코틀린에서는 패키지를 namespace 관리로만 사용하기 때문
- 추가로 internal 도입
  - 모듈 내부에서만 볼 수 있음
  - 모듈: 한 번에 한꺼번에 컴파일되는 코틀린 파일들
- 코틀린에서는 최상위 선언에 대해 private허용
  - 클래스, 함수, 프로퍼티 등
  - 그 선언이 들어있는 파일 내부에서만 사용
  - 하쉬 시스템의 자세한 구현 사항을 외부에 감추고 싶을 때 유용

|변경자|클래스 멤버|최상위 선언|
|---|---|---|
|public(기본)|모든 곳에서 볼 수 있음|모든 곳에서 볼 수 있음|
|internal|같은 모듈 내에서만|같은 모듈 안에서만|
|protected|하위 클래스 안에서만|최상위 선언에 적용 X|
|private|같은 클래스 안에서만|같은 파일 안에서만|

<br>

- 클래스의 기반 타입 목록에 들어있는 타입이나 제네릭 클래스의 타입 파라미터에 들어있는 타입 가시성은 클래스 자신의 가시성과 같거나 높아야하고, 메서드 시그니처에 사용된 모든 타입의 가시성은 메서드의 가시성과 같거나 더 높아야 함
  - 함수 호출이나 확장 시 필요한 모든 타입에 접근할 수 있게 보장함

> 코틀린 가시성 변경자와 자바
>
> public, protected, private 변경자는 자바 바이트코드에서도 유지
> private 클래스는 자바에서 불가능하므로 패키지 전용 클래스로 컴파일
> internal은 자바에서 딱 맞는게 없어서 public으로 변환
- 코틀린에서는 외부 클래스가 내부 클래스나 중첩된 클래스의 private 멤버에 접근 X


### 내부 클래스와 중첩된 클래스
- 중첩 클래스 사용 가능
- 코틀린에서는 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한 없음
  - 자바 static 중첩 클래스와 같음 
  - 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하고 싶다면 inner 변경자 사용
  - 내부 클래스에서 바깥쪽 클래스 참조에 접근하려면 `this@Outer` 사용

### 봉인된(sealed) 클래스 
- 상위 클래스에 sealed 를 붙이면 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있음
- sealed 클래스의 하위 클래스 정의할 때 반드시 상위 클래스 안에 중첩시켜야 함
  - when 절에서 디폴트 분기(else) 필요 없어짐
- 자동으로 open
- private 생성자 가짐 
  - 내부에서만 호출 가능
- sealed 인터페이스를 정의할 수 없음
  - 자바 쪽에서 구현하지 못하게 막을 수 있는 방법이 없음

### 클래스 초기화
- 코틀린은 주 생성자와 부 생성자 구분
- 주 생성자는 생성자 파라미터 지정하고 생성자 파라미터에 의해 초기화되는 프로퍼티 정의하는 목적에 쓰임
- `constructor` 주 생성자나 부 생성자 정의 시작할 때 사용, 생략 가능
- `init` 초기화 블록 시작
  - 클래스 객체 만들어질 때 실행될 초기화 코드 넣음
  - 주 생성자와 함께 사용
  ```Kotlin
  class User constructor(_nickname: String) {
    val nickname: String
    init {
        nickname = _nickname
    }
  }
  ```
- 주 생성자의 파라미터로 프로퍼티 초기화한다면 프로퍼티 정의와 초기화 간략히 쓸 수 있음
    ```Kotlin
    class User(val nickname: String)
    ```

- 인스턴스 생성시 new 키워드 없이 생성자 직접 호출
    ```Kotlin
    val harim = User("하림")
    ```
- 별도 생성자 정의하지 않으면 아무 인자 없는 디폴트 생성자 만들어줌
  - 상속한 하위 클래스에서는 기반 클래스의 생성자를 호출해야 함
  - 기반 클래스 이름 뒤에 빈 괄호 필수
  - 인터페이스 뒤에는 괄호 X

### 부 생성자
- 코틀린에서는 생성자가 여러 개 있는 경우 적음
- 하지만 여러 개 필요한 상황이 존재하긴 함
  - 프레임워크 클래스 확장할 때 여러 가지 방법으로 인스턴스 초기화할 수 있게 지원이 필요한 경우
- 부 생성자는 constructor 키워드로 시작
- this()로 다른 생성자 호출 가능

### 인터페이스에 선언된 프로퍼티 구현
- 인터페이스에 추상 프로퍼티 선언 넣을 수 있음
  ```Kotlin
  interface User {
    val nickname: String
  }
  ```
- 인터페이스에는 아무 상태도 포함할 수 없기 때문에 상태 저장해야 한다면 인터페이스를 구현한 하위 클래스에서 상태 저장을 위한 프로퍼티 등을 만들어야 함
- 주 생성자 안에 프로퍼티 직접 선언
  ```Kotlin
  class PrivateUser(override val nickname: String): User
  ```
  - 추상 프로퍼티 구현 -> override 표시
- 커스텀 getter로 프로퍼티 설정
  ```Kotlin
  class SubscribingUser(val email: String): User {
    override val nickname: String
      get() = email.substringBefore('@')
  }
  ```
  - 뒷받침하는 필드에 값을 저장하지 않고 매번 이메일 주소에서 별명 계산
- 초기화 식으로 프로퍼티 설정
    ```Kotlin
    class FacebookUser(val accountId: Int): User {
      override val nickname = getFacebookName(accountId)
    }
    ```
    - 함수 호출을 통해서 프로퍼티 값 초기화
    - 계산한 데이터를 뒷받침하는 필드에 저장했다가 불러옴
- 게터와 세터 있는 프로퍼티 선언 가능
  - 뒷받침하는 필드 참조 X
  - 커스텀 게터가 있다면 오버라이드하지 않고 상속 가능
  
### 게터와 세터에서 뒷받침하는 필드에 접근
- 값을 저장하되, 값을 변경하거나 읽을 때마다 정해진 로직 실행하는 프로퍼티 생성
- 접근자 안에서 프로퍼티를 뒷받침하는 필드에 접근해야 함
- 변경 가능 프로퍼티의 게터, 세터 중 한쪽만 직접 정의해도 됨

### 접근자 가시성 변경
- get, set 앞 가시성 변경자 추가할 수 있음

### 모든 클래스가 정의해야 하는 메서드
```Kotlin
class Client(val name: String, val postalCode: Int)
```

- `toString()`
  
  인스턴스 문자열 표현
  디버깅, 로깅 시 사용
  ```Kotlin
  override fun toString() = "Cleint(name = $name, postalCode = $postalCode)"
  ```

- `equals()` 
  
  객체 동등성 연산을 수행하기 위해서 오버라이드
  ```Kotlin
  override fun equals(other: Any?): Boolean {
    if(other == null || other !is Client) 
      return false
    return name == other.name && postalCode == other.postalCode
  }
  ```

- `hashCode()` 
  
  자바에서 equals 오버라이드할 때 hashCode도 반드시 오버라이드 해야 함
  > JVM 언어에서는 "equals가 true를 반환하는 두 객체는 반드시 같은 hashCode를 반환해야 한다"라는 제약이 있음
  ```Kotlin
  override fun hashCode(): Int = name.hashCode() * 31 + postalCode
  ```

### 데이터 클래스
- 클래스가 데이터를 저장하는 역할을 수행한다면 위의 3개 메서드 오버라이드 필요
- 코틀린의 데이터 클래스는 위의 메서드를 컴파일러가 자동으로 생성함
- equals, hashCode는 주 생성자에 나열된 모든 프로퍼티 대상으로 생성
- 불변성
  - 프로퍼티가 val일 필요는 없지만 데이터 클래스를 불변 클래스로 만들기를 권장함
  - `copy()` 메서드를 통해 객체 복사하면서 일부 프로퍼티 바꿀 수 있음
  ```Kotlin
  fun copy(name: String = this.name, 
      postalCode: Int = this.postalCode) = 
    Client(name, postalCode)
  ```

### 클래스 위임
- 코틀린에서는 클래스 기본으로 final -> open 으로 열 수 있음
- 상속을 허용하지 않는 클래스에 새로운 동작 추가해야 할 때 있음 
  > 데코레이터 패턴 : 상속 허용하지 않는 클래스 대신 사용할 수 있는 클래스를 만들고 기존 클래스와 같은 인터페이스를 데코레이터가 제공하게 하고 기존 클래스를 데코레이터 내부에 필드로 유지
  >
  > 단점: 준비 코드 많이 필요함

- 코틀린에서는 위임을 일급 시민 기능으로 지원 
- `by` 키워드를 통해 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실 명시

### object 키워드
- object 사용
  - `클래스 정의하면서 동시에 인스턴스 생성`
  - 객체 선언
    - 싱글턴 패턴 지원
    - 클래스를 정의하고 클래스의 인스턴스를 만들어서 변수에 저장하는 모든 작업을 한 문장으로 처리
    - 생성자는 쓸 수 없음
    ```Kotlin
    object Payroll {
      val allEmployees = arrayListOf<Person>()
      fun calculateSalary() {
        for(person in allEmployees) {
          ...
        }
      }
    }
    ```
    - 클래스나 인터페이스 상속 가능
    - 클래스 안에서 객체 선언 가능(인스턴스 1개)
  - 동반 객체
    - 코틀린은 static 키워드를 지원하지 않고 패키지 수준의 최상위 함수와 객체 선언 활용
    - but 최상위 함수는 클래스의 private 멤버 접근 X
    - 클래스의 인스턴스와 관계없이 호출해야 하는데, 내부 정보에 접근해야 하는 함수가 필요할 때 클래스에 중ㅈ첩된 객체 선언의 멤버 함수로 정의 필요
    - companion 을 붙이면 동반 객체로 만들 수 있음
    - 동반 객체는 클래스에 정의된 일반 객체이므로 이름 붙이거나 인터페이스 상속하거나 확장 함수나 프로퍼티 정의 가능
  - 무명 객체
    - 자바의 무명 내부 클래스 대신
    - 클래스나 인스턴스에 이름 붙이지 X
