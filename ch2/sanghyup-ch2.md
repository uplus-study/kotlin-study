# 코틀린 기초

# 1. 함수와 변수

## 1.1 함수

함수를 최상위 수준에 정의할 수 있다. 꼭 클래스안에 넣어야 할 필요가 없다.

> 문(statement)과 식(expression)의 구분
코틀린에서 if는 식이지 문이 아니다. 식은 값을 만들어내며 다른 식의 하위 요소로 계산에 참여할 수 있는 반면 문은 자신을 둘러싸고 있는 가장 안쪽 블록 최상위 요소로 존재하며 아무런 값을 만들어내지 않는다는 차이가 있다.
자바에서는 대부분의 제어 구조가 문인 반면 코틀린에서는 루프를 제외한 대부분의 제어 구조가 식이다.
> 

```kotlin
fun max(a: Int, b: Int): Int {
	return if (a > b) a else b
}
// 함수 = 식 형태로 표현 가능
fun max(a: Int, b: Int): Int = if (a > b) a else b

// 반환 값 생략 가능
// 코틀린 컴파일러가 컴파일 시점에 식의 결과 타입을 반환 타입으로 정해준다.
// 타입 추론
fun max(a: Int, b: Int) = if (a > b) a else b
```

그러나  실전 프로그램에는 아주 긴 함수에 return 문이 여럿 들어있는 경우가 자주 있다. 그런 경우 반환 타입을 꼭 명시하고 return을 반드시 사용한다면 함수가 어떤 타입의 값을 반환하고 어디서 그런 값을 반환하는지 더 쉽게 알아볼 수 있다.

## 1.2 변수

코틀린에서는 타입 지정을 생략해서 변수를 선언한다.

타입을 지정하지 않으면 컴파일러가 초기화 식을 분석해 초기화 식의 타입을 변수 타입으로 지정한다.

초기화 식을 사용하지 않고 변수를 선언하려면 변수 타입을 반드시 명시해야한다.

### 변경 가능한 변수와 변역 불가능한 변수

변수 선언시

**val**(value) → 변경 불가능한 참조를 저장하는 변수다. 자바에서 final

**var**(variable) → 변경 가능한 참조다.

**기본적으로 모든 변수를 val 키워드를 이용해 불변 변수로 선언하고 나중에 꼭 필요할 때에만 var로 변경하라**

val 변수는 정확히 한번 초기화 돼야 하지만 어떤 블록이 실행 될 때 오직 한 초기화 문장만 실행됨을 컴파일러가 확인할 수 있다면 조건에 따라 val을 여러 값으로도 초기화 가능하다.

```kotlin
ex)
val message: String
if(canPerformOperation()) {
	message = "Sucess"
 //.. 연산을 수행한다.
}else {
	message = "Failed"
}
```

var 키워드를 사용하면 변수의 값을 변경할 수 있지만 변수의 타입은 고정돼 바뀌지 않는다.

## 1.3 문자열 템플릿

```kotlin
val name = "Kotlin"
println("Hello, $name")
```

여러 스크립트 언어와 비슷하게 문자열 리터럴 필요한 곳에 변수를 넣되 변수 앞에 $를 추가한다.

복잡한 식도 `{}` 를 사용해서 문자열 템플릿 안에 넣을 수 있다.

# 2. 클래스와 프로퍼티

```kotlin
//Java
public class Person {
	private final String name;
	
	public Person(String name){
		this.name = name;
	}
	
	public String getName() {
		return name;
	}
}

// Kotlin
class Person(val name: String) 
```

## 2.1 프로퍼티

자바에서는 필드와 접근자를 하나로 묶어 **프로퍼티** 라고 부른다.

코틀린 프로퍼티는 자바의 필드와 접근자 메소드를 완전히 대신한다.

클래스에서 프로퍼티를 선언할 때는 앞에서 살펴본 변수를 선언하는 방법과 마찬가지로 val이나 var를 사용한다.

val로 선언한 프로퍼티는 읽기 전용이며, var로 선언한 프로퍼티는 변경 가능하다.

> 코틀린에서 is로 시작하는 프로퍼티의 getter에는 get이 붙지 않고 is~()로 사용한다.
> 

## 2.2 커스텀 접근자

```kotlin
class Rectangle(val height: Int, val width: Int) {
	val isSquare: Boolan
		get() {
			return height == width
		}
}
```

파라미터가 없는 함수를 정의하는 방식과 커스텀 게터를 정의하는 방식 중 구현이나 성능상 차이는 없다.

## 2.3 코틀린 소스코드 구조: 디렉터리와 패키지

코틀린에서는 여러 클래스를 한 파일에 넣을 수 있고, 파일의 이름도 마음대로 정할 수 있다.

코틀린은 자바와 달리 패키지가 디렉토리 구조를 따를 필요는 없다.

그러나 대부분의 경우 자바와 같이 패키지 별로 디렉터리를 구성하는 편이 낫다.

그렇지만 여러 클래스를 한 파일에 넣는 것을 주저해서는 안된다.

# 3. 선택 표현과 처리: enum 과 when

## 3.1 enum 클래스 정의

코틀린에서 enum은 소프트 키워드라고 부르는 존재다. enum은 class 앞에 있을 때는 특별한 의미를 지니지만 다른 곳에서는 이름에 사용할 수 있다. 그러나 class는 키워드라 다른데서 사용할 수 없다.

enum 클래스 안에서도 프로퍼티나 메소드르 정의할 수 있다.

enum 클래스 안에 메소드를 정의하는 경우 반드시 enum 상수 목록과 메소드 정의 사이에 세미콜론을 넣어야 한다.

## 3.2 when으로 enum 클래스 다루기

when도 if와 마찬가지로 값을 만들어내는 식이다.

when은 자바 switch와 달리 break를 넣지 않아도 된다.

Ex)

```kotlin
fun getWarmth(color: Color) = when(color) {
	Color.RED, Color.Orange, Color.Yellow -> "warm"
  Color.GREEN -> "neutral"
	Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold"
}
```

## 3.3 when과 임의의 객체를 함께 사용

코틀린의 when의 분기조건은 임의의 객체를 허용한다.

```kotlin
fun mix(c1: Color, c2:Color) = 
	when(setOf(c1, c2)) {
		setOf(RED, YELLOW) -> ORANGE,
		setOf(BLUE, YELLOW) -> GREEN,
		setOf(BLUE, VIOLET) -> INDIGO,
		else -> throw Exception("Dirty Color")
}
```

## 3.4 인자 없는 when 사용

when에 아무 인자도 없으면 각 분기의 조건이 Boolean 결과를 계산하는 식이어야 한다.

## 3.5 스마트 캐스트: 타입 검사와 타입 캐스트를 조합

코틀린에서 is는 자바의 instanceOf와 비슷하다.

그러나 is는 어떤 타입인지 먼저 검사하고 나면 굳이 변수를 원하는 타입으로 캐스팅하지 않아도 된다.

실제로는 컴파일러가 캐스팅을 수행해준다.

이를 **스마트 캐스트**라고 부른다

스마트 캐스트는 is로 변수에 든 값의 타입을 검사한 다음에 그 값이 바뀔 수 없는 경우에만 작동한다.

클래스의 프로퍼티에 대해 스마트 캐스트를 사용한다면 프로퍼티는 반드시 val이어야 하며 val이 아니거나 val이지만 커스텀 접근자를 사용하는 경우에는 해당 프로퍼티에 대한 접근이 항상 같은 값을 내놓는다고 확신할 수 없기 때문이다.

## 3.6 if를 when으로 리팩토링

Kotlin에서 If연쇄를 사용해서 식을 계산하기

```kotlin
fun eval(e: Expr): Int {
	if (e is Num) {
		val n = e as Num
		return n.value
	}
	if (e is Sum) {
		return eval(e.right) + eval(e.left))
	}
	throw IllegalArgumentException("Unknown expression")
}
```

값을 만들어내는 if 식

```kotlin
fun eval(e: Expr) = 
	if (e is Num) {
		e.value
	}
  else if (e is Sum) {
		return eval(e.right) + eval(e.left))
	}
	else {
		throw IllegalArgumentException("Unknown expression")
	}
}
```

when으로 다듬기

```kotlin
fun eval(e: Expr) = 
	when (e) {
	 is Num	-> e.value
   is Sum -> eval(e.right) + eval(e.left))
	 else -> throw IllegalArgumentException("Unknown expression")
}
```

# 4. 대상을 이터레이션: while과 for 루프

## 4.1 while 루프

코틀린에는 while과 do-while 루프가 있다. 

```kotlin
while(조건) {
	/*...*/
}

do {
	/*...*/
} while(조건) 
```

## 4.2 수에 대한 이터레이션: 범위와 수열

코틀린에는 자바의 for 루프에 해당하는 요소가 없다. 대신에 코틀린은 범위(Range)를 사용한다

`val oneToTen = 1..10` (양끝을 포함하는 폐구간이다.)

`..` 는 문자타입에도 적용할 수 있다. ex) `‘A’.. ‘F’`

```kotlin
for(i in 1..100) {
	/** 1부터 100까지 1씩 증가하는 수열 */
}

for(i in 100 downTo 1 step 2) {
	/** 100부터 1까지 2씩 감소하는 수열 */
}

```

반만 닫힌 범위를 사용하고 싶을때는 `until` 을 사용한다.

## 4.3 맵에 대한 이터레이션

```kotlin
val binaryReps = TreeMap<Char, String>()

for((letter, binary) in binaryReps)) {
		/** 맵에 대한 iteration letter에는 key가 들거가고 binary에는 value가 들어감 */
}

//맵에 사용한 구조 분해를 컬렉션에서도 사용할 수 있다.
val list = listOf("10","11", "1001")
for((index,element) in list.withIndex()) {
	println("$index: $element")
}

0: 10
1: 11
2: 1001
```

## 4.4 in으로 컬렉션이나 범위의 원소 검사

`in` ****연산자를 사용해 어떤 값이 범위에 속하는지 검사할 수 있다. 반대로 `!in`을 사용하면 어떤 값이 범위에 속하지 않는지 알 수 있다.

```kotlin
c in 'a' .. 'z' => 'a' <= c && c <= 'z'
```

# 5. 코틀린의 예외처리

코틀린의 예외처리는 자바와 거의 동일하다.

## 5.1 try, catch, finally

자바와 다른점은 `throws` 절이 코드에 없다는 점이다.

자바와 달리 코틀린은 checkedException과 uncheckedException을 구분하지 않는다.

> 자바는 checked exception을 강제한다. 하짐나 프로그래머들이 의미없이 예외를 다시 던지거나, 예외를 잡되 처리하지 않고 무시하는 코드를 작성하는 경우가 흔하다.  그래서 코틀린 예외를 이렇게 설계했다.
> 

## 5.2 try를 식으로 활용

코틀린의 try 키워드는 if나 when과 마찬가지로 식이다. 따라서 try 값을 변수에 대입할 수 있다.