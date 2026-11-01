## 오늘 수업의 목표

- 02에서 만든 FastAPI STT API를 재사용합니다.
- `MediaRecorder`로 25초 이내 마이크 소리를 녹음합니다.
- 브라우저에서 `FormData`로 음성 파일을 업로드합니다.
- FastAPI JSON 결과를 Jinja2 HTML 화면에 표시합니다.
- `localhost`/HTTPS의 마이크 보안 조건을 이해합니다.

## 전체 실행 흐름

```text
[브라우저: 녹음 시작]
       ↓ getUserMedia({audio:true})
[MediaRecorder: 녹음]
       ↓ ondataavailable → chunks → Blob
[FormData: audio, src_lang]
       ↓ POST /api/stt-to-ko/
[FastAPI: router → services → Whisper → 번역]
       ↓ JSON
[브라우저: stt_text / ko_text / translation_status 표시]
```

## PART 1. 02 실습 프로젝트를 확장합니다

**기존 작업을 지우지 않습니다.** 02에서 만든 `requirements.txt`, `speech/services.py`, `speech/router.py`는 그대로 둡니다. 새로운 것은 정적 파일과 HTML, 수정된 `main.py`입니다.

```text
fastapi_speech/
├── main.py                      # 이 단계에서 전체 교체
├── speech/
│   ├── __init__.py
│   ├── router.py                # 02와 동일
│   └── services.py              # 02와 동일
├── templates/
│   ├── base.html                # 새로 생성
│   └── speech/
│       └── record.html         # 새로 생성
├── static/
│   ├── css/style.css            # 새로 생성
│   └── js/record.js             # 새로 생성
└── tests/
    ├── test_api.py             # 02와 동일
    └── test_web.py             # 추가
```

```bash
cd ~/fastapi_speech
source .venv/bin/activate
mkdir -p templates/speech static/css static/js
```

## PART 2. `templates/base.html` 작성

`base.html`은 모든 HTML의 공통 레이아웃입니다. Jinja2의 `{% block %}`은 자식 템플릿이 값을 채울 공간을 만들며, `{% extends %}`는 공통 레이아웃을 상속합니다. Django의 `{% load static %}`은 사용하지 않습니다.

**`templates/base.html` — 파일 내용**

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% block title %}FastAPI STT 웹앱{% endblock %}</title>
  <link rel="stylesheet" href="{{ url_for('static', path='/css/style.css') }}">
</head>
<body><main class="container">{% block content %}{% endblock %}</main></body>
</html>
```

**핵심:** CSS 경로는 `{{ url_for('static', path='/css/style.css') }}`로 구성합니다. `main.py`에서 같은 이름인 `static`을 마운트해야 작동합니다.

## PART 3. `templates/speech/record.html` 작성

이 화면은 마이크 녹음 시작/종료 버튼, 음성 파일 업로드 버튼, STT 결과, 한국어 번역 결과를 보여 줍니다. 화면을 제공하는 Python 코드는 추론하지 않으며, **JavaScript가 POST API를 호출**합니다.

**`templates/speech/record.html` — 파일 내용**

```html
{% extends "base.html" %}
{% block title %}Whisper 음성 인식 + 번역{% endblock %}
{% block content %}
<h1>🎤 음성 녹음 → STT → 한국어 번역</h1>
<p>Chrome/Edge는 WebM, Safari 등은 지원하는 오디오 형식으로 녹음합니다. 브라우저 녹음은 25초에 자동 종료됩니다(서버는 30초 이하 파일 허용).</p>
<div class="card">
<label>원본 음성 언어
  <select id="srcLang"><option value="auto">auto - 자동 판단</option><option value="ko">ko - 한국어</option><option value="en">en - 영어</option></select>
</label>
<p>
<button type="button" id="recordBtn">녹음 시작</button>
<button type="button" id="stopBtn" disabled>녹음 종료</button>
</p>
<p>파일로 테스트: <input id="fileInput" type="file" accept="audio/*"></p>
<button type="button" id="fileBtn">파일 업로드 분석</button>
<p id="status" role="status">준비</p>
<audio id="playback" controls hidden></audio>
</div>
<div class="card">
<h2>STT 원문</h2><p id="sttText">-</p>
<h2>한국어 결과</h2><p id="translatedText">-</p>
<p id="translationStatus"></p>
</div>
<script src="{{ url_for('static', path='/js/record.js') }}" defer></script>
{% endblock %}
```

## PART 4. `static/js/record.js` 작성

### JavaScript 실행 주체

- `navigator.mediaDevices.getUserMedia()`는 **브라우저**가 마이크에 접근하도록 요청합니다.
- `MediaRecorder`는 브라우저가 마이크의 소리 데이터를 녹음하도록 제어합니다.
- `Blob`은 메모리의 녹음 데이터를 파일처럼 묶습니다.
- `FormData`는 서버에 전송할 `audio` 파일과 `src_lang`을 구성합니다.
- `fetch`는 `/api/stt-to-ko/`로 HTTP POST 요청을 보냅니다.
- Whisper 모델은 브라우저가 아닌 **FastAPI 서버 쪽**에서 실행됩니다.

### 왜 Axios 대신 Fetch를 사용하나요?

원본 자료의 `axios.post()`도 정상적인 HTTP 클라이언트입니다. 이번 개정에서는 **외부 CDN이 연결되지 않아도 실습이 가능**하도록 브라우저 기본 제공 `fetch`를 사용했습니다. 동작 자체는 같습니다.

| Axios | Fetch |
|---|---|
| `const res = await axios.post(url, form)` | `const res = await fetch(url, {method:'POST',body:form})` |
| `res.data`에 응답이 들어 있음 | `await res.json()`을 호출 |
| 4xx/5xx는 보통 자동으로 예외 | `res.ok`를 직접 검사해야 함 |

### 의사코드

```text
1. 녹음 버튼을 누르면 마이크 권한을 요청한다.
2. 브라우저가 지원하는 MIME(webm/mp4)을 선택한다.
3. MediaRecorder로 소리 데이터를 수집한다.
4. 종료 버튼 또는 25초 제한으로 녹음을 끝낸다.
5. Blob을 만든 뒤 그 형식에 맞는 확장자로 이름을 붙인다.
6. FormData에 audio와 src_lang을 넣는다.
7. POST /api/stt-to-ko/를 호출한다.
8. JSON의 STT 원문과 번역 결과를 DOM에 표시한다.
9. 마이크 스트림의 track을 종료하고 파일 URL을 관리한다.
```

### 코드 전체

**`static/js/record.js` — 파일 내용**

```javascript
"use strict";
// 이 코드는 브라우저에서 실행합니다. 서버의 Python 코드가 아닙니다.
const startBtn = document.getElementById("recordBtn");
const stopBtn = document.getElementById("stopBtn");
const fileBtn = document.getElementById("fileBtn");
const fileInput = document.getElementById("fileInput");
const srcLang = document.getElementById("srcLang");
const statusEl = document.getElementById("status");
const sttEl = document.getElementById("sttText");
const koEl = document.getElementById("translatedText");
const translationEl = document.getElementById("translationStatus");
const playback = document.getElementById("playback");
let mediaRecorder = null;
let stream = null;
let chunks = [];
let previewUrl = null;
let recordingTimeout = null;

function getRecordingMime() {
  // 브라우저가 지원하는 MIME만 사용 (webm/mp4는 브라우저에 따라 다름)
  for (const mime of ["audio/webm;codecs=opus", "audio/webm", "audio/mp4"]) {
    if (MediaRecorder.isTypeSupported(mime)) return mime;
  }
  return "";
}

async function uploadAudio(blob, filename) {
  const formData = new FormData();
  formData.append("audio", blob, filename);
  formData.append("src_lang", srcLang.value);
  statusEl.textContent = "서버에서 STT 처리 중입니다...";
  try {
    const res = await fetch("/api/stt-to-ko/", { method: "POST", body: formData });
    const payload = await res.json();
    if (!res.ok) throw new Error(payload.detail || `HTTP ${res.status}`);
    sttEl.textContent = payload.stt_text || "(인식된 문장이 없습니다)";
    koEl.textContent = payload.ko_text || "(결과 없음)";
    translationEl.textContent = `번역 상태: ${payload.translation_status}`;
    statusEl.textContent = "완료";
  } catch (error) {
    statusEl.textContent = `오류: ${error.message}`;
  }
}

startBtn.addEventListener("click", async () => {
  if (!navigator.mediaDevices || !window.MediaRecorder) {
    statusEl.textContent = "마이크 API를 사용할 수 없습니다. localhost 또는 HTTPS에서 열어주세요.";
    return;
  }
  try {
    stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    const mime = getRecordingMime();
    mediaRecorder = mime ? new MediaRecorder(stream, { mimeType: mime }) : new MediaRecorder(stream);
    chunks = [];
    mediaRecorder.ondataavailable = (event) => { if (event.data.size > 0) chunks.push(event.data); };
    mediaRecorder.onstop = async () => {
      clearTimeout(recordingTimeout);
      stream.getTracks().forEach((track) => track.stop());
      // 실제 녹음된 MIME을 확인해 서버가 허용하는 확장자를 붙입니다.
      const actualMime = (mediaRecorder.mimeType || mime || "").toLowerCase();
      const ext = actualMime.includes("webm") ? ".webm"
                : actualMime.includes("mp4") ? ".mp4"
                : actualMime.includes("ogg") ? ".ogg"
                : null;
      startBtn.disabled = false;
      stopBtn.disabled = true;
      if (!ext) {
        statusEl.textContent = `지원하지 않는 녹음 형식: ${actualMime || "알 수 없음"}`;
        return;
      }
      const blob = new Blob(chunks, { type: actualMime });
      if (!blob.size) {
        statusEl.textContent = "녹음된 데이터가 없습니다. 다시 시도하세요.";
        return;
      }
      if (previewUrl) URL.revokeObjectURL(previewUrl);
      previewUrl = URL.createObjectURL(blob);
      playback.src = previewUrl;
      playback.hidden = false;
      startBtn.disabled = false;
      stopBtn.disabled = true;
      await uploadAudio(blob, `recording${ext}`);
    };
    mediaRecorder.start();
    startBtn.disabled = true;
    stopBtn.disabled = false;
    statusEl.textContent = "녹음 중... (최대 25초)";
    recordingTimeout = setTimeout(() => { if (mediaRecorder.state === "recording") mediaRecorder.stop(); }, 25000);
  } catch (error) {
    if (stream) stream.getTracks().forEach((track) => track.stop());
    statusEl.textContent = `마이크 사용 실패: ${error.message}`;
  }
});

stopBtn.addEventListener("click", () => {
  if (mediaRecorder && mediaRecorder.state === "recording") mediaRecorder.stop();
});
fileBtn.addEventListener("click", async () => {
  if (!fileInput.files.length) { statusEl.textContent = "음성 파일을 선택하세요."; return; }
  await uploadAudio(fileInput.files[0], fileInput.files[0].name);
});
```

**원본 오류 수정:** `<script>` 안의 `<div id="recordBtn">`, `#recordBtn{color="red"}` 같은 코드는 JavaScript가 아니므로 실행할 수 없습니다. HTML, CSS, JavaScript를 각각 파일로 분리해야 합니다.

**브라우저별 음성 포맷:** Chrome/Edge는 대개 `audio/webm`, Safari는 `audio/mp4`, 일부 환경은 `audio/ogg` 등을 지원할 수 있으므로 `MediaRecorder.isTypeSupported()`와 녹음 후 실제 `mimeType`을 확인합니다. 서버 업로드 확장자에 `.webm`, `.mp4`, `.ogg`가 모두 포함되어 있습니다. 지원하지 않는 MIME은 오류 메시지를 표시합니다. 파일 이름만 `.wav`로 바꾼다고 실제 음성 포맷이 WAV로 변하는 것은 아닙니다.

**보안 조건:** `getUserMedia()`는 `http://localhost`/`http://127.0.0.1`처럼 브라우저가 안전하다고 취급하는 로컬 개발 환경 또는 HTTPS에서 사용합니다. 다른 PC의 `http://192.168...`로 접근할 때는 HTTPS가 필요할 수 있습니다.

## PART 5. CSS 만들기

**`static/css/style.css` — 파일 내용**

```css
body{font-family:system-ui,sans-serif;background:#f4f7fb;color:#142234;margin:0}
.container{max-width:800px;margin:32px auto;padding:0 20px}
.card{background:#fff;border:1px solid #dce3ee;border-radius:14px;padding:22px;margin:20px 0;box-shadow:0 4px 12px #1422340a}
button{border:0;border-radius:8px;background:#245db9;color:white;padding:12px 16px;cursor:pointer;margin:3px}
button:disabled{opacity:.5;cursor:not-allowed}
input,select{padding:8px;border:1px solid #ccd6e4;border-radius:8px}p{line-height:1.65}
```

## PART 6. `main.py`를 수정합니다

`main.py`는 **라우터·정적 파일·Jinja2 템플릿을 모두 연결하는 FastAPI 진입점**입니다. 이 시점에는 `speech/router.py`, `templates`, `static` 폴더가 모두 존재하므로 import 및 mount 오류가 나지 않습니다.

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

### 경로 정리

| 요청 | 기능 |
|---|---|
| `GET /` | `/record/`로 이동 |
| `GET /record/` | 마이크 녹음 HTML 화면 표시 |
| `POST /api/stt-to-ko/` | 02에서 만든 STT·번역 API 호출 |
| `GET /static/js/record.js` | 녹음 JavaScript 파일 |
| `GET /health` | 서버 기동 확인 |

## PART 7. 실행과 브라우저 테스트

```bash
# 터미널 1: 프로젝트 루트에서
cd ~/fastapi_speech
source .venv/bin/activate
python -m uvicorn main:app --reload
```

1. 브라우저에서 `http://127.0.0.1:8000/record/`를 엽니다.
2. 마이크 권한을 허용합니다.
3. 5~10초 정도 한국어로 녹음한 뒤 `녹음 종료`를 누릅니다.
4. `src_lang=ko`로 전사 결과가 나타나는지 확인합니다.
5. 영어 문장을 녹음하고 `src_lang=en`으로 번역 상태를 확인합니다.
6. 보안/권한 오류가 발생하면 주소가 `localhost`/HTTPS인지와 OS 마이크 권한을 확인합니다.

### 웹 화면 자동 테스트

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

```bash
python -m pytest -q
```

이 테스트는 웹 HTML·JavaScript 파일이 응답되는지 확인합니다. **마이크 사용 허용 팝업과 실제 녹음 동작은 브라우저에서 수동 확인**해야 합니다.

## PART 8. JavaScript에서 실수하기 쉬운 부분

| 증상 | 원인과 수정 방향 |
|---|---|
| `Unexpected token '<'` | `<script>`에 HTML 태그가 들어 있음: HTML 파일로 이동 |
| `navigator.mediaDevices is undefined` | 안전하지 않은 HTTP/브라우저 권한 문제 |
| `TypeError: Failed to fetch` | FastAPI 서버가 꺼져 있거나 잘못된 URL |
| `400` | 서버가 지원하지 않는 파일 확장자/포맷, 30초 초과 등 |
| `413` | 20MB 초과 업로드 |
| 번역 문장이 영어 그대로임 | `translation_status` 확인 (`skipped`/`failed`) |
| 모델 로딩이 오래 걸림 | 첫 요청 때 다운로드/CPU 모델 초기화 중일 수 있음 |

## PART 9. 미니 챌린지와 자가 점검

- [ ] 마이크 녹음과 파일 업로드 두 방법 모두 전사에 성공한다.
- [ ] 25초가 되면 자동으로 녹음이 종료된다.
- [ ] 한국어·영어·혼합 음성을 서로 다른 `src_lang`으로 비교한다.
- [ ] 브라우저 Developer Tools → Network에서 API 요청/응답 JSON을 확인한다.
- [ ] 마이크 녹음은 JavaScript/브라우저, STT 추론은 Python/FastAPI 서버가 담당함을 설명할 수 있다.

> **다음 04 단계:** 작동하는 현재 코드를 유지하면서, 라우터에 섞여 있는 파일·언어·용량 검증을 `validators.py`라는 별도 파일로 이동합니다.

