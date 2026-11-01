## 0. 전체 개발 흐름

원본 교재의 연결 관계를 유지합니다. `02`는 **입력 → 모델 추론 → 화면 출력**만 했고 DB 기록은 남기지 않았습니다. `04`는 **추론 결과를 SQLite에 저장하고 목록/상세/수정/삭제(CRUD)하는 웹 서비스**로 확장합니다. `06`에서는 이 저장 기능을 Redis·Celery 비동기 처리로 바꿉니다.

```text
[02] POST /predict/ → Hugging Face → HTML 출력(저장 없음)
                         ↓
[04] POST /new/ → 분석 → TextItem DB 저장 → 목록/상세/수정/삭제
                         ↓
[06] POST /new/ → DB 먼저 저장 → Celery 등록 → Worker 분석 → DB 갱신
```

### 0-1. 02에서 가져와야 할 파일

반드시 **02 실습을 끝낸 프로젝트 폴더**에서 계속합니다. 먼저 02의 `main.py`, `sentiment/hf_model.py`, 템플릿, CSS가 제대로 동작하는지 확인합니다. 이후 아래와 같이 파일을 추가하거나 일부 파일을 **전체 교체**합니다.

| 파일 | 작업 |
|---|---|
| `sentiment/db.py`, `sentiment/models.py` | 신규 |
| `sentiment/services.py` | 신규: 추론 함수 분리 |
| `sentiment/router.py` | 기존 파일 전체 교체: CRUD 라우터 추가 |
| `main.py` | 기존 파일 전체 교체: DB 초기화 추가 |
| `templates/sentiment/textitem_*.html` | 신규 4개 |
| `hf_model.py`, `predict.html`, `base.html`, CSS | 02의 파일 유지 |

### 0-2. SQLAlchemy 설치

```bash
cd ~/fastapi_sentiment
source .venv/bin/activate
uv pip install "sqlalchemy>=2,<3"
uv pip freeze > requirements.txt
```

초급 교재의 저장소는 원본처럼 **SQLite**를 사용합니다. `PostgreSQL`로의 교체는 다음 확장 실습에서 가능합니다. 이 문서에서는 설치 절차와 CRUD 학습을 단순하게 하기 위해 SQLite를 유지합니다.

## 1. SQLAlchemy 데이터베이스 연결 + TextItem 모델

Django의 `models.Model` 대신 FastAPI 프로젝트에 **SQLAlchemy ORM 모델**을 직접 정의합니다. `Field`와 `Mapped`는 데이터 컬럼을 의미하며 `Session`은 DB 변경을 읽고 저장하는 작업 공간입니다.

**`sentiment/db.py` — 신규 생성**

```python

# ============================================================
# ① SQLAlchemy 라이브러리 가져오기
# ============================================================
# create_engine   : 데이터베이스 연결을 관리하는 엔진 생성
# DeclarativeBase : ORM 모델의 부모 클래스
# sessionmaker    : DB 작업용 세션을 생성하는 도구

from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker


# ============================================================
# ② SQLite 데이터베이스 경로 설정
# ============================================================
# sqlite:///./sentiment.db
# : 현재 작업 폴더의 sentiment.db 파일 사용
#
# DB 파일이 없다면 실제 DB 연결 시 생성될 수 있습니다.
#
# 주의:
# 04·06 실습은 같은 프로젝트 루트에서 실행합니다.

DATABASE_URL = "sqlite:///./sentiment.db"


# ============================================================
# ③ 데이터베이스 엔진 생성
# ============================================================
# create_engine() : DB 연결을 관리하는 엔진을 준비
#
# DATABASE_URL : ②에서 지정한 SQLite 경로
#
# check_same_thread=False
# : SQLite 연결을 생성한 스레드와 다른 스레드에서도
#   사용할 수 있도록 허용하는 설정
#
# 단, 하나의 세션을 여러 요청이 동시에 공유해도
# 안전하다는 의미는 아닙니다.

engine = create_engine(
    DATABASE_URL,
    connect_args={
        "check_same_thread": False
    }
)


# ============================================================
# ④ DB 세션을 생성하는 SessionLocal 설정
# DB 세션(Session)은 FastAPI가 데이터베이스에서 조회·저장·수정·삭제 작업을 수행할 때 
# 사용하는 작업 공간입니다.
# 쉽게 말하면 데이터베이스에 작업을 요청하고, 그 작업을 관리하는 담당자라고 생각하면 됩니다.
# 이 코드는 DB 세션 자체가 아니라, DB 세션을 생성하는 도구를 만드는 코드입니다.
# ============================================================
# sessionmaker() : 새로운 DB 세션을 만드는 팩토리 생성
#
# bind=engine
# : ③에서 생성한 DB 엔진에 연결
#
# autoflush=False
# : 쿼리 전에 변경 내용을 자동 반영하지 않도록 설정
#
# expire_on_commit=False
# : commit 이후에도 이미 읽어온 객체 속성값 유지
#
# 주의:
# 여기서는 세션 자체를 생성하는 것이 아니라
# 세션을 만드는 SessionLocal을 준비합니다.

SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
    expire_on_commit=False
)


# ============================================================
# ⑤ ORM 모델의 공통 부모 클래스 생성
# ============================================================
# Base를 상속하여 데이터베이스 테이블 모델을 만듭니다.
#
# 예:
# class TextItem(Base):
#     ...
#
# 실제 TextItem 테이블 구조는 models.py에서 정의합니다.

class Base(DeclarativeBase):

    # 부모 클래스에 추가할 내용이 없으므로 pass 사용
    pass


# ============================================================
# ⑥ FastAPI에서 사용할 DB 세션 제공 함수
# ============================================================
# get_db() : API 요청 처리에 사용할 세션 제공
#
# router.py에서 다음과 같이 사용합니다.
#
# db: Session = Depends(get_db)
#
# FastAPI가 이 함수를 호출하여 세션을 전달합니다.

def get_db():

    # --------------------------------------------------------
    # ⑦ 새로운 데이터베이스 세션 생성
    # --------------------------------------------------------
    # SessionLocal()을 호출하여 DB 작업용 세션 생성
    #
    # with 문은 작업이 끝나면 세션을 닫도록 관리합니다.

    with SessionLocal() as db:

        # ----------------------------------------------------
        # ⑧ FastAPI 라우터에 DB 세션 전달
        # ----------------------------------------------------
        # yield : 생성한 DB 세션을 라우터에 제공
        #
        # 라우터는 이 세션을 사용하여
        # 조회, 추가, 수정, 삭제 작업을 수행합니다.

        yield db

    # --------------------------------------------------------
    # ⑨ DB 세션 종료
    # --------------------------------------------------------
    # 라우터에서 세션 사용이 끝나면
    # with 블록이 종료되면서 세션을 닫습니다.
    #
    # 단, 데이터 저장(commit)은 자동으로 하지 않습니다.
    # CRUD 코드에서 필요한 경우 db.commit()을 호출합니다.
```

**의사코드**

```
===========================================
  [1단계] 데이터베이스 사용 환경 준비
===========================================

[① SQLAlchemy 라이브러리 가져오기]

    create_engine 가져오기
    DeclarativeBase 가져오기
    sessionmaker 가져오기

                  ↓

[② SQLite 데이터베이스 경로 설정]

    DATABASE_URL 설정

    sqlite:///./sentiment.db

    프로젝트 루트의 sentiment.db 사용

                  ↓

[③ 데이터베이스 엔진 생성]

    create_engine() 실행

    SQLite 연결 설정 준비
    엔진 객체 생성

    engine 변수에 저장

                  ↓

[④ DB 세션 생성 도구 설정]

    sessionmaker() 실행

    engine 연결
    autoflush=False
    expire_on_commit=False

    SessionLocal 생성

                  ↓

[⑤ ORM 공통 부모 클래스 생성]

    Base 클래스 정의

    DeclarativeBase 상속

    이후 models.py에서

        class TextItem(Base):
            ...

    형태로 테이블 모델 작성


===========================================
  [2단계] FastAPI 요청 시 DB 세션 사용
===========================================

    사용자가 CRUD 기능 요청

    예: GET /items/

                  ↓

    sentiment/router.py

    db: Session = Depends(get_db)

                  ↓

[⑥ get_db() 함수 호출]

    FastAPI가 DB 세션 요청

                  ↓

[⑦ 새로운 DB 세션 생성]

    SessionLocal() 실행

    DB 작업용 세션 생성

                  ↓

[⑧ DB 세션 전달]

    yield db

    라우터 함수에서 세션 사용

        ├─ SELECT : 데이터 조회
        ├─ INSERT : 데이터 추가
        ├─ UPDATE : 데이터 수정
        └─ DELETE : 데이터 삭제

    저장이 필요하면 db.commit()

                  ↓

[⑨ DB 세션 종료]

    라우터에서 세션 사용 완료

    with 블록 종료

    DB 세션 닫기

                  ↓

        HTTP 응답 처리 완료
```

---
SQLAlchemy를 사용하여 SQLite 데이터베이스에 `text_items`라는 테이블을 만들기 위한 구조를 정의합니다.

파일별 역할을 구분하면

```
sentiment/
│
├── db.py
│    └── DB 연결 및 세션 관리
│
├── models.py
│    └── DB 테이블 구조 정의  
│
├── services.py
│    └── Hugging Face AI 감정분석 실행
│
└── router.py
     └── CRUD 요청 및 결과 처리

tests/
└── test_web.py
     └── API 정상 동작 여부 검증
```

![[Pasted image 20261101112103.png]]

**`sentiment/models.py` — 신규 생성**

```python

# ============================================================
# ① 날짜와 시간 처리 라이브러리 가져오기
# ============================================================
# datetime : 날짜와 시간을 표현하는 클래스
# timezone : 시간대 정보 설정
#
# timezone.utc : UTC(세계 표준시) 사용

from datetime import datetime, timezone


# ============================================================
# ② SQLAlchemy 데이터베이스 컬럼 타입 가져오기
# ============================================================
# Integer : 정수형 데이터
# Text    : 긴 문자열 데이터
# String  : 길이를 지정할 수 있는 문자열 데이터
# Float   : 실수형 데이터
# DateTime: 날짜 및 시간 데이터

from sqlalchemy import DateTime, Float, Integer, String, Text


# ============================================================
# ③ ORM 모델 작성에 필요한 기능 가져오기
# ============================================================
# Mapped        : Python 속성과 DB 컬럼의 연결 타입 표시
# mapped_column : DB 테이블 컬럼의 상세 설정

from sqlalchemy.orm import Mapped, mapped_column


# ============================================================
# ④ db.py에서 공통 부모 클래스 가져오기
# ============================================================
# Base는 db.py에서 만든 ORM 모델의 공통 부모 클래스입니다.
#
# TextItem이 Base를 상속하면
# SQLAlchemy에서 테이블 모델로 관리할 수 있습니다.

from .db import Base


# ============================================================
# ⑤ TextItem ORM 모델 및 테이블 이름 정의
# ============================================================
# TextItem : Python에서 사용하는 ORM 모델 클래스
#
# __tablename__ : 실제 데이터베이스 테이블 이름
#
# 이 클래스의 객체 하나는 DB의 데이터 한 행에 대응합니다.

class TextItem(Base):

    __tablename__ = "text_items"


    # --------------------------------------------------------
    # ⑥ id : 데이터 고유번호 (기본키)
    # --------------------------------------------------------
    # Integer          : 정수형
    # primary_key=True : 기본키(PK) 지정
    #
    # 각 감정분석 데이터를 구별하는 고유번호입니다.
    # SQLite에서는 정수 기본키로 자동 번호 생성이 가능합니다.

    id: Mapped[int] = mapped_column(
        Integer,
        primary_key=True
    )


    # --------------------------------------------------------
    # ⑦ text : 사용자가 입력한 문장
    # --------------------------------------------------------
    # Text           : 긴 문자열 저장
    # nullable=False : NULL 저장 불가
    #
    # 예: "오늘 정말 기분이 좋아요."
    #
    # 주의:
    # nullable=False는 NULL을 금지하는 설정입니다.
    # 빈 문자열까지 자동으로 검사하지는 않습니다.

    text: Mapped[str] = mapped_column(
        Text,
        nullable=False
    )


    # --------------------------------------------------------
    # ⑧ label : AI 감정분석 결과
    # --------------------------------------------------------
    # String(50)      : 최대 50자로 설정한 문자열 타입
    # default=""      : 값을 지정하지 않으면 빈 문자열 사용
    # nullable=False  : NULL 저장 불가
    #
    # 예: Positive, Negative, Neutral

    label: Mapped[str] = mapped_column(
        String(50),
        default="",
        nullable=False
    )


    # --------------------------------------------------------
    # ⑨ score : AI 감정분석 점수
    # --------------------------------------------------------
    # Float         : 실수형 데이터
    # float | None  : 실수 또는 None 허용
    # nullable=True : DB에 NULL 저장 가능
    #
    # 예: 0.95
    #
    # 아직 AI 분석 결과가 없다면 NULL로 둘 수 있습니다.

    score: Mapped[float | None] = mapped_column(
        Float,
        nullable=True
    )


    # --------------------------------------------------------
    # ⑩ created_at : 데이터 생성 시각
    # --------------------------------------------------------
    # DateTime(timezone=True) : 시간대 정보를 고려한 타입 선언
    # default=lambda: ...    : 기본 생성 시각을 계산하는 함수
    # datetime.now(timezone.utc) : 현재 UTC 시각
    #
    # 새 데이터를 INSERT할 때 별도 값을 지정하지 않으면
    # 기본값으로 현재 UTC 시각을 사용합니다.

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        default=lambda: datetime.now(timezone.utc)
    )
```

**의사코드**

```
===========================================
  [1단계] ORM 모델 작성에 필요한 기능 준비
===========================================

[① 날짜 및 시간 라이브러리 가져오기]

    datetime 가져오기
    timezone 가져오기

    데이터 생성 시각을 기록할 준비


                  ↓

[② DB 컬럼 타입 가져오기]

    Integer  : 정수
    Text     : 긴 문자열
    String   : 문자열
    Float    : 실수
    DateTime : 날짜와 시간


                  ↓

[③ ORM 모델 기능 가져오기]

    Mapped 가져오기
        → Python 속성과 DB 컬럼 연결

    mapped_column 가져오기
        → 컬럼 타입과 제약조건 설정


                  ↓

[④ Base 클래스 가져오기]

    db.py에서 Base 가져오기

    SQLAlchemy ORM 모델을
    정의할 준비 완료


===========================================
  [2단계] 데이터베이스 테이블 구조 정의
===========================================

[⑤ TextItem 모델 생성]

    TextItem 클래스 정의

    Base 클래스 상속

    테이블 이름 지정
        → text_items


                  ↓

[⑥ id 컬럼 정의]

    데이터 타입 : Integer

    primary_key=True

    각 데이터에 고유번호 부여


                  ↓

[⑦ text 컬럼 정의]

    데이터 타입 : Text

    NULL 저장 금지

    사용자가 입력한 문장 저장


                  ↓

[⑧ label 컬럼 정의]

    데이터 타입 : String(50)

    기본값 : 빈 문자열

    NULL 저장 금지

    AI 감정분석 결과 저장


                  ↓

[⑨ score 컬럼 정의]

    데이터 타입 : Float

    NULL 허용

    AI 감정분석 점수 저장


                  ↓

[⑩ created_at 컬럼 정의]

    데이터 타입 : DateTime

    기본값 : 현재 UTC 시각

    데이터 생성 시각 저장


===========================================
  [3단계] ORM 모델을 실제 DB 테이블에 반영
===========================================

    main.py에서 models.py 가져오기

                  ↓

    Base.metadata.create_all(bind=engine)

                  ↓

    없는 테이블을 DB에 생성

                  ↓

       SQLite : sentiment.db

                  ↓

        text_items 테이블 생성

        ┌────────────────────────┐
        │ id         INTEGER     │
        │ text       TEXT        │
        │ label      VARCHAR(50) │
        │ score      FLOAT       │
        │ created_at DATETIME    │
        └────────────────────────┘
```

| 필드 | 설명 |
|---|---|
| `id` | 기본키 |
| `text` | 사용자 원문 |
| `label` | 감정 분석 라벨 |
| `score` | 모델 점수, 저장 전에는 NULL 가능 |
| `created_at` | 최초 생성 시각(UTC) |

**중요:** `Base.metadata.create_all()`은 **없는 테이블을 생성**할 뿐 기존 테이블의 컬럼을 수정하는 마이그레이션이 아닙니다. 04 새 프로젝트에서는 충분하지만 06에서 필드가 늘어날 때에는 별도 변경 단계가 필요하므로 `06`에서 처리합니다. 실무에서는 Alembic 사용을 권장합니다.

## 2. `analyze_text()` 서비스 함수는 무엇이며, 왜 필요할까요?

02 실습에서는 사용자가 문장을 입력하면 `router.py`에서 Hugging Face AI 모델을 호출하여 감정분석을 실행하고, 그 결과를 HTML 화면에 보여주었습니다.

이번 04 실습에서는 여기에 **데이터베이스 저장 기능(CRUD)**을 추가합니다. 사용자가 새로운 글을 작성하면 AI가 문장을 분석한 후 결과를 DB에 저장하고, 기존 글을 수정하면 수정된 문장을 다시 분석하여 DB에 저장해야 합니다.

즉, **글을 새로 작성할 때와 수정할 때 모두 AI 감정분석이라는 동일한 작업이 필요합니다.**

그런데 AI 분석 코드를 생성(Create) 함수와 수정(Update) 함수에 각각 작성하면 같은 코드가 반복됩니다. 나중에 분석 방법을 변경할 때도 두 곳을 모두 수정해야 합니다.

이러한 중복을 없애기 위해 `sentiment/services.py`라는 파일을 새로 만들고, AI 감정분석을 담당하는 `analyze_text()` 함수를 작성합니다.

`analyze_text()`는 분석할 문장을 전달받아 Hugging Face 모델을 실행하고, 감정분석 결과인 `label`과 `score`를 반환하는 함수입니다.

이렇게 역할을 나누면 `router.py`는 사용자의 요청과 DB 저장을 담당하고, `services.py`는 AI 감정분석만 담당합니다.

**전체 흐름을 간단히 정리하면 다음과 같습니다.**

```
사용자가 글 작성 또는 수정
            ↓
        router.py
       (요청 처리)
            ↓
        services.py
       analyze_text()
            ↓
        hf_model.py
     (Hugging Face 모델)
            ↓
      AI 감정분석 실행
            ↓
       label / score
            ↓
        router.py
       (결과 DB 저장)
```

**핵심:** `services.py`는 여러 기능에서 공통으로 사용하는 AI 분석 코드를 한곳에 모아 놓은 파일입니다. 덕분에 같은 코드를 반복해서 작성하지 않아도 되고, 이후 다른 기능에서도 `analyze_text()`를 재사용할 수 있습니다.

**코드흐름**

```text
문장 입력 → 공백 검사 → get_classifier() 호출
→ classifier(text) 분석 → normalize_prediction() 결과 통일
→ {label, score} 반환
```

**`sentiment/services.py` — 신규 생성**

```python
# ============================================================
# ① Hugging Face 감정분석 관련 함수 가져오기
# ============================================================
# hf_model.py에서 앞서 작성한 두 함수를 가져옵니다.
#
# get_classifier()
# : Hugging Face 감정분석 모델 객체를 가져오는 함수
#
# normalize_prediction()
# : AI 분석 결과를 label과 score 형식으로 정리하는 함수

from .hf_model import get_classifier, normalize_prediction


# ============================================================
# ② 감정분석 서비스 함수 정의
# ============================================================
# analyze_text() : 입력 문장을 AI로 분석하는 공통 함수
#
# text: str  : 분석할 문장을 문자열로 전달받음
# -> dict   : 분석 결과를 딕셔너리로 반환
#
# 이 함수는 생성(Create)과 수정(Update) 기능에서
# 공통으로 호출하여 사용합니다.

def analyze_text(text: str) -> dict:
    """
    입력된 문장 하나를 AI 모델로 감정분석하고,
    label과 score를 반환합니다.

    HTTP 요청 처리와 DB 저장은 담당하지 않습니다.
    """


    # --------------------------------------------------------
    # ③ 입력 문장의 앞뒤 공백 제거
    # --------------------------------------------------------
    # strip() : 문자열 양쪽의 공백을 제거합니다.
    #
    # 예: "  오늘 기분이 좋아요  "
    #     → "오늘 기분이 좋아요"

    cleaned = text.strip()


    # --------------------------------------------------------
    # ④ 빈 문장인지 검사
    # --------------------------------------------------------
    # cleaned가 빈 문자열이면 분석을 진행하지 않습니다.
    #
    # raise ValueError()
    # : 잘못된 입력값이라는 오류를 발생시킵니다.
    #
    # 참고:
    # 문장 길이 1~500자 검사는 router.py에서 수행합니다.

    if not cleaned:
        raise ValueError(
            "빈 문장은 분석할 수 없습니다."
        )


    # --------------------------------------------------------
    # ⑤ Hugging Face 감정분석 모델 가져오기
    # --------------------------------------------------------
    # hf_model.py의 get_classifier() 함수를 호출합니다.
    #
    # 최초 호출 시 모델을 로딩하고,
    # 이후에는 캐시된 모델 객체를 재사용합니다.

    classifier = get_classifier()


    # --------------------------------------------------------
    # ⑥ 입력 문장에 대한 AI 감정분석 실행
    # --------------------------------------------------------
    # classifier() : 실제 감정분석을 수행합니다.
    #
    # cleaned         : 분석할 문장
    # truncation=True : 입력 토큰이 너무 길면 잘라서 처리
    # max_length=512  : 최대 입력 토큰 수 지정
    #
    # output에는 Hugging Face 모델의 예측 결과가 저장됩니다.

    output = classifier(
        cleaned,
        truncation=True,
        max_length=512
    )


    # --------------------------------------------------------
    # ⑦ AI 분석 결과를 정리하여 반환
    # --------------------------------------------------------
    # normalize_prediction()
    # : Hugging Face 결과에서 label과 score를 추출합니다.
    #
    # 예시 반환값:
    # {
    #     "label": "Positive",
    #     "score": 0.95
    # }
    #
    # 정리된 결과를 호출한 router.py에 전달합니다.

    return normalize_prediction(output)
```

**의사코드**

```
===========================================
  [1단계] AI 분석에 필요한 함수 준비
===========================================

[① Hugging Face 관련 함수 가져오기]

    hf_model.py에서

    get_classifier() 가져오기
    normalize_prediction() 가져오기


                  ↓


===========================================
  [2단계] 감정분석 서비스 함수 실행
===========================================

    사용자가 글 작성 또는 수정

                  ↓

    router.py에서 analyze_text() 호출

                  ↓

[② analyze_text(text) 함수 실행]

    분석할 문장 전달받기

    예: "  오늘 기분이 좋아요  "


                  ↓

[③ 입력 문장 공백 제거]

    text.strip() 실행

    앞뒤 공백 제거

    cleaned = "오늘 기분이 좋아요"


                  ↓

[④ 빈 문장 검사]

    cleaned가 빈 문자열인가?

           │
      ┌────┴────┐
      │         │
      예       아니요
      │         │
      ▼         ▼

    오류 발생   다음 단계 진행

    ValueError
                │
                ▼

[⑤ 감정분석 모델 가져오기]

    get_classifier() 호출

    모델이 처음 사용된다면
        Hugging Face 모델 로딩

    이미 로딩된 모델이 있다면
        기존 모델 재사용

    classifier에 모델 객체 저장


                  ↓

[⑥ AI 감정분석 실행]

    classifier(cleaned) 실행

    AI 모델이 문장의 감정을 분석

    예시 결과:

    [
        {
            "label": "Positive",
            "score": 0.95
        }
    ]

    output 변수에 저장


                  ↓

[⑦ 분석 결과 정리 후 반환]

    normalize_prediction(output) 실행

    리스트 형태의 분석 결과를
    딕셔너리 형태로 정리

    {
        "label": "Positive",
        "score": 0.95
    }

                  ↓

    분석 결과를 router.py에 반환

                  ↓

    router.py에서

    TextItem 객체 생성 또는 수정

                  ↓

    SQLAlchemy DB 세션을 통해
    SQLite에 분석 결과 저장
```

**각 코드의 역할**

| 순번  | 코드                                    | 핵심 역할                      |
| --- | ------------------------------------- | -------------------------- |
| ①   | `from .hf_model import ...`           | AI 모델 호출 함수와 결과 정리 함수 가져오기 |
| ②   | `def analyze_text(text: str) -> dict` | 공통 감정분석 함수 정의              |
| ③   | `text.strip()`                        | 문장 앞뒤 공백 제거                |
| ④   | `if not cleaned`                      | 빈 문장 검사 및 오류 처리            |
| ⑤   | `get_classifier()`                    | Hugging Face 모델 가져오기       |
| ⑥   | `classifier(cleaned, ...)`            | 실제 AI 감정분석 실행              |
| ⑦   | `normalize_prediction(output)`        | 결과를 딕셔너리로 정리하여 반환          |

`router.py`, `services.py`, `hf_model.py`는 어떻게 연결되나요?

```
        router.py
     (사용자 요청 처리)
             │
             │ analyze_text(text)
             ▼
        services.py
     (공통 감정분석 함수)
             │
             ├── ① 문장 공백 제거
             ├── ② 빈 문장 검사
             │
             ▼
        hf_model.py
     (Hugging Face 모델 관리)
             │
             ├── get_classifier()
             └── normalize_prediction()
             │
             ▼
        services.py
      (label / score 반환)
             │
             ▼
        router.py
       (분석 결과 저장)
             │
             ▼
         db.py
       (DB 세션 제공)
             │
             ▼
        models.py
     (TextItem 테이블 모델)
             │
             ▼
       sentiment.db
       (SQLite 저장)
```

---
## 3. FastAPI CRUD 라우터 작성

Django의 `ListView`, `DetailView`, `CreateView`, `UpdateView`, `DeleteView` 대신 각각 `@router.get`, `@router.post`를 선언합니다. 수정 시 기존 문장과 달라졌다면 감정분석을 **다시 실행**하고 새 라벨·점수를 DB에 저장합니다. 삭제는 실수 방지를 위해 GET 확인화면, POST 실제 삭제로 분리합니다.

### 3-1. HTTP 경로 설계

| 메서드 | 경로 | 기능 |
|---|---|---|
| GET / POST | `/predict/` | 분석만, DB 저장하지 않음 |
| GET | `/items/` | 목록 |
| GET / POST | `/new/` | 입력 폼 / 생성·분석·저장 |
| GET | `/items/{item_id}/` | 상세 |
| GET / POST | `/items/{item_id}/edit/` | 수정 폼 / 다시 분석·수정 |
| GET / POST | `/items/{item_id}/delete/` | 삭제 확인 / 실제 삭제 |

**파일 작성 전에 코드 처리**

```text
Create:
  입력을 받는다 → 1~500자 검사 → analyze_text()
  → TextItem 인스턴스 생성 → db.add → db.commit → 상세 페이지 이동
Update:
  id로 조회 → 없으면 404 → text 검사 → analyze_text()
  → 객체 필드 변경 → db.commit → 상세로 이동
Delete:
  id로 조회 → 없으면 404 → db.delete → db.commit → 목록 이동
```

**`sentiment/router.py` — 전체 교체**

```python

# ============================================================
# ① [02→04 수정] 필요한 라이브러리 가져오기
# ============================================================
# APIRouter       : URL과 처리 함수 연결
# Depends         : DB 세션 등의 의존성 주입
# Form            : HTML 입력값 받기
# HTTPException   : 404 등의 HTTP 오류 발생
# Request         : 브라우저 요청 정보
# HTMLResponse    : HTML 응답
# RedirectResponse: 다른 URL로 이동
# Jinja2Templates : HTML 템플릿 출력
#
# [04 신규]
# select     : DB 데이터 조회 SQL 구성
# Session    : DB 세션 타입
# get_db     : db.py에서 DB 세션 가져오기
# TextItem   : models.py에서 테이블 모델 가져오기
# analyze_text: services.py의 AI 감정분석 함수
#
# [02에서 제거]
# get_classifier, normalize_prediction
# → 이제 services.py에서 호출하므로
#   router.py에서 직접 가져오지 않습니다.

from fastapi import (
    APIRouter,
    Depends,
    Form,
    HTTPException,
    Request
)
from fastapi.responses import HTMLResponse, RedirectResponse
from fastapi.templating import Jinja2Templates

from sqlalchemy import select
from sqlalchemy.orm import Session

from .db import get_db
from .models import TextItem
from .services import analyze_text


# ============================================================
# ② [02 유지] 라우터와 HTML 템플릿 설정
# ============================================================
# router    : API 경로를 등록할 객체
# templates : HTML 파일이 들어 있는 폴더

router = APIRouter()
templates = Jinja2Templates(directory="templates")


# ============================================================
# ③ [04 신규] 공통 HTML 출력 함수
# ============================================================
# 02에서는 TemplateResponse()를 여러 번 작성했습니다.
#
# 이제 render() 함수를 만들어 HTML 출력 코드를 줄입니다.
#
# request : 브라우저 요청 정보
# name    : 출력할 HTML 파일 이름
# **context : HTML에 전달할 데이터
#
# 예:
# render(request, "predict.html", text="안녕하세요")
#
# 실제 템플릿 경로:
# sentiment/predict.html

def render(request, name, **context):

    return templates.TemplateResponse(
        request=request,
        name="sentiment/" + name,
        context=context
    )


# ============================================================
# ④ [02→04 수정] 입력값 검사를 공통 함수로 분리
# ============================================================
# 02에서는 predict() 안에서 문장 길이를 검사했습니다.
#
# 04에서는 감정분석, 글 작성, 글 수정에
# 같은 검사가 필요하므로 checked()로 분리합니다.
#
# 정상 입력 : 정리된 문장 반환
# 잘못된 입력 : None 반환

def checked(text):

    # 입력 문장 앞뒤 공백 제거
    cleaned = text.strip()

    # 문장이 1~500자가 아니면 None 반환
    if not 1 <= len(cleaned) <= 500:
        return None

    # 정상적인 입력 문장 반환
    return cleaned


# ============================================================
# ⑤ [04 신규] 첫 화면 접속 시 목록으로 이동
# ============================================================
# 사용자가 / 주소로 접속하면 /items/로 이동합니다.
#
# 303 : 다른 URL로 이동하도록 안내하는 상태 코드

@router.get("/", include_in_schema=False)
def home():

    return RedirectResponse(
        "/items/",
        status_code=303
    )


# ============================================================
# ⑥ [02→04 수정] 기존 감정분석 입력 화면
# ============================================================
# 기존 GET /predict/ 기능은 그대로 유지합니다.
#
# 변경된 점:
# TemplateResponse() 대신 render() 함수 사용

@router.get(
    "/predict/",
    response_class=HTMLResponse,
    name="predict_page"
)
def predict_page(request: Request):

    return render(
        request,
        "predict.html",
        text="",
        result=None,
        error=None
    )


# ============================================================
# ⑦ [02→04 수정] 감정분석 POST 요청과 입력 검사
# ============================================================
# 사용자가 [분석하기] 버튼을 누르면 실행됩니다.
#
# 02 : predict() 내부에서 직접 문장 검사
# 04 : checked() 공통 함수를 사용
#
# 잘못된 입력은 HTTP 400 응답을 반환합니다.

@router.post(
    "/predict/",
    response_class=HTMLResponse,
    name="predict"
)
def predict(
    request: Request,
    text: str = Form(default="")
):

    # 공통 함수로 입력값 검사
    value = checked(text)

    # 정상 문장이 아니라면 오류 화면 출력
    if value is None:

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
    # ⑧ [02→04 수정] 서비스 함수를 통한 AI 감정분석
    # --------------------------------------------------------
    # 02:
    # get_classifier()로 모델을 가져온 후 직접 분석
    #
    # 04:
    # services.py의 analyze_text() 함수 호출
    #
    # analyze_text()는 label, score를 반환합니다.
    #
    # 주의:
    # /predict/는 기존처럼 분석 결과만 출력하며
    # SQLite에는 저장하지 않습니다.

    return render(
        request,
        "predict.html",
        text=value,
        result={
            "input": value,
            **analyze_text(value)
        },
        error=None
    )


# ============================================================
# ⑨ [04 신규] 저장된 감정분석 글 목록 조회 (READ)
# ============================================================
# GET /items/
#
# Depends(get_db) :
# db.py에서 DB 세션을 받아옵니다.
#
# select(TextItem) :
# text_items 테이블의 데이터를 조회합니다.
#
# order_by(TextItem.id.desc()) :
# id가 큰 글부터 정렬합니다.
#
# .all() :
# 조회된 모든 결과를 가져옵니다.

@router.get(
    "/items/",
    response_class=HTMLResponse,
    name="item_list"
)
def item_list(
    request: Request,
    db: Session = Depends(get_db)
):

    # DB에 저장된 전체 글 조회
    items = db.scalars(
        select(TextItem).order_by(TextItem.id.desc())
    ).all()

    # 조회 결과를 목록 HTML에 전달
    return render(
        request,
        "textitem_list.html",
        items=items
    )


# ============================================================
# ⑩ [04 신규] 새 글 작성 화면 열기 (CREATE 화면)
# ============================================================
# GET /new/
#
# 사용자가 새 글을 작성할 수 있도록 HTML 폼을 보여줍니다.
# 아직 DB에 저장하거나 AI 분석을 실행하지 않습니다.

@router.get(
    "/new/",
    response_class=HTMLResponse,
    name="item_create_page"
)
def item_create_page(request: Request):

    return render(
        request,
        "textitem_form.html",
        item=None,
        text="",
        error=None
    )


# ============================================================
# ⑪ [04 신규] 새 글 저장 요청 및 입력값 검사
# ============================================================
# POST /new/
#
# 사용자가 [저장] 버튼을 누르면 실행됩니다.
#
# text : 입력 문장
# db   : SQLite 작업에 사용할 DB 세션
#
# 먼저 checked()로 입력 문장을 검사합니다.

@router.post("/new/", name="item_create")
def item_create(
    request: Request,
    text: str = Form(default=""),
    db: Session = Depends(get_db)
):

    # 입력 문장 검사
    value = checked(text)

    # 잘못된 문장이라면 저장하지 않고 오류 출력
    if value is None:

        return templates.TemplateResponse(
            request=request,
            name="sentiment/textitem_form.html",
            context={
                "item": None,
                "text": text,
                "error": "문장은 1~500자로 입력하세요."
            },
            status_code=400
        )


    # --------------------------------------------------------
    # ⑫ [04 신규] AI 분석 → SQLite 저장 → 상세 페이지 이동
    # --------------------------------------------------------
    # 1. analyze_text()로 AI 감정분석 실행
    # 2. TextItem 객체 생성
    # 3. db.add()로 저장할 객체 등록
    # 4. db.commit()으로 저장 확정
    # 5. db.refresh()로 저장된 정보 다시 읽기
    # 6. 상세 페이지로 이동
    #
    # 중요:
    # 현재 04는 동기식 처리입니다.
    # AI 분석이 완료된 후 DB에 저장하고 응답합니다.

    # AI 감정분석 실행
    result = analyze_text(value)

    # models.py에 정의한 TextItem 모델 객체 생성
    item = TextItem(
        text=value,
        label=result["label"],
        score=result["score"]
    )

    # DB 세션에 새 객체 등록
    db.add(item)

    # DB 저장 확정
    db.commit()

    # 저장된 데이터 다시 읽기
    db.refresh(item)

    # 저장된 글의 상세 페이지로 이동
    return RedirectResponse(
        request.url_for(
            "item_detail",
            item_id=item.id
        ),
        status_code=303
    )


# ============================================================
# ⑬ [04 신규] 저장된 글 상세 조회 (READ)
# ============================================================
# GET /items/{item_id}/
#
# item_id : 조회하려는 글의 고유번호(PK)
#
# db.get(TextItem, item_id) :
# 해당 id를 가진 글을 조회합니다.
#
# 데이터가 없다면 HTTP 404 오류를 발생시킵니다.

@router.get(
    "/items/{item_id}/",
    response_class=HTMLResponse,
    name="item_detail"
)
def item_detail(
    request: Request,
    item_id: int,
    db: Session = Depends(get_db)
):

    # 기본키(id)로 데이터 조회
    item = db.get(TextItem, item_id)

    # 해당 글이 없으면 404 오류
    if item is None:
        raise HTTPException(
            status_code=404,
            detail="해당 글이 없습니다."
        )

    # 조회 결과를 상세 페이지에 전달
    return render(
        request,
        "textitem_detail.html",
        item=item
    )


# ============================================================
# ⑭ [04 신규] 기존 글 수정 화면 열기 (UPDATE 화면)
# ============================================================
# GET /items/{item_id}/edit/
#
# 수정할 글을 DB에서 조회합니다.
# 기존 입력 문장을 HTML 폼에 표시합니다.
#
# 이 단계에서는 실제 수정 작업을 하지 않습니다.

@router.get(
    "/items/{item_id}/edit/",
    response_class=HTMLResponse,
    name="item_update_page"
)
def item_update_page(
    request: Request,
    item_id: int,
    db: Session = Depends(get_db)
):

    # 수정할 글 조회
    item = db.get(TextItem, item_id)

    # 글이 없으면 404
    if item is None:
        raise HTTPException(status_code=404)

    # 기존 내용을 수정 폼에 전달
    return render(
        request,
        "textitem_form.html",
        item=item,
        text=item.text,
        error=None
    )


# ============================================================
# ⑮ [04 신규] 기존 글 수정 요청 및 입력값 검사
# ============================================================
# POST /items/{item_id}/edit/
#
# 수정할 데이터를 먼저 조회하고,
# 새로운 문장이 정상 입력인지 검사합니다.

@router.post(
    "/items/{item_id}/edit/",
    name="item_update"
)
def item_update(
    request: Request,
    item_id: int,
    text: str = Form(default=""),
    db: Session = Depends(get_db)
):

    # 수정할 기존 데이터 조회
    item = db.get(TextItem, item_id)

    # 데이터가 존재하지 않으면 404
    if item is None:
        raise HTTPException(status_code=404)

    # 수정한 문장 검사
    value = checked(text)

    # 잘못된 입력이면 수정하지 않고 오류 출력
    if value is None:

        return templates.TemplateResponse(
            request=request,
            name="sentiment/textitem_form.html",
            context={
                "item": item,
                "text": text,
                "error": "문장은 1~500자로 입력하세요."
            },
            status_code=400
        )


    # --------------------------------------------------------
    # ⑯ [04 신규] AI 재분석 → 데이터 수정 → DB 저장
    # --------------------------------------------------------
    # 수정한 문장을 AI 모델로 다시 분석합니다.
    #
    # 이후 TextItem의 값을 변경하고 commit()합니다.
    #
    # 수정 항목:
    # text  : 수정된 문장
    # label : 새 감정분석 결과
    # score : 새 감정분석 점수

    # AI 감정분석 다시 실행
    result = analyze_text(value)

    # ORM 객체의 컬럼값 수정
    item.text = value
    item.label = result["label"]
    item.score = result["score"]

    # DB 수정사항 확정
    db.commit()

    # 수정된 글 상세 페이지로 이동
    return RedirectResponse(
        request.url_for(
            "item_detail",
            item_id=item_id
        ),
        status_code=303
    )


# ============================================================
# ⑰ [04 신규] 글 삭제 확인 화면 열기 (DELETE 화면)
# ============================================================
# GET /items/{item_id}/delete/
#
# 삭제할 글을 조회하고
# 정말 삭제할지 확인하는 HTML 화면을 보여줍니다.
#
# 이 단계에서는 아직 삭제하지 않습니다.

@router.get(
    "/items/{item_id}/delete/",
    response_class=HTMLResponse,
    name="item_delete_page"
)
def item_delete_page(
    request: Request,
    item_id: int,
    db: Session = Depends(get_db)
):

    # 삭제할 글 조회
    item = db.get(TextItem, item_id)

    # 존재하지 않으면 404
    if item is None:
        raise HTTPException(status_code=404)

    # 삭제 확인 화면 출력
    return render(
        request,
        "textitem_confirm_delete.html",
        item=item
    )


# ============================================================
# ⑱ [04 신규] 글 삭제 실행 (DELETE)
# ============================================================
# POST /items/{item_id}/delete/
#
# 사용자가 삭제 확인 버튼을 누르면 실행됩니다.
#
# 1. 삭제할 데이터 조회
# 2. db.delete()로 삭제 등록
# 3. db.commit()으로 삭제 확정
# 4. 글 목록 페이지로 이동

@router.post(
    "/items/{item_id}/delete/",
    name="item_delete"
)
def item_delete(
    item_id: int,
    db: Session = Depends(get_db)
):

    # 삭제할 글 조회
    item = db.get(TextItem, item_id)

    # 글이 없으면 404
    if item is None:
        raise HTTPException(status_code=404)

    # 해당 객체 삭제 등록
    db.delete(item)

    # DB 삭제 확정
    db.commit()

    # 목록 페이지로 이동
    return RedirectResponse(
        "/items/",
        status_code=303
    )
```

**의사코드**

```
================================================
[PART 1] FastAPI 라우터 기본 설정
================================================

① 필요한 라이브러리 가져오기
   ├─ FastAPI 요청·응답 기능
   ├─ SQLAlchemy DB 기능
   ├─ db.py → get_db()
   ├─ models.py → TextItem
   └─ services.py → analyze_text()
             ↓

② 라우터와 템플릿 설정
   ├─ router = APIRouter()
   └─ templates = Jinja2Templates(...)
             ↓

③ render() 함수 정의
   └─ HTML 파일과 데이터를 받아 화면 출력
             ↓

④ checked() 함수 정의
   ├─ 문장 앞뒤 공백 제거
   ├─ 1~500자 검사
   ├─ 정상이면 정리된 문장 반환
   └─ 잘못된 입력이면 None 반환
             ↓

⑤ GET / 요청
   └─ /items/ 목록 페이지로 이동


================================================
[PART 2] 기존 02 감정분석 기능 유지
================================================

⑥ GET /predict/
   └─ 빈 감정분석 입력 화면 출력
             ↓

   사용자가 문장을 입력하고 [분석하기] 클릭
             ↓

⑦ POST /predict/
   └─ checked()로 입력값 검사
             │
       ┌─────┴─────┐
       │           │
     잘못됨       정상
       │           │
       ▼           ▼
   오류 400     ⑧ analyze_text()
                   │
                   ▼
               hf_model.py
                   │
                   ▼
                AI 분석
                   │
                   ▼
               label / score
                   │
                   ▼
              predict.html
              분석 결과 표시

   ※ 이 기능은 DB에 저장하지 않음


================================================
[PART 3] 저장된 글 목록 조회 (READ)
================================================

⑨ GET /items/
   └─ FastAPI에서 get_db() 호출
             ↓
      DB 세션 전달받기
             ↓
      TextItem 테이블 조회
             ↓
      id 내림차순 정렬
             ↓
      textitem_list.html
      글 목록 출력


================================================
[PART 4] 새로운 글 작성 및 저장 (CREATE)
================================================

⑩ GET /new/
   └─ 새로운 글 작성 화면 출력
             ↓

   사용자가 문장 입력 후 [저장] 클릭
             ↓

⑪ POST /new/
   └─ checked()로 입력값 검사
             │
       ┌─────┴─────┐
       │           │
     잘못됨       정상
       │           │
       ▼           ▼
   오류 400     ⑫ AI 분석 및 저장
                   │
                   ▼
              analyze_text()
                   │
                   ▼
             label / score 생성
                   │
                   ▼
              TextItem 객체 생성
                   │
                   ▼
                 db.add()
                   │
                   ▼
                db.commit()
                   │
                   ▼
               db.refresh()
                   │
                   ▼
           DB에 저장된 고유번호 확인
                   │
                   ▼
           상세 페이지로 이동 (303)


================================================
[PART 5] 저장된 글 상세 조회 (READ)
================================================

⑬ GET /items/{item_id}/
   └─ DB 세션 받아오기
             ↓
      db.get(TextItem, item_id)
             ↓
       글이 존재하는가?
             │
       ┌─────┴─────┐
       │           │
      없음         있음
       │           │
       ▼           ▼
     404 오류    상세 화면 출력
                  (textitem_detail.html)


================================================
[PART 6] 기존 글 수정 (UPDATE)
================================================

⑭ GET /items/{item_id}/edit/
   └─ 기존 글 조회
             ↓
      수정 입력 화면 출력
             ↓

   사용자가 문장을 수정하고 [저장] 클릭
             ↓

⑮ POST /items/{item_id}/edit/
   ├─ 기존 글 조회
   └─ checked()로 수정 문장 검사
             │
       ┌─────┴─────┐
       │           │
     잘못됨       정상
       │           │
       ▼           ▼
   오류 400     ⑯ AI 재분석 및 수정
                   │
                   ▼
              analyze_text()
                   │
                   ▼
            새로운 label / score
                   │
                   ▼
             item.text 수정
             item.label 수정
             item.score 수정
                   │
                   ▼
                db.commit()
                   │
                   ▼
            상세 페이지로 이동 (303)


================================================
[PART 7] 기존 글 삭제 (DELETE)
================================================

⑰ GET /items/{item_id}/delete/
   └─ 삭제할 글 조회
             ↓
      삭제 확인 화면 출력
             ↓

   사용자가 [삭제] 버튼 클릭
             ↓

⑱ POST /items/{item_id}/delete/
   └─ 삭제할 글 조회
             │
       글이 존재하는가?
             │
       ┌─────┴─────┐
       │           │
      없음         있음
       │           │
       ▼           ▼
     404 오류    db.delete(item)
                   │
                   ▼
                db.commit()
                   │
                   ▼
              목록 페이지로 이동 (303)
```

**02와 04의 변경 사항**

|구분|02 교재|04 교재|변경 내용|
|---|---|---|---|
|감정분석|`get_classifier()` 직접 호출|`analyze_text()` 호출|수정|
|입력값 검사|`predict()` 안에서 직접 검사|`checked()` 공통 함수|수정|
|HTML 출력|`TemplateResponse()` 반복|`render()` 공통 함수|수정|
|DB 연결|없음|`get_db()`|신규|
|DB 테이블 모델|없음|`TextItem`|신규|
|글 목록|없음|`GET /items/`|신규|
|글 작성·저장|없음|`GET/POST /new/`|신규|
|상세 조회|없음|`GET /items/{item_id}/`|신규|
|수정|없음|`GET/POST /items/{item_id}/edit/`|신규|
|삭제|없음|`GET/POST /items/{item_id}/delete/`|신규|

현재 `router.py`는 두 가지 기능을 담당합니다

|구분|`/predict/`|`/new/`|
|---|---|---|
|목적|AI 감정분석 결과 확인|AI 분석 후 DB에 저장|
|AI 분석|실행|실행|
|DB 저장|없음|있음|
|처리 방식|동기식|동기식|
|교재|02에서 시작|04에서 추가|

현재 04 교재의 연결 구조

```
                 사용자
                    │
                    ▼
                main.py
                    │
                    ▼
            sentiment/router.py
                    │
          ┌─────────┴──────────┐
          │                    │
          ▼                    ▼
   [기존 02 기능]         [04 신규 기능]
    /predict/               /new/
          │                    │
          ▼                    ▼
   analyze_text()       analyze_text()
          │                    │
          ▼                    ▼
      AI 분석               AI 분석
          │                    │
          ▼                    ▼
      결과 출력             TextItem 생성
                               │
                               ▼
                          DB 세션(db)
                               │
                               ▼
                          SQLite 저장
                               │
                               ▼
                           결과 조회
```

여기서 `analyze_text()`는 별도의 `services.py` 파일에 있고, 두 기능이 공통으로 사용합니다.
두개의 파일로 나눈것이 아니고 URL 기능은 두 그룹으로 구분했지만, 라우터 파일 자체는 아직 하나입니다

---
## 4. FastAPI 앱에 DB·라우터 등록

`02`에서 이미 `sentiment/router.py`를 생성했습니다. 이제 수정한 라우터를 import하고 모델을 DB 메타데이터에 등록합니다.

**`main.py` — 전체 교체**

```python

# ============================================================
# ① [02 유지] FastAPI 실행에 필요한 라이브러리 가져오기
# ============================================================
# FastAPI     : 웹 애플리케이션을 생성하는 클래스
# StaticFiles : CSS, JavaScript, 이미지 등 정적 파일 제공
#
# 02 교재에서 작성한 코드와 동일합니다.

from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles


# ============================================================
# ② [02 유지] 감정분석 라우터 가져오기
# ============================================================
# sentiment/router.py의 router 객체를 가져옵니다.
#
# 02에서는 /predict/ 기능만 연결했지만,
# 04에서는 router.py가 CRUD 기능까지 확장되었습니다.
#
# import 코드는 동일하지만 연결되는 기능이 늘어났습니다.

from sentiment.router import router as sentiment_router


# ============================================================
# ③ [04 신규] 데이터베이스 연결 정보 가져오기
# ============================================================
# sentiment/db.py에서 두 가지를 가져옵니다.
#
# Base   : ORM 테이블 모델의 공통 부모 클래스
# engine : SQLite 데이터베이스 연결 관리 객체
#
# 02에서는 DB를 사용하지 않았으므로 없던 코드입니다.

from sentiment.db import Base, engine


# ============================================================
# ④ [04 신규] 데이터베이스 테이블 모델 가져오기
# ============================================================
# sentiment/models.py를 가져옵니다.
#
# models.py에는 TextItem 모델이 정의되어 있습니다.
# TextItem은 text_items 테이블의 구조를 나타냅니다.
#
# 이 파일을 import하면 SQLAlchemy가
# TextItem 테이블 정보를 Base.metadata에 등록합니다.
#
# 주의:
# 이 import를 제거하지 않습니다.
# 실제 테이블 생성 전 모델 정보를 등록하는 역할입니다.

from sentiment import models


# ============================================================
# ⑤ [02→04 수정] FastAPI 애플리케이션 생성
# ============================================================
# app : FastAPI 애플리케이션 객체
# title : Swagger UI(/docs)에 표시되는 제목
#
# [02 기존]
# title="FastAPI Sentiment Demo"
#
# [04 수정]
# title="FastAPI Sentiment CRUD"
#
# 감정분석 서비스에 CRUD 기능이 추가되었으므로
# 프로젝트 제목을 변경했습니다.

app = FastAPI(
    title="FastAPI Sentiment CRUD"
)


# ============================================================
# ⑥ [04 신규] SQLite 데이터베이스 테이블 생성
# ============================================================
# Base.metadata :
# SQLAlchemy에 등록된 ORM 테이블 설계 정보
#
# create_all(bind=engine) :
# engine이 연결한 DB에 없는 테이블을 생성합니다.
#
# 현재 models.py에 정의한 TextItem 모델을 기준으로
# text_items 테이블을 생성합니다.
#
# 주의:
# 이미 존재하는 테이블은 다시 생성하지 않습니다.
# 기존 테이블에 컬럼을 추가하거나 수정하는
# 마이그레이션 기능은 아닙니다.
#
# 실무의 DB 스키마 변경에는 Alembic 등을 사용합니다.

Base.metadata.create_all(
    bind=engine
)


# ============================================================
# ⑦ [02 유지] 정적 파일 폴더 등록
# ============================================================
# /static : 브라우저에서 접근하는 URL
# static  : 실제 정적 파일 폴더
# name    : Jinja2에서 사용할 정적 파일 경로 이름
#
# 예:
# /static/css/sentiment.css
#
# 02 교재와 동일하게 사용합니다.

app.mount(
    "/static",
    StaticFiles(directory="static"),
    name="static"
)


# ============================================================
# ⑧ [02 유지 + 04 기능 확장] 감정분석 라우터 등록
# ============================================================
# 02에서 작성한 include_router() 코드와 동일합니다.
#
# 다만 04의 router.py에는 다음 기능이 추가되었습니다.
#
# 기존 기능:
# GET  /predict/ : 감정분석 입력 화면
# POST /predict/ : 감정분석 실행 및 결과 출력
#
# 신규 기능:
# GET  /items/   : 저장된 글 목록 조회
# GET  /new/     : 새 글 작성 화면
# POST /new/     : 감정분석 및 DB 저장
#
# 그 외 상세 조회, 수정, 삭제 경로도 등록됩니다.
#
# router.py에 정의된 API들을
# FastAPI 애플리케이션에 연결합니다.

app.include_router(
    sentiment_router
)


# ============================================================
# ⑨ [02 유지] 서버 상태 확인 API 등록
# ============================================================
# 사용자가 GET /health 주소로 요청하면
# health() 함수가 실행됩니다.
#
# 기존 02 코드와 동일합니다.

@app.get(
    "/health",
    name="health"
)
def health():

    # --------------------------------------------------------
    # ⑩ [02 유지] 서버 정상 응답 반환
    # --------------------------------------------------------
    # Python 딕셔너리를 JSON으로 반환합니다.
    #
    # 예상 결과:
    # {"status": "ok"}
    #
    # 주의:
    # 이 응답만으로 DB 연결이나 AI 모델이
    # 정상 작동한다고 판단할 수는 없습니다.

    return {
        "status": "ok"
    }
```

**의사코드**

```
================================================
 [1단계] 기존 FastAPI 기능 준비
================================================

① [02 유지] FastAPI 라이브러리 가져오기

    FastAPI 가져오기
    StaticFiles 가져오기

                 ↓

② [02 유지] 감정분석 라우터 가져오기

    sentiment/router.py에서
    router 객체 가져오기

    sentiment_router 이름으로 사용

    ※ 04의 router.py에는 CRUD 기능도 포함


================================================
 [2단계] 새로운 SQLite DB 기능 추가
================================================

③ [04 신규] 데이터베이스 연결 정보 가져오기

    sentiment/db.py에서

    Base 가져오기
    engine 가져오기

                 ↓

④ [04 신규] ORM 테이블 모델 가져오기

    sentiment/models.py 가져오기

    TextItem 모델을 SQLAlchemy에 등록

    테이블 이름 : text_items

                 ↓

⑤ [02→04 수정] FastAPI 애플리케이션 생성

    app = FastAPI()

    프로젝트 제목 변경

    기존 : FastAPI Sentiment Demo
    변경 : FastAPI Sentiment CRUD

                 ↓

⑥ [04 신규] 데이터베이스 테이블 생성

    Base.metadata.create_all(bind=engine)

                 ↓

    SQLAlchemy가 등록된 테이블 정보 확인

                 ↓

    SQLite에 text_items 테이블이 있는가?

           ┌─────┴─────┐
           │           │
          없음         있음
           │           │
           ▼           ▼
        테이블 생성   기존 테이블 유지
           │           │
           └─────┬─────┘
                 ▼

          DB 준비 완료


================================================
 [3단계] FastAPI 웹 기능 연결
================================================

⑦ [02 유지] 정적 파일 폴더 연결

    /static URL 등록

    CSS, JavaScript, 이미지 제공

                 ↓

⑧ [02 유지 + 04 확장] 라우터 등록

    app.include_router(sentiment_router)

                 ↓

    /predict/ 감정분석 기능 등록

    /new/ 글 작성 및 DB 저장 기능 등록

    /items/ 목록 조회 기능 등록

    상세 조회 · 수정 · 삭제 기능 등록

                 ↓

⑨ [02 유지] 서버 상태 확인 API 등록

    GET /health 등록

                 ↓

⑩ [02 유지] 정상 상태 JSON 반환

    {
        "status": "ok"
    }


================================================
 [4단계] FastAPI 서버 실행
================================================

    python -m uvicorn main:app --reload

                 ↓

          FastAPI 서버 시작

                 ↓

          사용자 요청 대기

                 ↓

       사용자가 URL로 접속
                 │
       ┌─────────┼─────────┐
       │         │         │
       ▼         ▼         ▼

   /predict/   /new/     /items/
       │         │         │
       ▼         ▼         ▼

    AI 분석   AI 분석    DB 목록 조회
       │         │
       ▼         ▼

    결과 출력  SQLite 저장
                 │
                 ▼
             상세 화면 출력
```

---
## 5. 목록·상세·입력·삭제 확인 화면 (HTML + Jinja2)

`02`에서 작성한 `templates/base.html`, `templates/sentiment/predict.html`, CSS는 그대로 둡니다. 추가 HTML에는 Django의 `{% url %}`, `{% csrf_token %}`, `floatformat:4`를 사용하지 않고 Jinja2의 `url_for()`, `format()`을 사용합니다. 수업에서는 로그인/세션 없이 로컬에서만 사용합니다. 상용 서비스는 인증·권한·CSRF 보호를 추가해야 합니다.

**HTML + Jinja2 방식이란?**

HTML + Jinja2는 FastAPI 서버에서 데이터를 HTML에 전달하여 웹 화면을 동적으로 생성하는 방식입니다.

HTML은 웹페이지의 기본 구조를 만들고, Jinja2는 Python(FastAPI)에서 전달받은 데이터를 HTML에 표시하는 **템플릿 엔진(Template Engine)**입니다.

**HTML + Jinja2 방식의 장점**

|장점|핵심 설명|
|---|---|
|개발이 간단함|React, Vue.js 없이 Python과 HTML로 웹서비스 구현 가능|
|데이터 출력이 편리함|Python에서 전달한 변수를 `{{ result.label }}`처럼 HTML에 표시|
|코드 역할 분리|FastAPI는 요청·데이터 처리, Jinja2는 화면 출력 담당|
|반복 코드 감소|`base.html` 상속으로 공통 메뉴와 화면 구조 재사용|
|CRUD 구현에 적합|목록·상세·작성·수정·삭제 화면을 쉽게 구성|
|학습이 쉬움|Python과 기본 HTML을 아는 초보자가 전체 흐름을 이해하기 좋음|

**`templates/sentiment/textitem_list.html` — 신규 생성**

```html
{% extends "base.html" %}{% block content %}
<h1>감정 기록 목록</h1><a class="btn" href="{{ url_for('item_create_page') }}">새 글 작성</a>
<div class="card"><ul>
{% for item in items %}<li><a href="{{ url_for('item_detail', item_id=item.id) }}">{{ item.text|truncate(40) }}</a>
<span class="meta">({{ item.label }} / {% if item.score is not none %}{{ '%.4f'|format(item.score) }}{% else %}미분석{% endif %})</span></li>
{% else %}<li>저장된 글이 없습니다.</li>{% endfor %}
</ul></div>{% endblock %}
```

**`templates/sentiment/textitem_detail.html` — 신규 생성**

```html
{% extends "base.html" %}{% block content %}
<h1>감정 기록 상세</h1><div class="card"><p>입력: {{ item.text }}</p><p>라벨: {{ item.label }}</p>
<p>점수: {% if item.score is not none %}{{ '%.4f'|format(item.score) }}{% else %}미분석{% endif %}</p></div>
<a class="btn" href="{{ url_for('item_update_page', item_id=item.id) }}">수정</a>
<a class="btn" href="{{ url_for('item_delete_page', item_id=item.id) }}">삭제</a>
<a class="btn" href="{{ url_for('item_list') }}">목록</a>{% endblock %}
```

**`templates/sentiment/textitem_form.html` — 신규 생성**

```html
{% extends "base.html" %}{% block content %}
<h1>{% if item %}글 수정{% else %}새 글 작성{% endif %}</h1>
<form method="post" action="{% if item %}{{ url_for('item_update', item_id=item.id) }}{% else %}{{ url_for('item_create') }}{% endif %}">
<label for="text">문장 입력</label><textarea id="text" name="text" rows="4" maxlength="500">{{ text }}</textarea>
{% if error %}<p class="error">{{ error }}</p>{% endif %}
<button type="submit">저장(자동 분석)</button><a class="btn" href="{{ url_for('item_list') }}">목록</a></form>
{% endblock %}
```

**`templates/sentiment/textitem_confirm_delete.html` — 신규 생성**

```html
{% extends "base.html" %}{% block content %}
<h1>삭제 확인</h1><p>{{ item.text }}</p><p>정말 삭제할까요?</p>
<form method="post" action="{{ url_for('item_delete', item_id=item.id) }}">
<button type="submit">삭제</button><a class="btn" href="{{ url_for('item_detail', item_id=item.id) }}">취소</a>
</form>{% endblock %}
```

---
## 6. 직접 실행하기

**01:** 기존 서버가 실행 중이면 `Ctrl+C`로 종료합니다. 04의 파일을 작성한 후:

```bash
cd ~/fastapi_sentiment
source .venv/bin/activate
python -m uvicorn main:app --reload
```

**02:** `http://127.0.0.1:8000/predict/`에서 02의 즉시 감정분석이 그대로 되는지 확인합니다. **03:** `http://127.0.0.1:8000/new/`에서 새 문장 작성 후 저장하면 `/items/<id>/` 상세 페이지로 이동해야 합니다. **04:** 목록 → 상세 → 수정 → 삭제를 순서대로 실행합니다. **05:** DB 파일이 실제로 생성됐는지 `ls -lh sentiment.db`로 확인합니다.

**DBeaver 사용(선택):** SQLite 연결 → 프로젝트의 `sentiment.db` 선택 → `text_items` 테이블 조회. 04 단계는 Docker·Redis·Celery를 설치하지 않아도 실행됩니다.

### 6-1. 정상 기대 결과

```text
POST /new/   → 감정분석 수행 → DB 저장 → 303 Redirect
GET /items/ → 저장된 글 목록
GET /items/1/ → 분석 결과 상세
POST /items/1/edit/ → 입력 재분석 후 수정
POST /items/1/delete/ → DB 삭제
```

![[Pasted image 20260218162742.png]]

> **원본 이미지 참조 위치 유지:** CRUD 결과 화면 뒤에 원래 이미지 참조를 유지했습니다.

## 7. 자동 테스트: DB CRUD 및 기존 기능 회귀검사

**`tests/test_web.py` — 전체 교체**

```python

# ============================================================
# ① [02 유지] 자동 테스트에 필요한 라이브러리 가져오기
# ============================================================
# TestClient : FastAPI 서버를 별도로 실행하지 않고 API 테스트
# app        : main.py에서 생성한 FastAPI 애플리케이션
# router_mod : 라우터의 함수를 테스트용으로 교체하기 위해 사용
#
# 02 교재와 동일한 import 코드입니다.

from fastapi.testclient import TestClient
from main import app
import sentiment.router as router_mod


# ============================================================
# ② [02 유지] FastAPI 테스트 클라이언트 생성
# ============================================================
# TestClient(app) :
# FastAPI의 API에 GET / POST 요청을 보내는 테스트 도구
#
# Uvicorn 서버를 직접 실행하지 않아도 됩니다.

client = TestClient(app)


# ============================================================
# ③ [04 신규] 가짜 AI 감정분석 함수 작성
# ============================================================
# 02에서는 monkeypatch 안에 lambda로 가짜 모델을 작성했습니다.
#
# 04에서는 새로운 fake_analyze() 함수를 만들었습니다.
#
# 실제 Hugging Face 모델을 실행하지 않고
# 미리 정해진 감정분석 결과를 반환합니다.
#
# 문장에 "좋"이 포함되어 있으면 Positive
# 그렇지 않으면 Negative
# 점수는 항상 0.91을 반환합니다.
#
# 예:
# "좋아요" → Positive
# "싫어요" → Negative

def fake_analyze(text):

    return {
        "label": "Positive" if "좋" in text else "Negative",
        "score": 0.91
    }


# ============================================================
# ④ [02 유지] 서버 상태 확인 테스트
# ============================================================
# GET /health API가 정상적인 JSON을 반환하는지 검사합니다.

def test_health():

    # --------------------------------------------------------
    # ⑤ [02 유지] 서버 상태 응답값 확인
    # --------------------------------------------------------
    # client.get() : GET 요청 보내기
    # .json()      : JSON 응답을 Python 딕셔너리로 변환
    # assert       : 예상한 결과와 일치하는지 검사
    #
    # 예상 결과: {"status": "ok"}

    assert client.get("/health").json() == {
        "status": "ok"
    }


# ============================================================
# ⑥ [04 신규] CRUD 전체 기능 테스트
# ============================================================
# 02에서는 test_page_and_invalid()로
# 입력 화면과 잘못된 문장만 검사했습니다.
#
# 04에서는 test_crud()로 확장하여
# 입력 화면, 입력값 검사, 생성, 조회, 수정, 삭제를
# 하나의 테스트 함수에서 순서대로 검사합니다.
#
# monkeypatch :
# 실제 AI 분석 함수를 테스트용 함수로 임시 교체합니다.

def test_crud(monkeypatch):

    # --------------------------------------------------------
    # ⑦ [02→04 수정] AI 분석 함수를 가짜 함수로 교체
    # --------------------------------------------------------
    # 02에서는 get_classifier()를 교체했습니다.
    #
    # 04에서는 router.py가 analyze_text()를 호출하므로
    # 해당 함수를 fake_analyze()로 교체합니다.
    #
    # 이 테스트에서는 실제 AI 모델을 실행하지 않습니다.

    monkeypatch.setattr(
        router_mod,
        "analyze_text",
        fake_analyze
    )


    # --------------------------------------------------------
    # ⑧ [02 유지] 기존 감정분석 입력 화면 확인
    # --------------------------------------------------------
    # GET /predict/
    #
    # 02에서 만든 기존 입력 화면이
    # 04에서도 정상적으로 열리는지 검사합니다.
    #
    # 예상 결과: HTTP 200

    assert client.get(
        "/predict/"
    ).status_code == 200


    # --------------------------------------------------------
    # ⑨ [02→04 수정] 잘못된 입력값 검사
    # --------------------------------------------------------
    # 02에서는 POST /predict/의 잘못된 입력을 검사했습니다.
    #
    # 04에서는 새롭게 추가한 POST /new/에서
    # 공백만 입력했을 때 저장되지 않는지 검사합니다.
    #
    # 예상 결과: HTTP 400

    assert client.post(
        "/new/",
        data={
            "text": "  "
        }
    ).status_code == 400


    # --------------------------------------------------------
    # ⑩ [04 신규] 새 글 작성 및 DB 저장 테스트 (CREATE)
    # --------------------------------------------------------
    # POST /new/
    #
    # "좋아요" 문장을 전달합니다.
    #
    # router.py에서는 다음 작업을 수행합니다.
    #
    # 1. 입력값 검사
    # 2. fake_analyze() 호출
    # 3. TextItem 객체 생성
    # 4. SQLite DB 저장
    # 5. 상세 페이지로 이동
    #
    # follow_redirects=False :
    # 페이지 이동을 자동으로 따라가지 않고
    # HTTP 303 응답을 직접 확인합니다.

    r = client.post(
        "/new/",
        data={
            "text": "좋아요"
        },
        follow_redirects=False
    )

    # 저장 후 상세 페이지 이동 응답 확인
    assert r.status_code == 303


    # --------------------------------------------------------
    # ⑪ [04 신규] 저장된 글 상세 조회 테스트 (READ)
    # --------------------------------------------------------
    # headers["location"] :
    # 서버가 이동하도록 지정한 상세 페이지 URL
    #
    # 예: http://testserver/items/1/
    #
    # 해당 주소로 GET 요청을 보내고
    # Positive 결과가 화면에 표시되는지 검사합니다.

    uri = r.headers["location"]

    assert "Positive" in client.get(uri).text


    # --------------------------------------------------------
    # ⑫ [04 신규] 저장된 글 목록 조회 테스트 (READ)
    # --------------------------------------------------------
    # GET /items/
    #
    # 저장된 데이터가 목록 화면에도
    # 표시되는지 검사합니다.

    assert "Positive" in client.get(
        "/items/"
    ).text


    # --------------------------------------------------------
    # ⑬ [04 신규] 기존 글 수정 테스트 (UPDATE)
    # --------------------------------------------------------
    # POST /items/{item_id}/edit/
    #
    # 기존 "좋아요" 문장을 "싫어요"로 수정합니다.
    #
    # fake_analyze("싫어요")는 Negative를 반환합니다.
    #
    # router.py에서 수정된 문장과 감정분석 결과를
    # SQLite DB에 다시 저장합니다.
    #
    # 예상 결과: HTTP 303

    r = client.post(
        uri + "edit/",
        data={
            "text": "싫어요"
        },
        follow_redirects=False
    )

    assert r.status_code == 303


    # --------------------------------------------------------
    # ⑭ [04 신규] 수정된 감정분석 결과 확인
    # --------------------------------------------------------
    # 상세 페이지를 다시 조회합니다.
    #
    # 기존 Positive에서 Negative로
    # 변경되었는지 확인합니다.

    assert "Negative" in client.get(uri).text


    # --------------------------------------------------------
    # ⑮ [04 신규] 저장된 글 삭제 테스트 (DELETE)
    # --------------------------------------------------------
    # POST /items/{item_id}/delete/
    #
    # 데이터 삭제 요청을 보냅니다.
    #
    # router.py에서 db.delete()와 db.commit()을
    # 실행하여 데이터를 삭제합니다.
    #
    # 예상 결과: HTTP 303

    assert client.post(
        uri + "delete/",
        follow_redirects=False
    ).status_code == 303


    # --------------------------------------------------------
    # ⑯ [04 신규] 삭제된 데이터가 없는지 확인
    # --------------------------------------------------------
    # 삭제한 글의 상세 URL로 다시 접속합니다.
    #
    # 데이터가 삭제되었다면 조회할 수 없어야 합니다.
    #
    # 예상 결과: HTTP 404 Not Found

    assert client.get(uri).status_code == 404


# ============================================================
# ⑰ [02→04 수정] 기존 감정분석 기능 회귀 테스트
# ============================================================
# 02에서도 test_predict() 함수가 있었습니다.
#
# 변경된 점:
# 기존 get_classifier() 대신 analyze_text()를 교체
#
# /predict/ 기능이 CRUD 추가 후에도
# 정상적으로 동작하는지 검사합니다.
#
# 회귀 테스트(Regression Test):
# 새 기능을 추가하거나 코드를 수정한 뒤
# 기존 기능이 여전히 동작하는지 확인하는 테스트

def test_predict(monkeypatch):

    # 실제 AI 분석 함수를 가짜 함수로 교체
    monkeypatch.setattr(
        router_mod,
        "analyze_text",
        fake_analyze
    )

    # 기존 감정분석 API에 문장 전달
    # Positive 결과가 HTML에 표시되는지 검사
    assert "Positive" in client.post(
        "/predict/",
        data={
            "text": "좋아요"
        }
    ).text
```

**의사코드**

```
================================================
[PART 1] 테스트 환경 준비
================================================

① 테스트 라이브러리 가져오기

   TestClient 가져오기
   main.py의 app 가져오기
   sentiment/router.py 가져오기

                 ↓

② 테스트 클라이언트 생성

   client = TestClient(app)

   FastAPI에 테스트 요청을 보낼 준비

                 ↓

③ 가짜 감정분석 함수 만들기

   fake_analyze(text)

   문장에 "좋"이 포함되어 있는가?

        ┌──────┴──────┐
        │             │
       예           아니요
        │             │
        ▼             ▼
     Positive      Negative

   score는 항상 0.91 반환


================================================
[PART 2] 서버 상태 확인
================================================

④ test_health() 실행

   GET /health 요청

                 ↓

⑤ 서버 응답 확인

   {"status": "ok"} 인가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL


================================================
[PART 3] CRUD 기능 전체 테스트
================================================

⑥ test_crud(monkeypatch) 실행

                 ↓

⑦ 실제 AI 분석 함수 교체

   router.py의 analyze_text()를

   fake_analyze()로 임시 교체

                 ↓

⑧ 기존 감정분석 입력 화면 확인

   GET /predict/

   HTTP 200인가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL

                 ↓

⑨ 잘못된 입력값 검사

   POST /new/

   text = "  "

   checked()에서 공백 제거
   → 빈 문장으로 판단

   HTTP 400인가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL

                 ↓

⑩ 새 글 작성 (CREATE)

   POST /new/

   text = "좋아요"

                 ↓

   fake_analyze("좋아요")

                 ↓

   {
       "label": "Positive",
       "score": 0.91
   }

                 ↓

   TextItem 객체 생성

                 ↓

   db.add()
   db.commit()
   db.refresh()

                 ↓

   SQLite DB에 저장

                 ↓

   HTTP 303인가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL

                 ↓

⑪ 저장된 글 상세 조회 (READ)

   응답의 Location 주소 확인

   예: /items/1/

                 ↓

   GET /items/1/

   HTML에 "Positive"가 있는가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL

                 ↓

⑫ 저장된 글 목록 조회 (READ)

   GET /items/

   HTML에 "Positive"가 있는가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL

                 ↓

⑬ 기존 글 수정 (UPDATE)

   POST /items/1/edit/

   text = "싫어요"

                 ↓

   fake_analyze("싫어요")

                 ↓

   {
       "label": "Negative",
       "score": 0.91
   }

                 ↓

   TextItem 정보 수정

   db.commit()

                 ↓

   HTTP 303인가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL

                 ↓

⑭ 수정 결과 확인

   GET /items/1/

   HTML에 "Negative"가 있는가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL

                 ↓

⑮ 저장된 글 삭제 (DELETE)

   POST /items/1/delete/

                 ↓

   db.delete()
   db.commit()

                 ↓

   HTTP 303인가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL

                 ↓

⑯ 삭제 여부 확인

   GET /items/1/

   HTTP 404인가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL


================================================
[PART 4] 기존 감정분석 기능 회귀 테스트
================================================

⑰ test_predict(monkeypatch) 실행

   analyze_text()를 가짜 함수로 교체

                 ↓

   POST /predict/

   text = "좋아요"

                 ↓

   fake_analyze() 실행

                 ↓

   Positive 반환

                 ↓

   predict.html 결과 확인

   HTML에 "Positive"가 있는가?

        ├─ 예    → PASS
        └─ 아니요 → FAIL


================================================
       모든 테스트 완료
================================================
```

테스트 코드를 실행합니다.

```bash
python -m pytest -q
```

**02 교재와 비교해 실제로 변경된 부분**

|구분|02 교재|04 교재|
|---|---|---|
|`TestClient(app)`|사용|그대로 유지|
|`test_health()`|서버 상태 확인|그대로 유지|
|가짜 AI|`lambda`를 직접 사용|`fake_analyze()` 함수 신규 작성|
|잘못된 입력 테스트|`/predict/`|`/new/`|
|CRUD 테스트|없음|`test_crud()` 추가|
|AI 함수 교체 대상|`get_classifier()`|`analyze_text()`|
|기존 기능 검사|`test_predict()`|`test_predict()` 유지·수정|
|DB 저장 테스트|없음|SQLite에 실제 생성·수정·삭제 수행|

---
## 8. 06으로 이어지는 핵심 개념

**04 현재 동기 처리:** 사용자가 저장 버튼을 누른 후 모델 추론이 끝나야 DB에 결과가 저장되고 HTTP 응답을 받습니다.

**06에서 바꿀 비동기 처리:** DB에 입력 글을 먼저 저장하고 작업 ID를 Redis 브로커에 등록합니다. Celery 워커가 나중에 모델을 실행하고 DB의 결과를 갱신합니다. 두 방식을 같은 `/predict/`와 `/new/`에서 비교합니다.
