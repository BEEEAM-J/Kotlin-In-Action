## 3.1 코틀린에서 컬렉션 만들기

코틀린은 자신만의 컬렉션 기능을 제공하지 않는다. 이러한 이유는 표준 자바 컬렉션을 활용하면 자바 코드와 상호작용하기 더 쉽기 때문이다. 

코틀린 컬렉션은 자바 컬렉션과 같은 클래스이지만 자바보다 더 많은 기능을 사용할 수 있다. 

```kotlin
val nums = listOf(1, 14, 2)

println(nums.last())
println(nums.max())
```

위의 코드처럼 listof() 함수를 사용하여 컬렉션을 만들 수 있고, last(), max() 함수를 사용하여 컬렉션의 마지막 값, 최대 값을 구할 수 있다. 

## 3.2 함수를 호출하기 쉽게 만들기

자바 컬렉션은 디폴트 toString 구현이 들어있다.

```kotlin
val nums = listOf(1, 14, 2)

println(nums)

// 출력 결과
// [1, 14, 2]
```

그래서 list를 그대로 print 해도 잘 출력이 된다.

```kotlin
fun<T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String
): String {
    val res = StringBuilder(prefix)
    for ((idx, element) in collection.withIndex()) {
        if (idx > 0)res.append(separator)
        res.append(element)
    }

    res.append(postfix)
    return res.toString()
}

val list = listOf(1, 2, 3)
println(joinToString(list, "; ", "(", ")"))

// 출력 결과
// (1; 2; 3)
```

위 함수는 인자가 4개 필요한 함수이다. 실행하는데 큰 문제는 없지만 함수를 호출할 때 마다 매번 4개의 인자를 전달해야 할까? 그리고 호출하는 문장을 간단하게 만들 수 있을까? 밑에서 질문들의 답을 찾아볼 수 있다. 

### 3.2.1 이름 붙인 인자

```kotlin
joinToString(collection, “ “, “ “, “.”) 
```

이처럼 함수를 호출하면 각 인자들이 어떤 역할을 하는지 알기 힘들다. 함수의 본문을 봐야 알 수 있다. 

코틀린에서는 함수를 호출할 때 함수의 인자 중 인부(또는 전부)의 이름을 명시할 수 있다.

```kotlin
joinToString(list, separator = " ", prefix = " ", postfix = " ")
```

위와 같이 인자에 이름을 붙이면 그 뒤의 모든 인자들에 이름을 꼭 명시해줘야 혼동을 막을 수 있다. 

### 3.2.2 디폴트 파라미터 값

코틀린에서는 함수 선언할 때 파라미터의 디폴트 값을 지정할 수 있다. 

```kotlin
fun<T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    ...
}

println(joinToString(list))
println(joinToString(list, "; "))
println(joinToString(list, separator = "; "))

// 출력 결과
// 1, 2, 3
// 1; 2; 3
// 1; 2; 3
```

이렇게 하면 매번 함수의 인자를 4개씩 전달할 필요가 없다. 그리고 이름을 붙인 인자를 사용하는 경우 인자 목록의 중간을 생략하고 지정하고 싶은 인자를 이름 붙여서 순서와 관계 없이 인자를 전달할 수 있다.

함수의 디폴트 파라미터 값은 함수의 선언 부분에서 지정이 된다. 함수를 호출하는 시점에서 전달 받지 못한 인자들은 디폴트 값들로 적용이 된다.

### 3.2.3 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티

코틀린에서는 클래스 밖에 함수를 선언할 수 있다. 해당 함수를 사용하려면 정의된 패키지를 import 해야 한다. 하지만 소스 파일명은 필요하지 않다. 

이는 코틀린 컴파일러가 생성하는 클래스 명이 코틀린 소스 파일 명으로 되기 때문이다. 그래서 코틀린에서 최상위 함수는 해당 클래스의 정적인 메서드가 된다.

ex) join.kt (Kotlin)

```kotlin
package strings

fun joinToString(...): String {...}
```

ex) join.kt (Java)

```kotlin
public class JoinKt {
    public static String joinToString(...) {...}
}
```

최상위 함수가 포함되는 클래스의 이름을 바꾸려면 @JvmName 어노테이션을 파일의 맨 앞에 지정한다.

```kotlin
@file:JvmName("StringFunctions")         // 클래스 이름 지정하는 어노테이션

package strings

fun joinToString(...): String {...}
```

프로퍼티도 파일의 최상위 수준에 위치할 수 있다. 필요한 경우가 많지 않지만 가끔 유용할 때가 있다. 어떤 연산의 수행 횟수를 저장하는 var 프로퍼티를 만들 수 있다.

```kotlin
var count = 0

fun operation() {
	count++
}
```

최상위 프로퍼티를 활용하면 코드에 상수를 추가할 수 있다. 최상위 프로퍼티도 다른 프로퍼티들 처럼 접근자 메서드를 통해서 자바 코드에 노출이 되는데 이를 getter() 통해서 접근하는 것은 자연스럽지 못하다. 

그래서 const 변경자를 사용하여 public static final 필드로 컴파일하게 만들면 자연스럽게 사용할 수 있다. 

```kotlin
const val UNIX_LINE_SEPARATOR = "\n"
// ==
public static final String UNIX_LINE_SEPARATOR = "\n";
```

## 3.3 메서드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

확장 함수는 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 그 클래스 밖에서 선언된 함수다. 

```kotlin
fun String.lastChar(): Char = this[this.length - 1]

fun 수신 객체 타입.함수명(): 반환 타입 = 수신 객체

println("Kotlin".lastChar())

// 출력 결과
// n
```

확장 함수를 만들려면 추가하려는 함수명 앞에 그 함수가 확장될 클래스의 이름을 붙이면 된다. 클래스의 이름은 **수신 객체 타입**, 확장 함수가 호출되는 대상은 **수신 객체**라고 부른다. 

위의 함수는 String 클래스에 새로운 메서드를 추가하는 것과 같다.

확장 함수에서는 private, protected 속성을 사용할 수 없다. 

### 3.3.1 임포트와 확장 함수

확장 함수를 사용하기 위해서는 import 해야 한다. 확장 함수를 정의 하자마자 어디서든 사용할 수 있다면 한 클래스에서 같은 이름의 확장 함수로 오류가 생길 수 있다. 

```kotlin
import test.lastChar

fun main(args: Array<String>) {
    println("Kotlin".lastChar())
}
```

as 키워드를 사용하면 원래 정의했던 이름이 아닌 다른 이름으로 함수를 부를 수 있다. 

```kotlin
import test.lastChar as getLastChar

fun main(args: Array<String>) {
    println("Kotlin".getLastChar())
}
```

한 파일 안에서 같은 이름의 여러 확장 함수들을 호출하는 경우 위와 같이 이름을 바꿔서 부르는 것이 이름으로 인한 충돌을 해결할 수 있는 유일한 방법이다. 

### 3.3.2 자바에서 확장 함수 호출

내부적으로는 확장 함수는 수신 객체를 첫 번째 인자로 받는 정적 메서드다. 그래서 호출할 때 첫 번째 인자로 수신 객체를 넘기기만 하면 된다. 

```kotlin
char c = StringUtilKt.lastChar("Java");
```

### 3.3.3 확장 함수로 유틸리티 함수 정의

지금까지 봤던 내용들을 활용하여 앞에서 정의했던 joinToString() 함수를 바꿀 수 있다.

```kotlin
fun<T> Collection<T>.joinToString(              // Collection<t>에 대한 확장 함수
    separator: String = ", ",                   // 디폴트 파라미터 값
    prefix: String = "",
    postfix: String = ""
): String {
    val res = StringBuilder(prefix)
    for ((idx, element) in this.withIndex()) {  // 수신 객체 (Collection<T>)
        if (idx > 0)res.append(separator)
        res.append(element)
    }

    res.append(postfix)
    return res.toString()
}

val list = listOf(1, 2, 3)

println(list.joinToString())
println(list.joinToString(separator = "; "))

// 출력 결과
// 1, 2, 3
// 1; 2; 3
```

### 3.3.4 확장 함수는 오버라이드 할 수 없다.

```kotlin
fun View.showOff() = println("View")
fun Button.showOff() = println("Button")

val view: View = Button()
view.showOff()

// 출력 결과
// Button
```

이러한 결과가 나오는 이유는 확장 함수는 오버라이드할 수 없기 때문이다. 확장 함수는 클래스 밖에서 선언되고 코틀린은 호출될 확장 함수를 정적으로 결정하기 때문이다.

### 3.3.5 확장 프로퍼티

확장 프로퍼티는 상태를 저장할 방법이 없어서 아무 상태도 가질 수 없다. 하지만 이를 사용하면 짧게 코드를 작성할 수 있는 장점이 있다. 

확장 프로퍼티에는 뒷받침하는 필드가 없어서 게터를 꼭 정의해야 한다. 

```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1)                                // 프로퍼티 getter()
    set(value: Char) = this.setCharAt(length - 1, value)   // 프로퍼티 setter()

val sb = StringBuilder("Kotlin")
    sb.lastChar = '!'
    println(sb.lastChar)

// 출력 결과
// !
```

## 3.4 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

- vararg 키워드를 사용하면 호출 시 인자 개수가 달라질 수 있는 함수를 정의할 수 있다.
- 중위(infix) 함수 호출 구문을 사용하면 인자가 하나뿐인 메서드를 간편하게 호출할 수 있다.
- 구조 분해 선언을 사용하면 복합적인 값을 분해해서 여러 변수에 나눠 담을 수 있다.

### 3.4.1 자바 컬렉션 API 확장

앞에서 코틀린 컬렉션은 자바 컬렉션과 같은 클래스를 사용하지만 더 확장된 API를 제공한다고 했다. 그 해답은 확장 함수이다. 이 설명을 하면서 예시로 들었던 last(), max() 함수 둘 다 확장 함수이다. 

```kotlin
public fun <T> List<T>.last(): T {
    if (isEmpty())
        throw NoSuchElementException("List is empty.")
    return this[lastIndex]
}
```

위의 코드는 last() 함수이다. 첫 번째 줄을 보면 last() 함수가 List 클래스의 확장 함수인 것을 알 수 있다. 

### 3.4.2 가변 인자 함수: 인자의 개수가 달라질 수 있는 함수 정의

```kotlin
val list = listOf(2, 3, 5, 7, 11)
```

리스트를 생성하는 함수를 호출하면 원하는 만큼의 원소를 갖는 리스트를 만들 수 있다. 

listOf() 함수는 다음과 같다.

```kotlin
public fun <T> listOf(vararg elements: T): List<T> = 
    if (elements.size > 0) elements.asList() 
    else emptyList()
```

파라미터 앞에 vararg 변경자를 붙여서 가변적인 인자를 받는 것을 확인할 수 있다. 

이미 배열에 들어있는 원소를 가변 길이 인자로 넘길 때는 배열을 명시적으로 풀어서 배열의 각 원소가 인자로 전달되게 해야 된다. 이는 스프레드 연산자를 사용하면 되는데 이는 전달하려는 배열 앞에 *를 붙이면 된다.

```kotlin
val list = listOf("args: ", *args)
println(list)
```

### 3.4.3 값의 쌍 다루기: 중위 호출과 구조 분해 선언

맵을 만들려면 mapOf() 함수를 사용한다.

```kotlin
val map = mapOf(1 to "one", 2 to "two", 3 to "three")
```

위와 같이 선언하면 되는데 여기서 to는 키워드가 아니고 중위 호출을 통해서 불려진 일반 메서드다. 

중위 호출을 할 때는 수신 객체와 유일한 메서드 인자 사이에 메서드 이름을 넣는다. 이때 객체, 메서드 이름, 유일한 인자 사이에는 공백이 들어가야 한다.

```kotlin
1.to("one")   // 일반적인 호출
1 to "one"    // 중위 호출
```

인자가 하나 뿐인 일반 메서드, 인자가 하나 뿐인 확장 함수에 중위 호출을 사용할 수 있다. 중위 호출을 허용하려면 함수 선언 앞에 infix 변경자를 추가해야 한다. 

```kotlin
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

to() 함수는 두 원소로 이뤄진 순서쌍을 표현하는 Pair의 인스턴스를 반환한다. 

Pair의 내용으로 두 변수를 초기화 할 수 있다.

```kotlin
val (num, name) = 1 to "one"

println(num)
println(name)

// 출력 결과
// 1
// one
```

이러한 기능을 구조 분해 선언이라고 부른다.

루프를 사용할 때도 구조 분해 선언을 사용할 수 있다. withIndex를 활용하면 컬렉션의 원소와 index를 따로 변수에 담을 수 있다.

```kotlin
for ((index, element) in collection.withIndex()) {
   ...    
}
```

mapOf 함수의 선언은 다음과 같다.

```kotlin
public fun <K, V> mapOf(vararg pairs: Pair<K, V>): Map<K, V>
```

mapOf() 함수도 listOf() 함수처럼 인자를 가변적으로 받기 때문에 vararg 변경자를 사용하는데 인자가 Pair 타입이여야 하기 때문에 타입이 Pair로 정의되어 있는 것을 확인할 수 있다.

## 3.5 문자열과 정규식 다루기

코틀린 문자열은 자바 문자열과 같다. 코틀린은 다양한 확장 함수를 지원해서 표준 자바 문자열을 더 유연하게 사용할 수 있게 한다.

### 3.5.1 문자열 나누기

자바의 split() 함수의 구분 문자열은 정규식이다. 코틀린에서는 정규식을 파라미터로 받는 함수는 String이 아닌 Regex 타입의 값을 받는다. 그래서 split 함수에 전달하는 값의 타입에 따라 정규식이나 일반 텍스트 중 어느 것으로 문자열을 분리하는지 쉽게 알 수 있다. 

근데 split 확장 함수를 오버로딩한 버전 중에는 구분 문자열을 하나 이상 인자로 받는 함수가 있다. 

```kotlin
println("12.345-6.A".split("\\.|-".toRegex()))   // 정규식을 명시적으로 만들어 사용
println("12.345-6.A".split(".", "-"))            // 여러 구분 문자열 지정
```

두 식의 결과는 동일하다.

## 3.6 코드 다듬기: 로컬 함수와 확장

자바 코드를 작성할 때 DRY 원칙을 지키는 것을 어렵다. 긴 메서드들을 부분 부분으로 나눠서 재활용할 수 있는데 이럴 경우 클래스 안에 작은 메서드들이 많아지고, 이로 인해 메서드 간의 관계 파악이 어려워 코드를 이해 하기 힘들어진다.  

이러한 문제를 코틀린에서는 함수에서 추출한 함수를 원 함수 내부에 중첩 시켜 해결할 수 있다. 

```kotlin
class User(val id: Int, val name: String, val address: String)  {
    fun saveUser(user: User) {
        if(user.name.isEmpty()) {
            throw IllegalArgumentException (
                "Can't save user ${user.id}: empty name"
            )
        }

        if(user.address.isEmpty()) {
            throw IllegalArgumentException (
                "Can't save user ${user.id}: empty Address"
            )
        }
    }
}
```

위의 클래스를 보면 if문이 확인하는 변수만 다르지 동작은 비슷한 것을 볼 수 있다. 이를 로컬 함수를 활용하여 중복을 줄일 수 있다.

- 로컬 함수를 사용하여 코드 중복 줄이기

```kotlin
class User(val id: Int, val name: String, val address: String)  {
    fun saveUser(user: User) {
        fun validate(
            user: User,
            value: String,
            fieldName: String
        ) {
            if(value.isEmpty()) {
                throw IllegalArgumentException (
                    "Can't save user ${user.id}: empty $fieldName"
                )
            }
        }
    }
}
```

근데 로컬 함수는 자신이 속한 바깥 함수의 모든 파라미터와 변수를 이용할 수 있다. 이를 이용하면 불필요한 파라미터를 제거할 수 있다.

- 로컬 함수에서 바깥 함수의 파라미터 접근하기

```kotlin
class User(val id: Int, val name: String, val address: String)  {
    fun saveUser(user: User) {
        fun validate(value: String, fieldName: String) {
            if(value.isEmpty()) {
                throw IllegalArgumentException (
                    "Can't save user ${user.id}: empty $fieldName"
                )
            }
        }
    }
}
```

- 확장 함수로 추출하기

```kotlin
class User(val id: Int, val name: String, val address: String)
fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if(value.isEmpty()) {
            throw IllegalArgumentException (
                "Can't save user $id: empty $fieldName"
            )
        }
    }
}

fun saveUser(user: User) {
    user.validateBeforeSave()
}
```
