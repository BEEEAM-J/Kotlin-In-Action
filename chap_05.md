람다는 다른 함수에 넘길 수 있는 작은 코드 조각을 의미한다. 컬렉션 처리를 할 때 람다가 자주 사용된다. 

## 5.1 람다 식과 멤버 참조

### 5.1.1 람다 소개: 코드 블록을 함수 인자로 넘기기

람다 식을 사용하면 코드 블록을 직접 함수의 인자로 넘길 수 있다. Listener를 람다로 구현할 수 있다.

```kotlin
button.setOnClickListener { /* 클릭 시 수행할 동작 */ }
```

### 5.1.2 람다와 컬렉션

```kotlin
fun findTheOldest(people: List<Person>) {
    var maxAge = 0
    var theOldest: Person? = null
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            theOldest = person
        }
    }
    println(theOldest)
}
```

위의 함수는 컬렉션을 직접 검색하는 동작을 한다. 근데 코틀린에서는 위의 함수를 간단하게 만들 수 있다. 

```kotlin
println(people.maxBy { it.age })
println(people.maxBy(Person::age))

// 실행 결과
// Person(name=사자, age=30)
// Person(name=사자, age=30)
```

위와 같이 maxBy 함수를 사용하면 된다. 모든 컬렉션에 대해서 maxBy 함수를 호출할 수 있다.

중괄호 안에 코드는 비교에 사용할 값을 반환해주는 함수인데 이처럼 함수나 프로퍼티를 반환하는 역할을 하는 함다는 멤버 참조로 대치할 수 있다. 

### 5.1.3 람다 식의 문법

람다는 값을 전달할 수 있는 동작의 모음이다. 따로 선언해서 변수에 저장하는 것도 가능하다. 

코틀린 람다 식은 항상 중괄호 안에 들어있고, 화살표(→)가 인자와 본문을 구분해준다.

```kotlin
{ x: Int, y: Int -> x + y }
//       인자        본문
```

람다 식은 변수에 저장할 수 있다. 원하면 람다 식을 직접 호출하는 것도 가능하다.

```kotlin
val sum = { x: Int, y: Int -> x + y }       // 람다 식을 변수에 저장
println(sum(1, 2))
    
{ println(43) } ()                          // 람다 식을 직접 호출
```

코드 일부분을 블록으로 감싸서 실행할 필요가 있는 경우에는 run을 사용하면 좋다. run은 인자로 받은 람다를 실행해주는 라이브러리이다. 

```kotlin
run { println(43) }                         // run으로 람다 본문의 코드 실행
```

위에서 설명했던 maxBy 함수의 예시를 정식 람다로 작성하면 다음과 같다. 

```kotlin
people.maxBy({ p: Person -> p.age })
people.maxBy() { p: Person -> p.age }  // 맨 뒤의 인자가 람다라서 람다를 괄호 밖으로 뺌
people.maxBy { p: Person -> p.age }  
// 람다가 함수의 유일한 인자, 괄호 뒤에 람다 작성해서 빈 괄호 제거
```

위의 3줄의 코드는 약간은 다른 형식을 가지고 있지만 모두 같은 결과를 보여준다. 

코틀린에는 함수 호출 시 맨 뒤에 있는 인자가 람다이면 그 람다를 괄호 밖으로 빼낼 수 있는 문법 관습이 있다. 근데 람다가 함수의 유일한 인자이고, 괄호 뒤에 람다를 작성했다면 빈 괄호는 제거해도 된다.

```kotlin
people.maxBy { p -> p.age }
```

로컬 변수처럼 람다 파라미터도 컴파일러가 추론이 가능하다. 근데 컴파일러가 추론을 못하는 경우도 있는데 이럴 땐 그냥 타입을 명시해주면 된다!

```kotlin
people.maxBy { it.age }
```

람다의 파라미터가 하나뿐이고  그 타입을 컴파일러가 추론할 수 있는 경우에는 it을 사용할 수 있다. 그러면 더 간단한 람다 식을 작성할 수 있다.

람다를 변수에 저장할 때는 파라미터의 타입을 추론할 문맥이 없어서 타입을 꼭 명시해야 한다. 

```kotlin
val sum1 = { x: Int, y: Int ->
        println("sum1 실행!")
        x + y
}
```

위처럼 본문이 여러 줄인 람다에서는 마지막 줄이 결과가 된다. 

### 5.1.4 현재 영역에 있는 변수에 접근

람다를 함수 안에서 정의하면 함수의 파라미터뿐 아니라 람다 정의의 앞에 선언된 로컬 변수까지 람다에서 모두 사용할 수 있다. 

그리고 코틀린 람다에서는 파이널 변수가 아닌 변수에 접근이 가능하고 람다 바깥의 변수를 변경할 수 있다. 

람다 식 안에서 사용하는 외부 변수를 **람다가 포획한 변수**라고 부른다. 기본적으로  함수 안에 정의된 로컬 변수는 함수와 같은 생명주기를 가진다. 근데 어떤 함수가 자신의 로컬 변수를 포획한 람다를 반환하거나 다른 변수에 저장하면 로컬 변수의 생명주기가 함수의 생명주기와 달라질 수 있다. 함수가 끝난 뒤에 실행해도 람다 본문의 코드는 포획한 변수를 읽거나 쓸 수 있다.

하지만 람다를 이벤트 핸들러나 비동기적으로 실행되는 코드로 활용하는 경우 함수 호출이 끝난 다음에 로컬 변수가 변경될 수 있다.

```kotlin
fun tryToCountButtonClicks(button: Button): Int {
    var clicks = 0
    button.onClick { clicks ++ }
    return clicks
}
```

위의 함수는 항상 0을 반환하는데 이는 핸들러는 tryToCountButtonClicks가 clicks를 반환한 다음에 호출되기 때문이다. 

### 5.1.5 멤버 참조

코틀린은 자바처럼 함수를 값으로 바꿀 수 있는데 이중 콜론(::)을 사용한다. 이를 멤버 참조라고 부른다. 멤버 참조는 프로퍼니나 메서드를 단 하나만 호출하는 함수 값을 만들어준다. 

```kotlin
Person::age
// 클래스::멤버

person: Person -> person.age // Person::age 와 동일한 의미

fun salute() = println("Salute!")
run(::salute)                // 최상위 함수 참조도 가능
```

람다가 인자가 여럿인 다른 함수에 작업을 위임하는 경우 람다를 정의하지 않고 직접 위임 함수에 대한 참조를 제공하면 편리하다.

```kotlin
val action = { msg: Collection<String>, prefix: String ->
        printMessageWitjPrefix(msg, prefix)
    }

val nextAction = ::printMessageWitjPrefix
println(nextAction(errors, "Errors:"))
```

생성자 참조를 사용하면 클래스 생성 작업을 연기하거나 저장해 놓을 수 있다. 이중 콜론 뒤에 클래스 이름을 붙이면 생성자 참조를 만들 수 있다. 

```kotlin
val createPerson = ::Person        // Person 인스턴스 만드는 동작을 값으로 저장
val p = createPerson("범준", 25)
println(p)
```

## 5.2 컬렉션 함수형 API

함수형 프로그래밍 스타일을 사용하면 컬렉션 처리를 할 때 편하다. 대부분의 작업에 라이브러리 함수를 사용하고, 이로 인해 코드를 더 간결하게 작성할 수 있다.

### 5.2.1 필수적인 함수: filter와 map

filter 함수는 컬렉션을 돌면서 주어진 람다에 각 원소를 넘겨서 람다가 true를 반환하는 원소만 모은다.

```kotlin
val list = listOf(1, 2, 3, 4)
println(list.filter { it % 2 == 0 })      // 짝수만 반환
```

filter 함수는 원하지 않는 원소들을 컬렉션에서 제거한다. 하지만 filter는 원소를 반환할 수는 없다. 원소를 반환하려면 map 함수를 사용해야 한다. 

map 함수는 주어진 람다를 컬렉션의 각 원소에 적용하여 새 컬렉션을 만든다. 

```kotlin
val list = listOf(1, 2, 3, 4)
println(list.map { it * it })     // 각 원소에 제곱된 컬렉션
```

```kotlin
val people = listOf(Person("곰", 20), Person("사자", 30))
println(people.map { it.name })    // 각 원소의 이름만 갖는 컬렉션
println(people.map(Person::name))  // 멤버 참조 사용해서도 만들 수 있음!

// 실행 결과
// [곰, 사자]
// [곰, 사자]
```

필터와 변환 함수는 map에도 적용이 가능하다.

map 의 경우 키와 값을 처리하는 함수가 따로 존재한다. filterKeys와 mapKeys는 키를 걸러내거나 변환하고, filterValues와 mapValues는 값을 걸러내거나 변환한다.

### 5.2.2 all, any, count, find: 컬렉션에 술어 적용

all, any 함수는 컬렉션의 원소가 모두 조건에 만족하는지 또는 하나라도 조건에 만족하는지 확인하는데 사용할 수 있다.

count 함수는 술어를 만족하는 원소의 개수를 확인할 때 사용하고, 술어를 만족하는 원소를 찾고 싶은 경우에는 find 함수를 사용한다.

find 함수는 **가장 먼저 조건이 만족된다고 확인 된 원소를 반환**한다. 그래서 2개 이상의 원소가 조건을 만족해도 한 원소만 반환한다.

```kotlin
val canBeInClub27 = { p: Person -> p.age <= 27 }  // 조건: 나이가 27 이하
val people = listOf(Person("곰", 20), Person("사자", 30))
println(people.all(canBeInClub27))                // 사자가 27 이상이라서 false
println(people.any(canBeInClub27))                // 곰이 조건을 만족해서 true
println(people.count(canBeInClub27))              // 곰만 조건 만족해서 1
println(people.find(canBeInClub27))               // 조건을 만족하는 건 곰이라서 곰의 정보 출력
```

### 5.2.3 groupBy: 리스트를 여러 그룹으로 이뤄진 맵으로 변경

groupBy 함수는 컬렉션의 모든 원소를 원하는 기준에 따라서 여러 그룹으로 나눌 때 사용된다.

연산의 결과는 원하는 기준을 키로 갖고, 컬렉션의 원소들을 값으로 갖는 map 형식이다.

```kotlin
val people = listOf(
    Person("곰", 20),
    Person("사자", 30),
    Person("여우", 30),
    Person("호랑이", 40)
)
println(people.groupBy { it.age })

// 실행 결과
// {20=[Person(name=곰, age=20)], 30=[Person(name=사자, age=30), Person(name=여우, age=30)], 40=[Person(name=호랑이, age=40)]}
```

### 5.2.4 flatMap과 flatten: 중첩된 컬렉션 안의 원소 처리

flatMap 함수는 인자로 주어진 람다를 컬렉션의 모든 객체에 적용하고 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스트로 한데 모은다. 

```kotlin
val books = listOf(
    Book("어린 왕자", listOf("뭐시기")),
    Book("모비딕", listOf("외국 사람")),
    Book("누군가의 자서전", listOf("외국 사람")),
    Book("코틀린 인 액션", listOf("드미트리"))
)
println(books.flatMap { it.authors }.toSet())

// 실행 결과
// [뭐시기, 외국 사람, 드미트리]
```

flatMap 함수를 사용해서 책의 저자들이 원소로 있는 리스트를 만들고, toSet() 함수를 사용하여 중복을 제거한 집합으로 만들었기 때문에 실행 결과와 같은 결과를 확인할 수 있다.

## 5.3 지연 계산(lazy) 컬렉션 연산

앞에서 봤던 map이나 filter 함수는 결과 컬렉션을 즉시 생성한다. 이는 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담는다는 의미이다. 근데 시퀀스를 이용하면 중간 임시 컬렉션을 사용하지 않고 컬렉션 연산을 연쇄 할 수 있다.

```kotlin
people.map(Person::name).filter { it.startsWith("A") }
```

filter나 map 함수는 리스트를 반환한다. 이는 위 예시의 연쇄 호출이 리스트를 2개 만든다는 의미이다. 원본 리스트의 원소가 적으면 상관 없는데 많은 경우에는 효율이 떨어지기 때문에 시퀀스를 사용하는 것이 더 좋다.

```kotlin
people.asSequence()     // 원본 컬렉션을 시퀀스로 변경
    .map(Person::name)
    .filter { it.startsWith("A") }
    .toList()          // 시퀀스를 다시 리스트로 변경
```

시퀀스의 원소는 필요할 때 계산된다. 그래서 중간 값을 저장하지 않고 효율적으로 연산을 연쇄적으로 처리할 수 있다. 

> 큰 컬렉션에 대해 연쇄 연산을 처리하는 경우에는 시퀀스를 사용하는 것이 좋다.
> 

### 5.3.1 시퀀스 연산 실행: 중간 연산과 최종 연산

시퀀스에 대한 연산은 중간, 최종 연산으로 나뉜다. 중간 연산은 다른 시퀀스를 반환하고, 최종 연산은 결과를 반환한다. 

```kotlin
people.asSequence()     
    .map(Person::name)               // 중간 연산
    .filter { it.startsWith("A") }   // 중간 연산
    .toList()                        // 최종 연산
```

중간 연산은 항상 지연 계산된다. 그래서 최종 연산이 없으면 중간 연산이 실행되지 않는다. 

최종 연산이 있어야 지연되었던 중간 연산들이 실행된다.

```kotlin
people.asSequence()
    .map( Person::name)
    .filter { println("중간 연산 실행?"); it.startsWith("A") }
//    .toList()
```

그래서 위의 코드를 실행하면 아무런 결과가 나오지 않는다.

**내용 추가 필요!**

### 5.3.2 시퀀스 만들기

지금까지 본 시퀀스 예제들은 모두 asSequence() 함수를 통해서 만들었지만 generateSequence() 함수를 사용해도 된다.

## 5.4 자바 함수형 인터페이스 활용

추상 메서드가 하나만 있는 인터페이스를 **함수형 인터페이스** 또는 **SAM 인터페이스**라고 한다. 코틀린은 함수형 인터페이스를 인자로 취하는 자바 메서드를 호출할 때 람다를 넘길 수 있게 해준다. 그래서 자바 8 이전의 자바에서와 달리 setOnClickListener를 간단하게 호출할 수 있다.

```kotlin
button.setOnClickListener { view -> ... }
```

### 5.4.1 자바 메서드에 람다를 인자로 전달

위에서 언급했던 것처럼 코틀린은 함수형 인터페이스를 인자로 원하는 자바 메서드에 람다를 전달할 수 있다. 이를 대신해서 무명 객체를 명시적으로 만들어서 사용해도 된다. 

```kotlin
/* 자바 */
void postponeComputation(int delay, Runnable computation);

postponeComputation(1000) { println(42) }    // 람다 전달

postponeComputation(1000, object: Runnable {  // 무명 객체 전달
    override fun run() {
		    println(42)                  
    }
}

fun handleComputation(id: String) {
// postponeComputation 호출할 때마다 새로운 인스턴스 생성
    postponeComputation(1000) { println(42) }   
}
```

무명 객체를 사용하면 메서드를 호출할 때마다 새로운 객체가 생성되지만 람다는 같은 객체를 사용하는 차이점이 있다.

근데 람다가 주변 영역의 변수를 포획하면 매 호출 때마다 같은 객체를 사용할 수 없다. 이런 경우 컴파일러는 해당 변수를 포획한 새로운 인스턴스를 생성해준다.

### 5.4.2 SAM 생성자: 람다를 함수형 인스턴스로 명시적으로 변경

SAM 생성자는 람다를 함수형 인스턴스로 변환할 수 있게 컴파일러가 자동으로 생성한 함수다. 

```kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") }
}

createAllDoneRunnable().run()
```

위의 경우는 함수형 인터페이스의 인스턴스를 반환하는 메서드이다. 그래서 람다를 반환할 수 없어서 람다를 SAM 생성자로 감싸서 반환하였다.

SAM 생성자의 이름은 사용하려는 함수형 인터페이스이다. SAM 생성자는 그 함수형 인터페이스의 유일한 추상 메서드의 본문으로 사용할 람다만 인자로 받아서 반환한다.

람다로 생성한 함수형 인터페이스 인스턴스를 변수에 저장해야 하는 경우에도 SAM 생성자를 사용할 수 있다. 안드로이드의 경우 만약 여러 버튼에 같은 리스너를 적용하고 싶은 경우 다음과 같이 만들 수 있다.

```kotlin
val listener = OnClickListener { view -> 
    val text = when(view.id) {
        R.id.btn1 -> "btn1"
        R.id.btn2 -> "btn2"
        else -> "Unknown"
    }
    toast(text)
}
btn1.setOnClickListener(listener)
btn2.setOnClickListener(listener)
```

## 5.5 수신 객체 지정 람다: with와 apply

코틀린의 람다는 수신 객체를 명시하지 않고 람다 본문 안에서 다른 객체의 메서드를 호출할 수 있다. 이는 자바의 람다에는 없는 기능이다. 이러한 람다를 수신 객체 지정 람다라고 부른다.

### 5.5.1 with 함수

with 함수를 사용하면 어떤 객체의 이름을 반복적으로 사용하지 않으면서 해당 객체에 대한 다양한 연산을 수행할 수 있다.

```kotlin
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A' .. 'Z') {
        result.append(letter)
    }
    result.append("\nNow I know the alphabet!")
    return result.toString()
}
```

위의 함수는 result 변수에 StringBuilder를 담고, 이 변수에 대한 다양한 연산을 하고 있다. 근데 여기서 with() 함수를 사용하면 불필요하게 result를 작성하지 않아도 된다.

```kotlin
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A' .. 'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
    toString()
}
```

with() 함수는 람다 식의 본문에 있는 맨 마지막 식의 값을 반환한다.

### 5.5.2 apply 함수

apply() 함수는 with() 함수와 거의 같지만 유일한 차이는 apply() 함수는 항상 자신에게 전달된 객체를 반환한다는 점이다. 

<aside>
💡 with는 람다의 결과를 반환 / apply는 객체 자체를 반환

</aside>

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A' .. 'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}

println(alphabet().javaClass)

// 실행 결과
// class java.lang.StringBuilder
```

apply() 함수는 객체 자체를 반환하기 때문에 위의 alphabet() 함수의 반환 값의 타입을 확인해보면 StringBuilder 인 것을 확인할 수 있다.
