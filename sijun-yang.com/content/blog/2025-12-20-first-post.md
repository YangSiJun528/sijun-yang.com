+++
title = "첫 번째 블로그 글" 
description = "Zola로 만든 블로그의 첫 번째 예시 글입니다."

# zola가 파일 이름을 `날짜-페이지URI`로 분리해서 frontmatter를 추가해줌. 
# 필요하다면 직접 명시하거나 적용 옵션을 바꿀수도 있는데, 파일 이름 바꾸지만 않으면 괜찮을거 같음.

[extra]
page_style = "blog-post"
+++

이것은 예시 블로그 글입니다.

## 소제목

Zola를 사용하면 Markdown으로 쉽게 블로그를 작성할 수 있습니다.

### 코드 예시

```rust
fn main() {
    println!("Hello, Zola!");
}
```

### 리스트

- 항목 1
- 항목 2
- 항목 3

**굵은 글씨**와 *기울임꼴*도 사용할 수 있습니다.
