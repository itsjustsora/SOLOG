# 코틀린 클래스와 인스턴스

## 날짜
- 2026-06-25

## 🎯 오늘의 주제
- 자바 클래스와 코틀린 클래스 선언 방식의 차이 이해
- 코틀린의 프로퍼티가 필드, getter, setter를 함께 다루는 방식 정리
- 코틀린에서 인스턴스를 생성하고 프로퍼티에 접근하는 방법 이해
- 커스텀 getter를 사용해 계산된 프로퍼티를 표현하는 방법 정리
- 패키지와 import 문법, 최상위 함수 import 개념 이해

---

## 1. 자바 클래스와 코틀린 클래스 비교

먼저 사람을 표현하는 `Person` 클래스를 자바로 작성해보자.

```java
public class Person {
    private String name;
    private int age;
    private boolean isMarried;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public boolean isMarried() {
        return isMarried;
    }

    public void setMarried(boolean married) {
        isMarried = married;
    }
}
```

같은 클래스를 코틀린으로 작성하면 다음처럼 짧아진다.

```kotlin
class Person(val name: String, val age: Int, val isMarried: Boolean)
```

코틀린의 클래스 선언은 매우 간결하다. 자바에서 직접 작성하던 필드, 생성자, getter를 클래스 선언부에서 한 번에 표현할 수 있다. 마치 Lombok을 내장한 것처럼 반복 코드를 줄여주는 느낌에 가깝다.

---

## 2. 자바 클래스의 특징

자바에서는 일반적으로 필드를 `private`으로 숨기고, 외부에서는 getter와 setter를 통해 접근한다.

```java
private String name;

public String getName() {
    return name;
}

public void setName(String name) {
    this.name = name;
}
```

이 방식은 자바에서 오래 사용된 JavaBeans 스타일이다. 필드 자체를 직접 공개하지 않고, `field + getter + setter` 조합으로 하나의 프로퍼티처럼 사용하는 것이 일반적인 컨벤션이다.

또한 해당 필드를 초기화하는 생성자도 필요하다면 직접 선언해야 한다.

```java
public Person(String name, int age, boolean isMarried) {
    this.name = name;
    this.age = age;
    this.isMarried = isMarried;
}
```

즉, 자바에서는 단순히 데이터를 담는 클래스라도 필드, 생성자, getter, setter 같은 반복 코드가 많이 생기기 쉽다.

---

## 3. 코틀린 클래스의 특징

코틀린에서는 클래스 이름 옆에 생성자 파라미터와 프로퍼티를 함께 선언할 수 있다.

```kotlin
class Person(val name: String, val age: Int, val isMarried: Boolean)
```

이 코드에는 다음 의미가 들어 있다.

- `Person` 클래스가 선언된다.
- `name`, `age`, `isMarried` 프로퍼티가 선언된다.
- 세 프로퍼티를 초기화하는 주 생성자(primary constructor)가 만들어진다.
- `val`로 선언했기 때문에 각 프로퍼티는 읽기 전용으로 동작한다.
- 각 프로퍼티에 대한 getter가 자동으로 만들어진다.

코틀린에서 클래스 이름 옆 괄호에 작성하는 생성자를 **주 생성자(Primary Constructor)** 라고 한다. 주 생성자는 클래스의 가장 대표적인 생성자이며, 간단한 클래스에서는 이 선언만으로 충분한 경우가 많다.

만약 값을 변경할 수 있는 프로퍼티가 필요하다면 `var`를 사용할 수 있다.

```kotlin
class Person(var name: String, var age: Int, var isMarried: Boolean)
```

`var`로 선언한 프로퍼티는 getter와 setter가 모두 만들어진다. 반대로 `val`로 선언한 프로퍼티는 getter만 만들어지고 setter는 만들어지지 않는다.

코틀린에서는 getter와 setter를 직접 작성하는 경우가 많지 않다. 기본 동작을 바꾸고 싶을 때만 커스텀 접근자를 선언한다.

---

## 4. 프로퍼티와 필드

자바에서는 필드와 getter/setter를 분리해서 작성한다. 하지만 코틀린에서는 `name` 같은 선언을 보통 **프로퍼티(Property)** 라고 부른다.

```kotlin
class Person(val name: String)
```

여기서 `name`은 단순한 필드 하나만을 의미하지 않는다. 외부에서 값을 읽는 방법까지 포함한 개념이다.

```kotlin
val person = Person("James")
println(person.name)
```

겉으로는 필드에 직접 접근하는 것처럼 보이지만, 코틀린에서는 실제로 프로퍼티의 getter를 호출하는 방식으로 동작한다. 그래서 코틀린 코드에서는 `person.getName()`이 아니라 `person.name`처럼 사용한다.

이 차이를 이해하는 것이 중요하다. 코틀린은 객체의 상태를 단순 필드가 아니라 프로퍼티 중심으로 다룬다.

---

## 5. 인스턴스 생성과 프로퍼티 접근

자바에서는 객체를 만들 때 `new` 키워드를 사용한다.

```java
public static void main(String[] args) {
    Person person = new Person("James", 32, true);

    System.out.println(person.getName());
    System.out.println(person.isMarried());
}
```

코틀린에서는 `new` 키워드를 사용하지 않는다.

```kotlin
fun main(args: Array<String>) {
    val person = Person("James", 32, true)

    println(person.name)
    println(person.isMarried)
}
```

`Person("James", 32, true)`처럼 클래스 이름을 함수처럼 호출하면 인스턴스가 생성된다. 그리고 프로퍼티에 접근할 때는 getter 메서드를 직접 부르지 않고 `person.name`, `person.isMarried`처럼 사용한다.

---

## 6. Boolean 프로퍼티의 getter 차이

자바와 코틀린은 Boolean 프로퍼티를 사용할 때 문법적으로 차이가 있다.

자바에서는 Boolean 값을 읽기 위해 보통 `isMarried()` 형태의 getter를 호출한다.

```java
System.out.println(person.isMarried());
```

값을 변경하는 setter는 `setMarried()`처럼 사용한다.

```java
person.setMarried(false);
```

코틀린에서는 Boolean 프로퍼티도 다른 프로퍼티처럼 접근한다.

```kotlin
println(person.isMarried)
```

즉, 코틀린 코드에서는 `person.isMarried()`처럼 함수 호출 형태로 사용하지 않는다. `isMarried`는 함수가 아니라 프로퍼티이기 때문이다.

자바에서 코틀린 클래스를 사용할 때는 JVM getter 규칙에 따라 `isMarried()` 같은 메서드 형태로 보일 수 있다. 하지만 코틀린 코드 안에서는 프로퍼티 접근 문법을 사용한다고 이해하면 된다.

---

## 7. 커스텀 접근자

코틀린에서는 프로퍼티의 getter나 setter를 직접 정의할 수 있다. 이를 **커스텀 접근자(Custom Accessor)** 라고 한다.

다음은 사각형을 표현하는 `Rectangle` 클래스다.

```kotlin
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}
```

`isSquare`는 생성자에서 값을 직접 받은 프로퍼티가 아니다. `height`와 `width`를 비교해서 계산되는 프로퍼티다.

이 코드는 더 짧게 작성할 수도 있다.

```kotlin
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() = height == width
}
```

`isSquare`를 함수로 만들 수도 있다.

```kotlin
fun isSquare(): Boolean {
    return height == width
}
```

하지만 `isSquare`는 어떤 동작을 수행한다기보다 사각형이 정사각형인지 나타내는 특성에 가깝다. 그래서 함수보다 프로퍼티로 드러내는 것이 더 자연스럽다.

```kotlin
val rectangle = Rectangle(10, 10)

println(rectangle.isSquare)
```

이렇게 작성하면 `rectangle`의 특성을 읽는 코드처럼 보인다.

---

## 8. 패키지와 import

코틀린의 패키지와 import 문법은 자바와 거의 비슷하다.

```kotlin
package com.inflearn

import java.util.Random

class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}

fun createRandomRectangle(): Rectangle {
    val random = Random()
    return Rectangle(random.nextInt(), random.nextInt())
}
```

파일의 가장 위에 `package`를 작성하고, 그 아래에 필요한 클래스를 `import`한다. 그 다음 클래스, 함수, 프로퍼티를 선언한다.

자바와 다른 점은 코틀린에서는 클래스 밖에도 함수를 선언할 수 있다는 것이다. 위 예제의 `createRandomRectangle`은 클래스 내부에 들어 있지 않은 **Top-Level Function**이다.

---

## 9. 최상위 함수의 import

코틀린에서는 클래스뿐 아니라 최상위 함수도 import할 수 있다.

예를 들어 `com.inflearn` 패키지에 다음 함수가 있다고 하자.

```kotlin
package com.inflearn

fun createRandomRectangle(): Rectangle {
    val random = Random()
    return Rectangle(random.nextInt(), random.nextInt())
}
```

다른 파일에서는 이 함수를 직접 import해서 사용할 수 있다.

```kotlin
import com.inflearn.createRandomRectangle

fun main() {
    val rectangle = createRandomRectangle()
    println(rectangle.isSquare)
}
```

자바에서는 보통 클래스의 정적 메서드처럼 유틸리티 함수를 모아두는 경우가 많다. 코틀린에서는 꼭 그런 형태로 감싸지 않아도 파일 최상위에 함수를 선언하고 import해서 사용할 수 있다.

---

## 10. 정리

코틀린의 클래스와 인스턴스 문법에서 이번 강의의 핵심은 다음과 같다.

- 코틀린 클래스는 필드, 생성자, getter를 간결하게 선언할 수 있다.
- 클래스 이름 옆에 선언한 생성자는 **Primary Constructor**다.
- `val` 프로퍼티는 getter만 만들고, `var` 프로퍼티는 getter와 setter를 모두 만든다.
- 코틀린에서는 필드에 직접 접근하는 것처럼 보여도 실제로는 프로퍼티 접근 문법을 사용한다.
- 인스턴스를 만들 때 `new` 키워드를 사용하지 않는다.
- Boolean 프로퍼티도 코틀린에서는 `person.isMarried`처럼 접근한다.
- 계산된 값을 클래스의 특성으로 드러내고 싶을 때는 커스텀 getter를 사용할 수 있다.
- 클래스 밖에 선언한 함수는 **Top-Level Function**이며, 다른 파일에서 직접 import할 수 있다.
