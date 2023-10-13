# E.1 코루틴이란?

위키피디아에서는 코루틴을 다음과 같이 정의하고 있다. 

> 코루틴은 컴퓨터 프로그램 구성 요소 중 하나로 **비선점형 멀티태스킹**을 수행하는 일반화한 **서브루틴**이다. 코루틴은 실행을 **일시 중단**하고 **재개**할 수 있는 여러 진입 지점을 허용한다.
> 

서브루틴은 함수를 의미한다. 서브루틴을 실행할 수 있는 방법은 해당 서브루틴을 호출하는 것 뿐이다. 서브루틴은 값을 반환하면 실행 중이던 모든 상태를 잃어버린다. 

멀티태스킹은 여러 동작을 동시에 수행하는 것처럼 보이게 하거나 실제로 동시에 수행하는 것을 의미한다. 

비선점형은 운영체제가 멀티태스킹의 각 작업을 강제로 중지 시키고 다른 작업을 실행할 수 없는 것이다. 한 작업(프로세스)가 독점적으로 작업을 수행할 수 없게 된다. 

이를 종합해보면 코루틴은 다음과 같이 설명할 수 있을 것 같다. 

<aside>
💡 서로 실행을 독점하지 않고 각각 작업을 실행하는 서브루틴
</aside>

</br>
</br>
밑의 그림은 함수 A를 실행하다가 함수 B를 실행하는 상황과 함수 A를 실행하다가 코루틴 B를 실행하는 상황을 비교한 것이다. 

![서브루틴vs코루틴](https://github.com/BEEEAM-J/Kotlin-In-Action/assets/107917980/ac19f814-64d5-4fd6-b5ae-cadf80f36888)


서브루틴의 경우에는 실행 상태가 지워지기 때문에 처음부터 매번 호출할 때마다 처음부터 실행하지만 코루틴의 경우 실행 상태가 남아있기 때문에 이전에 실행 했던 지점부터 실행하는 것을 알 수 있다. 

# E.2 코틀린의 코루틴 지원: 일반적인 코루틴

안드로이드에서 코루틴을 사용하려면 다음과 같이 의존 관계를 추가해야 한다. 

```kotlin
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.1'
```

## E.2.1 여러 가지 코루틴

코루틴 빌더는 이름처럼 코루틴을 만들어준다. 코틀린에서는 코루틴에 원하는 동작을 람다로 넘겨서 코루틴으로 만들어서 실행할 수 있다.

### Kotlinx.coroutines.CoroutineScope.launch

launch 빌더는 코루틴을 Job 객체로 반환하며 기본적으로 만들어진 즉시 실행된다. 그리고 원하면 Launch가 반환하는 Job의 cancel 함수를 호출하여 코루틴을 중지할 수 있다. 

```kotlin
fun now() = ZonedDateTime.now().toLocalTime().truncatedTo(ChronoUnit.MILLIS)

fun log(msg: String) = println("${now() }: ${Thread.currentThread()}: ${msg}")

fun launchInGlobalScope() {
    GlobalScope.launch {
        log("Coroutine started")
    }
}

fun main() {
    log("main() started")
    launchInGlobalScope()
    log("launchInGlobalScope() executed")

    Thread.sleep(5000L)
    log("main() terminated")
}

// 실행 결과
// 01:54:44.108: Thread[main,5,main]: main() started
// 01:54:44.233: Thread[main,5,main]: launchInGlobalScope() executed
// 01:54:44.241: Thread[DefaultDispatcher-worker-2,5,main]: Coroutine started
// 01:54:49.247: Thread[main,5,main]: main() terminated
```

여기서 확인할 수 있는 점은 main 함수와 GlobalScope.launch가 만들어낸 코루틴은 다른 스레드에서 실행된다는 점이다. GlobalScope.launch로 만든 코루틴은 main 함수가 동작하는 동안만 실행이 된다. 그래서 main 함수에서 sleep 함수를 사용하지 않으면 코루틴은 실행되지 않는다. 

runblocking 함수는 코루틴의 실행이 끝날 때 까지 현재 스레드를 블록시키는 함수이다. 위의 코드에서 launchInGlobalScope 함수를 다음과 같이 바꿔보자

```kotlin
fun runBlockingExample() {
    runBlocking {
        launch {
            log("Coroutine started")
        }
    }
}

// 실행 결과
// 11:23:44.780: Thread[main,5,main]: main() started
// 11:23:44.863: Thread[main,5,main]: Coroutine started
// 11:23:44.864: Thread[main,5,main]: launchInGlobalScope() executed
// 11:23:49.870: Thread[main,5,main]: main() terminated
```

GlobalScope를 사용했을 때와는 다르게 모두 main 스레드에서 실행되는 것을 확인할 수 있다. 그리고 결과를 봐도 코루틴이 실행되는 시점이 다른 것을 확인할 수 있다. runblocking은 코루틴의 실행이 끝날 때 까지 현재 스레드를 블록하기 때문에 이와 같은 결과가 나오게 된다.

### Kotlinx.coroutines.CoroutineScope.async

async는 launch와 같은 역할을 하지만 유일한 차이점은 launch는 Job을 반환하는 것이고 async는 Deffered를 반환하는 것이다. 근데 Deffered는 Job을 상속한 클래스라서 launch 대신 async를 사용해도 상관없다. 

```kotlin
public fun kotlinx.coroutines.CoroutineScope.launch(
    context: kotlin.coroutines.CoroutineContext /* = compiled code */, 
    start: kotlinx.coroutines.CoroutineStart /* = compiled code */, 
    block: suspend kotlinx.coroutines.CoroutineScope.() -> kotlin.Unit
): kotlinx.coroutines.Job { /* compiled code */ }

public fun <T> kotlinx.coroutines.CoroutineScope.async(
    context: kotlin.coroutines.CoroutineContext /* = compiled code */, 
    start: kotlinx.coroutines.CoroutineStart /* = compiled code */, 
    block: suspend kotlinx.coroutines.CoroutineScope.() -> T
): kotlinx.coroutines.Deferred<T> { /* compiled code */ }
```

Deffered와 Job의 차이점은 Job은 아무 타입 파라미터가 없지만 Deffered는 제네릭 타입이라는 것과 Deffered 안에는 await() 함수가 정의 되어있다는 것이다. 그래서 Deffered의 await를 사용하여 코루틴이 값을 반환할 때까지 기다렸다가 결과를 받을 수 있다.

```kotlin
fun sumAll() {
    runBlocking {
        val d1 = async { delay(1000L); 1 }
        log("after async(d1)")
        val d2 = async { delay(2000L); 2 }
        log("after async(d2)")
        val d3 = async { delay(3000L); 3 }
        log("after async(d3)")

        log("1+2+3 = ${d1.await() + d2.await() + d3.await()}")
    }
}

// 실행 결과
// 12:09:26.820: Thread[main,5,main]: after async(d1)
// 12:09:26.828: Thread[main,5,main]: after async(d2)
// 12:09:26.829: Thread[main,5,main]: after async(d3)
// 12:09:29.842: Thread[main,5,main]: 1+2+3 = 6
```

d1, d2, d3을 실행하면 1+2+3 = 6초 걸릴 것으로 예상할 수 있지만 실제로는 3초만 걸린다. 이를 통해서 async를 사용하면 한 스레드에서 코드가 순차적으로 실행되는 것을 확인할 수 있다.  

### E.2.2 코루틴 컨텍스트와 디스패처

CoroutineContext는 코루틴이 실행 중인 여러 작업과 디스패처를 저장한다. 이는 코루틴 스코프의 인자로 받을 수 있다.  코틀린 런타임은 CoroutineContext를 가지고 다음에 실행할 작업과 어떻게 스레드를 배정할 지 결정한다. 

```kotlin
fun contextTest() {
    runBlocking {
        launch {
            println("main runBlocking: I'm working in thread ${Thread.currentThread().name}")
        }

        launch(Dispatchers.Unconfined) {
            println("Unconfined: I'm working in thread ${Thread.currentThread().name}")
        }

        launch(Dispatchers.Default) {
            println("Default   : I'm working in thread ${Thread.currentThread().name}")
        }

        launch(newSingleThreadContext("MyOwnThread")) {
            println("newSingleThreadContext: I'm working in thread ${Thread.currentThread().name}")
        }
    }
}

// 실행 결과
// Unconfined: I'm working in thread main
// Default   : I'm working in thread DefaultDispatcher-worker-1
// main runBlocking: I'm working in thread main
// newSingleThreadContext: I'm working in thread MyOwnThread
```

위의 예제를 통해서 같은 launch를 사용해도 CoroutineContext에 따라 코루틴이 다른 스레드에서 실행되는 것을 확인할 수 있다.

### E.2.3 코루틴 빌더와 일시 중단 함수

지금까지 본 launch, async, runblocking 모두 코루틴 빌더라고 불린다. 코루틴 빌더들은 코루틴을 만들어주는 함수이다. 이 3개 말고 다른 함수들도 있다.

- produce : 정해진 채널로 데이터를 스트림으로 보내는 코루틴 생성
- actor : 정해진 채널로 메시지를 받아 처리하는 액터를 코루틴으로 만든다.

delay, yield는 코루틴의 일시 중단 함수이다. 이 2개 말고 다른 함수들도 있다.

- withContext() : 다른 컨텍스트로 코루틴을 전환
- withTimeout() : 코루틴이 정해진 시간 안에 실행되지 않으면 예외 발생
- withTimeoutOrNull() : 코루틴이 정해진 시간 안에 실행되지 않으면 null을 반환
- awaitAll() : 모든 작업의 성공을 기다린다. 작업 중 하나라도 예외로 실패하면 예외 발생
- joinAll() : 모든 작업이 끝날 때까지 현재 작업을 일시 중단

## E.3 suspend 키워드와 코틀린의 일시 중단 함수 컴파일 방법

일시 중단 함수는 코루틴이나 일시 중단 함수의 내부가 아닌 곳에서 호출하는 것은 컴파일러 시점에서 막힌다. 

코틀린에서는 코루틴을 지원하기 위해 suspend 키워드를 제공한다. suspend 키워드를 fun 앞에 붙이면 일시 중단 함수로 만들 수 있다. 

일시 중단 함수 안에서 yield()를 해야 하는 경우 다음과 같은 동작들이 필요하다.

1. 코루틴에 진입할 때와 나갈 때 코루틴이 실행 중이던 상태를 저장, 복구하는 작업
2. 현재 실행 중이던 위치 저장, 재개할 때 해당 위치부터 실행하는 작업
3. 다음에 어떤 코루틴을 실행할 지 결정하는 작업

컴파일러는 1, 2번 동작을 컴파일 하기 위해 컨티뉴에이션 패싱 스타일 (CPS) 변환과 상태 기계를 활용하여 코드를 생성한다. 

CPS 변환은 프로그램 실행 중 특정 시점 이후의 내용을 별도의 함수로 뽑고 그 함수에게 실행한 결과를 넘겨서 처리하게 만드는 기술이다. 그래서 실행하던 위치, 상태를 저장, 복구가 가능하다. 

상태기계는 CPS를 사용할 때 너무 많은 코드가 컨티뉴에이션 함수가 되는 것을 막기 위해 코루틴이 다른 함수로 제어를 넘기는 상황에서만 컨티뉴에이션이 생기도록 만들 때 사용한다.
