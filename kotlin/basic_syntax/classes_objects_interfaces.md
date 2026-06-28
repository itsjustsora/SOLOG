# 코틀린 클래스, 객체, 인터페이스 활용

## 날짜
- 2026-06-28

## 🎯 오늘의 주제
- 인터페이스 구현과 상속 문법 정리
- `open`, `final`, `abstract` 변경자와 코틀린의 기본 상속 정책 이해
- `public`, `private`, `internal`, `protected` 가시성 변경자 정리
- `sealed class`를 사용해 제한된 계층을 표현하는 방식 이해
- 주 생성자, 부 생성자, 초기화 블록 사용 방식 정리
- backing field와 접근자 가시성 변경 이해
- `data class`, `object`, `companion object` 활용 방식 정리

---

## 1. 인터페이스와 구현

코틀린에서 인터페이스는 `interface` 키워드로 선언한다.

```kotlin
interface Clickable {
    fun click()
}
```

인터페이스를 구현하는 클래스는 콜론(`:`)을 사용한다.

```kotlin
class Button : Clickable {
    override fun click() {
        println("Button is clicked")
    }
}
```

```kotlin
fun main() {
    Button().click()
}
```

자바에서는 인터페이스 구현에 `implements`, 클래스 상속에 `extends`를 사용한다. 코틀린에서는 둘 다 콜론(`:`)으로 표현한다.

```java
class Button implements Clickable {
    @Override
    public void click() {
        System.out.println("Button is clicked");
    }
}
```

코틀린에서는 인터페이스의 메서드를 구현할 때 `override` 변경자를 반드시 붙여야 한다. 실수로 상위 타입의 메서드를 덮어쓰는 일을 막기 위한 명시적인 규칙이다.

---

## 2. 다중 인터페이스 구현

코틀린 클래스는 여러 인터페이스를 구현할 수 있다.

```kotlin
interface Clickable {
    fun click()

    fun showOff() = println("I'm clickable")
}

interface Focusable {
    fun showOff() = println("I'm focusable")
}
```

두 인터페이스가 같은 이름의 기본 구현 메서드를 가지고 있다면, 이를 동시에 구현하는 클래스에서는 컴파일 오류가 발생한다.

```kotlin
class Button : Clickable, Focusable {
    override fun click() {
        println("Button is clicked")
    }

    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

`Clickable`과 `Focusable`이 모두 `showOff()` 기본 구현을 제공하기 때문에, `Button` 입장에서는 어떤 구현을 사용할지 모호하다. 그래서 코틀린은 반드시 직접 `showOff()`를 오버라이드하도록 요구한다.

특정 인터페이스의 기본 구현을 호출하고 싶다면 `super<인터페이스명>.메서드명()` 형태를 사용한다.

---

## 3. open, final, abstract 변경자

코틀린의 클래스와 메서드는 기본적으로 `final`이다. 즉, 아무 표시가 없으면 상속하거나 오버라이드할 수 없다.

자바는 기본적으로 클래스와 메서드를 상속할 수 있고, 상속을 막고 싶을 때 `final`을 붙인다. 반면 코틀린은 기본적으로 막아두고, 허용하고 싶을 때 `open`을 붙인다.

```kotlin
open class RichButton : Clickable {
    fun disable() {}

    open fun animate() {}

    override fun click() {}
}
```

각 메서드의 의미는 다음과 같다.

- `disable()`: 기본적으로 final이므로 오버라이드할 수 없다.
- `animate()`: `open`이 붙어 있으므로 하위 클래스에서 오버라이드할 수 있다.
- `click()`: 상위 인터페이스 메서드를 오버라이드한 메서드다.

코틀린에서 오버라이드한 메서드는 기본적으로 다시 오버라이드 가능하다. 다시 막고 싶다면 `final override`를 사용할 수 있다.

```kotlin
final override fun click() {}
```

상속 가능한 설계는 신중해야 한다. 무분별한 상속과 오버라이드는 상위 클래스의 작은 변경이 하위 클래스에 예상치 못한 영향을 주는 **취약한 기반 클래스 문제(Fragile Base Class Problem)** 를 만들 수 있다. 코틀린이 기본적으로 final을 선택한 이유도 이 문제를 줄이기 위해서다.

---

## 4. Spring과 open 클래스

코틀린 클래스와 메서드가 기본적으로 final이라는 점은 Spring을 사용할 때 특히 중요하다.

Spring의 일부 기능은 메서드 호출 전후에 공통 처리를 끼워 넣는 방식으로 동작한다. 대표적으로 다음 기능들이 있다.

- `@Transactional`: 메서드 실행 전 트랜잭션을 시작하고, 성공 또는 실패에 따라 커밋/롤백 처리
- `@Cacheable`: 메서드 실행 전 캐시를 확인하고, 필요할 때만 실제 메서드 호출
- `@Async`: 메서드 호출을 비동기로 실행

예를 들어 `@Transactional`이 붙은 서비스가 있다고 하자.

```kotlin
@Service
class OrderService {
    @Transactional
    fun createOrder() {
        // 주문 생성 로직
    }
}
```

개발자는 `createOrder()`만 호출하는 것처럼 보지만, 실제로 Spring은 메서드 호출 앞뒤에 트랜잭션 처리를 추가해야 한다.

```text
호출자
  -> Spring Proxy
  -> 트랜잭션 시작
  -> 실제 OrderService.createOrder() 호출
  -> 커밋 또는 롤백
```

이때 Spring은 원본 객체를 바로 노출하지 않고, 앞에서 호출을 가로채는 **프록시 객체**를 만들어 사용한다. 프록시는 실제 객체를 감싸고 있다가 메서드 호출 전후에 필요한 공통 처리를 수행한다.

프록시를 만드는 방식은 크게 두 가지로 볼 수 있다.

- 인터페이스 기반 프록시: 인터페이스를 구현한 프록시 객체를 만든다.
- 클래스 기반 프록시: 원본 클래스를 상속한 프록시 객체를 만든다.

클래스 기반 프록시는 개념적으로 다음과 비슷하다.

```kotlin
open class OrderService {
    open fun createOrder() {
        // 주문 생성 로직
    }
}

class OrderServiceProxy : OrderService() {
    override fun createOrder() {
        // 트랜잭션 시작
        super.createOrder()
        // 커밋 또는 롤백
    }
}
```

문제는 코틀린에서 다음 클래스가 기본적으로 final이라는 점이다.

```kotlin
class OrderService {
    fun createOrder() {
        // 주문 생성 로직
    }
}
```

즉, 아무 표시가 없어도 상속과 오버라이드가 막혀 있다.

```text
final class OrderService
final fun createOrder()
```

이 상태에서는 Spring이 클래스 기반 프록시를 만들기 어렵다. 프록시가 원본 클래스를 상속하고 메서드를 오버라이드해야 하는데, 코틀린의 final 규칙 때문에 막히기 때문이다.

그래서 Kotlin + Spring 프로젝트에서는 보통 `kotlin-spring` 플러그인을 사용한다.

```kotlin
plugins {
    kotlin("plugin.spring") version "..."
}
```

이 플러그인은 Spring이 프록시로 다룰 가능성이 높은 클래스와 메서드를 컴파일 시점에 자동으로 open 처리해준다. 개발자가 매번 직접 `open`을 붙이지 않아도 된다.

예를 들어 다음 코드는 직접 `open`을 붙이지 않아도 Spring이 다루기 좋은 형태로 컴파일된다.

```kotlin
@Service
class OrderService {
    @Transactional
    fun createOrder() {
        // 주문 생성 로직
    }
}
```

개념적으로는 다음처럼 이해할 수 있다.

```kotlin
open class OrderService {
    open fun createOrder() {
        // 주문 생성 로직
    }
}
```

다만 모든 클래스를 open으로 만들 필요는 없다. Spring의 프록시 대상이 되는 클래스가 주로 문제가 된다.

open 처리가 필요한 대표적인 예시는 다음과 같다.

- `@Service` 클래스
- `@Repository` 클래스
- `@Controller`, `@RestController` 클래스
- `@Configuration` 클래스
- `@Transactional`, `@Cacheable`, `@Async`가 적용되는 클래스나 메서드

반대로 DTO, 요청/응답 객체, 단순 값 객체는 보통 open으로 만들 필요가 없다.

```kotlin
data class CreateOrderRequest(
    val productId: Long,
    val quantity: Int
)
```

이런 클래스는 Spring AOP 프록시 대상이 아니므로 기본 final 상태가 더 자연스럽다.

정리하면, Spring의 일부 애노테이션 기능은 호출 앞뒤에 공통 처리를 추가하기 위해 프록시를 사용한다. 클래스 기반 프록시를 만들려면 원본 클래스를 상속하고 메서드를 오버라이드할 수 있어야 한다. 하지만 코틀린은 기본적으로 final이기 때문에, Kotlin + Spring에서는 `kotlin-spring` 플러그인으로 필요한 클래스를 자동으로 open 처리하는 것이 일반적이다.

---

## 5. 가시성 변경자

코틀린의 주요 가시성 변경자는 다음과 같다.

- `public`: 어디서나 접근 가능
- `private`: 선언 위치에 따라 접근 범위가 달라짐
- `internal`: 같은 모듈 안에서만 접근 가능
- `protected`: 하위 클래스에서 접근 가능

코틀린은 기본 가시성이 `public`이다. 아무것도 붙이지 않으면 public으로 취급된다.

자바에는 아무 가시성도 붙이지 않았을 때 적용되는 package-private이 있다. 코틀린에는 자바의 package-private이 없다. 대신 `internal`이라는 가시성이 있다.

`internal`은 같은 모듈 안에서만 볼 수 있다. 여기서 모듈은 보통 Gradle 모듈이나 IntelliJ 프로젝트 설정에서 보이는 모듈 단위로 이해하면 된다.

```kotlin
internal class InternalService
```

`private`은 클래스 내부에서 사용하면 해당 클래스 안에서만 접근 가능하다. 파일 최상위에서 사용하면 해당 파일 안에서만 볼 수 있다.

```kotlin
private fun helper() {
    println("only in this file")
}
```

---

## 6. sealed class

`sealed class`는 상속 가능한 하위 타입을 의도적으로 제한하는 클래스다.

먼저 sealed class가 없는 경우를 보자.

```kotlin
interface Error

class FileError(val fileName: String) : Error

class DatabaseError(val dbmsType: DbmsType) : Error

enum class DbmsType {
    MYSQL, MARIA, ORACLE, H2
}

fun getCharacter(error: Error) =
    when (error) {
        is FileError -> "Error is occurred at ${error.fileName}"
        is DatabaseError -> "Error on DBMS : ${error.dbmsType}"
        else -> throw IllegalArgumentException("Unknown error type")
    }
```

`Error`가 인터페이스이면 컴파일러는 구현체가 어떤 것들이 있는지 모두 알기 어렵다. 그래서 `when`에서 `else`가 필요하다.

문제는 새로운 구현체가 추가되었을 때다. 여러 곳에서 `Error`를 `when`으로 분기하고 있다면, 모든 분기문을 직접 찾아 새 타입 처리를 추가해야 한다. 누락되면 런타임에 문제가 될 수 있다.

---

## 7. sealed class로 제한된 계층 만들기

`sealed class`를 사용하면 가능한 하위 타입을 제한할 수 있다.

```kotlin
sealed class Error {
    class FileError(val fileName: String) : Error()

    class DatabaseError(val dbmsType: DbmsType) : Error()
}

enum class DbmsType {
    MYSQL, MARIA, ORACLE, H2
}

fun getCharacter(error: Error) =
    when (error) {
        is Error.FileError -> "Error is occurred at ${error.fileName}"
        is Error.DatabaseError -> "Error on DBMS : ${error.dbmsType}"
    }
```

`sealed class`는 컴파일러가 가능한 하위 타입을 알 수 있게 해준다. 그래서 `when`에서 모든 하위 타입을 처리하면 `else`를 작성하지 않아도 된다.

이 방식의 장점은 변경에 강하다는 점이다. 새로운 하위 타입을 추가하면 기존 `when` 중 해당 타입을 처리하지 않은 곳에서 컴파일 오류가 발생한다. 덕분에 수정해야 할 분기문을 쉽게 찾을 수 있다.

`sealed class`는 다음과 같은 상황에 잘 어울린다.

- 성공/실패 결과 타입
- 화면 상태
- 제한된 에러 타입
- 외부에서 마음대로 확장되면 안 되는 도메인 상태

---

## 8. 주 생성자와 init block

코틀린 클래스 이름 뒤의 괄호 부분은 **주 생성자(Primary Constructor)** 다.

```kotlin
class User(val userName: String)
```

이 코드는 실제로 가장 많이 사용하는 형태다. 생성자 파라미터에 `val`이나 `var`가 붙으면, 해당 파라미터와 같은 이름의 프로퍼티가 함께 만들어진다.

가장 명시적으로 작성하면 다음과 같다.

```kotlin
class User constructor(_userName: String) {
    val userName: String

    init {
        userName = _userName
    }
}
```

`constructor` 키워드로 주 생성자를 명시하고, `init` 블록에서 생성자 파라미터를 프로퍼티에 할당한다.

`init` 블록은 객체가 생성될 때 실행되는 초기화 블록이다. 생성자 파라미터를 검증하거나, 프로퍼티를 초기화할 때 사용할 수 있다.

더 간단하게는 다음처럼 작성할 수 있다.

```kotlin
class User(_userName: String) {
    val userName: String = _userName
}
```

하지만 단순히 생성자 값을 프로퍼티로 저장하는 경우라면 처음 형태가 가장 간결하다.

```kotlin
class User(val userName: String)
```

---

## 9. 기본 파라미터 값을 갖는 생성자

생성자 파라미터에도 기본값을 지정할 수 있다.

```kotlin
data class User(
    val userName: String,
    val level: Int = 1
)
```

```kotlin
fun main() {
    println(User("Jekyll"))
    println(User("Hyde", 10))
}
```

`level`의 기본값이 `1`이므로 첫 번째 호출에서는 `level`을 생략할 수 있다.

```text
User(userName=Jekyll, level=1)
User(userName=Hyde, level=10)
```

함수의 디폴트 파라미터와 마찬가지로, 생성자에서도 기본값을 사용하면 불필요한 생성자 오버로딩을 줄일 수 있다.

---

## 10. 클래스 상속과 생성자 호출

상위 클래스를 상속할 때는 상위 클래스의 생성자를 호출해야 한다.

자바에서는 다음처럼 `super()`를 사용한다.

```java
public class Parent {
    private final String familyName;

    public Parent(String familyName) {
        this.familyName = familyName;
    }
}

public class Child extends Parent {
    private final String subName;

    public Child(String familyName, String subName) {
        super(familyName);
        this.subName = subName;
    }
}
```

코틀린에서는 클래스 선언부에서 상위 클래스 생성자를 호출할 수 있다.

```kotlin
open class Parent(
    val familyName: String
)

class Child(
    val subName: String,
    familyName: String
) : Parent(familyName)
```

`Child`는 `Parent`를 상속하면서 `Parent(familyName)`을 호출한다. 상위 클래스에 생성자가 있다면, 확장하는 쪽에서 반드시 어떤 방식으로든 해당 생성자를 호출해야 한다.

상위 클래스에 프로퍼티 없는 기본 생성자만 있더라도 호출 형태가 필요하다.

```kotlin
open class Parent

class Child : Parent()
```

---

## 11. 부 생성자

코틀린에서는 클래스 본문 안에 `constructor` 키워드로 **부 생성자(Secondary Constructor)** 를 만들 수 있다.

```kotlin
open class Parent(
    val familyName: String
)

class Child : Parent {
    private val subName: String

    constructor(subName: String) : this(subName, "")

    constructor(subName: String, familyName: String) : super(familyName) {
        this.subName = subName
    }
}
```

부 생성자에서 다른 생성자를 호출할 때는 `this()`를 사용한다.

```kotlin
constructor(subName: String) : this(subName, "")
```

상위 클래스 생성자를 호출할 때는 `super()`를 사용한다.

```kotlin
constructor(subName: String, familyName: String) : super(familyName)
```

코틀린에서는 주 생성자와 기본 파라미터 값으로 대부분의 생성자 요구를 해결할 수 있다. 부 생성자는 여러 생성 경로가 꼭 필요하거나, 자바 상호운용 때문에 필요한 경우에 주로 사용한다.

---

## 12. Backing Field

코틀린 프로퍼티의 getter와 setter는 직접 커스터마이징할 수 있다.

```kotlin
class Account {
    var balance: Long = 0
        set(value) {
            if (value < 0) throw IllegalStateException("잔액은 0원 이상만 가능합니다.")
            field = value
        }

    var accountName: String = ""
        get() = "계좌이름:$field"
}
```

`balance`의 setter는 음수 값이 들어오면 예외를 던진다. 검증을 통과한 값만 실제 프로퍼티에 저장한다.

여기서 `field`는 **backing field**에 접근하는 특별한 키워드다. getter와 setter 안에서만 사용할 수 있다.

```kotlin
field = value
```

만약 setter 안에서 `balance = value`처럼 프로퍼티 이름을 다시 사용하면 setter가 자기 자신을 계속 호출하게 된다. 그래서 실제 저장 공간에 접근할 때 `field`를 사용한다.

---

## 13. 접근자의 가시성 변경

getter와 setter의 가시성도 조절할 수 있다. 대표적으로 `private set`을 사용하면 외부에서는 값을 읽을 수만 있고, 직접 수정할 수는 없게 만들 수 있다.

```kotlin
class Account {
    var balance: Long = 0
        private set

    var accountName: String = ""
        get() = "계좌이름:$field"

    fun increaseBalance(value: Int) {
        this.balance += value
    }

    fun decreaseBalance(value: Int) {
        this.balance -= value
    }
}
```

```kotlin
fun main() {
    val account = Account()

    account.increaseBalance(100)
    account.accountName = "급여계좌"

    println("${account.accountName} 잔액 : ${account.balance}원")
}
```

`balance`는 외부에서 읽을 수는 있지만, 직접 대입할 수 없다.

```kotlin
// account.balance = 100 // 컴파일 오류
```

계좌 잔액처럼 중요한 값은 아무 곳에서나 수정되면 위험하다. `private set`을 사용하면 수정은 클래스 내부 메서드를 통해서만 가능하게 만들 수 있다.

---

## 14. data class

`data class`는 데이터를 저장하는 용도로 자주 사용하는 클래스다.

```kotlin
data class User(val name: String, val age: Int)
```

클래스 키워드 앞에 `data`만 붙이면 된다.

`data class`는 다음 기능을 자동으로 제공한다.

- `toString()`
- `equals()`
- `hashCode()`
- `copy()`
- 구조 분해를 위한 `componentN()` 함수

예를 들어 일반 객체의 기본 `toString()`은 내부 값을 알아보기 어렵다. 하지만 data class는 다음처럼 데이터를 포함한 문자열을 제공한다.

```text
User(name=John, age=42)
```

또한 `equals()`와 `hashCode()`가 자동으로 만들어지기 때문에, 객체 주소가 아니라 프로퍼티 값을 기준으로 동등성을 비교할 수 있다.

```kotlin
val user1 = User("John", 42)
val user2 = User("John", 42)

println(user1 == user2) // true
```

코틀린의 `==`는 내부적으로 `equals()`를 호출한다. data class에서는 값이 같으면 같은 객체처럼 비교된다.

---

## 15. object 키워드

`object` 키워드는 클래스 선언과 객체 생성을 동시에 한다. 주로 싱글턴 객체를 만들 때 사용한다.

```kotlin
object Family {
    val members = mutableListOf<Person>()
}
```

```kotlin
fun main() {
    Family.members.add(Person("snow", true))
}
```

`Family`는 별도로 생성자를 호출할 필요가 없다. 애플리케이션 안에서 하나의 인스턴스만 존재한다.

`object`는 다음과 같은 경우에 사용할 수 있다.

- 싱글턴 패턴처럼 인스턴스가 하나만 필요할 때
- Comparator 구현처럼 한 번만 사용할 객체가 필요할 때
- 상태를 공유하는 전역 객체가 필요할 때

다만 자바의 유틸 클래스처럼 static 메서드만 모아두는 용도라면, 코틀린에서는 `object`보다 최상위 함수를 사용하는 편이 더 자연스러운 경우가 많다.

---

## 16. 익명 객체

`object` 키워드는 익명 클래스에도 사용할 수 있다.

```kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

자바의 익명 클래스와 비슷하게, 어떤 클래스를 상속하거나 인터페이스를 구현하는 객체를 그 자리에서 바로 만들 수 있다.

한 번만 필요한 구현체를 위해 별도 클래스를 만들고 싶지 않을 때 유용하다.

---

## 17. companion object

코틀린에는 자바의 `static` 키워드가 없다. 대신 클래스 내부에 `companion object`를 선언해 정적 멤버처럼 사용할 수 있다.

```kotlin
class Child : Parent {
    private val subName: String
    private val age: Int

    constructor(subName: String, familyName: String, age: Int) : super(familyName) {
        this.subName = subName
        this.age = age
    }

    companion object {
        const val MAX_CHILDREN_COUNT = 4

        fun ofDefaultAge(
            subName: String,
            familyName: String
        ): Child = Child(subName, familyName, 0)
    }
}
```

`companion object` 안에는 상수나 팩토리 함수를 둘 수 있다.

```kotlin
val max = Child.MAX_CHILDREN_COUNT
val child = Child.ofDefaultAge("Jekyll", "Hyde")
```

자바에서 `static final` 상수나 `static factory method`를 만들던 패턴을 코틀린에서는 `companion object`로 표현할 수 있다.

다만 모든 함수를 companion object에 넣을 필요는 없다. 특정 클래스와 강하게 연결된 생성 함수나 상수라면 companion object가 적합하고, 독립적인 유틸 함수라면 최상위 함수가 더 간결할 수 있다.

---

## 18. 정리

코틀린 클래스, 객체, 인터페이스 활용에서 이번 강의의 핵심은 다음과 같다.

- 코틀린은 인터페이스 구현과 클래스 상속에 모두 콜론(`:`)을 사용한다.
- 인터페이스 메서드를 구현할 때는 `override`를 반드시 붙인다.
- 같은 기본 구현 메서드를 가진 인터페이스를 여러 개 구현하면 직접 오버라이드해야 한다.
- 코틀린 클래스와 메서드는 기본적으로 `final`이며, 상속과 오버라이드를 허용하려면 `open`을 붙인다.
- Spring의 프록시 기반 기능을 사용할 때는 클래스 기반 프록시 생성을 위해 open 처리가 필요할 수 있다.
- Kotlin + Spring에서는 `kotlin-spring` 플러그인이 Spring 주요 애노테이션이 붙은 클래스를 자동으로 open 처리해준다.
- 코틀린의 기본 가시성은 `public`이고, 같은 모듈 안에서만 보이게 하려면 `internal`을 사용한다.
- `sealed class`는 가능한 하위 타입을 제한해 `when` 분기를 더 안전하게 만든다.
- 클래스 이름 뒤 괄호는 주 생성자이며, `init` 블록은 객체 생성 시 실행되는 초기화 블록이다.
- 부 생성자는 `constructor`로 선언하며, `this()`나 `super()`로 다른 생성자를 호출한다.
- 커스텀 getter/setter 안에서는 `field`로 backing field에 접근한다.
- `private set`을 사용하면 외부에서 직접 값을 수정하지 못하게 막을 수 있다.
- `data class`는 데이터 저장용 클래스에 필요한 `toString`, `equals`, `hashCode` 등을 자동으로 제공한다.
- `object`는 클래스 선언과 객체 생성을 동시에 하며 싱글턴 객체를 만들 때 사용한다.
- `companion object`는 자바의 static 상수나 static factory method를 대체하는 용도로 사용할 수 있다.
