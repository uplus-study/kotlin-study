# 11장 DSL 만들기

## 11.1 API에서 DSL로


| 일반 구문                              | 간결한 구문                                    | 사용한 언어 특성      |
|------------------------------------|-------------------------------------------|----------------|
| `StringUtil.capitalize(s)`         | `s.capitalize()`                          | 확장 함수          |
| `1.to("one")`                      | `1 to "one"`                              | 중위 호출          |
| `set.add(2)`                       | `set += 2`                                | 연산자 오버로딩       |
| `map.get("key")`                   | `map["key"]`                              | get 메서드에 대한 관례 |
| `file.use({ f -> f.read() } )`     | `file.use { it.read() }`                  | 람다를 괄호 밖으로 빼내기 |
| `sb.append("yes") sb.append("no")` | `with (sb) { append("yes") append("no")}` | 수신 객체 지정 람다    |

### 영역 특화 언어라는 개념

DSL: Domain Specific Language, 영역 특화 언어 vs 범용 프로그래밍 언어

- DSL은 범용 프로그래밍 언어와 달리 더 **선언적**
    - 범용 프로그래밍 언어: 명령적
    - 선언적: 원하는 결과를 기술

단점: DSL을 범용 언어로 만든 호스트 애플리케이션과 함께 조합하기 어렵다

### 내부 DSL

범용 언어로 작성된 프로그램의 일부, 범용 언어와 동일한 문법 사용

ex) 외부 DSL: SQL, 내부 DSL: 익스포즈드 프레임워크

- 내부 DSL: 코틀린 코드, SQL 질의를 위한 것이지만 코틀린 라이브러리로 구현

### DSL의 구조

DSL에만 존재하는 특징 vs 일반 API: 구조 / 문법

명령-질의 API: 함수 호출 시퀀스에는 구조가 없음, 호출 간 맥락x

DSL: DSL 문법에 의해 정해지는 구조에 속한다

- 장점: 같은 문맥을 함소 호출마다 반복하지 않고 재사용 가능

### 내부 DSL로 HTML 만들기

라이브러리: `kotlinx.html` (https://github.com/Kotlin/kotlinx.html)

```kotlin
fun createSimpleTable() = createHTML().table {
    tr {
        td { +"cell" }
    }
}
```

```html

<table>
    <tr>
        <td>cell</td>
    </tr>
```

- 타입 안전성 보장: 코틀린 코드로 이루어짐, 컴파일러가 타입 검사
    - 코틀린 코드를 원하는 대로 사용 가능

```kotlin
fun createAnotherTable() = createHTML().table {
    val numbers = mapOf(1 to "one", 2 to "two")
    for ((num, string) in numbers) {
        tr {
            td { +"$num" }
            td { +string }
        }
    }
}
```

## 11.2 구조화된 API 구축: DSL에서 수신 객체 지정 DSL 사용

### 수신 객체 지정 람다와 확장 함수 타입

`buildString` 함수: `StringBuilder` 객체에 여러 내용 추가 가능

**일반 람다를 받는 함수**

```kotlin
fun buildString(
    builderAction: (StringBuilder) -> Unit
): String {
    val sb = StringBuilder()
    builderAction(sb) // 람다 인자로 StringBuilder를 넘긴다
    return sb.toString()
}

val s = buildString {
    it.append("Hello, ") // it은 StringBuilder 인스턴스
    it.append("World!")
}
```

람다 본문에서 매번 it을 참조해야 한다 -> 수신 객체 지정 람다로 바꿈

**수신 객체 지정 람다를 받는 함수**

```kotlin
fun buildString(
    builderAction: StringBuilder.() -> Unit
): String {
    val sb = StringBuilder()
    sb.builderAction() // StringBuilder 인스턴스를 람다의 수신 객체로 넘긴다
    return sb.toString()
}

val s = buildString {
    this.append("Hello, ") // this는 StringBuilder 인스턴스
    append("World!") // this를 생략해도 된다
}
```

- it 사용하지 않아도 된다
    - `append()`만 사용
- 확장 함수 타입 사용
    - `StringBuilder.() -> Unit`
    - `StringBuilder`를 수신 객체로 받는 함수 타입

확장 함수 타입의 변수 정의 -> 확장 함수처럼 호출, 인자로 넘길 수 있다

표준 라이브러리의 `buildString` 구현

```kotlin
fun buildString(
    builderAction: (StringBuilder) -> Unit
): String = StringBuilder().apply(builderAction).toString()
```

`apply`: 인자로 받은 람다/함수를 호출하면서 자신의 수신 객체를 람다/함수의 묵시적 수신 객체로 사용

```kotlin
inline fun <T> T.apply(block: T.() -> Unit): T {
    block(); return this
} // apply의 수신 객체를 수신 객체로 지정해 람다 호출

inline fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block() // 람다를 호출해 얻은 결과를 반환
```

apply vs with

- apply: 수신 객체의 메서드처럼 불린다, 수신 객체를 묵시적 인자로 받는다, 수신 객체 다시 반환
- with: 수신 객체를 첫 번째 파라미터로 받음, 람다 호출 결과를 반환

### 수신 객체 지정 람다를 HTML 빌더 안에서 사용

```kotlin
fun createSimpleTable() = createHTML().table {
    tr {
        td { +"cell" }
    }
}
```

- `tr`과 `td`는 `table`의 확장 함수
    - `td`는 `tr` 안에서만 접근 가능

```kotlin
open class Tag
class TABLE : Tag {
    fun tr(init: TR.() -> Unit) { /*...*/
    }
}
class TR : Tag {
    fun td(init: TD.() -> Unit) { /*...*/
    }
}
class TD : Tag {
    fun text(s: String) { /*...*/
    }
}
```

수신 객체 지정 람다가 다른 수신 객체 지정 람다 안에 들어가면 내부 람다에서 외부 람다에 정의된 수신 객체 사용

```kotlin
fun createSimpleTable() = createHTML().table {
    (this@table).tr { // this@table 타입: TABLE
        (this@tr).td { +"cell" } // 묵시적 수신 객체로 this@tr 사용
    }
}
```

```kotlin
open class Tag(val name: String) {
    private val children = mutableListOf<Tag>() // 중첩 태그 저장
    protected fun <T : Tag> doInit(child: T, init: T.() -> Unit) {
        child.init() // 자식 태그 초기화
        children.add(child) // 자식 태그 참조 저장
    }
    override fun toString() =
        "<$name>${children.joinToString("")}</$name>" // 태그 이름과 자식 태그 문자열 반환
}
```

각 태그는 자기 이름을 태그 안에 넣고, 자식 태그를 재귀적으로 문자열로 바꿔서 호출

자신이 새로 생성한 태그를 부모 태그가 가진 자식 목록에 추가 -> 동적으로 태그 생성 가능

### 코틀린 빌더: 추상화와 재사용을 가능하게 하는 도구

반복되는 내부 DSL 코드를 새 함수로 묶어서 사용 가능

## 11.3 invoke 관례를 사용한 더 유연한 블록 중첩

함수처럼 호출할 수 있는 객체를 만드는 클래스 정의

### invoke 관례: 함수처럼 호출할 수 있는 객체

```kotlin
class Greeter(val greeting: String) {
    operator fun invoke(name: String) {
        println("$greeting, $name!")
    }
}

val bavarianGreeter = Greeter("Servus")
bavarianGreeter("Dmitry") // Servus, Dmitry!
```

operator 재정의

### invoke 관례와 함수형 타입

인라인 제외 모든 람다: 함수형 인터페이스를 구현하는 클래스로 컴파일됨

-> 그 인터페이스 이름이 가리키는 개수만큼 파라미터를 받는 invoke 메서드 포함

복잡한 람다를 여러 메서드로 분리 + 분리 전 람다처럼 외부에서 호출하는 객체를 만들 수 있다

+ 타입 파라미터를 받는 함수에게 그 객체를 전달 가능

**함수 타입을 확장하면서 invoke()를 오버라이딩하기**

```kotlin
data class Issue(
    val id: String, val project: String, val type: String,
    val priority: String, val description: String
)

class ImportantIssuesPredicate(val project: String) : (Issue) -> Boolean { // 함수 타입을 부모 클래스로
    override fun invoke(issue: Issue): Boolean { // invoke 메서드 구현
        return issue.project == project && issue.isImportant()
    }
    private fun Issue.isImportant(): Boolean {
        return type == "Bug" &&
                (priority == "Major" || priority == "Critical")
    }
}

val i1 = Issue("IDEA-154446", "IDEA", "Bug", "Major", "Save settings failed")
val i2 = Issue(
    "KT-12183",
    "Kotlin",
    "Feature",
    "Normal",
    "Intention: convert several calls on the same receiver to with/apply"
)
val predicate = ImportantIssuesPredicate("IDEA") // 객체 생성
for (issue in listOf(i1, i2).filter(predicate)) {
    술어를 filter () 에 넘긴다
    println(issue.id) // IDEA-154446
}
```

술어의 로직이 복잡 -> 메서드를 나누고 이름을 붙여 명확히 함

- 람다를 함수 타입 인터페이스를 구현하는 클래스로 변환 + invoke 메서드를 오버라이드

### DSL의 invoke 관례: 그레이들에서 의존관계 정의

```kotlin
dependencies {
    compile("juit:junit:4.12")
}
```

```kotlin
class DependencyHandler {
    fun compile(coordinate: String) {
        println("Added dependency on $coordinate")
    }

    override fun invoke(
        body: DependencyHandler.() -> Unit
    ) {
        body() // 함수 타입의 인자를 그대로 호출
    }
}
```

## 11.4 실전 코틀린 DSL

### 중위 호출 연쇄: 테스트 프레임워크의 should

```kotlin
s should startWith("kot") and endWith("in") and haveSubstring("lin")

infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this) // should 구현

interface Matcher<T> {
    fun test(value: T)
}

class startWith(val prefix: String) : Matcher<String> { // DSL에서 일반적 명명규칙에서 벗어남
    override fun test(value: String) {
        if (!value.startsWith(prefix)) {
            throw AssertionError("String $value does not start with $prefix")
        }
    }
}
```

### 원시 타입에 대한 확장 함수 정의: 날짜 처리

```kotlin
val yesterday = 1.days.ago
val tomorrow = 1.days.fromNow
```

날짜 조작 DSL

```kotlin
val Int.days: Period get() = Period.ofDays(this)
val Period.ago: LocalDate get() = LocalDate.now() - this
val Period.fromNow: LocalDate get() = LocalDate.now() + this
```

days: Int 타입에 확장 함수 정의
ago: Period 클래스에 대한 확장 함수
- LocalDate JDK 클래스에는 코틀린의 - 연산자 관례와 일치하는 minus 메서드가 있음 -> - 연산 자동 적용

### 멤버 확장 함수: SQL을 위한 내부 DSL

멤버 확장: 클래스 안에 선언된 함수를 확장 함수로 정의

```kotlin
class Table {
    fun <T> Column<T>.primaryKey(): Column<T>
    fun <T> Column<T>.autoIncrement(): Column<Int>
}
```
두 함수는 Table 클래스의 멤버
- 메서드가 적용되는 범위 제한

### 안코: 안드로이드 UI를 동적으로 생성하기


