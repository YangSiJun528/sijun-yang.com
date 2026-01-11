+++
title = "Java 배열 인덱스 접근은 정말 O(1)인가?"
description = "프로그래밍 언어의 명세와 구현은 다르다"
date = 2025-05-21

[extra]
page_style = "post"
+++

누군가 개발 커뮤니티에 이런 질문을 올렸다.

> Java 배열의 인덱스 접근이 O(1)이라는데, 공식 문서에서 찾을 수가 없어요. 어디서 확인할 수 있나요?

나도 당연히 O(1)이라고 생각했지만, 막상 "어디에 그렇게 쓰여 있냐"고 물으면 대답하기 어려웠다.

### Java 배열 인덱스 접근의 내부 동작

실제로 확인해보자. 다음과 같은 간단한 Java 코드가 있다.

```java
public static void array() {
    int[] intArr = {1, 2, 3, 4, 5, 6, 7, 8, 9, 0};
    int intA = intArr[0];
    int intB = intArr[9];
}
```

자바에서 배열이나 사칙연산 같은 기본 기능은 자바 코드 레벨에서 내부 동작을 확인할 수 없다.
`java.util.ArrayList` 같이 자바 언어로 구현된 표준 라이브러리와 달리, 언어 사양에 의해 동작이 정의되는 더 저수준의 영역이기 때문이다.
그렇다면 배열의 동작을 확인하기 위해선 무엇을 봐야 할까?

먼저 자바 언어 명세(JLS)의 배열 접근 문서[^1]를 확인했다. 배열을 어떻게 사용할 수 있는지만 서술하고 있다.
구체적인 구현 방식이나 시간복잡도에 관한 요구사항은 없다.

다음으로 JVM 명세(JVMS)를 확인했다.
JVMS에서는 배열 인덱스 접근 시 어떤 바이트코드가 사용되는지 명시한다. 실제로 사용되는지 확인해보자.

잠깐 바이트코드에 대해 설명하면, Java는 컴파일 시 바이트코드로 변환되고 JVM이 이를 실행한다.
JVM은 스택 기반 가상 머신으로[^2], 값을 스택에 쌓고 명령어가 꺼내 연산하는 방식이다.
따라서 자바 프로그램의 실제 동작을 더 확실하게 알기 위해선 바이트코드를 보아야 한다.

위 자바 코드를 컴파일하면 배열 인덱스 접근은 다음과 같은 바이트코드로 변환된다.[^3]

```
// intArr[0] 접근
ALOAD 0      // 배열 참조를 스택에 푸시
ICONST_0     // 정수 0을 스택에 푸시 (인덱스)
IALOAD       // 스택에서 배열과 인덱스를 꺼내 해당 요소를 읽음

// intArr[9] 접근
ALOAD 0      // 배열 참조를 스택에 푸시
BIPUSH 9     // 정수 9를 스택에 푸시 (인덱스)
IALOAD       // 스택에서 배열과 인덱스를 꺼내 해당 요소를 읽음
```

JVMS의 명령어 문서[^4]를 보면, `IALOAD` 명령어는 스택에서 배열 참조와 인덱스를 꺼내 해당 요소를 읽어 스택에 푸시한다고 설명한다.
어떤 스택 조작을 하는지만 서술되어 있다. 어떻게 접근해야 하는지, 시간복잡도가 어떠해야 하는지는 명시하지 않는다.

따라서 JLS와 JVMS 어디에도 "배열 인덱스 접근은 O(1)이어야 한다"는 요구사항은 없다. 이제 남은 건 실제 JVM 구현체를 확인하는 것이다.

OpenJDK를 선택했다. Java SE의 공식 레퍼런스 구현체이며, 대부분의 JVM 구현이 이를 기반으로 하기 때문이다.
OpenJDK는 C++로 구현되어 있다. `objArrayOop.hpp`와 `objArrayOop.inline.hpp`[^5][^6]를 보면 배열 인덱스 접근의 내부 동작을 알 수 있다.

```
다음과 같은 순서로 동작한다.
1. base_offset_in_bytes()  -> 배열 헤더 크기
2. sizeof(T) * index       -> 요소 크기 × 인덱스
3. base() + offset         -> 실제 메모리 주소 계산
4. oop_load_at(offset)     -> 해당 주소에서 값 로드
```

핵심은 (여타 CS 책에서 배열의 구현 방식을 설명할 때처럼) 포인터 산술 연산이다. 배열의 시작 주소에 오프셋을 더해 직접 메모리에 접근한다.
요소의 개수와 무관하게 상수 시간에 동작한다. 따라서 OpenJDK에서는 O(1)이 맞다.

그러나 Java 언어 명세(JLS)나 JVM 명세(JVMS) 어디에도 "배열 인덱스 접근은 O(1)이어야 한다"는 요구사항은 없었다.
OpenJDK는 O(1)로 동작하지만, 그건 특정 구현체의 선택일 뿐 언어의 보장이 아니다.
다른 JVM 구현체가 같은 방식을 사용한다는 보장은 명세에 없다.
(물론 현실적으로 다른 방식을 쓸 이유는 없다. 배열의 특성상 거의 모든 구현체가 O(1)일 것이다.)

### 명세와 구현의 구분

프로그래밍 언어는 크게 두 레벨로 나뉜다.

- 명세(Specification): 언어가 어떻게 동작해야 하는지 정의한 문서. 문법, 타입 시스템, 의미론 등을 규정한다.
- 구현(Implementation): 명세를 실제로 실행 가능하게 만든 컴파일러나 런타임. 명세가 정의하지 않은 부분은 구현체가 자유롭게 결정한다.

Java의 명세는 JLS(Java Language Specification), JVMS(Java Virtual Machine Specification)이다. 구현은 OpenJDK, Amazon Corretto, Azul Zulu 등이 존재한다. 다양한 JDK 종류의 차이를 알아보려면 [Which Version of JDK Should I Use?](https://whichjdk.com/)를 참고하자.[^7]

대부분의 성능과 관련된 부분은 명세가 아닌 구현이 결정한다. 흔히 알고리즘 문제에서의 시간 복잡도, 공간 복잡도와 같은 것들은 구현이 결정하게 된다.

프로그래밍 언어의 이러한 점은 Database의 질의형 언어(Query Language)와도 비슷한 점이라고 생각하는데, 내부 구현을 명세에 숨김으로써 자유롭게 기존 동작을 유지하면서 런타임의 최적화를 할 수 있기 때문이다.
예시로 Java는 GC 알고리즘이 계속 발전하고 있으며, Netflix는 Java의 버전을 8에서 17로 업그레이드하면서 약 20%의 CPU 사용률을 향상할 수 있었다.[^8] 또한 언어 명세의 수정 없이 Virtual Thread가 도입되기도 했다.

"(OpenJDK의) Java 배열 인덱스 접근이 O(1)이다" 라는 말에서 OpenJDK나 다른 구현체의 이름이 빠진다면 정확하지 않은 말이다.
하지만 이런 내용을 일일이 지적하는 건 소모적인 일이다.

대부분의 개발자가 표준이나 사실상(de facto) 표준인 구현체를 사용한다. Java 개발자가 OpenJDK 계열을 쓰고, 거의 모든 Python 개발자가 CPython을 쓴다.
따라서 일상적인 상황에서는 암묵적으로 가장 표준에 가까운 구현체를 의미한다고 이해해도 무방할 것이다.

그럼에도 다른 런타임이나 구현체 간 차이를 분석하거나 이해하기 위해선 이러한 사실을 알고 있어야 한다.

C는 이 구분이 특히 중요한 언어다. 다양한 하드웨어에서 동작해야 하고, 수십 개의 컴파일러 구현체가 존재한다.
C 표준은 Undefined Behavior (UB), Unspecified Behavior (UsB), Implementation-defined Behavior (IdB)를 구분한다.[^9]
이 때문에 GCC에서 잘 돌던 코드가 Clang에서 다르게 동작하거나, 최적화 옵션에 따라 결과가 달라질 수 있다.
(이처럼 C는 문법에서 많은 함정을 가지고 있는데, 관심이 있다면 [C/C++를 가르치기 전에 생각해 보기 - finalchild](https://proofby.ac/teaching-c/)를 한 번 읽어보는걸 추천한다.)

반면 C 이후에 설계된 현대 언어들은 명세가 훨씬 엄격하다. 그래도 여전히 이 구분이 필요한 순간들이 있다.
Node.js의 대항마로 Bun이 등장했을 때, "Bun이 Node.js와 뭐가 다른가?"라는 질문에 답하려면 명세와 구현의 구분이 필요하다.
JavaScript(ECMAScript)는 언어 명세고, Node.js와 Bun은 그 구현체다. 이벤트 루프나 파일 시스템 API는 JavaScript 명세가 아니라 런타임 구현이다. Bun이 빠른 이유는 JavaScript 언어가 달라서가 아니라, 런타임 구현이 다르기 때문이다.

Python도 비슷하다. PyPy는 CPython과 달리 JIT 컴파일을 제공해 CPU Bound 작업에서 유리하다.
심지어 JVM 위에서 실행되는 Jython이라는 런타임도 있다. 이러한 차이는 Python 명세가 실행 환경을 규정하지 않기 때문에 가능하다.
Java의 GraalVM Native Image를 사용하면 JVM 없이 네이티브 바이너리로 컴파일할 수 있다.[^10] 이 경우 JVM의 동작을 전제로 한 코드(리플렉션 등)가 제한될 수 있다는 점을 예측할 수 있고, 실제로도 그렇다.

또한 프로그래밍 언어의 구현체는 OS와 런타임 버전에 따라 동작이 달라질 수 있다.[^11] 보안 취약점 문서나 버그 리포트에서 환경 명시가 필수인 이유다.

이처럼 명세와 구현은 다르다. 프로그래밍 언어와 런타임의 성능/특징을 올바르게 논하기 위해서는 이를 이해하는 것이 필요하다.[^12]

## References


[^1]: Oracle, ["JLS 15.10.3 - Array Access Expressions"](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.10.3), Java Language Specification SE 25
[^2]: Oracle, ["JVMS 2.5 - Run-Time Data Areas"](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-2.html#jvms-2.5), Java Virtual Machine Specification SE 25
[^3]: 전체 코드는 [Gist에 올려두었다](https://gist.github.com/YangSiJun528/3c4210f0709e19ac72070c62a6b7333c).
[^4]: Oracle, ["JVMS 6.5 - Instructions"](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-6.html#jvms-6.5), Java Virtual Machine Specification SE 25
[^5]: OpenJDK, ["objArrayOop.hpp"](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/oops/objArrayOop.hpp) 및 ["objArrayOop.inline.hpp"](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/oops/objArrayOop.inline.hpp), GitHub
[^6]: 코드에서 `oop`는 Ordinary Object Pointer의 약자로, "일반적인 객체를 가리키는 포인터"라는 뜻이다. OpenJDK 코드에는 이 설명이 없어서 OpenJDK 개발자였던 Aleksey Shipilëv의 블로그에서 의미를 확인했다. Aleksey Shipilëv, ["JVM Anatomy Quarks #23: Compressed References"](https://shipilev.net/jvm/anatomy-quarks/23-compressed-references/#_compressed_references) 및 ["Java Objects Inside Out"](https://shipilev.net/jvm/objects-inside-out/)
[^7]: 다양한 JDK 종류의 차이를 알아보려면 [Which Version of JDK Should I Use?](https://whichjdk.com/)를 참고하자.
[^8]: InfoQ, ["Netflix Adopts Java 17"](https://www.infoq.com/presentations/netflix-java/), InfoQ Presentations
[^9]: Stack Overflow, ["Undefined, unspecified and implementation-defined behavior"](https://stackoverflow.com/questions/2397984/undefined-unspecified-and-implementation-defined-behavior)
[^10]: Oracle, ["GraalVM Native Image"](https://www.graalvm.org/latest/reference-manual/native-image/), GraalVM Documentation
[^11]: 같은 소스코드라도 컴파일러, OS, 런타임에 따라 다른 결과물이 만들어진다. 그리고 컴파일러와 OS 자체도 프로그래밍 언어로 작성된 프로그램이므로 환경마다 구현이 다를 수 있다.
[^12]: 개인적으로 이런 저수준 동작에 대한 호기심이 있었다. 이 과정에서 간단한 책을 보거나 토이 프로젝트를 진행했었다. 특히 [SICP: JavaScript Edition](https://sourceacademy.org/sicpjs/)를 읽으며, 겉보기에는 동일한 언어라도 평가 전략이나 실행 모델과 같은 구현 선택에 따라 동작이 달라질 수 있다는 점이 인상 깊었다. 이런 관점에서 보면 Java 배열 인덱스 접근의 시간복잡도 또한 구현에 의해 결정될 가능성이 크다고 보았고, 이런 실제하는 예시를 통해서 언어의 명세와 구현을 구분하는 시야에 대해서 공유하고 싶었다.
