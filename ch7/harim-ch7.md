# 연산자 오버로딩과 기타 관례
관례: 어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법
- 자바 -> 언어 기능을 타입에 의존
- 코틀린 -> 언어 기능을 함수 이름을 통한 관례에 의존
  - 기존 자바 클래스를 코틀린 언어에 적용하기 위함
  - 확장 함수를 사용하면 기존 클래스에 새로운 메서드 추가 가능

### 산술 연산자 오버로딩
- **이항 산술 연산 오버로딩**
  ```Kotlin
  data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
  }

  val p1 = Point(10, 20)
  val p2 = Point(30, 40)
  println(p1 + p2) //Point(x=40, y=60)
  ```
  - plus 함수 앞에 `operator` 키워드를 붙여야 함
  - `operator` 변경자를 추가해 plus 함수 선언하고 나면 + 기호로 두 Point 객체 더할 수 있음
  - 연산자를 확장 함수로 구현하는게 일반적인 패턴

    ```Kotlin
    operator fun Pointer.plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
    ```
  - 코틀린에서는 프로그래머가 직접 연산자 만들어 사용할 수 없고 미리 정해둔 연산자만 오버로딩 가능
  - 관례에 따르기 위해 클래스에서 정의해야 하는 이름이 연산자별로 정해져있음
    |식|함수이름|
    |---|---|
    |a * b|times|
    |a / b|div|
    |a % b|mod(1.1부터 rem)|
    |a + b|plus|
    |a - b|minus|
  - 연산자 우선순위는 표준 숫자 타입에 대한 연산자 우선순위와 같음
  - 연산자 정의할 때 두 피연산자가 같은 타입일 필요 없음, 반환 타입도 마찬가지
  - 코틀린 연산자는 자동으로 교환 법칙을 지원하지 않음
  
  > 비트 연산자에 대해 특별한 연산자 함수 사용 X
  > 
  > 따라서 커스텀 타입에서 비트 연산자 정의 X, 대신 중위 연산자 표기법을 지원하는 일반 함수를 사용해 비트 연산 수행
  > 
  > shl(왼쪽 시프트), shr(오른쪽 시프트), ushr(오른쪽 시프트, 0으로 부호 비트 설정), and(비트 곱), or(비트 합), xor(비트 배타 합), inv(비트 반전)

- **복합 대입 연산자 오버로딩**
  - 복합 대입 연산자도 오버로딩 지원(+=, -=)
    ```Kotlin
    var point = Point(1, 2)
    point += Point(3, 4) // point = point + Point(3, 4)
    ```
  - 경우에 따라 += 연산으로 원래 객체의 내부 상태를 변경하게 만들고 싶을 때 있음
    - 변경 가능한 컬렉션에 원소 추가하는 경우
    ```Kotlin
    val numbers = ArrayList<Int>()
    numbers += 42
    println(numbers[0]) //42

    operator fun <T> MutableCollection<T>.plusAssign(element: T) {
        this.add(element)
    }
    ```
    - 반환 타입이 Unit 인 plusAssign 함수 정의하면 코틀린은 += 연산자에 그 함수 사용함
  - +=를 plus와 plusAssign 양쪽에 컴파일 할 수 있으나 동시에 정의하지 않는 것이 좋음
  - +, -는 항상 새로운 컬렉션을 반환, +=, -=는 항상 변경 가능한 컬렉션에 작용해 메모리에 있는 객체 상태를 변화 시킴

- **단항 연산자 오버로딩**
  ```Kotlin
  operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
  }
  ```
  - 단한 연산자 오버로딩하기 위해 사용하는 함수는 인자 X
    |식|함수이름|
    |---|---|
    |+a|unaryPlus|
    |-a|unaryMinus|
    |!a|not|
    |++a, a++|inc|
    |--a, a--|dec|

### 비교 연산자 오버로딩
- **동등성 연산자 equals**
  - 코틀린에서는 == 연산자 호출을 equals 메서드 호출로 컴파일함
    - 내부에서 인자가 널인지도 검사함
  - 식별자 비교 연산자(===)는 자바 == 연산자와 같음
    - 따라서 ===는 자신의 두 피연산자가 서로 같은 객체를 가리키는지 비교
    - === 오버로딩 X
  - equals 함수는 다른 연산자 오버로딩 관례와 달리 Any에 정의된 메서드이므로 `override` 필요
  - Any에서 상속받은 equals가 확장 함수보다 우선순위가 높기 때문에 equals를 확장 함수로 정의 X

- **순서 연산자 compareTo**
  - 자바에서 정렬, 최댓값/최솟값 등 비교해야 하는 알고리즘에 사용할 클래스에서 Comparable 인터페이스를 구현해야 함
  - Comparable의 compareTo 메서드는 한 객체와 다른 객체의 크기 비교해 정수로 나타내줌
  - 코틀린에서는 >, <, =>, <= 이런 비교 연산자는 compareTo 호출로 컴파일됨 -> 반환 Int
  - equals와 마찬가지로 오버라이딩 함수에 operator 붙일 필요 없음
    ```Kotlin
    class Person(
        val firstName: String, val lastName: String
    ): Comparable<Person> {
        override fun compareTo(other: Person): Int {
            return compareValuesBy(this, other, Person::lastName, Person::firstName)
        }
    }
    ```
    - compareValuesBy함수: 두 객체와 여러 비교 함수를 인자로 받음, 첫 번째 비교 함수에 두 객체 넘겨 같지 않다면 즉시 반환하고 객체가 같다면 두 번째 비교 함수로 비교
  
### 컬렉션과 범위에 대해 쓸 수 있는 관례
- **인덱스로 원소에 접근**
  - 인덱스 연산자를 사용해 원소를 읽는 연산은 get 연산자 메서드로 변환되고, 원소 쓰는 연산은 set 연산자 메서드로 변환됨 `mutableMap[key] = newValue` 
  - get 메서드 파라미터로 Int 아닌 다른 타입 사용 가능 
    - 맵의 경우 맵의 키 타입과 같은 임의의 타입 O
  - 여러 파라미터를 사용하는 get 정의 가능
    - 2차원 행렬이나 배열 표현, ..

- **in 관례**
  - 객체가 컬렉션에 들어있는지 검사 => contains
  - 10..20 은 닫힌 범위 (10~20)
  - 10 until 20 반열린 구간 (10~19)

- **rangeTo**
  - 범위 만드려면 .. 구문 사용
  - .. 연산자는 rangeTo 함수 간략하게 표현하는 방법
  - 해당 연산자를 아무 클래스에나 정의할 수 있다
    - Comparable 인터페이스 구현하면 rangeTo 정의할 필요 X
  - 다른 산술 연산자보다 우선순위 낮음

- for 루프 위한 iterator 관례
  - 코틀린의 for 루프는 in 연산자 사용
  - `for(x in list) { ... }` 은 list.iterator()를 호출해서 이터레이터를 얻은 후 자바와 마찬가지로 이터레이터에 대해 hasNext, next 호출 반복하는 식으로 변환됨
  - iterator 메서드를 확장 함수로 정의 가능 => 일반 자바 문자열에 대해 for 루프 가능
  
### 구조 분해 선언과 component 함수
- 구조 분해 선언
  - 구조 분해 사용하면 복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화 가능
    ```Kotlin
    val p = Point(10, 20)
    val (x, y) = p
    print(x) //10
    print(y) //20
    ```
  - 일반 변수 선언과 비슷하나 좌변에 여러 변수를 괄호로 묶음
  - 구조 분해 선언의 각 변수 초기화하기 위해서 componentN 함수 호출
    - N은 구조 분해 선언에 있는 변수 위치에 따라 붙는 번호
    - `val(a, b) = p` => `val a = p.component1(), val b = p.component2()` 
  - data 클래스의 주 생성자에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 componentN 함수 만듦
  - 여러 값을 반환할 때 유용
  - 배열, 컬렉션도 componentN 함수 있음
    ```Kotlin
    data class NameComponents(
        val name: String,
        val extension: String)
    
    fun splitFilename(fullName: String): NameComponents {
        val (name, extension) = fullName.split(('.', limit = 2))
        return NameComponents(name, extension)
    }
    ```
- 구조 분해 선언과 루프
  - 루프 안에서도 구조 분해 선언 사용 가능
    ```Kotlin
    fun printEntries(map: Map<String, String>) {
        for((key, value) in map) {
            println("$key -> $value")
        }
    }
    ```

### 위임 프로퍼티
> 위임: 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 작업을 처리하게 맡기는 디자인 패턴

```Kotlin
class Foo {
    var p: Type by Delegate()
}
```
- p 프로퍼티는 접근자 로직을 다른 객체에 위임함
  - `Delegate` 클래스 인스턴스를 위임 객체로 사용
  - 프로퍼티 위임 객체가 따라야 하는 관례 따르는 모든 객체를 위임에 사용할 수 있음
    ```Kotlin
    class Foo {
        private val deletgate = Delegate() //컴파일러가 생성한 도우미 프로퍼티
        var p: Type
        set(value: Type) = delegate.setValue(..., value)
        get() = delegate.getValue(...)
    }
    ```
    - 프로퍼티 위임 관례 따르는 Delegate 클래스는 getValue, setValue 메서드를 제공해야 함
    - p 프로퍼티에 접근할 때 일반 프로퍼티처럼 쓸 수 있음
    - 하지만 실제로 p의 게터나 세터는 Delegate 타입의 위임 프로퍼티 객체에 있는 메서드를 호출

### by lazy()를 사용한 프로퍼티 초기화 지연
- 지연 초기화: 객체의 일부분을 초기화하지 않고 남겨뒀다가 실제로 그 부분의 값이 필요한 경우 초기화할 때 쓰는 패턴
```Kotlin
class Person(val name: String) {
    private var _emails: List<Email>? = null
    val emails: List<Email>
        get() {
            if(_emails == null) {
                _emails = loadEmails(this)
            }
            return _emails!!
        }
}
```
- 뒷받침하는 프로퍼티 기법 사용함
  - `_emails` 프로퍼티는 값을 저장하고, `emails`는 `_emails` 프로퍼티에 대한 읽기 연산 제공
- 지연 초기화해야 하는 프로퍼티가 많아지면 코드가 복잡해짐, 또한 이 구현은 스레드 안전하지 않음 => 위임 프로퍼티 사용!
- 위임 프로퍼티는 데이터를 저장할 때 쓰이는 뒷받침하는 프로퍼티와 값이 오직 한 번만 초기화됨을 보장하는 게터 로직 함께 캡슐화함
  ```Kotlin
  class Person(val name: String) {
    val emails by lazy { loadEmails(this) }
  }
  ```
  - lazy 함수는 코틀린 관례에 맞는 getValue 메서드가 들어있는 객체를 반환함
  - lazy 인자는 값을 초기화할 때 호출할 람다
- 위임 객체를 감줘치 프로퍼티에 저장하고 주 객체의 프로퍼티를 읽거나 쓸 때마다 위임 객체의 getValue, setValue를 호출해줌

### 위임 프로퍼티 컴파일 규칙
```Kotlin
class C {
    var prop: Type by MyDelegate()
}
val c = C()
```
- 컴파일러는 MyDelegate 클래스의 인스턴스를 감춰진 프로퍼티에 저장하고 감춰진 프로퍼티를 `<delegate>` 라는 이름으로 부름
- 컴파일러는 프로퍼티 표현하기 위해 KProperty 타입의 객체 사용 -> `<property>` 라고 부름

```Kotlin
//컴파일러가 생성하는 코드
class C {
    private val <delegate> = MyDelegate()
    var prop: Type
        get() = <delegate>.getValue(this, <property>)
        set(value: Type) = <delegate>.setValue(this, <property>, value)
}
```

### 위임 프로퍼티 활용
- 확장 가능한 객체
  - 자신의 프로퍼티를 동적으로 정의할 수 있는 객체
  - 연락처별로 임의 정보 저장할 수 있게 허용하는 경우
    - 특별히 처리해야 하는 일부 필수 정보가 있고 사람마다 달라질 수 있는 추가 정보가 있음
    - 정보를 모두 맵에 저장하되, 맵을 통해 처리하는 프로퍼티를 통해 필수 정보를 제공
    ```Kotlin
    class Person {
        //추가정보
        Private val _attributes = hashMapOf<String, String>()
        fun setAttribute(attrName: String. value: String) {
            _attributes[attrName] = value
        }
        //필수정보
        val name: String
        get() = _attributes["name"]!!
    }

    //위 코드를 위임 프로퍼티 활용하도록 변경
    class Person {
        private val _attributes = hashMapOf<String, String>()
        fun setAttribute(attrName: String, value: String) {
            _attributes[attrName] = value
        }
        
        val name: String by _attributes //위임 프로퍼티로 맵 사용
    }
    ```
    - 표준 라이브러리가 Map, MutableMap 인터페이스에 대해 getValue, setValue 확장 함수 제공하기 때문에 가능
  - 프레임워크에서 활용
    ```Kotlin
    object Users: IdTable() { //DB 테이블
        val name = varchar("name", length = 50).index() //컬럼
        val age = integer("age") //컬럼
    }

    class User(id: EntityID): Entity(id) { //엔티티
        var name: String by Users.name
        var age: Int by Users.age
    }
    ```