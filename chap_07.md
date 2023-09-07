어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법을 코틀린에서는 **관례**라고 부른다. 이번 장에서는 코틀린이 지원하는 여러 관례와 사용법에 대해서 볼 수 있다.

자바는 언어 기능을 타입에 의존하는 반면에 코틀린은 관례에 의존한다. 이러한 이유는 기존의 자바 클래스를 코틀린 언어에 적용하기 위해서다. 기존 자바 클래스가 구현하는 인터페이스는 이미 고정이 되어 있지만 확장 함수를 통해서 정의하면 기존의 자바 코드를 바꾸지 않고 새로운 기능을 추가할 수 있게 된다. 

## 7.1 산술 연산자 오버로딩

산술 연산자를 다른 클래스에 사용하면 유용한 경우가 있다. BigInteger 클래스를 다룰 때나 컬렉션에 원소를 추가하는 경우 add를 명시적으로 사용하는 경우보다 +연산자를 사용하면 좋은 경우가 있다.

### 7.1.1 이항 산술 연산 오버로딩

```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}

val p1 = Point(1, 2)
val p2 = Point(2, 4)
println(p1 + p2)

// 실행 결과
// Point(x=3, y=6)
```

plus 함수를 따로 정의하여 Point 객체의 각 값의 합을 반환하는 함수를 구현하였다.

여기서 plus 함수에는 operator 키워드가 붙어야 한다. 이는 plus 함수가 연산자를 오버로딩하는 함수이기 때문이다. operator 키워드를 붙이면 해당 함수가 관례를 따르고 있음을 알릴 수 있다.

위와 같이 멤버 함수로 선언할 수도 있고 다음과 같이 **확장 함수**로 정의할 수도 있다. 

```kotlin
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}

val p1 = Point(1, 2)
val p2 = Point(2, 4)
println(p1 + p2)

// 실행 결과
// Point(x=3, y=6)
```

외부 함수의 클래스에 대한 연산자를 정의할 때는 관례를 따르는 이름의 확장 함수로 구현하는 게 일반적인 패턴이다. 

코틀린에서는 프로그래머가 직접 연산자를 만들어 사용할 수 없고, 미리 정해진 연산자만 오버로딩할 수 있다. 그리고 관례에 따르기 위해 클래스에서 정의해야 하는 이름이 연산자 별로 정해져 있다. 

| a * b | times |
| --- | --- |
| a / b | div |
| a % b | mod |
| a + b | plus |
| a - b | minus |

```kotlin
operator fun Point.minus(other: Point): Point {
    return Point(x - other.x, y - other.y)
}

operator fun Point.times(other: Point): Point {
    return Point(x * other.x, y * other.y)
}

operator fun Point.div(other: Point): Point {
    return Point(x / other.x, y / other.y)
}

println(p1 - p2)
println(p1 * p2)
println(p1 / p2)
```

operator 함수는 일반 함수와 마찬가지로 오버로딩 할 수 있다. 그래서 이름은 같지만 파라미터 타입이 서로 다른 연산자 함수를 여럿 만들 수 있다. 

### 7.1.2 복합 대입 연산자 오버로딩

plus와 같은 연산자를 오버로딩하면 코틀린은 + 연산자뿐 아니라 그와 관련 있는 연산자인 +=도 자동으로 지원한다. +=, -= 등의 연산자는 **복합 대입 연산자**라고 불린다. 

```kotlin
var point = Point(1, 2)
point += Point(3, 4)

println(point)

// 실행 결과
// Point(x=4, y=6)
```

+=나 -= 같은 복합 대입 연산자는 함수 이름에 Assign을 붙여서 정의할 수 있다.

```kotlin
operator fun<T> MutableCollection<T>.plusAssign(element: T) {
    this.add(element)
}

val list = arrayListOf(1, 2)
list += 3

// 실행 결과
// [1, 2, 3]
```

### 7.1.3 단항 연산자 오버로딩

이항 연산자처럼 단항 연산자도 오버로딩하여 정의할 수 있다.

```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

val p = Point(10, 20)
println(-p)

// 실행 결과
// Point(x=-10, y=-20)
```

단항 연산자를 오버로딩 하기 위해 사용하는 함수는 인자가 필요하지 않다. 

| +a | unaryPlus |
| --- | --- |
| -a | unaryMinus |
| !a | not |
| ++a, a++ | inc |
| --a, a-- | dec |

```kotlin
operator fun BigDecimal.inc() = this + BigDecimal.ONE

var bd = BigDecimal.ZERO
println(bd++)
println(++bd)

// 실행 결과
// 0
// 2
```

## 7.2 비교 연산자 오버로딩

### 7.2.1 동등성 연산자: equals

코틀린은 == 연산자 호출을 equals 메서드 호출로 컴파일한다. != 연산자를 호출하는 것도 equals 메서드 호출로 컴파일된다. 

==, != 연산자는 내부에서 null 검사를 하기 때문에 nullable 값에도 적용할 수 있다. 그래서 a == b를 실행하면 먼저 a가 null 인지 검사하고 null이 아닌 경우에 a.equals(b)를 호출한다. 

data class이면 자동으로 컴파일러가 equals를 생성해준다. 만약 직접 equals를 구현하면 다음과 비슷하게 구현할 수 있다.

```kotlin
override fun equals(obj: Any?): Boolean {
    if (obj === this) return true
    if (obj !is Point) return false
    return obj.x == x && obj.y == y
}
```

처음 if문에서 ===(식별자 비교) 연산자를 사용해서 두 변수가 동일한 객체인지 확인한다. === 연산자는 자신의 두 피연산자 서로 같은 객체를 가리키고 있는지 비교하는 연산자이다.  === 연산자는 오버로딩을 할 수 없다.

equals 함수는 Any에 정의된 메서드이기 때문에 override가 필요하다. 

### 7.2.2 순서 연산자: compareTo

자바에서 값을 비교하는 알고리즘을 사용하려면 Comparable 인터페이스를 구현해야 한다. 코틀린에서도 Comparable 인터페이스를 제공하는데 비교 연산자(>, <, >=, <=)를 호출하면 Comparable 인터페이스 내의 compareTo() 메서드를 호출하는 관례를 제공한다. 

```kotlin
class Person(val firstName: String, val lastName: String): Comparable<chap7.Person> {
    override fun compareTo(other: Person): Int {
        return compareValuesBy(this, other, Person::lastName, Person::firstName)
    }
}

val ps1 = Person("Alice", "Smith")
val ps2 = Person("Bob", "Johnson")
println("ps1 < ps2 = ${ps1 < ps2}")

// 실행 결과
// ps1 < ps2 = false
```

코틀린의 compareValuesBy() 함수는 두 객체와 여러 비교 함수를 인자로 받는다. 첫 번째 비교 함수에 두 객체를 넘겨 두 객체가 같지 않다는 결과가 나오면 그 결과를 즉시 반환하고, 두 객체가 같다는 결과가 나오면 두 번째 비교 함수를 통해 두 객체를 비교한다.

## 7.3 컬렉션과 범위에 대해 쓸 수 있는 관례

### 7.3.1 인덱스로 원소에 접근: get과 set

인덱스 연산자를 사용하여 컬렉션의 원소를 읽어오는 동작은 get() 메서드로 반환되고, 원소를 쓰는 동작은 set() 메서드로 반환된다. 

```kotlin
operator fun Point.get(index: Int): Int {
    return when(index) {
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

operator fun MutablePoint.set(index: Int, value: Int) {
    return when(index) {
        0 -> x = value
        1 -> y = value
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

val pointIndex = Point(10, 20)
println(pointIndex[1])

val mp = MutablePoint(3, 5)
mp[0] = 5
println(mp)

// 실행 결과
// 20
// MutablePoint(x=5, y=5)

```

### 7.3.2 in 관례

in은 객체가 컬렉션에 들어있는지 확인하는 연산자이다. 이런 경우 in과 대응되는 함수는 contains 함수다. a in c는 c.contains(a)로 변환된다.

```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point)
operator fun Rectangle.contains(p: Point): Boolean {
    return p.x in upperLeft.x until lowerRight.x && 
            p.y in upperLeft.y until lowerRight.y
}

val rect = Rectangle(Point(10, 20), Point(50, 50))
println(Point(20, 30) in rect)
println(Point(5, 5) in rect)

// 실행 결과
// true
// false
```

### 7.3.3 rangeTo 관례

코틀린에서 범위를 만들기 위해서는 .. 구문을 사용해야 한다. 1..10 은 1 ~ 10 까지의 수가 들어있는 범위를 나타낸다. 

rangeTo() 함수는 .. 연산자와 대응된다. .. 연산자는 아무 클래스에나 정의 할 수 있지만 어떤 클래스가 Comparable 인터페이스를 구현하면 rangeTo() 함수를 구현할 필요가 없다. 

### 7.3.4 for 루프를 위한 iterator

코틀린의 for 루프는 in 연산자를 사용한다. 이때 in 연산자는 앞에서 사용했던 것과는 의미가 다르다. 

for (i in List) { … } 와 같은 문장은 list.iterator()를 호출해서 이터레이터를 얻은 다음, 그 이터레이터에 대해서 hasNext와 next 호출을 반복하는 식으로 변환된다. 

코틀린에서는 이러한 것도 관례라서 iterator 함수를 확장하여 정의할 수 있다. 

## 7.4 구조 분해 선언과 component 함수

구조 분해를 사용하면 여러 변수를 한번에 초기화 할 수 있다. 구조 분해는 다음과 같이 사용할 수 있다.

```kotlin
val p = MutablePoint(10, 20)
val (a, b) = p

println(a)
println(b)

// 실행 결과
// 10
// 20
```

일반 변수처럼 = 연산자를 사용하지만 좌변의 여러 변수를 괄호() 로 묶는 차이점이 있다. 구조 분해 선언은 관례를 사용하는데 componentN() 함수를 사용한다. 이때 N은 선언에 있는 변수 위치에 따라 붙는 번호다.

val (a, b) = p 왼쪽의 식은 val a = p.component1(), val b = p.component2() 이렇게 변환이 된다.

data class는 자동으로 componentN() 함수를 생성해주지만 일반 클래스에서는 구현해야 한다. 이는 다음과 같이 구현할 수 있다.

```kotlin
class Point(val x: Int, val y: Int) {
    operator fun component1() = x
    operator fun component2() = y
}
```

### 7.4.1 구조 분해 선언과 루프

구조 분해 선언은 함수 본문 내의 선언문 뿐만 아니라 변수 선언이 들어갈 수 있는 어느 위치에서든 사용 가능하다. 

```kotlin
fun printEntries(map: Map<String, String>) {
    for ((key, value) in map) {
        println("$key -> $value")
    }
}
```

위와 같이 루프 안에서도 구조 분해 선언을 사용할 수 있다. 이 예제에서는 2개의 관례가 사용된다. 하나는 객체를 이터레이션 하는 관례이고 하나는 구조 분해 선언이다. 코틀린 표준 라이브러리에는 map에 대한 확장 함수로 iterator가 정의 되어 있기 때문에 map을 직접 이터레이션 할 수 있다.

## 7.5 프로퍼티 접근자 로직 재활용: 위임 프로퍼티

위임 프로퍼티를 사용하면 값을 뒷받침하는 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 작동하는 프로퍼티를 쉽게 구현할 수 있다. 이를 사용하여 값을 DB 테이블이나 브라우저, 세션, 맵 등에 저장할 수 있다. 

여기서 위임은 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 것이다. 

### 7.5.1 위임 프로퍼티 소개

위임 프로퍼티의 일반적인 문법은 다음과 같다.

```kotlin
class Foo {
    var p: Type by Delegate()
}
```

p 프로퍼티는 접근자 로직을 다른 객체에 위임한다. 그리고 Delegates 클래스의 인스턴스를 위임 객체로 사용한다. 

컴파일러는 숨겨진 도우미 프로퍼티를 만들고 그 프로퍼티를 위임 객체의 인스턴스로 초기화 한다. 그리고 p 프로퍼티는 그 프로퍼티에 자신의 작업을 위임한다. 

프로퍼티 위임 관례를 따르는 Delegate 클래스는 getValue, setValue 함수를 제공해야 한다. 

```kotlin
class Delegate {
    operator fun getValue(...) { ... }

    operator fun setValue(..., value: Type) { ... }
}

class Foo {
    var p: Type by Delegate()
}

val foo = Foo()
val oldValue = foo.p
foo.p = newValue
```

foo.p는 언뜻 보면 일반 프로퍼티 같지만 실제로는 Delegate의 getValue, setValue를 사용한다. 

### 7.5.2 위임 프로퍼티 사용: by lazy()를 사용한 프로퍼티 초기화 지연

지연 초기화는 객체의 일부분을 초기화 하지 않고 남겨두다가 그 부분의 값이 실제로 필요한 경우 초기화 할 때 사용되는 패턴이다. 초기화 과정에서 자원을 많이 사용하거나 객체를 사용할 때마다 꼭 초기화 하지 않아도 되는 프로퍼티에 대해서 사용할 수 있다. 

by lazy()를 사용하지 않아도 지연 초기화를 구현할 수 있다. 하지만 코틀린에서는 위임 프로퍼티를 사용하여 쉽게 구현 가능하다. 

```kotlin
class Person(val name: String) {
    val emails by lazy { loadEmails(this) }
}
```

### 7.5.4 위임 프로퍼티 컴파일 규칙

```kotlin
class C {
    var prop: Type by MyDelegate()
}

val c = C()
```

위의 코드를 실행하면 컴파일러는 다음과 같은 코드를 생성한다.

```kotlin
class C {
    private val <delegate> = MyDelegate()
    var prop: Type
    get() = <delegate>.getValue(this, <property>)
    set(value: Type) = <delegate>.setValue(this, <property>, value)           
}
```

컴파일러는 MyDelegate() 클래스의 인스턴스를 감춰진 프로퍼티에 저장하고 프로퍼티를 사용하기 위해서 KProperty 타입의 객체를 사용한다. 이 객체가 <property>이다.

### 7.5.5 프로터피 값을 맵에 저장

프로퍼티를 동적으로 정의할 수 있는 프로퍼티로 만들 때 위임 프로퍼티를 활용하는 경우가 있는데 이런 객체를 확장 가능한 객체라고 부르기도 한다. 

```kotlin
class Person {
    private val _attributes = hashMapOf<String, String>()
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }
    
    val name: String
        get() = _attributes["name"]!!      // 수동으로 맵에서 정보 꺼냄
}
```

위의 Person 클래스는 name 프로퍼티를 처리하기 위해서 개별 API를 제공하고 있다. 근데 위임 프로퍼티를 사용하면 이를 쉽게 만들 수 있다.

```kotlin
class Person {
    private val _attributes = hashMapOf<String, String>()
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }

    val name: String by _attributes    // 위임 프로퍼티로 맵 사용
}
```

이렇게 바꿀 수 있는 이유는 표준 라이브러리가 Map과 MutableMap 인터페이스에 대해 getValue, setValue 확장 함수를 제공하기 때문이다. getValue에서 맵에 프로퍼티 값을 저장할 때는 자동으로 프로퍼티 이름을 키로 활용한다.
