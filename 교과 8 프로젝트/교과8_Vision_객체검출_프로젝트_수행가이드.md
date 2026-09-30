
> **대상**: 교과 8 `[중간] Vision 기반 객체검출 모듈`  
> **운영**: 72시간 · 1일 8시간 × 9일 팀 프로젝트  
> **선행 프로젝트**: 교과 7 조각김치 이물검출 라벨링 프로젝트  
> **입력 데이터**: 교과 7에서 검수·확정한 4K JPG 900장 + YOLO TXT 900개  
> **핵심 기술**: Python · Ultralytics YOLO11 · PyTorch · OpenCV · Pandas · 데이터 QA · 모델 평가 · Git  
> **핵심 결과**: Baseline 모델 · 개선 모델 · Final Model · 성능보고서 · 실패분석 · Auto Label · 교과 14 Handoff  
> **보안**: 기업 제공 데이터는 NDA와 기관 보안정책을 우선하며 외부 공개 저장소·외부 클라우드에 업로드하지 않습니다.

---

# 0. 먼저 전체 프로젝트 흐름을 이해합니다

교과 7에서는 **사람이 신뢰할 수 있는 정답 데이터**를 만들었습니다.

교과 8에서는 그 정답 데이터를 이용하여 **AI가 새로운 이미지에서 이물질의 위치와 종류를 자동으로 찾도록 학습**합니다.

![[Pasted image 20260928154226.png]]

교과 7에서 확정한 데이터 구조와 Class 기준을 교과 8에서 임의로 바꾸지 않습니다.

```text
[교과 7]

4K JPG 900장
+
검수된 YOLO TXT 900개
+
Class / BBox 기준
+
Dataset Manifest
+
QA Summary
+
source_dataset / original_split / scene_type
        ↓
        ↓  교과 8로 연결
        ↓

[교과 8]

교과 7 FINAL 결과 확인
        ↓
학습 관점 Data Audit
        ↓
Dataset Split 정책 확정
        ↓
YOLO Dataset 구성
        ↓
작은 Smoke Test
        ↓
Baseline 학습
        ↓
Validation 평가
        ↓
실패 사례 분석
        ↓
한 가지 원인 개선
        ↓
Baseline vs Improved 비교
        ↓
Final Model 확정
        ↓
Frozen Test 최종평가
        ↓
이미지 Detection
        ↓
Auto Label 후보 생성
        ↓
교과 7 라벨링 도구로 Human Review
        ↓
Acceptance Test · 문서화
        ↓
[교과 14 제조 제품검사 프로젝트로 연결]
```

> **교과 7과 교과 8의 차이**
>
> 교과 7 = 정답 데이터를 만든다.  
> 교과 8 = 그 정답으로 모델을 학습하고, 평가하고, 실패 원인을 찾아 개선한다.

---

# 1. 교과 8에서 최종적으로 무엇을 만들어야 하나요?

교과 8은 `best.pt` 파일 하나만 만드는 프로젝트가 아닙니다.

교과 7에서 만든 정답 데이터를 이용하여  

```
Dataset 준비
        ↓
모델 학습
        ↓
성능 평가
        ↓
실패 원인 분석
        ↓
개선 실험
        ↓
Final Model 결정
        ↓
최종 Test
        ↓
Detection · Auto Label
        ↓
문서화 · 발표
```

까지 하나의 프로젝트로 완성합니다.

먼저 기억해야 할 것은 다음 두 가지입니다.

```text
산출물
→ 프로젝트가 끝났을 때 실제로 무엇이 남아 있어야 하는가?

완료 기준
→ 그 결과물이 어느 상태까지 완성되어야 하는가?
```

교과 8이 끝나면 다음 **8가지 결과물**이 있어야 합니다.

![[Group 36.png]]

| No. | 최종 산출물                        | 무엇을 만들어야 하나요?                                   |
| --: | ----------------------------- | ----------------------------------------------- |
|   1 | **학습용 Dataset Version**       | Train·Validation·Test로 확정한 Dataset과 Split 기록    |
|   2 | **Baseline Model**            | 개선 전 성능을 비교하기 위한 첫 기준 모델                        |
|   3 | **Failure Analysis**          | 모델이 무엇을 놓치고(FN), 무엇을 잘못 찾는지(FP) 분석한 결과          |
|   4 | **Improved / Final Model**    | 실패 원인을 바탕으로 한 가지를 개선하고 최종 선택한 모델                |
|   5 | **성능·실험 기록**                  | Precision·Recall·mAP, 학습조건, Baseline과 개선 결과 비교  |
|   6 | **Detection · Auto Label 결과** | 새 이미지 Detection 결과와 자동 라벨 후보 생성 결과              |
|   7 | **문서·교과 14 연결자료**             | README, QA, Test Report, Final Model의 성능과 한계 정리 |
|   8 | **최종 발표·시연**                  | 교과 7~8에서 무엇을 만들고 어떻게 개선했는지 설명하고 보여주는 결과         |

실제 파일로 생각하면 다음과 같습니다.

![[Pasted image 20260928193316.png]]

```
Dataset / Manifest
        +
baseline_best.pt
        +
failure_analysis.csv
        +
final_best.pt
        +
성능 · 실험 기록
        +
Prediction · Auto Label 결과
        +
README · Test Report
        +
최종 발표 · 시연
```

>**핵심**
>
>교과 8의 결과물은 모델 파일 하나가 아닙니다.
>어떤 데이터로 학습했고 → 모델이 어디에서 실패했고 → 무엇을 개선했고 → 최종 결과와 한계가 무엇인지까지 확인할 수 있어야 합니다.

---

# 2. 어디까지 해야 교과 8 프로젝트가 완료되나요?

앞에서는 **무엇을 만들어야 하는지** 확인했습니다.

이번에는 그 결과물이 **어느 상태까지 완성되어야 프로젝트가 끝난 것으로 보는지** 확인합니다.

단순히 파일이 만들어졌다고 완료되는 것은 아닙니다.

```
파일이 존재한다
        ↓
X

정상적으로 실행된다
+
결과를 확인했다
+
왜 그런 결과가 나왔는지 설명할 수 있다
        ↓
O
```

다음 조건을 모두 확인하면 교과 8 프로젝트를 완료한 것으로 봅니다.

![[Group 37.png]]

- 데이터 리키지(Data Leakage, 데이터 누수)는 AI 모델이 학습할 때 '시험 문제의 힌트나 정답이 실수로 유출되어 모델에 들어간 오류 상태'를 뜻합니다
  쉽게 말하면 데이터 리키지는 학생들이 모의고사(검증/테스트 데이터)를 보기 전에 교사가 실수로 시험 문제나 힌트가 적힌 유인물(학습 데이터)을 나눠준 상황과 같습니다.

| 단계                          | 완료 기준                                                                     |
| --------------------------- | ------------------------------------------------------------------------- |
| 1. Data                     | 교과 7 FINAL 데이터의 출처·Class·scene_type·QA 이력을 확인할 수 있다.                      |
| 2. Split                    | Train·Validation·Test가 확정되어 있고 Data Leakage를 점검했다.                        |
| 3. Smoke Test               | 작은 데이터로 학습 → Weight 생성 → 이미지 Prediction까지 정상 동작한다.                        |
| 4. Baseline                 | 첫 기준 모델을 만들고 Dataset·Epoch·Batch·Image Size 등의 학습조건을 기록했다.                |
| 5. Evaluation               | Precision·Recall·mAP·Class별 성능을 측정하고 의미를 설명할 수 있다.                        |
| 6. Failure Analysis       . | 실제 FP·FN·BBox·Class 오류를 확인하고 실패 원인 가설을 세웠다.                               |
| 7. Improvement              | 실패 원인을 근거로 주요 변수 하나를 바꾸어 실험했다.                                            |
| 8. Comparison               | Baseline과 Improved Model을 같은 Validation에서 비교했다.                           |
| 9. Final Model              | Validation 결과와 실패 사례를 근거로 Final Model을 선택했다.                              |
| 10. Frozen Test             | Final Model을 정한 뒤 Test에서 최종 성능을 측정했다.                                     |
| 11. 활용                      | Final Model로 이미지 Detection과 Auto Label이 동작하고 Human Review까지 연결된다.         |
| 12. 재현·문서화                  | Dataset Version·학습조건·실험결과를 다시 확인할 수 있고 README만 보고 다른 팀원이 핵심 기능을 실행할 수 있다. |
| 13. 발표·마무리                  | 프로젝트 과정·결과·한계를 발표하고 교과 14에서 이어갈 모델·성능·한계를 정리했다.                           |

---

## 2.1 그러면 성능은 몇 %가 나와야 하나요?

교과 8에서는 처음부터

```
mAP 90% 이상
Recall 85% 이상
```

처럼 **==`특정 숫자를 프로젝트 완료조건으로 정하지 않습니다.`==**

현재 Dataset의 Class 분포, 객체 크기, 장면 유형에 따라 실제 난이도가 다르기 때문입니다.

먼저 Baseline을 만들고 실제 성능을 확인한 뒤 개선 목표를 정합니다.

```
Baseline 학습
        ↓
Precision · Recall · mAP 측정
        ↓
FP · FN 확인
        ↓
가장 중요한 실패 문제 선택
        ↓
개선 목표 설정
        ↓
한 변수 개선 실험
        ↓
같은 Validation에서 비교
        ↓
Final Model 결정
        ↓
Frozen Test
        ↓
최종 성능과 한계 기록
```

예를 들어 다음처럼 목표를 정할 수 있습니다.

```
작은 이물질을 자주 놓친다.
→ FN을 줄여 보자.

정상 김치를 이물질로 잘못 찾는다.
→ FP를 줄여 보자.

특정 Class의 Recall이 낮다.
→ 해당 Class의 실패를 개선해 보자.

mAP는 좋아졌지만 Recall이 크게 떨어졌다.
→ 정말 좋은 개선인지 다시 확인하자.
```

따라서 교과 8에서는 다음처럼 생각합니다.

![[Pasted image 20260928194619.png]]

```
프로젝트 완료
≠
높은 mAP 숫자 하나


프로젝트 완료
=
Baseline 측정
+
성능지표 해석
+
FP / FN 분석
+
개선 목표 설정
+
한 변수 개선
+
Baseline과 비교
+
Final Model 결정
+
Frozen Test
+
실제 성능과 한계 설명
```

> **가장 중요한 완료 기준**
> 
> ==`“우리 모델의 mAP가 몇 %입니다.”`==에서 끝나는 것이 아닙니다.
> 
> “Baseline에서 이런 문제가 있었고 → 실제 실패 사례를 확인했고 → 이런 이유로 한 가지 조건을 바꾸었고 → 같은 Validation에서 결과가 이렇게 달라졌으며 → 최종 모델의 성능과 한계는 이렇습니다.”
> 
> 라고 팀이 설명할 수 있어야 합니다.

---
# 3. 교과 7에서 확정한 결과를 교과 8에서도 그대로 사용합니다

교과 8은 새로운 데이터로 처음부터 다시 시작하는 프로젝트가 아닙니다.

교과 7에서 같은 팀이 완성한 **검수된 데이터와 작업 기준을 그대로 이어서 사용**합니다.

교과 8을 시작하기 전에 먼저 다음 결과가 정상적으로 정리되어 있는지 확인합니다.

```text
JPG 900장
YOLO TXT 900개
Class 기준
BBox 기준
Dataset Manifest
QA Summary
source_dataset
original_split
scene_type
교과 7 최종 기록
```

이 중 Class ID, BBox 기준, 원본 데이터 출처와 같은 항목은  
교과 8에서 임의로 변경하지 않습니다.

```
교과 7
정답 데이터와 기준 확정
        ↓
교과 8
같은 기준으로 모델 학습·평가
```

> **핵심**
> 
> 교과 8에서는 교과 7의 데이터를 다시 라벨링하는 것이 아니라,  
> **교과 7에서 확정한 정답 데이터를 이용하여 AI 모델을 학습하는 단계로 넘어갑니다.**

---

## 2.1 Class 기준은 그대로 유지합니다

이번 프로젝트에서 사용하는 Class는 교과 7에서 확정한 기준과 같습니다.

|Class ID|이물 종류|교과 8 사용|
|---|---|---|
|0|나뭇잎·종이류|사용|
|1|플라스틱류·돌·금속 등|사용|
|2|나뭇가지류|사용|
|3|벌레류|사용|
|4|고무장갑|사용하지 않음 · 발견 시 기존 REVIEW 기록 확인|
|5|병해·갈변|사용|
|6|파·고추|사용|

예를 들어 Class 4를 사용하지 않는다고 해서

```
5 → 4
6 → 5
```

처럼 번호를 다시 정리하지 않습니다.

```
교과 7
Class ID 0~6
        ↓
교과 8
같은 Class ID 유지
```

Class 기준을 변경해야 하는 특별한 이유가 생기면 팀이 임의로 바꾸지 않고, 변경 이유와 
대응 관계를 별도로 기록한 뒤 Dataset Version을 새로 구분합니다.

> **Class 4가 실제 라벨에 0건인 경우**  
> `dataset.yaml`의 Class ID 체계는 그대로 유지합니다.  
> 정답 인스턴스가 0건이면 Class 4의 Recall·AP 같은 Class별 성능은 의미 있게 해석할 수 없으므로 보고서에 `미사용 Class / Instance 0건`으로 기록합니다.  
> 단, 모델이 Class 4를 예측했다면 해당 예측은 무시하지 말고 FP 또는 Class 오류 후보로 확인합니다.

---

## 2.2 이미지 유형 정보도 그대로 유지합니다

교과 7에서 기록한 `scene_type`도 교과 8에서 계속 사용합니다.

```
kimchi_with_target
→ 김치 + 검출 대상 객체

normal_kimchi
→ 정상 김치

object_only
→ 대상 객체 단독 이미지

other_review
→ 별도 확인이 필요한 이미지
```

이 정보는 모델이

```
김치가 있는 장면에서 잘하는가?
정상 김치에서 잘못 검출하지 않는가?
대상 객체 단독 이미지에서만 잘하는 것은 아닌가?
```

를 분석할 때 사용합니다.

따라서 교과 8에서도 `source_dataset`, `original_split`, `scene_type` 정보는 삭제하거나 
임의로 변경하지 않고 그대로 유지합니다.

---

# 4. 교과 8에서는 코딩을 어떻게 시작해야 하나요?

주니어는 보통 다음 지점에서 가장 많이 막힙니다.

```text
무슨 파일부터 만들어야 하지?
코드를 어디서 시작하지?
한 번에 어느 정도까지 만들어야 하지?
오류가 나면 무엇부터 봐야 하지?
학습 결과가 나오면 좋은 건지 어떻게 판단하지?
```

이번 프로젝트에서는 완성 코드를 한꺼번에 만들지 않습니다.

## 4.1 교과 8의 기본 코딩 패턴

모든 Python 파일을 다음 순서로 개발합니다.

![[Pasted image 20260928160026.png]]

```text
1. 이 파일이 해결할 문제를 한 문장으로 적기
        ↓
2. 입력이 무엇인지 정하기
        ↓
3. 출력이 무엇인지 정하기
        ↓
4. 정상 완료 조건 정하기
        ↓
5. 가장 작은 데이터로 구현
        ↓
6. 실행
        ↓
7. 출력 확인
        ↓
8. 오류가 있으면 현재 단계만 수정
        ↓
9. 통과하면 다음 기능 연결
        ↓
10. 실행조건과 결과 기록
```

# Python 파일을 만들기 전에 먼저 처리 과정을 생각합니다

프로젝트에서는 파일 이름을 정한 뒤 바로 코드를 작성하지 않습니다.

먼저 그 파일이 **무엇을 받아서 → 무엇을 처리하고 → 무엇을 만들어 내야 하는지**를 순서대로 생각합니다.

모든 파일에서 다음 사고방식을 반복합니다.

```
1. 이 파일은 왜 필요한가?
        ↓
2. 무엇을 입력받는가?
        ↓
3. 입력이 정상인지 무엇을 확인해야 하는가?
        ↓
4. 어떤 순서로 처리해야 하는가?
        ↓
5. 무엇을 결과로 만들어야 하는가?
        ↓
6. 성공했다는 것을 어떻게 확인하는가?
        ↓
7. 어떤 문제가 발생할 수 있는가?
        ↓
8. 처음에는 어떤 작은 데이터로 시험할 것인가?
        ↓
9. 작은 시험이 성공하면 무엇과 연결할 것인가?
        ↓
10. 어떤 실행조건과 결과를 기록할 것인가?
```

이 10가지는 특정 코드에만 사용하는 질문이 아닙니다.

교과 8에서 만드는 거의 모든 Python 파일에 같은 방식으로 적용할 수 있습니다.

---

## 예시 1 — `train_detector.py`를 만든다면

`train_detector.py`는 **이물질을 검출하는 모델을 학습시키는 기능**을 담당합니다.

코드를 작성하기 전에 다음과 같이 생각합니다.

```
1. 이 파일은 왜 필요한가?

교과 7에서 만든 이미지와 YOLO 정답 라벨을 이용하여
YOLO 객체검출 모델을 학습시키기 위해 필요하다.


2. 무엇을 입력받아야 하는가?

Dataset 정보
→ dataset.yaml

처음 사용할 모델
→ yolo11n.pt

학습 횟수
→ epochs

모델 입력 이미지 크기
→ imgsz

한 번에 학습할 이미지 수
→ batch

재현성을 위한 값
→ seed


3. 학습 전에 무엇을 확인해야 하는가?

dataset.yaml이 실제로 존재하는가?

Train / Validation 경로가 맞는가?

이미지와 TXT가 실제로 존재하는가?

Class 기준이 교과 7과 같은가?

GPU를 사용할 수 있는가?


4. 프로그램은 어떤 순서로 동작해야 하는가?

dataset.yaml 확인
        ↓
YOLO 사전학습 모델 Load
        ↓
학습조건 설정
        ↓
Train 데이터 학습
        ↓
Validation 평가
        ↓
가장 좋은 Weight 저장
        ↓
학습 로그 저장


5. 무엇이 결과로 나와야 하는가?

best.pt
last.pt
results.csv
학습 그래프
Validation 결과


6. 성공했다는 것은 어떻게 아는가?

학습이 마지막 Epoch까지 정상적으로 끝났다.

best.pt가 생성되었다.

Validation이 실행되었다.

results.csv가 생성되었다.

Loss가 NaN이 아니다.


7. 어떤 문제가 발생할 수 있는가?

Dataset 경로 오류

이미지와 라벨 대응 오류

CUDA를 찾지 못함

GPU 메모리 부족(OOM)

잘못된 Class ID

학습 중 NaN 발생

Weight가 생성되지 않음


8. 처음에는 어떻게 시험할 것인가?

처음부터 900장 전체로 학습하지 않는다.

Train 30~40장
+
Validation 5~10장
+
Epoch 1~3

으로 Smoke Test한다.


9. 작은 시험이 성공하면?

같은 train_detector() 기능을 사용하여

Smoke Test
        ↓
Baseline 학습
        ↓
Improved Model 학습

으로 확장한다.


10. 무엇을 기록해야 하는가?

Dataset Version
Model
Epoch
Image Size
Batch
Seed
GPU
실행시간
생성된 Weight
Validation 성능
```

즉 코드를 보기 전에 머릿속에는 이미 다음 흐름이 있어야 합니다.

```
Dataset
        ↓
Model
        ↓
Training Option
        ↓
Train
        ↓
Validation
        ↓
Weight
        ↓
학습 결과 기록
```

---

## 예시 2 — `dataset_audit.py`를 만든다면

이 파일은 모델을 학습시키는 파일이 아닙니다.

**학습 전에 데이터가 정상인지 자동으로 검사하는 파일**입니다.

```
1. 왜 필요한가?

잘못된 Dataset으로 몇 시간 동안 학습하는 것을 막기 위해 필요하다.


2. 무엇을 입력받는가?

이미지 폴더
YOLO TXT 폴더
교과 7 Dataset Manifest


3. 무엇을 먼저 확인하는가?

이미지 수

TXT 수

같은 이름의 JPG와 TXT가 존재하는지

TXT 형식이 올바른지

Class ID가 정상인지

BBox 좌표가 0~1 범위인지


4. 처리 순서는?

이미지 목록 읽기
        ↓
TXT 목록 읽기
        ↓
JPG ↔ TXT Pair 확인
        ↓
TXT 한 줄씩 읽기
        ↓
Class 검사
        ↓
BBox 좌표 검사
        ↓
Class별 개수 계산
        ↓
오류 목록 생성


5. 결과는?

dataset_summary.csv
audit_issue.csv


6. 성공 기준은?

전체 파일을 끝까지 검사한다.

오류 파일명이 기록된다.

Class별 BBox 수를 확인할 수 있다.

Dataset별 수량을 확인할 수 있다.


7. 발생 가능한 문제는?

TXT 없음

JPG 없음

TXT 줄 형식 오류

Class 범위 오류

BBox 좌표 오류

빈 TXT


8. 작은 시험은?

900장부터 검사하지 않는다.

이미지 5장
+
정상 TXT
+
일부러 오류를 만든 TXT

를 이용하여 검사 기능이 실제 오류를 찾는지 확인한다.


9. 성공하면?

900장 전체 Data Audit으로 확장한다.


10. 무엇을 기록하는가?

전체 이미지 수
전체 TXT 수
Pair Error
Empty Label 수
Class별 BBox 수
발견된 오류
```

여기서 주니어가 이해해야 할 핵심은

```
dataset_audit.py
=
"데이터를 학습하기 전에 건강검진하는 프로그램"
```

입니다.

---

## 예시 3 — `split_builder.py`를 만든다면

이 파일의 목적은 **900장의 데이터를 Train / Validation / Test로 나누는 것**입니다.

```
1. 왜 필요한가?

모델이 공부할 데이터와
개발 중 평가할 데이터와
마지막 시험 데이터를 구분하기 위해 필요하다.


2. 입력은?

교과 7 FINAL Dataset
Dataset Manifest
source_dataset
original_split
scene_type
capture_group


3. 먼저 무엇을 확인해야 하는가?

기존 validation 정보가 있는가?

같거나 매우 비슷한 촬영 이미지가 있는가?

Class 분포는 어떤가?

scene_type 분포는 어떤가?


4. 처리 순서는?

Manifest 읽기
        ↓
기존 source / split 확인
        ↓
유사 촬영 그룹 확인
        ↓
Split 정책 적용
        ↓
Train 결정
        ↓
Validation 결정
        ↓
Test 결정
        ↓
각 Split 분포 확인


5. 결과는?

train.csv
val.csv
test.csv
split_manifest.csv


6. 성공 기준은?

모든 이미지가 하나의 Split에만 속한다.

Train과 Test에 같은 이미지가 없다.

같은 촬영 그룹이 여러 Split에 섞이지 않는다.

Class와 scene_type 분포를 확인할 수 있다.


7. 발생 가능한 문제는?

같은 파일 중복

유사 이미지 누수

특정 Class가 Validation에 없음

Test가 너무 작음

특정 scene_type이 한쪽에만 몰림


8. 작은 시험은?

20~30장의 가상 Manifest로 먼저 Split을 실행한다.


9. 성공하면?

실제 900장 Manifest에 적용한다.


10. 무엇을 기록하는가?

Split 정책
Dataset Version
Train 수량
Validation 수량
Test 수량
Class 분포
scene_type 분포
```

중요한 것은 단순히

```
Python으로 파일을 복사한다
```

가 아닙니다.

주니어는 먼저

```
왜 이렇게 나누는가?
```

를 생각해야 합니다.

---

## 예시 4 — `evaluate_detector.py`를 만든다면

이 파일은 **모델을 학습하는 파일이 아니라 학습된 모델의 성능을 측정하는 파일**입니다.

```
1. 왜 필요한가?

"학습이 끝났다"가 아니라
"얼마나 잘 검출하는가"를 숫자로 확인하기 위해 필요하다.


2. 입력은?

학습된 best.pt

dataset.yaml

평가할 Split
→ val 또는 test

Image Size


3. 먼저 무엇을 확인하는가?

Weight가 존재하는가?

dataset.yaml이 맞는가?

지금 평가하려는 것이 Validation인가 Test인가?


4. 처리 순서는?

Weight Load
        ↓
평가 Dataset Load
        ↓
모델 Prediction
        ↓
Ground Truth와 비교
        ↓
Precision 계산
        ↓
Recall 계산
        ↓
mAP 계산
        ↓
Class별 결과 생성


5. 결과는?

Precision
Recall
mAP50
mAP50-95
Class별 성능
Confusion Matrix


6. 성공 기준은?

모든 Validation 데이터를 평가했다.

주요 지표가 정상적으로 출력되었다.

Class별 결과를 확인할 수 있다.


7. 어떤 문제가 발생할 수 있는가?

잘못된 Weight 사용

잘못된 Dataset 평가

Test를 너무 일찍 사용

Class Mapping 불일치


8. 작은 시험은?

Smoke Test Weight로 작은 Validation을 평가한다.


9. 성공하면?

Baseline 전체 Validation 평가로 확장한다.


10. 무엇을 기록하는가?

Model Version
Dataset Version
평가 Split
Precision
Recall
mAP
Class별 성능
```

주니어에게는 다음 질문을 항상 같이 하게 하면 좋습니다.

```
숫자가 나왔는가?
        ↓
X

그 숫자가 무엇을 의미하는가?
        ↓
O
```

---

## 예시 5 — `inference.py`를 만든다면

여기가 **실제로 새로운 이미지에서 이물질을 자동 검출하는 파일**입니다.

```
1. 왜 필요한가?

학습된 모델이 실제 새로운 이미지에서
이물질을 찾아낼 수 있는지 확인하기 위해 필요하다.


2. 입력은?

final_best.pt

새 이미지 또는 이미지 폴더

Confidence Threshold


3. 무엇을 확인해야 하는가?

Weight가 정상인가?

입력 이미지가 존재하는가?

Confidence 기준은 얼마인가?


4. 처리 순서는?

Final Model Load
        ↓
이미지 Load
        ↓
Model Inference
        ↓
BBox Prediction
        ↓
Class Prediction
        ↓
Confidence 확인
        ↓
화면 또는 파일에 결과 표시


5. 출력은?

BBox가 그려진 이미지

Class

Confidence

필요하면 검출 결과 CSV


6. 성공 기준은?

이미지를 정상적으로 읽는다.

검출 결과가 표시된다.

BBox가 이미지 범위를 벗어나지 않는다.

Class 이름이 정확하다.


7. 어떤 문제가 발생할 수 있는가?

Weight 경로 오류

이미지 읽기 실패

Confidence가 너무 낮아 FP 증가

Confidence가 너무 높아 FN 증가

Class 표시 오류


8. 작은 시험은?

Validation 이미지 1장으로 먼저 확인한다.


9. 성공하면?

이미지 1장
        ↓
여러 이미지
        ↓
이미지 폴더 전체

순서로 확장한다.


10. 무엇을 기록하는가?

사용한 Model
Confidence
입력 이미지
검출 결과
처리시간
```

따라서 구분은 이렇게 됩니다.

```
train_detector.py
→ AI를 공부시킨다

evaluate_detector.py
→ 공부한 AI를 시험한다

inference.py
→ 공부가 끝난 AI에게 새 이미지를 보여준다
```

---

## 예시 6 — `failure_analyzer.py`를 만든다면

이 파일은 **AI가 틀린 결과를 모아서 분석하기 위한 파일**입니다.

```
1. 왜 필요한가?

mAP가 낮다는 숫자만으로는
무엇을 고쳐야 하는지 알 수 없기 때문이다.


2. 입력은?

Ground Truth

모델 Prediction

Validation 결과

Dataset Manifest


3. 무엇을 찾아야 하는가?

FP
→ 없는데 있다고 검출

FN
→ 있는데 놓침

Class Error
→ 종류를 잘못 예측

BBox Error
→ 위치가 잘못됨


4. 처리 순서는?

Validation 결과 읽기
        ↓
오류가 큰 이미지 찾기
        ↓
FP / FN 구분
        ↓
Class 확인
        ↓
scene_type 확인
        ↓
반복되는 실패 패턴 찾기


5. 출력은?

failure_analysis.csv

대표 실패 이미지

실패 유형별 통계


6. 성공 기준은?

"성능이 낮다"가 아니라

"작은 플라스틱 객체의 FN이 반복된다"

처럼 구체적으로 설명할 수 있다.


7. 발생 가능한 문제는?

잘못된 Prediction과 GT 연결

같은 실패 중복 집계

단순히 Confidence만 보고 원인을 단정


8. 작은 시험은?

실패 이미지 5장만 먼저 분석한다.


9. 성공하면?

Validation 전체로 확장한다.


10. 무엇을 기록하는가?

실패 이미지
실패 유형
Class
scene_type
추정 원인
근거
다음 실험
```

이 파일의 최종 목적은 CSV를 만드는 것이 아니라

```
무엇을 개선해야 하는가?
```

를 찾아내는 것입니다.

---

## 예시 7 — `experiment_runner.py`를 만든다면

이 파일은 **실패분석 결과를 바탕으로 개선 실험을 반복 가능하게 실행하는 파일**입니다.

```
1. 왜 필요한가?

매번 학습 코드를 복사해서 조금씩 바꾸면
어떤 조건을 사용했는지 알기 어려워지기 때문이다.


2. 입력은?

Baseline 학습조건

변경할 변수 1개

Dataset Version


3. 먼저 무엇을 정하는가?

무엇을 바꿀 것인가?

왜 바꾸는가?

나머지 조건은 무엇을 그대로 둘 것인가?


4. 처리 순서는?

Baseline 조건 Load
        ↓
변수 하나 변경
        ↓
학습
        ↓
같은 Validation 평가
        ↓
Baseline과 비교
        ↓
결과 저장


5. 출력은?

Improved Weight

Validation Metrics

Experiment Log


6. 성공 기준은?

무엇을 바꿨는지 명확하다.

Baseline과 같은 기준에서 비교했다.

결과가 좋아지든 나빠지든 기록되었다.


7. 문제는?

여러 변수를 동시에 변경

Dataset Version 변경

Validation 변경

실패한 실험 삭제


8. 작은 시험은?

Epoch를 줄여 코드 흐름만 먼저 확인한다.


9. 성공하면?

정식 개선 실험을 실행한다.


10. 무엇을 기록하는가?

Experiment ID
변경 이유
변경 변수
Baseline 조건
새 조건
성능 변화
최종 판단
```

---

## 예시 8 — `auto_labeler.py`를 만든다면

이 파일은 **학습된 모델을 이용하여 새로운 이미지의 라벨 후보를 자동 생성하는 파일**입니다.

```
1. 왜 필요한가?

새 이미지마다 사람이 처음부터 BBox를 그리는 시간을 줄이기 위해 필요하다.


2. 입력은?

Final Model

라벨이 없는 새 이미지

Confidence Threshold


3. 무엇을 확인하는가?

이미지가 Train / Validation / Test에 사용되지 않은 새로운 이미지인가?

Final Model이 정상적으로 Load되는가?


4. 처리 순서는?

새 이미지 Load
        ↓
Final Model Prediction
        ↓
BBox 후보 생성
        ↓
Class 후보 생성
        ↓
YOLO 형식으로 변환
        ↓
후보 TXT 저장


5. 출력은?

자동 생성된 YOLO TXT 후보

Prediction 이미지


6. 성공 기준은?

이미지와 같은 이름의 TXT 후보가 생성된다.

BBox와 Class가 화면에 표시된다.

교과 7 라벨링 프로그램에서 다시 열 수 있다.


7. 발생 가능한 문제는?

잘못된 BBox 생성

FP를 라벨로 저장

FN 누락

Class 오류

Confidence 기준 오류


8. 작은 시험은?

새 이미지 5장으로 먼저 실행한다.


9. 성공하면?

새 이미지 전체에 후보 라벨을 생성한다.


10. 마지막 단계는?

자동 생성했다고 바로 정답으로 확정하지 않는다.

Auto Label
        ↓
교과 7 라벨링 GUI
        ↓
사람이 확인
        ↓
수정 / 삭제 / 추가
        ↓
승인
        ↓
Final Label
```

---

# 주니어가 모든 파일에 공통으로 적용할 사고 패턴

결국 파일 이름이 무엇이든 다음 방식으로 생각하면 됩니다.

```
① 왜 만드는가?
        ↓
② 무엇을 넣는가?
        ↓
③ 입력이 정상인지 어떻게 확인하는가?
        ↓
④ 어떤 순서로 처리하는가?
        ↓
⑤ 무엇이 나와야 하는가?
        ↓
⑥ 성공은 무엇인가?
        ↓
⑦ 실패하면 어떤 증상이 나타나는가?
        ↓
⑧ 작은 입력으로 어떻게 시험할 것인가?
        ↓
⑨ 성공하면 어디까지 확장할 것인가?
        ↓
⑩ 무엇을 기록해야 다시 실행할 수 있는가?
```

그리고 코드를 작성할 때는 이 사고과정을 다시 더 짧게 줄여서 기억합니다.

```
문제 정의
   ↓
입력
   ↓
처리
   ↓
출력
   ↓
성공 기준
   ↓
작은 테스트
   ↓
오류 수정
   ↓
확장
   ↓
기록
```



---

## 3.2 주니어를 위한 `PLAN → CODE → RUN → CHECK → RECORD` 규칙

```text
PLAN
무엇을 만들지 먼저 적는다
        ↓
CODE
한 기능만 구현한다
        ↓
RUN
작은 입력으로 직접 실행한다
        ↓
CHECK
예상한 결과와 같은지 확인한다
        ↓
RECORD
성공·실패·조건을 기록한다
```

이 규칙을 학습 코드에도 그대로 적용합니다.

```text
30~50장 Smoke Test
        ↓
성공
        ↓
전체 Train Baseline
        ↓
성공
        ↓
Validation 평가
        ↓
실패 원인 한 가지 선택
        ↓
한 변수만 변경
        ↓
재학습
```

---

## 3.3 오류가 발생했을 때의 디버깅 순서

오류가 발생했다고 프로그램 전체를 다시 만들지 않습니다.

```text
증상 기록
        ↓
같은 오류가 다시 발생하는지 재현
        ↓
어느 단계에서 처음 틀리는지 찾기
        ↓
입력 확인
        ↓
중간값 / 경로 / Shape / Class 확인
        ↓
원인 가설 1개 세우기
        ↓
한 가지 수정
        ↓
다시 실행
        ↓
해결 여부 기록
```

예:

```text
학습이 시작되지 않음
        ↓
모델부터 바꾸지 않음
        ↓
dataset.yaml 경로
        ↓
images / labels
        ↓
Class ID
        ↓
가상환경
        ↓
Ultralytics
        ↓
GPU
```

---

# 5. 왜 이렇게 작은 단계로 코딩하나요? — 주니어 학습 연구를 반영한 설계

프로그래밍 입문자가 어려워하는 것은 단순히 문법만이 아닙니다.

최근 고등교육 입문 프로그래밍 연구를 종합한 체계적 문헌고찰에서는 **프로그래밍 개념·문법/의미 오해, 문제해결 능력, 프로그램이 내부에서 어떻게 동작하는지에 대한 mental model 형성** 등이 주요 어려움으로 보고되었습니다.

또한 디버깅 교육에 대한 2024년 체계적 문헌고찰에서는 주니어의 복잡한 디버깅 활동을 지원하기 위해 **명시적인 절차, scaffolding, deliberate practice, visualization, metacognition** 등이 반복적으로 사용되었습니다.

주니어에게 완성 코드를 바로 주는 것만으로는 충분하지 않습니다.  
Parsons Problem을 활용한 연구에서는 코드 작성이 막힌 주니어에게 구조적 힌트를 제공하면 문제 해결 시간이 줄어드는 효과가 있었지만, 단순 지원만으로 자동적인 학습 향상이 보장되지는 않았습니다.

또 다른 Python 입문 연구에서는 **완성 예제를 처음에는 충분히 보여주되 점차 지원을 줄이고, 학습자가 “무엇을 알고 있는지·무엇이 막혔는지·다음 행동이 무엇인지”를 스스로 점검하게 하는 방식**이 문제해결과 자기조절에 효과적이었습니다.

따라서 이 가이드는 다음 방식으로 구성합니다.

```text
처음
→ 파일 역할·입력·출력·실행 예시를 많이 제공

중간
→ 코드 골격과 체크포인트를 제공

후반
→ 실패분석 결과를 보고 팀이 스스로 개선 실험 설계
```

> **이 프로젝트에서 AI에게 코드를 요청해도 괜찮습니다.**
>
> 다만 “전체 프로젝트를 만들어줘”라고 요청하지 않습니다.
>
> 다음처럼 작은 단위로 요청하고 반드시 실행 결과를 확인합니다.
>
> `이 함수의 입력과 출력은 무엇인가?`  
> `이 코드에서 Dataset path가 틀리면 어디서 실패하는가?`  
> `이 함수만 테스트할 수 있는 최소 예제를 만들어줘.`  
> `오류 메시지의 원인 후보를 우선순위대로 설명해줘.`

---

# 6. 교과 8에서 반드시 지킬 프로젝트 기준

![[Pasted image 20260928171201.png]]
## 6.1 Must Have

```text
교과 7 FINAL 확인
Dataset Audit
Split Freeze
YOLO Dataset 구성
Smoke Test
Baseline 학습
Validation 평가
Failure Analysis
한 변수 개선 실험
Baseline vs Improved 비교
Final Model
Frozen Test
이미지 Detection
Auto Label + Human Review
README + Acceptance Test
```

## 6.2 이번 교과에서 뒤로 미룰 것

다음 내용은 **교과 8의 필수 완료조건이 아닙니다.**  
이번 프로젝트에서는 Must Have Pipeline을 먼저 끝내고, 아래 항목은 구현하지 않아도 교과 8의 기본 프로젝트를 완료한 것으로 봅니다.

```text
여러 모델을 무작정 비교
대규모 Hyperparameter Search
Segmentation
복잡한 Ensemble
TensorRT 최적화
Jetson 배포
실시간 제조 판정 로직
FastAPI Dashboard
```

이 기술들이 불필요하다는 뜻은 아닙니다.  
장치 최적화·Jetson 배포·실시간 제조 판정처럼 실제 현장 시스템과 직접 연결되는 항목은 
교과 14 제조 제품검사 프로젝트에서 더 적절하게 확장합니다.

> **교과 8의 우선순위**  
> Dataset → 학습 → 평가 → 실패분석 → 한 변수 개선 → Final Model → 이미지 Detection → Auto Label → 문서화

---





---

# 7. 권장 개발 디렉토리

```text
subject8_kimchi_detection/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── configs/
│   ├── dataset.yaml
│   └── smoke_dataset.yaml
│
├── data/
│   ├── subject7_final/               # 교과 7 FINAL — 원본처럼 보호
│   │   ├── dataset1/
│   │   └── dataset2/
│   │
│   ├── yolo/                         # 교과 8에서 Freeze한 학습용 Dataset
│   │   ├── images/
│   │   │   ├── train/
│   │   │   ├── val/
│   │   │   └── test/
│   │   └── labels/
│   │       ├── train/
│   │       ├── val/
│   │       └── test/
│   │
│   ├── smoke/                        # 작은 End-to-End 확인용
│   └── new_unlabeled/                # Auto Label용 미사용 이미지
│
├── manifests/
│   ├── subject7_final_manifest.csv
│   ├── split_manifest.csv
│   ├── train.csv
│   ├── val.csv
│   └── test.csv
│
├── src/
│   ├── __init__.py
│   ├── dataset_audit.py
│   ├── split_builder.py
│   ├── label_visualizer.py
│   ├── train_detector.py
│   ├── evaluate_detector.py
│   ├── inference.py
│   ├── failure_analyzer.py
│   ├── experiment_runner.py
│   └── auto_labeler.py
│
├── scripts/
│   ├── 01_check_environment.py
│   ├── 02_audit_subject7_final.py
│   ├── 03_build_split.py
│   ├── 04_preview_dataset.py
│   ├── 05_smoke_train.py
│   ├── 06_train_baseline.py
│   ├── 07_evaluate_baseline.py
│   ├── 08_predict_images.py
│   ├── 09_collect_failures.py
│   ├── 10_train_experiment.py
│   ├── 11_evaluate_final_test.py
│   └── 12_auto_label.py
│
├── models/
│   ├── baseline/
│   ├── improved/
│   └── final/
│
├── runs/                              # Ultralytics 실행 결과
│
├── reports/
│   ├── dataset_summary.csv
│   ├── split_summary.csv
│   ├── smoke_test.md
│   ├── baseline_metrics.csv
│   ├── failure_analysis.csv
│   ├── experiment_log.csv
│   ├── final_metrics.csv
│   └── acceptance_test.md
│
├── demo/
│   ├── images/
│   └── auto_label/
│
└── docs/
    ├── project_baseline.md
    ├── data_contract.md
    ├── split_policy.md
    └── handoff_subject14.md
```

## 7.1 각 파일과 폴더의 역할

| 위치 | 역할                                            |
|---|---|
| `data/subject7_final/` | 교과 7에서 같은 팀이 완성한 검수 완료 데이터. 교과 8에서 직접 덮어쓰지 않음 |
| `data/yolo/` | 교과 8에서 Split을 Freeze한 실제 학습 구조                |
| `data/smoke/` | 작은 학습 Pipeline 확인용 데이터                        |
| `manifests/` | 어떤 파일이 어느 Split·Source·Scene에 속하는지 추적         |
| `configs/dataset.yaml` | YOLO가 Train·Val·Test와 Class 이름을 찾도록 연결        |
| `src/` | 여러 Script에서 재사용할 기능 함수                        |
| `scripts/` | 주니어가 실제 순서대로 실행하는 파일                          |
| `models/baseline/` | 첫 기준 모델                                       |
| `models/improved/` | 개선 실험 모델                                      |
| `models/final/` | 최종 확정 모델                                      |
| `runs/` | YOLO가 자동 생성하는 학습·평가 결과                        |
| `reports/` | 데이터·성능·실패·실험 결과                               |
| `demo/` | 발표·시연용 결과                                     |
| `docs/` | 팀 기준, 데이터 계약, Split 정책, 교과 14 인계              |

> `runs/`의 모든 Weight가 최종 모델은 아닙니다.  
> 설명 가능한 모델만 `models/`에 별도로 보관합니다.

---

# 8. Python 파일은 역할에 따라 나눕니다

프로젝트 코드를 모두 `main.py` 한 파일에 작성하지 않습니다.

```text
좋지 않은 예

main.py
├── 데이터 검사
├── Split
├── 학습
├── 평가
├── 추론
└── Auto Label
```

파일 하나가 너무 커지면 코드를 찾기 어렵고, 같은 기능을 여러 번 복사하게 됩니다.

이번 프로젝트에서는 Python 파일을 크게 두 종류로 나눕니다.

```
src/
→ 여러 곳에서 다시 사용할 "기능"을 작성하는 곳

scripts/
→ src의 기능을 불러와 실제 작업을 "실행"하는 곳
```

쉽게 말하면 다음과 같습니다.

```
src
= 기능을 만든다

scripts
= 그 기능을 실행한다
```

예를 들어 모델 학습은 다음처럼 나눕니다.

```
src/train_detector.py
→ YOLO 모델을 학습하는
  train_detector(...) 기능을 작성

        ↓ 불러와서 사용

scripts/06_train_baseline.py
→ Dataset, Epoch, Batch 등의 조건을 넣고
  Baseline 학습을 실제로 실행
```

이렇게 만들어 두면 이후 개선 실험에서도 같은 학습 기능을 다시 사용할 수 있습니다.

```
src/train_detector.py
        ↓
        ├── 06_train_baseline.py
        │      → Baseline 학습
        │
        └── 10_train_experiment.py
               → 개선 실험 학습
```

> **기억할 것**
> 
> `src` = 재사용할 기능  
> `scripts` = 실제 실행 파일

---

# 9. 프로젝트 공통 코딩 규칙

## 9.0 모든 Script는 프로젝트 루트에서 실행합니다

이 가이드의 상대경로는 모두 `subject8_kimchi_detection/`을 현재 작업 위치로 두는 것을 기준으로 작성합니다.

```bash
cd subject8_kimchi_detection
python scripts/01_check_environment.py
```

`configs/dataset.yaml`의 `path: data/yolo`도 이 실행 위치를 기준으로 사용합니다.

또한 `scripts/`에서 `src/`의 기능을 불러올 수 있도록 Script 시작부에서 프로젝트 루트를 Python import 경로에 추가합니다.

## 9.1 모든 Script의 시작 구조

```python
from pathlib import Path
import sys

PROJECT_ROOT = Path(__file__).resolve().parents[1]

if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

def main():
    # 1. 입력 경로 확인
    # 2. 기능 실행
    # 3. 결과 확인
    # 4. 결과 저장
    pass

if __name__ == "__main__":
    main()
```

Windows 환경에서 Ultralytics/PyTorch의 worker를 사용할 때도 `if __name__ == "__main__":` 구조를 사용하는 습관을 권장합니다.

## 9.2 경로를 문자열로 여기저기 하드코딩하지 않습니다

```python
# 좋지 않은 예
image = "F:/abc/project/data/...."

# 권장
PROJECT_ROOT = Path(__file__).resolve().parents[1]
DATA_DIR = PROJECT_ROOT / "data"
REPORT_DIR = PROJECT_ROOT / "reports"
```

## 9.3 실행 전에 입력이 존재하는지 확인합니다

```python
if not DATA_DIR.exists():
    raise FileNotFoundError(f"Dataset 경로를 찾을 수 없습니다: {DATA_DIR}")
```

## 9.4 결과는 화면에만 출력하지 않고 파일로 남깁니다

```text
Console
→ 지금 확인

CSV / MD / Weight
→ 나중에 재현·비교
```

---

# 10. 교과 8에서 사용하는 핵심 용어

| 용어           | 쉬운 설명                                 |
| ------------ | ------------------------------------- |
| Train        | AI가 실제로 공부하는 데이터                      |
| Validation   | 개발 중 어떤 모델이 더 좋은지 비교하는 데이터            |
| Test         | Final Model을 고른 뒤 마지막으로 평가하는 데이터      |
| Epoch        | Train 데이터 전체를 한 번 학습한 횟수              |
| Batch        | 한 번에 GPU로 처리하는 이미지 묶음                 |
| Image Size   | 모델에 넣을 때 사용하는 입력 크기                   |
| Weight       | 모델이 학습한 결과가 저장된 파일                    |
| Baseline     | 개선 전 첫 기준 모델                          |
| Precision    | AI가 찾았다고 한 것 중 맞은 정도                  |
| Recall       | 실제 객체를 얼마나 놓치지 않고 찾았는지                |
| mAP          | Class와 BBox를 함께 고려한 객체검출 대표 성능        |
| FP           | 실제로 없는데 있다고 잘못 검출                     |
| FN           | 실제로 있는데 놓침                            |
| Confidence   | 모델이 자신의 예측에 부여한 신뢰도                   |
| Data Leakage | 학습과 평가 데이터가 너무 비슷하거나 겹쳐 성능이 부풀려지는 문제  |
| Auto Label   | 모델이 BBox·Class 후보를 먼저 만들고 사람이 검수하는 방식 |

---

# 11. Dataset Split은 교과 7의 정보를 보존한 상태에서 결정합니다

교과 7에서 확인한 실제 원본 구조는 다음과 같습니다.

```text
Dataset 1
└── train

Dataset 2
├── train
└── validation
```

교과 7에서는 이 구조를 보존하여 넘겼습니다.

교과 8에서 **무조건 70:15:15로 다시 섞지 않습니다.**

## 11.1 권장 기본안

실제 수량과 분포를 확인한 뒤, 특별한 문제가 없다면 다음 순서를 권장합니다.

```text
기존 Dataset 2 / validation
→ Validation 후보로 우선 보존

기존 Dataset 1 / train
+
Dataset 2 / train
→ Train 후보 Pool

Train 후보 Pool 일부
→ 그룹 단위로 Frozen Test 생성

나머지
→ Train
```

왜 이미지 한 장씩 완전 무작위로 나누지 않을까요?

파일명이 시간 순서 또는 연속 촬영을 반영한다면 거의 같은 장면이 Train과 Test에 나뉠 수 있기 때문입니다.

따라서 가능하면 다음 메타데이터를 확인합니다.

```text
source_dataset
original_split
scene_type
capture_group 또는 촬영 묶음
```

## 11.2 기존 Validation이 너무 작거나 특정 장면에 치우친 경우

임의로 섞지 말고 다음을 기록합니다.

```text
문제 발견
        ↓
split_policy.md에 근거 작성
        ↓
팀 + 강사 합의
        ↓
새 Dataset Version 생성
        ↓
group 단위 재분할
        ↓
Class / scene_type 분포 다시 확인
        ↓
Split Freeze
```

> **Split을 바꾸면 Dataset Version도 바뀝니다.**

---

# 12. `dataset.yaml` 예시

교과 8에서는 이미 YOLO TXT가 존재하므로 JSON/XML 변환 단계가 없습니다.

```yaml
path: data/yolo

train: images/train
val: images/val
test: images/test

names:
  0: leaf_paper
  1: plastic_stone_metal
  2: branch
  3: insect
  4: rubber_glove_unused
  5: disease_browning
  6: green_onion_pepper
```

`path: data/yolo`는 **프로젝트 루트에서 Script를 실행하는 기준**입니다.  
`dataset.yaml` 파일이 `configs/` 안에 있다고 해서 `path`를 `../data/yolo`로 적는 방식으로 생각하지 않습니다. 실행 전에 실제 `data/yolo` 폴더가 존재하는지 확인합니다.

> 실제 프로젝트에서는 팀이 확정한 영문 이름을 사용하되,  
> `docs/data_contract.md`에 **한글 이름 ↔ 영문 이름 ↔ Class ID** 대응표를 함께 기록합니다.

---

# 13. 9일 전체 개발 일정

| 일차 | 핵심 목표 | 그날 반드시 남아야 하는 결과 |
|---|---|---|
| **1일차** | 교과 7 FINAL 확인 · 기준선 · 환경 · 작은 실행 확인 | Baseline 문서, 환경 확인, FINAL 확인 목록 |
| **2일차** | Dataset Audit · 분포·누수 위험 확인 · Split 정책 | Dataset Summary, Split Policy, QA 결과 |
| **3일차** | Split Freeze · YOLO 구조 · Preview · Smoke Test | train/val/test Manifest, dataset.yaml, Smoke Weight |
| **4일차** | 팀 공통 Baseline 학습 | baseline_best.pt, 학습조건, 로그 |
| **5일차** | Validation 평가 · 이미지 Detection | Baseline Metrics, 예측 이미지 |
| **6일차** | Failure Analysis | FP/FN 등 대표 실패 사례, 원인 가설 |
| **7일차** | 한 변수 개선 · 재학습 · 비교 | Experiment Log, Improved Model, 비교표 |
| **8일차** | Final Model · Frozen Test · Auto Label | final_best.pt, Test 성능, Auto Label 후보·검수 |
| **9일차** | Acceptance Test · 문서화 · 교과 14 Handoff | README, 최종보고서, 시연, Handoff |

> 앞 단계의 Gate를 통과하지 못하면 다음 단계로 넘어가지 않습니다.

---

# 14. 1일차 — FINAL 확인 · 기준선 · 환경 · 첫 실행 확인

![[Pasted image 20260928194906.png]]

## 14.0 오늘 해결할 질문

```text
교과 7에서 무엇을 받았는가?
        ↓
어떤 Class·Dataset 규칙을 그대로 이어가야 하는가?
        ↓
우리 팀의 개발 환경은 같은가?
        ↓
YOLO 사전학습 모델이 실제로 Load되는가?
```

## 14.1 교과 7 FINAL 결과를 확인합니다

확인:

- JPG 총 수
- YOLO TXT 총 수
- Dataset 1 / Dataset 2
- 기존 train / validation
- scene_type
- Class ID 0~6
- Class 4 처리 이력
- Empty Label 확인 결과
- 교과 7 QA Summary
- 남아 있는 주의사항

### 오늘 할 일은 다시 900장을 라벨링하는 것이 아닙니다

교과 7에서 전수검수한 데이터에 대해 **학습 관점의 계약과 구조를 확인**하는 것입니다.

---

## 14.2 `docs/project_baseline.md`를 만듭니다

최소 내용:

```text
Project Name
Dataset Version
Subject 7 Final 위치
Class ID / Name
원본 source_dataset
원본 split
Split 정책 결정 방법
Baseline Model
Baseline Image Size
Baseline Batch
Baseline Epoch
Seed
평가 지표
실험 규칙
Test 보호 규칙
팀 역할
완료조건
```

처음부터 모든 숫자를 확정하기 어렵다면 `TBD`로 표시하되 **누가 언제 결정하는지** 적습니다.

---

## 14.3 개발환경을 확인합니다

### `scripts/01_check_environment.py`

```python
import sys

import torch
import ultralytics

def main():
    print("Python:", sys.version.split()[0])
    print("PyTorch:", torch.__version__)
    print("Ultralytics:", ultralytics.__version__)
    print("CUDA available:", torch.cuda.is_available())

    if torch.cuda.is_available():
        print("GPU:", torch.cuda.get_device_name(0))

if __name__ == "__main__":
    main()
```

### 이 코드를 작성하기 전에 생각할 것

```text
입력
→ 현재 Python 환경

처리
→ 주요 버전과 CUDA 확인

출력
→ Console 환경정보

실패
→ 패키지 Import 실패 / CUDA 미인식
```

---

## 14.4 R-CNN 계열과 YOLO 계열을 짧게 비교한 뒤 YOLO 사전학습 모델 Load를 확인합니다

두 계열을 모두 구현하는 것이 이번 프로젝트의 목표는 아닙니다.  
먼저 객체검출 구조의 차이를 이해한 뒤, 실제 9일 프로젝트의 학습·평가·개선은 YOLO11로 진행합니다.

| 구분 | R-CNN 계열 | YOLO 계열 |
|---|---|---|
| 기본 생각 | 후보 영역을 찾고 단계적으로 분류·보정 | 한 번의 네트워크 흐름에서 위치와 Class를 함께 예측 |
| 특징 | 처리 단계가 상대적으로 복잡하며 구조 비교에 적합 | 빠른 검출과 일관된 학습·추론 Pipeline 구성에 적합 |
| 이번 프로젝트 | 개념 비교 | **실제 구현·학습에 사용** |

```text
R-CNN과 YOLO의 차이는 이해한다
        ↓
이번 9일 프로젝트 구현은 YOLO11에 집중한다
```

이제 YOLO 사전학습 모델을 정상적으로 Load할 수 있는지 확인합니다.

```python
from ultralytics import YOLO

def main():
    model = YOLO("yolo11n.pt")
    print(model.names)

if __name__ == "__main__":
    main()
```

처음부터 학습하지 않습니다.

오늘은 **모델 파일을 정상적으로 Load할 수 있는지**만 확인합니다.

---

## 14.5 README 뼈대를 만듭니다

README는 9일차에 기억을 더듬어 작성하지 않습니다.

1일차부터 다음 제목을 만들어 둡니다.

```text
프로젝트 목적
데이터 설명
Class
환경
실행방법
Dataset Split
Smoke Test
Baseline
평가
Failure Analysis
Improvement
Final Test
Auto Label
알려진 한계
교과 14 Handoff
```

---

## 14.6 1일차 기록

- `docs/project_baseline.md`
- 환경 버전
- GPU 확인 결과
- 교과 7 FINAL 확인 결과
- Issue Log
- README 초안

## 14.7 1일차 Daily Gate

- [ ] 교과 7 FINAL 데이터의 위치를 확인했습니다.
- [ ] JPG 900장 / YOLO TXT 900개라는 기준을 확인했습니다.
- [ ] Dataset 1 / 2와 원본 split 정보를 확인했습니다.
- [ ] Class ID 0~6을 그대로 사용합니다.
- [ ] `project_baseline.md` 초안을 만들었습니다.
- [ ] 모든 팀원이 같은 프로젝트 디렉토리를 사용합니다.
- [ ] Python / PyTorch / Ultralytics / GPU 상태를 확인했습니다.
- [ ] `YOLO("yolo11n.pt")` Load가 성공했습니다.
- [ ] README 초안을 만들었습니다.

---

# 15. 2일차 — Dataset Audit · 분포 확인 · Split 정책 확정

![[Pasted image 20260928195124.png]]
## 15.0 오늘의 목표

```text
"이 데이터를 어떤 방식으로 학습·평가에 나눌 것인가?"
```

를 데이터 근거로 결정합니다.

교과 7의 라벨 QA를 처음부터 반복하지 않습니다.

---

## 15.1 전체 Dataset을 자동 조사합니다

확인할 항목:

```text
전체 이미지 수
전체 TXT 수
Pair Error
Empty Label
Class별 BBox 수
이미지별 BBox 수
Dataset별 수량
original_split별 수량
scene_type별 수량
Class 4 존재 여부
좌표 범위 오류
```

### `src/dataset_audit.py`의 역할

```text
입력
→ 이미지 폴더 + 라벨 폴더 + 교과 7 Manifest

처리
→ Pair / YOLO 형식 / Class / 좌표 / 분포 확인

출력
→ dataset_summary.csv
→ audit_issue.csv
```

### YOLO 한 줄 검사 예시

```python
def validate_yolo_row(row: str) -> list[str]:
    issues = []
    parts = row.strip().split()

    if len(parts) != 5:
        return ["COLUMN_COUNT"]

    try:
        cls = int(parts[0])
        x, y, w, h = map(float, parts[1:])
    except ValueError:
        return ["PARSE_ERROR"]

    if cls < 0 or cls > 6:
        issues.append("CLASS_RANGE")

    if cls == 4:
        issues.append("CLASS4_REVIEW")

    if not (0.0 <= x <= 1.0 and 0.0 <= y <= 1.0):
        issues.append("CENTER_RANGE")

    if not (0.0 < w <= 1.0 and 0.0 < h <= 1.0):
        issues.append("SIZE_RANGE")

    return issues
```

> Empty TXT는 자동으로 오류라고 단정하지 않습니다.  
> 교과 7에서 정상 김치로 확인된 Negative Sample인지 FINAL 기록과 함께 확인합니다.

---

## 15.2 Scene Type과 Class 분포를 함께 봅니다

```text
Class 분포
→ 무엇을 학습해야 하는가?

scene_type 분포
→ 어떤 장면에서 학습하는가?
```

둘은 다른 정보입니다.

예:

```text
Class 2 나뭇가지가 충분한가?
+
나뭇가지 이미지가 object_only에만 몰려 있지 않은가?
+
kimchi_with_target에도 충분히 있는가?
```

이런 질문이 중요합니다.

---

## 15.3 Data Leakage 위험을 확인합니다

다음 항목을 확인합니다.

- 완전히 동일한 파일
- 파일명만 다른 동일 이미지
- 시간상 매우 가까운 연속 촬영 이미지
- 거의 같은 장면
- 같은 원본에서 파생된 변형 이미지

가능하면 `capture_group`을 Manifest에 추가합니다.

> 파일명의 시간 정보가 촬영 묶음을 의미하는지는 먼저 확인합니다.  
> 확인하지 않고 임의로 시간 문자열을 Session이라고 단정하지 않습니다.

---

## 15.4 Split 정책을 문서로 확정합니다

`docs/split_policy.md`

예:

```text
Validation
- 기존 Dataset 2 / validation을 우선 보존

Test
- 원본 train Pool에서 capture_group 단위로 별도 Holdout
- 개발 중 성능 개선에 사용하지 않음

Train
- 나머지 train Pool

공통
- source_dataset / original_split / scene_type 보존
- 동일 capture_group은 서로 다른 Split에 나누지 않음
- Class / scene_type 분포 확인
```

비율 자체보다 **누수를 막고 역할을 고정하는 것**이 더 중요합니다.

---

## 15.5 2일차 기록

- `reports/dataset_summary.csv`
- `reports/audit_issue.csv`
- `docs/data_contract.md`
- `docs/split_policy.md`
- `capture_group` 또는 그룹 정책
- Dataset Version

## 15.6 2일차 Daily Gate

- [ ] 전체 Pair 상태를 확인했습니다.
- [ ] Class별 BBox 수를 확인했습니다.
- [ ] scene_type 분포를 확인했습니다.
- [ ] Class 4 상태를 확인했습니다.
- [ ] Empty Label을 교과 7 FINAL 기록과 대조했습니다.
- [ ] 중복·유사 촬영 위험을 확인했습니다.
- [ ] Split 정책이 문서로 확정되었습니다.
- [ ] 아직 Test 이미지를 모델 개발에 사용하지 않았습니다.

---

# 16. 3일차 — Split Freeze · YOLO Dataset · Preview · Smoke Test

![[Pasted image 20260928195515.png]]

## 16.0 오늘의 목표

오늘은 처음으로

```text
Dataset
→ YOLO Load
→ 학습
→ Weight
→ Prediction
```

이 끝까지 연결되는지 확인합니다.

---

## 16.1 Split Manifest를 Freeze합니다

최소 컬럼:

```text
image_path
label_path
source_dataset
original_split
scene_type
capture_group
split
dataset_version
```

`split` 값:

```text
train
val
test
```

한 번 Freeze한 뒤 Baseline과 Improved 실험 중 임의로 바꾸지 않습니다.

---

## 16.2 학습용 폴더를 만듭니다

주니어에게는 실제 폴더 구조를 만드는 방식이 가장 이해하기 쉽습니다.

```text
data/yolo/
├── images/
│   ├── train/
│   ├── val/
│   └── test/
└── labels/
    ├── train/
    ├── val/
    └── test/
```

교과 7 FINAL은 그대로 보존하고, 교과 8 학습본을 별도로 구성합니다.

> 저장 공간이 부족한 환경에서는 이미지 경로 목록(`train.txt`, `val.txt`, `test.txt`) 방식을 사용할 수 있지만, 첫 프로젝트에서는 폴더 구조 방식이 더 이해하기 쉽습니다.

---

## 16.3 `dataset.yaml`을 연결합니다

`configs/dataset.yaml`

```yaml
path: data/yolo
train: images/train
val: images/val
test: images/test

names:
  0: leaf_paper
  1: plastic_stone_metal
  2: branch
  3: insect
  4: rubber_glove_unused
  5: disease_browning
  6: green_onion_pepper
```

경로가 실제 프로젝트 환경과 맞는지 확인합니다.

---

## 16.4 라벨을 다시 시각화합니다

교과 7에서 검수했지만, **Split 복사 과정에서 이미지와 TXT가 잘못 대응하지 않았는지** 확인합니다.

권장:

```text
Train 10장
Val   5장
Test  5장

+
각 Class
+
normal_kimchi
+
object_only
+
kimchi_with_target
```

BBox를 다시 그렸을 때 위치와 Class가 맞아야 합니다.

---

## 16.5 Smoke Test용 작은 Dataset을 만듭니다

Test는 사용하지 않습니다.

```text
Smoke Train
→ Train에서 30~40장

Smoke Val
→ Validation에서 5~10장

Smoke Test
→ 만들지 않음
```

작은 데이터도 실제 YOLO 폴더 구조를 그대로 사용합니다.

```text
data/smoke/
├── images/
│   ├── train/
│   └── val/
└── labels/
    ├── train/
    └── val/
```

그리고 `configs/smoke_dataset.yaml`을 만듭니다.

```yaml
path: data/smoke
train: images/train
val: images/val

names:
  0: leaf_paper
  1: plastic_stone_metal
  2: branch
  3: insect
  4: rubber_glove_unused
  5: disease_browning
  6: green_onion_pepper
```

Smoke Test에서도 교과 7에서 확정한 Class ID를 그대로 사용합니다.

성능을 높이는 것이 목적이 아닙니다.

```text
경로가 맞는가?
학습이 시작되는가?
Weight가 생성되는가?
Prediction이 되는가?
```

만 확인합니다.

---

## 16.6 `scripts/05_smoke_train.py`

```python
from pathlib import Path

from ultralytics import YOLO

PROJECT_ROOT = Path(__file__).resolve().parents[1]

def main():
    data_yaml = PROJECT_ROOT / "configs" / "smoke_dataset.yaml"

    model = YOLO("yolo11n.pt")

    model.train(
        data=str(data_yaml),
        epochs=3,
        imgsz=640,
        batch=8,
        seed=42,
        project=str(PROJECT_ROOT / "runs"),
        name="smoke_v1",
    )

if __name__ == "__main__":
    main()
```

### 실행 전 생각할 것

```text
입력
→ 작은 Train / Val Dataset

처리
→ 1~3 Epoch 학습

출력
→ Weight + 로그

성공
→ Weight 생성 + Prediction 가능
```

Batch `8`은 예시입니다. GPU 메모리가 부족하면 먼저 Batch를 줄입니다.

---

## 16.7 Smoke Weight로 이미지 1장을 Prediction합니다

```python
from ultralytics import YOLO

model = YOLO("runs/smoke_v1/weights/best.pt")
results = model.predict(
    source="path/to/sample.jpg",
    save=True,
    conf=0.25,
)
```

여기서는 정확도가 높지 않아도 됩니다.

---

## 16.8 3일차 Daily Gate

- [ ] Train / Val / Test Manifest가 Freeze되었습니다.
- [ ] Test를 Smoke Test에 사용하지 않았습니다.
- [ ] `dataset.yaml` 경로가 맞습니다.
- [ ] 이미지와 TXT가 Split 후에도 정확히 대응합니다.
- [ ] 1~3 Epoch 학습이 완료됩니다.
- [ ] `best.pt` 또는 `last.pt`가 생성됩니다.
- [ ] 이미지 Prediction이 실행됩니다.
- [ ] BBox · Class · Confidence가 표시됩니다.
- [ ] 오류와 해결방법을 `smoke_test.md`에 기록했습니다.

---

# 17. 4일차 — 팀 공통 Baseline 모델 학습

![[Pasted image 20260928195908.png]]

## 17.0 오늘의 목표

```text
개선 전에 비교할
첫 기준 모델 하나를 만든다.
```

팀원 6명이 서로 다른 모델을 무작정 돌리지 않습니다.

---

## 17.1 Baseline 조건을 Freeze합니다

예시:

```text
Model          : YOLO11n
Dataset        : kimchi_v1
Image Size     : 640
Batch          : 8
Epoch          : 50
Seed           : 42
Pretrained     : True
```

숫자는 예시입니다.

GPU, 수업시간, Smoke Test 결과를 보고 팀에서 확정합니다.

반드시 기록할 항목:

```text
Model
Dataset Version
Train / Val Manifest Version
Epoch
Batch
Image Size
Seed
Optimizer
Learning Rate
GPU
Ultralytics Version
학습 시작·종료 시간
```

---

## 17.2 재사용 가능한 학습 함수를 만듭니다

`src/train_detector.py`

```python
from ultralytics import YOLO

def train_detector(
    model_name: str,
    data_yaml: str,
    epochs: int,
    imgsz: int,
    batch: int,
    seed: int,
    project: str,
    name: str,
):
    model = YOLO(model_name)

    return model.train(
        data=data_yaml,
        epochs=epochs,
        imgsz=imgsz,
        batch=batch,
        seed=seed,
        project=project,
        name=name,
    )
```

### 왜 함수로 만드나요?

Day 7에서 개선 실험을 할 때 같은 학습 코드를 다시 사용하기 위해서입니다.

---

## 17.3 Baseline 실행 Script를 만듭니다

`scripts/06_train_baseline.py`

```python
from pathlib import Path
import sys

PROJECT_ROOT = Path(__file__).resolve().parents[1]

if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

from src.train_detector import train_detector

def main():
    train_detector(
        model_name="yolo11n.pt",
        data_yaml=str(PROJECT_ROOT / "configs" / "dataset.yaml"),
        epochs=50,          # 예시: 팀 Baseline 확정값 사용
        imgsz=640,
        batch=8,
        seed=42,
        project=str(PROJECT_ROOT / "runs"),
        name="baseline_v1",
    )

if __name__ == "__main__":
    main()
```

---

## 17.4 학습 중 무엇을 봐야 하나요?

주니어는 Loss가 내려가면 무조건 좋은 모델이라고 생각하기 쉽습니다.

오늘은 다음만 확인합니다.

```text
학습이 중단되지 않는가?
CUDA OOM이 없는가?
Loss가 NaN이 되지 않는가?
Validation이 매 Epoch 실행되는가?
best.pt가 생성되는가?
results.csv가 생성되는가?
```

성능 해석은 5일차에 합니다.

---

## 17.5 Baseline Weight를 따로 보관합니다

```text
runs/baseline_v1/weights/best.pt
        ↓
models/baseline/baseline_best.pt
```

파일만 복사하지 말고 어떤 조건의 Weight인지 함께 기록합니다.

---

## 17.6 4일차 Daily Gate

- [ ] 전체 Train Dataset으로 Baseline 학습이 완료되었습니다.
- [ ] Baseline 조건이 문서에 기록되어 있습니다.
- [ ] `baseline_best.pt`가 별도 보관되었습니다.
- [ ] 학습 로그와 `results.csv`를 찾을 수 있습니다.
- [ ] Dataset Version을 확인할 수 있습니다.
- [ ] Test Dataset은 사용하지 않았습니다.

---

# 18. 5일차 — Validation 평가 · 이미지 Detection

![[Pasted image 20260928200141.png]]

## 18.0 오늘의 목표

```text
"학습이 끝났다."
        ↓
"이 모델이 무엇을 잘하고 못하는지 설명할 수 있다."
```

로 넘어갑니다.

---

## 18.1 Validation을 실행합니다

`src/evaluate_detector.py`

```python
from ultralytics import YOLO

def evaluate_detector(
    model_path: str,
    data_yaml: str,
    split: str,
    imgsz: int,
):
    model = YOLO(model_path)

    return model.val(
        data=data_yaml,
        split=split,
        imgsz=imgsz,
        plots=True,
    )
```

`scripts/07_evaluate_baseline.py`

```python
from pathlib import Path
import sys

PROJECT_ROOT = Path(__file__).resolve().parents[1]

if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

from src.evaluate_detector import evaluate_detector

def main():
    metrics = evaluate_detector(
        model_path=str(PROJECT_ROOT / "models" / "baseline" / "baseline_best.pt"),
        data_yaml=str(PROJECT_ROOT / "configs" / "dataset.yaml"),
        split="val",
        imgsz=640,
    )

    print("Precision:", metrics.box.mp)
    print("Recall:", metrics.box.mr)
    print("mAP50:", metrics.box.map50)
    print("mAP50-95:", metrics.box.map)

if __name__ == "__main__":
    main()
```

---

## 18.2 지표를 쉬운 질문으로 바꿔 봅니다

```text
Precision
→ AI가 "이물이다"라고 한 결과를 얼마나 믿을 수 있는가?

Recall
→ 실제 이물을 얼마나 놓치지 않았는가?

mAP50
→ Class와 BBox까지 포함해 전체적으로 얼마나 잘 검출하는가?

Class별 성능
→ 어떤 이물 종류에서 특히 약한가?
```

제조 이물검출에서는 전체 mAP만 보지 않고 **FN과 Recall**도 반드시 봅니다.

---

## 18.3 이미지 Prediction을 확인합니다

```python
from ultralytics import YOLO

model = YOLO("models/baseline/baseline_best.pt")

model.predict(
    source="data/yolo/images/val",
    conf=0.25,
    save=True,
    project="runs/predict",
    name="baseline_val_images",
)
```

최소 다음 유형을 직접 봅니다.

- 김치 + 대상 객체
- 정상 김치
- 대상 객체 단독
- 작은 객체
- BBox 여러 개
- 모델이 틀린 것으로 보이는 이미지

---

## 18.4 5일차 Daily Gate

- [ ] Validation Precision을 기록했습니다.
- [ ] Validation Recall을 기록했습니다.
- [ ] mAP50과 mAP50-95를 기록했습니다.
- [ ] Class별 성능을 확인했습니다.
- [ ] 실제 Prediction 이미지를 확인했습니다.
- [ ] 정상 김치에서 FP가 발생하는지 확인했습니다.
- [ ] 작은 객체에서 FN이 발생하는지 확인했습니다.
- [ ] 실패 의심 이미지를 Day 6용으로 모았습니다.
- [ ] Test Dataset은 아직 모델 선택에 사용하지 않았습니다.

---

# 19. 6일차 — Failure Analysis

![[Pasted image 20260928200458.png]]
## 19.0 오늘의 목표

오늘은 성능을 올리는 날이 아닙니다.

```text
왜 틀렸는가?
```

를 찾는 날입니다.

---

## 19.1 실패 유형

```text
FP
→ 없는데 있다고 검출

FN
→ 있는데 놓침

BBox Error
→ 객체는 찾았지만 위치·크기가 많이 틀림

Class Error
→ 다른 Class로 예측

Small Object
→ 작은 객체를 반복적으로 놓침

Scene Error
→ 특정 scene_type에서 반복적으로 실패
```

---

## 19.2 이미지별 실패 후보를 CSV로 뽑습니다

현재 Ultralytics Validation 결과에서는 이미지별 TP·FP·FN 등의 정보를 확인할 수 있습니다.

```python
import pandas as pd
from ultralytics import YOLO

model = YOLO("models/baseline/baseline_best.pt")

metrics = model.val(
    data="configs/dataset.yaml",
    split="val",
    imgsz=640,
)

rows = []

for image_name, values in metrics.box.image_metrics.items():
    rows.append({
        "image": image_name,
        "precision": values["precision"],
        "recall": values["recall"],
        "f1": values["f1"],
        "tp": values["tp"],
        "fp": values["fp"],
        "fn": values["fn"],
    })

df = pd.DataFrame(rows)
df = df.sort_values(["fn", "fp"], ascending=False)
df.to_csv("reports/per_image_metrics.csv", index=False)
```

이 CSV에서 `fn`이나 `fp`가 큰 이미지를 우선적으로 직접 확인합니다.

---

## 19.3 실패 원인을 네 범주로 먼저 생각합니다

```text
Data 문제
→ 특정 Class가 너무 적음
→ 특정 scene_type에 편중
→ 작은 객체 부족

Label 문제
→ GT Class 오류
→ BBox 품질 오류

Model / Training 문제
→ 입력 크기
→ 학습량
→ 모델 크기

Inference 문제
→ Confidence Threshold
→ 후처리 조건
```

---

## 19.4 `failure_analysis.csv`

권장 컬럼:

```text
image
source_dataset
scene_type
failure_type
ground_truth
prediction
confidence
suspected_cause
evidence
next_action
```

예:

| image | failure_type | GT | Prediction | 추정 원인 | 다음 행동 |
|---|---|---|---|---|---|
| img_001.jpg | FN | plastic | none | 매우 작은 객체 | imgsz 실험 검토 |
| img_015.jpg | FP | none | leaf | 김치 배경과 혼동 | normal_kimchi FP 조사 |
| img_031.jpg | Class | branch | leaf | Class 시각 특징 유사 | Class별 데이터 분포 확인 |

---

## 19.5 6일차 Daily Gate

- [ ] 대표 실패 사례 10~20건을 확보했습니다.
- [ ] 실제 발생한 FP와 FN을 확인했습니다.
- [ ] scene_type별 실패 차이를 확인했습니다.
- [ ] 각 실패에 원인 가설을 작성했습니다.
- [ ] “성능이 낮다”가 아니라 구체적인 문제 한 가지를 선택했습니다.
- [ ] 7일차에서 바꿀 변수 하나를 결정했습니다.

---

# 20. 7일차 — 한 변수 개선 실험과 재학습

![[Pasted image 20260928200901.png]]
## 20.0 오늘의 목표

```text
무엇을 바꿨는가?
왜 바꿨는가?
같은 기준에서 실제로 좋아졌는가?
```

를 증명합니다.

---

## 20.1 개선 실험은 실패분석에서 시작합니다

예:

```text
관찰
작은 플라스틱 FN이 많다
        ↓
가설
640 입력에서는 작은 객체 특징이 부족할 수 있다
        ↓
실험
imgsz 640 → 960
        ↓
나머지 주요 조건은 동일
        ↓
같은 Validation 평가
```

`960`은 예시입니다. GPU 메모리와 Smoke Test 결과에 따라 다른 값을 선택할 수 있습니다.

---

## 20.2 한 번에 주요 변수 하나만 바꿉니다

좋은 실험:

```text
Baseline
imgsz = 640
        ↓
EXP001
imgsz = 960
```

좋지 않은 실험:

```text
Model 변경
+
imgsz 변경
+
Epoch 변경
+
Augmentation 변경
+
Dataset 변경
```

여러 조건을 한 번에 바꾸면 무엇 때문에 결과가 달라졌는지 설명하기 어렵습니다.

---

## 20.3 `experiment_log.csv`

권장 필드:

```text
experiment_id
date
dataset_version
model
imgsz
epochs
batch
seed
changed_variable
reason
validation_precision
validation_recall
validation_map50
validation_map50_95
decision
```

실패한 실험도 삭제하지 않습니다.

---

## 20.4 Dataset Version이 바뀌었다면 비교 기준도 다시 맞춥니다

예:

```text
Validation GT 오류 발견
        ↓
kimchi_v1 → kimchi_v2
        ↓
Baseline도 v2 Validation에서 다시 평가
        ↓
Improved도 v2 Validation에서 평가
        ↓
같은 Dataset Version에서 비교
```

---

## 20.5 7일차 Daily Gate

- [ ] 실패분석과 연결된 개선 가설이 있습니다.
- [ ] 주요 변수 하나만 바꿨습니다.
- [ ] Baseline과 동일한 Validation을 사용했습니다.
- [ ] Dataset Version이 동일합니다.
- [ ] 성능이 좋아졌든 나빠졌든 결과를 기록했습니다.
- [ ] 지표뿐 아니라 실패 이미지가 실제로 줄었는지 확인했습니다.
- [ ] Final Model 후보를 정했습니다.

---

# 21. 8일차 — Final Model · Frozen Test · Auto Label

![[Pasted image 20260928201126.png]]
## 21.0 오늘의 목표

Validation을 이용한 개발을 끝내고,

```text
Final Model 확정
        ↓
Frozen Test 최종평가
        ↓
Auto Label
```

까지 연결합니다.

---

## 21.1 Test를 보기 전에 Final Model을 선택합니다

```text
Baseline
+
Improved
        ↓
같은 Validation에서 비교
        ↓
지표 + 실패 사례
        ↓
Final Model 선택
```

선택한 Weight:

```text
models/final/final_best.pt
```

Final Model 선정 이유도 기록합니다.

---

## 21.2 Frozen Test 최종평가

`scripts/11_evaluate_final_test.py`

```python
from pathlib import Path

from ultralytics import YOLO

PROJECT_ROOT = Path(__file__).resolve().parents[1]

def main():
    model = YOLO(
        str(PROJECT_ROOT / "models" / "final" / "final_best.pt")
    )

    metrics = model.val(
        data=str(PROJECT_ROOT / "configs" / "dataset.yaml"),
        split="test",
        imgsz=640,   # Final Model 평가조건과 일치하도록 팀 기준 사용
        plots=True,
    )

    print("Test Precision:", metrics.box.mp)
    print("Test Recall:", metrics.box.mr)
    print("Test mAP50:", metrics.box.map50)
    print("Test mAP50-95:", metrics.box.map)

if __name__ == "__main__":
    main()
```

Test 성능이 낮다고 같은 Test를 보면서 반복 수정하지 않습니다.

낮은 Test 결과도 중요한 프로젝트 결과입니다.

---

## 21.3 Auto Label은 교과 7과 다시 연결합니다

가장 좋은 방법은 **학습·Validation·Test에 사용하지 않은 새 이미지**로 Auto Label을 확인하는 것입니다.

```text
새 이미지
        ↓
Final Model
        ↓
BBox + Class 후보 생성
        ↓
YOLO TXT 후보 저장
        ↓
교과 7 라벨링 GUI에서 Load
        ↓
사람이 확인
        ↓
수정 / 삭제 / 승인
        ↓
최종 라벨 저장
```

별도의 새 이미지가 제공되지 않았다면, **Frozen Test 최종평가를 모두 끝낸 뒤** Test 이미지 5~10장을 복사하고 정답 TXT는 별도로 보관한 상태에서 Auto Label 기능만 시연할 수 있습니다.

이 경우 Auto Label 결과를 이용하여 Final Model을 다시 학습하거나 Test 성능을 다시 맞추지 않습니다.  
목적은 **자동 라벨 후보 생성 → 사람 검수** 흐름이 실제로 연결되는지 확인하는 것입니다.

이 연결이 매우 중요합니다.

교과 7에서는 사람이 처음부터 검수·보정했고,  
교과 8에서는 AI가 먼저 후보를 만들고 사람이 검수합니다.

---

## 21.4 Auto Label 후보 생성 예시

```python
from pathlib import Path

from ultralytics import YOLO

PROJECT_ROOT = Path(__file__).resolve().parents[1]

def main():
    model = YOLO(
        str(PROJECT_ROOT / "models" / "final" / "final_best.pt")
    )

    model.predict(
        source=str(PROJECT_ROOT / "data" / "new_unlabeled"),
        conf=0.25,
        save=True,
        save_txt=True,
        save_conf=False,
        project=str(PROJECT_ROOT / "runs"),
        name="auto_label_candidates",
    )

if __name__ == "__main__":
    main()
```

`save_conf=False`로 저장한 후보 TXT는 사람이 검수한 뒤에만 정답 라벨로 승인합니다.

---

## 21.5 Auto Label에서 기록할 것

```text
전체 새 이미지 수
AI 후보 BBox 수
그대로 승인한 수
Class 수정 수
BBox 수정 수
삭제 수
누락 추가 수
REVIEW 수
```

이 기록은 교과 7과 비교했을 때 AI가 라벨링 작업을 얼마나 도왔는지 설명하는 자료가 됩니다.

---

## 21.6 8일차 Daily Gate

- [ ] Test를 보기 전에 Final Model을 확정했습니다.
- [ ] `final_best.pt`가 별도 보관되어 있습니다.
- [ ] Frozen Test에서 Precision·Recall·mAP를 기록했습니다.
- [ ] Test 결과를 보면서 같은 Test에 맞춘 반복 수정을 하지 않았습니다.
- [ ] 새 이미지 또는 최종평가가 끝난 데모용 이미지에서 Auto Label 후보를 만들었습니다.
- [ ] 후보 YOLO TXT를 교과 7 도구 또는 검수 UI로 확인했습니다.
- [ ] 사람이 수정·삭제·승인한 결과를 저장했습니다.

---

# 22. 9일차 — Final Gate · Acceptance Test · 발표 · 프로젝트 종료

![[Pasted image 20260928201400.png]]

9일차에는 새로운 모델이나 기능을 추가하지 않습니다.

오늘은 지금까지 만든 결과를 다시 확인하고,  
프로젝트를 다른 사람이 실행할 수 있는 상태로 정리한 뒤  
팀 발표와 시연을 마지막으로 교과 8 프로젝트를 종료합니다.

```text
Feature Freeze
        ↓
보안 · Git 최종 확인
        ↓
최종 산출물 확인
        ↓
Acceptance Test
        ↓
Final Gate 자가점검
        ↓
README · 보고서 · 제출본 확정
        ↓
발표 자료 준비
        ↓
최종 발표
        ↓
Detection · Auto Label 시연
        ↓
질의응답
        ↓
[교과 8 프로젝트 종료]
```

> **9일차 핵심**
> 
> 오늘은 더 좋은 모델을 만들기 위한 날이 아닙니다.
> 
> 지금까지 만든 결과를 안전하게 정리하고, 다시 실행할 수 있는지 확인하고,  
> 우리가 무엇을 만들었는지 설명한 뒤 프로젝트를 종료하는 날입니다.

## 22.1 가장 먼저 Feature Freeze합니다

9일차부터는 특별한 오류가 발견되지 않는 한 새로운 기능을 추가하지 않습니다.

```
새로운 Model 추가       X
새로운 Augmentation    X
새로운 기능 추가        X
Dataset 재구성          X
새로운 실험 시작        X
```

오늘 허용되는 작업은 다음과 같습니다.

```
실행 오류 수정
문서 오탈자 수정
잘못된 경로 수정
제출 파일 누락 보완
발표 자료 정리
시연 준비
```

이 시점의 Dataset과 Final Model을 최종 버전으로 확정합니다.

예:

```
Dataset Version
→ kimchi_v2_final

Final Model
→ final_best.pt

Project Version
→ subject8_final_v1
```

---

## 22.2 데이터 보안과 Git 상태를 최종 확인합니다

이번 프로젝트 데이터는 NDA가 적용되는 교육용 기업 데이터입니다.

따라서 프로젝트 종료 전에  
**외부로 나가면 안 되는 파일이 Git에 포함되어 있지 않은지 반드시 확인합니다.**

## 외부 GitHub에 올리지 않습니다

다음 파일은 공개 GitHub에 업로드하지 않습니다.

```
기업 제공 JPG

기업 제공 YOLO TXT

교과 7 FINAL Dataset

교과 8 Train / Validation / Test Dataset

새 이미지 데이터

학습 Weight

ONNX / TensorRT 모델

Ultralytics runs 결과 중
기업 이미지가 포함된 파일

Prediction 원본 이미지

Auto Label 대상 이미지

NDA 또는 기관 반출 금지 문서
```

기관에서 내부 GitLab 또는 Gitea를 제공한다면  
기관 보안정책에 따라 해당 저장소를 사용합니다.

> `.gitignore`를 작성했다고 해서 외부 GitHub 사용이 허용되는 것은 아닙니다.
> 
> **보안정책이 항상 `.gitignore`보다 우선합니다.**

---

## 22.2.1 프로젝트 루트에 `.gitignore`를 만듭니다

예:

```
# ==================================================
# 교과 7 / 8 기업 제공 Dataset
# ==================================================

data/subject7_final/
data/yolo/
data/smoke/
data/new_unlabeled/

# 이미지 원본
*.jpg
*.jpeg
*.png
*.bmp
*.tif
*.tiff

# ==================================================
# Model Weight / Export Model
# ==================================================

models/
runs/

*.pt
*.pth
*.onnx
*.engine

# ==================================================
# Auto Label / Prediction 결과
# 기업 데이터가 포함될 수 있으므로 제외
# ==================================================

demo/prediction_images/
demo/auto_label/

# ==================================================
# Python 환경
# ==================================================

.venv/
venv/
env/

__pycache__/
*.pyc
*.pyo

# ==================================================
# IDE / OS
# ==================================================

.vscode/
.idea/

.DS_Store
Thumbs.db

# ==================================================
# 임시파일
# ==================================================

*.tmp
*.log
```

단, 프로젝트 상황에 따라 Git으로 관리해야 하는 작은 비민감 예제 파일이 있다면  
팀과 강사가 확인한 뒤 예외처리합니다.

---

## 22.2.2 Git에 남겨도 되는 파일을 확인합니다

기관 내부 Git에서는 일반적으로 다음과 같은 파일을 관리합니다.

```
Python 소스코드

configs/

README.md

requirements.txt

.gitignore

docs/

Dataset 경로정보만 포함한 Manifest

민감 이미지가 포함되지 않은 CSV 통계

실험 조건

성능 수치

Issue 기록
```

단, CSV나 Markdown 안에 실제 이미지가 포함되어 있거나  
반출 금지 경로·개인정보·민감정보가 포함되어 있다면 그대로 올리지 않습니다.

---

## 22.2.3 Git 최종 확인

Commit 전에 반드시 확인합니다.

```bash
git status
```

확인할 것:

```
기업 Dataset이 보이지 않는가?

*.pt가 보이지 않는가?

runs/가 보이지 않는가?

Prediction 이미지가 보이지 않는가?

Auto Label 이미지가 보이지 않는가?

가상환경이 보이지 않는가?
```

문제가 있다면 Commit 전에 `.gitignore`를 수정합니다.

---

## 22.3 최종 산출물 8가지를 확인합니다

교과 8에서는 다음 결과가 모두 존재해야 합니다.

```
① 학습용 Dataset Version

② Baseline Model

③ Failure Analysis

④ Improved / Final Model

⑤ 성능 · 실험 기록

⑥ Detection · Auto Label 결과

⑦ README · QA · Test Report · 교과 14 연결자료

⑧ 최종 발표 · 시연
```

파일 하나가 있다는 것만 확인하지 않습니다.

**다른 사람이 그 결과가 무엇인지 이해할 수 있는 상태인지**도 확인합니다.

---

## 22.4 최종 제출 구조를 확인합니다

최종 제출 폴더는 다음처럼 정리합니다.

```
final_submission/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── configs/
│   └── dataset.yaml
│
├── docs/
│   ├── project_baseline.md
│   ├── data_contract.md
│   ├── split_policy.md
│   └── handoff_subject14.md
│
├── manifests/
│   ├── split_manifest.csv
│   ├── train.csv
│   ├── val.csv
│   └── test.csv
│
├── models/
│   ├── baseline_best.pt
│   └── final_best.pt
│
├── reports/
│   ├── dataset_summary.csv
│   ├── smoke_test.md
│   ├── baseline_metrics.csv
│   ├── failure_analysis.csv
│   ├── experiment_log.csv
│   ├── final_metrics.csv
│   └── acceptance_test.md
│
├── demo/
│   ├── prediction_images/
│   └── auto_label/
│
└── presentation/
    └── final_presentation.pdf
```

> 위 구조는 **내부 제출본 기준**입니다.
> 
> 기업 Dataset과 Weight를 외부 Git에 업로드한다는 뜻이 아닙니다.

---

## 22.5 README를 최종 확인합니다

README는 프로젝트를 처음 보는 사람이  
“무엇을 만들었고 어떻게 실행하는가?”를 이해할 수 있어야 합니다.

최소 내용:

```
프로젝트 목적

교과 7 → 교과 8 연결

사용 Dataset 설명

Class ID / Name

scene_type

Train / Validation / Test 구성

Data Leakage 방지 방법

개발환경

디렉토리 구조

설치방법

Smoke Test

Baseline 학습방법

Validation 방법

Failure Analysis

Improvement Experiment

Final Model

Frozen Test 결과

이미지 Detection

Auto Label

Human Review

현재 모델의 한계

교과 14에서 이어갈 내용

데이터 보안 주의사항
```

README에 기업 원본 이미지나 실제 Dataset을 직접 포함하지 않습니다.

---

## 22.6 다른 팀원이 README만 보고 다시 실행합니다

가능하면 해당 기능을 만든 사람이 아닌 다른 팀원이 실행합니다.

```
환경 확인
        ↓
Dataset 경로 확인
        ↓
Final Model Load
        ↓
이미지 Prediction
        ↓
Auto Label 후보 생성
        ↓
Human Review
```

새 PC가 없다면 다른 팀원의 가상환경에서 실행합니다.

여기서 중요한 것은

```
"내 컴퓨터에서만 된다."
```

가 아니라

```
"문서를 보고 다른 팀원도 실행할 수 있다."
```

입니다.

---

## 22.7 Acceptance Test를 수행합니다

### Dataset

- [ ] 교과 7 FINAL 데이터의 출처를 확인할 수 있습니다.
- [ ] Dataset Version을 확인할 수 있습니다.
- [ ] `source_dataset / original_split / scene_type`을 추적할 수 있습니다.
- [ ] Train / Validation / Test Manifest가 존재합니다.
- [ ] Split 정책을 설명할 수 있습니다.
- [ ] Test가 개발 중 모델 선택에 사용되지 않았습니다.

### Model

- [ ] Baseline Model과 Final Model을 구분할 수 있습니다.
- [ ] Baseline 학습조건이 기록되어 있습니다.
- [ ] `baseline_best.pt`가 있습니다.
- [ ] `final_best.pt`가 있습니다.
- [ ] Final Model을 선택한 이유가 기록되어 있습니다.

### Evaluation

- [ ] Baseline Validation 결과가 있습니다.
- [ ] Precision을 확인할 수 있습니다.
- [ ] Recall을 확인할 수 있습니다.
- [ ] mAP50을 확인할 수 있습니다.
- [ ] mAP50-95를 확인할 수 있습니다.
- [ ] Failure Analysis가 있습니다.
- [ ] Baseline vs Improved 비교 결과가 있습니다.
- [ ] Frozen Test 결과가 있습니다.

### Inference

- [ ] 이미지 Detection이 실행됩니다.
- [ ] BBox가 표시됩니다.
- [ ] Class가 표시됩니다.
- [ ] Confidence가 표시됩니다.

### Auto Label

- [ ] 새 이미지에서 후보 라벨을 만들 수 있습니다.
- [ ] YOLO TXT 후보가 저장됩니다.
- [ ] 교과 7 라벨링 도구에서 다시 열 수 있습니다.
- [ ] 사람이 수정·삭제·추가할 수 있습니다.
- [ ] 승인된 라벨을 저장할 수 있습니다.

---

## 22.8 발표 전에 Final Gate 자가점검을 합니다

이 체크리스트는 발표 후에 확인하는 것이 아닙니다.

**발표 전에 “우리 프로젝트가 실제로 끝났는가?”를 팀이 마지막으로 확인하는 단계입니다.**

### 교과 7 연결

- [ ] 교과 7 FINAL 데이터를 사용했습니다.
- [ ] JPG 900장 + YOLO TXT 900개 기준을 확인했습니다.
- [ ] Class ID 0~6 기준을 유지했습니다.
- [ ] `source_dataset / original_split / scene_type` 정보를 보존했습니다.
- [ ] 교과 7 QA 결과를 확인했습니다.

### Dataset

- [ ] Dataset Audit을 완료했습니다.
- [ ] Split Policy를 문서화했습니다.
- [ ] Data Leakage 위험을 확인했습니다.
- [ ] Train / Validation / Test를 Freeze했습니다.
- [ ] Dataset Version을 확인할 수 있습니다.

### 학습

- [ ] Smoke Test가 통과되었습니다.
- [ ] Baseline Model이 있습니다.
- [ ] Baseline 학습조건이 기록되어 있습니다.
- [ ] Improved Model 실험이 있습니다.
- [ ] 실패한 실험도 기록되어 있습니다.

### 평가

- [ ] Precision을 기록했습니다.
- [ ] Recall을 기록했습니다.
- [ ] mAP50을 기록했습니다.
- [ ] mAP50-95를 기록했습니다.
- [ ] Class별 성능을 확인했습니다.
- [ ] 대표 FP와 FN을 확인했습니다.
- [ ] 실패 원인 가설을 설명할 수 있습니다.

### Final Model

- [ ] Test를 보기 전에 Final Model을 선택했습니다.
- [ ] `final_best.pt`가 있습니다.
- [ ] Frozen Test 결과가 있습니다.
- [ ] Test에 맞춰 반복 튜닝하지 않았습니다.

### 활용

- [ ] 이미지 Detection이 됩니다.
- [ ] Auto Label 후보를 생성할 수 있습니다.
- [ ] Human Review를 완료할 수 있습니다.

### 문서

- [ ] README가 완성되었습니다.
- [ ] Dataset 관련 문서가 있습니다.
- [ ] Failure Analysis가 있습니다.
- [ ] Experiment Log가 있습니다.
- [ ] Final Test Report가 있습니다.
- [ ] Acceptance Test 결과가 있습니다.
- [ ] 교과 14 연결자료가 있습니다.

### 보안

- [ ] 기업 Dataset이 외부 Git에 포함되지 않았습니다.
- [ ] 원본 이미지가 외부 Git에 포함되지 않았습니다.
- [ ] YOLO TXT 원본이 외부 Git에 포함되지 않았습니다.
- [ ] `*.pt / *.onnx / *.engine`이 외부 Git에 포함되지 않았습니다.
- [ ] `runs/`가 외부 Git에 포함되지 않았습니다.
- [ ] Prediction 이미지가 외부 Git에 포함되지 않았습니다.
- [ ] `.gitignore`가 적용되어 있습니다.
- [ ] `git status`로 마지막 확인을 했습니다.

---

## 22.9 Final Gate를 통과한 뒤 발표 자료를 준비합니다

발표를 위해 새로운 모델을 만들지 않습니다.

이미 만들어 둔 다음 결과를 이용합니다.

```
Dataset Summary

Split 결과

Baseline 성능

Failure Analysis

Baseline vs Improved 비교

Final Test 결과

Prediction 이미지

Auto Label 결과

교과 14 연결 내용
```

권장 발표 자료는 **8~10장 정도**입니다.

```
1. 프로젝트 목표

2. 교과 7 — 정답 데이터 만들기

3. 교과 8 — Dataset과 Split

4. Baseline Model

5. Validation 결과

6. 대표 FP / FN

7. 개선 실험

8. Final Model · Frozen Test

9. Detection · Auto Label

10. 결론 · 한계 · 교과 14 연결
```

많이 보여주는 것이 좋은 발표는 아닙니다.

```
한 장
        ↓
질문 하나
        ↓
결과 하나
        ↓
핵심 설명
```

정도로 구성합니다.

---

## 22.10 교과 7과 교과 8을 하나의 이야기로 발표합니다

이번 발표는 교과 7과 교과 8을 따로 설명하는 발표가 아닙니다.

```
[교과 7 — 5일]

김치 이물질 데이터 확인
        ↓
라벨링 프로그램 개발
        ↓
YOLO Label 검수 · 보정
        ↓
QA · Cross Review
        ↓
신뢰할 수 있는 정답 Dataset 완성


                ↓


[교과 8 — 9일]

정답 Dataset 확인
        ↓
Train / Validation / Test 구성
        ↓
Smoke Test
        ↓
Baseline 학습
        ↓
Validation 평가
        ↓
Failure Analysis
        ↓
한 변수 개선
        ↓
Final Model
        ↓
Frozen Test
        ↓
Detection · Auto Label
```

팀은 다음 한 문장을 설명할 수 있으면 됩니다.

> “교과 7에서는 AI가 학습할 정답 데이터를 만들었고,  
> 교과 8에서는 그 데이터를 이용하여 객체검출 모델을 학습한 뒤  
> 실패 원인을 분석하고 한 가지 조건을 개선하여 Final Model까지 완성했습니다.”

---

## 22.11 발표는 약 10분 정도로 준비합니다

14일 동안 한 모든 작업을 발표하지 않습니다.

권장:

```
프로젝트 발표
약 7~8분

실제 시연
약 2~3분

질의응답
약 3~5분
```

발표에서 꼭 설명할 것은 다음 정도입니다.

```
① 무엇을 만들었는가?

② 어떤 데이터를 사용했는가?

③ Dataset을 어떻게 나누었는가?

④ Baseline은 어떤 결과였는가?

⑤ AI는 무엇을 틀렸는가?

⑥ 무엇을 한 가지 개선했는가?

⑦ Final Model은 어떤 결과였는가?

⑧ 실제로 어떻게 활용할 수 있는가?

⑨ 현재 한계는 무엇인가?

⑩ 교과 14에서는 무엇을 이어서 할 것인가?
```

다음 내용은 전부 설명하지 않아도 됩니다.

```
모든 Python 코드

모든 함수

모든 Epoch

모든 실험

모든 FP / FN

모든 라이브러리 옵션
```

---

## 22.12 팀 발표 역할을 나눕니다

6명이 모두 긴 발표를 할 필요는 없습니다.

예:

```
PM / Integrator
→ 프로젝트 목표 · 전체 흐름

Data / QA
→ 교과 7 데이터 · Dataset · Split

Training
→ Smoke Test · Baseline

Evaluation / Failure
→ Precision · Recall · mAP · FP / FN

Experiment
→ 개선 가설 · 한 변수 실험

Inference / Document
→ Final Model · Detection · Auto Label · 마무리
```

한 사람당 약 1분 내외의 핵심 설명이면 충분합니다.

각 팀원은 자신이 맡은 부분에 대한 질문에는 답할 수 있어야 합니다.

---

## 22.13 최종 시연은 핵심 2가지만 보여줍니다

모든 기능을 발표 현장에서 다시 실행하지 않습니다.

### 시연 1 — 이미지 Detection

```
Final Model Load
        ↓
새 이미지 입력
        ↓
BBox
        ↓
Class
        ↓
Confidence
```

---

### 시연 2 — Auto Label + Human Review

```
새 이미지
        ↓
Final Model
        ↓
BBox · Class 후보
        ↓
YOLO TXT 후보 저장
        ↓
교과 7 라벨링 도구
        ↓
사람이 검수 · 수정
        ↓
승인
```

이 시연은 교과 7과 교과 8이 실제로 연결된다는 것을 가장 잘 보여줍니다.

---

### 시연 실패에 대비합니다

발표 도중 실행환경 문제로 시연이 멈출 수 있습니다.

따라서 다음 백업자료를 준비합니다.

```
Prediction 이미지

Auto Label 결과

Human Review 화면

Final Test 결과
```

시연 오류가 발생해도 발표 전체가 중단되지 않도록 합니다.

---

## 22.14 발표에서 중요하게 보는 것

화려한 디자인이나 어려운 기술용어가 핵심이 아닙니다.

다음 내용을 설명할 수 있는지를 봅니다.

```
프로젝트를 이해하고 있는가?

교과 7과 교과 8이 어떻게 연결되는가?

Dataset을 왜 그렇게 구성했는가?

Baseline은 무엇인가?

모델은 어디에서 틀렸는가?

왜 그 조건을 개선했는가?

Final Model은 어떤 결과인가?

현재 모델의 한계는 무엇인가?

실제 다음 프로젝트에서 어떻게 사용할 것인가?
```

좋은 발표는

```
"mAP가 높게 나왔습니다."
```

로 끝나지 않습니다.

좋은 발표는 다음 흐름을 설명합니다.

```
이 데이터를 사용했습니다.
        ↓
이런 실패가 있었습니다.
        ↓
원인을 이렇게 생각했습니다.
        ↓
이 조건 하나를 바꿨습니다.
        ↓
결과가 이렇게 달라졌습니다.
        ↓
하지만 아직 이런 한계가 있습니다.
```

---

## 22.15 발표와 질의응답을 마치면 교과 8 프로젝트를 종료합니다

발표 후 새로운 기능을 추가하거나 모델을 다시 학습하지 않습니다.

최종 발표와 시연, 질의응답이 끝나면  
교과 8 프로젝트는 완료됩니다.

```
교과 7
정답 데이터 완성
        ↓
교과 8
객체검출 모델 학습 · 평가 · 개선
        ↓
Final Model
        ↓
Detection · Auto Label
        ↓
문서화
        ↓
Final Gate
        ↓
발표 · 시연
        ↓
[교과 8 프로젝트 종료]
```

이후에는 이번 프로젝트에서 정리한

```
Dataset
Final Model
성능 결과
Failure Analysis
알려진 한계
```

를 바탕으로 교과 14 제조 제품검사 프로젝트에서 다시 이어갑니다.