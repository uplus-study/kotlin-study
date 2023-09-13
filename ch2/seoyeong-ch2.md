# 2장. 코틀린 기초

## 기본 요소: 함수와 변수

### 함수
```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

- 함수 선언 : `fun` 키워드 사용
- 파라미터 이름 뒤에 해당 타입 지정, 변수도 마찬가지
- 최상위에 함수 정의 가능 (자바는 클래스 내부에 정의함)
- 배열도 일반 클래스와 마찬가지
- 세미클론(`;`) 필요없음.


>  #### 문(statement)과 식(expression)의 구분
>> #### 코틀린에서 if는 식!
>> - 식은 값을 만들어 내고 다른 식의 하위 요소로 계산 과정에 들어갈 수 있음. 문은 최상위 요소로 어떠한 값을 만들지 않음.
>> - 자바는 모든 제어 구조가 문!, 코틀린은 Loop를 제외하고 대부분 제어 구조가 식!

ex) 식이 본문인 함수, 타입 추론
```kotlin
fun max(a: Int, b: Int) = if(a>b) a else b
```

식이 본문인 함수는 반환 타입을 지정하지 않아도 본문 식을의 결과값으로 타입 지정 가능

### 변수
#### 변경 가능한 변수와 변경 불가능한 변수
- val(value) : 변경 불가능
- var(variable) : 변경 가능

```kotlin
val lauguages = arrayListOf("Java") // 불변 참조 선언
languages.add("Kotlin") // 참조가 가리키는 객체의 내부는 변경 가능


var a = 1
a = "string" // 컴파일 에러, var로 선언했지만 변수 타입은 고정되어 변하지 않음.

```

### 문자열 템플릿
```kotlin
fun main(args: Array<String>) {
    val name = "Seoyeong"
    
    println("${name} 반가워!") //중괄호로 변수명을 감싸서 사용
}
```

## 클래스와 프로퍼티
```java
// Java 코드
public class Person {
    private final String name;

    public Person(name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```
```kotlin
// Kotlin 코드
class Person(var name: String)
```

### 프로퍼티
#### Java
- 필드에 데이터 저장, 필드는 private로 선언
- 데이터에 접근하기 위해서 getter, setter와 같은 접근자 메소드 필요
- 필드와 접근자를 묶어서 property로 부름.

#### kotlin
- property를 언어 기본 기능으로 제공
- private 필드와 setter, getter를 기본적으로 제공
- 자바에 비해 코드 양이 축소됨.

```java
// Java 코드
person.setName("seoyeong")
// Kotlin 코드
person.name = "seoyeong"
```

#### 커스텀 접근자
```kotlin
class Rectangle(val height: Int, val width: Int) {
    val isSqure: Boolean
        // 프로퍼티 getter 정의
        get() {
            return height = width
        }
}
```

`isSquare` 프로퍼티에 값을 저장하기 위한 필드없이 값을 제공하기 위한 getter가 존재

### 코틀린 소스코드 구조: 디렉터리와 패키지

- 코틀린은 클래스, 함수 import 차이가 없고 모든 선언을 import로 가져올 수 있음.
- 자바에서는 디렉터리 구조가 패키지 구조를 따라가야하지만 코틀린에서는 꼭 그럴 필요는 없음.


## 선택 표현과 처리: enum과 when

### enum
 ```kotlin
 enum class Color(
    val r: Int, val g: Int, val b: Int
 ) {
    RED(255,0,0), BLUE(0,0,255);

    fun rgb() = (r*256+g)*256+b
 }
 ```
 - 코틀린에서 유일하게 세미클론(`;`) 필수

### when으로 enum 클래스 다루기
자바의 `switch` 같은 기능을 하지만 더 강력한 기능이다.

```kotlin
fun mix(c1: Color, c2: Color) = 
    when {
        (c1 == RED && c2 == YELLOW) -> ORANGE
        (c1 == YELLOW && c2 ==BLUE) -> GREEN

        else -> throw Exception(
            "Dirty color"
        )
    }
```
- 자바의 `switch`는 인자에 상수밖에 오지 못하지만 코틀린의 `when`은 인자로 객체가 올 수 있음.
- 자바는 `switch`문 각 분기에 `break`를 넣어야 하지만 `when`은 필요 업음.
- `when` 분기 조건에 인자가 없어도 됨. -> 불필요한 객체 생성 방지

### 스마트 캐스트: 타입 검사와 타입 캐스트를 조합

- `is`를 사용해서 변수 타입 검사
- `is` 검사 후에 다른 타입으로 캐스팅 안해도 컴파일할 때 캐스팅 됨.
- 해당 프로퍼티는 `val` 필수
- `val`여도 커스텀 접근자 사용하면 안됨. (변경의 여지가 있으므로)
- 명시적으로 타입 캐스팅을 한다면 `as` 사용

## 대상을 이터레이션: while과 for 루프
### while은 자바와 동일
### 수에 대한 이터레이션: 범위한 수열
- `..` 연산자를 사용하여 for문 구현 가능
- `..` 은 끝 값을 포함. 포함하지 않는 범위를 원하면 `until` 사용
(x in 0 until 10 -> 0..9)
```kotlin
for (i in 100 downTo 1 step 2) {
    print(i)
}

// 결과
100 98 96 ...
```

### 맵에 대한 이터레이션
```kotlin
fun main() {
    val binaryReps = TreeMap<Char, String>()

    for (c in 'A'..'F') {
        val binary = Integer.toBinaryString(c.toInt())
        binaryReps[c] = binary
    }

    for((letter, binary) in binaryReps) [
        println("$letter = $binary")
    ]
}
```
- 자바에서는 map에서 map.get(key), map.put(value) 형식으로 값을 저장하고 가져올 수 있었음. 코틀린에서는 map[key], map[key]=value로 값을 저장하고 가져올 수 있음.
- 코틀린에서는 컬렉션에서도 인덱스 저장을 위한 변수 없이 컬렉션을 이터레이션할 수 있음.

### in으로 컬렉션이나 범위의 원소 검사
- `in`, `!in` 연산자로 값이 범위에 속하는지 판단할 수 있음.
```kotlin
fun isLetter(c: Char) = c in 'a'..'z'
fun isNotDigit(c: Char) = c !in '0'..'9'
```

## 코틀린의 예외처리
- 예외처리는 자바와 비슷함.

### try, catch, finally
- 자바의 경우 `throws IOException` 구문이 체크 예외로 인해서 필요함
- 코틀린은 체크 예외, 언체크 예외를 따로 구분하지 않음.
- 코틀린에서는 자바와 다르게 함수가 던지는 예외를 선언하지 않아도 됨.


### try를 식으로 사용
```kotlin
fun readNumber(reader: BufferedReader) {
    val num = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        return
    }
    println(num)
}
```
- 코틀린의 `try`도 식
- `try` 값을 변수에 대입 가능