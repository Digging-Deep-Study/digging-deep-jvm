# 클래스 파일 구조

## 플랫폼 독립을 향한 초석

Java는 가상 머신에 의해 운영 체제나 기계어에 종속되지 않는, **플랫폼 독립적 저장 형식**을 택한다. 

또한, Java는 초기 설계 단계부터 가상 머신에서 다른 언어를 실행할 가능성을 염두에 두고 제작되어 타 언어를 지원할 수 있도록 하였다. (**언어 독립성**) 

이러한 특성 덕에, 오늘날 `Scala`, `Kotlin`, `Groovy` 등 다양한 언어가 JVM에서 실행될 수 있다.

이러한 언어 독립성을 보장하는 핵심은 **가상 머신**과 **바이트코드 저장 형식**이다. 

- 바이트코드 저장 형식 (`.class`) 
	- 컴파일러가 자바 class(`.java`)들을 바이트코드 파일(`.class`)로 변환한다.
- 가상 머신 
	- 바이트코드 파일을 클래스 로더에 의해 해석하고, JVM 메모리에 로드시킨다.

## 클래스 파일 구조 및 속성

### `.class` 파일 구조

Java의 클래스 파일 기본 구조는 같은 의사 구조(C의 구조체와 유사하다.)를 따른다. 이 구조는 부호 없는 숫자와 테이블로 구성된다. 

- 부호 없는 숫자 
	- 기본 데이터 타입
	- u1, u2, u4, u8(1, 2, 4, 8바이트)
- 테이블 
	- 여러개의 부호 없는 숫자나 다른 테이블로 구성된 복합 데이터 타입
	- 관례적으로 `_info`로 끝난다.

각 항목은 아래와 같이 **정해진 순서에 따라 나열**된다.

#### 클래스 메타 정보

- 매직 넘버 
	- 모든 클래스 파일의 처음 4바이트는 매직 넘버로 시작한다.
	- 해당 파일이 가상 머신이 허용하는 `.class` 파일임을 명시한다.  (`OXCAFEBABE`)

- Minor version(`minor_version`)
	- 2바이트로 JDK 마이너 버전을 명시한다.
- Major version(`major_version`)
	- 2바이트로 JDK 메이저 버전을 명시한다.

- Constant pool
	- `constant_pool_count` : 2바이트로, 상수 풀 항목 개수를 알려준다.
	- `constant_pool\[]` 
		- 각 원소는 `cp_info` 구조를 따른다.
		- 문자열, 클래스와 인터페이스명, 필드명 등 현재 클래스 파일의 다양한 상수를 명시한다.
		- 각 `constant_pool` 테이블의 시작은 `tag` 바이트로 시작한다.
		- 1부터  `constant_pool_count- 1` 까지의 인덱스를 가진다.
		- 0번째 인덱스: 상수 풀 인덱스를 가리키는 데이터에서 상수 풀 항목을 참조하지 않음을 표현해야 하는 특수한 경우에, 이 인덱스를 0으로 설정한다.
```java
cp_info {
    u1 tag;
    u1 info[];
}
```


- Access flags(`access_flags`)
	- 2바이트로, 해당 클래스 혹은 인터페이스 접근 권한을 명시한다.

| Flag명           | 값     | 설명                                                                |
| ---------------- | ------ | ------------------------------------------------------------------- |
| `ACC_PUBLIC`     | 0x0001 | `public`으로 선언. 패키지 외부에서 접근 가능.                       |
| `ACC_FINAL`      | 0x0010 | `final`로 선언. 하위 클래스가 허용되지 않음.                        |
| `ACC_SUPER`      | 0x0020 | `_invokespecial_` 명령어에 의해 상위 클래스 메서드를 특별히 처리함. |
| `ACC_INTERFACE`  | 0x0200 | 인터페이스                                                          |
| `ACC_ABSTRACT`   | 0x0400 | `abstract`로 선언. 인스턴스화 불가                                  |
| `ACC_SYNTHETIC`  | 0x1000 | `synthetic`으로 선언. 소스 코드에 존재하지 않음.                    |
| `ACC_ANNOTATION` | 0x2000 | 애노테이션 타입으로 선언.                                           |
| `ACC_ENUM`       | 0x4000 | `enum` 타입으로 선언.                                               |
| `ACC_MODULE`     | 0x8000 | 모듈인지 여부                                                       | 


다음 클래스 파일의 정보들은 클래스 파일 간 상속 관계를 정의하며, 인덱스 정보들은 상수 풀에 대해 유효한 인덱스로 구성된다.
- This class(`this_class`)
	- 현재 클래스의 인덱스를 표기한다.
- Super class(`super_class`)
	- 부모 클래스의 인덱스를 표기한다.
	- `java.lang.Object`를 제외한 모든 자바 클래스의 부모 클래스 인덱스는 값이 0이 아니다.
- Interfaces
	- `interfaces_count`
		- 현재 클래스 / 인터페이스의 direct superinterface 개수를 표기한다.
	- `interfaces[]` 
		- 구현된 인터페이스의 순서대로 인덱스를 표기한다.
		- 인덱스의 개수는 0 ≤ `i` < `interfaces_count`

- Fields 
	- `fields_count` 
		- 현재 클래스 / 인터페이스의 필드 수(클래스 변수, 인스턴스 변수)를 나타낸다.
	- `fields[]` :
		- 각 원소는 `field_info` 구조를 따른다.
		- 각 원소는 필드의 Access Flag(접근 권한), 필드명, type 등을 나타낸다.
		- 부모 클래스나 부모 인터페이스로부터 상속받은 필드는 필드 테이블 컬렉션에 나열하지 않는다.
```java
field_info {
    u2             access_flags;
    u2             name_index;       // 필드의 단순 이름
    u2             descriptor_index; // 필드 및 메서드 서술자
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

- Methods
	- `methods_count`
		- 메서드 테이블의 갯수를 나타낸다.
	- `methods[]`
		- 각 원소는 `method_info` 구조를 따른다.
```java
method_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```


#### 클래스 속성 정보

`final, Exceptions, SourceFile, Generics, Code` 등과 같은 클래스 속성 정보를 나타낸다.

- Attributes
	- `attributes_count`
	- `attributes[]`
		- 각 원소는 기본적으로 `attribute_info` 구조를 따른다.
```java
attribute_info {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```

| **Attribute**                        | **쓰이는 위치**    | **의미**                                                      |
| ------------------------------------ | ------------------ | ------------------------------------------------------------- |
| InnerClasses                         | 클래스             | 내부 클래스 및 인터페이스에 대한 정보                         |
| EnclosingMethod                      | 클래스             | 내부 클래스가 포함된 메서드에 대한 정보                       |
| Synthetic                            | 클래스,필드,메서드 | 컴파일러가 생성한 클래스를 나타내는 정보                      |
| Signature                            | 필드,메서드        | 제네릭 타입 정보가 포함된 메서드나 클래스의 시그니처          |
| SourceFile                           | 클래스             | 소스 파일 이름                                                |
| SourceDebugExtension                 | 클래스             | 디버깅 정보를 확장하는 데 사용되는 사용자 정의 정보           |
| Deprecated                           | 메서드, 필드       | 더 이상 사용되지 않는 클래스, 메서드 또는 필드를 나타냄       |
| RuntimeVisibleAnnotations            | 클래스,메서드,필드 | 런타임에 보이는 애노테이션                                    |
| RuntimeInvisibleAnnotations          | 클래스,메서드,필드 | 런타임에 보이지 않는 애노테이션                               |
| RuntimeVisibleParameterAnnotations   | 메서드             | 메서드 파라미터에 대한 런타임에 보이는 애노테이션             |
| RuntimeInvisibleParameterAnnotations | 메서드             | 메서드 파라미터에 대한 런타임에 보이지 않는 애노테이션        |
| AnnotationDefault                    | 메서드             | 애노테이션 타입 요소의 기본값                                 |
| BootstrapMethods                     | 클래스 파일        | 런타임에 동적으로 호출할 메서드에 대한 부트스트랩 메서드 정보 |
| Module                               | 클래스 파일        | 모듈 시스템 정보를 포함                                       |
| ModulePackages                       | 클래스 파일        | 모듈에 포함된 패키지들의 목록                                 |
| ModuleMainClass                      | 클래스 파일        | 모듈의 메인 클래스를 지정                                     |
| NestHost                             | 클래스 파일        | 현재 클래스의 호스트 클래스를 지정                            |
| NestMembers                          | 클래스 파일        | 현재 클래스의 네스트 멤버 클래스를 지정                       |
| Record                               | 클래스 파일        | 레코드 클래스에 대한 정보                                     |
| ConstantValue                        | 필드 테이블        | final 키워드로 정의된 상수값                                  |
| Code                                 | 메서드 테이블      | 자바 코드가 컴파일된 결과인 바이트 코드 명령어들              |
| StackMapTable                        | Code속성           | 스택 프레임의 변화를 나타내는 정보                            |
| Exceptions                           | 메서드 테이블      | 메서드가 던질 수 있는 예외 클래스들                           |
| LineNumberTable                      | Code속성           | 자바 소스 코드의 줄 번호와 바이트 코드 오프셋의 매핑          |
| LocalVariableTable                   | Code속성           | 메서드의 로컬 변수에 대한 디버그 정보                         |
| LocalVariableTypeTable               | 클래스             | 로컬 변수의 제네릭 타입 정보                                  |
| MethodParameters                     | 메서드 테이블      | 메서드의 매개변수에 대한 정보                                 |

%% TODO: 여기 보충 필요 %%

- `Code` 속성
- `Exceptions` 속성
	- JVM에서는 예외 테이블로 **핸들링 경로를 정의**한다.
- `LineNumberTable` 속성
- `LocalVariableTable, LocalVariableTypeTable` 속성
- `SourceFile, SourceDebugExtension` 속성
- `ConstantValue` 속성
- `InnerClasses` 속성
- `Deprecated, Synthetic` 속성
- `StackMapTable` 속성
- `Signature` 속성
- `BootstrapMethods` 속성
- `MethodParameters` 속성


### ByteViewer

`javap` 커맨드를 통해 바이트파일 구조를 확인할 수 있다.

다음 샘플 코드를 `javap -v -c` 옵션을 통해 컴파일하면, 클래스 구조를 확인할 수 있다.

```java
public class ByteCode {
    private static final String field = "string";

    public static String staticMethod() {
        return "staticMethod";
    }

}
```

![](/attatchments/20250101a77f9360.png)

## 바이트 코드 명령어 

Java 가상 머신의 명령어는 연산 코드(opcode)와 해당 작업에 필요한 피연산자(0개 이상)으로 이루어진다. 명령어 대부분이 피연산자 없이 연산 코드 하나로 구성되며, 피연산자는 피연산자 스택에 저장된다. 

바이트코드 명령어 집합은 연산 코드 길이가 1byte로 제한되기 때무에 최대 256개 연산코드만 표현이 가능하며, 클래스 파일 구조에서는 길이 정렬을 허용하지 않으므로 가상 머신이 런타임에 이 바이트들을 특정 구조로 재구성해야 한다.

자바 가상 머신의 명령어 집합에는 대다수 명령어 자체에 연산에 필요한 데이터 타입의 정보를 명시하고 있다. 대부분 명령어의 연산 코드 이름이 전용 데이터 타입을 뜻하는 문자로 시작한다.

`byte`, `char`, `short` 전용 명령어는 거의 없으며 `boolean` 타입 전용 명령어는 하나도 없다. 그래서 컴파일러는 컴파일타임이나 런타임에 `byte와` `short` 데이터는 `int` 타입으로 *부호 확장(sign extension)*한다.

- `load`, `store` 명령어
	- 스택 프레임의 지역 변수 테이블과 피연산자 스택 사이에서 데이터를 주고받는 데 쓰인다.
	- 데이터를 담는 역할의 피연산자 스택과 지역 변수 테이블은 주로 로드와 스토어 명령어로 조작한다. 
	- 객체의 필드나 배열의 원소에 접근할 때도 데이터를 피연산자 스택으로 전송한다.

- 산술 명령어
	- 피연산자 스택의 값 2개를 이용해 특정 산술 연산을 수행하고, 결과를 다시 피연산자 스택의 맨 위에 저장한다.
	- 데이터를 다루다 보면 오버플로가 발생할 수 있는데, JVM 명세에는 데이터 오버플로우 시 결과에 대한 규정을 명시하지 않고 있다. (런타임 예외 발생 X)
	- 나누는 값이 0이면 `ArithmeticException`을 던져야 한다.
	- 자바 가상 머신은 비정규화된 부동 소수점 수(denormalized floating-point number)와 점진적 언더플로(gradual underflow) 연산 규칙을 완벽하게 지원해야 한다. (IEEE 754)
		- 가까운 값으로 반올림
			- 모든 연산 결과를 적절한 정밀도로 반올림하고, 정확하지 않은 결과는 표현 가능한 가장 가까운 값으로 반올림한다. 
			- 표현 가능한 두 값이 수학적으로 정확한 값과 차이가 똑같다면 최하위 비트가 0인 값을 우선한다.
		- 부동 소수점 수를 정수로 변환 시 0에 가까운 값으로 반올림

- 형 변환 명령어
	- 숫자 타입 데이터를 다른 숫자 타입으로 변환한다.
	- 데이터 타입 확장시에는 명시적으로 표현하지 않더라도 문제가 없으나, 데이터 타입 축소 변환시에는 명령어를 반드시 명시해야 한다.
	- 축소 변환시 오버플로, 언더플로, 정밀도 손실이 발생할 수 있지만 이에 의해 런타임 예외가 발생하지 않도록 규정하고 있다.

- 객체 생성 및 접근 명령어
	- 생성된 객체, 배열 인스턴스의 필드 혹은 배열 원소는 객체 접근 명령어를 통해 얻어올 수 있다.

- 피연산자 스택 관리 명령어
	- 피연산자 스택을 직접 조작할 수 있다.

- 제어 전이 명령어
	- 프로그램 실행 흐름을 지정한 위치 명령어로 이동시킨다. 
	- PC 레지스터의 값을 조건부 / 무조건적으로 변경한다.

- 메서드 호출 및 반환 명령어
	- `invokevirtual`: 객체의 인스턴스 메서드를 호출하며, 객체의 실제 타입에 따라 디스패치(가상 메서드 디스패치)한다. 자바 언어에서 가장 많이 쓰이는 메서드 디스패치 방식이다.   
	- `invokeinterface`: 인터페이스 메서드를 호출하며, 런타임에 이 인터페이스 메서드를 구현한 객체를 검색하여 적절한 메서드를 찾는다.   
	- `invokespecial`: 인스턴스 초기화 메서드, private 메서드, 부모 클래스의 메서드를 포함하여 특수 처리가 필요한 일부 인스턴스 메서드를 호출한다.   
	- `invokestatic`: 클래스 메서드(`static` 메서드)를 호출한다.   
	- `invokedynamic`: 런타임에 호출 사이트 한정자가 참조하는 메서드를 동적으로 찾아 호출한다. 앞의 4개 호출 명령어의 디스패치 로직은 사용자가 변경할 수 없으나, 이 명령어의 디스패치 로직은 가상 머신 실행시 사용자가 설정 가능하다.

- 예외 처리 명령어
	- `throw`로 명시적 예외를 던지는 경우, `athrow` 명령어로 구현된다.
	- 명시적 예외 외에도 비정상적 상황을 만나면 예외를 던질 수 있다. 이전에는 예외 처리를  `jsr / ret`으로 구현했으나 이 방식은 더이상 사용되지 않으며, 예외 테이블로 **핸들링 경로를 정의**한다.

- 동기화 명령어
	- Java에서는 메서드 수준 동기화와 명령어 블록 동기화를 지원하며, 두 가지 방식 모두 구현에 모니터를 사용한다.
	- 메서드 수준 동기화는 바이트코드 명령어가 아니라 메서드 호출과 반환 명령어로 구현된다.
	- 명령어 블록 동기화는 일반적으로 `synchronized {}`로 표현되며, `moniterentor, monitorexit` 명령어로 구현된다.


---

출처

- [Oracle Java Virtual Machine Specification : Chapter 4. The `class` File Format](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.9)
- [Oracle Java Virtual Machine Specification : Chapter 6. The Java Virtual Machine Instruction Set](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html)
- [Wikipedia Java bytecode](https://en.wikipedia.org/wiki/Java_bytecode)
- [Wikipedia Java class file](https://en.wikipedia.org/wiki/Java_class_file)
- JVM 밑바닥까지 파헤치기 6장