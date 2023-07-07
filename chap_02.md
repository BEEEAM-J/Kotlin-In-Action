### 함수

**기본 형태**

```kotlin
fun 함수이름(파라미터: 파라미터 타입): 반환 타입 {
  함수 본문...
}

ex)
fun test(n1: Int, n2: Int): Int {
  return if (a > b) a else b
}
```
- fun 키워드를 사용하여 작성
- 자바와 달리 클래스 밖에서도 정의 가능

<aside>
💡 문(Statement) vs 식(Expression)<br>
  
문: 아무런 값을 만들어내지 않는다.<br>
식: 값을 만들어낸다.<br>

**무언가를 반환하는지 안하는지의 차이인듯?<br>
반환하면 식, 안하면 문**

</aside>

코틀린에서는 식이 본문인 함수가 자주 사용된다. 

```kotlin
ex) 블럭이 본문인 함수
fun test(n1: Int, n2: Int): Int {
		return if (a > b) a else b
}

ex) 식이 본문인 함수
fun test(n1: Int, n2: Int): Int = if(a > b) a else b

↓ 더 간단하게

fun test(n1: Int, n2L Int) = if(a > b) a else b
```

식이 본문인 함수의 경우 컴파일러가 타입 추론을 통해서 타입을 유추할 수 있기 때문에 반환 타입을 생략하고 작성하여도 된다. **(식이 본문인 함수만 가능!)**

<br>

### 변수

```kotlin
val ans = 42
val ans: Int = 42

val n: Int
n = 3
```

- 타입 생략 가능 (초기화 식을 사용하지  않고 변수를 선언할 때는 변수 타입 명시 해야함!)

- **val**: 변경할 수 없는 참조를 저장하는 변수
참조가 가리키는 객체 내부 값은 변경될 수 있음<br>
ex)<br>
val food = *arrayListOf*("ham")<br>
food.add("pizza")
- **var**: 변경 가능한 변수
변수의 타입은 고정

<br>

**문자열 템플릿**

더 쉽게 문자열 형식 지정 

```kotlin
val name: String = "Beeeam"
println("Hello! $name")

val n = 5
println("Hello! ${if (n > 10) "Bee" else "Who..?"}")
```

문자열 내에서 $문자열 변수 로 사용 가능

변수 말고 식을 넣으려면 중괄호로 감싸면 된다. ${} 형식

<br>

### class

자바에서는 데이터를 필드에 저장하고, 멤버의 필드는 보통 private으로 선언한다. 그리고 해당 데이터를 사용하게 하기 위해서 접근자 메서드를 제공한다. (getter(), setter()) 

프로퍼티? → 필드와 접근자를 묶은 개념

근데 코틀린에서는 변수의 종류를 통해서 이를 선언한다. 

```kotlin
class Person (
  val name: String,          // -> getter()
  var isMarried: Boolean     // -> getter(), setter()
)
```

private을 붙이지 않으므로 다른 클래스에서 해당 변수에 접근할 수 있게 한다. (접근 가능하게 함)

val → getter() 역할 

var → getter(), setter() 역할

<br>

**클래스 사용 예시**

```kotlin
val p = Person("Beeeam", false)
println(p.name)       // -> getter() 기능
println(p.isMarried)  // -> getter() 기능
p.isMarried = true    // -> setter() 기능
```

자바와 달리 생성자를 호출한다. 그리고 getter()와 setter()를 사용하지 않고 프로퍼티를 직접 사용한다.

<br>

**커스텀 접근자**

커스텀 접근자를 사용하면 프로퍼티 값을 필드에 저장할 필요가 없게 할 수 있다.

```kotlin
class Rect1(val height: Int, val width: Int) {
    val inSquare: Boolean
        get() {
            return height == width
        }
}
```

위와 같이 커스텀 접근자를 사용하면 값을 저장하는 필드가 필요 없다. 구현을 제공하는 게터만 존재한다.

<br>

**디렉터리와 패키지**

같은 패키지에 속해 있다면 다른 파일에서 정의한 코드를 사용할 수 있다. 하지만 다른 패키지에 있는 코드를 사용하기 위해서는 import를 해야 한다. import는 파일 앞에 선언해야 하고, import 키워드를 사용한다.

<br>

### enum 클래스

enum 클래스는 열거형 클래스이다. 코드가 단순해져서 가독성이 좋아지는 장점이 있다. 

각 열거 형은 enum 클래스의 인스턴스이다. 

```kotlin
enum class Color(
    val r: Int, val g: Int, val b: Int
) {
    RED(255, 0, 0),
    ORANGE(255, 165, 0),
    YELLOW(255, 255, 0);

    fun rgb() = (r * 256 + g) * 256 + b
}

println(Color.RED.rgb())
println(Color.YELLOW.r)
```

enum class 내부에 메서드를 정의하는 경우에는 상수와 메서드 사이에 세미클론(;)을 꼭 넣어야 한다.

<br>

**when 함수**

```kotlin
fun getMnemonic(color: Color) =
    when(color) {
        Color.RED, Color.ORANGE -> "RO"
        Color.YELLOW -> "Y"
        Color.GREEN -> "G"
    }

fun mix(c1: Color, c2: Color) =
    when(setOf(c1, c2)) {
        setOf(Color.RED, Color.YELLOW) -> "ORANGE"
        setOf(Color.RED, Color.YELLOW) -> "THINKING..."
        setOf(Color.RED, Color.YELLOW) -> "WHAT..?"
        else -> "X"
    }

println(getMnemonic(Color.ORANGE))
println(mix(Color.RED, Color.YELLOW))
```

함수의 파라미터로 위에서 선언한 enum class를 받고 when 함수를 통해서 각 enum class의 인스턴스에 대한 값을 선언할 수 있다. 

한 분기에 여러 값을 사용하려면 값 사이를 콤마(,)로 구분하면 된다.

위의 mix 함수 안의 when 함수의 분기 조건은 setof 함수를 사용하여 집합으로 선언하였다. 이 처럼 코틀린의 when 함수의 분기 조건은 임의의 객체를 허용한다.

<br>

### 스마트 캐스트

코틀린에서는 컴파일러가 캐스팅을 해주는데 이를 스마트 캐스트라고 한다. 스마트 캐스트는 is로 변수에 든 값의 타입을 검사한 다음에 그 값이 바뀔 수 없는 경우에만 동작한다. 

만약 원하는 타입으로 명시적으로 타입 캐스팅을 하려면 as 키워드를 사용하면 된다.

<br>

### 이터레이션

루프를 도는 함수들을 의미한다.

<br>

**while**

```kotlin
while (조건) {
		...
}

do {
		...
} while (조건)
```

위의 while문은 조건이 참인 동안만 블록 내부를 실행하고, 밑의 do while문은 **한 번은 무조건 실행**하고 다음에 조건이 참인 동안에 블록 내부를 실행하는 차이점이 있다.

<br>

**for**

for문에서는 범위를 사용하는데 이 범위는 숫자로 구성되는데 시작 값과 끝 값 사이에 .. 연산자를 사용해서 연결하여 만든다. 

ex) `val range = 1..10`

```kotlin
val rangeTest = 1..10
for (i in rangeTest) {
    println(i)
}

for (i in 10 downTo 1 step 2) {      // 10 ≥ i ≥ 1, 2씩 감소
    print("$i ")
}
for (i in 1 until 10) {              // 1 ≤ i < 10
    print("$i ")
}
```

downTo 키워드를 사용해서 범위를 만들 수 있는데 이는 역방향의 범위를 만들어 낸다. 그리고 step 키워드를 사용하여 범위에서 증가 값을 바꿀 수 있다. 

그래서 밑의 식을 실행하면 10 8 6 4 2 가 출력이 된다. 

.. 키워드나 downTo 키워드를 사용하면 끝 값이 포함이 되는데 끝 값을 포함하지 않게 하려면 until 키워드를 사용하여 범위를 정의하면 된다.

<br>

**map**

map은 key-value 쌍을 원소로 갖는 컬렉션이다. 

```kotlin
fun main(args: Array<String>) {
    val binaryReps = TreeMap<Char, String>()

    for (c in 'A'..'F') {
        val binary = Integer.toBinaryString(c.toInt())
        binaryReps[c] = binary
    }

    for ((letter, binary) in binaryReps) {
        println("$letter = $binary")
    }
}
```

첫 번째 for문을 통해서 binaryReps 맵의 key로 A ~ F의 Char가 들어가고, value 값으로 해당 char의 2진 표현이 들어간다. 그리고 두 번째 for문의 letter와 binary에 각각 map의 key, value가 할당되어 이터레이션한다.

<br>

**in**

지금까지는 컬렉션이나 범위에 대해서 in을 사용하였는데 in을 통해서 어떤 값이 범위나 컬렉션 내에 있는지 검사를 할 수도 있다. 

```kotlin
fun recognize(c: Char) =
    when(c) {
        in '0'..'9' -> "It's a digit"
        in 'a' .. 'z', in 'A' .. 'Z' -> "It's a letter"
        else -> "What..?"
    }
```

<br>

### **예외 처리**

코틀린의 예외 처리는 다른 언어들의 예외 처리와 비슷하다. 함수는 오류가 발생하면 예외를 throw할 수 있다.

자바와의 차이점은 코틀린의 throw는 식이라서 다른 식에 포함될 수 있다.

<br>

**try, catch, finally**

자바 처럼 예외를 처리할 때 try, catch, finally 절을 사용한다. 

```kotlin
fun readNumber(reader: BufferedReader): Int? {
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    }
    catch (e: NumberFormatException) {
        return null
    }
    finally {
        reader.close()
    }
}
```

위의 코드 처럼 코틀린에서도 예외 처리를 위해서 try, catch, finally 절을 사용할 수 있다. 

근데 자바와의 차이가 있는데 이는 코틀린의 try 키워드는 식이라서 변수 값에 대입할 수 있다. 

```kotlin
fun readNumber(reader: BufferedReader) {
    val num = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        null
    }
}
```

위의 코드는 NumberFormatException 예외가 발생하면 num 변수에 null이 들어가고, 예외가 발생하지 않으면 try 블록 내의 식이 num 변수의 값이 된다.
