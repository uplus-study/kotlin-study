# 7장. 연산자 오버로딩 기타 관례

## 산술 연산자 오버로딩

### 이항 산술 연산 오버로딩
- `operator`와 지정한 함수 이름을 사용해서 연산자 오버로딩

```Kotlin
data class Point(val x: Int, val y: Int) {
  operator fun plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
  }
}
```

- 확장 함수도 가능
- 여러 타입에 대해 연산자 오버로딩 가능
```Kotlin
operator fun Point.times(scale: Double): Point { }
```

### 복합 대입 연산자 오버로딩
- plus와 minus와 같은 연산자를 오버로딩하면, +=, ‐=의 복합 대입 연산자 자동 지원
- 리턴 타입이 Unit인 plusAssign 함수가 존재하면 += 연산자에 그 함수 사용
```Kotlin
operator fun <T> MutableCollection<T>.plusAssign(element: T) {
  this.add(element)
}
val numbers = ArrayList<Int>()
numbers += 42
```
- +=에 대해 plus와 plusAssign을 둘 다 존재하면 컴파일 오류
  - 변수를 val로 하면 plus 대신 plusAssign 사용
  - 일관된 사용을 위해 두 연산을 동시에 정의하지 말 것

### 단항 연산자 오버로딩

```Kotlin
operator fun BigDecimal.inc() = this + BigDecimal.ONE
var bd = BigDecimal.ZERO
println(bd++)
```

## 비교 연산자 오버로딩
모든 객체에 대해 비교 연산을 수행

### 동등성 연산자 : `equals`
```Kotlin
a == b : a?.equals﴾b﴿ ?: ﴾b == null﴿
```
- 확장 함수 불가능

### 순서 연산자 : `compareTo`
- <, <=
, >, >= 연산자를 Comparable.compareTo 호출로 컴파일
```Kotlin
a >= b : a.compareTo﴾b﴿ >= 0
```

## 컬렉션과 범위에 대해 쓸 수 있는 관례

### get
x[a,b] : x.get﴾a,b﴿

### set
x[a,b]=c : x.set﴾a,b,c﴿

### in
a in c : c.contains﴾a﴿

### rangeTo
`..`으로 범위 생성
```Kotlin
operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```

### for 루프와 iterator
`iterator()` 함수는 for 루프에서 사용 가능

### 구조 분해 선언과 component 함수
- 복합적인 값을 분해해서 여러 변수를 한꺼번에 초기화 가능

```Kotlin
val (a, b) = p // 해당 식은 아래와 같이 컴파일 됨.

val a1 = p.component1()
val b1 = p.component2()
```
- data 클래스의 주 생성자에 있는 프로퍼티에 대해 컴파일러가 자동으로 `componentN` 함수를 생성해줌.

### 구조 분해 선언과 루프
- 코틀린에서는 map을 직접 iteration 할 수 있음.
```Kotlin
for ((key, value) in map) {
  println("$key -> $value")
}
```

### 위임 프로퍼티
프로퍼티 접근을 다른 객체에 위임

```Kotlin
class Foo {
  var p: Type by Delegate()
  // p 프로퍼티에 대한 접근을 Delegate 객체에 위임
}
```

### 위임 프로퍼티 사용: by lazy()를 사용한 프로퍼티 초기화 지연

```Kotlin
class Person(val name: String) {
    private var _emails: List<Email>? = null
    val emails: List<Email>
       get() {
           if (_emails == null) {
               _emails = loadEmails(this)
}
           return _emails!!
       }
}

// 프로퍼티 초기화 지연
class Person(val name: String) {
  val emails by lazy { loadEmails(this) }
}

// lazy 함수는 코틀린 관례에 맞게 getValue 메서드가 들어있는 객체를 반환함.
public inline operator fun <T> Lazy<T>.getValue(
  thisRef: Any?, property: KProperty<*>): T = value
```

### 프로퍼티 값을 맵에 저장
- 코틀린에서 맵이 위임 객체를 위해 getValue, setValue 확장 함수를 제공함.
```Kotlin
class Person {
  private val _attributes = hashMapOf<String, String>()
  val name: String by _attributes
}
```
