# 코틀린의 함수와 변수

## 날짜
- 2026-06-25

## 🎯 오늘의 주제
- 코틀린 프로그램의 가장 기본적인 시작점인 `main` 함수 이해
- 코틀린 함수 선언 문법과 자바 문법의 차이 정리
- `val`, `var`를 기준으로 코틀린의 변수 선언 방식 이해
- 문자열 템플릿을 사용해 값을 문자열 안에 자연스럽게 삽입하는 방법 정리

---

## 1. 안녕, 코틀린?

가장 먼저 코틀린으로 `Hello, Kotlin!`을 출력하는 예제를 살펴보자.

```kotlin
fun main(args: Array<String>) {
    println("Hello, Kotlin!")
}
```

같은 내용을 자바로 작성하면 다음과 같다.

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

두 코드를 비교하면 코틀린의 기본적인 특징을 바로 확인할 수 있다.

- 함수는 `fun` 키워드로 시작한다.
- 함수를 반드시 클래스 안에 정의할 필요가 없다.
- `System.out.println` 대신 `println`을 사용할 수 있다.
- 문장 끝에 세미콜론(`;`)을 붙이지 않아도 된다.
- 배열을 다루기 위한 별도의 특수 문법이 없다.
- 타입을 변수명이나 파라미터명 뒤에 작성한다.

자바에서는 타입이 먼저 오고 이름이 뒤에 온다.

```java
String name;
```

코틀린에서는 이름이 먼저 오고 타입이 뒤에 온다.

```kotlin
val name: String = "Kotlin"
```

이 방식은 최근의 여러 모던 언어에서도 자주 볼 수 있는 형태다. 코드를 읽을 때는 `name`이라는 값이 있고, 그 값의 타입은 `String`이라고 해석하면 된다.

---

## 2. 함수의 기본 형태

코틀린에서 함수는 `fun` 키워드로 선언한다. 가장 일반적인 형태는 다음과 같다.

```kotlin
fun 함수명(파라미터명: 타입): 반환타입 {
    return 반환값
}
```

예를 들어 두 정수를 더하는 함수는 다음처럼 작성할 수 있다.

```kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

여기서 `a: Int`, `b: Int`는 각각 `a`와 `b`라는 파라미터가 `Int` 타입이라는 뜻이다. 함수 뒤의 `: Int`는 이 함수가 `Int` 값을 반환한다는 의미다.

---

## 3. Expression Body 함수

코틀린에서는 함수 본문이 하나의 식으로 끝나는 경우 더 짧게 작성할 수 있다.

```kotlin
fun sum(a: Int, b: Int): Int = a + b
```

이처럼 중괄호(`{}`) 없이 `=` 뒤에 하나의 표현식(expression)을 작성하는 함수를 **Expression Body Function**이라고 한다. `a + b`의 결과를 그대로 반환하며, 중괄호와 `return`을 생략할 수 있기 때문에 간단한 계산 함수에서 자주 사용된다.

반대로 중괄호와 `return`을 사용하는 일반적인 함수 형태는 **Block Body Function**이라고 볼 수 있다.

```kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

반환 타입도 생략할 수 있다.

```kotlin
fun sum(a: Int, b: Int) = a + b
```

코틀린 컴파일러가 `a + b`의 결과 타입을 보고 반환 타입이 `Int`라고 추론하기 때문이다. 이를 **타입 추론**이라고 한다.

다만 초반에는 반환 타입을 명시하는 편이 함수를 읽기 쉽다. 특히 함수가 길어지거나 공개 API처럼 다른 코드에서 많이 사용할 함수라면 반환 타입을 적어두는 것이 좋다.

---

## 4. 반환값이 없는 함수

자바에서 반환값이 없는 함수는 `void`를 사용한다.

```java
public void printSomething(String something) {
    System.out.println("Something : " + something);
}
```

코틀린에서는 반환값이 없을 때 `Unit`을 사용한다.

```kotlin
fun printSomething(something: String): Unit {
    println("Something : $something")
}
```

`Unit`은 자바의 `void`와 비슷하게 생각할 수 있다. 다만 코틀린에서는 `Unit`도 하나의 타입이다.

그리고 반환 타입이 `Unit`인 경우에는 보통 생략한다.

```kotlin
fun printSomething(something: String) {
    println("Something : $something")
}
```

실무 코드에서는 위처럼 `Unit`을 생략한 형태를 더 자주 볼 수 있다.

---

## 5. 변수 선언

코틀린에서 변수는 `val` 또는 `var`로 선언한다.

- `val`: 한 번 값을 할당하면 변경할 수 없는 변수
- `var`: 값을 변경할 수 있는 변수

`val`은 value의 약어로 볼 수 있고, `var`는 variable의 약어로 볼 수 있다.

자바에서는 일반 변수 선언이 기본적으로 변경 가능하다.

```java
String name = "Kotlin";
name = "Java";
```

값을 변경하지 못하게 만들고 싶다면 `final`을 명시해야 한다.

```java
final String name = "Kotlin";
// name = "Java"; // 컴파일 오류
```

반면 코틀린에서는 변경 불가능한 값을 나타내는 `val`을 일상적으로 먼저 사용한다. 즉, 자바에서는 변경 가능성이 기본이고 `final`로 불변을 표시하는 느낌이라면, 코틀린에서는 `val`을 기본 선택지로 두고 꼭 필요한 경우에만 `var`를 사용하는 흐름에 가깝다.

```kotlin
val i: Int = 123
val j = 321

var age: Int = 25
age = 26
```

`i`는 타입을 직접 작성한 예시이고, `j`는 타입을 생략한 예시다. `j`에 `321`을 할당했기 때문에 컴파일러는 `j`의 타입을 `Int`로 추론한다.

`age`는 `var`로 선언했기 때문에 이후에 값을 바꿀 수 있다.

```kotlin
var age: Int = 25
age = 26
```

반대로 `val`로 선언한 값은 다시 할당할 수 없다.

```kotlin
val name = "Kotlin"
// name = "Java" // 컴파일 오류
```

코틀린에서는 가능하면 `val`을 먼저 사용하고, 정말 값이 바뀌어야 할 때만 `var`를 사용하는 습관이 좋다. 이유는 단순히 문법 취향의 문제가 아니다.

- 값이 중간에 바뀌지 않으면 코드의 흐름을 추적하기 쉽다.
- 같은 이름의 값이 어느 시점에 어떤 값을 가지고 있는지 의심할 필요가 줄어든다.
- 상태 변경이 줄어들면 버그가 발생할 가능성도 줄어든다.
- 멀티스레드 환경에서도 변경 가능한 공유 상태가 적을수록 안전한 코드를 작성하기 쉽다.

따라서 코틀린에서는 먼저 `val`로 선언해보고, 이후 실제로 재할당이 필요한 상황에서만 `var`로 바꾸는 방식이 권장된다.

---

## 6. Top-Level 변수

코틀린은 함수와 마찬가지로 변수를 클래스 외부에 선언할 수 있다.

```kotlin
val language = "Kotlin"

fun main() {
    println(language)
}
```

이처럼 파일의 최상위에 선언된 값을 **Top-Level Variable** 또는 더 정확히는 **Top-Level Property**라고 부른다. 코틀린에서는 클래스 밖에 선언한 함수도 **Top-Level Function**이라고 한다.

```kotlin
val language = "Kotlin" // top-level property

fun printLanguage() { // top-level function
    println(language)
}
```

일부 문맥에서는 패키지에 속한 선언이라는 의미로 package-level이라는 표현을 쓰기도 하지만, 코틀린 공식 문맥에서는 보통 **top-level function**, **top-level property**라는 표현을 더 많이 사용한다. 간단한 예제나 상수 선언에서 자주 사용된다.

다만 아무 값이나 전역처럼 많이 두면 코드가 복잡해질 수 있다. 여러 곳에서 공유해야 하는 값인지, 특정 함수나 클래스 안에만 있어도 되는 값인지 구분해서 사용하는 것이 좋다.

---

## 7. 문자열 템플릿

코틀린에서는 문자열 안에 변수를 바로 삽입할 수 있다. 이를 **문자열 템플릿**이라고 한다.

```kotlin
val number = 20

println("Please give me $number dollars")
```

`$변수명` 형태를 사용하면 해당 변수의 값이 문자열 안에 들어간다.

조금 더 복잡한 식을 넣고 싶다면 `${}`를 사용한다.

```kotlin
val number = 20

println("I only have ${if (number < 10) number else 10} dollars")
```

`${}` 안에는 단순한 변수뿐 아니라 조건식 같은 표현식도 넣을 수 있다. 위 예제에서는 `number`가 10보다 작으면 `number`를 출력하고, 그렇지 않으면 `10`을 출력한다.

자바에서는 문자열과 값을 `+`로 이어 붙이는 경우가 많다.

```java
System.out.println("Please give me " + number + " dollars");
```

코틀린에서는 문자열 템플릿을 사용하면 더 읽기 쉬운 형태로 작성할 수 있다.

```kotlin
println("Please give me $number dollars")
```

---

## 8. 정리

코틀린의 함수와 변수 문법에서 이번 강의의 핵심은 다음과 같다.

- 함수는 `fun` 키워드로 선언한다.
- 파라미터와 변수의 타입은 이름 뒤에 작성한다.
- 함수 본문이 하나의 표현식이면 **Expression Body Function**으로 간결하게 작성할 수 있다.
- 반환값이 없는 함수는 `Unit`을 반환하며, 보통 `Unit`은 생략한다.
- 클래스 밖에 선언한 함수와 프로퍼티는 각각 **Top-Level Function**, **Top-Level Property**라고 부른다.
- 변수는 기본적으로 `val`을 우선 사용하고, 값이 변경되어야 할 때만 `var`를 사용한다.
- 문자열 안에 값을 넣을 때는 `$변수명` 또는 `${수식}` 형태의 문자열 템플릿을 사용한다.
