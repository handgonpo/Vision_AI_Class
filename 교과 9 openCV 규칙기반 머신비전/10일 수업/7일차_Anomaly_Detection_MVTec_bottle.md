
1~6일차에서는 대부분 어떤 불량을 검사할지 미리 정한 뒤 그에 맞는 방법으로 OK / NG를 판정했습니다.

```
1~3일차
병뚜껑 / 부품
→ 면적·개수·위치 검사

4~5일차
광천김 로고
→ 기준 로고와 비교

6일차
13자리 SN
→ OCR로 문자를 읽고 기준값과 비교
```

7일차에서는 검사 방법이 조금 달라집니다.

이번에는 특정 불량 하나를 찾는 것이 아니라,

> 정상 제품의 모습을 기준으로 만들어 두고, 정상과 많이 다른 제품을 이상(Anomaly)으로 찾아봅니다.

---

## 오늘은 어떤 데이터를 사용할까요?

오늘은 산업용 이상검출 실습에 많이 사용하는 **MVTec AD의 `bottle` 데이터**를 사용합니다.

쉽게 말하면 병 제품의 정상과 이상을 검사하는 데이터셋입니다.

```
정상 병
→ good

이상이 있는 병
→ broken_large
→ broken_small
→ contamination
```

의미는 다음과 같습니다.

```
good
→ 정상 제품

broken_large
→ 크게 파손된 제품

broken_small
→ 작게 파손된 제품

contamination
→ 오염이 있는 제품
```

오늘의 검사 문제를 한 문장으로 표현하면 다음과 같습니다.

> 정상적인 병의 특징을 기준으로 만들고, 새로운 병이 정상인지 파손·오염 등의 이상이 있는지 판정해 봅니다.

---

## 오늘 사용하는 데이터

강사가 미리 준비한 다음 데이터를 그대로 사용합니다.

```
samples/day07/original/bottle/
├── train/
│   └── good/
│
├── test/
│   ├── good/
│   ├── broken_large/
│   ├── broken_small/
│   └── contamination/
│
└── ground_truth/
```

---

## 오늘의 전체 흐름

![[Pasted image 20260901221557.png]]

```
정상 병 이미지
        ↓
정상의 특징을 기준으로 만듦
        ↓
처음 보는 병 이미지 입력
        ↓
정상과 얼마나 다른지 측정
        ↓
Normal / Anomaly 판정
        ↓
어디가 이상하게 보였는지 확인
        ↓
실패 원인 분석
```

후반 Challenge에서는 새로운 데이터셋을 사용하지 않습니다.

기존 정상 이미지를:

```
어둡게
밝게
흐리게
위치 이동
회전
```

시켜 보면서,

> 제품은 정상인데 촬영 조건이 달라져도 검사기가 이상으로 판단할 수 있는가?

도 확인합니다.

---

## 오늘 수업이 끝나면

다음 정도를 이해하고 직접 실행할 수 있으면 됩니다.

```
- Anomaly Detection이 무엇인지 설명할 수 있다.
- MVTec bottle의 정상과 이상 데이터를 구분할 수 있다.
- 정상 데이터로 정상 기준을 만들 수 있다.
- 새로운 이미지를 Normal / Anomaly로 판정할 수 있다.
- 파손·오염 이미지의 검사 결과를 확인할 수 있다.
- Heatmap으로 이상하게 판단한 위치를 확인할 수 있다.
- 촬영 환경 변화 때문에 발생하는 오탐을 확인할 수 있다.
- 검사 결과와 실패 원인을 기록할 수 있다.
```

> 7일차의 핵심은 “불량 종류를 하나씩 외워서 찾는 것”이 아니라, 정상 제품을 기준으로 정상과 다른 제품을 찾아보는 것입니다.

---

# 1. 7일차의 전체 흐름

7일차에서는 정상 제품을 기준으로 이상 제품을 찾아보는 Anomaly Detection을 실습합니다.

오늘 사용하는 데이터는 MVTec AD의 `bottle`입니다.

```
정상
→ good

이상
→ broken_large
→ broken_small
→ contamination
```

오늘은 파손이나 오염을 각각 따로 학습시키는 것이 아닙니다.

먼저 정상 병의 특징을 기준으로 만든 뒤 새로운 병이 그 기준과 얼마나 다른지 확인합니다.

```
정상 병 여러 장
        ↓
정상 기준 생성
        ↓
새로운 병 검사
        ↓
정상과 다른 정도 측정
        ↓
Normal / Anomaly
```

7일차 전체 실습은 다음 순서로 진행합니다.

![[Pasted image 20260901223141.png]]

```
정상 병 사진 준비
        ↓
정상 기준 만들기
        ↓
처음 보는 병 검사
        ↓
정상 / 이상 판정
        ↓
어디가 이상한지 확인
        ↓
촬영 조건을 바꾸어 다시 검사
        ↓
틀린 이유 분석
        ↓
결과 정리
```

7일차에서는 먼저 정상적인 병 사진들을 보고 ‘정상은 이런 모습이다’라는 기준을 만듭니다.

그다음 처음 보는 병 사진을 넣어 보고, 정상 기준과 많이 다르면 **이상**, 비슷하면 정상으로 판단해 봅니다.

이 과정에서 크게 깨진 병, 조금 깨진 병, 오염된 병이 실제로 이상으로 잘 구분되는지 확인합니다.

판정이 끝난 뒤에는 컴퓨터가 병의 어느 부분을 이상하게 봤는지 눈으로 확인해 보고, 정상 병 사진을 일부러 어둡게 하거나 흐리게 만들고 위치를 옮겨 보면서 제품은 정상인데 촬영 상태 때문에 이상으로 잘못 판단할 수도 있는지 실험합니다.

마지막에는 다른 정상 사진으로 같은 과정을 다시 해 보고, 어떤 경우에 잘 맞았고 어떤 경우에 틀렸는지 정리하면서 7일차 실습을 마무리합니다.

---

# 2. 앞 수업과 7일차는 어떻게 연결되나요?

7일차에서 사용하는 기술은 새롭지만 검사기를 만드는 기본 순서는 앞에서 계속 연습한 방식과 같습니다.

특히 3일차와 5일차에서 배운 생각이 중요합니다.

```
3일차

이미지
→ Feature(특징) 측정
→ 숫자 기준과 비교
→ OK / NG
```

5일차에서는 이 기준을 데이터를 이용하여 결정했습니다.

```
Calibration(판정 기준 조정)
        ↓
측정값 비교
        ↓
Threshold 결정(정상/이상 기준값을 정함)
        ↓
Rule Freeze(그 기준값과 판정 조건을 더 이상 바꾸지 않고 고정함)
        ↓
Test
```

7일차에서도 비슷합니다.

```
Normal Train(정상 학습용 데이터)
        ↓
정상 기준 생성
        ↓
Normal Validation(정상 검증용 데이터)
        ↓
Threshold 결정(정상/이상 기준값을 정함)
        ↓
Rule Freeze(그 기준값과 판정 조건을 더 이상 바꾸지 않고 고정함)
        ↓
Test
```

차이는 기준을 만드는 데이터입니다.

```
5일차

OK + NG
측정값을 비교하여
구분 기준을 만듦


7일차

Normal 중심으로
정상 기준을 만듦
        ↓
정상에서 많이 벗어나면
Anomaly(비정상)
```

즉 7일차에서도 중요한 생각은 같습니다.

> 먼저 기준을 만들고 → 기준을 Freeze한 뒤 → 처음 보는 Test 데이터에서 확인합니다.

이 구조는 교과9에서 반복해 온 검사 개발 흐름과도 일치합니다.

![[Pasted image 20260902110117.png]]

- Calibration(교정 . 조정)
- Threshold(임계값)
- Rule Freeze(규칙 고정)

---

# 3. 오늘 사용할 MVTec AD `bottle` 데이터

7일차에서는 MVTec AD의 `bottle` 데이터를 사용합니다.

원본 데이터는 다음 위치에 준비합니다.

```
samples/day07/original/bottle/
```

오늘 사용하는 데이터의 역할은 다음과 같습니다.

|데이터|의미|사용|
|---|---|---|
|`train/good`|정상 병|정상 기준 생성과 검증|
|`test/good`|처음 보는 정상 병|정상 Test|
|`test/broken_large`|크게 파손된 병|이상 Test|
|`test/broken_small`|작게 파손된 병|이상 Test|
|`test/contamination`|오염된 병|이상 Test|
|`ground_truth`|실제 결함 위치|Heatmap 비교|

여기서 가장 중요한 것은 기준을 만드는 데이터와 최종 Test 데이터를 구분하는 것입니다.

```
train/good
        ↓
정상 기준을 만드는 데 사용

test/*
        ↓
기준을 만든 뒤
처음 보는 데이터로 검사
```

또한 `train/good`의 정상 이미지는 이후 실습에서:

```
Normal Train
+
Normal Validation
```

으로 나누어 사용합니다.

세부적인 분리 방법은 실제 데이터 준비 실습에서 확인합니다.

---

# 4. 오늘 사용하는 Anomaly Detection(이상탐지) 방법

7일차에서는 불량의 모양을 하나씩 정해서 찾는 대신 정상 제품의 모습을 먼저 기준으로 만듭니다.

![[Pasted image 20260902114731.png]]

- ResNet18 = 이미지에서 모양, 색, 경계, 패턴 같은 특징을 찾아내는 CNN 모델

```
정상 병 여러 장
        ↓
정상의 특징을 숫자로 표현
        ↓
정상 기준 생성
```

새로운 병도 같은 방식으로 특징을 확인한 뒤 정상 기준과 비교합니다.

```
새로운 병
        ↓
정상 기준과 비교
        ↓
비슷함 → Normal
많이 다름 → Anomaly
```

오늘은 사전학습된 `ResNet18`을 이용하여 이미지의 특징을 숫자로 바꿉니다.

```
병 이미지
        ↓
ResNet18
        ↓
Feature
```

지금은 이 정도만 이해하면 됩니다.

`Feature Vector`, `Feature Map`, 거리 계산 등은 실제 실습에서 필요한 순간에 하나씩 확인합니다.

> 7일차의 핵심은 정상 제품을 기준으로 만들고, 새로운 제품이 그 기준에서 얼마나 벗어났는지를 확인하는 것입니다.

---
# Anomaly Detection(이상검출) 파일별 역할 및 설명

| 파일명                                   | 역할                 | 설명                                                                            |
| ------------------------------------- | ------------------ | ----------------------------------------------------------------------------- |
| `49_day06_mini_challenge.py`          | 6일차 Mini Challenge | 새로운 제품의 문자코드를 OCR로 읽고 OK / NG를 판정하는 6일차 마지막 실습 파일입니다.                         |
| `50_day07_check_anomaly_env.py`       | AI 환경·데이터셋 확인      | PyTorch·CUDA(GPU)·OpenCV 버전과 MVTec bottle의 정상·이상 이미지 폴더 및 이미지 수를 확인합니다.       |
| `51_day07_prepare_dataset.py`         | 데이터셋 분리            | MVTec bottle 원본을 보존하면서 정상 Train·Validation과 정상·이상 Test 데이터를 분리합니다.            |
| `day07_anomaly_core.py`               | 이상검출 공통 함수         | ResNet18 특징 추출, Anomaly Score 계산, Heatmap 생성 기능을 다른 파일에서 재사용하도록 정리합니다.        |
| `52_day07_build_normal_reference.py`  | 정상 기준 생성           | 정상 병 이미지 167장의 특징을 ResNet18로 추출하고 평균을 계산하여 Normal Reference를 만듭니다.            |
| `53_day07_measure_validation.py`      | 정상 데이터 Score 측정    | 정상 Validation 이미지 42장을 Normal Reference와 비교하여 Anomaly Score를 계산하고 CSV로 저장합니다. |
| `54_day07_compare_thresholds.py`      | Threshold 후보 비교    | 정상 Score의 90·95·99 Percentile을 계산하여 정상과 이상을 구분할 Threshold 후보를 비교합니다.          |
| `55_day07_freeze_anomaly_rule.py`     | 이상검출 Rule 확정       | 95 Percentile 기준으로 선택한 Threshold를 JSON에 저장하고 이후 Test에서 동일하게 사용하도록 고정합니다.      |
| `56_day07_test_anomaly.py`            | 정상·이상 이미지 일괄 검사    | 처음 보는 병 이미지 83장을 검사하여 Normal / Anomaly를 판정하고 TP·TN·FP·FN 결과를 CSV에 저장합니다.      |
| `57_day07_evaluate.py`                | 검사 성능 평가           | Test 결과로 Accuracy·Precision·Recall을 계산하고 잘못 판정한 FP·FN 이미지를 별도로 저장합니다.         |
| `58_day07_make_heatmaps.py`           | 이상 위치 시각화          | 대표 Test 이미지 8장의 Heatmap과 Overlay를 생성하고 실제 결함 위치인 Ground Truth Mask와 비교합니다.    |
| `59_day07_make_challenge_variants.py` | 촬영환경 변화 이미지 생성     | 정상 병 이미지 한 장을 Dark·Bright·Blur·Shift·Rotation의 5가지 조건으로 변형합니다.                |
| `60_day07_run_challenge.py`           | 촬영환경 변화에 따른 성능 검증  | 변형 이미지 5장에 기존 Rule을 적용하여 정상 제품이 촬영 조건 변화로 Anomaly로 잘못 판정되는지 확인합니다.            |
| `61_day07_mini_challenge.py`          | 나만의 이상검출 실험        | 다른 정상 병 이미지를 선택하고 Dark·Blur·Rotation 강도를 직접 조절하여 Score와 PASS / FP 결과를 분석합니다.  |

![[Pasted image 20261020092823.png]]

![[Pasted image 20261020093149.png]]

---

# 5. 오늘 추가되는 프로젝트 구조

6일차 마지막 실행 파일은:

```
src/49_day06_mini_challenge.py
```

였습니다.

따라서 7일차 실행 파일은 `50번`부터 이어서 작성합니다.

전체 프로젝트 구조를 다시 펼쳐 볼 필요는 없고, 7일차에 새로 사용하는 위치만 확인합니다.

```
subject9_machinevision/
│
├── samples/
│   └── day07/
│       ├── original/
│       ├── train/
│       ├── validation/
│       ├── test/
│       ├── ground_truth/
│       ├── challenge/
│       └── mini_challenge/
│
├── artifacts/
│   └── day07/
│
├── outputs/
│   └── day07/
│
├── configs/
├── reports/
│
└── src/
    ├── ...
    ├── 49_day06_mini_challenge.py
    ├── 50_day07_check_anomaly_env.py
    ├── 51_day07_prepare_dataset.py
    ├── day07_anomaly_core.py
    ├── 52_day07_build_normal_reference.py
    ├── 53_day07_measure_validation.py
    ├── 54_day07_compare_thresholds.py
    ├── 55_day07_freeze_anomaly_rule.py
    ├── 56_day07_test_anomaly.py
    ├── 57_day07_evaluate.py
    ├── 58_day07_make_heatmaps.py
    ├── 59_day07_make_challenge_variants.py
    ├── 60_day07_run_challenge.py
    └── 61_day07_mini_challenge.py
```

각 폴더가 정확히 언제 만들어지고 무엇이 저장되는지는 해당 실습에서 확인합니다.

---

# PART1 — 실습 환경 + MVTec bottle 데이터 준비 (7~12)

PART1에서는 아직 이상검출을 실행하지 않습니다.

먼저 오늘 사용할 프로젝트와 데이터를 준비하고 실행 환경을 확인합니다.

```
기존 프로젝트 열기
        ↓
MVTec bottle 원본 준비
        ↓
데이터가 제대로 들어왔는지 확인
        ↓
Python / PyTorch 환경 확인
        ↓
7일차 실습 시작
```
---

# 7. 기존 프로젝트를 다시 엽니다

6일차까지 사용한 프로젝트를 그대로 이어서 사용합니다.

프로젝트 폴더로 이동합니다.

```
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

새로운 프로젝트나 가상환경을 만들지 않습니다.

---

# 8. MVTec `bottle` 원본을 프로젝트에 넣습니다

먼저 원본을 넣을 폴더를 만듭니다.

```bash
mkdir -p samples/day07/original
```

다운로드한 MVTec AD의 `bottle` 폴더 전체를 다음 위치에 복사합니다.

```bash
explorer.exe .
```

```
samples/day07/original/bottle/
```

`original`은 다운로드한 원본을 그대로 보관하는 위치입니다.

이후 실습에서는 이 원본을 직접 수정하지 않습니다.

---

# 9. 원본 데이터가 제대로 들어왔는지 확인합니다

복사가 끝났다면 다음 명령으로 확인합니다.

```bash
ls samples/day07/original/bottle
```

다음 항목이 보이면 됩니다.

```
train
test
ground_truth
```

조금 더 확인하고 싶다면:

```bash
find samples/day07/original/bottle -maxdepth 2 -type d | sort
```

를 실행합니다.

이 단계에서는 각 폴더의 이미지 수를 하나씩 따로 셀 필요는 없습니다.

앞에서 이미 각 데이터의 역할을 확인했고, 뒤의 환경 점검 프로그램에서도 데이터 폴더와 이미지 수를 다시 확인합니다.

따라서 지금은:

```
MVTec bottle 원본 복사
        ↓
train / test / ground_truth 확인
        ↓
원본 준비 완료
```

까지만 확인하면 됩니다.

---

# 10. PyTorch와 Torchvision 환경을 확인합니다

오늘은 사전학습 CNN을 사용합니다.

```bash
python -c "import torch; print('Torch:', torch.__version__)"
```

```bash
python -c "import torchvision; print('Torchvision:', torchvision.__version__)"
```

GPU 사용 가능 여부:

```bash
python -c "import torch; print('CUDA:', torch.cuda.is_available())"
```

필요한 패키지를 확인합니다.

```bash
python -c "import cv2, numpy, pandas, PIL, matplotlib; print('Packages OK')"
```

필요한 패키지가 없다면 현재 `.venv`에서 설치합니다.

```bash
pip install torch torchvision numpy pandas matplotlib opencv-python pillow
```

```
- torch → PyTorch 딥러닝 라이브러리  
    ResNet18 같은 딥러닝 모델을 실행하고 Tensor 연산, GPU 연산을 수행합니다.
    
- torchvision → 이미지용 PyTorch 보조 라이브러리  
    ResNet18 같은 사전학습 이미지 모델과 이미지 전처리 기능을 제공합니다.
    
- numpy → 수치 계산 라이브러리 
    배열, 벡터, 거리 계산 같은 숫자 연산에 사용합니다.
    
- pandas → 표 형태 데이터 처리 라이브러리 
    CSV 결과를 읽고 저장하거나 Score를 정리할 때 사용합니다.
    
- matplotlib → 그래프·시각화 라이브러리 
    Score 분포나 결과 그래프를 그릴 때 사용합니다.
    
- opencv-python → OpenCV 이미지 처리 라이브러리  
    이미지 읽기, 크기 변경, Blur, 회전, Heatmap 처리 등에 사용합니다.
    
- pillow → Python 이미지 처리 라이브러리(PIL)  
    이미지를 열고 RGB 형식으로 변환하여 ResNet18에 넣기 전에 처리할 때 사용합니다.
```

환경을 기록합니다.

```bash
pip freeze > requirements.txt
```

---

# 11. `50_day07_check_anomaly_env.py` — 환경과 Dataset을 한 번에 점검합니다

이번에는 Python 프로그램을 이용하여:

```
PyTorch 환경
    +
GPU 사용 여부
    +
MVTec bottle 폴더
    +
정상 / 이상 이미지 수
```

를 한 번에 확인합니다.

> 이 프로그램이 정상적으로 실행되면 7일차 실습 환경과 원본 Dataset 준비가 완료된 것입니다.

파일:

```
src/50_day07_check_anomaly_env.py
```

작성합니다.

```python
from pathlib import Path

import cv2
import numpy as np
import torch
import torchvision


ROOT = Path(
    "samples/day07/original/bottle"
)

required_dirs = [
    ROOT / "train/good",

    ROOT / "test/good",
    ROOT / "test/broken_large",
    ROOT / "test/broken_small",
    ROOT / "test/contamination",

    ROOT / "ground_truth/broken_large",
    ROOT / "ground_truth/broken_small",
    ROOT / "ground_truth/contamination",
]


print("=" * 70)
print("DAY07 ANOMALY ENV CHECK")
print("=" * 70)

# --------------------------------------------------
# 1. 실행 환경 확인
# --------------------------------------------------
print("Torch      :", torch.__version__)
print("Torchvision:", torchvision.__version__)
print("OpenCV     :", cv2.__version__)
print("NumPy      :", np.__version__)
print("CUDA       :", torch.cuda.is_available())

if torch.cuda.is_available():
    print(
        "GPU        :",
        torch.cuda.get_device_name(0),
    )


# --------------------------------------------------
# 2. Dataset 폴더 확인
# --------------------------------------------------
print()
print("Dataset:", ROOT)

all_ok = True

for directory in required_dirs:
    exists = directory.exists()

    if not exists:
        all_ok = False

    print(
        f"[{'OK' if exists else 'MISSING'}]",
        directory,
    )

if not all_ok:
    raise FileNotFoundError(
        "MVTec bottle 폴더 구조를 다시 확인하세요."
    )


# --------------------------------------------------
# 3. 이미지 수 확인
# --------------------------------------------------
print()

print(
    "Train Good        :",
    len(
        list(
            (ROOT / "train/good").glob("*.png")
        )
    ),
)

print(
    "Test Good         :",
    len(
        list(
            (ROOT / "test/good").glob("*.png")
        )
    ),
)

print(
    "Broken Large      :",
    len(
        list(
            (ROOT / "test/broken_large").glob("*.png")
        )
    ),
)

print(
    "Broken Small      :",
    len(
        list(
            (ROOT / "test/broken_small").glob("*.png")
        )
    ),
)

print(
    "Contamination     :",
    len(
        list(
            (ROOT / "test/contamination").glob("*.png")
        )
    ),
)


print()
print("Day07 Environment OK")
```

실행합니다.

```
python src/50_day07_check_anomaly_env.py
```

정상적으로 실행되면 대략 다음과 같은 내용을 확인할 수 있습니다.

```
======================================================================
DAY07 ANOMALY ENV CHECK
======================================================================
Torch      : 2.13.0+cu130
Torchvision: 0.28.0+cu130
OpenCV     : 5.0.0
NumPy      : 2.5.2
CUDA       : True
GPU        : NVIDIA GeForce RTX 4080 SUPER

Dataset: samples/day07/original/bottle
[OK] samples/day07/original/bottle/train/good
[OK] samples/day07/original/bottle/test/good
[OK] samples/day07/original/bottle/test/broken_large
[OK] samples/day07/original/bottle/test/broken_small
[OK] samples/day07/original/bottle/test/contamination
[OK] samples/day07/original/bottle/ground_truth/broken_large
[OK] samples/day07/original/bottle/ground_truth/broken_small
[OK] samples/day07/original/bottle/ground_truth/contamination

Train Good        : 209
Test Good         : 20
Broken Large      : 20
Broken Small      : 22
Contamination     : 21

Day07 Environment OK
```

숫자를 외울 필요는 없습니다.

중요한 것은:

```
필요한 폴더가 모두 존재하는가?
        +
이미지가 실제로 들어 있는가?
        +
PyTorch 환경이 정상인가?
```

를 확인하는 것입니다.

### 의사코드

```
1. 7일차 MVTec bottle 원본 경로를 지정한다.

2. 수업에 필요한
   train / test / ground_truth 폴더 목록을 만든다.

3. PyTorch, Torchvision, OpenCV, NumPy 버전을 출력한다.

4. CUDA 사용 가능 여부를 확인한다.

5. CUDA를 사용할 수 있으면 GPU 이름도 출력한다.

6. 필요한 MVTec bottle 폴더가 모두 존재하는지 확인한다.

7. 하나라도 빠진 폴더가 있으면 오류를 발생시키고
   Dataset 구조를 다시 확인한다.

8. 정상 Train 이미지 수를 확인한다.

9. 정상 Test와
   broken_large / broken_small / contamination
   이미지 수를 확인한다.

10. 모든 점검이 끝나면
    Day07 Environment OK를 출력한다.
```

이 단계의 흐름만 간단히 보면:

```
실행 환경 확인
        ↓
Dataset 경로 확인
        ↓
필요한 폴더 확인
        ↓
이미지 수 확인
        ↓
7일차 실습 준비 완료
```

입니다.

---

# 12. ResNet18의 학습된 Weight를 준비합니다

7일차에서는 처음부터 ResNet18을 새로 학습시키지 않습니다.

이미 많은 이미지를 학습한 Pretrained ResNet18을 불러와 사용합니다.

```
ResNet18
        +
미리 학습된 Weight
        ↓
이미지의 특징 추출
```

여기서 `Weight`는 ResNet18이 이전 학습을 통해 얻은 **학습된 값**입니다.
즉 우리가 처음부터 학습시키지 않아도 이미 ImageNet으로 학습된 Weight가 들어 있는 ResNet18을 불러옵니다.

오늘은 다음 설정을 사용합니다.

```python
ResNet18_Weights.DEFAULT
```

처음 실행하는 경우에는 이 Weight 파일이 자동으로 다운로드될 수 있습니다.

따라서 본 실습 전에 한 번 실행하여 정상적으로 준비되는지 확인합니다.

```bash
python -c "from torchvision.models import resnet18, ResNet18_Weights; resnet18(weights=ResNet18_Weights.DEFAULT); print('ResNet18 Weight OK')"
```

정상적으로 준비되면 다음과 같이 출력됩니다.

```
ResNet18 Weight OK
```

한 번 다운로드된 Weight는 보통 컴퓨터에 저장되므로 이후 실행에서는 다시 받을 필요가 없습니다.

만약 다운로드 과정에서 오류가 발생한다면 인터넷 연결 상태를 먼저 확인합니다.

> 이 단계의 목적은 뒤의 실습에서 ResNet18을 바로 사용할 수 있도록 미리 학습된 Weight를 준비하는 것입니다.

간단히 보면:

```
Pretrained ResNet18 사용
        ↓
처음 실행 시 Weight 다운로드
        ↓
컴퓨터에 저장
        ↓
이후 Feature 추출에 사용
```
---

# PART2 — Anomaly Detection(이상탐지) 원리 + 공통 Feature 함수 (13~19)

---

# 13. Classification / Detection / Anomaly Detection을 구분합니다

이미지를 검사하는 AI 방법에는 여러 가지가 있습니다.

오늘은 그중 다음 세 가지 용어의 차이만 간단히 구분합니다.

| 방식                | 쉽게 말하면               | 결과               |
| ----------------- | -------------------- | ---------------- |
| Classification    | 이미지가 무엇인지 구분         | 클래스 이름           |
| Detection         | 이미지에서 무엇이 어디에 있는지 찾음 | 위치 + 클래스         |
| Anomaly Detection | 정상과 비교하여 이상한지 확인     | 정상 / 이상 또는 이상 정도 |

---

## Classification — 이미지가 무엇인지 구분합니다

Classification은 이미지 전체를 보고 **어떤 종류인지 분류하는 방법**입니다.

![[Pasted image 20260902130403.png]]

예를 들어 미리 다음 세 종류를 학습했다고 가정합니다.

```
Normal
Scratch
Crack
```

새로운 이미지를 넣으면:

```
새 이미지
        ↓
Classification
        ↓
Scratch
```

처럼 하나의 클래스로 분류합니다.

즉 질문은:

> **이 이미지는 어떤 종류인가?**

입니다.

---

## Detection — 무엇이 어디에 있는지 찾습니다

Detection은 결함의 종류뿐 아니라 **이미지 안에서 결함이 있는 위치까지 찾는 방법**입니다.

![[Pasted image 20260902130621.png]]

예를 들어:

```
Scratch + BBox
Crack + BBox
```

처럼 결함의 종류와 위치를 학습합니다.

새로운 이미지를 넣으면:

```
새 이미지
        ↓
Detection
        ↓
여기에 Scratch가 있음
```

처럼 위치와 클래스를 함께 알려줍니다.

즉 질문은:

> **무엇이 어디에 있는가?**

입니다.

---

## Anomaly Detection — 정상과 다른지를 확인합니다

Anomaly Detection은 조금 다릅니다.

![[Pasted image 20260902130807.png]]

여러 장의 정상 이미지를 이용하여 먼저 **정상의 기준**을 만듭니다.

```
Normal
Normal
Normal
Normal
        ↓
정상 기준 생성
```

그다음 새로운 이미지가 들어오면 정상 기준과 비교합니다.

```
새 이미지
        ↓
정상 기준과 비교
        ↓
비슷함 → Normal
많이 다름 → Anomaly
```

즉 질문은:

> **이 제품은 정상적인 모습에서 많이 벗어났는가?**

입니다.

---

세 가지를 한 줄로 정리하면:

```
Classification
→ 무엇인가?

Detection
→ 무엇이 어디에 있는가?

Anomaly Detection
→ 정상과 얼마나 다른가?
```

7일차에서는 이 중 **Anomaly Detection**을 사용합니다.

---

# 14. Feature란 무엇인가요?

CNN의 중간 Layer에서는 이미지의 Edge, Texture, Pattern, Shape와 관련된 많은 숫자가 만들어집니다.

![[Group 16.png]]

오늘은 이 숫자 표현을 `Feature`라고 부릅니다.

```text
Bottle Image
        ↓
ResNet18
        ↓
Feature Map
        ↓
Pooling
        ↓
Feature Vector
```

Feature Vector는 이미지 전체가 정상 기준과 얼마나 다른지 계산하는 데 사용합니다.

Feature Map은 어느 위치가 정상과 다른지 Heatmap을 만들 때 사용합니다.

---

# 15. Normal Reference란 무엇인가요?

정상 이미지 한 장만 기준으로 쓰지 않습니다.

![[Pasted image 20260902131655.png]]

```text
Normal 001 → Feature
Normal 002 → Feature
Normal 003 → Feature
...
        ↓
평균
        ↓
Normal Reference
```

정상 여러 장의 평균 Feature를 이용하면 한 장의 우연한 특징만 기준으로 사용하는 문제를 줄일 수 있습니다.

---

# 16. Anomaly Score는 어떻게 계산하나요?

검사 이미지의 Feature Vector와 Normal Reference Vector 사이의 Euclidean Distance를 계산합니다.

![[Pasted image 20260902132456.png]]

```text
Test Feature
        ↕ 거리
Normal Reference
```

```text
거리 작음
→ 정상과 비슷함
→ Score 낮음

거리 큼
→ 정상과 다름
→ Score 높음
```

오늘의 판정 Rule은 단순합니다.

```text
score <= threshold
→ Normal

score > threshold
→ Anomaly
```

---

# 17. `day07_anomaly_core.py` — 이상탐지에서 공통으로 사용할 기능을 만듭니다

앞에서 다음 원리를 배웠습니다.

```
병 이미지
        ↓
ResNet18
        ↓
Feature 추출
        ↓
정상 기준과 비교
        ↓
Anomaly Score
```

그리고 이미지의 어느 위치가 정상과 다른지 확인할 때는 Feature Map을 이용합니다.

이제 이 기능들을 여러 실습에서 반복해서 사용할 수 있도록 하나의 파일에 모아 둡니다.

파일:

```
src/day07_anomaly_core.py
```

> 이 파일은 단독으로 실행하는 프로그램이 아닙니다.  
> 뒤에서 작성하는 `52`, `53`, `56`, `58` 등의 프로그램이 이 파일의 함수를 불러서 사용합니다.

작성합니다.

```python
import cv2
import numpy as np
import torch
import torch.nn.functional as F

from PIL import Image
from torchvision.models import (
    resnet18,
    ResNet18_Weights,
)


# --------------------------------------------------
# 1. 실행 장치 설정
# --------------------------------------------------
DEVICE = torch.device(
    "cuda" if torch.cuda.is_available() else "cpu"
)


# --------------------------------------------------
# 2. Pretrained ResNet18 설정
# --------------------------------------------------
WEIGHTS = ResNet18_Weights.DEFAULT

# ResNet18에 맞는 이미지 전처리 방법
PREPROCESS = WEIGHTS.transforms()


# --------------------------------------------------
# 3. Feature 추출용 ResNet18 생성
# --------------------------------------------------
def create_backbone():
    model = resnet18(
        weights=WEIGHTS
    )

    # ResNet18의 layer3 출력까지 사용합니다.
    # 이미지의 위치 정보가 어느 정도 남아 있기 때문에
    # Feature Vector와 Heatmap에 모두 사용할 수 있습니다.
    feature_extractor = torch.nn.Sequential(
        *list(model.children())[:-3]
    )

    # 학습하지 않고 Feature 추출용으로 사용합니다.
    feature_extractor.eval()
    feature_extractor.to(DEVICE)

    return feature_extractor


# --------------------------------------------------
# 4. 이미지를 ResNet18 입력 형식으로 변환
# --------------------------------------------------
def load_image_tensor(image_path):
    image = Image.open(
        image_path
    ).convert("RGB")

    tensor = PREPROCESS(
        image
    ).unsqueeze(0)

    return tensor.to(DEVICE)


# --------------------------------------------------
# 5. Feature Vector + Feature Map 추출
# --------------------------------------------------
@torch.no_grad()
def extract_features(
    model,
    image_path,
):
    tensor = load_image_tensor(
        image_path
    )

    feature_map = model(
        tensor
    )

    # ----------------------------------------------
    # 이미지 전체 특징
    # Feature Map → Pooling → Feature Vector
    # ----------------------------------------------
    pooled = F.adaptive_avg_pool2d(
        feature_map,
        (1, 1),
    ).flatten(1)

    pooled = F.normalize(
        pooled,
        dim=1,
    )

    # ----------------------------------------------
    # 위치별 특징
    # Heatmap에서 사용
    # ----------------------------------------------
    normalized_map = F.normalize(
        feature_map,
        dim=1,
    )

    vector = (
        pooled[0]
        .cpu()
        .numpy()
    )

    spatial_map = (
        normalized_map[0]
        .cpu()
        .numpy()
    )

    return vector, spatial_map


# --------------------------------------------------
# 6. 이미지 전체 Anomaly Score 계산
# --------------------------------------------------
def anomaly_score(
    feature_vector,
    reference_vector,
):
    difference = (
        feature_vector
        - reference_vector
    )

    score = np.linalg.norm(
        difference
    )

    return float(score)


# --------------------------------------------------
# 7. 위치별 Anomaly 차이 계산
# --------------------------------------------------
def anomaly_map(
    spatial_feature,
    reference_map,
):
    difference = (
        spatial_feature
        - reference_map
    )

    score_map = np.sqrt(
        np.sum(
            difference ** 2,
            axis=0,
        )
    )

    return score_map


# --------------------------------------------------
# 8. Heatmap 이미지 생성
# --------------------------------------------------
def make_heatmap_overlay(
    image_path,
    score_map,
):
    image = cv2.imread(
        str(image_path)
    )

    if image is None:
        raise FileNotFoundError(
            image_path
        )

    height, width = image.shape[:2]

    resized_map = cv2.resize(
        score_map,
        (width, height),
        interpolation=cv2.INTER_CUBIC,
    )

    normalized = cv2.normalize(
        resized_map,
        None,
        0,
        255,
        cv2.NORM_MINMAX,
    )

    heatmap_uint8 = (
        normalized.astype(
            np.uint8
        )
    )

    heatmap_color = cv2.applyColorMap(
        heatmap_uint8,
        cv2.COLORMAP_JET,
    )

    overlay = cv2.addWeighted(
        image,
        0.65,
        heatmap_color,
        0.35,
        0,
    )

    return (
        heatmap_uint8,
        heatmap_color,
        overlay,
    )


# --------------------------------------------------
# 9. 저장된 Normal Reference 불러오기
# --------------------------------------------------
def load_reference(
    reference_path,
):
    data = np.load(
        reference_path
    )

    return (
        data["reference_vector"],
        data["reference_map"],
    )
```

---

## 이 코드에서는 무엇을 만들었나요?

이 파일에는 앞으로 반복해서 사용할 기능이 들어 있습니다.

```
create_backbone()
→ Pretrained ResNet18 준비


load_image_tensor()
→ 이미지를 ResNet18 입력 형태로 변환


extract_features()
→ Feature Vector
      +
Feature Map 추출


anomaly_score()
→ 이미지 전체가 정상 기준과
얼마나 다른지 Score 계산


anomaly_map()
→ 위치별 차이 계산


make_heatmap_overlay()
→ 위치별 차이를 Heatmap으로 표현


load_reference()
→ 저장해 둔 정상 기준 불러오기
```

전체 흐름으로 보면 다음과 같습니다.

```
이미지
        ↓
load_image_tensor()
→ 이미지를 읽고 ResNet18 입력 형식으로 변환
        ↓
ResNet18
→ 이미지의 특징 추출
        ↓
extract_features()
→ 추출된 특징을 두 가지 형태로 정리
        ↓
 ┌──────┴──────────────────────┐
 ↓                             ↓
Feature Vector             Feature Map
 ↓                             ↓
anomaly_score()            anomaly_map()
 ↓                             ↓
전체 차이 점수              위치별 차이
                               ↓
                         make_heatmap_overlay()
                               ↓
                            Heatmap
```

---

## 의사코드

```
1. CUDA를 사용할 수 있는지 확인하고
   GPU 또는 CPU를 실행 장치로 정한다.

2. ImageNet으로 미리 학습된
   ResNet18 Weight를 준비한다.

3. ResNet18 전체를 분류기로 사용하지 않고
   layer3까지를 Feature 추출기로 만든다.

4. 입력 이미지를 RGB로 읽는다.

5. ResNet18이 사용할 수 있는 크기와 형식으로
   이미지를 전처리한다.

6. ResNet18에 이미지를 넣어
   Feature Map을 얻는다.

7. Feature Map을 Pooling하여
   이미지 전체를 나타내는
   Feature Vector를 만든다.

8. Feature Vector는
   이미지 전체 Anomaly Score 계산에 사용한다.

9. Feature Map은
   이미지의 위치별 차이를 계산하는 데 사용한다.

10. 검사 Feature Vector와
    Normal Reference Vector의 거리를 계산하여
    Anomaly Score를 만든다.

11. 검사 Feature Map과
    Normal Reference Map의 위치별 차이를 계산한다.

12. 위치별 차이를 색으로 표현하여
    Heatmap을 만들 수 있도록 준비한다.

13. 뒤의 프로그램에서 사용할
    Normal Reference를 파일에서 불러오는 기능도 만든다.
```

> 아직 정상 기준 자체를 만든 것은 아닙니다.  
> 지금은 정상 기준을 만들고 검사하기 위해 필요한 공통 도구를 준비한 단계입니다.

---

# 18. 왜 ResNet18의 `layer3`까지 사용하나요?

이 부분은 day07_anomaly_core.py 코드 안에서 왜 `layer3`까지 사용했는지를 설명하는 단락입니다.

![[Pasted image 20260902135835.png]]

ResNet18은 여러 Layer를 차례대로 통과하면서 이미지의 특징을 추출합니다.

```
이미지
 ↓
초기 Layer
 ↓
중간 Layer
 ↓
후반 Layer
 ↓
최종 분류
```

뒤쪽 Layer로 갈수록 이미지 전체를 구분하는 데 필요한 특징은 강해지지만, 이미지에서 어느 위치에 특징이 있었는지에 대한 공간정보는 점점 줄어듭니다.

7일차에서는 두 가지가 모두 필요합니다.

```
Feature Vector
→ 이미지 전체가 정상과 얼마나 다른지 계산

Feature Map
→ 이미지의 어느 위치가 정상과 다른지 확인
```

그래서 오늘은 ResNet18의 중간 부분인 `layer3`까지 사용합니다.

코드의 이 부분입니다.

```python
feature_extractor = torch.nn.Sequential(
    *list(model.children())[:-3]
)
```

쉽게 말하면:

```
ResNet18 전체를 사용하여
최종 분류까지 하지 않고

        ↓

중간 Feature까지만 꺼내서 사용
```

하는 것입니다.

```
ResNet18 layer3
        ↓
 ┌──────┴──────┐
 ↓             ↓
Feature Vector  Feature Map
 ↓             ↓
전체 차이       위치별 차이
```

> `layer3`가 모든 이상탐지 문제에서 반드시 정답이라는 뜻은 아닙니다. 
> 7일차에서는 전체 Score와 Heatmap을 함께 실습하기 위한 교육용 선택으로 사용합니다.

---

# 19. 지금 만든 Baseline을 어떻게 이해해야 하나요?

#17에서 만든 코드는 아직 직접적인 검사 결과를 출력하지 않습니다.

이 파일은 뒤의 프로그램에서 다음 계산을 하기 위한 **공통 기능**입니다.

7일차에서 사용할 기본 검사 방식은 다음과 같습니다.

```
Pretrained ResNet18
        ↓
Feature 추출
        ↓
정상 여러 장의 Feature 평균
        ↓
Normal Reference
        ↓
새 이미지의 Feature와 비교
        ↓
Euclidean Distance
        ↓
Anomaly Score
```

이처럼 가장 먼저 만들어 보는 기본 검사 방법을 **Baseline**이라고 부릅니다.

쉽게 말하면:

> 복잡한 개선을 하기 전에 먼저 만들어 보는 기본형 검사기

입니다.

이후 실제 결과가 나오면 다음과 같이 해석합니다.

```
Score 낮음
→ 정상 기준과 비슷함

Score 높음
→ 정상 기준과 많이 다름
```

하지만 여기서 주의할 점이 있습니다.

Score가 높다고 해서 반드시 제품에 실제 결함이 있다는 뜻은 아닙니다.

예를 들어:

```
작은 결함
→ 전체 Feature 변화가 작아
  Score가 충분히 높아지지 않을 수 있음


조명 변화
위치 변화
배경 변화
→ 제품은 정상이어도
  Feature가 달라져
  Score가 높아질 수 있음
```

따라서 뒤의 실습에서는 단순히:

```
Score가 높다 / 낮다
```

만 확인하는 것이 아니라,

```
정상과 이상이 실제로 잘 구분되는가?

어떤 결함을 놓치는가?

정상인데 이상으로 판단하는 경우가 있는가?

어느 위치를 이상하게 보고 있는가?
```

까지 확인합니다.

> 7일차의 목적은 처음부터 완벽한 이상탐지 모델을 만드는 것이 아니라, 기본 Baseline을 만든 뒤 실제 결과를 보고 잘되는 조건과 실패하는 조건을 분석하는 것입니다.
---

# PART3 — Normal Train으로 정상 Feature 기준 생성 (20~24)

---

# 20. `51_day07_prepare_dataset.py` — 원본을 수업용 데이터로 분리합니다

앞에서 준비한 MVTec `bottle` 원본은 다음 위치에 있습니다.

```
samples/day07/original/bottle/
```

원본 데이터는 그대로 보존하고, 프로그램이 필요한 이미지를 복사하여 다음과 같은 수업용 데이터를 만듭니다.

```
MVTec 원본
        ↓
train/good
        ↓
 ┌──────┴──────┐
 ↓             ↓
80%            20%
 ↓             ↓
Normal Train   Normal Validation


test/good
→ Normal Test


test/broken_large
test/broken_small
test/contamination
→ Anomaly Test


ground_truth
→ 결함 위치 Mask
```

MVTec 원본에는 정상 학습 데이터가 하나의 `train/good` 폴더에 들어 있습니다.

하지만 7일차에서는 정상 데이터를 두 가지 역할로 나누어 사용해야 합니다.

```
원본 train/good
        ↓
무작위로 섞기
        ↓
 ┌──────────┴──────────┐
 ↓                     ↓
80%                    20%
 ↓                     ↓
Normal Train       Normal Validation
정상 기준 생성       Threshold 결정
```


파일:

```text
src/51_day07_prepare_dataset.py
```

작성합니다.

```python
import random
import shutil
from pathlib import Path


# --------------------------------------------------
# 1. 원본과 실습용 데이터 경로
# --------------------------------------------------
RAW_ROOT = Path(
    "samples/day07/original/bottle"
)

OUTPUT_ROOT = Path(
    "samples/day07"
)

SEED = 42
TRAIN_RATIO = 0.8

random.seed(SEED)


# --------------------------------------------------
# 2. 원본 데이터 확인
# --------------------------------------------------
if not RAW_ROOT.exists():
    raise FileNotFoundError(
        f"MVTec bottle 원본을 확인하세요: {RAW_ROOT}"
    )


# --------------------------------------------------
# 3. 이전에 생성한 실습용 데이터만 초기화
#    original 폴더는 절대 삭제하지 않습니다.
# --------------------------------------------------
generated_dirs = [
    OUTPUT_ROOT / "train",
    OUTPUT_ROOT / "validation",
    OUTPUT_ROOT / "test",
    OUTPUT_ROOT / "ground_truth",
]

for directory in generated_dirs:
    if directory.exists():
        shutil.rmtree(directory)


# --------------------------------------------------
# 4. 실습용 폴더 생성
# --------------------------------------------------
train_dir = (
    OUTPUT_ROOT
    / "train/normal"
)

val_dir = (
    OUTPUT_ROOT
    / "validation/normal"
)

test_normal_dir = (
    OUTPUT_ROOT
    / "test/normal"
)

test_anomaly_dir = (
    OUTPUT_ROOT
    / "test/anomaly"
)

ground_truth_dir = (
    OUTPUT_ROOT
    / "ground_truth"
)


for directory in [
    train_dir,
    val_dir,
    test_normal_dir,
    test_anomaly_dir,
    ground_truth_dir,
]:
    directory.mkdir(
        parents=True,
        exist_ok=True,
    )


# --------------------------------------------------
# 5. Normal Train / Validation 분리
# --------------------------------------------------
normal_files = sorted(
    (RAW_ROOT / "train/good").glob("*.png")
)

if not normal_files:
    raise FileNotFoundError(
        RAW_ROOT / "train/good"
    )

random.shuffle(normal_files)

split_index = int(
    len(normal_files)
    * TRAIN_RATIO
)

train_files = (
    normal_files[:split_index]
)

val_files = (
    normal_files[split_index:]
)


for path in train_files:
    shutil.copy2(
        path,
        train_dir / path.name,
    )

for path in val_files:
    shutil.copy2(
        path,
        val_dir / path.name,
    )


# --------------------------------------------------
# 6. Normal Test 복사
# --------------------------------------------------
for path in sorted(
    (RAW_ROOT / "test/good").glob("*.png")
):
    shutil.copy2(
        path,
        test_normal_dir / path.name,
    )


# --------------------------------------------------
# 7. Anomaly Test 복사
# --------------------------------------------------
defect_names = [
    "broken_large",
    "broken_small",
    "contamination",
]

for defect_name in defect_names:

    source_dir = (
        RAW_ROOT
        / "test"
        / defect_name
    )

    destination = (
        test_anomaly_dir
        / defect_name
    )

    destination.mkdir(
        parents=True,
        exist_ok=True,
    )

    for path in sorted(
        source_dir.glob("*.png")
    ):
        shutil.copy2(
            path,
            destination / path.name,
        )


# --------------------------------------------------
# 8. Ground Truth Mask 복사
# --------------------------------------------------
for defect_name in defect_names:

    source_dir = (
        RAW_ROOT
        / "ground_truth"
        / defect_name
    )

    destination = (
        ground_truth_dir
        / defect_name
    )

    destination.mkdir(
        parents=True,
        exist_ok=True,
    )

    if not source_dir.exists():
        continue

    for path in sorted(
        source_dir.glob("*.png")
    ):
        shutil.copy2(
            path,
            destination / path.name,
        )


# --------------------------------------------------
# 9. 결과 확인
# --------------------------------------------------
print("=" * 70)
print("DAY07 DATASET PREPARED")
print("=" * 70)

print(
    "Normal Train      :",
    len(train_files),
)

print(
    "Normal Validation :",
    len(val_files),
)

print(
    "Normal Test       :",
    len(
        list(
            test_normal_dir.glob("*.png")
        )
    ),
)


for defect_name in defect_names:

    count = len(
        list(
            (
                test_anomaly_dir
                / defect_name
            ).glob("*.png")
        )
    )

    print(
        f"{defect_name:18}:",
        count,
    )
```

실행합니다.

```bash
python src/51_day07_prepare_dataset.py
```

### 의사코드

```text
1. MVTec bottle 원본 경로를 지정한다.

2. 실습용 데이터를 만들
   samples/day07 경로를 지정한다.

3. Seed를 42로 고정하여
   실행할 때마다 같은 데이터가 나뉘도록 한다.

4. 원본 bottle 폴더가 실제로 있는지 확인한다.

5. 이전 실행에서 만들어진
   train / validation / test / ground_truth만 삭제한다.

6. original 폴더는 삭제하지 않고 그대로 보존한다.

7. 새로운
   train / validation / test / ground_truth
   폴더를 만든다.

8. 원본 train/good의 정상 이미지 목록을 읽는다.

9. 정상 이미지 순서를 섞는다.

10. 정상 이미지의 80%를
    Normal Train으로 복사한다.

11. 나머지 20%를
    Normal Validation으로 복사한다.

12. 원본 test/good을
    Normal Test로 복사한다.

13. broken_large /
    broken_small /
    contamination을
    Anomaly Test로 복사한다.

14. 각 이상 이미지의
    Ground Truth Mask도 복사한다.

15. 만들어진 Train /
    Validation / Test의 이미지 수를 출력한다.
```

---

# 21. 분리 결과를 확인합니다

정상으로 출력된 결과

```
======================================================================
DAY07 DATASET PREPARED
======================================================================
Normal Train      : 167
Normal Validation : 42
Normal Test       : 20
broken_large      : 20
broken_small      : 22
contamination     : 21
```

`train/good` 209장의 80%를 정수로 계산하기 때문에:

```
209 × 0.8
= 167.2
→ 167장 Train

나머지
→ 42장 Validation
```

이 됩니다.

역할별 데이터가 제대로 분리되었는지 확인합니다.

```bash
find samples/day07 -maxdepth 3 -type d | sort
```

```
Normal Train
→ 정상 기준을 만들 데이터

Normal Validation
→ Threshold를 결정할 데이터

Normal Test
→ 처음 보는 정상 데이터

Anomaly Test
→ 처음 보는 이상 데이터
```

---

# 22. `52_day07_build_normal_reference.py` — 정상 기준을 만듭니다

앞에서 MVTec의 정상 이미지를 다음과 같이 나누었습니다.

```
train/good
        ↓
80%
        ↓
samples/day07/train/normal
        ↓
Normal Train
```

이제 `Normal Train`에 있는 여러 장의 정상 병에서 Feature를 추출하여 정상 제품을 대표하는 기준값인 Normal Reference를 만듭니다.

한 장의 정상 이미지만 기준으로 사용하지 않고 여러 정상 이미지의 특징을 평균냅니다.

```
Normal 001
        ↓
Feature

Normal 002
        ↓
Feature

Normal 003
        ↓
Feature

   ...

        ↓
모든 정상 Feature 평균
        ↓
Normal Reference
```

오늘은 두 종류의 정상 기준을 만듭니다.

```
reference_vector
→ 이미지 전체가 정상과 얼마나 다른지 계산할 때 사용

reference_map
→ 이미지의 어느 위치가 정상과 다른지
  Heatmap으로 확인할 때 사용
```

> 이 과정에서 ResNet18을 새로 학습하는 것은 아닙니다. 이미 학습된 ResNet18을 이용하여 정상 이미지의 Feature를 뽑고 그 평균을 계산합니다.

파일:

```text
src/52_day07_build_normal_reference.py
```

작성합니다.

```python
from pathlib import Path

import numpy as np

from day07_anomaly_core import (
    create_backbone,
    extract_features,
)


# --------------------------------------------------
# 1. 입력 / 출력 경로
# --------------------------------------------------
TRAIN_DIR = Path(
    "samples/day07/train/normal"
)

OUTPUT_PATH = Path(
    "artifacts/day07/normal_reference.npz"
)

OUTPUT_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 2. Feature 추출용 ResNet18 준비
# --------------------------------------------------
model = create_backbone()


# --------------------------------------------------
# 3. 정상 이미지의 Feature를 저장할 리스트
# --------------------------------------------------
vectors = []
maps = []


# --------------------------------------------------
# 4. Normal Train 이미지 읽기
# --------------------------------------------------
image_files = sorted(
    TRAIN_DIR.glob("*.png")
)

if not image_files:
    raise RuntimeError(
        "Normal Train 이미지가 없습니다."
    )

print(
    "Normal Train:",
    len(image_files),
)


# --------------------------------------------------
# 5. 정상 이미지마다 Feature 추출
# --------------------------------------------------
for index, image_path in enumerate(
    image_files,
    start=1,
):
    vector, spatial_map = extract_features(
        model,
        image_path,
    )

    vectors.append(vector)
    maps.append(spatial_map)

    if (
        index == 1
        or index % 20 == 0
        or index == len(image_files)
    ):
        print(
            f"[{index}/{len(image_files)}]",
            image_path.name,
        )


# --------------------------------------------------
# 6. 여러 이미지의 Feature를 하나의 배열로 묶기
# --------------------------------------------------
vectors = np.stack(
    vectors
)

maps = np.stack(
    maps
)


# --------------------------------------------------
# 7. Feature Vector 평균
#    → 이미지 전체 비교용 정상 기준
# --------------------------------------------------
reference_vector = np.mean(
    vectors,
    axis=0,
)

vector_norm = np.linalg.norm(
    reference_vector
)

if vector_norm == 0:
    raise RuntimeError(
        "Normal Reference Vector를 만들 수 없습니다."
    )

reference_vector = (
    reference_vector
    / vector_norm
)


# --------------------------------------------------
# 8. Feature Map 평균
#    → Heatmap 비교용 정상 기준
# --------------------------------------------------
reference_map = np.mean(
    maps,
    axis=0,
)


# --------------------------------------------------
# 9. Normal Reference 저장
# --------------------------------------------------
np.savez_compressed(
    OUTPUT_PATH,
    reference_vector=reference_vector,
    reference_map=reference_map,
)


# --------------------------------------------------
# 10. 결과 확인
# --------------------------------------------------
print("=" * 70)
print("Normal Reference 생성 완료")
print("=" * 70)

print(
    "Vector shape:",
    reference_vector.shape,
)

print(
    "Map shape   :",
    reference_map.shape,
)

print(
    "Saved       :",
    OUTPUT_PATH,
)
```

실행합니다.

```bash
python src/52_day07_build_normal_reference.py
```

### 의사코드

```
1. Normal Train 이미지가 들어 있는
   samples/day07/train/normal 경로를 지정한다.

2. 정상 기준을 저장할
   artifacts/day07/normal_reference.npz
   경로를 지정한다.

3. Pretrained ResNet18 기반
   Feature 추출기를 준비한다.

4. Normal Train의 모든 PNG 이미지를 읽는다.

5. 이미지가 하나도 없으면 실행을 중단한다.

6. 정상 이미지 한 장씩 ResNet18에 넣는다.

7. 각 이미지에서
   Feature Vector와 Feature Map을 추출한다.

8. 모든 정상 이미지의
   Feature Vector를 모은다.

9. 모든 정상 이미지의
   Feature Map도 모은다.

10. Feature Vector들의 평균을 계산한다.

11. 평균 Feature Vector를 정규화하여
    reference_vector를 만든다.

12. Feature Map들의 평균을 계산하여
    reference_map을 만든다.

13. 두 정상 기준을
    normal_reference.npz에 저장한다.

14. 만들어진 Vector와 Map의 크기,
    저장 위치를 출력한다.
```

---

# 23. `normal_reference.npz`에 저장된 정상 기준을 확인합니다

앞의 프로그램을 실행한 결과:

```text
Normal Train: 167
[1/167] 000.png
[20/167] 025.png
[40/167] 051.png
[60/167] 078.png
[80/167] 100.png
[100/167] 123.png
[120/167] 146.png
[140/167] 171.png
[160/167] 199.png
[167/167] 208.png
======================================================================
Normal Reference 생성 완료
======================================================================
Vector shape: (256,)
Map shape   : (256, 14, 14)
Saved       : artifacts/day07/normal_reference.npz
```

이 결과는 **정상 이미지 167장의 특징을 모두 추출한 뒤 평균을 계산하여 정상 기준을 만들었다**는 뜻입니다.

```text
정상 이미지 167장
        ↓
각 이미지의 Feature 추출
        ↓
모든 Feature 평균
        ↓
Normal Reference 생성 완료
```

생성된 파일은 다음입니다.

```text
artifacts/day07/normal_reference.npz
```

이 파일 안에는 두 종류의 정상 기준이 저장되어 있습니다.

## `reference_vector` — 이미지 전체 비교용 정상 기준

실행 결과:

```
Vector shape: (256,)
```

은 정상 병을 대표하는 특징값이 256개의 숫자로 정리되었다는 뜻입니다.

쉽게 생각하면:

```
정상 병 여러 장
        ↓
전체 특징을 평균
        ↓
[숫자, 숫자, 숫자, ...]
        ↓
256개의 정상 특징값
```

입니다.

이 `reference_vector`는 나중에 새로운 병의 Feature Vector와 비교합니다.

```
새로운 병 Feature Vector
        ↕
reference_vector
        ↓
거리 계산
        ↓
Anomaly Score
```

즉,

> `reference_vector`는 새로운 병이 정상 병 전체 특징과 얼마나 다른지 계산하기 위한 기준입니다.

## `reference_map` — 위치별 비교용 정상 기준

실행 결과:

```
Map shape: (256, 14, 14)
```

은 정상 특징이 하나의 숫자 목록으로만 저장된 것이 아니라 14 × 14 위치정보를 유지한 형태로도 저장되었다는 뜻입니다.

쉽게 보면:

```
정상 병
        ↓
14 × 14 영역으로 특징 유지
        ↓
각 위치에 여러 특징값 저장
```

입니다.

이 기준은 나중에:

```
새로운 병의 Feature Map
        ↕
reference_map
        ↓
위치별 차이 계산
        ↓
Heatmap
```

을 만드는 데 사용합니다.

즉,

> `reference_map`은 병의 어느 위치가 정상 기준과 다르게 보이는지 확인하기 위한 기준입니다.

## 지금까지 무엇을 만든 것인가요?

중요한 것은 ResNet18을 새로 학습한 것이 아닙니다.

```
Pretrained ResNet18
        ↓
정상 이미지 167장 Feature 추출
        ↓
Feature 평균
        ↓
정상 기준 생성
```

한 것입니다.

정리하면:

```
reference_vector
→ 이미지 전체 비교용 정상 기준

reference_map
→ 위치별 비교용 정상 기준
```

두 기준이 다음 파일에 저장되었습니다.

```
artifacts/day07/normal_reference.npz
```

> 즉 지금까지는 “정상 병은 대략 이런 특징을 가진다”라는 기준을 만든 단계입니다.

참고]
`artifacts/day07/normal_reference.npz`는 이미지 파일이나 텍스트 파일이 아니라 NumPy가 저장한 압축 데이터 파일이라서 VS Code에서 더블클릭해도 사람이 읽을 수 있는 형태로 바로 열리지 않습니다.

이 파일 안에는 앞에서 만든 두 데이터가 들어 있습니다.

```
normal_reference.npz
├── reference_vector
│   → 이미지 전체 비교용 정상 기준
│
└── reference_map
    → Heatmap용 위치별 정상 기준
```

즉 `.npz`는 여러 NumPy 배열을 한 파일에 묶어서 저장하는 형식입니다.

`normal_reference.npz`는 정상 이미지 167장에서 추출한 Feature의 평균값을 저장한 파일입니다. 사람이 직접 열어 보는 문서 파일이 아니라, 뒤의 Python 프로그램이 불러와 새로운 이미지와 비교하기 위한 데이터 파일입니다.

---

# 24. 정상 기준을 만들었지만 아직 Normal / Anomaly를 판정할 수는 없습니다

이제 정상 기준은 만들어졌습니다.

![[Group 17.png]]

하지만 아직 새로운 병을 넣고 바로:

```
Normal
또는
Anomaly
```

라고 판정할 수는 없습니다.

왜냐하면 어느 정도까지의 차이를 정상으로 허용할 것인지 기준값이 아직 없기 때문입니다.

새로운 병을 정상 기준과 비교하면 각 이미지마다 Anomaly Score가 계산됩니다.

```
새로운 병
        ↓
Feature Vector 추출
        ↓
Normal Reference와 비교
        ↓
Anomaly Score 계산
```

하지만 아직은 이 Score를 어디까지 정상으로 볼지 정하지 않았습니다.

```
Anomaly Score
        ↓
아직 판정 기준 없음
        ↓
Normal / Anomaly 결정 불가
```

즉 지금 필요한 것은:

```
어느 정도의 Score까지
정상으로 허용할 것인가?
```

를 정하는 것입니다.

이 정상과 이상을 나누는 경계값을 **Threshold**라고 합니다.

```
Anomaly Score
        ↓
Threshold와 비교
        ↓
Threshold 이하 → Normal
Threshold 초과 → Anomaly
```

따라서 다음 단계에서는 먼저 `Normal Validation` 이미지를 이용하여 정상 이미지에서도 Score가 어느 정도 범위까지 나타나는지 확인합니다.

```
Normal Validation
        ↓
각 이미지의 Anomaly Score 계산
        ↓
정상 Score 분포 확인
        ↓
Threshold 결정
```

지금 단계에서는 아직 실제 Anomaly Score 값이나 Threshold 값을 정하지 않았습니다. 다음 실습에서 Validation 데이터를 이용하여 실제 값을 계산합니다.

---

# PART4 — Normal Validation으로 Threshold 결정 + Rule Freeze (25~31)

---

# 25. Normal Validation은 왜 필요한가요?

앞에서 `Normal Train` 167장을 이용하여 정상 기준인 `Normal Reference`를 만들었습니다.

```
Normal Train 167장
        ↓
Feature 추출
        ↓
평균
        ↓
Normal Reference
```

하지만 정상 기준을 만들었다고 바로 새로운 이미지를 Normal / Anomaly로 판정할 수 있는 것은 아닙니다.

이번에는 앞에서 따로 준비한:

```
samples/day07/validation/normal
```

의 **정상 이미지 42장**을 사용합니다.

이 이미지들도 모두 정상입니다.

```
Normal Validation 42장
        ↓
각 이미지 Feature 추출
        ↓
Normal Reference와 비교
        ↓
각 이미지의 Anomaly Score 계산
```

왜 정상 이미지끼리 비교하는데 Score가 생길까요?

정상 병이라고 해도 모든 사진이 완전히 똑같지는 않기 때문입니다.

```
병의 위치가 조금 다름
표면 모양이 조금 다름
밝기가 조금 다름
촬영 상태가 조금 다름
        ↓
Feature도 조금 다름
        ↓
Anomaly Score가 0이 아닐 수 있음
```

따라서 이번 단계에서 알고 싶은 것은:

> **정상 이미지라면 Anomaly Score가 어느 정도 범위까지 나오는가?**

입니다.

아직 Threshold를 결정하지 않습니다.

먼저:

```
정상 Validation
        ↓
Score 측정
        ↓
CSV 저장
```

까지만 진행합니다.

그다음 단계에서 이 Score들을 모아 보고 Threshold 후보를 결정합니다.

---

# 26. `53_day07_measure_validation.py` — 정상 Validation Score를 측정합니다

이번 프로그램에서는 앞에서 만든 두 가지 정상 기준 중:

```
reference_vector
→ 사용

reference_map
→ 아직 사용하지 않음
```

입니다.

`reference_vector`는 **이미지 전체가 정상 기준과 얼마나 다른지 계산**할 때 사용하기 때문입니다.

흐름은 다음과 같습니다.

```
Normal Validation 이미지
        ↓
ResNet18
        ↓
Feature Vector
        ↓
Normal Reference Vector와 비교
        ↓
Euclidean Distance
        ↓
Anomaly Score
        ↓
CSV 저장
```

각 Validation 이미지는 원래 모두 정상이라는 것도 함께 기록합니다.

```
파일명
   +
Actual = Normal
   +
Anomaly Score
```

파일:

```
src/53_day07_measure_validation.py
```

작성합니다.

```python
import csv
from pathlib import Path

from day07_anomaly_core import (
    anomaly_score,
    create_backbone,
    extract_features,
    load_reference,
)


# --------------------------------------------------
# 1. 입력 / 출력 경로
# --------------------------------------------------
VAL_DIR = Path(
    "samples/day07/validation/normal"
)

REFERENCE_PATH = Path(
    "artifacts/day07/normal_reference.npz"
)

OUTPUT_CSV = Path(
    "reports/day07_validation_scores.csv"
)


# --------------------------------------------------
# 2. 필요한 데이터가 있는지 확인
# --------------------------------------------------
if not VAL_DIR.exists():
    raise FileNotFoundError(
        f"Validation 폴더를 확인하세요: {VAL_DIR}"
    )

if not REFERENCE_PATH.exists():
    raise FileNotFoundError(
        f"Normal Reference를 확인하세요: {REFERENCE_PATH}"
    )


image_files = sorted(
    VAL_DIR.glob("*.png")
)

if not image_files:
    raise RuntimeError(
        "Validation 이미지가 없습니다."
    )


# --------------------------------------------------
# 3. Normal Reference 불러오기
# --------------------------------------------------
reference_vector, _ = load_reference(
    REFERENCE_PATH
)


# --------------------------------------------------
# 4. Feature 추출용 ResNet18 준비
# --------------------------------------------------
model = create_backbone()


# --------------------------------------------------
# 5. Validation 이미지의 Score 계산
# --------------------------------------------------
rows = []

print(
    "Normal Validation:",
    len(image_files),
)

for index, image_path in enumerate(
    image_files,
    start=1,
):
    vector, _ = extract_features(
        model,
        image_path,
    )

    score = anomaly_score(
        vector,
        reference_vector,
    )

    rows.append(
        {
            "file": image_path.name,
            "actual": "Normal",
            "anomaly_score": round(
                score,
                6,
            ),
        }
    )

    if (
        index == 1
        or index % 10 == 0
        or index == len(image_files)
    ):
        print(
            f"[{index}/{len(image_files)}]",
            image_path.name,
        )


# --------------------------------------------------
# 6. CSV 저장
# --------------------------------------------------
OUTPUT_CSV.parent.mkdir(
    parents=True,
    exist_ok=True,
)

with OUTPUT_CSV.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=[
            "file",
            "actual",
            "anomaly_score",
        ],
    )

    writer.writeheader()
    writer.writerows(rows)


# --------------------------------------------------
# 7. 결과 확인
# --------------------------------------------------
print("=" * 70)
print("Validation Score 측정 완료")
print("=" * 70)

print(
    "Images:",
    len(rows),
)

print(
    "Saved :",
    OUTPUT_CSV,
)
```

실행합니다.

```
python src/53_day07_measure_validation.py
```

정상적으로 실행되면 다음과 같이 출력됩니다.

```
Normal Validation: 42
[1/42] 001.png
[10/42] 039.png
[20/42] 086.png
[30/42] 154.png
[40/42] 197.png
[42/42] 206.png
======================================================================
Validation Score 측정 완료
======================================================================
Images: 42
Saved : reports/day07_validation_scores.csv
```

즉 앞에서 분리한 Normal Validation 정상 이미지 42장을 모두 검사했다는 뜻입니다.

```
Normal Validation 42장
        ↓
각 이미지에서 Feature Vector 추출
        ↓
Normal Reference와 비교
        ↓
Anomaly Score 42개 계산
        ↓
CSV 저장
```

결과는 다음 파일에 저장됩니다.

```
reports/day07_validation_scores.csv
```

파일을 열어 보면 다음과 같이 각 정상 이미지마다 서로 다른 Score가 기록되어 있습니다.

```
file,actual,anomaly_score
001.png,Normal,0.039874
006.png,Normal,0.065865
007.png,Normal,0.039087
008.png,Normal,0.055498
...
205.png,Normal,0.070818
206.png,Normal,0.043039
```

각 컬럼의 의미는 다음과 같습니다.

| 컬럼              | 의미                               |
| --------------- | -------------------------------- |
| `file`          | 검사한 Validation 이미지               |
| `actual`        | 실제 정답                            |
| `anomaly_score` | Normal Reference와 얼마나 다른지를 나타낸 값 |
여기서 모든 이미지의 `actual`은:

```
Normal
```

입니다.

하지만 `anomaly_score`는 모두 같지 않습니다.

실제 측정 결과를 보면 Score는 대략 다음 범위에 있습니다.

```
가장 낮은 Score
0.035927

평균 Score
약 0.050680

가장 높은 Score
0.070818
```

즉 정상 이미지라고 해서:

```
Score = 0
```

이 되는 것이 아닙니다.

실제로 정상 이미지 42장에서도:

```
0.035927
0.039874
0.050599
0.065865
0.070818
...
```

처럼 서로 다른 Score가 나왔습니다.

왜냐하면 같은 정상 병이라도 이미지마다:

```
병의 위치
밝기
표면 상태
촬영 상태
세부적인 모양
```

이 조금씩 다르기 때문입니다.

그래서 현재 결과에서 중요한 것은 어떤 이미지의 Score가 가장 높았는지 자체가 아니라, 정상 이미지의 Score가 어느 정도 범위에 분포하는지를 확인하는 것입니다.

```
Normal Validation 42장

낮은 Score
        ↓
대부분의 정상 Score
        ↓
높은 정상 Score

약 0.036
        ↓
약 0.051 부근
        ↓
약 0.071
```

현재 가장 높은 정상 Score는:

```
205.png
→ 0.070818
```

이고 가장 낮은 정상 Score는:

```
050.png
→ 0.035927
```

입니다.

따라서 이제 다음 질문을 할 수 있습니다.

> 정상 이미지의 Score가 이 정도 범위에서 나오는데, Normal(정상)과 Anomaly(비정상)를 나누는 Threshold를 어디에 둘 것인가?

아직 이 단계에서는:

```
Normal / Anomaly 최종 판정 (아직 불가능)

Threshold 확정 (아직 불가능)
```

입니다.

지금까지 한 것은:

```
정상 이미지 42장
        ↓
실제 Anomaly Score 측정
        ↓
정상 Score 데이터 확보
        ↓
CSV 저장
```

까지입니다.

### 다음단계

![[Group 18.png]]

다음 단계에서는 이 42개의 실제 Score를 이용하여:

```
90 Percentile
95 Percentile
99 Percentile
```

값을 계산하고 **Threshold 후보를 비교**합니다.

`Percentile`은 **백분위수**라고 합니다.

쉽게 말하면 데이터를 작은 값부터 큰 값까지 줄 세웠을 때 어느 위치의 값을 선택할 것인가입니다.

예를 들어 정상 Validation Score가 여러 개 있다고 하면:

```
낮은 Score                               높은 Score
0.03  0.04  0.04  0.05  0.05 ... 0.06  0.07
 ───────────────────────────────────────────→
```

### 90 Percentile

`90 Percentile`은:

> 전체 데이터의 약 90%가 이 값 이하에 오도록 하는 경계값

입니다.

```
정상 Score 전체
─────────────────────────────────────
약 90%                         약 10%
████████████████████████████████│████
                                ↑
                         90 Percentile
```

예를 들어 90 Percentile이 `0.065`라면:

```
정상 Validation Score의 약 90%
→ 0.065 이하

나머지 약 10%
→ 0.065보다 높음
```

이라는 뜻입니다.

현재 실제 데이터에서는 다음 단계에서 대략:

```
90 Percentile → 0.065345 부근
95 Percentile → 0.067469 부근
99 Percentile → 0.069548 부근
```

이 계산될 것입니다.

따라서 흐름은 이제 아주 명확합니다.

```
Normal Train 167장
(정상 학습용 데이터)
        ↓

Normal Reference 생성
(정상 참조 기준 만들기)
        ↓

Normal Validation 42장
(정상 검증용 데이터)
        ↓

Anomaly Score 42개 측정
(정상 기준과 얼마나 다른지 점수 계산)
        ↓

정상 Score 분포 확인
(정상에서는 Score가 어느 정도까지 나오는가?)
        ↓

Threshold 후보 비교
(정상 / 이상을 나눌 임계값 후보 비교)
        ↓

Threshold 결정
        ↓

Rule Freeze
        ↓

그다음에 최종 Test
```

---

## 의사코드

```
1. Normal Validation 이미지가 들어 있는
   samples/day07/validation/normal
   경로를 지정한다.

2. 앞에서 만든 Normal Reference 파일
   normal_reference.npz의 위치를 지정한다.

3. Validation 폴더와
   Normal Reference 파일이 실제로 있는지 확인한다.

4. Validation 이미지 목록을 읽는다.

5. Validation 이미지가 하나도 없으면
   실행을 중단한다.

6. normal_reference.npz에서
   reference_vector를 불러온다.

7. Feature 추출용
   Pretrained ResNet18을 준비한다.

8. Validation 정상 이미지를
   한 장씩 읽는다.

9. 각 이미지에서
   Feature Vector를 추출한다.

10. Validation Feature Vector와
    Normal Reference Vector의 거리를 계산한다.

11. 계산된 거리를
    Anomaly Score로 사용한다.

12. 각 이미지에 대해

    파일명
    실제 정답 = Normal
    Anomaly Score

    를 기록한다.

13. 모든 Validation 이미지의 결과를
    day07_validation_scores.csv에 저장한다.

14. 처리한 이미지 수와
    CSV 저장 위치를 출력한다.
```

전체 흐름만 아주 간단하게 보면:

```
Normal Validation 42장
        ↓
각 이미지 Feature 추출
        ↓
Normal Reference와 비교
        ↓
Anomaly Score 42개 생성
        ↓
CSV 저장
```
---

# 27. Normal Validation의 Score 분포를 그래프로 확인합니다

앞에서 정상 Validation 이미지 42장의 Anomaly Score를 계산하여 다음 파일에 저장했습니다.

```
reports/day07_validation_scores.csv
```

먼저 일부 데이터를 확인합니다.

```
head reports/day07_validation_scores.csv
```

실제 결과는 다음과 같습니다.

```
file,actual,anomaly_score
001.png,Normal,0.039874
006.png,Normal,0.065865
007.png,Normal,0.039087
008.png,Normal,0.055498
022.png,Normal,0.042966
...
```

하지만 숫자를 한 줄씩 보는 것만으로는 **정상 Score가 어느 범위에 많이 모여 있는지** 파악하기 어렵습니다.

그래서 Matplotlib을 이용하여 Score 분포를 그래프로 확인합니다.

```
CSV의 Score 42개
        ↓
Matplotlib
        ↓
Histogram
        ↓
정상 Score가
어느 구간에 많이 모이는지 확인
```

## Score 분포 그래프를 만듭니다

다음 코드를 실행합니다.

```bash
python - <<'PY'
from pathlib import Path

import matplotlib.pyplot as plt
import pandas as pd


INPUT_CSV = Path(
    "reports/day07_validation_scores.csv"
)

OUTPUT_PATH = Path(
    "reports/day07_validation_score_distribution.png"
)


df = pd.read_csv(INPUT_CSV)

scores = df[
    "anomaly_score"
]


plt.figure(
    figsize=(9, 5)
)

plt.hist(
    scores,
    bins=10,
    edgecolor="black",
)

plt.xlabel(
    "Anomaly Score"
)

plt.ylabel(
    "Number of Normal Images"
)

plt.title(
    "Day07 Normal Validation Score Distribution"
)

plt.tight_layout()

plt.savefig(
    OUTPUT_PATH,
    dpi=150,
)

plt.close()


print(
    "Saved:",
    OUTPUT_PATH,
)
PY
```

실행이 끝나면 다음 파일이 만들어집니다.

```
reports/day07_validation_score_distribution.png
```

VS Code에서 이미지를 열어 확인합니다.

---

### 그래프 결과를 확인합니다.

![[Pasted image 20260902173016.png]]

생성된 그래프를 보면 정상 Validation 42장의 Anomaly Score가 모두 같은 값이 아니라 여러 구간에 분포하고 있습니다.

```
가장 낮은 Score
→ 0.035927

평균 Score
→ 약 0.050680

가장 높은 Score
→ 0.070818
```

그래프의 의미는 다음과 같습니다.

```
가로축
→ Anomaly Score
→ 오른쪽으로 갈수록 Normal Reference와 더 다름

세로축
→ 해당 Score 구간에 있는 정상 이미지 수
```

실제 그래프를 보면 많은 정상 이미지가 대략 0.04 ~ 0.056 부근에 모여 있고, 일부 정상 이미지는 0.065 ~ 0.071 정도의 높은 Score도 가지고 있습니다.

즉 중요한 점은:

> 정상 이미지라고 해서 Anomaly Score가 0이 되는 것은 아니며, 정상 데이터 안에서도 Score의 차이가 존재합니다.

따라서 가장 높은 정상 Score 하나만 보고 기준을 정하는 것이 아니라, 정상 Score 전체의 분포를 보고 어디까지 정상으로 허용할지 결정해야 합니다.

```
정상 Score 42개
        ↓
분포 확인 완료
        ↓
어디까지 정상으로 허용할까?
        ↓
다음 단계
90 / 95 / 99 Percentile 비교
        ↓
Threshold 후보 결정
```

현재 단계에서는 아직 Threshold를 확정하거나 Normal / Anomaly 최종 판정을 하지 않습니다. 
다음 단계에서 Percentile을 이용해 Threshold 후보를 비교합니다.

---

# 28. `54_day07_compare_thresholds.py` — Threshold 후보를 비교합니다

앞에서 정상 Validation 이미지 42장의 Anomaly Score 분포를 확인했습니다.

이제 그 Score를 이용하여 다음 세 가지 Threshold 후보를 계산합니다.

```
90 Percentile
95 Percentile
99 Percentile
```

이번 단계의 목적은 아직 Threshold를 확정하는 것이 아니라 후보값을 숫자로 비교하는 것입니다.

```
Validation Score 42개
        ↓
90 / 95 / 99 Percentile 계산
        ↓
Threshold 후보 3개
        ↓
각 후보에서
정상 이미지가 몇 장 넘어가는지 비교
```

파일:

```
src/54_day07_compare_thresholds.py
```

작성합니다.

```python
from pathlib import Path

import numpy as np
import pandas as pd


# --------------------------------------------------
# 1. 입력 / 출력 파일
# --------------------------------------------------
INPUT_CSV = Path(
    "reports/day07_validation_scores.csv"
)

OUTPUT_CSV = Path(
    "reports/day07_threshold_compare.csv"
)


# --------------------------------------------------
# 2. Validation Score 불러오기
# --------------------------------------------------
df = pd.read_csv(
    INPUT_CSV
)

scores = df[
    "anomaly_score"
].to_numpy()


# --------------------------------------------------
# 3. Threshold 후보 계산
# --------------------------------------------------
candidates = []

for percentile in [
    90,
    95,
    99,
]:
    threshold = float(
        np.percentile(
            scores,
            percentile,
        )
    )

    # 해당 Threshold보다 높은 정상 이미지 수
    normal_over = int(
        (
            scores
            > threshold
        ).sum()
    )

    candidates.append(
        {
            "percentile": percentile,
            "threshold": round(
                threshold,
                6,
            ),
            "validation_normal_over_threshold": normal_over,
        }
    )


# --------------------------------------------------
# 4. 결과를 표 형태로 만들기
# --------------------------------------------------
result = pd.DataFrame(
    candidates
)


# --------------------------------------------------
# 5. CSV 저장
# --------------------------------------------------
OUTPUT_CSV.parent.mkdir(
    parents=True,
    exist_ok=True,
)

result.to_csv(
    OUTPUT_CSV,
    index=False,
    encoding="utf-8-sig",
)


# --------------------------------------------------
# 6. 결과 출력
# --------------------------------------------------
print("=" * 70)
print("Validation Score Summary")
print("=" * 70)

print(
    "Count:",
    len(scores),
)

print(
    "Min  :",
    round(
        float(scores.min()),
        6,
    ),
)

print(
    "Mean :",
    round(
        float(scores.mean()),
        6,
    ),
)

print(
    "Max  :",
    round(
        float(scores.max()),
        6,
    ),
)

print()

print(
    result.to_string(
        index=False
    )
)

print()

print(
    "Saved:",
    OUTPUT_CSV,
)
```

실행합니다.

```
python src/54_day07_compare_thresholds.py
```

---

## 실제 실행 결과

앞에서 측정한 정상 Validation 42장의 Score를 이용하면 다음과 같이 계산됩니다.

```
======================================================================
Validation Score Summary
======================================================================
Count: 42
Min  : 0.035927
Mean : 0.05068
Max  : 0.070818

 percentile  threshold  validation_normal_over_threshold
         90   0.065345                                 5
         95   0.067469                                 3
         99   0.069548                                 1

Saved: reports/day07_threshold_compare.csv
```

결과 파일은 다음 위치에 저장됩니다.

```
reports/day07_threshold_compare.csv
```

---

## 결과는 어떻게 읽나요?

세 후보를 간단히 비교하면 다음과 같습니다.

|후보|Threshold|Threshold보다 높은 정상 이미지|
|---|---|---|
|90 Percentile|`0.065345`|5장|
|95 Percentile|`0.067469`|3장|
|99 Percentile|`0.069548`|1장|
![[Pasted image 20260902175154.png]]

예를 들어 `90 Percentile`을 기준으로 사용한다고 가정하면:

```
Threshold = 0.065345
        ↓
정상 Validation 중
이 값보다 높은 이미지 = 5장
```

입니다.

반대로 Threshold를 높이면:

```
90 Percentile
0.065345
→ 5장 초과

95 Percentile
0.067469
→ 3장 초과

99 Percentile
0.069548
→ 1장 초과
```

처럼 정상인데 Threshold를 넘는 이미지 수가 줄어듭니다.

아직 어느 값을 최종 Threshold로 사용할지는 결정하지 않습니다.

```
90 Percentile
95 Percentile
99 Percentile
        ↓
후보 비교 완료
        ↓
다음 단계
어떤 값을 Baseline으로 사용할지 선택
```

---

## 의사코드

```
1. 앞에서 만든
   day07_validation_scores.csv를 읽는다.

2. CSV에서
   anomaly_score 42개를 가져온다.

3. 정상 Score의
   최소값 / 평균값 / 최대값을 확인한다.

4. Score 42개에서
   90 Percentile 값을 계산한다.

5. 그 값을 Threshold로 했을 때
   Threshold보다 높은 정상 이미지 수를 센다.

6. 같은 방법으로
   95 Percentile을 계산한다.

7. 같은 방법으로
   99 Percentile을 계산한다.

8. 세 Threshold 후보와
   각 후보를 넘는 정상 이미지 수를 표로 만든다.

9. 결과를
   day07_threshold_compare.csv에 저장한다.

10. 세 후보를 비교하고
    다음 단계에서 사용할 Threshold를 선택할 준비를 한다.
```

핵심 흐름만 보면:

```
정상 Score 42개
        ↓
90 / 95 / 99 Percentile
        ↓
0.065345
0.067469
0.069548
        ↓
각 기준을 넘는 정상 이미지
5장 / 3장 / 1장
        ↓
Threshold 후보 비교
```
---

# 29. 세 후보 중 95 Percentile을 Baseline으로 선택합니다

앞에서 세 가지 Threshold 후보를 비교했습니다.

|후보|Threshold|Threshold보다 높은 정상 이미지|
|---|---|---|
|90 Percentile|`0.065345`|5장|
|95 Percentile|`0.067469`|3장|
|99 Percentile|`0.069548`|1장|

세 값을 비교하면 다음과 같습니다.

```
90 Percentile
→ Threshold가 가장 낮음
→ 정상 이미지 5장이 기준 초과
→ 정상에도 비교적 엄격한 기준

95 Percentile
→ 세 후보의 중간
→ 정상 이미지 3장이 기준 초과

99 Percentile
→ Threshold가 가장 높음
→ 정상 이미지 1장만 기준 초과
→ 정상에는 비교적 느슨한 기준
```

그렇다면 어떤 값을 선택해야 할까요?

현재는 아직 실제 Anomaly Test를 사용하지 않았기 때문에:

```
어떤 Threshold가
실제 결함까지 가장 잘 구분하는가?
```

를 알 수 없습니다.

따라서 오늘 실습에서는 **세 후보 중 중간 수준인 `95 Percentile`을 교육용 Baseline으로 선택**합니다.

```
90 Percentile
조금 엄격함
        ↓

95 Percentile
중간 기준
★ 오늘의 Baseline

        ↓

99 Percentile
조금 느슨함
```

오늘 사용할 Threshold는 실제 계산 결과:

```
95 Percentile
        ↓
Threshold = 0.067469
```

입니다.

따라서 앞으로의 판정 규칙은:

```
Anomaly Score <= 0.067469
→ Normal

Anomaly Score > 0.067469
→ Anomaly
```

로 정합니다.

여기서 중요한 점은 **`0.067469`가 가장 좋은 Threshold라고 증명된 것은 아니라는 것**입니다.

오늘은:

```
정상 Validation Score 분포 확인
        ↓
90 / 95 / 99 후보 비교
        ↓
95 Percentile을
교육용 Baseline으로 선택
        ↓
Rule Freeze
        ↓
처음 보는 Test 데이터에서 성능 확인
```

하는 순서로 진행합니다.

> `95 Percentile`은 산업 표준이나 정답이 아닙니다. 오늘은 Threshold를 정하고 Freeze한 뒤 Test에서 결과를 확인하는 전체 과정을 학습하기 위해 선택한 Baseline입니다.

실제 현장에서는 정상 오탐(FP), 불량 미검출(FN), 제품의 품질 기준, 검사 목적 등을 함께 고려하여 Threshold를 결정합니다.

---

# 30. `55_day07_freeze_anomaly_rule.py` — 선택한 Threshold를 Rule로 고정합니다

앞 단계에서 세 가지 Threshold 후보를 비교했습니다.

```
90 Percentile → 0.065345
95 Percentile → 0.067469
99 Percentile → 0.069548
```

그리고 오늘의 교육용 Baseline으로:

```
95 Percentile
        ↓
Threshold = 0.067469
```

을 선택했습니다.

이제 이 값을 코드 안에서 계속 다시 정하는 것이 아니라 **설정 파일에 저장하여 이후 Test에서 동일한 기준을 사용**합니다.

```
95 Percentile 선택
        ↓
Threshold = 0.067469
        ↓
판정 Rule과 함께 JSON 저장
        ↓
Rule Freeze
        ↓
이후 Test에서는 이 Rule 그대로 사용
```

> 여기서 `Freeze`란 파일을 기술적으로 수정 불가능하게 만든다는 뜻이 아닙니다. **이번 Baseline 평가가 끝날 때까지 기준을 바꾸지 않고 동일하게 사용한다는 의미**입니다.

파일:

```
src/55_day07_freeze_anomaly_rule.py
```

작성합니다.

```python
import json
from pathlib import Path

import numpy as np
import pandas as pd


# --------------------------------------------------
# 1. 입력 / 출력 경로
# --------------------------------------------------
VALIDATION_CSV = Path(
    "reports/day07_validation_scores.csv"
)

RULE_PATH = Path(
    "configs/day07_anomaly_rules.json"
)


# --------------------------------------------------
# 2. 오늘 선택한 Threshold 기준
# --------------------------------------------------
SELECTED_PERCENTILE = 95


# --------------------------------------------------
# 3. Validation Score 확인
# --------------------------------------------------
if not VALIDATION_CSV.exists():
    raise FileNotFoundError(
        f"Validation Score 파일을 확인하세요: {VALIDATION_CSV}"
    )


df = pd.read_csv(
    VALIDATION_CSV
)

if "anomaly_score" not in df.columns:
    raise RuntimeError(
        "anomaly_score 컬럼이 없습니다."
    )

scores = df[
    "anomaly_score"
].to_numpy()

if len(scores) == 0:
    raise RuntimeError(
        "Validation Score가 없습니다."
    )


# --------------------------------------------------
# 4. 선택한 95 Percentile Threshold 계산
# --------------------------------------------------
threshold = round(
    float(
        np.percentile(
            scores,
            SELECTED_PERCENTILE,
        )
    ),
    6,
)


# --------------------------------------------------
# 5. Baseline Rule 구성
# --------------------------------------------------
rules = {
    "baseline_version": "day07-anomaly-v1.0",
    "dataset": "MVTec AD bottle",
    "backbone": "resnet18_imagenet_pretrained",
    "feature_layer": "layer3",
    "reference": "mean_normal_feature",
    "reference_path": "artifacts/day07/normal_reference.npz",
    "image_size": 224,
    "selected_percentile": SELECTED_PERCENTILE,
    "threshold_method": "normal_validation_95_percentile",
    "anomaly_threshold": threshold,
    "decision_rule": "score > threshold => Anomaly",
}


# --------------------------------------------------
# 6. Rule JSON 저장
# --------------------------------------------------
RULE_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)

RULE_PATH.write_text(
    json.dumps(
        rules,
        indent=2,
        ensure_ascii=False,
    ),
    encoding="utf-8",
)


# --------------------------------------------------
# 7. 결과 확인
# --------------------------------------------------
print("=" * 70)
print("DAY07 RULE FREEZE")
print("=" * 70)

print(
    "Version   :",
    rules["baseline_version"],
)

print(
    "Percentile:",
    rules["selected_percentile"],
)

print(
    "Threshold :",
    rules["anomaly_threshold"],
)

print(
    "Rule      :",
    rules["decision_rule"],
)

print(
    "Saved     :",
    RULE_PATH,
)
```

실행합니다.

```
python src/55_day07_freeze_anomaly_rule.py
```

현재 실제 Validation 결과를 사용하면 다음과 같이 나와야 합니다.

```
======================================================================
DAY07 RULE FREEZE
======================================================================
Version   : day07-anomaly-v1.0
Percentile: 95
Threshold : 0.067469
Rule      : score > threshold => Anomaly
Saved     : configs/day07_anomaly_rules.json
```

그리고 다음 파일이 생성됩니다.

```
configs/day07_anomaly_rules.json
```

파일을 열어보면 대략 다음과 같습니다.

```json
{
  "baseline_version": "day07-anomaly-v1.0",
  "dataset": "MVTec AD bottle",
  "backbone": "resnet18_imagenet_pretrained",
  "feature_layer": "layer3",
  "reference": "mean_normal_feature",
  "reference_path": "artifacts/day07/normal_reference.npz",
  "image_size": 224,
  "selected_percentile": 95,
  "threshold_method": "normal_validation_95_percentile",
  "anomaly_threshold": 0.067469,
  "decision_rule": "score > threshold => Anomaly"
}
```

이 파일에서 가장 중요한 부분은 다음입니다.

```
anomaly_threshold
→ 0.067469

decision_rule
→ score > threshold => Anomaly
```

따라서 이제부터의 판정 기준은:

```
Score <= 0.067469
→ Normal
→ 정상

Score > 0.067469
→ Anomaly
→ 이상
```

으로 고정됩니다.

### 의사코드

```
1. 앞에서 만든
   Validation Score CSV의 위치를 지정한다.

2. Rule을 저장할
   JSON 파일의 위치를 지정한다.

3. 앞에서 선택한
   95 Percentile을 사용할 값으로 지정한다.

4. Validation Score CSV를 읽는다.

5. anomaly_score 값 42개를 가져온다.

6. 42개의 Score에서
   95 Percentile 값을 계산한다.

7. 계산 결과를 소수점 6자리로 정리한다.

8. 실제 Threshold인
   0.067469를 얻는다.

9. ResNet18, layer3,
   Normal Reference,
   Threshold 등의 정보를
   하나의 Rule로 묶는다.

10. 판정 조건도 저장한다.

    Score <= Threshold
    → Normal

    Score > Threshold
    → Anomaly

11. 모든 설정을
    day07_anomaly_rules.json에 저장한다.

12. 저장한 Version과
    Threshold를 출력한다.

13. 이후 Test에서는
    이 Rule을 다시 계산하거나 변경하지 않고
    그대로 사용한다.
```

전체 흐름만 보면 더 간단합니다.

```
Validation Score 42개
        ↓
95 Percentile 선택
        ↓
Threshold = 0.067469
        ↓
판정 Rule 작성
        ↓
day07_anomaly_rules.json 저장
        ↓
Rule Freeze 완료
        ↓
이제 Test로 이동
```
---

# 31. 확정한 Rule을 그대로 Test에 적용합니다

앞 단계에서 오늘 사용할 판정 기준을 확정했습니다.

```python
Threshold = 0.067469
```

따라서 앞으로의 판정 Rule은 다음과 같습니다.

```text
Score <= 0.067469
→ Normal
→ 정상

Score > 0.067469
→ Anomaly
→ 이상
```

그리고 이 기준을 다음 파일에 저장했습니다.

```
configs/day07_anomaly_rules.json
```

이제 이 Rule은 `day07-anomaly-v1.0`으로 고정합니다.

```
Normal Train 167장
        ↓
Normal Reference 생성
        ↓

Normal Validation 42장
        ↓
Score 분포 확인
        ↓
90 / 95 / 99 Percentile 비교
        ↓
95 Percentile 선택
        ↓
Threshold = 0.067469
        ↓
Rule Freeze
day07-anomaly-v1.0
        ↓
이제 Test 시작
```

여기서 `Rule Freeze`의 의미는:

> Test 결과를 본 뒤 현재 Threshold를 마음대로 다시 바꾸지 않는 것입니다.

왜 그럴까요?

Test 데이터는 지금까지 기준을 만드는 데 사용하지 않은 처음 보는 데이터이기 때문입니다.

```
Test 실행
        ↓
결과가 좋음
→ 현재 Rule의 성능 확인

결과가 좋지 않음
→ Threshold를 바로 변경하지 않음
→ 어떤 이미지에서 실패했는지 분석
```

즉 Test 결과가 좋지 않더라도:

```
Test 결과가 나쁨
        ↓
Threshold 즉시 변경 
X
```

하지 않습니다.

대신:

```
Test 결과가 나쁨
        ↓
FP / FN 확인
        ↓
실패 원인 분석
        ↓
다음 버전에서 개선
```

합니다.

예를 들어 개선이 필요하다면 이후에는:

```
day07-anomaly-v1.0
        ↓
실패 분석
        ↓
새로운 기준 검토
        ↓
day07-anomaly-v1.1
```

처럼 새로운 버전으로 개선하는 것이 올바른 흐름입니다.

### 만약에 선택한 95의 결과가 나빴다면 (다시 정리하면) 

90이나 99를 다시 시도할 수는 있습니다. 
다만 핵심은 Test 결과를 보고 가장 잘 나온 Threshold를 골라버리면 안 된다는 것입니다.

쉽게 이해하면 이 흐름입니다.

![[Pasted image 20260902181938.png]]

```
Validation에서
90 / 95 / 99 후보 비교
        ↓
95 선택
        ↓
Rule Freeze
v1.0
        ↓
Test 실행
        ↓
결과가 좋지 않음
        ↓
FP / FN 원인 확인
        ↓
개발 단계로 돌아감
        ↓
Validation 기준으로
90 또는 99 등 다시 검토
        ↓
새 Rule 결정
        ↓
v1.1로 Freeze
        ↓
새로운 Test에서 다시 평가
```

즉 Test가 안 좋다고 그 자리에서 95 → 90 → 99를 계속 바꾸면서 가장 잘 나오는 것을 고르는 것은 피합니다.

왜냐하면 그렇게 하면 Test 데이터가 사실상 Threshold를 고르는 데 사용된 것이 되기 때문입니다.

아주 쉽게 구분하면:

```
Validation
→ 어떤 기준을 사용할지 결정하는 곳

Test
→ 정한 기준이 진짜 잘 되는지 확인하는 곳
```

예를 들어 v1.0에서 결과를 분석했더니:

```
FN이 너무 많다(모델이 맞다(Positive)라고 예측했는데, 실제로는 아니다(Negative/False)인 경우)
→ 이상을 너무 많이 놓침
→ Threshold가 너무 높을 가능성
→ 90 Percentile 같은 더 낮은 기준 검토

FP가 너무 많다(모델이 아니다(Negative)라고 예측했는데, 실제로는 맞다(Positive/False)인 경우)
→ 정상도 이상으로 많이 판정
→ Threshold가 너무 낮을 가능성
→ 99 Percentile 같은 더 높은 기준 검토
```

할 수 있습니다.

그러면 바로 기존 Test에서 값을 바꾸는 것이 아니라:

```
v1.0 Test 결과
        ↓
문제 발견
        ↓
Validation으로 돌아가 기준 재검토
        ↓
새 Threshold 결정
        ↓
v1.1
```

로 가는 것입니다.

> 지금부터는 Threshold를 만드는 단계가 끝났습니다. 이제 `0.067469`라는 동일한 기준을 처음 보는 Normal / Anomaly Test 데이터에 적용하여 실제 성능을 확인합니다.

---

# PART5 — Normal / Anomaly Test + 성능 평가 (32~38)

---

# 32. Test에서 사용할 데이터

```text
samples/day07/test/
├── normal/
└── anomaly/
    ├── broken_large/
    ├── broken_small/
    └── contamination/
```

각 폴더의 의미는 다음과 같습니다.

```
normal
→ 실제 정상 이미지

broken_large
→ 크게 파손된 이상 이미지

broken_small
→ 작게 파손된 이상 이미지

contamination
→ 오염이 있는 이상 이미지
```

중요한 점은 이 Test 이미지들을 앞에서 기준을 만드는 과정에 사용하지 않았다는 것입니다.

```
Normal Train
→ Normal Reference 생성에 사용

Normal Validation
→ Threshold 결정에 사용

Test Normal / Anomaly
→ 지금까지 사용하지 않음
```

즉 Test 데이터는 검사기 입장에서 처음 보는 이미지입니다.

```
Train / Validation으로
검사 기준 만들기
        ↓
Rule Freeze
Threshold = 0.067469
        ↓
처음 보는 Test 이미지 입력
        ↓
Normal / Anomaly 판정
        ↓
실제 정답과 비교
        ↓
검사기가 얼마나 잘 동작하는지 확인
```

Test 데이터는 기준을 만드는 데이터가 아니라, 이미 만들어진 검사 기준이 새로운 이미지에서도 제대로 동작하는지 확인하기 위한 데이터입니다.

---

# 33. `56_day07_test_anomaly.py` — Test 전체를 Batch로 검사합니다

```
한 장만 검사
→ 이미지 1장을 넣고 결과 확인

Batch 검사
→ 폴더 안의 여러 이미지를
   한 장씩 순서대로 자동 검사
```

이제 Freeze한 Rule을 Test 이미지 전체에 한 번에 적용합니다.

각 이미지마다 다음 정보를 기록합니다.

```
실제 정답
Anomaly Score
예측 결과
TP / TN / FP / FN
```

기억해두기

```
TP → 이상을 맞게 잡음
TN → 정상을 맞게 통과시킴
FP → 정상인데 이상이라고 함
FN → 이상인데 정상이라고 함

TP → 실제 이상 / 예측 이상
TN → 실제 정상 / 예측 정상
FP → 실제 정상 / 예측 이상
FN → 실제 이상 / 예측 정상

Positive = Anomaly = 이상
Negative = Normal = 정상

TP, TN → 맞게 판정 
FP, FN → 잘못 판정
```

즉 이번 코드는 Test 이미지를 실제로 판정하고 그 결과를 CSV로 남기는 단계입니다.

파일:

```
src/56_day07_test_anomaly.py
```

작성합니다.

```python
import csv
import json
from pathlib import Path

from day07_anomaly_core import (
    anomaly_score,
    create_backbone,
    extract_features,
    load_reference,
)


# --------------------------------------------------
# 1. 입력 / 출력 경로
# --------------------------------------------------
TEST_NORMAL_DIR = Path(
    "samples/day07/test/normal"
)

TEST_ANOMALY_DIR = Path(
    "samples/day07/test/anomaly"
)

REFERENCE_PATH = Path(
    "artifacts/day07/normal_reference.npz"
)

RULE_PATH = Path(
    "configs/day07_anomaly_rules.json"
)

OUTPUT_CSV = Path(
    "reports/day07_anomaly_scores.csv"
)


# --------------------------------------------------
# 2. 필요한 파일 / 폴더 확인
# --------------------------------------------------
required_paths = [
    TEST_NORMAL_DIR,
    TEST_ANOMALY_DIR,
    REFERENCE_PATH,
    RULE_PATH,
]

for path in required_paths:
    if not path.exists():
        raise FileNotFoundError(
            f"필요한 파일 또는 폴더를 확인하세요: {path}"
        )


# --------------------------------------------------
# 3. Freeze한 Rule 불러오기
# --------------------------------------------------
rules = json.loads(
    RULE_PATH.read_text(
        encoding="utf-8"
    )
)

threshold = float(
    rules["anomaly_threshold"]
)


# --------------------------------------------------
# 4. Normal Reference와 모델 준비
# --------------------------------------------------
reference_vector, _ = load_reference(
    REFERENCE_PATH
)

model = create_backbone()

rows = []


# --------------------------------------------------
# 5. 이미지 한 장을 검사하는 함수
# --------------------------------------------------
def inspect_image(
    image_path,
    actual,
    defect_type,
):
    vector, _ = extract_features(
        model,
        image_path,
    )

    score = anomaly_score(
        vector,
        reference_vector,
    )

    predicted = (
        "Anomaly"
        if score > threshold
        else "Normal"
    )


    # TP / TN / FP / FN 구분
    if (
        actual == "Anomaly"
        and predicted == "Anomaly"
    ):
        error_type = "TP"

    elif (
        actual == "Normal"
        and predicted == "Normal"
    ):
        error_type = "TN"

    elif (
        actual == "Normal"
        and predicted == "Anomaly"
    ):
        error_type = "FP"

    else:
        error_type = "FN"


    rows.append(
        {
            "file": str(image_path),
            "actual": actual,
            "defect_type": defect_type,
            "anomaly_score": round(
                score,
                6,
            ),
            "threshold": round(
                threshold,
                6,
            ),
            "predicted": predicted,
            "error_type": error_type,
            "correct": actual == predicted,
        }
    )


# --------------------------------------------------
# 6. Normal Test 검사
# --------------------------------------------------
for image_path in sorted(
    TEST_NORMAL_DIR.glob("*.png")
):
    inspect_image(
        image_path,
        actual="Normal",
        defect_type="good",
    )


# --------------------------------------------------
# 7. Anomaly Test 검사
# --------------------------------------------------
for defect_dir in sorted(
    TEST_ANOMALY_DIR.iterdir()
):
    if not defect_dir.is_dir():
        continue

    for image_path in sorted(
        defect_dir.glob("*.png")
    ):
        inspect_image(
            image_path,
            actual="Anomaly",
            defect_type=defect_dir.name,
        )


# --------------------------------------------------
# 8. 결과 확인
# --------------------------------------------------
if not rows:
    raise RuntimeError(
        "검사할 Test 이미지가 없습니다."
    )


# --------------------------------------------------
# 9. CSV 저장
# --------------------------------------------------
OUTPUT_CSV.parent.mkdir(
    parents=True,
    exist_ok=True,
)

with OUTPUT_CSV.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:
    writer = csv.DictWriter(
        file,
        fieldnames=[
            "file",
            "actual",
            "defect_type",
            "anomaly_score",
            "threshold",
            "predicted",
            "error_type",
            "correct",
        ],
    )

    writer.writeheader()
    writer.writerows(rows)


# --------------------------------------------------
# 10. 결과 출력
# --------------------------------------------------
print("=" * 70)
print("DAY07 TEST COMPLETE")
print("=" * 70)

print(
    "Threshold:",
    threshold,
)

print(
    "Images   :",
    len(rows),
)

print(
    "Saved    :",
    OUTPUT_CSV,
)
```

실행합니다.

```
python src/56_day07_test_anomaly.py
```

결과는 다음 파일에 저장됩니다.

```
reports/day07_anomaly_scores.csv
```

이 CSV에는 각 Test 이미지에 대해 다음과 같은 정보가 기록됩니다.

```
file
actual
defect_type
anomaly_score
threshold
predicted
error_type
correct
```

---

## 의사코드

```
1. Test Normal 폴더와
   Test Anomaly 폴더를 지정한다.

2. 앞에서 만든
   Normal Reference를 준비한다.

3. 앞에서 Freeze한
   day07_anomaly_rules.json을 읽는다.

4. JSON에서
   Threshold = 0.067469를 가져온다.

5. ResNet18 Feature Extractor를 준비한다.


6. Normal Test 이미지를 한 장씩 읽는다.

7. 이미지에서 Feature Vector를 뽑는다.

8. Normal Reference와 비교하여
   Anomaly Score를 계산한다.

9. Score와 Threshold를 비교한다.

   Score <= 0.067469
   → Normal

   Score > 0.067469
   → Anomaly


10. 실제 정답과 예측 결과를 비교한다.

    실제 Anomaly
    + 예측 Anomaly
    → TP

    실제 Normal
    + 예측 Normal
    → TN

    실제 Normal
    + 예측 Anomaly
    → FP

    실제 Anomaly
    + 예측 Normal
    → FN


11. broken_large,
    broken_small,
    contamination 이미지도
    같은 방법으로 모두 검사한다.

12. 각 이미지의 결과를
    rows에 기록한다.

13. 전체 결과를
    day07_anomaly_scores.csv에 저장한다.
```

전체 흐름만 보면 다음과 같습니다.

```
Test 이미지
        ↓
Feature Vector
        ↓
Normal Reference와 비교
        ↓
Anomaly Score
        ↓
Threshold = 0.067469과 비교
        ↓
Normal / Anomaly 예측
        ↓
실제 정답과 비교
        ↓
TP / TN / FP / FN
        ↓
CSV 저장
```

특히 이 단계에서는 **Threshold를 다시 계산하지 않고 `day07_anomaly_rules.json`에 Freeze한 값을 읽어서 사용한다**는 것이 핵심입니다.

---

# 34. Test 결과 CSV를 읽어봅니다

앞에서 Test 이미지 83장을 모두 검사하고 결과를 다음 파일에 저장했습니다.

```
reports/day07_anomaly_scores.csv
```

터미널에서 앞부분을 확인합니다.

```
head reports/day07_anomaly_scores.csv
```

CSV의 한 행은 이미지 한 장의 검사 결과입니다.

예를 들어:

```
samples/day07/test/normal/000.png,
Normal,
good,
0.050054,
0.067469,
Normal,
TN,
True
```

는 다음과 같이 읽습니다.

```
실제 정답
Normal
        ↓
Anomaly Score
0.050054
        ↓
Threshold
0.067469
        ↓
Score가 Threshold보다 낮음
        ↓
예측
Normal
        ↓
실제도 Normal
예측도 Normal
        ↓
TN
정확한 판정
```

각 컬럼의 의미는 다음과 같습니다.

|컬럼|의미|
|---|---|
|`file`|검사한 이미지|
|`actual`|실제 정답|
|`defect_type`|정상 또는 결함 종류|
|`anomaly_score`|Normal Reference와 얼마나 다른지 나타낸 점수|
|`threshold`|Freeze한 판정 기준 `0.067469`|
|`predicted`|검사기가 판정한 결과|
|`error_type`|TP / TN / FP / FN|
|`correct`|실제 정답과 예측이 같은지 여부|

이번 Test에서는 총 **83장**을 검사했습니다.

```
Normal Test
20장

Anomaly Test
63장
 ├─ broken_large 20장
 ├─ broken_small 22장
 └─ contamination 21장

합계
83장
```

---

# 35. TP / TN / FP / FN으로 Test 결과를 확인합니다

이번 Anomaly Detection에서는 다음과 같이 정의합니다.

```
Positive = Anomaly = 이상
Negative = Normal = 정상
```

그리고:

```
T = True  = 맞게 판정
F = False = 잘못 판정
```

따라서 네 가지 결과는 다음과 같습니다.

```
TP
실제 Anomaly
예측 Anomaly
→ 이상을 맞게 찾아냄

TN
실제 Normal
예측 Normal
→ 정상을 맞게 판정

FP
실제 Normal
예측 Anomaly
→ 정상인데 이상이라고 잘못 판정

FN
실제 Anomaly
예측 Normal
→ 이상인데 정상이라고 놓침
```

이번 실제 Test 결과를 세어보면:

```
TN = 18
→ 정상 18장을 정상으로 맞게 판정

FP = 2
→ 정상 2장을 이상으로 잘못 판정

TP = 59
→ 이상 59장을 이상으로 맞게 판정

FN = 4
→ 이상 4장을 정상으로 놓침
```

전체 흐름으로 보면:

```
Test 83장
        ↓
┌─────────────────────┐
│ 실제 Normal 20장     │
└─────────────────────┘
        ↓
TN 18장
FP  2장


┌─────────────────────┐
│ 실제 Anomaly 63장    │
└─────────────────────┘
        ↓
TP 59장
FN  4장
```

특히 이번 결과에서 눈여겨볼 부분은 `FN`입니다.

```
broken_small
000.png
007.png
009.png
011.png

→ 실제로는 Anomaly
→ 하지만 Normal로 판정
→ FN 4장
```

즉 작은 파손 일부를 검사기가 놓쳤습니다.

반면:

```
broken_large
20장 모두 TP

contamination
21장 모두 TP
```

로 큰 파손과 오염은 이번 Baseline에서 모두 찾아냈습니다.

핵심만 정리하면:

```
TP 59
→ 이상을 맞게 잡음

TN 18
→ 정상을 맞게 통과시킴

FP 2
→ 정상인데 이상이라고 함

FN 4
→ 이상인데 정상이라고 놓침
```

> 이번 Test 결과를 통해 단순히 “잘 된다 / 안 된다”가 아니라, 어떤 종류의 이미지를 잘 찾았고 어떤 종류에서 실패했는지 확인할 수 있습니다.

---

# 36. `57_day07_evaluate.py` — Test 성능을 계산하고 실패 이미지를 모읍니다

앞 단계에서는 Test 이미지 83장을 검사하여 각 이미지의 `TP / TN / FP / FN` 결과를 CSV에 저장했습니다.

이번에는 그 결과를 모아서 **검사기의 전체 성능을 숫자로 계산**합니다.

또한 잘못 판정한 이미지인:

```
FP
→ 실제 Normal인데 Anomaly로 잘못 판정

FN
→ 실제 Anomaly인데 Normal로 놓침
```

을 별도 폴더에 복사합니다.

이후에는 이 이미지들을 직접 보면서 **왜 검사기가 실패했는지 분석**하게 됩니다.

전체 흐름은 다음과 같습니다.

```
day07_anomaly_scores.csv
        ↓
TP / TN / FP / FN 개수 계산
        ↓
Accuracy / Precision / Recall 계산
        ↓
종류별 정확도 계산
        ↓
FP 이미지 → outputs/day07/fp
FN 이미지 → outputs/day07/fn
        ↓
평가 결과를 Markdown Report로 저장
```

파일:

```
src/57_day07_evaluate.py
```

작성합니다.

```python
import shutil
from pathlib import Path

import pandas as pd


# --------------------------------------------------
# 1. 입력 / 출력 경로
# --------------------------------------------------
INPUT_CSV = Path(
    "reports/day07_anomaly_scores.csv"
)

REPORT_PATH = Path(
    "reports/day07_test_metrics.md"
)

FP_DIR = Path(
    "outputs/day07/fp"
)

FN_DIR = Path(
    "outputs/day07/fn"
)


# --------------------------------------------------
# 2. 0으로 나누는 오류 방지
# --------------------------------------------------
def safe_divide(a, b):
    return a / b if b else 0.0


# --------------------------------------------------
# 3. Test 결과 CSV 확인
# --------------------------------------------------
if not INPUT_CSV.exists():
    raise FileNotFoundError(
        f"Test 결과 CSV를 확인하세요: {INPUT_CSV}"
    )


df = pd.read_csv(
    INPUT_CSV
)

if df.empty:
    raise RuntimeError(
        "Test 결과가 없습니다."
    )


# --------------------------------------------------
# 4. TP / TN / FP / FN 개수 계산
# --------------------------------------------------
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


# --------------------------------------------------
# 5. 성능 지표 계산
# --------------------------------------------------
accuracy = safe_divide(
    tp + tn,
    total,
)

precision = safe_divide(
    tp,
    tp + fp,
)

recall = safe_divide(
    tp,
    tp + fn,
)


# --------------------------------------------------
# 6. FP / FN 폴더 준비
# --------------------------------------------------
for directory in [
    FP_DIR,
    FN_DIR,
]:
    if directory.exists():
        shutil.rmtree(
            directory
        )

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )


# --------------------------------------------------
# 7. FP / FN 이미지 따로 복사
# --------------------------------------------------
for _, row in df.iterrows():
    source = Path(
        row["file"]
    )

    if not source.exists():
        continue

    if row["error_type"] == "FP":
        shutil.copy2(
            source,
            FP_DIR / source.name,
        )

    elif row["error_type"] == "FN":
        defect_type = row[
            "defect_type"
        ]

        destination = (
            FN_DIR
            / f"{defect_type}_{source.name}"
        )

        shutil.copy2(
            source,
            destination,
        )


# --------------------------------------------------
# 8. 이미지 종류별 정확도
# --------------------------------------------------
per_type = (
    df.groupby(
        "defect_type"
    )["correct"]
    .mean()
    .sort_index()
)


# --------------------------------------------------
# 9. 평가 Report 작성
# --------------------------------------------------
report_lines = [
    "# Day07 Test Metrics",
    "",
    "## Confusion Matrix",
    "",
    "| Actual / Predicted | Anomaly | Normal |",
    "|---|---:|---:|",
    f"| Anomaly | TP = {tp} | FN = {fn} |",
    f"| Normal | FP = {fp} | TN = {tn} |",
    "",
    "## Metrics",
    "",
    f"- Accuracy: {accuracy:.4f}",
    f"- Anomaly Precision: {precision:.4f}",
    f"- Anomaly Recall: {recall:.4f}",
    f"- FP: {fp}",
    f"- FN: {fn}",
    "",
    "## Type Accuracy",
    "",
]


for defect_type, value in per_type.items():
    report_lines.append(
        f"- {defect_type}: {value:.4f}"
    )


REPORT_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)

REPORT_PATH.write_text(
    "\n".join(
        report_lines
    ),
    encoding="utf-8",
)


# --------------------------------------------------
# 10. 결과 출력
# --------------------------------------------------
print("=" * 70)
print("DAY07 TEST EVALUATION")
print("=" * 70)

print(
    "Total    :",
    total,
)

print(
    "TP       :",
    tp,
)

print(
    "TN       :",
    tn,
)

print(
    "FP       :",
    fp,
)

print(
    "FN       :",
    fn,
)

print(
    "Accuracy :",
    round(
        accuracy,
        4,
    ),
)

print(
    "Precision:",
    round(
        precision,
        4,
    ),
)

print(
    "Recall   :",
    round(
        recall,
        4,
    ),
)

print()

print(
    per_type
)

print()

print(
    "Report:",
    REPORT_PATH,
)
```

실행합니다.

```
python src/57_day07_evaluate.py
```

앞에서 나온 실제 Test 결과를 기준으로 하면 대략 다음과 같이 출력되어야 합니다.

```
======================================================================
DAY07 TEST EVALUATION
======================================================================
Total    : 83
TP       : 59
TN       : 18
FP       : 2
FN       : 4
Accuracy : 0.9277
Precision: 0.9672
Recall   : 0.9365
```

종류별 정확도는:

```
broken_large     1.0000
broken_small     0.8182
contamination    1.0000
good             0.9000
```

정도가 됩니다.

즉 현재 Baseline은:

```
전체 83장
        ↓
77장 정확
6장 오판정

FP = 2장
FN = 4장
```

입니다.

---

## 의사코드

```
1. 앞에서 만든
   day07_anomaly_scores.csv를 읽는다.

2. 각 이미지의 error_type을 확인한다.

3. TP가 몇 개인지 센다.

4. TN이 몇 개인지 센다.

5. FP가 몇 개인지 센다.

6. FN이 몇 개인지 센다.


7. TP / TN / FP / FN을 이용하여
   Accuracy를 계산한다.

   전체 이미지 중
   얼마나 많이 맞혔는가?


8. Precision을 계산한다.

   Anomaly라고 판정한 것 중
   실제 Anomaly가 얼마나 되는가?


9. Recall을 계산한다.

   실제 Anomaly 중
   얼마나 많이 찾아냈는가?


10. FP 이미지를 찾는다.

    실제 Normal
    → 예측 Anomaly

    해당 이미지를
    outputs/day07/fp에 복사한다.


11. FN 이미지를 찾는다.

    실제 Anomaly
    → 예측 Normal

    해당 이미지를
    outputs/day07/fn에 복사한다.


12. good / broken_large /
    broken_small / contamination별
    정확도를 계산한다.


13. 전체 평가 결과를

    reports/day07_test_metrics.md

    파일에 저장한다.
```

핵심만 보면:

```
Test 결과 CSV
        ↓
TP 59 / TN 18
FP 2 / FN 4
        ↓
성능 계산
        ↓
Accuracy  = 0.9277
Precision = 0.9672
Recall    = 0.9365
        ↓
FP / FN 이미지 따로 모으기
        ↓
왜 실패했는지 직접 확인
```

---

# 37. 성능 숫자보다 실패한 이미지를 함께 확인합니다

이번 Baseline의 결과는 다음과 같습니다.

```
TP = 59
TN = 18
FP = 2
FN = 4
```

전체적으로는 많은 이미지를 맞게 판정했지만 완벽하지는 않습니다.

특히 실제 결과를 보면:

```
Normal 일부
→ Score가 Threshold보다 높음
→ FP 2장

broken_small 일부
→ Score가 Threshold보다 낮음
→ FN 4장
```

이 발생했습니다.

따라서 이제 중요한 질문은:

```
왜 정상 이미지가 Anomaly로 판정되었을까?

왜 작은 파손은 Normal로 판정되었을까?
```

입니다.

Threshold를 바로 변경하기 전에 실패한 이미지를 직접 확인하고 원인을 분석합니다.

---

# 38. FP / FN 이미지를 직접 확인합니다

평가 프로그램은 잘못 판정한 이미지를 자동으로 따로 모았습니다.

FP 이미지를 확인합니다.

```bash
find outputs/day07/fp -maxdepth 1 -type f
```

이번 결과에서는 **정상 이미지 2장**이 들어 있어야 합니다.

```
FP
→ 실제 Normal
→ 검사 결과 Anomaly
→ 2장
```

FN 이미지를 확인합니다.

```bash
find outputs/day07/fn -maxdepth 1 -type f
```

이번 결과에서는 `broken_small` 이미지 **4장**이 들어 있어야 합니다.

```
FN
→ 실제 Anomaly
→ 검사 결과 Normal
→ 4장
```

이제 이 이미지들을 직접 보면서:

```
FP 2장
→ 정상인데 왜 Score가 높았는가?

FN 4장
→ 작은 파손인데 왜 Score가 낮았는가?
```

를 분석합니다.

---

# PART6 — Heatmap으로 검사기가 어느 위치에 반응했는지 확인합니다 (39~45)

앞에서는 Test 결과를 숫자로 확인했습니다.

```
TP = 59
TN = 18
FP = 2
FN = 4
```

이 결과만으로도 검사 성능은 알 수 있습니다.

하지만 다음 질문에는 답하기 어렵습니다.

```
왜 이 이미지는 Anomaly Score가 높았을까?

왜 broken_small 일부는
Anomaly인데 Normal로 놓쳤을까?

검사기는 이미지의 어느 부분에서
정상과 다른 특징을 발견했을까?
```

그래서 이번에는 **Heatmap을 이용하여 이미지의 어느 위치에서 정상 기준과 차이가 발생했는지 눈으로 확인**합니다.

---

# 39. Anomaly Score는 판정하고, Heatmap은 위치를 확인합니다

앞에서는 Test 이미지마다 `Anomaly Score`를 계산했습니다.

예를 들어 실제 `broken_large/000.png`의 결과는 다음과 같습니다.

```
Anomaly Score = 0.175958
Threshold     = 0.067469
```

비교하면:

```
0.175958 > 0.067469
        ↓
Anomaly
```

이므로 이상으로 판정했습니다.

여기서 `Anomaly Score`는:

> 검사 이미지의 전체적인 특징이 정상 기준과 얼마나 다른지를 최종적으로 하나의 거리값으로 계산한 결과

입니다.

즉 Score를 보면:

```
정상과 비슷한가?
정상과 많이 다른가?
```

는 알 수 있습니다.

하지만 Score만 보고는 다음을 알 수 없습니다.

```
병의 어느 부분이 달랐을까?

깨진 부분 때문일까?

병 입구 때문일까?

배경이나 위치 때문일까?
```

그래서 `Heatmap`을 사용합니다.

```
Anomaly Score
→ 이 이미지가
   정상과 얼마나 다른가?
→ Normal / Anomaly 판정에 사용


Heatmap
→ 이미지의 어느 위치에서
   정상과 다른 특징이 나타났는가?
→ 결과 분석에 사용
```

쉽게 생각하면:

```
Anomaly Score
→ 검사 결과표

"이 병은 Anomaly입니다."


Heatmap
→ 검사기가 바라본 위치 표시

"특히 이 부분에서
 정상과 차이가 크게 나타났습니다."
```

따라서 두 가지의 역할은 다릅니다.

![[Pasted image 20260902191428.png]]

```
                Test 이미지
                     ↓
             ResNet18 Feature
                     ↓
          ┌──────────┴──────────┐
          ↓                     ↓
    Feature Vector          Feature Map
          ↓                     ↓
   전체 차이 계산            위치별 차이 계산
          ↓                     ↓
   Anomaly Score              Heatmap
          ↓                     ↓
 Normal / Anomaly        어디에 반응했는지 확인
      판정                    분석
```

> Anomaly Score는 판정을 위한 숫자이고, Heatmap은 그 이미지에서 어느 위치에 차이가 나타났는지 눈으로 확인하기 위한 그림입니다.

---

# 40. Ground Truth Mask와 비교하는 이유

MVTec의 Anomaly 이미지에는 실제 결함 위치를 표시한 정답 Mask가 있습니다.

![[{905077A3-34EC-4749-B333-4CA7EB083873}.png]]

현재 실습에서는 다음 위치에 있습니다.

```
samples/day07/ground_truth/
├── broken_large/
├── broken_small/
└── contamination/
```

두 데이터의 역할은 다릅니다.

```
Ground Truth Mask
→ 실제 결함이 있는 위치
→ 정답

Heatmap
→ 현재 Baseline이
   정상과 차이가 크다고 본 위치
```

그래서 두 이미지를 나란히 보면 다음을 확인할 수 있습니다.

```
실제 결함 위치
        ↕
Heatmap 반응 위치

비슷한 곳에 반응했는가?
다른 곳에도 반응했는가?
작은 결함에는 반응이 약한가?
```

중요한 점이 있습니다.

> `Ground Truth Mask`는 Threshold를 결정하거나 Normal / Anomaly를 판정하는 데 사용하지 않았습니다.

앞에서 이미 판정은 끝났습니다.

```
Train / Validation
        ↓
Threshold 결정
        ↓
Rule Freeze
        ↓
Test 판정
        ↓
──────── 여기까지 판정 완료 ────────
        ↓
Heatmap + Ground Truth
        ↓
왜 그런 결과가 나왔는지 분석
```

즉 Ground Truth는 지금 **결과 분석용**으로 사용합니다.

---

# 41. `58_day07_make_heatmaps.py` — 대표 Test 이미지의 Heatmap을 만듭니다

앞에서는 Test 이미지가 `Normal`인지 `Anomaly`인지 판정했습니다.

이번에는 대표 이미지를 몇 장 골라서:

> 검사기가 이미지의 어느 부분에서 정상과 차이가 있다고 보았는지 Heatmap으로 만들어 확인합니다.

이번 프로그램이 실제로 만드는 결과는 다음과 같습니다.

```
Test 이미지
        ↓
Feature Map 추출
        ↓
Normal Reference Map과 비교
        ↓
위치별 차이 계산
        ↓
Heatmap 생성
```

그리고 Heatmap만 보면 원본의 어느 위치인지 알아보기 어려울 수 있으므로, 원본 이미지 위에도 Heatmap을 겹쳐서 저장합니다.

```
원본 이미지
        +
Heatmap
        ↓
Overlay
```

따라서 한 장의 Test 이미지에서 기본적으로 다음 두 파일이 만들어집니다.

```
Heatmap
→ 정상과 차이가 나타난 위치를 색으로 표시

Overlay
→ 원본 이미지 위에 Heatmap을 겹쳐서 표시
```

Anomaly 이미지에는 한 가지를 더 저장합니다.

```
Ground Truth Mask
→ 실제 결함 위치
```

그래서 Anomaly 이미지는 다음 세 가지를 비교할 수 있습니다.

```
원본 이미지

Heatmap
→ 검사기가 정상과 다르다고 본 위치

Ground Truth Mask
→ 실제 결함 위치
```

이 세 결과를 보면서:

```
검사기가 반응한 위치와
실제 결함 위치가 비슷한가?
```

를 확인합니다.

## 모든 83장을 만드는 것은 아닙니다

이번 실습에서는 Heatmap의 원리를 이해하는 것이 목적이므로 Test 83장을 모두 처리하지 않습니다.

다음 대표 이미지 8장만 선택합니다.

```
Normal
→ 2장

broken_large
→ 2장

broken_small
→ 2장

contamination
→ 2장

합계
→ 8장
```

특히 앞에서 `broken_small` 일부가 `FN`으로 나타났기 때문에, 작은 결함에서는 Heatmap이 어떻게 나타나는지도 함께 관찰합니다.

전체적으로 보면 이번 코드가 하는 일은 다음과 같습니다.

```
대표 Test 이미지 8장 선택
        ↓
각 이미지의 Feature Map 추출
        ↓
Normal Reference Map과 비교
        ↓
Heatmap 생성
        ↓
원본 + Heatmap Overlay 생성
        ↓
Anomaly라면 Ground Truth Mask도 준비
        ↓
결과 이미지를 파일로 저장
```

결과는 다음 폴더에 저장합니다.

```
outputs/day07/heatmap/
```

> 즉 이번 코드는 Heatmap을 보기 위한 준비만 하는 코드가 아니라, 실제 Heatmap 이미지를 생성하고 저장하는 코드입니다.


파일:

```
src/58_day07_make_heatmaps.py
```

작성합니다.

```python
import shutil
from pathlib import Path

import cv2

from day07_anomaly_core import (
    anomaly_map,
    create_backbone,
    extract_features,
    load_reference,
    make_heatmap_overlay,
)


# --------------------------------------------------
# 1. 입력 / 출력 경로
# --------------------------------------------------
REFERENCE_PATH = Path(
    "artifacts/day07/normal_reference.npz"
)

TEST_NORMAL_DIR = Path(
    "samples/day07/test/normal"
)

TEST_ANOMALY_DIR = Path(
    "samples/day07/test/anomaly"
)

GROUND_TRUTH_DIR = Path(
    "samples/day07/ground_truth"
)

OUTPUT_DIR = Path(
    "outputs/day07/heatmap"
)


# --------------------------------------------------
# 2. 필요한 파일 / 폴더 확인
# --------------------------------------------------
required_paths = [
    REFERENCE_PATH,
    TEST_NORMAL_DIR,
    TEST_ANOMALY_DIR,
    GROUND_TRUTH_DIR,
]

for path in required_paths:
    if not path.exists():
        raise FileNotFoundError(
            f"필요한 파일 또는 폴더를 확인하세요: {path}"
        )


# --------------------------------------------------
# 3. 이전 Heatmap 결과 초기화
# --------------------------------------------------
if OUTPUT_DIR.exists():
    shutil.rmtree(
        OUTPUT_DIR
    )

OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 4. Normal Reference Map 불러오기
# --------------------------------------------------
_, reference_map = load_reference(
    REFERENCE_PATH
)


# --------------------------------------------------
# 5. Feature 추출용 ResNet18 준비
# --------------------------------------------------
model = create_backbone()


# --------------------------------------------------
# 6. 시각화할 대표 이미지 선택
# --------------------------------------------------
samples = []

normal_files = sorted(
    TEST_NORMAL_DIR.glob(
        "*.png"
    )
)

for path in normal_files[:2]:
    samples.append(
        (
            "good",
            path,
        )
    )


for defect_name in [
    "broken_large",
    "broken_small",
    "contamination",
]:
    defect_files = sorted(
        (
            TEST_ANOMALY_DIR
            / defect_name
        ).glob("*.png")
    )

    for path in defect_files[:2]:
        samples.append(
            (
                defect_name,
                path,
            )
        )


if not samples:
    raise RuntimeError(
        "Heatmap을 만들 Test 이미지가 없습니다."
    )


# --------------------------------------------------
# 7. 이미지별 Heatmap 생성
# --------------------------------------------------
for defect_type, image_path in samples:

    # Test 이미지의 Feature Map 추출
    _, spatial_map = extract_features(
        model,
        image_path,
    )

    # Normal Reference Map과 위치별 차이 계산
    score_map = anomaly_map(
        spatial_map,
        reference_map,
    )

    # Heatmap + Overlay 생성
    (
        _,
        heatmap_color,
        overlay,
    ) = make_heatmap_overlay(
        image_path,
        score_map,
    )


    stem = (
        f"{defect_type}_"
        f"{image_path.stem}"
    )


    # Heatmap 저장
    cv2.imwrite(
        str(
            OUTPUT_DIR
            / f"{stem}_heatmap.jpg"
        ),
        heatmap_color,
    )


    # 원본 + Heatmap Overlay 저장
    cv2.imwrite(
        str(
            OUTPUT_DIR
            / f"{stem}_overlay.jpg"
        ),
        overlay,
    )


    # --------------------------------------------------
    # 8. Anomaly라면 Ground Truth Mask도 저장
    # --------------------------------------------------
    if defect_type != "good":

        mask_path = (
            GROUND_TRUTH_DIR
            / defect_type
            / f"{image_path.stem}_mask.png"
        )

        if mask_path.exists():

            mask = cv2.imread(
                str(mask_path),
                cv2.IMREAD_GRAYSCALE,
            )

            cv2.imwrite(
                str(
                    OUTPUT_DIR
                    / f"{stem}_gt_mask.png"
                ),
                mask,
            )


    print(
        "Saved:",
        stem,
    )


print("=" * 70)
print("DAY07 HEATMAP COMPLETE")
print("=" * 70)

print(
    "Samples:",
    len(samples),
)

print(
    "Saved  :",
    OUTPUT_DIR,
)
```

실행합니다.

```
python src/58_day07_make_heatmaps.py
```

### 코드 검토 결과

이 코드의 핵심 계산은 맞습니다.

기존 코드에서도:

```
Feature Map 추출
→ Normal Reference Map과 차이 계산
→ Heatmap 생성
→ Ground Truth Mask 저장
```

흐름은 정확했습니다.

수정한 부분은 주로:

```
필요한 파일 확인
+
이전 Heatmap 결과 삭제
+
대표 이미지가 없는 경우 확인
+
실행 결과 출력
```

입니다.

특히 이전 `outputs/day07/heatmap`을 지우고 다시 만드는 이유는 이전 실행에서 생성된 이미지가 남아서 이번 결과와 섞이는 것을 방지하기 위해서입니다.

---

## 실행 후 무엇이 만들어지나요?

현재 데이터에서는 8장의 대표 이미지를 처리합니다.

```
good
000.png
001.png

broken_large
000.png
001.png

broken_small
000.png
001.png

contamination
000.png
001.png
```

따라서 터미널에는 대략 다음처럼 출력됩니다.

```
Saved: good_000
Saved: good_001
Saved: broken_large_000
Saved: broken_large_001
Saved: broken_small_000
Saved: broken_small_001
Saved: contamination_000
Saved: contamination_001

======================================================================
DAY07 HEATMAP COMPLETE
======================================================================
Samples: 8
Saved  : outputs/day07/heatmap
```

생성 파일을 확인합니다.

```
ls -lh outputs/day07/heatmap
```

Normal 이미지에는:

```
good_000_heatmap.jpg
good_000_overlay.jpg
```

처럼 Heatmap과 Overlay가 만들어집니다.

Anomaly 이미지에는:

```
broken_small_000_heatmap.jpg
broken_small_000_overlay.jpg
broken_small_000_gt_mask.png
```

처럼 세 가지가 만들어집니다.

```
Heatmap
→ Feature 차이만 색으로 표시

Overlay
→ 원본 이미지 위에 Heatmap을 겹쳐 표시

GT Mask
→ 실제 결함 위치
```

---

## 의사코드

```
1. Normal Reference 파일의 위치를 지정한다.

2. Normal Test,
   Anomaly Test,
   Ground Truth 폴더의 위치를 지정한다.

3. 필요한 파일과 폴더가 존재하는지 확인한다.

4. 이전에 만든 Heatmap 결과가 있으면 삭제한다.

5. normal_reference.npz에서
   reference_map을 불러온다.

6. ResNet18 Feature Extractor를 준비한다.

7. Normal Test에서 대표 이미지 2장을 선택한다.

8. broken_large에서 2장을 선택한다.

9. broken_small에서 2장을 선택한다.

10. contamination에서 2장을 선택한다.

11. 선택한 이미지에서
    Feature Map을 추출한다.

12. Test Feature Map과
    Normal Reference Map을 비교한다.

13. 14 × 14 위치마다
    정상과의 차이를 계산한다.

14. 위치별 차이를 색으로 표현하여
    Heatmap을 만든다.

15. Heatmap을 원본 이미지와 겹쳐
    Overlay를 만든다.

16. Anomaly 이미지라면
    해당 Ground Truth Mask도 찾는다.

17. Heatmap,
    Overlay,
    Ground Truth Mask를 저장한다.

18. 저장한 이미지 수와
    저장 위치를 출력한다.
```

전체 흐름만 보면:

```
Test 이미지
        ↓
Feature Map
        ↓
Normal Reference Map과 비교
        ↓
위치별 차이 계산
        ↓
Heatmap
        ↓
Overlay

Anomaly 이미지
        ↓
Ground Truth Mask도 함께 확인
```

---

# 42. 실제 Test 결과와 Heatmap을 함께 확인합니다

VS Code에서 다음 폴더를 엽니다.

```
outputs/day07/heatmap
```

이번에 선택한 대표 이미지 중 일부는 앞에서 판정 결과를 이미 확인했습니다.

![[Pasted image 20260902192650.png]]

먼저 `broken_large_000` 결과를 확인합니다.

앞의 Test 결과에서 이 이미지는:

```
broken_large/000.png

실제 정답
→ Anomaly

Anomaly Score
→ 0.175958

Threshold
→ 0.067469

0.175958 > 0.067469
        ↓
Anomaly 판정
        ↓
TP
```

였습니다.

Heatmap 폴더에는 다음 결과가 생성되어 있습니다.

```
broken_large_000_heatmap.jpg
broken_large_000_overlay.jpg
broken_large_000_gt_mask.png
```

현재 실제 화면을 비교하면:

```
Ground Truth Mask
→ 실제 파손 위치가 흰색으로 표시

Heatmap
→ 정상 기준과 차이가 큰 위치가
   노랑 / 빨강으로 표시
```

됩니다.

실제 결과에서도 Ground Truth의 파손 영역과 Heatmap의 강한 반응 영역이 비슷한 방향에 나타나는 것을 확인할 수 있습니다.

```
실제 결함 위치
        ↕
Heatmap의 강한 반응 위치

어느 정도 비슷함
```

즉 이 이미지는:

> 실제로 파손된 영역 주변에서 Feature 차이가 크게 나타났고, 최종 판정도 `Anomaly`로 맞게 나온 TP 사례입니다.

---

# 43. Heatmap과 Ground Truth는 같은 그림이 아닙니다

두 그림의 역할은 다릅니다.

```
Ground Truth Mask
→ 실제 결함 위치의 정답

Heatmap
→ 검사기가 정상 기준과
   차이가 크다고 본 위치
```

따라서 두 모양이 정확히 일치할 필요는 없습니다.

현재 `broken_large_000`에서도:

```
Ground Truth
→ 비교적 명확한 흰색 결함 영역

Heatmap
→ 결함 주변까지
   넓게 노랑 / 빨강으로 반응
```

하는 것을 볼 수 있습니다.

이것은 현재 Heatmap이 원본 픽셀을 직접 비교하는 것이 아니라 `14 × 14` Feature Map의 차이를 확대해서 보여주기 때문입니다.

또한:

```
빨간색
≠ 반드시 실제 결함
```

입니다.

> Heatmap은 실제 결함 위치의 정답이 아니라, **정상과 Feature 차이가 크게 나타난 위치를 보여주는 분석 자료**입니다.

---

# 44. 이제 작은 결함의 FN과 TP를 비교합니다

`broken_large_000`에서는 비교적 큰 결함을 잘 찾아냈습니다.

그렇다면 앞에서 놓쳤던 `broken_small`은 어떨까요?

실제 Test 결과를 다시 보면:

```
broken_small/000.png

Score = 0.053304
Threshold = 0.067469

→ Normal 판정
→ FN
```

반면:

```
broken_small/001.png

Score = 0.089468
Threshold = 0.067469

→ Anomaly 판정
→ TP
```

입니다.

따라서 다음 파일을 직접 비교해 봅니다.

```
broken_small_000_heatmap.jpg
broken_small_000_gt_mask.png

broken_small_001_heatmap.jpg
broken_small_001_gt_mask.png
```

확인할 질문은 간단합니다.

```
000.png
→ 실제 작은 파손 위치에
   Heatmap 반응이 얼마나 나타나는가?

001.png
→ 실제 결함 위치에서
   반응이 어떻게 나타나는가?
```

여기서는 **Heatmap 색만 보고 판정을 다시 하는 것이 아니라**, FN과 TP 이미지의 위치별 Feature 차이를 관찰하는 것이 목적입니다.

---

# 45. 왜 작은 결함을 일부 놓쳤을까요?

실제 Test 결과는:

```
broken_large
20장 모두 검출

contamination
21장 모두 검출

broken_small
22장 중 4장 FN
```

이었습니다.

현재 최종 판정에는 이미지 전체 특징을 요약한 `Feature Vector`를 사용합니다.

그래서 작은 파손은:

```
작은 파손
        ↓
일부 위치에서는 차이가 발생
        ↓
하지만 이미지 전체에서
차지하는 영역이 작음
        ↓
전체 Feature 변화가 작을 수 있음
        ↓
Anomaly Score가 낮게 나올 수 있음
        ↓
Threshold 이하
        ↓
FN
```

이 될 수 있습니다.

실제 `broken_small/000.png`가 그 사례입니다.

```
0.053304 <= 0.067469
        ↓
Normal
        ↓
FN
```

따라서 이번 PART6에서는 두 가지를 구분해서 봅니다.

```
broken_large_000
→ TP 사례
→ 실제 결함 위치와
   Heatmap 반응 위치 비교


broken_small_000 / 001
→ FN과 TP 비교
→ 작은 결함에서 왜 판정 차이가
   발생할 수 있는지 분석
```

### PART6 핵심

```
PART5
→ 얼마나 잘 맞았는가?
→ TP / TN / FP / FN

        ↓

PART6
→ 어디에 반응했는가?
→ Heatmap

        ↓

Ground Truth와 비교
        ↓

왜 맞았고
왜 놓쳤는지 분석
```

> Anomaly Score는 판정에 사용하고, Heatmap은 그 판정이 왜 나왔는지 이해하는 데 사용합니다.

---

### 이 PART6을 왜 하는지 한 번에 정리하면

앞 PART5에서 이미:

```
83장 검사
        ↓
TP 59
TN 18
FP 2
FN 4
```

를 확인했습니다.

PART6에서는 한 단계 더 들어갑니다.

```
성능 숫자 확인
        ↓
어떤 이미지가 실패했는지 확인
        ↓
Heatmap 생성
        ↓
어느 위치에서
Feature 차이가 발생했는지 확인
        ↓
Ground Truth와 비교
        ↓
왜 FP / FN이 발생할 수 있는지 분석
```

즉 PART5가 “얼마나 잘 맞았는가?”를 확인하는 단계라면, PART6는 “왜 맞았고 왜 틀렸는가?”를 이미지로 분석하는 단계라고 이해하면 가장 쉽습니다.

---

# PART7 — Challenge: 정상 이미지의 촬영 조건을 바꾸어 다시 검사합니다 (46~52)

앞에서는 다음 과정을 완료했습니다.

```
Normal Train
        ↓
Normal Reference 생성
        ↓
Normal Validation
        ↓
Threshold = 0.067469
        ↓
Rule Freeze
        ↓
Test 83장 검사
        ↓
TP / TN / FP / FN 확인
        ↓
Heatmap 분석
```

이번에는 새로운 검사 방법을 배우는 것이 아닙니다.

앞에서 만든 **같은 검사기와 같은 Rule**을 사용하되, 정상 이미지의 촬영 조건만 일부러 바꾸어 다시 검사합니다.

```
정상 이미지
        ↓
촬영 조건 변경
        ↓
Dark / Bright / Blur / Shift / Rotation
        ↓
같은 Normal Reference
        +
같은 Threshold 0.067469
        ↓
다시 검사
        ↓
Normal 유지?
또는
FP 발생?
```

> 제품은 그대로 정상인데 촬영 환경이 달라졌을 때 검사 결과가 어떻게 변하는지 확인하는 실습입니다.

---

# 46. 정상 Test 이미지 한 장을 다시 사용합니다

이번 Challenge에서는 다음 폴더의 첫 번째 정상 이미지를 사용합니다.

```
samples/day07/test/normal/000.png
```

이 이미지는 실제로 정상입니다.

이 한 장을 프로그램으로 5가지 형태로 변형합니다.

```
정상 원본
   ↓
 ┌──────┬──────┬──────┬──────┐
 ↓      ↓      ↓      ↓      ↓
Dark  Bright  Blur   Shift  Rotation
```

변형된 이미지도 제품 자체에는 결함이 없으므로 실제 정답은 모두:

```
Normal
```

입니다.

---

# 47. 이번 Challenge에서 확인할 것

확인할 내용은 간단합니다.

```
촬영 조건 변경
        ↓
Feature가 얼마나 달라지는가?
        ↓
Anomaly Score가 얼마나 변하는가?
        ↓
Threshold = 0.067469과 비교
        ↓
Normal이면 PASS
Anomaly이면 FP
```

즉:

```
실제 Normal
예측 Normal
→ PASS

실제 Normal
예측 Anomaly
→ FP
```

입니다.

---

# 48. `59_day07_make_challenge_variants.py` — 정상 이미지를 5가지 조건으로 변형합니다

먼저 검사할 변형 이미지를 만듭니다.

파일:

```
src/59_day07_make_challenge_variants.py
```

작성합니다.

```python
import shutil
from pathlib import Path

import cv2
import numpy as np


# --------------------------------------------------
# 1. 입력 / 출력 경로
# --------------------------------------------------
NORMAL_DIR = Path(
    "samples/day07/test/normal"
)

CHALLENGE_ROOT = Path(
    "samples/day07/challenge"
)

SOURCE_DIR = (
    CHALLENGE_ROOT
    / "source"
)

VARIANT_DIR = (
    CHALLENGE_ROOT
    / "variants"
)


# --------------------------------------------------
# 2. 이전 Challenge 결과 초기화
# --------------------------------------------------
if CHALLENGE_ROOT.exists():
    shutil.rmtree(
        CHALLENGE_ROOT
    )

SOURCE_DIR.mkdir(
    parents=True,
    exist_ok=True,
)

VARIANT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 3. Normal Test 이미지 선택
# --------------------------------------------------
normal_files = sorted(
    NORMAL_DIR.glob("*.png")
)

if not normal_files:
    raise RuntimeError(
        "Test Normal 이미지가 없습니다."
    )


# 첫 번째 Normal Test 이미지를 사용합니다.
source_path = normal_files[0]

source_copy = (
    SOURCE_DIR
    / source_path.name
)

shutil.copy2(
    source_path,
    source_copy,
)


image = cv2.imread(
    str(source_path)
)

if image is None:
    raise FileNotFoundError(
        source_path
    )


height, width = image.shape[:2]


# --------------------------------------------------
# 4. Dark
# --------------------------------------------------
variant = cv2.convertScaleAbs(
    image,
    alpha=0.60,
    beta=0,
)

cv2.imwrite(
    str(
        VARIANT_DIR
        / "normal_dark.png"
    ),
    variant,
)


# --------------------------------------------------
# 5. Bright
# --------------------------------------------------
variant = cv2.convertScaleAbs(
    image,
    alpha=1.20,
    beta=20,
)

cv2.imwrite(
    str(
        VARIANT_DIR
        / "normal_bright.png"
    ),
    variant,
)


# --------------------------------------------------
# 6. Blur
# --------------------------------------------------
variant = cv2.GaussianBlur(
    image,
    (11, 11),
    0,
)

cv2.imwrite(
    str(
        VARIANT_DIR
        / "normal_blur.png"
    ),
    variant,
)


# --------------------------------------------------
# 7. Position Shift
# --------------------------------------------------
matrix = np.float32(
    [
        [1, 0, 35],
        [0, 1, 20],
    ]
)

variant = cv2.warpAffine(
    image,
    matrix,
    (width, height),
    borderMode=cv2.BORDER_REFLECT,
)

cv2.imwrite(
    str(
        VARIANT_DIR
        / "normal_shift.png"
    ),
    variant,
)


# --------------------------------------------------
# 8. Rotation
# --------------------------------------------------
center = (
    width // 2,
    height // 2,
)

rotation_matrix = cv2.getRotationMatrix2D(
    center,
    6.0,
    1.0,
)

variant = cv2.warpAffine(
    image,
    rotation_matrix,
    (width, height),
    borderMode=cv2.BORDER_REFLECT,
)

cv2.imwrite(
    str(
        VARIANT_DIR
        / "normal_rotation.png"
    ),
    variant,
)


# --------------------------------------------------
# 9. 결과 확인
# --------------------------------------------------
print(
    "Challenge Source:",
    source_path,
)

print(
    "Saved:",
    VARIANT_DIR,
)
```

실행합니다.

```
python src/59_day07_make_challenge_variants.py
```

## 의사코드

```
1. Normal Test 이미지 폴더를 읽는다.

2. 첫 번째 정상 이미지
   000.png를 선택한다.

3. 원본을 Challenge source에 복사한다.

4. 이미지를 어둡게 만든다.

5. 이미지를 밝게 만든다.

6. 이미지를 흐리게 만든다.

7. 병의 위치를 이동한다.

8. 병을 6도 회전한다.

9. 변형된 5장을
   challenge/variants에 저장한다.
```

전체 흐름은:

```
normal/000.png
        ↓
5가지 영상 조건 변화
        ↓
normal_dark.png
normal_bright.png
normal_blur.png
normal_shift.png
normal_rotation.png
```

입니다.

---

# 49. 생성된 이미지를 확인합니다

파일이 만들어졌는지 확인합니다.

```bash
ls -lh samples/day07/challenge/variants
```

다음 5장이 있으면 됩니다.

```
normal_dark.png
normal_bright.png
normal_blur.png
normal_shift.png
normal_rotation.png
```

VS Code에서 직접 열어봅니다.

```
Dark
→ 어두워졌는가?

Bright
→ 밝아졌는가?

Blur
→ 흐려졌는가?

Shift
→ 위치가 이동했는가?

Rotation
→ 회전했는가?
```

![[{A6CF5918-567F-4CC6-A58A-D609D7B39458}.png]]

모두 **제품 자체의 실제 정답은 Normal**이라는 점을 확인합니다.

---

# 50. `60_day07_run_challenge.py` — 같은 Rule로 변형 이미지를 다시 검사합니다

이제 변형한 5장을 앞에서 만든 검사기에 다시 넣습니다.

중요한 점은 새 Threshold를 만들지 않는 것입니다.

```
Normal Reference
→ 그대로 사용

Threshold
→ 0.067469 그대로 사용

Rule
→ day07-anomaly-v1.0 그대로 사용
```

파일:

```
src/60_day07_run_challenge.py
```

작성합니다.

```python
import csv
import json
from pathlib import Path

from day07_anomaly_core import (
    anomaly_score,
    create_backbone,
    extract_features,
    load_reference,
)


# --------------------------------------------------
# 1. 입력 / 출력 경로
# --------------------------------------------------
VARIANT_DIR = Path(
    "samples/day07/challenge/variants"
)

REFERENCE_PATH = Path(
    "artifacts/day07/normal_reference.npz"
)

RULE_PATH = Path(
    "configs/day07_anomaly_rules.json"
)

OUTPUT_CSV = Path(
    "reports/day07_challenge_results.csv"
)


# --------------------------------------------------
# 2. 필요한 파일 확인
# --------------------------------------------------
required_paths = [
    VARIANT_DIR,
    REFERENCE_PATH,
    RULE_PATH,
]

for path in required_paths:
    if not path.exists():
        raise FileNotFoundError(
            f"필요한 파일 또는 폴더를 확인하세요: {path}"
        )


# --------------------------------------------------
# 3. Freeze한 Rule 불러오기
# --------------------------------------------------
rules = json.loads(
    RULE_PATH.read_text(
        encoding="utf-8"
    )
)

threshold = float(
    rules["anomaly_threshold"]
)


# --------------------------------------------------
# 4. Normal Reference와 모델 준비
# --------------------------------------------------
reference_vector, _ = load_reference(
    REFERENCE_PATH
)

model = create_backbone()

rows = []


# --------------------------------------------------
# 5. 변형 이미지 전체 검사
# --------------------------------------------------
for image_path in sorted(
    VARIANT_DIR.glob("*.png")
):
    vector, _ = extract_features(
        model,
        image_path,
    )

    score = anomaly_score(
        vector,
        reference_vector,
    )

    predicted = (
        "Anomaly"
        if score > threshold
        else "Normal"
    )

    rows.append(
        {
            "file": image_path.name,
            "actual": "Normal",
            "anomaly_score": round(
                score,
                6,
            ),
            "threshold": round(
                threshold,
                6,
            ),
            "predicted": predicted,
            "result": (
                "PASS"
                if predicted == "Normal"
                else "FP"
            ),
        }
    )


if not rows:
    raise RuntimeError(
        "검사할 Challenge 이미지가 없습니다."
    )


# --------------------------------------------------
# 6. CSV 저장
# --------------------------------------------------
OUTPUT_CSV.parent.mkdir(
    parents=True,
    exist_ok=True,
)

with OUTPUT_CSV.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=[
            "file",
            "actual",
            "anomaly_score",
            "threshold",
            "predicted",
            "result",
        ],
    )

    writer.writeheader()
    writer.writerows(rows)


# --------------------------------------------------
# 7. 결과 출력
# --------------------------------------------------
print("=" * 70)
print("DAY07 CHALLENGE")
print("=" * 70)

print(
    "Threshold:",
    threshold,
)

for row in rows:
    print(
        row["file"],
        "score=",
        row["anomaly_score"],
        "predicted=",
        row["predicted"],
        "result=",
        row["result"],
    )

print()

print(
    "Saved:",
    OUTPUT_CSV,
)
```

실행합니다.

```
python src/60_day07_run_challenge.py
```

---

## 의사코드

```
1. 앞에서 만든
   5개의 변형 이미지를 읽는다.

2. Freeze한 Rule JSON을 읽는다.

3. Threshold = 0.067469를 가져온다.

4. Normal Reference를 불러온다.

5. Dark 이미지를 검사한다.

6. Bright 이미지를 검사한다.

7. Blur 이미지를 검사한다.

8. Shift 이미지를 검사한다.

9. Rotation 이미지를 검사한다.

10. 각 이미지의
    Anomaly Score를 계산한다.

11. Score와 Threshold를 비교한다.

    Score <= 0.067469
    → Normal
    → PASS

    Score > 0.067469
    → Anomaly
    → FP

12. 5장의 결과를
    day07_challenge_results.csv에 저장한다.
```

핵심 흐름은:

```
변형된 정상 이미지 5장
        ↓
같은 Normal Reference
        ↓
Anomaly Score 계산
        ↓
같은 Threshold = 0.067469
        ↓
Normal / Anomaly
        ↓
PASS / FP
```

입니다.

---

# 51. Challenge 결과를 확인합니다

결과 CSV를 확인합니다.

```
cat reports/day07_challenge_results.csv
```

실제 결과는 다음과 같습니다.

```
file,actual,anomaly_score,threshold,predicted,result
normal_blur.png,Normal,0.065173,0.067469,Normal,PASS
normal_bright.png,Normal,0.104318,0.067469,Anomaly,FP
normal_dark.png,Normal,0.123422,0.067469,Anomaly,FP
normal_rotation.png,Normal,0.054642,0.067469,Normal,PASS
normal_shift.png,Normal,0.076099,0.067469,Anomaly,FP
```

모든 이미지의 실제 정답은 `Normal`입니다.

판정 기준은 앞에서 Freeze한:

```
Threshold = 0.067469
```

입니다.

따라서:

```
Score <= 0.067469
→ Normal
→ PASS

Score > 0.067469
→ Anomaly
→ 실제는 Normal이므로 FP
```

로 해석합니다.

## 실제 결과를 비교합니다

|변형|Anomaly Score|판정|결과|
|---|---|---|---|
|Blur|`0.065173`|Normal|PASS|
|Bright|`0.104318`|Anomaly|FP|
|Dark|`0.123422`|Anomaly|FP|
|Rotation|`0.054642`|Normal|PASS|
|Shift|`0.076099`|Anomaly|FP|

가장 높은 Score는:

```
Dark
→ 0.123422
→ Threshold 초과
→ FP
```

였습니다.

`Bright`와 `Shift`도 Threshold를 넘어 FP가 발생했습니다.

```
Bright
0.104318 > 0.067469
→ FP

Shift
0.076099 > 0.067469
→ FP
```

반면 `Blur`와 `Rotation`은 Threshold를 넘지 않았습니다.

```
Blur
0.065173 <= 0.067469
→ PASS

Rotation
0.054642 <= 0.067469
→ PASS
```

특히 Blur는:

```
0.065173
        ↓
Threshold 0.067469에 가까움
        ↓
이번 조건에서는 간신히 Normal 유지
```

했다는 점도 관찰할 수 있습니다.

## 이번 Challenge에서 무엇을 알 수 있나요?

```
제품 자체는 모두 정상
        ↓
촬영 조건만 변경
        ↓
Dark / Bright / Shift
→ FP 발생

Blur / Rotation
→ Normal 유지
```

즉 현재 Baseline은 이번 실험 조건에서는 **밝기 변화와 위치 변화에 비교적 민감하게 반응했다**는 것을 확인할 수 있습니다.

> 실제 결함이 없어도 촬영 조건이 달라지면 Feature가 변하고, Anomaly Score가 Threshold를 넘어 FP가 발생할 수 있습니다.

따라서 이번 Challenge의 핵심 결과는:

```
정상 제품
        +
환경 변화
        ↓
Feature 변화
        ↓
Anomaly Score 변화
        ↓
경우에 따라 FP 발생
```

입니다.

---

# 52. Challenge 결과는 Baseline의 약점을 확인하는 데 사용합니다

Challenge에서 FP가 발생하더라도 바로 Threshold를 변경하지 않습니다.

```
Challenge 결과
        ↓
PASS / FP 확인
        ↓
어떤 영상 변화에 민감한지 확인
        ↓
실패 원인 기록
```

왜냐하면 Threshold를 무조건 높이면:

```
Threshold 높임
        ↓
정상은 더 잘 통과할 수 있음
        ↓
하지만 작은 결함도
Normal로 통과할 가능성 증가
        ↓
FN 증가 가능
```

하기 때문입니다.

실제로 앞 Test에서도:

```
broken_small
→ FN 4장
```

이 이미 발생했습니다.

따라서 이번 Challenge의 목적은:

> Rule을 다시 맞추는 것이 아니라, 현재 `day07-anomaly-v1.0`이 조명·Blur·위치·회전 변화에 얼마나 강하거나 약한지 확인하는 것입니다.

---

# PART8 — Mini Challenge: 배운 흐름을 스스로 다시 적용합니다 (53~58)

---

# 53. Mini Challenge에서는 무엇을 하나요?

앞 Challenge에서는 정해진 조건으로 정상 이미지를 변형하여 검사했습니다.

이번에는 **다른 정상 이미지 한 장을 선택하고, 변형 강도를 직접 정해서 같은 과정을 스스로 반복**합니다.

```
다른 정상 이미지 선택
        ↓
변형 조건 3개 선택
        ↓
실행 전 결과 예상
        ↓
변형 이미지 생성
        ↓
Frozen Rule 적용
        ↓
Anomaly Score 확인
        ↓
PASS / FP 판단
        ↓
가장 민감한 조건 설명
```

새로운 Dataset을 만들거나 Threshold를 다시 정하지 않습니다.

```
Normal Reference
→ 그대로 사용

Threshold
→ 0.067469 그대로 사용

Rule
→ day07-anomaly-v1.0 그대로 사용
```

> Mini Challenge의 목적은 새로운 기술을 배우는 것이 아니라, **앞에서 배운 검사 흐름을 혼자 다시 적용할 수 있는지 확인하는 것**입니다.

---

# 54. 세 가지 조건을 직접 정합니다

앞 Challenge에서 사용한 값은 다음과 같았습니다.

```
Dark     alpha = 0.60
Blur     kernel = 11
Rotation = 6°
```

Mini Challenge에서는 값을 직접 바꿉니다.

예:

```
Dark
alpha = 0.75

Blur
kernel = 7

Rotation
angle = 10°
```

실행 전에 간단히 예상합니다.

```
Dark
→ PASS일까? FP일까?

Blur
→ PASS일까? FP일까?

Rotation
→ PASS일까? FP일까?
```

정답을 맞히는 것이 목적은 아닙니다.

> **예상한 뒤 실제 Score를 확인하고 왜 달랐는지 생각하는 것**이 중요합니다.

---

# 55. `61_day07_mini_challenge.py` — 직접 조건을 바꾸어 검사합니다

앞 Challenge에서 사용한 `normal/000.png`가 아니라 다음 정상 이미지인 `normal/001.png`를 사용합니다.

학생이 직접 바꿀 값은 세 가지입니다.

```
DARK_ALPHA = 0.75
BLUR_KERNEL = 7
ROTATION_ANGLE = 10.0
```

파일:

```
src/61_day07_mini_challenge.py
```

작성합니다.

```python
import json
import shutil
from pathlib import Path

import cv2

from day07_anomaly_core import (
    anomaly_score,
    create_backbone,
    extract_features,
    load_reference,
)


# --------------------------------------------------
# 1. 입력 / 출력 경로
# --------------------------------------------------
NORMAL_DIR = Path(
    "samples/day07/test/normal"
)

OUTPUT_ROOT = Path(
    "samples/day07/mini_challenge"
)

VARIANT_DIR = (
    OUTPUT_ROOT
    / "variants"
)

REFERENCE_PATH = Path(
    "artifacts/day07/normal_reference.npz"
)

RULE_PATH = Path(
    "configs/day07_anomaly_rules.json"
)


# --------------------------------------------------
# 2. 학생이 직접 바꿀 조건
# --------------------------------------------------
DARK_ALPHA = 0.75
BLUR_KERNEL = 7
ROTATION_ANGLE = 10.0


# --------------------------------------------------
# 3. 이전 결과 초기화
# --------------------------------------------------
if OUTPUT_ROOT.exists():
    shutil.rmtree(
        OUTPUT_ROOT
    )

VARIANT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 4. 앞 Challenge와 다른 정상 이미지 선택
# --------------------------------------------------
normal_files = sorted(
    NORMAL_DIR.glob("*.png")
)

if len(normal_files) < 2:
    raise RuntimeError(
        "Mini Challenge용 Normal 이미지가 부족합니다."
    )

source_path = normal_files[1]

image = cv2.imread(
    str(source_path)
)

if image is None:
    raise FileNotFoundError(
        source_path
    )

height, width = image.shape[:2]


# --------------------------------------------------
# 5. Dark
# --------------------------------------------------
variant = cv2.convertScaleAbs(
    image,
    alpha=DARK_ALPHA,
    beta=0,
)

cv2.imwrite(
    str(
        VARIANT_DIR
        / "mini_dark.png"
    ),
    variant,
)


# --------------------------------------------------
# 6. Blur
# --------------------------------------------------
variant = cv2.GaussianBlur(
    image,
    (
        BLUR_KERNEL,
        BLUR_KERNEL,
    ),
    0,
)

cv2.imwrite(
    str(
        VARIANT_DIR
        / "mini_blur.png"
    ),
    variant,
)


# --------------------------------------------------
# 7. Rotation
# --------------------------------------------------
center = (
    width // 2,
    height // 2,
)

matrix = cv2.getRotationMatrix2D(
    center,
    ROTATION_ANGLE,
    1.0,
)

variant = cv2.warpAffine(
    image,
    matrix,
    (width, height),
    borderMode=cv2.BORDER_REFLECT,
)

cv2.imwrite(
    str(
        VARIANT_DIR
        / "mini_rotation.png"
    ),
    variant,
)


# --------------------------------------------------
# 8. Frozen Rule 불러오기
# --------------------------------------------------
rules = json.loads(
    RULE_PATH.read_text(
        encoding="utf-8"
    )
)

threshold = float(
    rules["anomaly_threshold"]
)

reference_vector, _ = load_reference(
    REFERENCE_PATH
)

model = create_backbone()


# --------------------------------------------------
# 9. 세 이미지 검사
# --------------------------------------------------
print("=" * 70)
print("DAY07 MINI CHALLENGE")
print("=" * 70)

print(
    "Source   :",
    source_path,
)

print(
    "Threshold:",
    threshold,
)

print()


for image_path in sorted(
    VARIANT_DIR.glob("*.png")
):
    vector, _ = extract_features(
        model,
        image_path,
    )

    score = anomaly_score(
        vector,
        reference_vector,
    )

    predicted = (
        "Anomaly"
        if score > threshold
        else "Normal"
    )

    result = (
        "FP"
        if predicted == "Anomaly"
        else "PASS"
    )

    print(
        image_path.name,
        "score=",
        round(score, 6),
        "predicted=",
        predicted,
        "result=",
        result,
    )
```

실행합니다.

```
python src/61_day07_mini_challenge.py
```

---

# 56. 코드를 어떻게 이해하면 되나요?

의사코드로 다시 읽어봅니다.

```
1. 앞 Challenge와 다른
   정상 이미지 001.png를 선택한다.

2. Dark 강도를 직접 정한다.

3. Blur 강도를 직접 정한다.

4. Rotation 각도를 직접 정한다.

5. 세 개의 변형 이미지를 만든다.

6. 앞에서 Freeze한
   Threshold = 0.067469를 불러온다.

7. 세 이미지의
   Anomaly Score를 계산한다.

8. Score와 Threshold를 비교한다.

9. Normal이면 PASS로 기록한다.

10. Anomaly이면
    실제 정답은 Normal이므로
    FP로 기록한다.

11. 세 결과를 비교한다.
```

전체 흐름은 간단합니다.

```
다른 Normal 이미지
        ↓
3가지 조건 직접 변경
        ↓
Dark / Blur / Rotation
        ↓
같은 Frozen Rule
        ↓
Score 확인
        ↓
PASS / FP
```

---

# 57. 결과는 간단히 기록합니다

긴 보고서는 작성하지 않습니다.

파일:

```
reports/day07_mini_challenge.md
```

다음 정도만 작성합니다.

```
# Day07 Mini Challenge

Source:

Threshold: 0.067469

| 변형 | 내가 정한 값 | 예상 | 실제 Score | 결과 |
|---|---|---|---|---|
| Dark | | PASS / FP | | |
| Blur | | PASS / FP | | |
| Rotation | | PASS / FP | | |

가장 Score가 높았던 조건:

내 예상과 달랐던 조건:

한 줄 정리:
```

핵심은 많은 문장을 작성하는 것이 아닙니다.

```
예상
        ↓
실행
        ↓
실제 Score 확인
        ↓
PASS / FP 판단
        ↓
왜 그랬는지 한 줄 설명
```

이면 충분합니다.

---

# 58. Challenge와 Mini Challenge를 정리합니다

두 실습의 차이만 확인합니다.

```
Challenge
→ 모두 같은 조건 사용
→ Baseline의 약점 함께 확인


Mini Challenge
→ 다른 정상 이미지 사용
→ 변형 강도를 직접 결정
→ 예상
→ 실행
→ 결과 해석
```

두 실습 모두 중요한 원칙은 같습니다.

```
Normal Reference
그대로 사용

Threshold
0.067469 그대로 사용

Rule
day07-anomaly-v1.0 그대로 사용
```

> Mini Challenge를 끝냈다면 **같은 검사 Rule을 다른 조건에 적용하고 결과를 스스로 해석하는 과정**을 한 번 더 반복한 것입니다.

---

# PART9 — 실패 분석 + 결과 정리 + Git 마무리 (59~66)

---

7일차 실습의 마지막 단계입니다.

앞에서 이미 다음 결과를 확인했습니다.

```
Test
→ TP 59 / TN 18
→ FP 2 / FN 4

Challenge
→ Dark / Bright / Shift : FP
→ Blur / Rotation : PASS
```

이제 모든 내용을 다시 반복하지 않고:

```
대표 실패 확인
        ↓
짧게 원인 정리
        ↓
핵심 결과 확인
        ↓
README 정리
        ↓
Git 기록
        ↓
자가체크
```

순서로 7일차를 마무리합니다.

---

# 59. 대표 실패 사례 3개만 확인합니다

모든 실패 이미지를 다시 분석할 필요는 없습니다.

7일차에서 배운 대표적인 실패 유형을 하나씩 선택합니다.

```
① Test FP 1건
→ 정상인데 Anomaly로 잘못 판정

② Test FN 1건
→ 작은 결함인데 Normal로 놓침

③ Challenge FP 1건
→ 제품은 정상인데 촬영 조건 변화로 Anomaly 판정
```

Test FP와 FN 파일을 확인합니다.

```bash
find outputs/day07/fp -maxdepth 1 -type f
```

```bash
find outputs/day07/fn -maxdepth 1 -type f
```

Challenge 결과도 확인합니다.

```bash
cat reports/day07_challenge_results.csv
```

이번 실제 결과에서는 다음과 같은 특징을 확인했습니다.

```
Test FP
→ 정상 2장을 Anomaly로 잘못 판정

Test FN
→ broken_small 4장을 Normal로 놓침

Challenge FP
→ Dark / Bright / Shift에서 발생
```

---

# 60. 실패 원인을 짧게 정리합니다

파일:

```
reports/day07_failure_analysis.md
```

너무 긴 보고서를 작성하지 않습니다.

다음 세 사례만 간단히 기록합니다.

```markdown
# Day07 Failure Analysis

## Test FP

File:

Score:

원인 추정:


## Test FN

File:

Score:

원인 추정:


## Challenge FP

변형 조건:

Score:

원인 추정:


## 한 줄 정리

현재 Baseline이 잘 동작한 경우:

현재 Baseline이 약했던 경우:

다음에 개선하고 싶은 점:
```

여기서 중요한 것은 정답처럼 긴 설명을 작성하는 것이 아닙니다.

```
무엇을 틀렸는가?
        ↓
왜 틀렸을 것 같은가?
```

를 스스로 설명할 수 있으면 충분합니다.

---

# 61. 7일차의 핵심 결과를 확인합니다

7일차에서 만든 파일을 모두 외울 필요는 없습니다.

다음 핵심 결과만 확인합니다.

```
Normal Reference
→ artifacts/day07/normal_reference.npz

Freeze한 Rule
→ configs/day07_anomaly_rules.json

Test 결과
→ reports/day07_anomaly_scores.csv

성능 결과
→ reports/day07_test_metrics.md

Challenge 결과
→ reports/day07_challenge_results.csv

실패 분석
→ reports/day07_failure_analysis.md
```

실행 코드는:

```
src/50 ~ src/61
```

까지 작성했습니다.

이미지 분석 결과는:

```
outputs/day07/fp/
outputs/day07/fn/
outputs/day07/heatmap/
```

에서 확인할 수 있습니다.

> 파일명을 모두 암기하는 것이 아니라 **정상 기준 → Rule → Test 결과 → 실패 분석이라는 결과물이 남았는지** 확인하면 됩니다.

---

# 62. README에 7일차 결과를 짧게 정리합니다

`README.md`에는 수업 내용을 다시 길게 작성하지 않습니다.

다음 정도만 남깁니다.

```markdown
## Day 07 — Anomaly Detection

### Dataset

MVTec AD bottle

### Baseline

- Pretrained ResNet18
- Feature Layer: layer3
- Normal Reference: Normal Train Feature 평균
- Threshold: 0.067469
- Rule Version: day07-anomaly-v1.0

### Test Result

- Accuracy: 0.9277
- Precision: 0.9672
- Recall: 0.9365
- TP: 59
- TN: 18
- FP: 2
- FN: 4

### Challenge

정상 이미지를 Dark / Bright / Blur / Shift / Rotation으로 변형하여
같은 Frozen Rule로 다시 검사했습니다.

- FP: Dark / Bright / Shift
- PASS: Blur / Rotation

### 핵심 정리

정상 데이터로 Normal Reference를 만들고,
Validation에서 Threshold를 결정한 뒤
Freeze한 Rule을 Test에 적용했습니다.

촬영 환경 변화만으로도 FP가 발생할 수 있고,
작은 결함은 FN으로 놓칠 수 있음을 확인했습니다.
```

README의 목적은 **7일차에 무엇을 했고 어떤 결과가 나왔는지 나중에 다시 확인할 수 있게 남기는 것**입니다.

---

# 63. Git에 올리지 않을 데이터를 확인합니다

이미지 Dataset과 실행 중 생성되는 파일은 저장소에 올리지 않습니다.

`.gitignore`에 다음 항목이 있는지 확인합니다.

```
# Day07 image data
samples/day07/

# Generated feature data
artifacts/day07/

# Generated images
outputs/day07/

# Python cache
__pycache__/
*.pyc
```

> 7일차 원본을 포함한 이미지 데이터는 `samples/day07/` 아래에 있으므로 `.gitignore`의 `samples/day07/` 규칙으로 Git 추적에서 제외합니다.

Git에는 주로 다음 내용을 기록합니다.

```
src/
configs/
reports/
README.md
requirements.txt
.gitignore
```

---

# 64. 7일차 작업을 Git에 기록합니다

먼저 변경 내용을 확인합니다.

```bash
git status
```

파일을 추가합니다.

```bash
git add src configs reports README.md requirements.txt .gitignore
```

Commit합니다.

```bash
git commit -m "feat: add day07 anomaly detection baseline"
```

Commit이 만들어졌는지 확인합니다.

```bash
git log --oneline -5
```

여기서는 파일을 하나씩 긴 명령어로 나열할 필요가 없습니다.

> 오늘 작업한 코드와 결과를 Git에 하나의 작업 단위로 기록하면 됩니다.

---

# 65. 저장소에 반영합니다

외부 GitHub 사용이 허용되는 환경이라면 현재 브랜치를 Push합니다.

```bash
git branch --show-current
```

```bash
git push -u origin "$(git branch --show-current)"
```

기관 정책상 외부 GitHub 사용이 제한된다면 Push하지 않고:

```
Local Git
또는
기관 내부 Git 저장소
```

까지만 진행합니다.

마지막으로 다음만 확인합니다.

- 코드가 정상적으로 기록되었는가?
- `day07_anomaly_rules.json`이 포함되었는가?
- Reports가 포함되었는가?
- README가 수정되었는가?
- `samples/day07/` 이미지가 저장소에 올라가지 않았는가?
- `artifacts/day07/`가 저장소에 올라가지 않았는가?

---

# 66. 7일차 자가 체크리스트

수업을 마치기 전에 직접 체크합니다.

## 핵심 개념

- [ ]  Anomaly Detection이 정상 기준과 다른 이미지를 찾는 방법이라는 것을 설명할 수 있다.
- [ ]  Normal Train으로 Normal Reference를 만드는 이유를 설명할 수 있다.
- [ ]  Validation으로 Threshold를 결정하고 Test는 최종 평가에 사용한다는 차이를 설명할 수 있다.
- [ ]  Anomaly Score가 낮으면 정상과 비슷하고, 높으면 정상과 다르다는 것을 설명할 수 있다.
- [ ]  `Threshold = 0.067469`를 기준으로 Normal / Anomaly를 판정할 수 있다.
- [ ]  TP / TN / FP / FN의 의미를 구분할 수 있다.

## 결과 분석

- [ ]  Test 결과 `TP 59 / TN 18 / FP 2 / FN 4`를 확인했다.
- [ ]  `broken_small`에서 FN이 발생한 것을 확인했다.
- [ ]  Heatmap과 Ground Truth가 서로 다른 역할이라는 것을 설명할 수 있다.
- [ ]  정상 이미지도 촬영 조건 변화 때문에 FP가 발생할 수 있음을 확인했다.
- [ ]  Challenge에서 Dark / Bright / Shift가 FP였음을 확인했다.
- [ ]  Threshold를 무조건 높이거나 낮추면 FP와 FN의 균형이 달라질 수 있음을 이해했다.

## 직접 해보기와 마무리

- [ ]  Mini Challenge에서 다른 정상 이미지와 다른 변형 조건을 직접 사용해 보았다.
- [ ]  실행 전에 예상하고 실행 후 실제 Score와 비교해 보았다.
- [ ]  대표 FP / FN / Challenge 실패 사례의 원인을 한 번씩 생각해 보았다.
- [ ]  README에 7일차 핵심 결과를 정리했다.
- [ ]  필요한 코드와 Reports를 Git에 기록했다.
- [ ]  Dataset 이미지가 저장소에 포함되지 않았는지 확인했다.

---

### PART9 핵심

마지막에는 이것만 기억하면 됩니다.

```
7일차 검사 완료
        ↓
FP / FN 확인
        ↓
대표 실패 원인 분석
        ↓
결과 정리
        ↓
README
        ↓
Git 기록
        ↓
자가체크
        ↓
7일차 종료
```

> 7일차 마지막의 목적은 새로운 내용을 더 배우는 것이 아니라, 만든 검사 결과를 정리하고 자신이 전체 흐름을 이해했는지 확인하는 것입니다.

현재 Baseline은 Anomaly Detection의 기본 흐름이 정상적으로 동작하고, 큰 파손과 오염을 잘 찾아내는 수준까지는 확인했습니다. 하지만 작은 결함을 일부 놓치고 촬영 환경 변화에 FP가 발생하므로, 실제 현장에 사용하기 위해서는 추가 개선과 더 많은 실제 환경 평가가 필요합니다.

```
Baseline 제작
        ↓
성능은 어느 정도 나옴
        ↓
하지만 FN 발견
        ↓
환경 변화 FP 발견
        ↓
왜 실패했는지 분석
        ↓
다음 개선 방법 고민
```

