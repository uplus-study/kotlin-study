# 4 클래스, 객체, 인터페이스

## 4.1 클래스 계층 정의

### 4.1.1 코틀린 인터페이스

```kotlin
interface Clickable {
  fun click()
}
```

```kotlin
class Button: Clickable {
  override fun click() = println("I was clicked")
}
```

- 콜론으로 `extends`와 `implements`를 대체한다.
- swift랑 똑같다.
- 상위 클래스에 있는 메서드를 `override`할 땐 반드시 `override` 키워드를 써야한다.

```kotlin
interface Clickable {
  fun click()
  fun showOff() = println("I'm clickable!")
}
```

- 디폴트 구현을 가진 interface도 가능하다.
- 같은 이름을 가진 두 interface를 모두 구현하는 클래스를 선언하면 반드시 상위 인터페이스에 정의된 구현을 대체할 오버라이딩 메서드를 직접 제공해야한다.

### 4.1.2 open, final, abstract 변경자: 기본적으로 final

하위 클래스에서 오버라이드하게 의도된 클래스와 메서드가 아니라면 모두 final
어떤 클래스의 상속을 허용하려면 클래스 앞에 open 변경자를 붙여야 한다.
오버라이드를 허용하고 싶은 메서드나 프로퍼티 앞에도 open 변경자를 붙여야 한다.

### 4.1.3 가시성 변경자

코드 기반에 있는 선언에 대한 클래스 외부 접근을 제어한다.

```kotlin
public // 제한 없이 모든 파일에서 접근 가능
internal // 같은 모듈에 있는 파일만 접근
protected // private와 같으나 상속 관계에서 자식 클래스가 접근 가능
private // 다른 파일에서 접근 x
```

### 4.1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

코틀린 중첩 클래스에 아무런 변경자가 붙지 않으면 자바 `static` 중첩 클래스와 같다.
내부 클래스로 변경해 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 `inner` 변경자를 붙여야 한다.

```kotlin
class Outer {
  inner class Inner {
    fun getOuterReference(): Outer = this.@Outer
  }
}
```

### 4.1.5 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

상위 클래스에 `sealed` 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.
sealed 클래스의 하위 클래스를 정의할 때는 반드시 상위 클래스 안에 중첩시켜야 한다.

```kotlin
sealed class Expr {
    class Num(val value: Int): Expr()
    class Sum(val left: Expr, val right: Expr): Expr()
}

fun eval(e: Expr): Int =
    when (e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.left) + eval(e.right)
    }
```

`sealed`를 표시하지 않으면 else문을 반드시 적어야한다.
`sealed`를 표시한 클래스는 자동으로 `open`이 된다.

클래스 외부에 자신을 상속한 클래스를 둘 수 없다.

## 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

코틀린은 주 생성자와 부 생성자를 구분한다.
코틀린은 초기화 블록을 통해 초기화 로직을 추가할 수 있다.

### 4.2.1 클래스 초기화: 주 생성자와 초기화 블록

`constructor` : 주 생성자나 부 생성자 정의를 시작할 때 사용

`init` : 초기화 블록을 시작할 때

```kotlin
class User(_nickname: String) {
  val nickname = _nickname
}

class User(val nickname: String)
```

```kotlin
class Person constructor(value: String) {
  // code
}

// 생성자에 접근 제한자나 다른 옵션이 없다면 생략 가능
class Person(value: String) {
    init {
        // code
    }
}

// init은 없어도 되나 parameter를 쓰고 싶다면 val을 붙여야 한다
class Person(val value: String) {
    fun process() {
        print(value)
    }
}

// secondary constructor
class Person {
    constructor (value: String) {
        // code
    }
}

// default 없어도 init 사용 가능
```

### 4.2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화

생성자 생성 위임

```kotlin
class MyButton: View {
  constructor(ctx: Context): this(ctx, MY_STYLE) {
    // ...
  }

  constuctor(ctx: Context, attr: AttributeSet): super(ctx, attr) {
    // ...
  }
}
```

### 4.2.3 인터페이스에 선언된 프로퍼티 구현

```kotlin
interface User {
    val nickname: String
}

class PrivateUser(override val nickname: String) : User

class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@')
}

class FacebookUser(val accountId: Int) : User  {
    override val nickname = getFacebookName(accountId)
}
```

### 4.2.4 게터와 세터에서 뒷받침하는 필드에 접근

```kotlin
class User(val name: String) {
  var address: String = "unspecified"
    set(value: String) {
      println("""
        Address was chagned for $name:
        "$field" -> "$value".""".trimIndent())
      field = value
    }
}
```

- swift의 `willset`, `didset` 같은 느낌
- 변경 가능 프로퍼티의 게터와 세터 중 한 쪽만 직접 정의해도 된다
- `filed`로 기존 값을 읽고 쓸 수 있다

### 4.2.5 접근자의 가시성 변경

#### 비공개 세터가 있는 프로퍼티 선언

```kotlin
class LengthCounter {
  var counter: Int = 0
    private set

  fun addWord(word: String) {
    counter += word.length
  }
}
```

- 클래스 밖에서 이 프로퍼티의 값을 바꿀 수 없다

## 4.3 컴파일러가 생성한 메서드: 데이터 클래스와 클래스 위임

### 4.3.1 모든 클래스가 정의해야 하는 메서드

##### toString()

##### equals()

##### hashCode()

- equels를 override 할 때, 반드시 hashCode도 override 해야한다.
- JVM 언어에서 equels가 true를 반환하는 두 객체의 hashCode는 같아야한다.

### 4.3.2 데이터 클래스: 모든 클래스가 정의히야 하는 메서드 자동 생성

```kotlin
data class Client(val name: String, val postalCode: Int)
```

##### 추가 자동 생성 메서드

- `copy()`: 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해주는 메서드

### 4.3.3 클래스 위임: by 키워드 사용

## 4.4 object 키워드: 클래스 선언과 인스턴스 생성

static 같은 느낌(?)
앱 전체에 1개만 생성된다.

### 4.4.1 객체 선언: 싱글턴을 쉽게 만들기

```kotlin
object Payroll {
  val allEmployees = arrayListOf<Person>()

  fun calculateSalary() {
    for (person in allEmployee) {
      ...
    }
  }
}
```

- 생성자는 쓸 수 없다. 객체 선언 위치에서 생성자 호출 없이 즉시 만들어진다.

### 4.4.2 동반 객체: 팩토리 메서드와 정적 멤버가 들어갈 장소
