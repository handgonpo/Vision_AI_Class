## 오늘의 목표

1일차에서는 검사할 위치인 ROI를 정했습니다.

2일차에서는 ROI의 Brightness, Contrast, Noise, Blur와 같은 영상 품질을 확인하고 필요한 Image Enhancement를 선택한 뒤 Threshold / Otsu로 객체와 배경을 Binary Image로 분리했습니다.

3일차에서는 Binary Image에 Morphology와 Contour를 적용하여 `Area`, `Area Ratio`, `Count`, `BBox`, `Center` 같은 Feature를 숫자로 측정하고, 병뚜껑 있음 / 없음의 실제 측정값을 비교하여 Candidate Rule과 OK / NG 판정을 만들어 보았습니다.

4일차에서는 다음 질문으로 넘어갑니다.

> 면적이나 개수만으로는 구분하기 어려운 라벨·로고·부품을 기준 이미지와 어떻게 비교할 것인가?

오늘은 제품 라벨·로고처럼 모양과 무늬 자체가 중요한 검사 문제를 사용합니다.

오늘 수업이 끝나면 다음을 할 수 있어야 합니다.

```text
- 1~3일차의 ROI → 전처리 → 측정 → Rule 흐름과 4일차의 이미지 비교 흐름을 연결할 수 있다.
- Area·Count 같은 기하 Feature만으로 해결하기 어려운 검사 문제를 설명할 수 있다.
- GrabCut과 MeanShift의 역할과 한계를 구분할 수 있다.
- GrabCut으로 전경과 배경을 분리하고 초기 Rectangle 변화에 따른 결과를 비교할 수 있다.
- pyrMeanShiftFiltering의 sp·sr을 바꾸어 색상 영역 단순화 결과를 비교할 수 있다.
- Template과 Target의 의미를 설명할 수 있다.
- Template Matching으로 기준 라벨의 유사도 Score와 위치를 측정할 수 있다.
- Template Matching이 이동에는 대응하지만 회전·크기 변화에는 약해질 수 있음을 확인할 수 있다.
- Keypoint와 Descriptor의 차이를 설명할 수 있다.
- ORB 특징점과 Binary Descriptor를 생성하고 Hamming Distance로 매칭할 수 있다.
- SIFT 특징점과 Descriptor를 생성하고 L2 Distance로 매칭할 수 있다.
- Ratio Test가 명확한 대응점만 남기기 위한 과정임을 설명할 수 있다.
- Homography와 RANSAC을 이용하여 회전·이동된 평면 라벨의 위치를 추정할 수 있다.
- 정상·이동·회전·크기 변화 조건에서 Template / ORB / SIFT / Homography 결과를 비교할 수 있다.
- 검사 문제가 달라지면 기술도 달라져야 한다는 것을 설명할 수 있다.
- Mini Challenge에서 사용할 방법을 직접 선택하고 선택 이유와 한계를 기록할 수 있다.
- 오늘 측정한 Score와 Match Count를 최종 판정 기준으로 바로 확정하면 안 되는 이유를 설명할 수 있다.
- 결과 이미지, 비교 CSV, 분석 문서, README, Local Git 기록을 남길 수 있다.
```

---
# PART 1~11 — 4일차 준비 및 비교 데이터 구성

---
# 1. 오늘 수업의 전체 흐름

1~3일차에서는 이미지에서 검사할 영역을 정하고, 영상 상태를 개선한 뒤 객체의 크기·개수·위치를 숫자로 측정하는 방법을 배웠습니다. 

4일차에서는 한 단계 더 나아가 다음 질문을 다룹니다. 면적이나 개수만으로 구분하기 어려운 
라벨·로고를 기준 이미지와 어떻게 비교할 수 있을까요? 

이번에는 광천김 포장지의 로고를 기준으로 같은 제품이라도:

```text
위치가 달라지고 
회전하고 
크기가 달라질 때
```

각 검사 방법의 결과가 어떻게 달라지는지 비교합니다.
## 4 ~ 5일 수업 흐름

![[Pasted image 20261027160950.png]]

## 1.1 4일차 오늘 수업은 다음 순서로 진행합니다

![[Pasted image 20261026155522.png]]

### 4일차 실습은 크게 두 가지로 나뉩니다

| 기술                | 실습 목적                    | 특징점 검출 여부   | 5일차 연결    |
| ----------------- | ------------------------ | ----------- | --------- |
| GrabCut           | 제품과 배경 분리                | X           | 직접 연결 안 함 |
| MeanShift         | 비슷한 색상 영역 단순화            | X           | 직접 연결 안 함 |
| Template Matching | 기준 로고와 유사도 비교            | X           | O         |
| ORB               | 로고의 특징점 검출·매칭            | O           | 비교 학습 중심  |
| SIFT              | 회전·크기 변화에 대응하는 특징점 검출·매칭 | O           | O         |
| Homography        | 매칭된 특징점으로 로고 위치 추정       | SIFT 특징점 활용 | O         |

### 직접적으로 특징점을 찾고 연결하는 것은?

ORB와 SIFT가 특징점을 직접 검출합니다. 특히 5일차와 연결되는 핵심은 `SIFT → Matching → RANSAC → Homography`입니다.

==ORB의 Good Match는
기준 로고와 대상 이미지 사이에서
서로 비슷하다고 판단된 특징점 연결 개수입니다.==

==보통 Good Match가 많을수록
비슷한 특징이 잘 유지되었다고 볼 수 있습니다.==

==하지만 개수가 많다고 해서
항상 로고 위치를 정확하게 찾았다는 뜻은 아닙니다.==

==그래서 ORB는
“특징이 얼마나 비슷한가”를 보는 데 유용하고,
실제 위치를 더 정확히 확인할 때는
Homography 결과도 함께 봐야 합니다.==

### GrabCut과 MeanShift를 왜 배우나요?

| 구분     | GrabCut                | MeanShift                    |
| ------ | ---------------------- | ---------------------------- |
| 배우는 이유 | 필요한 객체만 분리하는 방법 이해     | 복잡한 색상과 작은 변화를 단순화하는 방법 이해   |
| 핵심 질문  | 어디까지가 제품이고 어디부터 배경인가?  | 어떤 픽셀들이 비슷한 색상 영역인가?         |
| 처리 방법  | 전경과 배경을 추정하여 Mask 생성   | 주변 위치와 색상 유사성을 고려해 픽셀 색상 평활화 |
| 결과물    | Binary Mask, 배경 제거 이미지 | 색상 영역이 단순해진 이미지              |
| 대표 활용  | 제품 분리, 객체 영역 측정        | 색상 분석, 영역 구분 전처리             |
| 특징점 검출 | 하지 않음                  | 하지 않음                        |

### 실제 산업현장에서는 어디에 활용하나요?

|실제 검사 상황|적합한 기술|활용 방법|
|---|---|---|
|컨베이어 위 제품만 배경에서 분리|GrabCut|제품 Mask 생성 후 외곽선이나 면적 측정|
|촬영한 제품 사진의 배경 제거|GrabCut|제품만 추출하여 후속 분석|
|제품의 외형이나 크기 검사|GrabCut|추출된 Mask에서 Contour·Area 계산|
|포장재의 색상이 여러 영역으로 나뉜 경우|MeanShift|비슷한 색상 영역을 단순화하여 분석|
|제품 표면에 작은 색상 잡음이 많은 경우|MeanShift|세부 색상 변화를 줄여 주요 영역 확인|
|색상이 다른 부품이나 영역 구분|MeanShift|색상 영역을 단순화한 뒤 Threshold·Contour 등 추가 처리|
|로고의 특징점과 위치 찾기|ORB·SIFT·Homography|기준 로고와 대상 이미지의 특징점 비교|
단, GrabCut은 초기 영역 지정이 필요할 수 있고, MeanShift는 색상을 단순화할 뿐 객체 Mask를 직접 완성하는 것은 아닙니다.

---

## 1.2 같은 데이터를 여러 방법으로 비교합니다

Template Matching, ORB, SIFT, Homography 실습에서는 같은 Reference와 같은 네 개의 Target을 사용합니다.

```
Reference
reference_label.jpg

        ↓

Targets

normal
shift
rot15
scale90
```

한 번에 여러 조건을 바꾸지 않고:

```
위치만 변경
회전만 변경
크기만 변경
```

하여 각 방법이 어떤 변화에 영향을 받는지 확인합니다.

즉 오늘의 목적은:

```
기술 실행
        ↓
측정값 확인
        ↓
조건을 바꿈
        ↓
결과가 어떻게 달라지는지 비교
        ↓
어떤 검사 방법이 적합한지 판단
```

하는 것입니다.

![[Pasted image 20261026160222.png]]

---

## 1.3 오늘 기술을 모두 연결해서 사용하는 것은 아닙니다

오늘 배우는 기술은 하나의 고정 Pipeline이 아닙니다.

다음처럼 무조건 연결하지 않습니다.

```
GrabCut
→ MeanShift
→ Template
→ ORB
→ SIFT
→ Homography
```

각 기술은 해결하는 문제가 다릅니다.

```
제품과 배경을 분리하고 싶다
→ GrabCut

비슷한 색상 영역을 단순화하고 싶다
→ MeanShift

위치·크기·회전이 거의 일정하다
→ Template Matching

특징적인 점을 비교하고 싶다
→ ORB / SIFT

회전·크기 변화가 있는
평면 라벨의 실제 위치까지 찾고 싶다
→ SIFT + RANSAC + Homography
```

따라서 4일차에서 가장 중요한 것은 기술 이름을 많이 아는 것이 아닙니다.

> 검사 문제와 이미지 조건을 보고 어떤 방법을 사용할지 선택하는 것이 핵심입니다.

![[Pasted image 20261026160603.png]]

---

## 1.4 오늘은 무엇을 실습하고 비교하나요?

4일차에서는 광천김 제품 이미지를 이용하여 **전체 제품을 처리하는 방법 2가지와 로고를 비교하는 방법 4가지**를 배웁니다.

### ① 전체 제품 이미지 처리

- **GrabCut:** 제품과 배경을 분리하고, 분리된 영역이 얼마나 정확한지 비교합니다.
    
- **MeanShift:** 비슷한 색상을 단순화하고, 설정값에 따라 글자와 무늬가 어떻게 달라지는지 비교합니다.
    

### ② 광천김 로고 비교

기준 로고(`reference_label.jpg`)와 정상·위치 이동·15° 회전·90% 크기 이미지 4장을 비교합니다.

- **Template Score:** 기준 로고와 얼마나 비슷한지를 나타내는 점수입니다.
    
- **Template Location:** 검사 이미지에서 기준 로고와 가장 비슷한 영역을 찾은 위치입니다.
    
- **ORB Good Match:** ORB로 찾은 특징점 중 서로 비슷하다고 판단한 연결 개수입니다.
    
- **SIFT Good Match:** SIFT로 찾은 특징점 중 서로 비슷하다고 판단한 연결 개수입니다.
    
- **Homography Inlier:** 특징점 연결 중 위치 관계까지 일치한다고 판단한 개수입니다.
    
- **Homography Inlier Ratio:** 전체 Good Match 중 Inlier가 차지하는 비율입니다.
    

Homography에서는 이러한 특징점의 위치 관계를 이용하여 **실제 광천김 로고의 위치를 사각형으로 표시**하는 것도 확인합니다.

### ③ 오늘 실습의 최종 목표

같은 로고라도 위치·회전·크기가 달라지면 검사 결과가 어떻게 달라지는지 비교하고, **어떤 상황에 어떤 검사 방법이 적합한지 이해하는 것**이 목표입니다.

4일차에서는 아직 측정값을 이용한 최종 OK/NG 기준을 확정하지 않습니다.

**5일차에서는** 여러 정상·불량 이미지의 측정값을 수집하고 비교하여 최종 판정 기준(Threshold·Rule)을 결정합니다.

![[Pasted image 20261026162454.png]]

---

# 2. 1일차부터 4일차까지 어떻게 연결되나요?

## 2.1 1~3일차에서 만든 검사 방식

앞의 병뚜껑 실습에서는 다음과 같이 진행했습니다.

```text
1일차
Camera / Image
→ ROI

        ↓

2일차
ROI
→ 영상 품질 확인
→ 필요한 전처리
→ Threshold / Otsu

        ↓

3일차
Binary
→ Morphology
→ Contour
→ Area / Count / BBox / Center
→ Feature 측정
→ Candidate Rule
→ OK / NG
```

이 방법은 다음처럼 형태가 단순하고 측정할 특징이 분명한 문제에 잘 맞습니다.

```text
병뚜껑이 있는가?
→ Area Ratio

부품이 4개인가?
→ Count

빨간 부품이 있는가?
→ HSV Mask + Area

부품 위치가 맞는가?
→ Center / BBox
```

---

## 2.2 그런데 로고나 글자가 다른 제품은 어떻게 구분할까요?

1~3일차에서는 병뚜껑의 면적(`Area`)을 측정하여 병뚜껑이 있는지 없는지 판단하는 방법을 배웠습니다.

그렇다면 **제품에 인쇄된 로고가 올바른지 검사할 때도 면적만 측정하면 될까요?**

예를 들어 크기는 같지만 서로 다른 글자가 인쇄된 두 라벨이 있다고 생각해 봅니다.

|구분|라벨 A|라벨 B|
|---|---|---|
|라벨 크기|가로 200px × 세로 100px|가로 200px × 세로 100px|
|면적|20,000px²|20,000px²|
|인쇄 내용|광천김|다른 로고|

두 라벨은 크기와 면적이 같습니다. 하지만 인쇄된 글자와 로고는 다릅니다.

따라서 면적(`Area`)만 측정하면 두 라벨의 차이를 구분할 수 없습니다.

```
라벨 A와 라벨 B
        ↓
면적 측정
        ↓
둘 다 20,000px²
        ↓
크기는 같지만 로고는 다름
        ↓
면적만으로는 구분할 수 없음
```

이럴 때는 **라벨의 크기가 아니라 실제 인쇄된 글자·로고·무늬가 기준 이미지와 얼마나 비슷한지** 비교해야 합니다.

4일차에서는 광천김 포장지에서 로고 부분을 ROI로 잘라 기준 이미지(`reference_label.jpg`)로 저장합니다.

이후 이 기준 로고를 전체 제품 이미지와 비교하여 다음을 확인합니다.

```
광천김 기준 로고 ROI
        +
검사할 전체 제품 이미지
        ↓
로고가 얼마나 비슷한가?
        ↓
로고가 어느 위치에 있는가?
        ↓
위치·회전·크기가 달라져도 찾을 수 있는가?
```

이를 위해 **Template Matching, ORB, SIFT, Homography**를 각각 실습합니다.

> **핵심:** 1~3일차에서는 주로 객체의 크기와 개수를 측정했다면, 4일차에서는 로고의 모양과 특징을 비교하여 같은 로고를 찾는 방법을 배웁니다.
> 
---

# 3. 다양한 제품 데이터로 검사 방법을 확장해 봅니다

1~3일차에서는 병뚜껑 데이터를 이용하여 `Area Ratio`를 측정하고 Rule로 OK / NG를 판정하는 기본 구조를 익혔습니다.

4일차에서는 병뚜껑과 다른 형태의 검사 문제를 경험하기 위해 광천김 제품 포장지의 로고 영역을 사용합니다.

```
광천김 포장지
        ↓
KC 로고 + (주)광천김 + 영문 영역
        ↓
위치 이동
회전
거리 변화
기울기 · 반사
        ↓
Template Matching
ORB
SIFT
Homography
```

광천김 포장지는 로고, 한글·영문 글자, 선과 모서리가 함께 있어 특징점이 충분하기 때문에 
ORB와 SIFT 실습에 적합합니다.

## 3.1 4일차 실습 데이터 준비

기준 제품을 한 개 정하고 다음과 같이 촬영합니다.

```
samples/day04/
├── source/
│   └── full_front.jpg
│
├── reference/
│   └── reference_logo.jpg
│
└── challenge/
    ├── shift.jpg
    ├── rotate.jpg
    ├── distance.jpg
    └── tilt_glare.jpg
```

![[Pasted image 20261020083245.png]]

각 이미지의 역할은 다음과 같습니다.

| 파일                   | 촬영 방법                      | 확인할 내용            |
| -------------------- | -------------------------- | ----------------- |
| `full_front.jpg`     | 제품 전체를 정면에서 촬영             | 기준 이미지            |
| `reference_logo.jpg` | `KC + (주)광천김 + 영문` 영역 Crop | 찾을 기준 패턴          |
| `shift.jpg`          | 제품 위치만 좌·우로 이동             | 위치 변화             |
| `rotate.jpg`         | 제품을 약 15~20° 회전            | 회전 변화             |
| `distance.jpg`       | 촬영 거리를 변경                  | 크기 변화             |
| `tilt_glare.jpg`     | 제품을 조금 기울여 약한 반사 발생        | 실제 촬영환경 Challenge |

가능하면 한 번에 여러 조건을 바꾸지 않습니다.

```
기준 이미지
    ↓
위치만 변경

기준 이미지
    ↓
회전만 변경

기준 이미지
    ↓
거리만 변경
```

이렇게 준비해야 어떤 변화 때문에 Matching 결과가 달라졌는지 비교하기 쉽습니다.

## 3.2 다른 제품으로도 같은 실습을 할 수 있습니다

광천김 포장지가 아니더라도 다음처럼 **글자·로고·모서리·독특한 무늬가 있는 제품**을 사용할 수 있습니다.

```
과자 봉지 또는 과자 상자
음료수 라벨
책 표지의 로고
제품 포장지
인쇄된 카드
문자가 포함된 스티커
QR 주변의 고정된 로고
```

반대로 거의 단색이거나 특징이 적은 물체는 ORB / SIFT 실습에 적합하지 않을 수 있습니다.

4일차의 핵심은 많은 데이터를 모으는 것이 아니라, 같은 제품에 위치·회전·크기·촬영환경 변화를 주고 각 Matching 방법이 어떻게 달라지는지 비교하는 것입니다.

---

# 4. 오늘 사용하는 기술의 역할부터 구분합니다

| 기술                      | 오늘 확인할 역할                                | 적합한 상황                   | 대표적인 한계                                 |
| ----------------------- | ---------------------------------------- | ------------------------ | --------------------------------------- |
| `GrabCut`               | 대략적인 제품 위치를 <br>이용해 전경·배경 분리             | 제품과 배경을 먼저 나누고 싶을 때      | 초기 Rectangle이 잘못되거나 전경·배경 색이 비슷하면 실패 가능 |
| `pyrMeanShiftFiltering` | 비슷한 위치·색상 영역을 <br>평탄하게 만들어 색상 <br>영역 단순화 | 색상 영역이 복잡한 이미지 정리        | 너무 강하면 글자·로고·작은 결함까지 사라질 수 있음           |
| `Template Matching`     | 기준 Template의 유사도와 위치 검색                  | 회전·크기 변화가 작고 위치만 달라지는 경우 | 회전·Scale 변화에 약함                         |
| `ORB`                   | 빠른 Keypoint·Descriptor 비교                | 빠른 특징점 비교가 필요할 때         | 특징이 적은 대상이나 큰 변화에서는 불안정할 수 있음           |
| `SIFT`                  | 회전·크기 변화에 비교적 강한 특징점 비교                  | 로고·라벨의 회전·Scale 변화가 있을 때 | ORB보다 계산량이 큰 편                          |
| `Homography`            | 대응 특징점으로 평면 <br>대상의 위치와 변형 추정            | 평면 라벨·카드가 이동·회전·기울어진 경우  | 충분한 올바른 Match가 필요하며 비평면·큰 변형에는 부적합      |
![[Pasted image 20260830121305.png]]

호모그라피 인라이어(Homography Inlier)는 
==컴퓨터 비전 분야에서 두 이미지의 평면을 일치시킬 때, 올바르게 짝지어진 정상적인 
매칭점(특징점)==을 의미합니다.

---

# 5. 오늘 추가할 파일명

3일차 마지막 파일은 다음이었습니다.

```text
src/23_day03_challenge.py
```

따라서 4일차는 `24`부터 이어갑니다.

```text
24_day04_prepare.py
25_grabcut.py
26_meanshift.py
27_template_matching.py
28_orb_matching.py
29_sift_matching.py
30_homography.py
31_day04_compare.py
32_day04_challenge.py
```

각 파일의 역할은 다음과 같습니다.

| 파일명                       | 역할                                                      |
| ------------------------- | ------------------------------------------------------- |
| `24_day04_prepare.py`     | 기준 제품 이미지에서 라벨 Template을 만들고 이동·회전·크기 <br>변화 Target을 준비 |
| `25_grabcut.py`           | Rectangle 초기값을 바꾸어 전경·배경 분리 결과 비교                       |
| `26_meanshift.py`         | `sp`, `sr`을 바꾸어 색상 영역 단순화 결과와 처리시간 비교                   |
| `27_template_matching.py` | 여러 Target에서 Template Score와 검출 위치 측정                    |
| `28_orb_matching.py`      | ORB Keypoint·Descriptor·Good Match 수 측정                 |
| `29_sift_matching.py`     | SIFT Keypoint·Descriptor·Good Match 수 측정                |
| `30_homography.py`        | SIFT Match + RANSAC으로 라벨 위치를 사각형으로 추정                   |
| `31_day04_compare.py`     | Template / ORB / SIFT / Homography의 측정 CSV를 하나로 <br>정리  |
| `32_day04_challenge.py`   | 새로운 실제 촬영 이미지에서 사용할 방법을 직접 선택하는 <br>Mini Challenge      |
![[Pasted image 20261020083515.png]]

![[Pasted image 20261020084029.png]]

4일차의 핵심은 같은 광천김 로고를 여러 검사 방법으로 비교하여, 위치·회전·크기 변화에 어떤 방법이 적합한지 이해하는 것입니다.

그림에서 특히 중요한 점은 GrabCut과 MeanShift는 별도의 실험이고, Template Matching·ORB·SIFT·Homography는 같은 기준 이미지와 비교 이미지를 각각 처리하는 방법이라는 것입니다.

31번에서 네 가지 측정 결과를 통합하고, 32번에서는 실제 촬영한 새로운 이미지로 학생들이 직접 검사 방법을 선택합니다.

---

# 6. 4일차 프로젝트 구조

1~3일차 프로젝트를 그대로 사용합니다.

새 프로젝트를 만들지 않습니다.

```text
subject9_machinevision/
│
├── .venv/
│
├── samples/
│   ├── day02/
│   ├── day03/
│   │
│   └── day04/
│       ├── source/
│       │   └── product_label.jpg
│       │
│       ├── reference/
│       │   ├── reference_label.jpg
│       │   └── reference_preview.jpg
│       │
│       ├── targets/
│       │   ├── target_normal.jpg
│       │   ├── target_shift.jpg
│       │   ├── target_rot15.jpg
│       │   └── target_scale90.jpg
│       │
│       └── challenge/
│           ├── challenge_01.jpg
│           ├── challenge_02.jpg
│           └── ...
│
├── outputs/
│   ├── day02/
│   ├── day03/
│   │
│   └── day04/
│       ├── grabcut/
│       ├── meanshift/
│       ├── template/
│       ├── orb/
│       ├── sift/
│       ├── homography/
│       └── challenge/
│
├── src/
│   ├── ...
│   ├── 23_day03_challenge.py
│   ├── 24_day04_prepare.py
│   ├── 25_grabcut.py
│   ├── 26_meanshift.py
│   ├── 27_template_matching.py
│   ├── 28_orb_matching.py
│   ├── 29_sift_matching.py
│   ├── 30_homography.py
│   ├── 31_day04_compare.py
│   └── 32_day04_challenge.py
│
├── reports/
│   ├── ...
│   ├── day04_template_results.csv
│   ├── day04_orb_results.csv
│   ├── day04_sift_results.csv
│   ├── day04_homography_results.csv
│   ├── day04_rotation_compare.csv
│   ├── day04_method_comparison.csv
│   └── day04_method_selection.md
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

# 7. 실습환경 다시 열기

프로젝트 폴더로 이동합니다.

```bash
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

1일차의 환경 점검 프로그램을 다시 실행합니다.

```bash
python src/00_check_env.py
```

정상이라면 다음과 같은 결과가 나옵니다.

```text
Python : 3.12.x
OpenCV: 5.x.x
NumPy : 2.x.x
Environment OK
```

VS Code를 엽니다.

```bash
code -r .
```

> 새 가상환경을 만들거나 OpenCV를 다시 설치하지 않습니다.  
> 1일차부터 같은 프로젝트 환경을 계속 사용합니다.

---

# 8. SIFT를 사용할 수 있는지 먼저 확인합니다

4일차 후반에서는 제품 라벨이나 로고의 특징점을 찾아 비교하기 위해 `ORB`와 `SIFT`를 사용합니다. SIFT는 이미지에서 다음과 같은 특징적인 지점을 찾고 비교하는 기술입니다.

```
글자의 모서리 
로고의 끝부분 
선이 만나는 지점 
독특한 무늬 
     ↓ 
Keypoint 찾기 
     ↓ 
주변 모양을 숫자로 표현 
     ↓ 
다른 이미지의 특징과 비교
```

특히 제품이 조금 회전하거나 크기가 달라져도 같은 특징을 비교하는 데 사용할 수 있기 때문에, 뒤에서 광천김 로고의 회전·거리 변화 실습에 사용합니다.

OpenCV에서는 SIFT를 다음 함수로 생성합니다.

```python
cv2.SIFT_create()
```

따라서 현재 설치된 OpenCV에서 이 함수를 사용할 수 있는지 먼저 확인합니다.

터미널에서 확인합니다.

```bash
python -c "import cv2; print('SIFT available:', hasattr(cv2, 'SIFT_create'))"
```

정상 예:

```text
SIFT available: True
```

현재 사용 중인 OpenCV에서 `True`가 나오면 그대로 진행합니다.

`True`의 의미는 다음과 같습니다.

```
현재 OpenCV에
cv2.SIFT_create 함수가 존재함
        ↓
오늘 SIFT 실습을 그대로 진행할 수 있음
```

만약 `False`가 나온 경우에는 현재 OpenCV 설치 상태를 확인합니다.

```bash
python -m pip show opencv-python
python -m pip list | grep opencv
```

여기서는 아직 SIFT 알고리즘을 자세히 배우는 단계가 아닙니다.  
지금은 뒤의 특징점 매칭 실습에 필요한 SIFT 기능이 현재 개발환경에서 사용할 수 있는지만 확인하고 넘어갑니다.

---

# 9. Day04 폴더를 만듭니다

터미널에서 실행합니다.

```bash
mkdir -p samples/day04/source
mkdir -p samples/day04/reference
mkdir -p samples/day04/targets
mkdir -p samples/day04/challenge

mkdir -p outputs/day04/grabcut
mkdir -p outputs/day04/meanshift
mkdir -p outputs/day04/template
mkdir -p outputs/day04/orb
mkdir -p outputs/day04/sift
mkdir -p outputs/day04/homography
mkdir -p outputs/day04/challenge
```

확인합니다.

```bash
find samples/day04 outputs/day04 -maxdepth 2 -type d | sort
```

---

# 10. 광천김 포장지를 4일차 기준 데이터로 사용합니다

4일차에서는 다음 광천김 포장지를 새로운 검사 대상으로 사용합니다. 

1~3일차에서는 병뚜껑의 면적을 측정했다면, 4일차에서는 제품 포장지에 있는 로고와 글자 무늬가 기준 이미지와 같은지 비교합니다. 

이번 실습에서 찾을 영역은 다음 부분입니다.

```text
KC 컬러 로고
    +
(주)광천김
    +
KWANGCHEONKIM CO., LTD.
```

이 영역에는 글자, 모서리, 선, 색상 변화가 충분히 포함되어 있어 Template Matching과 ORB·SIFT 특징점 비교를 연습하기에 적합합니다.

## 10.1 준비한 실제 촬영 이미지

다음과 같이 이미지를 준비합니다.

```
samples/day04/
├── source/
│   └── full_front.jpg
│
├── reference/
│   └── reference_logo.jpg
│
└── challenge/
    ├── shift.jpg
    ├── rotate.jpg
    ├── distance.jpg
    └── tilt_glare.jpg
```

각 파일의 역할은 다음과 같습니다.

| 파일                   | 역할                                |
| -------------------- | --------------------------------- |
| `full_front.jpg`     | 광천김 포장지를 정면에서 촬영한 기준 전체 이미지       |
| `reference_logo.jpg` | `KC + (주)광천김 + 영문` 영역을 잘라낸 기준 이미지 |
| `shift.jpg`          | 실제 제품의 위치를 이동하여 다시 촬영한 이미지        |
| `rotate.jpg`         | 실제 제품을 약간 회전하여 촬영한 이미지            |
| `distance.jpg`       | 카메라와 제품의 거리를 변경하여 촬영한 이미지         |
| `tilt_glare.jpg`     | 제품을 기울여 원근 변화와 비닐 반사가 발생한 이미지     |

## 10.2 `full_front.jpg` 한 장에서 비교용 Target을 만듭니다

먼저 실제 촬영한 `full_front.jpg` 한 장을 기준으로 사용합니다.

```
samples/day04/source/full_front.jpg
```

이 이미지는 광천김 포장지를 정면에서 촬영한 기준 원본 이미지입니다.

처음부터 실제 촬영한 `shift.jpg`, `rotate.jpg`, `distance.jpg`를 사용하지 않고, 먼저 Python 코드에서 `full_front.jpg`를 조금씩 변형하여 비교용 이미지를 만듭니다.

```
full_front.jpg
      │
      ├── 그대로 저장
      │      ↓
      │   target_normal.jpg
      │
      ├── 오른쪽 35px + 아래쪽 20px 이동
      │      ↓
      │   target_shift.jpg
      │
      ├── 이미지 중심을 기준으로 15° 회전
      │      ↓
      │   target_rot15.jpg
      │
      └── 이미지 중심을 기준으로 90% 크기로 축소
             ↓
          target_scale90.jpg
```

생성된 이미지는 다음 폴더에 저장합니다.

```
samples/day04/targets/
├── target_normal.jpg
├── target_shift.jpg
├── target_rot15.jpg
└── target_scale90.jpg
```

### 각 이미지는 무엇이 다른가요?

| 파일                   | 원본에서 변경하는 것 | 확인할 목적                   |
| -------------------- | ----------- | ------------------------ |
| `target_normal.jpg`  | 변경하지 않음     | 기준 상태                    |
| `target_shift.jpg`   | 위치만 이동      | 위치가 달라져도 찾는지 확인          |
| `target_rot15.jpg`   | 15° 회전      | 회전에 따른 Matching 변화 확인    |
| `target_scale90.jpg` | 90% 크기로 축소  | 크기 변화에 따른 Matching 변화 확인 |
중요한 점은 한 이미지에서 한 가지 조건만 바꾸는 것입니다.

예를 들어 `target_rot15.jpg`는:

```
제품 동일
조명 동일
배경 동일
이미지 품질 동일

변경된 것
→ 회전만 15°
```

입니다.

따라서 Template Matching의 Score가 낮아졌다면:

```
회전 때문에 결과가 변했을 가능성이 높다.
```

라고 비교하기 쉽습니다.

`target_scale90.jpg`도 마찬가지입니다.

```
제품 동일
조명 동일
배경 동일

변경된 것
→ 크기만 90%
```

이렇게 조건을 통제한 상태에서:

```
Template Matching
ORB
SIFT
Homography
```

를 같은 네 장의 이미지에 적용하여 위치·회전·크기 변화에 따라 각 방법의 결과가 어떻게 달라지는지 비교합니다.

> 이 단계는 새로운 사진을 네 장 촬영하는 것이 아닙니다.  
> `full_front.jpg` 한 장을 Python 코드로 복사·이동·회전·축소하여 실험용 Target 4장을 자동으로 만드는 과정입니다.

실제 촬영한:

```
shift.jpg
rotate.jpg
distance.jpg
tilt_glare.jpg
```

는 기본 원리를 확인한 뒤 Mini Challenge에서 실제 촬영환경 테스트용으로 사용합니다.

---

# 11. `24_day04_prepare.py` — Template과 비교 Target을 준비합니다

4일차 첫 번째 파일을 만듭니다.

```text
src/24_day04_prepare.py
```

이 코드는 다음 작업을 한 번에 수행합니다.

```
1. full_front.jpg를 읽는다.
2. 광천김 로고 영역을 Crop하여 reference_label.jpg를 만든다.
3. Template 위치를 표시한 reference_preview.jpg를 만든다.
4. 원본 전체 이미지를 target_normal.jpg로 저장한다.
5. 위치 이동 Target을 만든다.
6. 회전 Target을 만든다.
7. 크기 변화 Target을 만든다.
```

코드는 다음과 같습니다.

> 아래 `x1, y1, x2, y2`는 예시입니다.  
> 반드시 자신의 `full_front.jpg`에서  
> **KC 로고 + (주)광천김 + 영문** 영역에 맞게 수정합니다.

```python
from pathlib import Path

import cv2
import numpy as np


# --------------------------------------------------
# 1. 사용할 원본 이미지와 결과 저장 경로를 설정합니다.
#
# source_path
# → 광천김 전체 제품 이미지
#
# reference_dir
# → 잘라낸 기준 로고 ROI를 저장할 폴더
#
# target_dir
# → 위치·회전·크기를 변경한 비교 이미지를 저장할 폴더
# --------------------------------------------------

source_path = Path(
    "samples/day04/source/full_front.jpg"
)

reference_dir = Path(
    "samples/day04/reference"
)

target_dir = Path(
    "samples/day04/targets"
)


# --------------------------------------------------
# 2. 결과 이미지를 저장할 폴더를 생성합니다.
#
# parents=True
# → 상위 폴더가 없으면 함께 생성
#
# exist_ok=True
# → 폴더가 이미 있어도 오류 없이 진행
# --------------------------------------------------

reference_dir.mkdir(
    parents=True,
    exist_ok=True,
)

target_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 3. 광천김 전체 제품 이미지를 읽고 확인합니다.
#
# cv2.imread()
# → 이미지 파일을 읽는 함수
#
# image가 None이면 파일을 읽지 못한 상태입니다.
# --------------------------------------------------

image = cv2.imread(
    str(source_path)
)

if image is None:
    raise FileNotFoundError(
        source_path
    )


# --------------------------------------------------
# 4. 원본 이미지의 가로와 세로 크기를 확인합니다.
#
# image.shape[:2]
# → 이미지 높이(height), 너비(width)
#
# 이 값은 ROI 좌표 확인과
# Target 이미지 생성에 사용합니다.
# --------------------------------------------------

height, width = image.shape[:2]


# --------------------------------------------------
# 5. 기준 로고로 사용할 ROI 좌표를 설정합니다.
#
# 이번에 자를 영역:
# KC 로고 + (주)광천김 + 영문
#
# x1, y1 → ROI의 왼쪽 위 좌표
# x2, y2 → ROI의 오른쪽 아래 좌표
#
# 아래 좌표는 예시입니다.
# 실제 사진의 로고 위치에 맞게 수정해야 합니다.
# --------------------------------------------------

x1 = 510
y1 = 870
x2 = 950
y2 = 1080


# --------------------------------------------------
# 6. ROI 좌표가 이미지 범위를 벗어나지 않도록 조정합니다.
#
# 이미지 바깥쪽 좌표를 지정하면
# 올바른 영역을 잘라내지 못할 수 있습니다.
#
# min() → 이미지 크기를 넘지 않도록 제한
# max() → 좌표가 음수가 되지 않도록 제한
# --------------------------------------------------

x1 = max(0, min(x1, width))
x2 = max(0, min(x2, width))

y1 = max(0, min(y1, height))
y2 = max(0, min(y2, height))


# --------------------------------------------------
# 7. ROI 좌표가 올바른지 검사합니다.
#
# 왼쪽 좌표가 오른쪽보다 크거나 같으면
# 정상적인 ROI를 만들 수 없습니다.
#
# 위쪽 좌표가 아래쪽보다 크거나 같아도
# 오류를 발생시킵니다.
# --------------------------------------------------

if x1 >= x2 or y1 >= y2:
    raise ValueError(
        f"잘못된 Template 좌표: "
        f"{(x1, y1, x2, y2)}"
    )


# --------------------------------------------------
# 8. 전체 이미지에서 로고 ROI를 잘라 Reference를 만듭니다.
#
# image[y1:y2, x1:x2]
# → 지정한 사각형 영역만 잘라냅니다.
#
# reference_label.jpg
# → 이후 Template / ORB / SIFT / Homography에서
#   공통으로 사용할 기준 로고 이미지입니다.
# --------------------------------------------------

reference = image[
    y1:y2,
    x1:x2
]

reference_path = (
    reference_dir
    / "reference_label.jpg"
)

cv2.imwrite(
    str(reference_path),
    reference,
)


# --------------------------------------------------
# 9. 원본 이미지에 ROI 위치를 표시한 Preview를 만듭니다.
#
# image.copy()
# → 원본 이미지 복사
#
# cv2.rectangle()
# → 로고 ROI 위치에 초록색 사각형 표시
#
# reference_preview.jpg
# → 실제로 올바른 로고를 잘랐는지 확인하는 이미지
# --------------------------------------------------

preview = image.copy()

cv2.rectangle(
    preview,
    (x1, y1),
    (x2, y2),
    (0, 255, 0),
    3,
)

cv2.imwrite(
    str(
        reference_dir
        / "reference_preview.jpg"
    ),
    preview,
)


# --------------------------------------------------
# 10. 정상 Target 이미지를 만듭니다.
#
# target_normal.jpg
# → 원본 이미지를 변경하지 않고 그대로 저장
#
# 이후 다른 Target들과 결과를 비교할 때
# 기준 상태로 사용합니다.
# --------------------------------------------------

cv2.imwrite(
    str(
        target_dir
        / "target_normal.jpg"
    ),
    image,
)


# --------------------------------------------------
# 11. 위치를 이동한 Target 이미지를 만듭니다.
#
# 오른쪽으로 35px 이동
# 아래쪽으로 20px 이동
#
# cv2.warpAffine()
# → 이동 변환 행렬을 이미지에 적용
#
# target_shift.jpg
# → 제품 위치가 바뀌었을 때
#   로고를 찾을 수 있는지 비교하는 이미지
# --------------------------------------------------

shift_x = 35
shift_y = 20

move_matrix = np.float32([
    [1, 0, shift_x],
    [0, 1, shift_y],
])

shifted = cv2.warpAffine(
    image,
    move_matrix,
    (width, height),
    borderMode=cv2.BORDER_REPLICATE,
)

cv2.imwrite(
    str(
        target_dir
        / "target_shift.jpg"
    ),
    shifted,
)


# --------------------------------------------------
# 12. 이미지를 15도 회전한 Target을 만듭니다.
#
# 이미지 중심을 기준으로 회전합니다.
#
# cv2.getRotationMatrix2D()
# → 회전 변환 행렬 생성
#
# cv2.warpAffine()
# → 회전 변환 적용
#
# target_rot15.jpg
# → 로고가 회전했을 때
#   각 검사 방법의 결과가 달라지는지 비교
# --------------------------------------------------

center = (
    width / 2,
    height / 2,
)

rotation_matrix = cv2.getRotationMatrix2D(
    center,
    15,
    1.0,
)

rotated = cv2.warpAffine(
    image,
    rotation_matrix,
    (width, height),
    borderMode=cv2.BORDER_REPLICATE,
)

cv2.imwrite(
    str(
        target_dir
        / "target_rot15.jpg"
    ),
    rotated,
)


# --------------------------------------------------
# 13. 이미지를 90% 크기로 축소한 Target을 만듭니다.
#
# scale=0.90
# → 원본 크기의 90%로 축소
#
# 이미지 중심을 기준으로 축소합니다.
#
# target_scale90.jpg
# → 로고 크기가 달라졌을 때
#   검사 결과가 어떻게 변하는지 비교
# --------------------------------------------------

scale_matrix = cv2.getRotationMatrix2D(
    center,
    0,
    0.90,
)

scaled = cv2.warpAffine(
    image,
    scale_matrix,
    (width, height),
    borderMode=cv2.BORDER_REPLICATE,
)

cv2.imwrite(
    str(
        target_dir
        / "target_scale90.jpg"
    ),
    scaled,
)


# --------------------------------------------------
# 14. 생성된 이미지 정보를 출력하고 결과를 확인합니다.
#
# 확인할 내용:
#
# 1. Reference 이미지가 올바르게 잘렸는가?
# 2. 정상 Target은 원본과 같은가?
# 3. 이동 Target은 위치가 달라졌는가?
# 4. 회전 Target은 15도 회전되었는가?
# 5. 축소 Target은 90% 크기가 되었는가?
#
# 이 이미지들은 뒤의 로고 비교 실습에서 사용합니다.
# --------------------------------------------------

print(
    "Source :",
    source_path
)

print(
    "Template:",
    reference_path
)

print(
    "Template 좌표:",
    (x1, y1, x2, y2)
)

print(
    "Template 크기:",
    reference.shape
)

print(
    "Day04 Target 생성 완료"
)
```

### 의사코드

```text
[1] 원본 이미지와 저장 경로 설정

    광천김 전체 제품 이미지
    full_front.jpg의 경로를 정한다.

    기준 로고를 저장할 Reference 폴더와
    비교 이미지를 저장할 Target 폴더의
    경로를 정한다.


[2] 결과 저장 폴더 생성

    Reference 폴더를 만든다.

    Target 폴더를 만든다.

    이미 폴더가 있다면 그대로 사용한다.


[3] 전체 제품 이미지 읽기

    full_front.jpg를 읽는다.

    이미지가 존재하는지 확인한다.

    파일을 읽지 못하면 오류를 발생시킨다.


[4] 원본 이미지 크기 확인

    전체 이미지의
    가로(width)와 세로(height)를 확인한다.

    ROI를 자르고 이미지를 변형할 때
    이 크기 정보를 사용한다.


[5] 기준 로고 ROI 좌표 설정

    광천김 전체 사진에서

    KC 로고 + (주)광천김 + 영문

    영역의 사각형 좌표를 정한다.

    x1, y1 → 왼쪽 위
    x2, y2 → 오른쪽 아래


[6] ROI 좌표 범위 조정

    지정한 좌표가
    이미지 바깥으로 나가지 않도록 조정한다.

    0보다 작은 값은 0으로 제한한다.

    이미지 너비와 높이를 넘는 값은
    이미지 경계로 제한한다.


[7] ROI 좌표 유효성 검사

    사각형의 가로와 세로가
    정상적인 크기인지 확인한다.

    잘못된 좌표라면 오류를 발생시킨다.


[8] 기준 로고 Reference 생성

    전체 이미지에서
    광천김 로고 부분만 ROI로 잘라낸다.

    잘라낸 로고를 다음 이름으로 저장한다.

    reference_label.jpg

    이 이미지는 이후 모든 로고 비교 실험의
    공통 기준 이미지가 된다.


[9] ROI 위치 확인용 Preview 생성

    전체 제품 이미지를 복사한다.

    기준 로고를 잘라낸 위치에
    초록색 사각형을 그린다.

    reference_preview.jpg로 저장한다.

    사각형이 실제 로고를
    정확하게 감싸는지 확인한다.


[10] 정상 Target 생성

    원본 전체 이미지를 그대로 복사한다.

    target_normal.jpg로 저장한다.

    다른 조건의 Target과
    비교할 기준 이미지로 사용한다.


[11] 위치 이동 Target 생성

    원본 전체 이미지를

    오른쪽 35px
    아래쪽 20px

    이동시킨다.

    target_shift.jpg로 저장한다.

    제품 위치가 달라졌을 때
    로고를 찾을 수 있는지 확인할 예정이다.


[12] 15도 회전 Target 생성

    원본 이미지의 중심을 찾는다.

    중심을 기준으로
    이미지를 15도 회전시킨다.

    target_rot15.jpg로 저장한다.

    로고가 회전했을 때
    검사 결과가 어떻게 달라지는지
    비교할 예정이다.


[13] 90% 크기 Target 생성

    원본 이미지의 중심을 찾는다.

    이미지를 원본 크기의 90%로 축소한다.

    target_scale90.jpg로 저장한다.

    로고 크기가 달라졌을 때
    검사 결과가 어떻게 달라지는지
    비교할 예정이다.


[14] 생성 결과 확인

    기준 로고 ROI가 올바르게 잘렸는지 확인한다.

    다음 Target 4장이 생성되었는지 확인한다.

    ① target_normal.jpg
       → 정상 이미지

    ② target_shift.jpg
       → 위치 이동 이미지

    ③ target_rot15.jpg
       → 15도 회전 이미지

    ④ target_scale90.jpg
       → 90% 크기 이미지

    이후 Template Matching,
    ORB, SIFT, Homography 실습에서

    같은 Reference와
    같은 Target 4장을 사용하여

    각 검사 방법의 결과를 비교한다.
```

실행합니다.

```bash
python src/24_day04_prepare.py
```

결과를 확인합니다.

```text
samples/day04/reference/
├── reference_label.jpg
└── reference_preview.jpg

samples/day04/targets/
├── target_normal.jpg
├── target_shift.jpg
├── target_rot15.jpg
└── target_scale90.jpg
```

## 11.1 생성된 비교 이미지를 확인합니다

`24_day04_prepare.py`를 실행하면 하나의 `full_front.jpg`에서 조건이 다른 네 장의 이미지가 만들어집니다.

중요한 포인트는 생성된 4장 모두 `full_front.jpg`에서 만들어졌습니다.

```
제품        동일
조명        동일
배경        동일
촬영 품질    동일

달라진 것
→ 위치 / 회전 / 크기 중 하나
```

따라서 뒤에서 같은 검사 방법을 네 이미지에 적용하면:

```
위치가 바뀌면 결과가 어떻게 달라지는가?

15° 회전하면 결과가 어떻게 달라지는가?

크기가 90%로 변하면 결과가 어떻게 달라지는가?
```

를 비교할 수 있습니다.


---
## reference 결과 수정

![[Pasted image 20260830134814.png]]

```python
x1 = 180
y1 = 120
x2 = 420
y2 = 280
```

위의 위치를 직접 수정할수 있습니다.

ROI 배운 첫째날 그리드를 표시하여 정확히 잘라도 되니 위의 좌표값을 수정하여 다시 로고 주변을 넉넉히 ROI로 자릅니다.

좌표값을 아래와 같이 수정하고 얻은 결과 이미지

```python
x1 = 510
y1 = 870
x2 = 950
y2 = 1080
```

![[Pasted image 20261027162953.png|218]]![[Pasted image 20261027163001.png|212]]![[Pasted image 20261027163024.png|268]]

  reference_label.jpg               reference_logo.jpg              reference_preview.jpg

---
# PART 12~15 — 검사 1: GrabCut으로 제품과 배경 분리

---

# 12. GrabCut은 무엇을 하는 기술인가요?

3일차에서는 Threshold와 Contour를 사용해 Binary Image에서 객체를 찾았습니다.

하지만 객체와 배경의 밝기 차이가 명확하지 않다면 단순 Threshold만으로 전경을 분리하기 어려울 수 있습니다.

GrabCut은 사용자가 대략적으로:

```text
"이 사각형 안에
찾고 싶은 제품이 대부분 들어 있습니다."
```

라는 초기 정보를 주면 전경과 배경을 반복적으로 구분합니다.

```text
원본 이미지
        ↓
대략적인 Rectangle
        ↓
GrabCut
        ↓
전경 후보
배경 후보
        ↓
Foreground Mask
```

오늘은 GrabCut의 수학식을 깊게 다루지 않습니다.

> GrabCut은 제품 위치를 전혀 모르는 상태에서 자동으로 모든 객체를 찾는 검출기가 아니라, 
> 대략적인 초기 영역을 이용해 전경과 배경을 분리하는 방법입니다.

![[Pasted image 20260830135629.png]]

---

# 13. GrabCut은 픽셀을 4가지 상태로 나누어 판단합니다

![[Pasted image 20260830135859.png]]

이미지에 나오는 `GC`, `BGD`, `FGD`, `PR`은 OpenCV의 GrabCut 알고리즘에서 사용하는 약자입니다.

|약자|영어 원문|의미|
|---|---|---|
|GC|GrabCut|전경과 배경을 분리하는 알고리즘|
|BGD|Background|배경|
|FGD|Foreground|전경(검출할 객체)|
|PR|Probable|~일 가능성이 높은|
### GrabCut의 4가지 Mask 값

| OpenCV 상수       | 값   | 의미             |
| --------------- | --- | -------------- |
| `cv2.GC_BGD`    | 0   | 확실한 배경         |
| `cv2.GC_FGD`    | 1   | 확실한 전경         |
| `cv2.GC_PR_BGD` | 2   | 배경일 가능성이 높은 영역 |
| `cv2.GC_PR_FGD` | 3   | 전경일 가능성이 높은 영역 |

GrabCut은 이미지를 바로 `제품 / 배경` 두 가지로만 나누지 않습니다. 

먼저 이미지의 각 픽셀을 다음 4가지 상태로 구분하면서 전경과 배경을 반복해서 판단합니다.

```text
GC_BGD
→ 확실한 배경
→ "여기는 배경이 맞다"

GC_FGD
→ 확실한 전경
→ "여기는 제품이 맞다"

GC_PR_BGD
→ 배경일 가능성이 높은 영역
→ "아마 배경일 것이다"

GC_PR_FGD
→ 전경일 가능성이 높은 영역
→ "아마 제품일 것이다"
```

쉽게 생각하면 다음과 같습니다.

```text
확실한 배경
배경 같음
제품 같음
확실한 제품
```

GrabCut은 이 상태를 여러 번 다시 계산하면서 제품과 배경의 경계를 조금씩 정리합니다.\

최종 결과에서는 네 상태를 다시 두 그룹으로 묶습니다.

```
GC_BGD
+
GC_PR_BGD
        ↓
배경
→ 검은색


GC_FGD
+
GC_PR_FGD
        ↓
전경
→ 흰색
```

따라서 최종 `Foreground Mask`에서는:

```
흰색
→ 남겨 둘 제품 영역

검은색
→ 제거할 배경 영역
```

으로 확인할 수 있습니다.

예를 들어 광천김 포장지를 분리한다면:

```
원본 이미지
        ↓
GrabCut이 픽셀을 4가지 상태로 판단
        ↓
제품으로 판단한 영역을 하나로 묶음
        ↓
Foreground Mask
        ↓
광천김 포장지만 남김
```

오늘은 `GC_BGD`, `GC_FGD` 같은 이름이나 숫자를 외우는 것이 중요하지 않습니다.  
GrabCut이 처음부터 전경과 배경을 완벽하게 아는 것이 아니라, 확실한 영역과 가능성이 높은 영역을 나누어 반복적으로 판단한다는 원리만 이해하면 됩니다.

좀더 쉽게 설명하면

```
포토샵
사람이 마스크를 직접 정교하게 수정
→ 원하는 부분만 남김

GrabCut
사람이 대략적인 영역을 알려줌
→ 컴퓨터가 전경/배경을 추정해서 Mask 생성
```

GrabCut — 사람이 먼저 전경이 있을 대략적인 영역을 지정하면, OpenCV의 GrabCut이 이미지의 색상과 픽셀 정보를 분석하여 전경과 배경을 자동으로 구분합니다.

GrabCut은 사람이 이미지에서 분리하고 싶은 객체(예: 사람)가 포함된 영역을 마우스로 사각형(ROI)으로 지정하면, 컴퓨터가 해당 영역의 색상과 픽셀 특성을 분석하여 전경(사람)과 배경을 구분하는 알고리즘입니다.

처리 순서는 다음과 같습니다.

```
[사람]

광천김 제품 이미지를 확인
       ↓
제품을 둘러싸는 사각형 지정
       ↓
[OpenCV의 초기 설정]

사각형 바깥 → 확실한 배경 (GC_BGD)

사각형 안쪽 → 전경일 가능성이 높은 영역
               (GC_PR_FGD)
       ↓
[GrabCut 자동 분석]

전경과 배경의 색상 분포 분석
       ↓
픽셀 간 경계와 주변 관계 분석
       ↓
전경 / 배경 분류 반복
       ↓
최종 제품 영역 추출
```

---

# 14. `25_grabcut.py` — 초기 Rectangle을 바꾸어 비교합니다

파일을 만듭니다.

```text
src/25_grabcut.py
```

제품 가장자리 주변에 **약 10~20% 정도의 배경 여백**이 보이는 사진 데이터를 준비하여 아래 경로에 복사해둡니다.

```
samples/day04/source/
├── full_front.jpg
└── grabcut_scene.jpg
```

```bash
explorer.exe .
```

```python
# ==================================================
# [01] 라이브러리 불러오기
#
# Path         : 이미지 파일 및 저장 경로 관리
# perf_counter : GrabCut 실행 시간 측정
# cv2          : OpenCV 이미지 처리
# numpy        : Mask 및 배열 연산
# ==================================================

from pathlib import Path
from time import perf_counter

import cv2
import numpy as np


# ==================================================
# [02] 입력 이미지 및 결과 저장 경로 설정
#
# GrabCut으로 전경을 분리할 이미지 경로 지정
# 결과 이미지를 저장할 폴더 생성
# ==================================================

image_path = Path(
    "samples/day04/source/grabcut_scene.jpg"
)

output_dir = Path(
    "outputs/day04/grabcut"
)

# 저장 폴더가 없으면 자동 생성
output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# ==================================================
# [03] 원본 이미지 읽기
#
# 이미지 파일을 메모리로 불러오기
# 이미지가 존재하지 않으면 오류 발생
# 원본 이미지의 가로·세로 크기 확인
# ==================================================

image = cv2.imread(
    str(image_path)
)

if image is None:
    raise FileNotFoundError(
        image_path
    )

# 이미지 크기 확인
original_h, original_w = image.shape[:2]

print(
    "Original size:",
    original_w,
    "x",
    original_h,
)


# ==================================================
# [04] GrabCut 처리용 이미지 크기 조절
#
# 원본 이미지가 너무 크면 처리 시간이 증가함
#
# 가로 크기가 900px보다 클 경우:
#   1. 축소 비율 계산
#   2. 원본 비율을 유지하여 이미지 축소
#   3. 축소된 이미지 저장
#
# 가로 크기가 900px 이하이면 원본 크기 유지
# ==================================================

process_width = 900

if original_w > process_width:

    # 원본 가로 크기 대비 축소 비율
    scale = (
        process_width
        / original_w
    )

    # 원본 비율을 유지한 세로 크기 계산
    process_height = int(
        original_h * scale
    )

    # 이미지 축소
    image = cv2.resize(
        image,
        (
            process_width,
            process_height,
        ),
        interpolation=cv2.INTER_AREA,
    )


# 실제 처리할 이미지 크기
height, width = image.shape[:2]

print(
    "Process size :",
    width,
    "x",
    height,
)


# 처리용 이미지 저장
cv2.imwrite(
    str(
        output_dir /
        "resized_input.jpg"
    ),
    image,
)


# ==================================================
# [05] GrabCut 초기 Rectangle 영역 설정
#
# GrabCut은 객체가 포함된 사각형 영역을 기준으로
# 전경과 배경을 추정함
#
# 이번 실습에서는 사람이 마우스로 ROI를 지정하지 않고
# 이미지 크기와 여백 비율로 Rectangle을 자동 생성
#
# wide   : 여백 3%  → 가장 넓은 Rectangle
# medium : 여백 7%  → 중간 크기 Rectangle
# tight  : 여백 12% → 가장 좁은 Rectangle
#
# 세 가지 Rectangle 조건을 비교하여
# 초기 영역이 결과에 미치는 영향을 확인
# ==================================================

margin_ratios = [
    ("wide", 0.03),
    ("medium", 0.07),
    ("tight", 0.12),
]


# 세 가지 Rectangle 조건 반복 실행
for name, margin_ratio in margin_ratios:

    # 이미지 크기를 기준으로 여백 계산
    margin_x = int(
        width * margin_ratio
    )

    margin_y = int(
        height * margin_ratio
    )

    # Rectangle 시작 좌표
    rect_x = margin_x
    rect_y = margin_y

    # Rectangle 가로·세로 크기 계산
    rect_w = (
        width
        - 2 * margin_x
    )

    rect_h = (
        height
        - 2 * margin_y
    )

    # Rectangle 크기가 유효하지 않으면 건너뜀
    if rect_w <= 0 or rect_h <= 0:
        continue

    # GrabCut에 전달할 사각형 정보
    # (x, y, width, height)
    rectangle = (
        rect_x,
        rect_y,
        rect_w,
        rect_h,
    )


    # ==================================================
    # [06] GrabCut용 Mask와 내부 모델 준비
    #
    # GrabCut은 Mask를 이용하여 각 픽셀이
    # 전경인지 배경인지 구분함
    #
    # mask     : 전경/배경 분류 정보를 저장할 배열
    # bg_model : 배경의 색상 분포를 추정하는 내부 모델
    # fg_model : 전경의 색상 분포를 추정하는 내부 모델
    #
    # GC_INIT_WITH_RECT 방식에서는
    # Rectangle을 기준으로 초기 Mask가 설정됨
    # ==================================================

    # 이미지와 동일한 크기의 Mask 생성
    mask = np.zeros(
        (height, width),
        dtype=np.uint8,
    )

    # 배경 모델 초기화
    bg_model = np.zeros(
        (1, 65),
        dtype=np.float64,
    )

    # 전경 모델 초기화
    fg_model = np.zeros(
        (1, 65),
        dtype=np.float64,
    )


    # ==================================================
    # [07] GrabCut 알고리즘 실행
    #
    # Rectangle 바깥쪽:
    #   → 확실한 배경으로 처리
    #
    # Rectangle 안쪽:
    #   → 색상과 주변 픽셀 관계를 분석하여
    #      전경과 배경을 추정
    #
    # 3번의 반복(iteration)을 통해 분할 결과 개선
    # GrabCut 처리 시간 측정
    # ==================================================

    # 실행 시작 시간
    start = perf_counter()

    # GrabCut 실행
    cv2.grabCut(
        image,
        mask,
        rectangle,
        bg_model,
        fg_model,
        3,
        cv2.GC_INIT_WITH_RECT,
    )

    # 실행 시간 계산 (밀리초)
    elapsed_ms = (
        perf_counter()
        - start
    ) * 1000


    # ==================================================
    # [08] GrabCut 결과를 흑백 Mask로 변환
    #
    # GrabCut Mask에는 네 가지 분류값이 존재함
    #
    # GC_BGD    : 확실한 배경
    # GC_FGD    : 확실한 전경
    # GC_PR_BGD : 배경일 가능성이 높은 영역
    # GC_PR_FGD : 전경일 가능성이 높은 영역
    #
    # 확실한 전경 + 전경 가능성이 높은 영역
    #   → 흰색(255)
    #
    # 나머지 배경 영역
    #   → 검은색(0)
    #
    # 최종적으로 이진 Mask 생성
    # ==================================================

    raw_mask = np.where(
        (mask == cv2.GC_FGD)
        |
        (mask == cv2.GC_PR_FGD),
        255,
        0,
    ).astype(
        np.uint8
    )


    # ==================================================
    # [09] 가장 큰 Contour를 이용한 Mask 정리
    #
    # Raw Mask에는 작은 잡음이나 내부 구멍이
    # 존재할 수 있음
    #
    # 1. 외곽 Contour 검출
    # 2. 가장 큰 Contour 선택
    # 3. Contour 내부를 흰색으로 채움
    # 4. Clean Mask 생성
    #
    # 광천김 포장지처럼 하나의 큰 객체를
    # 분리하는 경우에 활용할 수 있음
    #
    # 주의:
    # 가장 큰 객체만 남기므로 작은 객체는 제거됨
    # ==================================================

    # Mask에서 바깥쪽 Contour 검출
    contours, _ = cv2.findContours(
        raw_mask,
        cv2.RETR_EXTERNAL,
        cv2.CHAIN_APPROX_SIMPLE,
    )

    # 정리된 Mask를 저장할 빈 배열 생성
    clean_mask = np.zeros_like(
        raw_mask
    )

    # Contour가 존재하는 경우
    if contours:

        # 면적이 가장 큰 Contour 선택
        largest_contour = max(
            contours,
            key=cv2.contourArea,
        )

        # 가장 큰 Contour 내부를 흰색으로 채움
        cv2.drawContours(
            clean_mask,
            [largest_contour],
            -1,
            255,
            thickness=cv2.FILLED,
        )


    # ==================================================
    # [10] Clean Mask를 이미지에 적용
    #
    # bitwise_and()를 이용하여
    # Clean Mask의 흰색 영역만 유지
    #
    # Mask 흰색(255):
    #   → 원본 픽셀 유지
    #
    # Mask 검은색(0):
    #   → 픽셀 제거 (검은색으로 표시)
    #
    # 결과:
    # 배경이 검은색인 전경 이미지 생성
    # ==================================================

    foreground = cv2.bitwise_and(
        image,
        image,
        mask=clean_mask,
    )


    # ==================================================
    # [11] Rectangle 위치 확인 이미지 생성
    #
    # GrabCut에 사용한 초기 사각형을
    # 이미지 위에 표시
    #
    # 원본 처리 이미지 복사
    # Rectangle을 노란색 선으로 표시
    #
    # 세 가지 조건의 사각형 크기를 비교
    # ==================================================

    # 이미지 복사
    preview = image.copy()

    # GrabCut 초기 Rectangle 표시
    cv2.rectangle(
        preview,
        (rect_x, rect_y),
        (
            rect_x + rect_w,
            rect_y + rect_h,
        ),
        (0, 255, 255),
        3,
    )


    # ==================================================
    # [12] 결과 저장 및 전경 비율 계산
    #
    # 조건별 네 가지 결과 이미지 저장
    #
    # 1. rect.jpg       : Rectangle 확인 이미지
    # 2. raw_mask.png   : GrabCut의 이진 Mask
    # 3. clean_mask.png : 정리된 Mask
    # 4. foreground.jpg : 전경 추출 결과
    #
    # 전체 이미지에서 전경 픽셀이
    # 차지하는 비율 계산
    #
    # 조건별 전경 비율과 실행 시간 출력
    # ==================================================

    # Rectangle 확인 이미지 저장
    cv2.imwrite(
        str(
            output_dir /
            f"{name}_rect.jpg"
        ),
        preview,
    )

    # GrabCut Raw Mask 저장
    cv2.imwrite(
        str(
            output_dir /
            f"{name}_raw_mask.png"
        ),
        raw_mask,
    )

    # Contour로 정리한 Clean Mask 저장
    cv2.imwrite(
        str(
            output_dir /
            f"{name}_clean_mask.png"
        ),
        clean_mask,
    )

    # 전경 추출 이미지 저장
    cv2.imwrite(
        str(
            output_dir /
            f"{name}_foreground.jpg"
        ),
        foreground,
    )

    # 전체 픽셀 중 전경 픽셀의 비율 계산
    foreground_ratio = (
        cv2.countNonZero(
            clean_mask
        )
        / clean_mask.size
    )

    # 조건별 검사 결과 출력
    print(
        f"{name:6s}",
        f"foreground_ratio={foreground_ratio:.4f}",
        f"time={elapsed_ms:.2f} ms",
    )
```

### 의사코드

```text
[01] 라이브러리 불러오기
    → 파일 경로, 실행 시간, 이미지 처리를 위한 라이브러리 준비

[02] 입력 이미지와 결과 저장 경로 설정
    → GrabCut을 적용할 이미지 경로 지정
    → 결과를 저장할 폴더 생성

[03] 원본 이미지 읽기
    → 이미지 파일을 불러오기
    → 이미지가 없으면 오류 발생
    → 원본 이미지의 가로·세로 크기 확인

[04] GrabCut 처리용 이미지 크기 조절
    → 원본 이미지의 가로 크기가 900px보다 크면 축소
    → 원본 비율을 유지하면서 크기 조절
    → 처리용 이미지를 파일로 저장

[05] GrabCut 초기 Rectangle 영역 설정
    → wide, medium, tight 세 가지 조건 준비
    → 이미지 크기에 비례하여 여백 계산
    → 각 조건에 맞는 사각형 좌표 계산
    → 사각형이 유효한지 확인

[06] GrabCut 실행에 필요한 Mask와 모델 준비
    → 이미지 크기와 같은 Mask 생성
    → 배경 모델(bg_model) 초기화
    → 전경 모델(fg_model) 초기화

[07] GrabCut 알고리즘 실행
    → Rectangle을 기준으로 전경과 배경 추정
    → GrabCut을 3회 반복(iteration)
    → 실행 시간 측정

[08] GrabCut 결과를 흑백 Mask로 변환
    → 확실한 전경과 전경일 가능성이 높은 픽셀 선택
    → 전경은 흰색(255)
    → 배경은 검은색(0)

[09] 가장 큰 Contour를 이용한 Mask 정리
    → 흑백 Mask에서 외곽선 찾기
    → 가장 큰 외곽선 선택
    → 외곽선 내부를 흰색으로 채우기
    → 작은 잡음과 내부 구멍을 정리한 Clean Mask 생성

[10] Clean Mask를 원본 처리 이미지에 적용
    → 흰색 영역의 원본 픽셀 유지
    → 검은색 영역의 픽셀 제거
    → 배경이 검은색인 전경 이미지 생성

[11] Rectangle 위치 확인 이미지 생성
    → 처리 이미지 복사
    → GrabCut에 사용한 사각형 그리기
    → Rectangle의 위치와 크기 확인

[12] 결과 저장 및 전경 비율 계산
    → Rectangle 확인 이미지 저장
    → Raw Mask 저장
    → Clean Mask 저장
    → Foreground 이미지 저장
    → 전체 이미지에서 전경이 차지하는 비율 계산
    → 전경 비율과 GrabCut 실행 시간 출력

※ [05]~[12] 단계는 wide, medium, tight 조건마다 반복
```

실행합니다.

```bash
python src/25_grabcut.py
```

결과 생성
```
outputs/day04/grabcut/
│
├── resized_input.jpg
│
├── wide_rect.jpg
├── wide_raw_mask.png
├── wide_clean_mask.png
├── wide_foreground.jpg
│
├── medium_rect.jpg
├── medium_raw_mask.png
├── medium_clean_mask.png
├── medium_foreground.jpg
│
├── tight_rect.jpg
├── tight_raw_mask.png
├── tight_clean_mask.png
└── tight_foreground.jpg
```

각 결과의 의미는 다음과 같습니다.

```
*_rect.jpg
→ GrabCut에게 알려 준
   초기 Rectangle

*_raw_mask.png
→ GrabCut이 처음 만든 Mask
→ 제품 내부에 검은 구멍이나
   작은 오류가 있을 수 있음

*_clean_mask.png
→ 가장 큰 제품 외곽을 찾아
   내부를 흰색으로 정리한 Mask

*_foreground.jpg
→ Clean Mask를 적용하여
   광천김 포장지만 남긴 결과
```

`Raw Mask`와 `Clean Mask`를 모두 저장하는 이유는

```
GrabCut 결과
        ↓
왜 후처리가 필요했는가?
        ↓
Contour를 이용하면
어떻게 정리되는가?
```

를 직접 비교하기 위해서입니다.


---

# 15. GrabCut 결과를 확인합니다

다음 폴더의 결과를 확인합니다.

```text
outputs/day04/grabcut/
```

이번 실습에서는 하나의 Rectangle마다 다음 네 가지 결과가 만들어졌습니다.

![[{DDE66B21-0DA1-4D3E-AAFA-82ADDB8FF130}.png]]

이 코드는 `wide`, `medium`, `tight` 세 가지 조건을 각각 독립적으로 실행하도록 작성되어 있습니다. 핵심은 다음 코드입니다.

```python
margin_ratios = [
    ("wide", 0.03),
    ("medium", 0.07),
    ("tight", 0.12),
]

for name, margin_ratio in margin_ratios:
```

| 구분     | 이미지 가장자리 여백 | GrabCut 초기 사각형 |
| ------ | ----------- | -------------- |
| wide   | 3%          | 가장 넓음          |
| medium | 7%          | 중간             |
| tight  | 12%         | 가장 좁음          |
```
같은 원본 이미지
       ↓
┌─────────────────┐
│ wide (여백 3%)  │
│ GrabCut 실행    │
│ Mask 생성       │
│ 전경 추출        │
└─────────────────┘
       +
┌─────────────────┐
│ medium (7%)     │
│ GrabCut 새로 실행│
│ Mask 생성       │
│ 전경 추출        │
└─────────────────┘
       +
┌─────────────────┐
│ tight (12%)     │
│ GrabCut 새로 실행│
│ Mask 생성       │
│ 전경 추출        │
└─────────────────┘
       ↓
세 조건의 결과를 각각 저장
       ↓
어느 사각형이 적절한지 비교
```

각 반복에서 `mask`, `bg_model`, `fg_model`을 새로 생성하기 때문에 앞의 검사 결과가 다음 검사에 누적되지 않습니다.

각 조건마다 Rectangle, Raw Mask, Clean Mask, Foreground의 4개 파일을 저장하므로 총 12개 결과 파일이 만들어집니다.

---
## 15.1 Raw Mask에서는 제품 내부가 일부 검게 보입니다

`wide_raw_mask.png`, `medium_raw_mask.png`를 보면 광천김 포장지 대부분은 흰색으로 분리되었습니다.

하지만 포장지 안쪽의 일부 글자나 어두운 영역이 검은색으로 나타난 부분도 있습니다.

```text
흰색
→ GrabCut이 전경이라고 판단한 영역

검은색
→ GrabCut이 배경이라고 판단한 영역
```

광천김 포장지에는 다음과 같이 밝기와 색상이 매우 다른 부분이 함께 있습니다.

```
흰색 포장지
빨간색 로고
검은 글자
검은 김 이미지
빛이 반사된 영역
```

따라서 GrabCut이 포장지 전체를 하나의 물체로 완벽하게 판단하지 못하고 일부 어두운 영역을 배경으로 잘못 분류할 수 있습니다.

```
광천김 포장지
        ↓
GrabCut
        ↓
제품 대부분은 전경으로 판단
        +
일부 어두운 부분은 배경으로 오인
        ↓
Raw Mask에 검은 구멍 발생
```

이것은 GrabCut이 고장 난 것이 아니라 이미지의 색상과 밝기 특성 때문에 발생할 수 있는 결과입니다.

## 15.2 Clean Mask에서는 제품 내부가 하나의 영역으로 정리됩니다

이번 실습의 목적은 광천김 포장지 안의 글자나 김 사진을 각각 분리하는 것이 아닙니다.

```
목표
→ 광천김 포장지 전체를 하나의 제품으로 분리
```

따라서 Raw Mask에서 가장 큰 외곽 Contour를 찾고 내부를 흰색으로 채웠습니다.

```
Raw Mask
        ↓
가장 큰 외곽 Contour 찾기
        ↓
Contour 내부를 흰색으로 채우기
        ↓
Clean Mask
```

실제 결과에서도 `clean_mask.png`를 보면 Raw Mask에 있던 내부의 검은 구멍이 대부분 사라지고 제품 전체가 하나의 흰색 영역으로 정리된 것을 확인할 수 있습니다.

이 부분은 3일차에서 배운 `Contour`가 새로운 문제에서도 다시 활용될 수 있다는 예입니다.

---

## 15.3 Foreground에서는 광천김 포장지만 남습니다

`*_foreground.jpg`는 Clean Mask를 원본 이미지에 적용한 결과입니다.

```
Clean Mask의 흰색
→ 원본 이미지를 남김

Clean Mask의 검은색
→ 원본 이미지를 제거
```

따라서 결과는 다음과 같이 보입니다.

```
원본
광천김 포장지 + 주변 배경

        ↓

GrabCut + Mask 정리

        ↓

Foreground
광천김 포장지는 남고
주변의 어두운 배경은 대부분 제거
```

이번 결과에서는 광천김 포장지와 주변의 검은 배경이 비교적 잘 분리되었습니다.

---

## 15.4 Rectangle을 너무 좁게 잡으면 제품 일부가 사라질 수 있습니다

`wide`, `medium`, `tight` 결과도 함께 비교합니다.

```
wide
→ Rectangle을 넉넉하게 지정

medium
→ 제품에 조금 더 가깝게 지정

tight
→ Rectangle을 더 좁게 지정
```

GrabCut에서 중요한 특징은 **초기 Rectangle 바깥 영역을 배경으로 본다는 것**입니다.

따라서 Rectangle이 제품보다 너무 작아지면:

```
제품 일부
        ↓
Rectangle 밖에 위치
        ↓
처음부터 배경으로 판단
        ↓
Foreground에서 제품 일부가 잘릴 수 있음
```

실제 결과에서도 Rectangle이 좁아질수록 제품 가장자리 일부가 사라질 수 있음을 확인할 수 있습니다.

따라서 초기 Rectangle은:

```
제품보다 너무 작지 않고

제품 전체가 들어가면서

주변 배경도 어느 정도 포함되도록
```

잡는 것이 중요합니다.

---

## 15.5 이번 실습에서 확인한 핵심

이번 결과를 한 번에 정리하면 다음과 같습니다.

```
원본 이미지
        ↓
처리하기 좋은 크기로 축소
        ↓
초기 Rectangle 지정
        ↓
GrabCut
        ↓
Raw Mask
제품 대부분은 찾았지만
내부에 일부 오류 발생
        ↓
Contour로 가장 큰 영역 정리
        ↓
Clean Mask
        ↓
Foreground
제품과 주변 배경 분리
```

> GrabCut은 한 번 실행하면 완벽한 Mask를 만들어 주는 기술이 아닙니다.
> 
> 초기 Rectangle과 이미지의 색상·밝기에 따라 결과가 달라질 수 있으며, 필요한 경우 Contour와 같은 후처리를 이용하여 결과를 정리할 수 있습니다.

---

## 15.6 그렇다면 GrabCut은 언제 사용하면 좋을까요?

|활용 분야|사용 상황|GrabCut의 역할|
|---|---|---|
|제품 배경 제거|제품 사진에 주변 배경이 함께 촬영된 경우|제품만 추출하고 배경 제거|
|산업용 머신비전|컨베이어 위 제품과 배경을 분리해야 하는 경우|제품 영역을 Mask로 추출|
|객체 면적 측정|제품이 차지하는 면적을 계산하는 경우|객체 픽셀만 추출하여 면적 계산|
|불량 검사 전처리|제품 외부 배경이 검사에 방해되는 경우|제품 영역만 남겨 후속 검사에 사용|
|이미지 합성|인물이나 상품의 배경을 변경하는 경우|전경만 분리하여 다른 배경과 합성|
|의료·과학 이미지|관심 영역과 주변 조직·배경을 구분하는 경우|관심 영역 분할을 보조|
|AI 데이터 준비|객체 영역만 분리한 학습 데이터가 필요한 경우|객체 Mask 또는 분리된 이미지 생성|
|특징 추출 전처리|특정 객체의 색상·형태만 분석하는 경우|배경 영향을 줄이고 객체 특징 분석|
GrabCut은 특히 객체의 대략적인 위치를 알고 있지만, 객체의 정확한 외곽선은 모르는 경우 유용합니다.

GrabCut과 일반적인 Mask의 차이

|구분|의미|
|---|---|
|Mask|어떤 픽셀을 사용할지 지정하는 영역 정보|
|GrabCut|전경·배경을 분석하여 Mask를 생성하는 알고리즘|
|Binary Mask|전경 255, 배경 0으로 표현한 최종 마스크|
|Foreground|Mask를 적용하여 배경을 제거한 결과 이미지|

핵심: Mask는 결과물이고, GrabCut은 그 Mask를 만드는 방법 중 하나입니다.

이번 4일차에서는 GrabCut으로 전경을 분리하는 원리를 배우는 것이 목적입니다. 이후 로고 Matching이나 5일차 Calibration에서 GrabCut 결과를 직접 사용하는 것은 아닙니다. 다만 실제 산업현장에서는 필요에 따라 GrabCut → Mask → 제품 영역 추출 → 특징 측정 또는 불량 검사로 연결할 수 있습니다.

---
# PART 16~18 — 검사 2: MeanShift로 색상 영역 단순화

---

# 16. MeanShift는 무엇을 확인하는 기술인가요?

MeanShift Filtering은 공간적으로 가깝고 색상이 비슷한 픽셀을 반복적으로 묶어 이미지의 색상 영역을 평탄하게 만듭니다.

![[Pasted image 20260830174449.png]]

```text
원본
→ 작은 색상 변화가 많음

MeanShift
→ 비슷한 색상 영역을
   더 단순한 영역으로 정리
```

오늘 사용하는 함수는 다음입니다.

```python
cv2.pyrMeanShiftFiltering()
```

주요 Parameter는:

```text
sp
→ Spatial Radius
→ 주변 위치를 얼마나 넓게 볼 것인가?

sr
→ Color Radius
→ 어느 정도 색상 차이까지
   같은 영역처럼 볼 것인가?
```

입니다.

값이 커질수록 일반적으로 더 강하게 평탄화됩니다.

하지만 너무 강하면 다음 정보도 사라질 수 있습니다.

```text
작은 글자
로고의 얇은 선
제품 Texture
작은 결함
Keypoint가 생기던 모서리
```

---

# 17. `26_meanshift.py` — `sp`, `sr`을 비교합니다

이번에는 광천김 포장지의 색상 영역을 단순화해 봅니다.

GrabCut에서는 제품과 주변 배경을 분리하기 위해:

```
samples/day04/source/grabcut_scene.jpg
```

를 사용했습니다.

광천김 포장지 안에 있는:

```
흰색 포장 영역
빨간색 로고
초록색 김
검은색 김
파란색 영역
작은 글자와 무늬
```

가 `sp`, `sr` 값에 따라 어떻게 단순해지는지를 확인하는 것이 목적입니다.

파일을 만듭니다.

```text
src/26_meanshift.py
```

```python
# ==================================================
# [01] 라이브러리 불러오기
#
# Path         : 입력 이미지 및 결과 저장 경로 관리
# perf_counter : MeanShift 실행 시간 측정
# cv2          : OpenCV 이미지 처리
# ==================================================

from pathlib import Path
from time import perf_counter

import cv2


# ==================================================
# [02] 입력 이미지 및 결과 저장 경로 설정
#
# MeanShift를 적용할 이미지 파일 경로 지정
# 결과 이미지를 저장할 폴더 경로 지정
# 저장 폴더가 없으면 자동 생성
# ==================================================

# 입력 이미지 경로
image_path = Path(
    "samples/day04/targets/target_normal.jpg"
)

# 결과 저장 폴더
output_dir = Path(
    "outputs/day04/meanshift"
)

# 저장 폴더 자동 생성
output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


# ==================================================
# [03] 원본 이미지 읽기
#
# 이미지 파일을 메모리로 불러오기
#
# 이미지가 존재하지 않거나 읽지 못하면
# FileNotFoundError 발생
#
# 원본 이미지의 가로·세로 크기 확인
# ==================================================

# 이미지 읽기
image = cv2.imread(
    str(image_path)
)

# 이미지 읽기 실패 확인
if image is None:
    raise FileNotFoundError(
        image_path
    )

# 원본 이미지 크기 확인
original_h, original_w = image.shape[:2]

# 원본 이미지 크기 출력
print(
    "Original size:",
    original_w,
    "x",
    original_h,
)


# ==================================================
# [04] MeanShift 처리용 이미지 크기 조절
#
# MeanShift는 주변 픽셀의 위치와 색상을
# 반복적으로 분석하므로 이미지가 클수록
# 처리 시간이 증가할 수 있음
#
# 가로 크기가 900px보다 큰 경우:
#   1. 축소 비율 계산
#   2. 원본 비율을 유지하여 이미지 축소
#
# 가로 크기가 900px 이하이면 기존 크기 유지
#
# 처리 이미지 크기를 출력하고
# 비교할 기준 이미지를 저장
# ==================================================

# 처리용 이미지의 최대 가로 크기
process_width = 900

# 원본 이미지가 900px보다 큰 경우
if original_w > process_width:

    # 원본 크기 대비 축소 비율 계산
    scale = (
        process_width
        / original_w
    )

    # 원본 비율을 유지한 세로 크기 계산
    process_height = int(
        original_h * scale
    )

    # 이미지 축소
    image = cv2.resize(
        image,
        (
            process_width,
            process_height,
        ),
        interpolation=cv2.INTER_AREA,
    )


# 실제 처리할 이미지 크기 확인
height, width = image.shape[:2]

# 처리 이미지 크기 출력
print(
    "Process size :",
    width,
    "x",
    height,
)


# MeanShift 결과 비교용 기준 이미지 저장
cv2.imwrite(
    str(
        output_dir /
        "resized_input.jpg"
    ),
    image,
)


# ==================================================
# [05] MeanShift Parameter 설정
#
# MeanShift는 주변 픽셀의 위치와 색상 유사성을
# 이용하여 비슷한 색상 영역을 부드럽게 만듦
#
# sp (Spatial Radius)
#   → 주변 위치를 얼마나 넓게 탐색할 것인가?
#   → 값이 커질수록 더 넓은 공간을 탐색
#
# sr (Color Radius)
#   → 어느 정도 색상 차이까지 비슷하게 볼 것인가?
#   → 값이 커질수록 색상 차이를 더 넓게 허용
#
# 네 가지 Parameter 조합을 준비하여
# 동일한 이미지에 적용하고 결과 비교
#
# (sp=10, sr=20) : 작은 탐색 범위
# (sp=20, sr=30) : 중간 탐색 범위
# (sp=30, sr=50) : 더 넓은 탐색 범위
# (sp=50, sr=80) : 가장 넓은 탐색 범위
# ==================================================

settings = [
    (10, 20),
    (20, 30),
    (30, 50),
    (50, 80),
]


# ==================================================
# [06] Parameter별 MeanShift 실행
#
# settings에 저장된 네 가지 조합을
# 하나씩 가져와 동일한 이미지에 적용
#
# pyrMeanShiftFiltering()은
# 픽셀의 공간적 위치와 색상 유사성을
# 기준으로 필터링을 수행함
#
# 처리 과정:
#   1. sp, sr 값을 가져오기
#   2. 시작 시간 기록
#   3. MeanShift 필터링 실행
#   4. 실행 시간 계산
#
# 결과:
# 색상이 비슷한 영역이 부드럽게 표현된 이미지
#
# 주의:
# 이 함수는 객체별 Label이나
# 이진 Mask를 직접 생성하지 않음
# ==================================================

# 네 가지 Parameter 조합 반복
for sp, sr in settings:

    # 실행 시작 시간 기록
    start = perf_counter()

    # MeanShift 필터링 실행
    shifted = cv2.pyrMeanShiftFiltering(
        image,
        sp=sp,
        sr=sr,
    )

    # 실행 시간 계산 (밀리초)
    elapsed_ms = (
        perf_counter()
        - start
    ) * 1000


    # ==================================================
    # [07] MeanShift 결과 이미지 저장
    #
    # 적용한 Parameter 값을 파일명에 포함
    #
    # 예:
    # meanshift_sp10_sr20.jpg
    # meanshift_sp20_sr30.jpg
    # meanshift_sp30_sr50.jpg
    # meanshift_sp50_sr80.jpg
    #
    # 결과 이미지를 JPG 파일로 저장
    # ==================================================

    # 결과 파일 경로 생성
    output_path = (
        output_dir /
        f"meanshift_sp{sp}_sr{sr}.jpg"
    )

    # MeanShift 결과 이미지 저장
    cv2.imwrite(
        str(output_path),
        shifted,
    )


    # ==================================================
    # [08] Parameter별 실행 결과 출력
    #
    # 각 Parameter 조합의 처리 시간 확인
    #
    # 출력 정보:
    #   sp   : 공간 탐색 범위
    #   sr   : 색상 탐색 범위
    #   time : MeanShift 처리 시간(ms)
    #
    # 네 가지 설정의 처리 시간을 비교
    # ==================================================

    print(
        f"sp={sp:02d} "
        f"sr={sr:02d} "
        f"time={elapsed_ms:.2f} ms"
    )


# ==================================================
# [09] 최종 결과 저장 경로 출력
#
# 네 가지 Parameter 조합의 처리가 완료되면
# 결과 이미지가 저장된 폴더 경로 출력
#
# 저장 결과:
#   resized_input.jpg
#   meanshift_sp10_sr20.jpg
#   meanshift_sp20_sr30.jpg
#   meanshift_sp30_sr50.jpg
#   meanshift_sp50_sr80.jpg
# ==================================================

print()

print(
    "MeanShift 결과 저장:",
    output_dir
)
```

### 의사코드

```text
[01] 라이브러리 불러오기
    → pathlib의 Path를 이용하여 파일 경로 관리
    → perf_counter를 이용하여 실행 시간 측정
    → OpenCV를 이용하여 이미지 처리

[02] 입력 이미지 및 결과 저장 경로 설정
    → MeanShift를 적용할 이미지 경로 지정
    → 결과 이미지가 저장될 폴더 지정
    → 저장 폴더가 없으면 자동 생성

[03] 원본 이미지 읽기
    → 이미지 파일을 불러오기
    → 이미지가 존재하지 않으면 오류 발생
    → 원본 이미지의 가로·세로 크기 확인

[04] MeanShift 처리용 이미지 크기 조절
    → 이미지의 가로 크기가 900px보다 큰지 확인
    → 900px보다 크면 원본 비율을 유지하여 축소
    → 900px 이하이면 기존 이미지 크기 유지
    → 처리용 이미지 크기 출력
    → 비교할 기준 이미지 저장

[05] MeanShift Parameter 설정
    → sp : 주변 픽셀의 공간적 탐색 범위
    → sr : 비슷한 색상으로 판단할 색상 탐색 범위
    → 네 가지 Parameter 조합 준비

       sp=10, sr=20
       sp=20, sr=30
       sp=30, sr=50
       sp=50, sr=80

[06] Parameter별 MeanShift 실행
    → 설정한 Parameter 조합을 순서대로 가져오기
    → 실행 시작 시간 기록
    → pyrMeanShiftFiltering() 실행
    → 공간적 위치와 색상 유사성을 기준으로
      비슷한 픽셀을 묶어 색상 영역을 부드럽게 만듦
    → 실행 종료 후 처리 시간 계산

[07] MeanShift 결과 이미지 저장
    → Parameter 값을 포함한 파일명 생성
    → 처리된 결과 이미지를 JPG 파일로 저장

[08] Parameter별 실행 시간 출력
    → 적용한 sp 값 출력
    → 적용한 sr 값 출력
    → 처리 시간(ms) 출력
    → [06]~[08] 단계를 총 4회 반복

[09] 최종 결과 저장 경로 출력
    → 모든 Parameter 조합의 처리가 끝났는지 확인
    → MeanShift 결과 이미지가 저장된 폴더 경로 출력
```

실행합니다.

```bash
python src/26_meanshift.py
```

결과는 다음 폴더에서 확인합니다.

```text
outputs/day04/meanshift/
├── resized_input.jpg
├── meanshift_sp10_sr20.jpg
├── meanshift_sp20_sr30.jpg
├── meanshift_sp30_sr50.jpg
└── meanshift_sp50_sr80.jpg
```

![[{C10275BB-B8C8-4189-87FD-C062FFA02AEC}.png]]

먼저 다음 이미지를 기준으로 봅니다.

```
resized_input.jpg
→ MeanShift를 적용하기 전 기준 이미지
```

그다음 네 결과를 순서대로 비교합니다.

```
resized_input.jpg
        ↓

meanshift_sp10_sr20.jpg
→ 비교적 약한 MeanShift

meanshift_sp20_sr30.jpg
→ 조금 더 강한 MeanShift

meanshift_sp30_sr50.jpg
→ 색상 영역이 더 단순해짐

meanshift_sp50_sr80.jpg
→ 강한 MeanShift
→ 작은 글자·무늬까지 사라질 수 있음
```

특히 광천김 포장지에서 다음 부분을 확대해서 비교해 봅니다.

```
광천김 빨간 로고의 글자

KC 로고의 작은 선

김의 복잡한 Texture

흰색 포장지의 작은 색상 변화

제품 가장자리

작은 영문 글자
```

확인할 핵심은:

```
sp / sr가 커질수록
        ↓
비슷한 색상이 더 크게 묶이는가?

동시에
        ↓
필요한 글자와 모서리까지
사라지고 있지는 않은가?
```

입니다.

---

## 17.3 이번 실습에서 확인할 핵심

이 실습의 목표는 가장 강한 MeanShift 값을 찾는 것이 아닙니다.

```
약한 MeanShift
→ 원본 정보가 많이 남음
→ 색상 단순화 효과는 작음

        ↓

중간 MeanShift
→ 작은 색상 변화가 줄어듦
→ 주요 모양은 어느 정도 유지

        ↓

강한 MeanShift
→ 색상 영역은 매우 단순해짐
→ 작은 글자·Texture·모서리까지
   사라질 가능성이 커짐
```

이렇게 MeanShift를 이용해 색을 단순화 하는 이유는 영상처리·검사 알고리즘이 필요한 특징을 더 안정적으로 사용할 수 있도록 불필요한 작은 색상 변화를 줄이는 전처리 방법입니다. 단, 너무 강하게 적용하면 필요한 글자. 모서리. 무늬까지 사라질수 있습니다.

현재 광천김 실습 기준으로 이해해 보면

```
원본 이미지
→ 빛 반사, 주름, 미세한 색 차이, 복잡한 Texture가 많음

        ↓ MeanShift

비슷한 색을 어느 정도 묶음
→ 색상 영역이 단순해짐
→ 불필요한 작은 변화 감소

        ↓

뒤의 검사 방법
Template / ORB / SIFT 등에서
필요한 특징을 좀 더 안정적으로 사용할 가능성
```

사진을 적당히 단순하게 정리해서 컴퓨터가 중요한 모양과 영역에 더 집중할 수 있게 하는 
"전처리"라고 이해하시면 됩니다

---

# 18. 전체 이미지 처리 실습을 마무리하고 로고 비교 실습으로 넘어갑니다

지금까지 광천김 제품 이미지를 이용하여 **GrabCut과 MeanShift**를 실습했습니다.

두 기술은 모두 OpenCV의 영상처리 방법이지만, 해결하려는 문제가 다릅니다.

|구분|GrabCut|MeanShift|
|---|---|---|
|무엇을 하나요?|제품과 배경을 분리합니다.|비슷한 색상 영역을 단순화합니다.|
|어떤 문제를 해결하나요?|제품과 주변 배경이 함께 촬영된 문제|이미지에 복잡한 색상과 작은 변화가 많은 문제|
|결과물|Mask, 배경을 제거한 제품 이미지|색상 영역이 단순해진 이미지|
|활용 목적|객체 추출, 외곽선 및 면적 측정|색상 영역 분석, 후속 영상처리 전처리|

### 18-1. 두 실습은 다음 로고 비교 실습과 어떻게 연결될까요?

**GrabCut과 MeanShift는 각각의 영상처리 방법을 이해하기 위한 독립적인 실습입니다.**

이번 수업에서는 GrabCut으로 만든 Mask나 MeanShift로 단순화한 이미지를 다음 로고 Matching의 입력으로 직접 사용하지 않습니다.

이제부터는 검사 목적이 달라집니다.

앞에서는 **제품 이미지 자체를 어떻게 처리할 것인가?**를 실습했다면, 다음부터는 **기준 로고와 검사 이미지 속 로고가 얼마나 일치하는가?**를 확인합니다.

|앞에서 완료한 실습 A|다음에 진행할 실습 B|
|---|---|
|전체 제품 이미지 처리|기준 로고와 Target 이미지 비교|
|GrabCut·MeanShift|Template Matching·ORB·SIFT·Homography|
|제품 분리와 색상 단순화|로고의 일치 여부와 위치·회전·크기 변화 분석|
|독립적인 영상처리 실험|5일차 검사 기준 결정으로 연결되는 실험|

### 18-2. 영상처리를 선택할 때 기억해야 할 점

영상처리는 많이 적용한다고 항상 좋은 결과가 나오는 것은 아닙니다.

예를 들어 MeanShift를 너무 강하게 적용하면 색상 영역은 단순해지지만 작은 글자나 무늬도 사라질 수 있습니다. 이런 정보가 줄어들면 ORB나 SIFT처럼 특징점을 이용하는 검사에서 매칭에 불리해질 가능성이 있습니다.

다만 이번 실습에서는 **MeanShift 결과와 ORB·SIFT 성능의 관계를 직접 측정하지는 않았습니다.** 실제 영향은 별도의 비교 실험으로 확인해야 합니다.

**핵심 정리**

GrabCut과 MeanShift는 제품 이미지의 전경 분리와 색상 단순화를 배우는 실습입니다.

이후 Template Matching·ORB·SIFT·Homography는 기준 로고와 검사 이미지를 비교하는 별도의 실습이며, 이 로고 Matching 기술과 측정값이 5일차 Calibration 및 OK/NG 검사 기준 결정으로 
이어집니다.

---
# PART 19~21 — 검사 3: Template Matching으로 로고 찾기

---

# 19. Template Matching은 어떻게 로고를 찾나요?

앞에서는 GrabCut과 MeanShift로 제품 이미지를 처리하는 방법을 배웠습니다.

이번에는 **광천김 제품 이미지에서 기준 로고와 같은 모양을 찾는 Template Matching 실습**을 진행합니다.

Template Matching은 **작은 기준 이미지(Template)를 큰 검사 이미지(Target)의 여러 위치에서 비교하여 가장 비슷한 영역을 찾는 기술**입니다.

이번 실습에서는 다음 이미지를 사용합니다.

|구분|이미지|역할|
|---|---|---|
|Template|`reference_label.jpg`|찾으려는 광천김 기준 로고|
|Target|`target_normal.jpg` 등 4장|기준 로고를 찾아볼 전체 제품 이미지|

## 19.1. Template Matching은 어떤 원리로 동작하나요?

Template Matching은 기준 로고를 검사 이미지 위에서 조금씩 이동시키며, 각 위치에서 얼마나 비슷한지 계산합니다.

```
기준 광천김 로고 (Template)
           +
전체 광천김 제품 이미지 (Target)
           ↓
Target의 여러 위치에서
Template과 모양 비교
           ↓
각 위치의 유사도 점수 계산
           ↓
가장 비슷한 위치 선택
           ↓
Score + Location
```

[그림 삽입 — Template Matching 동작 원리]

![[Pasted image 20260830190034.png]]

그림 설명: 작은 기준 로고를 검사 이미지의 여러 위치와 비교하여 가장 높은 유사도 점수를 가진 영역을 찾습니다.

## 19.2. Template Matching의 결과는 무엇인가요?

컴퓨터는 다음 두 가지를 알려줍니다.

|결과|의미|
|---|---|
|**Score**|기준 로고와 얼마나 비슷한가?|
|**Location**|가장 비슷한 영역이 어디에 있는가?|

이번 실습에서는 `cv2.TM_CCOEFF_NORMED`를 사용합니다. 유사도 점수는 -1~1 범위이며, **1에 가까울수록 기준 로고와 패턴이 비슷합니다.**

- CCOEFF = Correlation Coefficient(상관계수)
- NORMED = Normalized(정규화된)

예를 들어 다음과 같은 결과가 나왔다고 가정하겠습니다.

```
Score = 0.96
Location = (510, 870)
```

이는 Target 이미지의 **(510, 870)을 왼쪽 위 시작점으로 하는 영역이 기준 로고와 가장 비슷하고, 점수는 0.96**이라는 뜻입니다.

실제 코드에서는 `cv2.matchTemplate()`으로 유사도를 계산하고, `cv2.minMaxLoc()`으로 가장 높은 점수와 위치를 찾습니다.

단, Score가 높다고 해서 반드시 올바른 로고를 찾았다는 뜻은 아닙니다. **결과 이미지의 사각형이 실제 로고를 감싸고 있는지도 함께 확인해야 합니다.**

## 19.3. 왜 Target 이미지 4장을 비교하나요?

실제 공장에서는 제품이 항상 같은 위치와 각도로 촬영되지 않습니다.

그래서 앞에서 준비한 광천김 이미지 4장을 이용해 **위치·회전·크기 변화에 따라 Template Matching 결과가 어떻게 달라지는지** 확인합니다.

|이미지|변경 조건|확인할 내용|
|---|---|---|
|`target_normal.jpg`|정상|원래 로고를 잘 찾는가?|
|`target_shift.jpg`|위치 이동|위치가 이동해도 찾는가?|
|`target_rot15.jpg`|15° 회전|회전하면 점수가 달라지는가?|
|`target_scale90.jpg`|90% 크기로 축소|크기가 작아져도 찾는가?|

네 이미지에는 **동일한** `**reference_label.jpg**`**를 사용**합니다.

일반적으로 Template Matching은 위치 이동에는 대응할 수 있지만, 회전이나 크기 변화가 있으면 유사도가 낮아질 수 있습니다.

실제로 어떤 결과가 나오는지는 다음 코딩 실습에서 확인하겠습니다.

### 이번 실습에서 확인할 내용

1. 네 이미지에서 광천김 로고를 정확하게 찾았나요?
    
2. 어떤 조건에서 Score가 높거나 낮아졌나요?
    
3. 검출된 사각형이 실제 광천김 로고 위치와 일치하나요?
    

**핵심 정리**

Template Matching은 **기준 로고와 비슷한 영역을 찾고, 유사도(Score)와 위치(Location)를 측정하는 기술**입니다.

이번 실습에서는 아직 OK/NG 판정을 하지 않습니다. 네 이미지의 결과를 비교하여 Template Matching의 장점과 한계를 이해하는 것이 목표입니다.

---

# 20. `27_template_matching.py` — 여러 Target의 Score와 위치를 측정합니다

앞에서 다음 데이터를 만들었습니다.

```
Reference 
samples/day04/reference/reference_label.jpg

Targets
samples/day04/targets/
├── target_normal.jpg
├── target_shift.jpg
├── target_rot15.jpg
└── target_scale90.jpg
```

이번에는 **같은 `reference_label.jpg`를 네 개의 Target에 각각 적용**합니다.

![[Pasted image 20261027174606.png]]

```
                    reference_label.jpg
                            ↓
              ┌─────────────┼─────────────┼──────────────┐
              ↓             ↓             ↓              ↓
          normal          shift         rot15        scale90
              ↓             ↓             ↓             ↓
        Template       Template       Template       Template
        Matching       Matching       Matching       Matching
              ↓             ↓             ↓             ↓
           Score          Score          Score          Score
         Location       Location       Location       Location
```

비교할 값은 두 가지입니다.

```
Score
→ 기준 로고와 얼마나 비슷한가?

Location
→ Target의 어디에서 가장 비슷하게 찾았는가?
```

파일을 만듭니다.

```text
src/27_template_matching.py
```

```python
from pathlib import Path
from time import perf_counter
import csv

import cv2


# --------------------------------------------------
# 1. 사용할 이미지와 결과 저장 경로를 설정합니다.
#
# template_path
# → 앞의 24번 코드에서 잘라낸 광천김 기준 로고 ROI
#
# target_dir
# → 정상·이동·회전·축소 Target 이미지 4장이 있는 폴더
#
# output_dir
# → 로고를 찾은 위치를 사각형으로 표시한 이미지 저장
#
# report_path
# → 유사도 Score와 로고 위치 등을 CSV로 저장
#
# mkdir()
# → 결과 저장 폴더가 없으면 생성합니다.
# --------------------------------------------------

template_path = Path(
    "samples/day04/reference/reference_label.jpg"
)

target_dir = Path(
    "samples/day04/targets"
)

output_dir = Path(
    "outputs/day04/template"
)

report_path = Path(
    "reports/day04_template_results.csv"
)


output_dir.mkdir(
    parents=True,
    exist_ok=True,
)

report_path.parent.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 2. 비교할 Target 이미지 4장을 순서대로 지정합니다.
#
# normal
# → 원본과 동일한 정상 이미지
#
# shift
# → 위치를 이동한 이미지
#
# rot15
# → 15도 회전한 이미지
#
# scale90
# → 원본의 90% 크기로 축소한 이미지
#
# 동일한 기준 로고를 네 이미지에서 각각 찾아
# 검사 조건에 따른 결과 차이를 비교합니다.
# --------------------------------------------------

target_settings = [
    (
        "normal",
        target_dir / "target_normal.jpg",
    ),
    (
        "shift",
        target_dir / "target_shift.jpg",
    ),
    (
        "rot15",
        target_dir / "target_rot15.jpg",
    ),
    (
        "scale90",
        target_dir / "target_scale90.jpg",
    ),
]


# --------------------------------------------------
# 3. 광천김 기준 로고 Template을 Gray로 읽습니다.
#
# cv2.IMREAD_GRAYSCALE
# → 이미지를 Gray 형태로 읽습니다.
#
# Template
# → 전체 제품 이미지에서 찾으려는 작은 기준 로고
#
# 이미지를 읽지 못하면 오류를 발생시킵니다.
#
# template_h → 기준 로고의 높이
# template_w → 기준 로고의 너비
#
# 이 크기는 나중에 검출 위치에
# 사각형을 그릴 때 사용합니다.
# --------------------------------------------------

template = cv2.imread(
    str(template_path),
    cv2.IMREAD_GRAYSCALE,
)

if template is None:
    raise FileNotFoundError(
        template_path
    )


template_h, template_w = (
    template.shape[:2]
)

print(
    "Template size:",
    template_w,
    "x",
    template_h,
)


# --------------------------------------------------
# 4. 네 이미지의 검사 결과를 저장할 리스트를 만듭니다.
#
# rows = []
# → 처음에는 데이터가 없는 빈 리스트
#
# 각 Target에서 측정한
# Score, Location, 처리시간 등을
# 하나씩 추가합니다.
#
# 마지막 11번에서 이 리스트를 CSV로 저장합니다.
# --------------------------------------------------

rows = []


# --------------------------------------------------
# 5. Target 이미지 4장을 하나씩 읽고 Gray로 변환합니다.
#
# for 반복문
# → normal → shift → rot15 → scale90
#   순서로 하나씩 처리합니다.
#
# cv2.imread()
# → 전체 제품 이미지를 읽습니다.
#
# cv2.cvtColor()
# → 컬러 BGR 이미지를 Gray로 변환합니다.
#
# Template과 Target을 모두 Gray로 준비해야
# 같은 방식으로 밝기 패턴을 비교할 수 있습니다.
#
# Template이 Target보다 크면
# Template Matching을 실행할 수 없으므로
# 크기를 확인합니다.
# --------------------------------------------------

for condition, target_path in target_settings:

    target = cv2.imread(
        str(target_path)
    )

    if target is None:
        raise FileNotFoundError(
            target_path
        )


    gray = cv2.cvtColor(
        target,
        cv2.COLOR_BGR2GRAY,
    )


    target_h, target_w = (
        gray.shape[:2]
    )


    if (
        template_w > target_w
        or template_h > target_h
    ):
        raise ValueError(
            f"Template이 Target보다 큽니다: "
            f"{target_path.name}"
        )


    # --------------------------------------------------
    # 6. Template Matching으로 유사도와 위치를 찾습니다.
    #
    # 이번 코드의 핵심 기능입니다.
    #
    # cv2.matchTemplate()
    # → 작은 기준 로고를 전체 Target 이미지의
    #   여러 위치에 대입하여 유사도를 계산합니다.
    #
    # cv2.TM_CCOEFF_NORMED
    # → 정규화된 상관계수 방식으로 비교합니다.
    #
    # score_map
    # → 각 위치에서 계산한 유사도 점수의 모음
    #
    # cv2.minMaxLoc()
    # → 가장 높은 Score와 해당 위치를 찾습니다.
    #
    # max_score
    # → 기준 로고와 가장 비슷한 정도
    #
    # max_loc
    # → 가장 높은 Score가 나온 왼쪽 위 좌표
    #
    # perf_counter()
    # → 처리시간을 측정합니다.
    # --------------------------------------------------

    start = perf_counter()


    score_map = cv2.matchTemplate(
        gray,
        template,
        cv2.TM_CCOEFF_NORMED,
    )


    _, max_score, _, max_loc = (
        cv2.minMaxLoc(
            score_map
        )
    )


    elapsed_ms = (
        perf_counter() - start
    ) * 1000


    # --------------------------------------------------
    # 7. 가장 비슷한 로고 영역의 사각형 좌표를 계산합니다.
    #
    # max_loc
    # → 검출된 영역의 왼쪽 위 좌표
    #
    # top_left
    # → 사각형의 왼쪽 위
    #
    # bottom_right
    # → Template 너비와 높이를 더한 오른쪽 아래
    #
    # center_x, center_y
    # → 찾은 영역의 중심 좌표
    #
    # 이 좌표로 Target 이미지에서
    # 로고가 어디에 있는지 표시할 수 있습니다.
    # --------------------------------------------------

    top_left = max_loc

    bottom_right = (
        top_left[0] + template_w,
        top_left[1] + template_h,
    )


    center_x = (
        top_left[0]
        + template_w / 2
    )

    center_y = (
        top_left[1]
        + template_h / 2
    )


    # --------------------------------------------------
    # 8. 찾은 로고 위치를 Target 이미지에 표시합니다.
    #
    # target.copy()
    # → 원본을 변경하지 않기 위해 복사합니다.
    #
    # cv2.rectangle()
    # → 가장 비슷하다고 판단한 위치에
    #   초록색 사각형을 그립니다.
    #
    # cv2.putText()
    # → 사각형 주변에 Score를 표시합니다.
    #
    # 주의:
    # 초록색 사각형이 그려졌다고 해서
    # 실제 로고를 정확하게 찾았다는 뜻은 아닙니다.
    #
    # 결과를 열어 사각형이 올바른 로고를
    # 감싸는지 직접 확인해야 합니다.
    # --------------------------------------------------

    preview = target.copy()


    cv2.rectangle(
        preview,
        top_left,
        bottom_right,
        (0, 255, 0),
        3,
    )


    cv2.putText(
        preview,
        f"score={max_score:.3f}",
        (
            top_left[0],
            max(
                30,
                top_left[1] - 10,
            ),
        ),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.8,
        (0, 255, 0),
        2,
        cv2.LINE_AA,
    )


    # --------------------------------------------------
    # 9. 검출 결과 이미지를 파일로 저장합니다.
    #
    # 각 Target에서 찾은 로고의 위치와
    # 유사도 Score가 표시된 이미지를 저장합니다.
    #
    # normal_template.jpg
    # shift_template.jpg
    # rot15_template.jpg
    # scale90_template.jpg
    #
    # 이 결과들은 학생이 직접 열어
    # 검출 위치가 맞는지 확인하는 데 사용합니다.
    # --------------------------------------------------

    output_path = (
        output_dir
        / f"{condition}_template.jpg"
    )


    cv2.imwrite(
        str(output_path),
        preview,
    )


    # --------------------------------------------------
    # 10. 측정한 Score와 Location을 리스트에 기록합니다.
    #
    # condition
    # → 정상 / 이동 / 회전 / 축소 조건
    #
    # filename
    # → 사용한 Target 파일명
    #
    # template_score
    # → 가장 비슷한 영역의 유사도 점수
    #
    # x, y
    # → 찾은 영역의 왼쪽 위 좌표
    #
    # center_x, center_y
    # → 찾은 영역의 중심 좌표
    #
    # time_ms
    # → Template Matching 처리시간(ms)
    #
    # rows.append()
    # → 현재 Target의 측정 결과를 리스트에 추가
    #
    # print()
    # → 터미널에 현재 결과를 출력
    #
    # for문이 반복되면서
    # 네 Target의 결과가 모두 rows에 저장됩니다.
    # --------------------------------------------------

    rows.append({
        "condition": condition,
        "filename": target_path.name,
        "template_score": f"{max_score:.6f}",
        "x": top_left[0],
        "y": top_left[1],
        "center_x": f"{center_x:.2f}",
        "center_y": f"{center_y:.2f}",
        "time_ms": f"{elapsed_ms:.3f}",
    })


    print(
        f"{condition:7s}",
        f"score={max_score:.4f}",
        f"location={top_left}",
        f"time={elapsed_ms:.2f} ms",
    )


# --------------------------------------------------
# 11. 네 개 Target의 측정 결과를 CSV 파일로 저장합니다.
#
# for문이 모두 끝난 후 실행됩니다.
#
# csv.DictWriter()
# → 딕셔너리 형태의 데이터를 CSV로 저장
#
# writer.writeheader()
# → CSV의 첫 번째 행에 항목 이름 작성
#
# writer.writerows(rows)
# → normal / shift / rot15 / scale90
#   네 조건의 측정 결과를 저장
#
# 최종 CSV:
# reports/day04_template_results.csv
#
# 저장한 CSV에서는
# 조건별 Score와 Location을 비교할 수 있습니다.
# --------------------------------------------------

with report_path.open(
    "w",
    newline="",
    encoding="utf-8",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=[
            "condition",
            "filename",
            "template_score",
            "x",
            "y",
            "center_x",
            "center_y",
            "time_ms",
        ],
    )

    writer.writeheader()
    writer.writerows(rows)


print()
print(
    "CSV 저장:",
    report_path,
)

print(
    "결과 이미지:",
    output_dir,
)
```

### 의사코드

```text
[1] 사용할 이미지와 저장 경로 설정

    기준 로고 ROI의 파일 경로를 정한다.

    비교할 Target 이미지들이 들어 있는
    폴더 경로를 정한다.

    검사 결과 이미지를 저장할 폴더와
    CSV 파일 경로를 정한다.

    저장 폴더가 없으면 생성한다.


[2] 비교할 Target 이미지 4장 준비

    정상 이미지
    target_normal.jpg

    위치 이동 이미지
    target_shift.jpg

    15도 회전 이미지
    target_rot15.jpg

    90% 크기 이미지
    target_scale90.jpg

    네 이미지를 순서대로 비교할 수 있게 준비한다.


[3] 기준 로고 Template 읽기

    reference_label.jpg를 Gray로 읽는다.

    파일이 정상적으로 읽혔는지 확인한다.

    기준 로고의 가로와 세로 크기를 확인한다.

    이 기준 로고를 전체 Target 이미지에서 찾는다.


[4] 검사 결과를 저장할 리스트 준비

    rows라는 빈 리스트를 만든다.

    이후 네 Target의 측정 결과를
    하나씩 추가할 예정이다.


[5] Target 이미지 하나씩 읽기

    첫 번째 정상 Target을 읽는다.

    컬러 이미지를 Gray로 변환한다.

    Target의 가로와 세로 크기를 확인한다.

    Template이 Target보다 크지 않은지 검사한다.

    문제가 없다면 다음 단계로 진행한다.


[6] Template Matching 실행

    작은 기준 로고를
    전체 Target 이미지의 여러 위치에서 비교한다.

    각 위치의 유사도 Score를 계산한다.

    가장 높은 유사도 Score를 찾는다.

    가장 높은 Score가 나온 위치를 찾는다.

    Template Matching에 걸린 시간도 측정한다.


[7] 로고 검출 영역의 좌표 계산

    가장 높은 Score가 나온 위치를
    사각형의 왼쪽 위 좌표로 정한다.

    Template의 가로와 세로 크기를 이용해
    오른쪽 아래 좌표를 계산한다.

    찾은 로고 영역의 중심 좌표도 계산한다.


[8] 검출 위치와 Score를 이미지에 표시

    Target 이미지를 복사한다.

    가장 비슷한 영역에
    초록색 사각형을 그린다.

    사각형 주변에 Score를 표시한다.

    사각형이 실제 광천김 로고를 감싸는지
    나중에 결과 이미지로 확인한다.


[9] 검사 결과 이미지 저장

    현재 Target에 대한 검사 결과를 저장한다.

    정상 → normal_template.jpg
    이동 → shift_template.jpg
    회전 → rot15_template.jpg
    축소 → scale90_template.jpg

    네 조건의 이미지에서
    사각형 위치가 어떻게 다른지 비교할 수 있다.


[10] 측정 결과 기록 및 반복

    현재 Target의 조건을 기록한다.

    유사도 Score를 기록한다.

    로고를 찾은 위치를 기록한다.

    로고 중심 좌표를 기록한다.

    처리시간을 기록한다.

    기록한 값을 rows 리스트에 추가한다.

    터미널에도 결과를 출력한다.

    다음 Target으로 이동하여
    5번부터 10번까지 같은 작업을 반복한다.

    네 Target을 모두 검사하면 반복을 종료한다.


[11] 네 Target의 측정 결과를 CSV로 저장

    rows에 저장된
    네 조건의 측정 결과를 가져온다.

    CSV 파일의 항목 이름을 작성한다.

    normal / shift / rot15 / scale90의
    Score, Location, 처리시간을 저장한다.

    day04_template_results.csv 파일을 생성한다.

    학생은 이 CSV를 열어
    조건별 Template Matching 결과를 비교한다.
```

실행합니다.

```bash
python src/27_template_matching.py
```

결과 확인

아래 폴더에 4개의 파일이 만들어집니다.

```
outputs/day04/template/

normal_template.jpg
shift_template.jpg
rot15_template.jpg
scale90_template.jpg
```

![[{7706CCC9-A1E2-4783-B8CE-95382E7F85B7}.png]]

결과

```
Original size: 1514 x 1068
Process size : 900 x 634
sp=10 sr=20 time=791.85 ms
sp=20 sr=30 time=2698.16 ms
sp=30 sr=50 time=4441.80 ms
sp=50 sr=80 time=7999.65 ms

MeanShift 결과 저장: outputs/day04/meanshift
(.venv) (.venv) youjung@DESKTOP-SPK2PPO:~/ai_vision/subject9_machinevision$ python src/27_template_matching.py
Template size: 440 x 198
normal  score=0.9997 location=(510, 870) time=155.63 ms
shift   score=0.4534 location=(788, 359) time=25.40 ms
rot15   score=0.4750 location=(684, 680) time=23.28 ms
scale90 score=0.6240 location=(505, 825) time=20.27 ms

CSV 저장: reports/day04_template_results.csv
```

따라서 단순히 숫자만 보는 것이 아니라:

```
Score가 높은가?
        +
초록색 Rectangle이
실제 KC + (주)광천김 로고를 감싸고 있는가?
```

를 함께 확인합니다.

### 숫자 결과

다음 CSV에 저장됩니다.

```
reports/day04_template_results.csv
```

확인합니다.


---

# 21. Template Matching 결과를 비교합니다

먼저 실제 측정 결과를 확인합니다.

```bash
cat reports/day04_template_results.csv
```

이번 광천김 이미지에서는 다음과 같은 결과가 나왔습니다.

| 조건        |    Score | Location     | 결과                         |
| --------- | -------: | ------------ | -------------------------- |
| `normal`  | `0.9997` | `(510, 870)` | 실제 로고 위치를 정확하게 찾음          |
| `shift`   | `0.4534` | `(788, 359)` | 실제 로고가 아닌 다른 영역을 찾음        |
| `rot15`   | `0.4750` | `(684, 680)` | 회전으로 인해 Matching 성능이 크게 감소 |
| `scale90` | `0.6240` | `(505, 825)` | 실제 로고 근처를 찾았지만 Score 감소    |
결과 이미지에서도 초록색 Rectangle이 어디에 표시되었는지 함께 확인합니다.

```
Score
→ 얼마나 비슷하다고 판단했는가?

Rectangle 위치
→ 실제 광천김 로고를 찾았는가?
```

따라서 Score 숫자만 보는 것이 아니라 Score와 실제 검출 위치를 함께 확인해야 합니다.

## 21.1 `target_normal` — 기준 이미지는 정확하게 찾았습니다

결과:

```
Score = 0.9997
Location = (510, 870)
```

`reference_label.jpg`는 원래 `full_front.jpg`의:

```
(510, 870)
```

결과 이미지를 보면 초록색 사각형이 실제 `KC + (주)광천김` 로고를 정확하게 감싸고 있습니다.

![[Pasted image 20260830195601.png]]

즉 기준 상태에서는 Template Matching이 매우 잘 동작했습니다.

## 21.2 `target_shift` — 이번 결과에서는 로고를 제대로 찾지 못했습니다

결과:

```
Score = 0.4534
Location = (788, 359)
```

결과 이미지를 보면 초록색 사각형이 실제 광천김 로고가 아니라 이미지의 다른 부분에 표시되었습니다.

```
실제 로고 위치
        ≠
Template Matching이 찾은 위치
```

따라서 이번 `shift` 결과는 정확하게 찾지 못한 경우입니다.

Score도 기준 이미지의 `0.9997`보다 크게 낮아졌습니다.

> 이번 결과처럼 Score 숫자만 보는 것이 아니라, 초록색 사각형이 실제 찾으려는 로고를 감싸고 있는지도 반드시 함께 확인합니다.


## 21.3 `target_rot15` — 회전하자 로고를 제대로 찾지 못했습니다

결과:

```
Score = 0.4750
Location = (684, 680)
```

광천김 이미지를 `15°` 회전시키자 결과가 크게 달라졌습니다.

결과 이미지를 보면 초록색 사각형이 실제 로고 위치를 정확하게 찾지 못했습니다.

```
normal
Score = 0.9997
→ 정확하게 찾음

rot15
Score = 0.4750
→ Score 감소
→ 잘못된 위치를 찾음
```

즉 이번 결과에서는 회전이 생기자 Template Matching이 크게 영향을 받았습니다.

## 21.4 `target_scale90` — 크기가 달라지자 Score가 낮아졌습니다

결과:

```
Score = 0.6240
Location = (505, 825)
```

이번에는 전체 이미지를 `90%` 크기로 줄였습니다.

결과 이미지를 보면 초록색 사각형이 광천김 로고 근처를 찾기는 했지만, 기준 이미지처럼 정확하게 일치하지는 않습니다.

```
normal
Score = 0.9997

        ↓ 크기 변경

scale90
Score = 0.6240
```

즉 **같은 로고라도 크기가 달라지면 Template Matching Score가 낮아질 수 있음**을 확인했습니다.

## 21.5 네 결과를 비교합니다

이번 실습 결과를 간단하게 정리하면 다음과 같습니다.

|조건|Score|결과|
|---|---|---|
|`normal`|`0.9997`|로고를 정확하게 찾음|
|`shift`|`0.4534`|다른 위치를 찾음|
|`rot15`|`0.4750`|회전 후 다른 위치를 찾음|
|`scale90`|`0.6240`|로고 근처는 찾았지만 Score 감소|
이번 실습에서 가장 중요한 것은 
Template Matching 결과는 Score만 보지 않고, 실제로 찾은 위치가 맞는지도 함께 확인합니다.

---

이 실험을 한 이유는 광천김 로고의 위치·회전·크기가 달라져도 Template Matching이 로고를 정확하게 찾을 수 있는지 확인하기 위해서입니다.

핵심: 일부러 이미지를 이동·회전·축소한 것은 불량 이미지를 만들기 위해서가 아니라, Template Matching의 장점과 한계를 확인하고 ORB·SIFT를 배우는 이유를 이해하기 위해서입니다.

---
## 21.6 아직 Score로 OK / NG를 판단하지 않습니다

현재 측정값은:

```
0.9997
0.4534
0.4750
0.6240
```

입니다.

하지만 아직:

```
if template_score >= 0.80:
    result = "OK"
else:
    result = "NG"
```

처럼 기준을 만들지는 않습니다.

좀더 실험을 해보고 싶으면 아래 위치의 코드를 수정하여 다시 실행을 해봅니다.

```python
# 오른쪽으로 35px, 위쪽으로 20px 이동
shift_x = 35
shift_y = -20
```

그다음 순서대로 재실행합니다.

```bash
python src/24_day04_prepare.py
python src/27_template_matching.py
```

4일차에서는:

```
이미지 조건 변경
        ↓
Score가 어떻게 달라지는지 확인
        ↓
어떤 조건에서 잘 찾고
어떤 조건에서 실패하는지 관찰
```

하는 것이 목적입니다.

실제 OK / NG 기준은 5일차에 여러 이미지를 측정한 뒤 결정합니다.

---
# PART 22~26 — 검사 4: ORB 특징점 매칭

---

# 22. 이번에는 광천김 로고의 특징점을 비교합니다

이번에는 로고의 글자·모서리·선처럼 특징적인 부분을 찾아 비교하는 **ORB와 SIFT**를 배웁니다.

### 세 가지 방법은 무엇이 다른가요?

![[Pasted image 20260830182829.png]]

| 기술                    | 비교 방법                 | 주요 특징                         |
| --------------------- | --------------------- | ----------------------------- |
| **Template Matching** | 기준 이미지의 전체 패턴 비교      | 위치는 달라도 찾을 수 있지만 회전·크기 변화에 약함 |
| **ORB**               | 특징점(Keypoint)을 찾아 비교  | 빠르고 회전·크기 변화에 어느 정도 대응        |
| **SIFT**              | 특징점과 주변 특징 정보를 이용해 비교 | 회전·크기 변화에 비교적 강하지만 계산량이 큰 편   |
- **Keypoint(특징점):** 글자의 모서리나 선이 만나는 부분처럼 비교하기 좋은 지점
- **Descriptor(특징 기술자):** 특징점 주변의 모양을 컴퓨터가 비교할 수 있도록 숫자로 표현한 정보
- **Matching(매칭):** 기준 이미지와 검사 이미지에서 서로 비슷한 특징을 연결하는 과정
- **유사도 히트맵(Heatmap):** Template Matching에서 위치별 유사도를 색상으로 표현한 결과

앞의 Template Matching에서는 `reference_label.jpg`의 **전체 모양**을 Target과 비교했습니다. 
그 결과 기준 상태에서는 로고를 잘 찾았지만, 회전하거나 크기가 달라지면 Score가 낮아지는 것도 확인했습니다. 

이번에는 다른 방법을 사용해 봅니다.

```text
Template Matching
→ 로고 이미지 전체 모양을 비교

ORB
→ 로고 안의 특징적인 점들을 찾아 비교
```

ORB 실습에서도 새로운 이미지를 사용하지 않습니다.

앞에서 사용한 같은 광천김 데이터를 그대로 사용합니다.

```text
Reference
reference_label.jpg
→ KC + (주)광천김 + 영문 로고

Targets
target_normal.jpg
target_shift.jpg
target_rot15.jpg
target_scale90.jpg
```

즉 같은 네 장의 이미지를 사용해야 Template Matching과 ORB 결과를 비교할 수 있습니다.

## 22.1 Keypoint — 비교하기 좋은 특징적인 점을 찾습니다

광천김 로고를 자세히 보면 여러 특징적인 부분이 있습니다.

예를 들면:

```
KC 로고의 모서리

(주)광천김 글자의 모서리

선이 만나는 부분

영문 글자의 끝부분
```

ORB는 이런 부분을 **Keypoint**로 찾습니다.

쉽게 말하면:

> Keypoint = 이미지에서 비교하기 좋은 특징적인 위치

입니다.

![[Pasted image 20260830202138.png]]


## 22.2 Descriptor — 그 점의 주변 모양을 숫자로 표현합니다

Keypoint의 위치만 찾았다고 해서 두 이미지의 점이 같은 특징인지 바로 알 수 있는 것은 아닙니다.

그래서 각 Keypoint 주변의 모양을 컴퓨터가 비교할 수 있는 숫자로 표현합니다.

이것을 `Descriptor`라고 합니다.

```
Keypoint
→ 특징이 어디에 있는가?

Descriptor
→ 그 특징 주변이 어떻게 생겼는가?
   비교할 수 있도록 숫자로 표현
```

쉽게 연결하면:

```
광천김 Reference
        ↓
Keypoint 찾기
        ↓
Descriptor 만들기


광천김 Target
        ↓
Keypoint 찾기
        ↓
Descriptor 만들기

        ↓

서로 비슷한 Descriptor 찾기

        ↓

Matching
```

![[Pasted image 20260830202509.png]]

---

# 23. Good Match는 무엇인가요?

![[Pasted image 20261028073907.png]]

ORB가 Reference와 Target의 특징점을 비교하면 여러 개의 연결 후보가 만들어집니다.

하지만 모든 연결이 정확한 것은 아닙니다.

```
Reference 특징점
        ↓
Target 특징점과 비교
        ↓
여러 Match 후보 발생
        ↓
그중 더 믿을 만한 Match만 선택
        ↓
Good Match
```

즉:

> Good Match는 두 이미지에서 같은 특징이라고 볼 가능성이 높은 연결점입니다.

광천김으로 생각하면:

```
Reference의
'광' 글자 모서리
        ↓
Target에서도
같은 '광' 글자 모서리 발견
        ↓
Good Match
```

즉 Good Match는 여러 특징점 연결 중에서 비교적 신뢰할 수 있다고 선택한 Match입니다.

뒤의 ORB 코드에서는 이러한 연결을 고르기 위해 `Ratio Test`라는 방법을 사용합니다.

```python
if m.distance < 0.75 * n.distance:
    good_matches.append(m)
```

지금은 계산식을 외울 필요가 없습니다.

> Ratio Test는 여러 Match 후보 중에서 애매한 연결을 줄이고, 더 확실한 연결을 남기는 과정이라고 이해하면 됩니다.

---

# 24. ORB는 무엇을 확인하는 방법인가요?

![[Pasted image 20261028074107.png]]

ORB는 이미지 전체를 한 번에 비교하는 대신 여러 Keypoint를 찾아 서로 같은 특징인지 비교하는 방법입니다.

이번 광천김 실습에서는:

```
reference_label.jpg
        ↓
광천김 로고의
Keypoint 찾기
        ↓
Descriptor 만들기
        ↓

Target에서도
Keypoint + Descriptor 만들기
        ↓

서로 비교
        ↓

Good Match 찾기
```

과정을 수행합니다.

결과 이미지에서는 Reference와 Target 사이에 여러 개의 선이 나타납니다.

![[Pasted image 20260830203115.png]]

선이 연결된 부분이 ORB가 서로 비슷한 특징이라고 판단한 지점입니다.

오늘 확인할 값은 주로 다음입니다.

```
Keypoint 수
→ 이미지에서 특징점을 얼마나 찾았는가?

Good Match 수
→ Reference와 Target 사이에서
   비슷한 특징을 얼마나 찾았는가?

처리시간
→ 비교하는 데 얼마나 걸렸는가?
```

ORB는 비교적 빠른 특징점 비교 방법입니다.

코드에서는 다음과 같이 사용합니다.

```python
orb = cv2.ORB_create(
    nfeatures=1000
)
```

ORB가 만든 Descriptor는 `Hamming Distance`라는 방법으로 비교합니다.

```python
matcher = cv2.BFMatcher(
    cv2.NORM_HAMMING
)
```

여기서도 `Hamming Distance`의 계산 방법을 외울 필요는 없습니다.

> ORB가 만든 특징 정보를 서로 비교하기 위해 사용하는 거리 계산 방법이라고만 이해하면 충분합니다.

---

## 24.1 이번 ORB 실습에서 확인할 질문

앞에서 사용했던 네 장의 광천김 Target에 ORB를 똑같이 적용합니다.

```
normal
→ 기준 상태에서 특징점이 잘 연결되는가?

shift
→ 위치가 바뀌어도 특징점이 연결되는가?

rot15
→ 15° 회전해도 특징점이 남아 있는가?

scale90
→ 크기가 90%로 달라져도 특징점이 연결되는가?
```

즉 앞의 Template Matching과 같은 데이터를 사용하는 이유는 다음 결과를 비교하기 위해서입니다.

```
Template Matching
→ 이미지 전체 모양 비교

        VS

ORB
→ 이미지의 특징적인 점 비교
```

이제 `28_orb_matching.py`를 실행하여 실제 결과를 확인합니다.

---

# 25. `28_orb_matching.py` — 광천김 로고의 Good Match를 측정합니다

이번에는 앞에서 배운 과정을 실제 코드로 확인합니다.

```
광천김 Reference 
     ↓ 
Keypoint 찾기 
     ↓ 
Descriptor 만들기 광천김 Target 
     ↓ 
Keypoint 찾기 
     ↓ 
Descriptor 만들기 
     ↓  
Descriptor 비교 
     ↓ 
여러 Match 후보 
     ↓  
Ratio Test 
     ↓ 
Good Match 
     ↓ 
연결선을 그려 결과 확인
```

데이터는 Template Matching에서 사용했던 같은 데이터를 그대로 사용합니다.

```
Reference
reference_label.jpg

Targets
target_normal.jpg
target_shift.jpg
target_rot15.jpg
target_scale90.jpg
```

파일을 만듭니다.

```text
src/28_orb_matching.py
```

```python
from pathlib import Path
from time import perf_counter
import csv

import cv2


# --------------------------------------------------
# 1. 입력 이미지와 결과 저장 경로 설정
#
# 무엇을 하나요?
# - 비교 기준이 되는 Reference 이미지 위치를 지정합니다.
# - 검사할 Target 이미지 폴더를 지정합니다.
# - ORB 비교 이미지와 CSV 결과를 저장할 위치를 준비합니다.
# - 저장 폴더가 없으면 자동으로 생성합니다.
# --------------------------------------------------

reference_path = Path(
    "samples/day04/reference/reference_label.jpg"
)

target_dir = Path(
    "samples/day04/targets"
)

output_dir = Path(
    "outputs/day04/orb"
)

report_path = Path(
    "reports/day04_orb_results.csv"
)


output_dir.mkdir(
    parents=True,
    exist_ok=True,
)

report_path.parent.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 2. 비교할 Target 이미지 4장 준비
#
# 무엇을 하나요?
# - 정상, 이동, 회전, 크기 변화 이미지를 준비합니다.
# - 각 이미지에 같은 ORB 알고리즘을 적용합니다.
#
# 비교 조건
# normal  : 정상 이미지
# shift   : 위치가 이동한 이미지
# rot15   : 15도 회전한 이미지
# scale90 : 크기가 90%로 변한 이미지
# --------------------------------------------------

target_settings = [
    (
        "normal",
        target_dir / "target_normal.jpg",
    ),
    (
        "shift",
        target_dir / "target_shift.jpg",
    ),
    (
        "rot15",
        target_dir / "target_rot15.jpg",
    ),
    (
        "scale90",
        target_dir / "target_scale90.jpg",
    ),
]


# --------------------------------------------------
# 3. 기준 이미지(Reference)를 흑백으로 읽기
#
# 무엇을 하나요?
# - 광천김 로고의 기준 이미지를 불러옵니다.
# - ORB 특징점을 검출하기 위해 Grayscale로 읽습니다.
# - 파일을 읽지 못하면 오류를 발생시킵니다.
# --------------------------------------------------

reference = cv2.imread(
    str(reference_path),
    cv2.IMREAD_GRAYSCALE,
)

if reference is None:
    raise FileNotFoundError(
        reference_path
    )


# --------------------------------------------------
# 4. ORB 특징점 검출기 생성
#
# 무엇을 하나요?
# - ORB 알고리즘을 사용할 준비를 합니다.
# - nfeatures=1000은 검출할 주요 특징점 수의
#   목표 상한을 지정하는 설정입니다.
#
# ORB는 이미지의 특징적인 위치를 찾아
# 다른 이미지에서도 같은 특징이 있는지 비교합니다.
# --------------------------------------------------

orb = cv2.ORB_create(
    nfeatures=1000
)


# --------------------------------------------------
# 5. Reference의 Keypoint와 Descriptor 추출
#
# 무엇을 하나요?
# - 기준 이미지에서 특징점(Keypoint)을 찾습니다.
# - 각 특징점을 비교할 수 있는 Descriptor를 만듭니다.
#
# Keypoint   : 이미지에서 특징적인 위치
# Descriptor : 그 위치의 특징을 표현한 데이터
#
# Descriptor가 없으면 이후 비교가 불가능하므로
# 프로그램을 중단합니다.
# --------------------------------------------------

reference_keypoints, reference_descriptors = (
    orb.detectAndCompute(
        reference,
        None,
    )
)


if reference_descriptors is None:
    raise RuntimeError(
        "Reference에서 ORB Descriptor를 만들 수 없습니다."
    )


print(
    "Reference Keypoints:",
    len(reference_keypoints),
)


# --------------------------------------------------
# 6. 특징점 비교를 위한 Matcher 생성
#
# 무엇을 하나요?
# - Reference와 Target의 Descriptor를 비교할
#   BFMatcher를 준비합니다.
# - ORB는 이진 Descriptor이므로
#   Hamming Distance를 사용합니다.
#
# Hamming Distance가 작을수록
# 두 Descriptor가 비슷하다는 의미입니다.
#
# crossCheck=False:
# - 이후 knnMatch와 Ratio Test를 적용하기 위한 설정입니다.
# --------------------------------------------------

matcher = cv2.BFMatcher(
    cv2.NORM_HAMMING,
    crossCheck=False,
)


rows = []


# --------------------------------------------------
# 7. 네 개의 Target 이미지를 순서대로 읽기
#
# 무엇을 하나요?
# - Target 4장을 하나씩 반복해서 검사합니다.
# - 각 이미지를 읽고 Grayscale로 변환합니다.
# - 처리시간을 측정하기 위해 시작 시간을 기록합니다.
# --------------------------------------------------

for condition, target_path in target_settings:

    target_color = cv2.imread(
        str(target_path)
    )

    if target_color is None:
        raise FileNotFoundError(
            target_path
        )


    target_gray = cv2.cvtColor(
        target_color,
        cv2.COLOR_BGR2GRAY,
    )


    start = perf_counter()


    # --------------------------------------------------
    # 8. Target의 Keypoint와 Descriptor 추출
    #
    # 무엇을 하나요?
    # - Target 이미지에서도 ORB 특징점을 찾습니다.
    # - Reference와 비교할 Descriptor를 생성합니다.
    #
    # Reference와 Target에 동일한 ORB를 적용해야
    # 특징을 비교할 수 있습니다.
    # --------------------------------------------------

    target_keypoints, target_descriptors = (
        orb.detectAndCompute(
            target_gray,
            None,
        )
    )


    good_matches = []


    # --------------------------------------------------
    # 9. Reference와 Target의 Descriptor 비교
    #
    # 무엇을 하나요?
    # - BFMatcher의 knnMatch()를 사용합니다.
    # - Reference의 각 Descriptor에 대해
    #   가장 가까운 Target 후보 2개를 찾습니다.
    #
    # k=2:
    # - 가장 비슷한 후보와 두 번째 후보를 찾습니다.
    # - 두 후보의 거리를 다음 Ratio Test에서 비교합니다.
    # --------------------------------------------------

    if target_descriptors is not None:

        match_candidates = matcher.knnMatch(
            reference_descriptors,
            target_descriptors,
            k=2,
        )


        # --------------------------------------------------
        # 10. Ratio Test로 Good Match 선별
        #
        # 무엇을 하나요?
        # - 가장 가까운 후보와 두 번째 후보의
        #   Hamming Distance를 비교합니다.
        # - 가장 가까운 후보가 충분히 더 가까울 때만
        #   신뢰할 만한 Match로 인정합니다.
        #
        # 조건:
        # 첫 번째 거리 < 두 번째 거리 × 0.75
        #
        # 조건을 만족하면 Good Match에 추가합니다.
        # --------------------------------------------------

        for candidate in match_candidates:

            if len(candidate) < 2:
                continue


            m, n = candidate


            if m.distance < 0.75 * n.distance:

                good_matches.append(
                    m
                )


    elapsed_ms = (
        perf_counter() - start
    ) * 1000


    # --------------------------------------------------
    # 11. Good Match 정렬 및 표시 개수 제한
    #
    # 무엇을 하나요?
    # - Good Match를 Hamming Distance가 작은 순서로 정렬합니다.
    # - 더 가까운 Match가 앞에 오도록 합니다.
    # - 화면이 복잡해지지 않도록 상위 30개만 표시합니다.
    #
    # 주의:
    # - 실제 Good Match 전체 개수는 유지합니다.
    # - 화면에 표시하는 개수만 최대 30개로 제한합니다.
    # --------------------------------------------------

    good_matches = sorted(
        good_matches,
        key=lambda match: match.distance,
    )


    shown_matches = good_matches[:30]


    # --------------------------------------------------
    # 12. Reference와 Target의 특징점 연결 이미지 생성
    #
    # 무엇을 하나요?
    # - Reference와 Target을 나란히 배치합니다.
    # - 서로 대응되는 특징점을 선으로 연결합니다.
    # - Good Match가 어디에서 발생했는지 확인합니다.
    #
    # drawMatches()는 두 이미지에서 대응되는
    # Keypoint를 시각적으로 보여주는 함수입니다.
    # --------------------------------------------------

    preview = cv2.drawMatches(
        reference,
        reference_keypoints,
        target_color,
        target_keypoints,
        shown_matches,
        None,
        flags=(
            cv2.DrawMatchesFlags_NOT_DRAW_SINGLE_POINTS
        ),
    )


    # --------------------------------------------------
    # 13. ORB 특징점 비교 결과 이미지 저장
    #
    # 무엇을 하나요?
    # - 연결선이 표시된 결과 이미지를 JPG로 저장합니다.
    # - Target마다 별도의 결과 파일을 생성합니다.
    #
    # 예:
    # target_normal_orb.jpg
    # target_shift_orb.jpg
    # target_rot15_orb.jpg
    # target_scale90_orb.jpg
    # --------------------------------------------------

    output_path = (
        output_dir
        / f"{target_path.stem}_orb.jpg"
    )


    cv2.imwrite(
        str(output_path),
        preview,
    )


    # --------------------------------------------------
    # 14. Target별 ORB 측정값 기록
    #
    # 무엇을 하나요?
    # - 검사 조건과 이미지 이름을 기록합니다.
    # - Reference와 Target의 특징점 개수를 기록합니다.
    # - Ratio Test를 통과한 Good Match 개수를 기록합니다.
    # - 특징 추출 및 매칭 처리시간을 기록합니다.
    #
    # rows 리스트에 저장한 값은
    # 마지막 단계에서 CSV 파일로 저장합니다.
    #
    # 주의:
    # time_ms에는 이미지 저장과 시각화 시간이 포함되지 않습니다.
    # --------------------------------------------------

    rows.append({
        "condition": condition,
        "filename": target_path.name,
        "reference_keypoints": len(
            reference_keypoints
        ),
        "target_keypoints": len(
            target_keypoints
        ),
        "good_matches": len(
            good_matches
        ),
        "time_ms": f"{elapsed_ms:.3f}",
    })


    print(
        f"{condition:7s}",
        f"reference_kp={len(reference_keypoints)}",
        f"target_kp={len(target_keypoints)}",
        f"good={len(good_matches)}",
        f"time={elapsed_ms:.2f} ms",
    )


# --------------------------------------------------
# 15. 전체 검사 결과를 CSV 파일로 저장
#
# 무엇을 하나요?
# - 4개 Target에 대한 측정값을 하나의 CSV로 저장합니다.
# - 조건별 특징점 개수와 Good Match 수를 비교할 수 있습니다.
# - 정상, 이동, 회전, 크기 변화에 따라
#   ORB Matching 결과가 어떻게 달라지는지 분석합니다.
#
# 최종 산출물:
# reports/day04_orb_results.csv
#
# outputs/day04/orb/
#   ├── target_normal_orb.jpg
#   ├── target_shift_orb.jpg
#   ├── target_rot15_orb.jpg
#   └── target_scale90_orb.jpg
# --------------------------------------------------

with report_path.open(
    "w",
    newline="",
    encoding="utf-8",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=[
            "condition",
            "filename",
            "reference_keypoints",
            "target_keypoints",
            "good_matches",
            "time_ms",
        ],
    )

    writer.writeheader()
    writer.writerows(rows)


print()
print(
    "CSV 저장:",
    report_path,
)

print(
    "결과 이미지:",
    output_dir,
)
```
### 의사코드

```text
[1. 입력 이미지와 결과 저장 경로 설정]

기준 이미지(Reference) 경로를 정한다.
비교할 Target 이미지 폴더를 정한다.
ORB 결과 이미지와 CSV 저장 경로를 정한다.
저장 폴더가 없으면 새로 만든다.

          ↓

[2. 비교할 Target 이미지 4장 준비]

정상 이미지(normal)를 준비한다.
위치 이동 이미지(shift)를 준비한다.
15도 회전 이미지(rot15)를 준비한다.
90% 크기 이미지(scale90)를 준비한다.

          ↓

[3. 기준 이미지(Reference)를 흑백으로 읽기]

광천김 Reference 이미지를 불러온다.
흑백(Grayscale)으로 읽는다.
이미지가 없으면 오류를 발생시킨다.

          ↓

[4. ORB 특징점 검출기 생성]

ORB 알고리즘을 준비한다.
주요 특징점 수의 목표 상한을 1000으로 설정한다.

          ↓

[5. Reference의 Keypoint와 Descriptor 추출]

Reference에서 특징점 위치(Keypoint)를 찾는다.
각 특징점을 표현하는 Descriptor를 만든다.
Descriptor가 없으면 프로그램을 중단한다.
Reference 특징점 개수를 출력한다.

          ↓

[6. 특징점 비교를 위한 Matcher 생성]

BFMatcher를 준비한다.
ORB Descriptor 비교에 Hamming Distance를 사용한다.
Good Match를 저장할 리스트를 준비한다.

          ↓

[7. 네 개의 Target 이미지를 순서대로 읽기]

Target 이미지 4장을 하나씩 반복한다.
현재 Target 이미지를 불러온다.
이미지가 없으면 오류를 발생시킨다.
Target 이미지를 Grayscale로 변환한다.
처리 시작 시간을 기록한다.

          ↓

[8. Target의 Keypoint와 Descriptor 추출]

현재 Target에서 ORB 특징점을 찾는다.
각 특징점의 Descriptor를 만든다.
Good Match를 저장할 빈 리스트를 만든다.

          ↓

[9. Reference와 Target의 Descriptor 비교]

Target Descriptor가 있는지 확인한다.
Reference의 각 Descriptor에 대해
가장 가까운 Target 후보 2개를 찾는다.
Match 후보들의 Hamming Distance를 얻는다.

          ↓

[10. Ratio Test로 Good Match 선별]

Match 후보를 하나씩 확인한다.
후보가 2개 미만이면 건너뛴다.

가장 가까운 후보의 Distance와
두 번째 후보의 Distance를 비교한다.

첫 번째 거리 < 두 번째 거리 × 0.75

조건을 만족하면 Good Match에 추가한다.
특징 추출과 매칭 처리시간을 계산한다.

          ↓

[11. Good Match 정렬 및 표시 개수 제한]

Good Match를 Distance가 작은 순서로 정렬한다.
전체 Good Match 개수는 유지한다.
시각화에 사용할 상위 30개만 선택한다.

          ↓

[12. Reference와 Target의 특징점 연결 이미지 생성]

Reference와 Target 이미지를 나란히 놓는다.
대응되는 특징점을 선으로 연결한다.
ORB Matching 결과 이미지를 생성한다.

          ↓

[13. ORB 특징점 비교 결과 이미지 저장]

Target 이름을 이용해 저장 파일명을 만든다.
특징점 연결 결과 이미지를 JPG로 저장한다.

          ↓

[14. Target별 ORB 측정값 기록]

현재 Target의 검사 조건을 기록한다.
Reference 특징점 개수를 기록한다.
Target 특징점 개수를 기록한다.
Good Match 전체 개수를 기록한다.
특징 추출 및 매칭 처리시간을 기록한다.
측정 결과를 rows 리스트에 추가한다.
현재 Target의 측정 결과를 출력한다.

          ↓

아직 검사할 Target이 남아 있다면
7번으로 돌아가 다음 이미지를 검사한다.

4개 Target의 검사가 모두 끝나면
15번으로 이동한다.

          ↓

[15. 전체 검사 결과를 CSV 파일로 저장]

rows에 저장된 4개 Target 결과를 읽는다.
CSV 파일에 컬럼 제목을 작성한다.
Target별 검사 결과를 CSV로 저장한다.

CSV 저장 경로를 출력한다.
ORB 결과 이미지 폴더를 출력한다.

          ↓

[최종 결과]

① Target별 ORB Matching 이미지 4개
② ORB 측정 결과 CSV 파일 1개

          ↓

정상 / 이동 / 회전 / 크기 변화 조건에서
특징점 개수와 Good Match 개수를 비교한다.
```

실행합니다.

```bash
python src/28_orb_matching.py
```

이 프로그램에서는 다음 값을 측정합니다.

```
Reference Keypoint 수
→ 기준 광천김 로고에서 찾은 특징점 수

Target Keypoint 수
→ 검사 이미지에서 찾은 특징점 수

Good Match 수
→ Reference와 Target에서
   서로 비슷하다고 판단한 특징 연결 수

time_ms
→ ORB 특징점 비교에 걸린 시간
```

또한 Good Match 중 상위 30개를 선으로 연결한 이미지를 다음 폴더에 저장합니다.

---

# 26. ORB 결과를 확인합니다

먼저 실제 측정 결과를 확인합니다.

결과

```
Reference Keypoints: 886
normal  reference_kp=886 target_kp=1000 good=276 time=14.33 ms
shift   reference_kp=886 target_kp=1000 good=245 time=13.49 ms
rot15   reference_kp=886 target_kp=1000 good=232 time=11.83 ms
scale90 reference_kp=886 target_kp=1000 good=156 time=10.96 ms

CSV 저장: reports/day04_orb_results.csv
결과 이미지: outputs/day04/orb
```

```bash
cat reports/day04_orb_results.csv
```

이번 광천김 이미지에서는 다음과 같은 결과가 나왔습니다.

| 조건        | Reference Keypoint | Target Keypoint | Good Match |     처리시간 |
| --------- | -----------------: | --------------: | ---------: | -------: |
| `normal`  |                886 |            1000 |        276 | 14.33 ms |
| `shift`   |                886 |            1000 |        245 | 13.49 ms |
| `rot15`   |                886 |            1000 |        232 | 11.83 ms |
| `scale90` |                886 |            1000 |        156 | 10.96 ms |
결과 이미지에서 Reference와 Target 사이의 선은 **서로 같은 특징이라고 판단한 Good Match**를 나타냅니다.

결과 확인 파일

```
outputs/day04/orb/

target_normal_orb.jpg
target_rot15_orb.jpg
target_scale90_orb.jpg
target_shift_orb.jpg
```

![[{383A7D6B-A1FB-44CC-83BA-9239E7F49C14}.png]]

## 26.1 `target_normal` — 가장 많은 Good Match가 만들어졌습니다

결과:

```
Reference Keypoint = 886
Target Keypoint    = 1000
Good Match         = 276
```

기준 상태에서는 `276개`의 Good Match가 만들어졌습니다.

결과 이미지를 보면 Reference의 KC 로고와 `(주)광천김` 글자 주변 특징점이 Target의 같은 부분으로 많이 연결되어 있습니다.

```
같은 방향
같은 크기
같은 이미지
        ↓
특징점이 잘 대응
        ↓
Good Match = 276
```

이 값을 다른 조건과 비교하는 기준으로 사용합니다.

---

## 26.2 `target_shift` — 위치가 바뀌어도 많은 Match가 유지되었습니다

결과:

```
Good Match = 245
```

`normal`의 `276개`보다 조금 줄었지만 여전히 많은 특징점이 연결되었습니다.

```
normal
276

        ↓ 위치 변화

shift
245
```

결과 이미지에서도 여러 선이 광천김 로고의 같은 특징으로 연결되는 것을 확인할 수 있습니다.

즉 이번 데이터에서는 **제품 위치가 바뀌어도 ORB가 특징점을 비교할 수 있었습니다.**

앞의 Template Matching에서는 `shift` 이미지를 제대로 찾지 못했지만, ORB에서는 많은 특징점 연결이 남았습니다.

---

## 26.3 `target_rot15` — 15° 회전해도 많은 Match가 남았습니다

결과:

```
Good Match = 232
```

광천김 이미지를 `15°` 회전했지만 `232개`의 Good Match가 남았습니다.

```
normal
276

        ↓ 15° 회전

rot15
232
```

결과 이미지에서도 선들이 회전된 광천김 로고의 같은 부분으로 연결되는 것을 확인할 수 있습니다.

즉 이번 결과에서는 **ORB가 어느 정도의 회전 변화에도 특징점을 계속 비교할 수 있음**을 확인했습니다.

이 부분은 Template Matching 결과와 차이가 있습니다.

```
Template Matching
→ 15° 회전에서
   로고를 제대로 찾지 못함

ORB
→ 15° 회전에서도
   많은 Good Match 유지
```

---

## 26.4 `target_scale90` — 크기가 달라지자 Match가 가장 많이 감소했습니다

결과:

```
Good Match = 156
```

제품의 크기를 `90%`로 줄이자 Good Match 수가 가장 많이 감소했습니다.

```
normal
276

        ↓ 크기 90%

scale90
156
```

하지만 모든 Match가 사라진 것은 아닙니다.

결과 이미지에서도 광천김 로고의 여러 특징점이 계속 연결되어 있습니다.

즉 이번 데이터에서는:

```
크기가 달라짐
        ↓
Good Match 감소
        ↓
그래도 일부 특징은 계속 찾음
```

을 확인했습니다.

---

## 26.5 네 결과를 비교합니다

이번 결과를 간단하게 정리하면 다음과 같습니다.

|조건|Good Match|관찰|
|---|---|---|
|`normal`|`276`|가장 많은 특징이 연결됨|
|`shift`|`245`|위치가 달라져도 많은 Match 유지|
|`rot15`|`232`|15° 회전에서도 많은 Match 유지|
|`scale90`|`156`|크기 변화에서 Match가 많이 감소했지만 일부 유지|

이번 ORB 실습의 핵심은 다음과 같습니다.

```
Template Matching
→ 이미지 전체 모양을 비교

ORB
→ 여러 특징적인 점을 각각 비교
```

실제 결과에서도 ORB는 위치와 회전이 달라진 이미지에서 비교적 많은 특징점을 계속 연결했습니다.

---

## 26.6 Good Match 수만 보지는 않습니다

Good Match가 많다고 해서 결과가 무조건 올바른 것은 아닙니다.

결과 이미지의 연결선도 함께 확인합니다.

```
Good Match 수
        +
연결선 확인
        ↓
실제로 같은 로고의 특징끼리
연결되고 있는가?
```

이번 결과 이미지에서는 많은 선이 광천김 로고와 글자 주변의 대응되는 부분으로 연결되고 있습니다.

따라서 숫자와 결과 이미지를 함께 확인하는 것이 중요합니다.

> ORB 결과는 Good Match 개수와 실제 연결선이 올바른 특징을 연결하고 있는지를 함께 확인합니다.

---

## 26.7 다음에는 SIFT와 비교합니다

이번 ORB 결과에서는:

```
normal  → 276
shift   → 245
rot15   → 232
scale90 → 156
```

으로 측정되었습니다.

다음에는 같은 광천김 Reference와 같은 네 개의 Target에 `SIFT`를 적용합니다.

```
같은 데이터
        ↓

ORB 결과
        VS
SIFT 결과
        ↓

위치·회전·크기 변화에서
어느 방법이 어떻게 달라지는지 비교
```

따라서 다음 실습에서도 새로운 이미지를 만들지 않고 **동일한 네 장을 그대로 사용합니다.**

---
# PART 27~29 — 검사 5: SIFT 특징점 매칭

---

# 27. SIFT는 어떤 특징을 가지나요? 이번에는 같은 광천김 로고를 SIFT로 비교합니다


앞에서는 ORB를 이용하여 광천김 로고의 특징점을 비교했습니다. 이번에는 같은 데이터를 `SIFT`라는 다른 특징점 비교 방법으로 검사합니다.

```text
Reference
reference_label.jpg

Targets
target_normal.jpg
target_shift.jpg
target_rot15.jpg
target_scale90.jpg
```

즉 검사 대상은 ORB와 완전히 같습니다.

```text
ORB
→ 같은 광천김 이미지의 특징점 비교

SIFT
→ 같은 광천김 이미지의 특징점 비교
```

두 방법 모두:

```text
Keypoint 찾기
        ↓
Descriptor 만들기
        ↓
Reference와 Target 비교
        ↓
Good Match 찾기
```

라는 큰 흐름은 같습니다.

차이는 특징점을 찾고 표현하는 방법입니다.

## 27.1 SIFT는 무엇이 다른가요?

![[Pasted image 20261028074836.png]]

오늘 수준에서는 다음 정도만 이해하면 충분합니다.

```
ORB
→ 비교적 빠른 특징점 비교

SIFT
→ 회전이나 크기 변화가 있어도
   특징을 비교하는 데 강한 편
```

조금 더 기술적으로 보면:

```
ORB
→ Binary Descriptor
→ Hamming Distance

SIFT
→ 실수형 Descriptor
→ L2 Distance
```

를 사용합니다.

하지만 `Binary Descriptor`, `L2 Distance`의 계산식을 외울 필요는 없습니다.

> ORB와 SIFT가 특징점을 표현하고 비교하는 방법이 서로 다르다고 이해하면 됩니다.

이번 실습에서 중요한 질문은 다음입니다.

```
normal에서 SIFT는 얼마나 연결되는가?

shift에서도 Match가 유지되는가?

15° 회전에서도 Match가 유지되는가?

90% 크기에서도 Match가 유지되는가?

ORB 결과와 비교하면
어떤 차이가 나타나는가?
```

즉 이번 실습의 목적은:

> SIFT가 무조건 ORB보다 좋다는 것을 확인하는 것이 아니라, 같은 데이터에서 두 방법의 결과가 어떻게 달라지는지 직접 비교하는 것입니다.

---

# 28. `29_sift_matching.py` — SIFT Good Match를 측정합니다

앞에서 ORB에서 수행했던 것과 같은 순서로 진행합니다.

```
광천김 Reference 
     ↓  
SIFT Keypoint 찾기 
     ↓ 
Descriptor 만들기 광천김 Target 
     ↓ 
SIFT Keypoint 찾기 
     ↓ 
Descriptor 만들기 
     ↓ 
Descriptor 비교 
     ↓ 
Match 후보 
     ↓ 
Ratio Test 
     ↓ 
Good Match 
     ↓ 
연결선을 그려 결과 확인
```

파일을 만듭니다.

```text
src/29_sift_matching.py
```

```python
from pathlib import Path
from time import perf_counter
import csv

import cv2


# --------------------------------------------------
# 1. 입력 이미지와 결과 저장 경로 설정
#
# 무엇을 하나요?
# - 비교 기준이 되는 Reference 이미지 위치를 지정합니다.
# - 검사할 Target 이미지 폴더를 지정합니다.
# - SIFT 비교 이미지와 CSV 결과를 저장할 위치를 준비합니다.
# - 저장 폴더가 없으면 자동으로 생성합니다.
# --------------------------------------------------

reference_path = Path(
    "samples/day04/reference/reference_label.jpg"
)

target_dir = Path(
    "samples/day04/targets"
)

output_dir = Path(
    "outputs/day04/sift"
)

report_path = Path(
    "reports/day04_sift_results.csv"
)


output_dir.mkdir(
    parents=True,
    exist_ok=True,
)

report_path.parent.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 2. 비교할 Target 이미지 4장 준비
#
# 무엇을 하나요?
# - 정상, 이동, 회전, 크기 변화 이미지를 준비합니다.
# - 각 이미지에 동일한 SIFT 알고리즘을 적용합니다.
#
# 비교 조건
# normal  : 정상 이미지
# shift   : 위치가 이동한 이미지
# rot15   : 15도 회전한 이미지
# scale90 : 크기가 90%로 변한 이미지
#
# 목적:
# 이미지의 위치, 회전, 크기가 변해도
# SIFT가 같은 특징을 찾아낼 수 있는지 확인합니다.
# --------------------------------------------------

target_settings = [
    (
        "normal",
        target_dir / "target_normal.jpg",
    ),
    (
        "shift",
        target_dir / "target_shift.jpg",
    ),
    (
        "rot15",
        target_dir / "target_rot15.jpg",
    ),
    (
        "scale90",
        target_dir / "target_scale90.jpg",
    ),
]


# --------------------------------------------------
# 3. 현재 OpenCV에서 SIFT 사용 가능 여부 확인
#
# 무엇을 하나요?
# - 설치된 OpenCV에 SIFT_create 기능이 있는지 확인합니다.
#
# 왜 필요한가요?
# OpenCV 설치 버전이나 환경에 따라
# SIFT 기능을 사용할 수 없는 경우가 있기 때문입니다.
#
# SIFT_create가 없으면 프로그램을 중단합니다.
# --------------------------------------------------

if not hasattr(
    cv2,
    "SIFT_create",
):
    raise RuntimeError(
        "현재 OpenCV에서 SIFT_create를 사용할 수 없습니다."
    )


# --------------------------------------------------
# 4. 기준 이미지(Reference)를 흑백으로 읽기
#
# 무엇을 하나요?
# - 광천김 로고의 기준 이미지를 불러옵니다.
# - SIFT 특징점을 검출하기 위해 Grayscale로 읽습니다.
# - 이미지가 없거나 읽을 수 없으면 오류를 발생시킵니다.
#
# Reference는 이후 모든 Target과 비교할 기준입니다.
# --------------------------------------------------

reference = cv2.imread(
    str(reference_path),
    cv2.IMREAD_GRAYSCALE,
)

if reference is None:
    raise FileNotFoundError(
        reference_path
    )


# --------------------------------------------------
# 5. SIFT 특징점 검출기 생성
#
# 무엇을 하나요?
# - SIFT 알고리즘을 사용할 준비를 합니다.
#
# SIFT란?
# Scale-Invariant Feature Transform
#
# 이미지에서 특징적인 위치를 찾고,
# 크기나 회전 변화에도 비교할 수 있는
# 특징 정보를 만드는 알고리즘입니다.
#
# ORB와 달리 여기서는 SIFT의 기본 설정을 사용합니다.
# --------------------------------------------------

sift = cv2.SIFT_create()


# --------------------------------------------------
# 6. Reference의 Keypoint와 Descriptor 추출
#
# 무엇을 하나요?
# - Reference 이미지에서 특징점(Keypoint)을 찾습니다.
# - 각 특징점을 비교할 수 있는 Descriptor를 만듭니다.
#
# Keypoint:
# 이미지에서 특징적인 위치와 크기, 방향 등의 정보
#
# Descriptor:
# 특징점 주변의 영상 특징을 표현한 숫자 데이터
# SIFT는 일반적으로 128차원 Descriptor를 사용합니다.
#
# Descriptor가 없으면 비교가 불가능하므로 중단합니다.
# --------------------------------------------------

reference_keypoints, reference_descriptors = (
    sift.detectAndCompute(
        reference,
        None,
    )
)


if reference_descriptors is None:
    raise RuntimeError(
        "Reference에서 SIFT Descriptor를 만들 수 없습니다."
    )


print(
    "Reference Keypoints:",
    len(reference_keypoints),
)


# --------------------------------------------------
# 7. 특징점 비교를 위한 Matcher 생성
#
# 무엇을 하나요?
# - Reference와 Target의 Descriptor를 비교할
#   BFMatcher를 준비합니다.
# - SIFT는 L2 Distance를 사용하여 특징을 비교합니다.
#
# L2 Distance:
# 두 Descriptor 사이의 유클리드 거리입니다.
# 거리가 작을수록 특징이 비슷하다는 의미입니다.
#
# crossCheck=False:
# 이후 knnMatch와 Ratio Test를 사용하기 위한 설정입니다.
#
# rows:
# 각 Target의 측정 결과를 저장할 리스트입니다.
# --------------------------------------------------

matcher = cv2.BFMatcher(
    cv2.NORM_L2,
    crossCheck=False,
)


rows = []


# --------------------------------------------------
# 8. 네 개의 Target 이미지를 순서대로 읽기
#
# 무엇을 하나요?
# - 준비한 Target 이미지 4장을 하나씩 검사합니다.
# - 컬러 이미지를 읽은 후 Grayscale로 변환합니다.
# - 처리시간을 측정하기 위해 시작 시간을 기록합니다.
#
# for 반복문 내부에서는
# 9번부터 15번까지의 과정을 Target마다 실행합니다.
# --------------------------------------------------

for condition, target_path in target_settings:

    target_color = cv2.imread(
        str(target_path)
    )

    if target_color is None:
        raise FileNotFoundError(
            target_path
        )


    target_gray = cv2.cvtColor(
        target_color,
        cv2.COLOR_BGR2GRAY,
    )


    start = perf_counter()


    # --------------------------------------------------
    # 9. Target의 Keypoint와 Descriptor 추출
    #
    # 무엇을 하나요?
    # - Target 이미지에서도 SIFT 특징점을 찾습니다.
    # - Reference와 비교할 Descriptor를 생성합니다.
    #
    # Reference와 Target에 동일한 SIFT를 적용해야
    # 특징점의 Descriptor를 비교할 수 있습니다.
    #
    # good_matches:
    # Ratio Test를 통과한 대응점을 저장할 리스트입니다.
    # --------------------------------------------------

    target_keypoints, target_descriptors = (
        sift.detectAndCompute(
            target_gray,
            None,
        )
    )


    good_matches = []


    # --------------------------------------------------
    # 10. Reference와 Target의 Descriptor 비교
    #
    # 무엇을 하나요?
    # - BFMatcher의 knnMatch()를 사용합니다.
    # - Reference의 각 Descriptor에 대해
    #   가장 가까운 Target 후보 2개를 찾습니다.
    #
    # k=2:
    # 가장 가까운 후보와 두 번째 후보를 찾습니다.
    #
    # 두 후보 사이의 거리 차이를 이용해
    # 다음 Ratio Test에서 좋은 Match를 선별합니다.
    #
    # Target Descriptor가 없다면 Matching은 생략합니다.
    # --------------------------------------------------

    if target_descriptors is not None:

        match_candidates = matcher.knnMatch(
            reference_descriptors,
            target_descriptors,
            k=2,
        )


        # --------------------------------------------------
        # 11. Ratio Test로 Good Match 선별
        #
        # 무엇을 하나요?
        # - 가장 가까운 후보와 두 번째 후보의 거리를 비교합니다.
        # - 첫 번째 후보가 충분히 가까울 때만
        #   신뢰할 만한 Match 후보로 인정합니다.
        #
        # 조건:
        # 첫 번째 거리 < 두 번째 거리 × 0.75
        #
        # 조건을 만족하면 Good Match에 추가합니다.
        #
        # 목적:
        # 다른 특징점과도 비슷하게 보이는
        # 모호한 Match를 제거하는 것입니다.
        # --------------------------------------------------

        for candidate in match_candidates:

            if len(candidate) < 2:
                continue


            m, n = candidate


            if m.distance < 0.75 * n.distance:

                good_matches.append(
                    m
                )


    # 특징 추출과 Matching 처리시간을 밀리초로 계산합니다.
    # 이미지 읽기·정렬·시각화·저장 시간은 포함하지 않습니다.

    elapsed_ms = (
        perf_counter() - start
    ) * 1000


    # --------------------------------------------------
    # 12. Good Match 정렬 및 표시 개수 제한
    #
    # 무엇을 하나요?
    # - Good Match를 L2 Distance가 작은 순서로 정렬합니다.
    # - 가까운 Match가 앞에 오도록 합니다.
    # - 화면이 복잡해지지 않도록 상위 30개만 표시합니다.
    #
    # 주의:
    # 실제 Good Match 전체 개수는 그대로 유지합니다.
    # 화면에 표시하는 개수만 최대 30개로 제한합니다.
    # --------------------------------------------------

    good_matches = sorted(
        good_matches,
        key=lambda match: match.distance,
    )


    shown_matches = good_matches[:30]


    # --------------------------------------------------
    # 13. Reference와 Target의 특징점 연결 이미지 생성
    #
    # 무엇을 하나요?
    # - Reference와 Target 이미지를 나란히 배치합니다.
    # - 대응되는 특징점 사이를 선으로 연결합니다.
    # - SIFT Matching 결과를 시각적으로 보여줍니다.
    #
    # drawMatches():
    # 두 이미지 사이의 대응되는 Keypoint를
    # 선으로 연결하여 보여주는 OpenCV 함수입니다.
    #
    # 이를 통해 이동·회전·크기 변화에서도
    # 특징점이 어떻게 연결되는지 확인할 수 있습니다.
    # --------------------------------------------------

    preview = cv2.drawMatches(
        reference,
        reference_keypoints,
        target_color,
        target_keypoints,
        shown_matches,
        None,
        flags=(
            cv2.DrawMatchesFlags_NOT_DRAW_SINGLE_POINTS
        ),
    )


    # --------------------------------------------------
    # 14. SIFT 특징점 비교 결과 이미지 저장
    #
    # 무엇을 하나요?
    # - 연결선이 표시된 결과 이미지를 JPG로 저장합니다.
    # - Target마다 별도의 결과 파일을 생성합니다.
    #
    # 저장 파일 예시:
    # target_normal_sift.jpg
    # target_shift_sift.jpg
    # target_rot15_sift.jpg
    # target_scale90_sift.jpg
    # --------------------------------------------------

    output_path = (
        output_dir
        / f"{target_path.stem}_sift.jpg"
    )


    cv2.imwrite(
        str(output_path),
        preview,
    )


    # --------------------------------------------------
    # 15. Target별 SIFT 측정값 기록
    #
    # 무엇을 하나요?
    # - 검사 조건과 이미지 이름을 기록합니다.
    # - Reference와 Target의 특징점 개수를 기록합니다.
    # - Ratio Test를 통과한 Good Match 개수를 기록합니다.
    # - 특징 추출 및 Matching 처리시간을 기록합니다.
    #
    # rows 리스트에 저장한 값은
    # 마지막 16번 단계에서 CSV 파일로 저장합니다.
    # --------------------------------------------------

    rows.append({
        "condition": condition,
        "filename": target_path.name,
        "reference_keypoints": len(
            reference_keypoints
        ),
        "target_keypoints": len(
            target_keypoints
        ),
        "good_matches": len(
            good_matches
        ),
        "time_ms": f"{elapsed_ms:.3f}",
    })


    print(
        f"{condition:7s}",
        f"reference_kp={len(reference_keypoints)}",
        f"target_kp={len(target_keypoints)}",
        f"good={len(good_matches)}",
        f"time={elapsed_ms:.2f} ms",
    )


# --------------------------------------------------
# 16. 전체 검사 결과를 CSV 파일로 저장
#
# 무엇을 하나요?
# - 4개 Target의 측정값을 하나의 CSV로 저장합니다.
# - 특징점 개수와 Good Match 수를 비교할 수 있습니다.
# - 정상, 이동, 회전, 크기 변화에 따른
#   SIFT Matching 결과를 분석할 수 있습니다.
#
# 최종 산출물:
# reports/day04_sift_results.csv
#
# outputs/day04/sift/
#   ├── target_normal_sift.jpg
#   ├── target_shift_sift.jpg
#   ├── target_rot15_sift.jpg
#   └── target_scale90_sift.jpg
# --------------------------------------------------

with report_path.open(
    "w",
    newline="",
    encoding="utf-8",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=[
            "condition",
            "filename",
            "reference_keypoints",
            "target_keypoints",
            "good_matches",
            "time_ms",
        ],
    )

    writer.writeheader()
    writer.writerows(rows)


print()

print(
    "CSV 저장:",
    report_path,
)

print(
    "결과 이미지:",
    output_dir,
)
```
### 의사코드

```
[1. 입력 이미지와 결과 저장 경로 설정]

Reference 이미지 경로를 정한다.
Target 이미지 폴더를 정한다.
SIFT 결과 이미지와 CSV 저장 경로를 정한다.
저장 폴더가 없으면 새로 만든다.

          ↓

[2. 비교할 Target 이미지 4장 준비]

정상 이미지(normal)를 준비한다.
위치 이동 이미지(shift)를 준비한다.
15도 회전 이미지(rot15)를 준비한다.
90% 크기 이미지(scale90)를 준비한다.

          ↓

[3. 현재 OpenCV에서 SIFT 사용 가능 여부 확인]

설치된 OpenCV에 SIFT_create가 있는지 확인한다.
SIFT를 사용할 수 없다면 프로그램을 중단한다.

          ↓

[4. 기준 이미지(Reference)를 흑백으로 읽기]

광천김 Reference 이미지를 불러온다.
Grayscale로 읽는다.
이미지가 없으면 오류를 발생시킨다.

          ↓

[5. SIFT 특징점 검출기 생성]

SIFT 알고리즘을 준비한다.
이미지에서 특징점을 찾을 준비를 한다.

          ↓

[6. Reference의 Keypoint와 Descriptor 추출]

Reference에서 특징점(Keypoint)을 찾는다.
각 특징점의 Descriptor를 생성한다.
Descriptor가 없다면 프로그램을 중단한다.
Reference의 특징점 개수를 출력한다.

          ↓

[7. 특징점 비교를 위한 Matcher 생성]

BFMatcher를 준비한다.
Descriptor 비교 방식은 L2 Distance로 설정한다.
Good Match 결과를 저장할 rows 리스트를 준비한다.

          ↓

[8. 네 개의 Target 이미지를 순서대로 읽기]

Target 이미지 4장을 하나씩 반복한다.
현재 Target 이미지를 불러온다.
이미지가 없으면 오류를 발생시킨다.
Target을 Grayscale로 변환한다.
처리 시작 시간을 기록한다.

          ↓

[9. Target의 Keypoint와 Descriptor 추출]

현재 Target에서 SIFT 특징점을 찾는다.
각 특징점의 Descriptor를 생성한다.
Good Match를 저장할 빈 리스트를 만든다.

          ↓

[10. Reference와 Target의 Descriptor 비교]

Target Descriptor가 있는지 확인한다.
Descriptor가 있다면 BFMatcher를 실행한다.

Reference의 각 Descriptor에 대해
가장 가까운 Target 후보 2개를 찾는다.

두 후보 사이의 L2 Distance를 얻는다.

          ↓

[11. Ratio Test로 Good Match 선별]

Match 후보를 하나씩 확인한다.
후보가 2개 미만이면 건너뛴다.

가장 가까운 후보의 Distance와
두 번째 후보의 Distance를 비교한다.

첫 번째 거리 < 두 번째 거리 × 0.75

조건을 만족하면 Good Match에 추가한다.
특징 추출과 Matching 처리시간을 계산한다.

          ↓

[12. Good Match 정렬 및 표시 개수 제한]

Good Match를 L2 Distance가 작은 순서로 정렬한다.
전체 Good Match 개수는 유지한다.
시각화에 사용할 상위 30개만 선택한다.

          ↓

[13. Reference와 Target의 특징점 연결 이미지 생성]

Reference와 Target 이미지를 나란히 배치한다.
대응되는 특징점을 선으로 연결한다.
SIFT Matching 결과 이미지를 생성한다.

          ↓

[14. SIFT 특징점 비교 결과 이미지 저장]

Target 이름을 이용하여 저장 파일명을 만든다.
특징점 연결 결과 이미지를 JPG로 저장한다.

          ↓

[15. Target별 SIFT 측정값 기록]

현재 Target의 검사 조건을 기록한다.
Reference 특징점 개수를 기록한다.
Target 특징점 개수를 기록한다.
Good Match 전체 개수를 기록한다.
특징 추출 및 Matching 처리시간을 기록한다.

측정 결과를 rows 리스트에 추가한다.
현재 Target의 측정 결과를 출력한다.

          ↓

아직 검사할 Target이 남아 있다면
8번으로 돌아가 다음 이미지를 검사한다.

4개 Target의 검사가 모두 끝나면
16번으로 이동한다.

          ↓

[16. 전체 검사 결과를 CSV 파일로 저장]

rows에 저장된 4개 Target 결과를 읽는다.
CSV 파일에 컬럼 제목을 작성한다.
Target별 검사 결과를 CSV로 저장한다.

CSV 저장 경로를 출력한다.
SIFT 결과 이미지 폴더를 출력한다.

          ↓

[최종 결과]

① Target별 SIFT Matching 이미지 4개
② SIFT 측정 결과 CSV 파일 1개

          ↓

정상 / 이동 / 회전 / 크기 변화 조건에서
특징점 개수와 Good Match 개수를 비교한다.
```

실행합니다.

```bash
python src/29_sift_matching.py
```

---

# 29. SIFT 결과를 확인하고 ORB와 비교합니다

이번 광천김 이미지에서 SIFT를 실행한 결과는 다음과 같습니다.

```
Reference Keypoints: 672
normal  reference_kp=672 target_kp=7926 good=615 time=126.42 ms
shift   reference_kp=672 target_kp=7663 good=596 time=72.88 ms
rot15   reference_kp=672 target_kp=7829 good=458 time=65.74 ms
scale90 reference_kp=672 target_kp=6962 good=413 time=64.88 ms

CSV 저장: reports/day04_sift_results.csv
결과 이미지: outputs/day04/sift
```

| 조건 | Reference Keypoint | Target Keypoint | Good Match | 처리시간 |
|---|---:|---:|---:|---:|
| `normal` | 672 | 7926 | 615 | 126.42 ms |
| `shift` | 672 | 7663 | 596 | 72.88 ms |
| `rot15` | 672 | 7829 | 458 | 65.74 ms |
| `scale90` | 672 | 6962 | 413 | 64.88 ms |
실형후 다음 값을 확인할 수 있습니다.

```
Reference Keypoint 수
Target Keypoint 수
Good Match 수
처리시간
```

결과 이미지는 다음 폴더에 저장됩니다.

```
outputs/day04/sift/

target_normal_sift.jpg
target_shift_sift.jpg
target_rot15_sift.jpg
target_scale90_sift.jpg
```

결과 이미지의 선은 Reference와 Target에서 서로 비슷하다고 판단한 특징점의 연결입니다.

![[{26CC3D0A-C879-41F8-8C8D-CDA63B9A5D6D}.png]]

## 29.1 `target_normal` — 가장 많은 Good Match가 만들어졌습니다

결과:

```
Good Match = 615
```

기준 상태에서는 네 조건 중 가장 많은 Good Match가 만들어졌습니다.

```
normal
→ 같은 방향
→ 같은 크기
→ Good Match 615
```

결과 이미지에서도 광천김 로고와 글자 주변의 많은 특징점이 연결되는 것을 확인할 수 있습니다.

---

## 29.2 `target_shift` — 위치가 달라져도 Match가 거의 유지되었습니다

결과:

```
normal = 615
shift  = 596
```

제품의 위치가 달라졌지만 Good Match 수는 크게 줄지 않았습니다.

```
위치 변화
        ↓
615 → 596
        ↓
많은 특징점 유지
```

즉 이번 데이터에서는 **SIFT가 위치 변화 후에도 같은 특징을 많이 찾아냈습니다.**

---

## 29.3 `target_rot15` — 회전 후에도 많은 특징이 남았습니다

결과:

```
normal = 615
rot15  = 458
```

15° 회전하면서 Good Match 수는 감소했지만 여전히 많은 특징점이 연결되었습니다.

```
15° 회전
        ↓
615 → 458
        ↓
Match는 감소
하지만 많은 특징은 유지
```

앞의 Template Matching에서는 회전 후 로고 위치를 제대로 찾지 못했지만, SIFT에서는 회전된 이미지에서도 많은 특징점 연결이 남았습니다.

---

## 29.4 `target_scale90` — 크기가 달라져도 특징점이 남았습니다

결과:

```
normal  = 615
scale90 = 413
```

이미지 크기를 90%로 줄이자 Good Match가 감소했습니다.

하지만 `413개`의 Match가 남아 있어 크기가 달라진 상태에서도 여러 특징을 계속 비교할 수 있었습니다.

```
크기 변화
        ↓
615 → 413
        ↓
Match 감소
하지만 특징점 비교는 계속 가능
```

---

## 29.5 ORB와 SIFT 결과를 함께 봅니다

앞에서 측정한 ORB 결과와 함께 정리하면 다음과 같습니다.

|조건|ORB Good Match|SIFT Good Match|
|---|---|---|
|`normal`|276|615|
|`shift`|245|596|
|`rot15`|232|458|
|`scale90`|156|413|

하지만 이 숫자만 보고:

```
SIFT 615
ORB 276

→ SIFT가 두 배 더 정확하다
```

라고 판단하면 안 됩니다.

ORB와 SIFT는 특징점을 만드는 방법 자체가 다르고, 이번 실행에서도 Target에서 찾은 Keypoint 수가 크게 다릅니다.

```
ORB Target Keypoint
→ 최대 약 1000개

SIFT Target Keypoint
→ 약 7000~8000개
```

따라서 **두 방법의 Good Match 숫자를 그대로 정확도처럼 비교하지 않습니다.**

이번 실습에서는 다음을 확인한 것으로 충분합니다.

```
ORB
→ 위치·회전·크기 변화에서도
   특징점 Matching이 유지됨
→ 처리시간이 짧은 편

SIFT
→ 위치·회전·크기 변화에서도
   많은 특징점 Matching이 유지됨
→ ORB보다 처리시간이 긴 편
```

---

## 29.6 Good Match가 많아도 위치가 모두 정확한 것은 아닙니다

결과 이미지를 자세히 보면 중요한 사실이 하나 있습니다.

Reference는 하단의:

```
KC + (주)광천김 + 영문
```

영역입니다.

그런데 Target에는 비슷한 `광천김` 글자와 무늬가 다른 위치에도 존재합니다.

따라서 일부 Match 선은 실제 하단 로고뿐 아니라 다른 비슷한 글자 영역으로도 연결될 수 있습니다.

```
Good Match가 많음
        ↓
그런데

모든 Match가
실제 같은 위치를 연결한 것인가?
        ↓
아직 확인 필요
```

즉:

> **Good Match가 많다는 것과 실제 라벨 위치를 정확하게 찾았다는 것은 같은 의미가 아닙니다.**

그래서 다음 단계에서는 이 특징점들이 실제로 **같은 위치 관계를 가지고 있는지** 한 번 더 확인합니다.

```
SIFT Good Match
        ↓
올바른 위치 관계인지 확인
        ↓
RANSAC
        ↓
Homography
        ↓
실제 라벨 위치를 사각형으로 표시
```

다음 실습에서는 `Homography + RANSAC`을 이용하여 이 부분을 확인합니다.

---
# PART 30~33 — 검사 6: RANSAC + Homography로 라벨 위치 추정

---

# 30. 이번에는 실제 광천김 로고 위치까지 찾아봅니다

앞의 SIFT 실습에서는 Reference와 Target 사이에서 많은 Good Match를 찾았습니다.

예를 들어 이번 결과에서는:

```text
normal → 615 
shift → 596 
rot15 → 458 
scale90 → 413
```

개의 Good Match가 만들어졌습니다.

하지만 결과 이미지를 보면 모든 선이 실제 하단의 `KC + (주)광천김` 로고에만 연결되는 것은 아닙니다.

Target 이미지에는 비슷한 글자와 모양이 다른 위치에도 있기 때문입니다.

```
Good Match가 많음
        ↓
그런데
실제 같은 라벨끼리
연결된 것이 맞을까?
        ↓
한 번 더 확인 필요
```

이번에는 Good Match의 **위치 관계**를 이용하여 실제 광천김 로고 영역을 찾아봅니다.

이때 사용하는 방법이 `Homography`입니다.

## 30.1 Homography는 무엇을 하나요?

쉽게 말하면 Homography는:

> **Reference의 특징점들이 Target에서 어디로 이동했는지를 이용하여 Reference 전체의 위치를 계산하는 방법**입니다.

![[Pasted image 20261028075145.png]]

Homography가 특징점을 기억했다가 추적하는 것은 아니고, SIFT가 찾아낸 특징점들의 위치 관계를 이용해 로고가 어떻게 이동·회전·변형되었는지 수학적으로 계산하는 것입니다.

### SIFT와 Homography의 역할

| 기술         | 하는 일                                 |
| ---------- | ------------------------------------ |
| SIFT       | 기준 이미지와 검사 이미지에서 특징점을 찾아 서로 비교합니다.   |
| Good Match | 두 이미지에서 같은 특징일 가능성이 높은 점들을 선택합니다.    |
| RANSAC     | 잘못 연결된 특징점을 걸러내고, 일관된 위치 관계를 찾습니다.   |
| Homography | 올바르게 연결된 특징점으로 로고 전체가 어디에 있는지 계산합니다. |

이번 광천김 실습으로 보면:

```
reference_label.jpg
KC + (주)광천김
        ↓
SIFT로 Good Match 찾기
        ↓
Target에서 대응되는 특징점 확인
        ↓
특징점들의 위치 관계 계산
        ↓
광천김 로고가 있는 영역 계산
        ↓
사각형으로 표시
```

즉 앞의 SIFT에서는:

```
"서로 비슷한 특징점이 있는가?"
```

를 확인했다면,

Homography에서는:

```
"그 특징점들을 이용해서
라벨 전체가 어디에 있는가?"
```

를 확인합니다.

따라서 결과 화면에서는 여러 Match 선 대신 **광천김 로고 주변에 사각형이 제대로 그려지는지**가 중요한 확인 대상이 됩니다.

## 30.2 왜 최소 4개의 Match가 필요한가요?

라벨 전체의 위치와 기울기를 계산하려면 한두 개의 점만으로는 부족합니다.

그래서 Homography에서는 최소 4개 이상의 대응점을 사용합니다.

![[Pasted image 20260830212332.png]]

오늘은 Homography의 수식을 계산하지 않습니다.

다음 흐름만 이해하면 충분합니다.

```
SIFT
        ↓
Good Match 찾기
        ↓
올바른 Match를 다시 골라냄
        ↓
Homography 계산
        ↓
Reference의 네 모서리를
Target 위치로 옮김
        ↓
라벨 위치를 사각형으로 표시
```

![[Pasted image 20260830212627.png]]

---

# 31. RANSAC은 왜 같이 사용하나요?

앞에서 만든 Good Match가 모두 정확한 것은 아닙니다.

예를 들어:

![[Pasted image 20260830212933.png]]

```
Good Match 25개

        ↓

실제 같은 로고를 연결
20개

+

엉뚱한 곳을 연결
5개
```

처럼 잘못 연결된 Match가 섞일 수 있습니다.

이런 잘못된 Match까지 모두 사용하면 Homography가 엉뚱한 사각형을 만들 수 있습니다.

그래서 `RANSAC`을 함께 사용합니다.

쉽게 말하면:

> RANSAC은 여러 Good Match 중에서 같은 위치 관계를 잘 따르는 점들을 다시 골라내는 과정입니다.

```
Good Match
        ↓
올바른 연결 + 잘못된 연결
        ↓
RANSAC
        ↓
같은 움직임을 따르는
믿을 만한 점들 선택
        ↓
Inlier
        ↓
Homography 계산
```

---

## 31.1 이번 실습에서 확인할 값

이번에는 다음 세 값을 확인합니다.

```
Good Match Count
→ SIFT에서 찾은
   비교적 좋은 특징점 연결 수


Inlier Count
→ Good Match 중에서
   RANSAC이 위치 관계까지 맞다고
   판단한 연결 수


Inlier Ratio
→ Good Match 가운데
   Inlier가 차지하는 비율
```

예를 들어:

```
Good Match = 100
Inlier     = 80
```

이라면:

```
100개의 Good Match 중
80개가 같은 위치 관계를
잘 따랐다는 뜻
```

으로 이해하면 됩니다.

---

## 31.2 이번 단계의 핵심

앞의 실습과 연결하면 다음과 같습니다.

```
ORB / SIFT
→ 같은 특징점인가?

        ↓

Good Match
→ 비슷한 특징 연결

        ↓

RANSAC
→ 그중 위치 관계까지
   자연스러운 점을 선택

        ↓

Homography
→ 그 점들을 이용하여
   라벨 전체 위치 계산

        ↓

광천김 로고 위치를
사각형으로 표시
```

![[Pasted image 20260830213447.png]]

즉 이번 실습의 질문은 단순합니다.

> SIFT에서 찾은 특징점들을 이용하면, 회전하거나 크기가 달라진 광천김 로고의 실제 위치를 사각형으로 찾을 수 있을까?

다음 `30_homography.py`에서 직접 확인합니다.

---

# 32. `30_homography.py` — SIFT Match를 이용해 광천김 로고 위치를 찾습니다

이번 실습은 앞에서 SIFT로 찾은 Good Match를 이용하여:

```
SIFT 
     ↓
Good Match 
     ↓
RANSAC 
     ↓
Inlier 
     ↓
Homography 
     ↓
광천김 로고 위치 계산 
     ↓ 
사각형으로 표시
```

하는 과정입니다.

같은 광천김 데이터를 그대로 사용합니다.

```
Reference
reference_label.jpg

Targets
target_normal.jpg
target_shift.jpg
target_rot15.jpg
target_scale90.jpg
```

이번에는 다음 값을 확인합니다.

```
Good Match
→ SIFT에서 찾은 특징점 연결 수

Inlier
→ Good Match 중에서
   위치 관계까지 자연스러운 연결 수

Inlier Ratio
→ Good Match 중 Inlier의 비율

FOUND / NOT_FOUND
→ Homography를 이용해
   라벨 위치를 계산했는가?
```

파일을 만듭니다.

```text
src/30_homography.py
```

```python
from pathlib import Path
from time import perf_counter
import csv

import cv2
import numpy as np


# --------------------------------------------------
# 1. 입력 이미지와 결과 저장 경로 설정
#
# 무엇을 하나요?
# - 비교 기준인 Reference 이미지 경로를 지정합니다.
# - 검사할 Target 이미지 폴더를 지정합니다.
# - Homography 결과 이미지와 CSV 저장 경로를 설정합니다.
# - 저장할 폴더가 없으면 자동으로 생성합니다.
#
# Reference : 기준이 되는 광천김 로고 이미지
# Target    : 위치·회전·크기를 변경한 비교 이미지
# --------------------------------------------------

reference_path = Path(
    "samples/day04/reference/reference_label.jpg"
)

target_dir = Path(
    "samples/day04/targets"
)

output_dir = Path(
    "outputs/day04/homography"
)

report_path = Path(
    "reports/day04_homography_results.csv"
)


output_dir.mkdir(
    parents=True,
    exist_ok=True,
)

report_path.parent.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 2. 비교할 Target 이미지 4장 준비
#
# 무엇을 하나요?
# - 정상, 이동, 회전, 크기 변화 이미지를 준비합니다.
# - 각 이미지에 동일한 SIFT + Homography를 적용합니다.
#
# 비교 조건
# normal  : 정상 이미지
# shift   : 위치가 이동한 이미지
# rot15   : 15도 회전한 이미지
# scale90 : 크기가 90%로 변한 이미지
#
# 목적:
# 이미지의 위치·회전·크기가 변하더라도
# Reference 로고의 위치를 찾을 수 있는지 확인합니다.
# --------------------------------------------------

target_settings = [
    (
        "normal",
        target_dir / "target_normal.jpg",
    ),
    (
        "shift",
        target_dir / "target_shift.jpg",
    ),
    (
        "rot15",
        target_dir / "target_rot15.jpg",
    ),
    (
        "scale90",
        target_dir / "target_scale90.jpg",
    ),
]


# --------------------------------------------------
# 3. 현재 OpenCV에서 SIFT 사용 가능 여부 확인
#
# 무엇을 하나요?
# - 설치된 OpenCV에 SIFT_create 기능이 있는지 확인합니다.
#
# 왜 필요한가요?
# SIFT는 이미지의 특징점을 추출하는 알고리즘입니다.
# 현재 OpenCV 환경에서 SIFT를 사용할 수 없다면
# 이후 특징점 검출과 Homography 계산을 진행할 수 없습니다.
#
# 따라서 SIFT_create가 없으면 오류를 발생시킵니다.
# --------------------------------------------------

if not hasattr(
    cv2,
    "SIFT_create",
):
    raise RuntimeError(
        "현재 OpenCV에서 SIFT_create를 사용할 수 없습니다."
    )


# --------------------------------------------------
# 4. 기준 이미지(Reference)를 읽고 흑백으로 변환
#
# 무엇을 하나요?
# - 광천김 Reference 이미지를 컬러로 읽습니다.
# - SIFT 특징점 검출을 위해 Grayscale로 변환합니다.
# - 이미지를 읽지 못하면 오류를 발생시킵니다.
#
# 컬러 원본:
# 이후 Reference 영역의 네 모서리를 정의하는 기준입니다.
#
# Grayscale:
# SIFT 특징점을 추출할 때 사용합니다.
# --------------------------------------------------

reference = cv2.imread(
    str(reference_path)
)

if reference is None:
    raise FileNotFoundError(
        reference_path
    )


reference_gray = cv2.cvtColor(
    reference,
    cv2.COLOR_BGR2GRAY,
)


# --------------------------------------------------
# 5. SIFT 특징점 검출기 생성
#
# 무엇을 하나요?
# - SIFT 알고리즘을 사용할 준비를 합니다.
#
# SIFT란?
# Scale-Invariant Feature Transform
#
# 이미지에서 특징적인 위치를 찾아내고,
# 크기나 회전 변화에도 비교할 수 있는
# 특징 정보를 생성하는 알고리즘입니다.
#
# 이번 코드에서는 SIFT 기본 설정을 사용합니다.
# --------------------------------------------------

sift = cv2.SIFT_create()


# --------------------------------------------------
# 6. Reference의 Keypoint와 Descriptor 추출
#
# 무엇을 하나요?
# - Reference 이미지에서 특징점(Keypoint)을 찾습니다.
# - 각 특징점의 Descriptor를 생성합니다.
#
# Keypoint:
# 이미지에서 특징적인 위치와 크기, 방향 등의 정보
#
# Descriptor:
# 해당 특징점 주변의 영상 특징을 표현한 숫자 데이터
#
# SIFT Descriptor는 일반적으로 128차원으로 구성됩니다.
#
# Descriptor가 없다면 Target과 비교할 수 없으므로
# 프로그램을 중단합니다.
# --------------------------------------------------

reference_keypoints, reference_descriptors = (
    sift.detectAndCompute(
        reference_gray,
        None,
    )
)


if reference_descriptors is None:
    raise RuntimeError(
        "Reference에서 SIFT Descriptor를 만들 수 없습니다."
    )


print(
    "Reference Keypoints:",
    len(reference_keypoints),
)


# --------------------------------------------------
# 7. SIFT Descriptor 비교용 Matcher 생성
#
# 무엇을 하나요?
# - Reference와 Target의 특징점을 비교할
#   BFMatcher를 준비합니다.
# - SIFT Descriptor는 L2 Distance로 비교합니다.
#
# L2 Distance:
# 두 Descriptor 사이의 유클리드 거리입니다.
#
# 거리가 작을수록 두 특징이 비슷하다는 의미입니다.
#
# crossCheck=False:
# 이후 knnMatch()와 Ratio Test를 사용하기 위한 설정입니다.
# --------------------------------------------------

matcher = cv2.BFMatcher(
    cv2.NORM_L2,
    crossCheck=False,
)


# --------------------------------------------------
# 8. Reference 이미지의 네 모서리 좌표 준비
#
# 무엇을 하나요?
# - Reference 이미지의 가로·세로 크기를 구합니다.
# - 이미지의 네 모서리 좌표를 만듭니다.
#
# 네 모서리:
# ① 왼쪽 위
# ② 오른쪽 위
# ③ 오른쪽 아래
# ④ 왼쪽 아래
#
# 왜 필요한가요?
# 이후 Homography를 계산하면
# 이 네 모서리가 Target 이미지에서
# 어디에 위치하는지 변환할 수 있기 때문입니다.
#
# 이를 이용하여 Target에 로고 위치를
# 사각형으로 표시합니다.
# --------------------------------------------------

reference_h, reference_w = (
    reference_gray.shape[:2]
)


reference_corners = np.float32([
    [0, 0],
    [reference_w - 1, 0],
    [reference_w - 1, reference_h - 1],
    [0, reference_h - 1],
]).reshape(
    -1,
    1,
    2,
)


# 각 Target의 측정 결과를 저장할 리스트
rows = []


# --------------------------------------------------
# 9. 네 개의 Target 이미지를 순서대로 읽기
#
# 무엇을 하나요?
# - 준비한 Target 이미지 4장을 하나씩 검사합니다.
# - Target을 컬러로 읽고 Grayscale로 변환합니다.
# - 처리시간 측정을 시작합니다.
#
# for 반복문 내부에서는
# 10번부터 20번까지의 과정을 Target마다 수행합니다.
# --------------------------------------------------

for condition, target_path in target_settings:

    target = cv2.imread(
        str(target_path)
    )

    if target is None:
        raise FileNotFoundError(
            target_path
        )


    target_gray = cv2.cvtColor(
        target,
        cv2.COLOR_BGR2GRAY,
    )


    start = perf_counter()


    # --------------------------------------------------
    # 10. Target의 SIFT Keypoint와 Descriptor 추출
    #
    # 무엇을 하나요?
    # - Target에서도 SIFT 특징점을 찾습니다.
    # - 각 특징점의 Descriptor를 만듭니다.
    #
    # Reference와 Target에 동일한 SIFT를 적용하여
    # 서로 대응되는 특징점을 찾을 준비를 합니다.
    #
    # 다음 변수도 초기화합니다.
    #
    # good_matches : Ratio Test를 통과한 특징점 대응
    # inlier_count : RANSAC이 선택한 Inlier 개수
    # inlier_ratio : Good Match 중 Inlier 비율
    # status       : Homography 계산 성공 여부
    # preview      : 결과를 표시할 이미지
    # --------------------------------------------------

    target_keypoints, target_descriptors = (
        sift.detectAndCompute(
            target_gray,
            None,
        )
    )


    good_matches = []

    inlier_count = 0
    inlier_ratio = 0.0

    status = "NOT_FOUND"

    preview = target.copy()


    # --------------------------------------------------
    # 11. Reference와 Target의 Descriptor 비교
    #
    # 무엇을 하나요?
    # - BFMatcher의 knnMatch()를 사용합니다.
    # - Reference의 각 Descriptor에 대해
    #   가까운 Target Descriptor 후보 2개를 찾습니다.
    #
    # k=2:
    # 가장 가까운 후보와 두 번째로 가까운 후보를 구합니다.
    #
    # 두 후보의 거리를 비교하여
    # 다음 단계에서 Good Match를 선별합니다.
    #
    # Target Descriptor가 없다면 Matching을 생략합니다.
    # --------------------------------------------------

    if target_descriptors is not None:

        match_candidates = matcher.knnMatch(
            reference_descriptors,
            target_descriptors,
            k=2,
        )


        # --------------------------------------------------
        # 12. Ratio Test로 Good Match 선별
        #
        # 무엇을 하나요?
        # - 첫 번째 후보와 두 번째 후보의 거리를 비교합니다.
        # - 첫 번째 후보가 충분히 더 가까운 경우만 남깁니다.
        #
        # 조건:
        # 첫 번째 거리 < 두 번째 거리 × 0.75
        #
        # 조건을 만족하면 Good Match에 추가합니다.
        #
        # 왜 필요한가요?
        # 비슷한 특징점이 여러 곳에 나타날 수 있기 때문입니다.
        #
        # Ratio Test는 모호한 대응을 줄여
        # 비교적 신뢰할 만한 Match 후보를 선택합니다.
        # --------------------------------------------------

        for candidate in match_candidates:

            if len(candidate) < 2:
                continue


            m, n = candidate


            if m.distance < 0.75 * n.distance:

                good_matches.append(
                    m
                )


    # --------------------------------------------------
    # 13. Homography 계산에 필요한 대응점 준비
    #
    # 무엇을 하나요?
    # - Good Match가 최소 4개인지 확인합니다.
    # - Reference 특징점의 좌표를 추출합니다.
    # - 대응되는 Target 특징점의 좌표를 추출합니다.
    #
    # 왜 최소 4개인가요?
    # 일반적인 2D Homography를 계산하려면
    # 적절하게 배치된 대응점이 최소 4쌍 필요하기 때문입니다.
    #
    # 단, 대응점이 4개 이상이어도
    # 점들이 잘못 배치되거나 잘못 연결되었다면
    # Homography 계산에 실패할 수 있습니다.
    #
    # queryIdx:
    # Reference 특징점의 인덱스
    #
    # trainIdx:
    # Target 특징점의 인덱스
    # --------------------------------------------------

    if len(good_matches) >= 4:

        # Reference에서 Good Match의 좌표
        reference_points = np.float32([
            reference_keypoints[
                match.queryIdx
            ].pt
            for match in good_matches
        ]).reshape(
            -1,
            1,
            2,
        )


        # Target에서 Good Match의 좌표
        target_points = np.float32([
            target_keypoints[
                match.trainIdx
            ].pt
            for match in good_matches
        ]).reshape(
            -1,
            1,
            2,
        )


        # --------------------------------------------------
        # 14. RANSAC을 사용하여 Homography 계산
        #
        # 무엇을 하나요?
        # - Reference와 Target의 대응점으로
        #   Homography 변환 행렬을 계산합니다.
        # - RANSAC을 이용해 잘못된 대응점의 영향을 줄입니다.
        #
        # Homography:
        # 한 평면의 점들이 다른 이미지에서
        # 어디로 이동하는지 나타내는 변환 관계입니다.
        #
        # RANSAC:
        # 대응점 중에서 변환 관계를 잘 따르는 점을 찾아
        # 잘못된 대응점의 영향을 줄이는 알고리즘입니다.
        #
        # Inlier:
        # 계산한 변환 관계에 잘 맞는 대응점
        #
        # Outlier:
        # 변환 관계에 잘 맞지 않는 대응점
        #
        # 5.0:
        # RANSAC에서 사용하는 재투영 오차 허용 기준입니다.
        # 단위는 픽셀입니다.
        # --------------------------------------------------

        homography, inlier_mask = (
            cv2.findHomography(
                reference_points,
                target_points,
                cv2.RANSAC,
                5.0,
            )
        )


        if (
            homography is not None
            and inlier_mask is not None
        ):

            # --------------------------------------------------
            # 15. Inlier 개수와 Inlier Ratio 계산
            #
            # 무엇을 하나요?
            # - RANSAC이 선택한 Inlier 개수를 계산합니다.
            # - 전체 Good Match 중 Inlier 비율을 구합니다.
            #
            # Inlier Count:
            # 변환 관계에 잘 맞는 대응점 개수
            #
            # Inlier Ratio:
            # Inlier 개수 ÷ Good Match 전체 개수
            #
            # 예:
            # Good Match 40개
            # Inlier 32개
            #
            # Inlier Ratio = 32 / 40 = 0.8
            #
            # 즉 Good Match 중 80%가
            # 계산된 Homography 관계를 따른다는 뜻입니다.
            #
            # 주의:
            # Inlier Ratio가 높다고 해서
            # 반드시 올바른 로고를 찾았다는 의미는 아닙니다.
            # --------------------------------------------------

            inlier_count = int(
                inlier_mask.ravel().sum()
            )


            inlier_ratio = (
                inlier_count
                / len(good_matches)
            )


            # --------------------------------------------------
            # 16. Reference의 네 모서리를 Target 좌표로 변환
            #
            # 무엇을 하나요?
            # - 8번에서 만든 Reference의 네 모서리를 가져옵니다.
            # - 계산된 Homography 행렬을 적용합니다.
            # - 네 모서리가 Target에서 어디에 해당하는지 구합니다.
            #
            # perspectiveTransform():
            # Homography를 사용하여 점의 좌표를 변환합니다.
            #
            # 이 결과를 이용하면
            # Target 이미지에서 Reference가 대응되는 위치를
            # 사각형으로 그릴 수 있습니다.
            # --------------------------------------------------

            projected_corners = (
                cv2.perspectiveTransform(
                    reference_corners,
                    homography,
                )
            )


            # --------------------------------------------------
            # 17. 계산된 위치를 초록색 사각형으로 표시
            #
            # 무엇을 하나요?
            # - 변환한 네 모서리 좌표를 정수로 변환합니다.
            # - Target 이미지에 초록색 사각형을 그립니다.
            # - Homography 계산이 성공한 상태로 기록합니다.
            #
            # cv2.polylines():
            # 여러 점을 선으로 연결하는 함수입니다.
            #
            # (0, 255, 0):
            # OpenCV BGR 색상에서 초록색을 의미합니다.
            #
            # status = FOUND:
            # Homography 행렬이 계산되었다는 의미입니다.
            #
            # 주의:
            # FOUND는 최종 OK 판정이 아닙니다.
            # 사각형 위치가 실제 로고와 일치하는지는
            # 별도로 확인해야 합니다.
            # --------------------------------------------------

            polygon = np.int32(
                projected_corners
            )


            cv2.polylines(
                preview,
                [polygon],
                True,
                (0, 255, 0),
                3,
                cv2.LINE_AA,
            )


            status = "FOUND"


    # SIFT 추출·Matching·Homography 계산시간을 측정합니다.
    # 이미지 읽기, 결과 문자 표시, 이미지 저장시간은 제외합니다.

    elapsed_ms = (
        perf_counter() - start
    ) * 1000


    # --------------------------------------------------
    # 18. Homography 측정 결과를 이미지에 표시
    #
    # 무엇을 하나요?
    # - Target 이미지 위에 측정 결과를 문자로 표시합니다.
    #
    # 표시 항목:
    # good  : Good Match 전체 개수
    # inlier: RANSAC Inlier 개수
    # ratio : Inlier Ratio
    #
    # 예:
    # good=40 inlier=32 ratio=0.80
    #
    # 이 숫자를 보면서
    # 특징점 대응이 얼마나 기하학적으로 일치하는지
    # 비교할 수 있습니다.
    # --------------------------------------------------

    cv2.putText(
        preview,
        (
            f"good={len(good_matches)} "
            f"inlier={inlier_count} "
            f"ratio={inlier_ratio:.2f}"
        ),
        (20, 35),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.8,
        (0, 255, 0),
        2,
        cv2.LINE_AA,
    )


    # --------------------------------------------------
    # 19. Homography 결과 이미지 저장
    #
    # 무엇을 하나요?
    # - 사각형과 측정값이 표시된 이미지를 JPG로 저장합니다.
    # - Target마다 별도의 결과 파일을 생성합니다.
    #
    # 저장 파일 예시:
    # target_normal_homography.jpg
    # target_shift_homography.jpg
    # target_rot15_homography.jpg
    # target_scale90_homography.jpg
    #
    # 학생들은 각 결과 이미지를 비교하면서
    # 사각형 위치가 실제 광천김 로고와 일치하는지 확인합니다.
    # --------------------------------------------------

    output_path = (
        output_dir
        / f"{target_path.stem}_homography.jpg"
    )


    cv2.imwrite(
        str(output_path),
        preview,
    )


    # --------------------------------------------------
    # 20. Target별 Homography 측정값 기록
    #
    # 무엇을 하나요?
    # - Target마다 계산한 측정값을 rows 리스트에 저장합니다.
    #
    # 저장 항목:
    #
    # condition   : 정상·이동·회전·크기 변화 조건
    # filename    : Target 이미지 이름
    # good_matches: Ratio Test를 통과한 Match 개수
    # inliers     : RANSAC Inlier 개수
    # inlier_ratio: Inlier / Good Match
    # status      : FOUND 또는 NOT_FOUND
    # time_ms     : 처리시간(밀리초)
    #
    # 이렇게 기록한 결과는
    # 21번에서 하나의 CSV 파일로 저장합니다.
    # --------------------------------------------------

    rows.append({
        "condition": condition,
        "filename": target_path.name,
        "good_matches": len(
            good_matches
        ),
        "inliers": inlier_count,
        "inlier_ratio": f"{inlier_ratio:.6f}",
        "status": status,
        "time_ms": f"{elapsed_ms:.3f}",
    })


    print(
        f"{condition:7s}",
        f"good={len(good_matches)}",
        f"inlier={inlier_count}",
        f"ratio={inlier_ratio:.3f}",
        f"status={status}",
        f"time={elapsed_ms:.2f} ms",
    )


# --------------------------------------------------
# 21. 전체 Homography 측정 결과를 CSV로 저장
#
# 무엇을 하나요?
# - 4개 Target의 측정값을 하나의 CSV로 저장합니다.
# - Good Match, Inlier, Inlier Ratio를 비교합니다.
# - 정상·이동·회전·크기 변화에 따른
#   Homography 계산 결과를 분석합니다.
#
# 최종 산출물:
#
# reports/day04_homography_results.csv
#
# outputs/day04/homography/
#   ├── target_normal_homography.jpg
#   ├── target_shift_homography.jpg
#   ├── target_rot15_homography.jpg
#   └── target_scale90_homography.jpg
# --------------------------------------------------

with report_path.open(
    "w",
    newline="",
    encoding="utf-8",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=[
            "condition",
            "filename",
            "good_matches",
            "inliers",
            "inlier_ratio",
            "status",
            "time_ms",
        ],
    )

    writer.writeheader()
    writer.writerows(rows)


print()

print(
    "CSV 저장:",
    report_path,
)

print(
    "결과 이미지:",
    output_dir,
)
```
### 의사코드

```text
[1. 입력 이미지와 결과 저장 경로 설정]

광천김 Reference 이미지 경로를 정한다.
Target 이미지 폴더를 정한다.
Homography 결과 이미지 저장 경로를 정한다.
CSV 저장 경로를 정한다.
저장 폴더가 없으면 새로 만든다.

          ↓

[2. 비교할 Target 이미지 4장 준비]

정상 이미지(normal)를 준비한다.
위치 이동 이미지(shift)를 준비한다.
15도 회전 이미지(rot15)를 준비한다.
90% 크기 이미지(scale90)를 준비한다.

          ↓

[3. 현재 OpenCV에서 SIFT 사용 가능 여부 확인]

OpenCV에 SIFT_create가 있는지 확인한다.
SIFT를 사용할 수 없다면 프로그램을 중단한다.

          ↓

[4. 기준 이미지(Reference)를 읽고 흑백으로 변환]

광천김 Reference 이미지를 불러온다.
이미지가 없으면 오류를 발생시킨다.
Reference를 Grayscale로 변환한다.

          ↓

[5. SIFT 특징점 검출기 생성]

SIFT 알고리즘을 준비한다.
이미지에서 특징점을 찾을 준비를 한다.

          ↓

[6. Reference의 Keypoint와 Descriptor 추출]

Reference에서 특징점(Keypoint)을 찾는다.
각 특징점의 Descriptor를 생성한다.
Descriptor가 없다면 프로그램을 중단한다.
Reference 특징점 개수를 출력한다.

          ↓

[7. SIFT Descriptor 비교용 Matcher 생성]

BFMatcher를 준비한다.
Descriptor 비교 방식을 L2 Distance로 설정한다.
knnMatch와 Ratio Test를 사용할 준비를 한다.

          ↓

[8. Reference 이미지의 네 모서리 좌표 준비]

Reference의 가로·세로 크기를 구한다.

왼쪽 위 좌표를 만든다.
오른쪽 위 좌표를 만든다.
오른쪽 아래 좌표를 만든다.
왼쪽 아래 좌표를 만든다.

네 모서리 좌표를 배열로 저장한다.
Target별 측정값을 저장할 rows 리스트를 준비한다.

          ↓

[9. 네 개의 Target 이미지를 순서대로 읽기]

Target 이미지 4장을 하나씩 반복한다.
현재 Target 이미지를 불러온다.
이미지가 없으면 오류를 발생시킨다.
Target을 Grayscale로 변환한다.
처리 시작 시간을 기록한다.

          ↓

[10. Target의 SIFT Keypoint와 Descriptor 추출]

Target에서 SIFT 특징점을 찾는다.
각 특징점의 Descriptor를 생성한다.

Good Match 리스트를 초기화한다.
Inlier Count를 0으로 초기화한다.
Inlier Ratio를 0.0으로 초기화한다.
Status를 NOT_FOUND로 설정한다.
결과 표시용 Target 이미지를 복사한다.

          ↓

[11. Reference와 Target의 Descriptor 비교]

Target Descriptor가 있는지 확인한다.

Descriptor가 있다면 BFMatcher를 실행한다.

Reference의 각 Descriptor에 대해
가장 가까운 Target 후보 2개를 찾는다.

두 후보의 L2 Distance를 얻는다.

          ↓

[12. Ratio Test로 Good Match 선별]

Match 후보를 하나씩 확인한다.
후보가 2개 미만이면 건너뛴다.

가장 가까운 후보와
두 번째 후보의 거리를 비교한다.

첫 번째 거리 < 두 번째 거리 × 0.75

조건을 만족하면 Good Match에 추가한다.

          ↓

[13. Homography 계산에 필요한 대응점 준비]

Good Match 개수가 4개 이상인지 확인한다.

4개 미만이면 Homography 계산을 생략한다.

4개 이상이면:

Reference의 Good Match 좌표를 추출한다.
Target의 대응되는 Good Match 좌표를 추출한다.

두 이미지의 대응점 배열을 준비한다.

          ↓

[14. RANSAC을 사용하여 Homography 계산]

Reference 대응점과 Target 대응점을 입력한다.
RANSAC으로 Homography 행렬을 계산한다.

재투영 오차 허용 기준은 5.0픽셀로 설정한다.

변환 관계에 잘 맞는 점을 Inlier로 구분한다.
잘 맞지 않는 점은 Outlier로 구분한다.

Homography와 Inlier Mask를 반환받는다.

          ↓

[15. Inlier 개수와 Inlier Ratio 계산]

Homography와 Inlier Mask가 있는지 확인한다.

계산에 실패했다면
Status는 NOT_FOUND 상태로 유지한다.

계산에 성공했다면:

Inlier Mask에서 Inlier 개수를 센다.

Inlier Ratio =
Inlier Count / Good Match Count

를 계산한다.

          ↓

[16. Reference의 네 모서리를 Target 좌표로 변환]

8번에서 저장한 Reference의 네 모서리를 가져온다.
계산된 Homography 행렬을 적용한다.

Target에서 대응되는 네 모서리 위치를 계산한다.

          ↓

[17. 계산된 위치를 초록색 사각형으로 표시]

변환된 네 모서리 좌표를 정수로 변환한다.

Target 이미지에서
네 모서리를 초록색 선으로 연결한다.

Homography 계산에 성공했으므로
Status를 FOUND로 변경한다.

          ↓

Homography 계산을 완료하거나 생략했다면
처리시간을 계산한다.

          ↓

[18. Homography 측정 결과를 이미지에 표시]

현재 Target에 다음 값을 표시한다.

Good Match 개수
Inlier 개수
Inlier Ratio

결과 이미지를 확인할 수 있도록 준비한다.

          ↓

[19. Homography 결과 이미지 저장]

Target 이름을 이용해 저장 파일명을 만든다.

사각형과 측정값이 표시된 이미지를
JPG 파일로 저장한다.

          ↓

[20. Target별 Homography 측정값 기록]

검사 조건을 기록한다.
Target 이미지 이름을 기록한다.
Good Match 개수를 기록한다.
Inlier 개수를 기록한다.
Inlier Ratio를 기록한다.
Homography 계산 성공 여부를 기록한다.
처리시간을 기록한다.

측정값을 rows 리스트에 추가한다.
현재 Target의 검사 결과를 출력한다.

          ↓

아직 검사할 Target이 남아 있다면
9번으로 돌아가 다음 이미지를 검사한다.

4개 Target 검사가 모두 끝나면
21번으로 이동한다.

          ↓

[21. 전체 Homography 측정 결과를 CSV로 저장]

rows에 저장된 4개 Target 결과를 읽는다.

CSV 컬럼 제목을 작성한다.
Target별 측정값을 CSV로 저장한다.

CSV 저장 경로를 출력한다.
Homography 결과 이미지 폴더를 출력한다.

          ↓

[최종 결과]

① Target별 Homography 이미지 4개

   target_normal_homography.jpg
   target_shift_homography.jpg
   target_rot15_homography.jpg
   target_scale90_homography.jpg

② Homography 측정 결과 CSV 파일 1개

   day04_homography_results.csv

          ↓

정상 / 이동 / 회전 / 크기 변화 조건에서
Good Match와 Inlier Ratio를 비교한다.

초록색 사각형이 실제 광천김 로고의
위치와 일치하는지 확인한다.
```

실행합니다.

```bash
python src/30_homography.py
```

프로그램을 실행하면 다음 값을 확인할 수 있습니다.

```
Good Match
Inlier
Inlier Ratio
FOUND / NOT_FOUND
처리시간
```

결과 이미지는 다음 폴더에 저장됩니다.

```
outputs/day04/homography/

target_normal_homography.jpg
target_shift_homography.jpg
target_rot15_homography.jpg
target_scale90_homography.jpg
```

초록색 사각형이 표시되었다면 Homography가 계산한 **광천김 Reference 라벨의 위치**입니다.

실제 결과에 대한 해석은 프로그램 실행 후 확인합니다.

---

# 33. Homography 결과를 확인합니다

실행 결과는 다음과 같습니다.

```
Reference Keypoints: 675
normal  good=617 inlier=610 ratio=0.989 status=FOUND time=108.77 ms
shift   good=597 inlier=594 ratio=0.995 status=FOUND time=81.29 ms
rot15   good=459 inlier=447 ratio=0.974 status=FOUND time=80.98 ms
scale90 good=412 inlier=402 ratio=0.976 status=FOUND time=93.54 ms

CSV 저장: reports/day04_homography_results.csv
결과 이미지: outputs/day04/homography
```

![[{A2E67B72-8507-4319-A5E8-9346C8F10F85}.png]]

CSV에서도 같은 값을 확인할 수 있습니다.

```bash
cat reports/day04_homography_results.csv
```

결과 이미지는 다음 폴더에 저장됩니다.

```text
outputs/day04/homography/
```

특히 다음 이미지를 봅니다.

```text
target_normal_homography.jpg
target_shift_homography.jpg
target_rot15_homography.jpg
target_scale90_homography.jpg
```

초록색 사각형은 Homography가 계산한 **광천김 Reference 로고의 위치**입니다.

## 33.1 ## `target_normal` — 기준 로고 위치를 정확하게 찾았습니다

결과:

```text
Good Match   = 617
Inlier       = 610
Inlier Ratio = 0.989
Status       = FOUND
```

결과 이미지를 보면 초록색 사각형이 실제:

```
KC 로고
+
(주)광천김
+
영문 글자
```

영역을 정확하게 감싸고 있습니다.

또한 Good Match `617개` 중 `610개`가 Inlier로 남았습니다.

```
Good Match 617
        ↓
RANSAC
        ↓
Inlier 610
        ↓
대부분의 특징점 위치 관계가 일치
```

기준 상태에서는 Homography가 로고 위치를 매우 안정적으로 찾았습니다.

## 33.2 `target_shift` — 위치가 바뀌어도 로고 위치를 찾았습니다

결과:

```
Good Match   = 597
Inlier       = 594
Inlier Ratio = 0.995
Status       = FOUND
```

위치가 이동한 이미지에서도 Homography 계산에 성공했습니다.

```
Good Match 597
        ↓
Inlier 594
        ↓
Ratio 0.995
```

즉 대부분의 Match가 같은 위치 관계를 유지하고 있었습니다.

결과 이미지에서도 초록색 사각형이 이동한 로고 위치를 따라간 것을 확인할 수 있습니다.

이번 `shift` 이미지는 로고가 화면 아래쪽에 가까워 일부가 잘려 보이지만, 특징점의 위치 관계를 이용하여 로고 영역을 계속 추정했습니다.

---

## 33.3 `target_rot15` — 회전된 로고를 사각형도 함께 따라갔습니다

결과:

```
Good Match   = 459
Inlier       = 447
Inlier Ratio = 0.974
Status       = FOUND
```

15° 회전하면서 Good Match 수는 줄었습니다.

하지만 `459개` 중 `447개`가 Inlier로 남았습니다.

결과 이미지를 보면 중요한 차이를 확인할 수 있습니다.

```
제품이 15° 회전
        ↓
로고도 회전
        ↓
Homography 사각형도
같은 방향으로 회전
```

즉 Homography는 단순한 가로·세로 BBox가 아니라 **회전된 라벨의 방향까지 따라가며 위치를 표시**했습니다.

앞의 Template Matching에서는 15° 회전 후 로고 위치를 제대로 찾지 못했지만, SIFT 특징점과 Homography를 이용하자 회전된 로고 위치를 찾을 수 있었습니다.

---

## 33.4 `target_scale90` — 크기가 달라져도 로고 위치를 찾았습니다

결과:

```
Good Match   = 412
Inlier       = 402
Inlier Ratio = 0.976
Status       = FOUND
```

제품을 90% 크기로 줄이면서 Good Match 수는 감소했습니다.

```
normal
Good Match = 617

        ↓ 크기 변화

scale90
Good Match = 412
```

하지만 `402개`가 Inlier로 남았고 Homography도 정상적으로 계산되었습니다.

결과 이미지에서도 초록색 사각형이 **작아진 로고 크기에 맞춰 함께 작아진 것**을 확인할 수 있습니다.

```
제품 크기 감소
        ↓
로고 크기 감소
        ↓
Homography
        ↓
작아진 로고 위치를
사각형으로 표시
```

---

## 33.5 네 결과를 비교합니다

이번 결과를 정리하면 다음과 같습니다.

|조건|Good Match|Inlier|Inlier Ratio|결과|
|---|---|---|---|---|
|`normal`|617|610|0.989|로고 위치 정확하게 찾음|
|`shift`|597|594|0.995|이동한 로고 위치 찾음|
|`rot15`|459|447|0.974|회전된 로고 위치 찾음|
|`scale90`|412|402|0.976|작아진 로고 위치 찾음|

네 조건 모두:

```
status = FOUND
```

가 나왔습니다.

또한 모든 조건에서 Inlier Ratio가 약:

```
0.97 ~ 0.99
```

로 높게 나타났습니다.

쉽게 해석하면:

> **SIFT에서 찾은 Good Match 대부분이 실제로 같은 위치 관계를 잘 따르고 있었다는 뜻입니다.**

---

## 33.6 이번 결과에서 확인한 핵심

앞의 실습부터 연결하면 다음과 같습니다.

```
SIFT
→ 비슷한 특징점들을 많이 찾음

        ↓

Good Match
→ 비교적 좋은 연결을 남김

        ↓

RANSAC
→ 그중 위치 관계까지
   자연스러운 연결을 선택

        ↓

Inlier
→ 실제 위치 계산에 사용할
   신뢰할 수 있는 특징점

        ↓

Homography
→ 라벨 전체 위치 계산

        ↓

초록색 사각형으로 표시
```

실제 광천김 결과에서는:

```
normal
→ 정확한 위치

shift
→ 이동한 위치

rot15
→ 회전된 위치

scale90
→ 작아진 위치
```

를 모두 찾을 수 있었습니다.

> 이번 실습을 통해 특징점 Matching만 하는 것에서 끝나는 것이 아니라, Homography를 이용하면 특징점들의 위치 관계를 사용하여 실제 라벨 전체의 위치와 방향까지 계산할 수 있음을 확인했습니다.

---

## 33.7 그렇다고 Homography가 항상 성공하는 것은 아닙니다

이번 광천김 데이터에서는 결과가 잘 나왔지만 다음과 같은 경우에는 실패할 수 있습니다.

```
특징점이 너무 적은 경우

심하게 흐린 이미지

라벨이 너무 작게 보이는 경우

라벨 대부분이 가려진 경우

잘못된 Match가 너무 많은 경우
```

따라서 실제 검사에서는:

```
FOUND 확인
        +
Inlier 확인
        +
Inlier Ratio 확인
        +
실제 사각형 위치 확인
```

을 함께 보는 것이 중요합니다.

이번 실습에서는 네 조건 모두 높은 Inlier Ratio와 함께 실제 광천김 로고 위치를 잘 찾았습니다.

---
# PART 34~37 — 검사 결과 통합 비교 및 Day05 연결

---

# 34. 지금까지의 검사 결과를 한 번에 비교합니다

지금까지 같은 광천김 Reference와 같은 네 개의 Target에 여러 방법을 적용했습니다.

```text
같은 데이터 
    ↓ 
Template Matching 
ORB 
SIFT 
Homography 
    ↓ 
결과 비교
```

이 부분은 새로운 검사를 하는 것이 아닙니다.

앞에서 측정한 결과를 한곳에 모아:

> 어떤 방법이 위치·회전·크기 변화에서 어떻게 달라졌는가?

를 확인하는 정리 단계입니다.

---

# 35. `31_day04_compare.py` — 앞의 측정 결과를 하나로 모읍니다

앞에서 이미 다음 CSV가 만들어졌습니다.

```text
reports/day04_template_results.csv
reports/day04_orb_results.csv
reports/day04_sift_results.csv
reports/day04_homography_results.csv
```

이번 프로그램은 이미 저장된 값을 읽어서 하나의 비교표로 합칩니다.

앞에서 측정한 값을 한 표로 정리합니다.

```
Template 결과 ─┐
ORB 결과 ──────┤
SIFT 결과 ─────┼→ 하나의 비교 CSV
Homography 결과┘
```

파일을 만듭니다.

```text
src/31_day04_compare.py
```

```python
from pathlib import Path
import csv


# --------------------------------------------------
# 1. 기존 검사 결과 CSV와 최종 비교 CSV 경로 설정
#
# 무엇을 하나요?
# - 앞에서 실행한 4가지 검사 방법의 CSV 경로를 지정합니다.
# - 네 가지 검사 결과를 하나로 합칠 CSV 경로를 지정합니다.
#
# 입력 파일:
# day04_template_results.csv
# day04_orb_results.csv
# day04_sift_results.csv
# day04_homography_results.csv
#
# 출력 파일:
# day04_rotation_compare.csv
#
# 왜 필요한가요?
# 각 검사 방법의 결과가 서로 다른 CSV에 저장되어 있으므로
# 하나의 표로 합치기 위해 파일 경로를 준비합니다.
# --------------------------------------------------

report_dir = Path("reports")

paths = {
    "template": report_dir / "day04_template_results.csv",
    "orb": report_dir / "day04_orb_results.csv",
    "sift": report_dir / "day04_sift_results.csv",
    "homography": report_dir / "day04_homography_results.csv",
}

output_path = (
    report_dir /
    "day04_rotation_compare.csv"
)


# --------------------------------------------------
# 2. CSV 파일을 읽어 condition 기준으로 정리하는 함수
#
# 무엇을 하나요?
# - 전달받은 CSV 파일이 존재하는지 확인합니다.
# - csv.DictReader()로 CSV 데이터를 읽습니다.
# - condition을 Key로 사용하는 딕셔너리를 만듭니다.
#
# condition이란?
# 이미지에 적용한 검사 조건을 의미합니다.
#
# normal  : 정상
# shift   : 위치 이동
# rot15   : 15도 회전
# scale90 : 90% 크기 변화
#
# 왜 condition을 Key로 사용하나요?
# 서로 다른 CSV에서 같은 조건의 결과를 찾아
# 하나의 행으로 합치기 위해서입니다.
#
# 예:
# template["normal"]
# orb["normal"]
# sift["normal"]
# homography["normal"]
#
# 위 네 결과는 모두 normal 조건의 측정값입니다.
# --------------------------------------------------

def read_results(path):

    # 입력 CSV 파일이 없으면 오류 발생
    if not path.exists():
        raise FileNotFoundError(path)

    # CSV 파일 열기
    with path.open(
        "r",
        encoding="utf-8",
        newline="",
    ) as file:

        # CSV의 컬럼명을 Key로 사용하여 읽기
        rows = csv.DictReader(file)

        # condition을 Key로 하는 딕셔너리 반환
        return {
            row["condition"]: row
            for row in rows
        }


# --------------------------------------------------
# 3. 앞에서 만든 네 가지 검사 결과 CSV 읽기
#
# 무엇을 하나요?
# - Template Matching 결과를 읽습니다.
# - ORB Matching 결과를 읽습니다.
# - SIFT Matching 결과를 읽습니다.
# - Homography 결과를 읽습니다.
#
# 각 CSV는 read_results() 함수를 이용하여
# condition을 Key로 하는 딕셔너리로 변환합니다.
#
# 목적:
# 네 가지 검사 방법의 측정값을
# 같은 이미지 조건별로 비교할 준비를 합니다.
# --------------------------------------------------

template = read_results(
    paths["template"]
)

orb = read_results(
    paths["orb"]
)

sift = read_results(
    paths["sift"]
)

homography = read_results(
    paths["homography"]
)


# --------------------------------------------------
# 4. 비교할 네 가지 이미지 조건과 결과 리스트 준비
#
# 무엇을 하나요?
# - 비교할 이미지 조건 4개를 지정합니다.
# - 최종 비교 결과를 저장할 빈 리스트를 만듭니다.
#
# conditions:
# 어떤 순서로 결과를 정리할지 결정합니다.
#
# rows:
# 네 가지 검사 방법의 측정값을 합친 결과를 저장합니다.
#
# 왜 필요한가요?
# 모든 검사 방법에서 같은 조건을 기준으로
# 결과를 정리해야 공정하게 비교할 수 있기 때문입니다.
# --------------------------------------------------

conditions = [
    "normal",
    "shift",
    "rot15",
    "scale90",
]


rows = []


# --------------------------------------------------
# 5. 같은 이미지 조건의 검사 결과를 하나의 행으로 합치기
#
# 무엇을 하나요?
# - 네 가지 이미지 조건을 하나씩 반복합니다.
# - 각 조건에 해당하는 검사 결과를 가져옵니다.
# - 측정값들을 하나의 딕셔너리로 정리합니다.
#
# 가져오는 측정값:
#
# template_score:
# Template Matching에서 계산한 유사도 점수
#
# orb_good_matches:
# ORB Ratio Test를 통과한 특징점 대응 개수
#
# sift_good_matches:
# SIFT Ratio Test를 통과한 특징점 대응 개수
#
# homography_inliers:
# RANSAC에서 선택한 Inlier 개수
#
# inlier_ratio:
# Good Match 중 Inlier가 차지하는 비율
#
# homography_status:
# Homography 계산 성공 여부
#
# 왜 필요한가요?
# 검사 방법마다 다른 CSV에 저장된 결과를
# 동일한 이미지 조건을 기준으로 연결하기 위해서입니다.
# --------------------------------------------------

for condition in conditions:

    row = {
        "condition": condition,

        # Template Matching 점수
        "template_score":
            template[condition][
                "template_score"
            ],

        # ORB Good Match 개수
        "orb_good_matches":
            orb[condition][
                "good_matches"
            ],

        # SIFT Good Match 개수
        "sift_good_matches":
            sift[condition][
                "good_matches"
            ],

        # Homography Inlier 개수
        "homography_inliers":
            homography[condition][
                "inliers"
            ],

        # Homography Inlier Ratio
        "inlier_ratio":
            homography[condition][
                "inlier_ratio"
            ],

        # Homography 계산 성공 여부
        "homography_status":
            homography[condition][
                "status"
            ],
    }


    # --------------------------------------------------
    # 6. 통합한 측정값을 최종 결과 리스트에 추가
    #
    # 무엇을 하나요?
    # - 5번에서 만든 하나의 조건별 결과를 저장합니다.
    # - normal, shift, rot15, scale90을 순서대로 추가합니다.
    #
    # 반복이 끝나면 rows에는 총 4개의 행이 들어갑니다.
    #
    # 이 데이터를 마지막에 CSV 파일로 저장합니다.
    # --------------------------------------------------

    rows.append(row)


# --------------------------------------------------
# 7. 최종 비교 CSV의 컬럼명과 순서 지정
#
# 무엇을 하나요?
# - 최종 CSV에 저장할 컬럼을 정의합니다.
# - 각 검사 방법의 측정값이 표시될 순서를 정합니다.
#
# CSV 컬럼:
#
# condition
# template_score
# orb_good_matches
# sift_good_matches
# homography_inliers
# inlier_ratio
# homography_status
#
# 왜 필요한가요?
# CSV를 열었을 때 검사 조건별 측정값이
# 일관된 순서로 표시되도록 하기 위해서입니다.
# --------------------------------------------------

fieldnames = [
    "condition",
    "template_score",
    "orb_good_matches",
    "sift_good_matches",
    "homography_inliers",
    "inlier_ratio",
    "homography_status",
]


# --------------------------------------------------
# 8. 네 가지 검사 결과를 하나의 비교 CSV로 저장
#
# 무엇을 하나요?
# - day04_rotation_compare.csv 파일을 생성합니다.
# - 7번에서 정한 컬럼명을 첫 번째 행에 작성합니다.
# - 6번에서 정리한 네 가지 조건의 결과를 저장합니다.
#
# 저장되는 행:
# normal
# shift
# rot15
# scale90
#
# 최종 결과:
# reports/day04_rotation_compare.csv
#
# 이 CSV를 이용하여
# 이미지의 위치·회전·크기 변화에 따른
# 검사 방법별 측정값을 비교할 수 있습니다.
# --------------------------------------------------

with output_path.open(
    "w",
    encoding="utf-8",
    newline="",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=fieldnames,
    )

    # CSV 컬럼 제목 작성
    writer.writeheader()

    # 네 가지 조건의 측정 결과 작성
    writer.writerows(rows)


# --------------------------------------------------
# 9. 저장 경로와 조건별 측정 결과를 화면에 출력
#
# 무엇을 하나요?
# - 최종 CSV 파일의 저장 경로를 출력합니다.
# - 네 가지 이미지 조건의 주요 측정값을 출력합니다.
#
# 출력 항목:
# condition
# Template Score
# ORB Good Match
# SIFT Good Match
# Homography Inlier
# Homography Status
#
# 목적:
# CSV 파일을 직접 열기 전에도
# 터미널에서 검사 방법별 결과를 확인할 수 있습니다.
#
# 주의:
# 이 코드는 측정값을 비교표로 정리하는 프로그램입니다.
# 어떤 검사 방법이 가장 우수한지 자동으로 판단하거나
# OK / NG 판정을 수행하지는 않습니다.
# --------------------------------------------------

print("저장:", output_path)
print()


for row in rows:

    print(
        row["condition"],
        "| Template:",
        row["template_score"],
        "| ORB:",
        row["orb_good_matches"],
        "| SIFT:",
        row["sift_good_matches"],
        "| Inlier:",
        row["homography_inliers"],
        "| Status:",
        row["homography_status"],
    )
```
### 의사코드

```
[1. 기존 검사 결과 CSV와 최종 비교 CSV 경로 설정]

reports 폴더를 지정한다.

Template Matching 결과 CSV 경로를 정한다.
ORB 결과 CSV 경로를 정한다.
SIFT 결과 CSV 경로를 정한다.
Homography 결과 CSV 경로를 정한다.

최종 비교 결과를 저장할
day04_rotation_compare.csv 경로를 정한다.

          ↓

[2. CSV 파일을 condition 기준으로 정리하는 함수]

read_results() 함수를 만든다.

입력받은 CSV 파일이 있는지 확인한다.

파일이 없다면 오류를 발생시킨다.

파일이 있다면 CSV를 읽는다.

CSV의 각 행에서 condition 값을 확인한다.

condition을 Key로 사용하여
각 행의 측정값을 딕셔너리로 저장한다.

완성된 딕셔너리를 반환한다.

          ↓

[3. 앞에서 만든 네 가지 검사 결과 CSV 읽기]

read_results() 함수를 호출한다.

Template Matching 결과를 읽는다.
ORB Matching 결과를 읽는다.
SIFT Matching 결과를 읽는다.
Homography 결과를 읽는다.

각 결과를 condition 기준으로 정리한다.

          ↓

[4. 비교할 네 가지 이미지 조건과 결과 리스트 준비]

비교할 이미지 조건 4개를 준비한다.

normal
shift
rot15
scale90

최종 비교 결과를 저장할
빈 rows 리스트를 만든다.

          ↓

[5. 같은 이미지 조건의 검사 결과를 하나의 행으로 합치기]

이미지 조건을 하나씩 반복한다.

현재 condition에 해당하는
Template Matching 점수를 가져온다.

같은 condition의
ORB Good Match 개수를 가져온다.

같은 condition의
SIFT Good Match 개수를 가져온다.

같은 condition의
Homography Inlier 개수를 가져온다.

같은 condition의
Homography Inlier Ratio를 가져온다.

같은 condition의
Homography Status를 가져온다.

가져온 측정값을
하나의 딕셔너리로 정리한다.

          ↓

[6. 통합한 측정값을 최종 결과 리스트에 추가]

현재 condition의 통합 결과를
rows 리스트에 추가한다.

아직 비교할 조건이 남았다면
5번으로 돌아가 다음 조건을 처리한다.

네 가지 조건의 처리가 모두 끝나면
7번으로 이동한다.

          ↓

[7. 최종 비교 CSV의 컬럼명과 순서 지정]

CSV에 저장할 컬럼을 준비한다.

condition
template_score
orb_good_matches
sift_good_matches
homography_inliers
inlier_ratio
homography_status

컬럼 순서를 fieldnames에 저장한다.

          ↓

[8. 네 가지 검사 결과를 하나의 비교 CSV로 저장]

day04_rotation_compare.csv 파일을 연다.

CSV 컬럼 제목을 작성한다.

rows에 저장된 네 가지 조건의
측정 결과를 순서대로 작성한다.

파일 저장을 완료한다.

          ↓

[9. 저장 경로와 조건별 측정 결과를 화면에 출력]

최종 CSV 저장 경로를 출력한다.

rows 리스트를 하나씩 반복한다.

현재 이미지 조건을 출력한다.
Template Matching 점수를 출력한다.
ORB Good Match 개수를 출력한다.
SIFT Good Match 개수를 출력한다.
Homography Inlier 개수를 출력한다.
Homography Status를 출력한다.

네 가지 조건의 출력을 완료한다.

          ↓

[최종 결과]

reports/day04_rotation_compare.csv

          ↓

Template Matching
       +
ORB Matching
       +
SIFT Matching
       +
Homography
       ↓
네 가지 검사 결과 통합
       ↓
normal / shift / rot15 / scale90
조건별 측정값 비교
       ↓
어떤 조건에서 각 검사 방법의
측정값이 달라지는지 분석한다.
```

실행합니다.

```bash
python src/31_day04_compare.py
```

결과를 확인합니다.

```bash
cat reports/day04_rotation_compare.csv
```

---

# 36. 실제 결과를 한눈에 비교합니다

지금까지 측정한 결과를 모으면 다음과 같습니다.

| 조건        | Template Score | ORB Good Match | SIFT Good Match | Homography Inlier | Homography |
| --------- | -------------: | -------------: | --------------: | ----------------: | ---------- |
| `normal`  |         0.9997 |            276 |             615 |               610 | FOUND      |
| `shift`   |         0.4534 |            245 |             596 |               594 | FOUND      |
| `rot15`   |         0.4750 |            232 |             458 |               447 | FOUND      |
| `scale90` |         0.6240 |            156 |             413 |               402 | FOUND      |

이번 결과에서 확인할 핵심만 정리합니다.

```text
Template Matching
→ normal에서는 매우 잘 찾음
→ 이번 shift와 rot15에서는 제대로 찾지 못함
→ 크기 변화에서도 Score 감소


ORB
→ 위치·회전·크기가 달라져도
   특징점 Matching이 어느 정도 유지됨


SIFT
→ 네 조건 모두에서
   많은 특징점 Matching이 유지됨


SIFT + Homography
→ 네 조건 모두 FOUND
→ 이동·회전·크기가 달라져도
   실제 광천김 로고 위치를 사각형으로 찾음
```

단, ORB와 SIFT의 Good Match 숫자를 그대로 비교하여:

```
SIFT 615 > ORB 276
→ SIFT가 두 배 더 정확하다
```

라고 판단하지 않습니다.

두 방법은 Keypoint와 Descriptor를 만드는 방식이 다르기 때문입니다.

이번 실습에서는 다음 정도로 이해하면 충분합니다.

> 검사 조건이 단순하면 Template Matching을 사용할 수 있고, 회전·크기 변화에서도 특징을 비교하려면 ORB/SIFT를 검토하며, 실제 라벨 위치까지 찾으려면 Feature Matching + Homography를 사용할 수 있습니다.

---

# 37. 오늘은 최종 Rule을 만들지 않습니다

4일차에서는 다음 값을 측정했습니다.

```text
Template Score
ORB Good Match
SIFT Good Match
Homography Inlier / Ratio
```

하지만 오늘 측정한 값은 **기술의 동작을 비교하기 위한 값**입니다.

아직 다음과 같은 최종 기준은 만들지 않습니다.

```
Score가 몇 이상이면 OK?
Good Match가 몇 개 이상이면 OK?
Inlier가 몇 개 이상이면 OK?
```

현재는 몇 장의 실험 이미지밖에 사용하지 않았기 때문입니다.

```
4일차
→ 측정 방법과 기술의 특징 확인

        ↓

5일차
→ 여러 Calibration 이미지 측정
→ 정상·불량 값 비교
→ Threshold 결정
→ 최종 Rule
```

> 4일차는 무엇을 측정할지 배우는 날이고, 5일차는 그 측정값으로 어디서 OK / NG를 나눌지 결정하는 날입니다.

---
# PART 38~44 — Mini Challenge: 실제 촬영 이미지에 검사 방법 적용

---

# 38. Mini Challenge — 실제 촬영 이미지에 직접 적용해 봅니다

지금까지는 하나의 `full_front.jpg`를 코드로 변형하여 다음 조건을 비교했습니다.

```text
normal
shift
rot15
scale90
```

그리고 같은 광천김 로고를 다음 방법으로 검사했습니다.

```
Template Matching
→ 전체 모양 비교

ORB
→ 빠른 특징점 비교

SIFT
→ 특징점 비교

SIFT + Homography
→ 특징점을 이용해
   실제 로고 위치까지 찾기
```

이제 마지막 Mini Challenge에서는 **코드로 만든 변형 이미지가 아니라 실제로 다시 촬영한 사진**을 사용합니다.

오늘의 질문은 하나입니다.

> 실제 촬영에서는 제품의 위치·각도·거리가 조금씩 달라질 수 있습니다.  
> 이런 이미지에서 광천김 로고를 찾으려면 어떤 방법을 사용하는 것이 좋을까요?

정답 기술은 미리 정하지 않습니다.

4일차에서 배운 결과를 참고하여 직접 하나를 선택합니다.

---

# 39. 실제 촬영한 Challenge 이미지를 사용합니다

다음 이미지를 준비합니다.

```
samples/day04/challenge/
├── shift.jpg
├── rotate.jpg
├── distance.jpg
└── tilt_glare.jpg
```

각 이미지의 조건은 다음과 같습니다.

|이미지|실제 촬영 조건|
|---|---|
|`shift.jpg`|제품 위치를 조금 이동|
|`rotate.jpg`|제품을 조금 회전|
|`distance.jpg`|촬영 거리를 변경|
|`tilt_glare.jpg`|제품을 기울여 반사·원근 변화 발생|

앞의 코드 생성 Target과 차이가 있습니다.

```
앞의 실습
→ 프로그램으로 조건을 정확하게 변경

Mini Challenge
→ 사람이 제품을 다시 촬영
→ 실제 촬영 편차도 함께 발생
```

따라서 결과가 앞의 실험처럼 깔끔하게 나오지 않아도 됩니다.

그 차이를 확인하는 것도 Mini Challenge의 목적입니다.

---

# 40. 코딩 전에 사용할 방법을 하나 선택합니다

먼저 이미지 조건을 보고 방법을 선택합니다.

```
회전·크기 변화가 거의 없다
        ↓
Template Matching


위치·회전이 조금 달라진다
        ↓
ORB 또는 SIFT


회전된 로고의
실제 위치까지 표시하고 싶다
        ↓
SIFT + Homography
```

학생은 다음 중 **한 가지 방법을 먼저 선택**합니다.

```
Template Matching

ORB

SIFT

SIFT + Homography
```

모든 방법을 다 사용할 필요는 없습니다.

`reports/day04_method_selection.md`에는 다음 정도만 간단하게 기록합니다.

```
# Day04 Mini Challenge

검사 이미지:

가장 크게 달라진 조건:

선택한 방법:

선택 이유:

예상되는 한계:
```

예:

```
검사 이미지:
rotate.jpg

가장 크게 달라진 조건:
회전

선택한 방법:
SIFT + Homography

선택 이유:
회전된 상태에서도 특징점을 비교하고
로고 위치까지 확인하고 싶기 때문이다.

예상되는 한계:
반사가 심하거나 특징점이 부족하면
결과가 불안정할 수 있다.
```

---

# 41. `32_day04_challenge.py` — 앞에서 배운 코드를 재사용합니다

파일을 만듭니다.

```
src/32_day04_challenge.py
```

새로운 알고리즘을 만드는 것이 아닙니다.

앞에서 작성한 코드 중 **자신이 선택한 방법에 필요한 부분만 가져와 사용**합니다.

```
Template 선택
→ 27_template_matching.py 참고

ORB 선택
→ 28_orb_matching.py 참고

SIFT 선택
→ 29_sift_matching.py 참고

SIFT + Homography 선택
→ 30_homography.py 참고
```

기본 코드는 다음과 같이 시작합니다.

```python
from pathlib import Path

import cv2


reference_path = Path(
    "samples/day04/reference/reference_label.jpg"
)

challenge_dir = Path(
    "samples/day04/challenge"
)

output_dir = Path(
    "outputs/day04/challenge"
)

output_dir.mkdir(
    parents=True,
    exist_ok=True,
)


challenge_files = [
    challenge_dir / "shift.jpg",
    challenge_dir / "rotate.jpg",
    challenge_dir / "distance.jpg",
    challenge_dir / "tilt_glare.jpg",
]


# --------------------------------------------------
# 1. 사용할 방법을 선택합니다.
#
# TEMPLATE
# ORB
# SIFT
# SIFT_HOMOGRAPHY
# --------------------------------------------------

selected_method = "CHOOSE_METHOD"


reference = cv2.imread(
    str(reference_path)
)

if reference is None:
    raise FileNotFoundError(
        reference_path
    )


# --------------------------------------------------
# 2. Challenge 이미지를 하나씩 읽습니다.
# --------------------------------------------------

for target_path in challenge_files:

    target = cv2.imread(
        str(target_path)
    )

    if target is None:
        print(
            "읽기 실패:",
            target_path
        )
        continue


    # --------------------------------------------------
    # TODO
    #
    # 자신이 선택한 방법에 맞게
    # 27~30번 코드에서 필요한 부분을 가져옵니다.
    #
    # TEMPLATE
    # → Score + Location
    #
    # ORB
    # → Good Match
    #
    # SIFT
    # → Good Match
    #
    # SIFT_HOMOGRAPHY
    # → Good Match + Inlier + 로고 위치
    # --------------------------------------------------


    print(
        target_path.name,
        "method=",
        selected_method,
    )
```

실행합니다.

```
python src/32_day04_challenge.py
```

중요한 것은 코드를 처음부터 다시 만드는 것이 아닙니다.

```
검사 문제 확인
        ↓
방법 선택
        ↓
앞에서 만든 코드 재사용
        ↓
실제 촬영 이미지에 적용
        ↓
결과 확인
```

이 과정을 직접 해보는 것이 Mini Challenge의 핵심입니다.

---

# 42. 결과를 확인합니다

선택한 방법에 따라 확인할 값이 달라집니다.

```
Template Matching
→ Score
→ 실제 로고 위치를 찾았는가?


ORB / SIFT
→ Good Match
→ 실제 같은 특징끼리 연결되었는가?


SIFT + Homography
→ Good Match
→ Inlier
→ Inlier Ratio
→ 사각형이 실제 로고를 감싸는가?
```

네 이미지를 비교합니다.

|이미지|주요 변화|결과|관찰|
|---|---|---|---|
|`shift.jpg`|위치|직접 기록||
|`rotate.jpg`|회전|직접 기록||
|`distance.jpg`|크기|직접 기록||
|`tilt_glare.jpg`|기울기·반사|직접 기록||

성공 여부만 적는 것이 아니라 한 줄 정도 이유도 적습니다.

예:

```
rotate.jpg
→ SIFT + Homography 사용
→ 회전된 로고 위치를 찾음


tilt_glare.jpg
→ Match 수 감소
→ 비닐 반사와 기울기의 영향을 받은 것으로 보임
```

아직 이 결과로 최종 OK / NG Threshold를 만들지는 않습니다.

그 기준은 5일차 Calibration에서 결정합니다.

---

# 43. Mini Challenge 마무리

마지막으로 다음 질문에 답합니다.

```
1. 어떤 방법을 선택했는가?

2. 왜 그 방법을 선택했는가?

3. 어떤 이미지에서는 잘 동작했는가?

4. 어떤 이미지에서는 결과가 나빠졌는가?

5. 실제 검사에 사용한다면
   어떤 조건을 더 테스트해야 하는가?
```

Mini Challenge가 끝났다면 다음 내용을 이해한 것입니다.

```
검사 이미지 확인
        ↓
어떤 변화가 있는지 판단
        ↓
적절한 검사 방법 선택
        ↓
기존 코드 재사용
        ↓
실제 이미지에서 측정
        ↓
성공 / 실패 원인 관찰
```

> 4일차 Mini Challenge의 목표는 가장 복잡한 기술을 사용하는 것이 아니라, 검사 조건을 보고 적절한 방법을 스스로 선택해 보는 것입니다.

---

# 44. Mini Challenge 결과를 기록합니다

Mini Challenge가 끝났다면 선택한 방법과 결과를 간단하게 기록합니다.

파일:

```text
reports/day04_method_selection.md
```

작성 예시

```
# Day04 Mini Challenge

검사 이미지:
rotate.jpg

주요 변화:
회전

선택한 방법:
SIFT + Homography

선택 이유:
회전된 로고의 특징을 비교하고
실제 위치까지 확인하기 위해 선택했다.

결과:
로고 위치를 사각형으로 찾았다.

관찰:
회전된 이미지에서도 특징점 Matching이 유지되었다.

한계:
반사나 가림이 심해지면
특징점이 줄어들 수 있다.

Threshold:
Day05 Calibration 후 결정
```

어떤 방법을 선택했고, 왜 선택했으며, 실제 결과가 어땠는지만 기록하면 됩니다.

---
# PART 45~49 — 4일차 정리 및 수업 마무리

---
# 45. README에 4일차 내용을 정리합니다

`README.md`에 다음 내용을 추가합니다.

```
## Day 04 — Template · Feature Matching

### 학습 내용

광천김 포장지의 로고 영역을 기준으로
다음 방법을 실습했다.

- GrabCut
- MeanShift
- Template Matching
- ORB
- SIFT
- RANSAC
- Homography

### 비교 데이터

- normal
- shift
- rot15
- scale90

### 주요 결과

Template Matching
→ 기준 상태에서는 잘 찾았지만
   회전·크기 변화에서 Score가 감소했다.

ORB
→ 위치·회전·크기 변화에서도
   특징점 Matching이 어느 정도 유지되었다.

SIFT
→ 회전·크기 변화에서도
   많은 특징점 Matching이 유지되었다.

SIFT + Homography
→ 특징점의 위치 관계를 이용하여
   회전·이동·크기 변화가 있는 로고 위치를
   사각형으로 찾을 수 있었다.

### Mini Challenge

실제 다시 촬영한 이미지에
4일차에서 배운 방법 중 하나를 선택하여 적용하고
선택 이유와 결과를 기록했다.

### 주요 결과 파일

- reports/day04_template_results.csv
- reports/day04_orb_results.csv
- reports/day04_sift_results.csv
- reports/day04_homography_results.csv
- reports/day04_rotation_compare.csv
- reports/day04_method_selection.md
```

README는 **무엇을 실습했고 어떤 결과가 나왔는지 다른 사람이 빠르게 이해할 수 있는 정도**로 작성하면 충분합니다.

---

# 46. Git에 4일차 작업을 기록합니다

먼저 변경 내용을 확인합니다.

```bash
git status
```

코드와 문서를 추가합니다.

```bash
git add src reports README.md
```

Commit 합니다.

```bash
git commit -m "feat: complete day04 image matching practice"
```

로그를 확인합니다.

```bash
git log --oneline -5
```

GitHub 사용이 허용된 환경이라면 업로드합니다.

```bash
git push origin main
```

기관 정책상 외부 GitHub 업로드가 허용되지 않는 경우에는 `push`하지 않고 Local Git 기록까지만 남깁니다.

---

# 47. 4일차 자가 체크

수업을 마치기 전에 다음 항목만 확인합니다.

- [ ] GrabCut이 전경과 배경을 분리하는 방법임을 설명할 수 있다.
- [ ] MeanShift가 비슷한 색상 영역을 단순화하는 전처리임을 설명할 수 있다.
- [ ] Template과 Target의 차이를 설명할 수 있다.
- [ ] Template Matching의 Score와 Location을 확인할 수 있다.
- [ ] Keypoint와 Descriptor의 차이를 설명할 수 있다.
- [ ] ORB와 SIFT가 특징점을 비교하는 방법임을 이해했다.
- [ ] Good Match의 의미를 설명할 수 있다.
- [ ] RANSAC이 잘못된 Match를 줄이는 역할임을 설명할 수 있다.
- [ ] Homography로 라벨 위치를 사각형으로 표시했다.
- [ ] normal / shift / rot15 / scale90 결과를 비교했다.
- [ ] Mini Challenge에서 검사 방법을 직접 선택했다.
- [ ] 선택한 이유와 결과를 기록했다.
- [ ] README를 작성했다.
- [ ] Git Commit을 남겼다.

모든 세부 함수와 Parameter를 외울 필요는 없습니다.

중요한 것은:

```
검사 문제를 본다
        ↓
어떤 변화가 있는지 확인한다
        ↓
적절한 방법을 선택한다
        ↓
측정 결과를 확인한다
```

라는 흐름을 이해하는 것입니다.

---

# 48. 5일차로 연결합니다

4일차에서는 여러 방법을 사용하여 다음과 같은 **측정값**을 만들었습니다.

```
Template Score

ORB Good Match

SIFT Good Match

Homography Inlier / Ratio
```

하지만 아직 이 값으로 최종 `OK / NG` 기준을 만들지는 않았습니다.

5일차에서는 여러 Calibration 이미지에서 값을 반복 측정합니다.

```
여러 Calibration 이미지
        ↓
측정값 수집
        ↓
정상 / 불량 값 비교
        ↓
Threshold 결정
        ↓
Rule 작성
        ↓
Test
```

즉:

> 4일차는 어떤 방법으로 무엇을 측정할지 배웠고, 5일차는 그 측정값을 이용하여 실제 검사 기준을 만드는 단계입니다.

---

# 49. 4일차 마무리

오늘의 핵심만 다시 정리합니다.

```
같은 모양을 그대로 찾고 싶다
→ Template Matching

특징적인 점을 비교하고 싶다
→ ORB / SIFT

회전·크기 변화가 있는
평면 라벨의 위치까지 찾고 싶다
→ SIFT + RANSAC + Homography

제품과 배경을 분리해야 한다
→ GrabCut

색상 영역을 단순화할 필요가 있다
→ MeanShift
```

4일차의 목표는 모든 기술을 한꺼번에 사용하는 것이 아닙니다.

> 검사 문제와 이미지 조건을 보고 필요한 방법을 선택할 수 있는 것이 핵심입니다.