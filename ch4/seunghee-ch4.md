# Ch4. 클래스, 객체, 인터페이스

## 1. 클래스 계층 정의

### 코틀린 인터페이스
Java8의 인터페이스와 비슷
코틀린 인터페이스 안에는 추상 메소드뿐 아니라 구현이 있는 메소드도 정의할 수 있다.
```kotlin
interface Clickable {
fun click()
}

class Button : Clickable {
override fun click() = println("I was clicked")
}

// 결과값
// I was clicked
```
코틀린에서는 클래스 이름 뒤에 콜론(:)을 붙이고 인터페이스와 클래스 이름을 적는 것으로 클래스 확장과 인터페이스 구현을 모두 처리함.
자바와 마찬가지로 클래스는 인터페이스를 개수 제한 없이 마음대로구현할 수 있지만, 클래스는 오직 하나만 확장 가능
인터페이스 메소드도 디폴트 구현을 제공할 수 있으며 메소드 앞에 default를 붙여야 하는 자바 8과 달리 코틀린에서는 메소드를 특별한 키워드로 꾸밀 필요가 없음.
메소드 구현체를 메소드 시그니처 뒤에 추가하면 된다.
```kotlin
interface Clickable {
fun click() // 일반 메소드 선언
fun showOff() = println("I'm clickable!") // 디폴트 구현이 있는 메소드
}
```
만약 다중 인터페이스 상속 관계에서 동일한 메소드가 구현되어 있다면?
이를 구분하기 위해 예를들어 Focusable 인터페이스를 추가적으로 생성
```kotlin
interface Focusable {
fun setFocus(b: Boolean) =
println("I ${if (b) "got" else "lost"} focus.")

    fun showOff() = println("I'm focusable!")
}
```

Button 클래스를 생성하여 Clickable, Focusable 인터페이스를 다중 상속받을 경우
```kotlin
class Button : Clickable, Focusable {
override fun click() = println("I was clicked")
}
```
컴파일 단계에서 코틀린에서는 중복된 상위 메소드는 하위 클래스에서 반드시 구현되어야 한다는 컴파일 오류가 발생한다.
Button class에서도 중복된 상위 메소드인 showOff를 override해야만 함

```
[자바에서 코틀린의 메소드가 있는 인터페이스 구현하기]

코틀린은 자바 6와 호환되게 설계했다. 따라서 인터페이스의 디폴트 메소드를 지원하지 않는다. 
따라서 코틀린은 디폴트 메소드가 있는 인터페이스를 일반 인터페이스와 디폴트 메소드 구현이 정적 메소드로 들어있는 클래스를 조합해 구현한다. 
인터페이스에는 메소드 선언만 들어가며, 인터페이스와 함께 생성되는 클래스에는 모든 디폴트 메소드 구현이 **정적 메소드로** 들어간다. 
그러므로 디폴트 인터페이스가 포함된 코틀린 인터페이스를 자바 클래스에서 상속해 구현하고 싶다면 코틀린에서 메소드 본문을 제공하는 메소드를 포함하는 모든 메소드에 대한 본문을 작성해야 한다
**(즉 자바에서는 코틀린의 디폴트 메소드 구현에 의존할 수 없다)**
```

### open, final, abstract 변경자: 기본적으로 final
자바에서는 final로 명시적으로 상속을 금지하지 않는 모든 클래스를 다른 클래스가 상속할 수 있다.
코틀린의 클래스와 메소드는 기본적으로 final이다. 어떤 클래스의 상속을 허용하려면 클래스 앞에 open 변경자를 붙여야 한다.
그와 더불어 오버라이드를 허용하고 싶은 메소드나 프러퍼티의 앞에도 open 변경자를 붙여야 한다.
```kotlin
open class RichButton : Clickable { // 이 클래스는 열려있다. 다른 클래스가 이 클래스를 상속할 수 있다.

    fun disable() {} // 이 함수는 파이널이다. 하위 클래스가 이 메소드를 오버라이드할 수 없다. 

    open fun animate() {} // 이 함수는 열려있다. 하위 클래스에서 이 메소드를 오버라이드해도 된다. 

    override fun click() {} // 이 함수는 (상위 클래스에서 선언된) 열려있는 메소드를 오버라이드 한다. 오버라이드한 메소드는 기본적으로 열려있다. 
}
```
오버라이드하는 메소드의 구현을 하위 클래스에서 오버라이드하지 못하게 금지하려면 오버라이드하는 메소드 앞에 final을 명시해야 한다.
```kotlin
open class RichButton : Clickable {
// 여기 있는 'final'은 쓸데 없이 붙은 중복이 아니다.
// 'final'이 없는 'override' 메소드나 프로퍼티는 기본적으로 열려있다.
final override fun click() {}
}
```

```
[열린 클래스와 스마트 캐스트]

클래스의 기본적인 상속 가능 상태를 final로 함으로써 얻을 수 있는 큰 이익은 다양한 경우에 스마트 캐스트가 가능하다는 점이다. 
스마트 캐스트는 타입 검사 뒤에 변경될 수 없는 변수에만 적용 가능하기 때문. 
클래스 프로퍼티의 경우 이는 val이면서 커스텀 접근자가 없는 경우에만 스마트 캐스트 할 수 있으며 또한 프로퍼티가 final이어야만 한다는 의미. 
프로퍼티는 기본적으로 final이기 때문에 따로 고민할 필요 없이 대부분의 프로퍼티를 스마트 캐스트에 활용할 수 있다.
```

자바처럼 코틀린에서도 클래스를 abstract로 선언할 수 있다. abstract로 선언한 추상 클래스는 인스턴스화할 수 없다. 따라서 추상 멤버 앞에 open 변경자를 명시할 필요가 없다.
``` kotlin
abstract class Animated { // 이 클래스는 추상클래스다. 이 클래스의 인스턴스를 만들 수 없다.
abstract fun animate() // 이 함수는 추상 함수다. 이 함수에는 구현이 없다. 하위 클래스에서는 이 함수를 반드시 오버라이드해야 한다.
open fun stopAnimating() { ... } // 추상 클래스에 속했더라도 비추상 함수는 기본적으로 final이지만 원한다면 open으로 오버라이드를 허용할 수 있다.
fun animateTwice() { ... } // 추상 클래스에 속했더라도 비추상 함수는 기본적으로 final이지만 원한다면 open으로 오버라이드를 허용할 수 있다.
}
```

인터페이스 멤버는 항상 열려 있으며 final로 변경할 수 없으므로 final, open, abstract를 사용하지 않는다. 
인터페이스 멤버에게 구현체가 없으면 자동으로 추상 멤버가 되므로 멤버 선언 앞에 abstract 키워드를 덧붙일 필요가 없다.

### 클래스 내에서 상속 제어 변경자의 의미

| 변경자  | 이 변경자가 붙은 멤버는… | 설명                                                          |
| --- | --- |-------------------------------------------------------------|
| final | 오버라이드 할 수 없음 | 클래스 멤버의 기본 변경자다.                                            |
| open | 오버라이드 할 수 있음 | 반드시 open을 명시해야 오버라이드 할 수 있다.                                |
| abstract | 반드시 오버라이드 해야함 | 추상 클래스의 멤버에만 이 변경자를 붙일 수 있다. 추상멤버에는 구현이 있으면 안됨.             |
| override | 상위 클래스나 상위 인스턴스의 멤버를 오버라이드 하는 중 | 오버라이드하는 멤버는 기본적으로 열려있다. 하위클래스의 오버라이드를 금지하려면 final을 명시해야 한다. |

### 가시성 변경자: 기본적으로 공개
Kotlin의 기본 가시성은 public이다.
가시성 변경자는 클래스 구현에 대한 외부 접근을 제어한다.
코틀린은 패키지를 네임스페이스를 관리하기 위한 용도로만 사용하므로 패키지를 가시성 제어에 사용하지 않는다.
패키지 전용 가시성에 대한 대안으로 코틀린에는 internal이라는 새로운 가시성 변경자를 사용함.
internal은 "모듈 내부에서만 볼 수 있음"이라는 뜻으로 모듈은 한 번에 한꺼번에 컴파일되는 코틀린 파일들을 의미한다.
모듈 내부 가시성은 모듈의 구현에 대해 진정한 캡슐화를 제공한다는 장점이 있다.
(자바에서는 패키지가 같은 클래스를 선언하기만 하면 어떤 프로젝트의 외부에 있는 코드라도 패키지 내부에 있는 패키지 전용 선언에 쉽게 접근할 수 있다. 그래서 모듈의 캡슐화가 쉽게 깨진다.)

코틀린에서는 최상위 선언에 대해 private 가시성을 허용한다. 
그런 최상위 선언에는 클래스, 함수, 프로퍼티 등이 포함된다. 비공개 가시성인 최상위 선언은 그 선언이 들어있는 파일 내부에서만 사용할 수 있다. 
이 또한 하위 시스템의 자세한 구현 사항을 외부에 감추고 싶을 때 유용한 방법이다.

| 변경자             | 클래스 멤버              | 최상위 선언             |
|-----------------|---------------------|--------------------|
| public(기본 가시성임) | 모든 곳에서 볼 수 있다.      | 모든 곳에서 볼 수 있다.     |
| internal        | 같은 모듈 안에서만 볼 수 있다.  | 같은 모듈 안에서만 볼 수 있다. |
| protected       | 하위 클래스 안에서만 볼 수 있다. | (최상위 선언에 적용할 수 없음) |
| private         | 같은 클래스 안에서만 볼 수 있다. | 같은 파일 안에서만 볼 수 있다. |

``` kotlin
internal open class TalkativeButton : Focusable {
private fun yell() = println("Hey!")
protected fun whisper() = println("Let's talk!")
}

fun TalkativeButton.giveSpeesh() {// 오류: public 함수에서 가시성이 낮은 TalkactiveButton을 수신타입으로 사용 불가
yell() // 오류: 클래스를 확장한 함수는 그 클래스의 private, protected함수에 접근할 수 없다.
whisper() // 오류: 클래스를 확장한 함수는 그 클래스의 private, protected함수에 접근할 수 없다.
}
```

```
[코틀린의 가시성 변경자와 자바]

코틀린의 변경자는 컴파일된 자바 바이크토드 안에서도 그대로 유지되며 자바에서 동일하게 사용할 수 있다. 
유일한 예외는 private 클래스다. 자바에서는 클래스를 private으로 만들 수 없으므로 내부적으로 코틀린은 private 클래스를 패키지-전용 클래스로 컴파일한다.
자바에는 internal에 딱 맞는 가시성이 없다. 패키지-전용 가시성은 internal과는 전혀 다르다. 모듈은 보통 여러 패키지로 이뤄지며 서로 다른 모듈에 같은 패키지에 속한 선언이 들어 있을 수도 있다. 
따라서 internal 변경자는 바이트코드상에서는 public이 된다.
코틀린 선언과 그에 해당하는 자바 선언의 차이 때문에 코틀린에서는 접근할 수 없는 대상을 자바에서 접근할 수있는 경우가 생긴다. 
예를 들어 다른 모듈에 정의된 internal 클래스나 internal 최상위 선언을 모듈 외부의 자바 코드에서 접근할 수 있다.
```


### 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스
자바처럼 코틀린에서도 클래스 안에 다른 클래스를 선언할 수 있다. 
자바와의 차이는 코틀린의 중첩 클래스는 Outer Class 인스턴스에 대한 접근 권한을 얻으려면 명시적으로 inner를 붙여야 함.
기본적으로 코틀린의 내부클래스는 중첩된 외부 클래스에 대한 참조 없음.

[예시]
View 인터페이스 - 뷰의 상태를 가져와 저장하는 getCurrentState와 restoreState 메소드 선언 존재
``` kotlin
interface State: Serializable

interface View {
fun getCurrentState(): State
fun restoreState(state: State) { }
}
```

Button 뷰의 상태를 가져와 저장하는 클래스를 Button 클래스 내부에 선언할 때
[Java]
-> Java Code로 아래와 같이 선언 시 오류 발생 
``` java
// Java 구현
public class Button implements View {
@Override
public State getCurrentState() {
return new ButtonState();
}

		@Override
		public void resoreState(State state) { /*...*/ }
		public class ButtonState implements State { /*...*/ }
}

=> java.io.NotSerializableException: Button
//직렬화하려는 변수는 ButtonState 타입의 state 였는데 Button을 직렬화할 수 없다는 예외가 발생함. 왜냐하면 ButtonState 클래스는 바깥쪽 Button 클래스에 대한 참조를 묵시적으로 포함하기 때문
=> Java에서는 중첩 클래스를 static으로 선언하면 outer class에 대한 묵시적 참조가 사라지므로 ButtonState 클래스를 static으로 선언해야 함.
```

[Kotlin]
``` kotlin
class Button : View {

		override fun getCurrentState(): State = ButtonState();
				
		override fun resoreState(State state) { /*...*/ }

		class ButtonState : State { /*...*/ }
}
```
코틀린 중첩 클래스에 아무런 변경자가 붙지 않으면 자바 static 중첩 클래스와 같다. 
이를 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 inner 변경자를 붙여야 한다.
코틀린에서 바깥쪽 클래스의 인스턴스를 가르키려면 내부 클래스 Inner 안에서 **this@Outer**라고 써야 함.


### 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

[일반 인터페이스를 구현을 통해 식 표현하기]
``` kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =
when (e) {
is Num -> e.value
is Sum -> eval(e.right) + eval(e.left)
else -> // "else" 분기가 꼭 있어야 한다.
throw IllegalArgumentException("Unknown expression")
}
```

디폴트 분기가 있으면 실수로 새로운 클래스 처리를 잊어버려도 디폴트 분기가 선택되므로 이런 클래스 계층에 새로운 하위 클래스를 추가하더라도 컴파일러가 when이 모든 경우를 처리하는지 제대로 검사할 수 없다.

[Sealed 클래스로 식 표현]
상위 클래스에 sealed 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있으며 sealed 클래스의 모든 하위 클래스를 정의할 때는 반드시 상위 클래스 안에 중첩시켜야 한다.
sealed class는 자동으로 open, 
sealed 클래스를 상속한 경우를 명확히 제어 가능.
``` kotlin
sealed class Expr {
class Num(val value: Int) : Expr()
class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr): Int =
when (e) {
is Expr.Num -> e.value
is Expr.Sum -> eval(e.right) + eval(e.left)
}
```

## 2. 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

### 초기화 로직
주(primary) 생성자, 부(secondary) 생성자, 초기화 블럭(initializer block) 등의 방법

### 클래스 초기화: 주 생성자와 초기화 블록
[주 생성자] 
생성자 파라미터를 지정하고 그 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의하는 두 가지 목적으로 사용
클래스 이름 뒤에 오는 괄호로 둘러싸인 코드 부분
[init block]
클래스 객체가 인스턴스화 될 때 실행
주 생성자와 함께 사용되며 주 생성자의 제한적인 동작을 보완
클래스 안에 여러 초기화 블록을 선언할 수 있음

``` kotlin
class User**(val nickname: String)** //val은 이 파라미터에 상응하는 프로퍼티가 생성된다는 뜻

class User**(_nickname: String)** {// _nickname: 생성자 파라미터, nickname: 프로퍼티
val nickname = _nickname
}

class User constructor**(_nickname: String)** { // 파라미터가 하나만 있는 주 생성자
val nickName: String

		init { // 초기화 블록
				nickName = _nickName
		}
}
class User**(val nickname, String, val isSubscribed: Boolean = true)** //디폴트 값 제공 가능, 디폴트 값은 컴파일러가 자동으로 파라미터에서 제외하고 초기화 값으로 사용함, DI방식으로 제공되는 라이브러리 사용 시 파라미터 없는 생성자 유용
```

### 부 생성자: 상위 클래스를 다른 방식으로 초기화
부 생성자가 필요한 이유: 자바 상호운용성, 클래스 인스턴스를 생성할 때 파라미터 목록이 다른 생성 방법이 여럿 존재하는 경우

[인터페이스에 선언된 프로퍼티 구현]
``` kotlin
interface User {
val nickName: String
}

class PrivateUser(override val nickname: String) : User // 주 생성자에 있는 프로퍼티

class SubscribingUser(val email: String) : User {
override val nickname: String // 커스텀 게터
get() = email.substringBefore('@')
}

class FacebookUser(val accountId: Int) : User {
override val nickname = getFacebookName(accountId) // 프로퍼티 초기화 식
}
```
SubscribingUser의 nickname은 매번 호출될 때마다 substringBefore를 호출해 계산하는 커스텀 게터를 활용하고, 
FacebookUser의 nickname은 객체 초기화 시 계산한 데이터를 backing field에 저장했다가 불러오는 방식을 활용한다.
인터페이스에는 추상 프로퍼티뿐 아니라 게터와 세터가 있는 프로퍼티를 선언할 수도 있다. 
물론 그런 게터와 세터는 backing field를 참조할 수 없다.(backing field가 있다면 인터페이스에 상태를 저장한다는 의미인데 인터페이스는 상태를 저장할 수 없다)

``` kotlin
interface User {
val email: String
val nickName: String
get() = email.substringBefore('@') // 프로퍼티에 뒷받침하는 필드가 없다.대신 매번 결과를 계산해 돌려준다.
}
```
위와 같은 경우에 하위 클래스는 추상 프로퍼티인 email을 반드시 오버라이드해야 한다. 반면 nickname은 오버라이드하지 않고 상속할 수 있다.


### 게터와 세터에서 backing field에 접근
값을 저장하는 동시에 로직을 실행할 수 있게 하기 위해서는 해당 setter/getter 안에서 프로퍼티의 backing field에 접근할 수 있어야 한다.
[프로퍼티에 저장된 값의 변경 이력을 로그에 남기려는 예제]
``` kotlin
class User(val name: String) {
var address: String = "unspecified"
set(value: String) {
println("""
Address was changed for $name:
"$field" -> "$value".""".trimIndent())
field = value
}
}

>> val user = User("Alice")
>> user.address = "Elsenheimerstrasse 47, 80687 Muenchen"
Address was changed for Alice:
"unspecified" -> "Elsenheimerstrasse 47, 80687 Muenchen".

```


### 접근자의 가시성 변경
접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같지만 get이나 set 앞에 가시성 변경자를 추가해서 접근자의 가시성을 변경할 수 있다.
``` kotlin
class LengthCounter {
var counter: Int = 0
private set // 이 클래스 밖에서 이 프로퍼티의 값을 바꿀 수 없다.

    fun addWord(word: String) {
        counter += word.length
    }
}
```

## 3. 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임
자바 플랫폼에서는 클래스가 equals, hashCode, toString 등의 메소드를 구현해야 하지만 코틀린은 컴파일러가 생성해줌

### 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성
코틀린은 data라는 변경자를 클래스 앞에 붙이면 필요한 메소드를 컴파일러가 자동으로 만들어준다.
``` kotlin
data class Client(val name: String, val postalCode: Int)
```
이제 Client 클래스는 자바에서 요구하는 모든 메소드를 포함한다.
- 인스턴스 간 비교를 위한 euqals
- HashMap과 같은 해시 기반 컨테이너에서 키로 사용할 수 있는 hashCode
- 클래스의 각 필드를 각 순서대로 표시하는 문자열 표현을 만들어주는 toString

[데이터 클래스와 불변성: copy() 메소드]
데이터 클래스의 모든 프러피를 읽기 전용으로 만들어서 데이터 클래스를 불변 클래스로 만들라고 권장한다. HashMap 등의 컨테이너에 데이터 클래스 객체를 담는 경우엔 불변성은 필수적이다.
데이터 클래스 인스턴스를 불변 객체로 더 쉽게 활용할 수 있게 코틀린 컴파일러는 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해주는 copy 메소드를 제공한다
복사본은 원본과 다른 생명주기를 가지며, 복사를하면서 일부 프로퍼티 값을 바꾸거나 복사본을 제거해도 프로그램에서 원본을 참조하는 다른 부분에 전혀 영향을 끼치지 않는다.

### 클래스 위임: by 키워드 사용
상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 때 일반적으로 데코레이터 패턴을 사용하는데 
이 패턴의 핵심은 상속을 허용하지 않는 클래스 대신 사용할 수 있는 새로운 클래스를 만들되 기존 클래스와 같은 인터페이스를 데코레이터가 제공하게 만들고, 기존 클래스를 데코레이터 내부에 필드로 유지하는 것이다. 
즉, 새로 정의해야 하는 기능은 데코레이터의 메소드에 정의하고, 기존 기능은 데코레이터의 메서드가 기존 클래스의 메소드를 호출하는 방식
이런 접근 방법의 단점은 준비 코드가 상당히 많이 필요하다는 점.
```kotlin
class DelegatingCollection<T> : Collection<T> {
    private val innerList = arrayListOf<T>()

    override val size: Int get() = innerList.size

    override fun isEmpty(): Boolean = innerList.isEmpty()
    override val contains(element: T): Boolean = innerList.contains(element)
    override val iterator(): Iterator<T> = innerList.iterator()
    override val containsAll(elements): Boolean = innerList.containsAll(elements)
}
```
코틀린에서는 인터페이스를 구현할 때 by 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시할 수 있다.

```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = ArrayList<T>()
) : Collection<T> by innerList {
    override fun add(element: T): Boolean {...}
    ...
}

```
add와 addAll을 오버라이드해서 카운터를 증가시키고, MutableCollection 인터페이스의 나머지 메소드는 내부 컨테이너(innerSet)에게 위임한다.


### object 키워드: 클래스 선언과 인스턴스 생성
객체 선언은 싱글턴을 정의하는 방법 중 하나이며, 동반 객체는 인스턴스 메소드는 아니지만 어떤 클래스와 관련 있는 메소드와 팩토리 메소드를 담을 때 쓰인다. 
동반 객체 메소드는 접근할 때는 동반 객체가 포함된 클래스의 이름을 사용할 수 있다.
객체 식은 자바의 무명 내부 클래스 대신 쓰인다.

### 객체 선언: 싱글턴을 쉽게 만들기
object 키워드를 이용한 객체 선언 기능을 통해 싱글턴을 언어에서 기본 지원한다. 즉 객체 선언은 클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을 합친 선언이다.
생성자 사용 불가함. 생성자 호출 없이 즉시 생성되기 때문. 프로퍼티/메소드/초기화 블럭은 가능함.
객체 선언도 클래스나 인터페이스를 상속할 수 있다. 프레임워크를 사용하기 위해 특정 인터페이스를 구현해야 하는데, 클래스마다 단 하나씩만 있으면 되는 기능 구현시 유용.
``` kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()

    fun calculateSalary() {
        for (person in allEmployees) {
            ...
        }
    }
}

Payroll.allEmployees.add(Person())
Payroll.calculateSalary()
```

### 동반 객체: 팩토리 메소드와 정적 멤버가 들어갈 장소
코틀린 클래스 안에는 정적인 멤버가 없고, 코틀린 언어는 자바 static 키워드를 지원하지 않는다. 
대신 클래스 안에 정의된 객체 중 하나에 companion 이라는 특별한 표시를 붙이면 그 클래스의 동반 객체로 만들 수 있고,패키지 수준의 최상위 함수와 객체 선언이 된다.
동반 객체의 프로퍼티나 메소드에 접근하려면 그 동반 객체가 정의된 클래스 이름을 사용한다.
``` kotlin
class A {
companion object {
fun bar() {
println("Companion object called")
}
}
}

>>> A.bar()
Companion object called
```

동반 객체에서 private 생성자를 호출하는 예 - 동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있음.


```kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))

        fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
    }
}
```

### 동반 객체를 일반 객체처럼 사용
동반 객체는 클래스 안에 정의된 일반 객체다. 따라서 동반 객체에 이름을 붙이거나, 동반 객체가 인터페이스를 상속하거나, 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 있다.
``` kotlin
class Person(val name: String) {
companion object Loader {
fun fromJSON(jsonText: String) : Person = ... // 동반 객체에 이름을 붙인다
}
}

>>> person = Person.Loader.fromJSON("{name: 'Dmitry'}")
>>> person.name
Dmitry
>>> person2 = Person.fromJSON("{name: 'Brent'}")
>>> person2.name
Brent
```


### 동반 객체에서 인터페이스 구현
다른 객체 선언과 마찬가지로 동반 객체도 인터페이스를 구현할 수 있다.
``` kotlin
interface JSONFactory<T> {
fun fromJSON(jsonText: String): T
}
class Person(val name: String) {
companion object : JSONFactory<Person> {
override fun fromJSON(jsonText: String): Person = ... // 동반 객체가 인터페이스를 구현한다.
}
}
```
이제 JSON으로부터 각 원소를 다시 만들어내는 추상 팩토리가 있다면 Person 객체를 그 팩토리에게 넘길 수 있다.
=> 팩토리 패턴 구현에 적합


### 코틀린 동반 객체와 정적 멤버
클래스와 동반 객체는 일반 객체와 비슷한 방식으로, 클래스에 정의된 인스턴스를 가리키는 정적 필드로 컴파일된다. 동반 객체에 이름을 붙이지 않았다면자바 쪽에서 Companion이라는 이름으로 그 참조에 접근할 수 있다.
객체 식: 무명 내부 클래스를 다른 방식으로 작성
무명 객체(anonymous object)를 정의할 때도 object 키워드를 쓴다. 무명 객체는 자바의무명 내부 클래스를 대신한다.
``` kotlin
window.addMouseListener(
object : MouseAdapter() {
override fun mouseClicked(e: MouseEvent) { ... }
override fun mouseEntered(e: MouseEvent) { ... }
}
)

val listener = object : MouseAdapter() {
override fun mouseClicked(e: MouseEvent) { ... }
override fun mouseEntered(e: MouseEvent) { ... }
}
```
한 인터페이스만 구현하거나 한 클래스만 확장할 수 있는 자바의 무명 내부 클래스와 달리 코틀린 무명 클래스는 여러 인터페이스를 구현하거나 클래스를 확장하면서 인터페이스를 구현할 수 있다.
자바의 무명 클래스와 같이 객체 식 안의 코드는 그 식이 포함된 함수의 변수에 접근할 수 있다. 하지만 자바와 달리 final이 아닌 변수도 객체 식 안에서 사용할 수 있다.

