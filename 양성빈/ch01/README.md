# 자바 기술 시스템 소개

## 들어가며

자바는 프로그래밍 언어뿐 아니라 여러가지 소프트웨어와 명세로 구성된 기술 시스템을 통칭한다. 자바 기술 시스템은 크로스 플랫폼 소프트웨어를 개발하고 배포하는데 모든 것을 제공하기 때문에 다양한 곳에서 아주 널리 쓰고 있다.

자바는 객체지향 프로그래밍 언어라는 점 외에도 다양한 장점들이 존재한다. 대표적인 특징들을 보자.

> 📚 특징
>
> 1. "한 번 작성하면 어디서든 실행이 가능하다."
> 2. 안전한 메모리 관리 시스템
> 3. 런타임에 핫코드를 감지, 컴파일하고 최적화
> 4. 다양하고 풍부한 표준 API 제공 및 다양한 기능의 서드파티 라이브러리를 활용 가능

> 📚 용어정리
>
> 핫 스팟: 데이터 테더링 용어이기도 하지만 자바에서는 빈번하게 실행되어 전체 성능에 영향을 크게 주는 코드를 말한다.

자바를 이용할때 해당 특징들을 이용해 개발하는 것도 중요하지만 자바 기술 시스템 안에서는 어떻게 활용되고 구현되는지 이해도 중요할것 같다.

## 자바 기술 시스템

> 🗒️ 자바 기술 시스템
>
> - 자바 프로그래밍 언어
> - (다양한 하드웨어 플랫폼용) 자바 가상 머신 구현
> - 클래스 파일 포맷
> - 자바 클래스 라이브러리 API(표준 API)
> - 다른 기업과 오픈 소스 커뮤니티에서 제공하는 서드 파티 클래스 라이브러리

> 🗒️ JDK란?
>
> 자바 프로그래밍 언어, 자바 가상 머신, 자바 클래스 라이브러리를 묶어서 JDK라 한다.

> 🗒️ JRE란?
>
> 자바 SE API와 자바 가상 머신 그리고 배포 기술까지를 묶어서 JRE라고 한다.

![자바 기술 시스템 구성 요소](./assets/01.png)

자바 기술 시스템을 핵심 비즈니스로 관점을 옮겨서 보면 다음과 같이 나눠진다.

> - 자바카드: 소형 기기 및 변조 방지 보안 칩등에서 실행되는 자바 플랫폼
> - 자바ME: 안드로이드 어플리케이션 개발때 자바언어로 사용하는데 이와는 다름. 모바일 기기에서 실행되는 자바 프로그램용 플랫폼
> - 자바 SE: 데스크톱 어플리케이션 용 자바 플랫폼
> - 자바EE: 다층 계층 구조로 이루어진 기업규모 애플리케이션용 자바 플랫폼

> 📚 참고자료
>
> https://docs.oracle.com/javase/7/docs/
