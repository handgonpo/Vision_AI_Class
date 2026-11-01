## 1. 오늘 수업의 목표

- **동기:** 업로드 → FastAPI에서 이미지 분석 → 결과 JSON 응답(모델 추론이 끝날 때까지 대기)
- **비동기 작업 큐:** 업로드 → PostgreSQL에 작업 등록 → Redis Broker에 작업 ID 등록 → 먼저 `202 Accepted` 응답 → Celery Worker에서 분석 → PostgreSQL에 결과 저장 → 상태조회 API로 결과 수신
- **Redis 캐시:** 동일한 이미지와 동일 모델을 다시 분석할 때 캐시에 저장된 결과 재사용
- **비교:** HTTP 작업 접수 시간과 최종 결과 완료 시간을 **다른 지표**로 기록해 오해하지 않도록 합니다.

### 전체 흐름

![[Pasted image 20261101121104.png]]

```text
                         ┌────── 동기 API ──────┐
브라우저/Insomnia 이미지 ──┤ FastAPI → Redis Cache ├→ Hugging Face ResNet-18
                         │        MISS          │   (또는 Fine-tuning 모델)
                         │           ↑          │          ↓
                         │        HIT ←─────────┤      JSON 응답
                         └──────────────────────┘

                         ┌────── 비동기 API ───────────────────────┐
브라우저/Insomnia 이미지 ──┤ FastAPI → 파일 저장 → PostgreSQL(Job) │
                         │                  → Redis Broker       │
                         │                  → HTTP 202 + job_id │
                         └───────────────────────┬───────────────┘
                                                 ↓
                          Celery Worker → 파일 읽기 → Redis 캐시 조회
                                                 ↓ MISS
                                             ResNet-18 추론
                                                 ↓
                                     PostgreSQL 결과·상태 저장
                                                 ↓
                           브라우저 GET /api/async/jobs/{id}/ 반복
```

## 2. Docker는 처음 한 번 설치하고 계속 재사용합니다

> **환경:** Windows 10/11 + WSL2 Ubuntu + VS Code. 01에서 `fastapi_resnet18` 폴더와 `.venv`를 만들었다고 가정합니다. Docker Desktop은 Windows에 설치하고 WSL2 Integration을 켭니다. WSL 내부에 별도 Docker Engine을 중복 설치할 필요가 없습니다.

### 2-1. Windows의 WSL2 준비 여부 확인 — PowerShell

```powershell
wsl --status
wsl -l -v
```

Ubuntu 배포판에 `VERSION 2`가 보여야 합니다. 필요하다면 관리자 PowerShell에서 `wsl --install`을 실행하고 재부팅합니다. 기관 PC의 Windows·가상화 정책에 따라 관리자 권한이 필요할 수 있습니다.

### 2-2. Docker Desktop 최초 설치 — Windows

1. 공식 다운로드 페이지 <https://docs.docker.com/desktop/setup/install/windows-install/> 에서 Windows용 Docker Desktop을 다운로드합니다.
2. 설치 시 WSL 2 기반 엔진(WSL 2 backend) 사용을 선택합니다.
3. 설치를 마친 뒤 **Docker Desktop을 직접 실행**하고 엔진 실행 상태가 될 때까지 초기화합니다.
4. Docker Desktop **Settings → General → Use the WSL 2 based engine** 설정을 확인합니다.
5. **Settings → Resources → WSL Integration**에서 실습에 사용하는 Ubuntu 배포판을 활성화합니다.
6. Ubuntu(WSL2) 터미널을 새로 열어 아래 명령이 정상 동작하는지 확인합니다.

```bash
docker --version
docker compose version
docker info
```

`docker --version`만 되는데 `docker info`에서 데몬 연결 오류가 나면 Desktop 엔진이 시작되지 않았거나 WSL Integration이 꺼져 있을 가능성이 있습니다. 이때 FastAPI를 먼저 실행하지 말고 Docker 연결부터 확인합니다.

**주의:** 교육용 Compose는 Redis `6379`, PostgreSQL `5432`를 `127.0.0.1`에만 노출합니다. 기존 Redis 또는 PostgreSQL이 같은 포트를 사용한다면 컨테이너를 추가하기 전에 충돌을 확인합니다. `.env`와 예시 비밀번호는 교육용이며 상용 서비스에서 그대로 사용하면 안 됩니다.

### 2-3. 01 프로젝트 복사 및 설치 상태 확인

```bash
cd ~/fastapi_resnet18
source .venv/bin/activate
# 02용 requirements.txt를 아래 내용으로 전체 교체하고 나서 설치합니다.
```

**`requirements.txt` — 전체 코드**

```text
fastapi>=0.115,<1
uvicorn[standard]>=0.30,<1
python-multipart>=0.0.9,<1
jinja2>=3.1,<4
pillow>=10,<13
transformers>=4.47,<5
torch>=2.3,<3
torchvision>=0.18,<1
sqlalchemy>=2.0,<3
psycopg[binary]>=3.2,<4
redis>=5,<7
celery[redis]>=5.4,<6
httpx>=0.27,<1
pytest>=8,<10
python-dotenv>=1,<2
```

```bash
uv pip install -r requirements.txt
# uv 명령이 없을 때: python -m pip install -r requirements.txt
```

### 2-4. `compose.yaml`을 만들고 Redis · PostgreSQL 실행

**이 파일은 Docker로 실행할 서비스만 정의합니다.** FastAPI와 Celery는 이번 실습에서 학생의 `.venv`로 실행합니다. Docker 컨테이너 내부의 `localhost`와 WSL 호스트의 `localhost`는 다른 의미이므로, 이 코드에서는 Redis/PostgreSQL만 Docker에서 실행하고 Python 앱은 호스트에서 실행하는 구조로 고정합니다.

**`compose.yaml` — 전체 코드**

```yaml
services:
  redis:
    image: redis:7.4-alpine
    restart: unless-stopped
    ports:
      - '127.0.0.1:6379:6379'
    volumes:
      - redis_data:/data
    command: ["redis-server", "--appendonly", "yes"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 10

  postgres:
    image: postgres:17-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: vision_user
      POSTGRES_PASSWORD: vision_pass
      POSTGRES_DB: vision_db
    ports:
      - '127.0.0.1:5432:5432'
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U vision_user -d vision_db"]
      interval: 5s
      timeout: 3s
      retries: 10

volumes:
  redis_data:
  postgres_data:
```

```bash
# ★ 먼저 Docker Desktop을 실행해 두어야 합니다.
docker compose up -d
docker compose ps

# Redis 응답 확인: PONG
docker compose exec redis redis-cli ping

# PostgreSQL 접속 가능 여부 확인: accepting connections
docker compose exec postgres pg_isready -U vision_user -d vision_db
```

처음 실행할 때 이미지는 Docker Hub에서 다운로드됩니다. 중지했다가 다음 수업에 다시 켤 때는 `docker compose start`, 종료할 때는 `docker compose stop`을 사용합니다. 컨테이너 설정을 바꾼 경우에는 `docker compose up -d`를 다시 사용합니다. **`docker compose down -v`는 볼륨까지 삭제하여 데이터가 사라질 수 있으므로 실습 중 사용하지 않습니다.**

### 2-5. DBeaver에서 PostgreSQL 확인(선택)

DBeaver는 Docker 이미지가 아닌 PC 프로그램으로 설치하고, 새 PostgreSQL 연결을 만듭니다. Host `127.0.0.1`, Port `5432`, Database `vision_db`, Username `vision_user`, Password `vision_pass`를 입력합니다. **테이블은 FastAPI lifespan 시작 단계가 실행된 이후에 확인**할 수 있습니다. DBeaver는 데이터 마트가 아니라 DB 조회·SQL 실행용 관리 도구입니다.

## 3. 환경변수와 최종 파일 구조

이제 01의 소스코드를 삭제하지 않고 필요한 파일만 추가·교체합니다.

```text
fastapi_resnet18/
├── .env                        # 직접 생성; Git에 올리지 않음
├── .env.example
├── compose.yaml                # Docker Redis + PostgreSQL
├── requirements.txt            # 02 확장판
├── main.py                     # 02 버전으로 전체 교체
├── ai/
│   ├── __init__.py
│   ├── config.py                # 02 버전으로 교체
│   ├── hf_model.py              # 01 그대로 유지
│   ├── services.py              # 01 그대로 유지
│   ├── cache_service.py        # 신규
│   ├── database.py             # 신규
│   ├── models.py               # 신규
│   ├── celery_app.py           # 신규
│   ├── tasks.py                # 신규
│   ├── router_sync.py          # 02 버전으로 교체
│   ├── router_async.py         # 신규
│   └── router_web.py           # 02 버전으로 교체
├── training/finetune.py        # 01 그대로 유지
├── templates/
│   ├── base.html               # 02 버전으로 교체
│   ├── sync.html               # 01 그대로 유지
│   └── async.html              # 신규
├── static/style.css            # 01 그대로 유지
├── media/jobs/                 # 사용자가 업로드한 이미지 바이트 보관
├── artifacts/cap_quality/      # Fine-tuning 모델을 만들었다면 유지
└── tests/test_api.py           # API 통합 모의 테스트
```

`.env` 파일은 `.env.example`을 복사해 만듭니다. 01에서 학습을 마쳤으면 `MODEL_PATH=artifacts/cap_quality`, 아직 Fine-tuning하지 않았다면 `MODEL_PATH=microsoft/resnet-18`을 사용합니다. 두 실행 프로세스 모두 같은 값을 읽게 됩니다.

**`ai/config.py` — 전체 코드**

```python
import os
from pathlib import Path
from dotenv import load_dotenv

BASE_DIR = Path(__file__).resolve().parent.parent
load_dotenv(BASE_DIR / ".env")
MODEL_PATH = os.getenv("MODEL_PATH", "microsoft/resnet-18")
DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "postgresql+psycopg://vision_user:vision_pass@127.0.0.1:5432/vision_db",
)
REDIS_CACHE_URL = os.getenv("REDIS_CACHE_URL", "redis://127.0.0.1:6379/1")
CELERY_BROKER_URL = os.getenv("CELERY_BROKER_URL", "redis://127.0.0.1:6379/2")
CELERY_RESULT_BACKEND = os.getenv("CELERY_RESULT_BACKEND", "redis://127.0.0.1:6379/3")
MAX_UPLOAD_BYTES = 5 * 1024 * 1024
MAX_IMAGE_PIXELS = 12_000_000
MEDIA_DIR = BASE_DIR / "media" / "jobs"
```

**`.env.example` — 전체 코드**

```bash
# 첫 단계: 원본 이미지 분류(1000개 ImageNet 클래스)
MODEL_PATH=microsoft/resnet-18
# 추가학습 후: MODEL_PATH=artifacts/cap_quality
DATABASE_URL=postgresql+psycopg://vision_user:vision_pass@127.0.0.1:5432/vision_db
REDIS_CACHE_URL=redis://127.0.0.1:6379/1
CELERY_BROKER_URL=redis://127.0.0.1:6379/2
CELERY_RESULT_BACKEND=redis://127.0.0.1:6379/3
```

**`.gitignore` — 전체 코드**

```bash
.venv/
__pycache__/
.pytest_cache/
.env
media/jobs/*
!media/jobs/.gitkeep
artifacts/*
!artifacts/.gitkeep
*.db
dataset/
```

## 4. SQLAlchemy 데이터베이스 생성

PostgreSQL은 원본 이미지 원본 바이트를 DB에 넣는 대신 **작업 정보와 추론 결과**를 저장합니다. 업로드 이미지 자체는 `media/jobs/` 폴더에 보관합니다. API 서버와 워커가 같은 프로젝트 폴더를 공유해야 합니다.

**코드를 작성하기 전 의사코드**

```text
DB 접속 환경 변수 → SQLAlchemy Engine/Session 생성 → ImageJob 테이블 정의 → FastAPI lifespan 시작 단계에서 테이블이 없으면 생성
```

**`ai/database.py` — 전체 코드**

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker
from .config import DATABASE_URL

engine = create_engine(DATABASE_URL, pool_pre_ping=True)
SessionLocal = sessionmaker(bind=engine, expire_on_commit=False)

class Base(DeclarativeBase):
    pass


def init_db():
    from . import models  # noqa: F401  -- 모델 등록
    Base.metadata.create_all(bind=engine)
```

**`ai/models.py` — 전체 코드**

```python
from datetime import datetime, timezone
from sqlalchemy import DateTime, Float, Integer, JSON, String, Text
from sqlalchemy.orm import Mapped, mapped_column
from .database import Base

def utcnow():
    return datetime.now(timezone.utc)

class ImageJob(Base):
    __tablename__ = "image_jobs"
    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    image_path: Mapped[str] = mapped_column(String(150), nullable=False)
    status: Mapped[str] = mapped_column(String(20), default="queued", index=True)
    model_id: Mapped[str] = mapped_column(String(255), default="")
    top1_label: Mapped[str | None] = mapped_column(String(200), nullable=True)
    top1_score: Mapped[float | None] = mapped_column(Float, nullable=True)
    all_scores: Mapped[list | None] = mapped_column(JSON, nullable=True)
    inference_ms: Mapped[float | None] = mapped_column(Float, nullable=True)
    cache_hit: Mapped[bool | None] = mapped_column(nullable=True)
    error_message: Mapped[str | None] = mapped_column(Text, nullable=True)
    created_at: Mapped[datetime] = mapped_column(DateTime(timezone=True), default=utcnow)
    updated_at: Mapped[datetime] = mapped_column(DateTime(timezone=True), default=utcnow, onupdate=utcnow)
```

상태는 `queued → running → done` 또는 오류 시 `failed`입니다. `all_scores`는 JSON으로 보관하므로 DBeaver에서 확인할 수 있습니다. 운영 환경에서는 `create_all()`로 테이블 변경을 관리하지 않고 Alembic 같은 마이그레이션 도구를 사용하는 것이 좋습니다.

## 5. Redis를 캐시로 사용하는 service 확장

원본의 핵심 개념 **캐시 HIT이면 모델을 호출하지 않고 즉시 반환, MISS면 모델 추론 후 10분간 캐시에 기록**을 유지합니다. 캐시 키에 `MODEL_PATH`와 이미지 바이트 해시를 포함하여 모델을 바꾸었을 때 다른 결과가 섞이지 않도록 합니다.

**코드를 작성하기 전 의사코드**

```text
이미지와 모델 경로로 cache key → Redis GET → HIT이면 반환 → MISS이면 services.classify_image_bytes → Redis SETEX 600초 → 반환
```

**`ai/cache_service.py` — 전체 코드**

```python
import json
import logging
from functools import lru_cache
from .config import REDIS_CACHE_URL
from .services import cache_key, classify_image_bytes

log = logging.getLogger(__name__)

@lru_cache(maxsize=1)
def get_redis():
    from redis import Redis
    return Redis.from_url(REDIS_CACHE_URL, decode_responses=True,
                          socket_connect_timeout=1, socket_timeout=1)


def classify_with_cache(image_bytes: bytes) -> dict:
    """Redis 장애 시에도 모델 추론을 진행하는 실습용 캐시 폴백."""
    key = cache_key(image_bytes)
    try:
        saved = get_redis().get(key)
        if saved:
            result = json.loads(saved)
            return {**result, "cache_hit": True, "inference_ms": None}
    except Exception as exc:
        log.warning("Redis 조회 불가; 모델 추론으로 전환: %s", exc)

    result = classify_image_bytes(image_bytes)
    try:
        get_redis().setex(key, 600, json.dumps(result, ensure_ascii=False))
    except Exception as exc:
        log.warning("Redis 캐시 저장 실패: %s", exc)
    return result
```

`ai/services.py`와 `ai/hf_model.py`는 01의 코드를 **수정하지 않습니다**. 캐시 HIT에서는 실제 모델 추론을 하지 않으므로 `inference_ms=null`로 구분합니다. 캐시가 일시적으로 연결되지 않으면 실습용으로 추론은 계속되도록 폴백 처리했습니다. **큐 등록 자체는 Redis가 없으면 503 오류**가 발생합니다. Redis 캐시(`DB /1`), Celery 브로커(`DB /2`), Celery 결과 저장소(`DB /3`)를 서로 구분합니다.

## 6. Celery 앱과 워커 Task 만들기

FastAPI 서버가 직접 무거운 이미지를 Celery 큐에 넣는 대신, **DB에서 생성한 job_id만 전달**합니다. 워커는 DB에서 파일 경로를 읽고 추론한 후 DB 결과를 갱신합니다.

**코드를 작성하기 전 의사코드**

```text
Celery 앱 생성 → Redis Broker/Backend 설정 → tasks.py 등록 → 워커가 job_id 수신 → DB 상태 running → 이미지 읽기 → Redis 캐시/ResNet 추론 → DB done 또는 failed
```

**`ai/celery_app.py` — 전체 코드**

```python
from celery import Celery
from .config import CELERY_BROKER_URL, CELERY_RESULT_BACKEND

celery_app = Celery(
    "vision_ai",
    broker=CELERY_BROKER_URL,
    backend=CELERY_RESULT_BACKEND,
    include=["ai.tasks"],
)
celery_app.conf.update(
    task_serializer="json",
    accept_content=["json"],
    result_serializer="json",
    timezone="Asia/Seoul",
    task_track_started=True,
    broker_connection_retry_on_startup=True,
)
```

**`ai/tasks.py` — 전체 코드**

```python
from .celery_app import celery_app
from .config import MEDIA_DIR, MODEL_PATH
from .database import SessionLocal
from .models import ImageJob
from .cache_service import classify_with_cache

@celery_app.task(name="ai.classify_job")
def classify_job(job_id: int):
    """워커가 DB → 이미지 → Redis/AI → DB 순서로 처리한다."""
    with SessionLocal() as db:
        job = db.get(ImageJob, job_id)
        if job is None:
            return {"error": "job_not_found", "job_id": job_id}
        if job.status == "done":
            return {"job_id": job_id, "status": "done"}
        job.status = "running"
        db.commit()
        image_path = MEDIA_DIR / job.image_path
        try:
            image_bytes = image_path.read_bytes()
            result = classify_with_cache(image_bytes)
            job.model_id = result["model_id"]
            job.top1_label = result["top1"]["label"]
            job.top1_score = result["top1"]["score"]
            job.all_scores = result["all"]
            job.inference_ms = result["inference_ms"]
            job.cache_hit = result["cache_hit"]
            job.status = "done"
            job.error_message = None
            db.commit()
            return {"job_id": job_id, "status": "done"}
        except Exception as exc:
            db.rollback()
            job = db.get(ImageJob, job_id)
            job.status = "failed"
            job.error_message = f"{type(exc).__name__}: {exc}"[:1500]
            db.commit()
            raise
```

**실무 오류 주의:** 원본 Django 예제의 `failed` 처리와 자동 재시도가 결합되면 작업 재시도에서 실패 상태를 건너뛰는 버그가 생길 수 있습니다. 이번 기초 교재에서는 자동 재시도를 사용하지 않고 실패 상태를 명확히 기록합니다. 자동 재시도와 중복 실행 방지는 이후 별도 실습에서 다룹니다.

## 7. 동기 API를 Redis 캐시와 연결

01의 `/api/sync/classify/` 주소는 그대로 유지합니다. 단, 직접 모델을 호출하던 코드를 `classify_with_cache()`로 변경합니다. 이후 동기/비동기 방식이 같은 캐시 정책을 사용하도록 합니다.

**`ai/router_sync.py` — 전체 코드**

```python
from fastapi import APIRouter, File, HTTPException, UploadFile
from .config import MAX_UPLOAD_BYTES
from .services import InvalidImage
from .cache_service import classify_with_cache

router = APIRouter(tags=["동기 추론"])

@router.post("/api/sync/classify/")
def classify_sync(image: UploadFile = File(...)):
    """HTTP 요청이 추론이 끝날 때까지 기다린다."""
    image_bytes = image.file.read(MAX_UPLOAD_BYTES + 1)
    if len(image_bytes) > MAX_UPLOAD_BYTES:
        raise HTTPException(status_code=413, detail="5MB 이하 파일만 허용합니다.")
    try:
        return classify_with_cache(image_bytes)
    except InvalidImage as exc:
        raise HTTPException(status_code=400, detail=str(exc)) from exc
```

## 8. 비동기 작업 접수 API + 상태 조회 API 만들기

FastAPI는 `UploadFile`과 `File`로 업로드를 검증합니다. 유효한 이미지만 서버가 생성한 고유 파일명으로 보관하고 Job 테이블에 `queued`로 저장한 뒤 `classify_job.delay(job_id)`로 워커에게 맡깁니다. 요청은 `202 Accepted`로 먼저 응답합니다.

**코드를 작성하기 전 의사코드**

```text
이미지 검사 → 고유 이름으로 파일 저장 → DB queued 생성 → Celery.delay(job_id) → 202+job_id → GET 상태 API에서 queued/running/done/failed 반환
```

**`ai/router_async.py` — 전체 코드**

```python
from uuid import uuid4
from fastapi import APIRouter, File, HTTPException, UploadFile
from sqlalchemy.exc import SQLAlchemyError
from .config import MAX_UPLOAD_BYTES, MEDIA_DIR, MODEL_PATH
from .database import SessionLocal
from .models import ImageJob
from .services import InvalidImage, load_rgb_image
from .tasks import classify_job

router = APIRouter(tags=["비동기 큐"])

@router.post("/api/async/classify/", status_code=202)
def enqueue_classify(image: UploadFile = File(...)):
    image_bytes = image.file.read(MAX_UPLOAD_BYTES + 1)
    if len(image_bytes) > MAX_UPLOAD_BYTES:
        raise HTTPException(status_code=413, detail="5MB 이하 파일만 허용합니다.")
    try:
        load_rgb_image(image_bytes)  # 큐 등록 전에 유효한 이미지인지 확인
    except InvalidImage as exc:
        raise HTTPException(status_code=400, detail=str(exc)) from exc

    MEDIA_DIR.mkdir(parents=True, exist_ok=True)
    filename = f"{uuid4().hex}.bin"  # 사용자가 보낸 파일명은 사용하지 않는다.
    (MEDIA_DIR / filename).write_bytes(image_bytes)
    with SessionLocal() as db:
        job = ImageJob(image_path=filename, status="queued", model_id=MODEL_PATH)
        db.add(job)
        db.commit()
        db.refresh(job)
        job_id = job.id
        try:
            classify_job.delay(job_id)  # 추론은 별도 Celery 프로세스에서 수행
        except Exception as exc:
            job.status = "failed"
            job.error_message = "작업 큐 등록 실패: Redis 연결을 확인하세요."
            db.commit()
            raise HTTPException(status_code=503, detail=job.error_message) from exc
    return {"job_id": job_id, "status": "queued", "check_url": f"/api/async/jobs/{job_id}/"}

@router.get("/api/async/jobs/{job_id}/")
def get_job(job_id: int):
    with SessionLocal() as db:
        job = db.get(ImageJob, job_id)
        if job is None:
            raise HTTPException(status_code=404, detail="해당 작업이 없습니다.")
        return {
            "job_id": job.id,
            "status": job.status,
            "model_id": job.model_id,
            "top1": ({"label": job.top1_label, "score": job.top1_score}
                     if job.status == "done" else None),
            "all": job.all_scores if job.status == "done" else None,
            "inference_ms": job.inference_ms,
            "cache_hit": job.cache_hit,
            "error_message": job.error_message if job.status == "failed" else None,
        }
```

**주의:** `202`는 '분석 완료'가 아니라 '작업 접수'를 뜻합니다. 작업 상태 조회는 `GET /api/async/jobs/{job_id}/`입니다. `404`는 없는 작업 ID이고, Redis 브로커 접속 실패로 enqueue하지 못한 경우 `503`을 반환합니다.

**처리 저장 방식:** 모델 추론 이미지 바이트는 PostgreSQL 테이블이 아니라 `media/jobs/<uuid>.bin`으로 저장합니다. 확장자를 `.bin`으로 둔 것은 원본 파일명을 신뢰하지 않고 서버에서만 읽기 때문입니다. 이미지 디코딩은 Pillow가 수행합니다.

## 9. 화면 라우터 및 비동기 폴링 UI

01의 동기 화면은 유지합니다. 새로운 비동기 화면은 접수 요청에서 `job_id`를 받은 뒤 1초 간격으로 상태 조회 API를 호출합니다. 페이지에서 중지 버튼은 **상태 조회만 중지**하며 이미 접수된 Celery 작업을 취소하지는 않습니다.

**`ai/router_web.py` — 전체 코드**

```python
from fastapi import APIRouter, Request
from fastapi.responses import RedirectResponse
from fastapi.templating import Jinja2Templates
from .config import BASE_DIR

router = APIRouter(tags=["실습 화면"])
templates = Jinja2Templates(directory=str(BASE_DIR / "templates"))

@router.get("/", include_in_schema=False)
def home():
    return RedirectResponse(url="/vision/sync/")

@router.get("/vision/sync/")
def sync_page(request: Request):
    return templates.TemplateResponse(request=request, name="sync.html")

@router.get("/vision/async/")
def async_page(request: Request):
    return templates.TemplateResponse(request=request, name="async.html")
```

**`templates/base.html` — 전체 코드**

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% block title %}Vision AI 분류 실습{% endblock %}</title>
  <link rel="stylesheet" href="{{ url_for('static', path='/style.css') }}">
</head>
<body>
 <main class="wrap">
  <header><h1>ResNet-18 이미지 분류 실습</h1>
   <nav><a href="/vision/sync/">동기</a> · <a href="/vision/async/">비동기</a> · <a href="/docs">Swagger UI</a></nav>
  </header>
  {% block content %}{% endblock %}
 </main>
</body>
</html>
```

**`templates/async.html` — 전체 코드**

```html
{% extends "base.html" %}
{% block title %}비동기식 이미지 분류{% endblock %}
{% block content %}
<section class="panel"><h2>비동기식: 작업 ID를 먼저 받고 결과는 나중에 확인합니다</h2>
<input id="image" type="file" accept="image/jpeg,image/png,image/webp">
<button id="send">Celery 작업 등록</button><button id="cancel">상태조회 중지</button>
<img id="preview" class="preview" hidden>
<p id="state">이미지를 선택하세요.</p><pre id="output">{}</pre></section>
<script>
const fileEl=document.getElementById('image');let cancelled=false;
fileEl.onchange=()=>{const f=fileEl.files[0];if(f){const p=document.getElementById('preview');p.src=URL.createObjectURL(f);p.hidden=false;}};
document.getElementById('cancel').onclick=()=>{cancelled=true;document.getElementById('state').textContent='상태 조회만 중지했습니다(워커 작업은 계속됩니다).';};
const sleep=ms=>new Promise(resolve=>setTimeout(resolve,ms));
document.getElementById('send').onclick=async()=>{
 const file=fileEl.files[0];if(!file){alert('이미지를 선택하세요');return;}
 const button=document.getElementById('send');button.disabled=true;cancelled=false;
 const state=document.getElementById('state');const out=document.getElementById('output');
 const form=new FormData();form.append('image',file);const started=performance.now();
 state.textContent='작업 등록 중...';
 try{
   const res=await fetch('/api/async/classify/',{method:'POST',body:form});
   const data=await res.json();out.textContent=JSON.stringify(data,null,2);
   if(!res.ok){state.textContent=`등록 실패 HTTP ${res.status}`;return;}
   const accepted=Math.round(performance.now()-started);
   state.textContent=`접수 완료: ${accepted}ms, job_id=${data.job_id}`;
   for(let i=0;i<60 && !cancelled;i++){
     await sleep(1000);if(cancelled)break;
     const s=await fetch(data.check_url);const job=await s.json();
     out.textContent=JSON.stringify(job,null,2);
     state.textContent=`접수 ${accepted}ms / 전체 ${Math.round(performance.now()-started)}ms / 상태 ${job.status}`;
     if(['done','failed'].includes(job.status))break;
   }
 }catch(err){state.textContent=`요청 실패: ${err}`;}finally{button.disabled=false;}
};
</script>
{% endblock %}
```

`templates/sync.html`과 `static/style.css`는 **01에서 만든 파일을 그대로 사용**합니다. `/vision/sync/`와 `/vision/async/` 두 화면에서 같은 이미지를 비교합니다.

## 10. main.py 최종 변경 — 라우터가 모두 생긴 다음 연결

현재까지 `router_sync.py`, `router_async.py`, `router_web.py`, `database.py`를 생성했으므로 **이제** `main.py` 전체를 아래 내용으로 교체합니다. 존재하지 않는 라우터를 먼저 import하여 실행이 중단되는 오류를 피할 수 있습니다.

**코드를 작성하기 전 의사코드**

```text
FastAPI 앱 → static mount → web/sync/async router 연결 → lifespan 시작 시 DB 테이블 생성 → /health 등록
```

**`main.py` — 전체 코드**

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from ai.config import BASE_DIR
from ai.router_sync import router as sync_router
from ai.router_async import router as async_router
from ai.router_web import router as web_router
from ai.database import init_db

@asynccontextmanager
async def lifespan(app: FastAPI):
    # DB 연결이 불가능하면 서버 기동 시 알려준다 (Docker를 먼저 실행).
    init_db()
    yield

app = FastAPI(title="ResNet18 Fine-tuning · FastAPI · Redis · Celery", lifespan=lifespan)
app.mount("/static", StaticFiles(directory=str(BASE_DIR / "static")), name="static")
app.include_router(web_router)
app.include_router(sync_router)
app.include_router(async_router)

@app.get("/health")
def health():
    return {"status": "ok"}
```

## 11. Docker → Celery → FastAPI 실행 순서 (중요)

### 터미널 A — WSL2 Ubuntu: Redis·PostgreSQL 먼저 실행

```bash
cd ~/fastapi_resnet18
docker compose up -d
docker compose ps
docker compose exec redis redis-cli ping
# 예상: PONG
docker compose exec postgres pg_isready -U vision_user -d vision_db
```

### 터미널 B — WSL2 Ubuntu: Celery Worker 실행

```bash
cd ~/fastapi_resnet18
source .venv/bin/activate
celery -A ai.celery_app:celery_app worker --loglevel=info --pool=solo --concurrency=1
```

이 명령을 실행한 터미널은 **계속 켜 두어야 합니다.** `--pool=solo`는 초보자 실습에서 워커를 한 프로세스로 처리하여 로그를 쉽게 보게 하는 설정입니다. `MODEL_PATH=artifacts/cap_quality`이면 해당 폴더와 가중치가 존재하는지 먼저 확인합니다.

### 터미널 C — WSL2 Ubuntu: FastAPI 실행

```bash
cd ~/fastapi_resnet18
source .venv/bin/activate
uvicorn main:app --reload
```

`/health` 요청은 웹 서버 자체가 살아 있는지만 보여주며 Redis·PostgreSQL·모델까지 정상이라는 뜻은 아닙니다. 먼저 `docker compose ps`와 `PONG` 및 워커 로그를 따로 확인합니다.

### 서버가 열리면 확인할 URL

| URL | 역할 |
|---|---|
| `http://127.0.0.1:8000/health` | 서버 생존 여부 |
| `http://127.0.0.1:8000/docs` | Swagger API 테스트 |
| `http://127.0.0.1:8000/vision/sync/` | 동기 이미지 분류 화면 |
| `http://127.0.0.1:8000/vision/async/` | Redis·Celery 비동기 분류 화면 |
| `POST /api/sync/classify/` | 분석 완료까지 HTTP 대기 |
| `POST /api/async/classify/` | 작업 접수 후 202+job_id |
| `GET /api/async/jobs/{job_id}/` | 분석 작업 상태·결과 확인 |

## 12. API 실습: Insomnia 또는 curl

### 동기식

```bash
curl -X POST -F "image=@samples/example.jpg" http://127.0.0.1:8000/api/sync/classify/
```

### 비동기식 등록

```bash
curl -i -X POST -F "image=@samples/example.jpg" http://127.0.0.1:8000/api/async/classify/
```

예상 접수 JSON(숫자는 예시):

```json
{"job_id": 1, "status": "queued", "check_url": "/api/async/jobs/1/"}
```

### 상태 확인

```bash
curl http://127.0.0.1:8000/api/async/jobs/1/
```

```json
{
  "job_id": 1, "status": "done", "model_id": "microsoft/resnet-18",
  "top1": {"label": "some ImageNet class", "score": 0.8},
  "all": [{"label": "some ImageNet class", "score": 0.8}],
  "inference_ms": 100.0, "cache_hit": false, "error_message": null
}
```

원본 모델은 ImageNet 클래스 문자열을 반환하므로 `normal/scratch/crack`이라는 결과가 나오려면 실제 Fine-tuning 후 로컬 모델로 전환해야 합니다. JSON 값은 이해를 돕기 위한 예시이며 실제 출력은 모델·입력에 따라 달라집니다.

### 동기·비동기 화면 확인과 원본 이미지 위치 보존

> 아래 이미지는 **원본 Django/DRF + Cat/Dog 모델의 실습 화면**입니다. 순서와 이미지 링크를 유지했으며, 현 수업에서는 새 FastAPI 화면으로 재촬영하면 됩니다.

**동기식 화면:** `http://127.0.0.1:8000/vision/sync/`

![[Pasted image 20260207224941.png]]

**비동기식 화면:** `http://127.0.0.1:8000/vision/async/`

![[Pasted image 20260207224920.png]]

## 13. 동기/비동기 성능을 정확히 비교합니다

단순히 동기 요청 완료시간과 비동기 요청 접수시간을 나눈 수치를 **AI 추론 자체가 빨라졌다는 증거로 사용하면 안 됩니다.** 서로 작업이 다른 지표이기 때문입니다.

| 지표 | 동기 | 비동기 |
|---|---|---|
| HTTP 요청 응답시간 | 모델 추론 완료까지 | 큐 등록까지 |
| 전체 결과 완료시간 | HTTP 완료와 거의 일치 | 접수부터 worker 완료까지 별도 측정 필요 |
| AI 모델 추론시간 | `inference_ms` | `inference_ms` |
| 캐시 영향 | HIT/MISS 기록 필요 | HIT/MISS 기록 필요 |
| 동시 처리량 | 서버/모델 실행 자원에 따라 | 워커 수·큐·모델 자원에 따라 |

**실습 비교 규칙:** 동일한 이미지·동일한 모델 버전, 워밍업 유무 통일, 캐시 HIT/MISS 구분, 각 5~10회 반복, 중앙값 또는 평균 기록, 비동기에서는 enqueue 시간과 최종 완료시간을 모두 수집합니다. 이 교재의 화면은 각 시간을 따로 표시합니다.

## 14. 코드 테스트 파일: 실제 서버를 띄우기 전에 점검

다음 테스트는 PostgreSQL 대신 임시 SQLite를 쓰고, Hugging Face 모델/Redis 큐 등록을 모의 처리합니다. 이를 통해 **HTTP 업로드 검증과 DB 상태 전이·워커 로직의 함수 호출 경로**를 확인할 수 있습니다. 이 테스트 자체가 Docker·Redis·Celery 실제 구동의 증거는 아닙니다.

**`tests/test_api.py` — 전체 코드**

```python
"""통합 테스트: Hugging Face 모델과 Celery 브로커는 스텁으로 바꾼다.
실제 HF 추론/Docker/Redis/Celery 동작을 검증하지는 않는다.
"""
import os
import tempfile
from pathlib import Path
from io import BytesIO
from unittest.mock import patch

os.environ["DATABASE_URL"] = "sqlite:///" + str(Path(tempfile.mkdtemp()) / "api_test.db")

from fastapi.testclient import TestClient
from PIL import Image
import ai.services as services
import ai.router_async as router_async
import ai.tasks as tasks
from main import app


def image_bytes():
    im = Image.new("RGB", (40, 40), color=(90, 140, 210))
    buf = BytesIO()
    im.save(buf, format="PNG")
    return buf.getvalue()


def fake_classifier(image, top_k=5):
    return [{"label": "normal", "score": 0.92}, {"label": "scratch", "score": 0.08}]


def test_end_to_end(monkeypatch, tmp_path):
    monkeypatch.setattr(services, "get_classifier", lambda: fake_classifier)
    monkeypatch.setattr(router_async, "MEDIA_DIR", tmp_path / "jobs")
    monkeypatch.setattr(tasks, "MEDIA_DIR", tmp_path / "jobs")
    # Redis가 실제로 설치되지 않은 테스트에서도 같은 경로를 확인하도록 캐시 호출을 모의 처리한다.
    import ai.router_sync as router_sync
    import ai.cache_service as cache_service
    monkeypatch.setattr(cache_service, "classify_with_cache", services.classify_image_bytes)
    monkeypatch.setattr(router_sync, "classify_with_cache", services.classify_image_bytes)
    monkeypatch.setattr(tasks, "classify_with_cache", services.classify_image_bytes)
    monkeypatch.setattr(router_async.classify_job, "delay", lambda job_id: None)
    with TestClient(app) as client:
        assert client.get("/health").status_code == 200
        assert client.get("/vision/sync/").status_code == 200
        assert client.get("/vision/async/").status_code == 200
        assert client.get("/static/style.css").status_code == 200
        valid = client.post("/api/sync/classify/", files={"image": ("a.png", image_bytes(), "image/png")})
        assert valid.status_code == 200
        assert valid.json()["top1"]["label"] == "normal"
        invalid = client.post("/api/sync/classify/", files={"image": ("x.txt", b"bad", "text/plain")})
        assert invalid.status_code == 400
        oversized = client.post("/api/sync/classify/", files={"image": ("big.jpg", b"a" * (5 * 1024 * 1024 + 1), "image/jpeg")})
        assert oversized.status_code == 413
        queued = client.post("/api/async/classify/", files={"image": ("a.png", image_bytes(), "image/png")})
        assert queued.status_code == 202
        job_id = queued.json()["job_id"]
        assert client.get(f"/api/async/jobs/{job_id}/").json()["status"] == "queued"
        tasks.classify_job(job_id)  # 테스트에서는 worker 함수를 직접 호출
        done = client.get(f"/api/async/jobs/{job_id}/").json()
        assert done["status"] == "done"
        assert done["top1"]["label"] == "normal"
        assert client.get("/api/async/jobs/9999999/").status_code == 404


def test_broker_failure(monkeypatch, tmp_path):
    monkeypatch.setattr(router_async, "MEDIA_DIR", tmp_path / "jobs")
    monkeypatch.setattr(router_async.classify_job, "delay", lambda job_id: (_ for _ in ()).throw(ConnectionError("offline")))
    with TestClient(app) as client:
        response = client.post("/api/async/classify/", files={"image": ("a.png", image_bytes(), "image/png")})
        assert response.status_code == 503


def test_cache_hit_avoids_repeated_inference(monkeypatch):
    import json
    import ai.cache_service as cache_service
    store = {}
    class FakeRedis:
        def get(self, key):
            return store.get(key)
        def setex(self, key, seconds, value):
            assert seconds == 600
            store[key] = value
    calls = []
    def fake_inference(data):
        calls.append(data)
        return {"model_id": "test-model", "top1": {"label": "normal", "score": 1.0},
                "all": [{"label": "normal", "score": 1.0}], "inference_ms": 8.0, "cache_hit": False}
    monkeypatch.setattr(cache_service, "get_redis", lambda: FakeRedis())
    monkeypatch.setattr(cache_service, "classify_image_bytes", fake_inference)
    first = cache_service.classify_with_cache(b"same image bytes")
    second = cache_service.classify_with_cache(b"same image bytes")
    assert first["cache_hit"] is False
    assert second["cache_hit"] is True
    assert second["inference_ms"] is None
    assert len(calls) == 1


def test_worker_failure_records_failed(monkeypatch, tmp_path):
    import pytest
    monkeypatch.setattr(router_async, "MEDIA_DIR", tmp_path / "jobs")
    monkeypatch.setattr(tasks, "MEDIA_DIR", tmp_path / "jobs")
    monkeypatch.setattr(router_async.classify_job, "delay", lambda job_id: None)
    with TestClient(app) as client:
        queued = client.post("/api/async/classify/", files={"image": ("a.png", image_bytes(), "image/png")})
        assert queued.status_code == 202
        job_id = queued.json()["job_id"]
        def failed(data):
            raise RuntimeError("fake model crashed")
        monkeypatch.setattr(tasks, "classify_with_cache", failed)
        with pytest.raises(RuntimeError):
            tasks.classify_job(job_id)
        state = client.get(f"/api/async/jobs/{job_id}/").json()
        assert state["status"] == "failed"
        assert "fake model crashed" in state["error_message"]
```

```bash
pytest -q tests/test_api.py
```

### 실환경 장애 점검

| 증상 | 우선 확인 |
|---|---|
| `docker: command not found` | Docker Desktop WSL Integration 설정·WSL 터미널 재실행 |
| `Cannot connect to the Docker daemon` | Docker Desktop Engine이 실행 중인지 |
| `address already in use :6379` | 기존 Redis가 같은 호스트 포트를 사용 중인지 |
| `database connection refused :5432` | PostgreSQL 컨테이너 실행 및 포트 노출 |
| POST async → `503` | Redis 브로커 상태와 Celery 주소 설정 |
| 계속 `queued` | Worker 프로세스 실행 및 같은 Broker DB 사용 여부 |
| `failed` | Celery 터미널에서 자세한 예외 및 DB error_message 확인 |
| `'model ... not found'` | `.env` MODEL_PATH와 Fine-tuning 결과 경로, 서버·워커 재시작 |
| 모델 결과가 옛 클래스 | `.env`를 바꾼 후 두 프로세스 모두 재시작했는지 |
| PostgreSQL 테이블이 안 보임 | `uvicorn main:app` 시작 후 DBeaver 새로고침 |

## 15. 수업 종료와 다시 시작하는 순서

### 종료

```text
터미널 C: Ctrl+C (FastAPI 종료)
터미널 B: Ctrl+C (Celery Worker 종료)
터미널 A: docker compose stop (Redis/PostgreSQL 컨테이너 정지)
```

### 다음 날 재시작

```bash
cd ~/fastapi_resnet18
docker compose start
docker compose exec redis redis-cli ping
# 새 터미널: source .venv/bin/activate && celery -A ai.celery_app:celery_app worker --loglevel=info --pool=solo --concurrency=1
# 새 터미널: source .venv/bin/activate && uvicorn main:app --reload
```

PostgreSQL 데이터는 Docker named volume에 남습니다. 이미지 파일은 `media/jobs` 폴더에, 모델 파일은 `artifacts/cap_quality`에 남습니다. 프로젝트를 삭제하면 이 파일들은 함께 삭제될 수 있으니 별도 백업합니다.

## 16. 미니 챌린지 및 최종 체크리스트

학생이 직접 학습한 `normal/scratch/crack` 분류 모델을 `MODEL_PATH`로 지정하고, 같은 이미지를 동기와 비동기로 각각 3회 전송해 봅니다. 첫 회(CACHE MISS)와 반복(CACHE HIT)의 결과 및 응답시간을 기록하고 DBeaver에서 Job의 상태와 점수를 확인합니다. 이후 Celery Worker를 중지한 상태에서 요청을 보내면 `queued`가 유지되는지, 다시 실행했을 때 `done`으로 변경되는지 실험합니다.

**완료 점검:** [ ] Docker 최초 설치 [ ] WSL Integration [ ] `PONG` [ ] PostgreSQL ready [ ] DBeaver 연결 [ ] Worker 준비 [ ] FastAPI 실행 [ ] Sync 응답 [ ] Async 202+job_id [ ] queued→running→done [ ] Redis 캐시 반복 요청 [ ] 모델 재학습 경로 동일 [ ] 측정 조건 기록.

## 수업 참고자료

- Hugging Face ResNet-18: https://huggingface.co/microsoft/resnet-18
- Transformers 이미지 분류 Fine-tuning: https://huggingface.co/docs/transformers/tasks/image_classification
- FastAPI File Upload: https://fastapi.tiangolo.com/tutorial/request-files/
- Docker Desktop Windows 설치: https://docs.docker.com/desktop/setup/install/windows-install/
- Docker Compose: https://docs.docker.com/compose/
- Celery Redis Broker: https://docs.celeryq.dev/en/stable/getting-started/backends-and-brokers/redis.html
- SQLAlchemy ORM: https://docs.sqlalchemy.org/en/20/orm/quickstart.html

> **검증 범위:** Python 문법 및 FastAPI API·DB/상태 흐름은 모의 테스트로 검증합니다. Docker/Redis/Celery 실환경 작업 큐와 실제 Hugging Face 가중치 다운로드·성능 측정은 학생 PC에서 추가 확인해야 하며, 별도 실행 검증 없이 완료되었다고 주장하지 않습니다.
