JVM 문제해결도구를 통해 여러가지 문제에 쉽게 대응할 수 있다. '자바 가상 머신 명세'에서는 모니터링이나  문제 해결방법을 정의하고 있지 않으므로, 도구에 따라 지원하는 가상머신과 기능범위가 다르니 주의해야한다.

보통 도구에서는 다음 데이터들에 대해 다룰 수 있게 해준다.
- 예외 스택
- 가상 머신 운영 로그
- 가비지 컬렉터 로그
- 스레드 덤프 스냅숏(javacore 파일)
- 힙 덤프 스냅숏(hprof 파일)

이번 챕터에서는 아래에 대해 다룬다.
- JDK와 함께 배포되는 명령 줄 도구 여섯개
- GUI 문제 해결 도구 네개


### 기타 참고할만한 도구
- IBM J9, OpenJ9 VM
	- IBM
		- Support Assistant, Heap Analyzer, Thread and Monitor Dump An-alyzer, Pattern Modeling and Analysis Tool for Java Garbage Collector
	- ㅇㅣ클립스의 Memory Analyzer Tool
- HP-UX, SAP, 핫스팟 VM
	- HPjmeter, HPjtune


# 기본적인 문제 해결 도구

이곳에서 다루는 도구는 라이센스와 성숙 수준에 따라 세 범주로 나눌 수 있다.
- 상용 인증 도구
	- JMC와 JFR이 여기에 속함
	- JRockit에서 유래한 운영 및 관리 통합 도구인 JMC는 JDK7 업데이트 40부터 오라클 JDK에 통합됨
	- 개인 개발 목적으로는 무료, 상용 환경에서 이용하려면 비용을 지불해야 함
- 공식 지원 도구
	- 장기간 지원되는 도구들
	- 플랫폼이나 JDK 버전에 따라 작은 차이는 있을 수 있지만, 한 순간에 사라지지 않음 (사라지기전에 먼저 deprecated 됨)
- 실험적 도구
	- 소리 소문 없이 사라질 수도 있음

### 문제 해결 도구 구현
- 실제 동작하는 코드는 자바로 구현되어 JDK 도구 라이브러리에 담겨있음
- 리눅스 버전 JDK에서는 이 도구들 중 상당수가 쉘스크립트로 작성되어있음
- JDK6 부터는 JMX가 기본으로 켜져있다. (JDK5에서는 별도로 JMX 옵션 켜야함)





# 전체 도구 목록
---
오라클 스펙 문서에서, 제공되는 CLI 도구 목록을 확인할 수 있다. 이 중 대표적으로 jps, jstat, jinfo, jmap, jhat, jstack 등에 대해 아래에서 간단히 설명한다.
![image](https://github.com/user-attachments/assets/6cb6bc57-84cd-4a68-b892-889ff1478b22)

![[Pasted image 20241207232157.png]]
Link: https://docs.oracle.com/en/java/javase/21/docs/specs/man/


### jps: 가상 머신 프로세스 상태 도구
동작 중신 가상 머신 프로세스 목록을 보여주며 각 프로세스에서 가상 머신이 실행한 메인 클래스(main() 메서드를 포함하는 클래스)의 이름을 알려준다.

```sh
jps -ml
```
![image](https://github.com/user-attachments/assets/ea4f5fbe-4c88-400f-a9be-e1e5ec076bc4)
![image](https://github.com/user-attachments/assets/cf999c89-7777-4b71-a97a-8b39cb3d797b)

![[Pasted image 20241207225742.png]]
![[Pasted image 20241207225854.png]]


### jstat: 가상 머신 통계 정보 모니터링 도구
JVM 통계적 모니터링 도구(JVM stastics monitoring tool)인 jstat은 가상 머신의 다양한 작동 상태 정보를 모니터링하는 데 사용한다. 클래스 로딩, 메모리, 가비지 컬렉션, JIT컴파일과 같은 런타임 데이터를 보여 준다.

```java
jstat option vmid interval count
jstat -gc [자사vmid] 250 20  // 250ms간격으로 20번 물어본다.
```
![image](https://github.com/user-attachments/assets/24b5cac7-9f3b-4a3f-8cf8-9a6308d3b26d)
![image](https://github.com/user-attachments/assets/f5ba8d33-8fc2-412f-8b77-8a6645ab7e21)

![[Pasted image 20241207230446.png]]
![[Pasted image 20241207230118.png]]


### jinfo: 자바 설정 정보 도구
가상 머신의 다양한 매개 변수를 실시간으로 확인하고 변경하는 도구이다.

```sh
jinfo option vmid
jinfo -flag MaxHeapSize 62873
jinfo -flag ConcGCThreads 62873
```
![[Pasted image 20241207230700.png]]
![image](https://github.com/user-attachments/assets/3195d117-fdc0-4624-8c10-a455e6af7c8b)


### jmap: 자바 메모리 매핑 도구 
힙 스냅숏을 파일로 덤프해 주는 자바용 메모리 맵 명령어이다. 힙 스냅숏을 파일로 덤프함으로 충분한 파일 공간 및 메모리 사용량이 유지되어야 한다. 힙 스냅숏 덤프 외에도 jmap으로 F-큐, 자바 힙과 메서드 영역 상세 정보(사용량), 적용중인 컬렉터도 알아낼 수 있다.

`jmap`을 사용하지 않고도 자바 힙을 덤프하는 방법이 몇 가지 더 있기는 하다.

바로 `-XX;+HeapDumpOnOutOfMemoryError`옵션인데, 이를 사용하면 메모리가 오버플로될 떄 가상 머신이 자동으로 힙을 덤프해 준다.

리눅스에서는 kill -3 명령으로 가상머신에 프로세스 종료 시그널을 보내도 된다.
![image](https://github.com/user-attachments/assets/53e94d20-0457-49b4-ad9b-d04cc6e94128)

![[Pasted image 20241207231721.png]]
jmap에서는 동작을 안해서 jhsdb를 사용했다.


### jhat: 가상 머신 힙 덤프 스냅숏 분석 도구
jmap으로 덤프한 힙을 분석할 수 있는 도구, 하지만 이를 활용하는 사람은 많지 않다.
- 힙 스냅숏을 어플리케이션이 배포된 서버에서 직접 분석하는 일은 거의 없다.
- jhat의 분석 기능은 단순하다. (차라리 VisualVM 같은 다른 도구를 쓴다.)


### jstack: 자바 스택 추적 도구
스레드 스냅숏을 생성하는데 쓰인다. (스레드 덤프 또는 자바 코어 파일이라고 부른다.)

> jstack [options] vmid

<br/>

# GUI 도구

대표적으로 JConsole, JHSDB, VisualVM, JMC가 있다.


## JHSDB: 서비스 에이전트 기반 디버깅 도구
---
JDK는 JCMD와 JHSDB라는 두가지 다목적 통합 도구를 제공한다. 이들은 기본도구(CLI 명령)들의 기능들을 더욱 강력하게 제공한다. CLI 형태로도 사용이 가능하지만, GUI 또한 제공한다. 

JHSDB는 SA를 통해 프로세스 외부에서 디버깅할 수 있는 도구다. 

```ad-info
title: SA
핫스팟 가상머신이 제공하는 API 집합으로, 자바 가상 머신의 런타임 정보를 제공한다. 대부분 자바언어로 구현되었고 JNI 코드도 일부 사용한다. SA는 핫스팟의 내부 데이터 구조를 참고해 설계되었다. 이 구조들은 핫스팟 C++코드의 데이터를 자바 객체로 그대로 추상화한 것이다. SA의 API를 이용하면 다른 핫스팟의 내부 데이터를 독립된 자방 가상 머신 프로세스에서 분석할 수 있다. 또한 핫스팟이 런타임에 메모리를 어떻게 활용하는지 스냅숏의 덤프 파일로부터 상세하게 복원할 수 있다. SA의 동작 원리는 리눅스의 GCB, 윈도우의 Windbg와 비슷하다.

```

### 사용
맥에서는 안되나..?
![image](https://github.com/user-attachments/assets/bdf0a889-4a71-4585-ba9e-ca4c68db0050)

![[Pasted image 20241208005520.png]]
Link: https://stackoverflow.com/questions/52093130/mac-os-hsdb-hotspot-debugger-can-not-attach-to-the-process





## JConsole: 자바 모니터링 및 관리 콘솔
---
JMX에 기반한 GUI 모니터링 및 관리 도구, 주로 정보를 수집하고 JMX MBean을 통해 시스템의 매개 변수값을 동적으로 조정하는데 쓰임.

Memory 탭은 jstat 도구의 GUI 버전이며, 주요 용도는 가비지 컬렉터가 관리하는 가상머신메모리의 변화 추이 관찰이다. 자바 힙은 컬렉터가 직접관리하며 메서드 영역은 간접적으로 관리한다.

Threads 탭은 jstack의 GUI 버전이다. 스레드가 정지된 상황이라면 이 탭의 기능을 이용해 분석해 볼 수 있다.


```ad-info
title: JMX
가상 머신 자체는 물론 가상머신에서 구동되는 소프트웨어를 관리하는 용도다. 공개 기술이라서 모니터링과 관리 기능을 제공하는 미들웨어들은 대체로 JMX를 이용한다. 가상 머신이 JMX MBean을 다루는 프로토콜도 완전히 공개되어있다. 따라서 JMX 프로토콜을 지원하는 관리 콘솔의 API를 호출해도 되고, JMX 명세를 따르는 다른 소프트웨어를 이용해도 무방하다.
```
![image](https://github.com/user-attachments/assets/81f7586f-34c4-49cc-885b-9f1362ac4b69)
![image](https://github.com/user-attachments/assets/488ca2a0-db1c-4713-88b3-b14f96f49be7)
![image](https://github.com/user-attachments/assets/fb92ff70-3891-48a9-9bd6-d8cf8c5013df)
![image](https://github.com/user-attachments/assets/12652fbd-f636-4862-9e8a-8d7a50b19bb6)

![[Pasted image 20241208010535.png]]
![[Pasted image 20241208010810.png]]

![[Pasted image 20241208011634.png]]
![[Pasted image 20241208012125.png]]

```ad-info
title: -Xms -Xmx 매개변수의 역할
자바 힙의 크기를 제한하는 역할을 한다. 하지만 이 매개 변수들은 각 힙 세대의 크기를 명시하는 용도가 아니다. 이는 GUI에서 크기를 가늠해볼 수 있는데, 여기서 OldGen의 크기는  9,437kb이다.
```





## VisualVM: 다용도 문제 대응 도구
---
일반적인 운영 및 문제 대응 기능에 더해 성능 분석까지 제공하는 올인원 자바 문제 해결 도구
- 성능 분석 측면에서 JProfiler, YourKit 같은 상용 프로파일러에는 미치지 못함
- 하지만 VisualVM은 모니터링 대상에 특별한 에이전트 소프트웨어를 심지 않아도 되어 활용하기 쉽고, 애플리케이션 성능에 미치는 영향도 적음

### heapdump
![[Pasted image 20241208013317.png]]
![image](https://github.com/user-attachments/assets/61ad78d1-791a-48bb-86c3-9ccee0a3263d)

### 성능 분석 (profiler)
프로파일링은 런탕임 성능에 영향을 주는 편이므로 프로덕션 환경에서는 잘 쓰이지 않는 기능이다. (그래서 프로파일링 기능이 더 뛰어나고 애플리케이션에 주는 부담도 더 적은 JMC를 사용하기도 한다.)
![[Pasted image 20241208013606.png]]
![image](https://github.com/user-attachments/assets/611d6b17-dbd3-4625-b169-5e8a419a259b)

### BTrace
핫스팟 가상 머신의 인스트루먼트 기능(JVMTI 주요 구성 요소로, 이미 로딩되어 동작 중인 코드를 핫스팟 가상 머신이 런타임에 갱신할 수 있음)을 이용하여 원래 코드에 없던 디버깅 코드를 동적으로 삽입할 수 있다.
- 이 디버깅 코드는 대상 프로그램의 동작에 간섭하지 않아, 운영중인 프로그램에 매우 유용하다.
- 예를 들어 프로그램에 문제가 생겼는데, 필요한 정보를 로깅하지 않은 경우! 서비스 중단 없이 코드 추가가 가능하다.
![[Pasted image 20241208014025.png]]![image](https://github.com/user-attachments/assets/1e758769-e61a-40a7-81d8-c93356adce8a)

- 해당 기능은 자바 버전에 따라 정상적으로 수행되지 않을 수 있음





## JMC: 지속 가능한 온라인 모니터링 도구
---
마찬가지로 JMX를 활용한 모니터링 도구로 개인 용도로는 JMC, JFR을 무료로 사용할 수 있다. 

```ad-info
title: JFR
핫스팟 가상 머신에 내장된 모니터링 및 이벤트 기반 정보 수집 프레임워크
JProfiler 같은 다른 모니터링 도구와 비교해, 오라클은 '상시 구동' 기능을 특히 강조함
프로덕션환경에서 JFR이 처리량에 주는 영향은 대체로 1%를 넘지 않음
JFR 모니터링 프로세스는 애플리케이션 재시작없이 언제든 켜고 끌 수 있음
애플리케이션 소스 코드를 수정할 필요도 없고, 특정 에이전트를 함께 구동할 필요도 없음
```

```ad-info
title: JMC
원래 BEA 제품이기에, VisualVM과 달리 넷빈즈 플랫폼을 사용하지 않음
대신 IBM이 기증한 이클립스 RCP를 기본 프레임워크로 활용
JMC는 단독으로 실행하기보다는 이클립스 플러그인 형태로 쓰이는 모습이 더 흔함
JMC와 가상머신 통신에는 JMX 프로토콜이 사용됨
MBean이 제공하는 데이터를 보여주는 JMX 콘솔의 역할
JFR이 제공하는 데이터를 보여주는 JFR 분석 도구 역할
```

![[Pasted image 20241208014849.png]]
![image](https://github.com/user-attachments/assets/f7935f5b-98d8-49cb-9869-68e9ee4a1dfc)

다른 서버에서 구동중인 가상 머신을 모니터링하려면, 모니터링할 가상 머신 프로세스를 시작할  때 가상 머신 매개 변수로 명시해주어야한다. (port, ssl, 인증여부 등에 대한 정보)

<br/>

# 핫스팟 가상머신용 플러그인

IGV, CCV, 프로젝트 크리에이터, LogCompilation, HSDIS 등이 존재한다.
IGV, CCV는 11장에서 별도로 소개할 것이다. 


## HSDIS
---
오라클이 권장하는 핫스팟 가상 머신 JIT 컴파일 코드용 디스어셈블리 플러그인

JIT 컴파일러가 엄청난 양의 실행 코드를 동적으로 생성해 코드 캐시에 추가하는데, 이런 혼재된 환경에서 쉽게 디버깅할 간단한 방법은 없다. 그래서 문제를 우회하기 위해 HSDIS를 활용한다.
- -XX:+PrintAssembly:  (디어스어셈블된 코드 출력)
- -XComp: 컴파일모드로 실행된다. 코드는 JIT 컴파일이 바로 이루어진다.

디스어셈블리된 코드는 일반적으로 거대한 코드가 나오기 때문에 별도의 도구를 활용해서 해석해야한다.

![[Pasted image 20241208015742.png]]





## JITWatch
---
핫스팟 JIT 컴파일러용 로그 분석 및 시각화 도구로, HSDIS와 자주 함께 사용된다. 

![[Pasted image 20241208015841.png]]




















