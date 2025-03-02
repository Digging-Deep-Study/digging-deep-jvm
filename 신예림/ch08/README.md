# 바이트코드 실행 엔진

가상 머신 구현에서 실행 엔진이 바이트코드를 실행하는 방법은 

1. 해석 실행 (인터프리터 실행)
2. 컴파일 실행 (JIT 컴파일러로 네이티브 코드 생성 후 실행)

중 하나이다. 실행 엔진 하나에서 두 가지 방식을 혼용할 수도, 여러가지 JIT 컴파일러를 혼용할 수 있다.

어떤 방식을 선택하더라도 **모든 자바 가상 머신의 실행 엔진은 같은 입력에 같은 출력을 낸다.** 다시 말해, 자바 가상 머신의 논리적 결과는 항상 동일해야 한다.

## 런타임 스택 프레임 구조

자바 가상 머신은 메서드를 가장 기본적인 실행 단위로 사용하며, 메서드 호출과 실행을 위해 스택 프레임 데이터 구조를 사용한다.
이 스택 프레임에는 메서드의 지역 변수 테이블, 피연산자 스택, 동적 링크, 반환 주소 등 같은 정보가 담긴다.

실행 엔진 관점에서는 활성 스레드에서 스택 맨 위에 있는 메서드만 실행중이며, 스택 맨 위에 있는 스택 프레임만 유효하다. (**현재 스택 프레임**)
현재 스택 프레임이 실행 중인 메서드를 현재 메서드라고 한다.

### 지역 변수 테이블

메서드 매개 변수와 메서드 안에서 정의된 지역 변수를 저장하는 공간이다. 
용량 기준은 가장 작은 단위인 변수 슬롯이다.

자바 가상 머신은 지역 변수 테이블을 인덱스 방식으로 사용한다.

메서드 호출 시 매개변수들도 지역 변수 테이블을 통해 전달된다.
인스턴스 메서드에서 지역 변수 슬롯의 0번째 슬롯에 this가 저장된다. 이후 메서드 매개 변수들과 메서드 본문의 지역 변수들이 정의 순서와 유효 번위에 따라 저장된다. 

스택 프레임이 소비하는 메모리 절약을 위해 변수 슬롯을 재활용할 수도 있으나, 변수 슬롯이 재사용되면 기존 객체 참조가 남아 GC가 수거하지 못하는 경우가 발생할 수 있다.
이 부작용에 대한 해법으로 `null` 값을 할당해 변수 슬롯의 기존 정보를 삭제할 수도 있다. 하지만 이 방법은 현재 권장되지 않는다. 대신,

1. 변수 번위를 적절히 지정해 변수가 회수되는 시간을 제어해라.
2. 변수를 사용할 때 명시적으로 초기화해라.
    - 지역 변수는 반드시 명시적으로 초기화해야 하며, 그렇지 않은 경우 컴파일 단계에서 오류가 발생한다.


### 피연산자 스택

메서드 실행 시작에서는 비어 있다가, 실행하는 동안 다양한 바이트코드 명령어가 피연산자 스택에 값을 push하거나 pop한다. 
이 스택 최상단에 위치한 값을 통해 연산을 수행한다.

피연산자 스택의 원소 데이터 타입은 바이트코드 명령어의 순서와 정확히 일치해야 한다.

### 동적 링크

메서드에서 이용하는 외부 객체를 가리키는 참조 정보를 저장한다.

### 반환 주소

메서드 종료 방법은 다음 2개 뿐이다.

1. 실행 엔진이 반환 바이트코드 명령어를 만나면 메서드를 종료한다.
2. 메서드 실행 중 예외가 발생하고, 메서드 본문에서 예외 처리가 이루어지지 않으면 종료된다.

두 가지 방식 모두 종료 후 메서드 호출 위치로 돌아가야 한다. 

정상 종료되는 경우 호출자의 호출자의 다음 실행 위치를 반환 주소로 사용하며, 비정상 종료의 경우 예외 핸들러 테이블에 의해 반환 주소가 결정된다.

## 메서드 호출

메서드 호출 단계에서는 호출할 메서드의 버전을 선택한다.

### 해석

메서드 호출 대상은 모두 클래스 파일의 상수 풀에 심벌 참조로 기록되어 있다.

호출 대상이 미리 특정되는 경우,(클래스 로딩 단계에서 직접 참조가 가능한 경우) 이를 정적 해석이라 한다. 이에 부합하는 메서드가 정적 메서드와 private 메서드이다.

메서드 호출 유형에 따라 사용되는 바이트코드 명령어가 다르다.

- `invokestatic` : 정적 메서드 호출
- `invokespecial` : 인스턴스 생성자, private 메서드, 부모 클래스의 메서드 호출

위 두가지 명령어로 호출할 수 있는 메서드는 해석 단계에서 고유한 호출 버전 특정이 가능하다.
`final`한정자가 붙은 메서드도 해석 단계에 특정이 가능하다. (오버라이딩 불가)
이러한 메서드 유형을 통틀어 비가상 메서드라고 하며, 그 외 메서드들을 가상 메서드라 한다.

- `invokevirtual` : 가상 메서드 호출
- `invokeinteface` : 인터페이스 메서드 호출
- `invokedynamic` : 메서드 실행 전 런타임에 동적으로 해석된다. 사용자가 설정한 시작 방식에 의해 디스패치 로직이 달라질 수 있다.

### 디스패치

디스패치 호출은 정적/동적일수도, 단일/다중일수도 있다. 이를 조합하면 4가지 유형이 생겨난다.

자바는 기본적으로 단일 디스패치 언어이지만, 오버로딩(Overloading)에서는 정적 디스패치 과정에서 다중 선택이 이루어진다.
반면, 오버라이딩(Overriding)에서는 동적 단일 디스패치가 적용된다.

### 정적 디스패치

정적 디스패치는 컴파일 타임에 결정되며, 메서드 오버로딩(Overloading)과 관련이 있다.
JVM은 변수의 정적 타입(컴파일 타임 타입)을 기준으로 호출할 메서드를 선택한다.

```java
public class StaticDispatch {    
    static abstract class Human { }    
    static class Man extends Human { }    
    static class Woman extends Human { }    
    
    // 오버로딩된 메서드(1)     
    public void sayHello(Human guy) {        
        System.out.println("Hello, guy!");    
    }    
    
    // 오버로딩된 메서드(2)     
    public void sayHello(Man guy) {        
        System.out.println("Hello, gentleman!");    
    }    
    
    // 오버로딩된 메서드(3)     
    public void sayHello(Woman guy) {        
        System.out.println("Hello, lady!");    
    }    
    
    public static void main(String[] args) {        
        Human man = new Man();      // 정적 타입 = Human, 실제 타입 = Man        
        Human woman = new Woman();  // 정적 타입 = Human, 실제 타입 = Woman        
        
        StaticDispatch sr = new StaticDispatch();        
        sr.sayHello(man);    // Hello, guy!        
        sr.sayHello(woman);  // Hello, guy!

        // 명시적 타입 캐스팅 적용
        sr.sayHello((Man) man);    // Hello, gentleman!        
        sr.sayHello((Woman) woman); // Hello, lady!
    }
}
```

위 코드에서 `Human`을 변수의 정적 타입(=겉보기 타입)이라 하고, `Man`/`Women`을 실제 타입(=런타임 타입) 이라고 한다.

두 타입 모두 프로그램이 실행되는 동안에는 변경될 수 있으나, 정적 타입은 변수가 사용될 때만 변경되며 **변수 자체의 정적 타입은 변하지 않는다.**

가상 머신에서 호출할 메서드를 선택할 때 변수의 실제 타입이 아닌 **정적 타입을 참고**한다.


#### 메서드 오버로딩 우선순위

자바에서 오버로딩된 메서드가 여러 개 있을 경우, 선택 우선순위는 다음과 같다.

1. 정확한 타입 매칭 (예: `char` → `char`)  
2. 자동 형 변환 (예: `char` → `int`, `int` → `long`)  
3. 박싱 (예: `char` → `Character`)  
4. 상위 클래스 변환 (예: `Character` → `Serializable`, `Character` → `Object`)  
5. 가변 길이 매개변수 (varargs, 최후의 선택)

```java
import java.io.Serializable;

public class Overload {    
    public static void sayHello(char arg) {        
        System.out.println("Hello char");    
    }  

    public static void sayHello(int arg) {        
        System.out.println("Hello int");    
    }    
    
    public static void sayHello(long arg) {        
        System.out.println("Hello long");    
    } 
    
    public static void sayHello(Character arg) {        
        System.out.println("Hello Character");    
    }    
    
    public static void sayHello(Serializable arg) {        
        System.out.println("Hello Serializable");    
    }    

    public static void sayHello(Object arg) {        
        System.out.println("Hello Object");    
    }    
    
    public static void sayHello(char... arg) {        
        System.out.println("Hello char ...");    
    }    
    
    public static void main(String[] args) {        
        sayHello('a');  // Hello char    
    }
}
```

'a'는 char 타입이므로 sayHello(char)가 가장 먼저 선택된다.

만약 sayHello(char)를 주석 처리하면, char → int로 변환되어 sayHello(int)가 선택된다.

그다음 sayHello(long) → sayHello(Character) → sayHello(Serializable) → sayHello(Object) → sayHello(char...) 순서로 선택된다.

기본적으로 가장 구체적인 타입이 우선 선택되며 이후는 상대적 우선순위에 의해 결정된다.

### 동적 디스패치

```java
public class DynamicDispatch {    
    static abstract class Human {        
        protected abstract void sayHello();    
    }    
    
    static class Man extends Human {        
        @Override        
        protected void sayHello() {            
            System.out.println("Man said hello");        
        }    
    }    
    
    static class Woman extends Human {        
        @Override        
        protected void sayHello() {            
            System.out.println("Woman said hello");        
        }    
    }    
    
    public static void main(String[] args) {        
        Human man = new Man();      // 정적 타입 = Human, 실제 타입 = Man        
        Human woman = new Woman();  // 정적 타입 = Human, 실제 타입 = Woman        

        man.sayHello();     // 실제 타입이 Man이므로 → "Man said hello"        
        woman.sayHello();   // 실제 타입이 Woman이므로 → "Woman said hello"        

        man = new Woman();  // man의 실제 타입을 Woman으로 변경        
        man.sayHello();     // 실제 타입이 Woman이므로 → "Woman said hello"    
    }
}
```

위 코드에서 알 수 있듯, 오버라이딩 된 메서드는 실제 타입에 의거해 메서드를 호출한다. 

자바 가상 머신에서는 `invokevirtual` 명령어가 후보 메서드를 찾아 이 중 하나를 특정하는 방식으로 오버라이딩 다형성을 구현한다. 명령어의 런타임 해석은 다음 4단계로 이루어진다.

1. 피연산자 스택 상단 첫 번째 요소가 가리키는 객체의 **실제 타입(C라고 가정)**을 찾는다.
2. 타입 C에서 상수의 서술자 및 단순 이름과 일치하는 메서드를 찾으면 접근 권한이 있는지 검사한다. 권한이 있다면 이 메서드의 직접 참조를 반환하고 검색을 끝낸다. 권한이 없다면 `IllegalAccessError`를 던진다.
3. 그렇지 않으면 상속 계층을 따라 아래에서 위로 C의 상위 클래스에 대해 2번 과정을 수행한다.
4. 최상위 클래스까지도 적절한 메서드를 찾지 못하면 `AbstractMethodError`를 던진다.

또한, 필드의 값은 다형성과 무관하다. 상위 클래스 필드와 이름이 동일한 하위 클래스 필드를 선언하면 하위 클래스의 필드가 상위 클래스의 필드를 가린다.(Hiding)

```java
public class FieldHasNoPolymorphic {    
    static class Father {        
        public int money = 1;        
        
        public Father() {            
            money = 2;            
            showMeTheMoney();    
        }        
        
        public void showMeTheMoney() {            
            System.out.println("I am a Father, I have $" + money);        
        }    
    }    

    static class Son extends Father {        
        public int money = 3;        
        
        public Son() {            
            money = 4;            
            showMeTheMoney();        
        }        
        
        public void showMeTheMoney() {            
            System.out.println("I am a Son, I have $" + money);        
        }    
    }    

    public static void main(String[] args) {        
        Father guy = new Son();        
        System.out.println("This guy has $" + guy.money);    
    }
}

// 실행 결과
// I am a Son, I have $0
// I am a Son, I have $4
// This guy has $2
```

위 코드에서, 생성자 호출 순서 때문에 `Father` 생성자가 먼저 실행되고, `showMeTheMoney()`는 실제 객체의 타입(`Son`) 기준으로 실행된다. 이때 `Son.money` 는 Son 생성자 실행 전까지 0이므로, Son 생성자 실행 전 출력된 값은 0이다. 필드는 오버라이딩되지 않고 숨겨지므로(Hiding), Father 타입 변수로 접근하면 Father.money 값(2)이 출력된다.


### 단일 디스패치와 다중 디스패치

메서드의 수신 객체와 매게 변수를 합쳐 메서드 볼륨이라 한다.

단일 디스패치는 한 볼륨 내에서 대상 메서드를 선택하고, 다중 디스패치는 둘 이상의 볼륨 내에서 대상을 찾는다.

```java
public class Dispatch {    
    static class QQ {}    
    static class _360 {}    

    public static class Father {        
        public void hardChoice(QQ arg) {            
            System.out.println("Father chose a qq");        
        }        
        public void hardChoice(_360 arg) {            
            System.out.println("Father chose a 360");        
        }    
    }    

    public static class Son extends Father {        
        public void hardChoice(QQ arg) {            
            System.out.println("Son chose a qq");        
        }        
        public void hardChoice(_360 arg) {            
            System.out.println("Son chose a 360");        
        }    
    }    

    public static void main(String[] args) {        
        Father father = new Father();        
        Father son = new Son();        

        father.hardChoice(new _360());  // "Father chose a 360"
        son.hardChoice(new QQ());       // "Son chose a qq"
    }
}
```

컴파일 타임에는 변수 타입과 매개변수 타입을 기준으로 메서드를 결정하므로, 정적 디스패치 과정에서 다중 선택이 이루어진다.
런타임에는 객체의 실제 타입에 따라 실행될 메서드가 결정되므로, Java는 동적 디스패치 시 단일 디스패치를 수행한다.

이러한 특성은 Java의 현재 패러다임에 해당하며, 언어적 확장(C#의 dynamic 타입 추가 등)에 따라 변할 수 있다.

## 동적 타입 언어 지원

`invokedynamic` 명령어와 `java.lang.invoke` 패키지가 도입되면서, JVM에서 동적 타입 언어를 더 효율적으로 실행할 수 있도록 지원하게 되었다.

동적 타입언어는 타입 검사 과정 중 주요 단계가 런타임에 수행된다. 클로저, 그루비, 자바스크립트, PHP, 루비, 파이썬 등이 그 예이다. 반대로 타입 검사를 컴파일타임에 수행하는 언어로는 Java와 C++이 있다.

변수의 타입을 컴파일타임에 결정하는 정적 타입 언어의 가장 큰 이점은 컴파일러에 의해 타입 검사가 포괄적이고 엄격하게 수행된다는 점이다. 상대적으로 안정성이 뛰어나다. 반면 동적 타입 언어는 타입이 런타임에 결정되므로 개발자가 더 많은 자유를 누릴 수 있다.

### 자바와 동적 언어 타이핑

자바 가상 머신(JVM)은 원래 자바 언어를 위해 설계되었지만, 동적 타입 언어(예: 클로저, 그루비, JRuby)도 실행할 수 있도록 확장되었다. 그러나 JVM의 정적 타입 시스템과 동적 타입 언어의 런타임 메서드 결정 방식 간의 차이로 인해 성능 저하와 메모리 부담이 발생했다. 

특히 메서드 호출 최적화가 어려워 동적 언어 지원에 한계가 있었다. 이를 개선하기 위해 `invokedynamic` 과 `java.lang.invoke` 패키지가 도입되었다.

### `java.lang.invoke`

실제 실행 시점에 동적으로 메서드 해석을 수행할 수 있도록 지원한다. (메서드 핸들)

기본적인 사용 방식은 아래와 같다.

```java
import static java.lang.invoke.MethodHandles.lookup;
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodType;

public class MethodHandleTest {
    static class ClassA {
        public void println(String s) {
            System.out.println(s);
        }
    }

    public static void main(String[] args) throws Throwable {
        Object obj = System.currentTimeMillis() % 2 == 0 ? System.out : new ClassA();
        // obj의 타입이 무엇이든 다음 문장은 println() 메서드를 문제없이 호출한다.
        getPrintlnMH(obj).invokeExact("icyfenix");
    }

    private static MethodHandle getPrintlnMH(Object receiver) throws Throwable {
        // MethodType은 메서드의 반환값(methodType()의 첫 번째 매개 변수)과
        // 특정 매개 변수(methodType()의 두 번째 이후 매개 변수들)를 포함하는
        // '메서드 타입'을 뜻한다.
        MethodType mt = MethodType.methodType(void.class, String.class);
        
        // lookup() 메서드는 MethodHandles.lookup을 뜻한다.
        // 이 문장의 기능은 주어진 클래스에서 메서드 이름, 메서드 타입, 호출 권한이
        // 일치하는 메서드 핸들을 찾는 것이다.
        // 이때 가상 메서드가 호출되기 때문에 자바 언어의 규칙에 따라 메서드의
        // 첫 번째 매개 변수는 암묵적으로 메서드의 수신 객체, 즉 this가 가리키는
        // 객체를 나타낸다.
        // 이 매개 변수는 원래 매개 변수 목록으로 전달되었지만,
        // 지금은 bindTo() 메서드가 이 기능을 제공한다.
        return lookup().findVirtual(receiver.getClass(), "println", mt).bindTo(receiver);
    }
}

```

이 방식은 리플렉션 API과 유사한데, 가장 큰 차이는 리플렉션은 Java 언어를 위해 설계되었으나 메서드 핸들은 JVM에서 동작하는 모든 언어를 지원하기 위해 설계되었다는 것이다.

### `invokedynamic`

jdk 7에서 추가된 바이트코드 명령어이다.

기존 `invoke**` 명령어(`invokevirtual`, `invokestatic`, `invokespecial`, `invokeinterface`)는 JVM 내부에 메서드 해석 로직이 고정되어 있었다. 그러나 `invokedynamic`은 메서드 해석 책임을 **부트스트랩 메서드(BSM, Bootstrap Method)**로 위임할 수 있도록 한다. 즉, 개발자가 직접 디스패치 규칙을 정의할 수 있다.

동작 방식
1. `invokedynamic` 명령어는 첫 번째 매개변수로 `CONSTANT_InvokeDynamic_info` 상수를 가진다.
2. 해당 상수에는 부트스트랩 메서드(BSMs), 메서드 이름, 메서드 타입 정보가 포함된다.
3. 부트스트랩 메서드는 `CallSite` 객체를 반환하며, 이는 이후 해당 `invokedynamic` 호출을 최적화하여 캐싱하는 역할을 한다.

즉, `invokedynamic`을 사용하면 동적 메서드 호출을 최적화하고, JVM이 특정 언어의 호출 패턴을 학습하여 최적의 성능을 발휘할 수 있도록 도와준다.


## 스택 기반 바이트코드 해석 및 실행 엔진

javac 컴파일러는 프로그램 코드의 어휘 분석, 구문 분석, 추상 구문 트리 생성, 바이트코드 명령어 생성 과정을 처리한다. 이 작업 대부분은 가상 머신 외부에서 수행되며, 인터프리터는 가상 머신 내부에서 실행되므로, 자바 프로그램의 컴파일 과정은 가상 머신과 독립적으로 이루어진다.

### 스택 기반 명령어 집합과 레지스터 기반 명령어 집합

javac 컴파일러가 출력하는 바이트코드 명령어 스트림은 기본적으로 스택 기반 명령어 집합 아키텍처를 따른다. 다시 말해, 바이트코드 명령어 스트림은 대부분 메모리 주소 대신 피연산자 스택을 이용해 동작한다.

스택 기반 명령어 집합은 하드웨어 레지스터를 직접 사용하지 않는다. 대신, 가상 머신은 프로그램 카운터나 스택 최상위 데이터를 내부적으로 캐싱하여 성능을 최적화하려 한다. 이 방식은 하드웨어 의존성을 낮추는 장점이 있으며, 바이트코드 명령어가 상대적으로 단순하고 짧아질 수 있다는 이점도 있다.

단점은, 레지스터 기반 명령어 집합에 비해 실행 속도가 상대적으로 느리다는 점이다. 스택 기반 명령어 집합은 연산을 수행할 때마다 오퍼랜드를 스택에서 푸시/팝해야 하므로, 레지스터 기반 방식보다 메모리 접근이 빈번하게 발생하여 성능이 저하될 수 있다.

---

출처

- JVM 밑바닥까지 파헤치기 8장
- [Understanding Java method invocation with invokedynamic](https://blogs.oracle.com/javamagazine/post/understanding-java-method-invocation-with-invokedynamic)
- [JVM Specification - Chapter 6: The Java Virtual Machine Instruction Set - `invokedynamic`](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html#jvms-6.5.invokedynamic)

