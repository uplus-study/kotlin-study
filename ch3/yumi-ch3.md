# Ch3. 함수 정의와 호출

## 코틀린에서 컬렉션 만들기
코틀린은 자체 컬렉션은 제공하지 않고 표준 자바 컬렉션을 사용한다.
자바보다 더 많은 기능을 쓸 수 있다.

## 함수를 호출하기 쉽게 만들기
### 이름 붙인 인자
코틀린에서는 함수를 호출할 때 인자 중 일부의 이름을 명시할 수 있다.
~~~kotlin
joinToString(collection, separator= "", prefix= " ", postfix= ".")
~~~
### 디폴트 파라미터 값
코틀린에서는 함수 선언에서 파라미터의 디폴트 값을 지정할 수 있으므로 오버로드를 피할 수 있다.
~~~kotlin
fun <T> joinToString(
    collections: Collection<T>,
    separator: String = "",
    prefix: String = "",
    postfix: String = ""
): String
~~~
일반 호출문법을 사용하려면 함수를 선언할 때와 같은 순서로 인자를 지정해야한다. 그런 경우에 일부를 생략하면 뒷부분의 인자들이 생략된다.

>디폴트 값과 자바
>>자바에서 코틀린 디폴트 파라미터를 사용하더라도 직접 파라미터를 명시해야한다.
> @JvmOverloads 애노테이션을 함수에 추가하면 코틀린 컴파일러가 자동으로 맨 마지막 파라미터로 부터 파라미터를 하나씩 생략한 오버로딩한 자바 메소드를 추가해준다.

## 정적인 유틸 클래스 없애기: 최상위 함수와 프로퍼티
함수를 최상위 수준에 위치시키면 정의된 패키지를 임포트해야하지만 유틸리티 클래스를 만들 필요가 없다.
ex) strings 패키지에 joinToString 함수 생성
~~~kotlin
package strings

fun joinToString(...).: String { ... }
~~~

### 최상위 프로퍼티
- 프로퍼티도 파일의 최상위 수준에 놓을 수 있다.
- 최상위 프로퍼티는 정적 필드에 저장된다.
~~~kotlin
var opCount = 0 // 최상위 프로퍼티
fun performOperation() {
    opCount++
}
fun reportOperationCount() {
    // 최상위 프로퍼티의 값을 읽는다.
    println("Operation performed $opCount times")
}
~~~

## 메서드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티
확장함수 : 어턴 클래스의 멤버 메서드인 것 처럼 호출할 수 있지만 그 클래스 밖에 선언된 함수
- 수신 객체 타입 : 클래스 이름
- 수신 객체 : 확장함수가 호출되는 대상이 되는 값
~~~
// String : 수신 객체 타입
// "Kotlin" : 수신 객체
pinrtln("Kotlin".lastChar())
~~~
- 확장함수가 캡슐화를 깨지 않음, private이나 protected 멤버에도 접근 불가

### 임포트와 확장 함수
- 확장함수를 정의 하더라도 모든 소스 코드에서 사용할 수 있지 않음, 임포트 필요
- * 사용 가능
- as 키워드를 사용하여 다른 이름으로 바꿀 수 있음

### 확장 함수로 유틸리티 함수 정의
~~~kotlin
fun Collection<String>.join (
    seprarator: String = ", ",
    prefix: String = "",
    postfix: String = ""
) = joinToString(separator, prefix, postfix)
~~~

### 확장 함수는 오버라이드 할 수 없다.
- 확장 함수는 클래스 밖에 선언되어 있기 때문에 오버라이드 할 수 없다.
- 확장 함수는 정적 메소드로 컴파일 되기 때문에 확장 함수를 호출하는 변수의 정척 타입에 의해 어떤 확장 함수가 호출될지 결정된다.
- 어떤 클래스를 확장한 함수와 그 클래스의 멤버 함수의 이름과 시그니처가 같다면 확장 함수가 아니라 멤버 함수가 호출된다.

### 확장 프로퍼티
- 게터 정의 필요
~~~kotlin
var StingBuilder.lastChar: Char
    get() = get(length - 1) // 프로퍼티 게터
    set(value: Char) { //프로퍼티 세터
        this.setCharAt(length - 1, value)
    }
~~~

## 컬렉션 처리 : 가변 길이 인자, 중위 함수 호출, 라이브러리 지원
- vararg 키워드 : 호출 시 인자가 달라질 수 있는 함수 정의
- 중위 함수 호출 구문을 사용하면인자가 하나뿐인 메서드를 간편하게 호출
- 구조 분해 선언을 활용하면 복합적인 값을 분해해서 여려 변수에 나눠 담을 수 있다.

### 가변 길이 인자 함수 : 인자의 개수가 달라질 수 있는 함수 정의
~~~kotlin
fun listOf<T>(vararg values: T): List<T> { ... }
~~~
이미 배열에 들어있는 원소를 가변 인자로 넘길 때에는 배열 앞에 *를 붙여야 한다.
~~~kotlin
val list = listOf("args: ", *args)
~~~

### 중위 호출
- 인자가 하나뿐인 메소드를 호출할 때 사용
- infix 변경자를 함수 선언 앞에 추가
~~~kotlin
infix fun Any.to(other: Any) = Pair(this, other)
~~~

### 구조 분해 선언
- 복합적인 값을 분해해서 여러 변수에 나눠 담을 수 있다.
~~~kotlin
val (number, name) = 1 to "one"
~~~
- 루프에서도 사용 가능
~~~kotlin
for ((index, element) in collection.withIndex()) {
    println("$index: $element")
}
~~~

## 문자열과 정규식 다루기
코틀린 문자열은 자바 문자열과 같음 서로 변환 필요 없음

### 문자열 나누기
정규식을 파라미터로 받아와서 string을 split할 수 있다.
~~~kotlin
"12.345-6.A".split("\\.|-".toRegex())
~~~

### 정규식과 3중 따옴표로 묶은 문자열
일반 문자열에서 마침표를 이스케이프하려면 \\.라고 써야하지만 3중 따옴표 문자열에서는 \.로 사용 가능
~~~kotlin
fun parsePath(path: String) {
    val regex = """(.+)/(.+)\.(.+)""".toRegex()
    val matchResult = regex.matchEntire(path)
    
    if (matchResult != null) {
        val (directory, filename, extension) = matchResult.destructured
        println("Dir: $directory, name: $filename, ext: $extension")
    }
}
~~~

### 여러 줄 3중 따옴표 문자열
3중 따옴표 문자열에는 줄 바꿈을 표현하는 아무 문자열이나 이스케이프 없이 그냥 들어간다.
~~~kotlin
val kotlinLogo = """| //
                   .|//
                   .|/ \"""
~~~
trimMargin을 사용하면 문자열 직전의 공백을 제거
~~~kotlin
>>> println(kotlinLogo.trimMargin("."))
~~~

## 코드 다듬기: 로컬 함수와 확장
- 로컬 함수를 통해 중복을 제거할 수 있음
- 확장 함수를 통해 코드를 더 간결하게 만들 수 있음
- 중첩된 함수의 깊이가 깊어지면 코드를 읽기 어려우므로 한 단계만 중첩 권장
~~~kotlin
class User(val id: Int, val name: String, val address: String)

fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
               "Can't save user $id: empty $fieldName")
        }
    }

    validate(name, "Name")
    validate(address, "Address")
}

fun saveUser(user: User) {
    user.validateBeforeSave()

    // Save user to the database
}

fun main(args: Array<String>) {
    saveUser(User(1, "", ""))
}
~~~

## 요약
- 코틀린은 자체 컬랙션 클래스르 정의하지 않지만 자바 클래스를 확장해서 더 풍부한 API 제공
- 함수 파라미터의 디폴트 값을 정의하면 오버로드를 피할 수 있다.
- 이름붙인 인자를 사용하면 함수 인자가 많을 때 가독성을 높일 수 있다.
- 코틀린 파일에서 최상위 함수와 프로퍼티를 직접 선언할 수 있다.
- 확장 함수와 프로퍼티를 사용하면 외부 라이브러리에 정의된 클래스를 포함한 모든 클래스의 API를 확장할 수 있다.
- 중위 호출을 통해 인자가 하나 밖에 없는 메서드나 확장 함수를 깔끔하게 호출 할 수 있다.
- 코틀린은 정규식과 일반 문자열을 처리할 때 유용한 다양한 문자열 처리 함수를 제공한다.
- 3중 따옴표 문자열을 사용하면 이스케이프가 필요한 문자열을 깔끔하게 표현할 수 있다.
- 로컬 함수를 사용하면 코드를 깔끔하게 유지하면서 중복을 제거할 수 있다.