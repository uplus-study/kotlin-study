# E 코루틴과 Async/Await

## E.1 코루틴이란?

코루틴: 비선점형 멀티태스킹을 수행하는 일반화한 서브루틴

- 실행을 일시 중단하고 재개할 수 있는 여러 진입 지점을 허용
- 서브루틴: 함수/메서드
    - 서브루틴에 진입(진입 방법은 한 가지)할 때마다 활성 레코드가 스택에 할당 → 서브루틴 내부의 로컬 변수 등이 초기화
    - 반환 지점은 여럿, 반환되고 나면 활성 레코드가 스택에서 사라진다
- 멀티태스킹: 여러 작업을 동시에 수행하는 것처럼 보이거나, 실제로 동시에 수행
- 비선점형: 멀티태스킹의 각 작업을 수행하는 참여자들의 실행을 운영체제가 강제로 일시 중단시키고 다른 참여자를 실행하게 할 수 없다

→ 서로 협력해서 실행을 주고받으면서 작동하는 서브루틴

장점

- 일반 프로그램 로직을 기술하듯 작성하고, 상대편 코루틴에 데이터를 넘겨야하는 부분에서만 `yeild`를 사용

```kotlin
generator countdown(n) {
	while (n > 0) {
		yield n
		n -= 1
	}
}

for (i in countdown(10) {
	println(i)
}
```

## E.2 코틀린의 코루틴 지원: 일반적인 코루틴

코루틴을 구현할 수 있는 기본 도구를 언어가 제공

- `kotlin.coroutine` 패키지 밑에 기본 기능 지원
- 코틀린 1.3부터는 별도 설정 없이 모든 기능 사용 가능
- 다양한 형태의 코루틴: `kotlinx.coroutines` 패키지
    - https://github.com/Kotlin/kotlinx.coroutines

### 여러 가지 코루틴

**kotlinx.coroutines.CoroutineScope.launch**

- `launch`: 코루틴을 잡으로 반환, 코루틴은 기본적으로 즉시 실행
- `cancel()`을 호출하여 실행 중단 가능

```kotlin
import kotlinx.coroutines.GlobalScope
import kotlinx.coroutines.launch
import java.time.ZonedDateTime
import java.time.temporal.ChronoUnit

fun now() = ZonedDateTime.now().toLocalTime().truncatedTo(ChronoUnit.MILLIS)

fun log(msg: String) = println("${now()}:${Thread.currentThread()}: $msg")

fun launchInGlobalScope() {
    GlobalScope.launch {
        log("coroutine started.")
    }
}

fun main() {
    log("main() started")
    launchInGlobalScope()
    log("launchInGlobalScope executed")
    Thread.sleep(5000L)
    log("main() terminated")
}

01:03:41.526:Thread[main,5,main]: main() started
01:03:41.561:Thread[main,5,main]: launchInGlobalScope executed
01:03:41.565:Thread[DefaultDispatcher-worker-1,5,main]: coroutine started.
01:03:46.566:Thread[main,5,main]: main() terminated
```

메인 함수와 GlobalScope.launch가 만들어낸 코루틴이 서로 다른 스레드에서 실행된다

GlobalScope: 메인 스레드가 실행 중인 동안만 코루틴의 동작을 보장한다

```kotlin
fun runBlockingExample() {
    runBlocking {
        launch {
            log("GlobalScope.launch started.")
        }
    }
}

01:05:24.660:Thread[main,5,main]: main() started
01:05:24.696:Thread[main,5,main]: GlobalScope.launch started.
01:05:24.697:Thread[main,5,main]: launchInGlobalScope executed
01:05:29.706:Thread[main,5,main]: main() terminated
```

스레드가 모두 main 스레드

yield() 사용

```kotlin
fun yieldExample() {
    runBlocking {
        launch {
            log("1")
            yield()
            log("3")
            yield()
            log("5")
        }
        log("after first launch")
        launch {
            log("2")
            delay(1000L)
            log("4")
            delay(1000L)
            log("6")
        }
        log("after second launch")
    }
}

// 실행 결과
01:07:28.351:Thread[main,5,main]: after first launch
01:07:28.354:Thread[main,5,main]: after second launch
01:07:28.356:Thread[main,5,main]: 1
01:07:28.356:Thread[main,5,main]: 2
01:07:28.362:Thread[main,5,main]: 3
01:07:28.362:Thread[main,5,main]: 5
01:07:29.368:Thread[main,5,main]: 4
01:07:30.372:Thread[main,5,main]: 6
```

- launch는 즉시 반환됨
- runBlocking은 내부 코루틴이 모두 끝난 다음 반환됨
- delay(): 그 시간이 지날 때까지 다른 코루틴에게 실행을 양보
    - 모두 yield()를 쓴다면 1,2,3,4,5,6이 표시됨
    - 첫 번째 코루틴이 yield()를 하였지만, delay 상태에 있어서 다시 첫 번째 코루틴으로 돌아왔다

**kotlinx.coroutines.CoroutineScope.async**

async는 launch와 같은 일을 한다

- async는 Deffered를 반환한다
- Deffered는 Job을 상속하는 클래스

Deffered와 Job의 차이

- Job은 파라미터 없음, Deffered는 타입 파라미터가 있는 제네릭 타입
- Deffered 안에 await() 함수가 정의

async: 코드 블록을 비동기로실행, Deffered의 await를 사용하여 코루틴이 결과 값을 내놓을 때까지 기다렸다가 결과 값을 얻어온다

하나의 스레드에서 실행 → 병렬 처리

### 코루틴 컨텍스트와 디스패처

`CoroutineContext`: 코루틴이 실행 중인 여러 작업과 디스패처를 저장하는 일종의 맵

- 코틀린 런타임은 `CoroutineContext`을 사용해 다음에 실행할 작업을 선정, 어떻게 스레드에 배정할지 결정

```kotlin
runBlocking {
    launch {
        println("main runBlocking started. thread${Thread.currentThread().name}")
    }
    launch(Dispatchers.Unconfined) {
        println("Unconfined started. thread${Thread.currentThread().name}")
    }
    launch(Dispatchers.Default) {
        println("Default started. thread${Thread.currentThread().name}")
    }
    launch(newSingleThreadContext("MyOwnThread")) {
        println("newSingleThreadContext started. thread${Thread.currentThread().name}")
    }
}
```

전달하는 컨텍스트에 따라 서로 다른 스레드상에서 코루틴이 실행됨

### 코루틴 빌더와 일시 중단 함수

- produce
    - 정해진 채널로 데이터를 스트림으로 보내는 코루틴을 만든다
    - ReceiveChannel<> 반환
- actor
    - 정해진 채널로 메시지를 받아 처리하는 액터를 코루틴으로 만든다
    - SendChannel<> 반환, send() 메서드: 액터에게 메시지를 보낸다

delay(), yield(): 일시 중단 함수

다른 일시 중단 함수

- withContext: 다른 컨텍스트로 코루틴 전환
- withTimeout: 코루틴이 정해진 시간 안에 실행되지 않으면 예외 발생
- withTimeoutOrNull: 코루틴이 정해진 시간 안에 실행되지 않으면 null을 결과로 돌려줌
- awaitAll: 모든 작업의 성공을 기다린다
- joinAll: 모든 작업이 끝날 때까지 현재 작업 중단

## E.3 suspend 키워드와 코틀린의 일시 중단 함수 컴파일 방법

일반 함수에서 delay(), yield 사용시 컴파일 오류 발생

→ `suspend fun` 사용

```kotlin
suspend fun yieldThreeTimes() {
    log("1")
    delay(1000L)
    yield()
    log("2")
    delay(1000L)
    yield()
    log("3")
    delay(1000L)
    yield()
    log("4")
}
```

yield()를 해야 하는 경우 필요한 동작

- 코루틴 진입/나갈 때 실행 중이던 상태 저장/복구
- 현재 실행 중이던 위치 저장 + 다시 코루틴 재개될 때 해당 위치부터 실행을 재개
- 다음에 어떤 코루틴을 실행할지 결정: 컨텍스트 디스패처가 수행

상태 저장/복구, 위치 저장/재→ CPS(continuation passing style) 변환, 상태 기계를 활용

**CPS**

- 프로그램 실행 중 특정 시점 이후에 진행해야 하는 내용을 별도의 함수로 뽑고
- 그 함수에게 현재 시점까지 실행한 결과를 넘겨서 처리하게 만든다

## E.4 코루틴 빌더 만들기

### 제네레이터 빌더 사용법

```kotlin
fun idMaker() = generate<Int, Unit> {
    var index = 0
    while (index < 3) {
        yield(index++)
    }
}

fun main2() {
    val idMaker = idMaker()
    println(idMaker.next(Unit)) //0
    println(idMaker.next(Unit)) //1
    println(idMaker.next(Unit)) //2
    println(idMaker.next(Unit)) //null
}
```

### 제네레이터 빌더 구현

generator 반환: Generator<R, T> 타입


spring webflux + coroutine
https://huisam.tistory.com/entry/webflux-coroutine