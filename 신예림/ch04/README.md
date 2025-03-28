# 가상 머신 성능 모니터링과 문제 해결 도구

성능 모니터링 및 문제 해결을 위한 도구들은 라이선스와 성숙 수준에 따라 세가지 범주로 나눌 수 있다.

이러한 도구들의 대부분이 래퍼 형태의 명령 줄 도구이기 때문에 용량 크기가 20KB를 넘지 않는다.

실제 코드는 자바로 구현되어 JDK 도구 라이브러리에 있다.

1. 상용 인증 도구
    - JMC, JFR
    - 개인 사용 무료. 상용 환경 이용시 유료
2. 공식 지원 도구
    - 플랫폼 / JDK 버전에 따른 차이 있음
3. 실험적 도구

이러한 도구들이 Java로 구현된 이유는, 어플리케이션이 프로덕션에 배포되면 서버에 직접 접근하거나 SSH 등으로 원격 접속에 제약이 생길수 있기 때문이다.

이런 경우에도 JDK 도구 라이브러리 인터페이스 및 구현 코드를 이용해 어플리케이션에서 도구의 기능을 직접 제공할 수 있다.

## 기본 도구

### JPS

유닉스 명령어 `ps`에서 유래되었고 기능도 유사하다.
동작 중인 가상 머신 프로세스 목록을 보여주며, 각 프로세스에서 VM이 실행한 메인 클래스 이름과 로컬 가상 머신 식별자(= LVMID)를 알 수 있다.

여러 가상 머신 프로세스가 동시에 시작해 프로세스 이름으로 구분이 불가한 경우, jps를 통해 메인 클래스를 확인해야 한다.

원격 VM의 프로세스 상태도 확인이 가능하다.

| 옵션 | 설명                                                               |
| ---- | ------------------------------------------------------------------ |
| -q   | LVMID만 출력(메인 클래스 이름 생략)  |
| -m   | 가상 머신 프로세스 시작 시 `main()` 메서드에 전달된 매개 변수 출력   |
| -l   | 메인 클래스의 완전한 이름 출력(프로세스가 JAR 패키지를 실행중이라면 JAR 경로 출력) |
| -v   | 가상 머신 프로세스 시작 시 주어진 가상 머신 매개 변수 출력         |

### JSTAT

- 로컬 또는 원격 가상 머신 프로세스의 클래스 로딩, 메모리, 가비지 컬렉션, JIT 컴파일과 같은 런타임 데이터를 보여 준다.
- 텍스트 콘솔 환경만 제공하는 서버에서 런타임에 가상 머신의 성능 문제를 찾는 보편적인 도구이다.
- 다양한 모니터링 옵션 사용 가능

| 옵션                | 설명                                                                 |
|---------------------|----------------------------------------------------------------------|
| -class             | 클래스 로딩/언로딩 횟수, 총 점유 공간, 클래스 로딩 시간 모니터링          |
| -gc                | 자바 힙 상태 모니터링. Eden, 생존자 공간, 구세대 등의 사용량, 총 가비지 컬렉션 시간 등 |
| -gccapacity        | 기본적으로 `-gc`와 같지만 자바 힙의 다양한 공간의 최대/최소 사용량에 초점을 둠 |
| -gcutil            | 기본적으로 `-gc`와 같지만 전체 공간 중 사용률(%)에 초점을 둠               |
| -gccause           | `-gcutil` 기능에 추가로, 최근 가비지 컬렉션의 일어난 이유 출력             |
| -gcnew             | 신세대 가비지 컬렉션 상태 모니터링                                       |
| -gcnewcapacity     | 기본적으로 `-gcnew`와 같지만 최대/최소 사용량에 초점을 둠                |
| -gcold             | 구세대 가비지 컬렉션 상태 모니터링                                      |
| -gcoldcapacity     | 기본적으로 `-gcold`와 같지만 최대/최소 사용량에 초점을 둠                |
| -compiler          | JIT 컴파일러가 컴파일한 메서드의 소요 시간 등의 정보 출력                  |
| -printcompilation  | 런타임에 컴파일된 메서드를 출력                                          |

다음은 사용 예시이다.

```bash
# 프로세스 2764의 가비지 컬렉션 상태를 250밀리초 간격으로 20번 질의

jstat -gc 2764 250 20
```

프로덕션 환경처럼 GUI 쓰기 힘들때 실무에서 많이 쓴다.

### JINFO

가상 머신의 다양한 매개 변수를 실시간으로 확인하고 변경하는 도구

- 가상 머신 시작 시 명시한 매개 변수 목록을 보고 싶을 때는 `-flag` 매개 변수를 사용한다.
- 명시하지 않은 매개 변수의 시스템 기본값을 알고 싶다면 `-flag` 옵션 다음에 원하는 매개 변수 이름을 적어 주면 된다.

### JMAP

힙 스냅숏을 파일로 덤프해 주는 자바용 메모리 맵 명령어이다.
힙 스냅숏 덤프 외에도 F-큐, 자바 힙과 메서드 영역 상세 정보도 알아낼 수 있다.

| 옵션               | 설명                                                                                     |
|--------------------|------------------------------------------------------------------------------------------|
| -dump             | 자바 힙의 스냅숏을 덤프한다. 형식은 `-dump:[live,]format=b,file=<filename>`이다.          |
|                    | 하위 매개 변수인 `live`를 추가하면 살아 있는 객체만 덤프한다.                              |
| -finalizerinfo     | F-큐 안의 객체들을 출력한다. F-큐에는 Finalizer 스레드가 `finalize()` 메서드를            |
|                    | 실행해 주길 기다리는 객체들이 담겨 있다.                                                 |
| -heap             | 사용 중인 컬렉터, 매개 변수 설정, 세대 상태 등 자바 힙 관련 상세 정보를 출력한다.          |
| -histo            | 클래스와 인스턴스 개수, 총 용량 등 힙에 있는 객체들의 통계 정보를 출력한다.                |

### JHAT

JDK 8까지 제공되던 JVM 힙 분석 도구다.

jhat은 작은 HTTP-웹 서버를 내장하고 있어서 분석이 완료되면 웹 브라우저로 결과를 살펴볼 수 있다.(http://localhost:7000) 하지만 실무에서 jhat을 직접 사용해 분석하는 사람은 많지 않은데 그 이유는 다음과 같다.

1. 힙 스냅숏 덤프를 애플리케이션이 배포된 서버에서 직접 분석하는 일은 드물다.
    - 보통은 덤프 파일을 다른 기기로 복사해 분석한다.
2. jhat의 분석 기능은 타 분석 도구에 비해 단순한 편이다.

### JSTACK

jstack은 현재 가상 머신의 스레드 스냅숏을 생성하는 데 쓰인다.

스레드가 장시간 멈춰 있을 때 원인을 찾기 위해 생성한다.(데드락, 무한루프, 외부 자원 요청 등...)

위와 같은 5가지 도구 외에도, 최신 JDK에서는 JCMD, JHSDB의 명령 줄 모드처럼 더 좋은 대체제가 제공된다. 용법은 비슷하다.

## GUI 도구

### JHSDB

JDK는 JCMD와 JHSDB라는 두 가지 다목적 통합 도구를 제공한다.

- SA를 통해 프로세스 외부에서 디버깅이 가능하다.
    - SA: 핫스팟 가상 머신이 제공하는 API 집합으로 자바 가상 머신의 런타임 정보를 제공한다.

### JConsole

JMX에 기반한 GUI 모니터링 및 관리 도구이다.

- 정보를 수집하고 JMX MBean(Managed Bean)을 통해 시스템의 매개 변숫값을 동적으로 조정하는 데 쓰인다.
    - JMX는 가상 머신 + 가상 머신에서 구동되는 소프트웨어를 관리하는 용도

### VisualVM

VisualVM은 일반적인 운영 및 문제 대응 기능에 더해 성능 분석까지 제공하는 올인원 자바 문제 해결 도구다.

### JMC

JMC, JFR은 개인 용도로 무료 이용 가능하다.

- JFR은 대상 프로그램 성능에 거의 영향이 없다.
- 타 도구나 MBeans에서 얻을 수 있는 데이터보다 훨씬 높은 수준의 데이터를 제공한다.

## 핫스팟 가상 머신 플러그인과 도구

### HSDIS

오라클이 권장하는 핫스팟 가상 머신 JIT 컴파일 코드용 디스어셈블리 플러그인이다.
OpenJDK에 소스가 포함되어 있어 직접 빌드할 수도 있고 빌드된 바이너리 파일을 가져다 써도 무방하다.

### JITWatch

핫스팟 JIT 컴파일러용 로그 분석 및 시각화 도구이다.