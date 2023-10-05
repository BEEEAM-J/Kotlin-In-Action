9장에서는 **제네릭스**와 **실체화한 타입 파라미터(reified type parameter)**, **선언 지점 변성(declaration-site variance)** 등의 내용을 볼 수 있다. 

**실체화한 타입 파라미터**를 사용하면 인라인 함수 호출에서 타입 인자로 쓰인 구체적인 타입을 실행 시점에서 알 수 있다. 
(일반 클래스나 함수에서는 타입 인자 정보가 실행 시점에서 사라지기 때문에 불가능)

**선언 지정 변성**을 사용하면 기저 타입은 같지만 타입 인자가 다른 두 제네릭 타입 Type<A>와 Type<B>가 있을 때 타입 인자 A와 B의 상위/하위 타입 관계에 따라 두 제네릭 타입의 상위/하위 타입 관계가 어떻게 되는지 지정할 수 있다. 
(List<Any>를 인자로 받는 함수에게 List<Int>타입의 값을 전달할 수 있을지 여부 지정 가능)

## 9.1 제네릭 타입 파라미터

제네닉스를 사용하면 타입 파라미터를 받는 타입을 정의할 수 있다. 제네릭 타입의 인스턴스를 만들려면 타입 파라미터를 구체적으로 명시해야 한다. 

ex) List<String>, Map<String, Person>

### 9.1.1 제네릭 함수와 프로퍼티

제네릭 함수를 사용하면 모든 리스트를 다룰 수 있다. 제네릭 함수를 호출할 때는 반드시 타입 인자를 구체적인 타입으로 넘겨야 한다. 

ex) List의 slice 함수

```kotlin
fun <T> List<T>.slice(indices: IntRange): List<T>
```

함수의 타입 파라미터 T가 수신 객체와 반환 타입에 쓰인다. 수신 객체와 반환 타입 모두 List<T>이다. 이런 함수는 리스트에 대해 호출할 때 타입 인자를 명시적으로 지정할 수 있다. 

```kotlin
val letters = ('a'..'z').toList()

println(letters.slice<Char>(0..2)) // 타입 인자를 명시적으로 지정
println(letters.slice(10..13))     // 컴파일러가 T가 Char라고 추론한다.
```

두 호출의 결과 타입은 모두 List<Char>이다. 컴파일러가 반환 타입의 List<T>의 T를 Char로 치환한다. 

### 9.1.2 제네릭 클래스 선언

자바처럼 코틀린에서도 타입 파라미터를 넣은 꺽쇠 기호(<>)를 클래스 이름 뒤에 붙이면 클래스를 제네릭하게 만들 수 있다. 이렇게 하면 해당 타입 파라미터를 클래스 본문 안에서 일반 타입처럼 사용할 수 있다. 

```kotlin
interface List<T> {
    operator fun get(index: Int): T
    // ...
}
```

제네릭 클래스를 확장하는 클래스를 정의하려면 기반 타입의 제네릭 파라미터에 대해 타입 인자를 지정해야 한다. **구체적인 타입**을 넘길 수도 있고 아님 **타입 파라미터로 받은 타입**을 넘길 수도 있다. 

```kotlin
class StringList: List<String> {
    override fun get(index: Int): String {
        // ...
    }
}

class ArrayList<T>: List<T> {
    override fun get(index: Int): T {
        // ...
    }
}
```

ArrayList<T>의 T는 List<T>의 T와 다르다. 둘은 전혀 다른 타입 파라미터라서 T가 아니라 다른 이름을 사용해도 상관없다. 

### 9.1.3 타입 파라미터 제약

타입 파라미터 제약은 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능이다. 

어떤 타입을 제네릭 타입의 타입 파라미터에 대한 상한으로 지정하면 그 제네릭 타입을 인스턴스화할 때 사용하는 타입 인자는 반드시 그 상한 타입이거나 그 상한 타입의 하위 타입이여야 한다. 

제약을 하기 위해서는 타입 파라미터의 이름 뒤에 콜론(:)을 표시하고 뒤에 상한 타입을 작성하면 된다. 

```kotlin
fun <T: Number> oneHalf(value: T): Double {
    return value.toDouble() / 2.0
}

println(oneHalf(3.0))
// 결과
// 1.5

println(oneHalf("a"))
// Type mismatch.
// Required:Number
// Found:String
```

만약 타입 파라미터에 대해 둘 이상의 제약을 가해야 하는 경우에는 다른 구문을 사용한다. 

다음 함수는 문자열의 끝에 마침표(.)가 없으면 추가하는 함수이다. 이 함수는 타입 파라미터에 둘 이상의 제약을 해야한다. 

1. 문자열이여야 한다.
2. append 가능해야 한다.

```kotlin
fun <T> ensureTrailingPeriod(seq: T) where T: CharSequence, T: Appendable {
    if(!seq.endsWith('.')) {
        seq.append('.')
    }
}
```

where 뒤에 제약하고자 하는 타입 파리미터의 타입을 명시하면 된다. 

### 9.1.4 타입 파라미터를 널이 될 수 없는 타입으로 한정

타입 파라미터의 타입을 명시하지 않으면 자동으로 Any? 타입으로 추론된다. 

만약 널 가능성을 제외한 아무런 제약이 필요 없으면 타입 파라미터의 타입을 Any로 명시하면 된다.

```kotlin
// nullable
class Processor<T> {
    fun process(value: T) {
        value?.hashCode()
    }
}

// non-nullable
class Processor<T: Any> {
    fun process(value: T) {
        value.hashCode()
    }
}
```

## 9.2 실행 시 제네릭스의 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터

JVM의 제네릭스는 보통 타입 소거를 사용해 구현된다. 이는 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지 않다는 뜻이다. 함수를 inline으로 만들면 타입 인자가 지워지지 않을 수 있다. 

### 9.2.1 실행 시점의 제네릭: 타입 검사와 캐스트

제네릭 클래스 인스턴스가 그 인스턴스를 생성할 때 쓰인 타입 인자에 대한 정보를 유지하지 않는다. 

ex) List<String> 객체에 여러 문자열을 넣어도 실행 시점에서는 오직 List 객체로만 볼 수 있다. 그 List 객체가 어떤 타입의 원소를 저장하는지 실행 시점에는 알 수 없다. 

**타입 소거로 인해 생기는 한계**

실행 시점에서 어떤 값이 List인지 여부는 알 수 있다. 하지만 해당 List가 어떤 타입의 원소들을 갖는지는 알 수 없다. 

하지만 이런 정보들이 지워짐으로써 메모리 사용량이 줄어드는 장점이 있다. 

**스타 프로젝션**

코틀린에서는 타입 인자를 명시하지 않고 제네릭을 사용할 수 없다. 근데 스타 프로젝션을 사용하면 인자를 알 수 없는 제네릭 타입을 표현할 수 있다. 

```kotlin
// ex) 원소 타입을 알 수 없는 리스트
List<*>
```

as나 as? 캐스팅을 해도 제네릭 타입을 사용할 수 있다. 

실행 시점에서는 제네릭 타입의 타입 인자 값을 알 수 없기 때문에 캐스팅은 항상 성공한다. 이런 타입 캐스팅을 사용하면 경고(검사할 수 없는 캐스팅)이 나오지만 정상적으로 사용할 수 있다.

```kotlin
fun printSum(c: Collection<*>) {
    val intList = c as? List<Int> // Unchecked cast: Collection<*> to List<Int> 경고 발생
        ?: throw IllegalArgumentException("List is expected")
    println(intList.sum())
}
```

### 9.2.2 실체화한 타입 파라미터를 사용한 함수 선언

제네릭 타입의 타입 인자 정보는 실행 시점에서 지워지기 때문에 제네릭 클래스의 인스턴스가 있어도 그 인스턴스를 만들 때 사용된 타입 인자를 알아낼 수 없다. 제네릭 함수에서도 마찬가지다. 

```kotlin
fun <T> isA(value: Any) = value is T
// Cannot check for instance of erased type: T 에러 발생
```

하지만 inline 함수의 타입 파라미터는 실체화가 되기 때문에 실행 시점의 타입 인자를 알 수 있다.

```kotlin
inline fun <reified T> isA(value: Any) = value is T

println(isA<String>("abc"))
println(isA<Int>(123))
println(isA<String>(123))

// 실행 결과
// true
// true
// false
```

성능 향상을 위해서 inline 함수를 사용하기도 하지만 위의 경우처럼 실체화된 타입 파라미터를 사용하기 위해서 사용하기도 한다. 하지만 inline 함수의 크기가 커지면 좋지 않기 때문에 이런 경우에는 실체화된 타입에 의존하지 않는 부분을 일반 함수로 빼는 것이 좋다.

### 9.2.3 실체화한 타입 파라미터로 클래스 참조 대신

java.lang.Class 타입 인자를 파라미터로 받는 API에 대한 코틀린 어댑터를 구현하는 경우 실체화한 타입 파라미터를 자주 사용한다.
ex) JDK의 ServiceLoader

```kotlin
// 기존의 ServiceLoader를 사용하여 서비스 읽는 코드
val serviceImpl = ServiceLoader.load(Provider.Service::class.java)

// 실체화한 타입 파라미터 사용
inline fun <reified T> loadService(): ServiceLoader<T>? {
    return ServiceLoader.load(T::class.java) // T::class로 타입 파라미터의 클래스를 가져온다.
}

val serviceImpl = loadService<Service>()
```

### 9.2.4 실체화한 타입 파라미터의 제약

**사용할 수 있는 경우**

- 타입 검사와 캐스팅(is, !is, as, as?)
- 코틀린 리플렉션 API(::class)
- 코틀린 타입에 대응하는 java.lang.class 얻는 경우(::class.java)
- 다른 함수를 호출할 때 타입 인자로 사용

**할 수 없는 것**

- 타입 파라미터 클래스의 인스턴스 생성
- 타입 파라미터 클래스의 동반 객체 메서드 호출하기
- 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
- 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 reified로 지정하기

## 9.3 변성: 제네릭과 하위 타입

변성은 List<String>과 List<Any>와 같이 기저 타입은 같지만 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념이다.

### 9.3.1 변성이 있는 이유: 인자를 함수에 넘기기

List<Any> 타입을 인자로 받는 함수에 List<String>을 넘겨도 된다고 생각할 수 있다. String 클래스가 Any 클래스를 확장하기 때문이다. 리스트의 원소에 변경이 없는 조건에서는 안전하다.

```kotlin
// MutableList<String> 타입이 전달되면 리스트에 Int 값이 추가되는 동작이 실행되서 오류 
fun addAnswer(list: MutableList<Any>) {
    list.add(43)
}

val strings = mutableListOf("abc", "def")
addAnswer(strings)
```

### 9.3.2 클래스, 타입, 하위 타입

어떤 타입 A의 값이 필요한 모든 곳에 어떤 타입 B의 값을 넣어도 문제가 없으면 B는 A의 하위 타입이라고 한다. 상위 타입은 하위 타입의 반대 개념이다. 

ex) B가 A의 하위 타입이면 A는 B의 상위 타입이다. 

<aside>
💡 모든 타입은 자신의 하위 타입이다.

</aside>

non-nullable 타입은 nullable 타입의 하위 타입이다. 

![하위 타입 관계](https://github.com/BEEEAM-J/Kotlin-In-Action/assets/107917980/f0f02f78-e873-4b7e-a8fb-499649c1808a)


앞에서 List<Any> 타입의 인자에 List<String>타입을 넣을 수 있는지 살펴 본 적이 있다. String은 Any의 하위 타입이지만 MutableList<String>은 MutableList<Any>의 하위 타입이 될 수 없다. 

이와 같이 제네릭 타입을 인스턴스화 할 때 타입 인자로 서로 다른 타입이 들어가면 인스턴스 사이의 하위 타입 관계가 성립되지 않으면 그 제네릭 타입을 **무공변**이라고 한다. 

코틀린의 List 인터페이스는 읽기 전용 컬렉션이다. 그래서 A가 B의 하위 타입이면 List<A>는 List<B>의 하위 타입이다.  이런 클래스나 인터페이스를 **공변적**이라고 부른다.

![공변, 무공변](https://github.com/BEEEAM-J/Kotlin-In-Action/assets/107917980/39feb736-c90d-48c6-8992-33e0afc5afac)


### 9.3.3 공변성: 하위 타입 관계를 유지

코틀린에서 제네릭 클래스가 타입 파라미터에 대해 공변적임을 표시하려면 타입 파라미터 이름 앞에 out을 넣어야 한다. 

```kotlin
interface Producer<out T> {  // 클래스가 T에 대해 공변적이라는 의미
    fun produce(): T
}
```

클래스의 타입 파라미터를 공변적으로 만들면 함수 정의에 사용된 타입 파라미터의 타입과 타입 인자의 타입이 정확히 일치하지 않아도 그 클래스의 인스턴스를 함수의 인자나 반환 값으로 사용할 수 있다. 

```kotlin
// Ex)
open class Animal {
    fun feed() { ... }
}

class Herd<T: Animal> {
    val size: Int get() = 1
    operator fun get (i: Int): T { ... }
}

fun feedAll(animals: Herd<Animal>) {
    for (i in 0 until animals.size) {
        animals[i].feed()
    }
}
```

위의 코드에서 feedAll 함수는 인자로 Herd<Animal> 타입을 받는다. 여기에 Animal 클래스를 확장한 Cat이라는 클래스를 Herd 클래스의 T 타입 파라미터로 갖는 Herd<Cat>을 전달하면 어떻게 될까?

```kotlin
class Cat: Animal() {
    fun cleanLitter() { ... }
}

fun takeCareOfCats(cats: Herd<Cat>) {
    for(i in 0 until cats.size) {
        cats[i].cleanLitter()
        feedAll(cats) // Type mismatch: inferred type is Herd<Cat> but Herd<Animal> was expected
    }
}
```

Herd의 타입 파라미터를 무공변성으로 지정했기 때문에 타입 에러가 발생하는 것을 확인할 수 있다. 위의 에러를 해결하려면 Herd의 타입 파라미터를 공변적으로 만들면 된다. 

```kotlin
class Herd<out T: Animal> {
    ...
}
```

클래스 멤버를 선언할 때 타입 파라미터를 사용할 수 있는 지점은 in, out으로 나뉜다. 

```kotlin
interface Transformer<T> {
    fun transform(t: T): T
//                   in  out
}
// 파라미터 타입이 in 위치, 반환 타입이 out 위치이다.
```

타입 파라미터를 공변적으로 만들면 클래스 내부에서는 그 파라미터를 사용하는 방법을 제한한다. 그래서 클래스 타입 파라미터 앞에 out을 붙이면 in 위치에서는 사용할 수 없고 out 위치에서만 사용할 수 있게 제한된다. 

### 9.3.4 반공변성: 뒤집힌 하위 타입 관계

타입 B가 타입 A의 하위 타입인 경우 클래스<A>가 클래스<B>의 타입인 관계가 성립하면 제네릭 클래스<T>는 타입 인자 T에 대해 반공변이다. 즉 반공변은 하위 타입 관계가 뒤집힌 것이다. 

![공변, 반공변](https://github.com/BEEEAM-J/Kotlin-In-Action/assets/107917980/b1d49a3b-bf85-4b18-b007-6cb5a2f682f3)


반공변 in 키워드를 붙여서 사용하고 공변성과 반대로 in 위치에서만 사용이 가능하다. 

클래스나 인터페이스가 어떤 타입 파라미터에 대해서 공변적이면서 반공변적일 수도 있다. 고차함수를 인자로 받는 경우이다. 

### 9.3.5 사용 지점 변성: 타입이 언급되는 지점에서 변성 지정

클래스를 선언하면서 변성을 지정하면 그 클래스를 사용하는 모든 장소에서 변성 지정자가 영향을 끼쳐서 편리한데 이를 **선언 지점 변성**이라 부른다. 타입 파라미터가 있는 타입을 사용할 때 마다 해당 타입 파라미터를 하위 또는 상위 타입으로 대치할 수 있는지 명시하는 것은 **사용 지점 변성**이라고 부른다. 이를 사용하면 타입 파라미터의 변성을 선언할 수 없는 경우에 사용되는 지점에서 변성을 지정할 수 있다.

사용 지점 변성을 사용하면 타입 인자로 사용할 수 있는 타입의 범위가 넓어지는 장점이 있다.

### 9.3.6 스타 프로젝션: 타입 인자 대신 * 사용

앞에서 제네릭 타입 인자 정보가 없음을 표현하기 위해 스타 프로젝션을 사용한다고 말했다. 그래서 원소의 타입을 모르는 리스트는 List<*>로 표현할 수 있다. 

**스타 프로젝션의 의미**

MutableList<*> ≠ MutableList<Any?>
MutableList<*>은 정해진 구체적인 타입을 담지만 아직 원소의 타입을 모른다는 의미라서 다르다. 

```kotlin
val list: MutableList<Any?> = mutableListOf('a', 1, "qwe")
val chars = mutableListOf('a', 'b', 'c')
val unknownElements: MutableList<*> = 
    if (Random().nextBoolean()) list else chars
// unknownElements.add(42) // 컴파일러가 호출 금지
// Kotlin: The integer literal does not conform to the expected type Nothing
println(unknownElements.first())
```

위의 코드에서 MutableList<*>는 MutableList<out Any?>처럼 동작한다. 

Any?는 코틀린 모든 타입의 상위 타입이기 때문에 안전하게 원소를 꺼내올 수 있지만 원소의 타입을 모르는 리스트에 값을 추가하는 것은 불가능하다.
