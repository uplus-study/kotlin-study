# 8장. 고차 함수: 파라미터와 반환 값으로 람다 사용

## 고차 함수 정의
다른 함수를 파라미터로 넘기거나 함수를 반환하는 함수
(코틀린에서는 다른 함수 -> 람다, 함수 참조)

### 함수 타입
- 함수의 파라미터 타입을 괄호 안에 넣고 `->` 화살표 추가하고 반환 타입 지정

```Kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
val action: () -> Unit = { println(42) }
```

### 인자로 받은 함수 호출
```Kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
    val result = operation(2, 3)
    println("The result is $result")
}
```

### 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터
- 디폴트 값 지정 가능
```Kotlin
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
}
    result.append(postfix)
    return result.toString()
```
- 

### 함수를 함수에서 반환
- 함수의 반환 타입으로 함수의 타입을 지정
- return 에 람다나 함수 타입의 값을 내는 식 필요

```Kotlin
class Order(val itemCount: Int)
fun getShippingCostCalculator(
        delivery: Delivery): (Order) -> Double {
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
}
    return { order -> 1.2 * order.itemCount }
}
```

### 람다를 활용한 중복 제거
- 람다를 통해 일반 함수 중복 제거 가능

```Kotlin
data class SiteVisit(
            val path: String,
            val duration: Double,
            val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)

val averageWindowsDuration = log
            .filter { it.os == OS.WINDOWS }
            .map(SiteVisit::duration)
            .average()

// 고차함수로 중복 제거
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) = filter(predicate).map(SiteVisit::duration).average()
```

## 인라인 함수: 람다의 부가 비용 없애기
`inline` 변경자를 함수에 붙이면 컴파일러가 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 변경해줌.

### 컬렉션 연산 인라이
- `filter`, `map`은 인라인 함수
    - 해당 함수들의 본문은 인라이닝 됨.
    - 추가 객체, 클래스 생성 없음.
    - 중간 결과를 저장해야하므로 그만큼 비용이 증가 => `asSequence`를 통해서 결과값만 얻을 수 있음.

### 함수를 인라인으로 선언해야 하는 경우
- `inline` 사용하여도 람다를 인자로 받는 함수만 성능에 도움이 됨.
- 일반 함수 호출은 JVM에서 이미 강력한 인라이닝 지원
- 코틀린 인라인 함수는 바이크 코드에서 각 함수 호출 지점을 함수 본문으로 대치하므로 코드 중복 발생함.
- 코드 크기를 신경써서 인라인 사용.
    - 함수가 크면 바이트코드 전체가 커질 위험이 있음.

### 자원 관리를 위해 인라인된 람다 사용
- 자바의 try-with-resource와 같은 기능을 가진 `use` 함수 코틀린 라이브러리로 제공
```Kotlin
fun readFirstLineFromFile(path: String): String {
    BufferedReader(FileReader(path)).use { br ->
         return br.readLine()
    }
}
```

## 고차 함수 안에서 흐름 제어

### 람다 안의 return문: 람다를 둘러싼 함수로부터 반환
- 람다 안에서 return을 ㅅ용할 때 람다로부터만 반환되는 것이 아닌 그 람다를 호출하는 함수가 실행을 끝내고 반환됨.
- 자신을 둘러싸고 있는 블록보다 더 바깥에 있는 블록을 반환하게 만드는 return => `넌로컬 return`
    - 인라인 함수만 가능
```Kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}
```

### 람다로부터 반환: 레이블을 사용한 return
- 람다 식에서 로컬 return 사용 가능
    - for loop의 break와 비슷한 역할
    - 로컬 return, 넌로컬 return 구분을 위해 `label` 필요
```Kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach label@{
        if (it.name == "Alice") return@label
    }
    println("Alice might be somewhere")
}

// 람다를 인자로 받는 인라인 함수 이름을 return 뒤에 레이블로 사용 가능
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") return@forEach
    }
    println("Alice might be somewhere")
}
```

### 무명 함수: 기본적으로 로컬 return
- 코드 블록을 함수에 넘길 때 사용 가능
- 일반 함수와 유사하지만 함수 이름이나 파라미터 타입을 생략할 수 있음.
- 무명 함수 안에서 레이블이 붙지 않은 return 식은 무명 함수 자체를 반환하고 무명 함수를 둘라싼 다른 함수를 반환하지 않음.
```Kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(fun (person) {
        if (person.name == "Alice") return
        println("${person.name} is not Alice")
    })
}
```