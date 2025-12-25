+++
title = "Blocking/Non-blocking, Sync/Async 개념은 어디서부터 잘못됐을까"
description = "대부분의 블로그 글은 Blocking/Non-blocking과 Sync/Async를 잘못 설명하고 있다."
date = 2025-12-24

[extra]
page_style = "blog-post"
+++

## 개요

개발 블로그에서 Blocking/Non-blocking, Sync/Async 개념을 설명하는 글을 보면 이런 2x2 매트릭스를 많이 보았을 것이다.

![Blocking/Non-blocking, Sync/Async 비교 매트릭스](/blog/sync-async-matrix.png)


그리고 꼭 빠지지 않는 것이 Blocking + Async 조합에 대한 설명이다. 이 조합은 직관적이지 않아서 그런지 여러 억지스러운 비유가 붙곤 한다.

이 글에서는 다음과 같은 순서로 알아보겠다.

1. 잘못된 설명은 어디서 시작됐나
2. POSIX 표준의 실제 정의는 무엇인가
3. 블로그 설명이 잘못된 이유
4. Blocking/Non-blocking, Sync/Async의 차이를 어떻게 설명해야 할까

## 출처를 추적해보자

여러 블로그에서 나오는 방식의 구분은 IBM Developer의 ["Boost application performance using asynchronous IO"](https://developer.ibm.com/articles/l-async/)라는 글에서 시작되었다.

![원래 글의 비교](/blog/ibm-io-comparison.png)


이 글에서 제공하는 이미지를 보면, 블로그에서 보던 것과 비슷하게 생겼다. 또한, sync/async와 blocking/non-blocking의 기준도 블로그의 글과 비슷하다.

원문의 내용을 간단하게 요약해보자면 다음과 같다.

- 리눅스에서 I/O를 호출하는 함수는 여러가지가 있는데, `read()`, `select()` 등이 있다
- sync/async와 blocking/non-blocking을 비교하여 4개의 조합을 만들어서 설명한다
- AIO API는 async & non-blocking이라 효율적이라고 소개하며, 여러 예시 코드를 보여준다

그러나 이 글에는 큰 문제가 있는데, **POSIX의 sync/async와 blocking/non-blocking의 공식적인 정의와는 다르다는 것이다.** (Unix, Linux 또한 POSIX 계열의 OS이다.)

또한, 블로그 글에서는 이런 특정 OS에서의 I/O 용어로 쓰이는 의미를 다른 상황에서도 구분지으려고 하기 때문에 잘못된 설명으로 이해를 복잡하게 만든다.

## 공식 정의를 알아보자

> [NOTE]
> 이 글은 POSIX(Unix 계열 OS)를 기준으로 설명한다.

그럼 이제 공식적인 정의부터 알아보자.
IBM의 글이 Unix I/O를 설명하는 글이므로, Unix I/O의 블로킹/동기 관련 공식적인 정의를 찾아보겠다.

### 참고한 자료

다음 자료를 참고하였다. (이 파트는 넘겨도 이해하는데 지장이 없다. 원문이 궁금하지 않은 사람은 그냥 넘기면 된다.)

#### IEEE 1003.1-2024
IEEE에 등재된 공식적인 POSIX 스펙이다.
- 링크: https://pubs.opengroup.org/onlinepubs/9799919799/basedefs/V1_chap03.html

주요 정의
```
### 3.27 Asynchronous Input and Output
A functionality enhancement to allow an application process to queue data input and output commands with asynchronous notification of completion.

### 3.31 Asynchronous I/O Operation
An I/O operation that does not of itself cause the thread requesting the I/O to be blocked from further use of the processor.
This implies that the process and the I/O operation may be running concurrently.

### 3.48 Blocking
A property of an open file description that causes function calls associated with it to wait for the requested action to be performed before returning.

### 3.226 Non-Blocking
A property of an open file description that causes function calls involving it to return without delay when it is detected that the requested action associated with the function call cannot be completed without unknown delay.

### 3.371 Synchronous I/O Operation
An I/O operation that causes the thread requesting the I/O to be blocked from further use of the processor until that I/O operation completes.
```

#### Unix Network Programming - W. Richard Stevens
Unix Network Programming 라는 이름의 책.

이 책의 저자인 Stevens는 Unix 네트워크 관련한 유명한 책들을 집필하고, 여러 RFC 문서에도 참여하였다.

Chapter 6 "I/O Multiplexing: The `select` and `poll` Functions"의 "I/O Models" 섹션에서

```
## I/O Models

We first examine the basic differences in the five I/O models that are available to us under Unix:

- blocking I/O
- nonblocking I/O
- I/O multiplexing (select and poll)
- signal driven I/O (SIGIO)
- asynchronous I/O (the POSIX aio_ functions)

There are normally two distinct phases for an input operation:
1. Waiting for the data to be ready. This involves waiting for data to arrive on the network. When the packet arrives, it is copied into a buffer within the kernel.
2. Copying the data from the kernel to the process. This means copying the (ready) data from the kernel's buffer into our application buffer

### Synchronous I/O versus Asynchronous I/O

POSIX defines these two terms as follows:
- A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes.
- An asynchronous I/O operation does not cause the requesting process to be blocked.

Using these definitions, the first four I/O models (blocking, nonblocking, I/O multiplexing, and signal-driven I/O) are all synchronous because the actual I/O operation (recvfrom) blocks the process. Only the asynchronous I/O model matches the asynchronous I/O definition.
```

#### Linux man pages - man7
대표적인 리눅스 표준 참조 문서 집합이다.

`select`는 "synchronous I/O multiplexing"이라고 정의된다.
https://man7.org/linux/man-pages/man2/select.2.html
```
NAME
select, pselect, FD_CLR, FD_ISSET, FD_SET, FD_ZERO, fd_set - synchronous I/O multiplexing
```

그에 반해 비동기인 `io_uring`, `aio`는 "Asynchronous I/O"라고 명확하게 구분하고 있다.

https://man7.org/linux/man-pages/man7/io_uring.7.html
```
NAME
io_uring - Asynchronous I/O facility
```

https://man7.org/linux/man-pages/man7/aio.7.html
```
NAME
aio - POSIX asynchronous I/O overview
```

### 공식 정의를 요약해보자

위 자료들을 참고하면 Blocking/Non-blocking, Sync/Async 용어를 다음과 같이 정의할 수 있다.

- **Sync/Async**: I/O 작업 완료 시점까지 스레드가 block되는가?
    - block되면 Sync, block되지 않으면 Async이다.
- **Blocking/Non-blocking**: 함수 호출이 즉시 반환되는가?
    - 즉시 반환되면 Non-blocking, 반환되지 않으면 Blocking이다.

여기서 중요한 점은, **"I/O 작업 완료 시점"**이라는 것이 정확히 어떤 의미인지 알아야 한다. 이를 위해선 Unix I/O Model을 알아봐야 한다.

## Unix I/O에서 실제로 무슨 일이 일어나는가

Unix에서 사용 가능한 5가지 I/O 모델의 기본적인 차이점을 살펴보자.

1. blocking I/O
2. non-blocking I/O
3. I/O multiplexing (`select`와 `poll`)
4. signal driven I/O (`SIGIO`)
5. asynchronous I/O (POSIX `aio_` 함수들)

> [NOTE]
> Stevens의 원문은 네트워크 소켓을 기준으로 설명하지만, Unix의 I/O 모델은 모든 file descriptor (이하 fd)에 적용된다. 파일, 소켓, 파이프, 디바이스 등 모든 I/O는 fd를 통해 동일한 방식으로 작동한다. 따라서 여기서는 `read()`/`write()` 같은 일반적인 I/O 시스템 콜로 설명한다.

### 입력 작업의 두 단계

일반적으로 입력 작업에는 두 개의 구별되는 단계가 있다.

1. **데이터가 준비되기를 기다리는 단계** - 디스크에서 데이터를 읽거나, 네트워크에서 패킷이 도착하기를 기다린다. 데이터가 준비되면 커널 버퍼로 복사된다.
2. **커널에서 프로세스로 데이터를 복사하는 단계** - 준비된 데이터를 커널 버퍼에서 애플리케이션 버퍼로 복사한다.

### 1. Blocking I/O 모델

![Blocking I/O 모델의 Flow](/blog/blocking-io-model.png)


가장 널리 사용되는 I/O 모델이다. 기본적으로 모든 fd는 blocking 모드로 열린다.

작동 과정
1. 애플리케이션이 `read()` 시스템 콜을 호출
2. 커널이 데이터를 준비할 때까지 대기 (1단계에서 block)
3. 데이터가 준비되면 커널 버퍼에서 유저 버퍼로 복사 (2단계에서도 block)
4. 복사가 완료되면 `read()`가 반환

전체 과정 동안 애플리케이션은 blocked 상태

### 2. Non-blocking I/O 모델

![Non-blocking I/O 모델의 Flow](/blog/nonblocking-io-model.png)


fd에 `O_NONBLOCK` 플래그를 설정하면 non-blocking 모드가 된다.

작동 과정
1. 애플리케이션이 non-blocking으로 설정된 fd에 `read()`를 호출
2. 데이터가 준비되지 않았으면 `EAGAIN` 또는 `EWOULDBLOCK` 에러와 함께 즉시 반환
3. 애플리케이션은 계속해서 `read()`를 반복 호출 (폴링)
4. 데이터가 준비되면 커널 버퍼에서 유저 버퍼로 복사 (이 단계에서는 block)
5. 복사 완료 후 반환

문제점: 폴링으로 인한 CPU 시간 낭비

### 3. I/O Multiplexing 모델

![I/O Multiplexing 모델의 Flow](/blog/io-multiplexing-model.png)


`select()` 또는 `poll()`을 사용하여 여러 fd를 동시에 모니터링한다.

작동 과정
1. 애플리케이션이 `select()`를 호출, 관심있는 fd들을 등록
2. 하나 이상의 fd가 준비될 때까지 대기 (1단계에서 block)
3. 준비된 fd를 알려주며 `select()` 반환
4. 애플리케이션이 준비된 fd에 대해 `read()`를 호출
5. 커널에서 유저 버퍼로 데이터 복사 (2단계에서 block)
6. `read()` 반환

장점: 하나의 스레드로 여러 I/O를 처리 가능

### 4. Signal-Driven I/O 모델

![Signal-Driven I/O 모델의 Flow](/blog/signal-driven-io-model.png)


`SIGIO` 신호를 사용하여 I/O 준비 상태를 통지받는다.

작동 과정
1. `fcntl()`로 fd에 `O_ASYNC` 플래그 설정
2. `sigaction()`으로 `SIGIO` 신호 핸들러 등록
3. 시스템 콜은 즉시 반환, 애플리케이션은 다른 작업 수행
4. 데이터가 준비되면 커널이 `SIGIO` 신호 전송
5. 신호 핸들러에서 `read()`를 호출
6. 커널에서 유저 버퍼로 데이터 복사 (이 단계에서 block)
7. `read()` 반환

### 5. Asynchronous I/O 모델

![Asynchronous I/O 모델의 Flow](/blog/async-io-model.png)


진정한 비동기(true async) I/O. 커널이 모든 I/O 작업을 처리한다.

작동 과정
1. 애플리케이션이 `aio_read()`를 호출
2. 즉시 반환 (non-blocking)
3. 애플리케이션은 다른 작업 계속 수행
4. 커널이 백그라운드에서
    - 데이터 준비 (1단계)
    - 유저 버퍼로 복사 (2단계)
5. 모든 작업 완료 후 애플리케이션에 통지 (신호 또는 콜백)

특징: 애플리케이션은 어느 단계에서도 block되지 않음

### 5가지 I/O 모델의 비교

![5가지 I/O 모델의 비교표](/blog/io-models-comparison.png)

첫 네 가지 모델의 주요 차이점
- 첫 번째 단계(데이터 대기)의 처리 방식이 다름
- 두 번째 단계(데이터 복사)는 모두 동일 - `read()`에서 block됨

Asynchronous I/O만의 차이점
- 두 단계 모두 커널이 처리
- 애플리케이션은 전혀 block되지 않음

### Synchronous I/O versus Asynchronous I/O

POSIX 정의에 따른 구분
- **Synchronous I/O**: I/O 작업이 완료될 때까지 요청 프로세스가 blocked
- **Asynchronous I/O**: 요청 프로세스가 blocked되지 않음

이 정의를 적용하면
- **Synchronous**: blocking I/O, non-blocking I/O, I/O multiplexing, signal-driven I/O
    - 이유: 실제 I/O 작업(`read()`)에서 프로세스가 block됨
- **Asynchronous**: asynchronous I/O만 해당
    - 이유: 프로세스가 전혀 block되지 않음

> [NOTE]
> 다른 쓰레드에서 처리하면 되는거 아닌가 싶을 수 있는데, POSIX 정의에서는 어떤 스레드도 I/O를 위해 block되지 않아야 Asynchronous로 본다.

### I/O 모델 정리

| I/O 모델 | Blocking/Non-blocking | Sync/Async | 1단계 Block | 2단계 Block |
|---------|---------------------|------------|------------|------------|
| Blocking I/O | Blocking | Synchronous | Yes | Yes |
| Non-blocking I/O | Non-blocking | Synchronous | No (폴링) | Yes |
| I/O Multiplexing | Blocking | Synchronous | Yes | Yes |
| Signal-driven I/O | Non-blocking | Synchronous | No (신호) | Yes |
| Asynchronous I/O | Non-blocking | Asynchronous | No | No |

크게 보면 **Blocking Sync, Non-Blocking Sync, Asynchronous로 3가지 방법이 있을 뿐이다. Blocking+Asynchronous 같은 조합은 I/O 모델 기준으로는 말이 되지 않는다.**

### 왜 `select()`는 synchronous인가

IBM의 글에서는 `select()`를 asynchronous + blocking 조합으로 설명하지만, POSIX 정의상 synchronous다.

`select()`의 동작
1. 데이터 준비 여부만 알려줌
2. 실제 I/O는 애플리케이션이 `read()`를 호출해야 함
3. `read()`에서 데이터 복사 중 block됨 (여기서 block되기 때문에 synchronous다)

> [NOTE]
> 또한 `select()`이후 호출되는 `read()` 자체는 `O_NONBLOCK` 플래그에 따라 blocking/non-blocking 동작이 가능하므로 무조건 blocking이라고 볼 수도 없다.

## 여러 블로그 설명의 문제점

### 일반화의 문제

원래 자료의 출처인 IBM의 글에서 정의했던 **Blocking/Non-blocking, Sync/Async** 용어는 I/O Model에서의 차이를 구분하기 위해 정의하고 사용한 것이다. (심지어 정확하지도 않다!)

이 설명을 그 외의 영역에서도 일반화하고 구분하는 것이 문제다.

**일반적인 영역에서는 Blocking/Non-blocking, Sync/Async의 구분이 모호하고 동일한 것**으로 받아들여질 수 있으며, 그게 잘못된 용어 사용은 아니라고 생각한다.

#### 예시 1. Netty (Java)
자바에서 유명한 네트워크 라이브러리인 [Netty](https://github.com/netty/netty?tab=readme-ov-file#netty-project)는 환경에 따라 I/O Model 정의에 따르면 Sync & Non-blocking인 `epoll()` 호출을 사용하지만, README에서는 "asynchronous event-driven" 프레임워크라고 설명한다. 이는 애플리케이션 레벨에서의 비동기 프로그래밍 모델을 의미하는 것이지, POSIX I/O 모델의 정의를 따르는 것이 아니기 때문이다.

#### 예시 2. JavaScript의 Promise와 async/await
JavaScript에서 `async/await`는 비동기(Asynchronous) 프로그래밍을 위한 문법이지만, 브라우저나 Node.js의 실제 I/O 구현은 플랫폼마다 다르다.

그렇지만 대부분 (I/O 모델 관점에서는) Sync인 시스템 콜을 호출한다. 예시로 Node.js는 libuv를 통해 각 OS에 맞는 I/O multiplexing (Linux의 epoll, macOS의 kqueue 등) 또는 Blocking 호출을 처리하기 위한 별도의 Thread Pool에 위임하는 방식을 사용한다.

### 잘못된 비유

가장 대표적인 예시를 하나 가져와보았다.

> "Node.js + MySQL의 조합이 대표적인데, Node.js에서 비동기 방식으로 데이터베이스에 접근하기 때문에 Async이지만, MySQL 드라이버가 블로킹 방식으로 작동되기 때문에 Blocking + Async이다."

검색해보면 이 외에도 여러가지 비유를 찾아볼 수 있다.

이런 비유들은 대부분 잘못되었다고 생각하는데, 이유는 다음과 같다.

1. 이는 비동기 실행 환경에 동기 라이브러리를 사용한 잘못된 사용 예시일 뿐이다
2. Node.js와 MySQL 드라이버 레벨에 OS 레벨(I/O Model)의 정의를 적용할 필요가 없다
3. 이런 비교 상황에서 Blocking/Non-blocking, Sync/Async을 구분하는 공식적인 정의는 없다

따라서 Blocking와 Sync를 동등한 표현으로 볼 수 있고, Non-blocking와 Async또한 동등하게 볼 수 있다.

## 결론: Blocking/Non-blocking, Sync/Async의 차이는?

POSIX의 표준 정의를 참고하여 다음과 같이 정리할 수 있다.
- **커널이나 OS 레벨의 I/O 모델에서는** Blocking/Non-blocking, Sync/Async를 구분하여 5가지 모델이 있지만, 본질적으로 3가지 타입으로 구분할 수 있다.
  **Blocking Sync, Non-Blocking Sync, Asynchronous**
- **일반적인 상황에서는** 실행 흐름이 중단되지 않고 진행되는 논블로킹 방식이라면 비동기라고 부를 수 있다. 즉, 논블로킹과 비동기가 동일한 의미를 가진다. 이런 상황에선 I/O Model의 기준이 아니라 애플리케이션 레벨의 프로그래밍 모델을 지칭하는 것인데, 공식적인 정의가 없고 기준이 모호하기 때문이다.

중요한 것은 **맥락**이다. OS나 커널 레벨의 I/O를 논하는가, 아니면 애플리케이션 레벨의 프로그래밍 패턴을 논하는가에 따라 **용어의 의미가 달라질 수 있음**을 인지해야 한다.

## 출처

### 주요 참고 자료
- IEEE 1003.1-2024: Chapter 3. Definitions
    - https://pubs.opengroup.org/onlinepubs/9799919799/basedefs/V1_chap03.html
- Unix Network Programming - Stevens, Richard W./ Fenner, Bill/ Rudoff, Andrew
    - Chapter 6. I/O Multiplexing: The select and poll Functions, Section "I/O Models"
- Linux man pages (man7)
    - aio(7): https://man7.org/linux/man-pages/man7/aio.7.html
    - io_uring(7): https://man7.org/linux/man-pages/man7/io_uring.7.html
    - select(2): https://man7.org/linux/man-pages/man2/select.2.html

### 나머지 참고 자료
- Linux Questions - asynchronized I/O == multiplexing I/O?
    - https://www.linuxquestions.org/questions/programming-9/asynchronized-i-o-%3D%3D-multiplexing-i-o-467044/
- Comparing Two High-Performance I/O Design Patterns
    - https://www.artima.com/articles/comparing-two-high-performance-io-design-patterns
- [Youtube] BJ.20 비동기 프로그래밍, 비동기 I/O, 비동기 커뮤니케이션.. 각 맥락에 따라 비동기(asynchronous)의 의미를 설명합니다! - 쉬운코드
    - https://youtu.be/EJNBLD3X2yg
- Blocking-NonBlocking-Synchronous-Asynchronous - 뒤태지존의 끄적거림
    - https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/
- 어느 X(트위터) 글
    - https://x.com/finalchildmc/status/1654007292772376581
