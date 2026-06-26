# 코틀린 반복문

## 날짜
- 2026-06-26

## 🎯 오늘의 주제
- 코틀린의 `while`, `do-while` 반복문 문법 이해
- 범위(`range`)를 사용한 `for` 반복문 정리
- `downTo`, `step`, `until`을 사용한 반복 범위 제어 이해
- Map을 순회하면서 key와 value를 다루는 방법 정리
- `in`, `!in`을 사용한 범위 검사와 `when` 활용 이해

---

## 1. 코틀린 반복문의 특징

코틀린의 반복문은 자바와 매우 유사하다. 특히 `while`과 `do-while`은 자바에서 사용하던 방식과 거의 같다.

다만 `for` 반복문은 자바의 전통적인 인덱스 기반 반복문과 형태가 다르다. 코틀린에서는 범위나 컬렉션을 `in`으로 순회하는 방식이 기본이다.

```kotlin
for (i in 1..100) {
    println(i)
}
```

이 코드는 `1`부터 `100`까지 순서대로 반복한다. 자바의 `for (int i = 1; i <= 100; i++)`와 비슷한 의미다.

---

## 2. while 반복문

`while` 반복문은 조건이 참인 동안 블록을 반복 실행한다.

```kotlin
while (조건) {
    // ...
}
```

예를 들어 `i`가 5보다 작을 동안 값을 출력할 수 있다.

```kotlin
var i = 0

while (i < 5) {
    println(i)
    i++
}
```

`while`은 먼저 조건을 검사한 뒤 본문을 실행한다. 따라서 처음부터 조건이 거짓이면 본문은 한 번도 실행되지 않는다.

---

## 3. do-while 반복문

`do-while` 반복문은 본문을 먼저 한 번 실행한 뒤 조건을 검사한다.

```kotlin
do {
    // ...
} while (조건)
```

예를 들어 다음 코드는 `i < 5` 조건을 나중에 검사한다.

```kotlin
var i = 0

do {
    println(i)
    i++
} while (i < 5)
```

`do-while`은 조건이 처음부터 거짓이어도 본문이 최소 한 번은 실행된다. 사용자 입력을 한 번 받고 나서 반복 여부를 판단하는 코드에서 사용할 수 있다.

---

## 4. for 반복문과 범위

코틀린의 `for` 반복문은 범위나 컬렉션을 순회할 때 사용한다.

먼저 숫자가 짝수인지 홀수인지 판단하는 함수를 만들 수 있다.

```kotlin
fun evenOrOdd(n: Int) =
    when {
        n % 2 == 0 -> "even"
        else -> "odd"
    }
```

이제 `1`부터 `100`까지 반복하면서 각 숫자가 짝수인지 홀수인지 출력해보자.

```kotlin
fun main() {
    for (i in 1..100) {
        println(evenOrOdd(i))
    }
}
```

`1..100`은 `1`부터 `100`까지의 범위를 의미한다. 양쪽 끝 값을 모두 포함하므로 `100`도 반복 대상에 포함된다.

```kotlin
for (i in 1..100)
    println(evenOrOdd(i))
```

반복문 본문이 한 줄이라면 중괄호를 생략할 수 있다. 다만 처음에는 중괄호를 붙여두는 편이 실수를 줄이기 좋다.

---

## 5. downTo와 step

감소하는 방향으로 반복하려면 `downTo`를 사용한다.

```kotlin
for (i in 100 downTo 1) {
    println(i)
}
```

이 코드는 `100`부터 `1`까지 거꾸로 반복한다.

반복 간격을 조절하려면 `step`을 사용한다.

```kotlin
for (i in 100 downTo 1 step 3) {
    println(evenOrOdd(i))
}
```

이 코드는 `100, 97, 94, ...`처럼 3씩 감소하면서 반복한다. `downTo`도 양쪽 끝을 포함하므로 조건에 맞으면 마지막 값도 포함된다.

---

## 6. until

`..` 범위는 마지막 값을 포함한다.

```kotlin
for (i in 1..100) {
    println(i)
}
```

이 코드는 `100`까지 출력한다.

마지막 값을 포함하지 않으려면 `until`을 사용한다.

```kotlin
for (i in 1 until 100) {
    println(i)
}
```

이 코드는 `1`부터 `99`까지 출력한다. 즉, `100`은 포함되지 않는다.

자바의 `i < 100` 조건과 비슷한 의미로 이해할 수 있다.

```java
for (int i = 1; i < 100; i++) {
    System.out.println(i);
}
```

코틀린에서는 다음처럼 표현한다.

```kotlin
for (i in 1 until 100) {
    println(i)
}
```

---

## 7. Map과 이터레이션

코틀린에서는 `Map`도 `for` 반복문으로 순회할 수 있다.

```kotlin
val students = mutableMapOf<Int, String>()

students[1] = "Jack"
students[2] = "Diana"
students[3] = "Frost"
```

`mutableMapOf`는 변경 가능한 Map을 간단히 생성해주는 컬렉션 함수다. 위 예제에서 key는 `Int`, value는 `String`이다.

Map에 값을 추가할 때는 다음처럼 대괄호 문법을 사용할 수 있다.

```kotlin
students[1] = "Jack"
```

이는 `1`번 key에 `"Jack"`이라는 value를 저장한다는 의미다.

---

## 8. Map 순회

Map을 순회할 때는 각 항목을 하나씩 꺼낼 수 있다.

```kotlin
for (c in students) {
    println("번호:${c.key}, 이름:${c.value}")
}
```

여기서 `c`는 Map에 들어 있는 하나의 항목이다. 각 항목은 key와 value를 가지고 있기 때문에 `c.key`, `c.value`로 접근할 수 있다.

코틀린에서는 구조 분해 선언(destructuring declaration)을 사용해 key와 value를 바로 변수로 받을 수도 있다.

```kotlin
for ((num, name) in students) {
    println("번호:$num, 이름:$name")
}
```

`(num, name)`은 Map의 각 항목을 key와 value로 나누어 받는 문법이다.

- `num`에는 key가 들어간다.
- `name`에는 value가 들어간다.

따라서 `c.key`, `c.value`를 직접 쓰는 것보다 더 간결하게 읽을 수 있다.

> **💡 구조 분해 선언이란**
>
> 구조 분해 선언(Destructuring Declaration)은 하나의 객체가 가진 여러 값을 여러 변수로 한 번에 나누어 담는 문법이다. Map을 순회할 때 `(num, name)`처럼 작성하면 key와 value를 각각의 변수로 바로 받을 수 있다.

---

## 9. 컬렉션 생성 함수

코틀린에는 컬렉션을 쉽게 만들 수 있는 함수들이 있다.

```kotlin
val numbers = listOf(1, 2, 3)
val names = setOf("Jack", "Diana", "Frost")
val students = mapOf(1 to "Jack", 2 to "Diana", 3 to "Frost")
```

대표적으로 다음 함수들이 자주 사용된다.

- `listOf()`: 읽기 전용 List 생성
- `mutableListOf()`: 변경 가능한 List 생성
- `setOf()`: 읽기 전용 Set 생성
- `mutableSetOf()`: 변경 가능한 Set 생성
- `mapOf()`: 읽기 전용 Map 생성
- `mutableMapOf()`: 변경 가능한 Map 생성

첫 번째 강의에서 `val`과 `var`를 구분한 것처럼, 컬렉션도 읽기 전용과 변경 가능 컬렉션을 구분해서 사용할 수 있다.

---

## 10. in으로 범위 검사

`in`은 반복문에서만 사용하는 키워드가 아니다. 어떤 값이 특정 범위 안에 있는지 검사할 때도 사용할 수 있다.

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'

fun isDigit(c: Char) = c in '0'..'9'

fun isNotDigit(c: Char) = c !in '0'..'9'
```

`c in '0'..'9'`는 `c`가 문자 `'0'`부터 `'9'` 사이에 있는지 검사한다.

반대로 `!in`은 범위 안에 없다는 의미다.

```kotlin
fun isNotDigit(c: Char) = c !in '0'..'9'
```

`a in s..t`는 내부적으로 `s <= a && a <= t`와 같은 의미로 볼 수 있다. 즉, 양쪽 끝 값을 모두 포함한다.

---

## 11. when에서 in 사용하기

`when`에서도 `in`을 사용해 범위 조건을 표현할 수 있다.

```kotlin
fun recognize(c: Char) =
    when (c) {
        in '0'..'9' -> "숫자"
        in 'a'..'z', in 'A'..'Z' -> "알파벳"
        else -> "숫자도 알파벳도 아님"
    }
```

이 코드는 문자가 숫자인지, 알파벳인지, 둘 다 아닌지 판단한다.

- `in '0'..'9'`: 숫자 문자
- `in 'a'..'z', in 'A'..'Z'`: 소문자 또는 대문자 알파벳
- `else`: 숫자도 알파벳도 아닌 문자

`when`과 `in`을 함께 사용하면 여러 조건식을 `if`, `else if`로 길게 나열하지 않아도 된다. 범위 기반 조건이 많을 때 특히 읽기 좋다.

---

## 12. 정리

코틀린 반복문에서 이번 강의의 핵심은 다음과 같다.

- `while`과 `do-while`은 자바와 거의 같은 방식으로 동작한다.
- `for`는 `for (i in 범위)` 형태로 범위나 컬렉션을 순회한다.
- `1..100`은 `1`부터 `100`까지 양쪽 끝을 포함한다.
- 마지막 값을 제외하고 싶을 때는 `until`을 사용한다.
- 감소 방향 반복은 `downTo`, 반복 간격은 `step`으로 표현한다.
- Map을 순회할 때 `c.key`, `c.value`를 사용할 수 있다.
- 구조 분해 선언을 사용하면 `for ((key, value) in map)` 형태로 더 간결하게 순회할 수 있다.
- `mutableMapOf`, `listOf`, `mapOf`, `setOf` 같은 컬렉션 생성 함수를 사용할 수 있다.
- `in`과 `!in`은 값이 범위 안에 있는지 검사할 때도 사용된다.
- `when`에서 `in`을 사용하면 범위 기반 조건을 깔끔하게 표현할 수 있다.
