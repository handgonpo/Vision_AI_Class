## 오늘 수업의 목표

- DRF Serializer 역할을 FastAPI `File/Form`, Dependency 검증 함수와 비교합니다.
- 음성 확장자, 20MB 업로드, `ko/en/auto` 검증을 한 곳에서 관리합니다.
- 라우터는 요청·응답만, 서비스는 ffmpeg·Whisper·번역 로직만 담당하도록 정리합니다.
- 기존 `/api/stt-to-ko/`, `/record/`를 그대로 호출하면서 리팩토링 전후의 결과가 같음을 확인합니다.

## 1. 기존의 구조와 개선 후 구조

```text
[03 단계]
Browser → speech/router.py
            ├─ 언어값 검증
            ├─ 파일명·용량 검증
            └─ services.process_audio_bytes()

[04 단계]
Browser → speech/router.py
            ↓ Depends(validate_audio_request)
          speech/validators.py
            ├─ audio 필수 확인
            ├─ 확장자·20MB 검사
            └─ src_lang 허용값 검증
            ↓ ValidatedAudio
          speech/router.py
            ↓ services.process_audio_bytes()
          Whisper / 번역 → JSON
```

이 구조는 DRF와 사상이 비슷하지만 구현 방법이 다릅니다.

| 구분 | Django REST Framework | FastAPI |
|---|---|---|
| 파일 필수 확인 | `serializers.FileField()` | `UploadFile = File(...)` |
| 언어 선택값 검사 | `ChoiceField` | 별도 검증 함수 또는 `Literal`/Pydantic |
| 복합 검증 | `validate_audio()` | `validate_audio_request()` |
| 검증 결과 | `validated_data` | `ValidatedAudio` 객체 |
| 검증기 실행 | `.is_valid(raise_exception=True)` | `Depends()`로 자동 호출 |
| 요청 처리 | `APIView.post()` | `@router.post()` |
| JSON 응답 | `Response` | dict 반환 |

## PART 1. 프로젝트 구조 확인

03까지 작업한 `fastapi_speech` 폴더에서 **새 파일은 `validators.py` 하나**이며, `router.py`를 교체합니다.

```text
fastapi_speech/
├── requirements.txt              # 02 그대로
├── main.py                       # 03 그대로
├── speech/
│   ├── __init__.py
│   ├── services.py               # 02 그대로
│   ├── router.py                 # 지금 교체
│   └── validators.py             # 지금 생성
├── templates/base.html           # 03 그대로
├── templates/speech/record.html  # 03 그대로
├── static/js/record.js           # 03 그대로
├── static/css/style.css          # 03 그대로
└── tests/                        # API + 웹 자동 테스트
```

**실습 주의:** `main.py`를 다시 새로 만들 필요는 없습니다. 파일 변경 후 `uvicorn --reload`가 자동 반영합니다. `validators.py`를 만들기 **전**에 수정된 `router.py`를 서버에서 import하면 오류가 나므로 먼저 `validators.py`를 작성합니다.

## PART 2. 검증 파일 `speech/validators.py` 작성

### 2-1. 이번 파일이 하는 일

- `audio`가 multipart 파일로 넘어오는지 FastAPI에 선언합니다.
- 확장자와 크기를 확인합니다.
- 언어 선택값을 소문자로 표준화하고 `ko/en/auto`인지 확인합니다.
- 잘못된 입력으로 중간에 종료되어도 `finally`에서 업로드 파일을 닫습니다.
- 처리 함수가 사용할 `ValidatedAudio` 객체를 반환합니다.

**검증 파일은 Hugging Face 모델을 다운로드하거나 STT를 실행하지 않습니다.** 입력이 올바른지만 검사합니다.

### 2-2. 의사코드

```text
validate_audio_request 시작
    src_lang 소문자 변환
    허용되지 않은 값이면 400 반환
    파일 확장자가 허용 목록에 없으면 400 반환
    최대 20MB+1바이트만 읽기
    파일이 비어 있으면 400 반환
    용량 초과면 413 반환
    통과하면 ValidatedAudio(content, suffix, src_lang) 반환
```

### 2-3. 전체 코드

**`speech/validators.py` — 파일 내용**

```python
"""04단계: Django DRF Serializer 역할을 FastAPI Dependency로 분리."""
from dataclasses import dataclass
from pathlib import Path
from fastapi import File, Form, HTTPException, UploadFile

MAX_UPLOAD_BYTES = 20 * 1024 * 1024
ALLOWED_EXTS = {'.wav', '.mp3', '.m4a', '.flac', '.ogg', '.webm', '.mp4'}
ALLOWED_LANGS = {'ko', 'en', 'auto'}

@dataclass(frozen=True)
class ValidatedAudio:
    content: bytes
    suffix: str
    src_lang: str

async def validate_audio_request(
    audio: UploadFile = File(...), src_lang: str = Form('auto')
) -> ValidatedAudio:
    # [검토 보완] 언어·확장자 검증 도중 오류가 나도 업로드 파일을 닫습니다.
    try:
        normalized_lang = src_lang.strip().lower()
        if normalized_lang not in ALLOWED_LANGS:
            raise HTTPException(400, 'src_lang은 ko, en, auto 중 하나여야 합니다.')
        suffix = Path(audio.filename or '').suffix.lower()
        if suffix not in ALLOWED_EXTS:
            raise HTTPException(400, '지원하지 않는 음성 확장자입니다.')
        content = await audio.read(MAX_UPLOAD_BYTES + 1)
        if not content:
            raise HTTPException(400, '음성 파일이 비어 있습니다.')
        if len(content) > MAX_UPLOAD_BYTES:
            raise HTTPException(413, '20MB 이하 파일만 허용합니다.')
        return ValidatedAudio(content=content, suffix=suffix, src_lang=normalized_lang)
    finally:
        await audio.close()
```

### 2-4. 왜 `ValidatedAudio`를 만들었나요?

다른 파일이 업로드 객체 안을 직접 들여다볼 필요 없이 **이미 검증된 값**만 사용하게 하기 위해서입니다. `@dataclass(frozen=True)`는 단순 데이터 묶음이며, Pydantic 모델과는 다릅니다. 공개 API 입력을 JSON으로 받는 경우에는 Pydantic `BaseModel`을 적용할 수 있지만 이번 요청은 `multipart/form-data`이므로 파일·폼 검증을 Dependency로 묶었습니다.

## PART 3. `speech/router.py`를 교체합니다

03의 라우터는 20MB/확장자 검증 코드까지 포함했지만, 이번 라우터는 **검증을 직접 하지 않습니다.** `Depends(validate_audio_request)`가 먼저 실행됩니다.

### 3-1. 의사코드

```text
1. POST 요청을 받는다.
2. FastAPI가 validate_audio_request를 실행한다.
3. 검증 실패면 이미 400/413/422 응답을 반환한다.
4. 정상이라면 검증된 데이터로 STT 서비스 함수를 호출한다.
5. 서비스의 결과를 JSON으로 반환한다.
6. 음성 자체 오류는 400, 서버 모델 처리 오류는 503으로 구분한다.
```

### 3-2. 변경할 코드 전체

**`speech/router.py` — 파일 내용**

```python
"""04단계: 라우터는 요청, 검증기 호출, 응답만 담당합니다."""
from typing import Annotated
from fastapi import APIRouter, Depends, HTTPException
from starlette.concurrency import run_in_threadpool
from . import services
from .validators import ValidatedAudio, validate_audio_request

router = APIRouter()

@router.post('/api/stt-to-ko/', name='stt_to_ko')
async def stt_to_ko(data: Annotated[ValidatedAudio, Depends(validate_audio_request)]):
    try:
        return await run_in_threadpool(
            services.process_audio_bytes, data.content, data.suffix, data.src_lang
        )
    except ValueError as exc:
        raise HTTPException(status_code=400, detail=str(exc)) from exc
    except Exception as exc:
        raise HTTPException(status_code=503, detail='모델 처리에 실패했습니다. 서버 로그를 확인하세요.') from exc
```

### 3-3. 코드 핵심 해석

`Annotated[ValidatedAudio, Depends(validate_audio_request)]`는 **검증 함수를 FastAPI에 의존성으로 등록**하는 표현입니다. 라우터가 실행되기 전에 FastAPI가 검증 함수를 호출한 후 그 반환값을 `data`에 주입합니다.

`run_in_threadpool(...)`은 Whisper처럼 오래 걸리는 동기 추론이 이벤트 루프를 블로킹하지 않게 하려는 처리입니다. Redis·Celery처럼 **작업 번호를 반환하고 백그라운드에서 계속 처리하는 기능은 이 프로젝트에 포함되지 않습니다.** 본 API는 모델 추론이 끝나야 최종 JSON을 응답합니다.

## PART 4. 03의 `main.py`는 그대로 사용합니다

혼동을 방지하기 위해 최종 `main.py` 전체를 다시 보여 줍니다. 이 파일에서 라우터와 템플릿·정적 경로가 등록됩니다.

**`main.py` — 파일 내용**

```python
from pathlib import Path
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse, RedirectResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
from speech.router import router as speech_router

BASE_DIR = Path(__file__).resolve().parent
app = FastAPI(title='FastAPI Speech STT and Translation')
app.include_router(speech_router)
app.mount('/static', StaticFiles(directory=str(BASE_DIR / 'static')), name='static')
templates = Jinja2Templates(directory=str(BASE_DIR / 'templates'))

@app.get('/health')
def health():
    return {'status': 'ok'}

@app.get('/', include_in_schema=False)
def index():
    return RedirectResponse('/record/', status_code=307)

@app.get('/record/', response_class=HTMLResponse, name='record_page')
def record_page(request: Request):
    return templates.TemplateResponse(request=request, name='speech/record.html', context={})
```

`services.py` 역시 02에서 만든 코드와 동일합니다. 검증 책임만 옮긴 것이므로 **모델의 로딩·번역 구현을 다시 복사하거나 바꾸지 마세요.** 파일별 완성 소스는 ZIP의 `step04_validation/speech/services.py`에 포함되어 있습니다.

## PART 5. 서버 실행 및 요청 검증

```bash
cd ~/fastapi_speech
source .venv/bin/activate
python -m uvicorn main:app --reload
```

브라우저는 `http://127.0.0.1:8000/record/`, API는 `http://127.0.0.1:8000/api/stt-to-ko/`입니다. `docs`에서 POST 엔드포인트도 확인할 수 있습니다.

### 오류별 기대 결과

| 입력 | HTTP | 이유 |
|---|---|---|
| 정상 WAV 1개 + `ko` | 200 | STT 결과 반환(실제 모델 사용 시) |
| audio 누락 | 422 | FastAPI 필수 파일 필드 검사 |
| `src_lang=xx` | 400 | 허용되지 않은 언어 옵션 |
| `audio=sample.txt` | 400 | 확장자 검사 실패 |
| 비어 있는 WAV | 400 | 파일 내용 없음 |
| 20MB 초과 | 413 | 업로드 용량 제한 |
| WAV 확장자지만 실제 데이터가 손상됨 | 400 | ffmpeg 디코딩 실패 |
| 모델 다운로드/추론 중 예외 | 503 | 서버 처리 실패 |

**보안 참고:** 확장자 검사는 기본 입력 정책이지 파일 안전성의 완전한 보장은 아닙니다. 본 수업에서는 `ffmpeg`가 실제 파일을 해석하도록 하고, 운영 환경에서는 업로드 본문 제한, 악성 파일 검사, 인증·요청 제한 등의 추가 보안이 필요합니다.

## PART 6. 자동 테스트

다음 명령으로 API·입력 검증·임시파일 정리·WAV 디코딩·웹 HTML 응답을 검사합니다.

```bash
python -m pytest -q
```

완성된 프로젝트에는 `tests/test_api.py` 및 `tests/test_web.py`가 있습니다. 우선 아래 2개 파일을 확인해 테스트 목적을 이해하세요.

**`tests/test_api.py` — 파일 내용**

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

**`tests/test_web.py` — 파일 내용**

```python
from fastapi.testclient import TestClient
from main import app
client=TestClient(app)
def test_record_page():
 r=client.get('/record/');assert r.status_code==200 and 'recordBtn' in r.text
 assert '/static/js/record.js' in r.text
def test_js_served():
 r=client.get('/static/js/record.js');assert r.status_code==200 and 'MediaRecorder' in r.text
```

### 테스트 결과를 해석하는 방법

자동 테스트는 모델 자체를 다운로드하지 않고 **모의(Fake) 결과**를 사용합니다. 또한 실제 ffmpeg를 호출해 1초짜리 테스트 WAV가 16kHz 샘플로 처리되는지 확인합니다. 이 방식은 소스코드의 기본 HTTP·파일·검증 흐름을 빠르게 확인하는 데 유용하지만, 실모델 인식 정확도까지 증명하지는 않습니다.

## PART 7. 원본 DRF 코드에서 수정한 문제

- `Serializer` 클래스의 `validate_audio`를 그대로 FastAPI로 복사하지 않았습니다. FastAPI 방식인 `File`, `Form`, `Depends`를 적용했습니다.
- Django `manage.py`, `settings.py`, `urls.py`, `APIView`, `render`를 제거하고 FastAPI 앱·라우터·템플릿으로 전환했습니다.
- 03의 `/record/`과 API의 `/api/stt-to-ko/`를 같은 FastAPI 앱에 연결하여 중복 `api/` prefix 오류를 방지했습니다.
- 예외 `repr(e)`를 클라이언트에게 그대로 노출하지 않도록 수정했습니다.
- `tempfile`의 정리 책임을 `services.py`로 옮겼고, `finally`에서 삭제하도록 했습니다.
- 원본 서비스가 영어 번역 실패를 조용히 원문으로 돌려줬던 부분에 `translation_status`를 추가했습니다.

## PART 8. 미니 챌린지·GitHub·자가 체크리스트

### 미니 챌린지

1. 언어 옵션 `auto`를 제거하고 `ko/en`만 허용하도록 `validators.py`를 변경해 봅니다.
2. 20MB 제한을 10MB로 변경하고 11MB 파일로 413을 확인해 봅니다.
3. 서버 로깅과 사용자 오류 응답을 분리해 봅니다.
4. 같은 녹음에 `ko/en/auto`를 각각 선택해 결과를 비교합니다.

### GitHub 업로드

소스코드와 `requirements.txt`·학습 결과 README만 업로드합니다. `.venv`, 음성 개인정보, 모델 캐시, `__pycache__`는 제외합니다.

### 스스로 확인하기

- [ ] `validators.py`가 입력 검사만 담당한다.
- [ ] `router.py`는 검증된 요청으로 서비스 함수를 호출한다.
- [ ] `services.py`는 음성 디코딩·모델 추론·번역을 담당한다.
- [ ] `/record/`에서 녹음 결과가 화면에 표시된다.
- [ ] 20MB 초과 업로드는 413, 누락 파일은 422가 반환된다.
- [ ] FastAPI의 요청 함수 `async def`와 Celery 비동기 작업 큐가 서로 다름을 설명할 수 있다.

**최종 연결:** `00~01 개념과 모델 선정 → 02 FastAPI STT API → 03 JavaScript 마이크 UI → 04 FastAPI 검증 계층 분리` 순서로 하나의 실행 프로젝트가 완성됩니다.

