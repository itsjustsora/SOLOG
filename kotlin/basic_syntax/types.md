# 코틀린 타입

## 날짜
- 2026-06-28

## 🎯 오늘의 주제
- 코틀린의 nullable 타입과 non-null 타입 차이 이해
- 안전한 호출(`?.`), not-null 단언(`!!`), 엘비스 연산자(`?:`) 정리
- 자바와 함께 사용할 때 등장하는 플랫폼 타입 이해
- 코틀린의 원시 타입, 숫자 타입 변환 방식 정리
- `Any`, `Unit`, `Nothing`의 의미 이해
- 컬렉션의 nullability와 mutable/immutable 컬렉션 차이 정리

---

## 1. 코틀린 타입의 특징

코틀린의 타입은 자바와 매우 유사하지만, 가장 큰 차이는 **null을 타입 시스템에서 명확히 다룬다**는 점이다.

자바에서는 null인 객체를 잘못 사용하면 런타임에 `NullPointerException`이 발생한다.

```java
String name = null;
System.out.println(name.length()); // NullPointerException
```

그래서 자바 코드에서는 메서드 초반에 null 여부를 직접 확인하는 코드가 자주 등장한다.

```java
if (name == null) {
    return;
}
```

코틀린은 이런 문제를 줄이기 위해, 어떤 값이 null이 될 수 있는지 타입에 직접 표시한다. 즉, null 가능성을 컴파일 시점부터 검사한다.

---

## 2. 널이 될 수 있는 타입과 널이 될 수 없는 타입

코틀린에서는 null이 될 수 있는 타입과 null이 될 수 없는 타입을 구분한다.

```kotlin
val nullable: String?
val notNull: String
```

`String?`은 null이 될 수 있는 문자열이다. 물음표(`?`)는 "이 값은 없을 수도 있다"는 의미다.

```kotlin
val name: String? = null
```

반대로 `String`은 null이 될 수 없는 문자열이다.

```kotlin
val name: String = "Kotlin"
```

다음 코드는 컴파일 오류가 발생한다.

```kotlin
val name: String = null // 컴파일 오류
```

nullable 타입의 값을 non-null 타입 변수에 바로 넣을 수도 없다.

```kotlin
val nullableName: String? = null
val name: String = nullableName // 컴파일 오류
```

반대로 non-null 타입의 값은 nullable 타입 변수에 넣을 수 있다.

```kotlin
val notNullName: String = "Kotlin"
val nullableName: String? = notNullName
```

null이 될 수 없는 값은 null이 될 수 있는 자리에도 안전하게 들어갈 수 있기 때문이다.

---

## 3. 함수와 클래스에서의 nullability

nullability는 변수뿐 아니라 함수 파라미터, 반환 타입, 생성자, 프로퍼티에도 사용할 수 있다.

```kotlin
fun lengthOf(name: String): Int {
    return name.length
}
```

이 함수는 `name`이 null이 아니라고 보장한다. 따라서 함수 안에서 바로 `name.length`를 호출할 수 있다.

반면 nullable 파라미터를 받는다면 바로 메서드를 호출할 수 없다.

```kotlin
fun lengthOf(name: String?): Int {
    return name.length // 컴파일 오류
}
```

`name`이 null일 수 있기 때문에 컴파일러가 막는다.

클래스에서도 동일하게 사용할 수 있다.

```kotlin
data class Person(
    val name: String,
    val age: Int,
    val nickname: String?
)
```

`name`과 `age`는 반드시 값이 있어야 하고, `nickname`은 null일 수 있다.

---

## 4. nullable 객체 사용 방법

nullable한 객체에서 프로퍼티나 메서드를 사용하려면 null 가능성을 처리해야 한다.

```kotlin
fun main() {
    val nullablePerson: Person? =
        if (System.currentTimeMillis() % 2 == 0L) Person("Even", 22)
        else null
}
```

`nullablePerson`은 `Person?` 타입이다. 따라서 바로 `nullablePerson.age`를 호출할 수 없다.

nullable 객체를 다루는 대표적인 방법은 세 가지다.

- `if`로 null 여부를 검사한다.
- 안전한 호출 연산자 `?.`를 사용한다.
- not-null 단언 연산자 `!!`를 사용한다.

---

## 5. if 검사와 스마트 캐스트

`if`로 null이 아님을 확인하면, 해당 블록 안에서는 non-null 타입처럼 사용할 수 있다.

```kotlin
if (nullablePerson != null) {
    println("짝수 시간에만 존재하는 사람의 나이는 : ${nullablePerson.age}")
}
```

코틀린 컴파일러는 `nullablePerson != null` 검사를 보고, `if` 블록 안에서는 `nullablePerson`이 null이 아니라고 판단한다. 그래서 `nullablePerson.age`에 접근할 수 있다.

이것도 스마트 캐스트의 한 종류로 볼 수 있다. nullable 타입이 특정 범위 안에서 non-null 타입처럼 취급되는 것이다.

---

## 6. 안전한 호출 연산자

안전한 호출 연산자 `?.`는 값이 null이 아닐 때만 뒤의 프로퍼티나 메서드를 호출한다.

```kotlin
println("짝수 시간에만 존재하는 사람의 나이는 : ${nullablePerson?.age}")
```

동작 방식은 다음과 같다.

- `nullablePerson`이 null이 아니면 `nullablePerson.age` 값을 반환한다.
- `nullablePerson`이 null이면 전체 결과가 null이 된다.

즉, 다음 코드와 비슷하게 이해할 수 있다.

```kotlin
val age = if (nullablePerson != null) nullablePerson.age else null
```

실행 결과는 상황에 따라 달라진다.

```text
짝수 시간에만 존재하는 사람의 나이는 : 22
```

또는:

```text
짝수 시간에만 존재하는 사람의 나이는 : null
```

`?.`는 null 가능성이 있는 값을 안전하게 다룰 때 가장 자주 사용하는 연산자다.

---

## 7. not-null 단언 연산자

`!!`는 "이 값은 절대 null이 아니다"라고 컴파일러에게 단언하는 연산자다.

```kotlin
println("짝수 시간에만 존재하는 사람의 나이는 : ${nullablePerson!!.age}")
```

만약 실제로 값이 null이 아니라면 정상적으로 호출된다.

```text
짝수 시간에만 존재하는 사람의 나이는 : 22
```

하지만 값이 null이면 `NullPointerException`이 발생한다.

```text
Exception in thread "main" java.lang.NullPointerException
```

`!!`는 코틀린의 null safety를 개발자가 직접 우회하는 연산자다. 정말 null이 아님을 확신할 수 있는 경우가 아니라면 남용하지 않는 것이 좋다.

---

## 8. 엘비스 연산자

엘비스 연산자(`?:`)는 nullable 값이 null일 때 사용할 기본값을 지정한다.

```kotlin
fun nullableToNotNull(s: String?): String = s ?: "default string"
```

동작 방식은 다음과 같다.

- `s`가 null이 아니면 `s`를 반환한다.
- `s`가 null이면 `"default string"`을 반환한다.

다음 코드와 비슷하다.

```kotlin
fun nullableToNotNull(s: String?): String =
    if (s != null) s else "default string"
```

엘비스 연산자는 nullable 값을 non-null 값으로 바꿀 때 자주 사용한다.

```kotlin
val name: String? = getSomeNullableData()
val displayName: String = name ?: "anonymous"
```

`?:` 뒤에는 단순한 값뿐 아니라 예외도 사용할 수 있다.

```kotlin
val name = nullableName ?: throw IllegalArgumentException("name is required")
```

---

## 9. 플랫폼 타입

코틀린이 자바 코드나 자바 라이브러리에서 가져온 값의 null 여부를 정확히 알 수 없는 경우가 있다. 이런 타입을 **플랫폼 타입(Platform Type)** 이라고 한다.

자바 코드는 기본적으로 타입에 null 가능성을 명확히 표현하지 않는다.

```java
public class JavaPerson {
    public String getName() {
        return name;
    }
}
```

코틀린 입장에서는 `getName()`이 null을 반환할 수 있는지 알 수 없다. 그래서 개발자가 직접 nullable 또는 non-null로 받아야 한다.

```kotlin
val name1: String? = javaPerson.name
val name2: String = javaPerson.name
```

두 코드 모두 가능하다. 하지만 `name2`처럼 non-null로 받았는데 실제 값이 null이면 런타임에 문제가 생길 수 있다.

자바 코드에 `@NotNull`, `@Nullable` 같은 어노테이션이 붙어 있으면 코틀린이 nullability를 더 정확히 인식할 수 있다.

```java
@NotNull
public String getName() {
    return name;
}

@Nullable
public String getNickname() {
    return nickname;
}
```

플랫폼 타입을 사용할 때는 자바 API 문서나 실제 구현을 확인하고, 확실하지 않다면 nullable로 받아 안전하게 처리하는 편이 좋다.

---

## 10. 코틀린의 원시 타입

자바에는 원시 타입과 래퍼 타입이 구분된다.

```java
int i = 1;
Integer j = 1;
```

코틀린에서는 코드상으로 원시 타입과 래퍼 타입을 구분하지 않는다.

```kotlin
val i: Int = 1
val b: Boolean = true
val d: Double = 3.14
```

코틀린은 JVM에서 동작하므로 최종적으로 자바 클래스 파일을 만든다. 이때 상황에 따라 원시 타입이나 래퍼 타입을 적절히 사용한다.

일반적으로 특별한 이유가 없으면 JVM 원시 타입으로 처리된다.

```kotlin
val i: Int = 1
```

하지만 nullable 타입이거나 제네릭에 들어가면 래퍼 타입이 필요하다.

```kotlin
val nullableInt: Int? = null
val numbers: List<Int> = listOf(1, 2, 3)
```

원시 타입은 null을 담을 수 없고, 제네릭은 객체 타입을 필요로 하기 때문이다.

---

## 11. 숫자의 타입 변환

자바에서는 작은 숫자 타입에서 큰 숫자 타입으로 자동 변환이 일어나는 경우가 있다.

```java
int i = 1;
long l = i;
```

코틀린은 숫자 타입을 자동으로 변환하지 않는다.

```kotlin
val i: Int = 1
val l: Long = i // 컴파일 오류
```

직접 변환 함수를 호출해야 한다.

```kotlin
val i: Int = 1
val l: Long = i.toLong()
```

리터럴에 `L`을 붙이면 `Long` 타입 값이 된다.

```kotlin
val l: Long = 1L
```

코틀린은 다양한 숫자 변환 함수를 제공한다.

- `toByte()`
- `toShort()`
- `toInt()`
- `toLong()`
- `toFloat()`
- `toDouble()`
- `toChar()`

자동 변환을 허용하지 않는 이유는 의도하지 않은 타입 변환으로 인한 버그를 줄이기 위해서다.

---

## 12. Any와 Any?

`Any`는 코틀린의 최상위 타입이다. 자바의 `Object`와 비슷하게 볼 수 있다.

```kotlin
val value: Any = "Kotlin"
```

`Any`에는 null이 들어갈 수 없다.

```kotlin
val value: Any = null // 컴파일 오류
```

null까지 포함한 모든 값을 담고 싶다면 `Any?`를 사용한다.

```kotlin
val value: Any? = null
```

정리하면 다음과 같다.

- `Any`: null을 제외한 모든 타입의 최상위 타입
- `Any?`: null을 포함한 모든 값을 받을 수 있는 타입

---

## 13. Unit

`Unit`은 자바의 `void`와 비슷한 역할을 한다.

```kotlin
fun printSomething(): Unit {
    println("Something")
}
```

반환할 값이 없다는 의미다. 다만 코틀린에서는 `Unit`도 하나의 타입이다.

대부분의 경우 `Unit`은 생략한다.

```kotlin
fun printSomething() {
    println("Something")
}
```

실무 코드에서는 반환 타입이 `Unit`인 경우 명시하지 않는 형태를 더 자주 볼 수 있다.

---

## 14. Nothing

`Nothing`은 함수가 정상적으로 끝나지 않는다는 것을 나타내는 타입이다.

```kotlin
fun fail(message: String): Nothing {
    throw IllegalStateException(message)
}
```

이 함수는 항상 예외를 던진다. 정상적인 값을 반환하는 경우가 없다.

`Nothing`은 엘비스 연산자와 함께 사용할 때 유용하다.

```kotlin
val name = persons.find { it.age > 50 } ?: fail("50살이 넘는 사람이 없음")
```

`persons.find { it.age > 50 }`의 결과가 null이면 `fail()`이 호출되고 예외가 발생한다. null이 아니면 찾은 값이 `name`에 들어간다.

`fail()`은 정상적으로 값을 반환하지 않기 때문에 `Nothing` 타입으로 선언할 수 있다. 컴파일러는 이를 보고 `fail()` 이후에는 정상 흐름이 이어지지 않는다고 이해한다.

---

## 15. 컬렉션과 nullability

코틀린에서는 컬렉션 자체가 nullable일 수도 있고, 컬렉션 안의 원소가 nullable일 수도 있다.

```kotlin
val list: List<Int>
val listOfNullable: List<Int?>
val nullableList: List<Int>?
val nullableListOfNullable: List<Int?>?
```

각 타입의 의미는 다음과 같다.

- `List<Int>`: 리스트도 null이 아니고, 원소도 null이 아니다.
- `List<Int?>`: 리스트는 null이 아니지만, 원소는 null일 수 있다.
- `List<Int>?`: 리스트 자체가 null일 수 있지만, 원소는 null이 아니다.
- `List<Int?>?`: 리스트도 null일 수 있고, 원소도 null일 수 있다.

물음표가 어디에 붙어 있는지에 따라 의미가 달라진다.

```kotlin
val numbers: List<Int?> = listOf(1, null, 3)
val nullableNumbers: List<Int>? = null
```

위 두 타입은 완전히 다른 의미다.

---

## 16. 수정 가능한 컬렉션과 수정 불가능한 컬렉션

코틀린 컬렉션은 읽기 전용 컬렉션과 변경 가능한 컬렉션을 구분한다.

읽기 전용 컬렉션은 `Collection` 계열 인터페이스를 사용한다.

```kotlin
val numbers: List<Int> = listOf(1, 2, 3)
```

읽기 전용 컬렉션은 다음과 같은 기능을 제공한다.

- `get`
- `size`
- `iterator`
- `contains`

하지만 `add`, `remove`, `clear` 같은 변경 함수는 제공하지 않는다.

```kotlin
numbers.add(4) // 컴파일 오류
```

읽기 전용 컬렉션은 다음 함수로 만들 수 있다.

```kotlin
listOf(1, 2, 3)
setOf(1, 2, 3)
mapOf(1 to "one", 2 to "two")
```

변경 가능한 컬렉션은 `MutableCollection` 계열 인터페이스를 사용한다.

```kotlin
val numbers = mutableListOf(1, 2, 3)
```

변경 가능한 컬렉션은 읽기 기능에 더해 추가, 삭제, 수정 기능을 제공한다.

```kotlin
numbers.add(4)
numbers.remove(1)
numbers.clear()
```

변경 가능한 컬렉션은 다음 함수로 만들 수 있다.

```kotlin
mutableListOf(1, 2, 3)
mutableSetOf(1, 2, 3)
mutableMapOf(1 to "one", 2 to "two")
```

첫 번째 강의에서 변수는 가능하면 `val`을 먼저 사용하라고 했던 것처럼, 컬렉션도 가능하면 읽기 전용 컬렉션을 먼저 고려하는 것이 좋다. 변경이 꼭 필요한 경우에만 mutable 컬렉션을 사용하면 코드의 상태 변화를 줄일 수 있다.

---

## 17. 정리

코틀린 타입에서 이번 강의의 핵심은 다음과 같다.

- 코틀린은 타입에 null 가능성을 명시해 컴파일 시점부터 null을 관리한다.
- `String`은 null이 될 수 없고, `String?`은 null이 될 수 있다.
- nullable 타입은 non-null 타입 변수에 바로 대입할 수 없다.
- nullable 객체는 `if` 검사, `?.`, `!!` 등을 통해 사용한다.
- `?.`는 값이 null이면 전체 결과를 null로 만든다.
- `!!`는 null이 아님을 단언하지만, 실제 null이면 NPE가 발생한다.
- 엘비스 연산자 `?:`는 null일 때 사용할 기본값을 지정한다.
- 자바에서 가져온 플랫폼 타입은 null 여부가 불명확하므로 주의해야 한다.
- 코틀린은 코드상에서 원시 타입과 래퍼 타입을 구분하지 않는다.
- 숫자 타입은 자동 변환되지 않으므로 `toLong()`, `toInt()` 같은 변환 함수를 사용한다.
- `Any`는 null을 제외한 최상위 타입이고, `Any?`는 null을 포함한 모든 값을 받을 수 있다.
- `Unit`은 자바의 `void`와 비슷하고, `Nothing`은 정상적으로 끝나지 않는 함수를 나타낸다.
- 컬렉션은 컬렉션 자체의 nullability와 원소의 nullability를 구분해야 한다.
- 가능하면 읽기 전용 컬렉션을 먼저 고려하고, 변경이 필요할 때 mutable 컬렉션을 사용한다.
