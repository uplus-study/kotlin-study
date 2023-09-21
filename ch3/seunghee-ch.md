# Ch3. 함수 정의와 호출

## 1. 코틀린에서 컬렉션 만들기
코틀린이 자체 컬렉션을 제공하지 않음 - 표준 자바 컬렉션을 활용하면 자바 코드와 상호작용하기가 훨씬 더 쉽고 자바와 코틀린 컬렉션을 서로 변환할 필요가 없음.
```kotlin
val set = hashSetOf(1, 7, 53)
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```
## 2. 함수를 호출하기 쉽게 만들기
### 이름 붙인 인자
- 함수에 전달하는 인자 이름을 명시할 수 있다
### 디폴트 파라미터 값

자바: 이름/타입 반복으로 오버로딩 메서드가 너무 많아지는 문제 있음
코틀린: 함수 선언에서 디폴트 파라미터로 디폴트 값 지정. 인자를 생략 가능하고, 인자를 이름을 붙여서 디폴트 값 지정

```kotlin
fun <T> joinToString(
	collection: Collection<T>,
	separator: String = ", ",
	prefix: String = "",
	postfix: String = "@"
): String

joinToString(list, postfix = "%")
```
### 디폴트 값과 자바
코틀린 함수를 자바에서 호출하는 경우 모든 인자를 명시해야 한다. 코틀린 함수를 자바에서 호출하는 경우에는 그 코틀린 함수가 디폴트 파라미터 값을 제공하더라도 모든 인자를 명시해야 함.
자바에서 코틀린 함수를 자주 호출해야 한다면 자바 쪽에서 좀 더 편하게 코틀린 함수를 호출하기 위해 @JvmOverloads 애노테이션을 함수에 추가할 수 있다.
코틀린 컴파일러가 마지막 파라미터부터 파라미터를 하나씩 생략한 오버로딩한 자바 메소드를 자동으로 추가해주며, 함수 시그니처별로 생략된 파라미터들은 코틀린 함수의 디폴트 파라미터로 대체된다.

### 정적인 유틸 클래스 없애기: 최상위 함수와 최상위 프로퍼티
코틀린에서는 함수나 프로퍼티를 클래스 안에 선언하지 않고 소스파일의 최상위 수준에 배치할 수 있음. 
최상위 함수 - 무의미한 정적 메소드만 모아두는 클래스 생성 필요 없고 class 없이 패키지의 멤버 함수이므로 외부 패키지에서 사용 시 package만 import하면 됨. (@JvmName 어노테이션을 통해 최상위함수의 클래스명을 별도 지정 가능)
최상위 프로퍼티 - 정적필드에 저장, 공용값/상수, var도 가능하지만 const val이 적당

## 3. 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티
기존 코드와 코틀린 코드를 자연스럽게 통합하는 것은 코틀린의 핵심 목표이며, 기존 자바 API를 재작성하지 않고도 코틀린이 제공하는 추가 기능을 사용하고 싶을 때 확장 함수가 그런 역할을 해줄 수 있음.
개념적으로 확장 함수는 확장 함수는 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수다.
- 수신 객체 타입: 그 함수가 확장할 클래스의 이름 (`String`)
- 수신 객체: 확장함수가 호출되는 대상이 되는 값 (`this`)
[String 타입 확장]
```kotlin
package strings
fun String.lastChar(): Char = this.get(this.length - 1)
```
[List<String>, List<Owner> 타입 확장]
```kotlin
class EmailValidator {
companion object {
internal val regex = Regex("[A-Za-z0-9]+@[A-Za-z]+\\.[A-Za-z]+")
}
}

fun List<String>.filterValidEmails(): List<String> {
return this.filter { email -> email.isEmailFormat() }
}
fun String.isEmailFormat(): Boolean {
return EmailValidator.regex.matches(this)
}
fun List<Owner>.filterValidEmailOwners(): List<Owner> {
    return this.filter { owner -> owner.email?.let { it.isEmailFormat() } ?: false }
}

//사용
var list =
    emailService.getOwnerWithEmailList(provider).takeIf { it.isNotEmpty() } ?: return
var list =
    emailService.getOwnerWithEmailList(provider).filterValidEmailOwners().takeIf { it.isNotEmpty() } ?: return
```

[유틸 클래스이므로 최상위 프로퍼티로 변경하면]
```kotlin
package validator;
const val = regex = Regex("[A-Za-z0-9]+@[A-Za-z]+\\.[A-Za-z]+")

fun List<String>.filterValidEmails(): List<String> {
return this.filter { email -> email.isEmailFormat() }
}
fun String.isEmailFormat(): Boolean {
return EmailValidator.regex.matches(this)
}

=> 코드 구조: regex를 전역 상수화. 확장/유지 어려울 수 있음
=> 가독성: 클래스를 사용하는 것이 코드를 더 구조화 하고 모듈화 할 수 있고, EmailValidator 클래스에 관련 기능을 캡슐화 하고 있어 더 명확하고 읽기 쉬움
=> 확장성: 추후 다른 유효성 검사 또는 관련 기능 추가할 경우 클래스 내에 로직 추가가 용이
```

### 임포트와 확장 함수

확장 함수를 사용하기 위해서는 그 함수를 다른 클래스나 함수와 마찬가지로 임포트 해야 함.
확장 함수를 임포트 없이 사용한다면 동일한 이름의 확장 함수와 충돌할 수도 있기 때문에 임포트로 어떤 확장함수인지 명시해 주어야 한다.

```kotlin
import strings.lastChar // 명시적으로 사용
import strings.* // * 사용 가능
import strings.lastChar as last // as 키워드를 사용 하여 이름 충돌 방지
```
### 자바에서 확장 함수 호출
자바에서 확장 함수를 사용하려면 정적 메소드를 호출하면서 첫 번째 인자로 수신 객체를 넘기기만 하면 됨. 
다른 최상위 함수와 마찬가지로 확장 함수가 들어있는 자바 클래스 이름도 확장 함수가 들어있는 파일 이름에 따라 결정되며 확장 함수를 StringUtil.kt 파일에 정의했다면 다음과 같이 호출할 수 있다.
```kotlin
char c = StringUtilKt.lastChar("java");
```

### 확장 함수로 유틸리티 함수 정의
joinToString 함수를 Collection<T>에 대한 확장함수를 선언하는 것으로 변경. 코틀린 라이브러리가 제공하는 함수와 거의 유사
```kotlin
[before]
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ";",
    prefix: String = "(",
    postfix: String = ")"
): String {

    val result = StringBuilder(prefix)

    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}

fun main(args: Array<String>) {
    val list = listOf(1, 2, 3)
    println(joinToString(list, "; ", "(", ")"))
    println(joinToString(collection = list, separator = ";", prefix = "(", postfix = ")"))
    println(joinToString(list))
    println(joinToString(list, "; "))
}

[after]
fun <T> Collection<T>.joinToString(
separator: String = ", ",
prefix: String = "",
postfix: String = ""
): String {
val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}

fun main(args: Array<String>) {
val list = arrayListOf(1, 2, 3)
println(list.joinToString(" "))
}

```

### 확장 함수 정리
- 확장 함수는 단지 정적 메소드 호출에 대한 문법적인 편의일 뿐 일반 클래스 타입이 아닌 더 구체적인 타입을 수신 객체 타입으로 지정할 수도 있다.
- 확장 함수는 오버라이드할 수 없다.
- 확장 함수는 클래스의 일부가 아니다.
- 확장 함수는 클래스 밖에 선언된다.
- 이름과 파라미터가 완전히 같은 확장 함수를 기반 클래스와 하위 클래스에 대해 정의해도 실제로는 확장 함수를 호출할 때 
- 수신 객체로 지정한 변수의 컴파일 시점의 정적 타입에 의해 어떤 확장함수가 호출될지가 결정되지, 그 변수에 저장된 객체의 동적인 타입에 의해 확장 함수가 결정되지 않는다.
``` kotlin
fun View.shoOff() = println("I'm a view!")
fun Button.shoOff() = println("I'm a button!")

val view: View = Button()
view.shoOff() 
=> I'm a view! 출력
```
- 어떤 클래스를 확장한 함수와 그 클래스의 멤버 함수의 이름과 시그니처가 같다면 확장 함수가 아니라 멤버 함수가 호출된다(멤버 함수의 우선순위가 더 높다). 
- 클래스의 API를 변경할 경우 항상 이를 염두에 둬야 한다.

### 확장 프로퍼티
확장 프로퍼티를 사용하면 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가할 수 있다. 
일반 프로퍼티와 달리 상태를 저장할 적절한 방법이 없기 때문에 실제로 확장 프로퍼티는 아무 상태도 가질 수 없다.
``` kotlin
val String.lastChar: Char
get() = get(length -1)
```
backing filed (프로퍼티의 값을 저장하는 필드)가 없어서 기본 게터 구현을 제공할 수 없으므로 최소한 게터는 꼭 정의를 해야함.
마찬가지로 초기화 코드에서 계산한 값을 담을 장소가 전혀 없으므로 초기화 코드도 쓸 수 없음
``` kotlin
var StringBuilder.lastChar: Char
get() = get(length - 1)
set(value: Char) {
this.setCharAt(length - 1, value)
}

fun main(args: Array<String>) {
println("Kotlin".lastChar)
val sb = StringBuilder("Kotlin?")
sb.lastChar = '!'
println(sb)
}
```

## 4. 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

### 자바 컬렉션 API 확장
어떻게 자바 라이브러리 클래스의 인서턴스인 컬렉션에 코틀린이 새로운 기능을 추가할수 있었을까? 확장함수!!
코틀린 컬렉션은 자바와 같은 클래스를 사용하지만 더 확장된 API를 제공함.
IDE 잘 이용

### 가변 인자 함수: 인자의 개수가 달라질 수 있는 함수 정의
[vararg]
가변 길이 인자는 메소드를 호출할 때 원하는 개수만큼 값을 인자로 넘기면 자바 컴파일러가 배열에 그 값들을 넣어주는 기능임. 
코틀린의 가변 길이 인자도 자바와 비슷하지만 문법이 조금 다름. 
타입 뒤에 ...를 붙이는 대신 코틀린에서는 파라미터 앞에 vararg 변경자를 붙임
``` kotlin
public fun <T> listOf(vararg elements: T): List<T> = if (elements.size > 0) elements.asList() else emptyList()

fun main(args: Array<String>) {
val list = listOf("one", "two", "eight")
}
```

[스프레드 연산자 *]
이미 배열에 들어있는 원소를 가변 길이 인자로 넘길 때도 코틀린과 자바 구문이 다름.
자바에서는 배열을 그냥 넘기면 되지만 코틀린에서는 배열을 명시적으로 풀어서 배열의 각 원소가 인자로 전달되게 해야 함.
fun main(args: Array<String>) {
val list = listOf("args: ", *args)
}

### 값의 쌍 다루기: 중위 호출과 구조 분해 선언
[중위 호출]
중위 호출 시에는 수신 객체와 유일한 메소드 인자 사이에 메소드 이름을 넣음 (이때 객체/메소드 이름, 유일한 인자 사이에 공백 필수)
``` kotlin
[예시]
맵을 만들기 위한 mapOf 함수
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")

1.to("one") // "to" 메소드를 일반적인 방식으로 호출함
1 to "one" // "to" 메소드를 중위 호출 방식으로 호출함
```

함수(메소드)를 중위 호출에 사용하게 허용하고 싶으면 infix 변경자를 함수(메소드) 선언 앞에 추가해야 함.
인자가 하나뿐인 일반 메소드나 확장함수에서 사용 가능
```kotlin
infix fun Any.to(other: Any) = Pair(this, other)
```

[구조분해 선언]
```kotlin
/**
* Creates a tuple of type [Pair] from this and [that].
*
* This can be useful for creating [Map] literals with less noise, for example:
* @sample samples.collections.Maps.Instantiation.mapFromPairs
  */
  public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)

=> 제너릭 함수인 to의 간략화
  public infix fun Any.to(other: Any) = Pair(this, other)

```

=> Pair의 내용으로 두 변수를 즉시 초기화 할 수 있음
```kotlin
val (number, name) = 1 to "one"
```
  이런 기능을 구조 분해 선언이라고 부르며 다른 객체도 구조 분해를 적용 가능
```kotlin
  for ((index, element) in collection.withIndex()) {
  println("$index: $element")
  }
```

## 5. 문자열과 정규식 다루기
코틀린은 자바 문자열의 혼동을 야기할 수 있는 일부 메소드에 대해 추가로 다양한 확장 함수를 제공함
### 문자열 나누기

자바: 정규식을 split의 구분 문자열로 사용한다 -> 덜 명확

코틀린: 정규식 이외에도 여러 구분 문자열이나 문자 등 여러 조합의 파라미터를 받는 확장 함수 제공 -> 굿굿
```kotlin
"12.345-6.A".split("\\.|-".toRegex())) // 정규식 파라미터
"12.345-6.A".split(".", "-")) // 여러 구분 문자열 파라미터
"12.345-6.A".split('.', '-')) // 여러 구분 문자 파라미터

=> [12, 345, 6, A]
```

### 정규식과 3중 따옴표로 묶은 문자열
[String을 확장한 함수를 사용]
코틀린은 정규식과 일반 문자열을 처리할 때 유용한 다양한 문자열 처리 함수를 제공한다
```kotlin
//구분 문자열이 맨 나중/처음, 앞/뒤 반환 함수 존재
fun parsePath(path: String) {
val directory = path.substringBeforeLast("/")
val fullName = path.substringAfterLast("/")
val fileName = fullName.substringBeforeLast(".")
val extension = fullName.substringAfterLast(".")

println("Dir: $directory, name: $fileName, ext: $extension")
}

fun main(args: Array<String>) {
parsePath("/Users/yole/kotlin-book/chapter.adoc")
}

// result
// Dir: /Users/yole/kotlin-book, name: chapter, ext: adoc
```

[정규식과 삼중따옴표]
path에서 처음부터 마지막 슬래시 직전까지의 부분 문자열은 파일이 들어있는 디렉터리 경로.
path에서 마지막 마침표 다음부터 끝까지의 부분 문자열은 파일 확장자.
코틀린에서는 정규식을 사용하지 않고도 문자열을 쉽게 파싱할 수 있다.
정규식은 강력하기는 하지만 나중에 알아보기 힘든 경우가 많다. 정규식이 필요할 때는 코틀린 라이브러리를 사용하면 더 편하다.
자바 문자열로 표현하려면 수많은 이스케이프가 필요한 문자열의 경우 3중 따옴표 문자열을 사용하면 더 깔끔하게 표현할 수 있다.
```kotlin
fun parsePath(path: String) {
val regex = """(.+)/(.+)\\.(.+)""".toRegex() //삼중따옴표
val matchResult = regex.matchEntire(path)
if (matchResult != null) {
val (directory, filename, extension) = matchResult.destructured
println("Dir: $directory, name: $filename, ext: $extension")
}
}
```
- 3중 따옴표 문자열에서는 역슬래시(\)를 포함한 어떤 문자도 이스케이프 할 필요가 없다. 
- 정규식을 인자로 받은 path에 매치시킨다. 
- 매치에 성공하면 그룹별로 분해한 매치 결과를 의미하는 destructured 프로퍼티를 각 변수에 대입한다. 
- 이때 사용한 구조 분해 선언은 Pair로 두 변수를 초기화할 때 썼던 구문과 같다.
- 줄 바꿈을 표현하는 아무 문자열이 그대로 대입됨

## 6. 코드 다듬기: 로컬 함수와 확장
좋은 코드의 중요한 원칙 - 반복하지 말라(DRY: Don't Repeat Yourself), 중복이 없게 하라
메소드 추출 리팩토링을 적용해서 긴 메소드를 부분부분 나눠서 각 부분을 재활용할 수 있음 -> but 클래스 안에 작은 메소드가 많아지고 각 메소드 사이의 관계를 파악하기 힘들어서 코드를 이해하기 더 어려워질 수도 있음
-> 리팩토링을 진행해서 추출한 메소드를 별도의 내부 클래스안에 넣으면 코드를 깔끔하게 조직할 수는 있지만, 그에 따른 불필요한 준비 코드가 늘어남

코틀린에서는 함수에서 추출한 함수를 원 함수 내부에 중첩시킬 수 있다. 굿굿!! 그렇게 하면 문법적인 부가 비용을 들이지 않고도 깔끔하게 코드를 조직할 수 있다.
```kotlin
fun saveUser(user: User) {
if (user.name.isEmpty()) {
throw IllegalArgumentException(
"Can't save user ${user.id}: empty Name")
}

    if (user.address.isEmpty()) {
        throw IllegalArgumentException(
            "Can't save user ${user.id}: empty Address")
    }

    // Save user to the database
}
```
로컬 함수를 사용하여 name/address 검증 부분 충복 개선해보면,

```kotlin
fun saveUser(user: User) {

    fun validate(user: User,
                 value: String,
                 fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: empty $fieldName")
        }
    }

    validate(user, user.name, "Name")
    validate(user, user.address, "Address")

    // Save user to the database
}
```
검증 로직 중복은 사라졌고, 필요하면 User의 다른 필드에 대한 검증도 쉽게 추가할 수 있음
하지만 로컬 함수는 자신이 속한 바깥 함수의 모든 파라미터와 변수를 사용할 수 있으므로 User 객체를 로컬 함수에게 하나하나 전달해야 하지 않고 불필요한 User 파라미터를 없앨 수 있음
```kotlin
fun saveUser(user: User) {
fun validate(value: String, fieldName: String) { // user 파라미터를 중복 사용하지 않는다.
if (value.isEmpty()) {
throw IllegalArgumentException(
"Can't save user ${user.id}: " + // 바깥 함수의 파라미터에 직접 접근할 수 있다.
"empty $fieldName")
}
}

    validate(user.name, "Name")
    validate(user.address, "Address")

    // Save user to the database
}
```
더 개선하려면 validate함수를 User 클래스에 대한 확장한 함수로 만들 수도 있음
```kotlin
fun User.validateBeforeSave() {
fun validate(value: String, fieldName: String) {
if (value.isEmpty()) {
throw IllegalArgumentException(
"Can't save user $id: empty $fieldName")
}
}

    validate(name, "Name")
    validate(address, "Address")
}

fun saveUser(user: User) {
user.validateBeforeSave()

    // Save user to the database
}
```

