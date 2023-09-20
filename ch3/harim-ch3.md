# 함수 정의와 호출

### 컬렉션

```Kotlin
val set = hashSetOf(1, 7, 53) //java.util.HashSet
val list = arrayListOf(1, 7, 53) //java.util.ArrayList
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three") //java.util.HashMap
```

> to 는 키워드가 아니라 일반 함수임

- 코틀린은 자신만의 컬렉션 기능 X
  - 표준 자바 컬렉션 활용으로 자바 코드와 상호작용 쉬움
  - 코틀린에서 더 많은 기능 제공 ex) `last()`, `max()`, ..

### 원소 호출
- 자바 컬렉션에는 `toString()` 구현이 포함됨
  - 출력 형태 고정
- 코틀린에서 `joinToString` 을 통해 출력
  - 제네릭함 -> 어떤 타입의 컬렉션이든 처리 가능
  
```Kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String
): String {
    val result = StringBuilder(prefix)
    for((index, element) in collection,withInIndex() {
        if(index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```


- 호출할 때 각 인자가 어떤 역할을 하는지 구분 X => 함수 호출할 때 전달하는 인자 중 일부 이름 명시 가능
`joinToString(collection, " ", " ", ".")`
    -> `joinToString(collection, separator = " ", prefix = " ", postfix = ".")` 

- 많은 오버로딩 함수으로 인한 이름/타입 반복, 호출하는 함수 모호함 => 파라미터 디폴트 값 지정 가능
    ```Kotlin
    fun <T> joinToString(
        collection: Collection<T>,
        separator: String = ", ",
        prefix: String = "", 
        postfix: String = ""
    ): String 
    ```
    > 자바에서 호출하는 경우는 모든 인자 명시해야 함, 이때 좀 더 편하게 코틀린 함수 호출하고 싶다면 `@JvmOverloads` 함수에 추가하여 코틀린 컴파일러가 자동으로 맨 마지막 파라미터로부터 파라미터 하나씩 생략한 오버로딩한 자바 메서드 추가할 수 있음

- 함수를 클래스 안에 선언할 필요가 없음 => 최상위 함수와 프로퍼티
  - 어느 한 클래스에 포함시키기 어려운 코드 발생
  - 정적인 메서드를 모아두는 역할을 담당하는 클래스가 발생함 ex) JDK의 Collections 클래스
  - 코틀린에서는 함수를 최상위 수준에 위치시킬 수 있음
  - 다른 패키지에서 함수를 사용하고 싶다면 함수가 정의된 패키지 임포트 필요 
    > 코틀린 최상위 함수가 포함된 클래스 이름 바꾸고 싶다면 `@JvmName` 추가 <br>
    > 파일의 맨 앞, 패키지 이름 선언 이정에 위치해야 함
  - 최상위 프로퍼티 (정적 필드)
    - `const` 변경자 -> `public static final` 

### 확장 함수 (extension function)
  - 클래스의 멤버 메서드인 것처럼 호출하지만 클래스 밖에 선언된 함수
  - 함수 이름 앞에 **확장할 클래스 이름 추가** => 수신 객체 타입 (receiver type)
  - 확장 함수가 호출되는 대상이 되는 값 => 수신 객체 (receiver object)
    ```Kotlin
    fun String.lastChar(): Char = this.get(this.length - 1)
    
    println("kotlin".lastChar()) //수신객체타입: String, 수신객체: "kotlin"
    ```
  - 확장 함수 안에서는 클래스 내부에서만 사용할 수 있는 private/protected 멤버 사용 X
  
- 임포트
  - 확장 함수를 사용할 때 임포트 필요
  - 한 파일 안에서 다른 여러 패키지에 속해있는 이름 같은 함수 가져와 사용하는 경우 이름 바꿔서 임포트 가능 (as)
  - 코틀린 문법상 확장 함수는 짧은 이름 사용

- 자바에서 호출
  - 확장 함수는 수신 객체를 첫 번째 인자로 받는 정적 메서드 -> 자바에서 사용 쉬움
  - 첫 번째 인자로 수신 객체를 넘기기만 하면 됨
  
- 오버라이드 X
  - 클래스의 일부가 아니기 때문에 이름, 파라미터가 같은 확장 함수를 기반 클래스와 하위 클래스에 대해 정의해도 실제로는 확장 함수가 호출할 때 수신 객체로 지정한 변수의 정적 타입에 의해 어떤 함수가 호출될지 결정
  - 첫 번째 인자가 수신 객체인 정적 자바 메서드로 컴파일하기 때문!

### 확장 프로퍼티
- 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API 추가 가능
- 상태를 가지지 않음 <- 기존 클래스의 인스턴스 객체에 필드 추가할 방법 X
- 뒷받침하는 필드가 없기 때문에 게터 정의 필요
- 초기화 값을 담을 장소가 없기 때문에 초기화 코드 사용 X
```Kotlin
val String.lastChar: char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }
```

### 컬렉션 처리

- 확장 함수를 통해 자바 라이브러리 클래스의 인스턴스인 컬렉션에 대해 새로운 기능 추가 가능
  - `last`, `max` 
- 코틀린 표준 라이브러리는 수많은 확장 함수 포함
- 가변 인자 함수
  - 가변 길이 인자 `varargs`: 메서드를 호출할 때 원하는 개수만큼 값을 인자로 넘기면 컴파일러가 배열에 그 값들을 넣어주는 기능
  - 파라미터 앞에 `vararg` 변경자 사용
  - 배열을 명시적으로 풀어서 배열의 각 원소가 인자로 전달되게 해야 함 -> 스프레드 연산자 `spread` 
    ```Kotlin
    fun main(args: Array<String>) {
      val list = listOf("args: ", *args)
      print(list)
    }
    ```
- 중위 호출 `infix call` 
  - `to` 
  - 수신 객체와 유일한 메서드 인자 사이에 메서드 이름을 넣음 
  - 인자 하나뿐인 일반 메서드나 확장 함수에 사용
  - 함수 선언 앞에 `infix` 변경자 사용
- 구조 분해 선언 `destructuring declaration` 
  - `val (number, name) = 1 to "one"` 

### 문자열과 정규식 다루기
  
- 문자열 나누기
  - 자바에서 `split(".")` 의 경우 빈 배열 반환 -> 구분 문자열은 실제로 정규식이기 때문
    > . 은 정규식에서 모든 문자를 의미
  - 코틀린에서는 `split` 확장 함수 제공
    - 정규식을 파라미터로 받는 함수는 Regex 타입의 값을 받음
    - 전달하는 값 타입에 따라 정규식/일반 텍스트 중 어느 것으로 문자열 분리하는지 쉽게 알 수 있음

- 정규식과 3중 따옴표로 묶은 문자열
  - 파일 전체 경로명을 `디렉터리, 파일 이름, 확장자` 로 구분
  - 코틀린에서는 정규식 사용하지 않고도 쉽게 문자열 파싱 가능
  - 3중 따옴표 문자열(""") 에서는 문자 이스케이프 필요 X

- 여러 줄 3중 따옴표 문자열
  - 문자열 이스케이프 해결
  - 줄 바꿈 표현 쉬움
  - 문자열 템플릿(`${}`) 사용 가능
    - `val price = """${'$'}99.9"""` 
    - $ 를 3중 따옴표 문자열 안에 넣을 수 없음
  > 여러 줄 문자열을 보기 쉽게 하기 위해서 들여쓰기를 하고 들여쓰기 끝부분을 특별한 문자로 표시 후 `trimMargin` 사용해서 공백 및 문자 제거

### 로컬 함수와 확장
- 자바에서 중복코드를 없애기 위해서 메서드 추출 리팩토링 적용
  - 클래스 안에 작은 메서드 많아지고 메서드 사이 관계 파악이 어려움
  - 추출한 메서드를 별도 inner class에 넣을 수 있지만 불필요한 준비 코드 늘어남
- 코틀린에서는 함수에서 추출한 함수를 원 함수 내부에 중첩 가능 => 로컬 함수
  - 문법적 부가 비용 X, 깔끔하게 코드 조직 O
  ```Kotlin
  // 변경 전
  class User(val id: Int, val name: String, val address: String)
  fun saveUser(user: User) {
    if(user.name.isEmpty()) {
      throw IllegalArgumentException(
        "Can't save user ${user.id}: empty Name"
      )
    }
    if(user.address.isEmpty()) {
      throw IllegalArgumentException(
        "Can't save user ${user.id}: empty Address"
      )
    }

    ...
  }

  // 변경 후 - 로컬 함수 사용
  fun saveUser(user: User) {
    fun validate(user: User,
                value: String,
                fieldName: String) {
      if(value.isEmpty()) {
        throw IllegalArgumentException(
          "Can't save user ${user.id}: empty $fieldName"
        )
      }
    }

    validate(user, user.name, "Name")
    validate(user, user.address, "Address")

    ...
  }

  // 변경 후 - User 파라미터 삭제
    fun saveUser(user: User) {
      fun validate(value: String, fieldName: String) {
        if(value.isEmpty()) {
          throw IllegalArgumentException(
            "Can't save user ${user.id}: empty $fieldName" //바깥 파라미터 접근 가능
          )
        }
      }

    validate(user.name, "Name")
    validate(user.address, "Address")

    ...
  }

  // 변경 후 - 검증 로직을 User 클래스를 확장한 함수로 변경
  fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
      if(value.isEmpty()) {
        throw IllegalArgumentException(
          "Can't save user ${user.id}: empty $fieldName" //바깥 파라미터 접근 가능
        )
      }
    }

  validate(user.name, "Name")
  validate(user.address, "Address")
  }

  fun saveUser(user: User) {
    user.validateBeforeSave()
    
    ...
  }
  ```