# Chapter 7. 클래스 로딩 시점

## 7.1 클래스 로딩 시점
### 생명주기
- 클래스 로딩
- 링킹 : 클래스 파일을 JVM이 사용할 수 있는 형식으로 변환
  - 검증 : 클래스 파일이 JVM 규칙에 맞는지 확인
  - 준비 : 클래스에 정의된 static 필드에 대해 기본값을 할당
  - 해석 : 클래스, 메서드, 필드 참조를 실제 메모리 주소로 변환
- 초기화 : 클래스 로더가 static 초기화 블록과 static 변수 초기화를 수행
- 사용 : 클래스가 메모리에 로드되고, 필요한 메서드와 필드가 사용됨
- 소멸 : 더 이상 참조되지 않는 클래스는 GC에 의해 메모리에서 제거
