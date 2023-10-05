# 6장. 코틀린 타입 시스템

## 널 가능성

### 널이 될 수 있는 타입
- 코틀린에서는 NPE를 피할 수 있도록 널이 될 수 있는 타입과 될 수 없는 타입을 명시적으로 지원
- NPE를 런타임에서 컴파일 시점으로 옮김.
- `?` 연산자
  - 널이 될 수 있음.
  - 타입 이름 뒤에 물음표 붙임.

### 안전한 호출 연산자 `?.`
- null 검사와 메서드 호출을 연산 한번으로 가능
- 호출하는 값이 널이 아니면 일반 메서드를 호출하는 것처럼 작동하고 호출하는 값이 널이면 호출하지 않고 널로 결과값이 반환됨.

```Kotlin
class Employee(val name: String, val manager: Employee?)

val mname = employee.manager?.name
val country = company?.address?.country
```

### 엘비스 연산자 `?:`
- 좌항 값이 널이 아니면 좌항 값을 결과값으로 하고, 좌항 값이 널이면 우항 값을 결곽값으로 함.

```Kotlin
fun Person.countryName(): String {
  return this.company?.address?.country ?: "Unknown"
}
val addr = person.company?.address ?: throw IllegalArgumentException("No address")
```

### 안전한 캐트 연산자 `as?`
- 지정한 타입으로 캐스트를 수행하는데 해당 타입으로 변환할 수 없으면 NULL

```Kotlin
fun equals(o: Any?): Boolean {
  val other: Person = o as? Person ?: return false
  return oterPerson.firstName && oterPerson.lastName == lastName
}
```

### 널 아님 단언: `!!`
- 어떤 값이던지 널이 될 수 없는 타입으로 변경.
- 널인 경우 NPE 발생

```Kotlin
fun ignoreNulls(s: String?) {
  val notNull: String = s!! // s가 null이면 NPE
}
```

### let 함수
- 자신의 수신 객체를 파라미터로 전달 받은 람다에 넘김
- 널이 될 수 있는 값을 널이 아닌 값만 파라미터로 받는 함수에 넘길 때 사용
- `?.`를 함께 사용

```Kotlin
fun sendEmailTo(email: String){
  println("Sending Email to ${email}")
}

// let을 통해 null이 아니면 특정 람다 실행 (not null)
var email: String? = "neity16@daum.net"
email?.let{ sendEmailTo(it) }

// null로 아무 작동 일어나지 않음.
email = null
email?.let{ sendEmailTo(it) } 
```

### 나중에 초기화할 프로퍼티

#### `lateinit`
- 널이 될 수 없는 프로퍼티를 생성 시점 말고 나중에 초기화 함.
- 프로퍼티를 나중에 초기화하지 않고 사용할 경우 오류 발생
- 프로퍼티는 `var`여야 함

```Kotlin
class MyService{
  fun performAction() : String = "foo"
}

class MyTest{
  private lateinit var myService: MyService

  // 생성 후에 프로퍼티를 초기화! 
  @Before fun setUp(){
    myService = MyService()
  }

  @Test fun testAction(){
    // lateinit으로 선언하고 @Before를 통해 이후에 초기화하여 에러가 발생하지 않음.
    Assert.assertEquals("foo", myService.performAction())
  }
}
```

### 널 가능 타입의 확장
- 널 가능 타입을 확장하면 널이 될 수 있는 값에 대해 그 확장 함수를 호출할 수 있음
- 확장 함수 내부에서 널 검사 진행

```Kotlin
val str: String?
str.isNullOrBlank() 
```

### 타입 파라미터의 널 가능성
- 함수, 클래스 모든 타입 파라미터가 널이 될 수 있음.
- 널이 아님을 명시하려면 타입 상한을 잘 지정해야 함.

```Kotlin
// 타입 상한 지정하여 널 방지
fun <T: Any> printHashCode(t: T){
  println(t.hashCode()) // ?. 없어도 호출
}
```

### 널 가능성과 자바
- Java @Nullable String -> Kotlin String?
- Java @NotNull String -> Kotlin String

#### 플랫폼 타입
코틀린이 널 관련된 정보를 알 수 없는 자바 타입
- 널 가능 타입/널 불가능 타입 모두 가능
- 널 가능성을 코틀린에 맞게 처리해야 함.

```Kotlin
interface StringProcessor {
  void process(String value);
}

class StringPrinter : StringProcessor {
  override fun process(value: String) { // 널 불가능 타입으로 선언
    println(value)
  }
}

class NullableStringPrinter : StringProcessor {
  override fun process(value: String?) { // 널 가능 타입으로 선언
    value.?let { println(it) }
  }
}
```

## 코틀린의 원시 타입

### 원시타입
- 실행 시점에 가장 효율적인 방식으로 표현
- 대부분 코틀린 숫자 타입 -> 자바 원시 타입으로 컴파일
- 널 가능 원시 타입은 래퍼 타입으로 컴파일
  - 코틀린 Int? -> 자바 Integer
  - 자바 원시 타입은 널 참조를 가질 수 없기 때문
- 지네릭 클래스는 래퍼 타입으로 컴파일
  - JVM은 타입인자로 원시 타입을 허용하지 않기 때문

### 숫자 변환
- 숫자 타입 변환은 명시적으로 해야 변환됨.
- 숫자 리터럴은 컴파일러가 필요한 변환을 자동으로 처리함.
- 연산자를 각 타입에 대해 오버로딩함.


### 최상위 타입
`Any`
- 원시 타입을 포함한 모든 널 불가 타입의 조상 타입
- 코틀린 `Any`는 자바 `Object`와 유사함. 하만 `Object`는 참조 타입만 포함
`Any?`
- 널 포함하는 모든 값을 대입할 수 있는 변수 타입

### Unit 타입
- 자바의 void와 유사
- 아무것도 반환하지 않는 함수의 반환 타입으로 사용 가능
- void와 달리 타입 파라미터로도 사용 가능

### Nothing 타입
- 값을 반환하지 않음.
- 함수의 반환 타입, 타입 파라미터로만 사용(ex. 예외)

## 컬렉션과 배열

### 컬렉션
- 컬렉션에서 어떤 것이 널인지 명확하게 이해 필요
```Kotlin
List<Int?>
List<Int>?
List<Int?>?
```
- filterNotNull
  - 널이 될 수 있는 값을 가지는 컬렉션에서 null이 아닌 값들만 걸러줌.

#### 읽기 전용과 변경 가능한 컬렉션
- `Collection`: 조회 기능만
- `MutableCollection`: 수정 기능까지

### 코틀린 컬렉션과 자바
- 모든 코틀린의 컬렉션은 그에 맞는 자바 컬렉션 인터페이스의 인스턴스
- 실제 내부에서는 자바 구현체를 사용
![0_Lpp0--c1ymMIsQXv](https://github.com/uplus-study/kotlin-study/assets/68267278/11c79c9a-a9e3-4ca1-8208-cebf4dc229d8)

#### 생성함수
- List
  - 조회 전용: listOf
  - 변경 가능 타입: mutableListOf, arrayListOf
- Set
  - 조회 전용: setOf
  - 변경 가능 타입: mutableSetOf, hashSetOf, linkedSetOf, sortedSetOf
- Map
  - 조회 전용: mapOf
  - 변경 가능 타입: mutableMapOf, hashMapOf, linkedMapOf, sortedMapOf

### 컬렉션을 플랫폼 타입으로 다루기
- 자바의 컬렉션 타입은 조회 전용, 변경 가능으로 다룰 수  있음.
- 고려 사항
  - 컬렉션이 널이 될 수 있는지
  - 컬렉션 원소가 널이 될 수 있는지
  - 오버라이드하는 메서드가 컬렉션 변경을 할 수 있는지
- 자바 인터페이스나 클래스가 어떠한 맥락에서 사용되는지를 프로그래머가 파악해야 함.

### 배열
```Kotlin
val num: Array = arrayOf﴾1, 2, 3, 4﴿
val nulls = arrayOfNulls﴾10﴿ // Array<String?>
val nulls2 = Array﴾20﴿ { i ‐> ﴾i + 1﴿.toString﴾﴿ }
```
- `toTypedArray()`
  - 컬렉션 -> 배열

- 원시 타입 배열
 - IntArray -> int[]
 - ByteArray -> byte[]
 - CharArray -> char[]
 - BooleanArray -> boolean[]

 - Array 같은 타입은 `toIntArray` 등 함수를 통해 IntArray로 변환 가능