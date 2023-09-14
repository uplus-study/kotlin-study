# Ch2. 코틀린 기초

## 기본 요소 : 함수와 변수
- 함수 선언 시 fun 키워드 사용
- 파라미터 이름 뒤에 파라미터 타입을 쓴다.
- 함수를 최상위 수준에 정의할 수 있다.
- 끝에 세미콜론(;) 생략가능

### 함수
함수 선언 : fun 다음에 함수 이름을 쓰고, 괄호 안에 파라미터를 넣는다. 반환 타입은 파라미터 닫는 괄호 뒤에 위치 콜론(:) 으로 구분

### 문과 식의 구분
- 문 : 자신을 둘러싸고 있는 가장 안쪽 블록의 최상위 요소로 존재, 값을 만들지 않는다. 
  - 자바는 대부분 문
- 식 : 값을 만들어 내며 다른 식의 하위 요소로 계산에 참여
  - 코틀린은 대부분 식

### 식이 본문인 함수
- 블록이 본문인 함수 : 본문이 중괄호로 둘러싸인 함수
- 식이 본문인 함수 : 등호와 식으로 이루어진 함수
```kotlin
fun max(a: Int, b: Int): Int = if (a > b) a else b
```

### 변수
- 코틀린에서는 타입을 생략할 수 있음
- 타입을 지정하지 않을 경우 컴파일러가 초기화 식을 분석하여 변수 타입을 지정
- 최기화 식이 없을 경우는 타입 지정 필요

##### 변경 가능한 변수와 변경 불가능한 변수
기본적으로 val 사용 권장, 필요할 때만 var 사용
- val : 변경 불가능한 변수
  - 참조는 불변, 참조가 가리키는 객체의 내부 값은 변경 가능
- var : 변경 가능한 변수
  - 값은 변경 가능, type은 변경능불가

### 문자열 템플릿
- 문자열 템플릿 : 문자열 안에 변수나 식의 값을 나타내기 위해 $문자를 사용하는 방법
- $ 문자를 문자열에 넣고 싶으면 \를 붙여준다.
- 한글에 $를 사용하고 싶으면 ${}로 감싸준다.

## 클래스와 프로퍼티
- 자바에 있었던 생성자 코드 생략 가능
- 값 객체(코드가 없이 데이터만 저장하는 클래스)를 간결하게 표현
- 기본 가시성이 public이므로 생략가능

### 프로퍼티
자바의 필드와 접근자 메서드를 대신함
- val : 읽기 전용 프로퍼티
- var : 쓰기 전용 프로퍼티

#### 커스텀 접근자
클라이언트가 프로퍼티에 접근 할 때마다 게터가 프로퍼티 값을 계산하게 할 수 있음
~~~kotlin
class Rectangle(val height: Int, val width: Int) {
val isSquare: Boolean
get() = height == width
}
~~~

#### 코틀린 소스코드 구조 : 디렉터리와 패키지
- 같은 패키지에 속해 있다면 다른 파일에서 정의한 선언을 사용가능
- 다른 패키지에 정의한 선언은 임포트를 통해 선언 불러와야 함
- 코틀린에서는 어느 디렉토리에 소스코드를 위치시키든 관계 없으나, 패키지별로 구성하는 것을 권장

## 선택 표현과 처리 enum과 when
### enum 클래스 정의
- enum 클래스 안에 프로퍼티나 메서드를 정의 가능
- enum 클래스 안에 메서드를 정의하는 경우 세미콜론(;)필수
~~~kotlin
enum class Color(
        val r: Int, val g: Int, val b: Int
) {
  RED(255, 0, 0), ORANGE(255, 165, 0),
  YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
  INDIGO(75, 0, 130), VIOLET(238, 130, 238);

  fun rgb() = (r * 256 + g) * 256 + b
}
~~~

### when으로 enum 클래스 다루기
- when은 자바의 switch와 유사
- break 생략
~~~kotlin
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
    }
~~~
when에서는 분기 조건에 임의의 객체를 허용한다.
~~~kotlin
fun mix(c1: Color, c2: Color) =
        when (setOf(c1, c2)) {
            setOf(RED, YELLOW) -> ORANGE
            setOf(YELLOW, BLUE) -> GREEN
            setOf(BLUE, VIOLET) -> INDIGO
            else -> throw Exception("Dirty color")
        }
~~~

### 스마트 캐스트 : 타입 검사와 타입 캐스트를 조합
is를 사용해 검사하면 컴파일러가 자동으로 캐스팅을 수행함
- ex) (1+2)+4 계산하는 함수
- 어떤 식이 수라면 그 값을 반환
- 어떤 식이 합계라면 좌항과 우항을 더한 후 그 결과를 반환
~~~kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun evalWithLogging(e: Expr): Int =
    when (e) {
        is Num -> {
            println("num: ${e.value}")
            e.value
        }
        is Sum -> {
            val left = evalWithLogging(e.left)
            val right = evalWithLogging(e.right)
            println("sum: $left + $right")
            left + right
        }
        else -> throw IllegalArgumentException("Unknown expression")
    }
~~~
~~~
    println(evalWithLogging(Sum(Sum(Num(1), Num(2)), Num(4))))
~~~

## 대상을 이터레이션: while과 for 루프
### while
자바와 동일
### for
- 코틀린에서는 range를 사용, 시작값과 끝 값을 연결하여 범위를 만든다.
- ex) for(i in 1.. 10) : 1부터 10까지반복
- ex) for(i in 1 until 10) : 1부터 9까지 반복(우항 값 -1까지)

### 맵에 대한 이터레이션
~~~kotlin
fun main(args: Array<String>) {
    val binaryReps = TreeMap<Char, String>()

    for (c in 'A'..'F') {
        val binary = Integer.toBinaryString(c.toInt())
        binaryReps[c] = binary
    }

    for ((letter, binary) in binaryReps) {
        println("$letter = $binary")
    }
}
~~~

### in으로 컬렉션이나 범위의 원소 검사
~~~kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'
~~~

## 코틀린의 예외 처리
- 인스턴스 생성시에 new 제외
- 다른 식에 포함 가능

### try, catch, finally
~~~kotlin
fun readNumber(reader: BufferedReader): Int? {
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    }
    catch (e: NumberFormatException) {
        return null
    }
    finally {
        reader.close()
    }
}
~~~
- throws절이 코드에 없다.
- 자바에서는 체크 예외를 명시적으로 처리하는 것이 필요했으나, 코틀린에서는 예외 처리를 강제하지 않는다.
- 코틀린에서는 체크 예외와 언체크 예외를 구별하지 않음

### try를 식으로 사용
~~~kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    }
    catch (e: NumberFormatException) {
        return
    }
    println(number)
}
~~~
- try의 본문은 반드시 중괄호 {}로 둘러싸야 한다.
- 내부에 여러 문장이 있으면 마지막 식의 값이 전체 결과 값이다.

### 요약
- 함수를 정의할 때 fun 키워드를 사용한다. val은 읽기전용, var는 쓰기 전용 변수를 선언할 때 사용한다.
- 문자열 템플릿을 사용하면 문자열 안에 변수나 식의 값을 넣을 수 있다.
- 코틀린에서 값 객체 클래스를 간결하게 표현할 수 있다.
- if는 코틀린에서 식이며, 값을 만들어낸다.
- 코틀린의 when은 자바의 switch를 대신하며, 더 다양한 기능을 제공한다.
- 스마트 캐스트를 통해 is로 검사한 객체를 자동으로 캐스팅할 수 있다.
- 코틀린의 for 루프는 자바의 for-each 루프를 대신한다.
- 1..5와 같은 식은 범위르 생성하며 for 루프나 in, !in에서 사용할 수 있다.
- 코틀린 예외처리는 자바와 유사하나 예외처리를 강제하지 않는다.