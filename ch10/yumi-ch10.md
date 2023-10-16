# 애노테이션과 리플렉션

## 애노테이션 선언과 적용

- 자바와 유사
- @ + 이름
- replaceWith 파라미터를 통해 옛 버전을 대신할 수 있는 패턴을 제시

애노테이션 지정 문법

- 클래스 애노테이션 인자로 지정할 때 : @MyAnnotation(MyClass::class)
- 다른 애노테이션을 인자로 지정할 때 ReplaceWith 앞에 @를 사용하지 않음
- 배열을 인자로 지정할 때 : @RequestMapping(path= arrayOf("/foo", "/bar"))
- 프로퍼티를 애노테이션 인자로 사용하려면 앞에 const 변경자를 붙여야 한다.

### 애노테이션 대상

사용자 지점 대상 선언으로 애노테이션을 붙일 요소를 정할 수 있다.

<img width="221" alt="ch10 1" src="https://github.com/uplus-study/kotlin-study/assets/66773030/afed52ee-7690-49de-a7a6-ef0c8f550918">

사용자 지점 대상 지정

- property : 프로퍼티 전체, 자바에서 선언된 애노테이션에서는 사용 불가
- field : 프로퍼티에 의해 생성되는 필드
- get : 프로퍼티 게터
- set : 프로퍼티 세터
- receiver : 확장 함수나 프로퍼티의 수신 객체 파라미터
- param : 생성자 파라미터
- setparam : 세터 파라미터
- delegate : 위임 프로퍼티의 위임 인스턴스를 담아둔 필드
- file : 파일 안에 선언된 최상위 함수와 프로퍼티를 담아둔 클래스

자바와 차의점
애노테이션 인자로 클래스나 함수 선언이나 타입 외의 임의의 식을 허용

~~~kotlin
fun test(List<*>) {
    @Suppress("UNCHECKED_CAST")
    val strings = list as List<String>
}
~~~

> 자바 API를 애노테이션으로 제어하기
> - JvmName : 코틀린 선언이 만들어내는 자바 필드나 메서드 이름을 변경한다.
> - JvmStatic : 메서드, 객체 선언, 동반 객체에 적용하면 그 요소가 자바 정적 메서드로 노출된다.
> - JvmOverloads : 디폴트 파라미터가 있는 함수에 대해 컴파일러가 자동으로 오버로딩한 함수를 생성해준다.
> - JvmField : 프로퍼티를 자바 필드로 노출한다.

### 애노테이션을 활용한 JSON 직렬화 제어

직렬화 : 객체를 저장장치에 저장하거나 네트워크를 통해 전송하기 위해 텍스트나 이진 형식으로 변환하는 것, 반대는 역직렬화

<img width="420" alt="image" src="https://github.com/uplus-study/kotlin-study/assets/66773030/017292ff-262f-4c41-9c61-b0d601978f77">

- @JsonExclude : 직렬화나 역직렬화 시 그 프로퍼티를 무시할 수 있다.
- @JsonName : JSON에서 사용할 이름을 지정할 수 있다.

### 애노테이션 선언

애노테이션 클래스는 오직 선언이나 식과 관련 있는 메타데이터의 구조를 정의하기 때문에 내부에 아무 코드도 들어있을 수 없다.
파라미터가 있는 애노테이션을 정의하려면 애노테이션 클래스의 주 생성자에 파라미터를 선언해야 한다.
자바에서 선언한 애노테이션을 코틀린의 구성 요소에 적용할 때는 value를 제외한 모든 인자에 대해 이름 붙인 인자 구문을 사용해야만 한다.

### 메타애노테이션: 애노테이션을 처리하는 방법 제어

애노테이션 클래스에 적용할 수 이쓴ㄴ 애노테이션

- @Target : 애노테이션을 적용할 요소의 유형을 지정
- AnnotationTarget : 애노테이션이 붙을 수 있는 대상이 정의된 이넘, 둘 이상도 선언 가능

> @Retention 애노테이션
> 정의 중인 애노테이션 클래스를 소스 수준에서만 유지할지, .class 파일에 저장할지, 실행 시점에 리플렉션을 사용해 접근할 수 있게 할지를 지정하는 메타애노테이션
> 기본적으로 RUNTIME으로 지정되어 있다.

### 애노테이션 파라미터로 클래스 사용

어떤 클래스를 선언 메타데이터로 참조할 수 있는 기능이 필요할 때도 사용
ex) @DeserializeInterface

~~~kotlin
@DeserializeInterface(CompanyImpl::class)

annotation class DeserializeInterface(val targetClass: KClass<out Any>)
~~~

out을 쓰지 않으면 오직 Any::class만 인자로 넘길 수 있고, out을 쓰면 Any의 하위 클래스도 가능하다.

### 애노테이션 파라미터로 제네릭 클래스 받기

~~~ kotlin
interface ValueSerializer<T> {
    fun toJsonValue(value: T): Any?
    fun fromJsonValue(jsonValue: Any?): T
}
data class Person(
    @CustomSerializer(DateSerializer::class)
    val birthDate: Date,
    val name: String
)
annotation class CustomSerializer {
val serializerClass: KClass<out ValueSerializer<*>>
}
~~~

- DateSerializer::class는 받아들이지만 Date::class는 받아들이지 않는다.
- out을 사용하면 ValueSerializer를 구현하는 모든 클래스를 받아들인다.
- ValueSerializer를 사용하는 어떤 타입의 값이든 직렬화할 수 있게 허용한다.

## 리플렉션 : 실행 시점에 코틀린 객체 내부 관찰

실행 시점에 동적으로 객체의 프로퍼티와 메서드에 접근할 수 있게 해주는 방법
ex) JSON 직렬화 라이브러리

- 자바가 jva.lang.reflect 패키지를 통해 제공
- 코틀린은 kotlin.reflect 패키지를 통해 제공
- 코틀린 리플렉션 API는 kotlin-reflect.jar라는 별도의 .jar파일에 제공, 직접 프로젝트 의존관계에 리플렉션 .jar 파일을 추가해야 한다.
    - 메이븐 그룹/아티팩트 id : org.jetbrains.kotlin:kotlin-reflect

### 코틀린 리플렉션 API

KClass : 클래스 안에 있는 모든 선언을 열거하고 각 선언에 접근하거나 클래스의 상위 클래스를 얻는 등의 작업이 가능하다.

~~~kotlin
interface KClass<T : Any> {
    val simpleName: String?
    val qualifiedName: String?
    val members: Collection<KCallable<*>>
    val constructors: Collection<KFunction<T>>
    val nestedClasses: Collection<KClass<*>>
}
~~~

KCallable은 함수와 프로퍼티를 아우르는 공통 상위 인터페이스, call을 사용하면 함수나 프로퍼티의 게터 호출 가능

~~~kotlin
interface kCallable<out R>

fun call(vararg args: Any?): R
}

fun foo(x: Int) = println(x)
>>> val kFunction = ::foo
>>> kFunction.call(42)
42
~~~

kFunction의 invoke 메서드는 인자 개수나 타입이 맞아 떨어지지 않으면 호출 불가, call은 모든 타이브이 함수에 적용할 수 있는 일반적인 메서드

KProperty의 call은 프로퍼티의 게터를 호출, get 메서드도 제공

~~~
class Person(val name: String, val age: Int)
>>> val person = Person("Alice", 29)
>>> val memberProperty = Person::age
>>> memberProperty.get(person)
29
~~~

- memberProperty 변수는 KProperty<Person,Int>타입
    - 첫 번째 타입 파라미터는 수신 객체 타입
    - 두 번째 타입 파라미터는 프로퍼티 타입
- 최상위 수준이나 클래스 안에 정의된 프로퍼티만 리플렉션으로 접근 가능, 함수의 로컬 변수에는 접근할 수 없음

코틀린 리플렉션 API의 인터페이스 계층 구조

- <img width="376" alt="image" src="https://github.com/uplus-study/kotlin-study/assets/66773030/f3106a0a-00e8-4b98-bf95-0e77398d0bed">

## 리플렉션을 사용한 객체 직렬화 구현

~~~kotlin
private fun StringBuilder.serializeObject(obj: Any) {
    val kClass = obj.javaClass.kotlin
    val properties = kClass.memberProperties
    properties.joinToStringBuilder(
        this, prefix = "{", postfix = "}") { prop ->
        serializeString(prop.name)
        append(": ")
        serializePropertyValue(prop.get(obj))
    }
}

~~~