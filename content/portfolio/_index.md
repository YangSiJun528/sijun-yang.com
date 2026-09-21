+++
aliases = ["/ko/portfolio/"]
title = "포트폴리오"
description = ""
template = "portfolio.html"
+++

<header class="portfolio-header">
<div>
<h1 class="main-title">안녕하세요, 백엔드 엔지니어 양시준입니다.</h1>
</div>
</header>

<section class="portfolio-sheet" id="introduction" aria-label="소개">

Java와 Kotlin 기반의 Spring Boot를 주요 스택으로, AWS 환경에서 백엔드를 개발해왔습니다.<br>
요구사항 확인부터 설계와 구현, 운영 도구 개선과 장애 대응까지 서비스 운영과 밀접한 업무를 경험했습니다.

실무에서 AI로 구현 비용이 낮아지는 것을 경험하며, 기술적 판단과 검증이 더 중요하다고 느꼈습니다.<br>
크래프톤 정글에서 자료구조와 알고리즘을 학습하고 Pintos를 구현하며, 이를 뒷받침할 CS 기본기를 다졌습니다.

개발의 의미는 기술로 사람들에게 실질적인 가치를 제공하는 것이라고 생각합니다.  
회사에서 동료가 겪는 운영과 개발 과정의 불편을 찾아 개선했고,  
개인 프로젝트와 오픈소스에서도 실제 사용자가 있는 서비스를 만들고 개선해왔습니다.

AI 도구를 적극적으로 활용하며, 결과를 검증하고 코드 품질을 책임지는 것을 중요하게 생각합니다.  
정적 검사와 자동화 테스트를 바탕으로, 사람의 반복적인 개입을 줄이면서도 신뢰할 수 있는 결과를 만들어가고 있습니다.

<section class="portfolio-profile" id="profile" aria-labelledby="profile-heading">
<h2 id="profile-heading">프로필</h2>
<div class="portfolio-profile-content">
<img class="portfolio-profile-photo" src="/images/portfolio/sijun-yang-sky.png" alt="양시준 프로필 사진" width="1105" height="1423">
<ul class="portfolio-profile-links" aria-label="연락처와 프로필 링크">
<li>Email: <a href="mailto:yangsijun5528@gmail.com">yangsijun5528@gmail.com</a></li>
<li>GitHub: <a href="https://github.com/YangSiJun528" rel="me">@YangSiJun528</a></li>
<li>LinkedIn: <a href="https://www.linkedin.com/in/sijun-yang/" rel="me">sijun-yang</a></li>
<li>Hackers' Pub: <a href="https://hackers.pub/@sijun_yang" rel="me">@sijun_yang</a></li>
</ul>
</div>
</section>

</section>

<section class="portfolio-career" id="career" aria-label="경력">

<section class="portfolio-sheet portfolio-sheet--instagram" id="instagram" aria-label="IGOTIN - 경력 및 Instagram 업로드 운영의 개발자 의존도 해소">

## 경력

<header class="portfolio-project-header" id="igotin">
<img class="portfolio-logo" src="/images/portfolio/portfolio-igotin-logo.png" alt="IGOTIN 로고" width="225" height="225">
<div>

### IGOTIN - 본딧커뮤니티

[LinkedIn](https://www.linkedin.com/company/igotin/posts/?feedView=all) \| [서비스 소개](@/portfolio/igotin.md)<br>
백엔드 개발자<br>
<span class="portfolio-caption">2024. 07. - 2026. 02.</span><br>
Kotlin, Spring Boot, JPA, PostgreSQL, Redis Pub/Sub, AWS

</div>
</header>


IGOTIN은 북미 대학생을 위한 캠퍼스 기반 커뮤니티 서비스입니다.

<h4 class="portfolio-label">주요 기여</h4>

- 커뮤니티 가입과 승인, 채팅, 알림 등 사용자 기능의 백엔드 API 개발 및 개선
- 사용자 검색, 신고 처리, 운영 현황 조회 등 관리자 기능 개발 및 개선
- Instagram 게시 자동화, 가입 및 활동 통계 집계 등 외부 API 연동과 배치 작업 구현
- 운영 로그 기반 장애 원인 분석, API 응답 지연과 DB 쿼리 성능 개선

<section class="portfolio-case" aria-label="Instagram 업로드 운영의 개발자 의존도 해소">

#### Instagram 업로드 운영의 개발자 의존도 해소 {#instagram-upload}

<p class="portfolio-case-summary">개발자 지원이 자주 필요했던 업로드 기능의 개선을 문제 제기부터 설계, 백엔드 구현과 배포까지 주도하여,<br>
운영자가 개발자 없이 진행 상태와 결과를 확인하고 작업을 제어할 수 있도록 했습니다.</p>

<figure class="portfolio-diagram portfolio-case-figure">
<a href="/images/portfolio/portfolio-instagram-task-flow.png" target="_blank" rel="noopener" title="원본 크기로 보기">
<img src="/images/portfolio/portfolio-instagram-task-flow.png" alt="개선 이후 Instagram 업로드 작업 흐름: 요청 등록, 워커의 단계별 실행과 재시도, 결과 저장" width="4007" height="2523">
</a>
<figcaption>개선 이후 Instagram 업로드 작업 흐름</figcaption>
</figure>

<h5 class="portfolio-label">배경과 문제</h5>

IGOTIN은 학생이 작성한 자기소개를 학교별 [Instagram 계정](@/portfolio/igotin.md)에 자동 게시하는 기능을 제공했습니다.<br>
당시 운영 계정은 500개 이상이었고, 이 기능을 통해 많은 사용자가 유입됐습니다.

업로드 작업은 제한 확인, 스토리 업로드, 게시물 업로드, 내부 상태 갱신 등 여러 단계로 이루어졌습니다. 

기존에는 이 과정을 하나의 HTTP 요청에서 처리했고, 이로 인해 다음 문제가 존재했습니다.

1. 업로드가 성공하고 내부 작업은 실패로 남아있는 등의 상태 불일치가 발생했습니다.
2. 관리자 화면에는 작업의 최종 성공 여부만 표시돼 어느 단계에서 왜 실패했는지 알 수 없었습니다.   
운영자는 원인 파악과 복구를 매번 개발자에게 요청해야 했습니다.

<h5 class="portfolio-label">개선 과정</h5>

API는 작업 등록 후 즉시 응답하고, 실제 작업은 백그라운드에서 단계별로 수행하도록 분리했습니다.   
작업이 중단돼도 진행 상황을 확인하고 복구할 수 있도록, 각 단계의 진행 상태와 오류 원인을 기록했습니다.

소규모 팀의 운영 여건을 고려해, 필요한 작업 관리 기능을 갖추면서 추가 인프라 없이 운영하고자 하였습니다.  
DB를 사용하면서 작업 체이닝과 분기, 재시도 등 필요한 기능을 지원하는 [db-scheduler](https://github.com/kagkarlsson/db-scheduler)를 선택했습니다.

작업 중 발생한 오류의 특성에 따라 복구 방식을 나눴습니다.

- 자동 재시도: 일시적인 오류는 지수 백오프를 적용해 최대 3회 재시도합니다.
- 수동 재개: 조치가 필요한 경우에는 문제를 해결한 뒤 필요한 단계부터 재개할 수 있습니다.

또한 복구 과정에서 중복 게시가 발생하지 않도록, 기존에 누락된 Instagram API의 멱등 처리를 추가했습니다.

<h5 class="portfolio-label">결과</h5>

개발자에게 확인과 복구를 요청해야 했던 운영상의 불편을 해소했습니다.  
운영자가 관리자 화면에서 실패 위치와 원인을 확인하고, 작업을 제어할 수 있게 되었습니다.

[관리자 화면 전후 비교 예시](@/portfolio/igotin-admin.md)

</section>

</section>

<section class="portfolio-sheet portfolio-case" id="logging" aria-label="IGOTIN - 운영 API 병목 진단과 성능 개선 기반 구축">

#### 운영 API 병목 진단과 성능 개선 기반 구축

<p class="portfolio-case-summary">운영 로그를 바탕으로 병목 API와 지연 원인을 찾고, 개선 전후의 성능을 비교할 수 있는 환경을 구축했습니다.</p>

<h5 class="portfolio-label">배경과 문제</h5>

입사 당시에는 로그를 바탕으로 장애나 성능 문제를 분석하는 절차가 갖춰져 있지 않았습니다.

여러 요청의 로그가 섞여 있어 장애나 응답 지연의 원인을 추적하거나,   
API별 지연 시간과 에러율을 정량적으로 비교하기 어려웠습니다.

<h5 class="portfolio-label">개선 과정</h5>

기존 환경에서의 적용 가능성과 도입 및 운영 비용을 고려해 MDC와 AWS Athena를 선택했습니다.

MDC에 저장한 요청별 `requestId`로 같은 요청의 로그를 추적하고,  
S3에 적재한 ALB 액세스 로그는 Athena에서 API별로 묶어 분석할 수 있도록 구성했습니다.

API별, 기간별 p50/p95/p99 처리 시간과 에러율을 비교하는 Athena 쿼리를 작성해 재사용할 수 있도록 했습니다.

<h5 class="portfolio-label">결과</h5>

성능 지표를 기반으로 병목 API를 식별하고 요청별 로그로 원인을 분석할 수 있는 체계를 구축했습니다.  
수정 전후 지표 비교를 통해 개선 효과도 정량적으로 검증할 수 있게 되었습니다.

비교 쿼리는 팀에 공유하여 지속적으로 사용되었으며, 이를 활용해 다음 항목을 개선했습니다.

- 특정 채널 글쓰기 API: FCM 호출 비동기 전환, p95 응답시간 3,150ms → 495ms
- 뉴스피드 조회 API: 댓글 일괄 조회로 변경, p50 응답시간 2,151ms → 331ms
- 친구 추천 쿼리: `OR` 조인 분리 및 `UNION ALL` 적용, 쿼리 예상 시간 3,020ms → 61ms

</section>

<section class="portfolio-sheet portfolio-case portfolio-sheet--chat" id="chat" aria-label="IGOTIN - 채팅 기능 확장을 위한 독립 인프라 전환">

#### 채팅 기능 확장을 위한 독립 인프라 전환

<p class="portfolio-case-summary">채팅 데이터를 기존 PostgreSQL로 통합하고,<br>
채팅 로직과 실시간 이벤트 전달을 별도 서버로 분리해 구조를 개선했습니다.</p>

<figure class="portfolio-diagram portfolio-diagram--compact portfolio-case-figure">
<a href="/images/portfolio/portfolio-chat-message-flow.png" target="_blank" rel="noopener" title="원본 크기로 보기">
<img src="/images/portfolio/portfolio-chat-message-flow.png" alt="HTTP 메시지 요청을 API 서버와 PostgreSQL에서 처리하고 Redis Pub/Sub과 Live 서버를 거쳐 WebSocket으로 전달하는 채팅 구조. API 서버와 Live 서버의 배포 단위를 분리했다." width="2201" height="986">
</a>
<figcaption>개선 이후 채팅 메시지 처리와 이벤트 전달 흐름</figcaption>
</figure>

<h5 class="portfolio-label">배경과 문제</h5>

기존에는 채팅 데이터는 Firestore에, 나머지 데이터는 PostgreSQL에 나뉘어 관리되고 있었습니다.  
두 저장소를 함께 다뤄야 해 정합성 관리가 어렵고, 기능 추가나 변경 시 구현과 검증에 드는 부담이 컸습니다.

<h5 class="portfolio-label">개선 과정</h5>

먼저 Firestore의 채팅 데이터를 PostgreSQL로 옮기고, 메시지 저장과 조회 등 관련 로직을 API 서버로 통합했습니다.   
WebFlux 기반 환경에서 쉽게 적용할 수 있는 SSE를 선택해, 단일 서버에서 실시간 이벤트를 전달했습니다.   

하지만 단일 서버가 API 요청 처리와 SSE 연결 관리를 담당하며 배포마다 SSE 연결이 끊기는 문제가 반복됐습니다.   
이후 실시간 이벤트를 다른 기능에도 확장하려는 요구가 생기면서, 채팅 로직과 이벤트 전달 기능의 책임 경계가 모호해졌습니다.   

이에 채팅 로직과 실시간 이벤트 전달의 책임을 분리하기로 했습니다.

기존 채팅 HTTP API는 실행 환경만 옮겨 변경 범위를 줄이고, 실시간 이벤트 전달은 별도의 Live 서버로 분리했습니다.      
Live 서버의 통신 방식은 유지보수, 기능 확장성을 고려해 WebSocket을 선택했습니다.

Live 서버는 DB에 의존하지 않고 클라이언트의 WebSocket 연결과 채널 구독을 관리하며,   
API 서버가 Redis Pub/Sub으로 발행한 이벤트를 구독 중인 클라이언트에 fan-out 하도록 구성했습니다.

비용과 운영 복잡도를 고려해 Redis Pub/Sub을 사용하나,   
이벤트가 유실되어도 HTTP API로 DB에 저장된 상태를 다시 조회할 수 있도록 했습니다.

<h5 class="portfolio-label">결과</h5>

채팅과 제품 데이터를 한 DB에서 조회하고 처리할 수 있게 됐습니다.    
API 서버를 배포해도 Live 서버를 재시작하지 않고, 클라이언트의 WebSocket 연결을 유지할 수 있게 됐습니다.

공통 이벤트 전달 구조를 적용해, 향후 다른 기능에도 실시간 이벤트를 제공할 수 있는 기반을 마련했습니다.

</section>

</section>

<section class="portfolio-sheet" id="open-source" aria-label="오픈소스 기여">

## 오픈소스 기여

2023년부터 15개 이상의 오픈소스 프로젝트에 50회 이상 기여했습니다.<br>
전체 내역은 [GitHub](https://github.com/YangSiJun528/my-oss-contributions)에서 볼 수 있습니다.

### Spring Initializr <small class="portfolio-repository">(<a href="https://github.com/spring-io/initializr">github.com/spring-io/initializr</a>)</small>

Spring 프로젝트의 기본 구조와 의존성, 빌드 설정을 생성하는 프로젝트 생성 도구입니다.

- 프로젝트 설정 파일의 YAML 형식 지원 ([#1682](https://github.com/spring-io/initializr/pull/1682), [관련 글](https://bsky.app/profile/0.5ritter.de/post/3m4fon5ocu22n))
- Kotlin JPA 엔티티를 위한 All-Open 설정 지원 ([#1576](https://github.com/spring-io/initializr/pull/1576), [관련 글](https://blog.jetbrains.com/idea/2026/01/how-to-avoid-common-pitfalls-with-jpa-and-kotlin/#:~:text=When%20creating%20a%20new%20Spring%20project%20using%20the%20New%20Project%20wizard%20in%20IntelliJ%20IDEA%20or%20via%20start%2Espring%2Eio%2C%20both%20plugins%20are%20automatically%20configured%20for%20you%2E))
- 그 외 20회 이상 기여

### RustPython <small class="portfolio-repository">(<a href="https://github.com/RustPython/RustPython">github.com/RustPython/RustPython</a>)</small>

순수 Rust로 구현된 Python 인터프리터 프로젝트입니다.

- 2026 오픈소스 컨트리뷰션 아카데미의 멘티로 참여
- 기여 내역: [#8262](https://github.com/RustPython/RustPython/pull/8262), [#8537](https://github.com/RustPython/RustPython/pull/8537), [#8730](https://github.com/RustPython/RustPython/pull/8730) 등

</section>

<section class="portfolio-sheet" id="projects" aria-label="프로젝트">

## 프로젝트

<section class="portfolio-project portfolio-project--jungle" id="jungle-bell" aria-label="Jungle Bell">

<header class="portfolio-project-header">
<img class="portfolio-logo" src="/images/portfolio/portfolio-jungle-bell-logo.png" alt="Jungle Bell 로고" width="512" height="512">
<div>

### Jungle Bell

[GitHub](https://github.com/YangSiJun528/jungle-bell) \| [Website](https://jungle-bell.sijun-yang.com/)<br>
개인 프로젝트<br>
<span class="portfolio-caption">2026. 03. - 현재</span><br>
Tauri, Rust, React, TypeScript, Oracle Cloud

</div>
</header>

Jungle Bell은 크래프톤 정글에서 학습 시작/종료 확인을 놓치는 문제를 해결하기 위해 만든 데스크톱 앱입니다.

기획, 개발, 홍보, 운영을 맡아 40명 이상의 사용자를 확보하고, 운영 기간 평균 DAU 25명을 유지했습니다.

##### 설치 과정 개선

직접 앱을 추천하고 설치와 사용을 안내하면서 사용자가 막히는 지점을 찾아 개선했습니다.

- GitHub Releases에서 앱을 내려받는 방식이 익숙하지 않은 사용자를 위한 [설치 스크립트](https://github.com/YangSiJun528/jungle-bell#%EC%84%A4%EC%B9%98) 제공
- 설치 방법을 혼동하는 사용자를 위한 별도 [설치 가이드](https://jungle-bell.sijun-yang.com/#/install) 작성

##### 사용자 의견을 반영한 기능 확장

[GitHub Issues](https://github.com/YangSiJun528/jungle-bell/issues?q=-author%3AYangSiJun528%20-author%3Aapp%2Fdependabot)로 받은 의견을 반영해, 알림 앱을 캠퍼스 생활 정보를 제공하는 웹과 앱으로 확장했습니다.

##### AI를 활용한 개발과 검증

코드 탐색, 구현, 검토에 AI를 활용하면서, 역할별 에이전트와 반복 작업을 위한 스킬을 구성했습니다.

린트, 타입 검사, 의존성 규칙 검사를 포함한 코드 품질 관리 환경을 구성했습니다.   
이를 Git Hooks와 GitHub Actions에 연결해 품질 기준을 자동으로 검증하도록 했습니다.

</section>

<section class="portfolio-project" id="bracket-pair-guides" aria-label="Bracket Pair Guides">

<header class="portfolio-project-header">
<img class="portfolio-logo" src="/images/portfolio/portfolio-bracket-pair-guides-logo.svg" alt="Bracket Pair Guides 로고" width="40" height="40">
<div>

### Bracket Pair Guides

[GitHub](https://github.com/YangSiJun528/bracket-pair-guides) \| [JetBrains Marketplace](https://plugins.jetbrains.com/plugin/33518-bracket-pair-guides)<br>
개인 프로젝트<br>
<span class="portfolio-caption">2026. 08. - 현재</span><br>
Java, Kotlin, IntelliJ Platform SDK

</div>
</header>

Bracket Pair Guides는 JetBrains IDE 플러그인으로, 커서가 위치한 스코프에 가로/세로선 시각적 가이드를 제공합니다.

누적 다운로드 300회 이상, 평점 4.6점 이상을 기록했습니다.

##### 품질 검증을 통한 서비스 안정성 확보

UI 회귀 테스트를 추가하고 검사 결과와 캡처 이미지를 PR 댓글에서 확인할 수 있도록 했습니다. ([예시](https://github.com/YangSiJun528/bracket-pair-guides/pull/73#issuecomment-5740253751))

코드 변경에 따른 성능 저하를 확인할 수 있도록 성능 회귀 검사를 추가했습니다. ([예시](https://github.com/YangSiJun528/bracket-pair-guides/pull/74#issuecomment-5748679523))

</section>

<section class="portfolio-project" id="hello-gsm" aria-label="Hello, GSM">

<header class="portfolio-project-header">
<img class="portfolio-logo" src="/images/portfolio/portfolio-hello-gsm-logo.png" alt="Hello, GSM 로고" width="294" height="278">
<div>

### Hello, GSM

[GitHub](https://github.com/themoment-team/hellogsm-server) \| [Website](https://www.hellogsm.kr/) \| [팀 소개 페이지](https://themoment-landing.hellogsm.kr/)<br>
백엔드 개발자 / 총 6명 팀<br>
<span class="portfolio-caption">2022. 04. - 2023. 11.</span><br>
Java, Spring Boot, Spring Security, MySQL, Redis, AWS

</div>
</header>

Hello, GSM은 광주소프트웨어마이스터고등학교의 신입생 모집에 사용되는 입학지원 서비스입니다.<br>기존 시스템의 복잡한 과정과 불편한 UX를 개선하기 위해 2022년부터 교내 개발팀이 직접 개발/운영하고 있습니다.

백엔드 파트 리드로 초기 개발부터 참여해 Spring Boot 기반 서버 구조와 인증, 평가 배치 기능을 개발했습니다.  
이후 운영과 개발 절차를 문서화해 팀에 인계했으며, 현재까지도 의사결정에 기여하고 있습니다.

</section>

</section>

<section class="portfolio-sheet" id="credentials" aria-label="자격증 및 학력">

## 자격증

<ul class="resume-list">
<li>리눅스마스터 2급 <span class="portfolio-caption"><time datetime="2023-06">2023. 06.</time></span></li>
<li>SQL 전문가(SQLP) <span class="portfolio-caption"><time datetime="2022-12">2022. 12.</time></span></li>
<li>정보처리산업기사 <span class="portfolio-caption"><time datetime="2022-12">2022. 12.</time></span></li>
<li>데이터아키텍처 준전문가(DAsP) <span class="portfolio-caption"><time datetime="2022-07">2022. 07.</time></span></li>
<li>SQL 개발자(SQLD) <span class="portfolio-caption"><time datetime="2021-12">2021. 12.</time></span></li>
</ul>

## 학력 및 교육

<ul class="resume-list">
<li><h5>크래프톤 정글</h5>SW-AI Lab 12기 &amp; 심화과정 1기 <span class="portfolio-caption"><time datetime="2026-03">2026. 03.</time> - 현재</span></li>
<li><h5>광주소프트웨어마이스터고등학교</h5>소프트웨어개발과 5기 <span class="portfolio-caption"><time datetime="2021-03">2021. 03.</time> - <time datetime="2024-01">2024. 01.</time></span></li>
</ul>

</section>
