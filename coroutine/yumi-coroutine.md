# 코루틴과 Async/Await

## 코루틴이란?
코루틴은 컴퓨터 프로그램 구성 요소 중 하나로 비선점형 멀티태스킹을 수행하는 일반화한 서브루틴이다. 코루틴은 실행을 일시중단하고 재개 할 수 있는 여러 진입 지점을 허용한다.
- 서브루틴 : 여러 명령어를 모아 이름을 부여해서 반복 호출할 수 있게 정의한 프로그램 구성 요소
- 비선점형 멀티태스킹 : 여러 개의 프로그램을 동시에 실행하는 것처럼 보이게 하는 것
코루틴에 데이터를 넘겨야 하는 부분에서만 yield를 사용

## 코틀린의 코루틴 지원 : 일반적인 코루틴
의존성 추가

메이븐
~~~
<dependency>
    <groupId>org.jetbrains.kotlinx</groupId>
    <artifactId>kotlinx-coroutines-core</artifactId>
    <version>1.3.9</version>
</dependency>
~~~
그래들
~~~
dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.0.1'
}
~~~
안드로이드
~~~
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.0.1'
~~~
### 여러 가지 코루틴
kotlinx.coroutines.core 모듈에 있는 코루틴 빌더

kotlinx.coroutines.CouroutineScope.launch 
- launch는 코루틴을 잡으로 반환하며, 만들어진 코루틴은 즉시 실행, cancel()을 호출하면 실행을 취소할 수 있다.
- main함수와 GlobalScope.launch가 만들어낸 코루틴이 서로 다른 스레드에서 실행된다.

kotinx.coroutines.CouroutineScope.async
- async는 launch와 같은 일을 한다. 
- 다른 점은 launch는 Job을 반환, async는 Deffered를 반환
- Job은 타입 파라미터가 없ㄷ음, Deffered는 타입 파라미터가 있음

실행하려는 작업 시간이 얼마 걸리지 않거나 I/O에 의한 대기시간이 크고, CPU 코어 수가 작아 실행할 수 있는 스레드 개수가 한정적일 때에는 코루틴이 일반 스레드르 사용하는 비동기보다 처리속도가 빠르다.

### 코루틴 컨택스트와 디스패처
CorutineContext는 실제 코루틴이 실행중인 여러 작업과 디스패처를 저장하는 맵이다. 코루틴 런타임은 이 Corutinecontext를 사용해서 실행할 작업을 선정하고, 어떻게 스레드에 배정할지에 대한 방법을 결정한다.

### 코루틴 빌더와 일시 중단 함수
코루틴 빌더
- produce : 정해진 채널로 데이터를 스트림으로 보내는 코루틴을 만든다.
- actor : 정해진 채널로 메시지를 받아 처리하는 액터를 코루틴으로 만든다.
코루틴 중단함수
- withContext: 다른 컨텐스트로 코루틴을 전환
- withTimeout : 코루틴이 정해진 시간 안에 실행되지 않으면 예외를 발생시키게한다.
- withTimeoutOrNull : 코루틴이 정해진 시간 안에 실행되지 않으면 null을 결과로 돌려준다.
- awaitAll :  모든 작업의 성공을 기다린다. 작업 중 어느 하나가 예외로 실패하면 awaitAll도 그 예외로 실패한다.
- joinAll : 모든 작업이 끝날 때까지 현재 작업을 일시 중단

## suspend 키워드와 코틀린의 일시 쭝단 함수 컴파일 방법
함수 정의의 fun 앞에 suspend를 넣으면 일시 중단 함수를 만들 수 있다.
CPS 변환은 프로그램의 실행 중 특점 시점 이후에 진행해야 하는 내용을 별도의 함수로 뽑고, 그 함수에게 현재 시점까지 실행한 결과를 넘겨서 처리하게 만드는 소스코드 변환 기술
