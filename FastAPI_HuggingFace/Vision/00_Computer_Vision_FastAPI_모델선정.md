## 오늘의 목표

이미지 분류와 객체검출의 차이를 구분하고, Hugging Face에서 사전학습 모델을 평가할 때 **Task, 라이선스, 모델 구조, 추론 환경, Fine-tuning 가능성**을 확인할 수 있습니다.

## 3개 교재의 연결

```text
00 : 어떤 Vision AI Task와 모델을 사용할 것인가?
 ↓
01 : ResNet-18 다운로드 → 이미지 분류 → 내 데이터 Fine-tuning → FastAPI 동기 서빙
 ↓
02 : Redis 캐시 + PostgreSQL 작업 기록 + Celery Worker → 동기/비동기 비교
```

## YOLO를 배웠는데 왜 다른 모델을 사용할까요?

YOLO는 주로 **이미지 속 객체의 종류와 위치(바운딩 박스)**를 검출하는 실습에 사용했습니다. 이번에는 이미지를 통째로 `normal / scratch / crack`처럼 분류하는 **ResNet-18 이미지 분류**를 학습합니다. 따라서 지금 제공하는 모델은 위치를 그리는 객체검출 모델이 아니며, 결함 위치가 아니라 이미지 전체의 클래스를 예측합니다. 복수 결함과 위치까지 필요하면 객체검출 또는 Segmentation을 별도로 검토해야 합니다.

### 이번에 선택한 Hugging Face 모델

- 모델: [`microsoft/resnet-18`](https://huggingface.co/microsoft/resnet-18)
- Task: `image-classification`; 원본은 ImageNet-1K **1,000개 클래스** 분류용입니다.
- 라이선스: 모델 저장소에 `apache-2.0` 표시. 사업화 시에도 실제 재배포·고지 및 데이터 관련 조건을 따로 확인합니다.
- 모델 크기: 약 1,170만 파라미터로 실습 접근성이 좋은 편입니다.
- Fine-tuning: 원래 1,000개 출력층을 새 클래스 수에 맞게 교체하고, 사전학습한 Backbone을 활용합니다.
- **주의:** 아직 원본 ResNet-18은 제조 불량 판정기가 아닙니다. 학생이 라벨링한 새 데이터로 학습하기 전에는 `normal`과 `crack`을 정확히 구분한다고 주장할 수 없습니다.

공식 모델 카드: https://huggingface.co/microsoft/resnet-18  
Fine-tuning 가이드: https://huggingface.co/docs/transformers/tasks/image_classification
허깅페이스 모델: https://huggingface.co/datasets/AI-Lab-Makerere/beans?utm_source=chatgpt.com
데이터 다운로드: https://huggingface.co/datasets/AI-Lab-Makerere/beans/tree/main/data

다운로드후 폴더 정리

```
fastapi_resnet18/
└── dataset/
    ├── train/
    │   ├── angular_leaf_spot/
    │   ├── bean_rust/
    │   └── healthy/
    ├── val/
    │   ├── angular_leaf_spot/
    │   ├── bean_rust/
    │   └── healthy/
    └── test/
        ├── angular_leaf_spot/
        ├── bean_rust/
        └── healthy/
```

---

## 원본 교재의 CV Task 비교 설명 (FastAPI 기준으로 보정)

1️⃣ Hugging Face에서 검색할 때 Task(작업) 기준 키워드

Hugging Face에서 모델 찾을 때는 모델명보다 먼저 Task(작업 유형)를 정해 검색하면 목적에 맞는 모델을 찾기 쉽습니다.

---

2️⃣ CV 주요 기능별 검색어 + FastAPI 응답 형태 + 어디에 쓰는지

① Image Classification (이미지 분류)

🔍 검색어

```
image-classification
```

🧠 기능
- 이미지 전체를 하나의 라벨로 분류 (예: 고양이/강아지, 정상/불량, 음식 종류)

✅ FastAPI에서의 응답(JSON 응답 예시)

```json
{
  "task": "image-classification",
  "predictions": [
    {"label": "cat", "score": 0.98},
    {"label": "dog", "score": 0.01}
  ]
}
```

💡 언제 쓰면 최고?
- 가장 쉽고 안정적 (입문/포트폴리오/서비스 MVP)

---
② Object Detection (객체 탐지)

🔍 검색어

```
object-detection
```

🧠 기능
- 이미지 안의 여러 객체 + 위치(box) 찾기

✅ FastAPI 응답(JSON 응답 예시)

```json
{
  "task": "object-detection",
  "detections": [
    {
      "label": "person",
      "score": 0.93,
      "box": {"xmin": 0.12, "ymin": 0.10, "xmax": 0.45, "ymax": 0.88}
    }
  ]
}
```

💡 어디에 쓰나?
- CCTV, 상품 위치 표시, 차량/사람 카운팅, 안전장비 착용 탐지

> 포인트: 프론트에서 box를 그리려면 정규화 좌표(0~1)로 주는 게 편함.

---
③ Image Segmentation (이미지 분할)

🔍 검색어

```
image-segmentation
semantic-segmentation
instance-segmentation
```

🧠 기능
- 픽셀 단위로 영역을 나눔 (배경 제거, 사람/물체 마스크)
    
✅ FastAPI 응답 형태 2가지

1. 마스크 이미지를 파일/URL로 반환 (추천)
2. 
```json
{
"task":"image-segmentation",
"mask_image_url":"/media/masks/abc.png"
}
```

2. 마스크를 배열/압축 데이터로 반환(비추천: 무거움)

💡 어디에 쓰나?
- 배경 제거, 인물/제품 컷아웃, 의료 영상, 결함 영역 표시

---
④ OCR (문자 인식)

🔍 검색어

```
ocr
text-recognition
document-understanding
```

🧠 기능
- 이미지 속 텍스트 추출
    (영수증, 명함, 문서)
    

✅ FastAPI 응답(JSON 응답 예시)

```json
{
  "task": "ocr",
  "text": "Total: 12,900\\nDate: 2026-02-03",
  "blocks": [
    {"text": "Total: 12,900", "box": {"xmin": 0.1, "ymin": 0.2, "xmax": 0.6, "ymax": 0.25}}
  ]
}
```

💡 어디에 쓰나?
- 영수증 자동입력, 문서 검색/요약, 자동 DB 입력 파이프라인

---
⑤ Image-to-Text (이미지 설명 생성 / 캡셔닝)

🔍 검색어

```
image-to-text
image-captioning
```

기능
- 이미지를 자연어 문장으로 설명

✅ FastAPI 응답(JSON 응답 예시)

```json
{
"task":"image-to-text",
"caption":"A cat sitting on a sofa."
}
```

💡 어디에 쓰나?
- 접근성(ALT 텍스트), SNS 자동 설명, 사진 검색 태깅

---
⑥ Face 관련 (주의)

🔍 검색어

```
face-detection
face-recognition
```

기능
- 얼굴 위치 탐지 / (인식은 개인정보 이슈 큼)

✅ FastAPI에서 권장
- face-recognition(누구인지 식별) 은 피하고
- face-detection(얼굴 위치만) 정도로 제한하는 경우가 많음

---
3️⃣ FastAPI로 붙이기 쉬운 순서(실전 추천)

1순위: 이미지 분류

```
image-classification
```
- 제일 단순
- 성능/속도도 무난
- JSON 응답 깔끔

2순위: OCR / 이미지 캡셔닝

```
ocr
image-to-text
```
- 텍스트 결과라 DB 저장/검색/서비스화 쉬움

3순위: Object Detection

```
object-detection
```
- 프론트에서 박스 시각화 필요
- 그래도 임팩트 큼

---
4️⃣ Hugging Face 모델 고를 때 FastAPI 관점 체크리스트

✅ 1) pipeline로 바로 되는지 (제일 중요)

- 모델 페이지에 Transformers `pipeline` 예시와 분류 헤드 Fine-tuning 예시가 있는지 확인
- Task에 맞게 `pipeline("image-classification")` 같은 형태로 사용 가능하면 베스트

✅ 2) License 확인 (상업/배포 가능성)

- `Apache-2.0`, `MIT`, `BSD` 등은 일반적으로 수정·상업적 사용을 허용합니다. 실제 모델 가중치·코드·데이터의 개별 조건도 확인합니다.
- 아래는 조심:
    - `non-commercial`
    - `research only`
    - `no redistribution`

✅ 3) 모델 크기 / 속도

- FastAPI 서버에 붙이면 응답 시간이 UX를 좌우합니다.
- CPU 서버면 MobileNet/작은 ViT 계열이 현실적

✅ 4) 입력 타입

- 이미지 1장인지, 문서/멀티페이지인지 확인

---
5️⃣ FastAPI에서 API 엔드포인트는 이렇게 잡는 게 깔끔함

추천 구조:
- `POST /api/sync/classify/` (01 단계) / `POST /api/async/classify/` (02 단계)
- `POST /api/v1/vision/detect/` (추후 확장)
- `POST /api/v1/vision/ocr/` (추후 확장)
- `POST /api/v1/vision/caption/` (추후 확장)

요청:
- `multipart/form-data` 로 `image` 업로드

응답:
- 위에서 말한 JSON 포맷
---

## 수업 마무리 — 실제 모델 선정 체크리스트

- `Task`가 문제에 맞는지: 이미지 전체 분류 vs 바운딩 박스 검출 vs 픽셀 분할
- 라이선스는 무엇인지: 학습·수정·상업적 이용·재배포 고지
- 사전학습 모델과 **내가 만든 Fine-tuning 모델을 구분**했는지
- 데이터 수집 권한, 클래스 정의, Train/Val/Test 분리를 확인했는지
- FastAPI에서 결과 JSON 형태와 모델 입력 크기·속도를 확인했는지

**다음 파일:** `01_Image_Classification_ResNet18_FastAPI_FineTuning.md`에서 실제 코드를 순서대로 작성합니다.
