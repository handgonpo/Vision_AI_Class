> **교재 개정 기준 · 2026-10-31**  
> 기존 Django 기반 학습 목적과 설명·이미지 참조 위치를 유지하면서 실행 코드와 폴더 구조를 FastAPI로 바꿨습니다. 학습 기준: Windows + WSL2(Ubuntu 24.04), Python 3.11/3.12, VS Code, `uv` 또는 `pip`.  
> `02 → 04 → 05 → 06`은 **한 프로젝트가 확장되는 연속 실습**입니다. 제공된 단계별 실행프로젝트는 해당 단계까지 완성한 참조 코드입니다. 교재 순서대로 생성하면 존재하지 않는 파일을 먼저 import하지 않습니다.  
> 모델 파일 다운로드와 Redis/Docker 실행은 인터넷과 사용자 PC 환경이 필요합니다. 검증 내역과 한계는 별도 `README_검증결과.md`를 확인하세요.


# 02. 감정분석 모델을 FastAPI에 달아보기 — 상세 실습

## 0. 전체 흐름 먼저 보기

```text
브라우저(HTML 폼) → FastAPI 라우터 → Hugging Face pipeline → label/score
                                                       ↓
                                              Jinja2 템플릿 출력
```

기존 설명에서 강조한 핵심은 **모델을 요청마다 새로 만들지 않고 서버 프로세스 안에서 재사용하는 것**입니다. `pipeline()`을 처음 호출할 때 모델을 받거나 캐시에서 불러오고 이후 같은 프로세스에서 공유합니다. FastAPI가 여러 워커 프로세스로 동작하면 프로세스마다 별도 인스턴스가 생깁니다.

## 1. 개발환경과 가상환경 준비

먼저 WSL Ubuntu 터미널에서 실행합니다. VS Code에서는 이 프로젝트의 폴더를 연 다음 새 터미널을 열어도 됩니다.

```bash
cd ~
mkdir -p fastapi_sentiment
cd fastapi_sentiment
code -r .
python3 --version
# uv 미설치 시 공식 설치 안내: https://docs.astral.sh/uv/getting-started/installation/
uv venv --python 3.12
source .venv/bin/activate
uv pip install "fastapi>=0.115,<1" "uvicorn[standard]" python-multipart jinja2 "transformers>=4.55,<5" torch httpx pytest
uv pip freeze > requirements.txt
```

`uv` 설치가 어렵다면 `python3 -m venv .venv`, `source .venv/bin/activate`, `python -m pip install ...`로 동일하게 준비할 수 있습니다. 기존 가상환경이 활성화되어 있다면 `deactivate` 후 새 환경을 만드세요. `torch`는 PyTorch 계산 엔진으로 크기가 크며 최초 다운로드에 시간이 걸릴 수 있습니다.

| 패키지 | 이유 |
|---|---|
| FastAPI | 라우팅·요청·응답 처리 |
| Uvicorn | ASGI 개발 서버 |
| python-multipart | HTML Form 값 해석 |
| Jinja2 | HTML 동적 렌더링 |
| transformers | Hugging Face 모델 호출 |
| torch | 모델 추론 |

## 2. 첫 번째 실행: 존재하는 파일만 사용합니다

### 2-1. 폴더 생성

```bash
mkdir -p sentiment templates/sentiment static/css tests
touch sentiment/__init__.py
```

현재는 빈 디렉터리만 생성됐습니다. **아직 `sentiment/router.py`가 없으므로 `main.py`에서 import하면 안 됩니다.**

### 2-2. `main.py` 최초 작성 — 서버 확인만

```python
from fastapi import FastAPI
app = FastAPI(title="FastAPI Sentiment Demo")

@app.get("/health", name="health")
def health():
    return {"status": "ok"}
```

### 2-3. 서버 첫 실행

```bash
python -m uvicorn main:app --reload
```

브라우저 / Insomnia: `GET http://127.0.0.1:8000/health` → `{"status":"ok"}`. `GET /docs`에서도 자동 API 문서를 확인할 수 있습니다. 여기서 `/predict/`는 아직 없으므로 접속하지 않습니다. 서버 터미널은 열어 두고 파일을 편집하거나 `Ctrl+C`로 종료합니다.

## 3. Hugging Face 모델을 로딩하는 파일

처음에는 다음 단순 코드로 모델 호출을 이해합니다. 이것은 독립적인 모델 테스트용이며 아직 FastAPI와 연결되지 않았습니다.

```python
from transformers import pipeline
classifier = pipeline("text-classification", model="tabularisai/multilingual-sentiment-analysis")
print(classifier("오늘 정말 기분이 좋아요"))
```

실행하여 모델 테스트를 합니다.

```bash
python model_test.py
```

실행 결과

```
Device set to use cuda:0
[{'label': 'Positive', 'score': 0.8310655951499939}]
```

그러나 이 코드를 요청 함수에 그대로 넣으면 매 요청마다 `pipeline()` 생성이 반복될 수 있습니다. 모델을 프로세스 내에서 재사용하도록 `get_classifier()`를 분리합니다.

아래 코드의 처리 순서

```text
1. 모델 ID를 상수로 저장한다.
2. get_classifier()가 첫 호출되면 모델을 생성한다.
3. 이미 생성됐으면 같은 모델 객체를 돌려준다.
4. 추론 결과가 리스트/중첩 리스트인지 확인한다.
5. 최종 예측을 {label, score} 딕셔너리로 통일한다.
```

**`sentiment/hf_model.py` — 신규 생성**

```python

# ============================================================
# ① 필요한 라이브러리 가져오기
# ============================================================
# lru_cache : 한 번 생성한 모델 객체를 재사용하는 캐시 기능
# pipeline  : Hugging Face의 사전학습 모델을 쉽게 사용하는 API

from functools import lru_cache
from transformers import pipeline


# ============================================================
# ② 사용할 Hugging Face 모델 지정
# ============================================================
# MODEL_ID : 감정분석에 사용할 사전학습 모델의 이름
# 모델 이름만 지정한 상태이며, 아직 로딩하지 않습니다.

MODEL_ID = "tabularisai/multilingual-sentiment-analysis"


# ============================================================
# ③ 모델을 한 번만 로딩하도록 캐시 설정
# ============================================================
# maxsize=1 : 모델 객체 1개를 캐시에 보관
# 첫 호출에는 모델을 로딩하고, 이후에는 기존 모델을 재사용합니다.
# 단, 캐시는 현재 Python 프로세스 안에서만 유지됩니다.

@lru_cache(maxsize=1)


# ============================================================
# ④ 감정분석 모델을 준비하는 함수
# ============================================================
def get_classifier():
    """
    Hugging Face 감정분석 모델을 준비하여 반환합니다.
    최초 호출 시 모델을 로딩하고 이후에는 캐시된 객체를 반환합니다.
    """

    # text-classification : 텍스트 분류 작업
    # model=MODEL_ID     : ②에서 지정한 모델 사용

    return pipeline(
        "text-classification",
        model=MODEL_ID
    )


# ============================================================
# ⑤ AI 분석 결과를 일정한 형식으로 정리하는 함수
# ============================================================
def normalize_prediction(output) -> dict:
    """
    Hugging Face 분석 결과에서 첫 번째 예측을 꺼내
    label과 score를 가진 딕셔너리로 반환합니다.
    """

    # --------------------------------------------------------
    # ⑥ 리스트 형태의 결과에서 첫 번째 예측 꺼내기
    # --------------------------------------------------------
    # 예: [{"label": "Positive", "score": 0.95}]
    # 리스트가 여러 번 중첩되어 있어도 첫 번째 값을 계속 꺼냅니다.

    while isinstance(output, list):

        # 리스트가 비어 있으면 오류 발생
        if not output:
            raise ValueError(
                "감정분석 결과가 비어 있습니다."
            )

        # 첫 번째 예측 결과 선택
        output = output[0]


    # --------------------------------------------------------
    # ⑦ 분석 결과 형식 검사
    # --------------------------------------------------------
    # 최종 결과가 딕셔너리인지 확인합니다.
    # label과 score 값이 정상적으로 포함되어 있어야 합니다.

    if (
        not isinstance(output, dict)
        or not output.get("label")
        or "score" not in output
    ):
        raise ValueError(
            "예상하지 못한 감정분석 결과 형식입니다."
        )


    # --------------------------------------------------------
    # ⑧ 최종 분석 결과 반환
    # --------------------------------------------------------
    # label : 감정 분류 결과를 문자열로 변환
    # score : 모델 점수를 실수형으로 변환
    # FastAPI에서 사용하기 편한 딕셔너리로 반환합니다.

    return {
        "label": str(output["label"]),
        "score": float(output["score"])
    }

```

**의사코드**

```
[① 필요한 라이브러리 가져오기]

    모델을 재사용할 lru_cache 가져오기
    Hugging Face pipeline 가져오기


          ↓

[② 감정분석 모델 지정]

    사용할 Hugging Face 모델 이름 저장
    MODEL_ID 설정


          ↓

[③ 모델 캐시 설정]

    모델 객체를 1개만 보관하도록 설정
    이미 로딩된 모델은 재사용


          ↓

[④ get_classifier() 함수]

    모델이 처음 요청되었다면
        Hugging Face 모델 로딩
        모델 객체를 캐시에 저장

    이미 로딩된 모델이 있다면
        저장된 모델 객체 재사용

    모델 객체 반환


          ↓

[FastAPI에서 실제 감정분석 실행]

    사용자가 문장을 입력

    classifier = get_classifier()

    output = classifier(입력 문장)

    normalize_prediction(output) 호출


          ↓

[⑤ normalize_prediction() 함수]

    AI 분석 결과를 입력받음


          ↓

[⑥ 리스트 결과 처리]

    결과가 리스트라면
        리스트가 비어 있는지 확인

        비어 있다면
            오류 발생

        비어 있지 않다면
            첫 번째 예측 결과 꺼내기

        리스트가 중첩되어 있으면
            같은 작업 반복


          ↓

[⑦ 결과 형식 검사]

    결과가 딕셔너리인지 검사
    label이 존재하는지 검사
    score가 존재하는지 검사

    잘못된 결과라면
        오류 발생


          ↓

[⑧ 최종 결과 반환]

    label을 문자열로 변환
    score를 실수형으로 변환

    {
        "label": "Positive",
        "score": 0.95
    }

    FastAPI에 결과 전달
```


이 수정본은 원본의 전역 `_classifier=None`과 `global` 대신 `@lru_cache(maxsize=1)`을 사용합니다. **동일한 지연 로딩·재사용 원리를 더 짧게 표현한 것입니다.** `hf_model.py`는 모델을 보관·반환하고, `router.py`는 HTTP 요청만 처리하게 책임을 나눕니다.

`pipeline`을 실행하면 로컬에 없는 모델·토크나이저를 다운로드하고, 학습된 가중치를 메모리에 적재합니다. `MODEL_ID`를 문자열 변수로 선언하는 것만으로 모델 다운로드가 시작되지는 않습니다. 현재 모델 호출은 학습이 아니라 **추론**이며, 입력 문장은 사용자가 입력한 데이터이지 학습 데이터가 아닙니다.

**원본 설명의 핵심 비유:** FastAPI 서버는 사무실, 라우터는 전화를 받는 직원, `hf_model.py`는 AI 비서를 관리하는 장소, `pipeline`은 실제 계산을 수행하는 비서입니다. 전화가 올 때마다 비서를 새로 채용하지 않고 이미 준비한 모델을 사용합니다.

## 4. 입력값 검증 + Jinja2 렌더링 라우터

### 4-1. `sentiment/router.py`를 지금 생성합니다

사용자의 POST 요청에서 `text`를 꺼내고 1~500자 검증 후 감정분석합니다. FastAPI의 `Form`은 폼 값을 받아오고 `TemplateResponse`는 템플릿에 결과를 전달합니다.

**라우터 코드처리순서**

```text
GET /predict/ 요청 → 빈 입력 화면 렌더링
POST /predict/ 요청 → text 추출 → 공백 제거
  ├ 유효하지 않음: 400 오류 및 입력 화면
  └ 유효함: get_classifier() → classifier(text)
          → normalize_prediction() → HTML 결과 표시
```

**`sentiment/router.py` — 신규 생성**

```python

# ============================================================
# ① 필요한 라이브러리 가져오기
# ============================================================
# APIRouter      : URL과 처리 함수를 연결
# Form           : HTML 폼의 입력값 받기
# Request        : 브라우저의 HTTP 요청 정보
# HTMLResponse   : HTML 응답 형식 지정
# Jinja2Templates: HTML 템플릿을 화면에 출력

from fastapi import APIRouter, Form, Request
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates

# 앞에서 작성한 hf_model.py의 함수 가져오기
from .hf_model import get_classifier, normalize_prediction


# ============================================================
# ② 라우터와 HTML 템플릿 설정
# ============================================================
# router    : 감정분석 URL을 등록하는 객체
# templates : HTML 파일이 저장된 폴더 지정

router = APIRouter()
templates = Jinja2Templates(directory="templates")


# ============================================================
# ③ GET /predict/ : 감정분석 입력 화면 열기
# ============================================================
# 사용자가 브라우저에서 /predict/에 접속하면 실행됩니다.
# GET은 데이터를 조회하거나 화면을 요청할 때 사용합니다.

@router.get(
    "/predict/",
    response_class=HTMLResponse,
    name="predict_page"
)
def predict_page(request: Request):

    # --------------------------------------------------------
    # ④ 입력 화면에 필요한 초기값 전달
    # --------------------------------------------------------
    # text   : 처음에는 입력된 문장이 없음
    # result : 아직 분석 결과가 없음
    # error  : 오류 메시지가 없음

    return templates.TemplateResponse(
        request=request,
        name="sentiment/predict.html",
        context={
            "text": "",
            "result": None,
            "error": None
        }
    )


# ============================================================
# ⑤ POST /predict/ : 사용자가 입력한 문장 받기
# ============================================================
# HTML에서 [분석하기] 버튼을 누르면 실행됩니다.
# Form(default="")은 text 입력값을 가져옵니다.
# 입력값이 없으면 빈 문자열을 사용합니다.

@router.post(
    "/predict/",
    response_class=HTMLResponse,
    name="predict"
)
def predict(
    request: Request,
    text: str = Form(default="")
):

    # --------------------------------------------------------
    # ⑥ 입력 문장 정리
    # --------------------------------------------------------
    # strip() : 문장 앞뒤의 불필요한 공백 제거

    cleaned = text.strip()


    # --------------------------------------------------------
    # ⑦ 입력값 검사 (1~500자)
    # --------------------------------------------------------
    # 빈 문장이거나 500자를 초과하면 분석하지 않습니다.
    # 400 : 잘못된 입력 요청이라는 HTTP 상태 코드

    if not 1 <= len(cleaned) <= 500:

        return templates.TemplateResponse(
            request=request,
            name="sentiment/predict.html",
            context={
                "text": text,
                "result": None,
                "error": "문장은 1~500자로 입력하세요."
            },
            status_code=400
        )


    # --------------------------------------------------------
    # ⑧ Hugging Face 감정분석 모델 가져오기
    # --------------------------------------------------------
    # hf_model.py의 get_classifier() 호출
    # 처음에는 모델을 로딩하고 이후에는 재사용합니다.

    classifier = get_classifier()


    # --------------------------------------------------------
    # ⑨ AI 감정분석 실행
    # --------------------------------------------------------
    # cleaned : 분석할 문장
    # truncation=True : 토큰 길이가 길면 잘라서 처리
    # max_length=512  : 모델 입력의 최대 토큰 길이

    output = classifier(
        cleaned,
        truncation=True,
        max_length=512
    )


    # --------------------------------------------------------
    # ⑩ AI 분석 결과 정리
    # --------------------------------------------------------
    # normalize_prediction()으로 label과 score 추출
    # input에는 사용자가 입력한 문장을 저장합니다.
    # ** : 딕셔너리의 항목을 펼쳐서 합치는 문법

    result = {
        "input": cleaned,
        **normalize_prediction(output)
    }


    # --------------------------------------------------------
    # ⑪ 최종 분석 결과를 HTML 화면에 전달
    # --------------------------------------------------------
    # text   : 사용자가 입력한 문장
    # result : AI 분석 결과 (input, label, score)
    # error  : 오류가 없으므로 None
    # Jinja2가 전달받은 값을 HTML에 표시합니다.

    return templates.TemplateResponse(
        request=request,
        name="sentiment/predict.html",
        context={
            "text": cleaned,
            "result": result,
            "error": None
        }
    )
```


**의사코드**

```text
[① 필요한 라이브러리 가져오기]

    FastAPI의 APIRouter, Form, Request 가져오기
    HTMLResponse, Jinja2Templates 가져오기
    hf_model.py의 AI 관련 함수 가져오기

                  ↓

[② 라우터와 HTML 템플릿 설정]

    router 객체 생성
    templates 폴더 지정

                  ↓

        사용자가 /predict/ 접속
                  ↓
          어떤 HTTP 요청인가?
                  │
         ┌────────┴────────┐
         │                 │
        GET               POST
         │                 │
         ▼                 ▼

[③ 입력 화면 요청]     [⑤ 분석할 문장 받기]

    predict_page()         predict()
         │                 │
         ▼                 ▼

[④ 초기 화면 출력]     [⑥ 입력 문장 정리]

    text = ""              text.strip()
    result = None          앞뒤 공백 제거
    error = None                │
         │                      ▼
         │               [⑦ 입력값 검사]
         │
         │                   1~500자인가?
         │                      │
         │               ┌──────┴──────┐
         │               │             │
         │              아니요         예
         │               │             │
         │               ▼             ▼
         │         오류 메시지 출력  [⑧ 모델 가져오기]
         │         HTTP 400 응답
         │                           get_classifier()
         │                                │
         │                                ▼
         │                       [⑨ AI 감정분석 실행]
         │
         │                          classifier(cleaned)
         │                                │
         │                                ▼
         │                       [⑩ 분석 결과 정리]
         │
         │                       normalize_prediction()
         │
         │                       label, score 추출
         │                                │
         │                                ▼
         │                       [⑪ HTML 결과 전달]
         │
         │                       text   = 입력 문장
         │                       result = 분석 결과
         │                       error  = None
         │                                │
         └───────────────┬────────────────┘
                         ▼

              sentiment/predict.html

                브라우저 화면 출력
```

- `text.strip()`은 앞뒤 공백을 제거합니다.
- `Form(default="")`는 HTML에서 보낸 `text` 필드를 읽습니다.
- `max_length=512`는 토큰 길이를 제한합니다. 화면의 500자는 문자 길이 제한이므로 토큰 길이와 다릅니다.
- `normalize_prediction()`은 출력이 중첩 리스트여도 하나의 딕셔너리로 정리합니다.
- `result={"input":...}`에는 원본 입력·감정 라벨·점수를 넣어 Jinja2에 전달합니다.

## 5. `main.py` 최종 교체 — 라우터를 만든 뒤 연결합니다

이제 `router.py`가 실제로 존재하므로 안전하게 import할 수 있습니다. 2단계 코드에 덧붙이는 것이 아니라 **파일 전체를 교체**합니다.

**`main.py` — 신규 생성**

```python

# ============================================================
# ① FastAPI 실행에 필요한 라이브러리 가져오기
# ============================================================
# FastAPI     : 웹 API 애플리케이션을 만드는 클래스
# StaticFiles : CSS, JavaScript, 이미지 등 정적 파일 제공

from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles


# ============================================================
# ② 감정분석 라우터 가져오기
# ============================================================
# sentiment/router.py에서 생성한 router 객체를 가져옵니다.
# sentiment_router라는 이름으로 사용합니다.
#
# 주의: router.py 파일을 먼저 생성해야 import가 가능합니다.

from sentiment.router import router as sentiment_router


# ============================================================
# ③ FastAPI 애플리케이션 생성
# ============================================================
# app : FastAPI 서버에서 사용하는 애플리케이션 객체
# title : Swagger UI(/docs)에 표시되는 프로젝트 제목

app = FastAPI(
    title="FastAPI Sentiment Demo"
)


# ============================================================
# ④ 정적 파일 폴더 등록
# ============================================================
# /static : 브라우저에서 접근하는 URL 경로
# static  : 실제 프로젝트에 있는 폴더
# name    : Jinja2에서 URL을 생성할 때 사용하는 이름
#
# 예:
# /static/css/sentiment.css
#     → static/css/sentiment.css 파일 제공

app.mount(
    "/static",
    StaticFiles(directory="static"),
    name="static"
)


# ============================================================
# ⑤ 감정분석 라우터 등록
# ============================================================
# sentiment/router.py에 작성한 API 경로를
# FastAPI 애플리케이션에 연결합니다.
#
# 등록되는 경로:
# GET  /predict/ : 감정분석 입력 화면
# POST /predict/ : 감정분석 실행 및 결과 출력

app.include_router(
    sentiment_router
)


# ============================================================
# ⑥ 서버 상태 확인 API 등록
# ============================================================
# 사용자가 /health 주소로 GET 요청을 보내면
# health() 함수를 실행하도록 등록합니다.
#
# name="health" : 이 API 경로의 이름

@app.get(
    "/health",
    name="health"
)


# ============================================================
# ⑦ 서버 상태를 반환하는 함수
# ============================================================
# 서버가 HTTP 요청에 정상 응답하는지 확인합니다.
# 이 함수는 Redis, DB, AI 모델 상태까지 검사하지 않습니다.

def health():

    # --------------------------------------------------------
    # ⑧ JSON 형식으로 정상 상태 반환
    # --------------------------------------------------------
    # Python 딕셔너리를 FastAPI가 JSON으로 변환합니다.
    #
    # 결과: {"status": "ok"}

    return {
        "status": "ok"
    }
```

**의사코드**

```
[① FastAPI 라이브러리 가져오기]

    FastAPI 가져오기
    StaticFiles 가져오기

                ↓

[② 감정분석 라우터 가져오기]

    sentiment/router.py에서
    router 객체를 가져오기

    이름을 sentiment_router로 지정

                ↓

[③ FastAPI 애플리케이션 생성]

    app = FastAPI()

    프로젝트 제목 설정

                ↓

[④ 정적 파일 폴더 연결]

    URL /static 등록

    실제 static 폴더와 연결

    CSS, JavaScript, 이미지 제공 준비

                ↓

[⑤ 감정분석 라우터 등록]

    sentiment_router를 app에 연결

    GET  /predict/ 등록
    POST /predict/ 등록

                ↓

[⑥ 서버 상태 확인 API 등록]

    GET /health 주소 등록

    요청이 들어오면
        health() 함수 실행

                ↓

[⑦ health() 함수]

    서버 상태를 확인할 수 있도록
    응답 데이터 준비

                ↓

[⑧ JSON 데이터 반환]

    {
        "status": "ok"
    }


=====================================
       서버 실행 준비 완료
=====================================

터미널에서 실행

python -m uvicorn main:app --reload

                ↓

       Uvicorn 웹 서버 시작

                ↓

       브라우저 요청 기다리기

                ↓

       사용자가 URL 접속
                │
        ┌───────┴────────┐
        │                │
        ▼                ▼

    /health           /predict/
        │                │
        ▼                ▼

    health()       sentiment/router.py
        │                │
        ▼                ▼

  {"status":"ok"}  predict_page()
                   또는 predict()
        │                │
        ▼                ▼

    JSON 응답         HTML 응답
```

## 각 코드의 역할 정리

|순번|코드|핵심 역할|
|---|---|---|
|①|`FastAPI`, `StaticFiles`|서버와 정적 파일 기능 준비|
|②|`from sentiment.router import ...`|다른 파일에 작성한 라우터 가져오기|
|③|`app = FastAPI()`|FastAPI 애플리케이션 생성|
|④|`app.mount()`|CSS 등 정적 파일 URL 연결|
|⑤|`app.include_router()`|감정분석 API를 FastAPI에 등록|
|⑥|`@app.get("/health")`|서버 상태 확인 URL 등록|
|⑦|`def health()`|상태 확인 요청 처리|
|⑧|`return {"status": "ok"}`|JSON 응답 반환|

**중요:** `static/` 폴더는 2단계에서 생성했으므로 `StaticFiles` 등록 시 폴더 없음 오류가 나지 않습니다. 아직 HTML 파일이 없으므로 `/predict/` 테스트는 6단계 파일 작성 후 진행합니다.

## 6. HTML + Jinja2 및 CSS

```text
fastapi_sentiment/
├── main.py
├── sentiment/
│   ├── __init__.py
│   ├── hf_model.py
│   └── router.py
├── templates/
│   ├── base.html
│   └── sentiment/predict.html
├── static/css/sentiment.css
├── tests/test_web.py
└── requirements.txt
```

`{% extends %}`, `{% block %}`, `{% if %}`, `{% endif %}`는 **Jinja2 템플릿 문법**입니다. `{{ result.label }}`은 FastAPI에서 전달한 변수를 HTML로 표현합니다.

**`templates/base.html` — 신규 생성**

```html
<!doctype html>
<html lang="ko"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>{% block title %}감정분석{% endblock %}</title>
<link rel="stylesheet" href="{{ url_for('static', path='/css/sentiment.css') }}"></head>
<body><main class="container"><header><a href="/predict/">FastAPI Sentiment</a></header>
{% block content %}{% endblock %}</main></body></html>
```

**`templates/sentiment/predict.html` — 신규 생성**

```html
{% extends "base.html" %}
{% block title %}감정분석 테스트{% endblock %}
{% block content %}
<h1>감정분석 테스트</h1><p>문장을 입력하면 Hugging Face 모델이 label/score를 반환합니다.</p>
<form method="post" action="{{ url_for('predict') }}">
<label for="text">문장 입력</label><textarea id="text" name="text" rows="4" maxlength="500" placeholder="감정분석할 문장을 입력하세요">{{ text }}</textarea>
{% if error %}<p class="error">{{ error }}</p>{% endif %}
<button type="submit">분석하기</button><a class="btn" href="{{ url_for('predict_page') }}">초기화</a>
</form>
{% if result %}
<section class="card"><h2>결과</h2>
{% set label_lower = result.label|lower %}
{% if 'pos' in label_lower %}<span class="badge pos">{{ result.label }}</span>
{% elif 'neg' in label_lower %}<span class="badge neg">{{ result.label }}</span>
{% else %}<span class="badge neu">{{ result.label }}</span>{% endif %}
<p>입력: {{ result.input }}</p><p>라벨: {{ result.label }}</p><p>점수: {{ '%.4f'|format(result.score) }}</p></section>
{% endif %}
{% endblock %}
```

**`static/css/sentiment.css` — 신규 생성**

```css
*{box-sizing:border-box}body{font-family:system-ui,"Noto Sans KR",sans-serif;background:#0b1220;color:#f3f5fa;margin:0}a{color:#99c4ff} .container{max-width:880px;margin:40px auto;padding:24px}header{padding-bottom:22px;border-bottom:1px solid #637084} form,.card{background:#182338;border:1px solid #40516a;padding:24px;border-radius:14px;margin:16px 0}textarea{display:block;width:100%;background:#111d30;border:1px solid #60728b;color:white;border-radius:10px;padding:12px;margin:12px 0}button,.btn{display:inline-block;background:#93baff;color:#06101f;border:0;border-radius:8px;padding:10px 14px;cursor:pointer;text-decoration:none;margin:5px}.error{color:#ff9da6}.badge{display:inline-block;border-radius:16px;padding:5px 12px;border:1px solid #7c819b}.pos{color:#8fffbd}.neg{color:#ffafb4}.neu{color:#ffe39e}.meta{color:#adb9cb}.inline{display:inline}li{margin:12px 0}
```


## 7. 서버 실행 및 결과 확인

터미널에서 프로젝트 루트(`fastapi_sentiment`)인지 확인한 후:

```bash
source .venv/bin/activate
python -m uvicorn main:app --reload
```

브라우저에서 `http://127.0.0.1:8000/predict/` 접속 → 문장 입력 → **분석하기** 클릭. 최초 모델 로딩은 시간이 걸릴 수 있습니다. `/health`, `/docs`, `/static/css/sentiment.css`도 차례로 확인합니다.

예시 출력: 라벨 `Positive`, 점수 `0.8506`은 예시이며 정확한 결과는 실행한 모델의 버전과 입력에 따라 달라집니다. 점수는 항상 실제 세계의 정답 확률로 해석해서는 안 됩니다.

![[Pasted image 20260218153255.png]]

> **원본 이미지 참조 위치 유지:** 화면 확인 부분에 원래 Obsidian 이미지 임베드를 그대로 남겼습니다. 실제 이미지 파일은 별도 첨부가 없으면 MD만으로 표시되지 않을 수 있습니다.

### 7-1. 자동 테스트

테스트는 모델을 진짜 다운로드하지 않고 **가짜 예측 결과(Mock)**를 사용합니다. 이를 통해 라우팅·폼 입력·템플릿 렌더링이 제대로 작동하는지 빠르게 확인합니다.

**`tests/test_web.py` — 신규 생성**

```python

# ============================================================
# ① 자동 테스트에 필요한 라이브러리 가져오기
# ============================================================
# TestClient : 실제 웹 서버를 따로 켜지 않고 API 테스트
# app        : main.py에 생성한 FastAPI 애플리케이션
# router_mod : 테스트 중 감정분석 모델 호출을 임시 교체하기 위해 사용

from fastapi.testclient import TestClient
from main import app
import sentiment.router as router_mod


# ============================================================
# ② FastAPI 테스트 클라이언트 생성
# ============================================================
# client를 사용하여 GET / POST 요청을 테스트합니다.
# 별도로 Uvicorn 서버를 실행할 필요가 없습니다.

client = TestClient(app)


# ============================================================
# ③ 서버 상태 확인 API 테스트
# ============================================================
# GET /health 요청 시 정상적인 JSON을 반환하는지 검사

def test_health():

    # --------------------------------------------------------
    # ④ 서버 상태 응답 검사
    # --------------------------------------------------------
    # client.get() : GET 요청 전송
    # .json()      : 응답 JSON을 Python 딕셔너리로 변환
    # assert       : 예상 결과와 같은지 검사
    #
    # 예상 결과: {"status": "ok"}

    assert client.get("/health").json() == {
        "status": "ok"
    }


# ============================================================
# ⑤ 감정분석 입력 페이지와 잘못된 입력 테스트
# ============================================================
# 정상 GET 요청과 잘못된 POST 요청을 확인합니다.

def test_page_and_invalid():

    # --------------------------------------------------------
    # ⑥ GET /predict/ 입력 화면 테스트
    # --------------------------------------------------------
    # 정상적으로 HTML 화면이 열리면 HTTP 200 반환

    assert client.get(
        "/predict/"
    ).status_code == 200


    # --------------------------------------------------------
    # ⑦ 잘못된 문장 입력 테스트
    # --------------------------------------------------------
    # 공백만 입력한 문장은 유효하지 않습니다.
    # router.py의 입력값 검증 로직이 작동해야 합니다.
    #
    # 예상 결과: HTTP 400 Bad Request

    assert client.post(
        "/predict/",
        data={
            "text": " "
        }
    ).status_code == 400


# ============================================================
# ⑧ 정상 감정분석 요청 테스트
# ============================================================
# monkeypatch를 이용하여 실제 AI 모델을 가짜 모델로 교체
#
# 왜 가짜 모델을 사용할까요?
# - 실제 Hugging Face 모델 다운로드가 필요하지 않음
# - 테스트 시간이 짧음
# - 매번 동일한 결과로 API 기능을 검증할 수 있음

def test_predict(monkeypatch):

    # --------------------------------------------------------
    # ⑨ 실제 모델 호출을 가짜 모델로 교체
    # --------------------------------------------------------
    # router.py의 get_classifier() 함수를 임시 교체합니다.
    #
    # 가짜 모델은 입력 문장과 관계없이
    # Positive / 0.9 결과를 반환합니다.
    #
    # 테스트 종료 후 monkeypatch가 원래 함수를 복원합니다.

    monkeypatch.setattr(
        router_mod,
        "get_classifier",
        lambda: lambda text, **kwargs: [
            {
                "label": "Positive",
                "score": 0.9
            }
        ]
    )


    # --------------------------------------------------------
    # ⑩ 정상 POST 요청 및 분석 결과 확인
    # --------------------------------------------------------
    # "좋아요" 문장을 POST로 전달합니다.
    # FastAPI는 가짜 모델을 호출하고 HTML을 반환합니다.
    #
    # 검사 조건:
    # 1. 응답 상태 코드가 200인가?
    # 2. HTML에 "Positive" 문자열이 있는가?

    res = client.post(
        "/predict/",
        data={
            "text": "좋아요"
        }
    )

    assert res.status_code == 200
    assert "Positive" in res.text
```

**의사코드**

```
[① 필요한 테스트 라이브러리 가져오기]

    FastAPI TestClient 가져오기

    main.py의 app 가져오기

    sentiment/router.py 가져오기


                  ↓

[② 테스트 클라이언트 생성]

    client = TestClient(app)

    FastAPI 서버에 테스트 요청을
    보낼 준비 완료


                  ↓

====================================
       pytest 자동 테스트 시작
====================================

    pytest가 test_로 시작하는
    테스트 함수를 찾아 실행


                  ↓

[③ test_health() 실행]

    /health 주소로 GET 요청


                  ↓

[④ 서버 상태 응답 확인]

    JSON 결과 확인

    {"status": "ok"} 인가?

       ├─ 예    → PASS
       └─ 아니요 → FAIL


                  ↓

[⑤ test_page_and_invalid() 실행]

    감정분석 입력 화면과
    잘못된 입력을 검사


                  ↓

[⑥ 정상 페이지 요청]

    GET /predict/

    응답 상태 코드가 200인가?

       ├─ 예    → PASS
       └─ 아니요 → FAIL


                  ↓

[⑦ 잘못된 문장 입력]

    POST /predict/

    text = " "

    router.py에서 strip() 실행
    → 빈 문장으로 판단

    응답 상태 코드가 400인가?

       ├─ 예    → PASS
       └─ 아니요 → FAIL


                  ↓

[⑧ test_predict(monkeypatch) 실행]

    정상 감정분석 요청 테스트


                  ↓

[⑨ 실제 Hugging Face 모델을 임시 교체]

    get_classifier() 대신
    가짜 모델 함수 사용

    가짜 모델의 반환 결과

    [
        {
            "label": "Positive",
            "score": 0.9
        }
    ]


                  ↓

[⑩ 정상 문장으로 POST 요청]

    POST /predict/

    text = "좋아요"


                  ↓

    router.py의 predict() 실행


                  ↓

    가짜 감정분석 모델 호출


                  ↓

    normalize_prediction()
    label / score 정리


                  ↓

    predict.html 결과 반환


                  ↓

    두 가지 조건 검사

    ① HTTP 상태 코드 == 200
    ② HTML 안에 "Positive" 존재


                  ↓

       모두 만족하면 PASS
       하나라도 실패하면 FAIL


                  ↓

====================================
        자동 테스트 결과 출력
====================================
```

**테스트 실행하기**

```bash
python -m pytest -q
```

**02 완료 체크:** `/health` 200, `/predict/` GET 200, 빈 입력 POST 400, 정상 입력 POST 200, CSS 200. `04`에서는 **이 프로젝트를 확장하여 DB 저장·목록·상세·수정·삭제** 기능을 추가합니다.
