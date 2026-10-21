## 오늘의 목표

1일차에서는 이미지·저장 영상·스마트폰 실시간 영상이 결국 Image 또는 Frame으로 처리된다는 것을 확인하고, 검사할 위치인 ROI를 직접 정했습니다. 또한 같은 ROI도 제품 위치·카메라 위치·조명 조건에 따라 실패할 수 있다는 것을 확인했습니다.

2일차에서는 그 다음 질문을 다룹니다.

> 검사할 위치는 정했는데, 그 ROI가 컴퓨터가 검사하기 좋은 영상 상태인가?

2일차는 Image Enhancement를 중심으로 저대비·어두운·Noise 영상의 품질을 개선하고, 이미지마다 적절한 전처리 조합을 스스로 선택합니다.

오늘 수업이 끝나면 다음을 할 수 있어야 합니다.

```text
- 1일차에 만든 프로젝트와 파일 구조를 이어서 사용할 수 있다.
- OpenCV의 BGR, Gray, HSV가 각각 어떤 정보를 표현하는지 설명할 수 있다.
- Brightness와 Contrast가 픽셀값에 어떤 영향을 주는지 설명할 수 있다.
- Gamma Correction을 적용하고 여러 파라미터의 결과를 비교할 수 있다.
- CLAHE를 이용하여 저대비 ROI를 개선할 수 있다.
- Gaussian, Median, Bilateral Filter의 차이를 비교할 수 있다.
- Sharpening의 효과와 한계를 설명할 수 있다.
- Threshold와 Otsu를 이용하여 전처리 전·후의 분리 결과를 비교할 수 있다.
- 문제 이미지에 필요한 전처리를 스스로 선택하고 그 이유를 기록할 수 있다.
- 과도한 전처리가 오히려 결함을 없애거나 Noise를 키울 수 있음을 설명할 수 있다.
- 결과 이미지, 파라미터 표, 분석 문서, Git 기록을 남길 수 있다.
```

# 1. 오늘 수업의 전체 흐름

2일차도 1일차와 같은 방식으로 진행합니다.

기술 이름을 길게 설명한 뒤 코드를 외우는 방식이 아니라, 문제를 먼저 보고 필요한 기술을 적용한 뒤 결과와 실패 원인을 확인합니다.

![[Pasted image 20260828183223.png]]

```text
1일차 결과 확인
       ↓
Day02 실습 데이터 준비
       ↓
BGR · Gray · HSV 비교
       ↓
Brightness · Contrast
       ↓
Gamma Correction
       ↓
CLAHE
       ↓
Gaussian · Median · Bilateral
       ↓
Sharpening
       ↓
Threshold · Otsu 분리 결과 확인
       ↓
전처리 없음 · Gaussian · CLAHE
Otsu 결과 비교
       ↓
결과 해석
좋아진 점 · 나빠진 점 판단
       ↓
Glare 사례 확인
소프트웨어 문제 · 촬영환경 문제 구분
       ↓
Mini Challenge
새로운 문제 이미지 직접 판단
       ↓
필요한 전처리 직접 선택
       ↓
Parameter Table · 실습 기록 작성
       ↓
README · Local Git 정리
```

### 오늘의 사고방식

오늘은 전처리 기술을 무조건 적용하는 것이 아니라, 이미지에서 검사에 방해되는 문제를 먼저 찾고 필요한 기술만 선택하는 연습을 합니다.

예를 들어 이미지가 너무 어둡거나, Noise가 많거나, 경계가 흐린 경우처럼 병뚜껑을 구분하기 어렵게 만드는 원인을 확인합니다.

```text
문제 이미지 확인
      ↓
어두움 · 저대비 · Noise · Blur · 반사 등
검사를 방해하는 원인 찾기
      ↓
필요한 전처리 후보 선택
      ↓
파라미터를 바꾸어 결과 비교
      ↓
Threshold / Otsu에서
객체와 배경이 더 잘 분리되는지 확인
      ↓
사용 / 미사용 / 추가 확인 결정
```

핵심은 다음과 같습니다.

> 전처리 기술부터 선택하는 것이 아니라, 이미지의 문제를 먼저 찾고 실제 검사에 도움이 되는 경우에만 사용합니다.

---

# 2. 1일차와 2일차는 어떻게 연결되나요?

![[Pasted image 20261020173554.png]]

1일차의 핵심 흐름은 다음이었습니다.

```text
Image / Video Frame / 실시간 Frame
                ↓
               ROI
                ↓
정상 · 불량의 차이를 관찰
                ↓
나중에 측정할 특징을 생각
```

2일차에는 이 ROI가 다음 검사에 적합한 상태인지 확인합니다.

```text
1일차
검사할 위치 결정
       ↓
      ROI

2일차
ROI 영상 품질 확인
       ↓
Image Enhancement
       ↓
Threshold / Otsu로
분리 가능성 확인

3일차
Morphology
       ↓
Contour
       ↓
Area / Count / Position
       ↓
Rule
       ↓
OK / NG
```

따라서 오늘의 전처리는 최종 판정이 아닙니다.

> 오늘은 “영상 개선”이 목적이 아니라, 3일차의 객체 분리와 측정이 더 안정적으로 되도록 입력을 준비하는 것이 목적입니다.

---

# 결과 미리보기

| 실행 프로그램                     | 생성 결과                            |
| --------------------------- | -------------------------------- |
| `09_color_space.py`         | BGR·Gray·H·S·V 채널 이미지            |
| `10_brightness_contrast.py` | 밝기 값을 다르게 적용한 이미지                |
| `11_gamma.py`               | Gamma 0.6~2.0 비교 이미지             |
| `12_clahe.py`               | CLAHE 강도별 저대비 개선 이미지             |
| `13_filter_compare.py`      | Gaussian·Median·Bilateral 비교 이미지 |
| `14_sharpen.py`             | 원본·Blur·Sharpening 비교 이미지        |
| `15_threshold_otsu.py`      | Threshold 80·120·160·Otsu 결과     |
| `16_preprocess_compare.py`  | 4분할 종합 비교 이미지                    |
| `17_day02_challenge.py`     | 선택한 전처리 결과 및 Otsu 비교             |

수업전 어떤 과정으로 진행할지 미리보기

```bash
python src/09_color_space.py
python src/10_brightness_contrast.py
python src/11_gamma.py
python src/12_clahe.py
python src/13_filter_compare.py
python src/14_sharpen.py
python src/15_threshold_otsu.py
python src/16_preprocess_compare.py
python src/17_day02_challenge.py
```

![[Pasted image 20261020171314.png]]

개념적으로는 세그멘테이션처럼 병뚜껑 영역을 분리하는 과정과 비슷합니다. 
그러나 정확히는 딥러닝 기반 Semantic Segmentation이 아니라 
OpenCV의 전처리와 Otsu(오츠) 이진화(Binary Segmentation) 를 사용한 것입니다.

병뚜껑과 배경을 최대한 잘 분리하여, 3일차에 Contour·면적·위치 등을 측정하고 뚜껑의 유무를 OK/NG로 판정할 수 있도록 준비하는 것입니다.

오츠 알고리즘의 역사 이야기

```
1979년 일본의 영상처리·패턴인식 연구자인 오츠 노부유키(大津展之, Nobuyuki Otsu)가 
발표한 알고리즘입니다. 사람이 직접 정하던 이진화 기준값(Threshold)을 이미지의 밝기 분포에 대한 통계적 분석을 통해 자동으로 선택하는 방법으로, 두 그룹 간 분산이 최대가 되는 임계값을 
찾습니다. 연구자의 이름을 따서 오츠의 방법(Otsu's Method) 또는 오츠 이진화라고 부릅니다.
```

2일차는 병뚜껑의 형태를 배경과 잘 구분할 수 있도록 이미지를 전처리하고 이진화하는 과정이며, 3일차는 분리된 영역의 면적·위치·형태를 측정하여 병뚜껑 유무를 판정하는 과정입니다.

---
# 3. 파일명 확인

1일차 최종 프로젝트에는 `src/00_check_env.py`부터 `src/08_phone_camera_roi.py`까지 만들어져 있습니다.

2일차 파일은 09부터 이어서 작성합니다.

```text
09_color_space.py
10_brightness_contrast.py
11_gamma.py
12_clahe.py
13_filter_compare.py
14_sharpen.py
15_threshold_otsu.py
16_preprocess_compare.py
17_day02_challenge.py
```

| 파일명                         | 역할                    | 설명                                                                                                           |
| --------------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------ |
| `09_color_space.py`         | 색공간 <br>비교            | 같은 이미지를 BGR, Gray, HSV로 변환하고 H·S·V 채널을 각각 저장하여, 검사 목적에 따라 어떤 색공간이 적합한지 확인합니다.                                |
| `10_brightness_contrast.py` | 밝기·대비 <br>조절          | `alpha`, `beta` 값을 바꾸어 이미지의 Brightness와 Contrast를 조절하고, 너무 어둡거나 대비가 약한 영상이 어떻게 달라지는지 비교합니다.                  |
| `11_gamma.py`               | Gamma <br>보정          | 여러 Gamma 값을 적용하여 어두운 영역이나 중간 밝기 영역을 조절하고, 어떤 값이 검사 대상을 더 잘 보이게 만드는지 비교합니다.                                   |
| `12_clahe.py`               | 지역 대비 <br>개선          | 저대비 이미지를 Gray로 변환한 뒤 CLAHE(클래해)를 적용하여 제품과 배경의 지역적인 명암 차이를 개선하고 `clipLimit`에 따른 결과 차이를 확인합니다.                 |
| `13_filter_compare.py`      | Noise 제거 방법 비교        | Gaussian Blur, Median Blur, Bilateral Filter를 같은 이미지에 적용하여 Noise 감소 효과와 경계·작은 결함의 손실 정도를 비교합니다.              |
| `14_sharpen.py`             | 경계 강조                 | 흐릿한 이미지에 Sharpening Kernel을 적용하여 경계를 더 선명하게 만들고, 동시에 Noise가 함께 증가하는지 확인합니다.                                  |
| `15_threshold_otsu.py`      | 이진화 <br>결과 비교         | Gray 이미지에 여러 고정 Threshold 값과 Otsu Threshold를 적용하여 객체와 배경이 어떻게 분리되는지 비교합니다. 2일차에서는 전처리 효과를 확인하기 위한 용도로 사용합니다. |
| `16_preprocess_compare.py`  | 전처리 <br>전·후 <br>종합 비교 | `Original + Otsu`, `Gaussian + Otsu`, `CLAHE + Otsu` 결과를 한 장의 비교 이미지로 만들어 어떤 전처리가 다음 검사 단계에 더 유리한지 판단합니다.    |
| `17_day02_challenge.py`     | 스스로 <br>전처리 <br>선택    | 새로운 문제 이미지를 보고 학생이 직접 문제를 판단한 뒤, 필요한 전처리와 파라미터를 선택하여 결과를 만들고 선택 이유와 실패 원인까지 분석하는 Mini Challenge 파일입니다.       |
==`[참고]`== 
OpenCV는 초창기부터 컬러 채널을 기본적으로 `B → G → R` 순서로 사용해 왔습니다. 그래서
```python
image = cv2.imread("image.jpg")
```
로 읽으면 OpenCV 내부에서는 기본적으로 BGR입니다.

![[Pasted image 20261020081151.png]]

- H — Hue (색상): 빨강, 파랑, 초록 등 어떤 색인지를 나타냅니다.
- S — Saturation (채도): 색이 얼마나 선명하거나 흐린지를 나타냅니다.
- V — Value (명도): 색이 얼마나 밝거나 어두운지를 나타냅니다.


---

# 4. 2일차에서 사용할 프로젝트 구조

1일차 프로젝트를 새로 만들지 않습니다.

현재 `subject9_machinevision` 프로젝트를 그대로 사용하고 2일차 폴더와 파일만 추가합니다.

```text
subject9_machinevision/
│
├── .venv/
│
├── data/
│   ├── calibration/
│   ├── train/
│   ├── validation/
│   ├── test/
│   └── challenge/
│
├── samples/
│   ├── sample01.jpg
│   ├── ...
│   ├── sample.mp4
│   │
│   └── day02/
│       ├── cap_on_reference.jpg
│       ├── cap_off_reference.jpg
│       ├── dark.jpg
│       ├── low_contrast.jpg
│       ├── noise.jpg
│       ├── glare.jpg
│       ├── blur.jpg
│       └── challenge.jpg
│
├── outputs/
│   ├── roi/
│   ├── frames/
│   ├── video_roi/
│   ├── live_roi/
│   │   ├── cap_on.jpg
│   │   └── cap_off.jpg
│   │
│   └── day02/
│       ├── color/
│       ├── brightness/
│       ├── gamma/
│       ├── clahe/
│       ├── filter/
│       ├── sharpen/
│       ├── threshold/
│       ├── compare/
│       └── challenge/
│
├── src/
│   ├── 00_check_env.py
│   ├── 01_read_image.py
│   ├── 02_image_info.py
│   ├── 03_roi_crop.py
│   ├── 04_batch_roi.py
│   ├── 05_video_input.py
│   ├── 06_video_roi.py
│   ├── 07_webcam_input.py
│   ├── 08_phone_camera_roi.py
│   ├── 09_color_space.py
│   ├── 10_brightness_contrast.py
│   ├── 11_gamma.py
│   ├── 12_clahe.py
│   ├── 13_filter_compare.py
│   ├── 14_sharpen.py
│   ├── 15_threshold_otsu.py
│   ├── 16_preprocess_compare.py
│   └── 17_day02_challenge.py
│
├── reports/
│   ├── day01_problem_definition.md
│   ├── dataset_roles.csv
│   ├── day02_parameter_table.csv
│   └── day02_preprocess_reason.md
│
├── .gitignore
├── requirements.txt
└── README.md
```

---

# 5. 2일차 실습 데이터 준비하기

## 5.1 1일차에 저장한 ROI를 기준 이미지로 재사용합니다

스마트폰 실시간 ROI 실습을 완료했다면 다음 두 파일이 이미 있습니다.

```text
outputs/live_roi/cap_on.jpg
outputs/live_roi/cap_off.jpg
```

이 파일은 2일차에서 같은 검사 대상이 영상 품질에 따라 어떻게 달라지는지 비교하는 기준 이미지로 사용할 수 있습니다.

먼저 폴더를 만듭니다.

```bash
cd ~/ai_vision/subject9_machinevision

mkdir -p samples/day02
mkdir -p outputs/day02/color
mkdir -p outputs/day02/brightness
mkdir -p outputs/day02/gamma
mkdir -p outputs/day02/clahe
mkdir -p outputs/day02/filter
mkdir -p outputs/day02/sharpen
mkdir -p outputs/day02/threshold
mkdir -p outputs/day02/compare
mkdir -p outputs/day02/challenge
```

1일차의 실시간 ROI 이미지를 복사합니다.

```bash
cp outputs/live_roi/cap_on.jpg samples/day02/cap_on_reference.jpg
cp outputs/live_roi/cap_off.jpg samples/day02/cap_off_reference.jpg
```

---

## 5.2 오늘 사용할 문제 이미지

2일차에서는 1일차에서 확보한 ROI 이미지를 기준으로 전처리 실습을 진행하고, 추가로 직접 촬영한 영상을 이용하여 조명과 촬영환경 변화가 영상 품질에 어떤 영향을 주는지 확인합니다.

### 1일차에서 확보한 ROI 이미지

1일차 스마트폰 실시간 ROI 실습에서 다음 두 이미지를 저장했습니다.

```
samples/day02/
├── cap_on_reference.jpg
└── cap_off_reference.jpg
```

2일차에서 사용하기 쉽도록 `samples/day02/`로 복사했습니다.

이 두 이미지는 1일차에서 스마트폰 실시간 영상에 ROI를 적용하여 저장한 결과입니다.
```
cap_on_reference.jpg
→ 병뚜껑이 있는 ROI
→ 2일차 전처리 실습의 기준 이미지

cap_off_reference.jpg
→ 병뚜껑이 없는 ROI
→ 이후 정상·불량 상태에서
   같은 전처리가 어떻게 적용되는지 비교할 때 사용
```

그리고 1일차 ROI 실습 결과
![[{10FF1C04-80CA-4FE3-B115-A18C752CF276}.png]]

즉 1일차에서:
```
실시간 Camera Frame
        ↓
병뚜껑 ROI 선택
        ↓
cap_on / cap_off 확보
```

까지 했다면, 2일차에서는 이 ROI를 이용하여 영상 품질 변화와 전처리 효과를 확인합니다.

### 직접 촬영하여 확인할 영상 품질 문제

2일차에서는 수강자분들이 실제 머신비전 환경에서도 촬영조건을 관리해야 한다는 것을 이해하기 위해 조명 조건을 바꾸어 추가 이미지를 촬영합니다.

현재 준비한 이미지는 다음과 같습니다.

| 파일              | 촬영 상태                          | 사용 목적                  |
| --------------- | ------------------------------ | ---------------------- |
| `challenge.jpg` | 조명을 약하게 하여 전체적으로 어둡고 대비가 낮은 상태 | 어떤 전처리가 필요한지 학생이 직접 판단 |
| `glare.jpg`     | 한쪽에서 강한 조명을 비춘 상태              | 반사·그림자·포화 등 촬영환경 문제 확인 |
촬영할 때는 가능하면 다음 조건을 유지합니다.

```
같은 제품
    +
같은 카메라 위치
    +
비슷한 촬영 거리
    +
같은 검사 위치

        ↓

조명 조건만 변경
```

이렇게 해야 결과가 달라진 원인이 제품이나 위치 변화가 아니라 영상 품질 변화 때문인지 비교하기 쉽습니다.

![[{CA27C21C-7697-45E9-8699-EFE7CA79E7FD}.png]]

| 이미지    . | 역할                     | 이유                                                             |
| -------- | ---------------------- | -------------------------------------------------------------- |
| 1번째      | `cap_on_reference.jpg` | 비교적 정상적인 밝기와 대비를 가진 기준 이미지                                     |
| 2번째      | `challenge.jpg`        | 전체적으로 어둡고 대비가 낮아 Brightness / Gamma / CLAHE 중 무엇이 필요한지 판단하기 좋음 |
| 3번째      | `glare.jpg`            | 강한 조명 때문에 밝은 영역과 그림자가 생겨 촬영환경 변화가 검사에 미치는 영향을 확인하기 좋음          |

### 직접 촬영할때 예시

![[Pasted image 20260828113541.png]]


### 기준 ROI 이미지에서 문제 이미지 만들기

Brightness, Contrast, Noise, Blur처럼 코드로 명확하게 재현할 수 있는 영상 품질 문제는 별도로 촬영하지 않습니다.

`cap_on_reference.jpg` 하나를 기준으로 코드에서 만들어 사용합니다.

![[Pasted image 20260828113017.png]]

```
cap_on.jpg
    │
    ├── 밝기 감소
    │      ↓
    │   dark.jpg
    │
    ├── 대비 감소
    │      ↓
    │   low_contrast.jpg
    │
    ├── Noise 추가
    │      ↓
    │   noise.jpg
    │
    └── Blur 적용
           ↓
        blur.jpg
```

오늘 실습에서 코드로 만들어 사용할 문제 이미지는 다음과 같습니다.

|파일|만드는 방법|확인할 핵심|
|---|---|---|
|`dark.jpg`|`cap_on.jpg`의 밝기를 낮추어 생성|Brightness, Contrast, Gamma|
|`low_contrast.jpg`|`cap_on.jpg`의 Contrast를 낮추어 생성|CLAHE|
|`noise.jpg`|`cap_on.jpg`에 Noise를 추가하여 생성|Gaussian, Median, Bilateral|
|`blur.jpg`|`cap_on.jpg`에 Blur를 적용하여 생성|Sharpening의 효과와 한계|
이 방법을 사용하는 이유는 중요합니다.

```
원본 제품 동일
ROI 동일
이미지 크기 동일

        ↓

영상 품질 조건만 변경

        ↓

전처리 효과를 정확하게 비교 가능
```

---

# 6. 실습환경 다시 열기

프로젝트 폴더로 이동합니다.

```bash
cd ~/ai_vision/subject9_machinevision
```

현재 위치를 확인합니다.

```bash
pwd
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

정상적으로 활성화되면 터미널 앞에 다음과 같이 표시됩니다.

```text
(.venv)
```

VS Code를 엽니다.

```bash
code -r .
```

1일차에 만든 환경 점검 프로그램을 다시 실행합니다.

```bash
python src/00_check_env.py
```

오류 없이 다음과 같은 결과가 나오면 오늘 실습을 시작합니다.

```text
Python : 3.12.x
OpenCV: 5.x.x
NumPy : 2.x.x
Environment OK
```

> 오늘은 새 가상환경을 만들거나 OpenCV를 다시 설치하지 않습니다.  
> 1일차 환경을 그대로 이어서 사용하는 것이 정상입니다.

---

# 7. Image Enhancement는 왜 필요한가요?

## 7.1 전처리는 이미지를 예쁘게 만드는 작업이 아닙니다

사진 편집에서는 보기 좋은 이미지를 만드는 것이 목적일 수 있습니다.

머신비전에서는 목적이 다릅니다.

```text
사람이 보기 좋은 이미지
         ≠
컴퓨터가 검사하기 좋은 이미지
```

예를 들어 같은 병뚜껑인데 조명이 바뀌었다고 생각해 봅니다.

```text
정상 조명
병뚜껑 밝기 ≈ 180

어두운 조명
병뚜껑 밝기 ≈ 70
```

3일차에 다음과 같은 기준을 사용한다고 가정합니다.

```text
픽셀값 > 120
→ 객체 후보

픽셀값 <= 120
→ 배경 후보
```

같은 정상 병뚜껑인데도 촬영조건 때문에 객체가 사라질 수 있습니다.

```text
조명 변화
    ↓
픽셀값 변화
    ↓
Binary 결과 변화
    ↓
Contour 결과 변화
    ↓
면적·개수 변화
    ↓
잘못된 OK / NG 가능성
```

따라서 전처리의 목적은 다음과 같습니다.

> 촬영조건 때문에 발생한 영상 차이를 줄여서 뒤의 검사 단계가 더 안정적으로 동작하도록 만드는 것

---

## 7.2 그렇다고 모든 문제를 소프트웨어로 고치지는 않습니다

머신비전에서 매우 중요한 원칙입니다.

예를 들어 강한 반사로 픽셀이 완전히 흰색 `255`로 포화되었다면 원래 정보가 이미 사라졌을 수 있습니다.

```text
강한 반사
    ↓
픽셀 포화
    ↓
원래 표면 정보 손실
    ↓
후처리만으로 완전 복구 어려움
```

이런 경우 실제 현장에서는 다음을 먼저 검토할 수 있습니다.

```text
조명 각도 변경
카메라 각도 변경
광량 조절
확산 조명
편광 필터
제품 위치 고정
```

따라서 오늘은 전처리로 해결할 문제와 촬영환경에서 해결할 문제를 구분하는 연습도 합니다.

---

# 8. Color Space를 왜 먼저 확인하나요?

7단락에서는 촬영환경 때문에 이미지의 밝기·대비·Noise 등이 달라질 수 있고, 필요한 경우 Image Enhancement가 필요하다는 것을 확인했습니다.

그런데 이미지를 밝게 하거나 Noise를 제거하기 전에 먼저 결정해야 할 것이 있습니다.

```
이 검사에서 컴퓨터가 어떤 정보를 사용해야 하는가?
```

컬러 이미지 한 장에는 여러 정보가 함께 들어 있습니다.

```
제품 이미지
   ↓
색상 정보
밝기 정보
형태·경계 정보
```

모든 검사에서 이 정보를 전부 사용할 필요는 없습니다.

예를 들어 병뚜껑의 있음 / 없음만 검사한다고 생각해 봅니다.

```
병뚜껑의 색깔
→ 반드시 필요한가?

병뚜껑의 형태와 밝기 차이
→ 검사에 더 중요할 수 있음
```

이 경우에는 컬러 정보를 모두 유지하기보다 `Gray`로 변환하여 밝기와 형태를 중심으로 검사하는 것이 더 단순할 수 있습니다.

반대로 빨간색 스티커가 붙어 있는지 검사한다면:

```
형태만 본다
→ 부족할 수 있음

빨간색인가?
→ 중요한 검사 조건
```

이 경우에는 색상 정보를 분리해서 보기 쉬운 `HSV`가 더 적합할 수 있습니다.

따라서 Color Space를 확인하는 목적은:

```
이미지를 다른 색으로 보이게 만드는 것이 아니고   X

검사에 필요한 정보를
더 쉽게 측정할 수 있는 형태로 바꾸는 것입니다.   O
```

입니다.

---

## 8.1 대량의 이미지가 들어오면 어떻게 확인하나요?

실제 Dataset이 수백 장 또는 수천 장이라고 해서 처음부터 모든 이미지를 사람이 한 장씩 확인하면서 Color Space를 결정하는 것은 아닙니다.

먼저 대표적인 조건의 이미지를 확인합니다.

예를 들어:

```
정상 이미지 몇 장

불량 이미지 몇 장

밝은 조건

어두운 조건

반사가 있는 조건

제품 위치가 조금 다른 조건
```

등을 먼저 확인합니다.

2일차 실습에서도 같은 원리를 사용합니다.

```
cap_on_reference.jpg
→ 기준 영상

challenge.jpg
→ 어둡고 저대비 조건

glare.jpg
→ 강한 조명 조건
```

그리고 코드로 만든:

```
dark.jpg
low_contrast.jpg
noise.jpg
blur.jpg
```

를 함께 비교합니다.

이 자료를 보면서 먼저 다음 질문을 합니다.

```
검사하려는 차이가 무엇인가?

색깔이 중요한가?

밝기 차이가 중요한가?

형태와 경계만 있으면 되는가?

조명이 달라져도
검사에 사용할 정보가 유지되는가?
```

이 판단을 한 뒤 적절한 Color Space를 선택합니다.

> 실제 시스템에서는 이후 Histogram, 평균 밝기, 분산 등의 수치를 이용해 Dataset 전체의 영상 상태를 자동으로 분석할 수도 있습니다. 2일차에서는 먼저 대표 이미지를 직접 비교하며 원리를 이해합니다.

---

## 8.2 Color Space 선택은 전처리의 출발점입니다

전체 흐름을 다음처럼 이해하면 됩니다.

![[Pasted image 20260828132717.png]]

즉 무조건:

```
BGR
→ Gray
→ HSV
```

순서로 변환하는 것이 아닙니다.

세 가지 결과를 한 번 비교해 본 뒤 검사 목적에 맞는 표현을 선택하는 것입니다.

---

## 8.3 BGR — 원본 컬러 정보를 확인합니다

OpenCV의 `cv2.imread()`로 컬러 이미지를 읽으면 기본적으로 BGR 형태로 저장됩니다.

```
B = Blue
G = Green
R = Red
```

일반적으로 많이 사용하는 RGB와 채널 순서만 다릅니다.

```
일반적인 표현
R → G → B

OpenCV 기본
B → G → R
```

BGR은 특별한 검사기법이라기보다 OpenCV에서 읽은 원본 컬러 이미지의 기본 형태라고 이해하면 됩니다.

따라서 먼저 BGR 원본을 보고:

```
제품 색상이 검사에 중요한가?

색상 차이가 정상·불량을 구분하는가?

아니면 색상 없이도 구분 가능한가?
```

를 생각합니다.

---

## 8.4 Gray — 색상이 필요하지 않다면 밝기 정보만 사용합니다

Gray는 B·G·R의 컬러 정보를 하나의 밝기 정보로 변환합니다.

```
BGR
 ↓
Gray
 ↓
밝기 중심의 1채널 이미지
```

예를 들어 병뚜껑 존재 여부처럼:

```
병뚜껑 있음
vs
병뚜껑 없음
```

을 형태나 밝기 차이로 구분할 수 있다면 색상이 반드시 필요하지 않을 수 있습니다.

이런 문제에서는 Gray를 사용하면 이후 처리가 단순해집니다.

```
Gray
 ↓
Brightness / Contrast
 ↓
Threshold
 ↓
Morphology
 ↓
Contour
```

따라서 Gray는 특히 다음과 같은 경우 먼저 생각할 수 있습니다.

```
색상이 검사 기준이 아니다.

형태가 중요하다.

밝기 차이가 중요하다.

Threshold를 사용할 예정이다.

Contour로 면적이나 개수를 측정할 예정이다.
```

현재 병뚜껑 실습은 이쪽에 더 가깝습니다.

---

## 8.5 HSV — 색상 자체가 검사 기준이라면 사용합니다

HSV는 컬러 정보를 다음 세 가지로 분리합니다.

```
H = Hue
→ 어떤 색인가?

S = Saturation
→ 색이 얼마나 선명한가?

V = Value
→ 얼마나 밝은가?
```

예를 들어 제품에 빨간색 스티커가 붙어 있어야 한다고 생각해 봅니다.

```
정상
→ 빨간 스티커 있음

불량
→ 스티커 없음
또는
→ 다른 색 스티커
```

이 경우에는 단순한 Gray만으로는:

```
빨간색
파란색
초록색
```

이라는 색상 차이가 사라집니다.

따라서:

```
BGR 이미지
     ↓
HSV
     ↓
H 범위 확인
     ↓
빨간색 영역 분리
```

와 같은 방식이 더 적합할 수 있습니다.

또한 `V`를 보면 밝기 정보를 별도로 확인할 수 있으므로 조명 변화가 컬러 검사에 미치는 영향을 관찰하는 데도 도움이 됩니다.

---

## 8.6 어떤 Color Space를 선택해야 하나요?

처음부터 정답이 정해져 있는 것은 아닙니다.

검사 문제를 보고 선택합니다.

|검사 문제|먼저 생각할 Color Space|이유|
|---|---|---|
|병뚜껑 있음 / 없음|**Gray**|색상보다 형태·밝기 차이가 중요|
|와셔 개수|**Gray**|객체의 형태와 영역 분리가 중요|
|흠집·경계|**Gray**|밝기와 경계 정보가 중요|
|빨간색 스티커 검사|**HSV**|특정 색상 범위를 찾기 쉬움|
|제품 색상 불량|**HSV**|Hue와 Saturation 비교가 중요|
|원본 색상을 그대로 확인|**BGR**|OpenCV의 기본 컬러 정보 유지|

따라서 Color Space는 기술 이름부터 선택하는 것이 아니라 다음과 같이 결정합니다.

```
무엇을 검사하는가?
       ↓
정상과 불량의 차이가
어떤 정보에 있는가?
       ↓
색상?
밝기?
형태?
       ↓
적절한 Color Space 선택
```

---

## 8.7 이번 병뚜껑 실습에서는 왜 세 가지를 모두 비교하나요?

이번 병뚜껑 실습에서 최종적으로 BGR·Gray·HSV를 모두 사용하는 것이 목적은 아닙니다.

같은 이미지를 세 가지 방식으로 보면서 어떤 정보가 남고 어떤 정보가 사라지는지 직접 확인하기 위해서입니다.

```
같은 병뚜껑 이미지
        ↓
 ┌──────┼──────┐
 ↓      ↓      ↓
BGR    Gray    HSV
 ↓      ↓      ↓
원본    밝기    색상·채도·밝기
컬러    중심    분리
```

비교한 뒤 수강자분이 다음과 같이 판단할 수 있어야 합니다.

![[Pasted image 20260828133347.png]]

```
현재 병뚜껑 존재 검사는
색깔 자체가 핵심은 아니다.

        ↓

형태와 밝기 차이를
측정하는 것이 더 중요하다.

        ↓

Gray를 중심으로
다음 전처리를 진행해 보자.
```

반면 이후 색상 스티커 검사에서는:

```
색상이 핵심이다.
        ↓
HSV가 더 적합한지 확인한다.
```

라고 다른 선택을 할 수 있습니다.

---

## 8.8 Color Space를 선택한 다음에 Image Enhancement를 합니다

이제 7단락과 다음 실습이 자연스럽게 연결됩니다.

```
대표 이미지 확인
        ↓
검사할 특징 확인
        ↓
Color Space 선택
        ↓
영상 품질 문제 확인
        ↓
Brightness / Contrast
Gamma
CLAHE
Filter
Sharpening
        ↓
전처리 전·후 비교
        ↓
Threshold / Otsu
        ↓
다음 검사에 도움이 되는지 판단
```

즉:

> Color Space는 어떤 정보를 검사에 사용할지 결정하는 단계이고, Image Enhancement는 선택한 정보를 검사하기 좋은 상태로 개선하는 단계입니다.


---

# 9. `09_color_space.py` — BGR·Gray·HSV 비교하기

새 파일을 만듭니다.

```text
src/09_color_space.py
```

다음 코드를 작성합니다.

```python
from pathlib import Path

import cv2


image_path = Path("samples/day02/cap_on_reference.jpg")
output_dir = Path("outputs/day02/color")
output_dir.mkdir(parents=True, exist_ok=True)

image = cv2.imread(str(image_path))

if image is None:
    raise FileNotFoundError(
        f"이미지를 읽지 못했습니다: {image_path}"
    )

gray = cv2.cvtColor(
    image,
    cv2.COLOR_BGR2GRAY,
)

hsv = cv2.cvtColor(
    image,
    cv2.COLOR_BGR2HSV,
)

h, s, v = cv2.split(hsv)

cv2.imwrite(
    str(output_dir / "original_bgr.jpg"),
    image,
)

cv2.imwrite(
    str(output_dir / "gray.jpg"),
    gray,
)

cv2.imwrite(
    str(output_dir / "h_channel.jpg"),
    h,
)

cv2.imwrite(
    str(output_dir / "s_channel.jpg"),
    s,
)

cv2.imwrite(
    str(output_dir / "v_channel.jpg"),
    v,
)

print("입력 :", image_path)
print("저장 :", output_dir)
print("Color Space 결과 저장 완료")
```

실행합니다.

```bash
python src/09_color_space.py
```

결과를 확인합니다.

```text
outputs/day02/color/
├── original_bgr.jpg
├── gray.jpg
├── h_channel.jpg
├── s_channel.jpg
└── v_channel.jpg
```

최종 결과 예시 

![[{5FB49F5D-C794-4233-8448-E62D7943F4AC}.png]]

현재 결과는 이렇게 해석하면 됩니다.

| 결과                 | 현재 보이는 모습              | 해석                              |
| ------------------ | ---------------------- | ------------------------------- |
| `original_bgr.jpg` | 원래 병뚜껑 컬러 이미지          | OpenCV가 읽은 원본 BGR 영상            |
| `gray.jpg`         | 흑백이지만 병뚜껑 형태가 잘 보임     | 색상을 제거하고 밝기 중심 정보만 남긴 결과        |
| `h_channel.jpg`    | 회색 영역이 여러 덩어리처럼 보임     | 픽셀의 색상 종류(Hue)만 표현한 결과          |
| `s_channel.jpg`    | 대부분 어둡고 병뚜껑 일부만 밝음     | 색이 얼마나 선명한지 나타내는 채도(Saturation) |
| `v_channel.jpg`    | 원본 Gray와 비슷하게 형태가 잘 보임 | HSV에서 밝기(Value)만 나타낸 결과         |
### 결과 해석

```
- BGR: 원본 컬러 이미지입니다.
- Gray: 색상 정보를 제거하고 밝기 차이만 남겨 병뚜껑 형태가 잘 보입니다.
- H 채널: 색상 종류 정보인데, 현재 병뚜껑이 검정·회색 계열이라 활용도가 낮습니다.
- S 채널: 채도 정보로, 무채색에 가까워 전체적으로 어둡게 나타납니다.
- V 채널: 밝기 정보로, 병뚜껑과 배경의 밝기 차이가 비교적 잘 보입니다.
```

따라서 현재 병뚜껑 존재 여부 검사는 색상보다 형태와 밝기 차이가 중요하므로 Gray 중심으로 다음 전처리를 진행하는 것이 적절합니다.

## 9.1 의사코드

```text
1. 기준 이미지를 읽는다.

2. 결과를 저장할 폴더를 준비한다.

3. 이미지를 정상적으로 읽었는지 확인한다.

4. BGR 이미지를 Gray로 변환한다.

5. BGR 이미지를 HSV로 변환한다.

6. HSV를 H, S, V 세 채널로 분리한다.

7. 원본, Gray, H, S, V 결과를 각각 저장한다.

8. 저장된 이미지를 직접 열어 차이를 비교한다.
```

## 9.2 결과에서 확인할 질문

```text
Gray에서는 어떤 정보가 사라졌는가?

H 채널에서는 병뚜껑의 색상 차이가 보이는가?

S 채널은 색의 선명도 차이를 보여주는가?

V 채널은 밝기 변화에 어떻게 반응하는가?

병뚜껑 존재 여부만 검사한다면 색상이 꼭 필요한가?

색상 스티커를 검사한다면 Gray만으로 충분할까?
```

---

# 10. Brightness와 Contrast는 왜 조절하나요?

앞에서는 같은 이미지를 `BGR`, `Gray`, `HSV`로 비교하면서 검사에 어떤 정보가 필요한지 확인했습니다.

현재 병뚜껑 존재 검사에서는 색상 자체보다 형태와 밝기 차이가 중요하다고 판단했으므로, 이후에는 주로 Gray와 밝기 정보를 이용해 검사를 진행합니다.

그런데 같은 병뚜껑이라도 촬영할 때 조명이 달라지면 이미지의 밝기가 달라질 수 있습니다.

```text
같은 병뚜껑

정상 조명
→ 병뚜껑과 배경의 차이가 잘 보임

조명이 너무 어두움
→ 병뚜껑과 배경이 모두 어두워짐

대비가 부족함
→ 병뚜껑과 배경의 밝기 차이가 작아짐
```

이 상태로 다음 단계인 Threshold를 적용하면 같은 제품인데도 결과가 달라질 수 있습니다.

```text
촬영환경 변화
      ↓
밝기·대비 변화
      ↓
픽셀값 변화
      ↓
Threshold 결과 변화
      ↓
객체 영역이 제대로 분리되지 않을 수 있음
```

따라서 먼저 다음 질문을 합니다.
현재 이미지가 너무 어둡거나, 제품과 배경의 밝기 차이가 부족한가?
문제가 있다면 Brightness와 Contrast를 조절해 봅니다.

---

## 10.1 Brightness와 Contrast의 차이

### Brightness — 이미지 전체의 밝기를 조절합니다

Brightness를 높이면 전체 픽셀이 밝아지는 방향으로 이동합니다.

```
어두운 이미지

████████

      ↓ Brightness 증가

▓▓▓▓▓▓▓▓

      ↓

검사 대상이 더 잘 보일 수 있음
```

![[Pasted image 20260828135220.png]]

예를 들어 어두운 조건에서 촬영된 `dark.jpg`에 사용할 수 있습니다.

하지만 너무 많이 밝게 만들면 흰색에 가까운 영역의 정보가 사라질 수 있습니다.

---
### Contrast — 밝은 곳과 어두운 곳의 차이를 조절합니다

Contrast는 단순히 전체를 밝게 하는 것이 아니라 밝은 부분과 어두운 부분의 차이를 크게 또는 작게 만듭니다.

예를 들어:

![[Pasted image 20260828135844.png]]

```
낮은 Contrast

배경   = 120
제품   = 135

차이   = 15
```

제품과 배경이 비슷하게 보입니다.

Contrast를 높이면:

```
배경   → 더 어둡게
제품   → 더 밝게

        ↓

제품과 배경의 차이가 커질 수 있음
```

따라서 객체와 배경이 비슷한 밝기를 가지고 있을 때 Contrast 조정이 도움이 될 수 있습니다.

---

## 10.2 Brightness와 Contrast는 코드에서 어떻게 조절하나요?

OpenCV에서는 다음 관계로 단순하게 이해할 수 있습니다.

```
output = input × alpha + beta
```

여기서:

```
alpha
→ Contrast에 주로 영향

beta
→ Brightness에 주로 영향
```

예를 들어:

```
alpha = 1.0
beta = 40
```

이면 원래 픽셀값에 `40` 정도를 더하는 효과가 있으므로 전체적으로 밝아집니다.

```
원본
70  100  150

       ↓ beta +40

110  140  190
```

반면:

```
alpha = 1.5
beta = 0
```

이면 픽셀값의 차이가 더 크게 벌어지므로 Contrast가 증가합니다.

```
원본
70   100   150

       ↓ × 1.5

105   150   225
```

![[Pasted image 20260828140450.png]]

---

## 10.3 많이 조절하면 더 좋은가요?

그렇지 않습니다.

이미지 픽셀값은 일반적으로 다음 범위를 사용합니다.

```
0
→ 완전한 검정

255
→ 완전한 흰색
```

Brightness나 Contrast를 너무 강하게 적용하면 많은 픽셀이 `255`에 가까워질 수 있습니다.

```
원래 밝은 영역
220
230
240

      ↓ 과도한 밝기 증가

255
255
255
```

이렇게 되면 서로 달랐던 부분이 모두 흰색으로 바뀌면서 **세부정보를 구분할 수 없게 됩니다.**

즉:

```
밝게 만들었다
≠
검사하기 좋아졌다
```

입니다.

---

## 10.4 이번 실습에서는 무엇을 확인하나요?

이번에는 `dark.jpg`에 여러 `alpha`, `beta` 값을 적용하여 결과를 직접 비교합니다.

```
dark.jpg
    ↓
Brightness / Contrast 변경
    ↓
여러 결과 저장
    ↓
비교
```

결과를 볼 때 단순히:

```
어느 이미지가 가장 밝은가?
```

를 보는 것이 아닙니다.

다음 질문으로 판단합니다.

```
병뚜껑이 더 잘 구분되는가?

병뚜껑의 홈과 경계가 유지되는가?

밝은 영역이 하얗게 날아가지는 않았는가?

Noise까지 함께 강조되지는 않았는가?

다음 Threshold에서
객체와 배경을 분리하기 쉬워졌는가?
```

즉 이번 실습의 목적은 이미지를 밝게 만드는 것 자체가 아니라, 검사에 도움이 되는 적절한 밝기와 대비를 찾는 것입니다.

전체 흐름은 다음과 같습니다.

```
Color Space 선택
      ↓
Gray 중심으로 검사하기로 판단
      ↓
영상 품질 확인
      ↓
너무 어두운가?
대비가 부족한가?
      ↓
Brightness / Contrast 조절
      ↓
여러 Parameter 비교
      ↓
다음 검사에 도움이 되는 값 선택
```

---

# 11. `10_brightness_contrast.py` — 밝기·대비 비교하기

새 파일을 만듭니다.

```text
src/10_brightness_contrast.py
```

```python
from pathlib import Path

import cv2


# 1. 1일차에서 만든 기준 ROI
reference_path = Path(
    "samples/day02/cap_on_reference.jpg"
)

# 2. 오늘 사용할 문제 이미지
dark_path = Path(
    "samples/day02/dark.jpg"
)

# 3. 결과 저장 위치
output_dir = Path(
    "outputs/day02/brightness"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# 기준 이미지 읽기
reference = cv2.imread(
    str(reference_path)
)

if reference is None:
    raise FileNotFoundError(
        f"기준 이미지를 읽지 못했습니다: {reference_path}"
    )


# --------------------------------------------------
# 실습용 어두운 이미지 만들기
# --------------------------------------------------

dark = cv2.convertScaleAbs(
    reference,
    alpha=0.55,
    beta=0,
)

cv2.imwrite(
    str(dark_path),
    dark,
)

print("문제 이미지 생성:", dark_path)


# --------------------------------------------------
# Brightness / Contrast 비교
# --------------------------------------------------

settings = [
    (1.0, 0),
    (1.0, 20),
    (1.0, 50),
    (1.0, 80),
]


for alpha, beta in settings:

    result = cv2.convertScaleAbs(
        dark,
        alpha=alpha,
        beta=beta,
    )

    filename = (
        f"alpha_{alpha}_beta_{beta}.jpg"
    )

    output_path = (
        output_dir / filename
    )

    cv2.imwrite(
        str(output_path),
        result,
    )

    print("저장:", output_path)
```

실행합니다.

```bash
python src/10_brightness_contrast.py
```

## 11.1 의사코드

```text
1. 1일차에서 만든 병뚜껑 기준 ROI 이미지의 경로를 정한다.

2. 실습용으로 만들 어두운 이미지의 저장 경로를 정한다.

3. Brightness 비교 결과를 저장할 폴더를 준비한다.

4. 기준 ROI 이미지를 읽는다.

5. 이미지가 정상적으로 읽히지 않았다면 오류를 발생시킨다.

6. 기준 이미지의 밝기를 약 55% 수준으로 낮춘다.

7. 어둡게 만든 이미지를 dark.jpg로 저장한다.

8. Brightness를 비교할 beta 값을 준비한다.

   alpha는 1.0으로 고정하고

   beta = 0
   beta = 20
   beta = 50
   beta = 80

   순서로 사용한다.

9. dark.jpg에 첫 번째 alpha, beta 값을 적용한다.

10. 변환된 결과 이미지를 저장한다.

11. 나머지 beta 값도 같은 방식으로 반복 적용한다.

12. 모든 결과 이미지를 각각 다른 파일명으로 저장한다.

13. 저장된 결과를 비교하여
    beta가 커질수록 이미지 전체가 어떻게 밝아지는지 확인한다.

14. 너무 밝게 만든 경우
    병뚜껑의 경계나 밝은 영역의 세부정보가 사라지는지도 함께 확인한다.
```

## 11.2 Brightness 결과를 비교합니다

![[{94CBAAE1-541F-4FA9-8753-59D42CE60C4F}.png]]

이번 실습에서는 `alpha = 1.0`으로 고정하고 `beta` 값만 변경했습니다.

따라서 이번 결과에서는 Contrast를 비교하는 것이 아니라 Brightness가 증가할 때 이미지가 어떻게 달라지는지 확인합니다.

```
alpha = 1.0 고정

beta = 0
   ↓
beta = 20
   ↓
beta = 50
   ↓
beta = 80

전체 이미지가 점점 밝아짐
```

실제 결과를 비교하면 다음과 같습니다.

| alpha | beta | 결과 해석                                                     |
| ----: | ---: | --------------------------------------------------------- |
|   1.0 |    0 | 어둡게 만든 기준 이미지입니다. 병뚜껑의 어두운 부분이 많이 묻혀 있습니다.                |
|   1.0 |   20 | 전체가 조금 밝아져 병뚜껑의 형태와 홈이 조금 더 잘 보입니다.                       |
|   1.0 |   50 | 밝기가 더 증가하면서 병뚜껑과 주변 영역을 비교하기 쉬워집니다.                       |
|   1.0 |   80 | 전체가 상당히 밝아집니다. 병뚜껑은 잘 보이지만 밝은 배경과 투명 병 부분은 너무 밝아지기 시작합니다. |

### 결과에서 확인할 핵심

Brightness는 값을 크게 할수록 무조건 좋은 것이 아닙니다.

```
너무 어두움
→ 검사 대상이 잘 보이지 않음

적절한 밝기
→ 형태와 경계가 잘 보임

너무 밝음
→ 밝은 부분이 흰색에 가까워짐
→ 세부정보가 사라질 수 있음
```

따라서 이번 실습에서는 가장 밝은 이미지를 선택하는 것이 목적이 아닙니다.
==`병뚜껑의 형태와 경계는 잘 보이면서 밝은 영역의 정보가 지나치게 사라지지 않는 값을 찾는 것이 목적입니다.`==

현재 결과로 보면 
`beta=20~50` 정도가 비교적 자연스럽고, `beta=80`은 과도하게 밝게 했을 때 어떤 문제가 생길 수 있는지를 확인하기 위한 비교값으로 사용하기 좋습니다.

## 11.3 결과를 기록합니다

이번 실습에서는 `alpha = 1.0`으로 고정하고 `beta` 값만 변경하여 전체 밝기 변화를 비교했습니다.

결과를 확인한 뒤 다음 파일에 기록합니다.

```
reports/day02_preprocess_reason.md
```

| alpha | beta | 결과 해석                                           |
| ----: | ---: | ----------------------------------------------- |
|   1.0 |    0 | 가장 어두운 기준 이미지입니다. 병뚜껑의 홈과 경계가 상대적으로 잘 보이지 않습니다. |
|   1.0 |   20 | 전체가 조금 밝아지면서 병뚜껑의 형태가 좀 더 잘 보입니다.               |
|   1.0 |   50 | 밝기가 충분히 증가하여 병뚜껑과 주변 영역을 비교하기 쉬워졌습니다.           |
|   1.0 |   80 | 전체가 많이 밝아졌으며 밝은 배경과 투명 병 부분의 세부정보가 약해지기 시작합니다.  |

02_preprocess_reason.md 기록 작성 예시

```markdown
## 1. Brightness 비교

### 사용 이미지

dark.jpg

### 비교 조건

alpha = 1.0 고정

beta = 0, 20, 50, 80

### 결과

| alpha | beta | 결과 해석 |
|---:|---:|---|
| 1.0 | 0 | 가장 어두운 기준 이미지이다. 병뚜껑의 홈과 경계가 상대적으로 잘 보이지 않는다. |
| 1.0 | 20 | 전체가 조금 밝아지면서 병뚜껑 형태가 좀 더 잘 보인다. |
| 1.0 | 50 | 밝기가 충분히 증가하여 병뚜껑과 주변 영역을 비교하기 쉬워졌다. |
| 1.0 | 80 | 전체가 많이 밝아졌으며 밝은 배경과 투명 병 부분의 세부정보가 약해지기 시작한다. |

### 판단

현재 결과에서는 beta = 20~50 정도가 비교적 자연스럽게 보인다.

가장 밝은 값을 선택하는 것이 목적이 아니라,
병뚜껑의 형태와 경계가 잘 보이면서
밝은 영역의 세부정보가 지나치게 사라지지 않는 값을 선택해야 한다.
```

실습후 다음과 같이 순서로 기록합니다.

```
1. beta=0, 20, 50, 80 결과 이미지를 모두 확인합니다.

2. 각 결과에서 다음을 비교합니다.

   - 병뚜껑의 형태가 더 잘 보이는가?
   - 병뚜껑의 홈과 경계가 유지되는가?
   - 배경이 너무 밝아지지는 않았는가?
   - 밝은 부분의 세부정보가 사라지지는 않았는가?

3. 각 파라미터의 장점과 단점을 기록합니다.

4. 다음 Threshold 검사에 사용할 만한 결과인지 판단합니다.

5. 판단 결과를
   reports/day02_parameter_table.csv
   파일에 기록합니다.
```

### `day02_parameter_table.csv`에는 무엇을 기록하나요?

기준은 다음 7가지입니다.

| 항목                        | 무엇을 기록하나           |
| ------------------------- | ------------------ |
| `image`                   | 어떤 문제 이미지를 사용했는가   |
| `problem`                 | 이미지에 어떤 문제가 있었는가   |
| `method`                  | 어떤 전처리를 사용했는가      |
| `parameter`               | 어떤 파라미터를 사용했는가     |
| `improved`                | 무엇이 좋아졌는가          |
| `worse`                   | 무엇이 나빠졌는가          |
| `next_inspection_helpful` | 다음 검사에 도움이 될 것 같은가 |

그리고 CSV `reports/day02_parameter_table.csv` 를 예를 들어 이렇게 작성할수 있습니다.

```csv
image,problem,method,parameter,improved,worse,next_inspection_helpful
dark.jpg,dark,Brightness,beta=0,기준 이미지,전체적으로 어두워 병뚜껑 세부가 잘 보이지 않음,no
dark.jpg,dark,Brightness,beta=20,병뚜껑 형태가 조금 더 잘 보임,큰 변화는 적음,yes
dark.jpg,dark,Brightness,beta=50,병뚜껑과 주변 영역을 확인하기 쉬워짐,밝은 영역도 함께 증가,yes
dark.jpg,dark,Brightness,beta=80,병뚜껑이 매우 잘 보임,배경과 밝은 부분의 세부정보가 감소하기 시작함,partly
```

`day02_parameter_table.csv`는 코드가 자동으로 만드는 파일이 아닙니다.  
전처리 결과 이미지를 직접 비교한 뒤, 사용한 파라미터와 좋아진 점·나빠진 점·다음 검사에 사용할지 여부를 학생이 직접 기록하는 실험 기록표입니다.

파일들의 각각 역할은 다음과 같습니다.

```
Python 코드
→ 결과 이미지 생성

day02_parameter_table.csv
→ 파라미터별 결과를 짧게 정리

day02_preprocess_reason.md
→ 왜 그 방법을 선택했는지 상세하게 설명
```

CSV는 자동생성 할 수 있지만, 자동으로 저장해 버리면 수강자분이 이미지를 보고 판단하는 과정이 빠질 수 있기 때문에 직접 수작업으로 기록하는 훈련이 필요합니다.

다만 `alpha`, `beta`, 파일명, 평균 밝기 같은 객관적인 수치만 Python에서 자동으로 CSV에 저장하고, `좋아진 점/나빠진 점/사용 여부`는 수강자분이 직접 작성하게 만드는 방식으로 개선하는 것도 가능합니다.

---

# 12. Gamma Correction 이해하기

Brightness는 모든 픽셀에 비슷한 양을 더하는 방식으로 생각할 수 있습니다.

Gamma Correction은 밝기 분포를 **비선형적으로** 조절합니다.

따라서 어두운 중간 영역을 더 잘 보이게 하거나 반대로 너무 밝은 영상을 낮출 때 사용할 수 있습니다.

오늘 사용하는 코드에서는 다음 규칙을 따릅니다.

```text
gamma > 1.0
→ 더 밝아지는 방향

gamma < 1.0
→ 더 어두워지는 방향
```

> Gamma 구현 방법에 따라 표기 방향이 반대로 설명되는 자료도 있습니다.  
> 오늘 코드는 `1 / gamma`를 사용하므로 위 규칙으로 결과를 해석합니다.

![[Pasted image 20260828143953.png]]

---

# 13. `11_gamma.py` — 여러 Gamma 값 비교하기

앞에서 Brightness 실습을 하면서 `cap_on_reference.jpg`를 일부러 어둡게 만든 다음 다음 파일로 저장했습니다.

```
samples/day02/dark.jpg
```

이번에는 이 같은 `dark.jpg`를 이용하여 Gamma Correction이 어두운 영역을 어떻게 변화시키는지 확인합니다.

Brightness는 픽셀값에 일정한 값을 더해 이미지 전체를 밝게 만드는 방식이었다면, Gamma는 밝기 분포를 곡선 형태로 조절합니다.

```
Brightness
→ 전체 픽셀을 비슷한 방향으로 이동

Gamma
→ 어두운 영역·중간 밝기 영역을
   비선형적으로 변화
```

이번 코드에서는 다음 값을 비교합니다.

```
gamma = 0.6
gamma = 0.8
gamma = 1.0   ← 기준
gamma = 1.2
gamma = 1.5
gamma = 2.0
```

오늘 사용하는 구현에서는:

```
gamma < 1.0
→ 더 어두워짐

gamma = 1.0
→ 변화 없음

gamma > 1.0
→ 더 밝아짐
```

입니다.

> 이번 실습의 목적은 가장 밝은 Gamma 값을 찾는 것이 아니라, 어두운 영역은 충분히 보이면서 밝은 영역의 정보는 지나치게 손실되지 않는 값을 찾는 것입니다.

---

## 13.1 Gamma 비교 코드 작성하기

새 파일을 만듭니다.

```
src/11_gamma.py
```

다음 코드를 작성합니다.

```python
from pathlib import Path

import cv2
import numpy as np


# --------------------------------------------------
# 사용할 이미지 경로
# --------------------------------------------------

reference_path = Path(
    "samples/day02/cap_on_reference.jpg"
)

dark_path = Path(
    "samples/day02/dark.jpg"
)

output_dir = Path(
    "outputs/day02/gamma"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# dark.jpg 확인
# --------------------------------------------------

image = cv2.imread(
    str(dark_path)
)


# dark.jpg가 없다면
# 기준 ROI 이미지에서 자동으로 생성
if image is None:

    print(
        "dark.jpg가 없어 기준 이미지에서 생성합니다."
    )

    reference = cv2.imread(
        str(reference_path)
    )

    if reference is None:
        raise FileNotFoundError(
            f"기준 이미지를 읽지 못했습니다: "
            f"{reference_path}"
        )

    image = cv2.convertScaleAbs(
        reference,
        alpha=0.55,
        beta=0,
    )

    cv2.imwrite(
        str(dark_path),
        image,
    )

    print(
        "문제 이미지 생성:",
        dark_path,
    )


# --------------------------------------------------
# Gamma Correction 함수
# --------------------------------------------------

def adjust_gamma(
    source: np.ndarray,
    gamma: float,
) -> np.ndarray:

    if gamma <= 0:
        raise ValueError(
            "gamma는 0보다 커야 합니다."
        )

    inv_gamma = 1.0 / gamma

    table = np.array(
        [
            (
                (value / 255.0)
                ** inv_gamma
            ) * 255
            for value in np.arange(256)
        ],
        dtype=np.uint8,
    )

    return cv2.LUT(
        source,
        table,
    )


# --------------------------------------------------
# 여러 Gamma 값 비교
# --------------------------------------------------

gamma_values = [
    0.6,
    0.8,
    1.0,
    1.2,
    1.5,
    2.0,
]


for gamma in gamma_values:

    result = adjust_gamma(
        image,
        gamma,
    )

    output_path = (
        output_dir
        / f"gamma_{gamma}.jpg"
    )

    cv2.imwrite(
        str(output_path),
        result,
    )

    print(
        "저장:",
        output_path,
    )
```

실행합니다.

```bash
python src/11_gamma.py
```

정상적으로 실행되면 다음과 같은 결과가 만들어집니다.

```
outputs/day02/gamma/
├── gamma_0.6.jpg
├── gamma_0.8.jpg
├── gamma_1.0.jpg
├── gamma_1.2.jpg
├── gamma_1.5.jpg
└── gamma_2.0.jpg
```

전체 데이터 흐름은 다음과 같습니다.

```
cap_on_reference.jpg
        ↓
10_brightness_contrast.py에서
일부러 어둡게 생성
        ↓
dark.jpg
        ↓
11_gamma.py
        ↓
Gamma 값 변경
        ↓
0.6 / 0.8 / 1.0 / 1.2 / 1.5 / 2.0
        ↓
결과 이미지 비교
```

## 13.2 의사코드

```text
1. Gamma 결과를 저장할 폴더를 준비한다.

2. dark.jpg를 읽어본다.

3. dark.jpg가 존재하는지 확인한다.

4. dark.jpg가 없다면
   cap_on_reference.jpg를 읽는다.

5. 기준 이미지의 밝기를 낮춰
   dark.jpg를 자동으로 만든다.

6. Gamma Correction 함수를 준비한다.

7. Gamma 값이 0보다 큰지 확인한다.

8. 0~255의 픽셀값을
   Gamma 공식으로 변환하는
   Lookup Table을 만든다.

9. Gamma 값을 여러 개 준비한다.

   0.6
   0.8
   1.0
   1.2
   1.5
   2.0

10. 첫 번째 Gamma 값을
    dark.jpg에 적용한다.

11. 결과 이미지를 저장한다.

12. 나머지 Gamma 값도
    같은 방법으로 반복한다.

13. 모든 결과 이미지를 비교한다.

14. 어두운 영역은 잘 보이면서
    밝은 영역의 정보가
    지나치게 사라지지 않는지 확인한다.
```

## 13.3 결과는 무엇을 비교하나요?

### Gamma 결과 비교

![[{B9BB3127-6491-4596-8A8A-7380CA54C359}.png]]

|   Gamma | 실제 결과 해석                                         | 판단               |
| ------: | ------------------------------------------------ | ---------------- |
| **0.6** | 전체가 매우 어두워져 병뚜껑의 홈과 세부 형태가 거의 보이지 않습니다.          | 너무 어두워 검사에 불리함   |
| **0.8** | `0.6`보다 조금 밝아졌지만 여전히 병뚜껑의 어두운 영역이 많이 묻혀 있습니다.    | 다소 어두움           |
| **1.0** | `dark.jpg`와 동일한 기준 이미지입니다.                       | 비교 기준            |
| **1.2** | 병뚜껑의 세로 홈과 경계가 조금 더 잘 보이기 시작합니다.                 | 개선 효과 있음         |
| **1.5** | 어두운 영역이 더 살아나면서 병뚜껑 형태가 비교적 명확하게 보입니다.           | 현재 결과에서 비교적 적절   |
| **2.0** | 전체가 상당히 밝아져 병뚜껑 세부는 잘 보이지만 배경과 밝은 영역도 많이 밝아졌습니다. | 과도한 밝기 가능성 확인 필요 |
현재 결과 흐름을 보면

```
gamma = 0.6
→ 너무 어두움

gamma = 0.8
→ 조금 밝아짐

gamma = 1.0
→ 기준

gamma = 1.2
→ 어두운 영역 개선 시작

gamma = 1.5
→ 병뚜껑 경계와 홈이 비교적 잘 보임

gamma = 2.0
→ 많이 밝아짐
→ 밝은 영역까지 과도하게 증가할 가능성
```

먼저 `gamma=1.0`을 기준으로 봅니다.

```text
병뚜껑 경계가 더 잘 보이는가?

어두운 영역의 정보가 살아났는가?

밝은 부분이 너무 하얗게 변했는가?

배경 Noise도 함께 강조되었는가?

이 결과가 Threshold에 도움이 될 것 같은가?
```

### Gamma는 무엇을 기준으로 선택하나요?

`잘 보인다`의 기준은 사람이 보기 좋은지가 아니라 다음 검사 단계가 잘 되는가입니다.

예를 들어 병뚜껑 검사라면:

```
Gamma 후보
0.6 / 0.8 / 1.0 / 1.2 / 1.5 / 2.0
        ↓
각 이미지에 동일한 Threshold / Otsu 적용
        ↓
병뚜껑 영역이 제대로 분리되는지 비교
        ↓
Contour 추출
        ↓
면적·경계가 안정적으로 측정되는지 비교
        ↓
가장 안정적인 Gamma 선택
```

따라서 판단 기준은 다음과 같습니다.

| 판단 기준        | 확인하는 내용                        |
| ------------ | ------------------------------ |
| 경계 보존        | 병뚜껑의 외곽과 홈이 사라지지 않았는가          |
| 어두운 영역 정보    | 너무 검게 뭉개진 부분이 없는가              |
| 밝은 영역 정보     | 흰색으로 날아간 부분이 없는가               |
| Noise        | 밝게 만들면서 불필요한 Noise까지 증가하지 않았는가 |
| Threshold 결과 | 객체와 배경이 더 깨끗하게 분리되는가           |
| Contour 결과   | 객체의 면적·형태가 안정적으로 측정되는가         |

만약 원본 상태가 이미 검사에 가장 적합하다고 판단했다면 
==`Gamma Correction을 사용하지 않는다.`== 가 올바른 결론입니다.

---

# 14. 저대비 영상과 CLAHE

## 14.1 Contrast가 낮다는 것은 무엇인가요?

예를 들어 배경과 제품의 밝기가 다음과 같다고 생각합니다.

```text
배경 밝기 ≈ 120
제품 밝기 ≈ 130
```

차이가 작으면 사람 눈에는 형태가 보이더라도 컴퓨터가 단순한 밝기 기준으로 분리하기 어려울 수 있습니다.

CLAHE는 이미지를 작은 영역으로 나누어 **지역별 Contrast를 개선**하는 방법입니다.

```text
Low Contrast ROI
       ↓
CLAHE
       ↓
지역별 대비 개선
       ↓
경계·문자·형태가
더 잘 구분될 수 있음
```

하지만 너무 강하게 적용하면 배경 Texture와 Noise도 함께 강조될 수 있습니다.

![[Pasted image 20260828145756.png]]

---

# 15. `12_clahe.py` — 저대비 영상 만들고 CLAHE로 개선하기

앞의 Brightness 실습에서는 기준 ROI를 일부러 어둡게 만들어 `dark.jpg`를 생성했습니다.

이번 실습도 같은 방식으로 진행합니다.

먼저 1일차에서 확보한 기준 ROI:

```
samples/day02/cap_on_reference.jpg
```

를 이용하여 제품과 배경의 밝기 차이를 일부러 줄인 저대비 이미지를 만듭니다.

```text
cap_on_reference.jpg
        ↓
Contrast를 낮춤
        ↓
low_contrast.jpg
        ↓
Gray 변환
        ↓
CLAHE 적용
        ↓
clipLimit별 결과 비교
```

이렇게 만든 `low_contrast.jpg`에 CLAHE를 적용하여 지역적인 밝기 차이를 다시 구분하기 쉽게 만들 수 있는지 확인합니다.

## 15.1 코드 작성하기

새 파일을 만듭니다.

```
src/12_clahe.py
```

다음 코드를 작성합니다.

```python
from pathlib import Path

import cv2


# --------------------------------------------------
# 사용할 이미지 경로
# --------------------------------------------------

reference_path = Path(
    "samples/day02/cap_on_reference.jpg"
)

low_contrast_path = Path(
    "samples/day02/low_contrast.jpg"
)

output_dir = Path(
    "outputs/day02/clahe"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 기준 ROI 이미지 읽기
# --------------------------------------------------

reference = cv2.imread(
    str(reference_path)
)

if reference is None:
    raise FileNotFoundError(
        f"기준 이미지를 읽지 못했습니다: "
        f"{reference_path}"
    )


# --------------------------------------------------
# 실습용 저대비 이미지 만들기
# --------------------------------------------------

low_contrast = cv2.convertScaleAbs(
    reference,
    alpha=0.35,
    beta=80,
)

cv2.imwrite(
    str(low_contrast_path),
    low_contrast,
)

print(
    "문제 이미지 생성:",
    low_contrast_path,
)


# --------------------------------------------------
# Gray 변환
# --------------------------------------------------

reference_gray = cv2.cvtColor(
    reference,
    cv2.COLOR_BGR2GRAY,
)

low_contrast_gray = cv2.cvtColor(
    low_contrast,
    cv2.COLOR_BGR2GRAY,
)


# 기준 이미지와 저대비 이미지 저장
cv2.imwrite(
    str(
        output_dir
        / "reference_gray.jpg"
    ),
    reference_gray,
)

cv2.imwrite(
    str(
        output_dir
        / "low_contrast_gray.jpg"
    ),
    low_contrast_gray,
)


# --------------------------------------------------
# CLAHE 설정
# --------------------------------------------------

settings = [
    (1.0, (8, 8)),
    (2.0, (8, 8)),
    (3.0, (8, 8)),
    (4.0, (8, 8)),
]


# --------------------------------------------------
# 여러 clipLimit 비교
# --------------------------------------------------

for clip_limit, tile_size in settings:

    clahe = cv2.createCLAHE(
        clipLimit=clip_limit,
        tileGridSize=tile_size,
    )

    result = clahe.apply(
        low_contrast_gray
    )

    output_path = (
        output_dir
        / f"clahe_clip_{clip_limit}.jpg"
    )

    cv2.imwrite(
        str(output_path),
        result,
    )

    print(
        "저장:",
        output_path,
    )
```

실행합니다.

```bash
python src/12_clahe.py
```

---
## 15.1 의사코드

```text
1. 1일차에서 만든
   cap_on_reference.jpg를 읽는다.

2. 기준 이미지가 정상적으로
   읽혔는지 확인한다.

3. 기준 이미지의 Contrast를 낮춘다.

4. 저대비 결과를
   low_contrast.jpg로 저장한다.

5. 기준 이미지와
   저대비 이미지를 Gray로 변환한다.

6. 두 이미지를 저장하여
   Contrast가 실제로 낮아졌는지 비교한다.

7. CLAHE에서 사용할
   clipLimit 값을 준비한다.

   1.0
   2.0
   3.0
   4.0

8. tileGridSize는 8×8로 고정한다.

9. low_contrast_gray에
   첫 번째 CLAHE 설정을 적용한다.

10. 결과 이미지를 저장한다.

11. 나머지 clipLimit도
    같은 방법으로 반복한다.

12. 저대비 원본과
    CLAHE 결과들을 비교한다.

13. 객체의 경계가 좋아졌는지 확인한다.

14. 동시에 배경 Texture나 Noise도
    지나치게 강조되지 않았는지 확인한다.

15. 다음 Threshold/Otsu에
    어떤 결과가 더 유리한지 이후 비교한다.
```

## 15.3 CLAHE 결과에서 확인할 것

![[Pasted image 20260828152558.png]]

이번 결과에서는 `clipLimit`가 커질수록 지역 대비가 점차 강해지는 것을 확인할 수 있습니다.

| clipLimit | 결과 해석                                                          |
| --------: | -------------------------------------------------------------- |
|   **1.0** | 가장 부드러운 결과입니다. 병뚜껑의 홈과 경계가 보이지만 대비 개선 효과는 비교적 약합니다.            |
|   **2.0** | 병뚜껑의 세로 홈과 경계가 조금 더 분명하게 보입니다.                                 |
|   **3.0** | 홈과 경계가 더욱 강조되어 형태를 구분하기 쉬워집니다.                                 |
|   **4.0** | 가장 강한 대비가 나타납니다. 병뚜껑의 홈은 잘 보이지만 배경과 표면 Texture도 함께 강조되기 시작합니다. |
### 결과를 볼 때 다음을 확인합니다.

```
clipLimit가 증가하면서
병뚜껑의 세로 홈이 더 잘 보이는가?

병뚜껑과 주변 영역의 경계가
더 명확해지는가?

배경이나 병 표면의 Texture도
같이 강해지고 있지는 않은가?

불필요한 Noise까지
강조되고 있지는 않은가?
```

현재 결과만 육안으로 보면:

```
1.0
→ 대비 개선이 약함

2.0
→ 경계와 홈이 조금 더 잘 보임

3.0
→ 형태가 더 분명해짐

4.0
→ 가장 강하게 보이지만
   불필요한 Texture도 함께 증가할 가능성
```

따라서 여기서도 `clipLimit=4.0`이 가장 선명해 보인다고 바로 선택하지 않습니다.

CLAHE의 적절한 값은 사람이 보기에 가장 선명한 값이 아니라, 이후 Threshold/Otsu에서 병뚜껑 영역이 가장 안정적으로 분리되는 값으로 결정합니다.

---

# 16. Noise와 Filter

![[Pasted image 20260828153017.png]]

카메라 영상의 Noise는 조명이 부족하거나 센서 감도가 높아졌을 때, 또는 촬영환경이 좋지 않을 때 영상에 작은 점·거친 입자처럼 나타나는 불필요한 정보입니다.

이런 Noise가 많으면 뒤에서 Threshold나 Contour를 적용할 때 실제 제품이 아닌 작은 점들까지 객체나 결함으로 잘못 인식될 수 있기 때문에, 검사 전에 적절한 Filter를 사용해 줄여줄 필요가 있습니다.

오늘은 세 가지 대표 Filter를 비교합니다.

```text
Gaussian Blur
Median Blur
Bilateral Filter
```

## 16.1 Gaussian Blur

주변 픽셀의 가중 평균을 이용해 이미지를 부드럽게 만듭니다.

```text
장점
→ 일반적인 Noise 감소

단점
→ 작은 결함과 경계도 함께 흐려질 수 있음
```

---

## 16.2 Median Blur

주변 픽셀의 중간값을 사용합니다.

점 형태의 Salt-and-Pepper Noise에 효과적인 경우가 많습니다.

```text
장점
→ 점 Noise 제거에 유리한 경우가 많음

단점
→ Kernel이 커지면 작은 형태가 사라질 수 있음
```

---

## 16.3 Bilateral Filter

픽셀의 위치와 밝기 차이를 함께 고려하여 Noise를 줄이면서 경계를 어느 정도 보존하려고 합니다.

```text
장점
→ 경계를 비교적 유지하면서 Smoothing

단점
→ 계산량이 상대적으로 큼
```

---

# 17. `13_filter_compare.py` — Filter 비교하기

앞의 실습과 마찬가지로 이번에도 문제 이미지를 따로 준비하지 않습니다.

1일차에서 확보한 기준 ROI:

```
samples/day02/cap_on_reference.jpg
```

를 이용하여 코드에서 일부러 Noise를 추가한 다음:

```
samples/day02/noise.jpg
```

를 만듭니다.

그 후 같은 Noise 이미지에 세 가지 Filter를 적용하여 결과를 비교합니다.

```
cap_on_reference.jpg
        ↓
Noise 추가
        ↓
noise.jpg
        ↓
Gray
        ↓
 ┌──────┼────────┐
 ↓      ↓        ↓
Gaussian Median Bilateral
 ↓      ↓        ↓
      결과 비교
```

이번 실습에서는 센서에서 발생할 수 있는 잔잔한 Noise와 점 형태 Noise를 간단하게 흉내 내어 Filter마다 어떤 차이가 나타나는지 확인합니다.

> 실제 현장의 Noise는 카메라 센서, 조명, Gain, 통신환경 등에 따라 형태가 다를 수 있습니다. 
> 이번 실습의 `noise.jpg`는 Filter의 특징을 비교하기 위해 만든 학습용 문제 이미지입니다.

---

## 17.1 코드 작성하기

새 파일을 만듭니다.

```text
src/13_filter_compare.py
```

```python
from pathlib import Path

import cv2
import numpy as np


# --------------------------------------------------
# 사용할 이미지 경로
# --------------------------------------------------

reference_path = Path(
    "samples/day02/cap_on_reference.jpg"
)

noise_path = Path(
    "samples/day02/noise.jpg"
)

output_dir = Path(
    "outputs/day02/filter"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 기준 ROI 이미지 읽기
# --------------------------------------------------

reference = cv2.imread(
    str(reference_path)
)

if reference is None:
    raise FileNotFoundError(
        f"기준 이미지를 읽지 못했습니다: "
        f"{reference_path}"
    )


# --------------------------------------------------
# 실습용 Noise 이미지 만들기
# --------------------------------------------------

# 같은 결과가 반복되도록 Seed 고정
rng = np.random.default_rng(42)


# 1. 작은 Gaussian Noise 추가
gaussian_noise = rng.normal(
    loc=0,
    scale=18,
    size=reference.shape,
)

noisy = reference.astype(
    np.float32
) + gaussian_noise

noisy = np.clip(
    noisy,
    0,
    255,
).astype(np.uint8)


# 2. 일부 픽셀에 점 형태 Noise 추가
height, width = noisy.shape[:2]

noise_count = int(
    height * width * 0.01
)

ys = rng.integers(
    0,
    height,
    noise_count,
)

xs = rng.integers(
    0,
    width,
    noise_count,
)

half = noise_count // 2

noisy[
    ys[:half],
    xs[:half],
] = 0

noisy[
    ys[half:],
    xs[half:],
] = 255


# 문제 이미지 저장
cv2.imwrite(
    str(noise_path),
    noisy,
)

print(
    "문제 이미지 생성:",
    noise_path,
)


# --------------------------------------------------
# Gray 변환
# --------------------------------------------------

gray = cv2.cvtColor(
    noisy,
    cv2.COLOR_BGR2GRAY,
)


# --------------------------------------------------
# 여러 Filter 적용
# --------------------------------------------------

results = {
    "original_gray.jpg": gray,

    "gaussian_3.jpg": cv2.GaussianBlur(
        gray,
        (3, 3),
        0,
    ),

    "gaussian_7.jpg": cv2.GaussianBlur(
        gray,
        (7, 7),
        0,
    ),

    "median_3.jpg": cv2.medianBlur(
        gray,
        3,
    ),

    "median_7.jpg": cv2.medianBlur(
        gray,
        7,
    ),

    "bilateral.jpg": cv2.bilateralFilter(
        gray,
        9,
        75,
        75,
    ),
}


# --------------------------------------------------
# 결과 저장
# --------------------------------------------------

for filename, result in results.items():

    output_path = (
        output_dir / filename
    )

    cv2.imwrite(
        str(output_path),
        result,
    )

    print(
        "저장:",
        output_path,
    )
```

실행합니다.

```bash
python src/13_filter_compare.py
```

---

## 17.1 의사코드

```text
1. Noise 이미지를 읽는다.

2. Gray로 변환한다.

3. Gaussian 3×3을 적용한다.

4. Gaussian 7×7을 적용한다.

5. Median 3을 적용한다.

6. Median 7을 적용한다.

7. Bilateral Filter를 적용한다.

8. 모든 결과를 각각 저장한다.

9. Noise 감소 정도와
   경계·작은 결함의 손실 정도를 함께 비교한다.
```

## 17.2 결과에서 확인할 것

이번 결과는 각 Filter의 차이가 상당히 명확하게 나온 좋은 실습 결과입니다. 특히 이번에 만든 `noise.jpg`에는 점 형태의 Noise가 포함되어 있기 때문에 Median Blur의 특징이 잘 드러납니다.

이번 결과에서는 같은 Noise 이미지에 `Gaussian`, `Median`, `Bilateral` Filter를 적용하여 Noise가 얼마나 줄어드는지와 병뚜껑의 경계가 얼마나 유지되는지 비교합니다.

| 결과                  | 실제 결과 해석                                                                                |
| ------------------- | --------------------------------------------------------------------------------------- |
| `original_gray.jpg` | 검정·흰색의 작은 점 Noise가 전체적으로 많이 나타납니다. 병뚜껑의 형태는 보이지만 점들이 검사에 방해될 수 있습니다.                    |
| `gaussian_3.jpg`    | 전체적인 거친 Noise는 조금 부드러워졌지만 흰색·검정 점 Noise가 상당수 남아 있습니다. 경계는 비교적 유지됩니다.                    |
| `gaussian_7.jpg`    | `3×3`보다 전체가 더 부드러워졌지만 병뚜껑 경계와 홈도 함께 흐려졌습니다. 점 Noise도 완전히 제거되지 않았습니다.                    |
| `median_3.jpg`      | 점 형태 Noise가 크게 감소하면서 병뚜껑 외곽과 형태도 비교적 잘 유지되었습니다. 현재 결과에서는 가장 균형이 좋아 보입니다.                |
| `median_7.jpg`      | Noise는 많이 감소했지만 병뚜껑의 세로 홈과 경계까지 상당히 흐려졌습니다. Kernel이 너무 커지면 검사에 필요한 특징도 사라질 수 있음을 보여줍니다. |
| `bilateral.jpg`     | 병뚜껑 경계는 비교적 잘 유지되지만 점 형태 Noise가 많이 남았습니다. 이번 Noise 종류에는 Median보다 효과가 떨어지는 모습을 보입니다.     |
결과를 흐름으로 보면 다음과 같습니다.

```text
original_gray
→ 점 Noise가 매우 많음

Gaussian 3×3
→ Noise 조금 감소
→ 경계 비교적 유지
→ 점 Noise는 여전히 많이 남음

Gaussian 7×7
→ 더 부드러워짐
→ 경계와 세부정보도 흐려짐
→ 점 Noise 완전 제거는 어려움

Median 3×3
→ 점 Noise 크게 감소
→ 병뚜껑 형태도 비교적 유지
→ 현재 결과에서 좋은 후보

Median 7×7
→ Noise는 더 감소
→ 병뚜껑 홈과 경계까지 많이 흐려짐

Bilateral
→ 경계는 비교적 유지
→ 점 Noise가 많이 남아 있음
```

### 결과에서 반드시 확인할 질문

```
어떤 Filter에서 점 Noise가 가장 많이 줄었는가?

Noise를 줄인 뒤에도
병뚜껑의 외곽선이 유지되는가?

병뚜껑의 세로 홈과 작은 특징이 남아 있는가?

Kernel을 크게 했을 때
Noise뿐 아니라 검사할 특징도 사라지지 않는가?

Noise가 가장 적은 결과가
정말 검사에 가장 좋은 결과인가?
```

이번 결과에서 특히 중요한 비교는:

```
Median 3×3
        vs
Median 7×7
```

입니다.

`Median 7×7`이 더 매끄럽게 보이지만 검사에 필요한 병뚜껑의 홈과 경계까지 많이 사라졌습니다.

따라서 Noise를 많이 제거하는 것이 목적이 아니라, 검사에 필요한 특징을 유지하면서 불필요한 Noise만 적절히 줄이는 것이 목적입니다.

```
Median 3×3

좋아진 점
→ 흰색·검정 점 Noise가 크게 감소
→ 병뚜껑 외곽 형태 유지
→ 세로 홈도 어느 정도 유지

나빠진 점
→ 원본보다 일부 미세한 Texture는 감소

현재 판단
→ Noise 감소와 특징 보존의 균형이 가장 좋음
```

```
반면 Median 7×7은:

Noise 감소 ↑
하지만
세부정보 보존 ↓
```

이므로 과도한 Filtering의 실패 사례입니다.

즉 이번 실습을 통해 다음 결론을 얻을 수 있습니다.

```
Gaussian이 항상 좋은 것도 아니고

Median이 항상 좋은 것도 아니며

Bilateral이 더 복잡한 기술이라고
항상 더 좋은 것도 아니다.

        ↓

Noise 종류
+
보존해야 할 검사 특징

을 보고 Filter를 선택한다.
```

---

# 18. Blur는 언제 문제가 되나요?

머신비전 영상이 흐려지는 원인은 여러 가지가 있습니다.

```text
카메라 초점이 정확하지 않음
카메라 또는 제품의 진동
제품이 움직이는 동안 촬영
셔터속도가 느림
과도한 Blur Filter 적용
```

이렇게 영상이 흐려지면 제품의 경계와 작은 결함이 주변 픽셀과 섞이면서 약해질 수 있습니다.

예를 들어 작은 흠집을 검사한다고 생각합니다.

```text
작은 흠집
    ↓
Blur 발생
    ↓
주변 픽셀과 섞임
    ↓
흠집 경계가 약해짐
    ↓
불량을 놓칠 가능성
```

따라서 Noise를 줄이기 위해 Blur를 사용할 때도 다음 두 가지를 함께 확인해야 합니다.

```
Noise 감소
    +
검사 특징 보존
```

Noise가 많이 사라졌다고 반드시 좋은 전처리는 아닙니다. 검사해야 할 경계나 작은 결함까지 사라지지 않았는지 함께 확인해야 합니다.

---

# 19. Sharpening은 언제 사용하나요?

Sharpening은 조금 흐려진 영상에서 이미 존재하는 경계를 더 강조하고 싶을 때 사용할 수 있습니다.

예를 들어:

```text
병뚜껑 세로 홈이 약간 흐림
        ↓
경계가 완전히 사라진 것은 아님
        ↓
Sharpening 적용
        ↓
홈과 외곽 경계를 더 강조
```

따라서 다음과 같은 경우에 후보가 될 수 있습니다.

```
초점이 약간 흐림

경계가 조금 약함

전처리 과정에서
형태가 약간 부드러워짐
```

하지만 Sharpening은 없어진 정보를 복구하는 기술은 아닙니다.

```
약한 Blur
→ Sharpening으로 경계 강조 가능

심한 Blur
→ 원래 정보가 많이 손실됨
→ Sharpening만으로 복구 어려움
```

또한 원본에 Noise가 많으면:

```
경계 강조
    +
Noise도 함께 강조
```

될 수 있습니다.

따라서 Sharpening 역시 무조건 사용하는 것이 아니라 검사에 필요한 경계가 실제로 더 안정적으로 보이는지 확인한 뒤 사용 여부를 결정합니다.

---

# 20. `14_sharpen.py` — Blur 영상을 만들고 경계 강조하기

이번 실습에서도 `blur.jpg`를 별도로 준비하지 않습니다.

1일차에서 만든 기준 ROI:

```
samples/day02/cap_on_reference.jpg
```

를 이용하여 코드에서 일부러 약한 Blur를 만든 뒤 Sharpening을 적용합니다.

전체 흐름은 다음과 같습니다.

```text
cap_on_reference.jpg
        ↓
Gaussian Blur
        ↓
blur.jpg
        ↓
Sharpening
        ↓
sharpened.jpg
        ↓
세 이미지 비교
```

이번 실습의 목적은 Sharpening을 적용하면 원본으로 완전히 복원되는지를 확인하는 것이 아니라, 흐려진 경계를 어느 정도 다시 강조할 수 있는지와 그 한계를 확인하는 것입니다.

---

## 20.1 코드 작성하기

새 파일을 만듭니다.

```
src/14_sharpen.py
```

```python
from pathlib import Path

import cv2
import numpy as np


# --------------------------------------------------
# 사용할 이미지 경로
# --------------------------------------------------

reference_path = Path(
    "samples/day02/cap_on_reference.jpg"
)

blur_path = Path(
    "samples/day02/blur.jpg"
)

output_dir = Path(
    "outputs/day02/sharpen"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 기준 ROI 이미지 읽기
# --------------------------------------------------

reference = cv2.imread(
    str(reference_path)
)

if reference is None:
    raise FileNotFoundError(
        f"기준 이미지를 읽지 못했습니다: "
        f"{reference_path}"
    )


# --------------------------------------------------
# 실습용 Blur 이미지 만들기
# --------------------------------------------------

blurred = cv2.GaussianBlur(
    reference,
    (7, 7),
    0,
)

cv2.imwrite(
    str(blur_path),
    blurred,
)

print(
    "문제 이미지 생성:",
    blur_path,
)


# --------------------------------------------------
# Sharpening Kernel
# --------------------------------------------------

kernel = np.array(
    [
        [0, -1, 0],
        [-1, 5, -1],
        [0, -1, 0],
    ],
    dtype=np.float32,
)


# --------------------------------------------------
# Sharpening 적용
# --------------------------------------------------

sharpened = cv2.filter2D(
    blurred,
    -1,
    kernel,
)


# --------------------------------------------------
# 비교 결과 저장
# --------------------------------------------------

cv2.imwrite(
    str(
        output_dir / "reference.jpg"
    ),
    reference,
)

cv2.imwrite(
    str(
        output_dir / "blurred.jpg"
    ),
    blurred,
)

cv2.imwrite(
    str(
        output_dir / "sharpened.jpg"
    ),
    sharpened,
)

print(
    "Sharpening 결과 저장 완료"
)
```

실행합니다.

```bash
python src/14_sharpen.py
```

## 20.1 의사코드

```text
1. cap_on_reference.jpg를 읽는다.

2. 기준 이미지가 정상적으로
   읽혔는지 확인한다.

3. Gaussian Blur를 적용하여
   일부러 흐린 이미지를 만든다.

4. 흐린 이미지를
   blur.jpg로 저장한다.

5. 경계를 강조할
   Sharpening Kernel을 만든다.

6. blur.jpg에
   Sharpening을 적용한다.

7. 기준 이미지,
   Blur 이미지,
   Sharpening 이미지를
   각각 저장한다.

8. 세 이미지를 비교한다.

9. 병뚜껑의 경계와 세로 홈이
   다시 강조되었는지 확인한다.

10. 원본 수준으로 완전히
    복구되었는지는 따로 확인한다.

11. Noise나 불필요한 경계까지
    강해지지 않았는지 확인한다.
```

![[{EE1D9487-972F-43A4-8304-40E096884B14}.png]]

간단하게 보면:

```
기준 ROI
   ↓
Blur 생성
   ↓
경계 약화 확인
   ↓
Sharpening
   ↓
경계 다시 강조
   ↓
효과와 한계 비교
```

## 20.3 결과에서 확인할 것

세 이미지를 나란히 비교합니다.

![[{EDA4C323-40A6-419B-98A4-8F3DD8CF870E}.png]]

| 이미지             | 확인할 내용                   |
| --------------- | ------------------------ |
| `reference.jpg` | 원래 병뚜껑의 홈과 경계는 어떻게 보이는가  |
| `blurred.jpg`   | Blur 후 홈과 외곽선이 얼마나 흐려졌는가 |
| `sharpened.jpg` | 흐려진 경계가 다시 강조되었는가        |
핵심 질문은 세 가지면 충분합니다.

```
병뚜껑의 세로 홈과 경계가
Blur보다 잘 보이게 되었는가?

Sharpening 후에도
원본과 동일한 정보가 복구된 것은 아닌가?

Noise나 불필요한 경계까지
함께 강조되지는 않았는가?
```

### 핵심 판단

```
Blur
→ 정보를 약하게 만듦

Sharpening
→ 남아 있는 경계를 강조

하지만

이미 사라진 정보
→ 완전히 복구할 수 없음
```

따라서 심한 초점 실패나 Motion Blur는 Sharpening으로 해결하려 하기보다 카메라 초점·셔터속도·조명·제품 움직임 등의 촬영조건을 먼저 개선하는 것이 맞습니다.

이렇게 구성하면 `18`은 “Blur가 왜 위험한가”, `19`는 “Sharpening은 언제 사용하는가”, `20`은 “직접 Blur를 만들고 Sharpening의 효과와 한계를 확인한다”로 역할이 명확하게 나뉩니다.

---

앞에서는 영상에 문제가 있을 때 여러 전처리를 적용해 보았습니다.

```
어두움
→ Brightness / Gamma

저대비
→ CLAHE

Noise
→ Filter

Blur
→ Sharpening
```

그런데 여기까지는 사람이 결과를 보고:

```
조금 더 밝아졌다.

경계가 더 잘 보인다.

Noise가 줄었다.
```

정도로 판단한 것입니다.

머신비전에서는 이것만으로 **좋은 전처리라고 결정할 수 없습니다.**

결국 컴퓨터가 다음 단계에서 제품과 배경을 분리할 수 있어야 합니다.

```
전처리
    ↓
제품과 배경 분리
    ↓
객체 영역 확보
    ↓
Contour
    ↓
면적 · 개수 · 위치 측정
```

그래서 2일차에서는 Threshold와 Otsu를 **전처리 결과를 확인하기 위한 간단한 검사 도구**로 먼저 사용합니다.

> “사람 눈에 보기 좋아졌는가?”가 아니라 “컴퓨터가 객체와 배경을 더 잘 나눌 수 있는가?”를 확인하는 것이 목적입니다.

다만 오늘 Threshold를 깊게 배우거나 `OK / NG`까지 만들지는 않습니다.

```
2일차

전처리
   ↓
Threshold / Otsu
   ↓
분리 가능성 확인
```

다음 3일차에서:

```
Binary Image
    ↓
Morphology
    ↓
Contour
    ↓
Area / Count / Position
    ↓
Rule
    ↓
OK / NG
```

로 이어집니다. 교과 문서에서도 Threshold/Otsu 이후 Morphology와 Contour로 연결하는 구조로 되어 있습니다.

---

# 21. Threshold는 무엇을 하는 기술인가요?

현재 병뚜껑 이미지는 Gray로 바꾸어도 픽셀값이 모두 똑같지 않습니다.

예를 들어:

```
병뚜껑
→ 어두운 픽셀

배경
→ 밝은 픽셀
```

이라고 생각할 수 있습니다.

Threshold는 하나의 기준값을 정해서 픽셀을 두 그룹으로 나누는 방법입니다.

![[Pasted image 20260828171152.png]]

예를 들어:

```
Threshold = 120
```

으로 정하면 이번 병뚜껑 실습에서는:

```
0 ~ 119
→ 어두운 영역
→ 병뚜껑 후보

120 ~ 255
→ 밝은 영역
→ 배경 후보
```

처럼 나눌 수 있습니다.

이번 병뚜껑은 원래 어두운 색이고 배경은 밝은 색입니다.

그래서 Threshold 결과를 보기 쉽게 반대로 바꿉니다. 그것이 **Inverse Binary** 입니다.

```
어두운 병뚜껑
→ 흰색으로 표시

밝은 배경
→ 검정색으로 표시
```

즉, 병뚜껑 부분만 흰색 객체처럼 보이게 만들어서, 다음 단계에서 병뚜껑 영역을 쉽게 찾기 위한 방법입니다.

---

## 21.1 Threshold 값은 왜 여러 개 비교하나요?

Threshold는 Gray 이미지의 밝기값을 기준으로 객체와 배경을 나누는 기준선입니다.

문제는 이미지마다 밝기가 다르기 때문에 처음부터 어떤 값이 가장 적절한지 알 수 없다는 것입니다.

예를 들어 이번 실습에서는 다음 세 값을 비교합니다.

```
Threshold = 80
Threshold = 120
Threshold = 160
```

같은 이미지라도 기준값이 달라지면 흰색으로 선택되는 영역이 달라집니다.

```
같은 Gray 이미지
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
 80   120   160
 ↓     ↓     ↓
병뚜껑이 선택되는 범위가 달라짐
```

따라서 이번 실습의 목적은 80, 120, 160 중 정답을 외우는 것이 아니라, Threshold 값에 따라 객체와 배경의 분리 결과가 어떻게 달라지는지 직접 확인하는 것입니다.

이후에는 사람이 값을 하나씩 정하지 않아도 되도록 Otsu가 이미지의 밝기 분포를 보고 기준값을 자동으로 선택하는 방법도 비교합니다.

---

## 21.2 Otsu는 무엇이 다른가요?

![[Pasted image 20260828172449.png]]

고정 Threshold에서는 사람이 값을 정합니다.

```
사람
→ 80?
→ 120?
→ 160?
```

반면 Otsu는 Gray 이미지의 밝기 분포를 분석하여 객체와 배경을 나누기 위한 Threshold 값을 자동으로 계산합니다.

```
고정 Threshold 
→ 사람이 값 선택 

Otsu 
→ 이미지에서 값 자동 계산
```

하지만 Otsu도 항상 정답은 아닙니다.

```
조명이 고르지 않음

배경이 복잡함

반사가 심함

제품과 배경의 밝기가 너무 비슷함
```

따라서 Otsu도 자동 정답이 아니라 객체 분리를 위한 하나의 방법으로 이해합니다.

---

# 22. `15_threshold_otsu.py` — 저대비 영상에서 객체 분리 비교하기

![[Pasted image 20260828173424.png]]

앞의 `12_clahe.py`에서는 기준 ROI 이미지로부터 일부러 밝기 차이를 줄인 저대비 이미지를 만들었습니다.

```
samples/day02/low_contrast.jpg
```

이번 실습에서는 이 이미지를 Gray로 변환한 뒤, Threshold 기준값에 따라 병뚜껑과 배경이 어떻게 다르게 분리되는지 확인합니다.

```
low_contrast.jpg
        ↓
      Gray
        ↓
 ┌───────────────┐
 │               │
고정 Threshold    Otsu
80 / 120 / 160    자동 선택
 │               │
 └───────┬───────┘
         ↓
   Binary 결과 비교
```

고정 Threshold에서는 사람이 `80`, `120`, `160`처럼 기준값을 직접 정합니다.

반면 Otsu는 이미지의 밝기 분포를 분석하여 객체와 배경을 나누기 위한 기준값을 자동으로 계산합니다.

이번 단계에서 확인할 핵심은 하나입니다.

같은 이미지라도 Threshold 기준값에 따라 객체와 배경의 분리 결과가 크게 달라질 수 있다.

따라서 여기서는 어떤 값이 정답인지 결정하는 것이 아니라,

```
Threshold 값 변경
       ↓
흰색으로 선택되는 영역 변화
       ↓
병뚜껑과 배경의 분리 결과 변화
```

를 직접 확인합니다.

아직 이 단계에서는 CLAHE가 좋은지, Gaussian이 좋은지 비교하지 않습니다.

다음 `16_preprocess_compare.py`에서 동일한 Otsu 방식을 적용하여:

```
원본 + Otsu

Gaussian + Otsu

CLAHE + Otsu
```

즉 두 실습의 역할을 나누면 다음과 같습니다.

```
15_threshold_otsu.py
→ Threshold 기준값에 따라
  분리 결과가 어떻게 달라지는지 이해

16_preprocess_compare.py
→ 앞에서 배운 전처리가
  실제 객체 분리에 도움이 되었는지 확인
```

이렇게 이해하면 됩니다.

15번은 Threshold 자체를 이해하는 실습이고, 16번은 전처리 효과를 Threshold 결과로 검증하는 실습입니다.

---

## 22.1 코드 작성하기

새 파일을 만듭니다.

```
src/15_threshold_otsu.py
```

```python
from pathlib import Path

import cv2


# --------------------------------------------------
# 사용할 이미지 경로
# --------------------------------------------------

reference_path = Path(
    "samples/day02/cap_on_reference.jpg"
)

low_contrast_path = Path(
    "samples/day02/low_contrast.jpg"
)

output_dir = Path(
    "outputs/day02/threshold"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# low_contrast.jpg 확인
# --------------------------------------------------

image = cv2.imread(
    str(low_contrast_path)
)


# low_contrast.jpg가 없다면
# 기준 ROI에서 다시 생성
if image is None:

    print(
        "low_contrast.jpg가 없어 "
        "기준 이미지에서 생성합니다."
    )

    reference = cv2.imread(
        str(reference_path)
    )

    if reference is None:
        raise FileNotFoundError(
            f"기준 이미지를 읽지 못했습니다: "
            f"{reference_path}"
        )

    image = cv2.convertScaleAbs(
        reference,
        alpha=0.35,
        beta=80,
    )

    cv2.imwrite(
        str(low_contrast_path),
        image,
    )

    print(
        "문제 이미지 생성:",
        low_contrast_path,
    )


# --------------------------------------------------
# Gray 변환
# --------------------------------------------------

gray = cv2.cvtColor(
    image,
    cv2.COLOR_BGR2GRAY,
)

cv2.imwrite(
    str(output_dir / "gray.jpg"),
    gray,
)


# --------------------------------------------------
# 고정 Threshold 비교
# --------------------------------------------------

threshold_values = [
    80,
    120,
    160,
]


for value in threshold_values:

    _, binary = cv2.threshold(
        gray,
        value,
        255,
        cv2.THRESH_BINARY_INV,
    )

    output_path = (
        output_dir
        / f"threshold_{value}.jpg"
    )

    cv2.imwrite(
        str(output_path),
        binary,
    )

    print(
        "저장:",
        output_path,
    )


# --------------------------------------------------
# Otsu Threshold
# --------------------------------------------------

otsu_value, otsu = cv2.threshold(
    gray,
    0,
    255,
    cv2.THRESH_BINARY_INV
    + cv2.THRESH_OTSU,
)

otsu_path = (
    output_dir / "otsu.jpg"
)

cv2.imwrite(
    str(otsu_path),
    otsu,
)

print(
    "Otsu Threshold:",
    otsu_value,
)

print(
    "저장:",
    otsu_path,
)
```

실행합니다.

```bash
python src/15_threshold_otsu.py
```
---

## 22.2 의사코드

```text
1. low_contrast.jpg를 읽어본다.

2. 파일이 존재하지 않으면
   cap_on_reference.jpg를 읽는다.

3. 기준 이미지의 Contrast를 낮춰
   low_contrast.jpg를 만든다.

4. 저대비 이미지를 Gray로 변환한다.

5. Gray 이미지를 저장한다.

6. Threshold = 80을 적용한다.

7. 병뚜껑을 흰색 객체로 보기 위해
   Binary Inverse를 사용한다.

8. 결과를 저장한다.

9. 같은 방식으로
   Threshold = 120을 적용하고 저장한다.

10. Threshold = 160을 적용하고 저장한다.

11. 같은 Gray 이미지에 Otsu를 적용한다.

12. Otsu가 자동으로 선택한
    Threshold 값을 출력한다.

13. Otsu Binary 결과를 저장한다.

14. 80 / 120 / 160 / Otsu 결과를 비교한다.

15. 어떤 결과에서 병뚜껑과 배경이
    가장 안정적으로 분리되는지 확인한다.
```

![[Pasted image 20260828174433.png]]

## 22.3 Threshold / Otsu 결과 확인

같은 `gray.jpg`에 서로 다른 Threshold 값을 적용하자 결과가 크게 달라졌습니다.

| 결과                  | 실제 해석                                                     |
| ------------------- | --------------------------------------------------------- |
| `gray.jpg`          | Threshold 적용 전의 저대비 Gray 이미지입니다. 병뚜껑과 배경의 밝기 차이가 크지 않습니다. |
| `threshold_80.jpg`  | 거의 전체가 검정색이 되어 병뚜껑을 제대로 찾지 못했습니다. **기준값이 너무 낮습니다.**       |
| `threshold_120.jpg` | 병뚜껑 대부분이 흰색으로 나타나고 배경은 검정색으로 분리되었습니다. **비교적 잘 분리됩니다.**    |
| `threshold_160.jpg` | 거의 전체가 흰색이 되어 병뚜껑과 배경을 구분할 수 없습니다. **기준값이 너무 높습니다.**      |
| `otsu.jpg`          | Otsu가 밝기 분포를 보고 자동으로 기준값을 선택했습니다. 병뚜껑 형태가 비교적 잘 분리되었습니다.  |
결과를 간단히 보면:

```
Threshold 80
→ 너무 낮음
→ 객체를 거의 찾지 못함

Threshold 120
→ 병뚜껑 형태가 비교적 잘 분리됨

Threshold 160
→ 너무 높음
→ 배경까지 객체로 선택됨

Otsu
→ 자동으로 기준값 선택
→ 병뚜껑 형태가 비교적 잘 분리됨
```

### 이번 실습에서 확인한 핵심

같은 이미지라도 Threshold 기준값에 따라 Binary 결과가 완전히 달라질 수 있습니다.

현재 결과에서는 `120`과 `Otsu`가 비교적 좋은 후보로 보이지만, 아직 최종 선택은 아닙니다.

> 다음 실습에서는 전처리를 적용한 뒤 Otsu 결과가 더 안정적으로 바뀌는지 비교합니다.

즉 이번 실습의 핵심은 “120을 외우는 것”이 아니라, 너무 낮거나 높은 Threshold 값이 왜 객체 분리를 실패하게 만드는지 이해하는 것입니다.


---

# 23. 이제 전처리가 실제 분리에 도움이 되었는지 확인합니다

앞의 실습에서는 같은 저대비 이미지에서 Threshold 값에 따라 결과가 어떻게 달라지는지 확인했습니다.

이제 질문을 바꿉니다.

> 앞에서 배운 전처리를 적용하면 객체와 배경의 분리가 실제로 더 좋아질까요?

이를 확인하기 위해 같은 이미지에 서로 다른 전처리를 적용한 뒤 모두 Otsu로 분리합니다.

![[Pasted image 20260828175621.png]]

```
저대비 이미지
     ↓
 ┌────────────┬────────────┐
 ↓            ↓            ↓
전처리 없음   Gaussian     CLAHE
 ↓            ↓            ↓
Otsu          Otsu         Otsu
 ↓            ↓            ↓
Binary 결과 비교
```

여기서부터는 Threshold 자체를 배우는 것이 아니라 전처리의 효과를 검증하는 단계입니다.

---

# 24. `16_preprocess_compare.py` — 전처리가 객체 분리에 도움이 되었는지 비교하기

앞의 실습에서는 같은 저대비 이미지에 여러 Threshold 값을 적용하여 기준값에 따라 객체 분리 결과가 달라진다는 것을 확인했습니다.

이번에는 질문이 달라집니다.

> 전처리를 적용하면 병뚜껑과 배경의 분리가 실제로 더 좋아지는가?

즉 사람이 보기에 더 선명한 이미지를 고르는 것이 아니라, 동일한 Otsu를 적용했을 때 어떤 전처리가 병뚜껑 영역을 더 안정적으로 분리하는지 확인하는 실습입니다.

## 24.1 코드 작성하기

새 파일:

```text
src/16_preprocess_compare.py
```

```python
from pathlib import Path

import cv2


# --------------------------------------------------
# 사용할 이미지 경로
# --------------------------------------------------

reference_path = Path(
    "samples/day02/cap_on_reference.jpg"
)

low_contrast_path = Path(
    "samples/day02/low_contrast.jpg"
)

output_dir = Path(
    "outputs/day02/compare"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# low_contrast.jpg 확인
# --------------------------------------------------

image = cv2.imread(
    str(low_contrast_path)
)


# low_contrast.jpg가 없다면
# 기준 ROI에서 다시 생성
if image is None:

    print(
        "low_contrast.jpg가 없어 "
        "기준 이미지에서 생성합니다."
    )

    reference = cv2.imread(
        str(reference_path)
    )

    if reference is None:
        raise FileNotFoundError(
            f"기준 이미지를 읽지 못했습니다: "
            f"{reference_path}"
        )

    image = cv2.convertScaleAbs(
        reference,
        alpha=0.35,
        beta=80,
    )

    cv2.imwrite(
        str(low_contrast_path),
        image,
    )

    print(
        "문제 이미지 생성:",
        low_contrast_path,
    )


# --------------------------------------------------
# Gray 변환
# --------------------------------------------------

gray = cv2.cvtColor(
    image,
    cv2.COLOR_BGR2GRAY,
)


# --------------------------------------------------
# 1. 전처리 없이 Otsu
# --------------------------------------------------

original_value, original_otsu = cv2.threshold(
    gray,
    0,
    255,
    cv2.THRESH_BINARY_INV
    + cv2.THRESH_OTSU,
)


# --------------------------------------------------
# 2. Gaussian 후 Otsu
# --------------------------------------------------

gaussian = cv2.GaussianBlur(
    gray,
    (5, 5),
    0,
)

gaussian_value, gaussian_otsu = cv2.threshold(
    gaussian,
    0,
    255,
    cv2.THRESH_BINARY_INV
    + cv2.THRESH_OTSU,
)


# --------------------------------------------------
# 3. CLAHE 후 Otsu
# --------------------------------------------------

clahe = cv2.createCLAHE(
    clipLimit=2.0,
    tileGridSize=(8, 8),
)

clahe_image = clahe.apply(
    gray
)

clahe_value, clahe_otsu = cv2.threshold(
    clahe_image,
    0,
    255,
    cv2.THRESH_BINARY_INV
    + cv2.THRESH_OTSU,
)


# --------------------------------------------------
# Otsu가 선택한 값 확인
# --------------------------------------------------

print(
    "Original Otsu:",
    original_value,
)

print(
    "Gaussian Otsu:",
    gaussian_value,
)

print(
    "CLAHE Otsu:",
    clahe_value,
)


# --------------------------------------------------
# 비교용 이미지 만들기
# --------------------------------------------------

view_size = (
    400,
    300,
)


def make_view(
    source,
    title,
    is_binary=False,
):

    if is_binary:
        interpolation = cv2.INTER_NEAREST
    else:
        interpolation = cv2.INTER_AREA

    resized = cv2.resize(
        source,
        view_size,
        interpolation=interpolation,
    )

    if len(resized.shape) == 2:
        canvas = cv2.cvtColor(
            resized,
            cv2.COLOR_GRAY2BGR,
        )
    else:
        canvas = resized.copy()

    cv2.rectangle(
        canvas,
        (0, 0),
        (view_size[0], 40),
        (0, 0, 0),
        -1,
    )

    cv2.putText(
        canvas,
        title,
        (10, 27),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.6,
        (255, 255, 255),
        2,
    )

    return canvas


# --------------------------------------------------
# 2 × 2 비교 화면
# --------------------------------------------------

top = cv2.hconcat(
    [
        make_view(
            gray,
            "Low Contrast Gray",
        ),
        make_view(
            original_otsu,
            "Original + Otsu",
            is_binary=True,
        ),
    ]
)

bottom = cv2.hconcat(
    [
        make_view(
            gaussian_otsu,
            "Gaussian + Otsu",
            is_binary=True,
        ),
        make_view(
            clahe_otsu,
            "CLAHE + Otsu",
            is_binary=True,
        ),
    ]
)

compare = cv2.vconcat(
    [
        top,
        bottom,
    ]
)


# --------------------------------------------------
# 결과 저장
# --------------------------------------------------

output_path = (
    output_dir
    / "preprocess_compare.png"
)

cv2.imwrite(
    str(output_path),
    compare,
)

print(
    "저장:",
    output_path,
)
```

---

### [[2일차 16코드 Jupyter로 코드 리뷰 방법]] 바로가기

jupyter 실행

```bash
jupyter notebook --no-browser --port=8888
```

---

실행합니다.

```bash
python src/16_preprocess_compare.py
```

결과:

```text
outputs/day02/compare/
└── preprocess_compare.png
```

데이터 연결 순서입니다.

```
cap_on_reference.jpg
        ↓
12_clahe.py
        ↓
low_contrast.jpg
        ↓
16_preprocess_compare.py
```

## 24.2 의사코드

```text
[INPUT — 입력]

1. low_contrast.jpg를 불러옵니다.
2. 이미지가 없다면 기준 ROI 이미지로 생성합니다.

           ↓

[PROCESS — 영상처리]

3. 컬러 이미지를 Gray로 변환합니다.

4. 세 가지 방법을 비교합니다.

   A. Gray
      → Otsu

   B. Gray
      → Gaussian Blur
      → Otsu

   C. Gray
      → CLAHE
      → Otsu

           ↓

[COMPARE — 결과 비교]

5. 각 Otsu의 임계값을 확인합니다.

6. 세 가지 Binary 결과를 비교합니다.

7. 어느 방법이 객체와 배경을
   잘 분리하는지 관찰합니다.

           ↓

[OUTPUT — 결과 저장]

8. 네 개의 이미지를 2×2로 합칩니다.

9. preprocess_compare.png로 저장합니다.

           ↓

[3일차 수업 연결]

10. Binary 이미지의 객체 영역을
    Morphology와 Contour로 측정하는
    다음 수업의 목적을 이해합니다.
```

---

## 24.3 결과에서 확인할 것

![[{9BD5E8BE-DF06-40DB-93F3-87B0313024DD}.png]]

세 결과를 비교하면서 **병뚜껑과 배경이 어떻게 분리되었는지** 확인합니다.

- **Original + Otsu**  
    병뚜껑 전체 형태는 비교적 잘 나타나지만 내부와 아래쪽에 검정 영역이 일부 남아 있습니다.
- **Gaussian + Otsu**  
    병뚜껑 형태가 조금 더 부드러워졌지만 작은 경계와 세부 형태도 함께 단순해졌습니다.
- **CLAHE + Otsu**  
    병뚜껑 형태는 잘 나타나지만 내부와 아래쪽에 불필요한 영역도 조금 더 보입니다.

결과를 볼 때는 다음을 확인합니다.

```
병뚜껑 전체 형태가 잘 남아 있는가?

병뚜껑 내부가 불필요하게 끊기지 않는가?

배경이 흰색 객체로 함께 선택되지 않는가?

전처리 때문에 필요한 경계가 사라지지 않았는가?
```

여기서는 어떤 방법이 가장 좋은지 바로 결정하지 않고, 세 결과의 차이를 먼저 관찰합니다.

---

# 25. 전처리가 실제 검사에 도움이 되었는지 판단합니다

이제 앞에서 관찰한 결과를 바탕으로 판단합니다.

이번 결과에서는:

```
Original + Otsu
→ 전처리 없이도 비교적 잘 분리됨

Gaussian + Otsu
→ 큰 개선은 없고 일부 경계가 단순해짐

CLAHE + Otsu
→ 일부 영역은 강조되지만
   불필요한 영역도 함께 나타남
```

따라서 이번 이미지에서는 **전처리를 추가했다고 반드시 객체 분리가 좋아진 것은 아닙니다.**

```
전처리 적용
     ↓
무조건 좋아지는 것은 아님

Otsu 결과 비교
     ↓
실제로 도움이 되는 경우에만 사용
```

현재 결과에서는 `Original + Otsu`도 충분한 후보입니다.

> 전처리를 사용하지 않는 것이 더 안정적이라면, 추가 전처리를 하지 않는 것도 올바른 판단입니다.

---

# 26. 소프트웨어로 해결하기 어려운 문제도 확인합니다

이번에는 직접 촬영한 다음 이미지를 확인합니다.

```
samples/day02/glare.jpg
```

이 이미지는 전처리 기술을 하나 더 연습하기 위한 것이 아니라, **촬영환경 자체가 나쁘면 소프트웨어만으로 해결하기 어려울 수 있다는 것을 확인하기 위한 사례**입니다.

사진을 보고 다음 두 가지만 확인합니다.

```
강한 빛 때문에
병뚜껑의 일부 정보가 사라졌는가?

전처리를 해도
사라진 경계나 세부정보가 돌아오는가?
```

빛이 너무 강해 일부 영역이 완전히 하얗게 날아갔다면 이미 촬영 단계에서 정보가 손실된 것입니다.

```
강한 반사
    ↓
픽셀 포화
    ↓
원래 정보 손실
    ↓
전처리만으로 복구하기 어려움
```

이 경우에는 전처리를 계속 추가하기보다:

```
조명 각도 변경
조명 밝기 조절
카메라 위치 조정
```

처럼 촬영환경을 먼저 개선하는 것이 더 적절합니다.

> 머신비전에서는 모든 문제를 코드로 해결하는 것이 아니라, 소프트웨어 문제인지 촬영환경 문제인지 구분하는 것도 중요한 판단입니다.

---

# PART 5. 이제 스스로 문제를 해결합니다

지금까지는 강사와 함께 문제 이미지를 보면서 다음 과정을 연습했습니다.

```
문제 확인
    ↓
원인 판단
    ↓
전처리 후보 선택
    ↓
파라미터 비교
    ↓
Threshold / Otsu 결과 확인
    ↓
사용 여부 판단
```

이제부터는 강사가 어떤 기술을 사용하라고 먼저 알려주지 않습니다.

새로운 이미지를 보고 같은 순서로 직접 판단해 봅니다.

```
[1단계] 28~30
새 문제를 보고 스스로 해결

        ↓

[2단계] 31~33
결과를 확인하고 강사 예시와 비교

        ↓

[3단계] 34~37
내가 한 선택과 실패를 기록

        ↓

[4단계] 38~39
오늘 배운 판단 방법 정리
```

---

# 28 Mini Challenge — 이제 스스로 해결합니다

지금까지는 강사와 함께 각 전처리 기술을 하나씩 실습했습니다.

이제부터는 새로운 `challenge.jpg`를 보고 **어떤 문제가 있는지 학생이 직접 판단하고 필요한 전처리를 선택**합니다.

정해진 정답은 없습니다.

```
문제 이미지 확인
        ↓
문제 원인 판단
        ↓
전처리 후보 선택
        ↓
Parameter 비교
        ↓
Otsu 결과 확인
        ↓
사용 여부 결정
```

---

#  Mini Challenge 이미지를 확인합니다

![[Pasted image 20260828192701.png]]

미니 챌린지는 위의 이미지중에서 challenge.jpg 이미지를 사용하여 작업합니다.

참고로 세 이미지의 역할은 다음과 같습니다.

```
cap_on_reference.jpg
→ 정상 상태 참고

challenge.jpg
→ Mini Challenge 실습

glare.jpg
→ 촬영환경 문제 확인
```

이번 Challenge에서는 다음 이미지만 사용합니다.

```
samples/day02/challenge.jpg
```

이 이미지는 정상 조명보다 어둡게 촬영한 새로운 조건의 이미지입니다.

먼저 코드를 작성하지 말고 `challenge.jpg`를 직접 봅니다.

현재 이미지에서는 다음과 같은 문제를 생각해 볼 수 있습니다.

```
전체적으로 어두운가?

병뚜껑과 주변의 밝기 차이가 충분한가?

병뚜껑의 홈과 경계가 잘 보이는가?

Noise나 Blur 문제도 있는가?

전처리 없이도 검사할 수 있을 정도인가?
```

한 문장으로 문제를 정의합니다.

예:

```
전체적으로 밝기가 낮아
병뚜껑의 경계와 세부 형태가
정상 조명보다 구분하기 어렵다.
```

> 문제를 찾는 것이 먼저이며, 전처리 기술을 먼저 고르지 않습니다.

---

### 직접 촬영한 원본이미지가 너무 크므로 ROI로 만듦니다.

제공해준 이미지를 프로젝트 폴더안에 복사해서 넣습니다.

```bash
explorer.exe .
```

제공해준 challenge.jpg를 samples/day02/ 안에 넣습니다. 아래 디렉토리를 참조합니다.

다음으로는 ROI 전용 폴더를 만듦니다.

```bash
cd ~/ai_vision/subject9_machinevision
mkdir -p roi_temp
```

ROI를 위한 새로운 디렉토리 생성 구조는 다음과 같습니다.

```
subject9_machinevision/
│
├── samples/
│   └── day02/
│       └── challenge.jpg
│
└── roi_temp/
    └── crop_challenge.py
```

```python
from pathlib import Path
import shutil

import cv2


image_path = Path(
    "samples/day02/challenge.jpg"
)

temp_dir = Path(
    "roi_temp"
)

backup_path = (
    temp_dir / "challenge_original.jpg"
)

temp_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 이미지 읽기
# --------------------------------------------------

image = cv2.imread(
    str(image_path)
)

if image is None:
    raise FileNotFoundError(
        f"이미지를 읽지 못했습니다: {image_path}"
    )


# --------------------------------------------------
# 원본 백업
# --------------------------------------------------

if not backup_path.exists():

    shutil.copy2(
        image_path,
        backup_path,
    )

    print(
        "원본 백업:",
        backup_path,
    )


# --------------------------------------------------
# 화면 크기 조절
# --------------------------------------------------

height, width = image.shape[:2]

max_width = 900
max_height = 700

scale = min(
    1.0,
    max_width / width,
    max_height / height,
)

display_width = int(
    width * scale
)

display_height = int(
    height * scale
)

preview = cv2.resize(
    image,
    (
        display_width,
        display_height,
    ),
)


# --------------------------------------------------
# ROI 선택 변수
# --------------------------------------------------

start_point = None
end_point = None

dragging = False
roi_selected = False


# --------------------------------------------------
# 마우스 이벤트
# --------------------------------------------------

def mouse_callback(
    event,
    x,
    y,
    flags,
    param,
):

    global start_point
    global end_point
    global dragging
    global roi_selected

    # 마우스 왼쪽 버튼 누름
    if event == cv2.EVENT_LBUTTONDOWN:

        start_point = (
            x,
            y,
        )

        end_point = (
            x,
            y,
        )

        dragging = True
        roi_selected = False


    # 마우스를 누른 상태로 이동
    elif (
        event == cv2.EVENT_MOUSEMOVE
        and dragging
    ):

        end_point = (
            x,
            y,
        )


    # 마우스 버튼 놓음
    elif event == cv2.EVENT_LBUTTONUP:

        end_point = (
            x,
            y,
        )

        dragging = False
        roi_selected = True

        print(
            "ROI 선택:",
            start_point,
            "→",
            end_point,
        )


# --------------------------------------------------
# Window
# --------------------------------------------------

window_name = (
    "ROI - Drag and press ENTER"
)

cv2.namedWindow(
    window_name,
    cv2.WINDOW_AUTOSIZE,
)

cv2.setMouseCallback(
    window_name,
    mouse_callback,
)


print()
print("ROI 선택 방법")
print("-----------------------------")
print("1. 병뚜껑 왼쪽 위에서")
print("   마우스 왼쪽 버튼을 누릅니다.")
print()
print("2. 버튼을 누른 상태로")
print("   오른쪽 아래까지 드래그합니다.")
print()
print("3. 마우스를 놓습니다.")
print()
print("4. 초록색 사각형을 확인합니다.")
print()
print("5. ENTER 또는 SPACE : 저장")
print("6. C : 다시 선택")
print("7. ESC : 취소")
print("-----------------------------")
print()


# --------------------------------------------------
# ROI 선택 화면
# --------------------------------------------------

while True:

    canvas = preview.copy()

    if (
        start_point is not None
        and end_point is not None
    ):

        cv2.rectangle(
            canvas,
            start_point,
            end_point,
            (0, 255, 0),
            2,
        )

    cv2.imshow(
        window_name,
        canvas,
    )

    key = (
        cv2.waitKey(20)
        & 0xFF
    )


    # ESC
    if key == 27:

        cv2.destroyAllWindows()

        print(
            "ROI 선택 취소"
        )

        raise SystemExit


    # C
    if key in (
        ord("c"),
        ord("C"),
    ):

        start_point = None
        end_point = None

        dragging = False
        roi_selected = False

        print(
            "ROI 다시 선택"
        )


    # ENTER 또는 SPACE
    if key in (
        13,
        10,
        32,
    ):

        if not roi_selected:

            print(
                "먼저 ROI를 드래그하세요."
            )

            continue

        break


cv2.destroyAllWindows()


# --------------------------------------------------
# 좌표 정리
# --------------------------------------------------

x1, y1 = start_point
x2, y2 = end_point

left = min(
    x1,
    x2,
)

right = max(
    x1,
    x2,
)

top = min(
    y1,
    y2,
)

bottom = max(
    y1,
    y2,
)


# --------------------------------------------------
# 화면 좌표 → 원본 좌표
# --------------------------------------------------

left = int(
    left / scale
)

right = int(
    right / scale
)

top = int(
    top / scale
)

bottom = int(
    bottom / scale
)


# --------------------------------------------------
# ROI Crop
# --------------------------------------------------

roi = image[
    top:bottom,
    left:right,
]

if roi.size == 0:

    raise ValueError(
        "ROI 영역이 올바르지 않습니다."
    )


# --------------------------------------------------
# ROI 저장
# --------------------------------------------------

cv2.imwrite(
    str(image_path),
    roi,
)


print()
print("ROI 저장 완료")
print("-----------------------------")
print("left   :", left)
print("top    :", top)
print("right  :", right)
print("bottom :", bottom)
print()
print(
    "저장:",
    image_path,
)
```

실행:

```
python roi_temp/crop_challenge.py
```

윈도우창에 이미지가 보이면 마우스로 드레그 하여 ROI영역을 만든뒤 엔터를 눌러서 저장합니다.

그리고 samples/day02/challenge.jpg를 확인하여 ROI적용이 되었는지 확인합니다.

---

# 29. 필요한 전처리 후보를 선택합니다

ROI로 자른 `challenge.jpg`를 다시 확인합니다.

이번 이미지에서는 병뚜껑이 밝은 색이고 주변 배경도 회색 계열이므로 다음을 먼저 관찰합니다.

```
병뚜껑과 배경의 밝기 차이가 충분한가?

병뚜껑의 외곽 경계가 잘 보이는가?

세로 홈과 작은 형태가 유지되는가?

전체 이미지가 너무 어둡거나 뿌옇지는 않은가?

전처리를 하지 않아도
객체와 배경이 충분히 분리될 수 있는가?
```

문제를 정의한 뒤 필요한 기술만 후보로 선택합니다.

예:

```
전체 밝기가 부족하다
→ Brightness / Gamma

병뚜껑과 배경의 대비가 약하다
→ Contrast / CLAHE

경계가 약하다
→ Sharpening

현재 상태로 충분하다
→ 전처리 미사용
```

모든 기술을 적용할 필요는 없습니다.

> 중요한 것은 이미지를 더 밝고 선명하게 만드는 것이 아니라, 병뚜껑과 배경을 더 안정적으로 분리할 수 있게 만드는 것입니다.

---

# 30. `17_day02_challenge.py`를 완성합니다

새 파일을 만듭니다.

```
src/17_day02_challenge.py
```

강사는 기본 구조와 Otsu 비교 코드까지 제공합니다.

학생은 **TODO 부분에서 자신이 선택한 전처리만 작성**합니다.

```python
from pathlib import Path

import cv2


image_path = Path(
    "samples/day02/challenge.jpg"
)

output_dir = Path(
    "outputs/day02/challenge"
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
        f"이미지를 읽지 못했습니다: {image_path}"
    )


# --------------------------------------------------
# TODO
#
# 1. 이미지의 문제를 판단합니다.
# 2. 필요한 전처리를 선택합니다.
# 3. Parameter를 직접 정합니다.
# 4. result에 최종 결과를 저장합니다.
# --------------------------------------------------


result = image.copy()


# --------------------------------------------------
# 전처리 결과 저장
# --------------------------------------------------

result_path = (
    output_dir
    / "challenge_result.jpg"
)

cv2.imwrite(
    str(result_path),
    result,
)


# --------------------------------------------------
# Gray 변환
# --------------------------------------------------

def to_gray(source):

    if len(source.shape) == 2:
        return source

    return cv2.cvtColor(
        source,
        cv2.COLOR_BGR2GRAY,
    )


original_gray = to_gray(
    image
)

result_gray = to_gray(
    result
)


# --------------------------------------------------
# 전처리 전 Otsu
#
# 이번 Challenge에서는
# 밝은 병뚜껑을 흰색 객체로 확인합니다.
# --------------------------------------------------

_, original_otsu = cv2.threshold(
    original_gray,
    0,
    255,
    cv2.THRESH_BINARY
    + cv2.THRESH_OTSU,
)


# --------------------------------------------------
# 전처리 후 Otsu
# --------------------------------------------------

_, result_otsu = cv2.threshold(
    result_gray,
    0,
    255,
    cv2.THRESH_BINARY
    + cv2.THRESH_OTSU,
)


cv2.imwrite(
    str(
        output_dir
        / "challenge_original_otsu.jpg"
    ),
    original_otsu,
)

cv2.imwrite(
    str(
        output_dir
        / "challenge_result_otsu.jpg"
    ),
    result_otsu,
)


print("입력 :", image_path)
print("결과 :", result_path)
print("Challenge 완료")
```

학생이 직접 수정하는 핵심 부분은:

```python
result = image.copy()
```

선택한 전처리를 적용한 결과를 `result`에 저장합니다.

### 왜 앞 실습과 Threshold 방식이 다른가요?

앞에서 사용한 기준 이미지는 검은 병뚜껑이었기 때문에:

```
어두운 병뚜껑
→ 흰색 객체
```

로 만들기 위해 `THRESH_BINARY_INV`를 사용했습니다.

이번 Challenge에서는 병뚜껑이 밝은 색이므로:

```
밝은 병뚜껑
→ 흰색 객체
```

로 보기 위해 `THRESH_BINARY`를 사용합니다.

> Threshold 방식도 항상 하나를 외우는 것이 아니라 검사할 객체를 어떻게 분리할 것인지에 따라 선택합니다.

---

# 31. Parameter를 바꾸어 결과를 비교합니다

처음 선택한 Parameter가 정답이라고 생각하지 않습니다.

선택한 기술의 값을 2~3개 정도 바꾸어 비교합니다.

```
Brightness
→ beta 변경

Gamma
→ gamma 변경

CLAHE
→ clipLimit 변경

Sharpening
→ 적용 강도가 너무 크지 않은지 확인
```

결과에서는 다음을 봅니다.

```
병뚜껑과 배경의 차이가 커졌는가?

병뚜껑 외곽이 잘 유지되는가?

세로 홈이 사라지지 않았는가?

배경 Texture나 Noise가
오히려 강해지지는 않았는가?
```

가장 강한 값을 찾는 것이 목적은 아닙니다.

---

# 32. 전처리 전·후 Otsu 결과를 비교합니다

실행 후 다음 세 파일을 확인합니다.

```
challenge_result.jpg

challenge_original_otsu.jpg

challenge_result_otsu.jpg
```

특히 다음 두 결과를 비교합니다.

```
전처리 전
challenge_original_otsu.jpg

        VS

전처리 후
challenge_result_otsu.jpg
```

확인합니다.

```
밝은 병뚜껑 영역이
흰색 객체로 잘 나타나는가?

병뚜껑 외곽이 끊기지 않는가?

배경까지 흰색으로 많이 선택되지 않는가?

전처리 전보다 객체와 배경의
분리가 실제로 좋아졌는가?
```

병뚜껑 아래의 밝은 링이나 병목 일부가 함께 흰색으로 나타날 수도 있습니다.

이 경우 바로 실패라고 결론 내리지 않습니다.

```
Otsu만으로 완전히 분리되는가?

        ↓

아니라면

3일차에서
Morphology
Contour
Area / Position
Rule

등을 이용해 더 구분할 수 있는가?
```

까지 연결해서 생각합니다.

---

# 33. 최종적으로 사용할지 결정합니다

결과를 보고 다음 중 하나를 선택합니다.

```
사용

미사용

3일차에서 추가 확인
```

기록 예시는 다음 형태로 작성합니다.

```
문제:
병뚜껑과 배경의 대비가 약함

시도:
Gamma
CLAHE

최종 선택:
실제 결과를 보고 작성

좋아진 점:
작성

나빠진 점:
작성

Otsu 결과:
전처리 전·후 차이를 작성

판단:
사용 / 미사용 / 3일차 추가 확인
```

강사가 특정 기술을 정답으로 미리 정하지 않습니다.

> **실제 Otsu 분리 결과가 좋아졌는지를 보고 결정합니다.**

---

# PART 6. 내가 한 실험을 기록합니다

Challenge가 끝났다면 새로운 전처리를 계속 추가하지 않습니다.

이제 **무엇을 시험했고 왜 그 방법을 선택했는지** 기록합니다.

---

# 34. Challenge 결과를 Parameter Table에 추가합니다

다음 파일을 사용합니다.

```
reports/day02_parameter_table.csv
```

Challenge에서 실제로 사용한 방법과 Parameter를 기록합니다.

```
image
problem
method
parameter
improved
worse
next_inspection_helpful
```

예를 들어 Gamma 값을 여러 개 시험했다면 각각 기록할 수 있습니다.

```
어떤 값을 사용했는가?

무엇이 좋아졌는가?

무엇이 나빠졌는가?

Otsu 분리에 도움이 되었는가?
```

> 예제 값을 그대로 입력하지 않고 **자신이 실제로 실행한 결과를 기록합니다.**

---

# 35. Challenge 선택 이유를 기록합니다

다음 파일의 Challenge 부분을 작성합니다.

```
reports/day02_preprocess_reason.md
```

```
## Challenge

### 문제

작성:

### 시도한 전처리

작성:

### 비교한 Parameter

작성:

### 최종 선택

작성:

### 선택 이유

작성:

### 좋아진 점

작성:

### 나빠진 점

작성:

### Otsu 비교 결과

작성:

### 최종 판단

- 사용
- 미사용
- 3일차 추가 확인
```

길게 작성할 필요는 없습니다.

> **왜 이 방법과 Parameter를 선택했는지 설명할 수 있으면 됩니다.**

---

# 36. 잘되지 않았던 시도도 하나 기록합니다

잘된 결과만 기록하지 않습니다.

실험 중 효과가 좋지 않았던 방법 하나도 남깁니다.

예:

```
시도:
Gamma를 강하게 적용

현상:
병뚜껑은 밝아졌지만
배경도 함께 밝아졌다.

Otsu 결과:
객체와 배경의 분리가
크게 좋아지지 않았다.

판단:
현재 값은 사용하지 않는다.
```

또는:

```
시도:
CLAHE를 강하게 적용

현상:
병뚜껑의 홈은 강조되었지만
배경 Texture도 함께 강조되었다.

판단:
현재 Parameter는 사용하지 않는다.
```

실패한 결과도 중요한 실험 결과입니다.

---

# 37. `glare.jpg`는 별도의 촬영환경 사례입니다

`glare.jpg`는 Mini Challenge의 입력 이미지가 아닙니다.

강한 조명이나 반사가 생겼을 때 **소프트웨어 전처리 문제인지 촬영환경 문제인지 구분**하기 위한 사례입니다.

```
samples/day02/glare.jpg
```

확인합니다.

```
강한 반사로 정보가 사라졌는가?

밝은 영역이 흰색에 가깝게 포화되었는가?

원래 경계와 세부정보가 남아 있는가?
```

정보가 이미 손실되었다면:

```
전처리를 계속 추가
        X

조명 방향
조명 밝기
카메라 각도
촬영 위치 조정
        O
```

를 먼저 생각합니다.

> 모든 이미지 문제를 코드로 해결하는 것이 머신비전의 목적은 아닙니다.

---

# PART 7. 오늘 배운 판단 방법을 정리합니다

여기부터는 새로운 실습을 하지 않습니다.

Mini Challenge에서 사용한 판단 과정을 정리합니다.

---

# 38. 이미지 문제에 따라 기술 후보를 생각합니다

|이미지 문제|먼저 생각할 후보|확인할 점|
|---|---|---|
|너무 어두움|Brightness, Gamma|밝은 영역 포화|
|대비가 약함|Contrast, CLAHE|Noise·Texture 증가|
|일반 Noise|Gaussian|경계 흐림|
|점 Noise|Median|작은 특징 손실|
|Noise + 경계|Bilateral|처리시간|
|흐린 경계|Sharpening|Noise 강조|
|객체·배경 분리|Threshold / Otsu|실제 분리 결과|
|색상이 중요|HSV|조명 변화|
|강한 반사·포화|촬영환경 개선|정보 손실 여부|

이 표는 정답표가 아닙니다.

> 이미지 문제를 발견했을 때 **어떤 기술부터 시험해 볼지 생각하기 위한 후보표**입니다.

---

# 39. Mini Challenge의 문제 해결 순서를 정리합니다

이번 Challenge에서 수행한 과정은 다음과 같습니다.

```
원본 challenge.jpg
        ↓
병뚜껑 주변 ROI 선택
        ↓
ROI 이미지 확인
        ↓
이미지 문제 정의
        ↓
필요한 전처리 후보 선택
        ↓
Parameter 변경·비교
        ↓
전처리 결과 확인
        ↓
Otsu 전·후 결과 비교
        ↓
사용 / 미사용 / 추가 확인 결정
        ↓
선택 이유와 실패 결과 기록
```

핵심은 다음과 같습니다.

```
배운 기술을 모두 적용한다.
              X

문제를 먼저 확인한다.
        ↓
필요한 기술만 시험한다.
        ↓
객체와 배경의 분리가
실제로 좋아졌는지 확인한다.
              O
```

이번 `challenge.jpg`처럼 **병뚜껑과 배경이 모두 비교적 밝은 계열인 이미지**에서는 단순히 더 밝게 만드는 것이 반드시 좋은 결과를 의미하지 않습니다.

> **최종 기준은 밝기나 선명함 자체가 아니라, 병뚜껑 영역을 다음 검사 단계에서 얼마나 안정적으로 사용할 수 있는가입니다.**
---

# 40. README에 Day02 내용을 추가합니다

2일차에서 무엇을 배우고 어떤 결과를 만들었는지 `README.md`에 간단히 기록합니다.

1일차 내용 아래에 추가합니다.

```markdown
## Day 02 — Image Enhancement와 검사 영상 품질 개선

1일차에 설정한 ROI를 이용하여
촬영조건과 영상 품질이 검사 결과에 어떤 영향을 주는지 확인했다.

### 실습 내용

- BGR / Gray / HSV 비교
- Brightness / Contrast
- Gamma Correction
- CLAHE
- Gaussian / Median / Bilateral Filter
- Sharpening
- Threshold / Otsu
- 전처리 전·후 객체 분리 결과 비교
- Mini Challenge를 통한 전처리 직접 선택

### 핵심 결과

- `outputs/day02/compare/preprocess_compare.png`
- `outputs/day02/challenge/challenge_result.jpg`
- `reports/day02_parameter_table.csv`
- `reports/day02_preprocess_reason.md`

### 오늘의 핵심

전처리 기술을 먼저 선택하는 것이 아니라

문제 확인
→ 필요한 전처리 선택
→ Parameter 비교
→ Threshold / Otsu 결과 확인
→ 사용 여부 결정

순서로 판단한다.

### 다음 학습

Day 03에서는 Binary 이미지에
Morphology와 Contour를 적용하여
객체의 면적·개수·위치를 측정하고
Rule 기반 OK / NG 판정으로 연결한다.
```

---

# 41. 2일차 결과물을 확인합니다

오늘 만든 결과가 정상적으로 저장되었는지 확인합니다.

```bash
find outputs/day02 -type f
```

최소한 다음 결과는 확인합니다.

```
outputs/day02/compare/preprocess_compare.png
outputs/day02/challenge/challenge_result.jpg
reports/day02_parameter_table.csv
reports/day02_preprocess_reason.md
```

코드도 확인합니다.

```
09_color_space.py
10_brightness_contrast.py
11_gamma.py
12_clahe.py
13_filter_compare.py
14_sharpen.py
15_threshold_otsu.py
16_preprocess_compare.py
17_day02_challenge.py
```

---

# 42. Local Git에 오늘 작업을 기록합니다

먼저 변경된 파일을 확인합니다.

```bash
git status
```

코드와 실습 기록을 추가합니다.

```bash
git add src reports README.md
```

Commit 합니다.

```bash
git commit -m "feat: complete day02 image enhancement practice"
```

기록을 확인합니다.

```bash
git log --oneline
```

`samples/`와 `outputs/`가 `.gitignore`에 포함되어 있다면 이미지가 Git에 등록되지 않는 것은 정상입니다.

---

# 43. 2일차 완료 확인

수업을 마치기 전에 다음 내용을 확인합니다.

- [ ] 이미지 품질 문제에 따라 필요한 전처리를 선택할 수 있다.
- [ ] Parameter를 변경하며 결과 차이를 비교했다.
- [ ] Threshold / Otsu로 객체와 배경의 분리를 확인했다.
- [ ] 전처리가 항상 필요한 것은 아니라는 것을 이해했다.
- [ ] Mini Challenge에서 직접 전처리를 선택했다.
- [ ] 선택 이유와 실패한 결과를 기록했다.
- [ ] README와 Local Git을 정리했다.

---

# 44. 3일차로 연결합니다

2일차에서는 검사에 사용할 이미지의 품질 문제를 확인하고, 필요한 전처리를 선택한 뒤 Threshold / Otsu로 병뚜껑과 배경이 분리되는지 확인했습니다.

```
1일차
검사 대상 정의
→ ROI 설정

        ↓

2일차
이미지 품질 문제 확인
→ 필요한 전처리 선택
→ Parameter 비교
→ Threshold / Otsu
→ Binary Image 생성 (otsu.jpg 자체가 Binary Image)

        ↓

3일차
Binary Image 정리
→ Morphology
→ Contour
→ Area / Count / Position 측정
→ Rule
→ OK / NG
```

즉 2일차 결과는 아직 최종 검사 결과가 아닙니다.

> 3일차부터는 오늘 만든 Binary 이미지에서 실제 객체를 찾고 숫자로 측정하여 OK / NG 판정을 시작합니다.