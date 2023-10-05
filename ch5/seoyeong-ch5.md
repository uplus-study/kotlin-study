# 5장. 람다로 프로그래밍

## 람다 식과 멤버 참조

### 람다
함수를 값처럼 사용할 수 있기에 무명 내부 클래스로 일련의 동작을 변수에 저장하거나 넘겼던 작업을 대신할 수 있음.

### 람다식 문법
- 파라미터, 본문으로 구성
- `->` 를 사용하여 파라미터 목록과 람다 본문을 구분
- `{}`로 둘러싸서 사용
- 런타임시 코틀린 람다 호출은 부가 비용이 없음.

### 람다식 예시
```Kotlin
// 변수에 저장
val sum = { x: Int, y: Int -> x+y }

// 즉시 사용
{ println(42) }()

// run 함수로 람다 실행
run { println(42) }

val people = listOf(Person("Alice", 20), Person("Bob", 31))

// 람다가 마지막 파라미터이면, 괄호 밖에 위치 가능
people.maxBy( { p: Person ‐> p.age } )
people.maxBy { p: Person ‐> p.age }

// 파라미터 추론 가능하면 생략 가능
people.maxBy { p ‐> p.age }

// 파라미터가 한 개, 타입 추론 가능하면 it 사용
people.maxBy { it.age }

fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        // 바깥 함수 파라미터 접근
        println("$prefix $it")
    }
}

fun printProblemCounts(response: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0
    response.forEach {
        if (it.startsWith("4")) {
            // 람다 바깥의 변수 접근 및 수정
            clientErrors++
        } else {
        }
    }
}
```

### 멤버 참조
이미 선언된 함수를 값으로 사용 시 멤버 참조를 사용
```Kotlin
// '::'를 사용한 멤버 참조, 멤버를 호출하는 람다와 같은 타입이어야 함.
val getAge = Person::age

fun sendMail(p: Person, msg: String): Unit 

// (p: Person, msg: String): Unit 타입 함수
val nextAction = ::sendMail

// 생성자 참조
val createPerson = ::Person
```

### 바운드 멤버 참조 (1.1~)
```Kotlin
val p = Person("Dmitry", 34)

// p에 엮인 멤버 참조
val ageFunc = p::age
println(ageFunc())
```

## 컬렉션 함수형 API

### 컬렉션 함수 예시

```Kotlin
list.filter { it % 2 == 0}
list.map { it * it }

// 컬렉션 연산자 즉시 생성
list.filter { it % 2 == 0 } // 새로운 컬렉션
    .map { it * 2 } // 새로운 컬렉션

list.count
list.all { it > 3 }
list.any { it > 2 }
list.find { it > 3 } // 널가능 타입, 조건을 충족하는 첫 번째 원소 (firstOrNull과 동일)
map.filter { entry ‐> entry.key % 2 == 0 } // entry: Map.Entry
map.map { entry ‐> entry.key * 2 to entry.value }

// map은 filterKeys, mapKeys, filterValues, mapValues 제공함.

val strs = listOf("12", "345", "11", "456")
val grouped: Map<Int, List<String>> = strs.groupBy { it.length }

val books = listOf(
    Book("동용", listOf("작가1", "작가2")),
    Book("시집", listOf("작가3", "작가1"))
)
    
val authors = books.flatMap { it.authros } // List<String> [작가1, 작가2, 작가3, 작가1
    .toSet()
```

## 지연 계산(lazy) 컬렉션 연산

- 시퀀스는 최종 결과값에 대해서만 생성. 중간 연산의 결과를 저장하지 않고 결과값을 도출해 더 효율적
- 중간 연산은 항상 지연 계산

```Kotlin
// 수많은 원소의 리스트가 만들어져 비율적
people.map(Person::name)
    .filter { it.startsWith("A") } 
    .toList()


// 시퀀스를 활용하여 효율을 높임
people.asSequence()
    .map(Person::name) 
    .filter { it.startsWith("A") }
    .toList()
```

### 시퀀스 만들기
- `generateSequence()`로 시퀀스 생성
```Kotlin
val numbers = generateSequence(0) { it + 1 }

// 조건에 해당하는 것을 찾으면 directory 더 찾지 않음.
fun File.isInsideHiddenDirectory() = generateSequence(this) { it.parentFile }.any { it.isHidden }
```

## 자바 함수형 인터페이스 활용

> SAM 인터페이스
> - `Single Abstract Method`
> - 추상 메서드가 단 하나만 존재하는 인터페이스
> - `Runnable`, `Callable`

### Java 메서드에 람다를 인자로 전달
함수형 인터페이스를 파라미터로 가지는 자바 메서드를 호출할 때 람다를 넘길 수 있음.

```Kotlin
// java
void postponeComputation(int delay, Runnable computation)

// 람다를 인자로 전달
postponeComputation(1000) { println(42) }

// 여서는 객체를 명시적으로 선언하고 호출시마다 새로운 객체를 생성함.
postponeComputation(1000, object: Runnable {
    override fun run() {
        println(42)
    }
})
```

## 수신 객체 지정 람다: wit와 apply

수신 객체를 명시하지 않고 람다 본문에서 다른 객체의 메서드를 호출할 수 있는 것

### with
첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 지정

```Kotlin
// with 사용 X
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter) // result 중복
    }
    result.append("\nNow I know the alphabet.") // result 중복
    return result.toString()
}

// with 사용
fun alphabet() = with(StringBuilder() {
  for (letter in 'A'..'Z') {
    append(letter)
  }
  append("\nNow I know the alphabet!")
  toString()
})
```

### apply
- `with`와 유사, 하지만 항상 자신에게 전달된 객체를 반환함.
- `with`는 람다의 결과, `apply`는 객체 자체를 반환

```Kotlin
fun alphabet() = StringBuilder().apply {
  for(letter in 'A'..'Z'){
    append(letter)
  }
  append("append!")
}.toString()
```
