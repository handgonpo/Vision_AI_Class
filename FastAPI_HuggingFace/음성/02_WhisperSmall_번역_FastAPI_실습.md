## 오늘의 학습 목표

1. WSL2 Ubuntu에 ffmpeg와 Python 가상환경을 준비합니다.
2. Whisper와 번역 모델을 한 번만 로딩하도록 작성합니다.
3. FastAPI `UploadFile`/`Form`을 사용해 WAV·MP3 파일을 받습니다.
4. STT 및 번역 결과를 JSON으로 반환합니다.
5. Insomnia와 curl로 성공·실패 요청을 비교합니다.

## 오늘 수업의 전체 흐름

```text
프로젝트 폴더 → 가상환경 → ffmpeg 설치
    ↓
speech/services.py (모델 준비·디코딩·STT·번역)
    ↓
speech/router.py (POST 파일 업로드)
    ↓
main.py (FastAPI 앱·라우터 연결)
    ↓
python -m pytest -q → uvicorn 실행
    ↓
Insomnia / curl 검증
```

## PART 1. 실습 환경과 폴더 생성

### 1-1. 실행 환경

- Python 3.11~3.12 권장 / WSL2 Ubuntu 기준
- FastAPI + Uvicorn + Transformers + PyTorch + NumPy + ffmpeg
- 01에서 선정한 Whisper와 영어→한국어 번역 모델
- **이 프로젝트에서는 Docker·Redis·Celery를 설치하지 않습니다.** HTTP 업로드 API부터 확인합니다.

### 1-2. Ubuntu에서 프로젝트 생성

```bash
# 이미 가상환경이 활성화되어 있다면 필요할 때만 deactivate
cd ~
mkdir -p fastapi_speech
cd fastapi_speech
code -r .
# Ubuntu에서 venv를 만들기 위한 패키지와 오디오 디코더 먼저 설치
sudo apt update
sudo apt install -y python3-venv ffmpeg
python3 --version
python3 -m venv .venv
source .venv/bin/activate
ffmpeg -version
```

`python3-venv`를 먼저 설치하는 이유는 Ubuntu 환경에 따라 `python3 -m venv` 실행에 필요한 패키지가 기본 설치되어 있지 않을 수 있기 때문입니다. `ffmpeg`는 MP3·M4A·WEBM을 Whisper가 처리할 수 있는 16kHz 단일 채널 PCM으로 디코딩합니다. 오디오 길이는 30초 이하로 제한합니다.

### 1-3. requirements.txt 준비

파일을 프로젝트 루트에 만들고 아래 내용을 입력합니다. `torch`는 컴퓨터의 GPU와 CUDA 조합에 따라 별도의 공식 설치 가이드를 확인해야 하며, CPU 추론도 지원합니다.

**`requirements.txt` — 신규 생성**

```text
fastapi>=0.115,<1
uvicorn[standard]>=0.30,<1
python-multipart>=0.0.18,<1
jinja2>=3.1,<4
transformers>=4.49,<5
# torch는 PC의 CPU/CUDA에 맞는 설치가 필요합니다.
torch>=2.3,<3
numpy>=1.26,<3
sentencepiece>=0.2,<1
sacremoses>=0.1,<1
httpx>=0.27,<1
pytest>=8,<10
```

패키지 설치:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -c "import fastapi, numpy, torch, transformers; print('라이브러리 import 성공')"
```

처음 요청 시 Hugging Face 모델 파일이 다운로드됩니다. 인터넷이 연결되어야 하며 Whisper Small과 번역 모델을 함께 로딩하면 CPU 메모리 사용량이 커질 수 있습니다. 모델이 큰 경우 `whisper-base`로 줄여 테스트할 수 있지만, 본 교재의 기본값은 `whisper-small`입니다.

### 1-4. 전체 폴더 구조

```text
fastapi_speech/
├── .venv/                    # 가상환경
├── requirements.txt
├── main.py                  # FastAPI 진입점
├── speech/
│   ├── __init__.py          # 빈 파일
│   ├── services.py          # ffmpeg + Whisper + 번역
│   └── router.py            # POST /api/stt-to-ko/
└── tests/
    └── test_api.py         # 모델 모의 테스트
```

```bash
mkdir -p speech tests
touch speech/__init__.py
```

**주의:** 이 단계에서 `main.py`를 실행하지 않습니다. 아직 `router.py`가 존재하지 않으므로 `include_router`를 호출하면 import 오류가 납니다. 먼저 아래 `services.py`와 `router.py`를 순서대로 만듭니다.

## PART 2. `speech/services.py`를 작성합니다

### 2-1. 이번 파일의 역할

이 파일은 **AI 처리 로직만 담당**합니다. 웹 URL, 업로드 응답, HTML 화면을 작성하지 않습니다.

```text
오디오 파일 경로
   ↓ decode_audio → ffmpeg → 16kHz float32
   ↓ safe_asr_transcribe → Whisper
   ↓ maybe_translate_to_ko → 번역 여부 판단
   ↓ translate_en_to_ko → 번역 모델
   ↓ process_audio_bytes → 응답용 dict
```

기존 교재의 `get_asr`, `get_mt`, `contains_latin`, `looks_like_korean`, `safe_asr_transcribe`, `maybe_translate_to_ko`의 책임을 유지했습니다. 여기에 `decode_audio`를 명확히 분리해 파일 형식별 오류를 줄이고, 임시파일 정리를 한 곳에 모았습니다.

### 2-2. 의사코드

```text
1. 첫 요청이라면 Whisper를 생성하고 모델 객체를 재사용하도록 보관한다.
2. 입력한 파일을 ffmpeg로 변환한다.
3. src_lang이 ko/en이면 Whisper에 해당 언어를 알려준다.
4. Whisper 결과 dict에서 text를 꺼낸다.
5. src_lang=en 또는 auto의 영어 문장이면 번역한다.
6. 번역에 실패하면 status=failed를 반환한다.
7. 응답 dict를 만들어 반환한 뒤 임시 파일을 삭제한다.
```

### 2-3. 코드 전체

**`speech/services.py` — 신규 생성**

```python
"""Hugging Face Whisper STT + Marian EN→KO 번역. 모델은 최초 추론 시에만 로딩."""
import logging
import os
import re
import subprocess
import tempfile
import threading
from pathlib import Path

import numpy as np

logger = logging.getLogger(__name__)
ASR_MODEL_ID = os.getenv("ASR_MODEL_ID", "openai/whisper-small")
MT_MODEL_ID = os.getenv("MT_MODEL_ID", "Helsinki-NLP/opus-mt-tc-big-en-ko")
SAMPLE_RATE = 16000
MAX_AUDIO_SECONDS = 30
_asr = None
_mt_bundle = None
_load_lock = threading.Lock()
_infer_lock = threading.Lock()  # CPU/GPU에서 복수 요청이 모델 자원을 동시에 과도하게 사용하지 않도록 직렬화


def get_asr():
    global _asr
    if _asr is None:
        with _load_lock:
            if _asr is None:
                import torch
                from transformers import pipeline
                device = 0 if torch.cuda.is_available() else -1
                _asr = pipeline("automatic-speech-recognition", model=ASR_MODEL_ID, device=device)
    return _asr


def get_mt():
    """v4/v5에서 통용되는 AutoModel 직접 로딩 방식; 번역 pipeline 별도 의존 제거."""
    global _mt_bundle
    if _mt_bundle is None:
        with _load_lock:
            if _mt_bundle is None:
                import torch
                from transformers import AutoTokenizer, AutoModelForSeq2SeqLM
                tok = AutoTokenizer.from_pretrained(MT_MODEL_ID)
                model = AutoModelForSeq2SeqLM.from_pretrained(MT_MODEL_ID)
                device = torch.device('cuda:0' if torch.cuda.is_available() else 'cpu')
                model.to(device)
                model.eval()
                _mt_bundle = (tok, model, device)
    return _mt_bundle


def decode_audio(path: str) -> np.ndarray:
    """ffmpeg가 여러 입력 포맷을 float32/mono/16k raw PCM으로 통일합니다."""
    try:
        completed = subprocess.run(
            ["ffmpeg", "-nostdin", "-v", "error", "-i", str(path),
             "-t", str(MAX_AUDIO_SECONDS + 1), "-f", "f32le", "-ac", "1", "-ar", str(SAMPLE_RATE), "pipe:1"],
            capture_output=True, timeout=60, check=False,
        )
    except FileNotFoundError as exc:
        raise ValueError("ffmpeg가 설치되지 않았습니다. sudo apt install ffmpeg") from exc
    except subprocess.TimeoutExpired as exc:
        raise ValueError("오디오 디코딩 시간이 초과됐습니다.") from exc
    if completed.returncode != 0:
        raise ValueError("오디오 파일을 해석하지 못했습니다. WAV/MP3/WEBM 등을 확인하세요.")
    values = np.frombuffer(completed.stdout, dtype=np.float32)
    if not len(values):
        raise ValueError("오디오 데이터가 비어 있습니다.")
    if len(values) > SAMPLE_RATE * MAX_AUDIO_SECONDS:
        raise ValueError(f"최대 {MAX_AUDIO_SECONDS}초 음성만 허용합니다.")
    return values


def safe_asr_transcribe(audio_path: str, src_lang: str = 'auto') -> str:
    samples = decode_audio(audio_path)
    options = {'task': 'transcribe'}
    if src_lang == 'ko':
        options['language'] = 'korean'
    elif src_lang == 'en':
        options['language'] = 'english'
    with _infer_lock:
        out = get_asr()({'array': samples, 'sampling_rate': SAMPLE_RATE}, generate_kwargs=options)
    if not isinstance(out, dict) or not isinstance(out.get('text'), str):
        raise RuntimeError('Whisper 출력이 예상된 text 구조가 아닙니다.')
    return out['text'].strip()


def translate_en_to_ko(text: str) -> str:
    import torch
    tok, model, device = get_mt()
    data = tok(text, return_tensors='pt', truncation=True, max_length=256)
    data = {key:val.to(device) for key,val in data.items()}
    with _infer_lock, torch.inference_mode():
        ids = model.generate(**data, max_new_tokens=128)
    return tok.decode(ids[0], skip_special_tokens=True).strip()


def contains_latin(text: str) -> bool:
    return bool(re.search(r'[A-Za-z]', text or ''))


def looks_like_korean(text: str) -> bool:
    return bool(re.search(r'[가-힣]', text or ''))


def maybe_translate_to_ko(src_lang: str, stt_text: str) -> tuple[str, bool, str]:
    """(최종 텍스트, 번역 모델 호출 성공 여부, 상태) 반환."""
    if not stt_text:
        return '', False, 'empty'
    if src_lang == 'ko':
        return stt_text, False, 'skipped'
    should_translate = src_lang == 'en' or (
        src_lang == 'auto' and contains_latin(stt_text) and not looks_like_korean(stt_text)
    )
    if not should_translate:
        return stt_text, False, 'skipped'
    try:
        translated = translate_en_to_ko(stt_text)
        if not translated:
            raise ValueError('번역 결과가 비어 있습니다.')
        return translated, True, 'translated'
    except Exception:
        logger.exception('영어→한국어 번역에 실패했습니다.')
        return stt_text, False, 'failed'


def process_audio_bytes(audio_bytes: bytes, suffix: str, src_lang: str) -> dict:
    """요청 바이트 → 안전한 임시파일 → ASR → 선택적 번역 → JSON 계약."""
    tmp_path = None
    try:
        with tempfile.NamedTemporaryFile(mode='wb', suffix=suffix, delete=False) as f:
            tmp_path = f.name
            f.write(audio_bytes)
        stt_text = safe_asr_transcribe(tmp_path, src_lang)
        ko_text, used, status = maybe_translate_to_ko(src_lang, stt_text)
        return {
            'src_lang': src_lang,
            'stt_text': stt_text,
            'ko_text': ko_text,
            'used_translation': used,
            'translation_status': status,
        }
    finally:
        if tmp_path is not None:
            Path(tmp_path).unlink(missing_ok=True)
```

### 2-4. 주요 함수 이해

| 함수 | 입력 | 반환 | 역할 |
|---|---|---|---|
| `get_asr()` | 없음 | Whisper pipeline 객체 | 최초 1회 모델 로딩 |
| `get_mt()` | 없음 | 토크나이저·번역 모델·device | 최초 1회 번역 모델 로딩 |
| `decode_audio()` | 파일 경로 | `np.ndarray` | ffmpeg로 16kHz 디코딩 |
| `safe_asr_transcribe()` | 파일 경로·언어 | 문자열 | 음성 → 텍스트 |
| `maybe_translate_to_ko()` | 언어·STT 결과 | 텍스트·bool·상태 | 번역 여부 선택 및 예외 처리 |
| `process_audio_bytes()` | bytes·확장자·언어 | dict | 임시파일 저장부터 결과 출력까지 조립 |

**모델 캐시와 Redis 캐시의 차이:** 여기의 `_asr`/`_mt_bundle`은 무거운 모델 객체를 **프로세스 메모리에 재사용**하는 것입니다. Redis는 특정 요청의 계산 **결과를 저장**하는 별도 시스템입니다. 이번 수업에는 Redis가 필요하지 않습니다.

**Whisper의 언어 지정:** `generate_kwargs`에 `task='transcribe'`와 필요한 경우 `language='korean'` 또는 `english`를 전달합니다. 언어 자동 인식이 항상 정확한 것은 아니므로 같은 파일을 `auto`와 `ko/en`으로 비교해 보세요.

**번역 실패의 해석:** 서버가 번역 모델을 다운로드하지 못하거나 추론에 실패하면 `translation_status='failed'`와 영어 원문을 반환합니다. 따라서 영어 텍스트가 나온 것만 보고 한글 번역이 작동했다고 판단하면 안 됩니다.

**단축/오디오 제한:** 이번 코드는 20MB 이하, 30초 이하로 제한합니다. 긴 강의 파일은 별도의 청크 처리와 비동기 작업 큐가 필요합니다.

## PART 3. `speech/router.py`를 생성합니다

### 3-1. API가 하는 일

`POST /api/stt-to-ko/`는 음성 업로드와 `src_lang`을 검사한 후 서비스 함수를 호출합니다. **FastAPI의 `File`/`Form`은 multipart 데이터 추출을 도와주며 DRF Serializer가 아닙니다.** 04에서 검증 로직을 별도 파일로 옮겨 역할을 더 명확히 분리합니다.

### 3-2. 의사코드

```text
1. POST로 audio와 src_lang을 받는다.
2. src_lang이 ko/en/auto인지 검사한다.
3. 허용된 확장자와 20MB 크기를 검사한다.
4. 무거운 AI 계산은 스레드풀에서 호출한다.
5. 계산 결과 dict를 FastAPI가 자동으로 JSON 반환한다.
6. 잘못된 파일은 400, 큰 파일은 413, 서버 모델 오류는 503으로 구분한다.
```

### 3-3. 실제 코드

**`speech/router.py` — 신규 생성**

```python
"""02·03단계: 업로드 검증과 추론 호출을 하나의 FastAPI 라우터에서 확인."""
from pathlib import Path
from fastapi import APIRouter, File, Form, HTTPException, UploadFile
from starlette.concurrency import run_in_threadpool
from . import services

router = APIRouter()
MAX_UPLOAD_BYTES = 20 * 1024 * 1024
ALLOWED_EXTS = {'.wav', '.mp3', '.m4a', '.flac', '.ogg', '.webm', '.mp4'}

@router.post('/api/stt-to-ko/', name='stt_to_ko')
async def stt_to_ko(audio: UploadFile = File(...), src_lang: str = Form('auto')):
    # 1. 입력 검증 (04단계에서는 별도 validators.py로 옮깁니다)
    src_lang = src_lang.strip().lower()
    if src_lang not in {'ko', 'en', 'auto'}:
        raise HTTPException(status_code=400, detail='src_lang은 ko, en, auto 중 하나여야 합니다.')
    ext = Path(audio.filename or '').suffix.lower()
    if ext not in ALLOWED_EXTS:
        raise HTTPException(status_code=400, detail=f'지원하지 않는 확장자: {ext or "(없음)"}')
    # 20MB+1 바이트까지만 읽어서 업로드 폭주를 방지합니다.
    try:
        content = await audio.read(MAX_UPLOAD_BYTES + 1)
    finally:
        await audio.close()
    if not content:
        raise HTTPException(status_code=400, detail='음성 파일이 비어 있습니다.')
    if len(content) > MAX_UPLOAD_BYTES:
        raise HTTPException(status_code=413, detail='20MB 이하 파일만 허용합니다.')
    # 2. 무거운 Whisper 추론은 스레드풀로 보내 이벤트 루프 블로킹을 피합니다.
    try:
        return await run_in_threadpool(services.process_audio_bytes, content, ext, src_lang)
    except ValueError as exc:
        raise HTTPException(status_code=400, detail=str(exc)) from exc
    except Exception as exc:
        # 오류 상세/스택을 외부 사용자에게 노출하지 않습니다.
        raise HTTPException(status_code=503, detail='모델 처리에 실패했습니다. 서버 로그를 확인하세요.') from exc
```

**왜 `run_in_threadpool`을 사용하나요?** `async def` 라우터에서 무거운 동기식 Whisper 추론을 직접 수행하면 이벤트 루프가 정지할 수 있습니다. 무거운 동기 작업을 별도 스레드에 위임합니다. 이것이 Redis·Celery의 *작업 큐 비동기 처리*와 동일하다는 뜻은 아닙니다. 이번 수업의 HTTP 응답은 **추론이 끝날 때까지 기다리는 동기 완료형 응답**입니다.

## PART 4. `main.py`를 마지막에 작성합니다

`router.py`가 실제로 생성된 다음에만 `main.py`에서 import합니다.

**`main.py` — 신규 생성**

```python
from fastapi import FastAPI
from speech.router import router as speech_router

app = FastAPI(title='FastAPI Speech STT and Translation')
app.include_router(speech_router)

@app.get('/health')
def health():
    return {'status': 'ok'}
```

### 4-1. 실행 명령

```bash
# 프로젝트 루트: ~/fastapi_speech
python -m uvicorn main:app --reload
```

서버가 실행된 상태에서 다음 주소를 확인합니다.

- `http://127.0.0.1:8000/health` → `{"status":"ok"}`
- `http://127.0.0.1:8000/docs` → FastAPI Swagger UI

`POST /api/stt-to-ko/`는 GET 주소창에 입력하면 실행되지 않습니다. 반드시 Insomnia, curl 또는 Swagger UI에서 파일을 포함한 **POST** 요청을 보내야 합니다.

## PART 5. 학생이 직접 음성 파일을 준비합니다

휴대폰 녹음 앱으로 다음 문장을 각각 말하고, 프로젝트 폴더 안에 복사합니다.

| 파일 | 읽을 문장 |
|---|---|
| `sample_ko.m4a` | 안녕하세요. 이 파일은 STT 테스트를 위한 한국어 음성 파일입니다. |
| `sample_en.m4a` | This is an English-only audio file. We are testing speech to text recognition. |
| `sample_any.m4a` | 안녕하세요. This is a mixed language audio file. 감사합니다. |

Windows에서 WSL 프로젝트를 탐색기로 열려면 `explorer.exe .`를 실행합니다. Windows 다운로드 폴더에서 복사하려면 `ls /mnt/c/Users`로 Windows 사용자 폴더를 먼저 확인하세요. 사용자 이름을 임의로 가정하지 않습니다.

```bash
# 프로젝트 루트에서 실행
explorer.exe .
# 파일을 복사한 뒤
ls -lh sample_*.m4a
# 포맷 확인용 WAV 파일도 생성
ffmpeg -y -i sample_ko.m4a -ar 16000 -ac 1 sample_ko.wav
ffprobe -hide_banner sample_ko.wav
```

`ffmpeg` 변환은 모델이 실행되는 데 필수 선행 단계는 아닙니다. 우리 `services.py` 안에서도 자동 변환하기 때문입니다. 여기에서는 **오디오 디코딩의 의미를 눈으로 확인하는 실습**입니다.

## PART 6. Insomnia와 curl로 API 확인

### 6-1. 한국어 전사

```bash
curl -sS -X POST 'http://127.0.0.1:8000/api/stt-to-ko/' \
  -F 'audio=@sample_ko.wav' -F 'src_lang=ko'; echo
```

예상 응답 **구조** (실제 전사 문장은 녹음 품질에 따라 변합니다):

```json
{
  "src_lang": "ko",
  "stt_text": "안녕하세요. 이 파일은 STT 테스트를 위한 한국어 음성 파일입니다.",
  "ko_text": "안녕하세요. 이 파일은 STT 테스트를 위한 한국어 음성 파일입니다.",
  "used_translation": false,
  "translation_status": "skipped"
}
```

### 6-2. 영어 → 한국어 번역

```bash
curl -sS -X POST 'http://127.0.0.1:8000/api/stt-to-ko/' \
  -F 'audio=@sample_en.m4a' -F 'src_lang=en'; echo
```

`translation_status='translated'`이면 번역 모델 호출이 완료된 것입니다. `failed`이면 `ko_text`가 원문으로 남습니다. **번역 모델을 설치만 했다고 자동으로 번역에 성공하는 것은 아닙니다.**

### 6-3. 언어 자동 판단

```bash
curl -sS -X POST 'http://127.0.0.1:8000/api/stt-to-ko/' \
  -F 'audio=@sample_any.m4a' -F 'src_lang=auto'; echo
```

영어와 한국어가 섞인 경우 자동 판단 규칙에 의해 `translation_status='skipped'`가 될 수 있습니다. 이는 코드 오류가 아니라 01에서 정한 교육용 규칙입니다.

### 6-4. Insomnia 화면으로 확인

Insomnia에서 `POST http://127.0.0.1:8000/api/stt-to-ko/` 요청을 만들고 Body의 `Multipart Form` 또는 `Form Data`를 선택합니다.

| Name | Type | Value |
|---|---|---|
| `audio` | File | 녹음 파일 선택 |
| `src_lang` | Text | `auto`, `ko`, `en` |

원본 교재의 Insomnia 캡처 참조를 같은 실습 구간에 유지합니다. 이미지 파일이 Obsidian Vault에 있어야 표시됩니다.

![[Pasted image 20260205093634.png]]

![[Pasted image 20260205094231.png]]

응답 JSON 예시를 실제 테스트 결과와 비교하고 `stt_text`, `ko_text`, `used_translation`, `translation_status` 네 필드를 확인합니다.

## PART 7. API 테스트 자동화

실제 모델 다운로드와 무관하게 **업로드 라우팅·오류 응답·ffmpeg WAV 디코딩·번역 분기·임시파일 정리**를 검증합니다. 완성된 실행 프로젝트에는 다음 테스트 파일이 함께 있습니다.

**`tests/test_api.py` — 신규 생성**

```python
"""실제 모델 다운로드 없이 라우팅·검증·JSON 계약을 검사하는 테스트."""
import io
import wave
from pathlib import Path
from fastapi.testclient import TestClient
from main import app
from speech import services

client = TestClient(app)

def wav_bytes():
    out = io.BytesIO()
    with wave.open(out, 'wb') as f:
        f.setnchannels(1); f.setsampwidth(2); f.setframerate(16000)
        f.writeframes(b'\x00\x00' * 16000)
    return out.getvalue()

def fake_process(content, suffix, lang):
    assert content and suffix == '.wav'
    return {'src_lang':lang,'stt_text':'Hello world', 'ko_text':'안녕하세요',
            'used_translation':True,'translation_status':'translated'}

def test_health():
    assert client.get('/health').json() == {'status':'ok'}

def test_upload_success(monkeypatch):
    monkeypatch.setattr(services, 'process_audio_bytes', fake_process)
    response = client.post('/api/stt-to-ko/', files={'audio':('sample.wav',wav_bytes(),'audio/wav')}, data={'src_lang':'en'})
    assert response.status_code == 200, response.text
    assert response.json()['ko_text']=='안녕하세요'

def test_invalid_src_lang():
    response=client.post('/api/stt-to-ko/', files={'audio':('sample.wav',wav_bytes(),'audio/wav')}, data={'src_lang':'xx'})
    assert response.status_code == 400

def test_missing_file():
    assert client.post('/api/stt-to-ko/',data={'src_lang':'ko'}).status_code == 422

def test_invalid_extension():
    response=client.post('/api/stt-to-ko/', files={'audio':('sample.txt',b'not an audio','text/plain')})
    assert response.status_code == 400

def test_empty_audio():
    response=client.post('/api/stt-to-ko/', files={'audio':('sample.wav',b'','audio/wav')})
    assert response.status_code == 400

def test_oversize():
    response=client.post('/api/stt-to-ko/', files={'audio':('sample.wav',b'1'*(20*1024*1024+1),'audio/wav')})
    assert response.status_code == 413

def test_translation_rules(monkeypatch):
    monkeypatch.setattr(services, 'translate_en_to_ko', lambda txt: '내일 회의에 참석합니다.')
    assert services.maybe_translate_to_ko('ko','안녕하세요')[2]=='skipped'
    assert services.maybe_translate_to_ko('auto','Hello world')[2]=='translated'
    assert services.maybe_translate_to_ko('auto','한국어 and English')[2]=='skipped'
    assert services.maybe_translate_to_ko('en','Hello world')[1] is True

def test_translation_fail_closed(monkeypatch):
    def fail(txt): raise RuntimeError('model offline')
    monkeypatch.setattr(services,'translate_en_to_ko',fail)
    out=services.maybe_translate_to_ko('en','Hello')
    assert out == ('Hello',False,'failed')

def test_ffmpeg_wav_decode(tmp_path):
    source=tmp_path/'sample.wav';source.write_bytes(wav_bytes())
    samples=services.decode_audio(str(source))
    assert len(samples)==16000

def test_audio_pipeline_contract(tmp_path, monkeypatch):
    class FakePipe:
        def __call__(self, audio, generate_kwargs):
            assert audio['sampling_rate']==16000
            assert len(audio['array'])==16000
            assert generate_kwargs == {'task':'transcribe','language':'korean'}
            return {'text':'안녕하세요'}
    monkeypatch.setattr(services,'get_asr',lambda: FakePipe())
    path=tmp_path/'test.wav';path.write_bytes(wav_bytes())
    assert services.safe_asr_transcribe(str(path), 'ko')=='안녕하세요'

def test_temp_file_cleanup(monkeypatch):
    captured={}
    def fake_transcribe(path, lang):
        captured['path']=path
        assert Path(path).exists()
        return '안녕하세요'
    monkeypatch.setattr(services,'safe_asr_transcribe',fake_transcribe)
    payload=services.process_audio_bytes(wav_bytes(),'.wav','ko')
    assert payload['stt_text']=='안녕하세요'
    assert not Path(captured['path']).exists()

def test_invalid_wav_decoder(tmp_path):
    path=tmp_path/'bad.wav';path.write_bytes(b'NOT_WAV')
    import pytest
    with pytest.raises(ValueError):services.decode_audio(str(path))
```

```bash
# 반드시 프로젝트 루트에서 실행
python -m pytest -q
```

**검증 구분:** 테스트에서는 네트워크로 모델을 실제 다운로드하지 않도록 모의 함수(Fake)를 사용합니다. 테스트가 모두 통과해도 **실제 Whisper 정확도와 영어→한국어 모델의 전사·번역 출력은 학생 PC에서 별도 확인해야 합니다.**

### 실제 Hugging Face 모델 로딩 확인 (자동 테스트와 별개)

위의 `pytest`는 가짜 모델로 테스트하므로, **처음 수업을 진행하는 PC에서는 실제 모델도 별도로 로딩해 보아야 합니다.** 인터넷이 필요하고 모델 파일 다운로드와 메모리 확보에 시간이 걸릴 수 있습니다.

```bash
# 먼저 모델 ID와 로딩 자체를 확인합니다. 실제 음성 품질 검증은 아닙니다.
python - <<'PYMODEL'
from speech.services import get_asr, get_mt
get_asr()
print('Whisper 모델 로딩 성공')
get_mt()
print('영어→한국어 번역 모델 로딩 성공')
PYMODEL
```

모델 로딩 성공 후에는 PART 6의 **실제 한국어·영어 녹음**으로 `stt_text`, `ko_text`, `translation_status`를 확인합니다. GPU가 없어도 CPU로 실행 가능하지만 메모리와 처리 시간은 환경마다 다릅니다.

## PART 8. CPU/GPU 확인

```bash
python -c "import torch; print('CUDA:', torch.cuda.is_available()); print('GPU:', torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU 추론')"
```

`get_asr()`/`get_mt()`는 CUDA를 사용할 수 있으면 GPU, 아니면 CPU를 사용합니다. `CUDA=True`여도 모델이 GPU를 사용하지 않는 문제는 CUDA용 PyTorch 설치, GPU 드라이버와 WSL 연동을 확인해야 합니다. 모델 최초 로딩은 시간이 오래 걸릴 수 있고, 측정 시 첫 요청과 두 번째 요청을 구분해야 합니다.

## PART 9. 미니 챌린지 및 정리

### 미니 챌린지

1. `src_lang='ko'`와 `auto`를 같은 한국어 WAV로 비교합니다.
2. 영어 음성에서 `translation_status`가 `translated`인지 확인합니다.
3. 확장자가 `.txt`인 파일을 올려 `400`, 너무 큰 WAV를 올려 `413`을 확인합니다.
4. 인식 오류가 발생한 문장을 기록하고 가능한 원인을 2가지 이상 정리합니다.

### GitHub 업로드

`.venv/`, 임시 녹음 파일, 개인정보가 들어간 실제 녹음, Hugging Face 모델 캐시를 업로드하지 않습니다. 소스코드와 `requirements.txt`, 실습 결과 README만 버전 관리합니다.

### 스스로 점검

- [ ] `python -m uvicorn main:app --reload`로 서버가 실행된다.
- [ ] `/health`가 JSON을 반환한다.
- [ ] WAV 또는 M4A 파일을 POST하면 4개 결과 필드가 반환된다.
- [ ] 영어 음성의 번역 실패 여부를 `translation_status`로 구분한다.
- [ ] FastAPI 서버와 Whisper 모델의 역할을 각각 설명할 수 있다.

> **다음 03 단계:** 지금 만든 API와 서비스 코드를 유지한 채, 브라우저 마이크를 JavaScript MediaRecorder로 녹음하여 `/api/stt-to-ko/`로 전송합니다.

