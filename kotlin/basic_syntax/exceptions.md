# 코틀린의 예외 처리

## 날짜
- 2026-06-26

## 🎯 오늘의 주제
- 코틀린에서 예외를 던지는 기본 방식 이해
- `try`, `catch`, `finally` 문법 정리
- `try-catch-finally`가 식(expression)으로 사용될 수 있다는 점 이해
- 자바의 checked exception과 코틀린 예외 처리 방식의 차이 정리
- Kotlin + Spring에서 `@Transactional` 사용 시 주의할 점 이해

---

## 1. 코틀린의 예외 처리

코틀린의 예외 처리 문법은 자바와 거의 비슷하다.

예외를 던질 때는 `throw`를 사용하고, 예외를 잡을 때는 `try`, `catch`, `finally`를 사용한다.

다만 자바와 비교했을 때 중요한 차이가 있다.

- 예외 객체를 만들 때 `new`를 사용하지 않는다.
- `try-catch-finally`를 값처럼 사용할 수 있다.
- checked exception을 함수 시그니처에 강제하지 않는다.

특히 checked exception을 강제하지 않는다는 점은 코틀린을 자바나 Spring과 함께 사용할 때 반드시 이해해야 한다.

---

## 2. 기본적인 예외 던지기

코틀린에서 예외를 던지는 방식은 자바와 거의 같다.

```kotlin
if (str.length > 10) {
    throw IllegalStateException("str($str) is too long")
}
```

자바에서는 예외 객체를 생성할 때 `new`를 사용한다.

```java
throw new IllegalStateException("str(" + str + ") is too long");
```

코틀린에서는 `new` 키워드가 없다.

```kotlin
throw IllegalStateException("str($str) is too long")
```

클래스의 인스턴스를 만들 때와 마찬가지로 예외 객체도 클래스 이름을 함수처럼 호출해서 생성한다.

---

## 3. try, catch, finally

기본적인 `try`, `catch`, `finally` 구조는 자바와 거의 동일하다.

```kotlin
fun parse(numberStr: String): Int =
    try {
        Integer.parseInt(numberStr)
    } catch (e: Exception) {
        throw IOException("일부러 발생시키는 checked exception")
    } finally {
        println("무조건 실행되는 코드블록")
    }
```

각 블록의 역할은 다음과 같다.

- `try`: 예외가 발생할 수 있는 코드를 실행한다.
- `catch`: 예외가 발생했을 때 처리한다.
- `finally`: 예외 발생 여부와 관계없이 마지막에 실행된다.

`finally`는 주로 자원 정리나 로그 출력처럼 반드시 실행해야 하는 코드에 사용한다.

---

## 4. try-catch-finally는 식이다

코틀린에서 `try-catch-finally`는 `if`, `when`처럼 식(expression)으로 사용할 수 있다.

즉, `try`의 결과나 `catch`의 결과를 변수에 담거나 함수의 반환값으로 사용할 수 있다.

```kotlin
fun parse(numberStr: String): Int =
    try {
        Integer.parseInt(numberStr)
    } catch (e: NumberFormatException) {
        0
    }
```

위 함수는 숫자 문자열을 `Int`로 변환한다. 변환에 성공하면 `Integer.parseInt(numberStr)`의 결과가 반환되고, 실패하면 `catch` 블록의 마지막 값인 `0`이 반환된다.

변수에 담을 수도 있다.

```kotlin
val number = try {
    Integer.parseInt(numberStr)
} catch (e: NumberFormatException) {
    0
}
```

이처럼 `try-catch`가 하나의 값을 만들어내기 때문에 코드를 더 간결하게 작성할 수 있다.

다만 `finally` 블록의 값은 전체 `try` 식의 결과가 되지 않는다. `finally`는 결과를 반환하기 위한 블록이라기보다, 마지막에 반드시 실행되는 정리 블록으로 이해하는 것이 좋다.

---

## 5. Checked Exception 차이

자바에는 checked exception과 unchecked exception의 구분이 있다.

자바에서 checked exception을 던지는 코드는 다음 둘 중 하나를 반드시 해야 한다.

- `try-catch`로 직접 처리한다.
- 메서드 시그니처에 `throws`를 선언한다.

예를 들어 자바에서 `IOException`을 던지려면 다음처럼 작성해야 한다.

```java
public int parse(String numberStr) throws IOException {
    try {
        return Integer.parseInt(numberStr);
    } catch (Exception e) {
        throw new IOException("일부러 발생시키는 checked exception");
    }
}
```

`throws IOException`을 선언하지 않으면 컴파일 오류가 발생한다.

반면 코틀린에서는 checked exception이라도 함수 시그니처에 `throws IOException`을 붙이지 않아도 된다.

```kotlin
fun parse(numberStr: String): Int =
    try {
        Integer.parseInt(numberStr)
    } catch (e: Exception) {
        throw IOException("일부러 발생시키는 checked exception")
    }
```

코틀린은 checked exception 처리를 컴파일러가 강제하지 않는다. 그래서 함수 선언만 보고 이 함수가 `IOException`을 던질 수 있는지 바로 알기 어렵다.

---

## 6. 장점과 주의점

코틀린이 checked exception을 강제하지 않는 것은 장점도 있다.

- 불필요한 `throws` 선언이 줄어든다.
- 예외를 억지로 감싸거나 다시 던지는 코드가 줄어든다.
- 자바보다 함수 시그니처가 간결해진다.

하지만 단점도 있다.

- 어떤 함수가 checked exception을 던질 수 있는지 놓치기 쉽다.
- 호출하는 쪽에서 예외 처리를 빼먹어도 컴파일러가 알려주지 않는다.
- 자바 라이브러리나 Spring과 함께 사용할 때 예외 정책을 더 신경 써야 한다.

즉, 코틀린에서는 예외 처리가 덜 번거로운 대신, 개발자가 예외 흐름을 더 의식적으로 관리해야 한다.

---

## 7. Kotlin + Spring에서의 주의사항

Spring에서 `@Transactional`을 사용할 때는 예외 종류에 따라 롤백 여부가 달라진다.

기본적으로 Spring 트랜잭션은 unchecked exception인 `RuntimeException`이나 `Error`가 발생하면 롤백한다. 반면 checked exception은 기본 설정에서는 롤백 대상이 아니다.

```kotlin
@Transactional
fun saveSomething() {
    throw IOException("checked exception")
}
```

위 코드에서 `IOException`은 checked exception이다. Spring의 기본 트랜잭션 정책에서는 이 예외가 발생해도 롤백되지 않을 수 있다.

자바에서는 checked exception을 던지면 컴파일러가 `throws` 선언이나 `try-catch` 처리를 요구한다. 그래서 개발자가 해당 예외를 인지할 가능성이 높다.

하지만 코틀린에서는 checked exception을 함수 시그니처에 강제하지 않는다. 그래서 `IOException` 같은 checked exception이 발생할 수 있는데도 코드상에서 쉽게 드러나지 않을 수 있다.

그 결과 예외는 발생했지만 트랜잭션은 롤백되지 않는 상황이 생길 수 있다.

---

## 8. 어떻게 대응할 수 있을까

Kotlin + Spring에서 checked exception이 발생할 수 있는 코드를 작성할 때는 다음을 고려하는 것이 좋다.

- checked exception이 발생할 수 있는 지점을 명확히 파악한다.
- 필요한 경우 `try-catch`로 예외를 잡아 도메인에 맞는 unchecked exception으로 변환한다.
- checked exception에서도 롤백이 필요하다면 `rollbackFor`를 명시한다.

예를 들어 checked exception에서도 롤백이 필요하다면 다음처럼 작성할 수 있다.

```kotlin
@Transactional(rollbackFor = [IOException::class])
fun saveSomething() {
    throw IOException("checked exception")
}
```

또는 checked exception을 잡아서 런타임 예외로 바꿔 던질 수도 있다.

```kotlin
@Transactional
fun saveSomething() {
    try {
        riskyCall()
    } catch (e: IOException) {
        throw IllegalStateException("저장 중 외부 호출 실패", e)
    }
}
```

이 방식은 예외를 숨기는 것이 아니라, 현재 계층에서 어떤 의미의 실패인지 다시 표현하는 방식에 가깝다.

---

## 9. 정리

코틀린 예외 처리에서 이번 강의의 핵심은 다음과 같다.

- 코틀린도 자바처럼 `throw`, `try`, `catch`, `finally`로 예외를 처리한다.
- 예외 객체를 만들 때 `new` 키워드를 사용하지 않는다.
- `try-catch-finally`는 식으로 사용할 수 있으며, 결과값을 반환할 수 있다.
- `finally`는 결과값을 만들기보다 예외 여부와 관계없이 실행되는 정리 블록이다.
- 코틀린은 checked exception 처리를 컴파일러가 강제하지 않는다.
- checked exception이라도 함수 시그니처에 `throws`를 붙일 필요가 없다.
- Kotlin + Spring에서 `@Transactional`을 사용할 때 checked exception은 기본적으로 롤백되지 않을 수 있다.
- 롤백이 필요하다면 `rollbackFor`를 명시하거나 unchecked exception으로 변환하는 방식을 고려할 수 있다.
