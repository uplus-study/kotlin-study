## 8.1 고차 함수 정의

고차 함수 = 다른 함수를 인자로 받거나 함수를 반환하는 함수 (함수 = 람다, 함수 참조를 사용 가능)

```kotlin
list.filter { x > 0 }
```

술어 함수를 인자로 받으므로 고차 함수

### 함수 타입

```kotlin
val sum = { s: Int, y: Int -> x + y } // 람다를 변수에 대입
val sum: (Int, Int) -> Int = { x, y -> x + y } // 각 변수에 구체적인 타입 선언을 추가
```

`(파라미터 타입) -> (반환 타입)`

- 함수 타입을 선언할 때는 반환 타입 반드시 명시 → `Unit`을 빼먹으면 안 된다

```kotlin
val canReturnNull: (Int, Int) -> Int? = {x, y -> null } // 반환 타입이 널이 될 수 있는 타입
val funOrNull: ((Int, Int) -> Int)? = null // 널이 될수 있는 함수 타입 변수
```

```kotlin
fun performRequest(
	url: String,
	callback: (code: Int, content: String) -> Unit
) { ... } // 함수 타입에서 파라미터 이름 지정 가능
// 지정한 파라미터 이름과 정의할 람다의 이름이 일치하지 않아도 ok, 가독성이 좋아짐
```

### 인자로 받은 함수 호출

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
	val result = operation(2, 3) // 함수 타입 파라미터 호출
	println("The result is $result")
}
```

일반 함수 호출하는 구문과 같다

```kotlin
fun String.filter(predicate: (Char) -> Boolean) : String
```

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*VoD88TNNgQuLdPR0y8CmJA.png)



- 파라미터 함수: 문자(Char) 타입을 파라미터로 받아 Boolean 결과 값 반환

```kotlin
fun String.filter(predicate: (Char) -> Boolean): String {
    val sb = StringBuilder()
    for (index in 0 until length) {
        val element = get(index)
        if (predicate(element)) sb.append(element) // predicate 파라미터로 전달받은 함수 호출
    }
    return sb.toString()
}

>>> println("ab1c".filter { it in 'a'..'z' })
abc
```

### 자바에서 코틀린 함수 타입 사용

컴파일된 코드 안에서 함수 타입: 일반 인터페이스로 바뀜 (`FunctionN` 인터페이스를 구현하는 객체)

코틀린 표준 라이브러리: 함수 인자의 개수에 따라 `Function0<R>`, `Function1<P1, R>` 등 인터페이스 제공

각 인터페이스에는 `invoke` 메서드 정의가 포함 → 함수 실행

- 함수 타입인 변수는 `invoke` 메서드 본문에 람다의 본문이 들어간다

자바에서도 함수 타입을 사용하는 코틀린 함수 호출 가능 (자바 8 람다 사용)

```kotlin
/* Kotlin declaration */
fun processTheAnswer(f: (Int) -> Int) {
    println(f(42))
}
/* Java */
>>> processTheAnswer(number -> number + 1);
43
```

- 자바 8 이전: `FunctionN` 인터페이스의 `invoke` 메서드를 구현하는 무명 클래스를 넘겨주면 사용 가능

```java
/* Java */
>>> processTheAnswer(
	new Function1<Integer, Integer>() {
	    @Override
	    public Integer invoke(Integer number) {
	        System.out.println(number);
	        return number + 1;
	} 
});
```

- 자바에서 코틀린 표준 라이브러리가 제공하는 람다를 인자로 받는 확장 함수 호출 가능
    - 수신 객체를 확장 함수의 첫 번째 인자로 명시적으로 넘겨야 한다

```java
/* Java */
>>> List<String> strings = new ArrayList();
>>> strings.add("42");
>>> CollectionsKt.forEach(strings, s -> { // strings: 확장 함수의 수신 객체 -> 명시적으로 넘겨야 한다
    System.out.println(s);
		return Unit.INSTANCE; // Unit 타입 명시적으로 반환
});
```

### 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터

파라미터를 함수 타입으로 선언할 때도 **디폴트 값**을 정할 수 있다

```kotlin
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = "",
        transform: (T) -> String = { it.toString() } // 함수 타입 파라미터 디폴트 값 지정
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(transform(element)) // 파라미터 ㅏㅁ수 호출
    }
    result.append(postfix)
    return result.toString()
}

println(letters.joinToString()) // 디폴트 변환 함수 사용
println(letters.joinToString { it.toLowerCase() }) // 람다를 인자로 전달
```

널이 될 수 있는 함수 타입 사용

- null 여부 검사 또는 안전 호출 활용

```kotlin
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = "",
        transform: ((T) -> String)? = null // 널이 될 수 있는 파라미터
): String {
    val result = StringBuilder(prefix)
		for ((index, element) in this.withIndex()) {
	    if (index > 0) result.append(separator)
	    val str = transform?.invoke(element) // 안전 호출
	        ?: element.toString() // 람다를 인자로 받지 않은 경우 처리하기
	    result.append(str)
		}
		result.append(postfix)
    return result.toString()
}
```

### 함수를 함수에서 반환

프로그램의 상태나 다른 조건에 따라 달라질 수 있는 로직 존재

```kotlin
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int)

fun getShippingCostCalculator(
        delivery: Delivery): (Order) -> Double { // 함수를 반환하는 함수 선언
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount } // 함수에서 람다를 반환
}
    return { order -> 1.2 * order.itemCount }
}
>>> val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
>>> println("Shipping costs ${calculator(Order(3))}")

```

### 람다를 활용한 중복 제거

```kotlin
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
```

방문 데이터 분석하기

- OS별로 구분, 모바일 디바이스 사용자(여러 OS 타입), 특정 페이지 등 조건 파라미터가 추가될 경우 중복 코드 증가

→ 함수 타입을 사용하여 필요한 조건을 파라미터로 뽑아낼 수 있다

```kotlin
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
        filter(predicate).map(SiteVisit::duration).average()
>>> println(log.averageDurationFor {
     it.os in setOf(OS.ANDROID, OS.IOS) })

>>> println(log.averageDurationFor {
     it.os == OS.IOS && it.path == "/signup" })
8.0
```

파라미터에 함수를 추가하여 다양한 조건을 간단히 구현 가능

## 8.2 인라인 함수: 람다의 부가 비용 없애기

람다를 활용한 코드의 성능?

- 코틀린은 보통 람다를 무명 클래스로 컴파일, 람다 식을 사용할 때마다 새로운 클래스가 만들어지지는 않음
    - 람다가 변수 포획: 람다가 생성되는 시점마다 새로운 무명 클래스 객체 생성
    - 실행 시점에 무명 클래스 생성에 따른 부가 비용

→ 일반 명령문만큼 효율적인 코드?: `inline` 변경자 사용하기

### 인라이닝이 작동하는 방식

`inline` 선언: 함수의 본문이 인라인된다

- 함수를 호출하는 코드를 함수를 호출하는 바이트코드 대신, 함수 본문을 번역한 바이트 코드로 컴파일한다

```kotlin
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
        lock.lock()
        try {
            return action()
				}
				finally {
            lock.unlock()
        }
}
val l = Lock()
synchronized(l) {
		// ... }
```

다중 스레드 환경에서 공유 자원에 대한 동시 접근을 막기 위한 코드

함수를 inline으로 선언 → `synchronized`를 호출하는 코드는 자바의 `synchronized`문과 같아진다

```kotlin
fun foo(l: Lock) {
	println("Before sync")
	synchronized(l) {
		println("Action")
	}
	println("After sync")
}
```

위 코드를 컴파일한 코드

```kotlin
fun foo(l: Lock) {
	println("Before sync")
	l.lock()
	try {
      println("Action")
	}
	finally {
      lock.unlock()
  }
	println("After sync")
}
```

- `synchronized` 함수의 본문 + 전달된 람다의 본문도 함께 인라이닝된다
    - 코틀린 컴파일러는 람다의 본문에 만들어지는 바이트코드를 무명 클래스로 감싸지 않는다 (람다를 호출하는 코드(`synchronized`) 정의의 일부분으로 간주)

인라인 함수를 호출하면서 함수 타입의 변수를 넘기는 경우

- 인라인 함수를 호출할 때 함수 타입의 변수의 코드를 알 수 없음 → 람다 본문은 인라이닝되지 않는다

```kotlin
class LockOwner(val lock: Lock) {
    fun runUnderLock(body: () -> Unit) {
        synchronized(lock, body)
    }
}

class LockOwner(val lock: Lock) {
    fun __runUnderLock__(body: () -> Unit) {
        lock.lock()
        try {
					body() }
        finally {
            lock.unlock()
				} 
		}
}
```

### 인라인 함수의 한계

- 람다를 사용하는 모든 함수를 인라이닝할 수는 없다
- 람다가 본문에 직접 들어가기 때문에 함수가 파라미터로 전달받는 람다를 본문에 사용하는 방식이 한정적
    - 람다를 다른 변수에 저장하고 나중에 그 변수를 사용하는 경우에는 인라이닝될 수 없다

인라이닝할 수 있는 경우

- 인라인 함수의 본문에서 람다식을 바로 호출
- 람다 식을 인자로 전달받아 바로 호출

인라이닝할 수 없는 경우 → `Illegal usage of inline-parameter`

둘 이상의 람다를 인자로 받는 함수에서 일부 람다만 인라이닝하고 싶을 때

- `noinline` 변경자를 파라미터 앞에 붙여 인라이닝 금지

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
  // ...
}
```

### 컬렉션 연산 인라이닝

코틀린 표준 라이브러리의 컬렉션 함수: 람다를 인자로 받는다

ex) `filter` 함수: 인라인 함수 → 컬렉션을 직접 거르는 함수를 구현한 것과 같이 동작

`filter`와 `map`을 연쇄해서 사용

- 람다 식과 멤버 참조 사용
    - 인라인 함수로 추가 객체/클래스 생성은 없음
    - 리스트를 걸러낸 결과를 저장하는 중간 리스트 생성: `filter` → 중간 리스트 → `map`
        - 처리할 원소가 많다면 `asSequence`를 사용하여 중간 리스트 부가 비용 감소-중간 시퀀스는 람다를 필드에 저장하는 객체로 표현
        - → 시퀀스는 람다를 인라인하지 않는다 (람다를 저장해야 하기 때문)

결론: 시퀀스를 통해 성능을 향상시킬 수 있는 경우는 컬렉션 크기가 큰 경우뿐

크기가 작은 컬렉션은 람다가 인라이닝되지 않아 생기는 성능 이슈가 더 클 수 있다

### 함수를 인라인으로 선언해야 하는 경우

inline 키워드를 사용해도 **람다를 인자로 받는 함수**만 성능이 좋아질 가능성이 높다

- 일반 함수 호출
    - JVM이 강력하게 인라이닝 지원
    - 코드 실행을 분석하여 가장 이익이 되는 방향으로 호출을 인라이닝 (바이트코드 → 기계어 코드)
    - 바이트코드에서는 각 함수 구현이 한 번만 있으면 됨, 그 함수를 호출하는 부분에서 코드 중복이 필요하지 않음
    - 코틀린 인라인 함수
        - 바이트 코드에서 각 함수 호출 지점을 함수 본문으로 대치 → 코드 중복이 생김
        - 함수를 직접 호출할 때 스택 트레이스가 더 깔끔하다
- 람다를 인자로 받는 함수
    1. 인라이닝을 통해 없앨 수 있는 부가 비용이 많다
        - 함수 호출 비용 줄이기
        - 람다를 표현하는 클래스, 람다 인스턴스를 만들 필요가 없다
    2. JVM이 함수 호출/람다를 인라이닝해 줄 정도로 똑똑하지 못하다
    3. 일반 람다에서 사용할 수 없는 기능 사용 가능

inline 변경자를 함수에 무분별하게 붙일 경우 바이트코드가 전체적으로 아주 커질 수 있다

### 자원 관리를 위해 인라인된 람다 사용

`withLock`

```kotlin
fun <T> Lock.withLock(action: () -> T): T { // 락을 획득한 후 작업하는 과정(action)을 별도 함수로 분리
    lock()
    try {
        return action()
    } finally {
        unlock()
		} 
}

val l: Lock = ...
l.withLock {
	... // 락에 의해 보호되는 자원 사용
}
```

자바의 `try-with-resource`

```java
/* Java */
static String readFirstLineFromFile(String path) throws IOException {
    try (BufferedReader br =
                   new BufferedReader(new FileReader(path))) {
			return br.readLine();
		}
}
```

코틀린 `use` 함수: 람다를 호출한 후 자원을 닫아 줌

- 정상 종료 + 예외가 발생한 경우에도 자원을 닫아줌
- 인라인 함수

```kotlin
fun readFirstLineFromFile(path: String): String {
    BufferedReader(FileReader(path)).use { br -> // BufferedReader 객체를 만들고 use 함수를 호출하면서 람다를 넘긴다
				return br.readLine() // 자원(파일)에서 맨 처음 가져온 한 줄을 readFirstLineFromFile에서 반환 (람다 반환x)
    }
}
```

## 고차 함수 안에서 흐름 제어

### 람다 안의 return문: 람다를 둘러싼 함수로부터 반환

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return // 람다 함수(forEach)가 아닌 lookForAlice 함수에서 반환됨
				} 
		}
    println("Alice is not found")
}

```

넌로컬 `return`: 자신을 둘러싸고 있는 블록보다 더 바깥에 있는 다른 블록을 반환하게 만든다

- 람다 안에서 `return` 사용: 그 람다를 호출하는 함수가 실행을 끝내고 반환

인라이닝되지 않는 함수에 전달되는 람다 안에서 return은 사용 불가

→ 람다를 변수에 저장할 수 있어서 바깥쪽 함수로부터 반환된 후에 호출이 가능하기 때문

### 람다로부터 반환: 레이블을 사용한 return

람다에서 로컬 return 사용

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach label@{ // 람다 식 앞에 레이블을 붙인다
        if (it.name == "Alice") return@label // 앞에서 정의한 레이블을 참조
    }
    println("Alice might be somewhere") // 함수가 return되지 않아 이 줄이 출력됨
}

fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") return@forEach // 람다식으로부터 반환
    }
    println("Alice might be somewhere") // 함수가 return되지 않아 이 줄이 출력됨
}

>>> lookForAlice(people)
Alice might be somewhere
```

함수 이름을 return 레이블로 사용할 수 있음

레이블을 명시할 경우 함수 이름은 레이블로 사용 불가

### 무명 함수: 기본적으로 로컬 return

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(fun (person) { // 람다 대신 무명함수 사용
        if (person.name == "Alice") return // 무명함수가 return ehla
        println("${person.name} is not Alice")
    })
>>> lookForAlice(people)
Bob is not Alice
```