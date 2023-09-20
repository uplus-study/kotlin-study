# 클래스, 객체, 인터페이스

- 코틀린은 기본적으로 final이며 public이다.
- 중첩 클래스에는 외부 클래스에 대한 참조가 없다.
- 코틀린 컴파일러는 유용한 메서드를 자동으로 생성해준다.

## 클래스 계층 정의

- 코틀린의 가시정/접근 변경자는 자바와 비슷하지만 기본 가시성은 다르다.
- sealed는 클래스 상속을 제한한다.

### 코틀린 인터페이스

- 코틀린 인터페이스는 자바와 유사
- 추상 메서드 뿐 아니라 구현이 있는 메서드도 정의 가능
- 아무런 상태도 들어갈 수 없다.

인터페이스는 interface 키워드로 정의한다.
이 인터페이스를 구현하는 click()에 대한 구현을 제공해야한다.

~~~kotlin
interface Clickable {
    fun click()
}
~~~

- 클래스 이름 뒤에 콜론(:)을 붙이고 인터페이스와 클래스 이름을 적는 것으로 클래스 확장과 인터페이스 구현을 모두 처리한다.
- 인터페이스 구현은 무제한, 클래스는 하나만 확장 가능
- override 변경자는 반드시 필요하다.

~~~kotlin
class Button : Clickable {
    override fun click() = println("I was clicked")
}
~~~

디폴트 구현이 포함된 메서드

- 메서드 본문을 메서드 시그니처 뒤에 추가

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}
```

한 클래스에서 두 인터페이스를 함께 구현하는데 모두 동일한 이름을 가진 디폴트 메서드가 있을 경우 오버라이딩 메서드가 필요하다.

```kotlin
class Button : Clickable, Focusable {
    override fun click() = println("I was clicked")
    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

### open, final, abstract 변경자: 기본적으로 final

- 코틀린의 클래스와 메서드는 기본적으로 final이다.
- 상속을 허용하려면 open 변경자를 붙여야한다.
- 오버라이드를 허용하고 싶은 메서나 프로퍼티 앞에도 open 변경자를 붙여야 한다.

~~~kotlin
open class RichButton : Clickable { // 다른 클래스가 이 클래스를 상속 가능
    fun disable() {} // final 메서드, 하위 클래스에서 오버라이드 불가
    open fun animate() {} // 오버라이드 가능
    override fun click() {} // 열려있는 메서드를 오버라이드
}
~~~

오버라이드 금지하기

~~~kotlin
open class RichButton : Clickable {
    final override fun click() {} // final로 오버라이드 금지
}
~~~

> 열린 클래스와 스마트 캐스트
> > 스마트 캐스트는 타입 검사 후에 변경될 수 없는 변수에만 적용 가능
> > - 클래스 프로피티는 val이면서 커스텀 접근자가 없는 경우
> > - 프로퍼티가 final인 경우

추상클래스 정의하기

~~~kotlin
abstract class Animated { // 추상 클래스, 이 클래스의 인스턴스를 만들 수 없다.
    abstract fun animate() // 추상 메서드, 반드시 구현해야 한다.
    open fun stopAnimating() {} // 오버라이드 허용
    fun animateTwice() {} // 기본적으로 final
}
~~~

|변경자|이 변경자가 붙은 멤버는| 설명                             |
|---|---|--------------------------------|
|final|오버라이드 할 수 없다.| 기본적으로 적용                       |
|open|오버라이드 할 수 있다.| open을 명시해야 상속한 클래스에서 오버라이드 가능  |
|abstract|반드시 오버라이드 해야한다.| 하위 클래스에서 반드시 오버라이드 해야한다.       |
|override|오버라이드 한다.| 상위 클래스나 상위 인스턴스의 멤버를 오버라이드 한다. |

### 가시성 변경자: 기본적으로 공개

- internal : 모듈 내부에서만 볼 수 있다.

|변경자|클래스 멤버|최상위 선언|
|---|---|---|
|public|모든 곳에서 볼 수 있다.|모든 곳에서 볼 수 있다.|
|internal|같은 모듈 안에서만 볼 수 있다.|같은 모듈 안에서만 볼 수 있다.|
|protected|하위 클래스 안에서만 볼 수 있다.|최상위 선언에 적용할 수 없다.|
|private|클래스 안에서만 볼 수 있다.|파일 안에서만 볼 수 있다.|

> 코틀린의 가시성 변경자와 자바
> > 코틀린을 자바로 컴파일하면 가시성 변경자가 어떻게 변환되는지 알아보자.
> > - 코틀린의 private 클래스는 패키지-전용 클래스로 컴파일한다.
> > - internal 변경자는 자바에서 public으로 변경

### 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없음

|클래스 B 안에 젇ㅇ의된 클래스 A|자바에서는|코틀린에서는|
|---|---|---|
|중첩 클래스(바깥쪽 클래스에 대한 참조를 저장하지 않음)|static class A|class A|
|내부 클래스(바깥쪽 클래스에 대한 참조를 저장함)|class A|inner class A|

### 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

- 클래스를 상속할 수 없게 하려면 클래스 앞에 sealed 변경자를 붙인다.
- sealed 클래스의 하위 클래스를 정의할 때는 반드시 상위 클래스 안에 중첩시켜야한다.

~~~kotlin
sealed class Expr {
    // 기반 클래스의 모든 하위 클래스를 중첩 클래스로 나열
    class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr): Int =
    when (e) { // 모든 하위 클래스를 검사하므로 else 분기가 없어도 된다.
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    }
~~~

## 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

- 주 생성자와 부 생성자 구분
- 초기화 블록을 통해 초기화 로직을 추가

### 클래스 초기화: 주 생성자와 초기화 블록
~~~kotlin
class User(val nickname: String)  // 주 생성자
~~~
- constructor 키워드를 사용하여 클래스를 정의하고 생성자를 선언할 수 있다.
- init 키워드는 초기화 블록을 시작, 객체가 만들어질 때 실행
- 애노테이션이나 가시성 변경자가 없다면 constructor를 생략 가능
~~~kotlin
class User(_nickname: String) // 주 생성자
{
    val nickname: String 
    init { 
        nickname = _nickname // 초기화 블록
    }
}
~~~
- 주 생성자 이름 앞에 val을 추가하는 방식으로 프로퍼티 정의와 초기화를 간략히 쓸 수 있음
- 생성자 파라미터에 디폴트 값을 정의 가능
~~~kotlin
class User(val nickname: String, 
            val isSubscribed: Boolean = true)
~~~
클래스에 기반 클래스가 있다면 주생성자에서 기반 클래스의 생성자를 호출해야 함
~~~kotlin
open class User(val nickname: String)
class TwitterUser(nickname: String) : User(nickname)
~~~
클래스를 정의할 때 생성자를 정의하지 않은면 컴파일러에서 디폴트 생성자를 생성
~~~kotlin
open class Button
~~~
Button 클래스를 상속한 하위클래스는 Button클래스의 생성자를 호출해야함
~~~kotlin
class RadioButton : Button()
~~~
어떤 클래스를 외부에서 인스턴스화하지 못하게 막고 싶다면 모든 생성자를 private으로 설정

### 부 생성자: 상위 클래스를 다른 방식으로 초기화
- constructor 키워드로 시작, 여러개 선언 가능
- 클래스를 확장하면서 똑같이 부 생성자를 정의할 수 있음
- 클래스에 주 생성자가 없다면 부 생성자는 반드시 상위 클래스를 초기화 하거나 다른 생성자에게 생성을 위임 해야 함

### 인터페이스에 선언된 프로퍼티 구현
- 인터페이스에는 추상 프로퍼티 선언 가능
- 게터와 세터는 뒷받침하는 필드를 찹조할 수 없다.

### 게터와 세터에서 뒷받침하는 필드에 접근
세터에서 뒷받침하는 필드 접근하기
~~~kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println("""
                Address was changed for $name:
                "$field" -> "$value".""".trimIndent())
            field = value
        }
}
~~~

### 접근자의 가시성 변경
- 접근자의 가시성을 변경하려면 접근자 앞에 가시성 변경자를 붙여야 한다.
~~~kotlin
class LengthCounter {
    var counter: Int = 0
        private set // 이 클래스 밖에서는 이 프로퍼티의 값을 변경할 수 없다.
    fun addWord(word: String) {
        counter += word.length
    }
}
~~~

## 컴파일러가 생성한 메서드: 데이터 클래스와 클래스 위임
### 모든 클래스가 정의해야 하는 메서드
#### 문자열 표현 : toString()
~~~kotlin
class Client(val name: String, val postalCode: Int) {
    override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
~~~
#### 객체의 동등성 : equals()
~~~kotlin
class Client(val name: String, val postalCode: Int) {
    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Client)
            return false
        return name == other.name &&
                postalCode == other.postalCode
    }
}
~~~
> 동등성 연산에 ==를 사용함
>>
> > 자바에서는 ==은 참조 동등성을 검사하고, equals는 객체의 내용 동등성을 검사한다.
> 코틀린에서는 ==가 내부적으로 equals을 호출해서 비교한다. 참조 비교를 위해서는 ===을 사용한다.

#### 해시 컨테이너: hashCode()
JVM언어에서는 "equals() true를 반화하기 위해서는 두 객체는 반드시 같은 hashCode()를 반환해야 한다"는 제약이 있다. 따라서 equals()가 의도대로 동작하려면 hashCode()를 오버라이드해야 한다.
~~~kotlin
class Client(val name: String, val postalCode: Int) {
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
~~~

### 데이터 클래스 : 모든 클래스가 정의해야 하는 메서드 자동 생성
- 데이터 클래스는 컴파일러가 equals(), hashCode(), toString()을 자동으로 생성

#### 데이터 클래스와 불변성 : copy() 메서드
- var 프로퍼티를 사용할 수 있지만 val 프로퍼티를 권장
- HashMap 등의 컨테이너에 데이터 클래스 객체를 담는 경우 불변성이 필수
- copy() 메서드는 객체를 복사함녀서 일부 프로퍼티를 바꿀 수 있음, 복사본을 제거해도 원본에 영향 없음

#### 클래스 위임 : by 키워드 사용
데코레이터 패턴
- 상속을 허용하지 않는 클래스 대신 사용할 수 있는 새로운 클래스를 만들되 기존 클래스와 같은 인터페이스를 데코레이터가 제공하게 만들고, 기존 클래스를 데코레이터의 내부에 필드로 유지한다.
- by 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임중이라는 사실을 명시한다.
- 메서드의 일부 동작을 변경하고 싶은 경우 메소드를 오버라이드하면 컴파일러가 생성한 메서드 대신 오버라이드한 메서드가 쓰인다.
~~~kotlin
class DelegatingCollection<T> (
    innerList: Collection<T> = ArrayListOf<T>()
    ) : Collection<T> by innerList ()
~~~
클래스 위임 사용하기
~~~kotlin
class CountingSet<T>(
    val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet { // MutableCollection의 구현을 innerSet에게 위임
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
~~~

## object 키워드: 클래스 선언과 인스턴스 생성
- 객체 선언 : 싱클턴을 정의하는 방법 중 하나
- 동반 객체 : 인스턴스 메서드는 아니지만 어떤 클래스와 관련있는 메서드와 팩토리 메서드를 담을 때 쓰인다. 동반 객체 메스드에 접근할 때 동반 객체가 포함된 클래스의 이름 사용 가능
- 객체 식은 자바의 무명 내부 클래스 대신 쓰인다.

### 객체 선언 : 싱글턴을 쉽게 만들기
- 클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을 합친 선언
- 생성자는 객체 선언에 쓸 수 없음
- 구현 내부에 다른 상태가 필요하지 않은 경우 유용

### 동반 객체 : 팩토리 메서드와 정적 멤버가 들어갈 장소
- 코틀린은 자바의 static 키워드를 지원하지 않는다.
- 최상위 함수와 객체 선언을 활용
- 클래스 안에 정의된 객체 중 하나의 companion 키워드를 붙인 객체를 정의하면 그 클래스의 동반 객체가 된다.
- 동반 객체의 멤버는 클래스 이름을 통해 호출할 수 있다.
- 자바의 정적 메서드 호출이나 정적 필드 사용 구문과 같아짐

부 생성자를 팩토리 메서드로 대신하기
~~~kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) =
            User(email.substringBefore('@'))
        fun newFacebookUser(accountId: Int) =
            User(getFacebookName(accountId))
    }
}
~~~
클래스 이름을 사용해 동반 객체의 메서드를 호출할 수 있다.
~~~kotlin
val subscribingUser = User.newSubscribingUser("bob@gmail.com")
val facebookUser = User.newFacebookUser(4)
~~~
팩토리 메서드를 사용하면 생성할 필요가 없는 객체는 생성하지 않을 수 있다. 클래스를 확장해야하는 경우에는 동반 객체 멤버를 하위 클래스에서 오버라이드 할 수 없으므로 여러 생성자를 사용하는 편이 낫다.

### 동반 객체를 일반 객체처럼 사용
- 동반 객체에 이름을 붙이면 동반 객체의 일반 메서드나 프로퍼티에 접근할 때 동반 객체의 이름을 사용할 수 있다.

#### 동반 객체에서 인터페이스 구현
~~~kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}
class Person(val name: String) {
    companion object : JSONFactory<Person> {
        override fun fromJSON(jsonText: String): Person = ... // 동반 객체가 인터페이스를 구현
    }
}
~~~

> 코틀린 동반 객체와 정적 멤버
> > - 동반 객체에 이름을 붙이지 않으면 자바에서 Companion이라는 이름으로 그 참조에 접근할 수 있다.
> > - 자바에서 사용하기 위해 코틀린 클래스의 멤버를 정적인 멤버로 만드려면 @JvmStatic 애노테이션을 붙여야 한다.
> > - 정적 필드가 필요하다면 @JvmField 애노테이션을 붙인다.

#### 동반 객체 확장
클래스에 동반 객체가 있으면 그 객체 안에 함수를 정의함으로써 클래스에 대한 확장 함수를 만들 수 있다.
~~~kotlin
class Person(val firstName: String, val lastName: String) {
    companion object {} // 비어있는 동반 객체 선언
}
fun Person.Companion.fromJSON(json: String): Person {
    ... // 확장 함수 선언
}
val p = Person.fromJSON(json)
~~~
#### 객체 식: 무명 내부 클래스를 다른 방식으로 작성
- object 키워드를 쓴다.
- 객체에 이름을 붙여야 한다면 변수에 무명 객체를 대입하면 된다.
- 무명 객체는 싱글턴이 아니다.
- 객체의 식 안에서 바깥의 변수에 접근할 수 있다.
> - 객체 식은 무명 객체 안에서 여러 메서드를 오버라이드 하는 경우에 유용
> - 메서드가 하나뿐인 인터페이스를 구현하는 경우 SAM(Single Abstract Method)가 유용

## 요약
- 코틀린의 인터페이스는 디폴트 구현을 포함
- 모든 코틀린 선언은 기본적으로 final 이며 public 이다.
- 선언이 final이 되지 않게 만드려면 앞에 open을 붙여야한다.
- internal 변경자를 사용하면 같은 모듈 안에서만 볼 수 있다.
- 중첩 클래스 기본적으로 내부 클래스가 아니다. inner 키워드를 중첩 클래스 선언 앞에 붙여서 내부 클래스로 만들어야한다.
- sealed 클래스를 상속하는 클래스를 정의하려면 반드시 부모 클래스 정의 안에 중첩 클래스로 정의 필요
- 초기화 블록과 부 생성자를 활용해 클래스 인스턴스를 더 유연하게 초기화할 수 있다.
- field 식별자를 통해 게터와 세터 안에서 프로퍼티의 데이터를 저장하는 데 쓰이는 뒷받침 필드 참조 가능
- 데이터 클래스를 사용하면 컴파일러가 equals(), hashCode(), toString()을 자동으로 생성
- 클래스 위임을 사용하면 데코레이터 패턴을 간결하게 구현할 수 있다.
- 객체 선언을 사용하면 코틀린 답게 싱글턴 클래스를 정의할 수 있다.
- 동반 객체는 자바의 정적 메서드와 필드 정의를 대신한다.
- 동백 객체도 인터페이스를 구현 가능, 외부에서 동반 객체에 대한 확장 함수와 프로퍼티를 정의할 수 있다.
- 객체 식을 사용하면 무명 내부 클래스를 정의할 수 있다.