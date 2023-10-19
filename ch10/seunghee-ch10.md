# 10장 애노테이션과 리플렉션

애노테이션과 리플렉션을 사용하면 함수가 정의된 클래스의 이름과 함수 이름, 파라미터 이름 알아야 하는 제약에서 벗어나 임의의 클래스를 다룰 수 있음.
- 애노테이션 사용: 라이브러리가 요구하는 의미를 클래스에게 부여할 수 있음.
- 리플렉션을 사용: 실행 시점에 컴파일러 내부 구조를 분석할 수 있음.

## 애노테이션 선언과 적용
### 애노테이션 적용
> **애노테이션의 인자:** 원시 타입의 값, 문자열, enum, 클래스 참조, 다른 애노테이션 클래스, 언급된 요소들로 이뤄진 배열 등이 가능
- 클래스를 애노테이션 인자로 지정할 때: @MyAnnotation(MyClass::class) // ::class를 클래스 이름 뒤에 뒤에 넣어야 함
- 다른 애노테이션을 인자로 지정할 때: 인자로 들어가는 애노테이션의 이름 앞에 @를 넣지 않아야 함 (Deprecated의 인자로 들어가므로 ReplaceWith앞에 @ 삭제)
```kotlin
@Deprecated("Use removeAt(index) instead.", ReplaceWith("removeAt(index)"))
fun remove(index: Int) {
...
}
```
- 배열을 인자로 지정할 때: 아래와 같이 arrayOf 함수를 사용
```kotlin
@RequestMapping(path = arrayOf("/foo", "/bar"))
```
- 자바에서 선언한 애노테이션 클래스를 사용할 때: value라는 이름의 파라미터가 필요에 따라 자동으로 가변 길이 인자로 변환되므로 아래처럼 arrayOf함수 쓰지 않아도 됨.
```kotlin
@JavaAnnotationWithArrayValue("abc", "foo", "bar")처럼 arrayOf 함수를 쓰지 않아도 됨
```

> **어노테이션 인자** :
애노테이션 인자를 컴파일 시점에 알 수 있어야 함 -> 임의의 프로퍼티를 인자로 지정할 수 없음 -> 프로퍼티를 애노테이션 인자로 const 붙여야함
```kotlin
const val TEST_TIMEOUT = 100L
@Test(timeout = TEST_TIMEOUT) fun testMethod() { ... }
```

### 메타 애노테이션: 애노테이션을 처리하는 방법 제어
애노테이션 클래스에 적용할 수 있는 애노테이션
```kotlin
@Target(AnnotaionTarget.PROPERTY)
annotation class JsonExclude
```
@Target 메타애노테이션은 애노테이션을 적용할 수 있는 요소의 유형을 지정함.
애노테이션 클래스에 대해 구체적인 @Target을 지정하지 않으면 모든 선언에 적용할 수 있는 애노테이션이 됨

>> @Retention 애노테이션
@Retention은 정의 중인 애노테이션 클래스를 소스 수준에서만 유지할지 .class 파일에 저장할지, 실행 시점에 리플렉션을 사용해 접근할 수 있게 할지를 지정하는 메타애노테이션

### 리플렉션: 실행 시점에 코틀린 객체 내부 관찰
리플렉션은 실행 시점에 (동적으로) 객체의 프로퍼티와 메소드에 접근할 수 있게 해주는 방법
보통 객체의 메소드나 프로퍼티에 접근할 때 컴파일러는 그 선언을 정적 컴파일 시점 보장하지만
타입과 관계없이 객체를 다뤄야 하거나 객체가 제공하는 메소드나 프로퍼티 이름을 오직 실행 시점에만 알 수 있는 경우에 리플렉션 사용

> 예시: JSON 직렬화 라이브러리
직렬화 라이브러리는 어떤 객체든 JSON으로 변환할 수 있어야 하고, 실행 시점이 되기 전까지는 라이브러리가 직렬화할 프로퍼티나 클래스에 대한 정보를 알 수 없음

코틀린 리플렉션 사용 시 두 가지 서로 다른 리플렉션 API 사용 필요
- 자바가 java.lang.reflect 패키지를 통해 제공하는 표준 리플렉션 API: 코틀린 클래스는 일반 자바 바이트코드로 컴파일되므로 자바 리플렉션 API도 코틀린 클래스를 컴파일한 바이트코드를 지원
- 코틀린이 kotlin.reflect 패키지를 통해 제공하는 코틀린 리플렉션 API: 자바에는 없는 프로퍼티나 널이 될 수 있는 타입과 같은 코틀린 고유 개념에 대한 리플렉션을 제공, 현재 코틀린 리플렉션 API는 자바 리플렉션 API를 완전히 대체할 수 있는 복잡한 기능을 제공하지는 않는다.

코틀린 리플렉션 API: KClass, KCallable, KFunction, KProperty
- 클래스를 표현하는 KClass
- MyClass:class라는 식을 쓰면 KClass의 인스턴스를 얻을 수 있음, 클래스를 컴파일 시점에 알고 있다면 KClass 인스턴스를 얻기 위해 ClassName::class를 사용하지만 실행 시점에 obj 변수에 담긴 객체로부터 KClass 인스턴스를 얻기 위해서는 obj.javaClass.kotlin을 사용함
  아래 코드는 클래스 이름과 그 클래스에 들어있는 프로퍼티 이름을 출력하고 member Properties를 통해 클래스와 모든 조상 클래스 내부에 정의된 비확장 프로퍼티를 모두 가져오는 코드임
  class Person(val name: String, val age: Int)
```kotlin
class Person(val name: String, val age: Int)

val person = Person("Alice", 29)
val kClass = person.javaClass.kotlin 
println(kClass.simpleName) => Person

kClass.memberProperties.forEach { 
    println("${it.name}") 
}
=> age
-> name
```

KClass 선언을 보면 클래스의 내부를 살펴볼 때 사용할 수 있는 다양한 메소드 제공
```kotlin
interface KClass<T : Any> {
val simpleName: String?
val qualifiedName: String?
val members: Collection<KCallable<*>>
val constructors: Collection<KFunction<T>>
val nestedClasses: Collection<KClass<*>>
...
}
```


### 리플렉션을 사용한 객체 직렬화 구현
직렬화 함수는 객체의 모든 프로퍼티를 직렬화 함

```kotlin
private fun StringBuilder.serializeObject(obj: Any) {
val kClass = obj.javaClass.kotlin // 객체의 KClass를 얻는다.                  
val properties = kClass.memberProperties // 클래스의 모든 프로퍼티를 얻는다.         
properties.joinToStringBuilder(
this, prefix = "{", postfix = "}") { prop ->
serializeString(prop.name) // 프로퍼티 이름을 얻는다.                     
append(": ")
serializePropertyValue(prop.get(obj)) // 프로퍼티 값을 얻는다.         
}
}
```

### 애노테이션을 활용한 직렬화 제어
JSON 직렬화 과정을 제어하는 과정 중에서 @JsonExclude를 사용하여 특정 필드들을 제외
```kotlin
private fun StringBuilder.serializeObject(obj: Any) {
obj.javaClass.kotlin.memberProperties
.filter { it.findAnnotation<JsonExclude>() == null }
.joinToStringBuilder(this, prefix = "{", postfix = "}") {
serializeProperty(it, obj)
}
}
```

- KFunction과 KProperty 인터페이스는 모두 KCallable을 확장함.
- KClassable은 제네릭 call 메소드를 제공함
- KCallable.callBy 메소드를 사용하면 메소드를 호출하면서 디폴트 파라미터값을 사용할 수 있음
- KFunction0, KFunctiuon1 등의 인터페이스는 모두 파라미터 수가 다른 함수를 표현하며, invoke 메소드를 사용해 함수를 호출할 수 있음
- KProperty0는 최상위 프로퍼티나 변수, KProperty1은 수신 객체가 있는 프로퍼티에 접근할 때 쓰는 인터페이스이고, 두 인퍼테이스 모두 GET 메소드를 사용해 프로퍼티 값을 가져올 수 있음
- KMutableProperty0과 KMutableProperty1은 각각 KProperty0과 KProperty1을 확장하며, set 메소드를 통해 프로퍼티값을 변경할 수 있게 해줌
