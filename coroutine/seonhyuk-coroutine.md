# 코루틴과 Async/Await

## 1. 코루틴이란?

> 코루틴은 컴퓨터 프로그램 구성 요소 중 하나로 비선점형 멀티태스킹을 수행하는 일반화한 서브루틴이다. 코루틴은 실행을 일시 중단(suspend)하고 재개(resume)할 수 있는 여러 진입 지점을 허용한다.

**서브루틴** : 여러 명령어를 모아 이름을 부여해서 반복호출할 수 있게 정의한 프로그램 구성요소로, 다른 말로 함수(메서드)

- 진입하는 유일한 방법은 함수 호출
- 호출 할 때 활성 레코드라는 것이 스택에 할당되면서 서브루틴 내부의 로컬 변수 등이 초기화
- 서브루틴 안에서 여러 번 return을 사용할 수 있기 때문에 서브루틴이 실행을 중단하고 제어를 호출한 쪽에게 돌려주는 지점은 여럿 있을 수 있다.
- 반환되고 나면 활성 레코드가 스택에서 사라지기 때문에 실행 중이던 모든 상태를 잃어버린다.
- 서브루틴을 여러 번 반복 실행해도 항상 같은 결과를 반복해서 얻게 된다.

**멀티태스킹** : 여러 작업을 동시에 수행하는 것처럼 보이거나 실제로 동시에 수행하는 것
**비선점형** : 멀티태스킹의 각 작업을 수행하는 참여자들의 실행을 운영체제가 강제로 일시 중단시키고 다른 참여자를 실행하게 만들 수 없다. 각 참여자들이 서로 자발적으로 협력해야만 비선점형 멀티태스킹이 제대로 작동할 수 있다.

> 따라서, 코루틴이란 서로 협력해서 실행을 주고받으면서 작동하는 여러 서브루틴을 말한다.

코루틴을 사용하는 경우 일반적인 프로그램 로직을 기술하듯 코드를 작성하고 상대편 코루틴에 데이터를 넘겨야 하는 부분에서만 yield를 사용하면 된다는 점

## 2 코틀린의 코루틴 지원: 일반적인 코루틴

코틀린은 특정 코루틴을 언어가 지원하는 형태가 아니라, 코루틴을 구현할 수 있는 기본 도구를 언어가 제공하는 형태
코틀린 1.3부터는 코틀린을 설치하면 별도의 설정 없이도 모든 기능을 사용할 수 있다.

### 2.1 여러 가지 코루틴

코틀린에서는 코루틴 빌더에 원하는 동작을 람다로 넘겨서 코루틴을 만들어 실행하는 방식으로 코루틴을 활용

`kotlinx.coroutines.CoroutineScope.launch`

`launch`는 코루틴을 잡으로 반환하며, 만들어진 코루틴은 기본적으로 즉시 실행된다.
원하면 launch가 반환한 잡의 cancel()을 호출해 코루틴 실행을 중단시킬 수 있다.
launch가 작동하려면 CoroutineScope 객체가 블록의 this로 지정돼야 하는데, 다른 suspend 함수 내부라면 해당 함수가 사용 중인 CoroutineScope가 있겠지만, 그렇지 않은 경우에는 GlobalScope를 사용하면 된다.

```kt
import kotlinx.coroutines.*
import java.time.ZonedDateTime
import java.time.temporal.ChronoUnit

fun now() = ZonedDateTime.now().toLocalTime().truncatedTo(ChronoUnit.MILLIS)

fun log(msg:String) = println("${now()}:${Thread.currentThread()}: ${msg}")

fun launchInGlobalScope() {
    GlobalScope.launch {
        Thread.sleep(3000L)
        log("coroutine started.")
    }
}

fun main(args: Array<String>) {
    log("main() started.")
    launchInGlobalScope()
    log("launchInGlobalScope() executed")
    Thread.sleep(5000L)
    log("main() terminated")
}

// 23:49:27.131:Thread[main,5,main]: main() started.
// 23:49:27.153:Thread[main,5,main]: launchInGlobalScope() executed
// 23:49:30.160:Thread[DefaultDispatcher-worker-1,5,main]: coroutine started.
// 23:49:32.158:Thread[main,5,main]: main() terminated
```

메인 함수와 GlobalScope.launch가 만들어낸 코루틴이 서로 다른 스레드에서 실행된다.
GlobalScope는 메인 스레드가 실행 중인 동안만 코루틴의 동작을 보장해준다.
스레드는 모두 main 스레드

```kt
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
            delay(1000)
            log("4")
            delay(1000)
            log("6")
        }
        log("after second launch")
    }
}

fun main(args: Array<String>) {
    log("main() started.")
    yieldExample()
    log("launchInMainThread")
}
```

1. log("main() started.") 실행
2. yieldExample() 실행 - runBlocking이므로 같은 스레드에서 실행되며 내부 코루틴이 모두 끝난 후 반환
3. launch - log("after first launch") - launch - log("after second launch")가 순차적으로 실행되나, launch는 즉시반환되어 launch 내부 코루틴이 실행되기 전에 log가 먼저 찍힘
4. 첫 번째 코루틴 실행되서 log("1")
5. yield()호출 해 두 번째 코루틴으로 양보 후 log("2")
6. 두 번째 코루틴이 1초간 delay되어 다시 첫 번째 코루틴 log("3") 실행
7. 이후 yield()가 있으나 두 번째 코루틴이 아직 delay 상태라 ("5") 실행
8. 이후 log("4") - delay - log("6")이 찍히고 yieldExample() 반환
9. log("launchInMainThread") 실행

`kotlinx.coroutins.CoroutineScope.async`

`launch`와 사실상 같은 일을 한다.
`launch`는 Job을 반환하지만 `async`는 Deferred를 반환한다.

Job은 아무 타입 파라미터가 없는데, Deferred는 타입 파라미터가 있는 제네릭 타입이며, await() 함수가 정의돼 있다.

```kt
fun sumAll() {
    runBlocking {
        val d1 = async { delay(1000); 1}
        log("after async(d1)")
        val d2 = async { delay(1000); 2}
        log("after async(d2)")
        val d3 = async { delay(1000); 3}
        log("after async(d3)")

        log("1+2+3 = ${d1.await() + d2.await() + d3.await()}")
        log("after await all & add")
    }
}

fun main(args: Array<String>) {
    log("main() started.")
    sumAll()
    log("sumAll() executed")
    log("main() terminated")
}
```

- 실행하려는 작업이 시간이 얼마 걸리지 않거나 I/O에 의한 대기 시간이 크고, CPU 코어 수가 작아 동시에 실행할 수 있는 스레드 개수가 한정된 경우에는 특히 코루틴과 일반 스레드를 사용한 비동기 처리 사이에 차이가 커진다.

### 2.2 코루틴 컨텍스트와 디스패처

- launch, async 등은 모두 CoroutineScope의 확장 함수
- CoroutineScope에는 CoroutineContext 타입의 필드 하나만 들어있다.
- CoroutineScope는 CoroutineContext 필드를 launch 등의 확장 함수 내부에서 사용하기 위한 매개체 역할만을 담당

```kt
public interface CoroutineScope {
    public val coroutineContext: CoroutineContext
}
```

- 원한다면 확장함수들에 CoroutineContext를 넘길 수 있다.

**CoroutineContext** : 실제로 코루틴이 실행 중인 여러 작업과 디스패처를 저장하는 일종의 맵

```kt
fun contextExample() {
    runBlocking {
        launch {
            log("main runBlocking")
        }
        launch(Dispatchers.Unconfined) {
            log("Unconfined")
        }
        launch(Dispatchers.Default) {
            log("Default")
        }
        launch(newSingleThreadContext("MyOwnThread")) {
            log("newSingleThreadContext")
        }
    }
}

fun main(args: Array<String>) {
    contextExample()
}

// 00:50:09.187:Thread[main,5,main]: Unconfined
// 00:50:09.190:Thread[DefaultDispatcher-worker-1,5,main]: Default
// 00:50:09.194:Thread[MyOwnThread,5,main]: newSingleThreadContext
// 00:50:09.194:Thread[main,5,main]: main runBlocking
```

- 넘기는 context마다 다른 thread에서 코루틴이 실행

### 2.3 코루틴 빌더와 일시 중단 함수

launch, async, runBlocking은 모두 코루틴 빌더 - 코루틴을 만들어주는 함수

`kotlinx-coroutines-core` 모듈이 제공하는 코루틴 빌더는 2가지가 더 있다.

**produce** : 정해진 채널로 데이터를 스트림으로 보내는 코루틴을 만든다. 이 함수는 ReceiveChannel<>을 반환한다. 그 채널로부터 메시지를 전달받아 사용할 수 있다.

**actor** : 정해진 채널로 메시지를 받아 처리하는 액터를 코루틴으로 만든다. 이 함수가 반환하는 SendChannel<> 채널의 send() 메서드를 통해 액터에게 메시지를 보낼 수 있다.

delay와 yield는 일시 중단 함수라고 부른다. 이 외에도

- withContext : 다른 컨텍스트로 코루틴을 전환
- withTimeout : 코루틴이 정해진 시간 안에 실행되지 않으면 예외 발생
- withTimeoutOrNull : 코루틴이 정해진 시간 안에 실행되지 않으면 null을 반환
- awaitAll : 모든 작업의 성공을 기다린다.
- joinAll : 모든 작업이 끝날 때까지 현재 작업을 일시 중단시킨다.

이 있다.

## 3 suspend 키워드와 코틀린의 일시 중단 함수 컴파일 방법

코루틴이 아닌 일반 함수 속에서 delay나 yield를 쓰면 오류 발생
일시 중단 함수를 코루틴이나 일시 중단 함수가 아닌 함수에서 호출하는 건 컴파일러 수준에서 금지된다.

-> 그러면 일시 중단 함수는 어떻게 만드는가?

- 코틀린은 코루틴 지원을 위해 suspend 키워드를 제공, 함수 앞에 넣으면 일시 중단 함수를 만들 수 있다.

```kt
suspend fun yieldThreeTimes() {
    log("1")
    delay(1000)
    yield()
    log("2")
    delay(1000)
    yield()
    log("3")
}

fun suspendExample() {
    GlobalScope.launch {
        yieldThreeTimes()
    }
}
```

suspend 함수의 작동 방식

CPS(continuation passing style) 변환과 상태 기계를 활용해 코드 생성

- CPS 변환은 프로그램의 실행 중 특정 시점 이후에 진행해야 하는 내용을 별도의 함수로 뽑고, 그 함수에게 현재 시점까지 실행한 결과를 넘겨서 처리하게 만드는 소스코드 변환 기술
- 다음에 할 일을 함수 형태로 전달
- callback 스타일 프로그래밍과도 유사
