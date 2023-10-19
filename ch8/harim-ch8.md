# 고차 함수: 파라미터와 반환 값으로 람다 사용

### 고차 함수 정의
고차 함수: 다른 함수를 인자로 받거나 함수를 반환하는 함수  ex)filter

코틀린에서는 람다나 함수 참조를 사용해 함수를 값으로 표현 가능 

**함수 타입**
```Kotlin
val sum = { x: Int , y: Int -> x + y }
val action = { println(42) }
```
- 컴파일러가 sum, action이 **함수 타입**임을 추론함
- 변수에 구체적인 타입 선언 추가 O
```Kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
val action: () -> Unit = { println(42) }
```
- 널이 될 수 있는 함수 타입 정의 가능  `var funOrNull: ((Int, Int) -> Int)? = null`
- 함수 타입에서 파라미터 이름 지정 가능

**고차 함수 구현**
- String에 대한 filter 함수
    ```Kotlin   
    fun String.filter(predicate: (Char) -> Boolean): String
    ``` 
    - filter 함수는 술어 함수를 파라미터(predicate)로 받음
    - predicate는 Char 타입을 파라미터로 받고 Boolean 타입을 결과 값으로 반환

**자바에서 코틀린 함수 타입 사용**
- 컴파일된 코드 안에서 함수 타입은 일반 인터페이스 바뀜 `FunctionN<P1, .. ,R>`
  - 인터페이스에 `invoke` 메서드를 호출하여 함수 실행 가능
- 함수 타입 사용하는 코틀린 함수 자바에서 호출 O
    ```Kotlin 
    //코틀린 선언
    fun processTheAnswer(f: (Int) -> Int) {
        println(f(42))
    }

    //자바 호출
    //자바 8 이전은 무명 클래스 넘김, 자바 8 이후 람다 넘김
    processTheAnswer(number -> number + 1);
    ```
- 자바에서는 수신 객체를 확장 함수의 첫 번째 인자로 명시적으로 넘겨야 하므로 코드가 깖끔하지는 않음

**디폴트 값 지정 또는 널이 될 수 있는 함수 타입 파라미터**
- 3장에서 살펴본 `joinToString` 예제 사용
  - 이 구현은 컬렉션의 각 원소를 문자열로 변환함 ☹️ 
  - toString 메서드를 통해 객체를 문자열로 바꿈
  - 이런 경우 원소를 문자열로 바꾸는 방법을 람다로 전달 가능 -> 파라미터에 대한 디폴트 값 지정
```Kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ".
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() }
): String {
    val result = StringBuilder(prefix)
    for((index, element) in this.withIndex()) {
        if(index > 0) result.append(separator)
        result.append(transform(element))
    }
    result.append(postfix)
    return result.toString()
}
```
- 널이 될 수 있는 함수 타입 사용 가능
  - 그 함수를 직접 호출할 수 없음! 
  ```Kotlin
  fun foo(callback: (() -> Unit)?) {
    ...
    if(callback != null) {
        callback() //널 여부 명시적으로 검사 후 호출
    }
  }
  ```
  - 일반 메서드처럼 invoke는 안전 호출 구문이므로 `callback?.invoke()` 처럼 호출 가능
    ```Kotlin
    fun <T> Collection<T>.joinToString(
        separator: String = ", ".
        prefix: String = "",
        postfix: String = "",
        transform: ((T) -> String)? = null
    ): String {
        val result = StringBuilder(prefix)
        for((index, element) in this.withIndex()) {
            if(index > 0) result.append(separator)
            val str = transform?.invoke(element) ?: element.toString()
            result.append(str)
        }
        result.append(postfix)
        return result.toString()
    }
    ```

**함수를 함수에서 반환**
- 함수의 반환 타입으로 함수 타입을 지정해야 함
- 프로그램의 상태나 다른 조건에 따라 달라질 수 있는 로직 반환이 필요할 때

**람다로 중복 제거**
- 웹사이트 방문 기록 분석하는 예시
```Kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)

enum class OS { WINDOWS, LINUX, MAX, IOS, ANDROID }

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)
```
- 여기서 특정 OS 사용자의 방문 시간 출력하고 싶다면
```Kotlin
fun List<SiteVisit>.averageDurationFor(os: OS) = 
filter { it.os == os }.map(SiteVisit::duration).average()
```

- 추가로 모바일 디바이스(iOS, android)의 평균 방문 시간을 구하고 싶다면
```Kotlin
val averageMobilDuration = log
    .filter{ it.os.in setOf(OS.IOS, OS.ANDROID) }
    .map(SiteVisit::duration)
    .average()
```
  - 플랫폼 표현하는 간단한 파라미터로는 상황 처리 X
  - 더 복잡한 질의 있는 경우 있을 때는 람다가 유용
```Kotlin
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) = 
    filter(predicate).map(SiteVisit::duration).average()
```

### 인라인 함수: 람다의 부가 비용 없애기
- `inline` 선언 -> 컴파일러는 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 바꿈
```Kotlin
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
synchronized(1) {
    ...
}
```
- synchronized 함수를 inline으로 선언했으므로 synchronized를 호출하는 코드는 모두 자바의 synchronized문과 같아짐

```Kotlin
fun foo(l: Lock) {
    println("Before sync")
    synchronized(1) {
        println("Action")
    }
}
```
- 위의 코드는 아래 그림과 같은 바이트코드를 만들어냄
  - synchronized 함수 본문 + 전달된 람다 본문 모두 인라이닝됨
  
![img](https://github.com/uplus-study/kotlin-study/assets/126530537/6099add3-c7c9-4752-a019-6113f2738035)

- 인라인 함수 호출하면서 람다 대신 함수 타입의 변수 넘길 수 있음
    ```Kotlin
    class LockOwner(val lock: Lock) {
        fun runUnderLock(body: () -> Unit) {
            synchronized(lock, body) //람다 대신 함수 타입인 변수 인자로 넘김
        }
    }
    ```
  - 이런 경우는 변수에 저장된 람다의 코드를 알 수 없으므로 람다 본문은 인라이닝되지 않음
- 한 인라인 함수를 두 곳에서 각각 다른 람다를 사용해 호출한다면 따로 인라이닝됨
  - 인라인 함수 본문 코드가 호출 지점에 복사되고 각 람다의 본문이 인라인 함수의 본문 코드에서 람다를 사용하는 위치에 복사됨
  

**인라인 함수의 한계**
- 인라이닝 방식으로 람다를 사용하는 모든 함수를 인라이닝 할 수 X
  - 람다가 본문에 직접 펼쳐지기 때문에 함수가 파라미터로 전달받은 람다를 본문에 사용하는 방식이 한정됨
  - 파라미터로 받은 람다를 다른 변수에 저장하고 나중에 변수를 사용한다면 람다 인라이닝 X
- 인라인 함수의 본문에서 람다 식을 바로 호출하거나 람다 식을 인자로 전달받아 바로 호출하는 경우에 람다를 인라이닝할 수 있음
  - 이런 경우가 아니라면 컴파일러는 인라이닝 금지 시킴
- 둘 이상의 람다를 인자로 받는 함수에서 일부 람다만 인라이닝 원할 때 `noinline` 변경자를 파라미터 이름 앞에 붙여 인라이닝 금지할 수 있음
- 코틀린에서는 어떤 모듈이나 서드파티 라이브러리 안에서 인라인 함수 정의하고 밖에서 해당 인라인 함수를 사용할 수 있음
- 자바에서도 코틀린에서 정의한 인라인 함수 호출 가능 
  - 컴파일러는 인라인 함수를 일반 함수 호출로 컴파일함

**컬렉션 연산 인라이닝**
- 코틀린 표준 라이브러리의 컬렉션 함수는 대부분 람다를 인자로 받음! 
- filter 함수는 인라인 함수 
  - filter 써서 생긴 바이트코드와 직접 연산 구현해서 생긴 바이트코드는 거의 같음
  - 코틀린다운 연산을 컬렉션에 대해 안전하게 사용 가능하고 코틀린의 함수 인라이닝 성능 신경쓰지 않아도 됨
- filter와 map을 연쇄해서 사용한다면
  - 두 함수의 본문은 인라이닝되고 추가 객체나 클래스 생성 X
  - 하지만 중간 리스트를 추가 및 읽게 됨 -> 부가 비용 ↑ 
  - 시퀀스를 사용하게 되면 부가 비용은 줄지만 중간 시퀀스는 람다를 필드에 저장하는 객체로 표현되고 최종 연산은 중간 시퀀스에 있는 여러 람다를 연쇄 호출함 => 시퀀스는 람다 인라인 X

**인라인으로 선언해야 하는 경우**
- inline을 사용해도 *람다를 인자로 받는 함수* 만 성능이 좋아질 가능성이 높음
- 일반 함수 호출의 경우, JVM이 강력하게 인라이닝 지원함
  - JVM 최적화 활용한다면, 바이트코드에서 각 함수 구현 정확히 한 번만 있으면 되고 호출하는 부분에서는 코드 중복 X
  - 반면 코틀린 인라인 함수는 바이트코드에서 각 함수 호출 지점을 함수 본문으로 대치하게 때문에 코드 중복 O
- 람다를 인자로 받는 함수 인라이닝하면 이익이 많음
  - 인라이닝 통해 함수 호출 비용 줄일 수 있고 람다 표현하는 클래스와 람다 인스턴스 객체를 만들 필요 X
  - 현재 JVM은 함수 호출과 람다를 인라이닝해 줄 정도로 똑똑하지 X
  - 일반 람다에서는 사용할 수 없는 몇 가지 기능 제공 -> ex)non-local 반환
- inline 함수의 코드 크기에 주의해야 함
  - 바이트코드가 전체적으로 아주 커질 수 있음
  
**자원 관리를 위해 인라인된 람다 사용**
- 람다로 중복 없애는 일반적인 패턴 중 하나는 어떤 작업 하기 전 자원 획득하고 마친 후 자원 해제하는 '자원 관리'
- 일반적으로 try/finally문 사용
> **코틀린 라이브러리 함수: withLock**
> 
> Lock 인터페이스의 확장 함수
> ```Kotlin
>fun <T> Lock.withLock(action: () -> T): T {
> >    lock()
>     try {
>         return action()
>     } finally {
>         unlock()
>     }
> }
> ```

- 자바7에서는 `try-with-resource`문을 통해 해당 패턴 구현
```java
static String readFirstLineFromFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))){
        return br.readLine();
    }
}
```
- 코틀린에서는 함수 타입의 값을 파라미터로 받는 함수를 통해 해당 패턴 구현 가능 => `use` 함수
```Kotlin
fun readFirstLineFromFile(path: String): String {
    BufferedReaduer(FileReader(path)).use { br ->
        return br.readLine()
    }
}
```
  - use 함수: 닫을 수 있는 자원에 대한 확장 함수, 람다를 인자로 받음
  - 람다가 정상 종료한 경우나 람다 안에서 예외 발생한 경우 모두 자원 닫음
  - 람다 본문 안 return은 넌로컬 return -> 람다가 아니라 readFileFirstLineFromFile 함수 끝내면서 값을 반환함
  
### 고차 함수 안에서 흐름 제어
- **람다 안의 return문: 람다 둘러싼 함수로부터 반환**
  - 람다 안에서 return 사용하면 람다로부터만 반환되는 게 아니고 람다 호출하는 함수가 실행 끝내고 반환함
  - `non-local return`: 자신 둘러싸고 있는 블록보다 더 바깥에 있는 다른 블록을 반환하게 만드는 return
  - 코틀린에서는 언어가 제공하는 기본 구성 요소가 아니라 람다를 받는 함수로 for, synchronized와 같은 기능 구현함 -> 자바에서의 return과 같은 의미
  - return 이 바깥쪽 함수 반환시키는 때는 람다를 인자로 받는 함수가 인라인 함수인 경우뿐
  - 인라이닝되지 않는 함수에 전달되는 람다 안에서는 return 사용 X

- **람다로부터 반환: 레이블을 사용한 return**
  - 람다식 안에서 로컬 return은 for 루프의 break 역할과 비슷
  - 로컬 return은 람다 실행 끝내고 람다 호출했단 코드의 실행 이어감
  - 로컬/넌로컬 return 구분을 위해 `label`을 사용
    - return으로 실행 끝내고 싶은 람다 식 앞에는 레이블을 붙임
    ```Kotlin
    people.forEach label@{
        if(it.name == "Alice") return@label
    }
    ```
  - 람다에 레이블 붙이는 대신 람다를 인자로 받는 인라인 함수 이름을 return 뒤에 레이블로 사용해도 됨
    ```Kotlin
    poeple.forEach {
        if(it.name == "Alice") return@forEach
    }
    ```

- **무명 함수: 기본적으로 로컬 return**
  - 무명 함수는 코드 블록을 함수에 넘길 때 사용하는 다른 방법
    ```Kotlin
    people.filter(fun (person): Boolean {
        return person.age < 30
    })
    ```
  - 무명 함수도 일반 함수와 같은 반환 타입 지정 규칙을 따름
    - 블록이 본문인 무명 함수는 반환 타입 명시해야 하지만, 식을 본문으로 하는 무명 함수의 반환 타입은 생략 가능
  - 무명 함수 안에서 레이블 붙지 않은 return 식은 무명 함수 자체 반환시킴
  - return에 적용되는 규칙은 fun 키워드 사용해 정의된 가장 안쪽 함수를 반환시킴
    - 람다 식은 fun 사용해 정의되지 않으므로 람다 본문의 return은 바깥 함수를 반환 시킴
![img](https://github.com/uplus-study/kotlin-study/assets/126530537/559e7892-608c-4260-a40d-62da97976806)