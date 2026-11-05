## Keypoint 좌표 · Confidence · Pose 품질 분석

1일차에는 실제 JPG·MP4·IP Webcam을 입력하여 다음 Pose Pipeline이 정상적으로 동작하는지 확인했습니다.

![[Pasted image 20261104121212.png]]

```text
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

2일차에는 새로운 Pose 모델을 만들거나 학습하지 않습니다.

오늘은 1일차에서 얻었던 Pose 결과를 눈으로 보는 단계에서 숫자로 읽고 비교하는 단계로 확장합니다.

![[Pasted image 20260904183011.png]]

```text
1일차
Pose 결과를 눈으로 확인
        ↓
2일차
Keypoint 좌표 확인
        ↓
Confidence 확인
        ↓
촬영 조건별 비교
        ↓
Pose 품질 판단
```

> **오늘의 핵심**
>
> Skeleton이 화면에 잘 보인다고 끝나는 것이 아니라, 각 관절의 좌표와 Confidence를 확인하고 어떤 조건에서 Pose가 안정적이거나 불안정한지 근거를 가지고 설명하는 것이 목표입니다.

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

```
- 1일차의 subject10_pose_action 프로젝트와 가상환경을 그대로 이어서 사용할 수 있습니다.
- Person BBox Confidence와 Keypoint Confidence의 차이를 설명할 수 있습니다.
- COCO 17 Keypoint의 Pixel 좌표를 읽을 수 있습니다.
- Pixel 좌표를 0~1 범위의 Normalized 좌표로 바꾸는 이유를 설명할 수 있습니다.
- 각 Keypoint의 Confidence를 읽고 낮은 관절을 찾을 수 있습니다.
- Confidence Threshold를 바꾸어 Skeleton에 포함되는 관절이 어떻게 달라지는지 
  비교할 수 있습니다.
- 정면·양팔 들기·거리·측면·가림·화면 이탈 조건의 Pose 결과를 비교할 수 있습니다.
- 여러 이미지를 한 번에 분석하여 Pose 품질 비교 CSV를 만들 수 있습니다.
- MP4를 Frame 단위로 처리하여 Frame별 Pose 품질을 CSV로 저장할 수 있습니다.
- 숫자가 좋지 않은 Frame을 찾아 실제 영상과 다시 연결해 볼 수 있습니다.
- 팀별로 조건 하나를 정하고 실제 데이터를 촬영하여 Pose 품질 실험을 설계할 수 있습니다.
- 오늘 만든 코드와 분석 결과를 README에 정리하고 Git으로 저장할 수 있습니다.
```

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260905152824.png]]

# 오늘 사용할 데이터

2일차에서는 새로운 공개 데이터셋을 추가하지 않습니다.

현재 준비된 **6장의 직접 촬영 이미지**와 1일차에서 사용한 실제 MP4를 사용합니다.

특히 `front.jpg`는 기본 정면 Pose를 확인하는 기준 이미지이면서,
`distance_far.jpg`와 비교할 때는 **가까운 거리의 기준 이미지**로도 사용합니다.

| 데이터                         | 준비 <br>방법 | 오늘 사용하는 이유                                                                      |
| --------------------------- | --------- | ------------------------------------------------------------------------------- |
| `front.jpg`                 | 직접 <br>촬영 | 기본 정면 Pose의 기준을 확인하고, `distance_far.jpg`와 비교하여 화면 속 사람 크기 변화에 따른 Pose 품질 차이를 확인 |
| `arms_up.jpg`               | 직접 <br>촬영 | 팔을 올렸을 때 어깨·팔꿈치·손목 Keypoint의 위치와 Confidence 변화 확인                               |
| `side.jpg`                  | 직접 <br>촬영 | 측면에서 좌·우 관절이 서로 겹칠 때 Keypoint와 Skeleton이 어떻게 달라지는지 확인                           |
| `partial_body.jpg`          | 직접 <br>촬영 | 발목 등 몸 일부가 화면 밖으로 나갔을 때 해당 Keypoint의 Confidence와 Skeleton 변화 확인                 |
| `occlusion.jpg`             | 직접 <br>촬영 | 한쪽 팔 등 신체 일부가 다른 신체에 가려졌을 때 Keypoint Confidence가 어떻게 달라지는지 확인                   |
| `distance_far.jpg`          | 직접 <br>촬영 | `front.jpg`보다 사람이 멀리 있어 화면에서 작아졌을 때 BBox 크기와 Keypoint Confidence 변화 비교          |
| `solution_ci_pose_demo.mp4` | 1일차 MP4   | 정지 이미지가 아니라 실제 영상에서 Frame별 Pose 품질과 Confidence 변화를 숫자로 확인                       |

오늘은 **같은 Pose 모델에 서로 다른 입력 조건을 넣었을 때 결과가 어떻게 달라지는지** 확인합니다.

```text
같은 YOLO Pose
        +
서로 다른 촬영 조건
        ↓
BBox 변화
Keypoint 변화
Confidence 변화
Skeleton 변화
```

현재 본 실습에 필요한 이미지 파일은 다음 **6장**입니다.

```text
data/day02/images/

front.jpg
arms_up.jpg
side.jpg
partial_body.jpg
occlusion.jpg
distance_far.jpg
```

> `front.jpg`와 `distance_far.jpg`는 가까움과 멀어짐을 비교하는 두 조건으로 사용합니다.  
> 별도의 `distance_near.jpg`, `distance_mid.jpg`는 필요하지 않습니다.

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | Pose 결과를 숫자로 정리 | 관절 데이터를 읽고 저장 |
| OpenCV | 이미지·영상 Frame 처리 | 촬영 이미지와 MP4를 읽고 결과 저장 |
| Ultralytics YOLO Pose | Person·Keypoint 추론 | 사람과 17개 관절 위치 찾기 |
| NumPy | Keypoint 숫자 계산 | 평균 Confidence와 배열 처리 |
| Pandas | CSV 결과 확인 | 품질이 낮은 Frame을 빠르게 찾기 |
| CSV | 분석 결과 저장 | 이미지·Frame별 숫자를 표 형태로 기록 |
| Confidence Threshold | 사용할 관절 기준 비교 | 어느 정도 신뢰도의 관절까지 볼지 결정 |
| Git | 2일차 코드 기록 | 오늘 추가한 코드와 문서를 버전관리 |

> 새로운 라이브러리를 다시 설치하는 날이 아닙니다. 
> 1일차의 `pyproject.toml`, `.venv`, `yolo11n-pose.pt` 환경을 그대로 사용합니다.

---

# PART 1. 1일차 프로젝트에서 2일차를 이어서 시작합니다

---

# 1. 1일차와 2일차가 어떻게 연결되는지 확인합니다

1일차에는 다음 질문에 답했습니다.

```text
이미지나 영상에서
사람과 관절을 찾을 수 있는가?
```

2일차에는 질문이 달라집니다.

```text
찾은 관절이
얼마나 안정적인가?

어떤 관절이
불안정한가?

촬영 조건이 달라지면
숫자가 어떻게 변하는가?
```

따라서 오늘은 다음 위치에 해당합니다.

![[Pasted image 20260916143104.png]]

2일차에서 신뢰할 수 있는 Keypoint를 확인해야 3일차에서 관절각도와 Feature를 만들 때 잘못된 좌표를 그대로 사용하지 않을 수 있습니다.

---

# 2. 1일차 프로젝트를 다시 엽니다

새로운 프로젝트를 만들지 않습니다.

WSL Ubuntu 터미널에서 1일차 프로젝트로 이동합니다.

```bash
cd ~/ai_vision/subject10_pose_action
```

현재 위치를 확인합니다.

```bash
pwd
```

다음과 비슷하면 정상입니다.

```text
/home/사용자계정/ai_vision/subject10_pose_action
```

1일차 가상환경을 다시 활성화합니다.

```bash
source .venv/bin/activate
```

현재 Python 위치를 확인합니다.

```bash
which python
```

다음처럼 프로젝트의 `.venv` 안에 있는 Python이 나오면 됩니다.

```text
.../subject10_pose_action/.venv/bin/python
```

### 지금 무엇을 한 것인가요?

```text
1일차 프로젝트 위치로 이동
        ↓
1일차에서 만든 .venv 활성화
        ↓
같은 Python 환경으로
2일차 수업 시작
```

> 2일차마다 새 가상환경을 만드는 것이 아닙니다. 교과 10의 10일 동안 같은 프로젝트 환경을 이어서 사용합니다.

---

# 3. 1일차 코드와 실행환경을 짧게 확인합니다

먼저 1일차에 만든 주요 파일을 확인합니다.

```bash
find scripts src configs -maxdepth 2 -type f | sort
```

1일차 기준으로 다음 파일들이 확인되면 됩니다.

```text
configs/project.json
src/common.py

scripts/00_check_environment.py
scripts/01_image_input.py
scripts/02_video_input.py
scripts/03_pose_image.py
scripts/04_pose_video.py
scripts/05_ip_webcam_capture.py
```

실행환경도 한 번 확인합니다.

```bash
python scripts/00_check_environment.py
```

다음 항목을 확인합니다.

```text
OpenCV      → 정상
PyTorch     → 정상
Ultralytics → 정상
CUDA        → True 또는 CPU 사용 가능
Pose model  → OK
```

> 1일차에서 모델이 정상적으로 로드되었다면 오늘 다시 설치할 필요는 없습니다. 
> 수업 시작 전 같은 환경을 사용하고 있는지만 짧게 확인합니다.

---

# 4. 2일차 폴더를 추가합니다

오늘 사용할 입력과 분석 결과를 1일차와 구분하여 저장합니다.

```bash
mkdir -p \
data/day02/images \
data/day02/videos \
reports/day02/images \
reports/day02/videos \
reports/day02/tables \
reports/day02/challenge
```

확인합니다.

```bash
find data/day02 reports/day02 -maxdepth 2 -type d | sort
```

오늘 추가되는 구조는 다음과 같습니다.

```text
subject10_pose_action/
│
├── data/
│   ├── day01/
│   └── day02/
│       ├── images/
│       └── videos/
│
├── reports/
│   ├── day01/
│   └── day02/
│       ├── images/
│       ├── videos/
│       ├── tables/
│       └── challenge/
│
├── scripts/
│   ├── 06_check_day02_data.py
│   ├── 07_pose_image_detail.py
│   ├── 08_draw_skeleton_quality.py
│   ├── 09_compare_pose_images.py
│   └── 10_pose_video_quality.py
│
└── src/
    └── pose_utils.py
```

### 왜 Day02 폴더를 따로 만들까요?

```text
원본 입력
→ data/day02

실행 결과
→ reports/day02
```

날짜별 입력과 결과를 섞지 않으면 나중에 문제가 생겼을 때 어떤 수업에서 만든 결과인지 쉽게 찾을 수 있습니다.

---

# PART 2. 2일차에 사용할 데이터와 핵심 개념을 준비합니다

---

# 5. 2일차 분석용 이미지 6장을 입력 폴더에 배치합니다

2일차에서는 1일차에 IP Webcam으로 촬영한 이미지를 그대로 복사하여 사용하지 않습니다.

1일차에서는 JPG·MP4·IP Webcam을 실제 입력으로 연결하고, YOLO Pose를 이용하여 사람과 관절이 정상적으로 검출되는지 확인했습니다.

2일차에서는 학습 목적이 달라집니다.

```
1일차
입력 연결
        ↓
YOLO Pose
        ↓
사람과 관절 검출 확인

        ↓

2일차
조건이 다른 이미지 비교
        ↓
Keypoint 좌표
        ↓
Confidence
        ↓
Pose 품질 분석
```

따라서 2일차 본 실습에서는 조건별 Pose 품질을 비교하기 위해 준비한 6장의 이미지를 사용합니다.

사용할 이미지는 다음과 같습니다.

```
front.jpg
arms_up.jpg
side.jpg
partial_body.jpg
occlusion.jpg
distance_far.jpg
```

각 이미지는 다음과 같은 역할을 합니다.

```
front.jpg
→ 기본 정면 Pose의 기준

arms_up.jpg
→ 팔을 올렸을 때
  Shoulder · Elbow · Wrist의
  Keypoint와 Confidence 확인

side.jpg
→ 사람이 측면을 보고 있을 때
  좌·우 관절이 겹치면서
  Pose 품질이 어떻게 달라지는지 확인

partial_body.jpg
→ 신체 일부가 화면 밖으로 벗어났을 때
  해당 Keypoint의 Confidence 변화 확인

occlusion.jpg
→ 팔이나 신체 일부가 가려졌을 때
  가려진 관절의 Confidence 변화 확인

distance_far.jpg
→ 사람이 화면에서 작게 보일 때
  BBox 크기와 Keypoint Confidence 변화 확인
```

---

## 5.1 현재 2일차 이미지 폴더를 확인합니다

먼저 프로젝트 루트에 있는 상태에서 다음 명령을 실행합니다.

```bash
ls -lh data/day02/images/
```

아직 강사가 제공한 이미지를 넣지 않았다면 다음처럼 나타날 수 있습니다.

```bash
total 0
```

이 결과는 오류가 아닙니다.

```
data/day02/images/
        ↓
폴더는 생성되어 있음
        ↓
하지만 이미지 파일은 아직 없음
```

이제 제공한 2일차 실습 이미지를 이 폴더에 넣습니다.

---

## 5.2 Windows 파일 탐색기로 프로젝트 폴더를 엽니다

WSL 터미널에서 현재 프로젝트 루트인지 확인합니다.

```bash
pwd
```

다음과 비슷한 경로가 나오면 됩니다.

```bash
/home/사용자계정/ai_vision/subject10_pose_action
```

현재 프로젝트 폴더를 Windows 파일 탐색기로 엽니다.

```bash
explorer.exe .
```

Windows 파일 탐색기가 열리면 다음 폴더로 이동합니다.

```
data
└── day02
    └── images
```

즉 최종 위치는 다음입니다.

```
subject10_pose_action/data/day02/images/
```

---

## 5.3 제공한 이미지 6장을 `images` 폴더에 넣습니다

제공한 다음 6장의 이미지를 선택합니다.

```
front.jpg
arms_up.jpg
side.jpg
partial_body.jpg
occlusion.jpg
distance_far.jpg
```

그리고 다음 폴더에 복사합니다.

```
data/day02/images/
```

최종 폴더 구조는 다음과 같습니다.

```
subject10_pose_action/
│
└── data/
    └── day02/
        └── images/
            ├── front.jpg
            ├── arms_up.jpg
            ├── side.jpg
            ├── partial_body.jpg
            ├── occlusion.jpg
            └── distance_far.jpg
```

---

## 5.4 이미지 6장이 정상적으로 들어갔는지 확인합니다

Windows 파일 탐색기에서 복사가 끝났다면 WSL 터미널로 돌아옵니다.

다음 명령을 실행합니다.

```bash
ls -lh data/day02/images/
```

정상적으로 배치되었다면 다음 6개 파일이 보여야 합니다.

```
arms_up.jpg
distance_far.jpg
front.jpg
occlusion.jpg
partial_body.jpg
side.jpg
```

파일 순서는 다르게 표시될 수 있습니다.

중요한 것은 **6개 파일이 모두 존재하는지**입니다.

2일차에서 요구하는 입력 데이터도 이 6개 이미지입니다.

---

## 5.5 왜 1일차 이미지를 그대로 사용하지 않나요?

![[Pasted image 20261104121853.png]]

1일차와 2일차는 같은 프로젝트와 같은 Pose 모델을 이어서 사용하지만, 실습의 목적은 서로 다릅니다.

```
[1일차]

JPG · MP4 · IP Webcam
        ↓
Frame
        ↓
YOLO Pose
        ↓
COCO 17 Keypoint

목적
→ 입력과 Pose Pipeline이
  정상적으로 동작하는지 확인
```

2일차에서는 이미 Pose가 정상적으로 동작한다는 것을 알고 있습니다.

따라서 다음 단계로 넘어갑니다.

```
[2일차]

조건이 다른 이미지 6장
        ↓
같은 YOLO Pose
        ↓
Keypoint 좌표
        ↓
Confidence
        ↓
BBox 크기
        ↓
조건별 Pose 품질 비교
```

즉 1일차에서 2일차로 이어지는 것은 **사진 파일 자체가 아니라 프로젝트 환경과 Pose 처리 Pipeline**입니다.

```
1일차 결과

Pose를 실행할 수 있음
        ↓
사람과 관절을 찾을 수 있음

        ↓ 이어서

2일차

찾은 관절의 좌표를 읽음
        ↓
Confidence를 확인함
        ↓
촬영 조건별로 품질을 비교함
```

따라서 2일차에서는 비교 조건이 명확하도록 강사가 준비한 동일한 6장의 데이터를 사용하여 같은 기준으로 분석합니다.

---

# 6. 2일차 입력 데이터가 올바르게 준비되었는지 코드로 확인합니다

실습을 시작하기 전에 파일명이 틀렸거나 이미지가 손상되어 있으면 이후 모든 코드에서 오류가 발생할 수 있습니다.

따라서 2일차의 첫 Python 코드는 **필수 이미지 6장이 모두 존재하고 OpenCV로 읽히는지 확인**하는 역할을 합니다.

### 이 코드는 왜 작성하나요?

수업 중 가장 흔한 오류 중 하나는 알고리즘 문제가 아니라 **파일명·경로·이미지 파일 자체의 문제**입니다.

```text
필수 파일 목록 확인
        ↓
파일 존재 여부 확인
        ↓
OpenCV로 실제 읽기
        ↓
Width · Height 확인
        ↓
모두 정상
        ↓
Pose 분석 시작
```

### 코드흐름

```text
필수 이미지 6개 이름을 준비한다
        ↓
각 파일이 실제로 존재하는지 확인한다
        ↓
OpenCV로 이미지를 읽는다
        ↓
읽히면 Width·Height 출력
        ↓
하나라도 없거나 읽히지 않으면 오류
        ↓
모두 정상이면 READY 출력
```

**파일: `scripts/06_check_day02_data.py`**

```python
from __future__ import annotations

# ① 필요한 라이브러리 불러오기
from pathlib import Path
import cv2


# ② 프로젝트 경로 및 2일차 이미지 폴더 지정
ROOT = Path(__file__).resolve().parents[1]
IMAGE_DIR = ROOT / "data" / "day02" / "images"


# ③ 2일차 실습에 필요한 이미지 6장 지정
REQUIRED_IMAGES = [
    "front.jpg",
    "arms_up.jpg",
    "side.jpg",
    "partial_body.jpg",
    "occlusion.jpg",
    "distance_far.jpg",
]


def main() -> None:

    # ④ 이미지 검사 결과를 저장할 목록 준비
    missing = []      # 파일이 없는 이미지
    unreadable = []   # 파일은 있지만 읽을 수 없는 이미지

    print(f"Image directory: {IMAGE_DIR}")
    print()

    # ⑤ 이미지 6장을 순서대로 검사
    for filename in REQUIRED_IMAGES:
        image_path = IMAGE_DIR / filename

        # 이미지 파일이 존재하는지 확인
        if not image_path.exists():
            missing.append(filename)
            print(f"[MISSING] {filename}")
            continue

        # OpenCV로 이미지 읽기
        image = cv2.imread(str(image_path))

        # 파일은 있지만 이미지를 읽을 수 없는 경우
        if image is None:
            unreadable.append(filename)
            print(f"[UNREADABLE] {filename}")
            continue

        # 정상 이미지의 너비와 높이 확인
        height, width = image.shape[:2]

        print(
            f"[OK] {filename:18s} "
            f"{width} x {height}"
        )

    # ⑥ 이미지 검사 결과 확인
    print()

    if missing or unreadable:

        # 존재하지 않는 이미지 목록 출력
        if missing:
            print("Missing:", ", ".join(missing))

        # 읽을 수 없는 이미지 목록 출력
        if unreadable:
            print("Unreadable:", ", ".join(unreadable))

        # 하나라도 문제가 있으면 오류 발생
        raise RuntimeError(
            "2일차 입력 이미지를 다시 확인하세요."
        )

    # ⑦ 모든 이미지가 정상이면 준비 완료 출력
    print("Day02 image data: READY")


# 프로그램 실행 시작
if __name__ == "__main__":
    main()
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ Path → 이미지 파일 경로 관리
   └─ cv2  → OpenCV로 이미지 읽기
                ↓
② 프로젝트와 이미지 폴더 경로 지정
   ├─ ROOT → 프로젝트 최상위 폴더
   └─ IMAGE_DIR → data/day02/images/
                ↓
③ 확인할 이미지 6장 지정
   ├─ front.jpg        → 정면
   ├─ arms_up.jpg      → 양팔 들기
   ├─ side.jpg         → 측면
   ├─ partial_body.jpg → 신체 일부만 보임
   ├─ occlusion.jpg    → 신체 일부 가림
   └─ distance_far.jpg → 먼 거리
                ↓
④ 검사 결과를 저장할 목록 준비
   ├─ missing    → 없는 이미지 목록
   └─ unreadable → 읽을 수 없는 이미지 목록
                ↓
⑤ 이미지 6장 순서대로 검사
   ├─ 이미지 파일이 존재하는가?
   │    ├─ 아니요 → MISSING 기록
   │    └─ 예 → OpenCV로 이미지 읽기
   │
   ├─ 이미지를 정상적으로 읽었는가?
   │    ├─ 아니요 → UNREADABLE 기록
   │    └─ 예 → 이미지 크기 확인
   │
   ├─ 정상 이미지 → OK + 너비·높이 출력
   └─ 다음 이미지 검사 (6장 반복)
                ↓
⑥ 전체 검사 결과 확인
   ├─ MISSING 또는 UNREADABLE이 있는가?
   │    ├─ 예 → 문제 파일 목록 출력
   │    │       RuntimeError 발생
   │    └─ 아니요 → 다음 단계
                ↓
⑦ 모든 이미지 준비 완료
   └─ Day02 image data: READY 출력
```

실행합니다.

```bash
python scripts/06_check_day02_data.py
```

정상이라면 다음과 비슷하게 출력됩니다.

```text
[OK] front.jpg          2552 x 4000
[OK] arms_up.jpg        1702 x 3574
[OK] side.jpg           2305 x 4000
[OK] partial_body.jpg   1924 x 2791
[OK] occlusion.jpg      2147 x 3797
[OK] distance_far.jpg   1575 x 3748

Day02 image data: READY
```

**실행 결과 해석**

- `[OK]`: 이미지 파일이 존재하며 OpenCV에서 정상적으로 읽혔다는 의미입니다.
    
- `front.jpg` 등: 2일차 실습에 사용할 이미지 파일명입니다.
    
- `2552 x 4000`: 이미지의 해상도로, 가로 2552픽셀 × 세로 4000픽셀을 의미합니다. 이미지마다 크기가 다를 수 있습니다.
    
- `Day02 image data: READY`: 필요한 이미지 6장을 모두 정상적으로 읽었으며, 다음 실습을 진행할 준비가 되었다는 의미입니다.
    

핵심: 아직 YOLO Pose를 실행한 것은 아닙니다. 이번 단계에서는 이미지 6장이 정상적으로 준비되었는지 확인했으며, 다음 실습에서 관절 위치와 Confidence를 분석합니다.

---

# 7. Person BBox Confidence와 Keypoint Confidence를 구분합니다

1일차 결과 이미지에는 다음과 같은 값이 보였습니다.

```text
person 0.92
```

이 값은 **사람을 검출한 BBox의 Confidence**입니다.

```text
Person BBox Confidence

이 사각형 안에 사람이 있다고
모델이 얼마나 확신하는가?
```

하지만 Pose에서는 각 관절에도 별도의 Confidence가 있습니다.

예:

```text
Left Shoulder = 0.98
Left Elbow    = 0.94
Left Wrist    = 0.52
Left Ankle    = 0.24
```

```text
Keypoint Confidence

이 위치가 해당 관절이라고
모델이 얼마나 신뢰하는가?
```

따라서 다음 상황이 가능합니다.

```text
사람 Detection은 잘 됨
BBox Confidence 높음
        ↓
하지만 팔이 가려짐
        ↓
Wrist Confidence 낮음
```

즉,

```text
사람을 잘 찾았다
    ≠
모든 관절을 잘 찾았다
```

오늘부터는 **사람 검출과 관절 품질을 따로 확인**합니다.

![[Pasted image 20260916151808.png]]

---

# 8. Pixel 좌표와 Normalized 좌표를 이해합니다

![[Pasted image 20260916152316.png]]

Pose 모델은 관절 위치를 `(x, y)` 숫자로 반환합니다.

예를 들어 이미지가 다음 크기라고 가정합니다.

```text
Width  = 1280
Height = 720
```

왼쪽 손목이 이미지 중앙에 있다면 Pixel 좌표는 다음처럼 표현할 수 있습니다.

```text
Left Wrist
x = 640
y = 360
```

## Pixel 좌표

Pixel 좌표는 **실제 이미지에서 몇 번째 픽셀에 관절이 있는지** 나타냅니다.

```text
왼쪽 위
(0, 0)
   ┌────────────────→ x 증가
   │
   │
   ↓ y 증가
```

OpenCV 이미지에서는 왼쪽 위가 `(0, 0)`입니다.

따라서 화면에서 아래로 내려갈수록 `y` 값은 커집니다.

## Normalized 좌표

Normalized 좌표는 Pixel 좌표를 이미지의 너비와 높이로 나누어 **0~1 범위로 바꾼 값**입니다.

```text
normalized_x = pixel_x / image_width
normalized_y = pixel_y / image_height
```

위 예에서는:

```text
640 / 1280 = 0.5
360 / 720  = 0.5
```

따라서:

```text
Pixel 좌표
(640, 360)

Normalized 좌표
(0.5, 0.5)
```

### 왜 Normalized 좌표가 필요한가요?

이미지 해상도가 달라지면 같은 위치라도 Pixel 숫자는 달라집니다.

```text
640 × 480 영상의 중앙
→ (320, 240)

1280 × 960 영상의 중앙
→ (640, 480)
```

하지만 Normalized 좌표로 바꾸면 둘 다 비슷합니다.

```text
(0.5, 0.5)
```

> 오늘은 Pixel과 Normalized 좌표를 모두 저장합니다. 3일차부터 사람 위치와 크기의 영향을 줄이는 정규화를 더 자세히 다룹니다.

---

# 9. COCO 17 Keypoint 정보를 공통 코드로 분리합니다

1일차 `03_pose_image.py` 안에는 COCO 17 Keypoint 이름이 직접 들어 있었습니다.

2일차부터 여러 분석 코드가 같은 관절 이름과 Skeleton 연결 정보를 반복해서 사용합니다.

따라서 공통 정보를 `src/pose_utils.py`로 분리합니다.

### 이 파일은 왜 작성하나요?

```text
여러 Python 파일마다
17개 관절 이름을 반복 작성
        X

src/pose_utils.py 한 곳에 정의
        ↓
필요한 스크립트에서 import
        O
```

이렇게 하면 관절 번호와 이름을 여러 파일에서 다르게 작성하는 실수를 줄일 수 있습니다.

### 코드흐름

```text
0~16번 관절 이름을 정의한다
        ↓
Skeleton에서 연결할 관절 쌍을 정의한다
        ↓
Confidence가 높으면 VALID
중간이면 WARNING
낮으면 LOW로 구분한다
```

**파일: `src/pose_utils.py`**

```python
from __future__ import annotations


# ① COCO 17개 Keypoint의 이름 정의
# YOLO Pose가 검출한 관절 번호(0~16)에 해당하는 이름
COCO_KEYPOINT_NAMES = [
    "nose",            # 0: 코
    "left_eye",        # 1: 왼쪽 눈
    "right_eye",       # 2: 오른쪽 눈
    "left_ear",        # 3: 왼쪽 귀
    "right_ear",       # 4: 오른쪽 귀
    "left_shoulder",   # 5: 왼쪽 어깨
    "right_shoulder",  # 6: 오른쪽 어깨
    "left_elbow",      # 7: 왼쪽 팔꿈치
    "right_elbow",     # 8: 오른쪽 팔꿈치
    "left_wrist",      # 9: 왼쪽 손목
    "right_wrist",     # 10: 오른쪽 손목
    "left_hip",       # 11: 왼쪽 골반
    "right_hip",      # 12: 오른쪽 골반
    "left_knee",      # 13: 왼쪽 무릎
    "right_knee",     # 14: 오른쪽 무릎
    "left_ankle",     # 15: 왼쪽 발목
    "right_ankle",    # 16: 오른쪽 발목
]


# ② Skeleton을 그릴 관절 연결 관계 정의
# (시작 관절 번호, 끝 관절 번호) 형태로 작성
COCO_SKELETON = [
    (15, 13),  # 왼쪽 발목 ↔ 왼쪽 무릎
    (13, 11),  # 왼쪽 무릎 ↔ 왼쪽 골반
    (16, 14),  # 오른쪽 발목 ↔ 오른쪽 무릎
    (14, 12),  # 오른쪽 무릎 ↔ 오른쪽 골반
    (11, 12),  # 왼쪽 골반 ↔ 오른쪽 골반
    (5, 11),   # 왼쪽 어깨 ↔ 왼쪽 골반
    (6, 12),   # 오른쪽 어깨 ↔ 오른쪽 골반
    (5, 6),    # 왼쪽 어깨 ↔ 오른쪽 어깨
    (5, 7),    # 왼쪽 어깨 ↔ 왼쪽 팔꿈치
    (6, 8),    # 오른쪽 어깨 ↔ 오른쪽 팔꿈치
    (7, 9),    # 왼쪽 팔꿈치 ↔ 왼쪽 손목
    (8, 10),   # 오른쪽 팔꿈치 ↔ 오른쪽 손목
    (1, 2),    # 왼쪽 눈 ↔ 오른쪽 눈
    (0, 1),    # 코 ↔ 왼쪽 눈
    (0, 2),    # 코 ↔ 오른쪽 눈
    (1, 3),    # 왼쪽 눈 ↔ 왼쪽 귀
    (2, 4),    # 오른쪽 눈 ↔ 오른쪽 귀
    (3, 5),    # 왼쪽 귀 ↔ 왼쪽 어깨
    (4, 6),    # 오른쪽 귀 ↔ 오른쪽 어깨
]


# ③ 관절 Confidence 판정 함수 정의
def confidence_state(
    confidence: float,
    valid_threshold: float = 0.50,
    warning_threshold: float = 0.30,
) -> str:

    # ④ Confidence 값에 따라 상태 반환

    # 0.50 이상이면 정상 신뢰도
    if confidence >= valid_threshold:
        return "VALID"

    # 0.30 이상 ~ 0.50 미만이면 주의
    if confidence >= warning_threshold:
        return "WARNING"

    # 0.30 미만이면 낮은 신뢰도
    return "LOW"
```

**의사코드**

```
① COCO 17개 Keypoint 이름 정의
   ├─ YOLO가 검출하는 관절 번호 0~16 준비
   ├─ 0번  → 코
   ├─ 5번  → 왼쪽 어깨
   ├─ 7번  → 왼쪽 팔꿈치
   ├─ 9번  → 왼쪽 손목
   └─ 나머지 관절도 이름과 연결
                ↓
② Skeleton 관절 연결 관계 정의
   ├─ 연결할 관절 번호를 쌍으로 저장
   ├─ (5, 7)  → 왼쪽 어깨와 팔꿈치
   ├─ (7, 9)  → 왼쪽 팔꿈치와 손목
   └─ 총 19개 연결 관계 정의
                ↓
③ confidence_state() 함수 정의
   ├─ 관절의 Confidence 값을 입력받음
   ├─ VALID 기준값   → 0.50
   └─ WARNING 기준값 → 0.30
                ↓
④ Confidence 값에 따라 상태 판정
   ├─ Confidence ≥ 0.50
   │       └─ VALID 반환
   │
   ├─ 0.30 ≤ Confidence < 0.50
   │       └─ WARNING 반환
   │
   └─ Confidence < 0.30
           └─ LOW 반환
```

### 왜 Confidence 기준을 0.50과 0.30으로 정했을까요?

YOLO Pose는 관절마다 **위치를 얼마나 확신하는지 나타내는 Confidence 값(0~1)**을 제공합니다.

이번 실습에서는 촬영 조건에 따라 관절의 Confidence가 어떻게 달라지는지 비교하기 위해 임시로 두 가지 기준을 정했습니다.

|Confidence|수업용 판정|의미|
|---|---|---|
|0.50 이상|VALID|상대적으로 신뢰도가 높은 관절|
|0.30 이상 ~ 0.50 미만|WARNING|신뢰도가 중간인 관절|
|0.30 미만|LOW|신뢰도가 낮아 확인이 필요한 관절|

**왜 하필 0.50과 0.30일까요?**

이 수치는 YOLO가 정한 공식 기준이 아닙니다. 학생들이 높은 값, 중간 값, 낮은 값을 쉽게 구분하도록 **교육용으로 임의 설정한 기준**입니다.

예를 들어 정면에서는 손목 Confidence가 `0.85`였지만 팔이 가려진 이미지에서는 `0.35`로 낮아졌다면, 촬영 조건이 관절 검출 신뢰도에 영향을 준다는 것을 확인할 수 있습니다.

**핵심:** 이번 실습은 관절의 정확도를 확정하는 것이 아니라, 같은 기준으로 6장의 이미지를 비교하며 Pose 품질 차이를 관찰하는 것이 목적입니다. 실제 프로젝트에서는 검증 데이터를 통해 기준값을 다시 정해야 합니다.

---

# PART 3. 한 이미지의 Pose 결과를 숫자로 읽습니다

---

# 10. 한 이미지의 Keypoint를 CSV로 저장하는 코드를 작성합니다

![[Pasted image 20261104124642.png]]

1일차에는 YOLO Pose로 사람의 관절 17개를 검출하고, 관절 위치를 이미지에 표시하거나 터미널에서 좌표를 확인했습니다.

이번에는 **관절 좌표와 Confidence를 CSV 파일로 저장하여, 다른 이미지의 Pose 결과와 비교할 수 있도록** 만들겠습니다.

### ① 이 코드는 왜 작성하나요?

예를 들어 정면 사진에서는 손목이 잘 검출되지만, 팔이 가려진 사진에서는 손목의 위치가 부정확하거나 Confidence가 낮아질 수 있습니다.

이러한 차이를 숫자로 비교하려면 각 관절의 좌표와 Confidence를 파일로 저장해야 합니다.

CSV 파일로 저장하면 Excel이나 Pandas에서 관절별 결과를 쉽게 확인하고 비교할 수 있습니다.

CSV를 답안지처럼 이해하셔도 됩니다. 다만 정확하게는 YOLO Pose가 분석한 결과를 숫자로 기록한 결과표입니다.

예를 들어 다음과 같습니다.

- 이미지(JPG) → AI에게 주는 문제지
    
- YOLO Pose → 이미지를 분석하는 AI
    
- CSV 파일 → AI가 찾은 관절 위치와 신뢰도를 기록한 답안지

```
[문제지] 사람 이미지
         ↓
[AI] YOLO Pose 분석
         ↓
[답안지] CSV
         ├─ 왼쪽 손목 위치: (320, 520)
         ├─ Confidence: 0.88
         └─ Quality: VALID
```

단, 중요한 차이점이 있습니다.

CSV는 AI가 예측한 답안지이지, 실제 정답을 기록한 정답지(Ground Truth)는 아닙니다.

따라서 CSV에 `VALID`라고 기록되더라도 실제 관절 위치가 정확하다고 보장되는 것은 아닙니다.

자세와 촬영 조건을 정확하게 정해서 촬영하면 CSV 결과를 훨씬 의미 있게 비교할 수 있습니다. 다만 여기서 중요한 것은 AI의 예측 결과(CSV)와 실제 정답(Ground Truth)을 구분하는 것입니다.

### CSV와 정답지(Ground Truth)는 무엇이 다른가요?

YOLO Pose에서 CSV는 AI가 예측한 결과를 기록하는 파일이고, Ground Truth는 실제 정답을 기록한 데이터입니다.

| 구분         | CSV (AI 예측 결과)            | Ground Truth (정답지)      |
| ---------- | ------------------------- | ----------------------- |
| 누가 만드나요?   | YOLO Pose가 자동 생성한 결과를 저장  | 사람이 직접 표시하고 검수          |
| 무엇을 기록하나요? | AI가 예측한 관절 좌표와 Confidence | 실제 관절의 정답 좌표            |
| 언제 사용하나요?  | 2일차 Pose 품질 비교            | 모델의 실제 정확도를 평가할 때       |
| 목적         | AI가 관절을 어떻게 찾았는지 비교       | AI가 관절을 얼마나 정확히 찾았는지 검증 |
![[Pasted image 20261104133813.png]]
### ② 앞에서 작성한 어떤 파일과 연결되나요?

|파일|역할|
|---|---|
|`configs/project.json`|YOLO 모델명, 이미지 크기, 검출 기준 설정|
|`src/common.py`|JSON 설정 읽기 및 GPU·CPU 선택|
|`src/pose_utils.py`|17개 관절 이름과 Confidence 판정 기준 제공|
|`scripts/07_pose_image_detail.py`|위 설정과 기능을 활용하여 이미지의 관절 정보를 CSV로 저장|

앞에서 작성한 공통 파일들을 재사용하므로 동일한 설정과 기준으로 분석할 수 있습니다.

### ③ CSV에는 어떤 정보를 저장하나요?

|저장 정보|의미|
|---|---|
|Person BBox Confidence|이미지에서 사람을 검출한 신뢰도|
|Pixel x, y|원본 이미지에서 관절이 위치한 픽셀 좌표|
|Normalized x, y|이미지 너비와 높이로 나눈 정규화 좌표(0~1 범위)|
|Keypoint Confidence|각 관절 위치에 대한 모델의 신뢰도|
|Quality|Confidence를 기준으로 구분한 VALID / WARNING / LOW|

예를 들어 손목의 픽셀 좌표가 `(500, 300)`이고 이미지 크기가 `1000 × 600`이라면, 정규화 좌표는 `(0.5, 0.5)`가 됩니다.

정규화 좌표를 함께 저장하는 이유는 이미지 크기가 달라도 관절의 상대적인 위치를 비교하기 위해서입니다.

### ④ 코드의 전체 처리 흐름

```
① 이미지 파일 입력
        ↓
② project.json 설정 읽기
   공통 함수 및 관절 기준 불러오기
        ↓
③ YOLO Pose 실행
   사람 BBox와 17개 관절 검출
        ↓
④ 관절별 데이터 추출
   ├─ Pixel x, y
   ├─ Normalized x, y
   └─ Confidence
        ↓
⑤ Confidence에 따라
   VALID / WARNING / LOW 구분
        ↓
⑥ Pose 결과 이미지와 CSV 저장
        ↓
⑦ 다음 이미지와 품질 비교에 활용
```

**파일:** `scripts/07_pose_image_detail.py`

이번 실습의 핵심은 **이미지에 표시된 관절을 숫자 데이터로 저장하는 것**입니다. 이렇게 저장한 CSV는 이후 정면·측면·가림·먼 거리 등 촬영 조건에 따른 관절 검출 품질을 비교하는 기초 자료가 됩니다.

```python

from __future__ import annotations

# ① 필요한 라이브러리 불러오기
import argparse
import csv
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
from src.pose_utils import (
    COCO_KEYPOINT_NAMES,
    confidence_state,
)


# ③ 터미널에서 이미지 경로와 Confidence 기준 입력받기
def parse_args():
    parser = argparse.ArgumentParser()

    # 분석할 이미지 지정 (기본값: front.jpg)
    parser.add_argument(
        "--source",
        default=str(
            ROOT
            / "data"
            / "day02"
            / "images"
            / "front.jpg"
        ),
        help="분석할 이미지 파일",
    )

    # 관절을 VALID로 판정할 Confidence 기준
    parser.add_argument(
        "--kp-conf",
        type=float,
        default=0.50,
        help="VALID로 판단할 Keypoint Confidence 기준",
    )

    return parser.parse_args()


def main() -> None:

    # ④ 입력 이미지 확인 및 크기 가져오기
    args = parse_args()

    source = Path(args.source)

    # 이미지 파일 존재 여부 확인
    if not source.exists():
        raise FileNotFoundError(
            f"이미지가 없습니다: {source}"
        )

    # OpenCV로 이미지 읽기
    image = cv2.imread(str(source))

    if image is None:
        raise RuntimeError(
            f"이미지를 읽지 못했습니다: {source}"
        )

    # 원본 이미지의 높이와 너비 확인
    height, width = image.shape[:2]

    # ⑤ 프로젝트 설정 읽기 및 YOLO Pose 모델 실행
    config = load_config()
    device = resolve_device(config["device"])

    model = YOLO(config["pose_model"])

    # 이미지에서 사람과 17개 관절 추론
    results = model.predict(
        source=image,
        imgsz=config["image_size"],
        conf=config["person_confidence"],
        device=device,
        verbose=False,
    )

    # 첫 번째 이미지의 추론 결과 가져오기
    result = results[0]

    # ⑥ 사람과 Keypoint 검출 여부 확인
    if (
        result.boxes is None
        or result.keypoints is None
        or result.keypoints.xy is None
        or len(result.boxes) == 0
    ):
        print("사람 또는 Keypoint를 찾지 못했습니다.")
        return

    # ⑦ BBox 좌표·신뢰도와 관절 데이터 추출
    # 사람의 Bounding Box 좌표
    boxes_xyxy = result.boxes.xyxy.cpu().numpy()

    # 사람 검출 Confidence
    boxes_conf = result.boxes.conf.cpu().numpy()

    # 17개 관절의 Pixel x, y 좌표
    xy = result.keypoints.xy.cpu().numpy()

    # 관절별 Confidence 가져오기
    if result.keypoints.conf is not None:
        kp_conf = result.keypoints.conf.cpu().numpy()
    else:
        # 관절 Confidence 정보가 없을 때 대체값 사용
        kp_conf = np.ones(
            xy.shape[:2],
            dtype=float,
        )

    # ⑧ BBox와 Skeleton이 표시된 결과 이미지 저장
    output_image = (
        ROOT
        / "reports"
        / "day02"
        / "images"
        / f"{source.stem}_pose.jpg"
    )

    # 결과 저장 폴더 생성
    output_image.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    # YOLO Pose 결과를 이미지로 저장
    saved = cv2.imwrite(
        str(output_image),
        result.plot(),
    )

    if not saved:
        raise RuntimeError(
            f"결과 이미지를 저장하지 못했습니다: {output_image}"
        )

    # ⑨ CSV 저장 경로와 컬럼 이름 준비
    output_csv = (
        ROOT
        / "reports"
        / "day02"
        / "tables"
        / f"{source.stem}_keypoints.csv"
    )

    output_csv.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    # CSV에 저장할 데이터 항목 정의
    fieldnames = [
        "person_id",
        "box_conf",
        "keypoint_id",
        "keypoint_name",
        "pixel_x",
        "pixel_y",
        "normalized_x",
        "normalized_y",
        "keypoint_conf",
        "quality",
    ]

    # CSV에 기록할 데이터 목록
    rows = []

    # ⑩ 검출된 사람별 BBox 정보 확인
    person_count = min(
        len(boxes_xyxy),
        len(xy),
    )

    print(f"Detected people: {person_count}")
    print(f"Image size: {width} x {height}")

    # 사람을 한 명씩 처리
    for person_index in range(person_count):

        # 해당 사람의 BBox 좌표
        x1, y1, x2, y2 = boxes_xyxy[person_index]

        print()
        print(f"Person {person_index + 1}")

        print(
            "BBox:",
            f"({x1:.1f}, {y1:.1f})",
            f"({x2:.1f}, {y2:.1f})",
        )

        print(
            f"BBox Confidence: "
            f"{boxes_conf[person_index]:.3f}"
        )

        # ⑪ 관절별 좌표 정규화 및 Confidence 품질 판정
        # 한 사람의 17개 Keypoint를 순서대로 처리
        for keypoint_id, name in enumerate(
            COCO_KEYPOINT_NAMES
        ):

            # 관절의 원본 Pixel 좌표
            x, y = xy[
                person_index,
                keypoint_id,
            ]

            # 해당 관절의 Confidence
            confidence = kp_conf[
                person_index,
                keypoint_id,
            ]

            # Pixel x 좌표를 이미지 너비로 나누기
            normalized_x = (
                float(x) / width
                if width > 0
                else 0.0
            )

            # Pixel y 좌표를 이미지 높이로 나누기
            normalized_y = (
                float(y) / height
                if height > 0
                else 0.0
            )

            # Confidence 기준으로 관절 품질 구분
            quality = confidence_state(
                float(confidence),
                valid_threshold=args.kp_conf,
                warning_threshold=0.30,
            )

            # CSV 한 행에 저장할 관절 정보
            row = {
                "person_id": person_index + 1,
                "box_conf": round(
                    float(boxes_conf[person_index]),
                    4,
                ),
                "keypoint_id": keypoint_id,
                "keypoint_name": name,
                "pixel_x": round(float(x), 2),
                "pixel_y": round(float(y), 2),
                "normalized_x": round(
                    normalized_x,
                    5,
                ),
                "normalized_y": round(
                    normalized_y,
                    5,
                ),
                "keypoint_conf": round(
                    float(confidence),
                    4,
                ),
                "quality": quality,
            }

            # CSV 저장용 목록에 관절 정보 추가
            rows.append(row)

            # 터미널에 관절 분석 결과 출력
            print(
                f"{keypoint_id:02d} "
                f"{name:16s} "
                f"pixel=({x:7.1f}, {y:7.1f}) "
                f"norm=({normalized_x:.4f}, {normalized_y:.4f}) "
                f"conf={confidence:.3f} "
                f"{quality}"
            )

    # ⑫ 관절 분석 결과를 CSV 파일로 저장
    with output_csv.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:

        writer = csv.DictWriter(
            file,
            fieldnames=fieldnames,
        )

        # CSV 첫 줄에 컬럼 이름 저장
        writer.writeheader()

        # 사람별·관절별 데이터 저장
        writer.writerows(rows)

    # 최종 결과 파일 경로 출력
    print()
    print(f"Pose image: {output_image}")
    print(f"Keypoint CSV: {output_csv}")


# 프로그램 실행 시작
if __name__ == "__main__":
    main()

```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ argparse → 터미널 입력값 받기
   ├─ csv      → 관절 데이터를 CSV로 저장
   ├─ Path     → 파일 경로 관리
   ├─ cv2      → 이미지 읽기 및 저장
   ├─ numpy    → 관절 좌표 데이터 처리
   └─ YOLO     → Pose 모델 실행
                ↓
② 프로젝트 경로와 공통 함수 준비
   ├─ ROOT → 프로젝트 최상위 폴더
   ├─ common.py → 설정 읽기 및 GPU/CPU 선택
   └─ pose_utils.py → 관절 이름과 품질 기준
                ↓
③ 터미널 실행 옵션 준비
   ├─ --source → 분석할 이미지 경로
   │             기본값: front.jpg
   └─ --kp-conf → VALID 판단 기준
                  기본값: 0.50
                ↓
④ 입력 이미지 확인
   ├─ 이미지 파일이 존재하는가?
   │    ├─ 아니요 → 오류 발생
   │    └─ 예 → OpenCV로 이미지 읽기
   └─ 이미지 Width · Height 확인
                ↓
⑤ YOLO Pose 추론 실행
   ├─ project.json 설정 읽기
   ├─ GPU 또는 CPU 선택
   ├─ yolo11n-pose.pt 모델 로딩
   └─ 이미지에서 사람과 관절 추론
                ↓
⑥ 사람과 관절 검출 여부 확인
   ├─ 검출 결과가 있는가?
   │    ├─ 아니요 → 안내 출력 후 종료
   │    └─ 예 → 다음 단계
                ↓
⑦ YOLO Pose 예측 데이터 추출
   ├─ BBox 좌표
   ├─ 사람 검출 Confidence
   ├─ 관절별 Pixel x, y
   └─ 관절별 Confidence
                ↓
⑧ Pose 결과 이미지 저장
   ├─ BBox 표시
   ├─ Keypoint 및 Skeleton 표시
   └─ reports/day02/images/
      front_pose.jpg 저장
                ↓
⑨ CSV 저장 준비
   ├─ 저장 경로 지정
   ├─ CSV 컬럼 10개 정의
   └─ 관절 데이터 목록(rows) 준비
                ↓
⑩ 사람별 BBox 정보 확인
   ├─ Person 1
   │    ├─ BBox 좌표
   │    └─ BBox Confidence
   └─ 사람이 여러 명이면 각각 처리
                ↓
⑪ 관절별 데이터 계산 및 품질 판정
   ├─ 17개 Keypoint를 하나씩 확인
   ├─ Pixel x, y 좌표 가져오기
   ├─ Normalized 좌표 계산
   │    ├─ x / 이미지 Width
   │    └─ y / 이미지 Height
   ├─ 관절 Confidence 확인
   ├─ Confidence 기준으로 품질 구분
   │    ├─ 0.50 이상 → VALID
   │    ├─ 0.30~0.50 미만 → WARNING
   │    └─ 0.30 미만 → LOW
   └─ 관절 정보를 rows 목록에 추가
                ↓
⑫ CSV 파일로 결과 저장
   ├─ CSV 첫 줄에 컬럼 이름 저장
   ├─ 각 관절의 좌표와 Confidence 저장
   ├─ Quality 판정 결과 저장
   └─ reports/day02/tables/
      front_keypoints.csv 저장
```

---

# 11. 정면 이미지의 Keypoint를 숫자로 확인합니다

먼저 가장 단순한 `front.jpg`부터 분석합니다.

```bash
python scripts/07_pose_image_detail.py \
  --source data/day02/images/front.jpg
```

정상적으로 실행되면 터미널에 실제 이미지의 크기와 사람 검출 결과, COCO 17개 관절의 좌표와 Confidence가 출력됩니다.

이번 `front.jpg`의 실제 실행 결과는 다음과 같습니다.

```text
Detected people: 1
Image size: 2552 x 4000

Person 1
BBox: (847.9, 299.9) (1820.9, 3443.4)
BBox Confidence: 0.923

00 nose             pixel=(1308.2,  518.9) norm=(0.5126, 0.1297) conf=0.996 VALID
01 left_eye         pixel=(1374.9,  467.5) norm=(0.5387, 0.1169) conf=0.989 VALID
02 right_eye        pixel=(1252.3,  462.4) norm=(0.4907, 0.1156) conf=0.978 VALID
03 left_ear         pixel=(1467.0,  539.1) norm=(0.5749, 0.1348) conf=0.934 VALID
04 right_ear        pixel=(1175.9,  529.3) norm=(0.4608, 0.1323) conf=0.743 VALID
...
09 left_wrist       pixel=(1744.0, 1822.9) norm=(0.6834, 0.4557) conf=0.993 VALID
...
15 left_ankle       pixel=(1437.8, 3177.7) norm=(0.5634, 0.7944) conf=0.991 VALID
16 right_ankle      pixel=(1300.9, 3182.9) norm=(0.5097, 0.7957) conf=0.991 VALID
```

먼저 위 결과에서 다음 내용을 확인합니다.

```
Detected people: 1
→ 이미지에서 사람 1명을 찾았습니다.

Image size: 2552 x 4000
→ 현재 front.jpg의 실제 해상도입니다.

BBox Confidence: 0.923
→ 검출된 영역에 사람이 있다고 모델이 높은 Confidence로 판단했습니다.

17개 Keypoint
→ nose부터 right_ankle까지
   COCO 17개 관절의 위치가 출력되었습니다.
```

이번 `front.jpg`에서는 17개 Keypoint의 Confidence가 모두 `0.50` 이상이므로 오늘 정한 교육용 기준에서는 모두 `VALID`로 표시되었습니다.

이 결과만 보고 모든 이미지에서 Pose가 항상 잘 검출된다고 판단하면 안 됩니다.  
이후 `side`, `occlusion`, `partial_body`, `distance_far` 이미지에서도 같은 값을 확인하여 
촬영 조건에 따라 Keypoint Confidence가 어떻게 달라지는지 비교합니다.

결과 파일도 함께 생성되었습니다.

```
reports/day02/images/front_pose.jpg
reports/day02/tables/front_keypoints.csv
```

파일을 확인합니다.

```bash
ls -lh \
reports/day02/images/front_pose.jpg \
reports/day02/tables/front_keypoints.csv
```

CSV의 앞부분을 확인합니다.

```bash
head -10 reports/day02/tables/front_keypoints.csv
```

Windows 탐색기에서 결과 이미지를 확인할 수도 있습니다.

```bash
explorer.exe "$(wslpath -w reports/day02/images)"
```

vscode에서 직접 경로 따라 확인해도 됩니다.

---

# 12. CSV 한 줄이 무엇을 의미하는지 실제 결과로 읽어봅니다

`front_keypoints.csv`에는 **관절 하나의 분석 결과가 한 줄씩 저장**됩니다.

주요 열은 다음과 같습니다.

|열|의미|
|---|---|
|`person_id`|이미지에서 몇 번째 사람인지 표시|
|`box_conf`|사람 BBox의 Confidence|
|`keypoint_id`|COCO 17 Keypoint 번호|
|`keypoint_name`|관절 이름|
|`pixel_x`, `pixel_y`|원본 이미지에서의 실제 Pixel 위치|
|`normalized_x`, `normalized_y`|이미지 너비와 높이를 기준으로 0~1 범위로 바꾼 위치|
|`keypoint_conf`|해당 관절 위치에 대한 모델의 Confidence|
|`quality`|오늘의 기준에 따른 `VALID / WARNING / LOW`|

이번에는 실제 실행 결과의 **왼쪽 손목(`left_wrist`)**을 읽어봅니다.

```
keypoint_id   = 9
keypoint_name = left_wrist

pixel_x       = 1744.0
pixel_y       = 1822.9

normalized_x  = 0.6834
normalized_y  = 0.4557

keypoint_conf = 0.993
quality       = VALID
```

이 한 줄을 순서대로 해석하면 다음과 같습니다.

```
COCO 17의 9번 관절
left_wrist를 찾음
        ↓
원본 2552 × 4000 이미지에서

x = 1744.0
y = 1822.9
부근에 위치
        ↓
이미지 크기를 기준으로
0~1 범위로 바꾸면

x = 0.6834
y = 0.4557
        ↓
Keypoint Confidence
0.993
        ↓
0.50 이상이므로
오늘 기준에서는 VALID
```

실제로 Normalized 좌표가 어떻게 만들어졌는지도 확인할 수 있습니다.

```
normalized_x
= 1744.0 / 2552
≈ 0.6834

normalized_y
= 1822.9 / 4000
≈ 0.4557
```

즉 같은 손목 위치를 두 가지 방식으로 표현한 것입니다.

```
Pixel 좌표
(1744.0, 1822.9)

→ 원본 이미지에서 실제 위치

Normalized 좌표
(0.6834, 0.4557)

→ 이미지 크기를 0~1 기준으로 바꾼 상대 위치
```

그리고 `keypoint_conf = 0.993`은 다음처럼 이해합니다.

```
0.993
        ↓
모델이 이 위치를
left_wrist라고
매우 높은 Confidence로 판단
        ↓
오늘 기준 0.50 이상
        ↓
VALID
```

> **주의:** `0.993`을 “왼쪽 손목일 확률이 정확히 99.3%”라고 해석하지 않습니다.  
> 이 값은 모델이 예측한 관절 위치의 신뢰 정도를 서로 비교하기 위한 Confidence 값으로 사용합니다.

이제 `front.jpg` 한 장을 이용하여 **Keypoint 좌표와 Confidence를 읽는 방법**을 확인했습니다.

뒤의 PART 5에서는 지금 배운 방법을 6장의 이미지에 자동으로 적용하여 촬영 조건별 Pose 품질을 한 번에 비교합니다.

```
지금까지
front.jpg 한 장
        ↓
Keypoint 좌표
Confidence
Pixel / Normalized 좌표
읽는 방법 확인

        ↓

PART 5에서

front
arms_up
side
partial_body
occlusion
distance_far
        ↓
같은 YOLO Pose로 자동 분석
        ↓
BBox Confidence
BBox Area Ratio
평균 Keypoint Confidence
LOW 관절 수
        ↓
촬영 조건별 Pose 품질 비교
```

따라서 여기서는 한 이미지의 결과를 읽는 방법까지만 이해하면 됩니다.  
6장의 조건별 비교는 뒤의 PART 5에서 한 번에 수행합니다.

---

# PART 4. Confidence Threshold에 따라 Skeleton이 어떻게 달라지는지 확인합니다

---

# 13. Confidence가 낮은 관절을 제외하여 Skeleton을 다시 그립니다

앞에서는 YOLO Pose가 검출한 관절을 모두 표시했습니다.

이번에는 **관절의 Confidence가 일정 기준 이상일 때만 점과 Skeleton 선을 그리는 코드**를 작성하겠습니다.

### ① 관절 Confidence 기준은 언제 정했나요?

앞에서 작성한 `src/pose_utils.py`에서 다음과 같이 구분했습니다.

|Confidence|판정|의미|
|---|---|---|
|0.50 이상|VALID|상대적으로 신뢰도가 높은 관절|
|0.30 이상 ~ 0.50 미만|WARNING|신뢰도가 중간인 관절|
|0.30 미만|LOW|신뢰도가 낮은 관절|

이 수치는 YOLO의 공식 기준이 아니라 **우리가 Confidence 차이를 비교하도록 정한 교육용 기준**입니다.

### ② 이번 코드에서는 어떻게 사용하나요?

예를 들어 **Confidence가 0.50 이상인 관절만 표시**한다고 가정하겠습니다.

- 왼쪽 어깨 Confidence `0.90` → 점 표시
    
- 왼쪽 팔꿈치 Confidence `0.80` → 점 표시
    
- 왼쪽 손목 Confidence `0.25` → 점 표시하지 않음
    

Skeleton 선은 연결된 **두 관절이 모두 0.50 이상일 때만** 그립니다.

따라서 어깨와 팔꿈치를 연결하는 선은 표시되지만, 팔꿈치와 손목을 연결하는 선은 표시되지 않습니다.

### ③ 왜 이렇게 그리나요?

Confidence가 낮은 관절까지 모두 연결하면 모델이 확신하지 못하는 위치도 정상적인 Skeleton처럼 보일 수 있습니다.

따라서 **신뢰도가 낮은 관절을 시각화에서 제외하여 어떤 관절의 검출이 불안정한지 쉽게 확인**하는 것이 목적입니다.

관절을 제외한다고 YOLO 모델의 예측 결과가 바뀌는 것은 아닙니다.

### ④ 앞에서 작성한 어떤 파일과 연결되나요?

|파일|역할|
|---|---|
|`configs/project.json`|YOLO 모델과 기본 설정 관리|
|`src/common.py`|설정값 읽기 및 GPU·CPU 선택|
|`src/pose_utils.py`|17개 관절 이름, Skeleton 연결 관계, Confidence 기준 정의|
|`scripts/07_pose_image_detail.py`|관절 좌표와 Confidence를 CSV로 저장|
|`scripts/08_draw_skeleton_quality.py`|Confidence 기준을 적용하여 Skeleton을 다시 그림|

### ⑤ 코드의 전체 처리 흐름

```
① 이미지 입력
        ↓
② YOLO Pose 실행
        ↓
③ 17개 관절의 좌표와 Confidence 확인
        ↓
④ Confidence 0.50 이상인가?
   ├─ 예 → 관절 점 표시
   └─ 아니요 → 점 표시하지 않음
        ↓
⑤ 연결된 두 관절이 모두 0.50 이상인가?
   ├─ 예 → Skeleton 선 그리기
   └─ 아니요 → 선 그리지 않음
        ↓
⑥ 결과 이미지 저장
```

**파일:** `scripts/08_draw_skeleton_quality.py`

이번 실습의 핵심은 **YOLO가 찾은 관절을 모두 표시하는 것과 Confidence 기준을 적용해 표시하는 것의 차이를 비교하는 것**입니다.

이를 통해 어떤 관절이 상대적으로 신뢰도가 낮은지 눈으로 확인할 수 있습니다.

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
from src.pose_utils import (
    COCO_SKELETON,
)


# ③ 분석할 이미지와 Keypoint Confidence 기준 설정
def parse_args():
    parser = argparse.ArgumentParser()

    # 분석할 이미지 경로 (기본값: front.jpg)
    parser.add_argument(
        "--source",
        default=str(
            ROOT
            / "data"
            / "day02"
            / "images"
            / "front.jpg"
        ),
    )

    # 관절을 표시할 최소 Confidence (기본값: 0.50)
    parser.add_argument(
        "--kp-conf",
        type=float,
        default=0.50,
    )

    return parser.parse_args()


def main() -> None:

    # ④ 이미지 파일 확인 및 OpenCV로 읽기
    args = parse_args()

    source = Path(args.source)

    if not source.exists():
        raise FileNotFoundError(
            f"이미지가 없습니다: {source}"
        )

    image = cv2.imread(str(source))

    if image is None:
        raise RuntimeError(
            f"이미지를 읽지 못했습니다: {source}"
        )

    # ⑤ 프로젝트 설정 읽기 및 YOLO Pose 추론
    config = load_config()
    device = resolve_device(config["device"])

    model = YOLO(config["pose_model"])

    results = model.predict(
        source=image,
        imgsz=config["image_size"],
        conf=config["person_confidence"],
        device=device,
        verbose=False,
    )

    result = results[0]

    # ⑥ Keypoint 검출 여부 확인
    if (
        result.keypoints is None
        or result.keypoints.xy is None
        or len(result.keypoints.xy) == 0
    ):
        print("Keypoint를 찾지 못했습니다.")
        return

    # ⑦ 관절 좌표 및 Confidence 가져오기
    # 관절별 Pixel x, y 좌표
    xy = result.keypoints.xy.cpu().numpy()

    # 관절별 Confidence
    if result.keypoints.conf is not None:
        confidence = (
            result.keypoints.conf
            .cpu()
            .numpy()
        )
    else:
        # Confidence 정보가 없으면 대체값 1.0 사용
        confidence = np.ones(
            xy.shape[:2],
            dtype=float,
        )

    # 원본 이미지 복사 (그림을 그릴 이미지)
    output = image.copy()

    # ⑧ 사람의 Bounding Box 그리기
    if result.boxes is not None:
        boxes = (
            result.boxes.xyxy
            .cpu()
            .numpy()
        )

        # 검출된 사람의 BBox를 하나씩 그림
        for box in boxes:
            x1, y1, x2, y2 = box.astype(int)

            cv2.rectangle(
                output,
                (x1, y1),
                (x2, y2),
                (255, 255, 255),
                2,
            )

    # ⑨ Confidence 기준을 적용하여 Skeleton 선 그리기
    for person_index in range(len(xy)):
        points = xy[person_index]
        confs = confidence[person_index]

        # COCO_SKELETON에 정의된 관절 연결 관계 확인
        for start_id, end_id in COCO_SKELETON:

            # 두 관절 모두 Confidence 기준 이상인가?
            if (
                confs[start_id] >= args.kp_conf
                and confs[end_id] >= args.kp_conf
            ):

                # 연결할 두 관절의 좌표 가져오기
                start = tuple(
                    points[start_id].astype(int)
                )

                end = tuple(
                    points[end_id].astype(int)
                )

                # 두 관절을 초록색 선으로 연결
                cv2.line(
                    output,
                    start,
                    end,
                    (0, 255, 0),
                    2,
                )

        # ⑩ Confidence 기준 이상인 관절 점과 번호 표시
        for keypoint_id, (point, conf) in enumerate(
            zip(points, confs)
        ):

            # 기준보다 Confidence가 낮으면 표시하지 않음
            if conf < args.kp_conf:
                continue

            # 관절의 Pixel 좌표
            x, y = point.astype(int)

            # 관절 위치에 노란색 점 표시
            cv2.circle(
                output,
                (x, y),
                5,
                (0, 255, 255),
                -1,
            )

            # 관절 번호(0~16)를 점 옆에 표시
            cv2.putText(
                output,
                str(keypoint_id),
                (x + 6, y - 6),
                cv2.FONT_HERSHEY_SIMPLEX,
                0.45,
                (255, 255, 255),
                1,
            )

    # ⑪ Confidence 기준을 파일명에 넣어 결과 이미지 저장
    threshold_name = int(
        args.kp_conf * 100
    )

    output_path = (
        ROOT
        / "reports"
        / "day02"
        / "images"
        / (
            f"{source.stem}_"
            f"skeleton_conf{threshold_name:02d}.jpg"
        )
    )

    # 저장할 폴더 생성
    output_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    # Skeleton 결과 이미지 저장
    saved = cv2.imwrite(
        str(output_path),
        output,
    )

    if not saved:
        raise RuntimeError(
            f"결과를 저장하지 못했습니다: {output_path}"
        )

    print(f"Saved: {output_path}")


# 프로그램 실행 시작
if __name__ == "__main__":
    main()
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ argparse → 터미널 실행 옵션 받기
   ├─ Path     → 이미지 경로 관리
   ├─ cv2      → 이미지 읽기 및 그림 그리기
   ├─ numpy    → 관절 좌표 처리
   └─ YOLO     → Pose 모델 실행
                ↓
② 프로젝트 경로 및 공통 함수 준비
   ├─ ROOT → 프로젝트 최상위 폴더
   ├─ common.py → 설정 읽기·GPU/CPU 선택
   └─ pose_utils.py → Skeleton 연결 정보
                ↓
③ 이미지 및 Confidence 기준 설정
   ├─ --source → 분석할 이미지
   │             기본값: front.jpg
   └─ --kp-conf → 관절 표시 최소 신뢰도
                  기본값: 0.50
                ↓
④ 이미지 파일 확인
   ├─ 이미지가 존재하는가?
   │    ├─ 아니요 → 오류 발생
   │    └─ 예 → OpenCV로 이미지 읽기
                ↓
⑤ YOLO Pose 추론
   ├─ project.json 설정 불러오기
   ├─ GPU 또는 CPU 선택
   ├─ YOLO Pose 모델 로딩
   └─ 사람과 17개 관절 위치 추정
                ↓
⑥ Keypoint 검출 여부 확인
   ├─ 관절을 찾았는가?
   │    ├─ 아니요 → 안내 출력 후 종료
   │    └─ 예 → 다음 단계
                ↓
⑦ 관절 좌표와 Confidence 추출
   ├─ 관절별 x, y 좌표 가져오기
   ├─ 관절별 Confidence 가져오기
   └─ 원본 이미지를 복사하여 출력용 준비
                ↓
⑧ 사람 Bounding Box 표시
   ├─ 사람의 BBox 좌표 읽기
   └─ 원본 이미지에 흰색 사각형 그리기
                ↓
⑨ Confidence 기준으로 Skeleton 선 그리기
   ├─ 연결할 두 관절 확인
   ├─ 두 관절 모두 0.50 이상인가?
   │    ├─ 예 → 초록색 선 그리기
   │    └─ 아니요 → 선을 그리지 않음
   └─ 모든 관절 연결 관계 반복
                ↓
⑩ Confidence 기준으로 관절 점 표시
   ├─ 관절 Confidence가 0.50 이상인가?
   │    ├─ 예 → 노란색 점과 번호 표시
   │    └─ 아니요 → 표시하지 않음
   └─ 17개 관절 반복
                ↓
⑪ 최종 Skeleton 이미지 저장
   ├─ Confidence 기준을 파일명에 반영
   ├─ reports/day02/images/ 폴더 생성
   └─ front_skeleton_conf50.jpg 저장
```

---

# 14. 08번 코드의 실제 Keypoint Confidence를 확인하고 Threshold를 적용해 봅니다

앞에서 Pose 모델은 각 관절마다 **Keypoint Confidence**를 출력한다는 것을 확인했습니다.

이번에는 이 Confidence를 이용하여 다음 질문에 답해 봅니다.

```
모델이 찾은 모든 관절을
그대로 사용해도 될까?

        ↓

Confidence가 낮은 관절은
제외해야 할까?
```

이때 사용하는 기준이 **Threshold**입니다.

```
Keypoint Confidence >= Threshold
→ 관절 사용

Keypoint Confidence < Threshold
→ 관절 제외
```

예를 들어 손목의 Confidence가 `0.52`라면:

```
Threshold 0.30
→ 0.52 >= 0.30
→ 사용

Threshold 0.50
→ 0.52 >= 0.50
→ 사용

Threshold 0.70
→ 0.52 < 0.70
→ 제외
```

즉 Threshold를 높이면 Confidence가 낮은 관절부터 사용 대상에서 제외됩니다.

> Threshold는 Pose를 다시 계산하는 값이 아닙니다.  
> **이미 찾은 관절 중 어느 정도 Confidence 이상의 관절을 사용할 것인지 정하는 기준**입니다.

기존 원리 설명 그림은 여기에서 그대로 확인합니다.

![[Group 22 (1).png]]

---

## 14.1 먼저 `front_keypoints.csv`의 실제 Confidence를 확인합니다

이번에는 임의로 Threshold를 정하지 않고, 앞에서 만든 실제 결과를 먼저 확인합니다.

`front_keypoints.csv`에는 17개 관절의 Confidence가 저장되어 있습니다.

Confidence가 낮은 순서부터 확인합니다.

```bash
python - <<'PY'
import pandas as pd

path = "reports/day02/tables/front_keypoints.csv"
df = pd.read_csv(path)

print(
    df[
        ["keypoint_name", "keypoint_conf"]
    ]
    .sort_values("keypoint_conf")
    .to_string(index=False)
)
PY
```

실제 결과에서 낮은 값부터 보면 다음과 같습니다.

```
right_ear     0.7428
left_ear      0.9340
right_eye     0.9783
right_wrist   0.9882
left_eye      0.9887
right_elbow   0.9898
...
```

가장 낮은 관절은:

```
right_ear
Confidence = 0.7428
```

입니다.

반대로 대부분의 관절은 `0.98 ~ 1.00`에 가까운 높은 Confidence를 가지고 있습니다.

따라서 이번 이미지에서는 Threshold를 다음처럼 정해 보는 것이 좋습니다.

```
0.70
→ 가장 낮은 0.7428보다 낮은 기준
→ 17개 관절 모두 사용될 것으로 예상

0.90
→ 0.7428보다 높은 기준
→ right_ear가 제외될 것으로 예상

0.95
→ 0.9340보다도 높은 기준
→ right_ear와 left_ear가 제외될 것으로 예상
```

> `0.70`, `0.90`, `0.95`는 공식적인 정답값이 아닙니다.  
> 이번 `front.jpg`의 실제 Confidence를 보고 Threshold의 동작을 눈으로 확인하기 위해 선택한 실험값입니다.

---

## 14.2 실제 Confidence를 기준으로 Threshold를 바꾸어 봅니다

입력 이미지는 모두 같은 `front.jpg`입니다.

바꾸는 것은 **Threshold 하나뿐**입니다.

### Threshold 0.70

```bash
python scripts/08_draw_skeleton_quality.py \
  --source data/day02/images/front.jpg \
  --kp-conf 0.70
```

![[Pasted image 20261104142320.png|209]]

예상 결과:

```
가장 낮은 Confidence = 0.7428

0.7428 >= 0.70
        ↓
17개 관절 모두 사용
```

결과 파일:

```
front_skeleton_conf70.jpg
```

---
### Threshold 0.90

```bash
python scripts/08_draw_skeleton_quality.py \
  --source data/day02/images/front.jpg \
  --kp-conf 0.90
```

![[Pasted image 20261104142357.png|205]]

`right_ear`를 확인합니다.

```
right_ear = 0.7428

0.7428 < 0.90
        ↓
right_ear 제외
```

따라서 `right_ear`의 점과 이 관절을 이용하는 Skeleton 연결선이 표시되지 않습니다.

결과 파일:

```
front_skeleton_conf90.jpg
```

---
### Threshold 0.95

```bash
python scripts/08_draw_skeleton_quality.py \
  --source data/day02/images/front.jpg \
  --kp-conf 0.95
```

![[Pasted image 20261104142440.png|233]]

이번에는 다음 두 관절이 기준보다 낮습니다.

```
right_ear = 0.7428
left_ear  = 0.9340

둘 다 < 0.95
```

따라서 양쪽 귀 Keypoint와 해당 관절을 사용하는 Skeleton 연결선이 제외됩니다.

결과 파일:

```
front_skeleton_conf95.jpg
```

세 결과를 함께 확인합니다.

```
explorer.exe "$(wslpath -w reports/day02/images)"
```

---

## 14.3 실제 결과를 비교합니다

이번 실험에서는 Threshold를 높였을 때 실제로 다음 변화가 나타나는지 확인합니다.

![[{45F95BDA-812F-4D39-9739-6FC84310CA96}.png]]

```
Threshold 0.70
→ 17개 관절 사용

        ↓

Threshold 0.90
→ right_ear 제외

        ↓

Threshold 0.95
→ right_ear
→ left_ear
   제외
```

즉 이번에는 앞의 `0.30 / 0.50 / 0.70` 실험과 달리 **실제 Confidence 값 사이에 Threshold를 설정했기 때문에 관절이 제외되는 모습을 눈으로 확인할 수 있습니다.**

```
Confidence 확인
        ↓
Threshold 설정
        ↓
기준보다 낮은 관절 찾기
        ↓
해당 관절 제외
        ↓
Skeleton 변화 확인
```

---

## 14.4 그런데 왜 Confidence가 낮은 관절을 제외하나요?

예를 들어 사람이 한쪽 팔을 몸 뒤로 숨겼다고 생각해 봅니다.

![[Pasted image 20260916183714.png]]

카메라에서는 손목이 잘 보이지 않습니다.

```
실제 상황

손목이 몸 뒤에 가려짐
        ↓
카메라에서 손목 위치를
정확히 확인하기 어려움
```

그래도 Pose 모델은 주변의 어깨·팔꿈치 위치를 이용하여 손목 위치를 **추정해서 표시할 수 있습니다.**

예를 들어:

```python
right_wrist Confidence = 0.38
```

이라고 나왔다고 가정합니다.

화면에는 손목 점이 보일 수 있지만, 모델도 그 위치를 충분히 확신하지 못하고 있다는 뜻입니다.

이 좌표를 그대로 사용하면:

```
실제 손목 위치와 다른 좌표
        ↓
잘못된 Skeleton 연결
        ↓
잘못된 관절 위치 사용
        ↓
다음 수업의 관절각도 계산에도
잘못된 값이 들어갈 수 있음
```

과 같은 문제가 생길 수 있습니다.

그래서 Threshold를 사용하여 **Confidence가 너무 낮은 관절을 그대로 사용하지 않도록 걸러냅니다.**

```
Wrist Confidence = 0.38

Threshold = 0.50

0.38 < 0.50
        ↓
이번 분석에서는
Wrist를 사용하지 않음
```

즉 Threshold의 목적은:

```
Skeleton을 일부러 지우는 것 X

신뢰하기 어려운 관절 좌표가
다음 분석에 그대로 들어가는 것을
줄이는 것 O
```

입니다.

---

## 14.5 이번 `front.jpg` 결과에서 무엇을 배웠나요?

이번 `front.jpg`는 정면에서 전신이 잘 보이는 이미지입니다.

![[Pasted image 20260916184816.png]]

실제 Confidence를 보면:

```
가장 낮음
right_ear = 0.7428

그다음
left_ear = 0.9340

나머지 대부분
약 0.98 이상
```

이었습니다.

따라서 실제 결과에 맞게 Threshold를 바꾸면:

```
0.70
→ 모두 사용

0.90
→ 가장 낮은 right_ear 제외

0.95
→ right_ear + left_ear 제외
```

라는 변화를 확인할 수 있습니다.

이번 실습에서 기억할 것은 두 가지입니다.

```
Confidence
→ 모델이 예측한 관절 위치를
  얼마나 신뢰하는가?

Threshold
→ 그 Confidence를 보고
  어느 관절까지 사용할 것인가?
```

그리고 Threshold는 높을수록 좋은 값이 아닙니다.

이번 `front.jpg`의 `left_ear = 0.9340`이 실제로 올바른 위치에 잘 검출되어 있다고 하더라도 Threshold를 `0.95`로 설정하면 사용하지 않게 됩니다.

따라서:

```
Threshold가 너무 낮음
→ 신뢰하기 어려운 관절까지 사용할 수 있음

Threshold가 너무 높음
→ 사용할 수 있는 관절까지 제외할 수 있음
```

이 됩니다.

> **결론:** 
> 먼저 실제 Confidence 값을 확인하고, 분석 목적에 맞는 Threshold를 정해야 합니다. Threshold는 “가장 높은 값을 선택하는 것”이 아니라 신뢰하기 어려운 관절을 걸러내면서 필요한 관절은 남기기 위한 기준입니다.

---

# 15. Threshold 실험값과 실제 비교 기준을 구분합니다

14번에서는 Threshold가 실제로 어떻게 관절을 제외하는지 눈으로 확인하기 위해 `front.jpg`의 Confidence 값에 맞춰 다음 값을 사용했습니다.

```
0.70
0.90
0.95
```

이 값들은 Threshold의 동작 원리를 확인하기 위한 실험값입니다.

```
14번

실제 Confidence 확인
        ↓
Threshold를 바꿈
        ↓
어떤 관절이 제외되는지
Skeleton으로 확인
```

하지만 이제부터는 목적이 달라집니다.

PART 5에서는 여러 촬영 조건의 Pose 품질을 **같은 기준으로 비교**해야 합니다.

따라서 기본 비교 기준은 다시 다음과 같이 사용합니다.

```
Keypoint Confidence < 0.50
→ LOW 관절

Keypoint Confidence >= 0.50
→ 기본 분석에 사용
```

즉 두 Threshold는 역할이 다릅니다.

```
14번의 0.70 / 0.90 / 0.95
→ Threshold 원리를 눈으로 확인하기 위한 실험

이후의 0.50
→ 여러 이미지의 Pose 품질을
   같은 기준으로 비교하기 위한 기준
```

> `0.50`도 모든 Pose 프로젝트에서 사용하는 절대적인 정답값은 아닙니다.  
> 이번 수업에서는 여러 촬영 조건을 **동일한 기준으로 비교하기 위해 사용하는 교육용 기준값**입니다.

이제 다음 단계에서는 이 `0.50` 기준을 동일하게 적용하여 다음 6장의 Pose 품질을 비교합니다.

```
front
arms_up
side
partial_body
occlusion
distance_far
        ↓
같은 기준으로 분석
        ↓
평균 Keypoint Confidence
LOW 관절 수
가장 낮은 관절
        ↓
촬영 조건별 Pose 품질 비교
```
---

# PART 5. 여러 촬영 조건의 Pose 품질을 한 번에 비교합니다

---

# 16. 여러 이미지를 자동으로 비교하는 코드를 작성합니다

이미지가 많아지면 한 장씩 터미널 결과를 읽고 기록하기 어렵습니다.

이번에는 `data/day02/images/`의 이미지를 모두 분석하여 한 개의 비교표를 만듭니다.

비교할 핵심 값은 다음과 같습니다.

```text
사람 수
BBox Confidence
BBox Area Ratio
평균 Keypoint Confidence
낮은 Confidence 관절 수
가장 낮은 관절 이름
가장 낮은 관절 Confidence
```

`BBox Area Ratio`는 전체 이미지에서 사람 BBox가 차지하는 비율입니다.

```text
BBox Area Ratio가 큼
→ 화면에서 사람이 상대적으로 크게 보임

BBox Area Ratio가 작음
→ 화면에서 사람이 상대적으로 작게 보임
```

### 이 코드는 왜 작성하나요?

```text
이미지 1장씩 확인
        ↓
조건이 많아지면 비교하기 어려움
        ↓
같은 기준으로 자동 분석
        ↓
한 개의 CSV로 정리
        ↓
조건별 차이 비교
```

### 의사코드로 읽어보기

```text
Day02 이미지 목록을 가져온다
        ↓
각 이미지에 YOLO Pose 실행
        ↓
사람이 여러 명이면
BBox Confidence가 가장 높은 사람을
대표 사람으로 선택
        ↓
대표 사람 BBox 크기 계산
        ↓
17개 Keypoint Confidence 평균 계산
        ↓
0.50 미만 관절 수 계산
        ↓
가장 Confidence가 낮은 관절 찾기
        ↓
결과 이미지 저장
        ↓
모든 결과를 CSV 한 개로 저장
```

**파일: `scripts/09_compare_pose_images.py`**

```python
from __future__ import annotations

# ① 필요한 라이브러리 불러오기
import csv
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
from src.pose_utils import (
    COCO_KEYPOINT_NAMES,
)


# ③ 분석할 이미지 파일 확장자 지정
IMAGE_EXTENSIONS = {
    ".jpg",
    ".jpeg",
    ".png",
}


def main() -> None:

    # ④ 프로젝트 설정 및 Keypoint Confidence 기준 읽기
    config = load_config()

    # GPU 또는 CPU 선택
    device = resolve_device(config["device"])

    # 관절 신뢰도 기준 (기본값 0.50)
    kp_threshold = config.get(
        "keypoint_confidence",
        0.50,
    )

    # ⑤ 입력 이미지 및 결과 저장 폴더 지정
    # 2일차 이미지가 저장된 폴더
    image_dir = (
        ROOT
        / "data"
        / "day02"
        / "images"
    )

    # Pose 결과 이미지를 저장할 폴더
    output_dir = (
        ROOT
        / "reports"
        / "day02"
        / "images"
        / "comparison"
    )

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    # 전체 이미지의 품질 비교표 CSV 경로
    table_path = (
        ROOT
        / "reports"
        / "day02"
        / "tables"
        / "image_quality_summary.csv"
    )

    table_path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    # ⑥ 분석할 이미지 목록 가져오기
    # 지정 폴더에서 JPG, JPEG, PNG 파일 찾기
    image_paths = sorted(
        path
        for path in image_dir.iterdir()
        if path.suffix.lower()
        in IMAGE_EXTENSIONS
    )

    # 이미지가 없으면 오류 발생
    if not image_paths:
        raise RuntimeError(
            f"이미지가 없습니다: {image_dir}"
        )

    # ⑦ YOLO Pose 모델 불러오기
    model = YOLO(config["pose_model"])

    # 이미지별 분석 결과를 저장할 목록
    rows = []

    # ⑧ 이미지를 한 장씩 읽고 크기 확인
    for image_path in image_paths:
        image = cv2.imread(str(image_path))

        # 이미지를 읽지 못하면 건너뛰기
        if image is None:
            print(f"Skip: {image_path}")
            continue

        # 이미지의 너비, 높이, 전체 면적 계산
        height, width = image.shape[:2]
        image_area = width * height

        # ⑨ 현재 이미지에 YOLO Pose 적용
        results = model.predict(
            source=image,
            imgsz=config["image_size"],
            conf=config["person_confidence"],
            device=device,
            verbose=False,
        )

        result = results[0]

        # 검출된 사람 수 확인
        people = (
            len(result.boxes)
            if result.boxes is not None
            else 0
        )

        # 분석 결과의 기본값 설정
        main_box_conf = 0.0
        bbox_area_ratio = 0.0
        mean_kp_conf = 0.0
        low_keypoints = 17
        lowest_name = "NO_PERSON"
        lowest_conf = 0.0

        # ⑩ 대표 사람의 BBox와 관절 품질 분석
        if (
            result.boxes is not None
            and result.keypoints is not None
            and result.keypoints.conf is not None
            and len(result.boxes) > 0
        ):

            # 사람별 BBox Confidence 가져오기
            box_conf = (
                result.boxes.conf
                .cpu()
                .numpy()
            )

            # 사람별 BBox 좌표 가져오기
            boxes = (
                result.boxes.xyxy
                .cpu()
                .numpy()
            )

            # 사람별 17개 관절 Confidence 가져오기
            kp_conf = (
                result.keypoints.conf
                .cpu()
                .numpy()
            )

            # BBox Confidence가 가장 높은 사람 선택
            main_index = int(
                np.argmax(box_conf)
            )

            # 대표 사람의 BBox Confidence
            main_box_conf = float(
                box_conf[main_index]
            )

            # 대표 사람의 BBox 좌표
            x1, y1, x2, y2 = boxes[main_index]

            # BBox 면적 계산
            bbox_area = max(
                0.0,
                float(x2 - x1),
            ) * max(
                0.0,
                float(y2 - y1),
            )

            # BBox 면적 / 전체 이미지 면적
            # 이미지에서 사람이 차지하는 비율
            if image_area > 0:
                bbox_area_ratio = (
                    bbox_area / image_area
                )

            # 대표 사람의 관절 Confidence
            conf = kp_conf[main_index]

            # 17개 관절의 평균 Confidence
            mean_kp_conf = float(
                np.mean(conf)
            )

            # 기준값보다 Confidence가 낮은 관절 수
            low_keypoints = int(
                np.sum(
                    conf < kp_threshold
                )
            )

            # Confidence가 가장 낮은 관절 번호
            lowest_id = int(
                np.argmin(conf)
            )

            # 해당 관절의 이름 확인
            lowest_name = (
                COCO_KEYPOINT_NAMES[
                    lowest_id
                ]
            )

            # 가장 낮은 관절 Confidence
            lowest_conf = float(
                conf[lowest_id]
            )

        # ⑪ BBox와 Skeleton 결과 이미지 저장
        output_path = (
            output_dir
            / f"{image_path.stem}_pose.jpg"
        )

        saved = cv2.imwrite(
            str(output_path),
            result.plot(),
        )

        if not saved:
            raise RuntimeError(
                f"결과 이미지를 저장하지 못했습니다: {output_path}"
            )

        # ⑫ 현재 이미지의 분석 결과를 비교표에 추가
        rows.append(
            {
                "image": image_path.name,

                "people": people,

                "main_box_conf": round(
                    main_box_conf,
                    4,
                ),

                "bbox_area_ratio": round(
                    bbox_area_ratio,
                    4,
                ),

                "mean_keypoint_conf": round(
                    mean_kp_conf,
                    4,
                ),

                "low_keypoint_count": (
                    low_keypoints
                ),

                "lowest_keypoint": (
                    lowest_name
                ),

                "lowest_keypoint_conf": round(
                    lowest_conf,
                    4,
                ),

                "keypoint_threshold": (
                    kp_threshold
                ),
            }
        )

        # 이미지별 분석 결과를 터미널에 출력
        print(
            image_path.name,
            "people=",
            people,
            "bbox_ratio=",
            f"{bbox_area_ratio:.3f}",
            "mean_kp=",
            f"{mean_kp_conf:.3f}",
            "low_kp=",
            low_keypoints,
        )

    # ⑬ 전체 이미지의 품질 비교 결과를 CSV로 저장
    with table_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:

        # CSV 컬럼 이름 지정
        writer = csv.DictWriter(
            file,
            fieldnames=[
                "image",
                "people",
                "main_box_conf",
                "bbox_area_ratio",
                "mean_keypoint_conf",
                "low_keypoint_count",
                "lowest_keypoint",
                "lowest_keypoint_conf",
                "keypoint_threshold",
            ],
        )

        # CSV 첫 줄에 컬럼 이름 저장
        writer.writeheader()

        # 이미지별 분석 결과 저장
        writer.writerows(rows)

    print()
    print(f"Summary saved: {table_path}")


# 프로그램 실행 시작
if __name__ == "__main__":
    main()
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ csv    → 이미지별 비교표 저장
   ├─ Path   → 파일 경로 관리
   ├─ cv2    → 이미지 읽기 및 저장
   ├─ numpy  → 평균·최솟값 등 계산
   └─ YOLO   → Pose 모델 실행
                ↓
② 프로젝트 경로와 공통 함수 준비
   ├─ ROOT → 프로젝트 최상위 폴더
   ├─ common.py → 설정 읽기·GPU/CPU 선택
   └─ pose_utils.py → COCO 관절 이름
                ↓
③ 분석할 이미지 확장자 지정
   └─ JPG · JPEG · PNG
                ↓
④ 프로젝트 설정 읽기
   ├─ project.json 설정 불러오기
   ├─ GPU 또는 CPU 선택
   └─ Keypoint Confidence 기준 확인
      기본값 0.50
                ↓
⑤ 입력 및 결과 저장 폴더 준비
   ├─ data/day02/images/
   │    → 분석할 이미지
   ├─ reports/day02/images/comparison/
   │    → Pose 결과 이미지
   └─ reports/day02/tables/
        → CSV 비교표
                ↓
⑥ 이미지 목록 가져오기
   ├─ 이미지 폴더 검색
   ├─ JPG·JPEG·PNG 파일 찾기
   └─ 이미지가 없으면 오류 발생
                ↓
⑦ YOLO Pose 모델 불러오기
   └─ yolo11n-pose.pt
                ↓
⑧ 이미지 한 장씩 읽기
   ├─ 이미지 읽기
   ├─ Width · Height 확인
   └─ 전체 이미지 면적 계산
                ↓
⑨ 이미지별 YOLO Pose 실행
   ├─ 사람 검출
   ├─ BBox 추출
   ├─ 17개 Keypoint 추정
   └─ 검출된 사람 수 확인
                ↓
⑩ 대표 사람의 Pose 품질 분석
   ├─ BBox Confidence가 가장 높은 사람 선택
   ├─ 사람 검출 Confidence 확인
   ├─ BBox 면적 비율 계산
   ├─ 17개 관절 평균 Confidence 계산
   ├─ 0.50 미만 관절 개수 계산
   └─ Confidence가 가장 낮은 관절 확인
                ↓
⑪ Pose 결과 이미지 저장
   ├─ 사람 BBox 표시
   ├─ Keypoint · Skeleton 표시
   └─ comparison/ 폴더에 JPG 저장
                ↓
⑫ 이미지별 분석 결과 기록
   ├─ 이미지 파일명
   ├─ 검출된 사람 수
   ├─ 사람 검출 Confidence
   ├─ BBox 면적 비율
   ├─ 평균 관절 Confidence
   ├─ 낮은 Confidence 관절 수
   └─ 가장 낮은 Confidence 관절
                ↓
   다음 이미지가 있으면 ⑧~⑫ 반복
                ↓
⑬ 전체 비교 결과 CSV 저장
   ├─ 모든 이미지의 분석 결과 합치기
   ├─ CSV 컬럼 이름 작성
   └─ image_quality_summary.csv 저장
```

> 2일차 기본 실험은 한 사람을 촬영하는 조건입니다. 여러 사람이 함께 찍힌 경우 `people` 값이 2 이상으로 표시되므로 결과를 해석할 때 주의합니다.

---

# 17. 6장의 이미지에서 Pose 품질을 한 번에 비교합니다

앞에서는 `front.jpg` 한 장의 Keypoint를 자세히 확인했습니다.

이번에는 준비한 6장의 이미지를 모두 같은 방법으로 분석하여 **촬영 조건에 따라 Pose 품질이 
어떻게 달라지는지 비교**합니다.

실행합니다.

```bash
python scripts/09_compare_pose_images.py
```

이번 실습의 실제 결과는 다음과 같았습니다.

```
arms_up.jpg
people=1
bbox_ratio=0.675
mean_kp=0.960
low_kp=0

distance_far.jpg
people=1
bbox_ratio=0.149
mean_kp=0.971
low_kp=0

front.jpg
people=1
bbox_ratio=0.300
mean_kp=0.975
low_kp=0

occlusion.jpg
people=1
bbox_ratio=0.265
mean_kp=0.953
low_kp=1

partial_body.jpg
people=1
bbox_ratio=0.338
mean_kp=0.907
low_kp=2

side.jpg
people=1
bbox_ratio=0.199
mean_kp=0.754
low_kp=4
```

분석 결과는 다음 CSV에도 저장됩니다.

```
reports/day02/tables/image_quality_summary.csv
```

확인합니다.

```bash
cat reports/day02/tables/image_quality_summary.csv
```

Pandas를 이용하면 표 형태로 보기 좋게 확인할 수 있습니다.

```bash
python - <<'PY'
import pandas as pd

path = "reports/day02/tables/image_quality_summary.csv"
df = pd.read_csv(path)

print(df.to_string(index=False))
PY
```

이번 실제 결과를 정리하면 다음과 같습니다.

| 이미지                | 사람 수 | BBox Conf | BBox Area Ratio | 평균 KP Conf | LOW 관절 수 | 가장 낮은 관절   | 가장 낮은 Conf |
| ------------------ | ---- | --------- | --------------- | ---------- | -------- | ---------- | ---------- |
| `front.jpg`        | 1    | 0.9228    | 0.2996          | 0.9754     | 0        | right_ear  | 0.7428     |
| `arms_up.jpg`      | 1    | 0.8380    | 0.6752          | 0.9599     | 0        | right_ear  | 0.6541     |
| `distance_far.jpg` | 1    | 0.9260    | 0.1489          | 0.9710     | 0        | right_ear  | 0.7383     |
| `occlusion.jpg`    | 1    | 0.9181    | 0.2649          | 0.9533     | 1        | right_ear  | 0.4797     |
| `partial_body.jpg` | 1    | 0.9420    | 0.3384          | 0.9073     | 2        | left_ankle | 0.4907     |
| `side.jpg`         | 1    | 0.8373    | 0.1993          | 0.7541     | 4        | right_ear  | 0.0278     |

> 여기서 `LOW`는 오늘 사용하는 기준인 **Keypoint Confidence < 0.50**인 관절을 뜻합니다.

위의 결과를 그림으로 쉽게 이해해 보면 다음과 같습니다.

![[Pasted image 20260916191001.png]]

---

# 18. 비교표에서 Pose 품질 차이를 해석합니다

17번에서 6장의 이미지를 같은 YOLO Pose 모델로 분석하여 하나의 비교표를 만들었습니다.

이제 숫자를 다시 나열하는 것이 아니라, 어떤 순서로 읽고 무엇을 판단해야 하는지 확인합니다.

![[Pasted image 20260916194514.png]]

먼저 세 가지 질문만 봅니다.

```
① 사람은 정상적으로 검출되었는가?
        ↓
people

② 관절은 전체적으로 안정적인가?
        ↓
mean_keypoint_conf

③ 특히 불안정한 관절이 있는가?
        ↓
low_keypoint_count
lowest_keypoint
```

`BBox Area Ratio`는 Pose 품질 점수가 아니라 **화면에서 사람이 얼마나 크게 보이는지 확인하는 값**입니다.

```
bbox_area_ratio 큼
→ 화면에서 사람이 크게 보임

bbox_area_ratio 작음
→ 화면에서 사람이 작게 보임
```

---

## 18.1 사람 검출과 관절 품질은 따로 봅니다

이번 6장의 결과는 모두:

```
people = 1
```

입니다.

즉 YOLO Pose는 모든 이미지에서 사람 한 명을 찾았습니다.

하지만 Keypoint 결과는 같지 않았습니다.

```
사람 검출
6장 모두 성공
        ↓

하지만

Keypoint 품질은
촬영 조건마다 다름
```

이 차이가 중요한 이유는 다음과 같습니다.

```
사람을 잘 찾았다
        ≠
모든 관절도 잘 찾았다
```

예를 들어 `side.jpg`에서도 사람은 정상적으로 검출되었습니다.

하지만 실제 결과는:

```
mean_keypoint_conf = 0.7541
low_keypoint_count = 4
```

였습니다.

즉 **사람은 찾았지만 일부 관절은 불안정했다**는 뜻입니다.

---

## 18.2 전체 결과에서는 어떤 차이가 있었나요?

17번의 실제 결과를 보면 크게 세 그룹으로 나눠 볼 수 있습니다.

```
[전체적으로 안정적]

front
distance_far
arms_up

→ 평균 Keypoint Confidence 높음
→ LOW 관절 0개


[일부 관절 주의]

occlusion

→ LOW 관절 1개


[관절 품질 저하가 더 크게 나타남]

partial_body
→ LOW 관절 2개

side
→ LOW 관절 4개
→ 평균 Keypoint Confidence도 가장 낮음
```

![[Pasted image 20261104144054.png]]

이번 데이터에서는 **`side.jpg`에서 가장 큰 품질 저하가 나타났습니다.**

```
side.jpg

mean_keypoint_conf = 0.7541
low_keypoint_count = 4
lowest_keypoint = right_ear
lowest_keypoint_conf = 0.0278
```

측면에서는 좌·우 관절이 화면상에서 겹쳐 보이기 쉬우므로 Pose 모델이 일부 관절의 위치를 구분하기 어려웠을 가능성이 있습니다.

하지만 이것을:

```
측면이면 항상 Pose가 나쁘다
```

라고 일반화하면 안 됩니다.

이번 실습에서 확인한 것은:

> **이번에 촬영한 6장의 데이터에서는 `side.jpg`가 가장 불안정하게 나타났다.**

입니다.

---

## 18.3 이번 비교에서 무엇을 배웠나요?

이번 6장의 결과를 통해 다음 관계를 확인했습니다.

```
같은 YOLO Pose 모델
        +
서로 다른 촬영 조건
        ↓
Keypoint Confidence가 달라질 수 있음
        ↓
LOW 관절 수도 달라질 수 있음
        ↓
Pose 품질도 달라질 수 있음
```

따라서 Pose 결과를 볼 때는 Skeleton이 보인다는 사실만 확인하지 않습니다.

```
사람 검출
        ↓
평균 Keypoint Confidence
        ↓
LOW 관절 수
        ↓
가장 낮은 관절
        ↓
실제 이미지 확인
```

의 순서로 함께 확인합니다.

---

## 18.4 Confidence가 낮은 자세를 발견하면 어떻게 하나요?

촬영 목적에 따라 판단이 달라집니다.

|목적|Confidence가 낮게 나오면?|
|---|---|
|2일차 품질 분석 실습|그대로 보관합니다. 왜 낮아졌는지 분석하는 중요한 데이터입니다.|
|실제 동작인식 데이터 수집|촬영 각도·거리·조명을 조정해 다시 촬영할 수 있습니다.|
|관절각도 계산|필요한 관절의 Confidence가 낮다면 해당 좌표를 주의하거나 계산에서 제외합니다.|

예를 들어 이번 `side.jpg`에서는 오른쪽 귀의 Confidence가 `0.0278`로 매우 낮았습니다.

하지만 팔꿈치 각도를 계산하는 것이 목적이라면 오른쪽 귀의 Confidence는 중요하지 않을 수 있습니다. 어깨·팔꿈치·손목의 검출 품질을 확인해야 합니다.

## 18.5 그럼 촬영 전에 Pose를 사전 점검하는 것이 좋은가요?

네. 실제 프로젝트에서는 사전 촬영 테스트를 하는 것이 좋은 방법입니다.

```
① 분석할 동작과 필요한 관절 결정
             ↓
② 해당 동작을 미리 촬영
             ↓
③ YOLO Pose로 Confidence 확인
             ↓
④ 필요한 관절이 안정적으로 검출되는가?
      │
      ├─ 예 → 본 촬영 진행
      │
      └─ 아니요
            ↓
        촬영 조건 조정
        카메라 각도·거리·조명
            ↓
        다시 테스트
```

단, 측면 동작을 반드시 분석해야 하는 프로젝트라면 측면 촬영을 포기하는 것이 아니라, 측면에서도 관절을 잘 검출할 수 있는 방법을 찾는 것이 중요합니다.

---

## 18.6 불안정한 촬영 조건을 찾으면 어떻게 활용하나요?

이번 결과에서는 `side.jpg`가 가장 불안정했습니다.

```
side.jpg

평균 Keypoint Confidence = 0.7541
LOW 관절 수 = 4
```

이 결과를 찾은 이유는 단순히 **“side가 나쁘다”라고 평가하기 위해서가 아닙니다.**

다음 분석에서 **주의해야 할 조건을 미리 찾기 위해서**입니다.

```
side 조건에서
여러 관절의 Confidence가 낮음
        ↓
이 좌표를 그대로 사용하면
잘못된 관절 위치가 들어갈 수 있음
        ↓
다음 단계의
관절각도 · Feature 계산에도
오류가 이어질 수 있음
```

따라서 실제로는 다음과 같이 활용할 수 있습니다.

### ① 촬영 조건을 조정할 수 있다면

정확한 측면 촬영이 꼭 필요한 것이 아니라면, 관절이 서로 겹치지 않도록 촬영 방향을 조정할 수 있습니다.

```
완전한 측면
→ 좌·우 관절이 많이 겹침

        ↓

조금 더 정면이 보이도록 촬영
→ 좌·우 관절을 구분하기 쉬움
```

즉 **Pose 품질이 좋은 촬영 환경을 만드는 데 결과를 활용**할 수 있습니다.

---

### ② 측면 촬영을 반드시 사용해야 한다면

Confidence가 낮은 관절을 무조건 다음 계산에 사용하지 않습니다.

예를 들어:

```
right_ear Confidence = 0.0278
        ↓
Threshold 0.50보다 낮음
        ↓
신뢰하기 어려운 Keypoint
        ↓
분석에서 제외하거나
주의가 필요한 값으로 처리
```

앞에서 작성한 코드도 같은 원리를 사용합니다.

```
if conf < kp_threshold:
    continue
```

즉 Pose 모델을 다시 학습시키는 것이 아니라, 신뢰하기 어려운 관절을 다음 분석에 그대로 사용하지 않도록 처리하는 것입니다. 
Confidence 기준으로 관절 사용 여부를 제어하는 코드는 이미 13~14번에서 학습했습니다.

---

### ③ 다음 수업의 관절각도 계산에서도 활용합니다

예를 들어 3일차에 팔꿈치 각도를 계산하려면 다음 세 관절이 필요합니다.

```
Shoulder
    ↓
Elbow
    ↓
Wrist
```

그런데 측면 촬영 때문에 Wrist Confidence가 매우 낮다면:

```
Shoulder → 정상
Elbow    → 정상
Wrist    → Confidence 낮음
        ↓
세 점으로 계산한 각도도
신뢰하기 어려움
```

따라서 이런 경우에는 낮은 Confidence 관절을 이용한 각도를 그대로 믿지 않는 처리가 필요합니다.

2일차에서 Keypoint 품질을 먼저 확인하는 이유도 바로 이것입니다. 
현재 교재에서도 2일차에서 신뢰할 수 있는 Keypoint를 확인해야 3일차의 관절각도와 Feature 
계산에 잘못된 좌표가 그대로 들어가는 것을 줄일 수 있다고 연결하고 있습니다.

---

### 18번의 최종 결론

```
불안정한 촬영 조건 발견
        ↓
왜 Confidence가 낮아졌는지 확인
        ↓
촬영 조건을 개선할 수 있으면 개선

또는

Confidence가 낮은 관절은
다음 계산에서 제외·주의 처리
        ↓
잘못된 관절 좌표가
관절각도와 Feature에
그대로 전달되는 것을 줄임
```

> 이번 `side.jpg` 결과를 찾은 이유는 
> “측면은 나쁘다”라고 결론내리기 위해서가 아니라, 
> 측면 촬영에서는 일부 Keypoint가 불안정할 수 있다는 것을 알고 촬영 방법이나 
> 이후 분석 방법을 조정하기 위해서입니다.

---

# 19. `front.jpg`와 `distance_far.jpg`로 거리 조건만 따로 비교합니다

18번에서는 6장의 전체적인 차이를 확인했습니다.

이번에는 여러 조건 중 거리 하나만 골라 조금 더 자세히 확인합니다.

비교할 이미지는 다음 두 장입니다.

```
front.jpg
→ 가까운 거리의 기준

distance_far.jpg
→ 사람이 더 멀리 있는 조건
```

이번 비교에서 확인할 질문은 하나입니다.

> 사람이 화면에서 작아졌을 때 Pose 품질도 같이 떨어졌을까요?

---

## 19.1 먼저 사람 크기를 비교합니다

![[Group 23.png]]

두 이미지의 `BBox Area Ratio`를 확인합니다.

```
front.jpg
0.2996

distance_far.jpg
0.1489
```

즉:

```
front
사람이 화면에서 상대적으로 큼

        ↓ 멀어짐

distance_far
사람이 화면에서 훨씬 작아짐
```

실제로 `distance_far.jpg`에서는 사람 BBox가 차지하는 비율이 약 절반 수준으로 줄었습니다.

따라서 거리 조건의 차이는 데이터에서도 정상적으로 나타났습니다.

---

## 19.2 그런데 Keypoint 품질도 같이 떨어졌을까요?

이번에는 평균 Keypoint Confidence를 비교합니다.

```
front.jpg
0.9754

distance_far.jpg
0.9710
```

LOW 관절 수도 확인합니다.

```
front.jpg
LOW = 0

distance_far.jpg
LOW = 0
```

결과를 연결하면:

```
사람 크기

0.2996
   ↓
0.1489

크게 감소


하지만


평균 Keypoint Confidence

0.9754
   ↓
0.9710

거의 변화 없음


LOW 관절

0
↓
0

변화 없음
```

따라서 이번 실제 데이터에서는:

> 사람이 화면에서 훨씬 작아졌지만 Pose 품질은 거의 유지되었습니다.

---

## 19.3 이 결과를 어떻게 해석해야 하나요?

여기서 다음처럼 해석하면 안 됩니다.

```
X

사람이 멀어져도
Pose 품질은 항상 문제없다.
```

이번 실험으로 확인한 사실은 다음뿐입니다.

```
O

이번 distance_far.jpg에서는
사람 크기는 크게 줄었지만

현재 거리 범위에서는
Keypoint Confidence가
크게 떨어지지 않았다.
```

즉 거리 변화가 있었다는 사실과 Pose 품질이 실제로 어떻게 변했는지는 각각 숫자로 확인해야 합니다.

이번 결과에서는 오히려 거리보다:

```
partial_body
→ LOW 2개

side
→ LOW 4개
```

처럼 화면 이탈이나 측면 자세에서 더 큰 품질 저하가 나타났습니다.

---

## 19.4 거리 비교의 결론

이번 `front.jpg`와 `distance_far.jpg`의 결과는 다음 한 문장으로 정리할 수 있습니다.

> 사람의 화면상 크기는 크게 줄었지만, 이번 촬영 거리에서는 평균 Keypoint Confidence와 LOW 관절 수가 거의 변하지 않아 Pose 품질은 잘 유지되었습니다.

이 실습에서 중요한 것은 결과를 미리 정해 놓는 것이 아닙니다.

```
"멀어졌으니 나빠졌을 것이다"
        X

실제 숫자 확인
        ↓
이미지 확인
        ↓
실제로 어떻게 변했는지 설명
        O
```

---

# PART 6. 실제 MP4를 Frame 단위로 분석합니다

---

# 20. 1일차 MP4를 2일차 분석용으로 준비합니다

1일차에서 사용한 실제 MP4를 그대로 다시 사용합니다.

```bash
cp data/day01/video/solution_ci_pose_demo.mp4 \
data/day02/videos/pose_quality_demo.mp4
```

확인합니다.

```bash
ls -lh data/day02/videos/pose_quality_demo.mp4
```

1일차에서는 이 영상을 다음처럼 사용했습니다.

```text
MP4
→ Frame 반복
→ Frame마다 Pose
→ 결과 영상 확인
```

2일차에서는 같은 영상에서 Frame마다 **Pose 품질 숫자**를 저장합니다.

```text
MP4
→ Frame 반복
→ Frame마다 Pose
→ Frame마다 Confidence 계산
→ 품질 CSV 저장
```

---

# 21. 영상의 Frame별 Pose 품질을 저장하는 코드를 작성합니다

이 코드는 다음 세 가지 결과를 만듭니다.

```text
① Pose가 표시된 결과 MP4

② Frame별 품질 CSV

③ Frame별 Keypoint CSV
```

### 이 코드는 왜 작성하나요?

1일차에는 결과 영상을 재생하면서 Skeleton을 눈으로 확인했습니다.

2일차에는 영상의 모든 순간을 사람이 직접 볼 수 없기 때문에 **Frame별 품질 숫자를 같이 저장**합니다.

```text
영상 전체를 눈으로만 검사
        X

Frame별 품질 CSV 생성
        ↓
문제가 있을 가능성이 높은 Frame 검색
        ↓
그 시간의 실제 영상 다시 확인
        O
```

### 의사코드로 읽어보기

```text
MP4를 연다
        ↓
Frame을 한 장 읽는다
        ↓
YOLO Pose 실행
        ↓
대표 사람을 선택한다
        ↓
17개 Keypoint Confidence 평균 계산
        ↓
Threshold보다 낮은 관절 수 계산
        ↓
가장 낮은 관절 찾기
        ↓
Frame 번호와 시간과 함께 CSV 저장
        ↓
다음 Frame에서 반복
        ↓
Pose 결과 MP4도 함께 저장
```

**파일: `scripts/10_pose_video_quality.py`**

```python

from __future__ import annotations

# ① 필요한 라이브러리 불러오기
import argparse
import csv
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
from src.pose_utils import (
    COCO_KEYPOINT_NAMES,
)


# ③ 터미널에서 영상 경로 및 실행 옵션 입력받기
def parse_args():
    parser = argparse.ArgumentParser()

    # 분석할 MP4 영상 (기본값)
    parser.add_argument(
        "--source",
        default=str(
            ROOT
            / "data"
            / "day02"
            / "videos"
            / "pose_quality_demo.mp4"
        ),
    )

    # LOW 관절을 판단할 Confidence 기준
    parser.add_argument(
        "--kp-conf",
        type=float,
        default=0.50,
    )

    # 최대 처리 Frame 수 (0이면 영상 전체)
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

    # 입력 영상이 존재하는지 확인
    if not source.exists():
        raise FileNotFoundError(
            f"영상이 없습니다: {source}"
        )

    # project.json 설정값 불러오기
    config = load_config()

    # GPU 또는 CPU 선택
    device = resolve_device(config["device"])

    # ⑤ YOLO Pose 모델 불러오기 및 MP4 열기
    model = YOLO(config["pose_model"])

    capture = cv2.VideoCapture(str(source))

    # 영상 연결 여부 확인
    if not capture.isOpened():
        raise RuntimeError(
            f"영상을 열 수 없습니다: {source}"
        )

    # ⑥ 영상 FPS·해상도 및 첫 Frame 확인
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

    # 원본 영상의 높이와 너비 확인
    height, width = frame.shape[:2]

    # ⑦ Pose 결과 MP4 저장 준비
    output_video = (
        ROOT
        / "reports"
        / "day02"
        / "videos"
        / f"{source.stem}_pose.mp4"
    )

    # 결과 저장 폴더 생성
    output_video.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    # 결과 영상을 저장할 VideoWriter 생성
    writer = cv2.VideoWriter(
        str(output_video),
        cv2.VideoWriter_fourcc(*"mp4v"),
        output_fps,
        (width, height),
    )

    if not writer.isOpened():
        capture.release()
        raise RuntimeError(
            "결과 MP4를 생성할 수 없습니다."
        )

    # ⑧ Frame 품질 CSV와 Keypoint CSV 저장 준비
    # Frame별 품질 요약 CSV 경로
    frame_table = (
        ROOT
        / "reports"
        / "day02"
        / "tables"
        / f"{source.stem}_frame_quality.csv"
    )

    # Frame별 관절 상세 CSV 경로
    keypoint_table = (
        ROOT
        / "reports"
        / "day02"
        / "tables"
        / f"{source.stem}_keypoints.csv"
    )

    # CSV 저장 폴더 생성
    frame_table.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    # CSV에 기록할 데이터를 담을 목록
    frame_rows = []
    keypoint_rows = []

    # Frame 번호 초기화
    frame_id = 0

    # ⑨ Frame별 YOLO Pose 추론 및 결과 영상 저장
    while success:

        # 처리 중인 Frame 번호 증가
        frame_id += 1

        # 현재 Frame에 YOLO Pose 적용
        results = model.predict(
            source=frame,
            imgsz=config["image_size"],
            conf=config["person_confidence"],
            device=device,
            verbose=False,
        )

        # 현재 Frame의 추론 결과 가져오기
        result = results[0]

        # BBox + Keypoint + Skeleton 표시 후 MP4 저장
        writer.write(result.plot())

        # ⑩ 검출된 사람 수 및 품질 기본값 설정
        people = (
            len(result.boxes)
            if result.boxes is not None
            else 0
        )

        # 사람이나 관절을 찾지 못한 경우의 기본값
        mean_kp_conf = 0.0
        low_keypoints = 17
        lowest_name = "NO_PERSON"
        lowest_conf = 0.0

        # ⑪ 대표 사람의 관절 Confidence 분석
        if (
            result.boxes is not None
            and result.keypoints is not None
            and result.keypoints.xy is not None
            and result.keypoints.conf is not None
            and len(result.boxes) > 0
        ):

            # 사람별 BBox Confidence 가져오기
            box_conf = (
                result.boxes.conf
                .cpu()
                .numpy()
            )

            # 사람별 17개 관절 좌표 가져오기
            xy = (
                result.keypoints.xy
                .cpu()
                .numpy()
            )

            # 사람별 관절 Confidence 가져오기
            conf = (
                result.keypoints.conf
                .cpu()
                .numpy()
            )

            # BBox Confidence가 가장 높은 사람 선택
            main_index = int(
                np.argmax(box_conf)
            )

            # 대표 사람의 관절 좌표 및 Confidence
            main_xy = xy[main_index]
            main_conf = conf[main_index]

            # 17개 관절 Confidence 평균 계산
            mean_kp_conf = float(
                np.mean(main_conf)
            )

            # Threshold보다 Confidence가 낮은 관절 수
            low_keypoints = int(
                np.sum(
                    main_conf < args.kp_conf
                )
            )

            # 가장 Confidence가 낮은 관절 번호
            lowest_id = int(
                np.argmin(main_conf)
            )

            # 가장 신뢰도가 낮은 관절 이름
            lowest_name = (
                COCO_KEYPOINT_NAMES[
                    lowest_id
                ]
            )

            # 해당 관절의 Confidence
            lowest_conf = float(
                main_conf[lowest_id]
            )

            # ⑫ 현재 Frame의 17개 Keypoint 상세 정보 기록
            for keypoint_id, name in enumerate(
                COCO_KEYPOINT_NAMES
            ):

                # 관절별 Pixel x, y 좌표
                x, y = main_xy[keypoint_id]

                # 해당 관절의 Confidence
                confidence = main_conf[keypoint_id]

                # 관절 하나의 정보를 CSV 목록에 추가
                keypoint_rows.append(
                    {
                        "frame_id": frame_id,

                        "time_sec": round(
                            (frame_id - 1)
                            / output_fps,
                            3,
                        ),

                        "keypoint_id": keypoint_id,
                        "keypoint_name": name,

                        "pixel_x": round(
                            float(x),
                            2,
                        ),

                        "pixel_y": round(
                            float(y),
                            2,
                        ),

                        "confidence": round(
                            float(confidence),
                            4,
                        ),
                    }
                )

        # ⑬ 현재 Frame의 Pose 품질 요약 기록
        frame_rows.append(
            {
                # Frame 번호
                "frame_id": frame_id,

                # 현재 Frame의 시간(초)
                "time_sec": round(
                    (frame_id - 1)
                    / output_fps,
                    3,
                ),

                # 검출된 사람 수
                "people": people,

                # 17개 관절 평균 Confidence
                "mean_keypoint_conf": round(
                    mean_kp_conf,
                    4,
                ),

                # 기준 미만 관절 개수
                "low_keypoint_count": (
                    low_keypoints
                ),

                # 가장 신뢰도가 낮은 관절 이름
                "lowest_keypoint": (
                    lowest_name
                ),

                # 가장 낮은 관절 Confidence
                "lowest_keypoint_conf": round(
                    lowest_conf,
                    4,
                ),

                # 이번 분석에 사용한 기준값
                "keypoint_threshold": (
                    args.kp_conf
                ),
            }
        )

        # ⑭ 진행 상황 확인 및 다음 Frame 읽기
        # 30개 Frame마다 진행 상황 출력
        if frame_id % 30 == 0:
            print(
                f"Processed frame: {frame_id}"
            )

        # 지정한 최대 Frame 수에 도달했는지 확인
        if (
            args.max_frames > 0
            and frame_id >= args.max_frames
        ):
            break

        # 다음 Frame 읽기
        success, frame = capture.read()

    # ⑮ 영상 자원 해제 및 Frame 품질 CSV 저장
    # 원본 영상 및 결과 영상 파일 닫기
    capture.release()
    writer.release()

    # Frame별 품질 요약 CSV 작성
    with frame_table.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:

        writer_csv = csv.DictWriter(
            file,
            fieldnames=[
                "frame_id",
                "time_sec",
                "people",
                "mean_keypoint_conf",
                "low_keypoint_count",
                "lowest_keypoint",
                "lowest_keypoint_conf",
                "keypoint_threshold",
            ],
        )

        # 컬럼 이름 저장
        writer_csv.writeheader()

        # Frame별 품질 데이터 저장
        writer_csv.writerows(frame_rows)

    # ⑯ Keypoint 상세 CSV 저장 및 최종 결과 출력
    with keypoint_table.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:

        writer_csv = csv.DictWriter(
            file,
            fieldnames=[
                "frame_id",
                "time_sec",
                "keypoint_id",
                "keypoint_name",
                "pixel_x",
                "pixel_y",
                "confidence",
            ],
        )

        # 컬럼 이름 저장
        writer_csv.writeheader()

        # Frame별 17개 관절 데이터 저장
        writer_csv.writerows(keypoint_rows)

    # 최종 처리 결과 출력
    print()
    print(f"Frames processed: {frame_id}")
    print(f"Pose video: {output_video}")
    print(f"Frame table: {frame_table}")
    print(f"Keypoint table: {keypoint_table}")


# 프로그램 실행 시작
if __name__ == "__main__":
    main()
```

**의사코드**

```
① 필요한 라이브러리 불러오기
   ├─ argparse → 터미널 실행 옵션 받기
   ├─ csv      → 분석 결과 CSV 저장
   ├─ Path     → 영상 및 결과 파일 경로 관리
   ├─ cv2      → 영상 Frame 읽기·저장
   ├─ numpy    → Confidence 평균·최솟값 계산
   └─ YOLO     → Pose 추론 실행
                ↓
② 프로젝트 경로와 공통 함수 준비
   ├─ ROOT → 프로젝트 최상위 폴더
   ├─ common.py → 설정 읽기·GPU/CPU 선택
   └─ pose_utils.py → COCO 17개 관절 이름
                ↓
③ 터미널 실행 옵션 준비
   ├─ --source → 분석할 MP4 영상
   ├─ --kp-conf → 낮은 관절을 구분할 기준
   │              기본값 0.50
   └─ --max-frames → 최대 처리 Frame 수
                     기본값 150
                     0이면 영상 전체
                ↓
④ 입력 영상 및 프로젝트 설정 확인
   ├─ MP4 파일이 존재하는가?
   │    ├─ 아니요 → 오류 발생
   │    └─ 예 → 다음 단계
   ├─ project.json 설정 읽기
   └─ GPU 또는 CPU 선택
                ↓
⑤ YOLO Pose 모델 로딩 및 MP4 열기
   ├─ yolo11n-pose.pt 불러오기
   ├─ VideoCapture로 영상 열기
   └─ 영상 연결 성공 여부 확인
                ↓
⑥ 영상의 기본 정보 확인
   ├─ FPS 가져오기
   ├─ 첫 번째 Frame 읽기
   └─ 영상 Width · Height 확인
                ↓
⑦ Pose 결과 MP4 저장 준비
   ├─ 결과 저장 경로 설정
   ├─ VideoWriter 생성
   └─ 원본 영상 해상도와 FPS 설정
                ↓
⑧ 분석 결과를 저장할 CSV 준비
   ├─ Frame 품질 CSV 경로 생성
   ├─ Keypoint 상세 CSV 경로 생성
   ├─ frame_rows 목록 준비
   └─ keypoint_rows 목록 준비
                ↓
⑨ 현재 Frame에 YOLO Pose 적용
   ├─ Frame 번호 증가
   ├─ 사람 및 17개 관절 추론
   ├─ BBox + Skeleton 표시
   └─ 결과 Frame을 MP4에 저장
                ↓
⑩ 사람 검출 여부 확인
   ├─ 검출된 사람 수 확인
   └─ 분석 기본값 준비
                ↓
⑪ 대표 사람의 관절 품질 분석
   ├─ BBox Confidence가 가장 높은 사람 선택
   ├─ 17개 Keypoint 좌표 추출
   ├─ 17개 Confidence 평균 계산
   ├─ 0.50 미만 관절 수 계산
   └─ 가장 Confidence가 낮은 관절 확인
                ↓
⑫ Frame별 Keypoint 상세 데이터 기록
   ├─ Frame 번호 및 시간
   ├─ 관절 번호(0~16)
   ├─ 관절 이름
   ├─ Pixel x, y 좌표
   └─ 관절 Confidence
                ↓
⑬ 현재 Frame의 품질 요약 기록
   ├─ Frame 번호 및 시간
   ├─ 사람 수
   ├─ 평균 Keypoint Confidence
   ├─ LOW 관절 수
   ├─ 가장 낮은 Confidence 관절
   └─ 사용한 Threshold
                ↓
⑭ 다음 Frame 처리 여부 확인
   ├─ 30 Frame마다 진행 상황 출력
   ├─ 최대 Frame 수에 도달했는가?
   │    ├─ 예 → 반복 종료
   │    └─ 아니요 → 다음 Frame 읽기
   └─ 영상이 끝나면 반복 종료

        다음 Frame이 있다면
             ⑨~⑭ 반복
                ↓
⑮ 영상 자원 해제 및 Frame 품질 CSV 저장
   ├─ VideoCapture 해제
   ├─ VideoWriter 해제
   ├─ Frame별 품질 데이터 정리
   └─ frame_quality.csv 저장
                ↓
⑯ Keypoint 상세 CSV 저장 및 마무리
   ├─ 각 Frame의 관절 좌표 저장
   ├─ 관절별 Confidence 저장
   ├─ keypoints.csv 저장
   └─ 최종 결과 파일 경로 출력
```

---

# 22. 먼저 60 Frame만 분석합니다

처음부터 긴 영상을 모두 처리하지 않고 일부 Frame으로 Pipeline을 확인합니다.

```bash
python scripts/10_pose_video_quality.py \
  --source data/day02/videos/pose_quality_demo.mp4 \
  --max-frames 60
```

정상적으로 끝나면 다음 결과가 생성됩니다.

```text
reports/day02/videos/
└── pose_quality_demo_pose.mp4

reports/day02/tables/
├── pose_quality_demo_frame_quality.csv
└── pose_quality_demo_keypoints.csv
```

파일을 확인합니다.

```bash
ls -lh reports/day02/videos/
```

```bash
ls -lh reports/day02/tables/
```

영상의 처음부터 끝까지 모두 처리하려면:

```bash
python scripts/10_pose_video_quality.py \
  --source data/day02/videos/pose_quality_demo.mp4 \
  --max-frames 0
```

> `--max-frames 150`은 앞쪽 150개 Frame까지만 분석하고, `--max-frames 0`은 Frame 수 제한 없이 영상이 끝날 때까지 분석합니다.

---

# 23. 실제 Frame 품질 CSV를 읽어봅니다

앞에서 영상을 Frame 단위로 분석하여 다음 파일을 만들었습니다.

```
reports/day02/tables/
pose_quality_demo_frame_quality.csv
```

이 CSV는 **영상의 각 순간에서 Pose가 얼마나 안정적으로 검출되었는지 숫자로 기록한 표**입니다.

먼저 앞부분을 확인합니다.

```bash
head -20 \
reports/day02/tables/pose_quality_demo_frame_quality.csv
```

이번 실제 영상은 총 **60개 Frame**이 분석되었습니다.

```
Frame 1
        ↓
Frame 2
        ↓
...
        ↓
Frame 60
```

마지막 시간이 약 `1.967초`이므로 이번 영상은 약 2초 정도의 짧은 영상입니다.

> `--max-frames 150`으로 실행했더라도 영상 자체가 60 Frame이면 영상이 끝나는 시점에서 분석도 끝납니다.

---

## 23.1 CSV 한 줄은 무엇을 의미하나요?

주요 값은 다음과 같습니다.

```
frame_id
→ 영상의 몇 번째 Frame인가?

time_sec
→ 영상의 몇 초 지점인가?

people
→ 사람을 몇 명 찾았는가?

mean_keypoint_conf
→ 17개 관절 Confidence의 평균

low_keypoint_count
→ Confidence가 0.50보다 낮은 관절이 몇 개인가?

lowest_keypoint
→ 그 Frame에서 가장 Confidence가 낮은 관절

lowest_keypoint_conf
→ 그 관절의 실제 Confidence
```

즉 한 줄은 다음 질문에 대한 답입니다.

```
"이 순간의 Pose는
전체적으로 얼마나 안정적이고,
특히 어떤 관절이 불안정한가?"
```

---

## 23.2 실제 1번 Frame을 읽어봅니다

실제 첫 번째 Frame의 결과는 다음과 같습니다.

```
frame_id              = 1
time_sec               = 0.000
people                 = 1
mean_keypoint_conf     = 0.8619
low_keypoint_count     = 1
lowest_keypoint        = left_ear
lowest_keypoint_conf   = 0.1732
```

쉽게 읽으면:

```
영상 시작 지점
0.000초
        ↓
사람 1명 정상 검출
        ↓
17개 관절 평균 Confidence
0.8619
        ↓
하지만
0.50보다 낮은 관절이 1개 있음
        ↓
그 관절은 left_ear
Confidence = 0.1732
```

여기서 중요한 점은 **평균값만 보면 안 된다는 것**입니다.

```
평균 Confidence = 0.8619
→ 전체적으로는 비교적 높아 보임

하지만

left_ear = 0.1732
→ 특정 관절 하나는 매우 낮음
```

따라서 Pose 품질을 볼 때는:

```
평균 Confidence
        +
LOW 관절 수
        +
가장 낮은 관절
```

을 함께 확인합니다.

---

# 24. 실제로 품질이 낮은 Frame을 찾아봅니다

영상의 60개 Frame을 하나씩 눈으로 확인하는 것은 비효율적입니다.

그래서 CSV에서 먼저 확인이 필요한 Frame 후보를 찾습니다.

이번에는 다음 순서로 정렬합니다.

```
LOW 관절 수가 많은 Frame 우선
        ↓
같다면
평균 Confidence가 낮은 Frame 우선
```

실행합니다.

```bash
python - <<'PY'
import pandas as pd

path = (
    "reports/day02/tables/"
    "pose_quality_demo_frame_quality.csv"
)

df = pd.read_csv(path)

worst = df.sort_values(
    [
        "low_keypoint_count",
        "mean_keypoint_conf",
    ],
    ascending=[False, True],
).head(10)

print(worst.to_string(index=False))
PY
```

이번 실제 결과에서 가장 먼저 확인할 Frame은 다음과 같았습니다.

|Frame|시간|평균 Confidence|LOW 관절 수|가장 낮은 관절|가장 낮은 Confidence|
|---|---|---|---|---|---|
|25|0.800초|0.7925|**4**|left_eye|0.3449|
|26|0.833초|0.7934|**4**|left_eye|0.3423|
|24|0.767초|0.7788|3|left_eye|0.2467|
|36|1.167초|0.7813|3|right_eye|0.2231|
|33|1.067초|0.7834|3|left_eye|0.3576|

따라서 이번 영상에서는 **약 0.8초 부근을 먼저 확인하는 것이 좋습니다.**

---

## 24.1 25번 Frame에서는 무엇이 불안정했나요?

`pose_quality_demo_keypoints.csv`에서 25번 Frame을 자세히 확인한 실제 결과입니다.

```
left_eye   = 0.3449
right_eye  = 0.4121
left_ear   = 0.4736
right_ear  = 0.4941
```

네 관절이 모두 오늘의 기준인 `0.50`보다 낮습니다.

```
Frame 25
0.800초

        ↓

LOW 관절 4개

left_eye
right_eye
left_ear
right_ear

        ↓

얼굴 주변 Keypoint가
집중적으로 불안정
```

즉 이번 결과에서는 팔이나 다리보다 **눈과 귀 주변 Keypoint의 Confidence가 낮아진 순간**을 찾은 것입니다.

---

## 24.2 그러면 25번 Frame은 Pose 실패인가요?

아직 그렇게 단정하지 않습니다.

CSV가 알려준 것은:

> “0.800초 부근에서 얼굴 주변 Keypoint의 Confidence가 낮으니 실제 영상을 확인해 보자.”

까지입니다.

```
CSV
        ↓
Frame 25가 의심됨
        ↓
0.800초 확인
        ↓
실제 영상과 Skeleton 확인
        ↓
왜 Confidence가 낮았는지 판단
```

결과 영상을 엽니다.

```bash
explorer.exe "$(wslpath -w reports/day02/videos)"
```

다음 파일을 재생합니다.

```
pose_quality_demo_pose.mp4
```

약 **0.80초 부근**을 확인합니다.

이때 다음을 봅니다.

```
얼굴이 카메라를 향하고 있었는가?

고개가 돌아가거나 숙여졌는가?

눈이나 귀가 잘 보이지 않았는가?

움직임 때문에 얼굴이 흐려졌는가?

Skeleton의 얼굴 부분이
실제 위치와 맞게 표시되었는가?
```

![[Pasted image 20260916203347.png]]

### 실제 영상에서 0.80초 부근을 확인합니다

다음 결과 영상을 재생합니다.

```
pose_quality_demo_pose.mp4
```

약 **0.80초 부근**을 확인하면 사람이 바닥에서 팔을 굽힌 자세를 취하고 있습니다.

이 순간에는 몸 전체는 비교적 잘 보이지만, 얼굴이 작게 보이고 고개가 아래쪽·옆쪽으로 향하면서 눈과 귀가 정면처럼 선명하게 보이지 않습니다.

실제 CSV에서도 이 순간의 얼굴 Keypoint Confidence가 낮았습니다.

```
Frame 25
time = 0.800초

left_eye  = 0.3449
right_eye = 0.4121
left_ear  = 0.4736
right_ear = 0.4941

→ 4개 모두 0.50 미만
```

영상과 숫자를 함께 보면 다음과 같이 해석할 수 있습니다.

```
바닥 자세로 몸이 수평에 가까워짐
        ↓
얼굴이 작고 옆·아래 방향으로 보임
        ↓
눈·귀 위치가 정면보다 구분하기 어려움
        ↓
얼굴 Keypoint Confidence 감소
        ↓
LOW 관절 4개 발생
```

반면 어깨·팔·다리 등의 Skeleton은 계속 비교적 잘 연결되어 있습니다.

따라서 이번 결과는 사람 전체를 찾지 못한 문제가 아니라, 특정 자세에서 얼굴 주변 Keypoint만 상대적으로 불안정해진 경우로 해석할 수 있습니다.

**실제 결과 해석:** 
약 0.80초에서는 바닥 자세와 얼굴 방향 때문에 눈·귀가 카메라에서 명확하게 보이지 않아 얼굴 Keypoint의 Confidence가 낮아졌습니다. 사람과 몸 전체 Pose는 계속 검출되었지만, 얼굴 주변 관절은 주의해서 사용해야 하는 Frame입니다.

---

## 24.3 왜 숫자와 영상을 같이 봐야 하나요?

이번 실제 결과를 보면 그 이유를 이해할 수 있습니다.

```
Frame 25

평균 Confidence = 0.7925
LOW 관절 = 4개
        ↓
숫자로 이상 후보 발견
```

하지만 이 숫자만으로:

```
고개를 돌렸다
얼굴이 가려졌다
빠르게 움직였다
Pose가 틀렸다
```

라고 결정할 수는 없습니다.

따라서 분석 순서는 항상 다음과 같습니다.

```
CSV에서
불안정한 Frame 찾기
        ↓
Frame 번호와 시간 확인
        ↓
실제 영상으로 이동
        ↓
Skeleton과 실제 사람 확인
        ↓
낮은 관절과 촬영 상황 비교
        ↓
원인 판단
```

---

## 24.4 이번 실제 영상에서 무엇을 발견했나요?

CSV 전체를 보면 다음 특징이 있습니다.

```
60개 Frame 모두
people = 1

→ 사람 자체는 계속 정상 검출
```

하지만:

```
LOW 관절 수

1개인 Frame → 6개
2개인 Frame → 36개
3개인 Frame → 16개
4개인 Frame → 2개
```

였습니다.

특히 LOW로 자주 나타난 관절은 **얼굴 주변 Keypoint**였습니다.

```
left_eye
left_ear
right_eye
right_ear
일부 Frame의 nose
```

즉 이번 영상에서는:

```
사람 Detection
→ 비교적 안정적

몸 전체 Pose
→ 상당 부분 검출됨

하지만

일부 순간의 얼굴 Keypoint
→ Confidence가 낮아짐
```

이라는 패턴을 발견했습니다.

---

## 24.5 이 결과를 어디에 활용하나요?

이 분석의 목적은 단순히 “나쁜 Frame을 찾았다”에서 끝나는 것이 아닙니다.

예를 들어 다음 수업에서 Keypoint 좌표를 이용해 Feature를 만들 때:

```
Confidence가 매우 낮은 Keypoint
        ↓
좌표도 신뢰하기 어려울 수 있음
        ↓
그 좌표를 그대로 Feature에 사용하면
분석 결과에 영향을 줄 수 있음
```

따라서 먼저 품질을 확인하여:

```
신뢰할 수 있는 Keypoint
→ 사용

Confidence가 너무 낮은 Keypoint
→ 주의하거나 제외
```

할 수 있습니다.

### 23~24번의 핵심

```
23번
60개 Frame의 Pose 품질을
CSV 숫자로 확인

        ↓

24번
LOW 관절이 많은 Frame을
자동으로 찾음

        ↓

25번·26번 Frame
약 0.80초 부근 확인

        ↓

바닥 자세와 얼굴 방향 때문에
눈·귀 Keypoint Confidence가 낮아진 것을
실제 영상에서 확인
```

> **결론:**  
> 이번 영상에서는 사람은 60개 Frame 모두 정상적으로 검출되었습니다. 하지만 약 `0.80초` 부근의 25~26번 Frame에서는 얼굴이 작고 옆·아래 방향으로 보여 눈과 귀 Keypoint의 Confidence가 낮아졌습니다.  
> 
> 즉 CSV를 이용하면 Pose가 불안정한 순간을 빠르게 찾고, 
> 실제 영상에서 어떤 자세나 촬영 조건 때문에 품질이 낮아졌는지 확인할 수 있습니다.
---

# PART 7. Team Mini Challenge — 우리 팀 사진으로 Pose 품질을 다시 확인합니다

---

# 25. Mini Challenge의 목표를 확인합니다

지금까지는 강사가 준비한 이미지를 이용하여 Pose 품질을 분석했습니다.

이번에는 팀에서 직접 사진을 촬영하고, 오늘 배운 방법을 이용하여 결과를 스스로 해석해 봅니다.

새로운 코드를 작성하는 실습이 아닙니다.

```
우리 팀이 직접 촬영
        ↓
YOLO Pose 실행
        ↓
Confidence 확인
        ↓
두 촬영 조건 비교
        ↓
어떤 조건에서
관절 품질이 달라졌는지 설명
```

이번 Challenge의 질문은 하나입니다.

> **촬영 조건 하나를 바꾸면 Keypoint 품질은 어떻게 달라질까요?**

---

# 26. 팀별로 비교할 촬영 조건 하나를 선택합니다

다음 네 가지 중 **하나만 선택**합니다.

|선택|조건 A|조건 B|
|---|---|---|
|거리|가까이|멀리|
|방향|정면|측면|
|가림|팔이 잘 보임|한쪽 팔 일부 가림|
|화면 이탈|전신이 화면 안|한쪽 손목 또는 발목이 화면 밖|

예를 들어 방향을 선택했다면:

```
조건 A
정면 촬영

VS

조건 B
측면 촬영
```

입니다.

### 촬영할 때 가장 중요한 규칙

**비교하려는 조건 하나만 바꿉니다.**

예를 들어 방향을 비교한다면:

```
같은 사람
같은 장소
같은 카메라
비슷한 거리
같은 자세

        ↓

방향만

정면 → 측면
```

으로 바꿉니다.

여러 조건을 동시에 바꾸면 어떤 이유 때문에 결과가 달라졌는지 알기 어렵습니다.

---

# 27. 조건 A와 B를 각각 2장씩 촬영합니다

각 팀은 총 **4장**만 촬영합니다.

```
조건 A
2장

+

조건 B
2장

=

총 4장
```

같은 팀원 한 명을 촬영하고 다른 팀원은 화면 밖으로 이동합니다.

파일명은 간단하게 다음처럼 사용합니다.

```
challenge_A_01.jpg
challenge_A_02.jpg

challenge_B_01.jpg
challenge_B_02.jpg
```

사진을 다음 위치에 넣습니다.

```
data/day02/images/
```

Windows 파일 탐색기를 이용한다면:

```bash
explorer.exe .
```

를 실행하여 다음 폴더에 넣습니다.

```
data/day02/images/
```

촬영이 끝났으면 파일이 있는지만 확인합니다.

```bash
ls -lh data/day02/images/challenge_*.jpg
```

---

# 28. 먼저 대표 사진 두 장을 자세히 확인합니다

조건 A와 조건 B에서 한 장씩 선택합니다.

```
challenge_A_01.jpg
challenge_B_01.jpg
```

오늘 본수업에서 사용한 코드를 그대로 실행합니다.

```bash
python scripts/07_pose_image_detail.py \
  --source data/day02/images/challenge_A_01.jpg
```

```bash
python scripts/07_pose_image_detail.py \
  --source data/day02/images/challenge_B_01.jpg
```

새로운 내용을 분석하는 것이 아닙니다.

다음 항목만 다시 확인합니다.

```
① 사람은 1명 검출되었는가?

② Person BBox Confidence는 얼마인가?

③ 가장 낮은 Keypoint는 무엇인가?

④ 그 Keypoint Confidence는 얼마인가?

⑤ Pixel 좌표와
   Normalized 좌표가 함께 저장되었는가?
```

두 대표 사진의 결과를 간단히 기록합니다.

|확인 항목|조건 A|조건 B|
|---|---|---|
|Person BBox Confidence|||
|가장 낮은 Keypoint|||
|가장 낮은 Keypoint Confidence|||
|LOW 여부 (`< 0.50`)|||

여기서는 모든 관절 값을 옮겨 적지 않습니다.

**가장 낮은 Keypoint가 무엇인지 찾을 수 있으면 충분합니다.**

---

# 29. 우리 팀 사진 4장을 한 번에 비교합니다

이제 본수업에서 사용했던 비교 코드를 다시 실행합니다.

```
python scripts/09_compare_pose_images.py
```

결과 CSV에서 우리 팀이 촬영한 `challenge_` 이미지만 확인합니다.

```bash
python - <<'PY'
import pandas as pd

path = "reports/day02/tables/image_quality_summary.csv"

df = pd.read_csv(path)

challenge = df[
    df["image"].str.startswith("challenge_")
]

print(challenge.to_string(index=False))
PY
```

이번에는 모든 열을 복잡하게 분석하지 않습니다.

다음 네 가지만 봅니다.

```
people
→ 사람 1명을 제대로 찾았는가?

bbox_area_ratio
→ 화면 속 사람 크기는 어떻게 달라졌는가?

mean_keypoint_conf
→ 전체 관절 Confidence 평균은 어떻게 달라졌는가?

low_keypoint_count
→ 0.50보다 낮은 관절은 몇 개인가?
```

결과를 간단하게 적습니다.

|이미지|조건|BBox Area Ratio|평균 KP Conf|LOW 관절 수|
|---|---|---|---|---|
|`challenge_A_01.jpg`|A||||
|`challenge_A_02.jpg`|A||||
|`challenge_B_01.jpg`|B||||
|`challenge_B_02.jpg`|B||||

---

# 30. 가장 차이가 컸던 결과 한 장을 확인합니다

네 장 중 다음 조건에 해당하는 사진을 하나 찾습니다.

```
평균 Keypoint Confidence가 낮거나

LOW 관절 수가 많거나

특정 관절 Confidence가
다른 사진보다 많이 낮은 이미지
```

그 사진의 실제 Pose 결과도 함께 봅니다.

```
숫자로 불안정한 사진 발견
        ↓
실제 이미지 확인
        ↓
어떤 관절이 불안정했는지 확인
        ↓
왜 그런 결과가 나왔는지 생각
```

예를 들어 측면 촬영에서 LOW 관절이 늘었다면:

```
측면 촬영
        ↓
좌·우 관절이 겹쳐 보임
        ↓
일부 Keypoint Confidence 감소
```

처럼 설명할 수 있습니다.

가림 실험이라면:

```
팔 일부 가림
        ↓
팔꿈치 또는 손목이
카메라에서 잘 보이지 않음
        ↓
해당 Keypoint Confidence 감소
```

처럼 실제 사진과 연결하여 설명합니다.

> 결과가 예상과 다르게 나와도 괜찮습니다.  
> 중요한 것은 실제 숫자와 사진을 근거로 결과를 설명하는 것입니다.

---

# 31. Threshold는 필요한 경우 한 번만 다시 확인합니다

Challenge 사진에서 Confidence가 낮은 관절이 실제로 발견되었다면, 그 사진 한 장에서만 Threshold를 다시 확인합니다.

예를 들어 가장 낮은 Keypoint Confidence가 `0.42`였다면:

```
Threshold = 0.30
→ 해당 관절 사용

Threshold = 0.50
→ 해당 관절 제외
```

본수업에서 배운 다음 원리를 다시 확인하면 됩니다.

```
Confidence >= Threshold
→ 관절 사용

Confidence < Threshold
→ 관절 제외
```

LOW 관절이 없고 Pose 결과도 안정적이라면 Threshold 실험을 다시 반복하지 않아도 됩니다.

Mini Challenge의 목적은 Threshold 값을 여러 번 바꾸는 것이 아니라, Confidence가 낮은 관절을 발견했을 때 Threshold를 왜 사용하는지 설명할 수 있는지 확인하는 것입니다.

---

# 32. 팀의 결과를 짧게 기록합니다

긴 보고서를 작성하지 않습니다.

다음 내용만 `reports/day02/challenge/challenge_notes.md`에 기록합니다.

```markdown
# Day02 Team Mini Challenge

## 1. 우리 팀 실험

- 선택한 조건:
- 조건 A:
- 조건 B:
- 바꾼 조건 한 가지:

## 2. 결과

- 조건 A 평균 Keypoint Confidence:
- 조건 B 평균 Keypoint Confidence:

- 조건 A LOW 관절 수:
- 조건 B LOW 관절 수:

- 가장 불안정했던 이미지:
- 가장 낮았던 Keypoint:
- 해당 Confidence:

## 3. 우리 팀의 해석

- 어떤 조건에서 Pose 품질이 더 불안정했는가?
- 실제 사진을 보았을 때 그 이유는 무엇이라고 생각하는가?
- Confidence가 낮은 관절을 이후 분석에 사용할 때 무엇을 주의해야 하는가?
```

세 번째 질문은 오늘 배운 내용을 연결하는 질문입니다.

예를 들어:

```
Confidence가 매우 낮은 Wrist
        ↓
좌표도 신뢰하기 어려울 수 있음
        ↓
3일차 관절각도 계산에
그대로 사용하면 주의가 필요
```

라고 설명할 수 있습니다.

2일차에서 신뢰할 수 있는 Keypoint를 먼저 확인하는 이유는 이후 관절각도와 Feature 계산에 불안정한 좌표가 그대로 들어가는 것을 줄이기 위해서입니다.

---

# 33. Mini Challenge 완료 조건을 확인합니다

다음 항목만 확인하면 Challenge를 완료한 것입니다.

- [ ]  비교할 촬영 조건 하나를 선택했다.
- [ ]  같은 사람을 이용하여 조건 A 2장, 조건 B 2장을 촬영했다.
- [ ]  두 조건에서 비교하려는 조건 하나만 바꾸었다.
- [ ]  대표 이미지 A/B의 BBox Confidence와 Keypoint Confidence를 확인했다.
- [ ]  사진 4장의 평균 Keypoint Confidence와 LOW 관절 수를 비교했다.
- [ ]  가장 불안정한 사진과 Keypoint를 찾았다.
- [ ]  실제 사진을 보고 Confidence가 낮아진 이유를 설명했다.
- [ ]  결과를 `challenge_notes.md`에 짧게 기록했다.

### Mini Challenge의 최종 흐름

```
팀별 촬영 조건 선택
        ↓
조건 A 2장
조건 B 2장
        ↓
대표 사진 상세 분석
        ↓
4장 전체 비교
        ↓
평균 Keypoint Confidence
LOW 관절 수 확인
        ↓
가장 불안정한 사진 찾기
        ↓
실제 사진과 숫자를 함께 보고
원인 설명
        ↓
짧게 기록
```

> **Mini Challenge의 핵심:**  
> 
> 강사가 준비한 사진으로 배웠던 Pose 품질 분석을 우리 팀이 직접 촬영한 사진에 다시 적용해 보고, 숫자와 실제 촬영 상황을 연결해서 설명하는 것입니다.
---

# PART 8. 오늘 만든 결과를 정리하고 버전관리합니다

---

# 34. README에 2일차 내용을 정리합니다

2일차 실습을 마쳤으므로 오늘 배운 내용과 주요 결과를 `README.md`에 정리합니다.

1일차 README를 새로 만드는 것이 아니라 **기존 README 아래에 Day 02 내용을 이어서 추가**합니다.

````markdown
## Day 02 - Keypoint 좌표와 Pose 품질 분석

### 오늘 사용한 입력

- 강사가 준비한 조건별 이미지 6장
  - `front.jpg`
  - `arms_up.jpg`
  - `side.jpg`
  - `partial_body.jpg`
  - `occlusion.jpg`
  - `distance_far.jpg`
- 1일차에서 사용한 실제 MP4
- Team Mini Challenge 직접 촬영 이미지

### 오늘 학습한 내용

- Person BBox Confidence와 Keypoint Confidence 구분
- COCO 17 Keypoint Pixel 좌표 확인
- Normalized 좌표 이해
- Keypoint Confidence와 Threshold 이해
- Confidence 기준에 따른 Skeleton 표시
- 여러 이미지의 Pose 품질 CSV 비교
- BBox Area Ratio 비교
- 평균 Keypoint Confidence와 LOW 관절 확인
- MP4의 Frame별 Pose 품질 분석
- 불안정한 Frame을 숫자로 찾고 실제 영상과 연결
- 촬영 조건 하나를 바꾸어 Pose 품질 비교

### 실제 실습에서 확인한 결과

- `front.jpg`
  - 평균 Keypoint Confidence: `0.9754`
  - LOW 관절: `0`
  - 정면 기준 이미지에서 전체적으로 안정적

- `side.jpg`
  - 평균 Keypoint Confidence: `0.7541`
  - LOW 관절: `4`
  - 이번 6장 중 가장 불안정한 결과

- `front.jpg` VS `distance_far.jpg`
  - 사람의 화면상 크기는 크게 감소
  - 평균 Keypoint Confidence는 `0.9754 → 0.9710`
  - LOW 관절은 두 이미지 모두 `0`
  - 이번 거리 범위에서는 Pose 품질이 잘 유지됨

- MP4 Frame 분석
  - 사람은 60개 Frame 모두 검출됨
  - 약 `0.80초`의 25~26번 Frame에서 얼굴 주변 Keypoint Confidence가 낮아짐
  - 실제 영상에서 바닥 자세와 얼굴 방향이 함께 나타난 것을 확인

### Day 02 Pipeline

```text
이미지 · MP4
        ↓
YOLO Pose
        ↓
Person BBox
+
COCO 17 Keypoint
        ↓
Pixel / Normalized 좌표
        ↓
Keypoint Confidence
        ↓
Threshold
        ↓
조건별 Pose 품질 비교
        ↓
Frame별 Pose 품질 분석
        ↓
숫자와 실제 이미지·영상 연결
```

### 주요 실행 파일

```text
scripts/
├── 06_check_day02_data.py
├── 07_pose_image_detail.py
├── 08_draw_skeleton_quality.py
├── 09_compare_pose_images.py
└── 10_pose_video_quality.py

src/
└── pose_utils.py
```

### 주요 결과 파일

```text
reports/day02/tables/
├── front_keypoints.csv
├── image_quality_summary.csv
├── pose_quality_demo_frame_quality.csv
└── pose_quality_demo_keypoints.csv

reports/day02/challenge/
└── challenge_notes.md
```

### 다음 수업

3일차에는 2일차에서 확인한 Keypoint를 이용하여
관절각도와 정규화된 Frame Feature를 만듭니다.

```text
Keypoint 좌표
+ Confidence
        ↓
신뢰할 수 있는 관절 확인
        ↓
관절각도
+ 정규화
        ↓
Frame Feature
```
````

README에는 코드 전체를 복사하지 않습니다.

```
무엇을 배웠는가?
        ↓
어떤 코드를 사용했는가?
        ↓
어떤 결과가 만들어졌는가?
        ↓
실제 결과에서 무엇을 확인했는가?
        ↓
다음 수업과 어떻게 연결되는가?
```

를 빠르게 확인할 수 있으면 됩니다.

---

# 35. `.gitignore`를 확인하고 2일차 Git Commit을 만듭니다

2일차에는 이미지·영상뿐 아니라 여러 CSV 결과도 생성되었습니다.

이 파일들은 실행할 때 다시 만들 수 있는 결과이므로 Git에는 **코드와 문서 중심으로 저장**합니다.

1일차의 `.gitignore`에 이미지·영상·모델·가상환경 제외 규칙이 이미 있다면 그대로 사용하고, 2일차 CSV 결과를 제외하기 위해 다음 내용을 추가합니다.

```
# Generated Day02 pose tables
reports/day02/tables/*.csv
```

현재 상태를 확인합니다.

```bash
git status
```

다음과 같은 원본 데이터와 생성 결과는 Git에 올라가지 않는 것이 좋습니다.

```
data/
*.jpg
*.mp4
*.pt
reports/day02/tables/*.csv
```

반면 오늘 작성한 다음 파일은 저장해야 합니다.

```
src/pose_utils.py

scripts/06_check_day02_data.py
scripts/07_pose_image_detail.py
scripts/08_draw_skeleton_quality.py
scripts/09_compare_pose_images.py
scripts/10_pose_video_quality.py

README.md
reports/day02/challenge/challenge_notes.md
.gitignore
```

이번에는 무조건 `git add .`부터 실행하지 않고, 먼저 어떤 파일이 등록되는지 확인합니다.

```bash
git status
```

필요한 코드와 문서를 등록합니다.

```bash
git add \
  .gitignore \
  README.md \
  src/pose_utils.py \
  scripts/06_check_day02_data.py \
  scripts/07_pose_image_detail.py \
  scripts/08_draw_skeleton_quality.py \
  scripts/09_compare_pose_images.py \
  scripts/10_pose_video_quality.py \
  reports/day02/challenge/challenge_notes.md
```

다시 확인합니다.

```bash
git status
```

원본 이미지·MP4·CSV가 `Changes to be committed`에 포함되지 않았는지 확인합니다.

정상이라면 Commit합니다.

```bash
git commit -m "feat: add day02 pose keypoint quality analysis"
```

최근 기록을 확인합니다.

```bash
git log --oneline -5
```

1일차 Commit 다음에 2일차 Commit이 추가되어 있으면 됩니다.

### 원격 저장소를 사용하는 경우

먼저 현재 연결 상태를 확인합니다.

```bash
git remote -v
```

이미 `origin`이 있다면 다시 `git remote add`를 하지 않습니다.

기관에서 허용한 원격 저장소를 사용하는 환경이라면:

```bash
git push
```

를 실행합니다.

> 외부 GitHub 사용이 금지된 수업환경에서는 Local Git 또는 기관 내부 GitLab/Gitea까지만 사용합니다. 촬영 원본 이미지와 영상은 저장소에 올리지 않습니다.

---

# 36. 2일차 자가 체크리스트를 확인합니다

오늘 실제로 학습한 내용을 기준으로 하나씩 확인합니다.

- [ ]  Person BBox Confidence와 Keypoint Confidence의 차이를 설명할 수 있다.
- [ ]  COCO 17 Keypoint의 Pixel `(x, y)` 좌표를 읽을 수 있다.
- [ ]  Normalized 좌표가 무엇이고 왜 필요한지 설명할 수 있다.
- [ ]  Keypoint Confidence가 관절 위치를 얼마나 신뢰하는지 나타내는 값이라는 것을 이해했다.
- [ ]  Confidence가 높다고 관절 위치가 반드시 정답이라는 뜻은 아니라는 것을 이해했다.
- [ ]  Threshold가 낮은 Confidence 관절을 사용 대상에서 제외하기 위한 기준이라는 것을 이해했다.
- [ ]  `front.jpg`의 실제 Confidence를 보고 `0.70 / 0.90 / 0.95` Threshold에서 Skeleton이 어떻게 달라지는지 확인했다.
- [ ]  여러 이미지를 비교할 때는 `0.50`을 공통 LOW 기준으로 사용했다는 것을 이해했다.
- [ ]  BBox Area Ratio가 화면에서 사람이 차지하는 크기를 나타낸다는 것을 설명할 수 있다.
- [ ]  6장의 촬영 조건에서 평균 Keypoint Confidence와 LOW 관절 수를 비교할 수 있다.
- [ ]  이번 데이터에서 `side.jpg`가 가장 불안정하게 나타난 이유를 설명할 수 있다.
- [ ]  `front.jpg`와 `distance_far.jpg`를 비교하여 사람 크기와 Pose 품질을 따로 판단할 수 있다.
- [ ]  MP4를 Frame 단위로 분석하여 평균 Keypoint Confidence와 LOW 관절 수를 확인할 수 있다.
- [ ]  CSV에서 불안정한 Frame을 찾고 실제 영상의 해당 시간과 연결할 수 있다.
- [ ]  약 `0.80초` 부근에서 얼굴 Keypoint Confidence가 낮아진 실제 결과를 설명할 수 있다.
- [ ]  Mini Challenge에서 직접 촬영한 두 조건의 Pose 품질을 비교하고 결과를 설명할 수 있다.
- [ ]  README에 2일차 학습 내용과 실제 결과를 정리했다.
- [ ]  2일차 코드와 문서를 Git Commit으로 저장했다.

---

## 다음 시간 — Keypoint를 분석용 Feature로 바꿉니다

2일차에서는 **관절을 찾은 뒤 그 좌표를 얼마나 신뢰할 수 있는지** 확인했습니다.

3일차에는 이 좌표를 실제 동작 분석에서 사용할 수 있는 숫자 구조로 바꿉니다.

```
2일차

COCO 17 Keypoint
        ↓
Pixel / Normalized 좌표
        ↓
Confidence
        ↓
신뢰할 수 있는 Keypoint 확인

        ↓

3일차

관절각도 계산
        ↓
Hip Center 기준 위치 정규화
        ↓
Shoulder Width 기준 크기 정규화
        ↓
34개 정규화 좌표
+
4개 관절각도
        ↓
38-D Frame Feature
```

> **2일차에서 반드시 기억할 내용:**  
> 
> Skeleton이 보인다고 바로 좋은 Pose 결과라고 판단하지 않습니다. 
> Keypoint의 좌표와 Confidence를 함께 확인하고, 신뢰하기 어려운 관절이 있다면 
> 이후 관절각도와 Feature 계산에 그대로 사용하지 않도록 주의해야 합니다.