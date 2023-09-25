8장에서는 고차 함수와 인라인 함수에 대해서 공부한다. 고차 함수는 람다를 인자로 받거나 반환하는 함수를 말한다. 고차 함수를 사용하면 코드를 간결하게 만들 수 있고 코드 중복도 없앨 수 있고 더 나은 추상화를 구축할 수 있다. 인라인 함수는 람다를 사용함에 따라 발생할 수 있는 성능상 부가 비용을 없애고 람다 안에서 더 유연하게 흐름을 제어할 수 있게 한다.

## 8.1 고차 함수 정의

고차 함수는 위에서 언급한 것 처럼 **다른 함수**를 **인자로 받거나** **함수를 반환**하는 함수이다. 

### 8.1.1 함수 타입

```kotlin
val sum = { x: Int, y: Int -> x + y }
val action = { println(42) }
```

sum과 action은 람다를 값으로 갖는 변수이다. 타입 추론 때문에 따로 타입을 명시하지 않았지만 이 람다의 타입을 정의하게 되면 다음과 같이 할 수 있다.

```kotlin
val sumDefineType: (Int, Int) -> Int = { x, y -> x + y }
val actionDefineType: () -> Unit = { println(42) }
```

**(파라미터 타입, 파라미터 타입) → 반환 타입** 의 형식으로 타입을 정의할 수 있다.

```kotlin
val canReturnNull: (Int, Int) -> Int? = { x, y -> null}
val funOrNull: ((Int, Int) -> Int)? = null
```

다른 함수들과 마찬가지로 **파라미터 타입이나 반환 타입**이 **nullable** 타입이 될 수 있고, **함수의 타입**을 **nullable**로 정의할 수도 있다.

### 8.1.2 인자로 받은 함수 호출

```kotlin
fun twoAndThree(operator: (Int, Int) -> Int) {
    val result = operator(2, 3)
    println("The result is $result")
}

twoAndThree { a, b -> a + b }
twoAndThree { a, b -> a * b }

// 실행 결과
// The result is 5
// The result is 6
```

함수를 인자로 받고 싶으면 인자의 타입을 위에서 봤던 함수 타입으로 정의하면 된다. 그리고 사용을 할 때는 정의했던 함수 타입에 맞춰서 인자를 전달하면 된다. 

### 8.1.3 자바에서 코틀린 함수 타입 사용

함수 타입을 사용하는 코틀린 함수를 자바에서 호출이 가능하다. **자바 8** **람다**를 넘기면 자동으로 함수 타입의 값으로 변환된다. **자바 8 이전**의 자바에서는 **필요한 FunctionN 인터페이스의 invoke 메서드**를 구현하는 **무명 클래스를 넘기면 된다**. 

### 8.1.4 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터

함수의 인자로 함수 타입을 선언하는 경우에도 디폴트 값을 사용할 수 있다. 

```kotlin
fun twoAndThree(
    operator: (Int, Int) -> Int = { a, b -> a + b}
) {
    val result = operator(2, 3)
    println("The result is $result")
}

twoAndThree()
twoAndThree { a, b -> a - b }

// 실행 결과
// The result is 5
// The result is -1
```

일반 함수의 인자에 디폴트 값을 정의하듯이 인자 타입 뒤에 디폴트 값을 정의할 수 있다. 그래서 함수를 아무 인자 없이 호출하면 디폴트 인자 값을 사용하여 + 연산을 하고 인자로 함수를 넘겨주면 해당 함수에 맞는 연산을 하는 것을 볼 수 있다.

nullable 함수 타입을 사용할 수도 있다. 

```kotlin
fun foo(callback: (() -> Unit)?) {
    
    if (callback != null) {
        callback()
    }
}
```

근데 nullable 함수 타입을 사용하면 해당 함수를 직접 호출할 수 없다. NULL 여부를 명시적으로 호출이 가능하다. 

invoke() 메서드를 사용하면 이를 더 쉽게 구현할 수 있다.

```kotlin
fun foo(callback: (() -> Unit)?) {
    callback?.invoke()
}
```

### 8.1.5 함수를 함수에서 반환

```kotlin
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int) {
    fun getShippingCostCalculator(
        delivery: Delivery
    ): (Order) -> Double {
        if (delivery == Delivery.EXPEDITED) {
            return { order -> 6 + 2.1 * order.itemCount }
        }
        return { order -> 1.2 * order.itemCount }
    }
}
```

함수를 반환하는 함수를 정의하려면 함수의 반환 타입을 함수 타입으로 지정해야 한다. 위의 함수는 Order를 받아서 Double로 반환하는 함수를 의미한다. 이렇게 함수를 반환하는 함수는 상태나 조건에 따라 로직이 달라지는 프로그램을 구현할 때 유용하다. 

### 8.1.6 람다를 활용한 중복 제거

람다를 사용하면 중복을 간결하고 쉽게 제거할 수 있다. 

```kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val logs = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)
```

위와 같은 코드에서 WINDOWS 사용자의 접속 시간의 평균을 알아내기 위해서는 다음 코드를 사용하면 된다.

```kotlin
val averageWindowsDuration = logs
    .filter { it.os == OS.WINDOWS }
    .map { SiteVisit::duration }
    .average()
```

근데 만약 MAC 사용자의 접속 시간 평균을 알아내기 위해서는 `it.os == [OS.WINDOWS](http://OS.WINDOWS)` 부분을 바꿔야 하지만 확장 함수로 정의하면 중복을 제거할 수 있다. 

```kotlin
fun List<SiteVisit>.averageDurationFor(os: OS) =
    filter { it.os == os }.map { SiteVisit::duration }.average()
```

만약 IOS, ANDROID 사용자들의 평균 접속 시간을 알아내려면 다음의 코드를 사용하면 된다.

```kotlin
val averageMobileDuration = logs
    .filter { it.os in setOf(OS.IOS, OS.ANDROID)  }
    .map { SiteVisit::duration }
    .average()
```

근데 여기서 함수 타입, 고차 함수를 사용하면 코드의 중복을 앞에서처럼 줄일 수 있다.

```kotlin
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
    filter (predicate).map(SiteVisit::duration).average()
```

## 8.2 인라인 함수: 람다의 부가 비용 없애기

람다가 변수를 포획하면 람다가 생성되는 시점마다 새로운 무명 클래스 객체를 생성하는데 이럴 때 부가적인 비용이 든다. 그래서 람다를 사용하는 것이 같은 작업을 하는 일반 함수를 사용하는 것보다 덜 효율적이다. 

inline 변경자를 함수에 붙이면 컴파일러는 해당 함수의 본문을 바이트코드로 바꾼다. 그래서 inline 변경자를 사용하면 된다.

### 8.2.1 인라이닝이 작동하는 방식

inline 변경자가 붙은 함수는 컴파일러가 그 함수를 호출하는 코드를 함수 본문을 번역한 코드로 컴파일한다. 

```kotlin
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    }
    finally {
        lock.unlock()
    }
}

val l = Lock()
synchronized(l) {
    // ...
}
```

위의 함수를 호출하는 코드는 자바의 synchronized문과 똑같아 보이지만 자바에서는 임의의 객체에 대해서 synchronized를 사용할 수 있지만 이 함수는 Lock 클래스의 인스턴스를 요구한다는 차이가 있다. 

synchronized 함수를 inline으로 선언했기 때문에 synchronized를 호출하는 코드는 모두 자바의 synchronized문과 같아진다

```kotlin
 fun foo(l: Lock) {
    println("Before sync")

    synchronized(l) {
        println("Action")
    }
    println("After sync")
}
```

```kotlin
fun _foo_ (l: Lock) {
    println("Before sync") // synchronized 함수를 호출하는 foo 함수의 코드
    l.lock() // synchronized 함수가 인라이닝 된 코드
    try {    // synchronized 함수가 인라이닝 된 코드
        println("Action")  // 람다 코드의 본문이 인라이닝 된 코드
    } finally {    // synchronized 함수가 인라이닝 된 코드
        l.unlock() // synchronized 함수가 인라이닝 된 코드
    }              // synchronized 함수가 인라이닝 된 코드
    println("After sync") // synchronized 함수를 호출하는 foo 함수의 코드
}
```

synchronized 함수의 본문뿐 아니라 synchronized에 전달된 람다의 본문도 함께 인라이닝된다. 

인라인 함수를 호출하면서 람다 대신에 함수 타입의 변수를 넘길 수도 있다.

```kotlin
class LockOwner(val lock: Lock) {
    fun runUnderLock(body: () -> Unit) {
        synchronized(lock, body)
    }
}
```

이런 경우에서는 람다 본문은 인라이닝 되지 않고 synchronized 함수의 본문만 인라이닝 된다. 

### 8.2.2 인라인 함수의 한계

인라이닝 방식으로 람다를 사용하는 모든 함수를 인라이닝 할 수 없다. 함수가 인라이닝 될 때 그 함수에 인자로 전달된 람다 식의 본문은 결과 코드에 직접 들어갈 수 있는데 이 때문에 전달 받은 람다를 본문에서 사용하는 방식이 한정적이게 된다. 

함수의 파라미터로 받은 람다는 다른 변수에 저장하고 나중에 사용할 수 없는데 이는 람다를 표현하는 객체가 어딘가에는 존재해야 하기 때문이다. 

### 8.3.3 컬렉션 연산 인라이닝

코틀린 표준 라이브러리의 컬렉션 함수는 대부분 람다를 인자로 받는다. 표준 라이브러리를 사용하는 것과 직접 연산을 구현하는 것 둘 중 어느 것이 더 효율적일까?

```kotlin
// 코틀린 표준 라이브러리 사용

val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.filter { it.age < 30 })
```

```kotlin
// 직접 구현

val result = mutableListOf<Person>()
for(person in people) {
    if (person.age < 30) result.add(person)
}
println(result)
```

코틀린의 filter 함수는 인라인 함수이다. 그래서 filter 함수의 바이트코드는 그 함수에 전달된 람다 본문의 바이트코드와 함께 filter를 호출한 위치에 들어간다. 그래서 표준 라이브러리를 사용한 예제에서 filter를 사용해서 생긴 바이트코드와 직접 구현한 예제에서 생긴 바이트코드는 거의 같다. 

### 8.2.4 함수를 인라인으로 선언해야 하는 경우

모든 함수에 inline 키워드를 사용하면 코드를 더 빠르게 만들 수 있다는 생각을 할 수 있다. 하지만 inline 함수를 사용해도 람다를 인자로 받는 함수만 성능이 좋아질 가능성이 높다. 

람다를 인자로 받는 함수는 인라이닝 하면 좋은 점이 많다.

1. 인라이닝을 통해 없앨 수 있는 부가 비용이 상당하다. (함수 호출 비용, 람다를 표현하는 클래스, 람다 인스턴스에 해당하는 객체 만들 필요 X)
2. JVM은 함수 호출과 람다를 인라이닝 해 줄 정도로 똑똑하지 못하다.
3. 일반 람다에서는 사용할 수 없는 몇 가지 기능을 사용할 수 있다. 

inline 변경자를 함수에 사용할 때는 코드의 크기에 주의를 기울여야 한다. 인라이닝은 바이트코드 크기를 증가시키기 때문이다.

### 8.2.5 자원 관리를 위해 인라인된 람다 사용

람다로 중복을 없앨 수 있는 일반적인 패턴 중 하나는 자원 관리이다. 자원 관리는 보통 try/finally문으로 try문 안에서는 자원 획득, finally문 안에서는 자원 해제하는 방식으로 구현된다. 

자바에서는 try-with-resource문으로 이를 처리할 수 있다. 

```kotlin
static String readFirstFromFile(String path) throws IOException {
        try (BufferedReader br =
                     new BufferedReader(new FileReader(path))) {
            return br.readLine();
        } 
    }
```

코틀린에서는 자바의 try-with-resource와 같은 기능을 제공하는 use 함수가 있다. 

```kotlin
fun readFirstFromFile(path: String): String {
    BufferedReader(FileReader(path)).use { br -> return br.readLine() }
}
```

use 함수는 close 할 수 있는 자원의 확장 함수이며 람다를 인자로 받는다.  use는 람다를 호출 한 다음 자원을 close한다. 

## 8.3 고차 함수 안에서 흐름 제어

### 8.3.1 람다 안의 return문: 람다를 둘러싼 함수로부터 반환

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}
```

람다 안에서 return을 사용하면 그 람다를 호출하는 함수가 실행을 끝내고 반환된다. 이와 같이 자신을 둘러싸고 있는 블록보다 더 바깥에 있는 다른 블록을 반환하게 만드는 return문을 넌로컬(non-local return)이라고 부른다. 이는 람다를 인자로 받는 함수가 인라인 함수인 경우에만 가능하다. 

### 8.3.2 람다로부터 반환: 레이블을 사용한 return

람다 식에서도 로컬 return을 사용할 수 있다. 람다 안에서 로컬 return은 for 반복문의 break와 비슷한 역할을 한다. 로컬 return은 람다의 실행을 끝내고 람다를 호출했던 코드의 실행을 계속한다. 

로컬 return과 넌로컬 return을 구분하기 위해서는 label을 사용해야 한다. 

```kotlin
fun lookForAlice(people: List<Person>) {
	people.forEach label@{
		if (it.name == "Alice") return@label
	}
	println("Alice might be somewhere")
}
```

return으로 실행을 끝내고 싶은 람다 식 앞에 레이블을 붙이고 return 키워드 뒤에 그 레이블을 추가하면 된다.

람다에 레이블을 붙여서 사용하는 대신 람다를 인자로 받는 인라인 함수의 이름을 return 뒤에 레이블로 사용해도 된다.

```kotlin
fun lookForAlice(people: List<Person>) {
	people.forEach {
		if (it.name == "Alice") return@forEach
	}
	println("Alice might be somewhere")
}
```

### 8.3.3 무명 함수: 기본적으로 로컬 return

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(fun (person) {
        if (it.name == "Alice") return // 가장 가까운 함수인 무명 함수에서 리턴
        println("${person.name} is not Alice")
    })
}
```
