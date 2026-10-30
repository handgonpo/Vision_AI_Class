## 오늘의 목표

7일차에서는 정상 데이터가 충분할 때는 어떻게 검사했을까요?

7일차에서는 **정상 이미지를 충분히 가지고 있는 상황**을 가정했습니다.

MVTec `bottle`의 정상 병 여러 장을 Pretrained ResNet18에 넣어 이미지의 특징을 숫자로 바꾸고, 이 특징들을 모아 **정상 제품의 기준(Normal Reference)** 을 만들었습니다. 새로운 병이 이 정상 기준과 비슷하면 `Normal`, 많이 다르면 `Anomaly`로 판단하는 방식이였습니다.

```text
정상 병 이미지 여러 장
        ↓
Pretrained ResNet18
        ↓
정상 이미지의 Feature 추출
        ↓
Normal Reference
'정상은 대략 이런 특징을 가진다'
        ↓
Normal Validation
정상 이미지가 기준에서 얼마나 떨어지는지 확인
        ↓
Threshold 결정
'여기까지는 정상으로 보자'
        ↓
Rule Freeze
판정 기준 고정
        ↓
처음 보는 Test 이미지
        ↓
Normal / Anomaly 판정
```

즉, 정상 데이터를 충분히 모아 정상 기준을 만들고, 새로운 이미지가 그 정상 기준에서 얼마나 벗어나는지를 이용하여 이상을 찾았습니다.

그리고 후반에는 정상 제품이라도 밝기·흐림·위치·회전 같은 촬영 조건이 달라지면 이상으로 잘못 판단될 수 있다는 점도 확인했습니다.

---
### 그러면 8일차에는 무엇이 달라질까요?

오늘은 검사 방법 자체보다 먼저 **데이터 조건을 바꾸어 봅니다.**

실제 제조 현장에서는 항상 많은 데이터를 가지고 검사를 시작할 수 있는 것은 아닙니다.

예를 들어 다음과 같은 상황이 있을 수 있습니다.

```text
불량 이미지가 거의 없다.

정상 이미지도 충분하지 않다.

기준으로 사용할 이미지가 한 장뿐이다.

정상·불량 예제가 몇 장밖에 없다.
```

그렇다면 질문이 달라집니다.

```
데이터가 충분함
        ↓
7일차
정상 분포를 이용한 Anomaly Detection


데이터가 매우 적음
        ↓
8일차
적은 데이터로
어떻게 검사 기준을 만들 것인가?
```

쉽게 설명하면 7일차에서는 정상 데이터가 충분할 때 정상 기준을 만들었고, 8일차에서는 데이터가 매우 적을 때 어떤 전략을 사용할지 비교해 보는 것입니다.

이 질문이 오늘 8일차 수업의 출발점입니다.

---

## 오늘 수업은 크게 이렇게 진행합니다

처음부터 모든 용어를 외울 필요는 없습니다.

오늘 수업은 크게 다음 흐름으로 진행합니다.

```
① Day08 데이터 준비
        ↓
② One-shot
정상 이미지 1장을 기준으로 검사
        ↓
③ Few-shot
소수의 정상·불량 예제로 검사
        ↓
④ 같은 이미지에서
One-shot / Few-shot 결과 비교
        ↓
⑤ 부족한 불량 데이터 합성
        ↓
⑥ Challenge
직접 조건을 바꾸어 다시 실험
        ↓
⑦ 실패 원인과 결과 정리
```

수업 중에 등장하는

```
Reference
Calibration
Support
Prototype
Query
Ground Truth Mask
Metadata
```

와 같은 용어는 지금 모두 외울 필요가 없습니다.

각 용어가 실제로 필요한 실습에 들어갔을 때 하나씩 의미를 확인합니다.

> 지금은 “사용할 수 있는 데이터의 양이 달라지면 검사 기준을 만드는 방법도 달라진다”는 것만 기억하면 됩니다.

---

## 오늘도 7일차의 ResNet18을 그대로 사용합니다

오늘 새로운 CNN을 처음부터 학습하지 않습니다.

7일차에서 사용했던 ImageNet Pretrained ResNet18을 다시 사용합니다.

이미 준비된 원본은 다음 위치에 있습니다.

```text
samples/day07/original/bottle/
├── train/
│   └── good/
├── test/
│   ├── good/
│   ├── broken_large/
│   ├── broken_small/
│   └── contamination/
└── ground_truth/
    ├── broken_large/
    ├── broken_small/
    └── contamination/
```

8일차에서는 같은 원본에서 필요한 이미지만 따로 복사하여 데이터가 적은 상황을 일부러 만듭니다.

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있으면 됩니다.

```text
- Zero-shot / One-shot / Few-shot의 차이를 설명할 수 있다.
- 7일차 Anomaly Detection과 8일차 소량 데이터 실험의 차이를 설명할 수 있다.
- One-shot에서 Reference가 왜 한 장인지 설명할 수 있다.
- Calibration 데이터와 Reference 데이터의 역할을 구분할 수 있다.
- Pretrained ResNet18으로 이미지 Feature를 추출할 수 있다.
- Cosine Similarity가 높고 낮다는 의미를 설명할 수 있다.
- 정상 Calibration으로 One-shot Threshold를 정하고 Freeze할 수 있다.
- Few-shot에서 Support와 Query를 구분할 수 있다.
- Normal / Anomaly Support로 Prototype을 만들 수 있다.
- 같은 Common Query에서 One-shot과 Few-shot을 비교할 수 있다.
- Support 수를 1 / 3 / 5장으로 바꾸었을 때 결과가 어떻게 달라지는지 확인할 수 있다.
- MVTec Ground Truth Mask에서 실제 결함 Patch를 추출할 수 있다.
- 정상 이미지에 결함 Patch를 붙여 Synthetic Defect를 만들 수 있다.
- 합성 이미지 Metadata를 기록할 수 있다.
- Domain Gap과 Synthetic Quality Check가 필요한 이유를 설명할 수 있다.
- 데이터 상황을 보고 Zero-shot / One-shot / Few-shot / Anomaly / 
  Defect Generation 중 후보 방법을 선택할 수 있다.
```

---

# 1. 8일차의 전체 흐름

오늘 교재는 PART1부터 PART9까지 구성되어 있습니다.

지금 각 PART의 세부 내용을 모두 이해할 필요는 없습니다.

먼저 전체 위치만 확인하고 실제 실습에 들어가면서 하나씩 진행합니다.

![[Pasted image 20260911144942.png]]

```text
PART1
실습 환경 + Day08 데이터 준비
7~12
        ↓

PART2
Zero / One / Few-shot 개념
+ 공통 Feature 함수
13~18
        ↓

PART3
One-shot 실습
19~25
        ↓

PART4
Few-shot 실습
26~32
        ↓

PART5
같은 Query에서
One-shot / Few-shot 결과 비교
33~37
        ↓

PART6
Defect Generation
부족한 불량 이미지 합성
38~45
        ↓

PART7
Challenge
Support 수를 바꾸어 비교
46~50
        ↓

PART8
Mini Challenge
새 Defect Type으로 다시 적용
51~55
        ↓

PART9
실패 분석 + 결과 정리
README + Git + 자가체크
56~63
```

각 PART에서 사용하는 세부 개념과 코드는 해당 실습에 들어갔을 때 하나씩 배웁니다.

## 1.1 PART1 — 먼저 데이터를 정확하게 준비합니다

오늘은 Dataset을 새로 수집하지 않습니다.

7일차에서 보존한 MVTec `bottle` 원본에서 8일차에 필요한 데이터만 다시 선택합니다.

```text
MVTec bottle 원본
        ↓
Day08에 필요한 이미지 선택
        ↓
각 실험에 맞게 역할 분리
        ↓
데이터가 서로 겹치지 않는지 확인
```

One-shot과 Few-shot에서는 **어떤 이미지를 기준으로 사용했고 어떤 이미지를 평가에 사용했는가**가 매우 중요합니다.

같은 이미지를 기준을 만드는 데 사용하고 다시 평가에도 사용하면 실제보다 결과가 좋아 보일 수 있습니다.

따라서 오늘은 먼저 코드로 데이터가 겹치지 않는지 확인합니다.

## 1.2 PART2 — 적은 데이터에서 사용할 방법을 구분합니다

PART2에서는 Zero-shot, One-shot, Few-shot이 어떤 상황에서 사용되는지 먼저 구분합니다.

그리고 이후 실습에서 공통으로 사용할 ResNet18 Feature 추출 함수를 준비합니다.

```text
Bottle Image
        ↓
Pretrained ResNet18
        ↓
512차원 Feature Vector
```

오늘은 이 Feature를 이용해 이미지끼리 얼마나 비슷한지를 비교합니다.

## 1.3 PART3 — 정상 이미지 한 장으로 검사합니다

PART3에서는 **One-shot**을 실습합니다.

정상 이미지 한 장을 기준 이미지로 정하고 새로운 이미지와 얼마나 비슷한지 확인합니다.

```text
Normal Reference 1장
        ↓
Reference Feature
        ↓
정상 이미지의 Similarity 범위 확인
        ↓
판정 기준 결정
        ↓
새로운 이미지 검사
        ↓
Normal / Anomaly
```

세부적으로 왜 Calibration 데이터가 필요한지, Threshold는 어떻게 결정하는지는 실제 One-shot 실습에서 확인합니다.

## 1.4 PART4 — 몇 장의 예제로 검사합니다

PART4에서는 **Few-shot**을 실습합니다.

이번에는 한 장이 아니라 정상과 불량 예제를 각각 몇 장씩 사용합니다.

```text
Normal 예제 몇 장
        ↓
정상 대표 Feature

Anomaly 예제 몇 장
        ↓
불량 대표 Feature

새로운 이미지
        ↓
어느 대표 Feature와
더 가까운지 비교
        ↓
Normal / Anomaly
```

이때 만들어지는 대표 Feature를 `Prototype`이라고 부릅니다.

지금 정의를 외우기보다 실제 Feature를 만들어 보면서 의미를 확인합니다.

## 1.5 PART5 — 같은 이미지에서 두 방법을 비교합니다

One-shot과 Few-shot 결과를 서로 비교하려면 **같은 평가 이미지**를 사용해야 합니다.

![[Pasted image 20260912204256.png]]

서로 다른 이미지에서 평가하면 방법 차이 때문인지 데이터 차이 때문인지 구분하기 어렵습니다.

따라서 오늘은 같은 Query를 두 방법에 넣어 결과를 비교합니다.

```
- Normal = 정상 이미지
- Anomaly = 비정상·이상 이미지
```

```
One-shot

정상 기준 이미지 1장
        ↓
새 이미지와 비교
        ↓
Normal / Anomaly

---------------------------------

Few-shot

정상 이미지 몇 장
        +
불량 이미지 몇 장
        ↓
각 그룹의 대표 특징 생성
        ↓
새 이미지가 어느 쪽과 더 가까운지 비교
        ↓
Normal / Anomaly
```

## 1.6 PART6 — 부족한 불량 이미지를 만들어 봅니다

실제 현장에서는 정상 데이터보다 불량 데이터가 훨씬 부족한 경우가 많습니다.

PART6에서는 MVTec에 포함된 실제 결함 위치 정보를 이용하여 간단한 합성 불량을 만들어 봅니다.

```text
실제 불량 이미지
+
실제 결함 위치
        ↓
결함 부분 추출
        ↓
정상 이미지에 합성
        ↓
Synthetic Defect
```

여기서 중요한 것은 합성 이미지를 많이 만드는 것이 아닙니다.

> **만들어진 이미지가 실제 학습이나 검사에 사용할 만큼 자연스럽고 의미가 있는지 확인하는 과정도 필요합니다.**

그래서 합성 결과와 함께 Metadata와 품질도 확인합니다.

## 1.7 PART7 — 데이터 수가 달라지면 결과도 달라질까요?

Few-shot은 사용하는 예제가 매우 적습니다.

따라서 어떤 몇 장을 선택했는가에 따라 결과가 달라질 수 있습니다.

Challenge에서는 Support 수를 바꾸어 봅니다.

```text
Support 1장
Support 3장
Support 5장
        ↓
같은 Query 검사
        ↓
성능 비교
```

이 실험을 통해

> **예제가 많아지면 항상 좋아지는가?**

또는

> **적은 데이터에서는 어떤 이미지를 선택했는가도 중요한가?**

를 직접 확인합니다.


## 1.8 PART8 — 다른 결함에도 다시 적용합니다

본 실습에서는 주로 `broken_small` 결함을 사용합니다.

Mini Challenge에서는 새로운 Dataset을 추가하지 않고, 아직 사용하지 않은 `contamination` 결함을 이용하여 같은 흐름을 스스로 다시 적용합니다.

```
본 실습
broken_small
        ↓
방법 이해

Mini Challenge
contamination
        ↓
같은 방법을 스스로 다시 적용
```

새로운 알고리즘을 배우는 문제가 아니라,

> **오늘 배운 과정을 다른 결함에서도 다시 적용할 수 있는지 확인하는 실습입니다.**

## 1.9 PART9 — 결과보다 근거를 정리합니다

마지막에는 단순히 Accuracy 숫자 하나만 확인하지 않습니다.

오늘 실험에서 다음 내용을 정리합니다.

```text
어떤 데이터를 사용했는가?

어떤 방법을 사용했는가?

One-shot과 Few-shot 결과는 어떻게 달랐는가?

Support 수를 바꾸면 결과가 어떻게 변했는가?

합성한 결함은 자연스러웠는가?

어떤 경우에 잘못 판단했는가?

왜 그런 결과가 나왔는가?
```

그리고 실습 결과를 README와 Git 기록으로 정리합니다.


---

# 2. 7일차와 8일차는 어떻게 연결되나요?

7일차와 8일차는 완전히 다른 수업이 아닙니다.

둘 다 사전학습된 ResNet18 Feature를 이용하지만 **사용할 수 있는 데이터의 양과 기준을 만드는 방법**이 달라집니다.

| 구분 | 7일차 | 8일차 |
|---|---|---|
| 문제 | 정상과 다른 제품 찾기 | 데이터가 적을 때 검사 방법 찾기 |
| 주요 데이터 | 정상 Train 여러 장 | 1장 또는 소수 Support |
| 기준 | Normal Reference | Reference 또는 Prototype |
| 비교값 | Euclidean Distance | Cosine Similarity |
| 결과 | Normal / Anomaly | Normal / Anomaly |
| 추가 실습 | 환경 변화 Stress Test | Support 수 변화 + Defect Generation |

# 오늘 수업이 끝나면

오늘 수업이 끝났을 때 모든 수식을 외울 필요는 없습니다.

대신 다음 질문에 답할 수 있으면 됩니다.

```
정상 데이터가 충분하다면?
        ↓
7일차 Anomaly Detection

-----------------------------------

정상 기준 이미지가 한 장이라면?
        ↓
One-shot

-----------------------------------

정상·불량 예제가 몇 장 있다면?
        ↓
Few-shot

-----------------------------------

불량 이미지가 너무 부족하다면?
        ↓
Defect Generation도 고려
```

그리고 가장 중요한 질문은 이것입니다.

> **현재 가지고 있는 데이터의 양과 상태를 보고 어떤 검사 전략을 선택할 것인가?**

8일차의 목표는 One-shot이나 Few-shot 중 하나가 항상 더 좋다고 결론 내리는 것이 아닙니다.

> **데이터 조건이 달라지면 검사 전략도 달라져야 한다는 것을 직접 실험하고 판단하는 것이 오늘 수업의 핵심입니다.**


---

7일차에서 만든 `train / validation / test / challenge` 결과 폴더를 8일차의 원본처럼 다시 사용하지 않습니다.

8일차는 7일차에서 보존한 원본 위치에서 필요한 이미지만 다시 선택합니다.

```text
samples/day07/original/bottle/
        ↓
읽기 전용 원본 보존
        ├───────────────┐
        ↓               ↓
7일차 실습용 분리       8일차 실습용 분리
samples/day07/...       samples/day08/...
```

이렇게 하면 7일차에서 변형하거나 생성한 파일이 8일차 데이터에 섞이지 않고, 두 일차가 같은 원본을 기준으로 독립적인 데이터 역할을 구성할 수 있습니다.

---

# 3. 오늘 사용할 본 실습 데이터

오늘은 7일차에서 사용했던 MVTec `bottle` 원본 중 **정상 이미지와 작게 파손된 이미지(`broken_small`) 일부만 선택하여 사용**합니다.

오늘 실습에서 필요한 데이터는 크게 두 종류입니다.

```
Normal
→ 아무런 결함이 없는 정상 병 이미지

broken_small
→ 일부가 작게 파손된 병 이미지
```

이 두 종류의 이미지를 이용하여 One-shot, Few-shot, Defect Generation을 차례대로 실습합니다.

## 3.1 정상 이미지 — `good`

MVTec `bottle`에서 `good`은 **파손이나 오염이 없는 정상 병 이미지**입니다.

오늘 정상 이미지는 한 가지 목적으로만 사용하는 것이 아니라, 실험에 따라 역할을 나누어 사용합니다.

```text
정상 병 이미지
train/good
        ↓
 ┌───────────────┬────────────────┐
 ↓               ↓                ↓
One-shot       Few-shot        Cut-Paste
기준 만들기     정상 예제 만들기    정상 바탕 이미지
```

본 실습에서는 다음과 같이 사용합니다.

| 사용 위치                   |  수량 | 무엇을 위해 사용하나요?                  |
| ----------------------- | --: | ------------------------------ |
| One-shot Reference      |  1장 | 정상 기준이 되는 대표 이미지               |
| One-shot Calibration    | 20장 | 정상 이미지의 비교값 범위를 확인하고 판정 기준을 정함 |
| Few-shot Normal Support |  5장 | Few-shot에서 정상 그룹의 특징을 만드는 예제   |
| Cut-Paste Normal Target | 10장 | 결함을 붙여 합성 불량을 만들 정상 원본         |
One-shot과 Few-shot을 최종적으로 비교할 때는 `train/good`이 아니라 **처음 보는 정상 이미지**가 필요합니다.

그래서 `test/good`에서 정상 이미지 10장을 따로 사용합니다.

```
test/good
        ↓
정상 Query 10장
        ↓
One-shot과 Few-shot에
똑같이 입력하여 결과 비교
```

즉, **기준을 만드는 정상 이미지와 마지막에 검사하는 정상 이미지를 분리**하는 것입니다. 현재 교재도 `train/good`에서 기준용 데이터를 나누고 `test/good`에서 공통 정상 Query 10장을 사용하도록 구성되어 있습니다.

## 3.2 작게 파손된 이미지 — `broken_small`

`broken_small`은 **병의 일부가 작게 파손된 불량 이미지**입니다.

오늘은 이 데이터를 본수업의 대표 불량으로 사용합니다.

```text
broken_small
작게 파손된 병
        ↓
 ┌──────────────┬────────────────┐
 ↓              ↓                ↓
Few-shot      Query           Cut-Paste
불량 예제      평가용 불량       결함 원본
```

본 실습에서는 다음과 같이 나눕니다.

| 사용 위치                    |  수량 | 무엇을 위해 사용하나요?                              |
| ------------------------ | --: | ------------------------------------------ |
| Few-shot Anomaly Support |  5장 | Few-shot에서 불량 그룹의 특징을 만드는 예제               |
| Common Query Anomaly     | 10장 | One-shot과 Few-shot을 비교할 때 사용하는 불량 Test 이미지 |
| Cut-Paste Defect Source  |  2장 | 실제 결함 부분을 추출하여 합성 불량을 만드는 원본               |
즉, `broken_small` 하나를 가지고도 실습 목적에 따라 **기준을 만드는 이미지**, **평가하는 이미지**, **합성에 사용하는 이미지**로 역할을 나누어 사용합니다.

## 3.3 오늘 사용할 데이터 흐름을 한 번에 정리합니다

복잡하게 외울 필요는 없습니다.

오늘 본수업에서 사용하는 데이터의 핵심은 다음과 같습니다.

![[Pasted image 20260913140351.png]]

위의 이미지는 오늘 사용할 MVTec `bottle` 이미지를 **실험 목적에 따라 역할별로 나누는 구조**를 보여줍니다. 

정상 병 이미지인 `good` 데이터 중 일부는 One-shot의 정상 기준으로 사용하고, 일부는 Few-shot의 정상 예제로 사용합니다. 또 일부 정상 이미지는 모델이 처음 보는 **평가용 Query**로 남겨 두고, 다른 일부는 결함을 붙여 Synthetic Defect를 만들기 위한 **Cut-Paste 정상 바탕 이미지**로 사용합니다.

`broken_small`도 마찬가지입니다. 목적에 따라 서로 다른 이미지로 분리했다는 뜻입니다.

Query는 기준을 만든 뒤 **처음 보는 이미지처럼 넣어서 결과를 확인하는 평가용 데이터**입니다.

따라서 오늘 데이터 준비의 핵심은 같은 Dataset의 이미지를 기준을 만드는 데이터, 평가하는 데이터, 합성에 사용하는 데이터로 서로 겹치지 않게 나누어 사용하는 것입니다.

오늘 사용할 데이터는 이미 처음부터 상태별로 폴더가 나뉘어져 있는 것으로 실습을 하나 
실제 현업에서는 예를 들어 공장에서 직접 촬영한 원본 사진만 있다면 사람이 먼저 확인하여 정상인지, 작은 파손인지, 오염인지 라벨을 붙이거나 검수해야 합니다.

```
현업에서 직접 수집한 원본 이미지
        ↓
사람이 확인·라벨링
        ↓
정상 / 작은 파손 / 오염 등으로 정리
        ↓
Train / Validation / Test
또는
Support / Query 등으로 역할 분리
        ↓
모델 학습·검사·평가
```

즉 실제 현업에서는 직접 촬영한 데이터라면 이보다 앞 단계에서 사람이 정상·불량을 확인하고 라벨링하는 과정이 필요할 수 있습니다.


여기서 가장 중요한 것은 이미지 수를 외우는 것이 아닙니다.

> 같은 Dataset이라도 기준을 만드는 데이터와 최종적으로 검사하는 데이터를 분리해야 한다는 것을 이해하면 됩니다.

실제 폴더를 만드는 단계에서는 프로그램이 각 이미지를 어떤 역할로 나누는지 다시 확인합니다.

---

# 4. 오늘은 어디까지 직접 실습할까요?

앞에서 오늘 배울 방법의 큰 흐름을 확인했습니다.

여기서는 개념을 다시 설명하지 않고, **오늘 어떤 것은 직접 만들어 보고 어떤 것은 개념만 이해하면 되는지** 확인합니다.

## 4.1 Zero-shot은 개념만 이해합니다

Zero-shot은 **해당 불량 예제가 거의 없는 상황에서 어떤 방법을 생각해 볼 수 있는지** 알아보는 수준으로 진행합니다.

오늘 별도의 Zero-shot AI 모델을 새로 설치하거나 구현하지 않습니다.

> “예제가 하나도 없다면 어떤 방법을 선택할 수 있을까?”를 생각해 보는 단계입니다.

---

## 4.2 One-shot과 Few-shot은 직접 실습합니다

오늘의 주요 실습은 One-shot과 Few-shot입니다.

```
One-shot
→ 정상 이미지 1장을 기준으로 검사

Few-shot
→ 정상·불량 이미지 몇 장을 이용하여 검사
```

두 방법을 직접 실행한 뒤 같은 평가 이미지에서 결과가 어떻게 달라지는지 비교합니다.

세부 계산 방법과 새로운 용어는 해당 실습에 들어갔을 때 하나씩 확인합니다.

---

## 4.3 부족한 불량 데이터도 직접 만들어 봅니다

마지막에는 불량 데이터가 부족한 상황을 가정하여 간단한 합성 불량 이미지를 만들어 봅니다.

```
실제 불량 이미지에서
결함 부분 추출
        ↓
정상 이미지에 붙이기
        ↓
새로운 합성 불량 이미지
```

오늘은 이처럼 이해하기 쉬운 **Cut-Paste 방식**을 사용합니다.

VAE, GAN, Diffusion과 같은 생성모델을 이용한 본격적인 합성데이터 제작은 **교과 11에서 따로 학습**합니다.

---

## 오늘의 실습 범위를 한 문장으로 정리하면

> Zero-shot은 이런 방법도 있다는 것을 이해하고, One-shot·Few-shot·간단한 Defect Generation은 직접 실행해 보는 수업입니다.

이 정도만 알고 실제 실습으로 들어가면 됩니다.

---

# 각 파일의 역할 및 연결흐름

| 파일명                                    | 역할                      | 설명                                                                                           |
| -------------------------------------- | ----------------------- | -------------------------------------------------------------------------------------------- |
| `61_day07_mini_challenge.py`           | 7일차 Mini Challenge      | 정상 병 이미지의 촬영 조건을 변경하여 기존 이상검출 모델의 성능과 오탐 발생 원인을 분석하는 7일차 마무리 파일입니다.                          |
| `62_day08_check_lowdata_env.py`        | AI 환경·데이터셋 확인           | PyTorch·CUDA(GPU)·OpenCV 버전과 MVTec bottle 데이터의 폴더 및 실습에 필요한 이미지 수를 확인합니다.                    |
| `63_day08_prepare_dataset.py`          | 소량 데이터셋 준비              | 원본 이미지를 One-shot·Few-shot·Common Query·Cut-Paste·Mini Challenge용으로 중복 없이 분리합니다.              |
| `day08_feature_core.py`                | Feature 추출 공통 함수        | ResNet18로 이미지의 특징을 추출하고 Cosine Similarity를 계산하며 Few-shot의 Prototype을 만드는 공통 기능을 제공합니다.       |
| `64_day08_calibrate_one_shot.py`       | One-shot 판정 기준 생성       | 정상 Reference 1장과 Calibration 20장을 비교하여 Similarity Threshold를 계산하고 검사 Rule을 저장합니다.            |
| `65_day08_test_one_shot.py`            | One-shot 검사             | 확정한 Threshold를 사용하여 새로운 정상·불량 Query 20장을 검사하고 Normal / Anomaly 결과를 CSV로 저장합니다.               |
| `66_day08_build_fewshot_prototypes.py` | Few-shot 대표 기준 생성       | 정상 Support 5장과 불량 Support 5장의 특징을 각각 평균하여 Normal·Anomaly Prototype을 생성합니다.                   |
| `67_day08_test_fewshot.py`             | Few-shot 검사             | Query 이미지와 두 Prototype의 유사도를 비교하여 Normal / Anomaly를 판정하고 결과를 CSV로 저장합니다.                     |
| `68_day08_compare_methods.py`          | One-shot·Few-shot 성능 비교 | 동일한 Query에서 얻은 두 검사 결과를 비교하여 Accuracy·Precision·Recall·FP·FN 등의 차이를 분석합니다.                   |
| `day08_cutpaste_core.py`               | 불량 이미지 합성 공통 함수         | 실제 불량 이미지와 Ground Truth Mask에서 결함 부분을 추출하고 정상 이미지에 합성하는 Cut-Paste 기능을 제공합니다.                 |
| `69_day08_make_cutpaste_defects.py`    | Synthetic Defect 생성     | Cut-Paste 공통 함수를 사용하여 `broken_small` 결함이 포함된 합성 불량 이미지 20장과 생성 정보를 저장합니다.                    |
| `day08_support_count_challenge.py`     | Support 수 변경 실험         | Few-shot의 정상·불량 Support 수를 각각 1장·3장·5장으로 변경하여 동일한 Query에서 성능 차이를 비교하는 Challenge입니다.          |
| `day08_mini_challenge.py`              | 새로운 결함에 실습 적용           | `contamination` 결함으로 합성 불량 이미지를 만들고, 실험 결과와 데이터 조건에 따른 검사 방법 선택을 정리하는 Mini Challenge용 파일입니다. |

| 구분       | 해당 파일                              | 의미                                      |
| -------- | ---------------------------------- | --------------------------------------- |
| 공통 함수 모듈 | `day08_feature_core.py`            | 여러 One-shot·Few-shot 실행 파일이 가져다 사용하는 기능 |
| 공통 함수 모듈 | `day08_cutpaste_core.py`           | 합성 불량 생성과 Mini Challenge에서 재사용하는 기능     |
| 추가 실습 파일 | `day08_support_count_challenge.py` | 학생이 Support 수를 바꾸어 비교하는 실습              |
| 추가 실습 파일 | `day08_mini_challenge.py`          | 학생이 스스로 실습 과정을 응용하는 Challenge           |
![[Pasted image 20261020093654.png]]

![[Pasted image 20261020094015.png]]

---
# 5. 오늘 추가되는 프로젝트 구조

7일차 마지막 실행 파일은 다음이었습니다.

```text
src/61_day07_mini_challenge.py
```

따라서 8일차 실행 파일은 **62번부터 이어서** 작성합니다.

```text
subject9_machinevision/
│
├── samples/
│   ├── day07/
│   │   └── original/
│   │       └── bottle/             # 7일차에서 준비한 읽기 전용 원본
│   │
│   └── day08/
│       ├── one_shot/
│       │   ├── reference/
│       │   └── calibration/
│       │       └── normal/
│       │
│       ├── few_shot/
│       │   └── support/
│       │       ├── normal/
│       │       └── anomaly/
│       │
│       ├── common_query/
│       │   ├── normal/
│       │   └── anomaly/
│       │       └── broken_small/
│       │
│       ├── cutpaste/
│       │   ├── normal/
│       │   └── defect_source/
│       │       └── broken_small/
│       │           ├── images/
│       │           └── masks/
│       │
│       └── mini_challenge/
│           ├── normal/
│           └── defect_source/
│               └── contamination/
│                   ├── images/
│                   └── masks/
│
├── artifacts/
│   └── day08/
│       ├── one_shot_reference.npy
│       └── fewshot_prototypes.npz
│
├── configs/
│   ├── day08_one_shot_rules.json
│   └── day08_fewshot_config.json
│
├── outputs/
│   └── day08/
│       ├── cutpaste/
│       │   └── generated/
│       └── mini_challenge/
│           └── generated/
│
├── reports/
│   ├── day08_dataset_summary.md
│   ├── day08_one_shot_calibration.csv
│   ├── day08_one_shot_results.csv
│   ├── day08_fewshot_results.csv
│   ├── day08_method_comparison.csv
│   ├── day08_defect_metadata.csv
│   ├── day08_support_count_challenge.csv
│   ├── day08_synthetic_quality.md
│   ├── day08_mini_challenge_metadata.csv
│   ├── day08_mini_challenge.md
│   ├── day08_strategy_selection.md
│   └── day08_failure_analysis.md
│
└── src/
    ├── ...
    ├── 61_day07_mini_challenge.py
    ├── 62_day08_check_lowdata_env.py
    ├── 63_day08_prepare_dataset.py
    ├── day08_feature_core.py
    ├── 64_day08_calibrate_one_shot.py
    ├── 65_day08_test_one_shot.py
    ├── 66_day08_build_fewshot_prototypes.py
    ├── 67_day08_test_fewshot.py
    ├── 68_day08_compare_methods.py
    ├── day08_cutpaste_core.py
    ├── 69_day08_make_cutpaste_defects.py
    ├── day08_support_count_challenge.py
    └── day08_mini_challenge.py
```

번호가 붙은 8일차의 핵심 실행 파일은 `62`부터 `69`까지 이어집니다.  
`day08_support_count_challenge.py`와 `day08_mini_challenge.py`는 이미 만든 Day08 Feature·Cut-Paste 함수를 재사용하는 추가 실습 파일이므로 번호를 붙이지 않습니다. 이렇게 하면 다음 9일차 실행 파일을 `70`번부터 시작할 수 있습니다.

---

# 6. 오늘 데이터에서 지켜야 할 세 가지 원칙

지켜야할 3가지 원칙

```
① 원본 데이터는 보관하고 실습용 복사본을 사용한다.  
② 기준을 만드는 사진과 시험해 보는 사진은 서로 달라야 한다.  
③ One-shot과 Few-shot을 비교할 때는 똑같은 시험 사진을 사용한다.
```

### 실습 데이터를 왜 나누어 사용할까요?

앞에서 MVTec `bottle` 이미지를 실습 목적에 따라 여러 역할로 나누었습니다.

이번에는 **왜 굳이 데이터를 이렇게 나누어 사용하는지** 간단히 이해하고 넘어갑니다.

지금부터 설명하는 내용은 우리가 직접 파일을 하나씩 관리하라는 뜻이 아닙니다.  
뒤에서 작성할 데이터 준비 코드가 대부분 자동으로 처리합니다.

우리는 **왜 이렇게 나누는지만 이해하면 됩니다.**

## 6.1 원본 데이터는 그대로 보관합니다

7일차에서 준비한 MVTec 원본은 다음 위치에 있습니다.

```
samples/day07/original/bottle/
```

이 폴더의 이미지는 실습 중 직접 덮어쓰거나 다른 이미지로 바꾸지 않습니다.

예를 들어 뒤의 실습에서는 이미지를 복사하거나, 결함을 붙이거나, 새로운 합성 이미지를 만들게 됩니다.

그 결과를 원본 폴더에 다시 저장해 버리면:

```
처음 받은 원본 이미지
+
실습에서 만든 이미지
+
변형된 이미지
```

가 섞일 수 있습니다.

그러면 나중에 **어떤 것이 처음 받은 원본이었는지 알기 어려워집니다.**

따라서 다음처럼 사용합니다.

```
MVTec 원본
samples/day07/original/bottle/
        ↓
원본은 그대로 보관
        ↓
필요한 이미지를 복사
        ↓
samples/day08/
에서 실습
```

> 원본은 보관하고, 실습은 복사한 데이터로 진행합니다.

---

## 6.2 기준을 만드는 이미지와 시험하는 이미지는 다르게 사용합니다

오늘부터 자주 등장하는 두 용어가 있습니다.

```
Support
→ 검사 기준을 만드는 데 사용하는 예제

Query
→ 만든 기준이 잘 작동하는지 시험해 보는 이미지
```

쉽게 생각하면 다음과 같습니다.

```
Support
= 공부할 때 참고한 예제

Query
= 공부가 끝난 뒤 풀어보는 시험 문제
```

만약 같은 이미지를 Support와 Query에 모두 사용한다면 어떻게 될까요?

```
이미지 A로 기준을 만듦
        ↓
다시 이미지 A를 넣어서 평가
        ↓
이미 본 이미지로 시험하는 셈
```

이렇게 하면 결과가 실제보다 좋아 보일 수 있습니다.

그래서 오늘 데이터 준비 코드에서는 **Support로 사용한 이미지가 Query에 다시 들어가지 않도록 자동으로 분리**합니다.

우리는 사진을 하나씩 확인하며 직접 나눌 필요는 없습니다.

> **기준을 만들 때 사용한 이미지는 최종 시험에 다시 사용하지 않는다.**

이 원리만 이해하면 됩니다.

---

## 6.3 One-shot과 Few-shot은 같은 시험 이미지로 비교합니다

오늘은 One-shot과 Few-shot 두 방법을 비교합니다.

그런데 서로 다른 사진으로 시험하면 공정하게 비교하기 어렵습니다.

예를 들어:

```
One-shot
→ 쉬운 이미지로 시험

Few-shot
→ 어려운 이미지로 시험
```

이렇게 하면 결과가 다른 이유가 **방법 때문인지 사진 때문인지 알 수 없습니다.**

그래서 두 방법에 **똑같은 Query 이미지**를 넣습니다.

```
같은 정상·불량 Query
        ↓
   ┌───────────────┐
   ↓               ↓
One-shot        Few-shot
   ↓               ↓
결과 확인        결과 확인
   └───────┬───────┘
           ↓
       공정하게 비교
```

이때 두 방법에 공통으로 사용하는 평가 이미지를 오늘 수업에서는 **Common Query**라고 부릅니다.

쉽게 말하면:

> **One-shot과 Few-shot이 똑같은 시험 문제를 풀게 하는 것입니다.**

---

## 6.4 지금은 이것만 기억하면 됩니다

```
원본 이미지
→ 그대로 보관

기준을 만드는 이미지
→ Support

기준을 확인하는 시험 이미지
→ Query

One-shot과 Few-shot 비교
→ 같은 Query 사용
```

이 원칙들은 지금 외우기 위한 내용이 아닙니다.

뒤에서 실제 데이터를 나누고 One-shot과 Few-shot을 실행해 보면 왜 이렇게 구분했는지 
자연스럽게 확인하게 됩니다.

---

# PART1 — 실습 환경 + Day08 데이터 준비 (7~12)

---

# 7. 기존 프로젝트를 다시 엽니다

7일차까지 사용한 프로젝트를 그대로 이어서 사용합니다.

```bash
cd ~/ai_vision/subject9_machinevision
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

현재 Python을 확인합니다.

```bash
which python
```

VS Code를 엽니다.

```bash
code -r .
```

새 프로젝트나 새 `.venv`를 만들지 않습니다.

---

# 8. MVTec 원본이 그대로 있는지 확인합니다

7일차에서 이미 준비했으므로 다시 다운로드하지 않습니다.

```bash
ls samples/day07/original/bottle
```

다음 항목이 보이면 됩니다.

```text
train
test
ground_truth
```

세부 폴더도 확인합니다.

```bash
find samples/day07/original/bottle -maxdepth 2 -type d | sort
```

이 단계에서는 이미지를 다른 폴더로 수동 복사하지 않습니다.

다음 프로그램이 필요한 이미지만 자동으로 분리합니다.

---

# 9. `62_day08_check_lowdata_env.py` — 8일차 실습을 시작할 수 있는지 먼저 확인합니다

본격적인 One-shot과 Few-shot 실습에 들어가기 전에 **현재 컴퓨터 환경과 MVTec `bottle` 데이터가 제대로 준비되어 있는지 먼저 확인**합니다.

뒤의 실습에서는 여러 Python 라이브러리와 GPU를 사용하고, MVTec 데이터에서 필요한 이미지를 자동으로 나누어 사용합니다.

그런데 필요한 폴더가 없거나 이미지 수가 부족한 상태에서 바로 다음 실습으로 넘어가면 뒤에서 갑자기 오류가 발생할 수 있습니다.

그래서 먼저 다음 내용을 한 번에 확인하는 점검 프로그램을 작성합니다.

```
Python 실습 환경이 정상인가?
        +
MVTec bottle 원본이 있는가?
        +
오늘 사용할 폴더가 모두 있는가?
        +
실습에 필요한 이미지 수가 충분한가?
        ↓
모두 정상
        ↓
Day08 Environment OK
```

이 프로그램은 아직 One-shot이나 Few-shot을 실행하는 코드가 아닙니다.  
오늘 실습을 시작하기 전에 필요한 환경과 데이터를 점검하는 준비 코드입니다.

파일을 만듭니다

```text
src/62_day08_check_lowdata_env.py
```

다음 코드를 작성합니다.

```python
from pathlib import Path

import cv2
import numpy as np
import pandas as pd
import torch
import torchvision
import PIL
from PIL import Image


ROOT = Path("samples/day07/original/bottle")

required_dirs = [
    ROOT / "train/good",
    ROOT / "test/good",
    ROOT / "test/broken_small",
    ROOT / "test/contamination",
    ROOT / "ground_truth/broken_small",
    ROOT / "ground_truth/contamination",
]


print("=" * 70)
print("DAY08 LOW-DATA ENV CHECK")
print("=" * 70)

print("Torch      :", torch.__version__)
print("Torchvision:", torchvision.__version__)
print("OpenCV     :", cv2.__version__)
print("NumPy      :", np.__version__)
print("Pandas     :", pd.__version__)
print("Pillow     :", PIL.__version__)
print("CUDA       :", torch.cuda.is_available())

if torch.cuda.is_available():
    print("GPU        :", torch.cuda.get_device_name(0))

print()
print("Dataset    :", ROOT)

for directory in required_dirs:
    if not directory.exists():
        raise FileNotFoundError(directory)
    print("[OK]", directory)


counts = {
    "train_good": len(list((ROOT / "train/good").glob("*.png"))),
    "test_good": len(list((ROOT / "test/good").glob("*.png"))),
    "broken_small": len(list((ROOT / "test/broken_small").glob("*.png"))),
    "contamination": len(list((ROOT / "test/contamination").glob("*.png"))),
}

print()
for name, count in counts.items():
    print(f"{name:15s}: {count}")


minimum_required = {
    "train_good": 41,
    "test_good": 10,
    "broken_small": 17,
    "contamination": 2,
}

for name, minimum in minimum_required.items():
    if counts[name] < minimum:
        raise RuntimeError(
            f"{name} 이미지가 부족합니다. "
            f"필요: {minimum}, 현재: {counts[name]}"
        )

print()
print("Day08 Environment OK")
```

실행합니다.

```bash
python src/62_day08_check_lowdata_env.py
```

MVTec `bottle` 원본이 7일차와 동일하게 준비되어 있다면 이미지 수는 대략 다음과 같이 확인됩니다.

```text
train_good     : 209
test_good      : 20
broken_small   : 22
contamination  : 21

Day08 Environment OK
```

### 이 코드가 하는 일을 먼저 이해합니다

이 프로그램은 크게 네 가지를 확인합니다.

```
① 필요한 라이브러리 확인
        ↓
② GPU 사용 가능 여부 확인
        ↓
③ MVTec bottle 폴더 확인
        ↓
④ 이미지 수량 확인
        ↓
실습 가능 여부 판단
```

다음 부분은 오늘 실습을 진행하는 데 필요한 **최소 이미지 수**를 지정하는 코드 입니다.

```python
minimum_required = {
    "train_good": 41,
    "test_good": 10,
    "broken_small": 17,
    "contamination": 2,
}
```

숫자만 보면 갑자기 `41`, `17` 등이 등장하기 때문에 왜 필요한지 이해하는 것이 중요합니다.

오늘 계획된 데이터 역할을 합치면 다음과 같습니다.

#### `train_good` — 최소 41장

```
One-shot Reference          1장
One-shot Calibration       20장
Few-shot Normal Support     5장
Cut-Paste Normal Target    10장
Mini Challenge Normal       5장
                            ───
합계                        41장
```

따라서 정상 Train 이미지는 최소 41장이 필요합니다.

#### ### `test_good` — 최소 10장

```
Common Query Normal
10장
```

One-shot과 Few-shot을 같은 정상 이미지에서 비교하기 위한 평가 데이터입니다.

#### ### `broken_small` — 최소 17장

```
Few-shot Anomaly Support     5장
Common Query Anomaly        10장
Cut-Paste Defect Source      2장
                             ───
합계                         17장
```

따라서 `broken_small`은 최소 17장이 필요합니다.

#### ### `contamination` — 최소 2장

```
Mini Challenge
Defect Source
2장
```

`contamination`은 뒤의 Mini Challenge에서 사용할 결함 원본입니다.

#### 만약에 이미지가 부족하면 프로그램을 중단합니다.

다음 코드는 실제 이미지 수와 필요한 최소 이미지 수를 비교합니다.

```python
for name, minimum in minimum_required.items():
    if counts[name] < minimum:
        raise RuntimeError(
            f"{name} 이미지가 부족합니다. "
            f"필요: {minimum}, 현재: {counts[name]}"
        )
```

예를 들어 `broken_small`이 17장 필요한데 실제로 10장밖에 없다면:

```
필요: 17
현재: 10
```

이므로 프로그램을 중단합니다.

### 전체 코드의 흐름

```
프로그램 시작
        ↓
필요한 Python 라이브러리 불러오기
        ↓
MVTec bottle 원본 경로 지정
        ↓
오늘 필요한 Dataset 폴더 목록 지정
        ↓
라이브러리 버전 출력
        ↓
CUDA 사용 가능 여부 확인
        ↓
GPU 사용 가능하면 GPU 이름 출력
        ↓
필요한 Dataset 폴더 존재 여부 확인
        ↓
폴더가 하나라도 없음?
   ┌────┴────┐
  YES        NO
   ↓          ↓
오류 발생    계속 진행
             ↓
각 폴더의 PNG 이미지 수 계산
             ↓
현재 이미지 수 출력
             ↓
오늘 필요한 최소 이미지 수와 비교
             ↓
이미지가 부족한가?
   ┌─────────┴─────────┐
  YES                  NO
   ↓                    ↓
오류 발생        Day08 Environment OK
                        ↓
                  다음 실습 진행
```

### 의사코드

```text
1. 오늘 실습에 필요한 라이브러리를 불러온다.

2. 7일차에서 보존한
   MVTec bottle 원본 폴더를 ROOT로 지정한다.

3. 오늘 사용할 폴더를 목록으로 만든다.

   - 정상 Train
   - 정상 Test
   - broken_small
   - contamination
   - broken_small의 Ground Truth
   - contamination의 Ground Truth

4. 현재 설치된 라이브러리 버전을 출력한다.

   - PyTorch
   - Torchvision
   - OpenCV
   - NumPy
   - Pandas
   - Pillow

5. CUDA를 사용할 수 있는지 확인한다.

6. CUDA를 사용할 수 있다면
   현재 사용 가능한 GPU 이름도 출력한다.

7. 필요한 Dataset 폴더를 하나씩 확인한다.

8. 폴더가 존재하면 [OK]를 출력한다.

9. 필요한 폴더가 하나라도 없다면
   FileNotFoundError를 발생시키고 실행을 중단한다.

10. 각 데이터 폴더 안에 있는
    PNG 이미지 수를 센다.

11. 확인한 이미지 수를
    counts에 저장한다.

12. 각 Dataset의 현재 이미지 수를 출력한다.

13. 오늘 전체 실습에 필요한
    최소 이미지 수를 정의한다.

    train_good     → 41장
    test_good      → 10장
    broken_small   → 17장
    contamination  → 2장

14. 실제 이미지 수와
    최소 필요 이미지 수를 하나씩 비교한다.

15. 이미지 수가 부족하다면
    필요한 수량과 현재 수량을 출력하고
    RuntimeError를 발생시킨다.

16. 모든 폴더와 이미지 수가 정상이라면

    Day08 Environment OK

    를 출력한다.

17. 다음 데이터 분리 실습으로 진행한다.
```

---

# 10. 7일차의 ResNet18 Weight를 그대로 재사용합니다

7일차에서는 이미 학습되어 있는 **Pretrained ResNet18**을 사용했습니다.

이때 사용한 설정이 다음입니다.

```python
ResNet18_Weights.DEFAULT
```

여기서 `Weight`는 ResNet18이 많은 이미지를 미리 학습하면서 얻어 둔 **학습된 값**입니다.

쉽게 생각하면:

```
ResNet18
    +
미리 학습된 Weight
    ↓
이미지의 특징을 찾아낼 수 있는 상태
```

입니다.

7일차에서는 이 ResNet18을 이용하여 병 이미지에서 Feature를 추출하고, 정상 이미지들의 특징을 모아 정상 기준을 만들었습니다.

```
7일차

병 이미지
        ↓
Pretrained ResNet18
        ↓
Feature 추출
        ↓
정상 기준 생성
```

8일차에서도 같은 ResNet18과 같은 Weight를 그대로 사용합니다.

```
7일차에서 사용한
ResNet18_Weights.DEFAULT
        ↓
8일차에서도 그대로 사용
```

따라서 8일차를 위해 새로운 모델을 학습하거나 새로운 Weight를 따로 만들지 않습니다.

> 7일차와 8일차는 같은 ResNet18로 이미지의 특징을 추출하고, 그 특징을 사용하는 방법을 다르게 실습합니다.

---

# 11. `63_day08_prepare_dataset.py` — 실습용 데이터를 역할별로 나눕니다

앞에서 MVTec `bottle` 원본 데이터가 충분히 준비되어 있는지 확인했습니다.

이제 원본 이미지 중에서 오늘 실제로 사용할 이미지를 골라 **실습 목적에 맞는 폴더로 자동 분리**합니다.

이 프로그램은 아직 One-shot이나 Few-shot 검사를 실행하지 않습니다.

> 앞으로 사용할 이미지들을 미리 역할별로 나누어 준비하는 데이터 정리 프로그램입니다.

쉽게 생각하면 다음과 같습니다.

```
MVTec bottle 원본 이미지
        ↓
오늘 필요한 이미지 선택
        ↓
역할에 따라 서로 다르게 분리
        ↓
One-shot용
Few-shot용
평가용
Cut-Paste용
Mini Challenge용
        ↓
samples/day08에 복사
```

### 왜 이미지는 썩은 뒤 나눌까요?

원본 파일 이름 순서대로 앞에서부터 계속 고르면 특정한 순서가 그대로 반영될 수 있습니다.

그래서 먼저 이미지 순서를 섞은 뒤 필요한 수량을 가져옵니다.

```
원본 이미지 목록
        ↓
순서 섞기
        ↓
앞에서 필요한 수량만 선택
        ↓
역할별 분리
```

그런데 실행할 때마다 다른 이미지가 선택되면 학생마다 결과가 달라질 수 있습니다.

그래서 다음 값을 사용합니다.

```python
SEED = 42
```

`Seed`를 고정하면 이미지를 섞더라도 **매번 같은 순서로 섞이게 됩니다.**

즉:

```
학생 A 실행
        ↓
같은 이미지 선택

학생 B 실행
        ↓
같은 이미지 선택

다시 실행
        ↓
같은 이미지 선택
```

이 되도록 만드는 것입니다.

> Seed의 숫자 `42` 자체가 중요한 것은 아닙니다.  
> 같은 값을 사용하여 수업에서 같은 데이터가 선택되도록 하는 것이 중요합니다.


파일:

```text
src/63_day08_prepare_dataset.py
```

작성합니다.

```python
import random
import shutil
from pathlib import Path


RAW_ROOT = Path("samples/day07/original/bottle")
DAY08_ROOT = Path("samples/day08")
REPORT_PATH = Path("reports/day08_dataset_summary.md")

SEED = 42


# --------------------------------------------------
# 1. Day08 폴더를 새로 준비
# --------------------------------------------------
if DAY08_ROOT.exists():
    shutil.rmtree(DAY08_ROOT)


def copy_files(files, destination):
    destination.mkdir(parents=True, exist_ok=True)

    for path in files:
        shutil.copy2(path, destination / path.name)


def copy_defect_sources(files, defect_type, destination_root):
    image_dir = destination_root / defect_type / "images"
    mask_dir = destination_root / defect_type / "masks"

    image_dir.mkdir(parents=True, exist_ok=True)
    mask_dir.mkdir(parents=True, exist_ok=True)

    for image_path in files:
        mask_path = (
            RAW_ROOT
            / "ground_truth"
            / defect_type
            / f"{image_path.stem}_mask.png"
        )

        if not mask_path.exists():
            raise FileNotFoundError(mask_path)

        shutil.copy2(image_path, image_dir / image_path.name)
        shutil.copy2(mask_path, mask_dir / mask_path.name)


# --------------------------------------------------
# 2. 원본 목록
# --------------------------------------------------
normal_train = sorted((RAW_ROOT / "train/good").glob("*.png"))
normal_test = sorted((RAW_ROOT / "test/good").glob("*.png"))
broken_small = sorted((RAW_ROOT / "test/broken_small").glob("*.png"))
contamination = sorted((RAW_ROOT / "test/contamination").glob("*.png"))


rng_normal = random.Random(SEED)
rng_broken = random.Random(SEED + 1)
rng_contamination = random.Random(SEED + 2)

rng_normal.shuffle(normal_train)
rng_broken.shuffle(broken_small)
rng_contamination.shuffle(contamination)


# --------------------------------------------------
# 3. Normal 데이터 역할 분리
# --------------------------------------------------
one_reference = normal_train[0:1]
one_calibration = normal_train[1:21]
few_normal_support = normal_train[21:26]
cutpaste_normal = normal_train[26:36]
mini_normal = normal_train[36:41]

common_normal_query = normal_test[:10]


# --------------------------------------------------
# 4. broken_small 역할 분리
# --------------------------------------------------
few_anomaly_support = broken_small[0:5]
common_anomaly_query = broken_small[5:15]
cutpaste_defect_source = broken_small[15:17]


# --------------------------------------------------
# 5. contamination은 Mini Challenge용
# --------------------------------------------------
mini_defect_source = contamination[:2]


# --------------------------------------------------
# 6. 중복 검사
# --------------------------------------------------
normal_groups = [
    set(one_reference),
    set(one_calibration),
    set(few_normal_support),
    set(cutpaste_normal),
    set(mini_normal),
]

for i in range(len(normal_groups)):
    for j in range(i + 1, len(normal_groups)):
        if normal_groups[i] & normal_groups[j]:
            raise RuntimeError("Normal Train 역할 간 데이터가 겹칩니다.")

broken_groups = [
    set(few_anomaly_support),
    set(common_anomaly_query),
    set(cutpaste_defect_source),
]

for i in range(len(broken_groups)):
    for j in range(i + 1, len(broken_groups)):
        if broken_groups[i] & broken_groups[j]:
            raise RuntimeError("broken_small 역할 간 데이터가 겹칩니다.")


# --------------------------------------------------
# 7. 복사
# --------------------------------------------------
copy_files(
    one_reference,
    DAY08_ROOT / "one_shot/reference",
)

copy_files(
    one_calibration,
    DAY08_ROOT / "one_shot/calibration/normal",
)

copy_files(
    few_normal_support,
    DAY08_ROOT / "few_shot/support/normal",
)

copy_files(
    few_anomaly_support,
    DAY08_ROOT / "few_shot/support/anomaly",
)

copy_files(
    common_normal_query,
    DAY08_ROOT / "common_query/normal",
)

copy_files(
    common_anomaly_query,
    DAY08_ROOT / "common_query/anomaly/broken_small",
)

copy_files(
    cutpaste_normal,
    DAY08_ROOT / "cutpaste/normal",
)

copy_defect_sources(
    cutpaste_defect_source,
    "broken_small",
    DAY08_ROOT / "cutpaste/defect_source",
)

copy_files(
    mini_normal,
    DAY08_ROOT / "mini_challenge/normal",
)

copy_defect_sources(
    mini_defect_source,
    "contamination",
    DAY08_ROOT / "mini_challenge/defect_source",
)


# --------------------------------------------------
# 8. Summary 저장
# --------------------------------------------------
summary = f"""# Day08 Dataset Summary

Seed: {SEED}

## One-shot

- Reference: {len(one_reference)}
- Normal Calibration: {len(one_calibration)}

## Few-shot Support

- Normal: {len(few_normal_support)}
- Anomaly(broken_small): {len(few_anomaly_support)}

## Common Query

- Normal: {len(common_normal_query)}
- Anomaly(broken_small): {len(common_anomaly_query)}

## Cut-Paste

- Normal Target: {len(cutpaste_normal)}
- Defect Source(broken_small): {len(cutpaste_defect_source)}

## Mini Challenge

- Normal Target: {len(mini_normal)}
- Defect Source(contamination): {len(mini_defect_source)}

## Data Leakage Check

PASS
"""

REPORT_PATH.parent.mkdir(parents=True, exist_ok=True)
REPORT_PATH.write_text(summary, encoding="utf-8")


print("=" * 70)
print("DAY08 DATASET PREPARE")
print("=" * 70)
print("One-shot Reference        :", len(one_reference))
print("One-shot Calibration      :", len(one_calibration))
print("Few-shot Normal Support   :", len(few_normal_support))
print("Few-shot Anomaly Support  :", len(few_anomaly_support))
print("Common Query Normal       :", len(common_normal_query))
print("Common Query Anomaly      :", len(common_anomaly_query))
print("Cut-Paste Normal          :", len(cutpaste_normal))
print("Cut-Paste Defect Source   :", len(cutpaste_defect_source))
print("Mini Defect Source        :", len(mini_defect_source))
print("Data Leakage Check        : PASS")
print("Saved                     :", REPORT_PATH)
```

### 전체 코드 흐름

```
프로그램 시작
        ↓
MVTec bottle 원본 이미지 목록 읽기
        ↓
정상 / broken_small / contamination 준비
        ↓
Seed를 이용하여
항상 같은 순서로 이미지 섞기
        ↓
정상 train/good 분리
        ↓
One-shot Reference
One-shot Calibration
Few-shot Normal Support
Cut-Paste Normal
Mini Challenge Normal
        ↓
test/good에서
Common Query Normal 선택
        ↓
broken_small 분리
        ↓
Few-shot Anomaly Support
Common Query Anomaly
Cut-Paste Defect Source
        ↓
contamination에서
Mini Challenge Defect Source 선택
        ↓
역할별 이미지가 서로 겹치는지 검사
        ↓
겹침?
   ┌────┴────┐
  YES        NO
   ↓          ↓
실행 중단    계속 진행
             ↓
samples/day08에 역할별 복사
             ↓
결함 Source는
이미지 + Mask 함께 복사
             ↓
사용 수량을
day08_dataset_summary.md에 저장
             ↓
Data Leakage Check = PASS
             ↓
8일차 실습용 데이터 준비 완료
```

실행합니다.

```bash
python src/63_day08_prepare_dataset.py
```

정상적으로 실행되면 다음 수량을 확인합니다.

```text
One-shot Reference        : 1
One-shot Calibration      : 20
Few-shot Normal Support   : 5
Few-shot Anomaly Support  : 5
Common Query Normal       : 10
Common Query Anomaly      : 10
Cut-Paste Normal          : 10
Cut-Paste Defect Source   : 2
Mini Defect Source        : 2
Data Leakage Check        : PASS
```

여기에서 가장 중요한 것은 마지막입니다.

```
Data Leakage Check : PASS
```

이는 역할이 다른 데이터 사이에 잘못 겹친 이미지가 발견되지 않았다는 뜻입니다.

### 의사코드

```text
1. 7일차에서 보관한
   MVTec bottle 원본 위치를 지정한다.

2. 8일차 실습용 데이터를 저장할
   samples/day08 위치를 지정한다.

3. 이전에 만들어진 samples/day08이 있다면
   새로 분리하기 위해 초기화한다.

4. 원본에서 다음 이미지 목록을 읽는다.

   - train/good
   - test/good
   - test/broken_small
   - test/contamination

5. Seed 값을 고정한다.

6. 정상 이미지와 불량 이미지의 순서를 섞는다.

7. train/good 정상 이미지에서
   서로 다른 이미지를 다음 역할로 나눈다.

   - One-shot Reference 1장
   - One-shot Calibration 20장
   - Few-shot Normal Support 5장
   - Cut-Paste Normal 10장
   - Mini Challenge Normal 5장

8. test/good에서
   Common Query Normal 10장을 선택한다.

9. broken_small에서
   서로 다른 이미지를 다음 역할로 나눈다.

   - Few-shot Anomaly Support 5장
   - Common Query Anomaly 10장
   - Cut-Paste Defect Source 2장

10. contamination에서
    Mini Challenge Defect Source 2장을 선택한다.

11. 정상 Train에서 나눈 역할들 사이에
    같은 이미지가 중복되어 있는지 검사한다.

12. broken_small에서 나눈 역할들 사이에도
    같은 이미지가 중복되어 있는지 검사한다.

13. 중복이 발견되면
    프로그램을 중단한다.

14. 중복이 없다면
    선택한 이미지를 samples/day08의
    역할별 폴더에 복사한다.

15. Cut-Paste에 사용할 결함 이미지는
    해당 Ground Truth Mask도 함께 찾는다.

16. 결함 이미지와 Mask를
    각각 images / masks 폴더에 복사한다.

17. Mini Challenge용 contamination도
    이미지와 Mask를 함께 복사한다.

18. 각 역할에 몇 장이 배정되었는지 계산한다.

19. 결과를
    reports/day08_dataset_summary.md
    파일에 저장한다.

20. 모든 역할 분리가 정상적으로 끝나면

    Data Leakage Check = PASS

    를 출력한다.

21. 8일차 One-shot / Few-shot /
    Cut-Paste 실습에 사용할
    데이터 준비를 완료한다.
```

---

# 12. 준비된 Day08 Dataset을 직접 확인합니다

폴더를 확인합니다.

```bash
find samples/day08 -maxdepth 5 -type d | sort
```

파일 수도 확인합니다.

```bash
find samples/day08/one_shot/reference -type f | wc -l
find samples/day08/one_shot/calibration/normal -type f | wc -l
find samples/day08/few_shot/support/normal -type f | wc -l
find samples/day08/few_shot/support/anomaly -type f | wc -l
find samples/day08/common_query/normal -type f | wc -l
find samples/day08/common_query/anomaly/broken_small -type f | wc -l
```

다음 파일도 열어봅니다.

```bash
cat reports/day08_dataset_summary.md
```

여기까지 완료되면 **8일차의 데이터 역할이 고정된 상태**입니다.

이후 실습에서 결과가 마음에 들지 않는다고 Support나 Query 이미지를 임의로 바꾸지 않습니다.

---

# PART2 — Zero / One / Few-shot 개념 + 공통 Feature 함수 (13~18)

---

# 13. 오늘 실습에서 데이터 수에 따라 방법을 구분합니다

앞에서 오늘 사용할 방법의 큰 흐름을 확인했습니다.

여기서는 다시 이론을 길게 설명하지 않고, **데이터가 얼마나 있는지에 따라 어떤 이름을 사용하는지만 간단히 정리**합니다.

|방법|쉽게 이해하면|
|---|---|
|Zero-shot|사용할 예제가 없는 상황|
|One-shot|기준 예제가 1장인 상황|
|Few-shot|기준 예제가 몇 장 있는 상황|
|일반 학습|충분한 데이터를 가지고 학습하는 상황|

오늘 직접 실습하는 것은:

```
One-shot
   +
Few-shot
```

입니다.

Zero-shot은 앞에서 확인한 것처럼 **“예제가 하나도 없다면 어떤 방법을 고려할 수 있을까?”를 생각해 보는 수준**으로만 다룹니다.

---

# 14. Zero-shot은 여기서 더 깊게 다루지 않습니다

Zero-shot은 오늘 직접 코드를 작성하지 않습니다.

실제 현장에서는 **아직 한 번도 발생하지 않은 불량**을 검사해야 하는 상황이 생길 수 있습니다.

예를 들어 새 제품 생산을 시작했는데:

```
정상 제품 이미지는 있음

하지만
특정 불량 이미지는 아직 0장

예:
새로운 균열
처음 보는 오염
새로운 파손 형태
```

일 수 있습니다.

이처럼 해당 불량 예제가 하나도 없는 상황에서는 Zero-shot 같은 방법을 후보로 생각해 볼 수 있습니다.

```
해당 불량 예제 0장
        ↓
Zero-shot 같은 방법도 고려
```

오늘은 이 정도 개념만 이해하고, 실제 실습은 **One-shot과 Few-shot**에 집중합니다.

> Zero-shot의 세부 구현 방법은 오늘 수업 범위가 아닙니다.

---

# 15. One-shot은 어떤 상황에서 사용할까요?

실제 현장에서는 새로운 제품 검사를 빨리 시작해야 하지만, **기준으로 삼을 수 있는 확실한 정상 샘플이 한 장뿐인 상황**이 있을 수 있습니다.

예를 들어:

```
새 제품 생산 시작
        ↓
정상으로 확인된 대표 제품 1개 확보
        ↓
아직 많은 기준 데이터를 모으지 못함
        ↓
우선 이 한 장을 기준으로
새로운 제품과 비교해 볼 수 있을까?
```

이처럼 **대표 기준으로 사용할 이미지가 한 장뿐일 때** One-shot 방식을 생각해 볼 수 있습니다.

오늘 실습에서는:

```
정상 Reference 1장
        ↓
새 이미지와 비교
        ↓
얼마나 비슷한지 확인
```

하는 방식으로 진행합니다.

다만 실제 판정을 위해서는 정상 이미지가 어느 정도 달라질 수 있는지도 확인해야 하므로, 별도의 정상 이미지를 이용해 판정 기준을 조정합니다.

> 오늘 One-shot에서 핵심은 **“정상 기준을 대표하는 Reference는 1장”**이라는 점입니다.

뒤의 실습에서 이 한 장을 기준으로 실제 이미지를 어떻게 비교하는지 직접 확인합니다.

---

# 16. 이미지를 비교할 때 Similarity를 사용합니다

One-shot과 Few-shot에서는 이미지끼리 얼마나 비슷한지 비교해야 합니다.

오늘은 그 비교값을 `Similarity`라고 부릅니다.

```
Similarity 높음
→ 서로 비슷함

Similarity 낮음
→ 서로 다름
```

지금은 계산식을 외울 필요가 없습니다.

오늘 실습에서는 다음 정도만 이해하면 됩니다.

```
기준 이미지
        ↕
새 이미지
        ↓
Similarity 계산
        ↓
얼마나 비슷한지 확인
```

뒤에서 작성할 공통 함수가 이 값을 계산해 줍니다.

> 지금은 **“Similarity는 두 이미지의 특징이 얼마나 비슷한지 나타내는 값”**이라고 이해하면 충분합니다.

---

# 17. Few-shot은 어떤 상황에서 사용할까요?

제품 검사를 시작한 지 얼마 되지 않아 데이터가 많지는 않지만, **정상과 불량 이미지를 몇 장씩 확보한 상황**이 있을 수 있습니다.

예를 들어:

```
새 제품 생산 시작
        ↓
처음에는 정상 기준 이미지 1장만 확보
        ↓
One-shot으로 검사 시작
        ↓
시간이 지나면서 데이터가 조금 더 모임
        ↓
정상 이미지 몇 장
        +
실제 불량 이미지 몇 장 확보
```

이렇게 한 장보다는 데이터가 늘었지만, 아직 일반적인 모델을 충분히 학습할 만큼 많지는 않은 상황에서 Few-shot 방식을 생각해 볼 수 있습니다.

오늘은 다음과 같이 실습합니다.

```
정상 이미지 5장
        +
불량 이미지 5장
        ↓
각 그룹의 특징을 모아
대표 기준 만들기
        ↓
새 이미지 입력
        ↓
정상 쪽과 더 비슷한가?
불량 쪽과 더 비슷한가?
        ↓
Normal / Anomaly 판정
```

여기서 기준을 만드는 데 사용하는 몇 장의 예제를 `Support`라고 부릅니다.

```
Support
→ 기준을 만들기 위해 사용하는 예제 이미지
```

정상 Support 여러 장의 특징을 모아 만든 대표 기준을 `Normal Prototype`, 불량 Support 여러 장의 특징을 모아 만든 대표 기준을 `Anomaly Prototype`이라고 합니다.

처음 보는 평가 이미지는 `Query`라고 부릅니다.

```
Support
→ 기준을 만드는 이미지

Query
→ 만들어진 기준으로 검사해 보는 이미지
```

이때 처음 보는 평가 이미지를 `Query`라고 부릅니다.

지금은 용어를 외우기보다 다음 흐름만 이해하면 됩니다.

> Few-shot은 정상과 불량 예제가 몇 장씩 있을 때, 각 그룹의 특징을 대표하는 기준을 만든 뒤 새로운 이미지가 어느 쪽과 더 비슷한지 확인하는 방법입니다.


### One-shot과 Few-shot은 어떻게 비교하나요?

두 방법을 한꺼번에 하나의 코드에서 실행하는 것은 아닙니다.

오늘 수업에서는 순서대로 진행합니다.

```
먼저 One-shot 실습
        ↓
정상 Reference 1장으로 검사
        ↓
One-shot 결과 저장

        ↓

그다음 Few-shot 실습
        ↓
정상 5장 + 불량 5장으로 기준 생성
        ↓
Few-shot 결과 저장

        ↓

마지막 비교
        ↓
같은 평가 이미지에서
두 결과 비교
```

즉,

```
One-shot
→ 별도로 실행

Few-shot
→ 별도로 실행

마지막
→ 같은 Query 결과를 비교
```

합니다.

이렇게 해야 한 장만 사용했을 때와 몇 장의 예제를 사용했을 때 결과가 어떻게 달라지는지 
공정하게 확인할 수 있습니다.

다음 단계에서는 먼저 One-shot과 Few-shot에서 공통으로 사용할 이미지 특징 추출 기능을 만들고, 이후 두 방법을 하나씩 실제로 실행합니다.

---

# 18. `day08_feature_core.py` — One-shot과 Few-shot이 함께 사용할 공통 기능을 만듭니다

앞에서 One-shot과 Few-shot은 서로 다른 방식이라고 배웠습니다.

```
One-shot
→ 정상 기준 이미지 1장과 비교

Few-shot
→ 정상·불량 예제 몇 장으로
   대표 기준을 만든 뒤 비교
```

판정 방법은 다르지만, 두 실습 모두 처음에는 똑같은 작업이 필요합니다.

```
이미지 읽기
        ↓
ResNet18에 넣기
        ↓
이미지를 숫자 특징으로 바꾸기
        ↓
다른 특징과 얼마나 비슷한지 비교
```

즉 One-shot과 Few-shot 모두 이미지를 같은 방식으로 Feature로 바꾸고, Feature끼리 비교해야 합니다.

---

## 왜 공통 파일을 만들까요?

만약 One-shot 코드와 Few-shot 코드마다 같은 기능을 따로 작성한다면 다음처럼 됩니다.

```
One-shot 코드
→ ResNet18 준비
→ 이미지 전처리
→ Feature 추출
→ Similarity 계산


Few-shot 코드
→ ResNet18 준비
→ 이미지 전처리
→ Feature 추출
→ Similarity 계산
```

같은 코드가 계속 반복됩니다.

그런데 복사해서 여러 파일에 넣으면 나중에 한쪽만 수정되거나 설정이 조금 달라질 수도 있습니다.

```
One-shot
→ 전처리 A

Few-shot
→ 실수로 전처리 B

        ↓

두 방법의 결과를
공정하게 비교하기 어려움
```

그래서 공통으로 필요한 기능을 한 파일에 만들어 두고, 뒤의 여러 프로그램에서 불러서 사용합니다.

```
             day08_feature_core.py
                       ↓
        ┌──────────────┴──────────────┐
        ↓                             ↓
     One-shot                     Few-shot
        ↓                             ↓
같은 Feature 추출 방식          같은 Feature 추출 방식
같은 Similarity 계산            같은 Similarity 계산
```

> 여러 프로그램에서 똑같이 필요한 기능이 있다면 한 번만 만들어 두고 다시 사용하는 것이 좋습니다.

이런 파일을 오늘은 **공통 기능 파일**이라고 생각하면 됩니다.

---

## 이 파일에서 준비하는 기능

이 파일에서는 크게 다음 기능을 만듭니다.

```
① ResNet18 준비

② 이미지 → Feature 변환

③ Feature끼리 Similarity 계산

④ 여러 Feature의 평균 기준 만들기
```

이 중에서:

```
One-shot과 Few-shot 공통
→ ResNet18 준비
→ Feature 추출
→ Similarity 계산

Few-shot에서 추가 사용
→ 여러 Feature 평균
→ Prototype 생성
```

입니다.

즉 `build_prototype()`까지 모든 기능이 완전히 공통인 것은 아닙니다.

Few-shot에서 사용할 기능도 같은 Feature 관련 기능이므로 이 파일에 함께 모아 둡니다.

---

## 이 파일은 직접 검사하는 프로그램이 아닙니다

중요한 점이 있습니다.

`day08_feature_core.py`를 만든다고 바로:

```
Normal
Anomaly
```

결과가 나오는 것은 아닙니다.

이 파일은 뒤의 프로그램에서 사용할 **도구들을 준비하는 파일**입니다.

```
day08_feature_core.py
        ↓
공통 기능 준비
        ↓
64_day08_calibrate_one_shot.py
65_day08_test_one_shot.py
66_day08_build_fewshot_prototypes.py
67_day08_test_fewshot.py
        ↓
실제 One-shot / Few-shot 실습
```

쉽게 비유하면:

> 지금은 검사를 실행하는 것이 아니라, 검사할 때 계속 사용할 도구를 먼저 만들어 두는 단계입니다.

---

## 이 파일이 처리하는 전체 흐름

한 장의 이미지가 들어오면 다음 순서로 처리합니다.

```
이미지 파일
        ↓
이미지 열기
        ↓
ResNet18이 사용할 수 있는 형태로 변환
        ↓
Pretrained ResNet18
        ↓
Feature 추출
        ↓
512개의 숫자로 표현
        ↓
비교하기 좋은 형태로 정리
        ↓
Feature 반환
```

이렇게 만들어진 Feature는 뒤에서 두 가지 방식으로 사용됩니다.

```
                Feature
                   ↓
        ┌──────────┴──────────┐
        ↓                     ↓
     One-shot              Few-shot
        ↓                     ↓
Reference와 비교       여러 Support Feature
        ↓                     ↓
Similarity             평균하여 Prototype
                              ↓
                       Query와 Similarity 비교
```

파일:

```text
src/day08_feature_core.py
```

작성합니다.

```python
import numpy as np
import torch
import torch.nn.functional as F
from PIL import Image
from torchvision import transforms
from torchvision.models import resnet18, ResNet18_Weights


DEVICE = torch.device(
    "cuda" if torch.cuda.is_available() else "cpu"
)


PREPROCESS = transforms.Compose(
    [
        transforms.Resize((224, 224)),
        transforms.ToTensor(),
        transforms.Normalize(
            mean=[0.485, 0.456, 0.406],
            std=[0.229, 0.224, 0.225],
        ),
    ]
)


def create_feature_model():
    model = resnet18(
        weights=ResNet18_Weights.DEFAULT
    )

    feature_model = torch.nn.Sequential(
        *list(model.children())[:-1]
    )

    feature_model.eval()
    feature_model.to(DEVICE)

    return feature_model


@torch.no_grad()
def extract_feature(model, image_path):
    image = Image.open(image_path).convert("RGB")

    tensor = (
        PREPROCESS(image)
        .unsqueeze(0)
        .to(DEVICE)
    )

    feature = model(tensor).flatten(1)

    feature = F.normalize(
        feature,
        p=2,
        dim=1,
    )

    return feature[0].cpu().numpy()


def cosine_similarity(feature_a, feature_b):
    return float(
        np.dot(feature_a, feature_b)
    )


def build_prototype(features):
    stacked = np.stack(features)
    prototype = stacked.mean(axis=0)

    norm = np.linalg.norm(prototype)

    if norm == 0:
        raise RuntimeError("Prototype norm이 0입니다.")

    return prototype / norm
```

## 코드가 만든 기능을 확인합니다

### `create_feature_model()`

```
Pretrained ResNet18 준비
        ↓
최종 분류 결과는 사용하지 않음
        ↓
이미지 특징을 꺼내는 용도로 사용
```

뒤의 One-shot과 Few-shot이 모두 이 모델을 사용합니다.

---

### `extract_feature()`

이미지 한 장을 받아 Feature로 바꿉니다.

```
이미지
        ↓
크기와 형식 정리
        ↓
ResNet18
        ↓
Feature
        ↓
512개의 숫자
```

즉 컴퓨터가 이미지끼리 비교할 수 있도록 **이미지를 숫자로 바꾸어 주는 기능**입니다.

---

### `cosine_similarity()`

Feature 두 개가 얼마나 비슷한지 계산합니다.

```
Feature A
        +
Feature B
        ↓
Similarity
```

오늘은 다음 방향만 기억합니다.

```
Similarity 높음
→ 서로 비슷함

Similarity 낮음
→ 서로 다름
```

One-shot과 Few-shot 모두 이 비교 기능을 사용합니다.

---

### `build_prototype()`

이 기능은 주로 Few-shot에서 사용합니다.

```
Feature 1
Feature 2
Feature 3
Feature 4
Feature 5
        ↓
평균
        ↓
대표 Feature
```

여러 예제의 특징을 평균하여 하나의 대표 기준을 만듭니다.

이 값을 뒤에서 `Prototype`이라고 사용합니다.

### 전체 코드 흐름

```
프로그램에서 공통 함수 불러오기
        ↓
사용할 장치 결정
GPU 또는 CPU
        ↓
이미지 전처리 방법 준비
        ↓
Pretrained ResNet18 준비
        ↓
이미지 한 장 입력
        ↓
이미지를 224 × 224 크기로 정리
        ↓
ResNet18 입력 형태로 변환
        ↓
ResNet18에 이미지 입력
        ↓
512-D Feature 추출
        ↓
Feature 정규화
        ↓
Feature 반환
        ↓
   ┌───────────────┬─────────────────┐
   ↓               ↓                 ↓
One-shot        Few-shot          Few-shot
Reference 비교   Query 비교       Support 평균
   ↓               ↓                 ↓
Similarity      Similarity         Prototype
```


### 의사코드

```
1. NumPy, PyTorch, Pillow,
   Torchvision 등 필요한 라이브러리를 불러온다.

2. 현재 컴퓨터에서 CUDA를 사용할 수 있는지 확인한다.

3. CUDA를 사용할 수 있다면 GPU를 사용하고,
   사용할 수 없다면 CPU를 사용하도록 설정한다.

4. ResNet18에 이미지를 넣기 전에 사용할
   공통 이미지 전처리 방법을 준비한다.

5. 이미지 크기를 224 × 224로 맞춘다.

6. 이미지를 PyTorch가 처리할 수 있는
   Tensor 형태로 바꾼다.

7. ResNet18이 사용하던 기준에 맞게
   이미지 값을 정리한다.

8. ImageNet으로 미리 학습된
   ResNet18을 불러온다.

9. ResNet18의 마지막 분류 결과는 사용하지 않고
   이미지 특징을 얻을 수 있는 부분까지 사용한다.

10. 만든 ResNet18을
    Feature 추출용 모델로 준비한다.

11. 이미지 경로가 들어오면
    해당 이미지를 RGB로 읽는다.

12. 앞에서 만든 공통 전처리를 적용한다.

13. 이미지를 ResNet18에 넣는다.

14. ResNet18이 만든 Feature를
    한 줄의 숫자 형태로 정리한다.

15. Feature를 서로 비교하기 좋은 형태로
    정규화한다.

16. 최종 Feature를 NumPy 배열로 반환한다.

17. Feature 두 개가 들어오면
    Cosine Similarity를 계산한다.

18. Similarity 값으로
    두 Feature가 얼마나 비슷한지 비교할 수 있게 한다.

19. 여러 개의 Feature가 들어오면
    하나의 배열로 모은다.

20. 여러 Feature의 평균을 계산한다.

21. 평균 Feature를 다시 정규화한다.

22. Few-shot에서 사용할
    Prototype을 반환한다.

23. 뒤의 One-shot 프로그램에서는

    Feature 추출
        +
    Similarity 계산

    기능을 가져다 사용한다.

24. 뒤의 Few-shot 프로그램에서는

    Feature 추출
        +
    Prototype 생성
        +
    Similarity 계산

    기능을 가져다 사용한다.
```


### 지금 단계에서 꼭 이해할 것

지금 가장 중요한 것은 다음 구조입니다.

```
이미지
        ↓
공통 함수
        ↓
Feature
```

그리고 이 Feature를:

```
One-shot
→ 한 장의 기준 Feature와 비교

Few-shot
→ 여러 Feature로 대표 기준을 만든 뒤 비교
```

한다는 차이입니다.

> One-shot과 Few-shot은 판정 방식은 다르지만, 이미지를 Feature로 바꾸는 과정은 같기 때문에 공통 함수로 만들어 재사용합니다.

---

# PART3 — One-shot: Reference 1장으로 검사하기 (19~25)

---

# 19. One-shot 실험 구조를 다시 확인합니다

오늘 One-shot 데이터는 다음 세 역할로 나뉩니다.

```text
Reference
정상 1장
        ↓
정상 대표 Feature

Calibration
정상 20장
        ↓
정상 Similarity 분포
        ↓
Threshold

Common Query
Normal 10장 + Anomaly 10장
        ↓
최종 확인
```

Threshold는 Query를 보기 전에 정합니다.

---

# 20. `64_day08_calibrate_one_shot.py` — Reference를 기준으로 판정 기준을 만듭니다

앞에서 One-shot에서는 **정상 Reference 1장**을 기준으로 새로운 이미지를 비교한다고 배웠습니다.

하지만 Reference 한 장만 있다고 바로:

```
Normal
또는
Anomaly
```

를 결정할 수 있는 것은 아닙니다.

정상 이미지도 촬영 상태나 위치가 조금씩 다르기 때문에 Reference와 완전히 똑같지는 않습니다.

따라서 먼저 정상 이미지들을 Reference와 비교하여:

> **정상이라면 Reference와 어느 정도까지 비슷하게 나오는가?**

를 확인해야 합니다.

오늘 준비한 데이터는 다음과 같습니다.

```
Reference
정상 1장

        +

Calibration
정상 20장
```

이번 프로그램은 이 두 데이터를 이용하여 **One-shot에서 사용할 판정 기준값을 만드는 코드**입니다.

---

## 이 코드에서는 무엇을 하나요?

전체적으로 다음 작업을 합니다.

```
정상 Reference 1장
        ↓
Feature 추출
        ↓
One-shot의 기준 Feature 저장

        +

정상 Calibration 20장
        ↓
각 이미지의 Feature 추출
        ↓
Reference와 Similarity 계산
        ↓
정상 이미지의 Similarity 범위 확인
        ↓
5 Percentile 계산
        ↓
Threshold 결정
        ↓
One-shot 판정 Rule 저장
```

여기서 중요한 것은 **아직 Common Query를 검사하지 않는다는 것**입니다.

지금은:

```
기준 만들기
```

단계입니다.

다음 실습에서 처음 보는 Common Query 이미지를 넣어 이 기준이 실제로 잘 작동하는지 확인합니다.

---

## 왜 Calibration 이미지가 필요한가요?

Reference는 한 장뿐입니다.

예를 들어 Reference와 다른 정상 이미지를 비교했을 때:

```
0.99
0.98
0.97
...
```

처럼 조금씩 다른 Similarity가 나올 수 있습니다.

정상이라고 해서 항상:

```
Similarity = 1.0
```

이 되는 것은 아닙니다.

그래서 정상 Calibration 20장을 이용해 먼저:

```
정상 이미지는
Reference와 비교했을 때

Similarity가
대략 어느 범위까지 나오는가?
```

를 확인합니다.

그리고 그 결과를 이용해:

```
여기 이상이면 Normal로 보자
```

라는 기준을 정합니다.

그 기준값이 오늘의 `Threshold`입니다.

---

## 이번 코드에서 만들어지는 결과

프로그램이 끝나면 세 종류의 결과가 남습니다.

```
Reference Feature
→ artifacts/day08/one_shot_reference.npy

Calibration Similarity 결과
→ reports/day08_one_shot_calibration.csv

One-shot 판정 기준
→ configs/day08_one_shot_rules.json
```

쉽게 보면:

```
npy
→ 기준 이미지의 숫자 특징 저장

csv
→ 정상 20장의 Similarity 기록

json
→ 앞으로 사용할 Threshold와 판정 Rule 저장
```

입니다.

---

파일을 만듭니다

```text
src/64_day08_calibrate_one_shot.py
```

작성합니다.

```python
import csv
import json
from pathlib import Path

import numpy as np

from day08_feature_core import (
    create_feature_model,
    cosine_similarity,
    extract_feature,
)


REFERENCE_DIR = Path("samples/day08/one_shot/reference")
CALIBRATION_DIR = Path("samples/day08/one_shot/calibration/normal")

ARTIFACT_PATH = Path("artifacts/day08/one_shot_reference.npy")
RULE_PATH = Path("configs/day08_one_shot_rules.json")
CSV_PATH = Path("reports/day08_one_shot_calibration.csv")

PERCENTILE = 5


reference_files = sorted(REFERENCE_DIR.glob("*.png"))

if len(reference_files) != 1:
    raise RuntimeError(
        f"Reference는 1장이어야 합니다. 현재: {len(reference_files)}"
    )

reference_path = reference_files[0]
calibration_files = sorted(CALIBRATION_DIR.glob("*.png"))

if not calibration_files:
    raise RuntimeError("Calibration 이미지가 없습니다.")


model = create_feature_model()
reference_feature = extract_feature(model, reference_path)

ARTIFACT_PATH.parent.mkdir(parents=True, exist_ok=True)
np.save(ARTIFACT_PATH, reference_feature)


rows = []
similarities = []

for image_path in calibration_files:
    feature = extract_feature(model, image_path)
    similarity = cosine_similarity(
        reference_feature,
        feature,
    )

    similarities.append(similarity)

    rows.append(
        {
            "file": image_path.name,
            "similarity": round(similarity, 6),
        }
    )


threshold = float(
    np.percentile(
        similarities,
        PERCENTILE,
    )
)


CSV_PATH.parent.mkdir(parents=True, exist_ok=True)

with CSV_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:
    writer = csv.DictWriter(
        file,
        fieldnames=["file", "similarity"],
    )
    writer.writeheader()
    writer.writerows(rows)


rules = {
    "baseline_version": "day08-one-shot-v1.0",
    "dataset": "MVTec AD bottle",
    "method": "one_reference_cosine_similarity",
    "backbone": "resnet18_imagenet_pretrained",
    "reference_count": 1,
    "reference_file": reference_path.name,
    "reference_path": str(ARTIFACT_PATH),
    "calibration_count": len(calibration_files),
    "threshold_method": "normal_calibration_5_percentile",
    "similarity_threshold": threshold,
    "decision_rule": "similarity >= threshold => Normal",
}

RULE_PATH.parent.mkdir(parents=True, exist_ok=True)
RULE_PATH.write_text(
    json.dumps(
        rules,
        indent=2,
        ensure_ascii=False,
    ),
    encoding="utf-8",
)


print("=" * 70)
print("DAY08 ONE-SHOT CALIBRATION")
print("=" * 70)
print("Reference          :", reference_path.name)
print("Calibration Images :", len(calibration_files))
print("Similarity Min     :", round(min(similarities), 6))
print("Similarity Mean    :", round(float(np.mean(similarities)), 6))
print("Similarity Max     :", round(max(similarities), 6))
print("5 Percentile       :", round(threshold, 6))
print("Rule               : similarity >= threshold => Normal")
print("Saved Rule         :", RULE_PATH)
```

실행합니다.

```bash
python src/64_day08_calibrate_one_shot.py
```

이번 실제 실행 결과는 다음과 같습니다.

```text
Reference          : 037.png
Calibration Images : 20
Similarity Min     : 0.98001
Similarity Mean    : 0.987521
Similarity Max     : 0.992414
5 Percentile       : 0.982113
Rule               : similarity >= threshold => Normal
Saved Rule         : configs/day08_one_shot_rules.json
```

### 실행 결과를 읽어봅니다

먼저

```
Reference : 037.png
```

는 오늘 One-shot에서 **정상 기준 이미지로 `037.png` 한 장을 사용했다**는 뜻입니다.

```
Calibration Images : 20
```

은 이 Reference와 비교하기 위해 별도로 준비한 정상 이미지 20장을 모두 사용했다는 뜻입니다.

## 정상 이미지의 Similarity는 어느 정도였나요?

정상 Calibration 20장을 Reference와 비교한 결과:

```
가장 낮은 Similarity
→ 0.980010

평균 Similarity
→ 0.987521

가장 높은 Similarity
→ 0.992414
```

가 나왔습니다.

즉 현재 정상 이미지들은 대체로:

```
0.98 ~ 0.99 정도
```

의 높은 Similarity를 보였습니다.

쉽게 해석하면:

> **Reference인 `037.png`와 다른 정상 이미지들의 Feature가 전체적으로 상당히 비슷하게 나타났습니다.**

하지만 정상 이미지끼리도 완전히 같은 값은 나오지 않았습니다.

```
정상 이미지 A
→ 0.980...

정상 이미지 B
→ 0.987...

정상 이미지 C
→ 0.992...
```

처럼 차이가 있습니다.

이것이 Calibration을 먼저 하는 이유입니다.

---

## 이번 Threshold는 `0.982113`입니다

정상 Calibration 결과의 낮은 쪽을 기준으로 `5 Percentile`을 계산한 결과:

```
Threshold
= 0.982113
```

이 나왔습니다.

따라서 앞으로 사용할 One-shot 판정 Rule은:

```
Similarity >= 0.982113
→ Normal

Similarity < 0.982113
→ Anomaly
```

가 됩니다.

쉽게 말하면:

```
새 이미지
        ↓
Reference 037.png와 비교
        ↓
Similarity 계산
        ↓

0.982113 이상
→ Reference와 충분히 비슷함
→ Normal

0.982113 미만
→ Reference와 상대적으로 덜 비슷함
→ Anomaly
```

로 판단합니다.

---

## 이번 One-shot 판정 기준을 확인합니다

정상 Calibration 20장의 Similarity를 계산한 결과:

```
Similarity Min     : 0.980010
Similarity Mean    : 0.987521
Similarity Max     : 0.992414
5 Percentile       : 0.982113
```

이 나왔습니다.

이번 실습에서는 계산된 `5 Percentile` 값을 One-shot의 Threshold로 사용합니다.

```
Threshold = 0.982113
```

따라서 판정 조건은 다음과 같습니다.

```
Similarity >= 0.982113
→ Normal

Similarity < 0.982113
→ Anomaly
```

즉 `64_day08_calibrate_one_shot.py`를 실행하여 이번 One-shot 검사에서 사용할 판정 기준 `0.982113`을 만들었습니다.

이 값은 다음 파일에도 저장됩니다.

```
configs/day08_one_shot_rules.json
```

> 왜 가장 낮은 Similarity `0.980010`을 그대로 사용하지 않고 `5 Percentile = 0.982113`을 Threshold로 사용하는지는 다음 21단락에서 확인합니다.

---

# 이 코드가 처리한 전체 흐름

```
프로그램 시작
        ↓
Reference 폴더 확인
        ↓
Reference가 정확히 1장인가?
   ┌────┴────┐
  NO         YES
   ↓          ↓
오류 발생   계속 진행
              ↓
Reference = 037.png 선택
              ↓
Pretrained ResNet18 준비
              ↓
Reference 이미지 Feature 추출
              ↓
one_shot_reference.npy 저장
              ↓
Calibration 정상 이미지 20장 읽기
              ↓
각 이미지 Feature 추출
              ↓
각 Feature를
Reference Feature와 비교
              ↓
Similarity 20개 생성
              ↓
Min / Mean / Max 확인
              ↓
Similarity 20개에서
5 Percentile 계산
              ↓
Threshold = 0.982113
              ↓
판정 Rule 결정
              ↓
Similarity >= Threshold
→ Normal
              ↓
Calibration 결과 CSV 저장
              ↓
Threshold와 Rule JSON 저장
              ↓
One-shot 기준 준비 완료
              ↓
다음 단계
Common Query Test
```

---

# 의사코드

```
1. One-shot Reference 이미지가 들어 있는
   폴더 위치를 지정한다.

2. 정상 Calibration 이미지가 들어 있는
   폴더 위치를 지정한다.

3. Reference Feature,
   Calibration 결과,
   판정 Rule을 저장할 위치를 지정한다.

4. Reference 폴더의 PNG 이미지를 확인한다.

5. Reference가 정확히 1장이 아니라면
   One-shot 조건과 맞지 않으므로
   실행을 중단한다.

6. Reference 이미지 1장을 선택한다.

7. Calibration 폴더의
   정상 이미지 목록을 읽는다.

8. Calibration 이미지가 없다면
   판정 기준을 만들 수 없으므로
   실행을 중단한다.

9. 앞에서 만든 공통 함수를 이용하여
   Pretrained ResNet18 Feature 모델을 준비한다.

10. Reference 이미지를
    ResNet18 Feature로 변환한다.

11. 만들어진 Reference Feature를
    one_shot_reference.npy에 저장한다.

12. Calibration 정상 이미지 20장을
    한 장씩 읽는다.

13. 각 Calibration 이미지를
    같은 ResNet18을 이용하여
    Feature로 변환한다.

14. 각 Calibration Feature와
    Reference Feature의
    Cosine Similarity를 계산한다.

15. 계산된 Similarity 20개를
    목록에 모은다.

16. 각 이미지의 파일명과 Similarity를
    CSV에 기록할 수 있도록 정리한다.

17. 정상 Similarity 20개의
    최소값을 확인한다.

18. 정상 Similarity 20개의
    평균값을 확인한다.

19. 정상 Similarity 20개의
    최대값을 확인한다.

20. 정상 Similarity의
    5 Percentile 값을 계산한다.

21. 계산된 5 Percentile을
    One-shot의 Similarity Threshold로 사용한다.

22. 판정 조건을 다음과 같이 정한다.

    Similarity >= Threshold
    → Normal

    Similarity < Threshold
    → Anomaly

23. Calibration 이미지 20장의
    Similarity 결과를

    reports/day08_one_shot_calibration.csv

    에 저장한다.

24. Reference 파일,
    사용한 Calibration 수,
    Threshold,
    판정 조건 등의 정보를

    configs/day08_one_shot_rules.json

    에 저장한다.

25. 이렇게 정한 Threshold를
    다음 Common Query Test에서
    다시 계산하거나 임의로 바꾸지 않는다.

26. 다음 실습에서는 처음 보는
    Normal / Anomaly Query 이미지를 넣어
    이 Rule이 실제로 얼마나 잘 작동하는지 확인한다.
```

---

## 지금 단계에서 꼭 이해할 것

이번 코드에서 **최종 One-shot 성능을 확인한 것은 아닙니다.**

지금까지 한 것은:

```
Reference 1장
        +
정상 Calibration 20장
        ↓
Similarity 확인
        ↓
Threshold = 0.982113
        ↓
One-shot Rule 결정
```

까지입니다.

다음 단계에서 비로소:

```
처음 보는 Common Query
Normal 10장 + Anomaly 10장
        ↓
방금 만든 Rule 적용
        ↓
실제 One-shot 검사 결과 확인
```

으로 넘어갑니다.

> 즉 단락 20은 One-shot으로 검사한 단계가 아니라, 
> One-shot 검사를 하기 전에 Reference와 정상 Calibration을 이용해 판정 기준을 먼저 만든 단계입니다.
> 
---

# 21. 왜 정상 Similarity의 5 Percentile을 사용하나요?

20번 실습에서 정상 Calibration 20장을 Reference와 비교하여 정상 이미지도 Similarity가 모두 똑같지는 않다는 것을 확인했습니다.

이번 실험에서는 이 정상 Similarity 중 **낮은 쪽 5% 지점**을 판정 기준으로 사용합니다.

```text
정상 Calibration Similarity
        ↓
높은 값들
        ↓
대부분의 정상 이미지
        ↓
낮은 쪽 5% 지점 확인
        ↓
Threshold 결정
```

이번 실행에서 계산된 값은:

```
Threshold = 0.982113
```

입니다.

따라서 다음 단계부터는 이 기준을 사용합니다.

```text
Similarity >= 0.982113
→ Normal

Similarity < 0.982113
→ Anomaly
```

## 왜 가장 낮은 값을 그대로 사용하지 않을까요?

정상 이미지 한 장에서 우연히 유난히 낮은 Similarity가 나올 수도 있습니다.

그 한 장의 값만 그대로 기준으로 사용하면 판정 기준이 지나치게 낮아질 수 있습니다.

그래서 오늘은 정상 데이터 전체의 분포를 보고 **낮은 쪽 일부를 기준으로 Threshold를 정하는 방법**을 사용합니다.

```
정상 데이터 한 장의 값만 사용
        ↓
우연한 값에 크게 영향을 받을 수 있음


정상 Calibration 여러 장 사용
        ↓
전체적인 정상 Similarity 범위 확인
        ↓
5 Percentile로 기준 결정
```

`5 Percentile`이 모든 제조 검사에서 반드시 사용해야 하는 정답이라는 뜻은 아닙니다.

오늘은 다음 과정을 연습하기 위한 **교육용 Baseline**입니다.

```
Calibration 데이터로
Threshold 결정
        ↓
Threshold 고정
        ↓
처음 보는 Query에 적용
        ↓
실제 성능 확인
```

여기서 가장 중요한 원칙은:

> **평가용 Query 결과를 본 뒤 Threshold를 마음대로 다시 조정하지 않는 것입니다.**

Query를 보기 전에 판정 기준을 정하고 고정해야, 처음 보는 데이터에서도 그 기준이 실제로 잘 작동하는지 확인할 수 있습니다.

---

# 22. 저장된 One-shot Rule을 확인합니다

20번에서 Reference 1장과 정상 Calibration 20장을 이용하여 One-shot 판정 기준을 만들었습니다.

그 기준은 다음 파일에 저장되어 있습니다.

```
configs/day08_one_shot_rules.json
```

이 파일은 단순한 실행 결과가 아니라,

> **앞으로 새로운 이미지를 검사할 때 사용할 One-shot의 기준과 조건을 기록해 둔 설정 파일입니다.**

확인합니다.

```bash
cat configs/day08_one_shot_rules.json
```

이번 실습에서는 다음과 같이 저장되었습니다.

```json
{
  "baseline_version": "day08-one-shot-v1.0",
  "dataset": "MVTec AD bottle",
  "method": "one_reference_cosine_similarity",
  "backbone": "resnet18_imagenet_pretrained",
  "reference_count": 1,
  "reference_file": "037.png",
  "reference_path": "artifacts/day08/one_shot_reference.npy",
  "calibration_count": 20,
  "threshold_method": "normal_calibration_5_percentile",
  "similarity_threshold": 0.9821131706237793,
  "decision_rule": "similarity >= threshold => Normal"
}
```

## 이 파일은 무엇을 기록하고 있나요?

이번 One-shot 실험에서 사용한 조건을 다시 확인해 보겠습니다.

|항목|이번 결과|의미|
|---|---|---|
|`baseline_version`|`day08-one-shot-v1.0`|이번 One-shot 기준의 버전|
|`dataset`|`MVTec AD bottle`|사용한 Dataset|
|`method`|`one_reference_cosine_similarity`|Reference 1장과 Similarity를 비교하는 방식|
|`backbone`|`resnet18_imagenet_pretrained`|Feature 추출에 사용한 Pretrained ResNet18|
|`reference_count`|`1`|정상 Reference를 정확히 1장 사용|
|`reference_file`|`037.png`|기준으로 사용한 정상 이미지|
|`reference_path`|`one_shot_reference.npy`|`037.png`에서 추출한 Feature를 저장한 파일|
|`calibration_count`|`20`|Threshold를 정할 때 사용한 정상 이미지 수|
|`threshold_method`|`normal_calibration_5_percentile`|정상 Calibration의 5 Percentile을 기준으로 사용|
|`similarity_threshold`|`0.982113...`|실제 계산된 판정 기준값|
|`decision_rule`|`similarity >= threshold => Normal`|최종 판정 방법|
## 가장 먼저 확인할 것은 Reference입니다

```
reference_count : 1
reference_file  : 037.png
```

이 결과는 이번 실험이 실제로:

```
정상 이미지 037.png
        ↓
Reference 1장
        ↓
One-shot 기준
```

으로 만들어졌다는 뜻입니다.

즉 이름만 One-shot인 것이 아니라, **실제로 정상 Reference 한 장을 기준으로 사용했다는 것을 기록으로 확인할 수 있습니다.**

---

## Reference 이미지는 Feature로 저장되어 있습니다

다음 항목도 확인합니다.

```
reference_path
→ artifacts/day08/one_shot_reference.npy
```

여기에는 `037.png` 이미지 자체가 저장된 것이 아닙니다.

앞의 공통 Feature 함수에서 `037.png`를 ResNet18에 넣어 만든 **숫자 Feature**가 저장되어 있습니다.

```
037.png
        ↓
Pretrained ResNet18
        ↓
Reference Feature
        ↓
one_shot_reference.npy
```

다음 Query 검사에서는 이미지를 다시 기준으로 만드는 것이 아니라, **이 저장된 Reference Feature를 불러와 사용합니다.**

---

## Threshold도 함께 저장되어 있습니다

이번 결과에서 가장 중요한 값 중 하나는 다음입니다.

```
similarity_threshold
→ 0.9821131706237793
```

20번 실행 화면에서는 보기 쉽게:

```
0.982113
```

으로 반올림해서 출력했지만, JSON에는 계산된 값이 더 정밀하게 저장되어 있습니다.

둘은 같은 Threshold입니다.

```
실행 화면
0.982113

JSON
0.9821131706237793
```

이 값을 이용하여 다음과 같이 판정합니다.

```
새 이미지
        ↓
Reference와 Similarity 계산
        ↓

Similarity >= 0.982113...
        ↓
Normal

Similarity < 0.982113...
        ↓
Anomaly
```

---

## 이 Threshold는 어떻게 만든 값인가요?

JSON에도 다음 내용이 기록되어 있습니다.

```
calibration_count
→ 20

threshold_method
→ normal_calibration_5_percentile
```

즉:

```
정상 Calibration 20장
        ↓
Reference와 각각 Similarity 계산
        ↓
정상 Similarity 분포 확인
        ↓
5 Percentile
        ↓
Threshold = 0.982113...
```

과정을 거쳐 만들어진 값이라는 뜻입니다.

따라서 `0.982113`이라는 숫자를 사람이 임의로 입력한 것이 아닙니다.

> **정상 Calibration 데이터를 이용하여 프로그램이 계산한 값입니다.**

---

## 마지막으로 판정 Rule을 확인합니다

```
decision_rule
→ similarity >= threshold => Normal
```

뜻은 간단합니다.

```
Reference와 충분히 비슷하다
        ↓
Similarity가 Threshold 이상
        ↓
Normal


Reference와 덜 비슷하다
        ↓
Similarity가 Threshold 미만
        ↓
Anomaly
```

입니다.

---

## 왜 이 파일을 저장해 둘까요?

이제 One-shot의 기준은 다음과 같이 모두 정해졌습니다.

```
Reference
→ 037.png

Reference Feature
→ one_shot_reference.npy

Calibration
→ 정상 20장

Threshold
→ 0.982113...

판정 Rule
→ Threshold 이상이면 Normal
```

이 값을 다음 Query 검사에서 다시 계산하거나 임의로 바꾸지 않습니다.

```
기준 생성 완료
        ↓
Rule 저장
        ↓
Rule Freeze
        ↓
처음 보는 Common Query에 적용
```

> **이 JSON 파일은 “이번 One-shot 검사를 어떤 조건으로 만들었는가?”를 기록하고, 다음 검사에서도 같은 기준을 그대로 사용할 수 있게 해주는 파일입니다.**

---

# 23. `65_day08_test_one_shot.py` — 고정한 One-shot Rule을 처음 보는 이미지에 적용합니다

20번에서는 정상 Reference 1장과 정상 Calibration 20장을 이용하여 One-shot 판정 기준을 만들었습니다.

이번 실습에서 결정된 기준은 다음과 같습니다.

```
Reference
→ 037.png

Threshold
→ 0.982113

판정 Rule
→ Similarity >= Threshold 이면 Normal
→ Similarity < Threshold 이면 Anomaly
```

22 단락에서는 이 기준이 다음 파일에 제대로 저장되어 있는지도 확인했습니다.

```
artifacts/day08/one_shot_reference.npy
→ Reference Feature

configs/day08_one_shot_rules.json
→ Threshold와 판정 Rule
```

이제 해야 할 일은 **이 기준이 처음 보는 이미지에서도 실제로 잘 작동하는지 확인하는 것**입니다.

---

## 왜 새로운 검사 코드가 필요한가요?

앞의 `64_day08_calibrate_one_shot.py`는 **판정 기준을 만드는 프로그램**이었습니다.

이번 `65_day08_test_one_shot.py`는 **만들어진 기준으로 실제 이미지를 검사하는 프로그램**입니다.

두 프로그램의 역할은 다릅니다.

```
64_day08_calibrate_one_shot.py

Reference 1장
+
정상 Calibration 20장
        ↓
Threshold 결정
        ↓
Rule 저장


65_day08_test_one_shot.py

저장된 Reference
+
저장된 Threshold
        ↓
처음 보는 Query 검사
        ↓
Normal / Anomaly 판정
```

즉 이번 단계부터 **실제 One-shot 검사 결과를 확인**합니다.

---

## 이번에는 어떤 이미지를 검사할까요?

데이터를 준비할 때 따로 남겨 두었던 `Common Query`를 사용합니다.

```
samples/day08/common_query/

Normal
→ 정상 이미지 10장

Anomaly / broken_small
→ 불량 이미지 10장
```

이 이미지들은 Reference나 Calibration을 만드는 데 사용하지 않은 **평가용 이미지**입니다.

```
기준을 만들 때 사용한 이미지
        ≠
이번에 검사할 Query 이미지
```

따라서 처음 보는 데이터에 가까운 조건으로 One-shot Rule을 시험할 수 있습니다. 현재 교재에서도 Common Query는 One-shot과 Few-shot 비교에 공통으로 사용하는 별도의 평가 데이터로 분리되어 있습니다.

---

## 이 프로그램에서는 무엇을 하나요?

이번 코드는 새로운 Threshold를 만들지 않습니다.

이미 만들어 둔 값을 그대로 불러옵니다.

```
Reference Feature 불러오기
        +
Threshold 불러오기
        ↓
Common Query 한 장 입력
        ↓
ResNet18 Feature 추출
        ↓
Reference Feature와
Similarity 계산
        ↓
Threshold와 비교
        ↓
Normal / Anomaly 판정
        ↓
실제 정답과 비교
        ↓
결과 저장
```

예를 들어 Query의 Similarity가 다음처럼 나왔다고 생각해 봅니다.

```
Similarity = 0.986
Threshold  = 0.982113
```

그러면:

```
0.986 >= 0.982113
        ↓
Normal
```

로 판정합니다.

반대로:

```
Similarity = 0.970
Threshold  = 0.982113
```

이라면:

```
0.970 < 0.982113
        ↓
Anomaly
```

로 판정합니다.

---

## 중요한 점 — Threshold는 다시 만들지 않습니다

이번 코드에서 특히 중요한 것은:

```
Threshold를 새로 계산하지 않는다.
```

는 것입니다.

20번에서 Calibration 데이터를 이용해:

```
Threshold = 0.982113
```

으로 결정했으므로, Query 결과가 어떻게 나오든 이 값을 그대로 사용합니다.

```
Calibration
        ↓
Threshold 결정
        ↓
Rule Freeze
        ↓
Common Query
        ↓
Frozen Rule 그대로 적용
```

> **Query 결과를 본 뒤 Threshold를 바꾸지 않고, 미리 결정한 기준이 실제로 얼마나 잘 작동하는지 확인하는 것이 이번 실습의 목적입니다.**

---

파일:

```text
src/65_day08_test_one_shot.py
```

작성합니다.

```python
import csv
import json
from pathlib import Path

import numpy as np

from day08_feature_core import (
    create_feature_model,
    cosine_similarity,
    extract_feature,
)


QUERY_ROOT = Path("samples/day08/common_query")
REFERENCE_PATH = Path("artifacts/day08/one_shot_reference.npy")
RULE_PATH = Path("configs/day08_one_shot_rules.json")
CSV_PATH = Path("reports/day08_one_shot_results.csv")


reference_feature = np.load(REFERENCE_PATH)

rules = json.loads(
    RULE_PATH.read_text(encoding="utf-8")
)

threshold = float(rules["similarity_threshold"])
model = create_feature_model()

rows = []


def inspect(image_path, actual, sample_id):
    feature = extract_feature(model, image_path)

    similarity = cosine_similarity(
        reference_feature,
        feature,
    )

    predicted = (
        "Normal"
        if similarity >= threshold
        else "Anomaly"
    )

    rows.append(
        {
            "sample_id": sample_id,
            "file": image_path.name,
            "actual": actual,
            "similarity": round(similarity, 6),
            "threshold": round(threshold, 6),
            "predicted": predicted,
            "correct": predicted == actual,
        }
    )


for image_path in sorted(
    (QUERY_ROOT / "normal").glob("*.png")
):
    inspect(
        image_path,
        "Normal",
        f"normal/{image_path.name}",
    )


anomaly_dir = (
    QUERY_ROOT
    / "anomaly"
    / "broken_small"
)

for image_path in sorted(anomaly_dir.glob("*.png")):
    inspect(
        image_path,
        "Anomaly",
        f"anomaly/broken_small/{image_path.name}",
    )


CSV_PATH.parent.mkdir(parents=True, exist_ok=True)

with CSV_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:
    writer = csv.DictWriter(
        file,
        fieldnames=rows[0].keys(),
    )
    writer.writeheader()
    writer.writerows(rows)


print("=" * 70)
print("DAY08 ONE-SHOT TEST")
print("=" * 70)
print("Threshold:", round(threshold, 6))
print()

for row in rows:
    print(
        row["sample_id"],
        "actual=",
        row["actual"],
        "predicted=",
        row["predicted"],
        "similarity=",
        row["similarity"],
        "correct=",
        row["correct"],
    )

print()
print("Saved:", CSV_PATH)
```

실행합니다.

```bash
python src/65_day08_test_one_shot.py
```

### 실행 결과를 해석합니다

이번 One-shot 검사는:

```
Normal Query  10장
Anomaly Query 10장
총 20장
```

을 검사했습니다.

결과를 정리하면 다음과 같습니다.

|구분|결과|
|---|---|
|Normal 정답|9 / 10|
|Anomaly 정답|8 / 10|
|전체 정답|17 / 20|
|Accuracy|**85%**|
|FP|1건|
|FN|2건|

즉 총 20장 중 **17장은 올바르게 판정했고 3장은 잘못 판정**했습니다.

---

## 오판이 나온 경우 어떻게 확인할까요?

검사 결과가 100%가 아니라고 해서 바로 코드를 수정하거나 Threshold를 바꾸지는 않습니다.

먼저 어떤 종류의 오판이 발생했는지 확인합니다.

```
FP
→ 실제 Normal인데 Anomaly로 판정

FN
→ 실제 Anomaly인데 Normal로 판정
```

이번 결과에서는:

```
FP 1건
FN 2건
```

이 발생했습니다.

---

## 대표 오판을 확인합니다

정상 이미지 중 한 장은:

```
normal/006.png

Actual     = Normal
Predicted  = Anomaly
Similarity = 0.959170
Threshold  = 0.982113
```

이었습니다.

실제로는 정상인데 Reference와의 Similarity가 Threshold보다 낮아 `Anomaly`로 판정되었습니다.

```
실제 Normal
        ↓
Similarity 0.959170
        <
Threshold 0.982113
        ↓
Anomaly로 오탐
```

반대로 다음 두 이미지는 실제 불량이지만:

```
broken_small/007.png
Similarity = 0.984909

broken_small/016.png
Similarity = 0.982900
```

Threshold보다 높게 나와 `Normal`로 판정되었습니다.

```
실제 Anomaly
        ↓
Similarity가 Threshold 이상
        ↓
Normal로 판정
        ↓
불량을 놓침
```

---

## 오판이 나오면 바로 원인을 단정하지 않습니다

Similarity 값만 보고:

```
이미지가 작아서 그렇다
Threshold가 잘못되었다
Reference가 잘못되었다
```

라고 바로 결론 내리지는 않습니다.

먼저 대표 실패 이미지를 몇 장만 직접 열어 확인합니다.

```
Reference 037.png
        VS
Normal 오판 이미지

확인할 것
→ 제품 위치
→ 크기
→ 밝기
→ 반사
→ 기울기
→ 전체적인 모양 차이
```

불량을 놓친 경우에는:

```
Reference 037.png
        VS
broken_small 오판 이미지

확인할 것
→ 결함 위치
→ 결함 크기
→ 결함이 전체 이미지에서 얼마나 작은가?
→ 전체적인 병 모양은 정상과 얼마나 비슷한가?
```

를 살펴봅니다.

---

## 모든 오판의 원인을 지금 완전히 증명할 필요는 없습니다

이번 실습의 목적은 오판 3장의 원인을 모두 완벽하게 밝혀내는 것이 아닙니다.

대표적인 실패를 확인하고 **가능한 원인 후보를 세우는 것**이 중요합니다.

예를 들어:

```
가능한 원인 ①
Reference 한 장이
모든 정상 모습을 대표하지 못했을 수 있음

가능한 원인 ②
조명·반사·위치 차이가
Feature에 영향을 주었을 수 있음

가능한 원인 ③
broken_small 결함이 작아서
전체 이미지 Feature에서는
차이가 충분히 크게 나타나지 않았을 수 있음

가능한 원인 ④
일부 이미지는 Threshold 경계에
매우 가까운 사례일 수 있음
```

처럼 기록합니다.

> **오판이 발생하면 먼저 대표 사례를 확인하고, 원인 가설을 세운 뒤 다음 실험에서 검증합니다.**

---

## 지금 Threshold를 다시 바꾸지는 않습니다

이번 Threshold `0.982113`은 이미 Calibration 데이터에서 결정했습니다.

따라서 Query 결과를 본 뒤:

```
006.png를 Normal로 만들기 위해
Threshold를 낮춰볼까?
```

처럼 바로 수정하지 않습니다.

그렇게 하면 **평가 결과를 본 뒤 판정 기준을 맞추는 것**이 되어 공정한 Test가 되지 않습니다.

```
Calibration
        ↓
Threshold 결정
        ↓
Rule Freeze
        ↓
Query Test
        ↓
결과 확인

오판 발생
        ↓
원인 후보 기록
        ↓
Threshold는 그대로 유지
```

합니다.

Threshold를 다시 검토해야 한다면 Query가 아니라 **별도의 Calibration 또는 Validation 데이터에서 다시 결정**해야 합니다.

---

## 이번 결과에서 무엇을 배울 수 있을까요?

이번 결과의 핵심은 `85%`라는 숫자 자체가 아닙니다.

```
Reference 1장
        ↓
대부분의 이미지는 구분 가능
        ↓
하지만 일부 Normal 오탐
일부 Anomaly 미검출 발생
        ↓
한 장의 Reference만으로는
모든 변화를 표현하기 어려울 수 있음
```

이라는 점을 확인한 것입니다.

> **One-shot은 적은 데이터로 빠르게 시작할 수 있지만, Reference 한 장의 대표성에 크게 영향을 받을 수 있습니다.**

이제 이 결과를 억지로 100%로 만들지 않고 그대로 유지합니다.

다음 Few-shot 실습에서는:

```
One-shot
Reference 1장

        VS

Few-shot
Normal Support 5장
+
Anomaly Support 5장
```

을 사용한 뒤, **같은 Common Query에서 결과가 어떻게 달라지는지 비교**합니다.

---

### 오판이 발생했을 때의 실무적인 대응 흐름

```
오판 발생
        ↓
FP / FN 종류 확인
        ↓
대표 실패 이미지 1~2장 확인
        ↓
가능한 원인 가설 작성
        ↓

Reference 문제?
촬영 조건 문제?
Threshold 경계 문제?
작은 결함 문제?
데이터 수 부족 문제?

        ↓
현재 Test에서는 기준 수정하지 않음
        ↓
다음 실험에서 한 가지 조건을 바꾸어 검증
        ↓
결과 비교
```

> 중요한 것은 오판을 없애는 것이 아니라, 왜 오판이 발생했는지 합리적인 가설을 세우고 다음 실험으로 연결하는 것입니다.


### 이코드의 전체 흐름

```
프로그램 시작
        ↓
저장된 Reference Feature 불러오기
one_shot_reference.npy
        ↓
저장된 One-shot Rule 불러오기
day08_one_shot_rules.json
        ↓
Threshold 읽기
0.982113...
        ↓
Pretrained ResNet18 준비
        ↓

Common Query 준비
        ↓
Normal 10장
+
Anomaly 10장
        ↓

Query 이미지 한 장 입력
        ↓
ResNet18 Feature 추출
        ↓
Reference Feature와
Cosine Similarity 계산
        ↓
Similarity와
Threshold 비교
        ↓

Similarity >= Threshold?
        ↓
   ┌────┴────┐
  YES        NO
   ↓          ↓
Normal      Anomaly
   └────┬─────┘
        ↓
실제 정답과 비교
        ↓
correct
True / False
        ↓
결과 목록에 저장
        ↓
다음 Query 이미지 검사
        ↓
20장 모두 반복
        ↓
day08_one_shot_results.csv 저장
        ↓
One-shot Test 완료
```

### 의사코드

```
1. Common Query 이미지가 저장된
   폴더 위치를 지정한다.

2. 20번에서 저장한
   Reference Feature 파일 위치를 지정한다.

3. 20번에서 저장한
   One-shot Rule JSON 위치를 지정한다.

4. 검사 결과를 저장할
   CSV 파일 위치를 지정한다.

5. one_shot_reference.npy에서
   Reference Feature를 불러온다.

6. day08_one_shot_rules.json을 읽는다.

7. JSON 안에 저장된
   similarity_threshold 값을 가져온다.

8. 이번 실습에서는 Threshold를
   다시 계산하지 않는다.

9. 앞에서 만든 공통 함수를 이용하여
   Pretrained ResNet18 Feature 모델을 준비한다.

10. 결과를 저장할
    빈 목록을 만든다.

11. Query 이미지 한 장이 들어오면
    ResNet18로 Feature를 추출한다.

12. Query Feature와
    Reference Feature의
    Cosine Similarity를 계산한다.

13. 계산된 Similarity를
    저장된 Threshold와 비교한다.

14. Similarity가 Threshold 이상이면

    Normal

    로 판정한다.

15. Similarity가 Threshold보다 낮으면

    Anomaly

    로 판정한다.

16. 실제 정답 actual과
    판정 결과 predicted를 비교한다.

17. 두 값이 같으면

    correct = True

    로 기록한다.

18. 두 값이 다르면

    correct = False

    로 기록한다.

19. 각 이미지에 대해 다음 내용을
    결과 목록에 저장한다.

    - sample_id
    - file
    - actual
    - similarity
    - threshold
    - predicted
    - correct

20. Common Query의
    정상 이미지 10장을 차례대로 검사한다.

21. 같은 방식으로
    broken_small 불량 이미지 10장도 검사한다.

22. 총 20장의 결과를
    reports/day08_one_shot_results.csv
    파일에 저장한다.

23. 실행 화면에도 각 이미지의
    실제 정답, 예측 결과,
    Similarity, 정답 여부를 출력한다.

24. 다음 24단락에서
    저장된 CSV를 확인하며
    어떤 이미지가 맞았고
    어떤 이미지가 틀렸는지 살펴본다.
```

## 이번 단계에서 꼭 이해할 것

20번과 23번의 차이를 구분하면 One-shot 전체 흐름이 훨씬 이해하기 쉽습니다.

```
20번
Reference + Calibration
        ↓
판정 기준 만들기
        ↓
Threshold = 0.982113


23번
처음 보는 Query
        ↓
이미 만들어 둔 Threshold 적용
        ↓
Normal / Anomaly 판정
```

> 20 단락은 One-shot의 기준을 만드는 단계이고, 23번은 그 기준으로 처음 보는 이미지를 실제 검사하는 단계입니다.


---

# 24. One-shot 결과 CSV를 확인합니다

23번에서는 Common Query 20장을 검사하고 결과를 다음 파일에 저장했습니다.

```bash
reports/day08_one_shot_results.csv
```

먼저 앞부분을 확인합니다.

```bash
head -n 11 reports/day08_one_shot_results.csv
```

이번 실행에서는 다음과 같은 결과를 확인할 수 있습니다.

```
sample_id,file,actual,similarity,threshold,predicted,correct
normal/000.png,000.png,Normal,0.992058,0.982113,Normal,True
normal/001.png,001.png,Normal,0.991581,0.982113,Normal,True
normal/002.png,002.png,Normal,0.983201,0.982113,Normal,True
normal/003.png,003.png,Normal,0.990551,0.982113,Normal,True
normal/004.png,004.png,Normal,0.991311,0.982113,Normal,True
normal/005.png,005.png,Normal,0.986896,0.982113,Normal,True
normal/006.png,006.png,Normal,0.959170,0.982113,Anomaly,False
normal/007.png,007.png,Normal,0.983366,0.982113,Normal,True
normal/008.png,008.png,Normal,0.987457,0.982113,Normal,True
normal/009.png,009.png,Normal,0.987604,0.982113,Normal,True
```

## 각 열은 무엇을 의미하나요?

|열|의미|
|---|---|
|`sample_id`|Common Query 안에서 이미지의 위치를 포함한 이름|
|`file`|실제 이미지 파일명|
|`actual`|Dataset에 기록된 실제 정답|
|`similarity`|Reference Feature와의 Similarity|
|`threshold`|Calibration에서 미리 정하고 고정한 판정 기준|
|`predicted`|One-shot이 예측한 결과|
|`correct`|실제 정답과 예측 결과가 같은지 여부|

결과를 읽을 때는 먼저 `actual`, `predicted`, `correct`를 함께 확인합니다.

```
actual = Normal
predicted = Normal
correct = True

→ 정상 이미지를 정상으로 맞게 판정
```

반면 이번 결과에는 다음과 같은 오판도 있습니다.

```
normal/006.png

actual     = Normal
similarity = 0.959170
threshold  = 0.982113
predicted  = Anomaly
correct    = False
```

이 이미지의 실제 정답은 `Normal`이지만 Similarity가 Threshold보다 낮았습니다.

```
Similarity 0.959170
        <
Threshold 0.982113
        ↓
Predicted = Anomaly
```

따라서:

```
실제 Normal
        ↓
Anomaly로 잘못 판정
        ↓
False Positive
```

입니다.

> `correct = False`는 프로그램 오류라는 뜻이 아니라, **현재 One-shot 판정 기준으로 해당 이미지를 잘못 분류했다는 뜻**입니다.

---

## 오판된 결과만 빠르게 확인할 수도 있습니다

CSV에서 `correct=False`인 결과만 확인합니다.

```
grep 'False' reports/day08_one_shot_results.csv
```

이번 실험에서는 총 3개의 오판이 확인됩니다.

```
Normal → Anomaly
1건
→ False Positive

Anomaly → Normal
2건
→ False Negative
```

이 오판들을 발견했다고 해서 지금 바로 Threshold를 수정하지는 않습니다.

```
오판 확인
        ↓
대표 이미지 몇 장 확인
        ↓
가능한 원인 기록
        ↓
현재 Threshold는 유지
        ↓
Few-shot 진행
        ↓
같은 Query에서 결과 비교
```

오판의 원인을 지금 모두 증명하는 것이 목적은 아닙니다.

> **One-shot에서 어떤 이미지가 틀렸는지 확인하고, 이후 Few-shot에서도 같은 이미지의 판정이 어떻게 달라지는지 비교하는 것이 다음 중요한 과정입니다.**

---

# 25. One-shot의 한계를 실제 결과와 연결합니다

이번 One-shot 실험에서는 정상 Reference로 **`037.png` 한 장**만 사용했습니다.

그리고 Common Query 20장을 검사한 결과:

```
정상 10장
→ 9장 정답
→ 1장 오판

불량 10장
→ 8장 정답
→ 2장 오판

전체
17 / 20 정답
Accuracy = 85%
```

이 나왔습니다.

즉 Reference 한 장만으로도 많은 이미지를 구분할 수 있었지만, **모든 정상과 불량을 완벽하게 구분하지는 못했습니다.**

---

## 정상인데 Anomaly로 판정된 경우

이번 결과에서는:

```
normal/006.png

Actual     = Normal
Predicted  = Anomaly
Similarity = 0.959170
Threshold  = 0.982113
```

이었습니다.

실제로는 정상 이미지지만 Reference `037.png`와의 Similarity가 기준보다 낮았습니다.

```
실제 Normal
        ↓
Reference와 비교
        ↓
Similarity = 0.959170
        <
Threshold = 0.982113
        ↓
Anomaly로 판정
        ↓
False Positive
```

이런 경우에는 Reference와 Query 이미지를 직접 비교해 볼 수 있습니다.

확인할 수 있는 항목은 다음과 같습니다.

```
제품 위치가 다른가?

크기가 조금 다른가?

조명이나 반사가 다른가?

기울기나 촬영 상태가 다른가?

전체적인 질감이 다르게 보이는가?
```

하지만 이미지를 보고 바로:

```
"이것 때문에 틀렸다."
```

라고 단정할 필요는 없습니다.

> **Reference 한 장이 정상 제품에서 나타날 수 있는 모든 변화를 충분히 대표하지 못했을 가능성이 있다**

정도로 이해하면 됩니다.

---

## 불량인데 Normal로 판정된 경우

이번 실험에서는 다음 두 이미지가 불량인데도 Normal로 판정되었습니다.

```
broken_small/007.png
Similarity = 0.984909

broken_small/016.png
Similarity = 0.982900
```

두 이미지 모두 Threshold `0.982113`보다 높았습니다.

```
실제 Anomaly
        ↓
Reference와 비교
        ↓
Similarity가 Threshold보다 높음
        ↓
Normal로 판정
        ↓
False Negative
```

이 경우에는 다음과 같은 가능성을 생각해 볼 수 있습니다.

```
결함이 전체 이미지에서 매우 작은가?

결함 부분을 제외하면
전체 병의 형태가 정상과 매우 비슷한가?

전체 이미지 Feature에서는
작은 국소 결함의 차이가
충분히 크게 나타나지 않았는가?
```

현재 One-shot 방식은 병의 특정 결함 위치만 따로 검사하는 것이 아니라, **이미지 전체에서 추출한 Feature를 Reference와 비교**합니다.

따라서 작은 결함은 전체적인 병 모양에 비해 영향이 작게 나타날 수도 있습니다.

다만 이것도 현재 단계에서는 **가능한 원인 후보**로 생각합니다.

---

## 오판이 나오면 무엇을 해야 할까요?

오판이 발생했다고 바로 Threshold를 바꾸지는 않습니다.

이번 Threshold는 이미 Calibration 단계에서 정하고 고정했습니다.

따라서 다음 순서로 처리합니다.

```
오판 확인
        ↓
FP / FN 구분
        ↓
대표 실패 이미지 확인
        ↓
가능한 원인 가설 작성
        ↓
현재 Threshold는 유지
        ↓
다음 실험 진행
```

즉 결과를 100%로 만들기 위해 바로 수치를 조정하는 것이 아니라:

> **어떤 조건에서 One-shot이 잘못 판단할 수 있는지 이해하는 것**

이 더 중요합니다.

---

## One-shot의 장점과 한계

One-shot의 장점은 분명합니다.

```
정상 Reference 1장
        ↓
빠르게 기준 생성
        ↓
많은 데이터 없이도
검사 시작 가능
```

하지만 Reference가 한 장뿐이므로 다음 한계가 있습니다.

```
정상 모습의 다양성
        ↓
한 장으로 모두 표현하기 어려움

작은 불량
        ↓
전체 Feature에서는
차이가 작게 나타날 수 있음

Reference 선택
        ↓
어떤 한 장을 선택했는가에
결과가 영향을 받을 수 있음
```

이번 실험 결과가 바로 이 점을 보여줍니다.

```
One-shot
Reference 1장
        ↓
Accuracy 85%
        ↓
FP 1건
FN 2건
        ↓
한 장의 기준만으로는
일부 상황에서 한계가 있음
```

> **One-shot은 데이터가 매우 적을 때 빠르게 시작할 수 있지만, Reference 한 장의 대표성에 크게 의존합니다.**

---

## 이제 Few-shot으로 넘어갑니다

지금 이 결과를 억지로 수정하여 100%로 만들지는 않습니다.

다음 Few-shot에서는:

```
One-shot
Normal Reference 1장

        VS

Few-shot
Normal Support 5장
+
Anomaly Support 5장
```

을 사용합니다.

그리고 **같은 Common Query**를 다시 검사하여 결과를 비교합니다.

```
One-shot 결과
        ↓
FP 1
FN 2
Accuracy 85%

        ↓

Few-shot 실행
        ↓
같은 Query 검사
        ↓
어떤 오판이 줄었는가?
새로운 오판은 생겼는가?
```

이 비교를 통해:

> **한 장의 Reference를 사용하는 방법과 몇 장의 예제를 사용하는 방법이 실제로 어떻게 다른지**

확인하는 것이 다음 실습의 목적입니다.

---

# PART4 — Few-shot: 소수 Support로 Prototype 만들기 (26~32)

---

# 26. Few-shot에서 사용할 실제 데이터를 확인합니다

앞에서는 Few-shot이 **정상과 불량 예제를 몇 장씩 이용하여 기준을 만드는 방법**이라고 배웠습니다.

이제 실제 실습에 사용할 데이터를 확인합니다.

오늘 기준을 만드는 데 사용할 Support는 다음과 같습니다.

```
Normal Support
5장

Anomaly Support
broken_small 5장
```

즉 이번 Few-shot에서는:

```
정상 예제 5장
        +
불량 예제 5장
        ↓
두 그룹의 기준 만들기
```

를 진행합니다.

그리고 기준을 만든 뒤 검사할 이미지는 앞의 One-shot에서 사용했던 **같은 Common Query**입니다.

```
Common Query

Normal
10장

Anomaly
10장
```

중요한 점은 Support와 Query가 서로 다른 이미지라는 것입니다.

```
Support
→ 기준을 만들 때 사용

Query
→ 만들어진 기준을 시험할 때 사용
```

또한 One-shot과 Few-shot을 공정하게 비교하기 위해 **Query는 바꾸지 않습니다.**

```
                    같은 Common Query
                     Normal 10장
                    Anomaly 10장
                          ↓
                ┌─────────┴─────────┐
                ↓                   ↓
             One-shot            Few-shot
                ↓                   ↓
             결과 저장            결과 저장
                └─────────┬─────────┘
                          ↓
                     나중에 비교
```

> 이번 Few-shot에서는 **기준을 만드는 데이터만 달라지고, 마지막에 시험하는 이미지는 One-shot과 동일하게 유지합니다.**
---

# 27. Few-shot에서는 여러 예제를 하나의 대표 기준으로 만듭니다

One-shot에서는 정상 Reference 한 장이 기준이었습니다.

Few-shot에서는 한 장이 아니라 여러 장의 Support가 있으므로, 각 이미지를 따로따로 Query와 비교하기보다 **여러 이미지의 특징을 모아 하나의 대표 기준**을 만듭니다.

정상 Support 5장을 각각 Feature로 바꿉니다.

```
Normal 1
→ Feature

Normal 2
→ Feature

Normal 3
→ Feature

Normal 4
→ Feature

Normal 5
→ Feature
        ↓
Feature 평균
        ↓
Normal Prototype
```

불량 Support도 같은 방식으로 처리합니다.

```
Anomaly 1
→ Feature

Anomaly 2
→ Feature

Anomaly 3
→ Feature

Anomaly 4
→ Feature

Anomaly 5
→ Feature
        ↓
Feature 평균
        ↓
Anomaly Prototype
```

이렇게 만들어진 두 개의 대표 기준이:

```
Normal Prototype

Anomaly Prototype
```

입니다.

쉽게 생각하면:

> **Normal Prototype은 정상 5장의 특징을 대표하는 기준이고, Anomaly Prototype은 불량 5장의 특징을 대표하는 기준입니다.**

여기서 중요한 것은 이미지 5장을 단순히 합치는 것이 아니라, **각 이미지를 ResNet18 Feature로 바꾼 뒤 그 Feature들의 평균을 계산한다는 점**입니다.

```
이미지 여러 장
        ↓
각각 Feature 추출
        ↓
Feature 평균
        ↓
하나의 대표 Feature
```

그리고 다음 Query 검사에서는 새로운 이미지가:

```
Normal Prototype과 더 비슷한가?

또는

Anomaly Prototype과 더 비슷한가?
```

를 비교하게 됩니다.

```
Query Feature
       ↓
 ┌─────┴─────┐
 ↓           ↓
Normal     Anomaly
Prototype Prototype
 ↓           ↓
Similarity Similarity
     \       /
      \     /
       비교
        ↓
더 높은 쪽으로 판정
```

하지만 **아직 Query를 검사하지 않습니다.**

먼저 지금은 두 Prototype을 만들어 저장하는 단계입니다.

---

# 28. `66_day08_build_fewshot_prototypes.py` — Few-shot의 두 기준을 만듭니다

이제 실제 코드로:

```
Normal Support 5장
        ↓
Normal Prototype

Anomaly Support 5장
        ↓
Anomaly Prototype
```

을 만듭니다.

앞에서 만든 `day08_feature_core.py`를 다시 사용하므로 ResNet18 Feature 추출 코드를 새로 작성하지 않습니다.

이번 프로그램에서는 공통 함수 중 다음 기능을 사용합니다.

```
create_feature_model()
→ ResNet18 Feature 모델 준비

extract_feature()
→ 각 Support 이미지를 Feature로 변환

build_prototype()
→ 여러 Feature의 평균으로 Prototype 생성
```

전체적으로는 다음 작업을 하는 프로그램입니다.

```
Few-shot Support 폴더 확인
        ↓

Normal Support 5장 읽기
        ↓
각 이미지 Feature 추출
        ↓
Normal Prototype 생성

        +

Anomaly Support 5장 읽기
        ↓
각 이미지 Feature 추출
        ↓
Anomaly Prototype 생성
        ↓

두 Prototype 저장
        ↓
사용한 Support 수와
실험 조건 저장
```

## 왜 Prototype을 먼저 파일로 저장할까요?

다음 Query 검사에서 Support 10장을 다시 읽어 Prototype을 새로 만들 수도 있습니다.

하지만 그렇게 하지 않고, 이번 단계에서 한 번 만든 기준을 저장해 둡니다.

```
Support
        ↓
Prototype 생성
        ↓
파일로 저장
        ↓
기준 고정
        ↓
다음 Query Test에서 재사용
```

One-shot에서도:

```
Reference + Calibration
        ↓
Threshold 결정
        ↓
Rule 저장
```

한 뒤 Query를 검사했습니다.

Few-shot도 비슷하게:

```
Support
        ↓
Prototype 결정
        ↓
Prototype 저장
        ↓
Query 검사
```

순서로 진행합니다.

> **이번 코드는 아직 Few-shot의 최종 성능을 확인하는 코드가 아닙니다. Query를 검사하기 전에 Normal과 Anomaly의 기준을 먼저 만드는 코드입니다.**

---

파일을 만듭니다

```text
src/66_day08_build_fewshot_prototypes.py
```

작성합니다.

```python
import json
from pathlib import Path

import numpy as np

from day08_feature_core import (
    build_prototype,
    create_feature_model,
    extract_feature,
)


SUPPORT_ROOT = Path("samples/day08/few_shot/support")
ARTIFACT_PATH = Path("artifacts/day08/fewshot_prototypes.npz")
CONFIG_PATH = Path("configs/day08_fewshot_config.json")


model = create_feature_model()


def extract_folder(folder):
    files = sorted(folder.glob("*.png"))

    if not files:
        raise RuntimeError(f"Support 이미지가 없습니다: {folder}")

    features = [
        extract_feature(model, image_path)
        for image_path in files
    ]

    return files, features


normal_files, normal_features = extract_folder(
    SUPPORT_ROOT / "normal"
)

anomaly_files, anomaly_features = extract_folder(
    SUPPORT_ROOT / "anomaly"
)


normal_prototype = build_prototype(normal_features)
anomaly_prototype = build_prototype(anomaly_features)


ARTIFACT_PATH.parent.mkdir(parents=True, exist_ok=True)

np.savez_compressed(
    ARTIFACT_PATH,
    normal_prototype=normal_prototype,
    anomaly_prototype=anomaly_prototype,
)


config = {
    "baseline_version": "day08-few-shot-v1.0",
    "dataset": "MVTec AD bottle",
    "method": "prototype_cosine_similarity",
    "backbone": "resnet18_imagenet_pretrained",
    "normal_support_count": len(normal_files),
    "anomaly_support_count": len(anomaly_files),
    "anomaly_support_type": "broken_small",
    "decision_rule": "choose prototype with higher similarity",
}

CONFIG_PATH.parent.mkdir(parents=True, exist_ok=True)
CONFIG_PATH.write_text(
    json.dumps(
        config,
        indent=2,
        ensure_ascii=False,
    ),
    encoding="utf-8",
)


print("=" * 70)
print("DAY08 FEW-SHOT PROTOTYPE")
print("=" * 70)
print("Normal Support  :", len(normal_files))
print("Anomaly Support :", len(anomaly_files))
print("Anomaly Type    : broken_small")
print("Saved Artifact  :", ARTIFACT_PATH)
print("Saved Config    :", CONFIG_PATH)
```

실행합니다.

```bash
python src/66_day08_build_fewshot_prototypes.py
```

### 실행결과 해석

```
Normal Support  : 5
Anomaly Support : 5
Anomaly Type    : broken_small
Saved Artifact  : artifacts/day08/fewshot_prototypes.npz
Saved Config    : configs/day08_fewshot_config.json
```

### 실행 결과를 해석합니다

먼저:

```
Normal Support  : 5
Anomaly Support : 5
```

는 이번 Few-shot 기준을 만들 때:

```
정상 이미지 5장
+
불량 이미지 5장
```

을 실제로 사용했다는 뜻입니다.

```
Anomaly Type : broken_small
```

은 이번 Anomaly 기준이 **모든 종류의 불량을 대표하는 것이 아니라 `broken_small` 이미지 5장을 이용해 만든 기준**이라는 뜻입니다.

그리고:

```
Saved Artifact
→ artifacts/day08/fewshot_prototypes.npz
```

에는 방금 만든 두 개의 Prototype이 저장됩니다.

```
fewshot_prototypes.npz

├─ normal_prototype
│  → 정상 Support 5장을 대표하는 Feature
│
└─ anomaly_prototype
   → broken_small Support 5장을 대표하는 Feature
```

즉 이미지 10장이 그대로 저장된 것이 아니라, 각 이미지를 ResNet18 Feature로 변환한 뒤 평균하여 만든 **대표 Feature 두 개가 저장된 것**입니다.

다음 파일도 만들어졌습니다.

```
Saved Config
→ configs/day08_fewshot_config.json
```

이 파일에는 이번 Few-shot 실험이 어떤 조건으로 만들어졌는지가 기록됩니다.

예를 들면:

```
정상 Support를 몇 장 사용했는가?

불량 Support를 몇 장 사용했는가?

어떤 불량 종류를 사용했는가?

어떤 Feature Extractor를 사용했는가?

어떤 방법으로 판정할 것인가?
```

와 같은 내용입니다.

> 여기까지는 **Few-shot의 기준을 만든 단계**입니다. 아직 Common Query를 검사하거나 Accuracy를 계산한 것은 아닙니다.

---

## 이번 코드가 처리한 전체 흐름

```
프로그램 시작
        ↓
Few-shot Support 폴더 위치 지정
        ↓
결과를 저장할
Artifact / Config 경로 지정
        ↓
Pretrained ResNet18 준비
        ↓

Normal Support 폴더 읽기
        ↓
정상 이미지 5장 확인
        ↓
각 이미지 Feature 추출
        ↓
Feature 5개 생성
        ↓
Feature 평균
        ↓
Normal Prototype 생성


Anomaly Support 폴더 읽기
        ↓
broken_small 이미지 5장 확인
        ↓
각 이미지 Feature 추출
        ↓
Feature 5개 생성
        ↓
Feature 평균
        ↓
Anomaly Prototype 생성
        ↓

Normal Prototype
+
Anomaly Prototype
        ↓
fewshot_prototypes.npz 저장
        ↓

이번 실험 조건 정리
        ↓
day08_fewshot_config.json 저장
        ↓

Support 수와
저장 위치 출력
        ↓
Few-shot 기준 생성 완료
```

---

## 의사코드

```
1. Few-shot Support 이미지가 있는
   폴더 위치를 지정한다.

2. 만들어진 Prototype을 저장할
   파일 위치를 지정한다.

3. 이번 Few-shot 실험 조건을 저장할
   Config 파일 위치를 지정한다.

4. 앞에서 만든 공통 함수를 이용하여
   Pretrained ResNet18 Feature 모델을 준비한다.

5. Normal Support 폴더의
   PNG 이미지 목록을 읽는다.

6. Normal Support 이미지가 하나도 없다면
   기준을 만들 수 없으므로 실행을 중단한다.

7. Normal Support 이미지를
   한 장씩 ResNet18에 입력한다.

8. 각 이미지에서 Feature를 추출한다.

9. 추출한 Normal Feature들을
   하나의 목록으로 모은다.

10. Normal Feature들의 평균을 계산한다.

11. 평균 Feature를 정리하여
    Normal Prototype을 만든다.

12. Anomaly Support 폴더의
    PNG 이미지 목록을 읽는다.

13. Anomaly Support 이미지가 하나도 없다면
    실행을 중단한다.

14. Anomaly Support 이미지를
    한 장씩 ResNet18에 입력한다.

15. 각 이미지에서 Feature를 추출한다.

16. 추출한 Anomaly Feature들을
    하나의 목록으로 모은다.

17. Anomaly Feature들의 평균을 계산한다.

18. 평균 Feature를 정리하여
    Anomaly Prototype을 만든다.

19. 만들어진 두 Prototype을

    artifacts/day08/fewshot_prototypes.npz

    파일에 함께 저장한다.

20. 이번 실험에서 사용한 조건을 정리한다.

    - Dataset
    - Feature Extractor
    - Normal Support 수
    - Anomaly Support 수
    - Anomaly Type
    - 판정 방식

21. 실험 조건을

    configs/day08_fewshot_config.json

    파일에 저장한다.

22. 실제 사용한 Normal Support 수와
    Anomaly Support 수를 화면에 출력한다.

23. 두 Prototype이 저장된 위치를
    화면에 출력한다.

24. 다음 실습에서는 저장된 두 Prototype을 불러와
    Common Query와 각각 Similarity를 계산한다.
```

---

## 지금 단계에서 꼭 이해할 것

One-shot에서는 기준이:

```
정상 Reference 1장
```

이었습니다.

Few-shot에서는 기준이:

```
Normal Support 5장
        ↓
Normal Prototype

Anomaly Support 5장
        ↓
Anomaly Prototype
```

으로 바뀌었습니다.

즉 지금까지 만든 것은 다음 두 개의 **대표 기준**입니다.

```
Normal Prototype
→ 정상 쪽 대표 기준

Anomaly Prototype
→ broken_small 불량 쪽 대표 기준
```

아직 다음 작업은 하지 않았습니다.

```
Common Query 입력
        ↓
Normal Prototype과 비교
        ↓
Anomaly Prototype과 비교
        ↓
Normal / Anomaly 판정
```

이 작업은 다음 Few-shot Test 코드에서 진행합니다.

> 28번의 목적은 Few-shot으로 판정하는 것이 아니라, 판정할 때 사용할 Normal / Anomaly 기준 두 개를 먼저 만드는 것입니다.


---

# 29. 저장된 Prototype과 Few-shot 설정을 확인합니다

28번에서는 다음 과정을 통해 Few-shot에서 사용할 두 개의 대표 기준을 만들었습니다.

```
Normal Support 5장
        ↓
각 이미지의 Feature 추출
        ↓
Feature 평균
        ↓
Normal Prototype


Anomaly Support 5장
        ↓
각 이미지의 Feature 추출
        ↓
Feature 평균
        ↓
Anomaly Prototype
```

그리고 만들어진 결과를 두 종류의 파일로 저장했습니다.

```
artifacts/day08/fewshot_prototypes.npz
→ 실제 판정에 사용할 Prototype 저장

configs/day08_fewshot_config.json
→ Prototype을 어떤 조건으로 만들었는지 기록
```

이제 두 파일이 정상적으로 만들어졌는지 확인합니다.

> 이 단계에서는 새로운 Prototype을 다시 만드는 것이 아닙니다. **앞에서 만든 Few-shot 기준과 실험 조건이 제대로 저장되었는지 확인하는 단계**입니다.

---

## 29-1. Prototype 파일이 생성되었는지 확인합니다

다음 명령을 실행합니다.

```bash
ls -lh artifacts/day08/fewshot_prototypes.npz
```

실행 결과:

```
-rw-r--r-- 1 youjung youjung 4.2K Sep 13 18:11 artifacts/day08/fewshot_prototypes.npz
```

이 결과에서 가장 중요한 부분은 다음입니다.

```
artifacts/day08/fewshot_prototypes.npz
```

파일이 실제로 존재하고 있습니다.

따라서 28번에서 만든:

```
Normal Prototype
+
Anomaly Prototype
```

이 파일에 정상적으로 저장되었다는 것을 확인할 수 있습니다.

---

# 29-2. Few-shot 실험 조건을 확인합니다

다음 명령을 실행합니다.

```bash
cat configs/day08_fewshot_config.json
```

실행 결과:

```json
{
  "baseline_version": "day08-few-shot-v1.0",
  "dataset": "MVTec AD bottle",
  "method": "prototype_cosine_similarity",
  "backbone": "resnet18_imagenet_pretrained",
  "normal_support_count": 5,
  "anomaly_support_count": 5,
  "anomaly_support_type": "broken_small",
  "decision_rule": "choose prototype with higher similarity"
}
```

이 파일을 통해 **이번 Few-shot 기준을 어떤 조건으로 만들었는지** 확인할 수 있습니다.

---

실행 결과에서 이번 실험이 다음 조건으로 저장되었는지만 확인합니다.

```
Normal Support   = 5장
Anomaly Support  = 5장
Anomaly Type     = broken_small
Decision Rule    = 두 Prototype 중 Similarity가 더 높은 쪽 선택
```

즉 이번 Few-shot에서는 정상 이미지 5장과 `broken_small` 불량 이미지 5장을 이용해 각각의 Prototype을 만들었습니다.

```
Normal Support 5장
        +
broken_small Support 5장
        ↓
Normal Prototype
Anomaly Prototype
        ↓
두 기준 준비 완료
```

`Config`의 다른 항목들은 이번 실험의 조건을 기록해 둔 정보이므로 지금 단계에서 하나씩 자세히 분석할 필요는 없습니다.

여기까지 확인했다면 **Few-shot에서 사용할 두 기준이 정상적으로 준비된 것입니다.**

다음 30단락에서는 저장된 Normal Prototype과 Anomaly Prototype을 불러와, One-shot에서 사용했던 것과 동일한 Common Query 20장을 실제로 검사합니다.

---

# 30. `67_day08_test_fewshot.py` — Common Query를 두 Prototype과 비교합니다

29번까지 Few-shot에서 사용할 두 기준을 준비했습니다.

```
Normal Prototype
Anomaly Prototype
```

이제부터는 **이 두 기준이 실제로 새로운 이미지를 얼마나 잘 구분하는지 검사**합니다.

앞에서 Prototype을 만드는 것까지만 했다면, 이번 코드는 그 Prototype을 실제 판정에 사용하는 단계입니다.

## 이번 코드는 왜 작성할까요?

Few-shot에서 기준을 만들었다고 해서 바로 성능을 알 수 있는 것은 아닙니다.

실제로 처음 보는 Query 이미지를 넣어 보고:

```
이 이미지는 Normal에 더 가까운가?

또는

Anomaly에 더 가까운가?
```

를 확인해야 합니다.

따라서 이번 프로그램에서는 28번에서 저장한:

```
artifacts/day08/fewshot_prototypes.npz
```

를 다시 불러옵니다.

그리고 One-shot에서 검사했던 것과 동일한 Common Query를 사용합니다.

```
Common Query

Normal 10장
Anomaly 10장

총 20장
```

같은 Query를 사용하는 이유는 나중에 **One-shot과 Few-shot의 결과를 같은 조건에서 비교하기 위해서**입니다.

---

## 이번에는 어떻게 판정할까요?

One-shot에서는 한 장의 Reference와 비교한 Similarity를 Threshold와 비교했습니다.

Few-shot에서는 방식이 조금 다릅니다.

Query 이미지 한 장에서 Feature를 추출한 뒤:

```
Query Feature
        ↓
        ┌──────────────────┐
        ↓                  ↓
Normal Prototype      Anomaly Prototype
        ↓                  ↓
Similarity 계산        Similarity 계산
        └────────┬─────────┘
                 ↓
             두 값 비교
```

합니다.

예를 들어:

```
Normal Similarity  = 0.94
Anomaly Similarity = 0.87
```

이라면 Normal Prototype과 더 비슷하므로:

```
Predicted = Normal
```

로 판정합니다.

반대로:

```
Normal Similarity  = 0.85
Anomaly Similarity = 0.92
```

라면 Anomaly Prototype과 더 비슷하므로:

```
Predicted = Anomaly
```

로 판정합니다.

즉 이번 Few-shot에서는 별도의 Threshold를 사용하는 것이 아니라:

> **Normal Prototype과 Anomaly Prototype 중 어느 쪽과 더 비슷한지를 비교하여 판정합니다.**

---

## 이번 코드에서는 무엇을 하나요?

프로그램은 다음 순서로 동작합니다.

```
저장된 Prototype 파일 불러오기
        ↓
Normal Prototype 가져오기
Anomaly Prototype 가져오기
        ↓
Pretrained ResNet18 준비
        ↓
Common Query 한 장 읽기
        ↓
Feature 추출
        ↓
Normal Prototype과 Similarity 계산
        ↓
Anomaly Prototype과 Similarity 계산
        ↓
두 값 비교
        ↓
Normal / Anomaly 판정
        ↓
실제 정답과 비교
        ↓
correct = True / False 기록
        ↓
20장 반복
        ↓
결과 CSV 저장
```

이번 코드에서 중요한 것은 **Prototype을 다시 만드는 것이 아니라, 앞에서 만든 Prototype을 그대로 불러와 사용한다는 점**입니다.

```
28번
Support 데이터
→ Prototype 생성 및 저장

29번
저장 결과 확인

30번
저장된 Prototype
→ Common Query 실제 검사
```

---

## 실행이 끝나면 무엇이 만들어질까요?

각 Query에 대해 다음 내용을 기록합니다.

```
sample_id
→ 어떤 이미지인지

actual
→ 실제 정답

normal_similarity
→ Normal Prototype과의 유사도

anomaly_similarity
→ Anomaly Prototype과의 유사도

predicted
→ Few-shot이 판정한 결과

correct
→ 실제 정답과 판정 결과가 같은지 여부
```

그리고 전체 결과를 다음 파일에 저장합니다.

```
reports/day08_fewshot_results.csv
```

이 CSV는 다음 단계에서 확인하고, 이후 One-shot 결과와 비교할 때도 사용합니다.

> **이번 코드의 목적은 “두 Prototype 중 어느 쪽과 더 비슷한가?”를 이용하여 Common Query 20장을 실제로 판정하는 것입니다.**

---

## 파일을 작성합니다

```text
src/67_day08_test_fewshot.py
```

작성합니다.

```python
import csv
from pathlib import Path

import numpy as np

from day08_feature_core import (
    create_feature_model,
    cosine_similarity,
    extract_feature,
)


QUERY_ROOT = Path("samples/day08/common_query")
ARTIFACT_PATH = Path("artifacts/day08/fewshot_prototypes.npz")
CSV_PATH = Path("reports/day08_fewshot_results.csv")


prototype_data = np.load(ARTIFACT_PATH)
normal_prototype = prototype_data["normal_prototype"]
anomaly_prototype = prototype_data["anomaly_prototype"]

model = create_feature_model()
rows = []


def inspect(image_path, actual, sample_id):
    feature = extract_feature(model, image_path)

    normal_similarity = cosine_similarity(
        feature,
        normal_prototype,
    )

    anomaly_similarity = cosine_similarity(
        feature,
        anomaly_prototype,
    )

    predicted = (
        "Normal"
        if normal_similarity >= anomaly_similarity
        else "Anomaly"
    )

    rows.append(
        {
            "sample_id": sample_id,
            "file": image_path.name,
            "actual": actual,
            "normal_similarity": round(normal_similarity, 6),
            "anomaly_similarity": round(anomaly_similarity, 6),
            "predicted": predicted,
            "correct": predicted == actual,
        }
    )


for image_path in sorted(
    (QUERY_ROOT / "normal").glob("*.png")
):
    inspect(
        image_path,
        "Normal",
        f"normal/{image_path.name}",
    )


anomaly_dir = (
    QUERY_ROOT
    / "anomaly"
    / "broken_small"
)

for image_path in sorted(anomaly_dir.glob("*.png")):
    inspect(
        image_path,
        "Anomaly",
        f"anomaly/broken_small/{image_path.name}",
    )


CSV_PATH.parent.mkdir(parents=True, exist_ok=True)

with CSV_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:
    writer = csv.DictWriter(
        file,
        fieldnames=rows[0].keys(),
    )
    writer.writeheader()
    writer.writerows(rows)


print("=" * 70)
print("DAY08 FEW-SHOT TEST")
print("=" * 70)

for row in rows:
    print(
        row["sample_id"],
        "actual=",
        row["actual"],
        "predicted=",
        row["predicted"],
        "normal_sim=",
        row["normal_similarity"],
        "anomaly_sim=",
        row["anomaly_similarity"],
        "correct=",
        row["correct"],
    )

print()
print("Saved:", CSV_PATH)
```

실행합니다.

```bash
python src/67_day08_test_fewshot.py
```

### 실행 결과 해석

이번 Few-shot 검사는 Common Query 20장을 검사했습니다.

| 구분         |      결과 |
| ---------- | ------: |
| Normal 정답  | 10 / 10 |
| Anomaly 정답 |  8 / 10 |
| 전체 정답      | 18 / 20 |
| Accuracy   | **90%** |
| FP         |      0건 |
| FN         |      2건 |
즉 **20장 중 18장을 올바르게 판정하고 2장을 잘못 판정**했습니다.

특히 One-shot에서 잘못 판정했던 정상 이미지:

```
normal/006.png
```

는 Few-shot에서는:

```
normal_sim  = 0.968309
anomaly_sim = 0.963833

→ Normal Prototype과 더 비슷함
→ Normal 판정
→ 정답
```

으로 개선되었습니다.

반면 다음 두 불량 이미지는 Normal로 잘못 판정되었습니다.

```
anomaly/broken_small/000.png
anomaly/broken_small/007.png
```

따라서 이번 결과에서는:

```
One-shot Accuracy = 85%
        ↓
Few-shot Accuracy = 90%
```

로 성능이 조금 향상되었습니다.

하지만 Few-shot도 모든 불량을 완벽하게 구분하지는 못했습니다.

> 여러 Support 이미지로 만든 Prototype을 사용하면서 정상 판정은 개선되었지만, 일부 불량은 여전히 정상과 Feature가 비슷하여 놓칠 수 있음을 확인했습니다.

지금 단계에서는 오판 2장의 원인을 모두 자세히 분석하기보다, Few-shot도 한계가 있다는 점만 확인하고 이후 One-shot과 Few-shot 결과를 비교하는 단계로 넘어가면 충분합니다.

### 코드 전체 흐름

이번 코드는 앞에서 만든 두 Prototype을 불러와 **Common Query 20장을 실제로 검사하는 코드**입니다.

전체 흐름은 다음과 같습니다.

```
프로그램 시작
        ↓
Common Query 경로 지정
        ↓
Few-shot Prototype 파일 경로 지정
        ↓
결과 CSV 저장 경로 지정
        ↓

fewshot_prototypes.npz 불러오기
        ↓
Normal Prototype 읽기
Anomaly Prototype 읽기
        ↓

Pretrained ResNet18 준비
        ↓

결과를 저장할 빈 목록 생성
        ↓

Common Query 이미지 한 장 읽기
        ↓
ResNet18로 Feature 추출
        ↓

Query Feature
        ↓
        ┌────────────────────┐
        ↓                    ↓
Normal Prototype        Anomaly Prototype
        ↓                    ↓
Cosine Similarity       Cosine Similarity
계산                     계산
        ↓                    ↓
normal_similarity       anomaly_similarity
        └─────────┬──────────┘
                  ↓
              두 값 비교
                  ↓

normal_similarity >= anomaly_similarity ?
        ↓
     ┌──┴──┐
    YES    NO
     ↓      ↓
  Normal  Anomaly
     └──┬───┘
        ↓
실제 정답 actual과 비교
        ↓
correct = True / False
        ↓
결과 목록에 저장
        ↓

Normal Query 10장 반복
        ↓
Anomaly Query 10장 반복
        ↓
총 20장 검사 완료
        ↓

day08_fewshot_results.csv 저장
        ↓
화면에 판정 결과 출력
        ↓
프로그램 종료
```

이 코드에서 가장 중요한 부분은 다음입니다.

```
Query Feature
        ↓
Normal Prototype과 비교

Query Feature
        ↓
Anomaly Prototype과 비교
        ↓
두 Similarity 중
더 높은 쪽 선택
```

즉 Few-shot에서는 One-shot처럼 Threshold 하나를 기준으로 판정하지 않고, 두 Prototype 중 
어느 쪽과 더 비슷한지를 비교하여 판정합니다.

### 의사코드

```
1. Common Query 이미지가 저장된
   폴더 위치를 지정한다.

2. 앞에서 만든
   fewshot_prototypes.npz 파일 위치를 지정한다.

3. 검사 결과를 저장할
   CSV 파일 위치를 지정한다.

4. fewshot_prototypes.npz 파일을 불러온다.

5. 파일 안에서
   Normal Prototype을 가져온다.

6. 파일 안에서
   Anomaly Prototype을 가져온다.

7. 이미지 Feature를 추출하기 위해
   Pretrained ResNet18을 준비한다.

8. 검사 결과를 저장할
   빈 목록을 만든다.


9. Common Query에서
   이미지 한 장을 가져온다.

10. 이미지에서
    ResNet18 Feature를 추출한다.

11. Query Feature와
    Normal Prototype의
    Cosine Similarity를 계산한다.

12. Query Feature와
    Anomaly Prototype의
    Cosine Similarity를 계산한다.


13. 두 Similarity를 비교한다.

14. Normal Similarity가
    Anomaly Similarity보다 크거나 같으면

    predicted = Normal

    로 판정한다.

15. Anomaly Similarity가 더 높으면

    predicted = Anomaly

    로 판정한다.


16. 실제 정답 actual과
    예측 결과 predicted를 비교한다.

17. 두 값이 같으면

    correct = True

    로 기록한다.

18. 두 값이 다르면

    correct = False

    로 기록한다.


19. 다음 내용을 결과 목록에 저장한다.

    - sample_id
    - file
    - actual
    - normal_similarity
    - anomaly_similarity
    - predicted
    - correct


20. Normal Query 10장을
    같은 방식으로 모두 검사한다.

21. Anomaly Query 10장도
    같은 방식으로 모두 검사한다.

22. 총 20장의 검사 결과를
    CSV 파일로 저장한다.

23. 각 이미지의

    실제 정답
    예측 결과
    Normal Similarity
    Anomaly Similarity
    정답 여부

    를 화면에도 출력한다.

24. 저장된 CSV를 이용하여
    Few-shot 결과를 확인한다.
```

---

# 31. # Few-shot 결과를 읽는 방법

30번에서 Common Query 20장을 검사한 결과가 다음 파일에 저장되었습니다.

```
reports/day08_fewshot_results.csv
```

앞부분을 확인합니다.

```bash
head -n 11 reports/day08_fewshot_results.csv
```

실행 결과:

```
sample_id,file,actual,normal_similarity,anomaly_similarity,predicted,correct
normal/000.png,000.png,Normal,0.990703,0.98462,Normal,True
normal/001.png,001.png,Normal,0.993053,0.986839,Normal,True
normal/002.png,002.png,Normal,0.990798,0.987417,Normal,True
normal/003.png,003.png,Normal,0.992335,0.985275,Normal,True
normal/004.png,004.png,Normal,0.99038,0.983978,Normal,True
normal/005.png,005.png,Normal,0.993298,0.983524,Normal,True
normal/006.png,006.png,Normal,0.968309,0.963833,Normal,True
normal/007.png,007.png,Normal,0.992056,0.983506,Normal,True
normal/008.png,008.png,Normal,0.989684,0.980107,Normal,True
normal/009.png,009.png,Normal,0.991442,0.985416,Normal,True
```

## 각 열의 의미

|열|의미|
|---|---|
|`sample_id`|검사한 Common Query 이미지|
|`file`|실제 파일명|
|`actual`|실제 정답|
|`normal_similarity`|Normal Prototype과의 유사도|
|`anomaly_similarity`|Anomaly Prototype과의 유사도|
|`predicted`|Few-shot이 판정한 결과|
|`correct`|실제 정답과 예측이 같은지 여부|

Few-shot에서는 `normal_similarity`와 `anomaly_similarity`를 직접 비교합니다.

---

## 실제 결과 한 줄을 읽어봅니다

다음 결과를 확인합니다.

```
normal/006.png,006.png,Normal,0.968309,0.963833,Normal,True
```

이 값은 다음과 같이 읽습니다.

```
Actual
= Normal

Normal Similarity
= 0.968309

Anomaly Similarity
= 0.963833
```

두 값을 비교하면:

```
0.968309
>
0.963833
```

이므로 Normal Prototype과 조금 더 비슷합니다.

```
Normal Prototype 쪽이 더 가까움
        ↓
Predicted = Normal
        ↓
Actual = Normal
        ↓
correct = True
```

즉 이 이미지는 Few-shot에서 올바르게 판정되었습니다.

---

## One-shot 결과와 연결해서 봅니다

`normal/006.png`는 앞의 One-shot에서는:

```
Actual    = Normal
Predicted = Anomaly
correct   = False
```

였습니다.

하지만 Few-shot에서는:

```
Actual    = Normal
Predicted = Normal
correct   = True
```

가 되었습니다.

즉 같은 이미지라도 판정 기준이 달라지면 결과가 달라질 수 있습니다.

```
One-shot
Reference 1장과 비교
        ↓
오판

Few-shot
Normal / Anomaly Prototype과 비교
        ↓
정답
```

이번 사례에서는 여러 Support 이미지로 만든 Prototype을 사용하면서 `normal/006.png`가 정상 쪽으로 올바르게 판정되었습니다.

---

## Few-shot 결과를 읽는 핵심

Few-shot에서는 다음 한 가지를 기억하면 됩니다.

```
normal_similarity
>
anomaly_similarity
        ↓
Normal

normal_similarity
<
anomaly_similarity
        ↓
Anomaly
```

그리고 마지막으로:

```
predicted == actual
        ↓
correct = True

predicted != actual
        ↓
correct = False
```

인지 확인합니다.

> 31 단락에서는 두 Similarity를 비교하여 Few-shot이 왜 Normal 또는 Anomaly로 판정했는지 읽을 수 있으면 충분합니다.

---

# 32. Few-shot 결과와 한계를 정리합니다

이번 Few-shot에서는:

```text
Normal Support 5장 
Anomaly Support 5장 
Anomaly Type broken_small
```

을 이용하여 두 Prototype을 만들었습니다.

같은 Common Query 20장을 검사한 결과:

```
Normal 정답   10 / 10
Anomaly 정답   8 / 10

전체 정답      18 / 20
Accuracy       90%
```

이었습니다.

One-shot에서 오판했던 `normal/006.png`가 Few-shot에서는 정상으로 올바르게 판정되면서 전체 정확도도:

```
One-shot
85%

    ↓

Few-shot
90%
```

로 향상되었습니다.

하지만 불량 2장은 여전히 `Normal`로 잘못 판정되었습니다.

즉 여러 장의 Support를 사용한다고 해서 모든 불량을 완벽하게 구분할 수 있는 것은 아닙니다.

특히 이번 Anomaly Prototype은:

```
broken_small 5장
        ↓
Feature 평균
        ↓
Anomaly Prototype
```

으로 만든 기준입니다.

따라서 이것은 모든 종류의 불량을 대표하는 기준이 아니라, **이번 실습에서 사용한 `broken_small` 몇 장의 특징을 대표하는 기준**으로 이해해야 합니다.

> **Few-shot은 One-shot보다 여러 예제를 이용해 기준을 만들 수 있다는 장점이 있지만, 어떤 Support 데이터를 사용했는가에 따라 결과가 달라질 수 있습니다.**

PART4에서는 여기까지 확인합니다.

다음 단계에서는 같은 Common Query에서 얻은 One-shot과 Few-shot 결과를 나란히 비교하여 어떤 방법이 더 잘 동작했는지 확인합니다.

---

# PART5 — 같은 Common Query에서 One-shot / Few-shot 비교 (33~37)

앞에서 같은 Common Query 20장을 이용하여 두 방법을 각각 검사했습니다.

```
One-shot
17 / 20 정답
Accuracy = 85%

Few-shot
18 / 20 정답
Accuracy = 90%
```

겉으로 보면 Few-shot이 더 높은 결과를 보였습니다.

하지만 단순히 Accuracy 숫자만 비교하기 전에 먼저 확인해야 할 것이 있습니다.

> **두 방법이 정말 같은 이미지를 검사했는가?**

그리고 그다음에는 Accuracy뿐 아니라 **어떤 오류가 줄었고 어떤 오류가 남았는지** 함께 비교합니다.

---

# 33. 두 결과가 같은 Common Query에서 나온 것인지 확인합니다

One-shot 결과는 다음 파일에 저장되어 있습니다.

```
reports/day08_one_shot_results.csv
```

Few-shot 결과는:

```
reports/day08_fewshot_results.csv
```

에 저장되어 있습니다.

두 실험은 모두 동일한:

```
samples/day08/common_query
```

를 사용했습니다.

즉 검사 대상은 모두:

```
Normal 10장
+
Anomaly 10장

총 20장
```

으로 동일합니다.

따라서 두 결과를 공정하게 비교하려면 각 CSV의 `sample_id`가 서로 같아야 합니다.

```
One-shot sample_id
        =
Few-shot sample_id
```

이어야 합니다.

예를 들어:

```
normal/006.png

anomaly/broken_small/007.png
```

와 같은 동일한 이미지가 두 방법에서 각각 어떻게 판정되었는지를 비교하게 됩니다.

다음 비교 프로그램에서는 이 `sample_id`가 서로 일치하는지 먼저 자동으로 확인합니다.

> **같은 Query를 사용했기 때문에 One-shot 85%와 Few-shot 90%를 직접 비교할 수 있습니다.**

---

# 34. Accuracy뿐 아니라 FP와 FN도 함께 비교합니다

현재 결과는 다음과 같습니다.

```
One-shot
Accuracy = 85%

Few-shot
Accuracy = 90%
```

Few-shot이 5%p 높아졌습니다.

하지만 Vision AI 검사에서는 Accuracy 하나만 보는 것으로는 충분하지 않습니다.

어떤 이미지를 틀렸는지도 함께 확인해야 합니다.

이번 실습에서는 **Anomaly를 Positive**로 정의합니다.

```
TP
실제 Anomaly
→ Anomaly로 올바르게 판정

TN
실제 Normal
→ Normal로 올바르게 판정

FP
실제 Normal
→ Anomaly로 잘못 판정

FN
실제 Anomaly
→ Normal로 잘못 판정
```

이번 실제 결과를 보면:

```
One-shot

Normal 정답   9 / 10
Anomaly 정답  8 / 10

FP = 1
FN = 2
Accuracy = 85%
```

Few-shot은:

```
Few-shot

Normal 정답   10 / 10
Anomaly 정답   8 / 10

FP = 0
FN = 2
Accuracy = 90%
```

이었습니다.

즉 Few-shot에서는:

```
One-shot FP = 1
        ↓
Few-shot FP = 0
```

으로 정상 이미지를 잘못 불량으로 판단했던 오탐이 줄었습니다.

반면:

```
One-shot FN = 2
Few-shot FN = 2
```

로 불량을 정상으로 놓친 경우는 여전히 남았습니다.

따라서 이번 결과는 단순히:

```
85% → 90%
```

만 보는 것보다:

```
정상 오탐은 줄었음
        ↓
하지만 불량 미검출은 여전히 존재
```

라고 해석하는 것이 더 중요합니다.

---

## 이번 비교에서 확인할 성능 지표

다음 비교 코드에서는 다음 값을 계산합니다.

```
Accuracy
→ 전체 이미지 중 맞춘 비율

Anomaly Precision
→ Anomaly라고 판정한 것 중
   실제 Anomaly인 비율

Anomaly Recall
→ 실제 Anomaly 중
   올바르게 찾아낸 비율
```

특히 제조 검사에서는 `FN`이 중요합니다.

```
FN
실제 불량
        ↓
Normal로 판정
        ↓
불량을 놓침
```

이기 때문입니다.

따라서 이번 비교에서는:

> 어느 방법의 Accuracy가 높은가?

뿐 아니라:

> 어느 방법이 정상 오탐을 줄였는가?

> 어느 방법이 실제 불량을 더 잘 찾아냈는가?

까지 함께 확인합니다.

---

## PART5에서 비교할 핵심

```
같은 Common Query
        ↓
One-shot 결과
        VS
Few-shot 결과
        ↓
Accuracy 비교
        ↓
FP 비교
        ↓
FN 비교
        ↓
Precision / Recall 비교
        ↓
어떤 방법이
어떤 오류를 줄였는지 해석
```

> 이번 비교의 목적은 단순히 85%와 90% 중 더 큰 숫자를 고르는 것이 아니라, One-shot과 Few-shot이 어떤 종류의 오류에서 차이를 보였는지 이해하는 것입니다.

---

# 35. `68_day08_compare_methods.py` — 두 방법을 같은 기준으로 비교합니다

# 두 방법을 같은 기준으로 비교합니다

앞에서 같은 Common Query 20장을 두 가지 방법으로 각각 검사했습니다.

```
One-shot
Accuracy = 85%

Few-shot
Accuracy = 90%
```

숫자만 보면 Few-shot이 더 좋아 보입니다.

하지만 두 방법을 제대로 비교하려면 단순히 Accuracy 하나만 보는 것이 아니라:

```
정말 같은 Query를 검사했는가?

정상을 얼마나 잘 구분했는가?

불량을 얼마나 잘 찾아냈는가?

어떤 종류의 오판이 발생했는가?
```

를 같은 기준으로 확인해야 합니다.

그래서 이번에는 앞에서 저장한 두 결과 CSV를 다시 읽어 **같은 방식으로 성능을 계산하고 비교하는 프로그램**을 작성합니다.

---

## 이번 코드는 왜 작성할까요?

One-shot과 Few-shot은 각각 이미 실행을 끝냈습니다.

결과도 다음 파일에 저장되어 있습니다.

```
One-shot
reports/day08_one_shot_results.csv

Few-shot
reports/day08_fewshot_results.csv
```

이제 사람이 CSV를 한 줄씩 세면서 비교할 수도 있지만, 이미지가 많아지면 실수하기 쉽습니다.

따라서 프로그램이 자동으로:

```
두 CSV 불러오기
        ↓
같은 Query인지 확인
        ↓
One-shot 성능 계산
        ↓
Few-shot 성능 계산
        ↓
두 결과를 같은 형식으로 정리
        ↓
비교 CSV 저장
```

하도록 만듭니다.

> 이번 코드는 새로운 이미지를 검사하는 코드가 아닙니다.  
> **이미 만들어진 One-shot과 Few-shot 결과를 모아서 비교하는 분석 코드입니다.**

---

## 가장 먼저 같은 Query인지 확인합니다

비교하기 전에 두 결과의 `sample_id`가 같은지 확인합니다.

```
One-shot에서 검사한 이미지
        =
Few-shot에서 검사한 이미지
```

여야 공정한 비교가 가능합니다.

만약 서로 다른 이미지가 들어 있다면 프로그램은 비교를 계속하지 않고 오류를 발생시킵니다.

```
Common Query가 다름
        ↓
비교 중단
```

즉 서로 다른 시험 문제를 풀고 나온 점수를 비교하는 일을 막는 것입니다.

---

## 어떤 성능을 계산할까요?

각 방법에 대해 다음 값을 계산합니다.

```
Accuracy
→ 전체 20장 중 몇 장을 맞혔는가?

Anomaly Precision
→ Anomaly라고 판정한 이미지 중
   실제 Anomaly는 얼마나 되는가?

Anomaly Recall
→ 실제 Anomaly 중
   몇 장을 찾아냈는가?

TP / TN / FP / FN
→ 어떤 종류의 정답과 오판이 발생했는가?
```

앞에서 살펴본 실제 결과도 이 계산을 통해 다시 정리됩니다.

```
One-shot
Accuracy = 85%
FP = 1
FN = 2

Few-shot
Accuracy = 90%
FP = 0
FN = 2
```

이번 프로그램은 이러한 값을 사람이 직접 세지 않고 자동으로 계산합니다.

---

## 실행이 끝나면 무엇이 만들어질까요?

비교 결과는 다음 파일에 저장합니다.

```
reports/day08_method_comparison.csv
```

이 파일에는 One-shot과 Few-shot이 같은 형식으로 정리됩니다.

```
method
samples
accuracy
anomaly_precision
anomaly_recall
tp
tn
fp
fn
```

따라서 다음 36번에서는 이 CSV를 보고:

```
어느 방법의 Accuracy가 높은가?

FP는 줄었는가?

FN은 줄었는가?

Anomaly Recall은 어떻게 달라졌는가?
```

를 비교할 수 있습니다.

---

## 이번 코드의 전체 역할

```
One-shot 결과 CSV
        ↓
        ┐
        │
        ├→ 같은 Query인지 확인
        │
Few-shot 결과 CSV
        ↓
        ↓
각 방법의 성능 계산
        ↓
Accuracy
Precision
Recall
TP / TN / FP / FN
        ↓
같은 형식으로 정리
        ↓
day08_method_comparison.csv 저장
```

> **35번의 목적은 One-shot과 Few-shot을 다시 실행하는 것이 아니라, 이미 얻은 두 결과를 같은 기준으로 계산하여 공정하게 비교하는 것입니다.**

---

## 파일을 작성합니다

```text
src/68_day08_compare_methods.py
```

작성합니다.

```python
import csv
from pathlib import Path

import pandas as pd


ONE_PATH = Path("reports/day08_one_shot_results.csv")
FEW_PATH = Path("reports/day08_fewshot_results.csv")
OUTPUT_PATH = Path("reports/day08_method_comparison.csv")


one = pd.read_csv(ONE_PATH)
few = pd.read_csv(FEW_PATH)


if set(one["sample_id"]) != set(few["sample_id"]):
    raise RuntimeError(
        "One-shot과 Few-shot의 Common Query가 서로 다릅니다."
    )


def metrics(df):
    actual = df["actual"]
    predicted = df["predicted"]

    tp = int(((actual == "Anomaly") & (predicted == "Anomaly")).sum())
    tn = int(((actual == "Normal") & (predicted == "Normal")).sum())
    fp = int(((actual == "Normal") & (predicted == "Anomaly")).sum())
    fn = int(((actual == "Anomaly") & (predicted == "Normal")).sum())

    total = tp + tn + fp + fn

    accuracy = (tp + tn) / total if total else 0.0
    precision = tp / (tp + fp) if (tp + fp) else 0.0
    recall = tp / (tp + fn) if (tp + fn) else 0.0

    return {
        "samples": total,
        "accuracy": accuracy,
        "anomaly_precision": precision,
        "anomaly_recall": recall,
        "tp": tp,
        "tn": tn,
        "fp": fp,
        "fn": fn,
    }


rows = []

for method, df in [
    ("one_shot", one),
    ("few_shot", few),
]:
    result = metrics(df)
    result["method"] = method
    rows.append(result)


fieldnames = [
    "method",
    "samples",
    "accuracy",
    "anomaly_precision",
    "anomaly_recall",
    "tp",
    "tn",
    "fp",
    "fn",
]

OUTPUT_PATH.parent.mkdir(parents=True, exist_ok=True)

with OUTPUT_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:
    writer = csv.DictWriter(
        file,
        fieldnames=fieldnames,
    )
    writer.writeheader()

    for row in rows:
        output = dict(row)
        output["accuracy"] = round(output["accuracy"], 4)
        output["anomaly_precision"] = round(output["anomaly_precision"], 4)
        output["anomaly_recall"] = round(output["anomaly_recall"], 4)
        writer.writerow(output)


print("=" * 70)
print("DAY08 METHOD COMPARISON")
print("=" * 70)

for row in rows:
    print()
    print("Method            :", row["method"])
    print("Samples           :", row["samples"])
    print("Accuracy          :", f"{row['accuracy']:.4f}")
    print("Anomaly Precision :", f"{row['anomaly_precision']:.4f}")
    print("Anomaly Recall    :", f"{row['anomaly_recall']:.4f}")
    print("TP / TN / FP / FN :", row["tp"], row["tn"], row["fp"], row["fn"])

print()
print("Saved:", OUTPUT_PATH)
```

실행합니다.

```bash
python src/68_day08_compare_methods.py
```

### 실행결과 해석

```
One-shot
Accuracy          = 0.8500
Anomaly Precision = 0.8889
Anomaly Recall    = 0.8000
TP / TN / FP / FN = 8 / 9 / 1 / 2

Few-shot
Accuracy          = 0.9000
Anomaly Precision = 1.0000
Anomaly Recall    = 0.8000
TP / TN / FP / FN = 8 / 10 / 0 / 2
```

핵심만 정리하면 다음과 같습니다.

| 항목                | One-shot | Few-shot |
| ----------------- | -------- | -------- |
| Accuracy          | 85%      | **90%**  |
| Anomaly Precision | 88.89%   | **100%** |
| Anomaly Recall    | 80%      | 80%      |
| FP                | 1        | **0**    |
| FN                | 2        | 2        |

| 성능 지표                 | 아주 쉽게 말하면                         |
| --------------------- | --------------------------------- |
| **Accuracy**          | 전체 이미지 중 **맞춘 비율**                |
| **Anomaly Precision** | Anomaly라고 판정한 것들 중 실제 Anomaly인 비율 |
| **Anomaly Recall**    | 실제 Anomaly들 중 제대로 찾아낸 비율          |
| **FP**                | 실제 Normal인데 Anomaly라고 잘못 판정       |
| **FN**                | 실제 Anomaly인데 Normal이라고 잘못 판정      |

이번 실험에서는 Few-shot의 Accuracy가:

```
85%
 ↓
90%
```

로 향상되었습니다.

특히 One-shot에서는 정상 이미지 한 장을 `Anomaly`로 잘못 판정했지만:

```
One-shot
FP = 1
```

Few-shot에서는:

```
Few-shot
FP = 0
```

으로 정상 이미지 오탐이 사라졌습니다.

반면 실제 불량을 `Normal`로 놓친 경우는:

```
One-shot FN = 2
Few-shot FN = 2
```

로 동일했습니다.

따라서 이번 결과는 다음처럼 해석할 수 있습니다.

> Few-shot은 정상 이미지의 오탐을 줄여 전체 Accuracy는 향상시켰지만, 불량을 놓치는 문제까지 개선하지는 못했습니다.

즉 `90%가 85%보다 높다`는 것만 보는 것이 아니라, 어떤 오류가 줄었고 어떤 오류가 남았는지 함께 확인하는 것이 중요합니다.

---

## 이 코드의 전체 흐름

이번 코드는 새로운 이미지를 다시 검사하는 코드가 아닙니다.

앞에서 이미 만들어진:

```
One-shot 결과 CSV
Few-shot 결과 CSV
```

를 불러와 같은 기준으로 성능을 계산하고 비교합니다.

전체 흐름은 다음과 같습니다.

```
프로그램 시작
        ↓
One-shot 결과 CSV 불러오기
        ↓
Few-shot 결과 CSV 불러오기
        ↓
두 결과의 sample_id 확인
        ↓
같은 Common Query인가?
        ↓
      ┌─────────────┐
      │             │
     YES            NO
      ↓              ↓
비교 계속        오류 발생
      ↓
One-shot
actual / predicted 확인
      ↓
TP / TN / FP / FN 계산
      ↓
Accuracy 계산
Precision 계산
Recall 계산
      ↓

Few-shot
actual / predicted 확인
      ↓
TP / TN / FP / FN 계산
      ↓
Accuracy 계산
Precision 계산
Recall 계산
      ↓

두 결과를 같은 형식으로 정리
      ↓
화면에 비교 결과 출력
      ↓
day08_method_comparison.csv 저장
      ↓
프로그램 종료
```

---

## 의사코드로 이해합니다

```
1. One-shot 결과 CSV의 위치를 지정한다.

2. Few-shot 결과 CSV의 위치를 지정한다.

3. 비교 결과를 저장할 CSV 위치를 지정한다.


4. One-shot 결과를 읽는다.

5. Few-shot 결과를 읽는다.


6. 두 결과의 sample_id를 비교한다.

7. 검사한 Query가 서로 다르면
   공정한 비교를 할 수 없으므로
   실행을 중단한다.


8. 성능을 계산하는 함수를 만든다.

9. 실제 정답 actual과
   예측 결과 predicted를 확인한다.


10. 실제 Anomaly를
    Anomaly로 맞힌 수를 계산한다.

    → TP

11. 실제 Normal을
    Normal로 맞힌 수를 계산한다.

    → TN

12. 실제 Normal을
    Anomaly로 잘못 판정한 수를 계산한다.

    → FP

13. 실제 Anomaly를
    Normal로 놓친 수를 계산한다.

    → FN


14. TP + TN + FP + FN을 이용해
    전체 이미지 수를 계산한다.

15. 전체 중 맞힌 비율을 계산한다.

    → Accuracy

16. Anomaly라고 판정한 것 중
    실제 Anomaly 비율을 계산한다.

    → Anomaly Precision

17. 실제 Anomaly 중
    찾아낸 비율을 계산한다.

    → Anomaly Recall


18. 같은 계산을
    One-shot에 적용한다.

19. 같은 계산을
    Few-shot에 적용한다.


20. 두 방법의 결과를
    같은 형식으로 정리한다.

21. 다음 값을 저장한다.

    - method
    - samples
    - accuracy
    - anomaly_precision
    - anomaly_recall
    - tp
    - tn
    - fp
    - fn


22. 비교 결과를

    reports/day08_method_comparison.csv

    파일로 저장한다.

23. 화면에도
    One-shot과 Few-shot 결과를 출력한다.
```

---

## 이번 코드에서 꼭 이해할 부분

이번 코드의 핵심은 다음과 같습니다.

```
두 방법을 다시 실행
        X

이미 저장된 두 결과를 읽어서
같은 기준으로 성능 비교
        O
```

그리고 비교할 때는 단순히:

```
85% vs 90%
```

만 보는 것이 아니라:

```
Accuracy
Precision
Recall
FP
FN
```

을 함께 확인합니다.

> 이번 결과에서는 Few-shot이 정상 오탐은 줄였지만, 불량 미검출은 여전히 남아 있다는 점을 확인했습니다.

---

# 36. 비교 결과에서 숫자 하나만 보지 않습니다

결과 파일:

```text
reports/day08_method_comparison.csv
```

을 확인합니다.

```bash
cat reports/day08_method_comparison.csv
```

다음 네 가지를 함께 봅니다.

```text
Accuracy
Anomaly Precision
Anomaly Recall
FP / FN
```

예를 들어 Accuracy가 같아도:

```text
방법 A
FP가 많음

방법 B
FN이 많음
```

이면 제조 검사에서 의미가 다릅니다.

불량을 놓치는 FN이 더 위험한 공정도 있고, 정상 제품을 계속 불량 처리하는 FP 비용이 큰 공정도 있습니다.

---

# 37. 7일차와 8일차 숫자를 단순 순위로 비교하지 않습니다

7일차 Anomaly Detection과 오늘 One/Few-shot은 데이터 구성과 판정 방식이 다릅니다.

```text
7일차
정상 Train 여러 장으로 기준 생성

8일차 One-shot
정상 Reference 1장

8일차 Few-shot
Normal 5 + broken_small 5
```

따라서:

```text
Accuracy가 가장 높은 방법 = 항상 가장 좋은 방법
```

이라고 결론내리지 않습니다.

먼저 실제 데이터 조건에 맞는 방법인지 확인해야 합니다.

---

# PART6 — Defect Generation: Ground Truth Mask + Cut-Paste (38~45)

앞의 PART에서는 적은 데이터로 검사하는 방법을 확인했습니다.

```
One-shot
Reference 1장
        ↓
Accuracy 85%

Few-shot
Normal Support 5장
Anomaly Support 5장
        ↓
Accuracy 90%
```

Few-shot에서는 여러 예제를 사용하면서 일부 결과가 개선되었습니다.

하지만 여기서 또 다른 문제가 생길 수 있습니다.

> 실제 불량 이미지 자체를 충분히 구할 수 없다면 어떻게 해야 할까요?

PART6에서는 이 문제를 다룹니다.

새로운 모델을 학습하는 것이 아니라, 실제 결함을 이용해 합성 불량 이미지를 만드는 가장 기본적인 방법을 실습합니다.

---

# 38. 왜 Defect Generation을 하나요?

실제 제조현장에서는 정상 제품은 계속 생산되기 때문에 정상 이미지는 비교적 많이 확보할 수 있습니다.

하지만 특정 불량은 자주 발생하지 않을 수 있습니다.

```
Normal
많이 확보 가능

Defect
적게 확보될 수 있음
```

예를 들어 Few-shot에서 불량 Support를 더 늘리고 싶어도 실제 불량 이미지가 몇 장밖에 없다면 데이터를 쉽게 늘리기 어렵습니다.

이럴 때 생각할 수 있는 방법 중 하나가 **Defect Generation**입니다.

오늘은 실제 불량 이미지에서 결함 부분만 가져와 정상 이미지에 붙여 봅니다.

```
실제 불량 이미지
        ↓
결함 부분만 추출
        ↓
Defect Patch

        +

정상 이미지
        ↓

Defect Patch 붙이기
        ↓
Synthetic Defect
합성 불량 이미지
```

단순히 똑같은 결함을 복사하는 것에서 끝나지 않고 크기와 회전, 위치를 조금씩 변경합니다.

```
실제 Defect Patch
        ↓
크기 변경
회전 변경
위치 변경
        ↓
정상 이미지에 합성
        ↓
여러 Synthetic Defect 생성
```

> PART6의 목적은 부족한 실제 불량 데이터를 보완하기 위해 합성 불량 이미지를 만드는 기본 과정을 이해하는 것입니다.

여기서 만든 Synthetic 이미지는 실제 데이터와 완전히 같다고 가정하지 않습니다. 뒤에서 직접 결과를 확인하면서 합성 품질도 함께 살펴봅니다.

---

# 39. 결함 부분은 어떻게 정확하게 찾을까요?

불량 이미지에서 결함 Patch를 잘라내려면 먼저:

```
이 이미지의 어느 부분이 실제 결함인가?
```

를 알아야 합니다.

MVTec AD에는 불량 이미지와 함께 **Ground Truth Mask**가 제공됩니다.

예를 들어 불량 이미지:

```
test/broken_small/000.png
```

에는 다음 Mask가 대응됩니다.

```
ground_truth/broken_small/000_mask.png
```

Mask에서는 결함 위치가 흰색으로 표시되어 있습니다.

쉽게 보면:

```
불량 원본 이미지
        +
Ground Truth Mask
결함 위치 표시
        ↓
Mask의 흰 영역 확인
        ↓
실제 결함 부분만 추출
        ↓
Defect Patch
```

입니다.

7일차에서는 Ground Truth Mask를 **Anomaly Heatmap이 실제 결함 위치를 잘 찾았는지 확인하는 정답**으로 사용했습니다.

8일차에서는 같은 Mask를 다른 목적으로 사용합니다.

```
7일차
Ground Truth Mask
→ 검출 위치가 맞는지 비교

8일차
Ground Truth Mask
→ 실제 결함 위치를 잘라내기
```

즉 이번에는 Mask를 이용하여 불량 이미지 전체가 아니라 **실제 결함 영역만 Patch로 추출**합니다.

---

# 40. 합성에 사용할 불량도 Query와 분리합니다

오늘 `broken_small` 이미지는 처음부터 역할을 나누어 준비했습니다.

```
broken_small

Support
5장
→ Few-shot 기준 생성

Query
10장
→ 실제 성능 평가

Cut-Paste Source
2장
→ Synthetic Defect 생성
```

여기서 Cut-Paste Source 2장은 **합성 데이터를 만들기 위한 재료**입니다.

Query 10장은 **성능을 평가하기 위한 시험 데이터**입니다.

따라서 두 역할을 섞지 않습니다.

```
Cut-Paste Source
→ 결함 Patch 추출에 사용

Common Query
→ 성능 평가에 사용
```

Cut-Paste Source로 사용한 이미지를 다시 Query에 넣지 않습니다. 현재 교재도 Support 5장, Query 10장, Cut-Paste Source 2장으로 역할을 분리하도록 구성되어 있습니다.

> 합성 데이터를 만드는 실습에서도 “데이터를 만드는 데 사용한 이미지”와 “평가에 사용하는 이미지”는 구분합니다.

---

# 41. `day08_cutpaste_core.py` — 결함을 잘라서 합성하는 공통 기능을 만듭니다

이제 실제 Cut-Paste 기능을 코드로 만듭니다.

그런데 이번 파일은 앞에서 작성했던 `65`, `67`, `68`번처럼 **직접 실행해서 결과를 확인하는 프로그램이 아닙니다.**

이번 파일의 역할은 다음 실습에서 사용할 **공통 기능을 미리 만들어 두는 것**입니다.

```
day08_cutpaste_core.py
        ↓
Cut-Paste에 필요한 기능 정의
        ↓
42번의 실행 프로그램이 불러와 사용
```

실제로 다음 42번의:

```
69_day08_make_cutpaste_defects.py
```

가 이 파일의 `generate_synthetic_set()` 함수를 불러와 Synthetic Defect 20장을 생성합니다.

---

## 이번 공통 파일은 왜 만들까요?

이번 PART에서는 실제 불량 이미지의 결함 부분을 잘라 정상 이미지에 붙여 **Synthetic Defect를 만드는 Cut-Paste 방식**을 사용합니다.

이 과정에는 다음과 같은 작업이 반복됩니다.

```
불량 이미지와 Mask 읽기
        ↓
결함 부분 추출
        ↓
크기·회전 변경
        ↓
정상 이미지에 붙이기
        ↓
합성 결과 저장
```

이 기능들은 이번 본 실습뿐 아니라 뒤의 Mini Challenge에서도 다시 사용하게 됩니다.

따라서 같은 코드를 여러 번 작성하지 않고, Cut-Paste에 필요한 기능을 하나의 공통 파일에 모아 둡니다.

```
day08_cutpaste_core.py
        ↓
Cut-Paste 공통 기능 정의
        ↓
본 실습에서 재사용
        ↓
Mini Challenge에서도 재사용
```

> **이번 파일은 합성 이미지를 바로 만들어 실행하는 프로그램이 아니라, 뒤에서 사용할 Cut-Paste 기능을 미리 준비하는 공통 도구 파일입니다.**

---

## 이번 코드가 전체적으로 할 일

코드를 작성하면 다음 작업을 할 수 있게 됩니다.

```
실제 불량 이미지
        +
Ground Truth Mask
        ↓
결함 부분 추출
        ↓
정상 이미지에 합성
        ↓
Synthetic Defect 생성
```

여기서는 먼저 이 기능들을 만들고, 다음 단계에서 실제로 Synthetic Defect 여러 장을 생성합니다.

---

파일을 작성합니다

```text
src/day08_cutpaste_core.py
```

작성합니다.

```python
import csv
import random
from pathlib import Path

import cv2
import numpy as np


def extract_patch(image_path, mask_path):
    image = cv2.imread(str(image_path))
    mask = cv2.imread(str(mask_path), cv2.IMREAD_GRAYSCALE)

    if image is None:
        raise FileNotFoundError(image_path)

    if mask is None:
        raise FileNotFoundError(mask_path)

    binary = np.where(mask > 0, 255, 0).astype(np.uint8)
    points = cv2.findNonZero(binary)

    if points is None:
        raise RuntimeError(f"Mask가 비어 있습니다: {mask_path}")

    x, y, w, h = cv2.boundingRect(points)

    patch = image[y:y + h, x:x + w].copy()
    patch_mask = binary[y:y + h, x:x + w].copy()

    return patch, patch_mask


def rotate_pair(patch, mask, angle):
    height, width = patch.shape[:2]
    center = (width / 2.0, height / 2.0)

    matrix = cv2.getRotationMatrix2D(
        center,
        angle,
        1.0,
    )

    rotated_patch = cv2.warpAffine(
        patch,
        matrix,
        (width, height),
        flags=cv2.INTER_LINEAR,
        borderMode=cv2.BORDER_REFLECT,
    )

    rotated_mask = cv2.warpAffine(
        mask,
        matrix,
        (width, height),
        flags=cv2.INTER_NEAREST,
        borderValue=0,
    )

    return rotated_patch, rotated_mask


def paste_patch(
    background,
    patch,
    mask,
    rng,
    scale_range=(0.7, 1.2),
    angle_range=(-15.0, 15.0),
):
    result = background.copy()

    scale = rng.uniform(*scale_range)
    angle = rng.uniform(*angle_range)

    new_width = max(4, int(patch.shape[1] * scale))
    new_height = max(4, int(patch.shape[0] * scale))

    patch = cv2.resize(
        patch,
        (new_width, new_height),
        interpolation=cv2.INTER_LINEAR,
    )

    mask = cv2.resize(
        mask,
        (new_width, new_height),
        interpolation=cv2.INTER_NEAREST,
    )

    patch, mask = rotate_pair(
        patch,
        mask,
        angle,
    )

    image_h, image_w = result.shape[:2]
    patch_h, patch_w = patch.shape[:2]

    if patch_w >= image_w or patch_h >= image_h:
        raise RuntimeError("Defect Patch가 정상 이미지보다 큽니다.")

    # MVTec bottle은 제품이 중앙에 있으므로
    # 교육용 Baseline에서는 중앙 영역을 우선 사용합니다.
    x_min = int(image_w * 0.25)
    x_max = int(image_w * 0.75) - patch_w

    y_min = int(image_h * 0.10)
    y_max = int(image_h * 0.90) - patch_h

    if x_max <= x_min:
        x_min = 0
        x_max = image_w - patch_w

    if y_max <= y_min:
        y_min = 0
        y_max = image_h - patch_h

    x = rng.randint(x_min, x_max)
    y = rng.randint(y_min, y_max)

    roi = result[
        y:y + patch_h,
        x:x + patch_w,
    ]

    alpha = mask.astype(np.float32) / 255.0
    alpha = cv2.GaussianBlur(alpha, (3, 3), 0)
    alpha = alpha[..., None]

    blended = (
        patch.astype(np.float32) * alpha
        + roi.astype(np.float32) * (1.0 - alpha)
    )

    result[
        y:y + patch_h,
        x:x + patch_w,
    ] = np.clip(blended, 0, 255).astype(np.uint8)

    info = {
        "x": x,
        "y": y,
        "width": patch_w,
        "height": patch_h,
        "rotation": round(angle, 2),
        "scale": round(scale, 3),
    }

    return result, info


def generate_synthetic_set(
    normal_dir,
    defect_image_dir,
    defect_mask_dir,
    output_dir,
    metadata_path,
    defect_type,
    count,
    seed,
    scale_range=(0.7, 1.2),
    angle_range=(-15.0, 15.0),
):
    normal_files = sorted(Path(normal_dir).glob("*.png"))
    defect_files = sorted(Path(defect_image_dir).glob("*.png"))

    if not normal_files:
        raise RuntimeError("Normal Target 이미지가 없습니다.")

    if not defect_files:
        raise RuntimeError("Defect Source 이미지가 없습니다.")

    output_dir = Path(output_dir)
    metadata_path = Path(metadata_path)

    output_dir.mkdir(parents=True, exist_ok=True)
    metadata_path.parent.mkdir(parents=True, exist_ok=True)

    # 재실행할 때 이전 synthetic 결과를 섞지 않습니다.
    for old_file in output_dir.glob("synthetic_*.png"):
        old_file.unlink()

    rng = random.Random(seed)
    rows = []

    for index in range(count):
        normal_path = rng.choice(normal_files)
        defect_path = rng.choice(defect_files)

        mask_path = (
            Path(defect_mask_dir)
            / f"{defect_path.stem}_mask.png"
        )

        background = cv2.imread(str(normal_path))

        if background is None:
            raise FileNotFoundError(normal_path)

        patch, mask = extract_patch(
            defect_path,
            mask_path,
        )

        synthetic, info = paste_patch(
            background,
            patch,
            mask,
            rng,
            scale_range=scale_range,
            angle_range=angle_range,
        )

        output_name = f"synthetic_{index:03d}.png"
        output_path = output_dir / output_name

        cv2.imwrite(str(output_path), synthetic)

        rows.append(
            {
                "synthetic_file": output_name,
                "source_normal": normal_path.name,
                "source_defect": defect_path.name,
                "defect_type": defect_type,
                "seed": seed,
                **info,
            }
        )

    with metadata_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=rows[0].keys(),
        )
        writer.writeheader()
        writer.writerows(rows)

    return rows
```

이 공통 함수는 다음 일을 담당합니다.

```text
Mask로 결함 위치 찾기
        ↓
Defect Patch 자르기
        ↓
크기 변경
        ↓
회전
        ↓
정상 이미지 중앙 영역에 배치
        ↓
Mask로 합성
        ↓
Metadata 반환
```

---

# 42. `69_day08_make_cutpaste_defects.py` — Synthetic Defect 20장을 만듭니다

41 단락에서는 Cut-Paste에 필요한 공통 기능을 만들었습니다.

```
결함 Patch 추출
크기 변경
회전
정상 이미지에 합성
Metadata 기록
```

하지만 `day08_cutpaste_core.py`는 **기능만 정의해 둔 파일**이므로, 아직 실제 Synthetic 이미지는 만들어지지 않았습니다.

이번에는 앞에서 만든 공통 기능을 불러와 **실제로 합성 불량 이미지 20장을 생성하는 실행 프로그램**을 작성합니다.

```
41번
day08_cutpaste_core.py
→ Cut-Paste 기능 준비

        ↓

42번
69_day08_make_cutpaste_defects.py
→ 준비한 기능을 실제로 실행
→ Synthetic Defect 20장 생성
```

---

## 이번 코드에서는 무엇을 사용하나요?

입력 데이터는 다음과 같습니다.

```
Normal Target
→ 정상 Bottle 이미지

Defect Source
→ broken_small 실제 불량 이미지

Ground Truth Mask
→ 실제 결함 위치
```

이 데이터를 이용하여:

```
정상 이미지 선택
        ↓
실제 결함 Patch 추출
        ↓
크기 / 회전 / 위치 변경
        ↓
정상 이미지에 합성
        ↓
Synthetic Defect 생성
```

을 반복합니다.

이번 실습에서는 총 **20장**을 생성합니다. 현재 코드도 `count=20`, `seed=42`, 크기 범위 `0.7~1.2`, 회전 범위 `-15°~15°`로 실행하도록 구성되어 있습니다.

---

## 실행하면 무엇이 만들어질까요?

합성 이미지는:

```
outputs/day08/cutpaste/generated
```

에 저장되고,

각 이미지가 어떤 조건으로 만들어졌는지는:

```
reports/day08_defect_metadata.csv
```

에 기록됩니다.

즉 이번 코드는:

> 41단락에서 만든 Cut-Paste 기능을 실제 데이터에 적용하여 Synthetic Defect 20장과 생성 기록을 만드는 실행 파일입니다.

---
파일을 작성합니다

```text
src/69_day08_make_cutpaste_defects.py
```

작성합니다.

```python
from pathlib import Path

from day08_cutpaste_core import (
    generate_synthetic_set,
)


NORMAL_DIR = Path("samples/day08/cutpaste/normal")

DEFECT_ROOT = Path(
    "samples/day08/cutpaste/defect_source/broken_small"
)

OUTPUT_DIR = Path("outputs/day08/cutpaste/generated")
METADATA_PATH = Path("reports/day08_defect_metadata.csv")


rows = generate_synthetic_set(
    normal_dir=NORMAL_DIR,
    defect_image_dir=DEFECT_ROOT / "images",
    defect_mask_dir=DEFECT_ROOT / "masks",
    output_dir=OUTPUT_DIR,
    metadata_path=METADATA_PATH,
    defect_type="broken_small",
    count=20,
    seed=42,
    scale_range=(0.7, 1.2),
    angle_range=(-15.0, 15.0),
)


print("=" * 70)
print("DAY08 CUT-PASTE DEFECT GENERATION")
print("=" * 70)
print("Defect Type      : broken_small")
print("Generated Images :", len(rows))
print("Output           :", OUTPUT_DIR)
print("Metadata         :", METADATA_PATH)
```

실행합니다.

```bash
python src/69_day08_make_cutpaste_defects.py
```

### 실행 결과를 해석합니다

```text
Defect Type      : broken_small
Generated Images : 20
Output           : outputs/day08/cutpaste/generated
Metadata         : reports/day08_defect_metadata.csv
```

이번 실행에서는 `broken_small` 실제 결함 Patch를 이용하여 Synthetic Defect 20장을 생성했습니다. 생성된 이미지는 `outputs/day08/cutpaste/generated`에 저장되고, 각 이미지가 어떤 원본과 조건으로 만들어졌는지는 `reports/day08_defect_metadata.csv`에 기록됩니다.

```
broken_small 결함 이용
        ↓
Synthetic Defect 20장 생성
        ↓
이미지 저장 완료
+
Metadata 저장 완료
```

여기서 확인한 것은 합성 이미지 생성이 정상적으로 완료되었다는 것입니다. 실제 불량처럼 자연스럽게 만들어졌는지는 다음 단계에서 이미지를 직접 확인합니다.

### ## 코드의 전체 흐름

이번 코드는 41번에서 만든 `generate_synthetic_set()` 함수를 실제 데이터에 적용하는 실행 코드입니다. 함수 내부에서는 정상 이미지와 불량 Source를 선택하고, 대응하는 Mask로 결함 Patch를 추출한 뒤 크기·회전·위치를 변경하여 합성하고 Metadata까지 저장합니다.

```
프로그램 시작
        ↓
Normal 이미지 폴더 지정
        ↓
broken_small Defect Source 지정
        ↓
Ground Truth Mask 위치 지정
        ↓
출력 이미지 폴더 지정
        ↓
Metadata 저장 경로 지정
        ↓
generate_synthetic_set() 실행
        ↓

정상 이미지 한 장 선택
        ↓
불량 Source 한 장 선택
        ↓
대응하는 Mask 찾기
        ↓
결함 Patch 추출
        ↓
Patch 크기 변경
        ↓
Patch 회전
        ↓
붙일 위치 선택
        ↓
정상 이미지에 합성
        ↓
Synthetic 이미지 저장
        ↓
생성 조건 Metadata 기록
        ↓
20장까지 반복
        ↓
생성 결과 출력
```

### 의사코드

```
1. 정상 이미지가 있는 폴더를 지정한다.

2. broken_small 불량 이미지가 있는
   폴더를 지정한다.

3. Ground Truth Mask가 있는
   폴더를 지정한다.

4. Synthetic 이미지를 저장할
   폴더를 지정한다.

5. Metadata를 저장할
   CSV 파일 위치를 지정한다.

6. generate_synthetic_set()을 실행한다.


7. 정상 이미지 중 한 장을 선택한다.

8. 실제 불량 이미지 중 한 장을 선택한다.

9. 선택한 불량 이미지에 대응하는
   Ground Truth Mask를 찾는다.

10. Mask를 이용하여
    실제 결함 Patch를 추출한다.

11. 결함 Patch의 크기를 변경한다.

12. 결함 Patch를 회전한다.

13. 정상 이미지에서
    Patch를 붙일 위치를 정한다.

14. 정상 이미지에
    결함 Patch를 합성한다.

15. 완성된 Synthetic Defect를
    이미지 파일로 저장한다.

16. 다음 정보를 Metadata에 기록한다.

    - 사용한 Normal 이미지
    - 사용한 Defect 이미지
    - Defect Type
    - 위치
    - 크기
    - 회전
    - Scale

17. 같은 과정을 반복하여
    총 20장을 생성한다.

18. 생성된 이미지 수와
    저장 위치를 화면에 출력한다.
```

핵심만 기억합니다.

```
41번
→ Cut-Paste 기능을 만듦

42번
→ 그 기능을 실제로 실행
→ Synthetic Defect 20장 생성
```

이번 단계의 목표는 실제 불량 Patch를 이용해 합성 불량 데이터를 만들어 보는 것입니다. 생성 품질이 좋은지는 다음 단계에서 직접 확인합니다.

---

# 43. 생성한 Synthetic Defect가 정상적으로 만들어졌는지 확인합니다

42번에서는 `broken_small` 결함을 이용하여 Synthetic Defect 20장을 만들었습니다.

먼저 실제로 20장이 생성되었는지 확인합니다.

```bash
find outputs/day08/cutpaste/generated -name "synthetic_*.png" | wc -l
```

정상적으로 생성되었다면:

```
20
```

이 출력됩니다.

이번에는 합성 이미지가 **어떤 원본과 조건으로 만들어졌는지** 기록한 Metadata도 확인합니다.

```bash
head -n 10 reports/day08_defect_metadata.csv
```

Metadata에는 다음과 같은 정보가 저장되어 있습니다.

```
synthetic_file
source_normal
source_defect
defect_type
x / y
width / height
rotation
scale
```

즉 Synthetic 이미지 한 장에 대해:

```
어떤 정상 이미지를 사용했는가?
        +
어떤 실제 결함을 사용했는가?
        +
어디에 붙였는가?
        +
얼마나 확대·축소하고 회전했는가?
```

를 다시 확인할 수 있습니다.

> 지금까지는 **합성 데이터가 만들어졌다는 사실과 생성 조건을 확인한 단계**입니다. 아직 이 이미지가 실제 학습이나 검사에 사용할 만큼 좋은 데이터인지는 판단하지 않았습니다.

---

# 44. Synthetic Defect는 사용하기 전에 직접 확인합니다

코드가 오류 없이 실행되어 20장이 만들어졌다고 해서 모두 좋은 데이터라는 뜻은 아닙니다.

예를 들어 다음과 같은 문제가 있을 수 있습니다.

```
결함이 제품 밖에 붙음

결함 크기가 너무 큼

회전이 부자연스러움

Patch 경계가 눈에 띔

실제 결함처럼 보이지 않음
```

따라서 생성 결과 중 대표 이미지 몇 장을 직접 열어 확인합니다.

```
outputs/day08/cutpaste/generated
```

![[{6569837F-6ED2-4C90-9D92-4F5CA7D7F7EC}.png]]

오늘은 모든 이미지를 정밀하게 평가하는 것이 아니라 대표 이미지 5장 정도를 확인하면 충분합니다.

### 생성된 Synthetic Defect의 품질을 확인합니다

42번에서 `broken_small` 결함을 이용하여 Synthetic Defect 20장을 생성했습니다.

이제 생성된 이미지를 직접 열어 봅니다.

이번 결과를 보면 이미지마다 결함의:

```
위치
크기
회전
경계
```

가 서로 다르게 만들어진 것을 확인할 수 있습니다.

하지만 모든 이미지가 실제 불량 데이터로 사용하기에 자연스러운 것은 아닙니다.

실제 생성 결과를 보면 일부 이미지는 결함 Patch가 병에 비해 상당히 크거나 길게 퍼져 있고, 밝은 경계가 강하게 보여 **합성한 흔적이 눈에 띄는 경우**도 있습니다.

따라서 생성 후에는 반드시 품질을 확인해야 합니다.

## 무엇을 확인할까요?

이번 결과에서는 다음 네 가지만 간단히 봅니다.

|확인 항목|확인할 내용|
|---|---|
|위치|결함이 Bottle 제품 영역 안에 자연스럽게 위치하는가?|
|크기|`broken_small`이라고 보기에는 지나치게 크지 않은가?|
|경계|Patch를 붙인 경계나 밝은 부분이 너무 눈에 띄지 않는가?|
|형태|전체적으로 실제 작은 파손처럼 보이는가?|

특히 이번 결과에서는 **크기와 경계**를 주의해서 봅니다.

```
결함 Patch가 너무 큼
        ↓
broken_small의 특성과 맞지 않을 수 있음

Patch 경계가 강하게 보임
        ↓
실제 결함보다
붙여 넣은 이미지처럼 보일 수 있음
```

---

## 실제 생성 결과를 예시로 확인합니다

예를 들어 다음과 같이 판단해 볼 수 있습니다.

|파일|Keep / Reject|간단한 이유|
|---|---|---|
|`synthetic_000.png`|Reject|결함이 너무 크고 길게 퍼져 있어 `broken_small`로 보기 어려움|
|`synthetic_004.png`|Keep 후보|비교적 작은 영역에 결함이 위치하며 전체 크기도 상대적으로 작음|
|`synthetic_007.png`|Reject|결함 영역이 크고 밝은 경계가 강하게 보여 합성 느낌이 큼|
|`synthetic_013.png`|Keep 후보|결함이 비교적 국소적으로 나타나고 크기도 상대적으로 작음|
|`synthetic_017.png`|Keep 후보|다른 결과보다 결함 크기가 작고 한 영역에 모여 있음|

여기서 `Keep 후보`라고 표현하는 이유가 중요합니다.

현재 화면만 보고는 실제 `broken_small` 원본과 완전히 같은 품질이라고 확정할 수 없습니다.

따라서:

```
1차
Synthetic 이미지만 보고
위치 / 크기 / 경계 확인

        ↓

2차
실제 broken_small 이미지와 비교

        ↓

최종
Keep / Reject 결정
```

하는 것이 더 정확합니다.

---

## 간단한 품질 기록 예시

다음과 같이 짧게 작성하면 충분합니다.

```
# Day08 Synthetic Quality Check

## 확인 대상

outputs/day08/cutpaste/generated

## 검수 결과

| 파일 | Keep / Reject | 간단한 이유 |
|---|---|---|
| synthetic_000.png | Reject | 결함이 너무 크고 길게 퍼져 있음 |
| synthetic_004.png | Keep | 결함이 비교적 작고 제품 영역 안에 위치함 |
| synthetic_007.png | Reject | 결함 크기가 크고 Patch 경계가 눈에 띔 |
| synthetic_013.png | Keep | 결함이 국소적으로 나타나 비교적 자연스러움 |
| synthetic_017.png | Keep | 결함 크기가 작고 한 영역에 모여 있음 |

## 확인 결과

일부 이미지는 비교적 자연스럽게 생성되었지만,
일부 이미지는 결함이 너무 크거나 Patch 경계가 눈에 띄었다.

따라서 Synthetic 이미지는 생성 후 바로 사용하지 않고
품질을 확인하여 사용할 이미지만 선택해야 한다.
```

---

## 문제가 보인다면 어떻게 개선할 수 있을까요?

이번 결과를 보고 다음과 같은 개선 방향을 생각할 수 있습니다.

```
결함이 너무 크다
→ scale_range를 더 작게 조정

위치가 부자연스럽다
→ 붙일 수 있는 영역을 더 제한

경계가 너무 눈에 띈다
→ Blending 방법 개선

형태가 다양하지 않다
→ 더 다양한 실제 Defect Source 사용
```

이번 8일차에서는 이 값들을 하나씩 수정하여 다시 실험하지는 않습니다.

Synthetic 데이터를 Few-shot에 추가하여 성능을 다시 비교하려면:

```
어떤 Synthetic 이미지를 사용할 것인가?

몇 장을 추가할 것인가?

Real과 Synthetic을 어떤 비율로 섞을 것인가?

같은 Query에서 성능이 실제로 개선되는가?
```

와 같은 **별도의 실험 조건을 다시 설계해야 하기 때문**입니다.

따라서 이번에는:

```
Synthetic 생성
        ↓
품질 확인
        ↓
Keep / Reject 판단
        ↓
개선 방법 생각하기
```

까지 진행합니다.

> 8일차의 목적은 합성데이터로 성능을 다시 높이는 것이 아니라, 불량 데이터가 부족할 때 Synthetic Data를 만들고 품질을 확인하는 기본 과정을 경험하는 것입니다.

---

# 45. Domain Gap을 이해하고 PART6을 마무리합니다

Synthetic Defect가 자연스럽게 보여도 실제 불량과 완전히 같지는 않을 수 있습니다.

실제 데이터와 합성 데이터 사이의 이러한 차이를 `Domain Gap`이라고 합니다.

```
Real Defect
        VS
Synthetic Defect

차이가 날 수 있는 요소

조명
질감
경계
색상
그림자
결함 위치
결함 크기
카메라 노이즈
```

따라서 합성데이터는:

```
생성
        ↓
품질 확인
        ↓
사용 가능한 데이터 선별
        ↓
필요한 경우
학습·검사 데이터 보강에 활용
```

하는 순서로 다루는 것이 중요합니다.

이번 PART6에서는 다음 흐름을 경험했습니다.

```
불량 데이터 부족
        ↓
실제 결함 + Ground Truth Mask
        ↓
Defect Patch 추출
        ↓
정상 이미지에 Cut-Paste
        ↓
Synthetic Defect 생성
        ↓
Metadata 기록
        ↓
품질 확인
        ↓
사용 가능 여부 판단
```

> Synthetic Data는 많이 만드는 것보다, 실제 데이터와 얼마나 비슷하고 사용할 만한 품질인지 확인하는 과정이 중요합니다.

VAE, GAN, Diffusion, 3D Graphics를 이용한 다양한 합성 방법과 **Synthetic Data를 실제 학습에 추가했을 때 성능이 어떻게 달라지는지 비교하는 과정은 교과11에서 더 본격적으로 다룹니다.**

---

# PART7 — Mini Challenge: 오늘 배운 흐름을 스스로 다시 적용합니다 (46~50)

지금까지는 강의 자료를 따라가며 다음 과정을 함께 실습했습니다.

```
적은 데이터 상황 확인
        ↓
One-shot
Reference 1장으로 검사
        ↓
Few-shot
Normal / Anomaly Support로
Prototype 생성
        ↓
같은 Common Query에서
두 방법 비교
        ↓
불량 데이터가 부족한 경우
Cut-Paste Synthetic Defect 생성
        ↓
합성 결과 품질 확인
```

이제 새로운 알고리즘을 배우지 않습니다.

이번 Mini Challenge에서는 오늘 사용했던 코드와 공통 함수를 다시 이용하면서 조건을 조금 바꾸어 봅니다.

목표는 완성 코드를 그대로 따라 쓰는 것이 아니라:

> 오늘 배운 흐름을 이해하고, 조건이 달라졌을 때 스스로 실험을 다시 구성할 수 있는지 확인하는 것입니다.

약 3시간 동안 다음 세 가지를 진행합니다.

```
Challenge 1
Support 수를 바꾸어 Few-shot 다시 실험
        ↓
Challenge 2
다른 Defect Type으로 Cut-Paste 다시 적용
        ↓
Challenge 3
데이터 상황을 보고
사용할 방법 직접 선택
        ↓
최종 한 줄 정리
```

---

# 46. Challenge 1 — Support 수가 달라지면 결과도 달라질까요?

앞의 Few-shot에서는:

```
Normal Support 5장
Anomaly Support 5장
```

을 사용했습니다.

이번에는 같은 Support 데이터 중 사용하는 장수만 변경합니다.

```
실험 A
Normal 1장 + Anomaly 1장

실험 B
Normal 3장 + Anomaly 3장

실험 C
Normal 5장 + Anomaly 5장
```

Common Query는 바꾸지 않습니다.

```
Common Query

Normal 10장
Anomaly 10장

항상 동일
```

즉 이번 실험에서 바꾸는 조건은:

```
Support 수
1 → 3 → 5
```

하나입니다.

### 먼저 결과를 예상해 봅니다

실행하기 전에 팀에서 간단히 생각합니다.

```
Support가 많아지면
항상 Accuracy가 높아질까?

FP는 줄어들까?

FN은 줄어들까?
```

정답을 맞히는 것이 목적은 아닙니다.

**실행하기 전에 예상하고, 실제 결과와 비교하는 것**이 중요합니다.

---

# 47. 기존 Few-shot 코드를 변형하여 직접 실험합니다

새로운 Few-shot 알고리즘을 만드는 것이 아닙니다.

앞에서 사용했던:

```
day08_feature_core.py
67_day08_test_fewshot.py
```

의 흐름을 다시 활용합니다.

이번에는 Support 수를 다음과 같이 바꾸어 반복합니다.

```
SUPPORT_COUNTS = [1, 3, 5]
```

전체 아이디어는 다음과 같습니다.

```
Support 1장씩 선택
        ↓
Normal Prototype
Anomaly Prototype
        ↓
같은 Common Query 20장 검사
        ↓
Accuracy / FP / FN 기록

        ↓

Support 3장씩 선택
        ↓
새 Prototype
        ↓
같은 Common Query 검사
        ↓
Accuracy / FP / FN 기록

        ↓

Support 5장씩 선택
        ↓
새 Prototype
        ↓
같은 Common Query 검사
        ↓
Accuracy / FP / FN 기록
```

앞에서 작성한 Few-shot 코드를 참고하여:

```
Support 수 변경
        ↓
Prototype 다시 생성
        ↓
같은 Query 검사
        ↓
성능 계산
```

부분을 스스로 연결합니다.

막히면 다음 의사코드를 참고합니다.

```
1. Support 수를 1, 3, 5로 준비한다.

2. Common Query Feature는
   같은 이미지를 계속 사용한다.

3. Support 수 하나를 선택한다.

4. Normal Support에서
   해당 장수만 Feature를 추출한다.

5. Normal Prototype을 만든다.

6. Anomaly Support에서도
   같은 장수만 Feature를 추출한다.

7. Anomaly Prototype을 만든다.

8. Common Query 20장을
   두 Prototype과 비교한다.

9. Normal / Anomaly를 판정한다.

10. Accuracy, FP, FN을 계산한다.

11. Support 1 / 3 / 5 결과를 저장한다.

12. 세 결과를 비교한다.
```

실행 결과에서는 모든 지표를 길게 분석하지 않습니다.

다음 세 가지만 봅니다.

|Support 수|Accuracy|FP|FN|
|---|---|---|---|
|1장||||
|3장||||
|5장||||

그리고 한 가지만 생각합니다.

> Support 수가 바뀌면서 실제 판정 결과도 달라졌는가?

Support가 많아졌다고 반드시 성능이 일정하게 좋아져야 하는 것은 아닙니다.

적은 데이터에서는 몇 장을 사용했는가뿐 아니라 어떤 이미지가 Support로 들어갔는가도 Prototype에 영향을 줄 수 있기 때문입니다.

---

# 48. Challenge 2 — 다른 Defect Type에도 같은 합성 흐름을 적용합니다

앞에서는:

```
broken_small
```

결함으로 Cut-Paste Synthetic Defect를 만들었습니다.

이번에는 아직 본 실습에서 사용하지 않은:

```
contamination
```

으로 같은 과정을 다시 적용합니다.

새로운 Cut-Paste 코드를 처음부터 다시 작성할 필요는 없습니다.

앞에서 만든:

```
day08_cutpaste_core.py
```

를 재사용합니다.

이번에는 다음 조건으로 Synthetic Defect 5장만 만들어 봅니다.

```
Defect Type
→ contamination

생성 수
→ 5장

Scale
→ 0.6 ~ 1.0

Rotation
→ -10 ~ 10 degree
```

지금부터 해야 할 일은 기존 실행 코드를 복사하는 것이 아니라, 어떤 입력과 파라미터를 바꾸어야 하는지 찾아 수정하는 것입니다.

```
broken_small 실행 코드 확인
        ↓
어떤 부분이 Defect Type인가?
        ↓
어떤 부분이 Source 경로인가?
        ↓
어떤 부분이 생성 수인가?
        ↓
어떤 부분이 Scale / Rotation인가?
        ↓
contamination 조건으로 변경
        ↓
실행
```

정상적으로 실행되었다면:

```
Synthetic contamination
5장
```

이 생성되어야 합니다.

---

## 생성 결과는 세 장 정도만 확인합니다

모든 이미지를 다시 정밀하게 기록할 필요는 없습니다.

생성된 5장 중 대표적인 3장 정도를 열어 봅니다.

다음만 확인합니다.

```
위치가 자연스러운가?

크기가 적절한가?

Patch 경계가 눈에 띄는가?

contamination처럼 보이는가?
```

그리고:

```
가장 괜찮은 이미지 1장

가장 어색한 이미지 1장
```

만 선택합니다.

> 목표는 품질 보고서를 길게 작성하는 것이 아니라, 같은 Cut-Paste 흐름을 다른 결함에도 적용하고 결과를 스스로 판단해 보는 것입니다.

---

# 49. Challenge 3 — 데이터 상황을 보고 방법을 선택합니다

마지막으로 오늘 배운 방법을 실제 상황과 연결합니다.

코드를 추가로 작성하지 않습니다.

다음 네 상황을 보고 팀에서 가장 먼저 고려할 방법을 정합니다.

|상황|먼저 고려할 방법|
|---|---|
|정상 데이터는 충분하지만 불량 예제가 없음||
|확실한 Normal Reference 1장과 Calibration용 정상 데이터는 있지만 불량 데이터가 거의 없음||
|Normal / Anomaly 예제가 각각 몇 장씩 있음||
|실제 불량 이미지가 너무 적어 데이터 보강이 필요함||

오늘 배운 후보는 다음과 같습니다.

```
Anomaly Detection
Zero-shot
One-shot
Few-shot
Defect Generation
추가 데이터 수집
```

각 상황마다 긴 설명을 쓸 필요는 없습니다.

예를 들어:

```
상황
Normal / Anomaly가 각각 몇 장 있다.

선택
Few-shot

이유
소수의 정상·불량 예제로
Prototype을 만들 수 있기 때문
```

정도의 한두 문장이면 충분합니다.

정답을 하나로 고정하지 않습니다.

> 현재 가지고 있는 데이터 조건과 선택한 방법이 서로 맞는지 설명할 수 있으면 됩니다.

---

# 50. Mini Challenge를 한 장으로 마무리합니다

별도의 보고서를 여러 개 만들지 않습니다.

다음 파일 하나에 Mini Challenge 결과만 간단히 정리합니다.

```
reports/day08_mini_challenge.md
```

다음 정도면 충분합니다.

```
# Day08 Mini Challenge

## 1. Support Count 비교

| Support | Accuracy | FP | FN |
|---:|---:|---:|---:|
| 1 |  |  |  |
| 3 |  |  |  |
| 5 |  |  |  |

가장 달라진 점:

---

## 2. contamination Synthetic Defect

생성 수: 5장

가장 자연스러웠던 이미지:

가장 어색했던 이미지:

한 줄 이유:

---

## 3. 상황별 방법 선택

정상 데이터 충분 / 불량 0장:
→

Normal Reference 1장 중심:
→

Normal / Anomaly 소수 예제:
→

불량 데이터 자체가 부족함:
→

---

## 오늘의 한 줄 결론

데이터가 적을 때 검사 방법을 선택할 때 가장 중요하다고 생각한 점:
```

길게 작성할 필요는 없습니다.

이 파일의 목적은 평가 보고서를 만드는 것이 아니라:

```
오늘 배운 내용
        ↓
조건을 직접 변경
        ↓
다시 실행
        ↓
결과 확인
        ↓
왜 달라졌는지 생각
        ↓
상황에 맞는 방법 선택
```

까지 스스로 해봤다는 것을 남기는 것입니다.

> Mini Challenge의 목표는 새로운 코드를 많이 작성하는 것이 아니라, 
> 오늘 배운 One-shot / Few-shot / Defect Generation의 흐름을 스스로 연결하고 조건을 바꾸어 다시 적용해 보는 것입니다.

---

# PART9 — 오늘 배운 내용 정리 + README + Git + 자가체크 (56~63)

오늘 실습과 Mini Challenge까지 모두 완료했습니다.

마지막에는 새로운 코드를 작성하지 않습니다.

지금까지 직접 실행한 결과에서 오늘 무엇을 배웠는지 확인하고, 필요한 내용만 간단히 기록한 뒤 Git에 저장하고 마무리합니다.

```
오늘 결과 확인
        ↓
README에 핵심만 기록
        ↓
Git Commit / Push
        ↓
자가 체크
        ↓
8일차 종료
```

---

# 56. 오늘의 핵심 결과만 다시 확인합니다

오늘 가장 중요한 실험 결과는 다음과 같습니다.

```
One-shot
Reference 1장
        ↓
Accuracy 85%
FP 1
FN 2


Few-shot
Normal Support 5장
Anomaly Support 5장
        ↓
Accuracy 90%
FP 0
FN 2
```

이번 결과에서는 Few-shot이 One-shot보다 Accuracy가 높아졌고 정상 이미지의 오탐 `FP`는 줄었습니다.

하지만:

```
One-shot FN = 2
Few-shot FN = 2
```

로 실제 불량을 정상으로 놓치는 문제는 남아 있었습니다.

따라서 오늘의 결론은:

> Few-shot이 무조건 완벽하다는 것이 아니라, 사용할 수 있는 데이터와 기준을 만드는 방법에 따라 결과가 달라질 수 있다는 것입니다.

---

# 57. Synthetic Defect에서 확인한 것도 한 가지만 기억합니다

오늘 `broken_small` 실제 결함과 Ground Truth Mask를 이용하여 Synthetic Defect를 만들었습니다.

```
실제 결함
+
Ground Truth Mask
        ↓
Defect Patch
        ↓
정상 이미지에 Cut-Paste
        ↓
Synthetic Defect
```

실제로 생성된 이미지를 확인해 보니:

```
자연스러운 이미지도 있음
        +
크기·경계가
부자연스러운 이미지도 있음
```

을 확인했습니다.

따라서 다음 원칙만 기억합니다.

> Synthetic Data는 생성되었다고 바로 사용하는 것이 아니라, 품질을 확인한 뒤 사용할 데이터를 선택해야 합니다.

Mini Challenge에서는 같은 방법을 다른 결함인 `contamination`에도 다시 적용해 보았습니다.

---

# 58. 오늘 배운 내용을 README에 간단히 남깁니다

README에 하루 수업 내용을 다시 복사할 필요는 없습니다.

다음 정도만 추가합니다.

```
## Day 08 — Low-Data Inspection

### One-shot

- Normal Reference: 1장
- Common Query: Normal 10장 + Anomaly 10장
- Accuracy: 85%
- FP: 1
- FN: 2

### Few-shot

- Normal Support: 5장
- Anomaly Support: 5장
- Common Query: One-shot과 동일
- Accuracy: 90%
- FP: 0
- FN: 2

### Comparison

이번 실험에서는 Few-shot에서 정상 오탐은 줄었지만
불량 미검출 FN은 여전히 남았다.

### Defect Generation

- Defect Type: broken_small
- Ground Truth Mask 기반 Cut-Paste
- Synthetic Defect 20장 생성
- 생성 후 품질 확인 필요

### Mini Challenge

- Support 수를 1 / 3 / 5장으로 변경하여 결과 비교
- contamination 결함으로 Cut-Paste 다시 적용

### 오늘의 핵심

데이터가 적을 때는 데이터의 수와 상태를 먼저 확인하고
One-shot, Few-shot, Anomaly Detection, Defect Generation 등의
후보 방법을 선택해야 한다.
```

Support Count Challenge의 실제 결과가 나왔다면 `Mini Challenge` 부분에 **가장 중요한 결과 한 줄만 추가**하면 됩니다.

---

# 59. Git에 저장하기 전에 변경 내용을 확인합니다

지금까지 작성한 코드와 README를 확인합니다.

```bash
git status
```

Git에 기록할 파일을 추가합니다.

```bash
git add src configs reports README.md requirements.txt .gitignore
```

별도로 오늘 만든 모든 파일명을 다시 확인할 필요는 없습니다.

`git status`에서 필요한 변경 내용이 정상적으로 보이면 됩니다.

---

# 60. 8일차 작업을 Commit합니다

Commit합니다.

```bash
git commit -m "feat: complete day08 low-data inspection"
```

Commit 결과를 확인합니다.

```bash
git log --oneline -5
```

외부 Git 저장소 사용이 허용된 환경이라면 현재 브랜치를 Push합니다.

```bash
git push -u origin "$(git branch --show-current)"
```

외부 GitHub 사용이 제한된 교육환경에서는 **Local Git 또는 기관 내부 Git 저장소에 Commit하는 것까지만 진행**합니다.

> 중요한 것은 GitHub 자체가 아니라, 오늘 작성한 코드와 변경 내용을 하나의 작업 단위로 기록하는 것입니다.

---

# 61. 8일차 자가 체크리스트

수업을 마치기 전에 다음 항목을 직접 체크합니다.

### 데이터와 개념

- [ ]  Reference / Calibration / Support / Query의 차이를 설명할 수 있다.
- [ ]  One-shot과 Few-shot의 차이를 설명할 수 있다.
- [ ]  Support와 Query를 같은 이미지로 사용하면 안 되는 이유를 이해한다.
- [ ]  Similarity가 높다는 것이 무엇을 의미하는지 설명할 수 있다.

### 실제 실습

- [ ]  One-shot에서 Reference와 Threshold를 이용해 Query를 판정했다.
- [ ]  Few-shot에서 Normal / Anomaly Prototype을 만들었다.
- [ ]  One-shot과 Few-shot을 같은 Common Query에서 비교했다.
- [ ]  Accuracy뿐 아니라 FP와 FN도 함께 확인했다.
- [ ]  Support 수를 바꾸어 Few-shot 결과가 달라질 수 있음을 확인했다.
- [ ]  Ground Truth Mask를 이용해 Synthetic Defect를 만들었다.
- [ ]  Synthetic 이미지는 생성 후 품질 확인이 필요하다는 것을 이해한다.

### 스스로 판단하기

- [ ]  데이터가 적다고 무조건 Few-shot을 사용하는 것은 아니라는 것을 설명할 수 있다.
- [ ]  불량 데이터가 거의 없을 때 고려할 수 있는 방법을 한 가지 이상 말할 수 있다.
- [ ]  검사 결과가 100%가 아니어도 바로 Threshold를 바꾸면 안 되는 이유를 이해한다.

모든 항목을 완벽하게 외울 필요는 없습니다.

체크가 어려운 항목이 있다면 해당 실습 부분만 다시 확인합니다.

---

# 62. 오늘 배운 내용을 한 번에 정리합니다

8일차의 전체 흐름은 이것만 기억하면 됩니다.

```
데이터가 매우 적다
        ↓
현재 가지고 있는 데이터 확인
        ↓

기준 이미지 1장
→ One-shot

정상·불량 예제 몇 장
→ Few-shot

        ↓
같은 Query에서 결과 비교
        ↓
Accuracy + FP + FN 확인
        ↓

불량 데이터가 부족함
        ↓
Defect Generation도 고려
        ↓
Synthetic 생성
        ↓
품질 확인
```

오늘 가장 중요한 것은 알고리즘 이름을 외우는 것이 아닙니다.

```
현재 데이터가
얼마나 있는가?
        ↓
어떤 방법을 사용할 수 있는가?
        ↓
같은 조건에서 성능을 확인
        ↓
오판이 나오면
원인을 생각하고 다음 실험으로 연결
```

하는 흐름을 이해하는 것입니다.

> **데이터의 양과 상태가 달라지면 검사 전략도 달라질 수 있습니다.**

---

# 63. 8일차를 마무리합니다

오늘은 다음 과정을 직접 경험했습니다.

```
One-shot
        ↓
Few-shot
        ↓
두 방법 비교
        ↓
Support 수 변경
        ↓
Defect Generation
        ↓
다른 결함으로 Mini Challenge
```

여기까지 직접 실행하고 결과를 설명할 수 있다면 8일차의 핵심 학습은 완료한 것입니다.

```
README 정리
        ↓
Git Commit
        ↓
필요한 경우 Push
        ↓
자가 체크 완료
        ↓
8일차 종료
```