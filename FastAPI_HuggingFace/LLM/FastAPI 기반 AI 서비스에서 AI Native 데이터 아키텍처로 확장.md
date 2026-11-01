## 최종 전체 디렉토리

```
fastapi_sentiment/
│
├── .venv/                              # 02: Python 가상환경
│
├── main.py                             # 02 생성 → 04 수정 → 06 최종 수정
├── requirements.txt                    # 02 생성 → 04·06 패키지 추가
├── compose.yaml                        # 06: Docker Redis 컨테이너 설정
├── sentiment.db                        # 04: SQLite DB 생성 → 06 확장
│
├── sentiment/
│   ├── __init__.py                      # 02: Python 패키지 인식
│   │
│   ├── hf_model.py                      # 02: Hugging Face 모델 로딩 및 결과 정리
│   ├── router.py                        # 02 생성 → 04 CRUD → 06 비동기 처리
│   ├── services.py                      # 04 생성 → 06 Redis 캐시 추가
│   │
│   ├── db.py                            # 04: SQLAlchemy DB 연결 및 Session
│   ├── models.py                        # 04: TextItem → 06 상태 필드 확장
│   ├── migrate_06.py                    # 06: 기존 SQLite 테이블 컬럼 확장
│   │
│   ├── settings.py                      # 06: Redis·Celery 연결 설정
│   ├── celery_app.py                    # 06: Celery 애플리케이션 생성
│   └── tasks.py                         # 06: Celery Worker 감정분석 작업
│
├── templates/
│   ├── base.html                        # 02: 공통 HTML 템플릿
│   │
│   └── sentiment/
│       ├── predict.html                 # 02: 동기식 감정분석 화면
│       │
│       ├── textitem_list.html           # 04 생성 → 06 상태 표시 추가
│       ├── textitem_detail.html         # 04 생성 → 06 상태 조회·자동 갱신
│       ├── textitem_form.html           # 04 생성 → 06 비동기 작업 등록
│       └── textitem_confirm_delete.html # 04: 삭제 확인 화면
│
├── static/
│   └── css/
│       └── sentiment.css                # 02: 전체 화면 CSS
│
└── tests/
    └── test_web.py                      # 02 생성 → 04·06 테스트 코드 수정
```

## 꼭 기억할 파일 7개

|파일|핵심 함수·객체|초보자용 역할|
|---|---|---|
|`main.py`|`app = FastAPI()`|서버 출입구|
|`router.py`|`item_create()`, `predict()`, `item_status()`|요청 접수 담당|
|`db.py` / `models.py`|`get_db()`, `TextItem`|DB 연결 및 저장 구조|
|`celery_app.py`|`celery_app = Celery()`|Worker 설정|
|`tasks.py`|`analyze_textitem_task()`|AI 작업 수행 담당|
|`services.py`|`analyze_text()`|캐시 검사 및 추론 요청|
|`hf_model.py`|`get_classifier()`, `normalize_prediction()`|AI 모델 준비 및 결과 정리|

---
## 1. 처음 서버를 실행할 때 — 어떤 파일이 먼저 실행될까요?

![[Pasted image 20261101102021.png]]

![[Pasted image 20261101102041.png]]

왜 서버를 순서대로 실행할까요?

현재 FastAPI 감정분석 프로젝트는 Redis만 Docker에서 실행하고, Celery와 FastAPI는 WSL2의 Python 가상환경에서 실행합니다.

![[Pasted image 20261101103514.png]]

### 서버 실행 순서를 지키는 이유

현재 실습에서는 **Redis → Celery Worker → FastAPI** 순서로 실행합니다.

Redis가 실행되지 않으면 비동기 작업 등록이 실패할 수 있고, Celery Worker가 실행되지 않으면 등록된 AI 분석 작업이 처리되지 않고 대기할 수 있기 때문입니다.

따라서 실습에서는 이 순서대로 실행하여 오류를 줄이고, 각 서버의 정상 동작을 확인합니다.

**Docker Compose의 YAML 파일은 여러 서비스의 실행 환경과 연결 관계를 설정하는 파일입니다.**

현재 작성한 `docker-compose.yml`은 Redis 컨테이너만 관리하며, Celery Worker와 FastAPI는 각각 별도의 터미널에서 실행합니다.

향후 여러 서비스를 Docker Compose로 통합하면 YAML 파일에 서비스 간 의존관계를 설정하여 실행 순서를 관리할 수 있습니다.

---
## 2. 글을 새로 작성하면 — AI 내부에서는 어떻게 처리될까요?

![[Pasted image 20261101102344.png]]

![[Pasted image 20261101104248.png]]

![[mermaid-diagram.png]]

---
## 3. 동기식과 비동기식 차이

![[Pasted image 20261101104655.png]]

![[mermaid-diagram (1).png]]

---

# 현재 FastAPI 기반 AI 서비스에서 AI Native 데이터 아키텍처로 확장

![[Pasted image 20261101105721.png]]

현재까지는 Hugging Face에서 사전학습된 AI 모델을 가져와 FastAPI에 연결하고, 사용자가 입력한 데이터를 AI 모델이 분석하여 결과를 반환하는 서비스를 구현했습니다.

이후 Redis와 Celery Worker를 추가하여 동기식과 비동기식의 차이를 학습했습니다. 동기식은 사용자의 요청 안에서 AI 추론을 완료한 뒤 응답하는 방식이고, 비동기식은 FastAPI가 작업을 접수한 뒤 Redis 작업 큐를 통해 Celery Worker에 분석을 맡기는 방식입니다. 이 과정에서 Docker를 설치하고 Redis 컨테이너를 실행하여 작업 큐와 캐시를 활용하는 방법까지 학습했습니다.

## 1. PostgreSQL과 Docker를 활용한 데이터베이스 확장

다음 단계에서는 현재 SQLite로 구현한 데이터베이스를 PostgreSQL로 전환합니다.

기존에는 SQLAlchemy를 활용하여 감정분석 입력과 결과를 저장하고, CRUD 기능으로 데이터를 생성·조회·수정·삭제했습니다. 앞으로는 PostgreSQL을 기반으로 ERD를 설계하고, 사용자 정보, 입력 데이터, AI 분석 작업, AI 추론 결과, 모델 버전 등을 관계형 테이블로 구분하여 관리합니다.

또한 Docker Compose에 PostgreSQL, FastAPI, Celery Worker 서비스를 추가하여 Redis와 함께 실행하는 구조로 확장합니다. 각 서비스는 독립된 컨테이너에서 실행되며, 필요한 경우 FastAPI와 Celery Worker는 동일한 Python 실행 이미지를 사용하면서 서로 다른 명령으로 실행할 수 있습니다.

이 단계의 목표는 단순히 데이터베이스를 변경하는 것이 아니라, **AI 서비스에서 발생하는 입력 데이터와 추론 결과를 관계형 데이터베이스에 체계적으로 저장하고 관리하는 것**입니다.

## 2. 기존 CRUD에서 AI Native 데이터 설계로 확장

그다음 단계에서는 기존의 CRUD 중심 데이터 관리를 AI Native 데이터 아키텍처로 확장합니다.

기존 CRUD는 그대로 유지합니다. PostgreSQL은 사용자의 입력, 작업 상태, AI 추론 결과 등을 관리하는 운영 데이터베이스 역할을 담당합니다.

여기에 AI 학습과 분석을 위한 데이터 수집·정제·활용 과정을 추가합니다. 사용자가 입력한 텍스트나 AI 추론 결과 같은 구조화된 데이터는 PostgreSQL에 저장하고, 이미지·영상·음성과 같은 대용량 원본 파일은 MinIO 또는 Amazon S3 같은 객체 저장소에 보관할 수 있습니다. PostgreSQL에는 이러한 파일의 저장 경로와 관련 메타데이터를 기록합니다.

이처럼 운영 과정에서 발생하는 데이터를 지속적으로 수집하고, 향후 AI 학습과 성능 개선에 다시 사용할 수 있는 구조를 만드는 것이 중요한 목표입니다.

## 3. 실시간 AI 처리와 레이크하우스 데이터 처리 구분

AI가 분석해야 하는 모든 데이터를 반드시 실시간으로 레이크하우스에 전달할 필요는 없습니다.

예를 들어 사용자가 감정분석 문장을 입력하면 FastAPI와 Celery Worker가 해당 데이터를 바로 분석하고, 결과를 PostgreSQL에 저장합니다. 이 과정은 사용자에게 AI 분석 서비스를 제공하기 위한 운영 처리입니다.

반면 레이크하우스는 이렇게 운영 중에 축적된 데이터를 장기적으로 보관하고 정제하여 분석·통계·AI 재학습 등에 활용하는 역할을 담당합니다.

따라서 **실시간 AI 추론은 FastAPI와 Celery를 중심으로 처리하고, 데이터 분석과 재학습을 위한 처리는 별도의 레이크하우스 파이프라인으로 구성**하는 것이 좋습니다.

산업현장의 카메라 영상이나 센서 데이터처럼 즉각적인 불량 판정 또는 이상 감지가 필요한 경우에는 실시간 또는 준실시간 처리가 필요할 수 있습니다. 그러나 일별 감정분석 통계, 추론 성능 분석, 학습 데이터 준비 등은 일정 주기로 처리하는 배치 방식으로도 충분합니다.

## 4. 레이크하우스 구축 및 데이터 정제

PostgreSQL에 축적된 운영 데이터와 객체 저장소의 원본 데이터를 레이크하우스로 연결합니다.

이때 모든 데이터를 매번 처음부터 복사하는 대신, 신규 또는 변경된 데이터를 주기적으로 가져오거나 필요한 경우 CDC(Change Data Capture) 등의 기술을 활용하여 증분 수집하는 구조를 고려합니다.

레이크하우스에서는 일반적으로 Bronze, Silver, Gold의 세 단계로 데이터를 구성합니다.

Bronze는 원본 데이터를 가능한 한 변경하지 않고 보존하는 단계입니다. Silver는 중복 데이터, 잘못된 값, 누락된 정보 등을 검사하고 분석에 사용할 수 있도록 정제하는 단계입니다. Gold는 정제된 데이터를 목적에 맞게 가공하여 분석 및 비즈니스 활용에 적합하도록 구성하는 단계입니다.

실습에서는 MinIO를 저장소로 사용하고, Parquet와 Apache Iceberg 같은 데이터 형식을 활용하여 레이크하우스의 기본 구조를 구현할 수 있습니다. DuckDB, Trino 등의 SQL 엔진을 연결하면 저장된 데이터를 SQL로 조회하고 분석할 수 있습니다.

## 5. 데이터 마트와 통계·대시보드 구축

레이크하우스에서 정제한 데이터를 활용하여 데이터 마트를 구성합니다.

데이터 마트는 전체 데이터 중 특정 업무나 분석 목적에 필요한 데이터만 정리한 구조입니다.

예를 들어 감정분석 서비스라면 날짜별 분석 요청 건수, 긍정·부정·중립 비율, 시간대별 이용 현황, 모델별 추론 소요 시간과 실패 건수 등을 집계할 수 있습니다.

이러한 통계 데이터는 레이크하우스의 Gold 계층에서 관리하거나, 필요에 따라 PostgreSQL의 별도 마트 테이블에 저장하여 FastAPI 또는 Grafana 대시보드에서 조회하도록 구성할 수 있습니다.

이렇게 하면 기존 CRUD 서비스는 개별 데이터를 관리하고, 데이터 마트는 축적된 데이터를 통계적으로 분석하는 역할을 수행하게 됩니다.

## 6. 레이크하우스와 AI 학습·Fine-tuning 연결

AI Native 데이터 설계에서 중요한 최종 단계는 축적한 데이터를 AI 학습 및 모델 개선에 다시 활용하는 것입니다.

레이크하우스에 저장된 데이터를 모두 AI 학습에 사용하는 것이 아니라, Silver 계층의 정제 데이터 또는 학습 목적에 맞게 가공된 데이터 중에서 품질이 검증되고 정답 라벨이 확보된 데이터를 선별합니다.

이 데이터를 학습·검증·테스트 데이터셋으로 구분하고, 기존 Hugging Face 모델에 추가 학습이나 Fine-tuning을 수행할 수 있습니다. 이때 데이터셋 버전, 모델 버전, 평가 결과를 함께 관리하여 어떤 데이터로 학습한 모델이 실제 서비스에서 어떤 성능을 보였는지 추적합니다.

새로운 모델의 성능이 기존 모델보다 개선되었다는 사실이 검증되면, 모델을 다시 FastAPI 또는 Celery Worker의 추론 환경에 적용합니다.

즉, **AI 추론 → 데이터 축적 → 데이터 정제 → 학습 데이터셋 생성 → Fine-tuning → 성능 평가 → 모델 재배포**로 이어지는 순환 구조를 구현하는 것입니다.

## 7. 최종 교육 목표

최종적으로는 현재의 FastAPI 감정분석 서비스를 시작점으로 PostgreSQL, Docker, Redis, Celery, 레이크하우스, 데이터 마트, AI 재학습까지 연결된 하나의 AI Native 데이터 기반 서비스를 구현하는 것이 목표입니다.

단순히 AI 모델을 FastAPI에 연결하여 추론 결과를 보여주는 수준에서 벗어나, **운영 중 생성된 데이터를 체계적으로 관리하고, 이를 통계 분석과 AI 모델 성능 개선에 반복적으로 활용할 수 있는 데이터 중심 AI 서비스**로 발전시키는 과정입니다.