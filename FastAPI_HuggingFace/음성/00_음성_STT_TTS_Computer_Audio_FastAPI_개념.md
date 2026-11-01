## 오늘 수업의 목표

1. STT와 TTS의 입력·출력 차이를 설명한다.
2. Hugging Face에서 ASR/번역/TTS 모델을 Task 기준으로 검색한다.
3. FastAPI에서 음성 파일을 받아 JSON으로 반환하는 서버 구조를 이해한다.
4. 다음 01~04 교재가 어떤 관계인지 파악한다.

## 1. STT(Speech-to-Text)란?

사람이 말한 소리를 컴퓨터가 텍스트로 바꾸는 AI 기술입니다. 한국어 음성은 한국어 텍스트로, 영어 음성은 영어 텍스트로 전사(Transcription)할 수 있습니다.

| 입력 | 출력 예시 |
|---|---|
| 한국어 음성: “오늘 날씨 어때?” | 오늘 날씨 어때? |
| 영어 음성: “How are you today?” | How are you today? |

```text
사람의 음성 → WAV/MP3/WEBM → 음성 디코딩(16kHz 단일 채널)
           → Whisper ASR 추론 → 텍스트 → FastAPI JSON 응답
```

STT 모델은 소리 패턴과 문맥을 바탕으로 적절한 문장을 예측합니다. 잡음·사투리·화자·언어에 따라 전사 오류가 발생할 수 있으므로 실제 음성 파일로 테스트해야 합니다.

## 2. STT는 어디에 사용할까요?

스마트폰 음성 입력·음성 비서·유튜브 자동 자막·강의 녹취·회의록·고객센터 상담 분석·청각 접근성 향상 등이 대표적입니다. 회의에서 *누가 말했는지*를 구분하려면 STT와 별개로 화자 분리(Diarization) 기술이 필요합니다.

STT는 **AI 파이프라인의 시작점**이 될 수 있습니다.

```text
음성 → STT → 텍스트 → 번역/검색/요약/감정 분석/LLM
```

## 3. TTS(Text-to-Speech)란?

글을 음성으로 변환하는 반대 방향의 기술입니다.

```text
입력: “안녕하세요” (문자열)
        ↓
TTS 모델
        ↓
출력: WAV 등 음성 데이터
```

STT는 **듣고 글자로 변환**, TTS는 **글을 음성으로 생성**합니다. TTS 모델은 언어 지원·음성 품질·라이선스가 각각 다르므로 실제 사용 전에 확인해야 합니다. **이번 00~04 교재의 실행 과제는 STT + 영어→한국어 텍스트 번역까지**이며 TTS 음성 합성 코드는 포함하지 않습니다. 초보자가 두 기능을 혼동하지 않도록 구분합니다.

## 4. Hugging Face에서 모델을 검색하는 방법

Hugging Face Models 화면에서 모델명보다 **Task**를 먼저 고릅니다.

| 목적 | Task/검색어 | 예 |
|---|---|---|
| 음성 → 텍스트 | `automatic-speech-recognition` | `openai/whisper-small` |
| 텍스트 → 음성 | `text-to-speech` | 선택 모델별 언어·라이선스 별도 확인 |
| 영어 → 한국어 텍스트 | `translation` | `Helsinki-NLP/opus-mt-tc-big-en-ko` |

이번 교재의 공식 Whisper는 `openai/whisper-small`입니다. 원본에 기재된 `AventIQ-AI/...`와 역할은 비슷하지만, 공식 모델 카드의 실행 예제를 기준으로 교체했습니다. STT 모델은 Apache-2.0, 번역 모델은 CC BY 4.0으로 표시됩니다. 출처 표시 등 라이선스 조건은 배포 전에 다시 확인하세요.

- Whisper: https://huggingface.co/openai/whisper-small
- 영어→한국어 번역: https://huggingface.co/Helsinki-NLP/opus-mt-tc-big-en-ko
- 공식 FastAPI 업로드 문서: https://fastapi.tiangolo.com/tutorial/request-forms-and-files/

## 5. 모델의 세 가지 역할을 구분해 봅니다

| 구분 | 의미 | 이번 수업의 예 |
|---|---|---|
| 모델 가중치 | 학습된 파라미터 | `openai/whisper-small` |
| Python 라이브러리 | 모델 로딩/추론 API | `transformers`, `torch` |
| 웹 프레임워크 | 업로드 요청과 응답 처리 | `FastAPI` |

모델 파일 자체가 FastAPI 서버는 아닙니다. FastAPI는 클라이언트와 AI 추론 코드 사이에서 HTTP 요청·응답을 담당합니다.

## 6. FastAPI에서 오디오를 처리하는 전체 구조

```text
[브라우저 / Insomnia]
     │ 음성 업로드 multipart/form-data
     ▼
[FastAPI /api/stt-to-ko/]
     │ 검증: 파일·언어·용량
     ▼
[서비스 함수 services.py]
     ├── ffmpeg: 소리 디코딩 및 16kHz 정규화
     ├── Whisper: 음성 → 텍스트
     └── 필요한 경우 번역 모델: 영어 → 한국어
     ▼
[JSON 응답]
     ├── stt_text
     ├── ko_text
     ├── used_translation
     └── translation_status
```

### Django/DRF와 비교하면

| 기존 구조 | FastAPI 구조 |
|---|---|
| `urls.py` + `include(...)` | `main.py` + `app.include_router(...)` |
| `APIView.post()` | `@router.post(...)` |
| DRF `Serializer` | FastAPI `File/Form` + 의존성 검증 함수 |
| DRF `Response({...})` | Python `dict` 반환 → JSON |
| Django `render` | Jinja2 `TemplateResponse` |
| `python manage.py runserver` | `uvicorn main:app --reload` |

**주의:** Django 기본 뷰도 JSON을 반환할 수 있습니다. DRF만 JSON을 제공한다는 식으로 설명하면 부정확합니다.

## 7. 뒤에 이어지는 실습

| 문서 | 무엇을 배우나요? | 실행 유무 |
|---|---|---|
| **00** 현재 문서 | STT/TTS·모델 검색·FastAPI 개념 | 개념 |
| **01** | STT→번역 파이프라인과 모델 선택 | 구조/설계 |
| **02** | Whisper + 번역 모델 + FastAPI 업로드 API | 첫 실행 |
| **03** | JavaScript MediaRecorder 웹 녹음 → FastAPI API | 02 확장 |
| **04** | 라우터와 입력 검증 분리·예외·테스트 | 03 리팩토링 |

> 다음 문서인 01에서는 영어 음성을 어떻게 한국어 텍스트로 만드는지 살펴본 뒤 02에서 실제 코드를 작성합니다.

