# 람다로 프로그래밍

### 람다 식, 람다
> 다른 함수에 넘길 수 있는 작은 코드 조각
- 쉽게 공통 코드 구조를 라이브러리 함수로 뽑아낼 수 있음
- 예전 자바에서는 무명 내부 클래스를 통해서 일련의 동작을 변수에 저장하거나 다른 함수에 넘기는 작업을 함
- 함수형 프로그래밍에서는 함수를 값처럼 다룸
-> 함수를 직접 다른 함수에 전달 가능
-> 람다 식으로 코드 더욱 간결해짐
- 자바8에서 람다 적용 가능, 코틀린도 가능 

### 람다 식 문법

```Kotlin
{ x: Int, y: Int -> x + y }

val sum = { x: Int, y: Int -> x + y } //변수에 저장

{ println(42) } {} //람다 식 직접 호출, 출력: 42
```

- 람다 정의 후 변수에 저장 가능
- 코틀린에서 람다 식은 항상 중괄호에 선언
- 화살표로 인자 목록과 본문 구분
- z코드 일부분을 블록으로 둘러싸 실행할 때 `run` 사용
    ```Kotlin
    run { println(42) } //람다 본문에 있는 코드 실행
    ```

예제
```Kotlin
people.maxBy({ p: Person -> p.age })
```
- 맨 뒤 인자가 람다식이라면 괄호 빼낼 수 있음
- 람다가 어떤 함수의 유일한 인자면서 괄호 뒤 람다를 썼다면 빈 괄호 없앨 수 있음
    ```Kotlin
    people.maxBy() { p: Person -> p.age }
    people.maxBy { p: Person -> p.age }
    ```
- 파라미터 타입 제거 가능 <- 컴파일러가 타입 추론
    > 람다를 변수에 저장할 때는 타입 추론 X -> 타입 명시 필요
    ```Kotlin
    people.maxBy { p -> p.age }
    ```
- `it` 파라미터 하나뿐이고 타입을 컴파일러가 추론할 수 있는 경우 사용 
    ```Kotlin
    people.maxBy { it.age }
    ```

### 현재 영역에 있는 변수에 접근
- 자바 메서드 안에 무명 내부 클래스 정의할 때 메서드의 로컬 변수를 무명 내부 클래스에서 사용 O
- 람다도 마찬가지로 함수의 파라미터, 람다 정의 앞에 선언된 로컬 변수 사용 O
- 자바와 다르게 코틀린 람다 안에서는 **파이널 변수가 아닌 변수에 접근 가능, 람다 안에서 바깥 변수 변경 가능**
- 포획 변수: 람다 안에서 사용하는 외부 변수
  - 함수가 자신의 로컬 변수 포획한 람다 반환하거나 다른 변수에 저장한다면 로컬 변수/함수의 생명주기 달라짐 
  - 파이널 변수 포획한 경우, 람다 코드를 변수 값과 함께 저장
  - 파이널 X 변수 포획한 경우, 특별한 래퍼로 감싸 나중에 변경하거나 읽을 수 있게 한 다음 래퍼에 대한 참조를 람다 코드와 함께 저장
- 변경 가능한 변수 포획하기
    ```Kotlin
    var counter = 0
    val inc = { counter++ }
    ```
- ⚠️ 주의할 점 !
  - 람다를 이벤트 핸들러나 비동기적으로 실행되는 코드로 활용하는 경우, 함수 호출이 끝난 다음에 로컬 변수가 변경될 수도 있음

### 멤버 참조
- 넘기려는 코드 블록이 이미 함수로 선언된 경우,
  - 함수를 호출하는 람다를 만들면 됨 -> 하지만 중복임! 😫
  - 함수를 직접 넘기기 위해 값으로 바꿈, 이중 콜론 사용 => **멤버 참조**
    ```Kotlin
    val getAge = Person::age

    //val getAage = { person: Person -> person.age } 와 같은 표현
    ```
- 멤버 참조는 프로퍼티나 메서드를 단 하나만 호출하는 함수 값을 만들어줌
- 멤버 참조 뒤에는 괄호 X
- 최상위 선언된 함수나 프로퍼티 참조 가능
    ```Kotlin
    fun salute() = println("Salute!")
    
    run(::salute) // 출력: Salute!
    ```
- 인자 여럿인 다른 함수한테 작업 위임하는 경우 람다 정의하지 않고 직접 위임 함수에 대한 참조 제공하는 것이 편리
- **생성자 참조**를 사용하여 클래스 생성 작업 연기하거나 저장해둘 수 있음
  - :: 뒤에 클래스 이름 넣어 생성자 참조 만듦
- 확장 함수도 멤버 함수와 같은 방식으로 참조 가능

> 바운드 멤버 참조
>
> 코틀린 1.0에서는 클래스 메서드나 프로퍼티에 대한 참조 얻은 후 참조 호출할 때 인스턴스 객체를 제공해야 했음
> 1.1부터 바운드 멤버를 지원하여 멤버 참조를 생성할 때 클래스 인스턴스를 함께 저장한 다음 그 인스턴스에 대해 멤버를 호출해줌 -> 호출 시 수신 대상 객체 별도로 지정할 필요 X
> ```Kotlin
> val p = Person("Peter", 30)
> val personAgeFunction = Person::age
> println(personAgeFunction(p))
>
> //1.1 이후
> val dmitrysAgeFunction = p::age
> println(dmitrysAgeFunction())
> ```
>

### 컬렉션 함수형 API
- filter 함수: 컬렉션을 이터레이션해서 주어진 람다에 각 원소를 넘겨서 람다가 true를 반환하는 원소만 모음
  ```Kotlin
  val list = listOf(1, 2, 3, 4)
  println(list.filter { it % 2 == 0}) // [2, 4]
  ```
    - 컬렉션에서 원치 않는 원소를 제거함
  
- map 함수: 주어진 람다를 컬렉션의 각 원소에 적용한 결과를 모아 새 컬렉션을 만듦
  ```Kotlin
  val list = listOf(1, 2, 3, 4)
  println(list.map { it * it }) //[1, 4, 9, 16]
  ```

- all, any 함수: 컬렉션의 원소가 어떤 조건을 만족하는지 판단
  
- count 함수: 조건 만족하는 원소 개수 반화
  
- find 함수: 조건 만족하는 첫 번째 원소 반환
  
> count와 size
>
> count는 조건 만족하는 원소의 **개수**만 추척, 만족하는 원소를 따로 저장하지 않음
> size는 컬렉션을 대상으로 함
>

- groupBy 함수: 리스트를 여러 그룹으로 이루어진 맵으로 변경
  - 컬렉션 원소 구분하는 특성 -> 키
  - 키 값에 따른 각 그룹 -> 값

- flatMap 함수
    ```Kotlin
    class Book(val title: String, val authors: List<String>)

    books.flatMap { it.authors }.toSet() //books 컬렉션에 있는 책을 쓴 모든 저자의 집합
    ```
    - 먼저 인자로 주어진 람다를 컬렉션의 모든 객체에 적용(map)
    - 람다 적용한 결과 얻어지는 여러 리스트를 한 리스트로 모음(flatten)
    - flatten: 특별히 반환해야 할 내용 없이 리스트의 리스트를 평평하게 펼칠 때 사용

### 지연 계산
- 앞서 설명한 함수는 결과 컬렉션을 즉시(eagerly) 생성
  - 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담음
- `시퀀스(sequence)`를 사용하면 중간 임시 컬렉션 사용하지 않고도 컬렉션 연산 가능 -> 성능 👍
```Kotlin
people.map(Person::name).filter{ it.startsWith('A') }

//sequence 사용
people.asSequence() //시퀀스로 변환
    .map(Person::name)
    .filter{ it.startsWith('A') }
    .toList() //결과 시퀀스를 리스트로 변환
```

- Sequence 인터페이스
  - 한 번에 하나씩 열거될 수 있는 원소의 시퀀스를 표현
  - `iterator` 메서드 -> 시퀀스로부터 원소 값 얻음
  - 시퀀스 원소는 실제로 그 원소가 필요할 때 계산됨 
  - `asSequence` 확장 함수를 호출하여 컬렉션을 시퀀스로 바꿈

### 시퀀스 연산: 중간 연산, 최종 연산
- 중간 연산 (intermediate)
  - 다른 시퀀스를 반환함
  - 그 시퀀스는 최초 시퀀스의 원소를 변환하는 방법을 앎 
  - 항상 지연 계산됨
- 최종 연산 (terminal)
  - 결과 반환
  - 결과는 최초 컬렉션에 대해 변환 적용한 시퀀스로부터 일련의 계산을 수행해 얻을 수 있는 컬렉션이나 원소, 숫자 또는 객체

```Kotlin
//이 코드 실행하면 출력 X -> map, filter 연산이 늦춰짐
listOf(1, 2, 3, 4).asSequence()
    .map{ print("map($it) "); it * it }
    .filter { print("filter($it) "); it % 2 == 0 }

//최종 연산 호출될 때 연기되었던 모든 연산 적용됨
listOf(1, 2, 3, 4).asSequence()
    .map{ print("map($it) "); it * it }
    .filter { print("filter($it) "); it % 2 == 0 }
    .toList()

> map(1) filter(1) map(2) filter(4) map(3) filter(9) map(4) filter(16)
```

> **연산 수행 순서 주의!**
>
> 직접 연산) map 을 수행해서 새 시퀀스 얻고 filter 수행할 것
> 
> 시퀀스) 모든 연산은 각 원소에 대해 순차 처리됨 -> 첫 번째 원소가 처리된 후 두 번째 원소가 처리됨
>
> 따라서 원소 연산 차례로 적용하다가 결과가 얻어지면 그 이후 원소에 대해서는 변환 X
> ![img](https://github.com/uplus-study/kotlin-study/assets/126530537/4fead322-f57f-43c5-9090-138417dbe162)


- `generateSequence` 함수로도 시퀀스 만들 수 있음

```Kotlin
val naturalNumbers = generateSequence(0) { it + 1 } //시퀀스
val numbersTo100 = naturalNumberes.takeWhile { it <= 100 } //시퀀스
println(numbersTo100.sum()) 

> 5050
```

### 자바 함수형 인터페이스 활용
- 코틀린에서는 무명 클래스 인스턴스 대신 람다를 넘길 수 있음
```Kotlin
//무명 내부 클래스
button.setOnClickListener(new OnClickListener() {
  @Override
  public void onClick(View v) {
    ...
  }
})

//코틀린 람다
button.setOnClickListener { View -> ... }
```
- `onClickListener` 에 추상 메서드가 단 하나만 있기 때문에 가능 
    => 함수형 인터페이스, SAM(Single Abstract Method) 인터페이스
- 코틀린에서는 함수형 인터페이스를 인자로 취하는 자바 메서드를 호출할 때 람다 넘길 수 있음

- 예제
  ```Kotlin
  //java
  void postponeComputation(int delay, Runnable computation);

  //kotlin
  postponeComputation(1000) { println(42) }
  ```
  - 컴파일러가 자동으로 람다를 Runnable 인스턴스로 변환해줌
    - Runnable을 구현한 무명 클래스 인스턴스
    - 람다 본문을 무명 클래스의 유일한 추상 메서드 본문으로 사용
  - 람다와 무명 객체 사이 차이 존재
    - 객체를 명시적으로 선언하는 경우 메서드를 호출할 때마다 새로운 객체 생성
    - 람다에 대응하는 무명 객체를 메서드 호출할 때마다 반복 사용
    - 람다가 주변 영역의 변수 포획한다면 매 호출마다 같은 인스턴스 사용 X -> 매번 주변 영역 변수 포획한 새로운 인스턴스 생성해줌
  - 컬렉션을 확장한 메서드에 람다를 넘기는 경우 다름
    - 코틀린 inline으로 표시된 코틀린 함수에게 람다를 넘기면 아무런 무명 클래스 만들어지지 않음
  
- SAM 생성자
  - 람다를 함수형 인터페이스의 인스턴스로 변환할 수 있게 컴파일러가 자동으로 생성한 함수
  - 컴파일러가 자동으로 람다를 함수형 인터페이스 무명 클래스로 바꾸지 못하는 경우 SAM 생성자 사용 O
    - 함수형 인터페이스의 인스턴스 반환해야 하는 경우 람다를 SAM 생성자로 감싸야 함
    ```Kotlin
    fun createAllDoneRunnable(): Runnable {
      return Runnable { println("All done!") }
    }

    createAllDoneRunnable().run()
    > All Done!
    ```
  - SAM 생성자 이름은 사용하려는 함수형 인터페이스 이름과 같음 
  - 함수형 인터페이스의 유일한 추상 메서드 본문에 사용할 람다만을 인자로 받아 함수형 인터페이스 구현하는 클래스의 인스턴스 반환
  - 람다로 생성한 함수형 인터페이스 인스턴스를 변수에 저장하는 경우에도 사용

> **람다와 리스너 등록/해제하기**
>
> 람다에는 `this` 가 없음. 따라서 람다를 변환할 무명 클래스 인스턴스를 참조할 방법이 없음
>
> 이벤트 리스너가 이벤트 처리하다가 자신의 리스너 등록을 해제해야 한다면 람다를 사용할 수 없고 대신 무명 객체를 사용!

  - 함수형 인터페이스 요구하는 메서드 호출할 때 대부분 SAM 변환을 컴파일러가 자동으로 수행할 수 있지만, 가끔 오버로드한 메서드 중 어떤 타입의 메서드를 선택해 람다를 변환해 넘겨줘야 할지 모호할 때는 명시적으로 SAM 생성자 적용

### 수신 객체 지정 람다
> 수신 객체를 명시하지 않고 람다 본문 안에서 다른 객체의 메서드를 호출할 수 있게 함

- with 함수
  - 어떤 객체 이름 반복하지 않고 객체에 대해 다양한 연산 수행 가능
```Kotlin
fun alphabet(): String {
  val result = StringBuilder()
  for (letter in 'A'..'Z') {
    result.append(letter)
  }
  result.append("\nNow I know the alphabet!")
  return result.toString()
}

//with 사용
fun alphabet(): String {
  val stringBuilder = StringBuilder()
  return with(stringBuilder) {
    for (letter in 'A'..'Z') {
      this.append(letter) //this 명시해서 앞에서 지정한 수신 객체 메서드 호출
    }
    append("\nNow I know the alphabet!") //this 생략
    this.toString() //람다에서 값 반환
  }
}

//리팩토링
fun alphabet() = with(StringBuilder() {
  for (letter in 'A'..'Z') {
    append(letter)
  }
  append("\nNow I know the alphabet!")
  toString()
})
```
  - 실제로는 파라미터 2개 있는 함수
    - 첫 번째 파라미터: stringBuilder, 두번째 파라미터: 람다
  - 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만듦

> 수신 객체 지정 람다와 확장 함수 비교
>
> 확장 함수 안에서 this는 함수가 확장하는 타입의 인스턴스를 가리킴
>
> 일반 함수 <-> 일반 람다, 확장 함수 <-> 수신 객체 지정 람다

> 메서드 이름 충돌시 `this@OuterClass.toString()` 과 같이 사용

- apply 함수
  - with 함수와 같으나 항상 자신에게 전달된 객체(수신 객체)를 반환함
  ```Kotlin
  fun alphabet() = StringBuiler().apply {
    for (letter in 'A'..'Z') {
      append(letter)
    }
    append("\nNow I know the alphabet!")
  }.toString()
  ```
  - apply 실행한 결과는 StringBuilder 객체이기 때문에 객체의 `toString`을 호출해서 String 객체를 얻을 수 있음
  - 객체 인스턴스 만들면서 즉시 프로퍼티 중 일부를 초기화하는 경우 유용