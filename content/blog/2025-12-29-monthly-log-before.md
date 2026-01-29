+++
title = "[월간 기록] 이전 자료 모음"
description = "월간 기록 시작 전 모아둔 자료들"
date = 2025-12-28

[extra]
page_style = "post"
series = "monthly-log"
+++

월간 기록을 시작했는데, 이전에 정리한 자료 중에서도 좋은게 많다. 

"월간 기록"이라고 볼 순 없지만 버리기는 아까워, 최근 몇 개월 동안 모아두었던 기록 중에 다시 찾아볼 만한 것들을 정리했다.

## 수집

{{ section_desc(desc="흥미롭거나 유용했던 자료들") }}

### [Computer Networks: A Systems Approach (무료 원서)](https://book.systemsapproach.org/index.html)

나는 한글 번역본을 책으로 가지고 있는데, 영어 원서는 무료로 공개되어 있다. 네트워크를 제대로 공부하고 싶을 때 참고하면 좋을 듯.

### [Interactive SICP JS](https://sourceacademy.org/sicpjs/index)

SICP의 설명을 Scheme 대신 기능이 제한된 JS로 대체한 에디션이다. 인터넷에서 무료로 볼 수 있고 실행 환경을 구축할 필요 없이 웹사이트에서 실습할 수 있다.

원문 또한 검색하면 공식적으로 무료로 풀려있고, 원문과 SICP in JS의 내용을 비교하고 싶다면 여기의 [비교판](https://sicp.sourceacademy.org/#)에서 볼 수 있다.

> SICP(Structure and Interpretation of Computer Programs, 컴퓨터 프로그램의 구조와 해석)는 [전 MIT 입문 프로그래밍 교재](https://www.wisdomandwonder.com/link/2110/why-mit-switched-from-scheme-to-python)로, Scheme(Lisp 방언)을 사용해 추상화, 재귀, 데이터 구조, 인터프리터/컴파일러 구현 등 컴퓨팅의 본질적 개념을 다룬다.

나는 특히 3장에서 substitution model에서 environment model로 전환하는 것과 4장에서 겉보기에는 동일한 언어라도 평가 전략이나 실행 모델과 같은 구현 선택에 따라 동작이 달라질 수 있다는 점이 인상 깊었다.

### Cloudflare 한국에서의 망 연동 문제

Cloudflare는 망 사용료로 인한 비용 문제로 CDN 사용 시 국외의 노드와 연결된다.
한국에 ICN(인천) 리전이 있지만, Enterprise 플랜을 사용해야만 고정이 가능하다.

CDN 뿐만 아니라 R2나 다른 기능들에서도 한국 리전이 사용되지 않는 듯 하나 확실하진 않다.

이에 대한 공식적인 내용을 찾아보기는 어렵지만, 사실 상 망 사용료로 인한 문제임이 확실한 것 같다.

- [클라우드플레어 임원 "한국 망 사용료가 비슷하다고? 20배 이상 비싸"](https://www.gameple.co.kr/news/articleView.html?idxno=208925)
- [관련 블로그 글](https://marinesnow34.github.io/2023/09/23/s3r2/)
- [나무위키](https://namu.wiki/w/Cloudflare#s-4), [가온위키(처음 보는 위키인데 내용이 많아서 링크)](https://www.gaonwiki.com/a/s5J)

### [SQL과 관계형 모델](https://kihyo-park.github.io/blog/rdb-intro/)

관계형 모델의 이론과 RDB를 연관지어 설명한다.

실제 DM 사용 경험이 적다곤 하지만, 이런 관점의 글은 본 적 없고 인상깊었다.

아마 블로그 글 저자가 컴파일러나 언어학에 관심이 있어서 이런 관점의 글이 나온거 같음.

### [템플릿 콜백 패턴과 스프링 트랜잭션 관리 방법](https://gist.github.com/YangSiJun528/84f6b453b6687491a2b1756498f5ed5f)

고등학교 후배의 포트폴리오 프로젝트 코드를 보다가 궁금해져서 찾아본 내용.
라이브러리 사용자가 Context를 알지 않아도 실행 가능하게 하는 편리한 패턴이라고 생각된다.

전역이 아닌 별도의 생명 주기를 가지는 Context는 어떤 식으로 관리되는지 생각해볼 수 있었다.

### [$5짜리 프롬프트로 $2,418짜리 취약점 찾은 썰](https://new-blog.ch4n3.kr/llm-found-security-issues-from-django-ko/) | [GeekNews](https://news.hada.io/topic?id=24828)

내용이 좋다. 실제로 LLM을 사용한 긍정적이고 효과적인 방법이라고 볼 수 있을 듯. 오픈소스 기여나 학습 측면에서 이런 시도가 나오고 있다는건 좋은 것 같다.

하지만 이와 반대로 LLM이 오픈소스 생태계 유지에 악영향을 줄 수 있을 것 같아 걱정이다.
FFmpeg, cURL, [Spring](https://www.linkedin.com/posts/snicoll_goodbye-albon-idrizi-httpslnkdin-activity-7405253562458378240-Jxu9/) 같은 여러 오픈소스에서도 AI Slop이 많아져 불만을 표하는 글을 보기도 했고.

### [SQLite 창시자가 제안한 "CRLF 그만 보내기" 선언 (그리고 철회)](https://fossil-scm.org/home/ext/stop-requiring-crlf.md) | [HackerNews](https://news.ycombinator.com/item?id=41830717)

이런 역사적 잔재는 생각보다 많다. tty, base64 등. 지금 당연하게 쓰는 관행이 사실은 수십 년 전 하드웨어 제약의 흔적인 경우가 많다.

SQLite 창시자 Richard Hipp이 "이제 CR은 그만 보내자"는 선언문을 작성했다가 철회했다. 주장이 과격했고, 호환성 문제가 있었기 떄문이다. 실제로 SQLite 관련 코드를 수정했다가 다른 서비스나 언어 구현에서 처리가 안 되어 깨지는 사례가 발생했다.

그럼에도 HN 토론에서 많이 언급됐듯이, 이런 레거시는 언젠가 정리해야 한다는 의견에 동의한다.

### Railway Oriented Programming (ROP)

[Railway Oriented Programming (ROP)](https://kciter.so/posts/railway-oriented-programming/)
[Railway Oriented Programming이란? (Medium)](https://0e.medium.com/그래서-rop가-뭔데-씹덕아-railway-oriented-programming-4e8070c04bda)

Rust의 `Result`, Kotlin의 `Result` 도 이 패턴을 사용한다. try-catch와 달리 예외 흐름을 선형적으로 처리할 수 있다.

Spring 트랜잭션 같은 기존 패턴과는 잘 맞지 않아서 실무에서 자주 쓰진 않았다.
실패 경로를 명시적으로 다뤄야 할 때 써본 적은 있지만, 돌이켜보면 크고 복잡한 코드의 레거시를 갈아엎지 못하던 상황에서 나온 코드였다.

ROP 자체는 좋다고 생각하지만, JVM 생태계는 프레임워크/라이브러리가 예외 기반이라, 제한된 영역이 아니라면 ROP 스타일이 잘 맞지 않아 보인다.

### 좋은 디자인이란? - 게슈탈트와 밀도로 알아보기

색이나 예쁜 걸 말하는 게 아니라 레이아웃이나 어떻게 정보를 전달할 것인가를 다루는 내용들.

- [UI Density (Matthew Ström)](https://matthewstrom.com/writing/ui-density)
- [HackerNews 토론](https://news.ycombinator.com/item?id=43925732)
- [게슈탈트 심리학과 UI](https://brunch.co.kr/@artneighbor/78)

레이아웃을 잘 꾸미고, 서비스 특성에 따라 밀도를 조절하기

### [Frameworkless Frontend Development 스터디](https://github.com/gdsc-ssu/2023-FE-with-no-framework)

프레임워크 없이 프론트엔드 개발하기. 내용은 괜찮은데 책으로 보면 불편할 듯. 블로그로 원서를 찾아보거나 직접 구현 분석하는 게 더 나을 것 같다.

[원본 코드 (GitHub)](https://github.com/Apress/frameworkless-front-end-development/tree/master)

(아래 작업 파트에 누가 스터디한 레포 포크해서 개인적으로 정리한 레포를 적어둠.)

### [최적화를 위한 알고리즘](https://algorithmsbook.com/optimization/#download)

- [GeekNews](https://news.hada.io/topic?id=24757)
- [Jupyter Notebook 버전](https://github.com/algorithmsbooks/algforopt-notebooks)

최적화 문제를 다루는 알고리즘 책. 무료로 공개되어 있고, Julia 언어로 구현되어 있다. ([Julia](https://namu.wiki/w/Julia) 나무위키)

다만 이런 의견도 있음을 주의하고 보기.

> 책에 Firefly, Cuckoo Search 같은 메타휴리스틱이 포함된 걸 보고 놀람. 이 알고리즘들은 학계에서 신뢰받지 못하고, ITOR 논문에서도 비판받았음

### [Top GitHub Users in South Korea](https://github.com/gayanvoice/top-github-users/blob/main/markdown/public_contributions/south_korea.md)

깃헙 유저들 나라 별 기여/팔로워 순 순위를 모아놓은 레포에서 한국인끼리 모아둔 페이지.

내리다 보면 아는 사람이나 유명한 블로그 쓰는 사람들이 종종 보인다.

내 계정도 있는데, 팔로워, 기여 수 조건이 있어 아슬아슬하게 들어가있다. 기여는 학교에서 안 공개 프로젝트 덕에 높게 나오는거 같다.

### [OOP의 35년 실수 - Casey Muratori 발표영상 In BSC 2025](https://www.youtube.com/watch?v=wo84LFzx5nI)

[Claude AI 요약](https://claude.ai/share/3f5044c9-e393-4f97-a177-f10879c5201c)

Casey Muratori가 OOP의 구조적 문제점을 지적한 강연. 논쟁적이지만 생각해볼 거리를 많이 던져준다.

ECS, Fat Struct 같은 패턴이 이전부터 있었고, 특정 상황에서는 이게 더 적절하다는 것.

현재의 OOP가 범용 대규모 프로그래밍에 유리하다는 인식과 달리 초기에는 제한된 도메인(분산 시스템)의 소규모 팀에서 탄생한 개념이라는게 인상깊었음.

### [선언적 프로그래밍에 대한 착각과 오해](https://evan-moon.github.io/2025/09/07/declarative-programming-misconceptions-and-essence/)

선언형 프로그래밍이 무엇인지, 왜 오해받는지에 대한 깊이 있는 분석.

### [What Every Programmer Should Know About Memory](https://lwn.net/Articles/250967/)

- [공식 PDF (114페이지)](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf)
- [구글 검색하기](https://www.google.com/search?q=ycombinator+%22What+Every+Programmer+Should+Know+About+Memory%22)

메모리 계층 구조, 캐시, NUMA 등 하드웨어 수준의 메모리 동작 원리를 다룬 유명한 글. 114페이지의 방대한 분량이라 모든 내용을 알아야 하는 건 아니라는 주장이 HN에서 꾸준히 나온다. (오래된 글이라 그런지 인용이 너무 많이 되서 해커뉴스 링크들를 주기보다는 그냥 구글에 검색하는게 나을 정도) 그럼에도 일부 핵심 개념은 성능 최적화를 할 때 중요하므로, 필요할 때 참고하면 좋을 듯.

### [Thread dump — the simple tool for debugging Java-applications in production | by Gustav Karlsson](https://blogg.bekk.no/thread-dump-the-simple-tool-for-debugging-java-applications-in-production-1cfed0d0d120)

프로덕션 환경에서 Java 애플리케이션의 성능 문제나 데드락을 진단하는 실용적인 방법. Thread dump를 어떻게 수집하고 분석하는지 단계별로 설명한다.

### [내가 만든 방어 유틸 함수 5종 세트 | 포프머신](https://blog.popekim.com/ko/2025/10/11/defensive-assertion-utils.html)

서버에서 유용한 커스텀 Assert 함수 패턴. 로깅, 슬랙 알림, 에러 로그, 심지어 사이렌 등으로 알리고, 중요도에 따라 다르게 구성한다.

코드로써 의미가 명확하고 가독성이 좋다. 테스트까진 아니지만 효율적이고 그에 준하는 효과 (가정이 깨지는 순간 파악, 의도적인 범위/제한 명시)를 가진다.

좋아보인다. 도입해볼만한 가치가 있을 듯.

### [Reflections on Trusting Trust - Ken Thompson](https://www.cesarsotovalero.net/blog/revisiting-ken-thompson-reflection-on-trusting-trust.html)

[한국어 AI 번역 (Gist)](https://gist.github.com/YangSiJun528/309fe1a139f818284558822277b8ca70)

Ken Thompson의 유명한 튜링상 수상 강연. 컴파일러에 백도어를 심어도 소스 코드 검사만으로는 발견할 수 없다는 걸 보여준 논문."신뢰를 어디까지 확장할 수 있는가?"라는 근본적인 보안 질문을 던진다.

### [The Surprising Truth About Pixels and Accessibility](https://www.joshwcomeau.com/css/surprising-truth-about-pixels-and-accessibility/)

px, em, rem 단위를 언제 어떻게 사용해야 하는지에 대한 실용적인 가이드. 특히 사용자의 접근성 관점에서 설명함.

- 대부분 상황: rem. 사용자 기본 폰트 크기 설정(=사용자가 의도한 크기)에 따라 크기가 바뀌기 떄문
- 부모/자신의 font-size에 비례해야 할 때: em
- 고정된 크기: px

### [You no longer need JavaScript](https://lyra.horse/blog/2025/08/you-dont-need-js/) | [HackerNews](https://news.ycombinator.com/item?id=12690842)

현대 CSS와 HTML만으로 대부분의 웹 기능을 구현할 수 있다는 점을 보여주는 글.

지금 이 글을 보여주는 개인 블로그를 만들 때 사용한 css 작성 스타일에도 영향을 주었다.

## 작업

{{ section_desc(desc="학습, 개발, 실험 등 직접 손댄 것들") }}

### 네트워크에서 MAC과 IP가 분리된 이유

"Rust in Action" p.345를 보고 궁금해져서 찾아본 내용.

IP와 MAC은 다른 용도를 가지고 서로 다른 단체에 의해 다른 연도에 만들어졌다:
- MAC: LAN 내부 하드웨어의 통신용
- IP: 네트워크 간 통신용, 먼저 만들어졌다.

두 가지를 연결하기 위해 ARP가 등장했다. 참고로 MAC과 IP가 항상 같이 쓰이는 건 아니다. (MAC 주소 개념을 사용하지 않거나, IP 주소 ↔ 링크 주소 매핑이 다른 방식으로 처리되는 경우도 있음)

- [Perplexity 검색 결과](https://www.perplexity.ai/search/daeum-naeyongeul-dwisbadcimhan-NuIVwuwVS666X3VvAJEX.A#0)
- [정리 Gist](https://gist.github.com/YangSiJun528/877cc0530c38d056ed988525a2392469)

### [Frameworkless Frontend Development 구현 연습](https://github.com/YangSiJun528/forked-2023-FE-with-no-framework)

[원본 스터디 레포](https://github.com/gdsc-ssu/2023-FE-with-no-framework)를 fork해서 직접 구현해본 프로젝트. 프레임워크 없이 순수 JavaScript로 프론트엔드 애플리케이션을 만들면서 라우팅, 상태 관리, 컴포넌트 시스템 등의 기본 원리를 학습했다.

프레임워크가 내부적으로 어떻게 동작하는지 이해하고, 실제로 직접 구현해보면서 웹 플랫폼의 기초를 다질 수 있었다.

### [CHIP-8 에뮬레이터 (C 언어)](https://github.com/YangSiJun528/c-chip-8)

C 언어로 CHIP-8 에뮬레이터를 구현한 프로젝트.

CPU 명령어 해석, 메모리 관리를 직접 만들면서 이해할 수 있었음.

### [awesome-for-me](https://github.com/YangSiJun528/awesome-for-me)

개인적으로 유용하다고 생각하는 자료들을 정리한 저장소.

지금은 개인 디스코드나 월간 기록으로 자료 관리 방식을 바꿔서 더 이상 업데이트하지는 않고 있음.
