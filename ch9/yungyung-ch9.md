# 9장 제네릭스

## 9.1 제네릭 타입 파라미터

제네릭스: 타입 파라미터를 받는 타입을 정의 가능

코틀린 컴파일러: 타입 인자도 추론 가능

- 빈 리스트를 만드는 경우 타입인자를 명시

```kotlin
val authors = listOf("Ditry", "Svetlana")
val readers: MutableList<String> = mutableListOf()
val readers = mutableListOf<String>()
```

### 제네릭 함수와 프로퍼티

제네릭 함수 호출: 반드시 구체적 타입으로 타입 인자를 넘겨야 한다

- 대부분 컴파일러가 타입 인자 추론

클래스/인터페이스 안에 정의된 메서드, 확장 함수, 최상위 함수에서 타입 파라미터 선언 가능

- 확장 함수: 수신 객체/파라미터 타입에 대한 타입 파라미터 사용 가능

제네릭 확장 프로퍼티

```kotlin
val <T> List<T>.penultimate: T // 모든 리스트 타입에 제네릭 확장 프로퍼티 사용 가능
get() = this[size - 2]
>>> println(listOf(1, 2, 3, 4).penultimate)
```

- 일반 프로퍼티는 타입 파라미터를 가질 수 없다 → 클래스 프로퍼티에 여러 타입의 값을 저장할 수는 없음

### 제네릭 클래스 선언

```kotlin
interface List<T> { // List 인터페이스에 T라는 타입 파라미터 정의
	operator fun get(index: Int): T // 인터페이스 안에서 T를 일반 타입처럼 사용
	// ...
}
```

제네릭 클래스를 확장하는 클래스 정의

- 기반 타입의 제네릭 파라미터에 대해 타입 인자 지정 필요
    - 구체적 타입
    - 타입 파라미터로 받은 타입 (하위 클래스도 제네릭 클래스)

```kotlin
class StringList: List<String> { // 타입 인자: String - 구체적
	override fun get(index: Int): String = ... }
class ArrayList<T> : List<T> { // 타입 인자: 제네릭 타입, 앞의 List의 T와 다른 타입 파라미터
	override fun get(index: Int): T = ...
}
```

클래스가 자기 자신을 타입 인자로 참조 가능 ex) `Comparable`

### 타입 파라미터 제약

클래스나 함수에 사용할 수 있는 타입 인자 제한

- 어떤 타입을 제네릭 타입의 타입 파라미터에 대한 사항으로 지정

```kotlin
fun <T : Number> List<T>.sum(): T // Number는 상항
```

타입 파라미터에 대해 둘 이상의 제약을 가하는 경우

```kotlin
fun <T> ensureTrailingPeriod(seq: T)
				where T : CharSequence, T : Appendable {
		if (!seq.endsWith('.')) {
				seq.append('.')
		}
}
```

### 타입 파라미터를 널이 될 수 없는 타입으로 한정

```kotlin
class Processor<T : Any> {
		fun process(value: T) {
			value.hashCode()
		}
}
```

`Any`를 상한으로 사용

## 9.2 실행 시 제네릭스의 동작: 타입 파라미터와 실체화된 타입 파라미터

JVM의 제네릭스: 타입 소거를 사용

- 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지 않음

코틀린: `inline` 함수 사용시 타입 인자가 지워지지 않게 할 수 있다

### 실행 시점의 제네릭: 타입 검사와 캐스트

코틀린 제네릭 타입 인자 정보는 런타임에 지워진다

ex) List의 경우 List 객체가 어떤 타입의 원소를 저장하는지 실행 시점에 알 수 없다

- 컴파일러가 타입 인자를 알고 올바른 타입의 값만 넣도록 보장한다

타입 소거의 한계

- 실행 시점에 타입 인자를 검사할 수 없다
    - ex) `is` 검사로 타입 인자로 지정한 타입을 확인할 수 없다

장점

- 저장해야 하는 타입 정보의 크기가 감소 → 메모리 사용량 감소
    - 코틀린은 제네릭 타입 사용 시 타입 인자 명시가 필요
        - 어떤 값이 리스트라는 사실을 확인하는 방법 = 스타 프로젝션 사용

```kotlin
if (value is List<*>) { ... }
```

`as` 캐스팅에도 제네릭 타입 사용 가능

- 타입 인자가 다른 타입으로 캐스팅해도 캐스팅에 성공한다
- 컴파일러: unchecked cast 경고

타입 인자가 다른 경우에는 `IllegalArgumentException`이 아닌 `ClassCastException` 발생 (캐스트는 성공, 실행하는 시점에서 예외 발생

코틀린 컴파일러: 컴파일 시점에 타입 정보가 주어진 경우 → is 검사 수행 가능

```kotlin
fun printSum(c: Collection<Int>) {
	if (c is List<Int>) {
		println(c.sum())
	}
}
```

### 실체화한 타입 파라미터를 사용한 함수 선언

인라인 함수의 타입 파라미터는 실체화된다 → 실행 시점에 타입 인자를 알 수 있다

```kotlin
inline fun <reified T> isA(value: Any) = value is T
>>> println(isA<String>("abc"))
true
>>> println(isA<String>(123))
false
```

```kotlin
inline fun <reified T> // reified: 타입 파라미터가 실행 시점에 지워지지 않음을 표시
				Iterable<*>.filterIsInstance(): List<T> {
	val destination = mutableListOf<T>()
	for (element in this) {
		if (element is T) { // 타입 인자 검사 가능
			destination.add(element)
		}
	}
	return destination
}
```

자바 코드에선 `reified` 타입 파라미터를 사용하는 `inline`함수 호출 불가

- inline 함수의 이점: 실체화한 타입 파라미터 사용 가능

### 실체화한 타입 파라미터로 클래스 참조 대신

`java.lang.Class` 타입 인자를 파라미터로 받는 API에 대한 코틀린 어댑터 구축 시 실체화한 타입 파라미터 사용

```kotlin
val serviceImpl = ServiceLoader.load(Service::class.java)
```

`::class.java` 코틀린 클래스에 대응하는 `java.lang.Class` 참조를 얻는 방법

```kotlin
val serviceImpl = loadService<Service>()

inline fun <reified T> loadService() { // 타입 파라미터에 reified 사용
	return ServiceLoader.load(T::class.java) // T::class로 타입 파라미터의 클래스 가져오기
}
```

### 실체화한 타입 파라미터의 제약

사용 가능한 경우

- 타입 검사와 캐스팅
- 코틀린 리플렉션 API (`::class`)
- 코틀린 타입에 대응하는 `java.lang.Class` 얻기
- 다른 함수를 호출할 때 타입 인자로 사용

사용 불가능한 경우

- 타입 파라미터 클래스의 인스턴스 생성
- 타입 파라미터 클래스의 동반 객체 메서드 호출
- 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 인자로 넘기기
- 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 `reified`로 지정
    - 실체화한 타입 파라미터를 사용하는 함수는 자신에게 전달되는 모든 람다와 함께 인라이닝
        - 인라이닝 금지하려면 `noinline` 변경자 붙이기

## 9.3 변성: 제네릭과 하위 타입

변성: 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지

### 변성이 있는 이유: 인자를 함수에 넘기기

`List<Any>` 타입의 파라미터를 받는 함수에 `List<String>` 을 넘기면

- List 인터페이스의 타입 인자로 들어가는 경우 안전성 보장x

```kotlin
fun addAnswer(list: MutableList<Any>) {
	list.add(42)
}

>>> val strings = mutableListOf("abc", "bac") 
>>> addAnswer(strings) // 컴파일된다면
>>> println(strings.maxBy { it.length }) // 실행 시점에 예외 발생
ClassCastException: Integer cannot be cast to String
```

코틀린 컴파일러는 위와 같은 함수 호출을 금지

- 원소의 추가나 변경이 없는 경우 `List<Any>` 타입의 파라미터를 받는 함수에 `List<String>` 을 넘기면 안전하다

### 클래스, 타입, 하위 타입

모든 코틀린 클래스가 적어도 둘 이상의 타입을 구성할 수 있다 → 널이 될 수 있는 타입

제네릭 클래스: 제네릭 타입의 타입 파라미터를 구체적인 타입 인자로 바꿔줘야 한다

ex) `List`: 클래스 O, 타입X, `List<Int>`: 제대로 된 타입O → **각각의 제네릭 클래스는 많은 타입을 만들어낸다**

하위 타입: 타입 A의 값이 필요한 모든 장소에 타입 B의 값을 넣어도 문제가 없다면 B는 A의 하위 타입이다

- 모든 타입이 자신의 하위 타입이다

```kotlin
fun test(i: Int) {
	val n: Numbe = i // Int는 Number의 하위 타입
	fun f(s: String) { }
	f(i) // Int는 String의 하위 타입X
}
```

어떤 값의 타입이 변수의 하위 타입인 경우에만 값을 변수에 대입하게 허용

간단한 경우 하위 타입 = 하위 클래스

- 널이 될 수 있는 타입: 하위 타입 ≠ 하위 클래스
- 널이 될 수 없는 타입은 널이 될 수 있는 타입의 하위 타입이다 (반대는 성립x)

무공변: 제네릭 타입을 인스턴스화할 때 서로 다른 타입이 들어가면 인스턴스 타입 사이의 하위 타입 관계가 성립되지 않는 경우의 제네릭 타입

자바에서는 모든 클래스가 무공변이다

공변적이다: A가 B의 하위 타입이면, `List<A>`는 `List<B>`의 하위 타입이다

### 공변성: 하위 타입 관계를 유지

`Producer<T>`에서 `Producer<A>`가 `Producer<B>`의 하위 타입이면 `Producer`는 공변적이다

→ 하위 타입 관계가 유지된다

```kotlin
interface Producer<**out** T> { // out: 공변적이다
		fun produce(): T
}
```

클래스의 타입 파라미터를 공변적으로 만들면 함수 정의에 사용한 파라미터 타입과 타입 인자의 타입이 정확히 일치하지 않아도 그 클래스의 인스턴스를 함수 인자/반환값으로 사용 가능

타입 파라미터를 공변적으로 지정 → 클래스 내부에서 그 파라미터를 사용하는 방법을 제한한다

타입 파라미터를 사용할 수 있는 지점

- in: 파라미터 타입, 함수가 T 타입의 값을 소비
- out: 반환 타입, 함수가 T 타입의 값을 생산

`out` 키워드: 아웃 위치에서만 T를 사용하게 허용, 인 위치에서는 사용하지 못하게 막는다

- 공변성: 하위 타입 관계가 유지
- 사용 제한: T를 아웃 위치에서만 사용할 수 있다

→ `List<T>` 인터페이스: 리스트에 T 타입의 값을 추가/변경하는 메서드는 없다 → `List`는 공변적

cf) `MutableList<T>`는 공변적인 클래스로 선언할 수 없다 (변경하는 메서드 존재)

생성자 파라미터는 인/아웃 어느 쪽도 아니다 → `out` 키워드가 있는 파라미터도 생성자에 사용 가능

변성: 코드에서 위험할 여지가 있는 메서드를 호출할 수 없게 만ㅁ들어 제네릭 타입의 인스턴스 역할을 하는 클래스 인스턴스를 잘못 사용하는 일이 없게 방지한다

- 생성자는 나중에 호출할 수 있는 메서드가 아니어서 위험할 여지가 없다
- `val`, `var` 키워드가 생성자 파라미터에 있다면 게터/세터 정의와 같다
    - `val`: 아웃 위치
    - `var`: 인/아웃 위치 모두에 해당

비공개 메서드의 파라미터는 인도 아웃도 아니다

### 반공변성: 뒤집힌 하위 타입 관계

반공변성: 공변성을 거울에 비친 상

- 타입 B가 타입 A의 하위 타입인 경우 `Consumer<A>`가 `Consumer<B>`의 하위 타입인 관계

`in` 키워드: T 타입의 값을 소비하기만 하는 경우, 인 위치에 쓰인다

- Comparator 구현에서 string을 비교하는 경우, Comparator<Any> 타입을 사용할 수 있다
    - 그 타입의 하위 타입에 속하는 모든 값을 비교할 수 있기 때문

클래스 정의에 변성을 직접 기술하면 그 클래스를 사용하는 모든 장소에 변성이 적용된다

### 사용 지점 변성: 타입이 언급되는 지점에서 변성 지정

선언 지점 변성: 클래스를 선언하면서 변성을 지정

- 사용하는 지점에서 변성에 대해 정의 필요x

사용 지점 변성: 타입 파라미터가 있는 타입을 사용할 때마다 어떤 타입으로 대치할 수 있는지 명시

- 자바 `? extends`, `? super`

```kotlin
fun <T: R, R> copyData(source: MutableList<T>,
											destination: MutableList<R>) {
		for (item in source) {
		destination.add(item)
		}
}
>>> val ints = mutableListOf(1, 2, 3)
>>> val anyItems = mutableListOf<Any>()
>>> copyData(ints, anyItems)
>>> println(anyItems)
[1, 2, 3]
```

source의 T는 destination의 R의 하위 타입이어야 한다

```kotlin
fun <T> copyData(source: MutableList<out T>,
								destination: MutableList<T>) {
		for (item in source) {
		destination.add(item)
		}
}
```

함수 구현이 아웃 위치에 있는 타입 파라미터를 사용하는 메서드만 호출한다면, 함수 정의 시 변성 변경자를 추가할 수 있다

→ 타입 선언에서 타입 파라미터를 사용하는 위치라면 어디에나 변성 변경자를 붙일 수 있다

- 파라미터 타입
- 로컬 변수 타입
- 함수 반환 타입 등

→ 타입 프로젝션: source는 일반적인 `MutableList`가 아니라, `MutableList`를 프로젝션한 타입

```kotlin
fun <T> copyData(source: MutableList<T>,
								destination: MutableList<in T>) {
		for (item in source) {
			destination.add(item)
		}
}
```

in 프로젝션을 사용하여 소비자 역할을 수행한다고 표시

- 코틀린의 사용 지점 변성 선언 = 자바의 한정 와일드카드

### 스타 프로젝션: 타입 인자 대신 * 사용

- `MutableList<*>`는 `MutableList<Any?>`와 같지 않다
    - `MutableList<Any?>`: 모든 타입의 원소를 담을 수 있다
    - `MutableList<Any?>`: 어떤 정해진 구체적 타입의 원소만을 담을 수 있는 리스트 + 그 원소의 타입을 정확히 모른다
- 컴파일러는 `MutableList<*>`를 아웃 프로젝션 타입으로 인식
    - 타입을 모르는 원소를 꺼내올 수는 있지만, 변경/추가할 수는 없다
    - 자바의 `MyType<?>`에 대응

https://kotlinlang.org/docs/generics.html#star-projections

- 타입 파라미터를 시그니처에서 전혀 언급하지 않거나/데이터를 읽지만 타입에 관심이 없는 경우 사용
- 스타 프로젝션 우회: 제네릭 타입 파라미터 도입
- 값을 만들어내는 메서드만 호출, 값의 타입에는 신경쓰지 않는다