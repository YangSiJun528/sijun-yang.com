# 부동소수점 자세하게 알아보기

## 부동소수점의 필요성
1. 컴퓨터의 숫자 표현 한계
    - 컴퓨터는 이진수(0과 1) 로 데이터를 표현한다.  
    - 정수는 각 비트를 2의 거듭제곱(1, 2, 4, 8, …) 으로 해석하여 쉽게 표현 가능하다.  
    - 하지만 소수 부분이 있는 실수(real number) 는 이진수로 정확히 표현하기 어렵다.  

### 2. 고정소수점 표현(Fixed-point)
- 정수 비트와 소수 비트를 고정된 개수로 나누어 표현한다.  
    - 예: 정수부 4비트 + 소수부 4비트  
- 소수부 비트는 ½, ¼, ⅛, 1⁄16 등을 의미한다.  
    - 예를 들어, 1.5(=1 + ½)나 4.75(=4 + ½ + ¼)를 정확히 표현할 수 있다.  
    - 그러나 2.8과 같은 수는 이 방식으로 정확히 표현할 수 없으며 근사값만 저장된다.  

### 3. 고정소수점의 한계
- 표현 가능한 숫자의 범위(동적 범위) 가 제한된다.  
    - -> 일부 비트를 소수부에 사용하면 큰 수를 표현할 수 있는 범위가 줄어듦.  
- 실수는 무한하지만 컴퓨터의 비트 수는 유한하기 때문에, 모든 실수를 정확히 표현하는 것은 불가능하다.   

### 4. 부동소수점의 등장(Floating-point)
- 이를 해결하기 위해 정수부/소수부 대신 ‘가수’와 ‘지수’ 로 표현하는 방식이 도입되었다.
- 이는 과학적 표기법(scientific notation) 을 기반으로 한다.
- 현대 컴퓨터는 이를 표준화한 IEEE 754 부동소수점 형식을 사용한다.

## 과학적 표기법(scientific notation)

- 과학적 표기법이란?
    - > 핵심은 굉장히 다른 척도를 설명하는 데 같은 수의 숫자를 쓴다는 것이다. 컴퓨터 과학자들은 바로 이 통찰을 활용해 큰 범위의 수를 표기하는 너비가 고정된 형식을 만들어 낸다. - Rust In Action p.180
    - 큰 수나 작은 수를 간단하게 표현하기 위한 방식
    - 예: 목성의 질량을 1.898×10^27kg, 개미의 질량을 3.801×10^-4kg으로 나타낸다.

- 과학적 표기법 구성 요소
    - 부호(Sign) : 양수/음수 구분 (선택적 요소)
    - 가수(Mantissa/Significand) : 숫자의 유효 자릿수
    - 밑(Base) : 과학적 표기법에선 보통 10 (컴퓨터에선 2진수를 사용)
    - 지수(Exponent) : 소수점 위치 이동량

## IEEE 754 표준

- 숫자의 과학적 표기법을 기반으로 하는 부동소수점 자료형의 구현 방법을 정의하는 표준 문서이다.
- 컴퓨터의 메모리에는 한계가 있기 때문에. 넒은 범위를 표현 가능하지만 정확한 수를 나타내지 못한다는 단점이 있다.

### IEEE 754의 저장 형식

"부호 > 지수 > 가수" 순서대로 저장된다.

컴퓨터는 2진수로 저장되기 때문에 밑은 항상 2로 고정되어 저장하지 않는다.

- 부호 비트(Sign bit)
    - 1비트 사용
    - `0` → 양수, `1` 음수
- 지수부(Exponent)
    - Bias(편향) 을 적용해 계산됨. 
        - Bias는 “중간값을 0으로 맞추기 위해 더하는 상수”이며, 계산식은 `2^(지수 비트 수 - 1) - 1`. (예: 8비트 → bias = 127)
        - 음수/양수를 구분하지 않아서 1비트를 아낄 수 있다.
    - 예시
        - 실제 지수(E)가 -3일 때
        - 저장되는 값(E+Bias)은 124(127 + -3)
- 가수부(Mantissa / Fraction)
    - 2진 과학적 표기법에서 항상 `1.xxx` 형태이므로, 맨 앞의 `1`은 생략(implicit 1) 되어 저장되지 않는다. 
        - 1비트 저장공간을 아낄 수 있다.

### IEEE 754 타입의 저장 형식

다음은 대표적인 2가지 타입. 

| 타입                  | 전체 크기       | 부호 | 지수 | 가수 | Bias |
| -------------------- | ------------- | -- | -- | -- | ---- |
| Single (float)   | 32비트 (4바이트) | 1  | 8  | 23 | 127  |
| Double (double)  | 64비트 (8바이트) | 1  | 11 | 52 | 1023 |

이 2가지 경우만으로 대부분의 요구사항을 표현할 수 있다.  
그 외에도 binary16(GPU 연산에서 자주 쓰임), binary128, binary256, Decimal32/64/128(10진수 기반)이 정의되어있다.

### 특수한 값들 (예약된 조합)

지금까지의 셜명으로는 다음 값을 표현할 수 없다는 문제가 있다.
1. 가수가 항상 1이여서 0을 표현할 수 없다 / 0에 가까운 값을 표현할 수 없다
2. 계산 결과가 너무 크다
3. 0으로 나누거나 음수의 제곱근을 취하는 경우

실수는 이론적으로 무한 범위를 가지므로, 특정 지수부/가수부 조합을 예약하여 다음과 같은 특수값을 표현한다.

| 지수부                  | 가수부 | 의미                                             |
| --------------------- | ---- | ---------------------------------------------- |
| `0`                   | `0`  | ±0 (양의/음의 0)                                     |
| `0`                   | ≠`0` | 서브노멀(subnormal) 수 — 0에 가까운 작은 값 표현용         |
| `MAX` (`255`/`2047`)  | `0`  | ±∞ (무한대)                                         | 
| `MAX` (`255`/`2047`)  | ≠`0` | NaN (Not a Number) — 계산 불가능한 값 (예: 0/0, √(-1)) |

### 심화 내용 들어가면서

여기서부턴 자료를 확인하긴 했는데, 부정확한 내용이 있을 수 있음. 아니면 너무 딥한 내용이라 건너 뛴 부분도 있다.

주로 부동소수점 자료들에 대해서 햇갈리기 쉽거나, 실제 사용 시 주의할 부분에 대해서 다룰 것.

대부분 정밀도와 관련된 내용임. 그래서 아래 내용을 외우는것보다 부동소수점의 특징으로 인해서 발생하는 주의점? 정도로 이해하면서, 부동소수점 자체를 이해하려는게 중요해보임.

### 이해하기 좋기 위해서 내가 쓸 표현 방식 설명

여기까지 보면 현실의 수를 float/double로 저장하면 정확하지 않을 수 있다는 것을 알 것임.

앞으로 설명을 쉽게 하기 위해서 다음 표현을 사용하겠다.
- 실제 수: N
- float/double 저장 형식으로 처리된 된 수: float(N) / double(N)
- 정수형 저장 형식, 편의를 위해 자료형 구분 없음: int(N)
- 사용 예: 
    - `float(0.1) + float(0.2) != float(0.3)`, `float(0.1) + float(0.2) != 0.3`
    - `int(1) + int(2) == int(3)`

### subnormal(비정규화) 값

비정규화 값은 IEEE 754 부동소수점 표현에서, 정상(normal) 값이 표현할 수 있는 최소값보다 더 작은 값을 저장하기 위한 특수한 방식.

지수가 0일 때 사용되며, 이 경우 가수부의 가장 앞에 숨겨진 1을 두지 않고 그대로 저장한다. (`0.xxxxx` 꼴)

정상 float가 표현하는 최소값(1.18×10^−38)보다 더 작은 값을 나타낼 수 있다.

subnormal의 필요성에 대한 잘 정리된 글: https://stackoverflow.com/a/53203428

### 정밀도(significand precision)

유효 자릿수, 표현 가능한 십진수 자릿수 정도로 표현 가능하다.

> For most of our purposes when we say that a format has n-digit precision we mean that over some range, typically [10^k, 10^(k+1)), where k is an integer, all n-digit numbers can be uniquely identified. [^1]
> 대부분의 경우, 어떤 형식이 n자리 정밀도를 가진다고 말할 때, 이는 특정 범위(일반적으로 [10^k, 10^(k+1)))에서 모든 n자리 숫자를 고유하게 식별할 수 있다는 것을 의미합니다. 

- 위 수식과 문맥에서 쓰는 기호 정리 `[10^k, 10^(k+1)`
    - `[`: 이상
    - `)`: 미만
    - `k`: 범위
    - `n`: 정밀도
- 예:
    - k=6, n=7: `[1,000,000 ~ 10,000,000)` 범위에서 7자리 정밀도
    - k=-1, n=4: `[0.1 ~ 1.0)` 범위에서 4자리 정밀도

#### 여기 아래서부터는 자료 참고해서 내용 보완하기, 이 이상은 실무에서 필요하면 그 때 배워도 되지 않을까?

#### Wobble(흔들림) - TODO: 이거 보완해야 함 Wobble이 그래서 왜, 뭐가 문제인지 모르겠음. 이 섹션은 AI 요약본 복붙한거

Wobble의 정확한 정의는 [^3]를 참고하여 수정해야 한다.

- ULP (Units in the Last Place): 부동소수점 수의 마지막 자리 단위. 어떤 수와 그 다음 표현 가능한 수 사이의 간격.
- ULP = 해당 지수에서 표현 가능한 최소 간격
- 오차를 정수 단위로 측정 가능 (직관적)
- 알고리즘 정확도 분석의 표준 단위

Wobble: 부동소수점에서 0.5 ulp의 절대 오차가 값에 따라 다른 상대 오차를 갖는 현상

수학적 정의
```
부동소수점 형식: β (진법), p (정밀도)
지수 e인 구간에서:

절대 오차 (고정): (β/2)β⁻ᵖ × βᵉ
값의 범위: βᵉ ≤ x < β^(e+1)

상대 오차 범위:
  최소값에서: (β/2)β⁻ᵖ × βᵉ / β^(e+1) = (β/2)β⁻ᵖ⁻¹
  최대값에서: (β/2)β⁻ᵖ × βᵉ / βᵉ = (β/2)β⁻ᵖ

→ Wobble = β배 차이
```
핵심: 절대 오차는 같지만, 값의 크기에 따라 상대 오차가 β배 변동

#### 진법 변환 비트 낭비 - TODO: 이거 보완해야 함. 여기도 AI

> The variation in relative precision due to wobble is important later on and can mean that we 'waste' almost an entire digit, binary or decimal, when converting numbers from the other base.[^1]  
> 흔들림으로 인한 상대적 정밀도의 변화는 나중에 중요한 문제가 되며, 다른 진법에서 숫자를 변환할 때 이진수나 십진수 등 거의 전체 숫자를 '낭비'하게 될 수 있습니다.

Wobble로 인한 낭비:
- 맥락: 오차 분석 시
- 의미: Wobble 때문에 최악 케이스 고려 필요
- 결과: 오차 한계 계산 시 β배 여유 필요

Wobble과 별개로 진법 차이로 인한 십진 <-> 이진 정확 표현 불가능한 문제가 있음. (대부분의 수의 소인수가 2 * 5를 포함하지 않으므로...)

#### 정밀도는 상대적이다. - TODO: 보완 필요

> We have said several times that the precision of a floating-point value is proportional to its magnitude. So instead of saying that the number is precise to the nearest integer (like we do for integer formats), we say that a floating-point value is precise to X% of its value.[^2]    
> 부동 소수점 값의 정밀도는 크기에 비례한다고 여러 번 말씀드렸습니다. 따라서 정수 형식처럼 숫자가 가장 가까운 정수까지 정확하다고 말하는 대신, 부동 소수점 값은 해당 값의 X% 까지 정확하다고 말합니다.

> Recall that only a finite set of numbers can be exactly represented in any computer arithmetic. As the magnitudes of numbers get smaller and approach zero, the gap between neighboring representable numbers narrows. Conversely, as the magnitude of numbers gets larger, the gap between neighboring representable numbers widens.[^4]  
> 모든 컴퓨터 연산에서 정확하게 표현할 수 있는 숫자의 수는 유한하다는 점을 기억하세요. 숫자의 크기가 작아지고 0에 가까워질수록, 표현 가능한 이웃 숫자들 사이의 간격은 좁아집니다. 반대로, 숫자의 크기가 커질수록, 표현 가능한 이웃 숫자들 사이의 간격은 넓어집니다.
> ![example-img-01](https://docs.oracle.com/cd/E19957-01/806-3568/images/fig2-6.gif)

부동 소수점 값의 정밀도는 절대적인 수치가 아닌, 크기에 비례함. (가장 높은 자리수부터 N 자리수까지 정확하다고 표현 가능함.)

따라서 큰 수와 작은 수의 계산을 수행했음에도 결과가 변화지 않는 문제가 발생할 수 있음.

이 점 때문에, 수가 커질수록 유효하게 표현 가능한 가장 작은 수 단위(아마 ULP?라고 불렀던거 같음)도 증가함.

> 시각적인 자료로 보고 싶다면, [^2]이 자료의 숫자 선(number line) 부분을 참고하자.

### 그래서 정확한 정밀도 값이 무엇인가? - 보완 조금만? 여기는 이해보다는 사실 요약에 가까워서

흔히들 다음과 같이 정의한다. 그런데 저 수의 의미가 무엇일까?

- 정밀도(significand precision)
    - 유효 자릿수, 표현 가능한 십진수 자릿수
- IEEE 754 Single
    - 6 to 9 significant decimal digits
- IEEE 754 Double
    - 15 to 17 significant decimal digits

> So d-digit precision (d a positive integer) means that if we take all d-digit decimal numbers over a specified range, convert them to b-bit floating-point, and then convert them back to decimal — rounding to nearest to d digits — we will recover all of the original d-digit decimal numbers. In other words, all d-digit numbers in the range will round-trip. [^5]  
> 따라서 d자리 정밀도(양의 정수)는 지정된 범위에 있는 모든 d자리 십진수를 b비트 부동 소수점으로 변환한 후, 다시 d자리에 가장 가까운 자리로 반올림하여 십진수로 변환하면 원래의 d자리 십진수를 모두 복구할 수 있음을 의미합니다. 다시 말해, 해당 범위에 있는 모든 d자리 숫자가 라운드트립됩니다.

round-trip: 왕복 변환/복원 정확도 정도로 번역 가능. 

[^5] 이 자료에 따르면 다음과 같이 표현할 수 있다. 단, subnormal(비정규화) 값을 포함하지 않는다. (subnormal은 0자리수까지 내려간다. [^1])

- 10진수 → float/double → 10진수 
    - "몇 자리 10진수가 안전하게 왕복하나?"
    - float: 6자리 (안전), 6-8자리 (범위에 따라 다름)
    - double: 15자리 (안전), 15-16자리 (범위에 따라 다름)
    - 메모: 10진수를 2진수이면서 제한된 공간인 float/double에 저장하면서 값 손실이 발생하는데, 범위에 따라 자리수에 차이가 있으므로 가장 작은 자리수가 보장된다.
- float/double → 10진수 → float/double
    - "float/double을 복원하려면 몇 자리로 출력해야 하나?" (10진수로 저장하고 그걸 float/double으로 저장해도 이전과 동일한 값을 가지려면?)
    - float: 9자리
    - dobule: 17자리
    - `ceiling(1 + significand_bits * log10(2))` 수식으로 구할 수 있다. 
    - 관련 글(수식 설명은 없음.)[^6]

### float의 올바른 계산을 위한 기법 - TODO: Impl

앱실론 뭐시기...

이거는 필요하면 공부하기... 솔직히 여기까지는 너무 과한듯

- https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html
- https://randomascii.wordpress.com/2012/02/25/comparing-floating-point-numbers-2012-edition/

### 하드웨어 관련 설정 - TODO: Impl

아마 계산 시 반올림/내림/올림 설정이나 여러 옵션 있는걸로 아는데...

이걸 알아야 할 일이 있을지는 모르겠지만, 일단 TODO로 적어둠.

### 정밀도와 관련된 여러가지 실용적인 팁

- [시간은 float로 저장하지 않기](https://randomascii.wordpress.com/2012/02/13/dont-store-that-in-a-float/): 상대적인 시간?그거를 잘 보여주는 실제 예시임.
    - float의 1.0을 1초로 계산했을 때, 1일만 지나도 7.8밀리초 배수의 배수로만 표현할 수 있음. 
    - N밀리초 단위로 계산하는 로직이 있으면 시간이 지남에 따라 올바르지 않게 동작할 수 있음.
    - double이나 uint64을 사용하면 안전함. 
        - double: 수천 년 이후에도 밀리초 미만의 정밀도를 유지한다.
        - uint64: 마이크로초 단위로 저장해도 약 58만 년까지 저장 가능.
    - 애플의 NSTimeInterval는 double, unix의 time_t는 정수형 타입을 사용한다.
- float의 정확한 값은 100자리를 넘길수 있다[^1]
    - float(N)은 2진수로 근사치를 저장하므로, 실제 float(N)을 십진수로 저장해서 정확한 값을 보려면 최대 100자리수 이상이 필요하다.
    - 그러나 이 값을 근사치 값일 뿐이라 100자리수까지 보지는 않는다.
- [^2] Integers are exact! As long as they’re not too big.
    - 부동 소수점 숫자는 충분히 작다면 정수를 정확하게 표현할 수 있습니다.

---

[^1]: [Float Precision–From Zero to 100+ Digits – tech blog of Bruce Dawson](https://randomascii.wordpress.com/2012/03/08/float-precisionfrom-zero-to-100-digits-2/)
[^2]: [Floating Point Demystified, Part 1 – Josh Haberman](https://blog.reverberate.org/2014/09/what-every-computer-programmer-should.html)
[^3]: [What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html#689)
[^4]: [IEEE Arithmetic - oracle](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_math.html)
[^5]: [Decimal Precision of Binary Floating-Point Numbers](https://www.exploringbinary.com/decimal-precision-of-binary-floating-point-numbers/)
[^6]: [They sure look equal… - tech blog of Bruce Dawson](https://randomascii.wordpress.com/2012/02/11/they-sure-look-equal/)

그 외 참고자료

[Comparing Floating Point Numbers, 2012 Edition](https://randomascii.wordpress.com/2012/02/25/comparing-floating-point-numbers-2012-edition/) - 부동소수점을 사용한 실용적인 계산 연산을 하는 법을 다루는것 같음.