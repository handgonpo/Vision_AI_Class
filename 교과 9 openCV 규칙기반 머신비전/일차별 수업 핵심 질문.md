
|일차|핵심 질문|학생이 이해해야 할 것|
|---|---|---|
|1일차|어디를 검사할까?|Image, Frame, ROI|
|2일차|이미지를 어떻게 개선할까?|전처리와 이진화|
|3일차|무엇을 측정하고 판단할까?|Morphology, Contour, Rule|
|4일차|기준 로고와 얼마나 비슷할까?|Template, ORB, SIFT, Homography|
|5일차|OK/NG 기준을 어떻게 정할까?|Calibration, Rule Freeze, Test|
|6일차|인쇄된 문자를 어떻게 검사할까?|OCR, Regex, Expected Value|
|7일차|정상과 다른 제품을 어떻게 찾을까?|ResNet18 기반 이상검출|
|8일차|데이터가 적으면 어떻게 검사할까?|One-shot, Few-shot, 결함 합성|
|9일차|결과를 어떻게 검증하고 관리할까?|재현성, 실험관리, Drift, Hybrid|
|10일차|검사 기능을 다른 장치에서도 사용할 수 있을까?|FastAPI, JSON, Web, Client/Server|

### 전체 코드 흐름을 먼저 파악

#### 첫 번째 — 전체 흐름을 의사코드로 보여주기
코드를 작성하기 전에 입력부터 출력까지의 데이터 이동을 먼저 이해합니다.

#### 두 번째 — 의미 있는 기능 단위로 나누기
입력, 검증, 처리, 결과 확인, 출력과 저장으로 나누되 실제 프로그램 구조에 따라 조정합니다.

#### 세 번째 — Jupyter에서 중간 결과 확인
각 단계의 변수값이나 이미지 상태를 확인하고 학생이 처리 결과를 예측해 봅니다.

#### 네 번째 — `.py` 파일로 전체 실행
분리해서 이해한 코드가 하나의 프로그램으로 연결되는지 확인합니다.

### Jupyter로 코드를 분리해서 실행하는 방법

실제 1일차 `03_roi_crop.py`로 예를 들어보겠습니다

교재의 이 파일은 이미지에서 ROI를 자르고 저장하는 프로그램입니다.

먼저 코드 흐름을 이해합니다

```
이미지 경로 설정
       ↓
이미지 읽기
       ↓
이미지 읽기 성공 여부 확인
       ↓
이미지 Width / Height 확인
       ↓
ROI 좌표 설정
       ↓
좌표 범위 검증
       ↓
ROI 자르기
       ↓
ROI 이미지 저장
       ↓
원본에 사각형 표시
       ↓
Preview 이미지 저장
       ↓
결과 출력
```

Cell 1 — 이미지 입력과 확인

입력

```python
from pathlib import Path
import cv2

image_path = Path("samples/sample01.jpg")
image = cv2.imread(str(image_path))

if image is None:
    raise FileNotFoundError(image_path)

print("이미지 Shape:", image.shape)
print("자료형:", image.dtype)
  
```

학생에게 다음 질문을 합니다.

"이미지를 읽었는데 왜 바로 ROI를 자르지 않고 `image.shape`를 확인할까요?"

이미지를 읽었는지 확인하고, 가로·세로 크기를 알아야 올바른 ROI 좌표를 정할 수 있기 때문입니다.

Cell 2 — ROI 좌표 설정과 검증

검증

```python
height, width = image.shape[:2]

x1, y1 = 200, 100
x2, y2 = 700, 500

x1 = max(0, min(x1, width))
x2 = max(0, min(x2, width))
y1 = max(0, min(y1, height))
y2 = max(0, min(y2, height))

if x1 >= x2 or y1 >= y2:
    raise ValueError("ROI 좌표가 올바르지 않습니다.")

print("이미지 크기:", width, height)
print("ROI 좌표:", x1, y1, x2, y2)
  
```

학생들이 좌표값을 바꾸어 보고, 값이 이미지 범위를 벗어나면 어떻게 되는지 확인합니다.

Cell 3 — 실제 ROI 처리

로직

```python
roi = image[y1:y2, x1:x2]

print("원본:", image.shape)
print("ROI:", roi.shape)
print("ROI 픽셀 수:", roi.shape[0] * roi.shape[1])
  
```

이 단계가 ROI를 실제로 자르는 핵심 처리입니다. 출력된 Shape를 비교하면 원본에서 일부 영역만 추출되었다는 사실을 숫자로 확인할 수 있습니다.

Cell 4 — 눈으로 결과 확인

시각화

```python
import matplotlib.pyplot as plt

roi_rgb = cv2.cvtColor(roi, cv2.COLOR_BGR2RGB)

plt.imshow(roi_rgb)
plt.axis("off")
plt.show()
  
```

실제 잘라낸 부분이 맞는지 확인합니다. OpenCV는 BGR 순서를 사용하므로 Matplotlib에 표시할 때 RGB로 변환합니다.

그다음 원래 교재의 `03_roi_crop.py`를 완성해 직접 실행하고 ROI 및 Preview 파일이 정상 저장되는지 확인하면 됩니다.

이 과정이 바로 질문하신 '중간중간 print로 확인하며 코드 이해하기'를 실습에 적용한 방법입니다.

단순히 최종 결과가 나오는 것과 비교하면 학생들이 변수, 배열, 좌표, 이미지 처리의 관계를 훨씬 구체적으로 이해할 수 있습니다.

## 4.2 모든 중간 결과를 `print()`로 확인해야 할까요?

아닙니다. 데이터의 형태에 따라 적절한 확인 방법이 다릅니다.

|데이터 유형|확인할 내용|주로 사용하는 코드|
|---|---|---|
|숫자·변수|값, 자료형, 범위|`print()`, `type()`, `min()`, `max()`|
|이미지|크기, 채널, 실제 화면|`.shape`, `plt.imshow()`|
|Binary Image|흑백 결과, 흰색 픽셀 수|`plt.imshow()`, `np.count_nonzero()`|
|Contour|객체 개수, 면적, 윤곽선|`cv2.contourArea()`, `cv2.drawContours()`|
|Batch 처리|전체·성공·실패 수|`len()`, `print()`, `DataFrame`|
|CSV 결과|데이터 구조, 일부 결과|`pd.read_csv()`, `.head()`, `.info()`|
|AI Feature|Feature Shape, Score, 분포|`.shape`, `print()`, `plt.hist()`|
|최종 판정|OK/NG, 판정 이유, 성능지표|`print()`, `confusion_matrix()`|

# 5. Jupyter에서 단순히 순서대로 실행하면 안 되는 경우

1일차의 `05_video_input.py`, `07_webcam_input.py`, `08_phone_camera_roi.py`가 좋은 예입니다.

`05_video_input.py`에는 다음과 같은 코드가 있습니다.

```python
while True:
    ret, frame = cap.read()

    if not ret:
        break

    frame_count += 1
```

이 코드는 한 번 실행하고 끝나는 것이 아니라 영상이 끝날 때까지 반복됩니다.

따라서 전체 프로그램은 다음처럼 동작합니다.

```
VideoCapture
      ↓
while 시작
      ↓
Frame 읽기
      ↓
읽기 성공?
  ├─ YES → Frame 처리 → 반복
  └─ NO  → 반복 종료
      ↓
cap.release()
```

Notebook에서는 영상 전체를 처리하기 전에 일부 Frame만 읽어서 모양과 크기, ROI 결과를 확인하는 것이 좋습니다.

실시간 카메라를 사용하는 `08_phone_camera_roi.py`는 더 복잡합니다.

- 마우스로 ROI 선택
    
- 키보드 입력 처리
    
- `while` 반복
    
- ROI 저장
    
- 종료 및 자원 정리
    

이러한 기능이 결합되어 있으므로, Notebook에서 모든 코드를 조각내어 실행하면 오히려 프로그램이 어떻게 동작하는지 혼란스러울 수 있습니다.

특히 카메라 연결을 끊지 않거나 GUI 창이 종료되지 않는 문제가 생길 수 있습니다.

## 일차별 Notebook 사용 권장

|일차|Notebook 활용|최종 실행|
|---|---|---|
|1일차|이미지·ROI 적극 활용|카메라는 `.py`|
|2일차|전처리마다 적극 활용|전체 비교 `.py`|
|3일차|Binary·Contour·Feature 분석|Rule·Batch `.py`|
|4일차|Template·특징점 결과 비교|Homography 포함 `.py`|
|5일차|Calibration 분포·Threshold 시각화|고정 Rule·Test `.py`|
|6일차|ROI·전처리·OCR 문자열 확인|Batch OCR `.py`|
|7일차|Feature·Score·Heatmap 분석|전체 이상검출 `.py`|
|8일차|Similarity·Prototype 비교|전체 실험 `.py`|
|9일차|CSV·성능지표·Drift 비교|재현성 실험 `.py`|
|10일차|JSON·서비스 응답 구조 이해|FastAPI 서버 실행|

참고로 Jupyter는 같은 Kernel의 변수들을 셀끼리 공유합니다. 그래서 이전 셀을 건너뛰거나 일부만 재실행하면 오래된 변수 때문에 잘못된 결과가 나타날 수 있습니다. 최종 확인 시에는 Kernel Restart → Run All로 전체 흐름을 재현하는 습관을 들이면 좋습니다.

# 6. 학생들이 코드를 정말 이해했는지 확인하는 방법

여기서는 기존 제안에 하나를 더 보완하고 싶습니다.

단순히 코드를 분리해 실행하고 결과를 보는 것만으로는 학생들이 충분히 이해했다고 보기 어렵습니다.

결과를 먼저 예측하게 하는 과정을 추가하면 좋습니다.

예를 들어:

1. ROI 좌표를 바꾸면 저장 이미지가 어떻게 달라질까요?
    
2. Threshold 값을 높이면 흰색 영역이 어떻게 달라질까요? (이진화 방식에 따라 결과가 다릅니다.)
    
3. Contour 면적 기준을 바꾸면 OK/NG 판정이 어떻게 바뀔까요?
    
4. OCR이 12자리만 읽었다면 어떤 검사 단계에서 실패할까요?
    

이런 질문을 하고 학생들이 자신의 예측과 실제 실행 결과를 비교하도록 합니다.

```
코드 흐름 이해
        ↓
중간 결과 예상
        ↓
코드 실행
        ↓
예측과 실제 결과 비교
        ↓
왜 이런 결과가 나왔는지 설명
        ↓
파라미터 직접 변경
        ↓
결과 변화 확인
```

이 방식은 코드를 단순히 실행하는 것과 프로그램을 이해하는 것 사이의 차이를 줄이는 데 도움이 됩니다.