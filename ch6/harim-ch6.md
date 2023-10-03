# 코틀린 타입 시스템
코틀린의 타입 시스템은 코드의 가독성 향상시키는 특성을 제공
- 널이 될 수 있는 타입
- 읽기 전용 컬렉션
- 불필요하거나 문제 되던 부분 제거 ex) 배열 지원
  

### 널 가능성 nullability
- NPE 피할 수 있게 돕는 특성
- null에 대한 접근 방법은 실행 시점에서 컴파일 시점으로 옮김
```Kotlin
//java -> s에 null이 들어온다면 NPE 발생
int strLen(String s) {
    return s.length();
}

//kotlin -> strLen에 null이거나 널 될 수 있는 인자 넘기는게 금지됨 -> 컴파일 오류
fun strLen(S: String) = s.length
```
- `?` 를 붙여 타입의 변수나 프로퍼티에 null 참조 저장할 수 있음 -> 널이 될 수 있는 타입
    ```Kotlin
    fun strLesnSafe(s: String?) = ...
    ```
  - 모든 타입은 기본적으로 널이 될 수 없는 타입   
  - 널이 될 수 있는 타입인 변수에 대해 `변수.메서드()` 처럼 메서드 호출 X
  - 널이 될 수 있는 값을 널이 될 수 없는 타입의 변수에 대입 X
  - null 과 비교하여 null이 아님이 확실한 영역에서 해당 값을 널이 될 수 없는 타입의 값처럼 사용 가능
    ```Kotlin
    fun strLenSafe(s: String?): Int = 
        if (s != null) s.length else 0
    ```

### 타입의 의미
> 타입은 분류로 ... 타입은 어떤 값들이 가능한지와 그 타입에 대해 수행할 수 있는 연산의 종류를 결정한다

- 자바에서
  - double: 64비트 부동소수점 수
    - 일반 수학 연산 수행 가능
  - String: 문자열이나 null 들어갈 수 있음
    - 두 종류의 값에 대해 실행할 수 있는 연산 다름
  - => 자바의 타입 시스템이 널을 제대로 다루지 못함
    - 널 여부 추가로 검사하기 전에는 변수에 대해 어떤 연산 수행할 수 있는지 알 수 없음
- 코틀린의 **널이 될 수 있는 타입**은 자바에서의 문제를 해결함
  - 각 타입의 값에 대해 어떤 연산이 가능할지 명확히 이해 가능
  - 실행 시점에 예외 발생시킬 수 있는 연산 판단 가능

### 널이 될 수 있는 값 다루는 연산자
- `?.`
  - 안전한 호출 연산자
  - null 검사와 메서드 호출을 한 번의 연산으로 수행
    `s?.toUpperCase()` 와 `if(s != null) s.toUpperCase() else null` 과 같음
  - 호출하려는 값이 null이면 호출이 무시되고 null이 결과가 됨
  - 객체 그래프에서 널이 될 수 있는 중간 객체가 여럿 있다면 한 식 안에서 안전한 호출 연쇄해서 함께 사용 가능
    ```Kotlin
    class Address(val streetAddress: String, val zipCode Int, val city: String, val country: String)
    class Company(val name: String, val address: Address?)
    class Person(val name: String, val company: Company?)

    fun Person.countryName(): String {
        val country = this.company?.address?.country
        return if (country != null) country else "Unknown"
    }
    ```

- 엘비스 연산자 `?:`
  - null 대신 사용할 디폴트 값 지정할 때 사용
    ```Kotlin
    fun foo(s: String?) {
        val t: String = s ?: ""
    }
    ```
  - 이항 연산자로 좌항을 계산한 값이 널인지를 검사함 
  - 좌항 값이 널이 아니라면 좌항 값을 결과로 하고, 널이라면 우항 값을 결과로 함
    ```Kotlin
    fun Person.countryName() = company?.address?.country ?: "Unknown"
    ```
  - 우항에 return, throw 등의 연산 넣을 수 있음
    - 함수의 전제 조건 검사하는 경우 유용
  
- `as?`
  - 어떤 값을 지정한 타입으로 캐스트함
  - 값을 대상 타입으로 변환할 수 없으면 null 반환
```Kotlin
class Person(val firstName: String, val lastName: String) {
    override fun equals(o: Any?): Boolean {
        val otherPerson = o as? Person ?: return false

        return otherPerson.firstName == firstName && 
            otherPerson.lastName == lastName
    }
} 
```

- 널 아님 단언 `!!`
  - 어떤 값이든 널이 될 수 없는 타입으로 바꿀 수 있음
  - 실제 널에 대해 !!를 적용하면 NPE 발생
  - 어떤 함수가 값이 널인지 검사한 다음에 다른 함수 호출한다고 해도 컴파일러는 호출된 함수 안에서 그 값을 사용할 수 있음을 인식할 수 없음, 굳이 널 검사를 다시 수행하고 싶지 않을 때 사용
  - !!를 널에 대해 사용해서 발생하는 예외의 스택 트레이스에는 어떤 식에서 예외가 발생했는지에 대한 정보가 없음 => **여러 !! 단언문 한 줄에 함께 쓰는 일은 피하라!**

- `let` 함수
  - 널이 될 수 있는 값을 널이 아닌 값만 인자로 받는 함수에 넘길 때 사용
  - 자신의 수신 객체를 인자로 전달받은 람다에게 넘김
  ```Kotlin
  fun sendEmailTo(email: String) { ... }

  val email: String? = ...
  sendEmailTo(email) //불가능!

  if(email != null) sendEmailTo(email) //널인지 검사 필요

  email?.let { email -> sendEmailTo(email) } //let 사용
  email?.let { sendEmailTo(it) } //람다 식에서 it 사용
  ```
- 나중에 초기화(late-initialized)할 프로퍼티
  - 객체 인스턴스를 일단 생성한 후 나중에 초기화하는 경우가 많음
  - 코틀린에서는 클래스 안의 널이 될 수 없는 프로퍼티를 생성자 안에서 초기화하지 않고 특별한 메서드 안에서 초기화할 수 없음
  - 널이 될 수 없는 타입이라면 반드시 널이 아닌 값으로 프로퍼티 초기화 필요
  - 이럴 때 `lateinit` 변경자 사용하여 프로퍼티를 나중에 초기화
    - 항상 `var` 여야 함

### 널이 될 수 있는 타입 확장
- 널이 될 수 있는 타입에 대한 확장 함수 정의하면 null 값 다루는 강력한 도구로 활용 가능
  - 어떤 메서드를 호출하기 전 수신 객체 역할 하는 변수가 널이 될 수 없다고 보장하는 대신, 직접 변수에 대해 메서드를 호출해도 확장 함수인 메서드가 알아서 널 처리 가능
  - String? 타입의 수신 객체에 대해서 `isNullOrEmpty`, `isNullOrBlank`가 있음
    ```Kotlin
    fun String?.isNullOrBlack(): Boolean = 
        this == null || this.isBlank()
    ```
- 널이 될 수 있는 타입에 대한 확장 정의하면 널이 될 수 있는 값에 대해 그 확장 함수 호출 가능
  - 그 함수 내부에서 this는 널이 될 수 있음 => 명시적으로 널 여부 검사 필요

### 타입 파라미터의 널 가능성
- 타입 파라미터는 기본적으로 널이 될 수 있음
  - ? 가 없더라도 널이 될 수 있는 타입임
- 타입 파라미터가 널이 아님을 확실히 하려면 널이 될 수 없는 타입 상한 지정해야 함
    ```Kotlin
    fun <T: Any> printHashCode(t: T) { //T가 널이 될 수 없는 타입임
        println(t.hashCode())
    }
    ```

### 널 가능성과 자바
- 자바 코드에도 애노테이션으로 표시된 널 가능성 정보가 있음
  - `@Nullable String` => `String?` 
  - `@NotNull String` => `String`
- 코틀린은 여러 널 가능성 애노테이션을 알아봄
  - JSR-305 표준, 안드로이드, 젯브레인스 도구들이 지원하는 애노테이션
  - 이런 널 가능성 애노테이션이 소스코드에 없는 경우 자바의 타입은 코틀린의 플랫폼 타입이 됨
- 플랫폼 타입
  - 코틀린이 널 관련 정보를 알 수 없는 타입
  - 널이 될 수 있는 타입으로 처리해도 되고, 널이 될 수 없는 타입으로 처리해도 됨
  - 플랫폼 타입에 대해 수행하는 모든 연산 책임은 개발자에게 있음!
  - 코틀린 컴파일러는 public 가시성인 코틀린 함수의 널이 아닌 타입인 파라미터와 수신 객체에 대한 널 검사를 추가해줌
  - 코틀린에서 플랫폼 타입 선언 X, 자바에서 가져온 타입만 플랫폼 타입이 됨
- 상속 
  - 코틀린에서 자바 메서드 오버라이드할 때 메서드의 파라미터와 반환 타입을 널이 될 수 있는 타입으로 선언할지 널이 될 수 없는 타입으로 선언할지 결정해야 함
  - 코틀린 컴파일러는 널이 될 수 없는 타입으로 선언한 모든 파라미터에 대해 널이 아님을 검사하는 단언문 만듦
  
### 코틀린의 원시 타입
- 자바에서는 원시 타입과 참조 타입 구분
  - 원시 타입에는 그 값이 직접 들어감
  - 참조 타입은 메모리상의 객체 위치가 들어감
  - 참조 타입이 필요한 경우 래퍼 타입으로 원시 타입 값을 감싸서 사용
- 코틀린에서는 원시 타입과 래퍼 타입 구분 X
  - 원시 타입의 값에 대해 메서드 호출 가능
  - 실행 시점에 숫자 타입은 가능한 한 가장 효율적인 방식으로 표현됨
  - 코틀린의 Int 타입은 자바의 int로 컴파일됨
  - 이런 컴파일이 불가능한 경우 컬렉션과 같은 제네릭 클래스 사용
  - 코틀린의 널이 될 수 있는 원시 타입을 사용하면 자바의 래퍼 타입으로 컴파일됨
  - 어떤 클래스의 타입 인자로 원시 타입을 넘기면 코틀린은 타입에 대한 박스 타입을 사용
    `val listOfInts = listOf(1, 2, 3) // Integer 타입으로 이루어진 리스트`
      - JVM에서 제네릭을 구현하는 방식으로, JVM은 타입 인자로 원시 타입 허용 X

### 숫자 변환
- 코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환 X
  ```Kotlin
  val i = 1
  val l: Long = i // Error!
  ```
- 대신 직접 변환 메서드 호출하면 가능
  ```Kotlin
  val i = 1
  val l: Long = i.toLong()
  ```
- 코틀린은 모든 원시 타입에 대한 변환 함수 제공
- 두 박스 타입 간의 equals 메서드는 그 안에 들어있는 값이 아니라 박스 타입 객체르 ㄹ비교함
- 코드에서 동시에 여러 숫자 타입을 사용하는 경우 예상치 못한 동작을 피하기 위해 각 변수를 명시적으로 변환해야 함
- 산술 연산자는 적당한 타입의 값을 받아들일 수 있게 이미 오버로드되어 있음

### Any 최상위 타입
- 자바에서는 Object가 클래스 계층의 최상위 타입
- 코틀린에서는 Any타입이 모든 널이 될 수 없는 타입의 조상 타입
  - Int 등의 원시 타입을 포함한 모든 타입의 조상 타입
- 내부에서 Any 타입은 java.lang.Object에 대응됨
  - 자바 메서드에서 Object를 인자로 받거나 반환하면 코틀린에서는 Any로 그 타입을 취급함
- 코틀린의 모든 클래스에는 toString, equals, hashCode 메서드가 존재하는데 이 세 메서드는 Any에 정의된 메서드를 상속한 것
  
### Unit 코틀린의 void
- 자바의 void와 같은 기능을 함
- 코틀린 함수의 반환 타입이 Unit이고 그 함수가 제네릭 함수를 오버라이드하지 않는다면, 내부에서 자바 void 함수로 컴파일됨
- Unit은 모든 기능을 갖는 일반적인 타입, void와 달리 Unit을 타입 인자로 사용 가능
- Unit 타입에 속한 값은 단 하나뿐이고 그 이름도 Unit임 => Unit값을 묵시적으로 반환
  ```Kotlin
  interface Processor<T> {
    fun process(): T
  }

  class NoResultProcessor: Processor<Unit> {
    override fun process() {
        //업무 처리 코드, return 명시할 필요 없음 => 컴파일러가 묵시적으로 return Unit 넣어줌
    }
  }
  ```
  - 함수형 프로그래밍에서 전통적으로 Unit은 '단 하나의 인스턴스만 갖는 타입'을 의미해왔음
    => 유일한 인스턴스 유무: 자바 void와 코틀린 Unit을 구분하는 차이

### Nothing 
- 결코 성공적으로 값을 돌려주는 일이 없음 => 반환 값 개념 자체가 의미 없는 함수가 존재함
- Nothing 타입은 아무 값도 포함하지 않음
  - 함수의 반환 타입이나 반환 타입으로 쓰일 타입 파라미터로만 쓸 수 있음
  - Nothing 반환하는 함수를 엘비스 연산자 우항에 사용해서 전제 조건 검사할 수 있음
    ```Kotlin
    fun fail(message: String): Nothing {
        throw IllegalStateException(message)
    }

    val address = company.address ?: fail("No address")
    ```

### 널 가능성과 컬렉션
- 타입 인자로 쓰인 타입에도 ?를 붙여 컬렉션 안에 널 값을 넣을 수 있음
- `List<Int?>` 는 Int? 타입의 값을 저장할 수 있음 => Int나 null 저장 가능
- List<Int?>와 List<Int>? 의 차이
  - List<Int?> 는 리스트 안의 각 값이 null이 될 수 있음
  - List<Int>? 는 전체 리스트가 널이 될 수 있음 
- `filterNotNull` 함수를 통해 컬렉션의 널 값을 걸러낼 수 있음
  
### 읽기 전용과 변경 가능한 컬렉션
- 코틀린에서는 컬렉션 안의 데이터에 접근하는 인터페이스와 컬렉션 안의 데이터를 변경하는 인터페이스를 분리함
- `Collection` 인터페이스를 사용하면 원소를 추가하거나 제거하는 메서드는 X
- `MutableCollection` 인터페이스는 `Collection` 을 확장해서 원소를 추가, 삭제, 원소 모두 지우는 등의 메서드 제공
- 프로그램에서 데이터에 어떤 일이 벌어지는지 더 쉽게 이해하기 위해서 구별
- `Collection` 인터페이스가 읽기 전용이라고 해서 꼭 변경 불가능한 컬렉션일 필요는 없음
  - 읽기 전용 인터페이스 타입인 변수를 사용할 때 그 인터페이스는 실제로는 어떤 컬렉션 인스턴스를 가리키는 수많은 참조 중 하나일 수 있음
  - 읽기 전용 컬렉션이 항상 스레드 안전하지는 않음
  
### 코틀린 컬렉션과 자바
- 모든 코틀린 컬렉션은 그에 상응하는 자바 컬렉션 인터페이스의 인스턴스임
- 코틀린의 읽기 전용과 변경 가능한 인터페이스 기본 구조는 java.util 패키지에 있는 자바 컬렉션 인터페이스의 구조를 그대로 옮겨 놓음
  - 변경 가능한 각 인터페이스는 자신과 대응하는 읽기 전용 인터페이스를 확장(상속)
  - 읽기 전용 인터페이스에는 컬렉션을 변경할 수 있는 모든 요소가 빠져 있음
  
|컬렉션 타입|읽기 전용 타입|변경 가능 타입|
|---|---|---|
|List|listOf|mutableListOf, arrayListOf|
|Set|setOf|mutableSetOf, hashSetOf, linkedSetOf, sortedSetOf|
|Map|mapOf|mutableMapOf, hashMapOf, linkedMapOf, sortedMapOf|

- 자바는 읽기 전용 컬렉션과 변경 가능 컬렉션을 구분하지 않으므로, 코틀린에서 읽기 전용 Collection으로 선언된 객체라도 자바 코드에서는 그 컬렉션 객체의 내용을 변경 O => 주의 필요🚨

### 컬렉션을 플랫폼 타입으로 다루기
- 자바 쪽에서 선언한 컬렉션 타입의 변수를 코틀린에서는 플랫폼 타입으로 봄
  => 프랫폼 타입인 컬렉션은 기본적으로 변경 가능성에 대해 알 수 없음
- 코틀린은 그 타입을 어느 쪽으로든 다룰 수 있음
- 컬렉션 타입이 시그니처에 들어간 자바 메서드 구현을 오버라이드하려는 경우 문제 발생
  - 컬렉션 타입이 널이 될 수 있는지
  - 컬렉션의 원소가 널이 될 수 있는지
  - 오버라이드하는 메서드가 컬렉션 변경할 수 있는지
  
  를 모두 고려하여 반영해야 함

### 객체의 배열과 원시 타입의 배열
- 코틀린 배열은 타입 파라미터를 받는 클래스임
- 배열의 원소 타입은 그 타입 파라미터에 의해 정해짐
- 배열 만드는 방법
  - arrayOf 함수에 원소를 넘김
  - arrayOfNulls 함수에 정수 값을 인자로 넘기면 모든 원소가 null이고 인자로 넘긴 값과 크기가 같은 배열을 만들 수 있음
  - Array 생성자는 배열 크기와 람다를 인자로 받아 람다를 호출해서 각 배열 원소를 초기화함
  ```Kotlin
  val letters = Array<String>(26) { i -> ('a' + i).toString() }
  ```
- 코틀린에서는 배열을 인자로 받는 자바 함수 호출하거나 vararg 파라미터 받는 코틀린 함수 호출하기 위해 자주 배열을 만듦
  - 이때 데이터가 이미 컬렉션에 있다면 배열로 변환 필요
  - `toTypedArray` 메서드를 사용하여 쉽게 배열로 바꿀 수 있음
- 코틀린은 원시 타입의 배열을 표현하는 별도 클래스를 각 원시 타입마다 제공함
  - IntArray, ByteArray, CharArray, ...
  - 각각 int[], byte[], char[] 로 컴파일됨
- 원시 타입의 배열 만드는 방법
  - 각 배열 타입의 생성자는 size 인자를 받아서 해당 원시 타입의 디폴트 값으로 초기화된 size 크기의 배열 반환
  - 팩토리 함수(intArrayOf,..)
  - 크기와 람다를 인자로 받는 생성자 사용
  ```Kotlin
  val fiveZeros = IntArray(5) //1번 방법
  val fiveZerosToo = intArrayOf(0, 0, 0, 0, 0) //2번 방법
  val squares = IntArray(5) { i -> (i+1) * (i+1) } //3번 방법
  ```
- 코틀린 표준 라이브러리는 배열 기본 연산(배열 길이 구하기, 원소 설정하기, 원소 익기) + 컬렉션에서 사용할 수 있는 확장 함수 제공 (filter, map)