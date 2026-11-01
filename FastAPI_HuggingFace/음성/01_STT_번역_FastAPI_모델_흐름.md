## 오늘 수업의 목표

- STT와 번역을 분리하여 설명할 수 있습니다.
- Whisper가 반환한 텍스트를 언제 번역 모델에 전달할지 정할 수 있습니다.
- `ko`, `en`, `auto`의 역할을 이해합니다.
- 02 실습에서 사용할 Hugging Face 모델을 확인합니다.

## 1. STT는 번역과 다릅니다

`Whisper`는 음성을 글자로 옮길 수 있습니다. 다국어 Whisper에는 영어로의 음성 번역 기능도 존재하지만, **영어 음성을 곧바로 한국어 글자로 바꿔 주는 일반적인 EN→KO 번역기가 아닙니다.** 이 수업은 두 모델을 명확히 분리합니다.

```text
[영어 음성]  I will join the meeting tomorrow.
       ↓
[1] Whisper — STT (전사)
       ↓
    I will join the meeting tomorrow.
       ↓
[2] EN→KO 번역 모델
       ↓
    내일 회의에 참석하겠습니다.
```

위 한국어 문장은 이해를 돕기 위한 예시이며 모델의 실제 출력 문장은 달라질 수 있습니다.

## 2. 사용할 Hugging Face 모델

**STT**: `openai/whisper-small` — 다국어 ASR, Task `automatic-speech-recognition`, Apache 2.0. 개발자가 `transformers.pipeline`으로 로딩할 수 있습니다.

**번역**: `Helsinki-NLP/opus-mt-tc-big-en-ko` — 영어 텍스트를 한국어 텍스트로 번역, CC BY 4.0. 모델 카드는 Transformers `pipeline('translation')` 예시를 제공하지만, 이 교재는 버전 호환성을 고려해 `AutoTokenizer`와 `AutoModelForSeq2SeqLM` 직접 호출 방식을 사용합니다.

원본 02 문서의 `AventIQ-AI/whisper_small_Automatic_speech_recognition`은 이번 수업에서는 사용하지 않습니다. 원본에 사용된 `Helsinki-NLP/opus-mt-en-ko`도 확인 가능한 영어→한국어 모델 ID로 변경했습니다.

- https://huggingface.co/openai/whisper-small
- https://huggingface.co/Helsinki-NLP/opus-mt-tc-big-en-ko

## 3. 언어 선택 정책

| 폼의 `src_lang` | Whisper 언어 지정 | 번역 수행 조건 |
|---|---|---|
| `ko` | Korean | 수행하지 않음 |
| `en` | English | 전사 결과가 있으면 시도 |
| `auto` | Whisper 자동 판단 | 전사문에 라틴문자가 있고 한글이 없으면 시도 |

`auto`는 **정확한 언어 탐지 모델이 아니라 문자열 기반의 입문용 규칙**입니다. 영어·한국어가 함께 나온 문장은 이 규칙에서 번역을 건너뛸 수 있습니다. 고도화할 때 언어 식별 모델이나 문장별 번역이 필요합니다.

## 4. API 계약을 미리 정합니다

**요청** `POST /api/stt-to-ko/`

- `multipart/form-data`의 `audio`: WAV/MP3/M4A/FLAC/OGG/WEBM/MP4 파일
- `src_lang`: `ko` 또는 `en` 또는 `auto`
- 파일 크기 최대 20MB, 길이 최대 30초 (교육 실습 제한)

**응답 예시**

```json
{
  "src_lang": "en",
  "stt_text": "I will join the meeting tomorrow.",
  "ko_text": "내일 회의에 참석하겠습니다.",
  "used_translation": true,
  "translation_status": "translated"
}
```

번역 실패 시 서버 전체를 중단하는 대신 `ko_text=stt_text`, `used_translation=false`, `translation_status="failed"`를 반환합니다. 따라서 **영어 원문이 그대로 나왔는데도 번역에 성공했다고 오인하지 않습니다.**

## 5. 실제 서비스의 흐름

```text
[음성 파일]
    ↓
[FastAPI : 파일/언어 검증]
    ↓
[임시파일 저장]
    ↓
[ffmpeg : 16 kHz 단일 채널 float32 변환]
    ↓
[Whisper pipeline : STT]
    ├─ src_lang=ko → STT 결과를 그대로 출력
    ├─ src_lang=en → 영어→한국어 번역 모델
    └─ src_lang=auto → 문자열 규칙으로 번역 여부 판단
    ↓
[JSON 응답]
    ↓
[임시 파일 삭제 (성공·실패 공통)]
```

## 6. 코드 작성 전에 의사코드로 정리합니다

```text
1. 서버가 업로드 요청을 받는다.
2. 음성 파일 존재 여부, 확장자, 용량, 언어 값을 검사한다.
3. 업로드 파일을 임시 경로에 저장한다.
4. ffmpeg로 Whisper에 맞는 샘플링 레이트로 변환한다.
5. Whisper 모델이 준비되지 않았다면 한 번만 로딩한다.
6. 소리를 텍스트로 전사한다.
7. 한국어 변환이 필요하다면 번역 모델을 로딩하고 번역한다.
8. 원문·최종문·번역 상태를 JSON으로 반환한다.
9. 성공/실패와 관계없이 임시파일을 지운다.
```

### 왜 `get_asr()`와 `get_mt()`를 만드나요?

모델을 HTTP 요청마다 다운로드·생성하면 너무 느리고 메모리도 많이 사용합니다. **지연 로딩(Lazy Loading)** 으로 첫 요청에 한 번 생성하고 같은 서버 프로세스 안에서 재사용합니다. 서버 작업자 프로세스가 여러 개라면 프로세스별로 각각 모델을 로딩합니다.

### 왜 Whisper와 번역 모델을 따로 사용할까요?

오류가 발생했을 때 어느 단계에서 문제가 생겼는지 구분할 수 있고, 번역만 다른 모델로 교체하기 쉽기 때문입니다. **STT 성공 = 번역 성공이 아닙니다.**

## 7. 다음 수업 연결

이제 02 교재에서 `speech/services.py`에 실제 모델 로딩과 STT·번역 함수를 작성하고, `speech/router.py`에 FastAPI 업로드 API를 생성합니다. 이 단계에서는 Docker나 Redis·Celery가 필요하지 않습니다. 모델을 직접 구동하는 기본기부터 확인합니다.

