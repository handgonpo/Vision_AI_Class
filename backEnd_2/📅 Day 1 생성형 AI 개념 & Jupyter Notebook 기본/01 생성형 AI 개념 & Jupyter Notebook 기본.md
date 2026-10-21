# 🎯 **오늘의 최종 목표**

- LLM(대규모 언어모델)의 개념과 동작 원리 이해
- Jupyter Notebook 완벽 적응
- OpenAI API를 활용해 기본 챗봇 구현
- `.env` 파일로 API 키를 안전하게 로드하는 방법 숙지
- AI 앱의 기본 구조(Prompt → LLM → Response) 이해

### 🟦 생성형 AI & LLM 개념

🎯 목표

AI가 텍스트를 생성하는 원리 이해 → “AI는 확률 기반 예측기”

---
📌 1) 생성형 AI란?

|구분|설명|
|---|---|
|**규칙기반 AI(과거)**|if-else, 전문가 시스템|
|**기계학습(2010년대)**|이미지 분류, 추천 알고리즘|
|**생성형 AI(2022~)**|텍스트/이미지/코드 생성 LLM 시대|

---
📌 2) LLM(Large Language Model) 개념

- 대량의 텍스트 데이터를 기반으로 학습된 초대규모 딥러닝 모델
- 다음 단어의 확률을 예측하는 모델
- GPT-4o-mini, GPT-4, Claude 3.5 Sonnet, Llama 3 등
    
---
📌 3) LLM의 기본 구조

“Prompt → 모델 → Token 생성 → Response”
1. 사용자가 Prompt 입력
2. 모델이 token-by-token으로 생성
3. 응답이 형성됨
    
---
📌 4) 실습 준비

- 오늘 배울 것:
    - Prompt 설계
    - API 호출
    - Notebook 기반 AI 활용
- 이후 2~3주차:  
    → LangChain, Streamlit, 자동화 AI 앱까지 이어짐
    
---
### 🟦 Prompt 기초 & LLM 응답 패턴 분석

🎯 목표
“Prompt Engineer”처럼 모델을 잘 다루기 위한 기초 이해

---
📌 Prompt = AI에게 주는 명령어/정보

Prompt 기본 구조
```
역할(role) 지정
문맥(context)
질문(question)
형식(format) 지정
```

---
📌 2) 나쁜 프롬프트 vs 좋은 프롬프트 예시

❌ 나쁜 프롬프트
```
설명해줘
```

✅ 좋은 프롬프트
```
너는 Python 전문가다. list, dict의 차이를 초보자 기준으로 설명하고 표 형태로 정리해줘.
```

---
🧪 실습

Notebook에서:
```
prompt = """
너는 친절한 Python 입문 강사다.
변수(variable)의 개념을 초보자도 이해할 수 있게 예시 포함 설명해줘.
"""
```

(아직 API는 안 쓰고 "야, 이런 질문을 보내는 거야" 개념만 실습)

---
### 🟦 Jupyter Notebook 환경 이해

🎯 목표
Notebook 개발 방식 완전 이해: 실행 단위, Markdown, 셀 구조

---
📌 Jupyter Notebook 실행

WSL에서:
```bash
uv add notebook uv run jupyter notebook
```
또는 VSCode 내부에서 직접 실행.

---

📌 Notebook UI 구성 설명
- 셀(Cell) : 코드 실행 단위
- Markdown 셀: 설명/문서 작성
- 단축키
    - `A` 위에 셀 추가
    - `B` 아래에 셀 추가
    - `M` Markdown 전환
    - `Y` 코드 전환
    - `Shift + Enter` 실행
        
---
🧪 실습
- Markdown 셀로 제목 만들기
- 코드 셀로 단순 계산 실행
- 이미지/수식 추가하는 법
    
---
### 🟦 Notebook 기본 문법 & Python 복습

🎯 목표
AI 실습 전에 Python Notebook 작업 방식 적응

---
📌 Python 기본 연습
```python
name = "Eunice"
age = 35
print(f"이름: {name}, 나이: {age}")
```

---
📌 간단한 데이터 처리
```python
nums = [1,3,5,7]
sum(nums)
```

---
📌 간단한 시각화 맛보기
```python
import matplotlib.pyplot as plt
plt.plot([1,2,3], [3,2,1])
plt.show()
```

---
🧪 실습

- Markdown으로 다양한 텍스트 꾸미기
- Python 셀에서 list/dict 실습
    
---
### 🟦 OpenAI API 세팅 & .env 구성

🎯 목표
API 키 관리 → 안전한 개발 습관 형성

---
`1)` python-dotenv 설치
```bash
uv add python-dotenv openai
```

---
`2)` .env 파일 생성
프로젝트 폴더에:
```
OPENAI_API_KEY="sk-xxxxxx..."
```

---
`3)` Notebook에서 .env 로딩
```python
from dotenv import load_dotenv
import os

load_dotenv()
api_key = os.getenv("OPENAI_API_KEY")
api_key
```

---
⚠️ 보안 강조

- 절대 GitHub에 .env 올리면 안 됨
- `.gitignore` 설정 실습
    
---
### 🟦 GPT-4o-mini 챗봇 기본 실습

🎯 목표
기본적인 “대화형 챗봇” 구현

---
📌 기본 코드 실행
```python
from openai import OpenAI
client = OpenAI(api_key=api_key)

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "user", "content": "안녕 GPT!"}
    ]
)
print(response.choices[0].message["content"])
```

---
🔍 분석할 내용
- model 이름
- messages 구조
- user / assistant role
- response 구조
- token 개념
    
---
🧪 실습

`1)` “친절한 영어 선생님” 챗봇 프롬프트 실습
`2)` “Python 디버깅 도우미” 챗봇 실습
`3)` “식단 추천 AI” 만들기
프롬프트 구조에 따라 결과가 어떻게 달라지는지 체감하기

---
### 🟦 나만의 Q&A 챗봇 만들기

🎯 목표
프롬프트 기반으로 사용자 정의 챗봇 제작

---
📌 실습 코드 템플릿
```python
system_prompt = """
너는 사용자의 질문을 간단하고 정확하게 요약해주는 Assistant이다.
'이해하기 쉬운 말'로 답변하도록 한다.
"""

def ask_ai(q):
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system","content": system_prompt},
            {"role": "user", "content": q}
        ]
    )
    return response.choices[0].message["content"]

ask_ai("장고에서 ORM이 뭐야?")
```

---
🧪 실습 과제
- “운동 추천 챗봇”
- “MBTI 챗봇”
- “영문 이메일 번역 챗봇”
- “코딩 오류 해결 챗봇”
    
---
### 🟦 오류 해결법 + Notebook 정리 + Git 업로드

🎯 목표
- Notebook 정리
- API 오류 해결법 학습
- GitHub 업로드로 포트폴리오 기반 만들기
    
---
`1)` 자주 발생하는 오류 모음

|오류|원인|
|---|---|
|401|API 키 오류|
|429|초과 요청|
|FileNotFoundError|.env 경로 문제|
|ModuleNotFoundError|uv add 미적용|

---
`2)` Notebook 정리
- 섹션 별 Markdown 제목 추가
- 코드 결과 이미지 저장
- 파일명: `day1_intro_to_llm.ipynb`
    
---
`3)` GitHub 업로드
```bash
git init
git add .
git commit -m "Add Day1 LLM Notebook"
git remote add origin https://github.com/<username>/ai_day1.git
git push -u origin main

```

---

최종 산출물
- `day1_intro_to_llm.ipynb`
- `.env` (로컬)
- README.md
- GitHub 리포지토리 생성 완료
    
---
### 🧠 수업 후 얻게 되는 역량

- Notebook 기반 AI 실습 능력
- Prompt 설계 기본 이해
- OpenAI API 사용 능력
- `.env` 기반 보안 관리
- LLM 응답 구조 이해
- 이후 LangChain·Streamlit로 넘어갈 수 있는 AI 기초 체력