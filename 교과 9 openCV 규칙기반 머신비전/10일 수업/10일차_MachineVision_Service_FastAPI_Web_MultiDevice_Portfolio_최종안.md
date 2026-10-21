## 오늘의 목표

1~9일차에서는 머신비전 검사 문제를 정의하고, 필요한 기술을 선택하고, 결과를 검증하고, Version과 실패 원인까지 관리하는 과정을 반복했습니다.

10일차는 새로운 머신비전 알고리즘을 하나 더 배우는 날이 아닙니다.

오늘은 지금까지 만든 검사 결과를 **다른 프로그램과 다른 Device에서도 사용할 수 있는 작은 서비스 형태**로 연결해 봅니다.

오늘의 핵심 질문은 다음과 같습니다.

> **내 Python 검사 코드가 내 PC 터미널 안에서만 끝나는 것이 아니라, 다른 PC·Tablet·Mobile에서도 사용할 수 있는 서비스가 되려면 무엇이 필요할까요?**

오늘은 6일차에서 이미 검증한 OCR 검사 모듈을 대표 예제로 재사용합니다.

새로운 OCR Rule을 다시 만들거나 Threshold를 다시 조정하지 않습니다.

![[Pasted image 20260915135403.png]]

```text
6일차에서 이미 만든 OCR 검사
        ↓
Frozen Rule 재사용
        ↓
검사 결과를 Python Dictionary로 정리
        ↓
JSON
        ↓
FastAPI
        ↓
Web Dashboard
        ↓
PC / Tablet / Mobile
```

10일차가 끝나면 다음을 설명하고 직접 실행할 수 있으면 됩니다.

```text
- 검사 알고리즘과 서비스 프로그램의 역할 차이를 설명할 수 있다.
- 기존 Frozen Rule을 다시 학습하거나 수정하지 않고 서비스에서 재사용할 수 있다.
- 검사 결과를 JSON 구조로 만들 수 있다.
- FastAPI의 Request / Response 의미를 설명할 수 있다.
- GET / POST의 차이를 간단히 설명할 수 있다.
- Swagger Docs에서 API를 직접 테스트할 수 있다.
- Web 화면에서 검사 대상 이미지를 선택하고 검사 결과를 확인할 수 있다.
- Final OK / NG뿐 아니라 Reason, OCR Text, Rule Version을 함께 확인할 수 있다.
- 같은 FastAPI 서버에 다른 Device가 Client로 접속하는 구조를 설명할 수 있다.
- PC / Tablet / Mobile이 각각 검사 알고리즘을 가지고 있을 필요가 없다는 것을 설명할 수 있다.
- 서비스 실행 화면과 Architecture를 Portfolio 결과물로 정리할 수 있다.
- 교과14에서 Jetson과 산업용 Camera가 들어오면 어떤 부분이 바뀌고 어떤 부분은 그대로 재사용되는지 설명할 수 있다.
```

> 10일차의 핵심은 **새로운 검사를 다시 만드는 것**이 아니라,  
> **이미 만든 검사 기능을 다른 프로그램과 Device가 사용할 수 있는 형태로 제공하는 것**입니다.

---

# 1. 1~9일차와 10일차를 한 번에 연결합니다

지금까지의 수업을 아주 짧게 다시 연결해 봅니다.

```text
1일차
Camera / Image / ROI
→ 무엇을 검사할지 정하고 검사할 영역을 선택

        ↓

2일차
Brightness / Contrast / Gamma / CLAHE / Filter
→ 검사하기 좋은 영상 상태인지 확인하고 필요한 전처리 선택

        ↓

3일차
Threshold / Morphology / Contour / Rule
→ 이미지 특징을 숫자로 측정하고 Rule로 OK / NG 판정

        ↓

4일차
Template / ORB / SIFT / Homography
→ 로고·라벨처럼 모양이 중요한 대상을 기준 이미지와 비교

        ↓

5일차
Calibration / Threshold / Rule Freeze / Test
→ 여러 데이터로 검사 기준을 만들고 처음 보는 Test에서 검증

        ↓

6일차
OCR / Regex / Expected Value
→ 제품에 인쇄된 문자를 읽고 형식과 기대값으로 검사

        ↓

7일차
Normal Reference / Anomaly Detection
→ 정의하기 어려운 이상을 정상 기준과의 차이로 검사

        ↓

8일차
One-shot / Few-shot
→ 데이터가 매우 적을 때 사용할 수 있는 검사 전략 비교

        ↓

9일차
Version / Reproducibility / Experiment / Drift / Hybrid
→ 결과의 출처를 기록하고, 한 변수 비교와 환경변화,
  여러 검사 결과 통합을 경험

        ↓

10일차
FastAPI / Web / Multi-Device / Portfolio
→ 지금까지 만든 검사 결과를
  다른 프로그램과 Device가 사용할 수 있는 서비스로 연결
```

10일차에서는 1~9일차 코드를 모두 하나의 거대한 프로그램으로 합치지 않습니다.

```text
Day01 코드
+
Day02 코드
+
...
+
Day09 코드

전부 연결
X
```

대신 다음 원칙을 사용합니다.

```text
검사 알고리즘은 이미 존재
        ↓
그중 하나의 안정된 모듈을 선택
        ↓
입력과 출력 형식을 정리
        ↓
API로 제공
        ↓
다른 Client에서 사용
```

오늘 대표 예제로 6일차 OCR을 선택하는 이유는 단순합니다.

```text
입력
→ 제품 이미지

실제 검사
→ OCR + Regex + Expected Value

출력
→ OK / NG + Reason
```

처럼 입력과 출력이 분명하여 서비스 구조를 배우기 좋기 때문입니다.

> 오늘의 목적은 OCR을 다시 배우는 것이 아닙니다.  
> OCR은 이미 완성된 검사 모듈의 예로 사용합니다.

---

# 2. 9일차와 10일차의 역할을 구분합니다

9일차에서는 다음을 배웠습니다.

```text
검사 결과가 있다
        ↓
어떤 Dataset인가?
어떤 Config인가?
어떤 Artifact인가?
어떤 Code Version인가?
        ↓
Version / Manifest
        ↓
Experiment
Drift
Hybrid
```

즉 9일차의 질문은:

```text
"이 검사 결과를
어떻게 관리하고 다시 설명할 것인가?"
```

였습니다.

10일차에서는 질문이 달라집니다.

```text
"이 검사 결과를
다른 프로그램이나 다른 Device가
어떻게 사용할 수 있게 할 것인가?"
```

오늘의 흐름은 다음과 같습니다.

```text
기존 Frozen 검사 모듈
        ↓
Service Core
        ↓
표준 Result
        ↓
JSON
        ↓
FastAPI
        ↓
HTTP
        ↓
Web Dashboard
        ↓
PC / Tablet / Mobile
```

---

# 3. 오늘 만들 최종 결과를 먼저 봅니다

오늘 수업이 끝나면 최종 구조는 다음과 같습니다.

![[Pasted image 20260915143221.png]]

```text
                    [ Inspection Server ]

제품 이미지
    ↓
Day06 OCR Frozen Rule
    ↓
SN ROI
    ↓
Tesseract OCR
    ↓
Regex
    ↓
Expected Value
    ↓
Final OK / NG + Reason
    ↓
Python Dictionary
    ↓
JSON
    ↓
FastAPI
    ↓
HTTP API
    ↓
────────────────────────────────────────────
        ↓               ↓               ↓
     PC Browser      Tablet          Mobile
        ↓               ↓               ↓
             같은 Web Dashboard
```

중요한 점은 이것입니다.

```text
Mobile 안에
OpenCV 설치 필요 X

Tablet 안에
Tesseract 설치 필요 X

다른 PC에
검사 Python 코드 복사 필요 X
```

검사 프로그램은 Server에서 실행됩니다.

다른 Device는 결과를 요청하고 화면에 표시하는 Client 역할을 합니다.

---

# 4. Server와 Client를 쉽게 이해합니다

## 4.1 Server

오늘의 Server는 다음 일을 합니다.

```text
검사 요청 받기
        ↓
검사 코드 실행
        ↓
결과 만들기
        ↓
JSON으로 응답
```

오늘은 FastAPI가 Server 프로그램의 역할을 합니다.

## 4.2 Client

Client는 Server에게 요청을 보내는 쪽입니다.

예:

```text
Chrome Browser
Edge Browser
Tablet Browser
Mobile Browser
다른 Python Program
```

오늘 Web 화면도 Client입니다.

## 4.3 실제 구조

![[Pasted image 20260915143537.png]]

```text
Client

"이 제품 검사해 주세요."
        ↓
HTTP Request
        ↓
FastAPI Server
        ↓
Machine Vision / OCR 검사
        ↓
HTTP Response
        ↓
{
  "final_decision": "NG",
  "reasons": ["NG_EXPECTED_MISMATCH"]
}
        ↓
Client 화면에 표시
```

---

# 5. API는 무엇인가요?

`API`는 프로그램끼리 기능과 데이터를 주고받기 위한 약속입니다.

오늘은 어렵게 생각하지 않습니다.

```text
사람
→ Web 화면 사용

Web 화면
→ API 호출

API
→ Python 검사 함수 실행

검사 함수
→ 결과 반환

API
→ JSON으로 Web에 전달
```

예를 들어:

```text
GET /api/health
```

는:

> "Server가 정상적으로 살아 있나요?"

를 묻는 주소입니다.

```text
POST /api/inspect
```

는:

> "이 제품을 검사해 주세요."

라는 요청입니다.

---

# 6. GET과 POST는 오늘 이 정도만 이해합니다

## GET

주로 정보를 읽을 때 사용합니다.

```text
GET /api/health
→ Server 상태 확인

GET /api/samples
→ 검사 가능한 Sample 목록 확인

GET /api/history
→ 최근 검사 기록 확인

GET /api/stats
→ OK / NG 통계 확인
```

## POST

Server에 데이터를 전달하여 어떤 작업을 요청할 때 사용합니다.

```text
POST /api/inspect

{
  "file_name": "0679256889404_side.png"
}
```

쉽게 말하면:

```text
GET
→ 보여 주세요.

POST
→ 이 입력으로 작업해 주세요.
```

정도로 이해하면 됩니다.

---

# 7. JSON은 왜 사용하나요?

지금까지 Python 프로그램 안에서는 검사 결과를 다음과 같은 `Dictionary`로 다룰 수 있었습니다.

```python
result = {
    "final_decision": "NG",
    "ocr_text": "0679256889404",
    "expected_text": "0679256889400",
    "reasons": [
        "NG_EXPECTED_MISMATCH"
    ],
}
```

그런데 오늘은 검사 결과를 Python 프로그램 안에서만 사용하는 것이 아닙니다.

```text
Python 검사 프로그램
        ↓
FastAPI
        ↓
Web Browser
        ↓
PC / Tablet / Mobile
```

처럼 **서로 다른 프로그램과 Device 사이에서 데이터를 주고받아야 합니다.**

이때 중요한 문제가 하나 생깁니다.

> Python의 Dictionary를 JavaScript나 다른 언어가 그대로 이해할 수 있을까요?

그대로 사용할 수는 없습니다.

---

## 7.1 Dictionary는 Python 내부에서 사용하는 자료형입니다

`dict`는 Python이 사용하는 자료구조입니다.

예:

```python
result = {
    "final_decision": "NG",
    "score": 0.92,
    "reasons": [
        "NG_EXPECTED_MISMATCH"
    ],
}
```

Python 프로그램 안에서는 매우 편리합니다.

```text
Python
        ↓
Dictionary 생성
        ↓
result["final_decision"]
        ↓
"NG"
```

하지만 `Dictionary` 자체는 **Python 메모리 안에 존재하는 Python 객체**입니다.

다른 프로그램이 Python의 메모리 객체를 그대로 가져가 사용할 수는 없습니다.

예를 들어 Web Browser에서 주로 사용하는 JavaScript에게:

```text
Python dict 객체를 그대로 전달
```

한다고 해서 자동으로 이해되는 것은 아닙니다.

그래서 프로그램끼리 데이터를 주고받을 때는  
서로 약속된 **공통 데이터 형식**이 필요합니다.

---

## 7.2 그래서 JSON 같은 공통 형식을 사용합니다

`JSON`은:

```text
JavaScript Object Notation
```

의 약자입니다.

이름에 JavaScript가 들어 있지만 현재는 JavaScript만 사용하는 형식이 아닙니다.

다음과 같은 다양한 환경에서 사용할 수 있습니다.

```text
Python
JavaScript
Java
C#
C++
Go
Mobile App
Web Browser
Server
```

즉 JSON은 쉽게 말하면:

> **서로 다른 프로그램이 데이터를 주고받기 위해 사용할 수 있는 공통 표현 형식**

입니다.

---

## 7.3 Dictionary와 JSON은 비슷하게 생겼습니다

Python Dictionary:

```python
{
    "final_decision": "NG",
    "ocr_text": "0679256889404",
    "expected_text": "0679256889400",
    "reasons": [
        "NG_EXPECTED_MISMATCH"
    ],
}
```

JSON:

```json
{
  "final_decision": "NG",
  "ocr_text": "0679256889404",
  "expected_text": "0679256889400",
  "reasons": [
    "NG_EXPECTED_MISMATCH"
  ]
}
```

겉으로 보면 매우 비슷합니다.

둘 다 기본적으로:

```text
Key : Value
```

형태를 사용하고,

```text
문자열
숫자
배열/List
중첩 구조
True / False와 비슷한 Boolean 값
빈 값
```

등을 표현할 수 있습니다.

그래서 Python Dictionary의 검사 결과를 JSON으로 변환하기가 편리합니다.

```text
Python Dictionary

{
    "final_decision": "NG",
    "score": 0.92
}

        ↓
JSON으로 변환

{
  "final_decision": "NG",
  "score": 0.92
}

        ↓
HTTP로 다른 프로그램에 전달
```

이처럼 **구조가 비슷하기 때문에 변환과 Parsing이 비교적 단순**합니다.

---

## 7.4 하지만 Dictionary와 JSON은 같은 것은 아닙니다

모양이 비슷하다고 같은 데이터 형식은 아닙니다.

가장 중요한 차이는 다음과 같습니다.

```text
Python Dictionary
→ Python 프로그램 내부에서 사용하는 객체

JSON
→ 프로그램 사이에서 주고받을 수 있도록 표현한 데이터 형식
```

예를 들어 Python에서는:

```python
{
    "pattern_ok": True,
    "result": None,
}
```

처럼 작성합니다.

JSON에서는:

```json
{
  "pattern_ok": true,
  "result": null
}
```

처럼 표현합니다.

차이가 보입니다.

```text
Python          JSON

True      →     true

False     →     false

None      →     null
```

따라서:

> **Dictionary와 JSON은 구조는 비슷하지만 서로 다른 것입니다.**

---

## 7.5 `Parsing`은 무엇인가요?

다른 프로그램에서 JSON을 받으면 단순한 글자로만 보는 것이 아니라  
프로그램에서 사용할 수 있는 구조로 해석해야 합니다.

이 과정을 흔히 `Parsing`이라고 합니다.

예를 들어 Server가 다음 JSON을 보냈다고 생각해 봅니다.

```json
{
  "final_decision": "NG",
  "reasons": [
    "NG_EXPECTED_MISMATCH"
  ]
}
```

Web Browser의 JavaScript에서는 이 데이터를 읽어서:

```text
final_decision
→ NG

reasons
→ NG_EXPECTED_MISMATCH
```

처럼 사용할 수 있습니다.

전체 흐름은 다음과 같습니다.

```text
Python 검사 결과

Dictionary
        ↓
JSON으로 변환
        ↓
HTTP Response
        ↓
Web Browser
        ↓
JSON Parsing
        ↓
JavaScript가 사용할 수 있는 Object
        ↓
화면에 표시

Final Result : NG

Reason :
NG_EXPECTED_MISMATCH
```

---

## 7.6 그러면 왜 API에서는 JSON을 많이 사용할까요?

Web API에서 JSON을 많이 사용하는 이유는 여러 가지입니다.

### 첫 번째 — 구조가 단순합니다

```json
{
  "result": "NG",
  "score": 0.92
}
```

처럼 사람이 읽기에도 비교적 쉽습니다.

---

### 두 번째 — Python Dictionary와 구조가 비슷합니다

우리가 이미 만든:

```python
result = {
    "result": "NG",
    "score": 0.92,
}
```

를 JSON 구조로 바꾸기 쉽습니다.

---

### 세 번째 — 여러 언어에서 지원합니다

```text
Python Server
        ↓
JSON
        ↓
JavaScript Web

Python Server
        ↓
JSON
        ↓
Mobile App

Python Server
        ↓
JSON
        ↓
다른 제조 시스템
```

처럼 서로 다른 언어로 만든 프로그램끼리도 데이터를 주고받기 좋습니다.

---

### 네 번째 — HTTP와 함께 사용하기 편합니다

Web API에서는 보통:

```text
HTTP Request
        ↓
Server
        ↓
HTTP Response
```

구조로 데이터를 주고받습니다.

이 Response 안에 JSON을 넣으면:

```text
HTTP
→ 데이터를 전달하는 방법

JSON
→ 전달할 데이터의 표현 형식
```

으로 역할을 나눌 수 있습니다.

---

## 7.7 REST API와 JSON은 같은 개념이 아닙니다

여기서 헷갈리기 쉬운 부분이 있습니다.

```text
REST
        ≠
JSON
```

입니다.

`REST`는 Web API를 어떻게 설계할 것인지에 대한 **Architecture Style**입니다.

예를 들어 오늘 사용하는:

```text
GET  /api/health

GET  /api/samples

POST /api/inspect

GET  /api/history
```

처럼:

```text
HTTP Method
+
Resource URL
```

을 이용하여 Client와 Server가 통신하도록 API를 설계할 수 있습니다.

반면 JSON은:

```text
그 API를 통해
어떤 형식으로 데이터를 주고받을 것인가?
```

와 관련된 것입니다.

따라서 다음처럼 이해하면 됩니다.

```text
REST / HTTP API
→ 어떻게 요청하고 응답할 것인가?

JSON
→ 그 요청과 응답 안에
   데이터를 어떤 형식으로 담을 것인가?
```

Web API에서는 이 두 가지가 함께 많이 사용되기 때문에  
`REST API + JSON` 구조를 자주 보게 됩니다.

하지만:

> **JSON을 사용했다고 자동으로 RESTful API가 되는 것은 아닙니다.**

---

## 7.8 오늘 FastAPI에서는 어떻게 연결될까요?

오늘 Python 검사 프로그램은 먼저 Dictionary를 만듭니다.

```python
result = {
    "final_decision": "NG",
    "ocr_text": "0679256889404",
    "expected_text": "0679256889400",
    "reasons": [
        "NG_EXPECTED_MISMATCH"
    ],
}
```

그리고 FastAPI의 함수에서:

```python
return result
```

처럼 반환할 수 있습니다.

FastAPI는 이처럼 JSON으로 표현 가능한 Python 데이터를  
HTTP Response로 전달할 수 있는 JSON 형태로 변환해 줍니다.

전체 흐름은 다음과 같습니다.

```text
Machine Vision / OCR 검사
        ↓

Python Dictionary

{
  검사 결과
}

        ↓

FastAPI

Python 데이터를
JSON Response로 변환

        ↓

HTTP Response

        ↓

Web Browser

JSON을 받아 Parsing

        ↓

화면 표시

Final : NG
Reason : NG_EXPECTED_MISMATCH
```

---

## 7.9 오늘 구조에서 각 기술의 역할을 구분합니다

```text
Python Dictionary
→ 검사 프로그램 내부에서 결과를 정리

JSON
→ 다른 프로그램에 전달할 수 있는 공통 데이터 형식

HTTP
→ Client와 Server 사이에서 데이터를 전달하는 통신 방법

FastAPI
→ Python 검사 기능을 HTTP API로 제공

JavaScript
→ JSON Response를 받아 Web 화면에 표시
```

따라서 오늘의 전체 구조는 다음과 같습니다.

```text
제품 이미지
        ↓
Python 검사 프로그램
        ↓
Dictionary
        ↓
FastAPI
        ↓
JSON Response
        ↓
HTTP
        ↓
PC / Tablet / Mobile Browser
        ↓
JSON Parsing
        ↓
Web Dashboard에 결과 표시
```

> **Dictionary는 Python 안에서 결과를 다루기 편한 구조이고, JSON은 그 결과를 Python 밖의 다른 프로그램과 주고받기 편한 공통 형식이라고 이해하면 됩니다.**

---

# 8. 오늘 수업의 전체 흐름

```text
PART1
전체 구조 이해
1~8
        ↓

PART2
Day10 Service 폴더 준비
+ 환경 확인
9~14
        ↓

PART3
기존 Day06 OCR을
Service Core로 연결
15~22
        ↓

PART4
검사 결과 JSON 구조 확인
+ Core 단독 Test
23~27
        ↓

PART5
FastAPI Server 만들기
28~36
        ↓

PART6
Swagger에서 API Test
37~40
        ↓

PART7
Web Dashboard 만들기
41~49
        ↓

PART8
다른 Device에서 사용
+ Network 이해
50~55
        ↓

PART9
Mini Challenge
+ 운영 관점 확인
56~59
        ↓

PART10
Portfolio 결과 정리
+ README + Git
+ 교과14 연결
60~68
```

오늘은 새 Dataset을 수집하지 않습니다.

6일차에서 이미 사용한 제품 이미지와 Frozen OCR Rule을 그대로 재사용합니다.

---

# Machine Vision Service · FastAPI · Web Dashboard: 파일별 역할 및 설명

### 1. `src/` — 실행 및 검증 파일

|파일명|역할|설명|
|---|---|---|
|`79_day09_run_hybrid_pipeline.py`|9일차 Hybrid 성능 검증|One-shot·Few-shot 결과를 OR·AND 규칙으로 결합하고 성능을 비교하는 9일차 마지막 파일입니다.|
|`80_day10_check_service_env.py`|서비스 환경 확인|FastAPI·OpenCV·Tesseract 등 필요한 실행환경과 6일차 OCR 이미지·설정 파일이 준비되어 있는지 확인합니다.|
|`81_day10_test_service_core.py`|OCR 검사 핵심 기능 검증|FastAPI를 실행하기 전에 `service_core.py`를 직접 호출하여 OCR 결과·Expected Text·OK/NG·판정 이유가 정상인지 확인합니다.|
|`82_day10_test_api.py`|FastAPI 통신 테스트|실행 중인 FastAPI 서버에 HTTP 요청을 보내 Sample 목록 조회와 OCR 검사 API가 정상 작동하는지 확인하고 결과를 JSON으로 저장합니다.|
|`83_day10_make_portfolio_summary.py`|포트폴리오 보고서 자동 생성|서비스 설정·OCR Rule·검사 이력을 읽어 프로젝트 구조, API 기능, OK/NG 검사 결과 등을 포트폴리오용 Markdown 문서로 정리합니다.|

### 2. `day10_service/` — 실제 서비스 구현 파일

| 파일명                 | 역할                  | 설명                                                                                        |
| ------------------- | ------------------- | ----------------------------------------------------------------------------------------- |
| `__init__.py`       | Python 패키지 구성       | `day10_service` 폴더를 Python 패키지로 사용하여 다른 Python 파일에서 내부 모듈을 가져올 수 있도록 합니다.                 |
| `service_core.py`   | 실제 OCR 검사 핵심 엔진     | 6일차의 Frozen OCR Rule을 재사용하여 제품 이미지의 SN을 읽고 형식과 예상 번호를 검사한 뒤 OK/NG·판정 이유·Version 등을 반환합니다. |
| `api_main.py`       | FastAPI 서버 및 API 제공 | 검사 함수를 HTTP API로 연결하고 Sample 목록·검사 결과·최근 기록·검사 통계 등을 브라우저나 다른 프로그램에 제공합니다.                |
| `static/index.html` | 웹 화면 구조             | 제품 선택, 이미지 미리보기, 검사 버튼, OCR 결과, OK/NG, 검사 이력 등을 표시할 Dashboard의 화면 요소를 만듭니다.               |
| `static/app.js`     | 웹 화면 동작 및 API 연동    | 검사 버튼 클릭, API 요청, 검사 결과 수신, OK/NG 표시, 최근 이력·통계 갱신 등 실제 화면 동작을 처리합니다.                      |
| `static/style.css`  | 웹 화면 디자인            | 검사 결과·버튼·표·이미지의 색상과 배치, 화면 크기에 따른 반응형 스타일을 적용하여 결과를 직관적으로 보여 줍니다.                         |
![[Pasted image 20261020100711.png]]

![[Pasted image 20261020101014.png]]

---

# PART2 — Day10 Service 폴더와 환경을 준비합니다 (9~14)

---

# 9. 기존 프로젝트를 그대로 엽니다

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

VS Code를 엽니다.

```bash
code -r .
```

10일차도 새 프로젝트와 새 `.venv`를 만들지 않습니다.

```text
Day01 ~ Day09
같은 프로젝트

        ↓

Day10
서비스 파일만 추가
```

---

# 10. 오늘 사용할 이전 결과를 확인합니다

오늘 핵심 실습에서는 6일차의 다음 파일을 사용합니다.

```text
samples/day06/product/

configs/day06_ocr_rules.json

configs/day06_labels.csv
```

각 역할은 다음과 같습니다.

```text
samples/day06/product/
→ 실제 검사할 제품 이미지

day06_ocr_rules.json
→ 6일차에서 고정한 OCR Rule
→ ROI / PSM / Whitelist / Regex / 전처리

day06_labels.csv
→ 교육용 Expected Value와 Ground Truth
```

오늘은 이 파일을 다시 만들어서는 안 됩니다.

특히:

```text
day06_ocr_rules.json
```

은 이미 6일차에서:

```text
day06-ocr-v1.0
```

으로 Freeze한 기준입니다.

---

# 11. Day10 폴더를 만듭니다

프로젝트 최상위에서 실행합니다.

```bash
mkdir -p day10_service/static
mkdir -p configs/day10
mkdir -p reports/day10
mkdir -p outputs/day10
```

오늘 추가할 구조는 다음과 같습니다.

```text
subject9_machinevision/
│
├── day10_service/
│   ├── __init__.py
│   ├── service_core.py
│   ├── api_main.py
│   │
│   └── static/
│       ├── index.html
│       ├── app.js
│       └── style.css
│
├── configs/
│   └── day10/
│       └── service_config.json
│
├── reports/
│   └── day10/
│       ├── service_history.json
│       ├── service_smoke_test.json
│       └── portfolio_summary.md
│
├── outputs/
│   └── day10/
│
└── src/
    ├── ...
    ├── 79_day09_run_hybrid_pipeline.py
    ├── 80_day10_check_service_env.py
    ├── 81_day10_test_service_core.py
    ├── 82_day10_test_api.py
    └── 83_day10_make_portfolio_summary.py
```

9일차의 마지막 번호가 붙은 핵심 실행 파일이 `79_day09_run_hybrid_pipeline.py`이므로 10일차 실행 파일은 `80`부터 시작합니다.

`service_core.py`와 `api_main.py`는 여러 요청에서 계속 사용하는 프로그램이므로 실행순서 번호를 붙이지 않습니다.

---

# 12. 필요한 Package를 설치합니다

오늘 새로 필요한 것은 FastAPI Server와 HTTP Test 도구입니다.

```bash
python -m pip install fastapi "uvicorn[standard]" httpx
```

설치 여부를 확인합니다.

```bash
python -c "import fastapi, uvicorn, httpx; print('Day10 Packages OK')"
```

정상이라면:

```text
Day10 Packages OK
```

가 출력됩니다.

현재 환경을 다시 기록합니다.

```bash
python -m pip freeze > requirements.txt
```

---

# 13. `service_config.json` — 검사 서비스의 기본 설정을 만듭니다

앞에서는 6일차에서 만든 OCR 검사 기준을 그대로 재사용하기로 했습니다.

이미 다음 파일이 있습니다.

```text
configs/day06_ocr_rules.json
```

이 파일에는:

```text
어떤 ROI를 사용할 것인가?

어떤 전처리를 사용할 것인가?

Tesseract PSM은 몇 번인가?

어떤 문자만 허용할 것인가?

어떤 Regex로 검사할 것인가?
```

같은 OCR 검사 기준이 들어 있습니다.

그런데 10일차에서는 OCR 검사뿐 아니라  
새로운 FastAPI 서비스 프로그램도 만들게 됩니다.

따라서 검사 기준과 서비스 설정을 분리해서 관리합니다.

```text
OCR 검사 기준
        ↓
day06_ocr_rules.json


FastAPI 서비스 설정
        ↓
service_config.json
```

---

## 13.1 왜 서비스 설정 파일이 따로 필요한가요?

예를 들어 우리가 만드는 서비스에는 다음과 같은 정보가 필요합니다.

```text
이 서비스의 이름은 무엇인가?

현재 서비스 Version은 무엇인가?

어떤 검사 모듈을 사용하고 있는가?

검사 이력을 몇 개까지 보관할 것인가?
```

이런 값들을 Python 코드 안에 직접 작성할 수도 있습니다.

예:

```python
SERVICE_NAME = "Subject9 Machine Vision Service"
SERVICE_VERSION = "day10-service-v1.0"
HISTORY_LIMIT = 100
```

하지만 이렇게 하면 설정을 바꿀 때마다 Python 코드를 수정해야 합니다.

```text
설정 변경
        ↓
Python 코드 수정
        ↓
잘못 수정하면 프로그램에도 영향
```

그래서 프로그램의 동작에 필요한 설정값을  
코드와 분리하여 별도의 JSON 파일에 저장합니다.

```text
Python Code
→ 프로그램이 어떻게 동작할지 작성

JSON Config
→ 프로그램이 어떤 설정으로 동작할지 기록
```

이렇게 역할을 나누는 것입니다.

---

## 13.2 오늘 만들 설정 파일의 역할

파일:

```text
configs/day10/service_config.json
```

을 만듭니다.

이 파일은 쉽게 말하면:

> “Day10 검사 서비스를 어떤 이름과 Version으로, 어떤 검사 모듈과 설정으로 실행할 것인지 적어 둔 서비스 설정표”

입니다.

---

## 13.3 `service_config.json`을 작성합니다

파일:

```text
configs/day10/service_config.json
```

작성합니다.

```json
{
  "service_name": "Subject9 Machine Vision Service",
  "service_version": "day10-service-v1.0",
  "inspection_module": "day06_ocr",
  "history_limit": 100,
  "allow_sample_inspection": true
}
```

---

## 13.4 각 항목은 무엇을 의미하나요?

### `service_name`

```json
"service_name": "Subject9 Machine Vision Service"
```

현재 우리가 만든 서비스의 이름입니다.

쉽게 말하면:

```text
"이 프로그램의 이름이 무엇인가?"
```

를 기록합니다.

예:

```text
Subject9 Machine Vision Service
```

나중에 여러 서비스가 생기면 어떤 프로그램인지 구분하는 데 사용할 수 있습니다.

---

### `service_version`

```json
"service_version": "day10-service-v1.0"
```

현재 서비스 프로그램의 Version입니다.

9일차에서 배운 Version 개념과 연결됩니다.

```text
처음 만든 서비스
→ day10-service-v1.0

        ↓

Web 기능 개선

        ↓

다음 Version
→ day10-service-v1.1
```

처럼 서비스 프로그램 자체의 변경 이력을 구분할 수 있습니다.

중요합니다.

```text
OCR Rule Version
        ≠
Service Version
```

입니다.

예를 들어:

```text
OCR Rule
day06-ocr-v1.0

Service
day10-service-v1.0
```

처럼 각각 별도로 관리합니다.

---

### `inspection_module`

```json
"inspection_module": "day06_ocr"
```

현재 이 서비스가 어떤 검사 기능을 사용하고 있는지 기록합니다.

오늘은 6일차 OCR 검사 모듈을 재사용하므로:

```text
day06_ocr
```

라고 기록합니다.

쉽게 말하면:

```text
FastAPI 서비스

        ↓

어떤 검사를 실행하나요?

        ↓

Day06 OCR 검사
```

라는 의미입니다.

나중에는 다음처럼 바뀔 수도 있습니다.

```text
day05_logo

day06_ocr

day07_anomaly

day09_hybrid
```

즉 `inspection_module`은:

> 현재 서비스가 어떤 검사 기능과 연결되어 있는지를 알려주는 이름

입니다.

---

### `history_limit`

```json
"history_limit": 100
```

검사 이력을 최대 몇 개까지 보관할지 정합니다.

오늘은 Database를 사용하지 않고:

```text
reports/day10/service_history.json
```

파일에 최근 검사 결과를 저장합니다.

제품이 계속 검사되면 결과가 계속 쌓입니다.

```text
제품 1
        ↓
제품 2
        ↓
제품 3
        ↓
...
        ↓
제품 101
        ↓
제품 102
```

모든 기록을 계속 저장하면 파일이 계속 커질 수 있습니다.

그래서 오늘은:

```text
최근 100개까지만 보관
```

하도록 정합니다.

```json
history_limit = 100
```

의 의미는:

> 최근 검사 결과를 최대 100건까지 유지한다

입니다.

---

### `allow_sample_inspection`

```json
"allow_sample_inspection": true
```

미리 준비된 Sample 이미지를 선택하여 검사할 수 있는 기능을 허용할지 정하는 설정입니다.

```
true
→ Sample 이미지 검사 허용

false
→ Sample 이미지 검사 비활성화
```

쉽게 말하면:

> Web 화면에서 준비된 제품 이미지를 골라 검사하는 기능을 사용할 것인지 정하는 값입니다.

단, 실제 Python 코드에서 이 값을 확인하도록 구현하지 않았다면 설정만 존재하고 동작에는 영향을 주지 않습니다.


---

## 13.5 두 JSON 파일의 역할을 확실히 구분합니다

지금 프로젝트에는 비슷해 보이는 JSON 파일이 두 개 있습니다.

### `day06_ocr_rules.json`

```text
OCR 검사를 어떻게 할 것인가?
```

를 정의합니다.

예:

```text
ROI

Preprocess Mode

PSM

Whitelist

Regex

OCR Rule Version
```

즉:

> 검사 결과 자체에 영향을 주는 기준

입니다.

---

### `service_config.json`

```text
검사 서비스를 어떻게 운영할 것인가?
```

를 정의합니다.

예:

```text
Service 이름

Service Version

사용할 검사 Module

History 보관 개수
```

즉:

> **FastAPI 서비스 프로그램의 실행 설정**

입니다.

---

## 13.6 그림으로 구분하면 다음과 같습니다

![[Pasted image 20260915170902.png]]

---

## 13.7 프로그램에서는 이 설정을 읽어서 사용합니다

뒤에서 Python 프로그램은 다음처럼 JSON 파일을 읽습니다.

```python
service_config = json.loads(
    SERVICE_CONFIG_PATH.read_text(
        encoding="utf-8"
    )
)
```

그러면 Python에서는 다음과 같이 사용할 수 있습니다.

```python
service_config["service_version"]
```

결과:

```text
day10-service-v1.0
```

또는:

```python
service_config["history_limit"]
```

결과:

```text
100
```

즉:

```text
JSON 설정 파일
        ↓
Python이 읽음
        ↓
Dictionary처럼 사용
        ↓
서비스 프로그램에서 설정 적용
```

의 흐름입니다.

이 부분은 앞에서 배운 JSON과 Dictionary의 관계와도 연결됩니다.

---

## 13.8 왜 코드와 설정을 분리하는지가 핵심입니다

오늘 기억해야 할 것은 JSON 문법 자체가 아닙니다.

```text
검사 기준
        ↓
Config 파일

서비스 설정
        ↓
Config 파일

실제 처리 방법
        ↓
Python Code
```

처럼 **코드와 설정을 분리하여 관리하는 사고방식**이 중요합니다.

나중에 서비스 이름이나 History 개수를 바꾸더라도:

```text
Python 코드 전체 수정
X

service_config.json의 설정값 변경
O
```

처럼 관리할 수 있습니다.

---

## 13.9 최종 정리

```text
day06_ocr_rules.json
→ 제품을 어떻게 검사할 것인가?
→ 검사 기준

service_config.json
→ 그 검사 기능을 어떤 서비스로 운영할 것인가?
→ 서비스 설정
```

한 문장으로 정리하면:

> `day06_ocr_rules.json`은 OCR 검사의 기준을 저장한 파일이고, `service_config.json`은 그 검사 기능을 FastAPI 서비스에서 어떤 이름과 Version, 운영 설정으로 사용할지 저장한 파일입니다.

---

# 14. `80_day10_check_service_env.py` — 시작 전 환경을 확인합니다

파일:

```text
src/80_day10_check_service_env.py
```

작성합니다.

```python
from pathlib import Path
import json
import shutil

import cv2
import fastapi
import httpx
import pytesseract
import uvicorn


PROJECT_ROOT = (
    Path(__file__)
    .resolve()
    .parent
    .parent
)

PRODUCT_DIR = (
    PROJECT_ROOT
    / "samples"
    / "day06"
    / "product"
)

OCR_RULE_PATH = (
    PROJECT_ROOT
    / "configs"
    / "day06_ocr_rules.json"
)

LABEL_PATH = (
    PROJECT_ROOT
    / "configs"
    / "day06_labels.csv"
)

SERVICE_CONFIG_PATH = (
    PROJECT_ROOT
    / "configs"
    / "day10"
    / "service_config.json"
)

REQUIRED_FILES = [
    OCR_RULE_PATH,
    LABEL_PATH,
    SERVICE_CONFIG_PATH,
]


print("=" * 70)
print("DAY10 SERVICE ENV CHECK")
print("=" * 70)

print("OpenCV   :", cv2.__version__)
print("FastAPI  :", fastapi.__version__)
print("Uvicorn  :", uvicorn.__version__)
print("HTTPX    :", httpx.__version__)
print("Tesseract:", shutil.which("tesseract"))

if shutil.which("tesseract") is None:
    raise RuntimeError(
        "Tesseract Engine이 없습니다."
    )


for path in REQUIRED_FILES:
    if not path.exists():
        raise FileNotFoundError(path)

    print("[OK]", path)


if not PRODUCT_DIR.exists():
    raise FileNotFoundError(PRODUCT_DIR)


image_paths = sorted(
    [
        path
        for path in PRODUCT_DIR.iterdir()
        if path.suffix.lower()
        in {".png", ".jpg", ".jpeg"}
    ]
)

if not image_paths:
    raise RuntimeError(
        "Day06 제품 이미지가 없습니다."
    )


ocr_rules = json.loads(
    OCR_RULE_PATH.read_text(
        encoding="utf-8"
    )
)

service_config = json.loads(
    SERVICE_CONFIG_PATH.read_text(
        encoding="utf-8"
    )
)


required_rule_keys = {
    "baseline_version",
    "roi_by_file",
    "preprocess_mode",
    "ocr_lang",
    "ocr_psm",
    "whitelist",
    "expected_pattern",
}

missing = (
    required_rule_keys
    - set(ocr_rules.keys())
)

if missing:
    raise RuntimeError(
        "Day06 OCR Rule Key가 부족합니다: "
        f"{sorted(missing)}"
    )


print()
print("Product Images :", len(image_paths))
print(
    "OCR Version    :",
    ocr_rules["baseline_version"],
)
print(
    "Service Version:",
    service_config["service_version"],
)
print(
    "Tesseract Ver  :",
    pytesseract.get_tesseract_version(),
)

print()
print("Day10 Service Environment OK")
```

실행합니다.

```bash
python src/80_day10_check_service_env.py
```

마지막에 다음 문장이 나오면 준비가 끝났습니다.

```text
Day10 Service Environment OK
```

### 의사코드

```text
FastAPI / Uvicorn / HTTPX를 불러온다
        ↓
Tesseract가 설치되어 있는지 확인한다
        ↓
Day06 제품 이미지 폴더를 확인한다
        ↓
Day06 Frozen OCR Rule을 확인한다
        ↓
Day06 Label CSV를 확인한다
        ↓
Day10 Service Config를 확인한다
        ↓
OCR Rule에 필요한 Key가 있는지 확인한다
        ↓
검사 이미지가 실제로 존재하는지 확인한다
        ↓
OCR Version과 Service Version을 출력한다
        ↓
모두 정상
        ↓
Day10 Service Environment OK
```

---

# PART3 — 기존 OCR 검사를 Service Core로 연결합니다 (15~22)

---

# 15. `service_core.py`는 무엇인가요?

이제부터 새로운 파일 두 개가 등장합니다.

```text
day10_service/service_core.py

day10_service/api_main.py
```

두 파일 모두 10일차에 새로 작성하는 파일입니다.

하지만 두 파일의 역할은 서로 다릅니다.

먼저 가장 중요한 점부터 확인합니다.

> `service_core.py`에서 새로운 OCR 검사 방법을 만드는 것은 아닙니다.

6일차에서 이미 다음 검사를 만들었습니다.

```text
제품 이미지
        ↓
SN ROI
        ↓
전처리
        ↓
Tesseract OCR
        ↓
문자열 정리
        ↓
Regex
13자리 숫자인가?
        ↓
Expected Value 비교
        ↓
OK / NG
```

10일차에서는 이 **6일차 검사 흐름을 그대로 재사용**합니다.

---

## 15.1 그러면 `service_core.py`는 왜 새로 만드나요?

6일차에서는 주로 Python 파일을 직접 실행했습니다.

예를 들어:

```text
Python 프로그램 실행
        ↓
제품 이미지 읽기
        ↓
OCR 검사
        ↓
Terminal / CSV에서 결과 확인
```

했습니다.

그런데 10일차에서는 검사 결과를 FastAPI와 Web에서 사용하려고 합니다.

그러려면 기존 검사 과정을:

```text
"파일을 실행하면 검사"

```

하는 형태보다:

```python
result = inspect_sample(file_name)
```

처럼 **필요할 때 호출할 수 있는 함수 형태**로 만드는 것이 편리합니다.

이 역할을 하는 새 파일이:

```text
service_core.py
```

입니다.

쉽게 말하면:

> 6일차에서 배운 OCR 검사 과정을 Web이나 API에서도 다시 사용할 수 있도록 하나의 검사 함수로 정리한 파일입니다.

---

## 15.2 `service_core.py` 안에는 무엇이 들어가나요?

10일차의 `service_core.py`에서는 새 검사법을 만드는 것이 아니라  
6일차에서 이미 사용했던 내용을 다시 연결합니다.

```text
6일차에서 만든 것

제품 이미지
   +
day06_ocr_rules.json
   +
day06_labels.csv

        ↓

10일차 service_core.py

이미지 읽기
        ↓
Day06 Rule 읽기
        ↓
SN ROI 자르기
        ↓
Day06에서 선택한 전처리 적용
        ↓
Tesseract OCR
        ↓
Regex 검사
        ↓
Expected Value 비교
        ↓
Final OK / NG
        ↓
Reason 생성
```

즉:

```text
새로운 OCR 알고리즘
X

6일차 OCR 검사 흐름 재사용
O
```

입니다.

---

## 15.3 그렇다면 `api_main.py`는 무엇인가요?

`api_main.py`도 10일차에 새로 만듭니다.

하지만 이 파일은 OCR을 직접 담당하지 않습니다.

역할은 다음과 같습니다.

```text
PC / Tablet / Mobile

"이 제품을 검사해 주세요."

        ↓

HTTP Request

        ↓

api_main.py
FastAPI가 요청을 받음

        ↓

service_core.py의
inspect_sample() 호출

        ↓

검사 결과 받음

        ↓

JSON Response

        ↓

PC / Tablet / Mobile
```

즉:

```text
service_core.py
→ 실제 제품 검사 담당

api_main.py
→ 외부에서 들어오는 요청과 응답 담당
```

입니다.

---

## 15.4 두 파일의 역할을 비교합니다

| 파일 | 쉽게 말하면 | 담당하는 일 |
|---|---|---|
| `service_core.py` | 검사 담당자 | 이미지 → OCR → Rule → OK/NG |
| `api_main.py` | 접수·전달 담당자 | HTTP 요청 받기 → 검사 함수 호출 → 결과 전달 |

예를 들어 식당에 비유하면 다음과 비슷합니다.

```text
손님
        ↓
주문

        ↓

직원
api_main.py

"주문을 받습니다."

        ↓

주방
service_core.py

"실제 음식을 만듭니다."

        ↓

직원
api_main.py

"완성된 결과를 전달합니다."

        ↓

손님
```

오늘의 검사 서비스도 비슷합니다.

```text
Client
        ↓
검사 요청

        ↓

api_main.py
요청 받기

        ↓

service_core.py
실제 OCR 검사

        ↓

api_main.py
JSON 결과 전달

        ↓

Client
결과 확인
```

---

## 15.5 실제 검사에서는 어떻게 연결되나요?

예를 들어 Web에서 사용자가 다음 제품을 선택했다고 생각합니다.

```text
0679256889404_side.png
```

Web이 FastAPI에 요청합니다.

```text
POST /api/inspect
```

그러면:

```text
api_main.py

"검사 요청이 들어왔다."

        ↓

service_core.py의

inspect_sample(
    "0679256889404_side.png"
)

호출

        ↓

service_core.py

제품 이미지 읽기
        ↓
Day06 Frozen Rule 읽기
        ↓
SN ROI
        ↓
Tesseract OCR
        ↓
Regex
        ↓
Expected Value
        ↓
Final OK / NG + Reason

        ↓

결과 Dictionary 반환

        ↓

api_main.py

Dictionary를 받아
JSON Response로 전달

        ↓

Web 화면
```

![[Pasted image 20260915174504.png]]

---

## 15.6 왜 검사 코드와 통신 코드를 나누나요?

물론 모든 코드를 `api_main.py` 하나에 작성할 수도 있습니다.

예:

```text
api_main.py

FastAPI
 +
이미지 읽기
 +
ROI
 +
OCR
 +
Regex
 +
Expected Value
 +
History
 +
JSON
```

하지만 이렇게 하면 코드가 너무 섞입니다.

```text
Web 통신 문제인가?

OCR 문제인가?

Rule 문제인가?

API 문제인가?
```

를 구분하기 어려워집니다.

그래서 역할을 나눕니다.

```text
service_core.py
        ↓
검사만 담당


api_main.py
        ↓
통신만 담당
```

이렇게 하면 나중에 오류가 발생했을 때도 원인을 찾기 쉽습니다.

---

## 15.7 Web이 없어도 `service_core.py`를 사용할 수 있습니다

```
web이란 HTML/JavaScript로 만든 Web Dashboard, 즉 브라우저에서 사용하는 화면을 뜻합니다.
```

`service_core.py`는 FastAPI 전용 파일이 아닙니다.

예를 들어 Python 코드에서 직접:

```python
from day10_service.service_core import inspect_sample

result = inspect_sample(
    "0637145699424_side.png"
)

print(result)
```

처럼 사용할 수도 있습니다.

즉:

![[Pasted image 20260915175803.png]]

```text
service_core.py
        ↓
실제 검사 기능

        ↓

┌──────────────────┬──────────────────┐
↓                  ↓                  ↓

FastAPI            Python Script      다른 프로그램

Web 서비스에서      직접 검사할 때       나중에 다른 시스템에서
사용                사용               사용 가능
```

여기서:

```text
FastAPI
Python Script
다른 프로그램
```

은 **디렉터리 구조가 아닙니다.**

> `service_core.py`에 만들어 둔 검사 기능을 여러 프로그램이 가져다 사용할 수 있다는 의미입니다.

---

## 15.8 `Batch Script`는 무엇을 뜻하나요?

여러 이미지를 한꺼번에 검사하는 Python 프로그램을 의미합니다.

예를 들어:

```text
제품 이미지 100장

        ↓

Python Batch 프로그램

        ↓

service_core.py의
inspect_sample()

100번 호출

        ↓

100개 검사 결과
```

처럼 사용할 수 있다는 뜻입니다.

즉 같은 `service_core.py`를:

```text
Web에서도 사용

여러 이미지 Batch 검사에서도 사용

다른 프로그램에서도 사용
```

할 수 있습니다.

![[Pasted image 20260915180106.png]]

---

## 15.9 오늘 실제로 만드는 구조를 다시 봅니다

```text
[6일차에서 이미 만든 검사 기준]

day06_ocr_rules.json
        +
day06_labels.csv
        +
제품 이미지

        ↓

[10일차에 새로 만드는 파일]

service_core.py
        ↓
6일차 OCR 검사 흐름을
호출 가능한 함수로 정리

inspect_sample()

        ↓

[10일차에 새로 만드는 파일]

api_main.py
        ↓
FastAPI로 HTTP 요청 받기
        ↓
inspect_sample() 호출
        ↓
검사 결과 받기
        ↓
JSON Response 전달

        ↓

PC / Tablet / Mobile
Web Browser에서 결과 확인
```

---

## 15.10 가장 쉽게 한 문장으로 정리합니다

```text
service_core.py
→ "실제로 검사하는 코드"

api_main.py
→ "검사 요청을 받고 결과를 전달하는 코드"
```

조금 더 정확히 말하면:

> `service_core.py`는 6일차에서 이미 검증한 OCR 검사 흐름을 10일차 서비스에서 재사용할 수 있도록 새로 정리하는 파일이고, 
> 
> `api_main.py`는 그 검사 함수를 FastAPI를 통해 PC·Tablet·Mobile 같은 외부 Client가 사용할 수 있도록 연결하는 파일입니다.

따라서 오늘 새로 배우는 것은:

```text
새 OCR 검사법
X

기존 검사 기능을
서비스용 구조로 분리하고 연결하는 방법
O
```

입니다.

---

# 16. 6일차 Frozen Rule을 다시 만들지 않습니다

오늘은 다음 과정을 반복하지 않습니다.

```text
ROI 다시 선택
전처리 다시 비교
Regex 다시 결정
PSM 다시 선택
Rule 다시 Freeze
```

`Frozen Rule`은 **그 과정을 거쳐 최종적으로 확정되어 더 이상 임의로 바꾸지 않기로 한 검사 기준 묶음**을 뜻합니다.

왜냐하면 오늘의 질문은:

```text
어떤 OCR Rule이 좋은가?
```

가 아니라:

```text
이미 만든 OCR 검사 결과를
어떻게 서비스로 제공할까?
```

이기 때문입니다.

---

# 17. `service_core.py`를 만듭니다

최종디렉토리 구조:

```
subject9_machinevision/
│
├── day10_service/
│   │
│   ├── __init__.py
│   │
│   ├── service_core.py
│   │   → 실제 OCR 검사 기능
│   │
│   ├── api_main.py
│   │   → FastAPI / HTTP 요청·응답
│   │
│   └── static/
│       │
│       ├── index.html
│       │   → Web 화면 구조
│       │
│       ├── app.js
│       │   → Web에서 API 호출
│       │
│       └── style.css
│           → Web 화면 디자인
│
├── outputs/
├── configs/
├── reports/
└── src/
```


파일:

```text
day10_service/service_core.py
```

작성합니다.

```python
from datetime import datetime
from pathlib import Path
import csv
import json
import re
import uuid

import cv2
import pytesseract


PROJECT_ROOT = (
    Path(__file__)
    .resolve()
    .parent
    .parent
)

PRODUCT_DIR = (
    PROJECT_ROOT
    / "samples"
    / "day06"
    / "product"
)

OCR_RULE_PATH = (
    PROJECT_ROOT
    / "configs"
    / "day06_ocr_rules.json"
)

LABEL_PATH = (
    PROJECT_ROOT
    / "configs"
    / "day06_labels.csv"
)

SERVICE_CONFIG_PATH = (
    PROJECT_ROOT
    / "configs"
    / "day10"
    / "service_config.json"
)

HISTORY_PATH = (
    PROJECT_ROOT
    / "reports"
    / "day10"
    / "service_history.json"
)


def load_json(path):
    if not path.exists():
        raise FileNotFoundError(path)

    return json.loads(
        path.read_text(
            encoding="utf-8"
        )
    )


def load_labels():
    if not LABEL_PATH.exists():
        raise FileNotFoundError(LABEL_PATH)

    with LABEL_PATH.open(
        "r",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        rows = list(
            csv.DictReader(file)
        )

    return {
        row["file"]: row
        for row in rows
    }


def list_samples():
    if not PRODUCT_DIR.exists():
        raise FileNotFoundError(PRODUCT_DIR)

    return sorted(
        [
            path.name
            for path in PRODUCT_DIR.iterdir()
            if path.suffix.lower()
            in {".png", ".jpg", ".jpeg"}
        ]
    )


def crop_roi(image, roi):
    x1, y1, x2, y2 = map(
        int,
        roi,
    )

    height, width = image.shape[:2]

    if not (
        0 <= x1 < x2 <= width
        and 0 <= y1 < y2 <= height
    ):
        raise ValueError(
            f"ROI가 이미지 범위를 벗어났습니다: {roi}"
        )

    cropped = image[
        y1:y2,
        x1:x2,
    ]

    if cropped.size == 0:
        raise RuntimeError(
            "OCR ROI가 비어 있습니다."
        )

    return cropped


def preprocess_for_ocr(
    roi,
    rules,
):
    mode = rules[
        "preprocess_mode"
    ]

    if mode == "original":
        processed = cv2.cvtColor(
            roi,
            cv2.COLOR_BGR2RGB,
        )

    elif mode == "gray":
        processed = cv2.cvtColor(
            roi,
            cv2.COLOR_BGR2GRAY,
        )

    elif mode == "clahe":
        gray = cv2.cvtColor(
            roi,
            cv2.COLOR_BGR2GRAY,
        )

        clip_limit = float(
            rules.get(
                "clahe_clip_limit",
                2.0,
            )
        )

        grid = tuple(
            rules.get(
                "clahe_grid",
                [8, 8],
            )
        )

        clahe = cv2.createCLAHE(
            clipLimit=clip_limit,
            tileGridSize=grid,
        )

        processed = clahe.apply(
            gray
        )

    elif mode == "otsu":
        gray = cv2.cvtColor(
            roi,
            cv2.COLOR_BGR2GRAY,
        )

        _, processed = cv2.threshold(
            gray,
            0,
            255,
            cv2.THRESH_BINARY
            + cv2.THRESH_OTSU,
        )

    else:
        raise ValueError(
            f"지원하지 않는 전처리: {mode}"
        )


    processed = cv2.resize(
        processed,
        None,
        fx=4.0,
        fy=4.0,
        interpolation=cv2.INTER_CUBIC,
    )

    return processed


def run_ocr(
    processed,
    rules,
):
    config = (
        f"--oem {int(rules.get('ocr_oem', 3))} "
        f"--psm {int(rules['ocr_psm'])} "
        f"-c tessedit_char_whitelist="
        f"{rules['whitelist']}"
    )

    raw_text = pytesseract.image_to_string(
        processed,
        lang=rules["ocr_lang"],
        config=config,
    )

    return raw_text


def inspect_sample(file_name):
    rules = load_json(
        OCR_RULE_PATH
    )

    service_config = load_json(
        SERVICE_CONFIG_PATH
    )

    labels = load_labels()


    if file_name not in list_samples():
        raise FileNotFoundError(
            f"등록되지 않은 Sample입니다: {file_name}"
        )

    if file_name not in labels:
        raise KeyError(
            f"Label 정보가 없습니다: {file_name}"
        )

    roi_map = rules[
        "roi_by_file"
    ]

    if file_name not in roi_map:
        raise KeyError(
            f"ROI 정보가 없습니다: {file_name}"
        )


    image_path = (
        PRODUCT_DIR
        / file_name
    )

    image = cv2.imread(
        str(image_path)
    )

    if image is None:
        raise FileNotFoundError(
            image_path
        )


    roi = crop_roi(
        image,
        roi_map[file_name],
    )

    processed = preprocess_for_ocr(
        roi,
        rules,
    )

    raw_text = run_ocr(
        processed,
        rules,
    )

    ocr_text = (
        raw_text
        .strip()
    )


    pattern_ok = bool(
        re.fullmatch(
            rules["expected_pattern"],
            ocr_text,
        )
    )


    expected_text = (
        labels[file_name][
            "expected_text"
        ]
        .strip()
    )

    expected_match = (
        ocr_text
        == expected_text
    )


    reasons = []

    if not ocr_text:
        reasons.append(
            "NG_OCR_EMPTY"
        )

    if not pattern_ok:
        reasons.append(
            "NG_FORMAT"
        )

    if not expected_match:
        reasons.append(
            "NG_EXPECTED_MISMATCH"
        )


    final_decision = (
        "OK"
        if pattern_ok
        and expected_match
        else "NG"
    )


    result = {
        "request_id":
            uuid.uuid4().hex[:12],

        "inspected_at":
            datetime.now().isoformat(
                timespec="seconds"
            ),

        "service_version":
            service_config[
                "service_version"
            ],

        "inspection_module":
            service_config[
                "inspection_module"
            ],

        "rule_version":
            rules[
                "baseline_version"
            ],

        "file_name":
            file_name,

        "measurements": {
            "ocr_text":
                ocr_text,

            "expected_text":
                expected_text,

            "pattern_ok":
                pattern_ok,

            "expected_match":
                expected_match,
        },

        "final_decision":
            final_decision,

        "reasons":
            reasons,
    }

    return result


def read_history():
    if not HISTORY_PATH.exists():
        return []

    return json.loads(
        HISTORY_PATH.read_text(
            encoding="utf-8"
        )
    )


def append_history(result):
    history = read_history()

    history.append(
        result
    )

    service_config = load_json(
        SERVICE_CONFIG_PATH
    )

    history_limit = int(
        service_config.get(
            "history_limit",
            100,
        )
    )

    history = history[
        -history_limit:
    ]

    HISTORY_PATH.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    HISTORY_PATH.write_text(
        json.dumps(
            history,
            indent=2,
            ensure_ascii=False,
        ),
        encoding="utf-8",
    )


def latest_result():
    history = read_history()

    if not history:
        return None

    return history[-1]


def stats():
    history = read_history()

    ok_count = sum(
        1
        for row in history
        if row["final_decision"]
        == "OK"
    )

    ng_count = sum(
        1
        for row in history
        if row["final_decision"]
        == "NG"
    )

    return {
        "total":
            len(history),

        "ok":
            ok_count,

        "ng":
            ng_count,
    }
```

---

# 18. `service_core.py`의 처리 흐름을 읽어봅니다

코드의 실행 흐름을 위에서 아래로 읽으면 다음과 같습니다.

```text
프로그램에서 사용할 경로 정의
        ↓
Day06 OCR Rule 읽기
        ↓
Day06 Label CSV 읽기
        ↓
검사 가능한 Sample 목록 확인
        ↓
선택한 파일 이름 확인
        ↓
파일에 대응하는 ROI 확인
        ↓
제품 이미지 읽기
        ↓
SN ROI Crop
        ↓
Frozen Rule의 preprocess_mode 적용
        ↓
4배 확대
        ↓
Tesseract OCR
        ↓
OCR Text 정리
        ↓
Regex 형식 확인
        ↓
Expected Value 비교
        ↓
실패 Reason 생성
        ↓
Final OK / NG
        ↓
Service Version
Rule Version
Measurements
Reason을 포함한 Dictionary 생성
        ↓
필요하면 History에 저장
```

![[Pasted image 20260915180849.png]]

---

# 19. `actual_text`, `expected_text`, `ocr_text`를 구분합니다

6일차에서는 OCR이 제대로 읽었는지 확인하기 위해
이미지에 실제로 인쇄된 문자열을 정답으로 미리 기록했습니다.

```text
actual_text
→ 실제 이미지에 인쇄되어 있는 정답
→ 교육용 Ground Truth

expected_text
→ 원래 인쇄되어 있어야 하는 값

ocr_text
→ Tesseract가 실제로 읽은 값
```

6일차에서는:

```text
ocr_text
        VS
actual_text
        ↓
OCR이 제대로 읽었는지 평가
```

할 수 있었습니다.

하지만 실제 운영에서는 제품의 `actual_text`를 검사 전에 미리 알 수 없습니다.

따라서 서비스에서는:

```text
ocr_text
        ↓
Regex 형식 검사
        ↓
expected_text와 비교
        ↓
Final OK / NG
```

로 판단합니다.

> `actual_text`는 교육용 평가 정답이고, 실제 서비스 판정에는 사용하지 않습니다.

---

# 20. Result에 Version도 함께 남깁니다

검사 결과가:

```text
NG
```

라고만 저장되어 있으면 나중에 어떤 기준으로 나온 결과인지 알기 어렵습니다.

그래서 결과와 함께 다음 정보를 남깁니다.

```text
Service Version
→ 어떤 서비스 프로그램이었는가?

Rule Version
→ 어떤 OCR 검사 기준이었는가?

File
→ 어떤 이미지를 검사했는가?

Measurements
→ 무엇을 읽고 비교했는가?

Reason
→ 왜 NG가 되었는가?
```

예:

```json
{
  "service_version": "day10-service-v1.0",
  "rule_version": "day06-ocr-v1.0",
  "file_name": "0679256889404_side.png",
  "final_decision": "NG",
  "reasons": [
    "NG_EXPECTED_MISMATCH"
  ]
}
```

즉:

```text
Result
+
Version
+
Reason
        ↓
나중에 검사 결과를 다시 추적 가능
```

합니다.

> 이것이 9일차에서 배운 Traceability와 연결됩니다.

---

# 21. History는 최근 검사 결과를 보기 위해 남깁니다

실제 제조에서는 제품을 계속 검사합니다.

```text
제품 1
제품 2
제품 3
...
```

따라서 최근 결과를 저장해 두면 Web Dashboard에서:

```text
Total 검사 수
OK 수
NG 수
최근 NG Reason
```

을 확인할 수 있습니다.

오늘은 Database 대신 간단하게:

```text
reports/day10/service_history.json
```

에 최근 결과를 저장합니다.

실제 시스템에서는 나중에:

```text
SQLite
PostgreSQL
MES Database
```

등으로 확장할 수 있습니다.

---

# 22. `service_core.py`에서 무엇을 바꾸고 무엇을 그대로 둘까요?

10일차에서 가장 중요한 것은
6일차 검사 기준을 다시 튜닝하지 않는 것입니다.

```text
그대로 유지

Day06 Frozen Rule
ROI
Preprocess
PSM
Regex
Expected Value 검사 기준
```

10일차에서 새로 추가하는 것은:

```text
검사 결과를 함수로 반환

Service Version 기록

Reason 기록

History 저장

FastAPI에서 호출할 수 있는 구조
```

입니다.

즉:

```text
6일차
"어떻게 검사할 것인가?"

        ↓

10일차
"이미 만든 검사를
어떻게 서비스에서 재사용할 것인가?"
```

로 역할이 달라집니다.

> 10일차에서는 검사 성능을 다시 튜닝하는 것이 아니라, 검증된 검사 기능을 서비스 구조로 연결하는 데 집중합니다.
---

# PART4 — Service Core를 API 없이 먼저 Test합니다 (23~27)

---

# 23. API보다 먼저 Core를 직접 Test하는 이유

FastAPI까지 한꺼번에 연결한 뒤 오류가 나면 원인을 찾기 어렵습니다.

```text
OCR 문제?
API 문제?
Web 문제?
Network 문제?
```

를 구분하기 어렵기 때문입니다.

따라서 먼저:

```text
Python 함수만 실행
        ↓
검사 Result가 정상?
        ↓
YES
        ↓
그 다음 FastAPI 연결
```

순서로 진행합니다.

---

# 24. `81_day10_test_service_core.py`를 만듭니다

파일:

```text
src/81_day10_test_service_core.py
```

작성합니다.

```python
from pathlib import Path
import json
import sys


PROJECT_ROOT = (
    Path(__file__)
    .resolve()
    .parent
    .parent
)

sys.path.insert(
    0,
    str(PROJECT_ROOT),
)


from day10_service.service_core import (
    inspect_sample,
    list_samples,
)


samples = list_samples()

if not samples:
    raise RuntimeError(
        "검사할 Sample이 없습니다."
    )


print("=" * 70)
print("DAY10 SERVICE CORE TEST")
print("=" * 70)


for file_name in samples:

    result = inspect_sample(
        file_name
    )

    print()
    print("File   :", file_name)
    print(
        "OCR    :",
        result[
            "measurements"
        ][
            "ocr_text"
        ],
    )
    print(
        "Expect :",
        result[
            "measurements"
        ][
            "expected_text"
        ],
    )
    print(
        "Final  :",
        result[
            "final_decision"
        ],
    )
    print(
        "Reason :",
        result[
            "reasons"
        ],
    )


first_result = inspect_sample(
    samples[0]
)

output_path = (
    PROJECT_ROOT
    / "reports"
    / "day10"
    / "service_smoke_test.json"
)

output_path.parent.mkdir(
    parents=True,
    exist_ok=True,
)

output_path.write_text(
    json.dumps(
        first_result,
        indent=2,
        ensure_ascii=False,
    ),
    encoding="utf-8",
)

print()
print(
    "Saved:",
    output_path,
)
```

실행합니다.

```bash
python src/81_day10_test_service_core.py
```

### 코드흐름

```
필요한 Module 불러오기
Path / json / sys
        ↓
현재 파일 위치를 기준으로
프로젝트 최상위 경로 찾기
        ↓
PROJECT_ROOT를
Python import 경로에 추가
        ↓
service_core.py에서

inspect_sample()
list_samples()

함수 불러오기
        ↓
검사 가능한 Sample 이미지 목록 가져오기
        ↓
Sample이 없으면
오류 발생 후 종료
        ↓
Sample 이미지를 하나씩 반복
        ↓
inspect_sample(file_name) 호출
        ↓
service_core.py에서
실제 OCR 검사 실행
        ↓
검사 결과 Dictionary 반환
        ↓
각 이미지의

File
OCR
Expected
Final
Reason

Terminal에 출력
        ↓
모든 Sample 검사 완료
        ↓
첫 번째 Sample을 다시 검사
        ↓
첫 번째 검사 결과를
first_result에 저장
        ↓
저장할 경로 설정

reports/day10/
service_smoke_test.json
        ↓
폴더가 없으면 생성
        ↓
first_result Dictionary를
JSON 문자열로 변환
        ↓
service_smoke_test.json 저장
        ↓
저장된 파일 경로 출력
```

### 의사코드

```
1. 필요한 Python Module을 불러온다.

2. 현재 실행 중인 파일의 위치를 기준으로
   프로젝트 최상위 폴더인 PROJECT_ROOT를 찾는다.

3. PROJECT_ROOT를 Python이 Module을 찾는 경로에 추가한다.

4. day10_service/service_core.py에서

   inspect_sample()
   list_samples()

   두 함수를 불러온다.

5. list_samples()를 실행하여
   검사 가능한 Sample 이미지 목록을 가져온다.

6. Sample 이미지가 하나도 없다면
   오류를 발생시키고 프로그램을 종료한다.

7. Sample 이미지를 하나씩 반복한다.

8. 현재 이미지의 파일명을
   inspect_sample(file_name)에 전달한다.

9. service_core.py에서 실제 OCR 검사를 수행한다.

10. 검사 결과를 Dictionary 형태로 돌려받는다.

11. Dictionary 안에서 다음 값을 꺼낸다.

    File
    OCR Text
    Expected Text
    Final Decision
    Reason

12. 꺼낸 값을 Terminal에 출력한다.

13. 모든 Sample 이미지에 대해
    7~12번 과정을 반복한다.

14. 모든 Sample 검사가 끝나면
    첫 번째 Sample을 다시 inspect_sample()로 검사한다.

15. 첫 번째 검사 결과를
    first_result에 저장한다.

16. 결과를 저장할 경로를 만든다.

    reports/day10/service_smoke_test.json

17. reports/day10 폴더가 없다면
    자동으로 생성한다.

18. first_result Dictionary를
    JSON 문자열 형태로 변환한다.

19. 변환한 JSON 데이터를
    service_smoke_test.json 파일에 저장한다.

20. 마지막으로
    저장된 파일 경로를 Terminal에 출력한다.
```

---

# 25. Core Test 결과를 확인합니다

실행 결과 6개 제품이 다음과 같이 판정되었습니다.

```text
총 6개 검사

OK
→ 4개

NG
→ 2개
```

NG가 나온 두 제품을 확인해 봅니다.

```text
0679256889404_side.png

OCR
→ 0679256889404

Expected
→ 0679256889400

Final
→ NG

Reason
→ NG_EXPECTED_MISMATCH
```

그리고:

```text
0716954573504_side.png

OCR
→ 0716954573504

Expected
→ 0716954573500

Final
→ NG

Reason
→ NG_EXPECTED_MISMATCH
```

여기서 중요한 것은 **OCR을 잘못 읽어서 NG가 된 것이 아니라는 점**입니다.

```text
OCR이 읽은 값
        ↓
13자리 문자열 확인
        ↓
Expected Value와 비교
        ↓
값이 다름
        ↓
NG_EXPECTED_MISMATCH
```

반대로 정상 제품은:

```text
OCR
=
Expected

        ↓

Final = OK

Reason = []
```

가 되었습니다.

`Reason = []`은:

> **실패한 검사 조건이 없다는 뜻입니다.**

따라서 이번 결과로 다음을 확인했습니다.

```text
Day06 Frozen OCR Rule
        ↓
service_core.py에서 정상 재사용
        ↓
OCR
        ↓
Expected Value 비교
        ↓
OK / NG
        ↓
Reason 생성
```

> 10일차에서는 이 검사 기준을 다시 조정하지 않습니다.  
> 이미 동작하는 검사 기능을 이제 FastAPI 서비스에 연결합니다.

---

# 26. `service_smoke_test.json`도 확인합니다

실행 마지막에 다음 파일이 만들어졌습니다.

```text
reports/day10/service_smoke_test.json
```

현재 작성한 Test 코드에서는 **첫 번째 Sample 1개의 검사 결과를 JSON으로 저장**합니다.

즉 이 파일의 목적은:

```text
"service_core.py가
검사 결과를 원하는 구조로
정상적으로 만들 수 있는가?"
```

를 간단히 확인하는 것입니다.

JSON에는 다음과 같은 정보가 들어갑니다.

```text
어떤 이미지를 검사했는가?

어떤 Rule Version을 사용했는가?

OCR이 무엇을 읽었는가?

Expected Value는 무엇인가?

최종 결과는 OK / NG인가?

실패 이유가 있는가?
```

즉 지금까지:

```text
제품 이미지
        ↓
service_core.py
        ↓
OCR 검사
        ↓
Result Dictionary
        ↓
JSON 저장
```

까지 정상적으로 연결되었습니다.

---

# 27. 아직 다른 Device가 사용할 수 있는 서비스는 아닙니다

지금까지 성공한 것은:

```text
Python 프로그램
        ↓
service_core.py
        ↓
제품 검사
        ↓
OK / NG + Reason
```

입니다.

즉 **검사 기능 자체는 정상적으로 동작하지만**, 아직 다른 PC·Tablet·Mobile이 이 기능을 요청할 수는 없습니다.

다음 단계에서는 FastAPI를 연결합니다.

```text
PC / Tablet / Mobile
        ↓
검사 요청
        ↓
HTTP
        ↓
FastAPI
        ↓
service_core.py
        ↓
실제 OCR 검사
        ↓
JSON Response
        ↓
Client 화면에 결과 표시
```

지금까지는:

```text
"검사 기능이 제대로 동작하는가?"
```

를 확인했다면,

다음부터는:

```text
"이 검사 기능을
다른 프로그램과 Device에서도
사용할 수 있게 하자."
```

로 넘어갑니다.

---

# PART5 — FastAPI Server를 만듭니다 (28~36)

---

# 28. `api_main.py`를 만듭니다

파일:

```text
day10_service/api_main.py
```

작성합니다.

```python
from pathlib import Path
import sys

from fastapi import FastAPI, HTTPException
from fastapi.responses import FileResponse
from fastapi.staticfiles import StaticFiles
from pydantic import BaseModel


PROJECT_ROOT = (
    Path(__file__)
    .resolve()
    .parent
    .parent
)

sys.path.insert(
    0,
    str(PROJECT_ROOT),
)


from day10_service.service_core import (
    PRODUCT_DIR,
    append_history,
    inspect_sample,
    latest_result,
    list_samples,
    read_history,
    stats,
)


STATIC_DIR = (
    PROJECT_ROOT
    / "day10_service"
    / "static"
)


app = FastAPI(
    title="Subject9 Machine Vision Service",
    version="1.0.0",
    description=(
        "Day10 FastAPI + Web Dashboard Practice"
    ),
)


class InspectRequest(BaseModel):
    file_name: str


app.mount(
    "/static",
    StaticFiles(
        directory=STATIC_DIR
    ),
    name="static",
)


@app.get("/")
def home():
    return FileResponse(
        STATIC_DIR
        / "index.html"
    )


@app.get("/api/health")
def health():
    return {
        "status": "ok",
        "service":
            "Subject9 Machine Vision Service",
    }


@app.get("/api/samples")
def samples():
    return {
        "samples":
            list_samples()
    }


@app.get(
    "/api/sample-image/{file_name}"
)
def sample_image(
    file_name: str,
):
    if file_name not in list_samples():
        raise HTTPException(
            status_code=404,
            detail="등록되지 않은 Sample입니다.",
        )

    return FileResponse(
        PRODUCT_DIR
        / file_name
    )


@app.post("/api/inspect")
def inspect(
    request: InspectRequest,
):
    try:
        result = inspect_sample(
            request.file_name
        )

        append_history(
            result
        )

        return result

    except FileNotFoundError as error:
        raise HTTPException(
            status_code=404,
            detail=str(error),
        )

    except (
        KeyError,
        ValueError,
        RuntimeError,
    ) as error:
        raise HTTPException(
            status_code=400,
            detail=str(error),
        )


@app.get("/api/latest")
def latest():
    result = latest_result()

    return {
        "latest": result
    }


@app.get("/api/history")
def history(
    limit: int = 20,
):
    rows = read_history()

    limit = max(
        1,
        min(
            int(limit),
            100,
        ),
    )

    return {
        "history":
            rows[-limit:]
    }


@app.get("/api/stats")
def inspection_stats():
    return stats()
```

---

# 29. API 코드의 처리 흐름을 읽어봅니다

```text
FastAPI App 생성
        ↓
Web Static 폴더 연결
        ↓
/ 접속
→ index.html 반환

        ↓

/api/health
→ Server 상태 반환

/api/samples
→ 검사 가능한 이미지 목록 반환

/api/sample-image/{file_name}
→ 선택한 제품 이미지 반환

/api/inspect
→ 검사 요청 받음
→ service_core.inspect_sample()
→ 검사 결과 생성
→ History 저장
→ JSON 반환

/api/latest
→ 가장 최근 검사 결과

/api/history
→ 최근 검사 이력

/api/stats
→ OK / NG 개수
```

---

# 30. FastAPI와 Service Core를 분리한 이유를 다시 확인합니다

좋지 않은 구조:

```text
FastAPI Route 안에

OpenCV
OCR
Regex
Rule
History
전부 작성
```

이렇게 하면 코드가 커지고 재사용하기 어렵습니다.

오늘 구조:

```text
service_core.py
→ 검사 로직

api_main.py
→ HTTP 연결
```

로 역할을 나누었습니다.

이 구조는 교과14에서도 중요합니다.

---

# 31. FastAPI Server를 실행합니다

프로젝트 최상위에서 실행합니다.

```bash
uvicorn day10_service.api_main:app \
  --host 127.0.0.1 \
  --port 8000 \
  --reload
```

정상이라면 다음과 비슷한 메시지가 보입니다.

```text
Uvicorn running on http://127.0.0.1:8000
```

이 터미널은 Server가 실행 중이므로 그대로 둡니다.

새 터미널이 필요하면 VS Code에서 Terminal을 하나 더 엽니다.

---

# 32. Browser에서 Health를 확인합니다

Browser에서 다음 주소를 엽니다.

```text
http://127.0.0.1:8000/api/health
```

다음 구조가 보이면 됩니다.

```json
{
  "status": "ok",
  "service": "Subject9 Machine Vision Service"
}
```

이 결과는:

```text
FastAPI Server가 실행 중이며
HTTP Request를 받을 수 있음
```

을 의미합니다.

---

# 33. Swagger Docs를 엽니다

FastAPI는 자동 API 문서를 제공합니다.

Browser:

```text
http://127.0.0.1:8000/docs
```

여기에서 다음 Endpoint를 확인합니다.

```text
GET  /api/health

GET  /api/samples

GET  /api/sample-image/{file_name}

POST /api/inspect

GET  /api/latest

GET  /api/history

GET  /api/stats
```

---

# 34. `POST /api/inspect`는 왜 중요한가요?

이 Endpoint가 실제 검사 기능과 연결됩니다.

```text
Client
        ↓
POST /api/inspect
        ↓
file_name 전달
        ↓
FastAPI
        ↓
service_core.inspect_sample()
        ↓
OCR 검사
        ↓
Final OK / NG
        ↓
JSON Response
```

즉 API가 단순히 저장된 JSON만 보여주는 것이 아니라 실제 검사 함수를 실행합니다.

---

# 35. Swagger에서 검사해 봅니다

# 35. Swagger에서 실제 제품 이미지를 검사해 봅니다

`POST /api/inspect`를 펼치고  
`Try it out`을 누릅니다.

이번에 검사할 파일은 6일차에서 사용했던 제품 이미지입니다.

```text
samples/day06/product/
└── 0637145699424_side.png
```

Request Body에 파일 이름을 전달합니다.

```json
{
  "file_name": "0637145699424_side.png"
}
```

여기서는 이미지를 새로 업로드하는 것이 아니라  
**Server에 이미 저장되어 있는 Sample 이미지의 파일 이름을 전달**합니다.

`Execute`를 누르면 FastAPI가:

```text
0637145699424_side.png
        ↓
service_core.py
        ↓
OCR 검사
        ↓
JSON 결과 반환
```

순서로 처리합니다.

실행 결과에서:

![[{C3B4976E-6F8D-473E-86D6-545310E1505C}.png]]

```text
Code : 200
```

이 나오면 **API 요청이 정상적으로 처리되었다**는 뜻입니다.

그리고 Response Body에서:

```text
ocr_text
→ 0637145699424

expected_text
→ 0637145699424

final_decision
→ OK

reasons
→ []
```

을 확인할 수 있습니다.

즉 이번 테스트에서는:

```text
OCR이 읽은 값
=
Expected Value
        ↓
Final OK
```

로 정상 판정되었습니다.

> `200 OK`는 HTTP 요청 성공을 의미하고, `final_decision: "OK"`는 실제 머신비전 검사 결과가 정상이라는 의미입니다.

---

# 36. 실제 API Response를 읽어봅니다

이번에는 앞에서 실제 검사했을 때 `NG`가 나온 제품을 기준으로 결과를 읽어봅니다.

검사 파일:

```text
samples/day06/product/
└── 0679256889404_side.png
```

이 파일의 실제 검사 결과는 다음과 같습니다.

```json
{
  "file_name": "0679256889404_side.png",
  "measurements": {
    "ocr_text": "0679256889404",
    "expected_text": "0679256889400",
    "pattern_ok": true,
    "expected_match": false
  },
  "final_decision": "NG",
  "reasons": [
    "NG_EXPECTED_MISMATCH"
  ]
}
```

결과를 위에서부터 읽어보면:

```text
검사 파일
0679256889404_side.png
        ↓
OCR이 읽은 값
0679256889404
        ↓
13자리 숫자 형식 확인
pattern_ok = true
        ↓
Expected Value
0679256889400
        ↓
OCR 값과 Expected Value가 다름
expected_match = false
        ↓
Final Decision = NG
        ↓
Reason
NG_EXPECTED_MISMATCH
```

즉 이 제품은 **OCR을 잘못 읽어서 NG가 된 것이 아닙니다.**

```text
OCR은 정상적으로 읽음
        ↓
Regex 형식도 정상
        ↓
하지만 Expected Value와 불일치
        ↓
NG
```

> `NG_EXPECTED_MISMATCH`는 **OCR이 읽은 값과 제품에 기대한 값이 서로 다르기 때문에 NG가 되었다**는 의미입니다.

---

# PART6 — API를 코드에서도 Test합니다 (37~40)

---

# 37. 이번에는 Python 프로그램에서 API를 호출합니다

앞에서는 Swagger Browser 화면에서 API가 정상적으로 동작하는지 확인했습니다. 

이번에는 다른 Python 프로그램이 FastAPI Server에 직접 요청을 보내도 같은 검사 결과를 받을 수 있는지 확인합니다.

```text
Python Client
        ↓
HTTP Request
        ↓
FastAPI Server
        ↓
service_core.py
        ↓
실제 OCR 검사
        ↓
JSON Response
        ↓
Python Client가 결과 확인
```

즉 이번 테스트의 핵심은: 

> 검사 코드를 직접 실행하지 않고도 API를 통해 검사를 요청할 수 있는가? 를 확인하는 것입니다.

---

# 38. `82_day10_test_api.py`를 만듭니다

FastAPI Server가 실행 중인 상태에서 새 터미널에서 작성·실행합니다.

파일:

```text
src/82_day10_test_api.py
```

작성합니다.

```python
from pathlib import Path
import json

import httpx


BASE_URL = (
    "http://127.0.0.1:8000"
)


with httpx.Client(
    timeout=30.0
) as client:

    health = client.get(
        f"{BASE_URL}/api/health"
    )

    health.raise_for_status()


    sample_response = client.get(
        f"{BASE_URL}/api/samples"
    )

    sample_response.raise_for_status()

    samples = sample_response.json()[
        "samples"
    ]


    if not samples:
        raise RuntimeError(
            "검사 가능한 Sample이 없습니다."
        )


    inspect_response = client.post(
        f"{BASE_URL}/api/inspect",
        json={
            "file_name":
                samples[0]
        },
    )

    inspect_response.raise_for_status()

    result = (
        inspect_response
        .json()
    )


print("=" * 70)
print("DAY10 API TEST")
print("=" * 70)

print(
    "Health:",
    health.json()
)

print(
    "Sample:",
    samples[0]
)

print(
    "Final:",
    result[
        "final_decision"
    ]
)

print(
    "Reason:",
    result[
        "reasons"
    ]
)


output_path = Path(
    "reports/day10/api_test_result.json"
)

output_path.parent.mkdir(
    parents=True,
    exist_ok=True,
)

output_path.write_text(
    json.dumps(
        result,
        indent=2,
        ensure_ascii=False,
    ),
    encoding="utf-8",
)

print(
    "Saved:",
    output_path
)
```

실행합니다.

```bash
python src/82_day10_test_api.py
```

이번 코드에서는 다음 순서로 API를 호출합니다.

```text
GET /api/health
        ↓
Server가 정상인지 확인
        ↓
GET /api/samples
        ↓
검사 가능한 Sample 목록 받기
        ↓
첫 번째 Sample 선택
        ↓
POST /api/inspect
        ↓
실제 OCR 검사 요청
        ↓
JSON 검사 결과 받기
```

---

# 39. 실제 실행 결과를 확인합니다

실제 결과는 다음과 같이 나왔습니다.

```text
DAY10 API TEST

Health:
{'status': 'ok',
 'service': 'Subject9 Machine Vision Service'}

Sample:
0637145699424_side.png

Final:
OK

Reason:
[]
```

하나씩 해석하면:

```text
Health: status = ok
→ FastAPI Server가 정상적으로 동작 중

        ↓

Sample
0637145699424_side.png
→ 검사할 첫 번째 제품 이미지를 정상적으로 받음

        ↓

POST /api/inspect
→ 해당 이미지 검사 요청

        ↓

Final = OK
→ 실제 OCR 검사 결과 정상

        ↓

Reason = []
→ 실패한 검사 조건이 없음
```

즉 이번 테스트에서는:

```text
Python Client
        ↓
FastAPI에 검사 요청
        ↓
0637145699424_side.png 검사
        ↓
OCR / Rule 검사 정상
        ↓
Final OK
        ↓
Python Client가 결과 수신
```

까지 정상적으로 연결되었습니다.

---

# 40. 여기서 가장 중요한 점

이번 Python 프로그램은 `service_core.py`의 검사 함수를 직접 호출한 것이 아닙니다.

```text
직접 호출

inspect_sample()
X
```

대신:

```text
URL
+
HTTP Request
+
JSON
```

을 이용하여 FastAPI Server에 검사를 요청했습니다.

즉:

```text
Client
→ 검사 내부 코드까지 알 필요 없음

Server
→ 실제 검사 실행

API
→ 요청과 결과를 연결
```

하는 구조가 만들어진 것입니다.

> 이제 검사 기능은 Python 내부에서만 사용하는 함수가 아니라, 다른 프로그램도 API를 통해 사용할 수 있는 서비스가 되었습니다.

---

# PART7 — Web Dashboard를 만듭니다 (41~49)

---

# 41. 이제 사람이 보기 쉬운 화면을 만듭니다

Swagger는 개발자에게 매우 편리합니다.

하지만 실제 작업자가 Swagger 화면을 사용할 가능성은 낮습니다.

작업자가 보고 싶은 것은 더 단순합니다.

```text
검사 이미지
        ↓
[ 검사하기 ]
        ↓
OK 또는 NG
        ↓
NG이면 Reason
```

그래서 간단한 Web Dashboard를 만듭니다.

---

# 42. Web 화면에서 필요한 것만 정합니다

오늘 화면은 다음 정도면 충분합니다.

```text
제품 이미지 선택

선택 이미지 Preview

검사하기 Button

Final Result
OK / NG

OCR Text

Expected Text

Reason

Service Version

Rule Version

오늘 누적 통계
Total / OK / NG

최근 검사 History
```

React나 Vue를 새로 배우지 않습니다.

오늘의 목적은 Front-End Framework가 아닙니다.

```text
HTML
CSS
JavaScript fetch()
```

만 사용합니다.

---

# 43. `index.html`을 만듭니다

파일:

```text
day10_service/static/index.html
```

작성합니다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="교과 9 · 10일차: 제조 문자 OCR 검사 웹 대시보드">
  <title>제조 문자 OCR 검사 | Machine Vision Dashboard</title>
  <link rel="stylesheet" href="/static/style.css">
  <script defer src="/static/app.js"></script>
</head>
<body>
  <div class="app-shell">
    <header class="page-header">
      <div class="brand-line"><span class="brand-icon" aria-hidden="true">▣</span><span>VISION AI · 교과 9 / 10일차</span></div>
      <div class="heading-row">
        <div>
          <h1>제조 문자 OCR 검사 대시보드</h1>
          <p class="subtitle">제품 이미지에서 일련번호(SN)를 읽고, 13자리 형식과 기준 번호를 검사합니다.</p>
        </div>
        <div id="serverStatus" class="server-status checking" role="status" aria-live="polite">● 서버 확인 중</div>
      </div>
    </header>

    <section class="how-it-works" aria-label="검사 처리 과정">
      <span class="how-label">검사 과정</span>
      <ol class="flow-list">
        <li><span class="flow-num">1</span><span>제품 이미지 선택</span></li>
        <li><span class="flow-num">2</span><span>SN 문자 OCR 인식</span></li>
        <li><span class="flow-num">3</span><span>형식·기준 번호 비교</span></li>
        <li><span class="flow-num">4</span><span>OK / NG 판정·기록</span></li>
      </ol>
    </section>

    <div id="notice" class="notice" role="alert" hidden></div>

    <main>
      <div class="workspace">
        <section class="card input-card" aria-labelledby="inputHeading">
          <div class="card-title-row">
            <div>
              <div class="eyebrow">STEP 01 · 검사 대상</div>
              <h2 id="inputHeading">검사할 제품 이미지 선택</h2>
            </div>
            <span class="tag neutral">기존 샘플 검사</span>
          </div>
          <p class="helper">6일차에서 준비한 제품 이미지를 선택합니다. 이미지는 서버에 저장되어 있으며 새로운 파일을 업로드하지 않습니다.</p>
          <label for="sampleSelect" class="field-label">검사 이미지 파일</label>
          <div class="controls">
            <select id="sampleSelect" aria-describedby="fileHelp"><option value="">제품 목록을 불러오는 중…</option></select>
            <button id="inspectButton" type="button" class="primary-button" disabled>선택 이미지 검사</button>
          </div>
          <p id="fileHelp" class="field-help">이미지를 선택한 뒤 검사 버튼을 누르세요.</p>
          <figure class="preview-area">
            <div class="preview-box">
              <div id="previewPlaceholder" class="preview-placeholder">이미지를 선택하면 이곳에 표시됩니다.</div>
              <img id="previewImage" alt="선택한 검사 제품 이미지" hidden>
            </div>
            <figcaption id="selectedFileName">선택된 이미지 없음</figcaption>
          </figure>
          <div class="info-note"><strong>검사 대상</strong><span>인쇄된 SN(일련번호) · Tesseract OCR · 6일차 확정 검사 규칙(Frozen Rule)</span></div>
        </section>

        <section class="card result-card" aria-labelledby="resultHeading">
          <div class="card-title-row">
            <div>
              <div class="eyebrow">STEP 02 · 검사 결과</div>
              <h2 id="resultHeading">OCR 검사 판정</h2>
            </div>
            <span class="tag accent">문자·코드 검사</span>
          </div>
          <div id="finalResult" class="decision waiting" role="status" aria-live="polite">
            <span class="decision-symbol" id="decisionSymbol" aria-hidden="true">…</span>
            <div><strong id="decisionTitle">검사 대기</strong><p id="decisionDescription">검사할 제품을 선택하고 버튼을 누르세요.</p></div>
          </div>

          <div class="comparison">
            <div class="comparison-item">
              <span>실제 OCR 인식값</span>
              <strong id="ocrText" class="mono">-</strong>
              <small>선택한 이미지의 SN 영역에서 읽은 값</small>
            </div>
            <div class="compare-sign" aria-hidden="true">↔</div>
            <div class="comparison-item">
              <span>기준 번호 (Expected)</span>
              <strong id="expectedText" class="mono">-</strong>
              <small>해당 제품에 인쇄되어 있어야 하는 값</small>
            </div>
          </div>

          <div class="checks">
            <div class="check-line"><span>① 13자리 숫자 형식 검사</span><strong id="patternStatus" class="check-status pending">검사 전</strong></div>
            <div class="check-line"><span>② OCR 값과 기준 번호 비교</span><strong id="matchStatus" class="check-status pending">검사 전</strong></div>
          </div>
          <div class="reason-box"><span class="section-caption">최종 판정 이유</span><p id="reasonText">검사 완료 후 판정 이유가 표시됩니다.</p><small id="reasonCode" class="reason-code"></small></div>
          <details class="metadata">
            <summary>검사 추적 정보 보기 (버전 · 요청 번호)</summary>
            <dl>
              <div><dt>OCR 검사 규칙 버전</dt><dd id="ruleVersion">-</dd></div>
              <div><dt>서비스 버전</dt><dd id="serviceVersion">-</dd></div>
              <div><dt>요청 ID</dt><dd id="requestId">-</dd></div>
              <div><dt>검사 시간</dt><dd id="inspectedAt">-</dd></div>
            </dl>
          </details>
        </section>
      </div>

      <section class="card stats-card" aria-labelledby="statsHeading">
        <div class="card-title-row"><div><div class="eyebrow">STEP 03 · 검사 기록</div><h2 id="statsHeading">저장된 검사 통계</h2></div><span class="tag neutral">최근 저장된 기록 기준</span></div>
        <div class="stats">
          <div class="stat total"><span>누적 검사 횟수</span><strong id="totalCount">0</strong><small>검사 버튼을 실행한 횟수</small></div>
          <div class="stat ok"><span>정상 (OK)</span><strong id="okCount">0</strong><small>두 검사 조건 모두 통과</small></div>
          <div class="stat ng"><span>불량 (NG)</span><strong id="ngCount">0</strong><small>하나 이상 조건 불일치</small></div>
        </div>
        <div class="ratio-row"><span>저장 기록 중 OK 비율</span><strong id="okRate">-</strong></div>
        <div class="ratio-track" aria-hidden="true"><div id="okRateFill"></div></div>
        <p class="metric-note">※ 이 비율은 저장된 검사 결과 중 OK의 비율이며, OCR 인식 정확도나 모델 성능지표가 아닙니다. 통계는 날짜 구분 없이 서버에 보관된 최근 검사 기록을 기준으로 합니다.</p>
      </section>

      <section class="card history-card" aria-labelledby="historyHeading">
        <div class="card-title-row"><div><div class="eyebrow">STEP 04 · 이력 추적</div><h2 id="historyHeading">최근 검사 이력</h2></div><span class="tag neutral">최근 10건</span></div>
        <p class="helper">언제, 어떤 이미지를 검사했고, 왜 OK 또는 NG로 판정되었는지 확인할 수 있습니다.</p>
        <div class="table-wrap">
          <table>
            <thead><tr><th scope="col">검사 시각</th><th scope="col">검사 파일</th><th scope="col">판정</th><th scope="col">판정 이유</th></tr></thead>
            <tbody id="historyBody"><tr><td colspan="4" class="empty-table">검사 이력을 불러오는 중입니다.</td></tr></tbody>
          </table>
        </div>
      </section>
    </main>
    <footer class="footer">교과 9 · 10일차 | Day06 Frozen OCR Rule → FastAPI → Web Dashboard <a href="/docs" target="_blank" rel="noopener noreferrer">API 문서 (Swagger) ↗</a></footer>
  </div>
</body>
</html>
```

---

# 44. `style.css`를 만듭니다

파일:

```text
day10_service/static/style.css
```

작성합니다.

```css
:root {

  --bg: #f3f6fb;

  --surface: #fff;

  --ink: #18283f;

  --muted: #617189;

  --line: #e4eaf3;

  --brand: #235ed5;

  --brand-dark: #1749af;

  --good: #13794b;

  --good-bg: #e8f7ee;

  --bad: #bf3740;

  --bad-bg: #fff0f0;

  --radius: 18px;

}

* { box-sizing: border-box; }

html { -webkit-text-size-adjust: 100%; }

body { margin: 0; background: var(--bg); color: var(--ink); font-family: 'Pretendard', 'Noto Sans KR', 'Malgun Gothic', 'Apple SD Gothic Neo', Arial, sans-serif; line-height: 1.6; }

button, select { font: inherit; }

button:focus-visible, select:focus-visible, a:focus-visible, summary:focus-visible { outline: 3px solid #81b1ff; outline-offset: 3px; }

[hidden] { display: none !important; }

.app-shell { width: min(1240px, calc(100% - 40px)); margin: 32px auto 36px; }

.page-header { margin-bottom: 22px; }

.brand-line { display: flex; align-items: center; gap: 9px; font-weight: 800; color: var(--brand); font-size: 13px; letter-spacing: .08em; }

.brand-icon { width: 32px; height: 32px; display: inline-flex; align-items: center; justify-content: center; border-radius: 10px; background: #dfeaff; color: #1d54c6; font-size: 19px; }

.heading-row { display: flex; align-items: center; justify-content: space-between; gap: 18px; margin-top: 9px; }

h1 { margin: 0 0 4px; font-size: clamp(26px, 2.6vw, 36px); letter-spacing: -.045em; line-height: 1.35; }

.subtitle { margin: 0; color: var(--muted); font-size: 15px; }

.server-status { font-size: 13px; font-weight: 800; padding: 9px 14px; border-radius: 999px; white-space: nowrap; }

.server-status.checking { color: #5b6b82; background: #e5ebf3; }

.server-status.online { color: #0d774e; background: #ddf5e8; }

.server-status.offline { color: #ad3240; background: #ffe4e5; }

.how-it-works { display: flex; align-items: center; gap: 24px; padding: 17px 21px; border-radius: 16px; background: #eaf1ff; border: 1px solid #d8e4fc; margin-bottom: 22px; }

.how-label { font-weight: 800; font-size: 14px; color: #1549a4; white-space: nowrap; }

.flow-list { list-style: none; display: flex; justify-content: space-between; gap: 12px; flex: 1; padding: 0; margin: 0; }

.flow-list li { display: flex; align-items: center; gap: 8px; color: #31547e; font-weight: 650; font-size: 13px; position: relative; }

.flow-list li:not(:last-child)::after { content: '→'; position: absolute; right: -21px; font-size: 19px; color: #7ca0df; }

.flow-num { flex: none; width: 26px; height: 26px; display: inline-flex; justify-content: center; align-items: center; color: #fff; background: #447be4; border-radius: 50%; font-weight: 800; }

.notice { border-radius: 12px; border: 1px solid #f5a7ac; background: #fff4f5; color: #a12632; padding: 12px 16px; margin-bottom: 20px; font-size: 14px; }

.workspace { display: grid; grid-template-columns: minmax(0, .97fr) minmax(0, 1.03fr); gap: 20px; align-items: stretch; }

.card { background: var(--surface); border: 1px solid var(--line); box-shadow: 0 6px 22px rgba(21, 51, 97, .055); padding: 24px; border-radius: var(--radius); min-width: 0; }

.stats-card, .history-card { margin-top: 20px; }

.card-title-row { display: flex; align-items: flex-start; justify-content: space-between; gap: 12px; margin-bottom: 12px; }

.eyebrow { font-size: 12px; text-transform: uppercase; font-weight: 850; letter-spacing: .09em; color: var(--brand); margin-bottom: 4px; }

h2 { font-size: clamp(20px, 2vw, 23px); letter-spacing: -.035em; line-height: 1.35; margin: 0; }

.tag { flex-shrink: 0; font-size: 12px; font-weight: 800; border-radius: 999px; padding: 5px 10px; white-space: nowrap; }

.tag.neutral { color: #536580; background: #f0f3f8; }

.tag.accent { color: #1b55b3; background: #e7efff; }

.helper { color: var(--muted); font-size: 13px; line-height: 1.65; margin: 0 0 20px; }

.field-label { display: block; font-weight: 800; font-size: 14px; margin: 0 0 8px; }

.controls { display: grid; grid-template-columns: minmax(0, 1fr) 156px; gap: 10px; }

select { width: 100%; min-width: 0; border: 1px solid #cbd6e6; background: #fff; color: var(--ink); border-radius: 11px; padding: 13px 11px; min-height: 48px; font-size: 14px; }

.primary-button { min-height: 48px; border: none; background: var(--brand); color: #fff; padding: 12px; border-radius: 11px; font-weight: 800; cursor: pointer; transition: background .15s, transform .15s; }

.primary-button:hover:not(:disabled) { background: var(--brand-dark); }

.primary-button:active:not(:disabled) { transform: translateY(1px); }

.primary-button:disabled { opacity: .5; cursor: not-allowed; }

.field-help { color: var(--muted); font-size: 12px; margin: 10px 0 0; }

.preview-area { margin: 19px 0 15px; }

.preview-box { height: 300px; border: 1px dashed #c9d6e8; background: #f6f9fe; border-radius: 13px; overflow: hidden; display: flex; align-items: center; justify-content: center; padding: 11px; }

.preview-box img { display: block; max-width: 100%; max-height: 100%; object-fit: contain; border-radius: 7px; }

.preview-placeholder { font-size: 14px; color: #7a8aa2; text-align: center; }

.preview-area figcaption { word-break: break-all; font-size: 13px; color: #596e89; padding: 9px 4px 0; text-align: center; }

.info-note { display: flex; gap: 7px; flex-wrap: wrap; background: #f1f6ff; border: 1px solid #e1ebfb; border-radius: 11px; padding: 11px 12px; color: #47617d; font-size: 12px; }

.info-note strong { color: #2052b1; }

.decision { display: flex; align-items: center; gap: 17px; padding: 19px; margin: 19px 0 17px; border: 1px solid #dce3ed; border-radius: 13px; background: #f1f4f8; color: #4e6079; min-height: 110px; }

.decision.ok { background: var(--good-bg); border-color: #b6e5c8; color: var(--good); }

.decision.ng { background: var(--bad-bg); border-color: #f3b8bb; color: var(--bad); }

.decision-symbol { display: inline-flex; justify-content: center; align-items: center; flex: none; width: 58px; height: 58px; font-size: 31px; font-weight: 800; border: 2px solid currentColor; border-radius: 50%; }

.decision strong { display: block; font-size: 26px; line-height: 1.2; }

.decision p { color: inherit; opacity: .85; font-size: 13px; margin: 5px 0 0; }

.comparison { display: grid; grid-template-columns: minmax(0, 1fr) 24px minmax(0, 1fr); gap: 8px; align-items: center; }

.comparison-item { background: #f7f9fc; padding: 14px; border: 1px solid #e9eef5; border-radius: 12px; min-width: 0; }

.comparison-item > span { display: block; color: #52667e; font-size: 12px; font-weight: 800; }

.comparison-item strong { font-size: clamp(15px, 1.6vw, 20px); display: block; margin: 5px 0 4px; overflow-wrap: anywhere; }

.comparison-item small { display: block; font-size: 11px; color: #77879a; }

.mono { font-family: ui-monospace, SFMono-Regular, Consolas, monospace; font-variant-numeric: tabular-nums; }

.compare-sign { font-size: 20px; color: #879bb3; text-align: center; }

.checks { margin-top: 17px; border-top: 1px solid var(--line); }

.check-line { display: flex; align-items: center; justify-content: space-between; gap: 10px; padding: 13px 0; border-bottom: 1px solid var(--line); font-weight: 650; font-size: 14px; }

.check-status { flex: none; border-radius: 999px; font-size: 12px; padding: 4px 11px; }

.check-status.pending { color: #6e7b8d; background: #eef1f5; }

.check-status.pass { color: #08734a; background: #e1f6e8; }

.check-status.fail { color: #b6313b; background: #ffe6e8; }

.reason-box { background: #f7f9fc; border-radius: 12px; border: 1px solid #e8edf4; padding: 14px 16px; margin-top: 16px; }

.section-caption { color: #546982; font-size: 12px; font-weight: 800; }

.reason-box p { font-size: 14px; font-weight: 700; margin: 4px 0 0; }

.reason-code { display: block; color: #6f8299; margin-top: 5px; font-family: ui-monospace, Consolas, monospace; font-size: 11px; overflow-wrap: anywhere; }

.metadata { border-top: 1px solid var(--line); margin-top: 18px; padding-top: 13px; }

.metadata summary { cursor: pointer; color: #446aab; font-size: 13px; font-weight: 750; }

.metadata dl { margin: 13px 0 0; }

.metadata dl div { display: flex; justify-content: space-between; gap: 10px; padding: 7px 0; border-bottom: 1px solid #edf0f6; font-size: 12px; }

.metadata dt { color: #61758b; }

.metadata dd { margin: 0; text-align: right; overflow-wrap: anywhere; font-family: ui-monospace, Consolas, monospace; }

.stats { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 14px; }

.stat { border: 1px solid #e1e8f2; border-radius: 13px; padding: 16px 20px; background: #f7f9fc; }

.stat.ok { background: #edfaf3; border-color: #c7e9d5; }

.stat.ng { background: #fff2f2; border-color: #f4d2d4; }

.stat span { display: block; color: #50627a; font-size: 13px; font-weight: 800; }

.stat strong { font-size: 32px; display: block; line-height: 1.45; font-variant-numeric: tabular-nums; }

.stat.ok strong { color: var(--good); }

.stat.ng strong { color: var(--bad); }

.stat small { color: #63758c; font-size: 12px; }

.ratio-row { display: flex; justify-content: space-between; align-items: center; margin-top: 18px; font-size: 13px; color: #556980; }

.ratio-row strong { color: #1f5db6; }

.ratio-track { height: 9px; border-radius: 10px; background: #e9edf4; margin-top: 8px; overflow: hidden; }

#okRateFill { height: 100%; width: 0%; background: #20a06e; transition: width .25s ease; }

.metric-note { font-size: 12px; line-height: 1.65; color: #74849a; margin: 10px 0 0; }

.table-wrap { overflow-x: auto; border: 1px solid var(--line); border-radius: 12px; }

table { width: 100%; border-collapse: collapse; min-width: 680px; font-size: 13px; }

th { background: #f3f6fb; font-weight: 850; color: #52667d; text-align: left; white-space: nowrap; }

th, td { padding: 12px 15px; border-bottom: 1px solid #edf0f5; vertical-align: middle; }

tbody tr:last-child td { border-bottom: none; }

tbody tr:hover { background: #f9fbfe; }

td:nth-child(2) { overflow-wrap: anywhere; max-width: 300px; }

.empty-table { color: #8190a3; text-align: center; padding: 25px; }

.result-pill { display: inline-block; border-radius: 999px; padding: 4px 11px; font-weight: 850; font-size: 12px; }

.result-pill.ok { color: var(--good); background: var(--good-bg); }

.result-pill.ng { color: var(--bad); background: var(--bad-bg); }

.footer { padding: 25px 5px 5px; display: flex; gap: 14px; flex-wrap: wrap; justify-content: space-between; font-size: 12px; color: #7e8da1; }

.footer a { color: #245fcb; font-weight: 750; text-decoration: none; }

.footer a:hover { text-decoration: underline; }

@media (max-width: 1040px) {

  .workspace { grid-template-columns: 1fr; }

  .preview-box { height: 340px; }

  .flow-list { flex-wrap: wrap; justify-content: flex-start; column-gap: 25px; }

  .flow-list li:not(:last-child)::after { display: none; }

}

@media (max-width: 640px) {

  .app-shell { width: min(100% - 24px, 560px); margin-top: 20px; }

  .card { padding: 19px 15px; border-radius: 14px; }

  .heading-row { flex-direction: column; align-items: flex-start; }

  .how-it-works { display: block; padding: 14px; }

  .flow-list { margin-top: 10px; display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }

  .flow-list li { font-size: 12px; }

  .controls { grid-template-columns: 1fr; }

  .preview-box { height: 255px; }

  .card-title-row { flex-wrap: wrap; }

  .comparison { grid-template-columns: minmax(0, 1fr); }

  .compare-sign { transform: rotate(90deg); line-height: 1; }

  .stats { gap: 8px; }

  .stat { padding: 11px 9px; }

  .stat span { font-size: 11px; }

  .stat strong { font-size: 25px; }

  .stat small { font-size: 10px; }

  .decision { padding: 14px; }

  .decision-symbol { width: 48px; height: 48px; font-size: 25px; }

  .decision strong { font-size: 22px; }

}
```

---

# 45. `app.js`를 만듭니다

파일:

```text
day10_service/static/app.js
```

작성합니다.

```javascript
// Day10 UI 개선: 기존 FastAPI API 구조를 바꾸지 않습니다.

// 사용 API: /api/health, /api/samples, /api/sample-image/{name},

//           /api/inspect, /api/stats, /api/history

  

const $ = (id) => document.getElementById(id);

const sampleSelect = $("sampleSelect");

const inspectButton = $("inspectButton");

const previewImage = $("previewImage");

const finalResult = $("finalResult");

let inspecting = false;

  

const REASONS = {

  NG_OCR_EMPTY: "OCR이 일련번호를 읽지 못했습니다.",

  NG_FORMAT: "인식한 문자가 정해진 13자리 숫자 형식과 다릅니다.",

  NG_EXPECTED_MISMATCH: "OCR 인식값과 제품의 기준 번호가 일치하지 않습니다.",

};

  

async function getJSON(url, options = {}) {

  const response = await fetch(url, { cache: "no-store", ...options });

  let data = {};

  try { data = await response.json(); } catch { /* 오류 메시지를 아래에서 구성 */ }

  if (!response.ok) {

    throw new Error(data.detail || `요청 실패 (HTTP ${response.status})`);

  }

  return data;

}

  

function showNotice(message) {

  const notice = $("notice");

  notice.textContent = message;

  notice.hidden = false;

}

function clearNotice() {

  $("notice").hidden = true;

  $("notice").textContent = "";

}

function setServerStatus(isOnline) {

  const element = $("serverStatus");

  element.textContent = isOnline ? "● 서버 연결 정상" : "● 서버 연결 확인 필요";

  element.className = `server-status ${isOnline ? "online" : "offline"}`;

}

function statusMessage(code) {

  return REASONS[code] || `추가 확인이 필요한 검사 조건 (${code})`;

}

function humanReason(reasons) {

  return (Array.isArray(reasons) && reasons.length)

    ? reasons.map(statusMessage).join(" ")

    : "모든 검사 조건을 통과했습니다.";

}

function setCheck(id, value) {

  const el = $(id);

  if (value === null || value === undefined) {

    el.textContent = "검사 전";

    el.className = "check-status pending";

  } else {

    el.textContent = value ? "통과" : "실패";

    el.className = `check-status ${value ? "pass" : "fail"}`;

  }

}

function renderWaiting(message = "검사할 제품을 선택하고 버튼을 누르세요.") {

  finalResult.className = "decision waiting";

  $("decisionSymbol").textContent = "…";

  $("decisionTitle").textContent = "검사 대기";

  $("decisionDescription").textContent = message;

  $("ocrText").textContent = "-";

  $("expectedText").textContent = "-";

  setCheck("patternStatus", null);

  setCheck("matchStatus", null);

  $("reasonText").textContent = "검사 완료 후 판정 이유가 표시됩니다.";

  $("reasonCode").textContent = "";

  ["ruleVersion", "serviceVersion", "requestId", "inspectedAt"].forEach((id) => $(id).textContent = "-");

}

function renderError(message) {

  finalResult.className = "decision waiting";

  $("decisionSymbol").textContent = "!";

  $("decisionTitle").textContent = "요청 오류";

  $("decisionDescription").textContent = "서버에서 검사를 완료하지 못했습니다.";

  $("reasonText").textContent = message;

  $("reasonCode").textContent = "검사 결과의 NG 판정과는 다릅니다.";

}

function displaySelectedSample() {

  const fileName = sampleSelect.value;

  renderWaiting(fileName ? "'선택 이미지 검사'를 누르면 OCR 검사를 실행합니다." : undefined);

  $("selectedFileName").textContent = fileName || "선택된 이미지 없음";

  $("previewPlaceholder").hidden = Boolean(fileName);

  previewImage.hidden = true;

  previewImage.removeAttribute("src");

  if (fileName) {

    previewImage.src = `/api/sample-image/${encodeURIComponent(fileName)}`;

    inspectButton.disabled = inspecting;

  } else {

    inspectButton.disabled = true;

  }

}

previewImage.addEventListener("load", () => { previewImage.hidden = false; });

previewImage.addEventListener("error", () => {

  previewImage.hidden = true;

  $("previewPlaceholder").textContent = "이미지 미리보기를 불러올 수 없습니다.";

  $("previewPlaceholder").hidden = false;

});

sampleSelect.addEventListener("change", displaySelectedSample);

  

async function loadSamples() {

  const data = await getJSON("/api/samples");

  const samples = Array.isArray(data.samples) ? data.samples : [];

  sampleSelect.replaceChildren();

  const placeholder = document.createElement("option");

  placeholder.value = "";

  placeholder.textContent = samples.length ? "검사 이미지를 선택하세요" : "사용 가능한 샘플이 없습니다";

  sampleSelect.appendChild(placeholder);

  for (const fileName of samples) {

    const option = document.createElement("option");

    option.value = fileName;

    option.textContent = fileName;

    sampleSelect.appendChild(option);

  }

  // 기존 화면처럼 첫 이미지가 미리 보이지만, 검사하기 전에는 판정하지 않습니다.

  if (samples.length) sampleSelect.value = samples[0];

  displaySelectedSample();

}

  

function renderResult(data) {

  const isOK = data.final_decision === "OK";

  finalResult.className = `decision ${isOK ? "ok" : "ng"}`;

  $("decisionSymbol").textContent = isOK ? "✓" : "✕";

  $("decisionTitle").textContent = isOK ? "OK · 정상" : "NG · 불량";

  $("decisionDescription").textContent = isOK

    ? "13자리 형식과 기준 번호가 모두 일치합니다."

    : "형식 또는 기준 번호 검사에서 불일치가 발견되었습니다.";

  const measurement = data.measurements || {};

  $("ocrText").textContent = measurement.ocr_text || "(인식 결과 없음)";

  $("expectedText").textContent = measurement.expected_text || "-";

  setCheck("patternStatus", measurement.pattern_ok);

  setCheck("matchStatus", measurement.expected_match);

  $("reasonText").textContent = humanReason(data.reasons);

  $("reasonCode").textContent = (data.reasons || []).join(", ") || "PASS";

  $("ruleVersion").textContent = data.rule_version || "-";

  $("serviceVersion").textContent = data.service_version || "-";

  $("requestId").textContent = data.request_id || "-";

  $("inspectedAt").textContent = data.inspected_at || "-";

}

  

async function runInspection() {

  if (inspecting || !sampleSelect.value) return;

  const fileName = sampleSelect.value;

  inspecting = true;

  inspectButton.disabled = true;

  sampleSelect.disabled = true;

  inspectButton.textContent = "검사 중…";

  clearNotice();

  finalResult.className = "decision waiting";

  $("decisionSymbol").textContent = "…";

  $("decisionTitle").textContent = "OCR 분석 중";

  $("decisionDescription").textContent = "선택한 이미지에서 일련번호를 읽고 비교하고 있습니다.";

  try {

    const data = await getJSON("/api/inspect", {

      method: "POST",

      headers: { "Content-Type": "application/json" },

      body: JSON.stringify({ file_name: fileName }),

    });

    renderResult(data);

    const updates = await Promise.allSettled([loadStats(), loadHistory()]);

    if (updates.some((result) => result.status === "rejected")) {

      showNotice("검사는 완료되었지만 통계 또는 이력을 불러오지 못했습니다. 새로고침 후 확인하세요.");

    }

  } catch (error) {

    const message = error instanceof Error ? error.message : String(error);

    renderError(message);

    showNotice(`검사 요청을 처리하지 못했습니다: ${message}`);

  } finally {

    inspecting = false;

    sampleSelect.disabled = false;

    inspectButton.disabled = !sampleSelect.value;

    inspectButton.textContent = "선택 이미지 검사";

  }

}

inspectButton.addEventListener("click", runInspection);

  

async function loadStats() {

  const data = await getJSON("/api/stats");

  const total = Number(data.total) || 0;

  const ok = Number(data.ok) || 0;

  const ng = Number(data.ng) || 0;

  $("totalCount").textContent = total.toLocaleString("ko-KR");

  $("okCount").textContent = ok.toLocaleString("ko-KR");

  $("ngCount").textContent = ng.toLocaleString("ko-KR");

  const okRate = total > 0 ? ok / total * 100 : 0;

  $("okRate").textContent = total ? `${okRate.toFixed(1)}%` : "-";

  $("okRateFill").style.width = `${Math.min(100, Math.max(0, okRate))}%`;

}

  

function addCell(row, text, className = "") {

  const cell = document.createElement("td");

  cell.textContent = text;

  if (className) cell.className = className;

  row.appendChild(cell);

  return cell;

}

async function loadHistory() {

  const data = await getJSON("/api/history?limit=10");

  const items = Array.isArray(data.history) ? [...data.history].reverse() : [];

  const body = $("historyBody");

  body.replaceChildren();

  if (!items.length) {

    const row = document.createElement("tr");

    const cell = addCell(row, "아직 저장된 검사 이력이 없습니다.", "empty-table");

    cell.colSpan = 4;

    body.appendChild(row);

    return;

  }

  for (const item of items) {

    const row = document.createElement("tr");

    addCell(row, String(item.inspected_at || "-").replace("T", " "));

    addCell(row, item.file_name || "-");

    const resultCell = document.createElement("td");

    const pill = document.createElement("span");

    const isOK = item.final_decision === "OK";

    pill.className = `result-pill ${isOK ? "ok" : "ng"}`;

    pill.textContent = item.final_decision || "-";

    resultCell.appendChild(pill);

    row.appendChild(resultCell);

    addCell(row, humanReason(item.reasons));

    body.appendChild(row);

  }

}

  

async function init() {

  renderWaiting();

  const results = await Promise.allSettled([getJSON("/api/health"), loadSamples(), loadStats(), loadHistory()]);

  setServerStatus(results[0].status === "fulfilled" && results[0].value.status === "ok");

  if (results[1].status === "rejected") {

    sampleSelect.replaceChildren();

    const errorOption = document.createElement("option");

    errorOption.textContent = "이미지 목록을 가져오지 못했습니다";

    sampleSelect.appendChild(errorOption);

    inspectButton.disabled = true;

    showNotice("이미지 목록을 불러오지 못했습니다. FastAPI 서버 실행 상태를 확인하세요.");

  }

  if (results[2].status === "rejected" || results[3].status === "rejected") {

    showNotice("통계 또는 검사 이력을 불러오지 못했습니다. 서버 연결을 확인하세요.");

  }

}

init();
```

---

# 46. JavaScript는 어떤 순서로 동작하나요?

```text
Web Page 열림
        ↓
GET /api/samples
        ↓
제품 목록을 Select에 넣음
        ↓
사용자가 제품 선택
        ↓
/api/sample-image/...
        ↓
Preview 표시
        ↓
검사하기 Button 클릭
        ↓
POST /api/inspect
        ↓
JSON 검사 결과 받음
        ↓
Final OK / NG 표시
        ↓
OCR Text / Expected / Reason 표시
        ↓
GET /api/stats
        ↓
통계 갱신
        ↓
GET /api/history
        ↓
최근 검사 목록 갱신
```

---

# 47. Browser에서 Web Dashboard를 실제로 실행해 봅니다

지금까지 다음 Web 파일을 모두 작성했습니다.

```text
day10_service/static/

index.html
→ 화면 구조

style.css
→ 화면 디자인

app.js
→ FastAPI와 연결
```

이제 작성한 Web Dashboard가 실제로 동작하는지 확인합니다.

---

## 47.1 FastAPI Server가 실행 중인지 확인합니다

Server를 실행했던 Terminal이 계속 실행 중이어야 합니다.

실행되어 있지 않다면 프로젝트 최상위에서:

```bash
uvicorn day10_service.api_main:app \
  --host 127.0.0.1 \
  --port 8000 \
  --reload
```

를 실행합니다.

정상이라면 다음과 비슷한 문장이 보입니다.

```text
Uvicorn running on http://127.0.0.1:8000
```

이 Terminal은 종료하지 않습니다.

---

## 47.2 Browser에서 Dashboard를 엽니다

Chrome 또는 Edge에서 다음 주소를 엽니다.

```text
http://127.0.0.1:8000/
```

화면에 다음 항목이 보여야 합니다.

```text
Machine Vision Inspection

1. 검사 제품 선택

2. 검사 결과

3. 오늘 검사 통계

4. 최근 검사
```

HTML이나 JavaScript를 수정한 직후라면 Browser에서:

```text
Ctrl + F5
```

로 새로고침합니다.

---

## 47.3 먼저 정상 제품을 선택합니다

제품 선택 Dropdown을 누릅니다.

6일차에서 사용했던 Sample 이미지들이 나타나야 합니다.

먼저 다음 파일을 선택합니다.

```text
0637145699424_side.png
```

이 파일은 Server의:

```text
samples/day06/product/
└── 0637145699424_side.png
```

에 있는 기존 제품 이미지입니다.

![[{7E5ABE51-3515-4544-98BE-607F4F431EDA}.png]]

제품을 선택하면 화면의 Preview 영역에 해당 이미지가 보여야 합니다.

전체 흐름은 다음과 같습니다.

```text
Dropdown에서 제품 선택
        ↓
app.js
        ↓
/api/sample-image/
0637145699424_side.png
요청
        ↓
FastAPI
        ↓
제품 이미지 반환
        ↓
Web Preview 표시
```

---

## 47.4 `검사하기` Button을 누릅니다

이미지가 보이면:

```text
검사하기
```

Button을 누릅니다.

그러면 Web에서 다음 과정이 실행됩니다.

```text
검사하기 클릭
        ↓
POST /api/inspect
        ↓
file_name 전달
        ↓
FastAPI
        ↓
service_core.py
        ↓
실제 OCR 검사
        ↓
JSON Response
        ↓
Web 화면에 결과 표시
```

`0637145699424_side.png`의 실제 검사 결과는:

```text
OCR Text
0637145699424

Expected
0637145699424

Final
OK

Reason
PASS
```

가 되어야 합니다.

즉:

```text
OCR이 읽은 값
=
Expected Value
        ↓
OK
```

입니다.

![[{33B6E5DA-7F7A-4622-870B-1BCA29053D6E}.png]]

---

## 47.5 NG 제품도 한 번 확인합니다

이번에는 다음 제품을 선택합니다.

```text
0679256889404_side.png
```

그리고 다시:

```text
검사하기
```

를 누릅니다.

이 제품에서는:

```text
OCR Text
0679256889404

Expected
0679256889400
```

처럼 값이 다르므로:

```text
Final
NG

Reason
NG_EXPECTED_MISMATCH
```

가 표시되어야 합니다.

즉 Web Dashboard에서 실제로:

```text
정상 제품
→ OK

불일치 제품
→ NG + Reason
```

두 경우를 모두 확인합니다.

![[{6C03AE20-DBF8-4E41-938D-B6680F490554}.png]]

---

## 47.6 통계와 History도 확인합니다

검사를 실행하면 화면 아래의 통계도 갱신됩니다.

예:

```text
Total
→ 지금까지 검사한 횟수

OK
→ OK 판정 횟수

NG
→ NG 판정 횟수
```

그리고 `최근 검사` 영역에는 방금 검사한 결과가 추가되어야 합니다.

```text
검사 시간

파일 이름

OK / NG

Reason
```

을 확인합니다.

즉 한 번의 `검사하기` Button은:

```text
제품 검사
        ↓
결과 표시
        ↓
통계 갱신
        ↓
History 추가
```

까지 연결됩니다.

![[screencapture-127-0-0-1-8000-2026-10-19-14_13_56.png]]

---

## 47.7 최종 확인

다음 항목이 모두 정상이라면 Web Dashboard가 완성된 것입니다.

- [ ] 제품 목록이 Dropdown에 보인다.
- [ ] 제품을 선택하면 Preview 이미지가 보인다.
- [ ] `검사하기` Button이 동작한다.
- [ ] 정상 제품에서 `OK`가 표시된다.
- [ ] NG 제품에서 `NG`와 Reason이 표시된다.
- [ ] OCR Text와 Expected 값이 표시된다.
- [ ] Rule Version과 Service Version이 표시된다.
- [ ] Total / OK / NG 통계가 갱신된다.
- [ ] 최근 검사 History가 추가된다.

---

# 48. 지금 무엇이 달라졌나요?

지금까지는 같은 OCR 검사 결과를 주로:

```text
Python 실행
        ↓
Terminal
CSV
JSON
```

에서 확인했습니다.

이제는 같은 `service_core.py`의 검사 결과를:

```text
제품 선택
        ↓
검사하기
        ↓
FastAPI
        ↓
OCR 검사
        ↓
JSON Response
        ↓
Web Dashboard
        ↓
OK / NG + Reason 확인
```

형태로 사용할 수 있습니다.

즉 검사 알고리즘이 새로 바뀐 것이 아닙니다.

```text
Day06 OCR 검사
그대로 재사용

        +

FastAPI

        +

Web Dashboard
```

를 연결한 것입니다.

> 이제 사용자는 Python Terminal을 직접 보지 않아도 Web 화면에서 검사 결과를 확인할 수 있습니다.

---

# 49. 검사 코드와 Web 화면은 서로 역할이 다릅니다

현재 구조를 다시 구분합니다.

```text
service_core.py
→ 실제 OCR 검사

api_main.py
→ HTTP 요청과 응답

index.html
→ 화면 구조

style.css
→ 화면 디자인

app.js
→ Web과 API 연결
```

예를 들어 Web 화면에서:

```text
글자 크기 변경

Button 위치 변경

표 디자인 변경

Layout 변경
```

을 하더라도:

```text
ROI
OCR
Regex
Expected Value
OK / NG Rule
```

은 바뀌지 않습니다.

반대로 나중에 OCR 검사 기능을 개선하더라도 `/api/inspect`의 결과 구조를 유지하면 Web 화면은 크게 바꿀 필요가 없습니다.

```text
Inspection Logic
service_core.py
        ↓
API
/api/inspect
        ↓
Web Client
index.html + app.js
```

> 검사 기능과 화면을 분리해 두면 어느 한쪽을 수정해도 다른 쪽에 미치는 영향을 줄일 수 있습니다. 이것이 오늘 만든 서비스 구조의 중요한 장점입니다.

---

# PART8 — 지금 만든 서비스를 실제 제조환경으로 확장해 봅니다 (50~53)

---

# 50. 지금까지 완성한 결과를 먼저 정리합니다

10일차에서는 새로운 OCR 검사 기준을 만들지 않았습니다.

6일차에서 이미 검증한 OCR Frozen Rule을 재사용하여 다음 구조를 만들었습니다.

```text
제품 Sample 이미지
        ↓
Day06 OCR Frozen Rule
        ↓
service_core.py
        ↓
OCR / Regex / Expected Value
        ↓
Final OK / NG + Reason
        ↓
Python Dictionary
        ↓
FastAPI
        ↓
JSON Response
        ↓
Web Dashboard
```

그리고 실제로 다음까지 확인했습니다.

```text
Service Core 직접 Test
        ↓
FastAPI Server 실행
        ↓
Swagger API Test
        ↓
Python Client API Test
        ↓
Web Dashboard
        ↓
OK / NG
Reason
History
Stats
```

즉 머신비전 검사 결과가 더 이상 Terminal 안에서만 끝나지 않고,
API와 Web 화면을 통해 사용할 수 있는 구조로 연결되었습니다.

---

# 51. 현재 시스템과 실제 제조환경의 차이를 생각해 봅니다

현재 실습에서는 준비된 제품 이미지를 사람이 선택하여 검사했습니다.

```text
[현재 실습]

samples/day06/product/
        ↓
제품 이미지 선택
        ↓
검사하기 Button
        ↓
PC에서 검사
        ↓
OK / NG
        ↓
Web Dashboard
```

하지만 실제 제조라인에서는 입력 방식이 달라질 수 있습니다.

```text
[실제 제조환경 예]

제품 도착
        ↓
Sensor / Trigger
        ↓
산업용 Camera 촬영
        ↓
실시간 Frame
        ↓
Edge Device
        ↓
AI + Machine Vision 검사
        ↓
OK / NG + Reason
        ↓
Dashboard / MES / 제어 시스템
```

따라서 오늘 만든 시스템은 완성된 제조라인 자체가 아니라,

> **실제 제조 검사 시스템으로 발전시키기 위한 Software 구조를 경험한 것**

으로 이해합니다.

---

# 52. 무엇이 바뀌고 무엇이 그대로 남을까요?

실제 제조환경으로 확장되면 다음 부분은 바뀔 수 있습니다.

```text
현재

저장된 Sample 이미지
        ↓
PC
        ↓
Day06 OCR 검사
        ↓
사람이 검사 Button 클릭
```

향후에는:

```text
산업용 Camera Frame
        ↓
Jetson Orin Nano Super
        ↓
YOLO / TensorRT
+
Machine Vision Rule
        ↓
자동 Trigger 검사
```

처럼 바뀔 수 있습니다.

하지만 오늘 배운 다음 구조는 그대로 재사용할 수 있습니다.

```text
Measurement
        ↓
Final Decision
        ↓
Reason
        ↓
Version
        ↓
Result Dictionary
        ↓
JSON
        ↓
API
        ↓
Dashboard
```

즉 중요한 생각은 다음과 같습니다.

```text
Inspection Core
→ 현장에 맞게 변경 가능

        ↓

Result / API / Dashboard 구조
→ 계속 재사용 가능
```

---

# 53. Final Integration Challenge — 확장 구조를 직접 정리합니다

이번에는 새로운 코드를 작성하지 않습니다.

대신 오늘 만든 시스템을 실제 제조환경으로 확장한다면
어떻게 바뀔지 간단히 정리합니다.

파일:

```text
reports/day10/final_extension_plan.md
```

작성합니다.

```markdown
# Day10 Final Extension Plan

## 1. 현재 입력

현재는 어떤 데이터를 검사하는가?

작성:

## 2. 실제 제조환경의 입력

실제 현장에서는 어떤 Camera 또는 Sensor 입력으로 바뀔 수 있는가?

작성:

## 3. 검사 실행 장치

현재 PC에서 실행하는 검사를
향후 어떤 장치에서 실행할 수 있는가?

작성:

## 4. Inspection Core

어떤 AI 또는 Machine Vision 기술로 확장할 수 있는가?

작성:

## 5. 검사 결과

어떤 Measurement / Decision / Reason을 남길 것인가?

작성:

## 6. 그대로 재사용할 구조

현재 만든 구조 중
향후에도 사용할 수 있는 부분은 무엇인가?

작성:
```

여기서는 정답을 맞히는 것이 목적이 아닙니다.

> **지금 만든 프로그램의 어느 부분이 검사 기능이고, 어느 부분이 서비스 구조인지 구분할 수 있는가?**

를 확인하는 것이 중요합니다.

---

# PART9 — Portfolio 결과로 정리하고 교과9를 마무리합니다 (54~61)

---

# 54. Portfolio에서는 무엇을 보여줘야 할까요?

Portfolio에 단순히:

```text
FastAPI를 사용했습니다.

HTML / CSS / JavaScript를 작성했습니다.
```

라고 적는 것만으로는 부족합니다.

다음 흐름이 보이는 것이 중요합니다.

```text
문제
        ↓
기존 검사 결과가
Python Terminal 안에서만 확인됨

        ↓

목표
        ↓
검사 기능을
다른 프로그램에서도 사용할 수 있는
서비스 구조로 변경

        ↓

기존 검사
        ↓
Day06 OCR Frozen Rule

        ↓

Service Core
        ↓
검사 결과 표준화

        ↓

FastAPI
        ↓
HTTP API

        ↓

Web Dashboard
        ↓
OK / NG
Reason
OCR Text
Version
History
Stats

        ↓

향후 확장
        ↓
Camera + Jetson
실시간 제조 검사
```

Portfolio의 핵심은:

> **무엇을 만들었는가보다 왜 이런 구조로 만들었는가를 설명하는 것**

입니다.

---

# 55. Portfolio용 Architecture를 작성합니다

파일:

```text
reports/day10/portfolio_architecture.md
```

작성합니다.

```markdown
# Machine Vision Inspection Service Architecture

```text
Product Image
        ↓
Inspection Core
        ↓
OCR / Rule
        ↓
Measurement
        ↓
Final OK / NG
        ↓
Reason
        ↓
Service Core
        ↓
Result Dictionary
        ↓
FastAPI
        ↓
JSON Response
        ↓
Web Dashboard
```

## 핵심 설계

- Day06 Frozen OCR Rule을 다시 수정하지 않고 재사용했다.
- 검사 기능과 HTTP API 코드를 분리했다.
- Final Decision뿐 아니라 Reason과 Version도 함께 반환했다.
- Web Dashboard에서 검사 결과와 History / Stats를 확인할 수 있도록 구성했다.
- 향후 Camera와 Edge Device 기반 검사로 Inspection Core를 교체할 수 있는 구조로 만들었다.
```

---

# 56. Portfolio에 남길 Screenshot을 정리합니다

다음 결과 화면을 저장합니다.

```text
1.
FastAPI Swagger
/docs

2.
Web Dashboard
정상 제품 OK 결과

3.
Web Dashboard
NG 제품 + Reason 결과

4.
History / Stats 화면

5.
Service Architecture Diagram
```

실제 수행하지 않은 Mobile / Tablet 접속 화면은
Portfolio 필수 결과에 포함하지 않습니다.

중요한 것은:

```text
검사 코드
        ↓
API
        ↓
실제 결과
        ↓
Web Dashboard
```

까지 연결되었다는 것을 보여주는 것입니다.

---

# 57. Portfolio Summary를 작성합니다

파일:

```text
reports/day10/portfolio_summary.md
```

다음 형식으로 간단히 정리합니다.

```markdown
# Day10 Portfolio Summary

## 프로젝트

Machine Vision Inspection Service

## 문제

기존 머신비전 검사 결과는
Python Terminal 또는 파일 중심으로 확인했다.

## 목표

기존 검사 기능을
Service Core와 FastAPI로 분리하고
Web Dashboard에서 사용할 수 있도록 연결한다.

## Reused Inspection

- Inspection Module: Day06 OCR
- Rule Version: day06-ocr-v1.0

## Service

- Service Version: day10-service-v1.0
- API: FastAPI
- Client: Web Browser

## 주요 결과

- Product Sample 선택
- OCR Text 확인
- Expected Value 비교
- Final OK / NG
- NG Reason
- Rule / Service Version
- History
- OK / NG Stats

## Architecture

```text
Product Image
        ↓
Inspection Core
        ↓
Final OK / NG + Reason
        ↓
FastAPI
        ↓
JSON Response
        ↓
Web Dashboard
```

## 현재 한계

현재 실습은 Server에 저장된 Sample 이미지를 선택하여 검사한다.

## 다음 확장

향후 산업용 Camera,
실시간 Frame,
Jetson,
AI Detection,
Machine Vision Rule,
Database 또는 MES 연동 구조로 확장할 수 있다.
```

---

# 58. README에 Day10 결과를 정리합니다

`README.md`에 다음 내용을 추가합니다.

```markdown
## Day 10 — Machine Vision Service

### 목표

기존 머신비전 검사 기능을
Service Core와 FastAPI로 연결하고
Web Dashboard에서 검사 결과를 확인한다.

### Architecture

```text
Inspection Module
        ↓
Service Core
        ↓
JSON
        ↓
FastAPI
        ↓
Web Dashboard
```

### Reused Inspection

- Day06 OCR Frozen Rule
- Rule Version: day06-ocr-v1.0

### API

- GET /api/health
- GET /api/samples
- POST /api/inspect
- GET /api/latest
- GET /api/history
- GET /api/stats

### Web

- Sample Preview
- Final OK / NG
- OCR Text
- Expected Text
- NG Reason
- Rule / Service Version
- History
- OK / NG Stats

### 핵심

검사 알고리즘을 만드는 것에서 끝나지 않고,
검사 결과를 API와 Web에서 사용할 수 있는
서비스 구조로 확장했다.
```

---

# 59. Local Git에 최종 상태를 기록합니다

먼저 상태를 확인합니다.

```bash
git status
```

10일차 코드와 설정을 기록합니다.

```bash
git add \
  day10_service \
  configs/day10 \
  reports/day10 \
  src/80_day10_check_service_env.py \
  src/81_day10_test_service_core.py \
  src/82_day10_test_api.py \
  README.md \
  requirements.txt
```

Commit합니다.

```bash
git commit \
  -m "feat: complete day10 machine vision service"
```

외부 Git 저장소 사용이 제한된 환경에서는
Local Git 또는 내부 저장소까지만 사용합니다.

---

# 60. 10일차 최종 체크리스트

## Inspection

- [ ] Day06 Frozen OCR Rule을 수정하지 않고 재사용했다.
- [ ] `service_core.py`에서 실제 OCR 검사가 실행된다.
- [ ] OK / NG와 Reason을 함께 반환한다.
- [ ] Rule Version과 Service Version을 구분할 수 있다.

## API

- [ ] FastAPI Server가 실행된다.
- [ ] `/api/health`가 정상이다.
- [ ] `/api/inspect`가 실제 검사 기능을 실행한다.
- [ ] Swagger에서 Request / Response를 확인했다.
- [ ] Python Client에서 API 호출을 확인했다.

## Web

- [ ] 제품 Sample을 선택할 수 있다.
- [ ] Preview가 표시된다.
- [ ] 검사하기 Button이 동작한다.
- [ ] OK / NG와 Reason이 표시된다.
- [ ] History와 Stats가 갱신된다.

## Portfolio

- [ ] Architecture를 정리했다.
- [ ] OK / NG 결과 Screenshot을 저장했다.
- [ ] `portfolio_summary.md`를 작성했다.
- [ ] 현재 한계와 확장 방향을 설명할 수 있다.
- [ ] README와 Git을 정리했다.

---

# 61. 교과9 전체를 마무리합니다

교과9에서는 여러 기술을 따로 배우는 것이 목적이 아니었습니다.

전체 흐름은 다음과 같습니다.

```text
검사 문제 정의
        ↓
Camera / Image / ROI
        ↓
영상 품질 확인
        ↓
전처리
        ↓
Feature 측정
        ↓
Rule 기반 판정
        ↓
Matching
        ↓
Calibration
        ↓
Rule Freeze
        ↓
Test / FP / FN
        ↓
OCR
        ↓
Anomaly Detection
        ↓
Low-data Strategy
        ↓
Version / Traceability
        ↓
Experiment / Drift / Hybrid
        ↓
Service Core
        ↓
JSON
        ↓
FastAPI
        ↓
Web Dashboard
```

교과9의 시작에서는:

```text
사람이 보는 정상과 불량의 차이를
컴퓨터가 측정할 수 있는 값으로
어떻게 바꿀 것인가?
```

를 고민했습니다.

마지막에는:

```text
그 검사 결과를
실제 프로그램과 사용자가
어떻게 사용할 수 있게 만들 것인가?
```

까지 연결했습니다.

---

# 62. 교과14에서는 어떻게 발전할까요?

현재 교과9에서는:

```text
저장된 제품 이미지
        ↓
PC
        ↓
OCR / Machine Vision
        ↓
Final OK / NG
        ↓
FastAPI
        ↓
Web Dashboard
```

구조를 만들었습니다.

교과14에서는 입력과 실행 환경이 실제 제조환경에 더 가까워집니다.

```text
산업용 Camera
        ↓
실시간 Frame / Trigger
        ↓
Jetson Orin Nano Super
        ↓
YOLO / TensorRT
+
Machine Vision Rule
        ↓
Measurement
        ↓
Final OK / NG + Reason
        ↓
API
        ↓
Dashboard / 다른 시스템
```

이때 주로 바뀌는 것은:

```text
Inspection Core
```

입니다.

반대로 다음 구조는 그대로 재사용할 수 있습니다.

```text
Measurement
Final Decision
Reason
Version
JSON
API
Dashboard
```

즉 10일차에서 만든 서비스 구조는
단순한 연습용 Web 화면이 아닙니다.

> **실제 Camera와 Edge Device가 연결되는 제조 검사 시스템으로 확장하기 위한 기본 Software Architecture를 경험한 것입니다.**

---

# 최종 마무리

교과9에서 가장 중요한 것은
개별 OpenCV 함수나 알고리즘 이름을 외우는 것이 아닙니다.

```text
문제 정의
        ↓
측정
        ↓
판정
        ↓
검증
        ↓
Rule Freeze
        ↓
실패 원인 관리
        ↓
Version 관리
        ↓
Service Interface
        ↓
실제 사용 가능한 결과 제공
```

까지 하나의 흐름으로 이해하는 것이 중요합니다.

학생이 다음 내용을 자신의 말로 설명할 수 있다면
교과9의 목표를 충분히 달성한 것입니다.

> **머신비전 검사는 OK / NG를 계산하는 코드에서 끝나는 것이 아니라, 검증된 결과를 다른 프로그램과 사용자가 사용할 수 있는 구조로 제공할 수 있어야 합니다.**