# 7. 연산자 오버로딩과 기타 관례

어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법을 코틀린에서는 관례(convention)이라고 부른다.

```kt
data class Point(val x: Int, val y: Int)
```

### 7.1 산술 연산자 오버로딩

### 7.1.1 이항 산술 연산 오버로딩

```kt
data class Point(val x: Int, val y: Int) {
  operator fun plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
  }
}
```

함수명을 `plus`로 정의하면 해당 클래스에 대해 `+`연산을 구현할 수 있다.

- 함수 앞에 `operator` 키워드를 붙여야 한다.

```kt
operator fun Point.plus(other: Point): Point {
  return Point(x + other.x, y + other.y)
}
```

- 확장 함수로도 쓸 수 있다.
- 코틀린에서는 프로그래머가 직접 연산자를 만들어 사용할 수 없다.
- 언어에서 미리 정한 연산자만 오버로딩 할 수 있다.
- 관례에 따르기 위해 클래스에서 정의해야 하는 이름이 연산자별로 정해져 있다.

| 식      | 함수 이름        |
| ------- | ---------------- |
| `a * b` | times            |
| `a / b` | div              |
| `a % b` | mod(1.1부터 rem) |
| `a + b` | plus             |
| `a - b` | minus            |

- 연산자 우선순위는 언제나 표준 순자 타입에 대한 연산자 우선순위와 같다.

> 코틀린 연산자는 자바에서 일반 함수로 호출할 수 있다. 자라를 코틀린에서 호출하는 경우에는 함수 이름이 코틀린의 관례에 맞는다면 연산자 식을 사용해 그 함수를 호출할 수 있다.

- 두 피연산자의 타입이 다른 경우 코틀린 연산자가 자동으로 교환 법칙을 지원하지는 않는다.
- 연산자 함수의 반환 타입이 꼭 두 피연산자 중 하나와 일치해야만 하는 것도 아니다.
- `operator` 함수도 오버로딩할 수 있다.

### 7.1.2 복합 대입 연산자 오버로딩

`plus`와 같은 연산자를 오버로딩하면 코틀린은 + 연산자뿐 아니라 그와 관련 있는 연산자인 `+=`도 자동으로 함께 지원한다.

경우에 따라 `+=` 연산이 참조를 바꾸는 것보다 원래 객체의 내부 상태를 변경하게 만들고 싶을 때는 반환 타입이 `Unit`인 `plusAssign`을 정의하면 복합대입 연산자에 이를 사용한다.

```kt
operator fun <T> MutableCollection<T>.plusAssign(element: T) {
  this.add(element)
}
```

코틀린 표준 라이브러리는 컬렉션에 대해 두 접근 방법을 함께 제공한다.
읽기 전용 컬렉션에서는 `+=`와 `-=` 는 변경을 적용한 복사본을 반환한다.

### 7.1.3 단항 연산자 오버로딩

절차는 이항 연산자와 같다.

| 식         | 함수 이름  |
| ---------- | ---------- |
| `+a`       | unaryPlus  |
| `-a`       | unaryMinus |
| `!a`       | not        |
| `++a, a++` | inc        |
| `--a, a--` | dec        |

전, 후위 연산자는 일반적인 값에 대한 전, 후위 연산자와 같은 의미를 제공한다.

## 7.2 비교 연산자 오버로딩

`equals`나 `compareTo`를 호출해야하는 자바와 달리 코틀린에서는 모든 객체에서 `==` 비교 연산자를 직접 사용할 수 있다.

### 7.2.1 동등성 연산자: equals

data class로 정의하면 equeals가 자동으로 생성된다.

직접 구현한다면 아래처럼 구현

```kt
class Point(val x: Int, val y: Int) {
  override fun equals(obj: Any?): Boolean {
    if (obj === this) return true
    if (obj !is Point) return false
    return obj.x == x && obj.y == y
  }
}
```

- `===`는 오버로딩 할 수 없다.

- 다른 연산자 오버로딩 관례와 달리 `equals`는 `Any`에 정의된 메서드이므로 `override`가 필요하다

### 7.2.2 순서 연산자: compareTo

정렬이나 최댓값, 최솟값 등 값을 비교해야 하는 알고리즘에서 사용할 클래스는 `Comparable` 인터페이스를 구현해야 한다.

```kt
class Person(private val name: String, val age: Int): Comparable<Person> {
    override fun compareTo(other: Person): Int {
        return compareValuesBy(this, other, Person::name, Person::age)
    }
}
```

## 7.3 컬렉션과 범위에 대해 쓸 수 있는 관례

### 7.3.1 인덱스로 원소 접근: get과 set

`mutableMap[key] = newValue`

코틀린에서는 인덱스 연산자도 관례를 따른다.

- 원소를 읽는 연산: `get`
- 원소를 쓰는 연산: `set`

```kt
operator fun Point.get(index: Int): Int {
    return when(index) {
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

operator fun Point.set(index: Int, value: Int) {
    when (index) {
        0 -> x = value
        1 -> y = value
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}
```

- `get` 메서드의 파라미터는 Int가 아니어도 된다.
- `set`이 받는 마지막 파라미터 값은 대입문의 우항에 들어가고, 나머지는 인덱스 연산자에 들어간다.
  - `x[a, b] = c -> x.set(a, b, c)`

### 7.3.2 in 관례

객체가 컬렉션에 들어있는지 검사하며, `contains`와 대응한다.

### 7.3.3 rangeTo 관례

- `rangeTo` 함수는 범위를 반환한다.
- 어떤 클래스가 `Comparable` 인터페이스를 구현하면 `rangeTo`를 정의할 필요가 없다.
- `rangeTo`가 `Comparable`에 대한 확장 함수로 구현되어 있다.

### 7.3.4 for 루프를 위한 iterator 관례

- `iterator` 메서드를 통해 이터레이터를 구현할 수 있다

```kt
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> = object: Iterator<LocalDate> {
  var current = start

  override fun hasNext() = current <= endInclusive

  override fun next() = current.apply {
    current = plusDays(1)
  }
}
```

## 7.4 구조 분해 선언과 component 함수

```kt
val p = Point(10, 20)
val (x, y) = p
```

내부에서 구조 분해 선언은 관례를 사용한다.
각 변수의 위치에 따라 `componentN`이라는 함수를 호출한다.

```kt
class Point (val x: Int, val y: Int) {
  operator fun component1() = x
  operator fun component2() = y
}
```

코틀린 표준 라이브러리에서는 맨 앞의 다섯 원소에 대한 `componentN`을 제공한다.

## 7.5 프로퍼티 접근자 로직 재활용: 위임 프로퍼티

위임 프로퍼티를 사용하면 값을 뒷받침하는 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 작동하는 프로퍼티를 쉽게 구현할 수 있다(?).

프로퍼티는 위임을 사용해 자신의 값을 필드가 아니라 데이터베이스 테이블이나 브라우저 세션, 맵 등에 저장할 수 있다.

### 7.5.1 위임 프로퍼티 소개

일반적인 문법

```kt
class Foo {
  var p: Type by Delegate()
}
```

- p 프로퍼티는 접근자 로직을 다른 객체에게 위임
- by 뒤에 있는 식을 계산에서 위임에 쓰일 객체를 얻는다.
- 프로퍼티 위임 객체가 따라야 하는 관례를 따르는 모든 객체를 위임에 사용할 수 있다.

```kt
class Foo {
  private val delegate = Delegate()
  var p: Type
  set(value: Type) = delegate.setValue(..., value)
  get() = delegate.getValue(...)
}
```

- 컴파일러는 숨겨진 도우미 프로퍼티를 만들고 그 프로퍼티를 위임 객체의 인스턴스로 초기화한다.
- 프로퍼티의 위임 관례를 따르는 `Delegate` 클래스는 `getValue`와 `setValue` 메서드를 제공해야 한다.

```kt
class Delegate {
  operator fun getValue(...) {...}
  operator fun setValue(..., value: Type) {...}
}
```

### 7.5.2 위임 프로퍼티 사용: by lazy()를 사용한 프로퍼티 초기화 지연

객체를 사용할 때마다 꼭 초기화하지 않아도 되는 프로퍼티에 대해 지연 초기화 패턴을 사용할 수 있다.

```kt
class Person(val name: String) {
    private var _emails: List<Email>? = null
    var emails: List<Email>
        get() {
            // 최초 접근 시 이메일을 가져오도록 한다.
            if (_emails == null) {
                _emails = loadEmails(this)
            }

            return _emails!!
        }
}
```

`_emails`를 뒷받침하는 프로퍼티로 사용하였다.

지연 초기화를 위임 프로퍼티로 구현하면

```kt
class Person(val name: String) {
    val emails by lazy { loadEmails(this) }
}
```

처럼 단순해진다.

- `lazy` 함수는 기본적으로 스레드 안전하다.
- 하지만 필요에 따라 동기화에 사용할 락을 `lazy` 함수에 전달할 수 있고, 다중 스레드 환경에서 사용하지 않을 프로퍼티를 위해 `lazy` 함수가 동기화를 하지 못하게 막을 수도 있다.

### 7.5.3 위임 프로퍼티 구현

```kt
open class PropertyChangeAware {
    protected val changeSupport = PropertyChangeSupport(this)

    fun addPropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.addPropertyChangeListener(listener)
    }

    fun removePropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.removePropertyChangeListener(listener)
    }
}

class Person(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    var age: Int = age
        set(newValue) {
            val oldValue = field
            field = newValue
            changeSupport.firePropertyChange("age", oldValue, newValue)
        }

    var salary: Int = salary
        set(newValue) {
            val oldValue = field
            field = newValue
            changeSupport.firePropertyChange("salary", oldValue, newValue)
        }
}
```

- 각 프로퍼티에 대해 `field` 키워드로 프로퍼티 변경을 통지하는 코드를 작성하면 중복이 많아진다.

```kt
class ObservableProperty(val propName: String, var propValue: Int, val changeSupport: PropertyChangeSupport) {
    fun getValue(): Int = propValue

    fun setValue(newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(propName, oldValue, newValue)
    }
}

class Person(val name: String, age: Int, salary: Int) : PropertyChangeAware() {

    val _age = ObservableProperty("age", age, changeSupport)
    var age: Int
        get() = _age.getValue()
        set(value) { _age.setValue(value) }

    var _salary = ObservableProperty("salary", salary, changeSupport)
    var salary: Int
        get() = _salary.getValue()
        set(value) { _salary.setValue(value) }
}
```

- `ObservableProperty`를 각각의 프로퍼티마다 만들어 게터와 세터에서 작업을 위임하면 중복되는 부분은 줄지만 여전히 준비코드가 길다.

```kt
class ObservableProperty(var propValue: Int, val changeSupport: PropertyChangeSupport) {
    operator fun getValue(p: Person, prop: KProperty<*>): Int = propValue

    operator fun setValue(p: Person, prop: KProperty<*>, newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(prop.name, oldValue, newValue)
    }
}

class Person(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    var age: Int by ObservableProperty(age, changeSupport)
    var salary: Int by ObservableProperty(salary, changeSupport)
}
```

- `by` 키워드를 사용해 위임 객체를 지정하면 코틀린 컴파일러가 자동으로 처리해준다.

```kt
class Person(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    private val observer = {
        prop: KProperty<*>, oldValue: Int, newValue: Int -> changeSupport.firePropertyChange(prop.name, oldValue, newValue)
    }

    var age: Int by Delegates.observable(age, observer)
    var salary: Int by Delegates.observable(salary, observer)
}
```

- `ObservableProperty`도 코틀린 표준 라이브러리를 사용할 수 있다
- `by`의 우항이 꼭 새 인스턴스를 만들 필요는 없다.

### 7.5.4 위임 프로퍼티 컴파일 규칙

- 컴파일러는 MyDelegate 클래스의 인스턴스를 감춰진 프로퍼티에 저장하며, 그 감춰진 프로퍼티를 `<delegate>`라는 이름으로 부른다.
- 컴파일러는 프로퍼티를 표현하기 위해 `KProperty` 타입의 객체를 사용한다.
- 컴파일러는 모든 프로퍼티 접근자 안에 `getValue`와 `setValue` 호출 코드를 생성해준다.

### 7.5.4 프로퍼티 값을 맵에 저장

`by` 키워드 뒤에 맵을 직접 넣어 값을 맵에 저장하는 위임 프로퍼티를 사용할 수 있다.

```kt
class Person {
    private val _attributes = hashMapOf<String, String>()

    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }

    val name: String by _attributes
}
```

### 7.5.6 프레임워크에서 위임 프로퍼티 활용
