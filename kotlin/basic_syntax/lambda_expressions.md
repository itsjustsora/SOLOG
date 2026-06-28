# 코틀린 람다식

## 날짜
- 2026-06-28

## 🎯 오늘의 주제
- 람다가 무엇인지, 왜 사용하는지 이해
- 코틀린 람다식의 기본 문법 정리
- 람다식을 함수 인자로 넘기거나 변수에 저장하는 방식 이해
- 람다식 간결화 규칙과 `it` 사용 방식 정리
- 멤버 참조와 컬렉션 함수형 API 활용 방식 이해
- `sequence`가 일반 컬렉션 연산과 다르게 동작하는 방식 정리

---

## 1. 람다란 무엇인가

람다는 기본적으로 **작은 코드 조각**이다. 보통 함수에 로직을 넘길 때 사용한다.

예를 들어 버튼을 클릭했을 때 실행할 동작을 생각해보자. 버튼 자체는 클릭되었을 때 무엇을 해야 하는지 모른다. 그래서 개발자가 "클릭되면 이 코드를 실행해줘"라는 동작을 넘겨야 한다.

람다가 없던 구형 자바에서는 이런 동작 하나를 넘기기 위해 익명 클래스를 만들어야 했다.

```java
button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View view) {
        // 클릭 시 수행할 동작
    }
});
```

클릭 시 수행할 동작 하나를 전달하기 위해 인터페이스를 구현하는 익명 클래스를 만들어야 하므로 코드가 길고 복잡하다.

Java 8부터는 자바에도 람다식이 도입되었다.

```java
button.setOnClickListener(view -> {
    // 클릭 시 수행할 동작
});
```

코틀린은 이 문법을 더 간결하게 제공한다.

```kotlin
button.setOnClickListener {
    // 클릭 시 수행할 동작
}
```

즉, 람다는 "나중에 실행할 코드 조각"을 값처럼 넘길 수 있게 해주는 문법이다.

---

## 2. 코틀린 람다식의 장점

람다식은 코틀린 컬렉션을 다룰 때 매우 자주 사용된다.

```kotlin
data class Person(
    val name: String,
    val age: Int
)

val persons = listOf(
    Person("Captain", 44),
    Person("Cyclops", 35),
    Person("Deadpool", 31),
    Person("Iceman", 54),
)
```

가장 나이가 많은 사람을 찾고 싶다면 다음처럼 작성할 수 있다.

```kotlin
fun main() {
    println(persons.maxByOrNull { it.age })
}
```

실행 결과는 다음과 같다.

```text
Person(name=Iceman, age=54)
```

`maxByOrNull`은 컬렉션에서 어떤 값을 기준으로 최댓값을 찾을지 알아야 한다. `{ it.age }`는 각 사람의 나이를 기준으로 비교하겠다는 뜻이다.

람다를 사용하면 반복문을 직접 작성하지 않아도 "무엇을 기준으로 처리할지"만 간결하게 전달할 수 있다.

---

## 3. 람다식의 기본 문법

먼저 일반 함수를 보자.

```kotlin
fun sum(x: Int, y: Int): Int = x + y
```

이 함수를 람다식으로 표현하면 다음과 같다.

```kotlin
val sumLambda = { x: Int, y: Int -> x + y }
```

람다식은 중괄호(`{}`)로 감싼다. 화살표(`->`) 왼쪽에는 파라미터를 쓰고, 오른쪽에는 실행할 코드를 쓴다.

```kotlin
{ x: Int, y: Int -> x + y }
```

구조를 나누면 다음과 같다.

- `x: Int, y: Int`: 람다가 받는 파라미터
- `->`: 파라미터와 본문을 구분하는 기호
- `x + y`: 람다의 본문이자 결과값

람다식의 마지막 값이 람다의 결과가 된다. 위 람다에서는 `x + y`가 결과값이다.

---

## 4. 람다를 사용하는 방법

람다는 크게 두 가지 방식으로 사용할 수 있다.

- 함수에 파라미터로 람다식을 넘긴다.
- 람다식을 변수에 저장하고 나중에 실행한다.

```kotlin
fun sum(x: Int, y: Int): Int = x + y

val sumLambda = { x: Int, y: Int -> x + y }

fun main() {
    println(sum(12, 34))
    println(sumLambda(12, 34))
    println({ x: Int, y: Int -> x + y }(12, 34))
}
```

첫 번째는 일반 함수 호출이다.

```kotlin
sum(12, 34)
```

두 번째는 변수에 저장된 람다를 호출하는 방식이다.

```kotlin
sumLambda(12, 34)
```

세 번째는 람다식을 만들자마자 바로 실행하는 방식이다.

```kotlin
{ x: Int, y: Int -> x + y }(12, 34)
```

실무에서는 세 번째처럼 즉시 실행하는 형태보다, 함수의 인자로 람다를 넘기거나 변수에 저장해 사용하는 경우가 더 많다.

---

## 5. 람다식 간결화 과정

코틀린 람다식은 여러 단계로 간결하게 만들 수 있다.

가장 정석적인 호출은 람다식을 함수 호출 괄호 안에 넣는 형태다.

```kotlin
println(persons.maxByOrNull({ person: Person -> person.age }))
```

함수의 마지막 인자가 람다식이면, 람다를 괄호 밖으로 뺄 수 있다.

```kotlin
println(persons.maxByOrNull() { person: Person -> person.age })
```

람다가 유일한 인자라면 빈 괄호도 생략할 수 있다.

```kotlin
println(persons.maxByOrNull { person: Person -> person.age })
```

컴파일러가 타입을 추론할 수 있다면 파라미터 타입도 생략할 수 있다.

```kotlin
println(persons.maxByOrNull { person -> person.age })
```

람다의 파라미터가 하나뿐이라면 기본 이름인 `it`을 사용할 수 있다.

```kotlin
println(persons.maxByOrNull { it.age })
```

이렇게 최종적으로 매우 간결한 코드가 된다.

---

## 6. it을 사용할 때 주의할 점

`it`은 단일 파라미터 람다에서 사용할 수 있는 기본 파라미터 이름이다.

```kotlin
persons.maxByOrNull { it.age }
```

이 코드는 `it`이 각 `Person` 객체를 의미한다는 것을 컴파일러가 추론할 수 있기 때문에 가능하다.

하지만 항상 `it`이 좋은 것은 아니다.

- 람다 파라미터가 여러 개이면 `it`을 사용할 수 없다.
- 람다식이 길어지면 `it`이 무엇을 의미하는지 헷갈릴 수 있다.
- 람다가 중첩되면 안쪽과 바깥쪽의 `it`을 구분하기 어려워진다.
- 컴파일러가 타입을 추론하기 어려운 상황에서는 명시적인 이름과 타입이 필요할 수 있다.

간단한 단일 파라미터 람다에서는 `it`이 편리하다. 하지만 코드가 길어지거나 의미를 분명히 드러내고 싶다면 직접 이름을 붙이는 편이 낫다.

```kotlin
persons.maxByOrNull { person -> person.age }
```

---

## 7. 멤버 참조

람다식이 이미 존재하는 프로퍼티나 함수를 그대로 가리키는 경우에는 **멤버 참조(Member Reference)** 를 사용할 수 있다.

```kotlin
println(persons.maxByOrNull { person -> person.age })
```

위 코드는 `Person`의 `age` 프로퍼티를 기준으로 최댓값을 찾는다. 이때 `Person::age`로 더 간결하게 표현할 수 있다.

```kotlin
println(persons.maxByOrNull(Person::age))
```

이중 콜론(`::`)은 특정 클래스의 프로퍼티나 메서드를 참조할 때 사용한다.

```kotlin
Person::age
```

최상위 함수나 최상위 값은 클래스 이름 없이 바로 참조할 수 있다.

```kotlin
fun isAdult(person: Person) = person.age >= 20

println(persons.filter(::isAdult))
```

멤버 참조는 이미 정의된 함수를 그대로 넘길 때 유용하다. 람다 안에서 단순히 다른 함수나 프로퍼티를 호출하는 경우라면 멤버 참조로 더 간결하게 표현할 수 있다.

---

## 8. 컬렉션 함수형 API

코틀린 컬렉션은 람다와 함께 사용하는 함수형 API를 많이 제공한다.

이미 정의된 컬렉션 함수형 API만 잘 활용해도 대부분의 단순 반복 처리는 `for`, `while` 없이 구현할 수 있다.

대표적인 함수는 다음과 같다.

- `filter`: 조건에 맞는 데이터만 남긴다.
- `map`: 데이터를 다른 형태로 변환한다.
- `all`: 모든 원소가 조건을 만족하는지 확인한다.
- `any`: 하나라도 조건을 만족하는 원소가 있는지 확인한다.
- `count`: 조건을 만족하는 원소 개수를 센다.
- `find`: 조건을 만족하는 첫 번째 원소를 찾고, 없으면 `null`을 반환한다.
- `groupBy`: 특정 값을 기준으로 원소를 그룹화한다.
- `flatMap`: 각 원소를 컬렉션으로 변환한 뒤 하나의 컬렉션으로 펼친다.

---

## 9. filter와 map

`filter`와 `map`은 가장 기본이 되는 컬렉션 함수다.

```kotlin
val persons = listOf(
    Person("Captain", 44),
    Person("Cyclops", 35),
    Person("Deadpool", 31),
    Person("Iceman", 54),
)
```

나이가 36세 초과인 사람만 필터링할 수 있다.

```kotlin
fun main() {
    println(persons.filter { it.age > 36 })
}
```

실행 결과는 다음과 같다.

```text
[Person(name=Captain, age=44), Person(name=Iceman, age=54)]
```

필터링한 뒤 다른 문자열 형태로 변환할 수도 있다.

```kotlin
fun main() {
    println(
        persons
            .filter { it.age > 36 }
            .map { "${it.name}'s age is ${it.age}" }
    )
}
```

실행 결과는 다음과 같다.

```text
[Captain's age is 44, Iceman's age is 54]
```

`filter`는 원소를 걸러내고, `map`은 원소를 다른 값으로 바꾼다. 둘을 연결하면 "조건에 맞는 데이터만 골라서 원하는 형태로 변환"하는 코드를 간결하게 작성할 수 있다.

---

## 10. all, any, count, find

조건을 검사하거나 원소를 찾을 때 자주 사용하는 함수들도 있다.

```kotlin
val persons = listOf(
    Person("Captain", 44),
    Person("Cyclops", 35),
    Person("Deadpool", 31),
    Person("Iceman", 54),
)
```

`all`은 모든 원소가 조건을 만족하는지 확인한다.

```kotlin
println(persons.all { it.age >= 30 })
```

`any`는 하나라도 조건을 만족하는 원소가 있는지 확인한다.

```kotlin
println(persons.any { it.age > 50 })
```

`count`는 조건을 만족하는 원소의 개수를 센다.

```kotlin
println(persons.count { it.age > 36 })
```

`find`는 조건을 만족하는 첫 번째 원소를 반환한다. 없으면 `null`을 반환한다.

```kotlin
println(persons.find { it.name.startsWith("C") })
```

이 함수들을 사용하면 단순 반복문과 조건문을 직접 작성하지 않아도 된다.

---

## 11. groupBy

`groupBy`는 특정 값을 기준으로 원소를 묶어 Map을 만든다.

```kotlin
val persons = listOf(
    Person("Captain", 44),
    Person("Cyclops", 35),
    Person("Deadpool", 31),
    Person("Iron Man", 31),
    Person("Iceman", 54),
    Person("Loki", 54),
    Person("Hulk", 54),
)
```

나이를 기준으로 그룹화할 수 있다.

```kotlin
fun main() {
    println(persons.groupBy { it.age })
}
```

실행 결과는 다음과 같은 형태다.

```text
{
    44=[Person(name=Captain, age=44)],
    35=[Person(name=Cyclops, age=35)],
    31=[Person(name=Deadpool, age=31), Person(name=Iron Man, age=31)],
    54=[Person(name=Iceman, age=54), Person(name=Loki, age=54), Person(name=Hulk, age=54)]
}
```

결과 타입은 나이를 key로 가지고, 같은 나이를 가진 사람 목록을 value로 가지는 Map이다.

```kotlin
Map<Int, List<Person>>
```

즉, `groupBy { it.age }`는 "나이별 사람 목록"을 만들어준다.

---

## 12. flatten

`flatten`은 중첩 컬렉션을 하나의 컬렉션으로 펼친다.

```kotlin
val nested = listOf(
    listOf(1, 2),
    listOf(3, 4),
    listOf(5)
)

println(nested.flatten())
```

실행 결과는 다음과 같다.

```text
[1, 2, 3, 4, 5]
```

즉, `List<List<T>>` 같은 구조를 `List<T>`로 만들어준다.

---

## 13. flatMap

`flatMap`은 `map`과 `flatten`을 합친 함수다.

```kotlin
fun main() {
    val strings = listOf("abc", "def")

    println(strings.map { it.toList() })
    println(strings.map { it.toList() }.flatten())
    println(strings.flatMap { it.toList() })
}
```

실행 결과는 다음과 같다.

```text
[[a, b, c], [d, e, f]]
[a, b, c, d, e, f]
[a, b, c, d, e, f]
```

`map { it.toList() }`는 문자열 하나를 문자 리스트로 바꾼다. 그래서 결과가 중첩 리스트가 된다.

```kotlin
List<List<Char>>
```

그 뒤 `flatten()`을 호출하면 하나의 리스트로 펼쳐진다.

```kotlin
List<Char>
```

`flatMap`은 이 두 단계를 한 번에 처리한다.

```kotlin
strings.flatMap { it.toList() }
```

즉, 다음 두 코드는 같은 결과를 만든다.

```kotlin
strings.map { it.toList() }.flatten()
strings.flatMap { it.toList() }
```

실무에서 항상 자주 쓰는 함수는 아니지만, 어떤 값을 여러 값으로 변환한 뒤 하나의 컬렉션으로 합쳐야 할 때 유용하다.

---

## 14. sequence

`sequence`는 자바의 Stream과 비슷한 역할을 한다.

일반 컬렉션 연산은 각 단계의 결과를 즉시 계산한다. 반면 sequence는 필요한 순간까지 계산을 미루고, 결과가 필요한 만큼만 계산한다.

```kotlin
val persons = listOf(
    Person("Captain", 44),
    Person("Cyclops", 35),
    Person("Deadpool", 31),
    Person("Iron Man", 31),
    Person("Iceman", 54),
    Person("Loki", 54),
    Person("Hulk", 54),
)
```

먼저 일반 컬렉션으로 실행해보자.

```kotlin
fun main() {
    println(
        persons.map {
            println(it)
            it.name
        }.find { it.startsWith("I") }
    )
}
```

실행 결과는 다음과 같다.

```text
Person(name=Captain, age=44)
Person(name=Cyclops, age=35)
Person(name=Deadpool, age=31)
Person(name=Iron Man, age=31)
Person(name=Iceman, age=54)
Person(name=Loki, age=54)
Person(name=Hulk, age=54)
Iron Man
```

일반 컬렉션은 먼저 모든 원소에 대해 `map`을 수행한다. 그 다음 `find`를 수행한다. 그래서 `Iron Man`을 찾을 수 있어도 이미 모든 사람의 이름을 만들어둔 상태다.

이번에는 sequence를 사용해보자.

```kotlin
fun main() {
    println(
        persons.asSequence()
            .map {
                println(it)
                it.name
            }
            .find { it.startsWith("I") }
    )
}
```

실행 결과는 다음과 같다.

```text
Person(name=Captain, age=44)
Person(name=Cyclops, age=35)
Person(name=Deadpool, age=31)
Person(name=Iron Man, age=31)
Iron Man
```

sequence는 원소 하나마다 `map`과 `find`를 이어서 처리한다. `Iron Man`에서 조건을 만족하는 값을 찾으면 더 이상 뒤의 원소를 계산하지 않는다.

---

## 15. sequence를 언제 사용할까

`sequence`는 대량 컬렉션에서 여러 단계의 연산을 연결하거나, 중간에 찾기 연산으로 멈출 수 있는 경우에 유용하다.

예를 들어 다음과 같은 상황에서 고려할 수 있다.

- 원소 개수가 많다.
- `map`, `filter` 같은 중간 연산이 여러 번 이어진다.
- `find`, `firstOrNull`, `take`처럼 중간에 멈출 수 있는 최종 연산을 사용한다.

하지만 항상 sequence가 더 좋은 것은 아니다. 작은 컬렉션에서는 일반 컬렉션 연산이 더 단순하고 충분히 빠른 경우가 많다. 성능 최적화가 필요하거나 데이터가 많을 때 sequence를 고려하면 된다.

---

## 16. 정리

코틀린 람다식에서 이번 강의의 핵심은 다음과 같다.

- 람다는 함수에 넘길 수 있는 작은 코드 조각이다.
- 코틀린 람다는 중괄호(`{}`)로 작성하고, `파라미터 -> 본문` 형태를 사용한다.
- 람다는 함수의 인자로 넘기거나 변수에 저장한 뒤 나중에 실행할 수 있다.
- 함수의 마지막 인자가 람다이면 괄호 밖으로 뺄 수 있다.
- 람다가 유일한 인자라면 빈 괄호도 생략할 수 있다.
- 단일 파라미터 람다에서는 기본 이름 `it`을 사용할 수 있다.
- 이미 존재하는 프로퍼티나 함수를 넘길 때는 `::`를 사용한 멤버 참조를 활용할 수 있다.
- `filter`는 조건에 맞는 원소만 남기고, `map`은 원소를 다른 값으로 변환한다.
- `all`, `any`, `count`, `find`, `groupBy` 같은 컬렉션 API는 반복문 없이 데이터를 다루게 해준다.
- `flatMap`은 `map` 이후 생긴 중첩 컬렉션을 하나로 펼쳐준다.
- `sequence`는 필요한 만큼만 계산하기 때문에 대량 컬렉션이나 중간에 멈출 수 있는 연산에서 유용할 수 있다.
