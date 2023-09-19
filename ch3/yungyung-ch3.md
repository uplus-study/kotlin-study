# 3장 함수 정의와 호출

## 3.1 코틀린에서 컬렉션 만들기


컬렉션 클래스: `java.util.*` 에 속한다 - 코틀린이 자신만의 컬렉션 기능 제공 X

- 표준 자바 컬렉션 활용: 자바 코드와 상호작용하기 쉽다
- 자바보다 많은 기능 제공

## 3.2 함수를 호출하기 쉽게 만들기

### 이름 붙인 인자

함수에 전달하는 인자 중 일부(또는 전부)의 이름을 명시할 수 있다

### 디폴트 파라미터 값

자바의 오버로딩 메서드가 너무 많아지는 문제: 이름, 타입이 계속 반복

→ 코틀린: 함수 선언에서 파라미터의 디폴트 값 지정 - 인자 일부를 생략 가능

- 인자 목록의 중간에 있는 인자를 생략하고, 지정하고 싶은 인자를 이름을 붙여서 **순서와 관계없이** 지정 가능

```kotlin
fun <T> joinToString(
	collection: Collection<T>,
	separator: String = ", ",
	prefix: String = "",
	postfix: String = ""
): String

joinToString(list, postfix = ";", prefix = "# ")
```

💡 자바에는 디폴트 파라미터 값이 없어서 코틀린 함수를 자바에서 호출하는 경우 모든 인자를 명시해야 한다

`@JvmOverloads` 애노테이션: 코틀린 컴파일러가 자동으로 맨 마지막 파라미터로부터 파라미터를 하나씩 생략한 오버로딩한 자바 메서드를 추가


### 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티

코틀린에서는 함수를 클래스 안에 선언할 필요 X

- 함수를 소스 파일의 최상위 수준에 위치

```kotlin
package strings

fun joinToString(...): String { ... }
```

ex. join.kt 클래스에 최상위 함수로 지정

```java
package strings;

public class JoinKt {
	public static String joinToString(...) {...}
}

JoinKt.joinToString(...);
```

자바 코드: 코틀린 파일의 모든 최상위 함수는 클래스의 정적인 메서드가 된다

💡 파일에 대응하는 클래스의 이름 변경: 파일에 `@JvmName` 애노테이션 추가

최상위 프로퍼티: 정적 필드에 저장, 상수 추가 가능

- `const` 변경자: `public static final` 필드로 컴파일(원시타입 / String만 가능)

## 3.3 메서드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

확장 함수: 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수

```kotlin
package strings
fun String.lastChar(): Char = this.get(this.length-1)

println("Kotlin".lastChar())
```

- 수신 객체 타입: 그 함수가 확장할 클래스의 이름 (`String`)
- 수신 객체: 확장함수가 호출되는 대상이 되는 값 (`this`)

클래스 내부에서만 사용할 수 있는 private/protected 멤버 사용 불가

### 임포트와 확장 함수

클래스 임포트와 동일한 구분 사용

```kotlin
import strings.lastChar
val c = "Kotlin".lastChar()

import strings.lastChar as last
val c = "Kotlin".last()
```

한 파일 안에서 이름이 같은 함수를 가져와 사용하는 경우 `as`를 사용하여 이름 충돌 방지

### 자바에서 확장 함수 호출

확장 함수 = 수신 객체를 첫 번째 인자로 받는 정적 메서드

```java
char c = StringUtilKt.lastChar("Java");
```

### 확장 함수로 유틸리티 함수 정의

더 구체적인 타입을 수신 객체 타입으로 지정 가능

```kotlin
fun Collection<String>.join(...) {}
```

- String 컬렉션에만 정의

### 확장 함수는 오버라이드 할 수 없다

![Untitled](https://essie-cho.com/assets/images/posts/figure3_2.png)

확장 함수는 클래스의 일부가 아니다. 클래스 밖에 선언된다

확장 함수가 저장된 객체의 동적인 타입에 의해 확장 함수가 결정되지 않는다

```kotlin
open class View {}
class Button: View() {}

fun View.shoOff() = println("I'm a view!")
fun Button.shoOff() = println("I'm a button!")

val view: View = Button()
view.shoOff() // I'm a view!
```

확장 함수는 정적으로 결정된다 (컴파일 시점) - `view`의 타입이 `View`이기 때문에 `View`의 확장함수가 호출

```java
View view = new Button();
ExtionsionKt.shoOff(view);
```

### 확장 프로퍼티

기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API 추가 가능

프로퍼티(라는 이름을 가졌)지만, 프로퍼티는 아무 상태도 가질 수 없다

```kotlin
val String.lastChar: Char
	get() = get(length - 1)
```

뒷받침하는 필드가 없기 때문에 **게터는 꼭 정의해야 한다**

## 3.4 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

### 자바 컬렉션 API 확장

코틀린 컬렉션: 자바와 같은 클래스 사용 + 더 확장된 API 제공 = 확장 함수 구현

### 가변 인자 함수: 인자의 개수가 달라질 수 있는 함수 정의

```kotlin
val list = listOf(2, 3, 5, 7, 11)

fun listOf<T>(varag values: T): List<T> {...}
```

가변 길이 인자 = 메서드를 호출할 때 원하는 개수만큼 값을 인자로 넘기면 자바 컴파일러가 배열에 그 값들을 넣어주는 기능

- 파라미터 앞에 `varag` 변경자
- 이미 배열에 있는 원소를 가변 길이 인자로 넘길 때: 스프레드 연산자(`*`) 사용
    - 배열에 들어있는 값과 다른 여러 값을 함께 호출할 수 있음

### 값의 쌍 다루기: 중위 호출과 구조 분해 선언

```kotlin
val map = mapOf(1 to "one", 7 to "seven")
```

- `to`: 일반 함수

중위 호출: 수신 객체와 유일한 메서드 인자 사이에 메서드 이름을 넣는다

```kotlin
1.to("one")
1 to "one" // 중위 호출 방식
```

- 인자가 하나뿐인 일반 메서드/확장 함수에서 사용 가능
- `infix` 변경자 추가

```kotlin
infix fun Any.to(other: Any) = Pair(this, other)
```

구조 분해 선언

```kotlin
val (number, name) = 1 to "one"
```

```kotlin
fun <K, V> mapOf(varag values: Pair<K, V>): Map<K, V>
```

## 3.5 문자열과 정규식 다루기

### 문자열 나누기

자바 split의 구분 문자열: 정규식

코틀린: 여러 조합의 파라미터를 받는 확장 함수 제공

- 정규식 파라미터: `Regex` 타입

```kotlin
"12.345-6.A".split("\\.|-".toRegex())) // 정규식 파라미터
"12.345-6.A".split(".", "-")) // 여러 구분 문자열 파라미터
```

### 정규식과 3중 따옴표로 묶은 문자열

`substringBeforeLast`, `substringAfterLast` 등 구분 문자열이 맨 나중/처음에 나타난 곳 앞/뒤 문자열 반환 함수 존재

3중 따옴표 문자열 - 문자 이스케이프 필요x - 정규식으로 활용

### 여러 줄 3중 따옴표 문자열

3중 따옴표 문자열: 줄 바꿈을 표현하는 아무 문자열이 그대로 들어감

## 3.6 코드 다듬기: 로컬 함수와 확장

함수에서 추출한 함수를 원 함수 내부에 중첩 가능 - 깔끔하게 코드 조직

```kotlin
fun saveUser(user: User) {
	fun validate(value: String, fieldName: String) {
		if (value.isEmpty()) {
			throw IllegalArgumentException("Can't save user ${user.id}: empty $fieldName")
		}
	}
	
	validate(user, user.name, "Name")
	validate(user, user.address, "Address")
}
```

로컬 함수는 자신이 속한 바깥 함수의 모든 파라미터/변수를 사용할 수 있다