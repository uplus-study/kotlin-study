# 코루틴과 Async/Await

### 코루틴이란
> 컴퓨터 프로그램 구성 요소 중 하나로 비선점형 멀티태스킹을 수행하는 일반화한 서브루틴
>
> 실행을 일시 중단하고 재개할 수 있는 여러 진입 지점을 허용한다

- 서브루틴: 여러 명령어 모아 이름 부여해서 반복 호출할 수 있게 정의한 프로그램 구성 요소, 함수, 메서드
  - 서브루틴에 진입하는 방법은 함수 호출하는 방법 뿐이며, 그때마다 활성 레코드가 스택에 할당되면서 서브루틴 내부 로컬 변수 등이 초기화됨
  - 서브루틴 안에서 여러 return문 사용 가능하므로 서브루틴 실행 중단하고 제어 호출한 쪽에서 돌려주는 지점은 여럿 있을 수 있음
  - 서브루틴 반환되고 나면 활성 레코드가 스택에서 사라지므로 실행 중인 모든 상태를 잃어버림 => 항상 같은 결과 반환
  
- 멀티태스킹: 여러 작업을 동시에 수행하는 것처럼 보이거나 실제로 동시에 수행하는 것
- 비선점형: 멀티태스킹 각 작업을 수행하는 참여자들의 실행을 운영체제가 강제로 일시 중단시키고 다른 참여자를 실행하게 만들 수 없음 => 자발적으로 협력해야 함
  
- 따라서 코루틴은 서로 협력해서 실행 주고받으면서 작동하는 여러 서브루틴
  - 예) 제네레이터 - 어떤 함수 A 실행되다가 제네레이터인 코투린 B 호출하면 A가 실행되던 스레드 안에서 코루틴 B의 실행이 시작됨, 실행 진행하다가 A에 양보(yield 명령인 경우가 많음)하고 A는 다시 코루틴 호출했던 다음 부분부터 실행하다가 다시 코루틴 B 호출
  - 이때 B는 이전 yield로 양보했던 지점부터 이어서 실행
- 코루틴 장점: 일반적인 프로그램 로직 기술하듯 코드 작성하고 상대편 코루틴에 데이터 넘겨야 하는 부분에서만 yield 사용하면 됨

### 코틀린의 코루틴 지원
- 코틀린은 특정 코루틴을 언어가 지원하는 형태가 아니라, 코루틴을 구현할 수 있는 기본 도구를 언어가 제공하는 형태
- 기본 기능들은 kotlin.coroutine 패키지 밑에 있음
- 코틀린이 지원하는 기본 기능 활용해 만든 다양한 형태의 코루틴은 kotlinx.coroutines 패키지 밑에 있음

- kotlinx.corountines.core 모듈 내의 코루틴 (코루틴 빌더)
  - kotlinx.corountines.CoroutinScope.launch
    - launch는 코루틴을 잡으로 반환, 만들어진 코루틴은 기본적으로 즉시 실행
    - launch가 반환한 잡의 cancel() 호출해서 실행 중단 가능
    - launch 작동하려면 CoroutineScope 객체가 블록의 this로 지정되어야 하는데 다른 suspend 함수 내부가 아니라면 GlobalScope 사용
    - ⚠️ 메인 함수와 GlobalScope.launch 만들어낸 코루틴이 서로 다른 스레드에서 실행됨
    - GlobalScope는 메인 스레드가 실행 중인 동안만 코루틴 동작을 보장함
  - kotlinx.corountines.CoroutineScope.async
    - launch와 비슷한 일을 하지만 async는 Deffered를 반환함
    - Deffered는 Job을 상속한 클래스
      - 차이는 Job은 아무 타입 파라미터가 없지만 Deffered는 타입 파라미터가 있는 제네릭 타입이라는 점, Deffered 안에는 await() 정의돼 있음
      - 타입 파라미터는 Deffered 코루틴이 계산 하고 돌려주는 값 타입
    - async는 코드 블록을 비동기로 실행 가능, async 반환하는 Deffered의 await 사용해서 코루틴이 결과 값 내놓을 때까지 기다렸다 값 얻을 수 있음

- 코루틴 컨텍스트와 디스패처
  - launch, async 모두 CoroutineScope의 확장 함수
  - CoroutineScope에는 CoroutineContext 타입 필드 하나만 있음
    - launch 등의 확장 함수 내부에서 사용하기 위한 매개체 역할만 담당
  - CoroutineContext는 실제로 코루틴 실행 중인 여러 작업과 디스패처를 저장하는 일종의 맵
  - 코틀린 런타임은 CoroutineContext 사용해서 다음 실행할 작업 선정하고 어떻게 스레드에 배정할지에 대한 방법 결정

- 코루틴 빌더와 일시 중단 함수
  - launch, async, runBlocking 모두 코투린 빌더 => 코루틴 만들어주는 함수
  - kotlinx-coroutines-core가 제공하는 추가 코루틴 빌더
    - `produce`: 정해진 채널로 데이터를 스트림으로 보내는 코루틴 만듦
    - `actor`: 정해진 채널로 메시지 받아 처리하는 액터를 코루틴으로 만듦
  - 일시 중단 함수: delay(), yield(), withContext, withTimeout, withTimeoutOrNull, awaitAll, joinAll

### suspend 키워드와 코틀린의 일시 중단 함수 컴파일 방법
- 일시 중단 함수를 코루틴이나 일시 중단 함수가 아닌 함수에서 호출하는 것은 컴파일 수준에서 금지됨
- 일시 중단 함수 만드는 방법
  - suspend를 fun 키워드 앞에 넣음!
- 일시 중단 함수 안에서 yield() 하는 경우,
  - 코루틴 진입할 때와 나갈 때 코루틴이 실행 중이던 상태 저장하고 복구하는 등의 작업 필요
  - 현재 실행 중인 위치 저장하고 다시 코루틴 재개될 때 해당 위치부터 실행 재개
  - 다음에 어떤 코루틴 실행할지 결정 => 코루틴 컨텍스트에 있는 디스패처에 의해 수행
- 일시 중단 함수 컴파일하는 컴파일러는 앞 두가지 작업 할 수 있는 코드 생성해야 함
- 이때 코틀린은 CPS(continuation passing style) 변환과 상태 기계 활용해 코드 생성함
  - CPS 변환은 프로그램 실행 중 특정 시점 이후 진행해야 하는 내용을 별도의 함수로 뽑고 함수에게 현재 시점까지 실행한 결과 넘겨서 처리하게 만드는 소스코드 변환 기술

### 코루틴 빌더 만들기
- 일반적으로 직접 만들 필요는 없음
- 제네레이터 빌더 구현
    ```Kotlin
    interface Generator<out R, in T> {
        fun next(param: T): R?
    }
    ```
  - generator 함수가 반환해야 하는 타입 -> Generator<R, T>
  - 이 제네레이터는 next(T) 호출해 R 타입의 값을 돌려받아 처리할 수 있게 해주는 객체
  - generator 받는 블록 안에서는 yield() 쓸 수 있어야 함
    - R 받아서 Unit 돌려주는 함수
    - yield() 들어있는 어떤 클래스를 this로 갖고 있어야 yield() 호출 가능
  ```Kotlin
  @RestrictsSuspension //suspend 붙은 함수에만 클래스를 수신 객체로 지정할 수 있게 함
  interface GeneratorBuilder<in T, R> {
    suspend fun yield(value: T): R
    suspend fun yieldAll(generator: Generator<T, R>, param: R)
  }
  ```
  - generator의 타입은 `fun <T,R> generate(block: suspend GeneratorBuilder<T, R>.(R) -> Unit): Generator<T, R>`
  - 이 함수 안 코루틴 만들고 코루틴이 맨 처음 호출했을 때와 다음 단계 진행할 때 실행할 코드 지정 필요
  - 이제 GeneratorBuilder와 Generator 구현 제공해야 함
    - GeneratorBuilder는 실제 다른 객체 만들지 않지만 yield 구현해야 함
    - Generator는 next를 제공하는데 next는 yield가 저장해둔 다음 단계 블록을 현재 스레드상에서 실행하고 LASTVALUE 값 반환
  - GeneratorBuilder와 Generator 구현은 서로 밀접히 물려 있고 내부 정보를 공유해야 하므로 두 인터페이스를 한 클래스가 구현하게 만듦
  - startCorouti은 코투린 끝난 후 처리해야 하는 Continuation을 두 번째 인자로 받아서 프로그램 나머지 부분을 실행하도록 함 => `Continuation` 구현 