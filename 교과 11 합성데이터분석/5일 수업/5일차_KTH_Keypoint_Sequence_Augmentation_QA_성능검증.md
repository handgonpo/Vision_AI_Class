## 오늘의 핵심 질문

> 사람의 움직임을 관절 좌표(Keypoint)로 저장하고, 위치·크기·움직임 속도 등을 다양하게 변형해 학습데이터를 늘리면 AI의 동작인식 정확도가 좋아질까요?

1~4일차에는 이미지와 3D 모델을 이용해 새로운 학습데이터를 만들거나 기존 데이터를 보강하는 방법을 배웠습니다.

5일차에는 사람의 동작 영상에서 관절 좌표를 추출하고, 시간순으로 연결한 Keypoint Sequence를 보강하는 방법을 배웁니다.

마지막에는 보강 전후의 AI 모델 성능을 비교하여 데이터 보강이 실제 동작인식에 도움이 되는지 확인합니다.

1~4일차에는 이미지와 3D 장면을 중심으로 합성·보강 데이터를 만들었습니다.

```text
1일차
Image Augmentation + VAE

2일차
DCGAN + WGAN-GP + Diffusion

3일차
Cut-Paste + YOLO BBox

4일차
3D Rendering
→ RGB + BBox + Mask + Depth
```

5일차에는 데이터 형태가 **사람의 움직임을 나타내는 시계열 데이터**로 바뀝니다.

![[Pasted image 20260908150456.png]]

5일차에는 사람의 동작 영상을 관절 좌표가 시간순으로 이어진 Keypoint Sequence로 바꾸고, 위치·크기·노이즈·속도 변화를 적용해 새로운 학습데이터를 만듭니다.  

만든 보강 데이터는 Automatic QA와 Human QA로 정상 여부를 확인한 뒤, 통과한 데이터만 
원본 Train에 추가합니다.  

마지막에는 원본만 학습한 모델과 보강 데이터를 추가한 모델을 같은 Validation/Test로 비교하여 실제로 성능이 좋아졌는지 확인합니다.

```text
KTH Action Video
        ↓
YOLO Pose
        ↓
Frame별 COCO17 Keypoint
        ↓
Keypoint Sequence
        ↓
Translation
Scale
Jitter
Time Warp
        ↓
Sequence QA
        ↓
Original Train
VS
Original + Approved Augmented Train
        ↓
같은 Validation / Test
        ↓
성능 비교
```

오늘의 목적은 Keypoint JSON을 많이 만드는 것이 아닙니다.

다음 질문에 답할 수 있도록 원본 영상 → Keypoint → Sequence Augmentation → QA → 성능검증의 전체 흐름을 직접 실행하는 것이 목표입니다.

```text
동작 영상은 어떤 Sequence로 바뀌는가?
        ↓
어떤 변형이 동작 의미를 유지하는가?
        ↓
어떤 보강 데이터가 정상적인가?
        ↓
Train에만 보강을 적용했는가?
        ↓
같은 Validation/Test에서 성능이 달라졌는가?
        ↓
보강 데이터가 실제로 도움이 되었는가?
```

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

- 1~4일차와 같은 교과 11 프로젝트와 1일차 `.venv`를 그대로 이어서 사용할 수 있습니다.
- 제공된 `day05_student_data`의 `main`과 `challenge` 역할을 구분할 수 있습니다.
- KTH 수업용 파일명 `S11_boxing_T01.avi`에서 Subject·Action·조건 ID를 읽을 수 있습니다.
- `boxing`, `handwaving`, `handclapping`의 세 Action을 구분할 수 있습니다.
- Train·Validation·Test가 이미 Subject 단위로 분리된 이유를 설명할 수 있습니다.
- 같은 Subject가 Train과 Validation/Test에 섞이면 안 되는 이유를 설명할 수 있습니다.
- KTH AVI 영상을 OpenCV로 읽고 기본 정보를 확인할 수 있습니다.
- YOLO Pose로 한 영상의 사람을 추론하고 COCO 17 Keypoint를 추출할 수 있습니다.
- Pixel Keypoint를 0~1 정규화 좌표로 저장할 수 있습니다.
- 여러 Frame의 Keypoint가 하나의 Sequence가 되는 구조를 설명할 수 있습니다.
- 원본 영상과 Keypoint JSON을 다시 연결하여 Skeleton을 확인할 수 있습니다.
- Translation·Scale·Jitter·Time Warp의 의미를 설명할 수 있습니다.
- Train Sequence에만 보강을 적용할 수 있습니다.
- 생성된 Sequence의 Source·Method·Parameter·Seed를 Metadata로 기록할 수 있습니다.
- JSON 구조·NaN·좌표 범위·Keypoint 수·유효 Frame 비율을 Automatic QA로 확인할 수 있습니다.
- Human QA에서 동작 의미가 유지되는지 직접 판단할 수 있습니다.
- Variable-length Sequence를 고정 길이 통계 Feature로 바꾸는 이유를 설명할 수 있습니다.
- Original Train으로 Baseline 모델을 만들 수 있습니다.
- Original Train + Approved Augmented Train으로 Augmented 모델을 만들 수 있습니다.
- 두 모델을 동일한 Validation/Test에서 비교할 수 있습니다.
- Accuracy와 Confusion Matrix를 보고 보강 효과를 해석할 수 있습니다.
- Mini Challenge에서 본 실습과 겹치지 않는 Subject 데이터로 같은 Pipeline을 스스로 반복할 수 있습니다.
- Mini Challenge에서 Jitter 강도 하나만 변경하고, Validation으로 더 적절한 조건을 선택한 뒤 선택된 Final 조건만 Test에서 평가할 수 있습니다.
- 1~5일차의 합성데이터 방법을 하나의 흐름으로 정리할 수 있습니다.
- 오늘 만든 코드·결과·실패 사례를 README와 Git에 정리할 수 있습니다.

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260908150902.png]]

5일차에는 사람의 동작 영상을 **YOLO Pose로 관절 좌표 시계열 데이터로 변환**하고, 위치·크기·노이즈·속도 변화를 적용해 보강 데이터를 만듭니다.  

그다음 원본과 보강 데이터를 **QA로 검수하여 정상 데이터만 선택**하고, 원본만 학습한 모델과 보강 데이터를 추가한 모델을 같은 Validation/Test에서 비교합니다.  

마지막 Mini Challenge에서는 Jitter를 이용해 관절 좌표에 작은 변화를 주는 실습을 진행합니다.

Jitter 강도를 0.005와 0.015로 다르게 설정하고, 다른 조건은 동일하게 유지한 상태에서 AI의 동작인식 정확도를 비교합니다.

이를 통해 관절 좌표에 어느 정도의 변화를 주는 것이 AI 학습에 도움이 되는지 확인하고, 데이터 보강이 항상 성능을 높이는 것은 아니라는 점도 이해합니다.

```text
4일차 프로젝트 확인
        ↓
교과 11 Root 이동
        ↓
5일차 폴더 생성
        ↓
1일차 .venv 재사용
        ↓
day05_student_data 배치
        ↓

[본 실습 Source 확인]

main/train
24 Videos
S11 S12 S13 S14

main/val
12 Videos
S19 S20

main/test
12 Videos
S02 S03

Action
boxing
handwaving
handclapping
        ↓
입력 데이터 자동 검사
        ↓
Subject Leakage 확인
        ↓

[실험 A — Video → Keypoint]

대표 3개 영상
        ↓
YOLO Pose
        ↓
COCO17 Keypoint JSON
        ↓
Skeleton Preview
        ↓
Main 48개 전체 Keypoint 추출
        ↓

[실험 B — Sequence Augmentation]

Main Train 24 Sequence
        ↓
Translation
Scale
Jitter
Time Warp Fast
Time Warp Slow
        ↓
120 Augmented Sequence
        ↓
Metadata
        ↓

[QA]

Original + Augmented
        ↓
Automatic QA
        ↓
Skeleton Preview
        ↓
Human QA
        ↓
Approved / Needs Review / Rejected
        ↓

[성능검증]

Original Train
        ↓
Baseline

Original Train
+
Approved Augmented Train
        ↓
Augmented

        ↓
Same Validation
Same Test
        ↓
Accuracy
Confusion Matrix
        ↓
보강 효과 해석
        ↓

[Mini Challenge]

challenge
Train 12 / Val 6 / Test 6
        ↓
Keypoint 추출
        ↓
Jitter 0.005
VS
Jitter 0.015
        ↓
다른 조건 동일
        ↓
Automatic QA + Human QA
        ↓
같은 Validation에서 Candidate 비교
        ↓
Validation Accuracy + Confusion Matrix
        ↓
더 적절한 Jitter 선택
        ↓
Final Jitter Freeze
        ↓
선택된 조건만 Test S05 최종 평가
        ↓
Final Test Accuracy + Confusion Matrix
        ↓
한 변수 실험 해석
        ↓

README · Git
        ↓
교과 11 최종 정리
```

### 오늘의 데이터 운영 원칙

5일차 실습에서는 학습용 데이터(Train)와 성능평가용 데이터(Validation·Test)를 구분하여 사용합니다.

- Main / Challenge: `main`은 본 실습, `challenge`는 Mini Challenge에서 사용합니다.
- 데이터 분리: 이미 구분된 Train·Validation·Test는 다시 나누지 않습니다.
- 데이터 보강(Augmentation): Train에만 적용하며, Validation과 Test는 원본 그대로 유지합니다.
- 성능 비교: 원본으로 학습한 모델(Baseline)과 보강 데이터를 추가한 모델(Augmented)을 동일한 Validation·Test 데이터로 평가합니다.
- Mini Challenge: Jitter 강도(0.005 / 0.015)를 Validation으로 비교하여 더 적절한 값을 선택합니다. 선택한 값은 변경하지 않고(Freeze), 마지막에 Test로 최종 성능을 확인합니다.
    
핵심은 학습데이터만 보강하고, 평가데이터는 동일하게 유지하여 보강 전후의 성능을 공정하게 비교하는 것입니다.

---

# 오늘 사용할 실제 데이터

## 1. 제공되는 5일차 데이터

수업 시작 전에 다음 폴더가 제공됩니다.

```text
day05_student_data/
│
├─ main/
│  ├─ train/
│  │  ├─ S11_boxing_T01.avi
│  │  ├─ S11_boxing_T02.avi
│  │  ├─ S11_handwaving_T01.avi
│  │  ├─ S11_handwaving_T02.avi
│  │  ├─ S11_handclapping_T01.avi
│  │  ├─ S11_handclapping_T02.avi
│  │  ├─ ...
│  │  └─ S14_handclapping_T02.avi
│  │
│  ├─ val/
│  │  ├─ S19_...
│  │  └─ S20_...
│  │
│  └─ test/
│     ├─ S02_...
│     └─ S03_...
│
├─ challenge/
│  ├─ train/
│  │  ├─ S15_...
│  │  └─ S16_...
│  ├─ val/
│  │  └─ S21_...
│  └─ test/
│     └─ S05_...
│
├─ classes.txt
├─ README_DATA.txt
├─ split_summary.csv
└─ video_manifest.csv
```

## 2. 본 실습 `main`

```text
Train
S11 S12 S13 S14
4 Subjects × 3 Actions × 2 조건
= 24 Videos

Validation
S19 S20
2 Subjects × 3 Actions × 2 조건
= 12 Videos

Test
S02 S03
2 Subjects × 3 Actions × 2 조건
= 12 Videos
```

총 48개 영상입니다.

## 3. Mini Challenge `challenge`

```text
Train
S15 S16
= 12 Videos

Validation
S21
= 6 Videos

Test
S05
= 6 Videos

Total
= 24 Videos
```

본 실습과 다른 Subject를 사용하므로 같은 코드를 다시 사용해도 실제 입력 데이터는 달라집니다.

## 4. 오늘 사용할 동작 데이터 이해하기

5일차에는 KTH 데이터셋에서 다음 **3가지 사람 동작(Action)**을 사용합니다.

|Class ID|동작 이름|의미|
|---|---|---|
|0|`boxing`|권투 동작|
|1|`handwaving`|손 흔들기|
|2|`handclapping`|손뼉 치기|

각 영상의 파일명에는 사람 번호, 동작 종류, 촬영 조건이 표시되어 있습니다.

예: `S11_boxing_T01.avi`

- `S11`: 11번 사람(Subject)
- `boxing`: 권투 동작(Action)
- `T01`: 촬영 조건을 구분하는 번호
    

## 5. 오늘은 어떤 데이터를 보강하나요?

5일차에는 영상에서 추출한 **관절 좌표(Keypoint)**를 다음 네 가지 방법으로 변형합니다.

|보강 방법|쉽게 이해하면|
|---|---|
|Translation|사람의 위치를 조금 이동|
|Scale|사람의 크기를 조금 변경|
|Jitter|관절 좌표에 작은 오차 추가|
|Time Warp|동작을 빠르게 또는 느리게 변경|

중요: 보강할 때는 권투가 손 흔들기로 바뀌는 것처럼 원래 동작의 의미가 달라지지 않도록 해야 합니다.

각 보강 방법의 구체적인 원리와 코드는 이후 실습에서 직접 배웁니다.

---

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | 전체 Sequence Pipeline 작성 | 영상부터 성능비교까지 실행 |
| pathlib | Main·Challenge·Split 경로 관리 | 데이터 위치를 안전하게 연결 |
| OpenCV | AVI 읽기·Skeleton Video 저장 | 영상 입출력 |
| Ultralytics YOLO Pose | 사람 Pose 추출 | Frame에서 COCO17 관절 찾기 |
| COCO17 Keypoint | 17개 관절 표현 | 사람 자세를 좌표로 저장 |
| NumPy | Sequence·Interpolation·Augmentation | 시계열 좌표 계산 |
| Pandas | Metadata·QA·Feature CSV | 결과를 표로 저장 |
| Translation | 전체 관절 위치 이동 | 화면 위치 변화 |
| Scale | 관절 크기 변화 | 사람 크기·거리 변화 |
| Jitter | 좌표에 작은 Noise | Pose 오차·미세 자세 변화 |
| Time Warp | Frame 길이 변화 | 빠른/느린 동작 |
| Subject-wise Split | 사람 단위 분리 | 같은 사람의 정보 누수 방지 |
| Automatic QA | 구조·범위 자동 검사 | 잘못된 JSON 선별 |
| Human QA | 동작 의미 검수 | 사람이 현실성 확인 |
| Statistical Sequence Feature | 가변 길이 Sequence를 고정 Feature로 변환 | 모델 입력 만들기 |
| Random Forest | 교육용 Action Classifier | Baseline과 Augmented 비교 |
| Accuracy / Confusion Matrix | 보강 효과 확인 | 어떤 Action이 맞고 틀렸는지 비교 |
| Metadata | Source·Method·Parameter·Seed 기록 | Data Lineage |
| Git | 코드와 실험 기록 관리 | 재현 가능한 수업 결과 저장 |

---

# PART 1. 4일차 프로젝트에서 5일차를 이어서 시작합니다

---

# 1. 교과 11의 5일을 연결합니다

![[Pasted image 20261015173121.png]]

5일차에는 5일 동안 반복한 다음 원칙을 실제 성능비교까지 연결합니다.

```text
Real Data (원본 데이터)
        ↓
부족한 조건 찾기
어떤 데이터가 더 필요한지 확인
        ↓
Synthetic / Augmented Data
새로운 데이터 생성 또는 기존 데이터 보강
        ↓
QA (품질 검사)
생성한 데이터가 정상인지 확인
        ↓
Metadata (생성 정보 기록)
어떤 방법과 조건으로 만들었는지 기록
        ↓
Train에 추가
검사를 통과한 보강 데이터를 학습에 사용
        ↓
Same Validation / Test
동일한 검증·테스트 데이터로 평가
        ↓
성능 비교
보강 전후 AI 모델의 정확도 비교
```

# 2. 교과 11 프로젝트 Root로 이동합니다

```bash
cd ~/ai_vision/subject11_synthetic_data
```

확인합니다.

```bash
ls
```

```text
day01_image_aug_vae
day02_generative_models
day03_detection_cutpaste
day04_3d_segmentation
```

# 3. 5일차 폴더를 만듭니다

```bash
mkdir -p day05_keypoint_sequence
cd day05_keypoint_sequence
```

```bash
mkdir -p \
src \
scripts \
data/day05_student_data \
data/day05_student_data/main/train \
data/day05_student_data/main/val \
data/day05_student_data/main/test \
data/day05_student_data/challenge/train \
data/day05_student_data/challenge/val \
data/day05_student_data/challenge/test \
data/keypoints \
data/features \
models \
results/main \
results/challenge \
reports/main \
reports/challenge
```

```bash
touch requirements_day05.txt
touch src/day05_common.py

touch scripts/00_check_day05.py
touch scripts/01_extract_keypoints.py
touch scripts/02_visualize_keypoints.py
touch scripts/03_augment_keypoints.py
touch scripts/04_qa_keypoints.py
touch scripts/05_build_features.py
touch scripts/06_compare_baseline_augmented.py

touch day05_notes.md
touch final_reflection.md
touch README.md
touch .gitignore
```

```bash
tree -L 2
```

예상:

```text
day05_keypoint_sequence/
├─ data/
│  ├─ features/
│  └─ keypoints/
├─ models/
├─ reports/
├─ results/
├─ scripts/
├─ src/
├─ requirements_day05.txt
├─ day05_notes.md
├─ final_reflection.md
├─ README.md
└─ .gitignore
```

---
# 4. 제공된 `day05_student_data`를 배치합니다

현재 준비한 `day05_student_data` 폴더 전체를 다음 위치에 복사합니다.

```text
day05_keypoint_sequence/
└─ data/
   └─ day05_student_data/
```

확인:

```bash
tree data/day05_student_data -L 3
```

오늘은 `boxing`, `handwaving`, `handclapping` 원본 전체 폴더를 사용하지 않습니다.

### 4-1. 동작 클래스 파일(`classes.txt`)을 확인합니다

5일차에서는 권투, 손 흔들기, 손뼉 치기 세 가지 동작을 학습합니다.

각 동작을 번호로 구분하기 위해 `classes.txt` 파일을 사용합니다. 이 파일은 제공된 `day05_student_data`에 포함되어 있어야 합니다.

파일을 다음 명령어로 생성합니다.

```bash
cat > data/day05_student_data/classes.txt <<'EOF'
0 boxing
1 handwaving
2 handclapping
EOF
```

작성한 파일의 내용을 확인합니다.

```bash
cat data/day05_student_data/classes.txt
```

실행 결과:

```
0 boxing
1 handwaving
2 handclapping
```

각 번호는 다음 동작을 의미합니다.

|Class ID|동작|의미|
|---|---|---|
|0|`boxing`|권투|
|1|`handwaving`|손 흔들기|
|2|`handclapping`|손뼉 치기|

최종 파일 위치는 다음과 같습니다.

```
day05_keypoint_sequence/
└── data/
    └── day05_student_data/
        ├── main/
        ├── challenge/
        ├── classes.txt  ← 확인 완료
        ├── README_DATA.txt
        ├── split_summary.csv
        └── video_manifest.csv
```

### 4-2. 제공해준 데이터를 경로에 맞게 복사

```
~/project11/day05_keypoint_sequence/
└── data/
    └── day05_student_data/
        └── main/
	        ├── train (24)
	        ├── val (12)
            └── test (12)
```

---
# 5. 1일차 가상환경을 그대로 사용합니다

```bash
source ../day01_image_aug_vae/.venv/bin/activate
```

```bash
which python
```

# 6. 5일차에 필요한 라이브러리를 확인합니다

`requirements_day05.txt`:

```text
ultralytics
opencv-python
numpy
pandas
scikit-learn
joblib
```

```bash
pip install -r ../requirements_day05.txt
```

```bash
python -c "import cv2, ultralytics, sklearn, joblib; print('Day05 libraries OK')"
```

```bash
pip freeze > reports/environment_freeze_day05.txt
```

# 7. YOLO Pose 모델을 준비합니다

교과 10에서 사용한 모델 또는 수업과 함께 제공된 모델을 다음 위치에 둡니다.

```text
models/yolo11n-pose.pt
```

원본 파일 (교과 10)

```bash
~/ai_vision/subject10_pose_action/yolo11n-pose.pt
```

복사할 위치 (교과 11의 5일차)

```
~/ai_visionproject11/day05_keypoint_sequence/models/
```

먼저 대상 폴더를 만들고 파일을 복사합니다.

```bash
# ① models 폴더 생성 (이미 있으면 그대로 유지)
mkdir -p ~/ai_visionproject11/day05_keypoint_sequence/models

# ② 교과 10의 YOLO Pose 모델 복사
cp ~/ai_vision/subject10_pose_action/yolo11n-pose.pt \
   ~/ai_visionproject11/day05_keypoint_sequence/models/
```

복사가 정상적으로 되었는지 확인

```bash
ls -lh ~/ai_visionproject11/day05_keypoint_sequence/models/yolo11n-pose.pt
```

---

# 8. Main과 Challenge에서 사용할 공통 경로 코드를 작성합니다

5일차에는 원본 영상, 관절 좌표(Keypoint), 보강 데이터, 성능평가 결과 등 여러 파일을 생성하고 저장합니다.

본 실습(`main`)과 Mini Challenge(`challenge`)에서 같은 코드를 사용할 수 있도록, 먼저 파일의 저장 위치를 정리하겠습니다.

### ① 각 데이터는 어디에 저장하나요?

모든 경로는 `day05_keypoint_sequence` 폴더를 기준으로 합니다.

|구분|저장 위치|역할|
|---|---|---|
|Input Video|`data/day05_student_data/`|제공받은 원본 동작 영상|
|Original Keypoint|`data/keypoints/`|영상에서 추출한 원본 관절 좌표|
|Augmented Keypoint|`data/keypoints/`|위치·크기·속도 등을 변형한 관절 좌표|
|Feature|`data/features/`|동작인식 모델 학습에 사용할 숫자 특징|
|Report|`reports/`|데이터 검사 및 성능평가 결과|

각 폴더 안에서는 `main`과 `challenge`를 구분하여 관리합니다.

### ② 실제 경로를 예로 들어봅니다

```
원본 동작 영상
data/day05_student_data/main/train/
        ↓
YOLO Pose로 관절 추출
        ↓
원본 Keypoint 저장
data/keypoints/main/original/train/
        ↓
관절 좌표 보강
        ↓
보강 Keypoint 저장
data/keypoints/main/augmented/{tag}/
        ↓
Feature 생성
data/features/main/{tag}/
        ↓
검사 및 성능평가 결과 저장
reports/main/
```

여기서 `{tag}`는 실험 결과를 구분하기 위한 이름입니다. 예를 들어 서로 다른 보강 방법이나 실험 조건으로 생성한 데이터를 별도로 저장할 때 사용합니다.

### ③ 왜 공통 코드를 작성하나요?

본 실습과 Mini Challenge는 사용하는 영상만 다르고 처리 과정은 거의 같습니다.

따라서 파일 경로를 매번 직접 작성하지 않고, 하나의 공통 Python 파일에서 관리하면 같은 코드를 재사용할 수 있습니다.

**파일: `src/day05_common.py`**

```python
from __future__ import annotations

import re
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]

DATA_ROOT = ROOT / "data" / "day05_student_data"
KEYPOINT_ROOT = ROOT / "data" / "keypoints"
FEATURE_ROOT = ROOT / "data" / "features"
REPORT_ROOT = ROOT / "reports"
RESULT_ROOT = ROOT / "results"
MODEL_ROOT = ROOT / "models"


VIDEO_PATTERN = re.compile(
    r"^(?P<subject>S\d{2})_"
    r"(?P<action>[A-Za-z0-9\-]+)_"
    r"(?P<take>T\d{2})\.avi$"
)


def read_classes() -> dict[int, str]:
    path = DATA_ROOT / "classes.txt"

    if not path.exists():
        raise FileNotFoundError(path)

    mapping = {}

    for index, raw_line in enumerate(
        path.read_text(encoding="utf-8").splitlines()
    ):
        line = raw_line.strip()

        if not line:
            continue

        parts = line.split(maxsplit=1)

        if len(parts) == 2 and parts[0].isdigit():
            class_id = int(parts[0])
            class_name = parts[1].strip()
        else:
            class_id = index
            class_name = line

        mapping[class_id] = class_name

    if not mapping:
        raise RuntimeError("classes.txt가 비어 있습니다.")

    return mapping


def parse_video_name(path: Path) -> dict[str, str]:
    match = VIDEO_PATTERN.match(path.name)

    if match is None:
        raise ValueError(
            f"수업용 파일명 형식이 아닙니다: {path.name}"
        )

    return match.groupdict()


def video_dir(asset_set: str, split: str) -> Path:
    return DATA_ROOT / asset_set / split


def list_videos(asset_set: str, split: str) -> list[Path]:
    directory = video_dir(asset_set, split)

    if not directory.exists():
        return []

    return sorted(directory.glob("*.avi"))


def original_keypoint_dir(
    asset_set: str,
    split: str,
) -> Path:
    path = KEYPOINT_ROOT / asset_set / "original" / split
    path.mkdir(parents=True, exist_ok=True)
    return path


def augmented_keypoint_dir(
    asset_set: str,
    tag: str,
) -> Path:
    path = KEYPOINT_ROOT / asset_set / "augmented" / tag
    path.mkdir(parents=True, exist_ok=True)
    return path


def feature_dir(asset_set: str, tag: str) -> Path:
    path = FEATURE_ROOT / asset_set / tag
    path.mkdir(parents=True, exist_ok=True)
    return path


def report_dir(asset_set: str) -> Path:
    path = REPORT_ROOT / asset_set
    path.mkdir(parents=True, exist_ok=True)
    return path


def result_dir(asset_set: str) -> Path:
    path = RESULT_ROOT / asset_set
    path.mkdir(parents=True, exist_ok=True)
    return path
```

### 의사코드로 읽어보기

```text
[시작]
   ↓
① 5일차 프로젝트의 기준 경로(ROOT)를 찾는다
   ↓
② 데이터와 결과를 저장할 공통 경로를 설정한다
   ├─ DATA_ROOT      → 제공받은 원본 영상
   ├─ KEYPOINT_ROOT  → 관절 좌표 데이터
   ├─ FEATURE_ROOT   → 모델 학습용 특징 데이터
   ├─ REPORT_ROOT    → 검사 및 평가 보고서
   ├─ RESULT_ROOT    → 실험 결과
   └─ MODEL_ROOT     → AI 모델 파일
   ↓
③ 영상 파일명의 형식을 정의한다
   │
   │  예: S11_boxing_T01.avi
   │
   ├─ S11    → Subject(사람 번호)
   ├─ boxing → Action(동작 종류)
   └─ T01    → Take(수업용 촬영 조건 ID)
   ↓
④ 필요한 기능(함수)을 정의한다
   │
   ├─ read_classes()
   │     ↓
   │   classes.txt 파일이 있는가?
   │     ├─ 아니요 → FileNotFoundError 발생
   │     └─ 예 → 파일 내용 읽기
   │                ↓
   │             빈 줄 제외
   │                ↓
   │             Class ID와 동작 이름 분리
   │                ↓
   │             클래스 정보가 비어 있는가?
   │                ├─ 예 → RuntimeError 발생
   │                └─ 아니요 → 클래스 정보 반환
   │
   ├─ parse_video_name()
   │     ↓
   │   영상 파일명이 정해진 형식과 일치하는가?
   │     ├─ 아니요 → ValueError 발생
   │     └─ 예 → Subject / Action / Take 반환
   │
   ├─ video_dir()
   │     ↓
   │   main 또는 challenge와
   │   train / val / test를 연결
   │     ↓
   │   원본 영상 폴더 경로 반환
   │
   ├─ list_videos()
   │     ↓
   │   원본 영상 폴더가 존재하는가?
   │     ├─ 아니요 → 빈 목록 반환
   │     └─ 예 → AVI 파일 검색
   │                ↓
   │             파일명순으로 정렬
   │                ↓
   │             영상 파일 목록 반환
   │
   ├─ original_keypoint_dir()
   │     ↓
   │   원본 Keypoint 저장 폴더 생성 또는 확인
   │     ↓
   │   폴더 경로 반환
   │
   ├─ augmented_keypoint_dir()
   │     ↓
   │   실험 이름(tag)에 맞는
   │   보강 Keypoint 저장 폴더 생성 또는 확인
   │     ↓
   │   폴더 경로 반환
   │
   ├─ feature_dir()
   │     ↓
   │   Feature 저장 폴더 생성 또는 확인
   │     ↓
   │   폴더 경로 반환
   │
   ├─ report_dir()
   │     ↓
   │   보고서 저장 폴더 생성 또는 확인
   │     ↓
   │   폴더 경로 반환
   │
   └─ result_dir()
         ↓
       실험 결과 저장 폴더 생성 또는 확인
         ↓
       폴더 경로 반환

⑤ 이후 실습 코드에서 필요한 함수를 호출한다
   ↓
   ├─ 원본 영상 목록 가져오기
   ├─ 영상 파일명에서 Subject / Action 확인
   ├─ 원본 Keypoint 저장 위치 가져오기
   ├─ 보강 Keypoint 저장 위치 가져오기
   ├─ Feature 저장 위치 가져오기
   └─ 보고서 및 실험 결과 저장 위치 가져오기
   ↓
[종료]
```

---
# 9. 실제 수업 데이터가 정확한지 자동으로 검사합니다

5일차 실습을 시작하기 전에 제공된 KTH 동작 영상이 올바르게 준비되어 있는지 확인합니다.

이번에 작성할 코드는 다음 내용을 자동으로 검사합니다.

- 데이터 수량: 본 실습(Main)은 총 48개, Mini Challenge는 총 24개의 영상이 준비되어 있는지 확인합니다.
    
- 동작 종류(Class): `boxing`, `handwaving`, `handclapping`의 3가지 동작이 올바르게 구성되어 있는지 확인합니다.
    
- 데이터 분리: Train(학습), Validation(검증), Test(시험)에 같은 사람(Subject)의 영상이 중복되지 않았는지 확인합니다.
    
- 영상 상태: OpenCV로 AVI 파일을 정상적으로 열 수 있는지, FPS(초당 프레임 수), Frame(영상 프레임 수), Width·Height(영상 크기)가 유효한지 검사합니다.
    

이 검사를 먼저 진행하는 이유는 잘못된 데이터로 관절 좌표를 추출하거나 모델을 학습하는 문제를 미리 방지하기 위해서입니다.

**파일: `scripts/00_check_day05.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

import cv2


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (
    list_videos,
    parse_video_name,
    read_classes,
)


EXPECTED = {
    "main": {
        "train": {
            "count": 24,
            "subjects": {"S11", "S12", "S13", "S14"},
        },
        "val": {
            "count": 12,
            "subjects": {"S19", "S20"},
        },
        "test": {
            "count": 12,
            "subjects": {"S02", "S03"},
        },
    },
    "challenge": {
        "train": {
            "count": 12,
            "subjects": {"S15", "S16"},
        },
        "val": {
            "count": 6,
            "subjects": {"S21"},
        },
        "test": {
            "count": 6,
            "subjects": {"S05"},
        },
    },
}


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="all",
        choices=["main", "challenge", "all"],
    )

    return parser.parse_args()


def check_video(
    path: Path,
) -> tuple[float, int, int, int]:
    capture = cv2.VideoCapture(str(path))

    if not capture.isOpened():
        raise RuntimeError(
            f"영상을 열 수 없습니다: {path}"
        )

    fps = float(
        capture.get(cv2.CAP_PROP_FPS)
    )

    frames = int(
        capture.get(cv2.CAP_PROP_FRAME_COUNT)
    )

    width = int(
        capture.get(cv2.CAP_PROP_FRAME_WIDTH)
    )

    height = int(
        capture.get(cv2.CAP_PROP_FRAME_HEIGHT)
    )

    capture.release()

    if (
        fps <= 0
        or frames <= 0
        or width <= 0
        or height <= 0
    ):
        raise RuntimeError(
            f"영상 정보가 비정상입니다: {path.name}"
        )

    return fps, frames, width, height


def check_asset_set(
    asset_set: str,
) -> None:
    classes = read_classes()

    expected_classes = {
        0: "boxing",
        1: "handwaving",
        2: "handclapping",
    }

    if classes != expected_classes:
        raise RuntimeError(
            "classes.txt가 수업 기준과 다릅니다.\n"
            f"현재: {classes}\n"
            f"기준: {expected_classes}"
        )

    split_subjects = {}

    print("=" * 68)
    print(f"Day05 Data Check - {asset_set}")
    print("=" * 68)

    for split in ["train", "val", "test"]:
        videos = list_videos(
            asset_set,
            split,
        )

        expected = EXPECTED[
            asset_set
        ][split]

        if len(videos) != expected["count"]:
            raise RuntimeError(
                f"{asset_set}/{split} 수가 "
                "수업 기준과 다릅니다.\n"
                f"기대: {expected['count']}\n"
                f"현재: {len(videos)}"
            )

        subjects = set()
        actions = set()
        takes = set()

        for path in videos:
            info = parse_video_name(path)

            subjects.add(
                info["subject"]
            )

            actions.add(
                info["action"]
            )

            takes.add(
                info["take"]
            )

            check_video(path)

        if subjects != expected["subjects"]:
            raise RuntimeError(
                f"{asset_set}/{split} Subject가 "
                "수업 기준과 다릅니다.\n"
                f"현재: {sorted(subjects)}"
            )

        if actions != {
            "boxing",
            "handwaving",
            "handclapping",
        }:
            raise RuntimeError(
                f"{asset_set}/{split} Action 구성이 "
                "수업 기준과 다릅니다."
            )

        if takes != {"T01", "T02"}:
            raise RuntimeError(
                f"{asset_set}/{split} T ID가 "
                "수업 기준과 다릅니다."
            )

        split_subjects[split] = subjects

        fps, _, width, height = (
            check_video(videos[0])
        )

        print(
            f"{split:5s}: "
            f"{len(videos):2d} videos / "
            f"Subjects {sorted(subjects)} / "
            f"FPS {fps:.2f} / "
            f"Size {width}x{height}"
        )

    if (
        split_subjects["train"] & split_subjects["val"]
        or split_subjects["train"] & split_subjects["test"]
        or split_subjects["val"] & split_subjects["test"]
    ):
        raise RuntimeError(
            f"{asset_set}에서 Subject Leakage가 있습니다."
        )

    print("Subject Leakage: NONE")


def main() -> None:
    args = parse_args()

    asset_sets = (
        ["main", "challenge"]
        if args.asset_set == "all"
        else [args.asset_set]
    )

    for asset_set in asset_sets:
        check_asset_set(asset_set)

    print("=" * 68)
    print("Day05 data check: READY")


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[프로그램 시작]
        ↓
① 필요한 라이브러리와 공통 함수 불러오기
   ├─ argparse → 실행 옵션 처리
   ├─ OpenCV → 영상 정보 확인
   └─ day05_common.py → 경로·파일명·Class 정보 가져오기
        ↓
② 수업에서 사용할 데이터 기준 설정
   │
   ├─ Main
   │   ├─ Train : 24개 (S11, S12, S13, S14)
   │   ├─ Val   : 12개 (S19, S20)
   │   └─ Test  : 12개 (S02, S03)
   │
   ├─ Challenge
   │   ├─ Train : 12개 (S15, S16)
   │   ├─ Val   :  6개 (S21)
   │   └─ Test  :  6개 (S05)
   │
   └─ Action Class
       ├─ 0 : boxing
       ├─ 1 : handwaving
       └─ 2 : handclapping
        ↓
③ 실행 옵션 확인 — parse_args()
   │
   ├─ --asset-set main      → 본 실습만 검사
   ├─ --asset-set challenge → Mini Challenge만 검사
   └─ --asset-set all       → 두 데이터 모두 검사 (기본값)
        ↓
④ 검사할 데이터 선택 — main()
        ↓
⑤ 데이터 검사 시작 — check_asset_set()
   │
   ├─ classes.txt 읽기
   │       ↓
   │   Class ID와 동작 이름이 정확한가?
   │       ├─ 아니요 → 오류 발생 및 중단
   │       └─ 예 → 다음 검사
   │
   └─ Train → Val → Test 순서로 검사
           ↓
⑥ 각 폴더의 AVI 영상 목록 가져오기 — list_videos()
           ↓
   영상 개수가 수업 기준과 일치하는가?
           ├─ 아니요 → 오류 발생 및 중단
           └─ 예 → 다음 검사
           ↓
⑦ 영상 파일명 분석 — parse_video_name()
   │
   │  예: S11_boxing_T01.avi
   │
   ├─ Subject : S11
   ├─ Action  : boxing
   └─ Take    : T01
           ↓
⑧ 각 영상 상태 검사 — check_video()
   │
   ├─ OpenCV로 영상 열기
   │       ↓
   │   정상적으로 열리는가?
   │       ├─ 아니요 → 오류 발생 및 중단
   │       └─ 예 → 영상 정보 확인
   │
   ├─ FPS    → 초당 프레임 수
   ├─ Frames → 전체 프레임 수
   ├─ Width  → 영상 가로 크기
   └─ Height → 영상 세로 크기
           ↓
   모든 값이 0보다 큰가?
           ├─ 아니요 → 오류 발생 및 중단
           └─ 예 → 정상
           ↓
⑨ 폴더별 데이터 구성 확인
   │
   ├─ Subject가 지정된 사람 번호와 일치하는가?
   ├─ Action이 3가지 모두 포함되어 있는가?
   └─ Take에 T01과 T02가 모두 있는가?
           ↓
   모든 조건을 만족하는가?
           ├─ 아니요 → 오류 발생 및 중단
           └─ 예 → 검사 결과 출력
           ↓
⑩ Train·Val·Test의 Subject 중복 확인
   │
   ├─ Train ↔ Val
   ├─ Train ↔ Test
   └─ Val ↔ Test
           ↓
   같은 Subject가 서로 다른 Split에 존재하는가?
           ├─ 예 → Subject Leakage 오류 발생 및 중단
           └─ 아니요 → Subject Leakage: NONE
           ↓
⑪ 선택한 데이터의 검사가 모두 끝났는가?
   │
   ├─ 아니요 → 다음 데이터(Main 또는 Challenge) 검사
   └─ 예 → 최종 결과 출력
           ↓
   "Day05 data check: READY"
           ↓
[검사 완료]
```

실행합니다.

```bash
python scripts/00_check_day05.py \
  --asset-set all
```

터미널 실행 결과

```
====================================================================
Day05 Data Check - main
====================================================================
train: 24 videos / Subjects ['S11', 'S12', 'S13', 'S14'] / FPS 25.00 / Size 160x120
val  : 12 videos / Subjects ['S19', 'S20'] / FPS 25.00 / Size 160x120
test : 12 videos / Subjects ['S02', 'S03'] / FPS 25.00 / Size 160x120
Subject Leakage: NONE
====================================================================
Day05 Data Check - challenge
====================================================================
train: 12 videos / Subjects ['S15', 'S16'] / FPS 25.00 / Size 160x120
val  :  6 videos / Subjects ['S21'] / FPS 25.00 / Size 160x120
test :  6 videos / Subjects ['S05'] / FPS 25.00 / Size 160x120
Subject Leakage: NONE
====================================================================
Day05 data check: READY
```

---

# PART 2. KTH 영상을 Keypoint Sequence로 변환합니다

---

# 10. 영상과 Keypoint Sequence의 차이를 이해합니다

영상(Video)은 여러 장의 이미지(Frame)가 시간순으로 연결된 것입니다.

**YOLO Pose**는 각 Frame에서 사람의 코, 어깨, 팔꿈치, 손목 등 **17개 관절 위치(Keypoint)**를 찾아줍니다.

```
KTH 동작 영상 (.avi)
        ↓
Frame 000 → 17개 Keypoint
Frame 001 → 17개 Keypoint
Frame 002 → 17개 Keypoint
        ...
        ↓
시간순으로 연결
        ↓
Keypoint Sequence
```

이렇게 관절 좌표를 시간순으로 연결한 데이터를 Keypoint Sequence(관절 좌표 시계열)라고 합니다. 이미 배웠지만 다시 작성해둡니다.

즉, **Keypoint는 한 순간의 자세**, **Keypoint Sequence는 시간에 따른 움직임**을 나타냅니다.

# 11. COCO 17 Keypoint를 확인합니다

YOLO Pose는 사람의 주요 관절과 얼굴 위치를 **COCO17 형식의 17개 Keypoint**로 표현합니다.

| 번호    | 관절 위치      |
| ----- | ---------- |
| 0     | 코          |
| 1~2   | 왼쪽·오른쪽 눈   |
| 3~4   | 왼쪽·오른쪽 귀   |
| 5~6   | 왼쪽·오른쪽 어깨  |
| 7~8   | 왼쪽·오른쪽 팔꿈치 |
| 9~10  | 왼쪽·오른쪽 손목  |
| 11~12 | 왼쪽·오른쪽 엉덩이 |
| 13~14 | 왼쪽·오른쪽 무릎  |
| 15~16 | 왼쪽·오른쪽 발목  |

![[Pasted image 20261016111357.png]]

### Keypoint 좌표를 간단히 복습합니다

교과 10에서는 YOLO Pose를 이용해 사람의 17개 Keypoint를 추출하고, 관절 위치를 숫자로 표현하는 방법을 배웠습니다.

각 Keypoint는 다음 세 가지 값으로 저장합니다.

```
[x_normalized, y_normalized, confidence]
```

- **x, y:** 영상 크기를 기준으로 0~1 사이로 정규화한 관절 위치
    
- **confidence:** AI가 예측한 관절 위치의 신뢰도
    

교과 11에서는 이 개념을 다시 깊이 학습하지 않습니다.

대신 **영상에서 추출한 관절 좌표를 시간순으로 연결하고, 위치·크기·좌표 오차·움직임 속도를 변형하여 새로운 학습데이터를 만드는 것**에 집중합니다.

즉, **교과 10이 Keypoint를 활용한 동작인식 수업이었다면, 교과 11은 Keypoint를 보강하여 동작인식 모델의 성능이 달라지는지 검증하는 수업**입니다.

---
# 12. 대표 영상 3개로 Keypoint 추출을 먼저 확인합니다

처음부터 Main 영상 48개를 모두 처리하지 않고, **세 가지 동작에서 대표 영상 1개씩을 선택하여 먼저 테스트**합니다.

```
S11_boxing_T01.avi       → 권투
S11_handwaving_T01.avi   → 손 흔들기
S11_handclapping_T01.avi → 손뼉 치기
```

세 영상에서 YOLO Pose가 관절을 정상적으로 추출하는지 확인한 뒤 전체 영상으로 확대합니다.

```
대표 영상 3개
      ↓
YOLO Pose로 Keypoint 추출
      ↓
JSON 저장 결과 확인
      ↓
Skeleton 영상으로 관절 위치 확인
      ↓
정상이라면 Main 전체 48개 처리
```

이렇게 진행하면 **관절 추출 과정에 문제가 있을 때 전체 영상을 처리하기 전에 발견할 수 있습니다.**

---

# 13. Video → Keypoint JSON 코드를 작성합니다

이번 코드는 **KTH 동작 영상에서 사람의 17개 관절 좌표를 추출하고, 시간순으로 JSON 파일에 
저장하는 프로그램**입니다.

교과 10에서 배운 YOLO Pose를 다시 활용하며, 이렇게 만든 원본 Keypoint Sequence는 이후 합성데이터 보강 실습의 입력으로 사용합니다.

### 코드의 전체 처리 흐름

```
KTH 영상(.avi) 열기
        ↓
Frame을 하나씩 읽기
        ↓
YOLO Pose로 사람의 관절 추출
        ↓
사람이 검출되었는가?
   ├─ 예 → 신뢰도가 가장 높은 사람의
   │       17개 Keypoint 선택
   └─ 아니요 → 17개 관절을 0값으로 기록
        ↓
관절 좌표를 0~1 범위로 정규화
        ↓
Frame별 Keypoint를 시간순으로 저장
        ↓
영상이 끝날 때까지 반복
        ↓
Keypoint Sequence를 JSON으로 저장
```

이 과정에서 **사람이 검출되지 않은 Frame도 기록**하여 원본 영상의 시간 흐름을 유지합니다.

이 파일을 작성한 후 12번에서 선택한 대표 영상 3개를 먼저 실행하고, 추출 결과가 정상인지 확인하겠습니다.

**파일: `scripts/01_extract_keypoints.py`**

```python
from __future__ import annotations

import argparse
import json
import sys
from pathlib import Path

import cv2
import numpy as np
from ultralytics import YOLO


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (
    list_videos,
    original_keypoint_dir,
    parse_video_name,
)


MODEL_PATH = (
    ROOT
    / "models"
    / "yolo11n-pose.pt"
)

COCO17_NAMES = [
    "nose",
    "left_eye",
    "right_eye",
    "left_ear",
    "right_ear",
    "left_shoulder",
    "right_shoulder",
    "left_elbow",
    "right_elbow",
    "left_wrist",
    "right_wrist",
    "left_hip",
    "right_hip",
    "left_knee",
    "right_knee",
    "left_ankle",
    "right_ankle",
]


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        default="main",
        choices=["main", "challenge"],
    )

    parser.add_argument(
        "--split",
        default="all",
        choices=["train", "val", "test", "all"],
    )

    parser.add_argument(
        "--demo",
        action="store_true",
    )

    parser.add_argument(
        "--conf",
        type=float,
        default=0.15,
    )

    parser.add_argument(
        "--imgsz",
        type=int,
        default=640,
    )

    parser.add_argument(
        "--overwrite",
        action="store_true",
    )

    return parser.parse_args()


def choose_person(result) -> int | None:
    if (
        result.keypoints is None
        or result.keypoints.xy is None
        or len(result.keypoints.xy) == 0
    ):
        return None

    if (
        result.boxes is not None
        and result.boxes.conf is not None
        and len(result.boxes.conf) > 0
    ):
        return int(
            result.boxes.conf.argmax().item()
        )

    return 0


def demo_videos() -> list[Path]:
    names = [
        "S11_boxing_T01.avi",
        "S11_handwaving_T01.avi",
        "S11_handclapping_T01.avi",
    ]

    train_videos = {
        path.name: path
        for path in list_videos(
            "main",
            "train",
        )
    }

    missing = [
        name
        for name in names
        if name not in train_videos
    ]

    if missing:
        raise RuntimeError(
            "Demo 영상이 없습니다: "
            + ", ".join(missing)
        )

    return [
        train_videos[name]
        for name in names
    ]


def extract_video(
    model,
    video_path: Path,
    output_path: Path,
    conf: float,
    imgsz: int,
) -> None:
    capture = cv2.VideoCapture(
        str(video_path)
    )

    if not capture.isOpened():
        raise RuntimeError(
            f"영상을 열 수 없습니다: {video_path}"
        )

    fps = float(
        capture.get(cv2.CAP_PROP_FPS)
    )

    width = int(
        capture.get(cv2.CAP_PROP_FRAME_WIDTH)
    )

    height = int(
        capture.get(cv2.CAP_PROP_FRAME_HEIGHT)
    )

    info = parse_video_name(
        video_path
    )

    frames = []
    frame_index = 0
    detected_frames = 0
    confidence_values = []

    while True:
        success, frame = capture.read()

        if not success:
            break

        result = model.predict(
            source=frame,
            conf=conf,
            imgsz=imgsz,
            verbose=False,
        )[0]

        person_index = choose_person(
            result
        )

        if person_index is None:
            keypoints = [
                [0.0, 0.0, 0.0]
                for _ in range(17)
            ]

            detected = False

        else:
            xy = (
                result.keypoints.xy[
                    person_index
                ]
                .detach()
                .cpu()
                .numpy()
            )

            if len(xy) != 17:
                raise RuntimeError(
                    "COCO17 Keypoint 수가 "
                    f"17이 아닙니다: {len(xy)}"
                )

            if (
                result.keypoints.conf
                is not None
            ):
                scores = (
                    result.keypoints.conf[
                        person_index
                    ]
                    .detach()
                    .cpu()
                    .numpy()
                )
            else:
                scores = np.ones(
                    17,
                    dtype=np.float32,
                )

            keypoints = []

            for joint_index in range(17):
                x = float(
                    xy[joint_index, 0]
                )

                y = float(
                    xy[joint_index, 1]
                )

                score = float(
                    scores[joint_index]
                )

                nx = (
                    x / width
                    if width > 0
                    else 0.0
                )

                ny = (
                    y / height
                    if height > 0
                    else 0.0
                )

                keypoints.append(
                    [
                        round(nx, 8),
                        round(ny, 8),
                        round(score, 6),
                    ]
                )

                if score > 0:
                    confidence_values.append(
                        score
                    )

            detected = True
            detected_frames += 1

        frames.append(
            {
                "frame_index": frame_index,
                "detected": detected,
                "keypoints": keypoints,
            }
        )

        frame_index += 1

    capture.release()

    if frame_index == 0:
        raise RuntimeError(
            f"Frame이 없습니다: {video_path.name}"
        )

    mean_conf = (
        float(
            np.mean(confidence_values)
        )
        if confidence_values
        else 0.0
    )

    payload = {
        "source_video": video_path.name,
        "subject": info["subject"],
        "action": info["action"],
        "take": info["take"],
        "fps": fps,
        "width": width,
        "height": height,
        "total_frames": frame_index,
        "detected_frames": detected_frames,
        "detection_rate": (
            detected_frames / frame_index
        ),
        "mean_keypoint_confidence": mean_conf,
        "keypoint_format": (
            "COCO17_normalized_xy_conf"
        ),
        "joint_names": COCO17_NAMES,
        "frames": frames,
    }

    output_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    output_path.write_text(
        json.dumps(
            payload,
            ensure_ascii=False,
            indent=2,
        ),
        encoding="utf-8",
    )

    print(
        video_path.name,
        "→",
        output_path.name,
        f"Detection={payload['detection_rate']:.3f}",
    )


def main() -> None:
    args = parse_args()

    if not MODEL_PATH.exists():
        raise FileNotFoundError(
            "YOLO Pose 모델이 없습니다:\n"
            f"{MODEL_PATH}"
        )

    if args.demo:
        if (
            args.asset_set != "main"
            or args.split not in {"all", "train"}
        ):
            raise ValueError(
                "--demo는 main/train에서 사용합니다."
            )

        work_items = [
            (
                path,
                original_keypoint_dir(
                    "main",
                    "train",
                ),
            )
            for path in demo_videos()
        ]

    else:
        splits = (
            ["train", "val", "test"]
            if args.split == "all"
            else [args.split]
        )

        work_items = []

        for split in splits:
            output_dir = (
                original_keypoint_dir(
                    args.asset_set,
                    split,
                )
            )

            for path in list_videos(
                args.asset_set,
                split,
            ):
                work_items.append(
                    (
                        path,
                        output_dir,
                    )
                )

    model = YOLO(
        str(MODEL_PATH)
    )

    for video_path, output_dir in work_items:
        output_path = (
            output_dir
            / f"{video_path.stem}.json"
        )

        if (
            output_path.exists()
            and not args.overwrite
        ):
            print(
                "[SKIP]",
                output_path.name,
            )
            continue

        extract_video(
            model=model,
            video_path=video_path,
            output_path=output_path,
            conf=args.conf,
            imgsz=args.imgsz,
        )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[프로그램 시작]
        ↓
① 필요한 라이브러리와 공통 함수 불러오기
   ├─ OpenCV → AVI 영상 읽기
   ├─ NumPy → 신뢰도 평균 계산
   ├─ YOLO → 사람의 관절 좌표 추출
   └─ day05_common.py → 영상 검색·경로·파일명 관리
        ↓
② YOLO Pose 모델 경로와 COCO17 관절 이름 설정
   │
   └─ models/yolo11n-pose.pt
        ↓
③ 실행 옵션 확인 — parse_args()
   │
   ├─ --asset-set → main / challenge
   ├─ --split     → train / val / test / all
   ├─ --demo      → 대표 영상 3개만 처리
   ├─ --conf      → YOLO 검출 신뢰도 기준 (기본 0.15)
   ├─ --imgsz     → YOLO 입력 이미지 크기 (기본 640)
   └─ --overwrite → 기존 JSON 덮어쓰기
        ↓
④ 모델 파일이 존재하는가? — main()
   │
   ├─ 아니요 → FileNotFoundError 발생 및 중단
   └─ 예 → 다음 단계
        ↓
⑤ 처리할 영상 선택
   │
   ├─ --demo 사용
   │     ↓
   │   demo_videos()
   │     ↓
   │   Main Train의 대표 영상 3개 검색
   │     ├─ S11_boxing_T01.avi
   │     ├─ S11_handwaving_T01.avi
   │     └─ S11_handclapping_T01.avi
   │     ↓
   │   영상이 없으면 오류 발생
   │
   └─ --demo 미사용
         ↓
       선택한 Main 또는 Challenge에서
       Train / Val / Test 영상 목록 가져오기
        ↓
⑥ YOLO Pose 모델 불러오기
   │
   └─ YOLO("models/yolo11n-pose.pt")
        ↓
⑦ 영상별 Keypoint 추출 시작
   │
   ├─ 저장할 JSON이 이미 존재하는가?
   │     ├─ 예 → [SKIP] (기본 동작)
   │     │        단, --overwrite 사용 시 다시 처리
   │     └─ 아니요 → 추출 진행
   │
   └─ extract_video()
         ↓
⑧ OpenCV로 AVI 영상 열기
   │
   ├─ 열 수 없음 → 오류 발생 및 중단
   └─ 정상 → FPS / Width / Height 확인
         ↓
⑨ 영상 파일명 분석 — parse_video_name()
   │
   │  예: S11_boxing_T01.avi
   │
   ├─ Subject → S11
   ├─ Action  → boxing
   └─ Take    → T01
         ↓
⑩ 영상에서 Frame 하나 읽기
   │
   ├─ 읽기 실패 또는 영상 끝 → ⑯으로 이동
   └─ 읽기 성공 → YOLO Pose 실행
         ↓
⑪ 사람이 검출되었는가? — choose_person()
   │
   ├─ 아니요
   │     ↓
   │   17개 Keypoint를 [0, 0, 0]으로 기록
   │     ↓
   │   detected = False
   │
   └─ 예
         ↓
       검출된 사람 중
       Bounding Box 신뢰도가 가장 높은 사람 선택
         ↓
       COCO17 Keypoint 추출
         ↓
       Keypoint가 정확히 17개인가?
         ├─ 아니요 → 오류 발생 및 중단
         └─ 예 → 다음 단계
         ↓
⑫ 각 관절의 좌표와 신뢰도 확인
   │
   ├─ x → 관절의 가로 위치(Pixel)
   ├─ y → 관절의 세로 위치(Pixel)
   └─ confidence → 관절 예측 신뢰도
         ↓
⑬ 관절 좌표 정규화
   │
   ├─ nx = x / width
   └─ ny = y / height
         ↓
   [nx, ny, confidence] 형태로 저장
         ↓
   detected = True
   검출 Frame 수 증가
         ↓
⑭ 현재 Frame의 결과를 목록에 추가
   │
   ├─ frame_index → Frame 번호
   ├─ detected    → 사람 검출 여부
   └─ keypoints   → 17개 관절 좌표
         ↓
⑮ 다음 Frame으로 이동
   │
   └─ 영상이 끝날 때까지 ⑩~⑮ 반복
         ↓
⑯ 영상 처리 완료
   │
   ├─ OpenCV 영상 닫기
   ├─ 총 Frame 수 확인
   ├─ 사람이 검출된 Frame 수 계산
   ├─ Detection Rate 계산
   │     ↓
   │   검출 Frame 수 / 전체 Frame 수
   │
   └─ 관절 Confidence 평균 계산
         ↓
⑰ Keypoint Sequence JSON 구성
   │
   ├─ 원본 영상 이름
   ├─ Subject / Action / Take
   ├─ FPS / Width / Height
   ├─ 전체 Frame 수
   ├─ 검출된 Frame 수
   ├─ Detection Rate
   ├─ 평균 Keypoint Confidence
   ├─ COCO17 관절 이름
   └─ Frame별 17개 Keypoint
         ↓
⑱ JSON 파일 저장
   │
   └─ data/keypoints/{main|challenge}/original/{train|val|test}/
         ↓
⑲ 다음 영상이 있는가?
   │
   ├─ 예 → ⑦부터 반복
   └─ 아니요 → 모든 영상 처리 완료
         ↓
[프로그램 종료]
```

# 14. 대표 3개 영상으로 먼저 실행합니다

```bash
python scripts/01_extract_keypoints.py \
  --asset-set main \
  --split train \
  --demo
```

대표 영상 3개를 처리하면 다음과 같은 JSON 파일이 생성됩니다.

```text
data/keypoints/main/original/train/
├─ S11_boxing_T01.json
├─ S11_handwaving_T01.json
└─ S11_handclapping_T01.json
```

JSON 일부를 확인합니다.

```bash
head -50 \
  data/keypoints/main/original/train/S11_boxing_T01.json
```

### JSON 추출 결과를 확인하고 해석합니다

YOLO Pose로 추출한 `S11_boxing_T01.json`을 열어보면 원본 영상 정보와 Frame별 관절 좌표가 저장되어 있습니다.

**① 영상 정보 확인**

```json
{
  "source_video": "S11_boxing_T01.avi",
  "subject": "S11",
  "action": "boxing",
  "take": "T01",
  "fps": 25.0,
  "width": 160,
  "height": 120,
  "total_frames": 351,
  "detected_frames": 351,
  "detection_rate": 1.0,
  "mean_keypoint_confidence": 0.8407
}
```

이 결과는 다음을 의미합니다.

- `total_frames: 351` → 총 351장의 Frame을 처리했습니다.
- `fps: 25.0` → 1초에 25장의 Frame이 있는 약 14초 영상입니다.
- `detected_frames: 351` → 모든 Frame에서 사람이 검출되었습니다.
- `detection_rate: 1.0` → 사람 검출률이 100%입니다.
- `mean_keypoint_confidence: 0.8407` → 추출된 관절의 평균 신뢰도는 약 84.1%입니다.
    

**② Frame별 관절 좌표 확인**

```json
{
  "frame_index": 0,
  "detected": true,
  "keypoints": [
    [0.42917724, 0.25337435, 0.921102]
  ]
}
```

위 코드는 첫 번째 Frame(`frame_index: 0`)에서 사람이 검출되었으며, 첫 번째 Keypoint인 코의 위치가 기록된 예시입니다.

세 숫자는 순서대로 정규화된 x좌표, y좌표, 관절 신뢰도를 의미합니다. 실제 JSON에는 이와 같은 좌표가 Frame마다 17개씩 저장됩니다.

**③ 대표 영상 3개의 결과 비교**

|동작 영상|Frame 수|사람 검출률|평균 관절 신뢰도|
|---|---|---|---|
|boxing|351|100%|84.1%|
|handwaving|475|100%|95.4%|
|handclapping|320|100%|96.1%|

세 영상 모두 모든 Frame에서 사람이 검출되었습니다. 다만 `boxing`의 평균 관절 신뢰도가 상대적으로 낮으므로, 일부 관절의 위치가 불안정하지 않은지 확인할 필요가 있습니다.

**④ 이번 결과에서 기억할 핵심**

사람 검출률이 100%라고 해서 **17개 관절이 모두 정확하게 추출되었다는 뜻은 아닙니다.**

지금은 원본 영상이 Keypoint Sequence JSON으로 변환되었는지 확인하는 단계입니다.

다음 실습에서는 추출한 관절 좌표를 실제 영상 위에 Skeleton으로 그려보고, 사람의 움직임을 정상적으로 따라가는지 직접 확인하겠습니다.

---
# 15. 추출한 Keypoint를 영상 위에 그려서 확인합니다

14번에서는 YOLO Pose로 추출한 관절 좌표가 JSON 파일에 정상적으로 저장되었는지 확인했습니다.

하지만 **숫자만으로는 관절이 실제 사람의 움직임을 정확하게 따라가는지 판단하기 어렵습니다.**

이번에는 저장된 Keypoint를 다시 불러와 **원본 영상 위에 관절 점과 연결선(Skeleton)을 그려서 확인**하겠습니다.

### 이번 코드는 무엇을 하나요?

```
원본 KTH 영상 (.avi)
          +
추출한 Keypoint (.json)
          ↓
Frame별 관절 좌표 불러오기
          ↓
17개 관절을 점으로 표시
          ↓
관절 사이를 선으로 연결
          ↓
Skeleton 영상 (.mp4) 저장
          ↓
실제 움직임과 관절 위치 비교
```

이렇게 만든 영상을 재생하면서 **손목과 팔꿈치가 동작을 제대로 따라가는지, 관절이 갑자기 사라지거나 잘못된 위치에 표시되지 않는지** 확인합니다.

또한 이후 실습에서 생성할 Augmented Sequence는 원본 영상과 Frame 수가 달라질 수 있습니다. 따라서 원본 영상 없이도 관절 움직임을 확인할 수 있도록, `--video`를 생략하면 빈 배경에 Skeleton만 그리는 기능도 함께 작성합니다.

이번 실습의 목적은 새로운 데이터를 생성하는 것이 아니라, **앞에서 추출한 Keypoint의 품질을 눈으로 확인하여 이후 데이터 보강에 사용할 수 있는지 판단하는 것**입니다.

**파일: `scripts/02_visualize_keypoints.py`**

```python
from __future__ import annotations

import argparse
import json
from pathlib import Path

import cv2
import numpy as np


SKELETON = [
    (5, 6),
    (5, 7),
    (7, 9),
    (6, 8),
    (8, 10),
    (5, 11),
    (6, 12),
    (11, 12),
    (11, 13),
    (13, 15),
    (12, 14),
    (14, 16),
]


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--json",
        required=True,
    )

    parser.add_argument(
        "--video",
        default="",
    )

    parser.add_argument(
        "--output",
        required=True,
    )

    parser.add_argument(
        "--confidence",
        type=float,
        default=0.15,
    )

    parser.add_argument(
        "--width",
        type=int,
        default=640,
    )

    parser.add_argument(
        "--height",
        type=int,
        default=480,
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    data = json.loads(
        Path(args.json).read_text(
            encoding="utf-8"
        )
    )

    capture = None

    if args.video:
        capture = cv2.VideoCapture(
            args.video
        )

        if not capture.isOpened():
            raise RuntimeError(
                "원본 영상을 열 수 없습니다."
            )

        width = int(
            capture.get(
                cv2.CAP_PROP_FRAME_WIDTH
            )
        )

        height = int(
            capture.get(
                cv2.CAP_PROP_FRAME_HEIGHT
            )
        )

        fps = float(
            capture.get(
                cv2.CAP_PROP_FPS
            )
        )

    else:
        width = args.width
        height = args.height

        fps = float(
            data.get(
                "fps",
                25.0,
            )
        )

        if fps <= 0:
            fps = 25.0

    output_path = Path(
        args.output
    )

    output_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    writer = cv2.VideoWriter(
        str(output_path),
        cv2.VideoWriter_fourcc(
            *"mp4v"
        ),
        fps,
        (width, height),
    )

    if not writer.isOpened():
        raise RuntimeError(
            "출력 VideoWriter를 열 수 없습니다."
        )

    for frame_index, item in enumerate(
        data["frames"]
    ):
        if capture is not None:
            success, frame = capture.read()

            if not success:
                break
        else:
            frame = np.full(
                (
                    height,
                    width,
                    3,
                ),
                32,
                dtype=np.uint8,
            )

        points = []

        for x, y, score in item["keypoints"]:
            px = int(x * width)
            py = int(y * height)

            points.append(
                (
                    px,
                    py,
                    score,
                )
            )

        if len(points) != 17:
            raise RuntimeError(
                "Keypoint 수가 17이 아닙니다."
            )

        for start, end in SKELETON:
            x1, y1, s1 = points[start]
            x2, y2, s2 = points[end]

            if (
                s1 >= args.confidence
                and s2 >= args.confidence
            ):
                cv2.line(
                    frame,
                    (x1, y1),
                    (x2, y2),
                    (255, 255, 255),
                    2,
                )

        for x, y, score in points:
            if score >= args.confidence:
                cv2.circle(
                    frame,
                    (x, y),
                    3,
                    (255, 255, 255),
                    -1,
                )

        cv2.putText(
            frame,
            f"{data['action']} frame={frame_index}",
            (12, 24),
            cv2.FONT_HERSHEY_SIMPLEX,
            0.55,
            (255, 255, 255),
            1,
            cv2.LINE_AA,
        )

        writer.write(frame)

    if capture is not None:
        capture.release()

    writer.release()

    print(
        "Saved:",
        output_path,
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[프로그램 시작]
        ↓
① 필요한 라이브러리 불러오기
   ├─ argparse → 실행 옵션 확인
   ├─ json     → Keypoint JSON 읽기
   ├─ pathlib → 파일 경로 관리
   ├─ OpenCV   → 영상 읽기·그리기·저장
   └─ NumPy    → 빈 배경 이미지 생성
        ↓
② Skeleton 연결 규칙 정의
   │
   ├─ 양쪽 어깨 연결
   ├─ 어깨 → 팔꿈치 → 손목
   ├─ 어깨 → 엉덩이
   ├─ 양쪽 엉덩이 연결
   └─ 엉덩이 → 무릎 → 발목
        ↓
③ 실행 옵션 확인 — parse_args()
   │
   ├─ --json       → 입력 JSON 경로 (필수)
   ├─ --video      → 원본 영상 경로 (선택)
   ├─ --output     → 출력 MP4 경로 (필수)
   ├─ --confidence → 관절 표시 기준 (기본 0.15)
   ├─ --width      → 빈 배경 가로 크기 (기본 640)
   └─ --height     → 빈 배경 세로 크기 (기본 480)
        ↓
④ Keypoint JSON 파일 읽기
   │
   └─ Frame별 COCO17 관절 좌표 불러오기
        ↓
⑤ 원본 영상(--video)을 지정했는가?
   │
   ├─ 예 → 원본 영상 위에 Skeleton 표시
   │    │
   │    ├─ OpenCV로 원본 AVI 열기
   │    ├─ 영상이 열리지 않으면 오류 발생
   │    └─ 영상의 Width / Height / FPS 가져오기
   │
   └─ 아니요 → 빈 배경에 Skeleton만 표시
        │
        ├─ 지정된 Width / Height 사용
        └─ JSON의 FPS 사용
            (FPS가 0 이하이면 25 사용)
        ↓
⑥ 출력 MP4 파일 준비
   │
   ├─ 출력 폴더가 없으면 생성
   ├─ MP4 VideoWriter 생성
   └─ 생성 실패 시 오류 발생
        ↓
⑦ JSON의 Frame을 하나씩 처리
   │
   ├─ 원본 영상을 사용하는 경우
   │     ↓
   │   AVI에서 해당 순서의 Frame 읽기
   │     ↓
   │   원본 영상이 끝나면 반복 종료
   │
   └─ 원본 영상을 사용하지 않는 경우
         ↓
       어두운 빈 배경(Canvas) 생성
        ↓
⑧ 현재 Frame의 Keypoint 가져오기
   │
   └─ 관절별 [x, y, confidence] 읽기
        ↓
⑨ 정규화 좌표를 Pixel 좌표로 변환
   │
   ├─ px = x × 영상 가로 크기
   └─ py = y × 영상 세로 크기
        ↓
⑩ 관절이 정확히 17개인가?
   │
   ├─ 아니요 → 오류 발생
   └─ 예 → 다음 단계
        ↓
⑪ 관절 사이에 Skeleton 연결선 그리기
   │
   ├─ SKELETON에 정의된 관절 쌍 확인
   │
   ├─ 연결할 두 관절의 신뢰도가
   │   모두 0.15 이상인가? (기본 설정)
   │
   │   ├─ 예 → 흰색 선으로 연결
   │   └─ 아니요 → 연결선 표시하지 않음
   │
   └─ 정의된 모든 관절 쌍에 반복 적용
        ↓
⑫ Keypoint 위치를 점으로 표시
   │
   ├─ 신뢰도 ≥ 기준값 → 흰색 점 표시
   └─ 신뢰도 < 기준값 → 점 표시하지 않음
        ↓
⑬ 화면에 동작 이름과 Frame 번호 표시
   │
   └─ 예: boxing frame=0
        ↓
⑭ 현재 Frame을 MP4 파일에 기록
   │
   └─ writer.write(frame)
        ↓
⑮ 처리할 JSON Frame이 더 있는가?
   │
   ├─ 예 → ⑦로 돌아가 반복
   └─ 아니요 → 반복 종료
        ↓
⑯ 사용한 영상과 출력 파일 닫기
   │
   ├─ 원본 영상 사용 시 capture.release()
   └─ writer.release()
        ↓
⑰ 저장 완료 메시지 출력
   │
   └─ Saved: 출력파일경로.mp4
        ↓
[프로그램 종료]
```

실행합니다.

```bash
python scripts/02_visualize_keypoints.py \
  --json data/keypoints/main/original/train/S11_boxing_T01.json \
  --video data/day05_student_data/main/train/S11_boxing_T01.avi \
  --output results/main/S11_boxing_T01_pose.mp4
```

터미널 결과 확인

```
  --json data/keypoints/main/original/train/S11_boxing_T01.json \
  --video data/day05_student_data/main/train/S11_boxing_T01.avi \
  --output results/main/S11_boxing_T01_pose.mp4
Saved: results/main/S11_boxing_T01_pose.mp4
```

탐색기를 열어서 직접 영상을 확인합니다.

```bash
explorer.exe .
```


다음 항목을 확인합니다.

```text
사람을 따라가는가?

손목·팔꿈치가 동작을 따라가는가?

사람이 작아졌을 때 검출이 흔들리는가?

일부 Frame에서 관절이 사라지는가?
```

---
# 16. Main 영상 48개 전체에서 Keypoint를 추출합니다

15번에서는 대표 영상 3개의 Skeleton을 확인했습니다.

이제 같은 YOLO Pose 코드를 사용하여 **Main 데이터셋의 전체 48개 영상을 Keypoint JSON으로 변환**하겠습니다.

### ① 전체 영상 처리하기

```
python scripts/01_extract_keypoints.py \
  --asset-set main \
  --split all
```

- `--asset-set main` : Main 데이터셋을 사용합니다.
    
- `--split all` : Train, Validation, Test 영상을 모두 처리합니다.
    

### ② 실행 결과 확인하기

실제 실행 결과에서 다음과 같은 메시지를 확인했습니다.

```
[SKIP] S11_boxing_T01.json
S11_boxing_T02.avi → S11_boxing_T02.json Detection=1.000

[SKIP] S11_handclapping_T01.json
[SKIP] S11_handwaving_T01.json

S13_handwaving_T02.avi → S13_handwaving_T02.json Detection=0.996
```

각 메시지의 의미는 다음과 같습니다.

- `**[SKIP]**` : 대표 영상 3개는 앞에서 이미 JSON을 만들었으므로 다시 처리하지 않았습니다.
    
- `**Detection=1.000**` : 해당 영상의 모든 Frame에서 사람이 검출되었습니다. (100%)
    
- `**Detection=0.996**` : 해당 영상의 약 99.6% Frame에서 사람이 검출되었습니다.
    

대부분 영상의 검출률이 100%였고, `S13_handwaving_T02.avi`에서는 일부 Frame에서 사람이 검출되지 않았습니다.

이것은 프로그램 오류를 의미하지 않습니다. 다만 **검출되지 않은 Frame이나 관절 위치에 문제가 없는지는 이후 QA 과정에서 확인**해야 합니다.

### ③ 최종 JSON 파일 확인하기

이번 실행에서는 기존 3개 JSON을 재사용하고, 나머지 45개를 새로 생성했습니다.

따라서 Main 데이터셋에는 총 48개 JSON이 준비됩니다.

```
data/keypoints/main/original/
│
├── train/ → 24개 JSON
│
├── val/   → 12개 JSON
│
└── test/  → 12개 JSON

총 48개 JSON
```

실제로 파일 개수가 맞는지 확인해 보겠습니다.

```
find data/keypoints/main/original/train -name "*.json" | wc -l

find data/keypoints/main/original/val -name "*.json" | wc -l

find data/keypoints/main/original/test -name "*.json" | wc -l
```

예상 결과는 순서대로 `24`, `12`, `12`입니다.

### ④ 이번 실습의 핵심 정리

```
KTH 원본 영상 48개
        ↓
YOLO Pose
        ↓
Frame별 COCO17 Keypoint 추출
        ↓
Keypoint Sequence JSON 저장
        ↓
Train 24개 / Val 12개 / Test 12개
        ↓
원본 Keypoint 데이터 준비 완료
        ↓
다음 단계: Sequence Augmentation
```

**핵심:** 이제 48개 영상의 관절 움직임이 숫자 데이터로 저장되었습니다.

---
# PART 3. 원본 Keypoint를 보강하여 새로운 학습데이터를 만듭니다

---

앞의 PART 2에서는 KTH 영상 48개를 YOLO Pose로 처리하여 **원본 Keypoint Sequence JSON 48개**를 만들었습니다.

이번 PART 3에서는 그중 **Train 데이터 24개에만** 변형을 적용하여 새로운 학습데이터를 생성하겠습니다.

---

# 17. 왜 Train 데이터만 보강할까요?

데이터 보강(Augmentation)은 기존 학습 데이터를 조금씩 변형하여 모델이 다양한 상황을 학습하도록 만드는 방법입니다.

예를 들어 복싱 동작의 Keypoint 좌표를 조금 이동하거나 노이즈를 추가하여 새로운 학습 데이터를 만듭니다.

현재 원본 Keypoint JSON 48개를 다음과 같이 나누었습니다.

```
       원본 Keypoint JSON 48개
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
    Train 24   Val 12    Test 12
       ↓         ↓         ↓
   데이터 보강  원본 유지   원본 유지
       ↓         ↓         ↓
    모델 학습   모델 검증   최종 평가
```

각 데이터의 역할은 다음과 같습니다.

- Train(학습): 모델이 공부하는 데이터이므로 다양한 연습을 위해 보강합니다.
    
- Validation(검증): 학습 중 모델이 얼마나 잘하는지 확인하므로 원본을 유지합니다.
    
- Test(평가): 학습이 끝난 모델의 최종 실력을 확인하므로 원본을 유지합니다.
    

왜 이렇게 할까요?

시험공부를 할 때는 다양한 연습문제를 풀어보는 것이 좋지만, 시험에서는 처음 보는 문제를 풀어야 실제 실력을 알 수 있기 때문입니다.

따라서 먼저 원본 데이터를 Train, Validation, Test로 나눈 후 Train만 보강합니다.

이렇게 하면 원본만 학습한 모델과 보강 데이터를 추가로 학습한 모델을 동일한 시험 데이터로 
비교하여, 데이터 보강이 실제로 성능 향상에 도움이 되었는지 확인할 수 있습니다.

즉, 교과 11에서 합성데이터를 만드는 목적은 Train(학습) 데이터를 보강하여 모델이 더 다양한 상황을 학습하도록 하는 것입니다.

---

# 18. 어떤 방법으로 Keypoint를 보강할까요?

이번에는 원본 관절 좌표에 다섯 가지 변형을 적용합니다.

| 보강 방법 | 무엇이 달라지나요? |
|---|---|
| Translation | 사람의 관절 위치를 전체적으로 조금 이동합니다. |
| Scale | 사람의 중심을 기준으로 관절 사이의 거리를 늘리거나 줄입니다. |
| Jitter | 관절 좌표에 작은 무작위 오차를 추가합니다. |
| Time Warp Fast | Frame 수를 줄여 움직임을 빠르게 만듭니다. |
| Time Warp Slow | Frame 수를 늘려 움직임을 느리게 만듭니다. |

이번 실습에서 사용할 기본 설정은 다음과 같습니다.

```text
Translation    → dx, dy 약 ±0.04 이내
Scale          → 0.90 ~ 1.10
Jitter         → sigma = 0.005
Time Warp Fast → 0.85
Time Warp Slow → 1.15
```

이 값들은 실습을 위한 설정이며, **동작이 지나치게 변형되지 않는지 이후 QA 과정에서 확인**합니다.

### 관절 좌표만 조금 바꾸는 것이 정말 AI 모델의 성능 향상에 도움이 되는가?

결론부터 말씀드리면 도움이 될 수 있습니다. 다만 어떤 모델을 학습하느냐에 따라 가치가 달라집니다.

#### 1. 실제로 무엇이 달라지는 걸까요?

예를 들어 한 사람이 손을 흔드는 영상이 있다고 생각해 보겠습니다.

![[Pasted image 20261016143506.png|153]]

원본 손목 좌표가 다음과 같다고 가정하겠습니다.

```
원본 손목 위치
x = 0.50
y = 0.30

Translation 적용 (오른쪽으로 0.04 이동)

변형된 손목 위치
x = 0.54
y = 0.30
```

영상 속 사람은 그대로입니다. 하지만 JSON에 저장된 손목 좌표가 달라졌습니다.

Scale을 적용하면 관절 사이의 거리가 달라지고, Jitter는 좌표에 미세한 오차를 추가하며, Time Warp는 움직임의 시간적 진행 속도를 바꿉니다.

즉, 실제 영상을 변형하는 것이 아니라 같은 동작의 다양한 Skeleton 패턴을 인위적으로 만드는 것입니다.

#### 그런데 왜 이것이 학습에 도움이 될까요?

핵심은 교과 10에서 어떤 데이터를 이용해 동작을 분류했는지에 있습니다.

![[Pasted image 20261016143653.png]]

여기서 Random Forest는 실제 영상의 색상이나 배경을 직접 보는 것이 아닙니다.

관절 좌표에서 계산한 특징을 보고 동작을 분류합니다.

그렇다면 사람의 키가 조금 다르거나, 카메라 속 위치가 달라지거나, 같은 손 흔들기라도 속도가 달라졌을 때 어떻게 될까요?

학습데이터가 다양하지 않다면 처음 보는 움직임에서 잘못 판단할 수 있습니다.

이때 Keypoint Augmentation으로 다양한 움직임 패턴을 추가하면, 모델이 원본 데이터의 일부 특성에만 지나치게 의존하지 않도록 학습하는 데 도움이 될 수 있습니다.

#### 다섯 가지 보강 방법의 실제 가치

| 방법             | 모델에 학습시키려는 변화                     |
| -------------- | --------------------------------- |
| Translation    | 화면에서 사람이 조금 다른 위치에 있어도 같은 동작임을 학습 |
| Scale          | 화면 속 사람의 크기가 조금 달라져도 동작을 인식하도록 학습 |
| Jitter         | 관절 검출에 작은 오차가 있어도 안정적으로 인식하도록 학습  |
| Time Warp Fast | 같은 동작을 조금 빠르게 수행하는 경우를 학습         |
| Time Warp Slow | 같은 동작을 조금 느리게 수행하는 경우를 학습         |

### 이번 코드는 무엇을 하나요?

앞에서 저장한 **Train의 원본 Keypoint JSON 24개를 읽어 다섯 가지 방법으로 변형하고, 새로운 JSON 파일로 저장하는 프로그램**입니다.

원본 데이터는 그대로 보존하며, 새로 생성한 데이터에는 어떤 보강 방법을 사용했는지 Metadata도 기록합니다.

### 코드의 전체 처리 흐름

```text
Main Train 원본 Keypoint JSON 24개
                ↓
       원본 Sequence 읽기
                ↓
       다섯 가지 보강 적용
                ↓
     ├─ Translation
     ├─ Scale
     ├─ Jitter
     ├─ Time Warp Fast
     └─ Time Warp Slow
                ↓
       보강된 Sequence 생성
                ↓
       Frame 수와 검출률 재계산
                ↓
       보강 방법·설정값 기록
                ↓
       새로운 JSON 파일 저장
                ↓
       24개 × 5가지 = 120개
```

특히 Time Warp는 Frame 수가 달라지므로 `total_frames`, `detected_frames`, `detection_rate`를 새 Sequence에 맞게 다시 계산합니다.

**핵심:** 이번 실습은 새로운 영상을 촬영하는 것이 아니라, **기존 관절 움직임을 조금씩 변형하여 
다양한 학습용 Keypoint Sequence를 만드는 과정**입니다.

생성한 120개 데이터는 바로 학습에 사용하지 않고, 다음 PART에서 QA를 진행하여 
동작이 자연스럽게 유지되었는지 확인합니다.

---

# 19. Keypoint Sequence Augmentation 코드를 작성합니다

### 이번 코드는 무엇을 하나요?

앞에서 저장한 **Train의 원본 Keypoint JSON 24개를 읽어 다섯 가지 방법으로 변형하고, 
새로운 JSON 파일로 저장하는 프로그램**입니다.

원본 데이터는 그대로 보존하며, 새로 생성한 데이터에는 어떤 보강 방법을 사용했는지 
Metadata도 기록합니다.

### 코드의 전체 처리 흐름

```
Main Train 원본 Keypoint JSON 24개
                ↓
       원본 Sequence 읽기
                ↓
       다섯 가지 보강 적용
                ↓
     ├─ Translation
     ├─ Scale
     ├─ Jitter
     ├─ Time Warp Fast
     └─ Time Warp Slow
                ↓
       보강된 Sequence 생성
                ↓
       Frame 수와 검출률 재계산
                ↓
       보강 방법·설정값 기록
                ↓
       새로운 JSON 파일 저장
                ↓
       24개 × 5가지 = 120개
```

특히 Time Warp는 Frame 수가 달라지므로 
`total_frames`, `detected_frames`, `detection_rate`를 새 Sequence에 맞게 다시 계산합니다.


**작성할 파일:** `scripts/03_augment_keypoints.py`

```python
from __future__ import annotations

import argparse
import json
import random
import shutil
import sys
from pathlib import Path

import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (
    augmented_keypoint_dir,
    original_keypoint_dir,
    report_dir,
)


BASE_SEED = 42

SUPPORTED_METHODS = [
    "translate",
    "scale",
    "jitter",
    "timewarp_fast",
    "timewarp_slow",
]


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        required=True,
        choices=[
            "main",
            "challenge",
        ],
    )

    parser.add_argument(
        "--tag",
        required=True,
    )

    parser.add_argument(
        "--translate-max",
        type=float,
        default=0.04,
    )

    parser.add_argument(
        "--scale-min",
        type=float,
        default=0.90,
    )

    parser.add_argument(
        "--scale-max",
        type=float,
        default=1.10,
    )

    parser.add_argument(
        "--jitter-sigma",
        type=float,
        default=0.005,
    )

    parser.add_argument(
        "--time-fast",
        type=float,
        default=0.85,
    )

    parser.add_argument(
        "--time-slow",
        type=float,
        default=1.15,
    )

    parser.add_argument(
        "--methods",
        nargs="+",
        choices=SUPPORTED_METHODS,
        default=SUPPORTED_METHODS,
    )

    parser.add_argument(
        "--overwrite",
        action="store_true",
    )

    return parser.parse_args()


def to_array(
    data: dict,
) -> np.ndarray:
    return np.asarray(
        [
            item["keypoints"]
            for item in data["frames"]
        ],
        dtype=np.float32,
    )


def make_output(
    original: dict,
    array: np.ndarray,
) -> dict:
    output = dict(original)
    output_frames = []

    detected_frames = 0
    confidence_values = []

    for frame_index in range(
        len(array)
    ):
        frame = array[
            frame_index
        ]

        detected = bool(
            np.any(
                frame[:, 2] > 0
            )
        )

        if detected:
            detected_frames += 1

        positive_scores = (
            frame[
                frame[:, 2] > 0,
                2,
            ]
        )

        confidence_values.extend(
            positive_scores.tolist()
        )

        output_frames.append(
            {
                "frame_index": frame_index,
                "detected": detected,
                "keypoints": frame.tolist(),
            }
        )

    output["frames"] = output_frames
    output["total_frames"] = len(
        output_frames
    )
    output["detected_frames"] = (
        detected_frames
    )
    output["detection_rate"] = (
        detected_frames
        / len(output_frames)
        if output_frames
        else 0.0
    )
    output[
        "mean_keypoint_confidence"
    ] = (
        float(
            np.mean(
                confidence_values
            )
        )
        if confidence_values
        else 0.0
    )

    return output


def translate(
    array: np.ndarray,
    dx: float,
    dy: float,
) -> np.ndarray:
    result = array.copy()

    valid = (
        result[:, :, 2] > 0
    )

    x = result[:, :, 0]
    y = result[:, :, 1]

    x[valid] = x[valid] + dx
    y[valid] = y[valid] + dy

    result[:, :, 0] = np.clip(
        x,
        0,
        1,
    )

    result[:, :, 1] = np.clip(
        y,
        0,
        1,
    )

    return result


def scale_body(
    array: np.ndarray,
    factor: float,
) -> np.ndarray:
    result = array.copy()

    for frame_index in range(
        len(result)
    ):
        valid = (
            result[
                frame_index,
                :,
                2,
            ] > 0
        )

        if valid.sum() < 2:
            continue

        center = (
            result[
                frame_index,
                valid,
                :2,
            ].mean(
                axis=0
            )
        )

        result[
            frame_index,
            valid,
            :2,
        ] = (
            center
            + (
                result[
                    frame_index,
                    valid,
                    :2,
                ]
                - center
            )
            * factor
        )

    result[:, :, :2] = np.clip(
        result[:, :, :2],
        0,
        1,
    )

    return result


def jitter(
    array: np.ndarray,
    sigma: float,
    rng,
) -> np.ndarray:
    result = array.copy()

    valid = (
        result[:, :, 2] > 0
    )

    noise = rng.normal(
        0,
        sigma,
        size=result[:, :, :2].shape,
    )

    xy = result[:, :, :2]
    xy[valid] = (
        xy[valid]
        + noise[valid]
    )

    result[:, :, :2] = np.clip(
        xy,
        0,
        1,
    )

    return result


def time_warp(
    array: np.ndarray,
    factor: float,
) -> np.ndarray:
    old_length = len(array)

    new_length = max(
        2,
        int(
            round(
                old_length
                * factor
            )
        ),
    )

    old_position = np.linspace(
        0,
        old_length - 1,
        old_length,
    )

    new_position = np.linspace(
        0,
        old_length - 1,
        new_length,
    )

    output = np.zeros(
        (
            new_length,
            array.shape[1],
            array.shape[2],
        ),
        dtype=np.float32,
    )

    for joint in range(
        array.shape[1]
    ):
        for channel in range(
            array.shape[2]
        ):
            output[
                :,
                joint,
                channel,
            ] = np.interp(
                new_position,
                old_position,
                array[
                    :,
                    joint,
                    channel,
                ],
            )

    output[:, :, :2] = np.clip(
        output[:, :, :2],
        0,
        1,
    )

    output[:, :, 2] = np.clip(
        output[:, :, 2],
        0,
        1,
    )

    return output


def save_variant(
    output_dir: Path,
    source_path: Path,
    original: dict,
    array: np.ndarray,
    method: str,
    parameter: str,
    seed: int,
    rows: list[dict],
) -> None:
    output = make_output(
        original,
        array,
    )

    output[
        "augmentation"
    ] = {
        "method": method,
        "parameter": parameter,
        "source": source_path.name,
        "seed": seed,
    }

    output_name = (
        f"{source_path.stem}"
        f"__{method}.json"
    )

    output_path = (
        output_dir
        / output_name
    )

    output_path.write_text(
        json.dumps(
            output,
            ensure_ascii=False,
            indent=2,
        ),
        encoding="utf-8",
    )

    rows.append(
        {
            "file": output_name,
            "source": source_path.name,
            "subject": original["subject"],
            "action": original["action"],
            "method": method,
            "parameter": parameter,
            "seed": seed,
            "approved": "pending",
        }
    )


def main() -> None:
    args = parse_args()

    if not (
        0
        <= args.translate_max
        <= 0.20
    ):
        raise ValueError(
            "--translate-max 범위를 확인하세요."
        )

    if not (
        0.5
        <= args.scale_min
        <= args.scale_max
        <= 1.5
    ):
        raise ValueError(
            "Scale 범위를 확인하세요."
        )

    if not (
        0
        <= args.jitter_sigma
        <= 0.10
    ):
        raise ValueError(
            "--jitter-sigma 범위를 확인하세요."
        )

    if not (
        0.50
        <= args.time_fast
        < 1.0
    ):
        raise ValueError(
            "--time-fast는 0.50 이상 1.0 미만이어야 합니다."
        )

    if not (
        1.0
        < args.time_slow
        <= 1.50
    ):
        raise ValueError(
            "--time-slow는 1.0 초과 1.50 이하여야 합니다."
        )

    source_dir = (
        original_keypoint_dir(
            args.asset_set,
            "train",
        )
    )

    source_files = sorted(
        source_dir.glob(
            "*.json"
        )
    )

    if not source_files:
        raise RuntimeError(
            "Train Original Keypoint JSON이 없습니다."
        )

    output_dir = (
        augmented_keypoint_dir(
            args.asset_set,
            args.tag,
        )
    )

    if any(
        output_dir.iterdir()
    ):
        if not args.overwrite:
            raise RuntimeError(
                "이미 Augmented 결과가 있습니다.\n"
                "다시 만들려면 --overwrite를 사용하세요."
            )

        shutil.rmtree(
            output_dir
        )

        output_dir.mkdir(
            parents=True,
            exist_ok=True,
        )

    rows = []

    for index, source_path in enumerate(
        source_files
    ):
        seed = (
            BASE_SEED
            + index
        )

        py_rng = random.Random(
            seed
        )

        np_rng = np.random.default_rng(
            seed
        )

        original = json.loads(
            source_path.read_text(
                encoding="utf-8"
            )
        )

        array = to_array(
            original
        )

        if (
            array.ndim != 3
            or array.shape[1:] != (
                17,
                3,
            )
        ):
            raise RuntimeError(
                f"Keypoint Shape 오류: "
                f"{source_path.name} "
                f"{array.shape}"
            )

        if "translate" in args.methods:
            dx = py_rng.uniform(
                -args.translate_max,
                args.translate_max,
            )

            dy = py_rng.uniform(
                -args.translate_max,
                args.translate_max,
            )

            save_variant(
                output_dir,
                source_path,
                original,
                translate(
                    array,
                    dx,
                    dy,
                ),
                "translate",
                (
                    f"dx={dx:.5f},"
                    f"dy={dy:.5f}"
                ),
                seed,
                rows,
            )

        if "scale" in args.methods:
            scale_factor = (
                py_rng.uniform(
                    args.scale_min,
                    args.scale_max,
                )
            )

            save_variant(
                output_dir,
                source_path,
                original,
                scale_body(
                    array,
                    scale_factor,
                ),
                "scale",
                (
                    f"factor="
                    f"{scale_factor:.5f}"
                ),
                seed,
                rows,
            )

        if "jitter" in args.methods:
            save_variant(
                output_dir,
                source_path,
                original,
                jitter(
                    array,
                    args.jitter_sigma,
                    np_rng,
                ),
                "jitter",
                (
                    f"sigma="
                    f"{args.jitter_sigma}"
                ),
                seed,
                rows,
            )

        if "timewarp_fast" in args.methods:
            save_variant(
                output_dir,
                source_path,
                original,
                time_warp(
                    array,
                    args.time_fast,
                ),
                "timewarp_fast",
                (
                    f"factor="
                    f"{args.time_fast}"
                ),
                seed,
                rows,
            )

        if "timewarp_slow" in args.methods:
            save_variant(
                output_dir,
                source_path,
                original,
                time_warp(
                    array,
                    args.time_slow,
                ),
                "timewarp_slow",
                (
                    f"factor="
                    f"{args.time_slow}"
                ),
                seed,
                rows,
            )

    metadata_path = (
        report_dir(
            args.asset_set
        )
        / f"{args.tag}_metadata.csv"
    )

    pd.DataFrame(
        rows
    ).to_csv(
        metadata_path,
        index=False,
        encoding="utf-8-sig",
    )

    print("=" * 68)
    print("Sequence Augmentation Completed")
    print("=" * 68)
    print(
        "Methods       :",
        ", ".join(args.methods),
    )
    print(
        "Original Train:",
        len(source_files),
    )
    print(
        "Augmented     :",
        len(rows),
    )
    print(
        "Output        :",
        output_dir,
    )
    print(
        "Metadata      :",
        metadata_path,
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[프로그램 시작]
        ↓
① 필요한 라이브러리와 공통 함수 불러오기
   │
   ├─ argparse → 실행 옵션 확인
   ├─ json     → Keypoint JSON 읽기·저장
   ├─ random   → 보강에 사용할 무작위 값 생성
   ├─ NumPy    → 관절 좌표 변형·계산
   ├─ Pandas   → Metadata CSV 저장
   └─ day05_common.py → 입력·출력 폴더 관리
        ↓
② 사용할 보강 방법과 기본 설정 확인
   │
   ├─ Translation    → 최대 이동값 0.04
   ├─ Scale          → 0.90 ~ 1.10
   ├─ Jitter         → sigma 0.005
   ├─ Time Warp Fast → 0.85
   └─ Time Warp Slow → 1.15
        ↓
③ 실행 옵션 확인 — parse_args()
   │
   ├─ --asset-set → main / challenge (필수)
   ├─ --tag       → 이번 실험 결과를 구분할 이름 (필수)
   ├─ --methods   → 적용할 보강 방법 선택 (기본 5가지)
   ├─ --translate-max → 이동 범위
   ├─ --scale-min / --scale-max → 크기 변형 범위
   ├─ --jitter-sigma → 좌표 오차 강도
   ├─ --time-fast / --time-slow → 시간 변형 비율
   └─ --overwrite → 기존 실험 결과를 삭제하고 재생성
        ↓
④ 보강 설정값 검사 — main()
   │
   ├─ Translation 이동 범위가 유효한가?
   ├─ Scale 최소·최대 범위가 유효한가?
   ├─ Jitter 강도가 유효한가?
   └─ Time Warp 비율이 유효한가?
        ↓
   잘못된 값이 있는가?
   ├─ 예 → ValueError 발생 및 중단
   └─ 아니요 → 다음 단계
        ↓
⑤ Train 원본 Keypoint JSON 불러오기
   │
   └─ data/keypoints/main/original/train/
        ↓
   원본 JSON이 존재하는가?
   ├─ 아니요 → 오류 발생 및 중단
   └─ 예 → JSON 목록 가져오기
        ↓
   Main Train 원본 24개
        ↓
⑥ 보강 데이터 저장 폴더 준비
   │
   └─ augmented_keypoint_dir(asset_set, tag)
        ↓
   같은 tag의 기존 결과가 있는가?
   │
   ├─ 예
   │    ├─ --overwrite 없음 → 오류 발생 및 중단
   │    └─ --overwrite 사용 → 기존 결과 삭제 후 재생성
   │
   └─ 아니요 → 새 결과 저장 준비
        ↓
⑦ Train JSON 파일 하나 선택
   │
   ├─ 원본 JSON 읽기
   ├─ Frame별 Keypoint 가져오기 — to_array()
   └─ NumPy 배열로 변환
        ↓
   배열 구조 확인
   │
   └─ [Frame 수, 17개 Keypoint, 3개 값]
        ↓
   배열 구조가 정상인가?
   ├─ 아니요 → 오류 발생 및 중단
   └─ 예 → 다음 단계
        ↓
⑧ 무작위 값 생성 준비
   │
   ├─ Seed = 42 + 원본 파일 순번
   ├─ Python Random 준비
   └─ NumPy Random 준비
        ↓
   같은 Seed를 사용하면
   같은 입력·설정에서 보강 결과를 재현할 수 있음
        ↓
⑨ 다섯 가지 Sequence Augmentation 적용
   │
   ├──────────────────────────────────────┐
   │                                      │
   ▼                                      │
   [A. Translation — translate()]         │
   │                                      │
   ├─ dx, dy 무작위 선택                   │
   ├─ 신뢰도 > 0인 관절만 선택              │
   ├─ 모든 유효 관절의 x에 dx 더하기         │
   ├─ 모든 유효 관절의 y에 dy 더하기         │
   ├─ 좌표를 0~1 범위로 제한                │
   └─ 위치가 이동한 새 Sequence 생성        │
   │                                      │
   │  예: (0.50, 0.30)                    │
   │       → (0.54, 0.32)                 │
   │                                      │
   ▼                                      │
   [B. Scale — scale_body()]              │
   │                                      │
   ├─ Scale 비율 무작위 선택                │
   ├─ 각 Frame의 유효 관절 좌표 확인         │
   ├─ 유효 관절이 2개 미만이면 건너뛰기       │
   ├─ 유효 관절의 평균 좌표를 중심으로 계산    │
   ├─ 중심에서 관절까지 거리를 확대·축소      │
   ├─ 좌표를 0~1 범위로 제한                │
   └─ 관절 사이 거리가 달라진 Sequence 생성  │
   │                                      │
   ▼                                      │
   [C. Jitter — jitter()]                 │
   │                                      │
   ├─ Gaussian Noise 생성                  │
   ├─ 신뢰도 > 0인 관절만 선택               │
   ├─ x, y 좌표에 작은 Noise 추가           │
   ├─ 좌표를 0~1 범위로 제한                │
   └─ 미세한 좌표 오차가 있는 Sequence 생성  │
   │                                      │
   ▼                                      │
   [D. Time Warp Fast — time_warp()]      │
   │                                      │
   ├─ 원본 Frame 수 확인                   │
   ├─ 새 Frame 수 = 원본 수 × 0.85         │
   ├─ 최소 2개 Frame 유지                  │
   ├─ 관절별 x, y, confidence 보간         │
   ├─ 좌표와 신뢰도를 0~1 범위로 제한        │
   └─ Frame 수가 줄어든 Sequence 생성       │
   │                                      │
   │  예: 원본 100 Frame                   │
   │       → 85 Frame                     │
   │                                      │
   ▼                                      │
   [E. Time Warp Slow — time_warp()]      │
   │                                      │
   ├─ 원본 Frame 수 확인                   │
   ├─ 새 Frame 수 = 원본 수 × 1.15         │
   ├─ 관절별 x, y, confidence 보간         │
   ├─ 좌표와 신뢰도를 0~1 범위로 제한        │
   └─ Frame 수가 늘어난 Sequence 생성       │
   │                                      │
   │  예: 원본 100 Frame                   │
   │       → 115 Frame                    │
   │                                      │
   └──────────────────┬───────────────────┘
                      ↓
⑩ 각 보강 결과를 JSON으로 변환 — make_output()
   │
   ├─ 변형된 Frame을 처음부터 다시 확인
   ├─ confidence > 0인 관절이 하나라도 있으면
   │  detected = True로 기록
   ├─ Frame 번호 다시 부여
   ├─ total_frames 다시 계산
   ├─ detected_frames 다시 계산
   ├─ detection_rate 다시 계산
   └─ mean_keypoint_confidence 다시 계산
        ↓
⑪ 보강 정보 기록 — save_variant()
   │
   ├─ method    → 어떤 보강 방법을 사용했는가?
   ├─ parameter → 어떤 설정값을 사용했는가?
   ├─ source    → 어떤 원본 JSON에서 만들었는가?
   └─ seed      → 어떤 무작위 Seed를 사용했는가?
        ↓
⑫ 보강된 JSON 파일 저장
   │
   ├─ S11_boxing_T01__translate.json
   ├─ S11_boxing_T01__scale.json
   ├─ S11_boxing_T01__jitter.json
   ├─ S11_boxing_T01__timewarp_fast.json
   └─ S11_boxing_T01__timewarp_slow.json
        ↓
   원본 JSON 1개에서
   보강 JSON 5개 생성
        ↓
⑬ 다음 Train 원본 JSON이 있는가?
   │
   ├─ 예 → ⑦~⑫ 반복
   └─ 아니요 → 보강 작업 완료
        ↓
   원본 Train 24개 × 5가지 방법
        ↓
   보강 JSON 총 120개 생성
        ↓
⑭ Metadata CSV 생성
   │
   ├─ 보강 JSON 파일명
   ├─ 원본 JSON 파일명
   ├─ Subject / Action
   ├─ 보강 방법
   ├─ 사용한 설정값
   ├─ Random Seed
   └─ approved = pending
        ↓
   reports/main/{tag}_metadata.csv 저장
        ↓
⑮ 실행 결과 출력
   │
   ├─ Methods        : 적용한 보강 방법
   ├─ Original Train : 24
   ├─ Augmented      : 120
   ├─ Output         : 보강 JSON 저장 폴더
   └─ Metadata       : Metadata CSV 저장 경로
        ↓
[프로그램 종료]


──────────────────────────────────────
[최종 생성 결과]
──────────────────────────────────────

원본 Train JSON 24개
        │
        └─ 그대로 보존

새로운 Augmented JSON 120개
        │
        └─ 다섯 가지 방법으로 변형한 관절 데이터

Metadata CSV
        │
        └─ 어떤 원본을 어떻게 변형했는지 기록
        │
        └─ QA 승인 상태는 pending

        ↓
다음 PART — Augmented Data QA
        ↓
좌표 범위·Frame 수·관절 움직임 검사
        ↓
학습에 사용할 보강 데이터 선택
```

---
# 20. Main Train 데이터를 다섯 가지 방법으로 보강합니다

19번에서 작성한 코드를 실행하여 **Train의 원본 Keypoint JSON 24개를 변형하고 새로운 학습데이터를 생성**하겠습니다.

이 과정에서는 실제 영상을 수정하지 않습니다. 영상에서 추출한 관절 좌표와 시간 흐름을 변형하고, 그 결과를 별도의 JSON 파일로 저장합니다.

### ① Sequence Augmentation 실행하기

```bash
python scripts/03_augment_keypoints.py \
  --asset-set main \
  --tag main_aug \
  --jitter-sigma 0.005
```

- `--asset-set main` : Main 데이터셋의 Train을 사용합니다.
    
- `--tag main_aug` : 생성한 결과를 `main_aug`라는 이름으로 구분합니다.
    
- `--jitter-sigma 0.005` : Jitter에 적용할 좌표 오차 강도를 지정합니다.
    

`--methods`를 생략했으므로 다섯 가지 보강 방법이 모두 적용됩니다.

### ② 실제 생성 결과 확인하기

첨부된 Metadata CSV를 확인한 결과, 다음과 같이 보강 데이터가 기록되어 있습니다.

|보강 방법|생성 기록 수|
|---|---|
|Translation|24개|
|Scale|24개|
|Jitter|24개|
|Time Warp Fast|24개|
|Time Warp Slow|24개|
|**합계**|**120개**|

즉, **원본 JSON 하나에서 다섯 종류의 새로운 Keypoint Sequence를 생성하는 방식**입니다.

```
원본 Train JSON 24개
          ↓
다섯 가지 방법으로 좌표·시간 변형
          ↓
보강 데이터 120개 생성
          ↓
원본 데이터는 그대로 보존
```

### ③ 생성된 JSON 파일 확인하기

보강된 데이터는 다음 폴더에 저장하도록 구성했습니다.

```
data/keypoints/main/augmented/main_aug/
│
├── S11_boxing_T01__translate.json
├── S11_boxing_T01__scale.json
├── S11_boxing_T01__jitter.json
├── S11_boxing_T01__timewarp_fast.json
├── S11_boxing_T01__timewarp_slow.json
│
└── ... 나머지 보강 JSON
```

실제 JSON 파일이 120개 있는지도 확인해 보겠습니다.

```bash
find data/keypoints/main/augmented/main_aug \
  -maxdepth 1 -type f -name "*.json" | wc -l
```

정상적으로 모두 저장되었다면 `120`이 출력됩니다.

### ④ 실제 Jitter 보강 결과 살펴보기

첨부된 `S11_boxing_T01__jitter.json`을 보면 다음 보강 정보가 기록되어 있습니다.

```json
"augmentation": {
  "method": "jitter",
  "parameter": "sigma=0.005",
  "source": "S11_boxing_T01.json",
  "seed": 42
}
```

원본과 보강된 첫 번째 Frame의 코(0번 Keypoint) 좌표를 비교해 보겠습니다.

```
원본 코 좌표
[0.42917724, 0.25337435, 0.921102]
              ↓
      Jitter 보강 적용
              ↓
보강된 코 좌표
[0.43070084, 0.24817441, 0.921102]
```

관절의 x, y 좌표가 조금 달라졌지만 신뢰도는 유지되었습니다. 이것이 **실제 영상을 바꾸지 않고 관절 위치에 작은 오차를 추가한 결과**입니다.

이 예시에서 원본과 Jitter 보강 데이터의 Frame 수는 모두 351개입니다.

### ⑤ Metadata CSV 확인하기

생성한 보강 데이터의 정보를 다음 파일에 기록했습니다.

```
reports/main/main_aug_metadata.csv
```

Metadata에는 원본 파일명, 보강 방법, 적용한 설정값, Random Seed, 승인 상태가 기록됩니다.

실제 CSV의 첫 번째 기록은 다음과 같습니다.

|항목|실제 값|
|---|---|
|원본 파일|`S11_boxing_T01.json`|
|보강 방법|`translate`|
|적용값|`dx=0.01115, dy=-0.03800`|
|Seed|`42`|
|승인 상태|`pending`|

여기서 `pending`은 **데이터가 생성되었지만 아직 품질 검사를 통과하지 않았다는 의미**입니다.

```
보강 데이터 생성
        ↓
Metadata 기록
        ↓
approved = pending
        ↓
아직 학습 사용 승인 전
        ↓
다음 PART에서 QA 진행
```

### ⑥ 이번 실습 결과 정리

이번 실습에서는 원본 Train JSON 24개에 다섯 가지 보강 방법을 적용하여 총 120건의 보강 데이터 생성 기록을 확인했습니다.

**하지만 데이터가 많아졌다고 해서 모두 좋은 학습데이터가 되는 것은 아닙니다.**

---
# 21. 보강된 Keypoint를 Skeleton 영상으로 확인합니다

20번에서는 Train 원본 JSON 24개를 다섯 가지 방법으로 변형하여 **120개의 보강 JSON**을 
생성했습니다.

이번에는 실제로 생성한 `S11_boxing_T01`의 보강 데이터 5개를 **Skeleton 영상으로 변환하여 
관절 움직임이 자연스러운지 확인**하겠습니다.

### ① 보강 데이터를 MP4로 변환하기

```bash
for METHOD in translate scale jitter timewarp_fast timewarp_slow
do
  python scripts/02_visualize_keypoints.py \
    --json data/keypoints/main/augmented/main_aug/S11_boxing_T01__${METHOD}.json \
    --output results/main/S11_boxing_T01_${METHOD}.mp4
done
```

이 명령어는 다섯 가지 보강 JSON을 차례대로 읽어 각각 MP4 영상으로 저장합니다.

`--video`를 지정하지 않았으므로 **원본 사람이 나오는 영상이 아니라, 어두운 배경에 관절 점과 Skeleton만 표시**됩니다.

### ② 생성되는 파일 확인하기

```
results/main/
├─ S11_boxing_T01_translate.mp4
├─ S11_boxing_T01_scale.mp4
├─ S11_boxing_T01_jitter.mp4
├─ S11_boxing_T01_timewarp_fast.mp4
└─ S11_boxing_T01_timewarp_slow.mp4
```

### ③ 다섯 영상의 움직임 비교하기

|영상|확인할 내용|
|---|---|
|Translation|관절 위치가 이동해도 권투 동작이 유지되는가?|
|Scale|사람의 Skeleton 크기가 자연스럽게 변했는가?|
|Jitter|손목과 팔꿈치가 과도하게 흔들리지 않는가?|
|Time Warp Fast|움직임이 지나치게 빨라지지 않았는가?|
|Time Warp Slow|움직임이 지나치게 느려지지 않았는가?|

실제 원본 `S11_boxing_T01`은 351 Frame입니다. 보강된 데이터에서는 Time Warp Fast가 약 298 Frame, Slow가 약 404 Frame으로 변경됩니다.

### ④ 이번 실습 결과 정리

```
보강 JSON 5개
       ↓
Skeleton MP4 5개 생성
       ↓
관절 움직임을 직접 비교
       ↓
부자연스러운 변형 확인
       ↓
다음 PART — Human QA 판단에 활용
```

**핵심:** 보강 데이터의 생성이 완료되었더라도 실제 동작이 자연스러운지는 별도로 확인해야 합니다.

이번 Preview에서 확인한 내용은 다음 PART의 Human QA에서 학습데이터 사용 여부를 판단하는 근거가 됩니다.

---
# PART 4. 원본과 보강된 Keypoint 데이터의 품질을 검사합니다

앞에서는 원본 Keypoint 24개를 보강하여 120개의 새로운 JSON을 만들고, Skeleton 영상으로 움직임을 확인했습니다.

이번 PART에서는 **원본과 보강된 데이터에 오류가 없는지 검사하고, AI 모델 학습에 사용해도 되는지 판단**하겠습니다.

---

# 22. Automatic QA와 Human QA의 차이를 이해합니다

QA(Quality Assurance)는 데이터의 품질을 확인하는 과정입니다.

이번에는 두 가지 방법으로 검사합니다.

|구분|Automatic QA (자동 검사)|Human QA (사람의 검사)|
|---|---|---|
|검사 방법|Python 코드로 검사|Skeleton 영상을 보고 판단|
|검사 내용|JSON 구조, 관절 개수, 좌표 오류, 유효 Frame 비율|동작의 자연스러움, 관절의 흔들림, 움직임 속도|
|목적|데이터에 기술적인 문제가 없는지 확인|실제 동작의 의미가 유지되는지 확인|

예를 들어 JSON에 17개 관절 좌표가 모두 정상적으로 저장되어 있어도, 팔이 갑자기 비정상적인 방향으로 움직일 수 있습니다.

따라서 **자동 검사에서 문제가 없더라도 사람의 눈으로 움직임을 확인하는 과정이 필요합니다.**

---

# 23. Automatic QA 코드를 작성합니다

### 이번 코드는 무엇을 하나요?

앞에서 만든 **원본과 보강된 Keypoint JSON을 읽어 데이터 구조와 관절 좌표에 문제가 없는지 자동으로 검사하는 프로그램**입니다.

특히 관절이 충분히 검출된 Frame의 비율과 관절 움직임의 변화량을 계산합니다.

### 코드의 전체 처리 흐름

```
원본 + 보강 Keypoint JSON
             ↓
       JSON 파일 읽기
             ↓
       데이터 구조 검사
       ├─ Frame이 존재하는가?
       ├─ 관절이 17개인가?
       ├─ NaN / Infinity가 없는가?
       └─ 좌표가 0~1 범위인가?
             ↓
       유효 Frame 비율 계산
             ↓
       관절 이동량 계산
       ├─ mean_joint_step
       └─ max_joint_step
             ↓
       QA 결과 기록
             ↓
       Human QA에서 추가 확인
```

### 유효 Frame은 어떻게 판단할까요?

이번 실습에서는 한 Frame에 **Confidence가 0.10 이상인 관절이 8개 이상** 있으면 유효 Frame으로 계산합니다.

예를 들어 전체 100 Frame 중 90 Frame이 이 조건을 만족했다면 유효 Frame 비율은 `0.90`입니다.

유효 Frame 비율이 `0.30` 미만이면 자동으로 추가 검토(Review) 대상으로 분류합니다.

이 기준은 오늘 실습을 위한 교육용 설정이며, 실제 산업 현장의 절대적인 품질 기준은 아닙니다.

### 관절 이동량도 확인합니다

- `mean_joint_step` : 연속된 Frame 사이에서 관절이 이동한 거리의 평균입니다.
    
- `max_joint_step` : 연속된 Frame 사이에서 관찰된 가장 큰 관절 이동량입니다.
    

이 값이 크다고 무조건 오류는 아닙니다. 권투처럼 빠른 움직임이 있는 동작도 있기 때문입니다.

따라서 이동량은 자동 불합격 기준이 아니라 **원본과 보강 데이터의 움직임을 비교하는 참고 지표**로 사용합니다.

이번 실습에서는 데이터의 기술적인 오류를 자동으로 검사하고, 다음 Human QA에서는 Skeleton 움직임까지 확인하여 학습에 사용할 데이터를 결정하겠습니다.

**파일: `scripts/04_qa_keypoints.py`**

```python
from __future__ import annotations

import argparse
import json
import math
import sys
from pathlib import Path

import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (
    augmented_keypoint_dir,
    original_keypoint_dir,
    report_dir,
)


MIN_VALID_FRAME_RATIO = 0.30
JOINT_CONFIDENCE = 0.10


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--asset-set",
        required=True,
        choices=["main", "challenge"],
    )

    parser.add_argument(
        "--tag",
        required=True,
    )

    return parser.parse_args()


def motion_statistics(
    frames: list[dict],
) -> tuple[float, float]:
    steps = []

    previous = None

    for frame in frames:
        keypoints = frame.get(
            "keypoints",
            []
        )

        if len(keypoints) != 17:
            previous = None
            continue

        current = np.asarray(
            keypoints,
            dtype=np.float32,
        )

        if previous is not None:
            valid = (
                (previous[:, 2] >= JOINT_CONFIDENCE)
                & (current[:, 2] >= JOINT_CONFIDENCE)
            )

            if valid.any():
                delta = (
                    current[
                        valid,
                        :2,
                    ]
                    - previous[
                        valid,
                        :2,
                    ]
                )

                distance = np.linalg.norm(
                    delta,
                    axis=1,
                )

                steps.extend(
                    distance.tolist()
                )

        previous = current

    if not steps:
        return 0.0, 0.0

    return (
        float(np.mean(steps)),
        float(np.max(steps)),
    )


def check_file(
    path: Path,
    split: str,
    source_type: str,
) -> dict:
    reasons = []

    try:
        data = json.loads(
            path.read_text(
                encoding="utf-8"
            )
        )
    except Exception:
        return {
            "file": path.name,
            "split": split,
            "source_type": source_type,
            "subject": "",
            "action": "",
            "method": "",
            "frames": 0,
            "valid_frame_ratio": 0.0,
            "mean_joint_step": 0.0,
            "max_joint_step": 0.0,
            "invalid_value": 1,
            "out_of_range": 0,
            "wrong_joint_count": 0,
            "auto_pass": False,
            "reason": "json_read_error",
        }

    frames = data.get(
        "frames",
        []
    )

    if not frames:
        reasons.append(
            "no_frames"
        )

    valid_frames = 0
    invalid_value = 0
    out_of_range = 0
    wrong_joint_count = 0

    for frame in frames:
        keypoints = frame.get(
            "keypoints",
            []
        )

        if len(keypoints) != 17:
            wrong_joint_count += 1
            continue

        valid_joint_count = 0

        for point in keypoints:
            if len(point) != 3:
                invalid_value += 1
                continue

            x, y, score = point

            values = [
                x,
                y,
                score,
            ]

            if not all(
                isinstance(
                    value,
                    (int, float),
                )
                and math.isfinite(
                    value
                )
                for value in values
            ):
                invalid_value += 1
                continue

            if score < 0 or score > 1:
                invalid_value += 1
                continue

            if score > 0 and not (
                0 <= x <= 1
                and 0 <= y <= 1
            ):
                out_of_range += 1

            if score >= JOINT_CONFIDENCE:
                valid_joint_count += 1

        if valid_joint_count >= 8:
            valid_frames += 1

    frame_count = len(
        frames
    )

    valid_ratio = (
        valid_frames / frame_count
        if frame_count > 0
        else 0.0
    )

    if (
        valid_ratio
        < MIN_VALID_FRAME_RATIO
    ):
        reasons.append(
            "low_valid_frame_ratio"
        )

    if invalid_value > 0:
        reasons.append(
            "invalid_value"
        )

    if out_of_range > 0:
        reasons.append(
            "out_of_range"
        )

    if wrong_joint_count > 0:
        reasons.append(
            "wrong_joint_count"
        )

    (
        mean_joint_step,
        max_joint_step,
    ) = motion_statistics(frames)

    augmentation = data.get(
        "augmentation",
        {},
    )

    return {
        "file": path.name,
        "split": split,
        "source_type": source_type,
        "subject": data.get(
            "subject",
            "",
        ),
        "action": data.get(
            "action",
            "",
        ),
        "method": (
            augmentation.get(
                "method",
                "original",
            )
        ),
        "frames": frame_count,
        "valid_frame_ratio": round(
            valid_ratio,
            4,
        ),
        "mean_joint_step": round(
            mean_joint_step,
            6,
        ),
        "max_joint_step": round(
            max_joint_step,
            6,
        ),
        "invalid_value": invalid_value,
        "out_of_range": out_of_range,
        "wrong_joint_count": (
            wrong_joint_count
        ),
        "auto_pass": (
            len(reasons) == 0
        ),
        "reason": "|".join(
            reasons
        ),
    }


def main() -> None:
    args = parse_args()

    rows = []

    for split in [
        "train",
        "val",
        "test",
    ]:
        directory = (
            original_keypoint_dir(
                args.asset_set,
                split,
            )
        )

        for path in sorted(
            directory.glob(
                "*.json"
            )
        ):
            rows.append(
                check_file(
                    path,
                    split,
                    "original",
                )
            )

    aug_dir = (
        augmented_keypoint_dir(
            args.asset_set,
            args.tag,
        )
    )

    for path in sorted(
        aug_dir.glob(
            "*.json"
        )
    ):
        rows.append(
            check_file(
                path,
                "train",
                "augmented",
            )
        )

    if not rows:
        raise RuntimeError(
            "QA할 Keypoint JSON이 없습니다."
        )

    dataframe = pd.DataFrame(
        rows
    )

    output_path = (
        report_dir(
            args.asset_set
        )
        / f"{args.tag}_qa.csv"
    )

    dataframe.to_csv(
        output_path,
        index=False,
        encoding="utf-8-sig",
    )

    passed = int(
        dataframe[
            "auto_pass"
        ].sum()
    )

    print("=" * 68)
    print("Keypoint Sequence QA")
    print("=" * 68)
    print(
        "Total :",
        len(dataframe),
    )
    print(
        "PASS  :",
        passed,
    )
    print(
        "REVIEW:",
        len(dataframe) - passed,
    )
    print(
        "Saved:",
        output_path,
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[프로그램 시작]
        ↓
① 필요한 라이브러리와 공통 함수 불러오기
   │
   ├─ argparse → 실행 옵션 확인
   ├─ json     → Keypoint JSON 읽기
   ├─ math     → NaN / Infinity 검사
   ├─ NumPy    → 관절 이동거리 계산
   ├─ Pandas   → QA 결과 CSV 저장
   └─ day05_common.py → 데이터 경로 관리
        ↓
② Automatic QA 기준 설정
   │
   ├─ JOINT_CONFIDENCE = 0.10
   │   → 관절 신뢰도 기준
   │
   └─ MIN_VALID_FRAME_RATIO = 0.30
       → 유효 Frame 비율 최소 기준
        ↓
③ 실행 옵션 확인 — parse_args()
   │
   ├─ --asset-set → main / challenge
   └─ --tag       → 보강 데이터 실험 이름
        ↓
④ 검사할 JSON 파일 수집 — main()
   │
   ├─ Original Train JSON 24개
   ├─ Original Validation JSON 12개
   ├─ Original Test JSON 12개
   │
   └─ Augmented Train JSON 120개
        ↓
   검사 대상 총 168개 (Main 기본 실습 기준)
        ↓
⑤ JSON 파일 하나 선택 — check_file()
   │
   └─ JSON 파일 읽기
        ↓
   JSON을 정상적으로 읽었는가?
   │
   ├─ 아니요
   │   ├─ json_read_error 기록
   │   ├─ auto_pass = False
   │   └─ 다음 JSON 검사
   │
   └─ 예
        ↓
⑥ Frame 데이터가 존재하는가?
   │
   ├─ 아니요 → no_frames 기록
   └─ 예 → 다음 검사 진행
        ↓
⑦ Frame을 하나씩 검사
   │
   ├─ 관절이 정확히 17개인가?
   │   │
   │   ├─ 아니요
   │   │   ├─ wrong_joint_count 증가
   │   │   └─ 다음 Frame 검사
   │   │
   │   └─ 예 → 각 관절 검사
   │
   ├─ 관절 값이 [x, y, confidence]
   │   총 3개로 구성되어 있는가?
   │   │
   │   ├─ 아니요 → invalid_value 증가
   │   └─ 예 → 다음 검사
   │
   ├─ 숫자가 정상인가?
   │   │
   │   ├─ 숫자가 아니거나 NaN / Infinity
   │   │     └─ invalid_value 증가
   │   │
   │   └─ 정상 → 다음 검사
   │
   ├─ confidence가 0~1 범위인가?
   │   │
   │   ├─ 아니요 → invalid_value 증가
   │   └─ 예 → 다음 검사
   │
   ├─ confidence > 0인 관절의
   │   x, y 좌표가 0~1 범위인가?
   │   │
   │   ├─ 아니요 → out_of_range 증가
   │   └─ 예 → 다음 검사
   │
   └─ confidence ≥ 0.10인가?
       │
       ├─ 예 → 유효 관절 수 증가
       └─ 아니요 → 유효 관절 수 증가하지 않음
        ↓
⑧ 현재 Frame에 유효 관절이 8개 이상인가?
   │
   ├─ 예 → valid_frames 1 증가
   └─ 아니요 → 증가하지 않음
        ↓
⑨ 다음 Frame이 있는가?
   │
   ├─ 예 → ⑦~⑧ 반복
   └─ 아니요 → Frame 검사 완료
        ↓
⑩ 유효 Frame 비율 계산
   │
   └─ valid_frame_ratio
      = 유효 Frame 수 / 전체 Frame 수
        ↓
   유효 Frame 비율이 0.30 미만인가?
   │
   ├─ 예 → low_valid_frame_ratio 기록
   └─ 아니요 → 비율 기준 통과
        ↓
⑪ 지금까지 발견한 오류 확인
   │
   ├─ invalid_value > 0
   │   → invalid_value 기록
   │
   ├─ out_of_range > 0
   │   → out_of_range 기록
   │
   └─ wrong_joint_count > 0
       → wrong_joint_count 기록
        ↓
⑫ 관절 움직임 분석 — motion_statistics()
   │
   ├─ 이전 Frame의 Keypoint 준비
   ├─ 현재 Frame의 Keypoint 확인
   │
   ├─ 연속된 두 Frame 모두에서
   │   confidence ≥ 0.10인 관절 선택
   │
   ├─ 같은 관절의 x, y 좌표 차이 계산
   │
   ├─ 관절별 이동거리 계산
   │   distance = √(dx² + dy²)
   │
   ├─ 전체 이동거리의 평균 계산
   │   → mean_joint_step
   │
   └─ 가장 큰 이동거리 계산
       → max_joint_step
        ↓
   ※ 관절 개수가 17개가 아닌 Frame에서는
     이전 Frame과의 이동량 비교를 끊음
        ↓
⑬ Automatic QA 결과 판단
   │
   ├─ 기록된 오류 이유가 없는가?
   │   │
   │   ├─ 예 → auto_pass = True (PASS)
   │   └─ 아니요 → auto_pass = False (REVIEW)
   │
   └─ mean_joint_step / max_joint_step은
       참고 지표로만 기록
       (자동 PASS / REVIEW 기준에 사용하지 않음)
        ↓
⑭ 현재 JSON의 QA 결과 기록
   │
   ├─ 파일명 / Split / Original·Augmented
   ├─ Subject / Action / Augmentation Method
   ├─ Frame 수
   ├─ 유효 Frame 비율
   ├─ 평균·최대 관절 이동량
   ├─ 오류 개수
   ├─ auto_pass
   └─ reason (검사에서 발견한 문제)
        ↓
⑮ 다음 JSON 파일이 있는가?
   │
   ├─ 예 → ⑤~⑭ 반복
   └─ 아니요 → 전체 검사 완료
        ↓
⑯ 검사한 JSON이 하나도 없는가?
   │
   ├─ 예 → 오류 발생 및 중단
   └─ 아니요 → QA 보고서 생성
        ↓
⑰ 전체 QA 결과를 CSV로 저장
   │
   └─ reports/main/main_aug_qa.csv
        ↓
⑱ 최종 검사 결과 출력
   │
   ├─ Total  → 전체 검사 파일 수
   ├─ PASS   → 자동 검사 통과 수
   ├─ REVIEW → 추가 검토 필요 수
   └─ Saved  → QA 보고서 경로
        ↓
[프로그램 종료]


────────────────────────────────────
[최종 결과]
────────────────────────────────────

Original JSON 48개
       +
Augmented JSON 120개
          ↓
     Automatic QA
          ↓
   ┌──────┴────────┐
   ↓               ↓
  PASS           REVIEW
   │               │
   │               └─ 문제 원인 확인
   │
   └─ 자동 검사 기준 통과
          ↓
    Human QA 진행
          ↓
Skeleton 움직임의 자연스러움 확인
          ↓
학습데이터 사용 여부 최종 결정
```

---
# 24. Automatic QA 결과를 확인합니다

앞에서 실행한 Automatic QA 결과를 확인하겠습니다.

**실제 검사 결과**

|검사 대상|파일 수|PASS|REVIEW|
|---|---|---|---|
|Original Train|24|24|0|
|Original Validation|12|12|0|
|Original Test|12|12|0|
|Augmented Train|120|120|0|
|**합계**|**168**|**168**|**0**|

모든 JSON이 자동 검사 기준을 통과했으며, 데이터 구조와 좌표 범위 등의 오류는 발견되지 않았습니다.

QA 결과 파일:

```
reports/main/main_aug_qa.csv
```

특히 다음 항목을 확인합니다.

|항목|의미|
|---|---|
|`valid_frame_ratio`|관절이 충분히 검출된 Frame의 비율|
|`mean_joint_step`|Frame 사이 평균 관절 이동량|
|`max_joint_step`|Frame 사이 최대 관절 이동량|
|`auto_pass`|자동 검사 통과 여부|
|`reason`|오류가 발견된 이유|

**실제 결과에서 주의할 파일**

```
S13_handwaving_T02__timewarp_fast.json

auto_pass       = True
valid_frame_ratio = 0.9976
max_joint_step  = 0.763015
```

이 파일은 자동 검사에 통과했지만 관절의 최대 이동량이 크게 나타났습니다.

따라서 Skeleton 영상을 직접 확인하여 움직임이 자연스러운지 살펴봐야 합니다.

---

# 25. Human QA로 보강 데이터의 품질을 확인합니다

Automatic QA에서는 원본 48개와 보강 데이터 120개, 총 **168개가 모두 PASS**되었습니다.

이제 Skeleton 영상을 보고 실제 움직임이 자연스러운지 확인하겠습니다.

### ① 실제 Skeleton 결과 확인하기

생성한 다섯 가지 보강 영상을 확인한 결과, 사람의 자세와 팔 움직임은 대체로 구분됩니다.

다만 일부 화면에서는 관절이 서로 겹쳐 보이므로, **영상 전체를 재생하면서 동작이 자연스럽게 
이어지는지 확인**해야 합니다.

특히 다음을 살펴봅니다.

- Translation / Scale: 사람의 위치와 크기가 자연스러운가?
    
- Jitter: 손목과 팔꿈치가 과도하게 흔들리지 않는가?
    
- Time Warp: 움직임이 부자연스럽게 빨라지거나 느려지지 않는가?
    

### ② 실제 QA에서 발견한 주의사항

`S13_handwaving_T02__timewarp_fast.json`의 검사 결과입니다.

```
Automatic QA : PASS
Frame 수     : 418
max_joint_step : 0.763015

398 → 399 Frame
왼쪽 발목 좌표가 크게 이동

400 Frame
사람 검출 누락
```

자동 검사는 통과했지만, 해당 구간에서 관절 위치가 비정상적으로 변했을 가능성이 있습니다.

따라서 이 데이터는 **추가 확인이 필요한 상태(needs_review)**로 판단합니다.

### ③ Human QA 결과 기록하기

직접 영상을 실제 움직임을 보고 판단해야 합니다.

**파일:** `**reports/main/main_aug_human_qa.csv**` 

작성 예시입니다.

```
method,decision,note
translate,needs_review,전체 움직임 확인 필요
scale,needs_review,관절 겹침 및 크기 변화 확인 필요
jitter,needs_review,관절 흔들림 확인 필요
timewarp_fast,needs_review,S13 handwaving의 Frame 398~400 이상 구간 확인 필요
timewarp_slow,needs_review,전체 움직임 속도 확인 필요
```

현재는 정지 이미지와 일부 JSON 결과만 확인했으므로 
모든 방법을 우선 `needs_review`로 기록합니다.

영상 전체를 확인한 뒤 문제가 없다면 `approved`, 문제가 있으면 `rejected`로 변경합니다.

### 실제 관찰을 반영한 QA 결과

| 보강 방법          | 판단             | 주요 확인 사항                |
| -------------- | -------------- | ----------------------- |
| Translation    | `needs_review` | 자세가 갑자기 변하는 구간          |
| Scale          | `needs_review` | Skeleton이 지나치게 좁아지는 구간  |
| Jitter         | `needs_review` | 인접 Frame 사이 관절의 급격한 움직임 |
| Time Warp Fast | `needs_review` | S13 영상의 관절 이동량 급증       |
| Time Warp Slow | `needs_review` | 자세 변화와 Skeleton 폭 축소    |

현재 전부 `needs_review`로 작성한 이유

Frame 간격이 짧은데 자세가 갑자기 크게 바뀌거나, 몸통과 다리가 거의 일직선으로 겹치는 장면이 있습니다.

다만 이것이 실제 사람이 옆으로 회전한 결과인지, 원본 YOLO Pose의 관절 추정 오류인지, Augmentation 때문에 발생한 문제인지는 아직 구분되지 않습니다.

따라서 지금 바로 `approved`로 처리하는 것은 적절하지 않습니다.

다음에는 원본 Skeleton과 보강 Skeleton을 같은 구간에서 비교하는 것이 중요합니다. 원본에서도 같은 문제가 발생한다면 보강 알고리즘보다 원본 Keypoint의 품질을 먼저 점검해야 합니다.

### ④ 이번 실습 결과 정리

```
Original 48개 + Augmented 120개
              ↓
        Automatic QA
              ↓
        168개 모두 PASS
              ↓
          Human QA
              ↓
     Skeleton 움직임 확인
              ↓
       문제 구간 추가 검사
              ↓
       approved / rejected
              ↓
   승인된 데이터만 모델 학습에 사용
```

**핵심:** 자동 검사에서 PASS가 나와도 움직임이 잘못될 수 있습니다. 
Human QA에서는 실제 동작을 확인하고 문제가 없는 데이터만 학습에 사용하도록 결정합니다.

---
# PART 5. 관절 움직임을 숫자로 요약하고 모델 성능을 비교합니다

앞에서는 Keypoint 데이터의 품질을 검사했습니다.

이번 PART에서는 **관절 움직임을 Random Forest가 학습할 수 있는 숫자 데이터(Feature)로 변환**합니다.

이후 원본 데이터만 학습한 모델과 보강 데이터를 추가로 학습한 모델의 성능을 비교하겠습니다.

---

# 26. 왜 Keypoint Sequence를 Feature로 변환할까요?

영상마다 길이가 다르기 때문입니다.

예를 들어 같은 권투 동작이라도 Frame 수가 다를 수 있습니다.

```
Original → 120 Frames
Fast     → 102 Frames
Slow     → 138 Frames
```

Random Forest는 모든 학습데이터의 입력 숫자 개수가 같아야 합니다.

따라서 영상이 몇 Frame이든 **관절의 움직임을 204개의 숫자로 요약**하겠습니다.

각 관절의 x, y 좌표에서 다음 여섯 가지 특징을 계산합니다.

|특징|쉽게 이해하기|
|---|---|
|`mean`|관절의 평균 위치|
|`std`|관절 위치가 얼마나 변했는지|
|`min`|가장 작은 좌표값|
|`max`|가장 큰 좌표값|
|`mean absolute velocity`|Frame 사이 평균 좌표 변화량|
|`max absolute velocity`|Frame 사이 가장 큰 좌표 변화량|

```
17개 관절
    ×
x, y 좌표 2개
    ×
통계 특징 6개
    ↓
총 204개 Feature
```

즉, 영상 길이가 달라도 모두 동일한 204개의 숫자로 변환되므로 Random Forest에 입력할 수 있습니다.

---

# 27. 관절 좌표가 누락되면 어떻게 처리할까요?

YOLO Pose가 일부 Frame에서 관절을 제대로 찾지 못할 수 있습니다.

이번에는 **Confidence 0.10 미만인 좌표를 신뢰하기 어려운 값으로 판단**하고, 주변의 정상 좌표를 이용해 보정합니다.

예를 들어 손목 좌표가 다음과 같다면:

```
Frame 1 → x = 0.40
Frame 2 → 좌표 누락
Frame 3 → x = 0.50
```

앞뒤 좌표를 이용하여 중간값을 추정합니다.

```
Frame 1 → x = 0.40
Frame 2 → x = 0.45 (보간)
Frame 3 → x = 0.50
```

이 방법을 **선형보간(Linear Interpolation)**이라고 합니다.

처음이나 마지막의 좌표가 누락되면 가장 가까운 유효 좌표를 사용하고, 해당 관절에 유효 좌표가 전혀 없으면 0으로 유지합니다.

단, 보간한 값은 실제 측정값이 아니라 계산으로 추정한 값입니다.

---

# 28. Sequence Feature Dataset 코드를 작성합니다

### 이번 코드는 무엇을 하나요?

**Automatic QA와 Human QA를 통과한 Keypoint JSON을 읽어 각 Sequence를 204개의 Feature로 변환하는 프로그램**입니다.

원본과 보강 데이터 모두 같은 방법으로 Feature를 계산합니다.

### 코드의 전체 처리 흐름

```
Original Keypoint JSON 48개
Augmented Keypoint JSON 120개
              ↓
          QA 결과 확인
              ↓
    ┌─────────┴─────────┐
    ↓                   ↓
 Original             Augmented
 auto_pass=True       auto_pass=True
                      +
                      Human QA approved
    ↓                   ↓
    └─────────┬─────────┘
              ↓
       Keypoint Sequence 읽기
              ↓
       누락된 관절 좌표 보간
              ↓
       관절별 6가지 통계 계산
              ↓
       204개 Feature 생성
              ↓
        Feature Dataset 저장
              ↓
     Baseline vs Augmented
          모델 성능 비교
```

**데이터 사용 규칙**

- Original Train / Val / Test: Automatic QA를 통과한 데이터 사용
    
- Augmented Train: Automatic QA를 통과하고 Human QA에서 승인된 방법의 데이터만 사용
    
- Validation / Test: 보강하지 않은 원본만 사용
    

**작성할 파일:** `**scripts/05_build_features.py**`

이번 실습의 핵심은 **길이가 서로 다른 관절 움직임을 동일한 204개 숫자로 변환하여, 보강 데이터가 동작인식 모델의 성능에 도움이 되는지 비교할 준비를 하는 것**입니다.

**파일: `scripts/05_build_features.py`**

```python
from __future__ import annotations

"""Day 05: Keypoint Sequence -> 204-dimensional feature CSV.

Usage:
  # Original baseline (does not require Human QA approval)
  python scripts/05_build_features.py --asset-set main --tag main_aug --baseline-only

  # Original + approved augmented sequences (requires Human QA approval)
  python scripts/05_build_features.py --asset-set main --tag main_aug
"""

import argparse
import json
import sys
from pathlib import Path

import numpy as np
import pandas as pd

ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import (  # noqa: E402
    augmented_keypoint_dir,
    feature_dir,
    original_keypoint_dir,
    report_dir,
)

JOINT_CONFIDENCE = 0.10
STAT_NAMES = (
    "mean",
    "std",
    "min",
    "max",
    "mean_abs_velocity",
    "max_abs_velocity",
)
REQUIRED_QA_COLUMNS = {"file", "split", "source_type", "method", "auto_pass"}


def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="Convert approved pose sequences into 204-feature datasets."
    )
    parser.add_argument(
        "--asset-set", required=True, choices=["main", "challenge"]
    )
    parser.add_argument("--tag", required=True)
    parser.add_argument(
        "--baseline-only",
        action="store_true",
        help="Build only Original Train/Validation/Test (no Human QA required).",
    )
    return parser.parse_args()


def load_qa(asset_set: str, tag: str) -> pd.DataFrame:
    path = report_dir(asset_set) / f"{tag}_qa.csv"
    if not path.is_file():
        raise FileNotFoundError(f"Automatic QA 파일이 없습니다: {path}")

    qa = pd.read_csv(path, encoding="utf-8-sig")
    missing = REQUIRED_QA_COLUMNS - set(qa.columns)
    if missing:
        raise RuntimeError(f"QA CSV 필수 열 누락: {', '.join(sorted(missing))}")
    return qa


def pass_mask(series: pd.Series) -> pd.Series:
    """Accept True/true/1/yes from CSV files."""
    return series.fillna("").astype(str).str.strip().str.lower().isin(
        ["true", "1", "yes"]
    )


def load_approved_methods(asset_set: str, tag: str) -> set[str]:
    path = report_dir(asset_set) / f"{tag}_human_qa.csv"
    if not path.is_file():
        raise FileNotFoundError(
            f"Human QA 파일이 없습니다: {path}\n"
            "영상을 확인하고 Human QA를 기록한 뒤 다시 실행하세요."
        )

    human = pd.read_csv(path, encoding="utf-8-sig")
    missing = {"method", "decision"} - set(human.columns)
    if missing:
        raise RuntimeError(f"Human QA CSV 필수 열 누락: {', '.join(sorted(missing))}")

    methods = human["method"].fillna("").astype(str).str.strip().str.lower()
    decisions = human["decision"].fillna("").astype(str).str.strip().str.lower()

    allowed_decisions = {"approved", "needs_review", "rejected"}
    invalid = sorted(set(decisions) - allowed_decisions)
    if invalid:
        raise RuntimeError(f"Human QA decision 값 오류: {invalid}")

    # Avoid accidentally approving one method while a duplicate row rejects it.
    records = pd.DataFrame({"method": methods, "decision": decisions})
    if records["method"].eq("").any():
        raise RuntimeError("Human QA에 비어 있는 method 값이 있습니다.")
    if records["method"].duplicated().any():
        raise RuntimeError("Human QA에 중복된 method가 있습니다. 방법별 한 행만 작성하세요.")

    approved = set(records.loc[records["decision"] == "approved", "method"])
    if not approved:
        raise RuntimeError(
            "approved로 결정된 Augmentation 방법이 없습니다.\n"
            "현재는 --baseline-only 옵션으로 Original Feature만 생성할 수 있습니다.\n"
            "보강 데이터는 Human QA에서 실제로 확인한 뒤 승인하세요."
        )
    return approved


def approved_original_files(qa: pd.DataFrame, split: str) -> set[str]:
    selection = (
        qa["source_type"].eq("original")
        & qa["split"].eq(split)
        & pass_mask(qa["auto_pass"])
    )
    return set(qa.loc[selection, "file"].astype(str))


def approved_augmented_files(
    qa: pd.DataFrame, approved_methods: set[str]
) -> set[str]:
    method = qa["method"].fillna("").astype(str).str.strip().str.lower()
    selection = (
        qa["source_type"].eq("augmented")
        & qa["split"].eq("train")
        & pass_mask(qa["auto_pass"])
        & method.isin(approved_methods)
    )
    return set(qa.loc[selection, "file"].astype(str))


def interpolate_xy(array: np.ndarray) -> np.ndarray:
    """Fill missing x/y using valid neighboring points; keep confidence unchanged."""
    result = array.copy()
    positions = np.arange(len(result))

    for joint in range(17):
        valid = result[:, joint, 2] >= JOINT_CONFIDENCE
        valid_positions = positions[valid]

        if len(valid_positions) == 0:
            result[:, joint, :2] = 0.0
            continue

        for coordinate in (0, 1):
            result[:, joint, coordinate] = np.interp(
                positions,
                valid_positions,
                result[valid, joint, coordinate],
            )
    return result


def feature_dict(data: dict) -> dict[str, float]:
    frames = data.get("frames", [])
    if not frames:
        raise RuntimeError("Frame이 없는 Keypoint JSON입니다.")

    try:
        array = np.asarray(
            [frame["keypoints"] for frame in frames], dtype=np.float32
        )
    except (KeyError, TypeError, ValueError) as exc:
        raise RuntimeError("Keypoint JSON을 배열로 변환할 수 없습니다.") from exc

    if array.ndim != 3 or array.shape[1:] != (17, 3):
        raise RuntimeError(f"Keypoint Shape 오류: {array.shape} (예상: [T,17,3])")
    if not np.isfinite(array).all():
        raise RuntimeError("Keypoint에 NaN 또는 Infinity가 포함되어 있습니다.")

    array = interpolate_xy(array)
    features: dict[str, float] = {}

    for joint in range(17):
        for coordinate, name in ((0, "x"), (1, "y")):
            values = array[:, joint, coordinate]
            velocity = np.diff(values)
            if velocity.size == 0:
                velocity = np.array([0.0], dtype=np.float32)

            statistics = {
                "mean": float(np.mean(values)),
                "std": float(np.std(values)),
                "min": float(np.min(values)),
                "max": float(np.max(values)),
                "mean_abs_velocity": float(np.mean(np.abs(velocity))),
                "max_abs_velocity": float(np.max(np.abs(velocity))),
            }
            for stat_name in STAT_NAMES:
                features[f"j{joint:02d}_{name}_{stat_name}"] = statistics[stat_name]

    return features


def rows_from_directory(
    directory: Path, allowed: set[str], source_type: str
) -> list[dict]:
    rows = []
    for path in sorted(directory.glob("*.json")):
        if path.name not in allowed:
            continue
        try:
            data = json.loads(path.read_text(encoding="utf-8"))
            row = {
                "file": path.name,
                "subject": data["subject"],
                "action": data["action"],
                "source_type": source_type,
            }
            row.update(feature_dict(data))
        except (KeyError, ValueError, TypeError, RuntimeError) as exc:
            raise RuntimeError(f"Feature 생성 실패 ({path}): {exc}") from exc
        rows.append(row)
    return rows


def save_rows(rows: list[dict], path: Path) -> None:
    if not rows:
        raise RuntimeError(f"저장할 Feature 데이터가 없습니다: {path.name}")
    path.parent.mkdir(parents=True, exist_ok=True)
    pd.DataFrame(rows).to_csv(path, index=False, encoding="utf-8-sig")


def main() -> None:
    args = parse_args()
    qa = load_qa(args.asset_set, args.tag)
    output_dir = feature_dir(args.asset_set, args.tag)

    # Always select original sequences from Automatic QA PASS rows.
    original_rows: dict[str, list[dict]] = {}
    for split in ("train", "val", "test"):
        allowed = approved_original_files(qa, split)
        original_rows[split] = rows_from_directory(
            original_keypoint_dir(args.asset_set, split),
            allowed,
            "original",
        )

    for split in ("train", "val", "test"):
        if not original_rows[split]:
            raise RuntimeError(f"{split}에 Automatic QA를 통과한 원본 JSON이 없습니다.")

    if args.baseline_only:
        # Do not load Human QA or augmented data in this mode.
        for split in ("train", "val", "test"):
            save_rows(original_rows[split], output_dir / f"{split}_original.csv")

        print("=" * 60)
        print("Original Baseline Features (Human QA 불필요)")
        print("=" * 60)
        print("Train Original :", len(original_rows["train"]))
        print("Validation     :", len(original_rows["val"]))
        print("Test           :", len(original_rows["test"]))
        print("Feature Dir    :", output_dir)
        print("Augmented Feature: 생성하지 않음 (Human QA 보류)")
        return

    # Full comparison mode: only approved augmentation methods are eligible.
    approved_methods = load_approved_methods(args.asset_set, args.tag)
    allowed_augmented = approved_augmented_files(qa, approved_methods)
    aug_rows = rows_from_directory(
        augmented_keypoint_dir(args.asset_set, args.tag),
        allowed_augmented,
        "augmented",
    )
    if not aug_rows:
        raise RuntimeError(
            "Automatic QA와 Human QA를 모두 통과한 보강 JSON이 없습니다."
        )

    train_original = original_rows["train"]
    train_with_aug = train_original + aug_rows

    save_rows(train_original, output_dir / "train_original.csv")
    save_rows(train_with_aug, output_dir / "train_with_aug.csv")
    save_rows(original_rows["val"], output_dir / "val_original.csv")
    save_rows(original_rows["test"], output_dir / "test_original.csv")

    print("=" * 60)
    print("Sequence Feature Dataset")
    print("=" * 60)
    print("Approved Methods:", sorted(approved_methods))
    print("Train Original :", len(train_original))
    print("Approved Aug   :", len(aug_rows))
    print("Train + Aug    :", len(train_with_aug))
    print("Validation     :", len(original_rows["val"]))
    print("Test           :", len(original_rows["test"]))
    print("Feature Dir    :", output_dir)


if __name__ == "__main__":
    main()

```

### 의사코드로 읽어보기

```text
[프로그램 시작]
        ↓
① 실행 옵션 확인 — parse_args()
   │
   ├─ --asset-set main
   │   → Main 데이터 사용
   │
   ├─ --tag main_aug
   │   → 보강 실험 이름
   │
   └─ --baseline-only
       → 원본 데이터만 처리할지 결정
        ↓
② Automatic QA 결과 읽기 — load_qa()
   │
   └─ reports/main/main_aug_qa.csv
        ↓
   QA 파일과 필수 열이 존재하는가?
   │
   ├─ 아니요 → 오류 발생 및 중단
   └─ 예 → QA 결과 불러오기
        ↓
③ 원본 Keypoint JSON 선택
   │
   ├─ Train → auto_pass=True인 원본 선택
   ├─ Val   → auto_pass=True인 원본 선택
   └─ Test  → auto_pass=True인 원본 선택
        ↓
   현재 Main 실습의 실제 QA 결과
   │
   ├─ Train 24개 PASS
   ├─ Val   12개 PASS
   └─ Test  12개 PASS
        ↓
④ 선택된 JSON을 Feature로 변환
   — rows_from_directory() / feature_dict()
   │
   ├─ JSON 파일 하나 읽기
   │       ↓
   │   Frame별 Keypoint 가져오기
   │       ↓
   │   [Frame 수, 17개 관절, 3개 값] 배열 생성
   │       ↓
   │   배열 형태와 NaN / Infinity 검사
   │       ↓
   │   올바르지 않으면 오류 발생 및 중단
   │       ↓
   │   누락된 좌표 보정 — interpolate_xy()
   │       │
   │       ├─ Confidence ≥ 0.10
   │       │   → 정상 좌표 사용
   │       │
   │       ├─ 중간에 좌표가 누락됨
   │       │   → 앞뒤 좌표로 선형보간
   │       │
   │       ├─ 처음이나 끝에 좌표가 누락됨
   │       │   → 가장 가까운 정상 좌표 사용
   │       │
   │       └─ 해당 관절의 정상 좌표가 전혀 없음
   │           → x, y를 0으로 유지
   │       ↓
   │   관절 17개에 대해 통계값 계산
   │       │
   │       ├─ x 좌표
   │       │   ├─ mean   : 평균
   │       │   ├─ std    : 표준편차
   │       │   ├─ min    : 최솟값
   │       │   ├─ max    : 최댓값
   │       │   ├─ mean_abs_velocity
   │       │   │   → Frame 사이 평균 절대 변화량
   │       │   └─ max_abs_velocity
   │       │       → Frame 사이 최대 절대 변화량
   │       │
   │       └─ y 좌표도 동일하게 계산
   │       ↓
   │   17개 관절 × 2개 좌표 × 6개 통계
   │                = 204 Features
   │       ↓
   │   JSON 하나를 CSV의 한 행으로 변환
   │       │
   │       ├─ file        : 파일명
   │       ├─ subject     : 촬영 대상자
   │       ├─ action      : 동작 정답
   │       ├─ source_type : original / augmented
   │       └─ 204개 Feature
   │
   └─ 선택된 모든 JSON에 반복 적용
        ↓
⑤ --baseline-only 옵션이 있는가?
   │
   ├─ YES (원본 Baseline 모드)
   │    │
   │    ├─ Human QA 확인하지 않음
   │    ├─ Augmented JSON 사용하지 않음
   │    │
   │    ├─ train_original.csv 저장
   │    │   → 원본 Train 24개
   │    │
   │    ├─ val_original.csv 저장
   │    │   → 원본 Validation 12개
   │    │
   │    └─ test_original.csv 저장
   │        → 원본 Test 12개
   │             ↓
   │       생성 결과 출력
   │             ↓
   │       [프로그램 종료]
   │
   └─ NO (원본 + 보강 모드)
        ↓
⑥ Human QA 결과 읽기 — load_approved_methods()
   │
   └─ reports/main/main_aug_human_qa.csv
        ↓
   Human QA 파일이 존재하는가?
   │
   ├─ 아니요 → 오류 발생 및 중단
   └─ 예 → 파일 읽기
        ↓
   method / decision 값 확인
        ↓
   approved된 보강 방법이 있는가?
   │
   ├─ 아니요
   │   → 오류 발생 및 중단
   │   → Human QA 완료 후 다시 실행
   │
   └─ 예
       → 승인된 방법 목록 저장
        ↓
⑦ 보강 Keypoint JSON 선택
   — approved_augmented_files()
   │
   ├─ Augmented Train 데이터인가?
   ├─ Automatic QA에서 PASS인가?
   └─ Human QA에서 승인된 방법인가?
        ↓
   세 조건을 모두 만족하는가?
   │
   ├─ 아니요 → 해당 JSON 제외
   └─ 예 → Feature 생성 대상으로 선택
        ↓
⑧ 승인된 Augmented JSON도 Feature 변환
   │
   └─ ④와 동일한 함수 사용
       │
       ├─ Keypoint 읽기
       ├─ 누락 좌표 보간
       ├─ 204개 Feature 계산
       └─ CSV 행 생성
        ↓
⑨ 학습데이터 두 종류 구성
   │
   ├─ Original Baseline
   │   → 원본 Train Feature 24개
   │
   └─ Original + Augmented
       → 원본 Train 24개
         +
         승인된 보강 Feature
        ↓
⑩ CSV 파일 저장 — save_rows()
   │
   ├─ train_original.csv
   │   → 원본 Train만 저장
   │
   ├─ train_with_aug.csv
   │   → 원본 Train + 승인된 보강 데이터
   │
   ├─ val_original.csv
   │   → 원본 Validation 저장
   │
   └─ test_original.csv
       → 원본 Test 저장
        ↓
⑪ 최종 생성 결과 출력
   │
   ├─ Approved Methods
   ├─ Train Original
   ├─ Approved Aug
   ├─ Train + Aug
   ├─ Validation
   ├─ Test
   └─ Feature 저장 위치
        ↓
[프로그램 종료]


────────────────────────────────────────
[다음 Random Forest 수업과 연결]
────────────────────────────────────────

train_original.csv
        ↓
  Random Forest A
   (원본만 학습)
        │
        │
train_with_aug.csv
        ↓
  Random Forest B
  (원본 + 보강 학습)
        │
        │
        └───────────┐
                    ↓
         동일한 val_original.csv
             모델 성능 비교
                    ↓
         보강 데이터가 성능 향상에
             도움이 되었는가?
                    ↓
          모델 선택 및 Test 평가
```

---
# 29. Main Feature Dataset을 생성합니다

앞에서 만든 Keypoint를 Random Forest가 학습할 수 있는 **204개의 숫자 Feature**로 변환하겠습니다.

현재 Human QA에서는 보강 데이터의 움직임을 추가로 확인해야 하므로, 먼저 원본 데이터만 처리합니다.

### ① 원본 Feature 생성하기

Automatic QA 결과 파일 `reports/main/main_aug_qa.csv`가 있는지 확인한 뒤 실행합니다.

```
python scripts/05_build_features.py \
  --asset-set main \
  --tag main_aug \
  --baseline-only
```

### ② 실제 실행 결과 확인하기

```
Original Baseline Features (Human QA 불필요)
============================================================
Train Original : 24
Validation     : 12
Test           : 12
Feature Dir    : .../data/features/main/main_aug
Augmented Feature: 생성하지 않음 (Human QA 보류)
```

원본 48개가 정상적으로 Feature로 변환되었습니다.

생성된 파일은 다음과 같습니다.

|파일|데이터 수|용도|
|---|---|---|
|`train_original.csv`|24개|모델 학습|
|`val_original.csv`|12개|모델 성능 비교|
|`test_original.csv`|12개|최종 성능 평가|

저장 위치: `data/features/main/main_aug/`

현재는 `--baseline-only`를 사용했으므로 Human QA 승인 없이 실행되며, **보강 데이터가 포함된** `**train_with_aug.csv**`**는 아직 생성되지 않았습니다.**

---

# 30. Baseline과 Augmented 모델의 차이를 이해합니다

이번 실습에서는 보강 데이터를 추가했을 때 AI 모델의 성능이 좋아지는지 비교합니다.

|구분|Baseline 모델|Augmented 모델|
|---|---|---|
|학습 데이터|원본 24개|원본 24개 + 승인된 보강 데이터|
|알고리즘|Random Forest|Random Forest|
|Validation|동일한 원본 12개|동일한 원본 12개|
|Test|동일한 원본 12개|동일한 원본 12개|

```
[Baseline]
Original Train 24개
        ↓
   Random Forest A
        ↓
   Validation 평가


[Augmented]
Original Train 24개
        +
승인된 Augmented Train
        ↓
   Random Forest B
        ↓
   동일한 Validation 평가

        ↓
    두 모델 성능 비교
        ↓
    최종 Test 평가
```

두 모델은 같은 알고리즘과 평가 데이터를 사용합니다.

**차이점은 학습할 때 보강 데이터를 추가했는지 여부입니다.**

따라서 성능 차이를 통해 보강 데이터가 도움이 되었는지 확인할 수 있습니다.

단, Test 데이터는 최종 평가용입니다. Test 결과를 보고 보강 설정을 다시 변경하지 않습니다.

---

# 31. Baseline vs Augmented 성능 비교 코드를 작성합니다

### 이번 코드는 무엇을 하나요?

원본만 학습한 Random Forest와 보강 데이터를 추가로 학습한 Random Forest의 성능을 비교하는 프로그램입니다.

다만 현재는 Human QA에서 승인된 보강 방법이 없으므로 
**Baseline 모델은 준비할 수 있지만 두 모델의 성능 비교는 아직 진행할 수 없습니다.**

보강 데이터의 품질을 확인하여 사용 가능한 방법을 `approved`로 변경한 뒤, `05_build_features.py`를 `--baseline-only` 없이 실행해 `train_with_aug.csv`를 생성해야 합니다.

**이번 단계의 핵심:** 원본 Feature 48개 생성은 완료되었습니다. 다음에는 Random Forest Baseline을 학습하고, Human QA 승인 후 보강 데이터를 추가한 모델과 성능을 비교하겠습니다.

**파일: `scripts/06_compare_baseline_augmented.py`**

```python
from __future__ import annotations

"""Day 05: Random Forest Baseline vs approved Augmented training.

Original baseline only (no Human QA approval required):
    python scripts/06_compare_baseline_augmented.py --asset-set main --tag main_aug --eval-split both --baseline-only

Automatic selection (runs baseline only if train_with_aug.csv is missing):
    python scripts/06_compare_baseline_augmented.py --asset-set main --tag main_aug --eval-split both

When train_with_aug.csv is present and approved by Human QA, compare both models.
"""

import argparse
import json
import sys
from pathlib import Path

import joblib
import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix

ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day05_common import feature_dir, read_classes, report_dir  # noqa: E402

SEED = 42
LABELS = ["boxing", "handwaving", "handclapping"]
VALID_DECISIONS = {"approved", "needs_review", "rejected"}


def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="Evaluate Random Forest baseline, optionally comparing approved augmentation."
    )
    parser.add_argument("--asset-set", required=True, choices=["main", "challenge"])
    parser.add_argument("--tag", required=True)
    parser.add_argument(
        "--eval-split",
        choices=["val", "test", "both"],
        default="both",
        help="val: 후보 비교 / test: 조건 확정 후 최종평가 / both: 최종 보고",
    )
    parser.add_argument(
        "--baseline-only",
        action="store_true",
        help="Human QA/보강 Train 없이 Original Baseline 모델만 평가합니다.",
    )
    return parser.parse_args()


def load_xy(path: Path) -> tuple[pd.DataFrame, np.ndarray, np.ndarray, list[str]]:
    if not path.is_file():
        raise FileNotFoundError(
            f"Feature CSV 파일이 없습니다: {path}\n"
            "먼저 scripts/05_build_features.py를 실행하세요."
        )

    dataframe = pd.read_csv(path, encoding="utf-8-sig")
    if dataframe.empty:
        raise RuntimeError(f"Feature CSV에 데이터가 없습니다: {path}")

    if "action" not in dataframe.columns:
        raise RuntimeError(f"action 열이 없습니다: {path}")

    feature_columns = [col for col in dataframe.columns if col.startswith("j")]
    if len(feature_columns) != 204:
        raise RuntimeError(
            f"Feature 개수 오류 ({path.name}): {len(feature_columns)}개 (예상: 204개)"
        )

    try:
        x = dataframe[feature_columns].apply(pd.to_numeric, errors="raise").to_numpy(
            dtype=np.float64
        )
    except (ValueError, TypeError) as exc:
        raise RuntimeError(f"Feature에 숫자가 아닌 값이 있습니다: {path}") from exc

    if not np.isfinite(x).all():
        raise RuntimeError(f"Feature에 NaN 또는 Infinity가 있습니다: {path}")

    y = dataframe["action"].astype(str).str.strip().to_numpy()
    if not set(y).issubset(set(LABELS)):
        raise RuntimeError(f"알 수 없는 Action Label이 있습니다: {path}")

    return dataframe, x, y, feature_columns


def make_model() -> RandomForestClassifier:
    # 원래 수업 코드의 동일한 모델 설정을 유지합니다.
    return RandomForestClassifier(
        n_estimators=300,
        random_state=SEED,
        class_weight="balanced",
        n_jobs=-1,
    )


def approved_methods(asset_set: str, tag: str) -> set[str]:
    """Return only explicitly approved methods; no approval means no augmentation."""
    path = report_dir(asset_set) / f"{tag}_human_qa.csv"
    if not path.is_file():
        print(f"[보류] Human QA 파일이 없습니다: {path}")
        return set()

    human = pd.read_csv(path, encoding="utf-8-sig")
    if not {"method", "decision"}.issubset(human.columns):
        raise RuntimeError(f"Human QA의 method/decision 열이 없습니다: {path}")

    methods = human["method"].fillna("").astype(str).str.strip().str.lower()
    decisions = human["decision"].fillna("").astype(str).str.strip().str.lower()
    if methods.eq("").any() or methods.duplicated().any():
        raise RuntimeError("Human QA의 method가 비어 있거나 중복되었습니다.")
    if not set(decisions).issubset(VALID_DECISIONS):
        raise RuntimeError("Human QA decision은 approved/needs_review/rejected만 허용됩니다.")
    return set(methods[decisions == "approved"])


def check_augmented_qa(
    dataframe: pd.DataFrame, approved: set[str], asset_set: str, tag: str
) -> None:
    """Do not train on a stale CSV created before a Human QA decision changed."""
    required = {"file", "source_type"}
    if not required.issubset(dataframe.columns):
        raise RuntimeError("train_with_aug.csv에 file/source_type 열이 없습니다.")

    augmented = dataframe[dataframe["source_type"].eq("augmented")]
    if augmented.empty:
        raise RuntimeError("train_with_aug.csv에 보강 데이터가 없습니다.")

    methods_in_csv: set[str] = set()
    for filename in augmented["file"].astype(str):
        stem = Path(filename).stem
        if "__" not in stem:
            raise RuntimeError(f"Augmented 파일 이름의 method를 확인할 수 없습니다: {filename}")
        methods_in_csv.add(stem.rsplit("__", 1)[1].lower())

    unapproved = methods_in_csv - approved
    if unapproved:
        raise RuntimeError(
            f"이전에 만들어진 train_with_aug.csv에 현재 승인되지 않은 방법이 있습니다: {sorted(unapproved)}\n"
            "Human QA를 재확인한 뒤 05_build_features.py를 다시 실행하세요."
        )

    # Verify individual files were Automatic QA PASS too, even for a previously created CSV.
    qa_path = report_dir(asset_set) / f"{tag}_qa.csv"
    if not qa_path.is_file():
        raise FileNotFoundError(f"Automatic QA 보고서가 없습니다: {qa_path}")
    qa = pd.read_csv(qa_path, encoding="utf-8-sig")
    needed = {"file", "auto_pass", "source_type", "split", "method"}
    if not needed.issubset(qa.columns):
        raise RuntimeError("Automatic QA 보고서에 필요한 열이 없습니다.")
    mask = (
        qa["source_type"].eq("augmented")
        & qa["split"].eq("train")
        & qa["auto_pass"].astype(str).str.strip().str.lower().isin(["true", "1", "yes"])
        & qa["method"].astype(str).str.strip().str.lower().isin(approved)
    )
    passed_names = set(qa.loc[mask, "file"].astype(str))
    not_passed = set(augmented["file"].astype(str)) - passed_names
    if not_passed:
        raise RuntimeError(
            f"train_with_aug.csv에 Automatic QA 통과 기록이 없는 파일이 있습니다: {sorted(not_passed)[:5]}"
        )


def evaluate(model: RandomForestClassifier, x: np.ndarray, y: np.ndarray):
    predicted = model.predict(x).tolist()
    return float(accuracy_score(y, predicted)), predicted


def save_confusion(
    output_dir: Path,
    tag: str,
    model_name: str,
    split: str,
    y_true: np.ndarray,
    predicted: list[str],
    *,
    legacy_test_name: bool,
) -> Path:
    filename = (
        f"{tag}_{model_name}_confusion.csv"
        if legacy_test_name
        else f"{tag}_{model_name}_{split}_confusion.csv"
    )
    path = output_dir / filename
    matrix = confusion_matrix(y_true, predicted, labels=LABELS)
    pd.DataFrame(matrix, index=LABELS, columns=LABELS).to_csv(
        path, encoding="utf-8-sig"
    )
    return path


def main() -> None:
    args = parse_args()
    input_dir = feature_dir(args.asset_set, args.tag)
    output_dir = report_dir(args.asset_set)
    output_dir.mkdir(parents=True, exist_ok=True)

    original_df, x_train_original, y_train_original, feature_columns = load_xy(
        input_dir / "train_original.csv"
    )

    class_values = set(read_classes().values())
    if class_values != set(LABELS):
        raise RuntimeError("classes.txt와 교재 Action Label 구성이 다릅니다.")
    if set(y_train_original) != class_values:
        raise RuntimeError("Original Train에 세 가지 Action이 모두 포함되어야 합니다.")

    augmented_path = input_dir / "train_with_aug.csv"
    comparison = not args.baseline_only and augmented_path.is_file()

    if args.baseline_only:
        print("[Baseline 모드] --baseline-only 옵션: 원본 모델만 평가합니다.")
    elif not augmented_path.is_file():
        print("[Baseline 모드] train_with_aug.csv가 아직 없습니다.")
        print("[안내] 보강 모델 성능은 비교하지 않으며, 원본 Baseline만 평가합니다.")
        print("       Human QA 승인 후 05_build_features.py를 일반 모드로 실행하세요.")
    else:
        methods = approved_methods(args.asset_set, args.tag)
        if not methods:
            comparison = False
            print("[Baseline 모드] Human QA의 approved 방법이 없습니다.")
            print("[안내] 기존 보강 파일이 있어도 사용하지 않습니다.")
        else:
            print("Approved Methods:", sorted(methods))

    augmented_df = None
    augmented_model = None
    if comparison:
        augmented_df, x_train_aug, y_train_aug, augmented_cols = load_xy(augmented_path)
        if feature_columns != augmented_cols:
            raise RuntimeError("Original과 Augmented Train의 Feature 열이 다릅니다.")
        if set(y_train_aug) != class_values:
            raise RuntimeError("Augmented Train에 세 가지 Action이 모두 포함되어야 합니다.")
        check_augmented_qa(augmented_df, methods, args.asset_set, args.tag)

        augmented_model = make_model()
        augmented_model.fit(x_train_aug, y_train_aug)

    baseline_model = make_model()
    baseline_model.fit(x_train_original, y_train_original)

    splits = ["val", "test"] if args.eval_split == "both" else [args.eval_split]
    scores: dict[str, dict[str, float]] = {"baseline": {}}
    reports: dict[str, dict] = {}
    if comparison:
        scores["augmented"] = {}

    for split in splits:
        _, x_eval, y_eval, eval_columns = load_xy(input_dir / f"{split}_original.csv")
        if eval_columns != feature_columns:
            raise RuntimeError(f"{split} Feature 열이 Train과 다릅니다.")
        if set(y_eval) != class_values:
            raise RuntimeError(f"{split} 데이터에 세 가지 Action이 모두 필요합니다.")

        for name, model in (
            [("baseline", baseline_model)]
            + ([("augmented", augmented_model)] if comparison else [])
        ):
            score, predicted = evaluate(model, x_eval, y_eval)
            scores[name][split] = score
            reports[f"{name}_{split}"] = classification_report(
                y_eval, predicted, labels=LABELS, output_dict=True, zero_division=0
            )
            save_confusion(
                output_dir,
                args.tag,
                name,
                split,
                y_eval,
                predicted,
                legacy_test_name=(args.eval_split == "both" and split == "test"),
            )

    # Keep the original full-comparison filenames. Baseline-only has its own filenames.
    mode_prefix = "" if comparison else "baseline_"
    if args.eval_split == "both":
        summary_path = output_dir / f"{args.tag}_{mode_prefix}performance.csv"
        report_path = output_dir / f"{args.tag}_{mode_prefix}classification_report.json"
    else:
        summary_path = output_dir / f"{args.tag}_{mode_prefix}performance_{args.eval_split}.csv"
        report_path = (
            output_dir
            / f"{args.tag}_{mode_prefix}{args.eval_split}_classification_report.json"
        )

    rows = []
    for name in (["baseline", "augmented"] if comparison else ["baseline"]):
        train_count = len(original_df) if name == "baseline" else len(augmented_df)
        row = {"model": name, "train_count": train_count}
        if args.eval_split == "both":
            row.update(
                val_accuracy=scores[name]["val"],
                test_accuracy=scores[name]["test"],
            )
        else:
            row.update(eval_split=args.eval_split, accuracy=scores[name][args.eval_split])
        rows.append(row)

    pd.DataFrame(rows).to_csv(summary_path, index=False, encoding="utf-8-sig")
    report_path.write_text(
        json.dumps(reports, ensure_ascii=False, indent=2), encoding="utf-8"
    )

    model_dir = ROOT / "models" / args.asset_set / args.tag
    model_dir.mkdir(parents=True, exist_ok=True)
    joblib.dump(baseline_model, model_dir / "baseline.joblib")
    if comparison:
        joblib.dump(augmented_model, model_dir / "augmented.joblib")

    print("=" * 68)
    print("Baseline vs Augmented" if comparison else "Original Baseline Only")
    print("=" * 68)
    print("Eval Split     :", args.eval_split)
    print("Train Original :", len(original_df))
    if comparison:
        print("Train + Aug    :", len(augmented_df))
    for split in splits:
        baseline_acc = scores["baseline"][split]
        if comparison:
            aug_acc = scores["augmented"][split]
            print(
                f"{split.upper():4s} Baseline={baseline_acc:.4f} "
                f"Augmented={aug_acc:.4f} Delta={aug_acc-baseline_acc:+.4f}"
            )
        else:
            print(f"{split.upper():4s} Baseline={baseline_acc:.4f} (Augmented: 보류)")
    print("Saved          :", summary_path)
    print("Report         :", report_path)
    print("Model          :", model_dir / "baseline.joblib")
    if not comparison:
        print("[주의] 보강 효과 비교는 진행되지 않았습니다.")
    if "test" in splits:
        print("[주의] Test 결과를 보고 보강 파라미터를 변경하지 마세요.")


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[프로그램 시작]
        ↓
① 필요한 라이브러리 불러오기
   │
   ├─ Pandas       → Feature CSV 읽기·결과 저장
   ├─ NumPy        → 학습 데이터 배열 처리
   ├─ RandomForest → 동작 분류 모델
   ├─ Scikit-learn → Accuracy·Confusion Matrix 계산
   └─ Joblib       → 학습한 모델 저장
        ↓
② 실행 옵션 확인 — parse_args()
   │
   ├─ --asset-set main
   │   → Main 데이터 사용
   │
   ├─ --tag main_aug
   │   → 현재 실험 이름
   │
   ├─ --eval-split both
   │   → Validation과 Test 모두 평가
   │
   └─ --baseline-only (선택)
       → 원본 모델만 평가
        ↓
③ 원본 Train Feature 불러오기 — load_xy()
   │
   └─ train_original.csv
        ↓
   파일이 존재하는가?
   │
   ├─ 아니요 → 오류 발생 및 중단
   └─ 예 → 파일 읽기
        ↓
④ 학습 데이터 검사
   │
   ├─ 데이터가 비어 있지 않은가?
   ├─ action 정답 열이 존재하는가?
   ├─ Feature가 정확히 204개인가?
   ├─ 모든 Feature가 정상적인 숫자인가?
   ├─ NaN / Infinity가 없는가?
   └─ Action Label이 올바른가?
        ↓
   검사 결과가 정상인가?
   │
   ├─ 아니요 → 오류 발생 및 중단
   └─ 예 → 학습 데이터 준비
        ↓
⑤ Action 종류 확인
   │
   ├─ boxing       → 권투
   ├─ handwaving   → 손 흔들기
   └─ handclapping → 박수
        ↓
   Original Train에 세 동작이 모두 있는가?
   │
   ├─ 아니요 → 오류 발생 및 중단
   └─ 예 → 다음 단계
        ↓
⑥ Augmented Train 사용 가능 여부 확인
   │
   ├─ --baseline-only 옵션이 있는가?
   │   │
   │   └─ YES → 원본 Baseline만 평가
   │
   └─ NO
       ↓
   train_with_aug.csv가 존재하는가?
   │
   ├─ 아니요
   │   → Original Baseline 모드 선택
   │
   └─ 예
       ↓
   Human QA에서 approved된 방법이 있는가?
   │
   ├─ 아니요
   │   → 보강 데이터 사용하지 않음
   │   → Original Baseline 모드 선택
   │
   └─ 예
       ↓
⑦ 보강 학습데이터 검사 — check_augmented_qa()
   │
   ├─ train_with_aug.csv 읽기
   ├─ Feature가 원본과 동일한 204개인가?
   ├─ 승인된 보강 방법만 포함되어 있는가?
   ├─ 각 파일이 Automatic QA를 통과했는가?
   └─ 세 가지 Action이 모두 포함되어 있는가?
        ↓
   모든 조건을 만족하는가?
   │
   ├─ 아니요 → 오류 발생 및 중단
   └─ 예 → Baseline vs Augmented 비교 모드
        ↓
⑧ Random Forest 모델 준비 — make_model()
   │
   ├─ n_estimators = 300
   │   → Decision Tree 300개 사용
   │
   ├─ random_state = 42
   │   → 실험 재현을 위한 난수 설정
   │
   ├─ class_weight = balanced
   │   → 동작별 데이터 수 차이를 고려
   │
   └─ n_jobs = -1
       → 사용 가능한 CPU 코어 활용
        ↓
⑨ Random Forest 학습
   │
   ├─ [Baseline 모델 A]
   │   │
   │   ├─ Original Train 24개
   │   └─ Random Forest 학습
   │
   └─ [Augmented 모델 B — 승인된 경우만]
       │
       ├─ Original Train 24개
       ├─ 승인된 Augmented Train 추가
       └─ 동일한 설정의 Random Forest 학습
        ↓
⑩ 평가 데이터 선택
   │
   ├─ --eval-split val
   │   → val_original.csv
   │
   ├─ --eval-split test
   │   → test_original.csv
   │
   └─ --eval-split both
       → Validation + Test 모두 사용
        ↓
⑪ 평가 데이터 검사
   │
   ├─ Feature가 204개인가?
   ├─ Train과 Feature 열 순서가 같은가?
   └─ 세 가지 Action이 모두 포함되어 있는가?
        ↓
   검사 결과가 정상인가?
   │
   ├─ 아니요 → 오류 발생 및 중단
   └─ 예 → 모델 성능 평가
        ↓
⑫ 모델 예측 — evaluate()
   │
   ├─ 학습한 Random Forest에
   │   Validation 또는 Test Feature 입력
   │
   ├─ 모델이 Action 예측
   │   → boxing / handwaving / handclapping
   │
   └─ 실제 정답과 예측 결과 비교
        ↓
⑬ 모델 성능 계산
   │
   ├─ Accuracy
   │   → 전체 중 정답을 맞힌 비율
   │
   ├─ Confusion Matrix
   │   → 어떤 동작을 다른 동작으로
   │     잘못 예측했는지 확인
   │
   └─ Classification Report
       ├─ Precision → 정밀도
       ├─ Recall    → 재현율
       └─ F1-score  → 정밀도·재현율 종합
        ↓
⑭ Augmented 모델도 평가했는가?
   │
   ├─ 아니요
   │   → Baseline 성능만 기록
   │
   └─ 예
       → 동일한 Validation / Test로 평가
       → 두 모델 Accuracy 비교
       → Delta = Augmented - Baseline
        ↓
⑮ 성능 결과 CSV / JSON 저장
   │
   ├─ Baseline 전용 모드
   │   │
   │   ├─ main_aug_baseline_performance.csv
   │   ├─ main_aug_baseline_val_confusion.csv
   │   ├─ main_aug_baseline_confusion.csv
   │   └─ main_aug_baseline_classification_report.json
   │
   └─ Baseline vs Augmented 비교 모드
       │
       ├─ main_aug_performance.csv
       ├─ main_aug_baseline_val_confusion.csv
       ├─ main_aug_augmented_val_confusion.csv
       ├─ main_aug_baseline_confusion.csv
       ├─ main_aug_augmented_confusion.csv
       └─ main_aug_classification_report.json

   ※ 위 파일명은 --eval-split both 기준
        ↓
⑯ 학습된 모델 저장 — joblib.dump()
   │
   └─ models/main/main_aug/
       │
       ├─ baseline.joblib
       │   → 원본으로 학습한 모델
       │
       └─ augmented.joblib
           → 보강 데이터가 승인되어
             비교 학습을 실행했을 때만 저장
        ↓
⑰ 최종 성능 결과 출력
   │
   ├─ Baseline 전용 실행이라면
   │   │
   │   ├─ Original Train 수
   │   ├─ Validation Accuracy
   │   ├─ Test Accuracy
   │   └─ Augmented 비교 보류 안내
   │
   └─ Augmented 비교까지 진행했다면
       │
       ├─ Original Train 수
       ├─ Train + Aug 데이터 수
       ├─ Baseline Accuracy
       ├─ Augmented Accuracy
       └─ 두 모델의 Accuracy 차이
        ↓
[프로그램 종료]


─────────────────────────────────────────
[현재 실제 실행 결과]
─────────────────────────────────────────

Original Train 24개
        ↓
204개 Feature
        ↓
Random Forest Baseline 학습
        ↓
  ┌──────────────┴──────────────┐
  ↓                             ↓
Validation 12개             Test 12개
  ↓                             ↓
10개 정답                     9개 정답
  ↓                             ↓
Accuracy 83.33%              Accuracy 75.00%
  │                             │
  └──────────────┬──────────────┘
                 ↓
          Baseline 결과 저장
                 ↓
         baseline.joblib 저장
                 ↓
      Augmented 모델 비교는 보류
      (Human QA 승인 방법 없음)


─────────────────────────────────────────
[이후 Augmented 데이터 승인 시]
─────────────────────────────────────────

           Human QA 승인 완료
                  ↓
       승인된 Augmented Feature 생성
                  ↓
             모델 학습
        ┌─────────┴─────────┐
        ↓                   ↓
   Baseline 모델       Augmented 모델
   Original Train      Original Train
                       + Approved Aug
        ↓                   ↓
        └─────────┬─────────┘
                  ↓
          동일한 Validation 평가
                  ↓
          두 모델 성능 비교
                  ↓
        최종 모델 및 설정 확정
                  ↓
             Test 최종 평가
                  ↓
         보강 효과 최종 결과 정리
```

---
# 32. Main 모델의 성능을 검증합니다

앞에서 원본 Keypoint 24개를 204개 Feature로 변환했습니다.

이번에는 이 데이터로 **Random Forest를 학습하고, 처음 보는 동작도 올바르게 구분하는지 확인**하겠습니다.

### ① 성능 평가 실행하기

```bash
python scripts/06_compare_baseline_augmented.py \
  --asset-set main \
  --tag main_aug \
  --eval-split both
```

`--eval-split both`는 Validation과 Test를 모두 평가한다는 의미입니다.

### ② 실제 실행 결과 확인하기

|평가 항목|실제 Accuracy|맞힌 데이터|
|---|---|---|
|Validation|83.33%|12개 중 10개|
|Test|75.00%|12개 중 9개|

현재는 Human QA에서 승인된 보강 데이터가 없으므로 **원본 데이터만 학습한 Baseline 모델의 성능**입니다.

즉, 처음 보는 12개의 Test 동작 중 9개를 올바르게 분류했고, 3개는 잘못 분류했습니다.

### ③ 생성된 결과 파일 확인하기

```
reports/main/
├─ main_aug_baseline_performance.csv
├─ main_aug_baseline_val_confusion.csv
├─ main_aug_baseline_confusion.csv
└─ main_aug_baseline_classification_report.json

models/main/main_aug/
└─ baseline.joblib
```

각 파일의 역할은 다음과 같습니다.

- `performance.csv`: 모델의 Accuracy 확인
    
- `confusion.csv`: Boxing, Handwaving, Handclapping 중 어떤 동작을 잘못 구분했는지 확인
    
- `classification_report.json`: 동작별 Precision, Recall, F1-score 확인
    
- `baseline.joblib`: 학습이 완료된 Random Forest 모델
    

현재는 보강 모델을 학습하지 않았으므로 `augmented.joblib`과 보강 모델의 평가 결과는 새로 생성되지 않습니다.

---

# 33. 실제 모델 성능을 해석합니다

### ① 현재 Baseline 모델의 성능은 어떨까요?

```
Original Train 24개
        ↓
Random Forest 학습
        ↓
Validation 12개
        ↓
Accuracy 83.33%
        ↓
Test 12개
        ↓
Accuracy 75.00%
```

Validation에서는 10개, Test에서는 9개를 맞혔습니다.

**즉, 세 가지 Action을 구분하는 모델이 만들어졌지만, 아직 안정적인 동작인식 모델이라고 판단하기는 어렵습니다.**

평가 데이터가 각각 12개뿐이므로 단 한 개를 더 맞히거나 틀려도 Accuracy가 약 8.33%p 달라집니다.

따라서 Accuracy만 보지 말고 Confusion Matrix를 통해 어떤 동작을 혼동했는지도 확인해야 합니다.

### ② 보강 데이터의 효과는 어떻게 판단할까요?

Human QA를 마치고 Augmented 모델까지 학습하면 다음과 같이 비교합니다.

|비교 결과|의미|
|---|---|
|Augmented Accuracy가 높음|보강이 도움이 되었을 가능성|
|두 모델의 Accuracy가 비슷함|보강 효과가 크지 않을 가능성|
|Augmented Accuracy가 낮음|보강 데이터의 품질이나 학습 조건을 점검할 필요|

다만 아직 Augmented 모델의 실제 Accuracy는 계산하지 않았으므로, **보강이 성능을 향상시켰는지는 판단할 수 없습니다.**

또한 이번 Test 결과를 확인했다고 해서 보강 파라미터를 변경해서는 안 됩니다. 이후 모델을 개선할 때는 Validation 결과를 사용하고, Test는 최종 평가용으로 유지해야 합니다.

### ③ 이번 실습의 최종 정리

```
Original Keypoint 24개
          ↓
204개 Feature 생성
          ↓
Random Forest 학습
          ↓
Validation Accuracy 83.33%
Test Accuracy       75.00%
          ↓
Baseline 성능 기록 완료
          ↓
Human QA 및 보강 데이터 검토
          ↓
승인된 Augmented Train 생성
          ↓
Baseline vs Augmented 비교
```

**이번 실습에서는 원본 데이터로 Random Forest를 학습하고 성능을 측정하는 데 성공했습니다.**

다음에는 보강된 Keypoint의 품질 문제를 검토한 후, 승인된 데이터가 모델의 성능 향상에 실제로 도움이 되는지 비교하겠습니다.

---

# PART 6. 5일차 실습 결과를 확인하고 교과 11을 정리합니다

---

# 34. 5일차 Main 실습의 최종 산출물을 확인합니다

이번 실습에서는 원본 영상 48개에서 Keypoint를 추출하고, Train 24개에 다섯 가지 보강을 적용하여 120개의 새로운 Sequence를 만들었습니다.

현재까지의 결과를 확인하겠습니다.

|구분|실제 결과|상태|
|---|---|---|
|Original Keypoint|Train 24 / Val 12 / Test 12|완료|
|Augmented Keypoint|120개 JSON 생성|완료|
|Metadata|`main_aug_metadata.csv`|완료|
|Automatic QA|168개 모두 PASS|완료|
|Human QA|5개 보강 방법 모두 `needs_review`|추가 검토|
|Feature|원본 48개 → 204개 Feature|완료|
|Baseline 모델|Random Forest 학습 및 평가|완료|
|Augmented 모델|승인된 보강 데이터 없음|보류|

### 실제로 생성된 Feature 및 성능평가 파일

```
data/features/main/main_aug/
├─ train_original.csv
├─ val_original.csv
└─ test_original.csv

reports/main/
├─ main_aug_metadata.csv
├─ main_aug_qa.csv
├─ main_aug_human_qa.csv
├─ main_aug_baseline_performance.csv
├─ main_aug_baseline_val_confusion.csv
├─ main_aug_baseline_confusion.csv
└─ main_aug_baseline_classification_report.json

models/main/main_aug/
└─ baseline.joblib
```

`train_with_aug.csv`, `augmented.joblib`과 Augmented 성능평가 보고서는 Human QA 승인이 완료된 후 생성합니다.

핵심: 보강 데이터를 120개 만들었더라도 모두 AI 학습에 사용할 수 있는 것은 아닙니다. 관절 움직임이 자연스러운지 검사한 후, 문제가 없는 데이터만 학습에 사용합니다.

---

# 35. 1~5일차에 기록한 Metadata의 공통점을 확인합니다

교과 11에서는 날짜마다 서로 다른 형태의 합성·보강 데이터를 만들었습니다.

| 일차     | 생성·보강한 데이터             | 주요 Metadata                                            |
| ------ | ---------------------- | ------------------------------------------------------ |
| 1일차  . | 이미지 보강                 | 원본 이미지, 보강 방법, Seed                                    |
| 2일차    | GAN / Diffusion 생성 이미지 | 생성 모델, Epoch, Prompt, Seed                             |
| 3일차    | Cut-Paste 객체 합성        | Background, Object, Position, Class                    |
| 4일차    | 3D Rendering 데이터       | 3D Object, Camera, Light, Seed                         |
| 5일차    | Keypoint Sequence 보강   | Source Video, Subject, Action, Method, Parameter, Seed |

Metadata는 **어떤 원본에서 어떤 방법과 조건으로 데이터를 만들었는지 추적하기 위한 기록**입니다.

예를 들어 5일차에 Skeleton 문제가 발생하면 Metadata를 이용해 원본 파일과 보강 설정을 다시 확인할 수 있습니다.

---

# 36. 5일 동안 진행한 QA의 공통 원칙을 정리합니다

|일차|주요 QA 검사|
|---|---|
|1일차|이미지 깨짐, 밝기, 형태|
|2일차|생성 실패, 반복 이미지, 실제 데이터와의 차이|
|3일차|Bounding Box, Class, Label|
|4일차|RGB, Mask, Depth, Bounding Box|
|5일차|Frame, Keypoint, 좌표, 동작의 자연스러움|

검사 대상은 달랐지만 목적은 같습니다.

```
원본 데이터
     ↓
합성 / 보강 데이터 생성
     ↓
QA 품질 검사
     ↓
사용 가능 여부 판단
     ↓
승인된 데이터만 Train에 추가
```

5일차에서는 Automatic QA 168개가 모두 PASS였지만, 일부 Skeleton에서 급격한 자세 변화가 관찰되어 Human QA 승인을 보류했습니다.

이는 **자동 검사 통과가 곧 학습데이터 사용 가능을 의미하지 않는다**는 실제 사례입니다.

---

# 37. 교과 11의 성능비교 원칙을 확인합니다

합성·보강 데이터의 효과는 같은 조건에서 모델 성능을 비교해야 알 수 있습니다.

![[Pasted image 20261016161737.png]]

```
[Baseline]
Original Train
      ↓
Random Forest A


[Augmented]
Original Train
      +
QA 승인된 보강 Train
      ↓
Random Forest B


[공정한 비교]
동일한 Validation
      ↓
모델 및 조건 비교·선택
      ↓
최종 조건 확정(Freeze)
      ↓
동일한 Test로 최종 평가
```

현재 5일차 Main의 실제 Baseline 결과는 다음과 같습니다.

|평가|Accuracy|정답 수|
|---|---|---|
|Validation|83.33%|10 / 12|
|Test|75.00%|9 / 12|

아직 Augmented 모델의 성능은 측정하지 않았으므로 보강 효과는 판단할 수 없습니다.

**중요:** Validation은 모델과 조건을 선택할 때 사용하고, Test는 최종 성능을 확인할 때 사용합니다. 이미 확인한 Test 결과에 맞추어 보강 파라미터를 변경해서는 안 됩니다.

---

# 38. 실제 실패 사례를 기록합니다

이번에는 Automatic QA를 통과했지만 움직임에 문제가 의심되는 데이터를 실패 분석 사례로 기록하겠습니다.

**작성할 파일:** `**reports/main/failure_cases.md**`

```markdown
# Day 05 Failure Cases

## Case 1. Time Warp 이후 관절 이동량 급증

- File: S13_handwaving_T02__timewarp_fast.json
- Subject: S13
- Action: handwaving
- Augmentation: timewarp_fast
- Parameter: factor=0.85
- Automatic QA: PASS
- Human QA: needs_review

### Problem

- max_joint_step = 0.763015
- Frame 398→399에서 왼쪽 발목 좌표가 크게 이동
- Frame 400에서 사람 검출 누락
- Skeleton 움직임이 부자연스러운지 추가 확인 필요

### Possible Cause

- 원본 YOLO Pose의 관절 추정 불안정
- Time Warp 보간 과정에서 불안정한 좌표가 반영되었을 가능성

### Fix

- 원본과 보강 Sequence의 대응 구간 비교
- 문제 원인을 확인한 뒤 보정 또는 데이터 제외 결정

### Result

- 현재 needs_review
- 원인 분석 및 최종 승인 미완료
- 학습데이터 사용 보류
```

실패 사례를 기록하는 이유는 단순히 오류를 찾기 위해서가 아닙니다.

**어떤 데이터에서 문제가 발생했고, 왜 발생했으며, 어떻게 해결할 것인지 남겨 다음 실험에서도 같은 문제를 반복하지 않기 위해서입니다.**

---

이번 PART에서는 5일차의 실제 산출물과 남은 검토 사항을 정리하고, 교과 11에서 공통으로 사용한 Metadata·QA·성능비교 원칙을 확인했습니다.

---

# PART 7. Mini Challenge — 새로운 Subject로 Jitter 강도를 비교합니다

이번 Mini Challenge는 5일차에 작성한 **기존 Script 00~06을 다시 사용**하는 실습입니다. 새로운 모델이나 알고리즘을 만들지 않습니다.

**스스로 해결할 질문:** 같은 동작 데이터에서 **Jitter를 0.005와 0.015로 적용하면 Skeleton의 품질과 Action Recognition 성능은 어떻게 달라질까요?**

> **진행 원칙:** 먼저 JSON 생성과 QA를 완료합니다. Human QA에서 `approved`한 보강 데이터만 모델 학습에 사용하며, 두 조건 중 어떤 것을 사용할지는 **Validation** 결과로 정합니다. **Test는 최종 선택 후 한 조건에만 사용합니다.**

---

# 39. Mini Challenge의 목표와 데이터를 확인합니다

본 실습의 `main` 대신 사람 번호(Subject)가 다른 `challenge` 데이터를 사용합니다.

| 구분 | Subject | 영상 수 | 역할 |
|---|---|---:|---|
| Train | S15, S16 | 12 | 모델이 학습할 데이터 |
| Validation | S21 | 6 | Jitter 후보 비교·선택 |
| Test | S05 | 6 | 선택한 조건의 최종 평가 |
| **전체** | | **24** | |

동작은 기존과 같은 `boxing`, `handwaving`, `handclapping` 세 가지입니다.

**이번에 변경할 변수는 Jitter 강도(`sigma`) 하나뿐입니다.** 원본 Train, Validation, 모델 설정, 204개 Feature 계산 방법, Seed는 그대로 유지합니다.

```text
Challenge Original Train 12개
              │
        ┌─────┴─────┐
        ↓           ↓
  Jitter 0.005  Jitter 0.015
        ↓           ↓
    각각 QA       각각 QA
        ↓           ↓
   승인된 데이터만 모델에 추가
        └─────┬─────┘
              ↓
       동일한 Validation 비교
              ↓
         Final 조건 선택
              ↓
        조건 확정(Freeze)
              ↓
     선택한 조건만 Test 평가
```

---

# 40. Challenge 데이터가 준비되었는지 검사합니다

먼저 파일 개수와 Train·Validation·Test 사이에 같은 사람이 겹치는지 확인합니다.

```bash
python scripts/00_check_day05.py \
  --asset-set challenge
```

다음을 확인하세요.

```text
Train : 12개 (S15, S16)
Val   :  6개 (S21)
Test  :  6개 (S05)
Subject Leakage: NONE
Day05 data check: READY
```

**직접 답해 보세요:** 같은 Subject가 Train과 Validation에 모두 포함되면 왜 공정한 평가가 어려울까요?

검사에서 오류가 발생하면 다음 단계로 넘어가지 말고 원본 영상 경로와 파일명을 먼저 확인합니다.

---

# 41. 영상 24개를 원본 Keypoint로 변환합니다

앞에서 만든 YOLO Pose 추출 코드를 그대로 실행합니다.

```bash
python scripts/01_extract_keypoints.py \
  --asset-set challenge \
  --split all
```

정상적으로 처리되면 다음 JSON이 준비됩니다.

```text
data/keypoints/challenge/original/
├─ train/   12개
├─ val/      6개
└─ test/     6개
```

원본 영상의 관절 추정이 정상적인지 대표 영상 하나를 확인합니다.

```bash
python scripts/02_visualize_keypoints.py \
  --json data/keypoints/challenge/original/train/S15_boxing_T01.json \
  --video data/day05_student_data/challenge/train/S15_boxing_T01.avi \
  --output results/challenge/S15_boxing_T01_pose.mp4
```

생성된 영상을 재생하여 손목·팔꿈치가 실제 사람의 동작을 따라가는지 확인하세요. **원본에서 이미 관절이 튄다면 보강 후에도 같은 문제가 나타날 수 있습니다.**

---

# 42. 실험 A — Jitter 0.005를 적용합니다

`--methods jitter`를 지정하여 **다섯 보강 방법 중 Jitter만 실행**합니다.

### ① 보강 데이터 생성

```bash
python scripts/03_augment_keypoints.py \
  --asset-set challenge \
  --tag jitter005 \
  --jitter-sigma 0.005 \
  --methods jitter
```

원본 Train 12개에서 Jitter JSON 12개가 생성됩니다. 원본은 변경되지 않습니다.

```text
data/keypoints/challenge/augmented/jitter005/
→ 보강 JSON 12개

reports/challenge/jitter005_metadata.csv
→ 사용한 원본·sigma·Seed 기록
```

이미 같은 `tag`의 결과가 있다면 프로그램이 중단됩니다. 결과를 덮어쓰기 전에 기존 실험 기록을 확인하세요. `--overwrite`는 해당 보강 결과를 삭제하고 다시 생성하므로 주의합니다.

### ② Automatic QA 수행

```bash
python scripts/04_qa_keypoints.py \
  --asset-set challenge \
  --tag jitter005
```

검사 대상은 **원본 24개 + 보강 12개 = 총 36개**입니다.

`reports/challenge/jitter005_qa.csv`에서 `auto_pass`, `reason`, `mean_joint_step`, `max_joint_step`을 확인하세요. **PASS 수는 실제 실행 결과를 기록**합니다.

### ③ Skeleton 확인 및 Human QA

```bash
python scripts/02_visualize_keypoints.py \
  --json data/keypoints/challenge/augmented/jitter005/S15_boxing_T01__jitter.json \
  --output results/challenge/S15_boxing_jitter005.mp4
```

이 영상은 **검은 배경에 Skeleton만 표시**됩니다. 원본 영상과 비교하며 관절이 갑자기 튀는지 확인하세요. 가능하면 다른 Action·Subject의 영상도 추가로 보고, Automatic QA의 이동량이 큰 파일을 우선 검사합니다.

Human QA 파일을 직접 만듭니다.

**파일: `reports/challenge/jitter005_human_qa.csv`**

```bash
cat > reports/challenge/jitter005_human_qa.csv <<'CSV'
method,decision,note
jitter,needs_review,영상과 관절 움직임 확인 중
CSV
```

영상을 확인한 후 CSV를 직접 수정합니다.

- `approved` — 확인한 동작이 자연스럽고 학습에 사용해도 되겠다고 판단
- `needs_review` — 원인을 더 확인해야 함
- `rejected` — 학습에 사용하기 부적절함

**승인 전에는 `approved`로 변경하지 마세요.** 현재 코드는 방법 단위로 승인하므로, 의심스러운 파일이 있으면 원본과 비교한 뒤 문제를 해결하거나 해당 방법의 승인을 보류해야 합니다.

### ④ Feature 생성과 Validation 평가

**아직 승인하지 못했다면** 원본 Baseline만 실행할 수 있습니다.

```bash
python scripts/05_build_features.py \
  --asset-set challenge \
  --tag jitter005 \
  --baseline-only

python scripts/06_compare_baseline_augmented.py \
  --asset-set challenge \
  --tag jitter005 \
  --eval-split val \
  --baseline-only
```

**Human QA에서 `jitter`를 승인했다면**, 다음 명령으로 보강 데이터까지 포함하여 평가합니다.

```bash
python scripts/05_build_features.py \
  --asset-set challenge \
  --tag jitter005

python scripts/06_compare_baseline_augmented.py \
  --asset-set challenge \
  --tag jitter005 \
  --eval-split val
```

승인된 보강 JSON이 12개이고 모두 Automatic QA에 통과했다면 학습 데이터는 **Original 12개 + Jitter 12개 = 24개**가 됩니다. 실제 학습 개수는 터미널에서 확인하세요.

---

# 43. 실험 B — Jitter만 0.015로 변경합니다

이번에는 **Jitter 노이즈의 강도를 세 배로 높여** 비교합니다.

```text
실험 A: sigma = 0.005
실험 B: sigma = 0.015
```

두 실험은 같은 원본 JSON과 Seed를 사용하므로 **노이즈의 무작위 패턴은 같고 크기만 달라집니다.** 다른 보강 방법은 추가하지 않습니다.

### ① 보강 → Automatic QA

```bash
python scripts/03_augment_keypoints.py \
  --asset-set challenge \
  --tag jitter015 \
  --jitter-sigma 0.015 \
  --methods jitter

python scripts/04_qa_keypoints.py \
  --asset-set challenge \
  --tag jitter015
```

이번에도 **보강 JSON 12개**, QA 검사 대상 **36개**가 예상됩니다. 실제 PASS·REVIEW 결과를 적습니다.

### ② Skeleton → Human QA

```bash
python scripts/02_visualize_keypoints.py \
  --json data/keypoints/challenge/augmented/jitter015/S15_boxing_T01__jitter.json \
  --output results/challenge/S15_boxing_jitter015.mp4
```

두 Jitter 영상을 비교하면서 **손목·팔꿈치의 흔들림, 급격한 자세 변화, 동작 의미 유지 여부**를 확인하세요.

**파일: `reports/challenge/jitter015_human_qa.csv`**

```bash
cat > reports/challenge/jitter015_human_qa.csv <<'CSV'
method,decision,note
jitter,needs_review,영상과 관절 움직임 확인 중
CSV
```

직접 확인한 결과에 따라 `decision`과 `note`를 수정하세요.

### ③ Feature → Validation 평가

**승인 전에는 Baseline만** 확인합니다.

```bash
python scripts/05_build_features.py \
  --asset-set challenge \
  --tag jitter015 \
  --baseline-only

python scripts/06_compare_baseline_augmented.py \
  --asset-set challenge \
  --tag jitter015 \
  --eval-split val \
  --baseline-only
```

**`jitter`가 실제로 승인된 경우에만** 보강 모델을 실행합니다.

```bash
python scripts/05_build_features.py \
  --asset-set challenge \
  --tag jitter015

python scripts/06_compare_baseline_augmented.py \
  --asset-set challenge \
  --tag jitter015 \
  --eval-split val
```

---

# 44. 두 Jitter 실험 결과를 비교합니다

이제 **직접 실행해서 얻은 결과**로 아래 표를 완성하세요. 성능이나 QA 결과를 미리 예상하여 적지 않습니다.

| 비교 항목 | Jitter 0.005 | Jitter 0.015 |
|---|---|---|
| 생성한 보강 JSON | 12개 예상 | 12개 예상 |
| Automatic QA PASS (보강 12개 중) | 직접 기록 | 직접 기록 |
| 평균 `mean_joint_step` | 직접 계산·기록 | 직접 계산·기록 |
| 가장 큰 `max_joint_step` | 직접 확인·기록 | 직접 확인·기록 |
| Human QA | `approved` / `needs_review` / `rejected` | 직접 기록 |
| Baseline Validation Accuracy | 실제 값 | 실제 값 |
| Augmented Validation Accuracy | 승인 시 기록 | 승인 시 기록 |
| Validation Delta (Aug - Base) | 승인 시 기록 | 승인 시 기록 |
| Confusion Matrix에서 많이 헷갈린 동작 | 실제 결과 | 실제 결과 |

다음 파일을 열어 표에 사용할 수 있습니다.

```text
reports/challenge/jitter005_qa.csv
reports/challenge/jitter015_qa.csv

reports/challenge/jitter005_human_qa.csv
reports/challenge/jitter015_human_qa.csv

승인된 실험의 Validation 비교 결과:
reports/challenge/jitter005_performance_val.csv
reports/challenge/jitter015_performance_val.csv

승인된 실험의 Confusion Matrix:
reports/challenge/jitter005_baseline_val_confusion.csv
reports/challenge/jitter005_augmented_val_confusion.csv
reports/challenge/jitter015_baseline_val_confusion.csv
reports/challenge/jitter015_augmented_val_confusion.csv
```

Baseline 전용으로만 실행했다면 결과 파일명은 `jitter005_baseline_performance_val.csv`, `jitter015_baseline_performance_val.csv`입니다. **보강 성능 파일이 없다고 임의로 숫자를 채우지 마세요.**

Validation 비교 결과 CSV는 `model`, `train_count`, `eval_split`, `accuracy` 열을 사용합니다. Accuracy가 `0.8333`이라면 **83.33%**로 읽습니다.

**확인할 질문**

1. Baseline Validation Accuracy가 두 실험에서 같은가? 같은 원본 Train·Validation·모델 설정을 사용했다면 동일해야 합니다.
2. Jitter 0.015에서 관절 이동량과 영상의 흔들림이 어떻게 달라졌는가?
3. Accuracy뿐 아니라 어떤 Action을 잘못 분류했는지 Confusion Matrix에서도 차이가 있는가?
4. 성능이 좋아져도 Human QA에서 문제가 발견됐다면 학습에 사용해도 될까?

> Validation은 영상이 6개뿐이므로 **한 개의 예측 차이가 Accuracy 약 16.67%p**에 해당합니다. 숫자가 조금 달라졌다고 단정하지 말고 QA와 오분류 결과를 함께 봅니다.

---

# 45. 더 적절한 Jitter를 선택하고 Test는 마지막에 평가합니다

이번 Mini Challenge는 **Jitter 강도 하나만 바꾼 비교 실험**입니다.

```text
원본 Train 12개 + Jitter 0.005
             vs
원본 Train 12개 + Jitter 0.015
                 ↓
         동일한 Validation
         + QA 결과 비교
                 ↓
     학습에 사용할 조건 하나 선택
                 ↓
        설정 확정(Freeze)
                 ↓
        선택한 조건만 Test
```

### 선택 기준

- **두 후보 모두 `approved`**: Validation Accuracy, Confusion Matrix, 움직임의 자연스러움을 종합하여 하나를 선택합니다.
- **한 후보만 `approved`**: 승인된 후보만 최종 학습 후보가 될 수 있습니다. 승인되지 않은 후보와 성능 우열을 단정하지 않습니다.
- **두 후보 모두 승인되지 않음**: 보강 모델의 최종 Test를 진행하지 않고, 발견한 문제와 보류 이유를 기록합니다.

### 최종 Test 실행 (승인·선택이 완료된 경우만)

아래의 `jitter005`는 예시입니다. **자신이 Validation과 QA로 선택한 Tag로 바꾼 후 한 번만 실행**하세요.

```bash
SELECTED_TAG=jitter005

python scripts/06_compare_baseline_augmented.py \
  --asset-set challenge \
  --tag "$SELECTED_TAG" \
  --eval-split test
```

이 명령은 **선택된 한 Tag에서 Baseline과 Augmented 모델을 같은 Test(S05)에 평가**합니다. 두 Tag를 차례로 Test하지 않습니다.

생성되는 주요 파일:

```text
reports/challenge/<선택태그>_performance_test.csv
reports/challenge/<선택태그>_baseline_test_confusion.csv
reports/challenge/<선택태그>_augmented_test_confusion.csv
reports/challenge/<선택태그>_test_classification_report.json
```

Test 결과를 본 뒤 Jitter 강도나 모델 설정을 다시 바꾸지 않습니다. 그렇지 않으면 Test가 사실상 후보 선택에 사용되어 공정한 최종 평가가 어려워집니다.

---

# 46. 직접 수행한 결과를 `challenge_notes.md`에 기록합니다

**파일: `reports/challenge/challenge_notes.md`**

아래 양식을 복사하여 **실제 출력값과 직접 관찰한 내용으로 채우세요.** 실행하지 못했거나 승인되지 않은 항목은 `미실행(사유)`로 적습니다.

```markdown
# Day 05 Mini Challenge — Jitter 강도 비교

## 1. Challenge 데이터
- Train: S15, S16 / 12개
- Validation: S21 / 6개
- Test: S05 / 6개
- Subject Leakage:
- 생성한 Original Keypoint JSON 수:
- 확인한 Original Skeleton 영상:

## 2. Candidate 비교

| 항목 | Jitter 0.005 | Jitter 0.015 |
|---|---|---|
| 보강 JSON 수 | | |
| Automatic QA PASS 수 | | |
| mean_joint_step 특징 | | |
| max_joint_step 특징 | | |
| Skeleton 관찰 내용 | | |
| Human QA 결정 | | |
| Baseline Val Accuracy | | |
| Augmented Val Accuracy | | |
| Validation Delta | | |
| 헷갈린 Action(Confusion Matrix) | | |

## 3. Final 조건 선택
- 두 실험에서 같게 유지한 것:
- 바꾼 변수:
- 선택한 Tag(또는 선택 보류):
- 선택 근거(Validation + QA):
- Final 조건 Freeze 여부:

## 4. Final Test
- Test 실행 여부와 사유:
- 선택된 Tag:
- Baseline Test Accuracy:
- Augmented Test Accuracy:
- Test Delta:
- Confusion Matrix에서 확인한 점:
- Test 확인 후 설정 변경 여부: 변경하지 않음

## 5. 실패·의심 사례
- 파일명:
- 관찰한 문제(없다면 '발견하지 못함'):
- Automatic QA 결과:
- Human QA 결과:
- 원인으로 의심되는 점:
- 다음에 확인하거나 고칠 점:

## 6. 결론
- Jitter 강도를 높였을 때 어떤 변화가 있었는가?
- 이번 결과만으로 보강 효과를 확정하기 어려운 이유는?
```

**실패 사례는 `challenge`에서 직접 발견한 파일로 기록**합니다. Main의 실패 사례를 그대로 복사하지 않습니다.

---

# 47. Mini Challenge 완료 여부를 스스로 확인합니다

- [ ] Challenge 24개 영상과 Subject Leakage를 확인했다.
- [ ] 원본 Keypoint 24개를 만들고 Skeleton을 확인했다.
- [ ] Jitter 0.005와 0.015를 **각각 12개씩** 생성했다.
- [ ] 두 실험의 Automatic QA 및 Skeleton 비교를 수행했다.
- [ ] Human QA CSV 두 개를 직접 작성하고 판단 근거를 남겼다.
- [ ] `approved`되지 않은 보강 데이터를 학습에 사용하지 않았다.
- [ ] 원본 Baseline과, 승인된 경우 Augmented 모델을 **Validation만** 평가했다.
- [ ] QA·Accuracy·Confusion Matrix를 함께 비교하여 선택하거나 보류했다.
- [ ] 선택된 한 조건만 Freeze 후 Test를 평가했다(승인·선택된 경우).
- [ ] Test 결과로 Jitter 파라미터를 다시 조정하지 않았다.
- [ ] `challenge_notes.md`에 결과와 문제·의심 사례를 기록했다.

---

# 48. 약 3시간 동안 Mini Challenge를 진행합니다

| 실습 단계                               |
| ----------------------------------- |
| 데이터 검사 및 24개 Keypoint 추출            |
| Original Skeleton 확인                |
| 실험 A — Jitter 0.005·QA              |
| 실험 B — Jitter 0.015·QA              |
| Feature 생성 및 Validation 성능평가        |
| QA·Confusion Matrix 비교, Final 조건 선택 |
| 승인된 경우 Final Test, 문제 사례 정리         |
| `challenge_notes.md` 작성·검토          |

실행 속도와 Human QA에 필요한 시간에 따라 일정은 달라질 수 있습니다. **정해진 시간보다 QA 판단과 실험 조건을 지키는 것이 더 중요합니다.**

**Mini Challenge의 최종 목표:** Jitter 0.005와 0.015 중 무엇이 더 적절한지 **실제 QA와 Validation 결과로 설명**하고, 선택한 조건에 대해서만 Test 결과를 보고하는 것입니다.

---

# PART 8. 5일차 실습 결과를 기록하고 교과 11을 마무리합니다

이제 Main 실습과 Mini Challenge의 과정을 반복 설명하지 않고, **실제로 만든 결과를 문서로 남기고 Git에 기록**하겠습니다. Main의 수치는 이미 확인한 결과를 사용하고, Challenge의 성능 수치는 자신의 `challenge_notes.md`에서 가져옵니다.

---

# 49. `day05_notes.md`에 5일차 결과를 기록합니다

**작성할 파일: `day05_notes.md`**

새로운 설명을 길게 작성하지 말고, 확인한 결과와 아직 완료하지 못한 부분만 기록합니다.

```markdown
# Subject 11 — Day 05 실습 결과

## 1. Main 실습 결과
- 원본 영상 / Keypoint JSON: 48개 (Train 24, Val 12, Test 12)
- Action: boxing, handwaving, handclapping
- Train Subject: S11, S12, S13, S14
- Val Subject: S19, S20 / Test Subject: S02, S03
- 보강: Train 24개 × 5가지 = 120개 JSON
- Automatic QA: 총 168개 PASS, REVIEW 0개
- Human QA: 5가지 방법 모두 needs_review
- Feature: 원본 48개를 각각 204개 특징값으로 변환
- Baseline Random Forest: Train 24개
- Validation: 83.33% (10/12)
- Test: 75.00% (9/12)
- Augmented 성능: 미평가 (Human QA 승인 보류)

## 2. 발견한 문제
- 파일: S13_handwaving_T02__timewarp_fast.json
- 관찰: max_joint_step=0.763015, 관절 급변 의심
- 현재 조치: needs_review, 원본과 비교 후 판단
- 자세한 기록: reports/main/failure_cases.md

## 3. Mini Challenge 결과
- 비교 조건: Jitter sigma=0.005 / 0.015
- 실제 QA·Validation 결과: reports/challenge/challenge_notes.md 참조
- 선택한 Jitter(또는 선택 보류): [본인 결과 입력]
- Final Test(평가했을 때만): [본인 결과 입력 / 미실행 사유]

## 4. 오늘의 결론
- 데이터 생성 후 QA로 사용 가능 여부를 판단해야 한다.
- 후보 비교는 Validation, 최종 평가는 확정된 조건의 Test를 사용한다.
```

**주의:** Main의 `Augmented Accuracy`, `Test Delta`는 아직 측정되지 않았으므로 임의의 숫자를 적지 않습니다. Mini Challenge 결과도 학생이 실제로 얻은 값만 기록합니다.

---

# 50. 교과 11의 방법 선택표를 완성합니다

**작성할 파일: `reports/method_selection_matrix.md`**

어떤 상황에서 1~5일차의 방법을 활용할지 간단히 정리합니다.

```markdown
# Subject 11 — Method Selection Matrix

| 필요한 데이터 | 고려할 방법 | 꼭 확인할 QA |
|---|---|---|
| 밝기·회전 등 이미지 조건 다양화 | Image Augmentation / VAE (1일차) | 형태 보존·복원 품질 |
| 새로운 이미지 생성 | GAN / WGAN-GP / Diffusion (2일차) | 다양성·생성 실패·Domain Gap |
| 객체 탐지용 조합·Label 확보 | Cut-Paste (3일차) | 합성 경계·BBox·Class |
| 카메라·조명·Mask·Depth 확보 | 3D Rendering (4일차) | RGB·Mask·Depth·BBox 일치 |
| 사람 동작의 좌표·시간 변화 확보 | Keypoint Augmentation (5일차) | 관절 움직임·Action 의미 유지 |

**선택 원칙:** 데이터를 왜 늘리는지 먼저 정하고, QA 후 같은 평가 데이터에서 실제 효과를 확인한다.
```

---

# 51. `final_reflection.md`에 스스로 배운 내용을 정리합니다

**작성할 파일: `final_reflection.md`**

아래 네 가지 질문에 자신의 실험 경험을 바탕으로 2~3문장씩 답합니다. 앞에서 작성한 결과표를 다시 복사할 필요는 없습니다.

```markdown
# Subject 11 — Final Reflection

1. 1~5일차 중 실제 프로젝트에 가장 활용하고 싶은 합성·보강 방법은 무엇이며, 그 이유는 무엇인가?
   - 나의 답:

2. 5일차 Main에서는 Automatic QA가 모두 PASS였는데도 Human QA를 보류한 이유는 무엇인가?
   - 나의 답:

3. Mini Challenge에서 Jitter 0.005와 0.015를 비교하며 가장 중요하게 확인한 결과는 무엇인가?
   - 나의 답:

4. 앞으로 제조 Vision AI나 산업안전 Action AI 프로젝트에서 합성데이터를 사용하기 전에 무엇을 확인하겠는가?
   - 나의 답:
```

---

# 52. `README.md`에 프로젝트 사용 방법을 기록합니다

**작성할 파일: `README.md`**

다른 사람이 프로젝트를 열었을 때 무엇을 실습했고, 어디에서 결과를 확인하면 되는지 알 수 있도록 작성합니다.

````markdown
# Subject 11 — Day 05
KTH Keypoint Sequence Augmentation · QA · Evaluation

## 데이터
- Main: Train 24 / Val 12 / Test 12
- Challenge: Train 12 / Val 6 / Test 6
- Action: boxing / handwaving / handclapping

## 처리 흐름
Video → YOLO Pose → COCO17 JSON → Train 보강 → QA → 204-D Feature → Random Forest

## 코드
`scripts/00_check_day05.py` ~ `scripts/06_compare_baseline_augmented.py`

## Main 실행 예시
```bash
python scripts/00_check_day05.py --asset-set main
python scripts/05_build_features.py --asset-set main --tag main_aug --baseline-only
python scripts/06_compare_baseline_augmented.py --asset-set main --tag main_aug --eval-split both --baseline-only
```

## 확인된 Main 결과
- Original Keypoint 48개 / Augmented JSON 120개
- Automatic QA: 168/168 PASS
- Human QA: 추가 검토 중 (Augmented 학습·평가 보류)
- Baseline: Validation 83.33%, Test 75.00%

## 보고서
- Main: `day05_notes.md`, `reports/main/`
- Mini Challenge: `reports/challenge/challenge_notes.md`
- 교과 11 정리: `reports/method_selection_matrix.md`, `final_reflection.md`

## 주의
보강 데이터는 Automatic QA 통과와 Human QA 승인을 확인한 뒤 Train에만 추가합니다.
평가에 사용한 Test를 보고 보강 파라미터를 다시 조정하지 않습니다.
원본 영상·생성 JSON·Feature·모델은 Git에 포함하지 않습니다.
````

여기에는 **현재 Main이 Baseline까지만 완료되었다는 사실**을 그대로 기록합니다. Mini Challenge의 세부 수치와 판단 근거는 `challenge_notes.md`에 정리했으므로 중복 작성하지 않습니다.

---

# 53. `.gitignore`를 작성합니다

**작성할 파일: `.gitignore`**

Git에는 코드와 작은 보고서 중심으로 기록하고, 원본 영상이나 생성된 대용량 파일은 제외합니다.

```gitignore
# Python / 환경
__pycache__/
*.pyc
.venv/
.vscode/

# 제공 영상·생성 데이터
data/day05_student_data/
data/keypoints/
data/features/
*.avi
*.mp4
*.mov

# 모델·미리보기
models/
results/
*.pt
*.pth
*.joblib

# 운영체제
.DS_Store
Thumbs.db
```

`reports/`의 작은 CSV·Markdown 결과는 Git에 기록할 수 있습니다. 단, 원본 자료의 배포 권한과 기관 보안 규정을 먼저 확인합니다.

---

# 54. 코드와 실습 문서를 Git에 저장합니다

**실제 사용한 Day 05 프로젝트 폴더**에서 실행합니다.

```bash
cd ~/project11/day05_keypoint_sequence

git rev-parse --show-toplevel
git status

git add .gitignore README.md day05_notes.md final_reflection.md \
  requirements_day05.txt scripts/ src/ reports/

git diff --cached --stat
git status
```

Stage 목록에 원본 AVI·Keypoint JSON·Feature CSV·모델 파일이 없는지 확인한 뒤 Commit합니다.

```bash
git commit -m "docs: finalize subject11 day05 results and challenge"
```

원격 저장소가 연결되어 있고 **기관에서 업로드를 허용한 경우에만** Push합니다.

---

# 55. 오늘의 최종 산출물을 체크합니다

- [ ] `day05_notes.md` — Main 실제 수치와 보류 사유를 기록했다.
- [ ] `reports/main/failure_cases.md` — 발견한 문제를 기록했다.
- [ ] `reports/challenge/challenge_notes.md` — 두 Jitter의 실제 결과와 판단을 기록했다.
- [ ] `reports/method_selection_matrix.md` — 1~5일차 활용 방법을 정리했다.
- [ ] `final_reflection.md` — 네 질문에 자신의 생각으로 답했다.
- [ ] `README.md`와 `.gitignore` — 사용 방법과 제외 파일을 정리했다.
- [ ] Git Stage를 검토하고 Commit했다.

승인하지 않은 보강 데이터를 학습에 넣지 않았는지, Challenge의 Test 결과로 Jitter 강도를 다시 변경하지 않았는지도 마지막으로 확인합니다.

---

# 56. 교과 11을 마무리합니다

교과 11의 1~5일차에서는 이미지 보강, 생성모델, Cut-Paste, 3D Rendering, Keypoint Sequence 보강을 실습했습니다.

방법은 달라도 가장 중요한 질문은 같습니다.

> **부족한 데이터를 어떤 방법으로 보완하고, 새로 만든 데이터가 실제 학습에 도움이 되는지 어떻게 확인할 것인가?**

5일차 Main에서는 원본 Baseline **Validation 83.33%, Test 75.00%**를 확인했고, 보강 데이터는 Skeleton 품질 문제로 사용 승인을 보류했습니다. Mini Challenge에서는 **Jitter 강도 하나만 변경하는 비교 방법**을 연습했습니다. 두 실습의 기록을 남겼다면 오늘 학습은 마무리됩니다.

앞으로 제조 검사나 산업안전 동작인식 프로젝트에서도 **데이터 부족 원인 확인 → 적절한 보강 방법 선택 → QA → 학습 → 공정한 성능평가**의 순서를 활용해 보세요.
