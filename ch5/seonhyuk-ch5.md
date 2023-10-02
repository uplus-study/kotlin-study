# 5. 람다로 프로그래밍

## 5.1 람다 식과 멤버 참조

### 5.1.1 람다 소개: 코드 블록을 함수 인자로 넘기기

함수형 프로그래밍에서는 함수를 값처럼 다루는 접근 방법을 택함으로써 무명 내부 클래스를 사용하여 일련의 목적을 달성하는 번거로움을 줄일 수 있다.

### 5.1.2 람다와 컬렉션

```kotlin
val people = listOf(Person(“Alice”, 29), Person(“Bob”, 31))
println(people.maxBy { it.age })
```

각 컬렉션에 대한 라이브러리 함수를 쓰자

멤버 참조

- `people.maxBy(Person::age)`
- 위 코드와 같은 역할을 한다.

```kotlin
// { 파라미터 -> 본문 }
people.maxBy({ it.age })
// 함수의 마지막 람다 인자라면 괄호 밖으로 뺄 수 있다.
people.maxBy() { it.age }
// 람다가 유일한 인자이고, 괄호 뒤에 람다를 썼다면 빈 괄호를 없애도 된다.
people.maxBy { it.age }
```

- 컴파일러가 타입추론을 하기 때문에 별도로 타입을 적을 필요는 없다.
- 람다를 변수에 저장할 때는 추론할 타입이 존재하지 않기 때문에, 타입을 명시해야한다.
- 파라미터 이름을 따로 명시하지 않을 경우 `it`이 파라미터의 이름이 된다.
- 람다가 중첩되는 경우 쓰지 않는 걸 권장한다.

### 5.1.4 현재 영역에 있는 변수에 접근

람다를 함수 안에서 정의하면 람다 정의 앞에 선언된 로컬 변수도 사용 가능하다.
자바와 다르게 final이 아니어도 사용할 수 있으며, 람다 내에서 로컬 변수의 값 변경도 가능하다.

```kotlin
fun printProblemCounts(response: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0

    response.forEach {
        if (it.startsWith("4")) {
            clientErrors ++
        } else if (it.startsWith("5")) {
            serverErrors ++
        }
    }

    println("$clientErrors client errors, $serverErrors server errors")
}
```

람다를 이벤트 핸들러나 다른 비동기적으로 실행되는 코드로 활용하는 경우 함수 호출이 끝난 다음에 로컬 변수가 변경될 수도 있다.

### 5.1.5 멤버 참조

`::`를 사용하여 함수를 값으로 바꿀 수 있다.

```kotlin
val getAge = Person::age

// ==

val getAge = { person: Person -> person.age }
```

`::`를 사용하는 식을 멤버 참조라고 부른다.

- 프로퍼티나 메서드를 단 하나만 호출하는 함수 값을 만들어준다.
- 최상위에 선언된 함수나 프로퍼티를 참조할 수도 있다. `::salute`
- 확장함수도 똑같은 방법으로 참조할 수 있다.

## 5.2 컬렉션 함수형 API

### 5.2.1 필수적인 함수: filter와 map

### 5.2.2 all, any, count, find: 컬렉션에 술어 적용

- `find`는 `firstOrNull`과 같다. 조건을 만족하는 원소가 없으면 null을 반환하는데, 이 사실을 더 명확히 하려면 후자를 쓸 수 있다.

### 5.2.3 groupBy: 리스트를 여러 그룹으로 이뤄진 맵으로 변경

### 5.2.4 flatMap과 flatten: 중첩된 컬렉션 안의 원소 처리

## 5.3 지연 계산(lazy) 컬렉션 연산

```kotlin
people.map(Person::name).filter { it.startsWith("A") }
```

이런 함수는 `map` 과 `filter`가 모두 리스트를 반환하기 때문에, 리스트를 두 개 만들어 원소가 수 백만 개가 되면 비효율적일 수 있다.
이를 더 효율적으로 만들기 위해 시퀀스를 사용하게 만들 수 있다.

```kotlin
people.asSequance()
    .map(Person::name)
    .filter { it.startsWith("A") }
    .toList()
```

시퀀스의 원소는 필요할 때 계산된다. 따라서 중간 처리 결과를 저장하지 않고도 연산을 연쇄적으로 적용해서 효율적으로 계산을 수행할 수 있다.
시퀀스 원소를 인덱스를 사용해 접근하는 등 다른 API 메서드가 필요하다면 시퀀스를 리스트로 변환해야 한다.

### 5.3.1 시퀀스 연산 실행: 중간 연산과 최종 연산

시퀀스의 중간연산은 항상 지연 계산된다.

```kotlin
listOf(1, 2, 3, 4).asSequence()
    .map { print("map ($it)"); it * it }
    .filter { print("filter ($it)"); it %2 == 0 }
```

- 때문에, 이 코드는 아무것도 출력하지 않는다.

```kotlin
listOf(1, 2, 3, 4).asSequence()
    .map { print("map ($it) \n"); it * it }
    .filter { print("filter ($it) \n"); it %2 == 0 }
    .toList()

/* 결과
map (1)
filter (1)
map (2)
filter (4)
map (3)
filter (9)
map (4)
filter (16)
*/
```

리스트로 `map`과 `filter`를 적용하면 `map`을 모두 수행하고 `filter`를 수행하지만 시퀀스로 하면 각 원소별로 지연 계산한다.

```kotlin
listOf(1, 2, 3, 4).asSequence()
    .map { print("map ($it) \n"); it * it }
    .filter { print("filter ($it) \n"); it %2 == 0 }
    .find { it == 4 }

/* 결과
map (1)
filter (1)
map (2)
filter (4)
*/
```

각 수행하는 연산의 순서도 성능에 영향을 끼치니 잘 적용하자.

### 5.3.2 시퀀스 만들기

시퀀스를 만드는 다른 방법으로는 `generateSequence` 함수가 있다.

```kotlin
val naturalNumbers = generateSequence(0) { it + 1 }
val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
println(numbersTo100.sum())
```

시퀀스를 사용하는 일반적인 용례 중 하나는 객체의 조상으로 이뤄진 시퀀스를 만들어내는 것이다(?)

```kotlin
fun File.isInsideHiddenDirectory() = generateSequence(this) { it.parentFile }.any { it.isHidden }
```

어떤 파일의 상위 디렉터리를 뒤지면서 숨김 속성을 가진 디렉터리가 있는지 검사하는 함수다.
시퀀스를 사용하면 조건을 만족하는 디렉터리를 찾은 뒤에는 더 이상 상위 디렉터리를 뒤지지 않는다.

## 5.4 자바 함수형 인터페이스 활용

추상 메서드가 단 하나만 있는 인터페이스를 함수형 인터페이스 또는 SAM(Single Abstract Method) 인터페이스라고 한다.
코틀린은 함수형 인터페이스를 인자로 취하는 자바 메서드를 호출할 때 람다를 넘길 수 있게 해준다.

### 5.4.1 자바 메서드에 람다를 인자로 전달

```java
static void postponeComputation(int delay, Runnable computation) {}
```

`Runnable` 타입을 파라미터로 받는 자바 메서드에 코틀린 람다를 전달할 수 있다.

```kotlin
postponeComputation(1000) { println(42) }
```

`Runnable`을 구현하는 무명 객체를 만들어서 사용할 수도 있다.

```kotlin
postponeComputation(1000, object: Runnable {
    override fun run() {
        println(42)
    }
})
```

객체를 명시적으로 선언하면 호출 시점마다 새로운 객체를 만들어낸다.
하지만, 람다는 대응하는 무명 객체를 메서드를 호출할 때마다 반복 사용한다.

람다가 외부의 변수를 사용한다면, 호출시마다 새로운 인스턴스를 생성한다.

### 5.4.2 SAM 생성자: 람다를 함수형 인터페이스로 명시적으로 변경

함수형 인터페이스의 인스턴스를 반환하는 함수가 있다면, 람다를 직접 반환할 수 없고, 반환하고픈 람다를 SAM 생성자로 감싸야 한다.

```kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") }
}
```

## 5.5 수신 객체 지정 람다: with와 apply

### 5.5.1 with 함수

with는 수신 객체 지정 람다를 활용하여, 객체 이름을 반복하지 않고도 그 객체에 대한 다양한 연산을 수행할 수 있게 해준다.

```kotlin
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("\nNow I know the alphabet.")
    return result.toString()
}
```

with를 활용하면 다음과 같이 바꿀 수 있다.

```kotlin
fun alphabet(): String = with(StringBuilder()){
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet.")
    toString()
}
```

### 5.5.2 apply 함수

거의 with와 같으나, 항상 수신 객체를 반환한다.

```kotlin
fun alphabet2() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet.")
}.toString()
```
