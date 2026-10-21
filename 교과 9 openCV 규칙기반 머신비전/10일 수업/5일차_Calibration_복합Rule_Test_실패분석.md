## 오늘의 목표

1일차에서는 **검사할 위치를 정하고 ROI를 확보하는 방법**을 배웠습니다.

2일차에서는 이미지의 `Brightness`, `Contrast`, `Noise`, `Blur`를 확인하고 영상.이미지 품질을 개선하는 방법을 배웠습니다.

3일차에서는 병뚜껑을 대상으로 `Area Ratio`, `Count`, `BBox`, `Center` 같은 Feature를 숫자로 측정하고 간단한 Candidate Rule을 만들어 보았습니다.

4일차에서는 광천김 로고를 대상으로 `Template Matching`, `ORB`, `SIFT`, `RANSAC`, `Homography`를 사용하여 로고를 비교하고 측정하는 여러 방법을 실습했습니다.

5일차에서는 여기서 한 단계 더 나아갑니다.

> 여러 정상·불량 이미지의 측정값을 비교하여 실제 OK / NG 기준을 만들고, 그 기준이 새로운 이미지에서도 잘 동작하는지 확인합니다.


오늘의 핵심 질문은 다음과 같습니다.

> 여러 이미지에서 측정한 값을 이용해 어디까지 OK이고 어디부터 NG인지 어떻게 정하고, 
> 그 기준이 새로운 이미지에서도 잘 동작하는지 어떻게 확인할까요?

오늘 수업이 끝나면 다음을 할 수 있어야 합니다.

- Calibration 데이터로 `OK / NG` 기준을 만들 수 있다.
- 여러 Feature를 비교하여 필요한 Rule을 선택할 수 있다.
- 결정한 Rule을 고정하여 새로운 Test 이미지에 적용할 수 있다.
- `TP / TN / FP / FN`으로 검사 결과를 평가할 수 있다.
- 실패한 이미지의 원인을 확인할 수 있다.
- 다른 제품에서도 `Calibration → Rule → Test` 과정을 반복할 수 있다.

> 5일차의 핵심은 Threshold 숫자를 외우는 것이 아니라, 
> 데이터를 근거로 검사 기준을 만들고 검증하는 과정을 익히는 것입니다.

---

# 1. 5일차의 전체 흐름

5일차는 새로운 Matching 알고리즘을 계속 추가하는 날이 아닙니다.

4일차에서 배운 `Template Matching`, `SIFT`, `RANSAC`, `Homography`를 여러 이미지에 반복 적용하여 실제 검사 Rule을 만들고 검증하는 날입니다.

### 5일차 전체 흐름

![[Pasted image 20260831182816.png]]

그림의 흐름을 간단히 읽으면 다음과 같습니다.

```text
PART1  실습 환경 + 데이터 준비
7~9
        ↓
PART2  측정 이해 + 공통 함수
10~15
        ↓
PART3  Calibration
16~22
측정 → 분포 확인 → Threshold 후보 찾기
        ↓
PART4  Baseline Rule 결정
23~25
Rule 선택 → JSON 저장 → Freeze
        ↓
PART5  Test
26~28
처음 보는 6장에 동결 Rule 적용
        ↓
PART6  Evaluation
29~31
TP / TN / FP / FN + 성능 평가
        ↓
PART7  Mini Challenge
32~38
다른 제품에서
Calibration → Rule → Test 직접 반복
        ↓
PART8  정리
39~43
README + 자가체크 + Git 기록
```

## 2.1 PART1 — 실습 환경과 데이터를 준비합니다

먼저 오늘 사용할 폴더와 데이터를 준비합니다.

```
- Calibration 데이터
- Test 데이터
- Reference 이미지
- 결과 저장 폴더
```

즉, 실습을 시작할 수 있는 기본 환경을 만드는 단계입니다.

---

## 2.2 PART2 — 공통 측정 함수를 준비합니다

4일차에서 배운:

```
- Template Matching
- SIFT
- Homography
```

를 여러 이미지에 반복 적용할 수 있도록 공통 함수로 정리합니다.

---

## 2.3 PART3 — Calibration으로 기준 후보를 찾습니다

Calibration 12장을 측정하고:

```
- OK / NG 값 비교
- X-Y 분포 확인
- Threshold 후보 비교
```

를 통해 판정 기준 후보를 찾습니다.

---

## 2.4 PART4 — 사용할 Rule을 결정합니다

Calibration에서 찾은 기준 중 하나를 선택하고:

```
- 복합 Rule 결정
- JSON 저장
- Baseline Freeze
```

까지 진행합니다.

---

## 2.5 PART5 — 처음 보는 Test 이미지에 적용합니다

동결한 Rule을 Test 6장에 그대로 적용하여:

```
- OK / NG 판정
- 실제 정답과 비교
```

를 진행합니다.

---

## 2.6 PART6 — Test 결과를 평가합니다

Test 결과에서:

```
- TP / TN / FP / FN
- Accuracy
- Precision
- Recall
```

을 확인하고 잘못 판정된 이미지도 살펴봅니다.

| 용어            | 영어             | 이번 수업에서의 의미                 |
| ------------- | -------------- | --------------------------- |
| **TP**        | True Positive  | 실제 NG → NG로 맞게 검출           |
| **TN**        | True Negative  | 실제 OK → OK로 맞게 판정           |
| **FP**        | False Positive | 실제 OK → NG로 잘못 판정           |
| **FN**        | False Negative | 실제 NG → OK로 잘못 통과           |
| **Accuracy**  | Accuracy       | 전체 중 얼마나 많이 맞혔는가            |
| **Precision** | Precision      | NG라고 판정한 것 중 실제 NG가 얼마나 되는가 |
| **Recall**    | Recall         | 실제 NG 중 얼마나 놓치지 않고 찾아냈는가    |

---

## 2.7 PART7 — 다른 제품으로 직접 반복합니다

광천김이 아닌 다른 제품을 사용하여:

```
Calibration → Rule → Test
```

과정을 직접 다시 해봅니다.

---

## 2.8 PART8 — 결과를 정리합니다

마지막으로:

```
- README 정리
- 자가체크
- Git Commit
```

까지 진행하고 5일차를 마무리합니다.

---

# 2. 4일차 데이터에서 5일차를 시작합니다

4일차에서는 **광천김 포장지의 로고 영역**을 검사 대상으로 사용했습니다.

기준이 되는 로고 이미지를 만들고, 같은 제품에 위치·회전·크기 변화를 주어 다음 방법들의 측정 결과를 비교했습니다.

```text
광천김 포장지
        ↓
기준 로고 준비
        ↓
normal / shift / rot15 / scale90
        ↓
Template Matching
ORB
SIFT
Homography
        ↓
Score / Match / Inlier 등
측정값 비교
```

4일차의 목적은 이러한 값을 **측정하고 비교하는 것**이었기 때문에 아직 최종 OK / NG 기준은 정하지 않았습니다.

5일차에서는 여기서 이어서 **같은 광천김 로고 검사 문제를 사용합니다.**

기준 데이터는 4일차에서 만든 다음 파일을 사용합니다.

```
samples/day04/reference/reference_label.jpg
samples/day04/targets/target_normal.jpg
```

그리고 새로운 Calibration 이미지와 Test 이미지를 추가하여 여러 이미지의 측정값을 모읍니다.

```
4일차
광천김 로고의 Matching 방법 학습
        ↓
측정값 확인
        ↓
아직 Rule 확정 X

        ↓

5일차
같은 광천김 로고 검사 계속 사용
        ↓
Calibration 이미지 여러 장 측정
        ↓
정상 / 불량 측정값 비교
        ↓
Threshold 결정
        ↓
복합 Rule
        ↓
Test
        ↓
실패 분석
```

4일차가 ==`“어떻게 측정할 것인가?”`==를 배우는 날이었다면, 
5일차는 ==`“측정한 값을 어떤 기준으로 OK / NG 판정에 사용할 것인가?”`==를 배우는 날입니다.

---

# 3. 오늘 무엇을 검사할까요?

5일차에서도 4일차와 같은 **광천김 로고 검사**를 계속 사용합니다.

## 검사 대상

```text
광천김 포장지의
KC + (주)광천김 + 영문 로고 영역
```

기준이 되는 로고 이미지는 다음 파일입니다.

`reference_label.jpg` : 정답처럼 비교할 기준 로고
![[Pasted image 20260831114455.png]]

이 이미지는 “정상 로고는 이렇게 생겼다”라고 알려주는 기준 이미지입니다.

그리고 실제 검사 대상 이미지는 다음과 같습니다.

`target_normal.jpg` : 그 기준 로고를 찾아볼 검사 이미지
![[Pasted image 20260831114641.png]]

```
reference_label.jpg
→ 찾고 싶은 정상 로고

target_normal.jpg
→ 그 로고가 들어 있는 검사 이미지
```

5일차에서는 이 관계를 기준으로 여러 정상·불량 이미지를 추가하여 검사합니다.

```text
기준 로고
        +
여러 검사 이미지
        ↓
Matching 값 측정
        ↓
정상과 불량 비교
        ↓
OK / NG 기준 결정
```

오늘의 기본 검사 질문은 하나입니다.

```
검사 이미지 안에 기준과 같은 광천김 로고가 정상적으로 존재하는가?
```

### OK

```
로고가 정상적으로 보이고
기준 로고와 충분히 Matching됨
        ↓
OK
```

제품이 조금 이동하거나 회전하더라도 같은 로고가 정상적으로 확인된다면 OK로 볼 수 있습니다.

### NG

```
로고 일부가 가려짐
또는
기준 로고와 충분히 Matching되지 않음
        ↓
NG
```

즉 5일차에서는:

> 정상 로고는 OK로 통과시키고, 로고가 정상적으로 확인되지 않는 이미지는 NG로 검출하는 검사 기준을 만들어 봅니다.

이후에는 여러 Calibration 이미지의 실제 측정값을 비교하여 어느 정도부터 OK / NG로 판단할 것인지 Threshold와 Rule을 결정합니다.

---

# 4. 오늘은 여러 이미지에서 검사값을 비교합니다

4일차에서는 광천김 로고를 찾기 위해 여러 방법을 사용해 보았습니다.

```text
Template Matching
SIFT
Homography
```

그리고 각 방법에서 다음과 같은 **숫자 결과**가 나온다는 것도 확인했습니다.

```text
Template Matching
→ 로고가 얼마나 비슷한가?

SIFT
→ 서로 비슷한 특징점이 얼마나 많이 연결되는가?

Homography
→ 연결된 특징점 중 실제로 올바른 대응이 얼마나 남는가?
```

하지만 이미지 한 장의 결과만 보고 바로:

```
이 값 이상이면 OK
이 값보다 작으면 NG
```

라고 정할 수는 없습니다.

왜냐하면 같은 정상 제품이라도 촬영할 때마다 값이 조금씩 달라질 수 있기 때문입니다.

그래서 오늘은 여러 정상·불량 이미지를 검사하여 실제 값이 어느 정도 나오는지 먼저 확인합니다.

```
정상 이미지 여러 장
        +
불량 이미지 여러 장
        ↓
검사값 측정
        ↓
정상과 불량의 차이 비교
        ↓
OK / NG 기준 결정
```

5일차의 핵심은 검사 기준값을 미리 정하는 것이 아니라 실제 데이터를 측정한 뒤 결정하는 것입니다.

---

# 5. 오늘 사용할 이미지는 역할을 나누어 준비합니다

5일차 본 실습에서는 이미지를 목적에 따라 `Calibration`과 `Test`로 나누어 사용합니다.

| 데이터           | 쉽게 말하면               | 오늘 하는 일                            |
| ------------- | -------------------- | ---------------------------------- |
| `Calibration` | **기준을 만드는 연습용 데이터**  | 정상·불량 값을 비교하여 OK / NG 기준을 결정       |
| `Test`        | **정해진 기준을 시험하는 데이터** | 처음 보는 이미지에서도 판정이 잘 되는지 확인          |

사용 순서는 다음과 같습니다.

```text
Calibration 이미지
        ↓
여러 장을 측정
        ↓
OK / NG 기준 결정
        ↓
Rule 저장

        ↓

Test 이미지
        ↓
정해진 Rule 그대로 검사
        ↓
맞게 판정되는지 확인

```

여기서 중요한 규칙은 하나입니다.

> **Test 결과가 마음에 들지 않는다고 기준을 계속 바꾸지 않습니다.**

기준을 정하는 것은 `Calibration`, 정한 기준을 평가하는 것은 `Test`의 역할이기 때문입니다.

그리고 4일차에서 만든 `reference_label.jpg`는 이 세 그룹과는 다릅니다.

```
reference_label.jpg
→ "이 로고를 찾아라"라고 알려주는 기준 로고

Calibration
→ 얼마부터 OK로 볼지 결정

Test
→ 그 기준이 잘 작동하는지 확인
```

---

```
32_day04_challenge.py
4일차 — 이미지 매칭 방법 실습 완료
            ↓
day05_logo_features.py
Template·SIFT·Homography 공통 측정 함수
            ↓
33_day05_check_features.py
대표 이미지 1장으로 정상 동작 확인
            ↓
34_day05_measure_calibration.py
Calibration 12장 측정 → CSV 저장
            ↓
35_day05_analyze_calibration.py
OK / NG 분포 분석 → Threshold 후보 탐색
            ↓
36_day05_freeze_rule.py
복합 Rule 확정 → JSON 저장
            ↓
37_day05_test.py
새로운 Test 6장 → OK / NG 판정
            ↓
38_day05_evaluate.py
성능 평가 → FP / FN 실패 분석
            ↓
39_day05_mini_challenge.py
다른 제품으로 전체 과정 자율 실습
```

| 파일명                               | 역할                     | 설명                                                                                                             |
| --------------------------------- | ---------------------- | -------------------------------------------------------------------------------------------------------------- |
| `32_day04_challenge.py`           | 4일차 Mini Challenge     | 새로운 실제 촬영 이미지에서 Template·ORB·SIFT·Homography 중 적합한 검사 방법을 직접 선택하고 결과를 비교합니다.                                   |
| `day05_logo_features.py`          | 로고 특징값 측정 공통 함수        | 4일차에서 배운 Template Matching·SIFT·Homography를 함수로 정리하여 여러 이미지의 Score·Good Match·Inlier Ratio 등을 반복 측정할 수 있게 합니다. |
| `33_day05_check_features.py`      | 공통 함수 정상 작동 확인         | 광천김 이미지 1장을 먼저 검사하여 Template Score·SIFT Good Match·Homography·Inlier Ratio가 정상적으로 계산되는지 확인합니다.                 |
| `34_day05_measure_calibration.py` | Calibration 데이터 측정     | 정상(OK) 6장과 불량(NG) 6장, 총 12장의 특징값을 측정하고 결과를 CSV로 저장합니다.                                                         |
| `35_day05_analyze_calibration.py` | 판정 기준(Threshold) 후보 분석 | Calibration 측정값의 정상·불량 분포를 그래프로 비교하고, SIFT·Inlier Ratio 기준값 조합을 시험하여 오류가 적은 Rule 후보를 찾습니다.                     |
| `36_day05_freeze_rule.py`         | 복합 Rule 확정·동결          | Calibration에서 선택한 판정 기준을 JSON 파일로 저장하고, 이후 Test에서 동일한 기준을 사용하도록 고정합니다.                                         |
| `37_day05_test.py`                | 새로운 이미지 OK / NG 검사     | 확정된 Rule을 처음 보는 Test 이미지 6장에 적용하여 OK / NG를 판정하고, 실제 정답 및 판정 이유를 CSV와 결과 이미지로 저장합니다.                            |
| `38_day05_evaluate.py`            | 성능 평가·실패 분석            | Test 결과에서 TP·TN·FP·FN과 Accuracy·Precision·Recall을 계산하고, 잘못 판정한 이미지를 별도 폴더에 모읍니다.                               |
| `39_day05_mini_challenge.py`      | 나만의 검사 Rule 만들기        | 다른 제품의 로고를 사용하여 Calibration → Threshold 결정 → Rule 확정 → Test → 실패 분석 과정을 학생 스스로 수행합니다.                          |
![[Pasted image 20261020085525.png]]

![[Pasted image 20261020085435.png]]

---

# 6. 5일차 데이터는 새 폴더에 분리합니다

4일차 파일을 덮어쓰지 않습니다.

다음 구조를 사용합니다.

```text
subject9_machinevision/
│
├── samples/
│   ├── day02/
│   ├── day03/
│   ├── day04/
│   │
│   └── day05/
│       ├── reference/
│       │   ├── reference_label.jpg
│       │   └── expected_normal.jpg
│       │
│       ├── resized/
│       │   ├── ok_01.jpg ~ ok_06.jpg
│       │   ├── ng_01.jpg ~ ng_06.jpg
│       │   ├── test_ok_01.jpg ~ test_ok_03.jpg
│       │   └── test_ng_01.jpg ~ test_ng_03.jpg
│       │
│       └── mini_challenge/
│           ├── reference/
│           ├── calibration/
│           │   ├── ok/
│           │   └── ng/
│           └── test/
│               ├── ok/
│               └── ng/
│
├── outputs/
│   └── day05/
│       ├── calibration/
│       ├── test/
│       ├── fp/
│       └── fn/
│
├── configs/
│   └── day05_logo_rules.json
│
├── src/
│   ├── ...
│   ├── 32_day04_challenge.py
│   ├── day05_logo_features.py
│   ├── 33_day05_check_features.py
│   ├── 34_day05_measure_calibration.py
│   ├── 35_day05_analyze_calibration.py
│   ├── 36_day05_freeze_rule.py
│   ├── 37_day05_test.py
│   ├── 38_day05_evaluate.py
│   └── 39_day05_mini_challenge.py
│
└── reports/
    ├── day05_calibration_measurements.csv
    ├── day05_rule_candidates.csv
    ├── day05_test_results.csv
    ├── day05_test_metrics.md
    └── day05_mini_challenge.md
```

4일차 마지막 코드가 `32_day04_challenge.py`였으므로 5일차 실행 파일은 `33`부터 이어집니다.

공통 Feature 함수는 여러 실행 파일에서 재사용할 것이므로 숫자를 붙이지 않고:

```text
src/day05_logo_features.py
```

로 둡니다.

---
# PART1 실습 환경 + 데이터 준비 (7~9)

---

# 7. Day05 폴더를 만듭니다

프로젝트 폴더로 이동합니다.

```bash
cd ~/ai_vision/subject9_machinevision
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

폴더를 만듭니다.

```bash
mkdir -p samples/day05/reference
mkdir -p samples/day05/resized

mkdir -p samples/day05/mini_challenge/reference
mkdir -p samples/day05/mini_challenge/calibration/ok
mkdir -p samples/day05/mini_challenge/calibration/ng
mkdir -p samples/day05/mini_challenge/test/ok
mkdir -p samples/day05/mini_challenge/test/ng

mkdir -p outputs/day05/calibration
mkdir -p outputs/day05/test
mkdir -p outputs/day05/fp
mkdir -p outputs/day05/fn

mkdir -p configs
mkdir -p reports
```

---

# 8. 4일차 Reference를 Day05에 복사하여 고정합니다

4일차 파일을 직접 수정하지 않도록 복사합니다.

```bash
cp samples/day04/reference/reference_label.jpg \
   samples/day05/reference/reference_label.jpg

cp samples/day04/targets/target_normal.jpg \
   samples/day05/reference/expected_normal.jpg
```

확인합니다.

```bash
ls -lh samples/day05/reference
```

다음 두 파일이 있으면 됩니다.

```text
reference_label.jpg
expected_normal.jpg
```

---
# 9. 5일차 Calibration과 Test 데이터를 확인합니다

5일차 실습을 위해 광천김 이미지를 총 **18장** 준비했습니다.

모든 이미지는 동일하게 다음 해상도를 사용합니다.

```
1920 × 1080
```

데이터는 역할에 따라 다음과 같이 나누어 사용합니다.

```
Calibration
├── OK 6장
└── NG 6장

Test
├── OK 3장
└── NG 3장

총 18장
```

현재 준비된 파일은 다음과 같습니다.

```
ok_01.jpg
ok_02.jpg
ok_03.jpg
ok_04.jpg
ok_05.jpg
ok_06.jpg

ng_01.jpg
ng_02.jpg
ng_03.jpg
ng_04.jpg
ng_05.jpg
ng_06.jpg

test_ok_01.jpg
test_ok_02.jpg
test_ok_03.jpg

test_ng_01.jpg
test_ng_02.jpg
test_ng_03.jpg
```

---

## 9.1 Calibration 데이터는 기준을 만드는 데 사용합니다

먼저 다음 12장을 `Calibration` 데이터로 사용합니다.

```
OK
ok_01.jpg
ok_02.jpg
ok_03.jpg
ok_04.jpg
ok_05.jpg
ok_06.jpg

NG
ng_01.jpg
ng_02.jpg
ng_03.jpg
ng_04.jpg
ng_05.jpg
ng_06.jpg
```

`OK` 이미지는 광천김 로고가 정상적으로 보이는 사진입니다.

```
제품 위치가 조금 다름
촬영 각도가 조금 다름
촬영 거리가 조금 다름

하지만

KC + (주)광천김 + 영문 로고
→ 정상적으로 확인됨
```

`NG` 이미지는 같은 검사 대상에서 로고가 정상적으로 확인되지 않도록 준비한 사진입니다.

```
로고 일부 가림
로고의 다른 부분 가림
가려진 범위 증가
        ↓
Reference와 정상적인 Matching이
어려워지도록 만든 이미지
```

이 12장의 측정값을 비교하여 **어느 정도의 값을 OK / NG 기준으로 사용할 것인지 결정**합니다.

---

## 9.2 Test 데이터는 정해진 기준을 확인하는 데 사용합니다

다음 6장은 Calibration에 사용하지 않고 따로 보관합니다.

```
OK
test_ok_01.jpg
test_ok_02.jpg
test_ok_03.jpg

NG
test_ng_01.jpg
test_ng_02.jpg
test_ng_03.jpg
```

사용 순서는 다음과 같습니다.

```
Calibration 12장
        ↓
측정값 비교
        ↓
OK / NG Rule 결정
        ↓
Rule 동결
        ↓
Test 6장
        ↓
처음 보는 이미지에서도
판정이 잘 되는지 확인
```

> Test 이미지를 보고 기준값을 다시 정하는 것이 아니라, Calibration에서 정한 기준이 새로운 이미지에서도 동작하는지를 확인합니다.

---
# PART2 무엇을 측정할지 이해 + 공통 함수 (10~15)

---

# 10. 데이터를 준비했으니 이제 무엇을 할까요?

앞에서 5일차 실습용 광천김 이미지 18장을 준비했습니다.

```
samples/day05/resized/

Calibration
→ OK 6장
→ NG 6장

Test
→ OK 3장
→ NG 3장
```

가장 먼저 사용할 것은 **Calibration 12장**입니다.

먼저 각각의 사진에서 **광천김 로고가 기준 로고와 얼마나 잘 맞는지를 숫자로 측정**합니다.

```
기준 로고
reference_label.jpg
        +
Calibration 이미지 한 장
        ↓
로고 비교
        ↓
측정값 생성
```

이 작업을 OK 6장과 NG 6장에 똑같이 반복합니다.

```
ok_01.jpg  → 측정값
ok_02.jpg  → 측정값
...
ok_06.jpg  → 측정값

ng_01.jpg  → 측정값
ng_02.jpg  → 측정값
...
ng_06.jpg  → 측정값
```

그런 다음 정상과 불량의 결과가 실제로 얼마나 다른지 비교합니다.

```
OK 6장의 측정값
        VS
NG 6장의 측정값
        ↓
어떤 값이
정상과 불량을 잘 구분하는지 확인
```

> 5일차는 기준값을 먼저 정하는 것이 아니라, 실제 데이터를 먼저 측정한 뒤 OK와 NG의 차이를 보고 기준을 만드는 과정입니다.

---

# 11. 어떤 값을 측정할까요?

4일차에서 광천김 로고를 비교하면서 다음 기술을 이미 실습했습니다.

```
Template Matching
SIFT
RANSAC
Homography
```

5일차에서는 새로운 기술을 배우는 것이 아니라, 이 방법들을 여러 이미지에 반복해서 적용합니다.

이번 Calibration에서 우선 다음 값을 확인합니다.

### Template Score

```
기준 로고와
검사 이미지의 로고 모양이
얼마나 비슷한가?
```

### SIFT Good Match

```
기준 로고와 검사 이미지 사이에서
서로 비슷하게 연결된 특징점이
몇 개인가?
```

### Homography Inlier

```
연결된 특징점 중에서
실제로 같은 로고의 위치 관계라고
판단할 수 있는 점이 몇 개인가?
```

### Inlier Ratio

```
Good Match 중
올바른 대응점이 차지하는 비율
```

따라서 Calibration 이미지 한 장을 검사하면 대략 다음과 같은 결과가 만들어집니다.

```
ok_01.jpg

Template Score
SIFT Good Match
Homography Inlier
Inlier Ratio
```

이 작업을 12장에 반복합니다.

```
Reference
   │
   ├─ ok_01.jpg → 측정
   ├─ ok_02.jpg → 측정
   ├─ ...
   ├─ ok_06.jpg → 측정
   │
   ├─ ng_01.jpg → 측정
   ├─ ng_02.jpg → 측정
   └─ ng_06.jpg → 측정

          ↓

12장의 측정 결과
          ↓
CSV로 저장
          ↓
OK / NG 비교
```

여기서 한 가지 문제가 있습니다.

4일차처럼 이미지마다 같은 SIFT와 Homography 코드를 계속 복사해서 작성하면 코드가 매우 길어집니다.

그래서 다음 단계에서는:

```
이미지 한 장 입력
        ↓
Template / SIFT / Homography 측정
        ↓
측정값 반환
```

을 담당하는 공통 측정 함수를 먼저 만들어 둡니다.

그러면 이후에는:

```
ok_01.jpg → 공통 함수
ok_02.jpg → 공통 함수
...
ng_06.jpg → 공통 함수
```

처럼 같은 코드를 반복해서 사용할 수 있습니다.

> 즉, 다음에 만드는 공통 함수는 새로운 기술을 배우기 위한 코드가 아니라 4일차에서 배운 측정 방법을 여러 이미지에 반복 적용하기 쉽게 정리하는 코드입니다.

---

# 12. 4일차 코드를 그대로 복사하지 않고 공통 측정 함수로 정리합니다

4일차에는 기술마다 별도 파일로 실습했습니다.

```text
27_template_matching.py
28_orb_matching.py
29_sift_matching.py
30_homography.py
```

그 구조는 각 기술을 배우기에는 좋았습니다.

하지만 5일차에는 수십 장의 이미지에서 같은 측정을 반복해야 합니다.

따라서 중복 코드를 줄이기 위해 공통 함수를 한 파일로 정리합니다.

```text
src/day05_logo_features.py
```

이 파일의 역할은 다음과 같습니다.

```text
Reference 준비
        ↓
Target 한 장 입력
        ↓
Template Score
SIFT Good Match
Homography Inlier
Inlier Ratio
Center
Scale
        ↓
측정값 반환
```

오늘의 새 알고리즘을 만드는 것이 아니라 **4일차에서 배운 코드를 재사용 가능한 형태로 정리하는 작업**입니다.

---

# 13. 여러 이미지를 반복 측정할 수 있도록 공통 함수를 만듭니다

앞에서 준비한 Calibration 이미지는 총 12장입니다.

```
OK 6장
NG 6장
```

이제 각 이미지에 4일차에서 배운 다음 처리를 반복해야 합니다.

```
Reference 읽기
        ↓
Template Matching
        ↓
SIFT 특징점 비교
        ↓
Good Match 선택
        ↓
RANSAC
        ↓
Homography
        ↓
측정값 반환
```

그런데 이 코드를 이미지마다 다시 작성하면 같은 코드가 계속 반복됩니다.

예를 들어:

```
ok_01.jpg용 코드
ok_02.jpg용 코드
ok_03.jpg용 코드
...
ng_06.jpg용 코드
```

를 따로 만드는 것은 좋지 않습니다.

그래서 실제 프로그램에서는 반복해서 사용하는 기능을 함수로 만들어 한곳에 정리합니다.

```
이미지 한 장 입력
        ↓
공통 측정 함수 실행
        ↓
측정 결과 반환
```

5일차에서는 다음 파일을 공통 측정 모듈로 사용합니다.

```
src/day05_logo_features.py
```

> 이 파일은 새로운 검사 기술을 배우기 위한 코드가 아닙니다.  
> 4일차에서 배운 Template Matching, SIFT, Homography를 여러 이미지에 반복 적용하기 쉽게 함수로 정리하는 단계입니다.

---

# 14. `day05_logo_features.py` — 로고 측정 공통 함수를 만듭니다

파일을 만듭니다.

```
src/day05_logo_features.py
```

현재 단계에서는 측정에 필요한 기능만 만듭니다.

아직 Threshold와 Rule을 결정하지 않았으므로 다음 기능은 넣지 않습니다.

```
OK / NG 판정
FP / FN 계산
NG Reason
Rule JSON 적용
```

이 기능들은 Calibration 결과를 확인하고 Threshold를 정한 뒤 추가합니다.

다음 코드를 작성합니다.

```python
from pathlib import Path

import cv2
import numpy as np


# --------------------------------------------------
# 1. 이미지 읽기
# --------------------------------------------------

def read_image(path):

    path = Path(path)

    image = cv2.imread(
        str(path)
    )

    if image is None:
        raise FileNotFoundError(
            f"이미지를 읽지 못했습니다: {path}"
        )

    return image


# --------------------------------------------------
# 2. Reference 준비
# --------------------------------------------------

def prepare_reference(reference_path):

    reference = read_image(
        reference_path
    )

    reference_gray = cv2.cvtColor(
        reference,
        cv2.COLOR_BGR2GRAY
    )


    # SIFT를 사용할 수 있는지 확인
    if not hasattr(
        cv2,
        "SIFT_create"
    ):
        raise RuntimeError(
            "현재 OpenCV에서 SIFT를 사용할 수 없습니다."
        )


    # SIFT 객체 생성
    sift = cv2.SIFT_create(
        nfeatures=1500
    )


    # 기준 로고의 특징점과 Descriptor 생성
    ref_keypoints, ref_descriptors = (
        sift.detectAndCompute(
            reference_gray,
            None
        )
    )


    if (
        ref_descriptors is None
        or len(ref_keypoints) < 4
    ):
        raise RuntimeError(
            "Reference에서 충분한 SIFT 특징점을 찾지 못했습니다."
        )


    return {
        "image": reference,
        "gray": reference_gray,
        "sift": sift,
        "keypoints": ref_keypoints,
        "descriptors": ref_descriptors,
        "width": reference.shape[1],
        "height": reference.shape[0],
    }


# --------------------------------------------------
# 3. Target 한 장 측정
# --------------------------------------------------

def measure_target(
    target_path,
    reference_data
):

    target = read_image(
        target_path
    )

    target_gray = cv2.cvtColor(
        target,
        cv2.COLOR_BGR2GRAY
    )


    # Reference에서 미리 준비한 값 가져오기
    reference_gray = (
        reference_data["gray"]
    )

    ref_keypoints = (
        reference_data["keypoints"]
    )

    ref_descriptors = (
        reference_data["descriptors"]
    )

    sift = (
        reference_data["sift"]
    )


    # ==================================================
    # 1. Template Matching
    # ==================================================

    template_result = cv2.matchTemplate(
        target_gray,
        reference_gray,
        cv2.TM_CCOEFF_NORMED
    )


    (
        _,
        template_score,
        _,
        template_location
    ) = cv2.minMaxLoc(
        template_result
    )


    # ==================================================
    # 2. Target SIFT 특징점
    # ==================================================

    target_keypoints, target_descriptors = (
        sift.detectAndCompute(
            target_gray,
            None
        )
    )


    good_matches = []


    if target_descriptors is not None:

        matcher = cv2.BFMatcher(
            cv2.NORM_L2
        )


        knn_matches = matcher.knnMatch(
            ref_descriptors,
            target_descriptors,
            k=2
        )


        # Ratio Test
        for pair in knn_matches:

            if len(pair) < 2:
                continue


            m, n = pair


            if (
                m.distance
                < 0.75 * n.distance
            ):

                good_matches.append(
                    m
                )


    # ==================================================
    # 3. RANSAC + Homography
    # ==================================================

    homography_found = 0

    inlier_count = 0

    inlier_ratio = 0.0

    polygon = None


    if len(good_matches) >= 4:

        source_points = np.float32(
            [
                ref_keypoints[
                    match.queryIdx
                ].pt
                for match in good_matches
            ]
        ).reshape(
            -1,
            1,
            2
        )


        target_points = np.float32(
            [
                target_keypoints[
                    match.trainIdx
                ].pt
                for match in good_matches
            ]
        ).reshape(
            -1,
            1,
            2
        )


        homography, mask = (
            cv2.findHomography(
                source_points,
                target_points,
                cv2.RANSAC,
                5.0
            )
        )


        if (
            homography is not None
            and mask is not None
        ):

            homography_found = 1


            # RANSAC을 통과한 Match 개수
            inlier_count = int(
                mask.ravel().sum()
            )


            # Good Match 중 Inlier 비율
            inlier_ratio = (
                inlier_count
                / len(good_matches)
            )


            # ------------------------------------------
            # Reference 로고 영역을 Target에 투영
            # ------------------------------------------

            ref_width = (
                reference_data["width"]
            )

            ref_height = (
                reference_data["height"]
            )


            corners = np.float32(
                [
                    [0, 0],

                    [
                        ref_width - 1,
                        0
                    ],

                    [
                        ref_width - 1,
                        ref_height - 1
                    ],

                    [
                        0,
                        ref_height - 1
                    ],
                ]
            ).reshape(
                -1,
                1,
                2
            )


            projected = (
                cv2.perspectiveTransform(
                    corners,
                    homography
                )
            )


            if np.isfinite(
                projected
            ).all():

                polygon = (
                    projected
                    .reshape(-1, 2)
                )


    # ==================================================
    # 측정 결과 반환
    # ==================================================

    return {

        "target_path":
            str(target_path),

        "template_score":
            float(template_score),

        "template_x":
            int(
                template_location[0]
            ),

        "template_y":
            int(
                template_location[1]
            ),

        "sift_good_matches":
            int(
                len(good_matches)
            ),

        "homography_found":
            int(
                homography_found
            ),

        "homography_inliers":
            int(
                inlier_count
            ),

        "inlier_ratio":
            float(
                inlier_ratio
            ),

        "polygon":
            polygon,

        "target_image":
            target,
    }
```

---

## 14.1 이 공통 함수는 무엇을 하나요?

코드가 길어 보이지만 역할은 크게 세 부분입니다.

### `read_image()`

이미지를 읽습니다.

```
이미지 경로
        ↓
OpenCV로 이미지 읽기
        ↓
이미지 반환
```

---

### `prepare_reference()`

4일차에서 만든 기준 로고를 준비합니다.

사용하는 파일은 다음입니다.

```
samples/day04/reference/reference_label.jpg
```

처리 과정은:

```
reference_label.jpg
        ↓
Gray
        ↓
SIFT
        ↓
Keypoint
        ↓
Descriptor
```

입니다.

기준 로고는 12장의 Calibration 이미지에서 계속 똑같이 사용합니다.

따라서 이미지마다 다시 만들 필요 없이 한 번만 계산해서 재사용합니다.

---

### `measure_target()`

이 함수가 실제 측정을 담당합니다.

예를 들어:

```
samples/day05/resized/ok_01.jpg
```

을 넣으면:

```
reference_label.jpg
        +
ok_01.jpg
        ↓
Template Matching
        ↓
SIFT Matching
        ↓
RANSAC
        ↓
Homography
```

를 실행한 뒤 다음 값을 반환합니다.

```
template_score

sift_good_matches

homography_found

homography_inliers

inlier_ratio
```

즉 앞으로는 긴 코드를 다시 작성하지 않고:

```python
measurement = measure_target(
    image_path,
    reference_data
)
```

처럼 호출하면 이미지 한 장의 검사값을 한 번에 측정할 수 있습니다.

---

# 15. 공통 함수가 실제 이미지에서 동작하는지 먼저 확인합니다

공통 함수를 만들었다고 바로 Calibration 12장을 처리하지 않습니다.

먼저 이미지 한 장만 넣어 정상적으로 측정되는지 확인합니다.

이런 확인 작업을 흔히 간단한 Smoke Test라고 생각하면 됩니다.

```
공통 함수 작성
        ↓
이미지 1장으로 먼저 확인
        ↓
정상 동작
        ↓
Calibration 12장 전체 적용
```

확인용 파일을 만듭니다.

```
src/33_day05_check_features.py
```

다음 코드를 작성합니다.

```python
from pathlib import Path

from day05_logo_features import (
    prepare_reference,
    measure_target,
)


REFERENCE_PATH = Path(
    "samples/day04/reference/reference_label.jpg"
)

TARGET_PATH = Path(
    "samples/day05/resized/ok_01.jpg"
)


# --------------------------------------------------
# 1. Reference 준비
# --------------------------------------------------

reference_data = prepare_reference(
    REFERENCE_PATH
)


print(
    "Reference Keypoints:",
    len(
        reference_data["keypoints"]
    )
)


# --------------------------------------------------
# 2. 실제 Target 한 장 측정
# --------------------------------------------------

measurement = measure_target(
    TARGET_PATH,
    reference_data
)


# --------------------------------------------------
# 3. 결과 확인
# --------------------------------------------------

print()
print("=" * 60)

print(
    "Target:",
    TARGET_PATH.name
)

print(
    "Template Score:",
    round(
        measurement["template_score"],
        4
    )
)

print(
    "SIFT Good Match:",
    measurement[
        "sift_good_matches"
    ]
)

print(
    "Homography Found:",
    measurement[
        "homography_found"
    ]
)

print(
    "Homography Inlier:",
    measurement[
        "homography_inliers"
    ]
)

print(
    "Inlier Ratio:",
    round(
        measurement["inlier_ratio"],
        4
    )
)

print("=" * 60)
```

실행합니다.

```
python src/33_day05_check_features.py
```

---

## 15.1 여기서는 무엇을 확인하나요?

아직 OK / NG 판정을 하는 것이 아닙니다.

확인할 것은:

```
reference_label.jpg를 정상적으로 읽었는가?

SIFT 특징점을 만들었는가?

ok_01.jpg를 정상적으로 읽었는가?

Template Score가 계산되는가?

SIFT Good Match가 계산되는가?

Homography가 계산되는가?

Inlier Ratio가 계산되는가?
```

입니다.

출력은 다음과 같은 형태가 됩니다.

```
Reference Keypoints: 675

============================================================
Target: ok_01.jpg
Template Score: 0.5358
SIFT Good Match: 174
Homography Found: 1
Homography Inlier: 167
Inlier Ratio: 0.9598
============================================================
```

숫자는 현재 촬영한 광천김 이미지에 따라 달라집니다.

---

## 15.2 첫 번째 실제 측정 결과를 읽어봅니다

아직 이 숫자로 `OK / NG`를 판정하지 않습니다.

지금은 각 숫자가 무엇을 의미하는지 확인합니다.

### Reference Keypoints = 675

```
Reference Keypoints: 675
```

기준 이미지인 `reference_label.jpg`에서 SIFT가 675개의 특징점 후보를 찾았다는 뜻입니다.

광천김 로고에는 글자와 모서리, 선과 무늬가 많기 때문에 비교에 사용할 특징점이 충분히 만들어졌습니다.

### Template Score = 0.5358

```
Template Score: 0.5358
```

Template Matching으로 기준 로고와 `ok_01.jpg`를 비교했을 때 얻은 유사도입니다.

```
Template Matching
→ 기준 로고 전체 모양을 그대로 비교

현재 측정값
→ 0.5358
```

`ok_01.jpg`는 정상 이미지이지만 Score가 반드시 `0.9`처럼 매우 높게 나오는 것은 아닙니다.

실제 촬영에서는 위치, 크기, 각도, 밝기 등의 작은 차이가 생길 수 있기 때문입니다.

따라서 지금은:

```
0.5358이면 OK
```

라고 판단하지 않습니다.

나머지 OK와 NG 이미지에서도 값을 측정한 뒤 비교해야 합니다.

---

### SIFT Good Match = 174

```
SIFT Good Match: 174
```

기준 로고와 `ok_01.jpg` 사이에서 SIFT가 찾은 특징점 중 서로 비슷하다고 판단하여 남긴 대응점이 174개라는 뜻입니다.

```
Reference 특징점
        ↓
Target 특징점과 비교
        ↓
Ratio Test
        ↓
174개의 Good Match
```

즉 기준 로고와 검사 이미지 사이에서 상당히 많은 특징점이 연결되었습니다.

---

### Homography Inlier = 167

```
Homography Inlier: 167
```

174개의 Good Match가 모두 정확한 연결이라는 보장은 없습니다.

그래서 RANSAC을 사용하여 서로 일관된 위치 관계를 갖는 Match만 다시 선택했습니다.

```
Good Match 174개
        ↓
RANSAC
        ↓
실제로 일관성 있게 맞는 점
167개
```

이 167개를 `Inlier`라고 합니다.

---

### Inlier Ratio = 0.9598

```
Inlier Ratio: 0.9598
```

Good Match 174개 중 167개가 Inlier로 남았습니다.

```
167 / 174
≈ 0.9598
≈ 95.98%
```

즉, SIFT가 찾은 대응점의 약 96%가 Homography에서도 일관된 대응점으로 인정된 것입니다.

현재 `ok_01.jpg`에서는 기준 로고와 검사 이미지 사이의 특징점 관계가 상당히 안정적으로 만들어졌다고 볼 수 있습니다.

---

## 15.3 `Homography Found = 1`은 무엇인가요?

```
Homography Found: 1
```

은 다음 과정이 성공했다는 뜻입니다.

```
reference_label.jpg
        +
ok_01.jpg
        ↓
SIFT 특징점 비교
        ↓
Good Match 확보
        ↓
RANSAC
        ↓
Homography 계산 성공
```

즉 프로그램이 기준 로고와 검사 이미지 사이의 위치 관계를 계산할 수 있었다는 의미입니다.

반대로:

```
Homography Found: 0
```

이면 충분한 대응점을 확보하지 못했거나 일관된 Homography를 계산하지 못했다는 뜻입니다.

다만 여기서도 주의합니다.

```
Homography Found = 1
→ 무조건 OK라는 뜻 X

Homography Found = 0
→ 무조건 NG라는 뜻 X
```

현재는 **측정 결과**일 뿐입니다.

---

## 15.4 이번 한 장에서 무엇을 확인했나요?

`ok_01.jpg` 한 장을 측정해 보니 다음과 같은 특징이 나타났습니다.

```
Template Score
→ 0.5358

SIFT Good Match
→ 174개

Homography Inlier
→ 167개

Inlier Ratio
→ 0.9598

Homography
→ 계산 성공
```

특히 눈여겨볼 부분은 다음입니다.

```
Template Score는 0.5358

하지만

SIFT 특징점은 많이 연결되었고
그중 약 96%가 RANSAC에서도
일관된 대응점으로 남음
```

즉 같은 정상 이미지를 보더라도 검사 방법에 따라 결과를 표현하는 숫자가 다릅니다.

그래서 한 장의 결과만 보고 기준을 정하지 않습니다.

다음 단계에서는 동일한 공통 측정 함수를:

```
OK 6장
+
NG 6장
```

에 모두 적용합니다.

그 결과를 비교하면서:

> Template Score, SIFT Good Match, Inlier Ratio 중 어떤 값이 실제로 OK와 NG를 잘 구분하는가?

를 확인합니다.

이것이 다음 Calibration 실습의 핵심입니다.


---

## 15.5 왜 이미지 한 장만 먼저 확인했나요?

12장을 한꺼번에 처리했는데 코드에 문제가 있다면 어느 부분에서 문제가 생겼는지 찾기 어렵습니다.

따라서 실제 개발에서도:

```
공통 함수 작성
        ↓
대표 이미지 1장으로 확인
        ↓
정상 동작 확인
        ↓
여러 데이터에 반복 적용
```

순서로 진행하는 것이 안전합니다.

---
# PART3 **Calibration** — 12장 측정 → 분포 비교 → Rule 후보 찾기 (16~22)

---

# 16. Calibration 12장의 여러 검사값을 한 번에 측정합니다

앞에서 `ok_01.jpg` 한 장을 이용하여 공통 측정 함수가 정상적으로 동작하는 것을 확인했습니다.

이제 같은 측정을 Calibration 데이터 전체에 적용합니다.

사용할 이미지는 다음 12장입니다.

```
samples/day05/resized/

OK
├── ok_01.jpg
├── ok_02.jpg
├── ok_03.jpg
├── ok_04.jpg
├── ok_05.jpg
└── ok_06.jpg

NG
├── ng_01.jpg
├── ng_02.jpg
├── ng_03.jpg
├── ng_04.jpg
├── ng_05.jpg
└── ng_06.jpg
```

`test_ok_01.jpg`, `test_ng_01.jpg` 등의 Test 이미지는 아직 사용하지 않습니다.

---

## 15.1 이번 실습에서는 무엇을 하나요?

모든 Calibration 이미지에 완전히 같은 검사 조건을 적용합니다.

```
같은 Reference
reference_label.jpg

        ↓
같은 Template Matching
        ↓
같은 SIFT 설정
        ↓
같은 Ratio Test
        ↓
같은 RANSAC 조건
        ↓
같은 Homography 계산
```

그리고 이미지마다 다음 값을 기록합니다.

```
Template Score

SIFT Good Match

Homography 계산 성공 여부

Homography Inlier 수

Inlier Ratio
```

예를 들어:

```
ok_01.jpg
→ Template Score
→ SIFT Good Match
→ Inlier
→ Inlier Ratio

ok_02.jpg
→ 같은 값 측정

...

ng_06.jpg
→ 같은 값 측정
```

이렇게 해야 나중에:

```
OK 6장에서는 어떤 값이 나오는가?

NG 6장에서는 어떤 값이 나오는가?

어떤 측정값이
OK와 NG를 가장 잘 구분하는가?
```

를 비교할 수 있습니다.

> 지금은 여러 방법을 함께 **측정**하는 단계입니다. 여러 조건을 묶어서 OK / NG를 판정하는 **복합 Rule은 아직 만들지 않습니다.**

---

## 15.2 왜 여러 값을 함께 측정하나요?

한 가지 측정값만 보고 검사 기준을 만들면 실제 촬영환경에서 불안정할 수 있습니다.

예를 들어 정상 이미지인데도:

```
Template Score
→ 생각보다 낮을 수 있음
```

앞에서 실제 `ok_01.jpg`에서도:

```
Template Score = 0.5358
```

이 나왔습니다.

하지만 같은 이미지에서:

```
SIFT Good Match = 174

Homography Inlier = 167

Inlier Ratio = 0.9598
```

로 특징점 대응은 상당히 안정적이었습니다.

따라서 지금부터는 한 값만 보는 것이 아니라:

```
Template은 어떻게 변하는가?

SIFT는 어떻게 변하는가?

Homography는 어떻게 변하는가?
```

를 같은 12장의 이미지에서 나란히 비교합니다.

---

# 17. `34_day05_measure_calibration.py` — Calibration 전체를 측정합니다

==`Calibration 12장 측정 : 이미지에서 숫자를 뽑아 CSV에 모으는 코드`==

먼저 12장을 측정합니다

```
Calibration 12장
OK 6장 + NG 6장
        ↓
34_day05_measure_calibration.py
        ↓
각 이미지에서 측정

Template Score
SIFT Good Match
Homography Inlier
Inlier Ratio
        ↓
CSV 저장
```

결과:

```
reports/day05_calibration_measurements.csv
```

아직 Rule을 찾는 코드가 아니고 검사값을 수집하는 코드를 작성합니다.

파일을 만듭니다.

```
src/34_day05_measure_calibration.py
```

다음 코드를 작성합니다.

```python
from pathlib import Path
import csv

from day05_logo_features import (
    prepare_reference,
    measure_target,
)


# --------------------------------------------------
# 경로 설정
# --------------------------------------------------

REFERENCE_PATH = Path(
    "samples/day04/reference/reference_label.jpg"
)

DATA_DIR = Path(
    "samples/day05/resized"
)

OUTPUT_CSV = Path(
    "reports/day05_calibration_measurements.csv"
)


# --------------------------------------------------
# Calibration 이미지 목록을 만듭니다.
# --------------------------------------------------

def find_calibration_images():

    image_infos = []


    # OK 6장
    for image_path in sorted(
        DATA_DIR.glob("ok_*.jpg")
    ):

        image_infos.append(
            (
                image_path,
                "OK"
            )
        )


    # NG 6장
    for image_path in sorted(
        DATA_DIR.glob("ng_*.jpg")
    ):

        image_infos.append(
            (
                image_path,
                "NG"
            )
        )


    return image_infos


# --------------------------------------------------
# 실행
# --------------------------------------------------

def main():

    OUTPUT_CSV.parent.mkdir(
        parents=True,
        exist_ok=True
    )


    # ==============================================
    # 1. 기준 로고 준비
    # ==============================================

    reference_data = prepare_reference(
        REFERENCE_PATH
    )


    print(
        "Reference Keypoints:",
        len(
            reference_data["keypoints"]
        )
    )


    # ==============================================
    # 2. Calibration 이미지 확인
    # ==============================================

    image_infos = (
        find_calibration_images()
    )


    print(
        "Calibration Images:",
        len(image_infos)
    )


    if len(image_infos) != 12:

        print(
            "주의: Calibration 이미지는 "
            "OK 6장 + NG 6장 = 12장을 예상합니다."
        )


    rows = []


    # ==============================================
    # 3. 12장에 같은 측정을 반복
    # ==============================================

    for image_path, actual in image_infos:

        measurement = measure_target(
            image_path,
            reference_data
        )


        row = {

            "file":
                image_path.name,

            "actual":
                actual,

            "template_score":
                round(
                    measurement[
                        "template_score"
                    ],
                    6
                ),

            "sift_good_matches":
                measurement[
                    "sift_good_matches"
                ],

            "homography_found":
                measurement[
                    "homography_found"
                ],

            "homography_inliers":
                measurement[
                    "homography_inliers"
                ],

            "inlier_ratio":
                round(
                    measurement[
                        "inlier_ratio"
                    ],
                    6
                ),
        }


        rows.append(
            row
        )


        print(
            row
        )


    # ==============================================
    # 4. CSV 저장
    # ==============================================

    fieldnames = [

        "file",

        "actual",

        "template_score",

        "sift_good_matches",

        "homography_found",

        "homography_inliers",

        "inlier_ratio",
    ]


    with OUTPUT_CSV.open(
        "w",
        encoding="utf-8-sig",
        newline=""
    ) as file:

        writer = csv.DictWriter(
            file,
            fieldnames=fieldnames
        )

        writer.writeheader()

        writer.writerows(
            rows
        )


    # ==============================================
    # 5. 완료 확인
    # ==============================================

    print()
    print("=" * 70)

    print(
        "Calibration 측정 완료"
    )

    print(
        "전체 이미지:",
        len(rows)
    )

    print(
        "저장:",
        OUTPUT_CSV
    )

    print("=" * 70)


if __name__ == "__main__":
    main()
```

---

### 코드 실행

프로젝트 루트에서 실행합니다.

```bash
python src/34_day05_measure_calibration.py
```

프로그램은 다음 순서로 동작합니다.

```
reference_label.jpg 준비

        ↓

SIFT 특징점 미리 계산

        ↓

ok_01.jpg 측정
ok_02.jpg 측정
ok_03.jpg 측정
ok_04.jpg 측정
ok_05.jpg 측정
ok_06.jpg 측정

        ↓

ng_01.jpg 측정
ng_02.jpg 측정
ng_03.jpg 측정
ng_04.jpg 측정
ng_05.jpg 측정
ng_06.jpg 측정

        ↓

12장의 측정값을
CSV에 한 줄씩 저장
```

### 의사코드

```
1. 광천김 기준 로고 이미지를 읽는다.

2. 기준 로고에서
   SIFT 특징점과 Descriptor를 한 번 만든다.

3. samples/day05/resized 폴더에서
   ok_로 시작하는 이미지 6장을 찾는다.

4. 이 이미지들은 실제 상태를 OK라고 기록한다.

5. ng_로 시작하는 이미지 6장을 찾는다.

6. 이 이미지들은 실제 상태를 NG라고 기록한다.

7. 첫 번째 이미지를 공통 측정 함수에 넣는다.

8. Template Score를 측정한다.

9. SIFT Good Match 수를 측정한다.

10. RANSAC + Homography를 계산한다.

11. Homography가 만들어졌는지 확인한다.

12. Inlier 수를 계산한다.

13. Inlier Ratio를 계산한다.

14. 측정값을 한 줄의 데이터로 만든다.

15. 나머지 Calibration 이미지에도
    똑같은 과정을 반복한다.

16. 총 12장의 결과를 CSV에 저장한다.
```

Calibration 측정 결과

```
Reference Keypoints: 675
Calibration Images: 12
{'file': 'ok_01.jpg', 'actual': 'OK', 'template_score': 0.535846, 'sift_good_matches': 174, 'homography_found': 1, 'homography_inliers': 167, 'inlier_ratio': 0.95977}
{'file': 'ok_02.jpg', 'actual': 'OK', 'template_score': 0.544862, 'sift_good_matches': 174, 'homography_found': 1, 'homography_inliers': 158, 'inlier_ratio': 0.908046}
{'file': 'ok_03.jpg', 'actual': 'OK', 'template_score': 0.556364, 'sift_good_matches': 184, 'homography_found': 1, 'homography_inliers': 164, 'inlier_ratio': 0.891304}
{'file': 'ok_04.jpg', 'actual': 'OK', 'template_score': 0.425379, 'sift_good_matches': 151, 'homography_found': 1, 'homography_inliers': 142, 'inlier_ratio': 0.940397}
{'file': 'ok_05.jpg', 'actual': 'OK', 'template_score': 0.552585, 'sift_good_matches': 161, 'homography_found': 1, 'homography_inliers': 147, 'inlier_ratio': 0.913043}
{'file': 'ok_06.jpg', 'actual': 'OK', 'template_score': 0.440258, 'sift_good_matches': 74, 'homography_found': 1, 'homography_inliers': 60, 'inlier_ratio': 0.810811}
{'file': 'ng_01.jpg', 'actual': 'NG', 'template_score': 0.531258, 'sift_good_matches': 126, 'homography_found': 1, 'homography_inliers': 120, 'inlier_ratio': 0.952381}
{'file': 'ng_02.jpg', 'actual': 'NG', 'template_score': 0.442885, 'sift_good_matches': 99, 'homography_found': 1, 'homography_inliers': 81, 'inlier_ratio': 0.818182}
{'file': 'ng_03.jpg', 'actual': 'NG', 'template_score': 0.484058, 'sift_good_matches': 141, 'homography_found': 1, 'homography_inliers': 109, 'inlier_ratio': 0.77305}
{'file': 'ng_04.jpg', 'actual': 'NG', 'template_score': 0.536214, 'sift_good_matches': 161, 'homography_found': 1, 'homography_inliers': 139, 'inlier_ratio': 0.863354}
{'file': 'ng_05.jpg', 'actual': 'NG', 'template_score': 0.525607, 'sift_good_matches': 107, 'homography_found': 1, 'homography_inliers': 94, 'inlier_ratio': 0.878505}
{'file': 'ng_06.jpg', 'actual': 'NG', 'template_score': 0.461591, 'sift_good_matches': 109, 'homography_found': 1, 'homography_inliers': 91, 'inlier_ratio': 0.834862}

======================================================================
Calibration 측정 완료
전체 이미지: 12
저장: reports/day05_calibration_measurements.csv
```

현재 결과물은:

```
터미널
→ 이미지별 측정값 확인

CSV
→ 12장의 측정값 저장
```

입니다.

---

## 17.1 결과는 어디에 저장되나요?

실행이 끝나면 다음 파일이 생성됩니다.

```
reports/day05_calibration_measurements.csv
```

CSV에는 다음 항목이 들어갑니다.

|항목|의미|
|---|---|
|`file`|검사한 이미지 이름|
|`actual`|실제 정답인 OK / NG|
|`template_score`|Template Matching 유사도|
|`sift_good_matches`|SIFT에서 남은 좋은 대응점 수|
|`homography_found`|Homography 계산 성공 여부|
|`homography_inliers`|RANSAC을 통과한 대응점 수|
|`inlier_ratio`|Good Match 중 Inlier의 비율|

| 파일          | 실제 상태 | Template | SIFT Good | Inlier | Inlier Ratio |
| ----------- | ----- | -------: | --------: | -----: | -----------: |
| `ok_01.jpg` | OK    |   0.5358 |       174 |    167 |       0.9598 |
| `ok_02.jpg` | OK    |   0.5449 |       174 |    158 |       0.9080 |
| `ok_03.jpg` | OK    |   0.5564 |       184 |    164 |       0.8913 |
| `ok_04.jpg` | OK    |   0.4254 |       151 |    142 |       0.9404 |
| `ok_05.jpg` | OK    |   0.5526 |       161 |    147 |       0.9130 |
| `ok_06.jpg` | OK    |   0.4403 |        74 |     60 |       0.8108 |
| `ng_01.jpg` | NG    |   0.5313 |       126 |    120 |       0.9524 |
| `ng_02.jpg` | NG    |   0.4429 |        99 |     81 |       0.8182 |
| `ng_03.jpg` | NG    |   0.4841 |       141 |    109 |       0.7731 |
| `ng_04.jpg` | NG    |   0.5362 |       161 |    139 |       0.8634 |
| `ng_05.jpg` | NG    |   0.5256 |       107 |     94 |       0.8785 |
| `ng_06.jpg` | NG    |   0.4616 |       109 |     91 |       0.8349 |
```
ok_01.jpg
→ 사람이 알고 있는 실제 정답 = OK

ng_01.jpg
→ 사람이 알고 있는 실제 정답 = NG
```

그 옆의 숫자들이 OpenCV가 실제 이미지에서 측정한 값입니다.

---

## 17.2 먼저 Template Score를 비교합니다

정상과 불량의 범위를 확인합니다.

```
OK
0.4254 ~ 0.5564

NG
0.4429 ~ 0.5362
```

두 범위를 놓고 보면 상당히 많이 겹칩니다.

```
OK  ────────────────
     0.425       0.556

NG     ────────────
       0.443     0.536
```

예를 들어:

```
ok_04.jpg
Template = 0.4254

ng_04.jpg
Template = 0.5362
```

입니다.

오히려 NG인 `ng_04.jpg`의 Template Score가 정상인 `ok_04.jpg`보다 높습니다.

따라서 현재 데이터에서는:

> Template Score가 높다고 무조건 OK라고 판정할 수 없습니다.

라는 사실을 확인할 수 있습니다.

4일차에서 배운 것처럼 Template Matching은 촬영 위치·회전·크기 변화에 영향을 받을 수 있기 때문입니다.

쉽게 다시 설명하면

```
OK와 NG의 Template Score가 서로 다른 구간으로 나뉘지 않고 같은 숫자 영역에 섞여 있다는 뜻
```

입니다.

예를 들어 `0.50` 정도의 값은 OK에서도 나올 수 있고 NG에서도 나올 수 있습니다.

```
OK 범위 : 0.4254 ~ 0.5564 
NG 범위 : 0.4429 ~ 0.5362 

겹치는 구간 
→ 약 0.4429 ~ 0.5362
```

즉 이 겹치는 구간에서는 Template Score만 보고는 이 이미지가 OK인지 NG인지 확실히 구분하기 어렵습니다.

따라서 Template Score 하나만으로 기준을 만들기보다, SIFT Good Match나 Inlier Ratio 같은 다른 측정값도 함께 비교해야 합니다.

---

## 17.3 SIFT Good Match를 비교합니다

이번에는 특징점 대응 개수를 확인합니다.

```
OK
74 ~ 184개

NG
99 ~ 161개
```

평균적으로 보면:

```
OK 평균
약 153개

NG 평균
약 124개
```

로 정상 이미지에서 더 많은 Match가 만들어졌습니다.

하지만 이것도 완전히 분리되지는 않습니다.

예를 들어:

```
ok_06.jpg
→ 74개

ng_04.jpg
→ 161개
```

입니다.

즉:

```
SIFT Match가 많다
→ 정상일 가능성이 높아질 수 있음

하지만

Match 수 하나만으로
모든 OK / NG를 구분할 수는 없음
```

이라는 것을 실제 데이터에서 확인했습니다.

---

## 17.4 Homography Inlier도 비교합니다

RANSAC을 통과한 실제 대응점 수입니다.

```
OK
60 ~ 167개

NG
81 ~ 139개
```

참고] RANSAC(랜색): 여러 Match 중에서 잘못 연결된 점을 제외하고, 서로 일관되게 맞는 점들만 골라내는 방법.


평균은:

```
OK
약 140개

NG
약 106개
```

입니다.

정상 이미지가 전체적으로 높은 편이지만 여기에서도 범위가 겹칩니다.

따라서 Inlier 개수 역시 단독으로 바로 Threshold를 정하기에는 부족할 가능성이 있습니다.

---

## 17.5 Inlier Ratio를 비교합니다

이번에는 Good Match 중 실제로 일관된 Match의 비율을 확인합니다.

```
OK
0.8108 ~ 0.9598

NG
0.7731 ~ 0.9524
```

정상 평균은 약:

```
0.904
```

불량 평균은 약:

```
0.853
```

으로 정상 쪽이 조금 더 높습니다.

하지만 중요한 결과가 하나 있습니다.

```
ok_06.jpg
→ 0.8108

ng_01.jpg
→ 0.9524
```

불량인 `ng_01.jpg`가 정상인 `ok_06.jpg`보다 훨씬 높은 Inlier Ratio를 보입니다.

따라서:

> Inlier Ratio가 높다고 반드시 정상 로고라는 뜻도 아닙니다.

SIFT와 Homography는 로고 일부가 가려져 있더라도 남아 있는 특징점끼리 정확하게 대응되면 높은 값을 만들 수 있기 때문입니다.

이것이 이번 NG 데이터에서 확인한 중요한 특징입니다.

---

## 17.6 Homography Found는 어떻게 나왔나요?

12장 모두:

```
homography_found = 1
```

이 나왔습니다.

즉:

```
OK 6장
→ Homography 계산 성공

NG 6장
→ Homography 계산 성공
```

입니다.

왜 NG인데도 Homography를 찾았을까요?

현재 NG 이미지는 로고를 일부 가린 형태입니다.

```
로고 일부 가림
        ↓
가려지지 않은 글자와 모서리는 남아 있음
        ↓
SIFT가 남아 있는 특징점을 찾음
        ↓
Homography 계산 가능
```

따라서:

```
Homography Found = 1
→ 로고와 대응되는 특징점이 존재한다
```

는 뜻이지,

```
Homography Found = 1
→ 제품이 정상이다
```

라는 뜻은 아닙니다.

---

## 17.7 이번 Calibration에서 발견한 핵심

처음에는 다음과 같이 예상했을 수 있습니다.

```
OK
→ 높은 값

NG
→ 낮은 값

        ↓

중간에 Threshold 하나 설정
```

하지만 실제 결과는 그렇게 단순하지 않았습니다.

```
Template Score
→ OK / NG가 많이 겹침

SIFT Good Match
→ 정상 평균은 높지만 일부 겹침

Homography Inlier
→ 정상 평균은 높지만 일부 겹침

Inlier Ratio
→ 정상 평균은 높지만 일부 NG도 매우 높음

Homography Found
→ 12장 모두 1
```

즉 현재 데이터에서는 한 가지 숫자만 보고 정상과 불량을 완벽하게 나누기 어렵습니다.

이 결과는 실습 실패가 아닙니다.

오히려 이것이 Calibration을 하는 이유입니다.

```
실제 데이터 측정
        ↓
예상과 실제가 같은지 확인
        ↓
값이 겹치는 것을 발견
        ↓
어떤 Feature를 사용할지 고민
        ↓
여러 조건을 조합할 필요가 있는지 확인
```

---

## 17.8 특히 확인해야 할 이미지가 있습니다

다음 이미지는 이후 분석에서 다시 살펴볼 가치가 있습니다.

실제 사진을 다시 보고:

```
회전이 컸는가?
거리가 달랐는가?
로고가 작게 촬영되었는가?
Blur가 있었는가?
빛 반사가 있었는가?
```

를 확인할 필요가 있습니다.

실제 이미지를 보면

```
ok_06.jpg

SIFT Good Match = 74
Inlier = 60
Inlier Ratio = 0.8108
```

정상 이미지인데 다른 OK 이미지보다 값이 크게 낮습니다.

`ok_06.jpg`
![[Pasted image 20260831190251.png|325]]

### 실제 이미지를 확인한 결과

사진을 보면 제품이 다른 정상 이미지보다 조금 멀리 촬영되어 로고가 작게 보이고, 촬영 각도와 포장지의 반사도 조금 다릅니다.

```
로고 자체
→ 정상

하지만

촬영 거리 / 크기 / 각도 차이
        ↓
특징점 Matching 감소
        ↓
SIFT와 Inlier 값이 낮아짐
```

즉 제품은 정상이어도 촬영조건이 달라지면 Matching 값이 낮아질 수 있음을 확인할 수 있습니다.


반대로:

```
ng_01.jpg

SIFT Good Match = 126
Inlier = 120
Inlier Ratio = 0.9524
```

는 불량인데도 매우 높은 Inlier Ratio가 나왔습니다.

`ng_01.jpg`
![[Pasted image 20260831190350.png|357]]

### 실제 이미지를 확인한 결과

카드로 로고 일부를 가렸지만 가려지지 않은 영역에는 여전히 로고의 글자와 특징점이 많이 남아 있습니다.

```
로고 일부 가림
        ↓
Good Match 수는 감소
        ↓
하지만 남아 있는 Match들은
서로 위치관계가 매우 잘 맞음
        ↓
Inlier Ratio는 높게 나옴
```

즉 Inlier Ratio가 높다고 반드시 정상이라는 뜻은 아니라는 것을 실제 이미지에서 확인할 수 있습니다.


또:

```
ng_04.jpg

SIFT Good Match = 161
Inlier = 139
```

NG인데도 SIFT Good Match가 정상 이미지와 비슷하게 많이 나왔습니다.

`ng_04.jpg`
![[Pasted image 20260831190427.png|357]]

### 실제 이미지를 확인한 결과

로고 일부가 가려져 있지만 **주변의 글자와 로고 특징이 상당 부분 그대로 남아 있기 때문에 SIFT가 많은 특징점을 연결할 수 있습니다.**

```
일부 로고 가림
        ↓
남아 있는 특징이 많음
        ↓
SIFT Good Match는 높게 나옴

하지만

일부 대응점의 위치관계가 불안정
        ↓
Inlier Ratio는 0.8634로 낮아짐
```

따라서 SIFT Match 개수 하나만으로도 OK / NG를 판단하기 어렵다는 것을 확인할 수 있습니다.

### 세 이미지를 통해 확인한 핵심

```
ok_06
→ 정상이어도 촬영조건 때문에 값이 낮아질 수 있음

ng_01
→ 불량이어도 Inlier Ratio가 높을 수 있음

ng_04
→ 불량이어도 SIFT Match가 많을 수 있음
```

> 그래서 하나의 측정값만으로 판정하지 않고, SIFT Good Match와 Inlier Ratio 같은 여러 값을 함께 비교하여 복합 Rule을 만드는 것입니다.

---

## 17.9 지금까지 한 일을 정리합니다

이번 실습에서는:

```
광천김 Calibration 이미지 12장
        ↓
같은 Reference 사용
        ↓
같은 측정 조건 적용
        ↓
Template Score
SIFT Good Match
Homography Inlier
Inlier Ratio
        ↓
CSV 저장
        ↓
OK와 NG 결과 비교
```

까지 진행했습니다.

아직 Threshold와 Rule은 정하지 않았습니다.

실제 결과를 보니 한 가지 Feature로는 정상과 불량이 깔끔하게 나뉘지 않았습니다.

따라서 다음 단계에서는 이 CSV를 이용하여:

```
어떤 Threshold 후보가 있는가?
        ↓
한 가지 조건으로 충분한가?
        ↓
여러 Feature를 조합하면
더 잘 구분할 수 있는가?
        ↓
복합 Rule 후보 만들기
```

를 실제 데이터로 확인합니다.

---

# 18. 이제 Rule 후보에 사용할 값을 정합니다

앞에서 Calibration 12장의 실제 측정값을 비교했습니다.

그 결과 한 가지 값만으로는 OK와 NG를 확실하게 나누기 어렵다는 것을 확인했습니다.

그래서 이번에는 다음 두 값을 함께 사용합니다.

| 측정값               | 쉽게 말하면                        |
| ----------------- | ----------------------------- |
| `SIFT Good Match` | 비슷한 특징점이 충분히 많이 연결되었는가?       |
| `Inlier Ratio`    | 연결된 특징점들이 실제 위치관계에서도 제대로 맞는가? |

쉽게 생각하면 다음과 같습니다.

```
특징점이 충분히 많이 연결되었는가?
                +
그 연결이 실제로도 잘 맞는가?
```

따라서 앞으로 시험할 Rule의 기본 모양은 다음과 같습니다.

![[Pasted image 20260831201251.png]]

```
Homography 계산 성공

        AND

SIFT Good Match >= ?

        AND

Inlier Ratio >= ?

        ↓

모두 만족
→ OK

하나라도 부족
→ NG
```

여기서 아직 모르는 것은 두 개입니다.

```
SIFT는 몇 이상이어야 할까?

Inlier Ratio는 몇 이상이어야 할까?
```

이 숫자가 바로 Threshold입니다.

> 다음 단계에서는 이 숫자를 사람이 감으로 정하지 않고, Calibration 데이터를 이용하여 여러 Threshold 후보를 직접 시험해 봅니다.

---

# 19. 여러 Threshold 후보를 시험하여 좋은 Rule을 찾아봅니다

이제 해야 할 일은 간단합니다.

```
SIFT 기준값
        +
Inlier Ratio 기준값
        ↓
어떤 조합이
OK와 NG를 가장 잘 구분할까?
```

예를 들어 다음과 같은 여러 후보를 생각할 수 있습니다.

![[Pasted image 20260831201736.png]]

```
후보 A
SIFT >= 100
Inlier Ratio >= 0.80

후보 B
SIFT >= 120
Inlier Ratio >= 0.85

후보 C
SIFT >= 140
Inlier Ratio >= 0.90
```

하지만 어떤 후보가 좋은지는 숫자만 보고 알 수 없습니다.

그래서 각 후보를 Calibration 12장에 실제로 적용해 봅니다.

```
Threshold 후보 하나 선택
        ↓
Calibration 12장에 적용
        ↓
프로그램이 OK / NG 판정
        ↓
사람이 알고 있는 실제 정답과 비교
        ↓
TP / TN / FP / FN 확인
        ↓
오류 개수 기록
```

그리고 다른 Threshold 조합으로 같은 작업을 반복합니다.

```
후보 A 적용
→ 오류 확인

후보 B 적용
→ 오류 확인

후보 C 적용
→ 오류 확인

...

        ↓

오류가 적은 후보 찾기
```

특히 이번 제조검사에서는 **불량을 OK로 잘못 통과시키는 FN을 중요하게 봅니다.**

따라서 프로그램은 후보들을 비교한 뒤:

```
FN이 적고
        ↓
전체 오류가 적고
        ↓
FP도 적은 후보
```

를 위쪽에 보여줍니다.

---

## 19.1 그래프로 생각하면 더 쉽습니다

이번 Rule은 두 개의 값을 사용하므로 X-Y 그래프로 볼 수 있습니다.

![[Pasted image 20260831202313.png]]

```
Y축
Inlier Ratio
    ↑
    │                   ● OK
    │             ●
    │
────┼────────────────────  ← Inlier Threshold
    │         × NG
    │    ×
    │
    └────────│────────────→ X축
             SIFT Threshold
             SIFT Good Match
```

기준선은 두 개입니다.

```
세로선
→ SIFT Good Match 기준

가로선
→ Inlier Ratio 기준
```

현재 Rule에서는:

```
세로선보다 오른쪽
        AND
가로선보다 위쪽
        ↓
OK
```

입니다.

즉 Threshold 후보를 찾는다는 것은 쉽게 말하면:

> “세로선과 가로선을 어디에 그어야 실제 OK와 NG를 가장 적게 틀릴까?”를 여러 번 시험하는 과정입니다.

---

## 19.2 전체 과정을 한 번에 보면

![[Pasted image 20260831202603.png]]

```
Calibration 측정값 CSV
        ↓
SIFT Threshold 후보 만들기
        +
Inlier Ratio Threshold 후보 만들기
        ↓
여러 조합 만들기
        ↓
Calibration 12장에 적용
        ↓
실제 OK / NG와 비교
        ↓
TP / TN / FP / FN 계산
        ↓
오류가 적은 후보 순서로 정렬
        ↓
좋은 Rule 후보 확인
```

이 작업을 프로그램으로 수행하는 것이 다음 코드입니다.

```
src/35_day05_analyze_calibration.py
```

이 프로그램의 입력과 출력은 다음과 같습니다.

```
입력
reports/day05_calibration_measurements.csv

        ↓

35_day05_analyze_calibration.py

        ↓

출력 1
reports/day05_rule_candidates.csv
→ Threshold 후보별 결과

출력 2
outputs/day05/calibration/
sift_inlier_distribution.png
→ OK / NG 분포 그래프
```

> **이번 실습의 핵심은 좋은 Threshold 숫자를 외우는 것이 아니라, 여러 후보를 실제 데이터에 적용하여 오류가 적은 기준을 찾는 방법을 이해하는 것입니다.**




![[Pasted image 20260831153638.png]]

---

## 19.3 `35_day05_analyze_calibration.py` — 후보 Rule을 비교합니다

==`측정값 분석 + Rule 후보 탐색 :  CSV를 보고 어떤 기준값이 좋은지 찾아보는 코드 `==

`34_day05_measure_calibration.py`에서 만든 CSV를 분석합니다

```
day05_calibration_measurements.csv
        ↓
35_day05_analyze_calibration.py
        ↓
OK / NG 값의 분포 확인
        ↓
SIFT Threshold 후보 생성
+
Inlier Ratio Threshold 후보 생성
        ↓
여러 조합 시험
        ↓
Calibration 12장에 적용
        ↓
TP / TN / FP / FN 비교
        ↓
오류가 적은 Rule 후보 찾기
```

그리고 두 가지 결과를 만듭니다.

```
day05_rule_candidates.csv
→ 어떤 Threshold 조합이 좋았는지

sift_inlier_distribution.png
→ OK / NG가 그래프에서 어떻게 분포하는지
```

즉 `35_day05_analyze_calibration.py` 이코드는 측정한 숫자를 분석해서 Rule 후보를 찾는 코드를 작성하는 겁니다.


파일생성:

```text
src/35_day05_analyze_calibration.py
```

이번 프로그램의 입력과 출력은 다음과 같습니다.

```
입력
reports/day05_calibration_measurements.csv

        ↓
여러 Threshold 후보 시험
        ↓
출력 1
reports/day05_rule_candidates.csv

출력 2
outputs/day05/calibration/
sift_inlier_distribution.png
```

코드 작성전 먼저 이번 실습에 필요한 라이브러리를 설치합니다.

```bash
python -m pip install pandas matplotlib
```

각 라이브러리의 역할은 다음과 같습니다.

| 라이브러리        | 이번 실습에서 하는 일                       |
| ------------ | ---------------------------------- |
| `Pandas`     | Calibration CSV를 읽고 OK/NG 측정값을 분석함 |
| `Matplotlib` | SIFT와 Inlier Ratio의 분포를 그래프로 그림    |
```
Pandas
→ CSV를 읽고 측정값을 표처럼 분석

NumPy
→ 여러 Threshold 후보값 생성

Matplotlib
→ OK와 NG의 측정값 분포를 그래프로 표현
```

다음 코드를 작성합니다.

```python
from pathlib import Path

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt


# --------------------------------------------------
# 파일 경로
# --------------------------------------------------

CSV_PATH = Path(
    "reports/day05_calibration_measurements.csv"
)

CANDIDATE_CSV = Path(
    "reports/day05_rule_candidates.csv"
)

PLOT_PATH = Path(
    "outputs/day05/calibration/sift_inlier_distribution.png"
)


# --------------------------------------------------
# 1. Threshold 후보 만들기
# --------------------------------------------------

def make_candidates(
    series,
    count=15,
    integer=False,
):

    values = pd.to_numeric(
        series,
        errors="coerce",
    ).dropna()


    if values.empty:
        return []


    low = float(
        values.min()
    )

    high = float(
        values.max()
    )


    # 모든 값이 같으면 후보도 하나만 사용
    if low == high:

        if integer:
            return [int(low)]

        return [low]


    # 최소값부터 최대값까지
    # 15개의 후보를 균등하게 생성
    candidates = np.linspace(
        low,
        high,
        count,
    )


    if integer:

        return sorted(
            set(
                int(round(value))
                for value in candidates
            )
        )


    return sorted(
        set(
            round(
                float(value),
                4,
            )
            for value in candidates
        )
    )


# --------------------------------------------------
# 2. 실제 정답과 임시 판정 비교
#
# 제조검사에서는 NG를 Positive로 정의합니다.
# --------------------------------------------------

def classify_error(
    actual,
    predicted,
):

    # 실제 NG → NG로 검출
    if (
        actual == "NG"
        and predicted == "NG"
    ):
        return "TP"


    # 실제 OK → OK로 통과
    if (
        actual == "OK"
        and predicted == "OK"
    ):
        return "TN"


    # 실제 OK → NG로 잘못 판정
    if (
        actual == "OK"
        and predicted == "NG"
    ):
        return "FP"


    # 실제 NG → OK로 잘못 통과
    if (
        actual == "NG"
        and predicted == "OK"
    ):
        return "FN"


    return "UNKNOWN"


# --------------------------------------------------
# 실행
# --------------------------------------------------

def main():

    # ==============================================
    # 1. Calibration CSV 읽기
    # ==============================================

    if not CSV_PATH.exists():

        raise FileNotFoundError(
            CSV_PATH
        )


    df = pd.read_csv(
        CSV_PATH
    )


    print("=" * 70)
    print("Calibration 데이터")
    print("=" * 70)

    print(
        "전체 이미지:",
        len(df)
    )


    print(
        df["actual"].value_counts()
    )


    # ==============================================
    # 2. OK / NG 기본 통계 확인
    # ==============================================

    numeric_columns = [
        "template_score",
        "sift_good_matches",
        "homography_inliers",
        "inlier_ratio",
    ]


    print()

    print("=" * 70)
    print("OK / NG 측정값 통계")
    print("=" * 70)


    statistics = (
        df.groupby("actual")[
            numeric_columns
        ]
        .agg(
            [
                "min",
                "mean",
                "max",
            ]
        )
    )


    print(
        statistics
    )


    # ==============================================
    # 3. Threshold 후보 만들기
    # ==============================================

    sift_candidates = make_candidates(
        df["sift_good_matches"],
        count=15,
        integer=True,
    )


    inlier_candidates = make_candidates(
        df["inlier_ratio"],
        count=15,
        integer=False,
    )


    print()

    print(
        "SIFT Threshold 후보:"
    )

    print(
        sift_candidates
    )


    print()

    print(
        "Inlier Ratio Threshold 후보:"
    )

    print(
        inlier_candidates
    )


    # ==============================================
    # 4. 모든 후보 조합 시험
    # ==============================================

    rows = []


    for sift_min in sift_candidates:

        for inlier_min in inlier_candidates:

            tp = 0
            tn = 0
            fp = 0
            fn = 0


            # Calibration 12장에
            # 현재 후보 Rule을 적용
            for _, row in df.iterrows():

                predicted = (
                    "OK"

                    if (
                        int(
                            row["homography_found"]
                        )
                        == 1

                        and int(
                            row["sift_good_matches"]
                        )
                        >= sift_min

                        and float(
                            row["inlier_ratio"]
                        )
                        >= inlier_min
                    )

                    else "NG"
                )


                error = classify_error(
                    row["actual"],
                    predicted,
                )


                if error == "TP":
                    tp += 1

                elif error == "TN":
                    tn += 1

                elif error == "FP":
                    fp += 1

                elif error == "FN":
                    fn += 1


            # --------------------------------------
            # 정확도 계산
            # --------------------------------------

            total = (
                tp
                + tn
                + fp
                + fn
            )


            accuracy = (
                (tp + tn) / total
                if total > 0
                else 0.0
            )


            # 현재 후보 결과 저장
            rows.append(
                {
                    "sift_good_matches_min":
                        sift_min,

                    "inlier_ratio_min":
                        inlier_min,

                    "accuracy":
                        round(
                            accuracy,
                            4,
                        ),

                    "tp":
                        tp,

                    "tn":
                        tn,

                    "fp":
                        fp,

                    "fn":
                        fn,

                    "total_error":
                        fp + fn,
                }
            )


    # ==============================================
    # 5. 후보 Rule을 좋은 순서로 정렬
    # ==============================================

    result_df = pd.DataFrame(
        rows
    )


    result_df = (
        result_df.sort_values(
            [
                "fn",
                "total_error",
                "fp",
                "sift_good_matches_min",
                "inlier_ratio_min",
            ],
            ascending=[
                True,
                True,
                True,
                True,
                True,
            ],
        )
        .reset_index(drop=True)
    )


    # ==============================================
    # 6. 후보 Rule CSV 저장
    # ==============================================

    CANDIDATE_CSV.parent.mkdir(
        parents=True,
        exist_ok=True,
    )


    result_df.to_csv(
        CANDIDATE_CSV,
        index=False,
        encoding="utf-8-sig",
    )


    print()

    print("=" * 70)
    print("후보 Rule 상위 15개")
    print("=" * 70)


    print(
        result_df
        .head(15)
        .to_string(
            index=False
        )
    )


    print()

    print(
        "저장:",
        CANDIDATE_CSV
    )


    # ==============================================
    # 7. 가장 위의 후보 가져오기
    # ==============================================

    best = result_df.iloc[0]


    best_sift = int(
        best["sift_good_matches_min"]
    )

    best_inlier = float(
        best["inlier_ratio_min"]
    )


    print()

    print("=" * 70)
    print("현재 가장 좋은 후보")
    print("=" * 70)

    print(
        "SIFT Good Match >=",
        best_sift
    )

    print(
        "Inlier Ratio >=",
        best_inlier
    )

    print(
        "Accuracy:",
        best["accuracy"]
    )

    print(
        "FP:",
        int(best["fp"])
    )

    print(
        "FN:",
        int(best["fn"])
    )


    # ==============================================
    # 8. 분포 그래프 만들기
    # ==============================================

    PLOT_PATH.parent.mkdir(
        parents=True,
        exist_ok=True,
    )


    plt.figure(
        figsize=(9, 6)
    )


    # OK와 NG를 다른 모양으로 표시
    for actual, marker in [
        ("OK", "o"),
        ("NG", "x"),
    ]:

        subset = df[
            df["actual"] == actual
        ]


        plt.scatter(
            subset["sift_good_matches"],
            subset["inlier_ratio"],
            label=actual,
            marker=marker,
        )


        # 각 점에 파일명 표시
        for _, row in subset.iterrows():

            plt.annotate(
                row["file"],
                (
                    row["sift_good_matches"],
                    row["inlier_ratio"],
                ),
                xytext=(4, 4),
                textcoords="offset points",
                fontsize=8,
            )


    # 선택된 후보 Threshold 표시
    plt.axvline(
        best_sift,
        linestyle="--",
        label=(
            f"SIFT >= {best_sift}"
        ),
    )


    plt.axhline(
        best_inlier,
        linestyle="--",
        label=(
            f"Inlier >= {best_inlier:.4f}"
        ),
    )


    plt.xlabel(
        "SIFT Good Matches"
    )

    plt.ylabel(
        "Homography Inlier Ratio"
    )

    plt.title(
        "Day05 Calibration Distribution"
    )


    plt.legend()

    plt.grid(
        alpha=0.2
    )


    plt.savefig(
        PLOT_PATH,
        dpi=150,
        bbox_inches="tight",
    )


    plt.close()


    print()

    print(
        "분포 이미지:",
        PLOT_PATH
    )


if __name__ == "__main__":
    main()
```

실행합니다.

```bash
python src/35_day05_analyze_calibration.py
```

결과는 다음 두 파일입니다.

```text
reports/day05_rule_candidates.csv
outputs/day05/calibration/sift_inlier_distribution.png
```

## 19.4 이 코드에서 실제로 하는 일을 먼저 봅니다

코드가 길어 보이지만 실제로 하는 일은 그림에서 본 순서와 같습니다.

```
① CSV 읽기
        ↓
② OK / NG 측정값 확인
        ↓
③ SIFT Threshold 후보 생성
        +
   Inlier Ratio Threshold 후보 생성
        ↓
④ 두 후보를 여러 가지로 조합
        ↓
⑤ Calibration 12장에 하나씩 적용
        ↓
⑥ 실제 정답과 비교
        ↓
⑦ FP / FN 등 오류 계산
        ↓
⑧ 오류가 적은 후보를 위로 정렬
        ↓
⑨ CSV와 그래프로 저장
```

### 의사코드

```text
1. 앞에서 만든 Calibration CSV를 읽는다.

2. CSV에서 OK 6장과 NG 6장의
   측정값을 확인한다.

3. SIFT Good Match의
   최소값과 최대값 사이에서
   Threshold 후보를 여러 개 만든다.

4. Inlier Ratio도
   최소값과 최대값 사이에서
   Threshold 후보를 여러 개 만든다.

5. SIFT 후보 하나와
   Inlier Ratio 후보 하나를 선택한다.

6. 두 값을 이용하여
   임시 Rule을 만든다.

   Homography 성공
   AND
   SIFT >= 후보값
   AND
   Inlier Ratio >= 후보값

   → 모두 만족하면 OK
   → 아니면 NG

7. 이 Rule을 Calibration 12장에 적용한다.

8. 프로그램의 임시 판정과
   실제 OK / NG를 비교한다.

9. 몇 장을 맞혔는지 계산한다.

10. 정상인데 NG라고 한 경우와
    불량인데 OK라고 한 경우도 계산한다.

11. 다른 Threshold 조합으로 바꾸어
    같은 작업을 다시 반복한다.

12. 모든 후보의 결과를 비교한다.

13. 불량을 OK로 통과시킨 경우가 적고
    전체 오류도 적은 후보를 위쪽에 배치한다.

14. 후보 결과를 CSV로 저장한다.

15. SIFT와 Inlier Ratio의 위치를
    그래프로 저장한다.
```

---
## 19.5 실행 결과

vscode 터미널 결과

```
======================================================================
Calibration 데이터
======================================================================
전체 이미지: 12
actual
OK    6
NG    6
Name: count, dtype: int64

======================================================================
OK / NG 측정값 통계
======================================================================
       template_score                      ... inlier_ratio                    
                  min      mean       max  ...          min      mean       max
actual                                     ...                                 
NG           0.442885  0.496935  0.536214  ...     0.773050  0.853389  0.952381
OK           0.425379  0.509216  0.556364  ...     0.810811  0.903895  0.959770

[2 rows x 12 columns]

SIFT Threshold 후보:
[74, 82, 90, 98, 105, 113, 121, 129, 137, 145, 153, 160, 168, 176, 184]

Inlier Ratio Threshold 후보:
[0.7731, 0.7864, 0.7997, 0.8131, 0.8264, 0.8397, 0.8531, 0.8664, 0.8797, 0.8931, 0.9064, 0.9198, 0.9331, 0.9464, 0.9598]

======================================================================
후보 Rule 상위 15개
======================================================================
 sift_good_matches_min  inlier_ratio_min  accuracy  tp  tn  fp  fn  total_error
                   129            0.8664    0.9167   6   5   1   0            1
                   129            0.8797    0.9167   6   5   1   0            1
                   137            0.8664    0.9167   6   5   1   0            1
                   137            0.8797    0.9167   6   5   1   0            1
                   145            0.8664    0.9167   6   5   1   0            1
                   145            0.8797    0.9167   6   5   1   0            1
                   129            0.8931    0.8333   6   4   2   0            2
                   129            0.9064    0.8333   6   4   2   0            2
                   137            0.8931    0.8333   6   4   2   0            2
                   137            0.9064    0.8333   6   4   2   0            2
                   145            0.8931    0.8333   6   4   2   0            2
                   145            0.9064    0.8333   6   4   2   0            2
                   153            0.8664    0.8333   6   4   2   0            2
                   153            0.8797    0.8333   6   4   2   0            2
                   160            0.8664    0.8333   6   4   2   0            2

저장: reports/day05_rule_candidates.csv

======================================================================
현재 가장 좋은 후보
======================================================================
SIFT Good Match >= 129
Inlier Ratio >= 0.8664
Accuracy: 0.9167
FP: 1
FN: 0

분포 이미지: outputs/day05/calibration/sift_inlier_distribution.png
```

---

## 19.6 상위 Rule 후보를 확인합니다

프로그램을 실행한 결과 여러 Threshold 조합 중 가장 위에 나온 후보는 다음과 같습니다.

```
SIFT Good Match >= 129
Inlier Ratio >= 0.8664

Accuracy = 0.9167
FP = 1
FN = 0
```

쉽게 말하면 현재 후보 Rule은 다음과 같습니다.

```
SIFT Good Match가 129 이상
        AND
Inlier Ratio가 0.8664 이상
        ↓
OK

둘 중 하나라도 기준 미달
        ↓
NG
```

이 후보를 Calibration 이미지 12장에 적용한 결과:

```
12장 중 11장
→ 올바르게 판정

1장
→ 잘못 판정

Accuracy
→ 약 91.67%

FP = 1
→ 정상 1장을 NG로 잘못 판정

FN = 0
→ 불량을 OK로 통과시킨 경우 없음
```

이라는 결과가 나왔습니다.

즉 이번 분석에서는 **Calibration 데이터를 이용하여 OK와 NG를 비교적 잘 구분하는 Baseline Rule 후보를 찾은 것**입니다.

> 아직 최종 Rule로 확정한 것은 아닙니다. 다음 단계에서 이 후보의 결과를 확인한 뒤, 실제 Test에 사용할 Baseline Rule로 결정하고 동결합니다.
---

# 20. 그래프에서 기준이 어떻게 나뉘는지 확인합니다

다음 그래프가 만들어졌습니다.

```
outputs/day05/calibration/sift_inlier_distribution.png
```

![[Pasted image 20260831163846.png]]

그래프는 Calibration 12장의 위치를 한눈에 보여줍니다.

```
X축
→ SIFT Good Match

Y축
→ Inlier Ratio

●
→ 실제 OK

×
→ 실제 NG
```

두 개의 점선은 현재 선택된 기준 후보입니다.

```
세로선
→ SIFT = 129

가로선
→ Inlier Ratio = 0.8664
```

현재 Rule에서는:

```
세로선 오른쪽
        AND
가로선 위쪽
        ↓
OK
```

로 판단합니다.

즉 두 조건을 동시에 만족해야 정상입니다.

![[Pasted image 20260831205228.png]]

---

## 20.1 왜 두 조건을 같이 사용했나요?

그래프에서 `ng_01.jpg`를 보면:

```
SIFT Good Match = 126
Inlier Ratio = 0.9524
```

입니다.

Inlier Ratio는 높지만:

```
126 < 129
```

이므로 SIFT 기준을 통과하지 못합니다.

따라서:

```
SIFT → 실패
Inlier Ratio → 통과
        ↓
NG
```

가 됩니다.

반대로 `ng_04.jpg`는:

```
SIFT Good Match = 161
Inlier Ratio = 0.8634
```

입니다.

이번에는 SIFT는 통과하지만:

```
0.8634 < 0.8664
```

이므로 Inlier Ratio에서 실패합니다.

```
SIFT → 통과
Inlier Ratio → 실패
        ↓
NG
```

즉:

```
SIFT만 사용
→ ng_04가 통과할 수 있음

Inlier Ratio만 사용
→ ng_01이 통과할 수 있음

두 조건을 함께 사용
→ 둘 다 NG로 판정
```

할 수 있었습니다.

이것이 이번에 **복합 Rule을 사용하는 이유**입니다.

---

# 21. 한 장은 왜 잘못 판정되었나요?

현재 결과는:

```
FP = 1
FN = 0
```

입니다.

잘못 판정된 이미지는 `ok_06.jpg`입니다.

실제 측정값은:

```
ok_06.jpg

SIFT Good Match = 74
Inlier Ratio = 0.8108
```

현재 기준과 비교하면:

```
SIFT
74 < 129
→ 실패

Inlier Ratio
0.8108 < 0.8664
→ 실패
```

입니다.

따라서 프로그램은:

```
실제 상태
OK

        ↓

프로그램 판정
NG
```

로 판단했습니다.

즉 **정상 이미지를 불량으로 잘못 판정한 한 건**이 발생한 것입니다.

---

## 21.1 FP와 FN을 이번 결과로 이해합니다

이번 수업에서는 `NG`를 검출해야 할 대상으로 봅니다.

|실제 상태|프로그램 판정|구분|의미|
|---|---|---|---|
|NG|NG|TP|불량을 올바르게 검출|
|OK|OK|TN|정상을 올바르게 통과|
|OK|NG|FP|정상을 불량으로 잘못 판정|
|NG|OK|FN|불량을 정상으로 잘못 통과|

이번 결과는:

```
TP = 6
→ NG 6장을 모두 검출

TN = 5
→ OK 6장 중 5장을 정상 통과

FP = 1
→ 정상 1장을 NG로 잘못 판정

FN = 0
→ 불량을 OK로 통과시킨 경우 없음
```

입니다.

특히 제조검사에서는:

```
FN

실제 NG
→ 프로그램 OK
→ 불량품 통과
```

가 중요한 오류입니다.

이번 Calibration에서는:

```
FN = 0
```

이므로 준비한 NG 6장은 모두 잡아냈습니다.

---

# 22. 현재 후보 Rule을 정리합니다

이번 Calibration 분석에서 나온 첫 번째 후보는 다음과 같습니다.

```
Homography 계산 성공

        AND

SIFT Good Match >= 129

        AND

Inlier Ratio >= 0.8664

        ↓

OK
```

하나라도 조건을 만족하지 못하면:

```
NG
```

입니다.

쉽게 읽으면:

```
① 기준 로고와
   위치 관계를 계산할 수 있는가?

        ↓ YES

② 서로 비슷하게 연결된 특징점이
   129개 이상인가?

        ↓ YES

③ 그 특징점 중
   실제로 일관되게 맞는 비율이
   약 86.64% 이상인가?

        ↓ YES

        OK
```

---

## 22.1 `129`와 `0.8664`가 유일한 정답은 아닙니다

실제 출력 결과를 보면 다음과 같은 후보들도 있습니다.

```
129 / 0.8797
137 / 0.8664
137 / 0.8797
145 / 0.8664
145 / 0.8797
```

이 후보들도 모두:

```
Accuracy = 0.9167
FP = 1
FN = 0
```

으로 같은 결과를 냈습니다.

따라서:

```
129 / 0.8664
```

가 절대적인 정답이라는 뜻은 아닙니다.

여러 후보를 시험한 결과 현재 Calibration 12장에서는 비슷하게 잘 동작하는 기준이 여러 개 존재한다는 뜻입니다.

---

## 22.2 이번 실습에서 무엇을 한 건가요?

이번 단계의 핵심을 한 번만 정리하면 다음과 같습니다.

```
Calibration 이미지 12장 측정
        ↓
측정값 CSV 생성
        ↓
SIFT와 Inlier Ratio 분포 확인
        ↓
여러 Threshold 조합 시험
        ↓
실제 OK / NG와 비교
        ↓
오류가 적은 후보 확인
        ↓
현재 후보
SIFT >= 129
Inlier Ratio >= 0.8664
```

즉 이번 실습은:

> 실제 데이터를 이용하여 여러 기준을 시험해 보고, OK와 NG를 비교적 잘 구분하는 Rule 후보를 찾는 과정

이었습니다.

![[Pasted image 20260831205444.png]]

---

# PART4 실제 검사에 사용할 Baseline Rule로 결정하고 파일로 저장하는 단계 (23~25)

---

# 23. 후보 Rule을 실제 사용할 Baseline Rule로 결정합니다

앞에서는 Calibration 이미지 12장을 이용하여 여러 판정 기준을 시험했습니다.

그 결과 상위 후보 중 하나로 다음 기준을 찾았습니다.

```
Homography 계산 성공

     AND

SIFT Good Match >= 129

     AND

Inlier Ratio >= 0.8664
```

Calibration 12장에 적용했을 때:

```
Accuracy = 91.67%

FP = 1
FN = 0
```

의 결과가 나왔습니다.

지금까지는 이것을 **Rule 후보**라고 불렀습니다.

```
Calibration
        ↓
여러 Threshold 조합 시험
        ↓
성능 비교
        ↓
상위 Rule 후보 발견
```

이제 다음 단계로 넘어가기 위해 이 후보를 **첫 번째 Baseline Rule**로 선택합니다.

```
현재 후보

SIFT >= 129
Inlier Ratio >= 0.8664
        ↓
Baseline Rule로 선택
        ↓
다음 Test에서 사용할 기준
```

여기서 `Baseline Rule`은 쉽게 말하면:

> **앞으로 Test할 때 기준으로 사용할 첫 번째 검사 규칙**입니다.

---

## 23.1 왜 Rule을 파일로 저장할까요?

기준값을 Python 코드 안에 직접 작성할 수도 있습니다.

예를 들어:

```
if sift_good_matches >= 129:
```

처럼 사용할 수 있습니다.

하지만 이렇게 하면 검사 기준이 코드 안에 섞이게 됩니다.

5일차에서는 역할을 다음처럼 나눕니다.

```
Python 코드
→ 어떻게 검사할 것인가?

JSON 설정 파일
→ 어떤 기준값으로 판정할 것인가?
```

따라서 Baseline Rule은 다음 파일에 따로 저장합니다.

```
configs/day05_logo_rules.json
```

이렇게 해두면 이후 Test 프로그램은 Python 코드에 숫자를 다시 적지 않고 **JSON에 저장된 동일한 기준을 읽어서 사용**할 수 있습니다.

```
day05_logo_rules.json

SIFT >= 129
Inlier Ratio >= 0.8664
Homography 성공

        ↓

Test 프로그램이 읽음

        ↓

항상 같은 기준으로
OK / NG 판정
```

---

## 23.2 저장할 Baseline Rule을 다시 확인합니다

JSON에 저장할 첫 번째 Baseline Rule은 다음과 같습니다.

```
조건 1
Homography 계산 성공

        AND

조건 2
SIFT Good Match >= 129

        AND

조건 3
Inlier Ratio >= 0.8664

        ↓

세 조건 모두 만족
→ OK

하나라도 만족하지 못함
→ NG
```

이 Rule에는 버전 이름을 붙입니다.

```
day05-logo-v1.0
```

`v1.0`은 **현재 Calibration 결과를 바탕으로 처음 결정한 Baseline Rule 버전**이라는 의미입니다.

현재 상태를 정리하면:

```
Calibration 완료
        ↓
Rule 후보 비교 완료
        ↓
Baseline Rule 선택 완료
        ↓
아직 파일 저장 전
```

> 이제 다음 실습에서 이 Baseline Rule을 JSON 파일에 저장하여 **Rule Freeze를 완료**합니다.

---

# 24. `36_day05_freeze_rule.py` — Baseline Rule을 JSON으로 저장하고 동결합니다

앞에서 사용할 Baseline Rule을 결정했습니다.

이제 다음 코드로 그 기준을:

```
configs/day05_logo_rules.json
```

에 저장합니다.

```
Baseline Rule 결정
        ↓
36_day05_freeze_rule.py
        ↓
JSON 저장
        ↓
day05-logo-v1.0
        ↓
Rule Freeze 완료
```

파일을 만듭니다.

```
src/36_day05_freeze_rule.py
```

다음 코드를 작성합니다.

```
src/36_day05_freeze_rule.py
```

다음 코드를 작성합니다.

```python
from pathlib import Path
import json


# --------------------------------------------------
# 저장할 JSON 파일
# --------------------------------------------------

OUTPUT_PATH = Path(
    "configs/day05_logo_rules.json"
)


# --------------------------------------------------
# Calibration 분석에서 선택한 Threshold
# --------------------------------------------------

SIFT_GOOD_MATCHES_MIN = 129

INLIER_RATIO_MIN = 0.8664


# --------------------------------------------------
# Rule 저장
# --------------------------------------------------

def main():

    rules = {

        # Rule 버전
        "baseline_version": "day05-logo-v1.0",

        # 무엇을 검사하는가?
        "inspection_target":
            "광천김 로고 존재 및 정합 검사",

        # 이번 수업에서는 NG를 Positive로 정의
        "positive_class": "NG",

        # 비교에 사용하는 기준 로고
        "reference": {
            "reference_label":
                "samples/day04/reference/reference_label.jpg"
        },

        # 실제 OK / NG 판정에 사용할 조건
        "required_rules": {

            "homography_found":
                True,

            "sift_good_matches_min":
                SIFT_GOOD_MATCHES_MIN,

            "inlier_ratio_min":
                INLIER_RATIO_MIN,
        },

        # Calibration에서 확인한 결과
        "calibration_result": {

            "accuracy":
                0.9167,

            "fp":
                1,

            "fn":
                0,
        },

        "note":
            "Calibration 결과로 결정한 Day05 Baseline Rule"
    }


    # --------------------------------------------------
    # configs 폴더가 없으면 생성
    # --------------------------------------------------

    OUTPUT_PATH.parent.mkdir(
        parents=True,
        exist_ok=True
    )


    # --------------------------------------------------
    # JSON 저장
    # --------------------------------------------------

    with OUTPUT_PATH.open(
        "w",
        encoding="utf-8"
    ) as file:

        json.dump(
            rules,
            file,
            ensure_ascii=False,
            indent=2
        )


    # --------------------------------------------------
    # 결과 확인
    # --------------------------------------------------

    print("=" * 70)

    print(
        "Rule 동결 완료"
    )

    print("=" * 70)

    print(
        "저장:",
        OUTPUT_PATH
    )

    print()

    print(
        json.dumps(
            rules,
            ensure_ascii=False,
            indent=2
        )
    )


if __name__ == "__main__":
    main()
```

---

## 24.1 코드를 실행합니다

프로젝트 루트에서 실행합니다.

```
python src/36_day05_freeze_rule.py
```

실행 결과

```
======================================================================
Rule 동결 완료
======================================================================
저장: configs/day05_logo_rules.json
```

이 출력됩니다.

그리고 다음 파일이 생성됩니다.

```
configs/
└── day05_logo_rules.json
```

### 의사코드

```
1. Rule을 저장할 JSON 파일 위치를 정한다.

2. Calibration 분석에서 선택한
   SIFT 기준값 129를 준비한다.

3. Inlier Ratio 기준값
   0.8664를 준비한다.

4. 검사 Rule의 이름을
   day05-logo-v1.0으로 정한다.

5. 어떤 Reference 이미지를 사용하는지 기록한다.

6. Homography가 성공해야 한다는 조건을 기록한다.

7. SIFT Good Match가
   129 이상이어야 한다고 기록한다.

8. Inlier Ratio가
   0.8664 이상이어야 한다고 기록한다.

9. Calibration에서 얻은
   Accuracy / FP / FN도 함께 기록한다.

10. 모든 내용을
    day05_logo_rules.json에 저장한다.
```

---

## 24.2 만들어진 JSON을 확인합니다

다음 파일을 엽니다.

```
configs/day05_logo_rules.json
```

다음과 같은 내용이 저장되어 있어야 합니다.

```json
{
  "baseline_version": "day05-logo-v1.0",
  "inspection_target": "광천김 로고 존재 및 정합 검사",
  "positive_class": "NG",
  "reference": {
    "reference_label": "samples/day04/reference/reference_label.jpg"
  },
  "required_rules": {
    "homography_found": true,
    "sift_good_matches_min": 129,
    "inlier_ratio_min": 0.8664
  },
  "calibration_result": {
    "accuracy": 0.9167,
    "fp": 1,
    "fn": 0
  },
  "note": "Calibration 결과로 결정한 Day05 Baseline Rule"
}
```

여기서 가장 중요한 부분은 다음 세 가지입니다.

```
homography_found = true

sift_good_matches_min = 129

inlier_ratio_min = 0.8664
```

앞으로 검사 프로그램은 이 값을 읽어서:

```
Homography 성공?
        AND
SIFT >= 129?
        AND
Inlier Ratio >= 0.8664?
        ↓
OK / NG
```

를 판단하게 됩니다.

---

# 25. 왜 이것을 `Freeze`라고 하나요?

지금까지는 Calibration 데이터를 보면서 기준을 찾는 과정이었습니다.

```
Calibration
        ↓
측정
        ↓
값 비교
        ↓
여러 후보 시험
        ↓
Rule 선택
```

여기까지는 기준값을 바꾸어도 됩니다.

하지만 이제:

```
day05_logo_rules.json
```

으로 기준을 저장했습니다.

이 시점부터는 이 기준을 **Baseline Rule**로 사용합니다.

```
Calibration 종료
        ↓
Rule 결정

SIFT >= 129
Inlier Ratio >= 0.8664
        ↓
JSON 저장
        ↓
day05-logo-v1.0
```

이 상태를 Rule Freeze라고 합니다.

쉽게 말하면:

> Calibration에서 기준을 정하는 작업은 끝났고, 이제부터는 같은 기준을 사용한다는 뜻입니다.

---

## 25.1 왜 기준을 고정해야 하나요?

앞으로 새로운 데이터를 검사했는데 결과가 마음에 들지 않는다고 기준을 계속 바꾸면 문제가 생깁니다.

예를 들어:

```
새 이미지 검사
        ↓
틀림
        ↓
Threshold 수정
        ↓
다시 검사
        ↓
또 틀림
        ↓
Threshold 또 수정
```

을 반복하면 우리가 만든 Rule이 실제 새로운 데이터에서도 잘 작동하는지 알 수 없습니다.

따라서:

```
Calibration
→ 기준을 만드는 데이터

Rule Freeze
→ 기준 확정

이후 새로운 데이터
→ 확정한 기준으로 검사
```

순서를 지킵니다.

---

## 25.2 지금까지의 흐름을 정리합니다

5일차의 흐름이 여기서 한 단계 완성됩니다.

![[Pasted image 20260831211034.png]]

```
광천김 Calibration 12장
        ↓
Template / SIFT / Homography 측정
        ↓
측정값 CSV
        ↓
OK / NG 분포 비교
        ↓
Threshold 후보 여러 개 시험
        ↓
후보 Rule 비교
        ↓
선택

SIFT >= 129
Inlier Ratio >= 0.8664
        ↓
JSON 저장
        ↓
Baseline Rule
day05-logo-v1.0
```

즉 이번 실습에서 한 것은:

> 앞에서 데이터로 찾아낸 판정 기준을 코드 속 임시 숫자로 두지 않고, 이후에도 동일하게 사용할 수 있도록 JSON 설정 파일로 저장한 것입니다.

---
# PART5 **Test** — 처음 보는 6장에 Rule 적용 (26~28)

---
# 26. 동결한 Rule을 처음 보는 Test 이미지에 적용합니다

Calibration 단계에서는 12장의 데이터를 이용하여 판정 기준을 만들었습니다.

현재 선택한 기준은 다음과 같습니다.

```
Homography 계산 성공

AND

SIFT Good Match >= 129

AND

Inlier Ratio >= 0.8664

        ↓

OK
```

그리고 이 기준을 다음 파일에 저장했습니다.

```
configs/day05_logo_rules.json
```

이제부터는 기준을 만드는 작업이 아니라, 만들어 놓은 기준이 새로운 이미지에서도 잘 작동하는지 확인하는 작업입니다.

이를 `Test`라고 합니다.

---

## 26.1 이번 Test에서는 무엇을 하나요?

앞에서 따로 준비해 둔 Test 이미지 6장을 사용합니다.

```
samples/day05/resized/

test_ok_01.jpg
test_ok_02.jpg
test_ok_03.jpg

test_ng_01.jpg
test_ng_02.jpg
test_ng_03.jpg
```

이 6장은 Calibration 기준을 결정할 때 사용하지 않았습니다.

따라서 다음 순서로 검사합니다.

```
동결한 Rule JSON
        +
Test 이미지 한 장
        ↓
4일차와 같은 방법으로 측정
        ↓
SIFT Good Match
Inlier Ratio
Homography Found
        ↓
JSON의 기준값과 비교
        ↓
Predicted
OK / NG
        ↓
사람이 알고 있는 실제 정답과 비교
```

예를 들어:

```
test_ok_01.jpg

실제 정답
→ OK

프로그램 판정
→ ?

두 결과를 비교
```

합니다.

> Test의 목적은 기준을 다시 찾는 것이 아니라, Calibration에서 정한 기준이 새로운 데이터에서도 잘 작동하는지 확인하는 것입니다.

---

## 26.2 Test에서는 기준을 바꾸지 않습니다

Test를 하면서 결과가 마음에 들지 않아도 다음 값은 그대로 사용합니다.

```
SIFT >= 129

Inlier Ratio >= 0.8664
```

즉:

```
Test 결과가 틀림
        ↓
Threshold 바로 수정 X

먼저
왜 틀렸는지 분석
```

합니다.

이렇게 해야 현재 Rule의 실제 성능을 알 수 있습니다.

---

## 26.3 `37_day05_test.py` — Test 데이터 6장을 검사합니다

파일을 만듭니다.

```
src/37_day05_test.py
```

다음 코드를 작성합니다.

```python
from pathlib import Path
import csv
import json

import cv2
import numpy as np

from day05_logo_features import (
    prepare_reference,
    measure_target,
)


# --------------------------------------------------
# 경로
# --------------------------------------------------

RULE_PATH = Path(
    "configs/day05_logo_rules.json"
)

DATA_DIR = Path(
    "samples/day05/resized"
)

OUTPUT_DIR = Path(
    "outputs/day05/test"
)

OUTPUT_CSV = Path(
    "reports/day05_test_results.csv"
)


# --------------------------------------------------
# 1. Test 이미지 목록 만들기
# --------------------------------------------------

def find_test_images():

    image_infos = []


    # 실제 정상 Test 이미지
    for image_path in sorted(
        DATA_DIR.glob("test_ok_*.jpg")
    ):

        image_infos.append(
            (
                image_path,
                "OK"
            )
        )


    # 실제 불량 Test 이미지
    for image_path in sorted(
        DATA_DIR.glob("test_ng_*.jpg")
    ):

        image_infos.append(
            (
                image_path,
                "NG"
            )
        )


    return image_infos


# --------------------------------------------------
# 2. 측정값에 Rule 적용
# --------------------------------------------------

def judge_measurement(
    measurement,
    rules,
):

    reasons = []

    required = rules[
        "required_rules"
    ]


    # Homography 성공 여부
    if (
        required["homography_found"]
        and measurement["homography_found"] != 1
    ):

        reasons.append(
            "HOMOGRAPHY_NOT_FOUND"
        )


    # SIFT Good Match 검사
    if (
        measurement["sift_good_matches"]
        < required["sift_good_matches_min"]
    ):

        reasons.append(
            "LOW_SIFT_MATCH"
        )


    # Inlier Ratio 검사
    if (
        measurement["inlier_ratio"]
        < required["inlier_ratio_min"]
    ):

        reasons.append(
            "LOW_INLIER_RATIO"
        )


    # 실패 이유가 하나도 없으면 OK
    if not reasons:

        predicted = "OK"

    else:

        predicted = "NG"


    return predicted, reasons


# --------------------------------------------------
# 3. 실제 정답과 프로그램 판정 비교
#
# 이번 수업에서는 NG를 Positive로 둡니다.
# --------------------------------------------------

def classify_error(
    actual,
    predicted,
):

    if (
        actual == "NG"
        and predicted == "NG"
    ):
        return "TP"


    if (
        actual == "OK"
        and predicted == "OK"
    ):
        return "TN"


    if (
        actual == "OK"
        and predicted == "NG"
    ):
        return "FP"


    if (
        actual == "NG"
        and predicted == "OK"
    ):
        return "FN"


    return "UNKNOWN"


# --------------------------------------------------
# 4. 결과 이미지 만들기
# --------------------------------------------------

def annotate_result(
    measurement,
    actual,
    predicted,
    reasons,
):

    image = (
        measurement["target_image"]
        .copy()
    )


    polygon = measurement.get(
        "polygon"
    )


    # Homography로 찾은 로고 영역 표시
    if polygon is not None:

        points = np.int32(
            polygon
        ).reshape(
            -1,
            1,
            2
        )


        # OK와 NG의 선 색상을 구분
        if predicted == "OK":

            line_color = (
                0,
                255,
                0
            )

        else:

            line_color = (
                0,
                0,
                255
            )


        cv2.polylines(
            image,
            [points],
            True,
            line_color,
            3
        )


    reason_text = (
        "+".join(reasons)
        if reasons
        else "PASS"
    )


    lines = [

        f"Actual: {actual}",

        f"Predicted: {predicted}",

        (
            "SIFT Good: "
            f"{measurement['sift_good_matches']}"
        ),

        (
            "Inlier Ratio: "
            f"{measurement['inlier_ratio']:.4f}"
        ),

        (
            "Template: "
            f"{measurement['template_score']:.4f}"
        ),

        f"Reason: {reason_text}",
    ]


    y = 40


    for line in lines:

        cv2.putText(
            image,
            line,
            (30, y),
            cv2.FONT_HERSHEY_SIMPLEX,
            0.8,
            (255, 255, 255),
            2,
            cv2.LINE_AA
        )

        y += 35


    return image


# --------------------------------------------------
# 실행
# --------------------------------------------------

def main():

    # ==============================================
    # 1. 동결한 Rule 읽기
    # ==============================================

    if not RULE_PATH.exists():

        raise FileNotFoundError(
            "Rule JSON이 없습니다. "
            "먼저 36_day05_freeze_rule.py를 실행하세요."
        )


    with RULE_PATH.open(
        "r",
        encoding="utf-8"
    ) as file:

        rules = json.load(
            file
        )


    print("=" * 70)

    print(
        "Baseline Rule:",
        rules["baseline_version"]
    )

    print(
        "SIFT >=",
        rules[
            "required_rules"
        ][
            "sift_good_matches_min"
        ]
    )

    print(
        "Inlier Ratio >=",
        rules[
            "required_rules"
        ][
            "inlier_ratio_min"
        ]
    )


    # ==============================================
    # 2. Reference 준비
    # ==============================================

    reference_path = Path(
        rules[
            "reference"
        ][
            "reference_label"
        ]
    )


    reference_data = prepare_reference(
        reference_path
    )


    # ==============================================
    # 3. Test 이미지 확인
    # ==============================================

    image_infos = (
        find_test_images()
    )


    print(
        "Test Images:",
        len(image_infos)
    )


    if len(image_infos) != 6:

        print(
            "주의: Test 이미지는 "
            "OK 3장 + NG 3장 = 6장을 예상합니다."
        )


    OUTPUT_DIR.mkdir(
        parents=True,
        exist_ok=True
    )

    OUTPUT_CSV.parent.mkdir(
        parents=True,
        exist_ok=True
    )


    rows = []


    # ==============================================
    # 4. Test 6장 검사
    # ==============================================

    for image_path, actual in image_infos:

        # ------------------------------------------
        # 측정
        # ------------------------------------------

        measurement = measure_target(
            image_path,
            reference_data
        )


        # ------------------------------------------
        # 동결 Rule로 판정
        # ------------------------------------------

        predicted, reasons = (
            judge_measurement(
                measurement,
                rules
            )
        )


        # ------------------------------------------
        # 실제 정답과 비교
        # ------------------------------------------

        error_type = classify_error(
            actual,
            predicted
        )


        # ------------------------------------------
        # 결과 이미지 생성
        # ------------------------------------------

        result_image = annotate_result(
            measurement,
            actual,
            predicted,
            reasons
        )


        output_image = (
            OUTPUT_DIR
            / (
                f"{image_path.stem}"
                "_result.jpg"
            )
        )


        cv2.imwrite(
            str(output_image),
            result_image
        )


        # ------------------------------------------
        # CSV 결과
        # ------------------------------------------

        row = {

            "file":
                image_path.name,

            "actual":
                actual,

            "predicted":
                predicted,

            "error_type":
                error_type,

            "ng_reason":
                (
                    ";".join(reasons)
                    if reasons
                    else "PASS"
                ),

            "template_score":
                round(
                    measurement[
                        "template_score"
                    ],
                    6
                ),

            "sift_good_matches":
                measurement[
                    "sift_good_matches"
                ],

            "homography_found":
                measurement[
                    "homography_found"
                ],

            "homography_inliers":
                measurement[
                    "homography_inliers"
                ],

            "inlier_ratio":
                round(
                    measurement[
                        "inlier_ratio"
                    ],
                    6
                ),

            "result_image":
                str(output_image),
        }


        rows.append(
            row
        )


        print(
            row
        )


    # ==============================================
    # 5. CSV 저장
    # ==============================================

    fieldnames = [
        "file",
        "actual",
        "predicted",
        "error_type",
        "ng_reason",
        "template_score",
        "sift_good_matches",
        "homography_found",
        "homography_inliers",
        "inlier_ratio",
        "result_image",
    ]


    with OUTPUT_CSV.open(
        "w",
        encoding="utf-8-sig",
        newline=""
    ) as file:

        writer = csv.DictWriter(
            file,
            fieldnames=fieldnames
        )

        writer.writeheader()

        writer.writerows(
            rows
        )


    # ==============================================
    # 6. Test 결과 요약
    # ==============================================

    tp = sum(
        row["error_type"] == "TP"
        for row in rows
    )

    tn = sum(
        row["error_type"] == "TN"
        for row in rows
    )

    fp = sum(
        row["error_type"] == "FP"
        for row in rows
    )

    fn = sum(
        row["error_type"] == "FN"
        for row in rows
    )


    total = len(rows)


    accuracy = (
        (tp + tn) / total
        if total
        else 0.0
    )


    print()

    print("=" * 70)

    print(
        "Test 완료"
    )

    print("=" * 70)

    print(
        "전체:",
        total
    )

    print(
        "Accuracy:",
        round(
            accuracy,
            4
        )
    )

    print(
        "TP:",
        tp
    )

    print(
        "TN:",
        tn
    )

    print(
        "FP:",
        fp
    )

    print(
        "FN:",
        fn
    )

    print(
        "CSV:",
        OUTPUT_CSV
    )

    print(
        "결과 이미지:",
        OUTPUT_DIR
    )


if __name__ == "__main__":
    main()
```

---

## 26.4 의사코드로 읽어봅니다

```
1. 앞에서 저장한 Rule JSON을 읽는다.

2. JSON에서 다음 기준을 가져온다.

   SIFT >= 129
   Inlier Ratio >= 0.8664
   Homography 성공

3. JSON에 저장된 Reference 경로를 읽는다.

4. Reference의 SIFT 특징점을 준비한다.

5. test_ok_로 시작하는 이미지 3장을 찾는다.

6. 이 이미지의 실제 정답을 OK라고 기록한다.

7. test_ng_로 시작하는 이미지 3장을 찾는다.

8. 이 이미지의 실제 정답을 NG라고 기록한다.

9. Test 이미지 한 장을 측정한다.

10. SIFT Good Match를 구한다.

11. Homography를 계산한다.

12. Inlier Ratio를 구한다.

13. 동결한 Rule과 측정값을 비교한다.

14. 조건을 모두 만족하면 OK,
    하나라도 실패하면 NG로 판정한다.

15. 프로그램 판정과 실제 정답을 비교한다.

16. TP / TN / FP / FN을 기록한다.

17. NG라면 어떤 조건에서 실패했는지 기록한다.

18. 검출 위치와 측정값을 결과 이미지에 표시한다.

19. 같은 작업을 Test 6장에 반복한다.

20. 모든 결과를 CSV로 저장한다.

21. 최종 Accuracy와
    TP / TN / FP / FN을 출력한다.
```

---

## 26.5 실행합니다

```
python src/37_day05_test.py
```

이번에는 결과가 **숫자만 저장되는 것이 아닙니다.**

다음 두 종류의 결과물이 만들어집니다.

```
reports/day05_test_results.csv
```

```
outputs/day05/test/

test_ok_01_result.jpg
test_ok_02_result.jpg
test_ok_03_result.jpg

test_ng_01_result.jpg
test_ng_02_result.jpg
test_ng_03_result.jpg
```

![[{5341ACDB-44B5-4519-B2B9-2152F6EF5F92}.png]]

---

# 27. 실제 Test 결과를 확인합니다

앞에서 동결한 Baseline Rule을 처음 보는 Test 이미지 6장에 적용했습니다.

사용한 기준은 다음과 같습니다.

```
Baseline Rule: day05-logo-v1.0
SIFT >= 129
Inlier Ratio >= 0.8664
Test Images: 6
{'file': 'test_ok_01.jpg', 'actual': 'OK', 'predicted': 'NG', 'error_type': 'FP', 'ng_reason': 'LOW_INLIER_RATIO', 'template_score': 0.581604, 'sift_good_matches': 169, 'homography_found': 1, 'homography_inliers': 132, 'inlier_ratio': 0.781065, 'result_image': 'outputs/day05/test/test_ok_01_result.jpg'}
{'file': 'test_ok_02.jpg', 'actual': 'OK', 'predicted': 'NG', 'error_type': 'FP', 'ng_reason': 'LOW_INLIER_RATIO', 'template_score': 0.45615, 'sift_good_matches': 156, 'homography_found': 1, 'homography_inliers': 131, 'inlier_ratio': 0.839744, 'result_image': 'outputs/day05/test/test_ok_02_result.jpg'}
{'file': 'test_ok_03.jpg', 'actual': 'OK', 'predicted': 'NG', 'error_type': 'FP', 'ng_reason': 'LOW_INLIER_RATIO', 'template_score': 0.500078, 'sift_good_matches': 204, 'homography_found': 1, 'homography_inliers': 174, 'inlier_ratio': 0.852941, 'result_image': 'outputs/day05/test/test_ok_03_result.jpg'}
{'file': 'test_ng_01.jpg', 'actual': 'NG', 'predicted': 'NG', 'error_type': 'TP', 'ng_reason': 'LOW_SIFT_MATCH;LOW_INLIER_RATIO', 'template_score': 0.392391, 'sift_good_matches': 58, 'homography_found': 1, 'homography_inliers': 43, 'inlier_ratio': 0.741379, 'result_image': 'outputs/day05/test/test_ng_01_result.jpg'}
{'file': 'test_ng_02.jpg', 'actual': 'NG', 'predicted': 'NG', 'error_type': 'TP', 'ng_reason': 'LOW_SIFT_MATCH;LOW_INLIER_RATIO', 'template_score': 0.515177, 'sift_good_matches': 111, 'homography_found': 1, 'homography_inliers': 85, 'inlier_ratio': 0.765766, 'result_image': 'outputs/day05/test/test_ng_02_result.jpg'}
{'file': 'test_ng_03.jpg', 'actual': 'NG', 'predicted': 'NG', 'error_type': 'TP', 'ng_reason': 'LOW_SIFT_MATCH', 'template_score': 0.536272, 'sift_good_matches': 105, 'homography_found': 1, 'homography_inliers': 96, 'inlier_ratio': 0.914286, 'result_image': 'outputs/day05/test/test_ng_03_result.jpg'}

======================================================================
Test 완료
======================================================================
전체: 6
Accuracy: 0.5
TP: 3
TN: 0
FP: 3
FN: 0
CSV: reports/day05_test_results.csv
결과 이미지: outputs/day05/test
```


```
Baseline Rule
day05-logo-v1.0

Homography 계산 성공

AND

SIFT Good Match >= 129

AND

Inlier Ratio >= 0.8664
```

실행 결과는 다음과 같습니다.

| 파일               | 실제  | 판정  | SIFT | Inlier Ratio | 결과  |
| ---------------- | --- | --- | ---: | -----------: | --- |
| `test_ok_01.jpg` | OK  | NG  |  169 |       0.7811 | FP  |
| `test_ok_02.jpg` | OK  | NG  |  156 |       0.8397 | FP  |
| `test_ok_03.jpg` | OK  | NG  |  204 |       0.8529 | FP  |
| `test_ng_01.jpg` | NG  | NG  |   58 |       0.7414 | TP  |
| `test_ng_02.jpg` | NG  | NG  |  111 |       0.7658 | TP  |
| `test_ng_03.jpg` | NG  | NG  |  105 |       0.9143 | TP  |

전체 결과는:

```
Test 이미지 = 6장

TP = 3
TN = 0
FP = 3
FN = 0

Accuracy = 0.5
         = 50%
```

입니다.

쉽게 말하면:

```
불량 3장
→ 모두 불량으로 검출
→ 성공

정상 3장
→ 모두 불량으로 판정
→ 실패
```

따라서 현재 Baseline은 불량을 놓치지는 않았지만 정상 제품까지 너무 많이 NG로 판정하는 상태입니다.

---

## 27.1 정상 Test 이미지 3장은 왜 모두 NG가 되었나요?

정상 이미지 세 장의 결과를 먼저 봅니다.

```
test_ok_01
SIFT = 169        → 통과
Ratio = 0.7811    → 실패

test_ok_02
SIFT = 156        → 통과
Ratio = 0.8397    → 실패

test_ok_03
SIFT = 204        → 통과
Ratio = 0.8529    → 실패
```

현재 기준은:

```
SIFT >= 129
AND
Inlier Ratio >= 0.8664
```

입니다.

세 이미지 모두 SIFT는 충분했습니다.

하지만:

```
0.7811 < 0.8664

0.8397 < 0.8664

0.8529 < 0.8664
```

로 **Inlier Ratio 기준을 통과하지 못했습니다.**

따라서:

```
실제 상태
OK

SIFT
통과

Homography
성공

Inlier Ratio
기준 미달
        ↓
Predicted = NG
        ↓
FP
```

가 되었습니다.

> 이번 Test에서는 정상 이미지의 주요 실패 원인이 `LOW_INLIER_RATIO`로 동일하게 나타났습니다.

---

## 27.2 불량 Test 이미지 3장은 어떻게 판정되었나요?

불량 이미지 세 장은 모두 올바르게 NG로 판정되었습니다.

### `test_ng_01.jpg`

```
SIFT = 58
→ 129보다 낮음

Inlier Ratio = 0.7414
→ 0.8664보다 낮음

Reason
LOW_SIFT_MATCH
+
LOW_INLIER_RATIO

→ NG
→ TP
```

### `test_ng_02.jpg`

```
SIFT = 111
→ 기준 미달

Inlier Ratio = 0.7658
→ 기준 미달

Reason
LOW_SIFT_MATCH
+
LOW_INLIER_RATIO

→ NG
→ TP
```

### `test_ng_03.jpg`

이 이미지는 조금 다릅니다.

```
SIFT = 105
→ 기준 미달

Inlier Ratio = 0.9143
→ 기준 통과
```

즉 Inlier Ratio는 높았지만 SIFT가 부족했기 때문에:

```
SIFT
실패

Inlier Ratio
통과
        ↓
NG
        ↓
TP
```

가 되었습니다.

이 결과에서도 **두 조건을 함께 사용하는 이유**를 확인할 수 있습니다.

---

## 27.3 결과 이미지는 무엇을 확인해야 하나요?

생성된 결과 이미지도 함께 확인합니다.

```
outputs/day05/test/
```

결과 이미지에는:

```
Actual
Predicted
SIFT Good Match
Inlier Ratio
Template Score
Reason
```

이 표시됩니다.

또 Homography가 계산되었다면 프로그램이 찾은 로고 영역도 사각형으로 표시됩니다.

이번 6장에서는 모두:

```
homography_found = 1
```

이었습니다.

즉 **로고 위치관계 자체는 모두 계산할 수 있었지만, 최종 OK / NG 판정에서는 SIFT와 Inlier Ratio 기준 때문에 결과가 달라진 것**입니다.

---

## 27.4 이번 Test에서 발견한 핵심

Calibration에서는 현재 Rule이:

```
12장 중 11장 정답
Accuracy = 91.67%
```

이었습니다.

하지만 처음 보는 Test에서는:

```
6장 중 3장 정답
Accuracy = 50%
```

으로 크게 떨어졌습니다.

특히:

```
Calibration에서는 잘 동작

        ↓

처음 보는 정상 Test 이미지

        ↓

Inlier Ratio가 예상보다 낮아짐

        ↓

정상 3장을 모두 NG로 판정
```

했습니다.

> 이것이 Test를 따로 하는 이유입니다. Calibration에서 성능이 좋았다고 해서 새로운 이미지에서도 같은 성능이 보장되는 것은 아닙니다.

---

# 28. NG Reason과 실제 Failure Cause를 구분합니다

이번 Test 결과에는 다음과 같은 `NG Reason`이 기록되었습니다.

```
LOW_SIFT_MATCH

LOW_INLIER_RATIO
```

이것은 **프로그램이 어떤 조건 때문에 NG라고 판정했는지 알려주는 값**입니다.

예를 들어 정상 이미지인 `test_ok_01.jpg`는:

```
Actual
→ OK

Predicted
→ NG

Reason
→ LOW_INLIER_RATIO
```

입니다.

이 뜻은 정확히:

```
Inlier Ratio = 0.7811
        <
Threshold = 0.8664
```

였다는 것입니다.

하지만 이것이 실제 원인까지 설명하는 것은 아닙니다.

---

## 28.1 이번 실제 결과에서 Failure Cause를 생각해 봅니다

정상 Test 이미지 세 장을 보면 모두:

```
Homography
→ 성공

SIFT
→ 129 이상

하지만

Inlier Ratio
→ 기준보다 낮음
```

이라는 공통적인 특징이 있습니다.

즉 프로그램 수준의 **NG Reason**은 명확합니다.

```
LOW_INLIER_RATIO
```

하지만 실제 **Failure Cause**는 이미지를 함께 보면서 확인해야 합니다.

예를 들어:

```
촬영 각도가 Calibration과 달랐는가?

제품 크기나 거리가 달라졌는가?

포장지의 구김이나 반사가 달라졌는가?

로고 주변 모습이 Calibration 이미지와 달랐는가?
```

등을 확인할 수 있습니다.

따라서 다음처럼 구분합니다.

|구분|이번 결과에서의 의미|
|---|---|
|**NG Reason**|Inlier Ratio가 `0.8664`보다 낮았음|
|**Failure Cause**|왜 정상 이미지의 Inlier Ratio가 낮아졌는지 실제 이미지를 보고 분석|

> `LOW_INLIER_RATIO`는 **측정 결과**이고, 촬영 각도·거리·반사 등의 차이는 그 결과가 발생한 **가능한 실제 원인**입니다.

---

## 28.2 이번 Test 결과가 알려주는 것은 무엇인가요?

이번 결과를 가장 간단히 정리하면:

```
NG 3장
→ 모두 잡음
→ FN = 0

하지만

OK 3장
→ 모두 NG로 판정
→ FP = 3
```

입니다.

따라서 현재 Baseline은:

```
불량을 놓치지 않는 데는 강함

하지만

정상 이미지에 대한 기준이 너무 엄격함
```

이라는 특성을 보였습니다.

특히 이번 Test에서는:

```
SIFT 기준
129
→ 정상 3장 모두 통과

Inlier Ratio 기준
0.8664
→ 정상 3장 모두 실패
```

했으므로 **현재 Rule에서 정상 Test 이미지의 판정을 가로막은 핵심 조건은 Inlier Ratio Threshold**였다는 것을 확인할 수 있습니다.

하지만 여기서 바로:

```
0.8664를 낮추자
```

라고 수정하지 않습니다.

다음 순서를 지킵니다.

```
Test 결과 확인
        ↓
FP = 3 확인
        ↓
공통 NG Reason 확인
LOW_INLIER_RATIO
        ↓
실제 이미지 확인
        ↓
왜 정상 이미지의 Ratio가
낮아졌는지 원인 분석
        ↓
현재 Baseline의 한계 기록
```

> **이번 Test는 실패한 실습이 아닙니다. Calibration에서 만든 Rule이 새로운 정상 이미지에는 충분히 일반화되지 않았다는 중요한 사실을 발견한 실습입니다.**

---
# PART6 **Evaluation** — 전체 성능 + FP/FN + 실패 원인 확인 (29~31)

---

# 29. Test 전체 성능을 계산합니다

앞의 Test 실습에서는 이미지마다 다음과 같은 결과를 만들었습니다.

```
test_ok_01.jpg
→ 실제 OK
→ 프로그램 판정 OK 또는 NG

test_ng_01.jpg
→ 실제 NG
→ 프로그램 판정 OK 또는 NG
```

그리고 모든 결과가 다음 파일에 저장되었습니다.

```
reports/day05_test_results.csv
```

이번에는 이 CSV를 읽어서 **Test 6장 전체에서 우리 Rule이 얼마나 잘 작동했는지 계산**합니다.

```
개별 이미지 판정 결과
        ↓
전체 결과 집계
        ↓
TP / TN / FP / FN
        ↓
Accuracy
Precision
Recall
```

> 26번이 **이미지 한 장씩 검사하는 실습**이었다면, 29번은 **그 검사 결과 전체를 평가하는 실습**입니다.

---

## 29.1 왜 Accuracy만 계산하지 않나요?

예를 들어 6장 중 5장을 맞혔다면:

```
Accuracy
= 5 / 6
≈ 83.3%
```

입니다.

하지만 **어떤 한 장을 틀렸는가**도 중요합니다.

예를 들어:

```
정상 제품을 NG라고 판단
→ FP

불량 제품을 OK라고 통과
→ FN
```

은 의미가 다릅니다.

특히 이번 제조검사에서는 불량을 찾아내는 것을 중요하게 보기 때문에 **NG를 Positive**로 정합니다.

|실제|프로그램 판정|구분|의미|
|---|---|---|---|
|NG|NG|TP|불량을 올바르게 검출|
|OK|OK|TN|정상을 올바르게 통과|
|OK|NG|FP|정상을 불량으로 잘못 판정|
|NG|OK|FN|불량을 정상으로 잘못 통과|

따라서 이번에는 다음 값을 함께 확인합니다.

```
Accuracy
→ 전체적으로 얼마나 맞혔는가?

NG Precision
→ NG라고 판정한 것 중
   실제 NG는 얼마나 되는가?

NG Recall
→ 실제 NG 중
   얼마나 놓치지 않고 찾아냈는가?

FP
→ 정상 과검출은 몇 건인가?

FN
→ 불량 미검출은 몇 건인가?
```

---

## 29.2 `38_day05_evaluate.py` — Test 결과를 평가합니다

파일을 만듭니다.

```
src/38_day05_evaluate.py
```

다음 코드를 작성합니다.

```python
from pathlib import Path
import shutil
import json

import pandas as pd


# --------------------------------------------------
# 입력 파일
# --------------------------------------------------

RESULT_CSV = Path(
    "reports/day05_test_results.csv"
)

RULE_PATH = Path(
    "configs/day05_logo_rules.json"
)


# --------------------------------------------------
# 출력 파일
# --------------------------------------------------

REPORT_PATH = Path(
    "reports/day05_test_metrics.md"
)

FP_DIR = Path(
    "outputs/day05/fp"
)

FN_DIR = Path(
    "outputs/day05/fn"
)


# --------------------------------------------------
# 0으로 나누는 오류를 막는 함수
# --------------------------------------------------

def safe_divide(a, b):

    if b == 0:
        return 0.0

    return a / b


# --------------------------------------------------
# 실행
# --------------------------------------------------

def main():

    # ==============================================
    # 1. Test 결과 CSV 확인
    # ==============================================

    if not RESULT_CSV.exists():

        raise FileNotFoundError(
            "Test 결과 CSV가 없습니다. "
            "먼저 37_day05_test.py를 실행하세요."
        )


    df = pd.read_csv(
        RESULT_CSV
    )


    required_columns = {
        "file",
        "actual",
        "predicted",
        "error_type",
        "result_image",
    }


    missing = (
        required_columns
        - set(df.columns)
    )


    if missing:

        raise RuntimeError(
            f"필요한 컬럼이 없습니다: {missing}"
        )


    # ==============================================
    # 2. 사용한 Rule 버전 확인
    # ==============================================

    baseline_version = "UNKNOWN"


    if RULE_PATH.exists():

        with RULE_PATH.open(
            "r",
            encoding="utf-8"
        ) as file:

            rules = json.load(
                file
            )


        baseline_version = rules.get(
            "baseline_version",
            "UNKNOWN"
        )


    # ==============================================
    # 3. TP / TN / FP / FN 집계
    # ==============================================

    counts = (
        df["error_type"]
        .value_counts()
        .to_dict()
    )


    tp = int(
        counts.get("TP", 0)
    )

    tn = int(
        counts.get("TN", 0)
    )

    fp = int(
        counts.get("FP", 0)
    )

    fn = int(
        counts.get("FN", 0)
    )


    total = (
        tp
        + tn
        + fp
        + fn
    )


    # ==============================================
    # 4. 성능 지표 계산
    # ==============================================

    accuracy = safe_divide(
        tp + tn,
        total
    )


    # NG를 Positive로 정의합니다.
    precision_ng = safe_divide(
        tp,
        tp + fp
    )


    recall_ng = safe_divide(
        tp,
        tp + fn
    )


    # ==============================================
    # 5. 기존 FP / FN 폴더 초기화
    #
    # 이전 실행 결과가 섞이지 않도록 합니다.
    # ==============================================

    for folder in [
        FP_DIR,
        FN_DIR,
    ]:

        if folder.exists():

            shutil.rmtree(
                folder
            )


        folder.mkdir(
            parents=True,
            exist_ok=True
        )


    # ==============================================
    # 6. FP / FN 결과 이미지를 따로 복사
    # ==============================================

    for _, row in df.iterrows():

        error_type = row[
            "error_type"
        ]

        source = Path(
            row["result_image"]
        )


        if not source.exists():
            continue


        if error_type == "FP":

            shutil.copy2(
                source,
                FP_DIR / source.name
            )


        elif error_type == "FN":

            shutil.copy2(
                source,
                FN_DIR / source.name
            )


    # ==============================================
    # 7. 평가 보고서 만들기
    # ==============================================

    REPORT_PATH.parent.mkdir(
        parents=True,
        exist_ok=True
    )


    report = f"""# Day05 Test Metrics

## Baseline

- Rule Version: {baseline_version}
- Positive Class: NG
- Test Images: {total}

## Confusion Matrix

| Actual / Predicted | NG | OK |
|---|---:|---:|
| NG | TP = {tp} | FN = {fn} |
| OK | FP = {fp} | TN = {tn} |

## Metrics

- Accuracy: {accuracy:.4f}
- NG Precision: {precision_ng:.4f}
- NG Recall: {recall_ng:.4f}
- FP: {fp}
- FN: {fn}

## Interpretation

- FP: 실제 OK인데 NG로 잘못 판정
- FN: 실제 NG인데 OK로 잘못 통과
"""


    REPORT_PATH.write_text(
        report,
        encoding="utf-8"
    )


    # ==============================================
    # 8. 터미널 출력
    # ==============================================

    print("=" * 70)

    print(
        "Day05 Test Evaluation"
    )

    print("=" * 70)


    print(
        "Rule:",
        baseline_version
    )

    print(
        "Test Images:",
        total
    )

    print()

    print(
        "TP:",
        tp
    )

    print(
        "TN:",
        tn
    )

    print(
        "FP:",
        fp
    )

    print(
        "FN:",
        fn
    )

    print()

    print(
        "Accuracy:",
        round(
            accuracy,
            4
        )
    )

    print(
        "NG Precision:",
        round(
            precision_ng,
            4
        )
    )

    print(
        "NG Recall:",
        round(
            recall_ng,
            4
        )
    )

    print()

    print(
        "평가 보고서:",
        REPORT_PATH
    )

    print(
        "FP 이미지:",
        FP_DIR
    )

    print(
        "FN 이미지:",
        FN_DIR
    )


if __name__ == "__main__":
    main()
```

실행합니다

```
python src/38_day05_evaluate.py
```

### 의사코드

```
1. 앞에서 만든 Test 결과 CSV를 읽는다.

2. 사용했던 Baseline Rule 버전을 확인한다.

3. Test 결과에서
   TP / TN / FP / FN의 개수를 센다.

4. 전체 이미지 중
   몇 장을 올바르게 판정했는지
   Accuracy를 계산한다.

5. NG라고 판정한 것 중
   실제 NG가 얼마나 되는지
   NG Precision을 계산한다.

6. 실제 NG 중
   얼마나 놓치지 않고 잡았는지
   NG Recall을 계산한다.

7. FP 이미지가 있다면
   outputs/day05/fp에 따로 복사한다.

8. FN 이미지가 있다면
   outputs/day05/fn에 따로 복사한다.

9. 전체 평가 결과를
   day05_test_metrics.md에 저장한다.
```

---

## 29.3 실행 결과를 확인합니다

실제 실행 결과는 다음과 같습니다.

```
======================================================================
Day05 Test Evaluation
======================================================================
Rule: day05-logo-v1.0
Test Images: 6

TP: 3
TN: 0
FP: 3
FN: 0

Accuracy: 0.5
NG Precision: 0.5
NG Recall: 1.0

평가 보고서: reports/day05_test_metrics.md
FP 이미지: outputs/day05/fp
FN 이미지: outputs/day05/fn
```

평가 결과는 다음 파일에도 저장됩니다.

```
reports/day05_test_metrics.md
```

그리고 오판정 이미지는 오류 종류에 따라 별도 폴더로 모입니다.

```
outputs/day05/fp/
→ FP 이미지 저장

outputs/day05/fn/
→ FN 이미지 저장
```

이번 실제 결과에서는:

```
FP = 3
FN = 0
```

이므로:

```
outputs/day05/fp/
├── test_ok_01_result.jpg
├── test_ok_02_result.jpg
└── test_ok_03_result.jpg

outputs/day05/fn/
└── 비어 있음
```

이 맞습니다.

즉 전체 결과 구조는 다음과 같습니다.

```
Test 결과 CSV
        ↓
38_day05_evaluate.py
        ↓
├─ 성능 보고서
│  reports/day05_test_metrics.md
│
├─ FP 이미지
│  outputs/day05/fp/
│  → 3장
│
└─ FN 이미지
   outputs/day05/fn/
   → 0장
```

> `fn` 폴더가 비어 있는 것은 오류가 아니라, 이번 Test에서 불량을 OK로 잘못 통과시킨 경우가 없었다는 뜻입니다.

---

# 30. 실제 평가 결과를 어떻게 읽나요?

이번 Test의 실제 성능은 다음과 같습니다.

```
TP = 3
TN = 0
FP = 3
FN = 0

Accuracy = 0.5000
NG Precision = 0.5000
NG Recall = 1.0000
```

하나씩 읽어보겠습니다.

### Accuracy = 0.5

```
6장 중
올바르게 판정한 이미지 = 3장

3 / 6
= 0.5
= 50%
```

즉 전체 Test 이미지의 **절반을 올바르게 판정했습니다.**

---

### NG Precision = 0.5

이번 프로그램은 Test 6장을 **모두 NG라고 판정**했습니다.

그중 실제 NG는 3장이었습니다.

```
프로그램이 NG라고 판정
→ 6장

그중 실제 NG
→ 3장

3 / 6
= 0.5
= 50%
```

즉:

> NG라고 판정한 이미지 중 절반만 실제 NG였습니다.

정상 제품도 3장이나 NG라고 판정했기 때문에 Precision이 낮아졌습니다.

---

### NG Recall = 1.0

실제 NG는 3장이었습니다.

```
실제 NG
→ 3장

프로그램이 NG로 검출
→ 3장

3 / 3
= 1.0
= 100%
```

즉:

> 실제 불량 3장은 하나도 놓치지 않고 모두 검출했습니다.

그래서:

```
FN = 0
```

입니다.

---

## 30.1 이번 결과를 한눈에 보면

|실제 상태|프로그램 판정|개수|결과|
|---|---|---|---|
|NG|NG|3|TP|
|NG|OK|0|FN|
|OK|OK|0|TN|
|OK|NG|3|FP|

쉽게 말하면:

```
불량 검출
NG 3장
→ 모두 잡음
→ 좋음

하지만

정상 통과
OK 3장
→ 모두 NG로 잘못 판정
→ 문제
```

따라서 현재 Baseline은:

> 불량을 놓치지는 않았지만, 정상 제품까지 너무 엄격하게 NG로 판정하는 Rule

이라고 해석할 수 있습니다.

---

## 30.2 FP 이미지를 직접 확인합니다

이번 Test에서는 FP가 3건 발생했습니다.

```
outputs/day05/fp/
```

에 다음 이미지가 저장되어 있습니다.

```
test_ok_01_result.jpg
test_ok_02_result.jpg
test_ok_03_result.jpg
```

세 이미지 모두:

```
Actual
→ OK

Predicted
→ NG

Error Type
→ FP
```

입니다.

그런데 측정값을 살펴보면 공통적인 특징이 있습니다.

|파일|SIFT Good|기준 129|Inlier Ratio|기준 0.8664|
|---|---|---|---|---|
|`test_ok_01.jpg`|169|통과|0.7811|실패|
|`test_ok_02.jpg`|156|통과|0.8397|실패|
|`test_ok_03.jpg`|204|통과|0.8529|실패|

즉 세 정상 이미지 모두:

```
Homography
→ 성공

SIFT Good Match
→ 기준 통과

하지만

Inlier Ratio
→ 기준 미달
```

이었습니다.

따라서 세 FP의 직접적인 `NG Reason`은 모두:

```
LOW_INLIER_RATIO
```

입니다.

---

## 30.3 FN 폴더에는 왜 이미지가 없나요?

이번 결과는:

```
FN = 0
```

입니다.

FN은:

```
실제 NG
        ↓
프로그램이 OK라고 판정
```

한 경우입니다.

이번에는 실제 NG 3장을 모두 NG로 검출했기 때문에:

```
outputs/day05/fn/
```

에 복사할 이미지가 없습니다.

즉 폴더가 비어 있는 것이 정상 결과입니다.

---

## 30.4 NG Reason과 실제 Failure Cause를 구분합니다

이번 FP 3장의 `NG Reason`은 모두 명확합니다.

```
LOW_INLIER_RATIO
```

이것은:

```
Inlier Ratio
<
0.8664
```

였다는 **프로그램 내부의 직접적인 판정 이유**입니다.

하지만 왜 정상 이미지의 Inlier Ratio가 낮아졌는지는 별도로 살펴봐야 합니다.

실제 이미지를 보면서 다음과 같은 조건을 확인합니다.

```
Calibration 이미지와
촬영 각도가 달랐는가?

제품 크기나 거리가 달랐는가?

포장지 구김이 달라졌는가?

빛 반사가 있었는가?

로고의 보이는 형태가
Calibration과 달라졌는가?
```

정리하면:

```
NG Reason
→ LOW_INLIER_RATIO
→ 프로그램이 NG라고 한 직접 이유

Failure Cause
→ 왜 Inlier Ratio가 낮아졌는가?
→ 실제 이미지를 보고 원인 분석
```

입니다.

---

# 31. Test에서 틀렸다고 바로 Rule을 바꾸지 않습니다

이번 Test에서는 정상 이미지 3장이 모두 FP가 되었습니다.

결과만 보면:

```
Inlier Ratio 기준
0.8664

        ↓

너무 높은 것 아닌가?

        ↓

기준을 낮추면 되지 않을까?
```

라고 생각할 수 있습니다.

하지만 **지금 바로 Threshold를 수정하지 않습니다.**

이번 Test의 목적은:

> **Calibration에서 만든 `day05-logo-v1.0`이 처음 보는 데이터에서 실제로 얼마나 잘 작동하는지 확인하는 것**

이기 때문입니다.

---

## 31.1 이번 결과에서 왜 바로 수정하지 않나요?

현재 실제 결과를 보면:

```
Calibration

Accuracy
→ 91.67%

        ↓

같은 Rule을 Test에 적용

        ↓

Test Accuracy
→ 50%
```

로 성능이 크게 떨어졌습니다.

그리고 정상 Test 이미지 세 장에서 공통적으로:

```
LOW_INLIER_RATIO
```

가 발생했습니다.

이것은 매우 중요한 결과입니다.

```
Calibration에서는 잘 동작했던 기준

        ↓

처음 보는 정상 이미지에서는
Inlier Ratio가 기준보다 낮아짐

        ↓

FP 3건 발생
```

즉 현재 Baseline이 **새로운 정상 이미지의 변화까지 충분히 받아들이지 못하고 있을 가능성**을 확인한 것입니다.

여기서 바로:

```
0.8664
↓
0.84
```

처럼 바꾸면 Test 데이터를 보고 Rule을 맞추는 것이 됩니다.

그러면 더 이상 이 Test 데이터로 현재 Baseline의 객관적인 성능을 평가하기 어렵습니다.

---

## 31.2 이번 실패 결과를 먼저 기록합니다

이번 Test에서는 FP 3건의 공통점을 다음과 같이 기록할 수 있습니다.

```
Baseline
day05-logo-v1.0

Actual
OK

Predicted
NG

FP
3건

공통 NG Reason
LOW_INLIER_RATIO

SIFT Good Match
→ 정상 3장 모두 기준 통과

Homography
→ 정상 3장 모두 계산 성공

Inlier Ratio
→ 정상 3장 모두 0.8664 미만
```

대표적으로:

```
test_ok_01.jpg

Actual
→ OK

Predicted
→ NG

SIFT Good Match
→ 169
→ 기준 통과

Inlier Ratio
→ 0.7811
→ 기준 미달

Error Type
→ FP

NG Reason
→ LOW_INLIER_RATIO
```

입니다.

다른 두 정상 이미지도 같은 유형입니다.

```
test_ok_02
Ratio = 0.8397
→ LOW_INLIER_RATIO

test_ok_03
Ratio = 0.8529
→ LOW_INLIER_RATIO
```

따라서 이번 Test에서 확인한 핵심은:

```
Test 실패
        ↓
즉시 Threshold 수정 X
        ↓
FP 이미지 확인
        ↓
공통 NG Reason 확인

LOW_INLIER_RATIO
        ↓
왜 정상 이미지에서
Ratio가 낮아졌는지 분석
        ↓
현재 Baseline의 한계로 기록
```

입니다.

> 현재 `day05-logo-v1.0`은 불량 3장을 모두 검출하여 NG Recall은 100%였지만, 정상 3장도 모두 NG로 판정하여 FP가 3건 발생했습니다. 따라서 현재 Rule은 불량 검출에는 강하지만 정상 제품을 통과시키는 데에는 지나치게 엄격한 특성을 보였습니다.


---

# PART7 Mini Challenge — 다른 제품으로 5일차 흐름을 스스로 반복합니다 (32~38)

### Mini Challenge 주제

> 다른 제품의 로고가 정상적으로 보이는지 검사하는 Rule을 직접 만들어 보세요.

제품은 팀별로 다음 중 하나를 선택하게 하면 좋습니다.

```
과자 봉지 로고
음료수 라벨
라면 봉지 로고
제품 박스 로고
책 또는 노트 표지
```

가능하면 글자와 모서리가 뚜렷하고 로고가 크게 보이는 제품을 선택합니다.

데이터는 직접 촬영을 합니다.

```
Reference
1장
→ 정면에서 정상 로고 촬영

Calibration OK
3장
→ 위치 / 거리 / 각도를 조금씩 변화

Calibration NG
3장
→ 로고 일부를 카드나 종이로 가림

Test OK
2장
→ Calibration에 사용하지 않은 새 사진

Test NG
2장
→ 다른 위치를 가린 새 사진
```

총:

```
Reference 1장
+
Calibration 6장
+
Test 4장

= 11장
```

이면 충분합니다.

특히 NG는 실제 제품을 훼손할 필요 없이:

```
포스트잇
종이
카드

        ↓

로고 일부 가림
```

으로 만들면 됩니다.

---
# 32. Mini Challenge에서는 무엇을 하나요?

지금까지 광천김을 이용하여 다음 과정을 함께 실습했습니다.

```
Reference 준비
        ↓
Calibration 이미지 측정
        ↓
Template / SIFT / Homography 값 확인
        ↓
OK / NG 분포 비교
        ↓
Threshold 후보 선택
        ↓
Rule 결정
        ↓
JSON으로 Rule Freeze
        ↓
처음 보는 Test 이미지 검사
        ↓
TP / TN / FP / FN 확인
        ↓
실패 원인 분석
```

Mini Challenge에서는 이 과정을 다른 제품에서 직접 다시 해봅니다.

새로운 알고리즘을 배우는 문제가 아닙니다.

> 5일차에서 배운 검사 개발 순서를 스스로 다시 적용할 수 있는지 확인하는 실습입니다.

강사는 광천김과 다른 간단한 제품 이미지를 제공합니다.

예:

```
과자 봉지 로고
음료수 라벨
제품 박스 로고
책 표지
인쇄 카드
```

가능하면 글자와 모서리가 어느 정도 있는 제품을 선택하는 것이 좋습니다.

---

# 33. Mini Challenge 데이터는 작게 준비합니다

Mini Challenge는 대규모 성능평가가 아닙니다.

작은 데이터로 전체 흐름을 다시 경험하는 것이 목적입니다.

권장 데이터는 다음 정도면 충분합니다.

```
Reference
→ 1장

Calibration
→ OK 3장
→ NG 3장

Test
→ OK 2장
→ NG 2장
```

예:

```
samples/day05/mini_challenge/
│
├── reference/
│   └── reference_label.jpg
│
├── calibration/
│   ├── ok/
│   │   ├── ok_01.jpg
│   │   ├── ok_02.jpg
│   │   └── ok_03.jpg
│   │
│   └── ng/
│       ├── ng_01.jpg
│       ├── ng_02.jpg
│       └── ng_03.jpg
│
└── test/
    ├── ok/
    │   ├── test_ok_01.jpg
    │   └── test_ok_02.jpg
    │
    └── ng/
        ├── test_ng_01.jpg
        └── test_ng_02.jpg
```

Mini Challenge에서는 별도의 `Lighting / Rotation / Blur / Glare` 스트레스 테스트 데이터를 추가로 만들지 않습니다.

5일차의 핵심은 `Calibration → Rule Freeze → Test → 실패 분석`의 전체 흐름을 다른 제품에 다시 적용하는 것이므로, Mini Challenge도 이 범위에 집중합니다.

---

# 34. 해야 할 일을 순서대로 확인합니다

Mini Challenge에서는 다음 순서를 지킵니다.

```
① 검사 대상 확인
        ↓
② Reference 준비
        ↓
③ Calibration OK / NG 측정
        ↓
④ 측정값 비교
        ↓
⑤ 사용할 Feature 선택
        ↓
⑥ Threshold 결정
        ↓
⑦ Rule 고정
        ↓
⑧ Test 실행
        ↓
⑨ 실제 정답과 비교
        ↓
⑩ 결과 해석
```

## 34.1 어떤 Feature를 사용할까요?

5일차에서 실제로 측정한 다음 값 중에서 선택합니다.

```
Template Score

SIFT Good Match

Homography Inlier

Inlier Ratio
```

최소 **2개 정도를 비교**하면 충분합니다.

광천김 실습에서는 실제 데이터를 확인한 결과:

```
SIFT Good Match
+
Inlier Ratio
```

를 사용했습니다.

하지만 새로운 제품에서도 똑같은 값이 가장 좋다는 보장은 없습니다.

먼저 측정값을 보고 판단합니다.

```
Template Score는
OK / NG 차이가 있는가?

SIFT Good Match는
어느 쪽이 높은가?

Inlier Ratio는
정상과 불량을 구분하는 데 도움이 되는가?
```

> Feature를 많이 사용하는 것이 목표가 아니라 **새로운 데이터에서 실제로 도움이 되는 Feature를 선택하는 것**이 목표입니다.

현재 공통 함수에서 측정하지 않는 `Position Error`, `Scale Ratio`는 이번 Mini Challenge에서는 사용하지 않습니다.

---

# 35. `39_day05_mini_challenge.py` — 직접 완성합니다

본 실습의 실행 파일은 `38_day05_evaluate.py`까지 이어졌습니다.

Mini Challenge에서는 다음 파일을 사용합니다.

```
src/39_day05_mini_challenge.py
```

이번에는 정답 코드를 모두 제공하지 않습니다.

학생은 오늘 만든 코드를 참고하여 필요한 부분을 직접 완성합니다.

```python
from pathlib import Path

from day05_logo_features import (
    prepare_reference,
    measure_target,
)


ROOT = Path(
    "samples/day05/mini_challenge"
)

REFERENCE_PATH = (
    ROOT
    / "reference/reference_label.jpg"
)

CALIBRATION_DIR = (
    ROOT
    / "calibration"
)

TEST_DIR = (
    ROOT
    / "test"
)


# --------------------------------------------------
# TODO 1
#
# Reference를 준비합니다.
#
# prepare_reference()를 사용합니다.
# --------------------------------------------------


# --------------------------------------------------
# TODO 2
#
# calibration/ok
# calibration/ng
#
# 이미지를 모두 읽어서 측정합니다.
#
# 확인할 값:
#
# template_score
# sift_good_matches
# homography_found
# homography_inliers
# inlier_ratio
# --------------------------------------------------


# --------------------------------------------------
# TODO 3
#
# OK와 NG 측정값을 비교합니다.
#
# 어떤 Feature가 구분에 도움이 되는지
# 최소 2개 확인합니다.
# --------------------------------------------------


# --------------------------------------------------
# TODO 4
#
# Calibration 결과를 보고
# Threshold를 결정합니다.
#
# 예:
#
# SIFT >= ?
# Inlier Ratio >= ?
#
# 숫자를 임의로 정하지 않습니다.
# --------------------------------------------------


# --------------------------------------------------
# TODO 5
#
# 결정한 Rule을 고정합니다.
#
# Test를 시작한 뒤에는
# Threshold를 수정하지 않습니다.
# --------------------------------------------------


# --------------------------------------------------
# TODO 6
#
# test/ok
# test/ng
#
# 이미지에 같은 Rule을 적용합니다.
# --------------------------------------------------


# --------------------------------------------------
# TODO 7
#
# Actual과 Predicted를 비교합니다.
#
# TP
# TN
# FP
# FN
#
# 을 확인합니다.
# --------------------------------------------------


print(
    "Day05 Mini Challenge를 완성하세요."
)
```

다음 코드를 참고할 수 있습니다.

```
day05_logo_features.py

34_day05_measure_calibration.py
→ 여러 이미지 측정 방법

35_day05_analyze_calibration.py
→ 측정값 비교와 Threshold 후보

36_day05_freeze_rule.py
→ Rule 저장 방법

37_day05_test.py
→ Test 적용 방법

38_day05_evaluate.py
→ TP / TN / FP / FN 계산 방법
```

중요한 것은 코드를 전부 암기하는 것이 아닙니다.

```
어떤 기존 코드를
어디에서 재사용해야 하는가?
```

를 스스로 판단하는 것도 Mini Challenge의 일부입니다.

---

# 36. Mini Challenge는 이 정도까지만 하면 됩니다

반드시 확인할 결과는 다음입니다.

```
Calibration에서

OK와 NG의 측정값이
어떻게 달랐는가?

        ↓

어떤 Feature를 선택했는가?

        ↓

Threshold를 얼마로 정했는가?

        ↓

Test에서

몇 장을 맞혔는가?

        ↓

FP / FN이 있었는가?
```

정확도가 낮게 나왔다고 Mini Challenge 실패가 아닙니다.

예를 들어:

```
Test 4장 중
3장 성공
1장 실패
```

했다면 실패한 한 장을 확인합니다.

```
실제 상태
→ OK

프로그램 판정
→ NG

SIFT Good Match
→ Threshold보다 낮음

가능한 이유
→ 촬영 각도 또는 Blur
```

정도만 설명할 수 있어도 충분합니다.

> 좋은 숫자를 만드는 것보다 자신이 만든 Rule이 왜 그렇게 동작했는지 설명할 수 있는 것이 더 중요합니다.

---

# 37. Mini Challenge 결과를 짧게 기록합니다

결과는 다음 파일 하나에 간단히 작성합니다.

```
reports/day05_mini_challenge.md
```

다음 정도면 충분합니다.

```
# Day05 Mini Challenge

## 검사 대상

작성:

## OK 기준

작성:

## NG 기준

작성:

## Calibration 데이터

- OK: 3장
- NG: 3장

## 확인한 Feature

- Feature 1:
- Feature 2:

## Calibration 결과

### OK

작성:

### NG

작성:

## 선택한 Rule

예:

SIFT Good Match >= ?

AND

Inlier Ratio >= ?

## Test 결과

- Test 이미지: 4장
- TP:
- TN:
- FP:
- FN:

## 가장 눈에 띄는 결과

파일:

Actual:

Predicted:

측정값:

왜 이런 결과가 나왔다고 생각하는가?

작성:

## 한 줄 정리

이번 제품에서 어떤 Feature가 가장 유용했는가?

작성:
```

별도의 긴 Failure Analysis 보고서나 Experiment Log를 다시 작성할 필요는 없습니다.

---

# 38. Mini Challenge 완료 기준

다음 항목까지 수행했다면 Mini Challenge는 완료입니다.

- [ ] 광천김과 다른 제품을 사용했다.
- [ ] Reference 이미지를 준비했다.
- [ ] Calibration OK / NG를 구분했다.
- [ ] Calibration 이미지를 실제로 측정했다.
- [ ] 최소 2개의 Feature를 비교했다.
- [ ] 측정값을 보고 Threshold를 결정했다.
- [ ] Test 전에 Rule을 고정했다.
- [ ] Test 이미지에 같은 Rule을 적용했다.
- [ ] TP / TN / FP / FN을 확인했다.
- [ ] 틀린 결과가 있다면 원인을 한 건 이상 확인했다.
- [ ] day05_mini_challenge.md에 결과를 짧게 기록했다.

Mini Challenge에서는 다음까지 할 필요는 없습니다.

```
새로운 AI 모델 학습 X

OCR 추가 X

Anomaly Detection 추가 X

GUI 제작 X

별도의 Lighting / Rotation / Blur / Glare
스트레스 테스트 추가 X

새로운 Feature 알고리즘 개발 X

높은 Accuracy가 나올 때까지
Threshold 반복 수정 X
```
---
# PART8 5일차 정리 · 자가체크 · Git 기록 (39~43)

---

# 39. README에 5일차 결과를 정리합니다

5일차에서 무엇을 했는지 나중에도 확인할 수 있도록 `README.md`에 핵심 결과만 기록합니다.

다음 형식을 참고하여 **자신이 실제 실행한 결과**를 작성합니다.

```
## Day 05 — Calibration · Rule Freeze · Test

### 검사 대상

광천김 포장지의 기준 로고 존재 및 정합 검사

### 사용한 기술

- Template Matching
- SIFT
- RANSAC
- Homography

### Calibration

- OK: 6장
- NG: 6장
- 측정값:
  - Template Score
  - SIFT Good Match
  - Homography Inlier
  - Inlier Ratio

### Baseline Rule

- Version: day05-logo-v1.0
- Homography Found: True
- SIFT Good Match >= 129
- Inlier Ratio >= 0.8664

### Test

- Test 이미지: 6장
- Accuracy:
- TP:
- TN:
- FP:
- FN:
- NG Precision:
- NG Recall:

### Test 실패 분석

- FP: 3
- FN: 0
- 공통 NG Reason: `LOW_INLIER_RATIO`
- Test 결과를 보고 Threshold를 다시 맞추지 않고 현재 Baseline의 한계로 기록

### Mini Challenge

검사한 다른 제품:

작성:

선택한 Feature:

작성:

Test 결과:

작성:

### 핵심 결과 파일

- reports/day05_calibration_measurements.csv
- reports/day05_rule_candidates.csv
- configs/day05_logo_rules.json
- reports/day05_test_results.csv
- reports/day05_test_metrics.md
- reports/day05_mini_challenge.md

### 오늘의 핵심

Calibration 데이터로 판정 기준을 결정하고,
기준을 동결한 뒤 처음 보는 Test 데이터에 적용했다.

Test 결과에서는 Accuracy뿐 아니라 FP와 FN을 확인했고,
FP 3건의 공통 NG Reason을 분석하여 현재 Baseline의 한계를 확인했다.
```

README는 보고서를 길게 다시 작성하는 문서가 아닙니다.

> **5일차에 무엇을 만들었고 어떤 결과가 나왔는지 다시 확인할 수 있을 정도만 기록하면 됩니다.**

---

# 40. 5일차 결과를 Git에 기록합니다

먼저 변경된 파일을 확인합니다.

```bash
git status
```

5일차에서 작성한 코드와 설정, 보고서, README를 추가합니다.

```bash
git add src configs reports README.md
```

Commit 합니다.

```bash
git commit -m "feat: complete day05 calibration rule test"
```

정상적으로 기록되었는지 확인합니다.

```bash
git log --oneline -5
```

5일차 Baseline을 별도로 표시하고 싶다면 Tag를 남길 수도 있습니다.

```bash
git tag day05-logo-v1.0
```

확인합니다.

```bash
git tag
```

---

## 40.1 GitHub 사용이 허용된 환경이라면 업로드합니다

Remote가 이미 연결되어 있다면:

```bash
git remote -v
```

를 먼저 확인합니다.

현재 브랜치를 확인합니다.

```bash
git branch --show-current
```

예를 들어 `main` 브랜치라면:

```bash
git push origin main
```

Tag까지 업로드하려면:

```bash
git push origin day05-logo-v1.0
```

합니다.

기관 정책상 외부 GitHub 업로드가 허용되지 않는 환경에서는 **`git push`를 하지 않고 Local Git Commit까지만** 진행합니다.

---

# 41. 5일차 자가체크

수업을 마치기 전에 직접 확인합니다.

## 데이터와 측정

- [ ]  Calibration과 Test의 역할 차이를 설명할 수 있다.
- [ ]  Calibration OK 6장과 NG 6장을 측정했다.
- [ ]  Template Score를 확인했다.
- [ ]  SIFT Good Match를 확인했다.
- [ ]  Homography Inlier와 Inlier Ratio를 확인했다.
- [ ]  한 장의 측정값만 보고 Threshold를 정하지 않았다.

## Calibration과 Rule

- [ ]  OK와 NG의 측정값 분포를 비교했다.
- [ ]  여러 Threshold 후보를 시험했다.
- [ ]  SIFT와 Inlier Ratio를 함께 사용하는 이유를 이해했다.
- [ ]  사용할 Rule을 결정했다.
- [ ]  `day05_logo_rules.json`을 만들었다.
- [ ]  Test 전에 Rule을 동결했다.

## Test와 평가

- [ ]  처음 보는 Test 이미지 6장에 같은 Rule을 적용했다.
- [ ]  Test 중 Threshold를 수정하지 않았다.
- [ ]  Actual과 Predicted를 비교했다.
- [ ]  TP / TN / FP / FN을 구분할 수 있다.
- [ ]  Accuracy를 확인했다.
- [ ]  FP와 FN이 있다면 결과 이미지를 직접 확인했다.
- [ ]  NG Reason과 실제 Failure Cause의 차이를 이해했다.

## Mini Challenge

- [ ]  광천김과 다른 제품을 사용했다.
- [ ]  새로운 Reference를 준비했다.
- [ ]  Calibration OK / NG를 측정했다.
- [ ]  최소 2개의 Feature를 비교했다.
- [ ]  직접 Threshold를 결정했다.
- [ ]  Test 전에 Rule을 고정했다.
- [ ]  새로운 Test 이미지에 Rule을 적용했다.
- [ ]  TP / TN / FP / FN을 확인했다.
- [ ]  결과를 `day05_mini_challenge.md`에 정리했다.

## 기록

- [ ]  README에 5일차 결과를 작성했다.
- [ ]  `git status`로 변경 내용을 확인했다.
- [ ]  Git Commit을 완료했다.
- [ ]  필요한 경우 `day05-logo-v1.0` Tag를 만들었다.
- [ ]  외부 업로드가 허용된 경우 GitHub에 Push했다.

---

# 42. 5일차에서 반드시 이해해야 할 핵심

5일차에서 가장 중요한 개념을 마지막으로 확인합니다.

### Calibration은 무엇인가요?

여러 정상·불량 이미지의 측정값을 비교하여:

```
어디까지 OK이고
어디부터 NG인가?
```

라는 검사 기준을 결정하는 과정입니다.

### Rule Freeze는 무엇인가요?

Calibration에서 결정한 기준을 고정하여:

```
이제부터 이 기준으로 검사한다.
```

라고 확정하는 과정입니다.

### Test는 무엇인가요?

동결한 Rule을 처음 보는 데이터에 적용하여:

```
새로운 이미지에서도
잘 동작하는가?
```

를 확인하는 과정입니다.

### FP와 FN은 무엇인가요?

이번 수업에서는 NG를 Positive로 사용했습니다.

```
FP
실제 OK → 프로그램 NG
→ 정상 과검출

FN
실제 NG → 프로그램 OK
→ 불량 미검출
```

특히 제조검사에서는 **불량을 정상으로 통과시키는 FN**을 주의해서 봅니다.

### 실패 분석은 무엇인가요?

Test에서 틀린 결과를 바로 Threshold 수정으로 덮지 않고, `FP / FN`, `NG Reason`, 실제 이미지를 함께 확인하여 현재 Rule이 어떤 조건에서 실패했는지 설명하는 과정입니다.

이번 Test에서는 정상 이미지 3장이 모두 `LOW_INLIER_RATIO`로 FP가 되었고, 이를 현재 Baseline의 한계로 기록했습니다.

### Mini Challenge는 무엇인가요?

광천김에서 강사와 함께 배운:

```
Calibration
→ Threshold
→ Rule
→ Freeze
→ Test
→ 평가
```

과정을 **다른 제품에서 학생이 스스로 다시 수행하는 실습**입니다.

---

# 43. 1~5일차 전체 흐름을 연결합니다

지금까지 배운 내용을 하나로 연결하면 다음과 같습니다.

```
[1일차]

무엇을 검사할 것인가?
        ↓
카메라 / 이미지
        ↓
ROI 설정


[2일차]

이미지 품질 확인
        ↓
Brightness
Contrast
Noise
Blur
        ↓
Image Enhancement


[3일차]

병뚜껑 검사
        ↓
Binary
Morphology
Contour
        ↓
Area / Count / BBox / Center
        ↓
Candidate Rule


[4일차]

광천김 로고 검사
        ↓
Template Matching
ORB
SIFT
RANSAC
Homography
        ↓
여러 측정 방법 비교


[5일차]

광천김 Calibration 데이터
        ↓
여러 이미지 측정
        ↓
OK / NG 분포 비교
        ↓
Threshold 후보 비교
        ↓
복합 Rule 결정
        ↓
Rule Freeze
        ↓
처음 보는 Test
        ↓
TP / TN / FP / FN
        ↓
성능 평가
        ↓
FP / FN 및 실패 원인 확인
        ↓
Mini Challenge
        ↓
다른 제품으로
학생이 전체 과정 직접 반복
```

5일차에서 외워야 할 가장 중요한 흐름은 이것입니다.

```
측정
→ 비교
→ 기준 결정
→ 기준 동결
→ Test
→ 평가
→ 실패 원인 확인
```

> 좋은 Threshold 숫자를 외우는 것이 목적이 아니라, 데이터를 근거로 검사 기준을 만들고 새로운 데이터에서 검증하는 머신비전 검사 개발 절차를 이해하는 것이 5일차의 핵심입니다.