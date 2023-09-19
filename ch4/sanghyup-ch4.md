# Ch4. 클래스, 객체, 인터페이스

# 1. 클래스 계층의 정의

## 1.1 코틀린 인터페이스

코틀린 인터페이스는 추상 메소드 뿐 아니라 구현 메소드도 정의할 수 있다.(자바8의 디폴트 메소드)

다만, 인터페이스에는 아무런 상태도 들어갈 수 없다.

자바에서는 extends와 implement 키워드를 사용하짐반, 코틀린에서는 클래스 이름 뒤에 :(콜론)을 붙이고 인터페이스와 클래스 이름을 적는 것으로 클래스 확장과 인터페이스 구현을 모두 처리한다.

자바와 달리 override 변경자를 꼭 사용해야 한다.

자바의 디폴트 메소드와 달리 default 키워드를 붙일 필요 없다.

```kotlin
interface Clickable {
	fun click()
	fun showOff() = println("I'm clickable!")
}

interface Focusable {
	fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus.")
	fun showOff() = println("I'm focusable!")
}

```

만약 위 인터페이스를 모두 구현하는 클래스에서 동일한 이름을 가진 디폴트 메소드를 상속받아 구현하지 않으면 컴파일 오류가 발생하게 된다.

`The class ‘’ must override public open fun showOff() because it inherits many implementation of it.`

> 자바에서는 코틀린의 디폴트 메소드 구현에 의존할수 없다.
> 

## 1.2 open, final, abstact 변경자: 기본적으로 final

코틀른의 클래스와 메소드는 기본적으로 final이다.

어떤 클래스의 상속을 허용하려면 클래스 앞에 open 변경자를 붙여야 한다. 그와 더불어 오버라이드를 허용하고 싶은 메소드나 프로퍼티 앞에도 open 변경자를 붙여야한다.

> 기본적으로 상속을 못하게 하는 이유는 어떤 클래스가 자신을 상속하는 방법에 대해 정확한 규칙을 제공하지 않는다면 그 클래스의 클라이언트는 기반 클래스를 작성한 사람의 의도와 다른 방식으로 메소드를 override 할 위험이 있다.
> 

open 클래스에 특정 메소드를 override 하지 못하게 하려면 final을 명시해야 한다.

자바처럼 코틀린에서도 클래스를 abstract로 선언할 수 있다. abstract로 선언한 추상 클래스는 인스턴스화 할 수 없다.

| 변경자  | 이 변경자가 붙은 멤버는… | 설명 |
| --- | --- | --- |
| final | 오버라이드 할 수 없음 | 클래스 멤버의 기본 변경자다. |
| open | 오버라이드 할 수 있음 | 반드시 open을 명시해야 오버라이드 할 수 있다. |
| abstract | 반드시 오버라이드 해야함 | 추상 클래스의 멤버에만 이 변경자를 붙일 수 있다. 추상멤버에는 구현이 있으면 안된다. |
| override | 상위 클래스나 상위 인스턴스의 멤버를 오버라이드 하는 중 | 오버라이드하는 멤버는 기본적으로 열려있다. 하위클래스의 오버라이드를 금지하려면 final을 명시해야한다.  |

## 1.3 가시성 변경자: 기본적으로 공개

아무 변경자도 없는 경우 선언은 모두 공개된다.

코틀린에는 package-private도 없다.

패키지 전용 가시성에 대안으로는 코틀린에는 internl이라는 새로운 가시성 변경자를 도입했다.

(`internal`은 모듈 내부에서만 볼 수 있음. 모듈이란 한번에 한꺼번에 컴파일되는 코틀린 파일들을 의미한다.)

코틀린은 최상위 선언에 대해서 private 가시성을 허용한다. 이렇게 되면 파일 내부에서만 사용할 수 있다.

| 변경자 | 클래스 멤버 | 최상위 선언 |
| --- | --- | --- |
| public(기본가시성) | 모든 곳에서 볼 수 있다. | 모든 곳에서 볼 수 있다. |
| internal  | 같은 모듈에서만 볼 수 있다. | 같은 모듈안에서만 볼 수 있다. |
| protected | 하위 클래스안에서만 볼 수 있다. | (최상위 선언에 적용X) |
| private | 같은 클래스 안에서만 볼 수 있다. | 같은 파일에서만 볼 수 있다. |

```kotlin
internal open class TalkativeButton : Focusable {
	private fun yell() = println("Hey!")
	protected fun whisper() = println("Let's talk")
}

fun TalktativeButton.giveSpeech() { //에러: public 멤버가 자신의 internal 수신타입인 TalkativeButton을 노출함
	yell() // yell에 접근할 수 없음
	whisper()  // whisper에 접근할 수 없음
}
```

확장함수는 private 함수나 protected함수에 접근할 수 없다.

## 1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

코틀린에서는 중첩 클래스에 아무런 변경자가 붙지 않으면 자바의 static 중첩클래스와 같다.
만약 이를 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 inner 변경자를 붙여야한다.

코틀린의 중첩클래스를 유용하게 사용하는 용례를 하나 살펴보자. 클래스 계층을 만들되 그 계층에 속한 클래스의 수를 제한하고 싶은 경우 중첩클래스를 쓰면 편리하다.

## 1.5 봉인된 클래스: 클래스 계층 정의시 계층 확장 제한

상위 클래스에 `sealed` 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.

```kotlin
sealed class Expr {
	class Num(val value:Int) : Expr()
	class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr) : Int = 
	when(e) {
		is Expr.Num -> e.value
		is Expr.Sum -> eval(e.right) + eval(e.left)  //else가 필요없음
}
```

# 2. 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

## 2.1 클래스 초기화: 주 생성자와 초기화 블록

클래스 이름 뒤에 오는 괄호로 둘러쌓인 코드가 **주 생성자** 

주 생성자는 생성자 파라미터를 지정하고 그 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의하는 두 가지 목적으로 쓰인다.

```kotlin
// 3가지 모두 동일한 클래스다. 
class User(val nickname: String) 

class User constructor(_nickname: String) {
	val nickname: String
	init {
		nickname = _nickname
	}
}

class User(_nickname: String) {
	val nickname = _nickname
}
```

생성자에도 디폴트 파라미터를 지정할 수 있다.

또한, 어떤 클래스를 외부에서 인스턴스화하지 못하게 막고 싶다면 모든 생성자를 private으로 만들 수 있다.

## 2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화

부 생성자는 `constructor` 키워드로 생성할 수 있다. 필요에 따라 얼마든지 부 생성자를 많이 선언해도 된다.

클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야한다.

## 2.3 인터페이스에 선언된 프로퍼티 구현

인터페이스에 추상 프로퍼티 선언을 넣을 수 있다. 

그러나 프로퍼티는 상태를 가질 수 없기 때문에 뒷받침하는 필드가 없다.

그러나 get()을 이용해서 매번 계산해서 줄 수는 있다.

```kotlin
interface User {                       //O
    val email: String
    val nickname: String
        get() = email
}

interface User {                       //X
    val email: String
    val nickname: String = email
}
```

## 2.4 게터와 세터에서 뒷받침하는 필드에 접근

```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value) {
            println("""
                Address was changed for $name:
                "$field" -> "$value".""".trimIndent())
            field = value
		}
}
```

접근자에 본문에서 field라는 특별한 식별자를 통해 뒷받침하는 필드에 접근할 수 있다. 게터는 field값을 읽을 수만 있고 세터에서는 field값을 읽거나 쓸수 있다.

## 2.5 접근자의 가시성 변경

접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같다.

# 3. 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임

## 3.1 모든 클래스가 정의해야 하는 메소드

### 문자열 표현(toString()), 객체의 동등성 (equals), 해시컨테이너(hashcode())

> 참고, 코틀린에서는 `==` 는 equals  `===`  자바의 객체 참조 == 와  같다
> 

## 3.2 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성

데이터 클래스는 toString(), equals, hashCode를 자동으로 생성해준다.

equals와 hashCode는 주 생성자에 나열된 모든 프로퍼티를 고려해 만들어진다.

생성된 equals는 모든 프로퍼티 값의 동등성을 확인한다.

hashCode는 메소드 모든 프로퍼티의 해시 값을 바탕으로 계산한 해시 값을 반환한다.

이때 주 생성자 밖에서 생성된 프로퍼티는 equals와 hashCode의 고려대상이 아니다.

### 데이터 클래스와 불변성: copy() 메소드

데이터 클래스 인스턴스를 불변 객체로 더 쉽게 복사하면서 일부 프로퍼티를 바꿀수 있게 해주는 copy메소드를 자동으로 생성한다.

## 3.3 클래스 위임: by 키워드 사용

상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 때, **데코레이터** 패턴을 사용한다.

상속을 허용하지 않는 클래스 대신 사용할 수 있는 새로운 클래스를 만들되 기존 클래스와 같은 인터페이스를 데코레이터 제공하게 만들고, 기존 클래스를 데코레이터 내부에 필드로 유지하는 것이다.

ex) 원소를 추가하려고 시도한 횟수를 기록하는 컬렉션을 구현해보자.

```kotlin
class CountingSet<T>(
    val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet {
    var objectsAdded = 0

    override fun add(element: T): Boolean {
        objectsAdded++
        return innerSet.add(element)
    }

    override fun addAll(c: Collection<T>): Boolean {
        objectsAdded += c.size
        return innerSet.addAll(c)
    }
}
```

위 코드를 보면 add, addAll은 오버라이드를 해서 카운터를 증가시키고, MutableCollection의 나머지 메소드는 내부 컨테이너(innerSet)에게 위임한다.

# 4. object 키워드: 클래스 선언과 인스턴스 생성

코틀린에서 object 키워드는 다양한 상황에서 사용하는데 모든 클래스를 정의하면서 동시에 인스턴스를 생성한다는 공통점이 있다.

- object declaration은 싱글턴을 정의하는 방법중에 하나이다.
- companion object는 인스턴스 메소드는 아니지만 어떤 클래스와 관련된 팩토리 메소드를 담을 때 쓰인다.
- 객체 식은 자바의 무명 내부  클래스 대신 쓰인다.

## 4.1 Object Declaration: 싱글턴을 쉽게 만들기

코틀린은 Object Declaration을 통해, 싱글턴 언어를 기본 지원한다. Object Declaration는 클래스 선언과 ㅋ그 클래스에 속한 단일 인스턴스의 선언을 합친 선언이다.

> 코틀린 object를 자바에서 사용하려면 INSTANCE 키워드를 사용하면 된다.
> 

## 4.2 companion object: 팩토리 메소드와 정적 메소드가 들어갈 장소

코틀린은 자바 static 키워드를 지원하지 않는다. 그래서 최상위 함수를 활용한다.

그러나 클래스 내부의 private 변수를 사용하고 싶을때 companion object 영역에 만들면 된다.

가장 큰 예로 팩토리 메소드를 해당 영역에 만들 수 있다. (주 생성자는 private)

## 4.3 companion object를 일반 객체처럼 사용

companion object에 이름을 붙일 수 있다.

### companion object에서 인터페이스 구현

```kotlin
interface JSONFactory<T>{
    fun fromJSON(json: String): T
}

class Person(val name: String) {
    companion object PersonJson: JSONFactory<Person> {
        override fun fromJSON(json: String): Person {
            return Person(json)
        }
    }
}
```

### companion object 확장

```kotlin
//비즈니스 로직 모듈
class Person(val firstName: String, val lastName: String) {
    companion object
}
// 클라이언트/서버 통신 모듈
fun Person.Companion.fromJSON(json: String): Person {
    ...
}

val p = Person.fromJSON("json")
```

## 4.4 객체 식: 무명 내부 클래스를 다른 방식으로 작성

무명 객체(anonymous object)를 정의할 떄도 object keyword를 사용한다.

```kotlin
window.addMouseListener(
    object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            // ...
        }
    }
)
```

object declaration과 동일하나 인스턴스에 이름을 붙이지 않는다.