
**대상:** Python·OpenCV·ROI를 배우고 3일차 규칙 기반 검사까지 완료한 학습자  
**환경:** Windows 11 + WSL2 Ubuntu + VS Code · 기존 `.venv` 재사용  
**입력:** 스마트폰 IP Webcam (`/video` MJPEG 등)  
**검사:** 뚜껑 있음 `OK`, 뚜껑 없음 `NG`, 검사가 불명확하면 `확인 필요`  
**서버:** FastAPI + Uvicorn, 로컬 실습(`127.0.0.1:8000`)

> 이 교재는 기존 프로젝트에 **실시간 입력과 웹 화면**을 추가하는 확장 실습입니다. 1~3일차 파일을 삭제하거나 전체를 새로 만들지 않습니다. 이 코드는 원래 3일차의 *딥러닝이 아닌 OpenCV Rule 검사*를 재사용한 교육용 예제입니다. 실제 검사 성능은 데이터와 촬영조건에 따라 달라집니다.

---

## PART 1. 오늘의 목표 · 핵심 질문

이번 실습의 핵심 질문은 다음과 같습니다.

> **3일차에 이미지 파일로 검사하던 OK/NG 규칙을 IP Webcam의 실시간 Frame에 반복 적용하고, 웹브라우저에 판정 결과를 표시하려면 어떻게 연결해야 할까요?**

오늘의 목표는 다음과 같습니다.

- 1일차의 IP Webcam 연결을 FastAPI에서도 사용할 수 있습니다.
- 브라우저에서 실시간 영상을 보고 **마우스로 검사 영역(ROI)을 선택**할 수 있습니다.
- 2~3일차의 Gray·Otsu·Morphology·Contour·Area Ratio를 새로운 Frame에도 적용할 수 있습니다.
- 기준 이미지의 실제 측정값으로 만든 Rule을 적용하여 OK / NG가 바뀌는 모습을 확인할 수 있습니다.
- 카메라가 끊겼거나 ROI가 정해지지 않은 경우 정상적인 검사 결과로 오해하지 않도록 구분할 수 있습니다.

### 한눈에 보는 연결 구조

![[Group 39.png]]

```text
[1일차] 스마트폰 IP Webcam
           ↓  실시간 Frame
    FastAPI 서버에서 영상 수신
           ↓
    웹 화면에서 마우스로 ROI 지정
           ↓
    Frame의 해당 영역만 Crop
           ↓
[2일차] Gray → Otsu 이진화
           ↓
[3일차] Opening → Closing → Contour
           ↓
    가장 큰 객체의 Area Ratio 측정
           ↓
    cap_on / cap_off로 정한 기준값과 비교
           ↓
       OK / NG / 확인 필요
           ↓
       FastAPI → 웹 화면에 갱신
```

**왜 FastAPI만으로 판정하지 않을까요?** FastAPI는 웹 요청과 영상·검사 결과 전달을 담당합니다. **판단 자체는 3일차 OpenCV Rule 코드**가 합니다. 새로운 AI 학습이나 `.pt` 모델은 필요하지 않습니다.

### IP Webcam은 어떤 방식으로 통신하나요?

![[Pasted image 20261025112045.png|430]]

OpenCV가 스마트폰의 영상 주소로 접속해 영상을 읽어오는 방식입니다.

### HTTP 요청(Request)이란?

HTTP 요청(Request)은 클라이언트가 서버에게 필요한 데이터나 기능을 요청하는 것입니다.

쉽게 말해, 클라이언트가 서버에게 "이 데이터를 보내주세요" 또는 "이 작업을 처리해 주세요"라고 요청하는 과정입니다.

예를 들어, 이번 병뚜껑 검사 실습에서는 스마트폰의 IP Webcam이 영상 데이터를 제공하고, Python의 OpenCV가 HTTP 요청을 통해 영상을 받아옵니다.

```
[클라이언트: Python OpenCV]
          │
          │ HTTP 요청(Request)
          │ "카메라 영상을 보내주세요."
          ▼
[서버: 스마트폰 IP Webcam]
          │
          │ HTTP 응답(Response)
          │ "영상 데이터를 보내드립니다."
          ▼
[클라이언트: Python OpenCV]
          │
          ▼
     영상 Frame 수신
```

핵심 용어

- 클라이언트(Client): 데이터나 작업을 요청하는 쪽 (예: 크롬 브라우저)
- 서버(Server): 요청을 받아 처리하고 응답하는 쪽 (예: FastAPI로 만든 서버)
- 요청(Request): 클라이언트가 HTTP 통신 방식을 이용하여 서버(FastAPI)에 데이터나 작업을 요청하는 것
- 응답(Response): 서버(FastAPI)가 요청을 처리한 결과를 HTTP 응답으로 브라우저에 전달하는 것
    

이번 실습에서는 HTTP 요청으로 영상 연결을 시작하고, MJPEG 스트리밍을 통해 하나의 HTTP 응답에서 영상 프레임을 연속적으로 전달받습니다.

즉, HTTP는 데이터를 주고받기 위한 통신 규칙이며, FastAPI는 이러한 HTTP 요청을 처리하는 Python 웹 프레임워크입니다.

### HTTP인데 어떻게 실시간 영상이 가능한가요?

일반적인 HTTP 요청은 요청 한 번에 응답을 한 번 받고 종료되는 경우가 많습니다. 하지만 HTTP는 응답 연결을 유지하면서 데이터를 계속 전송할 수도 있습니다.

현재 실습에 적용한 MJPEG가 바로 이런 방식입니다.

|구분|현재 실습에서의 동작|
|---|---|
|스마트폰 → Python|HTTP 연결을 통해 MJPEG 프레임을 연속 수신|
|Python → 브라우저 영상|HTTP MJPEG 스트리밍|
|Python → 브라우저 판정값|HTTP GET으로 0.4초마다 조회|
|브라우저 → Python ROI 좌표|HTTP POST로 JSON 전송|
|WebSocket|사용하지 않음|

실제 코드에서도 다음과 같이 확인됩니다.

```python
cap = cv2.VideoCapture(url)
```

OpenCV가 스마트폰의 HTTP 영상 URL에서 프레임을 읽습니다.

```python
return StreamingResponse(
    frames(),
    media_type="multipart/x-mixed-replace; boundary=frame"
)
```

---

## PART 2. 실제 사용할 데이터와 프로젝트 확인

### 2.1 기준 데이터는 3일차에서 사용한 두 장입니다

```text
samples/day03/reference/
├── cap_on.jpg   ← 병뚜껑 있음: 정상 기준 ROI 이미지
└── cap_off.jpg  ← 병뚜껑 없음: 불량 기준 ROI 이미지
```

이 파일은 1일차 IP Webcam으로 촬영한 ROI를 2일차와 3일차로 넘겨 사용한 **실제 기준 이미지**입니다. 둘 다 이미 잘라 놓은 ROI이므로, 기준 이미지를 읽을 때 또 다른 Crop을 하지 않습니다. 반면 **실시간 카메라 Frame은 전체 영상이므로 브라우저에서 새로 ROI를 지정**해야 합니다.

기준 파일을 아직 복사하지 않았다면 (실제로 존재하는 경로에서만) 다음 명령을 실행합니다.

```bash
mkdir -p samples/day03/reference
cp samples/day02/cap_on_reference.jpg samples/day03/reference/cap_on.jpg
cp samples/day02/cap_off_reference.jpg samples/day03/reference/cap_off.jpg
```

실제 기존 프로젝트에 맞는 위치를 먼저 확인하세요. 실습 파일의 경로가 다르면 `cp`가 실패할 수 있습니다. **기준 이미지는 반드시 같은 병·같은 ROI 구도에서 뚜껑 유무만 다르게 촬영**한 것이어야 합니다.

### 2.2 3일차 규칙은 무엇이었나요?

```text
cap_on.jpg → Area Ratio 측정 → cap_on_ratio
cap_off.jpg → Area Ratio 측정 → cap_off_ratio
                            ↓
candidate_threshold = (cap_on_ratio + cap_off_ratio) / 2
                            ↓
cap_on_ratio가 더 큰 조건이라면:
    새 Frame의 Ratio >= 기준값 → OK
    새 Frame의 Ratio <  기준값 → NG
```

예전 교재의 측정 **예시**는 다음과 같습니다.

| 기준 이미지 | Area Ratio 예시 |
|---|---:|
| `cap_on.jpg` | 0.661 |
| `cap_off.jpg` | 0.459 |
| 후보 기준 | 약 0.560 |

**주의:** `0.560`을 코드에 고정하지 않습니다. 서버를 시작할 때 실제 두 파일에서 측정한 값을 계산합니다. 또한 실제 데이터에서 `cap_on_ratio`가 더 작은 경우도 처리하도록 3일차 코드를 그대로 반영합니다. 서로 거의 구분되지 않는 경우 기준값을 만들지 않고 촬영·ROI를 다시 확인하도록 합니다.

### 2.3 최종 프로젝트에서 추가되는 부분

```text
subject9_machinevision/       ← 기존 폴더 그대로 사용
├── .venv/                    ← 기존 가상환경 재사용
├── samples/
│   └── day03/reference/
│       ├── cap_on.jpg
│       └── cap_off.jpg
├── src/
│   ├── 00_check_env.py       ← 기존 파일 유지
│   ├── ...
│   ├── 21_cap_rule.py        ← 기존 파일 유지
│   ├── 22_cap_batch_test.py  ← 기존 파일 유지
│   ├── __init__.py           ← 새 파일(빈 파일)
│   ├── cap_inspector.py      ← 새 파일: 공통 검사 함수
│   └── fastapi_cap_app.py    ← 새 파일: IP Webcam/FastAPI 서버
├── web/
│   └── cap_inspection.html   ← 새 파일: ROI 선택/검사 화면
├── tests/                    ← 선택: 검증 테스트
│   └── test_inspector.py
└── requirements.txt         ← 기존 파일 유지·업데이트
```

원래의 `08_phone_camera_roi.py`와 `21_cap_rule.py`는 수정하지 않습니다. 카메라 입력과 Rule 함수를 **새 파일에서 연결**합니다.

---

## PART 3. 개발환경과 실행 준비

### 3.1 WSL2에서 현재 프로젝트로 이동합니다

사용자 환경에서 `subject9_machinevision`이 홈 폴더에 있다면:

```bash
cd ~/subject9_machinevision
```

프로젝트가 `~/ai_vision/subject9_machinevision`에 있다면 대신 해당 경로로 이동합니다. **두 경로를 모두 실행할 필요는 없습니다.**

```bash
pwd
ls src
```

`pwd`의 마지막 폴더가 `subject9_machinevision`이고 `src/21_cap_rule.py`가 보이면 정상입니다.

### 3.2 기존 가상환경을 활성화합니다

```bash
source .venv/bin/activate
python src/00_check_env.py
```

오류 없이 `Environment OK`가 출력되어야 합니다. 여기서는 `.venv`를 다시 만들지 않습니다.

### 3.3 FastAPI와 Uvicorn만 추가로 설치합니다

```bash
python -m pip install fastapi uvicorn
python -m pip freeze > requirements.txt
python -c "import fastapi, uvicorn, cv2; print('추가 라이브러리 확인 완료')"
```

`FastAPI`는 웹서버에서 요청을 처리하는 Python 프레임워크이고, `Uvicorn`은 그 애플리케이션을 실제로 실행하는 서버입니다. OpenCV와 NumPy는 앞선 교과의 설치 환경을 그대로 사용합니다.

![[Pasted image 20261025113054.png|400]]

### Uvicorn(유비콘)이란?

Uvicorn은 FastAPI로 만든 웹 애플리케이션을 실행하고, 클라이언트의 HTTP 요청을 받아 FastAPI에 전달하는 웹서버(ASGI 서버)입니다.

쉽게 구분하면 다음과 같습니다.

- FastAPI: 클라이언트의 요청을 어떻게 처리할지 정의하는 웹 프레임워크
    
- Uvicorn: FastAPI 애플리케이션을 실행하고 HTTP 요청을 받을 수 있게 해주는 서버 프로그램

### 3.4 필요한 폴더를 준비합니다

```bash
mkdir -p web src tests
# 기존 src/__init__.py가 없다면 빈 파일 생성
[ -f src/__init__.py ] || touch src/__init__.py
ls samples/day03/reference/cap_on.jpg samples/day03/reference/cap_off.jpg
```

마지막 `ls`에서 두 파일이 모두 확인되어야 합니다.

### `__init__.py` 파일을 만드는 이유는?

`__init__.py`는 Python에게 해당 폴더를 패키지(Package)로 사용한다는 것을 알려주는 파일입니다.

쉽게 말해, 폴더 안에 있는 여러 Python 파일을 다른 Python 파일에서 불러와 사용할 수 있도록 구조를 명확하게 만드는 역할을 합니다.

### 현재 실습에서 왜 필요한가요?

```
subject9_machinevision/
│
├── src/
│   ├── __init__.py          ← 패키지 표시
│   ├── cap_inspector.py     ← 병뚜껑 검사 함수
│   └── fastapi_cap_app.py   ← FastAPI 서버
```

`__init__.py`가 있으면 Python은 `src`를 하나의 패키지로 명확하게 인식합니다.

따라서 다른 파일에서 다음과 같이 함수를 가져올 수 있습니다.

```python
from src.cap_inspector import inspect_roi
```

- `__init__.py` → 파일들이 들어 있는 폴더를 하나의 패키지로 표시
    
- `from ... import ...` → 그 패키지 안의 필요한 코드를 가져와 사용

즉, `__init__.py`는 패키지 표시, `import`는 코드 연결이라고 기억하시면 됩니다.

---

## PART 4. 코드 작성 ① — 1~3일차 병뚜껑 검사 코드를 함수로 만듭니다

### 4.1 이번 코드는 무엇을 하나요?

1~3일차에서는 스마트폰 IP Webcam으로 병뚜껑 이미지를 촬영하고, 검사할 영역(ROI)을 선택한 뒤 OpenCV를 이용하여 병뚜껑의 유무를 판정하는 방법을 배웠습니다.

이번에는 **1~3일차에서 배운 검사 과정을 하나의 함수로 만들어 재사용**하겠습니다.

지금까지는 준비된 이미지를 검사했다면, 앞으로는 스마트폰에서 실시간으로 들어오는 영상의 각 프레임에 동일한 검사를 반복 적용해야 합니다.

이를 위해 **이미지 한 장을 입력받아 병뚜껑의 유무를 검사하고, OK / NG / 확인 필요 결과를 반환하는 함수**를 만들겠습니다.

이번 코드는 1~3일차에서 배운 OpenCV 병뚜껑 검사 과정을 함수로 만들어, 실시간 영상에서도 반복해서 사용할 수 있도록 준비하는 코드입니다.

이렇게 함수를 만들어 두면 FastAPI에서도 같은 검사 코드를 반복해서 사용할 수 있습니다.

`src/cap_inspector.py` 파일을 새로 생성합니다.

### 4.2 코드의 전체 처리 흐름

```text
[기준값 준비]
cap_on.jpg + cap_off.jpg
        ↓
각 이미지의 Area Ratio 측정
        ↓
두 측정값으로 판정 기준값 생성
        ↓
기준값 저장
        │
        ▼
[새로운 이미지 검사]
ROI 이미지 입력
        ↓
Gray → Otsu 이진화
        ↓
Opening → Closing
        ↓
Contour 검출
        ↓
가장 큰 객체의 Area Ratio 측정
        ↓
기준값과 비교
        ↓
OK / NG / 확인 필요
        ↓
검사 결과 반환
```

`cap_inspector.py`가 이번 실시간 병뚜껑 검사 시스템의 핵심 검사 로직(Business Logic)을 담당하는 코드입니다.

역할을 구분하면 다음과 같습니다.

- `cap_inspector.py` → 핵심 검사 로직: OpenCV를 이용해 병뚜껑의 면적 비율을 측정하고 OK / NG / 확인 필요를 판정합니다.
    
- `fastapi_cap_app.py` → 서버 및 연결 로직: 스마트폰 영상을 받아 검사 함수를 호출하고, 결과를 웹브라우저에 전달합니다.
    
- `cap_inspection.html` → 사용자 화면(UI): 실시간 영상을 표시하고 마우스로 ROI를 선택하며 검사 결과를 보여줍니다.

### 완성 코드 — `src/cap_inspector.py`

```python

"""1~3일차 OpenCV 병뚜껑 검사 로직을 재사용 가능한 함수로 구성합니다."""


# ============================================================
# ① 필요한 라이브러리 불러오기
# - dataclass : 검사 기준값을 하나로 묶어서 저장
# - Path      : 이미지 파일 경로 처리
# - cv2       : 이미지 전처리 및 Contour 검출
# - numpy     : 이미지 배열 처리
# ============================================================

from dataclasses import dataclass
from pathlib import Path

import cv2
import numpy as np


# ============================================================
# ② 최소 객체 면적 기준 설정
# - ROI 전체 면적의 0.1%보다 작은 Contour는 제외합니다.
# - 너무 작은 노이즈가 검사에 포함되지 않도록 합니다.
# ============================================================

MIN_CONTOUR_AREA_RATIO = 0.001


# ============================================================
# ③ 병뚜껑 검사 기준값을 저장할 구조 만들기
# - cap_on_ratio  : 뚜껑이 있는 기준 이미지의 면적 비율
# - cap_off_ratio : 뚜껑이 없는 기준 이미지의 면적 비율
# - threshold     : 정상과 불량을 구분할 기준값
# - cap_present_when_greater :
#   뚜껑이 있을 때 면적 비율이 더 큰지 저장
#
# frozen=True : 생성 후 기준값을 변경하지 못하게 합니다.
# ============================================================

@dataclass(frozen=True)
class CapRule:
    cap_on_ratio: float
    cap_off_ratio: float
    threshold: float
    cap_present_when_greater: bool


# ============================================================
# ④ 입력 ROI 이미지가 정상인지 확인
# - ROI는 검사할 영역만 잘라 놓은 이미지입니다.
# - 비어 있거나 BGR 컬러 이미지가 아니면 오류 발생
# ============================================================

def measure_area_ratio(roi: np.ndarray) -> tuple[float, np.ndarray | None]:
    """ROI에서 가장 큰 Contour의 면적 비율을 계산합니다."""

    if roi is None or roi.size == 0 or roi.ndim != 3 or roi.shape[2] != 3:
        raise ValueError("ROI는 비어 있지 않은 BGR 이미지여야 합니다.")


    # ========================================================
    # ⑤ Gray → Otsu 이진화
    # - BGR 컬러 이미지를 Gray로 변환합니다.
    # - Otsu를 이용하여 자동으로 이진화 기준값을 정합니다.
    # - BINARY_INV를 적용하여 흑백을 반전합니다.
    # ========================================================

    gray = cv2.cvtColor(roi, cv2.COLOR_BGR2GRAY)

    _, binary = cv2.threshold(
        gray, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU
    )


    # ========================================================
    # ⑥ Opening → Closing으로 이진 이미지 정리
    # - Opening : 작은 노이즈 제거
    # - Closing : 객체 내부의 작은 틈이나 구멍 메우기
    # - 3×3 타원형 Kernel 사용
    # ========================================================

    kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (3, 3))

    opened = cv2.morphologyEx(
        binary, cv2.MORPH_OPEN, kernel, iterations=1
    )

    cleaned = cv2.morphologyEx(
        opened, cv2.MORPH_CLOSE, kernel, iterations=1
    )


    # ========================================================
    # ⑦ Contour 검출 및 작은 객체 제외
    # - 정리된 이진 이미지에서 외곽 윤곽선을 찾습니다.
    # - ROI 전체 면적을 계산합니다.
    # - ROI 면적의 0.1% 미만인 Contour는 제외합니다.
    # ========================================================

    contours, _ = cv2.findContours(
        cleaned.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE
    )

    height, width = roi.shape[:2]
    roi_area = float(height * width)

    min_area = roi_area * MIN_CONTOUR_AREA_RATIO

    valid = [
        c for c in contours
        if cv2.contourArea(c) >= min_area
    ]


    # ========================================================
    # ⑧ 가장 큰 객체의 Area Ratio 계산
    # - 유효한 Contour가 없으면 0.0과 None을 반환합니다.
    # - 가장 큰 Contour를 선택합니다.
    # - Contour 면적을 ROI 전체 면적으로 나눕니다.
    # ========================================================

    if not valid:
        return 0.0, None

    largest = max(valid, key=cv2.contourArea)

    return float(cv2.contourArea(largest)) / roi_area, largest


# ============================================================
# ⑨ 기준 이미지 2장 불러오기
# - cap_on.jpg  : 병뚜껑이 있는 이미지
# - cap_off.jpg : 병뚜껑이 없는 이미지
# - 파일이 없거나 두 이미지의 크기가 다르면 오류 발생
# ============================================================

def load_rule(reference_dir: Path) -> CapRule:
    """기준 이미지 2장에서 병뚜껑 판정 기준값을 만듭니다."""

    on_path = reference_dir / "cap_on.jpg"
    off_path = reference_dir / "cap_off.jpg"

    cap_on = cv2.imread(str(on_path))
    cap_off = cv2.imread(str(off_path))

    if cap_on is None or cap_off is None:
        raise FileNotFoundError(
            "기준 파일을 확인하세요: "
            f"{on_path} / {off_path}"
        )

    if cap_on.shape[:2] != cap_off.shape[:2]:
        raise ValueError(
            "cap_on.jpg와 cap_off.jpg의 ROI 크기가 서로 다릅니다."
        )


    # ========================================================
    # ⑩ 두 기준 이미지의 Area Ratio 측정
    # - 앞에서 만든 measure_area_ratio() 함수를 재사용합니다.
    # - 두 이미지의 면적 비율을 각각 계산합니다.
    # - 차이가 너무 작으면 구분하기 어려우므로 오류 발생
    # ========================================================

    on_ratio, _ = measure_area_ratio(cap_on)
    off_ratio, _ = measure_area_ratio(cap_off)

    if abs(on_ratio - off_ratio) < 0.01:
        raise ValueError(
            "기준 이미지의 Area Ratio 차이가 너무 작습니다. "
            "촬영·ROI·조명 조건을 다시 확인하세요."
        )


    # ========================================================
    # ⑪ 병뚜껑 판정 기준값 생성 및 저장
    # - 두 기준 이미지의 Area Ratio 중간값을 계산합니다.
    # - 뚜껑이 있을 때 비율이 더 큰지도 확인합니다.
    # - 계산한 값을 CapRule에 저장하여 반환합니다.
    # ========================================================

    return CapRule(
        cap_on_ratio=on_ratio,
        cap_off_ratio=off_ratio,
        threshold=(on_ratio + off_ratio) / 2.0,
        cap_present_when_greater=(on_ratio > off_ratio),
    )


# ============================================================
# ⑫ 새로운 이미지의 Area Ratio를 기준값과 비교
# - 뚜껑이 있을 때 비율이 큰 경우와 작은 경우를 구분합니다.
# - 기준값과 비교하여 OK 또는 NG를 반환합니다.
# ============================================================

def judge_cap(area_ratio: float, rule: CapRule) -> tuple[str, str]:
    """측정된 면적 비율을 기준값과 비교하여 판정합니다."""

    if rule.cap_present_when_greater:
        cap_present = area_ratio >= rule.threshold
    else:
        cap_present = area_ratio <= rule.threshold

    return ("OK", "") if cap_present else ("NG", "NG_CAP_AREA")


# ============================================================
# ⑬ 새로운 ROI의 최종 검사 결과 반환
# - 새로운 ROI 이미지가 들어오면 Area Ratio를 측정합니다.
# - 유효한 Contour가 없으면 CHECK(확인 필요)를 반환합니다.
# - Contour가 있으면 ⑫의 judge_cap()으로 판정합니다.
# - 판정 결과, 원인, 면적 비율을 딕셔너리로 반환합니다.
# ============================================================

def inspect_roi(roi: np.ndarray, rule: CapRule) -> dict:
    """새로운 ROI를 검사하고 최종 판정 결과를 반환합니다."""

    ratio, contour = measure_area_ratio(roi)

    # 유효한 Contour가 없으면 정상/불량을 단정하지 않습니다.
    if contour is None:
        return {
            "decision": "CHECK",
            "reason": "ROI_CONTOUR_NOT_FOUND",
            "ratio": ratio
        }

    # 기준값과 비교하여 OK 또는 NG를 결정합니다.
    decision, reason = judge_cap(ratio, rule)

    return {
        "decision": decision,
        "reason": reason,
        "ratio": ratio
    }
```

### 의사코드

```
[① 필요한 라이브러리 불러오기]
OpenCV, NumPy 등 필요한 기능 준비
        ↓
[② 최소 객체 면적 기준 설정]
너무 작은 객체를 검사에서 제외할 기준 설정
        ↓
[③ 병뚜껑 판정 기준을 저장할 구조 만들기]
cap_on 비율, cap_off 비율, 기준값 등 보관
        ↓
[④ 입력 ROI 이미지 확인]
이미지가 비어 있거나 잘못된 형식이면 오류 발생
        ↓
[⑤ Gray → Otsu 이진화]
컬러 이미지를 흑백으로 변환하고
객체와 배경을 구분
        ↓
[⑥ Opening → Closing]
작은 노이즈 제거 및 객체 영역 정리
        ↓
[⑦ Contour 검출 및 작은 객체 제외]
윤곽선을 찾고 너무 작은 객체는 제외
        ↓
[⑧ 가장 큰 객체의 Area Ratio 계산]
가장 큰 Contour 면적 ÷ ROI 전체 면적
        ↓
[⑨ 기준 이미지 2장 불러오기]
cap_on.jpg / cap_off.jpg
        ↓
[⑩ 두 기준 이미지의 면적 비율 측정]
cap_on_ratio / cap_off_ratio 계산
두 값의 차이가 충분한지 확인
        ↓
[⑪ 판정 기준값 생성 및 저장]
두 비율의 중간값을 Threshold로 설정
병뚜껑이 있을 때 비율이 커지는지도 저장
        ↓
[⑫ 새로운 이미지의 판정값 비교]
측정값과 기준값 비교
        ↓
     OK 또는 NG
        ↓
[⑬ 최종 검사 결과 반환]
새로운 ROI의 Area Ratio 측정
        ↓
유효한 Contour가 없다면 → CHECK(확인 필요)
유효한 Contour가 있다면 → ⑫의 판정 적용
        ↓
검사 결과와 Area Ratio 반환
```

### 4.3 이해해야 할 핵심 함수 4개

| 함수                     | 역할                             |
| ---------------------- | ------------------------------ |
| `measure_area_ratio()` | ROI 이미지에서 가장 큰 객체의 면적 비율을 측정   |
| `load_rule()`          | 정상·불량 기준 이미지 2장으로 판정 기준값 생성    |
| `judge_cap()`          | 측정된 면적 비율을 기준값과 비교해 OK / NG 판정 |
| `inspect_roi()`        | 새로운 ROI를 검사하고 최종 결과 반환         |

---

## PART 5. 코드 작성 ② — IP Webcam을 읽고 FastAPI로 영상을 전달합니다

### 5.1 이번 코드는 무엇을 하나요?

`src/fastapi_cap_app.py`를 만들겠습니다. 이 파일은 세 가지 역할을 맡습니다.

1. 스마트폰의 IP Webcam 영상 URL로 연결해서 Frame을 계속 읽습니다.
2. 웹에서 선택한 ROI를 Frame에 적용한 뒤 `cap_inspector.py`를 이용해 검사합니다.
3. 실시간 JPEG 영상(MJPEG)과 최신 검사 상태를 웹브라우저에 전달합니다.

**한 가지 주의:** 웹브라우저가 두 개 열려 있어도 카메라를 두 번 열지 않도록 **백그라운드 카메라 스레드 1개**에서 Frame을 읽고, 다른 브라우저들은 최신 결과만 함께 보도록 만들었습니다. 수업에서는 Uvicorn을 `--workers 1`로 실행합니다.

### 5.2 브라우저와 연결하는 URL의 의미

| 주소 | 방법 | 역할 |
|---|---|---|
| `/` | GET | HTML 검사 화면 제공 |
| `/api/camera` | POST | 스마트폰 IP Webcam URL 연결 |
| `/api/roi` | POST | 마우스로 고른 ROI 좌표 서버에 전달 |
| `/api/roi` | DELETE | 현재 ROI 초기화 |
| `/api/reload-rule` | POST | 기준 이미지 2장을 다시 읽어 Rule 갱신 |
| `/api/status` | GET | 최신 OK/NG, Area Ratio, 기준값 반환 |
| `/stream.mjpg` | GET | 실시간 JPEG Frame을 영상처럼 전송 |

FastAPI의 Request Body는 Pydantic으로 검증합니다. 따라서 ROI의 좌표 범위가 0~1이 아니거나 크기가 너무 작으면 서버가 오류로 응답합니다.

### 완성 코드 — `src/fastapi_cap_app.py`

```python

"""스마트폰 IP Webcam → OpenCV 검사 → FastAPI 실시간 웹 화면."""

# ① 라이브러리 가져오기 · 프로젝트 경로 지정
# FastAPI, OpenCV, Pydantic 및 검사 모듈을 불러옵니다.
from contextlib import asynccontextmanager
import ipaddress
from pathlib import Path
import threading
import time
from urllib.parse import urlsplit

import cv2
import numpy as np
from fastapi import FastAPI, HTTPException
from fastapi.responses import FileResponse, StreamingResponse
from pydantic import BaseModel, Field

from src.cap_inspector import CapRule, inspect_roi, load_rule

ROOT = Path(__file__).resolve().parents[1]
REFERENCE_DIR = ROOT / "samples" / "day03" / "reference"
HTML_FILE = ROOT / "web" / "cap_inspection.html"


# ② 대기 화면 이미지 준비
# 스마트폰 영상이 연결되기 전 보여 줄 JPEG를 만듭니다.
def placeholder_jpg() -> bytes:
    frame = np.full((480, 640, 3), (38, 43, 50), dtype=np.uint8)
    cv2.putText(frame, "IP WEBCAM - WAITING", (65, 227),
                cv2.FONT_HERSHEY_SIMPLEX, 0.9, (230, 230, 230), 2)
    return cv2.imencode(".jpg", frame)[1].tobytes()


WAITING_JPG = placeholder_jpg()


# ③ 실시간 검사 상태를 관리하는 클래스
# 카메라 주소, ROI, 기준값, 판정 결과, 최신 프레임 등을 보관합니다.
class LiveState:
    """한 대의 카메라를 단일 스레드에서 읽고 여러 브라우저에 공유합니다."""

    def __init__(self) -> None:
        self.lock = threading.Lock()
        self.url: str | None = None
        self.roi: tuple[float, float, float, float] | None = None
        self.rule: CapRule | None = None
        self.rule_error = "기준 이미지 읽기 전"
        self.camera_status = "NOT_CONNECTED"
        self.message = "스마트폰 IP Webcam 주소를 연결하세요."
        self.decision = "WAIT"
        self.reason = ""
        self.ratio: float | None = None
        self.frame_size: list[int] | None = None
        self.latest_jpg: bytes = WAITING_JPG
        self.last_frame_time = 0.0
        self.stop_event = threading.Event()
        self.thread: threading.Thread | None = None

    # ④ 기준 이미지로 검사 규칙 준비/갱신
    # cap_on·cap_off 이미지에서 판정 기준값을 계산합니다.
    def reload_rule(self) -> dict:
        try:
            rule = load_rule(REFERENCE_DIR)
        except (ValueError, FileNotFoundError) as exc:
            with self.lock:
                self.rule = None
                self.rule_error = str(exc)
                self.decision = "WAIT"
            return {"ok": False, "error": str(exc)}
        with self.lock:
            self.rule = rule
            self.rule_error = ""
            self.decision = "WAIT"
        return {"ok": True, "threshold": rule.threshold,
                "cap_on_ratio": rule.cap_on_ratio,
                "cap_off_ratio": rule.cap_off_ratio}

    # ⑤ 카메라 작업 시작·종료
    # 별도의 스레드를 시작하고 서버 종료 시 정리합니다.
    def start(self) -> None:
        self.stop_event.clear()
        self.reload_rule()
        self.thread = threading.Thread(target=self.camera_loop, daemon=True)
        self.thread.start()

    def stop(self) -> None:
        self.stop_event.set()
        if self.thread is not None:
            self.thread.join(timeout=3)

    # ⑥ 카메라 주소와 ROI 선택값 저장
    # 카메라를 바꾸면 기존 ROI와 검사 결과를 초기화합니다.
    def set_camera(self, url: str) -> None:
        with self.lock:
            self.url = url
            self.roi = None  # 새 영상에 이전 ROI를 적용하지 않습니다.
            self.camera_status = "CONNECTING"
            self.message = "카메라 연결 중입니다."
            self.decision = "WAIT"
            self.ratio = None
            self.latest_jpg = WAITING_JPG
            self.last_frame_time = 0.0

    def set_roi(self, roi: tuple[float, float, float, float] | None) -> None:
        with self.lock:
            self.roi = roi
            self.decision = "WAIT"
            self.ratio = None
            self.reason = ""

    # ⑦ 현재 검사 상태 조회
    # 카메라 연결, 판정값, Area Ratio, 기준값 등을 반환합니다.
    def get_status(self) -> dict:
        with self.lock:
            elapsed = (time.monotonic() - self.last_frame_time
                       if self.last_frame_time else None)
            camera_status = self.camera_status
            if camera_status == "ONLINE" and elapsed is not None and elapsed > 5:
                camera_status = "DISCONNECTED"
            rule = self.rule
            return {
                "camera_status": camera_status,
                "message": self.message,
                "decision": self.decision if camera_status == "ONLINE" else "WAIT",
                "reason": self.reason,
                "ratio": self.ratio,
                "roi": self.roi,
                "frame_size": self.frame_size,
                "rule_ready": rule is not None,
                "rule_error": self.rule_error,
                "threshold": round(rule.threshold, 5) if rule else None,
                "on_ratio": round(rule.cap_on_ratio, 5) if rule else None,
                "off_ratio": round(rule.cap_off_ratio, 5) if rule else None,
            }

    # ⑧ IP Webcam 연결 및 연결 실패 처리
    # OpenCV로 스마트폰 영상 URL에 접속하고 재연결을 시도합니다.
    def camera_loop(self) -> None:
        while not self.stop_event.is_set():
            with self.lock:
                url = self.url
            if not url:
                self.stop_event.wait(0.1)
                continue

            # 한 스레드에서만 VideoCapture를 엽니다.
            cap = cv2.VideoCapture(url)
            if not cap.isOpened():
                cap.release()
                with self.lock:
                    if self.url == url:
                        self.camera_status = "DISCONNECTED"
                        self.message = "IP Webcam 연결 실패: URL과 Wi-Fi를 확인하세요."
                self.stop_event.wait(1.0)
                continue

            try:
                while not self.stop_event.is_set():
                    with self.lock:
                        if self.url != url:
                            break
                        roi_coords = self.roi
                        rule = self.rule

                    # ⑨ 실시간 Frame 읽기
                    # 새 프레임을 읽고, 실패하면 상태를 갱신한 뒤 재연결합니다.
                    ok, frame = cap.read()
                    if not ok or frame is None:
                        with self.lock:
                            if self.url == url:
                                self.camera_status = "DISCONNECTED"
                                self.message = "프레임 수신 중단: 재연결을 시도합니다."
                                self.decision = "WAIT"
                                self.latest_jpg = WAITING_JPG
                        break

                    h, w = frame.shape[:2]
                    image = frame.copy()
                    decision = "WAIT"
                    reason = ""
                    ratio = None

                    # ⑩ ROI Crop 후 핵심 검사 함수 호출
                    # 선택한 영역을 잘라 cap_inspector.inspect_roi()로 판정합니다.
                    if roi_coords is not None:
                        nx1, ny1, nx2, ny2 = roi_coords
                        x1, x2 = round(nx1 * w), round(nx2 * w)
                        y1, y2 = round(ny1 * h), round(ny2 * h)
                        x1, x2 = max(0, x1), min(w, x2)
                        y1, y2 = max(0, y1), min(h, y2)
                        if x2 > x1 and y2 > y1:
                            region = frame[y1:y2, x1:x2]
                            if rule is not None:
                                try:
                                    result = inspect_roi(region, rule)
                                    decision = result["decision"]
                                    reason = result["reason"]
                                    ratio = result["ratio"]
                                except (ValueError, cv2.error):
                                    decision = "CHECK"
                                    reason = "ROI_PROCESS_FAILED"
                            color = ((0, 190, 0) if decision == "OK"
                                     else (0, 0, 225) if decision == "NG"
                                     else (0, 200, 255))
                            cv2.rectangle(image, (x1, y1), (x2, y2), color, 3)
                    else:
                        cv2.putText(image, "SELECT ROI ON WEB PAGE", (15, 40),
                                    cv2.FONT_HERSHEY_SIMPLEX, 0.75, (0, 230, 255), 2)

                    # ⑪ 검사 화면 표시 · JPEG 변환 · 최신 결과 저장
                    # 판정 문자와 ROI 표시가 포함된 Frame을 브라우저용으로 준비합니다.
                    line = f"{decision}  Area Ratio: {ratio:.3f}" if ratio is not None else decision
                    cv2.putText(image, line, (12, h - 18),
                                cv2.FONT_HERSHEY_SIMPLEX, 0.72, (0, 245, 255), 2)
                    success, encoded = cv2.imencode(
                        ".jpg", image, [cv2.IMWRITE_JPEG_QUALITY, 77]
                    )
                    if success:
                        with self.lock:
                            # 웹에서 ROI/카메라를 방금 변경했다면 이전 프레임 결과는 폐기합니다.
                            if self.url == url and self.roi == roi_coords and self.rule is rule:
                                self.latest_jpg = encoded.tobytes()
                                self.camera_status = "ONLINE"
                                self.message = "영상 수신 중"
                                self.decision = decision
                                self.reason = reason
                                self.ratio = ratio
                                self.frame_size = [w, h]
                                self.last_frame_time = time.monotonic()
            finally:
                cap.release()
            self.stop_event.wait(0.1)


# ⑫ 서버 실행 준비 및 종료 처리(Lifespan)
# FastAPI가 시작되면 카메라 스레드를 시작하고, 종료 시 중지합니다.
state = LiveState()


@asynccontextmanager
async def lifespan(_: FastAPI):
    state.start()
    yield
    state.stop()


app = FastAPI(title="교과 9 병뚜껑 실시간 검사", lifespan=lifespan)


# ⑬ HTTP 요청 Body 검증(Pydantic)
# 카메라 URL과 ROI 좌표의 자료형·허용 범위를 확인합니다.
class CameraRequest(BaseModel):
    url: str


class ROIRequest(BaseModel):
    x1: float = Field(ge=0.0, le=1.0)
    y1: float = Field(ge=0.0, le=1.0)
    x2: float = Field(ge=0.0, le=1.0)
    y2: float = Field(ge=0.0, le=1.0)


# ⑭ GET / : 브라우저에 HTML 화면 전달
@app.get("/")
def index():
    return FileResponse(HTML_FILE, media_type="text/html")


# ⑮ POST /api/camera : 카메라 URL 확인 후 연결 시도
# HTTP 형식, IP 주소, 포트 등을 검사합니다.
@app.post("/api/camera")
def connect_camera(body: CameraRequest):
    url = body.url.strip()
    parts = urlsplit(url)
    if parts.scheme not in ("http", "https") or not parts.hostname:
        raise HTTPException(400, "http(s)://IP:PORT/video 형식으로 입력하세요.")
    if parts.username or parts.password:
        raise HTTPException(400, "사용자 인증이 없는 교육용 IP Webcam 주소만 사용하세요.")
    try:
        addr = ipaddress.ip_address(parts.hostname)
        if not (addr.is_private or addr.is_loopback):
            raise ValueError("사설 IP 주소가 아닙니다")
        _ = parts.port  # 숫자가 아닌 포트는 거부
    except ValueError:
        raise HTTPException(400, "스마트폰의 사설 IP 주소(예: 192.168.x.x)를 사용하세요.")
    state.set_camera(url)
    return {"ok": True, "message": "연결을 시도합니다."}


# ⑯ POST /api/roi : 브라우저에서 선택한 ROI 좌표 저장
# 너무 작은 ROI는 거부합니다.
@app.post("/api/roi")
def set_roi(body: ROIRequest):
    if body.x2 - body.x1 < 0.03 or body.y2 - body.y1 < 0.03:
        raise HTTPException(400, "ROI가 너무 작습니다. 화면에서 더 넓게 드래그하세요.")
    state.set_roi((body.x1, body.y1, body.x2, body.y2))
    return {"ok": True, "roi": body.model_dump()}


# ⑰ DELETE /api/roi : ROI 선택 초기화
@app.delete("/api/roi")
def reset_roi():
    state.set_roi(None)
    return {"ok": True}


# ⑱ POST /api/reload-rule : 정상·불량 기준값 다시 계산
@app.post("/api/reload-rule")
def reload_rule():
    result = state.reload_rule()
    if not result["ok"]:
        raise HTTPException(400, result["error"])
    return result


# ⑲ GET /api/status : 최신 상태를 JSON으로 전달
@app.get("/api/status")
def get_status():
    return state.get_status()


# ⑳ GET /stream.mjpg : HTTP MJPEG 실시간 영상 전송
# 새 JPEG가 준비될 때마다 브라우저에 연속 응답합니다.
@app.get("/stream.mjpg")
def stream_video():
    def frames():
        while True:
            with state.lock:
                image_bytes = state.latest_jpg
            yield (b"--frame\r\n"
                   b"Content-Type: image/jpeg\r\n"
                   b"Cache-Control: no-cache\r\n\r\n"
                   + image_bytes + b"\r\n")
            time.sleep(0.10)  # 브라우저에 최대 약 10회/초 전송

    return StreamingResponse(
        frames(), media_type="multipart/x-mixed-replace; boundary=frame",
        headers={"Cache-Control": "no-store", "X-Content-Type-Options": "nosniff"},
    )
```

### 5.3 의사코드

```
[① 라이브러리와 프로젝트 경로 준비]
FastAPI, OpenCV, Pydantic 불러오기
기존 cap_inspector.py 검사 함수 가져오기
기준 이미지와 HTML 파일 경로 설정
        ↓
[② 카메라 연결 전 대기 이미지 생성]
IP WEBCAM - WAITING 화면 준비
        ↓
[③ 실시간 검사 상태 관리]
카메라 URL, ROI, 기준값, 판정 결과
최신 영상 Frame 저장 공간 준비
        ↓
[④ 병뚜껑 검사 기준값 불러오기]
cap_on.jpg / cap_off.jpg 읽기
Area Ratio 판정 기준 생성
        ↓
[⑤ 카메라 작업 시작 및 종료]
별도 Thread로 카메라 수신 시작
서버 종료 시 Thread 정리
        ↓
[⑥ 카메라 주소와 ROI 설정]
스마트폰 IP Webcam 주소 저장
웹에서 선택한 ROI 좌표 저장
        ↓
[⑦ 현재 검사 상태 조회]
카메라 연결 여부
OK / NG / CHECK
Area Ratio와 기준값 반환
        ↓
[⑧ 스마트폰 카메라 연결]
cv2.VideoCapture(URL)
HTTP IP Webcam 영상 연결
연결 실패 시 재시도
        ↓
[⑨ 실시간 Frame 읽기]
cap.read()로 영상 Frame 수신
수신 실패 시 연결 상태 변경
        ↓
[⑩ ROI Crop 및 OpenCV 검사]
웹에서 지정한 ROI 좌표 확인
전체 Frame에서 ROI Crop
        ↓
cap_inspector.py의 inspect_roi() 실행
        ↓
OK / NG / CHECK 판정
ROI 사각형 표시
        ↓
[⑪ 검사 결과 영상 생성 및 저장]
판정값과 Area Ratio 화면에 표시
JPEG 이미지로 변환
최신 영상 및 검사 결과 저장
        ↓
[⑫ FastAPI 애플리케이션 생성]
FastAPI 객체 생성
서버 시작·종료 시 실행할 작업 등록
        ↓
[⑬ 요청 데이터 검증 모델 작성]
Pydantic BaseModel 사용

CameraRequest → URL 문자열
ROIRequest → x1, y1, x2, y2 좌표
좌표값은 0~1 범위인지 검증
        ↓
[⑭ GET /]
브라우저 요청
        ↓
HTML 검사 화면 반환
        ↓
[⑮ POST /api/camera]
스마트폰 IP Webcam URL 전달
        ↓
URL 형식·IP·포트 확인
        ↓
카메라 연결 시도
        ↓
[⑯ POST /api/roi]
브라우저에서 선택한 ROI 좌표 수신
        ↓
좌표 유효성 및 최소 크기 확인
        ↓
ROI 저장
        ↓
[⑰ DELETE /api/roi]
선택한 ROI 초기화
        ↓
[⑱ POST /api/reload-rule]
cap_on / cap_off 기준 이미지 다시 읽기
        ↓
판정 기준값 다시 계산
        ↓
[⑲ GET /api/status]
현재 카메라 및 검사 상태 조회
        ↓
JSON 형태로 브라우저에 반환
        ↓
[⑳ GET /stream.mjpg]
최신 JPEG 영상을 반복해서 가져오기
        ↓
HTTP MJPEG 스트리밍
        ↓
웹브라우저에 실시간 영상 표시
```

### 5.4 꼭 알아야 할 핵심 구조

|구분|순번|핵심 역할|
|---|---|---|
|서버 내부 처리|①~⑫|카메라 영상을 받아 OpenCV로 검사하고 결과를 저장|
|데이터 검증|⑬ 및 ⑮~⑯|브라우저에서 보낸 URL과 ROI 좌표가 올바른지 확인|
|API 엔드포인트|⑭~⑳|HTTP 요청에 따라 화면, 검사 결과, 영상을 전달|

특히 ⑩번에서 사용하는 다음 코드가 이전 수업과 연결되는 가장 중요한 부분입니다.

```python
result = inspect_roi(region, rule)
```

이 코드는 앞서 만든 `cap_inspector.py`의 핵심 검사 함수를 호출하여 현재 영상의 병뚜껑을 판정하는 부분입니다.

이번 수업에서 가장 중요하게 이해해야 할 내용은 다음과 같습니다.

`cap_inspector.py`는 무엇을 검사하고 어떻게 판정할지 담당하고, `fastapi_cap_app.py`는 
언제 검사할지, 입력을 어떻게 받고 결과를 웹브라우저에 어떻게 전달할지 담당합니다.

---

## PART 6. 코드 작성 ③ — HTML 화면에서 직접 ROI를 고릅니다

### 6.1 이번 코드는 무엇을 하나요?

앞에서는 `cap_inspector.py`에 병뚜껑을 검사하는 핵심 로직을 만들고, `fastapi_cap_app.py`에서 스마트폰 영상과 검사 결과를 전달하는 FastAPI 서버를 만들었습니다.

이번에는 **사용자가 웹브라우저에서 카메라를 연결하고, ROI를 선택하고, 검사 결과를 확인할 수 있는 화면**을 만들겠습니다.

이때 HTML, CSS, JavaScript는 각각 다른 역할을 합니다.

- **HTML:** 카메라 연결 버튼, 영상 화면, 검사 결과 표시 영역 등 화면의 기본 구조를 만듭니다.
    
- **CSS:** 버튼, 글자, 배경색, OK/NG 결과의 색상 등 화면 디자인을 설정합니다.
    
- **JavaScript:** 버튼 클릭이나 마우스 드래그를 감지하고, 필요한 경우 FastAPI 서버에 HTTP 요청을 보냅니다.
    

특히 이번 실습에서 중요한 것은 JavaScript의 `이벤트(Event)`와 `fetch()`입니다.

이벤트는 사용자가 버튼을 클릭하거나 마우스를 움직이는 등의 동작을 의미합니다.

예를 들어 사용자가 **카메라 연결 버튼을 클릭하면**, JavaScript가 클릭 이벤트를 감지하고 `fetch()`를 이용하여 FastAPI의 `/api/camera` 엔드포인트에 HTTP POST 요청을 보냅니다.

FastAPI는 요청받은 카메라 주소를 확인하고 영상 연결을 시도합니다.

이처럼 **JavaScript는 사용자의 동작과 서버 기능을 연결하고,** `fetch()`**는 서버에 HTTP 요청을 보내고 응답을 받는 기능**을 담당합니다.

실시간 영상은 조금 다릅니다. HTML의 `<img>` 요소가 FastAPI의 `/stream.mjpg` 주소에 연결하여 MJPEG 영상을 표시합니다. 검사 결과는 JavaScript가 `/api/status`를 주기적으로 조회하여 갱신합니다.

따라서 별도의 React 프로젝트나 WebSocket 없이 HTML, CSS, JavaScript만으로 실시간 검사 화면을 만들 수 있습니다.

### 6.2 코드의 전체 처리 흐름 — 의사코드

```
[① HTML 화면 만들기]
카메라 주소 입력창
카메라 연결 버튼
실시간 영상 표시 영역
ROI 선택 영역
검사 결과 표시 영역
        ↓
[② CSS로 화면 디자인 설정]
버튼 색상과 크기
영상 및 검사 결과 배치
OK / NG / 확인 필요 색상
        ↓
[③ JavaScript로 화면 요소 연결]
입력창, 버튼, 영상, ROI 영역 가져오기
        ↓
[④ 카메라 연결 버튼 클릭 이벤트]
사용자가 카메라 주소 입력
        ↓
JavaScript fetch()
        ↓
POST /api/camera
        ↓
FastAPI가 카메라 연결 시도
        ↓
[⑤ 실시간 영상 표시]
HTML img 요소
        ↓
GET /stream.mjpg
        ↓
FastAPI의 MJPEG 영상 표시
        ↓
[⑥ 마우스로 ROI 선택]
드래그 시작과 종료 감지
        ↓
선택 영역의 좌표 생성
        ↓
JavaScript fetch()
        ↓
POST /api/roi
        ↓
FastAPI가 ROI 저장 및 검사 적용
        ↓
[⑦ 검사 결과 조회]
JavaScript가 0.4초마다
GET /api/status 요청
        ↓
FastAPI가 JSON으로 검사 결과 반환
        ↓
OK / NG / 확인 필요 표시
Area Ratio와 기준값 표시
        ↓
[⑧ 추가 버튼 기능]
ROI 다시 선택
    → DELETE /api/roi

기준 이미지 다시 읽기
    → POST /api/reload-rule
```

### 6.3 실제로 어떤 부분을 수정하게 될까요?

이번 실습에서 가장 자주 수정하거나 확인하게 될 부분은 다음과 같습니다.

|수정하려는 기능|수정할 코드|
|---|---|
|화면 제목, 버튼 이름, 안내 문구|HTML|
|버튼 색상, 글자 크기, 화면 배치|CSS|
|카메라 연결 버튼의 동작|JavaScript의 `connect.onclick`|
|ROI 선택 및 드래그 동작|JavaScript의 `pointerdown`, `pointermove`, `pointerup`|
|서버에 전달할 주소와 데이터|JavaScript의 `fetch()` 및 `jsonPost()`|
|검사 결과 갱신 주기|`setInterval(refreshStatus, 400)`|
|OK / NG / 확인 필요 표시 문구와 색상|JavaScript의 `refreshStatus()` 및 CSS|
|실제 병뚜껑 판정 기준|Python의 `cap_inspector.py`|
|API 요청 처리 및 데이터 검증|Python의 `fastapi_cap_app.py`|

특히 JavaScript에서 다음 코드는 꼭 이해해야 합니다.

```js
await jsonPost('/api/camera', {url});
```

이 코드는 FastAPI의 `/api/camera` 엔드포인트에 카메라 주소를 JSON으로 전달하는 것입니다. `jsonPost()` 함수 내부에서 `fetch()`를 사용합니다.

다른 예시도 살펴보겠습니다.

```js
await jsonPost('/api/roi', rect);
```

이 코드는 사용자가 마우스로 선택한 ROI 좌표를 FastAPI 서버에 전달합니다.

```js
const data = await requestJson('/api/status');
```

이 코드는 FastAPI 서버에서 현재 검사 결과를 가져옵니다.

**중요:** JavaScript에서 사용하는 `/api/camera`, `/api/roi`, `/api/status` 등의 주소는 앞에서 만든 FastAPI의 엔드포인트 주소와 일치해야 합니다. 서버의 엔드포인트 주소를 변경했다면 JavaScript에서도 동일하게 변경해야 합니다.

### 핵심 정리

이번 코드는 병뚜껑을 판정하는 새로운 검사 코드를 작성하는 과정이 아닙니다.

**기존에 만든 OpenCV 검사 기능과 FastAPI 서버를 사용자가 웹브라우저에서 편리하게 이용할 수 있도록 연결하는 과정**입니다.

HTML은 화면, CSS는 디자인, JavaScript는 이벤트 및 HTTP 요청, FastAPI는 서버 요청 처리, OpenCV는 실제 검사를 담당합니다.

### 완성 코드 — `web/cap_inspection.html`

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>교과 9 - 실시간 병뚜껑 검사</title>
  <style>
    :root { font-family: "Malgun Gothic", "Noto Sans KR", system-ui, sans-serif; color:#17243a; }
    * { box-sizing:border-box; }
    body { margin:0; background:#edf2f9; }
    main { max-width:1180px; margin:20px auto; padding:0 18px 32px; }
    header { background:#123574; color:white; padding:22px 26px; border-radius:14px; }
    h1 { margin:0; font-size:clamp(21px,3vw,30px); }
    header p { margin:9px 0 0; color:#dbeaff; }
    .panel { margin-top:15px; background:white; padding:17px; border-radius:12px; box-shadow:0 3px 16px #24436610; }
    label { display:block; font-weight:750; margin-bottom:8px; }
    .controls { display:flex; flex-wrap:wrap; gap:9px; }
    input { min-width:220px; flex:1; font-size:15px; padding:11px 12px; border:1px solid #bdcde3; border-radius:8px; }
    button { border:none; cursor:pointer; border-radius:8px; padding:11px 15px; font-size:14px; font-weight:700; background:#1957bf; color:white; }
    button.secondary { background:#55657a; }
    button:focus-visible, input:focus-visible { outline:3px solid #f3b95b; outline-offset:2px; }
    .columns { display:grid; grid-template-columns:minmax(0,2fr) minmax(250px,1fr); gap:15px; align-items:start; }
    .stage { position:relative; background:#263343; border-radius:9px; overflow:hidden; }
    #video { display:block; width:100%; height:auto; min-height:240px; }
    #selection { display:block; position:absolute; inset:0; width:100%; height:100%; cursor:crosshair; touch-action:none; }
    .hint { margin:8px 0 0; color:#52627a; font-size:14px; line-height:1.6; }
    .status-box { text-align:center; padding:24px 9px; background:#eef3fa; border-radius:10px; border:1px solid #d6e2f2; }
    #decision { font-size:clamp(36px,4vw,58px); font-weight:900; color:#53687f; letter-spacing:1px; }
    #decision.ok { color:#0e893c; }
    #decision.ng { color:#d72b2b; }
    #decision.check { color:#b27400; }
    #meaning { margin-top:7px; font-weight:700; }
    dl { display:grid; grid-template-columns:1fr auto; row-gap:14px; font-size:14px; }
    dt { color:#58677a; } dd { margin:0; font-weight:800; }
    #notice { margin-top:10px; min-height:24px; color:#aa2430; white-space:pre-wrap; font-size:14px; }
    .note { background:#fff9e7; padding:12px; border-left:4px solid #e7b73f; line-height:1.65; color:#5a4b28; font-size:14px; }
    @media(max-width:760px){ .columns{grid-template-columns:1fr;} main{margin-top:8px} }
  </style>
</head>
<body>
<main>
  <header>
    <h1>교과 9 | IP Webcam 실시간 병뚜껑 검사</h1>
    <p>Day01 ROI + Day02 Binary + Day03 Contour · Rule + FastAPI</p>
  </header>
  <section class="panel">
    <label for="camera-url">① 스마트폰 IP Webcam 영상 주소를 입력합니다.</label>
    <div class="controls">
      <input id="camera-url" type="url" placeholder="예: http://192.168.0.25:8080/video" autocomplete="off">
      <button id="connect" type="button">카메라 연결</button>
      <button id="reload-rule" class="secondary" type="button">기준 이미지 다시 읽기</button>
    </div>
    <div id="notice" role="alert"></div>
  </section>
  <div class="columns">
    <section class="panel">
      <label>② 영상에서 병뚜껑이 있어야 하는 영역을 마우스로 드래그합니다.</label>
      <div class="stage">
        <img id="video" src="/stream.mjpg" alt="스마트폰 카메라 실시간 영상">
        <canvas id="selection" aria-label="ROI 선택 화면"></canvas>
      </div>
      <div class="controls" style="margin-top:11px;">
        <button id="reset-roi" class="secondary" type="button">ROI 다시 선택</button>
      </div>
      <p class="hint">뚜껑이 있는 위치를 선택하면 같은 위치를 계속 검사합니다. 정상/불량 모두 같은 ROI를 사용하세요.</p>
    </section>
    <aside class="panel">
      <label>③ 실시간 검사 결과</label>
      <div class="status-box" aria-live="polite">
        <div id="decision">대기</div>
        <div id="meaning">카메라와 ROI를 준비하세요.</div>
      </div>
      <dl>
        <dt>카메라 상태</dt><dd id="camera-state">NOT_CONNECTED</dd>
        <dt>검사 Area Ratio</dt><dd id="ratio">-</dd>
        <dt>Rule 기준값</dt><dd id="threshold">-</dd>
        <dt>정상 기준 Ratio</dt><dd id="on-ratio">-</dd>
        <dt>불량 기준 Ratio</dt><dd id="off-ratio">-</dd>
        <dt>ROI 설정</dt><dd id="roi-state">미설정</dd>
      </dl>
      <div class="note">학습용 검사입니다. 병이 ROI에서 벗어나면 오판정이 발생할 수 있습니다. 뚜껑 없는 상태를 확인하기 전에 반드시 병 입구가 ROI에 있는지 확인하세요.</div>
    </aside>
  </div>
</main>
<script>
  const $ = id => document.getElementById(id);
  const img = $('video');
  const canvas = $('selection');
  const ctx = canvas.getContext('2d');
  let dragging = false, start = null, selected = null;

  function sizeCanvas() {
    const width = Math.round(img.getBoundingClientRect().width);
    const height = Math.round(img.getBoundingClientRect().height);
    if (width && height && (canvas.width !== width || canvas.height !== height)) {
      canvas.width = width;
      canvas.height = height;
      drawSelection();
    }
  }
  function drawSelection(current = null) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    const r = current || selected;
    if (!r) return;
    ctx.fillStyle = 'rgba(255, 200, 45, 0.13)';
    ctx.strokeStyle = '#facc40';
    ctx.lineWidth = 3;
    ctx.setLineDash([7, 4]);
    ctx.fillRect(r.x1 * canvas.width, r.y1 * canvas.height,
                 (r.x2 - r.x1) * canvas.width, (r.y2 - r.y1) * canvas.height);
    ctx.strokeRect(r.x1 * canvas.width, r.y1 * canvas.height,
                   (r.x2 - r.x1) * canvas.width, (r.y2 - r.y1) * canvas.height);
    ctx.setLineDash([]);
  }
  function pointFromEvent(e) {
    const box = canvas.getBoundingClientRect();
    return {
      x: Math.max(0, Math.min(1, (e.clientX - box.left) / box.width)),
      y: Math.max(0, Math.min(1, (e.clientY - box.top) / box.height))
    };
  }
  function rectangle(a, b) {
    return {x1:Math.min(a.x,b.x), y1:Math.min(a.y,b.y),
            x2:Math.max(a.x,b.x), y2:Math.max(a.y,b.y)};
  }
  async function requestJson(path, options) {
    const response = await fetch(path, options);
    const result = await response.json();
    if (!response.ok) throw new Error(typeof result.detail === 'string' ? result.detail : '서버 요청 실패');
    return result;
  }
  const jsonPost = (path, data) => requestJson(path, {
    method:'POST', headers:{'Content-Type':'application/json'}, body:JSON.stringify(data)
  });

  canvas.addEventListener('pointerdown', e => {
    e.preventDefault(); sizeCanvas();
    dragging = true; start = pointFromEvent(e);
    canvas.setPointerCapture(e.pointerId);
  });
  canvas.addEventListener('pointermove', e => {
    if (!dragging) return;
    drawSelection(rectangle(start, pointFromEvent(e)));
  });
  canvas.addEventListener('pointerup', async e => {
    if (!dragging) return;
    dragging = false;
    const rect = rectangle(start, pointFromEvent(e));
    if (rect.x2 - rect.x1 < 0.03 || rect.y2 - rect.y1 < 0.03) {
      $('notice').textContent = 'ROI가 너무 작습니다. 병뚜껑 영역을 넓게 드래그하세요.';
      drawSelection(); return;
    }
    try {
      await jsonPost('/api/roi', rect);
      selected = rect;
      drawSelection();
      $('notice').textContent = '';
    } catch (err) { $('notice').textContent = err.message; drawSelection(); }
  });
  canvas.addEventListener('pointercancel', () => {dragging = false; drawSelection();});

  $('connect').onclick = async () => {
    const url = $('camera-url').value.trim();
    try {
      await jsonPost('/api/camera', {url});
      selected = null; drawSelection();
      $('notice').textContent = '카메라 연결을 시도합니다. 영상이 나오면 ROI를 선택하세요.';
    } catch(err) { $('notice').textContent = err.message; }
  };
  $('reload-rule').onclick = async () => {
    try {
      const r = await requestJson('/api/reload-rule', {method:'POST'});
      $('notice').textContent = 'Rule을 다시 읽었습니다. 기준값: ' + r.threshold.toFixed(3);
    } catch(err) { $('notice').textContent = err.message; }
  };
  $('reset-roi').onclick = async () => {
    try {
      await requestJson('/api/roi', {method:'DELETE'});
      selected = null; drawSelection();
      $('notice').textContent = 'ROI를 초기화했습니다. 다시 드래그하세요.';
    } catch(err) { $('notice').textContent = err.message; }
  };

  async function refreshStatus() {
    try {
      const data = await requestJson('/api/status');
      $('camera-state').textContent = data.camera_status;
      $('ratio').textContent = data.ratio === null ? '-' : data.ratio.toFixed(3);
      $('threshold').textContent = data.threshold === null ? '-' : data.threshold.toFixed(3);
      $('on-ratio').textContent = data.on_ratio === null ? '-' : data.on_ratio.toFixed(3);
      $('off-ratio').textContent = data.off_ratio === null ? '-' : data.off_ratio.toFixed(3);
      $('roi-state').textContent = data.roi ? '설정됨' : '미설정';
      if (data.roi && !dragging) {
        selected = {x1:data.roi[0],y1:data.roi[1],x2:data.roi[2],y2:data.roi[3]};
        drawSelection();
      }
      const status = $('decision');
      status.className = '';
      if (data.decision === 'OK' && data.rule_ready) {
        status.textContent = 'OK'; status.className = 'ok';
        $('meaning').textContent = '병뚜껑 있음 (규칙 판정)';
      } else if (data.decision === 'NG' && data.rule_ready) {
        status.textContent = 'NG'; status.className = 'ng';
        $('meaning').textContent = '병뚜껑 없음 (규칙 판정)';
      } else if (data.decision === 'CHECK') {
        status.textContent = '확인 필요'; status.className = 'check';
        $('meaning').textContent = 'ROI 또는 윤곽선 검출을 확인하세요.';
      } else {
        status.textContent = '대기';
        $('meaning').textContent = !data.rule_ready ? '기준 이미지 확인 필요'
          : data.camera_status !== 'ONLINE' ? '카메라 연결 확인 필요'
          : 'ROI를 선택하세요.';
      }
      if (!data.rule_ready) $('notice').textContent = data.rule_error;
    } catch(err) {
      $('notice').textContent = '상태 확인 실패: ' + err.message;
    }
  }
  img.addEventListener('load', sizeCanvas);
  new ResizeObserver(sizeCanvas).observe(img);
  setInterval(refreshStatus, 400);
  refreshStatus(); sizeCanvas();
</script>
</body>
</html>
```

이번 실습에서는 HTML·CSS·JavaScript의 역할과 연결 구조를 쉽게 이해할 수 있도록 하나의 파일에 작성하며, 이후 프로젝트 규모가 커지면 유지보수와 코드 재사용을 위해 파일을 분리할 수 있습니다.

---

## PART 7. 실제 실행하기 — 순서대로 따라하세요

### 7.1 스마트폰에서 IP Webcam을 실행합니다

1. 스마트폰과 PC를 가능한 한 같은 Wi-Fi에 연결합니다.
2. 스마트폰의 IP Webcam 앱에서 영상 서버를 시작합니다.
3. 스마트폰 화면에 나타나는 IP 주소 및 포트를 확인합니다.
4. PC 브라우저에서 실제 스트림 URL을 먼저 열어 영상이 보이는지 확인합니다.

예시는 다음과 같습니다(실제 자신의 스마트폰 주소로 변경).

```text
http://192.168.0.25:8080/video
```

일부 앱에서는 `/videofeed` 등 다른 경로를 씁니다. **스마트폰 서버의 첫 화면 주소(예: `http://...:8080/`)와 실제 비디오 스트림 주소(`/video`)는 다를 수 있습니다.** 이번 코드는 MJPEG/영상 스트림 주소를 입력받습니다.

### 7.2 기존 OpenCV 카메라 프로그램을 종료합니다

1일차 `08_phone_camera_roi.py`가 아직 동작 중이라면 그 프로그램에서 `q`를 눌러 종료합니다. 여러 프로그램이 같은 카메라 스트림에 동시에 접속하면 앱 또는 네트워크 환경에 따라 문제가 생길 수 있기 때문입니다.

### 7.3 FastAPI 서버 실행

프로젝트 최상위 폴더와 가상환경을 확인한 뒤 실행합니다.

```bash
cd ~/subject9_machinevision
source .venv/bin/activate
python -m uvicorn src.fastapi_cap_app:app --host 127.0.0.1 --port 8000
```

프로젝트 위치가 다르다면 **첫 줄만 실제 경로로 변경**합니다. 이번 실습에서는 외부에 공개하지 않도록 `127.0.0.1`로 실행합니다.

성공 시 터미널에 다음과 비슷하게 표시됩니다.

```text
INFO:     Started server process [...]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000
```

이 터미널은 서버 실행 중이므로 닫지 않습니다. 종료는 `Ctrl + C`입니다. **서버 실행 때 스마트폰이 아직 켜져 있지 않아도 웹 페이지 자체는 열 수 있습니다.**

### 7.4 Windows 브라우저에서 접속합니다

PC의 Chrome·Edge 주소창에 다음을 입력합니다.

```text
http://127.0.0.1:8000
```

WSL2의 localhost 전달 설정에 따라 Windows 브라우저에서 접속할 수 있습니다. 접속되지 않으면 PART 9의 해결 방법을 참고하세요.

### 7.5 카메라를 연결합니다

1. 첫 번째 입력란에 실제 스마트폰 영상 URL을 입력합니다.
2. **카메라 연결** 버튼을 누릅니다.
3. 영상이 나타나는지, `카메라 상태`가 `ONLINE`인지 확인합니다.
4. 영상이 보이더라도 아직 ROI가 없다면 **대기**가 정상입니다.

### 7.6 실제 병뚜껑에서 ROI를 선택합니다

1. 병뚜껑이 닫힌 병을 카메라 앞에 놓습니다.
2. 병뚜껑이 있어야 하는 부분을 마우스로 드래그합니다.
3. 사각형이 표시되면 ROI가 설정됩니다.
4. 기준 이미지에서 저장했던 **뚜껑 위치와 유사한 구도**인지 확인합니다.
5. 처음에는 병과 스마트폰을 고정하고 진행합니다.

여기서 검출은 **마우스로 선택한 고정 ROI에서만** 일어납니다. 병이 크게 움직이면 자동으로 따라가는 기능은 구현되어 있지 않습니다.

### 7.7 뚜껑 있음 / 없음 실시간 실험

**실험 A — 병뚜껑 있음**

```text
병뚜껑을 닫는다
      ↓
ROI 안에 병뚜껑이 들어온다
      ↓
Area Ratio 측정
      ↓
Rule과 비교
      ↓
화면에 OK가 표시되는지 확인
```

**실험 B — 병뚜껑 없음**

```text
병과 카메라 위치를 그대로 유지한다
      ↓
병뚜껑만 제거한다
      ↓
ROI에는 여전히 병 입구가 보여야 한다
      ↓
Area Ratio 다시 측정
      ↓
Rule과 비교
      ↓
화면에 NG가 표시되는지 확인
```

**결과가 곧바로 바뀌지 않는다면** 카메라 네트워크 지연과 `/api/status` 조회 간격(0.4초)을 고려하여 확인합니다. 항상 즉시 정답이 나온다고 보장하지는 않습니다.

---

## PART 8. 판정 결과가 맞는지 검증합니다

### 8.1 세 가지 상태를 구분합니다

| 표시 | 의미 | 학생이 할 일 |
|---|---|---|
| **OK** | ROI의 특징값이 Rule 기준상 '뚜껑 있음' 쪽 | 실제 병뚜껑이 있는지 육안 비교 |
| **NG** | ROI의 특징값이 Rule 기준상 '뚜껑 없음' 쪽 | 실제로 뚜껑이 없는지 육안 비교 |
| **확인 필요** | 유효 Contour를 찾지 못했거나 ROI 처리에 실패 | ROI·조명·병 위치를 확인 |
| **대기 / 카메라 끊김** | 카메라 미연결, ROI 미설정, 기준 파일 오류 등 | 표시된 문제를 해결 후 다시 검사 |

**매우 중요:** 이 예제에는 **병 자체가 화면에 있는지 자동 검출하는 별도 알고리즘이 없습니다.** 따라서 병이 ROI 밖으로 이동했는데도 배경에서 다른 Contour가 잡히면 OK/NG가 잘못 표시될 수 있습니다. 카메라와 병 위치를 고정한 **교육용 검사 조건에서만** 결과를 해석하세요. `확인 필요`는 모든 ROI 이탈을 자동 검출하는 안전장치가 아닙니다.

### 8.2 Rule의 성능이라고 주장하면 안 되는 경우

기준 이미지 2장만으로 구한 Rule로 **같은 기준 2장을 다시 검사한 결과**는 모델 정확도라고 할 수 없습니다. 추가로 촬영한 새 이미지 또는 실시간 검사를 통해 정답과 비교해야 합니다. 3일차 Batch Test가 이 원리를 설명합니다.

### 8.3 최소 검증표

| 테스트 | 실제 상태 | 화면의 판정 | 정답 여부 | 메모 |
|---|---|---|---|---|
| 1 | 뚜껑 있음 | | | |
| 2 | 뚜껑 없음 | | | |
| 3 | 뚜껑 있음(위치 조금 변화) | | | |
| 4 | 뚜껑 없음(조명 조금 변화) | | | |
| 5 | 병을 ROI 밖으로 이동 | | | | 

5번은 **무조건 NG가 정답인 시험이 아닙니다.** 이 경우 검사 위치 자체가 신뢰하기 어려워, 실제 현업에서는 위치 정렬/검출 또는 유효성 검사를 추가해야 합니다.

### 8.4 같은 Rule을 유지한 채 비교합니다

결과가 틀리면 바로 기준값을 바꾸지 말고, 먼저 다음을 확인합니다.

```text
1. 스마트폰 해상도/거리/조명이 기준 촬영과 비슷한가?
2. ROI가 병뚜껑과 병 입구를 정상/불량 모두 같은 범위로 포함하는가?
3. Otsu의 Binary 결과에서 병뚜껑이 흰색 객체로 잘 분리되는가?
4. 불필요한 반사·그림자·손가락이 ROI에 포함되지는 않았는가?
5. 그다음 실제 Area Ratio를 비교해 기준을 조정할 것인가?
```

---

## PART 9. 오류 및 문제 해결

| 증상 | 원인 후보 | 확인 및 해결 |
|---|---|---|
| `ModuleNotFoundError: fastapi` | 가상환경에 FastAPI 미설치 | `source .venv/bin/activate` 후 `python -m pip install fastapi uvicorn` |
| `Could not import module 'src.fastapi_cap_app'` | 실행 위치 또는 파일명 오류 | 프로젝트 루트의 `src/fastapi_cap_app.py` 확인, 루트에서 실행 |
| `기준 이미지 읽기 전` 또는 기준 오류 | `samples/day03/reference/`에 두 이미지 없음 | 파일을 복사한 후 **기준 이미지 다시 읽기** 클릭 |
| 카메라 `DISCONNECTED` | IP 주소 변경, 스마트폰 앱 서버 미실행, Wi-Fi 분리 | 스마트폰 화면의 현재 주소를 확인, PC에서 `/video` 먼저 테스트 |
| 카메라 상태는 ONLINE인데 판정 대기 | ROI 미설정 또는 기준 규칙 오류 | 영상에서 마우스 드래그, 기준 이미지 파일 확인 |
| 영상이 느리게 보임 | 스마트폰/네트워크 지연 또는 JPEG 처리 부하 | 스마트폰 앱 해상도를 640×480 수준으로 낮춰 확인 |
| Windows 브라우저에서 `127.0.0.1:8000` 접속 실패 | WSL2 localhost 전달 설정 또는 서버 종료 | WSL에서 `curl http://127.0.0.1:8000/api/status` 확인, 터미널의 Uvicorn 로그 확인 |
| ROI 선택 후 값이 이상함 | 병 위치·ROI 좌표·조명 차이 | ROI 초기화 후 정상/불량 모두 동일한 병 입구 위치로 지정 |
| 병이 없는데 OK 또는 NG가 뜸 | **병 존재 확인 기능이 별도로 없음** | 검사 대상 위치를 고정, 필요하면 병 위치 검출/정렬 단계 추가 |
| `Address already in use` | 기존 서버가 포트 8000 사용 | 기존 Uvicorn 종료 또는 `--port 8001` 사용 |

WSL에서 API 연결 여부만 확인할 때:

```bash
curl http://127.0.0.1:8000/api/status
```

정상이라면 JSON 형식의 카메라 상태와 기준값이 출력됩니다. `camera_status`가 아직 `NOT_CONNECTED`여도 FastAPI 서버 연결 자체는 성공한 것입니다.

**보안:** 이 실습은 개인 PC의 로컬 웹서버만 엽니다. 웹 인터페이스에 인증 기능이 없으므로 공개 인터넷에 서버를 노출하거나 포트포워딩하지 마세요. 카메라 URL도 공개 저장소에 올리지 않습니다.

---

## PART 10. 자동 검증 — 실제 카메라가 없을 때

### 이번 테스트는 무엇을 하나요?

지금까지 작성한 코드가 정상적으로 작동하는지 확인하기 위해 **자동 코드 테스트(Automated Test)**를 진행하겠습니다.

코드 테스트란 프로그램을 실행했을 때 **예상한 결과가 나오는지, 오류 없이 기능이 동작하는지 확인하는 과정**입니다.

이번 실습에서는 실제 스마트폰 카메라나 병뚜껑 사진이 없어도 테스트할 수 있도록 가상의 이미지와 카메라를 사용합니다.

**이번 테스트에서 확인할 내용은 다음과 같습니다.**

1. **검사 로직:** 준비된 테스트 이미지에서 Area Ratio를 계산하고 OK / NG를 판정하는지 확인합니다.
    
2. **데이터 검증:** 잘못된 카메라 주소나 ROI 좌표를 입력했을 때 적절하게 오류를 처리하는지 확인합니다.
    
3. **API 연결:** FastAPI 엔드포인트에 HTTP 요청을 보내면 정상적으로 응답하는지 확인합니다.
    
4. **전체 연결:** 가짜 카메라에서 전달한 영상이 OpenCV 검사를 거쳐 서버의 검사 결과로 연결되는지 확인합니다.
    

자동 테스트에는 `pytest`를 사용하고, FastAPI의 HTTP 요청과 응답을 테스트하기 위해 `httpx`를 사용합니다.

**주의:** 테스트가 통과했다는 것은 검사 코드와 서버의 기능이 예상대로 연결되어 작동한다는 의미입니다. 실제 병뚜껑 검사 정확도가 높다는 뜻은 아니므로, 이후 실제 카메라와 병뚜껑을 이용한 검증이 별도로 필요합니다.

### 테스트 라이브러리 설치

```bash
python -m pip install pytest httpx
```

### 테스트 파일 생성

`tests/test_inspector.py` 파일을 새로 만듭니다.

### 완성 코드 — `tests/test_inspector.py`

```python

# ============================================================
# ① 테스트에 필요한 라이브러리 불러오기
# - OpenCV, NumPy : 가상 이미지 생성 및 검사
# - TestClient    : 실제 브라우저 없이 FastAPI 요청 테스트
# - cap_inspector : 앞에서 만든 병뚜껑 검사 함수
# - app, state    : FastAPI 애플리케이션과 검사 상태
# ============================================================

from pathlib import Path
import time

import cv2
import numpy as np
from fastapi.testclient import TestClient

from src.cap_inspector import (
    inspect_roi,
    judge_cap,
    load_rule,
    measure_area_ratio
)
from src.fastapi_cap_app import app, state


# ============================================================
# ② 테스트용 가상 병뚜껑 이미지 생성
# - 실제 사진 대신 사각형 패턴 이미지를 만듭니다.
# - cap_on=True  : 큰 검은색 사각형
# - cap_on=False : 작은 검은색 사각형
# - 실제 병뚜껑 판정 정확도를 평가하는 이미지는 아닙니다.
# ============================================================

def synthetic_roi(cap_on: bool) -> np.ndarray:
    """실제 사진 없이 검사 규칙을 확인하는 인공 패턴."""

    image = np.full((160, 200, 3), 235, np.uint8)

    if cap_on:
        cv2.rectangle(
            image, (35, 25), (165, 105), (15, 15, 15), -1
        )
    else:
        cv2.rectangle(
            image, (65, 90), (135, 120), (15, 15, 15), -1
        )

    return image


# ============================================================
# ③ 검사 기준값 생성 테스트 준비
# - 가상 cap_on / cap_off 이미지를 JPG로 저장합니다.
# - load_rule()을 이용하여 판정 기준값을 만듭니다.
# - tmp_path는 pytest가 제공하는 임시 폴더입니다.
# ============================================================

def test_rule(tmp_path: Path):

    cv2.imwrite(
        str(tmp_path / 'cap_on.jpg'),
        synthetic_roi(True)
    )

    cv2.imwrite(
        str(tmp_path / 'cap_off.jpg'),
        synthetic_roi(False)
    )

    rule = load_rule(tmp_path)


    # ========================================================
    # ④ OpenCV 검사 로직 테스트
    # - cap_on은 OK, cap_off는 NG가 나와야 합니다.
    # - 두 이미지의 Area Ratio와 기준값을 비교합니다.
    # - assert는 예상 결과와 다르면 테스트를 실패시킵니다.
    # ========================================================

    on = inspect_roi(synthetic_roi(True), rule)
    off = inspect_roi(synthetic_roi(False), rule)

    assert on['decision'] == 'OK'
    assert off['decision'] == 'NG'

    assert (
        rule.cap_on_ratio
        > rule.threshold
        > rule.cap_off_ratio
    )

    assert (
        measure_area_ratio(synthetic_roi(True))[0]
        > measure_area_ratio(synthetic_roi(False))[0]
    )

    assert judge_cap(rule.threshold, rule)[0] == 'OK'


# ============================================================
# ⑤ FastAPI 통합 테스트 준비
# - 테스트용 이미지 2장으로 가상 판정 기준을 만듭니다.
# - 이후 FastAPI에서도 이 기준을 사용합니다.
# ============================================================

def test_api_and_fake_camera(tmp_path: Path):

    cv2.imwrite(
        str(tmp_path / 'cap_on.jpg'),
        synthetic_roi(True)
    )

    cv2.imwrite(
        str(tmp_path / 'cap_off.jpg'),
        synthetic_roi(False)
    )

    fake_rule = load_rule(tmp_path)


    # ========================================================
    # ⑥ 실제 카메라 대신 사용할 준비
    # - FastAPI 서버 모듈을 불러옵니다.
    # - 기존 VideoCapture 함수를 보관합니다.
    # - 테스트 시작 시 cap_on 이미지를 사용하도록 설정합니다.
    # ========================================================

    import src.fastapi_cap_app as server

    original = server.cv2.VideoCapture
    which = {'on': True}


    # ========================================================
    # ⑦ 가짜 카메라 FakeCapture 만들기
    # - 실제 스마트폰 카메라를 사용하지 않습니다.
    # - isOpened()는 연결 성공 상태를 반환합니다.
    # - read()는 가상의 영상 Frame을 반환합니다.
    # - 카메라가 계속 영상을 보내는 상황을 흉내 냅니다.
    # ========================================================

    class FakeCapture:

        def __init__(self, url):
            self.url = url

        def isOpened(self):
            return True

        def read(self):
            time.sleep(.04)
            return True, synthetic_roi(which['on'])

        def release(self):
            pass


    # ========================================================
    # ⑧ FastAPI 테스트 환경 실행
    # - 실제 VideoCapture를 FakeCapture로 임시 교체합니다.
    # - TestClient로 FastAPI를 테스트합니다.
    # - 가상의 병뚜껑 판정 기준값을 적용합니다.
    # ========================================================

    server.cv2.VideoCapture = FakeCapture

    try:
        with TestClient(app) as client:

            with state.lock:
                state.rule = fake_rule
                state.rule_error = ''


            # =================================================
            # ⑨ API 화면과 카메라 URL 검증
            # - GET / : HTML 화면 요청이 성공하는지 확인
            # - 잘못된 URL : HTTP 400 오류가 나와야 함
            # - 올바른 URL : HTTP 200 응답이 나와야 함
            # =================================================

            assert client.get('/').status_code == 200

            assert client.post(
                '/api/camera',
                json={'url': 'https://example.com/video'}
            ).status_code == 400

            assert client.post(
                '/api/camera',
                json={
                    'url': 'http://192.168.0.30:8080/video'
                }
            ).status_code == 200


            # =================================================
            # ⑩ ROI 데이터 검증
            # - 잘못된 좌표를 보내면 HTTP 400 오류 확인
            # - 정상 좌표를 보내면 HTTP 200 응답 확인
            # - 이 과정에서 Pydantic 및 API 검증을 확인
            # =================================================

            assert client.post(
                '/api/roi',
                json={
                    'x1': 0.4,
                    'y1': 0.3,
                    'x2': 0.3,
                    'y2': 0.4
                }
            ).status_code == 400

            assert client.post(
                '/api/roi',
                json={
                    'x1': 0,
                    'y1': 0,
                    'x2': 1,
                    'y2': 1
                }
            ).status_code == 200


            # =================================================
            # ⑪ 검사 결과를 기다리는 함수
            # - GET /api/status로 현재 검사 상태를 조회합니다.
            # - 원하는 판정값이 나올 때까지 반복 확인합니다.
            # - 최대 60회 확인하며, 실패하면 오류를 발생시킵니다.
            # =================================================

            def wait_for(d):

                for _ in range(60):

                    status = client.get(
                        '/api/status'
                    ).json()

                    if (
                        status['decision'] == d
                        and status['camera_status'] == 'ONLINE'
                    ):
                        return status

                    time.sleep(.05)

                raise AssertionError(
                    f'No status {d}: {status}'
                )


            # =================================================
            # ⑫ 실시간 검사 결과 테스트
            # - cap_on Frame이 들어오면 OK인지 확인합니다.
            # - cap_off Frame으로 변경하면 NG인지 확인합니다.
            # - Area Ratio도 기준값과 올바르게 비교되는지 확인
            # =================================================

            assert (
                wait_for('OK')['ratio']
                > fake_rule.threshold
            )

            # 가짜 카메라를 뚜껑 없는 상태로 변경
            which['on'] = False

            assert (
                wait_for('NG')['ratio']
                < fake_rule.threshold
            )


            # =================================================
            # ⑬ ROI 초기화 및 테스트 환경 복구
            # - DELETE /api/roi 요청으로 ROI를 초기화합니다.
            # - 테스트 종료 후 기존 카메라 함수를 복구합니다.
            # =================================================

            assert client.delete(
                '/api/roi'
            ).json()['ok']

    finally:
        server.cv2.VideoCapture = original
        state.set_camera('')
```

### 의사 코드

```
[① 테스트에 필요한 라이브러리 불러오기]
OpenCV, NumPy, pytest에서 사용할 기능 준비
기존 OpenCV 검사 함수와 FastAPI 서버 가져오기
        ↓
[② 테스트용 가상 병뚜껑 이미지 생성]
실제 사진 대신 사각형 패턴 생성

cap_on  → 큰 검은색 사각형
cap_off → 작은 검은색 사각형
        ↓
[③ 검사 기준값 생성 테스트 준비]
가상 이미지 2장을 JPG로 저장
        ↓
load_rule() 실행
        ↓
정상 / 불량 판정 기준값 생성
        ↓
[④ OpenCV 검사 로직 테스트]
cap_on 이미지 검사 → OK 예상
cap_off 이미지 검사 → NG 예상
        ↓
Area Ratio와 기준값 비교
        ↓
assert로 예상 결과와 실제 결과 확인
        ↓
[⑤ FastAPI 통합 테스트 준비]
가상 기준 이미지 2장 생성
검사 기준값 불러오기
        ↓
[⑥ 실제 카메라 대신 사용할 준비]
기존 cv2.VideoCapture 함수 임시 보관
카메라가 보여줄 이미지 상태 설정
        ↓
[⑦ 가짜 카메라 FakeCapture 만들기]
실제 스마트폰 카메라 없이
가상의 Frame을 반복해서 반환
        ↓
[⑧ FastAPI 테스트 환경 실행]
VideoCapture를 FakeCapture로 임시 교체
TestClient로 FastAPI 애플리케이션 실행
가상 검사 기준값 적용
        ↓
[⑨ API 화면과 카메라 주소 검증]
GET / → HTML 화면 정상 응답 확인
잘못된 카메라 URL → 400 오류 확인
올바른 카메라 URL → 200 응답 확인
        ↓
[⑩ ROI 데이터 검증]
잘못된 ROI 좌표 → 400 오류 확인
정상 ROI 좌표 → 200 응답 확인
        ↓
[⑪ 검사 결과를 기다리는 함수]
GET /api/status 반복 호출
원하는 판정값이 나오는지 확인
        ↓
[⑫ 실시간 검사 결과 테스트]
가상 cap_on Frame → OK 확인
        ↓
가상 cap_off Frame으로 변경
        ↓
NG로 바뀌는지 확인
        ↓
[⑬ ROI 초기화 및 테스트 환경 복구]
DELETE /api/roi 호출
        ↓
기존 VideoCapture 함수 복구
카메라 주소 초기화
        ↓
테스트 종료
```

프로젝트 루트에서 실행합니다.

```bash
python -m pytest tests/test_inspector.py -q
```

개발 환경에서 코드 테스트가 성공하면 다음과 유사한 결과가 출력됩니다.

```text
..                                                                       [100%]
2 passed
```

- 첫 번째 테스트는 3일차 Rule의 방향과 OK/NG 계산을 확인합니다.
- 두 번째 테스트는 브라우저 화면 API, ROI 좌표 검증, 가짜 카메라의 OK→NG 변화를 확인합니다.

실제 스마트폰·Wi-Fi·촬영 환경은 이 자동 테스트에 포함되지 않으므로 PART 7~8의 수동 검증을 반드시 수행해야 합니다.

### 꼭 알아야 할 테스트 코드

|코드|핵심 역할|
|---|---|
|`synthetic_roi()`|실제 사진 대신 테스트용 이미지 생성|
|`test_rule()`|OpenCV 병뚜껑 검사 로직 테스트|
|`FakeCapture`|실제 카메라 대신 가상 프레임 제공|
|`TestClient(app)`|실제 브라우저 없이 FastAPI에 HTTP 요청 테스트|
|`assert`|예상 결과와 실제 결과가 일치하는지 확인|
|`test_api_and_fake_camera()`|FastAPI와 OpenCV 검사 로직의 연결 테스트|

---

## PART 11. Mini Challenge — 무엇이 잘못 판정되었는지 분석합니다

가장 간단한 Challenge는 **ROI를 너무 좁게 고른 경우와 넓게 고른 경우의 결과를 비교**하는 것입니다.

```text
같은 병과 조명에서
   ↓
A. 병뚜껑만 아주 좁게 ROI 지정
   ↓
B. 병 입구와 주변을 조금 포함해 ROI 지정
   ↓
각 경우 뚜껑 있음 / 없음 화면 검사
   ↓
Area Ratio와 OK/NG가 얼마나 안정적인지 비교
   ↓
더 적절한 ROI와 이유 기록
```

**기록 예시**

| 시도 | ROI 범위 | 뚜껑 있음 Ratio | 뚜껑 없음 Ratio | 판정이 안정적인가? |
|---|---|---:|---:|---|
| A | 좁게 | 직접 기록 | 직접 기록 | 직접 판단 |
| B | 조금 넓게 | 직접 기록 | 직접 기록 | 직접 판단 |

스스로 답해 보세요.

1. 실제 병뚜껑이 있는데 NG가 나온다면 어떤 촬영조건이 원인일까요?
2. 같은 Rule인데 ROI만 바꾸어도 왜 Area Ratio가 달라질까요?
3. FastAPI가 없을 때도 `cap_inspector.py`의 검사 알고리즘은 동작할 수 있을까요?
4. 병뚜껑이 항상 ROI에 들어오게 만들려면 카메라·제품 위치를 어떻게 개선해야 할까요?

답변과 관찰은 `reports/day03_rule_analysis.md`에 새로운 소제목 **FastAPI 실시간 확장 실습**을 만들어 간단히 추가합니다.

---

### ROI가 제대로 인식을 못하여 NG에서 한참뒤 OK로 바뀌는 원인찾기

현재 코드에서 실시간 영상 처리의 속도를 개선해야 한다고 추측했을 경우

지금 코드는 카메라 영상 수신 → OpenCV 검사 → JPEG 변환 → FastAPI 전송 → 브라우저 화면 갱신으로 구성되어 있기 때문에 어느 단계가 느린지 구분해야 합니다.

#### 먼저 어떤 부분이 느린지 확인

현재 화면에서 다음 세 가지를 비교하면 원인을 어느 정도 찾을 수 있습니다.

|확인할 현상|의심되는 원인|수정할 파일|
|---|---|---|
|실제 병뚜껑을 바꿨는데 영상 자체가 늦게 바뀜|IP Webcam 수신 지연·영상 버퍼|`fastapi_cap_app.py`|
|영상은 바로 바뀌지만 영상 안의 NG 표시가 늦게 OK로 바뀜|OpenCV 판정, ROI, 기준값 또는 처리 지연|`cap_inspector.py`, `fastapi_cap_app.py`|
|영상 안에는 OK인데 오른쪽 결과창만 NG로 표시됨|상태 조회 주기 또는 서버 응답 지연|`cap_inspection.html`|
현재 실습은 OpenCV가 영상에 직접 `OK / NG` 문자를 그려 넣기 때문에, 영상 안의 결과와 오른쪽 대시보드 결과를 비교하는 것이 가장 간단한 진단 방법입니다.

#### 실제 수정할 주요 코드 4곳

##### ① 스마트폰 영상 수신 — 가장 먼저 확인할 부분

파일: `src/fastapi_cap_app.py`

기존 코드:

```python
cap = cv2.VideoCapture(url)
```

이 코드는 스마트폰의 HTTP MJPEG 영상을 받아옵니다.

네트워크나 영상 처리 속도가 느리면 오래된 프레임이 버퍼에 남아 뒤늦게 처리되는 경우가 있습니다.

먼저 다음처럼 시도해 볼 수 있습니다.

```python
cap = cv2.VideoCapture(url)

# 지원되는 경우 내부 프레임 버퍼 크기를 줄입니다.
cap.set(cv2.CAP_PROP_BUFFERSIZE, 1)
```

다만 IP Webcam과 OpenCV의 영상 처리 백엔드에 따라 `CAP_PROP_BUFFERSIZE` 설정은 지원되지 않거나 적용되지 않을 수 있습니다. 따라서 이것만으로 해결된다고 보기는 어렵습니다.

스마트폰 IP Webcam 앱에서 영상 해상도를 640×480, 10~15 FPS 정도로 낮춰 테스트하는 것도 추천합니다. 높은 해상도로 많은 프레임을 전송할수록 네트워크와 처리 부담이 커질 수 있습니다.

### ② JPEG 영상 전송 속도

파일: `src/fastapi_cap_app.py`

기존 코드:

```python
time.sleep(0.10)  # 최대 약 10회/초 전송
```

이 코드는 `/stream.mjpg`에서 브라우저로 전송하는 속도를 제한합니다.

학생 1~2명이 테스트하는 상황에서 영상 표시를 더 부드럽게 하고 싶다면 다음처럼 수정할 수 있습니다.

```python
time.sleep(0.05)  # 최대 약 20회/초 전송
```

하지만 이 설정은 화면 전송 간격만 줄이는 것입니다. OpenCV 검사 자체가 빨라지는 것은 아닙니다.

여러 학생이 동시에 접속했다면 오히려 네트워크 사용량이 증가할 수 있으므로, 그 경우에는 기존 `0.10`을 유지하는 편이 좋습니다.

### ③ 브라우저 검사 결과 갱신 속도

파일: `web/cap_inspection.html`

기존 코드:

```js
setInterval(refreshStatus, 400);
```

이 코드는 JavaScript가 FastAPI의 `/api/status` 엔드포인트에 0.4초마다 HTTP GET 요청을 보내는 부분입니다.

학생 1~2명이 테스트한다면 아래처럼 변경해 볼 수 있습니다.

```js
setInterval(refreshStatus, 200);
```

변경 후에는 약 0.2초마다 상태를 조회합니다.

다만 30명이 동시에 접속하면 서버에 보내는 상태 요청이 더 많아지므로 권장하지 않습니다. 또한 요청이 이전 요청보다 오래 걸리면 조회가 겹칠 수 있습니다.

NG가 몇 초 이상 지속되는 문제라면 이 400ms 설정보다는 앞단의 영상 지연이나 판정 로직을 먼저 확인해야 합니다.

### ④ JPEG 압축률 조절

파일: `src/fastapi_cap_app.py`

기존 코드:

```python
success, encoded = cv2.imencode(
    ".jpg", image, [cv2.IMWRITE_JPEG_QUALITY, 77]
)
```

다음처럼 변경할 수 있습니다.

```python
success, encoded = cv2.imencode(
    ".jpg", image, [cv2.IMWRITE_JPEG_QUALITY, 65]
)
```

이미지 품질을 조금 낮추는 대신 전송할 데이터의 크기를 줄이는 방법입니다. 여러 브라우저에 동일한 영상을 제공하는 경우 특히 도움이 될 수 있습니다.

단, 실제 판정에는 압축하기 전 `frame`을 사용하므로 위 변경이 판정 기준값 자체를 바꾸지는 않습니다.


## 3. 확인 순서

1단계 — 브라우저 한 개만 연결해서 테스트

다른 학생들의 접속을 중단한 상태에서 병뚜껑을 바꿔 보세요. 이때 빨라진다면 동시 접속에 따른 서버·네트워크 부하나 공유 상태 문제를 의심할 수 있습니다.

2단계 — 스마트폰 영상 해상도 낮추기

640×480으로 설정하고 OpenCV에서 영상이 지연 없이 바뀌는지 확인합니다.

3단계 — Area Ratio 확인

병뚜껑을 씌웠을 때 측정값이 즉시 기준값을 넘어가는지 확인합니다. 영상이 빠른데도 NG가 지속된다면 ROI 위치나 Otsu·Contour 기반 판정이 원인일 수 있습니다.

4단계 — 필요할 때만 전송 속도 조절

적은 수의 브라우저에서 테스트할 때 `0.10 → 0.05`, `400 → 200` 변경을 비교합니다.

### 결론

현재 현상에서 가장 먼저 확인할 부분은 `cv2.VideoCapture(url)`의 영상 수신 지연과 `cap_inspector.py`의 실제 Area Ratio 판정입니다. 그리고 다수가 같은 서버에서 ROI를 수정하고 있다면 공유 상태 구조도 개선해야 합니다.

이 두 원인을 확인하지 않고 무조건 HTTP 요청 횟수나 영상 전송 FPS만 높이면 오히려 더 느려질 수 있습니다.

---

## PART 12. 실습 완료 정리

### 12.1 자가 체크리스트

- [ ] 기존 1~3일차 프로젝트와 가상환경을 재사용했다.
- [ ] `cap_on.jpg`, `cap_off.jpg` 기준 이미지 두 장을 확인했다.
- [ ] FastAPI와 Uvicorn 설치 및 서버 실행에 성공했다.
- [ ] 웹브라우저에서 IP Webcam 영상을 확인했다.
- [ ] 마우스로 병뚜껑 ROI를 선택하고 초기화할 수 있다.
- [ ] 병뚜껑 있는 상태에서 OK가 나오는지 확인했다.
- [ ] 병뚜껑 없는 상태에서 NG가 나오는지 확인했다.
- [ ] Area Ratio와 기준값을 확인하고 판정 이유를 설명할 수 있다.
- [ ] 카메라 끊김/ROI 실패를 실제 불량과 구분할 수 있다.
- [ ] 정상 촬영 조건과 한계, 실패 사례를 기록했다.

### 12.2 Local Git에 코드만 정리합니다

기존 `.gitignore`가 `samples/`, `outputs/`, `.venv/`를 제외하고 있는지 확인한 뒤, 로컬 Git에 작업을 기록합니다. **교육기관의 폐쇄망·내부 저장소 지침을 우선**합니다.

```bash
git status
git add src/cap_inspector.py src/fastapi_cap_app.py src/__init__.py web/cap_inspection.html requirements.txt
git commit -m "feat: add IP Webcam FastAPI cap inspection"
git push 
```

### 12.3 오늘 한 줄 정리

> 1일차에는 검사할 위치(ROI)를 정했고, 2일차에는 검사하기 좋은 영상으로 준비했으며, 3일차에는 숫자로 판단하는 Rule을 만들었습니다. 이번 확장 실습에서는 그 Rule을 IP Webcam의 실시간 Frame에 연결하여 FastAPI 웹 화면에서 OK/NG를 확인했습니다.


