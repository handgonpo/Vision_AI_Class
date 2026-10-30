## 오늘의 핵심 질문

> 배경 이미지와 투명 Object PNG를 합성할 때, 프로그램이 객체를 붙인 위치를 이용하여 Detection 학습용 이미지와 정답 BBox를 함께 자동 생성할 수 있을까?

![[Pasted image 20261012124800.png]]

1일차와 2일차에서는 주로 이미지 자체를 변형하거나 새로 생성하는 방법을 실습했습니다.

```text
1일차
Apple Red 1
→ Augmentation
→ VAE
→ Reconstruction / Random Sample

2일차
같은 Apple Train
→ DCGAN
→ WGAN-GP

Pretrained Diffusion
→ Prompt + Seed
→ 새로운 이미지
```

3일차에는 여기에서 한 단계 더 나아갑니다.

객체검출 모델이 학습하려면 이미지 한 장만 있어서는 충분하지 않습니다.

```text
Detection 학습데이터

Image
  +
Class
  +
Bounding Box
```

오늘은 수업 전에 준비된 작은 Source Data를 사용합니다.

```text
Background JPG
        +
투명 Object PNG
        ↓
Position / Scale / Rotation
        ↓
Cut-Paste
        ↓
Synthetic Detection Image
        +
Pixel BBox
        +
YOLO Label
        ↓
BBox 재시각화
        ↓
Automatic QA
        ↓
Human QA
        ↓
Metadata
```

본 실습에서는 `main` 데이터를 함께 사용합니다.

하루의 마지막 Mini Challenge에서는 본 실습과 겹치지 않도록 별도로 준비된 `challenge` 데이터를 사용하여 오늘의 전체 흐름을 스스로 다시 수행합니다.

```text
본 실습
main
→ Background 20장
→ Object 3 Class × 5장
→ Cut-Paste + YOLO BBox
→ QA

==============================

Mini Challenge
challenge
→ Background 10장
→ Object 3 Class × 3장
→ 같은 Pipeline을 스스로 반복
→ Rotation 조건 하나만 변경
→ 결과 비교·설명
```

오늘의 목적은 이미지를 많이 만드는 것이 아닙니다.

다음 질문에 답할 수 있도록 **합성 이미지와 정답 Label을 함께 만들고, 다시 검수하고, 생성 조건을 추적하는 전체 과정**을 경험하는 것이 목표입니다.

```text
어떤 Source를 사용했는가?
        ↓
어떤 위치·크기·회전으로 붙였는가?
        ↓
BBox는 정확한가?
        ↓
YOLO Label은 정상인가?
        ↓
합성 장면은 현실적인가?
        ↓
어떤 조건으로 만든 데이터인지 다시 찾을 수 있는가?
```

---

3일차는 쉽게 말하면 “포토샵 합성을 Python으로 자동화해서 Detection용 합성데이터를 만드는 실습” 같은 방식입니다.

포토샵으로 사람이 한다면 이런 작업입니다.

```
배경 사진 열기
        +
배경이 투명한 객체 PNG 가져오기
        ↓
크기 조절
회전
위치 이동
        ↓
배경 위에 붙이기
        ↓
합성 이미지 완성
```

---

3일차 Cut-Paste 합성데이터 제작 방식은
실제 검출 오류가 있는데, 그 오류 상황의 실제 데이터가 부족할 때  
적은 원본 데이터로 많은 학습 상황을 만들어 보려고  
Cut-Paste 합성데이터를 사용하는 것입니다.

조금만 더 정확히 말하면:

- 모델이 현장에서 **잘 못 찾는 경우**가 있음
- 그런데 그 상황의 실제 데이터를 많이 모으기 어려움
- 그래서
    - 배경 이미지
    - 객체 이미지(PNG 등)  
        를 이용해  
        **위치, 크기, 각도, 개수**를 바꿔가며 합성 이미지를 많이 만듦
- 동시에 **BBox 라벨도 자동 생성**해서 Detection 학습에 사용함

즉 핵심은:

```
실제 데이터 부족
        ↓
실패하는 상황을 더 많이 만들어야 함
        ↓
Cut-Paste로 다양한 장면 합성
        ↓
Detection 학습데이터와 정답 라벨을 함께 확보
```

다만 주의할 점은:

> **합성데이터를 많이 만든다고 무조건 좋아지는 것은 아닙니다.**

왜냐하면 너무 부자연스럽게 만들면  
실제 현장과 차이가 커서 오히려 도움이 안 될 수도 있기 때문입니다.

한 줄 결론으로 정리하면:

> **Cut-Paste는 실제 데이터가 부족할 때, 검출에 필요한 다양한 상황을 저비용으로 많이 만들어 학습을 보강하는 방법입니다.**

---

# 현업에서 Cut-Paste 합성데이터를 사용하는 사례

|현업 분야|실제로 해결하려는 문제|Cut-Paste로 만드는 데이터|AI 모델이 학습하는 것|
|---|---|---|---|
|제조업 컨베이어|제품이 놓인 위치나 방향이 매번 달라짐|제품 PNG를 컨베이어 배경의 여러 위치·크기·각도로 합성|제품 위치 검출, 종류 구분|
|제조라인 혼입 검사|정상 제품 사이에 다른 종류의 제품이 드물게 섞임|다른 제품 PNG를 정상 컨베이어 장면에 합성|잘못 섞인 제품 검출|
|물류창고|택배 상자의 크기·위치·방향이 다양함|여러 상자 PNG를 물류 작업대 배경에 배치|상자 검출 및 위치 파악|
|로봇 피킹|로봇이 다양한 위치의 부품을 집어야 함|부품 PNG를 작업대의 여러 위치에 합성|집어야 할 객체의 위치 검출|
|산업안전|통로나 작업 구역에 공구·장애물이 놓이는 경우가 드묾|공구·박스·장애물 PNG를 실제 작업장 바닥에 합성|위험 구역의 장애물 검출|
|재활용품 자동 분류|캔·페트병·종이컵이 다양한 상태로 등장|분리수거 컨베이어 위에 객체별 PNG를 합성|재활용품 종류와 위치 검출|
|매장 상품 인식|상품의 진열 위치와 배열이 달라짐|상품 PNG를 진열대 배경에 합성|상품 종류 및 위치 검출|
|농산물 선별|과일이 여러 위치와 크기로 나타남|과일 PNG를 선별 컨베이어 배경에 합성|과일 검출 및 개수 확인|

이 사례들은 Cut-Paste 적용 가능성이 있는 상황입니다. 실제 학습에 도움이 되는지는 합성 장면의 현실성과 별도 모델 평가로 검증해야 합니다.

### 컨베이어 검사 사례

![The conveyor chain, and conveyor belt on production line set up in clean room area.|285](https://images.openai.com/static-rsc-4/q9wJCnPxBbbjih6_nC2pHBdoDqH0hBel-IVuK_JOXdRJu9_yLTge597u-JmyU_Y3gMd6paqxGYeR1n4fhFxnGfnB52dmZiDiCcfgmiJ-c6Iywxi7Mvn5FWUvleypcHmWineRKlThtzxlZAcmQRdnzJqESelgfD9tv036grMweqrDXEO8b2KyvDzm76syAZ6x?purpose=inline)        ![Public Services at Allan Company|262](https://images.openai.com/static-rsc-4/RXtnS9k00GH7pUb5hr4ld6S7jz4C3uR5I6lLc1l0sjjEUS1kzogg1Xg8w4PborVqZ-F8pdJmAT91vCwYVmL_eyT3otuWqZnw5snI5ZSeMQdx82GM7Xr1_tUQkr_bV_KrVkN3f3faAy1v-UUO8gHE1aTYizr9hB_fIejl_dANgpM?purpose=inline)

① 실제 컨베이어 배경                                ② 검출하려는 제품 사진
                     
                                ↓
                                
                     Python Cut-Paste — 위치·크기·방향 변경

![AI Aids Recycling Efforts - Neousys Think Tank](https://images.openai.com/static-rsc-4/qHNv5SFkzMjxb3K4Gr7_0HR0S-Pllwjayuz5H9bAJZNr-W6MCHdsTuhOGDhm3AA1rw57soQMJuxzzn2-q-9aPe_ML1QMZmeVycS8kenOFCqPL2rgcWIx-ccXR1ZLYHCFAIcnMPJXrgmQ252uuXUJVpdlB0Qm7LjRZz37jO0j5DA?purpose=inline)

③ 다양한 제품 배치가 있는 합성 장면을 만드는 개념 예시


![[Pasted image 20261012140945.png]]

---
가령 컨베이어에서 플라스틱병과 캔을 구분하는 AI를 개발한다고 가정해 보겠습니다.

실제 현장에서 모든 위치와 방향으로 제품을 놓고 사진을 수천 장 촬영하려면 시간이 많이 걸립니다.

대신 다음과 같이 만듭니다.

```
빈 컨베이어 사진 20장
+
페트병 PNG 5장
+
캔 PNG 5장
        ↓
Python 자동 합성

페트병 왼쪽 / 가운데 / 오른쪽
캔 위쪽 / 아래쪽
제품 크기와 방향 변경
        ↓
Synthetic Image 1,000장
+
각 이미지의 BBox 자동 생성
        ↓
YOLO Detection 학습 후보 데이터
```

여기서 1,000장은 설명을 위한 예시입니다. 실제로 몇 장을 생성해야 유리한지는 데이터의 다양성과 모델 성능을 보고 결정해야 합니다.

이런 방식은 컨베이어처럼 배경과 카메라 위치가 비교적 일정한 환경에서 특히 실험해 볼 가치가 있습니다.

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

- 1~2일차와 같은 교과 11 프로젝트와 1일차 `.venv`를 그대로 이어서 사용할 수 있습니다.
- 3일차에 새로운 Source Data가 필요한 이유를 설명할 수 있습니다.
- `main`과 `challenge` 데이터의 역할을 구분할 수 있습니다.
- Background JPG와 투명 RGBA Object PNG의 차이를 설명할 수 있습니다.
- `drill`, `fire_extinguisher`, `wet_floor_sign`의 Class ID를 고정하여 사용할 수 있습니다.
- PNG의 Alpha Channel이 Cut-Paste에서 어떤 역할을 하는지 설명할 수 있습니다.
- 한 개의 객체를 Background에 합성할 수 있습니다.
- 합성 위치를 이용하여 Pixel BBox를 계산할 수 있습니다.
- Pixel BBox를 YOLO 정규화 좌표로 변환할 수 있습니다.
- 위치·크기·회전을 바꾸어 여러 Synthetic Detection Image를 자동 생성할 수 있습니다.
- 생성 이미지와 YOLO TXT를 같은 파일명으로 저장할 수 있습니다.
- 생성 조건을 Metadata CSV에 기록할 수 있습니다.
- YOLO Label을 다시 이미지 위에 그려 BBox를 확인할 수 있습니다.
- 모든 Label의 Class ID·좌표 범위·파일 대응을 Automatic QA로 확인할 수 있습니다.
- Automatic QA와 Human QA의 차이를 설명할 수 있습니다.
- 합성 가능한 데이터와 실제 학습에 사용할 수 있는 데이터가 같지 않다는 점을 설명할 수 있습니다.
- Mini Challenge에서 `challenge` Source Data로 오늘의 전체 흐름을 스스로 다시 수행할 수 있습니다.
- Mini Challenge에서 한 번에 하나의 조건만 변경하고 결과 차이를 설명할 수 있습니다.
- 오늘 만든 코드와 결과를 README와 Git에 정리할 수 있습니다.

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260908145548.png]]

3일차에는 먼저 제공된 **배경 이미지와 투명 객체 PNG가 정상적으로 준비되었는지 확인**한 뒤 
실습을 시작합니다.  

실험 A에서는 배경 한 장에 객체 한 개를 직접 붙여 보면서 **Cut-Paste → BBox 계산 → YOLO Label 생성** 과정을 이해하고, 실험 B에서는 같은 원리를 여러 이미지에 자동으로 반복하여 합성 Detection 데이터셋을 만듭니다.  

마지막에는 생성된 BBox와 Label이 올바른지 **Automatic QA와 Human QA로 검수**하고, Mini Challenge에서는 다른 데이터로 같은 과정을 반복하면서 Rotation 조건을 바꾸어 결과 차이를 스스로 분석합니다.

```text
2일차 프로젝트 확인
        ↓
1일차 .venv 재사용
        ↓
3일차 폴더 생성
        ↓
제공된 day03_student_data 배치
        ↓

[본 실습 Source 확인]

main/backgrounds
20장

main/objects
drill              5장
fire_extinguisher  5장
wet_floor_sign     5장
        ↓
classes.txt 확인
        ↓
입력 데이터 자동 검사
        ↓
Asset Preview
        ↓

[실험 A — Single Cut-Paste]

bg_001.jpg
+
drill_001.png
        ↓
Scale
Rotation
Position
        ↓
Cut-Paste
        ↓
Pixel BBox
        ↓
YOLO Label
        ↓
BBox Visualization
        ↓

[실험 B — Batch Dataset]

main Source Data
        ↓
Class 균형 구성
        ↓
Background Random
Object View Random
Position Random
Scale Random
Rotation Random
        ↓
Synthetic Image
+
YOLO Label
        ↓
Metadata
        ↓

[BBox와 QA]

YOLO Label
        ↓
Pixel BBox로 다시 변환
        ↓
이미지 위에 재시각화
        ↓
Automatic QA
        ↓
Human QA
        ↓
Approved / Needs Review / Rejected
        ↓

[Mini Challenge]

challenge/backgrounds
10장

challenge/objects
3 Class × 3장
        ↓
같은 Pipeline 스스로 반복
        ↓
Rotation 15° 기준 실험
        ↓
Rotation 30° 비교 실험
        ↓
한 변수 실험 결과 해석
        ↓

README · Git 정리
        ↓
자가 체크
```

> **오늘의 데이터 운영 원칙**
>
> `main`과 `challenge`는 이미 수업 전에 서로 다른 Source Image로 분리되어 있습니다.
>
> 본 실습 중에는 `main`만 사용합니다.
>
> Mini Challenge에서는 `challenge`만 사용합니다.
>
> 같은 `bg_001.jpg` 또는 `drill_001.png`라는 파일명이 있어도 `main`과 `challenge`는 서로 다른 폴더의 다른 Source입니다.
>
> 오늘 생성한 Synthetic Image는 자동으로 Train 승인하지 않습니다. BBox 재시각화와 QA를 거친 뒤 사용 여부를 판단합니다.

---

# 오늘 사용할 실제 데이터

## 1. 제공되는 3일차 Source Data

오늘은 새로운 이미지를 직접 촬영하거나 웹에서 다시 다운로드하지 않습니다.

수업 시작 전에 제공된 `day03_student_data`를 사용합니다.

```text
day03_student_data/
│
├─ main/
│  ├─ backgrounds/
│  │  ├─ bg_001.jpg
│  │  ├─ bg_002.jpg
│  │  ├─ ...
│  │  └─ bg_020.jpg
│  │
│  └─ objects/
│     ├─ drill/
│     │  ├─ drill_001.png
│     │  ├─ ...
│     │  └─ drill_005.png
│     │
│     ├─ fire_extinguisher/
│     │  ├─ fire_extinguisher_001.png
│     │  ├─ ...
│     │  └─ fire_extinguisher_005.png
│     │
│     └─ wet_floor_sign/
│        ├─ wet_floor_sign_001.png
│        ├─ ...
│        └─ wet_floor_sign_005.png
│
├─ challenge/
│  ├─ backgrounds/
│  │  ├─ bg_001.jpg
│  │  ├─ ...
│  │  └─ bg_010.jpg
│  │
│  └─ objects/
│     ├─ drill/
│     │  ├─ drill_001.png
│     │  ├─ drill_002.png
│     │  └─ drill_003.png
│     │
│     ├─ fire_extinguisher/
│     │  ├─ fire_extinguisher_001.png
│     │  ├─ fire_extinguisher_002.png
│     │  └─ fire_extinguisher_003.png
│     │
│     └─ wet_floor_sign/
│        ├─ wet_floor_sign_001.png
│        ├─ wet_floor_sign_002.png
│        └─ wet_floor_sign_003.png
│
└─ classes.txt
```

## 2. 본 실습 `main`

```text
Background
20장

Object
drill               5장
fire_extinguisher   5장
wet_floor_sign      5장

총 Object
15장
```

Object PNG는 객체 주변이 투명하도록 미리 정리되어 있습니다.

오늘은 Source Image와 원본 Mask를 다시 전처리하지 않습니다.

학생이 사용하는 단계에서는 이미 다음 상태입니다.

```text
RGB Object
+
Alpha Channel
        ↓
RGBA PNG
        ↓
Cut-Paste에 바로 사용
```

## 3. Mini Challenge `challenge`

```text
Background
10장

Object
drill               3장
fire_extinguisher   3장
wet_floor_sign      3장

총 Object
9장
```

`challenge` 데이터는 본 실습과 다른 Background와 다른 Object View를 사용합니다.

따라서 코드는 같아도 실제 입력은 달라집니다.

## 4. Class 순서는 고정합니다

`classes.txt`의 내용은 다음과 같습니다.

```text
drill
fire_extinguisher
wet_floor_sign
```

YOLO Class ID는 줄 순서로 결정합니다.

```text
0 → drill
1 → fire_extinguisher
2 → wet_floor_sign
```

오늘은 중간에 이 순서를 바꾸지 않습니다.

---

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | Cut-Paste·BBox·QA 코드 작성 | 오늘 실습 전체 실행 |
| pathlib | Source·결과 폴더 관리 | 경로를 안전하게 연결 |
| Pillow | RGBA Object 합성 | 투명 객체를 Background에 붙임 |
| Alpha Channel | 투명 영역 표현 | 객체만 보이게 합성 |
| OpenCV | BBox 재시각화·파일 검사 | Label을 다시 이미지에 그림 |
| Pandas | Metadata·QA CSV 처리 | 생성 조건과 검수 결과 기록 |
| Cut-Paste | 2D 합성데이터 생성 | Background에 객체 배치 |
| Position Randomization | 위치 변경 | 객체가 한 위치에만 몰리지 않게 함 |
| Scale Randomization | 크기 변경 | 여러 크기의 객체 생성 |
| Rotation Randomization | 방향 변경 | 여러 각도의 객체 생성 |
| Pixel BBox | 객체 위치 기록 | `x1, y1, x2, y2` |
| YOLO Label | 정규화 BBox 저장 | `class xc yc w h` |
| Automatic QA | Label 자동 검사 | 형식·범위·파일 오류 확인 |
| Human QA | 시각적 검수 | 합성 장면이 현실적인지 판단 |
| Metadata | 생성 이력 기록 | 어떤 Source와 조건으로 만들었는지 추적 |
| Git | 코드·실험 기록 관리 | 재현 가능한 실습 상태 저장 |

---

# PART 1. 2일차 프로젝트에서 3일차를 이어서 시작합니다

---

# 1. 1~3일차가 어떻게 연결되는지 확인합니다

```text
1일차

Real Image
→ Augmentation

Real Image
→ VAE
→ Reconstruction / Random Sample


2일차

Real Train
→ DCGAN / WGAN-GP

Pretrained Diffusion
→ Prompt + Seed

        ↓

새로운 이미지 생성


3일차

Background
+
Transparent Object PNG
        ↓
Cut-Paste
        ↓
새로운 이미지
+
정답 BBox 자동 생성
```

3일차에서 가장 큰 변화는 **이미지와 Label을 동시에 만든다**는 점입니다.

```text
1~2일차
Image 중심

3일차
Image + Class + BBox
```

---

# 2. 교과 11 프로젝트 Root로 이동합니다

```bash
cd ~/ai_vision/subject11_synthetic_data
```

현재까지의 폴더를 확인합니다.

```bash
ls
```

다음 폴더가 보여야 합니다.

```text
day01_image_aug_vae
day02_generative_models
```

3일차 폴더를 만듭니다.

```bash
mkdir -p day03_detection_cutpaste
cd day03_detection_cutpaste
```

---

# 3. 3일차 폴더와 파일을 준비합니다

다음 구조를 만듭니다.

```bash
mkdir -p \
src \
scripts \
data \
dataset/main \
dataset/challenge \
results/main \
results/challenge \
reports/main \
reports/challenge
```

파일을 준비합니다.

```bash
touch src/day03_common.py

touch scripts/00_check_day03.py
touch scripts/01_preview_assets.py
touch scripts/02_cutpaste_single.py
touch scripts/03_generate_cutpaste_dataset.py
touch scripts/04_visualize_yolo_bbox.py
touch scripts/05_validate_yolo_labels.py
touch scripts/06_make_qa_report.py

touch day03_notes.md
touch README.md
touch .gitignore
```

현재 구조를 확인합니다.

```bash
tree -L 2
```

다음과 비슷하면 됩니다.

```text
day03_detection_cutpaste/
├─ data/
├─ dataset/
│  ├─ main/
│  └─ challenge/
├─ reports/
│  ├─ main/
│  └─ challenge/
├─ results/
│  ├─ main/
│  └─ challenge/
├─ scripts/
├─ src/
├─ day03_notes.md
├─ README.md
└─ .gitignore
```

---

# 4. 제공된 `day03_student_data`를 `data/` 아래에 배치합니다

수업에서 제공된 `day03_student_data` 폴더를 다음 위치에 복사합니다.

```text
day03_detection_cutpaste/
└─ data/
   └─ day03_student_data/
```

복사한 뒤 확인합니다.

```bash
tree data/day03_student_data -L 4
```

다음 구조가 보여야 합니다.

```text
data/day03_student_data/
├─ main/
│  ├─ backgrounds/
│  └─ objects/
│     ├─ drill/
│     ├─ fire_extinguisher/
│     └─ wet_floor_sign/
├─ challenge/
│  ├─ backgrounds/
│  └─ objects/
│     ├─ drill/
│     ├─ fire_extinguisher/
│     └─ wet_floor_sign/
└─ classes.txt
```

오늘 수업에서는 원본 BG-20K 전체 폴더나 Object의 원본 `images/`, `masks/`를 사용하지 않습니다.

이미 수업용으로 정리된 `day03_student_data`만 사용합니다.

---

# 5. 1일차 가상환경을 그대로 사용합니다

현재 위치는 다음입니다.

```text
~/ai_vision/subject11_synthetic_data/day03_detection_cutpaste
```

1일차에서 만든 `.venv`를 활성화합니다.

```bash
source ../day01_image_aug_vae/.venv/bin/activate
```

Python 위치를 확인합니다.

```bash
which python
```

최종 결과 아래와 같이 가상환경 경로가 맞으면 정상적으로 활성화된 상태입니다.

```
/home/user1/project11/.venv/bin/python
```

라이브러리를 확인합니다.

```bash
python -c "import cv2, PIL, pandas; print('Day03 libraries OK')"
```

정상이라면:

```text
Day03 libraries OK
```

가 출력됩니다.

3일차를 위해 새로운 가상환경을 만들지 않습니다.

---

# 6. 3일차 실습에서 사용할 공통 기능을 준비합니다

앞에서 3일차 프로젝트 폴더를 만들고, 제공된 `day03_student_data`를 준비했습니다.

이제 본격적으로 이미지를 합성하기 전에 **실습 코드들이 공통으로 사용할 기능을 하나의 Python 파일에 작성**합니다.

## 이 코드는 무엇을 하나요?

3일차에는 여러 Python 파일을 작성하여 순서대로 실습합니다.

예를 들어 다음과 같은 작업이 필요합니다.

- 배경 이미지가 저장된 폴더 찾기
- 투명한 객체 PNG가 저장된 폴더 찾기
- `classes.txt`에서 객체의 Class 이름 읽기
- 합성 이미지와 YOLO Label을 저장할 경로 정하기
- 새로운 실험을 시작할 때 결과 폴더 정리하기

이러한 기능을 각각의 실습 코드에 반복해서 작성하면 코드가 길어지고, 경로를 수정할 때마다 여러 파일을 고쳐야 합니다.

그래서 자주 사용하는 기능을 `day03_common.py`라는 **공통 도우미 파일**에 모아 둡니다.

쉽게 말하면, 앞으로 작성할 Python 코드들이 **어디에서 데이터를 읽고, 어디에 결과를 저장해야 하는지 알려주는 파일**입니다.

## 이 파일에는 어떤 기능을 작성하나요?

| 공통 기능 | 역할 |
|---|---|
| `list_images()` | 지정한 폴더에서 이미지 파일 목록 가져오기 |
| `read_classes()` | `classes.txt`에서 Class 이름 읽기 |
| `asset_paths()` | 배경과 객체 PNG가 있는 폴더 경로 찾기 |
| `experiment_paths()` | 합성 이미지·Label·검수 결과의 저장 경로 정하기 |
| `reset_directory()` | 새로운 실험을 위해 지정한 결과 폴더 초기화하기 |

`reset_directory()`는 기존 폴더의 파일을 삭제하므로, 실제 실험 기록을 보존해야 할 때는 주의해서 사용합니다.

## 오늘 본 실습에서는 어떻게 사용하나요?

본 실습에서는 제공된 `main` 데이터를 사용합니다.

예를 들어 이후 실습에서 배경 이미지를 읽어야 한다면, 다음 경로를 찾아야 합니다.

`data/day03_student_data/main/backgrounds/`

객체 PNG를 읽어야 한다면 다음 경로가 필요합니다.

`data/day03_student_data/main/objects/`

앞으로 작성할 Python 코드는 이러한 경로를 직접 반복해서 작성하는 대신, `day03_common.py`의 공통 함수를 호출하여 가져옵니다.

따라서 현재는 **본 실습에 필요한 데이터와 결과 폴더를 쉽게 관리하는 기능을 만든다**고 이해하면 됩니다.

참고로 마지막 Mini Challenge에서는 별도로 준비된 `challenge` 데이터를 사용합니다. 이때도 코드를 새로 작성하지 않고, 같은 공통 기능을 재사용하도록 준비합니다.

**파일: `src/day03_common.py`**

```python
from __future__ import annotations

import shutil
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]

DATA_ROOT = (
    ROOT
    / "data"
    / "day03_student_data"
)

CLASS_FILE = DATA_ROOT / "classes.txt"

VALID_EXTENSIONS = {
    ".jpg",
    ".jpeg",
    ".png",
    ".bmp",
}


def list_images(
    directory: Path,
) -> list[Path]:
    if not directory.exists():
        return []

    return sorted(
        path
        for path in directory.iterdir()
        if path.is_file()
        and path.suffix.lower()
        in VALID_EXTENSIONS
    )


def read_classes() -> list[str]:
    if not CLASS_FILE.exists():
        raise FileNotFoundError(
            CLASS_FILE
        )

    classes = [
        line.strip()
        for line
        in CLASS_FILE.read_text(
            encoding="utf-8"
        ).splitlines()
        if line.strip()
    ]

    if not classes:
        raise RuntimeError(
            "classes.txt가 비어 있습니다."
        )

    return classes


def asset_paths(
    asset_set: str,
) -> dict[str, Path]:
    if asset_set not in {
        "main",
        "challenge",
    }:
        raise ValueError(
            f"지원하지 않는 asset_set: "
            f"{asset_set}"
        )

    root = DATA_ROOT / asset_set

    return {
        "root": root,
        "backgrounds": (
            root / "backgrounds"
        ),
        "objects": (
            root / "objects"
        ),
    }


def experiment_paths(
    asset_set: str,
    tag: str,
) -> dict[str, Path]:
    if not tag.strip():
        raise ValueError(
            "tag는 비어 있을 수 없습니다."
        )

    dataset_root = (
        ROOT
        / "dataset"
        / asset_set
        / tag
    )

    result_root = (
        ROOT
        / "results"
        / asset_set
        / tag
    )

    report_root = (
        ROOT
        / "reports"
        / asset_set
    )

    paths = {
        "images": (
            dataset_root
            / "images"
        ),
        "labels": (
            dataset_root
            / "labels"
        ),
        "visualized": (
            result_root
            / "visualized"
        ),
        "report": report_root,
    }

    report_root.mkdir(
        parents=True,
        exist_ok=True,
    )

    return paths


def reset_directory(
    directory: Path,
) -> None:
    if directory.exists():
        shutil.rmtree(
            directory
        )

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )
```

### 의사코드로 읽어보기

```text
[1. 프로젝트 기준 경로 설정]

현재 day03_common.py 파일의 위치를 확인한다
        ↓
3일차 프로젝트 ROOT 폴더를 찾는다
        ↓
입력 데이터가 저장된 DATA_ROOT 경로를 정한다
        ↓
classes.txt 파일의 위치를 정한다
        ↓
사용할 이미지 확장자를 정한다
JPG / JPEG / PNG / BMP


[2. list_images() — 이미지 목록 가져오기]

이미지 폴더 경로를 전달받는다
        ↓
폴더가 존재하는지 확인한다
        ↓
폴더가 없다면 빈 목록을 반환한다
        ↓
폴더가 있다면
    폴더 안의 파일들을 하나씩 확인한다
        ↓
    실제 파일인지 확인한다
        ↓
    이미지 확장자에 해당하는 파일만 선택한다
        ↓
파일명을 기준으로 정렬한다
        ↓
이미지 파일 경로 목록을 반환한다


[3. read_classes() — Class 목록 읽기]

classes.txt 파일이 존재하는지 확인한다
        ↓
파일이 없다면 오류를 발생시킨다
        ↓
파일이 있다면 내용을 읽는다
        ↓
한 줄씩 Class 이름을 가져온다
        ↓
앞뒤 공백과 빈 줄을 제거한다
        ↓
Class 목록이 비어 있다면 오류를 발생시킨다
        ↓
Class 이름 목록을 반환한다


[4. asset_paths() — 입력 Source 경로 찾기]

asset_set 값을 전달받는다
        ↓
main 또는 challenge인지 확인한다
        ↓
다른 값이라면 오류를 발생시킨다
        ↓
선택한 데이터의 기준 폴더를 찾는다
        ↓
Background 이미지 폴더 경로를 정한다
        ↓
Object PNG 폴더 경로를 정한다
        ↓
입력 Source 경로들을 Dictionary로 반환한다


[5. experiment_paths() — 결과 저장 경로 준비]

asset_set과 tag 값을 전달받는다
        ↓
tag가 비어 있는지 확인한다
        ↓
비어 있다면 오류를 발생시킨다
        ↓
실험별 Dataset 폴더 경로를 정한다
        ↓
합성 이미지 저장 경로를 정한다
        ↓
YOLO Label 저장 경로를 정한다
        ↓
BBox 시각화 이미지 저장 경로를 정한다
        ↓
검수 Report 저장 경로를 정한다
        ↓
Report 폴더가 없다면 생성한다
        ↓
결과 저장 경로들을 Dictionary로 반환한다

※ images / labels / visualized 폴더는
  이 함수에서 실제로 생성하지 않는다.


[6. reset_directory() — 결과 폴더 초기화]

초기화할 폴더 경로를 전달받는다
        ↓
폴더가 이미 존재한다면
    기존 폴더와 내부 파일을 모두 삭제한다
        ↓
빈 폴더를 다시 생성한다
```

### 이 코드에서 개발자가 알아야 할 핵심과 수정하는 상황

`day03_common.py`는 이미지 합성 알고리즘을 구현하는 파일이 아니라, 
**입력 데이터와 결과 파일을 공통으로 관리하는 파일**입니다.

따라서 모든 코드를 모두 리뷰 하는것 보다는 어떤 기능이 있고, 어떤 상황에서 수정해야 하는지를 이해하는 것이 중요합니다.

#### ① 개발자가 확인하거나 수정할 수 있는 부분

|핵심 코드|역할|언제 수정하나요?|
|---|---|---|
|`ROOT`|3일차 프로젝트의 기준 폴더|파일 구조 자체가 달라졌을 때|
|`DATA_ROOT`|제공된 Source Data의 위치|입력 데이터 폴더의 위치나 이름이 달라졌을 때|
|`CLASS_FILE`|`classes.txt`의 위치|Class 파일명이나 저장 위치가 달라졌을 때|
|`VALID_EXTENSIONS`|사용할 이미지 확장자|새로운 이미지 형식을 지원할 때|
|`list_images()`|이미지 파일 목록을 가져옴|하위 폴더까지 검색해야 하는 등 검색 방식이 달라질 때|
|`read_classes()`|Class 이름을 읽음|Class 파일의 형식이나 읽는 규칙이 달라질 때|
|`asset_paths()`|Background·Object의 입력 경로 관리|Source Data의 폴더 구조가 변경될 때|
|`experiment_paths()`|Image·Label·Report 저장 경로 관리|결과 저장 구조를 변경할 때|
|`reset_directory()`|기존 결과 폴더를 삭제하고 다시 생성|결과를 삭제하지 않고 보존하도록 동작을 바꿀 때|

#### ② 가장 먼저 확인할 부분은 `DATA_ROOT`입니다

현재 코드는 `day03_common.py`의 위치를 기준으로 프로젝트 Root를 자동으로 찾습니다.

- `ROOT` → `day03_detection_cutpaste/`
- `DATA_ROOT` → `day03_detection_cutpaste/data/day03_student_data/`
    

따라서 현재 수업에서 안내한 폴더 구조를 그대로 사용한다면 **경로를 직접 수정할 필요가 없습니다.**

나중에 데이터를 다른 폴더에 저장하거나 이름을 바꾸었다면 `DATA_ROOT`를 수정해야 합니다.

#### ③ `read_classes()`에서는 Class 순서가 중요합니다

이 함수는 `classes.txt`에 작성된 이름을 순서대로 읽습니다.

현재 Class는 다음과 같습니다.

|Class ID|Class Name|
|---|---|
|0|drill|
|1|fire_extinguisher|
|2|wet_floor_sign|

이 순서는 이후 YOLO Label의 Class ID와 연결됩니다.

새로운 객체를 추가하거나 Class 구성을 변경할 때는 `classes.txt`뿐 아니라 **입력 데이터 검사 코드와 Label 생성 코드도 함께 확인**해야 합니다.

특히 기존 데이터의 Class 순서를 임의로 바꾸면 생성된 Label의 의미가 달라질 수 있으므로 주의합니다.

#### ④ `experiment_paths()`는 결과를 구분하는 중요한 기능입니다

이 함수는 입력받은 실험 이름인 `tag`를 이용하여 결과 경로를 구분합니다.

예를 들어 같은 `main` 데이터라도 다음처럼 관리할 수 있습니다.

|실험|합성 이미지 저장 위치|
|---|---|
|Smoke Test|`dataset/main/smoke/images/`|
|Baseline|`dataset/main/baseline/images/`|
|다른 조건 실험|`dataset/main/experiment02/images/`|

`labels/`와 `visualized/`도 같은 방식으로 실험별 경로가 구분됩니다. 다만 Report 폴더는 `reports/main/`을 공통으로 사용하므로, 개별 Report 파일명에서도 실험을 구분해야 합니다.

**새로운 실험을 할 때마다 공통 코드를 수정하는 것이 아니라, 이후 실행 코드에서 다른** `**tag**` **값을 전달하면 됩니다.**

#### ⑤ `reset_directory()`는 특히 주의해야 합니다

이 함수는 지정한 폴더가 이미 존재하면 내부 파일까지 모두 삭제하고, 빈 폴더를 다시 만듭니다.

따라서 같은 결과 폴더를 초기화하면 이전에 생성한 이미지와 Label이 사라질 수 있습니다.

기존 실험 결과를 보관해야 한다면 **새로운 실험 이름을 사용하거나, 파일을 삭제하지 않도록 처리 방식을 변경**해야 합니다.

---

### 그러면 지금 수업에서는 무엇을 수정해야 하나요?

현재 제공된 `day03_student_data`의 폴더 구조와 Class 순서를 그대로 사용한다면 `**day03_common.py**`**를 수정하지 않고 사용하면 됩니다.**

앞으로 실습하면서 중요한 것은 다음 세 가지입니다.

**첫째,** 입력 이미지가 어디에 저장되어 있는지 확인합니다.

**둘째,** 각 실습 결과가 어느 폴더에 저장되는지 확인합니다.

**셋째,** 기존 결과를 삭제하거나 덮어쓸 수 있는 코드를 실행할 때 주의합니다.

> **핵심 정리:** `day03_common.py`는 실습 전체의 입력과 출력 경로를 관리하는 공통 도우미 파일입니다. 실제 개발에서는 데이터 구조가 달라질 때 경로·Class·파일 형식을 수정하고, 새로운 실험을 할 때는 기존 코드를 재사용하여 결과를 구분해 관리합니다.


---

# 7. Cut-Paste 실습에 사용할 입력 데이터를 자동으로 검사합니다

앞에서 배경 이미지와 객체 PNG가 들어 있는 폴더를 준비했습니다.

이제 실제 이미지를 합성하기 전에 **데이터가 올바르게 준비되었는지 자동으로 검사하는 Python 코드**를 작성합니다.

## 이 코드는 무엇을 하나요?

3일차 Cut-Paste 실습에서는 배경 이미지 위에 객체 PNG를 붙여 새로운 이미지를 만듭니다.

그런데 배경 이미지가 누락되거나, 객체 PNG에 투명한 배경이 없다면 이후 합성 과정에서 문제가 발생할 수 있습니다.

따라서 본격적인 실습을 시작하기 전에 다음 항목을 확인합니다.

| 검사 항목 | 확인하는 내용 | 검사하는 이유 |
|---|---|---|
| Class 목록 | `classes.txt`의 이름과 순서가 올바른가? | 객체에 잘못된 Class ID가 지정되는 문제 방지 |
| 배경 이미지 수 | Background 이미지가 20장인가? | 필요한 Source가 누락되지 않았는지 확인 |
| 배경 이미지 읽기 | OpenCV에서 이미지가 정상적으로 열리는가? | 손상되거나 읽을 수 없는 이미지 확인 |
| 객체 이미지 수 | 각 Class의 PNG가 5장씩 있는가? | 합성에 사용할 객체 이미지 누락 확인 |
| Alpha Channel | 객체 PNG에 투명한 영역과 보이는 영역이 있는가? | 객체 주변 배경까지 함께 붙는 문제 방지 |

## 데이터를 폴더에 업로드 합니다.

```bash
explorer.exe .
```

폴더가 열리면 제공된 데이터를 복사 붙여넣기 합니다.

![[Pasted image 20261012150848.png]]

오늘 본 실습에서는 `main` 폴더의 데이터를 사용합니다.

```text
day03_detection_cutpaste/data/main/
│
├── backgrounds/
│   └── JPG 이미지 20장
│
└── objects/
    ├── drill/
    │   └── PNG 이미지 5장
    │
    ├── fire_extinguisher/
    │   └── PNG 이미지 5장
    │
    └── wet_floor_sign/
        └── PNG 이미지 5장
```

---

## `classes.txt` 파일을 작성합니다

이미지를 복사했다면 이번에는 **검출할 객체의 이름과 순서를 정의하는 `classes.txt` 파일**을 만들겠습니다.

YOLO 객체검출에서는 객체의 종류를 숫자인 Class ID로 구분합니다.

| Class ID | Class 이름 | 의미 |
|---|---|---|
| 0 | `drill` | 전동드릴 |
| 1 | `fire_extinguisher` | 소화기 |
| 2 | `wet_floor_sign` | 미끄럼 주의 표지판 |

`classes.txt`에서는 **첫 번째 줄이 Class 0, 두 번째 줄이 Class 1, 세 번째 줄이 Class 2**가 됩니다.

따라서 반드시 정해진 순서대로 작성해야 합니다.

**생성할 파일: `data/day03_student_data/classes.txt`**

3일차 프로젝트 터미널에서 다음 명령어를 실행합니다.

```bash
mkdir -p data/day03_student_data

cat > data/day03_student_data/classes.txt <<'EOF'
drill
fire_extinguisher
wet_floor_sign
EOF
```

이 명령어는 `classes.txt`를 생성하고 세 가지 Class 이름을 한 줄씩 저장합니다.

기존에 같은 이름의 파일이 있으면 내용이 덮어써지므로, 이미 작성한 파일이 있다면 먼저 확인합니다.

###  `classes.txt`가 정상적으로 작성되었는지 확인합니다

```bash
cat data/day03_student_data/classes.txt
```

다음 내용이 출력되면 정상입니다.

```text
drill
fire_extinguisher
wet_floor_sign
```

### 최종 폴더 구조를 확인합니다

```text
day03_detection_cutpaste/
└── data/
    └── day03_student_data/
        │
        ├── classes.txt          ← 직접 작성
        │
        └── main/
            ├── backgrounds/     ← 배경 이미지 20장
            │
            └── objects/
                ├── drill/              ← PNG 5장
                ├── fire_extinguisher/  ← PNG 5장
                └── wet_floor_sign/     ← PNG 5장
```

**여기서 중요한 점은 `classes.txt`를 `main` 폴더 안이 아니라 `day03_student_data` 바로 아래에 저장해야 한다는 것입니다.**

---
## Alpha Channel도 왜 검사하나요?

Cut-Paste에서는 배경 이미지 위에 **객체 부분만 자연스럽게 붙여야 합니다.**

객체 이미지가 단순한 JPG이거나 투명 영역이 없는 PNG라면, 객체 주변의 흰색 배경까지 함께 붙을 수 있습니다.

따라서 객체 PNG에 실제로 투명한 영역이 존재하는지 확인합니다.

단, 이 검사는 투명 영역의 존재 여부를 확인하는 것이며, 객체의 윤곽이 완벽하게 분리되었는지까지 판단하지는 않습니다. 윤곽 품질은 이후 이미지 시각화 과정에서도 확인합니다.

파일을 작성합니다

**파일: `scripts/00_check_day03.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import cv2
from PIL import Image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    asset_paths,
    list_images,
    read_classes,
)


EXPECTED_CLASSES = [
    "drill",
    "fire_extinguisher",
    "wet_floor_sign",
]

EXPECTED_COUNTS = {
    "main": {
        "backgrounds": 20,
        "objects_per_class": 5,
    },
    "challenge": {
        "backgrounds": 10,
        "objects_per_class": 3,
    },
}


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    return parser.parse_args()


def has_alpha(
    path: Path,
) -> bool:
    with Image.open(path) as image:
        rgba = image.convert(
            "RGBA"
        )

        alpha = rgba.getchannel(
            "A"
        )

        alpha_min, alpha_max = (
            alpha.getextrema()
        )

    return (
        alpha_min < 255
        and alpha_max > 0
    )


def main() -> None:
    args = parse_args()

    classes = read_classes()

    if classes != EXPECTED_CLASSES:
        raise RuntimeError(
            "classes.txt 순서가 "
            "수업 기준과 다릅니다.\n"
            f"현재: {classes}\n"
            f"기준: {EXPECTED_CLASSES}"
        )

    paths = asset_paths(
        args.asset_set
    )

    expected = (
        EXPECTED_COUNTS[
            args.asset_set
        ]
    )

    backgrounds = list_images(
        paths["backgrounds"]
    )

    if len(backgrounds) != (
        expected["backgrounds"]
    ):
        raise RuntimeError(
            "Background 수가 다릅니다.\n"
            f"기대: "
            f"{expected['backgrounds']}\n"
            f"현재: {len(backgrounds)}"
        )

    unreadable = []

    for path in backgrounds:
        image = cv2.imread(
            str(path)
        )

        if image is None:
            unreadable.append(
                path.name
            )

    if unreadable:
        raise RuntimeError(
            "읽을 수 없는 Background: "
            + ", ".join(unreadable)
        )

    print("=" * 60)
    print(
        "Subject 11 - Day 03 Check"
    )
    print("=" * 60)

    print(
        "Asset Set   :",
        args.asset_set,
    )

    print(
        "Backgrounds :",
        len(backgrounds),
    )

    for class_id, class_name in enumerate(
        classes
    ):
        class_dir = (
            paths["objects"]
            / class_name
        )

        object_files = list_images(
            class_dir
        )

        if len(object_files) != (
            expected[
                "objects_per_class"
            ]
        ):
            raise RuntimeError(
                f"{class_name} 수가 "
                "수업 기준과 다릅니다.\n"
                f"기대: "
                f"{expected['objects_per_class']}\n"
                f"현재: {len(object_files)}"
            )

        alpha_bad = [
            path.name
            for path in object_files
            if not has_alpha(path)
        ]

        if alpha_bad:
            raise RuntimeError(
                f"{class_name}에서 "
                "Alpha 문제가 있습니다: "
                + ", ".join(
                    alpha_bad
                )
            )

        print(
            f"Class {class_id} "
            f"{class_name}: "
            f"{len(object_files)} "
            f"/ Alpha OK"
        )

    print("=" * 60)
    print("Day 03 check: READY")


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[1. 실습에 필요한 기능 준비]

Python 기본 라이브러리 불러오기
OpenCV 불러오기
Pillow 불러오기
        ↓
현재 3일차 프로젝트 ROOT 경로 찾기
        ↓
앞에서 작성한 day03_common.py의
공통 함수 3개 불러오기

- asset_paths() : 입력 폴더 경로 찾기
- list_images() : 이미지 목록 가져오기
- read_classes() : Class 이름 읽기


[2. 입력 데이터의 검사 기준 설정]

검사할 Class 이름과 순서 지정

0 : drill
1 : fire_extinguisher
2 : wet_floor_sign
        ↓
본 실습 main 데이터의 예상 수량 설정

Background : 20장
Object     : 각 Class 5장


[3. 프로그램 실행 옵션 설정 — parse_args()]

검사할 데이터 종류를 전달받는다
        ↓
별도로 지정하지 않으면 main 선택
        ↓
main 또는 challenge만 허용한다


[4. 투명 배경 검사 함수 — has_alpha()]

객체 이미지 파일을 연다
        ↓
RGBA 형식으로 변환한다
        ↓
Alpha Channel을 가져온다
        ↓
Alpha 값의 최솟값과 최댓값을 확인한다
        ↓
투명하거나 반투명한 픽셀이 존재하고
보이는 픽셀도 존재하는가?
        ↓
YES → True 반환
NO  → False 반환


[5. 입력 데이터 검사 시작 — main()]

실행 옵션을 읽는다
        ↓
classes.txt에서 Class 목록을 읽는다
        ↓
Class 이름과 순서가 수업 기준과 일치하는가?
        ↓
NO → 오류 메시지를 출력하며 실행 중단
YES → 다음 검사 진행
        ↓
선택한 데이터의 Background / Object 경로 찾기
        ↓
해당 데이터의 예상 이미지 수량 가져오기


[6. Background 이미지 검사]

Background 폴더에서 이미지 목록 가져오기
        ↓
이미지 수량이 예상 수량과 일치하는가?
        ↓
NO → 오류 발생 및 실행 중단
YES → 다음 검사 진행
        ↓
각 Background 이미지를 OpenCV로 열어보기
        ↓
열리지 않는 이미지가 있는가?
        ↓
YES → 문제가 있는 파일명을 알려주고 실행 중단
NO  → 다음 검사 진행


[7. 객체 PNG 검사 — Class별 반복]

drill부터 wet_floor_sign까지
각 Class를 하나씩 검사한다
        ↓
해당 Class 폴더에서 이미지 목록 가져오기
        ↓
이미지 수량이 5장인가? (main 기준)
        ↓
NO → 오류 발생 및 실행 중단
YES → 다음 검사 진행
        ↓
각 객체 이미지에 has_alpha() 적용
        ↓
투명 영역과 보이는 영역이 모두 존재하는가?
        ↓
NO → 문제가 있는 파일명을 알려주고 실행 중단
YES → 해당 Class의 검사 결과 출력

Class 0 drill: 5 / Alpha OK
Class 1 fire_extinguisher: 5 / Alpha OK
Class 2 wet_floor_sign: 5 / Alpha OK


[8. 최종 결과 확인]

모든 Class 검사가 끝났는가?
        ↓
모든 검사 통과
        ↓
Day 03 check: READY 출력
        ↓
Cut-Paste 실습에 사용할 입력 데이터 준비 확인
```

### 이 코드에서 특히 중요한 함수

|함수|초보자가 이해할 핵심|
|---|---|
|`parse_args()`|어떤 데이터 폴더를 검사할지 선택|
|`has_alpha()`|객체 이미지에 투명한 부분과 보이는 부분이 모두 있는지 확인|
|`main()`|이미지 수량·Class 순서·파일 읽기·투명 영역을 종합 검사|

### 이 코드에서 알아두어야 할 점

① `EXPECTED_COUNTS`는 검사 기준입니다.

현재 `main` 데이터는 배경 20장, Class별 객체 이미지 5장이어야 통과합니다. 실제 파일 수가 다르면 오류가 발생합니다.

② `has_alpha()`는 투명 영역의 존재만 검사합니다.

객체 윤곽이 깔끔하게 분리되었는지, 배경이 자연스럽게 제거되었는지까지 확인하는 것은 아닙니다.

또한 코드는 이미지를 RGBA로 변환하여 검사하므로, 원본 파일이 PNG 형식인지 또는 실제 Alpha Channel을 저장하고 있는지 자체를 엄격하게 확인하지는 않습니다. 다만 투명한 픽셀과 보이는 픽셀이 모두 존재해야 통과합니다.

③ 오류가 발생하면 다음 검사로 진행하지 않습니다.

`RuntimeError`가 발생하면 프로그램이 중단되므로, 표시된 문제를 수정한 뒤 다시 실행해야 합니다.

④ `READY`는 입력 데이터 준비 상태만 의미합니다.

아직 Cut-Paste 이미지나 YOLO BBox를 생성한 것은 아닙니다.

> 핵심 정리: 이 코드는 배경과 객체 이미지를 합성하기 전에 입력 데이터의 수량, Class 순서, 이미지 읽기 가능 여부, 객체의 투명 영역을 자동으로 검사하여 이후 실습에서 발생할 수 있는 오류를 미리 발견하는 프로그램입니다.

---

본 실습에서는 다음을 실행합니다.

```bash
python scripts/00_check_day03.py \
  --asset-set main
```

정상이라면 다음과 비슷하게 표시됩니다.

```text
============================================================
Subject 11 - Day 03 Check
============================================================
Asset Set   : main
Backgrounds : 20
Class 0 drill: 5 / Alpha OK
Class 1 fire_extinguisher: 5 / Alpha OK
Class 2 wet_floor_sign: 5 / Alpha OK
============================================================
Day 03 check: READY
```

### 실행 결과 해석

출력 결과를 보면 `main` 데이터의 Background 이미지 20장과 객체 이미지가 Class별로 5장씩 정상적으로 확인되었습니다.

`Alpha OK`는 각 객체 이미지에 **투명한 영역과 보이는 영역이 모두 존재한다**는 의미입니다.

마지막의 **`Day 03 check: READY`**는 Class 순서, 이미지 수량, 배경 이미지 읽기, 
객체의 투명 영역 검사를 모두 통과했다는 뜻입니다.

따라서 Cut-Paste 합성을 위한 기본 입력 데이터가 준비되었습니다. 다만 객체의 윤곽 품질이나 합성 이미지의 자연스러움은 이후 실습에서 별도로 확인해야 합니다.

---

# PART 2. 합성 전에 Background·Alpha·BBox·YOLO Label의 역할을 이해합니다

---

# 8. 자동 검사한 Source를 실제로 어떻게 사용할지 이해합니다

앞의 7번 실습에서는 `Day 03 check: READY`를 확인했습니다. 
이는 **파일 수량·Class 순서·이미지 읽기·투명 영역의 기본 조건을 통과했다**는 뜻입니다.

하지만 아직 **이미지가 실제로 어떻게 생겼는지**, 그리고 **어떤 원리로 Detection Label을 만들 수 있는지**는 확인하지 않았습니다. 먼저 각 데이터의 역할을 이해하겠습니다.

| 구분 | 예시 파일 | Cut-Paste에서 하는 일 |
|---|---|---|
| Background JPG | `bg_001.jpg` | 새로운 합성 이미지의 바탕이 됩니다. |
| Object RGBA PNG | `drill_001.png` | 바탕 위에 붙일 객체입니다. 주변 배경은 투명하게 처리되어 있습니다. |
| `classes.txt` | `drill` 등 | 합성된 객체가 어떤 종류인지 나타내는 Class ID를 결정합니다. |

예를 들어 `bg_001.jpg` 위에 `drill_001.png`를 붙이면 **드릴이 배경 속에 놓인 새로운 이미지**를 만들 수 있습니다.

이때 객체의 위치·크기·방향을 바꾸면 같은 원본 이미지로도 서로 다른 장면을 생성할 수 있습니다. 단, 실제 작업 환경과 맞는 배경과 객체 시점을 사용하는 것이 중요합니다.

![[Pasted image 20261012153827.png]]

---

# 9. 투명 PNG에서 Alpha Channel이 하는 일을 이해합니다

객체 PNG를 붙일 때는 **드릴만 보이고, 드릴 사진의 사각형 배경은 보이지 않아야** 합니다. 이 역할을 하는 것이 Alpha Channel입니다.

일반 이미지의 RGB는 색상을 나타내고, RGBA의 `A(Alpha)`는 각 픽셀이 얼마나 보이는지를 나타냅니다.

| Alpha 값 | 의미 | 합성하면 어떻게 보이나요? |
|---|---|---|
| `0` | 완전 투명 | 해당 픽셀에서는 아래 Background가 보입니다. |
| `1~254` | 반투명 | 객체와 Background가 섞여 보입니다. |
| `255` | 완전 불투명 | 해당 픽셀에서는 객체가 보입니다. |

따라서 객체 테두리의 Alpha가 적절하게 준비되어 있으면 배경에 붙였을 때 사각형 테두리가 드러나지 않습니다.

**주의:** 앞에서 확인한 `Alpha OK`는 투명한 픽셀이 존재한다는 기본 검사 결과입니다. 드릴 윤곽이 깔끔한지, 불필요한 배경이 남아 있지 않은지는 **13번의 미리보기 이미지에서 눈으로 확인**해야 합니다.

오늘은 이미 투명하게 준비된 Object PNG를 사용하므로, 직접 배경을 제거하거나 Mask를 만드는 작업은 하지 않습니다.

![[Pasted image 20261012154403.png]]

---

# 10. Cut-Paste에서 Pixel BBox가 자동으로 생성되는 원리를 이해합니다

앞선 수업에서 객체검출을 위한 BBox와 라벨링을 경험했습니다.

이번에는 사람이 객체의 위치를 직접 표시하는 대신, **Cut-Paste 과정에서 BBox를 자동으로 계산하는 방법**을 알아보겠습니다.

위 그림에서 빨간색 사각형이 객체를 감싸는 최종 BBox입니다.

예를 들어 이미지 크기가 `640 × 480`이고, 드릴의 BBox 좌표가 다음과 같다면:

```text
왼쪽 위     (160, 120)
오른쪽 아래 (320, 240)

BBox 크기 = 160 × 120 픽셀
```

### Cut-Paste에서는 어떻게 자동 계산할까요?

```text
Object PNG
    ↓
투명 여백 제거
    ↓
크기 조절 (Scale)
    ↓
회전 (Rotation)
    ↓
회전 후 투명 여백 다시 제거
    ↓
Background에 배치
    ↓
최종 위치와 크기로 Pixel BBox 자동 계산
```

프로그램은 객체를 **어디에 붙였고, 최종 크기가 얼마인지** 알고 있으므로 사람이 다시 BBox를 그릴 필요가 없습니다.

이때 BBox는 회전된 객체의 윤곽을 따라가는 것이 아니라, **최종 객체를 감싸는 축 정렬 사각형(Axis-Aligned BBox)**입니다.

**핵심:** 3일차에서는 기존에 배운 BBox를 직접 라벨링하는 대신, 합성된 객체의 위치와 크기를 이용해 자동 생성하고 이후 YOLO Label로 변환합니다.

![[Pasted image 20261012154726.png]]


![[Pasted image 20261012160144.png]]

---

# 11. Pixel BBox를 YOLO Label로 바꾸는 이유를 이해합니다

앞에서 구한 Pixel BBox는 이미지 안에서의 위치를 **픽셀 단위**로 표현한 값입니다.

YOLO Detection Label은 이 좌표를 **이미지 전체 너비와 높이를 기준으로 0~1 범위로 정규화**하여 저장합니다. 이를 통해 이미지 크기가 달라도 같은 형식으로 위치를 표현할 수 있습니다.

YOLO Label에서 좌표를 정규화하는 이유:
이미지는 크기가 서로 다를 수 있습니다. 따라서 BBox 좌표를 픽셀 값 그대로 사용하면 
이미지 크기에 따라 좌표의 범위가 달라집니다.

이렇게 하면 이미지 해상도가 달라도 객체의 상대적인 위치와 크기를 일관된 형식으로 표현할 수 있습니다.

YOLO TXT 파일에는 객체 하나당 다음과 같이 한 줄을 기록합니다.

```text
class_id x_center y_center width height
```

앞의 `640 × 480` 이미지와 BBox 예시를 그대로 사용해 보겠습니다.

| YOLO 값 | 계산 방법 | 결과 |
|---|---|---:|
| `class_id` | `classes.txt`에서 드릴의 번호 | `0` |
| `x_center` | `((160 + 320) / 2) / 640` | `0.375` |
| `y_center` | `((120 + 240) / 2) / 480` | `0.375` |
| `width` | `(320 - 160) / 640` | `0.250` |
| `height` | `(240 - 120) / 480` | `0.250` |

따라서 YOLO Label은 다음과 같습니다.

```text
0 0.375000 0.375000 0.250000 0.250000
```

여기서 **첫 번째 값 `0`은 Class ID**이고, 뒤의 네 값은 객체 BBox의 **중심 위치와 너비·높이**입니다. `classes.txt`의 Class 순서는 앞에서 이미 정했으므로 다시 변경하지 않습니다.

이후 실습에서는 이미지를 합성할 때 이 계산도 자동으로 수행하여 **합성 이미지 JPG와 정답 Label TXT**를 함께 저장하게 됩니다.

![[Pasted image 20261012161421.png]]

![[Pasted image 20261012162047.png]]

---

# 12. 첫 합성 전에 Source를 눈으로 확인할 준비를 합니다

지금까지 입력 데이터가 준비되었는지 검사했고, 객체를 붙일 때 Alpha·BBox·YOLO Label이 어떤 역할을 하는지 이해했습니다.

그러나 자동 검사를 통과한 파일이라도 **실제 모양이나 윤곽 품질까지 정상이라고 단정할 수는 
없습니다.** 따라서 본격적으로 합성하기 전에 대표 이미지를 한 화면에 모아 보겠습니다.

다음 13번 실습에서는 Python으로 아래 여섯 장을 한 장의 미리보기 이미지로 만듭니다.

| 미리보기 대상 | 확인할 내용 |
|---|---|
| Background 대표 이미지 3장 | 이미지가 정상적으로 보이는지, 어떤 장소나 바닥이 배경인지 |
| `drill` 대표 PNG 1장 | 드릴 모양과 투명 처리 상태 |
| `fire_extinguisher` 대표 PNG 1장 | 소화기 모양과 투명 처리 상태 |
| `wet_floor_sign` 대표 PNG 1장 | 표지판 모양과 투명 처리 상태 |

객체 PNG는 **체크무늬 배경 위에 표시**하여 투명한 영역을 쉽게 알아볼 수 있도록 하겠습니다.

이 단계에서는 **아직 배경과 객체를 합성하거나 BBox를 만들지 않습니다.** 먼저 원본을 눈으로 확인한 뒤, 다음 PART에서 실제 Cut-Paste와 YOLO Label 생성을 진행합니다.

![[Pasted image 20261012162522.png]]

---

# 13. Source Data를 한 화면에서 확인하는 코드를 작성합니다

12번에서 살펴본 것처럼, 자동 검사를 통과한 이미지도 실제 모양과 투명 처리 상태는 눈으로 확인해야 합니다.

이번에는 **배경 이미지 3장과 각 Class의 대표 객체 PNG 1장씩, 총 6장을 하나의 미리보기 이미지로 만드는 코드**를 작성하겠습니다.

이 코드는 배경 이미지를 불러오고, 투명한 객체 PNG는 체크무늬 배경 위에 표시한 뒤, 6장을 한 화면에 배치하여 `asset_preview.jpg`로 저장합니다.

저장된 이미지를 열어 **배경의 상태, 객체의 모양, 투명 영역의 품질**을 직접 확인하겠습니다.

아직 Cut-Paste 합성이나 BBox 생성은 하지 않습니다.

**작성할 파일:** `**scripts/01_preview_assets.py**`

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import (
    Image,
    ImageDraw,
)


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    asset_paths,
    list_images,
    read_classes,
)


TILE_SIZE = 220


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    return parser.parse_args()


def checkerboard(
    size: int,
    cell: int = 20,
) -> Image.Image:
    image = Image.new(
        "RGB",
        (size, size),
        (245, 245, 245),
    )

    draw = ImageDraw.Draw(
        image
    )

    for y in range(
        0,
        size,
        cell,
    ):
        for x in range(
            0,
            size,
            cell,
        ):
            if (
                (x // cell)
                + (y // cell)
            ) % 2:
                draw.rectangle(
                    [
                        x,
                        y,
                        x + cell - 1,
                        y + cell - 1,
                    ],
                    fill=(
                        218,
                        218,
                        218,
                    ),
                )

    return image


def make_tile(
    path: Path,
    transparent: bool,
) -> Image.Image:
    tile = checkerboard(
        TILE_SIZE
    )

    with Image.open(path) as image:
        if transparent:
            obj = image.convert(
                "RGBA"
            )

            obj.thumbnail(
                (
                    TILE_SIZE - 20,
                    TILE_SIZE - 20,
                ),
                Image.Resampling.LANCZOS,
            )

            tile_rgba = tile.convert(
                "RGBA"
            )

            x = (
                TILE_SIZE
                - obj.width
            ) // 2

            y = (
                TILE_SIZE
                - obj.height
            ) // 2

            tile_rgba.alpha_composite(
                obj,
                dest=(x, y),
            )

            return tile_rgba.convert(
                "RGB"
            )

        image = image.convert("RGB")

        image.thumbnail(
            (
                TILE_SIZE - 20,
                TILE_SIZE - 20,
            ),
            Image.Resampling.LANCZOS,
        )

        x = (
            TILE_SIZE
            - image.width
        ) // 2

        y = (
            TILE_SIZE
            - image.height
        ) // 2

        tile.paste(
            image,
            (x, y),
        )

        return tile


def main() -> None:
    args = parse_args()

    paths = asset_paths(
        args.asset_set
    )

    classes = read_classes()

    backgrounds = list_images(
        paths["backgrounds"]
    )[:3]

    items = [
        (
            path,
            False,
            path.name,
        )
        for path in backgrounds
    ]

    for class_name in classes:
        files = list_images(
            paths["objects"]
            / class_name
        )

        if files:
            items.append(
                (
                    files[0],
                    True,
                    class_name,
                )
            )

    columns = 3
    rows = (
        len(items)
        + columns - 1
    ) // columns

    header = 35

    canvas = Image.new(
        "RGB",
        (
            columns * TILE_SIZE,
            rows
            * (
                TILE_SIZE
                + header
            ),
        ),
        "white",
    )

    draw = ImageDraw.Draw(
        canvas
    )

    for index, (
        path,
        transparent,
        label,
    ) in enumerate(items):
        row = index // columns
        column = index % columns

        x = column * TILE_SIZE

        y = row * (
            TILE_SIZE
            + header
        )

        tile = make_tile(
            path,
            transparent,
        )

        canvas.paste(
            tile,
            (x, y),
        )

        draw.text(
            (
                x + 5,
                y + TILE_SIZE + 8,
            ),
            label,
            fill="black",
        )

    output_dir = (
        ROOT
        / "results"
        / args.asset_set
    )

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    output_path = (
        output_dir
        / "asset_preview.jpg"
    )

    canvas.save(
        output_path,
        quality=92,
    )

    print(
        "Saved:",
        output_path,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

대량 합성을 시작하기 전에 **Background와 Object Source가 수업 의도대로 보이는지 한 번에 확인**하기 위해 작성합니다.

### 의사코드로 읽어보기

```text
[1. 입력 데이터와 공통 함수 준비]

3일차 프로젝트 경로 확인
        ↓
day03_common.py의 공통 함수 불러오기
        ↓
main 데이터 선택


[2. 미리보기에 사용할 이미지 선택]

Background 이미지 목록 가져오기
        ↓
앞에서부터 3장 선택
        +
classes.txt에서 Class 목록 읽기
        ↓
drill의 첫 번째 PNG 선택
fire_extinguisher의 첫 번째 PNG 선택
wet_floor_sign의 첫 번째 PNG 선택
        ↓
총 6장 준비


[3. 미리보기 이미지 만들기]

220 × 220 크기의 이미지 칸(Tile) 준비
        ↓
Background JPG
→ 비율을 유지하며 크기 조절
→ 이미지 칸 중앙에 배치

Object PNG
→ RGBA로 불러오기
→ 비율을 유지하며 크기 조절
→ 체크무늬 배경 위에 투명 합성
        ↓
각 이미지 아래에 이름 표시


[4. 여섯 장을 한 화면에 배치]

3열 × 2행의 미리보기 구성
        ↓
첫 번째 줄 : Background 3장
두 번째 줄 : Object PNG 3장
        ↓
하나의 미리보기 이미지 완성


[5. 결과 저장]

results/main/ 폴더 확인
        ↓
폴더가 없다면 생성
        ↓
asset_preview.jpg 저장
        ↓
저장 경로를 터미널에 출력
```

---

## 개발자가 확인해야 할 핵심 코드

모든 코드를 리뷰할 필요는 없습니다. 어떤 이미지를 선택하고, 어떻게 배치하며, 
어디에 저장하는지 이해하는 것이 중요합니다.

|핵심 코드|역할|어떤 상황에서 수정하나요?|
|---|---|---|
|`TILE_SIZE = 220`|미리보기 이미지 한 칸의 크기|이미지가 너무 작거나 클 때|
|`backgrounds = list_images(...)[:3]`|Background 중 앞의 3장 선택|확인할 배경 이미지 수를 변경할 때|
|`files[0]`|각 Class에서 첫 번째 객체 선택|다른 객체 이미지도 확인하고 싶을 때|
|`checkerboard()`|투명 영역을 확인할 체크무늬 생성|체크무늬 크기나 색상을 변경할 때|
|`make_tile()`|이미지를 크기 조절하고 중앙에 배치|표시 크기나 배치 방법을 변경할 때|
|`alpha_composite()`|객체 PNG를 체크무늬 위에 투명 합성|투명 PNG를 올바르게 표시할 때 사용하는 핵심 기능|
|`columns = 3`|한 줄에 이미지 3장 배치|미리보기의 열 개수를 변경할 때|
|`header = 35`|이미지 아래 이름을 표시할 공간|파일명이 길어서 공간이 부족할 때|
|`output_path`|미리보기 이미지 저장 위치|결과 파일명이나 저장 폴더를 변경할 때|

### ① `make_tile()`과 `alpha_composite()`가 핵심입니다

일반 Background JPG는 크기를 조절하여 이미지 칸에 배치합니다.

반면 Object PNG는 투명 영역을 확인해야 하므로 체크무늬 배경에 합성합니다.

```python
tile_rgba.alpha_composite(
    obj,
    dest=(x, y),
)
```

이 코드는 객체의 Alpha 값을 적용하여 객체가 보이는 부분만 체크무늬 위에 표시합니다.

여기서 사용하는 체크무늬는 실제 합성데이터의 배경이 아닙니다. 투명 영역을 눈으로 확인하기 위한 미리보기용 배경입니다.

### ② 지금은 대표 이미지 6장만 확인합니다

```python
backgrounds = list_images(
    paths["backgrounds"]
)[:3]
```

`[:3]`은 정렬된 Background 목록의 앞에서 세 장만 선택한다는 뜻입니다.

```python
files[0]
```

`files[0]`은 각 Class의 첫 번째 객체 이미지를 선택한다는 뜻입니다.

따라서 오늘 생성하는 미리보기는 전체 35장(Background 20장 + Object 15장)을 모두 확인하는 것이 아니라, 대표 6장을 확인하는 예비 점검입니다. 전체 이미지의 품질을 보장하는 검사는 아닙니다.

### ③ 미리보기 크기와 저장 위치를 확인합니다

현재 설정은 다음과 같습니다.

```python
TILE_SIZE = 220

columns = 3

header = 35
```

따라서 대표 이미지가 6장이라면 미리보기는 3열 × 2행으로 구성되며, 
최종 이미지 크기는 660 × 510픽셀입니다.

저장 위치는 다음과 같습니다.

```
results/main/asset_preview.jpg
```

같은 명령어를 다시 실행하면 기존 `asset_preview.jpg`는 덮어써집니다.

## 3. 이 코드에서 무엇을 확인해야 하나요?

생성된 `asset_preview.jpg`를 열어 다음 항목을 확인합니다.

- Background 이미지가 정상적으로 표시되는가?
- 드릴·소화기·표지판이 각각 올바른 모양으로 보이는가?
- 객체 주변에 흰색 사각형이나 불필요한 배경이 남아 있지 않은가?
- 객체의 윤곽이 잘리거나 심하게 흐릿하지 않은가?
    
핵심 정리: `01_preview_assets.py`는 이미지를 합성하는 코드가 아닙니다. 실제 Cut-Paste를 시작하기 전에 Background 3장과 Object PNG 3장을 한 화면에 표시하여 Source Data의 상태를 눈으로 확인하는 코드입니다.

---
본 실습에서는 다음을 실행합니다.

```bash
python scripts/01_preview_assets.py \
  --asset-set main
```

결과:

```text
results/main/asset_preview.jpg
```

![[Pasted image 20261012163319.png]]

### 실행 결과 해석

생성된 `asset_preview.jpg`를 확인한 결과, Background 3장과 Object PNG 3장이 정상적으로 표시되었습니다.

|확인 항목|실제 확인 결과|판정|
|---|---|---|
|Background 정상 표시|하늘, 헬리콥터, 장미와 악보 이미지가 정상적으로 보임|정상|
|Object 투명 영역|드릴·소화기·표지판 주변에 체크무늬 배경이 보임|정상|
|Class 구분|`drill`, `fire_extinguisher`, `wet_floor_sign`을 시각적으로 구분할 수 있음|정상|
|불필요한 사각 배경|객체 주변에 흰색 사각형 배경이 뚜렷하게 남아 있지 않음|정상|

최종 결과: 대표 Source 이미지 6장의 미리보기 검사를 통과했습니다.

---

# PART 3. 객체 한 개를 합성하고 BBox를 직접 확인합니다

---

# 14. 처음부터 여러 장을 만들지 않습니다

오늘의 첫 번째 합성은 다음 한 쌍으로 시작합니다.

```text
main/backgrounds/bg_001.jpg
    +
main/objects/drill/drill_001.png
```

먼저 한 장에서 다음 전체 흐름을 확인합니다.

```text
Background 읽기
        ↓
RGBA Object 읽기
        ↓
Alpha 영역 Crop
        ↓
Scale 적용
        ↓
Rotation 적용
        ↓
회전 후 Alpha 영역 다시 Crop
        ↓
최종 Object 크기 확인
        ↓
Position 결정
        ↓
Cut-Paste
        ↓
최종 Object 기준 Pixel BBox
        ↓
YOLO Label
        ↓
BBox Visualization
```

한 장이 정확하지 않으면 100장을 자동 생성해도 의미가 없습니다.

---

# 15. Single Cut-Paste 코드를 작성합니다

**파일: `scripts/02_cutpaste_single.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import (
    Image,
    ImageDraw,
)


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    asset_paths,
    read_classes,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--background",
        default="bg_001.jpg",
    )

    parser.add_argument(
        "--class-name",
        default="drill",
    )

    parser.add_argument(
        "--object",
        default="drill_001.png",
    )

    parser.add_argument(
        "--scale",
        type=float,
        default=0.18,
    )

    parser.add_argument(
        "--rotation",
        type=float,
        default=10.0,
    )

    parser.add_argument(
        "--x-ratio",
        type=float,
        default=0.50,
    )

    parser.add_argument(
        "--y-ratio",
        type=float,
        default=0.50,
    )

    return parser.parse_args()


def crop_to_alpha(
    image: Image.Image,
) -> Image.Image:
    image = image.convert(
        "RGBA"
    )

    alpha = image.getchannel(
        "A"
    )

    bbox = alpha.getbbox()

    if bbox is None:
        raise RuntimeError(
            "보이는 Object Pixel이 없습니다."
        )

    return image.crop(
        bbox
    )


def to_yolo(
    x1,
    y1,
    x2,
    y2,
    image_width,
    image_height,
):
    x_center = (
        (x1 + x2) / 2
    ) / image_width

    y_center = (
        (y1 + y2) / 2
    ) / image_height

    width = (
        x2 - x1
    ) / image_width

    height = (
        y2 - y1
    ) / image_height

    return (
        x_center,
        y_center,
        width,
        height,
    )


def main() -> None:
    args = parse_args()

    if not 0 < args.scale < 1:
        raise ValueError(
            "--scale은 0과 1 사이여야 합니다."
        )

    if not 0 <= args.x_ratio <= 1:
        raise ValueError(
            "--x-ratio는 0~1이어야 합니다."
        )

    if not 0 <= args.y_ratio <= 1:
        raise ValueError(
            "--y-ratio는 0~1이어야 합니다."
        )

    paths = asset_paths(
        args.asset_set
    )

    classes = read_classes()

    if args.class_name not in classes:
        raise ValueError(
            f"알 수 없는 Class: "
            f"{args.class_name}"
        )

    class_id = classes.index(
        args.class_name
    )

    background_path = (
        paths["backgrounds"]
        / args.background
    )

    object_path = (
        paths["objects"]
        / args.class_name
        / args.object
    )

    if not background_path.exists():
        raise FileNotFoundError(
            background_path
        )

    if not object_path.exists():
        raise FileNotFoundError(
            object_path
        )

    background = Image.open(
        background_path
    ).convert("RGBA")

    obj = Image.open(
        object_path
    ).convert("RGBA")

    obj = crop_to_alpha(obj)

    bg_width, bg_height = (
        background.size
    )

    obj_width, obj_height = (
        obj.size
    )

    target_width = max(
        8,
        int(
            bg_width
            * args.scale
        ),
    )

    ratio = (
        target_width
        / obj_width
    )

    target_height = max(
        8,
        int(
            obj_height
            * ratio
        ),
    )

    obj = obj.resize(
        (
            target_width,
            target_height,
        ),
        Image.Resampling.LANCZOS,
    )

    obj = obj.rotate(
        args.rotation,
        expand=True,
        resample=(
            Image.Resampling.BICUBIC
        ),
    )

    obj = crop_to_alpha(obj)

    obj_width, obj_height = (
        obj.size
    )

    if (
        obj_width >= bg_width
        or obj_height >= bg_height
    ):
        raise RuntimeError(
            "Object가 Background보다 큽니다."
        )

    max_x = (
        bg_width
        - obj_width
    )

    max_y = (
        bg_height
        - obj_height
    )

    x1 = int(
        max_x
        * args.x_ratio
    )

    y1 = int(
        max_y
        * args.y_ratio
    )

    x2 = x1 + obj_width
    y2 = y1 + obj_height

    composite = background.copy()

    composite.alpha_composite(
        obj,
        dest=(x1, y1),
    )

    output_dir = (
        ROOT
        / "results"
        / args.asset_set
        / "single"
    )

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    image_path = (
        output_dir
        / "single_cutpaste.jpg"
    )

    label_path = (
        output_dir
        / "single_cutpaste.txt"
    )

    bbox_path = (
        output_dir
        / "single_cutpaste_bbox.jpg"
    )

    composite.convert(
        "RGB"
    ).save(
        image_path,
        quality=95,
    )

    (
        x_center,
        y_center,
        bbox_width,
        bbox_height,
    ) = to_yolo(
        x1,
        y1,
        x2,
        y2,
        bg_width,
        bg_height,
    )

    label_text = (
        f"{class_id} "
        f"{x_center:.6f} "
        f"{y_center:.6f} "
        f"{bbox_width:.6f} "
        f"{bbox_height:.6f}"
    )

    label_path.write_text(
        label_text + "\n",
        encoding="utf-8",
    )

    visualized = composite.copy()

    draw = ImageDraw.Draw(
        visualized
    )

    draw.rectangle(
        [
            x1,
            y1,
            x2,
            y2,
        ],
        outline="red",
        width=4,
    )

    draw.text(
        (
            x1,
            max(
                0,
                y1 - 18,
            ),
        ),
        args.class_name,
        fill="red",
    )

    visualized.convert(
        "RGB"
    ).save(
        bbox_path,
        quality=95,
    )

    print(
        "Background:",
        background_path.name,
    )

    print(
        "Object    :",
        object_path.name,
    )

    print(
        "Class ID  :",
        class_id,
    )

    print(
        "Pixel BBox:",
        [
            x1,
            y1,
            x2,
            y2,
        ],
    )

    print(
        "YOLO Label:",
        label_text,
    )

    print(
        "Saved:",
        output_dir,
    )


if __name__ == "__main__":
    main()
```

### 이 코드는 왜 작성하나요?

대량 생성 전에 **Cut-Paste와 BBox 계산이 정확히 연결되는지 한 장으로 검증**하기 위해 작성합니다.

이 코드의 `crop_to_alpha()`는 Object의 투명 여백을 제거합니다. Scale을 적용한 뒤 회전하면 다시 투명 여백이 생길 수 있으므로, 회전 후에도 한 번 더 Alpha 영역을 Crop합니다. 그 다음의 `obj_width`, `obj_height`가 최종 BBox 크기가 됩니다.

따라서 Single 실습에서 저장되는 BBox는 **회전 전 원본 PNG 사각형이 아니라 최종 합성 Object 기준 BBox**입니다.

### 의사코드로 읽어보기

```text
[1. 실습에 사용할 데이터와 합성 조건 설정]

공통 경로 관리 함수 불러오기
        ↓
Background 이미지 선택
→ bg_001.jpg

Object PNG 선택
→ drill_001.png

Class 선택
→ drill (Class ID: 0)

합성 조건 설정
→ Scale      : 0.18
→ Rotation   : 10°
→ X Position : 0.50
→ Y Position : 0.50


[2. 입력값과 파일 확인]

Scale과 Position 값이 허용 범위에 있는지 확인
        ↓
classes.txt에서 Class 목록 읽기
        ↓
선택한 Class가 존재하는지 확인
        ↓
Background와 Object 파일이 존재하는지 확인
        ↓
문제가 있다면 오류 발생 및 실행 중단


[3. Background와 Object 불러오기]

Background JPG 불러오기
        ↓
Object PNG 불러오기
        ↓
두 이미지를 RGBA 형식으로 변환
        ↓
Object의 Alpha Channel 확인


[4. Object의 투명 여백 제거]

Object의 Alpha Channel 가져오기
        ↓
Alpha 값이 0보다 큰 영역 찾기
        ↓
객체를 감싸는 최소 사각형 계산
        ↓
바깥쪽 투명 여백 제거 (Crop)
        ↓
실제 객체가 보이는 영역만 남기기


[5. Object 크기 조절 — Scale]

Background의 가로·세로 크기 확인
        ↓
Background 가로 길이 × Scale 계산
        ↓
Object의 목표 가로 크기 결정
        ↓
원본 가로·세로 비율을 유지하며
Object 크기 조절


[6. Object 회전 — Rotation]

Object를 지정한 각도만큼 회전
→ 10°
        ↓
회전하면서 생긴 투명 여백 확인
        ↓
Alpha 기준으로 투명 여백 다시 제거
        ↓
회전이 완료된 최종 Object 크기 확인
        ↓
Object가 Background 안에 들어가는지 검사


[7. Background에 Object 배치]

Background 크기와 최종 Object 크기 비교
        ↓
객체가 이동할 수 있는 최대 X/Y 범위 계산
        ↓
X Position = 0.50
Y Position = 0.50 적용
        ↓
객체를 붙일 왼쪽 위 좌표 결정

x1 = 객체의 시작 X 좌표
y1 = 객체의 시작 Y 좌표
        ↓
Background에 Object를 Alpha 합성
        ↓
새로운 Synthetic Image 생성


[8. Pixel BBox 자동 계산]

객체가 배치된 시작 좌표 확인
→ x1, y1
        ↓
최종 Object의 가로·세로 크기 확인
        ↓
x2 = x1 + Object Width
y2 = y1 + Object Height
        ↓
최종 Pixel BBox 생성

(x1, y1, x2, y2)


[9. YOLO Label 자동 생성]

Pixel BBox 가져오기
        ↓
이미지 전체 Width / Height 확인
        ↓
BBox 중심 좌표 계산
        ↓
중심 좌표와 너비·높이를
이미지 크기로 나누어 정규화
        ↓
Class ID와 정규화 좌표 연결

class_id x_center y_center width height
        ↓
YOLO TXT 파일 저장


[10. BBox 시각화]

완성된 Synthetic Image 복사
        ↓
Pixel BBox 좌표를 사용하여
객체 주변에 빨간색 사각형 그리기
        ↓
BBox 위에 Class 이름 표시
→ drill
        ↓
BBox 확인용 이미지 저장


[11. 최종 결과 저장 및 출력]

합성 이미지 저장
→ single_cutpaste.jpg

YOLO Label 저장
→ single_cutpaste.txt

BBox 시각화 이미지 저장
→ single_cutpaste_bbox.jpg
        ↓
터미널에 결과 출력

Background 파일명
Object 파일명
Class ID
Pixel BBox
YOLO Label
저장 위치
```

### 이 코드에서 가장 중요한 핵심

|핵심 함수 또는 코드|역할|
|---|---|
|`parse_args()`|사용할 이미지와 Scale·Rotation·Position 설정|
|`crop_to_alpha()`|객체 주변의 투명 여백 제거|
|`resize()`|객체 크기 조절|
|`rotate()`|객체 회전|
|`alpha_composite()`|Background에 객체 합성|
|`x1, y1, x2, y2`|최종 객체 위치와 크기를 이용한 Pixel BBox 계산|
|`to_yolo()`|Pixel BBox를 YOLO 정규화 좌표로 변환|
|`draw.rectangle()`|BBox를 이미지 위에 그려 결과 확인|

### 최종 생성되는 파일

```
results/main/single/
│
├── single_cutpaste.jpg
│   └── 배경과 드릴이 합성된 이미지
│
├── single_cutpaste.txt
│   └── YOLO Detection 정답 Label
│
└── single_cutpaste_bbox.jpg
    └── 빨간색 BBox가 표시된 확인용 이미지
```

핵심 정리: 이 실습에서 중요한 것은 사람이 직접 BBox를 그리지 않는다는 점입니다. 프로그램이 객체의 투명 여백을 제거하고 크기·회전·위치를 결정한 뒤, 최종 객체의 위치와 크기를 이용해 BBox와 YOLO Label까지 자동으로 생성합니다.

---

# 16. 본 실습 Single Cut-Paste를 실행합니다.

앞에서 작성한 `02_cutpaste_single.py`를 실행하여 **배경 이미지에 드릴을 합성하고, Pixel BBox와 YOLO Label이 자동으로 생성되는지** 확인하겠습니다.

Single Cut-Paste를 실행합니다

```bash
python scripts/02_cutpaste_single.py \
  --asset-set main \
  --background bg_001.jpg \
  --class-name drill \
  --object drill_001.png \
  --scale 0.18 \
  --rotation 10 \
  --x-ratio 0.50 \
  --y-ratio 0.50
```

이번 실습에서는 하늘 배경인 `bg_001.jpg`에 `drill_001.png`를 합성합니다.

| 실행 옵션 | 적용한 조건 |
|---|---|
| `--scale 0.18` | 회전 전 객체의 가로 크기를 배경 너비의 약 18%로 조절 |
| `--rotation 10` | 객체를 10도 회전 |
| `--x-ratio 0.50` | 객체를 배치할 수 있는 가로 범위의 중앙 |
| `--y-ratio 0.50` | 객체를 배치할 수 있는 세로 범위의 중앙 |

---

# 17. Single 결과를 반드시 직접 확인합니다

생성된 파일을 확인합니다

```text
results/main/single/
├─ single_cutpaste.jpg
├─ single_cutpaste.txt
└─ single_cutpaste_bbox.jpg
```

![[Pasted image 20261012164002.png]]

| 파일 | 실제 생성된 결과 |
|---|---|
| `single_cutpaste.jpg` | 하늘 배경에 노란색 드릴이 합성된 이미지 |
| `single_cutpaste.txt` | 드릴의 Class ID와 YOLO BBox 좌표 |
| `single_cutpaste_bbox.jpg` | 드릴 주위에 빨간색 BBox와 Class 이름이 표시된 이미지 |
### 실제 생성된 이미지를 해석합니다

**첫 번째 이미지 — `single_cutpaste.jpg`**

하늘 배경 위에 드릴 한 개가 합성되었습니다.

드릴 주변에 불필요한 흰색 사각형이 나타나지 않으므로 Alpha Channel을 이용한 투명 합성이 정상적으로 적용된 것을 확인할 수 있습니다.

**두 번째 이미지 — `single_cutpaste_bbox.jpg`**

회전된 드릴 주위에 빨간색 사각형과 `drill`이라는 Class 이름이 표시되었습니다.

이 결과는 프로그램이 객체의 최종 위치와 크기를 이용하여 **Pixel BBox를 자동으로 계산했음**을 보여줍니다.

### 실제 YOLO Label을 확인합니다

생성된 `single_cutpaste.txt`에는 다음 값이 저장되었습니다.

```text
0 0.499691 0.500000 0.145062 0.474074
```

| YOLO 값 | 의미 | 실제 결과 해석 |
|---|---|---|
| `0` | Class ID | 드릴 |
| `0.499691` | BBox 중심 X | 이미지 가로의 약 50% 위치 |
| `0.500000` | BBox 중심 Y | 이미지 세로의 50% 위치 |
| `0.145062` | BBox 너비 | 이미지 전체 너비의 약 14.5% |
| `0.474074` | BBox 높이 | 이미지 전체 높이의 약 47.4% |

객체가 화면 중앙 부근에 배치되어 있으므로 중심 좌표가 약 `0.5, 0.5`로 계산되었습니다.

또한 드릴이 세로 방향으로 길게 놓여 있어 BBox의 높이가 너비보다 크게 나타났습니다.

이를 통해 **객체의 최종 배치 결과가 YOLO Label에 정상적으로 반영되었음**을 확인할 수 있습니다.

### 합성 이미지의 품질도 함께 판단합니다

그런데 실제 이미지를 보면 드릴이 하늘에 떠 있는 모습입니다.

코드는 정상적으로 실행되었지만, 현실에서 드릴이 놓이는 환경으로는 부자연스럽습니다.

| 확인 항목 | 실제 결과 |
|---|---|
| Cut-Paste 합성 | 성공 |
| Alpha 투명 처리 | 정상적으로 보임 |
| Pixel BBox 자동 생성 | 정상적으로 보임 |
| YOLO Label 저장 | 성공 |
| 합성 장면의 현실성 | 부족 — 드릴이 하늘에 떠 있음 |
| 실제 학습데이터 활용 | 현재 상태에서는 권장하지 않음 |

이번 실습의 목적은 실제 학습데이터의 품질을 확보하는 것이 아니라, **합성 과정에서 BBox와 YOLO Label이 자동으로 생성되는 기술을 확인하는 것**입니다.

실제 학습데이터를 제작할 때는 드릴이 놓일 수 있는 작업대나 공장 등의 배경을 선택하고, 크기·방향·조명도 현실적인지 확인해야 합니다.

### 실행 결과를 정리합니다

이번 실습에서는 배경 이미지 한 장과 객체 PNG 한 장을 사용하여 합성 이미지, Pixel BBox, YOLO Label을 자동으로 생성하는 데 성공했습니다.

다만 기술적으로 합성에 성공했다고 해서 반드시 학습에 적합한 데이터가 되는 것은 아닙니다.

**핵심 정리:** Cut-Paste를 이용하면 객체검출용 BBox와 Label을 자동 생성할 수 있지만, 실제 학습데이터로 사용하려면 합성된 장면의 현실성과 BBox 정확성을 함께 검수해야 합니다.

이제 한 장에서 확인한 Cut-Paste 과정을 여러 이미지에 자동으로 반복하여 합성 Detection 데이터셋을 만들어 보겠습니다.

---

# PART 4. 여러 Synthetic Detection Image와 YOLO Label을 자동 생성합니다

---

# 18. 한 장씩 만들던 Cut-Paste를 여러 장으로 자동화합니다

앞선 실습에서는 배경에 드릴 한 개를 합성하고 BBox와 YOLO Label을 자동 생성했습니다.
이번에는 배경·객체·위치·크기·회전을 무작위로 변경하면서 같은 작업을 120번 반복하겠습니다.
세 가지 Class는 각각 40장씩 생성하여 데이터 수의 균형을 맞춥니다.
이렇게 하면 적은 원본 이미지로도 다양한 Detection 학습 후보 데이터를 만들 수 있습니다.

![[Pasted image 20261013104030.png]]

---

# 19. 합성 이미지의 생성 조건을 Metadata로 기록합니다

18번에서는 배경·객체·위치·크기·회전을 변경하여 120장의 합성 이미지를 만들기로 했습니다.

그런데 나중에 특정 이미지가 부자연스럽다면 **어떤 배경과 객체를 사용했고, 얼마나 회전시켰는지** 알아야 문제를 수정할 수 있습니다.

이를 위해 Python이 이미지 생성 조건을 `Metadata CSV`에 자동으로 기록하도록 하겠습니다.

### 실제로 어떤 정보를 저장하나요?

예를 들어 `synthetic_00023.jpg`라는 이미지를 만들었다고 가정하겠습니다.

|기록 항목|저장 예시|의미|
|---|---|---|
|`file`|`synthetic_00023.jpg`|생성된 이미지 이름|
|`background`|`bg_014.jpg`|사용한 배경|
|`object`|`fire_extinguisher_003.png`|사용한 객체 PNG|
|`class_id`|`1`|소화기|
|`scale`|`0.143`|객체 크기 조절 비율|
|`rotation`|`-7.2`|객체 회전각|
|`x1, y1, x2, y2`|실제 계산된 좌표|최종 Pixel BBox|
|`global_seed`|`42`|무작위 생성 재현을 위한 기준값|
|`approved`|`pending`|아직 검수하지 않은 상태|

위 값은 이해를 돕기 위한 예시이며, 실제 CSV에는 위치 비율, YOLO BBox, 이미지별 Seed 등도 함께 저장됩니다.

### Metadata를 직접 작성해야 하나요?

직접 작성하지 않고 **20번 Python 코드가 합성 이미지를 만들 때 자동으로 기록합니다.**

```
합성 이미지 1장 생성
        ↓
JPG 이미지 저장
        +
YOLO Label TXT 저장
        +
Metadata CSV에 생성 조건 1행 추가
        ↓
120장 생성 완료
        ↓
Metadata 120행 저장
```

**저장 파일:** `**reports/main/baseline_metadata.csv**`

이후 120장을 생성한 뒤 다음 명령어로 기록을 확인할 수 있습니다.

```bash
head -n 3 reports/main/baseline_metadata.csv
```

**핵심:** YOLO TXT는 모델 학습에 필요한 정답이고, Metadata CSV는 **합성 이미지가 어떤 조건으로 만들어졌는지 추적하기 위한 기록**입니다.

나중에 문제가 있는 이미지를 발견하면 Metadata에서 생성 조건을 확인하고 수정 방향을 판단할 수 있습니다.

---

# 20. Batch Cut-Paste 생성 코드를 작성합니다

이제 한 장씩 합성하던 과정을 자동으로 반복하는 Python 코드를 작성하겠습니다.

**이 코드는 무엇을 하나요?**

```text
Background와 Object 무작위 선택
              ↓
위치 · 크기 · 회전 변경
              ↓
Cut-Paste 합성
              ↓
Pixel BBox와 YOLO Label 자동 생성
              ↓
이미지 JPG + Label TXT 저장
              ↓
생성 조건 Metadata CSV 기록
              ↓
120장 생성 완료
```

생성한 이미지와 Label은 `dataset/main/baseline/`에, Metadata는 `reports/main/baseline_metadata.csv`에 저장됩니다.

이번 코드에서는 **생성할 이미지 수, 크기, 회전 범위 등을 실행할 때 변경**할 수 있도록 작성합니다.

먼저 소량의 이미지를 생성하여 코드가 정상적으로 동작하는지 확인한 다음, 전체 120장을 생성하겠습니다.

**파일: `scripts/03_generate_cutpaste_dataset.py`**

```python
from __future__ import annotations

import argparse
import csv
import math
import random
import sys
from pathlib import Path

from PIL import Image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    asset_paths,
    experiment_paths,
    list_images,
    read_classes,
    reset_directory,
)


# --------------------------------------------------
# 1. 실행 옵션
# --------------------------------------------------

def parse_args():
    parser = argparse.ArgumentParser(
        description="Generate balanced Cut-Paste images, YOLO labels, and metadata."
    )

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=["main", "challenge"],
    )

    parser.add_argument(
        "--tag",
        default="baseline",
    )

    parser.add_argument(
        "--count",
        type=int,
        default=120,
    )

    parser.add_argument(
        "--seed",
        type=int,
        default=42,
    )

    parser.add_argument(
        "--scale-min",
        type=float,
        default=0.08,
    )

    parser.add_argument(
        "--scale-max",
        type=float,
        default=0.22,
    )

    parser.add_argument(
        "--rotation",
        type=float,
        default=15.0,
    )

    return parser.parse_args()


# --------------------------------------------------
# 2. 객체 PNG의 투명 여백 제거
# --------------------------------------------------

def crop_to_alpha(image: Image.Image):
    image = image.convert("RGBA")

    alpha = image.getchannel("A")
    bbox = alpha.getbbox()

    if bbox is None:
        return None

    return image.crop(bbox)


# --------------------------------------------------
# 3. Pixel BBox → YOLO BBox 변환
# --------------------------------------------------

def to_yolo(
    x1,
    y1,
    x2,
    y2,
    image_width,
    image_height,
):
    x_center = (
        ((x1 + x2) / 2) / image_width
    )

    y_center = (
        ((y1 + y2) / 2) / image_height
    )

    width = (
        (x2 - x1) / image_width
    )

    height = (
        (y2 - y1) / image_height
    )

    return (
        x_center,
        y_center,
        width,
        height,
    )


# --------------------------------------------------
# 4. Batch 생성
# --------------------------------------------------

def main() -> None:
    args = parse_args()

    # 실행 조건 검사
    if args.count < 1:
        raise ValueError(
            "--count는 1 이상이어야 합니다."
        )

    if not (
        0 < args.scale_min
        <= args.scale_max
        < 1
    ):
        raise ValueError(
            "Scale 범위는 0 < min <= max < 1이어야 합니다."
        )

    if not (
        math.isfinite(args.scale_min)
        and math.isfinite(args.scale_max)
    ):
        raise ValueError(
            "Scale 값은 유한한 숫자여야 합니다."
        )

    if (
        not math.isfinite(args.rotation)
        or args.rotation < 0
    ):
        raise ValueError(
            "--rotation은 0 이상의 유한한 숫자여야 합니다."
        )

    # Source 및 출력 경로
    source = asset_paths(
        args.asset_set
    )

    output = experiment_paths(
        args.asset_set,
        args.tag,
    )

    # Class 목록
    classes = read_classes()

    if not classes:
        raise RuntimeError(
            "classes.txt에 Class가 없습니다."
        )

    # Background 목록
    backgrounds = sorted(
        list_images(source["backgrounds"]),
        key=lambda p: p.name,
    )

    if not backgrounds:
        raise RuntimeError(
            "Background가 없습니다."
        )

    # Class별 Object PNG 목록
    object_map = {}

    for class_name in classes:
        files = sorted(
            list_images(
                source["objects"] / class_name
            ),
            key=lambda p: p.name,
        )

        if not files:
            raise RuntimeError(
                f"{class_name} Object가 없습니다."
            )

        object_map[class_name] = files

    # 출력 폴더 초기화
    # 같은 tag로 재실행하면 기존 이미지와 Label이 삭제됩니다.
    reset_directory(
        output["images"]
    )

    reset_directory(
        output["labels"]
    )

    output["report"].mkdir(
        parents=True,
        exist_ok=True,
    )

    # Class 생성 개수를 균형 있게 구성
    schedule_rng = random.Random(
        args.seed
    )

    class_schedule = [
        classes[index % len(classes)]
        for index in range(args.count)
    ]

    schedule_rng.shuffle(
        class_schedule
    )

    rows = []

    # --------------------------------------------------
    # 5. 이미지 생성 반복
    # --------------------------------------------------

    for sample_index, class_name in enumerate(
        class_schedule
    ):
        class_id = classes.index(
            class_name
        )

        # 이미지마다 고정된 Seed
        sample_seed = (
            args.seed * 100_000
            + sample_index
        )

        sample_rng = random.Random(
            sample_seed
        )

        # Background와 Object 선택
        background_path = sample_rng.choice(
            backgrounds
        )

        object_path = sample_rng.choice(
            object_map[class_name]
        )

        # 무작위 합성 조건
        scale = sample_rng.uniform(
            args.scale_min,
            args.scale_max,
        )

        rotation_unit = sample_rng.uniform(
            -1.0,
            1.0,
        )

        x_ratio = sample_rng.random()
        y_ratio = sample_rng.random()

        rotation = (
            rotation_unit * args.rotation
        )

        # 이미지 불러오기
        with Image.open(
            background_path
        ) as image:
            background = image.convert(
                "RGBA"
            )

        with Image.open(
            object_path
        ) as image:
            obj = image.convert(
                "RGBA"
            )

        # 객체 투명 여백 제거
        obj = crop_to_alpha(
            obj
        )

        if obj is None:
            raise RuntimeError(
                "보이는 Object Pixel이 없습니다: "
                f"{object_path}"
            )

        bg_width, bg_height = (
            background.size
        )

        obj_width, obj_height = (
            obj.size
        )

        # --------------------------------------------------
        # 6. 객체 Scale 계산 — 오류 수정 부분
        # --------------------------------------------------

        # 원래 요청한 Scale
        requested_width = max(
            8,
            int(bg_width * scale),
        )

        requested_ratio = (
            requested_width / obj_width
        )

        # 가로와 세로를 모두 고려한 안전한 Scale
        # 회전해도 배경 안에 들어가도록 대각선 길이 기준 사용
        diagonal = math.hypot(
            obj_width,
            obj_height,
        )

        safe_ratio = (
            0.90 * min(bg_width, bg_height)
        ) / diagonal

        # 요청 Scale과 안전 Scale 중 작은 값 적용
        ratio = min(
            requested_ratio,
            safe_ratio,
        )

        target_width = max(
            1,
            round(obj_width * ratio),
        )

        target_height = max(
            1,
            round(obj_height * ratio),
        )

        # 실제 적용된 Scale
        effective_scale = (
            target_width / bg_width
        )

        # 객체 크기 변경
        obj = obj.resize(
            (
                target_width,
                target_height,
            ),
            Image.Resampling.LANCZOS,
        )

        # --------------------------------------------------
        # 7. Rotation 적용
        # --------------------------------------------------

        obj = obj.rotate(
            rotation,
            expand=True,
            resample=Image.Resampling.BICUBIC,
        )

        # 회전 후 투명 여백 다시 제거
        obj = crop_to_alpha(
            obj
        )

        if obj is None:
            raise RuntimeError(
                "회전 후 보이는 Object Pixel이 없습니다: "
                f"{object_path}"
            )

        obj_width, obj_height = (
            obj.size
        )

        # 회전 후 크기 검사
        if (
            obj_width > bg_width
            or obj_height > bg_height
        ):
            raise RuntimeError(
                "Object가 Background 안에 들어가지 않습니다: "
                f"sample={sample_index + 1}, "
                f"object={object_path.name}"
            )

        # --------------------------------------------------
        # 8. 객체 위치와 Pixel BBox 계산
        # --------------------------------------------------

        max_x = (
            bg_width - obj_width
        )

        max_y = (
            bg_height - obj_height
        )

        x1 = round(
            max_x * x_ratio
        )

        y1 = round(
            max_y * y_ratio
        )

        x2 = x1 + obj_width
        y2 = y1 + obj_height

        # --------------------------------------------------
        # 9. Cut-Paste 합성
        # --------------------------------------------------

        composite = background.copy()

        composite.alpha_composite(
            obj,
            dest=(x1, y1),
        )

        # 생성 파일 이름
        stem = (
            f"synthetic_{sample_index + 1:05d}"
        )

        image_path = (
            output["images"]
            / f"{stem}.jpg"
        )

        label_path = (
            output["labels"]
            / f"{stem}.txt"
        )

        # 합성 JPG 저장
        composite.convert(
            "RGB"
        ).save(
            image_path,
            quality=95,
        )

        # --------------------------------------------------
        # 10. YOLO Label 자동 생성
        # --------------------------------------------------

        (
            x_center,
            y_center,
            bbox_width,
            bbox_height,
        ) = to_yolo(
            x1,
            y1,
            x2,
            y2,
            bg_width,
            bg_height,
        )

        label_text = (
            f"{class_id} "
            f"{x_center:.6f} "
            f"{y_center:.6f} "
            f"{bbox_width:.6f} "
            f"{bbox_height:.6f}"
        )

        label_path.write_text(
            label_text + "\n",
            encoding="utf-8",
        )

        # --------------------------------------------------
        # 11. Metadata 기록 — 오류 수정 부분
        # --------------------------------------------------

        rows.append(
            {
                "file": image_path.name,

                "asset_set": args.asset_set,

                "tag": args.tag,

                "background": background_path.name,

                "object": object_path.name,

                "class_id": class_id,

                "class_name": class_name,

                # 요청한 Scale
                "scale": round(
                    scale,
                    5,
                ),

                # 실제 적용된 Scale
                "effective_scale": round(
                    effective_scale,
                    5,
                ),

                "rotation_limit": args.rotation,

                "rotation_unit": round(
                    rotation_unit,
                    6,
                ),

                "rotation": round(
                    rotation,
                    3,
                ),

                "x_ratio": round(
                    x_ratio,
                    6,
                ),

                "y_ratio": round(
                    y_ratio,
                    6,
                ),

                "x1": x1,

                "y1": y1,

                "x2": x2,

                "y2": y2,

                "yolo_x_center": round(
                    x_center,
                    6,
                ),

                "yolo_y_center": round(
                    y_center,
                    6,
                ),

                "yolo_width": round(
                    bbox_width,
                    6,
                ),

                "yolo_height": round(
                    bbox_height,
                    6,
                ),

                "global_seed": args.seed,

                "sample_seed": sample_seed,

                "approved": "pending",
            }
        )

    # --------------------------------------------------
    # 12. Metadata CSV 저장
    # --------------------------------------------------

    metadata_path = (
        output["report"]
        / f"{args.tag}_metadata.csv"
    )

    with metadata_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:

        writer = csv.DictWriter(
            file,
            fieldnames=list(
                rows[0].keys()
            ),
        )

        writer.writeheader()
        writer.writerows(rows)

    # --------------------------------------------------
    # 13. 생성 결과 출력
    # --------------------------------------------------

    print("=" * 60)
    print("Cut-Paste Dataset Generated")
    print("=" * 60)

    print(
        "Asset Set :", args.asset_set
    )

    print(
        "Tag       :", args.tag
    )

    print(
        "Generated :", len(rows)
    )

    print(
        "Images    :", output["images"]
    )

    print(
        "Labels    :", output["labels"]
    )

    print(
        "Metadata  :", metadata_path
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 이해하기

```
[1. 실험 조건 설정]

사용할 Source → main
실험 이름 → baseline
생성할 이미지 → 120장
Scale 범위 → 0.08 ~ 0.22
Rotation 범위 → -15° ~ +15°
Seed → 42
        ↓

[2. 입력 데이터 준비]

Background 이미지 목록 가져오기
        ↓
classes.txt에서 Class 목록 읽기
        ↓
Class별 Object PNG 목록 가져오기
        ↓
기존 실험의 이미지·Label 출력 폴더 초기화
        ↓
Class별 생성 수 균형 맞추기
drill → 40장
fire_extinguisher → 40장
wet_floor_sign → 40장
        ↓
Class 생성 순서를 Seed로 섞기
        ↓

[3. 합성 이미지 1장 생성 — 120번 반복]

이번 이미지의 Class와 Seed 결정
        ↓
Background와 Object PNG 무작위 선택
        ↓
Scale · Rotation · Position 무작위 결정
        ↓
객체 PNG의 투명 여백 제거
        ↓
요청한 Scale로 객체 크기 계산
        ↓
객체의 가로·세로 크기 검사
        ↓
배경보다 커지지 않도록 안전한 Scale 적용
        ↓
객체 크기 변경 및 Rotation 적용
        ↓
회전 후 투명 여백 제거
        ↓
최종 객체 크기 검사
        ↓
Position X / Y 계산
        ↓
Background에 Object 합성
        ↓
최종 Pixel BBox 계산
        ↓
YOLO 정규화 Label 계산
        ↓
합성 이미지 JPG 저장
YOLO Label TXT 저장
        ↓
생성 조건을 Metadata 목록에 기록
        ↓
다음 이미지 생성
        ↓

[4. 전체 결과 저장]

120장 반복 완료
        ↓
합성 이미지 JPG 120장
YOLO Label TXT 120개
        ↓
Metadata 목록을 CSV 파일로 저장
(120개 데이터 행 + 헤더)
        ↓
생성 개수와 저장 경로 출력
```

### 개발자가 알아야 할 핵심 수정 사항

이 코드는 **실행 옵션으로 생성 조건을 변경**할 수 있습니다. 입력·출력 폴더의 구조가 달라질 때만 공통 경로 관리 코드를 수정합니다.

|변경할 내용|수정 위치|핵심 설명|
|---|---|---|
|원본 데이터 경로|`day03_common.py` → `DATA_ROOT`|Source 폴더 위치 변경|
|사용할 데이터|`--asset-set`|`main` 또는 `challenge` 선택|
|이미지 생성 수|`--count`|예: `--count 200`|
|객체 크기 범위|`--scale-min`, `--scale-max`|무작위 크기 범위 변경|
|객체 회전 범위|`--rotation`|예: `30` → ±30°|
|무작위 생성 재현|`--seed`|동일한 입력 데이터·조건과 Seed 사용|
|실험 구분|`--tag`|실험별 결과를 별도 폴더에 저장|
|결과 저장 위치|`day03_common.py` → `experiment_paths()`|이미지·Label·Metadata 저장 경로 변경|
|Class 추가·변경|`classes.txt`, `objects/`|Class 이름·순서와 객체 폴더 확인|

### 코드 수정 시 주의할 사항

**① Scale은 자동으로 조절됩니다.**

수정된 코드는 객체가 배경보다 커지지 않도록 Scale을 자동으로 줄입니다.

Metadata에서 `scale`은 요청한 값, `effective_scale`은 실제 적용된 값입니다. 이미지가 지나치게 작거나 부자연스러운지는 별도로 확인해야 합니다.

**② 같은 실험 이름으로 재실행하면 결과가 덮어써집니다.**

```
--tag baseline     # 기존 실험
--tag scale_test   # 새로운 실험
```

같은 `tag`로 실행하면 기존 JPG·TXT가 초기화되고 Metadata CSV도 덮어써지므로, 이전 결과를 보존하려면 다른 `tag`를 사용합니다.

**③ Class 순서를 변경하면 Class ID도 달라집니다.**

`classes.txt`의 순서가 YOLO Label의 Class ID를 결정합니다. 따라서 순서를 변경했다면 기존 Label과도 일치하는지 확인해야 합니다.

### 핵심 정리

개발자는 주로 **입력 경로, 생성 수, Scale·Rotation 범위, 실험 이름, 출력 경로**를 수정합니다.

생성된 이미지와 YOLO Label이 정상적으로 저장되었는지 확인하고, 실제 학습에 적합한지는 이후 QA에서 검수합니다.

---

# 21. 먼저 6장 Smoke Test를 실행합니다

Smoke Test로 본격적으로 데이터를 많이 만들기 전에, 프로그램이 정상적으로 작동하는지 소량의 데이터로 미리 확인하는 테스트를 진행합니다.

먼저 3개 Class(전동드릴, 소화기, 미끄럼주의 표지판)가 각각 2장씩 포함되도록 총 6장의 합성 이미지를 생성합니다.

```bash
python scripts/03_generate_cutpaste_dataset.py \
  --asset-set main \
  --tag smoke \
  --count 6 \
  --seed 42 \
  --scale-min 0.08 \
  --scale-max 0.22 \
  --rotation 15
```

![[Pasted image 20261013111635.png]]

이 명령어는 배경 이미지에 객체를 다양한 위치·크기·각도로 붙여 합성 이미지 6장과 해당 객체의 YOLO BBox 정답 파일 6개를 자동 생성합니다.

생성된 파일을 확인합니다.

```bash
ls dataset/main/smoke/images
```

```bash
ls dataset/main/smoke/labels
```

이미지(JPG)와 정답 파일(TXT)은 다음처럼 파일명이 서로 일치해야 합니다.

```text
synthetic_00001.jpg
synthetic_00001.txt
```

왜 `smoke` 폴더에 저장하나요?

`smoke`는 프로그램이 정상적으로 작동하는지 시험하기 위한 폴더입니다.

이후 120장을 생성할 때는 `baseline`이라는 다른 폴더에 저장하므로, 앞에서 만든 테스트 이미지 6장과 본 실습 이미지 120장이 서로 섞이지 않습니다.

---

# 22. 본 실습 Dataset 120장을 생성합니다

앞에서 6장의 이미지와 YOLO Label이 정상적으로 생성되는 것을 확인했습니다.

이제 같은 Cut-Paste 방식으로 3개 Class가 각각 40장씩 포함된 총 120장의 합성 Detection 데이터셋을 생성합니다.

```bash
python scripts/03_generate_cutpaste_dataset.py \
  --asset-set main \
  --tag baseline \
  --count 120 \
  --seed 42 \
  --scale-min 0.08 \
  --scale-max 0.22 \
  --rotation 15
```

정상적으로 실행되면 다음 파일이 생성됩니다.

```text
dataset/main/baseline/
├─ images/
│  ├─ synthetic_00001.jpg
│  ├─ ...
│  └─ synthetic_00120.jpg
└─ labels/
   ├─ synthetic_00001.txt
   ├─ ...
   └─ synthetic_00120.txt
```

또한 어떤 배경과 객체를 사용하고, 어떤 크기·회전·위치로 합성했는지 기록한 Metadata 파일도 생성됩니다.

```text
reports/main/baseline_metadata.csv
```

이렇게 생성한 120장은 바로 AI 학습에 사용하는 것이 아니라, 다음 실습에서 BBox와 Label이 정확한지 검수할 데이터입니다.

---

# 23. 실제 생성된 YOLO Label과 Metadata를 확인합니다

120장의 합성 이미지가 생성되었습니다. 이제 첫 번째 이미지인 `synthetic_00001.jpg`를 기준으로 **YOLO Label과 Metadata에 무엇이 기록되었는지** 확인하겠습니다.

### ① YOLO Label을 확인합니다

```bash
cat dataset/main/baseline/labels/synthetic_00001.txt
```

**실제 실행 결과**

```
0 0.151389 0.626390 0.189815 0.201631
```

|값|의미|실제 결과|
|---|---|---|
|`0`|Class ID|drill|
|`0.151389`|중심 X|이미지 가로 약 15.1% 위치|
|`0.626390`|중심 Y|이미지 세로 약 62.6% 위치|
|`0.189815`|BBox 너비|이미지 너비의 약 19.0%|
|`0.201631`|BBox 높이|이미지 높이의 약 20.2%|

즉, 첫 번째 합성 이미지에는 **드릴 한 개가 왼쪽 아래쪽에 배치되었으며, 그 위치와 크기가 YOLO 형식으로 저장**되었습니다.

### ② Metadata를 확인합니다

```bash
head reports/main/baseline_metadata.csv
```

출력된 CSV에서 첫 번째 이미지의 주요 정보만 살펴보겠습니다.

|항목|실제 기록|의미|
|---|---|---|
|`file`|`synthetic_00001.jpg`|첫 번째 합성 이미지|
|`background`|`bg_010.jpg`|사용한 배경|
|`object`|`drill_002.png`|사용한 드릴 PNG|
|`class_id`|`0`|drill|
|`scale`|`0.1947`|요청한 객체 크기 비율|
|`effective_scale`|`0.19444`|실제 적용된 크기 비율|
|`rotation`|`-12.592`|드릴을 약 12.6도 시계 방향으로 회전|
|`x1, y1, x2, y2`|`61, 709, 266, 981`|실제 Pixel BBox|
|`global_seed`|`42`|전체 실험의 Seed|
|`approved`|`pending`|아직 검수 전|

여기서 `scale`과 `effective_scale`이 조금 다른 것은 실제 이미지 크기를 정수 픽셀 단위로 계산하고 필요하면 안전하게 축소하기 때문입니다.

특히 Metadata에 기록된 YOLO BBox 값은 앞에서 확인한 TXT 값과 동일합니다.

### ③ 두 파일의 역할을 구분합니다

```
synthetic_00001.jpg
       │
       ├── synthetic_00001.txt
       │   → 어떤 객체가 어디에 있는가?
       │   → YOLO 모델 학습용 정답
       │
       └── baseline_metadata.csv의 해당 행
           → 어떤 배경과 객체를 사용했는가?
           → 크기·회전·위치를 어떻게 설정했는가?
           → 생성 과정과 검수 상태 기록
```

**핵심 정리:** YOLO TXT는 **객체검출 모델이 사용하는 정답 좌표**이고, Metadata CSV는 **그 이미지가 어떻게 만들어졌는지 확인하는 기록**입니다.

현재 `approved=pending`이므로 120장의 생성은 완료되었지만, 실제 학습에 적합한지는 이후 QA 실습에서 확인해야 합니다.

---

# PART 5. BBox를 다시 그리고 Automatic QA와 Human QA를 수행합니다

---

# 24. YOLO Label을 이미지 위에 그리는 코드를 작성합니다

앞선 실습에서는 **120장의 합성 이미지와 YOLO Label을 자동 생성**했습니다.

그런데 TXT 파일에 좌표가 정상적으로 저장되었다고 해서 BBox가 객체를 정확하게 감싸고 있다고 단정할 수는 없습니다.

이번에는 저장된 **YOLO Label을 다시 읽어 합성 이미지 위에 BBox를 그리는 코드**를 작성하겠습니다.

### 이 코드는 무엇을 하나요?

```
합성 이미지 JPG 불러오기
        +
YOLO Label TXT 읽기
        ↓
YOLO 정규화 좌표를 Pixel 좌표로 변환
        ↓
객체 주변에 BBox와 Class 이름 표시
        ↓
확인용 이미지 저장
```

예를 들어 `synthetic_00001.jpg`와 `synthetic_00001.txt`를 읽으면 드릴 주변에 사각형과 `drill`이라는 이름을 표시합니다.

이번 실습에서는 생성된 120장 중 **20장을 시각화**하여 `results/main/baseline/visualized/`에 저장합니다.

### 무엇을 확인하나요?

- BBox가 실제 객체를 감싸는가?
- BBox가 지나치게 크거나 작지 않은가?
- Class 이름이 올바르게 표시되는가?
    

**핵심:** 이 코드는 새로운 BBox를 만드는 것이 아니라, **이미 생성된 YOLO Label이 실제 이미지에서 어떻게 표시되는지 확인하는 코드**입니다.

이후 Automatic QA에서는 Label의 형식과 좌표 범위를 검사하고, Human QA에서는 합성 장면이 현실적인지도 직접 판단하겠습니다.

**파일:** `scripts/04_visualize_yolo_bbox.py`

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import cv2


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    experiment_paths,
    read_classes,
    reset_directory,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--tag",
        default="baseline",
    )

    parser.add_argument(
        "--count",
        type=int,
        default=20,
    )

    return parser.parse_args()


def yolo_to_pixel(
    x_center,
    y_center,
    bbox_width,
    bbox_height,
    image_width,
    image_height,
):
    x_center *= image_width
    y_center *= image_height
    bbox_width *= image_width
    bbox_height *= image_height

    x1 = int(
        round(
            x_center
            - bbox_width / 2
        )
    )

    y1 = int(
        round(
            y_center
            - bbox_height / 2
        )
    )

    x2 = int(
        round(
            x_center
            + bbox_width / 2
        )
    )

    y2 = int(
        round(
            y_center
            + bbox_height / 2
        )
    )

    return (
        x1,
        y1,
        x2,
        y2,
    )


def main() -> None:
    args = parse_args()

    paths = experiment_paths(
        args.asset_set,
        args.tag,
    )

    classes = read_classes()

    reset_directory(
        paths["visualized"]
    )

    image_files = sorted(
        paths["images"].glob(
            "*.jpg"
        )
    )[: args.count]

    if not image_files:
        raise RuntimeError(
            "시각화할 이미지가 없습니다."
        )

    saved_count = 0

    for image_path in image_files:
        label_path = (
            paths["labels"]
            / (
                image_path.stem
                + ".txt"
            )
        )

        if not label_path.exists():
            continue

        image = cv2.imread(
            str(image_path)
        )

        if image is None:
            continue

        height, width = (
            image.shape[:2]
        )

        lines = (
            label_path
            .read_text(
                encoding="utf-8"
            )
            .splitlines()
        )

        for line in lines:
            values = line.split()

            if len(values) != 5:
                continue

            class_id = int(
                values[0]
            )

            x_center = float(
                values[1]
            )

            y_center = float(
                values[2]
            )

            bbox_width = float(
                values[3]
            )

            bbox_height = float(
                values[4]
            )

            (
                x1,
                y1,
                x2,
                y2,
            ) = yolo_to_pixel(
                x_center,
                y_center,
                bbox_width,
                bbox_height,
                width,
                height,
            )

            cv2.rectangle(
                image,
                (x1, y1),
                (x2, y2),
                (0, 0, 255),
                2,
            )

            class_name = (
                classes[class_id]
                if (
                    0
                    <= class_id
                    < len(classes)
                )
                else (
                    f"class_"
                    f"{class_id}"
                )
            )

            cv2.putText(
                image,
                class_name,
                (
                    x1,
                    max(
                        20,
                        y1 - 5,
                    ),
                ),
                (
                    cv2
                    .FONT_HERSHEY_SIMPLEX
                ),
                0.65,
                (0, 0, 255),
                2,
                cv2.LINE_AA,
            )

        output_path = (
            paths["visualized"]
            / image_path.name
        )

        cv2.imwrite(
            str(output_path),
            image,
        )

        saved_count += 1

    print(
        "Visualized:",
        saved_count,
    )

    print(
        "Output:",
        paths["visualized"],
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[1. 실행 조건 설정]

사용할 Source → main
실험 이름 → baseline
시각화할 이미지 수 → 20장
        ↓

[2. 입력 데이터 준비]

dataset/main/baseline/images/
→ 합성 이미지 JPG 불러오기

dataset/main/baseline/labels/
→ 각 이미지와 같은 이름의 YOLO TXT 확인

classes.txt
→ Class ID에 해당하는 이름 불러오기
        ↓

[3. BBox 시각화 — 이미지마다 반복]

합성 이미지 1장 선택
        ↓
같은 이름의 YOLO TXT 읽기
        ↓
Class ID와 YOLO BBox 좌표 추출
        ↓
YOLO 정규화 좌표를 Pixel 좌표로 변환
        ↓
이미지 위에 빨간색 BBox 그리기
        ↓
BBox 위에 Class 이름 표시
        ↓
시각화된 이미지 JPG 저장
        ↓
다음 이미지 처리
        ↓

[4. 결과 확인]

최대 20장 시각화
        ↓
results/main/baseline/visualized/
→ BBox가 표시된 이미지 저장
        ↓
실제로 저장된 이미지 수와 경로 출력
```

### 코드에서 알아야 할 핵심 기능

|코드|역할|
|---|---|
|`experiment_paths()`|이미지·Label·결과 폴더 경로 확인|
|`--count`|시각화할 최대 이미지 수 지정|
|`yolo_to_pixel()`|YOLO 정규화 좌표를 이미지 픽셀 좌표로 변환|
|`cv2.rectangle()`|객체 주변에 빨간색 BBox 그리기|
|`cv2.putText()`|BBox에 Class 이름 표시|
|`cv2.imwrite()`|BBox가 표시된 이미지 저장|

실제 코드의 주의사항: 파일명 순서대로 처음 20장을 선택하며, 이미지나 Label이 없으면 해당 이미지는 건너뜁니다. 또한 실행할 때 기존 `visualized` 폴더의 결과가 초기화됩니다.

핵심 정리: 이 코드는 새로운 BBox를 계산하는 코드가 아니라, 기존 YOLO Label을 이미지 위에 다시 그려서 객체의 위치·크기·Class가 올바른지 눈으로 확인하는 코드입니다. 실제 Label 형식 오류는 다음 Automatic QA 단계에서 검사합니다.

---
본 실습 결과 20장을 확인합니다.

```bash
python scripts/04_visualize_yolo_bbox.py \
  --asset-set main \
  --tag baseline \
  --count 20
```

결과:

```text
results/main/baseline/visualized/
```

---

# 25. BBox 재시각화 결과를 확인합니다

24번 코드로 생성한 이미지를 열어 **YOLO Label이 객체를 올바르게 표시하는지** 확인하겠습니다.

**결과 폴더:** `results/main/baseline/visualized/`

![[Pasted image 20261013115043.png]]
### ① 실제 생성된 이미지를 확인합니다

업로드한 결과에는 바다 배경 위에 노란색 드릴이 합성되어 있으며, 빨간색 BBox와 `drill`이라는 Class 이름이 표시되어 있습니다.

|확인 항목|실제 결과|
|---|---|
|Class 이름|`drill` — 정상|
|BBox 위치|드릴을 감싸고 있음|
|BBox 크기|드릴 전체를 대체로 정확히 감싸고 있음|
|이미지 경계|BBox가 이미지 밖으로 벗어나지 않음|
|합성 장면의 현실성|부족 — 드릴이 바다 위에 떠 있음|

**BBox 시각화는 정상적으로 보이지만, 합성 이미지 자체는 현실적인 작업 환경과 맞지 않습니다.**

### ② 나머지 이미지도 직접 확인합니다

`visualized` 폴더에 저장된 20장을 열어 다음 사항을 확인합니다.

- BBox가 객체 전체를 감싸는가?
- Class 이름이 올바른가?
- 크기가 작거나 회전된 객체도 정확하게 표시되는가?
- BBox가 이미지 밖으로 벗어나지 않는가?
- 객체의 크기·위치·배경이 현실적인가?
    
### ③ 결과를 어떻게 판단해야 하나요?

이번 드릴 이미지는 **YOLO BBox 시각화에는 성공했지만, 실제 학습데이터로 사용하기에는 부적절한 사례**입니다.

```
BBox와 Class 표시
→ 정상적으로 보임

합성 장면의 현실성
→ 부족

최종 학습데이터 승인
→ 추가 검수 필요
```

**핵심 정리:** YOLO Label이 자동 생성되었다고 해서 학습에 적합한 데이터가 되는 것은 아닙니다.

이번 단계에서는 BBox가 올바르게 표시되는지 눈으로 확인했습니다. 다음 26번에서는 전체 120장의 Label 형식과 좌표를 Automatic QA로 검사하고, 이후 Human QA에서는 합성 장면의 현실성까지 검수하겠습니다.

---

# 26. 모든 YOLO Label을 자동 검사하는 코드를 작성합니다

25번에서는 20장의 이미지를 직접 확인했습니다.

하지만 120장의 Label을 하나씩 열어서 검사하는 것은 번거롭습니다.

이번에는 **전체 120장의 YOLO Label에 형식이나 좌표 오류가 없는지 자동으로 검사하는 Python 코드**를 작성하겠습니다.

### 이 코드는 무엇을 하나요?

```text
합성 이미지 JPG 120장
        +
YOLO Label TXT 120개
        ↓
이미지와 Label 파일 연결 확인
        ↓
Class ID와 Label 형식 검사
        ↓
BBox 좌표와 크기 검사
        ↓
이미지 경계 초과 여부 확인
        ↓
PASS / FAIL 결과 저장
```

예를 들어 Class ID가 존재하지 않거나 BBox가 이미지 밖으로 벗어나면 오류로 판정합니다.

YOLO 좌표는 소수점 6자리로 저장하므로, 작은 반올림 오차를 허용하기 위해 `1e-6`의 허용오차를 적용합니다.

**핵심:** 이 코드는 **Label의 형식과 좌표가 정상인지 검사하는 Automatic QA**입니다.

단, 드릴이 하늘에 떠 있는 것처럼 합성 장면이 부자연스러운지는 판단하지 못합니다. 이러한 문제는 이후 Human QA에서 직접 검수합니다.

**파일: `scripts/05_validate_yolo_labels.py`**

```python
from __future__ import annotations

import argparse
import math
import sys
from pathlib import Path

import cv2
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    experiment_paths,
    read_classes,
)


EPSILON = 1e-6


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--tag",
        default="baseline",
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    paths = experiment_paths(
        args.asset_set,
        args.tag,
    )

    classes = read_classes()

    image_files = sorted(
        paths["images"].glob(
            "*.jpg"
        )
    )

    label_files = sorted(
        paths["labels"].glob(
            "*.txt"
        )
    )

    image_stems = {
        path.stem
        for path in image_files
    }

    label_stems = {
        path.stem
        for path in label_files
    }

    all_stems = sorted(
        image_stems
        | label_stems
    )

    rows = []

    for stem in all_stems:
        image_path = (
            paths["images"]
            / f"{stem}.jpg"
        )

        label_path = (
            paths["labels"]
            / f"{stem}.txt"
        )

        reasons = []

        if not image_path.exists():
            reasons.append(
                "missing_image"
            )

        if not label_path.exists():
            reasons.append(
                "missing_label"
            )

        if image_path.exists():
            image = cv2.imread(
                str(image_path)
            )

            if image is None:
                reasons.append(
                    "unreadable_image"
                )

        if label_path.exists():
            lines = (
                label_path
                .read_text(
                    encoding="utf-8"
                )
                .splitlines()
            )

            if not lines:
                reasons.append(
                    "empty_label"
                )

            for line_index, line in enumerate(
                lines,
                start=1,
            ):
                values = line.split()

                if len(values) != 5:
                    reasons.append(
                        f"line_{line_index}:"
                        "field_count"
                    )

                    continue

                try:
                    class_id = int(
                        values[0]
                    )

                    x_center = float(
                        values[1]
                    )

                    y_center = float(
                        values[2]
                    )

                    width = float(
                        values[3]
                    )

                    height = float(
                        values[4]
                    )

                except ValueError:
                    reasons.append(
                        f"line_{line_index}:"
                        "parse_error"
                    )

                    continue

                numeric_values = [
                    x_center,
                    y_center,
                    width,
                    height,
                ]

                if not all(
                    math.isfinite(value)
                    for value in numeric_values
                ):
                    reasons.append(
                        f"line_{line_index}:"
                        "non_finite"
                    )
                    continue

                if not (
                    0
                    <= class_id
                    < len(classes)
                ):
                    reasons.append(
                        f"line_{line_index}:"
                        "class_id"
                    )

                for name, value in [
                    (
                        "x_center",
                        x_center,
                    ),
                    (
                        "y_center",
                        y_center,
                    ),
                    (
                        "width",
                        width,
                    ),
                    (
                        "height",
                        height,
                    ),
                ]:
                    if not (
                        -EPSILON
                        <= value
                        <= 1.0 + EPSILON
                    ):
                        reasons.append(
                            f"line_"
                            f"{line_index}:"
                            f"{name}_range"
                        )

                if width <= 0:
                    reasons.append(
                        f"line_{line_index}:"
                        "width_zero"
                    )

                if height <= 0:
                    reasons.append(
                        f"line_{line_index}:"
                        "height_zero"
                    )

                x1 = (
                    x_center
                    - width / 2
                )

                y1 = (
                    y_center
                    - height / 2
                )

                x2 = (
                    x_center
                    + width / 2
                )

                y2 = (
                    y_center
                    + height / 2
                )

                if (
                    x1 < -EPSILON
                    or y1 < -EPSILON
                    or x2 > 1.0 + EPSILON
                    or y2 > 1.0 + EPSILON
                ):
                    reasons.append(
                        f"line_{line_index}:"
                        "bbox_outside"
                    )

        rows.append(
            {
                "file": (
                    stem + ".jpg"
                ),
                "auto_pass": (
                    len(reasons) == 0
                ),
                "reason": "|".join(
                    reasons
                ),
            }
        )

    if not rows:
        raise RuntimeError(
            "검사할 Image/Label이 없습니다."
        )

    dataframe = pd.DataFrame(
        rows
    )

    report_path = (
        paths["report"]
        / (
            f"{args.tag}"
            f"_validation.csv"
        )
    )

    dataframe.to_csv(
        report_path,
        index=False,
        encoding="utf-8-sig",
    )

    passed = int(
        dataframe[
            "auto_pass"
        ].sum()
    )

    failed = (
        len(dataframe)
        - passed
    )

    print("=" * 60)
    print(
        "YOLO Validation Completed"
    )
    print("=" * 60)

    print(
        "Total:",
        len(dataframe),
    )

    print(
        "PASS :",
        passed,
    )

    print(
        "FAIL :",
        failed,
    )

    print(
        "Report:",
        report_path,
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
Image Stem 목록을 만든다
        ↓
Label Stem 목록을 만든다
        ↓
Image와 Label이 서로 짝이 맞는지 확인한다
        ↓
이미지가 읽히는지 확인한다
        ↓
Label 한 줄이 5개 값인지 확인한다
        ↓
Class ID 범위를 확인한다
        ↓
NaN / Infinity 같은 비정상 숫자가 없는지 확인한다
        ↓
작은 반올림 허용오차를 적용해 좌표 범위를 확인한다
        ↓
Width / Height가 0보다 큰지 확인한다
        ↓
BBox가 이미지 밖으로 나가지 않는지 확인한다
        ↓
문제가 없으면 auto_pass
        ↓
CSV로 저장한다
```

본 실습 Dataset을 검사합니다.

```bash
python scripts/05_validate_yolo_labels.py \
  --asset-set main \
  --tag baseline
```

정상적인 실행 결과는 다음 형태입니다.

```text
Total: 120
PASS : 120
FAIL : 0
```

`PASS 120`은 **Label 형식과 좌표가 정상이라는 뜻**입니다.

합성 장면이 현실적이라는 뜻은 아닙니다.

---

# 27. Automatic QA와 Human QA의 역할을 구분합니다

Automatic QA가 잘하는 일:

```text
파일이 있는가?
좌표를 읽을 수 있는가?
Class ID가 정상인가?
0~1 범위를 지키는가?
BBox가 이미지 안에 있는가?
```

사람이 확인해야 하는 일:

```text
객체 크기가 현실적인가?
위치가 자연스러운가?
회전 범위가 적절한가?
객체가 공중에 떠 보이지 않는가?
배경과 객체의 조명이 지나치게 다른가?
BBox가 실제 객체 경계와 시각적으로 맞는가?
```

따라서:

```text
Automatic QA
+
Human QA
```

가 함께 필요합니다.

---

# 28. Human QA 기록용 CSV를 만드는 코드를 작성합니다

앞에서 만든 Metadata(이미지 생성 정보)와 Automatic QA(자동 검사 결과)를 하나로 합쳐, 
사람이 직접 검수할 수 있는 CSV 파일을 만듭니다.

생성된 CSV에는 이미지마다 사람이 확인한 결과를 `human_decision`에 기록하고, 문제가 있다면 그 이유를 `human_note`에 작성합니다.

이미 작성한 검수 결과가 실수로 삭제되지 않도록 같은 CSV 파일이 이미 존재하면 새로 만들지 
않고 실행을 중단합니다.

**파일: `scripts/06_make_qa_report.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day03_common import (
    experiment_paths,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--tag",
        default="baseline",
    )

    parser.add_argument(
        "--overwrite",
        action="store_true",
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    paths = experiment_paths(
        args.asset_set,
        args.tag,
    )

    metadata_path = (
        paths["report"]
        / (
            f"{args.tag}"
            f"_metadata.csv"
        )
    )

    validation_path = (
        paths["report"]
        / (
            f"{args.tag}"
            f"_validation.csv"
        )
    )

    if not metadata_path.exists():
        raise FileNotFoundError(
            metadata_path
        )

    if not validation_path.exists():
        raise FileNotFoundError(
            validation_path
        )

    output_path = (
        paths["report"]
        / (
            f"{args.tag}"
            f"_human_qa.csv"
        )
    )

    if (
        output_path.exists()
        and not args.overwrite
    ):
        raise RuntimeError(
            "기존 Human QA 파일이 있습니다.\n"
            f"{output_path}\n"
            "이미 작성한 검수 결과를 보호하기 위해 "
            "덮어쓰지 않습니다.\n"
            "정말 새로 만들 때만 --overwrite를 사용하세요."
        )

    metadata = pd.read_csv(
        metadata_path
    )

    validation = pd.read_csv(
        validation_path
    )

    merged = metadata.merge(
        validation,
        on="file",
        how="left",
        validate="one_to_one",
    )

    if merged[
        "auto_pass"
    ].isna().any():
        raise RuntimeError(
            "Metadata와 Validation의 file 대응이 "
            "완전하지 않습니다."
        )

    merged[
        "object_size_realistic"
    ] = ""

    merged[
        "position_realistic"
    ] = ""

    merged[
        "rotation_realistic"
    ] = ""

    merged[
        "edge_natural"
    ] = ""

    merged[
        "bbox_visual_ok"
    ] = ""

    merged[
        "human_decision"
    ] = ""

    merged[
        "human_note"
    ] = ""

    merged.to_csv(
        output_path,
        index=False,
        encoding="utf-8-sig",
    )

    print(
        "Saved:",
        output_path,
    )

    print(
        "Rows:",
        len(merged),
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[1. 입력 — 필요한 CSV 파일 확인]

실행 옵션 확인
→ asset-set: main 또는 challenge
→ tag: baseline 등 실험 이름
→ overwrite: 기존 파일 덮어쓰기 허용 여부
        ↓
Metadata CSV 경로 확인
→ 이미지 생성 정보

Validation CSV 경로 확인
→ Automatic QA 검사 결과
        ↓
두 파일이 존재하는지 확인
→ 없으면 오류를 알리고 중단


[2. 기존 파일 보호]

Human QA CSV 저장 경로 확인
        ↓
이미 Human QA 파일이 존재하는가?
        ├─ YES → 기본적으로 실행 중단
        │         기존 검수 기록 보호
        │
        │         단, --overwrite 옵션이 있으면 계속 진행
        │
        └─ NO → 다음 단계 진행


[3. 처리 로직 — CSV 데이터 연결]

Metadata CSV 읽기
        +
Validation CSV 읽기
        ↓
file 컬럼을 기준으로 데이터 연결
        ↓
각 이미지의 생성 정보와
자동 검사 결과를 한 행으로 합치기


[4. 데이터 검증]

Metadata와 Validation이
1:1로 연결되는지 확인
        ↓
auto_pass 값이 누락되었는지 확인
        ↓
누락된 값이 있으면
→ 오류를 알리고 중단

정상적으로 연결되면
→ 다음 단계 진행


[5. Human QA 검수 항목 추가]

사람이 직접 작성할 빈 컬럼 생성
        ↓
object_size_realistic
→ 객체 크기가 현실적인가?

position_realistic
→ 객체 위치가 자연스러운가?

rotation_realistic
→ 객체 회전이 자연스러운가?

edge_natural
→ 합성 경계가 자연스러운가?

bbox_visual_ok
→ BBox가 객체를 정확히 감싸는가?

human_decision
→ 최종 검수 판정

human_note
→ 검수 의견 및 문제점 기록


[6. 출력 — Human QA CSV 저장]

Metadata + Automatic QA + Human QA 항목
        ↓
하나의 CSV 파일로 저장
        ↓
reports/main/baseline_human_qa.csv
        ↓
저장된 파일 경로 출력
        ↓
CSV 데이터 행 수 출력
        ↓
실행 완료
```

실행합니다.

```bash
python scripts/06_make_qa_report.py \
  --asset-set main \
  --tag baseline
```

결과:

```text
reports/main/baseline_human_qa.csv
```

Human QA 내용을 작성한 다음 같은 명령을 다시 실행하면 기존 기록을 보호하기 위해 중단됩니다.

---

# 29. Human QA를 직접 수행합니다

앞에서 생성한 `baseline_human_qa.csv`에 사람이 직접 이미지를 확인한 결과를 기록하겠습니다.

실제 CSV를 확인한 결과, **120장 모두 Automatic QA를 통과(`True`)**했지만, Human QA는 아직 진행되지 않았습니다.

### ① 이미지 20장을 직접 확인합니다

다음 폴더의 이미지를 하나씩 열어봅니다.

```
results/main/baseline/visualized/
```

### ② CSV에 검수 결과를 기록합니다

파일: `reports/main/baseline_human_qa.csv`

|CSV 항목|확인할 내용|입력값|
|---|---|---|
|`object_size_realistic`|객체 크기가 적절한가?|yes / no|
|`position_realistic`|객체 위치가 자연스러운가?|yes / no|
|`rotation_realistic`|회전 방향이 적절한가?|yes / no|
|`edge_natural`|합성 경계가 자연스러운가?|yes / no|
|`bbox_visual_ok`|BBox가 정확한가?|yes / no|
|`human_decision`|최종 판정|approved / needs_review / rejected|
|`human_note`|판정한 이유|직접 작성|

### ③ 검수 결과를 작성합니다

예를 들어 드릴이 공중에 떠 있는 이미지라면 다음처럼 기록합니다.

```
human_decision: rejected

human_note:
BBox는 정확하지만 드릴이 공중에 떠 있어
실제 학습데이터로 사용하기 부적절함.
```

`approved`는 승인, `needs_review`는 추가 검토, `rejected`는 사용 부적합을 의미합니다.

검수한 20장만 결과를 입력하고, 확인하지 않은 나머지 100장은 빈칸으로 남겨둡니다.

핵심 정리: Automatic QA를 통과했다고 해서 좋은 학습데이터는 아닙니다. Human QA에서는 이미지의 현실성을 직접 확인하고, 승인하거나 거절한 이유를 기록하는 것이 중요합니다.

---

# PART 6. 본 실습 결과를 연결하고 데이터 편향과 Metadata를 해석합니다

---

# 30. Class별 생성 개수를 확인합니다

앞에서 생성한 120장의 합성 이미지가 세 가지 Class에 균등하게 분배되었는지 확인하겠습니다.

이번 코드는 `baseline_metadata.csv`에서 각 Class의 이미지 수를 자동으로 계산합니다.

Class별 이미지 개수를 확인합니다

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv(
    "reports/main/baseline_metadata.csv"
)

print(
    df["class_name"].value_counts()
)
PY
```

의사코드로 읽으면 다음과 같습니다.

```text
Metadata CSV 읽기
        ↓
class_name 확인
        ↓
Class별 이미지 개수 계산
        ↓
결과 출력
```

실행 결과

```text
class_name
drill                40
wet_floor_sign       40
fire_extinguisher    40
Name: count, dtype: int64
```

### 실행 결과 해석

실제 Metadata를 확인한 결과, 총 120장의 합성 이미지가 세 가지 Class에 각각 40장씩 균등하게 생성되었습니다.

- `drill`: 전동드릴 40장
- `wet_floor_sign`: 미끄럼 주의 표지판 40장
- `fire_extinguisher`: 소화기 40장
    
`Name: count`는 Class별 개수를 계산한 결과라는 뜻이며, `dtype: int64`는 개수가 
정수형 데이터로 표시된다는 의미입니다.

핵심 정리: 세 Class의 이미지 수는 균형을 이루고 있습니다. 하지만 Class 개수가 같다고 해서 모든 데이터 편향이 사라지는 것은 아닙니다. 객체의 위치(Position), 크기(Scale), 합성 장면의 현실성도 함께 확인해야 합니다.

---

# 31. Position과 Scale의 편향을 이해합니다

앞에서 Class별 이미지 개수가 균등한지 확인했습니다.

하지만 **객체의 위치(Position)와 크기(Scale)**도 다양해야 합니다.

| 구분       | 문제가 되는 경우     | 이유                        |
| -------- | ------------- | ------------------------- |
| Position | 객체가 항상 중앙에 있음 | 모델이 중앙에 있는 객체에만 익숙해질 수 있음 |
| Scale    | 객체가 항상 비슷한 크기 | 작거나 큰 객체를 잘 검출하지 못할 수 있음  |
| 현실성      | 드릴이 하늘에 떠 있음  | 실제 작업 환경과 다른 장면을 학습할 수 있음 |

실제 Metadata를 보면 객체의 위치와 크기를 다양하게 설정한 것을 확인할 수 있습니다.

예를 들어:

|이미지|Scale|X 위치 비율|Y 위치 비율|
|---|---|---|---|
|`synthetic_00001.jpg`|0.19470|0.069513|0.658168|
|`synthetic_00002.jpg`|0.17136|0.640818|0.162366|
|`synthetic_00003.jpg`|0.20875|0.057527|0.561299|

이 값들은 객체를 서로 다른 크기와 위치로 배치하도록 설정했다는 의미입니다. 여기서 X/Y 위치 비율은 객체가 이동할 수 있는 범위 안에서의 상대적 위치입니다.

그러나 무작위로 배치했다고 해서 항상 현실적인 이미지는 아닙니다.

```
다양한 Position + Scale
        ↓
여러 상황의 학습데이터 생성
        ↓
실제 환경과 자연스럽게 어울리는가?
        ↓
Human QA로 최종 확인
```

핵심: 데이터의 다양성과 현실성은 다릅니다. 위치와 크기가 다양하더라도 부자연스러운 이미지는 검수해야 합니다.

---

# 2. Metadata에서 이미지 생성 과정을 다시 확인합니다

Human QA에서 문제가 있는 이미지를 발견하면, Metadata를 통해 어떤 배경과 객체를 사용했고 어떤 조건으로 합성했는지 확인할 수 있습니다.

실제 생성된 `synthetic_00023.jpg`를 예로 살펴보겠습니다.

특정 이미지의 Metadata를 확인합니다

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv(
    "reports/main/baseline_metadata.csv"
)

print(
    df[
        df["file"]
        == "synthetic_00023.jpg"
    ].T
)
PY
```

의사코드로 읽으면 다음과 같습니다.

```text
Metadata CSV 읽기
        ↓
synthetic_00023.jpg 검색
        ↓
해당 이미지의 생성 기록만 선택
        ↓
결과를 세로로 출력
```

앞에서 조회한 `synthetic_00023.jpg`의 위치는 다음 명령어로 확인합니다.

```bash
realpath dataset/main/baseline/images/synthetic_00023.jpg
```

### 실제 실행 결과 해석

`synthetic_00023.jpg`의 Metadata를 조회한 결과입니다.

![[Pasted image 20261014111319.png|214]]

|확인 항목|실제 출력값|의미|
|---|---|---|
|`file`|synthetic_00023.jpg|확인한 합성 이미지|
|`background`|bg_015.jpg|사용한 배경 이미지|
|`object`|wet_floor_sign_003.png|사용한 객체 PNG|
|`class_id` / `class_name`|2 / wet_floor_sign|미끄럼 주의 표지판|
|`scale`|0.12645|요청한 크기 비율|
|`effective_scale`|0.12593|실제 적용된 크기 비율|
|`rotation_limit`|15.0|최대 회전 범위 ±15°|
|`rotation_unit`|-0.489808|회전각 계산에 사용한 비율|
|`rotation`|-7.347|실제 적용한 회전각|
|`x_ratio` / `y_ratio`|0.613726 / 0.967996|객체를 배치할 수 있는 범위에서의 상대적 위치|
|`x1, y1, x2, y2`|579, 1360, 716, 1575|실제 픽셀 단위 BBox|
|`yolo_x_center` / `yolo_y_center`|0.599537 / 0.905864|이미지 전체를 기준으로 정규화한 BBox 중심|
|`yolo_width` / `yolo_height`|0.126852 / 0.132716|정규화한 BBox 크기|
|`global_seed` / `sample_seed`|42 / 4200022|전체 실험 및 해당 이미지의 난수 기준값|
|`approved`|pending|아직 검수하지 않은 상태|

출력 맨 위의 `22`는 이미지 번호가 아니라 Pandas의 행 인덱스입니다. 0부터 시작하므로 23번째 데이터라는 뜻입니다.

### 실행 결과에서 무엇을 확인했나요?

`synthetic_00023.jpg`는 `bg_015.jpg` 배경에 미끄럼 주의 표지판을 합성한 이미지입니다. 객체의 크기와 회전각, 배치 위치, YOLO BBox 좌표까지 기록되어 있습니다.

또한 `approved`가 `pending`이므로 아직 최종 검수는 완료되지 않았습니다.

핵심 정리: 이렇게 특정 이미지에 사용된 원본과 생성 조건을 다시 확인하고 추적하는 것을 **Data Lineage(데이터 생성 이력 추적)**라고 합니다.

---

# 33. 본 실습의 전체 흐름을 한 번 연결합니다

오늘은 배경 사진에 객체를 합성하고, 객체의 위치를 표시하는 YOLO 정답 파일까지 자동으로 만드는 방법을 배웠습니다.

지금까지 진행한 실습을 순서대로 연결해 보겠습니다.

![[Pasted image 20261014111909.png]]

```text
[1. 원본 데이터 준비]

배경 사진 20장
+
투명한 객체 PNG 15장
(드릴 · 소화기 · 미끄럼 주의 표지판)
        ↓
이미지가 정상인지 검사하고 미리보기


[2. 이미지 한 장 합성]

배경에 드릴 붙이기
        ↓
객체 크기 · 회전 · 위치 조절
        ↓
객체를 감싸는 BBox 자동 계산
        ↓
YOLO 정답 파일(TXT) 생성


[3. 합성 이미지 120장 자동 생성]

앞에서 배운 합성 과정을 반복
        ↓
3개 Class × 40장씩 생성
        ↓
이미지 JPG + YOLO Label TXT 저장
        ↓
Metadata CSV에 생성 조건 기록


[4. 생성 결과 검사]

YOLO Label을 이미지 위에 다시 표시
        ↓
Automatic QA
→ 120장 모두 Label 형식·좌표 검사 통과
        ↓
Human QA
→ 사람이 직접 이미지의 현실성 확인
        ↓
승인 / 추가 검토 / 사용 부적합 판단


[5. 생성 기록 확인]

Class별 이미지 수 확인
        ↓
Position · Scale의 편향 이해
        ↓
Metadata로 이미지 생성 조건 추적
```

다음 문장을 자신의 실행 결과에 맞게 적어 둡니다.

```text
BBox 자동 생성이 가능한 이유:

본 실습에서 가장 부자연스러웠던 합성 조건:

Automatic QA로 찾을 수 있는 문제:

Human QA가 추가로 필요한 이유:
```

### 오늘 배운 내용을 확인합니다

다음 질문에 자신의 실습 결과를 바탕으로 답해 보세요.

|질문|생각해 볼 내용|
|---|---|
|BBox를 자동 생성할 수 있는 이유는?|객체를 붙인 위치와 크기를 프로그램이 알고 있기 때문|
|부자연스러운 합성 이미지는?|예: 드릴이 하늘이나 바다 위에 떠 있는 이미지|
|Automatic QA는 무엇을 검사했나요?|파일 누락, Class ID, YOLO Label 형식과 좌표|
|Human QA가 필요한 이유는?|자동 검사로는 합성 장면의 현실성을 판단하기 어렵기 때문|

전체 실습 기록은 Mini Challenge까지 마친 후 `day03_notes.md`에 정리합니다.

핵심 정리: 오늘은 단순히 합성 이미지 120장을 만드는 것이 아니라, 이미지와 YOLO 정답을 자동 생성하고 → 검사하고 → 생성 이력까지 추적하는 전체 과정을 배웠습니다.

---

# PART 7. Mini Challenge — 새로운 Source로 합성데이터를 만들고 Rotation 조건을 비교합니다

---

# 34. Mini Challenge의 목표를 확인합니다

지금까지는 강의에서 제공한 `main` 이미지로 **Cut-Paste → YOLO Label → QA → Metadata 추적**을 함께 실습했습니다. 이제는 **서로 다른 원본 이미지가 들어 있는 `challenge`** 데이터를 이용해 같은 작업을 스스로 해보겠습니다.

| 구분 | 본 실습 `main` | Mini Challenge `challenge` |
|---|---|---|
| Background | 20장 | **10장** |
| Object PNG | 3 Class × 5장 = 15장 | **3 Class × 3장 = 9장** |
| 진행 방식 | 함께 실습 | **기존 코드를 스스로 실행·확인** |

새 Python 파일을 만드는 과제가 아닙니다. 이미 작성한 `scripts/00_...py`부터 `06_...py`까지를 재사용합니다. 다만 **`--asset-set challenge`**를 지정하고, 두 실험을 구분하기 위해 **`--tag rot15`, `--tag rot30`**을 사용합니다.

```text
challenge 원본 확인
       ↓
객체 한 장 합성 + BBox 확인
       ↓
Rotation ±15° 데이터 90장 생성
       ↓
Rotation ±30° 데이터 90장 생성
       ↓
두 실험의 다른 조건이 같은지 확인
       ↓
BBox 재시각화 → Automatic QA → Human QA
       ↓
Metadata·이미지 비교 → 결론 기록
```

**오늘의 질문:** 같은 이미지와 생성 조건에서 회전 범위만 바꾸면, 객체의 모양·BBox·합성 장면의 자연스러움은 어떻게 달라질까요?

> 시작 전: 터미널이 `day03_detection_cutpaste` 프로젝트 폴더에 있고, 앞에서 사용한 `.venv`가 활성화되어 있는지 확인합니다. `main` 데이터나 기존 코드를 수정할 필요는 없습니다.

---

# 35. Challenge 요구사항 1 — 입력 데이터를 검사합니다

먼저 `challenge`에 필요한 원본 이미지가 모두 들어 있는지 확인합니다. 본 실습에서 배운 **입력 검사**를 새로운 Source에 적용하는 단계입니다.

```bash
python scripts/00_check_day03.py \
  --asset-set challenge
```

**확인할 결과:**

```text
Asset Set   : challenge
Backgrounds : 10
Class 0 drill: 3 / Alpha OK
Class 1 fire_extinguisher: 3 / Alpha OK
Class 2 wet_floor_sign: 3 / Alpha OK
Day 03 check: READY
```

`READY`는 이미지 수량·Class 순서·배경 읽기·투명 영역의 기본 조건을 통과했다는 뜻입니다. **합성 이미지의 품질까지 보장한다는 뜻은 아닙니다.** 결과를 확인하고 `challenge_notes.md`에 Background 10장, Object 9장, Class 3개 및 Class ID 순서를 기록하세요.

---

# 36. Challenge 요구사항 2 — 원본 이미지를 눈으로 확인합니다

자동 검사를 통과해도 실제 객체가 올바르게 보이는지 확인해야 합니다. 기존 미리보기 코드를 실행합니다.

```bash
python scripts/01_preview_assets.py \
  --asset-set challenge
```

**결과 파일:** `results/challenge/asset_preview.jpg`

```bash
explorer.exe results/challenge
```

미리보기에서 Background 대표 이미지 3장과 드릴·소화기·표지판 PNG 각 1장이 보이는지 확인하세요. 객체 주변의 체크무늬는 **투명 영역을 보여 주기 위한 미리보기 배경**입니다. `main`에서 보았던 이미지와 어떤 점이 다른지도 짧게 기록합니다.

---

# 37. Challenge 요구사항 3 — 표지판 한 장을 합성합니다

본 실습에서는 드릴을 합성했습니다. 이번에는 `wet_floor_sign`을 사용하여 **한 장의 합성 이미지와 YOLO Label**을 만들어 보세요.

```bash
python scripts/02_cutpaste_single.py \
  --asset-set challenge \
  --background bg_001.jpg \
  --class-name wet_floor_sign \
  --object wet_floor_sign_001.png \
  --scale 0.18 \
  --rotation 15 \
  --x-ratio 0.30 \
  --y-ratio 0.65
```

**결과 파일:**

```text
results/challenge/single/
├─ single_cutpaste.jpg       ← 합성 이미지
├─ single_cutpaste.txt       ← YOLO 정답
└─ single_cutpaste_bbox.jpg  ← BBox 확인 이미지
```

```bash
cat results/challenge/single/single_cutpaste.txt
explorer.exe results/challenge/single
```

직접 확인할 것은 세 가지입니다. **Class ID가 `2`인지, 빨간 BBox가 표지판을 감싸는지, 객체 주변에 불필요한 사각형 배경이 없는지** 살펴보세요. BBox가 정확해도 장면이 현실적인지는 별도로 판단하고, 그 이유를 기록합니다.

---

# 38. Challenge 요구사항 4 — 기준 실험 `rot15`를 생성합니다

이제 한 장 합성을 반복하여 **Rotation 범위 ±15°인 합성 이미지 90장**을 만듭니다. Scale은 `0.08~0.22`, Seed는 `2026`으로 고정합니다.

```bash
python scripts/03_generate_cutpaste_dataset.py \
  --asset-set challenge \
  --tag rot15 \
  --count 90 \
  --seed 2026 \
  --scale-min 0.08 \
  --scale-max 0.22 \
  --rotation 15
```

**생성 결과:**

```text
dataset/challenge/rot15/images/   ← JPG 90장
dataset/challenge/rot15/labels/   ← TXT 90개
reports/challenge/rot15_metadata.csv
```

Class별 생성 수를 확인해 보세요. 아래 코드는 Metadata의 `class_name`을 기준으로 각 객체 종류가 몇 장씩 생성되었는지 셉니다.

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv('reports/challenge/rot15_metadata.csv')
print(df['class_name'].value_counts())
PY
```

**확인 기준:** `drill`, `fire_extinguisher`, `wet_floor_sign`이 **각각 30장**이어야 합니다. 출력 순서는 달라도 됩니다.

---

# 39. Challenge 요구사항 5 — Rotation만 ±30°로 바꿔 비교합니다

두 번째 실험에서는 **`--rotation`만 `15`에서 `30`으로 변경**합니다. `--tag`도 별도 결과를 보관하기 위해 `rot30`으로 바꾸지만, 이 값은 이미지 합성 조건이 아니라 저장용 이름입니다.

| 조건 | 첫 번째 실험 | 두 번째 실험 |
|---|---|---|
| Source | challenge | challenge |
| 이미지 수 | 90장 | 90장 |
| Seed | 2026 | 2026 |
| Scale | 0.08~0.22 | 0.08~0.22 |
| **Rotation 범위** | **±15°** | **±30°** |
| Tag | rot15 | rot30 |

```bash
python scripts/03_generate_cutpaste_dataset.py \
  --asset-set challenge \
  --tag rot30 \
  --count 90 \
  --seed 2026 \
  --scale-min 0.08 \
  --scale-max 0.22 \
  --rotation 30
```

**생성 결과:** `dataset/challenge/rot30/`과 `reports/challenge/rot30_metadata.csv`

### 두 실험에서 Rotation 이외의 조건이 같은지 검사합니다

같은 Seed를 사용했더라도 실제로 조건이 같은지는 Metadata로 확인합니다. 아래 코드는 두 CSV에서 **Class·원본 파일·요청 Scale·배치 비율·Seed·회전 난수 비율**을 비교합니다.

```bash
python - <<'PY'
import pandas as pd

a = pd.read_csv('reports/challenge/rot15_metadata.csv')
b = pd.read_csv('reports/challenge/rot30_metadata.csv')

columns = [
    'file', 'asset_set', 'class_id', 'class_name',
    'background', 'object', 'scale', 'rotation_unit',
    'x_ratio', 'y_ratio', 'global_seed', 'sample_seed'
]

print('Non-rotation conditions same:', a[columns].equals(b[columns]))
PY
```

**예상:** `Non-rotation conditions same: True`

`True`이면 비교 대상 조건이 일치한다는 뜻입니다. `False`라면 두 명령어의 **Seed, Count, Scale, Source**를 다시 확인하세요.

> **중요:** 회전각이 바뀌면 회전 후 객체의 가로·세로 크기와 **Pixel BBox**도 바뀔 수 있습니다. 따라서 BBox 좌표가 달라도 오류라고 단정하지 않습니다. `x_ratio`, `y_ratio`는 배치 가능한 범위 안에서의 상대적 위치 비율이므로 두 실험에서 같게 유지합니다.

---

# 40. Challenge 요구사항 6 — 두 실험을 검수하고 비교합니다

이제 각 실험에 대해 **BBox 재시각화 → Automatic QA → Human QA 기록용 CSV 생성** 순서대로 진행합니다. **Automatic QA보다 Human QA CSV 생성 명령어를 먼저 실행하지 마세요.**

### ① Rotation ±15° 실험

```bash
python scripts/04_visualize_yolo_bbox.py \
  --asset-set challenge --tag rot15 --count 15

python scripts/05_validate_yolo_labels.py \
  --asset-set challenge --tag rot15

python scripts/06_make_qa_report.py \
  --asset-set challenge --tag rot15
```

### ② Rotation ±30° 실험

```bash
python scripts/04_visualize_yolo_bbox.py \
  --asset-set challenge --tag rot30 --count 15

python scripts/05_validate_yolo_labels.py \
  --asset-set challenge --tag rot30

python scripts/06_make_qa_report.py \
  --asset-set challenge --tag rot30
```

**생성 파일의 역할을 확인하세요.**

| 파일 | 확인할 내용 |
|---|---|
| `results/challenge/rot15/visualized/` | ±15° 이미지의 BBox 표시 15장 |
| `results/challenge/rot30/visualized/` | ±30° 이미지의 BBox 표시 15장 |
| `reports/challenge/rot15_validation.csv` | ±15° 전체 90장의 자동 검사 결과 |
| `reports/challenge/rot30_validation.csv` | ±30° 전체 90장의 자동 검사 결과 |
| `reports/challenge/rot15_human_qa.csv` | 사람이 직접 기록할 ±15° 검수표 |
| `reports/challenge/rot30_human_qa.csv` | 사람이 직접 기록할 ±30° 검수표 |

**Human QA 직접 수행:** 시각화된 15장씩을 열어 두 실험의 CSV에서 **해당 15개 행만** 검수합니다. `object_size_realistic`, `position_realistic`, `rotation_realistic`, `edge_natural`, `bbox_visual_ok`에 `yes/no`, `human_decision`에 `approved/needs_review/rejected`, `human_note`에 간단한 근거를 적습니다. **나머지 75장은 검수하지 않았다면 빈칸으로 둡니다.**

> `06_make_qa_report.py`는 빈 검수표를 만드는 코드이지 사람이 판정한 결과를 자동 입력하는 코드가 아닙니다. 이미 검수표가 있으면 기본적으로 덮어쓰기를 막으므로, 작성한 CSV를 보호하기 위해 **`--overwrite`를 사용하지 마세요.**

### ③ 실제 이미지와 Metadata를 함께 비교합니다

같은 파일명의 이미지를 두 폴더에서 열어 비교합니다. 예: `synthetic_00001.jpg`의 ±15°와 ±30° 결과입니다. 다음 코드는 **같은 번호의 회전각과 BBox**가 어떻게 달라졌는지 보여줍니다.

```bash
python - <<'PY'
import pandas as pd

cols = ['file', 'rotation', 'x1', 'y1', 'x2', 'y2']
a = pd.read_csv('reports/challenge/rot15_metadata.csv')[cols]
b = pd.read_csv('reports/challenge/rot30_metadata.csv')[cols]

print(a.head(3).to_string(index=False))
print('--- rot30 ---')
print(b.head(3).to_string(index=False))
PY
```

이 코드는 각각 **앞의 3장에 기록된 Rotation과 Pixel BBox를 출력**합니다. 동일한 `file` 행끼리 비교하되, **어느 범위가 더 자연스러운지는 실제 이미지를 보고 판단**해야 합니다.

### ④ 두 실험의 결과를 정리합니다

| 비교 항목 | ±15° | ±30° |
|---|---|---|
| Automatic QA PASS / FAIL | 직접 기록 | 직접 기록 |
| 확인한 이미지 수 | 15장 | 15장 |
| Human QA 승인 / 추가 검토 / 거절 | 직접 기록 | 직접 기록 |
| BBox가 자연스럽게 표시되었는가? | 직접 기록 | 직접 기록 |
| 객체의 방향과 배경이 자연스러운가? | 직접 기록 | 직접 기록 |
| 대표적인 문제 이미지 파일명 | 직접 기록 | 직접 기록 |

**판단 질문:** 회전 범위가 커지면서 객체 방향이 더 다양해졌나요? 반대로 객체가 비현실적인 각도로 놓이거나 BBox 크기가 크게 변한 사례는 없었나요? **Automatic QA PASS 수만으로 더 좋은 실험을 결정하지 마세요.**

---

# 41. Challenge 요구사항 7 — 실험 근거와 결론을 기록합니다

**작성할 파일:** `reports/challenge/challenge_notes.md`

```bash
mkdir -p reports/challenge
```

아래 형식으로 파일을 작성합니다. **숫자와 판정은 자신의 실행·검수 결과를 보고 채우세요.**

```markdown
# Day 03 Mini Challenge

## 1. 입력 데이터
- Background: 
- Object: 
- Class ID: 

## 2. Single Cut-Paste
- Background / Object: 
- YOLO Class ID: 
- Pixel BBox: 
- YOLO Label: 
- BBox 및 합성 장면 확인 결과: 

## 3. 두 Rotation 실험
| 항목 | ±15° | ±30° |
|---|---|---|
| 생성 장수 | | |
| Class별 개수 | | |
| Automatic QA PASS / FAIL | | |
| 직접 검수한 이미지 수 | | |
| Human QA 승인 / 추가 검토 / 거절 | | |

## 4. 한 변수 실험 확인
- 유지한 조건: 
- 변경한 조건: 
- Metadata 비교 결과(True/False): 
- 회전각 및 BBox에서 발견한 차이: 

## 5. 실패 사례 추적
- 파일명 / 실험 태그: 
- 눈으로 확인한 문제: 
- Metadata의 Background / Object / Scale / Rotation / Position: 
- 원인에 대한 내 생각: 

## 6. 최종 결론
- 더 적절하다고 판단한 Rotation 범위: 
- 선택한 이유(실제 이미지·QA 근거): 
- Automatic QA만으로 부족한 이유: 
```

최종 선택에는 정해진 정답이 없습니다. **회전각이 크다고 무조건 좋은 데이터는 아니며**, 합성 이미지가 현실적인지와 검수 결과를 근거로 설명하는 것이 중요합니다.

---

# 42. Mini Challenge 완료 여부를 직접 확인합니다

- [ ] `challenge` 입력 검사에서 `READY`를 확인했다.
- [ ] Background 10장과 Object 9장을 확인했다.
- [ ] Asset Preview를 열어 원본 상태를 확인했다.
- [ ] `wet_floor_sign`으로 Single Cut-Paste를 수행하고 Class ID·BBox·Label을 확인했다.
- [ ] `rot15`, `rot30`으로 각각 90장을 생성했다.
- [ ] 각 실험의 세 Class가 30장씩 생성되었는지 확인했다.
- [ ] Metadata 비교 결과 `Non-rotation conditions same: True`를 확인했다.
- [ ] 두 실험의 BBox를 각각 15장씩 재시각화했다.
- [ ] 두 실험의 Automatic QA PASS / FAIL을 확인했다.
- [ ] 시각화된 각 실험 15장에 대해 Human QA를 직접 기록했다.
- [ ] 문제 이미지 한 장 이상을 찾아 Metadata로 생성 조건을 추적했다.
- [ ] 두 Rotation 실험을 비교하고 선택 근거를 작성했다.
- [ ] `reports/challenge/challenge_notes.md`를 작성했다.

---

# 43. Mini Challenge 전체 진행 순서를 정리합니다

시간을 정해 서둘러 끝내기보다 **각 단계의 결과를 확인한 다음 다음 단계로 이동**하세요.

| 단계 | 무엇을 하나요? | 완료하면 확인할 결과 |
|---|---|---|
| **1. 원본 확인** | 입력 검사 → Asset Preview | `READY`, `asset_preview.jpg` |
| **2. 한 장 합성** | 표지판 Cut-Paste → BBox·Label 확인 | JPG·TXT·BBox 이미지 3개 |
| **3. 기준 데이터 생성** | Rotation ±15°로 90장 생성 | `rot15` JPG·TXT·Metadata |
| **4. 비교 데이터 생성** | Rotation만 ±30°로 변경 | `rot30` JPG·TXT·Metadata |
| **5. 동일 조건 확인** | 두 Metadata의 공통 조건 비교 | `Non-rotation conditions same: True` |
| **6. 결과 검수** | BBox 재시각화 → Automatic QA → Human QA | 각 실험별 QA CSV와 검수 기록 |
| **7. 원인 분석** | 같은 파일명 이미지와 Metadata 비교 | Rotation·BBox 변화 및 실패 사례 |
| **8. 최종 정리** | 비교표·결론·근거 기록 | `challenge_notes.md` |

```text
기존 코드 재사용
      ↓
새로운 challenge Source 적용
      ↓
Rotation 하나만 변경
      ↓
생성된 이미지와 Label 검수
      ↓
근거를 들어 더 적절한 조건 설명
```

**핵심 정리:** 이번 Mini Challenge의 목표는 합성 이미지를 많이 만드는 것이 아니라, **새 데이터에도 같은 개발·검수 절차를 적용하고, 하나의 조건을 바꾸어 결과를 비교하는 능력**을 기르는 것입니다.

---

# PART 8. 오늘의 핵심 결과만 기록하고 버전관리합니다

---

# 44. `day03_notes.md`에 확인된 결과만 기록합니다

본 실습에서 확인한 데이터 수·자동 검사·대표 Metadata·한 줄 결론만 남깁니다. 
Mini Challenge의 상세 비교는 이미 작성한 `reports/challenge/challenge_notes.md`에만 기록하므로 중복 작성하지 않습니다.

**파일: `day03_notes.md`**

```markdown
# 교과 11 · 3일차 실습 결과

## 본 실습 (`main`)

- 원본: 배경 JPG **20장**, 투명 객체 PNG **15장** (`drill`·`fire_extinguisher`·`wet_floor_sign`, 각 5장)
- 한 장 합성: `bg_001.jpg` + `drill_001.png` → 합성 이미지, BBox 이미지, YOLO Label 생성
- Batch: **120장** (`baseline`), Class별 **40장**, Seed `42`, Scale `0.08~0.22`, Rotation `±15°`
- Automatic QA: **PASS 120 / FAIL 0** (파일·Label 형식·좌표 검사 결과)
- Human QA: 공유된 CSV의 `human_decision`은 **미기록**. 실제 이미지 검수 후 판단해야 하므로 승인 수를 임의로 정하지 않음

## Metadata로 생성 조건 추적

`synthetic_00023.jpg` → 배경 `bg_015.jpg`, 객체 `wet_floor_sign_003.png` (Class `2`), 요청 Scale `0.12645`, 적용 Scale `0.12593`, Rotation `-7.347°`, Pixel BBox `(579, 1360, 716, 1575)`, Sample Seed `4200022`.

## Mini Challenge

Rotation `±15° / ±30°`의 **실제 수행·QA·비교·선택 근거**는 `reports/challenge/challenge_notes.md` 한 곳에서 관리합니다. 현재 제공된 자료에는 해당 실험의 실행 결과가 없어 결론을 미리 작성하지 않았습니다.

## 결론

**Cut-Paste는 합성 이미지와 YOLO Label을 함께 생성할 수 있지만, Automatic QA 통과만으로 학습 사용이 승인되는 것은 아닙니다.** 실제 이미지의 자연스러움과 BBox 품질을 사람이 확인해야 합니다.
```

> `PASS 120 / FAIL 0`은 YOLO Label 자동 검사가 통과했다는 뜻입니다. 공유된 Human QA CSV에는 직접 판정한 값이 없으므로 승인 수나 `rot15`·`rot30`의 우열을 임의로 기록하지 않습니다. 실제 검수·Challenge 완료 후 해당 기록만 갱신하세요.

---

# 45. `README.md`는 실행 안내만 간단히 남깁니다

README에는 실습 코드를 다시 옮겨 적지 않습니다. **무슨 데이터를 사용하고, 핵심 실행 명령이 무엇인지**만 정리합니다.

**파일: `README.md`**

````markdown
# Subject 11 · Day 03 — Cut-Paste Detection

배경 JPG와 투명 객체 PNG를 합성하여 **이미지·YOLO BBox·생성 이력(Metadata)**을 함께 만들고, 자동·수동 검수하는 실습입니다.

## 데이터와 결과

| 구분 | 원본 데이터 | 합성 결과 |
|---|---|---|
| `main` | 배경 20장, 객체 15장 | `baseline` 120장 (각 Class 40장) |
| `challenge` | 배경 10장, 객체 9장 | `rot15` / `rot30` 비교 실습 — 결과는 실행 후 확인 |

Class ID: `0 drill` · `1 fire_extinguisher` · `2 wet_floor_sign`

```text
합성 JPG + YOLO Label TXT
        ↓
Metadata CSV (생성 조건)
        ↓
BBox 시각화 → Automatic QA → Human QA
```

## Main 핵심 실행 명령

프로젝트 `day03_detection_cutpaste/`에서 기존 가상환경을 활성화한 뒤 실행합니다. 단일 합성과 미리보기 등 세부 단계는 3일차 실습 교재를 참고합니다.

```bash
python scripts/00_check_day03.py --asset-set main

python scripts/03_generate_cutpaste_dataset.py \
  --asset-set main --tag baseline --count 120 --seed 42 \
  --scale-min 0.08 --scale-max 0.22 --rotation 15

python scripts/04_visualize_yolo_bbox.py --asset-set main --tag baseline --count 20
python scripts/05_validate_yolo_labels.py --asset-set main --tag baseline
python scripts/06_make_qa_report.py --asset-set main --tag baseline
```

**실제 확인 결과:** Main 120장 생성, Automatic QA **PASS 120 / FAIL 0**. Human QA는 **사람이 직접 CSV에 입력**해야 합니다.

**기록 파일:** `day03_notes.md` (본 실습 요약), `reports/challenge/challenge_notes.md` (Mini Challenge 실험 비교).

> 주의: 동일한 `--tag`로 Batch 생성 코드를 재실행하면 해당 이미지·Label 폴더와 Metadata가 덮어써집니다. Human QA 파일은 기본적으로 재생성 시 보호되므로 `--overwrite`를 함부로 사용하지 마세요.

**다음 수업:** 2D 합성에서 3D Scene Rendering으로 확장하여 Camera·Light·Object를 바꾸고 RGB·BBox·Mask·Depth를 생성합니다.
````

---

# 46. Git에는 코드와 작은 문서만 기록합니다

이미지 원본과 대량 생성 결과까지 저장소에 넣을 필요는 없습니다. 3일차 프로젝트 폴더에 `.gitignore`를 작성합니다.

**파일: `.gitignore`**

```gitignore
# Python / local editor
__pycache__/
*.py[cod]
.venv/
.vscode/

# Provided source and generated datasets
data/day03_student_data/
dataset/
results/

# Generated CSV (keep small Markdown records in Git)
reports/**/*.csv

# OS
.DS_Store
Thumbs.db
```

이 설정은 `src/`, `scripts/`, `README.md`, `day03_notes.md`, `reports/challenge/challenge_notes.md` 같은 **코드와 짧은 Markdown 기록**을 남기고, 원본 이미지·합성 이미지·대량 CSV를 제외합니다. 조직의 반출 정책이 있으면 그 정책을 먼저 따릅니다.

---

# 47. 기존 Git 저장소에 변경사항을 Commit합니다

**3일차 폴더에서** 실행합니다. `git init`을 다시 하거나 Remote를 새로 추가하지 않습니다.

```bash
cd ..
git status

git add day03_detection_cutpaste/
git diff --cached --stat

git commit -m "docs: finalize day03 cut-paste lab"
git log -1 --oneline
```

`git diff --cached --stat`에서 `data/day03_student_data/`, `dataset/`, `results/`가 포함되지 않았는지 확인한 다음 Commit하세요. Push는 **기존 Remote가 있고 업로드가 허용된 환경에서만** 진행합니다.

---

# 48. 마지막으로 핵심만 자가 점검합니다

- [ ] `main`의 **120장 / Class별 40장** 생성 결과를 확인했다.
- [ ] YOLO Label과 BBox 시각화 결과를 확인했다.
- [ ] Automatic QA **PASS / FAIL**을 확인했다.
- [ ] 시각화된 이미지를 직접 보고 Human QA를 기록했다. *(미검수라면 미완료)*
- [ ] Mini Challenge의 `rot15`·`rot30` 결과와 선택 근거를 `challenge_notes.md`에 남겼다.
- [ ] `day03_notes.md`, `README.md`, Git Commit을 확인했다.

---

# 49. 3일차 최종 정리와 다음 수업 연결

오늘의 핵심은 한 가지입니다. Cut-Paste를 이용하면 이미지와 YOLO BBox를 함께 
자동 생성할 수 있지만, 생성 성공과 학습에 적합하다는 판정은 다릅니다. Label은 Automatic QA로 확인하고, 실제 장면의 자연스러움은 Human QA로 판단합니다.

4일차에는 2D 배경·객체 합성에서 **3D Scene Rendering**으로 확장합니다. Camera·Light·Object 조건을 변경하며 RGB·BBox·Mask·Depth 등의 데이터를 생성하는 
과정을 배웁니다.
