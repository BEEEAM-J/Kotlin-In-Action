## 4.1 클래스 계층 정의

코틀린에는 자바에는 없는 sealed 변경자가 있는데 이는 클래스의 상속을 제한한다.

### 4.1.1 코틀린 인터페이스

코틀린 인터페이스 안에는 추상 메서드 뿐만 아니라 구현이 있는 메서드도 정의할 수 있다. 하지만 아무런 상태(필드)도 들어갈 수 없다. 

```kotlin
interface Clickable {
    fun click()
}
```

위의 인터페이스는 추상 메서드인 click을 가지는 인터페이스이다. 이를 사용하기 위해서는 이 인터페이스를 구현하는 클래스를 만들어야 한다.

```kotlin
class Button: Clickable {
    override fun click() = println("Clicked!!")
}
```

코틀린에서는 **클래스 명 뒤에 콜론(:)**을 붙여서 **클래스 확장**, **인터페이스 구현**을 처리한다. 

클래스는 인터페이스를 개수에 제한 없이 구현이 가능하지만 클래스 확장은 하나만 가능하다. 

override 변경자는 인터페이스에 있는 프로퍼티나 메서드를 오버라이드 한다는 뜻이다. 자바에서와 달리 **코틀린에서는 이를 꼭 사용해야 한다.** 왜냐면 이는  실수로 상위 클래스의 메서드를 오버라이드 하는 경우를 막아주기 때문이다. 

인터페이스 메서드도 디폴트 구현을 제공할 수 있다. 

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("interface 내부 구현 Method")         // 디폴트 구현
}
```

디폴트 구현이 있는 메서드는 인터페이스를 구현하는 클래스에서 재정의 할 수도 있고, 재정의 하지 않는 경우에는 디폴트 구현을 사용한다. 

```kotlin
 interface Focusable {
    fun setFocus(b: Boolean)
        = println("I ${if (b) "got" else "lost"} focus.")
    fun showOff() = println("I'm focusable!")
}
```

Clickable 인터페이스와 Focusable 인터페이스는 showOff() 라는 동일한 이름을 갖는 메서드를 가지고 있다. 이러한 두 인터페이스를 한 클래스에서 구현하려면 어떻게 해야 할까?

→ 코틀린 컴파일러는 두 메서드를 아우르는 구현을 하위 클래스에 직접 구현하게 강제할 수 있다. 
    그래서 이를 이용하면 된다.

```kotlin
class Button: Clickable, Focusable {
    override fun click() = println("Clicked!!")
    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

이름과 시그니처가 같은 메서드에 **둘 이상의 디폴트 구현이 있는 경우** 인터페이스를 구현하는 하위 클래스에서 **명시적으로 새로운 구현을 제공**해야 한다.

코틀린에서 상위 타입의 구현을 호출하려면 super<상위 타입> 형식을 사용하면 된다. 

```kotlin
fun main(args: Array<String>) {
    Button().click()
    Button().setFocus(false)
    Button().showOff()
}

// 출력 결과
// Clicked!!
// I lost focus.
// interface 내부 구현 Method
// I'm focusable!
```

지금까지 봤던 예제를 실행하면 위와 같은 결과가 나온다. 두 인터페이스를 구현하는 Button 클래스에서 showOff() 함수 안에 각 인터페이스의 showOff() 함수를 선언하였기 때문에 각 인터페이스의 showOff() 함수의 디폴트 구현이 실행되는 것을 확인할 수 있다. 

### 4.1.2 open, final, abstract 변경자: 기본적으로 final

클래스들이 기본적으로 상속이 가능하면 편한 점도 많지만 문제가 생기는 경우도 많다. 

**취약한 기반 클래스** 문제가 발생할 수 있다. 이는 상위 클래스의 변경으로 하위 클래스가 상위 클래스에 대해 가졌던 가정이 변하는 경우에 발생한다. 상위 클래스의 변경이 하위 클래스의 동작을 예기치 않게 변경할 수 있기 때문에 **기반 클래스**가 **취약**하다고 한다.

이러한 문제에 대해서 “Effective Java”라는 책에서는 다음과 같이 언급한다. 

> 상속을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 상속을 금지하라
> 

무슨 뜻이냐면 **하위 클래스에서 오버라이드 하도록 의도하지 않은 것들은 모두 final로 만들라**는 뜻이다.

코틀린의 클래스와 메서드들은 기본적으로 final 이다. 만약 상속을 허용하려면 open 변경자를 앞에 붙이면 된다.

```kotlin
open class RichButton: Clickable {
    fun disable() {}             // final이라서 하위 클래스가 오버라이드 불가능
    open fun animate() {}        // 오버라이드 가능    
    override fun click() {}      // 오버라이드된 함수는 기본적으로 open이다.
}
```

오버라이드하는 메서드를 하위 클래스에서 오버라이드 하지 못하게 하려면 앞에 final 변경자를 붙이면 된다. 

```kotlin
open class RichButton: Clickable {
    final override fun click() {}
}
```

코틀린에서도 자바처럼 abstract를 사용하여 추상 클래스를 만들 수 있다. 이렇게 만든 추상 클래스는 구현이 없기 때문에 인스턴스화 할 수 없다. 추상 멤버는 항상 open이기 때문에 앞에 open 변경자를 붙이지 않아도 된다. 

```kotlin
abstract class Animated {
    abstract fun animate()        
    open fun stopAnimating() { }  // 추상 클래스 내의 비추상 함수는 기본으로 final이다
    fun animateTwin() { }         // open을 붙여서 오버라이드를 허용할 수 있다.
}
```

인터페이스 멤버는 final, open, abstract를 사용하지 않는다. 인터페이스 멤버는 항상 open이며 final로 바꿀 수 없다. 

![1](https://github.com/BEEEAM-J/Kotlin-In-Action/assets/107917980/040a258b-16ae-464f-bf73-6fdb1a8a16b5)

### 4.1.3 가시성 변경자: 기본적으로 공개

코틀린은 자바처럼 가시성 변경자가 public과 protected가 있다. 기본적으로 아무 변경자가 없는 경우에는 public이다. 코틀린에는 패키지 전용 가시성에 대한 대안으로 internal 변경자가 있다. 이는 모듈 내부에서만 볼 수 있음을 의미한다. 모듈은 한 번에 한꺼번에 컴파일되는 코틀린 파일들을 의미한다. 

코틀린에서는 최상위 선언(클래스, 함수, 프로퍼티)에 대해 private 가시성을 허용한다.
![2](https://github.com/BEEEAM-J/Kotlin-In-Action/assets/107917980/1f7cc852-e3ed-49c1-a244-eef7b0f0cb0f)

```kotlin
internal open class TalkativButton: Focusable {
    private fun yell() = println("Hey!")
    protected fun whisper() = println("Let's talk!")
}

fun TalkativButton.giveSpeech() {
    yell()
    whisper()
}
```

코틀린은 public인 giveSpeech() 함수 안에서 그보다 낮은 가시성이 낮은 타입을 참조하지 못하게 한다. 

이는 어떤 클래스가 상속하는 상위 클래스의 가시성이 더 높거나 같아야 하고, 메서드도 가시성이 같거나 더 높아야 하기 때문이다.

위의 예제는 오류가 발생하는데 오류를 해결하기 위해서는 TalkativButton 클래스의 가시성을 public으로 바꾸거나 giveSpeech() 함수의 가시성을 internal로 바꿔야 한다.

자바에서는 같은 패키지 안에서 protected 멤버에 접근할 수 있지만, 코틀린에서는 그렇지 않다. protected 멤버는 어떤 클래스나 그 클래스를 상속하는 클래스 내에서만 보인다. 하지만 해당 클래스를 확장한 함수는 그 클래스의 private이나 protected 멤버에 접근할 수 없다.

### 4.1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

자바처럼 코틀린에서도 클래스 내부에 클래스를 선언할 수 있다. 이는 도우미 클래스를 캡슐화하거나 코드 정의를 그 코드를 사용하는 곳 가까이에 두고 싶을 때 유용하다. 

근데 자바와의 차이는 코틀린의 중첩 클래스는 명시적으로 요청하지 않으면 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다는 점이다. 

중첩 클래스와 내부 클래스는 바깥 클래스에 참조의 가능 여부로 나뉜다. 중첩 클래스는 바깥 클래스에 대한 참조가 불가능하고, 내부 클래스는 바깥 클래스에 대한 참조가 가능하다. 

코틀린 중첩 클래스에 아무런 변경자가 붙어있지 않으면 자바 static 중첩 클래스와 같다. 만약 이를 내부 클래스로 만들려면 클래스 앞에 inner 변경자를 붙이면 된다. inner를 붙여서 내부 클래스로 만들면 중첩 클래스와 달리 바깥 클래스에 대해 참조를 할 수 있다. 바깥 클래스의 인스턴스에 접근하려면 “this@바깥 클래스 이름”을 사용하면 된다.

```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```

### 4.1.5 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

상위 클래스에 sealed 변경자를 붙이면 sealed 클래스가 되는데 이 클래스는 이를 상속한 하위 클래스의 정의를 제한 할 수 있다. sealed 클래스의 하위 클래스를 정의하려면 반드시 상위 클래스 안에 중첩 시켜야 한다. 

```kotlin
sealed class Expr {
    class Num(val value: Int): Expr()
    class Sum(val left: Expr, val right: Expr): Expr()
}

fun eval(e: Expr): Int =
    when(e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.left) + eval(e.right)
    }
```

위의 코드는 sealed 클래스를 사용하는 예제이다. 가장 먼저 sealed 클래스의 하위 클래스들이 sealed 클래스 내에 선언된 것을 확인할 수 있다. 

그리고 Expr 클래스를 상속 받는 클래스를 Num, Sum 클래스로 제한을 하였기 때문에 when문 에서 else 분기 처리를 하지 않아도 된다. 

### 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

코틀린에서 생성자는 주 생성자(클래스 본문 밖에서 정의), 부 생성자(클래스 본문 내에서 정의)로 구분이 된다. 그리고 초기화 블록을 통해서 초기화 로직을 추가할 수도 있다. 

### 4.2.1 클래스 초기화: 주 생성자와 초기화 블록

```kotlin
class User(_nickname: String) {
    val nickname: String
    
    init {
        nickname = _nickname
    }
}
```

클래스 명 옆에 괄호 안의 코드를 주 생성자라고 한다. 그리고 init 함수가 초기화 블록이다. 위의 코드는 주 생성자로 받은 값을 클래스 초기화할 때 클래스 내의 nickname 변수에 넣는 동작을 한다. 

```kotlin
class User(_nickname: String) {
    val nickname: String = _nickname
}
```

근데 nickname 프로퍼티를 초기화 하는 동작을 nickname 프로퍼티 선언에 포함 시킬 수 있어서 init 블록이 필요하지 않다. 

```kotlin
class User(_nickname: String = "none") {
    val nickname: String = _nickname
}
```

클래스 주 생성자에 위와 같이 디폴트 값을 제공할 수도 있다. 

```kotlin
open class User(_nickname: String) {
    val nickname: String = _nickname
}

class KakaoUser(name: String): User("paca")
```

만약 상속하는 클래스의 주 생성자 값이 필요하다면 위와 같이 상속하는 클래스 다음에 괄호 안에 값을 넣으면 된다. 

클래스를 정의할 때 별도로 생성자를 정의하지 않으면 컴파일러가 자동으로 디폴트 생성자를 만들어 주기 때문에 꼭 클래스를 호출할 때 빈괄호라도 있어야 한다.

### 4.2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화

부 생성자는 constructor 키워드로 시작한다. 

```kotlin
open class View {
    constructor(ctx: Context){ 
        
    }
    constructor(ctx: Constext, attr: AttributeSet)
}
```

부 생성자의 주된 필요 이유는 자바 상호운용성이다. 아니면 클래스 인스턴스를 생성할 때 파라미터 목록이 다른 생성 방법이 여럿 존재하는 경우에도 부 생성자가 여럿 필요하다.

### 4.2.3 인터페이스에 선언된 프로퍼티 구현

인터페이스에는 추상 프로퍼티 선언을 할 수 있다. 

```kotlin
interface User() {
    val nickname: String
}
```

위와 같이 선언한 경우에는 이를 구현하는 클래스에서 nickname 값을 얻을 수 있는 방법을 제공해야 한다. 

```kotlin
class PrivateUser(override val nickname: String): User

class SubcribingUser(val email: String): User {
    override val nickname: String
        get() = email.substringBefore('@')
}

class FaceBookUser(val accountId: Int): User {
    override val nickname = getFaceBookName(accountId)
}
```

인터페이스에 선언된 프로퍼티는 위와 같이 다양한 방식으로 구현이 가능하다. 

SubcribingUser 클래스와 FaceBookUser 클래스는 차이점이 있다. SubcribingUser는 매번 호출될 때마다 커스텀 게터를 활용하고, FaceBookUser는 객체 초기화 시 계산한 데이터를 뒷받침하는 필드에 저장해놨다가 불러오는 방식이다. 

인터페이스에 게터, 세터가 있는 프로퍼티 선언이 가능하다. 

```kotlin
interface User() {
    val email: String
    val nickname: String
        get() = email.substringBefore('@')
}
```

위와 같이 선언을 하면 email 프로퍼티는 반드시 override 해야하지만 nickname 프로퍼티는 상속이 가능하다.

### 4.2.4 게터와 세터에서 뒷받침하는 필드에 접근

```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println("""
                Address was changed for $name: 
                "$field" -> "$value".""".trimIndent())      // 뒷받침하는 필드 값 읽기
            field = value                                   // 뒷받침하는 필드 값 변경
        }
}

val user = User("Alice")
user.address = "경기도"

// 실행 결과
// Address was changed for Alice: 
// "unspecified" -> "경기도".
```

위의 코드는 프로퍼티에 저장된 값이 변경될 때마다 문장을 출력하는 예제다. 

커스텀 세터 구문 내에서 “field” 라는 특별한 식별자를 통해서 뒷받침하는 필드에 접근이 가능하다.  

### 4.2.5 접근자의 가시성 변경

접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같다. 하지만 get이나 set 앞에 가시성 변경자를 추가해서 접근자의 가시성을 변경할 수 있다. 

```kotlin
class LengthCounter {
    var counter: Int = 0
        private set             // 이 클래스 밖에서는 counter 값 변경 불가

    fun addWord(word: String) {
        counter += word.length
    }
}
```

counter 프로퍼티의 set 앞에 private을 붙여서 가시성을 변경한 것을 확인할 수 있다. 

```kotlin
fun main(args: Array<String>) {
    lengthCounter.counter = 6
}   
// Cannot assign to 'counter': the setter is private in 'LengthCounter'
```

그래서 위와 같이 LengthCounter 클래스 밖에서 counter 값을 임의로 변경하려고 하면 주석과 같은 내용의 오류가 보이게 된다. 

**프로퍼티에 대해 나중에 다룰 내용**

> lateinit 변경자는 프로퍼티를 생성자가 호출된 다음에 초기화 한다는 뜻이다.
> 

> 지연 초기화(lazy initialized) 프로퍼티는 위임 프로퍼티(delegated property)의 일종이다.
> 

> 자바 프레임워크와의 호환성을 위해 자바의 특징을 코틀린에서 에뮬레이션하는 애노테이션을 활용할 수 있다.
> 

## 4.3 컴파일러가 생성한 메서드: 데이터 클래스와 클래스 위임

### 4.3.1 모든 클래스가 정의해야 하는 메서드

자바와 마찬가지로 코틀린 클래스에서도 toString(), equals(), hashCode() 등을 오버라이드할 수 있다.  고객의 이름과 우편번호를 저장하는 클래스를 통해서 각각의 메서드들이 어떤 기능을 갖는지, 어떻게 정의해야 하는지 예제로 사용하겠다.

```kotlin
// 예제 클래스 
class Client(val name: String, val postalCode: String) {
}
```

**문자열 표현: toString()**

자바처럼 코틀린의 모든 클래스도 인스턴스의 문자열 표현을 얻을 방법을 제공한다. 

```kotlin
val client1 = Client("조범준", "1234")
println(client1)

// 실행 결과
// chap4.Client@340f438e
```

기본적으로 제공되는 문자열 표현은 위의 코드의 실행 결과와 같은 형태이다. 이러한 형태를 바꾸려면 toString() 메서드를 오버라이드 해야한다.

```kotlin
class Client(val name: String, val postalCode: String) {
    override fun toString(): String = "Client(name = $name, postalCode = $postalCode)"
}
```

toString() 메서드를 오버라이드 하여 위와 같이 정의하고 인스턴스의 문자열 표현을 찍으면 원하는 형태의 문자열을 얻을 수 있다. 

```kotlin
val client1 = Client("조범준", "1234")
println(client1)

// 실행 결과
// Client(name = 조범준, postalCode = 1234)
```

**객체의 동등성: equals()**

```kotlin
val client1 = Client("조범준", "1234")
val client2 = Client("조범준", "1234")

println(client1 == client2)

// 실행 결과
// false
```

위와 같이 같은 값을 갖는 객체들의 동등성 검사를 하면 false가 나오게 된다. 만약 같은 내용을 가지고 있으면 동등한 객체로 간주하게 하려면 어떻게 해야 할까?

equals() 메서드를 오버라이드 하면 된다.

```kotlin
class Client(val name: String, val postalCode: String) {
    override fun toString(): String = "Client(name = $name, postalCode = $postalCode)"

    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Client)      // Client 객체인지 확인
            return false
        return name == other.name && 
                postalCode == other.postalCode      // 두 객체의 값이 일치하는지 확인
    }
}
```

is는 어떤 값의 타입을 검사한다. 그래서 equals() 메서드를 오버라이드 하여 먼저 Client 객체인지 확인을 하고, 두 객체가 같은 값을 가지고 있는지 확인한 후 같으면 true를 반환하게 된다.

**해시 컨테이너: hashCode()**

```kotlin
val processed = hashSetOf(Client("조범준", "1234"))
println(processed.contains(Client("조범준", "1234")))

// 실행 결과
// false
```

hashSet은 원소를 비교할 때 비용을 줄이기 위해 먼저 객체의 해시 코드를 비교하고 같은 경우에만 실제 값을 비교한다. 위의 예제의 두 객체의 해시 코드가 다르기 때문에 두 번째 객체가 집합 안에 없다고 판단하였다. 

이러한 문제를 해결하기 위해서는 hashCode() 메서드를 오버라이드해서 다음과 같이 정의하면 된다.

```kotlin
class Client(val name: String, val postalCode: Int) {
		...
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```

### 4.3.2 데이터 클래스: 모든 클래스가 정의해야 하는 메서드 자동 생성

어떤 클래스가 데이터를 저장하는 역할만 하면 toString(), equals(), hashCode() 메서드를 오버라이드 하면 된다. 

근데 코틀린에서는 data class로 정의하면 위의 과정을 컴파일러가 자동으로 만들어준다.

```kotlin
data class Client(val name: String, val postalCode: Int)
```

equals()와 hashCode()는 주 셍성자 내의 프로퍼티를 고려해서 만들어진다. 이 때문에 주 생성자 밖에 정의된 프로퍼티들은 equals()나 hashCode()를 계산할 때 고려 대상이 아니다. 

**데이터 클래스와 불변성: copy() 메서드**

데이터 클래스의 프로퍼티들은 무조건 val 타입일 필요는 없다. 하지만 val로 만들어서 불변 클래스로 만드는 것이 권장된다. 

hashMap 등 컨테이너에 데이터 클래스를 담는 경우에는 불변 객체가 필수적이다. 

데이터 클래스 객체를 키로 하는 컨테이너가 있을 때 만약 데이터 클래스의 프로퍼티 값을 바꾸면 컨테이너의 상태가 변할 수 있다. 

그리고 불변 객체를 사용하면 프로그램이 더 쉽게 추론이 되는 장점이 있다. 특히 다중 스레드를 사용하는 프로그램의 경우 어떤 스레드가 사용 중인 데이터를 다른 스레드가 변경할 수 없어지기 때문에 스레드를 동기화 해야 할 필요가 줄어든다.

copy() 메서드는 객체를 복사하면서 일부 프로퍼티를 변경할 수 있게 하는 메서드인데 이를 사용하면 데이터 클래스를 불변 객체로 활용하기 더 쉬워진다. 

### 4.3.3 클래스 위임: by 키워드 사용

상속을 허용하지 않는 클래스에 동작을 추가하는 일반적인 방법은 데코레이터 방법이다. 이는 상속을 할 수 없는 클래스 대신에 데코레이터 클래스를 만들어서 사용하는 것인데 데코레이터 클래스는 기존 클래스와 같은 인터페이스를 제공할 수 있어야 하고, 기존 클래스를 내부에 필드로 유지하는 것이다. 

새로 정의하는 기능은 데코레이터의 메서드에 새로 정의하고, 기존 기능이 필요한 경우에는 기존 클래스의 메서드에 요청을 전달한다. 

이 방법의 단점은 준비 코드가 많다는 것이다.

```kotlin
class DelegatingCollection<T>() : Collection<T> {
    private val innerList = arrayListOf<T>()
    
    override val size: Int get() = innerList.size
    override fun isEmpty(): Boolean = innerList.isEmpty()
    override fun contains(element: T): Boolean = innerList.contains(element)
    override fun iterator(): Iterator<T> = innerList.iterator()
    override fun containsAll(elements: Collection<T>): Boolean =
        innerList.containsAll(elements)
}
```

근데 코틀린에서는 by 키워드를 통해서 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시할 수 있다.

```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = ArrayList<T>()
): Collection<T> by innerList { }
```

by 키워드를 사용하면 위와 같이 함수 body에 코드들이 사라진다. 앞에서 선언했던 준비 코드들은 컴파일러가 생성해준다. 만약 메서드 중에서 변경하고 싶은 부분이 있으면 해당 메서드를 오버라이드하면 된다.

**by 키워드를 사용한 예시 코드**

```kotlin
class CountingSet<T> (
    val innerSet: MutableCollection<T> = HashSet<T>()
): MutableCollection<T> by innerSet {     // MutableCollection의 구현을 innerSet에게 위임

    var objectsAdded = 0

    override fun add(element: T): Boolean {             // 위임하지 않고 새로운 구현 제공
        objectsAdded += 1
        return innerSet.add(element)
    }

    override fun addAll(c: Collection<T>): Boolean {    // 위임하지 않고 새로운 구현 제공
        objectsAdded += c.size
        return innerSet.addAll(c)
    }
} 
```

## 4.4 object 키워드: 클래스 선언과 인스턴스 생성

object 키워드는 클래스를 정의하는 동시에 인스턴스를 생성한다. 

**object 키워드를 사용하는 상황**

- 객체 선언은 싱글턴을 정의하는 방법 중 하나
- 동반 객체는 인스턴스 메서드는 아니지만 어떤 클래스와 관련된 메서드, 팩토리 메서드를 담을 때 사용
- 객체 식은 자바의 무명 내부 클래스 대신 사용

### 4.4.1 객체 선언: 싱글턴을 쉽게 만들기

코틀린은 싱글턴을 기본적으로 지원한다. 

> 싱글턴: 클래스의 유일한 객체 저장
> 

object 키워드를 사용해서 만들 수 있다. 클래스처럼 프로퍼티, 메서드, init 블록이 들어갈 수 있다. 하지만 생성자는 사용할 수 없다. 왜냐면 싱글턴 객체는 생성자 호출 없이 즉시 만들어지기 때문이다. 

```kotlin
object CaseInsensitiveFileComparator: Comparator<File> {
    override fun compare(o1: File, o2: File): Int {
        return o1.path.compareTo(o2.path, ignoreCase = true)
    }
}
```

object는 클래스나 인터페이스를 상속할 수도 있다. 그리고 클래스 내부에서 선언이 가능하다.

```kotlin
data class Person(val name: String) {
    object NameComparator: Comparator<Person> {
        override fun compare(o1: Person, o2: Person): Int = 
            o1.name.compareTo(o2.name)
    }
}
```

### 4.4.2 동반 객체: 팩토리 메서드와 정적 멤버가 들어갈 장소

코틀린은 자바의 static을 지원하지 않는다. 대신에 최상위 함수나 객체 선언을 활용한다. 근데 최상위 함수는 클래스의 비공개 멤버에 접근이 불가능하다. 근데 클래스 내부 정보에 접근해야 하는 경우에는 접근해야 하는 것을 중첩된 객체 선언의 멤버로 정의해야 한다. 이는 companion을 붙여서 만든다.

```kotlin
class A {
    companion object {
        fun bar() {
            println("Companion object Called!")
        }
    }
}
```

동반 객체는 private 생성자를 호출하기 좋은 위치이다. 동반 객체는 바깥 쪽 클래스의 private 생성자도 호출할 수 있어서 팩토리 패턴을 구현하기 가장 적합한 위치이다.

다음은 팩토리 패턴 구현 예시이다.

```kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
        
        fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
    }
}
```

팩토리 메서드는 매우 유용하다. 

- 목적에 따라 팩토리 메서드 이름을 정할 수 있다.
- 팩토리 메서드는 선언된 클래스의 하위 클래스의 객체를 반환할 수도 있다.
- 생성할 필요 없는 객체를 생성하지 않을 수도 있다.

만약 클래스를 확장해야 하는 경우에는 동반 객체를 사용하는 것보다 여러 생성자를 사용하는게 더 좋은 방법이다.

### 4.4.3 동반 객체를 일반 객체처럼 사용

동반 객체는 클래스 안의 일반 객체이기 때문에 이름을 붙이거나 인터페이스를 상속할 수도 있고, 내부에서 확장 함수와 프로퍼티를 정의할 수 있다.

```kotlin
class Person(val name: String) {
    companion object Loader {
        fun fromJson(jsonText: String): Person = ...
    }
}
```

**동반 객체에서 인터페이스 구현**

```kotlin
interface JSONFactory<T> {
    fun fromJson(jsonText: String): T
}

class Person(val name: String) {
    companion object : JSONFactory<Person> {
        fun fromJson(jsonText: String): Person = ...
    }
}
```

**동반 객체 확장**

```kotlin
class Person(val name: String) {
    companion object {
    }
}

fun Person.Companion.fromJson(json: String): Person {
    ...
}

// 호출
// val p = Person.fromJson(json)
```

동반 객체에 대한 확장함수를 작성하려면 원래 클래스에 비어있는 동반 객체라도 꼭 선언해야 한다. 

호출하는 형식을 보면 클래스 멤버 함수처럼 보이지만 멤버 함수가 아니다. 

### 4.4.4 객체 식: 무명 내부 클래스를 다른 방식으로 작성

무명 객체를 정의할 때도 object 키워드를 사용한다. 이는 자바의 무명 내부 클래스를 대신한다. 

코틀린의 무명 객체는 자바의 무명 내부 클래스와 달리 여러 인터페이스를 구현하거나 클래스를 확장하면서 인터페이스를 구현할 수 있다.

그리고 **객체 선언과 달리 무명 객체는 싱글턴이 아니다.** 그래서 객체 식이 사용될 때마다 새로운 인스턴스가 생성된다.
