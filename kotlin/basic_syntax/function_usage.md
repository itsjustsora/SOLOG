# 코틀린 함수 활용

## 날짜
- 2026-06-28

## 🎯 오늘의 주제
- 이름 붙인 인자(named argument)를 사용해 함수 호출을 명확하게 만드는 방법 이해
- 디폴트 파라미터 값으로 불필요한 오버로딩을 줄이는 방식 정리
- 최상위 함수와 프로퍼티가 자바의 유틸 클래스 역할을 대체하는 방식 이해
- 확장 함수와 확장 프로퍼티의 개념 정리
- 가변 인자 함수, 중위 호출, 구조 분해 선언 활용 방식 이해

---

## 1. 코틀린 함수의 기본 모습

코틀린에서 함수는 `fun` 키워드로 선언한다.

```kotlin
fun getMyName(
    firstName: String,
    lastName: String
): String = "$firstName $lastName"
```

이 함수는 `firstName`과 `lastName`을 받아 하나의 문자열로 합쳐 반환한다. 본문이 하나의 표현식으로 끝나기 때문에 **Expression Body Function** 형태로 작성할 수 있다.

```kotlin
fun getMyName(firstName: String, lastName: String): String {
    return "$firstName $lastName"
}
```

위처럼 중괄호와 `return`을 사용하는 **Block Body Function**으로도 작성할 수 있지만, 단순한 반환 함수라면 expression body가 더 간결하다.

---

## 2. 이름 붙인 인자

자바에서는 생성자나 메서드를 호출할 때 파라미터 순서를 정확히 기억해야 한다.

코틀린에서도 일반적인 호출은 순서를 기준으로 한다.

```kotlin
fun getMyName(
    firstName: String,
    lastName: String
): String = "$firstName $lastName"

fun main() {
    println(getMyName("Steve", "Jobs"))
    println(getMyName("Jobs", "Steve"))
}
```

두 번째 호출은 문법적으로는 문제가 없지만, 의도와 다르게 `firstName`에 `"Jobs"`, `lastName`에 `"Steve"`가 들어간다. 파라미터가 많아질수록 이런 실수가 생기기 쉽다.

코틀린에서는 호출할 때 인자에 이름을 붙일 수 있다. 이를 **이름 붙인 인자(Named Argument)** 라고 한다.

```kotlin
fun main() {
    println(getMyName(firstName = "Steve", lastName = "Jobs"))
}
```

이렇게 작성하면 어떤 값이 어떤 파라미터에 들어가는지 코드만 보고 바로 알 수 있다.

```kotlin
println(getMyName(lastName = "Jobs", firstName = "Steve"))
```

이름을 붙이면 순서를 바꿔 호출할 수도 있다. 다만 읽기 쉬운 코드를 위해 특별한 이유가 없다면 함수 선언 순서와 비슷하게 작성하는 편이 좋다.

---

## 3. 디폴트 파라미터 값

코틀린 함수는 파라미터에 기본값을 지정할 수 있다. 이를 **디폴트 파라미터 값(Default Parameter Value)** 이라고 한다.

```kotlin
fun getMyName(
    firstName: String = "foo",
    lastName: String = "bar"
): String = "$firstName $lastName"
```

기본값이 있으면 호출할 때 해당 인자를 생략할 수 있다.

```kotlin
fun main() {
    println(getMyName())
    println(getMyName("Steve"))
    println(getMyName(lastName = "Jobs"))
}
```

실행 결과는 다음과 같다.

```text
foo bar
Steve bar
foo Jobs
```

파라미터 없이 호출하면 모든 값이 기본값으로 채워진다. 첫 번째 인자만 넘기면 `firstName`만 `"Steve"`로 바뀌고, `lastName`은 기본값 `"bar"`를 사용한다. 특정 파라미터만 바꾸고 싶다면 이름 붙인 인자를 함께 사용하면 된다.

자바에서는 파라미터 조합이 조금씩 다르면 오버로딩된 메서드를 여러 개 만드는 경우가 많다.

```java
String getMyName() { ... }
String getMyName(String firstName) { ... }
String getMyName(String firstName, String lastName) { ... }
```

코틀린에서는 디폴트 파라미터 값을 사용하면 하나의 함수로 여러 호출 형태를 처리할 수 있다. 덕분에 불필요한 오버로딩을 줄일 수 있다.

---

## 4. 최상위 함수와 프로퍼티

코틀린은 함수와 프로퍼티를 클래스 밖, 즉 파일의 최상위에 선언할 수 있다.

```kotlin
package com.inflearn.math

const val ONE = 1
const val TWO = 2

fun sum(num1: Int, num2: Int) = num1 + num2
```

이처럼 클래스 안에 들어 있지 않은 함수는 **Top-Level Function**, 클래스 밖에 선언한 프로퍼티는 **Top-Level Property**라고 한다.

자바에서는 이런 코드를 작성하려면 보통 유틸 클래스를 만든다.

```java
package com.inflearn.math;

public final class NumberUtils {
    public static final Integer ONE = 1;
    public static final Integer TWO = 2;

    private NumberUtils() {
    }

    public static Integer sum(Integer num1, Integer num2) {
        return num1 + num2;
    }
}
```

자바에서는 정적 함수와 상수를 담기 위해 클래스를 만들고, 인스턴스 생성을 막기 위해 private 생성자까지 두는 경우가 많다. 코틀린에서는 최상위 함수와 프로퍼티를 사용하면 이런 불필요한 감싸기 코드를 줄일 수 있다.

---

## 5. const val

최상위 프로퍼티 예제에서 `const val`을 사용했다.

```kotlin
const val ONE = 1
```

`const val`은 컴파일 시점 상수를 의미한다. 일반적인 `val`은 한 번만 할당되는 읽기 전용 값이고, `const val`은 컴파일 시점에 값이 확정되는 상수다.

`const val`은 다음과 같은 제한이 있다.

- 최상위 또는 `object` 내부에 선언해야 한다.
- `String`이나 기본 타입처럼 컴파일 시점에 알 수 있는 값만 사용할 수 있다.
- 함수 호출 결과처럼 실행 시점에 계산되는 값은 사용할 수 없다.

간단한 상수 값은 `const val`로 선언하면 자바의 `public static final` 상수와 비슷한 방식으로 사용할 수 있다.

---

## 6. 확장 함수

코틀린에서는 기존 클래스에 함수를 추가한 것처럼 사용할 수 있다. 이를 **확장 함수(Extension Function)** 라고 한다.

```kotlin
fun String.double() = this + this

fun main() {
    println("do".double())
}
```

실행 결과는 다음과 같다.

```text
dodo
```

`String.double()`은 `String` 클래스에 `double`이라는 함수를 추가한 것처럼 보인다. 함수 안에서 `this`는 확장 대상 객체를 의미한다. 위 예제에서는 `"do"`가 `this`다.

자바에서는 라이브러리 클래스에 메서드를 추가하고 싶을 때 보통 상속을 고려하거나 별도의 유틸 메서드를 만든다. 하지만 상속할 수 없는 클래스도 있고, 단순한 기능을 위해 유틸 클래스를 계속 만드는 것도 번거롭다.

코틀린의 확장 함수를 사용하면 기존 클래스 코드를 수정하지 않고도 자연스럽게 기능을 추가한 것처럼 호출할 수 있다.

---

## 7. 확장 함수의 한계

확장 함수는 실제로 클래스 내부에 메서드를 추가하는 기능은 아니다.

내부적으로는 확장 대상 객체를 첫 번째 인자로 받는 정적 메서드에 가깝게 동작한다.

```kotlin
fun String.double() = this + this
```

개념적으로는 다음과 비슷하게 이해할 수 있다.

```kotlin
fun double(str: String) = str + str
```

그래서 확장 함수는 오버라이드할 수 없다. 클래스의 진짜 멤버 함수처럼 다형적으로 동작하는 것이 아니라, 컴파일 시점의 타입을 기준으로 호출될 함수가 결정된다.

확장 함수는 기존 타입을 편리하게 사용하는 문법이지, 기존 클래스의 상속 구조 자체를 바꾸는 기능은 아니다.

---

## 8. 확장 프로퍼티

함수뿐 아니라 프로퍼티도 확장할 수 있다. 이를 **확장 프로퍼티(Extension Property)** 라고 한다.

```kotlin
val String.lastChar: Char
    get() = get(this.length - 1)
```

기본적인 프로퍼티 문법과 비슷하지만, 프로퍼티 이름 앞에 확장 대상 타입인 `String`을 붙인다.

```kotlin
fun main() {
    println("Kotlin".lastChar)
}
```

실행 결과는 다음과 같다.

```text
n
```

확장 프로퍼티는 실제 필드를 추가하는 것이 아니다. 위 예제처럼 값을 계산해서 반환하는 getter를 제공하는 방식이다.

---

## 9. 컬렉션과 확장 함수

코틀린 컬렉션에서 자주 사용하는 여러 함수도 확장 함수로 제공된다.

```kotlin
println(setOf(1, 23, 45, 3).maxOrNull())
println(listOf("ab", "bc", "cd").last())
```

`maxOrNull()`은 컬렉션에서 가장 큰 값을 반환하고, 컬렉션이 비어 있으면 `null`을 반환한다. `last()`는 리스트의 마지막 요소를 반환한다.

`Set`이나 `List` 클래스 자체를 직접 수정하지 않아도, 코틀린 표준 라이브러리가 확장 함수를 통해 다양한 기능을 제공한다. 그래서 컬렉션을 다룰 때 체이닝 형태로 자연스럽게 함수를 호출할 수 있다.

---

## 10. 가변 인자 함수

가변 인자 함수는 인자를 여러 개 받을 수 있는 함수다. 코틀린에서는 `vararg` 키워드를 사용한다.

```kotlin
public fun <T> setOf(vararg elements: T): Set<T>

public fun <T> listOf(vararg elements: T): List<T>
```

`setOf`와 `listOf`는 인자를 원하는 개수만큼 받을 수 있다.

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val names = setOf("Jack", "Diana", "Frost")
```

`vararg elements: T`는 `T` 타입의 값을 여러 개 받을 수 있다는 의미다.

직접 가변 인자 함수를 만들 수도 있다.

```kotlin
fun printAll(vararg messages: String) {
    for (message in messages) {
        println(message)
    }
}

fun main() {
    printAll("hello", "kotlin", "function")
}
```

가변 인자는 호출하는 쪽에서 배열처럼 보이지 않고 여러 인자를 나열하는 형태로 사용할 수 있어서 편리하다.

---

## 11. 중위 호출

코틀린에서는 특정 함수를 점(`.`)과 괄호 없이 호출할 수 있다. 이를 **중위 호출(Infix Call)** 이라고 한다.

```kotlin
infix fun String.add(postfix: String) = this + postfix

fun main() {
    println("This is very ".add("good"))
    println("This is very " add "good")
}
```

두 호출은 같은 결과를 만든다.

```text
This is very good
This is very good
```

`infix` 함수는 일반 함수처럼 호출할 수도 있고, 중위 호출 형태로 호출할 수도 있다.

중위 호출을 사용하려면 보통 다음 조건을 만족해야 한다.

- `infix` 키워드가 붙어 있어야 한다.
- 멤버 함수 또는 확장 함수여야 한다.
- 파라미터가 하나여야 한다.

중위 호출은 코드 길이를 줄이고 자연어처럼 읽히는 호출을 만들 수 있다. 다만 모든 함수에 쓰기보다, 읽었을 때 의미가 명확한 경우에 사용하는 것이 좋다.

---

## 12. to와 mapOf

중위 호출의 대표적인 예시는 `to` 함수다.

```kotlin
println(mapOf("key" to "value", "k2" to "v2"))
```

위 코드는 다음과 같은 방식으로도 작성할 수 있다.

```kotlin
println(mapOf("key".to("value"), "k2".to("v2")))
```

또는 `Pair`를 직접 만들 수도 있다.

```kotlin
println(mapOf(Pair("key", "value"), Pair("k2", "v2")))
```

`to` 함수는 내부적으로 `Pair`를 만든다.

```kotlin
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

그리고 `mapOf`는 `Pair`를 가변 인자로 받는다.

```kotlin
public fun <K, V> mapOf(vararg pairs: Pair<K, V>): Map<K, V>
```

즉, 다음 코드는:

```kotlin
mapOf("key" to "value", "k2" to "v2")
```

다음 의미로 이해할 수 있다.

- `"key" to "value"`가 `Pair("key", "value")`를 만든다.
- `"k2" to "v2"`가 `Pair("k2", "v2")`를 만든다.
- `mapOf`가 여러 개의 `Pair`를 받아 Map을 만든다.

---

## 13. 구조 분해 선언

구조 분해 선언(Destructuring Declaration)은 하나의 객체가 가진 여러 값을 여러 변수로 한 번에 나누어 담는 문법이다.

```kotlin
val person = Pair("Snow", true)

val (name, isMarried) = person
```

`person`은 `Pair<String, Boolean>`이고, 첫 번째 값은 `"Snow"`, 두 번째 값은 `true`다.

- `name`에는 `"Snow"`가 들어간다.
- `isMarried`에는 `true`가 들어간다.

구조 분해 선언은 `Pair`나 `Triple`처럼 여러 값을 담는 타입에서 자주 사용한다.

---

## 14. Map 순회와 구조 분해

Map을 순회할 때도 구조 분해 선언을 사용할 수 있다.

```kotlin
val persons = mapOf(
    "Snow" to true,
    "Peter" to false,
    "Paul" to true,
)
```

`forEach`에서 각 항목을 바로 분해해서 사용할 수 있다.

```kotlin
persons.forEach { (name, isMarried) ->
    println("$name is married($isMarried)")
}
```

Map의 각 항목은 key와 value를 가진다. 위 코드에서는 key가 `name`, value가 `isMarried`에 들어간다.

만약 특정 값이 필요 없다면 언더스코어(`_`)를 사용할 수 있다.

```kotlin
persons.forEach { (name, _) ->
    println("$name is in this list")
}
```

`_`는 이 값은 사용하지 않겠다는 의미다. 위 예제에서는 `isMarried` 값이 필요 없으므로 이름만 꺼내 사용한다.

구조 분해 선언을 사용하면 컬렉션에 담긴 데이터를 바로 의미 있는 변수 이름으로 받을 수 있어 가독성이 좋아진다.

---

## 15. 정리

코틀린 함수 활용에서 이번 강의의 핵심은 다음과 같다.

- 이름 붙인 인자를 사용하면 파라미터 순서로 인한 실수를 줄일 수 있다.
- 디폴트 파라미터 값을 사용하면 불필요한 오버로딩을 줄일 수 있다.
- 최상위 함수와 프로퍼티를 사용하면 자바의 유틸 클래스처럼 불필요한 클래스를 만들지 않아도 된다.
- `const val`은 컴파일 시점에 값이 확정되는 상수를 선언할 때 사용한다.
- 확장 함수는 기존 클래스에 함수를 추가한 것처럼 호출할 수 있게 해준다.
- 확장 함수는 실제 멤버 함수가 아니기 때문에 오버라이드할 수 없다.
- 확장 프로퍼티는 기존 타입에 계산된 프로퍼티를 추가한 것처럼 사용할 수 있다.
- `vararg`를 사용하면 가변 인자 함수를 만들 수 있다.
- `infix` 함수는 점과 괄호 없이 중위 호출 형태로 사용할 수 있다.
- `to`는 `Pair`를 만드는 infix 확장 함수이고, `mapOf`는 여러 `Pair`를 받아 Map을 만든다.
- 구조 분해 선언을 사용하면 여러 값을 여러 변수로 한 번에 나누어 받을 수 있다.
