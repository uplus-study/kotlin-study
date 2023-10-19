# DSL 만들기

## API에서 DSL로
궁극적인 목표: 코드 가독성, 유지 보수성 가장 좋게 유지하는 것

-> 다른 클래스와의 상호작용 지점을 확인해야 함. 즉 API 살펴봐야 함

API가 깔끔하다
- 코드 읽는 독자들이 어떤 일이 벌어질지 명확하기 이해할 수 있어야 함, 이름과 개념 잘 선택
- 코드가 간결함, 불필요한 구문이나 번잡한 준비 코드가 적은 경우 => 여기에 초점을 둠!

**깔끔한 API를 위한 코틀린 기능**
|일반 구문|간결한 구문|사용한 언어 특성|
|-|-|-|
|StringUtil.capitalize(s)|s.capitalize()|확장 함수|
|1.to("one")|1 to "one"|중위 호출|
|set.add(2)|set += 2|연산자 오버로딩|
|map.get("key")|map["key"]|get 메서드 관례|
|filter.use({ f-> f.read() })|filter.use{ it.read() }|람다를 괄호 밖으로 빼내는 관례|
|sb.append("yes")<br>sb.append("no")|with(sb) {<br>append("yes")<br>append("no")<br>}|수신 객체 지정 람다|

### 영역 특화 언어
**DSL(Domain-Specific Language)** 
- 특정 과업 또는 영역에 초점을 맞추고 영역에 필요하지 않은 기능을 없앤 언어
- vs 범용 프로그래밍 언어
- SQL, 정규식
- 스스로 제공하는 기능을 제한하면서 특정 영역에 대한 연산을 더 간결하게 기술할 수 있음
- 범용 프로그래밍 언어와 달리 더 **선언적임**
  - 범용 프로그래밍 언어는 보통 명령적 -> 어떤 연산 완수하기 위해 필요한 각 단계 순서대로 기술
  - 선언적 언어는 원하는 결과를 기술하고 결과 달성 위해 필요한 세부 실행은 언어 해석하는 엔진에 맡김 
- DSL을 범용 언어로 만든 호스트 애플리케이션과 함께 조합하기 어려움
  - DSL 자체 문법이 있으므로 다른 언어 프로그램 안에 직접 포함시킬 수 없음

=> **내부 DSL**로 DSL의 단점 해결하면서 다른 이점 살리고자 함

### 내부 DSL
- 범용 언어로 작성된 프로그램 일부, 범용 언어와 동일한 문법 사용
- 내부/외부 DSL 비교
  - 외부 DSL: SQL, 내부 DSL: 코틀린으로 작성된 DB 프레임워크인 Exposed 프레임워크 제공하는 DSL
  ```sql
  --가장 많은 고객이 살고 있는 나라 알아내기
  SELECT Country.name, COUNT(Custom.id)
    FROM Country
    JOIN Customer
        ON Country.id = Customer.country_id
  GROUP BY Country.name
  ORDER BY COUNT(Customer.id) DESC
    LIMIT 1
  ```

  ```kt
  (Country join Customer)
      .slice(Country.name, Count(Customer.id))
      .selectAll()
      .groupBy(Country.name)
      .orderBy(Count(Customer.id), isAsc = false)
      .limit(1)
  ```
- 둘은 동일한 결과를 가져오지만 두 번쨰 버전은 일반 코틀린 코드임
- SQL 질의가 돌려주는 결과 집합을 코틀린 객체로 변환할 필요가 없음
- 코드는 어떤 구체적인 과업(SQL 질의 만듦) 달성하기 위한 것이지만, 범용 언어의 라이브러리로 구현됨 => 내부 DSL

### DSL 구조
명령-질의 API(command-query API)
- 함수 호출 시퀀스에 아무런 구조가 없고 호출 사이 맥락이 없음
  
DSL
- 메서드 호출은 DSL 문법에 의해 정해지는 더 커다란 **구조**에 속함
- 코틀린 DSL에서는 보통 람다를 중첩시키거나 메서드 호출 연쇄시키는 방식으로 구조 만듦
- 람다 중첩
  - 같은 문맥을 함수 호출 시마다 반복하지 않고 재사용 가능
  ```kt
  dependencies { //람다 중첩을 통해 구조 만듦 => 중복 줄어듦
    compile("junit:junit:4.11")
    compile("com.google.inject:guice:4.1.0")
  }
  ```
- 메서드 호출 연쇄
  ```kt
  str should startWith("kot")
  ```

### 내부 DSL 예제: HTML 만들기
```kt
fun createSimpleTable() = createHTML().
    table {
        tr {
            td { +"cell" }
        }
    }
```
- 코틀린 버전은 타입 안전성 보장, 컴파일 시점에 체크
- 동적으로 칸 생성 가능

## 구조화된 API 구축: DSL에서 수신 객체 지정 DSL 사용
### 수신 객체 지정 람다와 확장 함수 타입

람다 본문에서 매번 it 사용하지 않고 더 간단하게 호출하기 원함! => 수신 객체 지정 람다로 수정

람다의 인자 중 하나에게 수신 객체라는 상태 부여하면 이름과 마침표 명시하지 않아도 인자의 멤버 바로 사용 가능

```kt
fun buildString(
    builderAction: StringBuilder.() -> Unit //확장 함수 타입 사용
): String {
    val sb = StringBuilder()
    sb.builderAction() //StringBuilder 인스턴스를 람다 수신 객체로 넘김
    return sb.toString()
}

>>> val s = buildString {
    this.append("Hello, ") //this 는 StringBuilder 인스턴스
    append("World!") //this 생략 가능
}
>>> println(s)
Hello, World!
```

- 확장 함수 타입
  - 람다의 파라미터 목록에 있던 수신 객체 타입을 파라미터 목록 여는 괄호 앞으로 빼 놓으면서 중간에 마침표 붙인 형태
  `수신객체타입.파라미터타입 -> 반환타입   `
  - 확장 함수 본문에서 확장 대상 클래스에 정의된 메서드를 마치 클래스 내부에서 호출하듯이 사용
  
![img](https://github.com/uplus-study/kotlin-study/assets/126530537/bb0c2f3b-c489-433c-8b50-b946ce5e8bff)
- buildString 함수(수신 객체 지정 람다)의 인자는 확장 함수 타입의 파라미터와 대응
- 호출된 람다 본문 안에서는 수신객체(sb)가 묵시적 수신 객체(this)가 됨

- 확장 함수 타입의 변수를 정의 가능 -> 인자로 넘길 수 있음
- 소스코드상에서 수신 객체 지정 람다는 일반 람다와 똑같아 보임 
  - 수신 객체 있는지 보려면 람다가 전달되는 함수를 살펴봐야 함

표준 라이브러리 buildString 구현
```kt
fun buildString(builderAction: StringBuilder.() -> Unit): String {
    StringBuilder().apply(builderAction).toString()
}
```
- apply 함수는 인자로 받은 람다나 함수를 호출하면서 자신의 수신 객체 (StringBuilder 인스턴스)를 람다나 함수의 묵시전 수신 객체로 사용
- apply, with 모두 자신이 제공받은 수신 객체로 확장 함수 타입의 람다를 호출함

### 수신 객체 지정 람다를 HTML 빌더 안에서 사용
HTML 만들기 위한 코틀린 DSL => HTML 빌더

```kt
fun createSimpleTable() = createHTML().
    table {
        tr {
            td { +"cell" }
        }
    }
```
- 각 함수는 고차 함수로 수신 객체 지정 람다를 인자로 받음
- 각 수신 객체 지정 람다가 이름 결정 규칙을 바꿈
  - table 함수에 넘겨진 람다에서는 tr 함수 이용해 태그 만들 수 있지만 람다 밖에서는 tr 사용 X
  - 각 블록의 이름 결정 규칙은 람다 수신 객체에 의해 결정됨
  - table에 전달된 수신 객체는 TABLE이라는 타입이고 안에 tr 메서드 정의가 있음
  - 마찬가지로 tr함수는 TR 객체에 대한 확장 함수 타입의 람다를 받음
  ```kt
  open class Tag
  class TABLE: Tag {
    fun tr(init: TR.() -> Unit) //TR 타입을 수신 객체로 받는 람다를 인자로 지정
  }
  class TR: Tag {
    fun td(init: TD.() -> Unit)
  }
  class TD: Tag
  ```

- 수신 객체 지정 람다가 다른 수신 객체 지정 람다 안에 들어가면 내부 람다에서 외부 람다에 정의된 수신 객체 사용 가능
  - td 함수의 인자인 람다 안에서는 세 가지 수신 객체(this@table, this@tr, this@td) 사용 가능 
  - 코틀린 1.1부터 `@DslMarker` 사용해 중첩된 람다에서 외부 람다의 수신 객체 접근 못하게 제한 가능

```kt
//HTML 빌더 전체 구현
open class Tag(val name: String) {
    private val children = mutableListOf<Tag>() //모든 중첩 태그 저장
    protected fun <T : Tag> doInit(child: T, init: T.() -> Unit) {
        child.init() //자식 태그 초기화
        children.add(child) //자식 태그에 대한 참조 저장
    }
    override fun toString() = 
        "<$name>${children.joinToSTring("")}</$name>"
}
fun table(init: TABLE.() -> Unit) = TABLE().apply(init)
class TABLE : Tag("table") {
    //TR 태그 인스턴스를 새로 만들고 TABLE 태그의 자식으로 등록
    fun tr(init: TR.() -> Unit) = doInit(TR(), init) 
}
class TR : Tag("tr") {
    //TD 태그 인스턴스 새로 만들어서 TR 태그의 자식으로 등록
    fun td(init: TD.() -> Unit) = doInit(TD(), init)
}
class TD : Tag("td")
fun createTable() =
    table {
        tr {
            td {
            } 
        }
    }
>>> println(createTable())
<table><tr><td></td></tr></table>
```

수신 객체 지정 람다를 사용하면 DSL에 유용
- 코드 블록 내부에서 이름 결정 규칙 바꿀 수 있음 -> API에 구조 추가 가능
  
### 코틀린 빌더: 추상화와 재사용을 가능하게 하는 도구
내부 DSL 사용하면 일바 ㄴ코드와 마찬가지로 반복되는 내부 DSL 코드 조각을 새 함수로 묶어서 재사용 가능

예시) 애플리케이션에 드롭다운 목록 추가
- 코틀린에서 kotlinx.html 사용하면 HTML 구조 만들기 위해 div, button, ui, li 등 함수 사용 가능 
- div, button 모두 일반 함수이기 때문에 반복되는 로직을 별도 함수로 분리하면 코드를 더 읽기 쉬움 
```kt
fun dropdownExample() = createHTML().dropdown {
    dropdownButton { +"Dropdown" }
    dropdownMenu {
        item("#", "Action")
        item("#", "Another action")
        divider()
        dropdownHeader("Header")
        item("#", "Separated link")
    } 
}
```
- item 함수 구현
  - 파라미터 2개 -> href 에 들어갈 주소, 메뉴 원소 이름
  - 함수 코드는 드롭다운 메뉴 목록에 li { a(href) { +name } } 원소 새로 추가함
  - item 함수가 어떻게 li 호출??
  - li함수가 UL 클래스의 확장 함수이므로 UL 클래스의 확장 함수로 구현 가능 <br>
    `fun UL.item(href: String, name: String) = li { a(href) { +name }}`
  - item 함수는 항상 LI 태그를 추가함
  - 비슷하게 UL에 대해서 두 가지 확장 함수 정의해서 li 태그 없앨 수 있음 (divider(), dropdownHeader())
- dropdownMenu 구현
  - `dropdown-menu` CSS 클래스가 지정된 uㅣ xormfmf aksemfa
  - 인자로 태그 내용 채워 넣는 수신 객체 지정 람다를 받음
  ```kt
  dropdownMenu {
    item("#", "Action")
    ...
  }
  ```
  - ul 블록을 dropdownMenu{} 로 바꿀 수 있음 
  `fun DIV.dropdownMenu(block: UL.() -> Unit) = ul("dropdown-menu", block)`
- dropdown 함수


## invoke 관례 사용한 더 유연한 블록 중첩

### invoke 관례: 함수처럼 호출할 수 있는 객체
invoke는 get과 같은 역할을 함

다만 각괄호 대신 괄호 사용 
```kt
class Greeter(val greeting: String ) {
    operator fun invoke(name: String) { //클래스 객체 함수처럼 호출
        println("$greeting, $name!")
    }
}
>>> val bavarianGreeter = Greeter("Servus")
>>> bavarianGreeter("Dmitry") //bavarianGreeter.invoke("Dmitry")로 컴파일
Survus, Dmitry!
```
- invoke 메서드의 시그니처에 대한 요구사항 X => 원하는 대로 파라미터 개수나 타입 지정 가능

### invoke 관례와 함수형 타입
- 인라인하는 람다 제외한 모든 람다는 함수형 인터페이스(FunctionN) 구현하는 클래스로 컴파일됨
- 각 함수형 인터페이스 안에는 인터페이스 이름 가리키는 개수만큼 파라미터 받는 invoke 메서드 있음
```kt
interface Function2<in P1, in P2, out R> { //정확히 2개 인자 받음
    operator fun invoke(p1: P1, p2: P2): R
}
```
- 람다를 함수처럼 호출하면 관례에 따라 invoke 메서드 호출로 변환
=> 
  - 복잡한 람다를 여러 메서드로 분리하면서도 여전히 분리 전의 람다처럼 외부에서 호출할 수 있는 객체 만들 수 있음
  - 기존 람다를 여러 함수로 나눌 수 있으려면 함수 타입 인터페이스를 구현하는 클래스 정의 필요
  - 기반 인터페이스를 FunctionN 타입이나 `(P1, ... PN) -> R` 타입으로 명시
  - 람다 본문에서 따로 분리된 메서드가 영향 끼치는 영역 최소화 가능 
    - 술어 클래스 내부에서만 람다에서 분리해낸 메서드 볼 수 있음 => 관심사 분리

### DSL의 invoke 관례: gradle 의존관계 정의
한 API에 대해 두 형식 모두 지원하고 싶음
```kt
dependencies.compile("junit:junit:4.11")
dependencies {
    compile("junit:junit:4.11")
}
```
- DSL 사용자가 설정해야 할 항목이 많으면 중첩된 블록 구조 사용하고 설정 항목 적다면 간단한 함수 호출 구조 사용할 수 있음
- 첫 번째 경우 dependencies 변수에 대해 compile 메서드 호출
- 두 번째 경우 dependencies 안에 람다를 받는 invoke 메서드 정의하면 사용 가능


- dependencies 객체는 DependencyHandler 클래스의 인스턴스
  - 이 안에는 compile, invoke 메서드 정의되어 있음
  - invoke는 수신 객체 지정 람다를 파라미터로 받고 이 람다의 수신 객체는 다시 DependencyHandler임
  - DependencyHandler는 묵시적 수신 객체이므로 람다 안에서 메서드 직접 호출 가능
```kt
class DependencyHandler {
    fun compile(coordinate: String) { //일반적인 명령형 API 정의
        println("Added dependency on $coordinate")
    }

    //invoke 정의해 DSL 스타일 API 제공
    operate fun invoke(body: DependencyHandler.() -> Unit) {
        body() //this는 함수의 수신 객체이므로 this.body()와 같음
    }
}
```
두 번째 호출은 결과적으로 아래처럼 변환됨
```kt
dependencies.invoke({
    this.compile("org.jetbrains.kotlin:kotlin-reflect:1.0.0")
})
```
- dependencies를 함수처럼 호출하면서 람다를 인자로 넘김
- 람다의 타입은 확장 함수 타입(수신 객체 지정한 함수 타입)), 지정한 수신 객체 타입은 DependencyHanlder
- invoke 는 수신 객체 지정 람다를 호출

## 실전 코틀린 DSL

### 중위 호출 연쇄: 테스트 프레임워크의 should
DSL 에서 중위 호출 어떻게 활용하는지

```kt
a should startWith("kot") 

//should 함수 구현
infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this)

//Matcher 선언
interface Matcher<T> {
    fun test(value: T)
}

class startWith(val prefix: String): Matcher<String> {
    override fun test(value: String) {
        if(!value.startsWith(prefix)) {
            throw AssertionError("String $value does not start with $prefix")
        }
    }
}

``` 
- Matcher는 값에 대한 단언문 표현하는 제네릭 인터페이스
- startWith 클래스 첫 글자 대문자가 아님! DSL 에서는 일반적인 명명 규칙 벗어나야 할 때 있음

```kt
"kotlin" should start with "kot" //should, with 메서드 연쇄적으로 중위 호출

"kotlin".should(start).with("kot")

object start 
infix fun String.should(x: start): StartWrapper = StartWrapper(this)
class StartWrapper(val value: String) {
    infix fun with(prefix: String) = 
        if(!value.startsWith(prefix)) 
            throw AssertionError("String $value does not start with $prefix")
        else
            Unit
}
```
- object 로 선언한 타입을 파라미터 타입으로 사용할 필요가 없지만, DSL 문법 정의하기 위해 사용
  - start를 인자로 넘겨서 should 오버로딩한 함수 중 적절한 함수 선택 가능하고 결과로 StartWrapper 인스턴스를 받을 수 있음

### 원시 타입에 대한 확장 함수 정의: 날짜 처리
```kt
val yesterday = 1.days.ago
val tomorrow = 1.days.fromNow

//날짜 조작 DSL 정의
import java.time.Period
import java.time.LocalDate

val Int.days: Period
    get() = Period.ofDays(this)

val Period.ago: LocalDate
    get() = LocalDate.now() - this

val Period.fromNow: LocalDate
    get() = LocalDate.now() + this
```
- 원시 타입에 대한 확장 함수 정의하고 원시 타입 상수에 대해 확장 함수 호출

### 멤버 확장 함수: SQL 위한 내부 DSL
클래스 안에서 확장 함수와 확장 프로퍼티 선언

=> 클래스의 멤버인 동시에 확장하는 다른 타입의 멤버가 됨 : **멤버 확장**

```kt
//테이블 다루기 위해 Table 클래스 확장한 객체로 대상 테이블 정의
object Country : Table() {
    val id = integer("id").autoIncrement().primaryKey()
    val name = varchar("name", 50)
}

//테이블 생성하려면 SchemaUtils.create(Country) 메서드 호출
```
- id 는 Column<Int> 타입
- name은 Column<String> 타입
```kt
Class Table {
    //해당 타입으로 컬럼 생성
    fun integer(name: String): Column<Int>
    fun varchar(name: String, length: Int): Column<String>
    ...
}
```
- autoIncrement, primaryKey 같은 메서드를 사용해서 칼럼 속성 지정
- Column에 대해 메서드 호출 가능
- 각 메서드는 자신의 수신 객체 반환하기 때문에 메서드 연쇄 호출 가능
```kt
class Table{
    fun <T> Column<T>.primaryKey(): Column<T>
    fun Column<Int>.autoIncrement(): Column<Int>
    ...
}
```
- 두 함수는 Table의 멤버이므로 Table 클래스 밖에서 이 함수들 호출할 수 없음
=> 멤버 확장으로 정의하는 이유는 메서드 적용 범위 제한하기 위함
- 확장 함수 속성 중 수신 객체 타입을 제한하는 기능 있음
  - 테이블 안 어떤 컬럼이든 기본 키 될 수 있지만 자동 증가 칼럼은 정수 타입인 경우만

> 멤버 확장도 멤버다
>
> 멤버 확장은 클래스 내부에 속해 있어 확장성이 떨어짐
>
> Table 소스코드 수정하지 않고는 Table에 필요한 확장 함수나 프로퍼티 추가할 수 없음

```kt
//테이블 조인

val result = (Country join Customer)
    .select { Country.name eq "USA" }
result.forEach { println(it[Customer.name]) }
```
- eq메서드는 "USA"를 인자로 받는 함수이고 중위 표기식으로 식을 적음
- Column의 멤버 확장임
```kt
fun Table.select(where: SqlExpressionBuilder.() -> Op<Boolean>): Query
object SqlExpressionBuilder {
    infix fun<T> Column<T>.eq(t: T): Op<Boolean>
    ...
}
```

멤버 확장이 없다면 모든 함수를 Column의 멤버나 확장으로 정의해야 함

하지만 그렇게 정의하면 맥락과 관계없이 아무데서나 각 함수 사용할 수 있음

멤버 확장을 사용하여 각 함수를 사용할 수 있는 맥락 제어 가능

### 요약
- 내부 DSL은 여러 메서드 호출로 구성된 구조를 더 쉽게 표현할 수 있게 해주는 API 설계할 때 사용할 수 있는 패턴
- 수신 객체 지정 람다는 람다 본문 안에서 메서드를 결정하는 방식을 재정의 -> 여러 요소 중첩시키는 구조 만들 수 있음
- 수신 객체 지정 람다를 파라미터로 받은 경우 람다의 타입은 확장 함수 타입임
- 코틀린 내부 DSL 사용하면 코드 추상화하고 재활용 가능
- invoke 관례 사용하면 객체를 함수처럼 다룰 수 있음