# 코틀린 기초

### 함수
- fun 키워드
- 식이 본문인 함수는 return, 중괄호를 제거한 후 등호를 붙여 사용 가능, 이때 반환 타입도 생략 가능

### 변수
- 타입 지정 생략 가능
  - 타입이 없다면 초기화 필요 <- 컴파일러가 타입 추론하기 위해서
- '키워드'로 시작
  - `val`: 변경 불가능 참조 저장, 초기화 후 재대입 X, final
  - `var`: 변경 가능한 참조 저장, 일반 변수

> `val` 키워드로 불변 변수 선언 후 변경이 필요한 경우만 `var` 사용
>
> `val` 참조는 불변이지만 참고의 객체 내부 값은 변경 가능
> 
> `var` 변수 타입은 고정

```Kotlin
val answer = 42
val answer: Int = 42
``` 

### 문자열 템플릿
```Kotlin
println("Hello, $name!")
print("Hello, ${person.name}")

```
- 자바 문자열 접합 연산(+) 과 동일하지만 간결함
- $ 뒤에 변수, $ 뒤에 중괄호로 복잡한 식 가능


### 클래스
```Kotlin
class Person(val name: String)
```
- 값 객체: 코드가 없이 데이터만 저장하는 클래스
- 클래스의 기본 가시성은 public
  
### 프로퍼티
- 프로퍼티 = 필드 + 접근자
- `val` 선언된 프로퍼티: 읽기 전용 => getter O, setter X
- `var` 선언된 프로퍼티: 변경 가능 => getter O, setter O
> 이름이 isXXX 인 경우 프로퍼티 게터에 get이 붙지 않고 이름 그대로 사용
>

### 커스텀 접근자
프로퍼티 접근자를 직접 작성 가능

```Kotlin
class Rectangle(var height: Int, val width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }

        //이것도 가능
        get() = height == width 
}
```
예시의 isSquare에는 자체 값을 저장하는 필드 필요 없음

### 코틀린 소스코드 구조: 디렉터리와 패키지
- 자바는 모든 클래스를 패키지 단위로 관리
- 같은 패키지 내에 있다면 직접 사용 가능
- 다른 패키지에 잇다면 import 
- 코틀린) 여러 클래스를 한 파일에 넣을 수 있고 파일 이름 제약 
    > 그러나 자바와 같이 패키지별로 디렉터리 구성하는 것이 나음 

### enum 클래스
- 소프트 키워드: 다른 키워드와 합쳐져서는 특별한 의미가 있지만 단독으로 있을 때 이름으로 사용 가능  
  - e.g. enum
- 키워드: 특별한 의미를 가지고 있기 때문에 이름으로 사용 X
    - e.g. class
- 값 열거, 프로퍼티나 메서드 O
- 생성자와 프로퍼티 선언 
- enum 클래스 안 메서드를 정의하는 경우 enum 상수 목록과 메서드 사이 세미콜론 필요

- **when**
  - 자바에서 switch에 해당
  ```Kotlin
  fun getMnemonic(color: Color) = 
    when (color) {
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        ...
    }
  ```
  - when 분기 조건에 임의 객체 허용
    - set을 사용한다면 객체 사이 매치할 때 동등성 비교
  - 인자 없어도 됨 -> 조건이 Boolean 결과를 계산해야 함

- 스마트 캐스트
  - `is` 변수 타입 검사
  - 코틀린에서는 컴파일러가 캐스팅 해줌
  - `as` 원하는 타입으로 명시적 타입 캐스팅할 때 사용
  
### 이터레이션
- while
  - while, do-while 루프
- for
  - 코틀린에는 자바의 for 루프가 없음
  - 범위(range) 사용 -> `for (i in 1..10)`
    - 폐구간 e.g. 1..10 
    - downTo 역방향 수열
    - until: 끝이 포함되지 않는 범위
- 맵 이터레이션
  - `for((key, value) in map)` 
- `in`
  - 어떤 값이 범위에 속하는지 검사
  - `!in` 도 가능
  - 문자, 숫자, 비교 가능 클래스(Comparable 구현) 모두 범위 가능

### 예외 처리
- try, catch, finally
  - 자바에서 함수 작성 때 throws XXException 을 붙여야 함
  - checked exception이기 때문 -> 명시적 처리
  - 코틀린에서는 checked exception, unchecked exception 구분 X
  - 자바에서는 체크 예외 처리를 강제하는데 이때 의미 없이 예외를 던지거나 예외를 처리하지 않고 무시하는 경우가 많아서 오류 발생 방지를 못함
- try 식으로 사용
  - try 키워드는 `식`임 
  - 따라서 내부에 여러 문장이 있다면 마지막 식의 값이 결과 값

