# 최적화 사례 분석 및 실전

## 사례 분석

일단 일반적인 상황에서 어플리케이션에 문제가 발생했을 땐 하드웨어나 JDK 버전을 업그레이드하면 바로 해결 되는 경우가 많다.  
그래도 해결이 안된다면 상황을 분석해야 한다.

### 대용량 메모리 기기 대상 배포 전략

원래 적은 메모리를 사용하고 운영체제도 32비트였는데 64비트, 16GB 이상의 메모리를 사용하는 환경으로 바뀌었다.  
일반적인 상황이라면 성능이 좋아져야 하는데 오히려 성능이 떨어지는 경우가 있다.  
예를 들어 패러렐 컬렉터를 사용하는데 이 컬렉터는 메모리가 커질수록 성능이 떨어질 수 있다.  
왜냐면 GC가 동작할 때 메모리를 순회하면서 객체를 찾아야 하는데 메모리가 커지면 순회하는 시간이 늘어나기 때문이다.  
그래서 이런 경우에는 GC가 발생하지 않는 것이 제일 중요하다.  
사용자가 많은 시간대에는 GC가 발생하지 않게 하고 사용자가 적은 시간대에 스케줄러를 돌려서 GC를 돌리는 방법을 사용할 수 있다.  
그리고 구세대 영역에 객체들을 너무 많이 생성하지 않도록 주의해야 한다.  
또한 64비트는 32비트보다 성능이 떨어지기 때문에 압축 포인터를 사용해서 메모리를 줄이는 방법도 있다.  
그리고 프로세스를 여러 개로 나눠서 사용하는 방법도 있다.  
예를 들자면 동일한 어플리케이션이지만 8080 포트를 사용하는 서버 하나, 8081 포트를 사용하는 서버 하나 이런 식으로 나눠서 사용하는 방법이다.  
근데 이 방법은 디스크에 동시 쓰기 문제가 발생할 수 있기 때문에 주의해야 한다.  
또한 커넥션 풀을 더 많이 사용하게 되기에 JNDI를 사용해서 중앙 처리를 해야할 수 도 있지만 많이 복잡해서 유지보수가 어려워질 수 있다.  
그래서 이런 경우에는 중앙화된 캐시를 사용하는 방법이 있다.

### 클러스터 간 동기화로 인한 메모리 오버플로

데이터베이스를 예시로 들면 읽기와 쓰기가 잦고 경합이 치열한 경우 성능이 매우 떨어질 수 있다.  
그래서 이때도 중앙화된 캐시를 사용하는 방법이 있다.  
근데 이런 경우도 캐시가 너무 많이 쌓이면 메모리가 오버플로우가 발생할 수 있다.

### 힙 메모리 부족으로 인한 오버플로 오류

예시를 먼저 들자면 2GB의 힙 메모리를 사용하는데 1.6GB의 메모리를 자바 힙에 할당하고 0.4는 다이렉트 메모리와 같은 메모리에 할당하고 있다.  
이때 다이렉트 메모리가 0.4GB를 넘어가면 힙 메모리가 부족하다는 오류가 발생할 수 있다.  
그런데 문제는 다이렉트 메모리도 GC가 발생하는데 힙 메모리가 가득 찼을 때 처럼 동적으로 GC에 알리지는 못한다.  
그래서 catch 문으로 잡아서 처리해야 한다.  
또 다른 예시는 socket 버퍼 영역인데 receive, send 버퍼는 각각 37KB, 25KB로 설정되어 있다.  
하나씩 보면 작은 용량이지만 여러 개의 소켓이 생성되면 메모리가 부족할 수 있다.  
JNI를 호출 할 때도 힙이 아닌 네이티브 메모리를 사용하는데 이 메모리도 가득찼을 때 동적으로 GC를 호출하지 못한다.  
VM과 GC도 별도의 메모리를 사용하기 떄문에 인지 해야한다.

### 서버 가상 머신 프로세스 비정상 종료

서버 가상 머신 프로세스가 비정상 종료되는 경우도 있다.  
예를 들어 `중개 서버`를 생각하면 된다.  
요청을 받으면 다른 서버로 요청을 보내고 응답을 받아서 다시 클라이언트에게 응답을 보내는 서버이다.  
그런데 요청은 많이 들어오고 응답 시간이 매우 길어지면 대기중인 스레드와 소켓 연결이 점점 많아져서 서버 가상 머신 프로세스가 비정상 종료될 수 있다.  
그래서 이런 경우에는 Publiser-Subscriber 패턴을 사용하면 해결 된다.

### 부적절한 데이터 구조로 인한 메모리 과소비

데이터 구조를 잘못 사용하면 메모리를 많이 사용할 수 있다.  
예를 들자면 HashMap<Long, Long>을 사용한다고 가정하겠다.  
Long 객체는 8바이트의 마크 워드, 8바이트에 클래스 포인터, 데이터를 담기 위한 8바이트의 long 값이 들어가 있다.(총 24바이트)  
또한 Long 객체는 Map.entry에 저장되고 entry는 16바이트의 객체 헤더, 8바이트에 next 필드, 4바이트의 해시 필드, 그리고 정렬용 패딩으로 구성되어 총 32바이트가 들어간다.  
마지막으로 HashMap에서 8바이트의 참조를 통해 entry를 참조한다.  
Long 객체 2개 48바이트 + entry 32바이트 + 참조 8바이트 = 88바이트가 들어간다.  
메모리 사용량 중 유효 데이터 비율은 16바이트 / 88바이트 = 18%이다.

### 안전 지점으로 인한 긴 일시 정지

안전 지점은 스레드가 안전하게 일시 정지할 수 있는 지점이다.  
그리고 예시를 또 들자면 GC는 0.14초가 걸렸지만 사용자 스레드가 일시 정지된 시간은 2.26초가 걸린 경우가 있다.  
이유는 사용자 스레드 모두가 안전 지점에 도달하지 못해서 GC가 일시 정지된 시간이 길어진 것이다.  
일부 스레드들이 안전 지점에 도달하지 못해서 GC도 같이 대기 상태에 놓여진 것이다.  
조금더 자세히 보면 int 타입, 혹은 int 보다 작은 타입을 루프 변수로 사용하면 순환문이 안전 지점으로 인식되지 않아 순환문이 끝날 때까지 GC가 일시 정지된다.  
이런 경우를 카운티드 루프라고 한다.  
반대로 long 타입을 사용하면 안전 지점으로 인식되어 GC가 일시 정지되지 않는다.  
이런 경우는 언카운티드 루프라고 한다.  
위에 상황은 반복문이 int 타입을 사용했기 때문에 안전 지점으로 인식되지 않아 스레드들이 함께 일시 정지된 것이다.