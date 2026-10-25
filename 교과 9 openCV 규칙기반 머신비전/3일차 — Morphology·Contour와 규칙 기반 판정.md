## 오늘의 목표

1일차에서는 실제 이미지·저장 영상·스마트폰 실시간 영상에서 **검사할 위치인 ROI**를 정하고, 병뚜껑이 있는 상태와 없는 상태의 ROI 이미지를 직접 저장했습니다.

2일차에서는 그 ROI가 검사에 적합한 영상 상태인지 확인하면서 Brightness, Contrast, Gamma, CLAHE, Filter, Sharpening을 비교하고, 마지막에는 Threshold / Otsu를 이용해 **객체와 배경을 Binary Image로 분리**했습니다.

3일차에서는 그 다음 단계로 넘어갑니다.

> Binary Image에서 실제 객체를 어떻게 찾고, 사람의 판단을 숫자와 Rule로 바꿀 것인가?

오늘 수업이 끝나면 다음을 할 수 있어야 합니다.

```text
- 1·2일차에서 만든 실제 ROI 이미지를 그대로 이어서 사용할 수 있다.
- 이미 ROI로 잘라진 이미지에 다시 ROI 좌표를 적용하면 안 되는 이유를 설명할 수 있다.
- Threshold/Otsu 결과가 최종 판정이 아니라 Contour를 찾기 위한 입력이라는 것을 설명할 수 있다.
- Opening과 Closing을 적용하고 Kernel·Iteration에 따른 결과 차이를 비교할 수 있다.
- Binary Image에서 Contour를 찾을 수 있다.
- Contour의 Area, Count, Bounding Box, Center를 계산할 수 있다.
- 병뚜껑 있음 / 없음에서 실제 측정값이 어떻게 다른지 비교할 수 있다.
- 실제 측정값을 보기 전에 임의의 Area 기준값을 정하면 안 되는 이유를 설명할 수 있다.
- 두 기준 이미지에서 교육용 Candidate Rule을 만들 수 있다.
- 추가 촬영 이미지에 같은 Rule을 적용하고 결과를 비교할 수 있다.
- Mini Challenge에서 검사 문제에 맞는 Feature와 처리 방법을 직접 선택할 수 있다.
- 검사 결과와 NG Reason을 CSV와 분석 문서로 기록할 수 있다.
- README와 Local Git에 3일차 작업을 정리할 수 있다.
```

---

# 1. 오늘 수업의 전체 흐름

![[Pasted image 20261021102716.png]]

Morphology의 정확한 명칭은 수학적 형태학(Mathematical Morphology)입니다.
Otsu는 픽셀을 검정(0)과 흰색(255)으로 구분하는 작업이고, Morphology는 그렇게 구분된 흰색 영역의 모양을 정리하는 작업입니다.


**1일차는** IP Webcam의 실시간 화면에서 검사할 위치를 정하고 **ROI를 잘라 저장하는 방법**을 배운 날입니다.

**2일차는** ROI 이미지의 상태에 따라 Brightness, Gamma, CLAHE, Gaussian, Median 같은 여러 전처리 방법을 비교하고 어떤 방법을 사용할지 판단하는 과정을 배웠습니다. 마지막에는 Gray 이미지에 Threshold / Otsu를 적용하여 객체와 배경을 검정과 흰색의 Binary Image로 분리했고, 그 결과 중 하나가 `otsu.jpg`입니다.

**3일차는** 같은 ROI 이미지를 다시 사용하여 `Gray → Otsu`로 Binary Image를 다시 만든 뒤, Morphology로 영역을 정리하고 Contour로 객체를 찾습니다. 이후 Area, Count, BBox, Center 같은 Feature를 숫자로 측정하고, `cap_on`과 `cap_off`의 실제 측정값 차이를 비교하여 Rule을 만든 뒤 OK / NG를 판정하는 과정을 배우는 날입니다.

```text
1일차
스마트폰 IP Webcam 실시간 화면
        ↓
실시간 Frame 수신
        ↓
병뚜껑 검사 ROI 선택
        ↓
ROI 이미지만 잘라서 저장
        ↓
cap_on.jpg / cap_off.jpg 확보

        ↓

2일차
1일차 ROI 이미지를 기준 이미지로 사용
        ↓
cap_on_reference.jpg / cap_off_reference.jpg
        ↓
ROI 이미지 품질 문제 확인
        ↓
Brightness / Gamma / CLAHE / Filter 등
전처리 방법 비교
        ↓
필요한 전처리인지 판단
        ↓
Gray
        ↓
Threshold / Otsu 실습
        ↓
객체와 배경을
Binary로 분리하는 방법 확인

※ 여기서 만든 otsu.jpg를
   3일차 입력 파일로 넘기는 것은 아님

        ↓

3일차
같은 ROI 기준 이미지
cap_on / cap_off 다시 사용
        ↓
Gray
        ↓
2일차에서 확인한
Threshold / Otsu 방식 적용
        ↓
Binary Image 생성
        ↓
Morphology
Opening / Closing으로 영역 정리
        ↓
Contour
객체 경계 찾기
        ↓
Area / Count / BBox / Center 측정
        ↓
cap_on / cap_off
실제 측정값 비교
        ↓
Candidate Rule 만들기
        ↓
OK / NG 판정
        ↓
추가 촬영 ROI 이미지에
같은 Rule 반복 적용
        ↓
Batch Test 결과 확인
실패 사례 분석
        ↓
측정값과 판정 결과 비교
        ↓
Mini Challenge
        ↓
CSV · 분석문서 · README · Local Git
```

---

# 결과 미리보기

```bash
python src/22_cap_batch_test.py
```

![[Pasted image 20261021105135.png|160]]![[Pasted image 20261021105144.png|171]]![[Pasted image 20261021105154 (2).png|172]]![[Pasted image 20261021105202 1.png|168]]

사진에 표시된 `0.290`, `0.317`, `0.560`, `0.457`은 Area Ratio(면적 비율) 입니다.

---
### 오늘의 사고방식

오늘부터는 측정과 판정을 분리해서 생각합니다.

```text
OpenCV가 하는 일
→ 객체를 찾고 숫자를 측정

Python Rule이 하는 일
→ 그 숫자를 기준과 비교해 판단
```

예를 들어:

```text
largest_area_ratio = 0.31
valid_count = 1
center_x = 0.52
```

는 측정값입니다.

반면:

```text
이 값이면 병뚜껑이 있다고 볼 것인가?
→ OK

이 값이면 병뚜껑이 없다고 볼 것인가?
→ NG
```

는 사람이 만든 판정 Rule입니다.

> 오늘은 “Area가 얼마면 정상이다”라는 숫자를 외우게 아니고, 실제 이미지에서 값을 먼저 측정하고, 그 차이를 확인한 뒤 기준 후보를 만드는 날입니다.

![[Pasted image 20260829102708.png]]


---

# 2. 1일차부터 3일차까지 정확히 어떻게 연결되나요?

1일차 스마트폰 실시간 ROI 실습에서 다음 파일을 만들었습니다.

```text
outputs/live_roi/
├── cap_on.jpg
└── cap_off.jpg
```

2일차에서는 이 두 파일을 기준 이미지로 사용하기 위해 다음 이름으로 복사했습니다.

```text
samples/day02/
├── cap_on_reference.jpg
└── cap_off_reference.jpg
```

그리고 2일차 마지막에는 이 ROI 이미지를 Threshold / Otsu로 Binary Image로 바꾸어 병뚜껑과 배경이 분리되는지 확인했습니다.

3일차는 여기서 바로 이어집니다.

```text
samples/day02/cap_on_reference.jpg
samples/day02/cap_off_reference.jpg
        ↓
이미 ROI로 잘라진 실제 촬영 이미지
        ↓
Gray
        ↓
2일차에서 확인한 Binary 방식(otsu.jpg)
        ↓
Morphology
        ↓
Contour
        ↓
측정값
        ↓
Rule
```

## 2.1 매우 중요한 변경점 — 3일차에서는 ROI를 다시 자르지 않습니다

기존 3일차 예제처럼 다음 코드를 다시 넣지 않습니다.

```python
x1, y1, x2, y2 = 190, 160, 440, 460
roi = image[y1:y2, x1:x2]
```

왜냐하면 현재 `cap_on_reference.jpg`와 `cap_off_reference.jpg`는 1일차에 실시간 영상에서 이미 병뚜껑 ROI만 잘라 저장한 이미지이기 때문입니다.

```text
전체 Camera Frame
       ↓
1일차에서 ROI Crop
       ↓
cap_on.jpg / cap_off.jpg
       ↓
2일차에서 reference 이미지로 복사
       ↓
이미 ROI 상태
```

이 상태에서 과거의 전체 이미지 좌표를 다시 적용하면 다음 문제가 생길 수 있습니다.

```text
이미 잘린 ROI
       ↓
다시 x1:y1:x2:y2 Crop
       ↓
엉뚱한 부분만 남음
또는
빈 이미지가 만들어짐
```

따라서 3일차의 병뚜껑 주 실습은 입력 이미지 전체가 이미 ROI라고 가정합니다.

![[Pasted image 20260829183556.png]]

**1일차에는** IP Webcam 실시간 화면에서 병뚜껑 부분을 ROI로 잘라 `cap_on.jpg`, `cap_off.jpg`를 저장했습니다.

**2일차에는** 이 두 ROI 이미지를 `cap_on_reference.jpg`, `cap_off_reference.jpg`로 복사하여 여러 전처리 방법과 Threshold / Otsu를 연습했습니다. 이때 만들어진 `otsu.jpg`는 Binary Image를 만드는 방법을 확인하기 위한 실습 결과입니다.

**3일차에는** 2일차의 `otsu.jpg`를 그대로 사용하는 것이 아니라, 다시 `cap_on_reference.jpg`, `cap_off_reference.jpg`를 기준 이미지로 사용합니다. 여기에 `Gray → Otsu → Binary → Morphology → Contour → Feature 측정`을 적용하고, 두 이미지의 실제 측정값 차이를 비교하여 **Rule 기반 OK / NG 판정**을 만들게 됩니다.

```
1일차 ROI 이미지 확보 → 
2일차 전처리·이진화 방법 연습 → 
3일차 같은 ROI로 실제 측정값을 만들고 Rule 기반 OK / NG 판정
```

2일차의 `otsu.jpg`가 3일차로 넘어가는 것이 아니라, 2일차에서 배운 이진화 방법을 3일차에서 같은 ROI 이미지에 다시 적용하는 것입니다.

---

# 3. 2일차의 Binary 결과를 그대로 이어서 생각합니다

2일차 기준 병뚜껑은 어두운 색이고 배경은 상대적으로 밝았습니다.

그래서 기준 이미지 실습에서는 다음 방향을 사용했습니다.

```text
어두운 병뚜껑
        ↓
THRESH_BINARY_INV
        ↓
병뚜껑을 흰색 객체로 표시
```

Otsu까지 함께 사용하면 다음과 같습니다.

```python
otsu_value, binary = cv2.threshold(
    gray,
    0,
    255,
    cv2.THRESH_BINARY_INV
    + cv2.THRESH_OTSU,
)
```

3일차의 `cap_on_reference.jpg`, `cap_off_reference.jpg`도 같은 기준으로 시작합니다.

하지만 이것이 모든 이미지의 정답이라는 뜻은 아닙니다.

2일차 Mini Challenge처럼 밝은 병뚜껑을 흰색 객체로 보고 싶다면:

```text
밝은 객체
→ THRESH_BINARY
```

를 사용할 수 있습니다.

따라서 오늘도 다음 원칙을 유지합니다.

> Threshold 종류를 외우는 것이 아니라 검사할 객체를 흰색 영역으로 만들기 위해 어떤 방향이 맞는지 판단합니다.

---

# 4. 오늘 사용할 파일명과 프로젝트 구조

오늘 추가할 주요 파일은 다음과 같습니다.

```text
18_morphology_compare.py
19_contour_measure.py
20_cap_feature_compare.py
21_cap_rule.py
22_cap_batch_test.py
23_day03_challenge.py
```

전체 추가 구조는 다음과 같습니다.

```text
subject9_machinevision/
│
├── samples/
│   ├── day02/
│   │   ├── cap_on_reference.jpg
│   │   ├── cap_off_reference.jpg
│   │   ├── challenge.jpg
│   │   └── glare.jpg
│   │
│   └── day03/
│       ├── reference/
│       │   ├── cap_on.jpg
│       │   └── cap_off.jpg
│       │
│       ├── cap_on/
│       │   ├── cap_on_001.jpg   ← Batch Reference
│       │   ├── cap_on_002.jpg   ← Test
│       │   └── cap_on_003.jpg   ← Test
│       │
│       ├── cap_off/
│       │   ├── cap_off_001.jpg  ← Batch Reference
│       │   ├── cap_off_002.jpg  ← Test
│       │   └── cap_off_003.jpg  ← Test
│       │
│       └── challenge/
│           ├── ok_01.jpg
│           ├── ng_01.jpg
│           ├── ok_02.jpg
│           └── ng_02.jpg
│
├── outputs/
│   ├── ...
│   └── day03/
│       ├── morphology/
│       ├── contour/
│       ├── feature/
│       ├── rule/
│       ├── batch/
│       └── challenge/
│
├── src/
│   ├── 00_check_env.py
│   ├── ...
│   ├── 18_morphology_compare.py
│   ├── 19_contour_measure.py
│   ├── 20_cap_feature_compare.py
│   ├── 21_cap_rule.py
│   ├── 22_cap_batch_test.py
│   └── 23_day03_challenge.py
│
├── reports/
│   ├── day01_problem_definition.md
│   ├── dataset_roles.csv
│   ├── day02_parameter_table.csv
│   ├── day02_preprocess_reason.md
│   ├── day03_reference_features.csv
│   ├── day03_inspection_results.csv
│   └── day03_rule_analysis.md
│
├── README.md
└── ...
```

| 파일명                         | 역할               | 설명                                                                                               |
| --------------------------- | ---------------- | ------------------------------------------------------------------------------------------------ |
| `18_morphology_compare.py`  | Morphology 비교    | Binary 이미지에 Opening·Closing을 적용하고 Kernel 크기(3×3, 5×5, 9×9)에 따라 Noise 제거와 객체 형태가 어떻게 달라지는지 비교합니다. |
| `19_contour_measure.py`     | 객체 윤곽선 검출·측정     | 병뚜껑의 Contour(윤곽선)를 찾아 면적(Area), 면적 비율(Area Ratio), 위치(BBox), 중심점(Center)을 측정합니다.                 |
| `20_cap_feature_compare.py` | 정상·불량 특징값 비교     | 병뚜껑 있음(`cap_on`)과 없음(`cap_off`) 이미지의 면적 비율, 객체 개수, 중심 위치 등을 측정하여 어떤 특징값으로 구분할 수 있는지 확인합니다.       |
| `21_cap_rule.py`            | OK / NG 판정 규칙 생성 | 정상·불량 이미지에서 측정한 면적 비율의 중간값을 임시 기준으로 설정하고, `if` 조건문으로 OK / NG를 판정합니다.                             |
| `22_cap_batch_test.py`      | 여러 이미지 일괄 검사     | 기준 이미지 2장으로 만든 판정 규칙을 새로운 테스트 이미지 4장에 적용하여 정답 여부와 정확도를 확인하고 결과를 CSV로 저장합니다.                      |
| `23_day03_challenge.py`     | 나만의 검사 규칙 만들기    | 새로운 검사 문제를 직접 선택하고 필요한 전처리·특징값·판정 기준을 설계하여 OK / NG 검사 프로그램을 완성하는 Mini Challenge입니다.              |
![[Pasted image 20261020082225.png]]

---

# 5. 3일차 실습환경 다시 열기

프로젝트 폴더로 이동합니다.

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

OpenCV 환경도 확인합니다.

```bash
python src/00_check_env.py
```

VS Code를 엽니다.

```bash
code -r .
```

> 새 가상환경을 만들거나 OpenCV를 다시 설치하지 않습니다. 1일차부터 같은 환경을 계속 사용합니다.

---

# 6. 오늘 사용할 실제 기준 이미지를 Day03 폴더에 고정합니다

3일차에서 2일차의 기준 파일을 실수로 덮어쓰지 않도록 복사본을 만듭니다.

먼저 폴더를 만듭니다.

```bash
mkdir -p samples/day03/reference
mkdir -p samples/day03/cap_on
mkdir -p samples/day03/cap_off
mkdir -p samples/day03/challenge

mkdir -p outputs/day03/morphology
mkdir -p outputs/day03/contour
mkdir -p outputs/day03/feature
mkdir -p outputs/day03/rule
mkdir -p outputs/day03/batch
mkdir -p outputs/day03/challenge
```

2일차 기준 이미지를 복사합니다.

```bash
cp samples/day02/cap_on_reference.jpg \
   samples/day03/reference/cap_on.jpg

cp samples/day02/cap_off_reference.jpg \
   samples/day03/reference/cap_off.jpg
```

확인합니다.

```bash
ls -lh samples/day03/reference
```

다음 두 파일이 있으면 됩니다.

```text
cap_on.jpg
cap_off.jpg
```

이 파일은 새로 만든 가짜 이미지가 아닙니다.

```text
1일차 실제 스마트폰 촬영
→ 실제 ROI 저장
→ 2일차 전처리 기준
→ 3일차 Contour·Rule 기준
```

으로 이어지는 같은 데이터입니다.

---

# 7. Morphology가 왜 필요한가요?

2일차에서 Threshold / Otsu를 적용하면 흑백 Binary Image가 만들어집니다.

![[Group 12.png]]

예를 들어 병뚜껑이 흰색 객체가 되었다고 생각해 봅니다.

하지만 실제 Binary Image는 항상 깨끗하지 않습니다.

```text
작은 흰색 점
끊어진 경계
객체 안의 작은 검정 구멍
배경의 작은 흰색 영역
```

이런 상태에서 바로 Contour를 찾으면:

```text
실제 병뚜껑 1개
+
작은 Noise 여러 개
        ↓
Contour가 여러 개
```

가 될 수 있습니다.

Morphology는 이 Binary Image의 형태를 정리하는 단계입니다.

오늘은 다음 두 가지를 중심으로 봅니다.

```text
Opening
→ 작은 흰색 Noise 제거에 유리

Closing
→ 작은 구멍이나 끊어진 흰색 영역 연결에 유리
```

---

# 8. Opening과 Closing을 이해합니다

## 8.1 Opening

Opening은 작은 흰색 영역을 줄이는 데 많이 사용합니다.

![[Pasted image 20260829194329.png]]

하지만 너무 강하면 실제 객체의 작은 부분도 사라질 수 있습니다.

```text
Noise 제거 성공
        +
실제 병뚜껑 홈도 사라짐
        ↓
좋은 결과라고 할 수 없음
```

## 8.2 Closing

Closing은 흰색 객체 내부의 작은 구멍이나 끊어진 영역을 연결할 때 사용할 수 있습니다.

![[Pasted image 20260829194340.png]]

하지만 너무 강하면 서로 다른 객체가 하나로 붙을 수 있습니다.

---

# 9. Kernel과 Iteration은 왜 비교해야 하나요?

![[Pasted image 20260829195047.png]]

![[Pasted image 20260829195147.png]]

**Kernel**: Morphology가 주변 픽셀을 확인할 때 사용하는 작은 구조 요소입니다. 크기가 클수록 이미지에 미치는 영향 범위가 커집니다.

**Iteration**: 같은 Morphology 연산을 몇 번 반복할지 정하는 값입니다. 횟수가 많을수록 변화가 더 강하게 적용됩니다.

Opening과 Closing의 역할

```
Opening
→ 작은 흰색 Noise를 없애는 데 유리

Closing
→ 객체 안의 작은 검은 구멍을 메우거나
   끊어진 흰색 영역을 연결하는 데 유리
```

Kernel과 Iteration은 Opening/Closing을 얼마나 강하게 적용할지 정하는 값입니다.

```
Kernel
→ 한 번 연산할 때 주변을 얼마나 넓게 볼지
→ 3×3, 5×5, 9×9

Iteration
→ 그 연산을 몇 번 반복할지
→ 1회, 2회, 3회
```

예를 들어:

```
Closing + Kernel 3×3 + Iteration 1
→ 작은 검은 구멍을 약하게 메움

Closing + Kernel 9×9 + Iteration 3
→ 훨씬 넓고 강하게 메움
→ 너무 강하면 원래 형태까지 변형될 수 있음
```

정리하면

```
Opening / Closing (흰점을 없앨것인가 검은점을 메울것이냐)
= 무엇을 할 것인가?

Kernel (반영을 얼마나 정할거냐)
= 어느 정도 범위에 적용할 것인가?

Iteration (kernel을 몇번 반복할거냐)
= 몇 번 반복할 것인가?
```

따라서:

> Morphology의 목표는 “최대한 깨끗하게 만들기”가 아니라 Contour가 안정적으로 측정될 정도만 정리하는 것입니다.

---

# 10. `18_morphology_compare.py` — 실제 병뚜껑 Binary를 정리해 봅니다

첫 번째 3일차 파일을 만듭니다.

```text
src/18_morphology_compare.py
```

다음 코드는 `cap_on.jpg` 전체를 이미 ROI라고 보고 처리합니다.

```python

# ============================================================
# ① 필요한 라이브러리 불러오기
# - Path  : 이미지 파일 및 저장 폴더 경로 처리
# - cv2   : 이미지 전처리와 Morphology 수행
# - numpy : 이미지 배열 처리
# ============================================================

from pathlib import Path

import cv2
import numpy as np


# ============================================================
# ② 입력 이미지 및 결과 저장 경로 설정
# - cap_on.jpg : 병뚜껑이 있는 기준 이미지
# - output_dir : 전처리 결과 이미지 저장 폴더
# - 폴더가 없으면 자동으로 생성합니다.
# ============================================================

image_path = Path(
    "samples/day03/reference/cap_on.jpg"
)

output_dir = Path(
    "outputs/day03/morphology"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# ============================================================
# ③ 병뚜껑 이미지 불러오기 및 오류 확인
# - imread()로 기준 이미지를 읽습니다.
# - 이미지가 없거나 읽지 못하면 오류를 발생시킵니다.
# ============================================================

image = cv2.imread(
    str(image_path)
)

if image is None:
    raise FileNotFoundError(
        f"이미지를 읽지 못했습니다: {image_path}"
    )


# ============================================================
# ④ ROI 이미지 확인 및 Gray 변환
# - 입력 이미지는 이미 ROI로 잘린 이미지입니다.
# - 따라서 추가로 Crop하지 않습니다.
# - BGR 컬러 이미지를 Gray 이미지로 변환합니다.
# ============================================================

roi = image

gray = cv2.cvtColor(
    roi,
    cv2.COLOR_BGR2GRAY,
)


# ============================================================
# ⑤ Otsu 이진화 및 Gray·Binary 이미지 저장
# - Otsu가 밝기 기준값을 자동으로 계산합니다.
# - BINARY_INV : 어두운 병뚜껑을 흰색 객체로 변환
# - Gray와 Binary 결과를 각각 JPG로 저장합니다.
# ============================================================

otsu_value, binary = cv2.threshold(
    gray,
    0,
    255,
    cv2.THRESH_BINARY_INV
    + cv2.THRESH_OTSU,
)

cv2.imwrite(
    str(output_dir / "01_gray.jpg"),
    gray,
)

cv2.imwrite(
    str(output_dir / "02_binary.jpg"),
    binary,
)


# ============================================================
# ⑥ 비교할 Kernel 크기 설정
# - 3×3, 5×5, 9×9 Kernel을 비교합니다.
# - Kernel 크기에 따라 노이즈 제거와 객체 형태가 달라집니다.
# ============================================================

kernel_sizes = [
    3,
    5,
    9,
]


# ============================================================
# ⑦ Kernel 크기별 Opening → Closing 반복 실행
# - 반복문으로 3×3, 5×5, 9×9 Kernel을 차례대로 사용합니다.
# - Opening : 작은 흰색 노이즈 제거
# - Closing : 작은 검은색 틈이나 구멍 메우기
# - 각 Kernel의 결과 이미지를 2장씩 저장합니다.
# ============================================================

for size in kernel_sizes:

    # 현재 크기의 타원형 Kernel 생성
    kernel = cv2.getStructuringElement(
        cv2.MORPH_ELLIPSE,
        (size, size),
    )

    # Opening 적용
    opened = cv2.morphologyEx(
        binary,
        cv2.MORPH_OPEN,
        kernel,
        iterations=1,
    )

    # Opening 결과에 Closing 적용
    cleaned = cv2.morphologyEx(
        opened,
        cv2.MORPH_CLOSE,
        kernel,
        iterations=1,
    )

    # Opening 결과 저장
    cv2.imwrite(
        str(
            output_dir
            / f"kernel_{size}_opening.jpg"
        ),
        opened,
    )

    # Closing 결과 저장
    cv2.imwrite(
        str(
            output_dir
            / f"kernel_{size}_closing.jpg"
        ),
        cleaned,
    )


# ============================================================
# ⑧ 실행 결과 출력
# - 사용한 입력 이미지 경로를 확인합니다.
# - Otsu가 계산한 이진화 기준값을 확인합니다.
# - Morphology 비교 이미지 저장 완료 메시지를 출력합니다.
# ============================================================

print("입력:", image_path)
print("Otsu Threshold:", otsu_value)
print("Morphology 비교 결과 저장 완료")
```

### 의사코드

```text
[① 필요한 라이브러리 불러오기]
Path, OpenCV, NumPy 준비
        ↓
[② 입력 이미지 및 결과 저장 경로 설정]
입력: samples/day03/reference/cap_on.jpg
출력: outputs/day03/morphology/
        ↓
결과 저장 폴더 생성
        ↓
[③ 병뚜껑 이미지 불러오기]
cap_on.jpg 읽기
        ↓
이미지를 정상적으로 읽었는지 확인
        ↓
이미지 읽기 실패 → 오류 발생
        ↓
[④ ROI 이미지 확인 및 Gray 변환]
이미 ROI로 잘린 이미지이므로
추가 Crop 없이 그대로 사용
        ↓
BGR → Gray 변환
        ↓
[⑤ Otsu 이진화 및 결과 저장]
Otsu로 밝기 기준값 자동 계산
        ↓
어두운 병뚜껑 → 흰색 객체
밝은 배경 → 검은색
        ↓
01_gray.jpg 저장
02_binary.jpg 저장
        ↓
[⑥ 비교할 Kernel 크기 설정]
3×3, 5×5, 9×9
        ↓
[⑦ Kernel 크기별 Morphology 반복 실행]
        ↓
3×3 Kernel 생성
  → Opening → Closing → 이미지 저장
        ↓
5×5 Kernel 생성
  → Opening → Closing → 이미지 저장
        ↓
9×9 Kernel 생성
  → Opening → Closing → 이미지 저장
        ↓
Kernel별 Opening / Closing 결과 총 6장 저장
        ↓
[⑧ 실행 결과 출력]
입력 이미지 경로 출력
Otsu Threshold 출력
Morphology 비교 결과 저장 완료 출력
```

### 실행합니다.

```bash
python src/18_morphology_compare.py
```

### 실행 결과 확인

앞에서 작성한 코드를 실행하여 
**Kernel 크기(3×3, 5×5, 9×9)에 따라 병뚜껑 이미지가 어떻게 달라지는지** 확인하겠습니다.

```text
입력: samples/day03/reference/cap_on.jpg
Otsu Threshold: 111.0
Morphology 비교 결과 저장 완료
```

각 출력의 의미는 다음과 같습니다.

| 실행 결과 | 의미 |
|---|---|
| `입력: .../cap_on.jpg` | 병뚜껑이 있는 기준 이미지를 정상적으로 불러왔습니다. |
| `Otsu Threshold: 111.0` | Otsu 알고리즘이 이미지의 밝기를 분석하여 이진화 기준값을 111로 결정했습니다. |
| `Morphology 비교 결과 저장 완료` | Gray, Binary, Kernel별 Opening·Closing 결과를 저장하는 작업이 끝났습니다. |

**Otsu Threshold 111.0이란?**

Gray 이미지의 밝기는 0(검정)부터 255(흰색)까지입니다. Otsu 알고리즘이 자동으로 찾은 기준값은 111이며, 이번 코드에서는 `BINARY_INV`를 사용하므로 111보다 어두운 픽셀은 흰색, 밝은 픽셀은 검정으로 변환됩니다.

이 값은 검사 정확도나 병뚜껑 면적 비율이 아니라 **흑백 이진화에 사용하는 밝기 기준값**입니다.

### 저장된 결과 이미지의 경로확인

```
outputs/day03/morphology/
```

```text
subject9_machinevision/
└── outputs/
    └── day03/
        └── morphology/
            ├── 01_gray.jpg
            ├── 02_binary.jpg
            ├── kernel_3_opening.jpg
            ├── kernel_3_closing.jpg
            ├── kernel_5_opening.jpg
            ├── kernel_5_closing.jpg
            ├── kernel_9_opening.jpg
```

---

# 11. Morphology 결과에서 무엇을 확인하나요?

앞에서 하나의 Binary Image에 `3×3`, `5×5`, `9×9` Kernel을 사용하여 Opening과 Closing을 적용했습니다.

이번 실습에서 중요한 것은 어떤 이미지가 가장 깨끗해 보이는지를 고르는 것이 아닙니다.

각 단계에서 무엇이 변했는지를 순서대로 확인합니다.

```text
02_binary.jpg
      ↓
Opening
작은 흰색 영역을 줄임
      ↓
Closing
작은 검은 구멍과
끊어진 영역을 정리
      ↓
최종적으로
Contour를 측정하기 좋은지 확인
```

출력된 결과 이미지는 다음과 같습니다.

![[Pasted image 20261025130648.png]]

## 11.1 먼저 원본 Binary Image를 확인합니다

```
02_binary.jpg
```

현재 결과를 보면 병뚜껑은 흰색 객체로 분리되었지만 내부와 경계에 검은 영역이 남아 있습니다.

특히 다음 부분을 확인합니다.

```
병뚜껑 내부에 검은 틈이 있는가?

작은 흰색 Noise가 있는가?

병뚜껑 외곽이 끊겨 있는가?

병뚜껑 이외의 영역까지
흰색 객체로 선택되어 있는가?
```

이 이미지가 Morphology를 적용하기 전의 기준 상태입니다.

---

## 11.2 같은 Kernel에서 Opening 전·후를 비교합니다

![[Pasted image 20260829201805.png]]

먼저 `3×3`을 비교합니다.

```
02_binary.jpg
      ↓
kernel_3_opening.jpg
```

그리고 같은 방법으로:

```
02_binary.jpg
      ↓
kernel_5_opening.jpg
```

```
02_binary.jpg
      ↓
kernel_9_opening.jpg
```

을 비교합니다.

여기서는 다음을 봅니다.

```
작은 흰색 영역이 줄었는가?

병뚜껑 외곽이 유지되는가?

Kernel이 커지면서
실제 병뚜껑의 작은 부분까지
사라지고 있지는 않은가?
```

현재 결과에서도 Kernel이 커질수록 Binary 영역이 더 강하게 정리되는 것을 볼 수 있습니다.

즉:

```
3×3
→ 변화가 비교적 작음

5×5
→ 조금 더 강하게 정리

9×9
→ 변화가 가장 큼
→ 실제 형태까지 바뀔 가능성 증가
```

입니다.

---

## 11.3 다음으로 같은 Kernel에서 Opening과 Closing을 비교합니다

여기가 중요합니다.

예를 들어 `5×5`에서는:

```
kernel_5_opening.jpg
        ↓
kernel_5_closing.jpg
```

를 비교합니다.

현재 코드의 처리 순서는 다음과 같습니다.

```
Binary
   ↓
Opening
   ↓
Closing
```

따라서 `kernel_5_closing.jpg`는 Closing만 적용한 이미지가 아니라 `Opening → Closing`까지 완료된 결과입니다.

같은 방식으로:

```
kernel_3_opening.jpg
        ↓
kernel_3_closing.jpg
```

```
kernel_9_opening.jpg
        ↓
kernel_9_closing.jpg
```

을 비교합니다.

여기서는 다음 변화를 봅니다.

```
Opening 후 남아 있던
작은 검은 구멍이 메워졌는가?

끊어져 있던 흰색 영역이
조금 연결되었는가?

Closing 때문에
원래 남아 있어야 할 검은 틈까지
사라지지는 않았는가?
```

---

## 11.4 마지막으로 Kernel 크기에 따른 최종 결과를 비교합니다

Opening과 Closing의 역할을 확인했다면 이제 최종 결과끼리 비교합니다.

```
               3×3
02_binary ──→ Opening ──→ Closing
               
               5×5
02_binary ──→ Opening ──→ Closing

               9×9
02_binary ──→ Opening ──→ Closing
```

최종적으로 비교할 파일은:

```
kernel_3_closing.jpg

kernel_5_closing.jpg

kernel_9_closing.jpg
```

입니다.

현재 실제 결과를 보면 대략 다음처럼 해석할 수 있습니다.

```
3×3
→ 원래 Binary 형태가 비교적 많이 유지됨
→ 내부 검은 틈도 일부 남아 있음

5×5
→ 일부 작은 영역이 더 정리됨
→ 병뚜껑 형태도 비교적 유지됨

9×9
→ 내부의 검은 틈이 상당히 사라짐
→ 전체 형태가 더 단순하고 매끈해짐
→ 하지만 실제 구조까지 지나치게 메운 것은 아닌지 확인 필요
```

특히 현재 이미지에서 `9×9 Closing`은 가운데와 아래쪽의 검은 영역이 많이 사라졌습니다.

이것을 단순히:

```
더 깨끗하다
→ 9×9가 가장 좋다
```

라고 판단하면 안 됩니다.

왜냐하면 그 검은 부분 중 일부가 **실제 병뚜껑 구조에서 필요한 특징**일 수도 있기 때문입니다.

---

## 11.5 어떤 Kernel을 선택해야 하나요?

Morphology의 목적은 Binary Image를 완벽하게 매끈하게 만드는 것이 아닙니다.

다음 단계인 Contour 측정이 안정적으로 이루어지면 됩니다.

```
너무 약한 Morphology

Noise와 불필요한 영역이 남음
        ↓
Contour가 여러 개 발생할 수 있음


적절한 Morphology

불필요한 작은 영역 감소
+
실제 객체 형태 유지
        ↓
Contour가 안정적으로 측정됨


너무 강한 Morphology

객체의 실제 홈·틈·작은 구조까지 변형
        ↓
Area와 형태 측정값이 왜곡될 수 있음
```

따라서 최종 판단 질문은 이것입니다.

```
Noise는 줄었는가?

병뚜껑의 실제 외곽은 유지되었는가?

필요한 구조까지 사라지지는 않았는가?

다음 Contour에서
병뚜껑을 하나의 안정적인 객체로
측정할 수 있는가?
```

> Morphology의 목표는 가장 깨끗한 Binary Image를 만드는 것이 아니라, 
> 실제 객체의 형태를 최대한 유지하면서 Contour를 안정적으로 측정할 수 있을 정도로만 
> 정리하는 것입니다.

##### 다음 Contour 실습에서는 우선 `Kernel 3×3 + Opening 1회 + Closing 1회`를 기준값으로 사용하고, 이후 실제 측정 결과를 보면서 필요하면 조정합니다.

---

# 12. Contour란 무엇인가요?

Binary Image에서 연결된 흰색 영역의 외곽선을 찾으면 Contour를 만들 수 있습니다.

![[Pasted image 20260829202454.png]]

Contour를 찾으면 눈으로 보던 상태를 숫자로 바꿀 수 있습니다.

오늘 측정할 값은 다음과 같습니다.

| 측정값 | 의미 |
|---|---|
| `Area` | 객체의 면적 |
| `Count` | 유효 객체 개수 |
| `Bounding Box` | 객체를 감싸는 사각형 |
| `Center` | 객체 중심 위치 |

병뚜껑 검사에서는 특히 다음 질문을 합니다.

```text
뚜껑이 있을 때와 없을 때

가장 큰 Contour Area가 다른가?

유효 Contour 개수가 다른가?

가장 큰 객체의 위치가 다른가?

BBox 크기가 다른가?
```

즉 먼저 측정하고, 그 다음 어떤 값이 판정에 쓸 만한지 선택합니다.

---

# 13. `19_contour_measure.py` — 실제 Contour를 찾아 측정합니다

파일을 만듭니다.

```text
src/19_contour_measure.py
```

```python

# ============================================================
# ① 필요한 라이브러리 불러오기
# - Path  : 이미지 파일 및 저장 폴더 경로 처리
# - cv2   : 이진화, Morphology, Contour 검출 및 측정
# - numpy : 이미지 배열 처리
# ============================================================

from pathlib import Path

import cv2
import numpy as np


# ============================================================
# ② 입력 이미지 및 결과 저장 경로 설정
# - cap_on.jpg : 병뚜껑이 있는 기준 ROI 이미지
# - output_dir : Contour 검사 결과 저장 폴더
# - 저장 폴더가 없으면 자동으로 생성합니다.
# ============================================================

image_path = Path(
    "samples/day03/reference/cap_on.jpg"
)

output_dir = Path(
    "outputs/day03/contour"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# ============================================================
# ③ 병뚜껑 이미지 불러오기 및 오류 확인
# - imread()로 기준 이미지를 읽습니다.
# - 이미지가 없거나 읽지 못하면 오류를 발생시킵니다.
# ============================================================

image = cv2.imread(
    str(image_path)
)

if image is None:
    raise FileNotFoundError(
        f"이미지를 읽지 못했습니다: {image_path}"
    )


# ============================================================
# ④ ROI 이미지 확인 및 Gray 변환
# - 이미 ROI로 잘린 이미지이므로 다시 Crop하지 않습니다.
# - BGR 컬러 이미지를 Gray 이미지로 변환합니다.
# ============================================================

roi = image

gray = cv2.cvtColor(
    roi,
    cv2.COLOR_BGR2GRAY,
)


# ============================================================
# ⑤ Otsu 이진화
# - Otsu 알고리즘으로 밝기 기준값을 자동 계산합니다.
# - BINARY_INV로 어두운 병뚜껑을 흰색 객체로 변환합니다.
# ============================================================

_, binary = cv2.threshold(
    gray,
    0,
    255,
    cv2.THRESH_BINARY_INV
    + cv2.THRESH_OTSU,
)


# ============================================================
# ⑥ Morphology로 이진 이미지 정리
# - 3×3 타원형 Kernel을 생성합니다.
# - Opening : 작은 흰색 노이즈 제거
# - Closing : 작은 검은색 틈이나 구멍 정리
# ============================================================

kernel = cv2.getStructuringElement(
    cv2.MORPH_ELLIPSE,
    (3, 3),
)

opened = cv2.morphologyEx(
    binary,
    cv2.MORPH_OPEN,
    kernel,
    iterations=1,
)

cleaned = cv2.morphologyEx(
    opened,
    cv2.MORPH_CLOSE,
    kernel,
    iterations=1,
)


# ============================================================
# ⑦ Contour 검출 및 검출 실패 확인
# - 정리된 이미지에서 외곽 Contour를 검출합니다.
# - 검출한 전체 Contour 개수를 출력합니다.
# - Contour가 없으면 중간 결과를 저장하고 종료합니다.
# ============================================================

contours, _ = cv2.findContours(
    cleaned.copy(),
    cv2.RETR_EXTERNAL,
    cv2.CHAIN_APPROX_SIMPLE,
)

print("전체 Contour 수:", len(contours))


if not contours:

    cv2.imwrite(
        str(output_dir / "binary.jpg"),
        binary,
    )

    cv2.imwrite(
        str(output_dir / "cleaned.jpg"),
        cleaned,
    )

    print("Contour를 찾지 못했습니다.")

    raise SystemExit


# ============================================================
# ⑧ 가장 큰 Contour 선택 및 면적·Bounding Box 계산
# - 검출된 Contour 중 면적이 가장 큰 객체를 선택합니다.
# - Area : 선택한 Contour의 실제 면적
# - BBox : 객체를 둘러싸는 사각형의 위치와 크기
# - x, y : 사각형 왼쪽 위 좌표
# - w, h : 사각형 너비와 높이
# ============================================================

largest = max(
    contours,
    key=cv2.contourArea,
)

area = float(
    cv2.contourArea(largest)
)

x, y, w, h = cv2.boundingRect(
    largest
)


# ============================================================
# ⑨ Contour 중심점 계산
# - Moments를 이용하여 객체의 중심 좌표를 계산합니다.
# - m00이 0이 아니면 중심점(cx, cy)을 구합니다.
# - 계산할 수 없는 경우 center는 None으로 유지합니다.
# ============================================================

moments = cv2.moments(
    largest
)

center = None

if moments["m00"] != 0:

    cx = int(
        moments["m10"]
        / moments["m00"]
    )

    cy = int(
        moments["m01"]
        / moments["m00"]
    )

    center = (
        cx,
        cy,
    )


# ============================================================
# ⑩ Area Ratio 계산
# - ROI 전체 면적 = 이미지 너비 × 높이
# - Area Ratio = 가장 큰 Contour 면적 / ROI 전체 면적
# - 객체가 ROI에서 차지하는 면적 비율을 구합니다.
# ============================================================

height, width = roi.shape[:2]

roi_area = float(
    width * height
)

area_ratio = (
    area / roi_area
    if roi_area > 0
    else 0.0
)


# ============================================================
# ⑪ 검사 결과를 원본 ROI 이미지에 표시
# - 초록색 : 가장 큰 Contour 윤곽선
# - 파란색 : Bounding Box
# - 빨간색 : 객체 중심점
# - 노란색 : Area Ratio 측정값
# ============================================================

result = roi.copy()


# 가장 큰 Contour를 초록색으로 표시
cv2.drawContours(
    result,
    [largest],
    -1,
    (0, 255, 0),
    2,
)


# Bounding Box를 파란색 사각형으로 표시
cv2.rectangle(
    result,
    (x, y),
    (x + w, y + h),
    (255, 0, 0),
    2,
)


# Contour 중심점을 빨간색 점으로 표시
if center is not None:

    cv2.circle(
        result,
        center,
        5,
        (0, 0, 255),
        -1,
    )


# Area Ratio를 노란색 글자로 표시
cv2.putText(
    result,
    f"AreaRatio: {area_ratio:.3f}",
    (10, 25),
    cv2.FONT_HERSHEY_SIMPLEX,
    0.65,
    (0, 255, 255),
    2,
)


# ============================================================
# ⑫ 검사 결과 이미지 저장
# - binary.jpg         : Otsu 이진화 결과
# - cleaned.jpg        : Morphology 처리 결과
# - contour_result.jpg : Contour·BBox·중심점·비율 표시
# ============================================================

cv2.imwrite(
    str(output_dir / "binary.jpg"),
    binary,
)

cv2.imwrite(
    str(output_dir / "cleaned.jpg"),
    cleaned,
)

cv2.imwrite(
    str(output_dir / "contour_result.jpg"),
    result,
)


# ============================================================
# ⑬ 측정 결과 출력
# - Largest Area : 가장 큰 Contour의 면적
# - Area Ratio   : ROI 전체 면적 대비 Contour 면적 비율
# - BBox         : 객체를 둘러싼 사각형 좌표와 크기
# - Center       : 객체의 중심 좌표
# ============================================================

print("Largest Area:", area)
print("Area Ratio:", area_ratio)
print("BBox:", (x, y, w, h))
print("Center:", center)
print("Contour 측정 완료")
```

실행합니다.

```bash
python src/19_contour_measure.py
```

### 의사코드

```
[① 필요한 라이브러리 불러오기]
Path, OpenCV, NumPy 준비
        ↓
[② 입력 이미지와 결과 저장 경로 설정]
입력: samples/day03/reference/cap_on.jpg
출력: outputs/day03/contour/
        ↓
결과 저장 폴더 생성
        ↓
[③ 병뚜껑 이미지 불러오기]
cap_on.jpg 읽기
        ↓
이미지를 읽지 못하면 오류 발생
        ↓
[④ ROI 확인 및 Gray 변환]
이미 ROI로 잘린 이미지이므로 그대로 사용
        ↓
BGR → Gray 변환
        ↓
[⑤ Otsu 이진화]
밝기 기준값 자동 계산
        ↓
어두운 병뚜껑을 흰색 객체로 변환
        ↓
[⑥ Morphology로 이미지 정리]
3×3 Kernel 생성
        ↓
Opening → 작은 노이즈 제거
        ↓
Closing → 작은 틈 정리
        ↓
[⑦ Contour 검출]
정리된 이진 이미지에서
객체의 외곽 윤곽선 찾기
        ↓
전체 Contour 개수 출력
        ↓
Contour가 없으면 결과 이미지 저장 후 종료
        ↓
[⑧ 가장 큰 Contour 선택 및 측정]
가장 큰 Contour 선택
        ↓
Contour Area 측정
        ↓
Bounding Box 좌표 계산
        ↓
[⑨ Contour 중심점 계산]
Moments 계산
        ↓
객체의 중심 좌표(cx, cy) 계산
        ↓
[⑩ Area Ratio 계산]
ROI 전체 면적 계산
        ↓
가장 큰 Contour 면적 ÷ ROI 전체 면적
        ↓
[⑪ 검사 결과를 이미지에 표시]
초록색 → Contour
파란색 → Bounding Box
빨간색 → 중심점
노란색 → Area Ratio
        ↓
[⑫ 결과 이미지 저장]
binary.jpg
cleaned.jpg
contour_result.jpg
        ↓
[⑬ 측정 결과 출력]
전체 Contour 수
Largest Area
Area Ratio
BBox
Center
Contour 측정 완료
```

---

## 실행 결과 해석

실행 결과

```
전체 Contour 수: 5
Largest Area: 27878.0
Area Ratio: 0.6610233793332385
BBox: (6, 9, 195, 163)
Center: (104, 91)
Contour 측정 완료
```

![[{8FCC086A-B3A6-4470-888E-0AA43904E064}.png]]

결과 해석

```
Binary Image
      ↓
Contour 5개 검출
      ↓
그중 가장 큰 객체 선택
      ↓
Area = 27,878
Area Ratio ≈ 66.1%
BBox = 195 × 163
Center = (104, 91)
```

---
```
전체 Contour 수: 5
```

Binary Image에서 서로 떨어져 있는 흰색 영역의 외곽선이 총 5개 발견되었다는 뜻입니다. 즉 병뚜껑 하나만 잡힌 것이 아니라, 아래쪽 병목이나 작은 흰색 영역도 각각 별도의 Contour로 잡힌 것입니다.

---
```
Largest Area: 27878.0 
Area Ratio: 0.661
```

5개의 Contour 중 가장 큰 영역의 면적은 27,878 pixel²이고, ROI 전체의 약 66.1%를 차지한다는 뜻입니다. 현재 코드에서는 이 가장 큰 Contour를 병뚜껑의 대표 객체로 사용합니다.

---
```
BBox: (6, 9, 195, 163) 
Center: (104, 91)
```

가장 큰 객체를 감싸는 사각형의 위치와 크기, 그리고 객체 중심 위치를 계산한 결과입니다.

핵심은:
Contour가 5개라는 것은 병뚜껑이 5개라는 뜻이 아니라, Binary Image에서 분리된 흰색 영역의 외곽선이 5개 검출되었다는 뜻입니다.  
그중 가장 큰 Contour를 병뚜껑 후보로 선택해서 면적과 위치를 측정한 것입니다.

---

## 13.1 왜 Area 값뿐 아니라 Area Ratio도 계산하나요?

Contour Area는 픽셀수 이고 Area Ratio는 ROI 전체 대비 비율입니다.

![[image 15.png]]

두 값이 알려주는 내용이 조금 다릅니다.

실행 결과

```
전체 Contour 수: 5
Largest Area: 27878.0
Area Ratio: 0.6610233793332385
BBox: (6, 9, 195, 163)
Center: (104, 91)
Contour 측정 완료
```

| 측정값                  | 의미                           |
| -------------------- | ---------------------------- |
| `Area = 27878`       | 객체가 실제로 몇 픽셀 크기인가            |
| `Area Ratio = 0.661` | ROI 전체에서 객체가 얼마나 큰 비율을 차지하는가 |
즉,

```
Area
→ 객체의 절대적인 크기

Area Ratio
→ ROI 안에서 객체가 차지하는 상대적인 크기
```

입니다.

이미지나 ROI 크기가 달라지면 같은 물체라도 `Area` 픽셀값은 크게 달라질 수 있기 때문에, Area Ratio도 함께 기록하면 서로 비교하기가 더 쉬워집니다.

다만 Area Ratio 역시 카메라 거리, ROI 위치, 조명, Binary 결과가 달라지면 변할 수 있으므로 절대적인 정답값은 아닙니다.

현재 결과를 한 번에 해석하면

```
전체 Contour 수 = 5
→ 흰색으로 분리된 영역의 외곽선이 5개 검출됨

Largest Area = 27878
→ 그중 가장 큰 객체의 면적

Area Ratio = 0.661
→ 가장 큰 객체가 ROI의 약 66.1%를 차지

BBox = (6, 9, 195, 163)
→ 가장 큰 객체를 감싸는 사각형의 위치와 크기

Center = (104, 91)
→ 가장 큰 객체의 중심 위치
```

중요한 것은 아직 `0.661이면 OK`라고 판단하는 것이 아닙니다.

다음 단계에서 같은 방법으로 `cap_off`도 측정하여:

```
cap_on
Area Ratio = 0.661

        VS

cap_off
Area Ratio = ?
```

를 비교하고, 실제로 두 상태를 구분하는 데 사용할 수 있는 값인지 확인하게 됩니다.

따라서 지금 단계에서는 Area와 Area Ratio를 판정 기준으로 외우는 것이 아니라, 실제 객체를 숫자로 표현한 측정값으로 이해하면 됩니다.


---

# 14. 가장 중요한 단계 — `cap_on`과 `cap_off`를 같은 코드로 측정합니다

한 장의 정상 이미지에서 Area가 나왔다고 해서 바로 Rule을 만들면 안 됩니다.

반드시 비교 대상도 같은 처리조건으로 측정해야 합니다.

```text
cap_on
+
cap_off
      ↓
같은 Gray
같은 Threshold 방식
같은 Morphology
같은 Contour 방식
      ↓
측정값 비교
```

여기서 중요한 원칙이 하나 있습니다.

> `cap_on`에는 Kernel 3×3을 쓰고 `cap_off`에는 9×9을 쓰는 식으로 이미지마다 임의로 조건을 바꾸면 비교가 무너집니다.

같은 검사에서는 가능한 한 **같은 처리 Pipeline**을 적용하고 측정값의 차이를 봅니다.

---

# 15. `20_cap_feature_compare.py` — 실제 두 이미지의 특징을 비교합니다

파일을 만듭니다.

```text
src/20_cap_feature_compare.py
```

```python

# ============================================================
# ① 필요한 라이브러리 불러오기
# - Path  : 이미지 파일과 저장 폴더 경로 처리
# - csv   : 측정된 Feature를 CSV 파일로 저장
# - cv2   : 영상 전처리와 Contour 검출 및 측정
# - numpy : 이미지 배열 처리를 위한 라이브러리
# ============================================================

from pathlib import Path
import csv

import cv2
import numpy as np


# ============================================================
# ② 검사할 기준 이미지 2장의 경로 설정
# - cap_on  : 병뚜껑이 있는 정상 이미지
# - cap_off : 병뚜껑이 없는 불량 이미지
# - 두 이미지에 동일한 검사 함수를 적용합니다.
# ============================================================

input_files = {
    "cap_on": Path(
        "samples/day03/reference/cap_on.jpg"
    ),
    "cap_off": Path(
        "samples/day03/reference/cap_off.jpg"
    ),
}


# ============================================================
# ③ 결과 저장 경로 및 Noise 제거 기준 설정
# - output_dir : 처리된 이미지 저장 폴더
# - report_path: 측정 Feature를 저장할 CSV 경로
# - MIN_CONTOUR_AREA_RATIO:
#   전체 이미지 면적의 0.1% 미만인 Contour 제외
# ============================================================

output_dir = Path(
    "outputs/day03/feature"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)

report_path = Path(
    "reports/day03_reference_features.csv"
)

MIN_CONTOUR_AREA_RATIO = 0.001


# ============================================================
# ④ Gray → Otsu 이진화
# - measure_features()는 이미지 한 장을 입력받아 검사합니다.
# - 컬러 이미지를 Gray로 변환합니다.
# - Otsu로 이진화 기준값을 자동 계산합니다.
# - 어두운 병뚜껑 영역을 흰색 객체로 변환합니다.
# ============================================================

def measure_features(image):

    gray = cv2.cvtColor(
        image,
        cv2.COLOR_BGR2GRAY,
    )

    otsu_value, binary = cv2.threshold(
        gray,
        0,
        255,
        cv2.THRESH_BINARY_INV
        + cv2.THRESH_OTSU,
    )


    # ========================================================
    # ⑤ Morphology로 이진 이미지 정리
    # - 3×3 타원형 Kernel을 사용합니다.
    # - Opening : 작은 흰색 노이즈 제거
    # - Closing : 작은 검은색 틈이나 구멍 메우기
    # ========================================================

    kernel = cv2.getStructuringElement(
        cv2.MORPH_ELLIPSE,
        (3, 3),
    )

    opened = cv2.morphologyEx(
        binary,
        cv2.MORPH_OPEN,
        kernel,
        iterations=1,
    )

    cleaned = cv2.morphologyEx(
        opened,
        cv2.MORPH_CLOSE,
        kernel,
        iterations=1,
    )


    # ========================================================
    # ⑥ Contour 검출 및 작은 객체 제외
    # - 정리된 이미지에서 외곽 Contour를 찾습니다.
    # - 이미지 전체 면적을 계산합니다.
    # - 너무 작은 Contour는 Noise로 보고 제외합니다.
    # ========================================================

    contours, _ = cv2.findContours(
        cleaned.copy(),
        cv2.RETR_EXTERNAL,
        cv2.CHAIN_APPROX_SIMPLE,
    )

    height, width = image.shape[:2]

    image_area = float(
        width * height
    )

    min_area = (
        image_area
        * MIN_CONTOUR_AREA_RATIO
    )

    valid_contours = [
        contour
        for contour in contours
        if cv2.contourArea(contour)
        >= min_area
    ]


    # ========================================================
    # ⑦ 측정 Feature 저장 구조 준비
    # - Otsu 기준값과 유효 Contour 수를 저장합니다.
    # - 면적, 중심점, 경계 접촉 여부를 저장할 공간을 만듭니다.
    # - Contour가 없다면 기본값을 반환합니다.
    # ========================================================

    features = {
        "otsu_value": float(otsu_value),
        "valid_count": len(valid_contours),
        "largest_area": 0.0,
        "largest_area_ratio": 0.0,
        "center_x_norm": "",
        "center_y_norm": "",
        "touches_border": "",
    }

    annotated = image.copy()

    if not valid_contours:

        return (
            features,
            binary,
            cleaned,
            annotated,
        )


    # ========================================================
    # ⑧ 가장 큰 Contour 선택 및 면적·중심점 측정
    # - 가장 면적이 큰 Contour를 선택합니다.
    # - Contour Area를 계산합니다.
    # - Bounding Box의 위치와 크기를 구합니다.
    # - Moments를 이용하여 객체 중심점을 계산합니다.
    # ========================================================

    largest = max(
        valid_contours,
        key=cv2.contourArea,
    )

    area = float(
        cv2.contourArea(largest)
    )

    x, y, w, h = cv2.boundingRect(
        largest
    )

    moments = cv2.moments(
        largest
    )

    cx = None
    cy = None

    if moments["m00"] != 0:

        cx = (
            moments["m10"]
            / moments["m00"]
        )

        cy = (
            moments["m01"]
            / moments["m00"]
        )


    # ========================================================
    # ⑨ 객체의 주요 Feature 계산 및 저장
    # - touches_border:
    #   Bounding Box가 이미지 경계에 닿는지 확인
    # - largest_area_ratio:
    #   가장 큰 객체 면적 ÷ 이미지 전체 면적
    # - center_x_norm / center_y_norm:
    #   중심점 좌표를 0~1 범위로 정규화
    # ========================================================

    touches_border = (
        x <= 0
        or y <= 0
        or x + w >= width
        or y + h >= height
    )

    features[
        "largest_area"
    ] = area

    features[
        "largest_area_ratio"
    ] = (
        area / image_area
        if image_area > 0
        else 0.0
    )

    if cx is not None:
        features[
            "center_x_norm"
        ] = cx / width

    if cy is not None:
        features[
            "center_y_norm"
        ] = cy / height

    features[
        "touches_border"
    ] = touches_border


    # ========================================================
    # ⑩ 검사 결과를 이미지에 표시하고 반환
    # - 초록색 : 가장 큰 Contour
    # - 파란색 : Bounding Box
    # - 빨간색 : Contour 중심점
    # - Feature와 중간 결과 이미지를 함께 반환합니다.
    # ========================================================

    cv2.drawContours(
        annotated,
        [largest],
        -1,
        (0, 255, 0),
        2,
    )

    cv2.rectangle(
        annotated,
        (x, y),
        (x + w, y + h),
        (255, 0, 0),
        2,
    )

    if (
        cx is not None
        and cy is not None
    ):

        cv2.circle(
            annotated,
            (
                int(cx),
                int(cy),
            ),
            5,
            (0, 0, 255),
            -1,
        )

    return (
        features,
        binary,
        cleaned,
        annotated,
    )


# ============================================================
# ⑪ 기준 이미지 2장을 반복해서 검사
# - rows : CSV에 저장할 측정 결과 목록
# - cap_on과 cap_off 이미지를 차례대로 불러옵니다.
# - measure_features() 함수를 호출하여 특징을 측정합니다.
# ============================================================

rows = []


for label, image_path in input_files.items():

    image = cv2.imread(
        str(image_path)
    )

    if image is None:
        raise FileNotFoundError(
            f"이미지를 읽지 못했습니다: "
            f"{image_path}"
        )

    (
        features,
        binary,
        cleaned,
        annotated,
    ) = measure_features(
        image
    )


    # ========================================================
    # ⑫ 이미지별 Feature 정리 및 결과 저장
    # - label과 파일명을 측정 Feature에 추가합니다.
    # - CSV 저장을 위해 rows 목록에 추가합니다.
    # - 각 이미지의 Binary, Cleaned, Contour 결과를 저장합니다.
    # - 측정한 Feature 값을 터미널에 출력합니다.
    # ========================================================

    row = {
        "label": label,
        "file": image_path.name,
        **features,
    }

    rows.append(
        row
    )

    cv2.imwrite(
        str(
            output_dir
            / f"{label}_binary.jpg"
        ),
        binary,
    )

    cv2.imwrite(
        str(
            output_dir
            / f"{label}_cleaned.jpg"
        ),
        cleaned,
    )

    cv2.imwrite(
        str(
            output_dir
            / f"{label}_contour.jpg"
        ),
        annotated,
    )

    print()
    print("Label:", label)

    for key, value in features.items():
        print(
            f"{key}:",
            value,
        )


# ============================================================
# ⑬ 측정 Feature를 CSV 보고서로 저장
# - reports 폴더가 없으면 자동으로 생성합니다.
# - CSV의 열 이름(fieldnames)을 지정합니다.
# - cap_on과 cap_off의 측정값을 각각 한 행으로 저장합니다.
# - utf-8-sig : Excel에서 한글을 읽기 쉽게 저장
# ============================================================

report_path.parent.mkdir(
    parents=True,
    exist_ok=True,
)

with report_path.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:

    fieldnames = [
        "label",
        "file",
        "otsu_value",
        "valid_count",
        "largest_area",
        "largest_area_ratio",
        "center_x_norm",
        "center_y_norm",
        "touches_border",
    ]

    writer = csv.DictWriter(
        file,
        fieldnames=fieldnames,
    )

    writer.writeheader()
    writer.writerows(
        rows
    )


# ============================================================
# ⑭ 최종 CSV 저장 결과 출력
# - 측정 결과를 저장한 CSV 파일 경로를 출력합니다.
# ============================================================

print()
print(
    "측정 결과 CSV:",
    report_path,
)
```

실행합니다.

```bash
python src/20_cap_feature_compare.py
```

의사코드

```
[① 필요한 라이브러리 불러오기]
Path, CSV, OpenCV, NumPy 준비
        ↓
[② 입력 이미지 경로 설정]
cap_on.jpg  → 병뚜껑 있음
cap_off.jpg → 병뚜껑 없음
        ↓
[③ 결과 저장 경로와 Noise 제거 기준 설정]
이미지 출력 폴더 생성
CSV 저장 경로 지정
작은 Contour 제외 기준 설정
        ↓
[④ Gray → Otsu 이진화 함수 작성]
입력 이미지 받기
        ↓
BGR → Gray 변환
        ↓
Otsu 이진화
어두운 객체를 흰색으로 변환
        ↓
[⑤ Morphology로 이미지 정리]
3×3 Kernel 생성
        ↓
Opening → 작은 노이즈 제거
        ↓
Closing → 작은 틈 정리
        ↓
[⑥ Contour 검출 및 작은 객체 제외]
외곽 Contour 검출
        ↓
ROI 전체 면적 계산
        ↓
너무 작은 Contour 제외
        ↓
[⑦ 측정 결과 저장 구조 준비]
Otsu 값
유효 Contour 수
가장 큰 객체의 면적
Area Ratio
중심점 좌표
경계 접촉 여부
        ↓
유효 Contour가 없으면 기본 결과 반환
        ↓
[⑧ 가장 큰 Contour 선택 및 위치 측정]
가장 큰 Contour 선택
        ↓
Contour Area 계산
        ↓
Bounding Box 계산
        ↓
Moments로 중심점 계산
        ↓
[⑨ 객체 특징값 계산]
Area Ratio 계산
        ↓
중심점 좌표를 0~1 비율로 변환
        ↓
객체가 ROI 경계에 닿는지 확인
        ↓
Feature 딕셔너리에 저장
        ↓
[⑩ 측정 결과를 이미지에 표시]
초록색 → Contour
파란색 → Bounding Box
빨간색 → 중심점
        ↓
측정 Feature와 결과 이미지 반환
        ↓
[⑪ 기준 이미지 2장 반복 검사]
cap_on.jpg 불러오기
        ↓
measure_features() 실행
        ↓
cap_off.jpg 불러오기
        ↓
measure_features() 실행
        ↓
[⑫ 이미지 결과 저장 및 측정값 출력]
각 이미지의
Binary / Cleaned / Contour 결과 저장
        ↓
측정된 Feature 출력
        ↓
[⑬ CSV 보고서 생성]
두 이미지의 Feature를 표로 정리
        ↓
reports/day03_reference_features.csv 저장
        ↓
[⑭ 최종 결과 경로 출력]
생성된 CSV 파일 위치 안내
```

---

## 15.1 실행 결과 해석

두 이미지를 동일한 Gray → Otsu → Morphology → Contour 과정으로 측정했을 때 다음과 같은 차이가 나타났습니다.

실행 결과

```
Label: cap_on
otsu_value: 111.0
valid_count: 3
largest_area: 27878.0
largest_area_ratio: 0.6610233793332385
center_x_norm: 0.4908276852608745
center_y_norm: 0.4607695434870986
touches_border: False

Label: cap_off
otsu_value: 116.0
valid_count: 1
largest_area: 19347.0
largest_area_ratio: 0.45874235310855027
center_x_norm: 0.5359471485795252
center_y_norm: 0.5507468249107779
touches_border: True
```


![[{0A9A7247-406F-41F1-8FDD-565FCE893460}.png]]

| 측정값                  |    cap_on |   cap_off | 해석                           |
| -------------------- | --------: | --------: | ---------------------------- |
| `otsu_value`         |       111 |       116 | 자동으로 선택된 이진화 기준은 큰 차이가 없음    |
| `valid_count`        |         3 |         1 | 유효 Contour 개수에 차이가 나타남       |
| `largest_area`       |    27,878 |    19,347 | 뚜껑이 있을 때 가장 큰 객체의 면적이 더 큼    |
| `largest_area_ratio` | **0.661** | **0.459** | 뚜껑 있음 약 66.1%, 없음 약 45.9%    |
| `center_x_norm`      |     0.491 |     0.536 | 좌우 중심 위치는 비교적 비슷함            |
| `center_y_norm`      |     0.461 |     0.551 | 세로 중심 위치에는 약간 차이가 있음         |
| `touches_border`     |     False |      True | cap_off의 가장 큰 객체는 ROI 경계에 닿음 |
이번 결과에서 가장 눈에 띄는 값은 `largest_area_ratio`입니다.

```
cap_on  = 0.661  → 약 66.1%
cap_off = 0.459  → 약 45.9%

차이 ≈ 0.202
     ≈ 20.2%p
```

즉 현재 두 기준 이미지에서는 병뚜껑이 있을 때 가장 큰 흰색 객체가 ROI에서 차지하는 비율이 더 크게 측정되었습니다.

`valid_count`도 `3개 vs 1개`로 차이가 있지만, 작은 영역의 분리 상태나 Morphology에 따라 달라질 수 있으므로 지금 바로 주 판정 기준으로 정하지는 않습니다.

또한 `cap_off`의 `touches_border=True`는 가장 큰 Contour가 ROI 경계까지 연결되어 있다는 뜻입니다. 따라서 병뚜껑 자체가 아닌 병목이나 주변 영역을 함께 잡았을 가능성도 결과 이미지에서 확인해야 합니다.

현재 결과에서는 `largest_area_ratio`가 cap_on과 cap_off를 구분하기 위한 가장 이해하기 쉬운 Rule 후보로 보입니다. 다만 기준 이미지가 두 장뿐이므로 아직 최종 기준은 아니며, 다음 단계에서는 두 값 사이에 임시 Candidate Threshold를 만들어 Rule의 동작을 확인합니다.

---

# 16. 실제 측정값을 비교하여 Rule 후보를 찾습니다

앞에서는 병뚜껑이 있는 이미지 `cap_on`과 병뚜껑이 없는 이미지 `cap_off`를 동일한 Pipeline으로 측정했습니다.

```
Gray
→ Otsu Binary
→ Morphology
→ Contour
→ Feature 측정
```

이제 다음 CSV에서 두 이미지의 실제 측정값을 비교합니다.

```text
reports/day03_reference_features.csv
```

실제 결과는 다음과 같습니다.

| Feature              | cap_on병뚜껑 있음 | cap_off병뚜껑 없음 |
| -------------------- | -----------: | ------------: |
| `otsu_value`         |        111.0 |         116.0 |
| `valid_count`        |            3 |             1 |
| `largest_area`       |      27878.0 |       19347.0 |
| `largest_area_ratio` |    **0.661** |     **0.459** |
| `center_x_norm`      |        0.491 |         0.536 |
| `center_y_norm`      |        0.461 |         0.551 |
| `touches_border`     |        False |          True |

이 단계의 목적은 모든 값을 Rule에 넣는 것이 아니라, 병뚜껑 있음과 없음을 가장 잘 구분하는 Feature를 찾는 것입니다.

현재 결과에서는 `largest_area_ratio`의 차이가 가장 뚜렷합니다.

```text
cap_on — 병뚜껑 있음
→ Area Ratio ≈ 0.661
→ ROI의 약 66.1%

cap_off — 병뚜껑 없음
→ Area Ratio ≈ 0.459
→ ROI의 약 45.9%
```

즉 현재 두 기준 이미지에서는:

```text
병뚜껑 있음
→ 가장 큰 Contour가 ROI의 약 66%를 차지

병뚜껑 없음
→ 가장 큰 Contour가 ROI의 약 46%를 차지
```

라는 차이가 나타났습니다.

```
병뚜껑 있음  0.661
                  ↑
              큰 차이
                  ↓
병뚜껑 없음  0.459
```

`valid_count`도 다음과 같이 차이가 있습니다.

```
cap_on  = 3
cap_off = 1
```

하지만 Contour 개수는 작은 Noise나 Morphology 결과에 따라 쉽게 달라질 수 있으므로 주 Rule보다는 보조 Feature 후보로 봅니다.

`touches_border`도:

```
cap_on  = False
cap_off = True
```

로 차이가 있습니다.

`cap_off=True`는 병뚜껑이 없는 이미지에서 가장 큰 Contour가 ROI 경계까지 연결되었다는 뜻입니다. 따라서 결과 이미지를 보면서 병목이나 주변 영역이 큰 객체로 잡힌 것은 아닌지 확인하는 보조 정보로 사용합니다.

### 현재 결과에서의 판단

```
가장 유력한 Rule 후보
→ largest_area_ratio

보조적으로 확인
→ valid_count
→ center
→ touches_border
```

현재 두 기준값의 차이는:

```
0.661 - 0.459
≈ 0.202
```

즉 약 **20.2%p 차이**가 있습니다.

다만 아직 `cap_on` 1장과 `cap_off` 1장만 측정한 결과이므로:

```
0.661 = 무조건 OK
0.459 = 무조건 NG
```

처럼 최종 기준으로 확정하면 안 됩니다.

> 현재 단계에서는 `largest_area_ratio`가 병뚜껑 있음/없음을 구분할 가능성이 높은 Feature라는 것을 찾은 것입니다.

다음 단계에서는 두 값 사이에 후보 기준값(Candidate Threshold)을 만들어 `OK / NG` 판정이 어떻게 이루어지는지 확인합니다.

```
병뚜껑 없음
0.459
      ↓
   후보 기준
약 0.56
      ↑
0.661
병뚜껑 있음
```

여기서 약 `0.56`은 현재 두 기준 이미지의 측정값을 바탕으로 임시로 정한 값입니다.

```
Area Ratio가 약 0.56보다 크면
→ 병뚜껑 있음 → OK

Area Ratio가 약 0.56보다 작으면
→ 병뚜껑 없음 → NG
```

아직 기준 이미지가 두 장뿐이므로 이 값을 최종 기준으로 확정하지는 않습니다. 이후 더 많은 이미지를 촬영하고 측정하여 이 기준이 실제로 안정적으로 동작하는지 확인합니다.

![[Pasted image 20260829223033.png]]

위의 이미지에서 말하는 교육용의 뜻은 두 값 사이에 후보 기준값을 말합니다.

---

## 16.1 측정값의 차이가 작다면 어떻게 하나요?

이번에는 다음처럼 차이가 뚜렷했습니다.

```
병뚜껑 있음  → 0.661
병뚜껑 없음  → 0.459
```

그래서 `largest_area_ratio`를 Rule에 사용할 후보로 선택할 수 있습니다.

하지만 두 값이 거의 비슷하다면 이 값으로는 병뚜껑 있음과 없음을 구분하기 어렵다는 뜻입니다.

```
cap_on ≈ cap_off
        ↓
Area Ratio만으로 구분하기 어려움
        ↓
Count / Center 등 다른 Feature 확인
또는
ROI / Binary / Contour 결과 다시 확인
```

> 두 상태에서 차이가 잘 나타나는 Feature를 찾아 Rule에 사용하는 것이 핵심입니다.

---

# 17. 후보 기준값은 어떻게 정하나요?

앞에서 실제 측정 결과가 다음과 같이 나왔습니다.

```
병뚜껑 있음
Area Ratio = 0.661

병뚜껑 없음
Area Ratio = 0.459
```

이제 두 상태를 구분하기 위해 두 값 사이에 임시 기준값을 하나 정합니다.

예를 들어 두 값의 중간 정도인:

```
0.459
  ↓
약 0.56  ← 후보 기준값
  ↑
0.661
```

을 사용할 수 있습니다.

그러면 Rule은 다음처럼 만들 수 있습니다.

```
Area Ratio >= 0.56
→ 병뚜껑 있음
→ OK

Area Ratio < 0.56
→ 병뚜껑 없음
→ NG
```

여기서 `0.56`은 처음부터 임의로 정한 숫자가 아닙니다.

```
실제 cap_on 측정값
        +
실제 cap_off 측정값
        ↓
두 값의 차이 확인
        ↓
그 사이에서 후보 기준값 결정
```

한 것입니다.

다만 현재는 이미지가 각각 1장뿐이므로 `0.56`을 최종 기준으로 확정하지 않습니다. 3일차에서는 이 기준으로 Rule이 어떻게 동작하는지 확인하고, 이후 더 많은 이미지를 측정하면서 기준값을 다시 검토합니다.

---

# 18. `21_cap_rule.py` — 실제 Reference 두 장으로 Feature 후보 Rule을 만듭니다

앞에서 여러 Feature를 비교한 결과, 현재 이미지에서는 `largest_area_ratio`가 병뚜껑 있음과 없음을 가장 잘 구분하는 값으로 나타났습니다.

이제 이 두 값 사이에 후보 기준값(Candidate Threshold)을 만들고, 그 기준을 이용해 실제로 `OK / NG` 판정이 어떻게 이루어지는지 확인합니다.

현재 코드는 두 값의 중간값을 자동으로 계산합니다.

```
0.661
  ↓
약 0.560  ← 후보 기준값
  ↑
0.459
```

따라서 현재 측정 결과에서는 다음과 같은 Rule이 만들어집니다.

```
Area Ratio >= 약 0.560
→ 병뚜껑 있음
→ OK

Area Ratio < 약 0.560
→ 병뚜껑 없음
→ NG
```

이 단계에서 중요한 것은 `0.560`을 최종 기준으로 외우는 것이 아닙니다.

> 실제 이미지에서 측정한 Feature 값을 이용해 기준값을 만들고, 그 기준값을 `if` 조건으로 연결하여 OK / NG를 판정하는 과정을 확인하는 것이 목적입니다.


파일 생성:

```text
src/21_cap_rule.py
```

```python

# ============================================================
# ① 필요한 라이브러리 불러오기
# - Path  : 기준 이미지 및 결과 저장 경로 처리
# - cv2   : 영상 전처리, Contour 검출 및 판정 결과 표시
# - numpy : 이미지 배열 처리를 위한 라이브러리
# ============================================================

from pathlib import Path

import cv2
import numpy as np


# ============================================================
# ② 기준 이미지와 결과 저장 경로 설정
# - cap_on.jpg  : 병뚜껑이 있는 기준 이미지
# - cap_off.jpg : 병뚜껑이 없는 기준 이미지
# - OUTPUT_DIR  : 검사 결과 이미지 저장 폴더
# - MIN_CONTOUR_AREA_RATIO : 작은 객체 제외 기준(0.1%)
# ============================================================

CAP_ON_PATH = Path(
    "samples/day03/reference/cap_on.jpg"
)

CAP_OFF_PATH = Path(
    "samples/day03/reference/cap_off.jpg"
)

OUTPUT_DIR = Path(
    "outputs/day03/rule"
)

OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)

MIN_CONTOUR_AREA_RATIO = 0.001


# ============================================================
# ③ 이미지 불러오기 및 오류 확인
# - 입력받은 경로에서 이미지를 읽습니다.
# - 이미지가 없거나 읽을 수 없으면 오류를 발생시킵니다.
# ============================================================

def measure_area_ratio(
    image_path,
):

    image = cv2.imread(
        str(image_path)
    )

    if image is None:
        raise FileNotFoundError(
            f"이미지를 읽지 못했습니다: "
            f"{image_path}"
        )


    # ========================================================
    # ④ Gray → Otsu 이진화
    # - BGR 컬러 이미지를 Gray로 변환합니다.
    # - Otsu로 이진화 밝기 기준값을 자동 계산합니다.
    # - BINARY_INV로 어두운 병뚜껑을 흰색 객체로 변환합니다.
    # ========================================================

    gray = cv2.cvtColor(
        image,
        cv2.COLOR_BGR2GRAY,
    )

    _, binary = cv2.threshold(
        gray,
        0,
        255,
        cv2.THRESH_BINARY_INV
        + cv2.THRESH_OTSU,
    )


    # ========================================================
    # ⑤ Morphology로 이미지 정리
    # - 3×3 타원형 Kernel을 사용합니다.
    # - Opening : 작은 흰색 노이즈 제거
    # - Closing : 작은 검은색 틈이나 구멍 정리
    # ========================================================

    kernel = cv2.getStructuringElement(
        cv2.MORPH_ELLIPSE,
        (3, 3),
    )

    opened = cv2.morphologyEx(
        binary,
        cv2.MORPH_OPEN,
        kernel,
        iterations=1,
    )

    cleaned = cv2.morphologyEx(
        opened,
        cv2.MORPH_CLOSE,
        kernel,
        iterations=1,
    )


    # ========================================================
    # ⑥ Contour 검출 및 Area Ratio 측정
    # - 정리된 이미지에서 외곽 Contour를 찾습니다.
    # - ROI 전체 면적의 0.1%보다 작은 객체는 제외합니다.
    # - 가장 큰 Contour의 면적을 계산합니다.
    # - Area Ratio = Contour 면적 / ROI 전체 면적
    # - Contour가 없으면 0.0과 None을 반환합니다.
    # ========================================================

    contours, _ = cv2.findContours(
        cleaned.copy(),
        cv2.RETR_EXTERNAL,
        cv2.CHAIN_APPROX_SIMPLE,
    )

    height, width = image.shape[:2]

    image_area = float(
        width * height
    )

    min_area = (
        image_area
        * MIN_CONTOUR_AREA_RATIO
    )

    valid = [
        contour
        for contour in contours
        if cv2.contourArea(contour)
        >= min_area
    ]

    if not valid:

        return (
            0.0,
            image,
            None,
        )

    largest = max(
        valid,
        key=cv2.contourArea,
    )

    largest_area = float(
        cv2.contourArea(largest)
    )

    area_ratio = (
        largest_area / image_area
        if image_area > 0
        else 0.0
    )

    return (
        area_ratio,
        image,
        largest,
    )


# ============================================================
# ⑦ 정상·불량 기준 이미지의 Area Ratio 측정
# - 앞에서 만든 measure_area_ratio() 함수를 호출합니다.
# - cap_on과 cap_off의 면적 비율을 각각 계산합니다.
# - 측정값은 다음 단계의 판정 기준값 생성에 사용합니다.
# ============================================================

cap_on_ratio, _, _ = (
    measure_area_ratio(
        CAP_ON_PATH
    )
)

cap_off_ratio, _, _ = (
    measure_area_ratio(
        CAP_OFF_PATH
    )
)


# ============================================================
# ⑧ OK / NG 판정 기준값 생성
# - 두 기준 이미지의 Area Ratio 중간값을 계산합니다.
# - 이 값을 candidate_threshold로 저장합니다.
# - 뚜껑이 있을 때 면적 비율이 더 큰지도 확인합니다.
# ============================================================

candidate_threshold = (
    cap_on_ratio
    + cap_off_ratio
) / 2.0


cap_present_when_greater = (
    cap_on_ratio
    > cap_off_ratio
)


# ============================================================
# ⑨ 병뚜껑 유무 판정 함수 작성
# - 새로 측정한 Area Ratio를 기준값과 비교합니다.
# - 뚜껑이 있을 때 비율이 큰 경우와 작은 경우를 구분합니다.
# - 병뚜껑이 있다고 판정하면 OK를 반환합니다.
# - 병뚜껑이 없다고 판정하면 NG와 사유를 반환합니다.
# ============================================================

def judge_cap(
    area_ratio,
):

    if cap_present_when_greater:

        cap_present = (
            area_ratio
            >= candidate_threshold
        )

    else:

        cap_present = (
            area_ratio
            <= candidate_threshold
        )

    if cap_present:

        return (
            "OK",
            "",
        )

    return (
        "NG",
        "NG_CAP_AREA",
    )


# ============================================================
# ⑩ 기준값 측정 결과 출력
# - cap_on ratio      : 병뚜껑이 있는 이미지의 면적 비율
# - cap_off ratio     : 병뚜껑이 없는 이미지의 면적 비율
# - reference gap     : 두 기준 측정값의 차이
# - candidate threshold : OK / NG 구분을 위한 후보 기준값
# - GREATER / LESS    : 뚜껑이 있을 때의 비율 방향
# ============================================================

print(
    "cap_on ratio :",
    cap_on_ratio,
)

print(
    "cap_off ratio:",
    cap_off_ratio,
)

print(
    "reference gap:",
    abs(
        cap_on_ratio
        - cap_off_ratio
    ),
)

print(
    "candidate threshold:",
    candidate_threshold,
)

print(
    "cap present when:",
    "GREATER"
    if cap_present_when_greater
    else "LESS",
)


# ============================================================
# ⑪ 정상·불량 기준 이미지에 판정 규칙 적용
# - cap_on과 cap_off 이미지를 차례대로 검사합니다.
# - measure_area_ratio()로 면적 비율을 다시 측정합니다.
# - judge_cap()으로 OK / NG를 판정합니다.
# ============================================================

for label, image_path in [
    (
        "cap_on",
        CAP_ON_PATH,
    ),
    (
        "cap_off",
        CAP_OFF_PATH,
    ),
]:

    area_ratio, image, contour = (
        measure_area_ratio(
            image_path
        )
    )

    decision, reason = judge_cap(
        area_ratio
    )


    # ========================================================
    # ⑫ 판정 결과를 이미지에 표시하고 저장
    # - 가장 큰 Contour를 초록색으로 표시합니다.
    # - OK는 초록색, NG는 빨간색 글자로 표시합니다.
    # - Area Ratio도 이미지에 함께 표시합니다.
    # - cap_on_rule.jpg / cap_off_rule.jpg로 저장합니다.
    # ========================================================

    result = image.copy()

    if contour is not None:

        cv2.drawContours(
            result,
            [contour],
            -1,
            (0, 255, 0),
            2,
        )

    text = (
        f"{decision} "
        f"ratio={area_ratio:.3f}"
    )

    cv2.putText(
        result,
        text,
        (10, 30),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.65,
        (
            0,
            200,
            0,
        )
        if decision == "OK"
        else (
            0,
            0,
            255,
        ),
        2,
    )

    output_path = (
        OUTPUT_DIR
        / f"{label}_rule.jpg"
    )

    cv2.imwrite(
        str(output_path),
        result,
    )


    # ========================================================
    # ⑬ 최종 판정 결과 출력
    # - file       : 검사한 이미지 파일명
    # - area_ratio : 측정한 객체 면적 비율
    # - decision   : OK 또는 NG
    # - reason     : NG 판정 사유
    # ========================================================

    print()
    print("file:", image_path.name)
    print("area_ratio:", area_ratio)
    print("decision:", decision)
    print("reason:", reason)

```

실행합니다.

```bash
python src/21_cap_rule.py
```

## 의사코드

```
[① 필요한 라이브러리 불러오기]
Path, OpenCV, NumPy 준비
        ↓
[② 기준 이미지와 결과 저장 경로 설정]
cap_on.jpg  → 병뚜껑 있음
cap_off.jpg → 병뚜껑 없음
        ↓
결과 저장 폴더 생성
작은 Contour 제외 기준 설정
        ↓
[③ 이미지 불러오기 및 오류 확인]
이미지 파일 읽기
        ↓
파일이 없거나 읽을 수 없으면 오류 발생
        ↓
[④ Gray → Otsu 이진화]
컬러 이미지를 Gray로 변환
        ↓
Otsu로 밝기 기준값 자동 계산
        ↓
어두운 병뚜껑 영역을 흰색으로 변환
        ↓
[⑤ Morphology로 이미지 정리]
3×3 Kernel 생성
        ↓
Opening → 작은 노이즈 제거
        ↓
Closing → 작은 틈 정리
        ↓
[⑥ Contour 검출 및 Area Ratio 측정]
외곽 Contour 검출
        ↓
너무 작은 Contour 제외
        ↓
가장 큰 Contour 선택
        ↓
Contour 면적 ÷ 이미지 전체 면적
        ↓
Area Ratio 반환
        ↓
[⑦ 기준 이미지 2장의 Area Ratio 측정]
cap_on.jpg  → cap_on_ratio
cap_off.jpg → cap_off_ratio
        ↓
[⑧ OK / NG 판정 기준값 생성]
두 Area Ratio의 중간값 계산
        ↓
candidate_threshold 생성
        ↓
병뚜껑이 있을 때 비율이
더 큰지 또는 작은지 확인
        ↓
[⑨ 병뚜껑 판정 함수 작성]
새로운 Area Ratio 입력
        ↓
기준값과 비교
        ↓
뚜껑 있음 → OK
뚜껑 없음 → NG
        ↓
[⑩ 기준값 측정 결과 출력]
cap_on ratio
cap_off ratio
두 측정값 차이
판정 기준값
비교 방향 출력
        ↓
[⑪ 기준 이미지 2장에 판정 규칙 적용]
cap_on.jpg 검사
        ↓
Area Ratio 계산 → OK / NG 판정
        ↓
cap_off.jpg 검사
        ↓
Area Ratio 계산 → OK / NG 판정
        ↓
[⑫ 검사 결과 이미지 생성 및 저장]
검출된 Contour를 초록색으로 표시
        ↓
OK / NG와 Area Ratio를 이미지에 표시
        ↓
cap_on_rule.jpg
cap_off_rule.jpg 저장
        ↓
[⑬ 최종 판정 결과 출력]
검사 파일명
Area Ratio
OK / NG 판정값
판정 사유 출력
```

---

### [[3일차 21코드 Jupyter로 코드 리뷰방법]] 바로가기

---

## 실행 결과

```
cap_on ratio : 0.6610233793332385
cap_off ratio: 0.45874235310855027
reference gap: 0.20228102622468824
candidate threshold: 0.5598828662208943
cap present when: GREATER

file: cap_on.jpg
area_ratio: 0.6610233793332385
decision: OK
reason: 

file: cap_off.jpg
area_ratio: 0.45874235310855027
decision: NG
reason: NG_CAP_AREA
```

## 결과 해석

```
병뚜껑 있음
cap_on = 0.661

병뚜껑 없음
cap_off = 0.459

두 값의 차이
≈ 0.202

후보 기준값
≈ 0.560
```

`cap present when: GREATER`는 **Area Ratio가 기준값보다 큰 쪽을 병뚜껑 있음으로 판단한다**는 뜻입니다.

따라서 실제 판정은:

```
cap_on
0.661 >= 0.560
→ OK

cap_off
0.459 < 0.560
→ NG
→ NG_CAP_AREA
```

로 정상적으로 동작했습니다.

---

# 19. 이 Rule 결과를 “성능”이라고 부르면 안 됩니다

`21_cap_rule.py`에서는 다음 두 기준 이미지의 측정값으로 후보 기준값을 만들었습니다.

```text
cap_on.jpg
cap_off.jpg
```

그리고 기준값을 만드는 데 사용한 같은 두 이미지에 다시 Rule을 적용했습니다.

```
cap_on
→ OK

cap_off
→ NG
```

두 장 모두 올바르게 판정되었지만 이것을:

```text
정확도 100%
```

라고 말하면 안 됩니다.

왜냐하면 기준값을 만들 때 사용한 이미지로 다시 확인한 결과이기 때문입니다.

```text
측정값
        ↓
기준값
        ↓
if 조건
        ↓
OK / NG
```

라는 Rule의 구조를 확인한 것입니다.

즉 지금은 Rule의 동작을 확인한 단계이며, 이 Rule이 실제로 잘 작동하는지는 새로운 이미지를 추가로 넣어 확인해야 합니다.

다음 단계에서는 새로 촬영한 `cap_on`, `cap_off` 이미지에 **같은 Rule을 그대로 적용**하여 맞는 경우와 틀리는 경우를 확인합니다.

---

19번까지는 제공된 이미지와 기존 촬영 결과를 이용하여 **Rule이 어떻게 동작하는지 확인**했습니다.

이제부터는 촬영 조건을 맞춘 새로운 이미지를 사용하여 같은 Rule이 여러 데이터에서도 유지되는지 확인합니다.

이번 실습에서는 스마트폰 IP Webcam의 입력 해상도를 다음과 같이 통일했습니다.

```
640 × 480
```

그리고 같은 병을 이용하여 다음 두 상태를 각각 3장씩 다시 촬영했습니다.

```
병뚜껑 있음
→ 3장

병뚜껑 없음
→ 3장

총 6장
```

이번 데이터부터는 **같은 촬영 조건에서 확보한 이미지끼리 비교**합니다.

---

# 20 1일차의 실시간 ROI 프로그램을 재사용합니다

1일차에 작성한 스마트폰 실시간 ROI 프로그램을 다시 사용했습니다.

```
python src/08_phone_camera_roi.py
```

PC의 `Phone Camera` 창에서 병뚜껑 영역을 ROI로 선택하고 다음 키를 사용합니다.

```
1 → 병뚜껑 있음 저장
2 → 병뚜껑 없음 저장
```

이번에 새로 확보한 결과는 다음과 같습니다.

```
outputs/
├── live_roi03/
│   ├── cap_on.jpg
│   └── cap_off.jpg
│
├── live_roi03-1/
│   ├── cap_on.jpg
│   └── cap_off.jpg
│
└── live_roi03-2/
    ├── cap_on.jpg
    └── cap_off.jpg
```

![[Pasted image 20260829235819.png]]

세 번의 촬영은 모두 다음 조건을 기준으로 진행했습니다.

```
IP Webcam 입력 해상도
→ 640 × 480

같은 병 사용

카메라 위치
→ 가능한 한 고정

촬영 거리
→ 가능한 한 일정

병 위치
→ 비슷한 위치 유지

조명
→ 가능한 한 동일
```

따라서 이번 실습에서는 다음 6장을 하나의 비교 데이터로 사용합니다.

|촬영|병뚜껑 있음|병뚜껑 없음|
|---|---|---|
|1차|`live_roi03/cap_on.jpg`|`live_roi03/cap_off.jpg`|
|2차|`live_roi03-1/cap_on.jpg`|`live_roi03-1/cap_off.jpg`|
|3차|`live_roi03-2/cap_on.jpg`|`live_roi03-2/cap_off.jpg`|

---

## 20.1 왜 촬영 해상도를 통일했을까요?

앞으로 Rule 검사에서는 이미지에서 숫자를 측정하게 됩니다.

예를 들면 다음과 같습니다.

```
Threshold 후 픽셀 수
Contour 면적
객체의 Width · Height
위치
```

이러한 값은 영상의 해상도와 객체가 차지하는 픽셀 크기에 영향을 받을 수 있습니다.

따라서 서로 다른 해상도의 데이터를 그대로 섞으면 다음과 같은 문제가 생길 수 있습니다.

```
같은 병뚜껑

1920 × 1080 영상
→ 많은 픽셀로 표현

640 × 480 영상
→ 더 적은 픽셀로 표현

        ↓

면적과 크기 값이 달라질 수 있음
```

그래서 이번 실습부터는 입력 조건을 다음과 같이 고정합니다.

```
640 × 480
```

> Rule 기반 검사에서는 기준값을 만들기 전에 촬영 해상도와 검사환경을 가능한 한 일정하게 맞추는 것이 중요합니다.

기존 1920×1080 조건에서 촬영한 데이터는 삭제하지 않고 **이전 실습 확인용 자료**로 보관하며, 이번 Rule 비교에서는 사용하지 않습니다.

---

## 20.2 Day03 실습 데이터로 정리합니다

새로 확보한 640×480 조건의 6장을 하나의 실습 데이터로 정리합니다.

폴더를 만듭니다.

```bash
mkdir -p samples/day03/cap_on
mkdir -p samples/day03/cap_off
```

첫 번째 촬영 결과를 복사합니다.

```bash
cp outputs/live_roi03/cap_on.jpg \
   samples/day03/cap_on/cap_on_001.jpg

cp outputs/live_roi03/cap_off.jpg \
   samples/day03/cap_off/cap_off_001.jpg
```

두 번째 촬영 결과를 복사합니다.

```bash
cp outputs/live_roi03-1/cap_on.jpg \
   samples/day03/cap_on/cap_on_002.jpg

cp outputs/live_roi03-1/cap_off.jpg \
   samples/day03/cap_off/cap_off_002.jpg
```

세 번째 촬영 결과를 복사합니다.

```bash
cp outputs/live_roi03-2/cap_on.jpg \
   samples/day03/cap_on/cap_on_003.jpg

cp outputs/live_roi03-2/cap_off.jpg \
   samples/day03/cap_off/cap_off_003.jpg
```

최종적으로 다음과 같이 정리됩니다.

```
samples/day03/
│
├── cap_on/
│   ├── cap_on_001.jpg
│   ├── cap_on_002.jpg
│   └── cap_on_003.jpg
│
└── cap_off/
    ├── cap_off_001.jpg
    ├── cap_off_002.jpg
    └── cap_off_003.jpg
```

확인합니다.

```bash
find samples/day03 -type f | sort
```

---

## 20.3 이번 실습의 목적

한 장에만 잘 맞는 Rule은 실제 검사에 사용하기 어렵습니다.

따라서 이제부터는 다음 순서로 확인합니다.

```
640×480 조건으로 데이터 다시 촬영
        ↓
cap_on 3장
cap_off 3장
        ↓
같은 전처리 적용
        ↓
같은 특징 측정
        ↓
각 이미지의 측정값 비교
        ↓
cap_on과 cap_off 사이에
구분 가능한 차이가 있는지 확인
        ↓
Rule 기준값 후보 결정
```

20번부터는 한 장의 결과가 아니라 여러 장의 측정값을 비교하는 단계로 넘어갑니다.

---

# 21. `22_cap_batch_test.py` — 여러 실제 이미지에 같은 Rule을 적용합니다

파일:

```text
src/22_cap_batch_test.py
```

```python

# ============================================================
# ① 필요한 라이브러리 불러오기
# - Path : 이미지 및 결과 저장 경로 처리
# - csv  : 검사 결과를 CSV 파일로 저장
# - cv2  : 이미지 전처리, Contour 검출 및 판정 결과 표시
# ============================================================

from pathlib import Path
import csv

import cv2


# ============================================================
# ② 이미지 경로, 결과 저장 폴더 및 Noise 제거 기준 설정
# - cap_on / cap_off 폴더 지정
# - Reference 이미지 2장 지정
# - 검사 결과 이미지 및 CSV 저장 경로 지정
# - ROI 면적의 0.1%보다 작은 Contour는 제외
# ============================================================

CAP_ON_DIR = Path(
    "samples/day03/cap_on"
)

CAP_OFF_DIR = Path(
    "samples/day03/cap_off"
)

REFERENCE_ON = (
    CAP_ON_DIR / "cap_on_001.jpg"
)

REFERENCE_OFF = (
    CAP_OFF_DIR / "cap_off_001.jpg"
)

OUTPUT_DIR = Path(
    "outputs/day03/batch"
)

REPORT_PATH = Path(
    "reports/day03_inspection_results.csv"
)

OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)

REPORT_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)

MIN_CONTOUR_AREA_RATIO = 0.001


# ============================================================
# ③ 검사할 이미지 불러오기
# - measure()는 이미지 한 장의 특징을 측정하는 함수입니다.
# - 이미지를 읽지 못하면 None을 반환합니다.
# ============================================================

def measure(image_path):

    image = cv2.imread(
        str(image_path)
    )

    if image is None:
        return None


    # ========================================================
    # ④ Gray → Otsu 이진화
    # - BGR 이미지를 Gray로 변환합니다.
    # - Otsu로 밝기 기준값을 자동 계산합니다.
    # - BINARY_INV로 어두운 객체를 흰색으로 변환합니다.
    # ========================================================

    gray = cv2.cvtColor(
        image,
        cv2.COLOR_BGR2GRAY,
    )

    otsu_value, binary = cv2.threshold(
        gray,
        0,
        255,
        cv2.THRESH_BINARY_INV
        + cv2.THRESH_OTSU,
    )


    # ========================================================
    # ⑤ Morphology로 이미지 정리
    # - 3×3 타원형 Kernel을 사용합니다.
    # - Opening : 작은 흰색 노이즈 제거
    # - Closing : 작은 검은색 틈이나 구멍 정리
    # ========================================================

    kernel = cv2.getStructuringElement(
        cv2.MORPH_ELLIPSE,
        (3, 3),
    )

    opened = cv2.morphologyEx(
        binary,
        cv2.MORPH_OPEN,
        kernel,
        iterations=1,
    )

    cleaned = cv2.morphologyEx(
        opened,
        cv2.MORPH_CLOSE,
        kernel,
        iterations=1,
    )


    # ========================================================
    # ⑥ Contour 검출 및 작은 객체 제외
    # - 정리된 이미지에서 외곽 Contour를 검출합니다.
    # - 이미지 전체 면적을 계산합니다.
    # - 너무 작은 Contour는 Noise로 보고 제외합니다.
    # ========================================================

    contours, _ = cv2.findContours(
        cleaned.copy(),
        cv2.RETR_EXTERNAL,
        cv2.CHAIN_APPROX_SIMPLE,
    )

    height, width = image.shape[:2]

    image_area = float(
        width * height
    )

    min_area = (
        image_area
        * MIN_CONTOUR_AREA_RATIO
    )

    valid_contours = [
        contour
        for contour in contours
        if cv2.contourArea(contour)
        >= min_area
    ]


    # ========================================================
    # ⑦ 가장 큰 Contour의 Feature 측정 및 반환
    # - 가장 큰 Contour를 선택합니다.
    # - Area Ratio = Contour 면적 / ROI 전체 면적
    # - Moments를 이용하여 객체 중심점을 계산합니다.
    # - 중심점은 0~1 비율 좌표로 저장합니다.
    # - 측정 결과를 딕셔너리로 반환합니다.
    # ========================================================

    largest_contour = None
    largest_area_ratio = 0.0

    center_x_norm = ""
    center_y_norm = ""

    if valid_contours:

        largest_contour = max(
            valid_contours,
            key=cv2.contourArea,
        )

        largest_area = float(
            cv2.contourArea(
                largest_contour
            )
        )

        largest_area_ratio = (
            largest_area / image_area
            if image_area > 0
            else 0.0
        )

        moments = cv2.moments(
            largest_contour
        )

        if moments["m00"] != 0:

            cx = (
                moments["m10"]
                / moments["m00"]
            )

            cy = (
                moments["m01"]
                / moments["m00"]
            )

            center_x_norm = (
                cx / width
            )

            center_y_norm = (
                cy / height
            )

    return {
        "image": image,
        "otsu_value": float(
            otsu_value
        ),
        "valid_count": len(
            valid_contours
        ),
        "largest_contour": (
            largest_contour
        ),
        "largest_area_ratio": (
            largest_area_ratio
        ),
        "center_x_norm": (
            center_x_norm
        ),
        "center_y_norm": (
            center_y_norm
        ),
    }


# ============================================================
# ⑧ Reference 이미지 2장으로 판정 기준값 생성
# - cap_on_001.jpg와 cap_off_001.jpg를 측정합니다.
# - 이미지가 없으면 오류를 발생시킵니다.
# - 두 Area Ratio의 중간값을 판정 기준값으로 사용합니다.
# - cap_on일 때 Area Ratio가 더 큰지도 확인합니다.
# ============================================================

reference_on = measure(
    REFERENCE_ON
)

reference_off = measure(
    REFERENCE_OFF
)

if (
    reference_on is None
    or reference_off is None
):

    raise RuntimeError(
        "Reference 이미지를 확인하세요."
    )

on_ratio = (
    reference_on[
        "largest_area_ratio"
    ]
)

off_ratio = (
    reference_off[
        "largest_area_ratio"
    ]
)

candidate_threshold = (
    on_ratio + off_ratio
) / 2.0

cap_present_when_greater = (
    on_ratio > off_ratio
)

print(
    "Reference ON :",
    round(on_ratio, 4),
)

print(
    "Reference OFF:",
    round(off_ratio, 4),
)

print(
    "Threshold    :",
    round(candidate_threshold, 4),
)


# ============================================================
# ⑨ OK / NG 판정 함수 작성
# - 새로운 이미지의 Area Ratio를 입력받습니다.
# - Reference에서 만든 기준값과 비교합니다.
# - 뚜껑이 있다고 판정하면 cap_on, OK를 반환합니다.
# - 뚜껑이 없다고 판정하면 cap_off, NG를 반환합니다.
# ============================================================

def judge(area_ratio):

    if cap_present_when_greater:

        cap_present = (
            area_ratio
            >= candidate_threshold
        )

    else:

        cap_present = (
            area_ratio
            <= candidate_threshold
        )

    if cap_present:

        return (
            "cap_on",
            "OK",
            "",
        )

    return (
        "cap_off",
        "NG",
        "NG_CAP_AREA",
    )


# ============================================================
# ⑩ Test 이미지 4장 및 평가 변수 준비
# - Reference로 사용한 001 이미지는 제외합니다.
# - cap_on 2장, cap_off 2장을 검사합니다.
# - expected는 각 이미지의 실제 정답입니다.
# - total   : 실제로 검사한 이미지 수
# - correct : 정답과 일치한 이미지 수
# ============================================================

TEST_IMAGES = [
    (
        "cap_on",
        CAP_ON_DIR
        / "cap_on_002.jpg",
    ),
    (
        "cap_on",
        CAP_ON_DIR
        / "cap_on_003.jpg",
    ),
    (
        "cap_off",
        CAP_OFF_DIR
        / "cap_off_002.jpg",
    ),
    (
        "cap_off",
        CAP_OFF_DIR
        / "cap_off_003.jpg",
    ),
]

rows = []

total = 0
correct = 0


# ============================================================
# ⑪ Test 이미지에 동일한 검사 규칙 적용
# - 새로운 이미지를 한 장씩 불러옵니다.
# - measure()를 이용하여 Area Ratio를 측정합니다.
# - judge()로 cap_on / cap_off 및 OK / NG를 판정합니다.
# - 읽지 못한 이미지는 검사에서 제외합니다.
# ============================================================

for expected, image_path in TEST_IMAGES:

    measured = measure(
        image_path
    )

    if measured is None:

        print(
            "읽기 실패:",
            image_path,
        )

        continue

    predicted, decision, reason = (
        judge(
            measured[
                "largest_area_ratio"
            ]
        )
    )


    # ========================================================
    # ⑫ 예측 결과와 실제 정답 비교
    # - expected  : 실제 정답(cap_on / cap_off)
    # - predicted : 코드가 예측한 결과
    # - is_correct : 두 값의 일치 여부
    # - 전체 검사 수와 정답 수를 누적합니다.
    # ========================================================

    is_correct = (
        predicted == expected
    )

    total += 1

    if is_correct:
        correct += 1


    # ========================================================
    # ⑬ 검사 결과 이미지를 생성하고 저장
    # - 검출된 Contour를 초록색으로 표시합니다.
    # - 예측 결과와 Area Ratio를 이미지에 표시합니다.
    # - 정답과 일치하면 초록색, 불일치하면 빨간색입니다.
    # - 이미지별 결과를 JPG 파일로 저장합니다.
    # ========================================================

    result = measured[
        "image"
    ].copy()

    contour = measured[
        "largest_contour"
    ]

    if contour is not None:

        cv2.drawContours(
            result,
            [contour],
            -1,
            (0, 255, 0),
            2,
        )

    text = (
        f"{predicted} "
        f"{measured['largest_area_ratio']:.3f}"
    )

    cv2.putText(
        result,
        text,
        (10, 30),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.65,
        (
            0,
            200,
            0,
        )
        if is_correct
        else (
            0,
            0,
            255,
        ),
        2,
    )

    output_path = (
        OUTPUT_DIR
        / f"{image_path.stem}_result.jpg"
    )

    cv2.imwrite(
        str(output_path),
        result,
    )


    # ========================================================
    # ⑭ 이미지별 검사 결과를 CSV 저장 목록에 추가
    # - 파일명, 실제 정답, 예측값을 기록합니다.
    # - 정답 여부와 Area Ratio도 저장합니다.
    # - Contour 수, 중심점, 판정 사유를 함께 기록합니다.
    # ========================================================

    rows.append(
        {
            "file": image_path.name,
            "expected": expected,
            "predicted": predicted,
            "correct": is_correct,
            "largest_area_ratio": measured[
                "largest_area_ratio"
            ],
            "valid_count": measured[
                "valid_count"
            ],
            "center_x_norm": measured[
                "center_x_norm"
            ],
            "center_y_norm": measured[
                "center_y_norm"
            ],
            "decision": decision,
            "reason": reason,
        }
    )


# ============================================================
# ⑮ 검사 이미지 수 확인 및 CSV 보고서 저장
# - 검사한 이미지가 하나도 없다면 프로그램을 종료합니다.
# - 각 이미지의 검사 결과를 CSV 표로 저장합니다.
# - utf-8-sig로 저장하여 Excel에서 한글을 읽기 쉽게 합니다.
# ============================================================

if total == 0:

    print(
        "검사할 이미지가 없습니다."
    )

    raise SystemExit

with REPORT_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:

    fieldnames = [
        "file",
        "expected",
        "predicted",
        "correct",
        "largest_area_ratio",
        "valid_count",
        "center_x_norm",
        "center_y_norm",
        "decision",
        "reason",
    ]

    writer = csv.DictWriter(
        file,
        fieldnames=fieldnames,
    )

    writer.writeheader()

    writer.writerows(
        rows
    )


# ============================================================
# ⑯ 최종 Batch Test 성능 결과 출력
# - 확인 이미지 수 : 검사에 사용된 전체 이미지 수
# - 기준과 일치   : 정답과 예측값이 일치한 수
# - 불일치        : 틀리게 판정한 이미지 수
# - Accuracy      : 정답 수 / 전체 검사 수
# - 결과 CSV와 이미지 저장 경로를 출력합니다.
# ============================================================

print()
print("==============================")
print("Batch Test 결과")
print("==============================")

print(
    "확인 이미지 수:",
    total,
)

print(
    "기준과 일치:",
    correct,
)

print(
    "불일치:",
    total - correct,
)

print(
    "Accuracy:",
    round(
        correct / total,
        3,
    ),
)

print(
    "CSV:",
    REPORT_PATH,
)

print(
    "결과 이미지:",
    OUTPUT_DIR,
)

```

실행합니다.

```bash
python src/22_cap_batch_test.py
```

### 의사코드

```
[① 필요한 라이브러리 불러오기]
Path, CSV, OpenCV 준비
        ↓
[② 이미지 및 결과 저장 경로 설정]
cap_on / cap_off 폴더 지정
Reference 이미지 2장 지정
결과 이미지 및 CSV 저장 폴더 생성
작은 Contour 제외 기준 설정
        ↓
[③ 검사할 이미지 불러오기]
measure() 함수 정의
        ↓
이미지 파일 읽기
읽기 실패 → None 반환
        ↓
[④ Gray → Otsu 이진화]
BGR → Gray 변환
        ↓
Otsu 이진화
어두운 병뚜껑을 흰색 객체로 변환
        ↓
[⑤ Morphology로 이미지 정리]
3×3 Kernel 생성
        ↓
Opening → 작은 노이즈 제거
Closing → 작은 틈 정리
        ↓
[⑥ Contour 검출 및 작은 객체 제외]
외곽 Contour 검출
        ↓
이미지 전체 면적 계산
        ↓
너무 작은 Contour 제외
        ↓
[⑦ 가장 큰 객체의 Feature 측정]
가장 큰 Contour 선택
        ↓
Area Ratio 계산
        ↓
Moments로 중심점 계산
        ↓
중심점 좌표를 0~1로 변환
        ↓
측정 결과 반환
        ↓
[⑧ Reference 이미지로 판정 기준값 생성]
cap_on_001.jpg  → on_ratio
cap_off_001.jpg → off_ratio
        ↓
두 비율의 중간값 계산
        ↓
candidate_threshold 생성
        ↓
[⑨ OK / NG 판정 함수 작성]
새로운 이미지의 Area Ratio 입력
        ↓
기준값과 비교
        ↓
cap_on → OK
cap_off → NG
        ↓
[⑩ Test 이미지 4장 준비]
cap_on_002.jpg
cap_on_003.jpg
cap_off_002.jpg
cap_off_003.jpg
        ↓
결과 목록 및 정확도 계산 변수 준비
        ↓
[⑪ Test 이미지 반복 검사]
이미지 한 장씩 불러오기
        ↓
measure()로 Area Ratio 측정
        ↓
judge()로 OK / NG 판정
        ↓
[⑫ 예측 결과와 실제 정답 비교]
예측한 cap_on / cap_off
        VS
실제 cap_on / cap_off
        ↓
일치 여부 확인
전체 검사 수와 정답 수 기록
        ↓
[⑬ 판정 결과 이미지 생성 및 저장]
Contour를 초록색으로 표시
예측값과 Area Ratio 표시
        ↓
정답과 일치 → 초록색 글자
정답과 불일치 → 빨간색 글자
        ↓
결과 JPG 저장
        ↓
[⑭ 이미지별 검사 결과 기록]
파일명, 실제 정답, 예측값
Area Ratio, 중심점, OK / NG 등을
CSV 저장용 목록에 추가
        ↓
[⑮ 검사 결과 확인 및 CSV 저장]
검사한 이미지가 없으면 종료
        ↓
측정 결과 CSV 파일 생성
        ↓
[⑯ 최종 Batch Test 결과 출력]
전체 검사 이미지 수
정답 수
오답 수
Accuracy 계산
CSV 및 결과 이미지 경로 출력
```

---

## 21.1 이번 코드에서 중요한 변화

기존 코드에서는 별도의:

```
samples/day03/reference/
```

폴더가 필요했습니다.

이제는 필요 없습니다.

현재 실제 데이터 구조를 그대로 사용합니다.

```
samples/day03/
├── cap_on/
│   ├── cap_on_001.jpg   ← Reference
│   ├── cap_on_002.jpg   ← Test
│   └── cap_on_003.jpg   ← Test
│
└── cap_off/
    ├── cap_off_001.jpg  ← Reference
    ├── cap_off_002.jpg  ← Test
    └── cap_off_003.jpg  ← Test
```

---

## 결과 해석

```
Reference ON : 0.4379
Reference OFF: 0.2838
Threshold    : 0.3608

==============================
Batch Test 결과
==============================
확인 이미지 수: 4
기준과 일치: 4
불일치: 0
Accuracy: 1.0
CSV: reports/day03_inspection_results.csv
결과 이미지: outputs/day03/batch
```

결과는 현재 640×480 조건에서 만든 간단한 Rule이 나머지 4장의 테스트 이미지에 모두 잘 적용되었다는 뜻입니다.

핵심은 이 부분입니다.

```
Reference ON : 0.4379
Reference OFF: 0.2838
Threshold    : 0.3608
```

`largest_area_ratio`를 기준으로 보면,

```
뚜껑 있음 기준 이미지
→ 0.4379

뚜껑 없음 기준 이미지
→ 0.2838
```

으로 차이가 났습니다.

그리고 두 값의 중간값을 임시 기준으로 사용했습니다.

```
(0.4379 + 0.2838) / 2
≈ 0.3608
```

따라서 현재 Rule은 사실상 다음과 같습니다.

```
largest_area_ratio >= 0.3608
→ cap_on
→ OK

largest_area_ratio < 0.3608
→ cap_off
→ NG
```

즉 병뚜껑이 있을 때는 검출된 가장 큰 영역이 ROI에서 차지하는 비율이 더 크게 나타났고, 뚜껑이 없을 때는 더 작게 나타난 것입니다.

그리고 Batch Test 결과가:

```
확인 이미지 수: 4
기준과 일치: 4
불일치: 0
Accuracy: 1.0
```

이므로,

```
cap_on_002 → 맞음
cap_on_003 → 맞음

cap_off_002 → 맞음
cap_off_003 → 맞음
```

총 4장을 모두 정확하게 구분했다는 뜻입니다.

그래서 현재 실습 결과는 이렇게 정리하면 됩니다.

```
Reference 2장
→ Rule 기준 설정

Threshold = 0.3608
        ↓
새로운 Test 이미지 4장에
같은 Rule 적용
        ↓
4장 모두 정답
        ↓
현재 촬영 조건에서는
Rule이 정상적으로 동작함
```

다만 Accuracy 1.0 = 이 검사기가 실제로 100% 정확하다는 뜻은 아닙니다.

지금 Test 데이터가 겨우 4장이기 때문입니다.

```
Test 4장
4장 성공
→ 이번 소규모 실습에서는 100%

하지만

실제 검사기의 일반 성능이 100%
→ 아직 알 수 없음
```

정리하면

> 한 쌍의 기준 이미지에서 만든 Rule을 기준을 바꾸지 않고 새로운 4장의 이미지에 적용했더니 모두 정상적으로 구분되었습니다. 따라서 현재 촬영 조건에서는 `largest_area_ratio`가 병뚜껑 유무를 구분할 수 있는 유효한 특징 후보임을 확인했습니다.

---

# 22. Batch 결과를 확인하고 판정이 틀린 이미지가 있는지 살펴봅니다

다음 CSV를 확인합니다.

```text
reports/day03_inspection_results.csv
```

결과

```
file,expected,predicted,correct,largest_area_ratio,valid_count,center_x_norm,center_y_norm,decision,reason
cap_on_002.jpg,cap_on,cap_on,True,0.5599226237580234,1,0.4865991955292156,0.49990236620192463,OK,
cap_on_003.jpg,cap_on,cap_on,True,0.4566431924882629,2,0.48148468597358407,0.44985482102160756,OK,
cap_off_002.jpg,cap_off,cap_off,True,0.29029781387642944,6,0.4529701984416369,0.5347417142101677,NG,NG_CAP_AREA
cap_off_003.jpg,cap_off,cap_off,True,0.31687282566679553,7,0.4291398466942242,0.5606455874394706,NG,NG_CAP_AREA
```

### Batch Test 검사 결과 — CSV 표

|file|expected|predicted|correct|largest_area_ratio|valid_count|center_x_norm|center_y_norm|decision|reason|
|---|---|---|---|---|---|---|---|---|---|
|cap_on_002.jpg|cap_on|cap_on|True|0.559923|1|0.486599|0.499902|OK||
|cap_on_003.jpg|cap_on|cap_on|True|0.456643|2|0.481485|0.449855|OK||
|cap_off_002.jpg|cap_off|cap_off|True|0.290298|6|0.452970|0.534742|NG|NG_CAP_AREA|
|cap_off_003.jpg|cap_off|cap_off|True|0.316873|7|0.429140|0.560646|NG|NG_CAP_AREA|

가독성을 위해 소수점 여섯째 자리까지 표시했습니다.

이 CSV는 각 테스트 이미지에서 어떤 수치가 측정되었고, 그 수치로 Rule이 어떤 판정을 내렸는지 기록한 결과표입니다.

| 이미지         | 실제      | 예측      | Area Ratio | 판정  |
| ----------- | ------- | ------- | ---------: | --- |
| cap_on_002  | cap_on  | cap_on  | **0.5599** | OK  |
| cap_on_003  | cap_on  | cap_on  | **0.4566** | OK  |
| cap_off_002 | cap_off | cap_off | **0.2903** | NG  |
| cap_off_003 | cap_off | cap_off | **0.3169** | NG  |

그리고 결과 이미지도 확인합니다.

```
outputs/day03/batch/
```

![[{7175F7E3-984B-4159-9257-F40855791C63}.png]]


---

# 23. 3일차 Mini Challenge — 새로운 검사 Rule을 직접 설계합니다

지금까지 병뚜껑 예제를 따라 하면서 다음 과정을 경험했습니다.

```
ROI 이미지
    ↓
Gray / HSV 등 필요한 표현 선택
    ↓
Threshold / Mask
    ↓
Morphology
    ↓
Contour
    ↓
Feature 측정
    ↓
실제 값 비교
    ↓
Rule
    ↓
OK / NG
    ↓
새로운 이미지로 확인
```

이제부터는 완성된 코드를 그대로 따라 하지 않습니다.

새로운 검사 문제를 하나 정하고, 지금까지 배운 방법 중 **필요한 기술을 직접 선택하여 Rule을 만들어 봅니다.**

> Mini Challenge의 핵심은 복잡한 코드를 만드는 것이 아니라  
> **“무엇을 측정하면 정상과 불량을 구분할 수 있는가?”를 스스로 생각하는 것**입니다.

---

## 23.1 검사 문제를 하나 선택합니다

다음 중 하나를 선택하거나 직접 새로운 문제를 정합니다.

### A. 존재 여부 검사

예:

```
병뚜껑 있음 / 없음

부품 있음 / 없음

스티커 부착 / 미부착
```

사용할 수 있는 Feature 예:

```
Area
Area Ratio
Contour Count
```

---

### B. 객체 개수 검사

예:

```
정상
→ 부품 4개

불량
→ 부품 3개 또는 5개
```

사용할 수 있는 Feature:

```
Contour Count
```

---

### C. 색상 객체 검사

예:

```
빨간 부품 있음
→ OK

빨간 부품 없음
→ NG
```

2일차에서 배운 HSV를 활용할 수 있습니다.

```
BGR
↓
HSV
↓
Color Mask
↓
Morphology
↓
Contour
↓
Feature
↓
Rule
```

---

### D. 직접 정한 검사

교실에서 구할 수 있는 물체를 사용하여 간단한 검사 문제를 직접 정합니다.

예:

```
동전 개수

병뚜껑 색상

특정 부품 존재 여부

스티커 위치

간단한 크기 차이
```

문제는 복잡할 필요가 없습니다.

**눈으로 정상과 불량을 구분할 수 있고, 그 차이를 숫자로 측정할 수 있으면 됩니다.**

---

# 24. 먼저 검사 문제를 설계합니다

코드를 작성하기 전에 아래 내용을 먼저 적습니다.

```
검사 대상:

정상 상태:

불량 상태:

눈으로 보이는 차이:

검사할 영역:

사용할 전처리:

측정할 Feature:

OK 조건:

NG 조건:
```

예를 들어 객체 개수 검사라면:

```
검사 대상:
검은색 부품

정상 상태:
부품 4개

불량 상태:
3개 또는 5개

눈으로 보이는 차이:
객체 개수가 다름

사용할 처리:
Gray
→ Otsu
→ Morphology
→ Contour

측정할 Feature:
valid_count

OK 조건:
valid_count == 4

NG 조건:
valid_count != 4
```

색상이 중요한 문제라면 Gray를 반드시 사용할 필요는 없습니다.

```
색상이 중요함
        ↓
HSV 사용

형태나 밝기가 중요함
        ↓
Gray / Binary 사용
```

> 먼저 검사 문제를 정하고 필요한 기술을 선택합니다.  
> 기술을 먼저 정한 뒤 억지로 문제에 적용하지 않습니다.

---

# 25. Challenge용 이미지를 직접 준비합니다

다음 폴더를 사용합니다.

```
mkdir -p samples/day03/challenge
mkdir -p outputs/day03/challenge
```

최소한 정상과 불량 상태를 각각 준비합니다.

예:

```
samples/day03/challenge/
├── ok_01.jpg
└── ng_01.jpg
```

가능하다면 확인용 이미지를 한 장씩 더 준비합니다.

```
samples/day03/challenge/
├── ok_01.jpg
├── ok_02.jpg
├── ng_01.jpg
└── ng_02.jpg
```

역할은 다음과 같습니다.

```
ok_01 / ng_01
→ 특징을 비교하고 Rule을 생각하는 이미지

ok_02 / ng_02
→ 만든 Rule을 다시 확인하는 이미지
```

앞의 병뚜껑 실습과 마찬가지로 촬영할 때는 가능한 한 기본 조건을 유지합니다.

```
카메라 거리

촬영 방향

배경

조명

제품이 놓이는 위치
```

일부러 실패 이미지를 만들 필요는 없습니다.

---

# 26. 앞에서 만든 코드를 참고하여 직접 작성합니다

새 파일을 만듭니다.

```
src/23_day03_challenge.py
```

처음부터 모든 코드를 새로 외워서 작성할 필요는 없습니다.

앞에서 작성한 다음 파일을 참고할 수 있습니다.

```
18_morphology_compare.py
→ Morphology 참고

19_contour_measure.py
→ Contour / Area / BBox / Center 참고

20_cap_feature_compare.py
→ 여러 Feature 측정 참고

21_cap_rule.py
→ Rule 작성 참고

22_cap_batch_test.py
→ 새로운 이미지 Test 참고
```

즉:

```
기존 코드 복사
        ↓
내 검사 문제에 필요한 부분 선택
        ↓
불필요한 부분 제거
        ↓
Feature 변경
        ↓
Rule 직접 작성
```

방식으로 진행합니다.

---

## 26.1 기본 구조만 먼저 만듭니다

다음 구조를 참고하여 자신의 코드를 작성합니다.

```
from pathlib import Path

import cv2


image_path = Path(
    "samples/day03/challenge/ok_01.jpg"
)

output_dir = Path(
    "outputs/day03/challenge"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


image = cv2.imread(
    str(image_path)
)

if image is None:
    raise FileNotFoundError(
        image_path
    )


# ------------------------------------------
# TODO 1
# 검사 문제에 맞는 전처리를 선택합니다.
#
# 예:
# Gray
# HSV
# Threshold
# Color Mask
# ------------------------------------------



# ------------------------------------------
# TODO 2
# 필요한 경우 Morphology를 적용합니다.
# ------------------------------------------



# ------------------------------------------
# TODO 3
# Contour를 찾고 필요한 Feature를 계산합니다.
#
# 예:
# largest_area_ratio
# valid_count
# center
# ------------------------------------------



# ------------------------------------------
# TODO 4
# 실제 측정값을 확인한 뒤 Rule을 작성합니다.
# ------------------------------------------

decision = "CHECK"
reason = "RULE_NOT_DEFINED"


print(
    "Decision:",
    decision,
)

print(
    "Reason:",
    reason,
)
```

처음 실행했을 때:

```
Decision: CHECK
Reason: RULE_NOT_DEFINED
```

가 나오는 것은 오류가 아닙니다.

아직 자신의 Rule을 작성하지 않았다는 뜻입니다.

---

# 27. 먼저 측정하고 그다음 Rule을 만듭니다

앞의 병뚜껑 실습과 같은 원칙을 적용합니다.

```
정상 이미지
        ↓
Feature 측정

불량 이미지
        ↓
같은 Feature 측정

        ↓

두 값 비교

        ↓

구분 가능한 Feature 선택

        ↓

Rule 작성
```

예를 들어 객체 개수라면:

```
정상 이미지
valid_count = 4

불량 이미지
valid_count = 3

        ↓

Count를 Rule로 사용
```

그다음 코드를 작성합니다.

```
if valid_count == 4:
    decision = "OK"
    reason = ""
else:
    decision = "NG"
    reason = "NG_COUNT"
```

Area Ratio를 사용한다면 실제 값을 먼저 확인하고 기준을 정합니다.

```
정상
→ Area Ratio = ?

불량
→ Area Ratio = ?

        ↓

두 값의 차이 확인

        ↓

Candidate Threshold 결정
```

> 숫자를 먼저 정하고 이미지를 맞추지 않습니다.  
> **이미지를 측정한 뒤 숫자를 정합니다.**

---

# 28. 만든 Rule을 새로운 이미지에 다시 적용합니다

`ok_01.jpg`, `ng_01.jpg`로 Rule을 만들었다면 가능하면 다음 이미지에도 적용합니다.

```
ok_02.jpg
ng_02.jpg
```

이때 Rule을 바로 수정하지 않습니다.

```
Rule 결정
      ↓
Rule 고정
      ↓
새 이미지 적용
      ↓
결과 확인
```

확인할 내용은 단순합니다.

```
실제 정상
→ 프로그램도 OK인가?

실제 불량
→ 프로그램도 NG인가?
```

틀렸다면 먼저 원인을 생각합니다.

```
입력 이미지 문제?

Threshold / Mask 문제?

Morphology 문제?

Contour 문제?

선택한 Feature 문제?

Rule 기준 문제?
```

한 번 틀렸다고 바로 Parameter를 모두 바꾸지 않습니다.

---

# 29. 한 번에 한 가지씩 수정합니다

결과가 원하는 대로 나오지 않는다면 한 번에 하나의 조건만 변경합니다.

예:

```
Kernel
3 → 5
```

또는:

```
Binary 방향
INV → NORMAL
```

또는:

```
MIN_CONTOUR_AREA_RATIO
0.001 → 다른 값
```

다음처럼 여러 값을 동시에 바꾸지 않습니다.

```
Kernel 변경
+
Threshold 변경
+
Morphology 변경
+
Rule 변경

→ 무엇 때문에 결과가 달라졌는지 알기 어려움
```

대신:

```
한 변수 변경
        ↓
다시 실행
        ↓
결과 비교
        ↓
변화 원인 판단
```

순서로 진행합니다.

---

# 30. Mini Challenge 결과를 짧게 기록합니다

다음 파일을 작성합니다.

```
reports/day03_rule_analysis.md
```

앞에서 이미 배운 내용을 다시 길게 적을 필요는 없습니다.

다음 내용만 기록합니다.

```
# Day 03 Mini Challenge 결과

## 1. 검사 문제

검사 대상:

정상 상태:

불량 상태:


## 2. 사용한 처리

처리 흐름:

예:
Gray
→ Otsu
→ Morphology
→ Contour


## 3. 선택한 Feature

Feature:

선택 이유:


## 4. 실제 측정값

정상:

불량:


## 5. Rule

OK 조건:

NG 조건:

NG Reason:


## 6. 새로운 이미지 확인

확인한 이미지 수:

올바른 판정:

틀린 판정:


## 7. 결과 해석

잘 동작한 부분:

문제가 있었던 부분:

수정한 내용:


## 8. 결론

이 검사에서 가장 중요했던 Feature는 무엇이었는가?

어떤 조건이 바뀌면 이 Rule이 실패할 수 있는가?
```

실패 이미지가 없었다면 억지로 작성하지 않습니다.

```
틀린 판정:
없음
```

으로 기록하면 됩니다.

---

# 31. Mini Challenge 완료 조건

다음 항목을 확인합니다.

- [ ]  새로운 검사 문제를 직접 정했다.
- [ ]  정상과 불량 상태를 정의했다.
- [ ]  직접 사용할 이미지를 준비했다.
- [ ]  필요한 전처리를 스스로 선택했다.
- [ ]  Morphology가 필요한지 판단했다.
- [ ]  Contour에서 사용할 Feature를 선택했다.
- [ ]  실제 측정값을 먼저 확인했다.
- [ ]  측정값을 근거로 Rule을 작성했다.
- [ ]  OK / NG 판정이 실행되었다.
- [ ]  가능하면 새로운 이미지에도 같은 Rule을 적용했다.
- [ ]  결과가 이상하면 한 번에 한 변수씩 확인했다.
- [ ]  결과와 판단 근거를 `day03_rule_analysis.md`에 기록했다.

---

## Mini Challenge에서 가장 중요한 질문

Challenge가 끝났을 때 코드 전체를 외울 필요는 없습니다.

다음 흐름을 자신의 말로 설명할 수 있으면 됩니다.

```
무엇을 검사하려고 했는가?
        ↓
정상과 불량은 무엇이 달랐는가?
        ↓
그 차이를 어떤 영상처리로 분리했는가?
        ↓
Contour에서 무엇을 측정했는가?
        ↓
실제 측정값은 어떻게 달랐는가?
        ↓
그래서 어떤 Rule을 만들었는가?
        ↓
새로운 이미지에서도 동작했는가?
```

> **Mini Challenge의 목표는 새로운 OpenCV 문법을 배우는 것이 아니라, 지금까지 배운 기술을 스스로 선택하여 하나의 작은 검사 문제를 해결해 보는 것입니다.**

---

# 32. README에 Day03 내용을 추가합니다

`README.md`의 Day02 아래에 다음 내용을 추가합니다.

```markdown
## Day 03 — Morphology·Contour와 규칙 기반 판정

1일차에서 실제 촬영한 ROI 이미지와
2일차에서 학습한 Threshold / Otsu 방식을 이어서 사용하였다.

3일차에서는 Binary Image를 Morphology로 정리하고,
Contour를 이용하여 객체의 특징을 숫자로 측정한 뒤
실제 측정값을 근거로 Rule 기반 OK / NG 판정을 만들었다.


### 실습 흐름

ROI Image
→ Gray
→ Threshold / Otsu
→ Binary Image
→ Morphology
→ Contour
→ Feature 측정
→ cap_on / cap_off 비교
→ Candidate Rule
→ OK / NG
→ 새로운 실제 촬영 이미지 Batch Test
→ 결과 분석
→ Mini Challenge


### 주요 실습

- Opening / Closing의 역할 확인
- Kernel 3×3, 5×5, 9×9 결과 비교
- 실제 병뚜껑 ROI에서 Contour 검출
- Area / Area Ratio / Count / BBox / Center 측정
- `cap_on`과 `cap_off`의 Feature 비교
- 실제 측정값을 이용한 Candidate Threshold 생성
- Area Ratio 기반 OK / NG Rule 구현
- 640×480 조건으로 새로운 `cap_on / cap_off` 이미지 촬영
- Reference와 Test 이미지를 구분하여 같은 Rule 적용
- Batch Test 결과를 CSV와 결과 이미지로 확인
- 새로운 검사 문제를 직접 설계하는 Mini Challenge


### 병뚜껑 Rule 실습

기준 이미지에서 측정한 `largest_area_ratio`를 비교하여
병뚜껑 있음과 없음을 구분하는 후보 기준값을 만들었다.

```text
cap_on
→ largest_area_ratio가 상대적으로 큼

cap_off
→ largest_area_ratio가 상대적으로 작음

        ↓

두 값 사이에서
Candidate Threshold 결정

        ↓

Threshold 이상
→ cap_on
→ OK

Threshold 미만
→ cap_off
→ NG
```

---

# 33. Local Git으로 3일차 작업을 기록합니다

먼저 확인합니다.

```bash
git status
```

코드를 기록합니다.

```bash
git add src
git commit -m "feat: add day03 morphology contour rules"
```

문서를 기록합니다.

```bash
git add reports README.md
git commit -m "docs: add day03 rule inspection analysis"
```

기록을 확인합니다.

```bash
git log --oneline
```

`samples/`와 `outputs/`가 `.gitignore`에 포함되어 있다면 이미지가 Git에 올라가지 않는 것은 정상입니다.

기관 또는 기업 제공 데이터는 허용된 범위 밖으로 업로드하지 않습니다.

---

# 34. 오늘 만들어져 있어야 하는 주요 산출물

## Python 코드

```text
src/18_morphology_compare.py 
src/19_contour_measure.py 
src/20_cap_feature_compare.py 
src/21_cap_rule.py 
src/22_cap_batch_test.py 
src/23_day03_challenge.py
```

각 파일의 역할은 다음과 같습니다.

```
18_morphology_compare.py
→ Opening / Closing과 Kernel 크기 비교

19_contour_measure.py
→ Contour 검출과 Area / BBox / Center 측정

20_cap_feature_compare.py
→ cap_on / cap_off의 Feature 비교

21_cap_rule.py
→ 실제 측정값으로 Candidate Rule 생성

22_cap_batch_test.py
→ 새로 촬영한 이미지에 같은 Rule 적용

23_day03_challenge.py
→ 새로운 검사 문제를 직접 설계하고 Rule 작성
```


## 기준·검사 결과

```text
outputs/day03/
├── morphology/
├── contour/
├── feature/
├── rule/
├── batch/
└── challenge/
```

각 폴더에는 다음 결과가 저장됩니다.

```
morphology/
→ Binary와 Opening / Closing 비교 결과

contour/
→ Contour, BBox, Center 표시 결과

feature/
→ cap_on / cap_off Feature 비교 결과

rule/
→ Candidate Rule의 OK / NG 결과

batch/
→ 새로운 Test 이미지의 판정 결과

challenge/
→ Mini Challenge의 Binary / Contour / Rule 결과
```

## 실제 촬영 데이터

```text
samples/day03/
├── reference/
│   ├── cap_on.jpg
│   └── cap_off.jpg
│
├── cap_on/
│   ├── cap_on_001.jpg
│   ├── cap_on_002.jpg
│   └── cap_on_003.jpg
│
├── cap_off/
│   ├── cap_off_001.jpg
│   ├── cap_off_002.jpg
│   └── cap_off_003.jpg
│
└── challenge/
    ├── ok_01.jpg
    ├── ng_01.jpg
    └── ...
```

### 기록

```
reports/day03_reference_features.csv
reports/day03_inspection_results.csv
reports/day03_rule_analysis.md
```

역할은 다음과 같습니다

```
day03_reference_features.csv
→ 기준 이미지의 Feature 측정값 비교

day03_inspection_results.csv
→ 새로운 실제 이미지의 Batch Test 결과

day03_rule_analysis.md
→ Mini Challenge의 검사 문제, Feature, Rule, 결과 기록
```

---

# 35. 3일차 완료 체크

3일차 실습을 마친 뒤 다음 내용을 확인합니다.

## 1·2일차와의 연결

- [ ] 1일차에 촬영한 실제 ROI 이미지가 2일차와 3일차로 어떻게 이어지는지 설명할 수 있다.
- [ ] `cap_on_reference.jpg`, `cap_off_reference.jpg`가 이미 ROI로 잘린 이미지임을 이해했다.
- [ ] 이미 ROI인 이미지에 전체 Frame의 ROI 좌표를 다시 적용하지 않았다.
- [ ] 2일차에서 학습한 Threshold / Otsu 방식을 3일차에서 다시 적용했다.
- [ ] Binary Image가 최종 판정 결과가 아니라 Contour를 찾기 위한 입력이라는 것을 이해했다.

## Morphology

- [ ] Opening의 목적을 설명할 수 있다.
- [ ] Closing의 목적을 설명할 수 있다.
- [ ] Kernel 3×3, 5×5, 9×9 결과를 비교했다.
- [ ] Kernel이 커질수록 무조건 좋은 것은 아니라는 것을 확인했다.
- [ ] Morphology를 너무 강하게 적용하면 실제 객체 형태도 변할 수 있음을 이해했다.
- [ ] Morphology의 목적이 가장 깨끗한 이미지를 만드는 것이 아니라 Contour를 안정적으로 측정하는 것임을 이해했다.

## Contour와 Feature 측정

- [ ] Binary Image에서 Contour를 찾았다.
- [ ] 가장 큰 Contour를 선택했다.
- [ ] Contour Area를 계산했다.
- [ ] Area Ratio를 계산했다.
- [ ] Bounding Box를 계산했다.
- [ ] Center를 계산했다.
- [ ] 유효 Contour 개수를 확인했다.
- [ ] 작은 Contour를 Noise로 제외하는 기준이 필요할 수 있음을 이해했다.
- [ ] `cap_on`과 `cap_off`에 동일한 Pipeline을 적용했다.
- [ ] 두 이미지의 Feature를 CSV에 저장하고 비교했다.

## Feature 비교와 Candidate Rule

- [ ] 실제 측정값을 보기 전에 임의의 기준값을 먼저 정하지 않았다.
- [ ] `cap_on`과 `cap_off`의 Feature 차이를 비교했다.
- [ ] 현재 데이터에서 `largest_area_ratio`가 주요 Rule 후보임을 확인했다.
- [ ] 두 Reference 측정값 사이에서 Candidate Threshold를 만들었다.
- [ ] Candidate Threshold가 최종 Calibration 기준이 아니라는 것을 이해했다.
- [ ] 측정값과 Rule 판정을 구분하여 설명할 수 있다.

## 새로운 실제 이미지 Batch Test

- [ ] IP Webcam 입력 해상도를 640×480으로 통일하여 새로운 이미지를 촬영했다.
- [ ] `cap_on` 3장과 `cap_off` 3장을 준비했다.
- [ ] 첫 번째 `cap_on / cap_off` 한 쌍을 Reference로 사용했다.
- [ ] 나머지 4장을 Test 이미지로 분리했다.
- [ ] Reference에서 만든 Rule을 Test 전에 고정했다.
- [ ] Test 이미지에 같은 전처리와 같은 Rule을 적용했다.
- [ ] `expected`와 `predicted` 결과를 비교했다.
- [ ] Batch Test 결과를 `day03_inspection_results.csv`에 저장했다.
- [ ] 결과 이미지를 `outputs/day03/batch/`에서 확인했다.
- [ ] 소량의 Test 이미지에서 Accuracy가 1.0이어도 실제 검사 성능이 100%라는 뜻은 아님을 이해했다.
- [ ] 틀린 이미지가 있다면 먼저 실패 원인을 확인하고, 없다면 억지로 실패 사례를 만들지 않았다.

## Mini Challenge

- [ ] 새로운 검사 문제를 직접 정했다.
- [ ] 정상 상태와 불량 상태를 정의했다.
- [ ] 정상과 불량에서 눈으로 보이는 차이가 무엇인지 설명했다.
- [ ] Challenge용 이미지를 직접 준비했다.
- [ ] 검사 문제에 필요한 전처리 방법을 스스로 선택했다.
- [ ] 필요한 경우 Gray / Threshold / HSV / Mask 등을 선택했다.
- [ ] Morphology가 필요한지 스스로 판단했다.
- [ ] Area, Area Ratio, Count, BBox, Center 등에서 사용할 Feature를 선택했다.
- [ ] 실제 측정값을 먼저 확인한 뒤 Rule을 만들었다.
- [ ] OK / NG 조건을 직접 작성했다.
- [ ] 필요한 경우 NG Reason을 작성했다.
- [ ] 가능하면 새로운 이미지에도 같은 Rule을 다시 적용했다.
- [ ] 결과가 원하는 대로 나오지 않을 경우 한 번에 한 변수씩 수정했다.
- [ ] 사용한 Feature와 Rule을 자신의 말로 설명할 수 있다.

## 결과 기록과 정리

- [ ] `reports/day03_reference_features.csv`를 확인했다.
- [ ] `reports/day03_inspection_results.csv`를 확인했다.
- [ ] `reports/day03_rule_analysis.md`에 Mini Challenge 결과를 기록했다.
- [ ] README에 Day03 학습 내용을 정리했다.
- [ ] Local Git에 Day03 코드와 문서를 Commit했다.

---

## 참고 — 검사 문제에 따라 사용할 Feature가 달라집니다

규칙 기반 머신비전에서는 모든 검사에 같은 Feature를 사용하는 것이 아닙니다.

먼저 **정상과 불량에서 무엇이 달라지는지** 확인하고, 그 차이를 가장 잘 숫자로 표현할 수 있는 Feature를 선택합니다.

예를 들어 다음과 같습니다.

```text
병뚜껑 있음 / 없음
→ 객체가 차지하는 크기가 달라짐
→ Area / Area Ratio 활용

부품이 정확히 4개인지 확인
→ 객체의 개수가 중요함
→ Contour Count 활용

빨간 부품이 있는지 확인
→ 색상이 중요한 정보임
→ HSV / Color Mask 활용

부품이 정해진 위치에 있는지 확인
→ 객체의 위치가 중요함
→ Center / Bounding Box 활용
```

### 1. Area / Area Ratio

객체의 **크기나 존재 여부**가 정상과 불량을 구분하는 중요한 특징일 때 사용할 수 있습니다.

```
Contour
→ Area 측정
→ ROI 전체에서 차지하는 비율 계산
→ 정상 / 불량 비교
```

이번 병뚜껑 실습에서는 `largest_area_ratio`가 병뚜껑 있음과 없음을 비교하기에 적합한 Feature로 나타났습니다.

### 2. Count

객체가 **몇 개 존재하는지**가 중요한 검사에 사용할 수 있습니다.

예:

```
정상
→ 부품 4개

불량
→ 부품 3개 또는 5개
```

이 경우에는 유효한 Contour의 개수를 세어 Rule을 만들 수 있습니다.

```
valid_count == 4
→ OK

그 외
→ NG
```

### 3. HSV / Color Mask

객체의 **색상 자체가 정상과 불량을 구분하는 조건**이라면 Gray보다 HSV가 더 적합할 수 있습니다.

예:

```
빨간 부품 존재
→ OK

빨간 부품 없음
→ NG
```

이 경우에는:

```
BGR
→ HSV
→ 원하는 색상 영역 Mask
→ Contour
→ Feature 측정
```

과 같은 흐름을 사용할 수 있습니다.

### 4. Center / Bounding Box

객체가 있는지만 확인하는 것이 아니라 **정해진 위치에 제대로 놓여 있는지** 검사해야 할 때 사용할 수 있습니다.

```
Contour
→ Center 계산
→ 정상 위치와 비교
```

또는 Bounding Box의 위치와 크기를 이용하여 객체가 정상 위치에서 벗어났는지 확인할 수도 있습니다.

---

> **중요:** Feature는 많이 사용하는 것이 목적이 아닙니다.  
> 정상과 불량의 차이를 가장 안정적으로 설명할 수 있는 Feature를 선택하여 가능한 한 단순한 Rule부터 만드는 것이 중요합니다.

![[Pasted image 20260830005751.png]]