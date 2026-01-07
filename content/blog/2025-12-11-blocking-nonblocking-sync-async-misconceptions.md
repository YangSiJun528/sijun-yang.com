+++
title = "동기/비동기, 블로킹/논블로킹 개념은 어디서부터 잘못되었나"
description = "Blocking + Async는 존재하지 않는다"
date = 2025-12-11

[extra]
page_style = "post"
+++

여러 개발 관련 글에서 Blocking/Non-blocking과 Sync/Async를 설명할 때면 항상 등장하는 2x2 매트릭스가 있다. 4개의 조합으로 나누어 설명하는 이 표를 한 번쯤은 보았을 것이다.

![Blocking/Non-blocking, Sync/Async 비교 매트릭스](/blog/sync-async-matrix.png)

이 매트릭스에서 가장 이해하기 어려운 부분은 Blocking + Async 조합이다. 많은 블로그에서 이 조합을 설명하려고 시도하지만, 나는 대부분 억지스러운 설명이라는 인상을 받았다.

최근 이 개념을 다시 공부하다가 이러한 설명은 올바르지 않다는 사실을 알게 되었다. 우리가 그동안 별 의심없이 받아들였던 이 설명이 잘못된 이유를 말해보고자 한다. 

## 잘못된 설명의 출처

이러한 매트릭스 기반의 설명의 시작은 IBM Developer의 2006년 글 "Boost application performance using asynchronous I/O"인 것으로 보인다.[^1]

![원래 글의 비교](/blog/ibm-io-comparison.png)

IBM의 글은 리눅스 커널의 AIO(Asynchronous I/O) API를 소개하면서 sync/async와 blocking/non-blocking을 조합해 4가지로 구분했다. 

그러나 이 글의 구분은 POSIX 표준 정의와는 다르다. 이 글에선 `select`/`poll`를 blocking + async의 조합으로 소개하지만, POSIX 표준 정의에는 이런 조합이 등장할 수 없다.  

## POSIX 표준의 실제 정의

POSIX 표준[^2][^3]과 리처드 스티븐스(W. Richard Stevens)의 책 Unix Network Programming[^4]을 살펴보면 명확한 정의를 찾을 수 있다.

- Synchronous I/O: I/O 작업이 완료될 때까지 요청 프로세스가 blocked
- Asynchronous I/O: 요청 프로세스가 blocked되지 않음
- Blocking: 요청한 동작이 완료될 때까지 함수 호출이 대기
- Non-blocking: 요청한 동작을 즉시 완료할 수 없으면 지연 없이 반환

여기서 중요한 점은 I/O 작업 완료의 의미다.

## Unix I/O 모델의 두 단계

I/O 작업은 일반적으로 두 단계로 이루어진다:

1. 데이터 준비 단계 - 디스크에서 데이터를 읽거나 네트워크에서 패킷이 도착하기를 기다린다. 데이터가 준비되면 커널 버퍼로 복사된다.
2. 데이터 복사 단계 - 커널 버퍼에서 애플리케이션 버퍼로 데이터를 복사한다.

스티븐스는 Unix에서 사용 가능한 5가지 I/O 모델을 다음과 같이 구분한다. 이 모델은 커널이 I/O 준비와 수행을 어떻게 관여하느냐에 따른 대표적인 분류이다. 상호 배타적인 조합표를 의미하지 않는다.[^5]

### 1. Blocking I/O

![Blocking I/O 모델의 Flow](/blog/blocking-io-model.png)

가장 흔한 I/O 모델이다. `read()` 시스템 콜을 호출하면 데이터가 준비되고 복사가 완료될 때까지 프로세스가 block된다.

### 2. Non-blocking I/O

![Non-blocking I/O 모델의 Flow](/blog/nonblocking-io-model.png)

`O_NONBLOCK` 플래그를 설정하면 데이터가 준비되지 않았을 때 `EAGAIN` 에러와 함께 즉시 반환한다. 하지만 데이터가 준비되면 복사하는 동안은 여전히 block된다.

### 3. I/O Multiplexing

![I/O Multiplexing 모델의 Flow](/blog/io-multiplexing-model.png)

`select()`나 `poll()`을 사용해 여러 file descriptor를 동시에 감시한다. 준비된 fd에 대해 `read()`를 호출할 때 여전히 block된다.

### 4. Signal-Driven I/O

![Signal-Driven I/O 모델의 Flow](/blog/signal-driven-io-model.png)

`SIGIO` 신호를 등록하고 데이터가 준비되면 신호를 받는다. 신호를 받은 후 `read()`를 호출하면 여전히 block된다.

### 5. Asynchronous I/O

![Asynchronous I/O 모델의 Flow](/blog/async-io-model.png)

진정한 비동기 I/O다. `aio_read()`[^7]는 즉시 반환되고, 커널이 백그라운드에서 두 단계를 모두 처리한 후 완료를 통지한다.

## Synchronous vs Asynchronous의 핵심

![5가지 I/O 모델의 비교표](/blog/io-models-comparison.png)

스티븐스는 Unix Network Programming에서 이 모델들의 동기/비동기 여부를 명확하게 구분한다.

> POSIX defines these two terms as follows:  
>  
> A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes.  
> An asynchronous I/O operation does not cause the requesting process to be blocked.  
> Using these definitions, the first four I/O models (blocking, nonblocking, I/O multiplexing, and signal-driven I/O) are all synchronous because the actual I/O operation (recvfrom) blocks the process. Only the asynchronous I/O model matches the asynchronous I/O definition.  

다음과 같이 번역할 수 있다.

> POSIX는 이 두 용어를 다음과 같이 정의한다:  
>  
> 동기 I/O(synchronous I/O) 작업은 해당 I/O 작업이 완료될 때까지 요청한 프로세스를 블로킹한다.  
> 비동기 I/O(asynchronous I/O) 작업은 요청한 프로세스를 블로킹하지 않는다.  
> 이 정의에 따르면, 처음 네 가지 I/O 모델(블로킹 I/O, 논블로킹 I/O, I/O 멀티플렉싱, 시그널 기반 I/O)은 모두 동기 I/O에 해당한다. 실제 I/O 연산인 `recvfrom` 호출이 프로세스를 블로킹하기 때문이다.
> 오직 비동기 I/O 모델만이 POSIX에서 정의한 비동기 I/O에 해당한다.

### 5가지 I/O 모델의 동기/비동기 분류

| I/O 모델 | Blocking/Non-blocking | Sync/Async | 1단계 Block | 2단계 Block |
|---------|---------------------|------------|------------|------------|
| Blocking I/O | Blocking | Synchronous | Yes | Yes |
| Non-blocking I/O | Non-blocking | Synchronous | No (폴링) | Yes |
| I/O Multiplexing | Blocking | Synchronous | Yes | Yes |
| Signal-driven I/O | Non-blocking | Synchronous | No (신호) | Yes |
| Asynchronous I/O | Non-blocking | Asynchronous | No | No |

앞의 네 가지 모델은 모두 데이터 복사 단계(2단계)에서 block된다. 따라서 POSIX 정의상 모두 synchronous다. 오직 asynchronous I/O만이 두 단계 모두에서 block되지 않는다.

### 왜 `select()`는 synchronous인가?

자료조사 과정에서 본 여러 자료에서 `select()`를 asynchronous로 설명하는 경우를 많이 보았다. 

`select()`는 여러 I/O를 동시에 처리할 수 있어서 비동기처럼 보이지만, 실제로는 다음과 같이 동작한다:
1. 프로세스가 감시할 파일 디스크립터 집합을 준비한다.
2. `select()`는 각 파일 디스크립터가 I/O를 수행해도 블로킹되지 않을 상태인지를 알려준다.
3. 준비된 파일 디스크립터에 대해 애플리케이션이 `read()`를 직접 호출한다. 

이때 `read()` 호출은 시스템 콜이 완료될 때까지 호출자를 반환하지 않으며, POSIX 정의상 요청한 프로세스를 블로킹하므로 synchronous가 된다.

멀티스레드 환경에서도 마찬가지다. I/O를 수행하는 스레드는 시스템 콜이 완료될 때까지 블로킹되며, POSIX 정의에 따르면 이는 동기 I/O다. 
스레드 분리는 애플리케이션 레벨의 동시성 전략일 뿐이다. 커널의 I/O 모델을 바꿀 수는 없다. 

Linux man page도 `select`를 "synchronous I/O multiplexing"으로 명시하고 있다.[^6] 반면 `io_uring`[^8]이나 POSIX `aio_`[^7] 함수들은 "Asynchronous I/O"로 구분한다.

## 커널 I/O 개념과 프로그래밍 모델의 혼동

IBM의 글은 OS 레벨 I/O를 다루는 기술 문서로, 리눅스 시스템 프로그래밍을 나름의 맥락 안에서 설명한다.

그러나 이 글이 널리 인용되면서 두 가지 문제가 발생했다.
첫째, IBM 글에서 사용한 구분 방식은 POSIX 표준의 정의와 정확히 일치하지 않았다. 그럼에도 불구하고 해당 글이 반복적으로 인용되면서, 표준과 다른 설명이 사실처럼 확산되었다.
둘째, 이후 수많은 블로그와 기술 문서들이 이 2×2 매트릭스를 인용하는 과정에서 원래의 맥락을 잃었다. OS 커널 레벨의 I/O 용어가 프로그래밍 모델이나 애플리케이션 아키텍처 설명에까지 무분별하게 적용되었다.

대표적인 오해는 Node.js + MySQL 드라이버의 예시다.

> “Node.js + MySQL은 Blocking + Async의 예시다. Node.js는 비동기인데 MySQL 드라이버가 블로킹이라서…”

이 설명은 서로 다른 레벨의 개념을 혼합한 것이다.

- Node.js의 비동기: 이벤트 루프 기반의 프로그래밍 모델
- MySQL 드라이버의 블로킹: 라이브러리 차원의 API 구현 방식

이는 커널 레벨의 I/O 동작과는 직접적인 관련이 없으며, 애초에 IBM 문서가 다루던 맥락과도 다르다.
이처럼 추상화 레벨이 다른 개념들을 구분 없이 동일한 용어로 설명하다 보니, 개념적 혼란이 발생할 수밖에 없다.

## 프로그래밍 모델로서의 비동기과 내부 구현

많은 비동기 프로그래밍 모델을 제공하는 프레임워크들이 내부 구현에선 동기 시스템 콜을 사용한다.

Netty는 "asynchronous event-driven" 프레임워크를 표방하지만[^10], 실제로는 `epoll()`이나 `kqueue` 같은 I/O multiplexing을 사용한다. 이들은 POSIX 정의상 synchronous다.  
Node.js도 마찬가지다. libuv를 통해 플랫폼별로 최적화된 I/O multiplexing을 사용하거나, 파일 시스템 작업의 경우 별도 스레드 풀에서 blocking I/O를 수행한다.[^9]  
이것이 잘못된 설명이나 구현은 아니다. 애플리케이션 개발자 입장에서는 비동기 프로그래밍 모델을 제공받는 것이 중요하지, 내부적으로 어떤 시스템 콜을 사용하는지는 중요하지 않다.

## 결론

POSIX 표준에 따르면 I/O 모델은 본질적으로 세 가지로 구분할 수 있다.

1. Blocking Synchronous - 전통적인 blocking I/O와 I/O multiplexing
2. Non-blocking Synchronous - non-blocking I/O와 signal-driven I/O
3. Asynchronous - 진정한 비동기 I/O (POSIX aio, io_uring)

따라서 Blocking + Async라는 조합은 정의상 존재할 수 없다. Asynchronous는 정의상 어떤 단계에서도 block되지 않기 때문이다.

물론 애플리케이션 레벨에서는 이 구분이 덜 엄격하다. 프로그래밍 모델로서의 비동기와 시스템 콜 레벨의 비동기는 다른 개념이며, 이를 명확히 구분해야 한다. 
Netty나 Node.js 같은 프레임워크가 비동기를 표방하는 것은 애플리케이션 개발자에게 제공하는 프로그래밍 모델을 지칭하는 것이지, POSIX I/O 모델의 정의를 따르는 것이 아니다.

중요한 것은 맥락이다. OS나 커널 레벨의 I/O를 논할 때와 애플리케이션 레벨의 프로그래밍 패턴을 논할 때, 같은 용어가 다른 의미를 가질 수 있다. 이러한 차이를 인지하고 명확히 구분해서 사용해야 불필요한 혼란을 피할 수 있다.

## 나는 어떻게 구분하는가

커널 I/O 레벨에선 표준의 정의를 따른다.
- 동기 / 비동기: I/O 작업이 완료될 때까지 요청 프로세스가 블로킹되는지 여부
- 블로킹 / 논블로킹: 요청한 동작을 즉시 완료할 수 없을 때 함수 호출이 대기하는지 여부
  
다만 시스템 레벨 개발을 주로 하지 않기 때문에, 이 용어를 쓸 일은 많지 않다. 
이 레벨에서는 추상적인 용어보다 select, epoll, io_uring 같은 구체적인 시스템 콜 이름으로 대화하는 것이 더 명확하다고 생각한다.

어플리케이션 레벨에선 다음 기준으로 구분한다.
- 동기 / 비동기: 애플리케이션 레벨에서의 프로그래밍 모델, 전체 실행 흐름
- 블로킹 / 논블로킹: 함수 호출이나 개별 작업 단위에서의 동작

일반적으로 이 관점에서 이야기한다. 이렇게 구분하면 아키텍처를 설명하거나 문제를 분석할 때 명확하게 생각할 수 있다.   
예를 들어, “비동기 모델 환경에서 블로킹 호출을 사용해 전체 실행 흐름에 영향을 주었다.”, “동기 환경이더라도 오래 걸리는 I/O를 논블로킹으로 처리해 효율을 높일 수 있다.” 와 같이 모델과 동작을 분리해서 생각할 수 있다.  

## 부록

"Boost application performance using asynchronous I/O" 를 포함한 IBM Developer의 오래된 글이 아카이브되어 원래 작성 시점을 알 수 없었는데, [저자의 사이트에서 원본 자료가 링크된 글](https://www.cyberciti.biz/tips/linux-boost-application-performance-using-asynchronous-io.html)을 보고 2006년 작성되었다고 추정했다.

IBM의 설명이 한국에만 퍼진 이야기는 아닌 듯 하다. 영어, 중국어나 일본어로 작성된 자료에서도 2x2 매트릭스를 사용해 구분하는 글을 찾아볼 수 있었다.

조사 과정에서 참고한 자료들이다. 높은 신뢰성을 가지는 문서는 아니지만 개념을 이해하는 데 도움이 되어 남겨두었다.  
- [Asynchronized I/O vs Multiplexing I/O 토론 - Linux Questions](https://www.linuxquestions.org/questions/programming-9/asynchronized-i-o-%3D%3D-multiplexing-i-o-467044/)
- [Comparing Two High-Performance I/O Design Patterns](https://www.artima.com/articles/comparing-two-high-performance-io-design-patterns)
- [비동기 프로그래밍, 비동기 I/O, 비동기 커뮤니케이션 - 쉬운코드 (YouTube)](https://youtu.be/EJNBLD3X2yg)
- [Blocking-NonBlocking-Synchronous-Asynchronous - 뒤태지존의 끄적거림](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/)
- [Tokio와 Compio의 차이: 비동기 모델(epoll vs io_uring) - Reddit](https://www.reddit.com/r/rust/comments/1pn6010/compio_instead_of_tokio_what_are_the_implications/)
- [기초 탄탄! 독하게 시작하는 Java Part 3(하) : 소켓과 파일 I/O 강의 | 널널한 개발자 - 인프런](https://www.inflearn.com/course/%EA%B8%B0%EC%B4%88%ED%83%84%ED%83%84-%EB%8F%85%ED%95%98%EA%B2%8C-java-part3-2)

## References

[^1]: M. Tim Jones, ["Boost application performance using asynchronous I/O"](https://developer.ibm.com/articles/l-async/), IBM Developer, 2006.
[^2]: IEEE Std 1003.1-2024, ["The Open Group Base Specifications Issue 8"](https://pubs.opengroup.org/onlinepubs/9799919799/), IEEE and The Open Group, 2024.
[^3]: IEEE Std 1003.1-2004, ["The Open Group Base Specifications Issue 6"](https://pubs.opengroup.org/onlinepubs/009695399/), IEEE and The Open Group, 2004.
[^4]: W. Richard Stevens, Bill Fenner, Andrew M. Rudoff, "Unix Network Programming, Volume 1: The Sockets Networking API", 3rd Edition, Addison-Wesley, 2003.
[^5]: 예를 들면, I/O Multiplexing과 Non-blocking I/O 모델을 함께 사용할 수도 있다. `select()` 함수는 기본적으로 이벤트가 올때까지 wait하는 blocking 함수인데, fb에 `O_NONBLOCK` 플래그를 활성화 해서 Non-blocking 함수로 동작하게 할 수 있다. 이 경우, 여러 fb를 동시에 감시하면서 데이터의 준비 여부와 무관하게 즉시 반환된다.
[^6]: Linux man pages, ["select(2) - synchronous I/O multiplexing"](https://man7.org/linux/man-pages/man2/select.2.html)
[^7]: Linux man pages, ["aio(7) - POSIX asynchronous I/O overview"](https://man7.org/linux/man-pages/man7/aio.7.html)
[^8]: Linux man pages, ["io_uring(7) - Asynchronous I/O facility"](https://man7.org/linux/man-pages/man7/io_uring.7.html)
[^9]: libuv documentation, ["Design overview"](https://docs.libuv.org/en/v1.x/design.html)
[^10]: Netty Project, ["Netty v4.2 README"](https://github.com/netty/netty/blob/4.2/README.md), GitHub
