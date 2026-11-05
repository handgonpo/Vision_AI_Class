## COCO8-Pose · 실제 MP4 · IP Webcam 실습

오늘은 교과 10에서 앞으로 사용할 **Pose Estimation 실습환경을 만들고**, 실제 이미지·동영상·IP Webcam을 이용하여 사람의 자세가 관절 데이터로 바뀌는 시작 과정을 직접 확인합니다.

1일차에서는 Pose 모델을 새로 학습하지 않습니다. 이미 학습된 `yolo11n-pose.pt`를 사용하여 다음 흐름이 정상적으로 동작하는지 확인하는 것이 핵심입니다.

![[Pasted image 20260903120320.png]]

```text
실제 입력
JPG · MP4 · IP Webcam
        ↓
      Frame
        ↓
    YOLO Pose
        ↓
Person Bounding Box
        ↓
COCO 17 Keypoint
        ↓
     Skeleton
```

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

```
- Pose Estimation과 Action Recognition의 차이를 이해합니다.
- JPG·MP4·IP Webcam의 입력 방식은 달라도 결국 Frame으로 처리된다는 것을 이해합니다.
- WSL Ubuntu에 교과 10 프로젝트와 Python 가상환경을 구성합니다.
- OpenCV, PyTorch, Ultralytics와 GPU·CUDA 실행환경을 확인합니다.
- COCO8-Pose 실제 이미지를 OpenCV로 읽습니다.
- yolo11n-pose.pt를 이용해 사람의 Pose를 추론합니다.
- Person Bounding Box, COCO 17 Keypoint, Skeleton의 의미를 설명합니다.
- 실제 MP4의 FPS·Frame 수·해상도를 확인하고 Frame별 Pose 결과를 만듭니다.
- IP Webcam으로 스마트폰 카메라의 Frame을 WSL에서 읽고 저장합니다.
- 자신이 촬영한 이미지에 같은 YOLO Pose 코드를 적용합니다.
- 오늘 만든 코드와 결과를 README에 정리하고 Local Git으로 저장합니다.
```

> **오늘의 핵심**
>
> 모델을 학습하는 날이 아니라, 실제 입력 데이터가 `Frame → Pose → Keypoint`로 변환되는 전체 시작 과정을 직접 실행하고 이해하는 날입니다.

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260903120717.png]]

```text
교과 10 전체 구조 이해
        ↓
Pose Estimation과
Action Recognition 구분
        ↓
JPG · MP4 · IP Webcam의
입력 구조 이해

        ↓

프로젝트 폴더 생성
        ↓
Python 가상환경 생성
        ↓
OpenCV · PyTorch · Ultralytics 설치
        ↓
프로젝트 설정 파일과 공통 코드 작성

        ↓

COCO8-Pose + 실제 MP4 배치
        ↓
COCO8-Pose 압축 해제
        ↓
실제 이미지 8장 확인
        ↓
OpenCV로 이미지 읽기
        ↓
실제 MP4 정보 확인
FPS · Frame 수 · 해상도

        ↓

GPU · CUDA · PyTorch 환경 확인
        ↓
YOLO Pose 모델 확인
        ↓
COCO8-Pose 이미지에 첫 Pose 추론
        ↓
Person Bounding Box
        ↓
COCO 17 Keypoint
        ↓
Skeleton 이해

        ↓

실제 MP4
        ↓
Video → Frame 구조 이해
        ↓
각 Frame에 YOLO Pose 적용
        ↓
Pose 결과 영상 확인

        ↓

스마트폰 IP Webcam 연결
        ↓
WSL에서 Frame 수신 확인
        ↓
정면 · 양팔 들기 · 먼 거리
이미지 저장
        ↓
저장한 실제 이미지에 YOLO Pose 적용

        ↓

입력 조건별 결과 비교
        ↓
Mini Challenge
        ↓
.gitignore · README 작성
        ↓
Local Git Commit
        ↓
자가 체크
```

---

# 오늘 사용할 실제 데이터

수업에서 다음 두 파일을 사용합니다.

```text
coco8-pose.zip
solution_ci_pose_demo.mp4
```

카메라 실습에는 스마트폰의 **IP Webcam**을 사용합니다.

| 입력 | 오늘 사용하는 이유 |
|---|---|
| `coco8-pose.zip` | 실제 사람 이미지에서 Person BBox, COCO 17 Keypoint, Skeleton 확인 |
| `solution_ci_pose_demo.mp4` | 실제 영상이 Frame으로 나뉘고 Frame마다 Pose가 적용되는 과정 확인 |
| 스마트폰 IP Webcam | 자신의 실제 카메라 Frame을 입력하여 같은 Pose Pipeline 확인 |

`COCO8-Pose`의 `labels` 폴더는 오늘 학습에 직접 사용하지 않습니다. 오늘은 이미 학습된 Pose 모델로 **추론(Inference)** 만 수행합니다.

---

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| WSL Ubuntu | AI 개발환경 사용 | Pose 코드를 실행할 기본 작업공간 |
| Python | 실습 코드 작성 | 이미지·영상·AI 모델을 연결하는 언어 |
| `uv` | 가상환경·패키지 관리 | 교과 10 전용 Python 환경 관리 |
| OpenCV | JPG·MP4·IP Webcam 처리 | 이미지와 영상 Frame을 읽고 저장 |
| PyTorch | 딥러닝 실행환경 확인 | YOLO Pose가 동작하는 기반 |
| Ultralytics YOLO | Pose 추론 | 사람의 위치와 관절을 찾는 모델 사용 |
| COCO8-Pose | 실제 Pose 이미지 데이터 | 17개 사람 관절 구조 확인 |
| IP Webcam | 스마트폰 영상 스트림 | USB 웹캠 대신 실제 카메라 Frame 입력 |
| Git | 코드 버전관리 | 오늘 작성한 코드를 기록 |

---

# PART 1. Pose 수업의 전체 구조를 이해합니다

---

# 1. Pose Estimation과 Action Recognition을 구분합니다

두 기술은 서로 연결되지만 같은 작업은 아닙니다.

## Pose Estimation

![[Pasted image 20260903121044.png]]

Pose Estimation은 **한 장의 이미지 또는 한 Frame에서 사람의 주요 관절 위치를 찾는 작업**입니다.

```text
사람 이미지
    ↓
사람 위치 찾기
    ↓
Pose Estimation
    ↓
코
어깨
팔꿈치
손목
골반
무릎
발목
```

Pose 결과는 화면에 보이는 점과 선만 의미하지 않습니다.

내부적으로는 다음과 같은 관절 좌표가 만들어집니다.

```text
왼쪽 어깨 = (432, 217)
왼쪽 팔꿈치 = (397, 301)
왼쪽 손목 = (365, 387)
```

## Action Recognition

Action Recognition은 **여러 Frame에서 관절이 시간에 따라 어떻게 변하는지 보고 동작을 판단하는 작업**입니다.

![[Pasted image 20260903121657.png]]

예를 들어 팔 올리기는 다음처럼 표현할 수 있습니다.

```text
Frame 1
팔 아래
    ↓
Frame 2
팔 중간
    ↓
Frame 3
팔 위
    ↓
raise_arm
```

교과 10의 전체 흐름은 다음과 같습니다.

![[Pasted image 20260903131926.png]]

```text
Frame
  ↓
Pose
  ↓
Keypoint
  ↓
Feature
  ↓
Sequence
  ↓
Action Recognition
```

오늘은 이 중 다음 부분까지 직접 확인합니다.

```text
입력
 ↓
Frame
 ↓
Pose
 ↓
Keypoint
```

---

# 2. JPG·MP4·IP Webcam은 어떻게 다른가요?

세 입력은 Frame을 얻는 방법이 다릅니다.

![[Pasted image 20260903134552.png]]

```text
JPG 이미지
    ↓
Frame 1개
```

```text
MP4 영상
    ↓
Frame 1
Frame 2
Frame 3
...
```

```text
IP Webcam
    ↓
실시간 Frame
실시간 Frame
실시간 Frame
...
```

하지만 Frame을 얻은 뒤의 AI 처리 과정은 같습니다.

```text
Frame
  ↓
YOLO Pose
  ↓
Person Bounding Box
  ↓
COCO 17 Keypoint
  ↓
Skeleton
```

즉 오늘은 입력 방법은 달라도 Pose Pipeline의 중심 구조는 같다는 것을 확인합니다.

---

# 3. 세 가지 입력을 순서대로 사용하는 이유를 이해합니다

처음부터 실시간 카메라로 시작하면 카메라·네트워크·AI 모델 문제를 한꺼번에 확인해야 하므로 오류 원인을 찾기 어렵습니다.

따라서 다음 순서로 실습합니다.

```text
1. JPG
가장 단순한 입력으로 Pose가 동작하는지 확인
        ↓
2. MP4
여러 Frame을 반복 처리하는 방법 확인
        ↓
3. IP Webcam
계속 들어오는 실제 카메라 Frame 확인
```

이 순서대로 진행하면 문제가 생겼을 때 어느 단계의 문제인지 찾기 쉽습니다.

---

# 4. 각 파일들의 역할

```
subject10_pose_action/
├── configs/
│   └── project.json
├── src/
│   └── common.py
├── scripts/
│   ├── 00_check_environment.py
│   ├── 01_image_input.py
│   ├── 02_video_input.py
│   ├── 03_pose_image.py
│   ├── 04_pose_video.py
│   └── 05_ip_webcam_capture.py
├── data/
│   └── day01/
│       ├── coco8_pose/      (COCO8-Pose)
│       ├── video/           (solution_ci_pose_demo.mp4)
│       └── ip_webcam/       (스마트폰 촬영 결과 저장)
├── artifacts/               (추후 산출물)
├── reports/
│   └── day01/
│       └── challenge/
│           └── challenge_notes.md
├── pyproject.toml
├── .gitignore
└── README.md
```

### 1. Python 코드 및 설정 파일

|파일명|역할|설명|
|---|---|---|
|`configs/project.json`|프로젝트 공통 설정|YOLO Pose 모델명, 이미지 크기, Person Confidence, Keypoint Confidence, GPU·CPU 선택값 등을 관리합니다.|
|`src/common.py`|공통 기능 관리|프로젝트 경로 확인, JSON 설정 불러오기, GPU·CPU 실행 장치 선택, 필요한 폴더 생성 기능을 제공합니다.|
|`scripts/00_check_environment.py`|개발환경 점검|Python, OpenCV, PyTorch, CUDA, GPU 등 Pose 실습에 필요한 실행환경이 정상인지 확인합니다.|
|`scripts/01_image_input.py`|이미지 입력 확인|OpenCV로 JPG 이미지를 읽고 너비, 높이, 채널 수, 데이터 타입 등을 확인합니다.|
|`scripts/02_video_input.py`|동영상 입력 확인|OpenCV로 MP4 영상을 읽어 FPS, 전체 Frame 수, 해상도, 재생시간 등을 확인합니다.|
|`scripts/03_pose_image.py`|이미지 Pose 추론|사전학습된 YOLO Pose 모델을 사용하여 이미지 속 사람의 Bounding Box, COCO 17 Keypoint, Skeleton을 추출하고 시각화합니다.|
|`scripts/04_pose_video.py`|동영상 Pose 추론|MP4 영상의 Frame을 순서대로 읽어 YOLO Pose를 적용하고, Pose 결과가 표시된 영상을 생성합니다.|
|`scripts/05_ip_webcam_capture.py`|스마트폰 카메라 연결|IP Webcam으로 스마트폰 영상을 수신하고 정면, 양팔 들기, 먼 거리 등의 이미지를 촬영하여 저장합니다.|
|`pyproject.toml`|Python 프로젝트 환경 설정|프로젝트에서 사용할 Python 버전과 OpenCV, PyTorch, Ultralytics 등 필요한 라이브러리 목록을 관리합니다.|
|`.gitignore`|Git 관리 제외 설정|`.venv`, 데이터셋, 모델 가중치, 대용량 영상, 실행 결과 등 Git에 업로드하지 않을 파일과 폴더를 지정합니다.|
|`README.md`|프로젝트 실행 안내|프로젝트 목적, 개발환경, 폴더 구조, 실행방법 및 실습 결과를 정리합니다.|

### 2. 데이터 및 결과물 저장 폴더

| 폴더명 / 파일명                                    | 역할                   | 설명                                                                            |
| -------------------------------------------- | -------------------- | ----------------------------------------------------------------------------- |
| `data/day01/`                                | 1일차 데이터 관리           | 이미지, 동영상, 스마트폰 촬영 데이터를 입력 종류별로 구분하여 관리합니다.                                    |
| `data/day01/coco8_pose/`                     | COCO8-Pose 데이터 저장    | `coco8-pose.zip` 압축 해제 후 사람 이미지 8장과 라벨 파일을 보관합니다. 1일차에서는 이미지를 Pose 추론에 사용합니다. |
| `data/day01/video/`                          | MP4 원본 영상 저장         | `solution_ci_pose_demo.mp4`를 저장하고, Frame별 Pose 추론 실습의 입력 데이터로 사용합니다.          |
| `data/day01/ip_webcam/`                      | 스마트폰 촬영 이미지 저장       | IP Webcam으로 촬영한 정면, 양팔 들기, 먼 거리 등의 이미지를 저장하고 Pose 결과를 비교합니다.                  |
| `artifacts/`                                 | 모델 및 산출물 관리          | 이후 수업에서 생성되는 모델이나 재사용할 산출물을 보관하기 위한 폴더입니다.                                    |
| `reports/day01/`                             | 1일차 실습 결과 저장         | 이미지·영상의 Pose 추론 결과와 실습 보고 자료를 저장합니다.                                          |
| `reports/day01/challenge/`                   | Mini Challenge 관리    | 학생들이 스스로 진행하는 추가 실습의 결과와 분석 내용을 관리합니다.                                        |
| `reports/day01/challenge/challenge_notes.md` | Mini Challenge 결과 기록 | 학생들이 입력 조건별 Pose 결과, 관절 검출 차이, 문제점 및 개선 아이디어를 기록합니다.                          |
![[Pasted image 20261104095029.png]]

---

# PART 2. 교과 10 프로젝트와 개발환경을 만듭니다

---

# 5. WSL에 교과 10 프로젝트 폴더를 생성합니다

먼저 WSL Ubuntu 터미널을 엽니다.

현재 위치를 확인합니다.

```bash
pwd
```

홈 디렉터리로 이동합니다.

```bash
cd ~
```

교과 10 프로젝트 폴더를 만듭니다.

```bash
mkdir -p ~/ai_vision/subject10_pose_action
```

프로젝트 폴더로 이동합니다.

```bash
cd ~/ai_vision/subject10_pose_action
```

현재 위치를 다시 확인합니다.

```bash
pwd
```

다음과 비슷하면 정상입니다.

```text
/home/사용자계정/ai_vision/subject10_pose_action
```

VS Code를 현재 프로젝트 위치에서 엽니다.

```bash
code -r .
```

### 이 명령들이 하는 일

```text
홈 폴더로 이동
    ↓
subject10_pose_action 생성
    ↓
프로젝트 폴더로 이동
    ↓
VS Code에서 프로젝트 열기
```

---

# 6. 10일 동안 사용할 프로젝트 디렉터리를 만듭니다

앞으로 코드·설정·데이터·결과를 한 폴더에 섞지 않고 역할별로 나누어 관리합니다.

다음 명령을 실행합니다.

```bash
mkdir -p \
configs \
src \
scripts \
data/day01/coco8_pose \
data/day01/video \
data/day01/ip_webcam \
artifacts \
reports/day01
```

폴더가 생성되었는지 확인합니다.

```bash
find . -maxdepth 3 -type d | sort
```

기본 구조는 다음과 같습니다.

```text
subject10_pose_action/
│
├── configs/
├── src/
├── scripts/
│
├── data/
│   └── day01/
│       ├── coco8_pose/
│       ├── video/
│       └── ip_webcam/
│
├── artifacts/
├── reports/
│   └── day01/
│
├── pyproject.toml
├── .gitignore
└── README.md
```

각 폴더의 역할은 다음과 같습니다.

| 폴더 | 역할 |
|---|---|
| `configs` | 모델명, Confidence 등 프로젝트 설정 |
| `src` | 여러 실습에서 반복해서 사용할 공통 기능 |
| `scripts` | 수업 순서대로 실행할 Python 코드 |
| `data` | 원본 이미지·영상·카메라 캡처 |
| `artifacts` | 이후 만들어지는 모델 및 산출물 |
| `reports` | 실행 결과와 분석 결과 |

---

# 7. Python 가상환경을 만듭니다

가상환경은 교과 10에서 사용하는 Python 라이브러리를 다른 프로젝트와 분리하기 위해 사용합니다.

### `pip`와 `uv`는 무엇이 다른가요?

교과 9에서는 `pip`를 이용해 필요한 라이브러리를 하나씩 설치했습니다.

```
pip
→ 필요한 패키지를 직접 설치

예:
pip install opencv-python
```

교과 10에서는 `uv`를 사용하여 **가상환경과 라이브러리 목록을 프로젝트 단위로 함께 관리**합니다.

```
uv
→ 가상환경 생성
→ pyproject.toml 확인
→ 필요한 라이브러리 설치
→ uv.lock에 버전 기록
```

쉽게 정리하면 다음과 같습니다.

```
pip
→ 패키지를 설치하는 도구

uv
→ 가상환경 + 패키지 + 버전 관리를
   한 번에 관리하기 편한 도구
```

> 교과 10에서는 이후 여러 라이브러리와 실습환경을 계속 이어서 사용하므로, 프로젝트 환경을 일정하게 관리하기 위해 `uv`를 사용합니다.


Python 버전을 확인합니다.

```bash
python3 --version
```

Python 3.11 이상이면 오늘 작성할 `pyproject.toml` 조건과 맞습니다.

`uv`가 설치되어 있는지 확인합니다.

```bash
uv --version
```

uv가 설치되지 않았다면 uv설치를 먼저 합니다.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.local/bin/env
uv --version
```

가상환경을 생성합니다.

```bash
uv venv
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

현재 사용 중인 Python 위치를 확인합니다.

```bash
which python
```

다음처럼 프로젝트의 `.venv` 안에 있는 Python이 나오면 정상입니다.

```text
.../subject10_pose_action/.venv/bin/python
```

---

# 8. 필요한 Python 라이브러리를 설치합니다

이번에는 교과 10에서 사용할 라이브러리 목록을 `pyproject.toml`에 기록한 뒤 설치합니다.

### `pyproject.toml`은 무엇인가요?

`pyproject.toml`은 **이 Python 프로젝트에서 어떤 라이브러리와 Python 버전을 사용할지 기록하는 설정 파일**입니다.

예를 들어 현재 파일에는 다음 정보가 들어 있습니다.

```
프로젝트 이름
        ↓
사용할 Python 버전
        ↓
필요한 라이브러리 목록
        ↓
개발용 라이브러리 목록
```

`uv sync`를 실행하면 `uv`가 이 파일을 읽고 필요한 라이브러리를 `.venv` 가상환경에 설치합니다.

쉽게 말하면:

```
pyproject.toml
→ 이 프로젝트에 필요한 Python 환경을 적어 둔 목록표
```

라고 이해하면 됩니다.

**파일: `pyproject.toml`**

```toml
[project]
name = "subject10-pose-action"
version = "0.1.0"
description = "Subject 10 - 2D Pose Estimation and Action Analysis"
requires-python = ">=3.11"
dependencies = [
    "ultralytics",
    "opencv-python",
    "numpy",
    "pandas",
    "scikit-learn",
    "joblib",
    "matplotlib",
    "torch",
    "tqdm",
]

[dependency-groups]
dev = [
    "pytest",
    "ruff",
]
```

### 이 파일이 하는 일

```text
교과 10에서 필요한 라이브러리 목록을 기록
        ↓
uv가 목록을 읽음
        ↓
프로젝트의 .venv에 라이브러리 설치
        ↓
uv.lock에 설치 버전 기록
```

라이브러리를 설치합니다.

```bash
uv sync
```

> `uv sync`는 기본적으로 `dev` 그룹도 함께 동기화합니다.  
> 별도로 `uv sync --dev`를 입력하지 않습니다.

설치가 끝나면 가상환경을 다시 활성화합니다.

```bash
source .venv/bin/activate
```

주요 라이브러리가 정상적으로 import되는지 확인합니다.

```bash
python -c "import cv2; print('OpenCV:', cv2.__version__)"
```

```bash
python -c "import torch; print('PyTorch:', torch.__version__)"
```

```bash
python -c "import ultralytics; print('Ultralytics:', ultralytics.__version__)"
```

정상적인 출력 형태는 다음과 같습니다.

```text
OpenCV: 버전번호
PyTorch: 버전번호
Ultralytics: 버전번호
```

정확한 버전 숫자를 외우는 것이 목적은 아닙니다.

다음이 중요합니다.

```text
import 오류가 없음
        ↓
Python이 라이브러리를 정상적으로 찾음
```

---

# 9. 프로젝트 설정 파일과 공통 코드를 만듭니다

YOLO Pose를 실행하기 전에 **사용할 모델, 이미지 입력 크기, 검출 신뢰도, GPU·CPU 설정**을 먼저 정하겠습니다.

이 설정들을 여러 Python 파일에 반복해서 작성하지 않고, 하나의 JSON 파일에서 관리합니다.

## 9.1 프로젝트 설정 파일

### 왜 JSON 파일을 작성하나요?

JSON은 데이터를 저장할 뿐 아니라 **프로그램 실행에 필요한 설정값을 저장하는 용도**로도 사용합니다.

이번에 작성할 `project.json`은 학습 데이터가 아니라 **YOLO Pose의 공통 설정 파일**입니다.

### 어떤 설정을 저장하나요?

|설정 항목|역할|
|---|---|
|`pose_model`|사용할 YOLO Pose 모델 지정|
|`image_size`|모델의 입력 이미지 크기 지정 (640은 원본 이미지 크기를 바꾸는 것이 아님)|
|`person_confidence`|사람 검출 신뢰도 기준|
|`keypoint_confidence`|관절 신뢰도 기준 (2일차부터 활용)|
|`device`|`auto`이면 사용 가능한 CUDA GPU를 우선 선택하고, 없으면 CPU 사용|

### 어떤 Python 파일에서 사용하나요?

```
configs/project.json
        ↓ 설정값 읽기
    src/common.py
        ↓
        ├── 00_check_environment.py
        │       실행환경 및 모델 확인
        │
        ├── 03_pose_image.py
        │       이미지 Pose 추론
        │
        └── 04_pose_video.py
                영상 Pose 추론
```

즉, `project.json`은 설정값을 저장하고, `src/common.py`는 이 설정값을 읽어 다른 Python 코드에서 사용할 수 있도록 도와줍니다.

### 두 가지 JSON의 차이

| 구분    | project.json (설정 파일) | 결과 JSON (결과 데이터)   |
| ----- | -------------------- | ------------------ |
| 작성 시점 | 코드 실행 전              | 코드 실행 후            |
| 목적    | 프로그램을 어떻게 실행할지 지정    | 실행 결과를 저장하거나 전달    |
| 예시    | 모델명, 이미지 크기, 신뢰도     | 검출된 사람, 관절 좌표, 신뢰도 |

예를 들어 실행 전 — 설정 JSON

```json
{
  "pose_model": "yolo11n-pose.pt",
  "image_size": 640,
  "person_confidence": 0.25
}
```

의미: "이 모델을 사용하고, 이미지 입력 크기는 640으로 설정하고, 사람 검출 신뢰도는 0.25 이상으로 해주세요."

실행 후 — 결과 JSON (예시)

```json
{
  "person_detected": true,
  "keypoint_count": 17,
  "confidence": 0.92
}
```

의미: "실행 결과 사람이 검출되었고, 관절 17개의 결과가 생성되었습니다."

![[Pasted image 20261104101352.png]]

```
project.json
(실행 전 설정값)
      ↓
Python에서 JSON 읽기
      ↓
Python 딕셔너리로 변환
      ↓
YOLO Pose 실행
      ↓
결과 데이터 생성
      ↓
필요하면 결과를 JSON으로 저장
```

---
### 프로젝트 설정 파일을 작성합니다

**파일: `configs/project.json`**

`project.json`은 
`src/common.py`를 통해 읽히며, 
`00_check_environment.py`, 
`03_pose_image.py`, 
`04_pose_video.py`에서 사용됩니다.

```json
{
  "project_name": "subject10_pose_action",
  "pose_model": "yolo11n-pose.pt",
  "image_size": 640,
  "person_confidence": 0.25,
  "keypoint_confidence": 0.50,
  "device": "auto"
}
```

---
## 9.2 공통 Python 코드

앞에서 `project.json`에 YOLO 모델명, 이미지 크기, GPU·CPU 설정값을 저장했습니다.

이제 **여러 실습 코드에서 반복해서 사용하는 기능을 하나로 모은 `src/common.py`**를 작성하겠습니다.

이 파일은 YOLO Pose를 직접 실행하는 코드가 아니라, 다른 Python 파일이 필요할 때 불러서 사용하는 **공통 기능 모음**입니다.

### 어떤 기능을 담당하나요?

|공통 기능|역할|
|---|---|
|`PROJECT_ROOT`|프로젝트의 최상위 폴더 위치 확인|
|`load_config()`|`project.json`을 읽어 Python 딕셔너리로 변환|
|`resolve_device()`|CUDA 사용 가능 여부를 확인하여 GPU 또는 CPU 선택|
|`ensure_day01_dirs()`|1일차에 필요한 데이터·결과 폴더가 없으면 생성|

### 어떤 실습 파일에서 사용하나요?

```
configs/project.json
    │  실행 설정값 저장
    ↓
src/common.py
    │  설정 읽기 · GPU/CPU 선택
    │
    ├── 00_check_environment.py
    │     개발환경·모델 확인 및 폴더 생성
    │
    ├── 03_pose_image.py
    │     설정값을 사용하여 JPG Pose 추론
    │
    └── 04_pose_video.py
          설정값을 사용하여 MP4 Pose 추론
```

예를 들어 `03_pose_image.py`에서 다음과 같이 공통 기능을 불러와 사용합니다.

```python
from src.common import load_config, resolve_device

config = load_config()
device = resolve_device(config["device"])
```

- `load_config()` → JSON에 저장된 모델명, 이미지 크기 등의 설정을 가져옵니다.
    
- `resolve_device()` → 설정값에 따라 YOLO가 사용할 GPU 또는 CPU를 결정합니다.
    

이렇게 하면 각 실습 파일마다 설정을 읽거나 GPU를 확인하는 코드를 반복해서 작성할 필요가 없습니다.

### 코드 전체 흐름

```
프로젝트 최상위 폴더 찾기
        ↓
project.json 경로 확인
        ↓
JSON 설정값 읽기
        ↓
Python 딕셔너리로 변환
        ↓
GPU 사용 가능 여부 확인
        ↓
GPU 또는 CPU 선택
        ↓
필요한 폴더 생성 기능 준비
```

※ 마지막 폴더 생성 기능은 `ensure_day01_dirs()`를 호출했을 때 실행됩니다.

**파일:** `**src/common.py**`

이제 여러 실습 파일에서 재사용할 공통 기능을 직접 작성하겠습니다.

```python
from __future__ import annotations

# ① 필요한 라이브러리 불러오기
import json
from pathlib import Path
import torch


# ② 프로젝트 최상위 폴더와 설정 파일의 경로 지정
PROJECT_ROOT = Path(__file__).resolve().parents[1]
CONFIG_PATH = PROJECT_ROOT / "configs" / "project.json"


# ③ JSON 설정 파일 읽기
def load_config() -> dict:
    """project.json을 읽어 Python 딕셔너리로 반환합니다."""
    with CONFIG_PATH.open("r", encoding="utf-8") as file:
        return json.load(file)


# ④ GPU 또는 CPU 선택
def resolve_device(requested: str):
    """auto이면 CUDA 사용 가능 여부에 따라 GPU 또는 CPU를 선택합니다."""

    # auto가 아니면 사용자가 지정한 장치를 그대로 사용
    if requested != "auto":
        return requested

    # CUDA GPU를 사용할 수 있으면 GPU 0번 선택
    if torch.cuda.is_available():
        return 0

    # CUDA GPU를 사용할 수 없으면 CPU 선택
    return "cpu"


# ⑤ 1일차에 필요한 폴더 생성
def ensure_day01_dirs() -> None:
    """필요한 폴더가 없으면 생성합니다."""

    # 생성할 폴더 목록
    directories = [
        PROJECT_ROOT / "data" / "day01" / "coco8_pose",
        PROJECT_ROOT / "data" / "day01" / "video",
        PROJECT_ROOT / "data" / "day01" / "ip_webcam",
        PROJECT_ROOT / "reports" / "day01",
        PROJECT_ROOT / "artifacts",
    ]

    # 목록에 있는 폴더를 순서대로 확인하고 생성
    for directory in directories:
        directory.mkdir(parents=True, exist_ok=True)
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ json    → JSON 설정 파일 읽기
   ├─ Path    → 파일과 폴더 경로 관리
   └─ torch   → CUDA GPU 사용 가능 여부 확인
             ↓
② 프로젝트와 설정 파일 경로 지정
   ├─ PROJECT_ROOT → 프로젝트 최상위 폴더
   └─ CONFIG_PATH  → configs/project.json
             ↓
③ load_config() : JSON 설정 읽기
   ├─ project.json 파일 열기
   ├─ JSON 데이터를 Python 딕셔너리로 변환
   └─ 설정값 반환
             ↓
④ resolve_device() : GPU 또는 CPU 선택
   ├─ device가 auto인가?
   │
   ├─ 아니요 → 지정된 장치 그대로 반환
   │
   └─ 예 → CUDA GPU 사용 가능한가?
            ├─ 예 → GPU 0번 반환
            └─ 아니요 → CPU 반환
             ↓
⑤ ensure_day01_dirs() : 폴더 생성
   ├─ 데이터 폴더 목록 준비
   ├─ 결과 저장 폴더 목록 준비
   ├─ 각 폴더 존재 여부 확인
   └─ 없는 폴더만 생성
```

---

# PART 3. 실제 데이터를 프로젝트에 배치하고 입력을 확인합니다

---

# 10. 수업에서 사용할 두 파일을 프로젝트에 넣습니다

사용할 파일은 다음 두 개입니다.

```text
coco8-pose.zip
solution_ci_pose_demo.mp4
```

먼저 프로젝트 폴더를 Windows 파일 탐색기로 엽니다.

WSL 프로젝트 루트에서 실행합니다.

```bash
explorer.exe .
```

파일 탐색기에서 다음 위치에 파일을 넣습니다.

```text
coco8-pose.zip
→ data/day01/coco8-pose.zip

solution_ci_pose_demo.mp4
→ data/day01/video/solution_ci_pose_demo.mp4
```

복사가 끝나면 WSL 터미널에서 확인합니다.

```bash
ls -lh data/day01/coco8-pose.zip
```

```bash
ls -lh data/day01/video/solution_ci_pose_demo.mp4
```

두 명령에서 파일 크기가 표시되면 정상적으로 배치된 것입니다.

---

# 11. COCO8-Pose 압축을 풀고 8장의 이미지를 확인합니다

별도의 `unzip` 프로그램 설치 여부에 영향을 받지 않도록 Python 표준 `zipfile` 기능으로 압축을 풉니다.

```bash
python -m zipfile -e \
data/day01/coco8-pose.zip \
data/day01/coco8_pose
```

압축이 풀린 파일을 확인합니다.

```bash
find data/day01/coco8_pose -maxdepth 4 -type f | sort
```

COCO8-Pose에는 `images`와 `labels`가 들어 있습니다.

```text
images/
→ 실제 사람 이미지

labels/
→ 사람과 Keypoint의 정답 라벨
```

오늘은 `images`를 입력으로 사용합니다.

이미지 개수를 확인합니다.

```bash
find data/day01/coco8_pose \
-type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" \) \
| wc -l
```

정상적으로 압축이 풀렸다면 다음과 같이 나옵니다.

```text
8
```

> 압축 내부에 `coco8-pose/` 폴더가 한 단계 더 들어 있어도 문제가 없습니다.  
> 이후 실습에서는 `find` 명령으로 이미지를 찾기 때문에 실제 하위 폴더 깊이에 영향을 받지 않습니다.

---

# 12. 첫 번째 실습에 사용할 이미지를 선택합니다

COCO8-Pose에는 총 8장의 이미지가 들어 있습니다.

먼저 어떤 이미지들이 있는지 전체 목록을 확인합니다.

```bash
find data/day01/coco8_pose \
-type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" \) \
| sort
```

결과

```
data/day01/coco8_pose/coco8-pose/images/train/000000000036.jpg
data/day01/coco8_pose/coco8-pose/images/train/000000000049.jpg
data/day01/coco8_pose/coco8-pose/images/train/000000000077.jpg
data/day01/coco8_pose/coco8-pose/images/train/000000000086.jpg
data/day01/coco8_pose/coco8-pose/images/val/000000000110.jpg
data/day01/coco8_pose/coco8-pose/images/val/000000000113.jpg
data/day01/coco8_pose/coco8-pose/images/val/000000000136.jpg
data/day01/coco8_pose/coco8-pose/images/val/000000000151.jpg
```

이제 이 8장 중 **첫 번째 이미지를 첫 Pose 실습용으로 사용**합니다.

이미지 경로를 매번 길게 입력하지 않도록 첫 번째 이미지의 경로를 `FIRST_IMAGE`라는 변수에 저장합니다.

터미널에서 아래 코드와 같이 실행합니다.

```bash
FIRST_IMAGE=$(find data/day01/coco8_pose \
-type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" \) \
| sort | head -1)
```

쉽게 말하면 다음 작업입니다.

```
COCO8-Pose 이미지 8장 찾기
        ↓
파일명을 순서대로 정렬
        ↓
첫 번째 이미지 1장 선택
        ↓
그 이미지 경로를
FIRST_IMAGE 변수에 저장
```

여기서 말하는 변수는 현재 터미널(Bash) 안에 잠깐 만들어지는 변수입니다.
일시적인 변수로 껏다가 다시 켜면 사라집니다.

저장된 경로를 확인합니다.

```bash
echo "$FIRST_IMAGE"
```

결과

```
data/day01/coco8_pose/coco8-pose/images/train/000000000036.jpg
```

이제 Python 코드를 실행할 때 변수를 사용합니다

만약 변수로 설정하지 않았다면 아래 코드와 같이 전체 경로를 직접 입력해야 합니다.

```bash
python scripts/01_image_input.py \
  --source data/day01/coco8_pose/coco8-pose/images/train/000000000036.jpg
```

그러나 FIRST_IMAGE 변수 사용한다면

```bash
python scripts/01_image_input.py \
  --source "$FIRST_IMAGE"
```

위와 같습니다.

---

# 13. OpenCV로 실제 COCO8-Pose 이미지를 읽습니다

AI 모델에 넣기 전에 먼저 OpenCV가 실제 이미지 파일을 정상적으로 읽는지 확인합니다.

이 코드는 **이미지 파일을 읽고, 이미지의 너비·높이·채널 수·데이터 타입을 출력하는 코드**입니다.

### 의사코드로 읽어보기

```text
실행할 때 이미지 경로를 받는다
        ↓
파일이 실제로 존재하는지 확인한다
        ↓
OpenCV로 이미지를 읽는다
        ↓
읽기에 실패하면 오류를 알려준다
        ↓
너비 · 높이 · 채널 · 타입을 출력한다
```

**파일: `scripts/01_image_input.py`**

```python
from __future__ import annotations

# ① 필요한 라이브러리 불러오기
import argparse
from pathlib import Path
import cv2


# ② 터미널에서 이미지 경로 입력받기
def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--source",
        required=True,
        help="읽을 이미지 파일 경로",
    )

    return parser.parse_args()


# ③ 이미지 파일 경로 확인
def main() -> None:
    args = parse_args()

    image_path = Path(args.source)

    # 입력한 경로에 파일이 없으면 오류 발생
    if not image_path.exists():
        raise FileNotFoundError(
            f"파일이 없습니다: {image_path}"
        )

    # ④ OpenCV로 이미지 읽기
    image = cv2.imread(str(image_path))

    # 이미지 읽기에 실패하면 오류 발생
    if image is None:
        raise RuntimeError(
            f"OpenCV가 이미지를 읽지 못했습니다: {image_path}"
        )

    # ⑤ 이미지의 크기와 정보 확인
    height, width = image.shape[:2]

    print(f"Image path : {image_path}")
    print(f"Width      : {width}")
    print(f"Height     : {height}")
    print(f"Channels   : {image.shape[2]}")
    print(f"Shape      : {image.shape}")
    print(f"DataType   : {image.dtype}")


# ⑥ 프로그램 실행 시작
if __name__ == "__main__":
    main()
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ argparse → 터미널에서 입력값 받기
   ├─ Path     → 이미지 파일 경로 관리
   └─ cv2      → OpenCV로 이미지 읽기
                ↓
② 터미널에서 이미지 경로 입력받기
   ├─ --source 옵션 확인
   └─ 입력한 이미지 경로 저장
                ↓
③ 이미지 파일 경로 확인
   ├─ 입력받은 경로를 Path 객체로 변환
   └─ 파일이 존재하는가?
        ├─ 예 → 다음 단계
        └─ 아니요 → 오류 발생
                ↓
④ OpenCV로 이미지 읽기
   ├─ cv2.imread() 실행
   └─ 이미지 읽기에 성공했는가?
        ├─ 예 → 다음 단계
        └─ 아니요 → 오류 발생
                ↓
⑤ 이미지 정보 확인 및 출력
   ├─ Image path → 이미지 경로
   ├─ Width      → 가로 크기
   ├─ Height     → 세로 크기
   ├─ Channels   → 색상 채널 수
   ├─ Shape      → 배열의 전체 형태
   └─ DataType   → 픽셀 데이터 타입
                ↓
⑥ 프로그램 실행
   └─ main() 함수를 호출하여
      ②~⑤ 처리 과정 진행
```

실행하기 전에 `FIRST_IMAGE` 변수가 설정되어 있는지 다시 확인합니다.

```bash
echo "$FIRST_IMAGE"
```

비어 있다면 12번의 `FIRST_IMAGE=...` 명령을 다시 실행합니다.

이미지 입력 코드를 실행합니다.

```bash
python scripts/01_image_input.py \
  --source "$FIRST_IMAGE"
```

출력 형태는 다음과 비슷합니다.

```text
Image path : data/day01/coco8_pose/coco8-pose/images/train/000000000036.jpg
Width      : 481
Height     : 640
Channels   : 3
Shape      : (640, 481, 3)
DataType   : uint8
```

### 실행 결과 해석

JPG 파일을 OpenCV가 읽으면 Python 안에서는 숫자 배열이 됩니다.

```text
JPG 파일
   ↓
cv2.imread()
   ↓
NumPy 배열
   ↓
하나의 Frame
   ↓
AI 모델의 입력으로 사용 가능
```

---

# 14. 실제 MP4의 기본 정보를 확인합니다

이번에는 실제 동영상 파일을 OpenCV로 읽어 봅니다.

이 코드는 **MP4를 열고 FPS, 전체 Frame 수, 해상도, 재생시간을 확인한 뒤 실제로 끝까지 Frame을 읽을 수 있는지 검사하는 코드**입니다.

### 의사코드로 읽어보기

```text
MP4 파일 경로를 받는다
        ↓
파일이 있는지 확인한다
        ↓
VideoCapture로 영상을 연다
        ↓
FPS · Frame 수 · 해상도를 읽는다
        ↓
Frame을 한 장씩 끝까지 읽는다
        ↓
실제로 읽은 Frame 수를 센다
        ↓
영상 정보를 출력한다
```

**파일: `scripts/02_video_input.py`**

```python
from __future__ import annotations

# ① 필요한 라이브러리 불러오기
import argparse
from pathlib import Path
import cv2


# ② 터미널에서 MP4 영상 경로 입력받기
def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--source",
        required=True,
        help="읽을 MP4 파일 경로",
    )

    return parser.parse_args()


def main() -> None:

    # ③ 영상 파일 경로 확인
    args = parse_args()

    video_path = Path(args.source)

    # 입력한 경로에 파일이 없으면 오류 발생
    if not video_path.exists():
        raise FileNotFoundError(
            f"파일이 없습니다: {video_path}"
        )

    # ④ OpenCV로 MP4 영상 열기
    capture = cv2.VideoCapture(str(video_path))

    # 영상을 열지 못하면 오류 발생
    if not capture.isOpened():
        raise RuntimeError(
            f"영상을 열 수 없습니다: {video_path}"
        )

    # ⑤ 영상의 기본 정보 가져오기
    fps = capture.get(cv2.CAP_PROP_FPS)

    total_frames = int(
        capture.get(cv2.CAP_PROP_FRAME_COUNT)
    )

    width = int(
        capture.get(cv2.CAP_PROP_FRAME_WIDTH)
    )

    height = int(
        capture.get(cv2.CAP_PROP_FRAME_HEIGHT)
    )

    # ⑥ 영상을 Frame 단위로 끝까지 읽기
    frame_number = 0

    while True:
        success, _frame = capture.read()

        # 더 이상 읽을 Frame이 없으면 반복 종료
        if not success:
            break

        # 정상적으로 읽은 Frame 수 증가
        frame_number += 1

    # 영상 사용이 끝났으므로 자원 해제
    capture.release()

    # ⑦ 영상 정보와 재생시간 출력
    print(f"Video       : {video_path}")
    print(f"FPS         : {fps:.2f}")
    print(f"Frames      : {total_frames}")
    print(f"Read Frames : {frame_number}")
    print(f"Resolution  : {width} x {height}")

    # FPS가 정상적으로 확인되면 재생시간 계산
    if fps > 0:
        duration = total_frames / fps
        print(f"Duration    : {duration:.2f} sec")
    else:
        print("Duration    : FPS 정보를 읽지 못해 계산하지 않음")


# 프로그램 실행 시작
if __name__ == "__main__":
    main()
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ argparse → 터미널 입력값 받기
   ├─ Path     → 파일 경로 관리
   └─ cv2      → OpenCV로 영상 처리
                ↓
② 터미널에서 MP4 영상 경로 입력받기
   ├─ --source 옵션 확인
   └─ 입력한 영상 경로 저장
                ↓
③ 영상 파일 경로 확인
   ├─ 입력받은 경로를 Path 객체로 변환
   └─ 파일이 존재하는가?
        ├─ 예 → 다음 단계
        └─ 아니요 → 오류 발생
                ↓
④ OpenCV로 MP4 영상 열기
   ├─ cv2.VideoCapture() 실행
   └─ 영상을 정상적으로 열었는가?
        ├─ 예 → 다음 단계
        └─ 아니요 → 오류 발생
                ↓
⑤ 영상의 기본 정보 가져오기
   ├─ FPS          → 1초당 Frame 수
   ├─ Total Frames → 영상의 전체 Frame 수
   └─ Resolution   → 영상의 가로·세로 크기
                ↓
⑥ 영상을 Frame 단위로 끝까지 읽기
   ├─ 첫 번째 Frame 읽기
   ├─ 정상적으로 읽었는가?
   │    ├─ 예 → Frame 수 +1
   │    │        ↓
   │    │      다음 Frame 읽기 (반복)
   │    └─ 아니요 → 반복 종료
   └─ VideoCapture 자원 해제
                ↓
⑦ 영상 정보와 재생시간 출력
   ├─ Video       → 영상 파일 경로
   ├─ FPS         → 초당 Frame 수
   ├─ Frames      → 영상에 기록된 전체 Frame 수
   ├─ Read Frames → 실제 읽은 Frame 수
   ├─ Resolution  → 영상 해상도
   └─ Duration    → 전체 Frame 수 ÷ FPS
```

실행합니다.

```bash
python scripts/02_video_input.py \
  --source data/day01/video/solution_ci_pose_demo.mp4
```

출력 형태는 다음과 비슷합니다.

```
Video       : data/day01/video/solution_ci_pose_demo.mp4
FPS         : 30.00
Frames      : 60
Read Frames : 60
Resolution  : 640 x 360
Duration    : 2.00 sec
```

### 실행 결과 해석

영상은 하나의 이미지가 아니라 여러 Frame의 연속입니다.

```text
Video
=
Frame 1
Frame 2
Frame 3
Frame 4
...
```

이제 다음 단계에서 이 Frame에 Pose를 적용합니다.

---

# PART 4. YOLO Pose로 실제 사람 이미지를 분석합니다

---

# 15. GPU·CUDA·PyTorch와 Pose 모델을 확인합니다

Pose 추론을 시작하기 전에 실행환경을 한 번에 점검합니다.

먼저 WSL 터미널에서 NVIDIA GPU가 보이는지 확인합니다.

```bash
nvidia-smi
```

GPU 정보가 보이면 WSL에서 NVIDIA Driver를 인식하고 있는 것입니다.

---

### 만약 GPU 정보가 보이지 않는다면?

NVIDIA GPU가 장착되어 있어도 **GPU Driver 또는 WSL과의 연결 상태에 따라 `nvidia-smi`가 정상적으로 실행되지 않을 수 있습니다.**

```
nvidia-smi 정상
    ↓
GPU 사용 가능 여부 확인

nvidia-smi 비정상
    ↓
Driver / WSL GPU 연결 상태 확인 필요
```

교과 10에서는 장치 설정을 다음과 같이 `auto`로 사용합니다.

```
"device": "auto"
```

따라서 프로그램에서는 다음과 같이 처리합니다.

```
CUDA 사용 가능
    ↓
GPU 사용

CUDA 사용 불가능
    ↓
CPU 사용
```

> GPU가 보이지 않는 경우에도 우선 CPU로 실습을 진행할 수 있습니다. GPU가 장착된 PC인데도 인식되지 않는다면 별도로 NVIDIA Driver와 WSL GPU 연결 상태를 점검합니다.

이제 Python에서 PyTorch와 CUDA 상태를 확인하고, 마지막으로 `yolo11n-pose.pt` 모델을 불러옵니다.

---

이제 Python에서 PyTorch와 CUDA 상태를 확인하고, 마지막으로 `yolo11n-pose.pt` 모델을 불러옵니다.

> `yolo11n-pose.pt`가 프로젝트에 없으면 Ultralytics가 첫 실행 시 모델 파일을 내려받을 수 있습니다.  
> 인터넷 사용이 제한된 환경에서 모델 파일을 미리 받은 경우에는 `yolo11n-pose.pt`를 프로젝트 루트에 넣은 뒤 실행합니다.

이 코드는 Python·OpenCV·PyTorch·Ultralytics 버전, CUDA 사용 가능 여부, GPU 이름을 확인하고 Pose 모델이 정상적으로 로드되는지 검사하는 코드입니다.

- Ultralytics란?
	`Ultralytics`는 YOLO 기반의 객체 검출, 자세 추정(Pose), 분류 등의 AI 모델을 쉽게 사용할 수 있도록 제공하는 Python 라이브러리입니다.

### 의사코드로 읽어보기

```text
설정 파일을 읽는다
        ↓
CUDA를 사용할 수 있는지 확인한다
        ↓
GPU 또는 CPU를 선택한다
        ↓
Python과 라이브러리 버전을 출력한다
        ↓
Pose 모델을 불러온다
        ↓
환경 정보를 environment.txt에 저장한다
```

**파일: `scripts/00_check_environment.py`**

```python
from __future__ import annotations

# ① 필요한 라이브러리 불러오기
import platform
import sys
from pathlib import Path

import cv2
import torch
import ultralytics
from ultralytics import YOLO


# ② 프로젝트 경로 설정 및 공통 함수 불러오기
ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.common import (
    ensure_day01_dirs,
    load_config,
    resolve_device,
)


def main() -> None:

    # ③ 필요한 폴더 생성 및 프로젝트 설정 읽기
    ensure_day01_dirs()

    config = load_config()
    device = resolve_device(config["device"])

    # ④ Python·라이브러리·CUDA 실행환경 정보 수집
    lines = [
        f"Python: {platform.python_version()}",
        f"Platform: {platform.platform()}",
        f"OpenCV: {cv2.__version__}",
        f"PyTorch: {torch.__version__}",
        f"Ultralytics: {ultralytics.__version__}",
        f"CUDA available: {torch.cuda.is_available()}",
        f"PyTorch CUDA: {torch.version.cuda}",
    ]

    # GPU 사용 가능 여부와 GPU 이름 확인
    if torch.cuda.is_available():
        lines.append(
            f"GPU: {torch.cuda.get_device_name(0)}"
        )
    else:
        lines.append("GPU: CPU mode")

    # 실제 선택된 실행 장치 기록
    lines.append(f"Selected device: {device}")

    # ⑤ 수집한 환경정보 출력
    print("\n".join(lines))
    print("\nPose model loading...")

    # ⑥ YOLO Pose 모델 로딩 확인
    YOLO(config["pose_model"])

    print("Pose model load: OK")

    # ⑦ 환경 점검 결과를 TXT 파일로 저장
    report_path = (
        ROOT
        / "reports"
        / "day01"
        / "environment.txt"
    )

    # 결과 저장 폴더가 없으면 생성
    report_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    # 수집한 환경정보를 파일에 기록
    report_path.write_text(
        "\n".join(lines),
        encoding="utf-8",
    )

    print(f"\nSaved: {report_path}")


# 프로그램 실행 시작
if __name__ == "__main__":
    main()
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ platform    → 운영체제 정보 확인
   ├─ sys         → Python 모듈 검색 경로 설정
   ├─ Path        → 파일과 폴더 경로 관리
   ├─ cv2         → OpenCV 버전 확인
   ├─ torch       → PyTorch 및 CUDA 확인
   └─ ultralytics → YOLO 버전 및 모델 확인
                ↓
② 프로젝트 경로 설정 및 공통 함수 불러오기
   ├─ ROOT → 프로젝트 최상위 폴더 찾기
   └─ src/common.py에서 공통 함수 불러오기
                ↓
③ 필요한 폴더 생성 및 프로젝트 설정 읽기
   ├─ ensure_day01_dirs()
   │     → 1일차 폴더 생성
   ├─ load_config()
   │     → project.json 설정값 읽기
   └─ resolve_device()
         → GPU 또는 CPU 선택
                ↓
④ 실행환경 정보 수집
   ├─ Python 버전
   ├─ 운영체제 정보
   ├─ OpenCV 버전
   ├─ PyTorch 버전
   ├─ Ultralytics 버전
   └─ CUDA 사용 가능한가?
        ├─ 예 → GPU 이름 확인
        └─ 아니요 → CPU 모드 표시
                ↓
⑤ 수집한 환경정보 터미널에 출력
   ├─ 라이브러리 버전 표시
   ├─ CUDA 및 GPU 정보 표시
   └─ 선택된 실행 장치 표시
                ↓
⑥ YOLO Pose 모델 로딩 확인
   ├─ project.json에서 모델명 가져오기
   ├─ YOLO 모델 불러오기
   └─ 성공하면 "Pose model load: OK"
                ↓
⑦ 환경 점검 결과 저장
   ├─ reports/day01/ 폴더 확인
   ├─ environment.txt 파일 생성
   └─ 수집한 환경정보 저장
```

실행합니다.

```bash
python scripts/00_check_environment.py
```

정상적인 출력 형태는 다음과 같습니다.

```text
Python: ...
OpenCV: ...
PyTorch: ...
Ultralytics: ...
CUDA available: True
GPU: NVIDIA ...
Selected device: 0

Pose model loading...
Pose model load: OK
```

CPU 환경에서는 다음처럼 표시될 수 있습니다.

```text
Python: 3.12.3
Platform: Linux-6.18.33.2-microsoft-standard-WSL2-x86_64-with-glibc2.39
OpenCV: 5.0.0
PyTorch: 2.14.0+cu130
Ultralytics: 8.4.138
CUDA available: True
PyTorch CUDA: 13.0
GPU: NVIDIA GeForce RTX 4080 SUPER
Selected device: 0

Pose model loading...
Downloading https://github.com/ultraly…: 100% ━━━━━━━━━━━━ 6.0MB 15.2MB/s 0.4s
Pose model load: OK

Saved: /home/youjung/ai_vision/subject10_pose_action/reports/day01/environment.txt
```

1일차의 작은 이미지 Pose 실습은 CPU에서도 실행할 수 있습니다.


---

# 16. 첫 YOLO Pose 추론 코드를 작성합니다

앞에서는 OpenCV로 이미지를 읽고, YOLO Pose 모델이 정상적으로 로딩되는지 확인했습니다.

이번에는 **실제 사람 이미지에 YOLO Pose를 적용하여 사람의 위치와 17개 관절을 제대로 찾는지 확인**하겠습니다.

YOLO Pose는 이미지에서 사람을 찾고, 코·어깨·팔꿈치·손목·무릎 등 주요 관절의 위치를 자동으로 추정합니다.

### 이 코드는 무엇을 하나요?

- **Bounding Box(BBox):** 이미지에서 사람의 위치를 사각형으로 표시합니다.
    
- **Keypoint:** 사람의 주요 관절 17개 위치를 점으로 표시하고 좌표와 신뢰도를 확인합니다.
    
- **Skeleton:** 검출한 관절을 선으로 연결하여 사람의 자세를 보여줍니다.
    
- **결과 저장:** 관절이 표시된 이미지와 좌표·신뢰도가 기록된 TXT 보고서를 저장합니다.
    

### 코드의 전체 처리 흐름

```
① COCO8-Pose 이미지 1장 입력
           ↓
② project.json 설정 읽기
           ↓
③ YOLO Pose 모델 불러오기
           ↓
④ 이미지에서 사람과 관절 위치 추정
           ↓
⑤ BBox + Keypoint + Skeleton 표시
           ↓
⑥ 결과 이미지 저장
           ↓
⑦ 17개 Keypoint의
   x · y 좌표와 Confidence 확인
           ↓
⑧ TXT 보고서 저장
```

**파일:** `**scripts/03_pose_image.py**`

이 실습에서 중요한 것은 **AI가 사람의 관절 위치를 얼마나 정확하게 찾았는지 눈으로 확인하는 것**입니다.

결과 이미지에서 관절 점과 Skeleton이 실제 사람의 자세에 맞게 표시되는지 확인하고, TXT 파일에서는 각 관절의 좌표와 신뢰도를 살펴보겠습니다.

```python
from __future__ import annotations

# ① 필요한 라이브러리 불러오기
import argparse
import sys
from pathlib import Path

import cv2
import numpy as np
from ultralytics import YOLO


# ② 프로젝트 경로 설정 및 공통 함수 불러오기
ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.common import (
    load_config,
    resolve_device,
)


# ③ COCO 17개 Keypoint의 이름 정의
# YOLO가 추정한 관절 좌표를 이름과 연결할 때 사용
COCO_KEYPOINT_NAMES = [
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


# ④ 터미널에서 이미지 경로 입력받기
def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--source",
        required=True,
        help="Pose를 적용할 이미지 파일",
    )

    return parser.parse_args()


def main() -> None:

    # ⑤ 이미지 파일 확인 및 공통 설정 읽기
    args = parse_args()

    source = Path(args.source)

    # 이미지 파일이 존재하는지 확인
    if not source.exists():
        raise FileNotFoundError(
            f"파일이 없습니다: {source}"
        )

    # project.json의 설정값 불러오기
    config = load_config()

    # GPU 또는 CPU 선택
    device = resolve_device(config["device"])

    # ⑥ YOLO Pose 모델 불러오기 및 추론 실행
    model = YOLO(config["pose_model"])

    results = model.predict(
        source=str(source),
        imgsz=config["image_size"],
        conf=config["person_confidence"],
        device=device,
        verbose=False,
    )

    # 첫 번째 이미지의 추론 결과 가져오기
    result = results[0]

    # 검출된 사람 수 확인
    person_count = (
        len(result.boxes)
        if result.boxes is not None
        else 0
    )

    print(f"Detected people: {person_count}")

    # ⑦ BBox + Keypoint + Skeleton 결과 이미지 저장
    # YOLO 추론 결과를 원본 이미지 위에 표시
    output_image = result.plot()

    # 결과 이미지 저장 경로 지정
    output_path = (
        ROOT
        / "reports"
        / "day01"
        / f"pose_{source.stem}.jpg"
    )

    # 저장 폴더가 없으면 생성
    output_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    # 이미지 파일로 저장
    saved = cv2.imwrite(
        str(output_path),
        output_image,
    )

    if not saved:
        raise RuntimeError(
            f"결과 이미지를 저장하지 못했습니다: {output_path}"
        )

    print(f"Saved image: {output_path}")

    # ⑧ Keypoint 검출 여부 확인 및 좌표 데이터 가져오기
    if (
        result.keypoints is None
        or result.keypoints.xy is None
        or person_count == 0
    ):
        print("Keypoint가 검출되지 않았습니다.")
        return

    # 관절의 x, y 좌표를 NumPy 배열로 변환
    xy = result.keypoints.xy.cpu().numpy()

    # 관절별 Confidence 가져오기
    confidence = result.keypoints.conf

    if confidence is not None:
        confidence = confidence.cpu().numpy()
    else:
        # Confidence 정보가 없을 때 사용할 대체값
        confidence = np.ones(
            xy.shape[:2],
            dtype=float,
        )

    # ⑨ 사람별 17개 Keypoint의 좌표와 Confidence 출력
    report_lines = []

    # 검출된 사람을 한 명씩 처리
    for person_index in range(len(xy)):
        title = f"Person {person_index + 1}"

        print(f"\n{title}")
        report_lines.append(title)

        # 한 사람의 17개 관절을 순서대로 확인
        for keypoint_index, name in enumerate(
            COCO_KEYPOINT_NAMES
        ):
            # 관절의 x, y 좌표 가져오기
            x, y = xy[
                person_index,
                keypoint_index,
            ]

            # 해당 관절의 Confidence 가져오기
            conf = confidence[
                person_index,
                keypoint_index,
            ]

            # 출력할 문자열 만들기
            line = (
                f"{keypoint_index:02d} "
                f"{name:16s} "
                f"x={x:8.2f} "
                f"y={y:8.2f} "
                f"conf={conf:.3f}"
            )

            print(line)
            report_lines.append(line)

    # ⑩ Keypoint 분석 결과를 TXT 보고서로 저장
    report_path = (
        ROOT
        / "reports"
        / "day01"
        / f"pose_{source.stem}.txt"
    )

    report_path.write_text(
        "\n".join(report_lines),
        encoding="utf-8",
    )

    print(f"\nKeypoint report: {report_path}")


# 프로그램 실행 시작
if __name__ == "__main__":
    main()
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ argparse → 터미널에서 이미지 경로 받기
   ├─ Path     → 파일 및 폴더 경로 관리
   ├─ cv2      → 결과 이미지 저장
   ├─ numpy    → 관절 좌표 데이터 처리
   └─ YOLO     → Pose 모델 실행
               ↓
② 프로젝트 경로 및 공통 함수 준비
   ├─ ROOT → 프로젝트 최상위 폴더
   ├─ load_config() → JSON 설정 읽기
   └─ resolve_device() → GPU/CPU 선택
               ↓
③ COCO 17개 Keypoint 이름 정의
   └─ 코·눈·귀·어깨·팔꿈치·손목·
      골반·무릎·발목 이름 준비
               ↓
④ 터미널에서 이미지 경로 입력받기
   └─ --source 옵션으로 이미지 지정
               ↓
⑤ 입력 이미지 및 설정 확인
   ├─ 이미지 파일이 존재하는가?
   │     ├─ 예 → 계속 진행
   │     └─ 아니요 → 오류 발생
   ├─ project.json 설정 읽기
   └─ GPU 또는 CPU 선택
               ↓
⑥ YOLO Pose 모델 실행
   ├─ yolo11n-pose.pt 불러오기
   ├─ 이미지 입력
   ├─ 사람과 관절 위치 추정
   └─ 검출된 사람 수 확인
               ↓
⑦ Pose 결과 이미지 저장
   ├─ 사람 BBox 표시
   ├─ Keypoint 표시
   ├─ Skeleton 표시
   └─ reports/day01/pose_파일명.jpg 저장
               ↓
⑧ Keypoint 데이터 확인
   ├─ 관절이 검출되었는가?
   │     ├─ 아니요 → 안내 출력 후 종료
   │     └─ 예 → 좌표 가져오기
   ├─ x, y 좌표를 NumPy 배열로 변환
   └─ 관절별 Confidence 가져오기
               ↓
⑨ 사람별 17개 관절 정보 출력
   ├─ Person 1
   │    ├─ nose       → x, y, Confidence
   │    ├─ left_eye   → x, y, Confidence
   │    ├─ ...
   │    └─ right_ankle→ x, y, Confidence
   │
   └─ 사람이 여러 명이면 반복
               ↓
⑩ Keypoint TXT 보고서 저장
   ├─ 관절 이름 기록
   ├─ x, y 좌표 기록
   ├─ Confidence 기록
   └─ reports/day01/pose_파일명.txt 저장
```

---

# 17. COCO8-Pose 실제 이미지에 Pose를 실행합니다

터미널을 새로 열었거나 `FIRST_IMAGE` 값이 사라졌을 수 있으므로 다시 설정합니다.

```bash
FIRST_IMAGE=$(find data/day01/coco8_pose \
-type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" \) \
| sort | head -1)
```

확인합니다.

```bash
echo "$FIRST_IMAGE"
```

Pose 추론을 실행합니다.

```bash
python scripts/03_pose_image.py \
  --source "$FIRST_IMAGE"
```

정상적인 출력 형태는 다음과 같습니다.

```text
Detected people: 1
Saved image: /home/youjung/ai_vision/subject10_pose_action/reports/day01/pose_000000000036.jpg

Person 1
00 nose             x=  248.99 y=  243.43 conf=0.996
01 left_eye         x=  268.71 y=  222.69 conf=0.995
02 right_eye        x=  234.21 y=  228.90 conf=0.949
03 left_ear         x=  311.34 y=  232.05 conf=0.960
04 right_ear        x=  227.83 y=  242.16 conf=0.311
05 left_shoulder    x=  360.34 y=  331.37 conf=0.996
06 right_shoulder   x=  215.92 y=  347.61 conf=0.988
07 left_elbow       x=  393.22 y=  482.73 conf=0.975
08 right_elbow      x=  209.98 y=  489.58 conf=0.888
09 left_wrist       x=  445.63 y=  605.55 conf=0.912
10 right_wrist      x=  225.54 y=  466.30 conf=0.830
11 left_hip         x=  347.34 y=  594.26 conf=0.901
12 right_hip        x=  250.12 y=  601.85 conf=0.853
13 left_knee        x=  367.34 y=  625.26 conf=0.045
14 right_knee       x=  248.36 y=  640.00 conf=0.032
15 left_ankle       x=  354.34 y=  640.00 conf=0.003
16 right_ankle      x=  258.34 y=  640.00 conf=0.002

Keypoint report: /home/youjung/ai_vision/subject10_pose_action/reports/day01/pose_000000000036.txt

real    0m2.670s
user    0m5.132s
sys     0m2.694s
```

결과 파일을 확인합니다.

```bash
ls -lh reports/day01/
```

![[Pasted image 20261104104127.png|286]]

Windows 파일 탐색기로 결과 폴더를 열 수도 있습니다.

```bash
explorer.exe "$(wslpath -w reports/day01)"
```

다음 파일이 만들어집니다.

```text
pose_이미지이름.jpg
pose_이미지이름.txt
```

결과 이미지에서 다음 세 가지를 찾습니다.

```text
Person Bounding Box
+
Keypoint
+
Skeleton
```

---

# 18. Person Bounding Box를 이해합니다

사람 주변에 그려진 사각형을 **Person Bounding Box**라고 합니다.

```text
전체 이미지
        ↓
사람이 어디에 있는가?
        ↓
Bounding Box
```

Bounding Box는 앞으로 다음과 같은 작업에 사용됩니다.

- 화면에서 사람의 위치 확인
- 여러 사람이 있을 때 사람별 영역 구분
- Pose가 어느 사람에게 속하는지 확인
- 이후 Tracking에서 사람을 연결

이미지 상단의 `person 0.90`은 YOLO Pose 모델이 해당 영역에 사람이 있다고 판단한 신뢰도(Confidence)가 0.90, 즉 90%라는 의미입니다.

---

# 19. COCO 17 Keypoint 번호를 확인합니다

YOLO Pose에서 사람의 기본 몸 구조는 17개의 Keypoint로 표현됩니다.

| 번호 | 관절 |
|---:|---|
| 0 | Nose |
| 1 | Left Eye |
| 2 | Right Eye |
| 3 | Left Ear |
| 4 | Right Ear |
| 5 | Left Shoulder |
| 6 | Right Shoulder |
| 7 | Left Elbow |
| 8 | Right Elbow |
| 9 | Left Wrist |
| 10 | Right Wrist |
| 11 | Left Hip |
| 12 | Right Hip |
| 13 | Left Knee |
| 14 | Right Knee |
| 15 | Left Ankle |
| 16 | Right Ankle |

예를 들어 왼팔은 다음 세 점으로 이어집니다.

```text
5 Left Shoulder
      ↓
7 Left Elbow
      ↓
9 Left Wrist
```

오른팔은 다음과 같습니다.

```text
6 Right Shoulder
      ↓
8 Right Elbow
      ↓
10 Right Wrist
```

이 번호는 3일차의 관절각도 계산에서도 그대로 사용합니다.

---

# 20. Keypoint와 Skeleton의 차이를 이해합니다

![[Pasted image 20260903161814.png]]

**Keypoint**는 관절의 위치를 나타내는 점입니다.

**Skeleton**은 여러 Keypoint를 사람의 신체 구조에 맞게 연결한 선입니다.

```text
Keypoint
점 · 점 · 점
        ↓
신체 구조대로 연결
        ↓
Skeleton
어깨 ─ 팔꿈치 ─ 손목
골반 ─ 무릎 ─ 발목
```

Skeleton을 보면 숫자 좌표만 볼 때보다 Pose가 자연스러운지 빠르게 확인할 수 있습니다.

오늘은 결과를 눈으로 확인합니다.

2일차부터는 각 Keypoint의 좌표와 Confidence를 이용하여 왜 어떤 관절은 잘 잡히고 어떤 관절은 불안정한지 숫자로 분석합니다.

---

# PART 5. 실제 동영상에 Pose를 적용합니다

---

# 21. MP4에 Pose를 적용할 때의 구조를 이해합니다

14번에서 실제 MP4가 여러 Frame으로 이루어져 있다는 것을 확인했습니다.

이제 각 Frame에 같은 Pose 모델을 반복 적용합니다.

```text
solution_ci_pose_demo.mp4
        ↓
VideoCapture
        ↓
Frame 1 → YOLO Pose
Frame 2 → YOLO Pose
Frame 3 → YOLO Pose
...
        ↓
Pose 결과 Frame을 다시 영상으로 저장
```

오늘은 `Video → Frame → Pose`까지만 확인합니다.

여러 Frame의 Pose를 시간 순서로 분석하는 Sequence는 이후 수업에서 배웁니다.

---

# 22. 실제 영상의 Frame에 Pose를 적용하는 코드를 작성합니다

이 코드는 MP4를 한 Frame씩 읽고, 각 Frame에 YOLO Pose를 적용한 뒤, BBox와 Skeleton이 그려진 새 MP4를 저장하는 코드입니다.

### 의사코드로 읽어보기

```text
MP4 경로를 받는다
        ↓
영상 파일을 연다
        ↓
첫 Frame을 읽는다
        ↓
결과 영상을 저장할 VideoWriter를 만든다
        ↓
현재 Frame에 YOLO Pose를 적용한다
        ↓
BBox + Skeleton 결과 Frame을 저장한다
        ↓
다음 Frame을 읽는다
        ↓
지정한 Frame 수까지 반복한다
        ↓
결과 MP4를 닫고 저장한다
```

**파일: `scripts/04_pose_video.py`**

```python
from __future__ import annotations

# ① 필요한 라이브러리 불러오기
import argparse
import sys
from pathlib import Path

import cv2
from ultralytics import YOLO


# ② 프로젝트 경로 설정 및 공통 함수 불러오기
ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.common import (
    load_config,
    resolve_device,
)


# ③ 터미널에서 영상 경로와 처리할 Frame 수 입력받기
def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--source",
        required=True,
        help="Pose를 적용할 MP4 파일",
    )

    parser.add_argument(
        "--max-frames",
        type=int,
        default=150,
        help="처리할 최대 Frame 수. 0이면 영상 끝까지 처리",
    )

    return parser.parse_args()


def main() -> None:

    # ④ 영상 파일 확인 및 프로젝트 설정 읽기
    args = parse_args()

    source = Path(args.source)

    # 영상 파일이 존재하는지 확인
    if not source.exists():
        raise FileNotFoundError(
            f"파일이 없습니다: {source}"
        )

    # project.json 설정값 불러오기
    config = load_config()

    # GPU 또는 CPU 선택
    device = resolve_device(config["device"])

    # ⑤ YOLO Pose 모델 불러오기
    model = YOLO(config["pose_model"])

    # ⑥ MP4 영상 열기 및 영상 정보 확인
    capture = cv2.VideoCapture(str(source))

    if not capture.isOpened():
        raise RuntimeError(
            f"영상을 열 수 없습니다: {source}"
        )

    # 원본 영상의 FPS 확인
    fps = capture.get(cv2.CAP_PROP_FPS)

    # FPS를 읽지 못하면 30으로 설정
    output_fps = fps if fps > 0 else 30.0

    # 첫 번째 Frame 읽기
    success, frame = capture.read()

    if not success:
        capture.release()
        raise RuntimeError(
            "영상의 첫 Frame을 읽지 못했습니다."
        )

    # 영상의 높이와 너비 확인
    height, width = frame.shape[:2]

    # ⑦ Pose 결과 영상 저장 준비
    output_path = (
        ROOT
        / "reports"
        / "day01"
        / "pose_demo_output.mp4"
    )

    # 저장 폴더가 없으면 생성
    output_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    # MP4 영상 저장에 사용할 코덱 설정
    fourcc = cv2.VideoWriter_fourcc(*"mp4v")

    # 원본 해상도와 FPS를 기준으로 출력 영상 생성
    writer = cv2.VideoWriter(
        str(output_path),
        fourcc,
        output_fps,
        (width, height),
    )

    if not writer.isOpened():
        capture.release()
        raise RuntimeError(
            "결과 MP4 파일을 생성할 수 없습니다."
        )

    # ⑧ Frame별 YOLO Pose 추론 및 결과 저장
    frame_number = 0

    while success:

        # 처리한 Frame 수 증가
        frame_number += 1

        # 현재 Frame에 YOLO Pose 적용
        results = model.predict(
            source=frame,
            imgsz=config["image_size"],
            conf=config["person_confidence"],
            device=device,
            verbose=False,
        )

        # BBox + Keypoint + Skeleton 표시
        plotted = results[0].plot()

        # Pose가 표시된 Frame을 결과 영상에 저장
        writer.write(plotted)

        # ⑨ 최대 Frame 수 확인 및 다음 Frame 읽기
        if (
            args.max_frames > 0
            and frame_number >= args.max_frames
        ):
            break

        # 다음 Frame 읽기
        success, frame = capture.read()

    # ⑩ 영상 자원 해제 및 최종 결과 출력
    capture.release()
    writer.release()

    print(f"Processed frames: {frame_number}")
    print(f"Saved: {output_path}")


# 프로그램 실행 시작
if __name__ == "__main__":
    main()
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ argparse → 터미널 입력값 받기
   ├─ Path     → 파일과 폴더 경로 관리
   ├─ cv2      → 영상 읽기 및 저장
   └─ YOLO     → Pose 모델 실행
                ↓
② 프로젝트 경로 및 공통 함수 준비
   ├─ ROOT → 프로젝트 최상위 폴더
   ├─ load_config() → JSON 설정 읽기
   └─ resolve_device() → GPU/CPU 선택
                ↓
③ 터미널에서 실행 옵션 입력받기
   ├─ --source → 입력 MP4 영상 경로
   └─ --max-frames → 처리할 최대 Frame 수
      기본값 150, 0이면 영상 끝까지
                ↓
④ 입력 영상 및 설정 확인
   ├─ MP4 파일이 존재하는가?
   │    ├─ 아니요 → 오류 발생
   │    └─ 예 → 다음 단계
   ├─ project.json 설정 읽기
   └─ GPU 또는 CPU 선택
                ↓
⑤ YOLO Pose 모델 불러오기
   └─ yolo11n-pose.pt 로딩
                ↓
⑥ MP4 영상 열기
   ├─ VideoCapture로 영상 열기
   ├─ FPS 확인
   ├─ 첫 번째 Frame 읽기
   └─ 영상의 너비·높이 확인
                ↓
⑦ 결과 MP4 영상 저장 준비
   ├─ reports/day01/ 경로 지정
   ├─ MP4 코덱 설정
   └─ VideoWriter 생성
                ↓
⑧ 현재 Frame에 YOLO Pose 적용
   ├─ 사람 검출
   ├─ 17개 Keypoint 추정
   ├─ BBox + Skeleton 표시
   └─ 결과 Frame을 MP4에 저장
                ↓
⑨ 처리한 Frame 수 확인
   ├─ 최대 Frame 수에 도달했는가?
   │    ├─ 예 → 반복 종료
   │    └─ 아니요 → 다음 Frame 읽기
   │                       ↓
   │                  ⑧ 반복 실행
   └─ 영상이 끝나도 반복 종료
                ↓
⑩ 영상 처리 마무리
   ├─ VideoCapture 해제
   ├─ VideoWriter 해제
   ├─ 처리한 Frame 수 출력
   └─ 결과 MP4 저장 경로 출력
```

---

# 23. 실제 MP4 Pose를 실행하고 결과를 확인합니다

먼저 150 Frame만 처리하여 Pipeline이 정상인지 확인합니다.

```bash
python scripts/04_pose_video.py \
  --source data/day01/video/solution_ci_pose_demo.mp4 \
  --max-frames 150
```
앞쪽 150개 Frame까지만 Pose 처리하겠다는 뜻


정상이라면 다음과 비슷하게 출력됩니다.

```text
Processed frames: 150
Saved: .../reports/day01/pose_demo_output.mp4
```

파일이 실제로 만들어졌는지 확인합니다.

```bash
ls -lh reports/day01/pose_demo_output.mp4
```

파일 탐색기로 결과를 엽니다.

```bash
explorer.exe "$(wslpath -w reports/day01)"
```

![[Pasted image 20261104104728.png]]

`pose_demo_output.mp4`를 재생하여 다음을 확인합니다.

```text
사람이 움직인다
        ↓
BBox도 사람을 따라간다
        ↓
Keypoint와 Skeleton도 Frame마다 변한다
```

영상 전체를 처리하고 싶다면 `--max-frames 0`을 사용합니다.

```bash
python scripts/04_pose_video.py \
  --source data/day01/video/solution_ci_pose_demo.mp4 \
  --max-frames 0
```

Frame 수 제한 없이 영상의 마지막 Frame까지 전부 Pose 처리하겠다는 뜻

---

# 24. 이미지 Pose와 영상 Pose의 공통점을 정리합니다

JPG는 Frame이 한 장입니다.

```text
JPG
 ↓
Frame 1개
 ↓
YOLO Pose
```

MP4는 Frame이 여러 장입니다.

```text
MP4
 ↓
Frame 반복
 ↓
각 Frame에 YOLO Pose
```

공통 부분은 같습니다.

```text
Frame
 ↓
YOLO Pose
 ↓
Person Bounding Box
 ↓
COCO 17 Keypoint
 ↓
Skeleton
```

---

# PART 6. IP Webcam으로 자신의 실제 데이터를 입력합니다

---

# 25. IP Webcam을 사용하는 이유와 연결 조건을 이해합니다

WSL에서 USB·내장 웹캠이 바로 보이지 않는 환경에서는 스마트폰 카메라를 **네트워크 영상 스트림**으로 사용할 수 있습니다.

```text
스마트폰 카메라
        ↓
IP Webcam 앱
        ↓
HTTP 영상 주소
        ↓
Wi-Fi
        ↓
WSL OpenCV
        ↓
Frame
```

오늘 필요한 조건은 두 가지입니다.

```text
스마트폰
  +
PC
  ↓
같은 Wi-Fi에 연결
```

그리고 IP Webcam 앱이 영상 서버를 실행하고 있어야 합니다.

> 학교·기관 Wi-Fi에서 기기 간 통신을 막는 경우에는 같은 Wi-Fi에 연결되어 있어도 접속되지 않을 수 있습니다.  
> 이 경우 다른 허용된 네트워크 또는 모바일 핫스팟 환경을 사용합니다.

---

# 26. 스마트폰에서 IP Webcam을 실행하고 주소를 확인합니다

스마트폰에서 **IP Webcam** 앱을 실행합니다.

앱에서 카메라 서버를 시작합니다.

예를 들어 다음과 같은 기본 주소가 표시될 수 있습니다.

```text
http://192.168.0.10:8080
```

IP Webcam의 영상 스트림 주소는 보통 기본 주소 뒤에 `/video`를 붙입니다.

```text
http://192.168.0.10:8080/video
```

주소는 학생마다 다릅니다.

먼저 Windows 웹브라우저에서 스마트폰에 표시된 기본 주소를 열어 봅니다.

```text
http://스마트폰_IP:포트
```

브라우저에서 IP Webcam 화면이 보이면 PC와 스마트폰 사이의 네트워크 연결이 된 것입니다.

그 다음 실제 영상 스트림 주소를 사용합니다.

```text
http://스마트폰_IP:포트/video
```

---

# 27. OpenCV로 IP Webcam Frame을 읽고 저장하는 코드를 작성합니다

이 코드는 다음 두 가지 방법으로 사용할 수 있습니다.

```text
방법 1
--one-frame 사용
→ Frame 한 장만 받아 저장
→ 연결 확인에 사용

방법 2
--one-frame 없이 실행
→ 실시간 Preview
→ 1, 2, 3 키로 자세별 이미지 저장
```

저장 키는 다음과 같습니다.

```text
1 → front.jpg
2 → arms_up.jpg
3 → far.jpg
Q → 종료
```

### 의사코드로 읽어보기

```text
IP Webcam URL을 받는다
        ↓
VideoCapture로 네트워크 영상을 연다
        ↓
첫 Frame을 읽는다
        ↓
연결 성공 여부와 해상도를 확인한다
        ↓
--one-frame이면
connection_test.jpg 저장 후 종료
        ↓
일반 실행이면 Preview를 표시한다
        ↓
1 → 정면 저장
2 → 양팔 들기 저장
3 → 먼 거리 저장
Q → 종료
```

**파일: `scripts/05_ip_webcam_capture.py`**

```python
from __future__ import annotations

# ① 필요한 라이브러리 불러오기
import argparse
from pathlib import Path
import cv2


# ② 프로젝트 경로 및 이미지 저장 위치 설정
ROOT = Path(__file__).resolve().parents[1]

SAVE_DIR = ROOT / "data" / "day01" / "ip_webcam"


# ③ 터미널에서 IP Webcam 주소와 실행 옵션 입력받기
def parse_args():
    parser = argparse.ArgumentParser()

    # 스마트폰 카메라의 영상 스트림 주소
    parser.add_argument(
        "--url",
        required=True,
        help="IP Webcam video URL",
    )

    # 첫 Frame 한 장만 저장하는 연결 테스트 옵션
    parser.add_argument(
        "--one-frame",
        action="store_true",
        help="첫 Frame 한 장만 저장하고 종료",
    )

    return parser.parse_args()


# ④ 촬영한 Frame을 JPG 이미지로 저장하는 공통 함수
def save_image(
    frame,
    filename: str,
) -> Path:

    # 저장 폴더가 없으면 생성
    SAVE_DIR.mkdir(
        parents=True,
        exist_ok=True,
    )

    # 저장할 이미지 경로 만들기
    output_path = SAVE_DIR / filename

    # Frame을 JPG 파일로 저장
    saved = cv2.imwrite(
        str(output_path),
        frame,
    )

    # 이미지 저장에 실패하면 오류 발생
    if not saved:
        raise RuntimeError(
            f"이미지를 저장하지 못했습니다: {output_path}"
        )

    print(f"Saved: {output_path}")

    return output_path


def main() -> None:

    # ⑤ IP Webcam 연결 및 Frame 수신
    args = parse_args()

    # 스마트폰 IP Webcam 영상 연결
    capture = cv2.VideoCapture(args.url)

    if not capture.isOpened():
        raise RuntimeError(
            f"IP Webcam에 연결할 수 없습니다: {args.url}"
        )

    # 영상 버퍼 크기 설정 시도
    capture.set(
        cv2.CAP_PROP_BUFFERSIZE,
        1,
    )

    # 첫 번째 Frame 읽기
    success, frame = capture.read()

    if not success:
        capture.release()
        raise RuntimeError(
            "IP Webcam의 첫 Frame을 읽지 못했습니다."
        )

    # 수신한 영상의 해상도 확인
    height, width = frame.shape[:2]

    print(
        f"Connected: {width} x {height}"
    )

    # ⑥ 첫 Frame 한 장만 저장하는 연결 테스트
    if args.one_frame:
        save_image(
            frame,
            "connection_test.jpg",
        )

        capture.release()
        return

    # ⑦ 촬영할 자세와 키보드 조작 방법 안내
    print("1 : 정면 자세 저장")
    print("2 : 양팔 들기 저장")
    print("3 : 먼 거리 자세 저장")
    print("Q : 종료")

    # ⑧ 실시간 카메라 영상 화면에 표시
    while True:

        # 원본 Frame을 복사하여 화면 표시용으로 사용
        display_frame = frame.copy()

        # 화면 상단에 키보드 사용방법 표시
        cv2.putText(
            display_frame,
            "1: Front  2: Arms Up  3: Far  Q: Quit",
            (20, 40),
            cv2.FONT_HERSHEY_SIMPLEX,
            0.8,
            (255, 255, 255),
            2,
        )

        # 실시간 영상 화면 출력
        cv2.imshow(
            "Subject10 IP Webcam",
            display_frame,
        )

        # 키보드 입력 확인
        key = cv2.waitKey(1) & 0xFF

        # ⑨ 키보드 입력에 따라 이미지 저장 또는 종료
        if key == ord("q"):
            break

        # 1번: 정면 자세 저장
        if key == ord("1"):
            save_image(
                frame,
                "front.jpg",
            )

        # 2번: 양팔 들기 자세 저장
        if key == ord("2"):
            save_image(
                frame,
                "arms_up.jpg",
            )

        # 3번: 먼 거리 자세 저장
        if key == ord("3"):
            save_image(
                frame,
                "far.jpg",
            )

        # 다음 Frame 읽기
        success, frame = capture.read()

        if not success:
            print(
                "다음 Frame을 읽지 못했습니다."
            )
            break

    # ⑩ 카메라 연결 해제 및 화면 종료
    capture.release()
    cv2.destroyAllWindows()


# 프로그램 실행 시작
if __name__ == "__main__":
    main()
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ argparse → 터미널에서 실행 옵션 받기
   ├─ Path     → 파일과 폴더 경로 관리
   └─ cv2      → 카메라 영상 읽기·표시·저장
                ↓
② 프로젝트와 이미지 저장 경로 설정
   ├─ ROOT → 프로젝트 최상위 폴더
   └─ SAVE_DIR → data/day01/ip_webcam/
                ↓
③ 터미널 실행 옵션 준비
   ├─ --url → 스마트폰 IP Webcam 주소
   └─ --one-frame → 첫 Frame만 저장하는 옵션
                ↓
④ 이미지 저장 함수 정의
   ├─ 저장할 폴더 확인
   ├─ Frame을 JPG로 저장
   └─ 저장 성공 여부 확인
                ↓
⑤ 스마트폰 IP Webcam 연결
   ├─ 입력받은 URL로 카메라 연결
   ├─ 연결에 성공했는가?
   │    ├─ 아니요 → 오류 발생
   │    └─ 예 → 첫 Frame 읽기
   └─ 영상 해상도 확인
                ↓
⑥ 연결 테스트 옵션 확인
   ├─ --one-frame 사용했는가?
   │    ├─ 예 → connection_test.jpg 저장 후 종료
   │    └─ 아니요 → 실시간 촬영 진행
                ↓
⑦ 키보드 사용방법 안내
   ├─ 1 → 정면 자세
   ├─ 2 → 양팔 들기
   ├─ 3 → 먼 거리
   └─ Q → 종료
                ↓
⑧ 실시간 카메라 영상 표시
   ├─ 현재 Frame 복사
   ├─ 화면에 키보드 안내문 표시
   └─ OpenCV 창에 Frame 출력
                ↓
⑨ 키보드 입력에 따라 이미지 저장
   ├─ 1 → front.jpg 저장
   ├─ 2 → arms_up.jpg 저장
   ├─ 3 → far.jpg 저장
   ├─ Q → 반복 종료
   └─ 다음 Frame 읽기
           ↓
       ⑧~⑨ 반복
                ↓
⑩ 촬영 종료 및 자원 해제
   ├─ 카메라 연결 해제
   └─ OpenCV 화면 닫기
```

---

# 28. IP Webcam 연결을 단계별로 확인하고 세 이미지를 저장합니다

처음부터 Preview를 실행하지 않고 **Frame 한 장만 받아보는 테스트**부터 합니다.

먼저 ip webcam 서버를 실행합니다.

그리고 자신의 폰의 ip 주소와 같이 아래 주소를 변경하여 브라우저에서 엽니다.

```text
http://192.168.40.10:8080/video
```

자신의 주소로 바꾸어 실행합니다.

```bash
python scripts/05_ip_webcam_capture.py \
  --url "http://192.168.40.125:8080/video" \
  --one-frame
```

정상이라면 다음과 같이 표시됩니다.

```text
Connected: 640 x 480
Saved: /home/youjung/ai_vision/subject10_pose_action/data/day01/ip_webcam/connection_test.jpg
```

파일을 확인합니다.

```bash
ls -lh data/day01/ip_webcam/connection_test.jpg
```

첫 Frame 저장이 성공했다면 Preview를 실행합니다.

```bash
python scripts/05_ip_webcam_capture.py \
  --url "http://192.168.40.125:8080/video"
```

카메라 앞에서 다음 순서대로 자세를 취하고 키를 누릅니다.

```text
정면 자세
→ 1

양팔 들기
→ 2

카메라에서 조금 멀리 서기
→ 3

종료
→ Q
```

저장 결과를 확인합니다.

```bash
ls -lh data/day01/ip_webcam/
```

다음 파일이 있어야 합니다.

```text
connection_test.jpg
front.jpg
arms_up.jpg
far.jpg
```

## OpenCV Preview 창이 보이지 않는 경우

WSLg의 GUI 창을 사용할 수 없는 환경이라면 `--one-frame` 모드만으로도 카메라 입력 확인이 가능합니다.

다음처럼 한 Frame을 저장한 뒤 파일 탐색기에서 이미지를 확인합니다.

```bash
python scripts/05_ip_webcam_capture.py \
  --url "http://192.168.40.125:8080/video"
```

```bash
explorer.exe "$(wslpath -w data/day01/ip_webcam)"
```

## 연결이 안 될 때 확인할 것

```text
1. 스마트폰과 PC가 같은 네트워크인가?
2. IP Webcam 서버가 실행 중인가?
3. Windows 브라우저에서 스마트폰 기본 주소가 열리는가?
4. 코드에 입력한 IP와 포트가 정확한가?
5. 영상 주소 끝의 /video가 맞는가?
6. 학교·기관 Wi-Fi가 기기 간 통신을 차단하고 있지 않은가?
```

---

# 29. IP Webcam으로 저장한 실제 이미지에 Pose를 적용합니다

이제 공개 데이터가 아니라 자신이 직접 촬영한 이미지에 같은 Pose 코드를 사용합니다.

먼저 세 파일이 있는지 확인합니다.

```bash
ls -lh \
data/day01/ip_webcam/front.jpg \
data/day01/ip_webcam/arms_up.jpg \
data/day01/ip_webcam/far.jpg
```

정면 이미지에 Pose를 적용합니다.

```bash
python scripts/03_pose_image.py \
  --source data/day01/ip_webcam/front.jpg
```

양팔 들기 이미지에 Pose를 적용합니다.

```bash
python scripts/03_pose_image.py \
  --source data/day01/ip_webcam/arms_up.jpg
```

먼 거리 이미지에 Pose를 적용합니다.

```bash
python scripts/03_pose_image.py \
  --source data/day01/ip_webcam/far.jpg
```

결과를 확인합니다.

```bash
ls -lh reports/day01/
```

이제 다음 흐름을 실제로 완성했습니다.

```text
스마트폰 카메라
        ↓
IP Webcam
        ↓
실제 Frame
        ↓
JPG 저장
        ↓
YOLO Pose
        ↓
Person Bounding Box
        ↓
COCO 17 Keypoint
        ↓
Skeleton
```

1일차에서는 IP Webcam 영상 자체에서 Action Recognition을 수행하지 않습니다.

오늘은 실시간 카메라에서 Frame을 가져올 수 있고, 그 Frame에 같은 Pose Pipeline을 적용할 수 있다는 것까지 확인합니다.

---

# PART 7. 결과를 비교하고 스스로 문제를 해결합니다

---

# 30. 촬영 조건을 바꾸었을 때 Pose 결과를 비교합니다

앞에서 같은 사람을 서로 다른 조건으로 촬영하고 YOLO Pose를 적용했습니다.

```
front.jpg
→ 정면 자세

arms_up.jpg
→ 양팔을 든 자세

far.jpg
→ 카메라에서 멀리 떨어진 자세
```

이번에는 세 결과를 함께 보면서 **자세와 촬영 조건에 따라 Pose 결과가 어떻게 달라지는지 확인**합니다.

## 30.1 무엇을 보면 되나요?

처음에는 복잡하게 분석하지 않고 다음 세 가지만 봅니다.

```
① BBox
→ 사람의 크기와 위치가 어떻게 달라졌는가?

② Keypoint
→ 어깨·팔꿈치·손목의 위치가 어떻게 달라졌는가?

③ Skeleton
→ 선으로 연결된 자세가 실제 사람의 자세와 비슷한가?
```

### 정면과 양팔 들기를 비교합니다

정면에서는 손목이 몸 아래쪽에 있습니다.

```
정면 자세

어깨
 ↓
팔꿈치
 ↓
손목
```

양팔을 들면 손목과 팔꿈치가 위쪽으로 이동합니다.

```
양팔 들기

손목       손목
 ↑           ↑
팔꿈치     팔꿈치
   ↖       ↗
      어깨
```

즉,

```
사람의 자세 변화
        ↓
Keypoint 위치 변화
        ↓
Skeleton 모양 변화
```

가 나타납니다.

### 정면과 먼 거리를 비교합니다

사람이 카메라에서 멀어지면 영상에서 사람이 작게 보입니다.

따라서 사람을 둘러싸는 **BBox도 작아집니다.**

```
가까이 촬영
→ 사람이 크게 보임
→ BBox도 큼

멀리 촬영
→ 사람이 작게 보임
→ BBox도 작음
```

사람이 너무 작게 촬영되면 손목이나 발목처럼 작은 관절은 Pose 추정이 불안정해질 수도 있습니다.

## 30.2 실제 결과를 간단히 정리합니다

이번 결과를 예로 들면 다음과 같이 관찰할 수 있습니다.

|비교 항목|정면|양팔 들기|먼 거리|
|---|---|---|---|
|사람 검출|O|O|O|
|BBox|기본 크기|팔을 포함해 형태 변화|더 작아짐|
|손목 위치|몸 아래쪽|머리 위쪽|몸 아래쪽|
|Skeleton|정면 자세|양팔을 든 자세|작은 사람의 자세|
|확인할 점|기본 Pose|자세 변화|촬영 거리 변화|

화면에 표시되는

```
person 0.92
person 0.86
```

과 같은 값은 사람을 검출한 Confidence입니다.

> `0.92`를 “Pose가 92% 정확하다”라고 해석하지 않습니다. 
> 각 관절의 Confidence는 2일차부터 자세히 분석합니다.

### 오늘 확인할 핵심

```
자세가 달라지면
→ Keypoint와 Skeleton이 달라진다.

촬영 거리가 달라지면
→ 사람의 크기와 BBox가 달라진다.
```

> 1일차에서는 결과를 눈으로 비교하는 것까지만 합니다. 2일차부터는 Keypoint의 `(x, y)` 좌표와 Confidence를 이용하여 숫자로 분석합니다.

---

# 31. Team Mini Challenge — 촬영 조건에 따른 Pose Pipeline을 직접 검증합니다

지금까지는 준비된 순서에 따라 Pose를 실행했습니다.

이번 Mini Challenge에서는 팀이 직접 **어떤 조건에서 Pose 결과가 달라지는지 실험을 설계하고 확인**합니다.

새로운 AI 모델을 만들거나 학습하는 실습은 아닙니다.

오늘 배운 내용을 이용하여 다음 질문에 답하는 것이 목표입니다.

> **같은 YOLO Pose 모델이라도 입력 이미지와 촬영 조건이 달라지면 BBox, Keypoint, Skeleton 결과는 어떻게 달라질까?**

---

## 31.1 Challenge 목표

팀별로 다음 전체 과정을 직접 수행합니다.

```
실험 조건 결정
        ↓
실제 데이터 준비
        ↓
JPG / MP4 / IP Webcam 입력
        ↓
Frame
        ↓
YOLO Pose
        ↓
Person Bounding Box
        ↓
COCO 17 Keypoint
        ↓
Skeleton
        ↓
결과 비교
        ↓
잘 된 결과 / 불안정한 결과 찾기
        ↓
원인 추정
        ↓
팀 결과 정리
```

핵심은 단순히 Pose 프로그램을 실행하는 것이 아닙니다.

```
왜 이런 결과가 나왔는가?

어떤 조건에서는 잘 되었는가?

어떤 조건에서는 결과가 불안정했는가?
```

를 팀이 함께 생각해 보는 것이 중요합니다.

---

# 32. Challenge 요구사항을 확인합니다

팀별로 아래 요구사항을 모두 수행합니다.

## 요구사항 1 — 사용하지 않은 COCO8-Pose 이미지 2장 이상 분석

첫 번째 실습에서 사용하지 않은 COCO8-Pose 이미지 중 **2장 이상**을 선택합니다.

예를 들어 두 번째 이미지를 가져올 수 있습니다.

```bash
SECOND_IMAGE=$(find data/day01/coco8_pose \
-type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" \) \
| sort | sed -n '2p')
```

확인합니다.

```bash
echo "$SECOND_IMAGE"
```

Pose를 실행합니다.

```bash
python scripts/03_pose_image.py \
  --source "$SECOND_IMAGE"
```

다른 이미지도 같은 방법으로 선택하여 실행합니다.

팀은 결과를 보면서 다음을 확인합니다.

```
사람은 몇 명 검출되었는가?

BBox가 사람을 제대로 감싸고 있는가?

Keypoint가 사람의 관절 위치에 표시되었는가?

Skeleton이 실제 사람의 자세와 비슷한가?

잘못 잡히거나 이상하게 보이는 관절은 없는가?
```

---

## 요구사항 2 — IP Webcam으로 새로운 실험 조건을 직접 만듭니다

앞에서 촬영했던

```
정면
양팔 들기
먼 거리
```

를 그대로 반복하는 것이 아닙니다.

팀이 **새로운 촬영 조건을 최소 3가지 직접 결정**합니다.

예를 들어 다음과 같은 조건을 생각할 수 있습니다.

```
조건 A
몸을 옆으로 돌리기

조건 B
한쪽 팔만 올리기

조건 C
한쪽 팔을 몸 뒤로 숨기기

조건 D
의자에 앉기

조건 E
카메라 가까이 이동하기

조건 F
화면 가장자리에서 촬영하기
```

여기서 최소 3개를 선택하거나 팀에서 새로운 조건을 만들어도 됩니다.

단, 촬영 전에 먼저 **예상 결과를 기록**합니다.

예:

|촬영 조건|실행 전 예상|
|---|---|
|한쪽 팔 올리기|올라간 팔의 손목과 팔꿈치 위치가 위로 이동할 것 같다|
|옆으로 서기|좌우 관절 일부가 겹쳐 보일 수 있다|
|몸 일부 가리기|가려진 관절의 Skeleton이 불안정할 수 있다|

이 단계가 중요합니다.

```
먼저 예상
   ↓
실제 촬영
   ↓
Pose 실행
   ↓
예상과 실제 결과 비교
```

를 해야 단순 따라하기가 아니라 **실험**이 됩니다.

---

## 요구사항 3 — 촬영한 이미지에 Pose를 적용합니다

촬영한 이미지 각각에 오늘 사용한 프로그램을 적용합니다.

예:

```bash
python scripts/03_pose_image.py \
  --source data/day01/ip_webcam/team_pose01.jpg
```

```bash
python scripts/03_pose_image.py \
  --source data/day01/ip_webcam/team_pose02.jpg
```

```bash
python scripts/03_pose_image.py \
  --source data/day01/ip_webcam/team_pose03.jpg
```

파일명은 팀에서 알아보기 쉽게 정합니다.

예:

```
side.jpg
one_arm_up.jpg
sitting.jpg
```

중요한 것은 파일 이름보다 **어떤 조건으로 촬영한 데이터인지 알 수 있게 만드는 것**입니다.

---

# 33. 결과를 비교하고 가장 잘 된 사례와 어려운 사례를 찾습니다

이제 결과를 한 장씩 보는 것이 아니라 **서로 비교**합니다.

팀별로 다음 표를 작성합니다.

|입력|조건|사람 검출|BBox|Keypoint|Skeleton|팀의 판단|
|---|---|---|---|---|---|---|
|COCO 이미지 1|기존 데이터|O|정상|정상|자연스러움|잘 됨|
|COCO 이미지 2|기존 데이터||||||
|IP Webcam 1|한쪽 팔 올림||||||
|IP Webcam 2|옆으로 서기||||||
|IP Webcam 3|앉은 자세||||||

정확한 숫자를 계산할 필요는 없습니다.

1일차에서는 다음 세 가지를 중심으로 눈으로 판단합니다.

```
BBox
→ 사람을 제대로 찾았는가?

Keypoint
→ 관절 위치가 실제 사람과 비슷한가?

Skeleton
→ 연결된 사람의 자세가 자연스러운가?
```

---

## 반드시 하나의 실패 또는 불안정 사례를 찾습니다

모든 결과가 잘 나왔다면 촬영 조건을 조금 더 어렵게 바꾸어 봅니다.

예:

```
조금 더 멀리 이동

몸을 더 옆으로 돌림

팔 일부를 몸 뒤로 숨김

화면 가장자리로 이동
```

목적은 일부러 모델을 망가뜨리는 것이 아니라,

> **Pose 모델도 촬영 조건에 따라 결과가 달라질 수 있다는 사실을 직접 확인하는 것**

입니다.

불안정한 사례를 하나 골라 다음 형식으로 기록합니다.

```
실험 조건:
옆으로 서서 한쪽 팔이 몸에 가려짐

관찰한 결과:
한쪽 팔의 Skeleton이 실제 팔 위치와 다르게 나타남

우리 팀이 생각한 원인:
카메라에서 팔 전체가 잘 보이지 않았기 때문이라고 생각함
```

여기서는 아직 정답을 맞히는 것이 목적이 아닙니다.

**관찰 → 원인 추정**을 해보는 것이 목적입니다.

---

# 34. MP4 입력에서도 같은 Pipeline인지 확인합니다

앞에서는 이미지와 IP Webcam을 중심으로 실험했습니다.

이번에는 오늘 사용한 실제 MP4 영상도 다시 확인합니다.

빠른 확인을 위해 일부 Frame만 처리합니다.

```bash
python scripts/04_pose_video.py \
  --source data/day01/video/solution_ci_pose_demo.mp4 \
  --max-frames 150
```

팀에서 다음 질문에 답합니다.

```
JPG에서는 Frame을 몇 개 처리했는가?

MP4에서는 왜 여러 Frame을 반복해서 처리하는가?

IP Webcam에서는 Frame이 어디에서 계속 들어오는가?

세 입력에서 Frame을 얻은 뒤 사용하는 Pose 모델은 달라지는가?
```

팀의 답을 다음 흐름으로 완성합니다.

```
JPG
→ 이미지 1장
→ Frame 1개
         ┐
MP4      │
→ 여러 Frame ──→ YOLO Pose
         │           ↓
IP Webcam│        Person BBox
→ 실시간 Frame    ↓
         ┘       Keypoint
                   ↓
                Skeleton
```

---

# 35. 팀이 직접 Pose Pipeline을 설명합니다

이제 교재를 보지 않고 다음 빈칸을 팀원끼리 채워 봅니다.

```
JPG / MP4 / IP Webcam
          ↓
       ________
          ↓
       ________
          ↓
 Person Bounding Box
          ↓
       ________
          ↓
       ________
```

정답은 다음과 같습니다.

```
JPG / MP4 / IP Webcam
          ↓
        Frame
          ↓
      YOLO Pose
          ↓
 Person Bounding Box
          ↓
 COCO 17 Keypoint
          ↓
       Skeleton
```

하지만 정답을 외우는 것보다 **각 단계가 무엇을 의미하는지 설명할 수 있어야 합니다.**

팀원 한 명이 다음 질문에 답해 봅니다.

```
JPG와 MP4는 무엇이 다른가?

MP4와 IP Webcam은 무엇이 다른가?

왜 세 입력 모두 결국 Frame으로 처리되는가?

YOLO Pose는 Frame에서 무엇을 찾는가?

BBox와 Keypoint는 무엇이 다른가?

Keypoint와 Skeleton은 무엇이 다른가?
```

한 사람이 전부 설명하지 말고 팀원이 질문을 나누어 설명합니다.

---

# 36. 최종 결과물을 정리합니다

Mini Challenge가 끝나면 팀별로 다음 결과를 남깁니다.

```
① COCO8-Pose 새로운 이미지 결과 2개 이상

② 직접 촬영한 새로운 조건 이미지 3개 이상

③ 각 이미지의 Pose 결과

④ 조건별 비교표 1개

⑤ 가장 잘 나온 결과 1개

⑥ 가장 불안정한 결과 1개

⑦ 불안정한 이유에 대한 팀의 추정

⑧ JPG / MP4 / IP Webcam 공통 Pipeline 설명
```

폴더는 예를 들어 다음처럼 정리할 수 있습니다.

```
reports/day01/challenge/
│
├── coco_result01.jpg
├── coco_result02.jpg
│
├── side_result.jpg
├── one_arm_up_result.jpg
├── sitting_result.jpg
│
└── challenge_notes.md
```

`challenge_notes.md`에는 복잡한 보고서를 작성할 필요는 없습니다.

다음 정도만 기록합니다.

```
# Day01 Team Mini Challenge

## 우리가 선택한 촬영 조건

- 한쪽 팔 올리기
- 옆으로 서기
- 앉기

## 가장 잘 나온 결과

- 파일:
- 관찰:

## 가장 불안정한 결과

- 파일:
- 관찰:
- 예상 원인:

## 오늘 확인한 Pipeline

JPG / MP4 / IP Webcam
→ Frame
→ YOLO Pose
→ Person Bounding Box
→ COCO 17 Keypoint
→ Skeleton
```

---

# 37. Challenge 완료 조건을 확인합니다

다음 항목을 팀원이 함께 확인합니다.

- [ ]  사용하지 않은 COCO8-Pose 이미지 2장 이상을 분석했다.
- [ ]  팀에서 새로운 촬영 조건을 3개 이상 정했다.
- [ ]  촬영하기 전에 결과를 예상해 보았다.
- [ ]  직접 촬영한 이미지에 Pose를 적용했다.
- [ ]  BBox, Keypoint, Skeleton을 비교했다.
- [ ]  잘 나온 결과를 1개 선택했다.
- [ ]  불안정한 결과를 1개 이상 찾아보았다.
- [ ]  불안정한 이유를 팀에서 논의했다.
- [ ]  JPG, MP4, IP Webcam의 차이를 설명할 수 있다.
- [ ]  세 입력이 결국 같은 Pose Pipeline으로 연결된다는 것을 설명할 수 있다.
- [ ]  팀 결과를 `challenge_notes.md`에 정리했다.

### Mini Challenge의 핵심

```
단순히 프로그램 실행
        X

조건을 직접 결정
        ↓
결과를 예상
        ↓
실제 데이터 촬영
        ↓
Pose 실행
        ↓
결과 비교
        ↓
왜 달라졌는지 팀에서 생각
        O
```
---

# PART 8. 오늘 만든 결과를 정리하고 버전관리합니다

---

# 38. `.gitignore`와 README를 작성합니다

## 38.1 `.gitignore`를 먼저 작성합니다

가상환경, 원본 데이터, 영상, 모델 파일은 Git에 올리지 않습니다.

**파일: `.gitignore`**

```gitignore
# Python virtual environment
.venv/

# Python cache
__pycache__/
*.pyc
*.pyo

# IDE
.vscode/
.idea/

# Environment files
.env

# Raw data
data/

# Images and videos
*.jpg
*.jpeg
*.png
*.mp4
*.avi
*.mov

# AI models
*.pt
*.pth
*.onnx
*.engine

# Training outputs
runs/
weights/
checkpoints/
artifacts/

# OS files
.DS_Store
Thumbs.db
```

`uv.lock`은 설치된 라이브러리 버전을 재현하는 데 도움이 되므로 Git에 포함합니다.

## 38.2 README를 작성합니다

README는 오늘 무엇을 만들었는지 다른 사람이 빠르게 이해할 수 있도록 정리하는 문서입니다.

**파일: `README.md`**

````markdown
# Subject 10 - Pose Estimation & Action Analysis

교과 10 자세추정 이론·실습 프로젝트입니다.

## Day 01 - Pose 전체 구조·개발환경·실제 데이터 입력

### 사용한 입력

- COCO8-Pose
- Ultralytics Pose Demo MP4
- 스마트폰 IP Webcam 캡처 이미지

### 학습한 내용

- Pose Estimation과 Action Recognition의 차이
- WSL 프로젝트와 Python 가상환경 구성
- OpenCV / PyTorch / Ultralytics 확인
- GPU / CUDA 확인
- COCO8-Pose 이미지 입력
- YOLO Pose 추론
- Person Bounding Box
- COCO 17 Keypoint
- Skeleton
- 실제 MP4 Frame 처리
- MP4 Frame별 Pose
- IP Webcam 연결 및 Frame 저장
- 직접 촬영 이미지 Pose 추론

### Day 01 Pipeline

```text
JPG / MP4 / IP Webcam
        ↓
Frame
        ↓
YOLO Pose
        ↓
Person Bounding Box
        ↓
COCO 17 Keypoint
        ↓
Skeleton
```

### 주요 실행 파일

```text
scripts/
├── 00_check_environment.py
├── 01_image_input.py
├── 02_video_input.py
├── 03_pose_image.py
├── 04_pose_video.py
└── 05_ip_webcam_capture.py
```

### 주요 결과

- `reports/day01/environment.txt`
- COCO8-Pose 이미지 Pose 결과
- `reports/day01/pose_demo_output.mp4`
- IP Webcam 촬영 이미지 Pose 결과

### 다음 수업

2일차에는 Keypoint 좌표와 Confidence를 자세히 읽고,
촬영 조건에 따라 Pose 품질이 어떻게 달라지는지 분석합니다.
````

---

# 39. Local Git으로 1일차 결과를 저장합니다

Git을 처음 시작하기 전에 `.gitignore`가 저장되어 있는지 확인합니다.

프로젝트 루트에서 Git 저장소를 만듭니다.

```bash
git init
```

현재 상태를 확인합니다.

```bash
git status
```

다음 항목이 추적 대상으로 나타나지 않는지 확인합니다.

```text
.venv/
data/
*.jpg
*.mp4
*.pt
artifacts/
```

코드와 문서를 등록합니다.

```bash
git add .
```

다시 확인합니다.

```bash
git status
```

첫 Commit을 만듭니다.

```bash
git commit -m "feat: complete subject10 day01 pose basics"
```

## Git 사용자 정보 오류가 나오는 경우

다음과 같은 메시지가 나오면 Git 사용자 이름과 이메일이 아직 설정되지 않은 것입니다.

```text
Author identity unknown
```

현재 프로젝트에 사용할 정보를 설정합니다.

```bash
git config user.name "자신의 이름"
```

```bash
git config user.email "자신의 이메일"
```

다시 Commit합니다.

```bash
git commit -m "feat: complete subject10 day01 pose basics"
```

Commit 결과를 확인합니다.

```bash
git log --oneline
```

한 줄의 Commit 기록이 보이면 Local Git 저장이 완료된 것입니다.

---

# 40. 오늘의 결과와 자가 체크리스트를 확인합니다

## 오늘 완성한 결과

1일차의 결과는 하나의 거대한 프로그램이 아닙니다.

다음 작은 결과들이 하나의 Pipeline으로 연결되면 성공입니다.

```text
① 교과 10 프로젝트 디렉터리
② Python 가상환경
③ OpenCV · PyTorch · Ultralytics 환경
④ GPU · CUDA 확인 결과
⑤ COCO8-Pose 실제 이미지 입력
⑥ 실제 MP4 입력
⑦ YOLO Pose 이미지 추론
⑧ Person Bounding Box
⑨ COCO 17 Keypoint
⑩ Skeleton 결과
⑪ 실제 MP4 Frame별 Pose 결과
⑫ IP Webcam 연결
⑬ 정면 · 양팔 들기 · 먼 거리 이미지
⑭ 직접 촬영 이미지 Pose 결과
⑮ README
⑯ Local Git Commit
```

## 오늘 반드시 이해해야 할 핵심

![[Pasted image 20260903173258.png]]

```text
입력 장치는 달라질 수 있습니다.

JPG
MP4
IP Webcam
CCTV

    ↓

하지만 결국 AI가 처리하는 것은 Frame입니다.

    ↓

YOLO Pose
    ↓
Person Bounding Box
    ↓
COCO 17 Keypoint
    ↓
Skeleton
```

앞으로는 오늘 얻은 Keypoint를 숫자로 분석하고 시간 정보와 연결합니다.

```text
1일차

Frame
→ Pose
→ Keypoint

    ↓

2~3일차

Confidence
→ 좌표
→ 각도
→ Feature

    ↓

4~6일차

Sequence
→ Action Recognition

    ↓

7~10일차

Realtime
→ Tracking
→ 보강
→ Failure Analysis
```

## 1일차 자가 체크리스트

아래 항목을 직접 체크합니다.

- [ ]  Pose Estimation과 Action Recognition의 차이를 설명할 수 있다.
- [ ]  JPG·MP4·IP Webcam이 결국 Frame으로 처리된다는 것을 이해했다.
- [ ]  교과 10의 가상환경을 활성화하고 필요한 라이브러리를 사용할 수 있다.
- [ ]  GPU·CUDA 사용 가능 여부를 확인할 수 있다.
- [ ]  COCO8-Pose 이미지와 실제 MP4를 프로젝트에서 읽을 수 있다.
- [ ]  `yolo11n-pose.pt`를 이용하여 이미지에 Pose 추론을 실행할 수 있다.
- [ ]  Person Bounding Box, COCO 17 Keypoint, Skeleton의 차이를 설명할 수 있다.
- [ ]  MP4의 여러 Frame에 Pose가 반복 적용된다는 것을 이해했다.
- [ ]  IP Webcam에서 Frame을 받아 이미지로 저장할 수 있다.
- [ ]  직접 촬영한 이미지에도 같은 Pose Pipeline을 적용할 수 있다.
- [ ]  자세와 촬영 거리가 달라지면 BBox·Keypoint·Skeleton 결과도 달라질 수 있음을 확인했다.
- [ ]  다른 이미지나 촬영 조건으로 Pose 결과를 스스로 비교해 볼 수 있다.
- [ ]  README를 정리하고 Local Git Commit까지 완료했다.



