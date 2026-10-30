## 오늘의 핵심 질문

> 같은 실제 이미지를 현실적인 범위에서 변형하면 어떤 학습데이터를 만들 수 있고, VAE는 이미지의 특징을 어떻게 압축하고 다시 복원할까?

다음 질문에 답할 수 있도록 데이터를 나누고, 만들고, 확인하고, 기록하는 전체 흐름을 경험하는 것이 목표입니다.

```text
무엇을 학습용으로 사용할 것인가?
        ↓
어떤 변형이 현실적인가?
        ↓
생성한 데이터는 정상적인가?
        ↓
어떤 원본에서 어떻게 만들었는가?
        ↓
평가용 데이터가 학습에 섞이지 않았는가?
```

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

- 실제 이미지와 Augmented Image의 차이를 설명할 수 있습니다.
- `Apple Red 1` 데이터의 Training과 Test를 구분하여 사용할 수 있습니다.
- Training 데이터에서 Train과 Validation을 다시 나누고, 기존 Test는 평가용으로 유지할 수 있습니다.
- Data Leakage가 무엇인지 설명할 수 있습니다.
- Brightness·Rotation·Blur·Noise가 어떤 촬영 조건을 표현하는지 설명할 수 있습니다.
- 이미지 한 장에 Augmentation을 적용하고 결과를 눈으로 비교할 수 있습니다.
- Train 이미지에만 여러 Augmentation을 적용할 수 있습니다.
- 생성 이미지의 원본·방법·파라미터·Seed를 Metadata로 기록할 수 있습니다.
- VAE의 `Encoder → Latent Space → Decoder → Reconstruction` 흐름을 설명할 수 있습니다.
- VAE를 직접 학습하고 Train Loss와 Validation Loss를 확인할 수 있습니다.
- Test 이미지의 Original과 Reconstruction을 비교할 수 있습니다.
- Latent Space에서 무작위로 Sampling한 이미지가 어떻게 생성되는지 확인할 수 있습니다.
- Automatic QA와 Human QA의 차이를 설명할 수 있습니다.
- Train·Validation·Test 사이의 Leakage를 코드로 확인할 수 있습니다.
- Mini Challenge에서 `Banana 1`을 이용하여 오늘의 전체 Pipeline을 스스로 다시 수행할 수 있습니다.
- Mini Challenge에서 한 번에 하나의 조건만 바꾸고 결과 차이를 설명할 수 있습니다.
- VAE 조건은 Validation으로 비교하고, Test는 선택된 최종 조건의 마지막 확인에만 사용할 수 있습니다.
- 오늘 만든 코드와 실험 결과를 README와 Git에 정리할 수 있습니다.

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260908143606.png]]

오늘은 가지고 있는 이미지를 이용하여 학습 데이터를 더 다양하게 만드는 두 가지 방법을 배웁니다.

#### ==`첫 번째 방법은 Augmentation(데이터 증강) 입니다.`==

이미 가지고 있는 사과 이미지를 조금 밝게 하거나, 돌리거나, 흐리게 하거나, Noise를 넣어 
새로운 학습 이미지처럼 만드는 방법입니다.

```
원본 사과 이미지
        ↓
밝기 변경
회전
Blur
Noise
        ↓
조금씩 다른 사과 이미지 생성
```

#### ==`두 번째 방법은 VAE입니다.`==

VAE는 이미지를 단순히 돌리거나 밝게 만드는 것이 아니라, 
여러 사과 이미지를 보면서 “사과 이미지에는 어떤 공통적인 특징이 있는가?”를 모델이 학습하도록 하는 방법입니다.

학습이 끝나면 입력한 이미지를 다시 복원해 보거나, 
학습한 특징을 이용하여 새로운 형태의 이미지를 만들어 볼 수 있습니다.

```
여러 사과 이미지
        ↓
VAE가 이미지 특징 학습
        ↓
특징을 압축하여 기억
        ↓
이미지 다시 복원
또는
새로운 이미지 생성
```

따라서 오늘은 같은 사과 데이터를 가지고 서로 다른 두 가지 데이터 생성 방법을 비교합니다.

![[Pasted image 20260924175947.png]]

```
                    Apple Red 1 이미지
                           ↓
                  학습용 Train 준비
                           ↓
             ┌─────────────┴─────────────┐
             │                           │
             ↓                           ↓
       실험 A                       실험 B

   Augmentation                      VAE

원본 이미지를 직접              이미지 특징을
조금씩 변형                     모델이 학습
             │                           │
             ↓                           ↓
   변형 이미지 생성              복원 이미지 확인
                                 생성 이미지 확인
             │                           │
             └─────────────┬─────────────┘
                           ↓
                    결과가 정상인지 확인
                           ↓
                    두 방법 결과 비교
```

여기서 가장 중요한 것은 두 실험을 억지로 하나로 연결하는 것이 아닙니다.

```
Augmentation 결과
        ↓
VAE 입력
```

처럼 사용하는 것이 아니라,

```
같은 원본 Train
        ├─ Augmentation 실험
        └─ VAE 실험
```

처럼 같은 데이터에서 출발하여 서로 다른 방법을 각각 경험해 보는 것입니다.

오늘 수업이 끝나면 다음 차이를 이해하는 것이 가장 중요합니다.

```
Augmentation
→ 사람이 정한 방법으로 기존 이미지를 변형한다.

VAE
→ 모델이 여러 이미지를 보고 특징을 학습한다.
```

처음부터 어려운 생성모델의 수식을 이해하는 것이 목표는 아닙니다.

오늘은 직접 이미지를 만들고 결과를 눈으로 비교하면서

> “기존 이미지를 변형하는 것과 모델이 이미지 특징을 학습하는 것은 어떻게 다른가?”

를 이해하는 것이 첫 번째 목표입니다.

---

# 오늘 사용할 실제 데이터

## 1. 본 실습 — Fruits-360 `Apple Red 1`

오늘의 기본 실습 데이터는 **빨간 사과 이미지**입니다.

현재 준비된 데이터는 약 600장 규모이며, Fruits-360의 원래 구조를 유지하여 사용합니다.

대표적인 구성은 다음과 같습니다.

```text
Apple Red 1

Training
→ 약 492장

Test
→ 약 164장
```

실제 수량은 제공된 파일을 코드로 다시 확인합니다.

오늘은 다음처럼 사용합니다.

```text
기존 Training
        ↓
Train + Validation로 분리

기존 Test
        ↓
그대로 유지
```

예를 들어 Training이 492장이라면:

```text
Training 492장
        ↓
Train 약 419장
Validation 약 73장

Test 164장
→ 그대로 유지
```

> 숫자를 외우는 것이 목적은 아닙니다. 실제 파일 수에 따라 코드가 자동으로 계산합니다.

### 왜 빨간 사과 데이터를 사용하나요?

`Apple Red 1`은 한 종류의 객체가 비슷한 배경과 조건으로 반복되어 있어 1일차 생성모델의 기본 원리를 확인하기 좋습니다.

```text
같은 종류의 객체
        +
조금씩 다른 회전·형태
        ↓
Augmentation 변화가 눈에 잘 보임
        ↓
VAE가 어떤 특징을 유지하고
어떤 세부정보를 잃는지 비교하기 쉬움
```

오늘은 이 데이터가 실제 제조라인 데이터와 똑같다고 가정하지 않습니다.

**통제된 교육 데이터로 생성·보강 원리를 먼저 익히는 것**이 목적입니다.

---

## 2. Mini Challenge — Fruits-360 `Banana 1`

Mini Challenge에서는 `Banana 1`을 사용합니다.

```text
본 실습
Apple Red 1
→ 빨간색
→ 둥근 형태

Mini Challenge
Banana 1
→ 노란색
→ 길고 휘어진 형태
```

파일 형식과 데이터 성격은 비슷하지만 객체의 색과 형태가 확실히 다릅니다.

따라서 본 실습 코드를 그대로 복사하여 끝내는 것이 아니라 다음 내용을 다시 판단해야 합니다.

```text
같은 Augmentation 범위를 사용해도 되는가?

VAE는 사과와 다른 형태의 특징도 복원하는가?

Latent Dimension을 바꾸면
복원 결과가 어떻게 달라지는가?
```

Mini Challenge도 Fruits-360의 Training/Test 구분을 유지합니다.

---

## 3. 오늘 데이터에는 별도 라벨 파일이 필요하지 않습니다

1일차의 핵심은 Classification이나 Detection 학습이 아닙니다.

따라서 다음 파일은 필요하지 않습니다.

```text
Class Label TXT
BBox
YOLO Label
Mask
Segmentation Polygon
```

오늘 필요한 것은 **이미지 자체**입니다.

---

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | 전체 실습 코드 작성 | 데이터를 나누고 만들고 기록 |
| pathlib | 파일·폴더 경로 처리 | 이미지 위치를 안전하게 관리 |
| OpenCV | 이미지 변형·읽기·저장 | 밝기·회전·Blur·Noise 적용 |
| NumPy | Noise 생성·배열 계산 | 픽셀 값을 숫자로 처리 |
| Pandas | Metadata·Loss·QA CSV 저장 | 실험 결과를 표로 기록 |
| Matplotlib | Augmentation 결과 비교 | 여러 이미지를 한 화면에서 확인 |
| Pillow | VAE 입력 이미지 읽기 | RGB 이미지 로드 |
| PyTorch | VAE 구현·학습 | Encoder와 Decoder 학습 |
| TorchVision | Resize·Tensor 변환·결과 저장 | VAE용 이미지 전처리 |
| Train / Validation / Test | 실험 기준 고정 | 학습과 평가 데이터 분리 |
| Data Leakage | 평가 오류 방지 | Test 정보가 Train에 섞이지 않게 관리 |
| Metadata | 생성 이력 기록 | 어떤 원본을 어떻게 바꿨는지 저장 |
| Automatic / Human QA | 합성 이미지 품질검수 | 코드 검사 + 사람이 눈으로 확인 |
| Git | 코드와 실험 기록 저장 | 오늘 작업을 재현 가능한 상태로 보관 |

---

# PART 1. 교과 11의 1일차 프로젝트와 실행환경을 준비합니다

---

# 1. 교과 11에서 1일차가 어디에 해당하는지 확인합니다

교과 11에서는 5일 동안 서로 다른 합성·보강 방법을 경험합니다.

```text
1일차
일반 Augmentation + VAE
        ↓
2일차
GAN + WGAN-GP + Diffusion
        ↓
3일차
Cut-Paste + BBox 자동 생성
        ↓
4일차
3D Rendering + Segmentation
        ↓
5일차
Keypoint Sequence Augmentation
```

오늘은 그 시작입니다.

처음부터 복잡한 생성모델로 들어가지 않고 다음 두 가지를 먼저 구분합니다.

```text
Augmentation
→ 기존 이미지를 직접 변형

VAE
→ 이미지 특징을 압축하고
  다시 복원하는 생성모델
```

오늘 만들어 놓은 데이터 관리 방식은 2~5일차에도 반복해서 사용합니다.

```text
데이터 준비
→ 생성·보강
→ QA
→ Metadata
→ Leakage 방지
→ 결과 비교
```

---

# 2. 교과 11 프로젝트 폴더를 만듭니다

전체 디렉토리 구조

```
subject11_synthetic_data/
│
├─ .venv/                     ← 교과 11 전체 공통 Python 환경
├─ requirements.txt           ← 교과 11 공통 라이브러리 목록
├─ README.md                  ← 교과 11 전체 설명
├─ .gitignore                 ← 교과 11 전체 Git 제외 규칙
│
├─ day01_image_aug_vae/       ← 오늘 작업
├─ day02_...                  ← 2일차
├─ day03_...                  ← 3일차
├─ day04_...                  ← 4일차
└─ day05_...                  ← 5일차
```

오늘은 `day01_image_aug_vae`만 만듭니다.  
2~5일차 폴더는 각 수업을 시작할 때 추가합니다.

WSL 또는 Ubuntu 터미널을 실행합니다.

```bash
cd ~/ai_vision
```

교과 11 프로젝트 폴더를 만듭니다.

```bash
mkdir -p subject11_synthetic_data
cd subject11_synthetic_data
```

현재 위치를 확인합니다.

```bash
pwd
```

다음과 비슷하면 됩니다.

```text
/home/사용자계정/ai_vision/subject11_synthetic_data
```

## 교과 11 전체에서 사용할 공통 파일을 만듭니다

다음 세 파일은 1일차만 사용하는 파일이 아니라 **교과 11 전체에서 공통으로 사용**합니다.

```bash
touch requirements.txt
touch README.md
touch .gitignore
```

각 파일의 역할은 다음과 같습니다.

```
requirements.txt
→ 교과 11에서 사용하는 Python 라이브러리 기록

README.md
→ 교과 11의 전체 실습 흐름과 실행방법 기록

.gitignore
→ 데이터·모델·가상환경 등
  Git에 저장하지 않을 파일 지정
```

## 1일차 작업 폴더를 만듭니다

이제 오늘 사용할 폴더를 만듭니다.

```bash
mkdir -p day01_image_aug_vae
cd day01_image_aug_vae
```

현재 위치를 확인합니다.

```bash
pwd
```

다음과 비슷하면 됩니다.

```
/home/사용자계정/ai_vision/subject11_synthetic_data/day01_image_aug_vae
```

## 1일차 실습에 필요한 폴더를 만듭니다

```bash
mkdir -p \
src \
scripts \
data/apple/source_train \
data/apple/source_test \
data/apple/split/train \
data/apple/split/val \
data/apple/split/test \
data/apple/augmented/train \
data/banana/source_train \
data/banana/source_test \
data/banana/split/train \
data/banana/split/val \
data/banana/split/test \
data/banana/augmented/train \
models/apple \
models/banana \
results/apple/augmentation \
results/apple/vae \
results/banana/augmentation \
results/banana/vae \
reports/apple \
reports/banana
```

1일차에서 작성할 파일도 미리 만듭니다.

```bash
touch day01_notes.md

touch src/day01_common.py

touch scripts/00_check_env.py
touch scripts/01_check_dataset.py
touch scripts/02_prepare_split.py
touch scripts/03_augment_preview.py
touch scripts/04_augment_train.py
touch scripts/05_train_vae.py
touch scripts/06_qa_images.py
touch scripts/07_check_leakage.py
```

## 현재 폴더 구조를 확인합니다

교과 11 Root로 한 단계 이동합니다.

```bash
cd ..
```

현재 위치를 확인합니다.

```bash
pwd
```

```
.../subject11_synthetic_data
```

구조를 확인합니다.

```bash
tree -L 4
```

`tree` 명령이 없다면 설치합니다.

```bash
sudo apt install tree
```

현재는 대략 다음과 같은 구조가 보이면 됩니다.

```
├── README.md
├── day01_image_aug_vae
│   ├── data
│   │   ├── apple
│   │   │   ├── augmented
│   │   │   ├── source_test
│   │   │   ├── source_train
│   │   │   └── split
│   │   └── banana
│   │       ├── augmented
│   │       ├── source_test
│   │       ├── source_train
│   │       └── split
│   ├── day01_notes.md
│   ├── models
│   │   ├── apple
│   │   └── banana
│   ├── reports
│   │   ├── apple
│   │   └── banana
│   ├── results
│   │   ├── apple
│   │   │   ├── augmentation
│   │   │   └── vae
│   │   └── banana
│   │       ├── augmentation
│   │       └── vae
│   ├── scripts
│   │   ├── 00_check_env.py
│   │   ├── 01_check_dataset.py
│   │   ├── 02_prepare_split.py
│   │   ├── 03_augment_preview.py
│   │   ├── 04_augment_train.py
│   │   ├── 05_train_vae.py
│   │   ├── 06_qa_images.py
│   │   └── 07_check_leakage.py
│   └── src
│       └── day01_common.py
└── requirements.txt
```

### 지금 무엇을 한 것인가요?

```
교과 11 전체 프로젝트 생성
        ↓
subject11_synthetic_data
        ↓
교과 전체 공통 파일 준비
        ↓
1일차 작업 폴더 생성
        ↓
Apple 본 실습
+
Banana Mini Challenge
        ↓
data / models / results / reports 분리
```

`Apple Red 1` 본 실습과 
`Banana 1` Mini Challenge의 데이터와 결과를 서로 다른 폴더에 저장하므로 
나중에 실험 결과를 구분하고 비교하기 쉽습니다.

---

# 3. Python 가상환경을 만들고 필요한 라이브러리를 설치합니다

교과 11에서는 **매일 새로운 가상환경을 만들지 않습니다.**

오늘 프로젝트 Root에 `.venv`를 **한 번만 만들고**, 2~5일차에는 이 환경을 다시 활성화하여 사용합니다.

```
교과 11 Root
        ↓
      .venv
        ↓
 ┌──────┼──────┬──────┬──────┐
 ↓      ↓      ↓      ↓      ↓
1일차   2일차   3일차   4일차   5일차
```

새로운 라이브러리가 필요한 날에는 기존 `.venv`에 필요한 패키지만 추가합니다.

---

## 현재 위치를 확인합니다

현재 위치가 교과 11 Root인지 확인합니다.

```bash
pwd
```

다음과 비슷해야 합니다.

```
/home/사용자계정/ai_vision/subject11_synthetic_data
```

> `day01_image_aug_vae` 안에서 가상환경을 만들지 않습니다.

---

## 교과 11 공통 가상환경을 만듭니다

```bash
python3 -m venv .venv
```

구조는 다음과 같이 됩니다.

```
subject11_synthetic_data/
│
├─ .venv/
├─ requirements.txt
├─ README.md
├─ .gitignore
│
└─ day01_image_aug_vae/
```

`.venv`와 `day01_image_aug_vae`가 **같은 단계에 있는 것**이 중요합니다.

---

## 가상환경을 활성화합니다

```bash
source .venv/bin/activate
```

정상적으로 활성화되면 터미널 앞부분에 다음처럼 표시될 수 있습니다.

```bash
(.venv) 사용자계정@PC:...
```

Python이 어느 위치에서 실행되는지도 확인합니다.

```bash
which python
```

다음과 비슷하면 정상입니다.

```
/home/사용자계정/ai_vision/subject11_synthetic_data/.venv/bin/python
```

즉,

```
day01_image_aug_vae/.venv
```

가 아니라

```
subject11_synthetic_data/.venv
```

를 사용하고 있어야 합니다.

---

## 공통 `requirements.txt`를 작성합니다

프로젝트 Root의 `requirements.txt`에 1일차에서 필요한 라이브러리를 작성합니다.

```
numpy
pandas
matplotlib
opencv-python
Pillow
torch
torchvision
```

현재 파일 위치는 다음입니다.

```
subject11_synthetic_data/
└─ requirements.txt
```

1일차에서는 위 라이브러리부터 설치합니다.

2~5일차에서 추가 라이브러리가 필요하면 **이 파일에 추가하면서 같은 환경을 계속 사용**합니다.

---

## 필요한 라이브러리를 설치합니다

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

설치가 끝난 뒤 주요 패키지가 보이는지 간단히 확인할 수 있습니다.

```bash
pip list
```

---

## 현재 환경을 1일차 기록으로 저장합니다

1일차에서 사용한 Python 환경을 나중에 확인할 수 있도록 기록합니다.

```bash
pip freeze > day01_image_aug_vae/reports/environment_freeze.txt
```

파일은 다음 위치에 만들어집니다.

```
day01_image_aug_vae/
└─ reports/
   └─ environment_freeze.txt
```

> `environment_freeze.txt`에는 오늘 실제로 설치되어 있던 패키지와 버전이 기록됩니다.  
> 이후 문제가 생겼을 때 1일차 실행환경을 확인하거나 같은 환경을 다시 구성할 때 참고할 수 있습니다.

## 이제 1일차 실습 폴더로 이동합니다

가상환경은 교과 11 Root에 만들어졌습니다.

이제 실제 1일차 코드를 실행하기 위해 `day01_image_aug_vae` 폴더로 이동합니다.

```bash
cd day01_image_aug_vae
```

현재 위치를 확인합니다.

```bash
pwd
```

다음과 비슷하면 됩니다.

```bash
/home/사용자계정/ai_vision/subject11_synthetic_data/day01_image_aug_vae
```

---

# 4. Python·PyTorch·GPU 환경을 확인하는 코드를 작성합니다

먼저 Python과 주요 라이브러리가 정상적으로 설치되어 있고, PyTorch가 GPU를 사용할 수 있는지 확인합니다.

이 확인을 먼저 해두면 뒤에서 오류가 발생했을 때 **환경 문제인지 코드 문제인지** 구분하기 쉬워집니다.

**파일: `scripts/00_check_env.py`**

```python
import sys

import cv2
import numpy as np
import pandas as pd
import torch
import torchvision
from PIL import Image


def main() -> None:
    print("=" * 60)
    print("Subject 11 - Day 01 Environment Check")
    print("=" * 60)

    print("Python      :", sys.version.split()[0])
    print("OpenCV      :", cv2.__version__)
    print("NumPy       :", np.__version__)
    print("Pandas      :", pd.__version__)
    print("PyTorch     :", torch.__version__)
    print("TorchVision :", torchvision.__version__)
    print("Pillow      :", Image.__version__)

    print("-" * 60)

    cuda_available = torch.cuda.is_available()

    print("CUDA Available :", cuda_available)

    if cuda_available:
        print("GPU            :", torch.cuda.get_device_name(0))
    else:
        print("GPU            : CPU mode")

    print("=" * 60)
    print("Environment check completed.")


if __name__ == "__main__":
    main()
```

### ① 전체 흐름을 빠르게 이해하는 의사코드

```text
필요한 라이브러리를 불러온다

프로그램을 시작한다

제목을 출력한다

현재 설치된 버전을 확인한다
    Python 버전 확인
    OpenCV 버전 확인
    NumPy 버전 확인
    Pandas 버전 확인
    PyTorch 버전 확인
    TorchVision 버전 확인
    Pillow 버전 확인

CUDA를 사용할 수 있는지 확인한다

만약 CUDA를 사용할 수 있다면
    현재 사용 가능한 NVIDIA GPU 이름을 출력한다
그렇지 않다면
    CPU 모드라고 출력한다

환경 확인이 완료되었다는 메시지를 출력한다

main 함수를 실행한다
```

### ② 실제 코드와 1:1로 대응하는 의사코드

```python
import sys
# → Python 자체의 버전 정보를 확인하기 위해 sys를 불러온다

import cv2
# → OpenCV 버전과 이미지 처리 기능을 사용하기 위해 cv2를 불러온다

import numpy as np
# → NumPy 버전과 배열 계산 기능을 사용하기 위해 NumPy를 불러온다

import pandas as pd
# → Pandas 버전과 표 형태 데이터 처리 기능을 사용하기 위해 Pandas를 불러온다

import torch
# → PyTorch 버전과 GPU 사용 가능 여부를 확인하기 위해 PyTorch를 불러온다

import torchvision
# → TorchVision 버전을 확인하고 이미지 관련 PyTorch 기능을 사용하기 위해 불러온다

from PIL import Image
# → Pillow 이미지 라이브러리의 Image 기능을 불러온다


def main() -> None:
# → 환경 확인 작업을 수행할 main 함수를 만든다


    print("=" * 60)
# → 구분선 "="을 60개 출력한다

    print("Subject 11 - Day 01 Environment Check")
# → 환경 확인 프로그램의 제목을 출력한다

    print("=" * 60)
# → 다시 구분선을 출력한다


    print("Python      :", sys.version.split()[0])
# → 현재 사용하고 있는 Python 버전을 출력한다

    print("OpenCV      :", cv2.__version__)
# → 설치된 OpenCV 버전을 출력한다

    print("NumPy       :", np.__version__)
# → 설치된 NumPy 버전을 출력한다

    print("Pandas      :", pd.__version__)
# → 설치된 Pandas 버전을 출력한다

    print("PyTorch     :", torch.__version__)
# → 설치된 PyTorch 버전을 출력한다

    print("TorchVision :", torchvision.__version__)
# → 설치된 TorchVision 버전을 출력한다

    print("Pillow      :", Image.__version__)
# → 설치된 Pillow 버전을 출력한다


    print("-" * 60)
# → 라이브러리 정보와 GPU 정보를 구분하기 위해 "-" 구분선을 출력한다


    cuda_available = torch.cuda.is_available()
# → PyTorch가 CUDA를 사용할 수 있는지 확인하고
  그 결과를 cuda_available 변수에 저장한다


    print("CUDA Available :", cuda_available)
# → CUDA 사용 가능 여부를 True 또는 False로 출력한다


    if cuda_available:
# → CUDA를 사용할 수 있다면 아래 코드를 실행한다


        print("GPU            :", torch.cuda.get_device_name(0))
# → PyTorch가 인식한 첫 번째 NVIDIA GPU의 이름을 출력한다


    else:
# → CUDA를 사용할 수 없다면 아래 코드를 실행한다


        print("GPU            : CPU mode")
# → GPU 대신 CPU 모드로 실행된다고 출력한다


    print("=" * 60)
# → 마지막 구분선을 출력한다

    print("Environment check completed.")
# → 환경 확인 작업이 끝났다는 메시지를 출력한다


if __name__ == "__main__":
# → 이 Python 파일을 직접 실행한 경우인지 확인한다


    main()
# → 직접 실행한 경우 main 함수를 실행한다
```

### 이 코드는 왜 작성하나요?

본격적인 실습을 시작하기 전에 Python과 주요 라이브러리, GPU가 정상적으로 준비되어 있는지 확인합니다.

- Augmentation(데이터 증강)  
  기존 이미지를 밝게 하거나, 회전시키거나, 흐리게 하는 등 조금씩 변형하여 학습 데이터를 
  다양하게 만드는 방법입니다.  
  Augmentation은 대부분 CPU에서도 충분히 실행할 수 있습니다.

- VAE(Variational Autoencoder)  
  여러 이미지를 보면서 공통적인 특징을 학습하고, 그 특징을 이용해 이미지를 다시 복원하거나 새로운 이미지를 만들어 보는 생성모델입니다.  
  VAE 학습은 계산량이 많기 때문에 GPU를 사용하면 훨씬 빠르게 실행할 수 있습니다.

따라서 실습 전에 다음 항목을 확인합니다.

```text
Python이 정상적으로 실행되는가?
        ↓
OpenCV · NumPy · Pandas가 설치되어 있는가?
        ↓
PyTorch · TorchVision이 정상인가?
        ↓
CUDA를 사용할 수 있는가?
        ↓
GPU가 정상적으로 인식되는가?
```

실행합니다.

```bash
cd day01_image_aug_va
python scripts/00_check_env.py
```

실행 결과는 다음과 같습니다.

```
============================================================
Subject 11 - Day 01 Environment Check
============================================================
Python      : 3.14.4
OpenCV      : 5.0.0
NumPy       : 2.5.3
Pandas      : 3.0.6
PyTorch     : 2.14.1+cu130
TorchVision : 0.29.1+cu130
Pillow      : 12.3.0
------------------------------------------------------------
CUDA Available : True
GPU            : NVIDIA GeForce RTX 4060
============================================================
Environment check completed.
```

### 결과를 어떻게 해석하나요?

실행 결과에서 다음과 같이 출력되었습니다.

```
CUDA Available : True
GPU            : NVIDIA GeForce RTX 4060
```

`CUDA Available : True`가 출력되었으므로 
**PyTorch가 NVIDIA GPU와 CUDA를 정상적으로 사용할 수 있는 상태**입니다.

또한 현재 실습 PC의 GPU가 다음과 같이 정상적으로 인식되었습니다.

```
NVIDIA GeForce RTX 4060
```

따라서 현재 환경에서는 다음 실습을 진행할 수 있습니다.

```
Augmentation
→ OpenCV·NumPy 등을 이용하여 주로 CPU에서 처리
→ 필요에 따라 PyTorch 기반 GPU 연산도 사용 가능

VAE
→ PyTorch + CUDA
→ NVIDIA GeForce RTX 4060을 이용하여 GPU 학습 가능
```

현재 확인된 주요 환경도 정상적으로 로드되었습니다.

```
Python      : 3.14.4
OpenCV      : 5.0.0
NumPy       : 2.5.3
Pandas      : 3.0.6
PyTorch     : 2.14.1+cu130
TorchVision : 0.29.1+cu130
Pillow      : 12.3.0
CUDA        : 사용 가능
GPU         : NVIDIA GeForce RTX 4060
```

> `Environment check completed.`가 출력되고 
> `CUDA Available : True`와 GPU 이름이 정상적으로 표시되었다면, 
> **오늘 진행할 Augmentation과 VAE 실습에 필요한 기본 Python·PyTorch·GPU 실행환경이 정상적으로 준비된 것입니다.**

---

# 5. Apple과 Banana에서 함께 사용할 공통 기능을 만듭니다

오늘은 두 종류의 데이터를 사용합니다.

```
본 실습
→ Apple Red 1

Mini Challenge
→ Banana 1
```

두 데이터는 이미지 내용은 다르지만, 앞으로 수행하는 작업의 구조는 거의 같습니다.

```
원본 이미지 확인
        ↓
Train / Validation / Test 분리
        ↓
Augmentation
        ↓
VAE
        ↓
QA
        ↓
Leakage 확인
```

예를 들어 Apple을 사용할 때도 다음과 같은 폴더가 필요합니다.

```
data/apple/source_train
data/apple/split/train
data/apple/split/val
data/apple/split/test
results/apple
reports/apple
models/apple
```

Banana도 구조는 같습니다.

```
data/banana/source_train
data/banana/split/train
data/banana/split/val
data/banana/split/test
results/banana
reports/banana
models/banana
```

데이터 이름만 다를 뿐 폴더 구조와 처리 방법은 같습니다.

따라서 Apple용 경로 코드와 Banana용 경로 코드를 따로 작성하지 않고, **데이터 이름만 전달하면 필요한 폴더를 찾아주는 공통 기능**을 만들어 사용합니다.

**파일: `src/day01_common.py`**

```python
from __future__ import annotations

from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]

VALID_EXTENSIONS = {
    ".jpg",
    ".jpeg",
    ".png",
    ".bmp",
}


def list_images(directory: Path) -> list[Path]:
    if not directory.exists():
        return []

    return sorted(
        path
        for path in directory.iterdir()
        if path.is_file()
        and path.suffix.lower() in VALID_EXTENSIONS
    )


def dataset_paths(dataset: str) -> dict[str, Path]:
    data_root = ROOT / "data" / dataset
    result_root = ROOT / "results" / dataset
    report_root = ROOT / "reports" / dataset
    model_root = ROOT / "models" / dataset

    paths = {
        "source_train": data_root / "source_train",
        "source_test": data_root / "source_test",
        "split_train": data_root / "split" / "train",
        "split_val": data_root / "split" / "val",
        "split_test": data_root / "split" / "test",
        "augmented_train": data_root / "augmented" / "train",
        "result_aug": result_root / "augmentation",
        "result_vae": result_root / "vae",
        "report": report_root,
        "model": model_root,
    }

    for path in paths.values():
        path.mkdir(
            parents=True,
            exist_ok=True,
        )

    return paths
```
### 1. 전체 기능을 빠르게 이해하는 의사코드

```text
이 파일이 있는 위치를 기준으로
1일차 프로젝트의 ROOT 폴더를 찾는다

사용할 이미지 확장자를 정한다
→ jpg / jpeg / png / bmp


[이미지 목록을 찾는 기능]

이미지 폴더를 입력받는다
        ↓
폴더가 없으면
→ 빈 목록을 반환한다

폴더가 있으면
→ 폴더 안의 파일을 확인한다
→ 이미지 확장자만 선택한다
→ 파일 이름 순서로 정렬한다
→ 이미지 목록을 반환한다


[데이터셋 경로를 준비하는 기능]

데이터 이름을 입력받는다
예: "apple" 또는 "banana"
        ↓
해당 데이터의
data / results / reports / models
기준 경로를 만든다
        ↓
source_train
source_test
split/train
split/val
split/test
augmented/train
augmentation 결과
VAE 결과
report
model
경로를 준비한다
        ↓
필요한 폴더가 없으면 자동으로 만든다
        ↓
모든 경로를 Dictionary로 반환한다


결과적으로

dataset_paths("apple")
→ Apple 실습용 폴더들을 사용

dataset_paths("banana")
→ Banana 실습용 폴더들을 사용

list_images(폴더)
→ 해당 폴더의 이미지 파일 목록을 가져옴
```

### 2. 실제 코드와 1:1로 대응하여 이해하는 코드

```python
# 최신 Python의 타입 힌트 기능을 보다 안정적으로 사용할 수 있게 합니다.
from __future__ import annotations


# 파일과 폴더 경로를 편리하게 다루기 위해 Path를 가져옵니다.
from pathlib import Path


# 현재 day01_common.py 파일의 위치를 기준으로
# 한 단계 위의 1일차 프로젝트 폴더를 ROOT로 지정합니다.
#
# 예:
# day01_image_aug_vae/
# ├─ src/
# │  └─ day01_common.py
# ├─ data/
# ├─ results/
# ├─ reports/
# └─ models/
#
# ROOT → day01_image_aug_vae/
ROOT = Path(__file__).resolve().parents[1]


# 오늘 이미지 파일로 인정할 확장자를 모아 둡니다.
VALID_EXTENSIONS = {

    # JPG 이미지
    ".jpg",

    # JPEG 이미지
    ".jpeg",

    # PNG 이미지
    ".png",

    # BMP 이미지
    ".bmp",
}


# 특정 폴더에서 이미지 파일만 찾아 목록으로 반환하는 함수를 만듭니다.
#
# directory : 검사할 폴더
# 반환값    : 이미지 파일 경로들의 list
def list_images(directory: Path) -> list[Path]:

    # 전달받은 폴더가 존재하지 않는지 확인합니다.
    if not directory.exists():

        # 폴더가 없다면 오류를 내지 않고 빈 목록을 반환합니다.
        return []


    # 조건에 맞는 이미지 파일을 찾아
    # 이름 순서대로 정렬하여 반환합니다.
    return sorted(

        # 조건을 통과한 path를 결과 목록에 넣습니다.
        path

        # 입력받은 폴더 안의 항목을 하나씩 확인합니다.
        for path in directory.iterdir()

        # 현재 항목이 폴더가 아니라 실제 파일인지 확인합니다.
        if path.is_file()

        # 파일 확장자를 소문자로 바꾼 뒤
        # jpg, jpeg, png, bmp 중 하나인지 확인합니다.
        and path.suffix.lower() in VALID_EXTENSIONS
    )


# apple 또는 banana 같은 데이터 이름을 입력받아
# 해당 데이터에 필요한 폴더 경로를 준비하는 함수를 만듭니다.
#
# 예:
# dataset_paths("apple")
# dataset_paths("banana")
#
# 반환값:
# {"source_train": Path(...), ...} 형태의 Dictionary
def dataset_paths(dataset: str) -> dict[str, Path]:

    # 해당 데이터의 data 폴더 기준 경로를 만듭니다.
    # apple이면 → ROOT/data/apple
    # banana이면 → ROOT/data/banana
    data_root = ROOT / "data" / dataset

    # 해당 데이터의 결과 저장 기준 경로를 만듭니다.
    # 예: ROOT/results/apple
    result_root = ROOT / "results" / dataset

    # 해당 데이터의 보고서 저장 기준 경로를 만듭니다.
    # 예: ROOT/reports/apple
    report_root = ROOT / "reports" / dataset

    # 해당 데이터의 모델 저장 기준 경로를 만듭니다.
    # 예: ROOT/models/apple
    model_root = ROOT / "models" / dataset


    # 앞으로 자주 사용할 세부 폴더 경로에
    # 이해하기 쉬운 이름을 붙여 Dictionary로 정리합니다.
    paths = {

        # 원본 Training 이미지 폴더
        "source_train": data_root / "source_train",

        # 원본 Test 이미지 폴더
        "source_test": data_root / "source_test",

        # 분리한 Train 이미지 폴더
        "split_train": data_root / "split" / "train",

        # 분리한 Validation 이미지 폴더
        "split_val": data_root / "split" / "val",

        # 평가에 사용할 Test 이미지 폴더
        "split_test": data_root / "split" / "test",

        # Augmentation으로 만든 Train 이미지 저장 폴더
        "augmented_train": data_root / "augmented" / "train",

        # Augmentation 실험 결과 저장 폴더
        "result_aug": result_root / "augmentation",

        # VAE 실험 결과 저장 폴더
        "result_vae": result_root / "vae",

        # QA 등의 보고서 저장 폴더
        "report": report_root,

        # 학습한 모델 저장 폴더
        "model": model_root,
    }


    # 위에서 만든 모든 경로를 하나씩 가져옵니다.
    for path in paths.values():

        # 해당 폴더가 없으면 자동으로 생성합니다.
        path.mkdir(

            # 중간 상위 폴더가 없어도 함께 생성합니다.
            parents=True,

            # 이미 폴더가 존재해도 오류를 발생시키지 않습니다.
            exist_ok=True,
        )


    # 준비된 모든 폴더 경로를 Dictionary 형태로 반환합니다.
    return paths
```

---
## 이 파일은 무엇을 하나요?

이 파일은 **실험을 직접 실행하는 파일이 아닙니다.**

Apple과 Banana를 사용할 때 필요한 폴더 위치를 다른 Python 파일들에게 알려주는 **공통 경로 도우미 파일**입니다.

쉽게 보면 다음 역할을 합니다.

```
다른 실습 코드
        ↓
apple 또는 banana라는 이름 전달
        ↓
day01_common.py
        ↓
어느 data / results / reports / models
폴더를 사용해야 하는지 결정
```

## 어떤 코드가 Apple과 Banana를 구분하나요?

가장 중요한 부분은 다음 함수입니다.

```python
def dataset_paths(dataset: str) -> dict[str, Path]:
```

여기서 `dataset`에는 나중에

```
apple
```

또는

```
banana
```

가 들어옵니다.

그리고 다음 코드가 실제 경로를 만듭니다.

```python
data_root = ROOT / "data" / dataset
result_root = ROOT / "results" / dataset
report_root = ROOT / "reports" / dataset
model_root = ROOT / "models" / dataset
```

예를 들어 `dataset`이 `apple`이면 다음처럼 바뀝니다.

```
data/apple
results/apple
reports/apple
models/apple
```

반대로 `dataset`이 `banana`이면 다음처럼 됩니다.

```
data/banana
results/banana
reports/banana
models/banana
```

즉 이 한 줄의 `dataset` 값이 **어느 데이터 폴더를 사용할지 결정하는 스위치 역할**을 합니다. 1일차_AppleRed1_Augmentation_VAE_…

---

## 그러면 `--dataset apple`은 어디에 있나요?

`--dataset apple`은 이 공통 파일 안에 있는 코드가 아닙니다.

뒤에서 실제로 실행하는 스크립트가 받습니다.

예를 들어 `scripts/01_check_dataset.py`에는 다음 코드가 있습니다.

```python
parser.add_argument(
    "--dataset",
    default="apple",
    choices=["apple", "banana"],
)
```

다음처럼 실행하면

```python
python scripts/01_check_dataset.py --dataset apple
```

`apple`이라는 값이 `args.dataset`에 들어갑니다.

그리고 다음 코드가 실행됩니다.

```python
paths = dataset_paths(args.dataset)
```

즉 전체 흐름은 다음과 같습니다.

```
python scripts/01_check_dataset.py --dataset apple
                    ↓
             args.dataset
                    ↓
                 "apple"
                    ↓
       dataset_paths("apple")
                    ↓
       data/apple
       results/apple
       reports/apple
       models/apple
```

Mini Challenge에서는 명령어만 바꿉니다.

```bash
python scripts/01_check_dataset.py --dataset banana
```

그러면:

```
--dataset banana
        ↓
args.dataset = "banana"
        ↓
dataset_paths("banana")
        ↓
data/banana
results/banana
reports/banana
models/banana
```

로 자동으로 바뀝니다. 실제 `01_check_dataset.py`가 `--dataset` 값을 받은 뒤 `dataset_paths(args.dataset)`를 호출하는 구조입니다. 1일차_AppleRed1_Augmentation_VAE_…

---

## `ROOT`는 무엇인가요?

다음 코드도 중요합니다.

```python
ROOT = Path(__file__).resolve().parents[1]
```

대문자로 작성된 변수는 변경하지 말고 기준값처럼 사용하자는 파이썬의 관례입니다.
`parents[1]`는 **현재 파일을 기준으로 두 단계 위 폴더**입니다.

현재 파일은 다음 위치에 있습니다.

```
day01_image_aug_vae/
└─ src/
   └─ day01_common.py
```

따라서 `ROOT`는 한 단계 위의

```
day01_image_aug_vae/
```

가 됩니다.

그래서 모든 데이터 경로는 이 폴더를 기준으로 찾습니다.

```
day01_image_aug_vae/
├─ data/
├─ models/
├─ results/
├─ reports/
├─ scripts/
└─ src/
```

---

## `list_images()`는 무엇을 하나요?

다음 함수는 지정된 폴더 안에서 **이미지 파일만 찾아오는 역할**을 합니다.

```python
def list_images(directory: Path) -> list[Path]:
```

사용 가능한 이미지 확장자는 위에서 정해 놓았습니다.

```python
VALID_EXTENSIONS = {
    ".jpg",
    ".jpeg",
    ".png",
    ".bmp",
}
```

즉 폴더 안에 다른 파일이 있어도 이미지 파일만 골라서 가져옵니다.

```
폴더
├─ apple01.jpg   → 사용
├─ apple02.png   → 사용
├─ memo.txt      → 제외
└─ README.md     → 제외
```

---

## 이 공통 파일이 이후 어디에 사용되나요?

오늘 작성하는 여러 스크립트가 이 파일을 가져와 사용합니다.

```python
from src.day01_common import (
    dataset_paths,
    list_images,
)
```

예를 들어 이후 실습에서:

```
01_check_dataset.py
→ 입력 데이터 경로 확인

02_prepare_split.py
→ Train / Validation / Test 경로 결정

03_augment_preview.py
→ Augmentation 결과 저장 위치 결정

04_augment_train.py
→ 보강 이미지와 Metadata 저장 위치 결정

05_train_vae.py
→ VAE 데이터·모델·결과 경로 결정

06_qa_images.py
→ QA 결과 위치 결정

07_check_leakage.py
→ Leakage 검사 대상 경로 결정
```

처럼 반복해서 사용합니다.

### 핵심만 기억하면 됩니다

```
day01_common.py
→ Apple이나 Banana를 직접 학습하는 코드가 아님

dataset_paths(dataset)
→ 사용할 데이터 이름을 받아
  경로를 결정하는 공통 함수

--dataset apple / banana
→ 뒤의 실행 스크립트에서 입력

args.dataset
→ 공통 함수에 전달

결과
→ 같은 코드를 Apple과 Banana에서 재사용
```

---

# PART 2. Apple Red 1 데이터를 확인하고 공정한 실험 기준을 만듭니다

---

# 6. Apple Red 1 이미지를 입력 폴더에 넣습니다

이제 실제로 사용할 `Apple Red 1` 이미지를 준비합니다.

오늘은 Fruits-360에서 제공하는 데이터를 다음과 같이 사용합니다.

```
Fruits-360

Training / Apple Red 1
→ 오늘의 Training 원본

Test / Apple Red 1
→ 오늘의 Test 원본
```

이 두 데이터를 우리가 만든 프로젝트 폴더에 각각 복사합니다.

현재 프로젝트에서는 다음 위치를 사용합니다. 

```
data/apple/source_train/
→ 원래 Training 이미지

data/apple/source_test/
→ 원래 Test 이미지
```

## 현재 1일차 폴더에 있는지 확인합니다

터미널에서 현재 위치를 확인합니다.

```bash
pwd
```

다음과 비슷하면 됩니다.

```
/home/사용자계정/ai_vision/subject11_synthetic_data/day01_image_aug_vae
```

현재 WSL 폴더를 Windows 탐색기로 엽니다.

```bash
explorer.exe .
```

그러면 현재 작업 중인 `day01_image_aug_vae` 폴더가 Windows 파일 탐색기로 열립니다.

---

## Training 이미지를 넣습니다

Windows 탐색기에서 다음 폴더로 이동합니다.

```
day01_image_aug_vae
└─ data
   └─ apple
      └─ source_train
```

다른 탐색기 창에서 준비해 둔 Fruits-360 데이터의

```
Training
└─ Apple Red 1
```

폴더를 엽니다.

`Apple Red 1` 안에 있는 **이미지 파일 전체를 선택하여** 다음 위치에 복사합니다.

```
data/apple/source_train/
```

즉 결과는 다음과 같아야 합니다.

```
data/apple/source_train/
├─ 0_100.jpg
├─ 1_100.jpg
├─ 2_100.jpg
├─ ...
└─ Apple Red 1 Training 이미지
```

> `source_train/Apple Red 1/이미지`처럼 폴더를 한 단계 더 만들지 않습니다.  
> 이미지 파일들이 `source_train` 바로 아래에 있어야 합니다.

---

## Test 이미지를 넣습니다

이번에는 Fruits-360 데이터의

```
Test
└─ Apple Red 1
```

폴더를 엽니다.

그 안의 이미지 파일 전체를 다음 위치에 복사합니다.

```
data/apple/source_test/
```

결과는 다음과 같아야 합니다.

```
data/apple/source_test/
├─ 0_100.jpg
├─ 1_100.jpg
├─ 2_100.jpg
├─ ...
└─ Apple Red 1 Test 이미지
```

최종 구조는 다음과 같습니다.

```
data/apple/
│
├─ source_train/
│  ├─ 이미지
│  ├─ 이미지
│  └─ ...
│
└─ source_test/
   ├─ 이미지
   ├─ 이미지
   └─ ...
```

Training과 Test를 처음부터 따로 보관하는 이유는 학습에 사용할 이미지와 마지막 평가에 사용할 이미지를 섞지 않기 위해서입니다.

```
원래 Training
        ↓
나중에 Train + Validation으로 분리

원래 Test
        ↓
마지막 평가용으로 그대로 유지
```

> Fruits-360의 기존 Test 이미지는 Training 이미지와 섞지 않습니다.

이제 이미지 복사가 끝났다면 다음 단계의 `01_check_dataset.py`를 실행하여 파일이 실제로 들어갔는지, 정상적으로 읽히는지, 몇 장인지, 이미지 크기가 얼마인지 자동으로 확인합니다.


---

# 7. 입력 이미지가 실습할 준비가 되었는지 확인합니다

앞에서는 `00_check_env.py`를 실행하여 Python, 라이브러리, CUDA, GPU처럼 
**컴퓨터의 실행환경**이 정상인지 확인했습니다.

이제는 컴퓨터가 아니라 오늘 실제로 사용할 **Apple Red 1 이미지 데이터**를 확인합니다.

```
00_check_env.py
→ "내 컴퓨터가 실습할 준비가 되었는가?"

        ↓

01_check_dataset.py
→ "내가 사용할 이미지 데이터가
   실습할 준비가 되었는가?"
```

## 이 코드는 무엇을 하나요?

`01_check_dataset.py`는 본격적인 Augmentation이나 VAE 학습을 시작하기 전에 
**입력 데이터의 기본 상태를 자동으로 점검하는 코드**입니다.

다음 내용을 확인합니다.

```
Apple 데이터 경로를 찾는다
        ↓
Training 이미지 목록을 가져온다
        ↓
Test 이미지 목록을 가져온다
        ↓
이미지가 실제로 존재하는지 확인한다
        ↓
각 이미지를 OpenCV로 열어본다
        ↓
읽을 수 없는 이미지가 있는지 확인한다
        ↓
Training / Test 이미지 수를 확인한다
        ↓
대표 이미지의 Width × Height를 확인한다
        ↓
문제가 없으면
Dataset check: READY
```

쉽게 말하면 이 코드는

> **“Apple 이미지가 제대로 준비되어 있어서 다음 실습으로 넘어가도 되는가?”**

를 확인하는 **사전 점검 코드**입니다.

---

## 왜 이 작업을 먼저 하나요?

이후 실습에서는 현재 이미지를 이용하여 다음 작업을 계속 진행합니다.

```
현재 Apple 원본 데이터
        ↓
Train / Validation / Test 구성
        ↓
Augmentation
        ↓
VAE 학습
        ↓
QA
        ↓
Leakage 확인
```

그런데 시작 데이터에 문제가 있다면 뒤의 코드가 아무리 정확해도 실습은 정상적으로 진행되지 않습니다.

예를 들어:

```
source_train이 비어 있음
→ Train / Validation을 만들 수 없음

source_test가 비어 있음
→ 마지막 평가 데이터를 만들 수 없음

깨진 이미지가 있음
→ Augmentation이나 VAE 실행 중 오류 발생 가능

예상과 다른 이미지 크기
→ 이후 전처리나 모델 입력을 다시 확인해야 함
```

따라서 **모델을 학습하기 전에 입력부터 확인하는 습관**을 만드는 것이 중요합니다.

> 현업에서도 오류가 발생했을 때 바로 모델 코드부터 의심하기보다, 먼저 **입력 데이터가 정상인지 확인하는 것**이 기본적인 점검 순서입니다.

코드를 작성합니다.

**파일: `scripts/01_check_dataset.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import cv2


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    return parser.parse_args()


def check_group(name, files):
    unreadable = []

    print()
    print(f"[{name}]")
    print("Count:", len(files))

    for path in files:
        image = cv2.imread(str(path))

        if image is None:
            unreadable.append(path.name)

    if files:
        first = cv2.imread(str(files[0]))

        if first is not None:
            height, width = first.shape[:2]

            print(
                "First image size:",
                f"{width} x {height}",
            )

    if unreadable:
        print("Unreadable:", len(unreadable))

    return unreadable


def main() -> None:
    args = parse_args()
    paths = dataset_paths(args.dataset)

    train_files = list_images(
        paths["source_train"]
    )

    test_files = list_images(
        paths["source_test"]
    )

    if not train_files:
        raise RuntimeError(
            "source_train 이미지가 없습니다."
        )

    if not test_files:
        raise RuntimeError(
            "source_test 이미지가 없습니다."
        )

    train_bad = check_group(
        "SOURCE TRAIN",
        train_files,
    )

    test_bad = check_group(
        "SOURCE TEST",
        test_files,
    )

    if train_bad or test_bad:
        raise RuntimeError(
            "읽을 수 없는 이미지가 있습니다."
        )

    print()
    print(
        "Total:",
        len(train_files) + len(test_files),
    )

    print("Dataset check: READY")


if __name__ == "__main__":
    main()
```

### 1. 전체 처리 순서를 빠르게 이해하는 의사코드

```
프로그램 실행 옵션에서
apple 또는 banana를 선택한다

        ↓

선택한 데이터셋의
source_train / source_test 경로를 가져온다

        ↓

source_train 폴더에서
이미지 파일 목록을 가져온다

source_test 폴더에서
이미지 파일 목록을 가져온다

        ↓

Training 이미지가 하나도 없으면
오류를 발생시키고 프로그램을 중단한다

Test 이미지가 하나도 없으면
오류를 발생시키고 프로그램을 중단한다

        ↓

Training 이미지를 검사한다

    이미지 수를 출력한다

    모든 이미지를 하나씩 OpenCV로 읽어본다

    읽을 수 없는 이미지가 있으면
    따로 기록한다

    첫 번째 이미지의
    Width × Height를 확인한다

        ↓

Test 이미지도 같은 방법으로 검사한다

        ↓

Training 또는 Test에서
읽을 수 없는 이미지가 발견되면
오류를 발생시키고 중단한다

        ↓

Training 수 + Test 수를 계산하여
전체 이미지 수를 출력한다

        ↓

모든 기본 검사를 통과하면

Dataset check: READY

를 출력한다
```

한줄로 정리하면

```
01_check_dataset.py

→ 데이터 위치 찾기
→ Training / Test 이미지 목록 확인
→ 파일 수 확인
→ 모든 이미지가 실제로 읽히는지 확인
→ 대표 이미지 크기 확인
→ 이상이 없으면 READY
```

### 2. 실제 코드와 1:1로 대응하는 상세 코드 리뷰

```python
# 최신 Python의 타입 힌트를 안정적으로 사용할 수 있도록 합니다.
from __future__ import annotations


# 터미널에서 --dataset apple 같은 옵션을 받기 위해 사용합니다.
import argparse

# Python의 모듈 검색 경로를 수정하기 위해 사용합니다.
import sys

# 파일과 폴더 경로를 객체 형태로 다루기 위해 사용합니다.
from pathlib import Path


# 이미지를 실제로 읽을 수 있는지 검사하기 위해 OpenCV를 불러옵니다.
import cv2


# 현재 실행 중인 01_check_dataset.py의 위치를 기준으로
# 두 단계 위의 day01_image_aug_vae 폴더 경로를 구합니다.
#
# 예:
# day01_image_aug_vae/
# ├─ scripts/
# │  └─ 01_check_dataset.py
#
# ROOT → day01_image_aug_vae/
ROOT = Path(__file__).resolve().parents[1]


# Python이 ROOT 폴더 안의 src 모듈을 찾을 수 있도록
# 프로젝트 ROOT를 모듈 검색 경로의 맨 앞에 추가합니다.
sys.path.insert(0, str(ROOT))


# 앞에서 만든 공통 파일 day01_common.py에서
# 두 개의 공통 함수를 가져옵니다.
#
# dataset_paths()
# → apple 또는 banana의 폴더 경로를 알려줌
#
# list_images()
# → 해당 폴더 안의 이미지 파일 목록을 가져옴
from src.day01_common import (
    dataset_paths,
    list_images,
)


# 터미널에서 입력하는 실행 옵션을 처리하는 함수를 만듭니다.
def parse_args():

    # 명령행 옵션을 관리할 ArgumentParser를 만듭니다.
    parser = argparse.ArgumentParser()


    # --dataset 옵션을 추가합니다.
    #
    # 실행 예:
    # python scripts/01_check_dataset.py --dataset apple
    #
    # default="apple"
    # → 옵션을 쓰지 않으면 apple 사용
    #
    # choices=["apple", "banana"]
    # → apple 또는 banana만 입력 가능
    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )


    # 사용자가 입력한 옵션을 읽어 반환합니다.
    #
    # 예:
    # --dataset apple
    # → args.dataset == "apple"
    return parser.parse_args()


# 한 그룹의 이미지들을 검사하는 함수를 만듭니다.
#
# 예:
# name  → "SOURCE TRAIN"
# files → Training 이미지들의 경로 목록
def check_group(name, files):

    # 읽지 못한 이미지 이름을 저장할 빈 리스트를 만듭니다.
    unreadable = []


    # 출력 화면을 보기 좋게 한 줄 띄웁니다.
    print()

    # 현재 검사 중인 그룹 이름을 출력합니다.
    #
    # 예:
    # [SOURCE TRAIN]
    print(f"[{name}]")

    # 현재 그룹에 이미지가 몇 장 있는지 출력합니다.
    print("Count:", len(files))


    # 이미지 파일을 하나씩 검사합니다.
    for path in files:

        # OpenCV를 이용하여 현재 이미지를 읽습니다.
        image = cv2.imread(str(path))


        # OpenCV가 이미지를 읽지 못하면
        # image 값은 None이 됩니다.
        if image is None:

            # 읽지 못한 이미지의 파일명을
            # unreadable 목록에 기록합니다.
            unreadable.append(path.name)


    # 이미지 목록이 비어 있지 않은 경우에만 실행합니다.
    if files:

        # 첫 번째 이미지를 다시 읽습니다.
        #
        # 대표 이미지의 크기를 확인하기 위한 코드입니다.
        first = cv2.imread(str(files[0]))


        # 첫 번째 이미지가 정상적으로 읽혔는지 확인합니다.
        if first is not None:

            # OpenCV 이미지 배열의 앞 두 값을 가져옵니다.
            #
            # first.shape
            # → (height, width, channel)
            #
            # first.shape[:2]
            # → (height, width)
            height, width = first.shape[:2]


            # 첫 번째 대표 이미지의 크기를 출력합니다.
            #
            # 예:
            # First image size: 100 x 100
            print(
                "First image size:",
                f"{width} x {height}",
            )


    # 읽지 못한 이미지가 하나 이상 있다면 실행합니다.
    if unreadable:

        # 읽지 못한 이미지의 개수를 출력합니다.
        print("Unreadable:", len(unreadable))


    # 읽지 못한 이미지 목록을
    # 호출한 곳으로 반환합니다.
    return unreadable


# 프로그램의 실제 실행 흐름을 담당하는 main 함수를 만듭니다.
def main() -> None:

    # 사용자가 입력한 --dataset 옵션을 읽습니다.
    args = parse_args()


    # 선택한 데이터셋의 여러 폴더 경로를 가져옵니다.
    #
    # 예:
    # args.dataset == "apple"
    #
    # data/apple/source_train
    # data/apple/source_test
    # ...
    paths = dataset_paths(args.dataset)


    # Apple 또는 Banana의
    # source_train 폴더 안에 있는 이미지 목록을 가져옵니다.
    train_files = list_images(
        paths["source_train"]
    )


    # source_test 폴더 안에 있는 이미지 목록을 가져옵니다.
    test_files = list_images(
        paths["source_test"]
    )


    # Training 이미지 목록이 비어 있는지 확인합니다.
    if not train_files:

        # 이미지가 없다면 이후 실습을 진행할 수 없으므로
        # 오류를 발생시키고 프로그램을 중단합니다.
        raise RuntimeError(
            "source_train 이미지가 없습니다."
        )


    # Test 이미지 목록이 비어 있는지 확인합니다.
    if not test_files:

        # Test 이미지가 없다면
        # 오류를 발생시키고 프로그램을 중단합니다.
        raise RuntimeError(
            "source_test 이미지가 없습니다."
        )


    # Training 이미지 전체를 검사합니다.
    #
    # 결과로 읽지 못한 이미지 목록을
    # train_bad에 저장합니다.
    train_bad = check_group(
        "SOURCE TRAIN",
        train_files,
    )


    # Test 이미지 전체를 같은 방법으로 검사합니다.
    #
    # 읽지 못한 이미지 목록을
    # test_bad에 저장합니다.
    test_bad = check_group(
        "SOURCE TEST",
        test_files,
    )


    # Training 또는 Test 중 하나라도
    # 읽지 못한 이미지가 있는지 확인합니다.
    if train_bad or test_bad:

        # 문제가 있다면 이후 작업을 중단합니다.
        raise RuntimeError(
            "읽을 수 없는 이미지가 있습니다."
        )


    # 결과를 보기 좋게 한 줄 띄웁니다.
    print()


    # Training 수와 Test 수를 더하여
    # 전체 이미지 수를 출력합니다.
    print(
        "Total:",
        len(train_files) + len(test_files),
    )


    # 모든 기본 검사가 정상적으로 끝났음을 출력합니다.
    print("Dataset check: READY")


# 이 Python 파일을 직접 실행했을 때만
# 아래의 main()을 실행합니다.
if __name__ == "__main__":

    # 프로그램의 실제 실행을 시작합니다.
    main()
```

이 코드를 리뷰할 때는 전체를 모두 외우기보다는 다음 **5개 핵심 지점**을 중심으로 보면 됩니다.

```txt
1. parse_args()
→ 어떤 데이터셋을 검사할지 결정
→ apple / banana


2. dataset_paths(args.dataset)
→ 선택한 데이터의 폴더 위치를 가져옴


3. list_images(...)
→ source_train / source_test의 이미지 목록을 가져옴


4. check_group(...)
→ 이미지 수 확인
→ 모든 이미지를 cv2.imread()로 읽어봄
→ 읽지 못한 이미지 검사
→ 대표 이미지 크기 확인


5. RuntimeError + READY
→ 문제가 있으면 즉시 중단
→ 모두 통과하면 Dataset check: READY
```

이 코드에서 가장 중요하게 이해해야 하는 흐름은 이것입니다.

```
경로를 찾는다
        ↓
파일 목록을 가져온다
        ↓
데이터가 있는지 검사한다
        ↓
실제로 읽어본다
        ↓
문제가 있으면 중단한다
        ↓
문제가 없으면 다음 단계로 간다
```

실행합니다.

```bash
python scripts/01_check_dataset.py \
  --dataset apple
```

실제 결과:

```text
[SOURCE TRAIN]
Count: 492

[SOURCE TEST]
Count: 164

Total: 656

Dataset check: READY
```

결과 해석

```
SOURCE TRAIN
→ 492장의 Training 이미지가 정상적으로 확인됨
→ 대표 이미지 크기: 100 × 100

SOURCE TEST
→ 164장의 Test 이미지가 정상적으로 확인됨
→ 대표 이미지 크기: 100 × 100

Total
→ 전체 이미지 수: 656장
```

---

# 8. 데이터 확인이 끝났으므로 실험에 사용할 데이터를 나눌 준비를 합니다

7번에서는 `01_check_dataset.py`를 실행하여 현재 Apple 데이터가 실습에 사용할 수 있는 기본 상태인지 확인했습니다.

지금까지는 다음 질문에 답했습니다.

```
"Apple 이미지가 제대로 준비되어 있는가?"
```

이제부터는 한 단계 더 나아가 다음 질문을 해결해야 합니다.

```
"이 이미지들을 실제 AI 실험에서
어떤 역할로 사용할 것인가?"
```

앞으로 진행할 Augmentation과 VAE 실습에서는 모든 이미지를 한꺼번에 사용하지 않습니다.

모델을 학습하는 데이터와 학습 중간에 확인하는 데이터, 마지막 성능을 확인하는 데이터를 서로 구분해야 합니다.

그래서 다음 단계에서는 데이터를 **Train / Validation / Test** 세 역할로 나눕니다.

---
## Train·Validation·Test의 역할을 먼저 이해합니다

현재 Apple 원본 데이터는 다음처럼 준비되어 있습니다.

```
data/apple/source_train/
→ Fruits-360의 원래 Training 이미지

data/apple/source_test/
→ Fruits-360의 원래 Test 이미지
```

오늘은 Fruits-360의 원래 `Training` 데이터만 다시 나눕니다.

```
원래 Training
492장
        ↓
┌─────────────────┴─────────────────┐
│                                   │
↓                                   ↓
Train                           Validation
약 85%                          약 15%
│                                   │
모델이 실제로 학습             학습 상태와 조건 확인
```

Fruits-360에서 이미 따로 제공된 `Test` 데이터는 다시 나누지 않습니다.

```
원래 Test
164장
        ↓
그대로 Test로 유지
        ↓
모든 조건을 정한 뒤
마지막 성능 확인
```

쉽게 구분하면 다음과 같습니다.

```
Train
→ 모델이 공부하는 문제집

Validation
→ 공부가 잘 되고 있는지
  중간에 확인하는 모의시험

Test
→ 모든 공부와 조건 설정이 끝난 뒤
  마지막으로 보는 최종시험
```

따라서 앞으로의 실험에서는 각 데이터의 역할을 섞지 않는 것이 중요합니다.

```
Train
→ Augmentation 적용
→ VAE 학습

Validation
→ 조건 비교
→ 학습 상태 확인

Test
→ 마지막 결과 확인
```

---

# 9. 데이터를 나누기 전에 Data Leakage를 이해합니다

왜 굳이 데이터를 먼저 나누어야 할까요?

앞으로 우리는 Train 이미지를 회전하거나 밝기를 바꾸는 등 Augmentation을 수행합니다.

그런데 **데이터를 먼저 나누지 않고 Augmentation부터 수행하면 평가 결과가 잘못될 수 있습니다.**

예를 들어 다음과 같은 상황을 생각해 봅니다.

```
원본 apple_A
        ↓
Train에 들어감


같은 apple_A를 회전한 이미지
        ↓
Test에 들어감
```

두 이미지는 파일은 다르지만 사실상 **같은 원본에서 만들어진 매우 비슷한 이미지**입니다.

```
Train
apple_A 원본

Test
apple_A 회전 이미지

        ↓

모델 입장에서는
완전히 처음 보는 이미지가 아님
```

그런데 Test 결과만 보면 모델이 처음 보는 데이터를 잘 맞힌 것처럼 보일 수 있습니다.

이처럼 **학습에 사용된 정보가 Validation이나 Test에 섞여 평가 결과가 실제보다 좋아 보이는 문제**를 `Data Leakage`라고 합니다.

그래서 오늘은 반드시 다음 순서를 지킵니다.

```
원본 데이터 준비
        ↓
Train / Validation / Test 먼저 분리
        ↓
각 역할 고정
        ↓
Train에만 Augmentation 적용
        ↓
Validation / Test는 원본 그대로 유지
```

> **먼저 데이터를 나누고, 그다음 Train만 보강합니다.**

이 원칙은 1일차뿐 아니라 이후 교과 11 전체 실험에서도 계속 사용하는 중요한 기준입니다.

---

# 10. 이제 실제로 Train·Validation·Test를 만드는 코드를 작성합니다

8번에서는 **왜 데이터를 Train / Validation / Test로 구분해야 하는지** 배웠고,  
9번에서는 **왜 Augmentation보다 데이터 분리를 먼저 해야 하는지** 이해했습니다.

이제 그 원칙을 실제 코드로 구현합니다.

이번에 작성할 파일은 다음입니다.

```
scripts/02_prepare_split.py
```

이 코드는 현재 원본 데이터를 다음 구조로 바꾸어 줍니다.

```
현재 상태

source_train
492장

source_test
164장


        ↓
02_prepare_split.py 실행


실험용 데이터

source_train 492장
        ↓
Train 약 85%
Validation 약 15%

source_test 164장
        ↓
Test로 그대로 유지
```

즉 이 코드는 **모델을 학습하는 코드가 아닙니다.**

앞으로 Augmentation과 VAE 실험을 공정하게 진행할 수 있도록 **실험에 사용할 데이터의 역할을 먼저 확정하는 코드**입니다.

또한 같은 실험을 다시 실행했을 때 데이터가 매번 전혀 다르게 나뉘지 않도록 `Seed`를 고정합니다.

```
Seed 고정
        ↓
같은 조건으로 다시 실행
        ↓
비슷한 Split 재현 가능
```

그리고 어떤 이미지가 Train, Validation, Test 중 어디에 들어갔는지도 기록합니다.

```
원본 이미지
        ↓
어느 Split에 들어갔는가?
        ↓
CSV로 기록
```

따라서 `02_prepare_split.py`의 목적은 다음 한 문장으로 정리할 수 있습니다.

> Apple 데이터를 학습용·중간 확인용·최종 평가용으로 먼저 분리하여, 
> 이후 Augmentation과 VAE 실험을 공정하게 진행할 기준을 만드는 코드입니다.

이제 실제 코드를 작성합니다.

**파일: `scripts/02_prepare_split.py`**

```python
from __future__ import annotations

import argparse
import csv
import random
import shutil
import sys
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


SEED = 42
VAL_RATIO = 0.15


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--seed",
        type=int,
        default=SEED,
    )

    parser.add_argument(
        "--val-ratio",
        type=float,
        default=VAL_RATIO,
    )

    return parser.parse_args()


def reset_directory(directory: Path) -> None:
    if directory.exists():
        shutil.rmtree(directory)

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )


def copy_files(
    files,
    target_dir,
    split_name,
    rows,
):
    for source_path in files:
        target_path = (
            target_dir
            / source_path.name
        )

        shutil.copy2(
            source_path,
            target_path,
        )

        rows.append(
            {
                "file": source_path.name,
                "source_group": (
                    source_path.parent.name
                ),
                "split": split_name,
            }
        )


def main() -> None:
    args = parse_args()

    if not 0 < args.val_ratio < 1:
        raise ValueError(
            "--val-ratio는 0보다 크고 1보다 작아야 합니다."
        )

    paths = dataset_paths(
        args.dataset
    )

    source_train = list_images(
        paths["source_train"]
    )

    source_test = list_images(
        paths["source_test"]
    )

    if len(source_train) < 10:
        raise RuntimeError(
            "source_train 이미지가 너무 적습니다."
        )

    if not source_test:
        raise RuntimeError(
            "source_test 이미지가 없습니다."
        )

    rng = random.Random(
        args.seed
    )

    shuffled = source_train.copy()
    rng.shuffle(shuffled)

    val_count = int(
        len(shuffled)
        * args.val_ratio
    )

    val_count = max(
        1,
        min(
            val_count,
            len(shuffled) - 1,
        ),
    )

    val_files = shuffled[:val_count]
    train_files = shuffled[val_count:]
    test_files = source_test

    reset_directory(
        paths["split_train"]
    )

    reset_directory(
        paths["split_val"]
    )

    reset_directory(
        paths["split_test"]
    )

    rows = []

    copy_files(
        train_files,
        paths["split_train"],
        "train",
        rows,
    )

    copy_files(
        val_files,
        paths["split_val"],
        "val",
        rows,
    )

    copy_files(
        test_files,
        paths["split_test"],
        "test",
        rows,
    )

    report_path = (
        paths["report"]
        / "split_manifest.csv"
    )

    with report_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=[
                "file",
                "source_group",
                "split",
            ],
        )

        writer.writeheader()
        writer.writerows(rows)

    print("=" * 60)
    print("Dataset Split Completed")
    print("=" * 60)

    print("Dataset :", args.dataset)
    print("Train   :", len(train_files))
    print("Val     :", len(val_files))
    print("Test    :", len(test_files))
    print("Seed    :", args.seed)

    print()
    print("Manifest:", report_path)


if __name__ == "__main__":
    main()
```

### 1. 전체 처리 흐름을 빠르게 이해하는 의사코드

```text
실행할 데이터셋을 선택한다
→ apple 또는 banana

Validation 비율을 확인한다
→ 기본값 15%

Seed를 확인한다
→ 기본값 42

        ↓

선택한 데이터셋의 경로를 가져온다

        ↓

source_train 이미지 목록을 가져온다
source_test 이미지 목록을 가져온다

        ↓

Training 이미지가 너무 적으면 중단한다

Test 이미지가 없으면 중단한다

        ↓

source_train 이미지 목록을 복사한다

Seed를 사용하여
Training 이미지 순서를 랜덤하게 섞는다

        ↓

전체 Training의 약 15%를
Validation으로 선택한다

나머지 약 85%를
Train으로 선택한다

기존 source_test는
그대로 Test로 사용한다

        ↓

기존 split/train
기존 split/val
기존 split/test 폴더를 초기화한다

        ↓

Train 이미지를 split/train에 복사한다

Validation 이미지를 split/val에 복사한다

Test 이미지를 split/test에 복사한다

        ↓

각 이미지가
Train / Validation / Test 중
어디에 들어갔는지 기록한다

        ↓

split_manifest.csv로 저장한다

        ↓

Train / Val / Test 수량과
사용한 Seed를 출력한다
```

핵심만 줄이면 다음과 같습니다.

```
source_train
        ↓
Seed 42로 섞기
        ↓
약 85% → Train
약 15% → Validation

source_test
        ↓
그대로 Test 유지

        ↓

각 이미지의 Split 정보를
split_manifest.csv에 기록
```

### 2. 실제 코드와 1:1로 대응하는 상세 코드 리뷰

```python
# 최신 Python의 타입 힌트를 안정적으로 사용할 수 있게 합니다.
from __future__ import annotations


# 터미널에서 --dataset, --seed 같은 옵션을 받기 위해 사용합니다.
import argparse

# Split 결과를 CSV 파일로 기록하기 위해 사용합니다.
import csv

# Training 이미지의 순서를 랜덤하게 섞기 위해 사용합니다.
import random

# 파일 복사와 기존 폴더 삭제를 위해 사용합니다.
import shutil

# Python 모듈 검색 경로를 수정하기 위해 사용합니다.
import sys

# 파일과 폴더 경로를 편리하게 다루기 위해 사용합니다.
from pathlib import Path


# 현재 scripts/02_prepare_split.py 파일을 기준으로
# 두 단계 위에 있는 day01_image_aug_vae 폴더를
# 프로젝트 ROOT 경로로 지정합니다.
ROOT = Path(__file__).resolve().parents[1]


# ROOT 폴더를 Python 모듈 검색 경로에 추가합니다.
# 이를 통해 src.day01_common을 불러올 수 있습니다.
sys.path.insert(0, str(ROOT))


# 앞에서 만든 공통 파일에서
# 데이터 경로를 만드는 함수와
# 이미지 목록을 가져오는 함수를 불러옵니다.
from src.day01_common import (
    dataset_paths,
    list_images,
)


# 랜덤한 데이터 분할을 재현하기 위한 Seed입니다.
#
# 같은 데이터 + 같은 Seed
# → 같은 순서로 섞이므로
# → 같은 Split을 다시 만들 수 있습니다.
SEED = 42


# 원래 Training 데이터 중
# Validation으로 사용할 기본 비율입니다.
#
# 0.15 → 15%
VAL_RATIO = 0.15


# 터미널에서 입력한 실행 옵션을 읽는 함수를 만듭니다.
def parse_args():

    # 명령행 옵션을 관리할 Parser를 만듭니다.
    parser = argparse.ArgumentParser()


    # 사용할 데이터셋을 선택합니다.
    #
    # 기본값 → apple
    #
    # 허용 값
    # apple 또는 banana
    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )


    # 데이터 분할에 사용할 Seed를 입력받습니다.
    #
    # 입력하지 않으면 SEED = 42를 사용합니다.
    parser.add_argument(
        "--seed",
        type=int,
        default=SEED,
    )


    # Validation 비율을 입력받습니다.
    #
    # 입력하지 않으면 0.15
    # 즉 15%를 사용합니다.
    parser.add_argument(
        "--val-ratio",
        type=float,
        default=VAL_RATIO,
    )


    # 사용자가 입력한 옵션을 읽어 반환합니다.
    return parser.parse_args()


# Split 결과를 저장할 폴더를
# 깨끗한 상태로 다시 만드는 함수입니다.
def reset_directory(directory: Path) -> None:

    # 해당 폴더가 이미 존재한다면
    if directory.exists():

        # 이전 Split 결과를 폴더째 삭제합니다.
        shutil.rmtree(directory)


    # 빈 폴더를 다시 만듭니다.
    directory.mkdir(

        # 상위 폴더가 없어도 함께 생성합니다.
        parents=True,

        # 이미 폴더가 있어도 오류를 발생시키지 않습니다.
        exist_ok=True,
    )


# 선택된 이미지들을 원하는 Split 폴더로 복사하고
# Split 정보를 기록하는 함수입니다.
def copy_files(
    files,
    target_dir,
    split_name,
    rows,
):

    # 전달받은 이미지들을 하나씩 처리합니다.
    for source_path in files:

        # 복사될 목적지 경로를 만듭니다.
        #
        # 예:
        # split/train/파일이름.jpg
        target_path = (
            target_dir
            / source_path.name
        )


        # 원본 이미지를 목적지 폴더로 복사합니다.
        #
        # copy2는 파일 내용뿐 아니라
        # 가능한 경우 파일의 시간 정보 등도 함께 보존합니다.
        shutil.copy2(
            source_path,
            target_path,
        )


        # 현재 이미지가 어느 Split에 들어갔는지
        # 기록용 Dictionary를 rows에 추가합니다.
        rows.append(
            {

                # 이미지 파일 이름
                "file": source_path.name,

                # 원래 이미지가 어느 폴더에서 왔는지 기록합니다.
                "source_group": (
                    source_path.parent.name
                ),

                # train / val / test 중
                # 어디에 배정되었는지 기록합니다.
                "split": split_name,
            }
        )


# 프로그램의 전체 실행 흐름입니다.
def main() -> None:

    # 터미널에서 입력한 옵션을 읽습니다.
    args = parse_args()


    # Validation 비율이 올바른 값인지 확인합니다.
    #
    # 0보다 크고 1보다 작아야 합니다.
    #
    # 예:
    # 0.15 → 정상
    # 1.5  → 비정상
    # -0.2 → 비정상
    if not 0 < args.val_ratio < 1:

        # 잘못된 비율이면 실행을 중단합니다.
        raise ValueError(
            "--val-ratio는 0보다 크고 1보다 작아야 합니다."
        )


    # apple 또는 banana에 해당하는
    # 여러 데이터 경로를 가져옵니다.
    paths = dataset_paths(
        args.dataset
    )


    # source_train 폴더에서
    # 원래 Training 이미지 목록을 가져옵니다.
    source_train = list_images(
        paths["source_train"]
    )


    # source_test 폴더에서
    # 원래 Test 이미지 목록을 가져옵니다.
    source_test = list_images(
        paths["source_test"]
    )


    # Training 이미지가 10장보다 적은지 확인합니다.
    if len(source_train) < 10:

        # 너무 적으면 정상적인 분할 실습이 어렵기 때문에
        # 프로그램을 중단합니다.
        raise RuntimeError(
            "source_train 이미지가 너무 적습니다."
        )


    # Test 이미지가 하나도 없는지 확인합니다.
    if not source_test:

        # Test가 없으면 최종 평가용 데이터를 만들 수 없으므로
        # 프로그램을 중단합니다.
        raise RuntimeError(
            "source_test 이미지가 없습니다."
        )


    # 사용자가 지정한 Seed로
    # 독립적인 랜덤 객체를 만듭니다.
    #
    # 기본값:
    # Seed = 42
    rng = random.Random(
        args.seed
    )


    # 원본 source_train 목록은 그대로 보존하기 위해
    # 새로운 리스트를 복사해서 만듭니다.
    shuffled = source_train.copy()


    # 복사한 Training 이미지 목록의 순서를
    # 랜덤하게 섞습니다.
    #
    # 같은 Seed를 사용하면
    # 다시 실행해도 같은 순서로 섞입니다.
    rng.shuffle(shuffled)


    # Validation으로 사용할 이미지 수를 계산합니다.
    #
    # 예:
    # Training 492장 × 0.15
    # → 73.8
    # int()
    # → 73장
    val_count = int(
        len(shuffled)
        * args.val_ratio
    )


    # Validation 수가 너무 작거나 너무 커지지 않도록
    # 안전하게 보정합니다.
    val_count = max(

        # Validation을 최소 1장은 확보합니다.
        1,

        min(

            # 계산된 Validation 수와
            val_count,

            # 전체 Training보다 1장 적은 값 중
            # 더 작은 값을 선택합니다.
            #
            # 즉 Train에도 최소 1장은 남깁니다.
            len(shuffled) - 1,
        ),
    )


    # 섞인 이미지의 앞부분을
    # Validation으로 사용합니다.
    #
    # 예:
    # 앞 73장 → Validation
    val_files = shuffled[:val_count]


    # 나머지 이미지들을
    # 실제 Train으로 사용합니다.
    train_files = shuffled[val_count:]


    # 기존 Fruits-360 Test 데이터는
    # 다시 나누지 않고 그대로 Test로 사용합니다.
    test_files = source_test


    # 기존 split/train 폴더를 초기화합니다.
    reset_directory(
        paths["split_train"]
    )


    # 기존 split/val 폴더를 초기화합니다.
    reset_directory(
        paths["split_val"]
    )


    # 기존 split/test 폴더를 초기화합니다.
    reset_directory(
        paths["split_test"]
    )


    # 각 이미지가 어느 Split에 배정되었는지
    # 기록할 빈 리스트를 만듭니다.
    rows = []


    # Train으로 선택된 이미지들을
    # split/train 폴더에 복사합니다.
    #
    # 동시에 rows에
    # split="train" 정보를 기록합니다.
    copy_files(
        train_files,
        paths["split_train"],
        "train",
        rows,
    )


    # Validation으로 선택된 이미지들을
    # split/val 폴더에 복사합니다.
    #
    # 동시에 split="val" 정보를 기록합니다.
    copy_files(
        val_files,
        paths["split_val"],
        "val",
        rows,
    )


    # 기존 Test 이미지를
    # split/test 폴더에 복사합니다.
    #
    # 동시에 split="test" 정보를 기록합니다.
    copy_files(
        test_files,
        paths["split_test"],
        "test",
        rows,
    )


    # Split 기록을 저장할 CSV 파일 경로를 만듭니다.
    #
    # 예:
    # reports/apple/split_manifest.csv
    report_path = (
        paths["report"]
        / "split_manifest.csv"
    )


    # CSV 파일을 쓰기 모드로 엽니다.
    with report_path.open(

        # 기존 파일이 있으면 새로 작성합니다.
        "w",

        # CSV 작성 시 불필요한 빈 줄을 방지합니다.
        newline="",

        # Excel에서도 한글이 잘 보이도록
        # UTF-8 BOM 형식으로 저장합니다.
        encoding="utf-8-sig",
    ) as file:


        # Dictionary 형태의 rows 데이터를
        # CSV로 저장할 Writer를 만듭니다.
        writer = csv.DictWriter(
            file,

            # CSV의 열 이름을 지정합니다.
            fieldnames=[
                "file",
                "source_group",
                "split",
            ],
        )


        # CSV 첫 번째 줄에
        # 컬럼 이름을 작성합니다.
        writer.writeheader()


        # 지금까지 rows에 기록된
        # 모든 이미지의 Split 정보를 작성합니다.
        writer.writerows(rows)


    # 결과 구분선을 출력합니다.
    print("=" * 60)


    # 데이터 분할이 완료되었음을 출력합니다.
    print("Dataset Split Completed")


    # 결과 구분선을 출력합니다.
    print("=" * 60)


    # 어떤 데이터셋을 분할했는지 출력합니다.
    print("Dataset :", args.dataset)


    # 최종 Train 이미지 수를 출력합니다.
    print("Train   :", len(train_files))


    # 최종 Validation 이미지 수를 출력합니다.
    print("Val     :", len(val_files))


    # 최종 Test 이미지 수를 출력합니다.
    print("Test    :", len(test_files))


    # 데이터 분할에 사용한 Seed를 출력합니다.
    print("Seed    :", args.seed)


    # 한 줄 띄웁니다.
    print()


    # Split 정보가 저장된 CSV 위치를 출력합니다.
    print("Manifest:", report_path)


# 이 Python 파일을 직접 실행했을 때만
# main()을 실행합니다.
if __name__ == "__main__":

    # 실제 데이터 분할을 시작합니다.
    main()
```

## 코드 리뷰할 때 핵심만 다시 보면

이 코드에서는 특히 다음 부분을 이해하면 전체 흐름을 잡을 수 있습니다.

```
① source_train / source_test 읽기

source_train = list_images(...)
source_test = list_images(...)

        ↓

원본 데이터 가져오기
```

```
② Seed를 이용하여 Training 순서 섞기

rng = random.Random(args.seed)
shuffled = source_train.copy()
rng.shuffle(shuffled)

        ↓

같은 Seed로
분할 결과를 재현할 수 있게 함
```

```
③ Validation 개수 계산

val_count
= 전체 Training 수 × 0.15
```

```
④ 실제 데이터 분리

val_files = shuffled[:val_count]
train_files = shuffled[val_count:]
test_files = source_test

        ↓

Train
Validation
Test 확정
```

```
⑤ Split 폴더에 실제 이미지 복사

copy_files(...)
```

```
⑥ 어떤 파일이 어디에 들어갔는지 기록

split_manifest.csv

file
source_group
split
```

따라서 이 코드의 가장 중요한 의미는 다음과 같습니다.

> **원래 Training 데이터를 무작위로 섞어 Train과 Validation으로 나누고, 
> 기존 Test는 그대로 보호하면서, 
> 어떤 이미지가 어디에 배정되었는지까지 기록하는 코드입니다.**


실행합니다.

```bash
python scripts/02_prepare_split.py \
  --dataset apple
```

실제 실행 결과는 다음과 같습니다.

```text
============================================================
Dataset Split Completed
============================================================
Dataset : apple
Train   : 419
Val     : 73
Test    : 164
Seed    : 42

Manifest: .../reports/apple/split_manifest.csv
```

### 실행 결과를 간단히 해석합니다

원래 Training 492장이 다음처럼 나뉘었습니다.

```
Training 492장
        ↓
Train       419장
Validation   73장

Test 164장
        ↓
Test 164장 그대로 유지
```

`419 + 73 = 492`이므로 기존 Training 이미지가 빠지지 않고 Train과 Validation으로 분리되었습니다.

Test도 기존 164장이 그대로 유지되었으므로 **평가용 Test를 Training에 섞지 않고 보호한 상태**입니다.

---

## 이 코드는 무엇을 입력받고 무엇을 만들었나요?

앞에서 코드를 보면 `02_prepare_split.py`는 다음 작업을 한 번에 처리했습니다.

```
[입력]

--dataset apple
        +
data/apple/source_train/
492장
        +
data/apple/source_test/
164장

        ↓

[처리]

source_train 이미지 목록을 가져온다
        ↓
Seed 42를 사용해 순서를 섞는다
        ↓
약 85% → Train
약 15% → Validation
        ↓
기존 source_test는 그대로 Test로 유지한다
        ↓
각 이미지를 해당 Split 폴더에 복사한다
        ↓
각 이미지가 어디에 배치되었는지 기록한다

        ↓

[출력]

data/apple/split/train/
→ 419장

data/apple/split/val/
→ 73장

data/apple/split/test/
→ 164장

reports/apple/split_manifest.csv
→ 각 이미지의 Split 결과 기록
```

---

## Manifest를 확인합니다

우리는 교과 8 프로젝트에서 manifest를 작성한 적이 있습니다.
`Manifest`는 쉽게 말하면 **데이터 목록과 상태를 기록해 둔 명세표 또는 기록표**입니다.

이번 실습에서는 다음처럼 이해하면 됩니다.

```
실제 이미지
→ split/train
→ split/val
→ split/test

기록표
→ split_manifest.csv
```

확인합니다

```bash
head reports/apple/split_manifest.csv
```

실제 결과:

```
file,source_group,split
266_100.jpg,source_train,train
264_100.jpg,source_train,train
r_222_100.jpg,source_train,train
288_100.jpg,source_train,train
269_100.jpg,source_train,train
...
```

각 열은 다음 의미입니다.

|항목|의미|
|---|---|
|`file`|이미지 파일명|
|`source_group`|원래 어느 데이터에서 왔는지|
|`split`|최종적으로 Train / Val / Test 중 어디에 들어갔는지|

예를 들어

```
266_100.jpg,source_train,train
```

은

```
266_100.jpg
→ 원래 source_train 이미지
→ 이번 Split에서 train으로 배치
```

되었다는 뜻입니다.

`head`는 CSV의 앞부분만 보여주기 때문에 현재 `train`만 보이는 것은 정상입니다. Validation과 Test 기록도 뒤쪽에 있습니다

---

# 11. 오늘 실험에서 바꾸지 않을 기준을 정합니다

앞에서 Apple 데이터를 **Train / Validation / Test**로 나누는 작업을 마쳤습니다.

```
Train      : 419장
Validation : 73장
Test       : 164장
Seed       : 42
```

이제부터는 이 데이터 구성을 다시 바꾸지 않고 그대로 사용합니다.

왜냐하면 앞으로는 **데이터 구성을 바꾸는 실험이 아니라, Augmentation이나 VAE의 조건을 바꾸면서 결과를 비교**할 것이기 때문입니다.

```
고정하는 것

Train
Validation
Test
Seed

        ↓

바꾸어 보는 것

Augmentation 조건
VAE 조건

        ↓

결과 비교
```

예를 들어 Rotation 값을 바꾸거나 VAE 설정을 바꾸었을 때 결과가 달라졌다면, 
데이터 구성이 같기 때문에 **조건을 바꾼 영향이 무엇인지 비교하기 쉬워집니다.**

반대로 실험할 때마다 Train이나 Validation 데이터까지 달라진다면,

```
성능 차이가

데이터가 달라서 생긴 것인지

또는

실험 조건이 달라서 생긴 것인지
```

구분하기 어려워집니다.

따라서 오늘은 다음 상태를 **실험의 기준 상태**로 사용합니다.

```
Apple Red 1
        +
고정된 Train / Validation / Test
        +
Seed 42
```

이처럼 앞으로의 결과를 비교할 때 출발점이 되는 기준을 오늘 실습에서는 **Baseline**이라고 이해하면 됩니다.

```
Baseline
→ 비교를 시작하기 위한 기준 상태
```

이제 이 기준은 그대로 유지한 채, 다음 실습부터 **Train 데이터에만 Augmentation을 적용하고 VAE 실험을 진행**합니다.

---

# PART 3. Apple 이미지 한 장으로 Augmentation을 이해합니다

앞에서는 Apple 데이터를 Train·Validation·Test로 나누고 오늘 사용할 데이터 기준을 확정했습니다.

이제부터는 그중 **Train 이미지에 실제 변형을 적용해 보는 실습**을 시작합니다.

하지만 처음부터 Train 419장 전체를 한꺼번에 변형하지 않습니다.

먼저 이미지 한 장만 사용하여

```text
원본 이미지
        ↓
Brightness
Rotation
Blur
Noise
        ↓
변형 결과 확인
        ↓
이 정도 변형이 실제 데이터로 사용해도 괜찮은지 판단
```

하는 과정을 먼저 진행합니다.

즉 PART 3의 목적은

> Augmentation을 많이 만드는 것이 아니라, 어떤 변형을 적용했을 때 원래 사과의 의미가 유지되는지 직접 눈으로 확인하는 것입니다.

이 Preview가 괜찮다고 판단되면 뒤에서 Train 전체에 같은 종류의 Augmentation을 적용합니다.

---

# 12. Augmentation이 무엇인지 이해합니다

`Augmentation`은 기존 이미지를 조금씩 변형하여 학습 데이터의 조건을 다양하게 만드는 방법입니다.

예를 들어 실제 촬영 환경에서는 항상 똑같은 사진만 얻어지지 않습니다.

```
조명이 달라질 수 있음
→ Brightness

카메라나 객체의 각도가 달라질 수 있음
→ Rotation

초점이나 움직임 때문에 흐려질 수 있음
→ Blur

센서나 촬영환경에서 잡음이 생길 수 있음
→ Noise
```

오늘은 다음 네 가지 변형을 사용합니다.

|방법|쉽게 말하면|
|---|---|
|`Brightness`|이미지를 조금 밝거나 어둡게 만듦|
|`Rotation`|이미지를 조금 회전시킴|
|`Blur`|이미지를 조금 흐리게 만듦|
|`Noise`|픽셀에 작은 잡음을 추가함|

중요한 것은 강하게 변형하는 것이 목적이 아니라는 점입니다.

```
강한 변형
≠
좋은 Augmentation

실제 촬영에서 일어날 수 있는 변형
=
좋은 Augmentation 후보
```

예를 들어 사과 이미지를 너무 어둡게 만들어 형태가 보이지 않거나, 너무 많이 회전시켜 사과가 잘려 버리면 학습 데이터로 사용하기 어려울 수 있습니다.

그래서 먼저 한 장에서 결과를 확인합니다.

---

# 13. 이미지 한 장으로 Augmentation Preview를 만드는 코드를 작성합니다

이제 실제 Train 이미지 한 장을 가져와 네 가지 변형을 적용해 봅니다.

이번에 작성할 `03_augment_preview.py`는 대량의 데이터를 만드는 코드가 아니라, Augmentation 결과를 미리 시험해 보는 코드입니다.

이 코드는 다음 과정을 수행합니다.

```
Train 이미지 한 장 선택
        ↓
원본 그대로 확인
        ↓
Brightness 적용
Rotation 적용
Blur 적용
Noise 적용
        ↓
각 결과 저장
        ↓
한 화면에서 비교할 수 있는 Grid 저장
```

왜 먼저 한 장만 확인할까요?

만약 변형 조건이 너무 강하거나 부자연스러운데 바로 수백 장을 생성하면, 잘못된 데이터도 
함께 많이 만들어지기 때문입니다.

따라서 순서는 다음과 같습니다.

```
한 장으로 Preview
        ↓
눈으로 확인
        ↓
변형이 자연스러운가?
        ↓
괜찮다면
        ↓
Train 전체 Augmentation
```

즉 `03_augment_preview.py`의 역할은

> “이 Augmentation 조건을 Train 전체에 적용해도 괜찮은지 먼저 시험해 보는 코드”

라고 이해하면 됩니다.

실행이 끝나면 원본과 네 가지 변형 이미지, 그리고 한 화면에서 비교할 수 있는 Grid가 `results/apple/augmentation/`에 저장됩니다. 

이제 실제 코드를 작성합니다.

**파일: `scripts/03_augment_preview.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import cv2
import matplotlib.pyplot as plt
import numpy as np


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--rotation-angle",
        type=float,
        default=10.0,
    )

    return parser.parse_args()


def rotate_image(image, angle):
    height, width = image.shape[:2]

    center = (
        width // 2,
        height // 2,
    )

    matrix = cv2.getRotationMatrix2D(
        center,
        angle,
        1.0,
    )

    return cv2.warpAffine(
        image,
        matrix,
        (width, height),
        borderMode=cv2.BORDER_REFLECT_101,
    )


def add_noise(
    image,
    sigma=10,
    seed=42,
):
    rng = np.random.default_rng(
        seed
    )

    noise = rng.normal(
        0,
        sigma,
        image.shape,
    )

    result = (
        image.astype(np.float32)
        + noise
    )

    return np.clip(
        result,
        0,
        255,
    ).astype(np.uint8)


def main() -> None:
    args = parse_args()
    paths = dataset_paths(args.dataset)

    train_files = list_images(
        paths["split_train"]
    )

    if not train_files:
        raise RuntimeError(
            "Train 이미지가 없습니다. "
            "02_prepare_split.py를 먼저 실행하세요."
        )

    source_path = train_files[0]

    image = cv2.imread(
        str(source_path)
    )

    if image is None:
        raise RuntimeError(
            f"이미지를 읽지 못했습니다: {source_path}"
        )

    results = {
        "Original": image.copy(),
        "Brightness": cv2.convertScaleAbs(
            image,
            alpha=0.80,
            beta=0,
        ),
        "Rotation": rotate_image(
            image,
            args.rotation_angle,
        ),
        "Blur": cv2.GaussianBlur(
            image,
            (5, 5),
            0,
        ),
        "Noise": add_noise(
            image,
            sigma=10,
            seed=42,
        ),
    }

    output_dir = paths["result_aug"]

    for name, result in results.items():
        output_path = (
            output_dir
            / f"preview_{name.lower()}.jpg"
        )

        saved = cv2.imwrite(
            str(output_path),
            result,
        )

        if not saved:
            raise RuntimeError(
                f"저장 실패: {output_path}"
            )

    figure = plt.figure(
        figsize=(15, 4)
    )

    for index, (name, result) in enumerate(
        results.items(),
        start=1,
    ):
        axis = figure.add_subplot(
            1,
            len(results),
            index,
        )

        rgb = cv2.cvtColor(
            result,
            cv2.COLOR_BGR2RGB,
        )

        axis.imshow(rgb)
        axis.set_title(name)
        axis.axis("off")

    plt.tight_layout()

    angle_name = str(
        args.rotation_angle
    ).replace(".", "_")

    grid_path = (
        output_dir
        / f"augmentation_grid_rot{angle_name}.jpg"
    )

    plt.savefig(
        grid_path,
        dpi=150,
    )

    plt.close()

    print("Source:", source_path)
    print("Saved :", grid_path)


if __name__ == "__main__":
    main()
```

### 1. 전체 흐름을 빠르게 이해하는 의사코드

```text
apple 또는 banana를 선택한다

Rotation 각도를 입력받는다
→ 기본값 10도

        ↓

앞에서 만든 split/train 폴더에서
Train 이미지 목록을 가져온다

        ↓

Train 이미지가 없으면
02_prepare_split.py를 먼저 실행하라고 오류를 낸다

        ↓

Train의 첫 번째 이미지 1장을 읽는다

        ↓

같은 원본 이미지로 5개 결과를 만든다

Original
→ 원본 그대로

Brightness
→ 밝기를 낮춤

Rotation
→ 지정한 각도만큼 회전

Blur
→ Gaussian Blur 적용

Noise
→ 작은 랜덤 노이즈 추가

        ↓

각 결과 이미지를
results/.../augmentation/ 폴더에 저장한다

        ↓

5개 결과를 한 화면에 나란히 배치한다

Original | Brightness | Rotation | Blur | Noise

        ↓

비교용 Grid 이미지로 저장한다

        ↓

사용한 원본 이미지와
저장된 Grid 경로를 출력한다
```

핵심만 더 줄이면 다음과 같습니다.

```
Train 이미지 1장 선택
        ↓
원본 + 4가지 Augmentation 적용
        ↓
Brightness / Rotation / Blur / Noise
        ↓
각 결과 저장
        ↓
한 장의 비교 Grid로 저장
        ↓
"이 정도 변형이 현실적인가?" 눈으로 확인
```

### 2. 실제 코드와 1:1로 대응하여 읽기

아래에서는 **실제 코드 바로 아래에 `#` 주석으로 의미를 설명**했습니다.

```python
from __future__ import annotations
# 최신 Python의 타입 힌트를 안정적으로 사용할 수 있게 합니다.


import argparse
# --dataset, --rotation-angle 같은 실행 옵션을 받기 위해 사용합니다.

import sys
# Python이 프로젝트의 src 폴더를 찾을 수 있도록 검색 경로를 설정할 때 사용합니다.

from pathlib import Path
# 파일과 폴더 경로를 다루기 위해 사용합니다.


import cv2
# 이미지 읽기, 밝기 변경, 회전, Blur, 저장 등에 사용하는 OpenCV입니다.

import matplotlib.pyplot as plt
# 여러 Augmentation 결과를 한 화면의 Grid로 만들어 저장할 때 사용합니다.

import numpy as np
# Noise를 만들고 이미지 배열을 계산할 때 사용합니다.


ROOT = Path(__file__).resolve().parents[1]
# 현재 scripts/03_augment_preview.py를 기준으로
# day01_image_aug_vae 프로젝트 ROOT 경로를 구합니다.

sys.path.insert(0, str(ROOT))
# 프로젝트 ROOT를 Python 모듈 검색 경로에 추가합니다.
# 따라서 아래의 src.day01_common을 import할 수 있습니다.


from src.day01_common import (
    dataset_paths,
    list_images,
)
# 앞에서 만든 공통 함수를 가져옵니다.
#
# dataset_paths()
# → apple / banana의 폴더 경로를 가져옴
#
# list_images()
# → 폴더 안의 이미지 목록을 가져옴


def parse_args():
# 터미널에서 입력하는 옵션을 처리하는 함수를 만듭니다.

    parser = argparse.ArgumentParser()
    # 명령행 옵션을 관리할 Parser를 만듭니다.


    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )
    # 사용할 데이터셋을 선택합니다.
    #
    # 기본값 → apple
    # 선택 가능 → apple / banana


    parser.add_argument(
        "--rotation-angle",
        type=float,
        default=10.0,
    )
    # Rotation에서 사용할 회전 각도를 입력받습니다.
    #
    # 기본값 → 10도


    return parser.parse_args()
    # 사용자가 입력한 옵션을 반환합니다.


def rotate_image(image, angle):
# 이미지를 지정한 각도만큼 회전시키는 함수를 만듭니다.

    height, width = image.shape[:2]
    # 원본 이미지의 높이와 너비를 가져옵니다.


    center = (
        width // 2,
        height // 2,
    )
    # 이미지의 중심 좌표를 계산합니다.
    # 이 중심점을 기준으로 이미지를 회전합니다.


    matrix = cv2.getRotationMatrix2D(
        center,
        angle,
        1.0,
    )
    # 회전에 필요한 변환 행렬을 만듭니다.
    #
    # center → 회전 중심
    # angle  → 회전 각도
    # 1.0    → 이미지 크기는 그대로 유지


    return cv2.warpAffine(
        image,
        matrix,
        (width, height),
        borderMode=cv2.BORDER_REFLECT_101,
    )
    # 실제로 이미지에 회전을 적용합니다.
    #
    # 회전하면서 비는 가장자리 영역은
    # 주변 이미지를 반사하여 채웁니다.


def add_noise(
    image,
    sigma=10,
    seed=42,
):
# 이미지에 랜덤 Noise를 추가하는 함수를 만듭니다.
#
# sigma
# → Noise의 강도
#
# seed
# → 같은 Noise 결과를 다시 만들기 위한 기준값


    rng = np.random.default_rng(
        seed
    )
    # Seed를 이용하여 랜덤 숫자 생성기를 만듭니다.


    noise = rng.normal(
        0,
        sigma,
        image.shape,
    )
    # 이미지와 같은 크기의 랜덤 Noise를 만듭니다.
    #
    # 평균 → 0
    # 표준편차 → sigma


    result = (
        image.astype(np.float32)
        + noise
    )
    # 원본 이미지 픽셀값에 Noise를 더합니다.
    #
    # 계산 중 값 손실을 줄이기 위해
    # float32로 변환하여 계산합니다.


    return np.clip(
        result,
        0,
        255,
    ).astype(np.uint8)
    # 픽셀값이 이미지 범위인 0~255를 벗어나지 않도록 제한합니다.
    #
    # 마지막에 다시 일반 이미지 형식인 uint8로 변환합니다.


def main() -> None:
# 프로그램의 실제 실행 흐름을 시작합니다.

    args = parse_args()
    # 사용자가 입력한 dataset과 rotation-angle을 읽습니다.


    paths = dataset_paths(args.dataset)
    # 선택한 apple 또는 banana의 폴더 경로들을 가져옵니다.


    train_files = list_images(
        paths["split_train"]
    )
    # 앞에서 Split한 Train 폴더에서 이미지 목록을 가져옵니다.
    #
    # 주의:
    # source_train이 아니라 split/train을 사용합니다.


    if not train_files:
        raise RuntimeError(
            "Train 이미지가 없습니다. "
            "02_prepare_split.py를 먼저 실행하세요."
        )
    # Train 이미지가 없다면 실습을 중단합니다.
    #
    # 즉 02_prepare_split.py를 먼저 실행해야
    # 이 코드가 정상적으로 동작합니다.


    source_path = train_files[0]
    # Train 이미지 중 첫 번째 이미지 한 장을 선택합니다.
    #
    # 이 코드는 아직 전체 Train을 보강하는 것이 아니라
    # 대표 이미지 한 장으로 Augmentation을 Preview하는 단계입니다.


    image = cv2.imread(
        str(source_path)
    )
    # 선택한 이미지를 OpenCV로 읽습니다.


    if image is None:
        raise RuntimeError(
            f"이미지를 읽지 못했습니다: {source_path}"
        )
    # 이미지를 정상적으로 읽지 못했다면 실행을 중단합니다.


    results = {
        "Original": image.copy(),
        # 원본 이미지를 그대로 보관합니다.


        "Brightness": cv2.convertScaleAbs(
            image,
            alpha=0.80,
            beta=0,
        ),
        # 밝기 변화를 적용합니다.
        #
        # alpha=0.80
        # → 원본보다 약간 어둡게 만듭니다.


        "Rotation": rotate_image(
            image,
            args.rotation_angle,
        ),
        # 앞에서 만든 rotate_image()를 이용하여
        # 기본 10도 또는 사용자가 지정한 각도로 회전합니다.


        "Blur": cv2.GaussianBlur(
            image,
            (5, 5),
            0,
        ),
        # Gaussian Blur를 적용하여
        # 이미지가 약간 흐려진 상황을 만듭니다.
        #
        # (5, 5)
        # → Blur 계산에 사용하는 Kernel 크기입니다.


        "Noise": add_noise(
            image,
            sigma=10,
            seed=42,
        ),
        # 앞에서 만든 add_noise()를 이용하여
        # 작은 랜덤 Noise를 이미지에 추가합니다.
    }
    # 결과적으로 하나의 원본에서
    #
    # Original
    # Brightness
    # Rotation
    # Blur
    # Noise
    #
    # 총 5개 이미지를 준비합니다.


    output_dir = paths["result_aug"]
    # Augmentation 결과를 저장할 폴더를 가져옵니다.
    #
    # 예:
    # results/apple/augmentation/


    for name, result in results.items():
    # 5개의 결과를 하나씩 꺼냅니다.


        output_path = (
            output_dir
            / f"preview_{name.lower()}.jpg"
        )
        # 각 결과의 저장 파일명을 만듭니다.
        #
        # 예:
        # preview_original.jpg
        # preview_brightness.jpg
        # preview_rotation.jpg
        # preview_blur.jpg
        # preview_noise.jpg


        saved = cv2.imwrite(
            str(output_path),
            result,
        )
        # 각 Augmentation 결과를 실제 JPG 파일로 저장합니다.


        if not saved:
            raise RuntimeError(
                f"저장 실패: {output_path}"
            )
        # 파일 저장에 실패하면 프로그램을 중단합니다.


    figure = plt.figure(
        figsize=(15, 4)
    )
    # 5개의 이미지를 한 화면에 보여줄
    # Matplotlib Figure를 만듭니다.


    for index, (name, result) in enumerate(
        results.items(),
        start=1,
    ):
    # Original부터 Noise까지 결과를 하나씩 꺼냅니다.
    #
    # index는 1부터 시작합니다.


        axis = figure.add_subplot(
            1,
            len(results),
            index,
        )
        # 1행 × 5열 구조로 이미지를 배치합니다.


        rgb = cv2.cvtColor(
            result,
            cv2.COLOR_BGR2RGB,
        )
        # OpenCV 이미지는 BGR 순서이지만
        # Matplotlib은 RGB 순서를 사용합니다.
        #
        # 따라서 정상적인 색상으로 보이도록 변환합니다.


        axis.imshow(rgb)
        # 현재 이미지를 화면에 표시합니다.


        axis.set_title(name)
        # 이미지 위에
        # Original / Brightness / Rotation / Blur / Noise
        # 제목을 표시합니다.


        axis.axis("off")
        # 이미지 주변의 x축, y축 숫자는 숨깁니다.


    plt.tight_layout()
    # 이미지와 제목이 서로 겹치지 않도록
    # 배치를 자동으로 정리합니다.


    angle_name = str(
        args.rotation_angle
    ).replace(".", "_")
    # Rotation 값을 파일명에 안전하게 사용할 수 있도록 바꿉니다.
    #
    # 예:
    # 10.0 → 10_0


    grid_path = (
        output_dir
        / f"augmentation_grid_rot{angle_name}.jpg"
    )
    # 5개 이미지를 한 번에 비교할 Grid 이미지의
    # 저장 경로를 만듭니다.
    #
    # 예:
    # augmentation_grid_rot10_0.jpg


    plt.savefig(
        grid_path,
        dpi=150,
    )
    # 완성된 비교 Grid를 이미지 파일로 저장합니다.


    plt.close()
    # 사용한 Matplotlib Figure를 닫아
    # 메모리를 정리합니다.


    print("Source:", source_path)
    # 어떤 원본 이미지를 사용했는지 출력합니다.


    print("Saved :", grid_path)
    # 비교 Grid가 어디에 저장되었는지 출력합니다.


if __name__ == "__main__":
# 이 파일을 직접 실행했을 때만 아래 main()을 실행합니다.

    main()
    # Augmentation Preview 실습을 시작합니다.
```

## 이 코드 리뷰에서 특히 봐야 할 핵심

전체 코드를 다 외우기보다 다음 흐름만 확실히 이해하면 됩니다.

```
① split_train에서 이미지 한 장 선택

train_files = list_images(...)
source_path = train_files[0]

        ↓

② 같은 원본으로 4가지 변형 생성

Brightness
Rotation
Blur
Noise

        ↓

③ 각 결과를 파일로 저장

cv2.imwrite()

        ↓

④ Original과 변형 결과를
한 화면에 배치

Matplotlib

        ↓

⑤ 비교 Grid 저장

augmentation_grid_rot10_0.jpg
```

특히 이 단계의 목적은 **“Augmentation을 많이 만드는 것”이 아닙니다.**

```
Train 이미지 1장
        ↓
여러 변형을 미리 적용
        ↓
눈으로 비교
        ↓
사과의 의미가 유지되는가?
변형이 너무 강하지 않은가?
실제 환경에서 생길 법한 변화인가?
        ↓
적절한 조건을 결정
        ↓
그다음 전체 Train 보강
```

즉 `03_augment_preview.py`는 
**본격적으로 전체 Train 데이터를 늘리기 전에 Augmentation 조건이 적절한지 미리 확인하는 코드**라고 이해하면 가장 정확합니다.

---

실행합니다.

```bash
python scripts/03_augment_preview.py \
  --dataset apple
```

결과를 확인합니다.

```text
Source: /home/youjung/ai_vision/subject11_Synthetic_Data/day01_image_aug_vae/data/apple/split/train/0_100.jpg
Saved : /home/youjung/ai_vision/subject11_Synthetic_Data/day01_image_aug_vae/results/apple/augmentation/augmentation_grid_rot10_0.jpg
```

### 실행 결과를 해석합니다

`Source`는 이번 Preview에 사용된 원본 Train 이미지입니다.

```
Source
→ data/apple/split/train/0_100.jpg
```

즉 Train 이미지 중 한 장을 가져와 Augmentation을 시험한 것입니다.

`Saved`는 원본과 네 가지 변형을 한 화면에서 비교할 수 있도록 만든 결과 Grid 이미지입니다.

```
Saved
→ augmentation_grid_rot10_0.jpg
```

`rot10_0`은 이번 Preview에서 Rotation을 `10°`로 적용했다는 의미입니다.

실행 후 다음 파일들이 만들어집니다. 

![[Pasted image 20261008093327.png]]

```
results/apple/augmentation/
├─ preview_original.jpg
├─ preview_brightness.jpg
├─ preview_rotation.jpg
├─ preview_blur.jpg
├─ preview_noise.jpg
└─ augmentation_grid_rot10_0.jpg
```

각 파일은 다음 결과를 보여줍니다.

```
Original
→ 원본 이미지

Brightness
→ 밝기를 변경한 이미지

Rotation
→ 10° 회전한 이미지

Blur
→ 조금 흐리게 만든 이미지

Noise
→ 픽셀에 잡음을 추가한 이미지

Grid
→ 위 결과를 한 화면에서 비교
```

### 무엇을 확인하면 되나요?

지금은 어느 방법이 더 좋은지 점수를 계산하는 단계가 아닙니다.

Grid를 직접 보면서 다음 한 가지만 판단합니다.

> 변형된 이미지가 여전히 자연스러운 Apple 이미지로 보이는가?

즉,

```
원본과 비교
        ↓
변형이 너무 강하지 않은가?
        ↓
사과의 색과 형태가 유지되는가?
        ↓
실제 촬영에서도 일어날 수 있는 변화인가?
```

를 확인합니다.

이 Preview가 괜찮다면 다음 단계에서 한 장이 아니라 Train 전체에 Augmentation을 적용합니다.

---

# 14. Augmentation 결과를 눈으로 비교합니다

앞에서 만든 `augmentation_grid_rot10_0.jpg`를 열어 원본과 변형 이미지를 비교합니다.

이번 단계에서는 복잡한 수치를 계산하지 않고, 변형 후에도 원래 사과의 특징이 유지되는지 눈으로 확인합니다.

## Brightness

```text
너무 어둡거나 밝아지지 않았는가?

사과가 조금 어두워졌는가?

여전히 같은 빨간 사과로 볼 수 있는가?
```

## Rotation

```text
회전 후에도 사과가 자연스럽게 보이는가?


10° 정도 회전했을 때
사과의 의미가 유지되는가?

객체 일부가 지나치게 잘리지는 않았는가?
```

## Blur

```text
흐려졌지만 형태를 알아볼 수 있는가?

약간 흐려졌지만
형태를 구분할 수 있는가?
```

## Noise

```text
잡음이 있어도 색과 윤곽이 유지되는가?

Noise가 생겼지만
사과의 색·윤곽이 유지되는가?
```

변형이 눈에 잘 보인다는 이유만으로 좋은 데이터가 되는 것은 아닙니다.

---

# 15. Train 전체에 적용할 Augmentation 범위를 정합니다

14번에서는 Train 이미지 한 장에 Brightness, Rotation, Blur, Noise를 적용하여 
**변형 후에도 사과 이미지가 자연스럽게 유지되는지 눈으로 확인**했습니다.

이제 다음 단계에서는 한 장이 아니라 **Train 전체 이미지에 
Augmentation을 적용**해야 합니다.

하지만 모든 이미지를 똑같이 어둡게 하거나 
정확히 10도씩 회전시키는 것이 아니라, 
**현실적인 범위 안에서 조금씩 다른 변화를 주어 다양한 학습 데이터를 만들 것**입니다.

오늘은 다음 범위를 기본값으로 사용합니다.

```
Brightness
→ 원래 밝기의 약 80% ~ 120%

Rotation
→ 왼쪽 10° ~ 오른쪽 10°

Blur
→ 약한 흐림 정도
→ kernel 3 또는 5

Noise
→ 약한 ~ 중간 정도의 노이즈
→ sigma 5 / 10 / 15
```

이 값들은 모든 이미지에 적용되는 정답이 아닙니다.

```
너무 약한 변형
→ 원본과 거의 차이가 없음

적절한 변형
→ 실제 촬영에서 생길 수 있는 변화
→ 학습 데이터 다양성 증가

너무 강한 변형
→ 사과의 원래 특징이 훼손될 수 있음
```

따라서 14번 Preview에서 
**사과의 형태와 의미가 유지되는 범위인지 먼저 확인한 뒤**, 
그 범위를 Train 전체 보강에 사용합니다.

```
한 장으로 Preview
        ↓
변형이 자연스러운지 확인
        ↓
사용할 범위 결정
        ↓
다음 단계에서
Train 전체에 Augmentation 적용
```

> 15번의 목적은 새로운 이미지를 만드는 것이 아니라, 
> 다음 전체 Train 보강에서 사용할 “안전한 변형 범위”를 정하는 것입니다.
> 
---

# PART 4. Train 전체를 자동으로 보강하고 Metadata를 기록합니다

---

# 16. Metadata가 왜 필요한지 이해합니다

합성 이미지만 저장하면 나중에 다음 질문에 답하기 어렵습니다.

```text
이 이미지는 어떤 원본에서 만들었나?

무슨 방법을 사용했나?

회전각도는 몇 도였나?

어떤 Seed를 사용했나?
```

그래서 생성 이미지와 함께 **생성 이력**을 기록합니다.

오늘 Metadata에는 다음 정보를 남깁니다.

```text
생성 파일명
원본 파일명
원본 파일 Hash
Augmentation 방법
사용한 Parameter
Seed
검수 상태
```

---

# 17. Train 전체를 자동으로 보강하는 코드를 작성합니다

**파일: `scripts/04_augment_train.py`**

```python
from __future__ import annotations

import argparse
import hashlib
import random
import shutil
import sys
from pathlib import Path

import cv2
import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


SEED = 42


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--aug-per-image",
        type=int,
        default=2,
    )

    parser.add_argument(
        "--rotation-max",
        type=float,
        default=10.0,
    )

    return parser.parse_args()


def sha256_file(path: Path) -> str:
    digest = hashlib.sha256()

    with path.open("rb") as file:
        for chunk in iter(
            lambda: file.read(1024 * 1024),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()


def rotate_image(image, angle):
    height, width = image.shape[:2]

    center = (
        width // 2,
        height // 2,
    )

    matrix = cv2.getRotationMatrix2D(
        center,
        angle,
        1.0,
    )

    return cv2.warpAffine(
        image,
        matrix,
        (width, height),
        borderMode=cv2.BORDER_REFLECT_101,
    )


def apply_augmentation(
    image,
    method,
    py_rng,
    np_rng,
    rotation_max,
):
    if method == "brightness":
        alpha = py_rng.uniform(
            0.8,
            1.2,
        )

        result = cv2.convertScaleAbs(
            image,
            alpha=alpha,
            beta=0,
        )

        parameter = (
            f"alpha={alpha:.3f}"
        )

    elif method == "rotation":
        angle = py_rng.uniform(
            -rotation_max,
            rotation_max,
        )

        result = rotate_image(
            image,
            angle,
        )

        parameter = (
            f"angle={angle:.2f}"
        )

    elif method == "blur":
        kernel = py_rng.choice(
            [3, 5]
        )

        result = cv2.GaussianBlur(
            image,
            (kernel, kernel),
            0,
        )

        parameter = (
            f"kernel={kernel}"
        )

    elif method == "noise":
        sigma = py_rng.choice(
            [5, 10, 15]
        )

        noise = np_rng.normal(
            0,
            sigma,
            image.shape,
        )

        result = (
            image.astype(np.float32)
            + noise
        )

        result = np.clip(
            result,
            0,
            255,
        ).astype(np.uint8)

        parameter = (
            f"sigma={sigma}"
        )

    else:
        raise ValueError(
            f"Unknown method: {method}"
        )

    return result, parameter


def main() -> None:
    args = parse_args()
    paths = dataset_paths(args.dataset)

    train_files = list_images(
        paths["split_train"]
    )

    if not train_files:
        raise RuntimeError(
            "Train 이미지가 없습니다."
        )

    output_dir = (
        paths["augmented_train"]
    )

    if output_dir.exists():
        shutil.rmtree(output_dir)

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    methods = [
        "brightness",
        "rotation",
        "blur",
        "noise",
    ]

    rows = []

    for source_index, source_path in enumerate(
        train_files
    ):
        image = cv2.imread(
            str(source_path)
        )

        if image is None:
            print(
                "[SKIP]",
                source_path.name,
            )
            continue

        source_hash = sha256_file(
            source_path
        )

        for index in range(
            1,
            args.aug_per_image + 1,
        ):
            sample_seed = (
                SEED
                + source_index
                * args.aug_per_image
                + (index - 1)
            )

            py_rng = random.Random(
                sample_seed
            )

            np_rng = np.random.default_rng(
                sample_seed
            )

            method = py_rng.choice(
                methods
            )

            result, parameter = (
                apply_augmentation(
                    image,
                    method,
                    py_rng,
                    np_rng,
                    args.rotation_max,
                )
            )

            output_name = (
                f"{source_path.stem}"
                f"__{method}"
                f"__{index:02d}.jpg"
            )

            output_path = (
                output_dir
                / output_name
            )

            saved = cv2.imwrite(
                str(output_path),
                result,
            )

            if not saved:
                raise RuntimeError(
                    f"저장 실패: {output_path}"
                )

            rows.append(
                {
                    "file": output_name,
                    "method": method,
                    "source": source_path.name,
                    "source_hash": source_hash,
                    "parameter": parameter,
                    "seed": sample_seed,
                    "approved": "pending",
                }
            )

    metadata_path = (
        paths["report"]
        / "synthetic_metadata.csv"
    )

    pd.DataFrame(
        rows
    ).to_csv(
        metadata_path,
        index=False,
        encoding="utf-8-sig",
    )

    print("=" * 60)
    print("Augmentation Completed")
    print("=" * 60)

    print(
        "Original Train:",
        len(train_files),
    )

    print(
        "Generated     :",
        len(rows),
    )

    print(
        "Rotation Max :",
        args.rotation_max,
    )

    print(
        "Metadata     :",
        metadata_path,
    )


if __name__ == "__main__":
    main()
```

### 1. 전체 흐름을 빠르게 이해하는 의사코드

```text
apple 또는 banana를 선택한다

한 원본 이미지당
몇 장의 보강 이미지를 만들지 정한다
→ 기본값 2장

Rotation 최대 범위를 정한다
→ 기본값 ±10도

        ↓

split/train 폴더에서
Train 이미지 목록을 가져온다

        ↓

기존 augmented/train 폴더가 있으면
이전 결과를 지우고 새로 만든다

        ↓

사용할 Augmentation 방법을 준비한다

Brightness
Rotation
Blur
Noise

        ↓

Train 이미지를 하나씩 읽는다

        ↓

각 원본 이미지마다
기본값 2개의 보강 이미지를 만든다

각 보강 이미지마다

Seed를 만든다
        ↓
4가지 방법 중 하나를 랜덤 선택한다
        ↓
선택된 방법의 강도도 랜덤하게 정한다

Brightness
→ alpha 0.8 ~ 1.2

Rotation
→ -10° ~ +10°

Blur
→ kernel 3 또는 5

Noise
→ sigma 5 / 10 / 15

        ↓

보강 이미지를 저장한다

        ↓

동시에 Metadata를 기록한다

파일명
사용한 방법
원본 파일명
원본 Hash
사용한 Parameter
Seed
승인 상태

        ↓

모든 이미지 처리가 끝나면

synthetic_metadata.csv 저장

        ↓

원본 Train 수
생성된 Augmentation 수
Rotation 최대값
Metadata 위치

를 출력한다
```

핵심만 의사코드로 표현하면 다음과 같습니다.

```
Train 전체 이미지
        ↓
각 원본당 2개 보강 생성
        ↓
Brightness / Rotation / Blur / Noise
중 하나를 랜덤 선택
        ↓
보강 강도도 정해진 범위 안에서 랜덤 선택
        ↓
augmented/train에 저장
        +
synthetic_metadata.csv에 생성 이력 기록
```

### 2. 실제 코드와 1:1로 대응하는 상세 코드 리뷰

```python
from __future__ import annotations
# 최신 Python의 타입 힌트를 안정적으로 사용할 수 있게 합니다.


import argparse
# --dataset, --aug-per-image, --rotation-max 같은
# 실행 옵션을 받기 위해 사용합니다.

import hashlib
# 원본 이미지의 SHA-256 Hash 값을 만들기 위해 사용합니다.
# 나중에 어떤 원본에서 생성된 이미지인지 추적하는 데 사용합니다.

import random
# Augmentation 방법과 Parameter를 랜덤하게 선택할 때 사용합니다.

import shutil
# 기존 Augmentation 결과 폴더를 삭제할 때 사용합니다.

import sys
# Python 모듈 검색 경로를 설정할 때 사용합니다.

from pathlib import Path
# 파일과 폴더 경로를 편리하게 다루기 위해 사용합니다.


import cv2
# 이미지 읽기, 회전, 밝기, Blur, 저장에 사용합니다.

import numpy as np
# Noise 생성과 이미지 배열 계산에 사용합니다.

import pandas as pd
# 생성된 Metadata를 CSV 파일로 저장할 때 사용합니다.


ROOT = Path(__file__).resolve().parents[1]
# 현재 scripts 파일을 기준으로
# day01_image_aug_vae 프로젝트 ROOT 경로를 구합니다.


sys.path.insert(0, str(ROOT))
# 프로젝트 ROOT를 Python 모듈 검색 경로에 추가합니다.


from src.day01_common import (
    dataset_paths,
    list_images,
)
# 앞에서 만든 공통 함수를 가져옵니다.
#
# dataset_paths()
# → apple / banana 관련 폴더 경로를 가져옴
#
# list_images()
# → 지정한 폴더 안의 이미지 목록을 가져옴


SEED = 42
# Augmentation 결과를 재현하기 위한
# 기본 Seed 값입니다.


def parse_args():
# 터미널에서 입력받을 옵션을 정의합니다.

    parser = argparse.ArgumentParser()
    # 명령행 옵션을 관리할 Parser를 만듭니다.


    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )
    # 사용할 데이터셋을 선택합니다.
    #
    # 기본값 → apple


    parser.add_argument(
        "--aug-per-image",
        type=int,
        default=2,
    )
    # 원본 이미지 한 장당
    # 몇 장의 Augmentation 이미지를 만들지 정합니다.
    #
    # 기본값 → 2장


    parser.add_argument(
        "--rotation-max",
        type=float,
        default=10.0,
    )
    # Rotation의 최대 범위를 정합니다.
    #
    # 기본값 10
    # → -10도 ~ +10도 사이에서 랜덤 선택


    return parser.parse_args()
    # 입력된 실행 옵션을 반환합니다.


def sha256_file(path: Path) -> str:
# 파일의 SHA-256 Hash 값을 계산하는 함수입니다.

    digest = hashlib.sha256()
    # SHA-256 계산 객체를 만듭니다.


    with path.open("rb") as file:
    # 이미지 파일을 바이너리 읽기 모드로 엽니다.

        for chunk in iter(
            lambda: file.read(1024 * 1024),
            b"",
        ):
        # 파일을 한 번에 전부 읽지 않고
        # 1MB씩 나누어 읽습니다.

            digest.update(chunk)
            # 읽은 내용을 Hash 계산에 추가합니다.


    return digest.hexdigest()
    # 최종 SHA-256 Hash 문자열을 반환합니다.
    #
    # 같은 파일이면 같은 Hash가 만들어집니다.


def rotate_image(image, angle):
# 이미지를 지정한 각도로 회전시키는 함수입니다.

    height, width = image.shape[:2]
    # 이미지 높이와 너비를 가져옵니다.


    center = (
        width // 2,
        height // 2,
    )
    # 이미지 중심 좌표를 계산합니다.


    matrix = cv2.getRotationMatrix2D(
        center,
        angle,
        1.0,
    )
    # 이미지 중심을 기준으로
    # 지정한 각도만큼 회전하는 행렬을 만듭니다.


    return cv2.warpAffine(
        image,
        matrix,
        (width, height),
        borderMode=cv2.BORDER_REFLECT_101,
    )
    # 실제 회전을 적용합니다.
    #
    # 회전하면서 생기는 빈 영역은
    # 주변 픽셀을 반사하여 채웁니다.


def apply_augmentation(
    image,
    method,
    py_rng,
    np_rng,
    rotation_max,
):
# 선택된 Augmentation 방법을
# 실제 이미지에 적용하는 함수입니다.


    if method == "brightness":
    # 선택된 방법이 Brightness이면 실행합니다.

        alpha = py_rng.uniform(
            0.8,
            1.2,
        )
        # 밝기 배율을
        # 0.8 ~ 1.2 사이에서 랜덤 선택합니다.


        result = cv2.convertScaleAbs(
            image,
            alpha=alpha,
            beta=0,
        )
        # 선택한 alpha를 이용하여
        # 이미지 밝기를 변경합니다.


        parameter = (
            f"alpha={alpha:.3f}"
        )
        # 실제 사용한 밝기 값을 문자열로 기록합니다.


    elif method == "rotation":
    # Rotation이 선택된 경우입니다.

        angle = py_rng.uniform(
            -rotation_max,
            rotation_max,
        )
        # -rotation_max ~ +rotation_max 사이에서
        # 회전 각도를 랜덤 선택합니다.


        result = rotate_image(
            image,
            angle,
        )
        # 선택한 각도로 이미지를 회전합니다.


        parameter = (
            f"angle={angle:.2f}"
        )
        # 실제 사용한 회전 각도를 기록합니다.


    elif method == "blur":
    # Blur가 선택된 경우입니다.

        kernel = py_rng.choice(
            [3, 5]
        )
        # Blur 강도를 결정할 Kernel 크기를
        # 3 또는 5 중 하나로 선택합니다.


        result = cv2.GaussianBlur(
            image,
            (kernel, kernel),
            0,
        )
        # 선택한 Kernel로 Gaussian Blur를 적용합니다.


        parameter = (
            f"kernel={kernel}"
        )
        # 사용한 Kernel 값을 기록합니다.


    elif method == "noise":
    # Noise가 선택된 경우입니다.

        sigma = py_rng.choice(
            [5, 10, 15]
        )
        # Noise 강도를
        # 5, 10, 15 중 하나로 선택합니다.


        noise = np_rng.normal(
            0,
            sigma,
            image.shape,
        )
        # 이미지와 같은 크기의
        # Gaussian Noise를 만듭니다.


        result = (
            image.astype(np.float32)
            + noise
        )
        # 원본 이미지 픽셀값에 Noise를 더합니다.


        result = np.clip(
            result,
            0,
            255,
        ).astype(np.uint8)
        # 픽셀값이 0~255를 벗어나지 않도록 제한하고
        # 다시 uint8 이미지 형식으로 변환합니다.


        parameter = (
            f"sigma={sigma}"
        )
        # 실제 사용한 sigma 값을 기록합니다.


    else:
    # 정의되지 않은 방법이 들어온 경우입니다.

        raise ValueError(
            f"Unknown method: {method}"
        )
        # 잘못된 Augmentation 방법이면
        # 프로그램을 중단합니다.


    return result, parameter
    # 보강된 이미지와
    # 실제 사용한 Parameter 정보를 함께 반환합니다.


def main() -> None:
# 프로그램의 전체 실행 흐름입니다.

    args = parse_args()
    # 터미널에서 입력한 옵션을 읽습니다.


    paths = dataset_paths(args.dataset)
    # 선택한 데이터셋의 경로 정보를 가져옵니다.


    train_files = list_images(
        paths["split_train"]
    )
    # 앞에서 만든 split/train의
    # Train 이미지 목록을 가져옵니다.


    if not train_files:
        raise RuntimeError(
            "Train 이미지가 없습니다."
        )
    # Train 이미지가 하나도 없으면
    # Augmentation을 진행할 수 없으므로 중단합니다.


    output_dir = (
        paths["augmented_train"]
    )
    # 생성된 Augmentation 이미지를 저장할
    # augmented/train 폴더 경로를 가져옵니다.


    if output_dir.exists():
        shutil.rmtree(output_dir)
    # 이전 실행 결과가 남아 있다면
    # 결과가 섞이지 않도록 폴더를 삭제합니다.


    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )
    # 새로운 augmented/train 폴더를 만듭니다.


    methods = [
        "brightness",
        "rotation",
        "blur",
        "noise",
    ]
    # 사용할 Augmentation 방법 4가지를 준비합니다.


    rows = []
    # 각 생성 이미지의 Metadata를
    # 저장할 빈 리스트를 만듭니다.


    for source_index, source_path in enumerate(
        train_files
    ):
    # Train 이미지를 한 장씩 처리합니다.
    #
    # source_index
    # → 현재 원본 이미지 번호
    #
    # source_path
    # → 현재 원본 이미지 경로


        image = cv2.imread(
            str(source_path)
        )
        # 현재 Train 이미지를 OpenCV로 읽습니다.


        if image is None:
            print(
                "[SKIP]",
                source_path.name,
            )
            continue
        # 이미지를 읽을 수 없다면
        # 해당 파일은 건너뜁니다.


        source_hash = sha256_file(
            source_path
        )
        # 현재 원본 이미지의 SHA-256 Hash를 계산합니다.
        #
        # 생성된 이미지가 어떤 원본에서 만들어졌는지
        # 추적하기 위한 값입니다.


        for index in range(
            1,
            args.aug_per_image + 1,
        ):
        # 원본 이미지 한 장에서
        # 지정한 수만큼 Augmentation 이미지를 만듭니다.
        #
        # 기본값이면 2회 반복합니다.


            sample_seed = (
                SEED
                + source_index
                * args.aug_per_image
                + (index - 1)
            )
            # 각 생성 이미지마다
            # 서로 다른 Seed를 계산합니다.
            #
            # 하지만 같은 조건으로 다시 실행하면
            # 동일한 Seed가 만들어집니다.


            py_rng = random.Random(
                sample_seed
            )
            # Python random용
            # 랜덤 생성기를 만듭니다.


            np_rng = np.random.default_rng(
                sample_seed
            )
            # NumPy용 랜덤 생성기를 만듭니다.
            #
            # Noise 생성 등에 사용합니다.


            method = py_rng.choice(
                methods
            )
            # Brightness / Rotation / Blur / Noise 중
            # 하나를 랜덤하게 선택합니다.


            result, parameter = (
                apply_augmentation(
                    image,
                    method,
                    py_rng,
                    np_rng,
                    args.rotation_max,
                )
            )
            # 선택된 Augmentation을 실제로 적용합니다.
            #
            # result
            # → 생성된 이미지
            #
            # parameter
            # → 실제 사용한 변형값


            output_name = (
                f"{source_path.stem}"
                f"__{method}"
                f"__{index:02d}.jpg"
            )
            # 생성 이미지의 파일명을 만듭니다.
            #
            # 예:
            # 266_100__rotation__01.jpg
            # 266_100__noise__02.jpg


            output_path = (
                output_dir
                / output_name
            )
            # 생성 이미지를 저장할 전체 경로를 만듭니다.


            saved = cv2.imwrite(
                str(output_path),
                result,
            )
            # 실제 Augmentation 이미지를 파일로 저장합니다.


            if not saved:
                raise RuntimeError(
                    f"저장 실패: {output_path}"
                )
            # 저장에 실패하면 프로그램을 중단합니다.


            rows.append(
                {
                    "file": output_name,
                    "method": method,
                    "source": source_path.name,
                    "source_hash": source_hash,
                    "parameter": parameter,
                    "seed": sample_seed,
                    "approved": "pending",
                }
            )
            # 방금 생성한 이미지의 정보를
            # Metadata 목록에 기록합니다.
            #
            # file
            # → 생성 이미지 파일명
            #
            # method
            # → 어떤 Augmentation을 사용했는지
            #
            # source
            # → 어떤 원본 이미지에서 만들었는지
            #
            # source_hash
            # → 원본 이미지의 고유 Hash
            #
            # parameter
            # → 실제 사용한 변형 강도
            #
            # seed
            # → 재현에 사용한 Seed
            #
            # approved
            # → 아직 QA 전이므로 pending


    metadata_path = (
        paths["report"]
        / "synthetic_metadata.csv"
    )
    # Metadata를 저장할 CSV 경로를 만듭니다.


    pd.DataFrame(
        rows
    ).to_csv(
        metadata_path,
        index=False,
        encoding="utf-8-sig",
    )
    # rows에 기록된 Metadata를
    # Pandas DataFrame으로 만든 뒤 CSV로 저장합니다.


    print("=" * 60)
    print("Augmentation Completed")
    print("=" * 60)
    # Augmentation 작업이 완료되었음을 출력합니다.


    print(
        "Original Train:",
        len(train_files),
    )
    # 원본 Train 이미지 수를 출력합니다.


    print(
        "Generated     :",
        len(rows),
    )
    # 실제 생성된 Augmentation 이미지 수를 출력합니다.


    print(
        "Rotation Max :",
        args.rotation_max,
    )
    # 사용한 Rotation 최대값을 출력합니다.


    print(
        "Metadata     :",
        metadata_path,
    )
    # Metadata CSV가 저장된 위치를 출력합니다.


if __name__ == "__main__":
    main()
# 이 Python 파일을 직접 실행했을 때
# main()을 실행하여 전체 Augmentation 작업을 시작합니다.
```

이 코드에서 가장 중요한 흐름은 다음입니다.

```
split/train의 원본 이미지
        ↓
원본마다 정해진 개수만큼 반복
        ↓
Augmentation 방법 랜덤 선택
        ↓
Parameter 랜덤 선택
        ↓
보강 이미지 생성
        ↓
augmented/train에 저장
        +
Metadata 기록
```

특히 이전 Preview 코드와의 차이는 이렇게 이해하면 쉽습니다.

```
03_augment_preview.py
→ 대표 이미지 1장
→ 변형이 자연스러운지 먼저 확인

        ↓

04_augment_train.py
→ 확인한 범위를 이용
→ Train 전체 이미지에 실제 Augmentation 적용
→ 생성 이력까지 Metadata로 기록
```

즉 이 코드는 **“미리 확인한 Augmentation 기준을 Train 전체에 실제 적용하여 학습용 보강 데이터를 만드는 단계”**입니다.

---

실행합니다.

```bash
python scripts/04_augment_train.py \
  --dataset apple
```

### 실행 결과 해석

```
Original Train: 419
Generated     : 838
Rotation Max : 10.0
Metadata     : .../reports/apple/synthetic_metadata.csv
```

이번 실행에서는 **Train 원본 419장 전체에 Augmentation을 적용**했습니다.

기본 설정이 원본 한 장당 보강 이미지 2장을 생성하도록 되어 있으므로,

```
원본 Train 419장
        ↓
한 장당 2개씩 Augmentation
        ↓
419 × 2
        ↓
보강 이미지 838장 생성
```

되었습니다. `Generated : 838`이 정확히 출력되었으므로 이번 실행에서는 원본 419장에 대해 예정한 보강 이미지가 모두 만들어진 것으로 볼 수 있습니다.

`Rotation Max : 10.0`은 Rotation이 선택된 이미지의 경우 **-10° ~ +10° 범위 안에서 회전값을 사용했다**는 뜻입니다.

그리고 생성된 838장의 이미지가 어떻게 만들어졌는지는 다음 파일에 기록되었습니다.

```
reports/apple/synthetic_metadata.csv
```

Metadata에는 실제로 다음 정보가 저장되어 있습니다. 붙여넣은 텍스트(1)

```
생성 이미지 파일명
어떤 Augmentation을 사용했는지
어떤 원본 이미지에서 만들었는지
원본 Hash
실제 사용한 Parameter
Seed
승인 상태
```

예를 들어 실제 Metadata의 한 행은 다음 의미입니다.

```
0_100__brightness__01.jpg
→ 원본: 0_100.jpg
→ 방법: brightness
→ 밝기 값: alpha=0.810
→ Seed: 42
→ approved: pending
```

여기서 `pending`은 **이미지는 생성되었지만 아직 QA를 거쳐 학습용 데이터로 승인된 것은 아니다**라는 의미입니다. 붙여넣은 텍스트(1)

따라서 현재 단계의 결과를 한 줄로 정리하면:

> 원본 Train 419장을 이용해 보강 이미지 838장을 정상 생성했고, 
> 각 이미지가 어떻게 만들어졌는지 Metadata까지 기록한 상태입니다. 
> 다음 단계에서는 이 보강 데이터가 실제 학습에 사용해도 되는지 QA로 확인합니다.

---

# 18. 생성된 이미지와 Metadata를 직접 확인합니다

앞에서 Train 419장으로부터 **838장의 Augmentation 이미지가 생성되었고**, 
생성 과정이 `synthetic_metadata.csv`에 기록된 것을 확인했습니다.

이제 실제 파일과 기록 내용을 간단히 확인합니다.

## 생성 이미지를 확인합니다

```bash
ls data/apple/augmented/train | head
```

예:

```
...__brightness__01.jpg
...__rotation__02.jpg
...__noise__01.jpg
```

파일명만 보아도 어떤 Augmentation으로 만들어졌는지 일부 확인할 수 있습니다.

```
brightness
→ 밝기 변화

rotation
→ 회전

blur
→ 흐림

noise
→ 노이즈
```

---

## Metadata를 확인합니다

```bash
head reports/apple/synthetic_metadata.csv
```

`Metadata`는 **생성된 이미지가 어떻게 만들어졌는지 기록한 정보**입니다.

주요 항목은 다음과 같습니다.

|항목|의미|
|---|---|
|`file`|생성된 이미지 파일명|
|`method`|적용한 Augmentation 방법|
|`source`|어떤 원본 이미지에서 만들었는지|
|`source_hash`|원본 파일을 구분하기 위한 SHA-256 값|
|`parameter`|실제 사용한 밝기·각도·Noise 등의 값|
|`seed`|같은 결과를 재현하기 위한 난수 기준값|
|`approved`|QA 검수 상태|

예를 들어 다음 기록이 있다면,

```
0_100__brightness__01.jpg
method     → brightness
source     → 0_100.jpg
parameter  → alpha=0.810
seed       → 42
approved   → pending
```

다음 의미입니다.

```
원본 0_100.jpg
        ↓
Brightness 적용
        ↓
alpha=0.810 사용
        ↓
새 이미지 생성
        ↓
아직 QA 전이므로 pending
```

현재 `approved`가 모두 `pending`인 것은 정상입니다.

```
pending
→ 생성은 완료됨
→ 아직 학습용으로 승인된 것은 아님
```

> 이미지를 생성했다고 바로 Train에 사용하는 것이 아니라, 생성 이력을 기록하고 QA를 거친 뒤 사용 여부를 판단합니다.

---

# PART 5. 이번에는 VAE가 Apple 이미지의 특징을 직접 학습하게 합니다

앞에서는 사람이 직접 변형 방법을 정했습니다.

```
원본 Apple 이미지
        ↓
사람이 변형 방법을 정함
        ↓
Brightness
Rotation
Blur
Noise
        ↓
새로운 Augmented Image
```

이것이 **Augmentation**이었습니다.

이제는 조금 다른 실험을 해봅니다.

이번에는 우리가

```
10도 돌려라
밝기를 80%로 바꿔라
Noise를 넣어라
```

처럼 변형 방법을 직접 정하지 않습니다.

대신 여러 Apple 이미지를 모델에게 보여주고,

> **“이 사과 이미지들에는 어떤 공통적인 특징이 있는지 네가 학습해 봐.”**

라고 하는 것입니다.

이때 사용할 모델이 **VAE(Variational Autoencoder)**입니다.

```
여러 Apple Train 이미지
        ↓
VAE가 반복해서 학습
        ↓
사과 이미지의 공통적인 특징을
숫자로 표현하는 방법을 학습
        ↓
그 숫자 표현을 이용해
이미지를 다시 만들어 봄
```

따라서 PART 5에서 확인하려는 것은 단순히 **이미지를 압축하는 것 자체**가 아닙니다.

```
VAE가 여러 사과 이미지를 학습했을 때

사과의 색
사과의 전체적인 형태
배경
윤곽과 같은 특징을

어떻게 숫자로 표현하고
다시 이미지로 만들어 내는가?
```

를 직접 확인하는 실험입니다.

---

## Augmentation과 VAE는 무엇이 다른가요?

두 실험은 같은 Apple Train 데이터에서 시작하지만 방법이 다릅니다. 교재에서도 두 실험을 연결해서 사용하는 것이 아니라, 같은 원본 Train에서 각각 별도로 수행하도록 설계되어 있습니다. 1일차_AppleRed1_Augmentation_VAE_…

```
                    Apple Train
                         ↓
             ┌───────────┴───────────┐
             ↓                       ↓

       Augmentation                  VAE

 사람이 변형 규칙을 정함       모델이 이미지 특징을 학습
             ↓                       ↓
 밝기·회전·Blur·Noise          특징을 숫자로 표현
             ↓                       ↓
 기존 이미지를 직접 변형        다시 이미지로 복원
                                     ↓
                              새로운 Sample도 생성
```

쉽게 비교하면 다음과 같습니다.

|Augmentation|VAE|
|---|---|
|사람이 어떻게 바꿀지 정함|모델이 데이터의 특징을 학습|
|기존 이미지를 직접 변형|학습한 특징에서 이미지를 다시 만듦|
|Rotation, Brightness 등|Encoder, Latent, Decoder 사용|

따라서 앞에서 만든 838장의 Augmented Image를 VAE에 넣는 것이 아닙니다.

이번 수업에서는 두 실험을 분리합니다.

```
실험 A — Augmentation

원본 Train
        ↓
Brightness / Rotation / Blur / Noise
        ↓
Augmented Image


실험 B — VAE

같은 원본 Train
        ↓
VAE가 이미지 특징 학습
        ↓
이미지 복원
        +
새로운 Sample 생성
```

이렇게 해야 **“사람이 직접 이미지를 변형하는 방법”과 “모델이 데이터의 특징을 학습하는 방법”의 차이**를 명확하게 볼 수 있습니다.

---

# 19. VAE에서는 무엇을 학습하는지 먼저 이해합니다

VAE에게 Apple 이미지 여러 장을 보여주면 모델은 이미지를 픽셀 그대로 외우는 것이 아니라, **이미지를 다시 만드는 데 필요한 특징을 작은 숫자 표현으로 바꾸는 방법**을 학습합니다.

예를 들어 사람이 사과를 보면 모든 픽셀 숫자를 외우지 않아도 다음과 같은 특징으로 기억할 수 있습니다.

```
빨간색 계열이다
둥근 형태이다
가운데에 사과가 있다
위쪽에 꼭지가 있다
배경이 비교적 단순하다
```

VAE도 사람처럼 이런 문장을 만드는 것은 아니지만, 여러 이미지에서 반복되는 특징을 **숫자로 표현하는 방법**을 학습합니다.

전체 흐름은 다음과 같습니다.

```
Apple 이미지
        ↓
Encoder
        ↓
이미지의 특징을
작은 숫자 표현으로 바꿈
        ↓
Latent Space
        ↓
Decoder
        ↓
다시 Apple 이미지 생성
        ↓
Reconstruction
```

여기서 중요한 용어가 세 가지입니다.

```
Encoder
→ 이미지를 보고 특징을 작은 표현으로 바꾸는 부분

Latent Space
→ 이미지 특징이 숫자로 표현되는 공간

Decoder
→ 그 숫자 표현을 이용하여
  다시 이미지를 만드는 부분
```

---

## Reconstruction은 무엇인가요?

`Reconstruction`은 **복원된 이미지**라는 뜻입니다.

VAE에 원본 사과를 넣고,

```
원본 이미지
        ↓
Encoder
        ↓
Latent
        ↓
Decoder
        ↓
복원 이미지
```

과정을 거친 뒤 나온 결과입니다.

따라서 우리는 다음 두 이미지를 비교하게 됩니다.

```
Original
vs
Reconstruction

원래 사과        VAE가 다시 만든 사과
```

이 비교를 통해 다음을 확인합니다.

```
빨간색이 유지되었는가?

사과의 전체적인 형태가 남아 있는가?

윤곽이나 작은 무늬는 얼마나 사라졌는가?

VAE가 원본의 중요한 특징을
어느 정도 학습했는가?
```

복원 이미지가 원본과 완전히 똑같을 필요는 없습니다.

이번 실습의 목적은 **사진을 완벽하게 복사하는 모델을 만드는 것이 아니라, 이미지의 특징을 작은 표현으로 바꾼 뒤 다시 복원하는 과정을 이해하는 것**입니다.

---

# 20. Encoder → Latent Space → Decoder를 조금 더 자세히 봅니다

이번 실습에서는 원본 Apple 이미지를 먼저 `64 × 64 RGB` 크기로 맞춥니다.

```
64 × 64 RGB Apple Image
        ↓
Encoder
        ↓
Latent Space
        ↓
Decoder
        ↓
64 × 64 RGB Reconstruction
```

## Encoder

Encoder는 큰 이미지 정보를 단계적으로 줄여 **중요한 특징을 표현할 수 있는 작은 정보**로 바꿉니다.

```
Apple Image
        ↓
Convolution
        ↓
Feature 추출
        ↓
더 작은 표현
```

쉽게 말하면:

> **“이 이미지를 다시 설명하는 데 필요한 핵심 정보를 줄여서 표현해 보자.”**

라는 역할입니다.

---

## Latent Space

이번 실습에서는 다음 값을 사용합니다.

```
latent_dim = 32
```

즉 Decoder에 전달할 사과 이미지의 특징을 **32차원의 Latent 값 `z`**로 표현해 보는 것입니다.

```
큰 이미지 정보
        ↓
Encoder
        ↓
32개의 Latent 값으로 표현
        ↓
Decoder
        ↓
이미지 복원
```

여기서 `32`가 사과의 특징이 정확히 32개라는 뜻은 아닙니다.

**모델이 이미지를 표현할 수 있도록 우리가 정한 작은 숫자 공간의 크기**라고 이해하면 충분합니다.

---

## VAE는 왜 그냥 Encoder가 아니라 `mu`와 `logvar`를 만드나요?

일반적인 Autoencoder는 한 이미지를 하나의 고정된 Latent 값으로 바꾸는 방식으로 생각할 수 있습니다.

VAE는 조금 다르게 **하나의 점만 외우기보다, 어느 정도 범위를 가진 분포로 특징을 학습**합니다.

오늘은 수학식을 외울 필요가 없습니다.

다음 흐름만 이해합니다.

```
Apple Image
        ↓
Encoder
        ↓
mu + logvar
        ↓
"이 근처에서 사과의 특징을 표현할 수 있다"
라는 분포를 학습
        ↓
그 분포에서 z를 Sampling
        ↓
Decoder
        ↓
Apple Image 생성
```

왜 이렇게 할까요?

이 구조 덕분에 VAE는 단순히 입력 이미지를 다시 복원하는 것뿐 아니라, 학습한 Latent 공간에서 값을 뽑아 **새로운 이미지도 만들어 볼 수 있기 때문**입니다.

---

## 이번 실험에서는 두 가지 결과를 봅니다

VAE 학습이 끝나면 크게 두 가지 결과를 확인합니다.

### ① Reconstruction

실제 이미지를 VAE에 넣고 다시 만들어 봅니다.

```
실제 Test Apple
        ↓
Encoder
        ↓
Latent
        ↓
Decoder
        ↓
Reconstruction
```

확인할 질문:

```
원본의 색과 형태가 얼마나 유지되는가?
```

### ② Random Sample

이번에는 특정 원본 이미지를 넣지 않습니다.

```
학습한 Latent Space
        ↓
Random z Sampling
        ↓
Decoder
        ↓
새로운 이미지
```

즉:

```
Reconstruction
→ 원본이 있음
→ "이 사과를 다시 만들어 봐"


Random Sample
→ 특정 원본이 없음
→ "학습한 사과 특징을 이용해
   새로운 이미지를 만들어 봐"
```

라는 차이가 있습니다.

단순한 VAE와 작은 교육용 데이터에서는 Random Sample이 흐릿하거나 실제 사과처럼 완벽하지 않을 수 있습니다. 이번 실습에서는 **이미지 품질 경쟁보다 ‘모델이 학습한 특징으로 이미지를 만들 수 있다’는 생성모델의 기본 구조를 이해하는 것**이 더 중요합니다.

---

# 21. 이제 VAE를 실제로 학습하는 코드를 작성합니다

앞에서 VAE가 무엇을 하려는 모델인지 이해했습니다.

이제 작성할 `05_train_vae.py`는 **Apple Train 이미지를 이용하여 VAE를 실제로 학습시키고, 학습 결과를 이미지로 확인하는 코드**입니다.

이번 코드는 다음 순서로 동작합니다.

```
[입력]

Train 419장
Validation 73장
Test 164장

        ↓

[학습 준비]

이미지를 64 × 64로 변환

        ↓

[VAE 학습]

Train 이미지
        ↓
Encoder
        ↓
Latent
        ↓
Decoder
        ↓
Reconstruction

        ↓

원본과 Reconstruction의 차이를 계산
        ↓
모델을 반복해서 학습

        ↓

[중간 확인]

Validation
→ Epoch마다 Loss 확인

        ↓

[학습 완료]

VAE Model 저장
Loss 기록 저장

        ↓

[결과 확인]

Test Original
vs
Test Reconstruction

        +

Random Latent Sample
```

데이터의 역할도 앞에서 정한 원칙을 그대로 유지합니다.

```
Train
→ VAE가 실제로 학습

Validation
→ 학습이 진행되는 동안
  상태와 조건을 확인

Test
→ 학습과 조건 선택이 끝난 뒤
  마지막 Reconstruction 확인
```

그리고 중요한 점이 하나 있습니다.

```
data/apple/augmented/train/
→ 이번 VAE 학습에는 사용하지 않음

data/apple/split/train/
→ VAE 학습에 사용
```

즉 이번 코드는 앞에서 생성한 Augmented Image를 학습시키는 코드가 아니라, **원본 Train 이미지를 이용하여 VAE 자체가 Apple 이미지의 특징을 학습하도록 만드는 별도의 실험**입니다.

따라서 `05_train_vae.py`의 목적을 한 문장으로 정리하면:

> Apple Train 이미지를 VAE에게 학습시켜 이미지 특징을 Latent Space에 표현하게 하고, 그 특징으로 원본을 얼마나 복원할 수 있는지와 새로운 Sample을 만들 수 있는지 확인하는 코드입니다.

이제 실제 코드를 작성합니다.

**파일: `scripts/05_train_vae.py`**

```python
from __future__ import annotations

import argparse
import random
import sys
from pathlib import Path

import numpy as np
import pandas as pd
import torch
import torch.nn as nn
import torch.nn.functional as F

from PIL import Image

from torch.utils.data import (
    DataLoader,
    Dataset,
)

from torchvision import transforms
from torchvision.utils import save_image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


IMAGE_SIZE = 64
BATCH_SIZE = 32
SEED = 42
BETA = 0.001


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--epochs",
        type=int,
        default=10,
    )

    parser.add_argument(
        "--latent",
        type=int,
        default=32,
    )

    parser.add_argument(
        "--eval-split",
        choices=["val", "test"],
        default="test",
        help=(
            "Reconstruction을 만들 Split. "
            "조건 비교는 val, 최종 확인은 test를 사용합니다."
        ),
    )

    parser.add_argument(
        "--eval-only",
        action="store_true",
        help=(
            "이미 학습된 동일 조건의 Model을 불러와 "
            "선택한 Split의 Reconstruction만 생성합니다."
        ),
    )

    return parser.parse_args()


def set_seed(seed: int) -> None:
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)

    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)


class FlatImageDataset(Dataset):
    def __init__(
        self,
        root_dir,
        image_size=64,
    ):
        self.files = list_images(
            Path(root_dir)
        )

        if not self.files:
            raise RuntimeError(
                f"No images found: {root_dir}"
            )

        self.transform = transforms.Compose(
            [
                transforms.Resize(
                    (image_size, image_size)
                ),
                transforms.ToTensor(),
            ]
        )

    def __len__(self):
        return len(self.files)

    def __getitem__(self, index):
        path = self.files[index]

        image = Image.open(
            path
        ).convert("RGB")

        return self.transform(
            image
        )


class VAE(nn.Module):
    def __init__(
        self,
        latent_dim=32,
    ):
        super().__init__()

        self.encoder = nn.Sequential(
            nn.Conv2d(
                3,
                32,
                4,
                2,
                1,
            ),
            nn.ReLU(),
            nn.Conv2d(
                32,
                64,
                4,
                2,
                1,
            ),
            nn.ReLU(),
            nn.Conv2d(
                64,
                128,
                4,
                2,
                1,
            ),
            nn.ReLU(),
        )

        self.fc_mu = nn.Linear(
            128 * 8 * 8,
            latent_dim,
        )

        self.fc_logvar = nn.Linear(
            128 * 8 * 8,
            latent_dim,
        )

        self.fc_decode = nn.Linear(
            latent_dim,
            128 * 8 * 8,
        )

        self.decoder = nn.Sequential(
            nn.ConvTranspose2d(
                128,
                64,
                4,
                2,
                1,
            ),
            nn.ReLU(),
            nn.ConvTranspose2d(
                64,
                32,
                4,
                2,
                1,
            ),
            nn.ReLU(),
            nn.ConvTranspose2d(
                32,
                3,
                4,
                2,
                1,
            ),
            nn.Sigmoid(),
        )

    def encode(self, x):
        feature = self.encoder(x)

        feature = feature.view(
            feature.size(0),
            -1,
        )

        mu = self.fc_mu(
            feature
        )

        logvar = self.fc_logvar(
            feature
        )

        return mu, logvar

    def reparameterize(
        self,
        mu,
        logvar,
    ):
        std = torch.exp(
            0.5 * logvar
        )

        eps = torch.randn_like(
            std
        )

        return (
            mu
            + eps * std
        )

    def decode(self, z):
        feature = self.fc_decode(
            z
        )

        feature = feature.view(
            -1,
            128,
            8,
            8,
        )

        return self.decoder(
            feature
        )

    def forward(self, x):
        mu, logvar = self.encode(
            x
        )

        z = self.reparameterize(
            mu,
            logvar,
        )

        reconstruction = self.decode(
            z
        )

        return (
            reconstruction,
            mu,
            logvar,
        )


def vae_loss(
    reconstruction,
    original,
    mu,
    logvar,
):
    reconstruction_loss = F.mse_loss(
        reconstruction,
        original,
        reduction="mean",
    )

    kl_loss = -0.5 * torch.mean(
        1
        + logvar
        - mu.pow(2)
        - logvar.exp()
    )

    total_loss = (
        reconstruction_loss
        + BETA * kl_loss
    )

    return (
        total_loss,
        reconstruction_loss,
        kl_loss,
    )


def evaluate(
    model,
    loader,
    device,
):
    model.eval()

    total = 0.0
    reconstruction_total = 0.0
    kl_total = 0.0
    count = 0

    with torch.no_grad():
        for images in loader:
            images = images.to(
                device
            )

            reconstruction, mu, logvar = (
                model(images)
            )

            loss, recon_loss, kl_loss = (
                vae_loss(
                    reconstruction,
                    images,
                    mu,
                    logvar,
                )
            )

            batch_size = images.size(0)

            total += (
                loss.item()
                * batch_size
            )

            reconstruction_total += (
                recon_loss.item()
                * batch_size
            )

            kl_total += (
                kl_loss.item()
                * batch_size
            )

            count += batch_size

    if count == 0:
        raise RuntimeError(
            "Validation 데이터가 없습니다."
        )

    return (
        total / count,
        reconstruction_total / count,
        kl_total / count,
    )


def main() -> None:
    args = parse_args()

    if args.epochs < 1:
        raise ValueError(
            "--epochs는 1 이상이어야 합니다."
        )

    if args.latent < 1:
        raise ValueError(
            "--latent는 1 이상이어야 합니다."
        )

    set_seed(SEED)

    paths = dataset_paths(
        args.dataset
    )

    train_dataset = FlatImageDataset(
        paths["split_train"],
        image_size=IMAGE_SIZE,
    )

    val_dataset = FlatImageDataset(
        paths["split_val"],
        image_size=IMAGE_SIZE,
    )

    test_dataset = FlatImageDataset(
        paths["split_test"],
        image_size=IMAGE_SIZE,
    )

    generator = torch.Generator()
    generator.manual_seed(SEED)

    train_loader = DataLoader(
        train_dataset,
        batch_size=BATCH_SIZE,
        shuffle=True,
        generator=generator,
    )

    val_loader = DataLoader(
        val_dataset,
        batch_size=BATCH_SIZE,
        shuffle=False,
    )

    test_loader = DataLoader(
        test_dataset,
        batch_size=BATCH_SIZE,
        shuffle=False,
    )

    eval_loader = (
        val_loader
        if args.eval_split == "val"
        else test_loader
    )

    device = torch.device(
        "cuda"
        if torch.cuda.is_available()
        else "cpu"
    )

    print("Device     :", device)
    print("Train      :", len(train_dataset))
    print("Val        :", len(val_dataset))
    print("Test       :", len(test_dataset))
    print("Eval Split :", args.eval_split)
    print("Eval Only  :", args.eval_only)

    model = VAE(
        latent_dim=args.latent
    ).to(device)

    model_path = (
        paths["model"]
        / (
            f"vae_latent"
            f"{args.latent}"
            f"_e{args.epochs}.pt"
        )
    )

    loss_path = (
        paths["report"]
        / (
            f"vae_loss_latent"
            f"{args.latent}"
            f"_e{args.epochs}.csv"
        )
    )

    if args.eval_only:
        if not model_path.exists():
            raise FileNotFoundError(
                "먼저 같은 Epoch / Latent 조건으로 "
                f"학습해야 합니다: {model_path}"
            )

        state_dict = torch.load(
            model_path,
            map_location=device,
        )

        model.load_state_dict(
            state_dict
        )

        print(
            "Loaded Model:",
            model_path,
        )

    else:
        optimizer = torch.optim.Adam(
            model.parameters(),
            lr=1e-3,
        )

        history = []

        for epoch in range(
            1,
            args.epochs + 1,
        ):
            model.train()

            train_total = 0.0
            train_reconstruction = 0.0
            train_kl = 0.0
            train_count = 0

            for images in train_loader:
                images = images.to(
                    device
                )

                optimizer.zero_grad()

                reconstruction, mu, logvar = (
                    model(images)
                )

                loss, recon_loss, kl_loss = (
                    vae_loss(
                        reconstruction,
                        images,
                        mu,
                        logvar,
                    )
                )

                loss.backward()
                optimizer.step()

                batch_size = images.size(0)

                train_total += (
                    loss.item()
                    * batch_size
                )

                train_reconstruction += (
                    recon_loss.item()
                    * batch_size
                )

                train_kl += (
                    kl_loss.item()
                    * batch_size
                )

                train_count += batch_size

            train_loss = (
                train_total
                / train_count
            )

            train_recon = (
                train_reconstruction
                / train_count
            )

            train_kl_loss = (
                train_kl
                / train_count
            )

            (
                val_loss,
                val_recon,
                val_kl,
            ) = evaluate(
                model,
                val_loader,
                device,
            )

            history.append(
                {
                    "epoch": epoch,
                    "train_loss": train_loss,
                    "train_recon": train_recon,
                    "train_kl": train_kl_loss,
                    "val_loss": val_loss,
                    "val_recon": val_recon,
                    "val_kl": val_kl,
                }
            )

            print(
                f"Epoch "
                f"{epoch:02d}/"
                f"{args.epochs} "
                f"Train={train_loss:.6f} "
                f"Val={val_loss:.6f}"
            )

        torch.save(
            model.state_dict(),
            model_path,
        )

        pd.DataFrame(
            history
        ).to_csv(
            loss_path,
            index=False,
            encoding="utf-8-sig",
        )

    model.eval()

    with torch.no_grad():
        sample = next(
            iter(eval_loader)
        ).to(device)

        mu, _ = model.encode(
            sample
        )

        reconstruction = model.decode(
            mu
        )

        count = min(
            8,
            sample.size(0),
        )

        comparison = torch.cat(
            [
                sample[:count],
                reconstruction[:count],
            ],
            dim=0,
        )

        reconstruction_path = (
            paths["result_vae"]
            / (
                f"reconstruction_"
                f"{args.eval_split}_"
                f"latent{args.latent}"
                f"_e{args.epochs}.png"
            )
        )

        save_image(
            comparison,
            reconstruction_path,
            nrow=count,
        )

        generated_path = None

        if not args.eval_only:
            random_z = torch.randn(
                16,
                args.latent,
                device=device,
            )

            generated = model.decode(
                random_z
            )

            generated_path = (
                paths["result_vae"]
                / (
                    f"generated_samples_"
                    f"latent{args.latent}"
                    f"_e{args.epochs}.png"
                )
            )

            save_image(
                generated,
                generated_path,
                nrow=4,
            )

    print()
    print("Model          :", model_path)

    if loss_path.exists():
        print("Loss           :", loss_path)

    print(
        "Reconstruction :",
        reconstruction_path,
    )

    if generated_path is not None:
        print(
            "Random Samples :",
            generated_path,
        )

if __name__ == "__main__":
    main()
```

### 1. 전체 코드를 빠르게 이해하는 의사코드

```
apple 또는 banana를 선택한다

학습 Epoch 수를 정한다
→ 기본값 10

Latent Dimension을 정한다
→ 기본값 32

Reconstruction을 확인할 데이터 선택
→ val 또는 test

        ↓

Seed를 42로 고정한다
→ 같은 조건의 실험을 다시 재현할 수 있게 함

        ↓

Train / Validation / Test 이미지를 불러온다

        ↓

모든 이미지를
64 × 64 RGB Tensor로 변환한다

        ↓

VAE 모델을 만든다

Encoder
→ 이미지 Feature 추출

mu / logvar
→ Latent 분포 표현

z Sampling
→ 분포에서 Latent 값 선택

Decoder
→ Latent에서 이미지 복원

        ↓

VAE Loss 계산

Reconstruction Loss
→ 원본과 복원 이미지가 얼마나 다른지

KL Loss
→ Latent 분포가 너무 제멋대로 되지 않도록 조정

        ↓

Train 이미지로 학습

이미지 입력
→ VAE 통과
→ Loss 계산
→ 역전파
→ 모델 Parameter 수정

        ↓

한 Epoch가 끝날 때마다

Validation 데이터로
Loss 확인

        ↓

모든 Epoch 학습 완료

모델 저장
Loss 기록 CSV 저장

        ↓

Validation 또는 Test 이미지 일부 선택

        ↓

Encoder
→ mu 생성

        ↓

Decoder
→ Reconstruction 생성

        ↓

Original + Reconstruction
비교 이미지 저장

        ↓

일반 학습 모드라면

Random Latent z 생성
        ↓
Decoder에 입력
        ↓
새로운 Sample 16장 생성

        ↓

모델 / Loss / Reconstruction /
Random Sample 저장 위치 출력
```

더 핵심만 줄이면 다음과 같습니다.

```
Train 이미지
        ↓
Encoder
        ↓
Latent
        ↓
Decoder
        ↓
Reconstruction
        ↓
원본과 비교하여 Loss 계산
        ↓
모델 학습
        ↓
Validation으로 학습 상태 확인
        ↓
모델 저장
        ↓
Test Reconstruction 확인
        +
Random Sample 생성
```

`05_train_vae.py`에는 **입력 준비 + 모델 정의 + 학습 + 평가 + 결과 저장**이 
한 파일에 다 들어 있습니다.

이 긴 코드를 한 줄씩 전부 해석하기보다, 먼저 **5개 덩어리**로 나눠서 이해합니다.

- **입력 준비**: Train/Val/Test 이미지를 읽고 64×64 Tensor로 만듦
- **모델 정의**: Encoder → Latent → Decoder 구조를 만듦
- **Loss 정의**: 원본과 복원 이미지 차이 + KL Loss 계산
- **학습 처리**: Train으로 반복 학습하고 Validation Loss 확인
- **출력 처리**: 모델, Loss CSV, Reconstruction, Random Sample 저장

즉 아주 단순하게 보면:

```
[입력]

Train / Validation / Test 이미지
        ↓

[처리 1]

이미지를 64 × 64 Tensor로 변환
        ↓

[처리 2]

VAE
Encoder
→ Latent
→ Decoder
        ↓

[처리 3]

Loss 계산
→ backward()
→ optimizer.step()
→ 반복 학습
        ↓

[출력]

학습된 Model
Loss CSV
Reconstruction
Random Sample
```

**입력·처리·출력이 모두 한 파일 안에 들어 있는 구조**라고 보면 됩니다.

```
이 코드는 크게 3가지만 합니다.

1. 사과 이미지를 준비한다.
2. VAE에게 사과 이미지를 학습시킨다.
3. 학습이 끝난 뒤 결과 이미지를 저장한다.
```

그다음 세부 구조는 다음과 같습니다.

```
이미지 준비
        ↓
VAE 모델
        ↓
Loss 계산
        ↓
반복 학습
        ↓
결과 저장
```

그리고 코드도 아래와 같이 코드 분리를 할수 있습니다.

```
dataset.py
→ 이미지 읽기 / 전처리

vae_model.py
→ Encoder / Latent / Decoder

train_vae.py
→ Loss / 학습 반복

evaluate_vae.py
→ Reconstruction / Random Sample

05_train_vae.py
→ 위 기능들을 호출해서 전체 실행
```

핵심만 보면:

```
VAE 코드는 길어 보이지만
실제로 하는 일은 다섯 단계입니다.

데이터 준비
→ 모델 만들기
→ Loss 계산
→ 학습
→ 결과 저장
```

### 이 코드는 왜 작성하나요?

`05_train_vae.py`는 **원본 Train 이미지로 VAE를 학습하고, Validation Loss와 Reconstruction 결과를 확인하는 코드**입니다.

앞에서 만든 Augmented Image는 사용하지 않으며, 
이번에는 **VAE가 원본 Apple 이미지의 특징을 어떻게 학습하고 다시 복원하는지** 확인합니다.

---

# 22. VAE를 실제로 학습하고 결과를 확인합니다

앞에서 만든 `05_train_vae.py`를 기본 조건으로 실행합니다.

이번 실험 조건은 다음과 같습니다.

```
Dataset
→ Apple Red 1

Epoch
→ 10

Latent Dimension
→ 32

마지막 Reconstruction 확인 데이터
→ Test
```

실행합니다.

```bash
python scripts/05_train_vae.py \
  --dataset apple \
  --epochs 10 \
  --latent 32 \
  --eval-split test
```

실제 실행 결과는 다음과 같습니다.

```text
Device     : cuda
Train      : 419
Val        : 73
Test       : 164
Eval Split : test
Eval Only  : False

Epoch 01/10 Train=0.106142 Val=0.088216
Epoch 02/10 Train=0.072268 Val=0.043418
Epoch 03/10 Train=0.034429 Val=0.028831
Epoch 04/10 Train=0.026849 Val=0.024659
Epoch 05/10 Train=0.022076 Val=0.018257
Epoch 06/10 Train=0.016229 Val=0.014490
Epoch 07/10 Train=0.013675 Val=0.012342
Epoch 08/10 Train=0.011868 Val=0.011160
Epoch 09/10 Train=0.010441 Val=0.009440
Epoch 10/10 Train=0.009187 Val=0.008591
```

먼저 위쪽 결과부터 읽어봅니다.

```text
Device : cuda
→ GPU를 사용하여 VAE를 학습함

Train : 419
→ 실제 VAE 학습에 사용한 이미지 419장

Val : 73
→ Epoch마다 학습 상태를 확인한 이미지 73장

Test : 164
→ 학습에 사용하지 않고 마지막 결과 확인에 사용

Eval Split : test
→ 학습 완료 후 Test 이미지로 Reconstruction을 만듦

Eval Only : False
→ 기존 모델을 불러온 것이 아니라
  이번 실행에서 실제로 VAE를 새로 학습함
```

즉 이번 실행은 다음 작업을 한 것입니다.

```
Apple Train 419장
        ↓
VAE 학습
10 Epoch 반복
        ↓
Validation 73장으로
매 Epoch 상태 확인
        ↓
학습 완료
        ↓
Test 164장에서
Reconstruction 확인
        +
Random Sample 생성
```

학습이 끝난 뒤 다음 파일들이 저장되었습니다.

```
models/apple/
└─ vae_latent32_e10.pt
   → 학습된 VAE 모델

reports/apple/
└─ vae_loss_latent32_e10.csv
   → Epoch별 Loss 기록

results/apple/vae/
├─ reconstruction_test_latent32_e10.png
│  → Test 원본과 VAE 복원 결과 비교
│
└─ generated_samples_latent32_e10.png
   → 원본 없이 Latent에서 만든 16개 Sample
```

> 즉 22번에서는 **VAE 학습이 정상적으로 끝났고, 
> 모델·Loss 기록·복원 이미지·생성 이미지까지 모두 만들어졌는지 확인**합니다.

---

# 23. 실제 Loss를 보고 VAE가 학습되었는지 확인합니다

Loss 기록을 확인합니다.

```bash
cat reports/apple/vae_loss_latent32_e10.csv
```

여러 숫자가 나오지만 처음에는 `train_loss`와 `val_loss` 두 값만 보면 됩니다.

실제 결과를 처음과 마지막 Epoch만 비교해 보겠습니다.

|Epoch|Train Loss|Validation Loss|
|---|---|---|
|1|0.106142|0.088216|
|5|0.022076|0.018257|
|10|0.009187|0.008591|

변화를 보면 다음과 같습니다.

```
Epoch 1

Train Loss ≈ 0.106
Val Loss   ≈ 0.088

        ↓

Epoch 10

Train Loss ≈ 0.009
Val Loss   ≈ 0.009
```

두 Loss가 학습이 진행되면서 계속 낮아졌습니다.

쉽게 말하면:

```
처음

VAE가 Apple 이미지를
아직 잘 복원하지 못함
        ↓
Loss가 큼


학습 반복
        ↓


10 Epoch

원본과 복원 이미지의 차이가
처음보다 많이 줄어듦
        ↓
Loss가 작아짐
```

따라서 이번 결과에서는 **VAE가 학습하면서 Apple 이미지를 복원하는 방법을 
점점 익히고 있다**고 볼 수 있습니다.

또한 Train Loss만 내려간 것이 아니라 Validation Loss도 함께 내려갔습니다.

```
Train Loss ↓
+
Validation Loss ↓
```

따라서 현재 10 Epoch 결과에서는 **Train 데이터만 외우고 Validation에서 갑자기 나빠지는 뚜렷한 현상은 보이지 않습니다.**

Validation Loss가 Train Loss보다 약간 낮은 것도 이 결과만으로 문제라고 판단하지 않습니다.

---

## CSV의 다른 값은 무엇인가요?

CSV에는 다음 값도 있습니다.

```
train_recon
val_recon

train_kl
val_kl
```

VAE의 Loss는 크게 두 부분으로 만들어집니다.

```
Reconstruction Loss
        +
0.001 × KL Loss
        ↓
Total Loss
```

예를 들어 마지막 Epoch의 Train 결과는 다음과 같습니다.

```
train_recon
→ 0.007741

train_kl
→ 1.446137

BETA
→ 0.001
```

따라서 대략:

```
0.007741
+
0.001 × 1.446137
        ↓
약 0.009187
```

이 값이 화면에 출력된 `Train Loss`입니다.

우선 다음 정도만 기억하면 됩니다.

```
Reconstruction Loss
→ 원본과 복원 이미지의 차이

KL Loss
→ Latent Space가 너무 불규칙하게
  만들어지지 않도록 조절

Total Loss
→ 두 값을 함께 고려한
  VAE의 최종 학습 기준
```

> Loss가 낮아졌다는 사실만으로 좋은 생성모델이라고 결정하지는 않습니다. 
> 실제 Reconstruction과 생성 이미지도 반드시 눈으로 확인해야 합니다.

---

# 24. 실제 출력 이미지를 보고 VAE 결과를 확인합니다

23번에서는 Loss가 
학습하면서 낮아지는 것을 확인했습니다.

하지만 Loss 숫자만 보고 VAE가 잘 학습되었다고 판단할 수는 없습니다.

이번에는 VAE가 실제로 만든 이미지를 직접 확인합니다.

오늘 확인할 결과는 두 가지입니다.

```text
① Reconstruction
→ 실제 Test 이미지를 넣고 다시 복원한 결과

② Random Sample
→ 실제 이미지를 넣지 않고
  Random Latent 값에서 새로 만든 결과
```

두 결과는 의미가 다르므로 따로 확인합니다.

---

## 1. Original과 Reconstruction을 비교합니다

먼저 다음 파일을 엽니다.

```
results/apple/vae/
└─ reconstruction_test_latent32_e10.png
```

### 실제 출력 결과

![[Pasted image 20261007165751.png]]

이 이미지는 다음 구조로 저장되어 있습니다.

```
위쪽 8장
→ 실제 Test Apple 이미지

아래쪽 8장
→ VAE가 다시 만든 Reconstruction
```

즉 VAE가 한 일은 다음과 같습니다.

```
실제 Test Apple
        ↓
Encoder
        ↓
이미지 특징을 Latent로 표현
        ↓
Decoder
        ↓
Apple 이미지 다시 만들기
```

쉽게 말하면 VAE에게

> **“이 사과 이미지를 보고 특징을 파악한 다음 다시 만들어 봐.”**

라고 한 결과입니다.

### 무엇을 비교하면 되나요?

위쪽 원본과 아래쪽 복원 이미지를 같은 위치끼리 비교합니다.

```
원본의 빨간색이 유지되는가?

사과의 둥근 형태가 유지되는가?

사과라는 것을 알아볼 수 있는가?

작은 무늬와 윤곽은 얼마나 사라졌는가?

복원 이미지가 얼마나 흐려졌는가?
```

Reconstruction이 원본과 완전히 똑같을 필요는 없습니다.

![[Pasted image 20261007170019.png]]

이번 실습에서는 다음을 확인하는 것이 중요합니다.

```
원본 이미지
        ↓
작은 Latent 표현
        ↓
다시 이미지로 복원

이 과정에서도
사과의 주요 특징이 어느 정도 남아 있는가?
```

> Reconstruction은 **새로운 이미지를 마음대로 만든 결과가 아니라, 실제 입력 이미지를 VAE가 다시 복원한 결과**입니다.

---

## 2. Random Sample 결과를 확인합니다

이번에는 다음 파일을 엽니다.

```
results/apple/vae/
└─ generated_samples_latent32_e10.png
```

### 실제 출력 결과

![[Pasted image 20261007165108.png]]

현재 출력된 결과는 16장의 이미지를 `4 × 4`로 배치한 것입니다.

이 이미지는 앞의 Reconstruction과 만드는 방법이 다릅니다.

```
실제 Apple 이미지 입력
        ✕
사용하지 않음

Random Latent z 16개
        ↓
학습된 Decoder
        ↓
새로운 이미지 16장
```

![[Pasted image 20261007170357.png]]

### Random Sample의 원리를 순서대로 보면

#### 1. VAE는 먼저 Apple 이미지로 학습했습니다

학습할 때는 실제 사과 이미지가 들어갑니다.

```
Apple Train 이미지
        ↓
Encoder
        ↓
Latent z
        ↓
Decoder
        ↓
사과 이미지 복원
```

이 과정을 10 Epoch 반복하면서 Decoder도 학습합니다.

Decoder는 점점 이런 관계를 배우게 됩니다.

```
어떤 Latent 숫자 조합
        ↓
붉은색이 강한 이미지

다른 Latent 숫자 조합
        ↓
조금 둥근 이미지

또 다른 숫자 조합
        ↓
사과와 비슷한 형태
```

물론 실제로 `z1 = 빨간색`, `z2 = 둥근 정도`처럼 사람이 해석할 수 있게 정확히 
나뉘는 것은 아닙니다.

핵심은 **Decoder가 숫자 벡터를 이미지로 바꾸는 방법을 학습했다**는 것입니다.

---

#### 2. Random Sample에서는 이미지 대신 숫자를 넣습니다

여기가 핵심입니다.

Random Sample은 입력이 없는 것이 아닙니다.

**이미지 입력이 없을 뿐이고, 숫자 입력 `z`가 있습니다.**

코드에서는 이것입니다.

```python
random_z = torch.randn(
    16,
    args.latent,
    device=device,
)
```

현재 `latent = 32`이므로 쉽게 보면:

```
1번 z
[ 0.32, -0.75, 1.12, ..., 0.28 ]
→ 숫자 32개

2번 z
[-0.14,  0.91, 0.43, ..., -1.02]
→ 숫자 32개

3번 z
[ 1.20,  0.16, -0.81, ..., 0.55]
→ 숫자 32개

...

16번 z
→ 숫자 32개
```

이런 **32개의 랜덤 숫자 묶음을 16개 만든 것**입니다.

그리고 각각을 Decoder에 넣습니다.

```
Random 숫자 32개
        ↓
Decoder
        ↓
64 × 64 RGB 이미지
```

그래서 16개의 `z`를 넣었기 때문에 지금 보이는 **16장의 이미지**가 만들어진 것입니다.

---

### 3. 그런데 아무 숫자나 넣는데 왜 사과처럼 나오나요?

여기서 가장 중요한 질문입니다.

Decoder는 아무것도 모르는 상태가 아닙니다.

이미 앞에서 **Apple Train 419장으로 학습된 Decoder**입니다.

```
학습 전 Decoder

랜덤 숫자
        ↓
?
        ↓
의미 없는 Noise


Apple 419장으로 학습

        ↓

학습된 Decoder

Latent 숫자
        ↓
Apple 데이터에서 배운 패턴을 이용
        ↓
붉고 둥근 Apple 비슷한 이미지
```

즉 Random Sample을 만들 때 **사과 사진을 입력하지는 않지만**, 
Decoder 안에는 학습 과정에서 배운 Parameter가 이미 들어 있습니다.

쉽게 비유하면 이렇습니다.

```
화가에게 사과 사진 419장을 보여주며
계속 연습시킴

        ↓

화가가
"사과는 대체로 이런 색과 형태구나"
를 학습

        ↓

이제 특정 사과 사진을 보여주지 않고

"네가 배운 범위 안에서
사과 비슷한 것을 하나 그려봐"

        ↓

새로운 사과 비슷한 그림
```

Random Sample은 이것에 가깝습니다.

---

### 4. 그럼 랜덤 숫자는 도대체 어떤 범위에서 뽑나요?

완전히 아무 숫자나 막 넣는 것도 아닙니다.

VAE는 학습하면서 Latent 값들이 대체로 **표준정규분포** 근처에 있도록 유도합니다.

쉽게 말하면:

```
Latent Space

            많이 사용하는 영역
                  ↓
          · · · · · ·
       · · · · · · · · ·
      · · · · · · · · · ·
       · · · · · · · · ·
          · · · · ·
```

Random Sample에서는 이 학습된 Latent 공간과 맞도록 다음 분포에서 숫자를 뽑습니다.

```
평균 = 0
표준편차 = 1

        ↓

표준정규분포
N(0, 1)

        ↓

Random z
```

코드의:

```python
torch.randn(...)
```

이 바로 그 숫자를 만드는 것입니다.

따라서 정확한 흐름은:

```
Apple 419장으로 VAE 학습
        ↓
Latent가 일정한 분포를 갖도록 학습
        ↓
Decoder도
Latent → Apple 이미지
관계를 학습
        ↓

학습 완료

        ↓

표준정규분포에서
Random z 32개 추출
        ↓
학습된 Decoder
        ↓
Apple과 비슷한 새 이미지
```

입니다.

---

### 5. Reconstruction과 완전히 다른 점

이 둘을 구분하면 거의 다 이해한 것입니다.

```
Reconstruction

실제 사과 이미지
        ↓
Encoder
        ↓
"이 사과"에서 나온 z
        ↓
Decoder
        ↓
"이 사과"와 비슷하게 복원
```

반면:

```
Random Sample

실제 사과 이미지 없음

표준정규분포
        ↓
Random z
        ↓
Decoder
        ↓
학습한 Apple 패턴을 이용하여
새 이미지 생성
```

따라서 질문하신:

> “Reconstruction의 데이터 값을 랜덤하게 넣어서 그리는 거야?”

는 **아닙니다.**

정확히는:

> **Reconstruction에서 사용했던 특정 사과의 Latent 값을 섞는 것이 아니라, 표준정규분포에서 새로운 Latent 숫자를 직접 뽑아 Decoder에 넣는 것입니다.**

---

### 6. 그럼 이걸 왜 하나요?

이 실험의 목적은 **VAE가 단순히 입력 이미지를 복사하는 모델인지, 아니면 학습한 Latent 공간에서 새로운 이미지를 만들어낼 수도 있는 생성모델인지 확인하는 것**입니다.

```
Reconstruction만 확인

→ "입력 이미지를 다시 만들 수 있구나"


Random Sample까지 확인

→ "특정 입력이 없어도
   학습한 Latent 공간에서
   새로운 이미지를 만들 수 있구나"
```

그래서 Random Sample을 확인합니다.

현재 나온 이미지를 보면 아주 선명한 사과는 아니지만,

```
붉은색
+
둥근 덩어리
+
밝은 배경
```

같은 Apple 데이터의 큰 특징이 반복됩니다.

즉 이번 결과는:

> **Decoder가 Apple Train에서 배운 패턴을 이용해, 처음 보는 랜덤 Latent 숫자에서도 사과와 비슷한 이미지를 만들어 보았다.**

라고 이해하면 됩니다.

교재에서는 이 부분을 아주 짧게 다음처럼 설명해도 충분합니다.

```
Random Sample은 입력 이미지가 없는 것이 아니라
"이미지 대신 Random Latent 숫자"를 입력합니다.

Apple Train으로 학습된 Decoder는
이 숫자를 보고 학습했던 Apple의 색·형태·배경 패턴을 이용하여
새로운 이미지를 계산해서 만듭니다.

Reconstruction의 값을 섞는 것이 아닙니다.

표준정규분포에서 새로운 z를 뽑아
학습된 Decoder에 넣어 보는 실험입니다.
```

이 설명이 들어가야 초보자가 **“이미지도 안 넣었는데 왜 사과가 나오지?”**라는 핵심 의문을 해결할 수 있습니다.


즉 특정 원본 사과를 다시 만든 것이 아닙니다.

VAE가 학습한 Latent Space에서 임의의 값을 뽑고, 그 값을 Decoder에 넣어 이미지를 만든 것입니다.

---

## 3. 실제 Random Sample을 해석해 봅니다

실제 결과를 보면 이미지가 사진처럼 선명하지는 않습니다.

하지만 여러 Sample에서 다음과 같은 공통적인 모습이 나타납니다.

```
붉은색 계열
        +
둥근 물체 형태
        +
밝은 배경
```

실제 출력 이미지를 보면 16장의 결과 대부분에서 **붉은색과 둥근 형태가 반복적으로 나타나지만, 윤곽이나 세부 무늬는 상당히 흐릿합니다.**

이것을 다음처럼 이해하면 됩니다.

```
완전히 무작위 Noise만 생성
        ✕

Apple Train에서 보았던
색·형태 등의 특징이 일부 나타남
        ○

사진처럼 선명한 Apple 생성
        ✕
```

즉 이번 결과는

> **VAE가 Apple 이미지에서 반복되는 특징을 어느 정도 학습했지만, 10 Epoch의 간단한 모델만으로는 세부적인 형태까지 선명하게 생성하지 못한 상태**

라고 해석할 수 있습니다.

---

## 4. Reconstruction과 Random Sample을 구분합니다

두 결과가 헷갈리지 않도록 마지막으로 비교합니다.

|구분|Reconstruction|Random Sample|
|---|---|---|
|실제 이미지 입력|있음|없음|
|Encoder 사용|사용|사용하지 않음|
|Latent 값|입력 이미지에서 얻음|Random하게 생성|
|Decoder 사용|사용|사용|
|목적|원본을 얼마나 복원하는지 확인|학습한 특징으로 새 이미지를 만들 수 있는지 확인|

흐름으로 비교하면 다음과 같습니다.

```
Reconstruction

실제 Test Apple
        ↓
Encoder
        ↓
Latent
        ↓
Decoder
        ↓
복원 Apple
```

```
Random Sample

Random Latent z
        ↓
Decoder
        ↓
새로운 Apple 형태
```

---

## 5. 이번 결과에서 무엇을 배웠나요?

23번에서 Loss는 다음처럼 낮아졌습니다.

```
Train Loss
0.106142
        ↓
0.009187

Validation Loss
0.088216
        ↓
0.008591
```

그리고 24번에서는 실제 이미지 결과까지 확인했습니다.

```
Loss 감소
        ↓
숫자로 학습 진행 확인

        +

Reconstruction
        ↓
원본 특징을 얼마나 복원하는지 확인

        +

Random Sample
        ↓
Latent에서 이미지를 만들 수 있는지 확인
```

따라서 이번 VAE 실험의 결론은 다음과 같습니다.

> **VAE가 Apple 이미지의 특징을 Latent Space에 표현하는 방법을 학습했고, 그 정보를 이용해 원본 이미지를 복원하거나 새로운 Sample을 만들 수 있다는 기본 원리를 확인했습니다.**

다만 현재 Random Sample이 흐릿하므로,

```
"선명한 Apple 생성에 성공했다"
```

라고 해석하는 것은 적절하지 않습니다.

1일차의 목적은 이미지 품질 경쟁이 아니라 다음 원리를 직접 확인하는 것입니다.

```
이미지 학습
        ↓
Encoder
        ↓
Latent Space
        ↓
Decoder
        ↓
복원 또는 생성
```

이제 다음 단계에서는 **사람이 직접 이미지를 변형했던 Augmentation과, 데이터의 특징을 학습한 VAE가 어떻게 다른지 최종 정리**합니다.

---

# 25. 그러면 Augmentation과 VAE에서 실제로 무엇이 달랐나요?

이제 앞에서 실행한 두 실험의 결과를 실제로 비교해 봅니다.

## Augmentation에서는

우리가 직접 규칙을 정했습니다.

```
원본 Apple
        ↓
사람이 규칙 지정

Brightness
Rotation
Blur
Noise
        ↓
변형 Apple 이미지
```

예를 들어:

```
"이 이미지를 조금 어둡게 해라"

"10도 안에서 회전시켜라"

"Noise를 조금 넣어라"
```

처럼 **사람이 어떻게 바꿀지 알고 있습니다.**

---

## VAE에서는

우리가 사과의 특징을 직접 정하지 않았습니다.

```
Apple Train 419장
        ↓
VAE가 반복 학습
        ↓
Encoder
        ↓
Latent Space
        ↓
Decoder
        ↓
Reconstruction
또는
Random Sample
```

우리는 VAE에게:

```
"사과는 빨간색이어야 한다"

"둥글게 만들어라"

"배경은 흰색으로 만들어라"
```

라고 직접 코딩하지 않았습니다.

대신 여러 Apple 이미지를 보여주고 **그 안에서 반복되는 특징을 모델이 학습하도록 한 것**입니다.

이번 실제 결과에서도 Random Sample에:

```
붉은색 계열
   +
둥근 형태
   +
밝은 배경
```

이 반복적으로 나타나는 것을 확인할 수 있습니다.

따라서 1일차에서 가장 중요한 차이는 다음입니다.

```
Augmentation

사람이
"어떻게 바꿀지"
정한다

        ↓

기존 이미지를 변형


VAE

모델이
"데이터에는 어떤 특징이 있는지"
학습한다

        ↓

학습한 특징을 이용해
이미지를 복원하거나 생성
```

---

## 1일차 VAE 실험의 최종 결과

이번 실험을 한 줄로 정리하면 다음과 같습니다.

![[Pasted image 20261008074844.png]]

```
Apple Train 419장
        ↓
VAE 10 Epoch 학습
        ↓
Train 전체 Loss
0.106142 → 0.009187


Validation 전체 Loss
0.088216 → 0.008591
        ↓
학습이 진행될수록
Loss가 계속 감소
```

원본 이미지와 복원 이미지의 차이를 나타내는  
`Reconstruction Loss`도 실제로 감소했습니다.

```
Train Reconstruction Loss
0.105607 → 0.007741

Validation Reconstruction Loss
0.087386 → 0.007171
        ↓
VAE가 학습을 반복하면서
입력 이미지를 처음보다 더 비슷하게
복원하게 됨
```

학습이 끝난 뒤에는 두 가지 이미지 결과도 확인했습니다.

```
① Test Reconstruction

Test Input
        ↓
Encoder
        ↓
Latent
        ↓
Decoder
        ↓
Reconstruction
```

`latent`는 말 그대로 **직접 눈에 보이지 않는 잠재적인 표현**이고, 여기서는 **이미지 속 특징을 숫자로 표현한 공간**이라고 이해하면 됩니다.

예를 들어 사과 이미지라면 VAE가 학습하면서 다음과 같은 시각적 정보가 영향을 줄 수 있습니다. 

그래서 Latent Space는 이미지의 눈에 보이는 여러 특징을 작은 숫자 표현으로 바꿔 놓은 공간이라고 이해하면 됩니다.

```
사과의 전체적인 색
→ 얼마나 붉은가

사과의 형태
→ 둥근가 / 약간 길쭉한가

사과의 크기
→ 화면에서 크게 보이는가

사과의 위치
→ 가운데인지 약간 치우쳤는지

사과의 방향
→ 어느 정도 회전되어 있는지

밝기와 하이라이트
→ 밝은 부분이 어디에 있는지

윤곽
→ 사과의 외곽 모양

배경
→ 밝은 배경의 형태

표면의 큰 무늬
→ 전체적인 색 변화나 얼룩
```

Test 이미지를 VAE에 넣고 다시 복원한 결과를 통해  
**사과의 붉은색과 전체적인 둥근 형태가 어느 정도 유지되는지** 확인했습니다.

그리고 두 번째로 Random Sample을 만들었습니다.

```
② Random Sample

실제 이미지 입력 없음
        ↓
Random Latent z 16개
        ↓
학습된 Decoder
        ↓
새로운 이미지 16장
```

실제 Random Sample 결과에서는 이미지가 선명하지는 않았지만,

```
붉은색 계열
+
둥근 물체 형태
+
밝은 배경
```

과 같은 Apple 데이터의 일부 공통적인 특징이 나타나는 것을 확인했습니다.

따라서 1일차 VAE 실험의 전체 흐름은 다음과 같습니다.

```
Apple Train 419장
        ↓
VAE 10 Epoch 학습
        ↓
Train / Validation Loss 감소
        ↓
Reconstruction Loss도 감소
        ↓
Test Reconstruction 확인
        +
Random Sample 16장 생성
        ↓
Apple의 색·형태와 같은
일부 특징이 결과에 나타남
```

> **이번 VAE 실습의 목표는 사진처럼 완벽한 사과 이미지를 만드는 것이 아닙니다.**
> 
> Apple 이미지를 학습한 VAE가  
> **이미지의 특징을 Latent Space에 표현하고, 그 정보를 이용하여 입력 이미지를 다시 복원하거나, Random Latent 값에서 새로운 이미지를 만들어 낼 수 있다는 기본 원리**를 직접 확인한 것이 핵심입니다.

---

### `[ 중간문제]` Epoch가 증가하면 VAE 학습 결과는 어떻게 달라질까요?

지금까지는 다음 조건으로 VAE를 학습했습니다.

```
Dataset = Apple
Epoch = 10
Latent Dimension = 32
```

이번에는 **다른 조건은 그대로 유지하고 Epoch만 변경**해 봅니다.

이번 실험의 질문은 다음과 같습니다.

> **Epoch를 늘리면 VAE가 Apple 이미지를 더 잘 학습하고 복원할까요?**

---

#### 실험 조건

다음 조건은 변경하지 않습니다.

```
고정

Dataset
→ apple

Latent Dimension
→ 32

Train / Validation Split
→ 앞에서 만든 데이터 그대로 사용

Seed
→ 동일하게 유지
```

이번에 바꿀 것은 하나뿐입니다.

```
변경

Epoch
→ 1
→ 5
→ 10
→ 20
```

> 한 번에 여러 조건을 바꾸지 않습니다.  
> 그래야 결과 차이가 Epoch 때문인지 판단할 수 있습니다.

---

####   STEP 1. Epoch 1로 학습합니다

```bash
python scripts/05_train_vae.py \
  --dataset apple \
  --epochs 1 \
  --latent 32 \
  --eval-split val
```

결과에서 다음 내용을 기록합니다.

```
Train Loss
Validation Loss
Reconstruction 결과
```

---

####  STEP 2. Epoch 5로 학습합니다

```bash
python scripts/05_train_vae.py \
  --dataset apple \
  --epochs 5 \
  --latent 32 \
  --eval-split val
```

---

####  STEP 3. Epoch 10으로 학습합니다

```bash
python scripts/05_train_vae.py \
  --dataset apple \
  --epochs 10 \
  --latent 32 \
  --eval-split val
```

---

####  STEP 4. Epoch 20으로 학습합니다

```bash
python scripts/05_train_vae.py \
  --dataset apple \
  --epochs 100 \
  --latent 32 \
  --eval-split val
```

---

####  결과를 표에 기록합니다

|Epoch|Train Loss|Validation Loss|Reconstruction 관찰|
|---|---|---|---|
|1||||
|5||||
|10||||
|20||||

Reconstruction은 다음 기준으로 간단히 적습니다.

```
사과의 붉은색이 잘 유지되는가?

둥근 형태가 유지되는가?

윤곽이 얼마나 흐린가?

Epoch가 증가하면서
입력 이미지와 더 비슷해지는가?
```

---

####  문제 1

Epoch가 증가하면서 `Validation Loss`는 어떻게 변했나요?

```
답:
```

---

####  문제 2

Epoch 1과 Epoch 10의 Reconstruction을 비교합니다.

어떤 차이가 있었나요?

```
답:
```

---

####  문제 3

Epoch 10과 Epoch 20을 비교합니다.

```
Epoch 20이 Epoch 10보다
확실하게 더 좋아졌나요?

아니면 차이가 크지 않았나요?
```

관찰한 결과와 이유를 작성합니다.

```
답:
```

---

####  문제 4

다음 문장이 항상 맞는지 판단합니다.

> **“Epoch를 많이 늘릴수록 모델은 항상 더 좋아진다.”**

```
맞다 / 틀리다

이유:
```

---

####  문제 5

우리 팀이 최종 모델로 사용할 Epoch를 하나 선택합니다.

```
선택한 Epoch:

선택 이유:
```

단순히 Train Loss가 가장 작은 모델을 선택하지 않습니다.

다음 내용을 함께 봅니다.

```
Validation Loss
    +
Reconstruction 결과
```

---

####  문제 6

최종 Epoch를 결정한 뒤에만 Test 결과를 확인합니다.

예를 들어 최종적으로 `Epoch=10`을 선택했다면:

```bash
python scripts/05_train_vae.py \
  --dataset apple \
  --epochs 10 \
  --latent 32 \
  --eval-split test \
  --eval-only
```

왜 Epoch를 비교하는 과정에서는 Test가 아니라 Validation을 사용했는지 설명합니다.

```
답:
```

---

# PART 6. 생성한 데이터를 QA하고 Leakage를 확인합니다

앞에서는 같은 Apple Train 데이터를 이용하여 두 가지 실험을 진행했습니다.

```text
Augmentation
→ 보강 이미지 838장 생성

VAE
→ 모델 학습
→ Loss와 Reconstruction
→ Random Sample 확인
```

VAE의 학습 상태와 결과는 앞에서 이미 확인했습니다.

이제 PART 6에서는 **Augmentation으로 생성한 이미지가  
학습 데이터로 사용하기 전에 기술적으로 정상한지 확인하고,  
Train / Validation / Test 사이에 Leakage가 없는지 검사합니다.**

---

# 26. Automatic QA를 수행하는 코드를 작성합니다

`06_qa_images.py`는 **앞에서 생성한 Augmentation 이미지와 Metadata를 자동으로 검사하는 코드**입니다.

```text
Augmented Image
        +
Metadata
        ↓
생성 이미지가 실제로 존재하는가?
OpenCV로 정상적으로 읽히는가?
이미지 크기가 정상인가?
Metadata에 기록된 원본이 실제 Train 원본과 연결되는가?
        ↓
Automatic QA 결과 저장
```

VAE 모델을 다시 검사하는 단계가 아니라,  
**생성한 Augmentation 데이터의 기본 품질을 검사하는 단계**입니다.

코드를 작성합니다.

**파일: `scripts/06_qa_images.py`**

```python
from __future__ import annotations

import argparse
import hashlib
import sys
from pathlib import Path

import cv2
import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    return parser.parse_args()


def sha256_file(path: Path) -> str:
    digest = hashlib.sha256()

    with path.open("rb") as file:
        for chunk in iter(
            lambda: file.read(1024 * 1024),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()


def make_grid(
    image_paths,
    output_path,
    tile_size=128,
    columns=4,
):
    if not image_paths:
        return

    rows = []

    for start in range(
        0,
        len(image_paths),
        columns,
    ):
        batch = image_paths[
            start:start + columns
        ]

        tiles = []

        for path in batch:
            image = cv2.imread(
                str(path)
            )

            if image is None:
                continue

            tile = cv2.resize(
                image,
                (tile_size, tile_size),
            )

            cv2.putText(
                tile,
                path.stem.split("__")[-2],
                (5, 15),
                cv2.FONT_HERSHEY_SIMPLEX,
                0.35,
                (0, 0, 0),
                1,
                cv2.LINE_AA,
            )

            tiles.append(tile)

        while len(tiles) < columns:
            tiles.append(
                np.zeros(
                    (
                        tile_size,
                        tile_size,
                        3,
                    ),
                    dtype=np.uint8,
                )
            )

        rows.append(
            np.hstack(tiles)
        )

    if rows:
        grid = np.vstack(rows)

        saved = cv2.imwrite(
            str(output_path),
            grid,
        )

        if not saved:
            raise RuntimeError(
                f"QA Grid 저장 실패: {output_path}"
            )


def choose_human_qa_samples(
    metadata: pd.DataFrame,
    augmented_dir: Path,
) -> list[Path]:
    selected = []

    for method in [
        "brightness",
        "rotation",
        "blur",
        "noise",
    ]:
        group = metadata[
            metadata["method"] == method
        ]

        if group.empty:
            continue

        group = group.sample(
            n=min(
                4,
                len(group),
            ),
            random_state=42,
        )

        for filename in group["file"]:
            path = (
                augmented_dir
                / filename
            )

            if path.exists():
                selected.append(path)

    return selected


def main() -> None:
    args = parse_args()
    paths = dataset_paths(args.dataset)

    metadata_path = (
        paths["report"]
        / "synthetic_metadata.csv"
    )

    if not metadata_path.exists():
        raise FileNotFoundError(
            metadata_path
        )

    metadata = pd.read_csv(
        metadata_path
    )

    required_columns = {
        "file",
        "method",
        "source_hash",
    }

    missing_columns = (
        required_columns
        - set(metadata.columns)
    )

    if missing_columns:
        raise RuntimeError(
            "Metadata 열이 부족합니다: "
            + ", ".join(
                sorted(missing_columns)
            )
        )

    train_files = list_images(
        paths["split_train"]
    )

    train_hashes = {
        sha256_file(path)
        for path in train_files
    }

    rows = []

    for record in metadata.to_dict(
        orient="records"
    ):
        image_path = (
            paths["augmented_train"]
            / record["file"]
        )

        exists = image_path.exists()
        readable = False
        width = 0
        height = 0

        if exists:
            image = cv2.imread(
                str(image_path)
            )

            if image is not None:
                readable = True
                height, width = (
                    image.shape[:2]
                )

        source_hash_ok = (
            record["source_hash"]
            in train_hashes
        )

        auto_pass = (
            exists
            and readable
            and width > 0
            and height > 0
            and source_hash_ok
        )

        rows.append(
            {
                "file": record["file"],
                "method": record["method"],
                "exists": exists,
                "readable": readable,
                "width": width,
                "height": height,
                "source_hash_ok": source_hash_ok,
                "auto_pass": auto_pass,
            }
        )

    qa_path = (
        paths["report"]
        / "qa_results.csv"
    )

    pd.DataFrame(
        rows
    ).to_csv(
        qa_path,
        index=False,
        encoding="utf-8-sig",
    )

    grid_path = (
        paths["result_aug"]
        / "human_qa_grid.jpg"
    )

    human_qa_samples = (
        choose_human_qa_samples(
            metadata,
            paths["augmented_train"],
        )
    )

    make_grid(
        human_qa_samples,
        grid_path,
    )

    passed = sum(
        row["auto_pass"]
        for row in rows
    )

    print("QA total :", len(rows))
    print("QA pass  :", passed)
    print("QA fail  :", len(rows) - passed)
    print(
        "Human QA samples:",
        len(human_qa_samples),
    )
    print("QA CSV   :", qa_path)
    print("QA Grid  :", grid_path)


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
apple 또는 banana를 선택한다

        ↓

synthetic_metadata.csv를 읽는다
        +
split/train의 원본 이미지 목록을 가져온다

        ↓

Train 원본 이미지들의 SHA-256 Hash를 만든다

        ↓

Metadata에 기록된
Augmentation 이미지를 하나씩 검사한다

        ↓

파일이 실제로 존재하는가?
        ↓
OpenCV로 정상적으로 읽히는가?
        ↓
Width와 Height가 정상인가?
        ↓
Metadata의 source_hash가
실제 Train 원본 Hash에 존재하는가?

        ↓

모두 정상
→ auto_pass = True

하나라도 문제 있음
→ auto_pass = False

        ↓

전체 검사 결과를
qa_results.csv로 저장한다

        ↓

Brightness / Rotation / Blur / Noise에서
각각 최대 4장씩 대표 이미지를 선택한다

        ↓

최대 16장의 이미지를 하나의 Grid로 만든다

        ↓

human_qa_grid.jpg 저장

        ↓

QA 전체 수 / PASS / FAIL 출력
```

### 이 코드는 이렇게 이해하면 됩니다

이 코드 전체를 이해할 때는 개별 문법보다 **입력(Input) → 처리(Process) → 출력(Output)**을 먼저 보는 것이 좋습니다.

```
[입력]

Augmentation 이미지 838장
+
synthetic_metadata.csv
+
원본 Train 이미지

        ↓

[처리]

① 파일 존재 확인
② 이미지 읽기 확인
③ 이미지 크기 확인
④ 원본 Train과 Hash 연결 확인

        ↓

[출력]

qa_results.csv
→ 전체 이미지의 자동검사 결과

human_qa_grid.jpg
→ 사람이 눈으로 확인할 대표 이미지 최대 16장
```

여기에서 이해해야 할 핵심은 **두 가지 검사 목적이 다르다는 것**입니다.

```
Automatic QA

"파일과 데이터 기록이
기술적으로 정상한가?"

예:
파일이 있는가?
이미지가 열리는가?
크기가 정상인가?
실제 Train 원본에서 만들어졌는가?


Human QA

"이미지를 사람이 보았을 때
학습용으로 사용해도 괜찮은가?"

예:
너무 어둡지 않은가?
너무 흐리지 않은가?
Noise가 지나치지 않은가?
회전 때문에 객체가 잘리지 않았는가?
```

`[문제]` 위의 코드에서 아래 부분을 찾아보세요.

```
① metadata = pd.read_csv(...)
→ 무엇을 검사할지 Metadata를 읽음

② train_hashes = {...}
→ 실제 Train 원본의 Hash 목록을 만듦

③ auto_pass = (...)
→ Automatic QA의 합격 조건

④ qa_results.csv
→ 모든 이미지의 검사 결과를 저장

⑤ choose_human_qa_samples() + make_grid()
→ 사람이 확인할 대표 이미지 최대 16장을 준비
```

특히 이 코드의 중심은 사실 `auto_pass`입니다.

```python
auto_pass = (
    exists
    and readable
    and width > 0
    and height > 0
    and source_hash_ok
)
```

```
파일이 있고
AND
정상적으로 읽히고
AND
가로 크기가 정상이고
AND
세로 크기가 정상이고
AND
실제 Train 원본과 연결되어 있으면

→ PASS
```

Automatic QA 실행 결과를 확인합니다

```bash
python scripts/06_qa_images.py \
  --dataset apple
```

실제 결과는 다음과 같습니다.

```
QA total : 838
QA pass  : 838
QA fail  : 0
Human QA samples: 16

QA CSV:
reports/apple/qa_results.csv

QA Grid:
results/apple/augmentation/human_qa_grid.jpg
```

### 결과를 해석합니다

```
QA total : 838
→ 앞에서 생성한 Augmentation 이미지 838장을 검사함

QA pass : 838
→ 838장 모두 Automatic QA 조건 통과

QA fail : 0
→ 기술적인 문제가 발견된 이미지 없음

Human QA samples : 16
→ 사람이 눈으로 확인할 대표 이미지 16장을 선택함
```

즉 이번 결과는 다음을 의미합니다.

```
Augmentation 이미지 838장

        ↓

파일이 실제로 존재함
        +
OpenCV로 정상적으로 읽힘
        +
Width / Height가 정상임
        +
Metadata의 원본 Hash가
실제 Train 원본과 연결됨

        ↓

838장 모두 Automatic QA PASS
```

실제 `qa_results.csv`에서도 생성 이미지의 `exists`, `readable`, `source_hash_ok`, `auto_pass` 값이 `True`로 기록된 것을 확인할 수 있습니다.

하지만 여기서 주의합니다.

> **Automatic QA를 통과했다는 것은 파일과 기록이 기술적으로 정상하다는 뜻이지, 이미지가 좋은 학습데이터라는 뜻까지 보장하는 것은 아닙니다.**

그래서 코드가 함께 만든 `human_qa_grid.jpg`를 다음 단계에서 직접 확인합니다.

---

# 27. Human QA로 실제 이미지 품질을 확인합니다

26번에서 다음 파일이 만들어졌습니다.

```
results/apple/augmentation/human_qa_grid.jpg
```

이 파일을 직접 열어봅니다.

![[Pasted image 20261008090524.png]]

Grid에는 다음 네 가지 Augmentation에서 각각 최대 4장씩, 총 16장의 대표 이미지가 들어 있습니다.

```
1행 → Brightness 4장
2행 → Rotation 4장
3행 → Blur 4장
4행 → Noise 4장
```

이번에 실제로 만들어진 Grid를 보면 다음과 같이 해석할 수 있습니다.

```
Brightness
→ 밝기가 조금씩 다르지만
  모두 사과의 붉은색과 형태를 유지함

Rotation
→ 사과의 방향은 달라졌지만
  객체가 크게 잘리지 않고 형태가 유지됨

Blur
→ 표면의 세부 무늬는 부드러워졌지만
  사과의 전체 형태와 색은 구분 가능함

Noise
→ 표면에 잡음이 보이지만
  사과의 주요 색과 윤곽은 유지됨
```

따라서 이번 16장의 대표 Sample에서는 지나치게 강한 변형으로 사과의 의미가 
사라진 이미지는 눈에 띄지 않습니다.

Human QA에서는 다음을 확인합니다.

- [ ] 사과의 기본 형태가 유지되는가?
- [ ] Brightness가 지나치게 밝거나 어둡지 않은가?
- [ ] Rotation 때문에 객체가 심하게 잘리지 않았는가?
- [ ] Blur 후에도 사과를 알아볼 수 있는가?
- [ ] Noise 후에도 주요 색과 윤곽이 유지되는가?
- [ ] 실제 촬영환경에서 발생할 수 있는 변화라고 설명할 수 있는가?

이번 결과를 연결하면 다음과 같습니다.

```
Automatic QA
838 / 838 PASS
→ 파일과 Metadata는 기술적으로 정상

        ↓

Human QA
대표 이미지 16장 확인
→ 네 가지 Augmentation이
  사과의 주요 특징을 유지하는지 눈으로 확인

        ↓

현재 Sample 기준
→ 학습용 Augmentation 후보로 사용 가능
```

여기서 가장 중요한 차이는 이것입니다.

> Automatic QA는 “파일이 정상한가?”를 검사하고, 
> Human QA는 “이 이미지가 실제 학습에 사용할 만한가?”를 사람이 판단합니다.

현재 QA 결과만 보면 학습용 데이터로 사용해도 될 수준으로 보입니다.

Automatic QA에서 `838 / 838 PASS`가 나왔고, Human QA Grid에서도 Brightness·Rotation·Blur·Noise 모두 사과의 기본 형태와 색이 유지되어 있습니다. 특히 객체가 심하게 잘리거나, Blur 때문에 형태가 사라지거나, Noise가 과도하게 들어간 샘플은 눈에 띄지 않습니다.

현재 Automatic QA와 대표 16장 Human QA 결과를 기준으로 보면, 생성한 Augmentation 데이터는 학습용으로 사용 가능한 상태로 판단할 수 있습니다.

---

# 28. 오늘 실험의 데이터 분리가 안전했는지 최종 확인합니다

앞에서 오늘의 두 가지 실험을 모두 마쳤습니다.

```text
실험 A
Augmentation
→ Train 이미지로 보강 데이터 생성
→ Automatic QA
→ Human QA

실험 B
VAE
→ 원본 Train으로 모델 학습
→ Validation Loss 확인
→ Reconstruction / Random Sample 확인
```

이제 새로운 모델을 학습하는 것이 아닙니다.

마지막으로 확인할 것은

> **“오늘 사용한 Train·Validation·Test가 정말 서로 섞이지 않고 잘 분리되어 있었는가?”**

입니다.

만약 같은 이미지가 Train과 Validation 또는 Test에 동시에 들어가 있다면,  
모델이 이미 본 이미지를 평가에서 다시 보게 될 수 있습니다.

그러면 결과가 실제보다 좋아 보일 수 있습니다.

이것이 `Data Leakage`입니다.

```
정상

Train
Validation
Test

→ 서로 다른 데이터


문제

Train의 이미지
        ↓
같은 내용의 이미지가
Validation 또는 Test에도 존재

→ Data Leakage
```

이번 `07_check_leakage.py`는 파일 이름만 비교하지 않고  
이미지 파일의 **SHA-256 Hash**를 이용하여  
파일 내용이 완전히 같은 이미지가 서로 다른 Split에 들어가 있지 않은지 확인합니다.

또한 앞에서 만든 Augmentation 이미지도 확인합니다.

```
Augmented Image
        ↓
어떤 원본에서 만들어졌는가?
        ↓
그 원본은 Train에 있는가?
        ↓
Validation이나 Test 원본을
잘못 사용하지 않았는가?
```

즉 이 코드는 다음 두 가지를 확인합니다.

```
① Train / Validation / Test
   사이에 같은 이미지가 섞이지 않았는가?

② Augmentation 이미지는
   Train 원본에서만 만들어졌는가?
```

이 검사를 통과하면

> 오늘의 데이터 분리와 
> Augmentation 과정이 평가 데이터를 오염시키지 않은 상태였다고
> 확인할 수 있습니다.

이제 코드를 작성합니다.

**파일: `scripts/07_check_leakage.py`**

```python
from __future__ import annotations

import argparse
import hashlib
import sys
from pathlib import Path

import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day01_common import (
    dataset_paths,
    list_images,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    return parser.parse_args()


def sha256_file(path: Path) -> str:
    digest = hashlib.sha256()

    with path.open("rb") as file:
        for chunk in iter(
            lambda: file.read(1024 * 1024),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()


def hash_set(directory: Path) -> set[str]:
    return {
        sha256_file(path)
        for path in list_images(directory)
    }


def main() -> None:
    args = parse_args()
    paths = dataset_paths(args.dataset)

    train_hashes = hash_set(
        paths["split_train"]
    )

    val_hashes = hash_set(
        paths["split_val"]
    )

    test_hashes = hash_set(
        paths["split_test"]
    )

    train_val = (
        train_hashes
        & val_hashes
    )

    train_test = (
        train_hashes
        & test_hashes
    )

    val_test = (
        val_hashes
        & test_hashes
    )

    metadata_path = (
        paths["report"]
        / "synthetic_metadata.csv"
    )

    augmented_source_errors = 0

    if metadata_path.exists():
        metadata = pd.read_csv(
            metadata_path
        )

        if "source_hash" not in metadata.columns:
            raise RuntimeError(
                "Metadata에 source_hash 열이 없습니다."
            )

        for source_hash in metadata[
            "source_hash"
        ]:
            source_is_invalid = (
                source_hash not in train_hashes
                or source_hash in val_hashes
                or source_hash in test_hashes
            )

            if source_is_invalid:
                augmented_source_errors += 1

    leak_count = (
        len(train_val)
        + len(train_test)
        + len(val_test)
        + augmented_source_errors
    )

    report_path = (
        paths["report"]
        / "leakage_check.txt"
    )

    lines = [
        f"dataset={args.dataset}",
        f"train_val_duplicate={len(train_val)}",
        f"train_test_duplicate={len(train_test)}",
        f"val_test_duplicate={len(val_test)}",
        (
            "augmented_source_error="
            f"{augmented_source_errors}"
        ),
        f"leak_count={leak_count}",
    ]

    report_path.write_text(
        "\n".join(lines),
        encoding="utf-8",
    )

    for line in lines:
        print(line)

    if leak_count > 0:
        raise RuntimeError(
            "Data Leakage 후보가 있습니다."
        )

    print("Leakage check: PASS")


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
apple 또는 banana를 선택한다

        ↓

Train 이미지들의 SHA-256 Hash를 만든다
Validation 이미지들의 SHA-256 Hash를 만든다
Test 이미지들의 SHA-256 Hash를 만든다

        ↓

Train ↔ Validation
같은 이미지가 있는지 확인

Train ↔ Test
같은 이미지가 있는지 확인

Validation ↔ Test
같은 이미지가 있는지 확인

        ↓

synthetic_metadata.csv를 읽는다

        ↓

각 Augmentation 이미지의 원본 Hash를 확인한다

        ↓

원본이 Train에 있는가?
그리고
Validation / Test 원본은 아닌가?

        ↓

모든 중복과 오류 개수를 더한다

        ↓

Leak = 0
→ Leakage check: PASS

Leak > 0
→ Data Leakage 후보가 있으므로 FAIL

        ↓

검사 결과를
leakage_check.txt에 저장한다
```

## 이 코드에서 꼭 이해해야 하는 핵심

이 코드는 **새로운 모델을 학습하는 코드가 아닙니다.**

오늘 실험에 사용한 데이터가 정말 공정하게 분리되어 있었는지 
**마지막으로 감사(Audit)하는 코드**입니다.

```
Train
→ 모델이 학습한 데이터

Validation
→ 조건을 비교한 데이터

Test
→ 마지막 평가 데이터
```

이 셋에 같은 이미지가 섞여 있으면 평가 결과를 믿기 어려워집니다.

### 1. SHA-256 Hash는 이미지의 '지문'이라고 생각하면 됩니다

파일 이름이 달라도 내용이 완전히 같을 수 있습니다.

```
apple01.jpg
copy_apple.jpg

파일명은 다름
하지만 이미지 내용은 같을 수 있음
```

그래서 파일명을 비교하지 않고 이미지 파일의 SHA-256 Hash를 만듭니다.

```
이미지
→ SHA-256
→ 긴 고유값

같은 파일 내용
→ 같은 Hash
```

쉽게 생각하면 **“이미지마다 디지털 지문을 만들어 비교한다”**라고 이해하면 됩니다

### 2. 가장 중요한 비교는 세 가지입니다

코드의 핵심은 사실 이것입니다.

```python
train_hashes & val_hashes
train_hashes & test_hashes
val_hashes & test_hashes
```

`&`는 여기서 **양쪽에 공통으로 존재하는 값**을 찾는다는 뜻입니다.

```
Train ∩ Validation
→ 같은 이미지가 있나?

Train ∩ Test
→ 같은 이미지가 있나?

Validation ∩ Test
→ 같은 이미지가 있나?
```

정상이라면 모두 `0`이어야 합니다.

### 3. Augmentation의 원본도 검사합니다

앞에서 만든 838장의 Augmented Image는 반드시:

```
Train 원본
        ↓
Brightness / Rotation / Blur / Noise
        ↓
Augmented Image
```

여야 합니다.

이런 구조가 되면 안 됩니다.

```
Validation 원본
→ Augmentation
→ Train에서 사용

또는

Test 원본
→ Augmentation
→ Train에서 사용
```

그래서 Metadata의 `source_hash`를 확인하여 **Augmentation이 Train 원본에서만 만들어졌는지** 검사합니다.

### 4. 출력값은 이렇게 읽으면 됩니다

정상적인 결과:

```
train_val_duplicate=0
train_test_duplicate=0
val_test_duplicate=0
augmented_source_error=0
leak_count=0

Leakage check: PASS
```

의미는 아주 간단합니다.

```
Train ↔ Validation 중복 없음
Train ↔ Test 중복 없음
Validation ↔ Test 중복 없음

Augmentation 원본도
모두 Train에서만 가져옴

        ↓

Leakage 후보 0개

        ↓

PASS
```

### 이 코드에서 기억해야 할 한 문장

> `07_check_leakage.py`는 
> Train·Validation·Test의 이미지 지문(Hash)을 비교하고, 
> Augmentation의 원본도 Train에만 있는지 확인하여 
> 평가 데이터가 학습에 섞이지 않았는지 검사하는 코드입니다.

그리고 현재 코드는 **파일 내용이 완전히 동일한 Exact Duplicate**를 찾는 검사입니다. 비슷하게 찍힌 사진이나 연속 프레임처럼 내용이 유사하지만 
파일이 다른 경우까지 찾아내는 코드는 아닙니다.

실행합니다.

```bash
python scripts/07_check_leakage.py \
  --dataset apple
```

정상이라면 마지막에 다음이 표시됩니다.

```text
Leakage check: PASS
```

`PASS`가 의미하는 범위도 정확히 이해합니다.

```text
오늘 코드가 확인하는 것
→ 파일 내용이 완전히 같은 Exact Duplicate
→ Augmented Image가 Train 원본에서만 만들어졌는지

오늘 코드가 자동으로 확인하지 못하는 것
→ 서로 다른 파일이지만 매우 비슷한 Near-Duplicate
→ 같은 촬영 세션·같은 원본 개체에서 연속으로 얻은 유사 이미지
```

따라서 실제 프로젝트에서는 파일 Hash 확인만으로 Leakage 검사가 끝나지 않습니다.

사람·제품·촬영 세션·생산 Lot처럼 서로 연관된 데이터가 있다면 
관련된 데이터 묶음이 Train과 Test에 갈라지지 않도록 
Group 기준으로 Split해야 합니다.

1일차에서는 먼저 Exact Duplicate와 Augmentation 원본 Leakage를 코드로 확인하는 기본기를 익힙니다.

---

# 29. Apple 본 실습 결과를 한 번에 연결합니다

지금까지 한 일을 다시 연결합니다.

```text
Apple Red 1
        ↓
원래 Training / Test 확인
        ↓
Training
→ Train + Validation
        ↓
기존 Test 유지
        ↓
Train 한 장 Preview
        ↓
Brightness
Rotation
Blur
Noise
        ↓
Train 전체 Augmentation
        ↓
Metadata
        ↓
VAE
원본 Train 학습
Validation 확인
Test Reconstruction
Random Latent Sample
        ↓
Automatic QA
        ↓
Human QA
        ↓
Leakage Check
```

여기까지가 1일차 본 실습입니다.

이제 새로운 데이터인 `Banana 1`에 같은 흐름을 스스로 적용합니다.

---

# PART 7. Mini Challenge — Banana 1로 1일차 전체 흐름을 스스로 반복합니다

---

# 30. Mini Challenge의 목표와 전체 흐름을 확인합니다

Mini Challenge에서는 새로운 알고리즘을 배우지 않습니다.

Apple Red 1에서 함께 수행한 실험 절차를 이번에는 `Banana 1`에 스스로 적용합니다.

이번 Challenge의 핵심은 코드를 새로 만드는 것이 아니라 다음 세 가지를 직접 판단하는 것입니다.

```text
1. 데이터가 바뀌어도 같은 실험 절차를 적용할 수 있는가?

2. Augmentation 조건이 Banana에서도 적절한가?

3. VAE 조건을 Validation 결과로 비교하고 선택할 수 있는가?
```

오늘 작성한 스크립트를 그대로 재사용합니다.

```text
01_check_dataset.py
02_prepare_split.py
03_augment_preview.py
04_augment_train.py
05_train_vae.py
06_qa_images.py
07_check_leakage.py
```

Banana Challenge의 전체 흐름은 다음과 같습니다.

![[Pasted image 20261008100126.png]]

```text
Banana 1 원본 데이터
        ↓
입력 데이터 확인
        ↓
Train / Validation / Test 분리
        ↓
        ┌──────────────────────────────┴──────────────────────────────┐
        │                                                             │
        ↓                                                             ↓

[실험 A — Augmentation]                                  [실험 B — VAE]

원본 Train 한 장 Preview                                원본 Train으로 VAE 학습
        ↓                                                             ↓
Rotation 조건 비교                                      Latent 32 Validation 확인
        ↓                                                             ↓
사용할 Rotation 선택                                    Latent 16 Validation 확인
        ↓                                                             ↓
Train 전체 Augmentation                                 Validation 결과 비교
        ↓                                                             ↓
Metadata 확인                                           Latent 선택·Freeze
        ↓                                                             ↓
Automatic QA                                            Final Test Reconstruction
        ↓
Human QA

        └──────────────────────────────┬──────────────────────────────┘
                                       ↓
                              Leakage 최종 확인
                                       ↓
                         Apple과 Banana 결과 비교
                                       ↓
                            challenge_notes.md
```

중요합니다.

```text
Augmentation 결과
        ↓
VAE 입력
```

으로 사용하는 것이 아닙니다.

```text
같은 Banana 원본 Train
        ├─ Augmentation 실험
        └─ VAE 실험
```

처럼 두 실험을 각각 수행합니다.

---

# 31. Challenge 요구사항 1 — Banana 1 데이터를 준비하고 정상 여부를 확인합니다

Banana 1의 Training 이미지는 다음 위치에 넣습니다.

```text
data/banana/source_train/
```

Test 이미지는 다음 위치에 넣습니다.

```text
data/banana/source_test/
```

입력 데이터가 정상한지 확인합니다.

```bash
python scripts/01_check_dataset.py \
  --dataset banana
```

실행 결과를 보고 다음 내용을 기록합니다.

```text
Banana Training 수:

Banana Test 수:

전체 이미지 수:

대표 이미지 크기:

Dataset check:
```

`Dataset check: READY`가 출력되었다면 다음 단계로 진행합니다.

---

# 32. Challenge 요구사항 2 — Banana 데이터를 Train·Validation·Test로 나눕니다

Apple 실습에서 배운 것처럼 먼저 데이터를 나눈 뒤 Augmentation과 VAE를 진행합니다.

```bash
python scripts/02_prepare_split.py \
  --dataset banana
```

결과를 기록합니다.

| Split | 이미지 수 |
|---|---:|
| Train | |
| Validation | |
| Test | |

`split_manifest.csv`도 확인합니다.

```bash
head reports/banana/split_manifest.csv
```

다음 질문에 한 문장으로 답합니다.

> **왜 기존 Banana Test 이미지를 Train에 합치거나 Augmentation에 사용하면 안 될까요?**

```text
답:
```

---

# 33. Challenge 요구사항 3 — Banana 한 장에서 Augmentation 조건을 판단합니다

먼저 Apple과 같은 Rotation 10°를 적용합니다.

```bash
python scripts/03_augment_preview.py \
  --dataset banana \
  --rotation-angle 10
```

이번에는 Rotation만 15°로 변경합니다.

```bash
python scripts/03_augment_preview.py \
  --dataset banana \
  --rotation-angle 15
```

다음 두 Grid를 비교합니다.

```text
augmentation_grid_rot10_0.jpg
        VS
augmentation_grid_rot15_0.jpg
```

이번 비교에서는 다른 조건을 함께 바꾸지 않습니다.

```text
고정
→ 같은 Banana 데이터
→ Brightness / Blur / Noise 조건

변경
→ Rotation 10° → 15°
```

두 결과를 보고 다음을 판단합니다.

```text
Banana의 길고 휘어진 형태가 유지되는가?

회전하면서 객체가 부자연스럽게 잘리지는 않는가?

10°와 15° 중 어떤 범위를 Train 전체에 사용할 것인가?
```

```text
선택한 Rotation:

선택 이유:
```

정답은 하나가 아닙니다. 실제 Preview 결과를 근거로 선택합니다.

---

# 34. Challenge 요구사항 4 — Banana Train 전체를 보강하고 Metadata를 확인합니다

33번에서 선택한 Rotation 값을 사용합니다.

예를 들어 10°를 선택했다면:

```bash
python scripts/04_augment_train.py \
  --dataset banana \
  --rotation-max 10
```

15°를 선택했다면:

```bash
python scripts/04_augment_train.py \
  --dataset banana \
  --rotation-max 15
```

실행 결과에서 다음을 기록합니다.

```text
Banana Train 원본 수:

생성된 Augmented Image 수:

선택한 Rotation Max:
```

기본값은 원본 한 장당 보강 이미지 2장을 생성하므로 다음 관계도 확인합니다.

```text
Train 수 × 2
        ↓
예상 Augmented Image 수
        ↓
실제 Generated 수와 같은가?
```

Metadata를 확인합니다.

```bash
head reports/banana/synthetic_metadata.csv
```

다음 항목이 기록되어 있는지 확인합니다.

```text
생성 파일명
Augmentation 방법
원본 파일명
원본 Hash
Parameter
Seed
approved
```

이 단계의 질문은 다음입니다.

> **“Banana Train을 정상적으로 보강했고, 어떻게 생성했는지 다시 추적할 수 있는가?”**

---

# 35. Challenge 요구사항 5 — Banana VAE의 기본 조건을 Validation에서 확인합니다

이번에는 Augmented Image가 아니라 **원본 Banana Train**을 사용하여 VAE를 학습합니다.

```text
data/banana/augmented/train
→ VAE 입력으로 사용하지 않음

data/banana/split/train
→ VAE 학습에 사용
```

먼저 다음 조건을 기준으로 실행합니다.

```text
Epoch = 10
Latent = 32
Evaluation Split = Validation
```

```bash
python scripts/05_train_vae.py \
  --dataset banana \
  --epochs 10 \
  --latent 32 \
  --eval-split val
```

결과에서 다음 세 가지를 확인합니다.

```text
① Train / Validation Loss

② Validation Reconstruction

③ Random Sample
```

간단히 기록합니다.

```text
마지막 Train Loss:

마지막 Validation Loss:

Reconstruction에서 유지된 Banana 특징:

흐려지거나 사라진 특징:

Random Sample에서 보이는 색·형태:
```

여기서 Random Sample은 생성 특성을 관찰하는 보조 결과입니다.

**Latent 조건을 선택할 때는 Validation Loss와 Validation Reconstruction을 중심으로 판단합니다.**

---

# 36. Challenge 요구사항 6 — Latent 하나만 바꾸어 비교하고 최종 조건을 선택합니다

이번에는 다른 조건을 그대로 두고 `Latent Dimension`만 바꿉니다.

```text
실험 A
Epoch = 10
Latent = 32

실험 B
Epoch = 10
Latent = 16
```

Latent 16을 실행합니다.

```bash
python scripts/05_train_vae.py \
  --dataset banana \
  --epochs 10 \
  --latent 16 \
  --eval-split val
```

두 결과를 같은 기준으로 비교합니다.

| 조건 | 마지막 Validation Loss | Reconstruction의 색·형태 | Random Sample 관찰 |
|---|---:|---|---|
| Latent 32 | | | |
| Latent 16 | | | |

다음 질문에 답합니다.

```text
Validation Loss는 어느 조건이 더 안정적이었는가?

Banana의 노란색과 길고 휘어진 형태는
어느 Reconstruction에서 더 잘 유지되었는가?

Latent를 크게 또는 작게 만든다고
항상 결과가 좋아졌는가?

어떤 Latent를 최종 선택할 것인가?

선택 근거는 무엇인가?
```

```text
선택한 Latent:

선택 이유:
```

한 번에 Latent만 바꾼 이유도 설명합니다.

```text
답:
```

Validation 결과로 조건을 선택했다면 그 값을 Freeze합니다.

예를 들어 Latent 32를 선택했다면:

```bash
SELECTED_LATENT=32

python scripts/05_train_vae.py \
  --dataset banana \
  --epochs 10 \
  --latent "$SELECTED_LATENT" \
  --eval-split test \
  --eval-only
```

Latent 16을 선택했다면 `SELECTED_LATENT=16`으로 변경합니다.

최종 Test에서는 다음만 확인합니다.

```text
처음 보는 Banana Test에서도

노란색이 유지되는가?
길고 휘어진 형태가 유지되는가?
Banana라는 기본 특징을 알아볼 수 있는가?
```

Test 결과를 본 뒤 다시 Latent를 변경하지 않습니다.

```text
Validation
→ 조건 비교·선택

        ↓

Latent Freeze

        ↓

Test
→ 마지막 확인
```

---

# 37. Challenge 요구사항 7 — Augmentation QA와 Leakage를 최종 확인합니다

여기서 두 검사를 구분합니다.

```text
VAE
→ 35~36번에서
  Loss / Reconstruction / Random Sample로 결과를 확인함


Augmentation
→ 지금 Automatic QA / Human QA로
  생성된 보강 데이터의 품질을 확인함
```

즉 `06_qa_images.py`는 VAE 모델을 다시 검사하는 코드가 아닙니다.

앞에서 만든 Banana Augmentation 이미지와 Metadata를 검사합니다.

Automatic QA를 실행합니다.

```bash
python scripts/06_qa_images.py \
  --dataset banana
```

결과를 기록합니다.

```text
QA total:

QA pass:

QA fail:

Human QA samples:
```

`QA pass`가 생성된 Augmented Image 수와 같은지도 확인합니다.

그다음 `human_qa_grid.jpg`를 직접 열어봅니다.

다음 내용을 눈으로 확인합니다.

```text
Brightness
→ Banana의 노란색과 형태가 유지되는가?

Rotation
→ 객체가 심하게 잘리지 않았는가?

Blur
→ 흐려져도 Banana 형태를 알아볼 수 있는가?

Noise
→ 잡음이 있어도 주요 특징이 유지되는가?
```

```text
Human QA 판단:

학습용으로 사용하기 어려운 Sample이 있었는가?:

이유:
```

마지막으로 데이터 분리가 안전했는지 Leakage를 확인합니다.

```bash
python scripts/07_check_leakage.py \
  --dataset banana
```

정상적인 경우 다음과 같은 상태를 기대합니다.

```text
Train ↔ Validation 중복 0
Train ↔ Test 중복 0
Validation ↔ Test 중복 0
Augmentation 원본 오류 0

        ↓

Leakage check: PASS
```

이 단계까지 끝나면 Banana Challenge의 데이터 생성·모델 실험·품질검수·데이터 분리 확인이 모두 완료됩니다.

---

# 38. Mini Challenge 결과를 `challenge_notes.md`로 정리합니다

**파일: `reports/banana/challenge_notes.md`**

길게 작성하지 않습니다.

실제로 실행한 결과와 판단 근거만 정리합니다.

```markdown
# Day01 Mini Challenge - Banana 1

## 1. 데이터
- Source Training:
- Source Test:
- Train:
- Validation:
- Test:

## 2. Augmentation
- 비교한 Rotation: 10° / 15°
- 선택한 Rotation:
- 선택 이유:
- 생성된 Augmented Image 수:
- Metadata 확인 결과:

## 3. VAE
- 고정 Epoch: 10
- 비교한 Latent: 32 / 16
- Latent 32 Validation 결과:
- Latent 16 Validation 결과:
- 선택한 Latent:
- 선택 이유:
- Final Test Reconstruction 결과:

## 4. QA
- Automatic QA:
- Human QA:
- 학습용으로 사용하기 어려운 이미지가 있었는가?:
- 이유:

## 5. Leakage
- Leakage Check:
- Test를 Train에 섞지 않은 이유:

## 6. Apple과 Banana 비교
- Augmentation에서 공통으로 적용할 수 있었던 점:
- Banana에서 다시 판단해야 했던 점:
- Apple VAE 결과와 Banana VAE 결과의 차이:

## 7. 오늘의 결론
- Augmentation과 VAE의 가장 큰 차이:
- Validation을 사용한 이유:
- 오늘 가장 중요하다고 생각한 데이터 관리 원칙:
```

Challenge 전체 흐름을 다시 연결하면 다음과 같습니다.

```text
Banana 원본
        ↓
Split
        ↓
        ┌────────────────────┴────────────────────┐
        ↓                                         ↓

Augmentation                                VAE
        ↓                                         ↓
Metadata                                  Validation 비교
        ↓                                         ↓
QA                                        조건 Freeze
                                                  ↓
                                             Final Test

        └────────────────────┬────────────────────┘
                             ↓
                      Leakage 확인
                             ↓
                         결과 정리
```

---

# 39. Mini Challenge 완료 조건을 확인합니다

아래 항목을 직접 체크합니다.

- [ ] Banana Training / Test 데이터를 준비하고 `READY`를 확인했다.
- [ ] Train / Validation / Test를 분리하고 Test를 보호했다.
- [ ] Rotation 10°와 15°를 비교하고 근거를 가지고 하나를 선택했다.
- [ ] Train에만 Augmentation을 적용하고 Metadata를 확인했다.
- [ ] VAE가 Augmented Image가 아니라 원본 Train을 학습한다는 것을 구분했다.
- [ ] Latent 32와 16을 같은 Epoch에서 Validation으로 비교했다.
- [ ] Validation Loss와 Reconstruction을 근거로 Latent를 선택했다.
- [ ] 선택한 Latent를 Freeze한 뒤 Final Test를 확인했다.
- [ ] Automatic QA와 Human QA를 수행했다.
- [ ] Leakage Check에서 데이터 중복과 Augmentation 원본을 확인했다.
- [ ] Apple과 Banana 결과의 차이를 설명했다.
- [ ] `challenge_notes.md`에 실험 결과와 판단 근거를 기록했다.

---

# 40. Mini Challenge를 약 3시간으로 진행합니다

| 단계 | 권장 시간 |
|---|---:|
| Banana 데이터 확인·Split | 20분 |
| Augmentation Preview·Rotation 선택 | 25분 |
| Train 전체 Augmentation·Metadata 확인 | 25분 |
| VAE Latent 32 · Validation 확인 | 25분 |
| VAE Latent 16 · Validation 비교·선택 | 35분 |
| 선택 조건 Freeze·Final Test | 10분 |
| Automatic QA·Human QA·Leakage | 20분 |
| Apple vs Banana 비교·`challenge_notes.md` | 20분 |
| **합계** | **180분** |

Mini Challenge에서는 새로운 코드를 만드는 데 시간을 쓰지 않습니다.

```text
새 알고리즘 추가
        X

Apple에서 만든 코드 재사용
        ↓
Banana 데이터에 적용
        ↓
조건 하나씩 변경
        ↓
Validation으로 비교
        ↓
결과를 근거로 판단
        ↓
최종 Test
        O
```

Mini Challenge의 최종 목표는 단순히 모든 명령어를 실행하는 것이 아닙니다.

> **같은 Pipeline을 새로운 데이터에 다시 적용하고, 조건을 하나씩 비교한 뒤 자신의 결과를 근거로 선택할 수 있으면 1일차 핵심을 제대로 이해한 것입니다.**


---

# PART 8. 오늘 만든 결과를 정리하고 버전관리합니다

---

# 41. 오늘의 실험 기록을 `day01_notes.md`에 정리합니다

**파일: `day01_notes.md`**

다음 내용을 정리합니다.

```markdown
# Subject 11 - Day 01

## 1. 본 실습 데이터

- Dataset: Fruits-360
- Class: Apple Red 1
- Source Training:
- Source Test:
- Train:
- Validation:
- Test:

## 2. 사용한 Augmentation

- Brightness:
- Rotation:
- Blur:
- Noise:

## 3. Metadata

- 생성 이미지 수:
- 기록한 주요 항목:
- Seed:

## 4. VAE

- Epoch:
- Latent:
- Train Loss 특징:
- Validation Loss 특징:
- Reconstruction에서 유지된 특징:
- 사라지거나 흐려진 특징:
- Random Sample에서 확인한 특징:

## 5. QA

- Automatic QA 결과:
- Human QA에서 확인한 내용:

## 6. Leakage

- Leakage Check 결과:
- Test를 그대로 유지한 이유:

## 7. Mini Challenge

- Dataset: Banana 1
- 선택한 Rotation:
- Latent 32 Validation 결과:
- Latent 16 Validation 결과:
- 선택한 Latent:
- 선택 근거:
- Final Test Reconstruction:
- Apple과 다른 점:

## 8. 오늘의 결론

- Augmentation:
- VAE:
- 가장 중요한 데이터 관리 원칙:
```

---

# 42. README에 1일차 내용을 정리합니다

`README.md`에 다음과 같은 구조로 정리합니다.

````markdown
# Subject 11 - Synthetic Data

## Day 01 - Augmentation · VAE

### Main Dataset

- Fruits-360 `Apple Red 1`

### Mini Challenge Dataset

- Fruits-360 `Banana 1`

### Day 01 Pipeline

```text
Real Image
→ Train / Validation / Test
        ├─ Train Augmentation → Metadata
        └─ 원본 Train → VAE
                    → Reconstruction
                    → Random Sample
→ Augmented Image QA
→ Leakage Check
```

### Main Scripts

```text
scripts/
├─ 00_check_env.py
├─ 01_check_dataset.py
├─ 02_prepare_split.py
├─ 03_augment_preview.py
├─ 04_augment_train.py
├─ 05_train_vae.py
├─ 06_qa_images.py
└─ 07_check_leakage.py
```

### Key Results

- Augmentation Preview
- Synthetic Metadata
- VAE Model
- VAE Loss
- Validation Reconstruction for Condition Selection
- Final Test Reconstruction
- VAE Random Samples
- QA Result
- Leakage Check
- Banana Mini Challenge Notes

### Next

Day 02에서는 GAN·WGAN-GP·Diffusion을 이용하여
새로운 이미지 생성 방식을 비교합니다.
````

README에는 코드 전체를 복사하지 않습니다.

다음 내용만 빠르게 확인할 수 있으면 됩니다.

```text
무슨 데이터를 사용했는가?

무슨 코드를 실행하는가?

무슨 결과가 만들어지는가?

다음 수업으로 어떻게 이어지는가?
```

---

# 43. `.gitignore`를 작성합니다

원본 이미지·생성 이미지·모델·가상환경은 Git에 올리지 않습니다.

`.gitignore`에 다음 내용을 작성합니다.

```gitignore
# Python
.venv/
__pycache__/
*.pyc

# Input / generated image data
data/
results/

# Model
models/
*.pt
*.pth

# Large or generated reports
reports/*/qa_results.csv
reports/*/synthetic_metadata.csv
reports/*/vae_loss_*.csv

# Environment
reports/environment_freeze.txt

# Editor / OS
.vscode/
.DS_Store
```

다음과 같은 작은 문서 기록은 Git에 남길 수 있습니다.

```text
Python Source
README.md
day01_notes.md
challenge_notes.md
leakage_check.txt
```

기관 정책에 따라 원격 저장소 사용 범위는 달라질 수 있습니다.

원본 데이터는 저장소에 포함하지 않습니다.

---

# 44. Local Git에 1일차 결과를 저장합니다

교과 11은 2~5일차도 같은 프로젝트에 이어서 작업하므로 Git 저장소는 `day01_image_aug_vae` 안이 아니라 **교과 11 프로젝트 Root**에서 시작합니다.

먼저 Root로 이동합니다.

```bash
cd ~/ai_vision/subject11_synthetic_data
```

처음 한 번만 Git을 시작합니다.

```bash
git init
git branch -M main
```

상태를 확인합니다.

```bash
git status
```

1일차 폴더를 등록합니다.

```bash
git add day01_image_aug_vae
```

다시 확인합니다.

```bash
git status
```

다음 항목이 Git에 들어가지 않았는지 확인합니다.

```text
day01_image_aug_vae/data/
day01_image_aug_vae/results/
day01_image_aug_vae/models/
day01_image_aug_vae/.venv/
```

Commit합니다.

```bash
git commit -m "feat: complete subject11 day01 augmentation and VAE"
```

기록을 확인합니다.

```bash
git log --oneline
```

이렇게 시작하면 2~5일차 폴더도 같은 `subject11_synthetic_data` 저장소에 이어서 기록할 수 있습니다.

원격 저장소 사용이 허용된 환경이라면 연결된 저장소에 Push할 수 있습니다.

외부 저장소 사용이 제한된 환경에서는 Local Git 또는 기관 내부 Git만 사용합니다.

---

# 45. 1일차 자가 체크리스트를 확인합니다

오늘 배운 내용을 기준으로 직접 체크합니다.

- [ ] `Apple Red 1` Training과 Test의 역할을 구분할 수 있다.
- [ ] Training에서 Train과 Validation을 나누는 이유를 설명할 수 있다.
- [ ] Test를 학습 데이터 생성에 사용하지 않는 이유를 설명할 수 있다.
- [ ] Data Leakage가 무엇인지 설명할 수 있다.
- [ ] Brightness·Rotation·Blur·Noise의 의미를 설명할 수 있다.
- [ ] Augmentation Preview를 보고 변형 범위가 현실적인지 판단할 수 있다.
- [ ] Train에만 자동 Augmentation을 적용할 수 있다.
- [ ] Metadata에서 원본·방법·Parameter·Seed를 읽을 수 있다.
- [ ] VAE의 Encoder·Latent·Decoder 흐름을 설명할 수 있다.
- [ ] Train Loss와 Validation Loss의 역할을 구분할 수 있다.
- [ ] Test Original과 Reconstruction을 비교할 수 있다.
- [ ] VAE Random Sample과 Reconstruction의 차이를 설명할 수 있다.
- [ ] Automatic QA와 Human QA의 차이를 설명할 수 있다.
- [ ] Leakage Check 결과를 읽을 수 있다.
- [ ] `Banana 1`에서 같은 전체 Pipeline을 스스로 다시 실행했다.
- [ ] Mini Challenge에서 한 번에 하나의 조건만 변경했다.
- [ ] Latent 32와 16은 Validation Loss와 Validation Reconstruction으로 비교했다.
- [ ] Validation 근거로 Latent를 선택하고 Freeze했다.
- [ ] 선택된 Latent의 Test Reconstruction은 마지막에 한 번만 확인했다.
- [ ] Test를 본 뒤 Latent 조건을 다시 바꾸지 않았다.
- [ ] `day01_notes.md`와 `challenge_notes.md`를 작성했다.
- [ ] README를 정리했다.
- [ ] Git Commit을 완료했다.

---

# 46. 1일차에서 반드시 남겨야 할 세 가지 생각

## ① Augmentation

```text
기존 이미지를
현실적인 범위에서 변형하여
학습 조건을 다양하게 만드는 것
```

## ② VAE

```text
이미지를 특징 공간으로 압축하고
그 특징에서 다시 이미지를 복원하며

Latent Space에서 Sampling한 값으로
새로운 이미지를 만들어 보는
생성모델의 기본 구조
```

## ③ 데이터 관리

```text
먼저 Split
        ↓
Train에서만 데이터 생성
        ↓
Validation / Test 보호
        ↓
Metadata 기록
        ↓
QA
        ↓
Leakage 확인
```

> **합성데이터는 많이 만들었다고 끝나는 것이 아닙니다.**
>
> 어떤 원본에서 어떤 조건으로 만들었는지 기록하고, 사용할 수 있는 품질인지 확인하며, 평가 데이터가 학습에 섞이지 않았는지 검증해야 합니다.

---

# 다음 시간

2일차에는 VAE의 압축·복원에서 한 단계 더 나아갑니다.

```text
1일차

기존 이미지
→ Augmentation

이미지
→ Encoder
→ Latent
→ Decoder
→ Reconstruction / Random Sample

        ↓

2일차

Noise
→ GAN / WGAN-GP
→ 새로운 이미지 생성

Prompt / Seed
→ Diffusion
→ 새로운 이미지 생성
```

> **1일차에서 반드시 기억할 질문:**  
> 데이터를 새로 만들었다면, 그 데이터는 왜 만들었고, 어디에서 왔으며, 품질은 괜찮고, 평가 데이터와 분리되어 있는가?
