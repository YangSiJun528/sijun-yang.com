+++
title = "이력서"
description = ""
template = "resume.html"

[extra]
name = "양시준"
role = "주니어 백엔드 엔지니어"
contact_label = "연락처"
email_label = "이메일"
phone_label = "전화번호"
github_label = "GitHub"
photo_alt = "양시준 프로필 사진"
+++

## 핵심 역량

- Java/Kotlin, Spring Boot, PostgreSQL을 활용한 백엔드 서비스 개발 및 AWS 클라우드 인프라 운영
- Codex와 Hermes를 활용한 AI 하네스 및 루프 엔지니어링
- 로그와 지표를 통한 성능 분석 및 운영 문제 해결
- SQLP와 실무 경험을 바탕으로 한 DB 설계 및 쿼리 튜닝

## 경력

<div class="resume-entry">
<div class="resume-row resume-row--stack">

### 본딧커뮤니티

<p class="resume-meta">서울 강남</p>
<p class="resume-main">Backend Developer</p>
<p class="resume-meta"><time datetime="2024-07">2024. 07.</time> - <time datetime="2026-02">2026. 02.</time></p>

</div>
<div class="resume-entry-content">

- 북미 대학생 커뮤니티 <a href="https://www.linkedin.com/company/igotin">IGOTIN</a>에서 Kotlin과 Spring Boot 기반 백엔드 애플리케이션을 개발하고 AWS 환경에서 운영했습니다.
- Instagram 업로드 자동화 과정의 운영상 불편을 개선했습니다. 단일 HTTP 요청에 묶여 있던 외부 API 호출과 DB 작업을 `db-scheduler` 기반 단계 작업으로 분리해 상태 추적과 재시도를 구현했습니다.
- API와 DB의 병목을 추적하는 성능 진단 체계를 구축했습니다. 요청 추적에는 `MDC` 요청 ID를 사용하고 엔드포인트별 지연 확인에는 Athena 기반 ALB 로그 분석을 활용했습니다.
- 채팅 기능 확장에 대응해 Firestore 의존을 걷어내고 Redis Pub/Sub과 WebSocket 기반의 독립 채팅 인프라로 전환했습니다.

</div>
</div>

<div class="two-column-layout">
<div>

## 교육 및 학력

<div class="resume-entry">
<div class="resume-row resume-row--stack">

### 크래프톤 정글

<p class="resume-meta">경기도 용인</p>

</div>
<div class="resume-education-details">
<p class="resume-main">SW-AI Lab 12기 &amp; 심화과정 1기</p>
<p class="resume-meta"><time datetime="2026-03">2026. 03.</time> - 현재</p>

</div>
</div>

<div class="resume-entry">
<div class="resume-row resume-row--stack">

### 광주소프트웨어마이스터고등학교

<p class="resume-meta">전남광주</p>

</div>
<div class="resume-education-details">
<p class="resume-main">소프트웨어개발과 5기</p>
<p class="resume-meta"><time datetime="2021-03">2021. 03.</time> - <time datetime="2024-01">2024. 01.</time></p>

</div>
</div>

## 프로젝트

<ul class="resume-list">
<li>
<div class="resume-row resume-row--stack">
<span class="resume-main"><strong><a href="https://github.com/YangSiJun528/jungle-bell">Jungle Bell</a></strong></span>
<span class="resume-meta"><time datetime="2026-03">2026. 03.</time> - 현재</span>
</div>
<div class="resume-main">크래프톤 정글 생활 관리 데스크톱 앱</div>
</li>
<li>
<div class="resume-row resume-row--stack">
<span class="resume-main"><strong><a href="https://github.com/YangSiJun528/bracket-pair-guides">Bracket Pair Guides</a></strong></span>
<span class="resume-meta"><time datetime="2026-02">2026. 02.</time> - 현재</span>
</div>
<div class="resume-main">JetBrains IDE용 VSCode 스타일 Bracket Pair 플러그인</div>
</li>
<li>
<div class="resume-row resume-row--stack">
<span class="resume-main"><strong><a href="https://www.hellogsm.kr/">Hello, GSM</a></strong></span>
<span class="resume-meta"><time datetime="2022-04">2022. 04.</time> - <time datetime="2023-11">2023. 11.</time></span>
</div>
<div class="resume-main">광주SW마이스터고 입학 지원 서비스</div>
</li>
</ul>

</div>
<div>

## 자격증

<ul class="resume-list">
<li class="resume-row resume-row--compact"><span class="resume-main">리눅스마스터 2급</span><time class="resume-meta" datetime="2023-06">2023. 06.</time></li>
<li class="resume-row resume-row--compact"><span class="resume-main">SQL 전문가(SQLP)</span><time class="resume-meta" datetime="2022-12">2022. 12.</time></li>
<li class="resume-row resume-row--compact"><span class="resume-main">정보처리산업기사</span><time class="resume-meta" datetime="2022-12">2022. 12.</time></li>
<li class="resume-row resume-row--compact"><span class="resume-main">데이터아키텍처 준전문가(DAsP)</span><time class="resume-meta" datetime="2022-07">2022. 07.</time></li>
<li class="resume-row resume-row--compact"><span class="resume-main">SQL 개발자(SQLD)</span><time class="resume-meta" datetime="2021-12">2021. 12.</time></li>
</ul>

## 오픈소스

총 15+ 오픈소스, 40+ 기여

<ul class="resume-list">
<li>
<div class="resume-row resume-row--compact">
<span class="resume-main"><strong><a href="https://github.com/search?q=repo%3Aspring-io%2Finitializr+author%3AYangSiJun528&type=pullrequests">Spring Initializr</a></strong> (외부 기여자 2위)</span>
<span class="resume-meta"><time datetime="2024-09">2024. 09.</time> - 현재</span>
</div>
<ul class="resume-detail-list">
<li>Kotlin JPA 엔티티를 위한 All-Open 설정 지원 (<a href="https://github.com/spring-io/initializr/pull/1576">#1576</a>)</li>
<li>프로젝트 설정 파일의 YAML 형식 지원 (<a href="https://github.com/spring-io/initializr/pull/1682">#1682</a>)</li>
</ul>
</li>
<li>
<div class="resume-row resume-row--compact">
<span class="resume-main"><strong><a href="https://github.com/search?q=repo%3ARustPython%2FRustPython+author%3AYangSiJun528&type=pullrequests">RustPython</a></strong> (2026 OSSCA 멘티)</span>
<span class="resume-meta"><time datetime="2026-07">2026. 07.</time> - 현재</span>
</div>
</li>
</ul>

</div>
</div>
