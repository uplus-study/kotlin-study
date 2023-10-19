# 7장 연산자 오버로딩과 기타 관례

## 산술 연산자 오버로딩

```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}
```

```
val p1 = Point(10, 30)
val p2 = Point(30, 40)
println(p1 + p2)   // +로 계산하면 "plus" 함수가 호출된다.
```

plus 함수 앞에 operator 키워드를 붙여야 함. 연산자를 오버로딩하는 함수 앞에는 꼭 operator가 있어야 함


```kotlin
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}
```

연산자를 멤버 함수로 만드는 대신 확장 함수로 정의할 수도 있음

### 오버로딩 가능한 이항 산술 연산자

|식|함수 이름|
|-------|--------|
| a * b | times |
| a / b | div |
| a % b | mod(1.1부터 rem) |
| a + b | plus |
| a - b | minus |



## 결과 타입이 피연산자 타입과 다른 연산자 정의하기

```kotlin
operator fun Char.times(count: Int): String {
    return toString().repeat(count)
}
```
```
println('a' + 3)
aaa
```

Char을 좌항으로 받고 Int를 우항으로 받아서 String을 리턴하는 연산자 오버로딩

## 복합 대입 연산자 오버로딩

```kotlin
val point = Point(1, 2)
point += Point(3, 4)
println(point)
Point(x=4, y=6)
```

+=, -= 등의 연산자와 같은 복합 대입 연산자도 지원

## 단항 연산자 오버로딩

```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

val p = Point(10, 20)
println(-p) // Point(x=-10, y=-10)
```
미리 정해진 이름의 함수를 선언하면서 operator로 표시하면 됨
        
## 동등성 연산자: equal

코틀린은 == 연산자 호출을 equals 메소드 호출로 컴파일하며, != 연산자를 사용하는 식도 마찬가지로 equals로 컴파일 됨. 
a == b 라는 비교를 처리할 때 코틀린은 a가 널인지 판단해서 널이 아닌 경우에만 a.equals(b)를 호출합니다. a가 널이라면 b도 널인 경우에만 결과가 true


식별자 비교 연산자(===)는 자바의 == 연산자와 같습니다. 따라서 ====는 자신의 두 피연사자가 서로 같은 객체를 가르키는지 비교. (참고로 ===는 오버로딩 할 수 없음.)


## 순서 연산자: compareTo

자바에서는 값을 비교해야 할 때 Comparable 인터페이스의 compareTo 메소드를 구현해서 사용했습니다. 코틀린에서도 똑같은 Comparable 인터페이스를 지원하는데, 비교 연산자 (<, >, <=, >=)는 compareTo 호출로 컴파일됨

```kotlin
class Person(
    val firstName: String,
    val lastName: String
): Comparable<Person> {
    override fun compareTo(other: Person): Int {
        return compareValuesBy(this, other, Person::lastName, Person::firstName) //코틀린 표준 라이브러이의 compareValueBy 함수를 사용해 compareTo를 쉽고 간결하게 정의할 수 있음
    }
}
```
```
val p1 = Person("Alice", "Smith")
val p2 = Person("Bob", "Johson")
println(p1 < p2) // false
```

## 인덱스로 원소에 접근: get과 set

인덱스 연산자를 사용해 원소를 읽는 연산은 get 연산자 메소드로 반환되고, 원소를 쓰는 연산은 set 메소드 연산자 메소드로 반환됨

```kotlin
operator fun Point.get(index: Int): Int {
    return when(index) {
        0 -> x
        1 -> y
        else ->
            throw kotlin.IndexOutOfBoundsException("Invalid coordinate $index")
    }
}
```
get이라는 메소드를 만들고 operator 변경자를 붙이기만 하면 p[1] 이라는 식은 위의 정의한 get 메소드로 변환됨


```kotlin
data class MutablePoint(var x: Int, var y: Int)

operator fun MutablePoint.set(index: Int, value: Int) {
    when (index) {
        0 -> x = value
        1 -> y = value
        else ->
            throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}
```
```
val p = MutablePoint(10, 20)
p[1] = 42
println(p)  // MutablePoint(x=10, y=42)
```

위의 코드도 p[1] = 42를 통해서 위에서 정의한 set 메소드가 호출됨



## in 관례
in 연산자는 contains 함수 호출로 변환됨


## rangeTo 관례

start..end는 start.rangeTo(end) 함수 호출로 컴파일 되며 range 함수는 Comparable에 대한 확장 함수임


## for 루프를 위한 iterator 관례

```kotlin
operator fun CharSequence.iterator(): CharIterator
```
```
>>> for (c in "abc")
```

코틀린에서는 iterator 메소드를 확장 함수로 정의할 수 있음



## 프로퍼티 접근자 로직 재활용: 위임 프로퍼티

위임 프로퍼티를 사용하면 값을 뒷받침하는 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 작동하는 프로퍼티를 쉽게 구현할 수 있으며서 접근자 로직을 매번 재구현할 필요도 없음
예를 들어 프로퍼티는 위임을 사용해 자신의 값을 필드가 아니라 데이터베이스 테이블이나 브라우저, 세션, 맵 등에 저장할 수 있음

## 위임 프로퍼티 소개

```kotlin
class Foo {
    val p: type by Delegate()
}
```

p 프로퍼티는 접근자 로직을 다른 객체에게 위임함.  여기서는 Delegate 클래스의 인스턴스를 위임 객체로 사용.

```kotlin
class Foo {
    private val delegate: Delegate()  // 컴파일러가 생성한 도우미 프로퍼티
    val p: Type
    set(value: Type) = delegate.setValue(..., value)  // p 프로퍼티를 위해 컴파일러가 생성한 접근자는 "delegate" get, set 메소드 호출
    get() = delegate.getValue(...)    
}
```
```
val foo = Foo()
val oldValue = foo.p // foo.p 라는 프로퍼티 호출은 내부에서 delegate.getValue() 호출
foo.p = new Value    // foo.p = new Value => 내부에서 delegate.setValue() 호출
```

프로퍼티 위임 관례를 따르는 Delegate 클래스는 getValue, setValue 메소드를 제공해야 하며 코틀린 라이브러리는 프로퍼티 위임을 사용해 프로퍼티 초기화를 지연시켜주는 기능도 존재

## 위임 프로퍼티 사용: by lazy()를 사용한 프로퍼티 초기화 지연

지연 초기화는 객체의 일부분을 초기화하지 않고 남겨뒀다가 실제로 그 부분이 필요한 경우 초기화할 때 흔히 쓰이는 패턴으로 초기화 과정에 자원을 많이 사용하거나 객체를 사용할 때마다 꼭 초기화하지 않아도 되는 프로퍼티에 대해 지연 초기화 패턴을 사용할 수 있음

```kotlin
fun loadEmails(person: Person): List<Email>  {
    println("${person.name}의 이메일을 가져옴")
    return listof(...)
}
```

Person 클래스가 자신이 작성한 이메일의 목록을 제공할 때 불러우는데 시간이 오래 걸려 이메일 프로퍼티의 값을 최초로 사용할 때 단 한번만 데이터베이스에서 가져오고 싶다고 가정

<br>

```kotlin
class Person(val name: String) {
    private val _emails: List<Email>? = null  // 데이터를 저장하고 emails의 위임
    val emails = List<Email>
        get() {
            if (_emails == null) {
                _emails = loadEmails(this)
            }
            return _emails!!
        }
}
```

이메일을 불러오기 전에는 null을 저장하고, 불러온 다음에는 이메일 리스트를 저장하는 emails 프로퍼티를 추가해서 지연 초기화를 구현한 클래스를 보여줌. 
여기서는 뒷밤침하는 프로퍼티라는 기법을 사용하여 다른 프로퍼티인 emails는 _emails라는 프로퍼티에 대한 읽기 연산을 제공.
위임 프로퍼티는 데이터를 저장할 때 쓰이는 뒷받침하는 프로퍼티와 값이 오직 한 번만 초기화됨을 보장하는 게더 로직을 함께 캡슐화해줌

```kotlin
class Person(val name: String) {
    val emails by lazy { loadEmails(this) }
}
```

lazy 함수는 코틀린 관례에 맞는 시그니처의 getValue 메소드가 들어있는 객체를 반환하므로 by 키워드와 함께 사용해 위임 프로퍼티를 만들 수 있고, lazy 함수는 스레드에 안전함

## 위임 프로퍼티 컴파일 규칙

```kotlin
class C {
    var prop: Type by MyDelegate()
}
```

컴파일러는 MyDelegate 클래스의 인스턴스를 감춰진 프로퍼티에 저장하며 그 감춰진 프로퍼티를 <delegate> 라고 부름

## 프로퍼티 값을 맵에 저장

자신의 프로퍼티를 동적으로 정의할 수 있는 객체를 만들 때 위임 프로퍼티를 활용하는 경우가 많고 해당 객체를 확장 가능한 객체라고 함

```kotlin
class Person {
    // 추가 정보
    private val _attributes = hashMapOf<String, String>()
    fun setAttribute(attrName: String, value: String) {
        _attributes [attrName] = value
    }
    
    // 필수 정보
    val name: String
    get() = _attributes["name"]!!
}
```

위의 코드에서 by 키워드 뒤에 맵을 직접 넣으면 쉽게 위임 프로퍼티를 활용하게 변경할 수 있음

```kotlin
class Person {
    private val _attribute = hashMapOf<String, String>()
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }
    
    val name: String by _attribute
}
```
