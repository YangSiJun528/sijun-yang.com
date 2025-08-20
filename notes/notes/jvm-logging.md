# 자바 로깅
요즘에는 그냥 slf4j에 Logback, log4j2 구현체 중 하나를 쓴다고 보면 된다.

### slf4j
- Simple Logging Facade for Java
- 다양한 로깅 프레임워크에 대한 추상화 (인터페이스)
- 3가지 모듈
  - API: 인터페이스
  - Binding: 구현체와 연결, 컴파일 시점에 연결되며 1개의 구현체만 의존 가능
  - Bridge: 다양한 레거시 로깅 라이브러리의 API 호출을 가로채서 slf4j API가 호출되게 함
    - 여러 라이브러리를 쓰는 경우 여러 레거시 라이브러리가 있고, 변경이 불가능하므로 필요함

### Logback & log4j2
- Logback
  - slf4j 개발자가 Log4j의 후속 프로젝트로 Logback을 만듦
  - 현재 가장 많이 쓰임 (스프링도 기본으로 사용)
- log4j2
  - Apache에서 유지보수하고 있으며, Logback보다 더 빠르다고 함.

# 관련 자료
- [자바 로깅 프레임워크 히스토리 이해하기 | Blog](https://mark-kim.blog/history_of_java_logging/)
- [간단한 예시를 통해 이해하는 slf4j](https://mark-kim.blog/slf4j/)
- [간단한 예시를 통해 이해하는 Logback](https://mark-kim.blog/java_logback/)
