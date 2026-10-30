## 오늘의 핵심 질문

> 3D 모델을 가상 공간에 배치하고 카메라로 촬영하듯 이미지를 만들면, RGB 이미지와 함께 객체의 위치·영역·거리 정보도 자동으로 만들 수 있을까?

### 3일차와 4일차는 무엇이 다른가요?

![[Pasted image 20261014123938.png]]

3일차에는 배경 사진에 투명한 객체 PNG를 붙여 새로운 이미지를 만들었습니다.

```
[3일차 — 2D 이미지 합성]

배경 사진 JPG
        +
투명한 객체 PNG
        ↓
크기 · 회전 · 위치 변경
        ↓
Cut-Paste 합성
        ↓
합성 이미지 + YOLO BBox
```

4일차에는 **2D 사진 대신 실제 입체 형태를 가진 3D 모델**을 사용합니다.

가상 공간에 객체를 놓고 카메라와 조명을 설정하면, BlenderProc가 해당 장면을 이미지로 만들어 줍니다.

```
[4일차 — 3D 공간에서 이미지 생성]

3D 객체 모델
        +
가상 바닥
        +
카메라 · 조명
        ↓
객체 위치와 방향 설정
        ↓
BlenderProc로 Rendering
        ↓
RGB 이미지
+
BBox · Mask · Depth
```

### 어떤 결과가 만들어지나요?

|결과|쉽게 이해하면|
|---|---|
|RGB|카메라로 촬영한 것처럼 보이는 일반 이미지|
|COCO BBox|객체를 감싸는 사각형 좌표|
|Semantic Mask|픽셀마다 어떤 종류의 객체인지 표시|
|Instance Mask|개별 객체를 구분하여 표시|
|Depth|픽셀마다 카메라 기준 깊이 정보|

**최종 결과는 2D 이미지 또는 2D 형태의 데이터이지만, 이 데이터는 3D 공간의 정보를 이용해 생성됩니다.**

### Blender를 직접 사용하나요?

오늘은 Blender에서 직접 3D 모델을 제작하지 않습니다.

미리 준비된 **YCB-V 3D 모델**을 BlenderProc로 불러옵니다.

본 실습에서는 다음 두 객체를 사용합니다.

- `cracker_box` — 과자 상자
- `mustard_bottle` — 머스터드 소스병
    

Mini Challenge에서는 `power_drill`(전동드릴)을 사용합니다.

### 오늘 실습은 어떻게 진행하나요?

![[Pasted image 20261014124841.png]]

```
① 준비된 3D 모델과 실제 사진 확인
             ↓
② 3D 객체 · 카메라 · 조명 배치
             ↓
③ 첫 번째 Rendering 실행
             ↓
④ RGB · BBox · Mask · Depth 확인
             ↓
⑤ 카메라 · 조명 · 객체 조건 변경
             ↓
⑥ 여러 장의 합성데이터 자동 생성
             ↓
⑦ Automatic QA · Human QA
             ↓
⑧ 실제 사진과 합성 이미지 비교
             ↓
⑨ Mini Challenge
   카메라 거리만 변경하여 결과 비교
```

### 오늘 수업에서 가장 중요한 것은 무엇인가요?

3일차에는 **객체를 사진의 어느 위치에 붙였는지** 알고 있었기 때문에 BBox를 자동으로 만들 수 있었습니다.

4일차에는 한 단계 더 나아가 **3D 객체의 실제 형태와 위치, 카메라의 위치까지 알고 있기 때문에** BBox뿐 아니라 Mask와 Depth도 생성할 수 있습니다.

즉, 오늘의 목표는 단순히 3D 이미지를 많이 만드는 것이 아닙니다.

**3D 공간에서 AI 학습용 이미지와 정답 데이터를 함께 생성하고, 실제 사진과 비교하여 학습에 적합한지 판단하는 과정**을 배우는 것입니다.

---

# 3D 합성데이터(3D Synthetic Data)

![[Pasted image 20261014133905.png]]

## 1. 각 데이터의 의미

![[Pasted image 20261014134553.png]]


① RGB 합성 이미지 (Synthetic RGB Image)

3D 모델을 가상 카메라로 촬영하듯 렌더링한 일반 컬러 이미지입니다.
용도: 객체검출, 이미지 분류, Segmentation 등 Vision AI 모델의 학습 입력 데이터로 사용합니다.

---
② Bounding Box (COCO BBox)

이미지에서 객체가 어디에 있는지 사각형 좌표로 표시한 정답 데이터입니다.
용도: YOLO 같은 객체검출 모델이 객체의 종류와 위치를 학습할 때 사용합니다.

---
③ Semantic Segmentation Mask (의미적 분할 정답)

이미지의 각 픽셀이 어떤 종류(Class)에 속하는지 표시합니다. 같은 종류의 객체는 같은 Class ID로 표현합니다.
용도: 도로·사람·제품·불량 영역처럼 픽셀별로 무엇이 존재하는지 구분하는 AI 학습에 사용합니다.

---
④ Instance Segmentation Mask (개별 객체 분할 정답)

같은 종류라도 각각의 객체를 별도로 구분합니다. 예를 들어 병 3개가 있다면 각각 다른 Instance ID를 가집니다.
용도: 제품 개수 세기, 겹친 물체 구분, 로봇 피킹, 개별 객체 영역 검출 등에 활용합니다.

---
⑤ Depth Map (깊이 지도 / 거리 정답 데이터)

각 픽셀에 카메라 기준 깊이 값을 기록합니다. 사람이 보기 쉽도록 흑백으로 표시할 수 있지만, 실제 데이터에는 수치가 저장됩니다.
용도: 깊이 추정 AI 학습, 거리 분석, 로봇의 공간 인식, 장애물 회피 등에 활용합니다.

---
## 2. 실제 AI 학습에서는 어떻게 사용하나요?

RGB 이미지가 문제, BBox·Mask·Depth가 정답이라고 생각하면 쉽습니다.

|AI가 배우려는 능력|입력 데이터|정답 데이터|
|---|---|---|
|객체검출 (Object Detection)|RGB|BBox + Class|
|의미적 분할 (Semantic Segmentation)|RGB|Semantic Mask|
|개별 객체 분할 (Instance Segmentation)|RGB|Instance Mask|
|깊이 추정 (Depth Estimation)|RGB|Depth Map|

예를 들어 RGB 이미지에 과자 상자와 머스터드병이 보인다고 해보겠습니다.

객체검출 모델은 두 물체의 위치를 배우고, Segmentation 모델은 물체의 윤곽을 픽셀 단위로 배우며, Depth Estimation 모델은 이미지의 각 픽셀이 어느 깊이에 해당하는지를 배울 수 있습니다.

단, 4일차 수업은 이 모델들을 직접 학습시키는 수업이 아니라, 해당 모델의 학습에 필요한 데이터를 자동으로 생성하고 검수하는 실습입니다.

## 3. 4일차 실습에서 데이터는 어떻게 만들어지나요?

![[Pasted image 20261014135152.png]]

```
YCB-V 3D 모델
cracker_box + mustard_bottle
              ↓
BlenderProc
3D 객체 + 카메라 + 조명 설정
              ↓
3D Rendering
              ↓
┌───────────────────────────┐
│ Synthetic RGB Image       │
│ COCO Bounding Box         │
│ Semantic Segmentation     │
│ Instance Segmentation     │
│ Depth Map                 │
└───────────────────────────┘
              ↓
HDF5 + COCO Annotation 저장
              ↓
Automatic QA + Human QA
              ↓
AI 학습용 합성데이터 후보
```

업로드한 교재 기준으로 BlenderProc는 RGB·Depth·Segmentation 데이터를 HDF5에 저장하고, BBox 등은 COCO Annotation으로 저장합니다. 별도의 Preview 코드를 사용해 이를 PNG 이미지로 시각화합니다.

추가로 생성하는 `Metadata`는 객체 위치·카메라·조명·Seed 등의 생성 조건 기록이며, 실제 촬영한 `Real Reference`는 합성데이터가 아니라 합성 이미지와 비교하기 위한 실제 데이터입니다.

## 4. 이해할 핵심

3일차에는 2D 사진에 객체를 붙여 RGB + BBox를 생성했습니다.

4일차에는 3D 공간에서 객체를 렌더링하여 RGB + BBox + Segmentation + Depth를 생성합니다.

가장 중요한 차이는 3D 공간의 정보를 이미 알고 있으므로, 사람이 일일이 라벨링하지 않아도 여러 종류의 정답 데이터를 자동으로 만들 수 있다는 점입니다.

다만 자동 생성된 정답 데이터가 형식상 정확하더라도 합성 장면이 비현실적일 수 있으므로, 실제 학습에 사용하기 전 QA와 Real Reference 비교가 필요합니다.

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

- 3일차의 2D Cut-Paste와 4일차의 3D Rendering 차이를 설명할 수 있습니다.
- 1~3일차와 같은 교과 11 프로젝트를 이어서 사용할 수 있습니다.
- 1일차에서 만든 `.venv`를 그대로 재사용할 수 있습니다.
- 4일차에 제공된 `day04_student_data`의 `main`과 `challenge` 역할을 구분할 수 있습니다.
- YCB-V의 3D Model과 실제 촬영 Reference Image를 구분할 수 있습니다.
- BOP Object ID와 수업용 Class ID가 서로 다른 값이라는 점을 설명할 수 있습니다.
- `cracker_box`, `mustard_bottle`, `power_drill`의 고정 Class ID를 사용할 수 있습니다.
- Blender와 BlenderProc의 역할 차이를 설명할 수 있습니다.
- BlenderProc Script는 일반 `python`이 아니라 `blenderproc run`으로 실행해야 한다는 점을 이해할 수 있습니다.
- BOP 형식의 `.ply` 3D Object를 BlenderProc로 불러올 수 있습니다.
- 3D Object의 위치·회전과 Camera·Light의 역할을 설명할 수 있습니다.
- 하나의 3D Scene을 Rendering할 수 있습니다.
- 한 번의 Scene에서 RGB·Depth·Semantic Segmentation·Instance Segmentation을 생성할 수 있습니다.
- COCO Annotation에서 BBox가 자동 생성되는 흐름을 설명할 수 있습니다.
- Random Seed를 고정하고 Object·Camera·Light 조건을 바꾸어 여러 Synthetic Sample을 생성할 수 있습니다.
- 생성 조건을 Metadata JSON과 CSV로 기록할 수 있습니다.
- HDF5에 저장된 여러 Ground Truth를 Preview Image로 확인할 수 있습니다.
- COCO BBox와 HDF5 결과를 Automatic QA로 검사할 수 있습니다.
- Human QA가 Automatic QA와 다른 문제를 확인한다는 점을 설명할 수 있습니다.
- YCB-V 실제 Reference Image와 Synthetic Render를 비교하여 Domain Gap을 설명할 수 있습니다.
- Mini Challenge에서 `power_drill`로 같은 Pipeline을 스스로 반복할 수 있습니다.
- Mini Challenge에서 Camera Distance 하나만 변경하고 다른 Random 조건이 동일했는지 검증한 뒤 BBox 크기와 Domain Gap 변화를 비교할 수 있습니다.
- 오늘 만든 코드·실험 조건·결과를 README와 Git에 정리할 수 있습니다.

---

# 오늘 수업의 전체 흐름

![[Pasted image 20260908150044.png]]

4일차에는 준비된 **3D 모델·카메라·조명**을 이용해 BlenderProc로 한 장의 장면을 먼저 렌더링하고, RGB·BBox·Mask·Depth 같은 정답 데이터가 함께 만들어지는 원리를 이해합니다. 

그다음 위치·회전·카메라·조명 조건을 바꾸어 여러 장의 합성데이터를 자동 생성하고, 생성 조건을 Metadata로 기록합니다.  

마지막에는 생성 결과를 **QA로 검수하고 실제 이미지와 비교해 Domain Gap을 확인한 뒤**, 
Mini Challenge에서 다른 3D 객체로 같은 흐름을 스스로 반복합니다.

---

# 오늘 사용할 실제 데이터

## 1. 4일차에는 어떤 데이터를 사용하나요?

오늘은 **YCB-V 데이터셋**에서 수업에 필요한 3D 모델과 실제 촬영 이미지만 미리 준비한 `day04_student_data`를 사용합니다.

YCB-V는 실제 물체의 3D 모델과 카메라로 촬영한 이미지 등을 제공하는 데이터셋입니다.

오늘 사용할 자료는 크게 두 종류입니다.

- **3D Model (**`**.ply**`**)**: 가상 공간에서 물체를 배치하고 회전시키며 이미지를 생성하는 데 사용합니다.
    
- **Real Reference (**`**.png**`**)**: 실제 물체를 촬영한 사진입니다. 합성 이미지가 실제 사진과 
  얼마나 다른지 비교할 때 사용합니다.
    

수업용 데이터는 다음과 같이 구분되어 있습니다.

|폴더|사용 목적|포함된 객체|
|---|---|---|
|`main`|강사와 함께 진행하는 본 실습|과자 상자, 머스터드병|
|`challenge`|학생이 스스로 진행하는 Mini Challenge|전동드릴|

## 2. PLY 파일은 무엇인가요?

`.ply`는 **물체의 입체적인 형태를 저장하는 3D 데이터 파일 형식**입니다.

예를 들어 과자 상자는 단순한 평면 사진이 아니라 앞면·옆면·윗면 등을 가진 입체적인 물체입니다.

PLY 파일에는 이러한 입체 형상을 표현하는 점(Vertex)과 면(Face) 등의 정보가 저장될 수 있습니다.

따라서 PLY는 박스의 전개도 이미지가 아니라 **3D 공간에서 여러 방향으로 돌려 볼 수 있는 모델 데이터**입니다. 물체의 색상이나 재질 표현은 실제 모델에 포함된 정보에 따라 달라집니다.

오늘은 PLY 파일을 직접 제작하거나 수정하지 않습니다. BlenderProc를 이용해 준비된 3D 모델을 불러와 사용합니다.

## 3. 본 실습에서 사용할 `main` 데이터

본 실습에서는 **과자 상자와 머스터드병** 두 가지 객체를 사용합니다.

|구분|과자 상자|머스터드병|
|---|---|---|
|Class Name|`cracker_box`|`mustard_bottle`|
|3D 모델 파일|`obj_000002.ply`|`obj_000005.ply`|
|BOP Object ID|2|5|
|수업용 Class ID|1|2|
|실제 촬영 이미지|10장|10장|

**본 실습에서 사용하는 데이터는 3D 모델 2개와 실제 촬영 이미지 총 20장입니다.**

3D 모델은 BlenderProc에서 합성 이미지를 만드는 데 사용하고, 실제 촬영 이미지는 나중에 합성 결과와 비교하는 데 사용합니다.

## 4. Mini Challenge에서 사용할 `challenge` 데이터

Mini Challenge에서는 본 실습에 사용하지 않은 **전동드릴(`power_drill`)**을 사용합니다.

|확인 항목|데이터|
|---|---|
|Class Name|`power_drill`|
|3D 모델 파일|`obj_000015.ply`|
|BOP Object ID|15|
|수업용 Class ID|3|
|실제 촬영 이미지|10장|

학생들은 본 실습에서 만든 Python 코드를 재사용하여 새로운 3D 객체로 합성데이터를 생성하고, 카메라 거리 변경에 따른 결과 차이를 비교합니다.


---

## 5. 제공된 폴더와 파일의 위치 및 역할을 확인합니다

오늘은 제공받은 `day04_student_data` 폴더 안의 데이터를 사용합니다.

이 폴더에는 **3D 모델, 실제 촬영 사진, 모델 번호와 설정 정보**가 들어 있습니다.

### ① 먼저 3D 모델이 저장된 위치를 확인합니다

현재 Windows 탐색기에서 확인한 경로입니다.

`day04_student_data/main/bop/ycbv/`

이 폴더에는 다음 파일과 폴더가 있습니다.

```
ycbv/
├─ models/             ← 3D 모델 보관 폴더
├─ camera_cmu.json     ← 원본 카메라 정보
├─ camera_uw.json      ← 원본 카메라 정보
└─ dataset_info.md     ← 데이터셋 설명
```

`models` 폴더를 더블클릭하면 본 실습에 사용할 3D 모델이 들어 있어야 합니다.

```
models/
├─ models_info.json
├─ obj_000002.ply      ← 과자 상자
└─ obj_000005.ply      ← 머스터드병
```

여기서 `.ply`는 **물체의 입체적인 모양을 저장한 3D 모델 파일**입니다.

BlenderProc는 이 모델을 불러와 가상 공간에서 위치와 방향을 바꾸고 이미지를 생성합니다.

### ② 각 파일은 어디에 있으며 무슨 역할을 하나요?

아래 경로는 모두 `day04_student_data/`를 기준으로 합니다.

|파일 또는 폴더 경로|학생이 이해할 역할|
|---|---|
|`main/bop/ycbv/models/obj_000002.ply`|과자 상자의 3D 모델|
|`main/bop/ycbv/models/obj_000005.ply`|머스터드병의 3D 모델|
|`main/bop/ycbv/models/models_info.json`|각 3D 모델의 크기·경계 등에 관한 정보|
|`main/model_previews/`|3D 모델의 모습을 미리 보는 이미지|
|`main/real_reference/cracker_box/`|과자 상자를 실제 촬영한 이미지 10장|
|`main/real_reference/mustard_bottle/`|머스터드병을 실제 촬영한 이미지 10장|
|`main/data_config.json`|본 실습에서 불러올 모델 번호 등의 설정|
|`main/bop/ycbv/camera_uw.json`|원본 YCB-V 카메라 정보|
|`main/bop/ycbv/camera_cmu.json`|원본 YCB-V 카메라 정보|

**3D 모델과 실제 촬영 사진은 역할이 다릅니다.**

- **PLY 3D 모델:** 새로운 합성 이미지를 만드는 데 사용합니다.
    
- **Real Reference 사진:** 생성한 합성 이미지와 실제 사진을 비교하는 데 사용합니다.
    

참고로 `camera_uw.json`, `camera_cmu.json`은 원본 데이터의 카메라 정보를 담고 있습니다. 오늘 실습에서는 BlenderProc로 가상 카메라를 직접 설정하므로 이 파일을 렌더링에 직접 사용하지 않습니다.

### ③ Mini Challenge 데이터는 어디에 있나요?

Mini Challenge에서는 `main`이 아니라 `challenge` 폴더의 데이터를 사용합니다.

|경로|역할|
|---|---|
|`challenge/bop/ycbv/models/obj_000015.ply`|전동드릴 3D 모델|
|`challenge/bop/ycbv/models/models_info.json`|전동드릴 모델 정보|
|`challenge/model_previews/power_drill_model.png`|전동드릴 모델 미리보기|
|`challenge/real_reference/power_drill/`|전동드릴 실제 촬영 이미지 10장|
|`challenge/data_config.json`|Challenge에서 사용할 모델 설정|

### ④ 공통 설정 파일은 어디에 있나요?

다음 파일은 `main`이나 `challenge` 안이 아니라 `**day04_student_data**` **바로 아래**에 있습니다.

|경로|역할|
|---|---|
|`day04_student_data/classes.txt`|수업에서 사용하는 객체 종류의 이름|
|`day04_student_data/object_map.json`|3D 모델 번호와 수업용 Class 번호를 연결|
|`day04_student_data/README_DATA.txt`|제공 데이터 사용 안내|

이 설정 파일들은 Python 코드가 사용할 정보를 담고 있으므로 파일 이름이나 내용을 임의로 변경하지 않습니다.

---

## 6. BOP Object ID와 Class ID를 쉽게 이해합니다

![[Pasted image 20261014150434.png]]

오늘은 하나의 객체에 두 종류의 번호를 사용합니다.

처음에는 비슷해 보이지만 역할이 다릅니다.

**BOP Object ID는 3D 모델을 찾는 번호**이고, **Class ID는 생성된 이미지에서 객체 종류를 나타내는 정답 번호**입니다.

### ① 실제 데이터로 이해해 봅니다

|객체|BOP Object ID|수업용 Class ID|
|---|---|---|
|배경 / 바닥|없음|0|
|과자 상자 (`cracker_box`)|2|1|
|머스터드병 (`mustard_bottle`)|5|2|
|전동드릴 (`power_drill`)|15|3|

예를 들어 과자 상자를 합성한다고 생각해 보겠습니다.

```
[3D 모델을 불러올 때]

BOP Object ID = 2
         ↓
obj_000002.ply 찾기
         ↓
과자 상자 3D 모델 불러오기


[정답 데이터를 생성할 때]

Lesson Class ID = 1
         ↓
이 객체는 과자 상자라는 의미
         ↓
BBox · Segmentation 정답에 사용
```

머스터드병도 같은 방식입니다.

```
BOP Object ID = 5
→ obj_000005.ply 불러오기

Lesson Class ID = 2
→ 생성된 정답 데이터에서
   머스터드병으로 구분
```

`Class ID = 0`은 배경이나 바닥 영역을 구분하기 위한 값이며, 별도의 YCB-V 3D 모델 파일이 있는 것은 아닙니다.

---
### ② BOP Object ID와 Class ID는 왜 다른가요?

두 번호는 사용하는 목적이 다릅니다.

- BOP Object ID: 어떤 3D 모델 파일을 불러올지 알려주는 번호입니다.
    
- Class ID: 생성된 이미지에서 객체의 종류를 구분하는 정답 번호입니다.
    

예를 들어 과자 상자는 BOP ID 2로 3D 모델을 불러오고, 생성된 정답 데이터에는 Class ID 1로 기록합니다.

따라서 두 번호가 같을 필요는 없습니다.

### 핵심 정리

오늘은 직접 3D 모델을 만들지 않고, 미리 준비된 PLY 파일을 사용합니다.

```
PLY 3D 모델 불러오기
        ↓
BlenderProc로 이미지 생성
        ↓
RGB · BBox · Mask · Depth 생성
        ↓
실제 사진과 비교
```

오늘의 목표는 준비된 3D 모델을 활용하여 AI 학습용 이미지와 정답 데이터를 자동으로 만드는 방법을 배우는 것입니다.

---
# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | 경로·검사·Metadata·QA | 4일차 전체 자동화 |
| pathlib | 폴더·파일 경로 관리 | Main과 Challenge 연결 |
| BlenderProc | 3D Scene 자동 Rendering | Blender를 Python으로 제어 |
| BOP Loader | YCB-V `.ply` Model Loading | ID로 3D 모델 불러오기 |
| YCB-V | 3D Model + 실제 촬영 Reference | Real과 Synthetic 비교 |
| Camera | 촬영 위치·거리·시점 | 화면 구성 제어 |
| Light | 위치·밝기 | 조명 조건 제어 |
| 3D Rotation | 객체 방향 변경 | 여러 방향의 Object 생성 |
| RGB | 모델 입력 이미지 | 가상 Camera 결과 |
| Depth | Camera와 Pixel 거리 | 거리 정보 |
| Semantic Segmentation | Class별 Mask | 종류별 영역 |
| Instance Segmentation | Object별 Mask | 객체별 영역 |
| COCO Annotation | BBox·Mask 정보 | Detection/Segmentation 정답 |
| HDF5 | 여러 Rendering 결과 저장 | 결과 묶음 |
| Randomization | Camera·Light·Object 변화 | 다양한 Synthetic Data 생성 |
| Metadata | Seed·Camera·Light·Object 기록 | 생성 조건 추적 |
| Automatic QA | 파일·BBox·Ground Truth 검사 | 구조 오류 자동 확인 |
| Human QA | 현실성·Domain Gap 판단 | 사람이 품질 확인 |
| Git | 코드·설명 기록 | 재현 가능한 결과 관리 |

---

# PART 1. 3일차 프로젝트에서 4일차를 이어서 시작합니다

---
# 1. 1~4일차가 어떻게 연결되는지 확인합니다

```text
1일차
Real Image → Augmentation / VAE

2일차
Real Train → DCGAN / WGAN-GP
Pretrained Diffusion → Synthetic Image

3일차
2D Background + RGBA Object
→ Cut-Paste
→ RGB + BBox

4일차
3D Object + Camera + Light
→ Rendering
→ RGB + BBox + Mask + Depth
```

3일차와 4일차의 공통점은 다음입니다.

```text
프로그램이 Object의 위치를 알고 있음
        ↓
정답 Label 자동 생성 가능
```

차이는:

```text
3일차 → 2D Pixel 공간
4일차 → 3D World 공간
```

입니다.

---
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
```

4일차 폴더를 만듭니다.

```bash
mkdir -p day04_3d_segmentation
cd day04_3d_segmentation
```

---
# 3. 4일차 폴더와 파일을 준비합니다

```bash
mkdir -p \
src \
scripts \
data \
results/main \
results/challenge \
reports/main \
reports/challenge
```

```bash
touch requirements_day04.txt
touch src/day04_common.py

touch scripts/00_check_day04.py
touch scripts/01_preview_real_reference.py
touch scripts/02_render_single_scene.py
touch scripts/03_export_hdf5_preview.py
touch scripts/04_render_random_sample.py
touch scripts/05_generate_dataset.py
touch scripts/06_collect_metadata.py
touch scripts/07_qa_3d_dataset.py
touch scripts/08_compare_real_synthetic.py
touch scripts/09_compare_experiments.py

touch day04_notes.md
touch README.md
touch .gitignore
```

```bash
tree -L 2
```

예상:

```text
day04_3d_segmentation/
├─ data/
├─ reports/
├─ results/
├─ scripts/
├─ src/
├─ requirements_day04.txt
├─ day04_notes.md
├─ README.md
└─ .gitignore
```

---
# 4. 제공된 `day04_student_data`를 배치합니다

수업에서 제공된 `day04_student_data`를 다음 위치에 복사합니다.

```text
day04_3d_segmentation/
└─ data/
   └─ day04_student_data/
```

```bash
explorer.exe .
```

확인합니다.

```bash
tree data/day04_student_data -L 5
```

모든 `:Zone.Identifier` 파일 삭제

![[Pasted image 20261014151621.png]]

```bash
find data/day04_student_data -type f -name '*:Zone.Identifier' -delete
```

# 5. 1일차 가상환경을 그대로 사용합니다

```bash
source ../.venv/bin/activate
```

```bash
which python
```

새로운 가상환경을 만들지 않습니다.

---
# 6. Blender와 BlenderProc의 역할을 구분합니다

![[Pasted image 20261014150730.png]]

| 구분              | Blender          | BlenderProc     |
| --------------- | ---------------- | --------------- |
| 주된 목적           | 3D 모델링·장면 제작·렌더링 | AI 학습용 합성데이터 생성 |
| 작업 방식           | 화면에서 마우스·키보드로 조작 | Python 코드로 자동화  |
| 카메라·조명          | 직접 배치·조절         | 코드로 자동 배치·조절    |
| 이미지 생성          | 가능               | 가능              |
| BBox·Mask·Depth | 기능·스크립트로 생성 가능   | 데이터 생성 기능을 제공   |
| 여러 장 자동 생성      | Python 자동화로 가능   | 자동 생성 작업에 특화    |

오늘은 복잡한 3D 모델링을 하지 않습니다.

이미 준비된 YCB-V 3D Model을 불러옵니다.

실행 방식도 구분합니다.

```text
일반 Python Script
→ python scripts/00_check_day04.py

BlenderProc Script
→ blenderproc run scripts/02_render_single_scene.py
```

---
# 7. 4일차 추가 라이브러리를 설치합니다

`requirements_day04.txt`:

```text
blenderproc==2.7.0
h5py
matplotlib
numpy
pandas
Pillow
```

설치:

```bash
python -m pip install -r ../requirements_day04.txt
```

먼저 일반 Python 환경에서 필요한 패키지가 보이는지 확인합니다.

```bash
python -c "import h5py, matplotlib, numpy, pandas, PIL; print('Day04 Python libraries OK')"
```

그다음 BlenderProc CLI가 정상인지 확인합니다.

```bash
blenderproc --help
```

이 명령이 정상적으로 표시되기 전에는 Rendering Script로 넘어가지 않습니다.

결과

```
BlenderProc: A procedural Blender pipeline for photorealistic image generation.
```

`blenderproc --help`가 정상적으로 출력되었으므로 BlenderProc 설치 자체는 성공한 상태입니다.

환경 기록:

```bash
pip freeze > reports/environment_freeze_day04.txt
```

---
# 8. BlenderProc Quickstart로 Rendering 환경을 확인합니다

`libSM.so.6`는 Linux에서 그래픽 프로그램이 사용하는 시스템 라이브러리입니다.

```bash
sudo apt update
sudo apt install -y libsm6
```

BlenderProc가 실제로 3D 장면을 렌더링할 수 있는지 확인하는 기본 테스트입니다.

```bash
blenderproc quickstart
```

첫 실행에서는 커널 준비에 수 분에서 10~20분 이상 걸리는 경우도 있습니다

결과 확인:

```bash
find output -maxdepth 2 -type f
```

HDF5 시각화:

```bash
blenderproc vis hdf5 output/0.hdf5
```

정상 확인 후 삭제합니다.
`blenderproc quickstart`는 BlenderProc가 정상적으로 작동하는지 확인하기 위한 테스트 실습입니다.

```bash
rm -rf output
```

```
blenderproc quickstart
        ↓
테스트 이미지와 HDF5 생성
        ↓
blenderproc vis hdf5 output/0.hdf5
        ↓
생성된 결과가 정상인지 확인
        ↓
rm -rf output
        ↓
테스트용 파일 정리
```

테스트가 끝나면 실제 4일차 실습에서는 별도의 `results/` 폴더에 데이터를 저장하므로, 테스트용 `output` 폴더는 더 이상 필요하지 않습니다.

---
실제 수업 데이터로 넘어가 다음 순서대로 실습합니다.

```text
[1단계 - 실행환경 확인]

Python 라이브러리 확인
        ↓
blenderproc --help
        ↓
blenderproc quickstart
        ↓
output/0.hdf5 생성 확인


[2단계 - 실제 데이터 확인]

day04_student_data 확인
        ↓
PLY 3D 모델과 파일 경로 확인
        ↓
Main Single Render
        ↓
RGB · BBox · Mask · Depth 확인


[3단계 - 자동 생성 테스트]

Random Sample 1장 생성
        ↓
카메라 · 조명 · 객체 조건 기록 확인
        ↓
문제가 없으면 여러 장 자동 생성
(Batch Rendering)
```

**왜 한 장씩 먼저 테스트하나요?**

Quickstart는 BlenderProc가 기본적으로 실행되는지 확인하는 테스트입니다.

Single Render는 실제 과자 상자와 머스터드병 3D 모델을 불러와 이미지와 정답 데이터를 만들 수 있는지 확인합니다.

Random Sample은 카메라·조명·객체 조건을 자동으로 변경하고, 그 결과를 기록할 수 있는지 확인합니다.

이 세 단계가 정상적으로 완료되면 여러 장의 합성데이터를 생성합니다.

처음부터 많은 이미지를 만들지 않고 **한 장씩 확인한 뒤 다음 단계로 진행하면 오류를 쉽게 찾을 수 있습니다.**

---

# 9. Main과 Challenge의 공통 경로 코드를 작성합니다

**파일:** `**src/day04_common.py**`

## 이 코드는 무엇을 하나요?

4일차에는 두 종류의 데이터를 사용합니다.

- `main`: 과자 상자와 머스터드병으로 진행하는 본 실습
    
- `challenge`: 전동드릴로 진행하는 Mini Challenge
    

두 실습은 사용하는 객체가 다르지만, 데이터를 읽고 결과를 저장하는 방법은 같습니다.

예를 들어 3D 모델을 불러올 때는 다음 경로를 사용합니다.

```text
[Main]
data/day04_student_data/main/bop/ycbv/

[Challenge]
data/day04_student_data/challenge/bop/ycbv/
```

매번 Python 코드에 이 경로를 직접 작성하면 경로를 잘못 입력하거나 `main`과 `challenge`를 혼동할 수 있습니다.

그래서 **자주 사용하는 폴더 경로를 한곳에서 관리하는 공통 파일**을 만듭니다.

이 파일은 다음 작업을 담당합니다.

|기능|쉽게 이해하면|
|---|---|
|프로젝트 위치 확인|4일차 작업 폴더를 찾습니다.|
|데이터 경로 확인|`main` 또는 `challenge` 폴더를 찾습니다.|
|설정 파일 읽기|`data_config.json`, `object_map.json`을 읽습니다.|
|3D 모델 경로 확인|BlenderProc가 사용할 `bop/ycbv` 폴더를 찾습니다.|
|실제 사진 경로 확인|`real_reference` 폴더를 찾습니다.|
|결과 저장 경로 준비|`results/`, `reports/`의 필요한 폴더를 만듭니다.|

### 의사코드로 먼저 이해해 봅니다

```text
4일차 프로젝트 폴더 찾기
        ↓
day04_student_data 위치 확인
        ↓
main 또는 challenge 선택
        ↓
선택한 데이터의 설정과 경로 찾기
        ↓
3D 모델 · 실제 사진 경로 확인
        ↓
결과 저장 폴더 준비
```

예를 들어 `main`을 선택하면 과자 상자와 머스터드병의 데이터 경로를 찾고, `challenge`를 선택하면 전동드릴의 데이터 경로를 찾습니다.

**중요:** 이 파일이 직접 이미지를 생성하는 것은 아닙니다. 이후에 작성할 렌더링·검사 코드들이 필요한 파일을 정확하게 찾을 수 있도록 도와주는 역할입니다.

이제 다음 코드를 작성합니다.

**파일: `src/day04_common.py`**

```python
from __future__ import annotations

import json
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]

DATA_ROOT = (
    ROOT
    / "data"
    / "day04_student_data"
)


def load_json(path: Path):
    if not path.exists():
        raise FileNotFoundError(path)

    with path.open(
        "r",
        encoding="utf-8",
    ) as file:
        return json.load(file)


def asset_root(asset_set: str) -> Path:
    if asset_set not in {
        "main",
        "challenge",
    }:
        raise ValueError(
            f"지원하지 않는 asset_set: {asset_set}"
        )

    return DATA_ROOT / asset_set


def data_config(asset_set: str) -> dict:
    return load_json(
        asset_root(asset_set)
        / "data_config.json"
    )


def bop_dataset_path(asset_set: str) -> Path:
    return (
        asset_root(asset_set)
        / "bop"
        / "ycbv"
    )


def object_map() -> dict:
    return load_json(
        DATA_ROOT
        / "object_map.json"
    )


def lesson_objects(
    asset_set: str,
) -> list[dict]:
    mapping = object_map()

    return [
        item
        for item in mapping["objects"]
        if item["usage"] == asset_set
    ]


def real_reference_root(
    asset_set: str,
) -> Path:
    return (
        asset_root(asset_set)
        / "real_reference"
    )


def result_root(
    asset_set: str,
    tag: str,
) -> Path:
    path = (
        ROOT
        / "results"
        / asset_set
        / tag
    )

    path.mkdir(
        parents=True,
        exist_ok=True,
    )

    return path


def report_root(
    asset_set: str,
) -> Path:
    path = (
        ROOT
        / "reports"
        / asset_set
    )

    path.mkdir(
        parents=True,
        exist_ok=True,
    )

    return path
```

### 코드의 핵심 흐름

```
4일차 프로젝트 위치 찾기
ROOT
       ↓
제공 데이터 폴더 찾기
DATA_ROOT
       ↓
main 또는 challenge 선택
asset_root()
       ↓
필요한 데이터 경로와 정보 가져오기
├─ data_config()        → 실습 설정 읽기
├─ bop_dataset_path()   → 3D 모델 폴더 찾기
├─ object_map()         → 객체 번호 정보 읽기
├─ lesson_objects()     → 사용할 객체 목록 선택
└─ real_reference_root()→ 실제 촬영 사진 폴더 찾기
       ↓
결과 저장 폴더 준비
├─ result_root()        → results/
└─ report_root()        → reports/
       ↓
다른 실습 코드에서 재사용
```

이 코드는 직접 이미지를 생성하지 않고, 이후 렌더링 및 검사 코드가 필요한 데이터를 정확히 찾도록 도와줍니다.

### ② 핵심적으로 수정할 수 있는 부분

1. 데이터 폴더 이름이 변경될 때

```python
DATA_ROOT = ROOT / "data" / "day04_student_data"
```

`day04_student_data`를 다른 이름으로 변경했다면 이 경로도 수정해야 합니다.

2. 실습 종류가 추가될 때

```python
if asset_set not in {"main", "challenge"}:
```

현재는 `main`과 `challenge`만 사용할 수 있습니다. 새로운 실습 종류를 추가한다면 이 부분도 변경해야 합니다.

3. 결과 저장 폴더를 변경할 때

```python
ROOT / "results" / asset_set / tag
```

`results/` 대신 다른 폴더에 저장하려면 해당 경로를 변경하면 됩니다.

---
# 10. 실제 수업 데이터가 정확한지 자동으로 검사합니다

## 이 코드는 무엇을 하나요?

앞에서 `day04_student_data`의 폴더와 파일을 확인했습니다.

이제 Python 코드로 실습에 필요한 3D 모델과 실제 촬영 이미지가 빠짐없이 준비되어 있는지 자동으로 검사합니다.

데이터가 누락되거나 파일명이 잘못되면 BlenderProc가 실행 중 오류를 일으킬 수 있으므로, 렌더링을 시작하기 전에 먼저 확인하는 과정입니다.

### 어떤 내용을 자동으로 검사하나요?

|검사 항목|확인하는 내용|
|---|---|
|실습 데이터 구분|`main`은 객체 2개, `challenge`는 객체 1개인지 확인|
|3D 모델 파일|`models_info.json`과 필요한 `.ply` 파일이 존재하는지 확인|
|모델 번호|`data_config.json`의 BOP ID와 `object_map.json`의 BOP ID가 연결되는지 확인|
|실제 촬영 이미지|객체별로 사진이 10장씩 준비되어 있는지 확인|
|이미지 파일 상태|PNG·JPG 파일을 정상적으로 읽을 수 있는지 확인|

예를 들어 `main`을 검사하면 과자 상자의 `obj_000002.ply`, 머스터드병의 `obj_000005.ply`가 있는지 확인하고, 각 객체의 실제 사진도 10장씩 검사합니다.

## 코드의 핵심 흐름

```
main 또는 challenge 선택
        ↓
설정 파일과 객체 목록 읽기
        ↓
필요한 3D 모델 파일 확인
        ↓
BOP Object ID 확인
        ↓
실제 촬영 이미지 10장씩 확인
        ↓
이미지 파일이 정상인지 검사
        ↓
문제가 있으면 오류 메시지 출력
        ↓
모든 검사 통과 시
Day 04 data check: READY
```

**파일: `scripts/00_check_day04.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import Image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day04_common import (
    asset_root,
    bop_dataset_path,
    data_config,
    lesson_objects,
    real_reference_root,
)


EXPECTED = {
    "main": {
        "object_count": 2,
        "real_per_class": 10,
    },
    "challenge": {
        "object_count": 1,
        "real_per_class": 10,
    },
}


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--asset-set",
        default="main",
        choices=["main", "challenge"],
    )
    return parser.parse_args()


def image_files(directory: Path) -> list[Path]:
    valid = {".png", ".jpg", ".jpeg"}

    if not directory.exists():
        return []

    return sorted(
        path
        for path in directory.iterdir()
        if path.is_file()
        and path.suffix.lower() in valid
    )


def main() -> None:
    args = parse_args()

    expected = EXPECTED[args.asset_set]
    source_root = asset_root(args.asset_set)
    bop_root = bop_dataset_path(args.asset_set)
    config = data_config(args.asset_set)
    objects = lesson_objects(args.asset_set)

    print("=" * 64)
    print("Subject 11 - Day 04 Data Check")
    print("=" * 64)
    print("Asset Set:", args.asset_set)
    print("Source   :", source_root)

    if len(objects) != expected["object_count"]:
        raise RuntimeError(
            "Object Mapping 수가 수업 기준과 다릅니다."
        )

    if not (
        bop_root
        / "models"
        / "models_info.json"
    ).exists():
        raise FileNotFoundError(
            "models_info.json이 없습니다."
        )

    config_ids = [
        int(value)
        for value in config["bop_obj_ids"]
    ]

    reference_root = real_reference_root(
        args.asset_set
    )

    for item in objects:
        bop_id = int(item["bop_obj_id"])
        class_name = item["class_name"]
        lesson_id = int(
            item["lesson_class_id"]
        )

        if bop_id not in config_ids:
            raise RuntimeError(
                f"{class_name}의 BOP ID가 "
                "data_config.json에 없습니다."
            )

        model_path = (
            bop_root
            / "models"
            / f"obj_{bop_id:06d}.ply"
        )

        if not model_path.exists():
            raise FileNotFoundError(model_path)

        real_dir = (
            reference_root
            / class_name
        )

        real_files = image_files(real_dir)

        if len(real_files) != (
            expected["real_per_class"]
        ):
            raise RuntimeError(
                f"{class_name} Real Reference 수가 "
                "수업 기준과 다릅니다.\n"
                f"기대: {expected['real_per_class']}\n"
                f"현재: {len(real_files)}"
            )

        unreadable = []

        for path in real_files:
            try:
                with Image.open(path) as image:
                    image.verify()
            except Exception:
                unreadable.append(path.name)

        if unreadable:
            raise RuntimeError(
                f"{class_name}에서 읽을 수 없는 "
                "이미지가 있습니다: "
                + ", ".join(unreadable)
            )

        print("-" * 64)
        print("Class Name      :", class_name)
        print("Lesson Class ID :", lesson_id)
        print("BOP Object ID   :", bop_id)
        print("3D Model        :", model_path.name)
        print("Real Reference  :", len(real_files))

    print("=" * 64)
    print("Day 04 data check: READY")


if __name__ == "__main__":
    main()
```

### 코드의 핵심 구조 

```text
main 또는 challenge 선택
parse_args()
        ↓
실습에 필요한 데이터 기준 확인
EXPECTED
        ↓
데이터 폴더와 설정 정보 가져오기
day04_common.py의 공통 함수 사용
        ↓
객체 수 확인
main: 2개 / challenge: 1개
        ↓
3D 모델 파일 확인
models_info.json
obj_XXXXXX.ply
        ↓
BOP Object ID 일치 여부 확인
data_config.json
        ↓
실제 촬영 이미지 검사
객체별 10장
        ↓
이미지 파일을 정상적으로 읽을 수 있는지 확인
PIL.Image.verify()
        ↓
오류 발생 → 오류 메시지 출력 후 중단
정상 완료 → Day 04 data check: READY
```

## 상황에 따라 수정할 수 있는 핵심 코드

### 1. 검사할 객체 수와 이미지 수를 변경할 때

```python
EXPECTED = {
    "main": {
        "object_count": 2,
        "real_per_class": 10,
    },
    "challenge": {
        "object_count": 1,
        "real_per_class": 10,
    },
}
```

가장 중요한 설정 부분입니다.

현재는 본 실습에서 객체 2개, Challenge에서 객체 1개를 검사하며, 각 객체의 실제 사진은 10장씩 있어야 통과합니다.

예를 들어 실제 사진을 객체별 20장으로 늘렸다면 `real_per_class`를 `20`으로 변경할 수 있습니다.

단, 실제 제공 데이터의 수량과 일치하도록 수정해야 합니다.

### 2. 기본 검사 대상을 변경할 때

```python
parser.add_argument(
    "--asset-set",
    default="main",
    choices=["main", "challenge"],
)
```

- `default="main"`: 옵션을 입력하지 않으면 Main 데이터를 검사합니다.
    
- `choices`: 검사할 수 있는 데이터 종류를 제한합니다.
    

---
Challenge를 검사할 때는 코드를 수정할 필요 없이 실행 옵션만 변경하면 됩니다.

```bash
python scripts/00_check_day04.py --asset-set challenge
```

`--asset-set`처럼 앞에 `--`가 붙은 것은 명령줄 옵션(Command-line Option) 또는 CLI 옵션이라고 부릅니다.

CLI 옵션의 가장 큰 장점은 같은 Python 코드를 수정하지 않고도 데이터, 실험 조건, 저장 이름 등을 변경해 반복 실행할 수 있다는 것입니다.

---

### 3. 검사할 이미지 확장자를 추가할 때

```python
valid = {".png", ".jpg", ".jpeg"}
```

현재는 PNG와 JPG 이미지만 검사합니다.

다른 형식을 지원해야 한다면 확장자를 추가할 수 있습니다. 단, Pillow가 실제로 읽을 수 있는 형식이어야 합니다.

### 4. 3D 모델 파일명을 만드는 규칙

```python
f"obj_{bop_id:06d}.ply"
```

이 코드는 BOP Object ID를 파일명으로 바꿉니다.

예를 들어 다음과 같습니다.

```
BOP ID = 2
→ obj_000002.ply

BOP ID = 5
→ obj_000005.ply

BOP ID = 15
→ obj_000015.ply
```

`06d`는 숫자를 여섯 자리로 만들고, 앞의 빈자리를 `0`으로 채운다는 뜻입니다.

오늘 사용하는 YCB-V 데이터는 이 파일명 규칙을 따르므로 수정하지 않습니다.

### 핵심 정리

이 코드에서 학생들이 가장 중요하게 이해해야 할 부분은 **`EXPECTED`와 `--asset-set`**입니다.

- `EXPECTED`는 무엇을 몇 개 검사할지 정합니다.
    
- `--asset-set`은 어떤 실습 데이터를 검사할지 선택합니다.
    

나머지 코드는 이 기준에 따라 파일을 검사하는 기능입니다.

---

# PART 2. 3D Object와 Ground Truth 구조를 이해합니다

---

# 11. 실제 촬영 Reference Image를 확인합니다

## 이 코드는 무엇을 하나요?

앞에서 3D 모델과 실제 촬영 이미지가 정상적으로 준비되어 있는지 검사했습니다.

이번에는 **실제 촬영 이미지 10장씩을 하나의 큰 이미지로 모아 눈으로 확인하는 코드**를 
작성합니다.

본 실습에서는 과자 상자(`cracker_box`) 10장과 머스터드병(`mustard_bottle`) 10장, 총 20장을 확인합니다.

### 코드의 핵심 흐름

```
main 또는 challenge 선택
        ↓
real_reference 폴더 찾기
        ↓
객체별 실제 촬영 이미지 불러오기
        ↓
이미지 크기를 일정하게 조정
        ↓
사진 아래에 객체 이름과 파일명 표시
        ↓
여러 사진을 하나의 이미지로 배치
        ↓
real_reference_grid.jpg 저장
```


**파일: `scripts/01_preview_real_reference.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import Image, ImageDraw, ImageOps


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day04_common import (
    lesson_objects,
    real_reference_root,
    result_root,
)


TILE_W = 220
TILE_H = 180
LABEL_H = 36


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--asset-set",
        default="main",
        choices=["main", "challenge"],
    )
    return parser.parse_args()


def list_images(directory: Path):
    return sorted(
        path
        for path in directory.iterdir()
        if path.is_file()
        and path.suffix.lower()
        in {".png", ".jpg", ".jpeg"}
    )


def main() -> None:
    args = parse_args()

    objects = lesson_objects(args.asset_set)
    real_root = real_reference_root(
        args.asset_set
    )

    rows = []

    for item in objects:
        class_name = item["class_name"]

        rows.append(
            (
                class_name,
                list_images(
                    real_root / class_name
                ),
            )
        )

    columns = 5

    total_rows = sum(
        (len(files) + columns - 1)
        // columns
        for _, files in rows
    )

    canvas = Image.new(
        "RGB",
        (
            TILE_W * columns,
            total_rows
            * (TILE_H + LABEL_H),
        ),
        "white",
    )

    draw = ImageDraw.Draw(canvas)
    row_offset = 0

    for class_name, files in rows:
        class_rows = (
            len(files) + columns - 1
        ) // columns

        for index, path in enumerate(files):
            row = (
                row_offset
                + index // columns
            )

            column = index % columns

            x = column * TILE_W
            y = row * (
                TILE_H + LABEL_H
            )

            with Image.open(path) as image:
                image = (
                    ImageOps
                    .exif_transpose(image)
                    .convert("RGB")
                )

                image.thumbnail(
                    (
                        TILE_W - 12,
                        TILE_H - 12,
                    ),
                    Image.Resampling.LANCZOS,
                )

                px = (
                    x
                    + (TILE_W - image.width)
                    // 2
                )

                py = (
                    y
                    + (TILE_H - image.height)
                    // 2
                )

                canvas.paste(
                    image,
                    (px, py),
                )

            draw.text(
                (
                    x + 5,
                    y + TILE_H + 7,
                ),
                f"{class_name}: {path.name}",
                fill="black",
            )

        row_offset += class_rows

    output_dir = result_root(
        args.asset_set,
        "source"
    )

    output_path = (
        output_dir
        / "real_reference_grid.jpg"
    )

    canvas.save(
        output_path,
        quality=92,
    )

    print("Saved:", output_path)


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
① main 또는 challenge 선택
           ↓
② 실제 촬영 이미지 폴더 찾기
   real_reference/
           ↓
③ 객체별 이미지 목록 가져오기
   main → 과자 상자 10장 + 머스터드병 10장
   challenge → 전동드릴 10장
           ↓
④ 한 줄에 이미지 5장씩 배치하도록 설정
   TILE_W = 220
   TILE_H = 180
   LABEL_H = 36
           ↓
⑤ 전체 이미지를 담을 흰색 배경 생성
           ↓
⑥ 각 이미지를 순서대로 처리
   ├─ 이미지 열기
   ├─ 촬영 방향 정보(EXIF) 반영
   ├─ RGB 형식으로 변환
   ├─ 비율을 유지하면서 크기 조정
   ├─ 배경의 알맞은 위치에 배치
   └─ 객체 이름과 파일명 표시
           ↓
⑦ 객체별 이미지 배치 완료
           ↓
⑧ 결과 저장 폴더 준비
   results/main/source/
           ↓
⑨ 한 장의 JPG 이미지로 저장
   real_reference_grid.jpg
           ↓
⑩ 저장된 파일 경로 출력
```

### 핵심적으로 이해할 코드

|코드|역할|
|---|---|
|`--asset-set`|Main 또는 Challenge 데이터 선택|
|`TILE_W`, `TILE_H`|사진을 배치할 영역의 가로·세로 크기|
|`LABEL_H`|사진 아래 파일명을 표시할 공간|
|`columns = 5`|한 줄에 사진 5장 배치|
|`image.thumbnail()`|원본 비율을 유지하며 이미지 축소|
|`canvas.paste()`|큰 배경 이미지에 사진 붙이기|
|`draw.text()`|사진 아래 객체 이름과 파일명 표시|
|`canvas.save()`|완성한 이미지를 JPG로 저장|

핵심 정리: 이 코드는 실제 사진을 수정하거나 합성데이터를 생성하는 것이 아니라, 실제 사진 여러 장을 한눈에 비교할 수 있도록 미리보기 이미지로 정리하는 코드입니다. 원본 사진은 그대로 유지됩니다.

실행:

```bash
python scripts/01_preview_real_reference.py \
  --asset-set main
```

실제 사진을 가져온 경로

현재 WSL 환경을 기준으로 한 경로입니다.

① 과자 상자 사진 10장

```
/home/user1/project11/day04_3d_segmentation/
└── data/
    └── day04_student_data/
        └── main/
            └── real_reference/
                └── cracker_box/
                    ├── cracker_box_real_001.png
                    ├── cracker_box_real_002.png
                    ├── ...
                    └── cracker_box_real_010.png
```

② 머스터드병 사진 10장

```
/home/user1/project11/day04_3d_segmentation/
└── data/
    └── day04_student_data/
        └── main/
            └── real_reference/
                └── mustard_bottle/
                    ├── mustard_bottle_real_001.png
                    ├── mustard_bottle_real_002.png
                    ├── ...
                    └── mustard_bottle_real_010.png
```

이 파일에는 실제 촬영 이미지 20장이 한 화면에 정리되어 있습니다.

- 첫 번째와 두 번째 줄: 과자 상자(`cracker_box`) 참고 사진 10장
- 세 번째와 네 번째 줄: 머스터드병(`mustard_bottle`) 참고 사진 10장

### 실행 결과

![[Pasted image 20261014162736.png]]

### 왜 실제 사진 20장을 한 장으로 정리하나요?

이 과정은 새로운 이미지를 생성하는 실습이 아닙니다.

**이후 BlenderProc로 생성할 합성 이미지와 비교하기 위해 실제 촬영 환경을 미리 확인하는 작업**입니다.

사진 20장을 한눈에 살펴보면서 배경, 조명, 촬영 각도, 객체 배치의 특징을 확인합니다.

나중에 합성 이미지를 생성한 뒤 실제 사진과 비교하여 어떤 차이가 있는지 분석합니다. 이를 **Domain Gap(실제 데이터와 합성데이터의 차이)**이라고 합니다.

이 작업은 렌더링에 필수는 아니지만, 합성 이미지의 현실성을 판단하는 기준을 마련하기 위해 진행합니다.

---
# 12. `.ply`와 BOP Object ID를 이해합니다

방금 확인한 11 단락 결과는 실제 카메라로 촬영한 **2D 사진**입니다.

하지만 오늘 합성 이미지를 만들 때는 입체적인 형태를 가진 **3D 모델(.ply)**을 사용합니다.

![[Pasted image 20261014164827.png]]

예를 들어 과자 상자의 실제 사진은 한 방향에서 촬영한 이미지이지만, 3D 모델은 가상 공간에서 회전시켜 여러 방향에서 볼 수 있습니다.

| 3D 모델 파일 | 객체 이름 |
|---|---|
| `obj_000002.ply` | 과자 상자 (`cracker_box`) |
| `obj_000005.ply` | 머스터드병 (`mustard_bottle`) |
| `obj_000015.ply` | 전동드릴 (`power_drill`) |

여기서 BOP Object ID는 **어떤 3D 모델을 불러올지 알려주는 번호**입니다.

예를 들어 BOP ID가 `2`라면 BlenderProc는 `obj_000002.ply` 파일을 불러옵니다.

오늘은 이 3D 모델을 직접 만들거나 수정하지 않습니다.

---
# 13. BlenderProc로 3D 장면을 만드는 원리를 이해합니다

## BlenderProc는 어떤 작업을 하나요?

앞에서 우리는 과자 상자와 머스터드병의 3D 모델(`.ply`)이 준비되어 있다는 것을 확인했습니다.

이제 이 모델을 컴퓨터 안의 가상 공간에 놓고, 카메라와 조명을 설정하여 **실제로 사진을 촬영한 것처럼 새로운 이미지를 만들어 볼 것입니다.**

이러한 작업을 자동으로 수행하도록 도와주는 도구가 **BlenderProc**입니다.

예를 들어 실제 과자 상자를 촬영하려면 상자를 바닥에 놓고, 카메라와 조명을 설치해야 합니다.

BlenderProc에서는 이 과정을 Python 코드로 처리합니다.

## ① 필요한 구성요소는 어디에 있나요?

|구성요소|이미 준비되어 있나요?|오늘 하는 작업|
|---|---|---|
|Object (3D 물체)|**예**|제공된 과자 상자와 머스터드병 `.ply` 파일 불러오기|
|Plane (바닥)|아니요|Python 코드로 가상 바닥 생성|
|Camera (카메라)|아니요|Python 코드로 가상 카메라 위치와 방향 설정|
|Light (조명)|아니요|Python 코드로 가상 조명 생성 및 밝기 설정|

즉, **3D 물체의 모양은 이미 준비되어 있고, 촬영에 필요한 나머지 환경은 우리가 코드로 만드는 것입니다.**

## ② 그러면 이미지는 어떻게 만들어지나요?

![[Pasted image 20261014171011.png]]

```
[미리 제공된 데이터]

obj_000002.ply → 과자 상자
obj_000005.ply → 머스터드병
          ↓
[Python 코드로 장면 만들기]

3D 물체 불러오기
          ↓
바닥(Plane) 생성
          ↓
물체를 바닥 위에 배치
          ↓
가상 카메라 위치 설정
          ↓
가상 조명 위치와 밝기 설정
          ↓
[BlenderProc 렌더링]

카메라에서 보이는 장면을
2D 이미지로 생성
          ↓
RGB 합성 이미지
+
BBox · Mask · Depth 정답 데이터
```

이 과정을 **Rendering(렌더링)**이라고 합니다.

렌더링은 간단히 말해 **컴퓨터 안의 3D 장면을 카메라로 촬영하듯 2D 이미지로 만드는 작업**입니다.

## ③ 그럼 매번 똑같은 이미지만 만들어지나요?

그렇지 않습니다.

처음에는 카메라와 조명을 정해진 위치에 놓고 **합성 이미지 한 장을 생성하여 정상적으로 작동하는지 확인**합니다.

이후에는 Python 코드로 설정한 범위 안에서 물체의 위치와 회전, 카메라 거리, 조명 밝기 등을 자동으로 바꾸며 여러 장의 이미지를 생성합니다.

예를 들어 같은 과자 상자라도 카메라를 옮기거나 조명을 바꾸면 서로 다른 모습의 합성 이미지를 만들 수 있습니다.

## ④ 3일차와 무엇이 다른가요?

3일차에는 **이미 존재하는 2D 배경 사진 위에 객체 PNG를 붙여** 합성 이미지를 만들었습니다.

4일차에는 **3D 모델을 가상 공간에 배치하고 카메라로 촬영하듯** 합성 이미지를 만듭니다.

따라서 3일차는 주로 2D 좌표 `x, y`를 사용했지만, 4일차는 높이까지 표현할 수 있는 3D 좌표 `x, y, z`를 사용합니다.

## ⑤ 오늘은 어떤 카메라를 사용하나요?

제공 데이터에는 원본 YCB-V 촬영에 사용된 카메라 정보인 `camera_uw.json`, `camera_cmu.json`이 포함되어 있습니다.

하지만 오늘은 이 파일들을 직접 사용하지 않습니다.

**Python 코드로 가상 카메라를 새로 설정하고,** `**512 × 512**` **크기의 합성 이미지를 생성합니다.**

이후에는 우리가 만든 합성 이미지와 앞에서 확인한 실제 촬영 사진 20장을 비교합니다.

실제 사진과 합성 이미지의 배경, 조명, 촬영 각도 등이 얼마나 다른지 확인하는 과정을 **Domain Gap 분석**이라고 합니다.

### 핵심 정리

오늘은 3D 모델을 직접 제작하지 않습니다.

**준비된 3D 모델을 불러오고, Python 코드로 바닥·카메라·조명을 설정하여 새로운 합성 이미지와 AI 학습용 정답 데이터를 자동 생성하는 방법을 배웁니다.**

이제 실제 BlenderProc 코드에서 이 과정이 어떻게 구현되는지 하나씩 확인해 보겠습니다.

---

# 14. 3D 장면에서 어떤 정답 데이터가 만들어지나요?

실제 사진을 자동으로 모아 보여주는 것과 달리, BlenderProc는 3D 장면을 렌더링하면서 
이미지와 AI 학습에 사용할 정답 데이터(Ground Truth)를 함께 생성할 수 있습니다.

| 생성 데이터 | 쉽게 이해하면 |
|---|---|
| RGB | 가상 카메라로 촬영한 것처럼 보이는 이미지 |
| COCO BBox | 객체를 감싸는 사각형 위치 `[x, y, width, height]` |
| Semantic Mask | 픽셀마다 어떤 종류의 객체인지 표시 |
| Instance Mask | 객체 하나하나를 구분하여 표시 |
| Depth | 카메라를 기준으로 각 픽셀의 깊이를 기록 |

예를 들어 과자 상자와 머스터드병을 함께 렌더링하면, RGB 이미지뿐 아니라 **각 물체가 어디에 있고 어떤 영역을 차지하는지**도 자동으로 기록할 수 있습니다.

---

# 15. 3D에서는 왜 정답 데이터를 자동으로 만들 수 있나요?

우리가 실제 사진만 보면 컴퓨터는 물체의 정확한 위치나 깊이를 바로 알지 못합니다.

하지만 BlenderProc는 가상 공간에 물체를 직접 배치하기 때문에 처음부터 다음 정보를 알고 있습니다.

```text
3D 모델의 모양
        +
객체의 위치와 회전
        +
카메라의 위치와 방향
        +
객체의 Class ID
        ↓
BlenderProc Rendering
        ↓
RGB 이미지 생성
        +
BBox · Mask · Depth
정답 데이터 생성
```

예를 들어 BlenderProc는 과자 상자가 어느 위치에 있는지, 카메라에서 어떻게 보이는지 알고 있으므로 이미지 속 과자 상자의 영역을 계산할 수 있습니다.

**이것이 3D 합성데이터의 중요한 장점입니다.**

사람이 생성된 이미지를 하나씩 열어 사각형이나 영역을 직접 그리지 않아도, 컴퓨터가 정답 데이터를 함께 만들 수 있습니다.

단, 자동 생성된 데이터라도 실제 환경과 지나치게 다르거나 물체가 비정상적으로 배치되었을 수 있으므로, 이후 QA를 통해 품질을 확인해야 합니다.

---

# PART 3. Main 데이터로 첫 번째 3D Scene을 Rendering합니다

---
# 16. 첫 Rendering에서는 두 Object를 모두 사용합니다

앞에서는 실제 촬영 사진을 확인하고, BlenderProc가 3D 모델을 이용해 합성 이미지를 만드는 원리를 배웠습니다.

이제 **과자 상자와 머스터드병을 가상 공간에 배치하여 첫 번째 합성 이미지 한 장을 직접 생성**해 보겠습니다.

### 어떤 작업을 하나요?

미리 제공된 두 개의 3D 모델을 불러오고, Python 코드로 바닥·카메라·조명을 설정합니다.

![[Pasted image 20261014171445.png]]

```
[준비된 3D 모델]

obj_000002.ply → 과자 상자
obj_000005.ply → 머스터드병
         ↓
[가상 공간 구성]

두 물체를 바닥 위에 배치
         +
카메라 위치와 방향 설정
         +
조명 위치와 밝기 설정
         ↓
[BlenderProc 실행]

첫 번째 장면 렌더링
         ↓
RGB 합성 이미지 1장
+
BBox · Mask · Depth 정답 데이터
```

### 왜 먼저 한 장만 생성하나요?

3D 모델 불러오기부터 이미지 생성, 정답 데이터 저장까지 정상적으로 작동하는지 확인하기 위해서입니다.

이렇게 하면 3D 모델이 정상적으로 불러와지는지, 물체가 이미지에 제대로 나타나는지, BBox·Mask·Depth가 올바르게 생성되는지 확인하기 쉽습니다.

첫 번째 렌더링이 성공하면 이후 실습에서 객체·카메라·조명 조건을 자동으로 변경하여 여러 장의 합성 이미지를 생성합니다.

**핵심:** 이번 실습은 많은 이미지를 만드는 단계가 아니라, **과자 상자와 머스터드병을 이용해 첫 번째 합성 이미지와 정답 데이터를 정상적으로 생성할 수 있는지 확인하는 단계**입니다.

---
# 17. Single Scene Rendering 코드를 작성합니다

BlenderProc Rendering Script에서는 **`import blenderproc as bproc`를 실제 첫 import로 두는 실행 패턴**을 유지합니다.

```text
import blenderproc as bproc
        ↓
나머지 라이브러리 import
        ↓
bproc.init()
        ↓
Scene 구성
```

따라서 일반 Python Script에서 자주 사용하는 `from __future__ import annotations`도 BlenderProc Rendering Script 앞에는 두지 않습니다.

**파일: `scripts/02_render_single_scene.py`**

```python
import blenderproc as bproc

import argparse
import json
import sys
from pathlib import Path

import numpy as np


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day04_common import (
    bop_dataset_path,
    data_config,
    lesson_objects,
    result_root,
)


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--asset-set",
        default="main",
        choices=["main", "challenge"],
    )
    parser.add_argument(
        "--tag",
        default="single",
    )
    parser.add_argument(
        "--seed",
        type=int,
        default=42,
    )
    return parser.parse_args()


def main() -> None:
    args = parse_args()

    np.random.seed(args.seed)
    bproc.init()

    config = data_config(
        args.asset_set
    )

    object_info = {
        int(item["bop_obj_id"]): item
        for item
        in lesson_objects(args.asset_set)
    }

    bop_path = bop_dataset_path(
        args.asset_set
    )

    bop_ids = [
        int(value)
        for value
        in config["bop_obj_ids"]
    ]

    objects = bproc.loader.load_bop_objs(
        bop_dataset_path=str(bop_path),
        obj_ids=bop_ids,
        object_model_unit="mm",
        move_origin_to_x_y_plane=True,
    )

    if len(objects) != len(bop_ids):
        raise RuntimeError(
            "요청한 BOP Object 수와 "
            "불러온 Object 수가 다릅니다."
        )

    for obj in objects:
        bop_id = int(
            obj.get_cp("category_id")
        )

        item = object_info[bop_id]

        obj.set_cp(
            "bop_obj_id",
            bop_id,
        )

        obj.set_cp(
            "category_id",
            int(item["lesson_class_id"]),
        )

        obj.set_cp(
            "supercategory",
            "coco_annotations",
        )

        obj.set_name(
            item["class_name"]
        )

        obj.set_shading_mode("auto")

    if len(objects) == 2:
        x_positions = [-0.13, 0.13]
    else:
        x_positions = [0.0]

    object_rows = []

    for index, obj in enumerate(objects):
        rotation_z = (
            -0.20
            if index == 0
            else 0.20
        )

        if len(objects) == 1:
            rotation_z = 0.15

        obj.set_location(
            [
                x_positions[index],
                0.0,
                0.0,
            ]
        )

        obj.set_rotation_euler(
            [0.0, 0.0, rotation_z]
        )

        object_rows.append(
            {
                "name": obj.get_name(),
                "lesson_class_id": int(
                    obj.get_cp("category_id")
                ),
                "bop_obj_id": int(
                    obj.get_cp("bop_obj_id")
                ),
                "location": (
                    obj.get_location().tolist()
                ),
                "rotation": (
                    obj.get_rotation_euler()
                    .tolist()
                ),
            }
        )

    floor = bproc.object.create_primitive(
        "PLANE",
        scale=[0.55, 0.55, 1.0],
    )

    floor.set_name("floor")
    floor.set_location(
        [0.0, 0.0, -0.001]
    )
    floor.set_cp("category_id", 0)
    floor.set_cp(
        "supercategory",
        "background",
    )

    light = bproc.types.Light()
    light.set_type("POINT")

    light_location = [
        0.25,
        -0.35,
        0.85,
    ]

    light_energy = 500.0

    light.set_location(
        light_location
    )
    light.set_energy(
        light_energy
    )

    camera_location = np.array(
        [0.0, -0.75, 0.35]
    )

    poi = bproc.object.compute_poi(
        objects
    )

    rotation_matrix = (
        bproc.camera
        .rotation_from_forward_vec(
            poi - camera_location
        )
    )

    camera_pose = (
        bproc.math
        .build_transformation_mat(
            camera_location,
            rotation_matrix,
        )
    )

    bproc.camera.add_camera_pose(
        camera_pose
    )

    bproc.camera.set_resolution(
        512,
        512,
    )

    bproc.renderer.enable_depth_output(
        activate_antialiasing=False
    )

    render_data = (
        bproc.renderer.render()
    )

    seg_data = (
        bproc.renderer.render_segmap(
            map_by=[
                "instance",
                "class",
                "name",
            ],
            default_values={
                "category_id": 0,
            },
        )
    )

    render_data.update(seg_data)

    output_dir = result_root(
        args.asset_set,
        args.tag,
    )

    bproc.writer.write_hdf5(
        str(output_dir),
        render_data,
    )

    coco_dir = (
        output_dir
        / "coco_data"
    )

    bproc.writer.write_coco_annotations(
        str(coco_dir),
        instance_segmaps=(
            seg_data["instance_segmaps"]
        ),
        instance_attribute_maps=(
            seg_data[
                "instance_attribute_maps"
            ]
        ),
        colors=render_data["colors"],
        color_file_format="JPEG",
        supercategory="coco_annotations",
        append_to_existing_output=False,
    )

    metadata = {
        "asset_set": args.asset_set,
        "tag": args.tag,
        "seed": args.seed,
        "resolution": [512, 512],
        "objects": object_rows,
        "camera_location": (
            camera_location.tolist()
        ),
        "light_location": light_location,
        "light_energy": light_energy,
        "approved": "pending",
    }

    with (
        output_dir
        / "sample_metadata.json"
    ).open(
        "w",
        encoding="utf-8",
    ) as file:
        json.dump(
            metadata,
            file,
            ensure_ascii=False,
            indent=2,
        )

    print("=" * 64)
    print("Single 3D Scene Completed")
    print("=" * 64)
    print("Asset Set:", args.asset_set)
    print(
        "Objects  :",
        [
            row["name"]
            for row in object_rows
        ],
    )
    print("Output   :", output_dir)


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
① 실행 옵션 확인
   ├─ --asset-set : main 또는 challenge
   ├─ --tag       : 결과 저장 폴더 이름
   └─ --seed      : 난수 생성 기준값
          ↓
② BlenderProc 실행환경 초기화
   bproc.init()
          ↓
③ 3D 모델 불러오기
   main 데이터 선택 시
   ├─ obj_000002.ply → 과자 상자
   └─ obj_000005.ply → 머스터드병
          ↓
④ 객체 이름과 Class ID 설정
   ├─ 과자 상자 → Class ID 1
   └─ 머스터드병 → Class ID 2
          ↓
⑤ 3D 객체를 가상 공간에 배치
   ├─ 과자 상자 → 왼쪽
   └─ 머스터드병 → 오른쪽
   위치와 회전값 설정
          ↓
⑥ 가상 바닥(Plane) 생성
   └─ 두 객체가 놓일 바닥 준비
          ↓
⑦ 가상 조명(Light) 생성
   ├─ 종류: Point Light
   ├─ 위치 설정
   └─ 밝기: 500
          ↓
⑧ 가상 카메라(Camera) 설정
   ├─ 카메라 위치 지정
   ├─ 두 객체를 바라보도록 방향 설정
   └─ 이미지 해상도: 512 × 512
          ↓
⑨ BlenderProc 렌더링
   ├─ RGB 이미지 생성
   └─ Depth 데이터 생성
          ↓
⑩ Segmentation 정답 생성
   ├─ Semantic Mask → 객체 종류 구분
   └─ Instance Mask → 개별 객체 구분
          ↓
⑪ 생성된 결과 저장
   ├─ HDF5 → RGB · Depth · Segmentation
   └─ COCO → RGB 이미지 · BBox 등 Annotation
          ↓
⑫ Metadata 저장
   ├─ 사용한 객체
   ├─ BOP ID / Class ID
   ├─ 객체 위치와 회전
   ├─ 카메라 위치
   ├─ 조명 위치와 밝기
   └─ Seed 및 해상도
          ↓
⑬ 실행 완료 메시지 출력
   Single 3D Scene Completed
```

### 핵심적으로 이해해야 할 코드

|핵심 코드|역할|
|---|---|
|`bproc.init()`|BlenderProc 실행환경 초기화|
|`bproc.loader.load_bop_objs()`|PLY 3D 모델 불러오기|
|`obj.set_location()`|객체 위치 설정|
|`obj.set_rotation_euler()`|객체 회전 설정|
|`bproc.object.create_primitive("PLANE")`|바닥 생성|
|`light.set_energy()`|조명 밝기 설정|
|`bproc.camera.add_camera_pose()`|카메라 위치와 방향 적용|
|`bproc.renderer.render()`|RGB와 Depth 렌더링|
|`bproc.renderer.render_segmap()`|Semantic·Instance Mask 생성|
|`bproc.writer.write_hdf5()`|렌더링 결과 저장|
|`bproc.writer.write_coco_annotations()`|COCO 정답 데이터 저장|

### 핵심 정리

이 코드는 카메라와 조명 조건을 고정한 상태에서 첫 번째 합성 이미지 한 장을 생성하는 실습입니다.

생성된 결과는 다음 폴더에 저장됩니다.

```
results/main/single/
├── 0.hdf5                  ← RGB·Depth·Mask 등
├── sample_metadata.json    ← 생성 조건 기록
└── coco_data/
    ├── coco_annotations.json  ← BBox 등 정답 정보
    └── images/
        └── 000000.jpg         ← 합성 RGB 이미지
```

이 과정이 성공하면 다음 실습에서 객체·카메라·조명 조건을 자동으로 변경하여 여러 장의 합성데이터를 생성합니다.

참고로 이 코드의 `--seed`는 난수 생성 기준값이지만, 현재 Single Render의 객체·카메라·조명 설정은 고정되어 있습니다. `--seed` 값을 변경한다고 이 장면이 자동으로 달라지지는 않습니다.

---
# 18. Main Single Scene을 실행합니다

## 실행전 BlenderProc 2.7.0 실행환경 호환성 설정

BlenderProc 2.7.0에서 YCB-V 3D 모델을 이용한 렌더링을 수행할 때, 설치된 라이브러리의 버전과 내부 데이터 처리 방식 때문에 오류가 발생할 수 있습니다.

본 실습에서 확인한 호환성 문제를 방지하기 위해 다음 세 가지를 설정합니다.

### 1. ImageIO 버전 호환성 설정

BlenderProc 2.7.0의 기본 설정은 `imageio==2.9.0`을 사용합니다.

그러나 본 실습 환경에서 자동 설치된 BOP Toolkit은 `imageio>=2.35.1`을 요구하므로 두 라이브러리 간 버전 충돌이 발생했습니다.

**수정 파일**

```bash
code "$VIRTUAL_ENV/lib/python3.14/site-packages/blenderproc/python/utility/DefaultConfig.py"
```

파일을 열고 `default_pip_packages` 항목에서 ImageIO 버전을 수정합니다.

**수정 전**

`"imageio==2.9.0"`

**수정 후**

`"imageio==2.37.4"`

![[Pasted image 20261015083522.png]]

다른 설정은 변경하지 않고 파일을 저장합니다.

**수정 결과 확인**

```bash
grep -n 'imageio==' "$VIRTUAL_ENV/lib/python3.14/site-packages/blenderproc/python/utility/DefaultConfig.py"
```

출력에서 `imageio==2.37.4`가 확인되어야 합니다.

**Blender 내부 패키지 설치**

```bash
blenderproc pip install 'imageio==2.37.4'
```

이렇게 하면 BlenderProc의 기본 패키지 설정과 실제 설치 버전을 일치시킬 수 있습니다.

### 2. OpenEXR 이미지 읽기 기능 활성화

BlenderProc는 렌더링 과정에서 Depth와 Segmentation 데이터를 EXR 파일로 처리합니다.

현재 실습 환경에서는 OpenCV의 OpenEXR 기능이 비활성화되어 렌더링 후 이미지 파일을 읽는 과정에서 오류가 발생했습니다.

따라서 렌더링을 실행하기 전에 다음 명령어를 입력합니다.

```bash
export OPENCV_IO_ENABLE_OPENEXR=1
```

이 설정은 현재 터미널에 적용됩니다. 새로운 터미널에서 렌더링한다면 다시 실행해야 합니다.

### 3. NumPy 배열 자료형 호환성 수정

BlenderProc 2.7.0의 Segmentation 처리 과정에서는 정수형 배열에 실수 나눗셈을 수행하면서 
자료형 변환 오류가 발생했습니다.

이를 해결하기 위해 배열을 실수형으로 변환한 뒤 계산하도록 수정합니다.

**수정 파일**

```bash
code "$VIRTUAL_ENV/lib/python3.14/site-packages/blenderproc/python/utility/Utility.py"
```

파일을 열고 `map_back_from_equally_spaced_equidistant_values()` 함수에서 다음 부분을 찾습니다.

수정전
```python
# Calc the side length of a block.
block_length = space_size_per_dimension // num_splits_per_dimension

# Subtract a half of a block from all values
values -= block_length // 2

values = np.clip(values, 0, space_size_per_dimension)

# Calculate the block indices per dimension
values /= block_length
```

수정후
```python
# Calc the side length of a block.
block_length = space_size_per_dimension // num_splits_per_dimension

# 정수형 배열을 실수형으로 변환하여 계산 오류 방지
values = values.astype(np.float64, copy=True)

# Subtract a half of a block from all values
values -= block_length // 2

values = np.clip(values, 0, space_size_per_dimension)

# Calculate the block indices per dimension
values /= block_length
```

추가된 코드는 단 한 줄입니다.
```python
values = values.astype(np.float64, copy=True)
```

![[Pasted image 20261015082131.png]]

이 코드는 배열의 자료형을 실수형으로 변경하여 나눗셈 결과를 안전하게 저장하도록 합니다.

### 4. 첫 번째 렌더링 실행

위 설정을 완료하고 원본 PLY 모델과 해당 PNG 텍스처가 준비되었다면 첫 번째 렌더링을 실행합니다.

```bash
blenderproc run \
  scripts/02_render_single_scene.py \
  --asset-set main \
  --tag single \
  --seed 42
```

정상 완료 후 `results/main/single/` 폴더에서 RGB 이미지, HDF5, COCO Annotation, Metadata 파일이 생성되었는지 확인합니다.

**참고:** 이 설정은 현재 실습 환경의 BlenderProc 2.7.0에 적용한 호환성 수정입니다. BlenderProc 재설치 시 라이브러리 내부 파일이 원래 상태로 돌아갈 수 있습니다. 또한 Python 버전이나 설치 경로가 다른 환경에서는 수정 파일의 실제 경로를 먼저 확인해야 합니다.

결과를 확인합니다.

```bash
find results/main/single -maxdepth 3 -type f
```

```
results/main/single/0.hdf5
results/main/single/sample_metadata.json
results/main/single/coco_data/images/000000.jpg
results/main/single/coco_data/coco_annotations.json
```

`0.hdf5`, `coco_annotations.json`, `000000.jpg`, `sample_metadata.json`이 모두 있다면 첫 번째 렌더링의 저장 단계까지 완료된 것입니다.

![[Pasted image 20261015084145.png|218]]

---

#### a. 현업에서 합성데이터를 만들기 위한 준비물

| 준비할 자료                          | 역할                           | 필수 여부                    |
| ------------------------------- | ---------------------------- | ------------------------ |
| 3D Mesh (`FBX`, `OBJ`, `PLY` 등) | 제품의 입체적인 형태                  | 필수                       |
| UV 좌표                           | 2D 텍스처를 3D 표면의 어느 위치에 입힐지 지정 | 이미지 텍스처를 쓴다면 필요          |
| Texture (`PNG`, `JPG` 등)        | 제품 로고, 포장 디자인, 색상, 표면 무늬     | 실제 외관을 재현하려면 대체로 필요      |
| Material                        | 표면의 금속성, 거칠기, 반사, 투명도 등      | 사실적인 렌더링에 중요             |
| 객체의 분류 정보                       | 제품 이름, Class ID 등            | 정답 라벨 자동 생성에 필요          |
| 모델의 크기·단위·좌표 기준                 | 실제 크기와 위치를 맞추기 위한 정보         | 정확한 장면 구성에 필요            |
| 카메라·조명·배경 설정                    | 합성 이미지를 촬영할 가상 환경            | 렌더링에 필요                  |
| JSON 파일                         | 위 정보의 일부를 저장하거나 생성 결과를 기록    | 특정 JSON 파일 자체가 항상 필수는 아님 |
예를 들어 자사 제품이 과자 상자라면 다음과 같은 과정입니다.

```
[현업에서 준비]

제품의 3D 모델 제작 또는 3D 스캔
               ↓
메시 형태와 실제 크기 확인
               ↓
UV 언랩 및 텍스처 제작
(이미 완성된 모델이라면 생략)
               ↓
재질 설정 및 제품 외관 확인
               ↓
제품 이름·Class ID 정의
               ↓

[BlenderProc로 자동화]

모델과 재질 불러오기
               ↓
객체·카메라·조명·배경 배치
               ↓
조건을 바꿔가며 반복 렌더링
               ↓

[새롭게 생성되는 데이터] ]

RGB 합성 이미지
BBox · Mask · Depth
COCO JSON 등 정답 데이터
```

여기서 중요한 점은 UV 언랩된 PNG 자체가 합성데이터셋은 아니라는 것입니다. 
PNG는 합성데이터를 만들기 위한 입력 자료이고, BlenderProc가 만든 RGB·Mask·Depth 등이 생성 결과입니다.

#### b. 그런데 왜 다운로드한 YCB-V에는 PLY·PNG·JSON이 함께 있나요?

현재 강사가 제공한 다운로드한 것은 단순한 3D 모델 모음이 아니라 BOP이라는 데이터셋 규격에 맞춰 정리된 3D 모델 패키지이기 때문입니다.

현재 폴더를 보면

```
ycbv_models/
└── models/
    ├── models_info.json     ← 3D 모델의 크기 등 정보
    │
    ├── obj_000002.ply       ← 과자 상자 3D 모델
    ├── obj_000002.png       ← 과자 상자 텍스처
    │
    ├── obj_000005.ply       ← 머스터드병 3D 모델
    ├── obj_000005.png       ← 머스터드병 텍스처
    │
    ├── obj_000015.ply       ← 전동드릴 3D 모델
    └── obj_000015.png       ← 전동드릴 텍스처
```

여기서 `models_info.json`은 BOP 형식에서 사용하는 각 3D 모델의 크기, 3D 경계 상자, 직경 등을 기록한 메타데이터입니다.

그리고 지금 작성된 수업 코드는 다음 함수를 사용합니다.

```python
objects = bproc.loader.load_bop_objs(
    bop_dataset_path=str(bop_path),
    obj_ids=bop_ids,
    object_model_unit="mm",
    move_origin_to_x_y_plane=True,
)
```

이 함수는 일반 FBX 파일을 불러오는 함수가 아니라 BOP 규격의 모델을 불러오는 전용 함수입니다.

따라서 현업에서는 모델을 BOP 규격으로 준비해야 합니다.

#### c. 현업에서는 어떻게 데이터 준비하면 될까요?

예를 들어 공장에서 생산하는 두 제품을 검사하기 위한 합성데이터를 만든다고 가정해 보겠습니다.

![[Pasted image 20261015090321.png]]

두 사례 모두 3D 객체를 준비한 후에는 동일하게 BlenderProc에서 카메라·조명·객체 위치를 변경하며 RGB·BBox·Mask·Depth를 생성할 수 있습니다.

또한 BlenderProc 2.7.0은 BOP 전용 로더 외에도 일반 OBJ·PLY, Blender `.blend` 파일을 불러오는 방법을 제공합니다. FBX를 사용하는 경우에는 Blender에서 적절한 형식으로 변환하거나 FBX 가져오기 절차를 별도로 구현해야 합니다

---

# 19. RGB 이미지와 COCO BBox 정답 데이터를 확인합니다

앞에서는 BlenderProc를 이용하여 과자 상자와 머스터드병이 포함된 첫 번째 RGB 합성 이미지를 생성했습니다.

이번에는 **이미지에 있는 객체의 종류와 위치가 COCO JSON 파일에 올바르게 저장되었는지** 확인하겠습니다.

### ① COCO JSON 파일을 확인합니다

다음 명령어를 실행하세요.

```bash
python -m json.tool \
  results/main/single/coco_data/coco_annotations.json \
  | head -100
```

이 명령어는 저장된 JSON 파일을 읽기 쉽게 정렬하여 처음 100줄을 출력합니다. 새로운 데이터를 생성하는 코드는 아닙니다.

**실제 실행 결과**

```json
{
    "categories": [],
    "images": [
        {
            "id": 0,
            "file_name": "images/000000.jpg",
            "width": 512,
            "height": 512
        }
    ],
    "annotations": []
}
```

위 코드는 실제 출력에서 핵심 부분만 정리한 것입니다.

각 항목의 의미를 확인해 보겠습니다.

|항목|역할|현재 결과|
|---|---|---|
|`images`|생성된 이미지의 파일명과 크기|이미지 1장, 512×512 기록|
|`categories`|객체의 종류와 Class ID|비어 있음|
|`annotations`|객체의 위치(BBox)와 Segmentation 정답|비어 있음|

현재 결과를 보면 **RGB 이미지 정보는 정상적으로 기록되었지만, 객체의 종류와 위치에 대한 정답 정보는 아직 저장되지 않았습니다.**

이번 Main 실습에서 사용한 객체는 다음 두 개입니다.

```
Class ID 1 → cracker_box (과자 상자)
Class ID 2 → mustard_bottle (머스터드병)
```

따라서 정상적으로 정답 데이터가 생성되었다면 `categories`에 두 객체의 정보가 들어 있어야 하며, `annotations`에도 이미지에서 검출 가능한 객체들의 위치 정보가 저장되어야 합니다.

현재처럼 두 항목이 모두 `[]`라면 **COCO 정답 데이터 생성이 정상적으로 완료되지 않은 상태**입니다.

### ② BBox는 무엇인가요?

BBox(Bounding Box)는 이미지에서 물체가 차지하는 영역을 둘러싸는 사각형입니다.

COCO에서는 다음 네 개의 값으로 위치를 표현합니다.

```
[x, y, width, height]
```

|값|의미|
|---|---|
|`x`|사각형의 왼쪽 시작 위치|
|`y`|사각형의 위쪽 시작 위치|
|`width`|사각형의 가로 길이|
|`height`|사각형의 세로 길이|

예를 들어 정상적으로 생성된 COCO Annotation에 다음 정보가 있다고 가정하겠습니다.

```json
{
    "category_id": 1,
    "bbox": [120, 80, 150, 220]
}
```

이는 Class ID가 1인 과자 상자의 BBox가 이미지의 왼쪽에서 120픽셀, 위쪽에서 80픽셀인 지점부터 시작하여 가로 150픽셀, 세로 220픽셀의 크기를 가진다는 의미입니다.

**이 값은 설명을 위한 예시이며, 현재 생성된 JSON에 실제로 저장된 내용은 아닙니다.**

### ③ 현재 COCO 정답 데이터가 정상적인지 확인합니다

다음 명령어를 이용하면 이미지, 객체 종류, Annotation 개수를 간단하게 확인할 수 있습니다.

```bash
python - <<'PY'
import json

path = "results/main/single/coco_data/coco_annotations.json"

with open(path, encoding="utf-8") as f:
    data = json.load(f)

print("이미지 수:", len(data.get("images", [])))
print("객체 종류 수:", len(data.get("categories", [])))
print("Annotation 수:", len(data.get("annotations", [])))
PY
```

현재 제공된 JSON을 기준으로 예상되는 출력은 다음과 같습니다.

```
이미지 수: 1
객체 종류 수: 0
Annotation 수: 0
```

이 결과는 이미지가 한 장 생성되었지만, 객체 정답 정보는 아직 없다는 뜻입니다.

따라서 다음 실습으로 넘어가기 전에 BlenderProc의 Segmentation 결과와 Class ID가 COCO Annotation으로 올바르게 전달되는지 확인해야 합니다.

### ④ 왜 COCO BBox 정답을 확인하나요?

객체 탐지 AI 모델을 학습하려면 단순한 이미지뿐 아니라 이미지 속에 어떤 객체가 어디에 있는지를 알려주는 정답 데이터가 필요합니다.

BlenderProc는 3D 객체를 이용해 이러한 정답을 자동으로 생성할 수 있으므로, 사람이 이미지마다 사각형을 직접 그리는 작업을 줄일 수 있습니다.

하지만 정답 생성 기능을 실행했다고 해서 모든 데이터가 반드시 올바르게 저장되는 것은 아닙니다.

따라서 생성된 JSON에 실제 객체 정보가 들어 있는지 확인하는 과정이 필요합니다.

### 핵심 정리

이번 실습에서는 RGB 이미지와 COCO JSON 파일의 관계를 확인했습니다.

- **RGB 이미지:** 과자 상자와 머스터드병이 보이는 합성 이미지
- **COCO JSON:** 이미지에 포함된 객체의 종류, BBox, Segmentation 등 정답을 기록하는 파일
- **현재 결과:** RGB 이미지 정보는 있지만 `categories`와 `annotations`는 비어 있음
    

**따라서 현재 COCO 파일은 객체 탐지 학습용 정답 데이터로 사용할 수 없습니다.**

먼저 객체의 Class ID와 Segmentation 정보가 올바르게 생성되고 COCO JSON에 저장되도록 수정한 다음, 정상적으로 BBox가 기록되는지 다시 확인하겠습니다.

---
# 20. HDF5 파일에 저장된 렌더링 결과를 확인합니다

앞에서는 BlenderProc가 생성한 RGB 이미지와 COCO BBox 정답 데이터를 확인했습니다.

이번에는 하나의 HDF5 파일에 저장된 여러 종류의 렌더링 결과를 확인하겠습니다.

HDF5는 RGB 이미지, Depth(깊이 정보), Segmentation Mask 등 서로 다른 데이터를 하나의 파일에 저장할 수 있는 형식입니다.

이번 실습에서는 다음 파일을 확인합니다.

```txt
results/main/single/0.hdf5
```

다음 명령어를 실행하세요.
그러면 BlenderProc에서 제공하는 Viewer를 이용하면 파일에 저장된 데이터를 시각적으로 살펴볼 수 있습니다.

```bash
blenderproc vis hdf5 \
  results/main/single/0.hdf5
```

![[Pasted image 20261015091814.png]]

| 결과                 | 확인할 내용                     |
| ------------------ | -------------------------- |
| `class_segmaps`    | 픽셀별 객체 종류(Class ID)        |
| `colors`           | 과자 상자와 머스터드병이 렌더링된 RGB 이미지 |
| `depth`            | 카메라에서 물체 표면까지의 깊이 정보       |
| `instance_segmaps` | 개별 객체를 구분하는 Instance ID    |

**현재 결과를 살펴보겠습니다.**

- **RGB:** 두 물체가 정상적으로 렌더링되었습니다.
- **Depth:** 물체와 바닥의 깊이 차이가 색상으로 표현되었습니다. 녹색 계열로 보이는 것은 Viewer의 색상 표현 방식이고 다른 색상 계열로도 표현 가능합니다.
- **Class / Instance Mask:** 두 이미지가 모두 단색으로 나타났습니다. 객체 ID가 제대로 저장되었는지 추가 확인이 필요합니다.

특히 앞에서 COCO JSON의 `categories`와 `annotations`가 비어 있었으므로, Segmentation 데이터에 실제 객체 ID가 저장되었는지 검사해야 합니다.

---
현재 실행 중인 BlenderProc HDF5 Viewer를 종료하려면

1. 화면에 열린 Figure 1, Figure 2, Figure 3, Figure 4 창의 오른쪽 위 `X` 버튼을 각각 눌러 닫기
2. 모든 창을 닫으면 VS Code 터미널로 돌아옵니다.
3. 터미널에 입력할 수 있는 상태가 되면 정상적으로 종료된 것입니다.
    
만약 창을 닫아도 터미널이 계속 실행 중이라면 **터미널에서 `Ctrl + C`**를 눌러 종료하세요.

---
현재 RGB와 Depth는 생성되었지만, Class Mask와 Instance Mask가 모두 단색으로 보이고, 앞서 확인한 COCO JSON의 `annotations`도 비어 있으므로 먼저 Mask데이터를 확인합니다.

```bash
python - <<'PY'
import h5py
import numpy as np

with h5py.File("results/main/single/0.hdf5", "r") as f:
    for key in ["class_segmaps", "instance_segmaps"]:
        ids, counts = np.unique(
            f[key][:], return_counts=True
        )
        print(key, dict(zip(ids.tolist(), counts.tolist())))
PY
```

실행 결과

```
class_segmaps {0: 262144}
instance_segmaps {0: 262144}
```

검사 결과, Class Mask와 Instance Mask가 모두 정상적으로 생성되지 않았습니다.

### Segmentation Mask 검사 결과

```
class_segmaps:    {0: 262144}
instance_segmaps: {0: 262144}
```

현재 이미지의 크기는 512×512이므로 전체 픽셀은 262,144개입니다.

검사 결과, **모든 픽셀이 0(배경)으로 기록**되어 있습니다.

정상이라면 다음과 같이 객체별 ID가 구분되어야 합니다.

```
Class Mask
0 → 배경
1 → 과자 상자
2 → 머스터드병

Instance Mask
0 → 배경
1, 2 등 → 개별 객체
```

현재 RGB 이미지에는 두 객체가 보이지만, Segmentation 데이터에서는 객체를 구분하지 못하고 있습니다.

확인된 문제는 RGB 이미지에는 두 물체가 보이지만, Segmentation Mask에는 모든 픽셀이 0으로 저장된 것입니다.

BlenderProc 2.7.0 코드를 확인하면 Segmentation EXR 파일을 읽은 뒤 픽셀값을 객체 ID로 변환합니다. 따라서 EXR을 읽는 과정에서 값이 잘못 변환되었는지 먼저 확인하고 수정하겠습니다

## 1. Segmentation 코드 수정하기

VS Code에서 다음 파일을 엽니다.

```bash
code "$VIRTUAL_ENV/lib/python3.14/site-packages/blenderproc/python/renderer/SegMapRendererUtility.py"
```

파일 상단의 `import numpy as np` 근처에 다음 코드를 추가합니다.

```
import cv2
```

![[Pasted image 20261015094133.png|319]]

그다음 `Ctrl + F`로 아래 코드를 찾습니다.

수정 전

```python
segmentation = load_image(file_path)
print(file_path, segmentation.shape)
```

![[Pasted image 20261015094311.png|488]]



수정 후

```python
# EXR 원본의 픽셀값을 그대로 읽습니다.
segmentation = cv2.imread(
    file_path,
    cv2.IMREAD_UNCHANGED
)

if segmentation is None:
    raise RuntimeError("Segmentation EXR 읽기 실패")

# OpenCV의 BGR 순서를 RGB로 변경
segmentation = segmentation[:, :, :3][:, :, ::-1].copy()

print(
    "Segmentation EXR:",
    segmentation.dtype,
    "최소:", np.min(segmentation),
    "최대:", np.max(segmentation)
)
```

![[Pasted image 20261015094355.png|471]]

이렇게 수정하는 이유는 기존 ImageIO 대신 OpenCV로 EXR 원본의 자료형과 픽셀값을 유지해서 읽어보기 위해서입니다.

이번 수정은 확인된 원인을 바탕으로 한 해결 시도이며, 아직 최종 해결 여부가 검증된 것은 아닙니다. 기존 파일을 백업한 뒤 저장하세요.

## 2. 렌더링 다시 실행하기

```bash
OPENCV_IO_ENABLE_OPENEXR=1 blenderproc run \
  scripts/02_render_single_scene.py \
  --asset-set main \
  --tag single \
  --seed 42
```

실행 로그에서 `Segmentation EXR`의 자료형과 최솟값·최댓값을 확인합니다.

## 3. Mask가 정상적으로 생성되었는지 확인하기

렌더링이 완료되면 다음 명령어를 실행하세요.

```bash
python - <<'PY'
import h5py
import numpy as np

with h5py.File("results/main/single/0.hdf5", "r") as f:
    for key in ["class_segmaps", "instance_segmaps"]:
        print(key, np.unique(f[key][:]))
PY
```

실행 결과는 다음과 같습니다.

```
class_segmaps [0 1 2]
instance_segmaps [0 1 2 3]  
```

### 결과 해석

#### `class_segmaps [0 1 2]`

이 값은 Class Mask가 정상적으로 생성되었다는 뜻입니다.

```
0 → 배경 / 바닥
1 → cracker_box
2 → mustard_bottle
```

즉, 이제 픽셀 단위로  
어디가 배경이고, 어디가 과자 상자이며, 어디가 머스터드병인지 구분할 수 있습니다.

다시 확인해봅니다.

```bash
blenderproc vis hdf5 \
  results/main/single/0.hdf5
```

![[Pasted image 20261015112220.png]]

처음에는 Mask가 모두 배경으로만 나왔지만, 수정 후에는 객체별 영역이 정상적으로 분리되어 렌더링 결과가 올바르게 생성되었습니다.

현재 실행 중인 BlenderProc HDF5 Viewer를 종료하려면

1. 화면에 열린 Figure 1, Figure 2, Figure 3, Figure 4 창의 오른쪽 위 `X` 버튼을 각각 눌러 닫기
2. 모든 창을 닫으면 VS Code 터미널로 돌아옵니다.
3. 터미널에 입력할 수 있는 상태가 되면 정상적으로 종료된 것입니다.
    
만약 창을 닫아도 터미널이 계속 실행 중이라면 **터미널에서 `Ctrl + C`**를 눌러 종료하세요.

---
# 21. HDF5 결과를 PNG Preview로 저장합니다

20번에서는 BlenderProc Viewer를 이용하여 HDF5에 저장된 RGB·Depth·Segmentation 결과를 
확인했습니다.

이번에는 **HDF5 파일에 들어 있는 데이터를 각각 PNG 이미지로 저장하는 Python 프로그램**을 
작성하겠습니다.

### 이 코드는 무엇을 하나요?

HDF5는 여러 종류의 데이터를 하나의 파일에 저장할 수 있지만, 
일반 이미지 뷰어에서는 내용을 바로 확인하기 어렵습니다.

따라서 Python으로 HDF5 파일을 읽고, RGB·Depth·Mask를 각각 이미지로 변환합니다.

### 이 코드는 왜 작성하나요?

HDF5의 RGB·Depth·Semantic·Instance를 PNG로 남겨 비교하기 위해 작성합니다.

**파일: `scripts/03_export_hdf5_preview.py`**

```python
from __future__ import annotations

import argparse
from pathlib import Path

import h5py
import matplotlib.pyplot as plt
import numpy as np
from PIL import Image


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--input",
        required=True,
    )
    parser.add_argument(
        "--output",
        required=True,
    )
    return parser.parse_args()


def find_key(keys, words):
    for key in keys:
        lower = key.lower()

        if all(
            word in lower
            for word in words
        ):
            return key

    return None


def main() -> None:
    args = parse_args()

    input_path = Path(args.input)
    output_dir = Path(args.output)

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    if not input_path.exists():
        raise FileNotFoundError(input_path)

    with h5py.File(
        input_path,
        "r",
    ) as file:
        keys = list(file.keys())

        print("HDF5 Keys")
        for key in keys:
            print(" -", key)

        if "colors" in file:
            rgb = np.array(
                file["colors"]
            )

            if (
                rgb.ndim == 3
                and rgb.shape[-1] >= 3
            ):
                rgb = rgb[..., :3]

            if rgb.dtype != np.uint8:
                if rgb.max() <= 1.0:
                    rgb = rgb * 255.0

                rgb = np.clip(
                    rgb,
                    0,
                    255,
                ).astype(np.uint8)

            Image.fromarray(rgb).save(
                output_dir
                / "rgb.png"
            )

        depth_key = None

        for candidate in [
            "depth",
            "distance",
        ]:
            if candidate in file:
                depth_key = candidate
                break

        if depth_key:
            depth = np.array(
                file[depth_key]
            ).astype(np.float32)

            valid = (
                np.isfinite(depth)
                & (depth > 0)
            )

            normalized = np.zeros_like(
                depth,
                dtype=np.float32,
            )

            if valid.any():
                values = depth[valid]
                low = np.percentile(
                    values,
                    2,
                )
                high = np.percentile(
                    values,
                    98,
                )

                if high > low:
                    normalized[valid] = (
                        np.clip(
                            (
                                depth[valid]
                                - low
                            )
                            / (
                                high - low
                            ),
                            0,
                            1,
                        )
                    )

            plt.imsave(
                output_dir
                / "depth.png",
                normalized,
                cmap="gray",
            )

        class_key = find_key(
            keys,
            ["class", "seg"],
        )

        if class_key:
            class_mask = np.array(file[class_key])

            print("Class IDs:", np.unique(class_mask))

            if np.unique(class_mask).size < 2:
                raise ValueError(
                    "Class Mask에 객체 ID가 없습니다."
                )

            plt.imsave(
                output_dir / "semantic_mask.png",
                class_mask,
                cmap="tab10",
                vmin=0,
                vmax=9,
            )

        instance_key = find_key(
            keys,
            ["instance", "seg"],
        )

        if instance_key:
            instance_mask = np.array(file[instance_key])

            print("Instance IDs:", np.unique(instance_mask))

            if np.unique(instance_mask).size < 2:
                raise ValueError(
                    "Instance Mask에 객체 영역이 없습니다."
                )

            plt.imsave(
                output_dir / "instance_mask.png",
                instance_mask,
                cmap="tab10",
                vmin=0,
                vmax=9,
            )

    print(
        "Preview Saved:",
        output_dir,
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[20번에서 확인한 HDF5 파일]

results/main/single/0.hdf5
            ↓
Python으로 HDF5 데이터 읽기
            ↓
각 데이터 종류 확인
            ↓
PNG 이미지로 변환
            ↓

[21번에서 생성할 결과]

rgb.png
→ 렌더링된 컬러 이미지

depth.png
→ 깊이 차이를 회색조로 표현

semantic_mask.png
→ 객체 종류별 영역을 색상으로 구분

instance_mask.png
→ 개별 객체 영역을 색상으로 구분
```

실행:

```bash
python scripts/03_export_hdf5_preview.py \
  --input results/main/single/0.hdf5 \
  --output results/main/single/previews
```

결과해석

![[Pasted image 20261015114202.png]]

| 파일                  | 실행 결과                   | 판단    |
| ------------------- | ----------------------- | ----- |
| `rgb.png`           | 과자 상자와 머스터드병 표시         | 정상    |
| `semantic_mask.png` | 객체 종류별 색상 구분            | 정상    |
| `instance_mask.png` | 개별 객체 영역 구분             | 정상    |
| `depth.png`         | 전체가 검게 표시               | 확인 필요 |
| `0.hdf5`            | VS Code 텍스트 편집기로 열리지 않음 | 정상    |

특히 터미널에 다음 결과가 나왔습니다.

```
Class IDs: [0 1 2]
Instance IDs: [0 1 2 3]
Preview Saved: results/main/single/previews
```

이는 Segmentation 데이터에 객체 ID가 정상적으로 기록되었고 Preview 저장 코드도 끝까지 실행되었다는 의미입니다.

### Depth 이미지는 왜 검게 나타날까요?

현재 `03_export_hdf5_preview.py`는 Depth의 깊이값을 0~1 사이로 정규화한 뒤 흑백 이미지로 저장합니다.

```
HDF5의 실제 Depth 값
         ↓
깊이값의 2%·98% 백분위수 계산
         ↓
0~1 범위로 정규화
         ↓
회색조 PNG 저장
```

그런데 깊이값의 차이가 작거나, 대부분의 픽셀이 동일한 값을 가진 경우 정규화 결과가 전부 0에 가까워져 검게 표시될 수 있습니다.

이것만으로 원본 Depth 데이터가 잘못되었다고 판단할 수는 없습니다.
원인을 찾기 위해 원본 Depth 값을 확인해 봅니다.

```bash
python - <<'PY'
import h5py
import numpy as np

with h5py.File("results/main/single/0.hdf5", "r") as f:
    depth = f["depth"][:]

    valid = depth[np.isfinite(depth) & (depth > 0)]

    print("Depth 전체 크기:", depth.shape)
    print("유효 픽셀 수:", len(valid))

    if len(valid):
        print("최솟값:", valid.min())
        print("최댓값:", valid.max())
        print("2% 값:", np.percentile(valid, 2))
        print("98% 값:", np.percentile(valid, 98))
PY
```

터미널 결과

```
Depth 전체 크기: (512, 512)
유효 픽셀 수: 153720
최솟값: 1
최댓값: 1
2% 값: 1.0
98% 값: 1.0
```

문제는 PNG 저장 코드만의 문제가 아니라, 원본 HDF5에 저장된 Depth 값 자체가 모두 1로 기록되어 있다는 것입니다.

| 검사 항목      | 결과        | 의미             |
| ---------- | --------- | -------------- |
| Depth 크기   | 512 × 512 | 정상적인 이미지 크기    |
| 전체 픽셀      | 262,144개  | 이미지 전체         |
| 유효 픽셀      | 153,720개  | 0보다 크고 유한한 깊이값 |
| 최솟값        | ==1==     | 가장 가까운 유효 깊이값  |
| 최댓값        | ==1==     | 가장 먼 유효 깊이값    |
| 2% / 98% 값 | ==1 / 1== | 깊이값의 변화가 없음    |
현재 코드는 깊이값을 0~1 범위로 정규화하여 흑백 이미지로 저장합니다.

그런데 실제 데이터가 다음과 같습니다.

```
Depth 값

1, 1, 1, 1, 1, 1 ...
        ↓
최솟값 = 최댓값 = 1
        ↓
깊이 차이가 없음
        ↓
정규화할 범위가 없음
        ↓
모든 픽셀을 0으로 유지
        ↓
검은색 Depth Preview 생성
```

원본 Depth 데이터를 생성하고 저장하는 `02_render_single_scene.py`의 처리 과정을 먼저 점검해야 합니다.

우선 다음 명령어로 Depth의 자료형과 실제 값의 분포를 확인합니다.

```bash
python - <<'PY'
import h5py
import numpy as np

with h5py.File("results/main/single/0.hdf5", "r") as f:
    depth = f["depth"][:]

    print("자료형:", depth.dtype)
    print("고유값:", np.unique(depth)[:20])
    print("0인 픽셀:", np.count_nonzero(depth == 0))
    print("NaN 픽셀:", np.count_nonzero(np.isnan(depth)))
    print("무한대 픽셀:", np.count_nonzero(np.isinf(depth)))
PY
```

터미널 출력결과

현재 문제는 간단합니다. Depth(깊이) 데이터가 잘못 저장되어 이미지가 검게 나온 것입니다.

```
자료형: uint8
고유값: [0 1]
0인 픽셀: 108424
NaN 픽셀: 0
무한대 픽셀: 0
```

위에 보면 고유값: [0 1] 즉 깊이값이 0과 1로 되어 있습니다.

Depth는 카메라와 물체 사이의 거리를 기록하는 데이터입니다.

예를 들어 정상이라면 깊이값이 `0.65`, `0.82`, `1.15`처럼 서로 달라야 합니다.

그런데 현재는 0과 1만 저장되어 있어 물체 사이의 거리 차이를 제대로 표현하지 못합니다.

원인은 BlenderProc가 렌더링한 깊이 이미지(EXR)를 다시 읽으면서 숫자의 정밀도가 손실된 것으로 의심됩니다.

### 어떻게 해결하나요?

#### ① BlenderProc 내부 파일을 엽니다

터미널에서 실행하세요.

```bash
code "$VIRTUAL_ENV/lib/python3.14/site-packages/blenderproc/python/utility/BlenderUtility.py"
```

#### ② `load_image()` 함수의 EXR 읽기 부분을 수정합니다

`Ctrl + F`로 다음 코드를 찾으세요.

```python
file_ending = file_path[file_path.rfind(".") + 1:].lower()
```

그 바로 아래에 추가합니다.

```python
# 깊이값을 실수형 그대로 읽기
if file_ending == "exr":
    image = cv2.imread(
        file_path,
        cv2.IMREAD_UNCHANGED
    )

    if image is None:
        raise RuntimeError("EXR 읽기 실패")

    if image.ndim == 2:
        image = image[:, :, None]

    image = image[:, :, ::-1].copy()

    return image[:, :, :num_channels]
```

![[Pasted image 20261015120406.png]]

파일 상단에 `import cv2`가 없다면 추가하고 `Ctrl + S`로 저장합니다. 기존 코드의 다른 부분은 변경하지 마세요.

이 수정은 EXR의 깊이값이 정수로 바뀌지 않도록 원본 자료형으로 읽기 위한 것입니다.

#### 다시 렌더링합니다

기존 결과를 보존하기 위해 새로운 폴더에 생성합니다.

```bash
OPENCV_IO_ENABLE_OPENEXR=1 blenderproc run \
  scripts/02_render_single_scene.py \
  --asset-set main \
  --tag depth_fixed \
  --seed 42
```

#### 깊이값이 정상인지 확인합니다

```bash
python - <<'PY'
import h5py
import numpy as np

with h5py.File("results/main/depth_fixed/0.hdf5", "r") as f:
    d = f["depth"][:]
    print("자료형:", d.dtype)
    print("서로 다른 깊이값 수:", len(np.unique(d)))
PY
```

터미널 출력 결과

```
자료형: float32
서로 다른 깊이값 수: 31336
```

이 결과는  정상적인 Depth 데이터로 바뀐 상태입니다.

- **`float32`**  
    → Depth가 이제 정수형(`uint8`)이 아니라 **실수형 거리값**으로 저장되었다는 뜻입니다.  
    → 즉, 깊이 정보가 훨씬 정상적으로 저장되고 있습니다.
    
- **서로 다른 깊이값 수: `31336`**  
    → 깊이값이 0과 1 두 개만 있는 것이 아니라, **아주 다양한 거리값**이 들어 있다는 뜻입니다.  
    → 과자 상자, 머스터드병, 바닥, 배경의 거리 차이가 반영되었다고 볼 수 있습니다.
    

#### Depth 이미지가 실제로 바뀌었는지 확인합니다.

Preview 다시 만들기

```bash
python scripts/03_export_hdf5_preview.py \
  --input results/main/depth_fixed/0.hdf5 \
  --output results/main/depth_fixed/previews
```

터미널 결과

```
Class IDs: [0 1 2]
Instance IDs: [0 1 2 3]
Preview Saved: results/main/depth_fixed/previews
```

어떤 파일이 만들어졌는지 보기

```bash
ls results/main/depth_fixed/previews
```

![[Pasted image 20261015120847.png]]

이번 문제는 데이터 자체의 문제가 아니라 BlenderProc가 Depth 파일을 읽는 방식의 문제였습니다.
아주 쉽게 정리하면
BlenderProc 내부에서 EXR(깊이 파일)을 읽는 기본 방식이 현재 환경에서 우리 실습 결과와 잘 맞지 않았던 것입니다. 이런경우 위와 같이 설정을 수정하면 해결이 됩니다. 중요한건 결과가 나오면 반드시 확인하여 지금과 같이 사물이 정확히 인지 되는지 확인하는 것이 중요합니다.

---
# 22. 첫 번째 3D Rendering 실습을 마무리합니다

지금까지 과자 상자와 머스터드병의 3D 모델을 불러와 첫 번째 합성 이미지를 생성하고, HDF5에 저장된 결과를 확인했습니다.

실습 과정에서 발생한 환경 설정과 데이터 처리 문제도 수정하여 다음 결과를 확인했습니다.

|생성 데이터|최종 확인 결과|
|---|---|
|RGB|두 객체의 합성 이미지 생성|
|Semantic Mask|Class ID `[0, 1, 2]` 확인|
|Instance Mask|Instance ID `[0, 1, 2, 3]` 확인|
|Depth|실수형(`float32`) 깊이 데이터 생성|
|PNG Preview|RGB·Depth·Mask 이미지 저장|

이 과정에서 **BlenderProc는 단순히 3D 이미지만 만드는 것이 아니라, 객체의 종류와 영역, 깊이 정보도 함께 생성할 수 있다는 것**을 확인했습니다.

단, COCO BBox는 앞서 Annotation이 비어 있었으므로, 최종 렌더링 결과의 JSON에 객체 정보가 정상적으로 기록되었는지 확인해야 합니다.


---
# PART 4. Object·Camera·Light를 바꾸어 여러 Synthetic Scene을 생성합니다

---

# 23. 다양한 합성데이터를 생성하기 위해 Randomization을 이해합니다

지금까지 과자 상자와 머스터드병의 3D 모델을 불러와 첫 번째 합성 이미지를 만들었습니다.

이전 실습에서는 객체·카메라·조명 위치를 고정했기 때문에 같은 조건으로 다시 실행하면 거의 동일한 이미지가 만들어집니다.

하지만 AI 모델을 학습시키려면 물체를 다양한 위치와 방향, 촬영 환경에서 보여주는 이미지가 필요합니다.

예를 들어 같은 과자 상자라도 카메라가 가까이 있을 때와 멀리 있을 때, 조명이 밝을 때와 어두울 때의 모습이 달라집니다.

**이처럼 정해진 범위 안에서 촬영 조건을 무작위로 변경하는 방법을 Randomization(무작위화)이라고 합니다.**

### 이번에는 무엇을 변경하나요?

| 변경 대상 | 변경할 내용 | 이미지에 미치는 영향 |
|---|---|---|
| Object | X·Y 위치 | 물체가 놓이는 위치가 달라짐 |
| Object | Z축 회전 | 물체가 바라보는 방향이 달라짐 |
| Camera | 좌우 위치·높이·거리 | 물체가 보이는 크기와 각도가 달라짐 |
| Light | X·Y·Z 위치 | 빛과 그림자 방향이 달라짐 |
| Light | 밝기 | 이미지의 밝기가 달라짐 |

이 조건을 변경하면 같은 3D 모델로도 서로 다른 합성 이미지를 만들 수 있습니다.

```text
[기존 실습]

과자 상자 + 머스터드병
           ↓
객체 위치 고정
카메라 위치 고정
조명 위치 고정
           ↓
합성 이미지 1장 생성


[이번 실습]

과자 상자 + 머스터드병
           ↓
객체 위치·회전 변경
카메라 위치·거리 변경
조명 위치·밝기 변경
           ↓
서로 다른 합성 이미지 생성
```

단, 무작위로 변경한다고 해서 무조건 좋은 데이터가 만들어지는 것은 아닙니다.

물체가 화면 밖으로 벗어나거나, 조명이 지나치게 어둡거나, 실제 촬영 환경에서는 발생하기 어려운 이미지가 만들어질 수도 있습니다.

따라서 **Randomization으로 다양한 이미지를 생성한 다음에는 QA를 통해 결과가 적절한지 확인해야 합니다.**

---

# 24. 촬영 조건을 무작위로 변경하여 합성 이미지 한 장을 생성합니다

## 이 코드는 무엇을 하나요?

23번에서 Randomization이 필요한 이유를 이해했습니다.

이제 실제 Python 코드로 객체·카메라·조명의 조건을 자동으로 변경하고, **새로운 합성 이미지 한 장과 정답 데이터를 생성하는 프로그램**을 작성하겠습니다.

앞에서 작성한 `02_render_single_scene.py`와 기본적인 렌더링 과정은 같습니다.

하지만 가장 중요한 차이가 있습니다.

- `02_render_single_scene.py`: 미리 정해진 위치와 조명으로 이미지를 생성합니다.
- `04_render_random_sample.py`: 정해진 범위 안에서 위치와 조명을 무작위로 선택하여 이미지를 생성합니다.

즉, 17번에서 작성한 렌더링 코드를 발전시켜 **다양한 합성데이터를 자동으로 생성할 수 있도록 만드는 단계**입니다.

**파일: `scripts/04_render_random_sample.py`**

```python
import blenderproc as bproc

import argparse
import json
import random
import sys
from pathlib import Path

import numpy as np


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day04_common import (
    bop_dataset_path,
    data_config,
    lesson_objects,
)


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
        "--seed",
        type=int,
        required=True,
    )

    parser.add_argument(
        "--output",
        required=True,
    )

    parser.add_argument(
        "--camera-distance-min",
        type=float,
        default=0.65,
    )

    parser.add_argument(
        "--camera-distance-max",
        type=float,
        default=0.85,
    )

    parser.add_argument(
        "--light-min",
        type=float,
        default=350.0,
    )

    parser.add_argument(
        "--light-max",
        type=float,
        default=800.0,
    )

    parser.add_argument(
        "--rotation-max-deg",
        type=float,
        default=25.0,
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    if not (
        0.2
        <= args.camera_distance_min
        <= args.camera_distance_max
    ):
        raise ValueError(
            "Camera Distance 범위를 확인하세요."
        )

    if not (
        1.0
        <= args.light_min
        <= args.light_max
    ):
        raise ValueError(
            "Light 범위를 확인하세요."
        )

    if not (
        0
        <= args.rotation_max_deg
        <= 180
    ):
        raise ValueError(
            "Rotation 범위를 확인하세요."
        )

    rng = random.Random(args.seed)
    np.random.seed(args.seed)

    output_dir = Path(args.output)
    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    bproc.init()

    config = data_config(
        args.asset_set
    )

    object_info = {
        int(item["bop_obj_id"]): item
        for item
        in lesson_objects(args.asset_set)
    }

    bop_path = bop_dataset_path(
        args.asset_set
    )

    bop_ids = [
        int(value)
        for value
        in config["bop_obj_ids"]
    ]

    objects = bproc.loader.load_bop_objs(
        bop_dataset_path=str(bop_path),
        obj_ids=bop_ids,
        object_model_unit="mm",
        move_origin_to_x_y_plane=True,
    )

    if len(objects) != len(bop_ids):
        raise RuntimeError(
            "BOP Object Loading 수가 다릅니다."
        )

    for obj in objects:
        bop_id = int(
            obj.get_cp("category_id")
        )

        item = object_info[bop_id]

        obj.set_cp(
            "bop_obj_id",
            bop_id,
        )

        obj.set_cp(
            "category_id",
            int(item["lesson_class_id"]),
        )

        obj.set_cp(
            "supercategory",
            "coco_annotations",
        )

        obj.set_name(
            item["class_name"]
        )

        obj.set_shading_mode("auto")

    if len(objects) == 2:
        base_x = [-0.13, 0.13]
    else:
        base_x = [0.0]

    object_rows = []

    rotation_max = np.deg2rad(
        args.rotation_max_deg
    )

    for index, obj in enumerate(objects):
        x = (
            base_x[index]
            + rng.uniform(
                -0.035,
                0.035,
            )
        )

        y = rng.uniform(
            -0.04,
            0.04,
        )

        rotation_z = rng.uniform(
            -rotation_max,
            rotation_max,
        )

        obj.set_location(
            [x, y, 0.0]
        )

        obj.set_rotation_euler(
            [0.0, 0.0, rotation_z]
        )

        object_rows.append(
            {
                "name": obj.get_name(),
                "lesson_class_id": int(
                    obj.get_cp("category_id")
                ),
                "bop_obj_id": int(
                    obj.get_cp("bop_obj_id")
                ),
                "location": [
                    x,
                    y,
                    0.0,
                ],
                "rotation_z": rotation_z,
            }
        )

    floor = bproc.object.create_primitive(
        "PLANE",
        scale=[0.60, 0.60, 1.0],
    )

    floor.set_name("floor")
    floor.set_location(
        [0.0, 0.0, -0.001]
    )
    floor.set_cp("category_id", 0)
    floor.set_cp(
        "supercategory",
        "background",
    )

    light_location = [
        rng.uniform(-0.45, 0.45),
        rng.uniform(-0.45, 0.15),
        rng.uniform(0.65, 1.15),
    ]

    light_energy = rng.uniform(
        args.light_min,
        args.light_max,
    )

    light = bproc.types.Light()
    light.set_type("POINT")
    light.set_location(
        light_location
    )
    light.set_energy(
        light_energy
    )

    camera_distance = rng.uniform(
        args.camera_distance_min,
        args.camera_distance_max,
    )

    camera_location = np.array(
        [
            rng.uniform(-0.13, 0.13),
            -camera_distance,
            rng.uniform(0.26, 0.48),
        ]
    )

    poi = bproc.object.compute_poi(
        objects
    )

    rotation_matrix = (
        bproc.camera
        .rotation_from_forward_vec(
            poi - camera_location
        )
    )

    camera_pose = (
        bproc.math
        .build_transformation_mat(
            camera_location,
            rotation_matrix,
        )
    )

    bproc.camera.add_camera_pose(
        camera_pose
    )

    bproc.camera.set_resolution(
        512,
        512,
    )

    bproc.renderer.enable_depth_output(
        activate_antialiasing=False
    )

    render_data = (
        bproc.renderer.render()
    )

    seg_data = (
        bproc.renderer.render_segmap(
            map_by=[
                "instance",
                "class",
                "name",
            ],
            default_values={
                "category_id": 0,
            },
        )
    )

    render_data.update(seg_data)

    bproc.writer.write_hdf5(
        str(output_dir),
        render_data,
    )

    coco_dir = (
        output_dir
        / "coco_data"
    )

    bproc.writer.write_coco_annotations(
        str(coco_dir),
        instance_segmaps=(
            seg_data["instance_segmaps"]
        ),
        instance_attribute_maps=(
            seg_data[
                "instance_attribute_maps"
            ]
        ),
        colors=render_data["colors"],
        color_file_format="JPEG",
        supercategory="coco_annotations",
        append_to_existing_output=False,
    )

    metadata = {
        "asset_set": args.asset_set,
        "seed": args.seed,
        "objects": object_rows,
        "camera_distance": camera_distance,
        "camera_location": (
            camera_location.tolist()
        ),
        "light_location": light_location,
        "light_energy": light_energy,
        "rotation_max_deg": (
            args.rotation_max_deg
        ),
        "camera_distance_min": (
            args.camera_distance_min
        ),
        "camera_distance_max": (
            args.camera_distance_max
        ),
        "light_min": args.light_min,
        "light_max": args.light_max,
        "approved": "pending",
    }

    with (
        output_dir
        / "sample_metadata.json"
    ).open(
        "w",
        encoding="utf-8",
    ) as file:
        json.dump(
            metadata,
            file,
            ensure_ascii=False,
            indent=2,
        )

    print(
        "Completed Seed:",
        args.seed,
    )

    print(
        "Output        :",
        output_dir,
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
Set과 Seed를 받는다
        ↓
Randomization 범위를 받는다
        ↓
3D Object를 불러온다
        ↓
Object 위치와 회전을 Random하게 정한다
        ↓
Light 위치와 밝기를 Random하게 정한다
        ↓
Camera 거리와 위치를 Random하게 정한다
        ↓
RGB / Depth / Segmentation을 Rendering한다
        ↓
HDF5와 COCO를 저장한다
        ↓
실제 사용한 값을 Metadata에 기록한다
```

---
# 25. Randomization 결과 한 장을 생성하여 정상 작동을 확인합니다

24번에서는 객체·카메라·조명 조건을 무작위로 변경하여 합성데이터를 생성하는 Python 코드를 작성했습니다.

이제 이 코드가 정상적으로 작동하는지 확인하기 위해 **합성 이미지 한 장을 먼저 생성**하겠습니다.

이처럼 전체 데이터를 생성하기 전에 작은 규모로 프로그램의 정상 작동 여부를 확인하는 작업을 **Smoke Test(기본 동작 검사)**라고 합니다.

### ① 실행 전에 BlenderProc 환경을 확인합니다

앞에서 BlenderProc가 Depth와 Segmentation 데이터를 처리할 때 OpenEXR 기능이 비활성화되어 오류가 발생했습니다.

이 문제는 이미 해결 방법을 알고 있으므로, 이번에는 렌더링 전에 해당 기능을 먼저 활성화하겠습니다.

```bash
export OPENCV_IO_ENABLE_OPENEXR=1
```

설정이 적용되었는지 확인합니다.

```bash
echo $OPENCV_IO_ENABLE_OPENEXR
```

결과가 `1`이면 정상입니다.

이 설정은 BlenderProc가 Depth·Segmentation 데이터에 사용하는 EXR 파일을 정상적으로 읽도록 도와줍니다.

**참고:** 이 설정은 현재 터미널에 적용되므로, 새로운 터미널에서 실행할 때는 다시 설정해야 할 수 있습니다. 앞에서 수정한 BlenderProc 내부 코드는 그대로 사용합니다.

### ② Randomized Sample 한 장을 생성합니다

이제 24번에서 작성한 프로그램을 실행하겠습니다.

```bash
blenderproc run \
  scripts/04_render_random_sample.py \
  --asset-set main \
  --seed 100 \
  --output results/main/smoke/sample_0100 \
  --camera-distance-min 0.65 \
  --camera-distance-max 0.85 \
  --light-min 350 \
  --light-max 800 \
  --rotation-max-deg 25
```

이번 실행에서 사용하는 조건은 다음과 같습니다.

| 실행 옵션 | 의미 |
|---|---|
| `--asset-set main` | 과자 상자와 머스터드병 사용 |
| `--seed 100` | 무작위 조건을 생성할 기준 번호 |
| `--output` | 생성 결과를 저장할 폴더 |
| `--camera-distance-min/max` | 카메라 거리 범위 0.65~0.85 |
| `--light-min/max` | 조명 밝기 범위 350~800 |
| `--rotation-max-deg 25` | 객체 Z축 회전 범위 ±25도 |

프로그램은 이 범위 안에서 촬영 조건을 선택하고, 새로운 합성 이미지와 정답 데이터를 생성합니다.

### ③ 생성된 파일을 확인합니다

렌더링이 완료되었다면 다음 명령어를 실행합니다.

```bash
find results/main/smoke/sample_0100 \
  -maxdepth 3 \
  -type f
```

터미널 출력 결과

```
results/main/smoke/sample_0100/0.hdf5
results/main/smoke/sample_0100/sample_metadata.json
results/main/smoke/sample_0100/coco_data/images/000000.jpg
results/main/smoke/sample_0100/coco_data/coco_annotations.json
```

정상적으로 완료되면 다음과 같은 파일이 만들어져야 합니다.

```text
results/main/smoke/sample_0100/
├── 0.hdf5
├── sample_metadata.json
└── coco_data/
    ├── coco_annotations.json
    └── images/
        └── 000000.jpg
```

각 파일의 역할은 다음과 같습니다.

- `0.hdf5`: RGB·Depth·Segmentation 데이터
- `coco_annotations.json`: 객체 종류와 BBox 등의 정답 정보
- `000000.jpg`: 생성된 합성 RGB 이미지
- `sample_metadata.json`: 실제 사용한 카메라·조명·객체 설정값

### ④ 생성된 이미지를 직접 확인합니다

```bash
blenderproc vis hdf5 \
  results/main/smoke/sample_0100/0.hdf5
```

![[Pasted image 20261015124211.png]]

Viewer에서 RGB·Depth·Class Mask·Instance Mask를 확인합니다.

특히 앞에서 발생했던 문제가 다시 나타나지 않았는지 살펴보세요.

- RGB에 두 물체가 정상적으로 보이는가?
- Semantic Mask에서 객체 종류가 구분되는가?
- Instance Mask에서 개별 영역이 구분되는가?
- Depth에서 물체와 바닥의 깊이 차이가 나타나는가?

모두 잘 나타나 있는것이 확인됩니다.

### ⑤ Depth·Mask·COCO 정답 데이터를 확인합니다

이미지가 보기 좋게 생성되었더라도 실제 정답 데이터가 잘못 저장될 수 있습니다.

따라서 다음 명령어로 저장된 값도 간단히 확인합니다.

```bash
python - <<'PY'
import h5py
import json
import numpy as np

root = "results/main/smoke/sample_0100"

with h5py.File(f"{root}/0.hdf5", "r") as f:
    depth = f["depth"][:]
    valid = depth[np.isfinite(depth) & (depth > 0)]

    print("Depth 자료형:", depth.dtype)
    print("Depth 유효 고유값 수:", np.unique(valid).size)
    print("Class IDs:", np.unique(f["class_segmaps"][:]))
    print("Instance IDs:", np.unique(f["instance_segmaps"][:]))

with open(f"{root}/coco_data/coco_annotations.json") as f:
    coco = json.load(f)

print("COCO Categories:", len(coco["categories"]))
print("COCO Annotations:", len(coco["annotations"]))
PY
```

터미널 출력 결과

```
Depth 자료형: float32
Depth 유효 고유값 수: 21433
Class IDs: [0 1 2]
Instance IDs: [0 1 2 3]
COCO Categories: 2
COCO Annotations: 2
```

각 결과가 무엇을 의미하는지 확인해 보겠습니다.

| 검사 항목            | 실제 결과       | 의미                  |
| ---------------- | ----------- | ------------------- |
| Depth 자료형        | `float32`   | 깊이값이 실수형으로 저장됨      |
| Depth 고유값 수      | 21,433개     | 서로 다른 깊이값이 저장됨      |
| Class IDs        | `[0 1 2]`   | 배경·과자 상자·머스터드병 구분   |
| Instance IDs     | `[0 1 2 3]` | 서로 다른 객체 영역이 구분됨    |
| COCO Categories  | 2개          | 두 종류의 객체 정보가 기록됨    |
| COCO Annotations | 2개          | 두 객체에 대한 정답 정보가 생성됨 |
**특히 앞에서 발생했던 두 가지 문제가 해결되었습니다.**

첫 번째는 Depth 문제입니다. 이전에는 깊이값이 `0`과 `1`만 저장되었지만, 이번에는 서로 다른 깊이값이 21,433개 기록되었습니다. 따라서 물체 사이의 깊이 차이를 표현할 수 있는 데이터가 생성되었습니다.

두 번째는 COCO Annotation 문제입니다. 이전에는 객체의 정답 정보가 비어 있었지만, 이번에는 두 객체에 대한 Annotation이 정상적으로 생성되었습니다.

### ⑥ Smoke Test 결과를 정리합니다

이번 실습에서는 과자 상자와 머스터드병을 이용해 객체 위치·회전, 카메라 거리, 조명 조건을 무작위로 설정하고 새로운 합성 이미지 한 장을 생성했습니다.

실행 결과, RGB·Depth·Segmentation·COCO Annotation이 함께 저장되었고 주요 정답 데이터의 값도 정상적으로 확인되었습니다.

다만 실제 객체 영역과 BBox가 정확히 일치하는지, 이미지가 현실적인지는 이후 QA에서 추가로 확인합니다.

**25번 Smoke Test는 통과했습니다.**

이제 다음 26번에서는 지금 작성한 코드를 자동으로 반복 실행하는 프로그램을 만들고, 27번에서는 서로 다른 촬영 조건의 합성 이미지 8장을 생성하겠습니다.

---

# 26. 여러 Scene을 자동 생성하는 Launcher를 작성합니다

## 이 코드는 무엇을 하나요?

25번에서는 객체·카메라·조명 조건을 무작위로 변경하여 합성 이미지 한 장을 생성하고, 정상적으로 작동하는지 확인했습니다.

이제는 같은 작업을 여러 번 반복하여 다양한 합성데이터를 만들어 보겠습니다.

그런데 합성 이미지 8장을 생성하려고 매번 터미널에 `blenderproc run` 명령어를 직접 입력한다면 시간이 오래 걸리고 실수하기도 쉽습니다.

그래서 **한 번의 명령어로 BlenderProc를 여러 번 실행하고, 생성된 결과를 자동으로 저장하는 프로그램**을 작성하겠습니다.

이처럼 다른 프로그램을 실행하고 전체 작업을 관리하는 코드를 **Launcher(실행 관리 프로그램)**라고 합니다.

**파일: `scripts/05_generate_dataset.py`**

```python
from __future__ import annotations

import argparse
import csv
import shutil
import subprocess
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]


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
        "--count",
        type=int,
        default=8,
    )

    parser.add_argument(
        "--seed-start",
        type=int,
        default=200,
    )

    parser.add_argument(
        "--camera-distance-min",
        type=float,
        default=0.65,
    )

    parser.add_argument(
        "--camera-distance-max",
        type=float,
        default=0.85,
    )

    parser.add_argument(
        "--light-min",
        type=float,
        default=350.0,
    )

    parser.add_argument(
        "--light-max",
        type=float,
        default=800.0,
    )

    parser.add_argument(
        "--rotation-max-deg",
        type=float,
        default=25.0,
    )

    parser.add_argument(
        "--overwrite",
        action="store_true",
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    if args.count < 1:
        raise ValueError(
            "--count는 1 이상이어야 합니다."
        )

    blenderproc = shutil.which(
        "blenderproc"
    )

    if blenderproc is None:
        raise RuntimeError(
            "blenderproc 명령을 찾을 수 없습니다."
        )

    output_root = (
        ROOT
        / "results"
        / args.asset_set
        / args.tag
    )

    if output_root.exists():
        if not args.overwrite:
            raise RuntimeError(
                f"이미 결과 폴더가 있습니다:\n"
                f"{output_root}\n"
                "다시 만들려면 --overwrite를 추가하세요."
            )

        shutil.rmtree(output_root)

    output_root.mkdir(
        parents=True,
        exist_ok=True,
    )

    rows = []

    render_script = (
        ROOT
        / "scripts"
        / "04_render_random_sample.py"
    )

    for index in range(args.count):
        seed = (
            args.seed_start
            + index
        )

        sample_dir = (
            output_root
            / f"sample_{seed:04d}"
        )

        command = [
            blenderproc,
            "run",
            str(render_script),
            "--asset-set",
            args.asset_set,
            "--seed",
            str(seed),
            "--output",
            str(sample_dir),
            "--camera-distance-min",
            str(
                args.camera_distance_min
            ),
            "--camera-distance-max",
            str(
                args.camera_distance_max
            ),
            "--light-min",
            str(args.light_min),
            "--light-max",
            str(args.light_max),
            "--rotation-max-deg",
            str(
                args.rotation_max_deg
            ),
        ]

        print("=" * 64)
        print(
            f"Generating "
            f"{index + 1}/{args.count}"
        )
        print("Seed:", seed)
        print("=" * 64)

        subprocess.run(
            command,
            check=True,
            cwd=ROOT,
        )

        rows.append(
            {
                "sample": sample_dir.name,
                "seed": seed,
                "camera_distance_min": (
                    args.camera_distance_min
                ),
                "camera_distance_max": (
                    args.camera_distance_max
                ),
                "light_min": args.light_min,
                "light_max": args.light_max,
                "rotation_max_deg": (
                    args.rotation_max_deg
                ),
            }
        )

    plan_path = (
        output_root
        / "generation_plan.csv"
    )

    with plan_path.open(
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

    print(
        "Generation Completed:",
        output_root,
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[1단계 — 실행 옵션 확인]

프로그램 실행
      ↓
사용자가 입력한 옵션 확인
      ├─ asset-set      → main / challenge
      ├─ tag            → 결과 저장 폴더 이름
      ├─ count          → 생성할 이미지 수
      ├─ seed-start     → 시작 Seed 번호
      ├─ camera-distance→ 카메라 거리 범위
      ├─ light          → 조명 밝기 범위
      ├─ rotation       → 객체 회전 범위
      └─ overwrite      → 기존 결과 덮어쓰기 여부
      ↓
count가 1 이상인지 검사
      ↓
BlenderProc 실행 명령어가 있는지 확인


[2단계 — 결과 저장 폴더 준비]

결과 폴더 경로 만들기
results / asset-set / tag
      ↓
기존 결과 폴더가 있는가?
      ├─ YES
      │    ↓
      │  --overwrite 옵션이 있는가?
      │    ├─ NO  → 오류 출력 후 중단
      │    └─ YES → 기존 결과 폴더 삭제
      │
      └─ NO → 다음 단계 진행
      ↓
새로운 결과 폴더 생성


[3단계 — 합성 이미지 자동 생성]

04_render_random_sample.py 위치 확인
      ↓
count만큼 반복 실행
      ↓
첫 번째 Sample
      ├─ Seed = 200
      ├─ BlenderProc 실행
      └─ sample_0200/ 저장
      ↓
두 번째 Sample
      ├─ Seed = 201
      ├─ BlenderProc 실행
      └─ sample_0201/ 저장
      ↓
세 번째 Sample
      ├─ Seed = 202
      ├─ BlenderProc 실행
      └─ sample_0202/ 저장
      ↓
       ...
      ↓
여덟 번째 Sample
      ├─ Seed = 207
      ├─ BlenderProc 실행
      └─ sample_0207/ 저장

※ 각 실행마다 Seed를 변경하므로
   객체·카메라·조명 조건이 달라짐

※ 중간에 BlenderProc 실행 오류 발생 시
   프로그램 중단


[4단계 — 생성 조건 기록]

각 Sample의 생성 정보 수집
      ├─ Sample 이름
      ├─ Seed
      ├─ 카메라 거리 설정 범위
      ├─ 조명 밝기 설정 범위
      └─ 객체 회전 설정 범위
      ↓
generation_plan.csv 생성
      ↓
생성에 사용한 설정 범위를 표로 저장

※ 실제 무작위로 선택된 카메라·조명·객체 값은
   각 Sample의 sample_metadata.json에 저장


[5단계 — 최종 결과]

results/main/baseline/
      │
      ├─ generation_plan.csv
      │
      ├─ sample_0200/
      ├─ sample_0201/
      ├─ sample_0202/
      ├─ sample_0203/
      ├─ sample_0204/
      ├─ sample_0205/
      ├─ sample_0206/
      └─ sample_0207/
      ↓
Generation Completed 출력
      ↓
여러 장의 합성데이터 자동 생성 완료
```

---
# 27. Main Dataset을 8장 생성하고 한 화면에서 확인합니다

26번에서 작성한 Launcher를 실행하여 과자 상자와 머스터드병의 합성 이미지 8장을 자동 생성하겠습니다.

이번에는 Seed를 200부터 207까지 변경하면서 객체 위치·회전, 카메라 거리, 조명 조건이 서로 다른 이미지를 만듭니다.

### ① 합성 이미지 8장을 생성합니다

먼저 현재 터미널에서 OpenEXR 기능을 활성화합니다.

```bash
export OPENCV_IO_ENABLE_OPENEXR=1
```

다음 명령어를 실행합니다.

```bash
python scripts/05_generate_dataset.py \
  --asset-set main \
  --tag baseline \
  --count 8 \
  --seed-start 200 \
  --camera-distance-min 0.65 \
  --camera-distance-max 0.85 \
  --light-min 350 \
  --light-max 800 \
  --rotation-max-deg 25
```

이 명령어는 BlenderProc를 8번 자동 실행하고, 각 결과를 별도의 Sample 폴더에 저장합니다.

```text
results/main/baseline/
├── generation_plan.csv
├── sample_0200/
├── sample_0201/
├── sample_0202/
├── sample_0203/
├── sample_0204/
├── sample_0205/
├── sample_0206/
└── sample_0207/
```

### ② 생성된 합성 이미지 8장을 한 장으로 합칩니다

지금은 각 이미지가 별도 폴더에 저장되어 있어 전체 결과를 비교하기 어렵습니다.

따라서 **8장의 RGB 이미지를 4장씩 두 줄로 배치하여 하나의 미리보기 이미지로 저장**하겠습니다.

다음 명령어를 터미널에 그대로 입력합니다.

```bash
python - <<'PY'
from pathlib import Path
from PIL import Image, ImageDraw, ImageOps

root = Path("results/main/baseline")

files = sorted(
    root.glob("sample_*/coco_data/images/*.jpg")
)

if not files:
    raise RuntimeError("합성 이미지를 찾을 수 없습니다.")

# 한 줄에 4장씩 배치
cols = 4
size = 300
label_h = 30
rows = (len(files) + cols - 1) // cols

canvas = Image.new(
    "RGB",
    (cols * size, rows * (size + label_h)),
    "white"
)

draw = ImageDraw.Draw(canvas)

for i, path in enumerate(files):
    with Image.open(path) as img:
        img = ImageOps.contain(
            img.convert("RGB"),
            (size - 10, size - 10)
        )

    x = (i % cols) * size
    y = (i // cols) * (size + label_h)

    canvas.paste(
        img,
        (x + (size - img.width) // 2,
         y + (size - img.height) // 2)
    )

    draw.text(
        (x + 10, y + size + 5),
        path.parents[2].name,
        fill="black"
    )

output = root / "baseline_grid.jpg"
canvas.save(output, quality=95)

print("이미지 수:", len(files))
print("저장 완료:", output)
PY
```

이 코드는 기존 합성 이미지를 수정하지 않고, 각 이미지의 크기를 줄여 한 장의 비교 이미지로 
모으는 역할만 합니다.

### ③ 합쳐진 이미지를 확인합니다

```bash
저장 완료: results/main/baseline/baseline_grid.jpg
```

![[Pasted image 20261015134810.png]]

---

# 28. Sample별 Metadata를 하나의 CSV로 모읍니다

## 이 코드는 무엇을 하나요?

27번에서는 과자 상자와 머스터드병을 이용하여 서로 다른 촬영 조건의 합성 이미지 8장을 생성했습니다.

각 Sample 폴더에는 이미지뿐 아니라 `sample_metadata.json` 파일도 함께 저장되어 있습니다.

이 파일에는 이미지를 생성할 때 사용한 **객체의 위치와 회전, 카메라 위치와 거리, 조명 위치와 밝기, Seed** 등의 정보가 기록되어 있습니다.

그런데 이미지 8장의 생성 조건을 비교하려면 JSON 파일을 하나씩 열어야 하므로 불편합니다.

그래서 이번에는 **8개의 Metadata JSON 파일을 자동으로 읽어 하나의 CSV 파일로 정리하는 
프로그램**을 작성하겠습니다.

### 왜 Metadata를 하나의 CSV로 모으나요?

예를 들어 어떤 합성 이미지는 밝고, 어떤 이미지는 어둡게 생성되었다고 생각해 보겠습니다.

이때 Metadata를 확인하면 두 이미지가 각각 어떤 조명 밝기에서 만들어졌는지 비교할 수 있습니다.

또한 물체가 너무 작게 보이는 이미지가 있다면 해당 Sample의 카메라 거리를 확인할 수 있습니다.

즉, **이미지의 차이가 어떤 생성 조건에서 발생했는지 분석하기 위한 작업**입니다.


**파일: `scripts/06_collect_metadata.py`**

```python
from __future__ import annotations

import argparse
import json
from pathlib import Path

import pandas as pd


ROOT = Path(__file__).resolve().parents[1]


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

    return parser.parse_args()


def main() -> None:
    args = parse_args()

    source_root = (
        ROOT
        / "results"
        / args.asset_set
        / args.tag
    )

    rows = []

    for sample_dir in sorted(
        source_root.glob("sample_*")
    ):
        metadata_path = (
            sample_dir
            / "sample_metadata.json"
        )

        if not metadata_path.exists():
            continue

        with metadata_path.open(
            "r",
            encoding="utf-8",
        ) as file:
            data = json.load(file)

        row = {
            "sample": sample_dir.name,
            "asset_set": (
                data["asset_set"]
            ),
            "seed": data["seed"],
            "camera_distance": (
                data["camera_distance"]
            ),
            "camera_x": (
                data["camera_location"][0]
            ),
            "camera_y": (
                data["camera_location"][1]
            ),
            "camera_z": (
                data["camera_location"][2]
            ),
            "light_x": (
                data["light_location"][0]
            ),
            "light_y": (
                data["light_location"][1]
            ),
            "light_z": (
                data["light_location"][2]
            ),
            "light_energy": (
                data["light_energy"]
            ),
            "rotation_max_deg": (
                data["rotation_max_deg"]
            ),
            "approved": (
                data["approved"]
            ),
        }

        for object_index, item in enumerate(
            data["objects"],
            start=1,
        ):
            prefix = f"object{object_index}"

            row[
                f"{prefix}_name"
            ] = item["name"]

            row[
                f"{prefix}_class_id"
            ] = (
                item["lesson_class_id"]
            )

            row[
                f"{prefix}_bop_id"
            ] = item["bop_obj_id"]

            row[
                f"{prefix}_x"
            ] = item["location"][0]

            row[
                f"{prefix}_y"
            ] = item["location"][1]

            row[
                f"{prefix}_rotation_z"
            ] = item["rotation_z"]

        rows.append(row)

    if not rows:
        raise RuntimeError(
            "수집할 Metadata가 없습니다."
        )

    dataframe = pd.DataFrame(rows)

    report_dir = (
        ROOT
        / "reports"
        / args.asset_set
    )

    report_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    output_path = (
        report_dir
        / f"{args.tag}_metadata.csv"
    )

    dataframe.to_csv(
        output_path,
        index=False,
        encoding="utf-8-sig",
    )

    print(
        "Samples:",
        len(dataframe),
    )

    print("Saved:", output_path)


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[1단계 — 실행 옵션 확인]

프로그램 실행
       ↓
사용자가 입력한 옵션 확인
       ├─ --asset-set → main 또는 challenge
       └─ --tag       → 실험 결과 폴더 이름
       ↓
Metadata를 읽어올 폴더 찾기
results/main/baseline/


[2단계 — Sample별 Metadata 읽기]

sample_* 폴더 목록 검색
       ↓
첫 번째 Sample 확인
sample_0200/
       ↓
sample_metadata.json 파일이 있는가?
       ├─ NO  → 건너뛰고 다음 Sample 확인
       └─ YES → JSON 파일 읽기
       ↓
필요한 정보 추출
       ├─ Sample 이름
       ├─ Asset Set
       ├─ Seed
       ├─ Camera Distance
       ├─ Camera X / Y / Z
       ├─ Light X / Y / Z
       ├─ Light Energy
       ├─ Rotation Max
       └─ Approved 상태
       ↓

[3단계 — 객체 정보 추출]

JSON 안의 Objects 목록 확인
       ↓
첫 번째 객체 정보 추출
       ├─ Object Name
       ├─ Lesson Class ID
       ├─ BOP Object ID
       ├─ Object X / Y 위치
       └─ Object Z축 회전
       ↓
두 번째 객체가 있는가?
       ├─ YES → 두 번째 객체 정보도 추출
       └─ NO  → 다음 단계 진행
       ↓
추출한 정보를 하나의 행(Row)으로 정리
       ↓
rows 리스트에 추가


[4단계 — 나머지 Sample 반복 처리]

다음 Sample 폴더 확인
sample_0201/
       ↓
JSON 읽기 → Camera·Light·Object 정보 추출
       ↓
rows 리스트에 추가
       ↓
             ...
       ↓
마지막 Sample 확인
sample_0207/
       ↓
모든 Sample 처리 완료
       ↓
수집된 Metadata가 있는가?
       ├─ NO  → 오류 출력 후 프로그램 중단
       └─ YES → 다음 단계 진행


[5단계 — 하나의 CSV 파일로 정리]

수집된 rows 리스트
       ↓
Pandas DataFrame 생성
       ↓
각 Sample을 하나의 행으로 정리
       ↓
결과 저장 폴더 준비
reports/main/
       ↓
CSV 파일 생성
baseline_metadata.csv
       ↓
UTF-8 형식으로 저장


[6단계 — 최종 결과]

reports/main/
└── baseline_metadata.csv
       ↓
Sample 8개의 생성 조건을
하나의 표에서 비교 가능
       ↓
터미널에 결과 출력

Samples: 8
Saved: .../baseline_metadata.csv
       ↓
Metadata 수집 완료
```

실행:

```bash
python scripts/06_collect_metadata.py \
  --asset-set main \
  --tag baseline
```

### 실제 Metadata CSV 실행 결과 해석

27번에서 생성한 합성 이미지 8장의 Metadata를 하나의 CSV 파일로 정리한 결과입니다.

이 표를 보면 **각 이미지가 어떤 카메라·조명·객체 조건으로 만들어졌는지** 확인할 수 있습니다.

| 확인 항목 | 실제 결과 | 쉽게 이해하면 |
|---|---|---|
| Sample | `0200~0207` | 합성 이미지가 총 8장 생성되었습니다. |
| Seed | `200~207` | 이미지마다 다른 무작위 설정을 사용했습니다. |
| Camera Distance | `0.664~0.772` | 이미지마다 카메라의 거리 설정값이 달랐습니다. |
| Light Energy | `357~739` | 이미지마다 조명의 밝기 설정값이 달랐습니다. |
| Object 위치·회전 | 각각 다른 값 | 과자 상자와 머스터드병의 위치와 방향이 달랐습니다. |
| Class ID | `1`, `2` | 1은 과자 상자, 2는 머스터드병을 의미합니다. |
| Approved | `pending` | 아직 이미지 품질을 검사하고 승인하지 않은 상태입니다. |
### 실제 이미지 두 장을 비교해 봅니다

예를 들어 첫 번째와 두 번째 이미지의 생성 조건을 비교하면 다음과 같습니다.

| 항목 | Sample 0200 | Sample 0201 |
|---|---:|---:|
| Seed | 200 | 201 |
| Camera Distance | 0.706 | 0.772 |
| Light Energy | 357 | 739 |

두 이미지는 같은 과자 상자와 머스터드병을 사용했지만, 카메라 거리와 조명 설정이 다릅니다.

따라서 **같은 3D 모델을 사용하더라도 촬영 조건을 변경하여 서로 다른 합성 이미지를 생성할 수 있다는 것**을 알 수 있습니다.

### 최종 결과

이번 실습에서는 합성 이미지 8장을 생성하고, 각 이미지에 사용된 카메라·조명·객체 설정값을 CSV 파일로 정리했습니다.

Randomization이 정상적으로 적용되었다는 것을 확인했지만, 생성된 이미지의 품질까지 검증한 것은 아닙니다.

다음 29번부터는 Automatic QA(자동 품질 검사)를 통해 이미지와 정답 데이터가 정상적으로 생성되었는지 확인하겠습니다.

---

# PART 5. 3D Synthetic Dataset을 QA하고 Real Reference와 비교합니다

---

# 29. Automatic QA가 필요한 이유와 검사 항목을 이해합니다

27번에서는 과자 상자와 머스터드병의 합성 이미지 8장을 생성했고, 28번에서는 각 이미지의 생성 조건을 CSV 파일로 정리했습니다.

이제 생성된 데이터에 문제가 없는지 확인해야 합니다.

예를 들어 이미지가 정상적으로 보이더라도 객체의 위치를 알려주는 BBox가 누락되거나, Class ID가 잘못 저장되었을 수 있습니다.

이런 데이터를 그대로 AI 모델 학습에 사용하면 잘못된 정답을 학습할 수 있습니다.

따라서 이번에는 **Python 코드로 합성데이터의 파일과 정답 정보가 정상인지 자동으로 검사**하겠습니다.

이 작업을 **Automatic QA(자동 품질 검사)**라고 합니다.

### ① 어떤 내용을 검사하나요?

| 검사 대상 | 쉽게 이해하면 |
|---|---|
| HDF5 | RGB·Depth·Mask가 저장된 파일이 있는가? |
| Metadata | 이미지를 생성한 조건이 기록되어 있는가? |
| COCO JSON | 객체의 정답 정보가 저장된 파일이 있는가? |
| RGB | 컬러 이미지 데이터가 포함되어 있는가? |
| Depth | 깊이 데이터가 포함되어 있는가? |
| Semantic Mask | 객체 종류를 구분하는 데이터가 있는가? |
| Instance Mask | 개별 객체를 구분하는 데이터가 있는가? |
| COCO Annotation | 객체의 정답 정보가 누락되지 않았는가? |
| Class ID | 과자 상자 1, 머스터드병 2로 기록되었는가? |
| BBox | 사각형의 크기가 적절하고 이미지 범위를 벗어나지 않는가? |

### ② 왜 자동으로 검사하나요?

이번에는 합성 이미지가 8장뿐이지만, 실제 현업에서는 수천 장 이상의 이미지를 생성할 수도 있습니다.

이때 모든 파일과 정답 데이터를 사람이 하나씩 검사하는 것은 비효율적입니다.

따라서 먼저 프로그램이 파일 누락이나 잘못된 정답 형식을 검사하도록 합니다.

단, Automatic QA가 모든 문제를 찾아내는 것은 아닙니다.

이번에 작성할 코드는 주로 파일 존재 여부와 데이터 구조, COCO Annotation, BBox 범위를 검사합니다. Depth 값이 실제 거리와 맞는지, Mask 경계가 정확한지, 이미지가 현실적인지까지 완전히 판단하지는 않습니다.

이러한 부분은 이후 실제 이미지를 비교하고 사람이 직접 확인하는 **Human QA** 과정에서 추가로 살펴봅니다.

---
# 30. Automatic QA 코드를 작성합니다

## 이 코드는 무엇을 하나요?

29번에서 합성데이터를 검사해야 하는 이유와 검사 항목을 알아보았습니다.

이제 27번에서 생성한 **8개의 Sample을 자동으로 검사하고, 문제가 있는 데이터가 무엇인지 기록하는 프로그램**을 작성하겠습니다.

**파일: `scripts/07_qa_3d_dataset.py`**

```python
from __future__ import annotations

import argparse
import json
from pathlib import Path

import h5py
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]


EXPECTED_OBJECTS = {
    "main": 2,
    "challenge": 1,
}

EXPECTED_CLASS_IDS = {
    "main": {1, 2},
    "challenge": {3},
}


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
        "--overwrite",
        action="store_true",
    )

    return parser.parse_args()


def has_key(keys, words) -> bool:
    for key in keys:
        lower = key.lower()

        if all(
            word in lower
            for word in words
        ):
            return True

    return False


def main() -> None:
    args = parse_args()

    report_dir = (
        ROOT
        / "reports"
        / args.asset_set
    )

    report_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    output_path = (
        report_dir
        / f"{args.tag}_qa.csv"
    )

    if (
        output_path.exists()
        and not args.overwrite
    ):
        raise RuntimeError(
            "이미 QA CSV가 있습니다.\n"
            f"{output_path}\n"
            "Human QA 기록을 보호하기 위해 "
            "자동으로 덮어쓰지 않습니다.\n"
            "처음부터 다시 만들려면 "
            "--overwrite를 추가하세요."
        )

    source_root = (
        ROOT
        / "results"
        / args.asset_set
        / args.tag
    )

    expected_objects = (
        EXPECTED_OBJECTS[
            args.asset_set
        ]
    )

    expected_class_ids = (
        EXPECTED_CLASS_IDS[
            args.asset_set
        ]
    )

    rows = []

    for sample_dir in sorted(
        source_root.glob("sample_*")
    ):
        reasons = []

        hdf5_path = (
            sample_dir / "0.hdf5"
        )

        metadata_path = (
            sample_dir
            / "sample_metadata.json"
        )

        coco_path = (
            sample_dir
            / "coco_data"
            / "coco_annotations.json"
        )

        if not hdf5_path.exists():
            reasons.append(
                "missing_hdf5"
            )

        if not metadata_path.exists():
            reasons.append(
                "missing_metadata"
            )

        if not coco_path.exists():
            reasons.append(
                "missing_coco"
            )

        hdf5_keys = []

        if hdf5_path.exists():
            try:
                with h5py.File(
                    hdf5_path,
                    "r",
                ) as file:
                    hdf5_keys = list(
                        file.keys()
                    )
            except Exception:
                reasons.append(
                    "unreadable_hdf5"
                )

        if hdf5_keys:
            if "colors" not in hdf5_keys:
                reasons.append(
                    "missing_rgb"
                )

            if not (
                "depth" in hdf5_keys
                or "distance" in hdf5_keys
            ):
                reasons.append(
                    "missing_depth"
                )

            if not has_key(
                hdf5_keys,
                ["class", "seg"],
            ):
                reasons.append(
                    "missing_semantic"
                )

            if not has_key(
                hdf5_keys,
                ["instance", "seg"],
            ):
                reasons.append(
                    "missing_instance"
                )

        annotation_count = 0
        invalid_bbox_count = 0
        category_ids = set()

        if coco_path.exists():
            try:
                with coco_path.open(
                    "r",
                    encoding="utf-8",
                ) as file:
                    coco = json.load(file)

                images = {
                    int(item["id"]): item
                    for item
                    in coco.get(
                        "images",
                        [],
                    )
                }

                annotations = coco.get(
                    "annotations",
                    [],
                )

                annotation_count = len(
                    annotations
                )

                if annotation_count < (
                    expected_objects
                ):
                    reasons.append(
                        "missing_annotation"
                    )

                for annotation in annotations:
                    try:
                        category_ids.add(
                            int(
                                annotation[
                                    "category_id"
                                ]
                            )
                        )
                    except (
                        KeyError,
                        TypeError,
                        ValueError,
                    ):
                        reasons.append(
                            "invalid_category_id"
                        )

                    bbox = annotation.get(
                        "bbox"
                    )

                    if (
                        not bbox
                        or len(bbox) != 4
                    ):
                        invalid_bbox_count += 1
                        continue

                    x, y, width, height = bbox

                    image_info = images.get(
                        int(
                            annotation[
                                "image_id"
                            ]
                        )
                    )

                    if image_info is None:
                        invalid_bbox_count += 1
                        continue

                    image_width = float(
                        image_info["width"]
                    )

                    image_height = float(
                        image_info["height"]
                    )

                    if (
                        width <= 1
                        or height <= 1
                        or x < 0
                        or y < 0
                        or x + width
                        > image_width
                        or y + height
                        > image_height
                    ):
                        invalid_bbox_count += 1

                if (
                    category_ids
                    != expected_class_ids
                ):
                    reasons.append(
                        "class_id_mismatch"
                    )

                if invalid_bbox_count:
                    reasons.append(
                        "invalid_bbox"
                    )

            except Exception:
                reasons.append(
                    "unreadable_coco"
                )

        rows.append(
            {
                "sample": sample_dir.name,
                "annotation_count": (
                    annotation_count
                ),
                "category_ids": (
                    ",".join(
                        str(value)
                        for value
                        in sorted(category_ids)
                    )
                ),
                "invalid_bbox_count": (
                    invalid_bbox_count
                ),
                "auto_pass": (
                    len(reasons) == 0
                ),
                "reason": (
                    "|".join(reasons)
                ),
                "human_decision": "",
                "human_note": "",
            }
        )

    if not rows:
        raise RuntimeError(
            "검사할 Sample이 없습니다."
        )

    dataframe = pd.DataFrame(rows)

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

    failed = (
        len(dataframe) - passed
    )

    print("=" * 64)
    print(
        "3D Synthetic Automatic QA"
    )
    print("=" * 64)
    print("Total:", len(dataframe))
    print("PASS :", passed)
    print("FAIL :", failed)
    print("Saved:", output_path)


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[1단계 — 실행 옵션 확인]

Automatic QA 프로그램 실행
        ↓
사용자가 입력한 옵션 확인
        ├─ --asset-set → main / challenge
        ├─ --tag       → 검사할 실험 이름
        └─ --overwrite → 기존 QA 결과 덮어쓰기 여부
        ↓
검사 기준 설정
        ├─ main      → 객체 2개, Class ID {1, 2}
        └─ challenge → 객체 1개, Class ID {3}


[2단계 — QA 결과 저장 위치 확인]

결과 저장 경로 준비
reports/main/baseline_qa.csv
        ↓
기존 QA CSV 파일이 있는가?
        ├─ YES
        │    ↓
        │  --overwrite 옵션이 있는가?
        │    ├─ NO  → 오류 출력 후 중단
        │    └─ YES → 검사 계속 진행
        │
        └─ NO → 검사 계속 진행
        ↓
검사할 합성데이터 폴더 찾기
results/main/baseline/


[3단계 — Sample별 파일 확인]

sample_* 폴더 목록 가져오기
        ↓
첫 번째 Sample 선택
sample_0200/
        ↓
필수 파일이 있는지 확인
        ├─ 0.hdf5
        ├─ sample_metadata.json
        └─ coco_data/coco_annotations.json
        ↓
파일이 없다면 오류 원인 기록
        ├─ missing_hdf5
        ├─ missing_metadata
        └─ missing_coco
        ↓
존재하는 파일에 대해 다음 검사 진행


[4단계 — HDF5 데이터 구조 검사]

0.hdf5 파일 열기
        ↓
파일을 정상적으로 읽을 수 있는가?
        ├─ NO → unreadable_hdf5 기록
        └─ YES → HDF5 Key 목록 확인
        ↓
필요한 데이터가 있는지 검사
        ├─ colors              → RGB
        ├─ depth / distance    → Depth
        ├─ class_segmaps       → Semantic Mask
        └─ instance_segmaps    → Instance Mask
        ↓
필요한 Key가 없으면 오류 기록
        ├─ missing_rgb
        ├─ missing_depth
        ├─ missing_semantic
        └─ missing_instance

※ 이 단계에서는 데이터의 존재 여부를 검사합니다.
   Depth 실제 수치나 Mask 픽셀값까지
   검증하는 코드는 아닙니다.


[5단계 — COCO 정답 데이터 검사]

coco_annotations.json 파일 읽기
        ↓
이미지 정보(images) 가져오기
        ↓
객체 정답 정보(annotations) 가져오기
        ↓
Annotation 개수 확인
        ↓
Main 기준 객체 2개보다 적은가?
        ├─ YES → missing_annotation 기록
        └─ NO  → 다음 검사 진행
        ↓
각 Annotation의 Class ID 확인
        ↓
실제 Class ID 집합이 {1, 2}와 같은가?
        ├─ NO  → class_id_mismatch 기록
        └─ YES → 다음 검사 진행


[6단계 — BBox 위치와 크기 검사]

각 Annotation의 BBox 가져오기
        ↓
BBox = [x, y, width, height]
        ↓
다음 조건 검사
        ├─ BBox 값이 4개인가?
        ├─ width와 height가 1보다 큰가?
        ├─ x와 y가 음수가 아닌가?
        ├─ BBox 오른쪽 끝이 이미지 안에 있는가?
        └─ BBox 아래쪽 끝이 이미지 안에 있는가?
        ↓
잘못된 BBox 발견
        → invalid_bbox 기록
        ↓
COCO 파일 자체를 읽거나 처리하지 못하면
        → unreadable_coco 기록


[7단계 — Sample별 QA 결과 기록]

지금까지 발견한 오류 목록 확인
        ↓
오류가 하나도 없는가?
        ├─ YES → auto_pass = True
        └─ NO  → auto_pass = False
        ↓
Sample 검사 결과를 하나의 행으로 정리
        ├─ sample
        ├─ annotation_count
        ├─ category_ids
        ├─ invalid_bbox_count
        ├─ auto_pass
        ├─ reason
        ├─ human_decision → 빈칸
        └─ human_note     → 빈칸
        ↓
rows 리스트에 추가


[8단계 — 나머지 Sample 반복 검사]

다음 Sample 선택
sample_0201/
        ↓
파일 확인 → HDF5 검사 → COCO 검사 → BBox 검사
        ↓
QA 결과 기록
        ↓
           ...
        ↓
마지막 Sample 검사
sample_0207/
        ↓
모든 Sample 검사 완료
        ↓
검사한 Sample이 하나도 없다면 오류 발생


[9단계 — QA 결과 CSV 저장]

8개 Sample의 검사 결과 수집
        ↓
Pandas DataFrame 생성
        ↓
하나의 CSV 파일로 저장
        ↓
reports/main/baseline_qa.csv


[10단계 — 최종 검사 결과 출력]

전체 Sample 수 계산
        ↓
auto_pass = True인 Sample 수 계산
        ↓
auto_pass = False인 Sample 수 계산
        ↓
터미널에 다음 정보 출력

Total → 전체 검사 수
PASS  → 자동 검사 통과 수
FAIL  → 자동 검사 실패 수
Saved → QA CSV 저장 위치
        ↓
Automatic QA 완료
```

### Automatic QA를 실행하고 결과를 확인합니다

앞에서 작성한 Automatic QA 프로그램을 실행하여 합성데이터 8개를 검사하겠습니다.

```bash
python scripts/07_qa_3d_dataset.py \
  --asset-set main \
  --tag baseline
```

### ① 실제 실행 결과

```
================================================================
3D Synthetic Automatic QA
================================================================
Total: 8
PASS : 7
FAIL : 1
Saved: /home/user1/project11/day04_3d_segmentation/reports/main/baseline_qa.csv
```

실행 결과를 해석하면 다음과 같습니다.

|검사 항목|실제 결과|의미|
|---|---|---|
|Total|8|합성데이터 8개를 검사했습니다.|
|PASS|7|7개는 자동 검사 항목을 통과했습니다.|
|FAIL|1|1개에서 문제가 발견되었습니다.|
|Saved|`baseline_qa.csv`|검사 결과가 CSV 파일로 저장되었습니다.|

### ② FAIL이 발생한 Sample을 확인합니다

Automatic QA 결과, 8개 중 1개가 검사에 통과하지 못했습니다.

어떤 Sample에 문제가 있는지 확인하기 위해 다음 코드를 실행했습니다.

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv("reports/main/baseline_qa.csv")

failed = df[
    df["auto_pass"].astype(str).str.lower() == "false"
]

print(failed[["sample", "reason"]].to_string(index=False))
PY
```

**실제 실행 결과**

```
sample         reason
sample_0206    missing_annotation|class_id_mismatch
```

결과를 쉽게 해석하면 다음과 같습니다.

|확인 항목|실제 결과|의미|
|---|---|---|
|`sample`|`sample_0206`|Seed 206으로 생성한 데이터에서 문제가 발견됨|
|`missing_annotation`|오류 발생|필요한 객체 2개에 대한 Annotation이 모두 기록되지 않음|
|`class_id_mismatch`|오류 발생|Annotation에 기록된 Class ID가 예상한 `{1, 2}`와 일치하지 않음|

이번 실습에서는 과자 상자(Class ID 1)와 머스터드병(Class ID 2), 두 객체의 정답 정보가 필요합니다.

그런데 `sample_0206`에서는 Annotation 개수가 2개보다 적고, 기록된 Class ID도 예상과 일치하지 않았습니다.

**왜 이런 문제가 발생했을까요?**

카메라와 객체 위치를 무작위로 변경하면서 한 객체가 화면 밖으로 벗어났거나 다른 객체에 가려졌을 가능성이 있습니다.

다만 정확한 원인은 아직 확인되지 않았습니다.

### ③ 문제가 발생한 Sample을 추가 확인합니다

다음 명령어로 해당 Sample의 COCO Annotation을 확인합니다.

```bash
python - <<'PY'
import json

path = "results/main/baseline/sample_0206/coco_data/coco_annotations.json"

with open(path, encoding="utf-8") as f:
    data = json.load(f)

print("Annotation 수:", len(data["annotations"]))

for item in data["annotations"]:
    print("Class ID:", item["category_id"])
    print("BBox:", item["bbox"])
PY
```

**실제 실행 결과**

```text
Annotation 수: 1
Class ID: 1
BBox: [208, 144, 85, 239]
```

결과를 쉽게 해석하면 다음과 같습니다.

| 확인 항목 | 실제 결과 | 의미 |
|---|---|---|
| Annotation 수 | 1개 | 두 객체 중 한 객체의 정답만 기록됨 |
| Class ID | `1` | 과자 상자(`cracker_box`) |
| BBox | `[208, 144, 85, 239]` | 과자 상자의 위치와 크기 |
| 누락된 Class ID | `2` | 머스터드병(`mustard_bottle`)의 정답이 없음 |

이번 실습에서는 과자 상자(Class ID 1)와 머스터드병(Class ID 2)을 함께 렌더링했습니다.

그런데 검사 결과, **과자 상자의 BBox만 기록되었고 머스터드병의 Annotation은 생성되지 않았습니다.**

따라서 Automatic QA에서 다음 두 오류가 발생했습니다.

- `missing_annotation`: 필요한 Annotation 2개 중 1개만 존재합니다.
- `class_id_mismatch`: 예상한 Class ID `{1, 2}` 중 `2`가 누락되었습니다.

두 오류는 모두 **머스터드병의 정답 정보가 없는 것과 관련되어 있습니다.**

### ④ 머스터드병이 왜 누락되었을까요?

Randomization 과정에서 카메라와 객체 위치가 변경되면서 머스터드병이 화면 밖으로 벗어났거나 다른 물체에 가려졌을 가능성이 있습니다.

다만 COCO 결과만으로는 정확한 원인을 알 수 없습니다.

따라서 다음 명령어로 실제 생성된 RGB 이미지를 열어 확인합니다.

```bash
explorer.exe "$(wslpath -w results/main/baseline/sample_0206/coco_data/images/000000.jpg)"
```

![[Pasted image 20261015145659.png|330]]

`sample_0206`의 RGB 이미지를 직접 확인한 결과, **과자 상자만 보이고 머스터드병은 화면에 나타나지 않았습니다.**

따라서 COCO Annotation에도 과자 상자(Class ID 1)만 기록되어 있었으며, 머스터드병(Class ID 2)의 정답 정보는 누락되었습니다.

이번 오류는 단순히 JSON 저장 문제라기보다, **Randomization으로 설정한 촬영 조건에서 머스터드병이 카메라 화면에 나타나지 않은 것과 관련이 있습니다.**

### 최종 정리

전체 8개 Sample 중 7개는 Automatic QA를 통과했고, 1개(`sample_0206`)는 필요한 객체가 모두 보이지 않아 FAIL 처리되었습니다.

**이번 실습을 통해 합성데이터를 자동 생성한 뒤에는 객체가 정상적으로 화면에 나타나는지, 정답 정보가 제대로 생성되었는지 QA로 반드시 확인해야 한다는 점을 배웠습니다.**

---

# 31. Real과 Synthetic을 한 화면에서 비교하는 코드를 작성합니다

## 이 코드는 무엇을 하나요?

앞에서는 BlenderProc로 과자 상자와 머스터드병의 합성 이미지 8장을 생성하고 Automatic QA를 수행했습니다.

그런데 RGB 이미지와 정답 데이터가 정상적으로 만들어졌다고 해서 실제 촬영한 사진과 비슷하다는 뜻은 아닙니다.

예를 들어 실제 사진에는 자연스러운 그림자나 복잡한 배경이 있지만, 합성 이미지에는 단순한 바닥과 인공적인 조명만 나타날 수 있습니다.

이처럼 **실제 데이터와 합성데이터 사이에 나타나는 차이를 Domain Gap(도메인 차이)**이라고 합니다.

이번에는 실제 촬영 사진과 합성 이미지를 한 화면에 나란히 배치하여 이러한 차이를 쉽게 비교할 수 있도록 만들겠습니다.

**파일: `scripts/08_compare_real_synthetic.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import (
    Image,
    ImageDraw,
    ImageOps,
)


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day04_common import (
    lesson_objects,
    real_reference_root,
)


TILE_W = 240
TILE_H = 190
LABEL_H = 36


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

    return parser.parse_args()


def image_files(
    directory: Path,
) -> list[Path]:
    if not directory.exists():
        return []

    return sorted(
        path
        for path in directory.rglob("*")
        if path.is_file()
        and path.suffix.lower()
        in {".png", ".jpg", ".jpeg"}
    )


def make_row(
    files,
    label_prefix,
    columns,
):
    selected = files[:columns]

    canvas = Image.new(
        "RGB",
        (
            TILE_W * columns,
            TILE_H + LABEL_H,
        ),
        "white",
    )

    draw = ImageDraw.Draw(canvas)

    for column, path in enumerate(
        selected
    ):
        x = column * TILE_W

        with Image.open(path) as image:
            image = (
                ImageOps
                .exif_transpose(image)
                .convert("RGB")
            )

            image.thumbnail(
                (
                    TILE_W - 12,
                    TILE_H - 12,
                ),
                Image.Resampling.LANCZOS,
            )

            px = (
                x
                + (TILE_W - image.width)
                // 2
            )

            py = (
                (TILE_H - image.height)
                // 2
            )

            canvas.paste(
                image,
                (px, py),
            )

        draw.text(
            (
                x + 5,
                TILE_H + 7,
            ),
            f"{label_prefix} {column + 1}",
            fill="black",
        )

    return canvas


def main() -> None:
    args = parse_args()

    real_root = real_reference_root(
        args.asset_set
    )

    objects = lesson_objects(
        args.asset_set
    )

    real_files = []

    per_class = max(
        1,
        6 // len(objects),
    )

    for item in objects:
        class_name = item[
            "class_name"
        ]

        class_files = image_files(
            real_root
            / class_name
        )

        real_files.extend(
            class_files[:per_class]
        )

    synthetic_root = (
        ROOT
        / "results"
        / args.asset_set
        / args.tag
    )

    synthetic_files = sorted(
        synthetic_root.glob(
            "sample_*/"
            "coco_data/"
            "images/*"
        )
    )

    if not real_files:
        raise RuntimeError(
            "Real Reference가 없습니다."
        )

    if not synthetic_files:
        raise RuntimeError(
            "Synthetic RGB가 없습니다."
        )

    columns = min(
        len(real_files),
        len(synthetic_files),
        6,
    )

    real_row = make_row(
        real_files,
        "REAL",
        columns,
    )

    synthetic_row = make_row(
        synthetic_files,
        "SYNTHETIC",
        columns,
    )

    canvas = Image.new(
        "RGB",
        (
            max(
                real_row.width,
                synthetic_row.width,
            ),
            (
                real_row.height
                + synthetic_row.height
            ),
        ),
        "white",
    )

    canvas.paste(
        real_row,
        (0, 0),
    )

    canvas.paste(
        synthetic_row,
        (0, real_row.height),
    )

    output_path = (
        synthetic_root
        / "domain_gap_grid.jpg"
    )

    canvas.save(
        output_path,
        quality=92,
    )

    print("Saved:", output_path)


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
[1단계 — 실행 옵션 확인]

프로그램 실행
        ↓
사용자가 입력한 옵션 확인
        ├─ --asset-set → main / challenge
        └─ --tag       → 실험 결과 폴더 이름
        ↓
실제 촬영 이미지와 합성 이미지의
저장 경로 확인


[2단계 — 실제 촬영 이미지 준비]

Real Reference 폴더 찾기
        ↓
실습에 사용할 객체 목록 확인
        ↓
main 데이터인 경우
        ├─ cracker_box 실제 사진 3장 선택
        └─ mustard_bottle 실제 사진 3장 선택
        ↓
총 6장의 실제 사진 목록 만들기
        ↓
real_files에 저장


[3단계 — 합성 이미지 준비]

합성 이미지 폴더 찾기
results/main/baseline/
        ↓
각 Sample 폴더의 RGB 이미지 검색
        ↓
sample_0200/ → 000000.jpg
sample_0201/ → 000000.jpg
sample_0202/ → 000000.jpg
        ...
sample_0207/ → 000000.jpg
        ↓
파일명 순서대로 이미지 목록 정렬
        ↓
synthetic_files에 저장


[4단계 — 이미지 존재 여부 확인]

실제 촬영 이미지가 있는가?
        ├─ NO  → 오류 출력 후 중단
        └─ YES → 다음 검사
        ↓
합성 이미지가 있는가?
        ├─ NO  → 오류 출력 후 중단
        └─ YES → 다음 단계 진행
        ↓
비교할 이미지 수 결정
        ↓
실제·합성 이미지 수 중
작은 값을 선택하되 최대 6장


[5단계 — 실제 촬영 사진을 한 줄로 배치]

make_row() 함수 실행
        ↓
Real 이미지 6장 선택
        ↓
각 이미지 반복 처리
        ├─ 이미지 열기
        ├─ EXIF 촬영 방향 반영
        ├─ RGB 형식으로 변환
        ├─ 원본 비율을 유지하며 크기 축소
        ├─ 지정된 위치에 이미지 배치
        └─ 이미지 아래 REAL 번호 표시
        ↓
실제 사진 6장을 한 줄로 정리
        ↓
real_row 완성


[6단계 — 합성 이미지를 한 줄로 배치]

make_row() 함수 다시 실행
        ↓
Synthetic 이미지 6장 선택
        ↓
각 이미지 반복 처리
        ├─ 이미지 열기
        ├─ RGB 형식으로 변환
        ├─ 원본 비율을 유지하며 크기 축소
        ├─ 지정된 위치에 이미지 배치
        └─ 이미지 아래 SYNTHETIC 번호 표시
        ↓
합성 이미지 6장을 한 줄로 정리
        ↓
synthetic_row 완성


[7단계 — Real과 Synthetic 합치기]

두 줄을 담을 흰색 배경 생성
        ↓
위쪽에 실제 촬영 사진 배치
        ↓
아래쪽에 합성 이미지 배치
        ↓

┌─────────────────────────────────────┐
│ REAL 1  REAL 2  REAL 3  ... REAL 6 │
├─────────────────────────────────────┤
│ SYN 1   SYN 2   SYN 3   ... SYN 6  │
└─────────────────────────────────────┘

        ↓
한 장의 비교 이미지 완성


[8단계 — 비교 이미지 저장]

결과 저장 경로 지정
        ↓
results/main/baseline/
        ↓
domain_gap_grid.jpg 저장
        ↓
터미널에 저장 경로 출력
        ↓
프로그램 종료


[9단계 — 이후 Human QA에서 활용]

domain_gap_grid.jpg 열기
        ↓
실제 사진과 합성 이미지 비교
        ├─ 객체의 형태
        ├─ 객체의 크기
        ├─ 카메라 각도
        ├─ 조명과 그림자
        ├─ 재질과 색상
        └─ 배경 차이
        ↓
Domain Gap 확인
        ↓
합성데이터의 개선할 부분 판단
```

실행:

```bash
python scripts/08_compare_real_synthetic.py \
  --asset-set main \
  --tag baseline
```

결과:

```text
results/main/baseline/domain_gap_grid.jpg
```

![[Pasted image 20261015150356.png]]

실제 생성된 `domain_gap_grid.jpg`를 확인한 결과, 3D 모델의 형태와 색상은 비교적 잘 표현되었지만, 실제 사진과 합성 이미지 사이에는 배경·조명·객체 배치에서 뚜렷한 차이(Domain Gap)가 나타났습니다.

### Real과 Synthetic 비교 결과를 해석합니다

실제 촬영 이미지 6장과 BlenderProc로 생성한 합성 이미지 6장을 한 화면에 배치하여 비교했습니다.

| 비교 항목 | 실제 이미지(Real) | 합성 이미지(Synthetic) |
|---|---|---|
| 객체 형태·색상 | 실제 제품의 모습 | 제품 형태와 포장 색상이 비교적 잘 표현됨 |
| 배경 | 책상·의자·다른 물체 등 다양한 환경 | 단순한 흰색 바닥과 회색 배경 |
| 조명·그림자 | 촬영 환경에 따라 다양함 | 비교적 단순한 조명과 그림자 |
| 객체 배치 | 여러 물체가 함께 놓여 있음 | 과자 상자와 머스터드병 중심 |
| 객체 가시성 | 다양한 위치에서 촬영됨 | 일부 이미지에서 머스터드병이 화면 오른쪽에 잘림 |
| 이미지 질감 | 실제 카메라의 노이즈와 질감이 나타남 | 전체적으로 깨끗하고 단순함 |

### 이번 결과에서 확인한 점

합성 이미지에서는 과자 상자와 머스터드병의 형태와 색상이 비교적 자연스럽게 표현되었습니다.

그러나 실제 사진에는 여러 주변 물체와 복잡한 배경이 존재하는 반면, 합성 이미지는 단순한 바닥과 배경으로 구성되어 있습니다.

또한 일부 합성 이미지에서는 머스터드병이 화면 경계에서 잘려 보입니다. 따라서 카메라 위치와 객체 배치 조건을 개선할 필요가 있습니다.

이러한 차이가 바로 **Domain Gap(실제 데이터와 합성데이터 사이의 차이)**입니다.

### 핵심 정리

이번 실습을 통해 3D Rendering으로 제품의 합성 이미지를 생성할 수 있다는 것을 확인했습니다.

하지만 **합성 이미지가 정상적으로 생성되었다고 해서 실제 촬영 환경까지 충분히 재현한 것은 아닙니다.**

따라서 실제 AI 학습에 활용하려면 배경·조명·주변 물체·카메라 구도를 더욱 다양하게 구성하고, QA를 통해 데이터 품질을 확인해야 합니다.

---

# 32. Human QA와 Domain Gap을 직접 확인합니다

앞에서는 Automatic QA로 합성데이터 8개 중 7개가 통과하고, 1개가 실패한 것을 확인했습니다.

이번에는 실제 촬영 이미지(Real)와 합성 이미지(Synthetic)를 직접 비교하여 **AI 학습에 사용하기에 적절한 데이터인지 사람이 판단**하겠습니다.

이러한 과정을 **Human QA(사람이 직접 수행하는 품질 검사)**라고 합니다.

### ① 실제 결과를 바탕으로 품질을 확인합니다

| 검사 항목 | 실제 확인 결과 |
|---|---|
| Shape / Material | 제품의 형태와 포장 색상이 비교적 잘 표현됨 |
| Size / Camera | 실제 사진과 촬영 거리·구도가 다르며 일부 객체가 잘림 |
| Light | 실제 촬영 환경보다 조명이 단순함 |
| Background | 실제는 다양한 주변 물체가 있지만 합성 이미지는 흰색 바닥 중심 |
| Occlusion | 실제 사진에 비해 주변 물체에 가려지는 상황이 부족함 |
| Noise / Blur | 합성 이미지가 실제 사진보다 깨끗하게 표현됨 |
| BBox / Mask / Depth | 별도의 정답 데이터 검증이 필요함 |

특히 `sample_0206`에서는 머스터드병이 화면에 나타나지 않아 COCO Annotation에서도 누락되었습니다.

따라서 이번 실습에서 확인한 주요 문제는 **객체 누락과 실제 촬영 환경과의 차이(Domain Gap)**입니다.

### ② Human QA 결과를 기록합니다

`reports/main/baseline_qa.csv` 파일의 `human_decision`과 `human_note`에 검사 결과를 기록합니다.

| 판단 | 의미 |
|---|---|
| `approved` | 사람이 확인한 품질 기준을 충족함 |
| `needs_review` | 품질을 개선하거나 추가 확인이 필요함 |
| `rejected` | 현재 기준에서 학습용 데이터로 사용하기 부적절함 |

예를 들어 다음과 같이 기록할 수 있습니다.

**sample_0206**

```text
human_decision: rejected

human_note:
머스터드병이 화면에서 누락되어
필요한 두 객체의 정답 데이터가 모두 생성되지 않음.
```

**나머지 Sample**

실제 사진과 비교하여 배경·조명 등의 차이가 크다고 판단되면 다음과 같이 기록할 수 있습니다.

```text
human_decision: needs_review

human_note:
객체와 정답 데이터는 생성되었지만
실제 촬영 환경에 비해 배경과 조명이 단순함.
```

※ 위 판단은 기록 예시입니다. 각 Sample의 최종 승인 여부는 이미지를 직접 검사한 후 결정합니다.

Human QA를 기록한 CSV는 보존해야 하므로, 기존 파일에 `--overwrite` 옵션을 사용하여 Automatic QA를 다시 실행하지 않습니다.

### ③ 앞으로 합성데이터를 어떻게 개선할까요?

이번 결과를 바탕으로 실제 촬영 환경과 더 비슷한 이미지를 만들려면 다음 부분을 개선할 수 있습니다.

| 개선 항목 | 앞으로 적용할 방법 |
|---|---|
| 객체 누락 방지 | 두 객체가 카메라 화면 안에 들어오도록 위치·거리 조정 |
| 배경 개선 | 실제 사진과 비슷한 책상·바닥·주변 물체 추가 |
| 조명 개선 | 조명의 위치·밝기·그림자를 실제 환경에 가깝게 설정 |
| 현실감 개선 | 카메라 노이즈·Blur·재질 표현 등을 다양하게 적용 |

이러한 개선은 이후 BlenderProc의 Scene 구성과 Randomization 코드를 수정하여 적용할 수 있습니다. 현재 실습에서 모두 자동으로 개선되는 것은 아닙니다.

### 핵심 정리

이번 Human QA에서는 합성 이미지의 형태와 색상은 비교적 잘 생성되었지만, 객체 누락과 배경·조명 등의 Domain Gap이 존재한다는 것을 확인했습니다.

AI 학습용 합성데이터는 많이 생성하는 것뿐 아니라, 정답 데이터가 정확하고 실제 환경과 충분히 유사하도록 검수하고 개선하는 과정이 중요합니다.

---

# PART 6. 3일차와 4일차를 연결하고 Metadata를 해석합니다

---

# 33. 3일차 Cut-Paste와 4일차 3D Rendering을 비교합니다

3일차에서는 실제 배경 사진에 객체 PNG를 붙여 합성 이미지를 만들었습니다.

4일차에서는 3D 모델을 가상 공간에 배치하고 카메라와 조명을 설정하여 새로운 이미지를 만들었습니다.

두 방법의 가장 큰 차이는 **이미지를 생성하는 공간과 자동으로 만들 수 있는 정답 데이터의 종류**입니다.

| 비교 항목 | 3일차 Cut-Paste | 4일차 3D Rendering |
|---|---|---|
| 사용 데이터 | 배경 JPG + 객체 PNG | 3D 모델 |
| 작업 공간 | 2D 이미지 | 3D 가상 공간 |
| 객체 위치 | X, Y | X, Y, Z |
| 객체 회전 | 평면 회전 | 3D 회전 |
| Camera / Light | 기존 사진의 촬영 조건에 영향받음 | 직접 설정 가능 |
| BBox | 붙여 넣은 객체 영역으로 계산 | 렌더링된 객체 영역으로 자동 생성 |
| Segmentation | 별도 구현 필요 | 자동 생성 가능 |
| Depth | 실제 깊이 정보 생성이 어려움 | 자동 생성 가능 |
| 주요 특징 | 빠르고 간단한 이미지 합성 | 다양한 정답 데이터 생성 가능 |

### 핵심 정리

3일차와 4일차 모두 사람이 이미지마다 정답을 직접 그리지 않고 자동으로 생성할 수 있습니다.

하지만 4일차에서는 3D 모델과 카메라 정보를 활용하기 때문에 BBox뿐 아니라 객체 영역(Mask)과 깊이(Depth) 데이터도 함께 생성할 수 있다는 장점이 있습니다.

---
# 34. Metadata에서 한 Sample의 생성 조건을 다시 추적합니다

28번에서는 합성 이미지 8장의 생성 조건을 `baseline_metadata.csv`에 저장했습니다.

이번에는 그중 `sample_0203`이 어떤 조건에서 생성되었는지 직접 확인하겠습니다.

### ① Sample 0203의 Metadata를 확인합니다

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv(
    "reports/main/baseline_metadata.csv"
)

print(
    df[
        df["sample"]
        == "sample_0203"
    ].T
)
PY
```

### ② 실제 실행 결과를 해석합니다

| 확인 항목 | 실제 결과 | 의미 |
|---|---|---|
| Sample | `sample_0203` | 확인할 합성 이미지 |
| Seed | `203` | 무작위 조건을 생성한 기준 번호 |
| Camera Distance | `0.719` | 카메라 거리 설정값 |
| Camera X / Y / Z | `(-0.105, -0.719, 0.458)` | 가상 카메라의 위치 |
| Light X / Y / Z | `(0.295, -0.083, 1.012)` | 가상 조명의 위치 |
| Light Energy | `698.77` | 조명 밝기 설정값 |
| Rotation Max | `25°` | 객체 회전에 허용한 최대 각도 |
| Approved | `pending` | 아직 승인 전 상태 |

객체별 정보도 확인할 수 있습니다.

| 객체 | 위치(X, Y) | 실제 Z축 회전 |
|---|---|---|
| 과자 상자 | `(-0.159, 0.027)` | 약 5.0° |
| 머스터드병 | `(0.110, 0.022)` | 약 4.7° |

여기서 `Rotation Max = 25°`는 회전값의 허용 범위이고, 실제 적용된 회전각은 Metadata에 별도로 기록되어 있습니다.

### ③ Metadata를 왜 확인하나요?

예를 들어 합성 이미지가 너무 어둡거나 물체가 화면 밖으로 벗어났다고 생각해 보겠습니다.

이때 Metadata를 확인하면 해당 이미지를 생성할 때 사용한 카메라와 조명, 객체 위치를 알 수 있습니다.

이처럼 **어떤 데이터가 어떤 조건에서 만들어졌는지 추적하는 것을 Data Lineage(데이터 계보·이력 추적)**라고 합니다.

이번 실습에서는 Seed와 Metadata를 이용하여 합성 이미지의 생성 조건을 추적할 수 있습니다.

다만 Seed가 같더라도 Blender·BlenderProc·GPU·라이브러리 환경이 달라지면 렌더링 결과에 차이가 생길 수 있으므로, `reports/environment_freeze_day04.txt`와 실행환경 정보도 함께 보관하는 것이 좋습니다.

---
# 35. Main 본 실습의 전체 흐름을 정리합니다

지금까지 4일차에서는 과자 상자와 머스터드병의 3D 모델을 이용해 합성데이터를 생성하고, 
정답 데이터와 이미지 품질을 검사했습니다.

### ① 오늘 진행한 전체 과정을 연결합니다

![[Pasted image 20261015152155.png]]

```text
[합성데이터 생성]

YCB-V 3D 모델 준비
과자 상자 + 머스터드병
        ↓
BlenderProc
객체 · 카메라 · 조명 설정
        ↓
첫 번째 Single Rendering
        ↓
RGB · Depth · Semantic · Instance · COCO
        ↓
Randomization 적용
        ↓
합성데이터 8장 생성
        ↓
Metadata CSV 저장


[품질 검사]

합성데이터 8장
        ↓
Automatic QA
        ↓
PASS 7개 / FAIL 1개
        ↓
실패한 sample_0206 확인
        ↓
머스터드병이 화면에 나타나지 않음
        ↓
COCO Annotation 누락 확인


[실제 이미지와 비교]

Real Reference
과자 상자 · 머스터드병 실제 사진
        +
Synthetic RGB
        ↓
Real vs Synthetic 비교
        ↓
배경 · 조명 · 객체 배치 차이 확인
        ↓
Domain Gap 및 개선 방향 정리
```

### ② 실제 실습 결과를 최종 정리합니다

| 확인 항목 | 실제 결과 |
|---|---|
| 3D Rendering | 과자 상자와 머스터드병 합성 이미지 생성 |
| Depth | 실수형 깊이값 정상 생성 |
| Segmentation | Class·Instance 영역 구분 |
| COCO Annotation | 객체 종류와 BBox 정답 생성 |
| Randomization | 서로 다른 조건의 이미지 8장 생성 |
| Metadata | Sample별 생성 조건 CSV 저장 |
| Automatic QA | PASS 7개 / FAIL 1개 |
| 실패 사례 | `sample_0206`에서 머스터드병 누락 |
| Domain Gap | 배경·조명·주변 물체·촬영 구도의 차이 확인 |

### ③ 오늘 배운 내용을 정리합니다

**Q1. 3D Rendering에서는 왜 BBox를 자동으로 생성할 수 있나요?**

3D 모델의 위치와 카메라 정보를 알고 있으므로, 렌더링된 이미지에서 객체가 차지하는 영역을 계산하여 BBox를 자동으로 만들 수 있습니다.

**Q2. Semantic Mask와 Instance Mask의 차이는 무엇인가요?**

Semantic Mask는 객체의 종류(Class)를 구분하고, Instance Mask는 개별 객체를 구분합니다.

**Q3. 실제 사진과 합성 이미지에서 가장 큰 차이는 무엇이었나요?**

실제 사진에는 복잡한 배경과 다양한 물체가 있지만, 합성 이미지는 흰색 바닥과 단순한 조명으로 구성되어 있었습니다.

**Q4. Metadata와 QA를 이용해 어떤 실패를 발견했나요?**

`sample_0206`은 Seed 206으로 생성되었으며, 카메라 거리 설정값은 약 `0.689`, 조명 밝기는 약 `642.61`이었습니다.

이 Sample에서는 머스터드병이 화면에 나타나지 않아 COCO Annotation도 누락되었습니다.

다만 어떤 설정값이 직접적인 원인인지는 추가 분석이 필요합니다.

### ④ 4일차 최종 마무리

이번 실습에서는 준비된 3D 모델을 이용해 합성 이미지와 AI 학습용 정답 데이터를 자동으로 생성하는 전체 과정을 경험했습니다.

또한 **데이터를 많이 생성하는 것만큼 생성 결과를 검사하고, 실패 원인을 추적하며, 실제 환경과의 차이를 개선하는 것이 중요하다**는 점을 확인했습니다.

이제 Mini Challenge에서는 새로운 3D 객체인 전동드릴을 이용해 같은 과정을 직접 반복하고, 카메라 거리 변경이 합성 이미지에 어떤 영향을 미치는지 비교하겠습니다.

---

# PART 7. Mini Challenge — `power_drill`로 4일차 전체 흐름을 스스로 반복합니다

> **도전 질문:** 전동드릴을 같은 조건에서 촬영하되 **카메라 거리만 멀어지게 변경**하면, 이미지에서 보이는 드릴의 크기와 COCO BBox의 면적 비율은 어떻게 달라질까요?

이제부터는 강사와 함께 진행한 `main` 실습을 새로운 객체에 스스로 적용합니다. 새로운 알고리즘을 처음부터 개발하지 않습니다. **4일차에 이미 작성한 `scripts/00_check_day04.py`부터 `09_compare_experiments.py`까지의 흐름을 재사용**합니다.

**이번 Challenge의 실제 결과는 아직 생성하지 않았습니다.** 아래 명령어와 표의 값은 실습 조건과 확인 방법이며, PASS 수나 BBox 통계는 실행 결과를 보고 학생이 직접 기록합니다.

---

# 36. Mini Challenge의 목표와 전체 순서를 이해합니다

본 실습에서는 과자 상자와 머스터드병을 사용했습니다.

```text
main → cracker_box (Class 1) + mustard_bottle (Class 2)
```

Mini Challenge에서는 전동드릴 한 종류만 사용합니다.

```text
challenge → power_drill (Class 3, BOP ID 15)
```

**이제 해야 할 일은 다음과 같습니다.**

```text
전동드릴 입력 데이터 확인
        ↓
실제 전동드릴 사진 10장 살펴보기
        ↓
Single Rendering 1장
        ↓
RGB · Depth · Semantic · Instance · COCO 검사
        ↓
Randomized Smoke Test 1장
        ↓
Camera Baseline 6장 생성 → Metadata · QA
        ↓
Camera Far 6장 생성 → Metadata · QA
        ↓
두 실험의 동일 조건 확인
        ↓
같은 Seed끼리 BBox 크기 비교
        ↓
Real vs Synthetic · Human QA
        ↓
실패 사례와 Domain Gap 분석
        ↓
challenge_notes.md에 근거와 결론 기록
```

이번에는 **카메라 거리 하나만 의도적으로 변경**합니다. 객체 위치·회전, 조명, 카메라의 나머지 위치를 무작위로 정하되, **두 실험에 같은 Seed를 사용**하여 그 무작위 조건을 동일하게 유지합니다. 카메라가 드릴을 바라보는 *방향*은 카메라 거리에 따라 다시 계산됩니다.

---

# 37. Challenge 입력 데이터와 BlenderProc 실행환경을 확인합니다

## 이 단계에서는 무엇을 하나요?

Main에서 실제로 경험했던 오류를 다시 반복하지 않도록, **파일이 준비되었는지와 EXR 읽기 설정이 적용되었는지** 먼저 점검합니다.

### ① 프로젝트와 가상환경을 확인합니다

4일차 프로젝트 폴더에서 실행합니다. 이미 가상환경이 활성화되어 있으면 다시 활성화하지 않아도 됩니다.

```bash
pwd
source ../.venv/bin/activate
export OPENCV_IO_ENABLE_OPENEXR=1
echo "$OPENCV_IO_ENABLE_OPENEXR"
```

마지막 명령어가 `1`을 출력하면 현재 터미널에서 OpenEXR 읽기가 활성화된 것입니다. 새 터미널을 열면 이 설정을 다시 확인합니다.

> **중요:** 본 실습의 18~21번에서 수정했던 BlenderProc의 ImageIO·NumPy·EXR 읽기 호환성 코드를 **같은 설치환경에서 그대로 사용**합니다. Challenge마다 내부 파일을 다시 수정하는 과정은 아닙니다. BlenderProc를 재설치했다면 먼저 본 실습의 환경 검증을 다시 해야 합니다.

### ② 전동드릴 모델과 텍스처를 확인합니다

```bash
ls -lh data/day04_student_data/challenge/bop/ycbv/models/obj_000015.ply
ls -lh data/day04_student_data/challenge/bop/ycbv/models/obj_000015.png
```

- `.ply`: 전동드릴의 **입체 형태**
- `.png`: 해당 모델의 **표면 무늬·색상 텍스처**

본 실습에서 텍스처 누락으로 렌더링이 중단되었던 것처럼, 이 Challenge 모델도 참조하는 PNG 파일이 있어야 합니다. 둘 중 하나라도 없다면 렌더링하지 말고 제공 데이터의 누락 여부를 확인합니다.

### ③ 제공 데이터 검사 코드를 실행합니다

```bash
python scripts/00_check_day04.py \
  --asset-set challenge
```

확인할 값은 다음과 같습니다.

| 항목 | Challenge 기준 |
|---|---|
| Object | `power_drill` |
| BOP Object ID | `15` |
| Lesson Class ID | `3` |
| 3D Model | `obj_000015.ply` |
| Real Reference | `10장` |

`Day 04 data check: READY`가 출력되면 기본적인 모델과 실제 사진 검사가 끝납니다. 단, **현재 `00_check_day04.py`는 모델이 참조하는 PNG 텍스처까지 검사하지 않으므로** 위의 ②번 파일 확인을 생략하지 않습니다.

---

# 38. 실제 전동드릴 사진을 확인합니다

## 왜 먼저 실제 사진을 보나요?

나중에 합성 이미지를 평가할 때 **어떤 색상·재질·배경·촬영 각도가 실제 사진에 나타나는지** 비교 기준으로 사용하기 위해서입니다.

```bash
python scripts/01_preview_real_reference.py \
  --asset-set challenge
```

생성 파일:

```text
results/challenge/source/real_reference_grid.jpg
```

다음 명령어로 폴더를 열어 사진을 확인합니다.

```bash
explorer.exe results/challenge/source
```

전동드릴의 형태, 색상, 실제 배경의 복잡함, 주변 물체에 가려진 부분을 눈으로 확인합니다. **이 사진 10장은 렌더링 입력이 아니라 이후 비교에 사용하는 실제 참고 이미지**입니다.

---

# 39. 전동드릴 Single Render와 Smoke Test를 진행합니다

## ① 먼저 고정 조건으로 한 장을 생성합니다

```bash
blenderproc run \
  scripts/02_render_single_scene.py \
  --asset-set challenge \
  --tag single \
  --seed 42
```

생성 결과:

```text
results/challenge/single/
├── 0.hdf5
├── sample_metadata.json
└── coco_data/
    ├── coco_annotations.json
    └── images/000000.jpg
```

HDF5 결과를 화면에 띄웁니다.

```bash
blenderproc vis hdf5 \
  results/challenge/single/0.hdf5
```

각 Figure 창을 닫으면 Viewer가 종료됩니다. 이어서 PNG 미리보기를 만듭니다.

```bash
python scripts/03_export_hdf5_preview.py \
  --input results/challenge/single/0.hdf5 \
  --output results/challenge/single/previews
```

```bash
explorer.exe results/challenge/single/previews
```

**확인할 내용:** RGB에서 전동드릴이 보이는가? Semantic Mask에는 Class 3이 있는가? Instance Mask에 객체 영역이 있는가? Depth는 어두운 단색이 아닌 거리 차이를 표현하는가? COCO에는 전동드릴의 BBox가 존재하는가?

### ② 눈으로 본 결과와 HDF5·COCO의 실제값을 함께 확인합니다

앞선 본 실습에서 이미지가 보이는데도 Depth가 `uint8`의 0·1로 저장되거나 COCO Annotation이 비어 있었던 문제를 경험했습니다. 따라서 **파일이 존재하는지만 검사하지 않고 실제 값까지 확인**합니다.

```bash
python - <<'PY'
import json
import h5py
import numpy as np

root = "results/challenge/single"
with h5py.File(f"{root}/0.hdf5", "r") as f:
    depth = f["depth"][:]
    valid = depth[np.isfinite(depth) & (depth > 0)]
    classes = np.unique(f["class_segmaps"][:])
    instances = np.unique(f["instance_segmaps"][:])
    print("Depth 자료형:", depth.dtype)
    print("서로 다른 유효 Depth 값 수:", len(np.unique(valid)))
    print("Class IDs:", classes)
    print("Instance IDs:", instances)

with open(f"{root}/coco_data/coco_annotations.json", encoding="utf-8") as f:
    coco = json.load(f)

annotations = coco.get("annotations", [])
print("COCO Annotation 수:", len(annotations))
print("COCO Class IDs:", [a.get("category_id") for a in annotations])
print("COCO BBox:", [a.get("bbox") for a in annotations])
PY
```

이번 Challenge에서 기대하는 핵심은 **Depth가 실수형이고 깊이값이 다양하며, Class 3이 존재하고, 화면에 보이는 전동드릴의 COCO Annotation이 1개 이상 저장**되는 것입니다. Single Scene에 전동드릴 객체는 하나이므로 보통 Annotation도 1개가 예상됩니다. 다만 이 결과는 실제 실행으로 확인해야 합니다.

> **문제가 생긴다면:** Depth가 `uint8`이거나 0·1만 나온다면 본수업 21번의 EXR 읽기 수정을 다시 확인합니다. COCO Annotation이 0개라면 이미지·Mask를 확인하고, 20번의 Segmentation 수정이 적용되었는지 점검합니다. **이 검사를 통과하기 전에는 12장을 생성하지 않습니다.**

## ③ 새로운 Randomization 코드도 한 장만 먼저 테스트합니다

고정 조건의 Single Render와 무작위 조건의 Render는 서로 다른 프로그램입니다. 따라서 본 실습 25번과 마찬가지로 Randomization 코드도 먼저 한 번 실행합니다.

```bash
blenderproc run \
  scripts/04_render_random_sample.py \
  --asset-set challenge \
  --seed 500 \
  --output results/challenge/smoke/sample_0500 \
  --camera-distance-min 0.65 \
  --camera-distance-max 0.85 \
  --light-min 350 \
  --light-max 800 \
  --rotation-max-deg 25
```

```bash
blenderproc vis hdf5 \
  results/challenge/smoke/sample_0500/0.hdf5
```

RGB·Depth·Mask가 정상인지 살펴보고, 다음 명령어로 COCO 정답이 있는지도 확인합니다.

```bash
python - <<'PY'
import json
from pathlib import Path
p = Path("results/challenge/smoke/sample_0500/coco_data/coco_annotations.json")
coco = json.loads(p.read_text(encoding="utf-8"))
anns = coco.get("annotations", [])
print("Annotation 수:", len(anns))
print("Class IDs:", [a.get("category_id") for a in anns])
PY
```

결과가 정상이라면 다음 배치 생성으로 넘어갑니다. 객체가 프레임 바깥으로 잘리는 경우에는 QA에 기록할 수 있지만, **정답 데이터가 비어 있는 상황을 정상으로 간주하지 않습니다.**

---

# 40. 첫 번째 실험 — Camera Baseline 6장을 생성합니다

## 이번에는 무엇을 하나요?

전동드릴을 **기준 카메라 거리 0.65~0.85**에서 6장 생성합니다. 26~28번과 동일한 Launcher·Metadata·QA 코드를 재사용합니다.

| 항목 | 설정 |
|---|---|
| 실험 이름 | `camera_baseline` |
| 객체 | `power_drill` (Class 3) |
| 생성 수량 | 6장 |
| Seed | 500~505 |
| Camera Distance | 0.65~0.85 |
| Light Energy | 350~800 |
| Z축 회전 | ±25° |

### ① 6장 생성

```bash
python scripts/05_generate_dataset.py \
  --asset-set challenge \
  --tag camera_baseline \
  --count 6 \
  --seed-start 500 \
  --camera-distance-min 0.65 \
  --camera-distance-max 0.85 \
  --light-min 350 \
  --light-max 800 \
  --rotation-max-deg 25
```

`results/challenge/camera_baseline/sample_0500/`부터 `sample_0505/`까지 생성됩니다. 각 Sample에는 HDF5·Metadata·COCO가 저장됩니다.

### ② Metadata 수집

```bash
python scripts/06_collect_metadata.py \
  --asset-set challenge \
  --tag camera_baseline
```

`reports/challenge/camera_baseline_metadata.csv`에서 실제 적용된 카메라 거리·조명·객체 위치를 확인합니다. **Seed만 기록된 계획 CSV와, 실제 사용한 수치를 기록한 Metadata CSV는 서로 다릅니다.**

### ③ Automatic QA

```bash
python scripts/07_qa_3d_dataset.py \
  --asset-set challenge \
  --tag camera_baseline
```

결과는 `reports/challenge/camera_baseline_qa.csv`에 저장됩니다. 전체 6개 중 PASS가 몇 개이고 FAIL이 몇 개인지 **실제 출력으로 확인하여 기록**합니다. 자동 QA는 파일·Class·BBox 등을 검사하지만 Depth 값의 수치적 타당성이나 이미지 현실성까지 완전히 검사하지는 않습니다.

---

# 41. 두 번째 실험 — Camera Distance만 멀리 변경합니다

## 왜 같은 Seed를 사용하나요?

이번에는 카메라 거리만 **0.95~1.15**로 변경합니다. Seed는 앞의 실험과 동일하게 **500~505**를 사용합니다.

```text
                     Baseline           Far
Seed                 500~505            500~505    ← 동일
Object 위치·회전      같은 난수          같은 난수  ← 동일
Light 위치·밝기       같은 난수          같은 난수  ← 동일
Camera X·Z           같은 난수          같은 난수  ← 동일
Camera Distance      0.65~0.85          0.95~1.15  ← 변경
```

카메라가 멀어지면 객체를 계속 바라보도록 카메라 방향도 다시 계산됩니다. **이것은 거리 변경에 따른 결과**이지 별도로 Randomization 변수를 추가한 것이 아닙니다.

### ① Far 이미지 6장 생성

```bash
python scripts/05_generate_dataset.py \
  --asset-set challenge \
  --tag camera_far \
  --count 6 \
  --seed-start 500 \
  --camera-distance-min 0.95 \
  --camera-distance-max 1.15 \
  --light-min 350 \
  --light-max 800 \
  --rotation-max-deg 25
```

### ② Far Metadata 수집

```bash
python scripts/06_collect_metadata.py \
  --asset-set challenge \
  --tag camera_far
```

### ③ Far Automatic QA

```bash
python scripts/07_qa_3d_dataset.py \
  --asset-set challenge \
  --tag camera_far
```

이제 아래 두 CSV에 각각 6개의 QA 결과가 기록됩니다.

```text
reports/challenge/camera_baseline_qa.csv
reports/challenge/camera_far_qa.csv
```

### ④ 두 실험의 FAIL을 함께 확인합니다

```bash
python - <<'PY'
from pathlib import Path
import pandas as pd

for tag in ("camera_baseline", "camera_far"):
    path = Path(f"reports/challenge/{tag}_qa.csv")
    df = pd.read_csv(path)
    failed = df[df["auto_pass"].astype(str).str.lower() == "false"]
    print(f"[{tag}] Total={len(df)} PASS={len(df)-len(failed)} FAIL={len(failed)}")
    if not failed.empty:
        print(failed[["sample", "reason"]].to_string(index=False))
PY
```

- `missing_annotation`: 카메라에서 전동드릴이 보이지 않아 정답이 누락되었을 가능성 등이 있습니다.
- `class_id_mismatch`: 예상 Class 3이 기록되지 않은 상황입니다.
- `invalid_bbox`: BBox가 너무 작거나 이미지 범위를 벗어난 경우입니다.

**오류가 발생해도 결과를 바로 삭제하지 않습니다.** 해당 Sample의 `000000.jpg`, COCO JSON, Metadata를 확인하고 실패 원인을 `challenge_notes.md`에 기록합니다. 본 실습에서 `sample_0206`을 분석했던 것과 같은 방법입니다.

> **기존 결과 보호:** `05_generate_dataset.py`는 같은 결과 폴더가 이미 있으면 중단합니다. `--overwrite`는 해당 결과 폴더를 삭제하므로 무조건 사용하지 않습니다. 또한 `07_qa_3d_dataset.py`의 `--overwrite`는 사람이 적은 Human QA 기록을 덮어쓸 수 있으므로, **기록을 시작한 뒤에는 재실행하지 않습니다.**

---

# 42. 두 카메라 조건의 BBox 크기를 수치로 비교합니다

## 이 코드는 무엇을 하나요?

카메라를 멀리하면 전동드릴이 작게 보일 것이라고 예상할 수 있습니다. 하지만 눈으로만 판단하지 않고 **BBox가 512×512 이미지에서 차지하는 면적 비율**을 비교해 보겠습니다.

```text
BBox = [x, y, width, height]
                ↓
BBox 면적 = width × height
                ↓
BBox Area Ratio = BBox 면적 / (512 × 512)
```

예를 들어 면적 비율이 `0.10`이라면 BBox가 이미지 면적의 약 10%를 차지한다는 뜻입니다. **비율이 작을수록 화면에서 BBox가 차지하는 면적이 작습니다.** 단, 물체가 프레임 경계에서 잘리면 이 비율도 영향을 받습니다.

### ① 어떤 Sample을 비교해야 공정한가요?

이번 실습의 핵심은 **같은 Seed로 생성된 두 Sample을 짝지어 비교**하는 것입니다.

```text
Baseline sample_0500 ↔ Far sample_0500
Baseline sample_0501 ↔ Far sample_0501
...
Baseline sample_0505 ↔ Far sample_0505
```

단, 한쪽에서 드릴이 화면에 보이지 않거나 COCO가 누락되었다면 그 짝은 크기 비교에 사용할 수 없습니다. **두 실험 모두 Automatic QA에 통과한 짝만 통계에 포함**하고, 제외한 Sample은 별도로 기록합니다.

### ② 두 실험을 비교하는 Python 코드를 작성합니다

**파일: `scripts/09_compare_experiments.py` (Mini Challenge 보완 버전)**

이미 4일차 앞부분에서 빈 파일을 만들었다면 아래 내용으로 채웁니다. 기존 파일에 코드가 있다면 백업한 뒤 교체합니다. 이 보완 버전은 기존 비교 코드의 역할을 유지하면서 **QA에 실패한 Sample이 통계에 몰래 포함되는 문제를 방지**합니다.

```python
from __future__ import annotations

import argparse
import json
from pathlib import Path

import numpy as np
import pandas as pd

ROOT = Path(__file__).resolve().parents[1]


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--asset-set", required=True, choices=["main", "challenge"])
    parser.add_argument("--tag-a", required=True)
    parser.add_argument("--tag-b", required=True)
    return parser.parse_args()


def load_metadata(asset_set, tag):
    path = ROOT / "reports" / asset_set / f"{tag}_metadata.csv"
    df = pd.read_csv(path).sort_values("sample").reset_index(drop=True)
    if df.empty or df["sample"].duplicated().any():
        raise RuntimeError(f"Metadata의 Sample 구성을 확인하세요: {path}")
    return df


def check_same_conditions(first, second):
    if not first["sample"].equals(second["sample"]):
        raise RuntimeError("두 실험의 Sample 이름·개수가 다릅니다.")
    if set(first.columns) != set(second.columns):
        raise RuntimeError("두 실험의 Metadata 열 구성이 다릅니다.")

    # 의도적으로 바꾼 거리와, 거리에 따라 결정되는 Camera Y만 제외
    ignored = {
        "camera_distance", "camera_y",
        "camera_distance_min", "camera_distance_max", "approved",
    }
    different = []
    for col in first.columns:
        if col in ignored:
            continue
        left, right = first[col], second[col]
        if pd.api.types.is_numeric_dtype(left) and pd.api.types.is_numeric_dtype(right):
            same = np.allclose(
                left.to_numpy(), right.to_numpy(),
                rtol=1e-9, atol=1e-9, equal_nan=True,
            )
        else:
            same = left.astype(str).equals(right.astype(str))
        if not same:
            different.append(col)
    if different:
        raise RuntimeError("Camera Distance 외에 달라진 항목: " + ", ".join(different))


def load_pass_samples(asset_set, tag):
    path = ROOT / "reports" / asset_set / f"{tag}_qa.csv"
    qa = pd.read_csv(path)
    if qa.empty or qa["sample"].duplicated().any():
        raise RuntimeError(f"QA Sample 구성을 확인하세요: {path}")
    passed = qa["auto_pass"].astype(str).str.strip().str.lower().eq("true")
    return set(qa.loc[passed, "sample"])


def bbox_ratio(asset_set, tag, sample):
    path = (
        ROOT / "results" / asset_set / tag / sample
        / "coco_data" / "coco_annotations.json"
    )
    coco = json.loads(path.read_text(encoding="utf-8"))
    anns = coco.get("annotations", [])
    expected_ids = {"main": {1, 2}, "challenge": {3}}[asset_set]

    if len(anns) != len(expected_ids):
        raise ValueError(f"{tag}/{sample}: Annotation 개수가 예상과 다릅니다.")
    if {int(a["category_id"]) for a in anns} != expected_ids:
        raise ValueError(f"{tag}/{sample}: Class ID가 예상과 다릅니다.")

    images = {int(im["id"]): im for im in coco.get("images", [])}
    ratios = []
    for ann in anns:
        im = images[int(ann["image_id"])]
        x, y, w, h = [float(v) for v in ann["bbox"]]
        iw, ih = float(im["width"]), float(im["height"])
        if not (np.isfinite([x, y, w, h]).all()
                and w > 1 and h > 1 and x >= 0 and y >= 0
                and x + w <= iw and y + h <= ih):
            raise ValueError(f"{tag}/{sample}: BBox 값이 잘못되었습니다.")
        ratios.append((int(ann["category_id"]), (w * h) / (iw * ih)))
    return ratios


def main():
    args = parse_args()
    a = load_metadata(args.asset_set, args.tag_a)
    b = load_metadata(args.asset_set, args.tag_b)
    check_same_conditions(a, b)
    print("Non-distance conditions: SAME")

    common = load_pass_samples(args.asset_set, args.tag_a) & load_pass_samples(
        args.asset_set, args.tag_b
    )
    all_samples = set(a["sample"])
    selected = sorted(common & all_samples)
    excluded = sorted(all_samples - set(selected))
    print(f"Valid paired samples: {len(selected)} / {len(all_samples)}")
    print("Excluded samples:", excluded if excluded else "없음")
    if not selected:
        raise RuntimeError("두 조건 모두 QA PASS인 Sample이 없습니다.")

    rows = []
    for sample in selected:
        for tag in (args.tag_a, args.tag_b):
            for class_id, ratio in bbox_ratio(args.asset_set, tag, sample):
                rows.append({
                    "tag": tag, "sample": sample,
                    "category_id": class_id, "bbox_area_ratio": ratio,
                })

    detail = pd.DataFrame(rows)
    summary = (
        detail.groupby("tag")["bbox_area_ratio"]
        .agg(["count", "mean", "median", "min", "max"])
        .reset_index()
    )

    report_dir = ROOT / "reports" / args.asset_set
    report_dir.mkdir(parents=True, exist_ok=True)
    prefix = f"{args.tag_a}_vs_{args.tag_b}"
    detail_path = report_dir / f"{prefix}_bbox_detail.csv"
    summary_path = report_dir / f"{prefix}_bbox_summary.csv"
    detail.to_csv(detail_path, index=False, encoding="utf-8-sig")
    summary.to_csv(summary_path, index=False, encoding="utf-8-sig")
    print(summary.to_string(index=False))
    print("Detail :", detail_path)
    print("Summary:", summary_path)


if __name__ == "__main__":
    main()
```

### ③ 코드 실행 흐름을 의사코드로 이해합니다

```text
Baseline / Far Metadata CSV 열기
        ↓
같은 Sample·Seed인지 확인
        ↓
카메라 거리 외 조건이 같은지 확인
        ├─ 다름 → 오류 출력 후 비교 중단
        └─ 같음 → SAME 출력
        ↓
두 실험의 Automatic QA CSV 열기
        ↓
같은 Seed에서 둘 다 PASS한 Sample만 선택
        ↓
각 Sample의 COCO BBox 읽기
        ↓
BBox Area Ratio 계산
        ↓
실험별 평균·중앙값·최솟값·최댓값 계산
        ↓
Detail CSV + Summary CSV 저장
```

### ④ 비교 프로그램을 실행합니다

```bash
python scripts/09_compare_experiments.py \
  --asset-set challenge \
  --tag-a camera_baseline \
  --tag-b camera_far
```

먼저 출력되어야 하는 핵심 메시지:

```text
Non-distance conditions: SAME
```

이 문장은 **카메라 거리 관련 값 외에 비교 가능한 생성 조건이 같았다**는 뜻입니다. 이어서 `Valid paired samples`를 확인합니다. 두 실험 모두 PASS한 짝이 6개라면 6개를, 일부 FAIL이 있으면 남은 짝만 사용합니다.

생성 파일:

```text
reports/challenge/camera_baseline_vs_camera_far_bbox_detail.csv
reports/challenge/camera_baseline_vs_camera_far_bbox_summary.csv
```

| 항목 | Camera Baseline | Camera Far |
|---|---|---|
| 비교에 사용된 유효 Sample 쌍 수 | 실행 후 기록 | 실행 후 기록 |
| 평균 BBox Area Ratio | 실행 후 기록 | 실행 후 기록 |
| 중앙값 BBox Area Ratio | 실행 후 기록 | 실행 후 기록 |
| 가장 눈에 띄는 차이 | 실행 후 기록 | 실행 후 기록 |

**예상과 실제 결과를 구분하세요.** 일반적으로 먼 카메라에서 드릴이 더 작게 보일 가능성이 있지만, 비교 결과를 보기 전에 평균값이 얼마일지 단정하지 않습니다.

> **분석 주의:** QA PASS는 구조 검사의 통과일 뿐입니다. RGB에서 드릴이 심하게 잘린 Sample을 발견했다면 그 사실도 Human QA에 기록하고, BBox 평균을 해석할 때 주의합니다. 원래 코드처럼 Annotation이 없는 Sample을 조용히 건너뛰어 두 실험의 서로 다른 표본을 비교하면 결론이 왜곡될 수 있으므로, 이번 보완 코드는 같은 Seed의 양쪽 PASS만 비교합니다.

---

# 43. Real vs Synthetic과 두 카메라 실험 이미지를 함께 비교합니다

## ① 실제 사진과 각 실험의 합성 이미지를 비교합니다

본 실습 31번과 같은 프로그램을 다시 사용합니다.

```bash
python scripts/08_compare_real_synthetic.py \
  --asset-set challenge \
  --tag camera_baseline
```

```bash
python scripts/08_compare_real_synthetic.py \
  --asset-set challenge \
  --tag camera_far
```

두 이미지의 위치:

```text
results/challenge/camera_baseline/domain_gap_grid.jpg
results/challenge/camera_far/domain_gap_grid.jpg
```

- 위쪽 줄: 실제 전동드릴 사진(Real Reference) 6장
- 아래쪽 줄: 해당 조건의 합성 전동드릴 이미지 6장

**드릴의 화면 크기, 실제 촬영 각도와의 차이, 단순한 바닥·배경, 조명과 그림자, 잘림 현상**을 살펴봅니다. 이 코드는 QA PASS만 자동으로 선별하지 않으므로, 이미지 비교 시 FAIL Sample 여부도 함께 확인합니다.

## ② Baseline과 Far를 같은 Seed끼리 한 화면에 모읍니다

각 Sample을 하나씩 열지 않고도 카메라 거리의 영향을 비교할 수 있도록 **위쪽 6장은 Baseline, 아래쪽 6장은 Far**로 배치합니다.

```bash
python - <<'PY'
from pathlib import Path
from PIL import Image, ImageDraw, ImageOps

root = Path("results/challenge")
size, cols, label_h = 200, 6, 30
canvas = Image.new("RGB", (cols * size, 2 * (size + label_h)), "white")
draw = ImageDraw.Draw(canvas)

for row, tag in enumerate(("camera_baseline", "camera_far")):
    for col, seed in enumerate(range(500, 506)):
        path = root / tag / f"sample_{seed:04d}" / "coco_data/images/000000.jpg"
        if not path.exists():
            raise FileNotFoundError(path)
        with Image.open(path) as image:
            tile = ImageOps.contain(image.convert("RGB"), (size - 8, size - 8))
        x = col * size + (size - tile.width) // 2
        y = row * (size + label_h) + (size - tile.height) // 2
        canvas.paste(tile, (x, y))
        draw.text((col * size + 5, row * (size + label_h) + size + 3),
                  f"{tag} / {seed}", fill="black")

output = root / "camera_baseline_vs_far_grid.jpg"
canvas.save(output, quality=94)
print("Saved:", output)
PY
```

```bash
explorer.exe results/challenge
```

`camera_baseline_vs_far_grid.jpg`를 열면 **같은 Seed의 두 이미지를 위아래로 바로 비교**할 수 있습니다.

### ③ Human QA를 기록합니다

자동 검사 CSV를 열고 각 Sample의 `human_decision`, `human_note`를 작성합니다.

```text
reports/challenge/camera_baseline_qa.csv
reports/challenge/camera_far_qa.csv
```

- `approved`: 눈으로 확인한 기준까지 충족
- `needs_review`: 거리·각도·그림자·배경 등을 더 살펴봐야 함
- `rejected`: 드릴 누락 등으로 학습에 쓰기 부적절

모든 Sample을 억지로 `approved`로 바꿀 필요는 없습니다. **실패를 찾아낸 것도 중요한 Challenge 결과**입니다. CSV를 편집한 뒤에는 Automatic QA 명령을 다시 실행하지 않습니다.

다음 표를 학생의 실제 관찰로 완성합니다.

| 비교 항목 | Camera Baseline | Camera Far |
|---|---|---|
| 객체 화면 크기 | 직접 관찰 | 직접 관찰 |
| 평균 BBox Area Ratio | 통계 CSV 확인 | 통계 CSV 확인 |
| 객체 누락·잘림 여부 | QA + RGB 확인 | QA + RGB 확인 |
| 실제 Reference와의 크기 유사성 | 직접 관찰 | 직접 관찰 |
| 배경·조명의 Domain Gap | 직접 관찰 | 직접 관찰 |
| 선택 판단 근거 | 기록 | 기록 |

---

# 44. 결과 보고서 `challenge_notes.md`를 작성합니다

**파일: `reports/challenge/challenge_notes.md`**

아래 양식에서 **실제로 확인한 숫자와 사실**을 작성합니다. 본 실습의 `PASS 7 / FAIL 1`, `sample_0206`은 Main에서 나온 결과이므로 Challenge의 결과로 복사하면 안 됩니다.

```markdown
# Day 04 Mini Challenge — power_drill

## 1. 입력 데이터
- 객체 이름: power_drill
- BOP Object ID: 15
- Lesson Class ID: 3
- 3D 모델 / 텍스처 확인: 
- Real Reference: 10장

## 2. Single / Smoke Test
- Single RGB 화면에 드릴이 보이는가?:
- Depth 자료형 / 유효 고유값 수:
- Semantic Class ID:
- Instance 구분 결과:
- COCO Annotation 수 / BBox:
- Randomized Smoke Test 결과:

## 3. 실험 조건
| 항목 | Baseline | Far |
|---|---|---|
| Seed | 500~505 | 500~505 |
| 생성 수 | 6 | 6 |
| Camera Distance | 0.65~0.85 | 0.95~1.15 |
| Light Energy | 350~800 | 350~800 |
| 회전 범위 | ±25° | ±25° |

## 4. 자동 검사와 Metadata
- Baseline Automatic QA PASS / FAIL:
- Far Automatic QA PASS / FAIL:
- Non-distance conditions SAME 여부:
- 두 실험 모두 PASS한 Sample 쌍 수:
- 비교에서 제외한 Sample과 이유:

## 5. BBox 크기 비교
- Baseline 평균 BBox Area Ratio:
- Far 평균 BBox Area Ratio:
- 같은 Seed에서 드릴 크기가 어떻게 달라졌는가?:

## 6. Real vs Synthetic / Human QA
- 실제 사진과 가장 큰 차이:
- 카메라 거리 외에 남아 있는 Domain Gap:
- needs_review / rejected로 판단한 Sample과 이유:

## 7. 실패 또는 잠재 위험 분석
- 문제가 발생한 Sample(있다면):
- RGB · COCO · Metadata에서 확인한 증거:
- 원인에 대한 가설(확인 사실과 구분):
- 개선하고 싶은 설정:
- FAIL이 없었다면, 앞으로 발생할 수 있는 위험 한 가지:

## 8. 최종 결론
- 두 카메라 조건 중 선택한 조건:
- 선택한 이유(숫자 + 이미지 관찰):
- 3D Rendering에서 Ground Truth를 자동 생성할 수 있는 이유:
- Automatic QA만으로 충분하지 않은 이유:
```

**예시를 그대로 답으로 쓰지 마세요.** 학생마다 생성 결과와 QA 상태가 다를 수 있으므로 자신이 실제로 얻은 CSV와 이미지를 근거로 작성합니다.

---

# 45. Mini Challenge 완료 조건을 확인합니다

- [ ] `power_drill`의 BOP ID 15, Lesson Class ID 3을 구분했다.
- [ ] 전동드릴 PLY·PNG 텍스처·실제 사진 10장을 확인했다.
- [ ] OpenEXR 설정과 기존 BlenderProc 호환성 수정 상태를 확인했다.
- [ ] Single Render에서 RGB·Depth·Mask·COCO를 확인했다.
- [ ] Depth 자료형과 Class ID, COCO Annotation의 **실제값**을 검사했다.
- [ ] Randomized Smoke Test 한 장을 통과했다.
- [ ] Baseline 6장과 Far 6장을 생성했다.
- [ ] 두 실험의 Metadata CSV를 만들었다.
- [ ] 두 실험의 Automatic QA 결과를 확인하고 FAIL 원인을 조사했다.
- [ ] 같은 Seed를 사용하고 Camera Distance 외 조건 SAME을 검사했다.
- [ ] **양쪽 모두 QA PASS인 Sample 쌍**의 BBox 면적 비율을 비교했다.
- [ ] 두 조건의 합성 이미지를 한 화면에 모아 확인했다.
- [ ] 실제 사진과 Synthetic 이미지의 Domain Gap을 비교했다.
- [ ] Human QA 결과를 CSV에 기록하고 기존 결과를 보호했다.
- [ ] 실제 FAIL이 있었다면 분석했고, 없었다면 잠재 위험을 작성했다.
- [ ] 최종 카메라 거리 선택을 실제 결과로 설명했다.
- [ ] `reports/challenge/challenge_notes.md`를 완성했다.

---

# 46. 약 3시간 동안 Mini Challenge를 진행합니다

| 단계 | 권장 시간 |
|---|---:|
| 입력 데이터·실행환경·Real Preview 확인 | 20분 |
| Single Render·Depth/Mask/COCO 검사 | 25분 |
| Randomized Smoke Test | 15분 |
| Camera Baseline 6장 및 Metadata·QA | 25분 |
| Camera Far 6장 및 Metadata·QA | 25분 |
| 실패 데이터 확인·Camera 조건 SAME 검사 | 20분 |
| BBox 비교·이미지 Grid 생성 | 25분 |
| Real vs Synthetic·Human QA | 15분 |
| `challenge_notes.md`·완료 체크 | 10분 |
| **합계** | **180분** |

렌더링에 예상보다 시간이 오래 걸린다면 **먼저 1장·2장씩 실행을 검증한 뒤** 다음 단계로 진행하고, 시간 부족을 숨기기 위해 PASS 수나 BBox 수치를 임의로 작성하지 않습니다.

### 이 Challenge를 통해 무엇을 배웠나요?

```text
전동드릴 3D Model
        ↓
합성 이미지·정답 데이터 자동 생성
        ↓
Randomization — 카메라 거리만 변경
        ↓
Metadata로 조건 재현·비교
        ↓
Automatic QA → 실패 데이터 찾기
        ↓
Real vs Synthetic / Human QA
        ↓
숫자와 관찰을 근거로 개선 방향 결정
```

**중요한 것은 이미지를 많이 생성한 사실보다, 어떤 조건에서 만들었고 어느 부분이 정상인지·문제인지 설명할 수 있는 능력입니다.** 다음 PART 8에서는 Challenge에서 완성한 파일과 보고서를 README·Git 작업에 연결합니다.


---

# PART 8. 4일차 결과를 기록하고 버전관리합니다

앞에서 Main 실습과 Mini Challenge의 생성·검증 방법을 배웠습니다. 이제 **결과 기록 → README 정리 → Git 저장 → 완료 확인** 순서로 하루를 마무리합니다. 앞에서 실행한 렌더링이나 QA를 다시 반복하지 않습니다.

---

# 47. `day04_notes.md`에 실제 결과를 기록합니다

**파일: `day04_notes.md`**

앞에서 생성한 Metadata와 QA 결과를 다시 길게 복사하기보다는 **실험 조건·대표 결과·실패 사례·개선 방향**만 남깁니다.

- **Main:** 과자 상자(Class 1)·머스터드병(Class 2), Seed 200~207, 8장 생성
- **Automatic QA:** PASS 7 / FAIL 1
- **실패 원인 확인:** `sample_0206`에서 머스터드병이 RGB에 보이지 않고 COCO에는 과자 상자 BBox만 기록됨
- **Metadata 예시:** `sample_0203` — Camera Distance 약 0.719, Light Energy 약 698.77
- **Domain Gap:** 실제 사진보다 배경·조명이 단순하고 주변 물체·가림·노이즈가 부족함
- **Human QA:** 최종 판정은 실제로 작성한 결과만 기록
- **Mini Challenge:** 전동드릴의 Baseline/Far 결과는 직접 실행한 뒤 작성

실제 Main 결과를 반영한 별도 템플릿 **`day04_notes_본수업실제결과_학생완성형.md`**을 참고해 프로젝트의 `day04_notes.md`로 저장합니다. 학생마다 다른 Challenge 결과를 임의로 채우지 않습니다.

---

# 48. README에 수업 구조와 재실행 방법을 정리합니다

**파일: `README.md`**

README는 보고서가 아니라 **다른 사람이 프로젝트를 열고 실습 목적·실행 순서·결과 위치를 이해하도록 돕는 안내서**입니다.

반드시 포함할 내용은 다음과 같습니다.

1. Main / Challenge 데이터 및 Class ID
2. 3D Rendering → Randomization → Metadata → QA → Real/Synthetic 흐름
3. 주요 스크립트의 역할과 실행 명령어
4. 실제 Main 결과: 8장, PASS 7 / FAIL 1, `sample_0206` 누락 사례
5. Mini Challenge의 카메라 거리 비교 방법(결과 수치는 직접 기입)
6. `results/`는 Git에 업로드하지 않는다는 안내와 5일차 연결

완성된 별도 **`README_교과11_4일차_최종.md`**를 프로젝트의 `README.md`로 저장합니다.

---

# 49. `.gitignore`로 대용량 데이터를 제외합니다

**파일: `.gitignore`**

```gitignore
# Python / Editor
__pycache__/
*.pyc
.vscode/

# 제공 데이터와 생성된 이미지·렌더링 결과
data/day04_student_data/
results/
*.ply
*.hdf5
*.exr

# OS
.DS_Store
Thumbs.db
```

`results/`와 제공 원본 데이터는 Git에 올리지 않습니다. **스크립트·README·실습 노트·작은 보고서 CSV**를 중심으로 기록합니다. CSV에 민감한 정보가 있는지는 업로드 전에 확인합니다.

---

# 50. 기존 Git 저장소에 4일차 작업을 기록합니다

프로젝트의 `day04_3d_segmentation` 폴더에서 실행합니다. **새 Git 저장소는 만들지 않습니다.**

```bash
# 현재 위치와 기존 Git 저장소 확인
pwd
git rev-parse --show-toplevel
git status

# 현재 4일차 폴더의 허용된 파일만 스테이징
git add .

# 실제로 올라갈 파일 목록 확인
git diff --cached --name-only

# 파일을 확인한 뒤 Commit
git commit -m "docs: finalize day04 blenderproc lab and QA"

git push
```

`git add .`은 **현재 디렉터리 아래**를 대상으로 합니다. 반드시 4일차 폴더에 있는지 확인하세요. 이미 Git에서 추적 중인 대용량 파일은 `.gitignore`만으로 제외되지 않으므로 Staging 목록을 확인합니다.

---

# 51. 필수 항목만 자가 체크합니다

- [ ] Main 합성데이터 8장과 Depth·Mask·COCO 정답을 확인했다.
- [ ] Main QA PASS 7 / FAIL 1과 `sample_0206` 누락 이유를 설명할 수 있다.
- [ ] Metadata에서 생성 조건을 확인하고 Domain Gap을 정리했다.
- [ ] Mini Challenge의 Single/Smoke Test, Baseline/Far 생성·QA를 수행했다.
- [ ] **같은 Seed·비거리 조건 동일 여부를 검증**하고, 양쪽 QA PASS인 쌍으로 BBox를 비교했다.
- [ ] Human QA 및 `challenge_notes.md`에 실제 결과를 기록했다.
- [ ] `day04_notes.md`와 `README.md`를 작성했다.
- [ ] Git에 대용량 파일이 포함되지 않은 것을 확인하고 Commit했다.

*Mini Challenge를 아직 수행하지 않았다면 체크하지 않고 나중에 완료합니다.*

---

# 52. 4일차를 마무리하고 5일차로 연결합니다

**오늘의 결론:** 3D 모델을 렌더링하면 RGB·BBox·Mask·Depth를 자동 생성할 수 있습니다. 하지만 `sample_0206`처럼 **객체가 화면에 없거나 정답이 누락되는 경우**도 있으므로, `Metadata → Automatic QA → Human QA → Real/Synthetic 비교`를 통해 실제 활용 가능성을 판단해야 합니다.

