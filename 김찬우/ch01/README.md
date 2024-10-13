# Chapter 01: 자바 기술 시스템 소개

## 자바의 역사
```mermaid
timeline
    title Java 언어의 역사
    1991년: James Gosling이 Oak 언어 개발 시작
    1995
            : Oak 언어를 Java로 변경
            : JDK 1.0 공식 출시
    1997
            : JDK 1.1 출시 (Jar 파일 포맷, JDBC, 자바Beans, RMI 등 추가)
            : Inner Class, Reflection 추가
    1998
            : JDK 1.2 발표
            : 기술시스템을 J2SE, J2EE, J2ME 등으로 분리
            : EJB가 등장
            : JIT 컴파일러 탐재
    1999
            : HotSpot VM 출시
    2002: Java의 라이벌 Microsoft의 .NET 출시
    2004
            : JDK 5.0 출시
            : 제네릭, 열거형, 오토박싱 등 주요 기능 추가
    2006
            : JDK 6 출시
            : 오픈소스 전환 발표
    2009: Oracle이 Sun Microsystems 인수
    2014
            : JDK 8 출시
            : 람다 표현식, 스트림 API 도입
            : HotSpot VM의 영구 세대(PermGen) 완전 제거 (Metaspace로 변경됨)
    2018
            : JDK 11 출시
            : ZGC 등장 (실험 버전)
            : Graal VM 발표
    2019: JDK 12 출시
            : 셰넌도어 GC 등장
    2020: ZGC와 셰넌도어 GC가 정식 버전으로 변경
    2021: Java SE 17 출시 (LTS)
    2023: Java SE 21 출시 (LTS)
            : 가상 스레드 등장
```

### 주요하게 짚고 넘어갈 내용
- HotSpot VM과 JIT 컴파일러
- PermGen과 Metaspace
- ZGC와 셰넌도어

## Graal
### GraalVM
폴리그랏 VM(Polyglot VM)으로 다양한 언어(cross-language)를 지원하는 가상 머신
### Graal 컴파일러
GraalVM의 핵심 컴포넌트 중 하나로 HotSpot VM의 C2 컴파일러를 대체하는 JIT 컴파일러
