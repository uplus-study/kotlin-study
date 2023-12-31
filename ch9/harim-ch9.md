# 제네릭스

### 제네릭 타입 파라미터
- 제네릭스를 사용하면 타입 파라미터를 받는 타입 정의 가능
- 제네릭 타입의 인스턴스 만들려면 타입 파라미터를 구체적인 타입 인자로 치환해야 함
- 코틀린 컴파일러는 보통 타입과 마찬가지로 타입 인자도 추론 가능
- **제네릭 함수**
  - 호출할 때 반드시 구체적 타입으로 타입 인자를 넘겨야 함
  - 컬렉션 다루는 라이브러리 함수는 대부분 제네릭 함수
  ```Kotlin
  val authors = listOf("Dmitry", "Svetlana")
  val readers = mutableListOf<String>(/* ... */)
  fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T>

  readers.filter { it !in authors }
  ```
  - 람다 파라미터에 대해 자동으로 만들어진 변수 IT의 타입은 T(제네릭 타입)
  - 컴파일러는 filter가 List<T> 타입에 대해 호출될 수 있다는 사실을 알고 있음
  - filter의 수신 객체인 reader의 타입이 List<String> 라는 사실을 알고 있음 => `T = String` 이라는 사실을 추론함
  - 클래스나 인터페이스 안 정의된 메서드, 확장 함수, 최상위 함수에서 타입 파라미터 선언 가능
  - 제네릭 확장 프로퍼티 선언 가능
    > 확장이 아닌 일반 프로퍼티는 타입 파라미터 가질 수 없음. 
    > 
    > 클래스 프로퍼티에 여러 타입의 값을 저장할 수 없으므로 제네릭한 일반 프로퍼티는 말이 되지 않음

- **제네릭 클래스**
  - 자바와 마찬가지로 클래스를 제네릭하게 만들 수 있음
  - 제네릭 클래스를 확장하는 클래스 또는 제네릭 인터페이스를 구현하는 클래스를 정의하려면 기반 타입의 제네릭 파라미터에 대해 타입 인자를 지정해야 함
    - 구체적인 타입을 넘길 수도 있고 하위 클래스가 제네릭 클래스라면 타입 파라미터로 받은 타입을 넘길 수 있음
    ```Kotlin
    class StringList: List<String> {
        override fun get(index: int): String = ... }
    class ArrayList<T>: List<T> {
        override fun get(index: Int): T = ...}
    ```
    - StringList 클래스에서는 String 타입의 원소만 포함함
    - ArrayList는 자신만의 타입 파라미터 T를 정의하면서 T를 기반 클래스의 타입 인자로 사용
  - 클래스가 자기 자신을 타입 인자로 참조 가능
  
- **타입 파라미터 제약**
  - 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능
  - 예를 들어 리스트에 속한 모든 원소의 합을 구하는 sum 함수에 숫자 타입만 허용하도록 하는 것
  - 어떤 타입을 제네릭 타입의 타입 파라미터에 대한 상한으로 지정하면 그 제네릭 타입을 인스턴스화할 때 사용하는 타입 인자는 상한 타입이거나 그 상한 타입의 하위 타입만 가능
  - 타입 파라미터 이름 뒤에 콜론 표시 후 상한 타입 작성
   `fun <T: Number> List<T>.sum(): T`
  - 예시
  ```Kotlin
  fun <T: Comparable<T>> max(first: T, second: T): T {
    return if (first > second) first else second
  }
  ```
    - 두 파라미터 사이 더 큰 값 찾는 제네릭 함수
    - T의 상한 타입은 Comparable<T>
  - 드물지만 타입 파라미터에 대해 둘 이상의 제약 가해야 하는 경우도 있음
    ```Kotlin
    fun <T> ensureTrailingPeriod(seq: T)
        where T: CharSequence, T: Appendable { //타입 파라미터 제약 목록
            if(!seq.endWith(',')){
                seq.append('.')
            }
        }
    ```
  - 제네릭 클래스나 함수를 정의 후 인스턴스화할 때 널이 될 수 있는 타입 포함하는 어떤 타입으로 지정해도 타입 파라미터 치환 가능 
  - 아무런 상한 정하지 않은 타입 파라미터는 Any?를 상한으로 정한 파라미터와 같음
  - 항상 널이 될 수 없는 타입만 타입 인자로 받게 제약 가능
    - Any를 상한으로 사용 => `<T: Any>`

### 실행 시점의 제네릭
- JVM의 제네릭스는 보통 타입 소거를 사용해 구현됨
  - 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지 않음
- 자바와 마찬가지로 코틀린 제네릭 타입 인자 정보는 런타임에 지워짐
  - List<String> 객체 만들고 그 안에 문자열 여럿 넣더라도 실행 시점에는 그 객체를 오직 List로만 볼 수 있음
- 타입 소거의 한계
  - 타입 인자를 따로 저장하지 않기 때문에 실행 시점에 타입 인자 검사 X
    - `is` 검사에서 타입 인자로 지정한 타입을 검사할 수 없음
- **스타 프로젝션**
  - `if (value is List<*>) { ... }`
  - 타입 파라미터가 2개 이상 이라면 모든 타입 파라미터에 *를 포함시켜야 함
  - 인자를 알 수 없는 제네릭 타입을 표현할 때 사용 (자바의 List<?>와 비슷)
  - List임은 알 수 있지만 원소 타입은 알 수 없음
- as, as? 캐스팅에 제네릭 타입 사용 가능
  - ⚠️ 기저 클래스는 같지만 타입 인자 다른 타입으로 캐스팅에 성공하는 점 주의
  - 잘못된 타입의 원소가 들어있는 리스트 전달하면 실행 시점에 `ClassCastException` 발생

### 실체화한 타입 파라미터를 사용한 함수 선언
- 코틀린 제네릭 타입의 타입 인자 정보는 실행 시점에 지워짐
- 이런 제약을 피할 수 있는 경우: **인라인 함수!**
  - 인라인 함수의 타입 파라미터는 실체화되므로 실행 시점에 인라인 함수의 타입 인자를 알 수 있음

```Kotlin
inline fun <reified T> isA(value: Any) = value is T
```
- 인라인 함수로 만들고 파라미터를 `reified`로 지정하면 value의 타입이 T의 인스턴스인지 실행 시점에 검사 가능
- 실체화한 타입 파라미터 사용하는 예시
  - 표준 라이브러리 함수 `filterIsInstance`
  - 인자로 받은 컬렉션 원소 중 타입 인자로 지정한 클래스의 인스턴스만 모아서 만든 리스트 반환
  ```Kotlin
  val items = listOf("one", 2, "three")
  prinln(items.filterIsInstance<String>()) //[one, three]

  inline fun <reified T> Iterable<*>.filterIsInstance(): List<T> {
    val destination = mutableListOf<T>()
    for (element in this) { 
        if(element is T) { //각 원소가 타입 인자로 지정한 클래스의 인스턴스인지 확인
            destination.add(element)
        }
    }
    return destination
  }
  ```
  - 타입 인자로 String을 지정함으로써 문자열만 필요하다는 사실을 기술함
- 인라인 함수에서만 실체화한 타입 인자 쓸 수 있는 이유
  - 컴파일러는 인라인 함수의 본문을 구현한 바이트코드를 함수가 호출되는 모든 지점에 삽입
  - 실체화한 타입 인자를 사용해 인라인 함수를 호출하는 각 부분의 정확한 타입 인자 알 수 있음
  - 타입 파라미터가 아니라 구체적인 타입 사용하므로 만들어진 바이트코드는 실행 시점에 타입 소거의 영향 X
- 인라인 함수에는 실체화한 타입 파라미터가 여러 개이거나 실체화하지 않은 타입 파라미터가 함꼐 있을 수 있음
- java.lang.Class 타입 인자를 파라미터로 받는 API에 대한 코틀린 adapter 구축하는 경우 실체화된 타입 파라미터 자주 사용 
  - ServiceLoader: 어떤 추상 클래스나 인터페이스 표현하는 java.lang.Class 받아서 클래스나 인터페이스 구현한 인스턴스 반환
  ```Kotlin
  val serviceImpl = ServiceLoader.load(Service::class.java)

  //구체화한 타입 파라미터 사용
  val serviceImpl = loadService<Service>()

  //loadService 함수 정의
  inline fun <reified T> loadService() {
    return ServiceLoader.load(T::class.java)
  } 
  ```
- 실체화한 타입 파라미터 제약
  - 할 수 없는 행동
    - 타입 파라미터 클래스의 인스턴스 생성
    - 타입 파라미터 클래스의 동반 객체 메서드 호출
    - 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
    - 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 reified로 지정
  - 실체화한 타입 파라미터를 인라인 함수에만 사용할 수 있으므로 실체화한 타입 파라미터를 사용하는 함수는 자신에게 전달되는 모든 람다와 함께 인라이닝됨
    - 람다 내부에서 타입 파라미터 사용하는 방식에 따라서 람다를 인라이닝할 수 없는 경우 생기기도 하고 성능 문제로 람다 인라이닝하고 싶지 않을 수 있음
    - 그런 경우 `noinline`을 함수 타입 파라미터에 붙여 인라이닝 금지

### 변성 variance
- `List<String>`, `List<Any>` 와 같이 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지를 설명하는 개념
- Any 타입 값을 파라미터로 받는 함수에 String 값을 넘겨도 안전함
- 하지만 Any와 String이 List 인터페이스의 타입 인자로 들어가는 경우 위험할 수 있음
  - 리스트의 원소 추가하거나 변경한다면 타입 불일치가 생길 수 있음
- 코틀린에서는 리스트의 변경 가능성에 따라 적절한 인터페이스 선택하면 안전하지 못한 함수 호출 막을 수 있음
  
- **타입 vs 클래스**
  - 제네릭 클래스가 아닌 클래스에서는 클래스 이름을 타입으로 사용 가능
  - 클래스 이름을 널이 될 수 있는 타입에도 쓸 수 있음 => 코틀린 클래스는 적어도 둘 이상의 타입 구성 가능
  - 제네릭 클래스에서는 올바른 타입 얻으려면 타입 파라미터를 구체적인 타입 인자로 바꿔줘야 함
    - List는 타입은 아니지만 클래스임
    - 하지만 타입 인자 치환하면 무수히 많은 타입 만들 수 있음 `List<Int>`
  
- **하위 타입**
  - 어떤 타입 A의 값이 필요한 곳에 어떤 타입 B 값 넣어도 아무 문제 없다면 *타입 B는 타입 A의 하위타입*
  - 컴파일러는 변수 대입이나 함수 인자 전달 시 하위 타입 검사를 매번 수행함
  - 간단한 경우 하위 타입은 하위 클래스와 근본적으로 같음
    - 어떤 인터페이스 구현하는 클래스 타입은 그 인터페이스 타입의 하위 타입
    - 하지만 같지 않은 경우도 있음 => **널이 될 수 있는 타입**
    - 널이 될 수 없는 타입은 널이 될 수 있는 타입의 하위 타입이지만 모두 같은 클래스임
    ```Kotlin
    val s: String = "abc"
    val t: String? = s
    ```
    - 제네릭 타입을 인스턴스화할 때 타입 인자로 서로 다른 타입이 들어가면 인스턴스 타입 사이의 하위 타입 관계 성립하지 않으면 그 제네릭 타입을 '무공변'이라고 함
      - A, B가 서로 다르기만 하면 `MutableList<A>` 가 항상 `MutableList<B>`의 하위 타입은 아님
    - 자바에서는 모든 클래스가 무공변
    - 코틀린에서 A가 B의 하위 타입이면 `List<A>`는 `List<B>`의 하위 타입임 => **공변적**
- A가 B의 하위 타입일 때 `Producer<A>`가 `Producer<B>`의 하위 타입이면 Producer는 공변적임
- 하위 타입 관계가 유지되는 것!
- 코틀린에서 제네릭 클래스가 타입 파라미터에 대해 공변적임을 표시하려면 타입 파라미터 앞 `out`을 넣어야 함
```Kotlin
interface Producer<out T> {
    fun produce(): T
}
```
- 공변적으로 만들면 함수 정의에 사용한 파라미터 타입과 타입 인자의 타입이 정확히 일치하지 않아도 그 클래스의 인스턴스를 함수 인자나 반환 값으로 사용 가능
- 공변적으로 만들면 안전하지 못해서 공변적으로 만들 수 없는 클래스도 존재
- 타입 파라미터를 공변적으로 지정하면 클래스 내부에서 파라미터 사용하는 방법을 제한해야 함
- 타입 안정성 보장 위해 공변적 파라미터는 항상 아웃 위치에만 있어야 함 => T 타입 값 생산할 수 있지만 T 타입 값 소비 X
- 클래스 멤버 선언할 때 타입 파라미터 사용할 수 있는 지점 => 인 OR 아웃
  - T라는 타입 파라미터 선언하고 T를 사용하는 함수가 멤버로 있는 클래스
  - T가 함수의 반환 타입에 쓰인다면 T는 아웃 위치, T 타입 값 생산
  - T가 함수의 파라미터 타입에 쓰인다면 인 위치, T 타입 값 소비
  - 타입 파라미터 앞에 `out` 붙이면 클래스 안에서 T 사용하는 메서드가 아웃 위치에서만 T 사용하게 허용함 => T로 인해 생기는 하위 타입 관계의 타입 안전성 보장함
  
- out 키워드 의미
  - 공변성: 하위 타입 관계가 유지된다
  - 사용 제한: T를 아웃 위치에서만 사용할 수 있다

- MutableList<T> 를 타입 파라미터 T에 대해 공변적인 클래스로 선언할 수 없음 => T가 인에 쓰이기 때문
- 생성자 파라미터는 인/아웃 어느 쪽도 아니기 때문에 타입 파라미터가 out이어도 여전히 생성자 파라미터 선언에 사용 가능
- 변성의 역할
  - 코드에서 위험할 여지 있는 메서드를 호출할 수 없게 만듦
  - 제네릭 타입의 인스턴스 역할을 하는 클래스 인스턴스를 잘못 사용하는 일이 없게 방지
- 이런 규칙은 외부에서 볼 수 있는 (public, protected, internal) 클래스에만 적용 가능 
  - private 메서드의 파라미터는 인, 아웃 모두 아님
  
### 반공변성
- 반공변 클래스의 하위 타입 관계는 공변 클래스 경우와 반대
- Comparator 인터페이스
  - compare 메서드는 주어진 두 객체를 비교하는데 T 타입 값을 소비하기만 함 => T가 인 위치에서만 쓰임
  - 따라서 T 앞에 `in` 키워드 붙임
- 어떤 타입의 객체를 Comparator로 비교해야 한다면 그 타입이나 타입의 조상 타입을 비교할 수 있는 Comparator를 사용할 수 있음
  - Comparator<Any> 가 Comparator<String> 의 하위 타입임
  - 여기서 Any 는 Strng의 상위 타입 
  - 따라서 서로 다른 타입 인자에 대해 Comparato의 하위 타입 관계는 타입 인자의 하위 타입 관계와 정반대
- 타입 B가 타입 A의 하위 타입인 경우 `Consumer<A>` 가 `Consumer<B>` 의 하위 타입인 관계 성립하면 `Consumer<T>` 는 타입 인자 T에 대해 반공변

- in 키워드 
  - 키워드 붙은 타입이 클래스의 메서드 안으로 전달돼 메서드에 의해 소비됨
  - 타입 파라미터의 사용 제한 O => 인 위치에서만 

- 클래스/인터페이스가 어떤 타입 파라미터에 대해 공변적이면서 다른타입 파라미터에 대해서는 반공변적일 수 있음
  ```Kotlin
  interface Function1<in P, out R> {
    operator fun invoke(p: P): R
  }
  ```
- 자바에서는 클래스 사용하는 위치에서 와일드카드 사용해 그때그때 변성 지정 필요

### 사용 지점 변성
- 선언 지점 변성: 클래스 선언하면서 변성 지정 시 클래스 사용하는 모든 장소에 변성 지정자가 영향을 끼침
- 사용 지점 변성: 타입 파라미터가 있는 타입을 사용할 때마다 해당 타입 파라미터를 하위 타입이나 상위 타입 중 어떤 타입으로 대치할 수 있는지 명시
  - 자바
  - 코틀린도 지원 O
  - 클래스 안에서 어떤 타입 파라미터가 공변적인지 반공변적인지 선언할 수 없는 경우 특정 타입 파라미터가 나타나는 지점에 변성 정할 수 있음
  - 함수 구현이 아웃 또는 인 위치에 있는 타입 파라미터를 사용하는 메서드만 호출하면 정보 바탕으로 함수 정의 시 타입 파라미터에 변성 변경자 추가 가능
- 타입 선언에서 타입 파라미터 사용하는 위치 어디든 변성 변경자 붙일 수 있음
  - 이때 타입 프로젝션 일어남 => 제약을 가한 타입으로 만듦

### 스타 프로젝션
- 제네릭 타입 인자 정보 없음을 표현하기 위해 사용
- `MutableList<*>` 는 `MutableList<Any?>` 와 같지 않음
  - `MutableList<Any?>` 는 모든 타입의 원소 담을 수 있음
  - `MutableList<*>` 는 어떤 정해진 구체적인 타입의 원소만을 담는 리스트지만 원소의 타입을정확하게 모른다는 사실을 표현
  - 아무거나 담을 수 없음
  - 하지만 `MutableList<*>` 에서 원소를 얻을 수 있고 그 원소 타입이 Any? 의 하위 타입은 분명함
  - `MutableList<*>` 은 `MutableList<out Any?>` 처럼 동작함
    - 어떤 리스트의 원소 타입 모르더라도 리스트에서 안전하게 Any? 타입 원소를 꺼내올 수는 있지만 타입 모르는 리스트에 원소 마음대로 넣을 수 없음
- 타입 파라미터를 시그니처에서 전혀 언급하지 않거나 데이터 읽기는 하지만 타입에는 관심 없는 경우도 사용 가능
- 스타 프로젝션 사용할 때는 값 만들어내는 메서드만 호출할 수 있고 값의 타입에는 신경 쓰면 X