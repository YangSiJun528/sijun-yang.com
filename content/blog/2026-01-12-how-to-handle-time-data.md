+++
title = "시간 데이터를 올바르게 다루는 방법"
date = 2026-01-12

[extra]
page_style = "post"
+++

프로그래밍을 어느 정도 해봤다면, 시간 관련 버그를 한 번쯤은 겪거나 들어보았을 것이다.
서버 환경을 바꾸니 시간이 9시간 밀린다거나, 로컬에서는 잘 되던 게 배포만 하면 시간이 이상하게 보인다거나.

이런 버그가 종종 보이는 이유는 단순한 문자열[^1]이나 정수형 타입과 다르게 시간 데이터의 다루는 데 생각보다 많은 주의사항이 존재하기 때문이다[^2].

이 글에서는 시간 데이터의 두 가지 타입과 관련 문제들을 살펴본 뒤, 내가 사용하는 두 가지 원칙을 소개한다.
마지막으로 이 원칙에 영향을 준 JavaScript Temporal API를 간단히 다룬다.

익숙한 Java/JavaScript 생태계를 중심으로 설명하지만, 모든 언어에 적용 가능하다.

## 시간의 2가지 타입

시간 데이터는 크게 두 종류로 나눌 수 있다.

**Exact Time(물리적 순간)**: 유일한 하나의 시점을 가리킨다.
"2024-01-01T00:00:00+00:00"는 전 세계 어디서든 같은 순간이다.
Unix timestamp나 UTC 오프셋을 포함한 ISO 8601 형식(예: 2024-01-01T00:00:00Z)이 여기에 해당한다.  
**Wall-clock Time(벽시계 시간)**: 특정 시점이 아니라 날짜와 시간 값 자체를 의미한다.
"12월 25일"은 뉴욕 시간이나 한국 시간처럼 특정 지역의 시간을 의미하지 않는다.

JavaScript의 Temporal API가 이 개념을 잘 설명해서, 이 글에서도 Temporal의 용어를 차용하여 설명한다.

## 시간 데이터를 다룰 때 주의해야 할 점

### 타임존 정보 손실

Java의 `LocalDateTime`은 `Local-`로 시작하는 이름에서 유추할 수 있듯이 타임존 정보를 저장하지 않는다.
하지만 `LocalDateTime.now()`를 호출하면 시스템의 기본 타임존을 사용해서 현재 시각을 가져온다[^3].
Python의 `datetime.now()`나 C#의 `DateTime.Now`도 비슷하게 절대 시점을 저장하지 않는다.

이러한 함수를 보았을 때 타임존 변환이 일어났는지 알기 어렵다. 또한 내부 구현에서 타임존 정보를 유지하지 않는다. 
객체 생성하는 도중 타임존을 사용해서 변환은 했지만, 결과 객체에선 어떤 타임존이었는지는 저장하지 않는 것이다.
그래서 이 값을 직렬화하거나 저장하면 어느 지역의 시간인지 알 수 없게 된다.
직렬화할 때 로컬의 시간대 정보를 함께 보내면 되지 않나 싶지만,
`now()` 호출 시 타임존을 직접 지정하는 것도 가능하기 때문에 선언 시점을 제외하면 어떤 타임존이었는지 알 방법이 없다.

특히 요즘의 서비스에선 아주 많은 "로컬"이 존재할 수 있다.
클라이언트와 서버만 있는 간단한 웹 서비스도 2개의 노드가 존재한다.
MSA 서비스라면 LB, DB, Redis, 모니터링 등 서버만 수십 개가 넘어갈 수도 있다.
이때 타임존 없는 시간 문자열을 주고받으면, 원래 어떤 타임존이었는지 복구할 방법이 없다.

### 여러 기술의 함정

다양한 기술 스택에서 타임존 처리와 관련된 함정들이 존재한다. 
직렬화 라이브러리, ORM, DB 등에서 타임존 관련 기본 설정이나 옵션이 직관적이지 않아 의도하지 않은 동작을 유발할 수 있다. 
몇가지 예시를 살펴보자.

#### Jackson

`DeserializationFeature.ADJUST_DATES_TO_CONTEXT_TIME_ZONE`[^4]은 기본적으로 true로 설정되어 있는데,
이 옵션이 활성화되어 있으면 `ZonedDateTime`이나 `OffsetDateTime`을 역직렬화할 때 원본 타임존을 컨텍스트 타임존(기본값 UTC)으로 변환한다.
이 과정에서 시점 정보는 유지되지만, 원래 타임존 정보를 잃어버린다 (이 옵션은 Jackson 3버전 부터 `DateTimeFeature` 객체로 이관되었다[^5]).

#### ANSI SQL의 TIMESTAMP WITH TIME ZONE

ANSI SQL 표준의 `TIMESTAMP WITH TIME ZONE` 데이터 타입은 이름과 달리 타임존 정보를 유지해야 한다는 요구사항이 없다[^6]. 
그래서 Oracle의 경우 타임존 정보를 저장하는 반면, PostgreSQL은 타임존 정보를 저장하지 않는 등 제각각의 구현이 존재한다.

PostgreSQL을 자세하게 설명해보자면,
데이터 타입 `timestamp`(`timestamp without time zone`)는 문자열로 Wall-clock Time 정보를 저장한다. 그래서 시간 비교 등의 연산에서 취약하다[^7].
Exact Time을 저장하려면 `timestamptz`(`timestamp with time zone`)를 사용해야 하는데, UTC로 변환되어 저장되고 타임존 정보는 유지되지 않는다.
또한 조회 시 세션의 타임존에 의존하여 시간 결과를 반환한다.

## 시간 데이터를 다루는 2가지 원칙

나는 시간 데이터를 다룰 때 두 가지 원칙을 기준으로 삼고 있다.

이걸 꼭 따라야 한다는 말은 아니다. 
하지만 "모든 노드(클라이언트, 서버, DB, 캐시 등)에서 시간 데이터에 대한 해석이 일관되어야 한다."는 조건을 만족하려면 이 원칙과 비슷한 결론에 도달할 것이다.

### 원칙 1: Exact Time과 Wall-clock Time을 명확히 구분하라

시간 데이터가 특정 시점을 나타내는지, 아니면 단순한 날짜/시간 값인지 먼저 판단해야 한다.

로그 타임스탬프, 결제 완료 시각, 이벤트 발생 시각처럼 "언제 일어났는가"가 중요한 데이터는 Exact Time으로 저장한다.
Exact Time이 아니면 정확한 시점을 알 수 없기 때문이다.
`LocalDateTime.now()`처럼 타임존 정보가 사라지는 방식으로 저장하면, 나중에 원래 시점을 복구할 방법이 없다.

생일, 기념일, 공휴일처럼 특정 시점이라는 개념 자체가 없는 데이터는 Wall-clock Time으로 저장해서 날짜/시간 값 자체만 저장한다.
이런 데이터를 Exact Time으로 다루면 오히려 문제가 생길 수 있다.

단, 이 기준은 도메인에 따라 달라진다.
산부인과 병원 시스템이라면 출생아의 생일이 출생 시점(Exact Time)으로 관리되어야 할 수 있다.

#### 저장 포맷

각 타입에 적합한 저장 포맷을 사용해야 한다.

Exact Time:
- Unix timestamp (밀리초 또는 초 단위 정수)
- PostgreSQL의 `TIMESTAMPTZ`
- ISO 8601 with offset (예: `2024-01-01T00:00:00Z`)

Wall-clock Time:
- `DATE` 타입 (예: `2024-12-25`)
- `TIME` 타입 (예: `14:30:00`)
- 문자열 (예: `"2024-12-25"`, `"14:30"`)

#### 타입 객체 선언 시점 주의사항

대부분의 시간 관련 버그는 코드에서 시간 타입을 생성할 때 발생하므로 주의 깊게 보아야 한다.

- Exact Time이 필요한 곳에서 `LocalDateTime.now()`를 쓰고 있지는 않은지 확인한다.
  `Instant.now()` 또는 `System.currentTimeMillis()`를 사용해야 한다.
- Wall-clock Time이 필요한 곳에서 UTC 변환을 하고 있지는 않은지 확인한다.
  `LocalDate.of(2024, 12, 25)`처럼 날짜 값 자체만 저장해야 한다.

### 원칙 2: Exact Time은 UTC로 저장하고, 타임존은 별도 관리하라

Exact Time 데이터는 저장, 직렬화/역직렬화, 송수신 등 모든 과정에서 UTC 또는 Unix timestamp로 처리한다.

#### 타임존이 필요한 경우

비행기 출발 시각, 회의 시간처럼 특정 지역의 시간 표현이 필요한 경우에는 Exact Time과 타임존을 별도 필드로 관리한다.

내부적으로는 UTC(Unix timestamp)로 저장하고, 사용자에게 보여줄 때만 해당 타임존으로 변환한다.

이렇게 하면 시간 비교나 집계 등의 계산이 편리해진다. 저장된 값 자체가 바뀌지 않기 때문이다.
앞서 말했듯 타임존 타입의 지원이 아직 완전하지 않거나 주의해야 하는 시스템이 존재하기 때문에[^8],
모든 곳에서 타임존을 UTC로 고정하거나 타임존 정보가 없는 Exact Time인 Unix timestamp 형태로 관리하는 것이 안전하다.

#### 전송/저장 시점 주의사항

DB 저장, API 직렬화 등의 과정에서 정보가 손실되는지 확인해야 한다.
명시적인 포맷이나 타입 계약이 없다면, 동일한 값이라도 시스템마다 다르게 해석될 수 있다.

DB 데이터 타입, gRPC 같은 스키마 기반 통신 처럼 포맷(타입) 계약이 존재하면 이를 그대로 따른다.   
JSON이나 텍스트 기반 통신처럼 계약이 없거나 범용 포맷이 필요한 경우, 
Unix timestamp 또는 UTC 오프셋을 포함한 ISO 8601을 사용한다.

ISO 8601는 가독성이 높고, 대부분의 직렬화 라이브러리에서 기본 지원된다.
Unix timestamp는 숫자 자체가 절대 시각이므로 해석 오류 여지가 없고, 저장 공간도 적다. 
다만 자동 변환을 지원하지 않는 환경에서는 추가 처리가 필요하다.

#### DST와 정책 변경 대응

타임존과 시간대 정보가 포함된 데이터를 다루는 경우, 여러 가지 모호한 상황(Ambiguity)이 발생할 수 있다.
이와 관련된 구체적인 예시는 ["Temporal Time Zones and Resolving Ambiguity - Temporal Proposal Documentation"](https://tc39.es/proposal-temporal/docs/timezone.html)에서 찾을 수 있다[^9].

- DST 시작/종료 같은 일시적 Offset 이동으로 인해서 동일한 벽시계 시간이 두 번 발생(fall back)하거나 존재하지 않는 시간(spring forward)이 생길 수 있다.
- 정책 변경으로 TimeZone 정의가 변경되어 기존에 저장된 미래 시점의 값과 새로운 규칙 간 충돌이 발생할 수 있다.

지금 설명중인 2가지 원칙들을 따르면 이러한 문제 대부분을 피할 수 있다.

- UTC로 저장하면 DST 영향을 받지 않는다. DST는 특정 타임존의 오프셋이 변경되는 것이지, UTC 자체가 바뀌는 것이 아니기 때문이다.
- 타임존 ID를 별도 필드로 저장하면 정책 변경 시에도 올바른 현지 시간을 계산할 수 있다. IANA 타임존 데이터베이스가 업데이트되면 새로운 규칙이 자동으로 적용된다.

## Temporal API 소개

### 역사적 배경

1996년 Java 1.0에서 `java.util.Date`가 도입됐다.
이는 많은 문제가 있는 구현이였는데, 월이 0부터 시작하고 연도는 1900을 빼서 저장하고 객체가 mutable해서 언제든 값이 바뀔 수 있었다[^10].

JavaScript가 만들어질 때, Java의 Date 구현을 거의 그대로 가져왔다[^11].
Java는 이후 개선을 시도한 반면, Date는 거의 30년이 지난 지금까지 사용되고 있다. (그래서 프로덕션 환경에선 별도의 시간 라이브러리를 많이 쓴다.)

2002년 Stephen Colebourne이 Java Date의 문제를 보완한 Joda-Time 라이브러리를 만들었다.
Joda-Time은 큰 성공을 거뒀고, 2014년 Java 8의 공식 `java.time` 패키지(JSR-310)로 이어졌다[^12].

JavaScript에서는 Date의 문제를 해결하기 위해 Temporal API 제안이 진행되었고,
TC39(ECMAScript 기술위원회)에서 이 제안은 Stage 3(명세가 확정되어 구현을 진행) 상태에 있다.

### Temporal의 설계

Temporal은 Exact Time과 Wall-clock Time의 구분을 타입 시스템에서 강제한다.

![Temporal Object Model](/blog/temporal-object-model.svg)
[원본 링크](https://tc39.es/proposal-temporal/docs/object-model.svg)

위 다이어그램에서 왼쪽은 Exact Time(물리적 순간)을 아는 타입들이고, 오른쪽은 Wall-clock Time(벽시계 시간)을 아는 타입들이다.

`Temporal.Instant`는 물리적 순간만 표현한다. 타임존이나 캘린더 없이 절대 시점만 저장한다.
`Temporal.ZonedDateTime`은 물리적 순간에 타임존과 캘린더를 더해서 양쪽에 걸쳐 있다.
반면 `Plain-` 접두사가 붙은 타입들(`PlainDateTime`, `PlainDate`, `PlainTime` 등)은 타임존 정보가 없다.

![Temporal Persistence Model](/blog/temporal-persistence-model.svg)
[원본 링크](https://tc39.es/proposal-temporal/docs/persistence-model.svg)

이 다이어그램은 각 타입이 문자열로 직렬화될 때 어떤 정보가 포함되는지 보여준다.
`Instant`는 UTC 시간만, `ZonedDateTime`은 오프셋과 타임존 ID까지, `Plain` 타입들은 날짜/시간 정보만 포함된다.

### Plain vs Local

여기서 "Plain"이라는 네이밍이 중요하다. java.time의 "Local"은 이름에서 위치를 암시하지만, "Plain"은 타임존에 대한 어떤 가정도 없다는 것을 명확히 보여준다.
단순하게 날짜와 시간 값 자체만을 표현한다는 의미가 더 직관적으로 드러난다. Temporal의 설계자들 역시 이러한 의도를 가지고 Plain이라는 이름을 선택했다[^13][^14].

### Temporal.Now의 분리

Temporal은 "now"를 다루는 방식이 기존의 일반적인 라이브러리와 다르다.
`Temporal.Now`라는 별도 객체가 존재해서, `Temporal.Now.instant()`나 `Temporal.Now.plainDateTimeISO()` 같은 형태로 사용한다.
현재 시각 조회를 별도 API로 분리해서 타임존 정보 손실이 발생할 수 있다는 것을 API 표면에서 명확하게 알려준다.
반면 java.time에서는 `LocalDateTime.now()`처럼 각 타입에 now 메서드가 붙어 있어 정보 손실이 발생 가능하다는 것이 명확하지 않다.

### 애매모호한 상황 처리

Temporal은 Plain 타입에서 Exact 타입으로 변환할 때 발생하는 모호함을 다루는 방법을 개발자가 선택할 수 있게 한다[^9].

앞서 제시한 원칙을 따르면 이런 기능이 필요한 상황 자체가 드물어진다.
다만 레거시 데이터 마이그레이션이나 사용자 입력 처리 같은 예외 상황에서는 유용하므로, 어떤 옵션을 제공하는지 알아두면 좋다.

## References

[^1]: 사실 문자열도 그렇게 단순하지는 않다. 유니코드, 인코딩, 정규화 등 고려해야 할 사항이 많다. 이와 관련해서는 [The Absolute Minimum Every Software Developer Must Know About Unicode in 2023](https://tonsky.me/blog/unicode/), [UTF-8 Everywhere](https://utf8everywhere.org/#), [아�니 이 글자 왜 들어간 거예요?](https://yozm.wishket.com/magazine/detail/2836/) 같은 글을 읽어보는 걸 추천한다.
[^2]: 여기서 다루는 내용과 별개지만, 시간 데이터의 정확성 자체도 완벽하지 않다. 분산 시스템에서 "정확한 시간"을 정의하는 것 자체가 매우 어려운 문제다. 이와 관련해서는 ["Designing Data-Intensive Applications"](https://dataintensive.net/), Chapter 8: 'Unreliable Clocks'를 읽어보는 것을 추천한다.
[^3]: OpenJDK, ["LocalDateTime.java source code"](https://github.com/openjdk/jdk/blob/jdk-27%2B3/src/java.base/share/classes/java/time/LocalDateTime.java#L213), GitHub.
[^4]: Jackson, ["DeserializationFeature.ADJUST_DATES_TO_CONTEXT_TIME_ZONE"](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/DeserializationFeature.html), JavaDoc.
[^5]: Jackson 3.0.3, ["DateTimeFeature"](https://javadoc.io/doc/tools.jackson.core/jackson-databind/latest/tools.jackson.databind/tools/jackson/databind/cfg/DateTimeFeature.html), JavaDoc.
[^6]: SQL-92 Standard, Section 4.5 "Datetimes and intervals" - [Modern SQL](https://modern-sql.com/standard), [Full Draft](https://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt).
[^7]: PostgreSQL Wiki, ["Don't Do This"](https://wiki.postgresql.org/wiki/Don't_Do_This#Don't_use_timestamp_(without_time_zone)_to_store_UTC_times).
[^8]: Hibernate ORM Discussion, ["Support timestamp with timezone/offset"](https://github.com/hibernate/hibernate-orm/discussions/4201#discussioncomment-1291666), GitHub.
[^9]: TC39, ["Temporal Time Zones and Resolving Ambiguity"](https://tc39.es/proposal-temporal/docs/timezone.html), Temporal Proposal Documentation.
[^10]: Oracle, ["Legacy Date-Time Code"](https://docs.oracle.com/javase/tutorial/datetime/iso/legacy.html), The Java Tutorials.
[^11]: Allen Wirfs-Brock, Brendan Eich, ["JavaScript: The First 20 Years"](https://dl.acm.org/doi/10.1145/3386327), Proceedings of the ACM on Programming Languages, Volume 4, June 2020 ([비공식 한국어 번역](https://js-history.vercel.app/)).
[^12]: Oracle, ["Java Date Time APIs"](https://docs.oracle.com/javase/8/docs/technotes/guides/datetime/index.html), Java Platform, Standard Edition 8; JSR 310 Expert Group, ["JSR 310: Date and Time API"](https://jcp.org/en/jsr/detail?id=310), Java Community Process.
[^13]: TC39 Temporal Proposal, ["What should be the long-term name of LocalDateTime?"](https://github.com/tc39/proposal-temporal/issues/707), GitHub Issue #707.
[^14]: 여러 시간 라이브러리에서 사용되는 `Local-` 네이밍은 C언어의 [`localtime()`](https://en.cppreference.com/w/c/chrono/localtime.html) 함수에서 유래했을 것으로 추측한다. 표준처럼 굳어진 용어를 바꾸자는 주장에 마냥 동의하지는 않지만, `Local-`의 의미가 명확하지 않은 것은 사실이라 이 경우에는 이름을 바꾸는 결정이 합리적이라고 본다.
