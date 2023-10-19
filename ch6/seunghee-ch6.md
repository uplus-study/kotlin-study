# 6장: 코틀린 타입 시스템

## Nallable

: NullPointerException 오류를 피할 수 있게 돕기 위한 코틀린 타입 시스템의 특성으로 null에 대한 접근 방법 문제를 실행 시점에서 컴파일 시점으로 옮기기 위한 것
<br> <br>

인자로 null을 받기 위해서는 타입 이름 뒤에 ?를 사용

## 안전한 호출 연산자: ?.

코틀린이 제공하는 가장 유용한 도구 중 하나가 안전한 호출 연산자인 ?. 입니다. ?.은 null 검사와 메소드 호출을 한 번의 연산으로 수행

## 엘비스 연산자: ?:

코틀린은 null 대신 사용할 디폴트 값을 지정할 때 편리하게 사용할 수 있는 연산자를 제공
이 연산자는 이항 연산자로 좌항을 계산한 값이 널인지 검사해서 좌항 값이 널이 아니면 좌항 값을 결과로 하고, 좌항 값이 널이면 우항 값을 결과로 리턴

엘비스 연산자를 객체가 널인 경우 널을 반환하는 안전한 호출 연산자와 함께 사용해서 객체가 널인 경우에 대비한 값을 지정하는 경우도 많습니다.

## 안전한 캐스트: as?

대상 값을 as로 지정한 타입으로 바꿀 수 없으면 ClassCaseException이 발생
as? 연산자는 어떤 값을 지정한 타입으로 캐스트합니다. as?는 값을 대상 타입으로 변환할 수 없으면 null을 반환

## 널 아님 단언: !!

느낌표를 이중으로 사용하면 어떤 값이든 널이 될 수 없는 타입으로 강제로 바꿀 수 있고 실제 널에 대해 !!를 적용하면 NPE가 발생함


## let 함수

let 함수는 안전한 호출 연산자와 함께 사용하면 원하는 식을 평가해서 결과가 널인지 검사한 다음에 그 결과를 변수에 넣는 작업을 간단한 식을 사용해 한꺼번에 처리할 수 있음


## 나중에 초기화할 프로퍼티

코틀린에서는 일반적으로 생성자에서 모든 프로퍼티를 초기화해야 하는데 lateinit 변경자를 붙이면 프로퍼티를 나중에 초괴화 할 수 있음
나중에 초기화 하려는 프로퍼티는 항상 var 이어야 함

## 널이 될 수 있는 원시 타입: Int?, Boolean? 등

코틀린에서 널이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일 됨

kotlin
data class Person(val name: String, val age: Int? = null) {
    fun isOlderThan(other: Person): Boolean? {
        if (age == null || other.age == null) return null
        return age > other.age
    }
}


## 숫자 변환

코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않음

kotlin
val i = 1
val l: Long = i  // 컴파일 오류


kotlin
val i = 1
val l: Long = i.toLong();

코틀린은 Boolean을 제외한 모든 원시 타입에 대한 변환 함수를 제공함(ex: toByte(), toShort(), toChar())

## Any, Any?: 최상위 타입
코틀린에서는 Any 타입이 모든 널이 될 수 없는 타입의 조상 타입으로 Any가 Int 등의 원시 타입을 포함한 모든 타입의 조상임
Any가 널이 될 수 없는 타임임으로 널을 포함하려면 Any? 타입을 사용해야 하며 코틀린 함수가 자바 바이트코드의 Object로 컴파일 됨.
toString, equals, hashCode 라는 세 메소드는 Any에 정의된 메소드를 상속해서 모든 코틀린 클래스에서 사용할 수 있지만 wait, notify는 Any에서 사용할 수 없음

## Init 타입: 코틀린의 void

코틀린 Unit 타입은 자바 void와 같은 기능을 하지만 Unit은 모든 기능을 갖는 일반적인 타입이며, void와 달리 Unit을 타입 인자로 쓸 수 있음
함수형 프로그래밍에서 전통적으로 Unit은 '단 하나의 인스턴스만 갖는 타입'을 의미해 왔고 바로 그 유일한 인스턴스의 유무가 자바 void와 코틀린 Unit을 구분하는 가장 큰 차이입임

## 읽기 전용과 변경 가능한 컬렉션

읽기 전용의 컬렉션 인터페이스는 kotlin.collections.Collection이고, 컬렉션의 데이터를 수정하려면 kotlin.collections.MutableCollection 인터페이스를 사용해야 함.
컬렉션의 읽기 전용 인터페이스와 변경 가능 인터페이스를 구별한 이유는 프로그램에서 데이터에 어떤 일이 벌어지는지를 더 쉽게 이해하기 위함임

kotlin
fun <T> copyElements(source: Collection<T>, target: MutableCollection<T>) {
    for (item in source) {
        target.add(item)
    }
}



## 코틀린 컬렉션과 자바

## 객체의 배열과 원시 타입의 배열

코틀린에서 배열을 만드는 방법은 다양함.

- arrayOf 함수에 원소를 넘기면 배열 생성 가능
- arrayOfNulls 함수에 정수 값을 인자로 넘기면 모든 원소가 null이고 인자로 넘긴 값과 크기가 같은 배열을 만들 수 있음.
- Array 생성자는 배열 크기와 람다를 인자로 받아서 람다를 호출해서 각 배열 원소를 초기화 해줌.
