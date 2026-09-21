+++
aliases = ["/ko/portfolio/igotin-admin/"]
title = "IGOTIN 관리자 화면 전후 비교"
slug = "igotin-admin"
template = "portfolio-detail.html"
hidden = true
in_search_index = false
include_in_feeds = false

[extra]
return_anchor = "instagram"
+++

<section class="portfolio-sheet portfolio-sheet--admin" id="admin-comparison" aria-label="IGOTIN 관리자 화면 전후 비교">

## Instagram 관리자 화면 전후 비교

업로드 처리 백엔드 개선 전후의 상태 조회와 재시도 흐름을 비교한 예시 화면입니다.

<div class="portfolio-gallery portfolio-gallery--stack portfolio-admin-screens">

<figure class="portfolio-screenshot">
<p class="portfolio-figure-title"><strong>개선 전 예시 화면</strong></p>
<a href="/images/portfolio/portfolio-instagram-ui-before-v2.png" target="_blank" rel="noopener" aria-label="개선 전 예시 화면 원본 크기로 보기">
<img src="/images/portfolio/portfolio-instagram-ui-before-v2.png" alt="Instagram 업로드 개선 전 관리자 화면 예시" width="1584" height="993">
</a>
<figcaption><em>HTML로 재구성한 예시 화면, 회사의 자료를 사용하지 않았습니다.</em><br>최종 성공/실패만 표시, 실패 지점과 원인 파악 불가</figcaption>
</figure>

<figure class="portfolio-screenshot">
<p class="portfolio-figure-title"><strong>개선 후 예시 화면</strong></p>
<a href="/images/portfolio/portfolio-instagram-ui-after-v2.png" target="_blank" rel="noopener" aria-label="개선 후 예시 화면 원본 크기로 보기">
<img src="/images/portfolio/portfolio-instagram-ui-after-v2.png" alt="Instagram 업로드 개선 후 단계별 상태와 오류 로그를 표시하는 관리자 화면 예시" width="1584" height="993">
</a>
<figcaption><em>HTML로 재구성한 예시 화면, 회사의 자료를 사용하지 않았습니다.</em><br>단계별 상태와 오류 로그를 확인, 작업 재시도/중단 가능</figcaption>
</figure>

</div>

</section>
