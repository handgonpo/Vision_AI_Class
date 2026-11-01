
## 0. 오늘 수업의 목표

1. YOLO 객체검출과 ResNet 이미지 분류의 차이를 이해하고 Hugging Face 모델 라이선스를 점검합니다.
2. 원본 ResNet-18로 ImageNet 분류 결과를 확인합니다.
3. 내 데이터 폴더로 `normal / scratch / crack` 클래스 **Fine-tuning**을 수행합니다.
4. Fine-tuning한 모델을 `MODEL_PATH`로 지정하여 FastAPI에서 이미지 파일을 받아 JSON 및 HTML로 결과를 제공합니다.
5. 파일별 역할, 의사코드, 직접 실행·확인, 학습 데이터 누출 방지를 익힙니다.

![[Pasted image 20261101121940.png]]

### 한눈에 보는 처리 흐름

![[Pasted image 20261101120812.png]]

```text
로컬 이미지(JPG/PNG/WEBP)
        ↓ 브라우저 / Insomnia multipart/form-data
FastAPI /api/sync/classify/ (요청 처리)
        ↓ ai/services.py (이미지 검증 및 RGB 변환)
        ↓ ai/hf_model.py (프로세스별 모델 1회 로딩)
Hugging Face ResNet-18 또는 내 Fine-tuning 모델
        ↓ label / score / inference_ms
FastAPI JSON 응답 → HTML 화면에 표시
```

## 1. 기존 YOLO 수업과 오늘의 차이

| 기존 YOLO | 오늘 ResNet-18 |
|---|---|
| 한 이미지에서 여러 객체·위치 탐지 | 이미지 전체를 대표하는 단일 클래스 분류 |
| bounding box, class, confidence | label, score |
| 필요할 경우 YOLO 모델 Fine-tuning | ResNet 분류 헤드 교체 및 Fine-tuning |
| 산업장면에서 객체 위치까지 필요한 문제 | 대상이 중심에 있는 부품의 상태·종류 구분 문제 |

`microsoft/resnet-18` 모델 카드에는 Apache-2.0 라이선스, 1,000개 ImageNet 클래스, Hugging Face Transformers 추론 예제가 안내되어 있습니다. <https://huggingface.co/microsoft/resnet-18>.

## 2. 프로젝트 준비: 최초에는 Docker가 필요하지 않습니다

먼저 FastAPI 동기식 분류만 구현합니다. **이 단계에서 Redis, Docker, Celery가 없어도 정상적으로 동작해야 합니다.** 02단계에서 Docker와 PostgreSQL, Redis를 추가합니다.

WSL2 Ubuntu 터미널에서 다음 명령을 순서대로 실행합니다.

```bash
cd ~
mkdir -p fastapi_resnet18
cd fastapi_resnet18
code -r .

# 이미 uv를 설치했다면
uv venv --python 3.11
source .venv/bin/activate

# uv가 설치되어 있지 않다면 대신:
# python3.11 -m venv .venv
# source .venv/bin/activate
# python -m pip install -r requirements.txt
```

**중요:** 아직 `requirements.txt`를 만들지 않았다면 아래 3단계에서 먼저 파일을 작성한 뒤 패키지 설치를 수행합니다. 위의 `python -m pip install`은 대체 실행 방법을 설명하는 주석일 뿐 지금 바로 실행하는 명령이 아닙니다.

권장 Python은 3.11~3.12입니다. PyTorch, Transformers 설치는 인터넷 연결과 충분한 디스크 공간이 필요합니다.

```bash
# 프로젝트 루트에서 폴더만 먼저 생성
mkdir -p ai training templates static dataset/train dataset/val artifacts
# init 파일은 반드시 만드는 것은 아니지만 패키지 구분을 위해 만듭니다.
touch ai/__init__.py
```

### 최종 폴더 구조 (01 완성 시점)

```text
fastapi_resnet18/
├── .venv/
├── .env                 # MODEL_PATH (직접 만들기)
├── requirements.txt
├── main.py
├── test_api.py
├── ai/
│   ├── __init__.py
│   ├── config.py
│   ├── hf_model.py
│   ├── services.py
│   ├── router_sync.py
│   └── router_web.py
├── training/
│   └── finetune.py
├── templates/
│   ├── base.html
│   └── sync.html
├── static/style.css
├── dataset/
│   ├── train/{normal,scratch,crack}/
│   └── val/{normal,scratch,crack}/
└── artifacts/cap_quality/   # 학습 후 생성됨
```

> 디렉토리에 적힌 `{normal,scratch,crack}`은 실제 생성할 하위 폴더들의 이름을 나타낸 것입니다. 아직 `main.py`를 실행하지 마세요. 파일을 모두 만든 뒤 마지막에 서버를 실행합니다. 이전 교재처럼 존재하지 않는 라우터를 먼저 import하지 않기 위함입니다.

## 3. 패키지 설치

`requirements.txt`를 생성한 뒤 전체 내용을 입력합니다. FastAPI는 웹 요청을, Transformers와 PyTorch는 모델 추론·학습을, Pillow는 이미지를 처리합니다. `python-multipart`는 업로드 폼 처리를 위해 필요합니다.

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
httpx>=0.27,<1
pytest>=8,<10
python-dotenv>=1,<2
```

```bash
uv pip install -r requirements.txt
# uv를 사용하지 않는다면: python -m pip install -r requirements.txt
python -c "import fastapi, PIL, torch, transformers; print('패키지 import OK')"
```

여기서 설치 명령이 끝나지 않으면 다음 단계로 넘어가지 않습니다. GPU 사용 여부는 `python -c "import torch; print(torch.cuda.is_available())"`로 확인합니다. CPU로도 실행할 수 있지만 학습과 추론은 느릴 수 있습니다.

## 4. 모델과 프로젝트 설정

`MODEL_PATH`는 **처음에는 원본 Hugging Face 모델 ID**이며 Fine-tuning 후에는 **직접 저장한 로컬 경로**로 바꿉니다. 모델 변경 후 이미 실행 중인 Uvicorn/Worker 프로세스는 재시작해야 합니다.

**코드를 작성하기 전 의사코드**

```text
설정파일에서 MODEL_PATH 읽기 → 없으면 공식 사전학습 모델 ID 사용 → 다른 함수에서 동일한 설정 공유
```

**`ai/config.py` — 전체 코드**

```python
import os
from pathlib import Path
from dotenv import load_dotenv

BASE_DIR = Path(__file__).resolve().parent.parent
load_dotenv(BASE_DIR / ".env")
MODEL_PATH = os.getenv("MODEL_PATH", "microsoft/resnet-18")
MAX_UPLOAD_BYTES = 5 * 1024 * 1024
MAX_IMAGE_PIXELS = 12_000_000
```

**`.env.example` — 전체 코드**

```bash
MODEL_PATH=microsoft/resnet-18
# Fine-tuning 이후: MODEL_PATH=artifacts/cap_quality
```

`.env.example` 내용을 복사해 `.env`를 만들고 처음에는 `MODEL_PATH=microsoft/resnet-18`로 둡니다.

```bash
cp .env.example .env
```

#### 코드가 모델을 매번 다운로드하지 않게 하려면?

`hf_model.py`는 같은 프로세스에서 `get_classifier()` 최초 호출 때 `pipeline`을 생성하고 캐시합니다. **모델 파일은 Hugging Face 로컬 캐시에 남지만, 서버 프로세스가 다시 시작되면 모델은 메모리에 다시 로드됩니다.**

**`ai/hf_model.py` — 전체 코드**

```python
from functools import lru_cache

from .config import MODEL_PATH

@lru_cache(maxsize=1)
def get_classifier():
    """서버/워커 프로세스별 최초 요청 때 모델을 로드하고 재사용한다."""
    from transformers import pipeline

    return pipeline("image-classification", model=MODEL_PATH, device=-1)
```

## 5. 이미지 입력 검증과 AI 결과 정리 — services.py

DRF에서는 Serializer와 View 안에 이미지 처리 로직이 뒤섞여 있었습니다. FastAPI에서는 HTTP 입력 검증은 Router에서, 이미지 디코딩과 추론은 서비스 함수에서 처리합니다.

**코드를 작성하기 전 의사코드**

```text
1. 이미지 크기/파일형식 확인 → 2. Pillow로 열고 RGB 변환 → 3. ResNet 모델 불러오기 → 4. 추론 → 5. 점수 상위 5개 정리 → 6. JSON용 dict 반환
```

**`ai/services.py` — 전체 코드**

```python
import hashlib
import io
import time
from PIL import Image, ImageOps, UnidentifiedImageError
from .config import MAX_IMAGE_PIXELS, MAX_UPLOAD_BYTES, MODEL_PATH
from .hf_model import get_classifier

class InvalidImage(ValueError):
    pass


def load_rgb_image(image_bytes: bytes) -> Image.Image:
    """크기와 디코딩을 검증한 뒤 RGB 이미지를 반환한다."""
    if not image_bytes or len(image_bytes) > MAX_UPLOAD_BYTES:
        raise InvalidImage("이미지 파일은 1바이트 이상, 5MB 이하이어야 합니다.")
    try:
        with Image.open(io.BytesIO(image_bytes)) as source:
            if source.format not in {"JPEG", "PNG", "WEBP"}:
                raise InvalidImage("JPG, PNG, WEBP 이미지만 허용합니다.")
            if source.width * source.height > MAX_IMAGE_PIXELS:
                raise InvalidImage("이미지 해상도가 너무 큽니다(최대 1200만 픽셀).")
            image = ImageOps.exif_transpose(source).convert("RGB")
            image.load()
            return image
    except (UnidentifiedImageError, OSError, Image.DecompressionBombError) as exc:
        raise InvalidImage("올바른 이미지 파일이 아닙니다.") from exc


def classify_image_bytes(image_bytes: bytes) -> dict:
    """모델 추론만 담당한다. Redis 연동은 cache_service.py로 확장한다."""
    image = load_rgb_image(image_bytes)
    started = time.perf_counter()
    raw = get_classifier()(image, top_k=5)
    # Pipeline 버전에 따라 리스트가 한 번 더 감싸질 수 있으므로 정리한다.
    while isinstance(raw, list) and len(raw) == 1 and isinstance(raw[0], list):
        raw = raw[0]
    if not isinstance(raw, list) or not raw:
        raise RuntimeError("이미지 분류 모델이 결과를 반환하지 않았습니다.")
    items = [{"label": str(x["label"]), "score": float(x["score"])} for x in raw]
    items.sort(key=lambda x: x["score"], reverse=True)
    return {
        "model_id": MODEL_PATH,
        "top1": items[0],
        "all": items,
        "inference_ms": round((time.perf_counter() - started) * 1000, 2),
        "cache_hit": False,
    }


def cache_key(image_bytes: bytes) -> str:
    digest = hashlib.sha256(image_bytes).hexdigest()
    model = hashlib.sha256(MODEL_PATH.encode("utf-8")).hexdigest()[:16]
    return f"vision:classify:{model}:{digest}"
```

**핵심 코드 읽기:** `Image.open(io.BytesIO(...))`로 파일 내용을 메모리에서 열고, `convert('RGB')`로 입력 채널을 통일합니다. `get_classifier()(image, top_k=5)`에서 `get_classifier()`는 모델 객체를 반환하고 뒤의 `(image, ...)`가 실제 추론입니다. `top_k=5`는 상위 5개만 보여주는 설정이므로 1,000개 클래스 전체 점수는 아닙니다.

원본 실습의 `top1`과 `all`을 유지했습니다. 여기서 `all`은 '요청한 상위 점수 목록'이지 모델의 모든 1,000개 클래스가 아닙니다.

## 6. FastAPI 라우터 만들기 — JSON API

FastAPI에서는 Django `APIView`/`Serializer` 대신 `@router.post()`, `UploadFile`, `File`, `HTTPException`을 사용합니다. 서버에 이미지를 직접 업로드하면 결과 JSON을 돌려줍니다.

**코드를 작성하기 전 의사코드**

```text
이미지 업로드 요청 → 바이트 읽기 → 5MB 초과 시 413 → 이미지 포맷 오류 시 400 → 서비스 호출 → dict 자동 JSON 응답
```

**`ai/router_sync.py` — 전체 코드**

```python
from fastapi import APIRouter, File, HTTPException, UploadFile
from .config import MAX_UPLOAD_BYTES
from .services import InvalidImage
from .services import classify_image_bytes

router = APIRouter(tags=["동기 추론"])

@router.post("/api/sync/classify/")
def classify_sync(image: UploadFile = File(...)):
    """HTTP 요청이 추론이 끝날 때까지 기다린다."""
    image_bytes = image.file.read(MAX_UPLOAD_BYTES + 1)
    if len(image_bytes) > MAX_UPLOAD_BYTES:
        raise HTTPException(status_code=413, detail="5MB 이하 파일만 허용합니다.")
    try:
        return classify_image_bytes(image_bytes)
    except InvalidImage as exc:
        raise HTTPException(status_code=400, detail=str(exc)) from exc
```

API 주소는 **`POST /api/sync/classify/`**입니다. Insomnia에서는 `Multipart Form`에 이름을 `image`로 지정하고 JPEG/PNG/WebP 파일 하나를 첨부합니다. `/docs`에서도 `Try it out`으로 테스트할 수 있습니다.

## 7. Jinja2 화면 라우터

HTML 페이지는 데이터를 등록하지 않고 화면만 렌더링합니다. 실제 분류는 브라우저 JavaScript가 위 JSON API에 요청합니다.

**코드를 작성하기 전 의사코드**

```text
GET /vision/sync/ → HTML 페이지 표시 → JS fetch() → POST /api/sync/classify/ → 결과 JSON을 화면에 출력
```

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
```

## 8. HTML 템플릿과 CSS 작성

FastAPI는 Django 템플릿 태그인 `{% load static %}`를 사용하지 않습니다. Jinja2의 `url_for()`로 정적 파일 URL을 만듭니다. `sync.html`은 이미지를 미리 보고 서버 응답시간을 표시합니다.

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
   <nav><a href="/vision/sync/">동기</a> · <a href="/docs">Swagger UI</a></nav>
  </header>
  {% block content %}{% endblock %}
 </main>
</body>
</html>
```

**`templates/sync.html` — 전체 코드**

```html
{% extends "base.html" %}
{% block title %}동기식 이미지 분류{% endblock %}
{% block content %}
<section class="panel"><h2>동기식: 분석을 마칠 때까지 응답을 기다립니다</h2>
<p>원본 모델은 ImageNet 사물 분류용입니다. 정상/불량 분류는 Fine-tuning 후에만 사용할 수 있습니다.</p>
<input id="image" type="file" accept="image/jpeg,image/png,image/webp">
<button id="send">동기 분류</button>
<img id="preview" class="preview" hidden>
<p id="state">이미지를 선택하세요.</p><pre id="output">{}</pre></section>
<script>
const fileEl=document.getElementById('image');
fileEl.onchange=()=>{const f=fileEl.files[0];if(f){const p=document.getElementById('preview');p.src=URL.createObjectURL(f);p.hidden=false;}};
document.getElementById('send').onclick=async()=>{
 const file=fileEl.files[0];if(!file){alert('이미지를 선택하세요');return;}
 const button=document.getElementById('send');button.disabled=true;
 const state=document.getElementById('state');const out=document.getElementById('output');
 const form=new FormData();form.append('image',file);const started=performance.now();
 state.textContent='서버에서 추론 중...';
 try{const res=await fetch('/api/sync/classify/',{method:'POST',body:form});const data=await res.json();
  state.textContent=`HTTP ${res.status} / 응답까지 ${Math.round(performance.now()-started)}ms`;
  out.textContent=JSON.stringify(data,null,2);
 }catch(err){state.textContent=`요청 실패: ${err}`;}finally{button.disabled=false;}
};
</script>
{% endblock %}
```

**`static/style.css` — 전체 코드**

```css
*{box-sizing:border-box}body{margin:0;background:#0d1726;color:#edf4ff;font:16px/1.6 system-ui,-apple-system,sans-serif}.wrap{max-width:850px;margin:38px auto;padding:0 20px}header{margin-bottom:26px}nav a{color:#9bc5ff;margin-right:12px}.panel{padding:24px;border:1px solid #364961;border-radius:16px;background:#162338}.panel h2{font-size:21px}input,button{margin:8px 8px 8px 0;padding:10px;border-radius:8px}button{background:#78aafa;color:#0a1521;border:0;cursor:pointer;font-weight:700}button:disabled{opacity:.5}pre{white-space:pre-wrap;overflow-wrap:anywhere;background:#07101b;border-radius:10px;padding:16px}.preview{display:block;max-width:300px;max-height:250px;margin:12px 0;object-fit:contain}.preview[hidden]{display:none}
```

## 9. main.py를 마지막에 작성하여 서버 실행

지금까지 **`router_sync.py`, `router_web.py`, HTML, CSS를 모두 만들었습니다.** 따라서 마지막에 `main.py`에서 import해도 존재하지 않는 모듈 오류가 없습니다.

**코드를 작성하기 전 의사코드**

```text
FastAPI app 생성 → /static mount → 화면 라우터 등록 → 동기 분류 라우터 등록 → health 등록 → Uvicorn 실행
```

**`main.py` — 전체 코드**

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from ai.config import BASE_DIR
from ai.router_web import router as web_router
from ai.router_sync import router as sync_router

app = FastAPI(title="ResNet-18 Image Classification")
app.mount("/static", StaticFiles(directory=str(BASE_DIR / "static")), name="static")
app.include_router(web_router)
app.include_router(sync_router)

@app.get("/health")
def health():
    return {"status": "ok"}
```

### 순서대로 실행

```bash
# 프로젝트 루트, 가상환경 활성화 상태
uvicorn main:app --reload
```

브라우저에서 `http://127.0.0.1:8000/health` → `{"status":"ok"}`, `http://127.0.0.1:8000/docs` → Swagger UI, `http://127.0.0.1:8000/vision/sync/` → 이미지 업로드 화면을 확인합니다. 모델 추론은 최초 클릭 시 Hugging Face 다운로드가 발생할 수 있습니다. `Ctrl+C`를 눌러 서버를 종료합니다.

#### 코드 실행 결과 예시 (숫자는 예시일 뿐입니다)

```json
{
  "model_id": "microsoft/resnet-18",
  "top1": {"label": "tiger cat", "score": 0.71},
  "all": [{"label": "tiger cat", "score": 0.71}],
  "inference_ms": 85.5,
  "cache_hit": false
}
```

실제 `all`에는 최대 5개의 예측이 포함됩니다. 점수는 입력 이미지와 환경에 따라 달라지며 보정된 신뢰확률이라고 단정하면 안 됩니다.

### curl UI API 테스트

```bash
curl -X POST -F "image=@samples/example.jpg" http://127.0.0.1:8000/api/sync/classify/
```

`curl`에서 `samples/example.jpg`는 학생이 직접 준비한 파일의 경로로 바꿉니다.

## 10. Fine-tuning을 위한 데이터 준비

이제 **원본 모델로 분류하는 실습을 마쳤으므로** 우리만의 제조 품질 분류 클래스를 만듭니다. 예시는 병뚜껑을 촬영한 사진을 `normal`, `scratch`, `crack` 세 범주로 분류하는 프로젝트입니다. 분류를 위해서는 각 이미지에 정답 라벨이 필요합니다. **원본 ResNet의 1,000개 클래스에 3개를 그냥 더하는 것이 아니라, 이번 실습에서는 1,000개 출력층을 3개 출력층으로 교체합니다.**

```text
dataset/
├── train/
│   ├── normal/      # 정상 사진
│   ├── scratch/     # 스크래치 사진
│   └── crack/       # 균열 사진
└── val/
    ├── normal/
    ├── scratch/
    └── crack/
```

```bash
mkdir -p dataset/{train,val}/{normal,scratch,crack}
```

먼저 개인 실습용으로 클래스당 Train 최소 2장/Val 최소 1장이 있어야 코드가 실행되지만, 이 수량으로 유효한 산업용 모델을 만들 수는 없습니다. 비교 가능한 성능을 확인하려면 **촬영 환경 다양화, 클래스 균형, 충분한 표본, 별도 Test Set, 같은 원본 제품의 Train/Val 누출 방지**가 필요합니다. 공개 데이터셋을 사용할 경우 다운로드·학습·배포 권한을 따로 확인합니다.

**왜 처음부터 병뚜껑을 직접 찍을까요?** 학생이 이미 YOLO에서 ROI, 결함 종류를 배웠기 때문에 물체를 촬영한 한 장을 입력해 분류하는 작업으로 전환하기 쉽습니다. 결함 위치 검출은 별도 작업입니다.

## 11. Fine-tuning 학습 코드를 직접 작성합니다 — training/finetune.py

이 코드는 Hugging Face에서 ResNet 가중치를 다운로드하고, ImageNet 출력층을 3개 클래스 출력층으로 교체합니다. `--freeze-backbone` 옵션을 켜면 먼저 기존 특성 추출 층을 고정하고 **마지막 분류 헤드만 학습**합니다. 옵션을 빼면 전체 신경망이 Fine-tuning됩니다.

**코드를 작성하기 전 의사코드**

```text
클래스 폴더 읽기 → 데이터셋 생성 → 이미지 프로세서 불러오기 → 새 분류 헤드로 모델 로딩 → 반복 학습 → 검증 정확도 계산 → 가장 좋은 모델·프로세서 저장
```

**`training/finetune.py` — 전체 코드**

```python
"""예: python -m training.finetune --data dataset --epochs 3 --output artifacts/cap_quality"""
import argparse
import json
import random
from pathlib import Path
import torch
from torch.utils.data import Dataset, DataLoader
from PIL import Image
from transformers import AutoImageProcessor, AutoModelForImageClassification

BASE_MODEL = "microsoft/resnet-18"
EXTS = {".jpg", ".jpeg", ".png", ".webp"}

class FolderImages(Dataset):
    def __init__(self, root: Path, classes: list[str], processor):
        self.records = []
        self.processor = processor
        for idx, name in enumerate(classes):
            folder = root / name
            if not folder.is_dir():
                raise ValueError(f"필요한 클래스 폴더가 없습니다: {folder}")
            self.records.extend((path, idx) for path in sorted(folder.iterdir())
                                if path.suffix.lower() in EXTS and path.is_file())
        if not self.records:
            raise ValueError(f"이미지가 없습니다: {root}")
    def __len__(self):
        return len(self.records)
    def __getitem__(self, index):
        path, label = self.records[index]
        with Image.open(path) as raw:
            image = raw.convert("RGB")
            tensor = self.processor(images=image, return_tensors="pt")["pixel_values"][0]
        return {"pixel_values": tensor, "labels": torch.tensor(label, dtype=torch.long)}

def run(args):
    random.seed(42); torch.manual_seed(42)
    data = Path(args.data)
    train_root = data / "train"
    if not train_root.is_dir():
        raise ValueError("dataset/train/ 폴더를 먼저 준비하세요.")
    classes = sorted(p.name for p in train_root.iterdir() if p.is_dir())
    if len(classes) < 2:
        raise ValueError("클래스 폴더가 2개 이상 필요합니다.")
    processor = AutoImageProcessor.from_pretrained(BASE_MODEL)
    train_set = FolderImages(train_root, classes, processor)
    val_set = FolderImages(data / "val", classes, processor)
    train_counts = {n: sum(label == i for _, label in train_set.records) for i, n in enumerate(classes)}
    val_counts = {n: sum(label == i for _, label in val_set.records) for i, n in enumerate(classes)}
    if any(n < 2 for n in train_counts.values()) or any(n < 1 for n in val_counts.values()):
        raise ValueError("각 클래스에 학습 최소 2장, 검증 최소 1장이 필요합니다.")
    print(f"클래스: {classes} / Train: {len(train_set)}, Val: {len(val_set)}")
    model = AutoModelForImageClassification.from_pretrained(
        BASE_MODEL,
        num_labels=len(classes),
        id2label={i: name for i, name in enumerate(classes)},
        label2id={name: i for i, name in enumerate(classes)},
        ignore_mismatched_sizes=True,
    )
    # 기존 1000-class 분류 헤드는 크기가 맞지 않아 새롭게 초기화됩니다.
    # 초보자 첫 학습은 Backbone을 고정하고 마지막 분류 헤드만 학습합니다.
    if args.freeze_backbone:
        for name, parameter in model.named_parameters():
            parameter.requires_grad = name.startswith("classifier.")
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    model.to(device)
    optim = torch.optim.AdamW((p for p in model.parameters() if p.requires_grad), lr=args.lr)
    train_loader = DataLoader(train_set, batch_size=args.batch_size, shuffle=True, num_workers=0)
    val_loader = DataLoader(val_set, batch_size=args.batch_size, shuffle=False, num_workers=0)
    history=[]
    best_acc=-1.0
    output = Path(args.output)
    output.mkdir(parents=True, exist_ok=True)
    for epoch in range(args.epochs):
        model.train(); losses=[]
        for batch in train_loader:
            batch = {k: v.to(device) for k, v in batch.items()}
            optim.zero_grad()
            result = model(**batch)
            result.loss.backward()
            optim.step()
            losses.append(float(result.loss.item()))
        model.eval(); correct=0; total=0
        with torch.no_grad():
            for batch in val_loader:
                pixels=batch["pixel_values"].to(device)
                logits=model(pixel_values=pixels).logits
                correct += (logits.argmax(dim=-1).cpu() == batch["labels"]).sum().item()
                total += len(batch["labels"])
        acc=correct/total
        row={"epoch":epoch+1,"loss":round(sum(losses)/len(losses),4),"val_accuracy":round(acc,4)}
        history.append(row);print(row)
        if acc > best_acc:
            best_acc=acc
            model.save_pretrained(str(output), safe_serialization=True)
            processor.save_pretrained(str(output))
    (output / "training_report.json").write_text(json.dumps({"base_model":BASE_MODEL,
        "classes":classes,"train_count":len(train_set),"val_count":len(val_set),
        "best_val_accuracy":best_acc,"history":history},ensure_ascii=False,indent=2),encoding="utf-8")
    print(f"저장 완료: {output} / best val accuracy={best_acc:.4f}")

if __name__ == "__main__":
    parser=argparse.ArgumentParser()
    parser.add_argument("--data",default="dataset")
    parser.add_argument("--output",default="artifacts/cap_quality")
    parser.add_argument("--epochs",type=int,default=3)
    parser.add_argument("--batch-size",type=int,default=8)
    parser.add_argument("--lr",type=float,default=2e-4)
    parser.add_argument("--freeze-backbone",action="store_true")
    run(parser.parse_args())
```

### 첫 실습 실행

```bash
# FastAPI 서버 종료 후 동일 프로젝트 루트에서 실행
python -m training.finetune   --data dataset   --output artifacts/cap_quality   --epochs 3   --batch-size 8   --freeze-backbone
```

학습 완료 후 `artifacts/cap_quality/`에는 모델 가중치, 설정 파일, 전처리기 및 `training_report.json`이 저장됩니다. **학습이 실제로 성공했는지, val accuracy가 의미 있게 나왔는지 확인하고 넘어갑니다.**

### Fine-tuning 모델로 FastAPI 서빙 전환

`.env`를 다음과 같이 수정합니다.

```dotenv
MODEL_PATH=artifacts/cap_quality
```

```bash
# 기존 Uvicorn 서버가 있다면 Ctrl+C 후 재실행
uvicorn main:app --reload
```

서버와 Celery 워커는 프로세스가 다르므로 02 단계에서는 `.env`를 바꾼 후 **둘 다 재시작**해야 같은 Fine-tuning 모델을 사용합니다.

### 꼭 비교할 성능

| 항목 | 원본 ResNet-18 | 내 Fine-tuning 모델 |
|---|---|---|
| 클래스 | ImageNet 1,000개 | normal/scratch/crack 3개 |
| 분류 목적 | 일반 사물 종류 | 특정 제조 샘플의 상태 분류 |
| 평가 | 원본 모델의 공식 ImageNet 평가 | 내 동일한 Test Set에서 Accuracy, 클래스별 Precision/Recall/F1 |

원본 모델과 Fine-tuning 모델의 Accuracy를 **서로 다른 클래스·데이터셋끼리 직접 비교하면 안 됩니다.** 똑같은 세 클래스 문제를 풀도록 맞춘 기준선 모델과 비교해야 개선 효과를 주장할 수 있습니다.

## 12. 테스트·예외 처리와 실습 검증

학습 코드를 작성하는 것만큼 **이미지가 없거나 잘못되었을 때 400을 반환하는지** 확인하는 것이 중요합니다. 아래 테스트는 실제 HF 모델 다운로드 대신 가짜 분류 결과를 반환하도록 바꿔 HTTP 라우팅, 이미지 처리·응답 구조를 확인합니다.

**`test_api.py` — 전체 코드**

```python
from io import BytesIO
from PIL import Image
from fastapi.testclient import TestClient
import ai.services as services
from main import app

def test_sync(monkeypatch):
    monkeypatch.setattr(services, 'get_classifier', lambda: (lambda image, top_k=5:
        [{'label':'object','score':0.9}]))
    buf=BytesIO(); Image.new('RGB',(32,32),'blue').save(buf,format='PNG')
    with TestClient(app) as c:
        assert c.get('/health').status_code==200
        assert c.get('/vision/sync/').status_code==200
        assert c.get('/static/style.css').status_code==200
        r=c.post('/api/sync/classify/',files={'image':('test.png',buf.getvalue(),'image/png')})
        assert r.status_code==200 and r.json()['top1']['label']=='object'
        assert c.post('/api/sync/classify/',files={'image':('bad.txt',b'bad','text/plain')}).status_code==400
```

```bash
pytest -q test_api.py
```

**검증 구분:** 위 테스트로 FastAPI 라우팅·업로드 응답과 예외를 점검할 수 있지만, ResNet 모델을 실제로 다운로드하여 inference한 검증이나 Fine-tuning 실제 품질을 증명하지는 못합니다.

