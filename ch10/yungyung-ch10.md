# 10장 애노테이션과 리플렉션

## 10.1 애노테이션 선언과 적용

### 애노테이션 적용

```kotlin
@Deprecated("Use removeAt(index) instead.", ReplaceWith("removeAt(index)"))
fun remove(index: Int) {
    ...
}
```

`replaceWith` 파라미터: 옛 버전을 대신할 수 있는 패턴 제시

- remove를 호출하는 코드에 대해 경고 메시지 표시 + 퀵 픽스 제시

애노테이션 인자를 지정하는 문법

- 클래스를 애노테이션 인자로 지정: `@MyAnnotation(MyClass::class)`
- 다른 애노테이션을 인자로 지정: 애노테이션의 이름 앞에 @ 넣지 않기 `@MyAnnotation(MyOtherAnnotation::class)`
- 배열을 인자로 지정: `@RequestMapping(path = arrayOf("/foo", "/bar"))`

애노테이션 인자: 컴파일 시점에 알 수 있어야 한다

-> 임의의 프로퍼티를 인자로 지정 불가, const 변경자 필요

```kotlin
const val TEST_TIMEOUT = 100L

@Test(timeout = TEST_TIMEOUT)
fun testMethod() {
    ...
}
```

### 애노테이션 대상

코틀린 선언을 컴파일한 결과가 여러 자바 선언과 대응하는 경우 + 여러 자바 선언에 각각 애노테이션을 붙여야 하는 경우

- 어떤 요소에 애노테이션을 붙일지 표시

**사용 지점 대상** 선언

```kotlin
    @get:Rule // 프로퍼티 게터에 애노테이션을 붙임
```

자바에 선언된 애노테이션을 사용해 프로퍼티에 붙이는 경우 기본적으로 프로퍼티의 필드에 애노테이션이 붙는다

코틀린 애노테이션 선언: 프로퍼티에 직접 적용할 수 있는 애노테이션 생성 가능

사용 지점 대상 목록

- `property`: 프로퍼티, 자바에서 선언된 애노테이션에는 사용 불가
- `field`: 프로퍼티의 필드 (뒷받침하는 필드)
- `get`: 프로퍼티 게터
- `set`: 프로퍼티 세터
- `receiver`: 확장 함수나 프로퍼티의 수신 객체 파라미터
- `param`: 생성자 파라미터
- `setparam`: 세터 파라미터
- `delegate`: 위임 프로퍼티의 위임 인스턴스를 담아둔 필드
- `file`: 파일 안에 선언된 최상위 함수와 프로퍼티를 담아둔 클래스

    - `@JvmName`: 파일에 적용하는 애노테이션의 예

자바와 달리 애노테이션 인자로 임의의 식을 허용

```kotlin
fun test(list: List<*>) {
    @Suppress("UNCHECKED_CAST")
    val strings = list as List<String>
    // ...
}
```

**자바 API를 애노테이션으로 제어하기**

- @JvmName: 코틀린 선언이 만들어내는 자바 필드나 메서드 이름 변경
- @JvmStatic: 코틀린 함수를 자바 정적 메서드로 바꿈
- @JvmOverloads: 코틀린 함수(디폴트 파라미터o)에 대응하는 모든 자바 오버로딩 메서드 생성
- @JvmField: 코틀린 프로퍼티를 자바 필드로 바꿈

### 애노테이션을 활용한 JSON 직렬화 제어

직렬화: 객체를 저장하거나 네트워크를 통해 전송하기 위해 텍스트/이진 형식으로 변환
역직렬화: 텍스트/이진 형식으로 저장된 데이터로부터 원래의 객체를 만들어냄

JSON에는 객체의 타입이 저장x -> 타입 인자로 클래스 명시

제이키드 라이브러리: 모든 프로퍼티를 직렬화,프로퍼티 이름을 키로 사용

- `@JsonExclude`: 직렬화/역직렬화 시 프로퍼티 무시
- `@JsonName`: 프로퍼티 이름을 다른 이름으로 사용

### 애노테이션 선언

애노테이션 클래스: 선언, 식과 관련 있는 메타데이터의 구조를 정의

```kotlin
annotation class JsonName(val name: String) // 모든 파라미터 앞에 val
```

```java
public @interface JsonName {
    String value();
}
```

자바의 value 메서드: value를 제외한 모든 애트리뷰트에 이름을 명시해야 한다

코틀린의 애노테이션 적용: 일반적인 생성자 호출과 같다

자바에서 선언한 애노테이션을 코틀린에 적용 - value를 제외한 모든 인자에 대해 이름 붙인 인자 구문 사용

### 메타애노테이션: 애노테이션을 처리하는 방법 제어

메타애노테이션: 애노테이션 클래스에 적용할 수 있는 애노테이션

- `@Target`: 애노테이션을 적용할 요소의 종류를 제한

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
```

Target을 지정하지 않으면 모든 선언에 적용할 수 있음

메타 애노테이션 예시

```kotlin
@Target(AnnotationTarget.ANNOTATION_CLASS)
annotation class BindingAnnotation

@BindingAnnotation
annotation class MyBinding
```

대상을 `PROPERTY`로 지정한 애노테이션은 자바 코드에서 사용x

-> FIELD를 대상으로 추가해야 한다

> `@Retention 애노테이션`
> - 애노테이션을 언제까지 유지할 것인지 지정
> - `@Retention(AnnotationRetention.RUNTIME)` 애노테이션을 붙이면 애노테이션을 컴파일한 클래스 파일에도 애노테이션 정보가 남는다
> - `@Retention(AnnotationRetention.SOURCE)` 애노테이션을 붙이면 애노테이션을 컴파일하지 않는다
    > 자바 컴파일러: 기본적으로 .class 파일에 저장 / 런타임에는 사용 불가
> - 코틀린: 기본적으로 Retention을 RUNTIME으로 지정 -> 애노테이션을 런타임에 사용

### 애노테이션 파라미터로 클래스 사용

`@DeserializeInterface` 애노테이션: 인터페이스 타입인 프로퍼티에 대한 역직렬화를 제어

```kotlin
interface Company {
    val name: String
}

data class CompanyImpl(override val name: String) : Company

data class Person(
    val name: String,
    @DeserializeInterface(CompanyImpl::class) val company: Company
)
```

클래스 참조를 인자로 받는 애노테이션 정의

```kotlin
annotation class DeserializeInterface(val targetClass: KClass<out Any>)
```

`KClass`: 클래스 참조를 나타내는 클래스

### 애노테이션 파라미터로 제네릭 클래스 받기

```kotlin
interface ValueSerializer<T> {
    fun toJsonValue(value: T): Any?
    fun fromJsonValue(jsonValue: Any?): T
}

data class Person(
    val name: String,
    @CustomSerializer(DateSerializer::class) val birthDate: Date
)
```

CustomSerializer 구현

```kotlin
annotation class CustomSerializer(
    val serializerClass: KClass<out ValueSerializer<*>>
)
```

- out ValueSerializer<*>: ValueSerializer의 하위 타입을 받을 수 있다

> 애노테이션 인자로
> - 클래스: `KClass<out 허용할 클래스 이름>`
> - 제네릭 클래스: `KClass<out 허용할 클래스 이름<*>>`

## 10.2 리플렉션: 실행 시점에 코틀린 객체 내부 관찰

리플렉션: 실행 시점에 (동적으로) 객체의 프로퍼티와 메서드에 접근할 수 있게 해주는 방법

- 타입과 관계 없이 객체를 다뤄야 하거나, 객체가 제공하는 메서드나 프로퍼티 이름을 실행 시점에만 알 수 있는 경우
    - ex) 직렬화 라이브러리

코틀린 리플렉션

- 표준 리플렉션: 자바 패키지 `java.lang.reflect`
- 코틀린 리플렉션 API: `kotlin.reflect`

### 코틀린 리플렉션 API: KClass, KCallable, KFunction, KProperty

#### KClass

- 클래스 안에 있는 모든 선언 열거
- 각 선언에 접근
- 클래스의 상위 클래스를 얻음
- `MyClass::class`: KClass의 인스턴스를 얻음

```kotlin
class Person(val name: String, val age: Int)

val person = Person("Alice", 29)
val kClass = person.javaClass.kotlin // javaClass: 자바 클래스 참조를 얻음, kotlin: 코틀린 클래스 참조를 얻음 -> 코틀린 리플렉션 API로 옮겨온다
println(kClass.simpleName) // 클래스 이름
// Person

kClass.memberProperties.forEach { // 클래스의 모든 프로퍼티를 순회
    println("${it.name}") // 프로퍼티 이름과 값을 출력
}
// age
// name
```

```kotlin
interface KClass<T : Any> {
    val simpleName: String? // 클래스 이름
    val qualifiedName: String? // 패키지를 포함한 클래스 이름
    val members: Collection<KCallable<*>> // 클래스 안에 있는 모든 선언
    val nestedClasses: Collection<KClass<*>> // 중첩 클래스
    val constructors: Collection<KFunction<T>> // 생성자
}

```

#### KCallable

KCallable: 함수, 프로퍼티를 아우르는 공통 상위 인터페이스

- call 메서드 포함: 함수, 프로퍼티의 게터 호출

```kotlin
interface KCallable<out R> {
    fun call(vararg args: Any?): R
    ...
```

인자를 vararg 리스트로 전달

```kotlin
fun foo(x: Int) = println(x)
val kFunction = ::foo
kFunction.call(42)
// 42
```

call을 사용하여 함수를 호출할 수 있음

- call에 넘긴 인자 개수와 원래 함수의 파라미터 개수가 일치해야 한다
- 런타임 예외 발생(IlleagalArgumentException)

```kotlin
::foo
KFunction1<Int, Unit> // 파라미터와 반환값 타입 정보
```  

- KFunction1: 파라미터 개수가 1개인 함수를 나타냄
- invoke 메서드: 정해진 개수의 인자만 받아들임

```kotlin
fun sum(x: Int, y: Int) = x + y
val kFunction: KFunction2<Int, Int, Int> = ::sum
println(kFunction.invoke(1, 2) + kFunction(3, 4))
// 10
kFunction(1) // No value passed for parameter y
```

- invoke 메서드: 인자 개수/타입이 맞지 않으면 컴파일 오류

> KFunctionN 인터페이스
> KFunction을 확장
> N과 파라미터 개수가 같은 `invoke` 메서드 포함
> - 컴파일러가 생성한 합성 타입 (`kotlin.reflect` 패키지에 정의x)
> - KFunctionN 인터페이스를 직접 사용x

- call 메서드 호출: 프로퍼티의 게터 호출

프로퍼티 인터페이스

- KProperty0: 파라미터가 없는 프로퍼티 게터 (최상위 프로퍼티)

```kotlin
var counter = 0
val kProperty = ::counter
kProperty.setter.call(21) // 리플렉션 기능 - 세터 호출
println(kProperty.get()) // get호출: 21
```

- KProperty1: 파라미터가 1개인 프로퍼티 게터 (멤버 프로퍼티)
    - 제네릭 클래스
    - KProperty1<in T, out R>: T 타입의 수신 객체를 받아 R 타입의 값을 반환하는 프로퍼티

```kotlin
class Person(val name: String, val age: Int)

val person = Person("Alice", 29)
val memberProperty = Person::age
println(memberProperty.get(person)) // 29
```

함수의 로컬 변수에는 리플렉션으로 접근 불가

코틀린 리플렉션 API의 인터페이스 계층 구조

![image](https://github.com/uplus-study/kotlin-study/assets/109255398/01775fe4-55e9-4789-8073-6bff90808b91)

### 리플렉션을 사용한 객체 직렬화 구현

```kotlin
fun serialize(obj: Any): String
```

- 객체를 받아서 JSON 형식의 문자열로 변환
    - 객체의 프로퍼티, 값을 직렬화하며 `StringBuilder` 객체 뒤에 직렬화한 문자열 추가

```kotlin
private fun StringBuilder.serializeObject(obj: Any) {
    buildString { serializeObject(obj) }
}
```

객체 직렬화하기

```kotlin
private fun StringBuilder.serializeObject(obj: Any) {
    val kClass = obj.javaClass.kotlin // 객체의 KClass를 얻음
    val properties = kClass.memberProperties // 클래스의 모든 프로퍼티를 얻음
    properties.joinToStringBuilder(this, prefix = "{", postfix = "}") { prop ->
        serializeString(prop.name) // 프로퍼티 이름을 얻음
        append(": ")
        serializePropertyValue(it.get(obj)) // 프로퍼티 값을 얻음
    }
}
```

### 애노테이션을 활용한 직렬화 제어

`serializeObject` 함수가 애노테이션을 어떻게 처리하는가?

#### `@JsonExclude` 애노테이션

- KClass 인스턴스의 `memberProperties`를 사용해서 모든 멤버 프로퍼티를 가져옴
    - 위 애노테이션이 붙은 프로퍼티를 제외해야한다

`KAnnotatedElement` 인터페이스: `annotations` 프로퍼티 = 소스코드상에서 해당 요소에 적용된 모든 애노테이션 인스턴스의 컬렉션

- KProperty는 KAnnotatedElement의 하위 타입 -> `property.annotations`로 애노테이션 인스턴스 얻음

필요한 애노테이션 찾기

```kotlin
inline fun <reified T : Annotation> KAnnotatedElement.findAnnotation(): T? =
    annotations.filterIsInstance<T>().firstOrNull()

val properties = kClass.memberProperties.filter {
    it.findAnnotation<JsonExclude>() == null // 애노테이션을 검사
}
```

#### `@JsonName` 애노테이션

```kotlin
val jsonNameAnn = it.findAnnotation<JsonName>() // JsonName 애노테이션 찾기
val propName = jsonNameAnn?.name ?: it.name // 애노테이션의 인자를 사용하거나 프로퍼티 이름을 사용
```

#### `@CustomSerializer` 애노테이션

`getSerializer` 함수에 기초한다

```kotlin
data class Person(
    val name: String,
    @CustomSerializer(DateSerializer::class) val birthDate: Date
)

annotation class CustomSerializer(
    val serializerClass: KClass<out ValueSerializer<*>>
)

fun KProperty<*>.getSerializer(): ValueSerializer<Any?>? {
    val customSerializerAnn = findAnnotation<CustomSerializer>() ?: return null // 애노테이션을 찾음
    val serializerClass = customSerializerAnn.serializerClass
    val valueSerializer = serializerClass.objectInstance // object 선언으로 만든 싱글턴 객체를 얻음
        ?: serializerClass.createInstance() // 그렇지 않으면 클래스의 인스턴스를 만듦
    @Suppress("UNCHECKED_CAST")
    return valueSerializer as ValueSerializer<Any?>
}
```

이를 적용한 `serializePropertyValue` 함수

```kotlin
private fun StringBuilder.serializeProperty(
    prop: KProperty1<Any, *>,
    obj: Any
) {
    val name = prop.findAnnotation<JsonName>()?.name ?: prop.name
    serializeString(name)
    append(": ")
    val value = prop.get(obj)
    val jsonValue = prop.getSerializer()?.toJsonValue(value)  // 커스텀 직렬화기를 사용
        ?: value
    serializePropertyValue(jsonValue)
}
```

### JSON 파싱과 객체 역직렬화

```kotlin
inline fun <reified T : Any> deserialize(json: String): T
```

단계

- 어휘 분석기(lexical analyzer): 렉서(lexer)
    - 문자열을 토큰으로 분리
        - 문자 토큰: 문자를 표현 (콤마, 콜론, 중괄호, 대괄호 등)
        - 값 토큰: 문자열, 숫자, 불리언, null
- 문법 분석기(syntax analyzer): 파서(parser)
    - 토큰의 리스트를 구조화된 표현으로 변환
    - 의미 단위: 키/값 쌍, 배열
- 역직렬화 컴포넌트: 파싱 결과로 객체 생성

JSON 파서 콜백 인터페이스

```kotlin
interface JsonObject {
    fun setSimpleProperty(propertyName: String, value: Any?)
    fun createObject(propertyName: String): JsonObject
    fun createArray(propertyName: String): JsonObject
}
```

![image](https://github.com/uplus-study/kotlin-study/assets/109255398/012f7b78-6444-4ac8-8322-f6e61024bfce)

JSON 데이터로부터 객체를 만들어내기 위한 인터페이스

```kotlin
interface Seed : JsonObject {
    fun spawn(): Any? // 객체를 만들어내는 함수
    fun createCompositeProperty( // 중첩된 객체/리스트를 만들 때 사용
        propertyName: String,
        isList: Boolean
    ): JsonObject
    override fun createObject(propertyName: String) = createCompositeProperty(propertyName, false)
    override fun createArray(propertyName: String) = createCompositeProperty(propertyName, true)
}
```

최상위 역직렬화 함수

```kotlin
fun <T : Any> deserialize(json: String, targetClass: KClass<T>): T {
    val seed = ObjectSeed(targetclass, ClassInfoCache())
    Parser(json, seed).parse()
    return seed.spawn() as T
}
```

객체 역직렬화하기

```kotlin
class ObjectSeed<out T : Any>(
    targetClass: KClass<T>,
    val classInfoCache: ClassInfoCache
) : Seed {
    private val classInfo: ClassInfo<T> = // targetClass를 만들 때 필요한 정보 캐시
        classInfoCache[targetClass]
    private val valueArguments = mutableMapOf<KParameter, Any?>()
    private val seedArguments = mutableMapOf<KParameter, Seed>()
    private val arguments: Map<KParameter, Any?> // 생성자 파라미터와 그 값을 담음
        get() = valueArguments +
                seedArguments.mapValues { it.value.spawn() }
    override fun setSimpleProperty(propertyName: String, value: Any?) {
        val param = classInfo.getConstructorParameter(propertyName)
        valueArguments[param] = // 널생성자 파라미터 값이 간단한 값인 경우 값 기록
            classInfo.deserializeConstructorArgument(param, value)
    }
    override fun createCompositeProperty(
        propertyName: String, isList: Boolean
    ): Seed {
        val param = classInfo.getConstructorParameter(propertyName)
        val deserializeAs = // 프로퍼티에 대한 애노테이션을 찾음
            classInfo.getDeserializeClass(propertyName)
        val seed = createSeedForType( // 파라미터 타입에 따라 새 Seed 인스턴스를 만듦
            deserializeAs ?: param.type.javaType, isList
        )
        return seed.apply { seedArguments[param] = this } // 시드 객체를 맵에 기록
    }
    override fun spawn(): T = // 맵을 넘겨서 인스턴스를 만든다
        classInfo.createInstance(arguments)
}
```

### 최종 역직렬화 단계: callBy(), 리플렉션을 사용해 객체 만들기

`KCallable.call`: 디폴트 파라미터 값을 지원하지 않음
-> callBy() 사용

```kotlin
interface KCallable<out R> {
    fun call(vararg args: Any?): R
    fun callBy(args: Map<KParameter, Any?>): R
    ...
```
- callBy: 파라미터 이름과 값을 매핑한 맵을 인자로 받음
    - 키 값에 파라미터가 없는데, 디폴트 값이 있는 경우: 디폴트 값 사용
    - 장점: 파라미터의 순서를 신경 쓰지 않아도 된다
- 타입 처리 주의
    - args 맵에 들어있는 값의 타입이 파라미터 타입과 일치해야 한다
        - `KPrameter.type.isInstance(args[key])`를 사용해 타입 검사

#### ClassInfoCache

- 리플렉션 연산 비용을 줄이기 위한 클래스
- 역직렬화: 프로퍼티가 아니라 생성자 파라미터를 다룸 -> 애노테이션이 프로퍼티에 적용되기 때문에 파라미터에 해당하는 프로퍼티를 찾아야 한다
    - 클래스별로 한 번만 검색을 수행하고 결과를 캐싱
```kotlin
class ClassInfoCache {
    private val cache = mutableMapOf<KClass<*>, ClassInfo<*>>()
    @Suppress("UNCHECKED_CAST")
    operator fun <T : Any> get(targetClass: KClass<T>): ClassInfo<T> =
        cache.getOrPut(targetClass) { ClassInfo(targetClass) } as ClassInfo<T>
}
```

필수 파라미터가 모두 있는지 검증하기
```kotlin
private fun ensureAllParametersPresent(args: Map<KParameter, Any?>) {
    for (param in (constructor.parameters)) {
        if (args[param] == null && !param.isOptional) {
            throw IllegalArgumentException("Missing value for parameter ${param.name}")
        }
    }
}
```

파라미터에 디폴트 값이 있다면 `param.isOptional`이 true이다
