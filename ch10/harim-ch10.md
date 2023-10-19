# 애노테이션과 리플렉션
미리 알지 못하는 임의의 클래스를 다룰 때 사용
- 애노테이션: 라이브러리가 요구하는 의미를 클래스에 부여
- 리플렉션: 실행 시점에 컴파일러 내부 구조 분석 가능

## 애노테이션 선언과 적용

### 애노테이션 적용
자바와 같은 방법으로 애노테이션 사용
- 적용하려는 대상 앞에 애노테이션을 붙임
- `@애노테이션이름` 
- 함수나 클래스 등에 붙임

`@Deprecated` 
```kt
@Deprecated("Use removeAt(index) instead.", ReplaceWith("removeAt(index)"))
fun remove(index: Int){ ... }
``` 
- 코틀린에서는 replaceWith 파라미터로 옛버전을 대신할 수 있는 패턴 제시 가능
- reomve 호출하는 코드에 대해 경고메시지 표시 + 코드를 새로운 API 버전에 맞는 코드로 바꿔주는 quick fix 제시


**애노테이션 인자**
> 가능한 값: 원시 타입 값, 문자열, enum, 클래스 참조, 다른 애노테이션 클래스, 앞의 요소로 이루어진 배열
  
애노테이션 인자 지정 문법
- 클래스를 애노테이션 인자로 지정할 때 `::class` 를 클래스 이름 뒤에 넣음
  - `@MyAnnotation(MyClass::class)`
- 다른 애노테이션을 인자로 지정할 때는 인자로 들어가는 애노테이션 앞에 @ 넣지 않음
  - `ReplaceWith`는 애노테이션이지만 `Deprecated`의 인자로 들어가므로 @ 없음
- 배열을 인자로 지정할 때는 arrayOf 함수 사용
  - `@RequestMapping(path = arrayOf("/foo", "/bar"))`
  - 자바에서 선언한 애노테이션 클래스의 경우 value 파라미터가 필요에 따라 가변 길이 인자로 변환되는데, 이때는 arrayOf 함수 사용 안해도 됨

애노테이션 인자는 컴파일 시점에 알 수 있어야 함
따라서 임의의 프로퍼티를 인자로 지정할 수 없음

프로퍼티를 애노테이션 인자로 사용하려면 `const` 변경자 붙여야 함
```kt
// const 붙은 프로퍼티는 파일 맨 위나 object 안 선언
// 원시 타입이나 String으로 초기화 해야 함
const val TEST_TIMEOUT = 100L 

@Test(timeout = TEST_TIMEOUT) fun testMethod() { ... }
```

### 애노테이션 대상

애노테이션을 붙일 때 어떤 요소에 붙일지 표시 필요
- 사용 지점 대상 (use-site target): 애노테이션 붙일 요소 결정 가능
  - `@사용지점대상:애노테이션이름`
- 예시
  ```kt
  class hasTempFolder {
    @get:Rule 
    val folder = TemporaryFolder() 
    
    @Test
    fun testUsingTempFolder() {
        val createdFile = folder.newFile("myfile.txt")
        val createdFolder = folder.newFolder("subfolder")
    }
  }
  ```
  - 테스트 메서드 실행 규칙은 공개 필드나 메서드 앞에 지정
  - 코틀린 필드는 기본적으로 비공개이므로 `folder` 앞에 `@Rule`을 붙이면 예외 발생 

자바에 선언된 애노테이션을 사용해 프로퍼티에 애노테이션을 붙이는 경우, 기본적으로 **프로퍼티의 필드**에 애노테이션이 붙음

하지만 코틀린으로 애노테이션 선언하면 **프로퍼티**에 직접 적용 가능한 애노테이션 만들 수 있음

**사용 지점 대상 목록**
- property -> 자바 선언된 애노테이션은 X
- field
- get
- set
- receiver
- param
- setparam
- delegate
- file
  - package 선언 앞에서 파일 최상위 수준에만 적용 가능

자바와 달리 코틀린에서는 애노테이션 인자로 클래스나 함수 선언, 타입 외에 임의 식 허용
```kt
fun test(list: List<*>) {
    @Suppress("UNCHECKED_CAST") // 컴파일 경고 무시하기 위해 사용
    val strings = list as List<String>
}
```

> ### 자바 API를 애노테이션으로 제어하기
>
> 코틀린은 코틀린으로 선언한 내용을 자바 바이트코드로 컴파일하는 방법과 코틀린 선언을 자바에 노출하는 방법을 제어하기 위해 애노테이션 많이 제공
>
> 애노테이션 중 일부는 자바 언어의 일부 키워드를 대신함
>
> **코틀린 선언을 자바에 노출시키는 방법 변경하는 애노테이션**
>
> - @JvmName: 코틀린 선언이 만들어내는 자바 필드나 메서드 이름 변경
> - @JvmStatic: 메서드, 객체 선언, 동반 객체에 적용하면 요소가 정적 메서드로 노출
> - @JvmOverloads: 디폴트 파라미터 값이 있는 함수에 대해 컴파일러가 자동으로 오버로딩한 함수 생성
> - @JvmField: 게터나 세터가 없는 public 자바 필드로 프로퍼티 노출


### 애노테이션을 활용한 JSON 직렬화 제어

자바와 JSON 변환할 때 자주 쓰이는 라이브러리: Jackson, GSON => 코틀린 호환 O

예제로 JSON 직렬화 위한 순수 코틀린 라이브러리 제이키드 구현

```kt
data class Person(val name: String, val age: Int) 

>>> val person = Person("Alice", 29)
>>> println(serialize(person)) //Person 인스턴스 serialize 함수에 전달
{"age": 29, "name": "Alice"} //키, 값 쌍 -> 인스턴스의 프로퍼티 이름과 값

>>> val json = """{"name": "Alice", "age": 29}"""
>>>println(deserialize<Person>(json)) //역직렬화, 타입 인자로 클래스 명시 필요
Person(name=Alice, age=29)
```

애노테이션을 활용해서 객체를 직렬화/역직렬화하는 방법 제어
- 제이키드 라이브러리는 JSON 직렬화할 때 모든 프로퍼티를 직렬화하고 프로퍼티 이름을 키로 사용함
- 애노테이션 사용으로 이런 동작을 변경!
  - `@JsonExclude`: 직렬화/역직렬화 시 프로퍼티 무시
  - `@JsonName`: 프로퍼티를 표현하는 키/값 쌍의 키로 프로퍼티 이름 대신 애노테이션이 지정한 이름 쓰게 할 수 있음 
> age 프로퍼티는 반드시 디폴트 값 지정 필요 -> 안하면 역직렬화 시 인스턴스 만들 수 X
```kt
data class Person(
    @JsonName("alias") val firstName: String, //json으로 저장 시 키 변경
    @JsonExclude val age: Int? = null //직렬화, 역직렬화에 사용 X
)
```
![img](https://github.com/uplus-study/kotlin-study/assets/126530537/97694911-1678-4edb-be8d-5558066ded2a)

### 애노테이션 선언

```kt
annotation class JsonExclude //파라미터 없는 가장 단순한 애노테이션
```
- class 앞 `annotation` 변경자 붙어있음
- 애노테이션 클래스는 선언이나 식과 관련 있는 메타데이터 구조를 정의함 => 내부에 아무 코드 X => 컴파일러가 본문 정의하지 못하게 막음
- 파라미터 있는 경우 **주 생성자에 파라미터 선언**
  - 모든 파라미터 앞 `val` 키워드를 붙여야 함
    ```kt
    annotation class JsonName(val name: String)
    ```
- 자바 애노테이션 선언과 비교
    ```java
    public @interface JsonName {
        String value();
    }
    ```
  - 코틀린에서는 name 프로퍼티를 사용, 자바에서는 value라는 메서드 사용
  - 자바에서는 애노테이션 적용할 때 value 제외한 모든 attribute에는 이름 명시해야 함
  - 코틀린에서는 일반적인 생성자 호출과 같아서 이름 생략 가능
    ```kt
    //둘은 같음
    @JsonName(name = "first_name")
    @JsonName("first_name") 
    ```
  - 자바에서 선언한 애노테이션을 코틀린에서 적용할 때 value 제외한 모든 인자에 이름 붙인 인자 구문 사용해야 함

### 메타애노테이션
**메타애노테이션**: 애노테이션 클래스에 적용할 수 있는 애노테이션 
- 자바, 코틀린 모두 가능
- 표준 라이브러리에서 제공하는거 존재, 컴파일러가 애노테이션 처리하는 방법 제어
- 프레임워크 중에서도 메타애노테이션 제공하는 경우 있음

가장 많이 사용되는 매타애노테이션 `@Target`
- JsonExclude, JsonName 애노테이션에서도 적용 가능 대상 지정 위해 사용
    ```kt
    @Target(AnnotationTarget.PROPERTY)
    annotation class JsonExclude
    ```
- @Target 지정하지 않으면 모든 선언에 적용 가능
- 애노테이션이 붙을 수 있는 대상이 정의 된 enum 값 = AnnotationTarget
  - 클래스, 파일, 프로퍼티, 프로퍼티 접근자, 타입, 식 등
  - 둘 이상의 대상 한꺼번에 선언 가능
    - `@Target(AnnotationTarget.CLASS, AnnotationTarget.METHOD)`

- 메타애노테이션 직접 선언한다면 `ANNOTATION_CLASS` 를 대상으로 지정
```kt
@Target(AnnotationTarget.ANNOTATION_CLASS)
annotation class BindingAnnotation

@BindingAnnotation //애노테이션에 붙는 애노테이션 => 메타애노테이션
annotation class MyBinding
```

- 대상을 PROPERTY로 지정한 애노테이션은 자바에서 사용 X
  - 자바에서 사용하려면 FIELD를 두 번째 대상으로 추가 필요

> `@Retention 애노테이션`
>
> - 정의 중인 애노테이션 클래스를 소스 수준에서 유지할지
> - .class 파일에 저장할지
> - 실행 시점에 리플렉션 사용해 접근할 수 있게 할지
>
> 정하는 애노테이션
>
> 자바 컴파일러는 기본적으로 애노테이션 .class 파일에 저장, 런타임에 사용 x
>
> 하지만 대부분 애노테이션은 런타임에 사용할 수 있어야 함 
> => 코틀린에서는 기본적으로 애노테이션 @Retention을 `RUNTIME`으로 지정

<br>

### 애노테이션 파라미터로 클래스 사용
**jkid @DeserializeInterface** 
- 인터페이스 타입인 프로퍼티에 대한 역직렬화를 제어할 때 사용
- 인터페이스의 인스턴스를 직접 만들 수는 없기 때문에 역직렬화 시 어떤 클래스를 사용해 인터페이스 구현할지 지정
```kt
interface Company {
    val name: String
}

data class CompanyImpl(override val name: String): Company 

data class Person(
    val name: String,
    @DeserializeInterface(CompanyImpl::class) val company: Company
)
```
- Person 역직렬화 과정에서 company 프로퍼티를 CompanyImpl의 인스턴스를 만들어서 설정
- 역직렬화를 사용할 클래스 지정할 때 @DeserializeInterface 사용
> 클래스 가리킬 때 클래스 이름 뒤에 `::class` 키워드 사용

<br>

**클래스 참조를 인자로 받는 애노테이션 정의**
```kt
annotation class DesirializeInterface(val targetClass: KClass<out Any>)
```
- `KClass` 자바의 java.lang.Class 타입과 같은 역할을 하는 코틀린 타입
- KClass의 타입 파라미터는 이 KClass 인스턴스가 가리키는 코틀린 타입 지정
  - CompanyImpl::class의 타입은 KClass<CompanyImpl>
![img](https://github.com/uplus-study/kotlin-study/assets/126530537/8d491998-715a-47ee-a71e-e2de6249da7e)
- out 키워드로 모든 코틀린 타입 T에 대해 `KClass<T>` 가 `KClass<out Any>` 하위 타입이 됨 => DeserializeInterface 인자로 Any 확장하는 모든 클래스에 대한 참조 전달 가능

### 애노테이션 파라미터로 제네릭 클래스 받기

제이키드는 기본적으로 원시타입 아닌 프로퍼티를 중첩된 객체로 직렬화

변경하고 싶다면 값을 직렬화하는 로직 직접 제공 => `@CustomSerializer`
- 커스텀 직렬화 클래스에 대한 참조를 인자로 받음
- 직렬화 클래스는 `ValueSerializer` 인터페이스 구현해야 함
```kt
interface ValueSerializer<t> {
    fun toJsonValue(value: T): Any?
    fun fromJsonValue(jsonvalue: Any?): T
}

//날짜 직렬화 예제
data class Person(
    val name: String,
    @CustomSerializer(DateSerializer::class) val birthDate: Date //날짜 직렬화위해 ValueSerializer<Date> 구현한 DateSerializer 사용함
)
```
- `@CustomSeiralizer` 애노테이션 구현 방법
```kt
annotation class CustomSerializer(
    val serializerClass: KClass<out ValueSerializer<*>> //어떤 타입에 쓰일지 알 수 없으므로 스타 프로젝션 사용
)
```
![img](https://github.com/uplus-study/kotlin-study/assets/126530537/e8c744d3-7847-4858-a1d0-914eec09ca53)
- CustomSerializer가 ValueSerializer 구현하는 클래스만 인자로 받아야 함 => `out ValueSerializer<*>
- 클래스를 애노테이션 인자로 받아야 하면 애노테이션 파라미터 타입에 `KClass<out 허용클래스이름>` 씀

## 리플렉션
**리플렉션**: 실행 시점에 (동적으로) 객체의 프로퍼티와 메서드에 접근할 수 있게 해주는 방법

- 보통 객체의 메서드, 프로퍼티 접근할 때 프로그램 소스코드 안 구체적인 선언이 있는 메서드나 프로퍼티 이름 사용
- 컴파일러는 이름이 실제로 가리키는 선언을 컴파일 시점에 찾아 해당하는 선언이 실제 존재하는지 보장
- 타입과 관계없이 객체 다뤄야 하거나 객체가 제공하는 메서드나 프로퍼티 이름을 오직 실행 시점에만 알 수 있는 경우 존재
  - JSON 직렬화 라이브러리는 어떤 객체든 JSON으로 변환할 수 있어야 하고, 실행 시점 되기 전까지 직렬화할 프로퍼티나 클래스 정보를 알 수 없음 

*=> 리플렉션 사용!*

**코틀린에서 리플렉션 사용하는 방법**
- 자바가 제공하는 표준 리플렉션 `java.lang.reflect` 
  - 자바 리플렉션 API는 코틀린 클래스 컴파일한 바이트코드 완벽하게 지원 => 호환성👍
- 코틀린 리플렉션 API `kotlin.reflect` 
  - 자바에는 없는 프로퍼티나 널이 될 수 있는 타입 같은 코틀린 고유 개념에 대한 리플렉션 제공
  - but 자바 리플렉션 API를 완전히 대체할 수 있는 복잡한 기능을 제공하지는 앖음
  - 다른 JVM 언어에서 생성한 바이트코드 다룰 수 있음 (코틀린만 다루는게 아님!)
> 코틀린 리플렉션 API는 kotlin-reflect.jar 라는 별도의 jar 파일에 담겨 제공
>
> 새 프로젝트 생성할 때 리플렉션 패키지 jar 파일 의존관계가 자동으로 추가되지 X
>
> 따라서 코틀린 리플렉션 API 사용한다면, 직접 프로젝트 의존관계에 리플렉션 jar 파일 추가해야 함

### 코틀린 리플렉션 API: KClass, KCallable, KFunction, KProperty
**KClass**
- 클래스 안 모든 선언 열거하고 각 선언에 접근하거나 상위 클래스 얻는 등의 작업 가능
- `MyClass::class` => KClass 인스턴스 얻을 수 있음
- 실행 시점에 객체의 클래스 얻으려면 객체의 javaClass 프로퍼티를 사용해 객체의 자바 클래스를 얻어야 함  
  - `javaClass` = 자바의 `java.lang.Objet.getClass()` 
- 자바 클래스 얻었으면 .kotlin 확장 프로퍼티를 통해 자바에서 코틀린 리플렉션 API로 옮겨옴

```kt
class Person(val name: String, val age: Int)

>>> import kotlin.reflect.full.* //memberProperties 확장 함수 임포트
>>> val person = Person("Alice", 29)
>>> val kClass = person.javaClass.kotlin //KClass<Person> 인스턴스 반환
>>> println(kClass.simpleName)
Person
>>> kClass.memberProperties.forEach { println(it.name) } 
age
name
```

- memberProperties 등 KClass에서 사용할 수 있는 다양한 기능은 kotlin-reflect 라이브러리를 통해 제공하는 확장 함수 => `kotlin.reflect.full.*` 임포트 해서 사용

**KCallable**
- KClass 모든 멤버의 목록 KClassble 인스턴스의 컬렉션
```kt
interface KClass<T: Any> {
    ...
    val members: Collection<KCallable<*>>
    ...
}
```
- KCallable은 함수와 프로퍼티 아우르는 공통 상위 인터페이스
- call 메서드로 함수나 프로퍼티의 게터 호출 가능
```kt
interface KCallable<out R> {
    fun call(vararg args: Any?): R
    ...
}

fun foo(x: Int) = print(x)
>>> val kFunction = ::foo 
>>> kFunction.call(42)
42
```
- `::foo` 값 타입 => 리플렉션 API에 있는 **KFunction** 클래스의 인스턴스
- call에 넘긴 인자 개수와 원래 함수에 정의한 파라미터 개수가 맞아야 함
- 함수 호출 위해 더 구체적인 메서드 사용 가능
  - `::foo`의 타입 `KFunction1<Int, Unit>`에 파라미터와 반환 값 타입 정보 들어있음
  - KFunction1 인터페이스 통해 함수 호출하려면 invoke 메서드 사용
  - invoke는 정해진 개수의 인자만 받아들이고, 인자는 KFunction1 제네릭 인터페이스의 첫 번째 타입 파라미터
```kt
import kotiln.reflect.KFuncion2

fun sum(x: Int, y: Int) = x + y

>>> val kFunction: KFunction2<Int, Int, Int> = ::sum
>>> println(kFunction.invoke(1, 2) + kFunction(3, 4))
10
>>> kFunction(1) 
ERROR: No value passed for parameter p2
```
- invoke 를 사용해서 컴파일 시점에 인자 개수나 타입 매칭 확인 가능
- call 은 모든 타입의 함수에 적용할 수 있는 일반적인 메서드지만 타입 안전성 보장 X 

=> kFunction 인자 타입과 반환 타입 모두 안다면 invoke 메서드 사용
 
> **언제, 어떻게 KFunctionN 인터페이스 정의?**
>
> 각 KfunctionN 은 KFunction을 확장하고 N과 파라미터 개수 같은 invoke를 추가로 포함
>
> 이런 함수 타입들은 컴파일러가 생성한 **함성 타입(synthetic compiler-generated type)**
>
> 따라서 kotlin.reflect 패키지에서는 타입 정의 찾을 수 없음
>
> 합성 타입 사용으로 코틀린은 kotlin-runtime.jar 파일 크기를 줄일 수 있고 함수 파라미터 개수에 대한 인위적인 제약 피할 수 있음

**KProperty**의 call 메서드 호출 가능 
- 프로퍼티 게터 호출

하지만 프로퍼티 인터페이스는 프로퍼티 값 얻는 더 좋은 방법으로 **get 메서드** 제공
- get 메서드 접근하려면 프로퍼티 선언된 방법에 따라 올바른 인터페이스 사용해야 함
- 최상위 프로퍼티는 KProperty0 인터페이스의 인스턴스로 제공, 인자 없는 get 메서드 있음
```kt
var counter = 0
>>> val kProperty = ::counter
>>> kProperty.setter.call(21) //리플렉션 기능으로 세터 호출하여 인자 넘김
>>> println(kProperty.get()) //get 호출해 프로퍼티 값 가져옴
21
```
멤버 프로퍼티는 KProperty1 인스턴스로 표현됨, 인자 1개인 get메서드 들어있음
- 멤버 프로퍼티는 어떤 객체에 속해 있으므로 get 메서드에 프로퍼티 얻고자 하는 객체 인스턴스를 넘김
```kt
class Person(val name: String, val age: Int)

>>> val person = Person("Alice", 29)
>>> val memberProperty = Person::age //KProperty<Person, Int> 타입, 각각 수신객체타입, 프로퍼티 타입
>>> println(memberProperty.get(person)) //동적으로 person.age 가져옴
29
```

최상위 수준이나 클래스 안에 정의된 프로퍼티만 리플렉션으로 접근할 수 있음

함수의 로컬 변수에는 접근 X

![img](https://github.com/uplus-study/kotlin-study/assets/126530537/4794658f-af4a-4e54-a515-fb3828431f05)
실행 시점에 소스코드 요소에 접근하기 위해 사용하는 인터페이스 계층 구조


KClass, KFunction, KParameter -> KAnnotationElement 확장
- KClass: 클래스와 객체 표현
- KProperty: 모든 프로퍼티 표현
- KMutableProperty: var로 정의한 변경 가능한 프로퍼티 표현
- Getter, Setter: KProperty에 선언, KFunction 확장

<br>

### 리플렉션 사용한 객체 직렬화 구현
```kt
fun serialize(obj: Any): String //직렬화 함수, 객체 받아서 JSON 표현을 문자열로 반환
```
- 이 함수는 객체의 프로퍼티와 값을 직렬화하면서 StringBuilder 객체 뒤에 직렬화한 문자열을 추가함
- append 간결하게 하기 위해 직렬화 기능을 StringBuilder의 확장 함수로 구현
```kt
private fun StringBuilder.serializeObject(x: Any) {
    append(...)
}
```
- serializeObject 는 StringBuilder API를 확장 X
- 이 메서드는 여기서 벗어나면 쓸모가 없어짐 => private으로 지정
- 확장 함수 정의하여 serialize는 대부분의 작업을 serializeObject에 위임
```kt
fun serialize(obj: Any): String = buildString { serializeObject(obj) }
```
- `buildString`은 StringBuilder 생성해서 인자로 받은 람다 넘김

직렬화 함수 기능
- 원시 타입/문자열 -> JSON 수, 불리언, 문자열 값 등으로 변환
- 컬렉션 -> JSON 배열
- 모두 아닌 프로퍼티 -> 중첩된 JSON 객체
```kt
//객체 직렬화
private fun StringBuilder.serializeObject(obj: Any) {
  val kClass = obj.javaClass.kotlin
  val properties = kClass.memberProperties //클래스의 모든 프로퍼티 얻음

  properties.joinToStringBuilder( 
      this, prefix = "{", postfix = "}") { prop -> 
    serializeString(prop.name) //프로퍼티 이름 얻음
    append(": ")
    serializePropertyValue(prop.get(obj)) //프로퍼티 값 얻음
    }
}
```
- joinToStringBuilder는 프로퍼티 콤마로 분리
- serializeString은 JSON 명세에 따라 특수문자 이스케이프
- serializePropertyValue는 어떤 값이 원시 타입, 문자열, 컬렉션, 중첩된 객체 중 어떤 건지 확인하고 값에 따라 직렬화
- 어떤 객체의 클래스에 정의된 모든 프로퍼티 열거하기 때문에 정확히 각 프로퍼티가 어떤 타입인지 알 수 없음 
  - prop 변수 타입 => `KProperty1<Any, *>` 
  - prop.get(obj) 는 Any 타입 반환

<br>

### 애노테이션 활용한 직렬화 제어
앞서 JSON 직렬화 과정 제어하는 애노테이션을 serializeObject 함수가 어떻게 처리하는지

- @JsonExclude
  - 어떤 프로퍼티 직렬화 제외할 때
  - KAnnotatedElement 인터페이스에 annotations 프로퍼티 존재
    - annotations는 소스코드상에서 해당 요소에 적용된(@Retention을 RUNTIME으로 지정한) 모든 애노테이션 인스턴스의 컬렉션
  - KProperty는 KAnnotatedElement 확장하므로 property.annotations 통해 프로퍼티의 모든 애노테이션을 얻을 수 있음
  - findAnnotation 함수를 통해 특정 애노테이션을 찾음
  ```kt
  inline fun <reified T> KAnnotatedElement.findAnnotation(): T? 
    = annotations.filterIsInstance<T>().firstOrNull()
  ```
  - 인자로 전달받은 타입에 해당하는 애노테이션이 있으면 그 애노테이션 반환
  ```kt
  val properties = kClass.memberProperties
    .filter { it.findAnnotation<JsonExclude>() == null }
  ```

- @JsonName
  - 애노테이션에 전달한 인자도 필요함
  ```kt
  val jsonNameAnn = prop.findAnnotation<JsonName>() //@JsonName 애노테이션 있으면 인스턴스 반환
  val propName = jsonNameAnn?.name ?: prop.name //애노테이션에서 name 인자 찾고 인자 없으면 prop.name 사용
  ```

Person 클래스 인스턴스 직렬화
```kt
data class Person(
    @JsonName("alias") val firstName: String, //json으로 저장 시 키 변경
    @JsonExclude val age: Int? = null //직렬화, 역직렬화에 사용 X
)
```
- firstName 프로퍼티 직렬화할 때 jsonNameAnn에 JSONName 애노테이션 클래스에 해당하는 인스턴스 존재 => `jsonNameAnn?.name` 의 값은 "alias"

프로퍼티 필터링 포함하는 직렬화 
```kt
private fun StringBuilder.serializeObject(obj: Any) {
  obj.javaClass.kotlin.memberProperties
  .filter { it.findAnnotation<JsonExclude>() == null }
  .joinToStringBuilder(this, prefix = "{", postfix = "}") {
    serializeProperty(it, obj)
  }
}

//프로퍼티 직렬화 로직 분리
private fun StringBuilder.serializeProperty(
  prop: KProperty1<Any, *>, obj: Any
) {
  val jsonNameAnn = prop.findAnnotation<JsonName>()
  val propName = jsonNameAnn?.name ?: prop.name
  serializeString(propName)
  append(": ")
  serializePropertyValue(prop.get(obj))
}
```

@CustomSerializer 구현
- getSerializer 함수에 기초
  - @CustomSerializer 통해 등록한 ValueSerializer 인스턴스를 반환
```kt
fun KProperty<*>.getSerializer(): ValueSerializer<Any?>? {
  //CustomSerializer 애노테이션 있는지 확인
  val customSerializerAnn = findAnnotation<CustomSerializer>() ?: return null
  //CustomSerializer의 ValueSerializer 구현한 인스턴스 프로퍼티인 serializerClass
  val serializerClass = customSerializerAnn.serializerClass
  val valueSerializer = serializerClass.objectInstance
    ?: serializerClass.createInstance()

  @Suppress("UNCHECKED_CAST")
  return valueSerializer as ValueSerializer<Any?>
}
```
- getSerializer가 주로 KProperty 다루므로 KProperty의 확장 함수로 정의
- @CustomSerializer 값으로 클래스와 객체 처리할 때 KClass 클래스로 표현됨
- 다만 객체에 object 선언에 의해 생성된 싱글턴을 가리키는 objectInstance 프로퍼티가 있다는 점이 다름
  - 예를 들어 DateSerializer를 object 로 선언한 경우 objectInstance 프로퍼티에 DateSerializer 싱글턴 인스턴스가 들어있으므로 createInstance 호출할 필요가 없음
- 하지만 KClass가 일반 클래스 표현한다면 createInstance 호출해서 새 인스턴스를 만들어야 함
  
serializeProperty 구현 안에 getSerializer 사용
```kt
private fun StringBuilder.serializeProperty(
  prop: KProperty1<Any, *>, obj: Any
) {
  val name = prop.findAnnotation<JsonName>()?.name ?: prop.name
  serializeString(name)
  append(": ")

  val value = prop.get(obj)
  //커스텀 직렬화가 없으면 일반적인 방법으로 프로퍼티 직렬화
  val jsonValue = prop.getSerializer()?.toJsonValue(value) ?: value
  serializePropertyValue(jsonValue)
}
```

### JSON 파싱과 객체 역직렬화
```kt
inline fun <reified T: Any> deserialize(json: String): T

data class Author(val name: String)
data class Book(val title: String, val author: Author)

>>> val json = """{"title": "Catch-22", "author": {"name": "J.Heller"}}"""
>>> val book = deserialize<Book>(json)
>>> println(book)
Book(title=Catch-22, author=Author(name=J.Heller))
```
- JSON 역직렬화는 JSON 문자열 입력을 파싱하고 리플렉션 사용해 객체 내부에 접근해서 새로운 객체와 프로퍼티 생성하기 때문에 더 어려움

**제이키드 JSON 역직렬화 구현**
1. 어휘 분석기(lexical analyzer, 렉서 lexer)
   - 여러 문자로 이루어진 입력 문자열을 토큰의 리스트로 변환
     - 문자 토큰: 문자 표현함, 중요한 의미를 가짐(콤마, 콜론, 중괄호, 각괄호)
     - 값 토큰: 문자열, 수, 불리언 값, null 상수
2. 문법 분석기(syntax analyzer, 파서 parser)
   - 토큰의 리스트를 구조화된 표현으로 변환
   - JSON 상위 구조 이해하고 토큰을 JSON에서 지원하는 의미 단위로 변환하는 일
   - 의미 단위: 키/값 쌍, 배열
3. 파싱한 결과로 객체 생성하는 역직렬화 컴포넌트

`JsonObject` 인터페이스는 현재 역직렬화하는 중인 객체나 배열을 추적함

파서는 현재 객체의 새로운 프로퍼티 발견할 때마다 프로퍼티 유형에 해당하는 JsonObject의 함수를 호출

```kt
//JSON 파서 콜백 인터페이스
interface JsonObject {
  fun setSimpleProperty(propertyName: String, value: Any?)
  fun createObject(propertyName: String): JsonObject
  fun createArray(propertyName: String): JsonObject
}
```
- 각 메서드의 propertyName 프로퍼티는 JSON 키를 받음
  - 파서가 객체를 값으로 하는 author 프로퍼티를 만나면 createObject("author") 메서드 호출
  - 간단한 프로퍼티 값은 setSimpleProperty 호출하여 실제 값을 value에 넘기는 방식으로 등록
- JsonObject 구현하는 클래스는 새로운 객체 생성하고 새로 생성한 객체를 외부 객체에 등록하는 과정을 책임져야 함

![img](https://github.com/uplus-study/kotlin-study/assets/126530537/86c237ec-0808-4807-9f31-e194178325f5)
- 렉서는 문자열을 토큰 리스트로 바꿈
- 파서는 렉서가 만든 토큰 리스트를 분석하면서 의미 단위 만날 때마다 JsonObject 메서드 적절히 호출
- 역직렬화기는 JsonObject에 상응하는 코틀린 타입의 인스턴스를 만들어내는 JsonObject 구현 제공
  - 클래스 프로퍼티와 JSON 키 사이 대응 관계 찾아내고 중첩된 객체 값(Author) 만들어 냄
- 모든 충접 객체 만들고 난 뒤 필요한 클래스의 인스턴스를 새로 만듦


제이키드는 데이터 클래스와 함께 사용하려는 의도를 가지고 있어서 JSON에서 가져온 이름/값 쌍을 역직렬화하는 클래스의 생성자에 넘김

제이키드는 객체 생성한 다음 프로퍼티 설정하는 것을 지원하지 않음
따라서 제이키드 역직렬화기는 JSON에서 데이터 읽는 과정에서 중간에 만든 프로퍼티 객체들을 어딘가 저장해 뒀다가 생성자 호출할 때 사용해야 함

객체 생성 전 하위 요소 저장해야 하는 것을 생각하면 빌더 패턴과 비슷
- 물론 빌더 패턴은 타입이 미리 정해진 객체 만들기 위한 도구라는 차이 존재

여기서는 빌더 대신 시드(seed) 용어 사용

```kt
//JSON 데이터로 객체 만들어내기 위한 인터페이스
interface Seed: JsonObject {
  fun spawn(): Any? //객체 생성 과정 끝난 후 결과 인스턴스 반환
  fun createCompositeProperty( //중첩된 객체나 리스트 만들 때 사용
    propertyName: String,
    isList: Boolean
  ): Json Object

  override fun createObject(propertyName: String) = 
    createCompositeProperty(propertyName, false)
  override fun createArray(propertyName: String) = 
    createCompositeProperty(propertyName, true)
  
  ...
}
```

```kt
//최상위 역직렬화 함수
fun <T: Any> deserialize(json: Reader, targetClass: KClass<T>): T {
  val seed = ObjectSeed(targetClss, ClassInfoCache())
  Parser(json, seed).parse()
  return seed.spawn()
}
```
- 파싱을 하기 위해 직렬화할 객체의 프로퍼티 담을 ObjectSeed 생성
- 파서 호출하면서 입력 스트림 리더인 json과 시드를 인자로 전달
- 끝나면 spawn 함수 호출해서 결과 객체 생성

```kt
//객체 역직렬화
class ObjectSeed<out T: Any>(
        targetClass: KClass<T>,
        override val classInfoCache: ClassInfoCache
) : Seed {
    //targetclass 인스턴스 만들 때 필요한 정보 캐시
    private val classInfo: ClassInfo<T> = classInfoCache[targetClass] 

    private val valueArguments = mutableMapOf<KParameter, Any?>() //간단한 값 프로퍼티 저장
    private val seedArguments = mutableMapOf<KParameter, Seed>() //복합 프로퍼티 저장

    //생성자 파라미터와 값을 연결하는 맵을 만듦
    private val arguments: Map<KParameter, Any?>
        get() = valueArguments + seedArguments.mapValues { it.value.spawn() }

    override fun setSimpleProperty(propertyName: String, value: Any?) {
        val param = classInfo.getConstructorParameter(propertyName)
        valueArguments[param] = classInfo.deserializeConstructorArgument(param, value) //널생성자 파라미터 값이 간단한 값인 경우 값 기록
    }

    override fun createCompositeProperty(propertyName: String, isList: Boolean): Seed {
        val param = classInfo.getConstructorParameter(propertyName)
        val deserializeAs = classInfo.getDeserializeClass(propertyName)
        val seed = createSeedForType(
                deserializeAs ?: param.type.javaType, isList) //파라미터 타입 분석해서 적절히 ObjectSeed, ObjectListSeed, ValueListSeed 중 생성
        return seed.apply { seedArguments[param] = this }
    }

    //인자 맵 넘겨서 targetClass 타입의 인스턴스 만듦
    override fun spawn(): T = classInfo.createInstance(arguments)
}
```
- ObjectSeed는 결과 클래스에 대한 참조와 결과 클래스 안의 프로퍼티에 대한 정보를 저장하는 캐시인 classInfoCache 객체를 인자로 받음 -> 캐시 정보로 클래스 인스턴스 만듦

### 최종 역직렬화 단계: KCallable.callBy
KClass.call 인자 리스트 받아서 함수나 생성자를 호출해줌, 디폴트 파라미터 값 지원 X

`KCallable.callBy` 사용해서 디폴트 파라미터 값 지원 사용
```kt
interface KCallable<out R> {
  fun callBy(args: Map<KParameter, Any?>): R
}
```
- 파라미터와 파라미터 해당하는 값 연결해주는 맵을 인자로 받음
- 인자로 받은 맵에서 파라미터 없는데 파라미터 디폴트 값 정의돼 있다면 사용
- 파라미터 순서를 지킬 필요 없음
- 타입 처리 주의
  - args 맵에 들어있는 값 타입이 생성자 파라미터 타입과 일치해야 함! 특히 숫자 타입
- callBy 메서드에 생성자 파라미터와 값을 연결해주는 맵 넘기면 객체의 주 생성자 호출 가능
