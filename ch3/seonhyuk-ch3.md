# 3. 함수 정의와 호출

## 3.1 Collection

```kotlin
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")

println(map.javaClass) // class java.util.HashMap
println(list.javaClass) // class java.util.ArrayList
```

코틀린은 자체 컬렉션 기능을 제공하지 않고 기존 자바 컬렉션을 활용할 수 있다.
표준 자바 컬렉션을 활용하여 자바 코드와 상호작용하기가 더 쉽다.

원래 자바보다 더 많은 메서드를 사용할 수 있다 ~~(원래 자바 모름)~~

## 3.2 함수 호출 쉽게 하기

### 3.2.1 이름 붙인 인자

### 3.2.2 default parameter

- **swift랑 똑같다**

```kotlin
// 선언
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    result.append("\n")
    return result.toString()
}

// 호출
joinToString(list, separator = ";", prefix = "(", postfix = ")")
```

default parameter가 있으면 호출 부분에서 파라미터를 빼도 되며,
이름만 명시해준다면 parameter의 순서와 상관없이 동작한다
(자바에서 호출한다면 default parameter를 명시하더라도 모든 인자를 명시해야 한다.)

### 3.2.3 정적인 유틸리티 클래스 없애기: 최상위 함수 & 프로퍼티

```kotlin
package strings

public class JoinKt {
  public static String joinToString(...) { ... }
}
```

위에 처럼 클래스 내부에 메서드를 선언하지 않아도 코틀린 소스 파일의 이름에 대응하는 클래스를 만든다.

```kotlin
@file:JvmName("StringFunctions")
// annotation을 명시해주면 자바에서 해당 이름의 클래스로 호출할 수 있다.
```

#### 최상위 프로퍼티

```kotlin
const val UNIX_LINE_SEPARATOR = "\n"

// java
public static final String UNIX_LINE_SEPATATOR = "\n";
```

## 3.3 메서드를 다른 클래스에 추가

```kotlin
fun String.lastChar(): Char = this[this.length - 1]
fun String.lastChar(): Char = get(length - 1)
```

String class에 lastChar 메서드를 추가
this 생략이 가능하다

### 3.3.1 임포트와 확장함수

확장함수도 import를 해야 사용 가능하다.

### 3.3.3 확장 함수로 유틸리티 함수 정의

위에서의 `joinToString` 함수를 다음과 같이 바꿀 수 있다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    result.append("\n")
    return result.toString()
}
```

### 3.3.4 확장 함수는 오버라이드할 수 없다.

확장 함수는 클래스의 일부가 아니다. 확장 함수는 클래스 밖에 선언된다.

```kotlin
open class View {
    open fun click() = println("View clicked")
}

class Button: View() {
    override fun click() = println("Button clicked")
}
```

class를 override해서 쓸 땐, Button이 override한 메서드가 실행되지만,

```kotlin
fun View.showOff() = println("View")
fun Button.showOff() = println("Button")
```

확장 함수를 사용하면 정적 타입에 의해 결정된 View의 메서드가 실행된다.

### 3.3.5 확장 프로퍼티

기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있다.
기존 클래스의 인스턴스 객체에 필드를 추가할 방법은 없으므로 아무 상태도 가질 수 없지만 더 짧게 코드 작성이 가능하다.

기본 게터 구현을 제공할 수 없으므로, 최소한 게터는 꼭 정의를 해야 한다.

## 3.4 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

- `varang` 키워드를 사용하면 호출 시 인자 개수가 달라질 수 있는 함수를 정의할 수 있다
- 중위 함수호출 구문을 사용하면 인자가 하나뿐인 메서드를 간편하게 호출할 수 있다
- 구조 분해 선언을 사용하면 복합적인 값을 분해해서 여러 변수에 나눠 담을 수 있다

### 3.4.2 가변 인자 함수

```kotlin
val list = listOf(2, 3, 5, 7, 11)

fun listOf<T>(varang values: T): List<T> { ... }
```

**스프레드 연산자**

```kotlin
val list = listOf("args: ", *args)
```

### 3.4.3 값의 쌍 다루기, 중위 호출과 구조 분해 선언

중위 호출 시에는 수신 객체와 유일한 메서드 인자 사이에 메서드 이름을 넣는다

```kotlin
1.to("one")
1 to "one"
```

인자가 하나뿐인 일반 메서드나 인자가 하나뿐인 확장 함수에 중위 호출을 사용할 수 있다.

함수 선언 시 앞에 `infix` 변경자를 추가해야 한다.

```kotlin
infix fun Any.to(other: Any) = Pair(this.oter)
```

**구조 분해 선언**

```kotlin
val (number, name) = 1 to "one"
```

## 3.5 문자열과 정규식 다루기

### 3.5.1 문자열 나누기

### 3.5.2 정규식과 3중 따옴표로 묶은 문자열

코틀린은 정규식을 사용하지 않고도 문자열을 쉽게 파싱할 수 있다.

### 3.5.3 여러 줄 3중 따옴표 문자열

## 3.6 코드 다듬기

코틀린에서는 함수에서 추출한 함수를 원 함수 내부에 중첩시킬 수 있다.

```kotlin
fun saveUser(user: User) {
    if (user.name.isEmpty()) {
        throw IllegalArgumentException("Can't save user ${user.id}: empty Name")
    }

    if (user.address.isEmpty()) {
        throw IllegalArgumentException("Can't save user ${user.id}: empty Address")
    }
}

// 함수 내부에 validate 함수 선언

fun saveUser(user: User) {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("Can't save user ${user.id}: empty $fieldName")
        }
    }
    validate(user.name, "Name")
    validate(user.address, "Address")
}
```

검증 함수를 클래스의 확장 함수로 만들 수 있다.

```kotlin
fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("Can't save user $id: empty $fieldName")
        }
    }

    validate(name, "Name")
    validate(address, "Address")
}

fun saveUser(user: User) {
    user.validateBeforeSave()
}
```
