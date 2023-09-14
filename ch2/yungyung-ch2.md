# 2장 코틀린 기초

## 2.1 기본 요소: 함수와 변수

### Hello, World!

`System.out.println` 대신 `println` : 표준 자바 라이브러리 함수를 간결하게 사용할 수 있게 감싼 래퍼(wrapper) 제공

```kotlin
@kotlin.internal.InlineOnly
public actual inline fun println() {
    System.out.println()
}
```

### 함수

타입 추론: 컴파일러가 타입을 분석해 프로그램 구성 요소의 타입을 정해줌

식이 본문인 함수의 반환 타입만 생략 가능

### 변수

`val` (value): 변경 불가능한 참조 (자바 final 변수)

`var` (variable): 변경 가능한 참조 (자바 일반 변수)

- 타입 변경은 불가
- 컴파일러가 변수 서언 시점의 초기화식으로 타입 추론

## 2.2 클래스와 프로퍼티

코틀린 기본 가시성 = public

### 프로퍼티

자바: 데이터를 필드에 저장, private 설정, 접근자 메서드 제공 → 필드 + 접근자 = 프로퍼티

코틀린: 프로퍼티 - 필드+접근자 메서드 대신

- val: 읽기 전용 - (비공개) 필드 + (공개) getter
- var: 변경 가능 - (비공개) 필드 + (공개) getter + (공개) setter

프로퍼티 이름으로 getter/setter 자동 호출

### 커스텀 접근자

```kotlin
class Rectangle (val height: Int, val width: Int) {
		val isSquare: Boolean
			get() {
				return hight == width
			}
}
```

### 코틀린 소스코드 구조: 디렉터리와 패키지

클래스 임포트와 함수 임포트에 차이가 없음, 모든 선언을 import 키워드로 가져올 수 있다

- 최상위 함수: 함수 이름으로 임포트 가능

코틀린: 디렉터리 구조 ≠ 패키지 구조

- 디렉터리 없이도 패키지 구성 가능

## 2.3 선택 표현과 처리: enum과 when

### enum

enum: 소프트 키워드

### when

자바의 switch 문과 달리 각 분기의 끝에 break 필요x

```kotlin
when (color) {
		Color.RED -> "Richard"
		Color.ORANGE -> "Of"
		...
}
```

한 분기 안에 여러 값을 매치 패턴으로 사용

```kotlin
when (color) {
		Color.RED, Color.ORANGE -> "warm"
		...
}
```

자바의 switch 문: 분기 조건에 상수만 사용

when의 분기 조건은 임의의 객체 허용

```kotlin
fun mix (c1: Color, c2: Color) = 
		when (setOf(c1,c2)) {
				setOf(RED, YELLOW) -> ORANGE
				setOf(YELLOW, BLUE) -> GREEN
				setOf(BLUE, VIOLET) -> INDIGO
				else -> throw Exception("Dirty color")
		}
```

### 인자 없는 when 사용

```kotlin
fun mix (c1: Color, c2: Color) = 
		when {
				(c1 == RED && c2 == YELLOW) || (c1 == YELLOW && c2 == RED) -> ORANGE
				...
		}
```

인자 없는 when = 각 분기의 조건이 Boolean 결과 계산

### 스마트 캐스트: 타입 검사와 타입 캐스트를 조합

스마트 캐스트: 어떤 변수가 원하는 타입인지 is로 검사 → 컴파일러가 캐스팅을 수행

- is로 검사한 후 그 값이 바뀔 수 없는 경우에만(val / 커스텀 접근자x) 작동

## 2.4 대상을 이터레이션: while과 for 루프

수열: 어떤 범위에 속한 값을 일정한 순서로 이터레이션

`..` 연산자: 문자 타입의 값에도 적용 가능

```kotlin
for (c in 'A'..'F') {...}
```

### 맵 이터레이션

```kotlin
for ((letter, binary) in binaryReps) {
		println("$letter = $binary")
}
```

`letter` = key, `binary` = value

컬렉션

```kotlin
for ((index, element) in list.withIndex()) {
}
```

### in으로 컬렉션이나 범위의 원소 검사

```kotlin
c in 'a'..'z'

'a' <= c && c <= 'z'

when (c) {
	in '0'..'9' -> "It's a digit"
}
```

`in` 연산자: 컬렉션/범위에 값이 속하는지 확인 - 비교 가능한 클래스라면 범위 생성 가능

## 2.5 코틀린의 예외 처리

체크 예외와 언체크 예외 구별x (자바: 체크 예외 처리 강제)

### try를 식으로 사용

```kotlin
val number = try {
		Integer.parseInt(reader.readLine())
} catch (e: NumberformatException) {
		null		
}
```

try를 식으로 사용하여 변수에 대입 가능 → try의 본문은 반드시 중괄호로 둘러싸야 한다