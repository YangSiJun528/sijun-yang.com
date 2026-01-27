+++
title = "Thompson NFA 정규표현식 엔진 구현"
date = 2026-01-15
description = "정규표현식이 유한 상태 기계로 동작하는 원리를 이해하고 Python으로 구현"

[extra]
page_style = "note"
+++

## Thompson NFA란?

Thompson NFA는 Ken Thompson이 제안한 정규표현식 매칭 알고리즘이다. 백트래킹 없이 O(mn) 시간 복잡도를 보장하는 것이 특징이다.

## 핵심 개념

### NFA vs DFA

- **DFA (Deterministic Finite Automaton)**: 각 상태에서 입력에 대해 정확히 하나의 다음 상태
- **NFA (Non-deterministic Finite Automaton)**: 여러 가능한 경로를 동시에 추적

### ε-전이 (Epsilon Transition)

입력 소비 없이 다른 상태로 이동하는 전이. `a*`에서 "0번 매칭" 경로가 이에 해당한다.

## 구현 핵심

### 상태 표현

```python
class State:
    def __init__(self, char_code, out_edge=None, out_edge1=None):
        self.char_code = char_code  # 문자 or MATCH or SPLIT
        self.out_edge = out_edge    # 다음 상태
        self.out_edge1 = out_edge1  # SPLIT일 때 두 번째 분기
        self.lastlist = 0          # 중복 방문 방지
```

### Thompson 알고리즘

모든 가능한 상태를 집합으로 동시 추적:

```python
def add_state(states, s):
    if s is None or s.lastlist == list_id:
        return  # 이미 처리됨

    s.lastlist = list_id
    states.append(s)

    if s.char_code == STATE_SPLIT:
        add_state(states, s.out_edge)   # ε-전이 재귀
        add_state(states, s.out_edge1)
```

## 성능 비교

| 방식 | 시간 복잡도 | 공간 복잡도 | 장점 | 단점 |
|------|------------|------------|------|------|
| 백트래킹 | O(2^n) 최악 | O(n) | 역참조 지원 | 성능 예측 불가 |
| Thompson | O(mn) | O(m) | 예측 가능한 성능 | 확장 기능 제한 |

## 실제 사용 예

```python
# "a(b|c)*" 패턴으로 "abcbc" 매칭
pattern = "a(b|c)*"
text = "abcbc"
result = match(pattern, text)  # True
```

## 참고 자료

- [Regular Expression Matching Can Be Simple And Fast](https://swtch.com/~rsc/regexp/regexp1.html)
- [FSM2Regex - 유한 상태 기계 시각화](https://ivanzuzak.info/noam/webapps/fsm2regex/)

## 배운 점

정규표현식이 단순한 문자열 매칭이 아니라 오토마타 이론에 기반한 체계적인 알고리즘이라는 점이 흥미로웠다. 특히 Thompson NFA의 list_id 트릭으로 중복 방문을 방지하는 방식이 인상적이었다.