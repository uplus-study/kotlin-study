# Ch3. 함수 정의와 호출

# 1. 코틀린에서 컬렉션 만들기

코틀린은 자신만의 컬렉션 기능을 제공하지 않는다. 기존 자바 컬렉션을 활용할 수 있다.

코틀린이 자체 컬렉션을 제공하지 않는 이유는 표준 자바 컬렉션을 활용하면 자바 코드와 상호작용하기 더 쉽기 때문이다.

# 2. 함수를 호출하기 쉽게 만들기

## 2.1 이름 붙인 인자

코틀린에서는 작성한 함수를 호출할 때는 함수에 전달하는 인자 중 일부의 이름을 명시할 수 있다.
호출시 인자 중 어느 하나라도 이름을 명시하고 나면 혼동을 막기 위해 그 뒤에 오는 모든 인자는 이름을 꼭 명시해야 한다.

자바로 작성한 코드가 호출할 때는 이름붙인 인자를 사용할 수 없다.

```kotlin
joinToString(collection, separator= "", prefix= " ", postfix= ".")
```

## 2.2 디폴트 파라미터 값

코틀린에서는 함수 선언에서 파라미터의 디폴트 값을 지정할 수 있으므로 오버로드를 피할 수 있다.

```kotlin
fun <T> joinToString(
	collections: Collection<T>,
	separator: String = "",
	prefix: String = "",
	postfix: String = ""
): String
```

일반 호출문법을 사용하려면 함수를 선언할 때와 같은 순서로 인자를 지정해야한다. 그런 경우에 일부를 생략하면 뒷부분의 인자들이 생략된다.

이름붙인 인자를 사용하는 경우에는 인자 목록의 중간에 있는 인자를 생략하고 지정하고 싶은 인자를 이름붙여 순서와 관계없이 지정할 수 있다.

> 자바에서 코틀린 디폴트 파라미터를 사용하더라도 직접 파라미터를 명시해야한다. 
자바에서 코틀린 함수를 자주 사용해야한다면 함수 선언한 곳에 @JvmOverloads 를 사용하면 
코틀린 컴파일러가 자동으로 맨 마지막 파라미터로 부터 파라미터를 하나씩 생략한 오버로딩한 자바 메소드를 추가해준다.
각각의 오버로딩한 함수들은 시그니처에서 생략된 파라미터에 코틀린 함수의 디폴트 파라미터를 사용한다.
> 

## 2.3 정적인 유틸 클래스 없애기: 최상위 함수와 프로퍼티

자바와 달리 정적 메소드만 들어있는 무의미한 클래스를 만들지 않아도 된다.

```kotlin
// join.kt
package strings

fun joinToString(...).: String { ... }

// 컴파일 한 후 동일한 클래스
package strings

public class JoinKt {
	public static String joinToString(...).: String { ... }
}

```

이 함수가 어떻게 실행될 수 있는 걸까? JVM이 클래스 안에 들어있는 코드만을 실행할 수 있기 때문에 컴파일러는 이 파일을 컴파일할 때 새로운 클래스를 정의해준다.

코틀린 컴파일러가 생성하는 클래스의 이름은 최상위 함수가 들어있는 코틀린 소스 파일의 이름과 대응한다.

코틀린 파일의 모든 최상위 함수는 이 클래스의 정적인 메소드가 된다.

> 파일에 대응하는 클래스 이름 변경하려면 파일에 @JvmName 애노테이션을 추가하라.
@JvmName 애노테이션은 파일의 맨 앞. 패키지 이름 선언 이전에 위치해야한다.
> 

### 최상위 프로퍼티

함수와 마찬가지로 프로퍼티도 파일의 최상위 수준에 놓을 수 있다. 이런 프로퍼티 값은 정적 필드에 저장된다.

const 변경자를 추가하면 프로퍼티를 public static final 필드로 컴파일하게 만들 수 있다.

# 3. 메소드를 다른 클래스에 추가: 확장함수와 프로퍼티

**확장함수:** 어떤 클래스의 멤버 메소드인 것 처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수이다.

```kotlin
package strings
 
   //수신객채타입                수신객체   수신객체
fun String.lastChar(): Char = this.get(this.length - 1)

== 

fun String.lastChar(): Char = get(length - 1)
```

확장 함수를 만들려면 추가하려는 함수 이름 앞에 그 함수가 확장할 클래스의 이름을 덧붙이기만 하면 된다.
클래스 이름을 수신 객체 타입 (receiver type) 이라 부르며, 확장 함수가 호출되는 대상이 되는 값을 수신 객체(receiver object)라고 부른다.

확장함수는 다른 JVM 언어로 작성된 클래스도 확장할 수 있다.

## 3.1 임포트와 확장 함수

확장함수를 프로젝트 다른 소스코드에서 사용하려면 그 함수를 임포트 해야한다.

물론 *를 사용해서 임포트해서 사용할 수 있고, as 키워드를 이용하면 다른 이름으로 부를 수 있다.

```kotlin
import strings.lastChar

val c = "Kotlin".lastChar()

----------------------------

import strings.*

val c = "Kotlin".lastChar()

----------------------------

import strings.lastChar as last

val c = "Kotlin".last()
```

## 3.2 자바에서 확장 함수 호출

내부적으로 확장 함수는 수신 객체를 첫번째 인자를 받는 정적 메소드다.
또한 다른 최상위 함수와 마찬가지로 확장 함수가 들어있는 자바 클래스 이름도 확장 함수가 들어있는 파일 명에 따라 결정된다. 

```java
//만약 확장함수를 StringUtil.kt에 작성했다면
char c = StringUtilKt.lastChar("Java")
```

## 3.3 확장 함수로 유틸리티 함수 정의

## 3.4 확장 함수는 오버라이드 할 수 없다.

확장 함수는 클래스의 일부가 아니다. 확장 함수는 클래스 밖에 선언된다. 

이름과 파라미터 같은 완전히 같은 확장 함수를 기반 클래스와 하위 클래스에 대해 정의해도 실제로 확장 함수를 호출할 때 수신 객체로 지정한 변수의 정적 타입에 의해 어떤 확장 함수가 호출 될지 결정된다.

```kotlin
class View{}
class Button: View() {}

fun View.showOff() = println("I'm a view")
fun Button.showOff() = println("I'm a button")

>>> val view: View = Buttton()
>>> view.showOff()
I'm a view
```

> 어떤 클래스를 확장한 함수와 클래스의 멤버 함수의 이름과 시그니처가 같다면 항상 멤버 함수가 호출된다.(멤버 함수의 우선순위가 더 높다.)
> 

## 3.5 확장 프로퍼티

확장 프로퍼티를 사용하면 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있다

확장 함수와 마찬가지로 확장 프로퍼티도 일반적인 프로퍼티와 같은데, 단지 수신 객체 클래스가 추가 됐을뿐이다.

최소한 게터는 꼭 정의해야한다.

# 4. 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

## 4.1 자바 컬렉션 API 확장

자바 라이브러리 클래스의 인스턴스인 컬렉션에 대해 코틀린이 새로운 기능을 추가할 수 있었던 이유는 확장 함수를 통해 했던 것이다.

## 4.2 가변 인자 함수: 인자의 개수가 달라질 수 있는 함수 정의

코틀린의 가변길이 인자는 타입 뒤에 …을 붙이는 대신 파라미터 앞에 `vararg` 변경자를 붙인다.

이미 배열에 들어있는 원소를 가변 길이 인자로 넘길 때도 코틀린은 배열을 명시적으로 풀어서 배열의 각 원소가 인자로 전달되게 해야 한다. 기술적으로 `스프레드 연산자`가 그런 작업을 해준다

```kotlin
fun main(args: Array<String> {
	val list = listof("args:" , *args) // 스프레드 연산자가 배열을 펼쳐준다.
	println(list)
}
```

## 4.3 값의 쌍 다루기: 중위 호출과 구조 분해 선언

`중위 호출(infix call)`

인자가 하나뿐인 일반 메소드나 인자가 하나뿐인 확장 함수에 중위 호출을 사용할 수 있다. 메소드에 중위 호출을 사용하게 허용하고 싶으면 `infix`키워드를 함수 선언 앞에 추가해야 한다. 

```kotlin
1.to("one") 
1 to "one"

// to 함수의 정의를 간략하게 줄인 코드
infix fun Any.to(other: Any) = Pair(this. other) //Pair는 코틀린 표준 라이브러리 함수
```

Pair의 내용으로 두 변수를 즉시 초기화할 수 있다.

```kotlin
val (number, name) = 1 to "one"
```

이걸 `구조 분해 선언(destructing declaration)` 이라고 한다.

```kotlin
// mapOf의 구현
public fun <K, V> mapOf(vararg pairs: Pair<K, V>): Map<K, V> =
    if (pairs.size > 0) pairs.toMap(LinkedHashMap(mapCapacity(pairs.size))) else emptyMap()
```

# 5. 문자열과 정규식 다루기

## 5.1 문자열 나누기

코틀린에서는 자바의 split 대신 여러 가지 다른 조합의 파라미터를 받는 split 확장 함수를 제공함으로써 혼동을 야기하는 메소드를 감춘다. 정규식 파라미터로 받는 함수는 String이 아닌 Regex 타입의 값을 받는다.

split 확장함수를 오버로딩한 버전 중에는 구분 문자열을 하나 이상 인자로 받는 함수가 있다.

## 5.2 정규식과 3중 따옴포로 묶은 문자열

3중 따옴표 문자열에서는 역슬래시(\)를 포함한 어떤 문자도 이스케이프할 피룡가 없다. 예를 들어  일반 문자열을 사용해 정규식을 작성하는 경우 마침표 기호를 이스케이프하려면 \\.라고 써야하지만, 3중 따옴표 문자열에서는 \.라고 쓰면 된다.

## 5.3 여러 줄 3중 따옴표 문자열

3중 따옴표 문자열을 문자열 이스케이프를 피하기 위해서만 사용하지는 않는다. 3중 따옴표 문자열에는 줄 바꿈을 표현하는 아무 문자열이나 이스케이프 없이 그냥 들어간다.

여러 줄 문자열에는 들여쓰기나 줄 바꿈을 포함한 모든 문자가 들어간다.

# 6. 코드 다듬기

로컬함수를 이용해서 함수 내부에 함수를 중첩시켜서 중복을 제거할 수 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
	fun validate(valeu: String, fieldName: String) {
		if(value.isEmpty()) {
			throw IllegalArgumentException("Can't save user ${user.id}: empty $fieldName")
		}	
	}

	validate(user.name, "Name")
	validate(user.address, "Address")
}
```

코드를 더 개선시키고 싶으면 확장함수를 이용할 수도 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)

fun validateBeforeSave(user: User) {
	fun validate(valeu: String, fieldName: String) {
		if(value.isEmpty()) {
			throw IllegalArgumentException("Can't save user ${user.id}: empty $fieldName")
		}	
	}

	validate(user.name, "Name")
	validate(user.address, "Address")
}

fun saveUser(user: User) {
	user.validateBeforeSave()
}
```

그러나 중첩이 많아지면 코드릴 읽기 어려워진다. 따라서 일반적으로 한 단계만 중첩시키길 권장한다.