# 코틀린 기초

## 기본 요소

#### 함수

```kotlin
fun functionName(parameter: Type): returnType {
    return value
}
```

- 코틀린에서 함수 parameter는 모두 immutable한 값(val이 생략된 형태)

#### 변수

```kotlin
var // 수정 가능한 변수
val // 읽기 전용 변수
const val // 컴파일 시에 값이 결정되는 상수 타입, 기본형 데이터 타입과 String만 사용 가능, 지역 변수로 선언 불가능
```

##### 식이 본문이 함수

```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

- 코틀린에서 `if`는 식이지 문이 아니다.

```kotlin
// ex)
var a = 5
var b = 3
var bigger = if (a > b) {
    a
} else {
    b
}
```

- 이렇게 써도 a를 반환타입으로 인식해 bigger엔 5가 들어간다.

#### 문자열 템플릿

```kotlin
"some $변수 string"
"some ${변수 + 1} value" // 윗줄처럼 쓰면 변수가 한글일 경우 오류 발생하니 아래처럼 쓰는 습관을 들이기
```

#### 클래스와 프로퍼티

**클래스 구조**

```kotlin
class ClassName {
    var value
    fun functionName() {
        // code
    }
}
```

### 생성자

```kotlin
class Person constructor(value: String) {
    // code
}

// 생성자에 접근 제한자나 다른 옵션이 없다면 생략 가능
class Person(value: String) {
    init {
        // code
    }
}

// init은 없어도 되나 parameter를 쓰고 싶다면 val을 붙여야 한다
class Person(val value: String) {
    fun process() {
        print(value)
    }
}

// secondary constructor
class Person {
    constructor (value: String) {
        // code
    }
}

// default 없어도 init 사용 가능
```

## Object

```kotlin
object Person {
    var name: String = "Name"
    fun printName() {
        Log.d("class", "${name}입니다")
    }
}
```

- object는 클래스와 다르게 앱 전체에 1개만 생성된다

### Companion object

```kotlin
class Animal {
    companion object {
        var name: String = "None"
        fun printName() {
            Log.d("class", "${name}입니다")
        }
    }
    fun walk() {
        Log.d("class", "walk walk...")
    }
}

Pig.name = "Name"
Pig.printName()

val pig1 = Pig()
pig1.walk()
```

## Data Class

- 간단한 데이터 저장 용도

```kotlin
data class Name (val param1: Type1, var param2: Type2)

var name = Name("param1", "param2")
name.param1 = param3 // 불가
name.param2 = param4

name.toString() // Name(param1=param1, param2=param4)
var newName = name.copy()
```

- 클래스와 동일하게 사용가능하다
- 네트워크를 통해 데이터를 주고받고나, 로컬 앱의 데이터베이스에서 데이터를 다루기 위한 용도로 사용

### 상속과 확장

```kotlin
// 상속되는 부모 클래스
open class ClassName {
    // code
}

// 상속 받는 자식 클래스
class ClassName2: ClassName() {
    // code
}

// 생성자 파라미터 전달
class ClassName3(value: String): className(value) {
    // code
}
```

- 오버라이드도 가능하지만 상속되는 부모 클래스의 메서드에도 `open` 키워드를 사용해야한다
- 상속받는 자식 클래스의 메서드에는 `override` 키워드를 사용해야한다

### 설계단계에서의 추상화

```kotlin
abstract // 키워드 메서드 앞에 붙여서 사용
```

### 접근 제한자

```kotlin
private // 다른 파일에서 접근 x
internal // 같은 모듈에 있는 파일만 접근
protected // private와 같으나 상속 관계에서 자식 클래스가 접근 가능
public // 제한 없이 모든 파일에서 접근 가능
```

### when 조건문

```kotlin
// 다른 언어의 switch문과 유사
var now = 10
when (now) {
    8 -> {
        plintln("8")
    }
    9 -> {
        plintln("9")
    }
    10, 11 -> {
        plintln("10 or 11")
    }
    in 12..19 -> {
        plintln("12~19")
    }
    else -> {
        plintln("else")
    }
}

// if문처럼 사용도 가능하다
var currentTime = 6
when {
    currentTime == 5 -> {
        plintln("5")
    }
    currentTime > 5 -> {
        plintln("5>")
    }
    else -> {
        plintln("5<")
    }
}
```

### 반복문

**for 문**

```kotlin
for (value in start..end) {
    // code
    // value에는 start부터 end까지 실행(end 포함)
}

for (value in start until end) {
    // code
    // value에는 start부터 end-1까지 실행
}

for (value in start..end step 3) {
    // code
    // value에는 start부터 3을 계속 더하면서 end까지
}

for (value in start downTo end) {
    // code
    // value에는 start부터 end 까지 점점 감소하면서
}

for (value in collection) {
    // list, set, map에 있는 엘리먼트 반복
}
```

**while 문**

```kotlin
// 일반적인 while문과 동일
// do while 문도 동일
```

### try, catch, finally

다른 부분 대부분 비슷하나 try를 식처럼 사용할 수 있다.

```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Interger.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        return
    }

    println(number)
}
```
