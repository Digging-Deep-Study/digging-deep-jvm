# 프론트엔드 컴파일과 최적화

javac와 같은 프론트엔드 컴파일러는 코드 실행 효율 부분은 거의 최적화를 하지 않는다.  
이게 어느 정도냐면 컴파일러 최적화 매개 변수인 -0을 지정해도 효과가 거의 없다.   
런타임에 실행 효율을 높이는 최적화는 JIT 컴파일러가 담당하고 컴파일타임에는 개발자의 코딩 효율을 높이는 최적화는 프론트엔드 컴파일러가 담당한다.

## javac 컴파일러

javac는 자바 소스 코드를 바이트 코드로 변환하는 컴파일러이다.  
그리고 javac 컴파일러는 순수하게 자바 코드로 작성 되어서 구성 되어 있다.

javac compile 과정은 아래와 같다.

1. plugin annotation processor 초기화
2. 소스 코드를 읽고 구문 분석 및 심벌 테이블 생성
3. annotation processor로 annotation 처리
4. 문법 검사, 타입 검사
5. 데이터 흐름 분석
6. 편의 문법 제거
7. 바이트 코드 생성

### 심벌 테이블 채우기

심벌 테이블은 심벌 주소와 심벌 정보의 매핑 정보를 가지고 있는 집합이다.  
컴파일러가 소스 코드를 읽고 분석할 때 생성된다.  
구현 방식은 정렬된 심벌 테이블, 트리 심벌 테이블, 해시 심벌 테이블, 스택 형태의 심벌 테이블 등이 있다.

### 에너테이션 처리

annotation은 원래 일반적인 자바 코드와 똑같게 설계되었지만 프로그램 실행 중에만 역할을 수행하는 특별한 형태의 인터페이스이다.  
그러다가 JDK6부터 플러그인 할 수 있는 annotation processor가 표준으로 추가되었다.  
하지만 특정 annotation은 컴파일 타임에 미리 처리 될 수 있어서 front-end 컴파일러 동작에 영향을 줄 수 있다.  
plugin annotation processor는 이 과정에서 구문 트리를 수정하고 추가 할 수 있다.  
그런데 구문 트리를 수정하면 컴파일러는 다시 심벌 테이블을 채워야 한다.  
이런 작업이 반복되면 컴파일 속도가 느려진다.
annotation processor는 구문 트리의 모든 요소, 코드 주석에도 접근할 수 있다.    
그래서 annotation processor는 컴파일러의 동작을 변경할 수 있어서 주의해서 사용해야 한다.

### 의미 분석과 바이트 코드 생성

추상 구문 트리는 프로그램 코드를 잘 구조화해서 표현하지만 논리적으로 의미를 분석하기에는 부족하다.

javac가 수행하는 컴파일 과정에서 의미 분석은 `특성 검사`와 `데이터 및 제어 흐름 분석`으로 나뉜다.  
특성 검사는 문법 검사와 타입 검사를 포함한다.

데이터와 제어 흐름 분석은 프로그램이 맥랑상 논리적으로 올바른지 확인하는 작업이다.  
예를 들어 지역 변수가 사용되기 전에 초기화 되었는지, 변수가 사용되는 범위를 확인하는 작업이다.

또 final keyword 같은 경우 실제로 바이트코드화 되었을 때 final 키워드가 붙은 변수와 붙지 않은 변수는 바이트 단위까지 정확하게 일치한다.  
그렇기에 지역 변수를 final로 선언해도 런타임에는 아무런 영향을 미치지 않는다.  
변수의 불변성은 오직 컴파일 타임에 javac 컴파일러에 의해서만 보장된다.

### 편의 문법 제거

컴파일러는 편의 문법을 제거하고 바이트 코드를 생성한다.  
자바에 있는 편의 문법은 제네릭, 가변 변수, 오토박싱/언박싱 등이 있다.  
이런 편의 문법은 컴파일러가 바이트 코드로 변환할 때 복잡한 과정을 거치기 때문에 컴파일 속도가 느려진다.

## 제네릭

제네릭은 컴파일 타임에 타입을 체크하고 형변환을 자동으로 처리해주는 기능이며 본질은 매개 변수화된 다형성이라고 할 수 있다.    
제네릭은 컴파일 타임에만 사용되고 런타임에는 제네릭 타입 정보가 사라진다.(타입 소거)  
제네릭은 하위 호환성을 위해 타입 소거 방식을 사용한다.  
타입이 소거 되면서 많은 문제가 생기는데 예를 들어 int, long, Object 사이의 직접적인 형 변환이 불가능해서 제네릭 정보가 소거되면서 형 변환을 할 수 없게 된다.  
이 문제를 해결하기 위해 Wrapper 클래스를 사용해서 형 변환을 해야 한다.  
그러다보니 오토박싱/언박싱이 발생하고 이로 인해 성능 저하가 발생한다.  
그리고 또 오버로딩 시 제네릭은 컴파일 타임에 타입 소거로 인해 오버로딩이 불가능하다.

아래 코드는 제네릭이 타입 소거로 인해 오버로딩이 불가능한 코드이다.

```kotlin
class Foo {
    fun foo(array: Array<Int>) {
        println("Array of Ints")
    }

    fun foo(array: Array<String>) {
        println("Array of Strings")
    }
}
```

이 문제를 해결하려고 나는 한가지 방법을 생각해냈다.
실제로 제네릭이 소거 되는 부분은 Object로 변경되기 때문에 메서드 네이밍을 변경해주면 오버로딩이 가능할 것 같다.

- @Mangling을 만들어서 컴파일 타임에 맹글링을 진행한다.
- Annotation Processor와 PlugIn, Front Compiler, Javac Compiler에 소스코드를 추가한다.
- 제네릭에 대한 정보를 가져와서 네이밍을 변경해준다.

**Before compile**

```kotlin
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.SOURCE)
annotation class Mangling

class Foo {
    @Mangling
    fun foo(array: Array<Int>) {
        println("Array of Ints")
    }

    @Mangling
    fun foo(array: Array<String>) {
        println("Array of Strings")
    }
}
```

After compile

```kotlin
object ManglingFactory {
    private const val ALLOWED_CHARACTERS = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789"
    val manglingKeyword = (1..6).map { ALLOWED_CHARACTERS.random() }.joinToString()
}


class Foo {
    fun foo_$
    { ManglingFactory.manglingKeyword }_Int(array: Array)
    {
        println("Array of Ints")
    }

    @Mangling
    fun foo_$
    { ManglingFactory.manglingKeyword }_String(array: Array)
    {
        println("Array of Strings")
    }
}
```

### 조건부 컴파일

자바 컴파일러는 자바 파일을 하나씩 처리하지 않고 컴파일 단위 전체를 포괄하는 구문 트리를 만들어서 최상위 노드부터 순회하면서 컴파일을 진행한다.  
이 말은 즉슨 컴파일러는 컴파일 단위 전체를 처리하기 때문에 조건부 컴파일이 불가능하다는 것이다.  
그런데 if 문 조건에 상수를 넣으면 조건부 컴파일을 할 수 있다.

```kotlin
class Foo {
    fun foo() {
        if (true) {
            println("true")
        } else {
            println("false")
        }
    }
}
```

이렇게 하면 if 문 조건에 상수를 넣어서 조건부 컴파일을 할 수 있다.  
하지만 이 방법은 상수를 넣어야 하기 때문에 유연성이 떨어진다.  