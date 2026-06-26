# 코틀린의 선택과 표현 처리

## 날짜
- 2026-06-26

## 🎯 오늘의 주제
- 코틀린의 `when`이 자바의 `switch`보다 유연한 이유 이해
- `enum class` 선언 방식과 프로퍼티, 메서드를 가진 enum 정리
- `when`으로 enum 값을 다룰 때 exhaustive 조건의 장점 이해
- 인자가 없는 `when` 사용 방식 정리
- 타입 검사와 형변환을 함께 처리하는 스마트 캐스트 이해

---

## 1. 코틀린의 when

코틀린의 `when`은 자바의 `switch`와 비슷한 역할을 하지만, 훨씬 더 강력하고 유연하다.

자바의 `switch`는 주로 특정 값이 어떤 값과 일치하는지 비교할 때 사용한다. 반면 코틀린의 `when`은 다음과 같은 방식으로 사용할 수 있다.

- 하나의 값에 대한 분기 처리
- 여러 값을 하나의 조건으로 묶는 처리
- 조건식 자체를 나열하는 처리
- 타입 검사와 스마트 캐스트 처리
- 값을 반환하는 표현식(expression)으로 사용

즉, 코틀린의 `when`은 단순한 분기문이라기보다 여러 조건을 표현하고 결과값을 만들어내는 선택 표현식에 가깝다.

---

## 2. enum class

`when`을 enum과 함께 사용하는 경우가 많기 때문에 먼저 enum을 살펴보자.

자바의 enum은 다음처럼 선언한다.

```java
public enum Color {
    RED, ORANGE, YELLOW, GREEN;
}
```

코틀린에서는 `enum class` 키워드를 사용한다.

```kotlin
enum class Color {
    RED, ORANGE, YELLOW, GREEN
}
```

자바와 개념은 거의 비슷하다. 정해진 값의 집합을 타입으로 표현하고 싶을 때 사용한다. 예를 들어 색상, 주문 상태, 회원 등급처럼 가능한 값이 제한되어 있는 경우 enum으로 표현하기 좋다.

코틀린에서는 enum도 클래스이기 때문에 `enum class`라고 쓴다.

---

## 3. 프로퍼티와 메서드를 갖는 enum class

코틀린의 enum class는 프로퍼티와 메서드를 가질 수 있다.

```kotlin
enum class Color(val r: Int, val g: Int, val b: Int) {
    RED(255, 0, 0),
    ORANGE(255, 166, 0),
    YELLOW(255, 255, 0),
    GREEN(0, 255, 0);

    fun rgb() = (r * 256 + g) * 256 + b
}
```

`Color` enum의 각 값은 `r`, `g`, `b` 값을 가진다.

- `RED`는 `255, 0, 0`
- `ORANGE`는 `255, 166, 0`
- `YELLOW`는 `255, 255, 0`
- `GREEN`은 `0, 255, 0`

`rgb()` 함수는 각 색상의 RGB 값을 하나의 숫자로 계산한다.

```kotlin
val red = Color.RED

println(red.r)
println(red.rgb())
```

enum 상수 목록 뒤에 함수나 프로퍼티를 선언하려면 마지막 enum 상수 뒤에 세미콜론(`;`)을 붙인다. 코틀린에서는 보통 세미콜론을 생략하지만, enum 상수 목록과 본문을 구분해야 하는 이 경우에는 세미콜론이 필요하다.

---

## 4. when으로 enum 클래스 다루기

`when`은 enum과 함께 사용할 때 특히 읽기 좋다.

예를 들어 색상에 해당하는 한글 이름을 반환하는 함수를 만들 수 있다.

```kotlin
fun getKoreanColor(color: Color): String =
    when (color) {
        Color.RED -> "빨강"
        Color.ORANGE -> "오렌지"
        Color.YELLOW -> "노랑"
        Color.GREEN -> "녹색"
    }
```

이 함수는 `color` 값에 따라 다른 문자열을 반환한다. `when` 전체가 하나의 값을 만들어내기 때문에 `=` 뒤에 바로 사용할 수 있다. 이런 형태는 앞에서 배운 **Expression Body Function**과도 잘 어울린다.

같은 enum 값을 같은 결과로 묶고 싶다면 쉼표(`,`)로 여러 값을 나열할 수 있다.

```kotlin
fun getKoreanColor(color: Color): String =
    when (color) {
        Color.RED, Color.ORANGE -> "빨강 또는 오렌지"
        Color.YELLOW -> "노랑"
        Color.GREEN -> "녹색"
    }
```

여러 조건이 같은 결과를 반환할 때 `when`을 사용하면 중복을 줄이고 의도를 더 명확하게 드러낼 수 있다.

---

## 5. exhaustive 조건

`when`을 표현식으로 사용할 때는 모든 경우에 대해 결과값이 있어야 한다.

예를 들어 `Color` enum에 `RED`, `ORANGE`, `YELLOW`, `GREEN`이 있다면 이 모든 값을 처리해야 한다.

```kotlin
fun getKoreanColor(color: Color): String =
    when (color) {
        Color.RED -> "빨강"
        Color.ORANGE -> "오렌지"
        Color.YELLOW -> "노랑"
        Color.GREEN -> "녹색"
    }
```

이처럼 가능한 모든 경우를 빠짐없이 나열한 상태를 **exhaustive**하다고 표현한다.

만약 enum의 모든 구성요소를 나열하지 않고 `else`도 없다면 컴파일 오류가 발생한다.

```kotlin
fun getKoreanColor(color: Color): String =
    when (color) {
        Color.RED -> "빨강"
        Color.ORANGE -> "오렌지"
        // Color.YELLOW, Color.GREEN 처리 누락
    }
```

이 제약은 번거로워 보일 수 있지만 실제로는 큰 장점이 있다. 새로운 enum 값이 추가되었을 때, 해당 값을 처리하지 않은 `when` 표현식을 컴파일러가 알려주기 때문이다.

예를 들어 `Color`에 `BLUE`가 추가되면 기존 `when` 코드 중 `BLUE`를 처리하지 않은 부분에서 컴파일 오류가 발생한다. 덕분에 개발자는 수정해야 할 로직을 더 쉽게 찾을 수 있다.

자바에서 `if` 조건식이나 `switch` 문으로 처리했다면 새 값이 추가되었을 때 관련 조건을 직접 찾아다녀야 하는 경우가 많다. 코틀린에서는 exhaustive한 `when`을 사용하면 컴파일러의 도움을 받을 수 있다.

IntelliJ IDEA 같은 IDE에서는 enum의 일부 값이 누락된 `when`을 작성하면 `Add remaining branches` 같은 경고 또는 빠른 수정 메뉴가 뜨기도 한다. 이 기능을 사용하면 아직 작성하지 않은 enum 값을 IDE가 자동으로 분기 목록에 추가해준다.

```kotlin
fun getKoreanColor(color: Color): String =
    when (color) {
        Color.RED -> "빨강"
        // Add remaining branches 사용 시 나머지 enum 값 자동 추가
    }
```

enum 값이 많을수록 직접 타이핑하기보다 IDE의 자동완성이나 `Add remaining branches` 기능을 활용하는 편이 실수를 줄이는 데 도움이 된다.

---

## 6. else를 사용할지 모든 값을 나열할지

`when`에는 `else`를 사용할 수 있다.

```kotlin
fun getKoreanColor(color: Color): String =
    when (color) {
        Color.RED -> "빨강"
        Color.ORANGE -> "오렌지"
        else -> "기타 색상"
    }
```

하지만 enum을 다룰 때는 가능하면 `else` 없이 모든 값을 나열하는 편이 좋은 경우가 많다.

`else`를 사용하면 새 enum 값이 추가되어도 기존 코드가 계속 컴파일된다. 겉보기에는 편하지만, 새 값에 대한 처리를 빼먹어도 컴파일러가 알려주지 못한다.

반대로 모든 값을 직접 나열하면 새 enum 값이 추가되었을 때 누락된 분기를 컴파일 타임에 발견할 수 있다.

- `else` 사용: 기본 처리에는 편하지만 새 값 누락을 놓칠 수 있다.
- 모든 값 나열: 코드가 조금 길어질 수 있지만 변경에 더 안전하다.

따라서 enum의 각 값이 비즈니스적으로 의미가 있고 값마다 처리가 중요하다면 `else`보다 모든 값을 나열하는 방식을 우선 고려하는 것이 좋다.

---

## 7. 인자가 없는 when

`when`은 비교할 대상을 괄호 안에 넣지 않고 사용할 수도 있다.

```kotlin
fun mix(c1: Color, c2: Color): String =
    when {
        c1 == Color.RED && c2 == Color.YELLOW -> "오렌지"
        c1 == Color.YELLOW && c2 == Color.BLUE -> "녹색"
        else -> throw RuntimeException()
    }
```

인자가 없는 `when`은 `if`, `else if`, `else`를 나열한 구조와 비슷하다.

```kotlin
fun mix(c1: Color, c2: Color): String =
    if (c1 == Color.RED && c2 == Color.YELLOW) {
        "오렌지"
    } else if (c1 == Color.YELLOW && c2 == Color.BLUE) {
        "녹색"
    } else {
        throw RuntimeException()
    }
```

조건식이 길지 않고, 조건들의 형태가 서로 비슷할 때 인자가 없는 `when`을 사용하면 코드를 더 정돈된 형태로 표현할 수 있다.

다만 조건이 너무 복잡해지면 `when` 하나에 많은 로직이 몰릴 수 있다. 그럴 때는 조건을 별도의 함수로 분리하는 것이 더 읽기 좋다.

---

## 8. 스마트 캐스트

코틀린의 `when`은 타입 검사와 함께 사용할 때도 강력하다.

자바에서는 어떤 객체의 타입을 확인한 뒤 해당 타입의 메서드나 프로퍼티를 사용하려면 명시적 형변환이 필요하다.

```java
if (obj instanceof String) {
    String str = (String) obj;
    System.out.println(str.toLowerCase());
}
```

코틀린에서는 `is`로 타입을 검사하면, 검사에 성공한 블록 안에서 자동으로 해당 타입처럼 사용할 수 있다. 이를 **스마트 캐스트(Smart Cast)** 라고 한다.

```kotlin
fun printObject(obj: Any) =
    when (obj) {
        is String -> println(obj.lowercase())
        is Duration -> print(obj.nano)
        is LocalDateTime -> print(obj.month)
        else -> print("Unknown type")
    }
```

각 분기 안에서 `obj`는 자동으로 더 구체적인 타입으로 취급된다.

- `is String` 분기에서는 `obj.lowercase()`를 사용할 수 있다.
- `is Duration` 분기에서는 `obj.nano`에 접근할 수 있다.
- `is LocalDateTime` 분기에서는 `obj.month`에 접근할 수 있다.

즉, 타입 검사와 타입 캐스트가 동시에 일어나는 것처럼 사용할 수 있다.

---

## 9. 직접 형변환

스마트 캐스트가 자동으로 처리되지 않는 경우에는 직접 형변환을 할 수도 있다.

```kotlin
val dev = person as Developer
```

`as`는 명시적 형변환 연산자다. `person`을 `Developer` 타입으로 변환한다.

다만 `person`이 실제로 `Developer`가 아니라면 런타임 예외가 발생할 수 있다. 그래서 확실하지 않은 타입을 변환할 때는 안전한 형변환인 `as?`를 사용할 수 있다.

```kotlin
val dev = person as? Developer
```

`as?`는 변환에 실패하면 예외를 던지지 않고 `null`을 반환한다.

```kotlin
if (dev != null) {
    println(dev)
}
```

가능하다면 먼저 `is`를 사용한 타입 검사와 스마트 캐스트를 활용하고, 정말 필요한 경우에만 직접 형변환을 사용하는 편이 좋다.

---

## 10. if문에서의 스마트 캐스트

스마트 캐스트는 `when`뿐 아니라 `if`에서도 동작한다.

```kotlin
fun printObject(obj: Any): String =
    if (obj is String) {
        println(obj.lowercase())
        obj.lowercase()
    } else if (obj is Duration) {
        print(obj.nano)
        obj.nano.toString()
    } else {
        throw RuntimeException()
    }
```

`obj is String` 조건을 통과한 블록 안에서는 `obj`가 `String`처럼 취급된다. 그래서 `lowercase()` 메서드를 바로 호출할 수 있다.

`obj is Duration` 조건을 통과한 블록 안에서는 `obj`가 `Duration`처럼 취급된다. 그래서 `nano` 프로퍼티에 바로 접근할 수 있다.

코틀린의 `if`도 값을 반환할 수 있는 표현식이다. 그래서 위 함수는 `if` 전체의 결과를 함수의 반환값으로 사용한다.

---

## 11. 블록의 마지막 값

코틀린에서 `if`, `when` 같은 표현식의 각 분기가 블록(`{}`)으로 되어 있다면, 블록의 마지막 값이 그 분기의 결과값이 된다.

```kotlin
fun printObject(obj: Any): String =
    if (obj is String) {
        println(obj.lowercase())
        obj.lowercase()
    } else {
        "Unknown type"
    }
```

`obj is String` 분기에서는 마지막 줄인 `obj.lowercase()`가 결과값이 된다. `else` 분기에서는 `"Unknown type"`이 결과값이 된다.

이 개념은 코틀린 코드를 읽을 때 중요하다. 자바처럼 항상 `return`을 적는 방식에 익숙하다면 처음에는 어색할 수 있지만, 코틀린에서는 표현식의 마지막 값을 반환값처럼 사용하는 코드가 자주 등장한다.

다만 함수 본문을 중괄호로 작성한 **Block Body Function**에서는 반환값이 필요할 경우 `return`을 명시해야 한다.

```kotlin
fun printObject(obj: Any): String {
    return if (obj is String) {
        obj.lowercase()
    } else {
        "Unknown type"
    }
}
```

---

## 12. 정리

코틀린의 `when`, `enum`, 스마트 캐스트에서 이번 강의의 핵심은 다음과 같다.

- 코틀린의 `when`은 자바의 `switch`보다 더 유연한 선택 표현식이다.
- enum은 `enum class`로 선언하며, 프로퍼티와 메서드를 가질 수 있다.
- enum 상수 뒤에 본문을 선언하려면 세미콜론(`;`)으로 구분한다.
- `when`을 표현식으로 사용할 때는 모든 경우에 대한 결과값이 필요하다.
- enum을 다룰 때 `else` 없이 모든 값을 나열하면 새 enum 값이 추가되었을 때 컴파일러의 도움을 받을 수 있다.
- 인자가 없는 `when`은 `if`, `else if`, `else` 구조를 더 정돈된 형태로 표현할 수 있다.
- `is`로 타입을 검사하면 해당 블록 안에서 자동으로 스마트 캐스트가 적용된다.
- 직접 형변환이 필요할 때는 `as`를 사용할 수 있고, 실패 가능성이 있으면 `as?`를 고려한다.
- `if`와 `when`의 블록에서는 마지막 값이 해당 분기의 결과값이 된다.
