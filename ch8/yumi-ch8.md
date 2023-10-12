# 고차 함수: 파라미터와 반환 값으로 람다 사용

## 고차 함수 정의

고차함수란 다른 함수를 인자로 받거나 함수를 반환하는 함수

### 함수 타입

함수 타입의 정의를 위해서는 파라미터 타입을 괄호 안에 넣고, 그 뒤에 화살표(->)를 추가한 다음 반환 타입을 기술하면 된다.

![img](https://github.com/uplus-study/kotlin-study/assets/66773030/1ad15d68-9359-4e09-acf5-8075017886f6)

null이 될 수 있는 함수 타입을 표현하려면 파라미터 타입과 반환 타입 사이에 물음표(?)를 넣으면 된다.

```kotlin
var canReturnNull: (Int, Int) -> Int? = { x, y -> null }
```
함수 전체가 널이 될 수 있는 타입을 선언하기 위해서는 함수 타입을 괄호로 감싸고 그 뒤에 물음표를 붙여야만 한다.

```kotlin
var funOrNull: ((Int, Int) -> Int)? = null
```
### 인자로 받은 함수 호출
![img_1](https://github.com/uplus-study/kotlin-study/assets/66773030/bd9d4705-472d-47a9-aaab-b9842fe6f8b6)

### 자바에서 코틀린 함수 타입 사용
컴파일 된 코드 안에서 함수 타입은 일반 인터페이스로 변환, 인터페이스에 invoke 메서드를 호출하여 함수를 실행할 수 있다.

```kotlin
//코틀린 선언
fun processTheAnswer(f: (Int) -> Int) {
    println(f(42))
}
```
함수 타입을 사용하는 코틀린 함수를 자바에서도 쉽게 호출 가능
```java
//자바 호출
>>> processTheAnswer(number -> number + 1);
```
자바 8 이전의 자바에서는 필요한 FunctionN 인터페이스의 invoke 메서드를 구현하는 무명 클래스를 넘기면 된다.
```java
>>> processTheAnswer(new Function1<Integer, Integer>() {
    @Override
    public Integer invoke(Integer number) {
        System.out.println(number);
        return number + 1;
    }
});
```
반환 타입이 Unit인 함수나 람다를 자바로 작성할 수 있다. 반환타입을 void로 넘길 수 없고, 명시적으로 반환해주어야 한다.

### 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터
함수 타입의 파라미터에 디폴트 값으로 람다식을 넣을 수 있다.
```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
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
널이 될 수 있는 함수 타입 파라미터를 사용하여 함수를 호출
```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
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

### 함수를 함수에서 반환
함수를 반환하는 함수를 선언하려면 반환 타입에 함수 타입을 넣으면 된다.
```kotlin
enum class Delivery { STANDARD, EXPEDITED }
class Order(val itemCount: Int)
fun getShippingCostCalculator(
    delivery: Delivery
): (Order) -> Double {
    if(delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }
    return { order -> 1.2 * order.itemCount }
}
```
```kotlin
>>> val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
>>> println("Shipping costs ${calculator(Order(3))}")
Shipping costs 12.3
```

### 람다를 활용한 중복 제거
```kotlin
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
    filter(predicate)
        .map(SiteVisit::duration)
        .average()
```
```kotlin
>>> println(log.averageDurationFor { it.os in setOf(OS.ANDROID, OS.IOS) })
```
```kotlin
>>> println(log.averageDurationFor { it.os == OS.IOS && it.path == "/signup" })
```

## 인라인 함수 : 람다의 부가 비용 없애기
람다를 사용하면 코드를 간결하게 만들 수 있지만, 람다를 사용하면서도 성능을 유지하고 싶다면 인라인 함수를 사용해야 한다.

### 인라이닝이 작동하는 방식
함수를 호출하는 코드를 함수를 호출하는 바이트코드 대신에 함수 본문을 번역한 바이트코드로 컴파일 한다.
- synchronized 함수의 본문 뿐만 아니라 전달된 람다의 본문도 함께 인라이닝 된다. 
- 인라인 함수를 호출하면서 람다를 넘기는 대신에 함수 타입의 변수를 넘길 수도 있다. 
  - 이때 함수를 호출하는 코드 위치에서는 변수에 저장된 람다 코드를 알 수 없으므로 람다 본문은 인라이닝 되지 않고, synchronized 함수의 본문만 인라이닝 된다.

### 인라인 함수의 한계
- 인라인 함수의 본문에서 람다식을 바로 호출하거나 람다 식을 다른 변수에 저장한 다음에 호출하는 경우에는 인라이닝을 할 수 없다.
- 인라이닝 하면 안 되는 람다를 파라미터로 받는다면 noinline 변경자를 파라미터 이름 앞에 붙여서 인라이닝을 금지할 수 있다.

### 컬렉션 연산 인라이닝
- 표준라이브러리 filter만 사용할 경우 직접 구현한 것과 성능이 유사하다.
- filter와 map을 연계한 경우 리스트를 걸러낸 결과를 저장하는 중간 리스트를 만든다. 
  - asSequence를 사용하면 중간 리스트를 만들지 않고도 연산을 수행할 수 있다.
  - 시퀀스 연산에서는 람다가 인라이닝되지 않기 때문에 크기가 작은 경우 성능이 더 나쁠 수 있다.

### 함수를 인라인으로 선언해야 하는 경우
- 람다를 인자로 받는 함수를 자주 호출하는 경우

### 자원 관리를 위해 인라인된 람다 사용
- 어떤 작업을 하기 전에 자원을 획득하고 작업을 마친 후 자원을 해제한다.
- 코틀린에서는 withLock 함수 제공
~~~kotlin
val l: Lock = ...
l.withLock {
    // ...
}

fun <T> Lock.withLock(action: () -> T): T {
    lock()
    try {
        return action()
    } finally {
        unlock()
    }
}
~~~
- 코틀린에서는 use 함수가 자바의 try-with-resource와 비슷하게 동작한다.
- use 함수는 닫을 수 있는 자원에 대한 확장 함수며, 람다를 인자로 받는다.
- return은 넌로컬 return으로 readFirstLIneFromFile 함수를 끝내면서 값을 반환한다.
~~~kotlin
fun readFirstLineFromFile(path: String): String {
    BufferedReader(FileReader(path)).use { br ->
        return br.readLine()
    }
}
~~~

## 고차 함수 안에서 흐름 제어
### 람다 안의 return문: 람다를 둘러싼 함수로부터 반환
넌로컬 return : 람다 안에서 return을 사용하면 람다를 둘러싼 함수가 아니라 람다를 호출하는 함수가 실행을 끝내고 반환
- 람다를 인자로 받는 함수가 인라인 함수인 경우 
~~~kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if(it.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}
~~~

### 람다로부터 반환: 레이블을 사용한 return
- 람다 안에서 로컬 return은 for 루프의 break 역할과 비슷
- return으로 실행을 끝내고 싶은 람다 식 앞에 레이블을 붙이고, return 키워드 뒤에 레이블을 추가

![img_2](https://github.com/uplus-study/kotlin-study/assets/66773030/271ae6ee-6db0-4d6a-9457-b9d21e4c4a39)

### 무명 함수: 기본적으로 로컬 return
- 무명 함수는 코드 블록을 함수에 넘길 때 사용하는 다른 방법

![img_3](https://github.com/uplus-study/kotlin-study/assets/66773030/93260e87-6d34-4046-8249-680e219494ac)
