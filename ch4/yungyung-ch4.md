## 4.1 클래스 계층 정의

### 코틀린 인터페이스

- 추상 메서드 + 구현이 있는 메서드 정의 가능
- `:` 키워드: 클래스 확장, 인터페이스 구현 모두 처리
- `override` 변경자 꼭 사용
- 디폴트 구현 제공: 메서드 본문에 추가
- 클래스가 구현하는 두 상위 인터페이스에서 같은 이름의 메서드가 있을 경우: 상위 타입의 이름을 지정

    ```kotlin
    override fun shoOff()
    	super<Clickable>.shoOff()
    ```

### open, final, abstract 변경자: 기본적으로 final

취약한 기반 클래스: 하위 클래스가 기반 클래스에 대해 가졌던 가정이 기반 클래스를 변경함으로써 깨져버린 경우

- 코틀린의 클래스와 메서드: 기본적으로 `final`
- 상속/오버라이드 허용: `open` 변경자

```kotlin
open class RichButton : Clickable {
    final override fun click() {} // override 메서드/프로퍼티는 기본적으로 열려 있다 -> 상속 방지
}
```

> 열린 클래스와 스마트 캐스트
> final 프로퍼티 = 스마트 캐스트 가능
>

`abstract` 클래스: 인스턴스화할 수 없다 → `open` 변경자 명시 x

```kotlin
abstract class Animated {
    abstract fun animate() // 추상 함수 - 반드시 오버라이드

    open fun stopAnimating() {} // 추상 클래스의 비추상 함수: 기본적으로 final

    fun animateTwice() {} // 오버라이드 불가
}
```

| 변경자      | 오버라이드 상태                       | 설명                      |
|----------|--------------------------------|-------------------------|
| final    | 오버라이드 불가                       | 클래스 멤버의 기본 변경자          |
| open     | 오버라이드 가능                       | open 명시했을 때 오버라이드 가능    |
| abstract | 반드시 오버라이드해야 함                  | 추상 클래스의 멤버에만 가능         |
| override | 상위 클래스나 상위 인스턴스의 멤버를 오버라이드하는 중 | 오버라이드하는 멤버는 기본적으로 열려 있다 |

### 가시성 변경자: 기본적으로 공개

- 아무 변경자도 없는 경우 모두 공개(public)
- 패키지를 가시성 제어에 사용x
    - internal: 모듈 내부에서만 볼 수 있다
- private: 최상위 선언에 대해 허용-그 선언이 들어있는 파일 내부에서만 사용

| 변경자        | 클래스 멤버             | 최상위 선언            |
|------------|--------------------|-------------------|
| public(기본) | 모든 곳에서 볼 수 있다      | 모든 곳에서 볼 수 있다     |
| internal   | 같은 모듈 안에서만 볼 수 있다  | 같은 모듈 안에서만 볼 수 있다 |
| protected  | 하위 클래스 안에서만 볼 수 있다 | (최상위 선언에 적용 불가)   |
| private    | 같은 클래스 안에서만 볼 수 있다 | 같은 파일 안에서만 볼 수 있다 |

```kotlin
internal open class TalkativeButton : Focusable {
    private fun yell() = println("Hey!")
    protected fun whisper() = println("Let's talk!")
}

fun TalkativeButton.giveSpeech() { // error: public 멤버가 자신의 internal 타입(TalkativeButton) 호출
    yell() // error: yell은 private 멤버

    whisper() // error: whisper는 protected 멤버
}
```

- 어떤 클래스의 기반 타입 목록에 들어있는 타입이나 제네릭 클래스의 타입 파라미터에 들어있는 타입의 가시성은 그 클래스 자신의 가시성과 같거나 더 높아야 한다
- 메서드의 시그니처에 사용된 모든 타입의 가시성은 그 메서드의 가시성과 같거나 더 높아야 한다

### 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

- 클래스 안에 다른 클래스 선언 가능
- 중첩 클래스는 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다

자바에서 다른 클래스 안에 정의한 클래스는 자동으로 inner class가 된다

```kotlin

import java.io.Serializable

interface State : Serializable

interface View {
    fun getCurrentState(): State
    fun restoreState(state: State)
}

///////////////////////////////////

public class Button implements View {
    @Override
    public State getCurrentState() {
        return new ButtonState ();
    }

    public void restoreState(State state) {
    }

    public class ButtonState implements State {
}
}
```

- ButtonState 클래스는 Button 클래스에 대한 참조를 묵시적으로 포함 → ButtonState 직렬화 불가
    - Button이 직렬화가 불가능하기 때문
    - ButtonState를 `static class`로 선언 → 중첩 클래스의 바깥쪽 클래스에 대한 묵시적 참조가 사라진다

코틀린

```kotlin
class Button : View {
    override fun getCurrentState(): State = ButtonState()
    override fun restoreState(state: State) {}
    class ButtonState : State {} 
```

코틀린 중첩 클래스에 아무 변경자가 없으면 `static` 중첩 클래스와 같다

- 바깥쪽 클래스에 대한 참조를 포함하게 하려면 `inner` 변경자 필요

| 클래스 B 안에 정의된 클래스 A       | 자바             | 코틀린           |
|--------------------------|----------------|---------------|
| 중첩 클래스(B에 대한 참조 저장하지 않음) | static class A | class A       |
| 내부 클래스(B에 대한 참조 저장)      | class A        | inner class A |

코틀린 내부 클래스 안에서 바깥쪽 클래스에 접근하려면 `this@Outer` 표시

```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```

### 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

`sealed` 클래스: 상위 클래스를 상속한 하위 클래스 정의를 제한

```kotlin
sealed class Expr {
    class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}
```

기반 클래스를 `sealed`로 봉인

기반 클래스의 모든 하위 클래스를 중첩 클래스로 나열

`sealed` 클래스는 자동으로 `open`

`sealed` 인터페이스는 불가능 : 자바에서 구현하지 못하게 막을 수 있는 수단이 없다

## 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

### 클래스 초기화: 주 생성자와 초기화 블록

- 주 새성자: 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드

```kotlin
class User(val nickname: String)

class User constructor(_nickname: String) {
    val nickname: String

    init {
        nickname = _nickname // 초기화
    }
}
```

`constructor`: 주 생성자와 부 생성자 정의

주 생성자에 `val` 추가: 프로퍼티 생성 의미 → 프로퍼티 정의 + 초기화

💡 모든 생성자 파라미터에 디폴트 값을 지정하면 컴파일러가 자동으로 파라미터가 없는 생성자를 만들어준다 → DI 프레임워크와의 통합 용이

클래스에 기반 클래스가 있다면 기반 클래스 이름 뒤에 생성자 인자를 넘김

```kotlin
open class User(val nickname: String)

class TwitterUser(nickname: String) : User(nickname)
```

클래스 정의 시 생성자를 정의하지 않으면 컴파일러가 인자가 없는 디폴트 생성자를 만들어줌

생성자를 private으로 변경 가능

```kotlin
class Secretive private constructor() {}
```

### 부 생성자: 상위 클래스를 다른 방식으로 초기화

constructor 키워드를 사용하여 부 생성자 여러 개 선언 가능

```kotlin
class MyButton : View {
    constructor(ctx: Context) : super(ctx) {} // 상위 클래스의 생성자 호출

    constructor(ctx: Context) : this(ctx, MY_STYLE) {} // 이 클래스의 다른 생성자에 위임
}
```

### 인터페이스 선언된 프로퍼티 구현

인터페이스에 추상 프로퍼티 선언: 인터페이스를 구현하는 클래스가 프로퍼티 값을 얻을 수 있는 방법을 제공해야 한다

```kotlin
interface User {
    val nickname: String
}

class PrivateUser(override val nickname: String) : User

class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@') // 커스텀 게터
}

class FacebookUser(val accountId: Int) : User {
    override val nickname = getFacebookName(accountId) // 프로퍼티 초기화 식
}
```

인터페이스에 게터/세터가 있는 프로퍼티를 선언할 수 있다

```kotlin
interface User {
    val email: String // 반드시 오버라이드
    val nickname: String // 오버라이드하지 않고 상속 가능
        get() = email.substringBefore('@')
}
```

### 게터와 세터에서 뒷받침하는 필드에 접근

뒷받침 필드의 유무

- 컴파일러는 디폴트 접근자 구현/커스텀 게터, 세터 구현에 관계없이 `field`를 사용하는 프로퍼티에 대해 뒷받침하는 필드 생성
- `field`를 사용하지 않는 커스텀 접근자 구현 시 뒷받침 필드는 존재하지 않는다

💡 뒷받침 필드: **프로퍼티의 값을 저장하기 위한 필드** 

### 접근자의 가시성 변경

접근자의 가시성 = 프로퍼티의 가시성

- `get`/`set` 앞에 가시성 변경자를 추가해서 변경 가능

## 4.3 컴파일러가 생성한 메서드: 데이터 클래스와 위임

코틀린 컴파일러: `equals`, `hashCode`, `toString`을 자동 생성 (데이터 클래스)

### 모든 클래스가 정의해야 하는 메서드

코틀린 `==` : `equals` 메서드호출


💡 JPA Entity에서 `equals`, `hashCode` 메서드 오버라이드가 필요한가?



### 데이터 클래스: 모든 클래스가 정의해야 하는 메서드 자동 생성

- equals: 모든 프로퍼티 값의 동등성 확인
- hashCode: 주 생성자에 나열된 모든 프로퍼티 고려
- toString

**copy 메서드: 데이터 클래스와 불변성**

- 데이터 클래스를 불변 클래스로 만들 것을 권장
    - 다중 스레드에서 중요
- `copy`: 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해준다
    - 원본과 다른 생명주기
    - 원본을 참조하는 부분에 영향x

### 클래스 위임: by 키워드 사용

상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 때

- 데코레이터 패턴: 상속을 허용하지 않는 클래스 대신 사용할 수 있는 새로운 클래스를 만들되, 기존 클래스와 같은 인터페이스를 데코레이터가 제공하게 만들고, 기존 클래스를 데코레이터 내부에 필드로 유지
- 새로 정의해야 하는 기능: 데코레이터의 메서드에 정의
- 기존 기능: 데코레이터의 메서드가 기존 클래스의 메서드에게 요청을 전달
- 단점: 준비 코드가 많이 필요

```kotlin
class DelegatingCollection<T> : Collection<T> {
    private val innerList = arrayListOf<T>()

    override val size: Int get() = innerList.size

    override fun isEmpty(): Boolean = innerList.isEmpty()
}
```

코틀린: by 키워드를 통해 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시

```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = ArrayList<T>()
) : Collection<T> by innerList {
    override fun ~~~
}

```

새로 구현하는 메서드는 `override` 함수를 통해 구현

## 4.4 objet 키워드: 클래스 선언과 인스턴스 생성

`object` 키워드 = 클래스를 정의하면서 동시에 인스턴스(객체)를 생성한다

- 객체 선언(object declaration): 싱글턴 정의
- 동반 객체(companion object): 어떤 클래스와 관련 있는 메서드와 팩토리 메서드를 담을 때 쓰인다
- 객체 식: 무명 내부 클래스 대신

### 객체 선언: 싱글턴을 쉽게 만들기

인스턴스가 하나만 필요한 클래스가 유용한 경우 존재

싱글턴 패턴

```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()

    fun calculateSalary() {
        ..
    }
}
```

- 코틀린 객체 선언: 클래스 선언 + 클래스에 속한 단일 인스턴스의 선언
- 생성자 사용 불가 → 생성자 호출 없이 즉시 생성되기 때문
- 클래스/인터페이스 상속 가능

```kotlin
object CaseInsensitiveFileComparator : Comparator<File> {
    override fun compare(...) {
        ...
    }
}
```

Comparator 구현: 안에 데이터를 저장할 필요가 없음, 클래스마다 하나씩만 존재하면 된다 → 객체 선언이 적절

- 프레임워크 사용을 위해 특정 인터페이스 구현 + 다른 상태가 필요하지 않은 경우에 유용
- 일반 객체를 사용할 수 있는 곳에 싱글턴 객체 사용 가능

```kotlin
data class Person(val name: String) {
    object Namecomparator : Comparator<Person> {
        override fun compare(~~)
    }
}
```

- Comparator는 클래스 내부에 정의하는 게 더 바람직, 클래스 안에도 객체 선언 O

💡 자바에서 사용
`INSTANCE`로 컴파일 됨 (유일한 인스턴스에 대한 정적인 필드가 있는 자바 클래스로 컴파일)


### 동반 객체: 팩토리 메서드와 정적 멤버가 들어갈 장소

코틀린 클래스 안에는 정적인 멤버X

팩토리 메서드: 클래스의 인스턴스와 관계없이 호출 + 클래스 내부 정보에 접근해야할 때

```kotlin
class A {
    companion object {
        fun bar() {
            println("")
        }
    }
}
```

자바의 정적 메서드 호출/정적 필드 사용 구문과 같아진다

동반 객체: private 생성자를 호출하기 좋은 위치

- 자신을 둘러싼 클래스의 모든 private 멤버에 접근 가능
- 팩토리 패턴 구현에 적합하다

```kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))

        fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
    }
}
```

### 동반 객체를 일반 객체처럼 사용

- 동반 객체에 이름을 붙일 수 있다
    - 이름을 붙여 호출해도 되고, 클래스 이름을 사용해서 호출도 가능
- 인터페이스 구현

```kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
    companion object : JSONFactory<Person> {
        override fun fromJSON(jsonText: String): Person = ...
    }
}

fun loadFromJSON<T>(factory: JSONFactory<T>): T {}

loadFromJSON(Perosn) // 동반 객체의 인스턴스를 함수에 넘긴다
```

**동반 객체 확장**

```kotlin
class Person(val firstName: String, val lastName: String) {
    companion object {}
}

fun Person.Companion.fromJSON(json: String): Person {
    ...
} // 확장 함수

val p = Person.fromJSON(json) 
```

동반 객체 안에서 fromJSON 함수를 정의한 것처럼 `fromJson`호출 가능

클래스 멤버 함수처럼 보이지만 실제로는 X

- 이 기능의 유용성은..?

### 객체 식: 무명 내부 클래스를 다른 방식으로 작성

무명 객체(anonymous object): 자바의 무명 내부 클래스 대신

```kotlin
window.addMouseListener(
    object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            ...
        }
        override fun mouseEntered(e: MouseEvent) {
            ...
        }
    }
)
```

객체 이름 없이 사용

- 객체에 이름이 필요하다면 변수에 무명 객체를 대입

```kotlin
val listener = object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        ...
    }
    override fun mouseEntered(e: MouseEvent) {
        ...
    }
}
```

- 무명 객체는 싱글턴이 아니다 (쓰일 때마다 새로운 인스턴스가 생성됨)
- `final`이 아닌 변수도 객체 식 안에서 사용 가능
    - 객체 식 안에서 변수의 값 변경 가능