코틀린에서는 null이 될 수 있는 타입, 일기 전용 컬렉션을 제공한다. 그리고 자바 타입 시스템에서 불필요하거나 문제가 되는 부분들은 제거하였다.

## 6.1 널 가능성

널 가능성은 NullPointerException (NPE) 오류를 피할 수 있게 한다. 

### 6.1.1 널이 될 수 있는 타입

코틀린 타입 시스템은 널이 될 수 있는 타입을 명시적으로 지원한다. 

먼저 함수에 인자로 널이 들어올 수 없는 경우에는 다음과 같이 함수를 정의한다.

```kotlin
fun strLen(str: String) = str.length
```

이런 함수에 null을 인자로 넘기면 컴파일 시에 오류가 생긴다.

만약 null을 인자로 받을 수 있게 하려면 타입 이름에 물음표(?)를 명시하면 된다.

```kotlin
fun strLen(str: String?) = ...
```

물음표(?)가 명시된 타입에만 null이 들어갈 수 있다. 그래서 기본적으로 모든 타입이 null이 될 수 없다. 

nullable 타입인 변수에는 `변수.메서드` 로 직접적으로 메서드를 호출할 수 없다.

```kotlin
fun strLen(str: String?) = str.length

// 실행 결과
// Kotlin: Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type String?
```

nullable 값을 non-nullable에 넣을 수 없고 nullable 값을 non-nullable 인자로 전달할 수도 없다. 

```kotlin
val x: String? = null
val y: String = x

// 실행 결과
// Kotlin: Type mismatch: inferred type is String? but String was expected
```

nullable 타입을 사용하려면 컴파일러가 해당 변수가 확실히 null이 아닌 것을 알게 하면 된다. 이렇게 하기 위해서는 null과 비교하면 된다. 

```kotlin
fun strLen(str: String?): Int =
    if (str != null)
        str.length
    else
        0

println(strLen(null))
println(strLen("abc"))

// 실행 결과
// 0
// 3
```

### 6.1.2 타입의 의미

자바의 타입 시스템은 null을 제대로 다룰 수 없다. 특정 변수에 대한 널 여부 검사를 하지 않으면 해당 변수가 연산을 수행할 수 있는지 알 수 없다. 특정 변수가 특정한 위치에서 절대 null이 될 수 없다는 확신을 가지고 검사를 생략하는 경우가 있는데 만약 확신이 틀리면 NPE 오류가 발생한다. 

코틀린의 nullable 타입은 이런 문제들의 종합적인 해결책이다. 

### 6.1.3 안전한 호출 연산자: ?.

`?.` 연산자를 사용하면 변수의 null 여부 검사와 호출을 동시에 표현할 수 있다. 

```kotlin
fun printAllCaps(s: String?) {
    val allCaps: String? = s?.uppercase()
    println(allCaps)
}

fun printAllCapsIf(s: String?) {
    val allCaps: String? = if(s != null) s.uppercase() else null
    println(allCaps)
}

printAllCaps("abc")
printAllCaps(null)

printAllCapsIf("abc")
printAllCapsIf(null)

// 실행 결과
// ABC
// null
// ABC
// null
```

`s?.uppercase()` 와 `if(s != null) s.uppercase() else null` 이 같은 결과를 반환하는 것을 확인할 수 있다.

만약 객체 그래프에서 nullable한 중간 객체가 여러 개 있는 경우 한 식 안에서 안전한 호출을 연쇄해서 함께 사용하면 편할 때가 자주 있다.

```kotlin
class Address(val streetAddress: String, val zipCode: Int, 
              val city: String, val country: String)
class Company(val name: String, val address: Address?)
class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
    val country = this.company?.address?.country       // 안전한 호출 연쇄 사용
    return if (country != null) country else "Unknown" 
}

val address = Address("경기도", 3, "런던", "대한민국")
val company1 = Company("조범준", null)
val company2 = Company("조뱀준", address)
val person1 = chap6.Person("메카범준", company1)
val person2 = chap6.Person("메카뱀준", company2)

println(person1.countryName())
println(person2.countryName())

// 실행 결과
// Unknown
// 대한민국
```

안전한 호출을 연쇄적으로 사용하면 복잡해 보이는 null 검사를 한 줄에 처리할 수 있다.

### 6.1.4 엘비스 연산자: ?:

엘비스 연산자 (?:)는 null 대신 사용할 디폴트 값을 지정할 때 사용한다. 

`foo ?: bar` 코드에서 만약 foo가 null이면 bar가 결과가 되고 null이 아니면 foo가 결과가 된다.

```kotlin
fun strLenSafe(s: String?): Int = s?.length ?: 0
fun strLenSafeIf(s: String?): Int {
    return if (s != null)
        s.length
    else
        0
}

println(strLenSafe("abc"))
println(strLenSafe(null))

println(strLenSafeIf("abc"))
println(strLenSafeIf(null))

// 실행 결과
// 3
// 0
// 3
// 0
```

엘비스 연산자를 사용했을 때와 if문을 사용하였을 때와 결과가 같은 것을 확인할 수 있다.

코틀린에서는 return이나 throw 등의 연산도 식이다. 그래서 엘비스 연산자의 우항으로 들어갈 수 있다. 이를 이용하면 엘비스 연산자의 좌항이 null인 경우 예외를 발생하게 할 수 있는데 함수의 전제 조건을 확인할 때 유용하다.

```kotlin
class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)
class Company(val name: String, val address: Address?)
class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
    val country = this.company?.address?.country 
      ?: throw IllegalArgumentException("Exception")
    return if (country != null) country else "Unknown"
}

val address = Address("경기도", 3, "런던", "대한민국")
val company1 = Company("조범준", null)
val company2 = Company("조뱀준", address)
val person1 = chap6.Person("메카범준", company1)
val person2 = chap6.Person("메카뱀준", company2)

println(person2.countryName())
println(person1.countryName())

// 실행 결과
// 대한민국
// Exception in thread "main" java.lang.IllegalArgumentException: Exception
```

### 6.1.5 안전한 캐스트 as?

as 연산자로 타입 캐스트를 할 때 만약 캐스트할 수 없다면 예외가 발생한다. 캐스트를 하기 전에 is 연산자로 캐스트가 가능한지 미리 확인해볼 수 있다.

`as?` 연산자를 사용하면 먼저 확인하지 않아도 된다. `as?` 연산자는 대상 타입으로 변환이 불가능 하면 null을 반환한다. 

안전한 캐스트를 사용할 때 일반적인 패턴은 변환을 하고, 뒤에 엘비스 연산자를 사용하는 것이다. 

### 6.1.6 널 아님 단언: !!

널 아님 단언 (!!)을 사용하면 어떤 값이든 non-nullable로 바꾼다. 만약 null인 값에 사용하면 NPE이 발생한다. 

`foo!!` 의 경우 만약 foo가 null이 아닌 경우 결과는 foo가 되고 foo가 null인 경우 NPE가 발생한다. 

<aside>
💡 !! 연산자는 한줄에 하나만 사용하는게 좋다.

</aside>

### 6.1.7 let 함수

let 함수를 안전한 호출 연산자 (?.)와 사용하면 원하는 식을 평가해서 검사가 null인지 검사하고 그 결과를 변수에 넣는 작업을 간단하게 할 수 있다. 

흔하게 사용되는 경우는 null이 될 수 있는 값을 null이 될 수 없는 값만 인자로 받는 함수에 넘겨줄 때이다. 

```kotlin
fun sendEmail(email: String) {
    println("email sending to $email ...")
}

var email: String? = "beeeam@example.com"
    email?.let {
        sendEmail(it)
    }

var emailNull: String? = null
    emailNull?.let {
        sendEmail(it)
    }

// 실행 결과
// email sending to beeeam@example.com...
```

### 6.1.8 나중에 초기화할 프로퍼티

non-nullable 프로퍼티는 사용되기 전에 초기화 되어야 한다. 만약 초기화할 값을 제공할 수 없으면 nullable 타입으로 선언해야 한다. 이러한 경우 해당 프로퍼티를 사용하려면 null이 아님을 단언해야한다. 

lateinit을 사용하면 이런 불편한 점을 해결할 수 있다. lateinit을 사용하면 프로퍼티의 초기화를 나중에 할 수 있다. lateinit 프로퍼티는 항상 var여야 한다. 밑과 같이 사용하면 된다.

```kotlin
class MyTest {
    private lateinit var lazyprop: String
}
```

### 6.1.9 널이 될 수 있는 타입 확장

nullable 타입의 객체에 확장 함수를 선언하여 안전한 호출(?.)을 사용하지 않아도 되는 상황을 만들 수 있다.

```kotlin
var abc: String? = "qwer"

fun checkABC() {
    abc.isNullOrBlank()
}
```

isNullOrBlank() 함수는 확장 함수로 객체의 null 여부를 먼저 판단하고 null이 아닌 경우 isBlank() 함수를 반환한다. 그래서 위의 예시에서 프로퍼티 abc에 null을 넣어도 오류가 발생하지 않는다. 

### 6.1.10 타입 파라미터의 널 가능성

코틀린에서는 함수나 클래스의 모든 타입 파라미터는 기본적으로 null이 될 수 있다. nullable 타입을 포함하는 어떤 타입이라도 타입 파라미터를 대신할 수 있다. 그래서 타입 파라미터 T를 클래스나 함수 안에서 타입 이름으로 사용하면 물음표를 붙이지 않아도 T는 널이 될 수 있는 타입이 된다.

```kotlin
fun <T>printHashCode(t: T) {
    println(t?.hashCode())
}

printHashCode(null)

// 실행 결과
// null
```

타입 파라미터가 null이 아님을 확실히 하려면 null이 될 수 없는 타입 상한을 지정하면 된다.

```kotlin
fun <T: Any>printHashCode(t: T) {
    println(t.hashCode())
}

printHashCode(null)

// 실행 결과
// Kotlin: Null can not be a value of a non-null type TypeVariable(T)
```

### 6.1.11 널 가능성과 자바

코틀린은 자바와의 상호운용성이 장점 중 하나이다. 자바의 타입 시스템은 nullable을 지원하지 않는다. 만약 코틀린과 자바를 같이 사용하면 어떻게 될까?

**애노테이션**

자바 코드에는 애노테이션으로 표시된 null 가능성 정보가 있다. 

<aside>
💡 @Nullable String == String? <br>
💡 @NotNull String == String
<br>
</aside>
<br>

**플랫폼 타입**

플랫폼 타입은 코틀린이 null 관련 정보를 알 수 없는 타입을 말한다. 그 타입을 nullable 또는 non-nullable 타입으로 처리하여도 된다. 대신 null 인지 확인을 잘 해야 한다. 컴파일러가 모든 연산을 허용하기 때문에 null 값이 전달이 되면 NPE가 발생할 수 있다.

**상속**

자바 클래스나 인터페이스를 코틀린에서 구현하는 경우 null 가능성을 제대로 처리하는 것은 중요하다. 구현 메서드를 다른 코틀린 코드가 호출할 수 있기 때문에 컴파일러는 non-nullable 타입으로 선언한 모든 파라미터에 대해 null 여부 검사를 한다. 

```kotlin
// 자바
interface StringProcessor {
    void process(String value);
}

// 코틀린(구현)
class NullableStringPrinter: StringProcessor {
    override fun process(value: String?) {
        if (value != null)
            println(value)
    }
}
```

## 6.2 코틀린의 원시 타입

Int, Boolean, Any 등은 원시 타입이라고 부른다. 코틀린은 원시 타입과 래퍼 타입을 구분하지 않는다.

### 6.2.1 원시 타입: Int, Boolean 등

자바에서는 원시 타입과 래퍼 타입을 구분하였다. 원시 타입은 변수에 그 값이 직접 들어가고 래퍼 타입은 변수에 메모리 상의 객체 위치가 들어가는 차이가 있다. 

코틀린에서는 둘을 구분하지 않는다. 그래서 Int 타입이면 컬렉션을 만들 때도 Int에 컬렉션을 감싸서 만든다. 

```kotlin
val i: Int = 1
val listInt: List<Int> = listOf(1, 2, 3)
```

코틀린의 Int 타입은 자바의 int 타입으로 컴파일이 된다. 근데 Int 타입을 컬렉션의 타입 파라미터로 넘기면 래퍼 타입인 java.lang.Integer 객체가 들어간다. 

### 6.2.2 널이 될 수 있는 원시 타입: Int?, Boolean? 등

null은 래퍼 타입에만 들어갈 수 있다. 그래서 코틀린에서 nullable 원시 타입을 사용하면 자바의 래퍼 타입으로 컴파일 된다. 

```kotlin
data class Person(
    val name: String,
    val age: Int?
) {
    fun isOlderThan(other: Person): Boolean? {
        if (age == null || other.age == null)
            return null
        else
            return age > other.age
    }
}
```

nullable 타입의 두 변수는 null이 될 수 있기 때문에 바로 비교 연산을 할 수 없다. 그래서 먼저 null 검사를 하고 null이 아님을 단언한 후에 연산을 할 수 있다.

### 6.2.3 숫자 변환

코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않는다. 그래서 밑의 코드는 오류가 발생한다.

```kotlin
val i: Int = 1
val l: Long = i
```

이를 가능하게 하려면 직접 변환 메서드를 호출해야 한다.

```kotlin
val i: Int = 1
val l: Long = i.toLong()
```

코틀린은 모든 원시 타입에 대한 변환 함수를 제공한다. `to원시타입()` 이런 형식으로 제공된다.

코틀린은 타입 변환을 명시한다. 그래서 변수들끼리 연산을 하는 경우 명시적으로 타입을 변환하여 같은 타입으로 만든 다음 연산을 해야한다.

```kotlin
// 오류 나는거 
val x = 1
val listLong = listOf(1L, 2L, 3L)
println(x in listLong)

// 올바른 문법
val x = 1
val listLong = listOf(1L, 2L, 3L)
println(x.toLong() in listLong)
```

### 6.2.4 Any, Any?: 최상위 타입

코틀린에서는 Any 타입이 모든 non-nullable 타입의 조상 타입이다. 원시 타입 값을 Any 타입의 변수에 대입하면 자동으로 값을 객체로 감싼다.

```kotlin
val answer: Any = 42
println(answer::class.java)

// 실행 결과
// class java.lang.Integer
```

Any 타입의 변수에는 null이 들어갈 수 없다. 만약 null을 넣고 싶으면 Any? 타입을 사용하면 된다.

### 6.2.5 Unit 타입: 코틀린의 void

코틀린의 Unit 타입은 자바의 void와 같은 기능을 한다. 

```kotlin
fun f(): Unit { ... }

fun f() { ... }
```

코틀린의 Unit과 자바의 void의 다른 점은 Unit은 타입 인자로 사용할 수 있다. Unit 타입의 함수는 Unit 값을 묵시적으로 반환한다. 

### 6.2.6 Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다.

코틀린에는 반환 값의 개념 자체가 없는 의미 없는 함수가 일부 존재한다. 함수를 호출하는 코드를 분석하는 경우 함수가 정상적으로 끝나지 않는 다는 것을 알면 유용하다. 이런 표현을 위해서 코틀린에는 Nothing이라는 반환 타입이 있다.

```kotlin
fun fail(msg: String): Nothing {
    throw IllegalStateException(msg)
}

fail("Error occurred")

// 실행 결과
// java.lang.IllegalStateException: Error occurred
```

Nothing 타입은 아무 값도 포함하지 않는다. 그래서 함수의 반환 타입이나 반환 타입으로 사용될 타입 파라미터로만 사용할 수 있다.

## 6.3 컬렉션과 배열

### 6.3.1 널 가능성과 컬렉션

컬렉션 의 타입 인자에 물음표(?)를 붙이면 변수에서 처럼 null을 저장할 수 있게 된다. 

```kotlin
val result = ArrayList<Int?>()
for (line in reader.lineSequence()) {
    try {
        val num = line.toInt()
        result.add(num)
    }
    catch (e: NumberFormatException) {
        result.add(null)
    }
}
```

List<Int?>는 리스트의 원소로 Int 또는 null을 담을 수 있다. 그리고 List<Int>?는 전체 리스트가 null이 될 수 있기 때문에 조심해서 선언해야 한다.

nullable 값으로 이뤄진 컬렉션의 원소를 사용하려면 null 여부 검사를 해야 한다. 코틀린은 이러한 작업을 지원하는 함수를 제공하는데 filterNotNull() 함수이다. 

### 6.3.2 읽기 전용과 변경 가능한 컬렉션

코틀린 컬렉션은 자바 컬렉션과 달리 컬렉션 안의 데이터에 접근하는 인터페이스와 컬렉션 안의 데이터를 변경하는 인터페이스를 분리했다. 

Collection 인터페이스는 데이터를 읽는 여러 메서드가 있지만 수정하거나 제거하는 메서드는 없다. 수정이나 제거하려면 Mutable Collection 인터페이스를 사용하면 된다.

### 6.3.3 코틀린 컬렉션과 자바

코틀린은 자바 컬렉션 클래스들이 코틀린의 변경 가능한 인터페이스를 상속한 것처럼 취급한다. 이를 통해서 코틀린은 자바 호환성을 제공하면서 읽기 전용 인터페이스, 변경 가능 인터페이스를 분리 하였다. 

### 6.3.4 컬렉션을 플랫폼 타입으로 다루기

자바에서 선언한 컬렉션 또한 코틀린에서는 플랫폼 타입으로 본다. 그래서 읽기 전용 또는 변경 가능한 컬렉션 어느 쪽으로든 다룰 수 있다. 

하지만 컬렉션 타입이 시그니처에 들어간 자바 메서드 구현을 오버라이드 하려는 경우 두 타입의 차이가 문제가 된다. 그래서 읽기 전용 컬렉션 또는 변경 가능 컬렉션 둘 중 하나의 타입으로 표현할 지 결정해야 한다. 

- 컬렉션이 null이 될 수 있는가?
- 컬렉션의 원소가 null이 될 수 있는가?
- 오버라이드 하는 메서드가 컬렉션을 변경할 수 있는가?

위와 같은 생각을 하고 상황에 맞게 컬렉션 타입을 결정해야 한다.

```java
interface FileContentProcessor {
    void processContents(File path,
        byte[] binaryContents,
        List<String> textContents);
}
```

위의 자바 인터페이스를 코틀린에서 구현한다고 가정하자. 그러면 밑과 같은 선택을 해야 한다.

- 일부 파일은 이진 파일이며 이진 파일 안의 내용은 텍스트로 표현할 수 없으므로 리스트는 null이 될 수 있다.
- 파일의 각 줄은 null일 수 없으므로 이 리스트의 원소는 null이 될 수 없다.
- 이 리스트는 파일의 내용을 표현하며 그 내용을 바꿀 필요가 없으므로 읽기 전용이다.

이를 코틀린으로 구현하면 밑과 같다.

```kotlin
class FileIndexer: FileContentProcessor {
    override fun processContents(path: File, 
        binaryContents: ByteArray?, 
        textContents: List<String>?) {
        // ...
    }
}
```

### 6.3.5 객체의 배열과 원시 타입의 배열

코틀린 배열은 타입 파라미터를 받는 클래스이다.  코틀린에서 배열을 만드는 방법은 다양하다.

- arrayOf() 함수에 원소를 넣으면 배열을 만들 수 있다.
- arrayOfNull() 함수에 정수 값을 넘기면 모든 원소가 null이고, 인자로 넘긴 값과 크기가 같은 배열을 만들 수 있다.
- Array 생성자는 배열 크기와 람다를 인자로 받아서 람다를 호출하여 각 배열의 원소를 초기화 한다. arrayOf 함수를 사용하지 않고 각 원소가 null이 아닌 배열을 만들어야 하는 경우 Array 생성자를 사용한다.
