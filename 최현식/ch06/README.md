컴퓨터는 여전히 0과 1만 인식할 수 있다. 하지만 가상머신이 등장하고 그 위에서 동작하는 언어가 등장하며, 프로그램을 네이티브코드로 직접 컴파일하지 않아도 되었다. 이는 곧 언어가 운영체제, 기계어에 종속되지 않는 플랫폼 독립적 저장 형식이 되었다.

클래스 파일은 자바 가상 머신 실행 엔진이 사용하는 데이터이다.
가상 머신 실행 엔진을 더 깊이 이해하기 위해서는 클래스 파일의 구조를 이해해야 한다.

클래스 파일을 구성하는 다양한 요소와 각 요소의 정의, 데이터 구조, 용도를 자세히 다룬다.
(클래스 데이터를 실제로 저장하고 내용에 접근하는 방법 등에 대해 다룸)











# 플랫폼 독립을 향한 초석 
> '한 번 작성하면 어디서든 실행된다'

다양한 하드웨어 아키텍처와 운영체제 조합으로 다양한 플랫폼이 생겨났다. 자바는 '플랫폼 독립적'이기 위하여 애플리케이션 계층에서 작업이 수행된다.

### 언어 독립성
언어 독립성을 보장하는 핵심은 가상 머신과 바이트코드 저장 형식

![image](https://github.com/user-attachments/assets/658fda7e-3cf9-4a8e-878b-b626df8cc62f)

- 자바 가상 머신은 어떠한 프로그래밍 언어에도 종속되지 않는다.
- '클래스 파일'이라는 특정한 바이너리 파일 형식에만 의존할 뿐이다.
- 클래스 파일은 자바 가상 머신 명령어 집합과 심벌 테이블 등의 정보가 담긴다.
- 바이트코드 형식은 TuringComplete하기 때문에 어떠한 언어도 표현할수 있도록 보장한다.
	- 바이트코드 명령어의 표현 능력이 자바 언어보다 뛰어나, 각 언어의 특징 구현 용이!


```ad-question
title: TuringComplete?
튜링 완전성은 **모든 계산 가능한 문제를 해결할 수 있는 능력**을 뜻합니다.  
JVM의 바이트코드가 튜링 완전하다는 것은:
- 이 바이트코드를 사용하면 **이론적으로 모든 프로그램**을 만들 수 있다는 말입니다.
    - 계산기 프로그램부터 게임, 웹 서비스까지 어떤 프로그램도 작성 가능!

튜링 완전한 시스템이 되려면 두 가지 조건을 충족해야 합니다:
1. **조건 분기**
    - 어떤 상황에서 다른 코드를 실행할 수 있는 능력.  
        예: `if`문이나 `while`문 같은 것.
    - JVM 바이트코드에는 `if`나 `goto` 같은 명령어가 있어서 이런 조건 분기가 가능합니다.
2. **메모리 활용**
    - 이론적으로는 무한한 메모리가 있어야 하지만, 현실에서는 가능한 범위 안에서 메모리를 잘 활용하면 됩니다.
    - JVM은 메모리를 동적으로 할당하여 큰 프로그램도 실행할 수 있도록 설계되었습니다.
```










# 클래스 파일의 구조
모든 클래스 파일은 각각 하나의 클래스 또는 인터페이스(또는 패키지, 모듈)를 정의한다. 반면 클래스나 인터페이스를 꼭 파일에 담아 둘 필요는 없다. 예를들어 동적으로 생성하여 클래스 로더에 직접 제공할 수 있다. (디스크에 파일 형태로 존재할 필요는 없다.)



## ✅ 클래스 파일의 구조
```c
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```
- 바이트를 하나의 단위로 하는 이진 스트림 집합체
- 각 데이터 항목이 정해진 순서에 맞게, 구분 기호 없이 조밀하게 나열됨
- 따라서 낭비되는 공간 없이 프로그램을 실행하는 데 꼭 필요한 데이터로 채워짐
- 1바이트가 넘는 데이터 항목은 바이트 단위로 분할되며, 큰 단위의 바이트가 먼저 저장되는 빅 엔디언 방식으로 표현
- 클래스 파일에 데이터를 저장할 때는 C언어의 구조체와 비슷한 '의사 구조'를 이용한다.
- 클래스 구조에서 데이터가 저장되는 바이트 순서, 각 바이트의 의미, 길이, 순서가 모두 엄격하게 제한되어 변경할 수 없다.

```ad-question
title: 의사구조?
- 부호 없는 숫자 (unsigned number)
u1, u2, u4, u8 데이터 타입으로 숫자, 인덱스 참조, 수량 값을 기술하거나 UTF-8로 인코딩된 문자열 값을 구성할 수 있다.
- 테이블
여러 개의 부호없는 숫자나 또 다른 테이블로 구성된 복합 데이터 타입을 표현한다. 구분이 쉽도록 테이블 이름은 관례적으로 '-info'로 끝난다. 테이블은 계층적으로 구성된 복합 구조의 데이터를 설명하는데 사용된다. 
-> 테이블 이름이 -info 라기보다는 항목(entry)의 구조라고 보는게 맞지않낭? 
```




## ✅ 클래스 파일 구조 상세
![image](https://github.com/user-attachments/assets/4d39840a-83df-42e0-b1df-da237a30c565)


### magic
클래스 파일 형식임을 나타내는 구분자로 파일의 처음에 `0xCAFEBABE` 형태로 위치

### minor_version, major_version
- 클래스 파일 형식의 버전을 결정하는 항목으로 M은 주 버전번호를, m은 부버전 번호를 나타내며 M.m으로 표시
- JVM 구현체는 이 버전정보를 보고 실행 가능한지 판단할 수 있음
- 예를 들어 JDK 1.0.2 Oracle JVM은 클래스 파일 45.0 ~ 45.3을 지원함

### 상수풀
상수 풀에는 리터럴과 심벌 참조 두가지 데이터 담기게 된다. (다른 클래스와 가장 많이 공유됨)
- 리터럴
	- 자바 언어 수준에서 이야기하는 상수(final 문자열, 상수)와 비슷한 개념
- 심벌 참조
	- 컴파일과 관련된 개념으로 다음 유형의 상수들이 포함됨
	- 모듈에서 익스포트하거나 임포트하는 패키지
	- 클래스와 인터페이스의 완전한 이름
	- 필드 이름과 서술자
	- 메서드 이름과 서술자
	- 메서드 핸들과 메서드 타입
	- 동적으로 계산되는 호출 사이트와 동적으로 계산되는 상수

자바에서 링크는 가상머신이 클래스 파일을 로드할 때 동적으로 이루어진다. 이 때 상수풀에서 각 항목의 심벌참조를 가져오며 동적으로 링크가 진행된다.

**constant_pool_count**
- 상수풀에 몇개의 항목이 있는지 나타내는 항목
- constant_pool 테이블 항목 수에 1을 더한 값과 같다.
**constant_pool[]**
- 클래스 파일 구조 내에서 참조되는 다양한 문자열 상수, 클래스 및 인터페이스 이름, 필드 이름, 기타 상수를 나타내는 구조 테이블
- 각 constant_pool 테이블 항목은 "tag" 바이트로 시작함
- constant_pool 테이블은 `constant_pool_count - 1` 까지 인덱싱 됨

### access_flags 
- 접근플래그를 통해 해당 클래스(또는 인터페이스)의 속성과 엑세스 권한을 나타냄
	- 현재 클래스파일이 클래스인지 인터페이스인지, public인지, abstract인지, 클래스인 경우 final인지 등의 정보가 딤김
	- protected 같은 접근제어자는 어떻게 구분할까? → 이를 표현하는 flag도 존재한다.
	- default 접근제어자는 어떻게 구분할까? → 어떠한 flag도 올라가있지 않다면 default이다.
- 이 외에 다양한 접근플래그가 미래를 위해 예약되어있다. 이러한 플래그는 파일에서는 0으로 설정되어야하며 JVM 구현체에서는 무시되야 한다. 

| Flag Name        | Value  | Interpretation                                                                                                                    |
| ---------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------- |
| `ACC_PUBLIC`     | 0x0001 | Declared `public`; may be accessed from outside its package.공개로 선언되었습니다. 패키지 외부에서 액세스할 수 있습니다.                                    |
| `ACC_FINAL`      | 0x0010 | Declared `final`; no subclasses allowed.최종 선언됨; 하위 클래스는 허용되지 않습니다.                                                                |
| `ACC_SUPER`      | 0x0020 | Treat superclass methods specially when invoked by the _invokespecial_ instruction.Invokespecial 명령어로 호출할 때 슈퍼클래스 메서드를 특별히 처리합니다. |
| `ACC_INTERFACE`  | 0x0200 | Is an interface, not a class.클래스가 아니라 인터페이스입니다.                                                                                   |
| `ACC_ABSTRACT`   | 0x0400 | Declared `abstract`; must not be instantiated.초록을 선언했습니다. 인스턴스화하면 안 됩니다.                                                          |
| `ACC_SYNTHETIC`  | 0x1000 | Declared synthetic; not present in the source code.합성으로 선언됨; 소스 코드에는 없습니다.                                                        |
| `ACC_ANNOTATION` | 0x2000 | Declared as an annotation type.주석 유형으로 선언됩니다.                                                                                     |
| `ACC_ENUM`       | 0x4000 | Declared as an `enum` type.열거형으로 선언됩니다.                                                                                           |
Link: https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/classfile/ClassFile.html

### this_class, super_class, interfaces[]
- 각각 클래스 인덱스, 부모클래스 인덱스, 인터페이스 인덱스를 나타냄
- 해당 정보들을 통해 클래스 파일의 상속 관계를 규정함
- 자바 언어는 다중 상속을 허용하지 안흐므로 부모 클래스 인덱스는 하나뿐임
- interfaces[]에서 인터페이스들의 순서는 implements 키워드 뒤에 나열한 순서와 동일

### 필드 테이블
**fields_count**
- fields 테이블의 field_info 구조 수를 나타냄
- field_info 구조는 해당 클래스(또는 인터페이스)에 의해 선언된 클래스 변수와 인스턴스 변수 모두를 포함한 모든 필드를 나타냄
- 메서드 안에 선언된 지역 변수는 필드가 아님
**fields[]**
- 이 클래스(또는 인터페이스)에서 선언된 필드를 정의
	- 접근제어자 정보 (public, private, protected)
	- 변수 정보 (static, final, volatile, transient)
	- 컴파일러가 자동 생성한 필드인지 (synthetic)
	- enum 여부 (enum)
- 슈퍼클래스(인터페이스)에서 상속된 필드를 나타내는 항목은 포함되지 않음

### 메서드 테이블
**methods_count**
- methods 테이블의 method_info 구조 수를 나타냄
**methods[]**
- 위 필드테이블 처럼 접근제어자 등을 제어가능 +
- 이 클래스(또는 인터페이스)에서 선언된 모든 메서드를 나타냄
	- 인스턴스 메서드, 클래스 메서드, 인스턴스 초기화 메서드, 인터페이스 초기화 메서드 등을 포함
- 슈퍼클래스(인터페이스)에서 상속된 메서드를 나타내는 항목은 포함되지 않음


### 속성 테이블
311페이지~338페이지
**attributes_count**
- attributes 테이블에 있는 속성 수(attributes_info)를 나타냄
**attributes[]**
- 속성 테이블에서는 아래와 같은 사양을 나타낼 수 있음
	- `InnerClasses` ([§4.7.6](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.6 "4.7.6. The InnerClasses Attribute")),
	- `EnclosingMethod` ([§4.7.7](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.7 "4.7.7. The EnclosingMethod Attribute")), 
	- `Synthetic` ([§4.7.8](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.8 "4.7.8. The Synthetic Attribute")), 
	- `Signature` ([§4.7.9](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.9 "4.7.9. The Signature Attribute")), 
	- `SourceFile` ([§4.7.10](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.10 "4.7.10. The SourceFile Attribute")), 
	- `SourceDebugExtension` ([§4.7.11](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.11 "4.7.11. The SourceDebugExtension Attribute")), 
	- `Deprecated` ([§4.7.15](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.15 "4.7.15. The Deprecated Attribute")), 
	- `RuntimeVisibleAnnotations` ([§4.7.16](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.16 "4.7.16. The RuntimeVisibleAnnotations attribute")), 
	- `RuntimeInvisibleAnnotations` ([§4.7.17](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.17 "4.7.17. The RuntimeInvisibleAnnotations attribute")), 
	- `BootstrapMethods` ([§4.7.21](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.21 "4.7.21. The BootstrapMethods attribute"))
- JVM 구현체는 인식하지 못하는 ClassFile 구조의 속성 테이블에 있는 속성들은 자동으로 무시해야 함
- 해당 스펙문서에서 정의하지 않은 속성으로 인해 클래스파일 동작에 영향이 생기면 안됨



## 속성 상세
### Code 속성


final static은 생성자가 없다.
Inner class는 실제로 하나의 파일로 정의되낭? 하나의 파일에는 하나의클래스인데~ 클래스앙네 클래스는 괜찮은가?


### 참고자료
https://blog.hexabrain.net/397

https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html











# 바이트코드 명렁어 소개
JVM 명령어는 특정 작업을 뜻하는 연산코드(opcode:바이트 길이의 숫자)와 해당 작업에 필요한 0개 이상의 피연산자로 이루어진다. 명령어 대부분이 피연산자 없이 연산 코드 하나로 구성되며, 피연산자는 스택에 저장된다.

### 바이트코드 연산 특징
```c
do {
	PC 레지스터의 값을 자동으로 계산한다.
	바이트코드 스트림에서 PC 레지스터가 가리키는 위치의 연산 코드를 가져온다.
	(피연산자가 필요한 바이트코드라면) 바이트코드 스트림에서 피연산자를 가져온다.
	연산 코드가 정의하는 동작을 수행한다.
} while(바이트코드 스트림 길이 > 0);
```
**단점**
- 연산 코드 길이가 1바이트 (0~255)로 제한되기 때문에 최대 256개의 연산코드만 표현할 수 있음
- 클래스 파일 구조에서는 컴파일된 코드에 들어있는 피연산자의 길이 정렬을 허용하지 않음
- 1바이트가 넘는 데이터를 처리할 때는 가상머신이 런타임에 해당 바이트들을 특정 구조로 재구성
- 이러한 작업 때문에 바이트코드를 해석하고 실행하는 속도가 조금 느려진다.
	- 예를 들어 부호 없는 16비트 정수는 부호 없는 바이트 2개를 이용하여 저장해야 한다.
	- (byte1 << 8) | byte2
**장점**
- 피연산자 길이 정렬을 포기하여, 수많은 패딩과 공백을 없앨 수 있다.
- 연산 코드들이 바이트 하나로 표현되기 때문에 컴파일된 결과물이 짧고 간결함
- 최소한의 데이터로 높은 전송 효율을 추구하는 설계

### Virtual Machine Errors
내부 오류나 자원 제한으로 인해 바이트코드 명령을 수행하지 못하는 경우 아래와 같은 에러가 발생한다. 모두 VirtualMethodError의 하위 클래스이다. 
- `InternalError`
	- 가상 머신을 구현하는 소프트웨어의 결함, 기본 호스트 시스템 소프트웨어의 결함 또는 하드웨어의 결함으로 인해 Java Virtual Machine 구현에서 내부 오류가 발생했습니다. 이 오류는 감지되면 비동기적으로 전달되며(§2.10) 프로그램의 어느 지점에서나 발생할 수 있습니다.
- `OutOfMemoryError`
	- 구현에 가상 또는 물리적 메모리가 부족하여 자동 저장소 관리자가 객체 생성 요청을 충족하기에 충분한 메모리를 회수할 수 없습니다.
- `StackOverflowError`
	- 구현에 스레드에 대한 스택 공간이 부족합니다. 일반적으로 실행 프로그램의 오류로 인해 스레드가 무제한의 재귀 호출을 수행하고 있기 때문입니다.
- `UnknownError`
	- 예외 또는 오류가 발생했지만 Java Virtual Machine 구현이 실제 예외 또는 오류를 보고할 수 없습니다.


## 바이트코드 명령어
https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-6.html
- 1바이트에 불과한 연산 코드에 데이터 타입 정보까지 포함시켜 명령어를 구성함
- 따라서 자주 쓰이는 연산과 데이터 타입 조합에만 전용 명령어를 배정
- 전용 명령어가 없는 타입은 별도 지시문을 이용해서 지원되는 타입으로 변환해서 사용함
	- `byte`, `char`, `short` 전용 명령어는 거의 없음
	- `boolean` 타입 전용 명령어는 하나도 없음
	- 컴파일러는 컴파일타임 도는 런타임에 `byte`, `short` 데이터를 `int`형으로 부호 확장(sign extension)한다.
	- `boolean`, `char` 데이터 역시 int 타입으로 제로 확장(zero extension)한다.
	- 이 네가지 데이터 타입 관련 연산은 실제로는 int 타입 연산 코드를 사용하여 수행된다. 

```ad-tip
title: 부호확장
출처: https://eunajung01.tistory.com/103#%EB%B6%80%ED%98%B8%ED%99%95%EC%9E%A5-sign-extention
```
![image](https://github.com/user-attachments/assets/8aadc8d0-cd4d-4114-bf85-3da41d2a778b)


### 로드(iload)와 스토어(istore) 명령어
스택  프레임의 `지역 변수 테이블`과 `피연산자 스택` 사이에서 데이터를 주고받는데 쓰인다.

| 특징     | 지역변수 테이블                  | 피연산자 스택                            |
| ------ | ------------------------- | ---------------------------------- |
| 역할     | 메서드의 지역 변수 및 매개변수를 저장.    | 명령어 실행 시 필요한 데이터를 저장.              |
| 구조     | 고정 크기의 슬롯 배열.             | 가변 크기의 LIFO 스택.                    |
| 생명 주기  | 메서드 호출 시 생성, 메서드 종료 시 제거. | 명령어 실행 중 임시적으로 사용.                 |
| 명령어 예시 | `iload`, `istore`         | `iadd`, `imul`, `invokevirtual` 등. |
- 지역변수 테이블은 메서드가 실행될 때 생성되는 메모리 영역으로, 메서드에 의해 선언된 지역 변수와 메서드의 매개변수를 저장합니다.
- 지역변수 테이블은 메서드의 각 프레임(Stack Frame)에 포함되며, 특정 메서드에 국한된 데이터를 관리합니다.
- 피연산자 스택은 JVM이 명령어를 처리하기 위해 사용하는 임시 계산 공간입니다.
- 각 메서드의 스택 프레임에 포함되며, 명령어 실행 중 필요한 피연산자 및 결과값을 저장합니다.

### 산술 명령어
산술 명령어들은 피연산자 스택의 값 두 개를 이용해 특정한 산술 연산을 수행하고, 결과값을 다시 피연산자의 스택의 맨 위에 저장한다.
- 정수 데이터의 나누기(idiv, ldiv)와 나머지(irem, lrem) 연산 시 나누는 값이 0이면 `ArithmeticException`을 던진다. (이게 유일하게 산술 과정에서 발생하는 예외)
- 자바 가상 머신은 IEEE754가 정의한 `비정규화된 부동 소수점 수`, `점진적 언더플로` 규칙을 지원해야한다.
- 오버플로가 발생해도 예외로 처리하지 않는다.
- 부동소수점 연산이 오버플로되면 `signed 무한대`로 표현 (e.g. Double.POSITIVE_INFINITY)
- 부동소수점 연산 결과에 대한 명확한 수학적 정의가 없다면 Nan 값으로 표현 (e.g. Double.NaN)
	- NaN은 정수형 데이터 타입에는 존재하지 않는다.
	- 예를 들어 0을 0으로 나눈 경우에 발생한다. 

```ad-question
title: 비정규화된 부동 소수점 수
숫자가 너무 작아서 컴퓨터가 보통 방식으로는 표현할 수 없을 때, **특별한 방식**으로 표현해요.  
이 방식은 숫자가 0에 더 가까워도 "0"으로 포기하지 않고 그 작은 값을 유지하게 해줍니다.

예시  
부동 소수점 숫자는 보통 **"정규화된 형태"**로 표현되는데, 이때는 숫자가 "1.xxx × 10의 거듭제곱" 형태를 가져요.  
하지만 숫자가 너무 작아지면, 더는 "1.xxx" 형태로 표현할 수 없어서, "0.xxx × 10의 거듭제곱" 같은 특별한 형태로 나타내요.
- 정규화된 수: 1.23×10−31.23 \times 10^{-3}1.23×10−3
- 비정규화된 수: 0.12×10−30.12 \times 10^{-3}0.12×10−3
이렇게 하면 아주 작은 값도 잃어버리지 않고 계산에 활용할 수 있어요.
```


```ad-question
title: 점진적 언더플로
컴퓨터가 숫자를 계산하다가 점점 작아져서 "0"에 가까워질 때, **갑자기 0으로 사라지지 않고** 아주 천천히 줄어들어요.  
이 과정은 계산 결과가 더 자연스럽고 정확하게 이어지도록 해줍니다.

예시  
만약 점진적 언더플로가 없다면, 숫자가 0.00000010.00000010.0000001에서 0.000000010.000000010.00000001로 줄어들다가 갑자기 "0"이 되어버릴 수 있어요.  
하지만 점진적 언더플로 덕분에, 숫자가 0.0000000010.0000000010.000000001, 0.00000000010.00000000010.0000000001처럼 점점 작아지다가 마지막에야 "0"이 됩니다.
```

### 형변환 명령어
- 숫자 타입 데이터를 다른 숫자 타입으로 변환한다. 
- 주로 개발자가 소스코드에 명시한 형 변환을 구현하거나, 주어진 피연산자의 데이터타입을 처리하는 전용 연산 코드가 없는 경우를 처리하는데 사용
- 데이터 타입의 표현 범위가 더 넓어지는 경우(int → long 처럼) 명령어 명시 없어도 JVM이 알아서 수행
- 표현 범위가 축소되는 경우에는 데이터 손실이 발생할 수 있어 반드시 명시해줘야 함

### 객체 생성과 접근 명령어
- 클래스 인스턴스와 배열도 객체다.
- 하지만 JVM은 이 두가지 객체의 생성ㅅ과 조작에 서로 다른 바이트코드 명령어를 사용
- 생성된 객체나 배열 인스턴스의 필드들 또는 배열의 원소들은 객체 접근 명령어를 이용해 얻어 올 수 있음

### 피연산자 스택 관리 명령어
피연산자 스택을 직접 조작하기 위한 명령어들
- 피연산자 스택에서 원소 꺼내기: pop, pop2
- 스택의 최상위 값 한개 또는 두개를 복제한 다음, 복제된 값을 스택 최상위에 다시 넣기: dup, dup_x1
- 스택의 최상에 있는 값 두개를 치환: swap

### 제어 전이 명령어
프로그램의 실행흐름을 조건에 따라 또는 무조건적으로 지정한 위치의 명령어로 이동시킨다.
- 이동할 위치의 명령어가 '제어 전이 명령어'가 아니어야한다.
- 개념적으로는 PC 레지스터의 값을 수동으로 변경한다고 이해할 수 있음
- goto …

### 메서드 호출과 반환 명령어
- `invokevirtual`
	- 객체의 인스턴스 메서드를 호출하며, 객체의 실제 타입에 따라 디스패치(가상 메서드 디스패치)
	- 자바 언어에서 가장 많이 쓰이는 메서드 디스패치 방식이다.
- `invokeinterface`
	- 인터페이스 메서드를 호출
	- 런타임에 이 인터페이스 메서드를 구현한 객체를 검색하여 적절한 메서드를 찾음
- `invokespecial`
	- 인스턴스 초기화 메서드, private 메서드, 부모 클래스의 메서드를 포함하여 특수 처리가 필요한 일부 인스턴스 메서드를 호출
- `invokestatic`
	- 클래스 메서드(static 메서드)를 호출
- `invokedynamic`
	- 런타임에 호출 사이트 한정자가 참조하는 메서드를 동적으로 찾아 호출
	- 앞의 4개 호출 명령어의 디스패치 로직은 사용자가 변경할 수 없으나, 이 명령어의 디스패치 로직은 가상 머신 실행시 사용자가 설정 가능
	- 람다!!!
  - ![image](https://github.com/user-attachments/assets/ccaac8b7-614e-4934-9f3b-2e43ca7b5f9d)


### 예외 처리 명령어
자바 프로그램에서 throw문으로 예외를 명시적으로 던지는 작업은 athrow 명령어로 구현된다. 자바 가상 머신은 예외처리(catch문)를 바이트코드 명령어 대신 예외테이블을 이용해 구현한다.

### 동기화 명령어
JVM은 메서드 수준 동기화와 메서드 안의 명령어 블록 동기화를 지원한다. 두 구조의 동기화 모두 구현에는 모니터를 이용한다. `synchronized` 







자바에도 NaN이 있다니!

record 클래스를 상속받는게 레코드 이다.




